---
titleTemplate: ":title | WriteUp - NewStar CTF 2025"
---

<script setup>
import Container from '@/components/docs/Container.vue'
import SVGStackCaller from '@docs/assets/images/wp/2025/stack_caller.svg?component'
import SVGStackCallee from '@docs/assets/images/wp/2025/stack_callee.svg?component'
import SVGStackPush from '@docs/assets/images/wp/2025/stack_push.svg?component'
import SVGStackPop from '@docs/assets/images/wp/2025/stack_pop.svg?component'
import SVGStackLeave1 from '@docs/assets/images/wp/2025/stack_leave_mov_rsp_rbp.svg?component'
import SVGStackLeave2 from '@docs/assets/images/wp/2025/stack_leave_pop_rbp.svg?component'
import SVGStackRet from '@docs/assets/images/wp/2025/stack_ret_pop_rip.svg?component'
import SVGStackFuncStart1 from '@docs/assets/images/wp/2025/stack_func-start_push_rbp.svg?component'
import SVGStackFuncStart2 from '@docs/assets/images/wp/2025/stack_func-start_mov_rbp_rsp.svg?component'
</script>

# overflow

<Container type='info'>

本题考查栈溢出和对栈结构的理解。
</Container>

在本题开始之前，你需要先了解栈结构与栈溢出。

## 栈结构

栈是一种典型的后进先出（Last In First Out）的数据结构，它的基本结构是栈顶<span data-desc>（rsp 寄存器的值）</span>和栈底<span data-desc>（rbp 寄存器的值）</span>。其操作主要有压栈（push）与出栈（pop）两种操作。push 可以把寄存器里的值或者一个立即数压入栈，pop 则是把 rsp 寄存器<span data-desc>（也就是栈顶）</span>指向的值弹出给一个寄存器。

push 操作：

<SVGStackPush />

pop 操作：

<SVGStackPop />

以 64 位操作系统为例，对于函数调用栈来说，`[rbp+8]` 的位置会跟随一个 `return addr`，存储函数的返回地址。

## 函数调用栈

<Container type='warning'>

理解本节时，请严格区分「地址」「值」以及「指针」的概念。
</Container>

假设有以下函数调用关系：

```c
void send() {
    // ...
    return;
}

void entry() {
    send();
}
```

函数调用栈的栈空间示意图如下，左图为 `entry` 函数调用 `send` 函数前的栈空间状态，右图为调用 `send` 函数后某一时刻的栈空间状态。

<div style="display: flex; flex-direction: row; align-items: end;">

<SVGStackCaller />

<SVGStackCallee />

</div>

函数调用栈中，帧指针<span data-desc>（栈底指针）</span> `rbp` 指向的位置<span data-desc>（存储的地址）</span>存储着上一个函数<span data-desc>（调用者）</span>的栈底位置<span data-desc>（地址）</span>。如上图中，「previous rbp」的值为粉色的<span style="color: #D86ECC">「previous rbp」</span>的地址。

而返回地址 `return addr` 的位置<span data-desc>（`[rbp+8]`）</span>存储着调用该函数的下一条指令地址<span data-desc>（地址）</span>。

栈底元素和返回地址共同承担了函数 return 后「恢复现场」的职责。

### 函数返回

当函数 return 时，程序会执行 `leave` 和 `ret` 指令。

`leave` 等价于：

```asm
mov rsp, rbp    ; 将 rbp 的值存储到 rsp 寄存器
pop rbp         ; 弹出当前栈顶元素，并保存到 rbp 寄存器
```

> [!TIP]
>
> `pop xxx` 完成了两个操作：存储栈顶元素到 xxx，改变了栈指针（栈顶指针）。
>
> 以 64 位操作系统为例，等价于：
>
> ```asm
> mov xxx, [rsp]  ; 将栈顶的值（rsp 指针指向的值）加载到 xxx 寄存器
> add rsp, 8      ; 栈指针向上移动 8 字节（64 位）
> ```

`mov rsp, rbp` 最终将栈指针<span data-desc>（栈顶指针）</span>恢复到帧指针<span data-desc>（栈底指针）</span>的位置。

