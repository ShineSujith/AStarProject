# A* Algorithm Project
course: Bachelor of Engineering (Honours) in Software and Electronic Engineering
module: C++ Programming
by: Shine Sujith - G00414454

## Introduction
This is a version of the A* algorithm a commonly used algorithm for pathfinding. It combines features of uniform-cost search and pure heuristic search to efficiently compute optimal paths. The A* algorithm is a best-first search algorithm in which the cost associated with a node is **f(n) = g(n) + h(n)**, where **g(n)** is the cost of the path from the initial state to **node n** and **h(n)** is the heuristic estimate or the cost or a path from node n to a goal [1]. It checks the surrounding nodes (neighbours) to the current node for the **g** and **h** values then computes **f** by adding them togther. Once it finds the lowest **f** value it will chose it as the shortest path. In the event of a tie where two neighbours have the same **f** value there is limited tie breaking logic to use the node with the lower **g** value. The algorithm will repeat this process until it has reached the destination node. My version of the A * algorithm only uses one list to track the path with back tracking logic. In a normal A * algorithm you would have two lists an open (nodes that have not been evaluated) and a closed (nodes that have been evaluated) list.

## Project Management
### Research
I started the project off by doing research into what the A* algorithm is and how it works. My starting point was the resources linked in the course which helped me get an idea of the project structure I would use. The version of the A* algorithm I ended up writing was heavily inspired by the Geeks for Geeks example we were given []. I also used sites like c++ reference and Stack Overflow to look a example code and figure out how to implement it into my own project [] [].

### Code Structure
The code consists of the main.cpp file which is used to run the algorithm and the tests. It follows an **OO (Object Oriented)** style having two classes one for creating nodes that contains **f**, **g**, and **h** values used by the algorithm to determine the shortest path. The second class is called **grid** it contains methods to create and interact with the grid; it also has a method to run the algorithm called ```aStarSearch```.

Grid.h:

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

NodeBase.h:
```C++
#ifndef NODE_BASE_H
#define HODE_BASE_H

class NodeBase {
private:
	double g = 0;
	double h = 0;
	double f = 0;
public:
	void setG(double newG);
	double getF();
	void calculateH(const std::pair<int, int>& current, const std::pair<int, int>& dest);
};

#endif
```

### Planning
My plan was to split up this project into two-week sprints. In the first two weeks I planned to have a working version of the A* algorithm. I planned to spend the next two weeks polishing and improving the code. Then spend the last two weeks adding tests and writing my report.

I kept to this plan and had a working version of the algorithm done by the middle of the second week. However I was unable to finish polishing and improving the code by the end of week four, this was because I realised the version of the algorithm I wrote could not be considered a true A* algorithm due to the fact I was creating one node that acted as a parent node shared between all surrounding nodes of the current node. This meant the algorithm heavily relied on the heuristic value (h) and g barely affected the output. At this point it was closer to a greedy best first/hill climber algorithm. Week four was spent updating my logic to create a node for each neighbour node rather than a parent node as well as trying to use the improved g value for tiebreaking (decide which path is shorter if the h values are the same by using the lower g value node).

For the last two weeks of the project I continued to follow my plan to add tests and polish the code. I wrote tests to test potential senacios the code could run into like a success path, a failure path, if the start or end node is blocked or does not exist on the grid etc. I also started to use GitHub copilot to review my code, it was through these reviews that I discovered I had not written a true A * algorithm. During the last week I added some constructors, changed up my code to use pass by reference where possible and seeded the rand function I was using so the test would share the same grid layout allowing them to be more deterministic.

## Output
<img width="1105" height="418" alt="image" src="https://github.com/user-attachments/assets/a2c1bdb2-cfa6-4445-827f-20ec9cbc49b0" />
<sub>Figure 1</sub>

## Code/Core Content
### Grid.cpp
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

```IsValid``` function is used to check if a node is valid i.e. within the confines of the 2D array. This fucntion is almost one for one what it was in the Geeks for Geeks example the main changes being it is now a member function of the Grid class and the node argument has been changed to use pass by reference instead of pass by copy to improve memory efficientcy and speed.
```C++
bool Grid::isValid(const std::pair<int, int>& node) const {
	return (node.first >= 0) && (node.first < 20) && (node.second >= 0) && (node.second < 20);
};
```

```IsBlocked``` checks if a node is and obsticale (indecated by the number 1 or 3). 1 means it is an obsticale that existed when the grid was generated and 3 means it is a path the algorithm has tried but reached a dead end.
```C++
bool Grid::isBlocked(const std::pair<int, int>& current) const {
	if (grid[current.first][current.second] == 1 || grid[current.first][current.second] == 3) {
		return true;
	}
	else {
		return false;
	}
}
```

