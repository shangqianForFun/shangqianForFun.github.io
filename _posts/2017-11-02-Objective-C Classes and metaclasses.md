---
layout: post
title: "Classes and metaclasses"
---


# Classes and metaclasses

Objective-C is a class-based object system. Each object is an instance of some class; the object's isa pointer points to its class. That class describes the object's data: allocation size and ivar types and layout. The class also describes the object's behavior: the selectors it responds to and instance methods it implements.

The class's method list is the set of instance methods, the selectors that the object responds to. When you send a message to an instance, objc_msgSend() looks through the method list of that object's class (and superclasses, if any) to decide what method to call.

Each Objective-C class is also an object. It has an isa pointer and other data, and can respond to selectors. When you call a "class method" like [NSObject alloc], you are actually sending a message to that class object.

Since a class is an object, it must be an instance of some other class: a metaclass. The metaclass is the description of the class object, just like the class is the description of ordinary instances. In particular, the metaclass's method list is the class methods: the selectors that the class object responds to. When you send a message to a class - an instance of a metaclass - objc_msgSend() looks through the method list of the metaclass (and its superclasses, if any) to decide what method to call. Class methods are described by the metaclass on behalf of the class object, just like instance methods are described by the class on behalf of the instance objects.

What about the metaclass? Is it metaclasses all the way down? No. A metaclass is an instance of the root class's metaclass; the root metaclass is itself an instance of the root metaclass. The isa chain ends in a cycle here: instance to class to metaclass to root metaclass to itself. The behavior of metaclass isa pointers rarely matters, since in the real world nobody sends messages to metaclass objects.

More important is the superclass of a metaclass. The metaclass's superclass chain parallels the class's superclass chain, so class methods are inherited in parallel with instance methods. And the root metaclass's superclass is the root class, so each class object responds to the root class's instance methods. In the end, a class object is an instance of (a subclass of) the root class, just like any other object.

Confused? The diagram may help. Remember, when a message is sent to any object, the method lookup starts with that object's isa pointer, then continues up the superclass chain. "Instance methods" are defined by the class, and "class methods" are defined by the metaclass plus the root (non-meta) class.

In proper computer science language theory, a class and metaclass hierarchy can be more free-form, with deeper metaclass chains and multiple classes instantiated from any single metaclass. Objective-C uses metaclasses for practical goals like class methods, but otherwise tends to hide metaclasses. For example, [NSObject class] is identical to [NSObject self], even though in formal terms it ought to return the metaclass that NSObject->isa points to. The Objective-C language is a set of practical compromises; here it limits the class schema before it gets too, well, meta.

翻译:
Objective-C 是基于类的对象系统。每一个对象都是一些类的实例；这个对象的 isa 指针指向它所属的类。该类描述这个对象的数据信息：内存分配大小和实例变量的类型与布局。这个类也描述了对象的行为：它能够响应的选择器和它实现的实例方法。

类的方法列表是一个实例方法的集合，是对象响应的选择器。当你向一个实例发送一条消息，objc_msgSend()查询对象所属类（和它的父类，如果有）的方法列表，去决定调用哪个方法。

每个 Objective-C 类也是一个对象。它有一个 isa 指针和其他数据，并且可以响应选择器。当你调用一个“类方法”，例如：[NSObject alloc]，你实际上是向类对象发送了一条消息。

因为一个类是一个对象，它肯定也是其他类的实例，这个类是元类(metaclass)。元类是关于类对象的描述，就像类是普通实例对象的描述一样。实际上，元类的方法列表正是类方法：该类对象响应的选择器。当你向一个类（元类的实例）发送消息，objc_msgSend()查询元类（和它的父类，如果有）的方法列表，去决定调用哪个方法。元类为类对象描述类方法，就像类为实例对象描述实例方法一样。

元类？元类链是一直向下的吗？不是，一个元类是根元类的实例；根元类是它自身的实例。isa 指针链以一个环结束：实例指向类指向根元类指向根元类自身。元类的 isa 指针并不重要，因为在现实世界中，没人会向元类对象发送消息。

元类的父类就要更重要了。元类的父类链平行于类的父类链，因此类方法跟实例方法一样被继承。并且根元类的父类是根类，因此每个类对象都响应根类的实例方法。最后，一个类对象就像其他对象一样是根类的实例（或子类）。

晕了吧？这个图肯定会帮助你。记住，当一个消息被发送到任何对象，都会从对象的 isa 指针开始寻找应该调用的方法，然后继续沿着父类链向上寻找。“实例方法”被类定义，“类方法”被元类和根类（非元类）定义。

![](http://upload-images.jianshu.io/upload_images/671285-14497a391b617e99.jpg)

觉得翻译的很好,就不自己翻译了...

### Q&A 以下代码输出什么?

    NSLog(@"%@", NSStringFromClass([self class]));
    NSLog(@"%@", NSStringFromClass([super class]));

答案：不论输出什么输出相同

解释：objc中super是编译器标示符，并不像self一样是一个对象，遇到向super发的方法时会转译成objc_msgSendSuper(id self,SEL _cmd,...)，而参数中的对象还是self，于是从父类开始沿继承链寻找- class这个方法，最后在NSObject中找到（若无override），此时，[self class]和[super class]已经等价了  (答案来自我就叫sunny怎么了)


