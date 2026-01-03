---
layout: post
title: "Compose: remember 和 LaunchedEffect"
date: 2026-01-03 14:00:00 +0800
categories: [Android, Compose, Kotlin]
tags: [remember, LaunchedEffect, state, coroutine]
---

## 解决 @Composable 中定义 state 和启动协程的问题

```kotlin
var name by remember { mutableStateOf("弘征") }

Text(name)
```

下面是一个简单的 compose 例子：
1. 我们将 `name` 定义在 activity 层面，并没有定义在 `@Composable` 函数当中
2. 启动协程使用的是 `lifecycleScope.launch`

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

Composable 函数可能在任何时候执行，所以，自然而然的就会有另外的问题：
1. 如果在 `@Composable` 函数中如果定义 state应该怎么写？
2. 如果在 `@Composable` 函数中如果启动协程应该怎么写？

因为如果不做特殊处理，那么每次 recomposable，state 都会初始化，协程也会每次都重新启动，这个是不可控的。

答案很简单：
1. state 使用 `remember` 包一层，保证只初始化一次
2. 协程使用 `LaunchedEffect(Unit)` 代替，保证协程只初始化一次

```kotlin
Box(Modifier.safeContentPadding()) {
    var name by remember { mutableStateOf("段成睿") }
    Text(name)
    LaunchedEffect(Unit) {
        delay(3000)
        name = "弘征"
    }
}
```

## 避免重复计算

```kotlin
@Composable
fun CharCounter(value: String) {
    // 假设这里是一个耗时操作，例如网络请求
    val length = remember { value.length }
    Text("字符串的长度是：${length})
}
```

remember 可以传一个参数，当参数变的时候，会重复执行 lambda 里的计算代码

```kotlin
@Composable
fun CharCounter(value: String) {
    // 假设这里是一个耗时操作，例如网络请求
    val length = remember(value) { value.length }
    Text("字符串的长度是：${length})
}
```

这样，当传入不同的 value 时，会重新执行 `value.length`
