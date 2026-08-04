# 文章以Ubuntu 22.04 Server为例

## 前言

Mihomo是Clash Verge的内核，因此该方法建立在使用过**Clash Verge Rev**的基础上

## 步骤

1. 取出Windows设备配置
2. 安装和配置Mihomo for Linux
3. 注册为系统服务
4. 安装浏览器UI界面并在线管理Mihomo

## 步骤1：取出配置

准备好两个文件，位于：C:\Users\用户名\AppData\Roaming\io.github.clash-verge-rev.clash-verge-rev下

1. Country.mmdb
2. profiles/xxxxxxxx.yaml

<img width="804" height="468" alt="Image" src="https://github.com/user-attachments/assets/765471fd-4d04-488c-bdd6-9fa8d2c40f35" />

<img width="771" height="374" alt="Image" src="https://github.com/user-attachments/assets/7a99e3be-9419-492f-8d96-f2ce03b30799" />

**注** 将用户名替换为自己电脑的实际用户名。yaml文件可能有多个，原因是你之前使用过多个配置；如果不确定哪个有效，可以用文本查看工具（如记事本）打开并对比下面的节点和`Clash Verge`中的节点区别，从而精准找到。

## 步骤2：安装和配置Mihomo

### 安装Mihomo

- 下载Mihomo应用程序：`mihomo-linux-amd64-v1.19.29.gz`

```shell
wget https://github.com/MetaCubeX/mihomo/releases/download/v1.19.29/mihomo-linux-amd64-v1.19.29.gz
```

- 解压文件

```shell
gunzip mihomo-linux-amd64-v1.19.29.gz
```

- 重命名为clash

```shell
mv mihomo-linux-amd64-v1.19.29.gz mihomo
```

- 移动到`/usr/local/bin/`目录下（方便在任何位置调用Mihomo）

```shell
sudo mv mihomo /usr/local/bin/
```

- 设置为可执行文件

```shell
sudo chmod +x /usr/local/bin/mihomo
```

### 配置Mihomo

- 创建配置文件目录

```shell
sudo mkdir -p /etc/mihomo
```

- 上传`Country.mmdb`文件和`xxxxxxxx.yaml`文件到Ubuntu服务器，并将`xxxxxxxx.yaml`重命名为`config.yaml`

```shell
mv xxxxxxxx.yaml config.yaml
```

- 移动两个文件到配置文件目录

```shell
sudo mv Country.mmdb config.yaml /etc/mihomo/
```

- 修改配置文件

```shell
sudo vim /etc/mihomo/config.yaml
```

- 找到配置文件中的`external-controller`字段（管理面板默认是9093，不是7890）

```yaml
external-controller: '127.0.0.1:9093'
```

- 把值改为`'0.0.0.0:9093'`（这样在同一局域网下就可以被其他主机访问）

```yaml
external-controller: '0.0.0.0:9093'
secret: 'xxxxxxxxxx' # 设置 web ui 密钥（可选，默认没有此行）
```

## 步骤3：注册为系统服务

- 创建systemd服务文件

```shell
sudo vim /etc/systemd/system/mihomo.service
```

- 添加内容

```ini
[Unit]
Description=Mihomo Proxy Core
After=network.target

[Service]
Type=simple
User=root
ExecStart=/usr/local/bin/mihomo -d /etc/mihomo
Restart=always

[Install]
WantedBy=multi-user.target
```

- 重点说明

| 配置项         | 含义            |
| -------------- | --------------- |
| ExecStart      | mihomo 启动命令 |
| -d /etc/mihomo | mihomo 配置目录 |

- 重新加载systemd管理器的配置文件

```shell
sudo systemctl daemon-reload
```

## 步骤4：安装UI和在线管理Mihomo

### 安装UI

- 创建UI目录并进入

```shell
sudo mkdir -p /etc/mihomo/ui
cd /etc/mihomo/ui
```

- 下载Metacubexd并解压（解压后会得到一堆`index.html / js / css`文件以及一些图片和文件夹）

```shell
sudo wget https://github.com/MetaCubeX/metacubexd/releases/latest/download/compressed-dist.tgz
sudo tar -zxf compressed-dist.tgz
```

- 在Mihomo配置里启用Web UI

```shell
sudo vim /etc/mihomo/config.yaml
```

- 在`external-controller`字段后添加一个`external-ui`字段（`external-ui`指向的是解压后的目录）

```yaml
external-controller: '0.0.0.0:9093'
external-ui: /etc/mihomo/ui
secret: 'xxxxxxxxxx' # 设置 web ui 密钥（可选，默认没有此行）
```

- 启动Mihomo服务

```shell
sudo systemctl start mihomo
```

- 其他相关操作

```shell
sudo systemctl enable mihomo # 设为开机自启
sudo systemctl status mihomo # 查看服务状态
sudo systemctl stop mihomo # 停止服务
sudo systemctl restart mihomo # 重启服务
```

### 在线管理Mihomo

- 在可以连接Ubuntu服务器的其他主机的浏览器上输入`http://<服务器IP>:9093/ui`访问管理面板
- API 地址 / Host：`http://<服务器IP>:9093`
- Secret / Password：填入你在 `/etc/mihomo/config.yaml` 中设置的 `secret`（如果没设留空即可）
- 概览页面查看实时流量使用情况和网络延迟

<img width="1910" height="896" alt="Image" src="https://github.com/user-attachments/assets/2a128693-6b7d-402c-9f62-6bc7d3a190cd" />

<img width="1910" height="896" alt="Image" src="https://github.com/user-attachments/assets/cdc182a2-664f-44b8-a8e9-130db68c1c52" />

- 配置页面可以查看和切换节点

<img width="1910" height="896" alt="Image" src="https://github.com/user-attachments/assets/84eab635-2832-4c07-b769-698d68840c12" />

- 其它页面可自行探索

