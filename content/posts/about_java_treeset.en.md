---
title: "Some Usage and features on Java TreeSet"
date: 2022-06-18T17:10:44+08:00
draft: false
categories: ["Tech"]
---

## An Example (kotlin)
``` kotlin
import java.util.TreeSet

/**
 * Define a data class to test TreeSet Collection
 * Sort by TreeSet
 * id: player id
 * score: player score
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
    //create a TreeSet
    val treeSet: TreeSet<PlayerScore> = TreeSet()
    //create 3 PlayerScore object，and use 'id101Obj' reference to the first obj
    //put these objects into the treeset
    val id101Obj = PlayerScore(101,100)
    //put in different order 
    treeSet.add(PlayerScore(102,200))
    treeSet.add(id101Obj)
    treeSet.add(PlayerScore(103,300))

    println("Add the objects in a different order and print out by score from smallest to largest:")
    showTreeSet(treeSet)

    println("Add an object with duplicated id but different score, and print out by score from smallest to largest:")
    treeSet.add(PlayerScore(101,500))
    showTreeSet(treeSet)

    println("Change the score of the object whose id is 101 to 400， and print out by score from smallest to largest:")
    id101Obj.score = 400
    showTreeSet(treeSet)

}

private fun showTreeSet(treeSet: TreeSet<PlayerScore>) {
    treeSet.forEachIndexed { index, s ->
        println("rank:${index+1} score:${s.score} id:${s.id}")
    }
    println()
}
```
Console output:
```
Add the objects in a different order and print out by score from smallest to largest:
rank:1 score:100 id:101
rank:2 score:200 id:102
rank:3 score:300 id:103

Add an object with duplicated id but different score, and print out by score from smallest to largest:
rank:1 score:100 id:101
rank:2 score:200 id:102
rank:3 score:300 id:103
rank:4 score:500 id:101

Change the score of the object whose id is 101 to 400， and print out by score from smallest to largest:
rank:1 score:400 id:101
rank:2 score:200 id:102
rank:3 score:300 id:103
rank:4 score:500 id:101

```

## Analysis

First, `PlayerScore` implements `Comparable` interface , implements the function of sorting from smallest to largest. If there are only three data pieces at the beginning, insert them in different order, treeset does sort normally.

Then, when player 101 score was changed, we need to resort. We tried 2 ways to resort.
1. Change `id101Obj`'s score by reference.
2. Add a new obj which id is 101 and has a new score into treeset.

Result:
1.  `id101Obj`'s score was changed, but its rank is still 1.
2. Although the score is sorted, it has two objects which id is 101.

So how to resort treeset when player's score was changed?

One solution is that you can remove the object from treeset, and change its score, then add it to treeset. This is because the underlying TreeSet is a red-black tree, and the tree structure is adjusted when it is deleted and inserted.

Code:
``` kotlin 
    //create a TreeSet
    val treeSet: TreeSet<PlayerScore> = TreeSet()
    //create 3 PlayerScore object，and use 'id101Obj' reference to the first obj
    //put these objects into the treeset
    val id101Obj = PlayerScore(101,100)
    //put in different order 
    treeSet.add(PlayerScore(102,200))
    treeSet.add(id101Obj)
    treeSet.add(PlayerScore(103,300))

     println("Add the objects in a different order and print out by score from smallest to largest:")
    showTreeSet(treeSet)

    treeSet.remove(id101Obj)
    id101Obj.score = 400
    treeSet.add(id101Obj)
    println("remove id101Obj from TreeSet，change score，and then add to treeSet中， print out by score from smallest to largest:")
    showTreeSet(treeSet)
```
Console output:
```
Add the objects in a different order and print out by score from smallest to largest:
rank:1 score:100 id:101
rank:2 score:200 id:102
rank:3 score:300 id:103

remove id101Obj from TreeSet，change score，and then add to treeSet中， print out by score from smallest to largest:
rank:1 score:200 id:102
rank:2 score:300 id:103
rank:3 score:400 id:101
```

## Move Forward
The solution that "Remove it, edit it, and add it" can finish resort, but treeSet is thread-unsafe. If there are many threads update player's score, it might be cause some problems.

Although TreeSet can sort as it inserts, it only allow insert once. The modification of the score after insertion does not change the sorting.

TreeSet doesn't make sense for rank list that move around a lot.

## Another Rank List Solution
I use TreeSet because it can sort as it inserts, but it's not appropriate for a rank list.

So, if 
So another way to think about it is, if you only use a longer ArrayList to store the player's score object, you also sort it on insert/update/delete?

Insert/update/delete, modify the data in the ArrayList, and then sort.


