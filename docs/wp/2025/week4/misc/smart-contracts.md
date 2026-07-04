---
titleTemplate: ":title | WriteUp - NewStar CTF 2025"
---

# 区块链：智能合约

.sol 文件编译一下然后获取题目给出的地址

合约地址：0x88DC8f1de5Ff74d644C1a1defDc54869E5Ce3c08

unlock 操作会输入一个数，等于 0x0721 就会将当前地址的状态设置为 true

当地址状态为 true 时 getFlag 就可以拿到 FLAG

题目给出的代码：

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.0;

contract SimpleVault {
    string private FLAG = "FLAG{fake_flag}";
    uint256 private password = 0x0721;

    // 使用映射来记录每个地址的解锁状态
    mapping(address => bool) public unlocked;

    function unlock(uint256 _password) external {
        // 检查当前调用者是否已经解锁，如果已经解锁，则无需再次操作
        require(!unlocked[msg.sender], "Already unlocked!");
        if (_password == password) {
            // 只修改当前调用者（msg.sender）的解锁状态
            unlocked[msg.sender] = true;
        }
    }

    function getFlag() external view returns (string memory) {
        // 检查当前调用者是否已解锁
        require(unlocked[msg.sender], "Vault is locked. Unlock it first!");
        return FLAG;
    }
}
```

创建文件后编译

![Remix 部署合约](/assets/images/wp/2025/week4/smart-contracts_1.png)

编译后通过地址加载合约，0x88DC8f1de5Ff74d644C1a1defDc54869E5Ce3c08

![调用合约函数](/assets/images/wp/2025/week4/smart-contracts_2.png)

进行unlock操作，输入0x0721

![获得合约题 FLAG](/assets/images/wp/2025/week4/smart-contracts_3.png)
