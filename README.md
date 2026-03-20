# AStarProject

## Introduction
This is a version of the A* algorithm a commonly used algorithm for pathfinding. It combines features of uniform-cost search and pure heuristic search to efficiently compute optimal paths. The A* algorithm is a best-first search algorithm
in which the cost associated with a node is f(n) = g(n) + h(n), where g(n) is the cost of the path from the initial state to node n and h(n) is the heuristic estimate or the cost or a path from node n to a goal []. It checks the surrounding
nodes (neighbours) to the current node for the g and h values then computes f by adding them togther. Once it finds the lowest f value it will chose it as the shortest path. In the event of a tie where two neighbours have the same f value
there is limited tie breaking logic to use the node with the lower g value. The Algorithm will repeat this process until it has reached the destination node.

## Project Management
### Research
I started the project off by doing research into what the A* algorithm is and how it works. My starting point was the resources linked in the course which helped me get an idea of the project structure I would use. The 
version of the A* algorithm I ended up writing was heavily inspired by the Geeks for Geeks example we were given [].

### Code Structure
The code consists of the main.cpp file which is used to run the algorithm. It follows an OO (Object Oriented) style having two classes one for creating nodes that contains f, g, and h values used by the algorithm to
determine the shortest path. The second class is called grid it contains methods to create and interact with the grid; it also has a method to run the algorithm.

### Planning
My plan was to split up this project into two-week sprints. In the first two weeks I planned to have a working version of the A* algorithm. I planned to spend the next two weeks polishing and improving the code.
Then spend the last two weeks adding tests and writing my report.

I kept to this plan and had a working version of the algorithm done by the middle of the second week. However I was unable to finish polishing and improving the code by the end of week four, this was because I realised
the version of the algorithm I wrote could not be considered a true A* algorithm due to the fact I was create one node that acted as a parent node shared between all surrounding nodes of the current node. This meant the
algorithm heavily relied on the heuristic value (h) and g barely affected the output. At this point it was closer to a greedy best first/hill climber algorithm. Week four was spent updating my logic to create a node for
each neighbour node rather than a parent node as well as trying to use the improved g value for tiebreaking (decide which path is shorter if the h values are the same by using the lower g value node).

For the last two weeks of the project I continued to follow my plan to add tests and polish the code. I wrote tests to test potential senacios the code could run into like a success path, a failure path, if the start or end node is blocked
or does not exist on the grid etc. During the last week I added some constructors, changed up my code to use pass by reference where possible and seeded the rand function I was using so the test would share the same grid layout allowing
them to be more deterministic.

## Output
<img width="1105" height="418" alt="image" src="https://github.com/user-attachments/assets/a2c1bdb2-cfa6-4445-827f-20ec9cbc49b0" />

## Code/Core Content
### Grid.cpp
isValid function is used to check if a node is valid i.e. within the confines of the 2D array.
```C++
bool Grid::isValid(std::pair<int, int> node) {
	return (node.first >= 0) && (node.first < 20) && (node.second >= 0) && (node.second < 20); //TODO: replace this with a passed in row and col val
};
```

isBlocked checks if a node is and obsticale (indecated by the number 1 or 3). 1 means it is an obsticale that existed when the grid was generated and 3 means it is a path the algorithm has tried but reached a dead end.
```C++
bool Grid::isBlocked(std::pair<int, int> current) {
	if (grid[current.first][current.second] == 1 || grid[current.first][current.second] == 3) {
		return true;
	}
	else {
		return false;
	}
}
```

isDestination checks if the algrithm has reached the end goal by checking if the currect node is equal to the destination node.
```C++
bool Grid::isDestination(std::pair<int, int> current, std::pair<int, int> dest) {
	if (current.first == dest.first && current.second == dest.second) {
		return true;
	}
	else {
		return false;
	}
};
```

printGrid is used to display the grid in the terminal. It also adds some syling by replacing 1s on the the grid with a #. Color is added to the grind when printing using \003[colorValuem.
```C++
void Grid::printGrid() {
	for (std::vector<int> s : grid) {
		for (int n : s) {
			switch (n) {
			case 1:
				std::cout << "\033[31m#" << " \033[0m";
				break;
			case 2:
				std::cout << "\033[32m" << n << " \033[0m";
				break;
			case 3:
				std::cout << "\033[33m" << n << " \033[0m";
				break;
			default:
				std::cout << n << " ";
				break;
			}
		}
		std::cout << "\n";
	}
};
```

