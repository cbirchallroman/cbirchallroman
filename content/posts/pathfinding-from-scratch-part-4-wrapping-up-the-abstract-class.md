---
title: 'Pathfinding from Scratch, Part 4: Wrapping Up the Abstract Class'
date: 2020-02-19T22:19:23.069Z
thumb_img_path: /images/pathfinding4.jpg
excerpt: >-
  I show the complete pathfinding algorithm, and explain the user's role in
  implementing the algorithm through the Node abstract class.
layout: post
---
So now I have the following abstracted algorithm, where any information that the algorithm needs to know is encapsulated inside the data structure for the node. The function returns a queue of nodes in the order of traversal.

	// given Nodes of type N "to1", "to2", "to3" (endpoints) and "from" (startpoint)

	// create priority queue of objects of type N
	PriorityQueue<N> queue = new PriorityQueue<N>();

	// we're going to look for a path BACKWARDS to allow for multiple possible endpoints
	// 	so instead of enqueuing "from", we're going to enqueue the three endpoints
	//	and the goal for the algorithm will be the start node
	queue.Enqueue(to1);
	queue.Enqueue(to2);
	queue.Enqueue(to3);
	N goal = from;	// giving the N object "from" a new identifier so its purpose is clear
	

	// using a dictionary + hashset because access is O(n)
	//	the integer key is equal to each node's unique world position
	Dictionary<int, float> weights = new Dictionary<int, float>();
	HashSet<int> visited = new HashSet<int>();

	// while there are still unvisited nodes enqueued
	while(queue.Count > 0){

		// get current node
		N current = queue.Dequeue();

		// mark current node as visited
		visited.Add(current.GetIndex());

		// if this is the goal, iterate through nodes to construct the path
		if(current.Equals(goal)){

			return current.RetracePath();

		}
		
		// enqueue neighbors
		Stack<N> neighbors = current.GetNeighbors();
		while(neighbors.Count > 0){

			// if this spot is visited, continue
			//	visited will never contain any false values,
			//	so referring to visited.ContainsKey(int) is a safe bet
			if(visited.Contains(neighbor.GetIndex()))
				continue;

			// if queue contains neighbor already, replace with new object if weight is smaller
			//	otherwise don't consider this neighbor as an option
			if(weights.ContainsKey(neighbor.GetIndex())){

				float originalWeight = weights[neighbor.GetIndex()];
				if(neighbor.Weight < originalWeight)
					queue.Remove(neighbor);
				else
					continue;

			}

			// if we're past that, update weights[] and enqueue node
			weights[neighbor.GetIndex()] = neighbor.Weight;
			queue.Enqueue(neighbor);

		}

	}

	// return null if path is not found
	return null;

Let's return to the Node class to see what information is encapsulated and why.

# Revisiting Nodes

In the blog post about Nodes, I described four abstract functions left up to the user to implement:

	public abstract int GetIndex();			// get unique index of this node's coordinates
	public abstract float GetTravelCost(N to);	// get cost of traveling to other node of same type
	public abstract float GetDistance(N to);	// get distance from this node to other node of same type
	public abstract Stack<N> GetNeighbors();	// get accessible nodes adjacent to this node

Only two of these functions are called by the A* algorithm: GetIndex() which assigns each world position a unique identifying integer, and GetNeighbors() which returns a stack of every traversible node neighboring the current node. I discussed GetIndex() extensively in the last blog post. GetNeighbors() used to be a List<N>, but I changed it to Stack<N> to avoid using an iterator across a linked list.

There are two other (final) members of Node called by the algorithm: the RetracePath() function which returns the complete path as a queue of nodes once the goal is reached by iterating through each node's Parent, and the Weight field which returns the sum of the node's traveled distance from start and its estimated distance to the goal. Recall from the post on Nodes:

	public float Weight { get { return _distanceTraveled + _heuristic; } }

The expectation is that the user will use GetTravelCost(N) and GetDistance(N) to define the total weight of any node. Thus _distanceTraveled and _heuristic are defined as such:

	private float _distanceTraveled { get { return Parent == null ? 0 : Parent._distanceTraveled + Parent.GetTravelCost((N)this); } }
	private float _heuristic { get { return Goal == null ? 0 : GetDistance(Goal); }}

Because _distanceTraveled depends upon there being a predecessor node, it first checks if the current node has a Parent. The _distanceTraveled for any node is equal to the distance traveled by its parent plus the distance traveled between the parent and its successor node.

On the flip side, _heuristic depends on there being a goal node. Thus it first checks if the current node has a Goal, and then it sets _heuristic equal to the estimated distance from the current node to the goal.

GetNeighbors() is left for the user to implement. Therefore the implementation is specific to each possible world structure created by the user. Before getting into this, I should mention two important final Node functions which the pathfinding algorithm does not call.

SetParent(N parent) sets the current node's parent equal to some node *parent*. If *parent* is not null, then SetGoal(N goal) is called inside the body of SetParent(N). Both functions are public because the user is expected to use SetParent(N) inside the body of GetNeighbors(), and SetGoal(N) is called by the algorithm itself on each endpoint node. So we can rewrite the beginning of the pathfinding function as:

	// given Nodes of type N "to1", "to2", "to3" (endpoints) and "from" (startpoint)

	// create priority queue of objects of type N
	PriorityQueue<N> queue = new PriorityQueue<N>();

	// we're going to look for a path BACKWARDS to allow for multiple possible endpoints
	// 	so instead of enqueuing "from", we're going to enqueue the three endpoints
	//	and the goal for the algorithm will be the start node
	N goal = from;	// giving the N object "from" a new identifier so its purpose is clear	
	to1.SetGoal(goal);	// setting this node's goal to the "goal" node; repeat for each endpoint
	queue.Enqueue(to1);
	to2.SetGoal(goal);
	queue.Enqueue(to2);
	to3.SetGoal(goal);
	queue.Enqueue(to3);

With all this in mind, the pseudocode for some implementation of GetNeighbors() will look something like this:

	stack neighbors := {}

	for each position P adjacent to this node:
		create neighboring node N2 at P

		if N2 is traversable from this node:
			push N2 to neighbors

	return neighbors

What the positioning of each node looks like, and what determines if a node is traversable, should be developed by the user when they extend Node.

In the next post, I will develop a 2D world which uses the pathfinding algorithm to navigate through itself. This will require me to write a concrete implementation of Node, and whatever else comes with the structure of the virtual world. My concrete derived class, "Node2D", will interact with functions from "World2D" to determine which nodes are nearby and which of those are traversable.
