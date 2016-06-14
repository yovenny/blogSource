title: 关于EditText限制行数｀字符并提示
date: 2016-4-19 17:00:50
tags:
---
 ###产品给出需求，这不是很简单吗？于是在xml添加了如下代码：
```java
   <EditText
	android:maxLeßngth="100"
	android:maxLines="10"/>
```

 而且我只需要在

- TextWatcher.onTextChanged 中监听s.length>100，
- EditText.getLineCount>10 不就行了。

实际运行发现

-  s.length从来就不 >100,（maxLength="100“）字符max后，便不能提示。
-  maxLines（xml）只能对EditText高度进行限制，而不能限制实际字符行数。


 ###于是我便从网上寻找解决方案：

 ／＊＊
  ＊**方式1**：关键地方在beforeTextChanged中文字改变前记录下内容及坐标位置，afterTextChanged中，判断字符和行数是否超过限制，超过设置回之前保存的内容及游标位置，并弹出提示（事先移除掉edittext的textwatcher监听，避免设置内容时引起回调）
  ＊
 ＊＊／

```java
public class LimitedEditText extends EditText {

    /**
     * Max lines to be present in editable text field
     */
    private int maxLines = 1;

    /**
     * Max characters to be present in editable text field
     */
    private int maxCharacters = 50;

    /**
     * application context;
     */
    private Context context;

    public int getMaxCharacters() {
        return maxCharacters;
    }

    public void setMaxCharacters(int maxCharacters) {
        this.maxCharacters = maxCharacters;
    }

    @Override
    public int getMaxLines() {
        return maxLines;
    }

    @Override
    public void setMaxLines(int maxLines) {
        this.maxLines = maxLines;
    }

    public LimitedEditText(Context context, AttributeSet attrs, int defStyle) {
        super(context, attrs, defStyle);
        this.context = context;
    }

    public LimitedEditText(Context context, AttributeSet attrs) {
        super(context, attrs);
        this.context = context;
    }

    public LimitedEditText(Context context) {
        super(context);
        this.context = context;
    }

    @Override
    protected void onFinishInflate() {
        super.onFinishInflate();

        TextWatcher watcher = new TextWatcher() {

            private String text;
            private int beforeCursorPosition = 0;

            @Override
            public void onTextChanged(CharSequence s, int start, int before,
                                      int count) {
                //TODO sth
            }

            @Override
            public void beforeTextChanged(CharSequence s, int start, int count,
                                          int after) {
                text = s.toString();
                beforeCursorPosition = start;
            }

            @Override
            public void afterTextChanged(Editable s) {

            /* turning off listener */
                removeTextChangedListener(this);

            /* handling lines limit exceed */
                if (LimitedEditText.this.getLineCount() > maxLines) {
                    LimitedEditText.this.setText(text);
                    LimitedEditText.this.setSelection(beforeCursorPosition);
                    Toast.makeText(context, "你的描述超出了行数限制", Toast.LENGTH_SHORT)
                            .show();
                }

            /* handling character limit exceed */
                if (s.toString().length() > maxCharacters) {
                    LimitedEditText.this.setText(text);
                    LimitedEditText.this.setSelection(beforeCursorPosition);
                    Toast.makeText(context, "你的描述超出了字数限制", Toast.LENGTH_SHORT)
                            .show();
                }

            /* turning on listener */
                addTextChangedListener(this);

            }
        };

        this.addTextChangedListener(watcher);
    }
```

／／**方式二**：github上一个wedget：https://github.com/ggilrong/LimitedEditText.git 采用InputFilter限制可输入字符数，并onTextChanged中显示将可输入字数显示在label中，咦～～，这不就是微信输入框的效果吗？（不能限制行数，以label显示可输入字数代替toast提示）

／／**方式三**：字符数限制：在onTextChanged中超过字符数时，截取内容为最大的可输入字符，设置并提示。
- 行数限制：在afterTextChanged中超过行数时，（由于无法计算固定行数显示的最大字符量）所以 采用了字符量减－1的形式设置内容，重新触发afteprTextChanged判断EditText.getLineCount,n次循坏后toast提示。
- 其中截取开始位置要根据光标的位置
```java
  public static class SimpleLimitTextWatcher extends  SimpleTextWatcher{
        private boolean isLineTip=false;//标记一次行数超出
        private int mEndIndexOrigin =-1;//标记行数的起始光标位置
        private int mEndIndex;//行数判断的最终设置的光标位置
        private EditText mEdit;
        private int maxLines = 1;
        private int maxCharacters = 50;
        private Context mContext;


        public SimpleLimitTextWatcher(Context context,EditText editText,int maxCharacters,int maxLines){
            mContext=context;
            mEdit=editText;
            this.maxCharacters=maxCharacters;
            this.maxLines=maxLines;
        }

        @Override
        public void onTextChanged(CharSequence s, int start, int before, int count) {
            Editable editable = mEdit.getText();
            int len = editable.length();
            if (len > maxCharacters) {
                String str = s.toString();
                int cursorStart = mEdit.getSelectionStart();
                int cursorEnd = mEdit.getSelectionEnd();
                if (cursorStart == cursorEnd && cursorStart < str.length() && cursorStart >= 1) {
                    str = str.substring(0, cursorStart-(s.length()-maxCharacters)) + str.substring(cursorStart);
                } else {
                    str = str.substring(0, s.length()-(s.length()-maxCharacters));
                }
                int selEndIndex = Selection.getSelectionStart(editable)-(s.length()-maxCharacters);
                mEdit.setText(str);
                editable = mEdit.getText();
                int newLen = editable.length();
                if (selEndIndex > newLen) {
                    selEndIndex = editable.length();
                }
                Selection.setSelection(editable, selEndIndex);
                Toast.makeText(mContext, "你的描述超出了字数限制", Toast.LENGTH_SHORT).show();
            }
        }


        @Override
        public void afterTextChanged(Editable s) {
            int lines = mEdit.getLineCount();
            if (lines > maxLines) {
                Editable editable = mEdit.getText();
                isLineTip=true;
                String str = s.toString();
                int cursorStart = mEdit.getSelectionStart();
                int cursorEnd = mEdit.getSelectionEnd();

                if(mEndIndexOrigin !=-1){
                    cursorStart=cursorEnd= mEndIndexOrigin;
                }else{
                    mEndIndexOrigin =cursorStart;
                }
                if (cursorStart == cursorEnd && cursorStart < str.length() && cursorStart >= 1) {
                    str = str.substring(0, cursorStart-1) + str.substring(cursorStart);
                } else {
                    str = str.substring(0, s.length()-1);
                }
                mEndIndex = mEndIndexOrigin -1;
                mEndIndexOrigin--;
                mEdit.setText(str);//重新出发textwatcher事件
                editable = mEdit.getText();
                int newLen = editable.length();
                if (mEndIndex > newLen) {
                    mEndIndex = editable.length();
                }
                Selection.setSelection(editable, mEndIndex);
            }else{
                if(isLineTip){
                    Toast.makeText(mContext,"你的描述超出了行数限制", Toast.LENGTH_SHORT).show();
                    isLineTip=false;
                    mEndIndexOrigin =-1;
                }
            }
        }
    }
```


###总结：
- 方式一，代码比较简单，不足之处，超过限制不能截取多余字符，达不到原生的效果，
- 方式三，可以截取多余字符，但maxlines判断，循环截取，消耗较大。
- 方式二，UI显示比较友好，不失为Toast提示很好的替代方式，具体要看产品的需求啰。

  ### 写这个的原因：好吧，有点味不足到！！！，希望小小的东西能给自己些许提示吧？。




