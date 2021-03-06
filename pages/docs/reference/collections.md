---
type: doc
layout: reference
category: Other
title: "集合：List、Set、Map"
---

# 集合：List、Set、Map

与大多数语言不同，Kotlin 区分可变集合与不可变集合（lists、sets、maps 等）。精确控制什么时候集合可编辑有助于消除 bug 以及设计良好的 API。

预先了解一个可变集合的只读 _视图_ 与一个真正的不可变集合之间的区别是很重要的。它们都容易创建，但类型系统不能表达它们的差别，所以由你来跟踪（是否相关）。

Kotlin 的 `List<out T>` 类型是一个提供只读操作如 `size`、`get`等的接口。与 Java 类似，它继承自 `Collection<T>` 进而继承自 `Iterable<T>`。改变 list 的方法是由 `MutableList<T>` 加入的。这一模式同样适用于 `Set<out T>/MutableSet<T>` 及 `Map<K, out V>/MutableMap<K, V>`。

我们可以看下 list 及 set 类型的基本用法：

<div class="sample" markdown="1" theme="idea" data-highlight-only>

```kotlin
val numbers: MutableList<Int> = mutableListOf(1, 2, 3)
val readOnlyView: List<Int> = numbers
println(numbers)        // 输出 "[1, 2, 3]"
numbers.add(4)
println(readOnlyView)   // 输出 "[1, 2, 3, 4]"
readOnlyView.clear()    // -> 不能编译

val strings = hashSetOf("a", "b", "c", "c")
assert(strings.size == 3)
```

</div>

Kotlin 没有专门的语法结构创建 list 或 set。 要用标准库的方法，如
`listOf()`、 `mutableListOf()`、 `setOf()`、 `mutableSetOf()`。
在非性能关键代码中创建 map 可以用一个简单的[惯用法](idioms.html#只读-map)来完成：`mapOf(a to b, c to d)`。

注意上面的 `readOnlyView` 变量（译者注：与对应可变集合变量 `numbers`）指向相同的底层 list 并会随之改变。 如果一个 list 只存在只读引用，我们可以考虑该集合完全不可变。创建一个这样的集合的一个简单方式如下：

<div class="sample" markdown="1" theme="idea" data-highlight-only>

```kotlin
val items = listOf(1, 2, 3)
```

</div>

目前 `listOf` 方法是使用 array list 实现的，但是未来可以利用它们知道自己不能变的事实，返回更节约内存的完全不可变的集合类型。

注意这些类型是[协变的](generics.html#型变)。这意味着，你可以把一个 `List<Rectangle>` 赋值给 `List<Shape>` 假定 `Rectangle` 继承自 `Shape`（集合类型与元素类型具有相同的继承关系）。对于可变集合类型这是不允许的，因为这将导致运行时故障：你可能向 `List<Shape>` 中添加一个 `Circle`，而在程序的其他地方创建了一个其中含有 `Circle` 的 List<Rectangle>`。

有时你想给调用者返回一个集合在某个特定时间的一个快照, 一个保证不会变的：

<div class="sample" markdown="1" theme="idea" data-highlight-only>

```kotlin
class Controller {
    private val _items = mutableListOf<String>()
    val items: List<String> get() = _items.toList()
}
```

</div>

这个 `toList` 扩展方法只是复制列表项，因此返回的 list 保证永远不会改变。

List 与 set 有很多有用的扩展方法值得熟悉：

<div class="sample" markdown="1" theme="idea" data-highlight-only>

```kotlin
val items = listOf(1, 2, 3, 4)
items.first() == 1
items.last() == 4
items.filter { it % 2 == 0 }   // 返回 [2, 4]

val rwList = mutableListOf(1, 2, 3)
rwList.requireNoNulls()        // 返回 [1, 2, 3]
if (rwList.none { it > 6 }) println("No items above 6")  // 输出“No items above 6”
val item = rwList.firstOrNull()
```

</div>

…… 以及所有你所期望的实用工具，例如 sort、zip、fold、reduce 等等。

非常值得注意的是，对只读集合返回修改后的集合的操作（例如 `+`、 `filter`、 `drop` 等）并不会以原子方式创建其结果，因此如果没有合适的同步机制，在不同的线程中使用其结果是不安全的。

Map 遵循同样模式。它们可以容易地实例化与访问，像这样：

<div class="sample" markdown="1" theme="idea" data-highlight-only>

```kotlin
val readWriteMap = hashMapOf("foo" to 1, "bar" to 2)
println(readWriteMap["foo"])  // 输出“1”
val snapshot: Map<String, Int> = HashMap(readWriteMap)
```

</div>
