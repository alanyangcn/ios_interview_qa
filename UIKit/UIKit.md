

## 通过[UIImage imageNamed:]生成的对象什么时候被释放？

使用imageNamed 方法生成的UIImage对象，会在应用的bundle中寻找图片，如果找到则加入到系统缓存中。作为内存的cache，程序员无法操作cache，只能由系统自动处理。如果我们需要重复加载一张图片,那这无疑是一种很好的方式,因为系统能很快的从内存的cache找到这张图片,但是试想,如果加载很多很大的图片的时候,内存消耗过大的时候,就会会强制释放内存，即会遇到内存警告(memory warnings).

由于在iOS系统中释放图片的内存比较麻烦,所以冲易产生内存泄露。
像[[UIImageView alloc] init]还有一些其他的 init 方法，返回的都是 autorelease 对象。而 autorelease 不能保证什么时候释放，所以不一定在引用计数为 0 就立即释放，只能保证在 autoreleasepool 结尾的时候释放。

像 UIImage 还有 NSData 这种，大部分情况应该是延迟释放的，可以理解为到 autoreleasepool 结束的时候才释放。

## UIWindow，UIView，CALayer的区别

1. UIWindow
```
@interface UIWindow : UIView

@property(nonatomic) UIWindowLevel windowLevel;                   // default = 0.0
@property(nonatomic,readonly,getter=isKeyWindow) BOOL keyWindow;
- (void)becomeKeyWindow;                               // override point for subclass. Do not call directly
- (void)resignKeyWindow;                               // override point for subclass. Do not call directly

- (void)makeKeyWindow;
- (void)makeKeyAndVisible;                             // convenience. most apps call this to show the main window and also make it key. otherwise use view hidden property

@property(nullable, nonatomic,strong) UIViewController *rootViewController NS_AVAILABLE_IOS(4_0);  // default is nil
@end
```
继承自UIView，是一种特殊的 UIView，通常在一个app中只会有一个keyUIWindow。

iOS程序启动完毕后，创建的第一个视图控件就是UIWindow，接着创建控制器的view，最后将控制器的view添加到UIWindow上，于是控制器的view就显示在屏幕上了

主要作用是提供一个区域用来显示UIView；将事件分发给UIView；与UIViewController一起处理屏幕的旋转事件。

2. UIView

```
@interface UIView : UIResponder <NSCoding, UIAppearance, UIAppearanceContainer, UIDynamicItem, UITraitEnvironment, UICoordinateSpace, UIFocusItem, UIFocusItemContainer, CALayerDelegate>

@property(nonatomic,readonly,strong) CALayer  *layer;
@end

@interface UIResponder : NSObject <UIResponderStandardEditActions>
```
继承自UIResponder，间接继承自NSObject，主要是用来构建用户界面的，并且可以响应事件。

对于UIView，侧重于对内容的显示管理；其实是相对于CALayer的高层封装。

3. CALayer

```
@interface CALayer : NSObject <NSSecureCoding, CAMediaTiming>
```

直接继承自NSObject，所以不能响应事件

其实就是一个图层，UIView之所以能显示在屏幕上，主要是它内部有一个CALayer对象。在创建UIView时，它内部会自动创建一个图层，当UIView需要显示在屏幕上的时候，会调用drawRect:方法进行绘图，并且会将所有内容绘制到自己的图层上，绘图完毕后，系统会将图层拷贝到屏幕上，这样完成UIView的显示。


layer给view提供了基础设施，使得绘制内容和呈现更高效动画更容易、更低耗
layer不参与view的事件处理、不参与响应链


## 如何提升 tableview 的流畅度？

本质上是降低 CPU、GPU 的工作，从这两个大的方面去提升性能。

CPU：对象的创建和销毁、对象属性的调整、布局计算、文本的计算和排版、图片的格式转换和解码、图像的绘制
GPU：纹理的渲染
卡顿优化在 CPU 层面
尽量用轻量级的对象，比如用不到事件处理的地方，可以考虑使用 CALayer 取代 UIView
不要频繁地调用 UIView 的相关属性，比如 frame、bounds、transform 等属性，尽量减少不必要的修改
尽量提前计算好布局，在有需要时一次性调整对应的属性，不要多次修改属性
Autolayout 会比直接设置 frame 消耗更多的 CPU 资源
图片的 size 最好刚好跟 UIImageView 的 size 保持一致
控制一下线程的最大并发数量
尽量把耗时的操作放到子线程
文本处理（尺寸计算、绘制）
图片处理（解码、绘制）
卡顿优化在 GPU层面
尽量避免短时间内大量图片的显示，尽可能将多张图片合成一张进行显示
GPU能处理的最大纹理尺寸是 4096x4096，一旦超过这个尺寸，就会占用 CPU 资源进行处理，所以纹理尽量不要超过这个尺寸
尽量减少视图数量和层次
减少透明的视图（alpha<1），不透明的就设置 opaque 为 YES
尽量避免出现离屏渲染
iOS 保持界面流畅的技巧

1.预排版，提前计算
在接收到服务端返回的数据后，尽量将 CoreText 排版的结果、单个控件的高度、cell 整体的高度提前计算好，将其存储在模型的属性中。需要使用时，直接从模型中往外取，避免了计算的过程。

尽量少用 UILabel，可以使用 CALayer 。避免使用 AutoLayout 的自动布局技术，采取纯代码的方式

2.预渲染，提前绘制
例如圆形的图标可以提前在，在接收到网络返回数据时，在后台线程进行处理，直接存储在模型数据里，回到主线程后直接调用就可以了

避免使用 CALayer 的 Border、corner、shadow、mask 等技术，这些都会触发离屏渲染。

3.异步绘制
4.全局并发线程
5.高效的图片异步加载
