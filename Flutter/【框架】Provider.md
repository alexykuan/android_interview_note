### 状态管理

状态管理是 flutter 中比较核心的概念，`StatefulWidget`中的`State`对象保存了 ui 的状态，通过`setState`方法来更新 ui。合理的管理 State，可以使得代码更加清晰，维护起来更加方便。同时也使得页面更快的响应状态的变化。

### 状态提升

`StateLiftUp`是指将多个需要某种状态的组件的状态提升到他们的公共的父组件中进行管理。这样，多个组件就可以共享同一个状态，而不需要每个组件都单独管理自己的状态。在`Flutter`中，通过`InheritedWidget`可以方便地实现状态提升。无需通过构造函数传递状态，也不需要通过各种回调来更新状态，实现组件的解耦。

### Provider 的实际运用

通过`Provider#create`暴露一个对象给后代组件。后代组件通过`context.read()`和`context.watch()`来获取和监听暴露的对象。

```dart
Provider(
    create: (_) => Future.value(0),
    child: DescendantsWidget()
);

class DescendantsWidget extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    final int count = Provider.of(context);
    return Text(count.toString());
  }
}
```
