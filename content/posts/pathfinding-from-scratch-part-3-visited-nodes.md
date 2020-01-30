---
title: 'Pathfinding from Scratch, Part 3: Visited Nodes'
date: 2020-01-30T22:01:07.475Z
thumb_img_path: /images/pathfinding3.jpg
excerpt: >-
  I devised a formula to give each position in an n-dimensional world a unique
  identifier number.
layout: post
---
Nevertheless, the algorithm needs to know which nodes have already been visited and the current priorities of the enqueued nodes. The positioning of elements in the priority queue is dynamic because a node's priority can be updated so long as it's unvisited. This means I can't just grab the currently enqueued version of the node, because that would mean performing an O(n) search to find the node with the given coordinates. I opted to keep track of current priorities and visited nodes by using a 2D array, where each element's coordinates [x,y] corresponded to the coordinates of some node.

	// given integers sizeX, sizeY
	bool[,] visited = new bool[sizeX, sizeY];	// keep track of which nodes have been visited (true/false)
	float[,] weights = new float[sizeX, sizeY];	// keep track of the priorities/weights of enqueued nodes

This solution means that I have to supply the algorithm with the dimensions of the map. This is undesirable because the algorithm does not use this information for anything else, and it limits the algorithm to 2D planes. What if the user wanted to implement pathfinding over some 3D environment?

Reading over the algorithm, I realized that the program only has to relate information to certain 'spots' in the world. The algorithm doesn't have to know where the spots are; it only has to record information about spots or nodes which have been traversed. This means that instead of creating a 2D array of elements which correspond to a certain position, I could use a dictionary to simply associate those elements with a certain position.

	Dictionary<int, bool> visited = new Dictionary<int, bool>();
	Dictionary<int, float> weights = new Dictionary<int, float>();

Upon implementing this, I realized that the visited dictionary would never contain any false values representing unvisited nodes. So long as I only ever add visited nodes to the dictionary, each key's value is always going to be true. The bool value is redundant! So, I turned visited from an dictionary that maps integers to booleans, into a hashset that records integers only.

	HashSet<int> visited = new HashSet<int>();
	Dictionary<int, float> weights = new Dictionary<int, float>();
# The Key

With the information containers ready, I needed to develop a way for them to assign the necessary information to nodes in any dimension. The identifying field for each node would have to be an integer unique to its position in the world. Having worked with matrices before, I had a handy equation to convert two-dimensional coordinates to a scalar.

	// given:
	//	x, the position of the object in the x-axis [0, sizeX)
	//	y, the position of the object in the y-axis [0, sizeY)
	int positionID = x + y * sizeX;

This equation works because by calculating (y * sizeX), you ensure that there's no overlap between elements with the same values for different components. Let's represent this in binary numbers, using a world that's 16 by 16 tiles. If there is an element on tile (4,4), you could not just add the two components together because this could be confused with (0,8), (8,0), (6,2) and so on. You need to differentiate the different components in the unique identifier itself. So, because the world is 16 by 16, let's multiply the x component by 16. The result of 4 * 16 + 4 is 68. In binary, this is 0100 0100, or 0x44 in hexadecimal. The two components are preserved inside the integer itself! On the other hand, (0,8) is  10000000 = 128, and (8,0) is 0000 1000 = 8, and (6,2) is 0010 0110 = 38.

Although this is easily demonstrated in binary using a 2D map the length of whose axes are powers of two, this holds true for maps of *n* dimensions of irregular lengths. The important caveat for an irregular map is to make sure that a component is not multipled by the length of its own axis, but that of the preceding component. Also, for each additional dimension, you multiply the previous factor by the magnitude of the next axis rather than just multiplying by that one axis' length. The factor accumulates! Consider how, for a 3D map of 16 by 16 by 16, you would need another nibble to account for the third component of any element. This third component needs to be multipled not by 16, but by 16*16 or 256, to avoid overlap with the other two nibbles.

Here's the function I came up with to calculate the unique ID for some position in n dimensions:

	int GenerateIndex(int[] coordinates, int[] worldSize){

		int rank = worldSize.Length;	// the number of dimensions

		// if the coordinates and world size have different dimensions, return -1
		if(coordinates.Length != rank)
			return -1;
				

		int factor = 1;	// multiplication factor begins at 1
		int index = coordinates[rank - 1]; // the last component is added without multiplying it firs
		for(int dimension = rank - 2; dimension >= 0; dimension--){

			factor *= worldSize[dimension + 1];	// multiply factor by previous size dimension
			index += coordinates[dimension] * factor;	// add component times factor

		}

		return index;

	}

Although I include this function as part of the pathfinding namespace, I think it's ridiculous if anyone should actually use it. Any user should implement their own hardcoded function to calculate the position ID of a node without calling this function. Otherwise, they'd have to deal with an ugly for-loop and extra variables which must needlessly complicate time and space. If the user knows how many dimensions any world using their implementation has, simply hardcode for that. Since I doubt anyone's going to be implementing any pathfinding in four dimensions, here's an easy formula for a unique ID for a position in 3D.

	// given:
	//	x, the position of the object in the x-axis [0, sizeX)
	//	y, the position of the object in the y-axis [0, sizeY)
	//	z, the position of the object in the y-axis [0, sizeZ)
	int positionID = (x) + (y * sizeX) + (z * sizeX * sizeY);

That's better than initializing four different variables and looping for O(n)!