<SVGStackLeave1 />

`pop rbp` 最终将帧指针<span data-desc>（栈底指针）</span>变为上一个函数的栈底位置。

<SVGStackLeave2 />

这样，通过 `leave` 指令，栈就恢复到调用该函数之前的状态。

`ret` 等价于：

```asm
pop rip    ; 弹出当前栈顶元素，并保存到指令指针寄存器 rip
```

`rip` 寄存器存储着下一条将要执行的指令地址。

<SVGStackRet />

于是，再通过 `ret` 指令，程序跳转到 `return addr` 处继续执行。

### 函数调用

了解了返回操作后，我们再来理解函数调用时栈的变化，则是「保存现场」。「保存现场」需要保存两个内容：调用者的栈底位置和返回地址。

返回地址的保存由 `call` 指令完成的，它位于调用函数<span data-desc>（在上例代码中为 `entry`）</span>的诸多指令中。

假设有一个函数 `func`，它的（起始）地址为 `xxx`. `call xxx` 指令等价于：

```asm
push rip + 5      ; 将下一条指令地址压入栈中作为返回地址（5 代表指令长度）
jmp xxx           ; 跳转到 xxx 地址执行
```

而 `func` 函数开始时，通常会有如下两条指令：

```asm
push rbp          ; 将调用者的栈底位置弹入
mov rbp, rsp      ; 设置当前函数的栈底位置
```

`push rbp` 完成了调用者栈底位置的保存。

<SVGStackFuncStart1 />

`mov rbp, rsp` 设置了当前函数的栈底位置。

<SVGStackFuncStart2 />

这样，函数调用时栈完全切换到了被调用函数栈空间应有的状态。

## 栈溢出

栈溢出漏洞：通过向栈中写入超出预期的数据，覆盖返回地址或其他关键数据，从而劫持程序的执行流。

由于数组的低索引存储在栈的低地址<span data-desc>（靠近栈顶）</span>，高索引存储在栈的高地址<span data-desc>（靠近栈底）</span>，因此当向数组写入过多数据发生溢出时，可以覆盖到 `previous rbp` 和 `return addr` 等位于栈的更高地址的内容。

## 解题

将附件拖入 IDA Pro 中，按下 <kbd>F5</kbd> 进行反编译，查看 `main` 函数：

![main 函数](/assets/images/wp/2025/week1/overflow_3.png)

双击 `try` 函数，跟进查看，里面有一个 `gets` 函数，这个函数不限制读入的长度，就会造成栈溢出。

![try 函数](/assets/images/wp/2025/week1/overflow_4.png)

本题还有一个后门函数 `backd00r` 可以执行 shell.

![backd00r 函数](/assets/images/wp/2025/week1/overflow_5.png)

那么接下来思路就很清晰了，先填垃圾数据把栈填满，然后把 `return addr` 的位置写上我们的后门函数的地址就能顺利执行 `system("/bin/sh")`.

在反编译界面按 <kbd>TAB</kbd> 切换到汇编界面，再按 <kbd>Space</kbd> 可以切换视图。

![汇编视图](/assets/images/wp/2025/week1/overflow_6.png)

`backd00r` 函数的地址是 `0x401200`，但是直接填这个地址会造成栈不对齐的问题，所以我们填 `0x401201`<span data-desc>（不需要进行 `push rbp`）</span>。

> [!INFO] 栈对齐
> 在某些系统架构中，栈需要保持特定的对齐方式（例如 16 字节对齐），以提高内存访问效率和性能。如果栈未正确对齐，可能会导致程序崩溃或性能下降。因此，在进行栈操作时，确保栈指针（rsp）保持适当的对齐是非常重要的。
>
> 「+1 偏移」通常用于跳过函数开头的 `push rbp` 指令。

于是我们要先将 `buffer` 的 256 字节填满，再填 8 字节覆盖 `rbp`，最后填上 `0x401201` 即可。

附解题脚本：

```python
from pwn import *

r = remote('IP', PORT)
# r = process('./overflow')
payload = b'A' * (256 + 8) + p64(0x401201)
r.sendline(payload)
r.interactive()
```
