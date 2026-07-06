---
titleTemplate: ":title | WriteUp - NewStar CTF 2025"
---

# BLS 多重签名：零的裂变

这是一个关于 BLS（Boneh-Lynn-Shacham）签名系统中多重签名漏洞的题目。

### BLS 签名基础知识

1. BLS 签名基本概念

BLS 是一种基于椭圆曲线配对的数字签名方案。私钥 $sk$ 是一个随机数，公钥 $pk = sk \times G$，其中 $G$ 是椭圆曲线的生成元。签名 $\sigma = sk \times H(m)$，其中 $H(m)$ 是将消息映射到椭圆曲线上的哈希函数。

2. BLS 签名的线性特性

BLS 签名有一个重要的数学特性是线性。这意味着，公钥可以叠加，即 $pk_1 + pk_2 = (sk_1 + sk_2) \times G$；签名也可以叠加，即 $\sigma_1 + \sigma_2 = (sk_1 + sk_2) \times H(m)$。

3. 多重签名验证

在多重签名中，多个参与者共同对同一消息签名时，聚合公钥等于所有参与者的公钥之和，聚合签名等于所有参与者的签名之和，验证时检查 $e(\text{聚合公钥}, H(m)) = e(G, \text{聚合签名})$。

### 题目漏洞分析

关键漏洞：未检测零公钥攻击

题目中的关键检查：

```python
# 检查聚合公钥是否等于服务器公钥
agg_pk = bls._AggregatePKs(pks_bytes)
if agg_pk != self.pk_server:
    return False, "Aggregate PK must equal server PK"
```

这里要求我们提供的三个公钥的聚合结果必须等于服务器的公钥。正常情况下这很难做到，但利用 BLS 的线性特性，我们可以构造特殊的密钥对来绕过这个检查。

如果我们能找到两个私钥 $sk_1$ 和 $sk_2$，使得 $sk_1 + sk_2 = 0 \pmod{order}$。

那么对应的公钥 $pk_1 + pk_2 = (sk_1 + sk_2) \times G = 0 \times G = 零点$。

这样，如果我们构造：用户公钥 1 为 $pk_1$，用户公钥 2 为 $pk_2$，服务器公钥为 $pk_{server}$。

那么三个公钥的聚合结果为 $pk_1 + pk_2 + pk_{server} = 零点 + pk_{server} = pk_{server}$。

这样就绕过了聚合公钥的检查。

### 具体解题步骤

1. 生成攻击密钥对

```python
# 选择任意私钥值
x1 = 1337
# 计算对应的公钥
pk1 = bls.SkToPk(x1)

# 计算另一个私钥，使得两个私钥和为 0（模 order）
x2 = order - x1
pk2 = bls.SkToPk(x2)
```

这里 $x_1 + x_2 = order \equiv 0 \pmod{order}$，所以 $pk_1 + pk_2 = 零点$。

2. 注册两个公钥

分别用两个私钥对消息 `"POP"` 签名，生成证明（Proof of Possession），然后注册两个公钥。

```python
# 为第一个公钥生成 POP
pop1 = bls.Sign(x1, b"POP").hex()
# 注册第一个公钥
发送: {'type': 'register', 'pk': pk1_hex, 'pop': pop1}

# 为第二个公钥生成 POP
pop2 = bls.Sign(x2, b"POP").hex()
# 注册第二个公钥
发送: {'type': 'register', 'pk': pk2_hex, 'pop': pop2}
```

3. 获取服务器签名

```python
# 请求服务器对"get_flag"签名
发送: {'type': 'sign', 'msg': 'get_flag'}
接收: 服务器对"get_flag"的签名
```

4.构造攻击获取 FLAG

```python
# 构造三个公钥的列表
pks_list = [pk1_hex, pk2_hex, server_pk_hex]

# 提交获取 FLAG 的请求
发送: {
    'type': 'get_flag',
    'pks': pks_list,
    'sig': server_sig_hex
}
```

完整 EXP：