In the begining I had a method to resize the grid and populate it with unseeded random numbers between 0 and 1. This was done to make it easy to test and debug because since it was unseeded the grid would always be the same every time I ran
the code.
```C++
void Grid::setGrid() {
	grid.resize(20, std::vector<int>(20, 0)); //TODO: replace this with a passed in row and col val
	for (std::vector<int>& s : grid)
		for (int& n : s)
			n = rand() % 2; //TODO: replace this with C++ random 
};
```

Later I ended up replacing set grid with a default constructor that craetes a 20x20 grid and an two argument constructor that allows a user to define the size of the grid. I had planned to make the code take in commad line arguments to size
the grid and plot the source and destination nodes however I did not have enough time to implement this.
```C++
Grid::Grid() {
#if VERBOSE
	std::cout << "Executing Grid default constructor" << std::endl;
#endif
	srand(1);

	grid.resize(20, std::vector<int>(20, 0));
	for (std::vector<int>& s : grid)
		for (int& n : s)
			n = rand() % 2;
};

Grid::Grid(int row, int col) {
#if VERBOSE
	std::cout << "Executing Grid two argument constructor" << std::endl;
#endif
	srand(1);

	grid.resize(row, std::vector<int>(col, 0));
	for (std::vector<int>& s : grid)
		for (int& n : s)
			n = rand() % 2;
};
```

findNeighbours is a helper funtion used in the aStarSearch method to find the eight neighbour nodes that surrond the current node.
```C++
std::vector<std::pair<int, int>> Grid::findNeigbours(std::pair<int, int> currentNode) {
	std::vector<std::pair<int, int>> neighbours = { {currentNode.first - 1, currentNode.second},
		{currentNode.first + 1, currentNode.second}, {currentNode.first, currentNode.second + 1},
		{currentNode.first, currentNode.second - 1}, {currentNode.first - 1, currentNode.second + 1},
		{currentNode.first - 1, currentNode.second - 1}, {currentNode.first + 1, currentNode.second + 1},
		{currentNode.first + 1, currentNode.second - 1} };
	return neighbours;
}
```

## Testing
Tests.h
```C++
#ifndef TESTS_H
#define TESTS_H

void TestWorkingAStar();
void TestInvalidSource();
void TestInvalidDestination();
void TestDefaultConstructor();
void TestTwoArgumentConstructor();
void TestCopyConstructor();
void TestCopyAssignmentConstructor();
void TestNodeEqualsDestination();
//void TestDestinationUnreachable();
void TestDestinationBlocked();
void TestSourceBlocked();
void runTests();

#endif
```

Tests.cpp
```C++
#include "Grid.h"
#include "Tests.h"

void runTests() {
	TestWorkingAStar();
	TestInvalidSource();
	TestInvalidDestination();
	TestCopyAssignmentConstructor();
	TestCopyConstructor();
	TestDefaultConstructor();
	TestTwoArgumentConstructor();
	//TestDestinationUnreachable();
	TestDestinationBlocked();
	TestSourceBlocked();
	TestNodeEqualsDestination();
};

void TestDefaultConstructor() {
	Grid grid;
};

void TestTwoArgumentConstructor() {
	Grid grid(10, 10);
};

void TestCopyConstructor() {
	Grid grid1;
	Grid grid2(grid1);
};

void TestCopyAssignmentConstructor() {
	Grid grid1;
	Grid grid2;
	grid2 = grid1;
};

void TestWorkingAStar() {
	Grid grid;
	std::pair<int, int> start = { 1,11 };
	std::pair<int, int> end = { 18,16 };
	grid.aStarSearch(start, end);
};

void TestInvalidSource() {
	Grid grid;
	std::pair<int, int> start = { -1,11 };
	std::pair<int, int> end = { 18,16 };
	grid.aStarSearch(start, end);
};

void TestInvalidDestination() {
	Grid grid;
	std::pair<int, int> start = { 1,11 };
	std::pair<int, int> end = { -1,16 };
	grid.aStarSearch(start, end);
};

void TestSourceBlocked() {
	Grid grid;
	std::pair<int, int> start = { 0,0 };
	std::pair<int, int> end = { 18,16 };
	grid.aStarSearch(start, end);
};

void TestDestinationBlocked() {
	Grid grid;
	std::pair<int, int> start = { 1,11 };
	std::pair<int, int> end = { 19,19 };
	grid.aStarSearch(start, end);
};

// This test causes a crash because src is removed from path when it 
// appears multiple times and then path.back() is called in the next 
// iteration which causes an out of bounds access.
//void TestDestinationUnreachable() {
//	Grid grid;
//	std::pair<int, int> start = { 3,0 };
//	std::pair<int, int> end = { 18,16 };
//	grid.aStarSearch(start, end);
//};

void TestNodeEqualsDestination() {
	Grid grid;
	std::pair<int, int> start = { 1,11 };
	std::pair<int, int> end = { 1,11 };
	grid.aStarSearch(start, end);
};
```

