---
layout: post
title: "Swift Runtime - 类和对象"
date: 2019-04-22
categories: iOS
tags: [iOS, Swift]
---

## 编译阶段
```swift
class PureSwiftClass {
    private var private_var_property = 0
    @objc private var objc_private_var_property = 0
    var instance_property = 0
    @objc let objc_instance_let_property = 0
    @objc var objc_instance_var_property = 0

    func instance_method() {}
    @objc func objc_instance_method() {}
    @objc dynamic func objc_dynamic_instance_method() {}
}
```
下面是编译阶段生成的类信息：
```
_$s10TestObjectSwiftClassCN:
struct __objc_class {
    _OBJC_METACLASS_$__TtC10TestObjectSwiftClass, // metaclass
    _OBJC_CLASS_$_SwiftObject, // superclass
    __objc_empty_cache, // cache
    0x0, // vtable
    __objc_class__TtC10TestObjectSwiftClass_data+1 // data
}

__objc_class__ObjectSwiftClass_data:
struct __objc_data {
    0x80, // flags
    8,// instance start
    48,                                  // instance size
    0x0,
    0x0,                                 // ivar layout
    "ObjectSwiftClass",                     // name
    __objc_class__TtC10TestObjectSwiftClass_methods, // base methods
    0x0,                                 // base protocols
    __objc_class__TtC10Test6ObjectSwiftClass_ivars, // ivars
    0x0,                                 // weak ivar layout
    __objc_class__TtC10TestObjectSwiftClass_properties // base properties
}

// methods
__objc_class__ObjectSwiftClass_methods:
struct __objc_method_list { 
    0x18,                                // flags
    8                                    // method count
}

struct __objc_method {                                 
    "objc_private_instance_var_property",                     // name
    "q16@0:8",                              // signature
    -[_TtC10TestObjectSwiftClass objc_private_instance_var_property] // implementation
}
struct __objc_method {                                 
    "setObjc_private_var_property:",                     // name
}
struct __objc_method {
    "objc_instance_var_property",                     // name
}
struct __objc_method {
    "setObjc_instance_var_property:",                     // name
}
struct __objc_method {                                 
    "objc_instance_let_property",                     // name
}
struct __objc_method {                                 
    "objc_instance_method",                     // name
}
struct __objc_method {                                 
    "objc_dynamic_instance_method",                     // name
}
struct __objc_method {                                
    "init",                               // name
}

// ivars
__objc_class__TtC10TestObjectSwiftClass_ivars:
struct __objc_ivars {                               
    32,                                  // entsize
    5                                    // count
}
struct __objc_ivar {                                   
    "private_var_property",                     // name
}
struct __objc_ivar {                                   
    "objc_private_var_property",           // name
}
struct __objc_ivar {                                   
    "instance_var_property",                     // name
}
struct __objc_ivar {                                   
    "objc_instance_var_property",           // name
}
struct __objc_ivar {                                   
    "objc_instance_let_property",           // name
}
```
根据上面编译器生成的数据，可以得到一些信息：
#### class
- `Swift`类编译阶段会生成与`Objective-C`一样的类元数据，这也是为什么`Swift`和`Objective-C`可以互相调用。

> 泛型类不会生成类元数据`__objc_class`结构，不过会生成`roData`。 

- `class`如果没有显式继承某个类，都被隐式继承`SwiftObject`。

#### 属性
- 所有属性都会添加到`class_ro_t`中的`ivars`结构中，包括`private`属性。
- 使用`@objc`修饰的属性，`var`属性会添加`set/get`方法，`let`属性只会添加`get`方法。

> `Swift`类的 `属性`可以通过 `objc-runtime`进行修改和获取。

#### 方法
- 使用`@objc`修饰的方法会添加到`ro_class_t`的`methods`结构中。

## Swift结构
### ClassMetadata

`ClassMetadata`是`Swift`中所有类元数据格式。
```c
struct objc_object {
    Class isa;
}
struct objc_class: objc_object {
    Class superclass;
    cache_t cache;           
    class_data_bits_t bits;
}
struct swift_class_t: objc_class {
    uint32_t flags;//类标示
    uint32_t instanceAddressOffset;
    uint32_t instanceSize;//对象实例大小
    uint16_t instanceAlignMask;//
    uint16_t reserved;// 保留字段
    uint32_t classSize;// 类对象的大小
    uint32_t classAddressOffset;// 
    void *description;//类描述
};
```
`Swift`和`Objective-C`的类元数据是共用的，`Swift`类元数据只是`Objective-C`的基础上增加了一些字段。

> 源代码中也有一些地方直接使用 `reinterpret_cast`进行相互转换。

