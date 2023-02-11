---
title: Category
date: 2020-06-18 15:14
---


## Category的实现原理

通过runtime动态将分类的方法合并到类对象，元类中
```objective-c
struct _category_t {
    const char *name;
    struct _class_t *cls;
    const struct _method_list_t *instance_methods;
    const struct _protocol_list_t *protocols;
    const struct _prop_list_t *properties;
}
```



## Category 和 Extension的却别是什么

## Category中有load方法吗？ load方法是什么时候调用？ load方法能集成吗？

## load、initialize方法的区别是什么？ 他们再 category 中的调用顺序？以及出现继承是他们之间的调用过程？

## Category 能否添加成员变量？ 如果可以，如何给 Category添加成员变量