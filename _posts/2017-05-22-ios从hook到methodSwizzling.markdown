---
layout:     post    
title:      "从hook到IOS黑魔法－Method Swizzling"    
subtitle:   ""          
date:       2017-05-22            
author:     "elarc"                      
comments:	true
header-img: "img/elarc/methodSwzzling-bg.png"
---


#从hook到IOS黑魔法－Method Swizzling
##一 ，如何hook一个app
### 1，什么是Tweak

>在IOS逆向工程中，各种逆向的补丁统称为Tweak。通常意义上说的越狱开发，就是指开发一个Tweak。
Tweak依赖于Cydia Substrate（mobile Substrate)的动态库,Cydia Substrate在Cydia中，它的作者就是作者Jay Freeman。
Cydia上的Tweak都是基于Cydia Substrate实现的。

   Tweak的实质就是IOS平台的动态库。
动态库即动态链接库（Windows 下的 .dll，Linux 下的 .so，Mac 下的 .dylib/.tbd）。与静态库相反，动态库在编译时并不会被拷贝到目标程序中，目标程序中只会存储指向动态库的引用。等到程序运行时，动态库才会被真正加载进来。
Tweak用的正是dylib这种形式的动态库。

可以在越狱设备的/Library/MobileSubstrate/DynamicLibraries目录下查看手机上存在着的所有Tweak。
这个目录下除dylib外还存在着plist与bundle两种格式的文件，plist文件是用来标识该Tweak的作用范围，而bundle是Tweak所用到的资源文件

### 2，Tweak的种类
IOS的Tweak大致分为两种：

1.  在越狱设备上安装打包的deb格式的安装包
在cydia上发布，需要越狱才能安装，deb格式的安装包，IOS在越狱后，会默认安装Cydia Substrate动态库，它的作用是提供一个系统级的入侵管道，所有的Tweak都可以依赖它来进行开发，目前主流的开发工具有theos和IOSOpenDev，前者是采用makefile的一个编译框架，后者提供了一套xcode项目模版，可以直接使用xcode开发且可调试，但这个项目已经停止更新了，对高版本的xcode支持也不好。

2.  利用重签名直接使用开发者自己证书或企业证书打包成ipa,这样不需要越狱也可以安装,只是这种非越狱的限制比较大,通常只是用来给某个app做些修改注入或者类似的功能。
没有越狱的机器由于系统中没有Cydia Substrate这个库，我们有二个选择，第一个是直接把这个库打包进ipa当中，使用它的api实现注入，第二个是直接修改汇编代码；第一个适用于较为复杂的破解行为，而且越狱tweak代码可以复用，第二种适用于破解一些简单的条件语句。

### 3，Cydia Substrate的组成
Cydia Substrate主要由3部分组成：MobileHooker，MobileLoader 和 safe mode：

* MobileHooker用于替换覆盖系统的方法，这个过程被称为Hooking(挂钩)

它主要包含两个函数：
 ``` 
void MSHookMessageEx(Class class, SEL selector, IMP replacement, IMP *result);
void MSHookFunction(void*function,void* replacement,void** p_original);
MSHookMessageEx 主要作用于Objective-C函数
MSHookFunction 主要作用于C和C++函数
 ```
Logos语法就是对此函数做了一层封装，让编写hook代码变的更直观，上面的例子用的就是logos语法。
它定义一系列的宏和函数，底层调用objc－runtime和fishhook来替换系统或者目标应用的函数

* MobileLoader用来在目标程序启动时根据规则把指定目录的第三方的dylib加载进去，第三方的dylib也就是我们写的破解程序。
启动时MobileLoader会根据dylib的同名plist文件指定的作用范围，有选择的在不同进程里通过dlopen函数打开目录/Library/MobileSubstrate/DynamicLibraries/ 下的所有dylib。

* Safe mode类似于windows的安全模式，当一些系统级的hook代码发生crash时，Cydia Substrate会自动进入安全模式，安全模式下，会禁用所有的第三方动态库。因为APP程序质量参差不齐崩溃再所难免，Tweak本质是dylib，寄生在别人进程里。系统进程一旦出错，可能导致整个进程崩溃,崩溃后就会造成iOScrash。所以CydiaSubstrate引入了安全模式,在安全模式下所有基于CydiaSubstratede 的第三方dylib都会被禁用，便于查错与修复。

#### Mobileloader它是如何做到把第三方的dylib注入进目标程序的呢？

