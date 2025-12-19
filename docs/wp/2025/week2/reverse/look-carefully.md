---
titleTemplate: ":title | WriteUp - NewStar CTF 2025"
---

<script setup>
import Container from '@/components/docs/Container.vue'
</script>

# Look at me carefully

<Container type='info'>

本题考查对经过混淆后的代码的理解。
</Container>

首先打开 `main` 函数分析，发现 `sub_4016E0` 这个函数的调用似乎很多

![main 函数分析](/assets/images/wp/2025/week2/look-carefully_1.png)

函数内部插入了大量的无意义的混淆代码，静态很难分析。

![混淆代码](/assets/images/wp/2025/week2/look-carefully_2.png)

其实我们可以动态调试观察函数的两个参数指针的变化。~~根据题目提示其实这道题是不需要仔细看其中的逻辑的~~

首先我们输入 `abcdefghijklmnopqrstuvwxyz0123456789`，用于比较的是我们函数的第一个参数，我们在下方内存窗口跳转过去，然后观察内存变化。

![动态调试](/assets/images/wp/2025/week2/look-carefully_3.png)

我们执行了几个函数，观察内存窗口，发现该内存多了如下输入 `1fgj2s63`，根据和输入的观察发现是从输入中按照一定的顺序取出字符放到新的一片内存中，而顺序就是函数的第三个参数，而数组下标超出输入的长度的函数是不会进行取输入操作的。

![内存变化](/assets/images/wp/2025/week2/look-carefully_4.png)

所以我们可以根据这个规则逆推出 flag

```python
# 目标字符串
target = list("cH4_1elo{ookte?0dv_}alafle___5yygume??")
order = [
    27, 5, 6, 9, 28, 18, 32, 29, 4, 11, 15, 17, 22, 8, 34, 16, 19, 7, 26, 35,
    2, 14, 21, 0, 1, 25, 13, 23, 20, 30, 33, 10, 3, 12, 24, 31
]

flag = [''] * 36
for i, idx in enumerate(order):
    flag[idx] = target[i]

print("".join(flag))
```

得到flag：`flag{H4ve_you_lo0ked_at_me_c1o5ely?}`。
