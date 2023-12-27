# 简介

是一个 Linux 下终端的图形库，可以在终端绘制图形界面

1. 在终端中进行绘制
2. 基于字符的界面

```bash
sudo apt-get install libncurses5-dev
```

# 核心

`curses` 使用两个数据结构进行显示处理：
1. `stdscr`: 逻辑屏幕，实时的
2. `curscr`: 物理屏幕，调用 `refresh` 函数时才会刷新

# 基础

## 初始化

```cpp
WINDOW *initscr(void);
```
- 必须以此函数开始，返回 `stdscr` 的指针

## 销毁

```cpp
int endwin(void);
```
- 返回值为是否成功

# 功能

## 输出屏幕

```cpp
//当前位置添加字符
int addch(const chtype char_to_add);

//当前位置添加字符串
int addchstr(chtype *const string_to_add);

//类似与printf
int printw(char *format, ...);

//刷新物理屏幕
int refresh(void);

//围绕窗口绘制方框，用vertical绘制垂直边，用horizontal绘制水平边
int box(WINDOW *win_ptr, chtype vertical, chtype horizontal);

//插入一个字符（已有字符后移)
int insch(chtype char_to_insert);   

//插入空白行
int insertln(void);

//删除光标左边的字符
int delch(void);

//删除空白行
int deleteln(void);

//终端响铃
int beep(void);

//闪烁
int flash(void);
```
- `chtype` 是一个整数类型，高位表示 `ASCII`，低位表示属性，如：`'X' | A_BOLD` 即为 X 字符并带有 A_BOLD 属性

属性：
- `A_NORMAL`：默认属性。
- `A_STANDOUT`：突出显示，通常为反白显示。
- `A_REVERSE`：反显，即字符的前景色和背景色交换。
- `A_BLINK`：闪烁，即字符交替闪烁。
- `A_DIM`：暗淡，即字符降低亮度。
- `A_BOLD`：加粗。
- `A_UNDERLINE`：下划线。
- `A_PROTECT`：保护，即字符不可修改。
- `A_INVIS`：不可见，即字符不可见。
- `A_ALTCHARSET`：使用替代字符集（如 ASCII 字符集以外的字符集）。

## 读取屏幕

```cpp
//返回光标位置字符;
chtype inch(void);

//读取字符到string所指向的字符串中
int instr(char *string);

//读取numbers个字符到string所指向的字符串中
int innstr(char *string, int numbers);
```

## 清除屏幕

```cpp
//在屏幕的每个位置写上空白字符
int erase(void);

//使用一个终端命令来清除整个屏幕，相当于vi内的Ctrl+L
//内部调用了clearok来执行清屏操作,(在下次调用refresh时可以重现屏幕原文）
int clear(void);

//清除光标位置到屏幕结尾的内容
int clrtobot(void);

//清除光标位置到该行行尾的内容
int clrtoeol(void);
```

## 移动光标

```cpp
//移动stdcsr的光标位置
int move(int new_y, int new_x);

//设置一个标志，用于控制在屏幕刷新后curses将物理光标放置的位置。
int leaveok(WINDOW *window_ptr,bool leave_flag);
```

## 键盘模式

```cpp
//用于开启键盘输入字符的回显
int echo();

//用于关闭键盘输入字符的回显
int noecho();

//完成initscr后，输入模式为预处理模式，
//（1）所有处理是基于行的，就是说，只有按下	回车，输入数据才被传给程序；
//（2）键盘特殊字符启用，按下合适组合键会产生信号

//设置cbreak模式，字符一键入，直接传给程序
int cbreak();

//关闭
int nocbreak();

//关闭特殊字符处理
int raw();

//同时h恢复默认模式和特殊字符处
int noraw();
```

## 键盘输入

```cpp
//与标准io库的getchar, gets, scanf类似

//读取一个字符
int getch();

//读取一个字符串
int getstr(char *string);

//建议使用
int getnstr(char *string, int number);

int scanw(char*format,...);
```

## 窗口管理

```cpp
//创建从(start_y, start_x)开始的lines行，cols列的窗口。
WINDOW *newwin(int lines, int cols, int start_y, int start_x);   

//销毁上面创建的窗口，千万不要删除stdscr和curscr！
int delwin(WINDOW *window);

int mvwin(WINDOW *win, int new_y, int new_x);   //移动窗口
int wrefresh(WINDOW *win);
int wclear(WINDOW *win);
int werase(WINDOW *win);
//类似于上面的refresh, clear, erase，但是此时针对特定窗口操作，而不是stdcur

//指定该窗口内容已改变、
//下次wrefresh时，需重绘窗口。利用该函数，安排要显示的窗口
int touchwin(WINDOW *win);

//指定是否允许窗口卷屏
int scrollok(WINDOW *win, bool flag);    

//把窗口内容上卷一行
int scroll(WINDOW *win);

//子窗口除了没有自己的屏幕字符存储空间外，其他与新窗口相同。

//创建子窗口。
WINDOW *subwin(WINDOW *parent, int lines, int cols, int start_y, int start_x);

//销毁子窗口
int delwin(WINDOW *window);
```

## 彩色显示

```cpp
bool has_colors(void);

int start_color(void);

//eg: init_pair(1,COLOR_RED,COLOR_GREEN)，将红色前景绿色背景定义为一号颜色组合。
int init_pair(short pair_number,short foreground,short background);

//eg: COLOR_PAIR(1)，作为属性来访问，同A_BOLD
int COLOR_PAIR(int pair_number);

//和init_pair相反，通过颜色组合号，来获得颜色。
int pair_content(short pair_number,short *foreground,short *background);

//通过red green blue组合颜色
int init_color(short color_number,short red,short green,short blue);
```

## 获取位置

```cpp
//getyx()是一个宏
//得到目前游标的位置. (请注意! 是 y,x 而不是&y,&x )
void getyx(WINDOW *win,int y,int x);

//用于取得子窗口相对主窗口的起始坐标，它在更新子窗口时经常使用。
void getparyx(WINDOW *win, int y, int x);

//函数返回win窗口中最大的行数和最大的列数
void getmaxyx(WINDOW *win, int y, int x);
```