```objc
Class objcClass = [objcObject class];
ClassMetadata *classAsMetadata = reinterpret_cast<const ClassMetadata *>(objcClass);
```

### HeapObject
在`Swift`中，一个`class`对象实际上就是一个`HeapObject`结构体指针。`HeapObject`由`HeapMetadata`和`InlineRefCounts`组成，`HeapMetadata`是类对象元数据的指针，`InlineRefCounts`用于管理引用计数。
```cpp
struct HeapObject {
  HeapMetadata const *metadata;
  InlineRefCounts refCounts;
};
```
- `HeapMetadata`和`Objective-C`中的`isa_t`结构一样，使用`ISA_MASK`获取到类对象。

```objc
@interface ObjcClass: NSObject {
}

ObjcClass *objcObject = [ObjcClass new];
HeapObject *heapObject = static_cast<HeapObject *>(objcObject);
ObjcClass *objcObject2 =  static_cast<ObjcClass *>(heapObject);

[heapObject retain];
```

> 不过因为 `Objective-C`和`Swift`引用计数管理方式不一样，所以转换以后依然要使用之前的方式进行引用计数管理。

`Objective-C`和`Swift`对象结构：
```
Objc对象结构 {
    isa_t,
    实例变量
}
Swift对象结构 {
    metadata,
    refCounts, 
    实例变量
}
```

#### 创建对象
##### swift_allocObject
- `swift_allocObject`方法用于创建一个`Swift`对象。
```cpp
void *swift::swift_slowAlloc(size_t size, size_t alignMask) {
  void *p;
  // This check also forces "default" alignment to use AlignedAlloc.
  if (alignMask <= MALLOC_ALIGN_MASK) {
    p = malloc(size);
  } else {
    size_t alignment = (alignMask == ~(size_t(0)))
                           ? _swift_MinAllocationAlignment
                           : alignMask + 1;
    p = AlignedAlloc(size, alignment);
  }
  if (!p) swift::crash("Could not allocate memory.");
  return p;
}
static HeapObject *_swift_allocObject_(HeapMetadata const *metadata,
                                       size_t requiredSize,
                                       size_t requiredAlignmentMask) {
  auto object = reinterpret_cast<HeapObject *>(
      swift_slowAlloc(requiredSize, requiredAlignmentMask));
  // NOTE: this relies on the C++17 guaranteed semantics of no null-pointer
  // check on the placement new allocator which we have observed on Windows,
  // Linux, and macOS.
  new (object) HeapObject(metadata);//创建一个新对象，
  return object;
}
```
- 根据对象大小做字节对齐处理，之后调用`malloc`分配内存，之后会初始化实例变量。
- `metadata`表示类对象元数据。
- `requiredSize`和`requiredAlignmentMask`表示对象大小和字节对齐方式。

##### swift_initStackObject
- 在某些场景对象创建会被编译器优化为`swift_initStackObject`方法。`swift_initStackObject`在栈上创建一个对象。没有引用计数消耗，也不用`malloc`内存。
```cpp
HeapObject *
swift::swift_initStackObject(HeapMetadata const *metadata,
                             HeapObject *object) {
  object->metadata = metadata;
  object->refCounts.initForNotFreeing();
  return object;
}
```

#### 销毁对象
##### swift_deallocClassInstance
- `swift_deallocClassInstance`用于销毁对象，在对象`dealloc`时调用。
```cpp
void swift::swift_deallocClassInstance(HeapObject *object,
                                       size_t allocatedSize,
                                       size_t allocatedAlignMask) {
#if SWIFT_OBJC_INTEROP
  objc_destructInstance((id)object);
#endif
  swift_deallocObject(object, allocatedSize, allocatedAlignMask);//
}
```

- 调用`objc_destructInstance`方法释放关联对象和弱引用释放处理。

> `Objc runtime`的对象弱引用，不是`Swift`环境的弱引用。 

- 调用`swift_deallocObject`方法调用`free`回收内存。

#### 引用计数相关方法
- `swift_retain`和`objc`的实现类似，对引用计数进行`+1`，`溢出`时将一部分引用计数值保存到`sideTable`中。
- `swift_release`对引用计数进行`-1`，当引用计数为`0`时，调用销毁对象方法。
- `swift_weak`相关的方法用于管理`weak`弱引用。

### SwiftObject
在`Swift`中，一个`class`如果没有显式继承其他的类，都被隐式继承`SwiftObject`。`SwiftObject`实现了`NSObject`协议的所有方法和一部分`NSObject`类的方法。主要是重写了一部分方法，将方法实现改为`Swift`相关方法。
```objc
@interface SwiftObject<NSObject> {
 @private
  Class isa;
  InlineRefCounts refCounts;
}
@end
```

