### 高级参数设置（谨慎修改，除非您知道自己在做什么）

* __telemetry_extra__: 您希望传递给遥测系统的额外数据。这是一个字符串字段，如果您想传递对象，请确保使用 `JSON.stringify`。此字符串将添加到该测试的数据库条目中。
* __enable_quirks__: 启用浏览器特定的优化。这些优化会覆盖一些默认设置，但不会覆盖明确设置的参数。
  * 默认值: `true`
* __garbagePhp_chunkSize__: garbage.php发送的块大小（以MB为单位）
  * 默认值: `100`
  * 推荐值: `>=10`
  * 最大值: `1024`
* __xhr_dlMultistream__: 下载测试应打开的流数量
  * 默认值: `6`
  * 推荐值: `>=3`
  * 默认覆盖: 如果enable_quirks为true，Edge上为3
  * 默认覆盖: 如果enable_quirks为true，基于Chromium的浏览器上为5
* __xhr_ulMultistream__: 上传测试应打开的流数量
  * 默认值: `3`
  * 推荐值: `>=1`
* __xhr_ul_blob_megabytes__: 上传测试期间发送的Blob大小（以MB为单位）
  * 默认值: `20`
  * 默认覆盖: 基于Chromium的移动浏览器上为4（约版本65引入的限制），这将被强制执行
  * 默认覆盖: IE11和Edge当前使用不同的上传测试方法，此参数将被忽略
* __xhr_multistreamDelay__: 多个流应延迟多长时间（以毫秒为单位）
  * 默认值: `300`
  * 推荐值: `>=100`，`<=700`
* __xhr_ignoreErrors__: 如何应对下载/上传流和ping测试中的错误
  * `0`: 出错时测试失败（此测试以前版本的行为）
  * `1`: 流/ping失败时重新启动
  * `2`: 忽略所有错误
  * 默认值: `1`
  * 推荐值: `1`
* __time_dlGraceTime__: 实际测量下载速度前等待的时间（以秒为单位）。这是个好主意，因为我们希望等待TCP窗口达到最大值（或接近最大值）
  * 默认值: `1.5`
  * 推荐值: `>=0`
* __time_ulGraceTime__: 实际测量上传速度前等待的时间（以秒为单位）。这是个好主意，因为我们希望等待缓冲区填满（避免测试开始时的峰值）
  * 默认值: `3`
  * 推荐值: `>=1`
* __ping_allowPerformanceApi__: 切换是否使用Performance API来提高支持它的浏览器上Ping/Jitter测试的准确性。
  * 默认值: `true`
  * 默认覆盖: Firefox上为`false`，因为其Performance API实现不准确
* __useMebibits__: 使用mebibits/s而不是megabits/s表示速度
  * 默认值: `false`
* __overheadCompensationFactor__: HTTP和网络开销的补偿。默认值假设互联网上使用的典型MTU。如果您在具有不同MTU的内部网络中使用，或者如果您使用IPv6而不是IPv4，您可能需要更改此值。
  * 默认值: `1.06` 可能是所有开销的合理估计。这是通过比较测量速度和网络适配器报告的速度凭经验测量的。
  * `1048576/925000`: 旧默认值，可能过高
  * `1.0513`: 互联网上的HTTP+TCP+IPv6+ETH（经验测试，非计算）
  * `1.0369`: 互联网上的HTTP+TCP+IPv4+ETH的替代值（经验测试，非计算）
  * `1.081`: 互联网上的另一个替代值（经验测试，非计算）
  * `1514 / 1460`: TCP+IPv4+ETH，忽略HTTP开销
  * `1514 / 1440`: TCP+IPv6+ETH，忽略HTTP开销
  * `1`: 忽略开销。这测量的是您实际下载和上传文件的速度，而不是原始连接速度

### 多测试点

如果您想使用多个测试服务器，现在是添加所有测试点并选择最佳测试点的时候了。如果您不想使用此功能，请跳过此部分。

最好的方法是声明一个包含所有服务器的数组，并将其提供给速度测试：