这个我们要从二进制文件的结构说起，从下面的图来看，Mach-O文件的数据主体可分为三大部分，分别是头部（Header）、加载命令（Load commands）、和最终的数据（Data）。mobileloader会在目标程序启动时，会根据指定的规则检查指定目录是否存在第三方库，如果有，则会通过修改二进制的loadCommands，来把自己注入进所有的app当中，然后加载第三方库。

![mach-0.png](/blog/img/elarc/mach-0.png)

下面是别人用machoview来打开一个真实的二进制文件，可以看出，二进制当中所有引用到的动态库都放在Load commands段当中，所以，通过给这个段增加记录，就可以注入我们自己写的动态库了。

![loadCommand.png](/blog/img/elarc/loadCommod.png)

##### 什么是Mach-O

是Mach object文件格式的缩写，是一种可执行文件、目标代码、共享程序库、动态加载代码和核心DUMP。
是a.out格式的一种替代。Mach-O提供更多的可扩展性和更快的符号表信息存取。
Mach-O应用在基于Mach核心的系统上，目前NeXTSTEP、Darwin、Mac OS X（iPhone）都是使用这种可执行文件格式。

*   头部（header structure）。
*   加载命令（load command）。
*   段（segment）。可以拥有多个段（segment），每个段可以拥有零个或多个区域（section）。每一个段（segment）都拥有一段虚拟地址映射到进程的地址空间。
*   链接信息。一个完整的用户级Mach-o文件的末端是链接信息。其中包含了动态加载器用来链接可执行文件或者依赖库所需使用的符号表，字符串表等等。

> Objective-C的首选hook方案为Method Swizzle，他就是iOS的注入原理，类似于windows的钩子，所以我们注入也称为hook。于是，有人就想那既然OC上可以被hook，那就核心代码用C写好了。那就先说说iOS下C函数的hook方案fishhook .

