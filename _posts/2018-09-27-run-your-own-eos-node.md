---
layout: post
title: "EOS 节点搭建与合约部署"
description: ""
category: 
tags: []
---
{% include JB/setup %}

### 节点搭建

#### 安装

安装过程可以参考[官网](https://developers.eos.io/eosio-nodeos/docs/getting-the-code)，这里不做过多介绍，只记录几个需要注意的问题。

* 注意目前支持的操作系统和配置(7GB RAM、20GB Disk)
* curl 下载 Boost libraries 失败需要翻墙并配置 curl 代理
* 如果你使用虚拟机，需要在宿主机中开启代理如 shadowsocks，并开启代理端口访问，在虚拟机中配置宿主机代理地址
* Ubuntu 推荐使用 [16.xx](http://releases.ubuntu.com/)
* 执行 ./eosio_build.sh 时可以通过 -s EOS 指定 CoreSymbol

#### 启动

运行 `nodeos`，会在生成如下[配置目录](https://developers.eos.io/eosio-nodeos/docs/configuration-file)

* Mac OS: ~/Library/Application Support/eosio/nodeos/config
* Linux: ~/.local/share/eosio/nodeos/config

##### 开发环境(本地开发测试)

`nodeos` 默认生成的配置信息可用于开发环境，详细参考[官网步骤](https://developers.eos.io/eosio-nodeos/docs/setup-nodeos-for-development)

```
nodeos -e -p eosio --plugin eosio::chain_api_plugin --plugin eosio::history_api_plugin
```

##### 主网环境(同步主网数据)

修改上面生成的配置目录中 **config.ini** 文件

* 增加 p2p-peer-address 配置

节点列表数据可以参考：([*source1*](https://eosnodes.privex.io/)、[*source2*](https://docs.google.com/spreadsheets/d/1K_un5Vak3eDh_b4Wdh43sOersuhs0A76HMCfeQplDOY/edit#gid=0))

* 允许外部网络访问

http-server-address = 127.0.0.1:8888

* 增加 plugin 配置

```
plugin = eosio::chain_plugin
plugin = eosio::chain_api_plugin
plugin = eosio::net_plugin
plugin = eosio::net_api_plugin
plugin = eosio::history_plugin
plugin = eosio::history_api_plugin
plugin = eosio::http_plugin
plugin = eosio::http_client_plugin
```

在 nodeos 目录下新增 **genesis.json**（[*source1*](https://github.com/EOS-Mainnet/eos/blob/launch-rc-1.0.2/mainnet-genesis.json) [*source2*](https://eosnodes.privex.io/)）文件，内容如下：

```
{
  "initial_timestamp": "2018-06-08T08:08:08.888",
  "initial_key": "EOS7EarnUhcyYqmdnPon8rm7mBCTnBoot6o7fE2WzjvEX2TdggbL3",
  "initial_configuration": {
    "max_block_net_usage": 1048576,
    "target_block_net_usage_pct": 1000,
    "max_transaction_net_usage": 524288,
    "base_per_transaction_net_usage": 12,
    "net_usage_leeway": 500,
    "context_free_discount_net_usage_num": 20,
    "context_free_discount_net_usage_den": 100,
    "max_block_cpu_usage": 200000,
    "target_block_cpu_usage_pct": 1000,
    "max_transaction_cpu_usage": 150000,
    "min_transaction_cpu_usage": 100,
    "max_transaction_lifetime": 3600,
    "deferred_trx_expiration_window": 600,
    "max_transaction_delay": 3888000,
    "max_inline_action_size": 4096,
    "max_inline_action_depth": 4,
    "max_authority_depth": 6
  }
}
```

第一次运行需要指定 **--genesis-json**，后续运行不需要指定；**--delete-all-blocks** 清空已同步区块

```
nodeos --genesis-json genesis.json --delete-all-blocks
```

打开新的终端，并输入

```
cleos get info
```

查看 **chain_id** 与下面这串字符相同

```
aca376f206b8fc25a6ed44dbdc66547c36c6c33e3a119ffbeaef943642f0e906
```

如果相同的话，则证明我们成功连接到主网，否则，需要检查 **genesis.json** 配置是否正确，以及 **p2p-peer-address** 中节点列表是否能够正常连接。

最后，我们可以通过 **head\_block\_num** 参数值来确认是否成功同步新区块。

### 合约部署

#### 创建钱包

钱包用于存储私钥，创建钱包生成的密码用于解锁钱包，应妥善保存。--name 指定创建钱包名称，默认会创建 default.wallet 钱包

```
cleos create wallet --name mywallet
```

#### 导入账号 EOSIO

EOSIO 账号是默认生成的账号，可以用来部署 eosio.bios 系统合约；使用账号需要拥有私钥，因此我们需要先导入账号私钥。

```
cleos wallet import --private-key 5KQwrPbwdL6PhXujxW37FSSQZ1JiwsST4cqQzDeyXtP79zkvFD3
```

#### 部署 eosio.bios

现在将 eosio.bios 部署到 eosio 账号，以便于用于创建新账号等操作。

```
cleos set contract eosio build/contracts/eosio.bios -p eosio@active
```

#### 生成密钥对

密钥对用于控制账号的权限，EOS 密钥包含 owner 和 active，owner 拥有最高权限，包括修改更换 active key，active 拥有除修改 owner key 之前的所有权限。

```
cleos create key --to-console   // owner
cleos create key --to-console   // active
```

将生成的密钥对导入钱包，以便于后续使用，可指定钱包名称

```
cleos wallet import --private-key owner_priv_key
cloes wallet import --private-key active_priv_key
```

#### 创建账号

```
cleos create account eosio account_name owner_pub_key active_pub_key
```

#### 创建并部署合约

创建简单[《Hello World》](https://developers.eos.io/eosio-cpp/docs/hello-world) 合约并部署。

如果你想在测试网，比如 [JungleTestnet](http://jungle.cryptolions.io/#home) 或者主网上测试，可以在以上命令中添加 **--url** 参数指定需要操作的网络环境。


[*Ubuntu download*](http://releases.ubuntu.com/)  
[*EOS developers*](https://developers.eos.io/eosio-nodeos/docs/getting-the-code)
