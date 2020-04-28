---
title: iOS_interview09
p: iOS_interview/iOS_interview13.md
date: 2020-04-04 12:19:10
tags:
---

iOS面试题集合

## iOS有关Runtime的面试题

### 结构模型
这块考察的是OC的对象创建，以及OC的内存结构（对象的堆内存结构）
1、介绍下runtime的内存模型（isa、对象、类、metaclass、结构体的存储信息等）

(1) OC对象很简单，只有一个指向所属对象的isa指针（Class类型  objc_class指针类型）
``` C
/// objc.h
/// Represents an instance of a class.
struct objc_object {
    Class _Nonnull isa  OBJC_ISA_AVAILABILITY;      // 指向所属类的isa指针
};

/// A pointer to an instance of a class.
typedef struct objc_object *id;
```

(2) OC类对象(Class / *objc_class)
``` C
/// runtime.h
struct objc_class {
    /// 指向metaclass的isa指针
    Class _Nonnull isa  OBJC_ISA_AVAILABILITY;      

#if !__OBJC2__
    /// 指向父类的 Class 指针
    Class _Nullable super_class                              OBJC2_UNAVAILABLE;
    const char * _Nonnull name                               OBJC2_UNAVAILABLE;
    long version                                             OBJC2_UNAVAILABLE;
    long info                                                OBJC2_UNAVAILABLE;
    long instance_size                                       OBJC2_UNAVAILABLE;
    /// 实例列表
    struct objc_ivar_list * _Nullable ivars                  OBJC2_UNAVAILABLE;
    /// 方法列表
    struct objc_method_list * _Nullable * _Nullable methodLists                    OBJC2_UNAVAILABLE;
    /// objc_msgSend涉及的方法缓存
    struct objc_cache * _Nonnull cache                       OBJC2_UNAVAILABLE;
    /// 协议列表
    struct objc_protocol_list * _Nullable protocols          OBJC2_UNAVAILABLE;
#endif

} OBJC2_UNAVAILABLE;
/* Use `Class` instead of `struct objc_class *` */

/// objc.h
/// An opaque type that represents an Objective-C class.
typedef struct objc_class *Class;
``` 

(3) objc_method_list & 
``` C
///  runtime.h
struct objc_method_list {
    struct objc_method_list * _Nullable obsolete             OBJC2_UNAVAILABLE;

    int method_count                                         OBJC2_UNAVAILABLE;
#ifdef __LP64__
    int space                                                OBJC2_UNAVAILABLE;
#endif
    /* variable length structure */
    struct objc_method method_list[1]                        OBJC2_UNAVAILABLE;
}
/// An opaque type that represents a method in a class definition.
typedef struct objc_method *Method;                                                        OBJC2_UNAVAILABLE;

struct objc_method {
    /// SEL
    SEL _Nonnull method_name                                 OBJC2_UNAVAILABLE;
    /// 返回值和参数类型的 type encoding
    char * _Nullable method_types                            OBJC2_UNAVAILABLE;
    /// 方法实现的函数指针
    IMP _Nonnull method_imp                                  OBJC2_UNAVAILABLE;
}                                                            OBJC2_UNAVAILABLE;

/// objc.h
/// SEL的定义
/// An opaque type that represents a method selector.代表一个方法的不透明类型
typedef struct objc_selector *SEL;

/// IMP的定义
#if !OBJC_OLD_DISPATCH_PROTOTYPES
typedef void (*IMP)(void /* id, SEL, ... */ ); 
#else
typedef id _Nullable (*IMP)(id _Nonnull, SEL _Nonnull, ...); 
#endif
```
SEL: 
* objc_msgSend函数第二个参数类型为SEL，它是selector在OC中的表示类型(Swift中则是Selector)。也可以理解是区分方法的ID，SEL是这个ID的数据结构
* selector实质是映射到方法的C字符串，可以使用编译器命令@selector()或者 Runtime 系统的sel_registerName函数来获得一个 SEL 类型的方法选择器
* selector既然是一个C字符串，应该是ClassName+MethodName组合： 所以同一个类，selector只区分name和method， 不记录参数，所以重载不能参数不同的方法

IMP: 一个指向方法实现的C函数指针

(4) category_t
``` C
struct category_t {
    /// 类名
    const char *name;
    /// 要拓展的类对象，编译期间是不会定义的，在运行时通过name映射到对应的类对象
    classref_t cls;
    struct method_list_t *instanceMethods;
    struct method_list_t *classMethods;
    struct protocol_list_t *protocols;
    struct property_list_t *instanceProperties;
    // Fields below this point are not always present on disk.
    struct property_list_t *_classProperties;

    method_list_t *methodsForMeta(bool isMeta) {
        if (isMeta) return classMethods;
        else return instanceMethods;
    }

    property_list_t *propertiesForMeta(bool isMeta, struct header_info *hi);
};
```

2. 为什么要设计metaclass
metaclass在消息机制里面的角色

3. category 和 extension的区别
* 本质区别：
extension是编译器决议的，是类的组成部分，在编译期间和头文件的@interface&@implementation一起组成一个完整的类，extension用来隐藏累的私有信息，必须有类的源码才能添加extension

