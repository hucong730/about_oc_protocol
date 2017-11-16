# about_oc_protocol
Objective-C的protocol的本质就是一个Objective-C对象。

>这是objc里runtime对类的定义，就是一个c的结构体，然后里面有个`struct objc_protocol_list * _Nullable protocols`结构体指针存储这个对象遵守的协议。
```objective-c
struct objc_class {
    Class _Nonnull isa  OBJC_ISA_AVAILABILITY;

#if !__OBJC2__
    Class _Nullable super_class                              OBJC2_UNAVAILABLE;
    const char * _Nonnull name                               OBJC2_UNAVAILABLE;
    long version                                             OBJC2_UNAVAILABLE;
    long info                                                OBJC2_UNAVAILABLE;
    long instance_size                                       OBJC2_UNAVAILABLE;
    struct objc_ivar_list * _Nullable ivars                  OBJC2_UNAVAILABLE;
    struct objc_method_list * _Nullable * _Nullable methodLists                    OBJC2_UNAVAILABLE;
    struct objc_cache * _Nonnull cache                       OBJC2_UNAVAILABLE;
    struct objc_protocol_list * _Nullable protocols          OBJC2_UNAVAILABLE;
#endif

} OBJC2_UNAVAILABLE;
```

>下面对这个协议的结构体再进行探究，熟悉数据结构的应该都看得出来，是采用链表的存储方式来存储一个对象的协议，那其中`Protocol`就是协议的本身。
```objective-c
struct objc_protocol_list {
    struct objc_protocol_list * _Nullable next;
    long count;
    __unsafe_unretained Protocol * _Nullable list[1];
};

```

>然后在runtime.h里面搜索`Protocol`，发现了一个类型定义，`Protocol`是一个`objc_object`结构体。
```objective-c
#ifdef __OBJC__
@class Protocol;
#else
typedef struct objc_object Protocol;
#endif
```

>那`objc_object`到底是什么呢？它就是我们熟悉的对象了，还记得id指向任意的oc对象吗？下面看其定义。
```objective-c
/// Represents an instance of a class.
struct objc_object {
    Class _Nonnull isa  OBJC_ISA_AVAILABILITY;
};

/// A pointer to an instance of a class.
typedef struct objc_object *id;
```

>为了进一步验证我的判断，然后翻阅了苹果开源的runtime，[点击查看](https://opensource.apple.com/source/objc4/objc4-208/runtime/Protocol.h.auto.html)，发现`Protocol`就是继承自`Object`的。

```objective-c
#ifndef _OBJC_PROTOCOL_H_
#define _OBJC_PROTOCOL_H_

#import <objc/Object.h>

struct objc_method_description {
	SEL name;
	char *types;
};
struct objc_method_description_list {
        int count;
        struct objc_method_description list[1];
};

@interface Protocol : Object
{
@private
	char *protocol_name;
 	struct objc_protocol_list *protocol_list;
  	struct objc_method_description_list *instance_methods, *class_methods;
#ifdef NeXT_PDO	/* hppa needs 8 byte aligned protocol blocks */
#if defined(__hpux__) || defined(hpux)
	unsigned long	risc_pad; 
#endif /* __hpux__ || hpux */
#endif NeXT_PDO
}

/* Obtaining attributes intrinsic to the protocol */

- (const char *)name;

/* Testing protocol conformance */

- (BOOL) conformsTo: (Protocol *)aProtocolObject;

/* Looking up information specific to a protocol */

- (struct objc_method_description *) descriptionForInstanceMethod:(SEL)aSel;
- (struct objc_method_description *) descriptionForClassMethod:(SEL)aSel;

@end

#endif /* _OBJC_PROTOCOL_H_ */
```