```python
#!/usr/bin/env python3
from pwn import *
import json
import secrets
from py_ecc.bls import G2ProofOfPossession as bls

# BLS12-381 曲线参数
order = 52435875175126190479447740508185965837690552500527637822603658699938581184513

def solve(host='127.0.0.1', port=9999):
    # 连接服务器
    io = remote(host, port)

    try:
        # 1. 获取服务器信息
        io.sendline(json.dumps({'type': 'get_info'}).encode())
        response = json.loads(io.recvline().decode())

        if not response['success']:
            log.error(f"Failed to get server info: {response['message']}")
            return

        info = response['message']
        server_pk_hex = info['server_pk']
        log.info(f"Server PK: {server_pk_hex}")
        log.info(f"Server PK length: {len(server_pk_hex)}")

        # 2. 生成攻击密钥对
        log.info("Generating malicious key pair...")
        x1 = 1337  # 先用固定值测试
        pk1 = bls.SkToPk(x1)
        pk1_hex = pk1.hex()

        x2 = order - x1
        pk2 = bls.SkToPk(x2)
        pk2_hex = pk2.hex()

        log.info(f"PK1: {pk1_hex}")
        log.info(f"PK1 length: {len(pk1_hex)}")
        log.info(f"PK2: {pk2_hex}")
        log.info(f"PK2 length: {len(pk2_hex)}")

        # 3. 注册第一个公钥
        log.info("Registering PK1...")
        pop1 = bls.Sign(x1, b"POP").hex()
        log.info(f"POP1 length: {len(pop1)}")

        io.sendline(json.dumps({
            'type': 'register',
            'pk': pk1_hex,
            'pop': pop1
        }).encode())

        response = json.loads(io.recvline().decode())
        log.info(f"Register PK1 response: {response}")
        if not response['success']:
            log.error(f"Register PK1 failed: {response['message']}")
            return
        log.success("PK1 registered")

        # 4. 注册第二个公钥
        log.info("Registering PK2...")
        pop2 = bls.Sign(x2, b"POP").hex()
        log.info(f"POP2 length: {len(pop2)}")

        io.sendline(json.dumps({
            'type': 'register',
            'pk': pk2_hex,
            'pop': pop2
        }).encode())

        response = json.loads(io.recvline().decode())
        log.info(f"Register PK2 response: {response}")
        if not response['success']:
            log.error(f"Register PK2 failed: {response['message']}")
            return
        log.success("PK2 registered")

        # 5. 获取服务器对 "get_flag" 的签名
        log.info("Getting server signature for 'get_flag'...")
        io.sendline(json.dumps({'type': 'sign', 'msg': 'get_flag'}).encode())
        response = json.loads(io.recvline().decode())
        log.info(f"Sign response: {response}")

        if not response['success']:
            log.error(f"Get signature failed: {response['message']}")
            return

        server_sig_hex = response['message']
        log.info(f"Server signature: {server_sig_hex}")
        log.info(f"Server signature length: {len(server_sig_hex)}")

        # 6. 构造攻击并获取 FLAG
        log.info("Constructing attack...")
        pks_list = [pk1_hex, pk2_hex, server_pk_hex]

        log.info("Submitting PKs:")
        for i, pk in enumerate(pks_list):
            log.info(f"  PK{i + 1}: {pk[:32]}... (len: {len(pk)})")

        io.sendline(json.dumps({
            'type': 'get_flag',
            'pks': pks_list,
            'sig': server_sig_hex
        }).encode())

        response = json.loads(io.recvline().decode())
        log.info(f"Get FLAG response: {response}")

        if response['success']:
            FLAG = response['message']
            log.success(f"🎉 FLAG: {FLAG}")
        else:
            log.error(f"Failed to get FLAG: {response['message']}")

    except Exception as e:
        log.error(f"Error: {e}")
        import traceback
        traceback.print_exc()
    finally:
        io.close()

if __name__ == '__main__':
    import argparse

    parser = argparse.ArgumentParser(description='BLS Splitting Zero Attack')
    parser.add_argument('--host', default='47.104.154.99', help='Server host')
    parser.add_argument('--port', type=int, default=9999, help='Server port')

    args = parser.parse_args()

    context.log_level = 'info'

    solve(host=args.host, port=args.port)
```