```js
var SPEEDTEST_SERVERS=[
 server1,
 server2,
 ...
];
s.addTestPoints(SPEEDTEST_SERVERS);
```

列表中的每个服务器都是一个包含以下内容的对象：

* `name`: 此测试点的用户友好名称
* `server`: 服务器的URL。如果您的服务器仅支持HTTP或HTTPS，请分别在开头放置`http://`或`https://`；如果同时支持两者，请在开头放置`//`，它将被自动替换
* `dlURL`: 此服务器上下载测试的路径（garbage.php或替代品）
* `ulURL`: 此服务器上上传测试的路径（empty.php或替代品）
* `pingURL`: 此服务器上ping测试的路径（empty.php或替代品）
* `getIpURL`: 此服务器上getIP的路径（getIP.php或替代品）

这些参数都不能省略。

示例：

```js
{
 name:"米兰, IT",
 server:"http://backend1.myspeedtest.net/",
 dlURL:"garbage.php",
 ulURL:"empty.php",
 pingURL:"empty.php",
 getIpURL:"getIP.php"
}
```

现在，我们可以运行服务器选择器：

```js
s.selectServer(function(server){
    //做些什么
})
```

`selectServer`函数是异步的，以避免冻结UI，当它完成选择ping最低的服务器时，将运行回调函数。`server`参数是选定的服务器，您可以在UI中显示它（如果需要）。__在选择完成之前，您不能开始测试！__

您也可以手动设置测试点（例如，从UI中的组合框）：

```js
s.setSelectedServer(server)
```

其中`server`是您要使用的服务器。

### 运行测试

最后，我们可以运行测试：

```js
s.start();
```

在测试过程中，您的`onupdate`事件处理程序将定期被调用，提供可用于更新UI的数据。测试结束时，将调用您的`onend`处理程序。

您可以随时中止测试：

```js
s.abort();
```

测试完成后，您可以再次运行它，或者只是销毁`s`。

## 实现细节

本节的目的是帮助想要更改速度测试内部工作方式的开发人员。它将分为4个部分：`speedtest.js`、`speedtest_worker.js`、`backend`文件和`results`文件。

### `speedtest.js`

这是您的网页与速度测试之间的主要接口。它向页面隐藏了速度测试Web Worker，并提供了许多方便的函数来控制测试。

您可以将其视为有限状态机。以下是状态（使用getState()查看它们）：

* __0__: 在这里，您可以使用`setParameter("parameter",value)`函数更改速度测试设置（如测试持续时间）。从这里，您可以使用`start()`开始测试（进入状态3），或者使用`addTestPoint(server)`或`addTestPoints(serverList)`添加多个测试点（进入状态1）。此外，这是设置`onupdate(data)`和`onend(aborted)`事件回调的最佳时机。
* __1__: 在这里，您可以添加测试点。只有当您想使用多个测试点时才需要这样做。
    服务器定义为如下对象：

    ```jsonc
    {
        name: "用户友好名称",
        server:"http://yourBackend.com/",   //  <---- 服务器URL。您可以指定http://或https://。如果您的服务器同时支持两者，只需写//而不写协议
        dlURL:"garbage.php" //   <----- 服务器上garbage.php或其替代品的路径
        ulURL:"empty.php"  //  <----- 服务器上empty.php或其替代品的路径
        pingURL:"empty.php"  //  <----- 服务器上empty.php或其替代品的路径。选择器用此ping服务器
        getIpURL:"getIP.php"  //  <----- 服务器上getIP.php或其替代品的路径
    }
    ```

    在状态1中，您只能添加测试点，不能更改测试设置。完成后，使用selectServer(callback)选择ping最低的测试点。这是异步的，完成后，它将调用您的回调函数并移动到状态2。调用setSelectedServer(server)将手动选择服务器并移动到状态2。
