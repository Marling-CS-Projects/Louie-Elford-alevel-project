# 2.2.1.1 console based chess game side project - part 1

## Design

### Objectives

create a functioning game of chess using the unicode characters for chess pieces to display the pieces in the console, the game should have as features:

* [x] interface on the console by means of a chess board constructed with unicode characters
* [x] the chess board should update after every move with the new boardstate
* [x] all basic chess rules for each pieces movement should be implemented
* [x] en passant
* [x] castling
* [x] king should not be able to move into check
* [x] a message should display when the player attempts to make an illegal move and the move should be ignored and the boardstate resent to the console.
* [x] skeleton for turn tracking so that white and black pieces can only be moved on their respective turns.

### Usability Features

### Key Variables

| Variable Name | Use                                                                                               |
| ------------- | ------------------------------------------------------------------------------------------------- |
| board         | an 8 by 8 array containing references to the unicode character on each tile as a string constant. |

### Pseudocode

I didn't really have much of an outlook of what the code would look like as I was messing around with different ideas.

## Development

### Outcome

here's all the code currently as one massive oversized blob, it's all in one file and only currently runs in repl.

```
#include <iostream>
using namespace std;

class ChessBoard {
public:
  string KB = "\u2654"; // white king
  string QB = "\u2655"; // white queen
  string RB = "\u2656"; // white rook
  string BB = "\u2657"; // white bishop
  string NB = "\u2658"; // white knight
  string PB = "\u2659"; // white pawn
  string KW = "\u265A"; // black king
  string QW = "\u265B"; // black queen
  string RW = "\u265C"; // black rook
  string BW = "\u265D"; // black bishop
  string NW = "\u265E"; // black knight
  string PW = "\u265F"; // black pawn
  string EE = "\u25A1"; // empty space
  string board[8][8] = {
    {RW, NW, BW, QW, KW, BW, NW, RW}, // standard chess board, playing as white
    {PW, PW, PW, PW, PW, PW, PW, PW}, 
    {EE, EE, EE, EE, EE, EE, EE, EE},
    {EE, EE, EE, EE, EE, EE, EE, EE}, 
    {EE, EE, EE, EE, EE, EE, EE, EE},
    {EE, EE, EE, EE, EE, EE, EE, EE}, 
    {PB, PB, PB, PB, PB, PB, PB, PB},
    {RB, NB, BB, QB, KB, BB, NB, RB}
  };

  //special rules
  int turn;
  int enPassant;
  bool delay;
  bool WhiteCanCastleLeft;
  bool WhiteCanCastleRight;
  bool castleWleft;
  bool castleWright;
  bool BlackCanCastleLeft;
  bool BlackCanCastleRight;
  bool castleBleft;
  bool castleBright;
  int kingWPos[2];
  int kingBPos[2];

  // prints the current boardstate to console
  void printBoard() {
    for (int i = 7; i > -1; i--) {
      for (int j = 0; j < 8; j++) {
        cout << board[i][j] << " ";
      }
      cout << "\n";
    }
  }

  // checks whether input move is legal (could use some optimization)
  bool legalMoveCheck(int x1, int y1, int x2, int y2) {

    
    if (x1 > 7 || x1 < 0 || x2 > 7 || x2 < 0 || y1 > 7 || y1 < 0 || y2 > 7 || y2 < 0) return false;
    if (x1 == x2 && y1 == y2) return false;

    string piece = board[y1][x1];
    if (piece == EE) return false;
    //white moves
    if (turn % 2 == 1) {

      if (piece == KW) {
        cout << WhiteCanCastleRight;
        if (checkcheck(x2,y2,true) == true) {
          cout << "pingus";
          return false;
        }
        //castling rules
        cout << "ping";
        if (x1 == 4 && y1 == 0 && y2 == 0) { 
          cout << "ping1";
          if (x2 == 2 && WhiteCanCastleRight == true) {
            if (checkcheck(4,0,true) == false && checkcheck(3,0,true) == false) {
              if (board[0][1] == EE && board[0][2] == EE && board[0][3] == EE) {
                castleWleft = true;
                return true;
              }
            }
            return false;
          }
          if (x2 == 6 && WhiteCanCastleRight == true) {
            cout << "ping2";
            if (checkcheck(4,0,true) == false && checkcheck(5,0,true) == false) {
              cout << "ping3";
              if (board[0][6] == EE && board[0][5] == EE) {
                castleWright = true;
                return true;
              }
            }
            return false;
          }
        }
        if (colorcheck(x2, y2) == 1)
          return false;
        if ((x1 == x2 && abs(y2 - y1) == 1) || (y1 == y2 && abs(x2 - x1) == 1))
          return true;
        if (abs(x2 - x1) == abs(y2 - y1) && abs(x2 - x1) == 1)
          return true;
        return false;
      }
        
      if (piece == QW) {
          if (colorcheck(x2, y2) == 1)
            return false;
          if (x1 == x2) {
            if (y2 > y1 + 1) {
              for (int i = y1 + 1; i < y2; i++) {
                if (colorcheck(x1, i) != 0)
                  return false;
              }
              return true;
            }
            if (y2 < y1 - 1) {
              for (int i = y1 - 1; i > y2; i--) {
                if (colorcheck(x1, i) != 0)
                  return false;
              }
              return true;
            }
            return true;
          }
          if (y1 == y2) {
            if (x2 > x1 + 1) {
              for (int i = x1 + 1; i < x2; i++) {
                if (colorcheck(i, y1) != 0)
                  return false;
              }
              return true;
            }
            if (x2 < x1 - 1) {
              for (int i = x1 - 1; i > x2; i--) {
                if (colorcheck(i, y1) != 0)
                  return false;
              }
              return true;
            }
            return true;
          }
          if (abs(x2 - x1) == abs(y2 - y1)) {
            if (y2 > y1 + 1 && x2 > x1 + 1) {
              for (int i = 1; i < (x2 - x1); i++) {
                if (colorcheck(x1 + i, y1 + i) != 0)
                  return false;
              }
              return true;
            }
            if (y2 > y1 + 1 && x2 < x1 - 1) {
              for (int i = 1; i < y2 - y1; i++) {
                if (colorcheck(x1 - i, y1 + i) != 0)
                  return false;
              }
              return true;
            }
            if (y2 < y1 - 1 && x2 > x1 + 1) {
              for (int i = 1; i < x2 - x1; i++) {
                if (colorcheck(x1 + i, y1 - i) != 0)
                  return false;
              }
              return true;
            }
            if (y2 < y1 - 1 && x2 < x1 - 1) {
              for (int i = 1; i < x1 - x2; i++) {
                if (colorcheck(x1 - i, y1 - i) != 0)
                  return false;
              }
              return true;
            }
            return true;
          }
          return false;
        }
      
      if (piece == RW) {
          if (colorcheck(x2, y2) == 1)
            return false;
          if (x1 == x2) {
            if (y2 > y1 + 1) {
              for (int i = y1 + 1; i < y2; i++) {
                if (colorcheck(x1, i) != 0)
                  return false;
              }
              return true;
            }
            if (y2 < y1 - 1) {
              for (int i = y1 - 1; i > y2; i--) {
                if (colorcheck(x1, i) != 0)
                  return false;
              }
              return true;
            }
            return true;
          }
          if (y1 == y2) {
            if (x2 > x1 + 1) {
              for (int i = x1 + 1; i < x2; i++) {
                if (colorcheck(i, y1) != 0)
                  return false;
              }
              return true;
            }
            if (x2 < x1 - 1) {
              for (int i = x1 - 1; i > x2; i--) {
                if (colorcheck(i, y1) != 0)
                  return false;
              }
              return true;
            }
            return true;
          }
          return false;
        }
    
      if (piece == BW) {
          if (colorcheck(x2, y2) == 1)
            return false;
          if (abs(x2 - x1) == abs(y2 - y1)) {
            if (y2 > y1 + 1 && x2 > x1 + 1) {
              for (int i = 1; i < (x2 - x1); i++) {
                if (colorcheck(x1 + i, y1 + i) != 0)
                  return false;
              }
              return true;
            }
            if (y2 > y1 + 1 && x2 < x1 - 1) {
              for (int i = 1; i < y2 - y1; i++) {
                if (colorcheck(x1 - i, y1 + i) != 0)
                  return false;
              }
              return true;
            }
            if (y2 < y1 - 1 && x2 > x1 + 1) {
              for (int i = 1; i < x2 - x1; i++) {
                if (colorcheck(x1 + i, y1 - i) != 0)
                  return false;
              }
              return true;
            }
            if (y2 < y1 - 1 && x2 < x1 - 1) {
              for (int i = 1; i < x1 - x2; i++) {
                if (colorcheck(x1 - i, y1 - i) != 0)
                  return false;
              }
              return true;
            }
            return true;
          }
          return false;
        }
    
      if (piece == NW) {
          if (colorcheck(x2, y2) == 1)
            return false;
          if (((x2 == x1 + 2 || x2 == x1 - 2) && (y2 == y1 + 1 || y2 == y1 - 1)) ||
              ((x2 == x1 + 1 || x2 == x1 - 1) && (y2 == y1 + 2 || y2 == y1 - 2)))
            return true;
          return false;
        }
    
      if (piece == PW) {
        if (x1 != x2) {
          if (y2 == 5 && x2 == enPassant) {
            board[4][x2] = EE;
            return true;
          }
          if (((x2 == x1 - 1 || x2 == x1 + 1) && colorcheck(x2, y2) == 2 && y2 == y1 + 1)) return true;
          return false;
        }
        if (colorcheck(x2, y2) != 0)
          return false;
        if (y1 == 1) {
          if (y2 == 3 && colorcheck(x1, 2) == 0) {
            enPassant = x1;
            delay = true;
            return true;
          }
        }
        if (y2 == y1 + 1)
          return true;
        return false;
      }
    
    }
    //black moves
    if (turn % 2 == 0) {
    
      if (piece == KB) {
        if (checkcheck(x2,y2,false) == true) return false;
        //castling rules
        if (x1 == 4 && y1 == 7 && y2 == 7) { 
          if (x2 == 2 && BlackCanCastleRight == true) {
            if (checkcheck(4,7,false) == false && checkcheck(3,7,false) == false) {
              if (board[7][1] == EE && board[7][2] == EE && board[7][3] == EE) {
                castleBleft = true;
                return true;
              }
            }
            return false;
          }
          if (x2 == 6 && BlackCanCastleRight == true) {
            if (checkcheck(4,7,false) == false && checkcheck(5,7,false) == false) {
              if (board[7][6] == EE && board[7][5] == EE) {
                castleBright = true;
                return true;
              }
            }
            return false;
          }
        }
        if (colorcheck(x2, y2) == 2)
          return false;
        if ((x1 == x2 && abs(y2 - y1) == 1) || (y1 == y2 && abs(x2 - x1) == 1))
          return true;
        if (abs(x2 - x1) == abs(y2 - y1) && abs(x2 - x1) == 1)
          return true;
        return false;
      }
        
      if (piece == QB) {
          if (colorcheck(x2, y2) == 2)
            return false;
          if (x1 == x2) {
            if (y2 > y1 + 1) {
              for (int i = y1 + 1; i < y2; i++) {
                if (colorcheck(x1, i) != 0)
                  return false;
              }
              return true;
            }
            if (y2 < y1 - 1) {
              for (int i = y1 - 1; i > y2; i--) {
                if (colorcheck(x1, i) != 0)
                  return false;
              }
              return true;
            }
            return true;
          }
          if (y1 == y2) {
            if (x2 > x1 + 1) {
              for (int i = x1 + 1; i < x2; i++) {
                if (colorcheck(i, y1) != 0)
                  return false;
              }
              return true;
            }
            if (x2 < x1 - 1) {
              for (int i = x1 - 1; i > x2; i--) {
                if (colorcheck(i, y1) != 0)
                  return false;
              }
              return true;
            }
            return true;
          }
          if (abs(x2 - x1) == abs(y2 - y1)) {
            if (y2 > y1 + 1 && x2 > x1 + 1) {
              for (int i = 1; i < (x2 - x1); i++) {
                if (colorcheck(x1 + i, y1 + i) != 0)
                  return false;
              }
              return true;
            }
            if (y2 > y1 + 1 && x2 < x1 - 1) {
              for (int i = 1; i < y2 - y1; i++) {
                if (colorcheck(x1 - i, y1 + i) != 0)
                  return false;
              }
              return true;
            }
            if (y2 < y1 - 1 && x2 > x1 + 1) {
              for (int i = 1; i < x2 - x1; i++) {
                if (colorcheck(x1 + i, y1 - i) != 0)
                  return false;
              }
              return true;
            }
            if (y2 < y1 - 1 && x2 < x1 - 1) {
              for (int i = 1; i < x1 - x2; i++) {
                if (colorcheck(x1 - i, y1 - i) != 0)
                  return false;
              }
              return true;
            }
            return true;
          }
          return false;
        }
    
      if (piece == RB) {
          if (colorcheck(x2, y2) == 2)
            return false;
          if (x1 == x2) {
            if (y2 > y1 + 1) {
              for (int i = y1 + 1; i < y2; i++) {
                if (colorcheck(x1, i) != 0)
                  return false;
              }
              return true;
            }
            if (y2 < y1 - 1) {
              for (int i = y1 - 1; i > y2; i--) {
                if (colorcheck(x1, i) != 0)
                  return false;
              }
              return true;
            }
            return true;
          }
          if (y1 == y2) {
            if (x2 > x1 + 1) {
              for (int i = x1 + 1; i < x2; i++) {
                if (colorcheck(i, y1) != 0)
                  return false;
              }
              return true;
            }
            if (x2 < x1 - 1) {
              for (int i = x1 - 1; i > x2; i--) {
                if (colorcheck(i, y1) != 0)
                  return false;
              }
              return true;
            }
            return true;
          }
          return false;
        }
    
      if (piece == BB) {
          if (colorcheck(x2, y2) == 2)
            return false;
          if (abs(x2 - x1) == abs(y2 - y1)) {
            if (y2 > y1 + 1 && x2 > x1 + 1) {
              for (int i = 1; i < (x2 - x1); i++) {
                if (colorcheck(x1 + i, y1 + i) != 0)
                  return false;
              }
              return true;
            }
            if (y2 > y1 + 1 && x2 < x1 - 1) {
              for (int i = 1; i < y2 - y1; i++) {
                if (colorcheck(x1 - i, y1 + i) != 0)
                  return false;
              }
              return true;
            }
            if (y2 < y1 - 1 && x2 > x1 + 1) {
              for (int i = 1; i < x2 - x1; i++) {
                if (colorcheck(x1 + i, y1 - i) != 0)
                  return false;
              }
              return true;
            }
            if (y2 < y1 - 1 && x2 < x1 - 1) {
              for (int i = 1; i < x1 - x2; i++) {
                if (colorcheck(x1 - i, y1 - i) != 0)
                  return false;
              }
              return true;
            }
            return true;
          }
          return false;
        }
    
      if (piece == NB) {
          if (colorcheck(x2, y2) == 2)
            return false;
          if (((x2 == x1 + 2 || x2 == x1 - 2) && (y2 == y1 + 1 || y2 == y1 - 1)) ||
              ((x2 == x1 + 1 || x2 == x1 - 1) && (y2 == y1 + 2 || y2 == y1 - 2)))
            return true;
          return false;
        }
    
      if (piece == PB) {
        if (x1 != x2) {
          if (y2 == 2 && x2 == enPassant) {
            board[3][x2] = EE;
            return true;
          }
          if (((x2 == x1 - 1 || x2 == x1 + 1) && colorcheck(x2, y2) == 1 && y2 == y1 - 1)) return true;
          return false;
        }
        if (colorcheck(x2, y2) != 0) return false;
        if (y1 == 6) {
          if (y2 == 4 && colorcheck(x1, 5) == 0) {
            enPassant = x1;
            delay = true;
            return true;
          }
        }
        if (y2 == y1 - 1)
          return true;
        return false;
      }
        
    }
    return false;
  }

  // executes inputed move
  void Move(int x1, int y1, int x2, int y2) {
    string piece = board[y1][x1];
    board[y2][x2] = piece;
    board[y1][x1] = EE;

    if (piece == KW) {
      kingWPos[0] = x2;
      kingWPos[1] = y2;
    }

    if (piece == KB) {
      kingBPos[0] = x2;
      kingBPos[1] = y2;
    }
    
    //castling white
    if (castleWleft == true) {
      board[0][0] = EE;
      board[0][3] = RW;
      castleWleft = false;
      WhiteCanCastleLeft = false;
      WhiteCanCastleRight = false;
      return;
    }
    if (castleWright == true) {
      board[0][7] = EE;
      board[0][5] = RW;
      castleWright = false;
      WhiteCanCastleLeft = false;
      WhiteCanCastleRight = false;
      return;
    }
    
    //castling black
    if (castleBleft == true) {
      board[7][0] = EE;
      board[7][3] = RW;
      castleBleft = false;
      BlackCanCastleLeft = false;
      BlackCanCastleRight = false;
      return;
    }
    if (castleBright == true) {
      board[7][7] = EE;
      board[7][5] = RW;
      castleBright = false;
      BlackCanCastleLeft = false;
      BlackCanCastleRight = false;
      return;
    }

    //castling checks
    if (WhiteCanCastleLeft == true && WhiteCanCastleRight == true) {
      if (x1 == 4 && y1 == 0) {
        WhiteCanCastleLeft = false;
        WhiteCanCastleRight = false;
      }
      if (x1 == 0 && y1 == 0) WhiteCanCastleLeft = false;
      if (x1 == 7 && y1 == 0) WhiteCanCastleRight = false;
    }
    if (BlackCanCastleLeft == true && BlackCanCastleRight == true) {
      if (x1 == 4 && y1 == 7) {
        BlackCanCastleLeft = false;
        BlackCanCastleRight = false;
      }
      if (x1 == 0 && y1 == 7) BlackCanCastleLeft = false;
      if (x1 == 7 && y1 == 7) BlackCanCastleRight = false;
    }
  }

  // checks colour of the piece at the inputed coordinates, if there is no piece there returns a 0
  int colorcheck(int x, int y) { // 0: empty, 1: white, 2: black
    string piece = board[y][x];
    if (piece == EE) return 0;
    return (piece == KW || piece == QW || piece == RW || piece == BW || piece == NW || piece == PW)? 1 : 2;
  }

  //checks whether the square is threatened
  bool checkcheck(int x,int y, bool col) {
    if (col) {
      cout << x;
      cout << y;
      if (Check(x+1,y+1) == PB || Check(x+1,y-1) == PB) return true;      
      if (Check(x+1,y+2) == NB || Check(x+2,y+1) == NB || Check(x+1,y-2) == NB || Check(x+2,y-1) == NB || Check(x-1,y+2) == NB || Check(x-2,y+1) == NB || Check(x-1,y-2) == NB || Check(x-2,y-1) == NB) return true;
      if (delve(x,y,1,0,col) == true) return true;
      if (delve(x,y,0,1,col) == true) return true;
      if (delve(x,y,1,1,col) == true) return true;
      if (delve(x,y,-1,0,col) == true) return true;
      if (delve(x,y,0,-1,col) == true) return true;
      if (delve(x,y,-1,-1,col) == true) return true;
      return false;
    }
    if (Check(x+1,y+1) == PW || Check(x+1,y-1) == PW) return true;
    if (Check(x+1,y+2) == NW || Check(x+2,y+1) == NW || Check(x+1,y-2) == NW || Check(x+2,y-1) == NW || Check(x-1,y+2) == NW || Check(x-2,y+1) == NW || Check(x-1,y-2) == NW || Check(x-2,y-1) == NW) return true;
    if (delve(x,y,1,0,col) == true) return true;
    if (delve(x,y,0,1,col) == true) return true;
    if (delve(x,y,1,1,col) == true) return true;
    if (delve(x,y,-1,0,col) == true) return true;
    if (delve(x,y,0,-1,col) == true) return true;
    if (delve(x,y,-1,-1,col) == true) return true;
    return false;
  }

  bool delve(int x,int y,int sx,int sy, bool col) {
    x = x+sx;
    y = y+sy;
    if (col) {
      if (x<0 || x>7 || y<0 || y>7) return false;
      if (board[y][x] == QB) return true;
      if (sx != 0 != sy != 0) {if (board[y][x] == RB) return true;}
      if (sx != 0 && sy != 0) {if (board[y][x] == BB) return true;}
      if (board[y][x] == EE) {if (delve(x,y,sx,sy,col) == true) return true;};
      return false;
    }
    if (x<0 || x>7 || y<0 || y>7) return false;
    if (board[y][x] == QW) return true;
    if (sx != 0 != sy != 0) {if (board[y][x] == RW) return true;}
    if (sx != 0 && sy != 0) {if (board[y][x] == BW) return true;}
    if (board[y][x] == EE) {if (delve(x,y,sx,sy,col) == true) return true;};
    return false;
  }
  string Check(int x,int y) {
    if (x > 7 || x < 0 || y > 7 || y < 0) return "";
    return board[y][x];
  }
};

// create a ChessBoard called boarstate.
ChessBoard boardstate;

bool gameActive;
int ix1;
int iy1;
int ix2;
int iy2;

void MoveHandler(int x1, int y1, int x2, int y2) {
  if (boardstate.legalMoveCheck(x1, y1, x2, y2) == true) {
    boardstate.Move(x1, y1, x2, y2);
    boardstate.turn = boardstate.turn + 1;
    if (boardstate.delay == false) {
      boardstate.enPassant = 10;
      return;
    }
    boardstate.delay = false;
    return;
  }
  cout << "not a legal move";
  return;
}

void decode(string in) {
  ix1 = in[0];
  ix1 = ix1 - 97;
  iy1 = in[1];
  iy1 = iy1 - 49;
  ix2 = in[3];
  ix2 = ix2 - 97;
  iy2 = in[4];
  iy2 = iy2 - 49;
}

void chess() {
  boardstate.kingWPos[0] = 4;
  boardstate.kingWPos[1] = 0;
  boardstate.kingBPos[0] = 4;
  boardstate.kingBPos[1] = 7;
  boardstate.turn = 1;
  boardstate.WhiteCanCastleLeft = true;
  boardstate.WhiteCanCastleRight = true;
  boardstate.BlackCanCastleLeft = true;
  boardstate.BlackCanCastleRight = true;
  boardstate.castleWleft = false;
  boardstate.castleWright = false;
  boardstate.castleBleft = false;
  boardstate.castleBright = false;
  while (gameActive == true) {
    string instruct;
    getline(cin, instruct);
    decode(instruct);
    MoveHandler(ix1, iy1, ix2, iy2);
    cout << "\n\n";
    boardstate.printBoard();
    cout << "\n";
  }
}

int main() {
  cout << "Welcome to console chess!\n\ngive your move commands in the form [original coordinates]>[coordinates after move]\nfor example: e2>e4\n\n\n";
  boardstate.printBoard();
  cout << "\n";
  gameActive = true;
  chess();
  return 0;
}
```

