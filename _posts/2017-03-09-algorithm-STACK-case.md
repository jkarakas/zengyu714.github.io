---
layout: post
title:  Find a Path on Chessboard
date:   2017-03-09 22:53:20 +08:00
categories: Algorithm
tags:
- stacks
- recursive
- chessboard
---

* content
{:toc}


## Rethink
Today I participated in the campus recruitment of Alibaba Group and ran into some problem in programming test.
The test recommends `C++` as programming language, and I am not skillful enough in `C++` and algorithm out of practice.
So I decided to switch language to `Python` and thought about using `itertools` which reviewed yesterday. However, the short-cut is not feasible, and I realized that `stack` and `recursive` are the methods I had to use. Thus I had no choice but turned back to `C++` and reused its provided template. At the same time, the countdown was fleeting, I couldn't concentrate on thinking about algorithm. Lastly, I can only feel embarrassed to hand in the blank paper, it's just like a... a kind of shame : )


After having dinner, I determined to do something to revise `C++` and its grammar. The best way is to finish the programming assignment.


I must **confront failure squarely**. In any case, I need devote more endeavors on **Algorithm**, which is extremely important and fundamental, for solid work.


### For now, just KEEP MOVING.



## Solution
**probably**, because I couldn't recall the problem precisely. But at least, I tried and completed the task.

{% highlight cpp %}
#include "stdafx.h"

#include <iostream>
#include <vector>
#include <stack>

using namespace std;
typedef unsigned int uint;
const int LENGTH = 7;


template<class T>
class Stack : public stack<T>
{
public:
	T pop()
	{
		T temp = top();
		stack<T>::pop();
		return temp;
	}
};

class Position
{
public:
	Position(uint i = 0,uint j = 0)
	{
		x = i, y = j;
	}
	bool operator== (const Position &c) const
	{
		return x == c.x && y == c.y;
	}
private:
	int x, y;
	friend class Chess;
	friend istream& operator>> (istream &in, Position& p);
	friend ostream& operator<< (ostream &out, const Position& p);
};
istream& operator>> (istream &in, Position& p)
{
	in >> p.x >> p.y;
	return in;
}
ostream& operator<< (ostream &out, const Position& p)
{

	out << "[ " << p.x << " , " << p.y << " ]" << endl;
	return out;
}
class Chess
{
public:
	Chess();
	void findPath();
private:
	Position current_pos, end_pos, start_pos;
	vector<vector<uint>> chessboard;    //store the matrix
	Stack<Position> chess_stack;
	vector<Position> path_track;
	void pushUnvisited(uint, uint);
	friend ostream& operator<< (ostream&, const Chess&);
	uint length = LENGTH;
};
Chess::Chess()
{
	cout << "Please enter the non-negative value of the chessboard:\n";
	chessboard.resize(length);
	for (int i = 0; i < length; i++)
	{
		chessboard[i].resize(length);
		for (int j = 0; j < length; j++)
			cin >> chessboard[i][j];
	}
	cout << "Please enter the start position:\n";
	cin >> start_pos;
	cout << "Please enter the end position:\n";
	cin >> end_pos;

}
void Chess::pushUnvisited(uint i, uint j)
{
	if (chessboard[i-1][j-1] > 0 || (i == end_pos.x && j == end_pos.y))
		chess_stack.push(Position(i, j));
}
void Chess::findPath()
{
	uint i, j;
	current_pos = start_pos;
	while (!(current_pos == end_pos))
	{
		i = current_pos.x;
		j = current_pos.y;

		cout << *this;
		if (!(current_pos == start_pos))
			chessboard[i-1][j-1] = 0;
		if (i > 1)
			pushUnvisited(i - 1, j);
		if (i < LENGTH)
			pushUnvisited(i + 1, j);
		if (j > 1)
			pushUnvisited(i, j - 1);
		if (j < LENGTH)
			pushUnvisited(i, j + 1);

    // This condition is useless, always successful.
		if (chess_stack.empty())
		{
			cout << *this;
			cout << "Failure.\n";
			return;
		}
		else
		{
			path_track.push_back(current_pos);
			current_pos = chess_stack.pop();
		}
	}
	cout << *this;
	for (auto item : path_track)
		cout << item;
	cout << "Finally, it is end.\n";
}
ostream& operator<< (ostream& out, const Chess& chess) {
	for (int i = 0; i < chess.length; i++)
	{
		for (int j = 0; j < chess.length; j++)
			out << chess.chessboard[i][j] << ' ';
		out << endl;
	}
	out << endl;
	return out;
}
int main()
{
	Chess().findPath();
	return 0;
}
{% endhighlight %}
