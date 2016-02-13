---
layout: post
title: Finding A Knight's Tour
tags: algorithms, chess
---

{% include image.html url="/img/chess-knight.jpg" %}

A Knight's Tour is a sequence of moves done by a knight on a chessboard such that it visits each and every square exactly once. Subsequently, the objective of [the Knight's Tour  problem](https://en.wikipedia.org/wiki/Knight%27s_tour) is to determine whether there exists a Knight's Tour from a given starting position. In graph theory terms, it is a form of Hamiltonian path where you visit each vertex of the graph exactly once along the path. Tours can also be __cyclic__ or __closed__ if the final square is a knight's move away from the first and __acyclic__ or __open__ otherwise.

{% include image.html url="/img/knights-tour.gif" caption="An example of a Knight's Tour – Credit: Wikipedia" %}


On a typical 8 x 8 chessboard, according to [this paper](http://www.combinatorics.org/ojs/index.php/eljc/article/view/v3i1r5/comment), the number of undirected closed knight's tours was computed to be 13,267,364,410,532, _(directed would be twice that)_. The number of directed open tours on a 8 x 8 was not actually known and estimated to be about 2x10^16 until in 2013, [Alex Chernov computed](http://oeis.org/A165134
) that the number is 19,591,828,170,979,904.

On an irregular board, the smallest rectangular board dimensions in which there is an open knight's tour is 3 x 4.

I was trying to solve problem out manually on a chessboard. There are some strategies if you start from a corner, but I keep running into dead-ends, if I start from a non-corner position. So, I resort to programming and here are some of my attempts to search for a knight's tour.

## Backtracking

With a depth-first traversal with backtracking, a knight tour can be searched. The gist is that at each branch of traversal, go with the first branch and if the search reaches a dead-end, go back a step and try other alternatives. Eventually, either a tour will be found or there is no possible knight tour from the given starting position.

{% highlight python %}
X = 0
Y = 1

def chess2index(p, boardsize):
    x = boardsize[X] - int(p[1])
    y = ord(p[0]) - ord('a')
    return x, y

def is_legal(pos, board, boardsize):
    return 0 <= (pos[X]) < boardsize[X] and \
           0 <= (pos[Y]) < boardsize[Y] and \
           not board[pos]


def board_string(board, boardsize):
    string = ''
    for x in range(boardsize[X]):
        for y in range(boardsize[Y]):
            string += '%3d' % board[(x, y)]
        string += '\n'
    return string

def knight_tour(currentpos, board, count):
    if count >= boardsize[X] * boardsize[Y]:
        return True

    for d in offsets:
        newpos = currentpos[X]+d[X], currentpos[Y]+d[Y]
        if is_legal(newpos, board, boardsize):

            count += 1
            board[newpos] = count

            if knight_tour(newpos, board, count):
                return True
            else:
                board[newpos] = 0
                count -= 1
    return False


if __name__ == '__main__':
    n = int(raw_input('n: '))
    boardsize = (n, n)
    start = chess2index(raw_input('Start: '), boardsize)
    offsets = ((1, 2), (-1, 2), (1, -2), (-1, -2),
               (2, 1), (-2, 1), (2, -1), (-2, -1))
    board = {(x, y):0 for x in range(boardsize[X]) for y in range(boardsize[Y])}
    count = 1
    board[start] = count
    if knight_tour(start, board, count):
        print board_string(board, boardsize)
    else:
        print "No knight's tour"
{% endhighlight %}

Most of the magic happens in the `knight-tour` function in which, the search starts from the initial position on the board (`count=1`). From the current position, possible legal knight moves are generated and tried out by incrementing the count and marking it on the board. If a move turns out to be leading a dead end, reset the board position and the count will be decremented and it would reach the end of the function (`return False`) and exit from the a recursion instance. Eventually, a knight's tour will be found (`count == total`) provided that there is one.

This approach works reasonably well for small boards but as the size of the board grows _(`N >= 7` onwards)_, the search will take exponentially longer as there would be more "center" pieces which have 8 possible moves.

## Warnsdorff's rule
In order to improve from that, Warnsdorff's rule can be applied. [Warnsdorff's rule](https://en.wikipedia.org/wiki/Knight%27s_tour#Warnsdorff.27s_algorithm) is a heuristic which states that in each step of the tour, the square with the least possible moves the knight can make from that square is favored. If a tie occurs, it may be broken arbitrarily.

{% highlight python %}
def next_possible_moves(current, board, boardsize):
    next_moves = [(current[X]+d[X], current[Y]+d[Y]) for d in offsets]
    next_moves = [p for p in next_moves if is_legal(p, board, boardsize)]
    return next_moves

def warnsdorff(current, board, boardsize):
    moves = []
    for move in next_possible_moves(current, board, boardsize):
        moves.append((move, len(next_possible_moves(move, board, boardsize))))
    return sorted(moves, key=lambda x:x[1])

def knight_tour(start, board, count):
    current = start
    while count < len(board):
        newpos = warnsdorff(current, board, boardsize)
        if newpos:
            count += 1
            board[newpos[0][0]] = count
            current = newpos[0][0]
        else:
            return False
    return True
{% endhighlight %}

The idea is to have a list of possible moves (`next_possible_moves()`) and calculate and store the number of possible moves from each of those moves (`warnsdorff()`) and choose the move with fewest onward moves. Reaching a dead-end means, there are no possible moves returning from the `warnsdorff` function.

{% highlight python %}
X = 0
Y = 1

def chess2index(p, boardsize):
    x = boardsize[X] - int(p[1])
    y = ord(p[0]) - ord('a')
    return x, y

def is_legal(pos, board, boardsize):
    return 0 <= (pos[X]) < boardsize[X] and \
           0 <= (pos[Y]) < boardsize[Y] and \
           not board[pos]

def board_string(board, boardsize):
    string = ''
    for x in range(boardsize[X]):
        for y in range(boardsize[Y]):
            string += '%3d' % board[(x, y)]
        string += '\n'
    return string

def next_possible_moves(current, board, boardsize):
    next_moves = [(current[X]+d[X], current[Y]+d[Y]) for d in offsets]
    next_moves = [p for p in next_moves if is_legal(p, board, boardsize)]
    return next_moves

def warnsdorff(current, board, boardsize):
    moves = []
    for move in next_possible_moves(current, board, boardsize):
        moves.append((move, len(next_possible_moves(move, board, boardsize))))
    return sorted(moves, key=lambda x:x[1])

def knight_tour(start, board, count):
    current = start
    while count < len(board):
        newpos = warnsdorff(current, board, boardsize)
        if newpos:
            count += 1
            board[newpos[0][0]] = count
            current = newpos[0][0]
        else:
            return False
    return True

if __name__ == '__main__':
    n = int(raw_input('N: '))
    boardsize = (n, n)
    start = chess2index(raw_input('Start: '), boardsize)
    offsets = ((1, 2), (-1, 2), (1, -2), (-1, -2),
               (2, 1), (-2, 1), (2, -1), (-2, -1))
    board = {(x, y):0 for x in range(boardsize[X]) for y in range(boardsize[Y])}
    count = 1
    board[start] = count

    if knight_tour(start, board, count):
        print board_string(board, boardsize)
    else:
        print "No knight's tour"

{% endhighlight %}

## Output

{% highlight bash %}
> python knights-tour.py
N: 8
Start: a5
 44 17 20  3 26 41 22  5
 19  2 43 46 21  4 25 40
 16 45 18 27 42 51  6 23
  1 28 57 54 47 24 39 52
 56 15 48 35 58 53 50  7
 29 12 55 64 49 34 59 38
 14 63 10 31 36 61  8 33
 11 30 13 62  9 32 37 60
{% endhighlight %}

{% highlight bash %}
> python knights-tour.py
N: 10
Start: e2
   11   92   13   86    9   80   41   82    7   76
   14   85   10   79   42   83    8   77   40   71
   91   12   93   84   87   78   81   72   75    6
   96   15   90   43   94   73   88   65   70   39
   57   44   95  100   89   64   69   74    5   62
   16   97   58   55   68   99   66   63   38   25
   45   56   33   98   59   54   37   24   61    4
   32   17   46   49   36   67   60   53   26   23
   47   34   19   30    1   50   21   28    3   52
   18   31   48   35   20   29    2   51   22   27
{% endhighlight %}

This approach works well for really big boards as well. A Knight's Tour can be found in a few seconds during my testing with big boards. _(`N=200,300,400`)_

As you can see, I rely on Python's `sorted()` function and didn't do anything special for ties. However, as far as I have researched, there are [some approaches](http://dl.acm.org/citation.cfm?id=363463) to break the ties in more specific manners instead of choosing arbitrarily.