### Challenges

using unicode characters stored as strings in the 2d array presented some challenges which may not have been present if I had instead used integer references to chess pieces and then converted to unicode characters when displaying them. I couldn't use a switch statement when choosing which piece's rules to check against in the code because I couldn't find a good way to get a unique integer for each string unicode character because I don't understand the way the unicode character's were stored as strings very well, it was my impression that I'd be able to store unicode characters as chars but that didn't work and I resorted to strings. In the end I used a function for checking for legal moves and threw in an if statement for each piece, it's a lot of if statements and then nested if statements, perhaps each piece's rules could be placed in a function to tidy this up slightly but the current implementation isn't broken so hey why bother -- I mean that I chose to focus on more important problems with the code.

another challenge was how I was going to check if the king was in check, I decided to create a function that would check if a square at the input coordinates was threatened by a black piece if a true was passed to the function as a third input and white piece if a false is passed. (the third input took the form of a boolean "col".) I utilized this to prevent the king from moving to squares threatened by pieces of the other colour and also to ensure that it was considered an illegal move to castle through check.

Whilst making the check checking function I had to deal with the issue of trying to check coordinates outside of the array's limits two times, the first time was within the recursive function named "delve" that I used to check if the diagonals, horizontal and vertical were free of threats such as a queen, a rook or a bishop where I had to create an early exit condition to return false (no check) if it was about to check a coordinate that would be out of bounds. The second time was when I was adding the castling rules and realised that an error before hadn't made itself obvious to me. The first check I make within the function is to check each square a knight could threaten the king from for a knight of the opposite colour to the king. This lead to checking out of bounds coordinates, I fixed this by creating a function that I could replace all of the problematic references to the 2d array with which would first check if the coordinates being checked were out of bounds ("Check(int x, int y)".

mistakes with scoping and generation of the 2 dimensional array led me to create a class called Chess Board to store the array and the unicode character strings in. I could I assume to a great deal of tidying up with the current functions.

I accidentally pressed a button on repl that screwed with the formatting of the code. I'm going to use a proper ide asap.

## Testing

Well... It runs in repl.\
Due to the sheer number of posible positions in chess I couldn't test each possibility so here are a few tests I did on a small scale along the way. In addition to these tests I played a test game where I realized that I hadn't set the bounds quite correctly for the function that checks if a square is threatened (checkcheck) and fixed those when I found error.

### Tests

| Test | Instructions                                                                                                                         | What I expect                                                                            | What actually happens | Pass/Fail |
| ---- | ------------------------------------------------------------------------------------------------------------------------------------ | ---------------------------------------------------------------------------------------- | --------------------- | --------- |
| 1    | Run code                                                                                                                             | a chess board is displayed and the console accepts user input                            | As expected           | Pass      |
| 2    | try a few legal moves                                                                                                                | a new chess board appears with the new moves                                             | As expected           | Pass      |
| 3    | try a few illegal moves                                                                                                              | message prompt saying move is not legal then chess board is sent with the previous state | As expected           | Pass      |
| 4    | attempt to castle using the fastest possible castle (castling to the right after an e4 opening and moving the bishop and knight out) | all the moves are displayed correctly                                                    | As expected           | Pass      |
| 5    | attempt to move into check                                                                                                           | fails                                                                                    | As expected           | Pass      |

### Evidence

screenshot time:

![](<../.gitbook/assets/image (1).png>)
