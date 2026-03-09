# AStarProject

Grid.cpp

```C++
#include <iostream>
#include <vector>
#include <stdlib.h>
#include "Grid.h"
#include "NodeBase.h"

bool Grid::isValid(std::pair<int, int> node) {
	return (node.first >= 0) && (node.first < 20) && (node.second >= 0) && (node.second < 20); //TODO: replace this with a passed in row and col val
};

bool Grid::isBlocked(std::pair<int, int> current) {
	if (grid[current.first][current.second] == 1 || grid[current.first][current.second] == 3) {
		return true;
	}
	else {
		return false;
	}
}

bool Grid::isDestination(std::pair<int, int> current, std::pair<int, int> dest) {
	if (current.first == dest.first && current.second == dest.second) {
		return true;
	}
	else {
		return false;
	}
};

void Grid::setGrid() {
	grid.resize(20, std::vector<int>(20, 0)); //TODO: replace this with a passed in row and col val
	for (std::vector<int>& s : grid)
		for (int& n : s)
			n = rand() % 2; //TODO: replace this with C++ random 
};

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

void Grid::aStarSearch(std::pair<int, int> src, std::pair<int, int> dest) {
	std::vector<std::pair<int, int>> path = { src };
	if (isValid(src) == false || isBlocked(src) == true) {
		std::cout << "source is invalid";
		return;
	}

	if (isValid(dest) == false || isBlocked(dest) == true) {
		std::cout << "destination is invalid";
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
	system("cls");
	printGrid();
	if (destUnreachable == 50) {
		std::cout << "destination can not be reached\n";
	}
	std::cout << "Path: ";
	for (auto& n : path)
		std::cout << "(" << n.first << "," << n.second << ") ";
};

std::vector<std::pair<int, int>> Grid::findNeigbours(std::pair<int, int> currentNode) {
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
#include <iostream> //cout
#include "NodeBase.h"
#include "Grid.h"

int main() {

	Grid grid;
	std::pair<int, int> start = {1,11};
	std::pair<int, int> end = {18,16};

	grid.setGrid();
	grid.printGrid();
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
#include <vector>

#ifndef GRID_H
#define GRID_H

class Grid {
private:
	std::vector<std::vector<int>> grid;
public:
	bool isValid(std::pair<int, int> node);
	bool isBlocked(std::pair<int, int> current);
	bool isDestination(std::pair<int, int> current, std::pair<int, int> dest);
	void aStarSearch(std::pair<int, int> src, std::pair<int, int> dest);
	void setGrid();
	void printGrid();
	std::vector<std::pair<int, int>> findNeigbours(std::pair<int, int> currentNode);
};

#endif
```
NodeBase.h
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
	void calculateH(std::pair<int, int> current, std::pair<int, int> dest);
};

#endif
```
