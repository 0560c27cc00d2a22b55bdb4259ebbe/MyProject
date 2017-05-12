---
layout: post
title: Objective-C 消息转发
date: 2017-05-10
---
#### 项目需求  
1. 借助`runtime`获取目标类中的私有方法  
2. 调用获取目标类中的某个私有方法

#### 代码实现(不堪入目)

```
#import <objc/runtime.h>
#import <objc/message.h>

// 方法获取
id LenderClass = objc_getClass("TryViewController");
unsigned int outCount = 0;
// 获取所有的方法列表
Method *methods = class_copyMethodList(LenderClass, &outCount);

for(unsigned int i = 0; i < outCount; ++i)
{
    // 获取方法名称字符串
    Method method = methods[i];
    SEL methodName = method_getName(method);
    NSString *nameString = NSStringFromSelector(methodName);
    if ([nameString isEqualToString:@"clickButtonAction:"])
    {
        TryViewController *class = [LenderClass new];
        ((void (*)(id, SEL, UIButton *))(void *)objc_msgSend)(class, sel_registerName("clickButtonAction:"), nil);
        break;
    }
}
```
#### 方法解读
Objective-C 是一个动态语言，这意味着它不仅需要一个编译器，也需要一个运行时系统来动态得创建类和对象、进行消息传递和转发。也就是说，其实 `[receiver message]` 会被编译器转化为: `objc_msgSend(receiver, selector)`，如下:

```
@implementation TryClang
- (instancetype)init
{
    self = [super init];
    if (self)
    {
        [self show1];
    }
    return self;
}
- (void)show1
{
    NSLog(@"xxxxxxxxxxxxx show1 xxxxxxxxxxxxx");
}

```
使用`clang`命令:`clang -rewrite-objc MyClass.m` 命令将 OC 代码转换成 C++ 代码，更直观的看出方法的调用过程如下：

```
// - (instancetype)init 方法
static instancetype _I_TryClang_init(TryClang * self, SEL _cmd)
{
    self = ((TryClang *(*)(__rw_objc_super *, SEL))(void *)objc_msgSendSuper)((__rw_objc_super){(id)self, (id)class_getSuperclass(objc_getClass("TryClang"))}, sel_registerName("init"));
    if (self)
    {
    	// 现在我们只关注这句代码
        ((void (*)(id, SEL))(void *)objc_msgSend)((id)self, sel_registerName("show1"));
    }
    return self;
}
// - (void)show1 方法
static void _I_TryClang_show1(TryClang * self, SEL _cmd)
{
    NSLog((NSString *)&__NSConstantStringImpl__var_folders_xm_vfzbkt055yng75_3z17ycx7c0000gn_T_TryClang_f35ea0_mi_0);
}
// @end

static __NSConstantStringImpl __NSConstantStringImpl__var_folders_xm_vfzbkt055yng75_3z17ycx7c0000gn_T_TryClang_f35ea0_mi_0 __attribute__ ((section ("__DATA, __cfstring"))) = {__CFConstantStringClassReference,0x000007c8,"xxxxxxxxxxxxx show1 xxxxxxxxxxxxx",33};
```
当编译器遇到一个方法调用时，它会将方法的调用翻译成以下函数中的一个：

1. 发送给对象的父类的消息会使用`objc_msgSendSuper`;
2. 有数据结构作为返回值的方法会使用`objc_msgSendSuper_stret`或`objc_msgSend_stret`;
3. 其它的消息都是使用`objc_msgSend`发送的。

> `id objc_msgSend(id self, SEL op, ...)`
将消息发送给一个对象并返回一个值，其中 `self` 是消息接收者，`op` 是可变参数

默认情况下是没有参数和返回值的，64位下需要转换成这样的方法原型进行调用
> ((void (*)(id, SEL, UIButton *))(void *)objc_msgSend)(class, sel_registerName("clickButtonAction:"), nil);

#### 数据类型
在Objective-C中，所有的对象，都是继承自NSObject的，`objc.h`系统定义了如下的数据类型：
 
```
typedef struct objc_class *Class;
typedef struct objc_object *id;
struct objc_object {
    Class isa;
};
struct objc_class {
    Class isa;
}
 
/// 不透明结构体, selector
typedef struct objc_selector *SEL;
 
/// 函数指针, 用于表示对象方法的实现
typedef id (*IMP)(id, SEL, ...);
```
`id`指代`objc`中的对象，每个对象的在内存的结构并不是确定的，但其首地址指向的肯定是isa（通过isa指针运行时就能获取到`objc_class`）。

`objc_class`表示对象的 Class，它的结构是确定的，由编译器生成。

`SEL` 可以把它理解为一个字符串(标签)。可以用 Objective-C 编译器命令 `@selector()` 或者 `runtime`的`sel_registerName`函数来获得一个`SEL`类型的方法选择器(`@selector(clickButtonAction:)`与`sel_registerName("clickButtonAction:")`作用相同)。
>
1. Objective-C语言系统中`runtime`为我们维护了一个巨大的方法表
2. 在编译期间会扫描全部类的 .h 和 .m 文件，将其中所有的方法加入到方法表中
3. 在使用`@selector()`或`sel_registerName()`时会从这个方法表中查找对应的 `SEL`(没有找到就生成一个`SEL`添加到表中)
4. 在单独的`objc_class`中`SEl`是唯一的
 