```IsDestination``` checks if the algrithm has reached the end goal by checking if the currect node is equal to the destination node.
```C++
bool Grid::isDestination(const std::pair<int, int>& current, const std::pair<int, int>& dest) const {
	if (current.first == dest.first && current.second == dest.second) {
		return true;
	}
	else {
		return false;
	}
};
```

```PrintGrid``` is used to display the grid in the terminal. It also adds some syling by replacing 1s on the the grid with a #. Color is added to the grind when printing using \003[colorValuem.
```C++
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
```

In the begining I had a method to resize the grid and populate it with unseeded random numbers between 0 and 1. This was done to make it easy to test and debug because since it was unseeded the grid would always be the same every time I ran the code.
```C++
void Grid::setGrid() {
	grid.resize(20, std::vector<int>(20, 0));
	for (std::vector<int>& s : grid)
		for (int& n : s)
			n = rand() % 2; 
};
```

Later I ended up replacing set grid with a **default constructor** that craetes a 20x20 grid and an **two argument constructor** that allows a user to define the size of the grid. I had planned to make the code take in command line arguments to size the grid and plot the source and destination nodes however I did not have enough time to implement this.
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

I added a destructor to clear the grid if it ever goes out of scope, this was more so just to follow good practice and allow possible expansion upon the current code.
```C++
Grid::~Grid() {
#if VERBOSE
	std::cout << "Executing Grid destructor" << std::endl;
#endif
	grid.clear();
}
```

Since I added a destructor I had to follow the rule of three and add a copy constructor and a copy assignment constructor. Through a previous lab I learned that the vector library overload the assignment operator to make a deep copy of a vector when assigning them beacuse of this it made adding these constructors much more simple. I was running low on time when adding these constructors and this was a time that the inline AI suggestions came in handy.
```C++
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
```

```FindNeighbours``` is a helper funtion used in the ```aStarSearch``` method to find the eight neighbour nodes that surrond the current node.
```C++
std::vector<std::pair<int, int>> Grid::findNeigbours(const std::pair<int, int>& currentNode) const {
	std::vector<std::pair<int, int>> neighbours = { {currentNode.first - 1, currentNode.second},
		{currentNode.first + 1, currentNode.second}, {currentNode.first, currentNode.second + 1},
		{currentNode.first, currentNode.second - 1}, {currentNode.first - 1, currentNode.second + 1},
		{currentNode.first - 1, currentNode.second - 1}, {currentNode.first + 1, currentNode.second + 1},
		{currentNode.first + 1, currentNode.second - 1} };
	return neighbours;
}
```

The ```aStarSearch``` is the method for running the A* algortihm on the grid. It starts by checking if the start and end nodes are valid using isValid and isBlocked function. There is a destination unreachabe variable that prvents the code from getting stuck in an infinite loop. There are possitives and negatives to having the destination unreachable variable that main one being if the grid is too big 50 iterations may not be enough to reach the destination. The other downside is that if the start node is completly surrounded on all eight sides by blocking or invalid nodes destUnreachable does not work and the code crashes beacsue the back track logic I have removes path values that repeat, so when the start node repeats it is erased from the path once that happens in the next iteration the code calls path.back() on an empty path variable.

First the starting node is marked with a 2 this means that it is a part of the shortest path. Next it enters the while loop this is where the main logic for the algoritm takes place. The findNeighbours functionis called on the last element of the path vector which on the first iteration is src or the source/start node. newF and bestG are used to determine the shortest path they are first initailised to the max value of a double using DBL_MAX. In the while loop there is a for range loop that loops through the neighbours of the current node. The for loop starts by checking if the neighbour node is valid and is unblocked using the isValid and isBlocked functions. If the neighbour is valid a node is created for it using the NodeBase class. A static_cast<double> is used to cast the size of the current path to a double at complie time then node.setG is called to set the g value of the neighbour node. The heuristic value h is then calculated by calling node.calculateH and passing in the current neighbour node as well as the destination node. Once we have both the g and h values for the neighbour node we get f using node.getF. The f value for the neighbour is temporarily stored in the fVal variable, the code checks if the current neighbours f (fVal) is lower than newF by doing this for all the neighbour nodes we can determine the shortest path i.e. the neighbour with thw lowest f value.

In the case that fVal is equal to newF we use the g value to determine the shortest path although in the current state of the code this is unreliable since g is the size of the path. Sometimes there will be nodes that repeat beacuse the back track logic under the for range loop will push a node twice and on the next iteration the duplicate nodes will be erased and marked as a 3 indicating it is now blocked. If the node is not repeated and is the closest neighbour it is added to the path and marked as a 2. Finally once we reach the destination node we exit the while loop and print the grid using printGrid and display the path taken.
```C++
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
```