* __2__: 测试点已选择，准备开始测试。使用`start()`开始，这将移动到状态3
* __3__: 测试运行中。在这里，您的`onupdate`事件回调将定期被调用，接收来自工作线程的速度和进度数据。数据对象将传递给您的`onupdate`函数，包含以下项目：
        - `dlStatus`: 下载速度（Mbit/s）
        - `ulStatus`: 上传速度（Mbit/s）
        - `pingStatus`: ping值（ms）
        - `jitterStatus`: jitter值（ms）
        - `dlProgress`: 下载测试进度（浮点数0-1）
        - `ulProgress`: 上传测试进度（浮点数0-1）
        - `pingProgress`: ping/jitter测试进度（浮点数0-1）
        - `testState`: 测试状态（-1=未开始，0=开始中，1=下载测试，2=ping+jitter测试，3=上传测试，4=完成，5=中止）
        - `clientIp`: 执行测试的客户端IP地址（可选包含ISP和距离）
    测试结束时，将调用`onend`函数，并传入一个布尔值，指定测试是被中止还是正常结束。
    测试可以随时用`abort()`中止。
    测试结束后，它将移动到状态4
* __4__: 测试完成。如果需要，您可以通过调用`start()`再次运行它。

#### Speedtest类中的函数列表

##### getState()

返回测试状态：0=添加设置，1=添加服务器，2=服务器选择完成，3=测试运行中，4=完成

##### setParameter(parameter,value)

设置测试参数

* parameter: 要设置的参数名称的字符串
* value: 参数的新值

无效值或不存在的参数将被速度测试工作线程忽略。

##### addTestPoint(server)

添加测试点（多测试点）

* server: 要添加的服务器对象。必须包含以下元素：

    ```jsonc
    {
        name: "用户友好名称",
        server:"http://yourBackend.com/",   // 服务器URL。您可以指定http://或https://。如果您的服务器同时支持两者，只需写//而不写协议
        dlURL:"garbage.php", // 服务器上garbage.php或其替代品的路径
        ulURL:"empty.php", // 服务器上empty.php或其替代品的路径
        pingURL:"empty.php", // 服务器上empty.php或其替代品的路径。选择器用此ping服务器
        getIpURL:"getIP.php", // 服务器上getIP.php或其替代品的路径
    }
    ```

请注意，这将向发送给速度测试工作线程的参数添加`mpot`:`true`。

##### addTestPoints(list)

与addTestPoint相同，但您可以传递服务器数组

##### loadServerList(url,result)

从`url`指向的JSON文件加载服务器列表。

此过程是异步的，完成后将调用`result`函数。如果请求成功，将向函数传递包含加载的服务器列表的数组，否则将传递`null`。

##### getSelectedServer()

返回选定的服务器（多测试点）

##### setSelectedServer()

手动选择测试点之一（多测试点）

##### selectServer(result)

从添加的测试点列表中自动选择服务器。将选择ping最低的服务器。（多测试点）

如果服务器列表很长，选择器会并行检查多个服务器（默认：6个流）以加快速度。

此过程是异步的，完成后将调用传递的`result`回调函数，然后可以开始测试。

##### start()

开始测试。

注意（多测试点）：选定的服务器将添加到`telemetry_extra`字符串中。如果此字符串已设置，则`telemetry_extra`将是包含服务器和原始字符串的JSON字符串

测试期间，`onupdate(data)`回调函数将定期被调用，接收来自工作线程的数据。
测试结束时，将调用`onend(aborted)`函数，传入一个布尔值，告诉您测试是被中止还是正常结束。

##### abort()

在测试运行时中止测试。

### `speedtest_worker.js`

这是实际速度测试代码所在的地方。它从主线程接收设置，运行测试，并报告结果。

工作线程接受3个命令：

* `start`: 开始测试。可选地，测试设置可以作为JSON字符串在start和空格后传递
* `status`: 以JSON字符串形式返回当前状态。状态字符串内容与制作自定义前端部分中的事件处理程序部分中描述的相同。
* `abort`: 中止测试

#### 参数

除了制作自定义前端部分的测试设置部分中列出的参数外，还有一个额外的设置：

* `mpot`: 设置为true可使用多测试点运行测试。这将向所有请求添加`cors=true`（所有响应将包含CORS头）并启用一些额外的优化。
    默认值: `false`

