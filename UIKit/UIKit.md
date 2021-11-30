

### 通过[UIImage imageNamed:]生成的对象什么时候被释放？

使用imageNamed 方法生成的UIImage对象，会在应用的bundle中寻找图片，如果找到则加入到系统缓存中。作为内存的cache，程序员无法操作cache，只能由系统自动处理。如果我们需要重复加载一张图片,那这无疑是一种很好的方式,因为系统能很快的从内存的cache找到这张图片,但是试想,如果加载很多很大的图片的时候,内存消耗过大的时候,就会会强制释放内存，即会遇到内存警告(memory warnings).

由于在iOS系统中释放图片的内存比较麻烦,所以冲易产生内存泄露。
像[[UIImageView alloc] init]还有一些其他的 init 方法，返回的都是 autorelease 对象。而 autorelease 不能保证什么时候释放，所以不一定在引用计数为 0 就立即释放，只能保证在 autoreleasepool 结尾的时候释放。

像 UIImage 还有 NSData 这种，大部分情况应该是延迟释放的，可以理解为到 autoreleasepool 结束的时候才释放。

