---
title: 从Java的Type到Spring的ResolvableType
categories:
  - Java开发框架
tags:
  - Spring
  - Java开发框架
  - JavaSE
toc: true
date: 2020-08-10 11:39:45
---

## 从Java的Type到Spring的ResolvableType

-------------

### Type

Type是Java编程语言中所有类型的公共高级接口。

![Type家族](/Type家族.png)

Type体系包括以下: 
  1. 原始类型(Class)
  2. 参数化类型(ParameterizedType)
  3. 数组类型(GenericArrayType)
  4. 类型变量(TypeVariable)
  5. 基本类型(Class)

#### ParameterizedType 参数化类型

ParameterizedType表示参数化类型，比如Collection&lt;String&gt;
但不限于集合， 还包括其他平常用到的范型类，比如Class&lt;?&gt;

```java
public interface ParameterizedType extends Type {
    /**
     * 获取<>中的实际类型
     */
    Type[] getActualTypeArguments();
    /**
     * 获取<>前的类型
     */
    Type getRawType();
    /**
     * 如果一个类型是内部类，则返回这个类型的所属者，否则返回null
     */
    Type getOwnerType();
}
```

#### GenericArrayType 数组类型

GenericArrayType表示其组件类型为参数化类型或类型变量的数组类型。比如`T[]`,`List<T>[]`
它表示的是范型数组，而不是普通的String[]这样的数组

```java
public interface GenericArrayType extends Type {
    // 获得这个数组的元素类型，比如T[]  则获得T的type
    Type getGenericComponentType();
}
```

#### TypeVariable 类型变量

TypeVariable是用于类型变量的通用超接口。
它代表范型类型中的变量，即T、K、V

```java
public interface TypeVariable<D extends GenericDeclaration> extends Type, AnnotatedElement {
    /**
     * 获取范型的上限，比如List<T extends Exception>,则返回Exception； 默认是Object
    */
    Type[] getBounds();

    /**
     * 获取范型左边的实体，比如List<T>中的List
     */
    D getGenericDeclaration();

    /**
     * 返回在源码中的名称，比如T、K
     */
    String getName();

    /**
     * 获取范型的上限，不同于getBounds()的是，它可以获取到范型上界上添加的注解
     */
     AnnotatedType[] getAnnotatedBounds();
}
```

#### Class 原始类型、基本类型

与上面三个不同的是，Class是Type的一个实现类，属于原始类型，是Java反射的基础。
在程序运行期间，每一个类都对应一个Class对象，这个对象包含了类基本信息。

### ResolvableType

ResolvableType是Spring对Type家族的一个封装，用于简化范型的处理

ResolvableType的所有构造方法都是私有化的， 但是它提供了静态方法来构造。

![静态方法](/静态方法.png)

#### forClass系列方法

直接封装指定的类型

##### forRawClass(Class<?> clazz)

调用构造方法，并重写了三个方法

```java
public static ResolvableType forRawClass(@Nullable Class<?> clazz) {
    return new ResolvableType(clazz) {
        @Override
        public ResolvableType[] getGenerics() {
            return EMPTY_TYPES_ARRAY;
        }
        @Override
        public boolean isAssignableFrom(Class<?> other) {
            return (clazz == null || ClassUtils.isAssignable(clazz, other));
        }
        @Override
        public boolean isAssignableFrom(ResolvableType other) {
            Class<?> otherClass = other.getRawClass();
            return (otherClass != null && (clazz == null || ClassUtils.isAssignable(clazz, otherClass)));
        }
    };
}
```

##### forClass(Class<?> clazz)

直接调用构造方法

```java
public static ResolvableType forClass(@Nullable Class<?> clazz) {
    return new ResolvableType(clazz);
}
```

##### forClass(Class<?> baseType, Class<?> implementationClass)

传入基础类和实现类

```java
public static ResolvableType forClass(Class<?> baseType, Class<?> implementationClass) {
    Assert.notNull(baseType, "Base type must not be null");
    ResolvableType asType = forType(implementationClass).as(baseType);
    // 如果实现类是空的，那么返回基础类
    return (asType == NONE ? forType(baseType) : asType);
}
```

#### forMethod系列方法

获取方法上的参数或返回值的类型信息

##### forMethodParameter(Method method, int parameterIndex)

传入方法， index=-1时返回方法返回值的类型， 1返回方法的第一个参数的类型，2返回第二个，以此类推

```java
public static ResolvableType forMethodParameter(Method method, int parameterIndex) {
    Assert.notNull(method, "Method must not be null");
    return forMethodParameter(new MethodParameter(method, parameterIndex));
}
```

##### forMethodReturnType(Method method)

将方法的参数索引设为-1

```java
public static ResolvableType forMethodReturnType(Method method) {
    Assert.notNull(method, "Method must not be null");
    return forMethodParameter(new MethodParameter(method, -1));
}
```

#### forField系列

用于处理字段的类型

##### forType(Type type, TypeProvider typeProvider, VariableResolver variableResolver)

系列方法会调用到这个核心方法

```java
static ResolvableType forType(@Nullable Type type, @Nullable TypeProvider typeProvider, @Nullable VariableResolver variableResolver) {
    // 对TypeProvider进一步进行封装，为了得到一个可以序列化的TypeProvider
    if (type == null && typeProvider != null) {
        type = SerializableTypeWrapper.forTypeProvider(typeProvider);
    }
    if (type == null) {
        return NONE;
    }
    if (type instanceof Class) {
        return new ResolvableType(type, typeProvider, variableResolver, (ResolvableType) null);
    }

    // 缓存相关
    cache.purgeUnreferencedEntries();
    ResolvableType key = new ResolvableType(type, typeProvider, variableResolver);
    ResolvableType resolvableType = cache.get(key);
    if (resolvableType == null) {
        resolvableType = new ResolvableType(type, typeProvider, variableResolver, key.hash);
        cache.put(resolvableType, resolvableType);
    }
    return resolvableType;
}
```

### 总结

**Spring提供的ResolvableType是对Java原生Type家族的一个封装，它包含了类、方法、字段、构造函数，当我们需要处理它们时，只需要调用响应的方法就能获取它们的ResolvableType，ResolvableType就包含了这个对象的所有类型信息**