#### 下载测试

下载测试通过使用XHR将大型垃圾数据blob从服务器传输到客户端来执行。

测试使用多个流。如果一个流完成下载，它将重新启动。每个流的下载数据量使用XHR Level 2 `onprogress`事件进行跟踪。

测试流并不完全同步，因为我们不希望它们在完成时同时结束（如果它们这样做的话）。

每200ms，一个计时器会使用当前速度更新`dlStatus`字符串，并根据速度的高低计算一个"奖励"时间来缩短测试（当`time_auto`设置为`true`时）。

有关更多实现细节，请参见代码。

#### 上传测试

这与下载测试类似，但方向相反。生成一个大型垃圾数据blob，并使用多个流重复发送到服务器。

为了跟踪传输的数据量，使用XHR Level 2 `upload.onprogress`事件。

此测试有几个复杂之处：

* 某些浏览器没有正常工作的`upload.onprogress`事件。为此，我们使用小blob而不是大blob，并使用`onload`事件跟踪进度。这被称为IE11解决方法（但在某些版本的Edge和Safari中也发现了相同的错误）
* 当`mpot`设置为`true`时，必须先发送一个空请求以加载CORS头，然后才能开始测试

有关更多实现细节，请参见代码。

#### Ping + Jitter测试

Ping/Jitter测试__不是ICMP ping__。这是一个常见的误解。您不能通过HTTP使用ICMP，当然也不能在浏览器中使用。

此测试通过创建到服务器的持久HTTP连接，然后重复下载一个空文件，并测量请求和响应之间的时间来工作。

计时可以作为简单的时间戳差异测量，或者如果可用，使用Performance API。

Jitter是ping时间的方差。

有关更多实现细节，请参见代码。

### `backend`文件

#### `garbage.php`

使用OpenSSL为下载测试生成不可压缩的垃圾数据流。

它接受一个`ckSize` GET参数，指定要生成的垃圾数据量（以MB为单位，4-1024）。

#### `empty.php`

用于上传和ping测试的空文件。它只发送头来创建连接。

#### `getIP.php`

返回客户端IP、ISP和与服务器的距离。

GET参数：

* `isp`: 如果设置，从ipinfo.io获取ISP信息
* `distance`: 如果设置，计算与服务器的距离。您可以指定`km`或`mi`作为格式。

如果设置了`isp`，输出是一个包含以下内容的JSON字符串：

* `processedString`: 可以显示给用户的字符串
* `rawIspInfo`: 关于客户端的信息，作为JSON字符串，直接来自ipinfo.io

如果未设置`isp`，输出只是包含客户端IP地址的字符串。

注意：如果您的服务器位于某些代理、防火墙、VPN等后面，可能无法正确检测客户端的IP地址。如果发生这种情况，您必须分析来自客户端的流量，以找到包含原始IP地址的HTTP头的名称。`getIP.php`包含其中一些头，但不是全部。

#### CORS头

如果向这些文件传递GET参数`cors=true`，它们将发送以下CORS头：

```header
Access-Control-Allow-Origin: *
Access-Control-Allow-Methods: GET, POST
Access-Control-Allow-Headers: Content-Encoding, Content-Type
```

### `results`文件

#### `telemetry.php`

此文件将遥测信息存储到数据库中。

数据作为POST参数传递：

* `ispinfo`: ISP信息（如果启用，否则为空字符串）
* `extra`: 传递给工作线程的`telemetry_extra`字符串（如果设置，否则为空字符串）
* `dl`: 下载速度
* `ul`: 上传速度
* `ping`: ping时间
* `jitter`: jitter值
* `log`: 遥测日志（如果`telemetry_level`设置为`full`或更高，否则为空字符串）

#### `index.php`

为给定的测试ID生成可共享的结果图像。

GET参数：

* `id`: 要显示的测试的ID

此图像的外观可以通过编辑此文件中的变量来自定义。

#### `idObfuscation.php`

包含ID混淆和解混淆的实现。

