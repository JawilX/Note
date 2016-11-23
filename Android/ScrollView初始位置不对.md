```java
scrollView.setFocusableInTouchMode(true);
scrollView.setDescendantFocusability(ViewGroup.FOCUS_BEFORE_DESCENDANTS);
// 在布局文件里面设置无效
```

or

```java
scrollView.post(new Runnable() {
    @Override
    public void run() {
        scrollView.fullScroll(View.FOCUS_UP);
    }
});
```

