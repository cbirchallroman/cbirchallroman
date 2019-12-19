---
title: 'Pathfinding From Scratch, Part 1: Priority Queues'
subtitle: 'Part 1: Priority Queues'
date: 2019-12-19T20:43:41.694Z
thumb_img_path: /images/pathfinding1.jpg
excerpt: >-
  I developed an agnostic implementation of the A* pathfinding algorithm by
  first implementing my own generic priority queue. 
layout: post
---
For the last two weeks, I've been developing generic structures and algorithms in C#. My original idea was to make a second iteration of my long-term project *Capital City*, a city-building game, with improved architecture and so on. *Capital City* itself was a revision of an earlier project where I totally revamped the structure of the project, making better use of inheritance and encapsulation (along with improved databasing) because the products of the original project's blind development became unbearable to work with.

I found similar structural weaknesses in *Capital City*. Even though its development was planned out ahead of time, I began to develop in an ad hoc manner once I went out of the scope of that original plan. I told myself that if I were to make it again, I wouldn't leave one aspect unaccounted for before development. I would make better use of design patterns, data structures, and time complexity to make further development less of a pain in the ass.

# Priority Queues

For *Capital City* and its predecessor, I borrowed open source implementations of certain data structures. The priority queue was one of these: it was necessary to use for my pathfinding algorithm, but I didn't know how to implement it on my own (and at the time, I thought it would be the "boring" part of development--why reinvent the wheel?).

This time around, I wanted to implement my own priority queue. I wanted to make every aspect of my project modular, and to create each module from the bottom up without relying on the work of others. Furthermore, I wanted to exercise my knowledge of data structures and how to implement them. With all this background information out of the way, let's discuss implementation.

## Binary Trees vs. Array Lists

During my college course on data structures, I implemented a tree-based version of the priority queue structure.

![](/images/max-heap.svg.png "By Ermishin - Own work, CC BY-SA 3.0, https://commons.wikimedia.org/w/index.php?curid=12251273")

A binary tree is a structure made up of nodes, each with the capacity for one parent and two children to its left and right. Each node holds some kind of datum. A max heap is a binary tree where each node's datum is greater than that of either of its children. Look at how the above diagram depicts the structure: 100 is greater than both 19 and 36, and 19 is greater than both 17 and 3, and so on.

Consider the function of the priority queue: it returns the element of the highest (or lowest) priority within its set of elements. The binary heap is an easy way to implement a priority queue--all you have to do is return the element at the top of the heap!

Despite this, for my new implementation I opted to use an array list of elements instead of a binary heap. I didn't want to define a separate node class for my priority queue structure. I just wanted to easily sort my array list by comparing its elements directly. The sorting algorithm is the same as with the heap structure. In fact, the guide I followed calls this structure a heap despite using an array list to contain the elements. This implementation just as the great advantage of not holding each element inside a node. Check out [this great resource](http://pages.cs.wisc.edu/~vernon/cs367/notes/11.PRIORITY-Q.html#imp) for how to implement a priority queue in the same way--I'm not going to plagiarize their work, and it's already well-written.

I wrote the PriorityQueue\<T> class where T implements the IComparable\<T> interface. This allows each element of type T to be compared to one another, in order to be sorted by the structure. This way, the burden of comparing elements is put on the user. I used this specific structure for my pathfinding implementation, where the Node class implements IComparable\<Node>. However, I wanted to make a more broadly applicable implementation of the priority queue, for when the user doesn't want to define or use a class that compares itself to others.

## Separate Priority and Datum Values

I made a new class called PriorityQueue\<P, D> where P extends IComparable\<P> and D has no limitations. The secret to this implementation is a hidden class called PriorityNode\<P,D>, which implements IComparable towards itself. Then, the CompareTo(PriorityNode\<P, D>) function simply compares the P values of the two nodes.

Now that I have a class with two values that is compared to others based on one of those values, I can simply use this class in my original priority queue. Really, PriorityQueue\<P, D> is just an object that contains PriorityQueue\<T> where T is PriorityNode\<P, D>! This makes a priority queue where each element has an associated but external priority value, according to which all the elements are sorted.

You can check out the source code for both my priority queues and my pathfinding implementation on [this repository](https://github.com/cbirchallroman/generic-data-structures-cs)! Soon, I'll write something about how I implemented pathfinding.
