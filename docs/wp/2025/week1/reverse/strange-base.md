---
titleTemplate: ":title | WriteUp - NewStar CTF 2025"
---

<script setup>
import Container from '@/components/docs/Container.vue'
</script>

# Strange Base

<Container type='info'>

本题考查 Base64 编解码及其变种的逆向分析。
</Container>

没去符号表，一眼就能发现 base64，密文是 `T>6uTqOatL39aP!YIqruyv(YBA!8y7ouCa9=`

![密文](/assets/images/wp/2025/week1/strange-base_1.png)

进入分析看到编码表是：

`HElLo!A=CrQzy-B4S3|is'waITt1ng&Y0u^{/(>v<)*}GO~256789pPqWXVKJNMF`

![编码表](/assets/images/wp/2025/week1/strange-base_2.png)

所以我们找一个 base 解密脚本即可解密

```c
#include <stdio.h>
#include <string.h>
#include <stdint.h>
#include <stdlib.h>
const char * const base64char = "HElLo!A=CrQzy-B4S3|is'waITt1ng&Y0u^{/(>v<)*}GO~256789pPqWXVKJNMF";
int base64_decode( const char * base64, unsigned char * bindata )
{
    int i, j;
    unsigned char k;
    unsigned char temp[4];

    for ( i = 0, j = 0; base64[i] != '\0' ; i += 4 )
    {
        memset( temp, 0xFF, sizeof(temp) );
        for ( k = 0 ; k < 64 ; k ++ )
        {
            if ( base64char[k] == base64[i] )
                temp[0] = k;
        }
        for ( k = 0 ; k < 64 ; k ++ )
        {
            if ( base64char[k] == base64[i + 1] )
                temp[1] = k;
        }
        for ( k = 0 ; k < 64 ; k ++ )
        {
            if ( base64char[k] == base64[i + 2] )
                temp[2] = k;
        }
        for ( k = 0 ; k < 64 ; k ++ )
        {
            if ( base64char[k] == base64[i + 3] )
                temp[3] = k;
        }

        bindata[j++] = ((unsigned char)(((unsigned char)(temp[0] << 2)) & 0xFC)) |
                       ((unsigned char)((unsigned char)(temp[1] >> 4) & 0x03));
        if ( base64[i + 2] == '=' )
            break;

        bindata[j++] = ((unsigned char)(((unsigned char)(temp[1] << 4)) & 0xF0)) |
                       ((unsigned char)((unsigned char)(temp[2] >> 2) & 0x0F));
        if ( base64[i + 3] == '=' )
            break;

        bindata[j++] = ((unsigned char)(((unsigned char)(temp[2] << 6)) & 0xF0)) |
                       ((unsigned char)(temp[3] & 0x3F));
    }
    return j;
}

int main()
{
    unsigned char input[0x20] = {0};
    unsigned char output[0x20] = {0};
    unsigned char enc[] = "T>6uTqOatL39aP!YIqruyv(YBA!8y7ouCa9H";
    base64_decode(enc, output);
    printf("%s\n", output);
}
```

运行脚本得到 flag：`flag{Wh4t_a_cra2y_8as3!!!}`
