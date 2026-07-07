---
titleTemplate: ":title | WriteUp - NewStar CTF 2025"
---

# TIME HACKER

> 我所害怕的，只有时间

> [Misc] TIME HACKER.zip

这道题的知识点包含侧信道攻击、时间戳隐写和 EXIF 信息读取。

首先将压缩包文件拖入 010 Editor，发现文件尾部存在冗余的 Base64 编码数据，解码后可得到提示信息：

> I heard you can hack the time in different dimensions?

![压缩包尾部冗余数据](/assets/images/wp/2025/week5/time-hacker_1.png)
![Base64 解码提示](/assets/images/wp/2025/week5/time-hacker_2.png)
下面继续分析该提示的含义。

`password_checker.exe` 用于校验压缩包密码是否正确。
![password_checker 程序](/assets/images/wp/2025/week5/time-hacker_3.png)
直接逆向该程序获取密码并不现实，因为该 exe 加了 VMProtect 壳。

我们再把 FLAG.zip 文件拖入 010 尾部依旧存在冗余数据，解码后可获得提示

![FLAG.zip 尾部提示](/assets/images/wp/2025/week5/time-hacker_4.png)

> The password is ten digits

![密码长度提示](/assets/images/wp/2025/week5/time-hacker_5.png)
此时可以考虑直接爆破密码，但 ARCHPR 预估时间达到 7 天，无法在比赛时间内完成。

因此，前面提示中「hack the time」指向的是通过侧信道时间攻击获取压缩包密码。

输入密码后，程序会延迟一段时间再输出正确或错误，这正是漏洞所在。经过测试，可以推测后端校验逻辑大致如下：

```python
a = input()
def checker(a):
    if len(a) != len(secret):
        return False
    for i in range(len(a)):
        if(a[i] != secret[i]):
            (some operations that make time pass.)
            return False
    return True
```

因此只需要编写脚本反复调用 `password_checker.exe`，按响应时间逐位爆破密码即可。

```python
import time
import subprocess

def crack_password():
    password = ""
    exe_path = "exe_path"
    for i in range(10):
        times = {}
        for c in "0123456789":
            test = password + c + '0' * (9 - i)
            total_time = 0

            for _ in range(3):
                start = time.time()
                subprocess.run([exe_path], input=test.encode(), capture_output=True)
                total_time += time.time() - start
            times[c] = total_time / 3
        print(times)
        best_char = max(times, key=times.get)
        password += best_char
        print(f"第{i+1}位: {best_char} -> {password}")
    print(f"密码: {password}")
    return password

if __name__ == "__main__":
    crack_password()
```

得到密码 `2145768093`。

> 有选手通过猜测密码与被压缩文件的时间戳有关，在其时间戳附近爆破也得到了密码；这实际上与下一阶段有关。

观察解压后的图片。
![解压后的图片文件](/assets/images/wp/2025/week5/time-hacker_6.png)

> 一个文件包含三个时间：创建时间、访问时间、修改时间。

可以注意到修改时间异常，推测出题人通过该时间隐藏了信息。将其转换为时间戳后发现与密码值相差不大，因此很可能通过二者差值隐藏信息。打印差值：

```python
import os
import time

for i in range(1,37):
    mTime = os.stat(f"D://CTF//newstar//newstar2025 misc wp//[Misc] TIME HACKER//FLAG//{i}.png").st_mtime
    print(chr(int(mTime)-2145768093),end="")
```

此时已经能看出 FLAG 字符，但字符顺序不正确。图片的 EXIF 信息也是常见的信息隐藏载体，因此可以从 EXIF 中读取文件顺序。最终脚本如下：

```python
import piexif
from PIL import Image
import os
import time

def exif_read(file_path):
    img = Image.open(file_path)
    exif_dict = piexif.load(img.info["exif"])
    return exif_dict["Exif"][36867][-2:]
shunxu = []
for i in range(1,37):
    shunxu.append(int(exif_read(f"pngpath")))
print(shunxu)
FLAG = [""] * 50
for i in range(1,37):
    mTime = os.stat(f"pngpath").st_mtime
    print(mTime)
    FLAG[shunxu[i-1]] = chr(int(mTime)-2145768093)
strr = "".join(FLAG)
print(strr)
#FLAG{Y0u_4re_tHe_ReaL_TIME_H@cker!!}
```