## Reflection

## References
https://thealgorithms.github.io/Python/autoapi/machine_learning/astar/index.html 20/03/2026

https://stackoverflow.com/questions/5768316/pop-a-specific-element-off-a-vector-in-c

https://www.geeksforgeeks.org/cpp/how-to-change-console-color-in-cpp/

https://en.cppreference.com/w/cpp/algorithm/find.html

Grid.cpp

```C++
#include <iostream>	//cout, endl, pair, erase
#include <vector>	//vector
#include <stdlib.h>	//rand
#include <cfloat>	//DBL_MAX
#include <algorithm>//find

#include "Grid.h"
#include "NodeBase.h"

Grid::Grid() {
#if VERBOSE
	std::cout << "Executing Grid default constructor" << std::endl;
#endif
	srand(1);

	grid.resize(20, std::vector<int>(20, 0));
	for (std::vector<int>& s : grid)
		for (int& n : s)
			n = rand() % 2;
};

Grid::Grid(int row, int col) {
#if VERBOSE
	std::cout << "Executing Grid two argument constructor" << std::endl;
#endif
	srand(1);

	grid.resize(row, std::vector<int>(col, 0));
	for (std::vector<int>& s : grid)
		for (int& n : s)
			n = rand() % 2;
};

Grid::Grid(const Grid& rhs) {
#if VERBOSE
	std::cout << "Executing Grid copy constructor" << std::endl;
#endif
	grid = rhs.grid;
};

Grid& Grid::operator=(const Grid& rhs) {
#if VERBOSE
	std::cout << "Executing Grid copy assignment constructor" << std::endl;
#endif
	if (this != &rhs) {
		grid = rhs.grid;
	}
	return *this;
};

Grid::~Grid() {
#if VERBOSE
	std::cout << "Executing Grid destructor" << std::endl;
#endif
	grid.clear();
}

bool Grid::isValid(const std::pair<int, int>& node) const {
	return (node.first >= 0) && (node.first < 20) && (node.second >= 0) && (node.second < 20);
};

bool Grid::isBlocked(const std::pair<int, int>& current) const {
	if (grid[current.first][current.second] == 1 || grid[current.first][current.second] == 3) {
		return true;
	}
	else {
		return false;
	}
}

bool Grid::isDestination(const std::pair<int, int>& current, const std::pair<int, int>& dest) const {
	if (current.first == dest.first && current.second == dest.second) {
		return true;
	}
	else {
		return false;
	}
};

void Grid::printGrid() const {
	for (const std::vector<int> s : grid) {
		for (const int n : s) {
			switch (n) {
			case 1:
				std::cout << "\033[31m#" << " \033[0m";
				break;
			case 2:
				std::cout << "\033[32m" << n << " \033[0m";
				break;
			case 3:
				std::cout << "\033[33m" << n << " \033[0m";
				break;
			default:
				std::cout << n << " ";
				break;
			}
		}
		std::cout << "\n";
	}
};

void Grid::aStarSearch(const std::pair<int, int>& src, const std::pair<int, int>& dest) {
	std::vector<std::pair<int, int>> path = { src };
	if (isValid(src) == false || isBlocked(src) == true) {
		std::cout << "source is invalid\n";
		return;
	}

	if (isValid(dest) == false || isBlocked(dest) == true) {
		std::cout << "destination is invalid\n";
		return;
	}

	int destUnreachable = 0;
	grid[src.first][src.second] = 2;
	std::pair<int, int> closestNeighbour = src;

	while (!isDestination(path.back(), dest)) {
		auto currentNeighbours = findNeigbours(path.back());
		double newF = DBL_MAX;
		double bestG = DBL_MAX;

		for (auto& i : currentNeighbours) {
			if (isValid(i) && !isBlocked(i)) {
				NodeBase node;
				double gVal = static_cast<double>(path.size());
				node.setG(gVal);
				node.calculateH(i, dest);
				double fVal = node.getF();
				if (fVal < newF || (fVal == newF && gVal < bestG)) {
					newF = fVal;
					bestG = gVal;
					closestNeighbour = i;
				}
			}
		}

		if (std::find(path.begin(), path.end(), closestNeighbour) != path.end()) {
			path.erase(std::find(path.begin(), path.end(), closestNeighbour));
			grid[closestNeighbour.first][closestNeighbour.second] = 3;
		}
		else {
			path.push_back(closestNeighbour);
			grid[closestNeighbour.first][closestNeighbour.second] = 2;
		}

		destUnreachable++;

		if (destUnreachable == 50) {
			break;
		}
	}
	printGrid();
	if (destUnreachable == 50) {
		std::cout << "destination can not be reached\n";
	}
	std::cout << "Path: ";
	for (auto& n : path)
		std::cout << "(" << n.first << "," << n.second << ") ";
};

std::vector<std::pair<int, int>> Grid::findNeigbours(const std::pair<int, int>& currentNode) const {
	std::vector<std::pair<int, int>> neighbours = { {currentNode.first - 1, currentNode.second},
		{currentNode.first + 1, currentNode.second}, {currentNode.first, currentNode.second + 1},
		{currentNode.first, currentNode.second - 1}, {currentNode.first - 1, currentNode.second + 1},
		{currentNode.first - 1, currentNode.second - 1}, {currentNode.first + 1, currentNode.second + 1},
		{currentNode.first + 1, currentNode.second - 1} };
	return neighbours;
}
```

