---
title: "Java TreeSet的一些用法和特性"
date: 2022-06-18T17:10:44+08:00
draft: false
categories: ["技术文章"]
---

## 先看一个例子（kotlin实现）
``` kotlin
import java.util.TreeSet

/**
 * 定义一个用于测试TreeSet集合的结构
 * 用TreeSet进行排名
 * id: 玩家id
 * score: 玩家得分
 */
data class PlayerScore(var id: Int, var score: Int): Comparable<PlayerScore> {
    override fun compareTo(other: PlayerScore): Int {
        return if (score > other.score) {
            1
        } else if (score < other.score) {
            -1
        } else {
            0
        }
    }

}


fun main() {
    //创建一个TreeSet
    val treeSet: TreeSet<PlayerScore> = TreeSet()
    //创建3个PlayerScore，其中对id为101的对象存一个引用
    //将3个PlayerScore装入set中
    val id101Obj = PlayerScore(101,100)
    //按不同的顺序加入TreeSet
    treeSet.add(PlayerScore(102,200))
    treeSet.add(id101Obj)
    treeSet.add(PlayerScore(103,300))

    println("按不同的顺序加入TreeSet，按score从小到大输出:")
    showTreeSet(treeSet)

    println("再加入重复id，但分数不同的对象，按score从小到大输出:")
    treeSet.add(PlayerScore(101,500))
    showTreeSet(treeSet)

    println("直接修改id为101的对象的分数为400，按score从小到大输出:")
    id101Obj.score = 400
    showTreeSet(treeSet)

}

private fun showTreeSet(treeSet: TreeSet<PlayerScore>) {
    treeSet.forEachIndexed { index, s ->
        println("排名:${index+1} 分数:${s.score} id:${s.id}")
    }
    println()
}
```
运行结果：
```
按不同的顺序加入TreeSet，按score从小到大输出:
排名:1 分数:100 id:101
排名:2 分数:200 id:102
排名:3 分数:300 id:103

再加入重复id，但分数不同的对象，按score从小到大输出:
排名:1 分数:100 id:101
排名:2 分数:200 id:102
排名:3 分数:300 id:103
排名:4 分数:500 id:101

直接修改id为101的对象的分数为400，按score从小到大输出:
排名:1 分数:400 id:101
排名:2 分数:200 id:102
排名:3 分数:300 id:103
排名:4 分数:500 id:101

```
## 分析
首先，`PlayerScore`继承`Comparable`接口，实现了从小到大排序。在最开始只有3条数据时，按照不同的顺序插入`TreeSet`，可以正常的完成排序。

之后玩家101的分数发生改变，需要重新给玩家进行排名，我们分别通过两种方式进行操作：
1. 通过`id101Obj`的引用来对这个对象的分数进行修改；
2. 通过再往TreeSet中加入一个id为101，分数不同于之前的分数的对象。

结果：
第一种方式：玩家的积分被修改了，但是排名依旧为第1；
第二种方式：虽然积分正常排序，但是有两个id为101的对象。

所以如果我们的玩家积分发生改变的时候，要如何才能正常排序呢？

可以对某个对象先从TreeSet中删除，再往TreeSet中插入。这是因为TreeSet的底层是红黑树，在删除和插入的时候，会对树结构进行调整。
代码如下：
``` kotlin 
    //创建一个TreeSet
    val treeSet: TreeSet<PlayerScore> = TreeSet()
    //创建3个PlayerScore，其中对id为101的对象存一个引用
    //将3个PlayerScore装入set中
    val id101Obj = PlayerScore(101,100)
    //按不同的顺序加入TreeSet
    treeSet.add(PlayerScore(102,200))
    treeSet.add(id101Obj)
    treeSet.add(PlayerScore(103,300))

    println("按不同的顺序加入TreeSet，按score从小到大输出:")
    showTreeSet(treeSet)

    treeSet.remove(id101Obj)
    id101Obj.score = 400
    treeSet.add(id101Obj)
    println("先将id101Obj从TreeSet中删除，修改值，然后再插入TreeSet中，按score从小到大输出:")
    showTreeSet(treeSet)
```

结果如下：
```
按不同的顺序加入TreeSet，按score从小到大输出:
排名:1 分数:100 id:101
排名:2 分数:200 id:102
排名:3 分数:300 id:103

先将id101Obj从TreeSet中删除，修改值，然后再插入TreeSet中，按score从小到大输出:
排名:1 分数:200 id:102
排名:2 分数:300 id:103
排名:3 分数:400 id:101
```

## 更进一步
先删后加的方式是可以完成排序，但是TreeSet本身是线程不安全的。如果有多个线程都要更新玩家积分和排名，那么大概率会存在问题。
TreeSet虽然可以边插边排序，但是它只允许一次插入，插入之后积分的修改并不会修改排序。

对于排行榜上的排名经常变动的情况，用TreeSet是不太合理的。

## 另一种排行榜实现
本来选用TreeSet，是因为TreeSet拥有“边插入边排序”的特点，但在排行榜需要更新时并不方便。
那么换一种思路，如果只用变长的ArrayList来存储玩家积分对象，同样是在插入/更新/删除时进行排序？

先插入/更新/删除，对ArrayList内的数据进行修改，然后进行排序。