> 没有实现 `resolveInstanceMethod`，`forwardingTargetForSelector`等方法，这些方法可以在找不到特定方法时可以进行动态处理，应该是不想提供纯 `Swift`类在这块的能力。

比如`retain`，`release`方法改为了使用`swift runtime`进行引用计数管理:
```objc
- (id)retain {
  auto SELF = reinterpret_cast<HeapObject *>(self);
  swift_retain(SELF);
  return self;
}
- (void)release {
  auto SELF = reinterpret_cast<HeapObject *>(self);
  swift_release(SELF);
}
```

> 因为纯 `Swift`类不能直接与`Objective-C`交互，那么`SwiftObject`这样设计的目的是什么？

下面是两种使用场景：
- 就是将纯`Swift`类作为`id`参数传递到`Objective-C`方法中使用。
```objc
- (void)test:(id)object {
  [object retain];
  [object performSelector:@selector(objc_instance_method)];
}
```

```swift
let object = NSObject()
test(object)
```

- 使用消息发送的方式调用方法。
```swift
class SwiftClass {
    @objc dynamic func objc_dynamic_instance_method() {}
}
let object = SwiftClass()
object.objc_dynamic_instance_method()
```

> 不过以上场景应该是很少使用的，不清楚还有没有其它目的。而且这样设计的话，纯`Swift`类也应该可以被`Objective-C`直接使用。

## 初始化对象
### Objective-C
#### Objective-C使用Swift-NSObject子类
```swift
class SwiftClass: NSObject {
}
```
```objc
SwiftClass *object = [[SwiftClass alloc] init];
```
- 因为二进制文件中`Swift`类包含了和`Objective-C`一样的类数据信息，所以可以直接使用`Objective-C`的方式创建。

### Swift

#### Swift类
创建一个纯`Swift`类对象。
```swift
class SwiftClass {
}
SwiftClass()
```
##### swift_getInitializedObjCClass
```objc
Class swift::swift_getInitializedObjCClass(Class c) {
  [c self];// 为了确保objc-runtime realize class
  return c;
}
```

```cpp
Class objcClass = swift_getInitializedObjCClass(SwiftClass);
HeapObject *object = swift_allocObject(objcClass);
// 释放
swift_release(object);
```

#### 原生`Objective-C`类
创建一个原生`Objective-C`类对象。
```objc
@interface ObjectClass
@end
```
```swift
ObjectClass()
```
```objc
Class objcClass = swift_getInitializedObjCClass(ObjectClass);
Metadata *metadata = swift_getObjCClassMetadata(objcClass);
ClassMetadata *classMetadata = swift_getObjCClassFromMetadata(metadata);
ObjectClass *object = [classMetadata allocWithZone] init];
// 释放
objc_release(object);
```

> `swift_getObjCClassMetadata`和`swift_getObjCClassFromMetadata`有什么作用？

#### Swift-NSObject子类
创建一个`Swift`-`NSObject`子类对象。
```swift
class SwiftClass: NSObject {
}
SwiftClass()
```
```objc
Class objcClass = swift_getInitializedObjCClass(SwiftClass);
HeapObject *object = objc_allocWithZone(objcClass);
// 释放
objc_release(object);
```

#### Swift泛型类
创建一个`Swift`泛型类对象。
```swift
class GenericClass<T> {
}
GenericClass<Int>()
```
```cpp
MetadataResponse response = swift_getGenericMetadata();
ClassMetadata *classMetadata = swift_allocateGenericClassMetadata();
swift_initClassMetadata2(classMetadata);
HeapObject *object = swift_allocObject(objcClass);
```

- 根据泛型类型作为参数，调用`swift_getGenericMetadata`方法获取类对象缓存。存在缓存直接返回，没有缓存，调用`swift_allocateGenericClassMetadata`方法。

> 每一个不同的泛型类型都会创建一个新的`ClassMetadata`，之后保存到缓存中复用。

swift_allocateGenericClassMetadata：
- 创建一个新的`ClassMetadata`结构。
- 初始化`objc_class`和`swift_class_t`相关的属性, 同时设置`isa`和`roData`。

swift_initClassMetadataImpl：
- 设置`Superclass`，如果没有指明父类，会被设置为`SwiftObject`。
- 初始化`Vtable`。
- 设置`class_ro_t`的`InstanceStart`和`InstanceSize`字段，遍历`ivars`修改每个`ivar`的`offset`。
- 将该类注册到`objc runtime`。

