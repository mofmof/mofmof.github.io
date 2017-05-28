---
layout: blog
title: 強化学習(Q-Learning)でオセロAIを学習させてみた
category: blog
tags: [機械学習,machine learning,Python,強化学習,Reinforcement Learning,reversi,othello]
summary: というわけで、前回強化学習(Q-Learning)で四目並べを学習させてみたというのをやってうまくいかなかったので、別のゲームで
author: aharada
---

年2回、毎度楽しみにしている「開発合宿友の会」の開発合宿に来ております。やはりコード書くのはこの上ない幸せですね。

というわけで、前回[強化学習(Q-Learning)で四目並べを学習させてみた](http://tech.mof-mof.co.jp/blog/connect-four.html)というのをやってうまくいかなかったので、別のゲームで実装し直して学習させたらうまくいくかもしれないと思いまして、今回は盤面4x4のオセロのAIを作ってみます。

ソースコードはGitHubにあった[オセロの実装](https://github.com/feicun/Reversi)をベースに、前回の四目並べのソースコードを組み合わせて実装しました。

全てGitHubにアップしてあります。

[https://github.com/harada4atsushi/reversi](https://github.com/harada4atsushi/reversi)

## オセロ実装

盤面の実装。ベースのコードはオブジェクト志向的じゃなく、関数がたくさん実装されているだけだったので、盤面クラスとして切り出した。

が、随時個別に切り出していたので、だいぶ中途半端だと思う。

`board.py`

```
from gameplay import valid


class Board:
    def __init__(self, board_data):
        self.board_data = board_data

    def print(self):
        """ Print a board, with letters and numbers as guides """
        print("  " + " ".join(map(chr, range(ord('A'), ord('D') + 1))))
        for (x,y) in zip(range(1,9), self.board_data):
            print(str(x) + ' ' + self.to_circle(" ".join(y)))
        print("Black = %d, White = %d" % self.score())

        # For fun, here is a one-line board printer, without the
        # row and column labels
        # print "\n".join(["".join(x) for x in board])

    def to_circle(self, str):
        return str.replace('W', '◯').replace('B', '●')


    def score(self):
        """ returns the current score for the board as a tuple
            containing # of black pieces, # of white pieces """
        black = white = 0
        for row in self.board_data:
            for square in row:
                if (square == "B"):
                    black = black + 1
                elif (square == "W"):
                    white = white + 1
        return (black, white)


    def valid_positions(self, player):
        moves = []
        for i in range(4):
            for j in range(4):
                if valid(self.board_data, player.color, (i, j)):
                    moves.append((i, j))
        return moves


    def flattend_data(self):
        return [flatten for inner in self.board_data for flatten in inner]


    def color_of_more(self):
        res = self.score()

        if res[0] > res[1]:
            return 'B'
        elif res[0] < res[1]:
            return 'W'
        else:
            return ''
```

ゲーム進行役の実装。こちらも`board.py`と同様、ちょっと中途半端にクラスに切り出したところ。

`organizer.py`

```
import random
import time
from copy import deepcopy

from board import Board
from gameplay import newBoard, valid, doMove


class Organizer:
    def __init__(self, show_board=True, show_result=True, nplay=1, stat=100):
        self._show_board = show_board
        self._show_result = show_result
        self._nplay = nplay
        self._stat = stat


    def play_game(self, player1, player2, verbose=False, t=128):
        """ Takes as input two functions p1 and p2 (each of which
            calculates a next move given a board and player color),
            and returns either a tuple containing the score for black,
            score for white, and final board (for a normal game ending)
            or a tuple containing the final score for black, final score
            for white, and the  invalid move (for a game that ends with
            and invalid move """

        # p1 = player1.nextMove
        # p2 = player2.nextMove

        p1 = player1
        p2 = player2

        player1_win_count = 0
        player2_win_count = 0
        draw_count = 0

        for i in range(0, self._nplay):
            (p2, p1) = (p1, p2)

            board = Board(newBoard())
            p1time = t
            p2time = t
            p1realTime = t * 2  # gives a little extra time to each player
            p2realTime = t * 2

            while not self.game_over(board.board_data):
                tmpBoard = deepcopy(board.board_data)
                t1 = time.time()
                nextMove = p1.nextMove(tmpBoard, p1.color, p1time)
                t2 = time.time()
                p1time = p1time - (t2 - t1)
                p1realTime = p1realTime - (t2 - t1)
                # if p1time < 0:
                if (p1realTime < 0):
                    if p1.color == "B":
                        return (0, 16, board.board_data, "Timeout")
                    else:
                        return (16, 0, board.board_data, "Timeout")
                if valid(board.board_data, p1.color, nextMove):
                    doMove(board.board_data, p1.color, nextMove)
                else:
                    if p1.color == "B":
                        return (0, 16, board.board_data, "Bad Move: %s" % str(nextMove))
                    else:
                        return (16, 0, board.board_data, "Bad Move: %s" % str(nextMove))

                # p1.getGameResult(board.board_data, game_ended=self.game_over(board.board_data))
                p1.getGameResult(board.board_data)

                (p1, p2) = (p2, p1)
                (p1time, p2time) = (p2time, p1time)
                (p1realTime, p2realTime) = (p2realTime, p1realTime)

            # res = score(board) + (board,)


            if self._show_board:
                board.print()

            res = board.score()

            if res[0] > res[1]:
                player1_win_count += 1
            elif res[0] < res[1]:
                player2_win_count += 1
            else:
                draw_count += 1

            if self._nplay > 1 and i % self._stat == 0:
                print("Win count, player1(%s): %d, player2(%s): %d, draw: %d" % (player1.name, player1_win_count, player2.name, player2_win_count, draw_count))

        if self._nplay > 1:
            print("Win count, player1(%s): %d, player2(%s): %d, draw: %d" % (
                player1.name, player1_win_count, player2.name, player2_win_count, draw_count))

        # print(player1._q)


    def print_winner(self, winner, loser, winner_count, loser_count):
        if self._show_result:
            if winner_count == loser_count:
                print("TIE %s, %s, (%d to %d)" % (winner, loser, winner_count, loser_count))
            else:
                print("%s Wins %s Loses (%d to %d)" % (winner, loser, winner_count, loser_count))


    def game_over(self, board_data):
        """ return true if the game is over, that is, no valid moves """
        return valid(board_data, "B", 'pass') and valid(board_data, "W", 'pass')
```

プレーヤーは3種類実装した。相変わらず置けるところに適当に置くだけのランダムさん。

`random_player.py`

```
import random
from gameplay import valid

class RandomPlayer:

    def __init__(self, color):
        self.color = color
        self.name = 'random'

    def nextMove(self, board, color, time):
        moves = []
        for i in range(4):
            for j in range(4):
                if valid(board, color, (i,j)):
                    moves.append((i,j))
        if len(moves) == 0:
            return "pass"
        bestMove = moves[random.randint(0,len(moves) - 1)]
        return bestMove


    def nextMoveR(self, board, color, time):
        return self.nextMove(board, color, time)


    def getGameResult(self, board_data, game_ended=False):
        pass
```

続いて、オセロAIの実装の鉄板、竜王ミニマックスさん。実装内容はあまり深掘りしていないのですが、アルファベータさんかも知れない。

試行錯誤途中なので、けっこういらないコード混ざってるかもね。

`minmax_player.py`

```


from copy import deepcopy

import gameplay
from board import Board


class MinmaxPlayer:
    INF = float('inf')

    def __init__(self, color):
        self.color = color
        self.name = 'minmax'


    def nextMove(self, board, color, time):
        if gameplay.valid(board, color, 'pass'):
            return "pass"
        depth = 3
        # if time <= 170 and time > 100:
        #     depth = 5
        # if time <= 100 and time > 50:
        #     depth = 4
        # if time <= 50 and time > 20:
        #     depth = 3
        # if time <= 20 and time > 5:
        #     depth = 2
        # if time <= 5:
        #     depth = 1

        (move, value) = self.max_val(board, -self.INF, self.INF, depth, color)
        return move



    def get_valid_positions(self):
        moves = []
        for i in range(4):
            for j in range(4):
                if gameplay.valid(self._board, self._color, (i, j)):
                    moves.append((i, j))
        return moves


    def get_successors_list(self, positions):
        if self._color == 'B':
            idx = 0
        elif self._color == 'W':
            idx = 1

        successors_list = []
        before_score = gameplay.score(self._board)

        for position in positions:
            newBoard = deepcopy(self._board)
            gameplay.doMove(newBoard, self._color, position)
            after_score = gameplay.score(newBoard)
            gain = after_score[idx] - before_score[idx] - 1
            successors_list.append((position, newBoard, gain))

            # Board().print(newBoard)
        return successors_list


    def get_best_positions(self, successors_list):
        max_gain = 0
        best_positions = []

        for (move, state, gain) in successors_list:
            if max_gain < gain:
                max_gain = gain

        for (move, state, gain) in successors_list:
            if gain == max_gain:
                best_positions.append(move)

        return best_positions


    ### state in this program is always a "board"
    ### Check if the state is the end
    def endState(self, state):
        # TODO DRYにしたい
        return gameplay.valid(state, "B", 'pass') and gameplay.valid(state, "W", 'pass')


    def max_val(self, state, alpha, beta, depth, color, reversed=False):
        if self.endState(state):
            return None, self.utility(state, color)
        elif depth == 0:
            return None, self.evaluation(state, color)
        best = None
        v = -self.INF
        if not reversed:
            moves = self.successors(state, color)
        else:
            moves = self.successors(state, gameplay.opponent(color))
        for (move, state) in moves:
            value = self.min_val(state, alpha, beta, depth - 1, color, reversed)[1]
            if best is None or value > v:
                best = move
                v = value
            if v >= beta:
                return best, v
            alpha = max(alpha, v)
        return best, v



    def min_val(self, state, alpha, beta, depth, color, reversed=False):
        if self.endState(state):
            return None, self.utility(state, color)
        elif depth == 0:
            return None, self.evaluation(state, color)
        best = None
        v = self.INF
        if not reversed:
            moves = self.successors(state, gameplay.opponent(color))
        else:
            moves = self.successors(state, color)
        for (move, state) in moves:
            value = self.max_val(state, alpha, beta, depth - 1, color, reversed)[1]
            if best is None or value < v:
                best = move
                v = value
            if alpha >= v:
                return best, v
            beta = min(beta, v)
        return best, v


    ### Generate all the possible moves and
    ### the new state related with the move
    def successors(self, state, color):
        successors_list = []
        moves = []
        for i in range(4):
            for j in range(4):
                if gameplay.valid(state, color, (i, j)):
                    moves.append((i, j))
        for moves in moves:
            newBoard = deepcopy(state)
            gameplay.doMove(newBoard, color, moves)
            successors_list.append((moves, newBoard))
        return successors_list



    def utility(self, state, color):
        board = Board(state)
        answer = 0
        if board.score()[0] == board.score()[1]:
            answer = 0
        elif board.score()[0] < board.score()[1] and color == "W":
            answer = self.INF
        elif board.score()[0] > board.score()[1] and color == "B":
            answer = self.INF
        else:
            answer = -self.INF
        return answer


    ### Implementation of Positional Strategy
    def evaluation(self, state, color):
        result = 0
        # 8x8
        # weight = [[99,-8,8,6,6,8,-8,99],[-8,-24,-4,-3,-3,-4,-24,-8],
        # [8,-4,7,4,4,7,-4,8],[6,-3,4,0,0,4,-3,6],
        # [6,-3,4,0,0,4,-3,6],[8,-4,7,4,4,7,-4,8],
        # [-8,-24,-4,-3,-3,-4,-24,-8],[99,-8,8,6,6,8,-8,99]]

        # 4x4
        weight = [
            [2, 1, 1, 2],
            [1, 1, 1, 1],
            [1, 1, 1, 1],
            [2, 1, 1, 2],
        ]

        for i in range(4):
            for j in range(4):
                if state[i][j] == color:
                    result += weight[i][j]
                if state[i][j] == gameplay.opponent(color):
                    result -= weight[i][j]

        #if reversed:
        #    result = -result

        return result


    def getGameResult(self, board_data, game_ended=False):
        pass
```

最後に、今回の主役のQ学習さん。ポテンシャルは高いが教育が必要な方。

`qlearning_player.py`

```
import random
from copy import deepcopy

import gameplay
from board import Board
from player.random_player import RandomPlayer


class QlearningPlayer:
    INF = float('inf')
    DEFAULT_E = 0.2
    INITIAL_Q = 1  # default 1

    def __init__(self, color, e=DEFAULT_E, alpha=0.3):
        self.color = color
        self.name = 'ql'
        self._e = e
        self._alpha = alpha
        self._gamma = 0.9
        self._total_game_count = 0
        self._q = {}
        self._last_board = None
        self._last_move = None


    def nextMove(self, board, color, _):
        return self.policy(board, color)


    def policy(self, board_data, color):
        self._last_board = Board(board_data.copy())
        board = Board(board_data)
        positions = board.valid_positions(self)

        if len(positions) == 0:
            return "pass"

        # for debug
        # if self._total_game_count > 1000 and self._total_game_count % 100 == 0:
        #     print(self._total_game_count)

        # ゲーム回数が少ない間は、ある程度の確率で打ち手をランダムにする
        if random.random() < (self._e / (self._total_game_count // 10000 + 1)):
            move = random.choice(positions)
        else:
            qs = []
            for position in positions:
                qs.append(self.get_q(tuple(self._last_board.flattend_data()), position))

            # for debug
            # if self._total_game_count > 9000 and self._total_game_count % 300 == 0:
            #     # print(positions)
            #     print(qs)

            max_q = max(qs)

            if qs.count(max_q) > 1:
                # more than 1 best option; choose among them randomly
                best_options = [i for i in range(len(positions)) if qs[i] == max_q]
                i = random.choice(best_options)
            else:
                i = qs.index(max_q)
            move = positions[i]

        self._last_move = move
        return move


    def get_q(self, state, act):
        # encourage exploration; "optimistic" 1.0 initial values
        if self._q.get((state, act)) is None:
            self._q[(state, act)] = self.INITIAL_Q
        return self._q.get((state, act))


    # def getGameResult(self, board_data, game_ended=False):
    def getGameResult(self, board_data):
        board = Board(board_data[:])

        # 相手のターン行動後のQ値を取得するための処理
        tmp_player = RandomPlayer(gameplay.opponent(self.color))
        vp = board.valid_positions(tmp_player)
        if len(vp) != 0:
            gameplay.doMove(board.board_data, gameplay.opponent(self.color), random.choice(vp))
        game_ended = gameplay.valid(board_data, "B", 'pass') and gameplay.valid(board_data, "W", 'pass')

        reward = 0
        if game_ended:
            color = board.color_of_more()

            if color == self.color:
                reward = 1
            elif color == '':
                reward = 0
            elif color != self.color:
                reward = -1

        self.learn(self._last_board, self._last_move, reward, board, game_ended)

        if not game_ended:
            self._total_game_count += 1
            self._last_move = None
            self._last_board = None


    def learn(self, s, a, r, fs, game_ended):

        flattend_data = s.flattend_data()
        pQ = self.get_q(tuple(flattend_data), a)

        # 相手のターン行動後のQ値を取得するための処理
        # tmp_player = RandomPlayer(gameplay.opponent(self.color))
        # vp = fs.valid_positions(tmp_player)
        # if len(vp) != 0:
        #     gameplay.doMove(fs.board_data, gameplay.opponent(self.color), random.choice(vp))

        list = []
        for position in fs.valid_positions(self):
            list.append(self.get_q(tuple(fs.flattend_data()), position))

        # print(list)

        if game_ended or len(list) == 0:
            max_q_new = 0
        else:
            max_q_new = max(list)
        self._q[(tuple(flattend_data), a)] = pQ + self._alpha * ((r + self._gamma * max_q_new) - pQ)


    def change_to_battle_mode(self):
        self._e = 0
```

実行するコード。引数にプレーヤーの名前を渡します。

`gameplay.py`

```
import argparse


def opponent(x):
    """ Given a string representing a color (must be either "B" or "W"),
        return the opposing color """
    if x == "b" or x == "B" or x == '●':
        return "W"
    elif x == "w" or x == "W" or x == '◯':
        return "B"
    else:
        return "."


def validMove(board, color, pos):
    """ Given a 2D array representing a board, a string
        representing a color, and a tuple representing a
        position, return true if the position is a valid
        move for the color """
    if board[pos[0]][pos[1]] != '.':
        return False
    for i in range(-1, 2):
        for j in range(-1, 2):
            if i != 0 or j != 0:
                if canFlip(board, color, pos, (i,j)):
                    return True
    return False

def valid(board, color, move):
    """ Given a 2D array representing a board, a string
        representing a color, and either a tuple representing a
        position or the string "pass", return true if the move
        is a valid for the color """
    if move == "pass":
        for i in range(0,4):
            for j in range(0,4):
                if validMove(board, color, (i,j)):
                    return False
        return True
    else:
        return validMove(board, color, move)


def validPos(x,y):
    """ Return true of the (x,y) position is within the board """
    return x >= 0 and x < 4 and y >= 0 and y < 4

def doFlip(board, color, pos, direction):
    """ Given a 2D array representing a board, a color, a position
        to move to, and a tuple representing a direction ( (-1,0)
        for up, (-1,1) for up and to the right, (0,1) for to the right,
        and so on), flip all the pieces in the direction until a
        piece of the same color is found """
    currX = pos[0] + direction[0]
    currY = pos[1] + direction[1]
    while board[currX][currY] == opponent(color):
        board[currX][currY] = color
        (currX, currY) = (currX + direction[0], currY + direction[1])        



def doMove(board, color, pos):
    """ Given a 2D array representing a board, a color, and a position,
        implement the move on the board.  Note that the move is assumed
        to be valid """
    if pos != "pass":
        if validMove(board, color, pos):
            board[pos[0]][pos[1]] = color
            for i in range(-1, 2):
                for j in range(-1, 2):
                    if i != 0 or j != 0:
                        if canFlip(board, color, pos, (i,j)):
                            doFlip(board, color, pos, (i,j))


def canFlip(board, color, pos, direction):
    """ Given a 2D array representing a board, a color, a position
        to move to, and a tuple representing a direction ( (-1,0)
        for up, (-1,1) for up and to the right, (0,1) for to the right,
        and so on), determine if there is a sequence of opponent pieces,
        followed by a color piece, that would allow a flip in this direction
        from this position, if a color piece is placed at pos """
    currX = pos[0] + direction[0]
    currY = pos[1] + direction[1]
    if not validPos(currX,currY):
        return False
    if board[currX][currY] != opponent(color):
        return False
    while True:
        (currX, currY) = (currX + direction[0], currY + direction[1])
        if not validPos(currX, currY):
            return False
        if board[currX][currY] == color:
            return True
        if board[currX][currY] == '.':
            return False


def newBoard():
    """ Create a new board:  2D array of strings:
        'B' for black, 'W' for white, and '.' for empty """
    result = []
    for i in range(1):
        result = result + [["."]*4]
    result = result + [["."] * 1 + ["W","B"] + ["."] * 1]
    result = result + [["."] * 1 + ["B","W"] + ["."] * 1]
    for i in range(1):
        result = result + [["."]*4]
    return result


def get_player_instance(player_str, color):
    from player.random_player import RandomPlayer
    from player.minmax_player import MinmaxPlayer
    from player.qlearning_player import QlearningPlayer

    if player_str == 'random':
        return RandomPlayer(color)
    elif player_str == 'minmax':
        return MinmaxPlayer(color)
    elif player_str == 'ql':
        return QlearningPlayer(color)
    else:
        return RandomPlayer(color)



##options: -r is reversed game; -v is verbose (print board & time after each move)
##  -t is a time to play (other than the default)
if __name__ == "__main__":
    verbose = False
    clockTime = 320.0
    reversed = ""

    parser = argparse.ArgumentParser()
    parser.add_argument('--p1', default='ql', type=str)
    parser.add_argument('--p2', default='random', type=str)
    args = parser.parse_args()

    p1 = get_player_instance(args.p1, 'B')
    p2 = get_player_instance(args.p2, 'W')

    from organizer import Organizer
    organizer = Organizer(nplay=1000, show_board=False, show_result=False)
    # organizer = Organizer()
    organizer.play_game(p1, p2, verbose, clockTime)
```

## ランダム vs ランダム

今回の目指すゴールは、Q学習さんを鍛えて、竜王ミニマックスさんと「対等」な勝負をすること。「対等」と言ったのは、4x4オセロでは最善の手を打ち続けたら必ず後攻が勝利するようになっているので、先行後攻を50:50で振り分けたとして、最強同士の戦いをすると、常に勝利数は同等になるためです。

まずはランダムさん同士で1,000回程戦わせてみます。

先行後攻を毎回入れ替えた上でのランダム同士の戦いなのでほぼ互角です。

```
$ python gameplay.py --p1 random --p2 random
Win count, player1(random): 0, player2(random): 1, draw: 0
Win count, player1(random): 46, player2(random): 49, draw: 6
Win count, player1(random): 91, player2(random): 94, draw: 16
Win count, player1(random): 126, player2(random): 148, draw: 27
Win count, player1(random): 168, player2(random): 196, draw: 37
Win count, player1(random): 215, player2(random): 242, draw: 44
Win count, player1(random): 262, player2(random): 282, draw: 57
Win count, player1(random): 306, player2(random): 326, draw: 69
Win count, player1(random): 353, player2(random): 368, draw: 80
Win count, player1(random): 398, player2(random): 413, draw: 90
Win count, player1(random): 446, player2(random): 458, draw: 96
```

## 竜王ミニマックス vs ランダム

竜王ミニマックスさんでランダムさんを圧倒してやりましょう。こちらも1,000回程戦わせてみます。

当然竜王ミニマックスさんの圧勝です。

```
$ python gameplay.py --p1 minmax --p2 random
Win count, player1(minmax): 1, player2(random): 0, draw: 0
Win count, player1(minmax): 91, player2(random): 7, draw: 3
Win count, player1(minmax): 176, player2(random): 13, draw: 12
Win count, player1(minmax): 268, player2(random): 19, draw: 14
Win count, player1(minmax): 356, player2(random): 29, draw: 16
Win count, player1(minmax): 444, player2(random): 38, draw: 19
Win count, player1(minmax): 533, player2(random): 45, draw: 23
Win count, player1(minmax): 620, player2(random): 56, draw: 25
Win count, player1(minmax): 711, player2(random): 61, draw: 29
Win count, player1(minmax): 799, player2(random): 65, draw: 37
Win count, player1(minmax): 886, player2(random): 74, draw: 40
```

## Q学習 vs 竜王ミニマックス

いよいよ大本命Q学習さんと竜王ミニマックスさんをぶつけてみます。Q学習さんは10万回くらい戦わせて育てておきます。

`gameplay.py`の該当部分のコードを少し修正します。`change_to_battle_mode`で`ε-greedy`の`e`の値を0にすることで、Q学習の探索性をストップし、最善手で戦うようにします。

```
    from organizer import Organizer
    organizer = Organizer(nplay=100000, show_board=False, show_result=False)
    # organizer = Organizer()
    organizer.play_game(p1, p2, verbose, clockTime)

    # 学習済みのQ-Learningプレイヤーで再度戦わせる
    organizer = Organizer(nplay=300, show_board=False, show_result=False)
    p1.change_to_battle_mode()
    from player.minmax_player import MinmaxPlayer
    p2 = MinmaxPlayer('W')
    organizer.play_game(p1, p2, verbose, clock
```

実行。

竜王ミニマックスさんにフルボッコにされて負けました。

```
$ python gameplay.py --p1 ql --p2 ql

...省略

Win count, player1(ql): 43562, player2(ql): 43230, draw: 9509
Win count, player1(ql): 43612, player2(ql): 43274, draw: 9515
Win count, player1(ql): 43650, player2(ql): 43324, draw: 9527
Win count, player1(ql): 43699, player2(ql): 43369, draw: 9533
Win count, player1(ql): 43738, player2(ql): 43423, draw: 9540
Win count, player1(ql): 43786, player2(ql): 43467, draw: 9548
Win count, player1(ql): 43827, player2(ql): 43512, draw: 9562
Win count, player1(ql): 43875, player2(ql): 43558, draw: 9568
Win count, player1(ql): 43919, player2(ql): 43599, draw: 9583
Win count, player1(ql): 43959, player2(ql): 43647, draw: 9595
Win count, player1(ql): 44002, player2(ql): 43690, draw: 9609
Win count, player1(ql): 44042, player2(ql): 43741, draw: 9618
Win count, player1(ql): 44083, player2(ql): 43791, draw: 9627
Win count, player1(ql): 44129, player2(ql): 43836, draw: 9636
Win count, player1(ql): 44183, player2(ql): 43875, draw: 9643
Win count, player1(ql): 44228, player2(ql): 43920, draw: 9653
Win count, player1(ql): 44280, player2(ql): 43963, draw: 9658
Win count, player1(ql): 44327, player2(ql): 44007, draw: 9667
Win count, player1(ql): 44376, player2(ql): 44049, draw: 9676
Win count, player1(ql): 44417, player2(ql): 44098, draw: 9686
Win count, player1(ql): 44463, player2(ql): 44139, draw: 9699
Win count, player1(ql): 44511, player2(ql): 44180, draw: 9710
Win count, player1(ql): 44551, player2(ql): 44228, draw: 9722
Win count, player1(ql): 44597, player2(ql): 44274, draw: 9730
Win count, player1(ql): 44640, player2(ql): 44323, draw: 9738
Win count, player1(ql): 44681, player2(ql): 44370, draw: 9750
Win count, player1(ql): 44719, player2(ql): 44421, draw: 9761
Win count, player1(ql): 44771, player2(ql): 44460, draw: 9770
Win count, player1(ql): 44811, player2(ql): 44509, draw: 9781
Win count, player1(ql): 44854, player2(ql): 44556, draw: 9791
Win count, player1(ql): 44900, player2(ql): 44604, draw: 9797
Win count, player1(ql): 44942, player2(ql): 44653, draw: 9806
Win count, player1(ql): 44988, player2(ql): 44700, draw: 9813
Win count, player1(ql): 45034, player2(ql): 44746, draw: 9821
Win count, player1(ql): 45073, player2(ql): 44799, draw: 9829
Win count, player1(ql): 45116, player2(ql): 44849, draw: 9836
Win count, player1(ql): 45156, player2(ql): 44893, draw: 9852
Win count, player1(ql): 45204, player2(ql): 44934, draw: 9862

Win count, player1(ql): 0, player2(minmax): 1, draw: 0
Win count, player1(ql): 24, player2(minmax): 73, draw: 4
Win count, player1(ql): 55, player2(minmax): 139, draw: 7
Win count, player1(ql): 85, player2(minmax): 205, draw: 10
```

## まとめ
学習が収束していないというよりは、Q学習のどこかにバグがあるような気がしますね。ずっとデバッグしていたのですが、なかなか解決できず。

前回に引き続きQ学習さんが強くならないという結果でした。ざんぬん。
