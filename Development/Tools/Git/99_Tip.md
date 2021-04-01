
# 代理设置

## socks代理

```sh
git config --global http.proxy socks5://127.0.0.1:1080
git config --global https.proxy socks5://127.0.0.1:1080
```

## http代理

```sh
git config --global http.proxy http://127.0.0.1:1080
git config --global https.proxy https://127.0.0.1:1080
```

## 取消代理

```sh
git config --global --unset http.proxy
git config --global --unset https.proxy
```