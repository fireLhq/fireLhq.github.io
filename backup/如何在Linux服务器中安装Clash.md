# 文章以Ubuntu 22.04 Server为例

## 前言

该方法建立在使用过**Clash for Windows**的基础上

## 步骤

1. 取出Windows设备配置
2. 安装和配置Clash for Linux
3. 注册为系统服务
4. 安装浏览器UI界面并在线管理Clash

## 步骤1：取出配置

准备好两个文件：

1. Country.mmdb
2. profiles/xxxxxxxx.yml

<img width="1068" height="760" alt="Image" src="https://github.com/user-attachments/assets/b5583baf-8969-422b-a391-64aaa5c8c739" />

<img width="770" height="288" alt="Image" src="https://github.com/user-attachments/assets/42e7c0af-1eea-41a1-8775-8f8806d0c6e9" />

**注** yml文件可能有多个，原因是你之前使用过多个配置；如果不确定哪个有效，可以先选中`Clash`左侧的`Profiles`（配置）切换到配置页面，然后右键点击“有效”的配置再点击`Show in folder`（在资源管理器中显示），即可在资源管理器中定位到有效yml文件。

## 步骤2：安装和配置Clash

### 安装Clash

- 下载Clash应用程序：`clash-linux-amd64-v1.18.0.gz`

```shell
wget https://github.com/WindSpiritSR/clash/releases/download/v1.18.0/clash-linux-amd64-v1.18.0.gz
```

- 解压文件

```shell
gunzip clash-linux-amd64-v1.18.0.gz
```

- 重命名为clash

```shell
mv clash-linux-amd64-v1.18.0 clash
```

- 移动到`/usr/local/bin/`目录下（方便在任何位置调用Clash）

```shell
sudo mv clash /usr/local/bin/
```

- 设置为可执行文件

```shell
sudo chmod +x /usr/local/bin/clash
```

### 配置Clash

- 创建配置文件目录

```shell
sudo mkdir -p /etc/clash
```

- 上传`Country.mmdb`文件和`xxxxxxxx.yml`文件到Ubuntu服务器，并将`xxxxxxxx.yml`重命名为`config.yaml`

```shell
mv xxxxxxxx.yml config.yaml
```

- 移动两个文件到配置文件目录

```shell
sudo mv Country.mmdb config.yaml /etc/clash/
```

- 修改配置文件

```shell
sudo vim /etc/clash/config.yaml
```

- 找到配置文件中的`external-controller`字段（管理面板默认是9090，不是7890）

```yaml
# Clash 的 RESTful API
external-controller: '127.0.0.1:9090'
```

- 把值改为`'0.0.0.0:9090'`（这样在同一局域网下就可以被其他主机访问）

```yaml
# Clash 的 RESTful API
external-controller: '0.0.0.0:9090'
secret: 'xxxxxxxxxx' # 设置 web ui 密钥（可选）
```

## 步骤3：注册为系统服务

- 创建systemd服务文件

```shell
sudo vim /etc/systemd/system/clash.service
```

- 添加内容

```ini
[Unit]
Description=Clash Proxy Service
After=network.target

[Service]
Type=simple
ExecStart=/usr/local/bin/clash -d /etc/clash
Restart=on-failure
RestartSec=5s
User=root
LimitNOFILE=1048576

[Install]
WantedBy=multi-user.target
```

- 重点说明

| 配置项             | 含义               |
| ------------------ | ------------------ |
| ExecStart          | clash 启动命令     |
| -d /etc/clash      | clash 配置目录     |
| Restart=on-failure | 崩了自动重启       |
| LimitNOFILE        | 防止连接数过多报错 |

- 重新加载systemd管理器的配置文件

```shell
sudo systemctl daemon-reload
```

## 步骤4：安装UI和在线管理Clash

### 安装UI

- 创建UI目录并进入

```shell
sudo mkdir -p /etc/clash/ui
cd /etc/clash/ui
```

- 下载Yacd并解压（解压后会得到一个`public`文件夹，里面有一堆`index.html / js / css`文件）

```shell
sudo wget https://github.com/haishanh/yacd/releases/latest/download/yacd.tar.xz
sudo tar -xf yacd.tar.xz
```

- 在Clash配置里启用Web UI

```shell
sudo vim /etc/clash/config.yaml
```

- 在`external-controller`字段后添加一个`external-ui`字段（`external-ui`指向的是解压后的目录）

```yaml
# Clash 的 RESTful API
external-controller: '0.0.0.0:9090'
external-ui: /etc/clash/ui/public
secret: 'xxxxxxxxxx' # 设置 web ui 密钥（可选）
```

- 启动Clash服务

```shell
sudo systemctl start clash
```

- 其他相关操作

```shell
sudo systemctl enable clash # 设为开机自启
sudo systemctl status clash # 查看服务状态
sudo systemctl stop clash # 停止服务
sudo systemctl restart clash # 重启服务
```

### 在线管理Clash

- 在可以连接Ubuntu服务器的其他主机的浏览器上输入`http://<服务器IP>:9090/ui`访问管理面板
- 登录页面按以下步骤填写

<img width="640" height="800" alt="Image" src="https://github.com/user-attachments/assets/7876cf76-e020-4ac6-91ee-983808f1e487" />

- Overview（概览）页面查看实时流量使用情况

<img width="1920" height="934" alt="Image" src="https://github.com/user-attachments/assets/d8bdd116-59be-46cb-a1b9-978adb60b7c9" />

- Config（配置）页面可以设置语言

<img width="1920" height="934" alt="Image" src="https://github.com/user-attachments/assets/adf93b1e-e484-4c67-86c7-a2c31ea2e195" />

- 其它页面请自行研究