### View.invalidate和View.requestLayout的区别
- invalidate会触发重绘，但是不会触发measure和layout
- requestLayout会触发measure和layout，但是不会触发重绘