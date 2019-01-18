---
layout: post
title: "Cardano SL 节点搭建"
description: ""
category: 
tags: []
---
{% include JB/setup %}


### 节点搭建

#### 安装

首先需要安装 Nix 用于编译 Cardano (需重新登录终端才能生效)

```
curl https://nixos.org/nix/install | sh
```

接着配置 IOHK 二进制缓存，如果该文件不存在则创建 **/etc/nix/nix.conf**

```
substituters         = https://hydra.iohk.io https://cache.nixos.org
trusted-substituters =
trusted-public-keys  = hydra.iohk.io:f/Ea+s+dFdN+3Y/G+FDgSq+a5NEWhJGzdjvKNGv0/EQ= cache.nixos.org-1:6NCHdD59X431o0gWypbMrAURkbJ16ZPMQFGspcDShjY=
```

克隆 Cardano SL 源代码

```
git clone https://github.com/input-output-hk/cardano-sl.git
cd cardano-sl
git checkout master
git pull
```

编译并检查输出文件

```
nix-build -A cardano-sl-node-static --out-link master
// ... 等待 build 结束
ls master/bin   // 检查输出目录是否包含 cardano-node-simple
```

#### 连接主网

```
nix-build -A connectScripts.mainnet.wallet -o connect-to-mainnet
// ... 等待 build 结束
./connect-to-mainnet  // 生成目录 state-wallet-mainnet，并同步区块数据
```

连接测试网与主网过程相似

```
nix-build -A connectScripts.testnet.wallet -o connect-testnet-wallet.sh
./connect-testnet-wallet.sh
```

#### 钱包访问

Cardano SL 默认使用配置文件 sample-wallet-config.nix，如果需要调整，则在同一目录下重新创建一份 custom-wallet-config.nix，然后调整相关配置参数即可。Cardano SL 默认采用客户端和服务端证书的双向校验规则，如主网证书存放在 state-wallet-mainnet/tls 目录下，默认监听本地 localhost。

```
## 自定义钱包 ip 和 port
walletListen = "ip:8090";
## 禁用客户端身份验证
disableClientAuth = true;
```

接着我们来测试下钱包是否能正常访问

```
curl -k https://localhost:8090/api/v1/node-info | jq
```

curl 默认启用服务器证书校验，因此指定参数 -k 禁用

如果你启用了客户端证书校验，则需要在请求时手动指定客户端证书

```
curl -k --cert client.pem https://localhost:8090/api/v1/wallets | jq
```

如果你想使用代码访问接口，这里有一个 C# 的库 [AdaSharp](https://github.com/KanLei/AdaSharp)，证书可以通过下面命令生成

```
// 通过 pem 文件生成
openssl pkcs12 -export -in client.pem -out client.pfx
// openssl pkcs12 -export -in client.pem -inkey client.pem -out client.pfx
openssl pkcs12 -export -in client.pem -out client.p12
// 通过 cer 和 key 文件生成
openssl pkcs12 -export -in public.cer -inkey private.key -out cert_key.p12
```

cer 是公钥的base64存储，key 是私钥的base64存储，pem 是公钥和私钥的base64存储；pkcs12 是公钥和私钥的二进制存储，有 pfx 和 p12 两种格式，pfx 格式适用于 microsoft 平台，p12 适用于其它平台。


[*Cardano Docs*](https://github.com/input-output-hk/cardano-sl/tree/develop/docs/)  
[*Cardano Explorer*](https://cardanoexplorer.com/)  
[*Wallet API*](https://cardanodocs.com/technical/wallet/api/v1/)  
[*Yoroi Wallet*](https://yoroi-wallet.com/)