### 4，什么是fishhook
fishhook是一个非常简单的库，它能动态替换运行在IOS设备上Mach-O文件的符号表。是facebook的一个开源工具，github地址：
[fishhook](https://github.com/facebook/fishhook)


fishhook就是对间接符号表的偏移量动的手脚，定位每一个符号的名字，提供一个假的nlist结构体，使用自定义的函数地址来替换。从而达到hook的目的。

 ``` 
 #import <dlfcn.h>
 
 #import <UIKit/UIKit.h>
 
 #import "AppDelegate.h"
 #import "fishhook.h"
 
 static int (*orig_close)(int);
 static int (*orig_open)(const char *, int, ...);
 
 int my_close(int fd) {
   printf("Calling real close(%d)\n", fd);
   return orig_close(fd);
 }
 
 int my_open(const char *path, int oflag, ...) {
   va_list ap = {0};
   mode_t mode = 0;
 
 if ((oflag & O_CREAT) != 0) {
   // mode only applies to O_CREAT
   va_start(ap, oflag);
   mode = va_arg(ap, int);
   va_end(ap);
   printf("Calling real open('%s', %d, %d)\n", path, oflag, mode);
   return orig_open(path, oflag, mode);
 } else {
   printf("Calling real open('%s', %d)\n", path, oflag);
   return orig_open(path, oflag, mode);
  }
 }
 
 int main(int argc, char * argv[])
 {
 @autoreleasepool {
  rebind_symbols((struct rebinding[2]){{"close", my_close, (void *)&orig_close}, {"open", my_open, (void *)&orig_open}}, 2);
 
   // Open our own binary and print out first 4 bytes (which is the same
   // for all Mach-O binaries on a given architecture)
   int fd = open(argv[0], O_RDONLY);
   uint32_t magic_number = 0;
   read(fd, &magic_number, 4);
   printf("Mach-O Magic Number: %x \n", magic_number);
   close(fd);
 
 return UIApplicationMain(argc, argv, nil, NSStringFromClass([AppDelegate class]));
  }
 }
 ``` 
Sample output
``` 
    Calling real open('/var/mobile/Applications/161DA598-5B83-41F5-8A44-675491AF6A2C/Test.app/Test', 0) Mach-O Magic Number: feedface  Calling real close(3)
```  
    fishhook主要通过这个函数
``` 
    rebind_symbols((struct rebinding[2]){{"close", my_close, (void *)&orig_close}, {"open", my_open, (void *)&orig_open}}, 2);
``` 

### 5，Method swizzling
> Objective-C语言是一门动态语言，它将很多静态语言在编译和链接时期做的事放到了运行时来处理。优势在于写代码更具灵活性。如我们可以把消息转发给我们想要的对象，或者随意交换一个方法的实现动态添加属性等等。
    这种特性意味着Objective-C不仅需要一个编译器，还需要一个运行时系统来执行编译的代码。对于Objective-C来说，这个运行时系统就像一个操作系统一样：它让所有的工作可以正常的运行。这个运行时系统即Objc Runtime。Objc Runtime其实是一个Runtime库，它基本上是用C和汇编写的，这个库使得OC有了面向对象的能力。
 
例如类方法本质-类对象调用[NSObject class]：
``` 
     id:谁发送消息
     SEL:发送什么消息
     ((NSObject *(*)(id, SEL))(void *)objc_msgSend)([NSObject class], @selector(alloc));
``` 
例如创建对象的实质：
``` 
     id oldDriver = [[Lsj alloc] init];
 // 用最底层写
     objc_getClass(const char *name) 获取当前类
     sel_registerName(const char *str) 注册个方法编号
     让Lsj这个类对象发送了一个alloc消息，返回一个分配好的内存对象给你;再发送一个消息初始化.
 
     Lsj *oldDriver = objc_msgSend(objc_getClass("Lsj"), sel_registerName("alloc"));
     objc_msgSend(person,@selector(init)); // 无参
``` 
 
 
#####Method swizzling指的是改变一个已存在的选择器对应的实现的过程，它依赖于Objectvie-C的runtime，能够在运行时通过改变类的调度表（dispatch table）中选择器到最终函数间的映射关系。
![dispatchTable.png](/blog/img/elarc/dispatchTable.png)

    SEL：函数名，可通过@selector()获得SEL
    IMP：函数指针
    
具体怎么实现的：
 ``` 
Method origMethod = class_getInstanceMethod(class, origSelector);  //获取SEL的Method Method是一个结构体，IMP就在里面，看看结构：

struct objc_method {
    SEL method_name                                          OBJC2_UNAVAILABLE;
    char *method_types                                       OBJC2_UNAVAILABLE;
    IMP method_imp                                           OBJC2_UNAVAILABLE;
}
IMP origIMP = method_getImplementation(origMethod);  //获取Method中的IMP
IMP获取到了，然后连接SEL到别的IMP：
BOOL class_addMethod(Class cls, SEL name, IMP imp, const char *types);  //先增加新方法名SEL+原来的IMP
IMP method_setImplementation(Method m, IMP imp）;                       //然后将原来的method(SEL)重新分配新的IMP
void method_exchangeImplementations(Method m1, Method m2) //或者可以使用method的交换方法
``` 
  
##### 选择器、方法及实现

 * 选择器（typedef struct objc_selector *SEL）：选择器用于表示一个方法在运行时的名字，一个方法的选择器是一个注册到（或映射到）Objective-C运行时中的C字符串，它是由编译器生成并在类加载的时候被运行时系统自动映射。
 
 * 方法（typedef struct objc_method *Method）：一个代表类定义中一个方法的不明类型。
 
 * 实现（typedef id (*IMP)(id, SEL, ...)）：这种数据类型是实现某个方法的函数开始位置的指针，函数使用的是基于当前CPU架构的标准C调用规约。第一个参数是指向self的指针（也就是该类的某个实例的内存空间，或者对于类方法来说，是指向元类（metaclass）的指针）。第二个参数是方法的选择器，后面跟的都是参数。
 
 一个类（Class）维护一张调度表（dispatch table）用于解析运行时发送的消息；调度表中的每个实体（entry）都是一个方法（Method），其中key值是一个唯一的名字——选择器（SEL），它对应到一个实现（IMP）——实际上就是指向标准C函数的指针。
 
 > Method Swizzling就是改变类的调度表让消息解析时从一个选择器对应到另外一个的实现，同时将原始的方法实现混淆到一个新的选择器。

#####例子1，统计一下iOS应用中每个视图控制器展现给用户的次数：
        
我们可以给每个视图控制器对应的viewWillAppear:实现方法中增加相应的跟踪代码，但是这样做会产生大量重复的代码。子类化可能是另一个选择，但要求你将UIViewController、 UITableViewController、 UINavigationController 以及所有其他视图控制器类都子类化，这也会导致代码重复。那就可以用method swizzling了：
 ``` 
#import
@implementation UIViewController (Tracking)
    
+ (void)load {
  static dispatch_once_t onceToken;
  dispatch_once(&onceToken, ^{
    Class class = [self class];
        
    // When swizzling a class method, use the following:
    // Class class = object_getClass((id)self);
        
    SEL originalSelector = @selector(viewWillAppear:);
    SEL swizzledSelector = @selector(ylw_viewWillAppear:);
        
    Method originalMethod = class_getInstanceMethod(class, originalSelector);
    Method swizzledMethod = class_getInstanceMethod(class, swizzledSelector);
        
    BOOL didAddMethod =
    class_addMethod(class,
    originalSelector,
    method_getImplementation(swizzledMethod),
    method_getTypeEncoding(swizzledMethod));
        
    if (didAddMethod) {
        class_replaceMethod(class,
        swizzledSelector,
        method_getImplementation(originalMethod),
        method_getTypeEncoding(originalMethod));
    } else {
        method_exchangeImplementations(originalMethod, swizzledMethod);
    }
  });
}
    
#pragma mark - Method Swizzling
    
- (void)ylw_viewWillAppear:(BOOL)animated {
    [self ylw_viewWillAppear:animated];
    NSLog(@"viewWillAppear: %@", self);
}
    
@end
``` 
现在，当UIViewController或它子类的任何实例触发viewWillAppear:方法都会打印一条log日志。
    
+load是在一个类最开始加载时调用,因为method swizzling会影响全局，所以减少冒险情况就很重要。+load能够保证在类初始化的时候就会被加载，这为改变系统行为提供了一些统一性
    
Swizzling应该在dispatch_once中实现。因为swizzling会作用于全局，这有点AOP的意思了。我们需要在运行时采取所有可用的防范措施来保障原子性，来确保代码即使在多线程环境下也只会被执行一次。GCD中的diapatch_once就提供这些保障。
 
下面这段代码是否会导致一个死循环：
 ``` 
- (void)ylw_viewWillAppear:(BOOL)animated {
    [self ylw_viewWillAppear:animated];
    NSLog(@"viewWillAppear: %@", NSStringFromClass([self class]));
}
 ``` 
但其实并没有，在Swizzling的过程中，ylw_viewWillAppear:会被重新分配给UIViewController的viewWillAppear:的原始实现。反而，如果我们在这个方法中调用viewWillAppear:才会真的导致死循环，因为这个方法的实现会在运行时被swizzle到viewWillAppear:的选择器。
 
#### 例子2，去掉系统的弹窗：
 
比如一些定位、拍照等权限的弹窗，是系统自动的出现弹窗，我们无法手动隐藏这个弹窗，除非去掉这个功能。
但用Method Swzzling 就可以解决这个问题。 
 ``` 
#import "UIViewController+present.h"
#import <objc/runtime.h>
 
@implementation UIViewController (present)

+ (void)load {
  static dispatch_once_t onceToken;
  dispatch_once(&onceToken, ^{
    Method presentM = class_getInstanceMethod(self.class, @selector(presentViewController:animated:completion:));
    Method presentSwizzlingM = class_getInstanceMethod(self.class, @selector(elarc_presentViewController:animated:));
    method_exchangeImplementations(presentM, presentSwizzlingM);
 });
}
     
- (void)elarc_presentViewController:(UIViewController *)viewControllerToPresent animated:(BOOL)flag {
    if ([viewControllerToPresent isKindOfClass:[UIAlertController class]]) {
       return;
    }
    [self elarc_presentViewController:viewControllerToPresent animated:true];
}
 ``` 
向视图控制器的生命周期中注入操作、事件的响应、视图的绘制，或Foundation中的网络堆栈都是能够利用method swizzling产生明显效果的场景。当然还有一些其他的场景使用swizzling会是一个合适的选择。
#####注意事项
Swizzling被普遍认为是一种巫术，容易导致不可预料的行为和结果。尽管不是最安全的，但是如果你采取下面这些措施，method swizzling还是很安全的。
    
* 始终调用方法的原始实现： API为输入和输出提供规约，但它里面具体的实现其实是个黑匣子，在Method Swizzling过程中不调用它原始的实现可能会破坏一些私有状态，甚至是程序的其他部分。
    
* 避免冲突：给分类方法加前缀，一定要确保不要让你代码库中其他代码（或是依赖库）在做与你相同的事。
    
* 充分理解：不管你多么自信你能够swizzling Foundation、UIKit 或者其他内置框架，请记住所有这些都可能在下一个版本中就不好使。
* 加入 Siri 功能，可在 macOS Sierra 当中语音搜索讯息、文件、照片、网页等，甚至语音建立备忘录或开启 FaceTime 视讯。登陆 macOS Sierra 之后，Siri 终能跨苹果四大平台使用。