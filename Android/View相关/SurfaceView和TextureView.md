### SurfaceView和TextureView

SurfaceView和TextureView都是Android中用于显示视频和图片的高性能组件。

- SurfaceView有自己的surface，有自己的windowstate和layer，有子线程来控制绘制和刷新等。所以不具备View的一些特性，例如动画。但是性能好。
- TextureView是View的子类，通过surfacetexture来显示内容，具备view的特性，可以做动画操作，例如旋转、平移、缩放、透明度等动画等。对输出到surface的内容定制度高。

### 应用场景

- SurfaceView：使用高性能的绘制要求。例如流媒体播放、游戏等。
- TextureView：对动画、用户交互和客制化绘制内容有要求的场景。例如相机实时预览、视频录制、取景预览。