`IMP`是一个函数指针。objc中的方法最终会被转换成纯C的函数，`IMP`就是为了表示这些函数的地址。

以`((void (*)(id, SEL, UIButton *))(void *)objc_msgSend)(class, sel_registerName("clickButtonAction:"), nil)` 为例串联整个过程：

1. 把 `class` 和 `SEL` 转发出去
2. 通过`class`的 isa 指针找到对应的类
3. 先在`cache`中查找
4. 然后在`methodLists`中查找`clickButtonAction:`的标签 `SEL`
5. 根据找到的`SEL`查找到对应的`IPM`(key->value来理解)

注：
```
TryClang *clang = [[TryClang alloc] init];
[clang showshow];
    
TrySEL *sel = [[TrySEL alloc] init];
[sel showshow];
```
两个不同的类中有相同的`SEL`，当实现代码相同时他们拥有共有的`IPM`(触发加在`[clang showshow]`与`[sel showshow]`处的断点)
```
(lldb) p (void *)@selector(showshow)
(void *) $0 = 0x000000010c793d73

(lldb) p (void *)@selector(showshow)
(void *) $1 = 0x000000010c793d73
```
这种转发行为使 objc\_msgSend 变得特殊起来。因为它只是简单的查找合适的代码直接并跳转过去，这表示传入任何参数组合都可以，所以它是把这些参数留给`IMP`去读取的。`objc_msgSend` 源码（国外大神用C语言实现):

```
id  c_objc_msgSend( struct mulle_nsobject *self, SEL _cmd, ...)
{
   struct mulle_objc_class    *cls; // 类
   struct mulle_objc_cache    *cache; // 缓存列表
   unsigned int               hash;
   struct mulle_objc_method   *method; // 方法列表
   unsigned int               index; // 当前索引值
   
   if( self)
   {
      cls   = self->isa;
      cache = cls->cache;
      hash  = cache->mask;
      index = (unsigned int) _cmd & hash;
      
      do
      {
         method = cache->buckets[ index];
         if(! method)
            goto recache;
         index = (index + 1) & cache->mask;
      }
      while( method->method_name != _cmd);
      return( (*method->method_imp)( (id) self, _cmd));
   }
   return( (id) self);

	recache:
	/* ... */
	
   return( 0);
}
```
现在我们来看 recache 部分，也就是无法找到`Method`:  
跳转执行`_objc_msgSend_uncached_impcache`(跳转到`_class_lookupMethodAndLoadCache3`由汇编语言的实现回到了 C 函数的实现），这个函数只是简单的调用了另外一个函数`lookUpImpOrForward`。
> 关于`lookUpImpOrForward`这里讲的很[详细](http://draveness.me/message.html)

#### 需求还可以用这种简单快捷的实现：

```
Class class = NSClassFromString(@"TryViewController");
UIViewController *clang = [class new];
SEL selector = NSSelectorFromString(@"clickButtonAction:");
if ([clang respondsToSelector:@selector(clickButtonAction:)])
{
    [clang performSelector:selector];
}
```
`performSelector: withObject:`可以向一个对象传递任何消息，而不需要在编译的时候声明这些方法。`performSelector`只是一个执行动作，底层执行的仍旧是`objc_msgSend` (当`selector`不存在时编译器会直接崩溃)。
> the method `-respondsToSelector:` is not called by the runtime, it's usually called by the user (yourself or APIs that want to know if a delegate, for example, responds to an optional method of the protocol)  
[详细解读](http://stackoverflow.com/questions/4574465/objective-c-respondstoselector)

系统Foundation框架为我们提供了一些方法反射的API，我们可以通过这些API执行将字符串转为SEL等操作：
```
// SEL 和字符串转换
FOUNDATION_EXPORT NSString *NSStringFromSelector(SEL aSelector);
FOUNDATION_EXPORT SEL NSSelectorFromString(NSString *aSelectorName);

// Class 和字符串转换
FOUNDATION_EXPORT NSString *NSStringFromClass(Class aClass);
FOUNDATION_EXPORT Class __nullable NSClassFromString(NSString *aClassName);

// Protocol 和字符串转换
FOUNDATION_EXPORT NSString *NSStringFromProtocol(Protocol *proto) NS_AVAILABLE(10_5, 2_0);
FOUNDATION_EXPORT Protocol * __nullable NSProtocolFromString(NSString *namestr) NS_AVAILABLE(10_5, 2_0);
```
这个消息转发的详细流程参考
[http://yulingtianxia.com/blog/2016/06/15/Objective-C-Message-Sending-and-Forwarding/](http://yulingtianxia.com/blog/2016/06/15/Objective-C-Message-Sending-and-Forwarding/)

参考：

1. [http://oriochan.com/14710029019312.html](http://oriochan.com/14710029019312.html)
2. [http://www.mulle-kybernetik.com/artikel/Optimization/opti-9.html](http://www.mulle-kybernetik.com/artikel/Optimization/opti-9.html)
3. [http://blog.ibireme.com/2013/11/26/objective-c-messaging/](http://blog.ibireme.com/2013/11/26/objective-c-messaging/)
4. [http://blog.cocoabit.com/dong-shou-shi-xian-objc-msgsend/](http://blog.cocoabit.com/dong-shou-shi-xian-objc-msgsend/)
5. [http://www.cocoachina.com/ios/20160111/14927.html](http://www.cocoachina.com/ios/20160111/14927.html)
