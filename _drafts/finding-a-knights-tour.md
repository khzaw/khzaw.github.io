---
layout: post
title: Finding A Knigt's Tour
tags: algorithms, chess
---

A Knight's Tour is a sequence of moves done by a knight on a chessboard such that it visits each and every square exactly once. Hence, the Knight's Tour problem would be finding whether there exists a knight's tour giving a starting position. In fancy computer science terms, it is a form of Hamilton path, which basically means you visit each vertex of the graph exactly once along the path.

### Searching for a Knight's Tour

### Backtracking

### Warnsdorf's rule
  You could try to improve by making use of a heuristic known as Warnsdorf's rule. It states that in each step, the sqaure with the least possible moves the knight can make from that square is favored. 