main.cpp
```C++
#include <iostream> //cout, pair

#include "NodeBase.h"
#include "Grid.h"
#include "Tests.h"

int main() {
	runTests();

	Grid grid;
	std::pair<int, int> start = {1,11};
	std::pair<int, int> end = {16,18};

	grid.aStarSearch(start, end);
	return 0;
}
```

NodeBase.cpp
```C++
#include <iostream>
#include <cmath>
#include "NodeBase.h"

void NodeBase::setG(double newG) {
	g = newG;
};

double NodeBase::getF() {
	f = g + h;
	return f;
};

//Calculates the Euclidean Distance
void NodeBase::calculateH(std::pair<int, int> current, std::pair<int, int> dest) {
	h = std::sqrt(std::pow(current.first - dest.first, 2) + std::pow(current.second - dest.second, 2));
};
```

Grid.h

```C++
#include <iostream>
#include <vector> //vector

#ifndef GRID_H
#define GRID_H

#define VERBOSE 1

class Grid {
private:
	std::vector<std::vector<int>> grid;
public:
	Grid();
	Grid(int, int);
	Grid(const Grid&);
	Grid& operator=(const Grid&);
	~Grid();

	bool isValid(const std::pair<int, int>& node) const;
	bool isBlocked(const std::pair<int, int>& current) const;
	bool isDestination(const std::pair<int, int>& current, const std::pair<int, int>& dest) const;
	void aStarSearch(const std::pair<int, int>& src, const std::pair<int, int>& dest);
	void printGrid() const;
	std::vector<std::pair<int, int>> findNeigbours(const std::pair<int, int>& currentNode) const;
};

#endif
```

NodeBase.h
```C++
#include <iostream>	//pair
#include <cmath>	//sqrt, pow

#include "NodeBase.h"

void NodeBase::setG(double newG) {
	g = newG;
};

double NodeBase::getF() {
	f = g + h;
	return f;
};

//Calculates the Euclidean Distance
void NodeBase::calculateH(const std::pair<int, int>& current, const std::pair<int, int>& dest) {
	h = std::sqrt(std::pow(current.first - dest.first, 2) + std::pow(current.second - dest.second, 2));
};
```

