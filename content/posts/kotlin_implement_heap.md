---
title: "Kotlin实现二叉堆、大顶堆、优先级队列"
date: 2021-10-14T23:30:30+08:00
draft: false
tags: ["Python"]
categories: ["算法笔记"]
---

参考了

[https://www.bilibili.com/video/BV11t4y1r79L](https://www.bilibili.com/video/BV11t4y1r79L)

[https://blog.csdn.net/qq_19782019/article/details/78301832](https://blog.csdn.net/qq_19782019/article/details/78301832)



他们已经写的足够好了。我最近都在用Kotlin编程开发，我尝试用Kotlin实现了大顶堆，并且作为手动实现的优先级队列，通过了[Leetcode 347](https://leetcode-cn.com/problems/top-k-frequent-elements/)。

```kotlin
package main.kotlin

/**
 * 二叉堆（大顶堆、优先级队列）
 *
 * @Date 2021-10-14.
 * @author Johnathan Lin
 */
data class MaxHeap<T : Comparable<T>>(
  val arr: Array<T>,
  var size: Int
) {
  //将无序的数组构建一个二叉堆
  fun makeHeap() {
    for (i in (size - 1) downTo 0) {
      heapDown(i)
    }
  }

  //向二叉堆中加入元素
  fun addItem(value: T) {
    //TODO 添加的边界还未考虑
    val newIndex = size++
    arr[newIndex] = value
    heapUp(newIndex)
  }

  //移除堆顶元素
  fun removeItem(): T {
    //TODO 删除的边界还未考虑
    val removeValue = arr[0]
    val lastValue = arr[size - 1]
//    println("lastValue:$lastValue")
    arr[0] = lastValue
    size--
    heapDown(0)

    return removeValue
  }

  fun printHeap() {
    for (i in 0..(size - 1)) {
      print(arr[i].toString() + " ")
    }
    println()
  }

  private fun heapDown(p: Int) {
    var i = p
    while (i < size) {
      //和左右子节点（如果存在）比大小，如果自己是最大则结束，如果不是，则将自己这个位置换成最大的，然后再从被换的位置继续比
      val left = 2 * i + 1
      val right = 2 * i + 2
      var maxinum = i
      if (left < size && arr[maxinum] < arr[left]) {
        maxinum = left
      }
      if (right < size && arr[maxinum] < arr[right]) {
        maxinum = right
      }

      if (maxinum == i) {
        break
      }
//      println("交换i: $i ${arr[i]}, maxinum:$maxinum ${arr[maxinum]}")
      val tmp = arr[i]
      arr[i] = arr[maxinum]
      arr[maxinum] = tmp
      i = maxinum
    }
  }

  private fun heapUp(p: Int) {
    //从下往上不停地比较其父节点，比父节点大则交换
    var i = p
    while (i >= 0 && arr[i] > arr[(i - 1) / 2]) {
      val parentIndex = (i - 1) / 2
      val tmp = arr[i]
      arr[i] = arr[parentIndex]
      arr[parentIndex] = tmp
      i = parentIndex
    }
  }
}

//自己定义的类型，实现Comparable接口
data class Data(
  val name: String, //名字
  val weight: Int //权重
) : Comparable<Data> {
  override fun compareTo(other: Data): Int {
    return if (weight > other.weight) {
      1
    } else if (weight < other.weight) {
      -1
    } else {
      0
    }
  }

  override fun toString(): String {
    return "${name}-${weight}"
  }

}

fun main() {
  //TODO 这里由于Kotlin指定泛型Data为非空，所以要求对每个元素初始化，这步很浪费空间
  //TODO 如果改为Array<Data?> 则很多地方都要判断是否为空
  val arr: Array<Data> = Array(100) { i -> Data("null", i) }
  //测试数据
  arr[0] = Data("Kobe", 5)
  arr[1] = Data("James", 4)
  arr[2] = Data("Duncan", 6)
  arr[3] = Data("Garnett", 8)
  arr[4] = Data("Wall", 2)
  arr[5] = Data("Nash", 10)
  arr[6] = Data("Howard", 12)
  arr[7] = Data("Paul", 3)
  arr[8] = Data("Anthony", 7)
  arr[9] = Data("Yi", 11)
  println("构建二叉堆")
  val heap: MaxHeap<Data> = MaxHeap(arr, 10)
  //将无序的数组构建一个二叉堆
  heap.makeHeap()
  heap.printHeap()

  //加入一个元素，预计这个元素会到堆顶
  val added = Data("Gasol", 16)
  println("添加一个元素${added.toString()}")
  heap.addItem(added)
  heap.printHeap()

  val removed = heap.removeItem()
  println("移除了顶部元素${removed.toString()}")
  heap.printHeap()
  println("finish")

}
```

输出结果如下：
```
构建二叉堆
Howard-12 Yi-11 Nash-10 Garnett-8 James-4 Kobe-5 Duncan-6 Paul-3 Anthony-7 Wall-2 
添加一个元素Gasol-16
Gasol-16 Howard-12 Nash-10 Garnett-8 Yi-11 Kobe-5 Duncan-6 Paul-3 Anthony-7 Wall-2 James-4 
移除了顶部元素Gasol-16
Howard-12 Yi-11 Nash-10 Garnett-8 James-4 Kobe-5 Duncan-6 Paul-3 Anthony-7 Wall-2 
finish

Process finished with exit code 0

```




然后我将其进行修改，提交到LeetCode，通过，代码如下：

``` kotlin 
data class MaxHeap<T:Comparable<T>>(
  val arr:Array<T>,
  var size: Int
) {

  fun makeHeap() {
    for(i in (size - 1) downTo 0) {
      heapDown( i)
    }
  }

  fun heapDown(p: Int) {
    var i = p
    while(i < size) {
    //   if(i == 0) {
    //     println("i: $i, arr[i] = ${arr[i]}")
    //   }
      val left = 2 * i + 1
      val right = 2 * i + 2
      var maxinum = i
      if (left < size && arr[maxinum] < arr[left]) {
        maxinum = left
      }
      if (right < size && arr[maxinum] < arr[right]) {
        maxinum = right
      }

      if (maxinum == i) {
        break
      }
    //   println("交换i: $i ${arr[i]}, maxinum:$maxinum ${arr[maxinum]}")
      val tmp = arr[i]
      arr[i] = arr[maxinum]
      arr[maxinum] = tmp
      i = maxinum
    }
  }

  fun heapUp(p: Int) {
    var i = p
    while (i >= 0 && arr[i] > arr[(i-1)/2]) {
      val parentIndex = (i-1)/2
      val tmp = arr[i]
      arr[i] = arr[parentIndex]
      arr[parentIndex] = tmp
      i = parentIndex
    }
  }

  fun addItem(value: T) {
    val newIndex = size++
    arr[newIndex] = value
    heapUp(newIndex)
  }

  fun removeItem():T {
    val removeValue = arr[0]
    val lastValue = arr[size-1]
//    println("lastValue:$lastValue")
    arr[0] = lastValue
    size--
    heapDown(0)
    
    return removeValue
  }
}

data class Data(
  val value: Int,
  val num: Int
): Comparable<Data> {
  override fun compareTo(other: Data): Int {
    return if (num > other.num) {
      1
    } else if (num < other.num) {
      -1
    } else {
      0
    }
  }

  override fun toString(): String {
    return "${value}-${num}"
  }

}
class Solution {


    fun topKFrequent(nums: IntArray, k: Int): IntArray {
        val res = IntArray(k);
        val arr:Array<Data> = Array(nums.size)  {i -> Data(0, 0)}
        //  val arr: Array<Data> = Array(nums.size) { i -> Data(0, 0) }
        val maxHeap = MaxHeap(arr, 0)
        val map: HashMap<Int, Int> = hashMapOf()
        nums.forEach { it ->
            map[it] = map.getOrDefault(it, 0) + 1
        }
        map.forEach { key, value ->
            maxHeap.addItem(Data(key, value))
        }
        for(i in 1..k) {
            res[i-1] = maxHeap.removeItem().value
        }
        return res
    }
}
```