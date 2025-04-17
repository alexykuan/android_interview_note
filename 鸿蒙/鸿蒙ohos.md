## 声明式 UI

```ts
@Entry // 声明入口页面 可路由跳转
@Component // 声明UI组件
@State // 声明ui状态成员变量
@Builder // 声明UI构造器 用于在build函数中调用
```

简单的计数器例子 `Index.ets`

```ts
@Entry
@Component
struct Index {

    @State count: number = 0

    build() {
        RelativeContainer() {
            Text(`You have pushed this button ${this.count} times`).alignRules({
                middle: { anchor: '__container__', align: VerticalAlign.center },
                center: { anchor: '__container__', align: HorizontalAlign.center }
            })
            Button('click me').onClick(() => {
                this.count++
            }).alignRules({
                bottom: { anchor: '__container__', align: VerticalAlign.bottom },
                right: { anchor: '__container__', align: HorizontalAlign.right }
            })
        }
    }
}
```

### Entry 和 Component 的生命周期

Entry 的生命周期:

1. `onPageShow`
2. `onPageHide`
3. `onBackPress`

Component 的生命周期

1. `aboutToAppear`
2. `onDidBuild`
3. `aboutToDisappear`

### 自定义 Layout 容器

自定义容器类似于 android 里面的`ViewGroup`，需要重写 `onMeasureSize` 和 `onPlaceChildren`方法，`onMeasureSize`方法主要用于测量子组件的尺寸，并最终确定当前容器的尺寸。`onPlaceChildren`方法主要用于确定子组件的位置。调用`child.layout({x, y})`来确定子组件的位置。

```ts
onMeasureSize(selfLayoutInfo: GeometryInfo, children: Array<Measurable>, constraint: ConstraintSizeOptions)

onPlaceChildren(selfLayoutInfo: GeometryInfo, children: Array<Layoutable>, constraint: ConstraintSizeOptions)
```

## 状态管理

### @Prop 和@Link 装饰器

`@Prop`装饰器用于父组件像子组件传递数据，父组件更新数据后，子组件会自动更新，而子组件更新后不会将更新同步到父组件。

`@Link`装饰器用于共享数据，当子组件或者父组件更新后，都能自动地更新状态

```ts
@Component
struct Father {

    @State text: string = 'what to eat'

    build() {
        //控制改变text
    }
}

@Component
struct Child {
    @Link text: string = 'what to eat'

    build() {
        //控制改变text
    }
}
```

### @Provide 和@Consume 装饰器

`@Provide`和`@Consume`装饰器用于在祖先组件和后代组件之间共享数据，避免在组件之前嵌套很深的时候一层一层传递的麻烦。

### @Observed 和@ObjectLink

`@Observed`装饰器用于声明一个对象，当对象属性发生变化时，会自动更新 UI。装饰在类上。

`@ObjectLink`装饰器用于声明一个对象，当对象发生变化时，会自动更新 UI。装饰在属性上。

`@ObjectLink`无法在自定义组件中使用

### 应用和页面界别状态管理

- LocalStorage：页面级 UI 状态存储，通常用于 UIAbility 内、页面间的状态共享。

- AppStorage：特殊的单例 LocalStorage 对象，由 UI 框架在应用程序启动时创建，为应用程序 UI 状态属性提供中央存储。

- PersistentStorage：持久化存储 UI 状态，通常和 AppStorage 配合使用，选择 AppStorage 存储的数据写入磁盘，以确保这些属性在应用程序重新启动时的值与应用程序关闭时的值相同。

- Environment：应用程序运行的设备的环境参数，环境参数会同步到 AppStorage 中，可以和 AppStorage 搭配使用。
