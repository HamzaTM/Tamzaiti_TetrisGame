# Tetris  Game
# Table of Content 
- [Overview](#Overview)
- [Functionality](#Functionality)
- [Project Source Files](#Project#Source#Files)
- [Main Design](#Main#Design)
- [Tetris game Creation Steps](#Tetris#game#Creation#Steps)
- [Conclusion](#Conclusion)

## Overview
- In this report we are going to know how to create a game which has a been something special in our childhood called Tetris Game.
- The object of our game is to stack pieces dropped from the top of the playing area so that they fill entire rows at the bottom of the playing area.
- When a row is filled, all the blocks on that row are removed, the player earns a number of points, and the pieces above are moved down to occupy that row. If more than one row is filled, the blocks on each row are removed, and the player earns extra points.
- We are expecting that the game would look somthing like this:
![hahaha](https://user-images.githubusercontent.com/80457241/152692259-46f74fb5-d434-4b11-947e-65f906accb86.PNG)

##Project Source Files
My project contains 3 files:
- Tetris.h
- Tetris.Cpp
- main.cpp
#### Tetris.h
```cpp
#ifndef  TETRIS_H
#define  TETRIS_H
#include  <QWidget>
const  int  BlockSize=25;  //side  length  of  a  single  block  unit
const  int  Margin=7;  //scene  margins
const  int  AreaRow=25;  //number  of  scene  lines
const  int  AreaCol=15;  //number  of  scene  columns
//direction
enum  Direction
{
  UP,
  DOWN,
  LEFT,
  RIGHT,
  SPACE
};
//Define  Boundary  Information
struct  Border
{
  int  UpBound;
  int  DownBound;
  int  LeftBound;
  int  RightBound;
};
//coordinate
struct  BlockPoint
{
  int  PosX;
  int  PosY;
};
namespace  Ui  {
class  Widget;
}
class  Widget  :  public  QWidget
{
  Q_OBJECT
public:
  void  InitGame();  //Initialization
  void  StartGame();  //Start  the  Game
  void  GameOver();  //Game  Over
  void  ResetBlock();  //Reset  Block
  void  BlockMove(Direction  dir);  //Block  Change
  void  BlockRotate(int  block[4][4]);  //Block  Rotation
  void  CreateBlock(int  block[4][4],int  BlockID);  //generate  blocks
  void  GetBorder(int  block[4][4],Border  &border);  //Calculate  the  Boundaries
  void  ConvertStable(int  x,int  y);  //Convert  to  Stable  Block
  bool  IsCollide(int  x,int  y,Direction  dir);  //Determine  Whether  it  Will  Collide
public:
  explicit  Widget(QWidget  *parent  =  0);
  ~Widget();
  virtual  void  paintEvent(QPaintEvent  *event);  //Scene  Refresh
  virtual  void  timerEvent(QTimerEvent  *event);  //Timer  Event
  virtual  void  keyPressEvent(QKeyEvent  *event);  //Keyboard  Response
private:
  Ui::Widget  *ui;
private:
  int  GameArea[AreaRow][AreaCol];  //Scene  area,  1  means  active  block,  2  means  stable  block,  0  means  empty
  BlockPoint  BlockPos;  //current  block  coordinates
  int  CurBlock[4][4];  //current  block  shape
  Border  CurBorder;  //current  block  boundaries
  int  NextBlock[4][4];  //next  block  shape
  bool  isStable;  //Is  the  current  block  stable?
  int  score;  //game  score
  int  GameTimer;  //block  drop  timer
  int  PaintTimer;  //Render  refresh  timer
  int  Speed;  //Fall  time  interval
  int  Refresh;  //refresh  interval
};
#endif  //  TETRIS_H
```
#### Tetris.Cpp
```cpp
#include  <time.h>
#include  <QMessageBox>
#include  <QDebug>
#include  <QPainter>
#include  <QKeyEvent>
#include  "Tetris.h"
#include  "ui_widget.h"  
//Define  pattern  codes  and  boundaries
//  The  Square  Block
int  shape1[4][4]=
{
  {0,0,0,0},
  {0,1,1,0},
  {0,1,1,0},
  {0,0,0,0}
};
//The  Right  L
int  shape2[4][4]=
{
  {0,1,0,0},
  {0,1,0,0},
  {0,1,1,0},
  {0,0,0,0}
};
//The  Left  L
int  shape3[4][4]=
{
  {0,0,1,0},
  {0,0,1,0},
  {0,1,1,0},
  {0,0,0,0}
};
//The  Right  S
int  shape4[4][4]=
{
  {0,1,0,0},
  {0,1,1,0},
  {0,0,1,0},
  {0,0,0,0}
};
//The  Left  S
int  shape5[4][4]=
{
  {0,0,1,0},
  {0,1,1,0},
  {0,1,0,0},
  {0,0,0,0}
};
//The  T  Block
int  shape6[4][4]=
{
  {0,0,0,0}
  {0,0,1,0},
  {0,1,1,1},
  {0,0,0,0}
};
//The  I  Block
int  shape7[4][4]=
{
  {0,0,1,0},
  {0,0,1,0},
  {0,0,1,0},
  {0,0,1,0}
};
//Copy  the  square  pattern
inline  void  BlockCpy(int  dblock[4][4],int  sblock[4][4])
{
  for(int  i=0;i<4;i++)
  for(int  j=0;j<4;j++)
  dblock[i][j]=sblock[i][j];
}
Widget::Widget(QWidget  *parent)  :
  QWidget(parent),
  ui(new  Ui::Widget)
{
  ui->setupUi(this);
  //Adjust  window  size  layout
resize(AreaCol*BlockSize+Margin*4+4*BlockSize,AreaRow*BlockSize+Margin*2);
  //Initialize  the  game
  InitGame();
}
Widget::~Widget()
{
  delete  ui;
}
void  Widget::paintEvent(QPaintEvent  *event)
{
  QPainter  painter(this);
  //draw  game  scene  borders
  painter.setBrush(QBrush(Qt::darkGray,Qt::Dense2Pattern));
painter.drawRect(Margin,Margin,AreaCol*BlockSize,AreaRow*BlockSize);
  //block  trailer
  painter.setBrush(QBrush(Qt::blue,Qt::SolidPattern));
  for(int  i=0;i<4;i++)
  for(int  j=0;j<4;j++)
  if(NextBlock[i][j]==1)
painter.drawRect(Margin*3+AreaCol*BlockSize+j*BlockSize,Margin+i*BlockSize,BlockSize,BlockSize);
  //draw  score
  painter.setPen(Qt::black);
  painter.setFont(QFont("times",14));  painter.drawText(QRect(Margin*3+AreaCol*BlockSize,Margin*2+4*BlockSize,BlockSize*4,BlockSize*4),Qt::AlignCenter,"SCORE: "+QString::number(score));
  //Draw  falling  squares  and  stable  squares,  note  that  the  color  of  the  square  edge  is  based  on  setPen,  the  default  is  black
  for(int  i=0;i<AreaRow;i++)
  for(int  j=0;j<AreaCol;j++)
  {
  //draw  active  square
  if(GameArea[i][j]==1)
  {
  painter.setBrush(QBrush(Qt::red,Qt::SolidPattern));
painter.drawRect(j*BlockSize+Margin,i*BlockSize+Margin,BlockSize,BlockSize);
  }
  //draw  stable  blocks
  else  if(GameArea[i][j]==2)
  {
  painter.setBrush(QBrush(Qt::green,Qt::SolidPattern));
painter.drawRect(j*BlockSize+Margin,i*BlockSize+Margin,BlockSize,BlockSize);
  }
  }
}
void  Widget::timerEvent(QTimerEvent  *event)
{
  //block  falling
  if(event->timerId()==GameTimer)
  BlockMove(DOWN);
  //refresh  the  screen
  if(event->timerId()==PaintTimer)
  update();
}
void  Widget::keyPressEvent(QKeyEvent  *event)
{
  switch(event->key())
  {
  case  Qt::Key_Up:
  BlockMove(UP);
  break;
  case  Qt::Key_Down:
  BlockMove(DOWN);
  break;
  case  Qt::Key_Left:
  BlockMove(LEFT);
  break;
  case  Qt::Key_Right:
  BlockMove(RIGHT);
  break;
  case  Qt::Key_Space:
  BlockMove(SPACE);
  break;
  default:
  break;
  }
}
void  Widget::CreateBlock(int  block[4][4],int  BlockID)
{
  switch  (BlockID)
  {
  case  0:
  BlockCpy(block,shape1);
  break;
  case  1:
  BlockCpy(block,shape2);
  break;
  case  2:
  BlockCpy(block,shape3);
  break;
  case  3:
  BlockCpy(block,shape4);
  break;
  case  4:
  BlockCpy(block,shape5);
  break;
  case  5:
  BlockCpy(block,shape6);
  break;
  case  6:
  BlockCpy(block,shape7);
  break;
  default:
  break;
  }
}
void  Widget::GetBorder(int  block[4][4],Border  &border)
{
  //Calculate  the  upper,  lower,  left  and  right  boundaries
  for(int  i=0;i<4;i++)
  for(int  j=0;j<4;j++)
  if(block[i][j]==1)
  {
  border.DownBound=i;
  break;  //until  the  last  row  has  1
  }
  for(int  i=3;i>=0;i--)
  for(int  j=0;j<4;j++)
  if(block[i][j]==1)
  {
  border.UpBound=i;
  break;
  }
  for(int  j=0;j<4;j++)
  for(int  i=0;i<4;i++)
  if(block[i][j]==1)
  {
  border.RightBound=j;
  break;
  }
  for(int  j=3;j>=0;j--)
  for(int  i=0;i<4;i++)
  if(block[i][j]==1)
  {
  border.LeftBound=j;
  break;
}  
void  Widget::InitGame()
{
  for(int  i=0;i<AreaRow;i++)
  for(int  j=0;j<AreaCol;j++)
  GameArea[i][j]=0;
  Speed=700;
  Refresh=30;
  //Initialize  random  number  seed
  srand(time(0));
  //Score  clear  to  0
  score=0;
  //start  the  game
  StartGame();
}
void  Widget::ResetBlock()
{
  //generate  current  block
  BlockCpy(CurBlock,NextBlock);
  GetBorder(CurBlock,CurBorder);
  //generate  the  next  block
  int  BlockID=rand()%7;
  CreateBlock(NextBlock,BlockID);
  //Set  the  initial  block  coordinates,  with  the  upper  left  corner  of  the  block  as  the  anchor  point
  BlockPoint  StartPoint;
  StartPoint.PosX=AreaCol/2-2;
  StartPoint.PosY=0;
  BlockPos=StartPoint;
}
void  Widget::StartGame()
{
  GameTimer=startTimer(Speed);  //Start  the  game  timer
  PaintTimer=startTimer(Refresh);  //Enable  interface  refresh  timer
  //Generate  initial  next  block
  int  BlockID=rand()%7;
  CreateBlock(NextBlock,BlockID);
  ResetBlock();  //Generate  Blocks
}
void  Widget::GameOver()
{
  //game  over  stop  timer
  killTimer(GameTimer);
  killTimer(PaintTimer);
  QMessageBox::information(this,"Failed","GAME OVER");
}
void  Widget::BlockRotate(int  block[4][4])
{
  int  TempBlock[4][4];
  for(int  i=0;i<4;i++)
  for(int  j=0;j<4;j++)
  TempBlock[3-j][i]=block[i][j];
  for(int  i=0;i<4;i++)
  for(int  j=0;j<4;j++)
  block[i][j]=TempBlock[i][j];
}
void  Widget::ConvertStable(int  x,int  y)
{
  for(int  i=CurBorder.UpBound;i<=CurBorder.DownBound;i++)
  for(int  j=CurBorder.LeftBound;j<=CurBorder.RightBound;j++)
  if(CurBlock[i][j]==1)
  GameArea[y+i][x+j]=2;  //Do  not  confuse  x  and  y
}
bool  Widget::IsCollide(int  x,int  y,Direction  dir)
{
  //Use  the  copied  temporary  block  to  make  judgments
  int  TempBlock[4][4];
  BlockCpy(TempBlock,CurBlock);
  Border  TempBorder;
  GetBorder(TempBlock,TempBorder);
  //First  try  to  walk  a  square  in  a  certain  direction
  switch(dir)
  {
  case  UP:
  BlockRotate(TempBlock);
  GetBorder(TempBlock,TempBorder);  //Boundaries  to  be  recalculated  after  rotation
  break;
  case  DOWN:
  y+=1;
  break;
  case  LEFT:
  x-=1;
  break;
  case  RIGHT:
  x+=1;
  break;
  default:
  break;
  }
  for(int  i=TempBorder.UpBound;i<=TempBorder.DownBound;i++)
  for(int  j=TempBorder.LeftBound;j<=TempBorder.RightBound;j++)
  if(GameArea[y+i][x+j]==2&&TempBlock[i][j]==1||x+TempBorder.LeftBound<0||x+TempBorder.RightBound>AreaCol-1)
  return  true;
  return  false;
}
void  Widget::BlockMove(Direction  dir)
{
  switch  (dir)  {
  case  UP:
  if(IsCollide(BlockPos.PosX,BlockPos.PosY,UP))
  break;
  //Rotate  90  degrees  counterclockwise
  BlockRotate(CurBlock);
  //To  prevent  bugs  after  rotation,  i  and  j  reset  the  block  from  0  to  4
  for(int  i=0;i<4;i++)
  for(int  j=0;j<4;j++)
  GameArea[BlockPos.PosY+i][BlockPos.PosX+j]=CurBlock[i][j];
  //Recalculate  Boundaries
  GetBorder(CurBlock,CurBorder);
  break;
  case  DOWN:
  //Blocks  no  longer  move  when  they  reach  the  border
  if(BlockPos.PosY+CurBorder.DownBound==AreaRow-1)
  {
  ConvertStable(BlockPos.PosX,BlockPos.PosY);
  ResetBlock();
  break;
  }
  //Collision  detection,  only  calculates  the  upper,  lower,  left  and  right  boundaries,  first  try  to  walk  a  grid,  if  it  collides,  stabilize  the  block  and  then  jump  out
  if(IsCollide(BlockPos.PosX,BlockPos.PosY,DOWN))
  {
  //Turn  into  a  stable  block  only  if  it  can't  fall  in  the  end
  ConvertStable(BlockPos.PosX,BlockPos.PosY);
  ResetBlock();
  break;
  }
  //Restore  the  scene  on  the  block,  in  order  to  clear  the  block  residue  during  the  movement
  for(int  j=CurBorder.LeftBound;j<=CurBorder.RightBound;j++)
  GameArea[BlockPos.PosY][BlockPos.PosX+j]=0;
  //If  there  is  no  collision,  it  will  fall  one  block
  BlockPos.PosY+=1;
  //The  square  drops  one  square,  copy  it  to  the  scene,  pay  attention  to  the  left  and  right  borders
  for(int  i=0;i<4;i++)  //Must  be  0  to  4  not  bounds  index,  taking  into  account  bounds  recalculation  after  rotation
  for(int  j=CurBorder.LeftBound;j<=CurBorder.RightBound;j++)
  if(BlockPos.PosY+i<=AreaRow-1&&GameArea[BlockPos.PosY+i][BlockPos.PosX+j]!=2)  //Note  that  the  scene  array  is  not  out  of  bounds,  and  will  not  erase  stable  blocks
  GameArea[BlockPos.PosY+i][BlockPos.PosX+j]=CurBlock[i][j];
  break;
  case  LEFT:
  //to  the  left  border  or  the  collision  is  no  longer  to  the  left  if(BlockPos.PosX+CurBorder.LeftBound==0||IsCollide(BlockPos.PosX,BlockPos.PosY,LEFT))
  break;
  //Restore  the  right  scene  of  the  block,  in  order  to  clear  the  block  residue  during  the  movement
  for(int  i=CurBorder.UpBound;i<=CurBorder.DownBound;i++)
  GameArea[BlockPos.PosY+i][BlockPos.PosX+3]=0;
  BlockPos.PosX-=1;
  //Move  the  block  one  block  to  the  left  and  copy  it  to  the  scene
  for(int  i=CurBorder.UpBound;i<=CurBorder.DownBound;i++)
  for(int  j=0;j<4;j++)
  if(BlockPos.PosX+j>=0&&GameArea[BlockPos.PosY+i][BlockPos.PosX+j]!=2)  //Note  that  the  scene  array  is  not  out  of  bounds
  GameArea[BlockPos.PosY+i][BlockPos.PosX+j]=CurBlock[i][j];
  break;
  case  RIGHT:
  if(BlockPos.PosX+CurBorder.RightBound==AreaCol-1||IsCollide(BlockPos.PosX,BlockPos.PosY,RIGHT))
  break;
  //Restore  the  left  scene  of  the  block,  in  order  to  clear  the  block  residue  during  the  movement
  for(int  i=CurBorder.UpBound;i<=CurBorder.DownBound;i++)
  GameArea[BlockPos.PosY+i][BlockPos.PosX]=0;
  BlockPos.PosX+=1;
  //Move  the  block  one  block  to  the  right  and  copy  it  to  the  scene
  for(int  i=CurBorder.UpBound;i<=CurBorder.DownBound;i++)
  for(int  j=0;j<4;j++)
  if(BlockPos.PosX+j<=AreaCol-1&&GameArea[BlockPos.PosY+i][BlockPos.PosX+j]!=2)  //Note  that  the  scene  array  is  not  out  of  bounds
  GameArea[BlockPos.PosY+i][BlockPos.PosX+j]=CurBlock[i][j];
  break;
  case  SPACE:  //once  to  the  end
  //Move  down  one  space  at  a  time,  until  you  can't  move  down
  while(BlockPos.PosY+CurBorder.DownBound<AreaRow-1&&!IsCollide(BlockPos.PosX,BlockPos.PosY,DOWN))
  {
  //Restore  the  scene  on  the  block,  in  order  to  clear  the  block  residue  during  the  movement
  for(int  j=CurBorder.LeftBound;j<=CurBorder.RightBound;j++)
  GameArea[BlockPos.PosY][BlockPos.PosX+j]=0;
  //If  there  is  no  collision,  it  will  fall  one  block
  BlockPos.PosY+=1;
  //The  square  drops  one  square,  copy  it  to  the  scene,  pay  attention  to  the  left  and  right  borders
  for(int  i=0;i<4;i++)  //Must  be  0  to  4
  for(int  j=CurBorder.LeftBound;j<=CurBorder.RightBound;j++)
  if(BlockPos.PosY+i<=AreaRow-1&&GameArea[BlockPos.PosY+i][BlockPos.PosX+j]!=2)  //Note  that  the  scene  array  is  not  out  of  bounds,  and  will  not  erase  stable  blocks
  GameArea[BlockPos.PosY+i][BlockPos.PosX+j]=CurBlock[i][j];
  }
  ConvertStable(BlockPos.PosX,BlockPos.PosY);
  ResetBlock();
  break;
  default:
  break;
  }
  //Process  the  elimination  of  lines,  and  the  lines  above  the  entire  scene  are  moved  down  in  turn
  int  i=AreaRow-1;
  int  LineCount=0;  //Number  of  deleted  lines
  while(i>=1)
  {
  bool  isLineFull=true;
  for(int  j=0;j<AreaCol;j++)
  if(GameArea[i][j]==0)
  {
  isLineFull=false;
  i--;
  break;
  }
  if(isLineFull)
  {
  for(int  k=i;k>=1;k--)
  for(int  j=0;j<AreaCol;j++)
  GameArea[k][j]=GameArea[k-1][j];
  LineCount++;//Increase  the  number  of  lines  to  be  eliminated  each  time
  }
  }
  score+=LineCount*10;  //Score
  //Determine  if  the  game  is  over
  for(int  j=0;j<AreaCol;j++)
  if(GameArea[0][j]==2)  //There  is  also  a  stable  block  at  the  top
  GameOver();
}
```
#### main.cpp
```cpp
#include  "Tetris.h"
#include  <QApplication>
int  main(int  argc,  char  *argv[])
{
  QApplication  a(argc,  argv);
  Widget  w;
  w.show();
  return  a.exec();
}
```


 ## Functionality
 -   To move the Tetriminos left press  **Left arrow**  key.
 -  To move the Tetriminos right press **Right arrow** key.
 -   To rotate the Tetriminos press  **Up arrow**  key.
 -  To increase the speed press  **Down arrow**  key.
 -  To immediately reach the bottom area press  **Space bar**  key.
 ## Main Design
 On the left is the game area, and on the right is the area that displays the next block and the score.
 #### **Define the game data structure**
 Both game scenes and blocks are stored in two-dimensional arrays, 1 means there are squares, 0 means no squares.
 ##### Scene Data
 - We set the parameters of our scene game, like margin,colums,lines,and length
 ```cpp
const  int  BlockSize=25;  //side  length  of  a  single  block  unit
const  int  Margin=7;  //scene  margins
const  int  AreaRow=25;  //number  of  scene  lines
const  int  AreaCol=15;  //number  of  scene  columns
 ```
##### Block shape Data
- There are a total of 7 blocks of different shapes (The square block, The  Right L, The Left L, The right S, The Left S, The T block and The I block)
- Each shape can be represented by 4 coordinates in a 4*4 matrix. When changing the shape, it is a different set of coordinates
Here for example **The Square Block** :
 ```cpp
//  The  Square  Block
int  shape1[4][4]=
{
  {0,0,0,0},
  {0,1,1,0},
  {0,1,1,0},
  {0,0,0,0}
};
 ```
 ![image](https://user-images.githubusercontent.com/80457241/152694333-88131462-9dfc-4180-b98a-f3e6eb89f2fc.png)

- Since collision detection is involved, it is necessary to determine the upper, lower, left, and right boundaries of the block pattern:
 ```cpp
void  Widget::GetBorder(int  block[4][4],Border  &border)

{

  //Calculate  the  upper,  lower,  left  and  right  boundaries
  for(int  i=0;i<4;i++)
  for(int  j=0;j<4;j++)
  if(block[i][j]==1)
  {
  border.DownBound=i;
  break;  //until  the  last  row  has  1
  }
  for(int  i=3;i>=0;i--)
  for(int  j=0;j<4;j++)
  if(block[i][j]==1)
  {
  border.UpBound=i;
  break;
  }
  for(int  j=0;j<4;j++)
  for(int  i=0;i<4;i++)
  if(block[i][j]==1)
  {
  border.RightBound=j;
  break;
  }
  for(int  j=3;j>=0;j--)
  for(int  i=0;i<4;i++)
  if(block[i][j]==1)
  {
  border.LeftBound=j;
  break;
  }
}
 ```
- Here we define the game scene and the variables of the current block and the next block in the Widget class:
 ```cpp
  int  GameArea[AreaRow][AreaCol];  //Scene  area,  1  means  active  block,  2  means  stable  block,  0  means  empty
  BlockPoint  BlockPos;  //current  block  coordinates
  int  CurBlock[4][4];  //current  block  shape
  Border  CurBorder;  //current  block  boundaries
  int  NextBlock[4][4];  //next  block  shape
 ```
 ## Tetris game Creation Steps
 #### **Add render loop and timer events and keyboard listeners**
 - The scene of the game is refreshed every frame to achieve interface changes. Here we draw the borders,the score area,the falling shapes and the the area where the next shape appear:
 
  ```cpp
void  Widget::paintEvent(QPaintEvent  *event)
{
  QPainter  painter(this);
  //draw  game  scene  borders
  painter.setBrush(QBrush(Qt::darkGray,Qt::Dense2Pattern));
painter.drawRect(Margin,Margin,AreaCol*BlockSize,AreaRow*BlockSize);
  //block  trailer
  painter.setBrush(QBrush(Qt::blue,Qt::SolidPattern));
  for(int  i=0;i<4;i++)
  for(int  j=0;j<4;j++)
  if(NextBlock[i][j]==1)
painter.drawRect(Margin*3+AreaCol*BlockSize+j*BlockSize,Margin+i*BlockSize,BlockSize,BlockSize);
  //draw  score
  painter.setPen(Qt::black);
  painter.setFont(QFont("times",14));
painter.drawText(QRect(Margin*3+AreaCol*BlockSize,Margin*2+4*BlockSize,BlockSize*4,BlockSize*4),Qt::AlignCenter,"SCORE: "+QString::number(score));
  //Draw  falling  squares  and  stable  squares,  note  that  the  color  of  the  square  edge  is  based  on  setPen,  the  default  is  black
  for(int  i=0;i<AreaRow;i++)
  for(int  j=0;j<AreaCol;j++)
  {
  //draw  active  square
  if(GameArea[i][j]==1)
  {
  painter.setBrush(QBrush(Qt::red,Qt::SolidPattern));
painter.drawRect(j*BlockSize+Margin,i*BlockSize+Margin,BlockSize,BlockSize);
  }
  //draw  stable  blocks
  else  if(GameArea[i][j]==2)
  {
  painter.setBrush(QBrush(Qt::green,Qt::SolidPattern));
painter.drawRect(j*BlockSize+Margin,i*BlockSize+Margin,BlockSize,BlockSize);
  }
  }
}
 ```
- In our game they are two timers are required, one for the automatic falling of the block, and one for the refresh frame rate of the interface
```cpp
void  Widget::timerEvent(QTimerEvent  *event)
{
  //block  falling
  if(event->timerId()==GameTimer)
  BlockMove(DOWN);
  //refresh  the  screen
  if(event->timerId()==PaintTimer)
  update();
}
```
- Here we describe how the game is playing by keyboard response, likw how the upper piece rotates, moves left and right, and the space bar directly falls to the bottom:
```cpp
void  Widget::keyPressEvent(QKeyEvent  *event)
{
  switch(event->key())
  {
  case  Qt::Key_Up:
  BlockMove(UP);
  break;
  case  Qt::Key_Down:
  BlockMove(DOWN);
  break;
  case  Qt::Key_Left:
  BlockMove(LEFT);
  break;
  case  Qt::Key_Right:
  BlockMove(RIGHT);
  break;
  case  Qt::Key_Space:
  BlockMove(SPACE);
  break;
  default:
  break;
  }
}
```
#### **Blocks move, rotate, disappear, the next block appears and the end logic**
- The entire scene takes the upper left corner as the coordinates origin.
- The square pattern takes its upper left corner as the local coordinate origin.
-  The global coordinates x, y and the local coordinates j, i can be used for positioning.
- Every time we make move or rotate, we need to think whether there is a collision out of bounds with the original scene after rotating or moving one step. If not, the action will be taken. If there is, it will not change. Then the active block (red) is converted into a stable block (green).
- The principle of collision detection is to detect whether the original scene and the active block are not 0 at a certain point.
```cpp
void  Widget::BlockMove(Direction  dir)
{
  switch  (dir)  {
  case  UP:
  if(IsCollide(BlockPos.PosX,BlockPos.PosY,UP))
  break;
  //Rotate  90  degrees  counterclockwise
  BlockRotate(CurBlock);
  //To  prevent  bugs  after  rotation,  i  and  j  reset  the  block  from  0  to  4
  for(int  i=0;i<4;i++)
  for(int  j=0;j<4;j++)
  GameArea[BlockPos.PosY+i][BlockPos.PosX+j]=CurBlock[i][j];
  //Recalculate  Boundaries
  GetBorder(CurBlock,CurBorder);
  break;
  case  DOWN:
  //Blocks  no  longer  move  when  they  reach  the  border
  if(BlockPos.PosY+CurBorder.DownBound==AreaRow-1)
  {
  ConvertStable(BlockPos.PosX,BlockPos.PosY);
  ResetBlock();
  break;
  }
  //Collision  detection,  only  calculates  the  upper,  lower,  left  and  right  boundaries,  first  try  to  walk  a  grid,  if  it  collides,  stabilize  the  block  and  then  jump  out
  if(IsCollide(BlockPos.PosX,BlockPos.PosY,DOWN))
  {
  //Turn  into  a  stable  block  only  if  it  can't  fall  in  the  end
  ConvertStable(BlockPos.PosX,BlockPos.PosY);
  ResetBlock();
  break;
  }
  //Restore  the  scene  on  the  block,  in  order  to  clear  the  block  residue  during  the  movement
  for(int  j=CurBorder.LeftBound;j<=CurBorder.RightBound;j++)
  GameArea[BlockPos.PosY][BlockPos.PosX+j]=0;
  //If  there  is  no  collision,  it  will  fall  one  block
  BlockPos.PosY+=1;
  //The  square  drops  one  square,  copy  it  to  the  scene,  pay  attention  to  the  left  and  right  borders
  for(int  i=0;i<4;i++)  //Must  be  0  to  4  not  bounds  index,  taking  into  account  bounds  recalculation  after  rotation
  for(int  j=CurBorder.LeftBound;j<=CurBorder.RightBound;j++)
  if(BlockPos.PosY+i<=AreaRow-1&&GameArea[BlockPos.PosY+i][BlockPos.PosX+j]!=2)  //Note  that  the  scene  array  is  not  out  of  bounds,  and  will  not  erase  stable  blocks
  GameArea[BlockPos.PosY+i][BlockPos.PosX+j]=CurBlock[i][j];
  break;
  case  LEFT:
  //to  the  left  border  or  the  collision  is  no  longer  to  the  left
if(BlockPos.PosX+CurBorder.LeftBound==0||IsCollide(BlockPos.PosX,BlockPos.PosY,LEFT))
  break;
  //Restore  the  right  scene  of  the  block,  in  order  to  clear  the  block  residue  during  the  movement
  for(int  i=CurBorder.UpBound;i<=CurBorder.DownBound;i++)
  GameArea[BlockPos.PosY+i][BlockPos.PosX+3]=0;
  BlockPos.PosX-=1;
  //Move  the  block  one  block  to  the  left  and  copy  it  to  the  scene
  for(int  i=CurBorder.UpBound;i<=CurBorder.DownBound;i++)
  for(int  j=0;j<4;j++)
  if(BlockPos.PosX+j>=0&&GameArea[BlockPos.PosY+i][BlockPos.PosX+j]!=2)  //Note  that  the  scene  array  is  not  out  of  bounds
  GameArea[BlockPos.PosY+i][BlockPos.PosX+j]=CurBlock[i][j];
  break;
  case  RIGHT:
  if(BlockPos.PosX+CurBorder.RightBound==AreaCol-1||IsCollide(BlockPos.PosX,BlockPos.PosY,RIGHT))
  break;
  //Restore  the  left  scene  of  the  block,  in  order  to  clear  the  block  residue  during  the  movement
  for(int  i=CurBorder.UpBound;i<=CurBorder.DownBound;i++)
  GameArea[BlockPos.PosY+i][BlockPos.PosX]=0;
  BlockPos.PosX+=1;
  //Move  the  block  one  block  to  the  right  and  copy  it  to  the  scene
  for(int  i=CurBorder.UpBound;i<=CurBorder.DownBound;i++)
  for(int  j=0;j<4;j++)
  if(BlockPos.PosX+j<=AreaCol-1&&GameArea[BlockPos.PosY+i][BlockPos.PosX+j]!=2)  //Note  that  the  scene  array  is  not  out  of  bounds
  GameArea[BlockPos.PosY+i][BlockPos.PosX+j]=CurBlock[i][j];
  break;
  case  SPACE:  //once  to  the  end
  //Move  down  one  space  at  a  time,  until  you  can't  move  down
  while(BlockPos.PosY+CurBorder.DownBound<AreaRow-1&&!IsCollide(BlockPos.PosX,BlockPos.PosY,DOWN))
  {
  //Restore  the  scene  on  the  block,  in  order  to  clear  the  block  residue  during  the  movement
  for(int  j=CurBorder.LeftBound;j<=CurBorder.RightBound;j++)
  GameArea[BlockPos.PosY][BlockPos.PosX+j]=0;
  //If  there  is  no  collision,  it  will  fall  one  block
  BlockPos.PosY+=1;
  //The  square  drops  one  square,  copy  it  to  the  scene,  pay  attention  to  the  left  and  right  borders
  for(int  i=0;i<4;i++)  //Must  be  0  to  4
  for(int  j=CurBorder.LeftBound;j<=CurBorder.RightBound;j++)
  if(BlockPos.PosY+i<=AreaRow-1&&GameArea[BlockPos.PosY+i][BlockPos.PosX+j]!=2)  //Note  that  the  scene  array  is  not  out  of  bounds,  and  will  not  erase  stable  blocks
  GameArea[BlockPos.PosY+i][BlockPos.PosX+j]=CurBlock[i][j];
  }
  ConvertStable(BlockPos.PosX,BlockPos.PosY);
  ResetBlock();
  break;
  default:
  break;
  }
  //Process  the  elimination  of  lines,  and  the  lines  above  the  entire  scene  are  moved  down  in  turn
  int  i=AreaRow-1;
  int  LineCount=0;  //Number  of  deleted  lines
  while(i>=1)
  {
  bool  isLineFull=true;
  for(int  j=0;j<AreaCol;j++)
  if(GameArea[i][j]==0)
  {
  isLineFull=false;
  i--;
  break;
  }
  if(isLineFull)
  {
  for(int  k=i;k>=1;k--)
  for(int  j=0;j<AreaCol;j++)
  GameArea[k][j]=GameArea[k-1][j];
  LineCount++;//Increase  the  number  of  lines  to  be  eliminated  each  time
  }
  }
  ```
 - Eache time a line is eliminated,the score is moving up. In case there is a fixed shape appear in the top, then it's the game over widget rise.
 ![image](https://user-images.githubusercontent.com/80457241/152699532-fba1f5a0-ad9d-488f-aeaf-37bce5512b27.png)

```cpp 
  score+=LineCount*10;  //Score
  //Determine  if  the  game  is  over
  for(int  j=0;j<AreaCol;j++)
  if(GameArea[0][j]==2)  //There  is  also  a  stable  block  at  the  top
  GameOver();
} 
```
![image](https://user-images.githubusercontent.com/80457241/152699661-2cc8a613-63f5-4e77-ab4e-369b9e00b9a1.png)

- After each block is stable, we copy the next block to the current block to generate a new block to start diving  from the top.
```cpp
void  Widget::ResetBlock()
{
  //generate  current  block
  BlockCpy(CurBlock,NextBlock);
  GetBorder(CurBlock,CurBorder);
  //generate  the  next  block
  int  BlockID=rand()%7;
  CreateBlock(NextBlock,BlockID);
  //Set  the  initial  block  coordinates,  with  the  upper  left  corner  of  the  block  as  the  anchor  point
  BlockPoint  StartPoint;
  StartPoint.PosX=AreaCol/2-2;
  StartPoint.PosY=0;
  BlockPos=StartPoint;
}
```
#### ****Game control logic**
- Her we set the game initialization, initialization timer interval and randomization seed and score:
```cpp 
void  Widget::InitGame()
{
  for(int  i=0;i<AreaRow;i++)
  for(int  j=0;j<AreaCol;j++)
  GameArea[i][j]=0;
  Speed=700;
  Refresh=30;
  //Initialize  random  number  seed
  srand(time(0));
  //Score  clear  to  0
  score=0;
  //start  the  game
  StartGame();
}
```
- When the game starts, the blocks start appearing randomly:
```cpp
void  Widget::StartGame()
{
  GameTimer=startTimer(Speed);  //Start  the  game  timer
  PaintTimer=startTimer(Refresh);  //Enable  interface  refresh  timer
  //Generate  initial  next  block
  int  BlockID=rand()%7;
  CreateBlock(NextBlock,BlockID);
  ResetBlock();  //Generate  Blocks
}
```
- When ourgame is over, the timer stopped:
```cpp
void  Widget::GameOver()
{
  //game  over  stop  timer
  killTimer(GameTimer);
  killTimer(PaintTimer);
  QMessageBox::information(this,"Failed","GAME OVER");
}
```
## Conclusion
In this project we have used a lot of what we studied in Interfaces Hommes-Machines class with you sir. it was really a hard semester for me because i had problems with C++ cours ,plus working alone, but it was a good expirience. Because in the end am able to make this little project true .Am really happy about it, iwish you like it too sir.
with all respect 

