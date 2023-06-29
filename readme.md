# xray-warp-docker

## 项目介绍

通过xray的docker镜像, 将warp共享给其他设备使用

## 使用方法

1. 克隆仓库

   ```bash
   git clone https://www.github.com/lautumn1990/xray-warp-docker.git
   ```

1. 配置网络入口
   1. 需要公网访问
      1. 需要先配置好证书, 在`docker-compose.yml`中修改`xray`的`volumes`配置
      1. 配置好`config.json`中`inbounds`关于`trojan`中的配置, 配置`password`和`email`
   1. 不需要公网访问
      1. 注释掉`docker-compose.yml`中关于证书的配置
      1. 注释掉`config.json`中`inbounds`关于`trojan`中的配置

1. 配置warp相关配置, 在`config.json`的`outbounds`中配置`wireguard`
   1. 配置`secretKey`, 如果使用`wgcf`, 查看`wgcf-account.toml`文件中的`private_key`. 如果使用`warp-cli`, 查看`/var/lib/cloudflare-warp/reg.json`文件
   1. 配置`endpoint`, 可以使用默认, 可以使用下面的方法进行测速优选`curl -sSL https://gitlab.com/rwkgyg/CFwarp/raw/main/point/endip.sh -o endip.sh && chmod +x endip.sh && ./endip.sh`
   1. 配置`reserved`, 如果使用`wgcf`可以用以下脚本. 如果使用`warp-cli`, 参考[如何获得warp的"reserved"值](https://community.sagernet.org/t/warp-reserved/42/2)

      ```bash
      _account_token=`awk -F "['']" '/access_token/{print $2}' wgcf-account.toml`
      _account_device_id=`awk -F "['']" '/device_id/{print $2}' wgcf-account.toml`
      curl --request GET "https://api.cloudflareclient.com/v0a2158/reg/${_account_device_id}" \
           --silent \
           --location \
           --header 'User-Agent: okhttp/3.12.1' \
           --header 'CF-Client-Version: a-6.10-2158' \
           --header 'Content-Type: application/json' \
           --header "Authorization: Bearer ${_account_token}" | jq -r '.config.client_id' | base64 -d | xxd -p | fold -w2 | while read HEX; do printf '%d ' "0x${HEX}"; done | awk '{print "["$1", "$2", "$3"]"}'
      ```
   1. 其他配置根据需要, 可以改或者不改

1. 启动`docker-compose`

   ```bash
   docker-compose up -d
   ```

## 测试

### docker主机测试

```bash
curl -x socks5h://127.0.0.1:1081 http://www.cloudflare.com/cdn-cgi/trace
# 查看warp是不是on或者plus
```

### 客户端测试

```bash
# 需要使用docker主机的ip
curl -x socks5h://docker-hostip:1081 http://www.cloudflare.com/cdn-cgi/trace
```

如果有公网ip, 可以通过`Trojan`协议访问, 配置方法[参考](https://github.com/XTLS/Xray-examples/blob/main/Trojan-TCP-TLS%20(minimal)/config_client.json)

如果想使用其他协议, 可自行修改

---

## 参考

- [CFwarp](https://gitlab.com/rwkgyg/CFwarp)
- [Xray-examples](https://github.com/XTLS/Xray-examples)
- [wireguard](https://github.com/chika0801/Xray-examples/blob/main/wireguard.md)
