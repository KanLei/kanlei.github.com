---
layout: post
title: "Cosmos 节点运行和转账"
description: ""
category: 
tags: []
---
{% include JB/setup %}


### Cosmos 简介

Cosmos 是一个基于 BFT 共识算法，运行在 POS 系统上的去中心化网络，旨在提供区块链底层网络和共识，支持不同区块链间的跨链交互。更多内容请阅读 [Cosmos Intro](https://cosmos.network/intro)。

### 节点安装和启动

节点安装和运行可以参考 [Installation](https://cosmos.network/docs/gaia/installation.html)、[Run Mainnet](https://medium.com/cryptium-cosmos/how-to-install-cosmos-and-run-your-full-node-public-testnet-df886dc7b226)，macOS 上安装时可能存在以下问题

* `terminal` 需要配置代理
* 安装 [md5sha1sum](https://github.com/cosmos/cosmos-sdk/issues/3410#issue-403753260)
* 安装 [XCode Command Line Tools](https://github.com/ethereum/go-ethereum/issues/17940#issuecomment-434970303)


测试网 `genesis.json` 和 `seeds` 可以参考 [Testnets](https://github.com/cosmos/testnets)。

完成以上步骤后，会在 `$HOME` 路径下生成存放节点配置和区块信息的 **.gaiad** 目录，启动节点同步。

```bash
gaiad start
```

### 创建账号

> 如果需要离线生成账号，可将 `gaiacli` 拷贝到离线电脑，执行以下命令

Cosmos 账号采用 HD 结构，默认生成第0个账号

* `—-account num` 指定创建第 `num` 位置的账号
* `—-recover` 使用已有的助记词恢复账号

**account_name** 用于后续发送交易时使用，注意，此处的 **account_name** 只存在本地，表示一对公私钥，与 EOS 链上账号不同

```bash
gaiacli keys add <account_name>  # --recover --account num
```

输入加密账号的密码，并保存生成的**助记词**，查看账号列表

```bash
gaiacli keys list
```

查看账号信息（**需要先往该地址发送一笔转账，否则账号不会记录在区块链上**）

```
gaiad query account <pub_key>
```

### 单签转账

> **0.33.0** 版本 `gaiacli` 生成交易信息时需要读取 **keys.db** 目录，因此目前离线签名需要使用 REST Server

#### 在线生成交易、签名并广播

```bash
gaiacli tx send exchange_address 1muon --gas-prices=0.00001muon --from=account_name --node=http://ip:port --chain-id=gaia-13001
```

#### 在线生成交易、离线签名和在线广播

启动 REST Server 用于构造交易信息，参考 [rest-server](https://cosmos.network/docs/clients/lite/getting_started.html) 和 [setting-up-the-rest-server](https://cosmos.network/docs/clients/service-providers.html#setting-up-the-rest-server)

```bash
gaiacli rest-server --node=tcp://localhost:26657 --chain-id=gaia-13001 --laddr=tcp://ip:1317 --tls
```

使用 [REST API](https://cosmos.network/rpc/#/ICS20/post_bank_accounts__address__transfers) 构造交易信息，并保存到 unsignedTx.json 文件（`to_address` 指接收方地址）

```bash
curl -k -H "Content-Type: application/json" -d '{"base_req":{"from":"cosmos1we5hjfayx8yekv73cr3qqqvukpddxc7fa5najk","memo":"","chain_id":"gaia-13001","account_number":"0","sequence":"1","gas":"200000","gas_adjustment":"1.2","fees":[{"denom":"muon","amount":"1"}],"simulate":false, "generate_only":true},"amount":[{"denom":"muon","amount":"1"}]}' https://172.25.2.78:1317/bank/accounts/{to_address}/transfers > unsignedTx.json
```

离线签名交易，`--account-number` 和 `--sequence` 可通过查看账号信息获得

```bash
gaiacli tx sign unsignedTx.json --from=testaccount --chain-id=gaia-13001 --account-number=1053 --sequence=5 --offline > signedTx.json
```

广播交易

```bash
gaiacli tx broadcast signedTx.json
```

### 多签

生成多签账号

```bash
gaiacli keys add --multisig=mykeyname,mykeyname2 --multisig-threshold=2 multisig_keyname_12
```

发送一笔交易到多签地址

```bash
gaiacli tx send cosmos1jtkat7uncpm5qrw6hjvdg2x8cqzhtchnna3drd 1muon --chain-id=gaia-13001 --from=mykeyname
```

生成交易

```bash
gaiacli tx send cosmos1xschm0e3p6cwhvrcvv9jrfs8ffv4ggupelfscq 1muon --from=multisig_keyname_12 --generate-only > unsignedTx.json
```

生成签名 1

```bash
gaiacli tx sign --multisig=cosmos1jtkat7uncpm5qrw6hjvdg2x8cqzhtchnna3drd --name=mykeyname --output-document=name1signature.json unsignedTx.json
```

生成签名 2

```bash
gaiacli tx sign --multisig=cosmos1jtkat7uncpm5qrw6hjvdg2x8cqzhtchnna3drd --name=mykeyname2 --output-document=name2signature.json unsignedTx.json
```

合并签名

```bash
gaiacli tx multisign unsignedTx.json multisig_keyname_12 name1signature.json name2signature.json > signedTx.json
```

广播交易

```bash
gaiacli tx broadcast signedTx.json
```

### 使用 Ledger

首先，连接并解锁 Ledger，打开 Cosmos 应用

使用 Ledger 中的助记词生成账号

```bash
gaiacli keys add ledger_account —-ledger
```

在线发送交易

```bash
gaiacli tx send recipient_address 1muon -—chain-id=gaia-13002 —-from=ledger_account
```

离线签名交易

```bash
gaiacli tx sign unsignedTx.json --from=ledger_account --chain-id=gaia-13002 --account-number=1053 --sequence=1 --offline > signedTx.json
```


[*Mint Scan*](https://www.mintscan.io/)  
[*Documents*](https://cosmos.network/docs/)  
[*Cosmos Riot*](https://riot.im/app/#/room/#cosmos-validators:matrix.org)  
[*Cosmos Token Model*](https://github.com/cosmos/cosmos/blob/master/Cosmos_Token_Model.pdf)