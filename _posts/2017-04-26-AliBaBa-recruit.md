---
layout: post
title:  Find a Path on Chessboard
date:   2017-04-26 2017-04-26 22:38:06 +08:00
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
The test recommends `C++` as programming language, but I am not skillful enough in `C++` and the algorithm is out of practice as well.
The countdown was fleeting, I didn't have sufficient time to debug locally until it pass. Lastly, I can only feel embarrassed to hand in the uncompiled answer, it's just like a... a kind of shame : )

After having dinner, I determined to do something to revise `C++` and its grammar. The best way is to finish the programming assignment.

I must **confront failure squarely**. In any case, I need devote more endeavors on **Algorithm**, which is extremely important and fundamental, for further solid work.


### For now, just KEEP MOVING.

## Solution
**probably**, because I couldn't recall the problem precisely. But at least, I tried and completed the task.

{% highlight cpp %}
#include "stdafx.h"

#include <iostream>
#include <sstream>
#include <vector>
#include <math.h>
#include <algorithm>
#include <stack>

using namespace std;

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
	Position(int i = 0, int j = 0)
	{
		x = i, y = j;
	}
	bool operator== (const Position &c) const
	{
		return x == c.x && y == c.y;
	}
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
	Chess(int m, int n, const vector<int>& tx, const vector<int>& ty, const vector<int>& wx, const vector<int>& wy);
	bool findPath();
private:
	Position current_pos, end_pos, start_pos;
	vector<vector<int>> chessboard;    //store the matrix
	Stack<Position> chess_stack;
	vector<Position> path_track;
	void pushUnvisited(int, int);
	friend ostream& operator<< (ostream&, const Chess&);
};

Chess::Chess(int m, int n, const vector<int>& tx, const vector<int>& ty, const vector<int>& wx, const vector<int>& wy)
{
	float k, b = 0;
	int start_x, end_x, start_y, end_y = 0;
	int approx_y, approx_x = 0;
	chessboard.resize(m);
	for (int i = 0; i < m; i++)
	{
		chessboard[i].resize(n);
		for (int j = 0; j < n; j++)
			chessboard[i][j] = 1;
	}
	cout << *this;
	for (int i = 0; i < tx.size(); i++)
	{
		k = (1.0 * wy[i] - ty[i]) / (wx[i] - tx[i]);
		b = ty[i] - k * tx[i];
		start_x = min(tx[i], wx[i]);
		start_y = min(ty[i], wy[i]);
		end_x = max(tx[i], wx[i]);
		end_y = max(ty[i], wy[i]);
		if ((end_x - start_x) > (end_y - start_y))
			for (int j = start_x; j <= end_x; j++)
			{
				approx_y = round(j * k + b);
				chessboard[j][approx_y] = 0;
			}
		else
			for (int j = start_y; j <= end_y; j++)
			{
				approx_x = round((j - b) / k);
				chessboard[approx_x][j] = 0;
			}
	}
	start_pos = Position(0, 0);
	end_pos = Position(m - 1, n - 1);
	cout << *this;
}

void Chess::pushUnvisited(int i, int j)
{
	if (chessboard[i][j] > 0 || (i == end_pos.x && j == end_pos.y))
		chess_stack.push(Position(i, j));
}

bool Chess::findPath()
{
	int i, j;
	current_pos = start_pos;
	while (!(current_pos == end_pos))
	{
		i = current_pos.x;
		j = current_pos.y;

		if (!(current_pos == start_pos))
			chessboard[i][j] = -1;
		if (i > 1)
			pushUnvisited(i - 1, j);
		if (i < end_pos.y)
			pushUnvisited(i + 1, j);
		if (j > 1)
			pushUnvisited(i, j - 1);
		if (j < end_pos.x)
			pushUnvisited(i, j + 1);

		// This condition is useless, always successful.
		if (chess_stack.empty())
			return false;
		else
		{
			path_track.push_back(current_pos);
			current_pos = chess_stack.pop();

			cout << *this;
		}
	}
	return true;
}

ostream& operator<< (ostream& out, const Chess& chess) {
	for (int i = 0; i <= chess.end_pos.y; i++)
	{
		for (int j = 0; j <= chess.end_pos.x; j++)
			out << chess.chessboard[i][j] << ' ';
		out << endl;
	}
	out << endl;
	return out;
}
int main()
{
	int m = 0;
	int n = 0;
	vector<int> tx;
	vector<int> ty;
	vector<int> wx;
	vector<int> wy;
	int num = 1;

	tx.push_back(1);
	ty.push_back(1);
	wx.push_back(3);
	wy.push_back(4);

	Chess cs = Chess(5, 5, tx, ty, wx, wy);

	// for (int i = 0; i < num; ++i) {
	//	 int x1, y1, x2, y2;
	//	 std::cin >> x1 >> y1 >> x2 >> y2;
	//	 tx.push_back(x1);
	//	 ty.push_back(y1);
	//	 wx.push_back(x2);
	//	 wy.push_back(y2);
	// }

	// This case is alright, but... I don't know others.
	std::cout << cs.findPath() << endl;
	return 0;

{% endhighlight %}
