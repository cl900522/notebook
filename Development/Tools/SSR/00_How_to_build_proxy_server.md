安装SSR代理服务器
=====================
>[ShdowSocks非官方网站](https://shadowsocks.be/)

## 单个用户的代理服务
1. 运行自动安装脚本
    ```sh
    wget --no-check-certificate -O SSR.sh https://raw.githubusercontent.com/bakuniverse/back/master/SSR.sh && chmod +x SSR.sh && bash SSR.sh
    ```

2. 安装成功后会提示
    ![](/images/2017/09/QQ截图20170904115604.png)

3. 如果需要修改部分配置，则可以编辑配置文件并重启服务
    ```sh
    vim /etc/shadowsocks.json
    service shadowsocks restart
    ```

## 多用户的代理服务
1. 运行自动安装脚本
    ```sh
    # 内有BBR+锐速等插件一键安装
    wget -N –no-check-certificate https://softs.fun/Bash/ssrmu.sh && chmod +x ssrmu.sh && bash ssrmu.sh
    wget -N –no-check-certificate https://raw.githubusercontent.com/ToyoDAdoubi/doubi/master/ssrmu.sh && chmod +x ssrmu.sh && bash ssrmu.sh
    ```

## 安装速锐
1. 运行自动安装脚本
    ```sh
    wget --no-check-certificate -O appex.sh  https://raw.githubusercontent.com/0oVicero0/serverSpeeder_Install/master/appex.sh && chmod +x appex.sh && bash appex.sh install
    ```
