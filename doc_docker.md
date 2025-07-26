# 使用Docker镜像

LibreSpeed的Docker版本可在此处获取：[GitHub Packages](https://github.com/librespeed/speedtest/pkgs/container/speedtest)

# Alpine Linux变体

基于Alpine Linux的LibreSpeed Docker版本也可在此处获取：[GitHub Packages](https://github.com/librespeed/speedtest/pkgs/container/speedtest)，所有带有`-alpine`后缀的标签。此变体体积更小，但由于其工具链基于[musl](https://en.wikipedia.org/wiki/Musl) libc，可能会有略微不同的行为，如[此处](https://alpinelinux.org/about/)所述。

## 快速开始

如果您只是想尝试一下，最快的方法是：

```shell
docker run -p 80:8080 -d --name speedtest --rm ghcr.io/librespeed/speedtest
```

然后用浏览器访问服务器的80端口进行测试。如果80端口已被占用，请调整上述命令中的80:8080的第一个数字。默认以独立模式运行。

## Docker Compose

在生产环境中，我们建议使用docker-compose。

要使用[docker compose](https://docs.docker.com/compose/)启动容器，可以使用以下`docker-compose.yml`配置：

```yml
version: '3.7'
services:
  speedtest:
    container_name: speedtest
    image: ghcr.io/librespeed/speedtest:latest
    restart: always
    environment:
      MODE: standalone
      #TITLE: "LibreSpeed"
      #TELEMETRY: "false"
      #ENABLE_ID_OBFUSCATION: "false"
      #REDACT_IP_ADDRESSES: "false"
      #PASSWORD:
      #EMAIL:
      #DISABLE_IPINFO: "false"
      #IPINFO_APIKEY: "your api key"
      #DISTANCE: "km"
      #WEBPORT: 8080
    ports:
      - "80:8080" # webport映射 (host:container)
```

请根据预期的操作模式调整环境变量。

## 独立模式

如果您想在单个服务器上安装LibreSpeed，需要将其配置为独立模式。为此，将`MODE`环境变量设置为`standalone`。

测试可在80端口访问。

以下是此模式下可用的其他环境变量列表：

* __`TITLE`__: 速度测试的标题。默认值：`LibreSpeed`
* __`TELEMETRY`__: 是否启用遥测。如果启用，您可能希望数据被持久化。见下文。默认值：`false`
* __`ENABLE_ID_OBFUSCATION`__: 当启用遥测时设置为true，测试ID将被混淆，以避免暴露数据库内部的顺序ID。默认值：`false`
* __`REDACT_IP_ADDRESSES`__: 当启用遥测时设置为true，IP地址和主机名将从收集的遥测数据中删除，以提高隐私性。默认值：`false`
* __`DB_TYPE`__: 当设置为支持的数据库后端之一时，将使用它而不是默认的sqlite数据库后端。必须将TELEMETRY设置为`true`。还必须按照[doc.md](doc.md#creating-the-database)中所述创建数据库。支持的后端类型有：
  * sqlite - 无需额外设置
  * mysql, postgresql - 设置额外的环境变量：
    * DB_HOSTNAME - 数据库服务器的名称或IP
    * DB_PORT (仅mysql) - 数据库运行的端口
    * DB_NAME - 遥测数据库的名称
    * DB_USERNAME, DB_PASSWORD - 具有数据库读写权限的用户凭据
  * mssql - 尚未在docker镜像中支持（欢迎提交PR实现，需在`entrypoint.sh`中完成）
* __`PASSWORD`__: 访问统计页面的密码。如果未设置，统计页面将不允许访问。
* __`EMAIL`__: GDPR请求的电子邮件地址。启用遥测时必须指定。
* __`DISABLE_IPINFO`__: 如果设置为`true`，将不会从[ipinfo.io](https://ipinfo.io)或离线数据库获取ISP信息和距离。默认值：`false`
* __`IPINFO_APIKEY`__: [ipinfo.io](https://ipinfo.io)的API密钥。可选，但如果您想使用完整的[ipinfo.io](https://ipinfo.io) API（距离测量所需），则必需
* __`DISTANCE`__: 当`DISABLE_IPINFO`设置为false时，指定如何测量与服务器的距离。可以是`km`（公里）、`mi`（英里）或空字符串（禁用距离测量）。需要[ipinfo.io](https://ipinfo.io) API密钥。默认值：`km`
* __`WEBPORT`__: 允许为包含的Web服务器选择自定义端口。默认值：`8080`。请注意，您必须通过docker的-p参数暴露它。这不是服务在docker外部暴露的端口！

如果启用了遥测，统计页面将在`http://your.server/results/stats.php`可用，但必须指定密码。

### 持久化sqlite数据库

默认数据库驱动是sqlite。数据库文件写入`/database/db.sql`。

因此，如果您希望数据在镜像更新后保持不变，必须使用`-v $PWD/db-dir:/database`挂载卷。

#### 带遥测的独立模式示例

此命令以独立模式启动LibreSpeed，带有持久化遥测、ID混淆和统计密码，在86端口上：

```shell
docker run -e MODE=standalone -e TELEMETRY=true -e ENABLE_ID_OBFUSCATION=true -e PASSWORD="yourPasswordHere" -e WEBPORT=86 -p 86:86 -v $PWD/db-dir/:/database -it ghcr.io/librespeed/speedtest
```

## 多测试点

对于多个服务器，您需要设置1个或多个LibreSpeed后端和1个LibreSpeed前端。

### 后端模式

在后端模式下，LibreSpeed仅提供测试点，没有UI。为此，将`MODE`环境变量设置为`backend`。

以下后端文件可在80端口访问：`garbage.php`、`empty.php`、`getIP.php`

以下是此模式下可用的其他环境变量列表：

* __`IPINFO_APIKEY`__: [ipinfo.io](https://ipinfo.io)的API密钥。可选，但如果您想使用完整的[ipinfo.io](https://ipinfo.io) API（距离测量所需），则必需。如果未提供API密钥，将使用离线数据库。

#### 后端模式示例

此命令以后端模式启动LibreSpeed，使用默认设置，在80端口上：

```shell
docker run -e MODE=backend -p 80:8080 -it ghcr.io/librespeed/speedtest
```

### 前端模式

在前端模式下，LibreSpeed为客户端提供Web UI和服务器列表。为此：

* 将`MODE`环境变量设置为`frontend`
* 创建一个包含测试点的servers.json文件。语法如下：

    ```jsonc
    [
        {
            "name": "服务器1的友好名称",
            "server" :"//server1.mydomain.com/",
            "dlURL" :"garbage.php",
            "ulURL" :"empty.php",
            "pingURL" :"empty.php",
            "getIpURL" :"getIP.php"
        },
        {
            "name": "服务器2的友好名称",
            "server" :"https://server2.mydomain.com/",
            "dlURL" :"garbage.php",
            "ulURL" :"empty.php",
            "pingURL" :"empty.php",
            "getIpURL" :"getIP.php"
        },
        //...更多服务器...
    ]
    ```

    注意：如果服务器仅支持HTTP或HTTPS，请在server字段中指定协议。如果同时支持两者，只需使用`//`。
* 将此文件挂载到容器中的`/servers.json`（示例见文件末尾）

测试可在80端口访问。

此模式下可用的环境变量列表与[上面的独立模式](#standalone-mode)相同。

#### 前端模式示例

此命令以前端模式启动LibreSpeed，使用给定的`servers.json`文件，启用遥测、ID混淆、统计密码和持久化sqlite数据库存储结果：

```shell
docker run -e MODE=frontend -e TELEMETRY=true -e ENABLE_ID_OBFUSCATION=true -e PASSWORD="yourPasswordHere" -v $PWD/servers.json:/servers.json -v $PWD/db-dir/:/database -p 80:80 -it ghcr.io/librespeed/speedtest
```

### 双模式

在双模式下，LibreSpeed作为可连接到其他测试点的独立服务器运行。
为此：

* 将`MODE`环境变量设置为`dual`
* 按照前端模式的`servers.json`说明操作
* 第一个服务器条目应该是本地服务器，使用客户端可以访问的服务器端点地址。
