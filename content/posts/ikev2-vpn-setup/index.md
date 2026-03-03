---
title: 在虚拟机上通过 IKEv2 部署 IPSec VPN
date: "2025-11-14 12:42:48"
draft: true
---

**IPSec**（Internet Protocol Security）是一个在不安全的互联网中建立安全的网络隧道的协议族，可以为数据包提供认证和加密等功能。而 **IKEv2** 是用于自动化管理 **IPSec** 安全关联和密钥的协议，包括身份认证，密钥交换，协商加密算法。

## 更新与下载所需软件
```shell
apt update && apt upgrade -y
apt install -y curl wget vim net-tools ufw strongswan strongswan-pki iptables-persistent openssl dnsutils charon-systemd 
```

## 使用新体系
```shell
systemctl disable strongswan-starter
systemctl stop strongswan-starter

systemctl enable strongswan
systemctl start strongswan

# 验证
systemctl status strongswan             # active (running)
systemctl status strongswan-starter     # inactive (dead)
```

## 创建证书

CA 证书是建立 VPN 隧道过程中的身份认证和信任链建立的重要基础
### 生成证书
```shell
mkdir -p /root/vpn-pki && cd $_

# 生成 CA 私钥
ipsec pki --gen --type rsa --size 4096 --outform pem > private/ca.key

# 生成自签 CA 证书
ipsec pki --self --ca --lifetime 3650 \
  --in private/ca.key \
  --dn "C=CN, O=arcSYSu, CN=arcSYSuRootCA" \
  --outform pem > certs/ca.cert.pem

# 生成服务器私钥
ipsec pki --gen --type rsa --size 4096 --outform pem > private/server.key

# 生成服务器证书
ipsec pki --pub --in private/server.key --type rsa \
| ipsec pki --issue \
    --cacert certs/ca.cert.pem \
    --cakey private/ca.key \
    --dn "C=CN, O=arcSYSu, CN=yatcc-ai.com" \
    --san "yatcc-ai.com" \
    --flag serverAuth \
    --flag ikeIntermediate \
    --lifetime 1825 \
    --outform pem > certs/server.cert.pem                         # 后续要改成 Let's Encrypt 证书
  
mv *key /etc/swanctl/private
mv ca.cert.pem /etc/swanctl/x509ca
mv server.cert.pem /etc/swanctl/x509
```

由于私钥非常敏感，所以需要将文件权限设置为700：
```shell
chmod 700 /etc/swanctl/private
chmod 700 /etc/swanctl/x509ca
chmod 700 /etc/swanctl/x509
```

**PKCS #12**是一种归档文件格式，用来实现存储许多加密对象在一个单独的文件中。通常用其来打包一个私钥及相关的证书，或者打包信任链的全部项目
## 为用户生成客户端证书并导出 PKCS#12
```shell
ipsec pki --gen --type rsa --size 4096 --outform pem > lvzw.key     # 生成客户端私钥
ipsec pki --pub --in lvzw.key --type rsa \
| ipsec pki --issue --cacert ca.pem --cakey ca.key \
--dn "C=CN, O=arcSYSu, CN=lvzw" \
--lifetime 1095 --outform pem > lvzw.pem            # 用 CA 签发客户端证书
openssl pkcs12 -export -inkey lvzw.key -in lvzw.pem -certfile ca.pem -name "lvzw" -passout pass:060127 -out lvzw.p12      # 导出 PKCS#12
```

## 客户端安装 CA 证书 （）
```shell
cd /etc/swanctl/x509ca
openssl x509 -in ca.cert.pem -out ca.cer -outform der
```
然后把 `ca.cer` 文件分发给客户端，客户端在自己主机上安装 CA 证书（到受信任的根证书颁发机构）

## 配置 strongswan
```shell
# /etc/swanctl/swanctl.conf
connections{
    ikev2-eap-mschapv2{
        version = 2
        encap = yes
        proposals = 3des-sha1-modp1024
        rekey_time = 0s
        pools = pool-ipv4
        fragmentation = yes
        dpd_delay = 30s
        send_cert = always
        unique = never
        local{
            auth = pubkey
            id = yatcc-ai.com         # 服务端 VPN 身份识别（与证书 CN/SAN 匹配）
            certs = server.cert.pem
        }
        remote{
            auth = eap-mschapv2
            eap_id = %any
        }
        children{                               # IPsec 加密数据通道流量配置
            ikev2-eap-mschapv2{
                local_ts = 0.0.0.0/0            # 向客户端提供访问整个 IPv4 互联网的通道
                rekey_time = 0s
                dpd_action = clear
            }
        }
        
    }
}

pools{
    pool-ipv4{
        addrs = 10.10.10.0/24
        dns = 114.114.114.114,8.8.8.8
    }
}
secrets{
    eap-user{
        id = "lvzw"
        secret = "123456"
    }
}
```

## 开启 IP 转发与 NAT
```shell
echo "net.ipv4.ip_forward=1" > /etc/sysctl.d/99-ipforward.conf
sysctl --system

iptables -t nat -A POSTROUTING -s 10.10.10.0/24 -o ens3 -j MASQUERADE
netfilter-persistent save
```

## 启动 strongswan
```shell
systemctl enable strongswan-starter
systemctl restart strongswan-starter

systemctl status strongswan-starter
ipsec statusall
```

在客户端上尝试连接
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;
