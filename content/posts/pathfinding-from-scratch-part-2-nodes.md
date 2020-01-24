---
title: 'Pathfinding From Scratch, Part 2: Nodes'
subtitle: >-
  I designed an abstract class Node for users to implement fields and functions
  specific to their implementation of A*.
date: 2020-01-24T16:07:18.913Z
thumb_img_path: /images/pathfinding2.jpg
layout: post
---
With the priority queue structure out of the way, I could implement the A\* algorithm from scratch. There are better articles whose efforts I will not plagiarize that discuss the A\* algorithm at length. I recommend [Red Blob Game's article](https://www.redblobgames.com/pathfinding/a-star/introduction.html) and [Wikipedia's entry](https://en.wikipedia.org/wiki/A*_search_algorithm) about the algorithm.

A\* is a pathfinding algorithm which combines the efforts of two other algorithms, Dijkstra's algorithm and the greedy best-first search algorithm. Dijkstra's algorithm measures the shortest possible distance traveled from one node to any connected node on the map. The greedy best-first search algorithm approximates the shortest distance from any node to the end node. Dijkstra guarantees the shortest path while the greedy best-first algorithm guarantees a fast solution. By taking both calculations (shortest distance traveled + approximate remaining distance), A* represents a happy medium between effectiveness and efficiency.

My goal was to abstract the algorithm and its implementation. Ideally, you should be able to use the bare bones of the algorithm in any situation so long as you define parts specific to your implementation. For example, the algorithm does not need to know how to calculate the approximate distance from one node to the end node: the nodes can handle this problem among themselves. The first step is to separate out information specific to the algorithm and information that can be hidden from the algorithm. The algorithm should be the final implementation of itself. The information specific to the user implementation should be relegated to some Node abstract class. Thus the algorithm class should only work with nodes of the same type, and nodes should only cooperate with nodes of the same type. So we have two class headers to start off with:

	public sealed class Pathfinder<N> where N : Node<N> { ... }
	public abstract class Node<N> where N : Node<N> { ... }

The priority queue becomes important here! In the previous post, I showed that I made two versions: one where an object's priority depends upon its implementation of IComparable, the other where the priority and the datum (i.e. the object you push/pop from the queue) are separate values. If I implement the algorithm using the former type, I can hide the node-to-node comparison from the algorithm. So, the Node header becomes something like this now:

	public abstract class Node<N> : IComparable<N> where N : Node<N> { ... }

The Node class contains fields that all nodes need to have. Some fields, like the distance traveled and the approximate remaining distance (or heuristic!), don't need to be accessed outside of the abstract node itself. The user needs to implement a function to calculate the cost of going from one node to another, and another function to calculate how far away one node is from another. However, the user shouldn't have to interact with fields that depend on those functions but not vice versa. The more that's hidden, the less the user has to worry about.

	private float _distanceTraveled;
	private float _heuristic;
	
These two variables determine the weight of the node. This is the value by which one node is compared to another node, like comparing the ASCII values of characters in a string. The Weight should be a get-only field which adds together these values.

	public float Weight { get { return _distanceTraveled + _heuristic; } }
	// we're making this a public field because the algorithm needs to keep track of each enqueued node's Weight
	//	to find the shortest path to the same node

Then we can implement IComparable<N>.CompareTo(N) based on the weight field.

	public int CompareTo(N obj){ return (int)(obj.Weight - this.Weight); }

Now we can make a priorityqueue that takes objects of the Node class!

There are also fields that should be protected, because they may be important for the user's implementation of certain functions. One is the Parent node, which is the node from which you travel to the current node. This basically makes a linked list of nodes, allowing us to easily reconstruct the path in the algorithm. The other is the Goal/Endpoint node. Each node should know how far it is from the end of the path (ie. be able to calculate its own heuristic). The user may want access to the Goal node for their own implementation as well, like to determine if one node is too far from the endpoint to be viable. This could have been a private field, but I figured that some user might find it useful.

	protected N goal { get; private set; }
	protected N parent { get; private set; }
	// even though we want these fields to be accessible to implementations of Node
	// 	we don't need/want them to be changed outside of the abstract class itself

I have functions called SetParent() and SetWeightConditions() which allow the user to set the parent and goal nodes of any given node. I have dedicated functions because these functions also set the total distance traveled and the heuristic. The user shouldn't be allowed to change the parent or goal nodes without also affecting these fields as well.

With these private and protected fields, we can leave the following up to the user's implementation:

	public abstract float GetTravelCost(N to);	// get cost of traveling from here to other node adjacent to this one
	public abstract float GetDistance(N to);	// get heuristic distance from here to other node
	public abstract List<N> GetNeighbors();		// get list of accessible nodes adjacent to this node

There are other functions I have not included because their use does not become apparent until I implement the algorithm itself.
