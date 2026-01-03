---
layout: post
title: "Compose: `by` mutableStateOf"
date: 2026-01-03 12:00:00 +0800
categories: [Android, Compose, Kotlin]
tags: [compose, state, mutableStateOf, by, delegate]
---

以下是 compose 中，State 的常规写法：

```kotlin
class MainActivity : ComponentActivity() {
    var name by mutableStateOf("弘征")
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        enableEdgeToEdge()
        setContent {
            LearnComposeTheme {
                Box(Modifier.safeContentPadding()) {
                    Text(name)
                }
            }
        }

        lifecycleScope.launch {
            delay(3000)
            name = "段成睿"
        }
    }
}
```

## state 的代理写法

其中，我们使用了 `var name by mutableStateOf("弘征")` 来定义状态值，使用 `Text(name)` 来读取状态值，使用 `name = "段成睿"` 来更新状态值。

这实际上使用了 kotlin 的 `by` 关键字来实现了对象代理，在介绍 kotlin 中的对象代理之前，我们先看下状态的另一种写法。

## State 的常规写法

如果不使用 kotlin 的 by 关键字来做对象代理，应该怎么写呢？

```kotlin
class MainActivity : ComponentActivity() {
    val name = mutableStateOf("弘征")
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        enableEdgeToEdge()
        setContent {
            LearnComposeTheme {
                Box(Modifier.safeContentPadding()) {
                    Text(name.value)
                }
            }
        }

        lifecycleScope.launch {
            delay(3000)
            name.value = "段成睿"
        }
    }
}
```

有三处改动点：
1. name 的定义从 `var` 替换为 `val`，因为实际改变的值是 `name.value`，而 `name` 本身是不可变的
2. 读取值需要使用 `name.value`
3. 写入值需要使用 `name.value = "段成睿"`


## kotlin 中 `by` 的介绍

kotlin 中，`by` 关键字代理的属性称为代理属性([Delegated properties﻿](https://kotlinlang.org/docs/delegated-properties.html))

有些针对属性的通用能力，我们是不想每次都写一遍的，例如：懒加载的属性。

语法为：`val/var <property name>: <Type> by <expression>`。其含义通俗一点说，就是将这个属性的「读」、「写」代理给一个类。

具体来说，当需要读取这个属性的值的时候，会调用到这个代理对象的 `getValue()` 方法，当需要写入这个属性的值的时候，会调用到这个代理对象的 `setValue()` 方法。

代理类的 `getValue()` 和 `setValue()` 的方法签名是严格定义的：

```kotlin
import kotlin.reflect.KProperty

class Delegate {
    operator fun getValue(thisRef: Any?, property: KProperty<*>): String {
        return "$thisRef, thank you for delegating '${property.name}' to me!"
    }

    operator fun setValue(thisRef: Any?, property: KProperty<*>, value: String) {
        println("$value has been assigned to '${property.name}' in $thisRef.")
    }
}
```

除了直接定义代理类，也可以通过扩展方法来实现代理，例如 compose 中的 state：

```kotlin
@Suppress("NOTHING_TO_INLINE")
public inline operator fun <T> State<T>.getValue(thisObj: Any?, property: KProperty<*>): T = value

@Suppress("NOTHING_TO_INLINE")
public inline operator fun <T> MutableState<T>.setValue(
    thisObj: Any?,
    property: KProperty<*>,
    value: T,
) {
    this.value = value
}
```

## compose 中的 state 不能使用 by 的场景

下面代码，就会导致界面不能刷新

```kotlin
class MainActivity : ComponentActivity() {
    var name by mutableStateOf("弘征")
    var name1 = name
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        enableEdgeToEdge()
        setContent {
            LearnComposeTheme {
                Box(Modifier.safeContentPadding()) {
                    Text(name1)
                }
            }
        }

        lifecycleScope.launch {
            delay(3000)
            name = "段成睿"
        }
    }
}
```

这么改一下就可以了：

```kotlin
class MainActivity : ComponentActivity() {
    var name = mutableStateOf("弘征")
    var name1 = name
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        enableEdgeToEdge()
        setContent {
            LearnComposeTheme {
                Box(Modifier.safeContentPadding()) {
                    Text(name1.value)
                }
            }
        }

        lifecycleScope.launch {
            delay(3000)
            name.value = "段成睿"
        }
    }
}
```