有关实现细节，请参见代码，它基本上是一系列位操作。

#### `stats.php`

用于显示和搜索测试结果的简单UI。运行测试不需要此文件。

## 替代后端

如果由于某种原因您不能或不想使用PHP，速度测试可以与其他后端甚至无后端（功能有限）一起运行。

您将需要替换`backend/garbage.php`和`backend/empty.php`，以及可选的`backend/getIP.php`，并且测试需要知道在哪里找到它们：

```js
//速度测试初始化
var s=new Speedtest();
...
//自定义后端
setParameter("url_dl","您的garbage.php替代品的URL");
s.setParameter("url_ul","您的empty.php替代品的URL");
s.setParameter("url_ping","您的empty.php替代品的URL");
s.setParameter("url_getIp","您的getIP.php替代品的URL");
```

### `garbage.php`的替代品

`garbage.php`的替代品必须生成不可压缩的垃圾数据。

一个大文件（10-100 MB）是可能的替代品。您可以在此处获取一个[http://downloads.fdossena.com/geth.php?r=speedtest-bigfile](http://downloads.fdossena.com/geth.php?r=speedtest-bigfile)。

指向`/dev/urandom`的符号链接也是可以的。

如果您想制作自己的后端，请参见`garbage.php`的实现细节部分。

#### `empty.php`的替代品

您的替代品必须简单地响应HTTP代码200，不发送其他内容。您可能希望发送额外的头来禁用缓存。测试假设服务器发送`Connection:keep-alive`。

一个空文件可以用于此目的。

如果您想制作自己的后端，请参见`empty.php`的实现细节部分。

#### `getIP.php`的替代品

您的替代品可以简单地以纯文本形式响应客户端的IP，或者做更复杂的事情。

如果您想制作自己的后端，请参见`getIP.php`的实现细节部分。

### 无后端

速度测试可以运行，尽管功能有限，仅使用Web服务器作为后端，没有PHP或其他服务器端脚本。

您将能够运行下载和上传测试，但没有IP、ISP和距离检测，没有遥测和结果共享，只有一个测试点。

要做到这一点，您需要：

* `garbage.php`的替代品：一个大的不可压缩文件，如[这个](http://downloads.fdossena.com/geth.php?r=speedtest-bigfile)。我们将其称为`backend/garbage.dat`
* `empty.php`的替代品：一个空文件即可。我们将其称为`backend/empty.dat`

现在，您需要配置测试以使用它们。找到`s=new Speedtest()`并在其下方放置以下内容：

```js
s.setParameter("url_dl","backend/garbage.dat");
s.setParameter("url_ul","backend/empty.dat");
s.setParameter("url_ping","backend/empty.dat");
s.setParameter("test_order","P_D_U");
```

这将指向我们的静态文件，并将测试设置为仅执行ping/jitter、下载和上传测试。

## 故障排除

以下是用户报告的最常见问题以及如何解决它们。如果您仍然需要帮助，请通过[info@fdossena.com](mailto:info@fdossena.com)联系我。

### 下载测试结果非常低

garbage.php和empty.php（或您的替代品）是否可达？
按F12，选择网络并开始测试。您是否看到错误？（取消的请求不是错误）
如果开始了小下载，在文本编辑器中打开它。它是否说缺少openssl_random_pseudo_bytes()？在这种情况下，安装OpenSSL（在大多数发行版上安装Apache和PHP时通常会包含）。

#### 上传测试不准确，和/或我看到延迟峰值

检查您的服务器的最大POST大小，确保它至少为20MB，可能更多

#### 下载和/或上传结果略过乐观

测试经过微调，以在典型的IPv4互联网连接上运行。如果您在不同条件下使用它，请参见`overheadCompensationFactor`参数。

#### 所有测试都错误，给出极高的结果，浏览器滞后/崩溃

您正在localhost上运行测试，因此它试图测量环回接口的速度。测试旨在通过互联网连接从不同的机器上运行。

#### Ping测试显示实际ping的两倍

确保您的服务器正在发送`Connection:keep-alive`头

#### 服务器位于负载均衡器、代理等后面，我得到错误的IP地址

编辑getIP.php并将第14-23行替换为更适合您场景的内容。
例如：`$ip = $_SERVER['HTTP_X_FORWARDED_FOR'];`

#### 结果共享只生成空白图像

如果图像不显示，浏览器显示损坏的图像图标，则FreeType2未正确安装或配置。
如果图像是空白的，这通常是因为PHP无法在`results`文件夹中找到字体文件。您可以修复PHP配置或编辑`results/index.php`并为字体使用绝对路径。这是[PHP的已知问题](http://php.net/manual/en/function.imagefttext.php)，没有已知的真正解决方案。

#### 我的服务器位于Cloudflare后面，我无法在某些测试上达到全速

这不是与速度测试相关的问题，因为它几乎可以在任何HTTP文件上传/下载中复制。
转到您域名的DNS设置，将"DNS和HTTP代理（CDN）"更改为"仅DNS"，并等待设置应用（可能需要几分钟）。

#### 在Windows Server上，使用IIS，上传测试不起作用，控制台中可见CORS错误

这是配置问题。在wwwroot中创建一个名为web.config的文件，并调整以下代码：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<configuration>
  <system.webServer>
    <cors enabled="true" failUnlistedOrigins="false">
      <add origin="*">
        <allowHeaders allowAllRequestedHeaders="true" />
        <allowMethods>
          <add method="GET" />
          <add method="POST" />
          <add method="PUT" />
          <add method="DELETE" />
          <add method="OPTIONS" />
        </allowMethods>
        <exposeHeaders>
        </exposeHeaders>
      </add>
    </cors>
  </system.webServer>
</configuration>
```

#### ID混淆不起作用（输出不正确，结果图像空白）

ID混淆仅在64位PHP上工作（需要PHP_INT_SIZE为8）。
请注意，Windows上较旧版本的PHP 5即使是64位，也使用PHP_INT_SIZE为4。如果您处于这种情况，请更新您的PHP安装。

此外，确保Web服务器对`results`文件夹有写入权限。

## 已知的错误和限制

### 一般

* ping/jitter测试通过查看空XHR完成所需的时间来测量。它不是实际的ICMP ping。不同的浏览器也可能显示不同的结果，特别是在慢速设备上的非常快的连接上。

### IE特定

* 在具有高延迟的非常快的连接上，上传测试不精确（可能会被Edge 17修复）
* 在IE11上，在未知条件下错误地触发相同来源策略错误。似乎与从非标准URL（例如顶级域名<http://abc/speedtest>）运行测试有关。这些是IE11的相同来源策略实现中的错误，而不是速度测试本身的错误。
* 在IE11上，在未知情况下，在某些系统上测试只能运行一次，之后IE直到重新启动浏览器才会加载speedtest_worker.js。这是IE11的罕见错误。

### Firefox特定

* 在某些关闭硬件加速的Linux系统上，页面渲染使浏览器滞后，降低ping/jitter测试的准确性，甚至可能降低非常快连接上的下载和上传测试的准确性。

## 贡献

由于这是一个开源项目，您可以修改它。

如果您做了一些您认为应该进入主项目的更改，请在GitHub上发送Pull Request，或通过[info@fdossena.com](mailto:info@fdossena.com)联系我。
我们不要求您使用特定的编码约定，以任何您想要的方式编写代码，我们将在必要时更改格式。

捐赠也很受欢迎：您可以通过[PayPal](https://www.paypal.me/sineisochronic)或[Liberapay](https://liberapay.com/fdossena/donate)捐赠。

## 许可证

此软件受GNU LGPL许可证版本3或更新版本的保护。

简而言之：您可以免费使用、学习、修改和重新分发此软件和其修改版本，无论是免费还是收费。
您也可以在专有软件中使用它，但对此软件的所有更改必须保持在相同的GNU LGPL许可证下。

有关其他许可模式，请通过[info@fdossena.com](mailto:info@fdossena.com)联系我。