## Testing
Testing palyed an important part in helping debug my project. During the last week I added two files Tests.h to declare test functions and a Tests.cpp file which contian the function definitions. You might notice that there is a test funtion that is comment out this will be explained later. Earier I mentioned seeding the the rand function to generate a fixed grid, originally I planned on creating a fixed grid in the test file an dusing that however after talking to my lecturer I realised with my current setup it would be easier to just seed the grid generation using srand.

### Tests.h:
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

### Tests.cpp:
The runTests function is called in main to runn all the other test function.
```C++
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
```

TestDefaultConstructor tests if the default constructor is called when instanciating a grid object with no parameters.
```C++
void TestDefaultConstructor() {
	Grid grid;
};
```

TestInvalidSource annd TestInvalidDestination both test the isValid function to see if the source or destination node is within the grid.
```C++
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
```

Similarly TestSourceBlocked and TestDestinationBlocked tests the isBlocked function to see if the source or destination are on blocked squares 1 or # squares.
```C++
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
```

TestWoringAStar test the succussesful run of the algorithm.
```C++
void TestWorkingAStar() {
	Grid grid;
	std::pair<int, int> start = { 1,11 };
	std::pair<int, int> end = { 18,16 };
	grid.aStarSearch(start, end);
};
```

TestDestinationUnreachable function is commented out because it causes a crash. This was the test that helped me reailse that the isDestUnreachable variable had a down side due to my back track logic. If I did not write this test I would never have know and it show just how important testing really is.
```C++
// This test causes a crash because src is removed from path when it 
// appears multiple times and then path.back() is called in the next 
// iteration which causes an out of bounds access.
//void TestDestinationUnreachable() {
//	Grid grid;
//	std::pair<int, int> start = { 3,0 };
//	std::pair<int, int> end = { 18,16 };
//	grid.aStarSearch(start, end);
//};
```

The final test is used to test for unexpected behavior if the start and end nodes are the same value.
```C++
void TestNodeEqualsDestination() {
	Grid grid;
	std::pair<int, int> start = { 1,11 };
	std::pair<int, int> end = { 1,11 };
	grid.aStarSearch(start, end);
};
```

## Future Improvements
* Pass in grid dimensions, source, and destination variables using the command line.
* Handle the empty path.back() crash by either not removing the element using erase if it is equal to the source node or handle it by throwing an exception.
* Add additional logic to make the tie breaking logic more reliable.
* Properly Implement random grid generation using something like default_random_engine instead of rand.
* Make get f return g + h, that way I dont need a private f variable in my NodeBase class.

## Reflection
During the course of this project I learned a lot about C++ in general, I learned how to create unit tests for C++ code, and gained valuable experince using many of the libraries C++ offers like the vectors and algorithm libraries [8] [9]. I think a big mistake I made while creating the code for the A * algoritm was that I tried to diviate from what the algorithm is too early. Instead I should  have tried writing a correct version of the algritm then tried to make my own adjustments to it. If I did I would not have ran into a time issue when I realised that the code I wrote was note a true A * algoritm. If I were to do this project again I would plan better by giving myself some tolerance in terms of time near the end in case something unforseen happens like realising that the way I was implementing the node checking could not be considered using the A* algorithm.

## References
[1] "machine_learning.astar", thealgorithms. [Online]. <https://thealgorithms.github.io/Python/autoapi/machine_learning/astar/index.html> [Accessed: March, 20th, 2026]

[2] Tarodev, "Pathfinding", youtube. [Online]. <https://www.youtube.com/watch?v=i0x5fj4PqP4> [Accessed: Febuary, 15th, 2026]

[3] Computerphile, "A* (A Star) Search Algorithm", youtube. [Online]. <https://www.youtube.com/watch?v=ySN5Wnu88nE> [Accessed: Febuary, 10th, 2026]

[4] "A* Search Algorithm", geeksforgeeks. [Online]. <https://www.geeksforgeeks.org/dsa/a-search-algorithm/> [Accessed: Febuary, 10th, 2026]

[5] "C++ reference", cppreference. [Online]. <https://en.cppreference.com/w/cpp.html> [Accessed: Febuary, 11th, 2026]

[6] "stackoverflow", stackoverflow. [Online]. <https://stackoverflow.com/questions> [Accessed: Febuary, 11th, 2026]

[7] "How to Change Console Color in C++?", geeksforgeeks. [Online]. <https://www.geeksforgeeks.org/cpp/how-to-change-console-color-in-cpp/> [Accessed: Febuary, 11th, 2026]

[8] "pop a specific element off a vector in c++", stackoverflow. [Online]. <https://stackoverflow.com/questions/5768316/pop-a-specific-element-off-a-vector-in-c> [Accessed: Febuary, 11th, 2026]

[9] "std::find, std::find_if, std::find_if_not", cppreference. [Online]. <https://en.cppreference.com/w/cpp/algorithm/find.html> [Accessed: Febuary, 18th, 2026]