category是运行期决议的，和extension不同，根据结构体定义发现category是无法添加实例变量的，因为一个类的内存结构是在编译期就已已经决定了，而只能通过runtime的方式关联属性

* category的作用
官方：1. 将类的功能分散不同的文件，归纳功能，减少单个文件的源码，按需加载 2. 声明私有方法
开发者：1. 模拟多继承 2. 公开framework的私有方法 


* category的真面目：在编译期的数据结构
``` C
typedef struct category_t {
    // Category 关联的className
    const char *name;
    classref_t *cls;
    struct method_list_t *instanceMethods;
    struct method_list_t *classMethods;
    struct protocol_list_t *protocols;
    struct property_list_t *instanceProperties;
}
```

可以书写简单的Category的代码示例，使用clang的命令，查看编译器对Category的编译
`clang -rewrite-objc xxx.m`

    1. 首先编译器生成Category的方法列表和属性列表：OBJC$_CATEGORY_INSTANCE_METHODSMyClass$_MyAddition和属性列表OBJC$_PROP_LISTMyClass$_MyAddition，两者的命名都遵循了公共前缀+类名+category名字的命名方式。有一个需要注意到的事实就是category的名字用来给各种列表以及后面的category结构体本身命名，而且有static来修饰，所以在同一个编译单元里我们的category名不能重复，否则会出现编译错误。
    2. 其次，编译器生成了category本身OBJC$_CATEGORYMyClass$_MyAddition，并用前面生成的列表来初始化category本身。
    3. 最后，编译器在DATA段下的objc_catlist section里保存了一个大小为1的category_t的数组L_OBJC_LABELCATEGORY$（当然，如果有多个category，会生成对应长度的数组^_^），用于运行期category的加载。

* category在运行期的加载
在pre-main阶段：dyld加载动态库的时候: 在map_images函数中将项目全局的Category 附加到对应的类上面
``` C
// 注册dylib image变化的回调函数
dyld_register_image_state_change_handler(dyld_image_state_bound,
                                             1/*batch*/, &map_images);
```
1)、把category的实例方法、协议以及属性添加到类上
2)、把category的类方法和协议添加到类的metaclass上

addUnattachedCategoryForClass只是把类和category做一个关联映射，remethodizeClass添加实例方法，而对于添加类的实例方法而言，又会去调用attachCategoryMethods这个方法，我们去看下attachCategoryMethods

需要注意的有两点：

1)、category的方法没有“完全替换掉”原来类已经有的方法，也就是说如果category和原来类都有methodA，那么category附加完成之后，类的方法列表里会有两个methodA

2)、category的方法被放到了新方法列表的前面，而原来类的方法被放到了新方法列表的后面，这也就是我们平常所说的category的方法会“覆盖”掉原来类的同名方法，这是因为运行时在查找方法的时候是顺着方法列表的顺序查找的，它只要一找到对应名字的方法，就会罢休^_^，殊不知后面可能还有一样名字的方法。

* Catefory的load方法
1)、在类的+load方法调用的时候，我们可以调用category中声明的方法么？
2)、这么些个+load方法，调用顺序是咋样的呢？ 鉴于上述几节我们看的代码太多了，对于这两个问题我们先来看一点直观的：
答案：可以调用，因为附加category到类的工作会先于+load方法的执行 2)、+load的执行顺序是先类，后category，而category的+load执行顺序是根据编译顺序决定的。 目前的编译顺序是这样的：

* category的方法和覆盖
怎么调用到原来类中被category覆盖掉的方法？ 对于这个问题，我们已经知道category其实并不是完全替换掉原来类的同名方法，只是category在方法列表的前面而已，所以我们只要顺着方法列表找到最后一个对应名字的方法，就可以调用原来类的方法

* category的关联对象
但是关联对象又是存在什么地方呢？ 如何存储？ 对象销毁时候如何处理关联对象呢？
我们去翻一下runtime的源码，在objc-references.mm文件中有个方法_object_set_associative_reference：

我们可以看到所有的关联对象都是由AssociationsManager管理，
```
class AssociationsManager {
    static OSSpinLock _lock;
    static AssociationsHashMap *_map;               // associative references:  object pointer -> PtrPtrHashMap.
public:
    AssociationsManager()   { OSSpinLockLock(&_lock); }
    ~AssociationsManager()  { OSSpinLockUnlock(&_lock); }
    
    AssociationsHashMap &associations() {
        if (_map == NULL)
            _map = new AssociationsHashMap();
        return *_map;
    }
};
```
AssociationsManager里面是由一个静态的AssociationsHashMap来存储所有的关联对象。这相当于把所有对象的关联对象都存在一个全局map里面，而map的的key是这个对象的指针地址（任意两个不同对象的指针地址一定是不同的），而这个map的value又是另外一个AssociationsHashMap，里面保存了关联对象的kv对。

嗯，runtime的销毁对象函数objc_destructInstance里面会判断这个对象有没有关联对象，如果有，会调用_object_remove_assocations做关联对象的清理工作。

