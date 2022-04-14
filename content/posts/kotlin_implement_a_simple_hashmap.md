---
title: "Kotlin手动实现一个最简单的哈希表"
date: 2021-10-16T17:43:32+08:00
draft: false
tags: ["Kotlin", "HashMap"]
categories: ["算法笔记"]
---

参考的是《数据结构(C语言版)》上256页左右的哈希表的介绍，用了最简单的直接寻址法 + 链地址法。

用的是Kotlin。
```kotlin
package main.kotlin

/**
 * 手动实现简单的hash表
 * 简单的数组 +链表 （无红黑树）
 * 要求哈希函数可配置（被自我否决，太复杂了啦），这次就先做比较简单的 直接定址法 + 链地址法
 *
 * @Date 2021-10-16.
 * @author Johnathan Lin
 */

data class Node(
  val key: Int, //key
  var value: Int, //value
  var next: Node? //如果hash值重复了，则用头插法放进去
)

fun main() {
  // hash表，这次可为空
  val size = 100
  val hashArr: Array<Node?> = Array(size) { null }

  //插入 假设插入key 8 value 24
  println("插入key 8 value 24")
  set(hashArr, size, 8, 24) { k, s -> k % s }
  println("插入key 108 value 32")
  set(hashArr, size, 108, 32) { k, s -> k % s }
  var v = get(hashArr, size, 108) { k, s -> k % s }
  println("读取key为108: $v")
  println("删除key 108")
  remove(hashArr, size, 108) { k, s -> k % s }
  v = get(hashArr, size, 108) { k, s -> k % s }
  println("读取key为108: $v")
  v = get(hashArr, size, 8) { k, s -> k % s }
  println("读取key为8: $v")

}

/**
 * @param hashFunc 哈希函数 param1：key param2：size
 */
fun get(hashArr: Array<Node?>, size: Int, key: Int, hashFunc: (Int, Int) -> Int): Node? {
  val pos = hashFunc.invoke(key, size)
  val queueHead = hashArr[pos]
  if (queueHead == null) {
    return null
  } else {
    if (queueHead.key == key) {
      return queueHead
    }
    var p = queueHead
    while(p?.next != null) {
      if (p.next?.key == key) {
        return p.next
      }
      p = p.next
    }
  }
  return null
}

fun set(hashArr: Array<Node?>, size: Int, key: Int, value: Int, hashFunc: (Int, Int) -> (Int)) {
  //get 不到的时候才会set
  val node = get(hashArr, size, key, hashFunc)
  if (node != null) {
    node.value = value
  } else {
    val pos = hashFunc.invoke(key, size)
    val newNode = Node(key, value, null)
    if (hashArr[pos] == null) {
      hashArr[pos] = newNode
    } else {
      val next = hashArr[pos]?.next
      if (next == null) {
        hashArr[pos]?.next = newNode
      } else {
        newNode.next = next
        hashArr[pos]?.next = newNode
      }
    }
  }

}

fun remove(hashArr: Array<Node?>, size: Int, key: Int, hashFunc: (Int, Int) -> (Int)) {
  val pos = hashFunc.invoke(key, size)
  val queueHead = hashArr[pos]
  if (queueHead == null) {
    return
  } else {
    if (queueHead.key == key) {
      if (queueHead.next != null) { //好像jdk7有一个hashMap的bug？
        hashArr[pos] = queueHead.next
        return
      } else {
        hashArr[pos] = null
      }
    } else {
      var p = queueHead
      while(p?.next != null) {
        if (p.next?.key == key) {
          if (p.next?.next != null) {
            p.next = p.next?.next
          } else {
            p.next = null
          }
          return
        }
        p = p.next
      }
    }
  }
}

```

然后是输出：

``` plain
插入key 8 value 24
插入key 108 value 32
读取key为108: Node(key=108, value=32, next=null)
删除key 108
读取key为108: null
读取key为8: Node(key=8, value=24, next=null)

Process finished with exit code 0

```