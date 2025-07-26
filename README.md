![LibreSpeed Logo](https://github.com/librespeed/speedtest/blob/master/.logo/logo3.png?raw=true)

# LibreSpeed

无需Flash，无需Java，无需Websocket，拒绝花里胡哨。

这是一个非常轻量级的速度测试工具，使用Javascript、XMLHttpRequest和Web Workers实现。

## 立即试用

[进行速度测试](https://librespeed.org)（全是国外节点）
[国内可参考中科大测速站](https://test.ustc.edu.cn/)

## 兼容性

支持所有现代浏览器：IE11、最新版Edge、最新版Chrome、最新版Firefox、最新版Safari。
移动版本同样适用。

## 功能特点

* 下载速度测试
* 上传速度测试
* 延迟测试(Ping)
* 抖动测试(Jitter)
* IP地址、互联网服务提供商(ISP)、与服务器的距离（可选）
* 遥测数据收集（可选）
* 测试结果分享（可选）
* 多测试点支持（可选）

![速度测试运行截图](https://speedtest.fdossena.com/mpot_v6.gif)

## 服务器要求

* 性能良好的Web服务器（Apache 2，也支持nginx、IIS）
* PHP 5.4或更高版本（也支持其他后端）
* MariaDB或MySQL数据库用于存储测试结果（可选，也支持Microsoft SQL Server、PostgreSQL和SQLite）
* 高速互联网连接

## 安装方法

假设您已经安装了PHP和Web服务器，安装步骤非常简单。

1. 下载源代码并解压
2. 将以下文件复制到Web服务器的共享文件夹（例如Apache的/var/www/html/speedtest）：index.html、speedtest.js、speedtest_worker.js、favicon.ico和backend文件夹
3. （可选）如果使用nginx作为前端，我（oi-io）在项目根目录提供了一个nginx配置实例文件speedtest-zh.conf，请参考其配置nginx启用php
4. （可选）复制results文件夹，并使用其中的配置文件设置数据库
5. 确保您的权限允许执行（755）
6. 访问YOURSITE/speedtest/index.html，即可开始使用！

### 安装视频

这段视频展示了独立LibreSpeed服务器的安装过程：[Debian 12快速入门安装指南](https://fdossena.com/?p=speedtest/quickstart_deb12.frag)

后续将添加更多视频。

### 基于尊重著作权的原则（即使是开源软件），本分支选择在没有对原版做出重大更改之前尽量和原版的所有文件、样式保持一致，但是目前只翻译了LibreSpeed本身，下面列举的版本未做翻译。

## Android应用

用于构建LibreSpeed Android客户端的模板可在[此处](https://github.com/librespeed/speedtest-android)获取。

## 命令行客户端

命令行客户端可在[此处](https://github.com/librespeed/speedtest-cli)获取。

## Docker

Docker镜像可在[GitHub](https://github.com/librespeed/speedtest/pkgs/container/speedtest)上获取，请查看我们的[Docker文档](doc_docker.md)了解更多信息。
该镜像每周构建一次，以包含更新的ipinfo-DB用于ISP检测。这也确保安装了PHP的最新安全补丁。因此，我们建议使用`latest`镜像。

## Go后端

Go实现可在[`speedtest-go`](https://github.com/librespeed/speedtest-go)仓库中找到，由[Maddie Zhan](https://github.com/maddie)维护。

## Rust后端

Rust实现可在[`speedtest-rust`](https://github.com/librespeed/speedtest-rust)仓库中找到，由[Sudo Dios](https://github.com/sudodios)维护。

## Node.js后端

部分Node.js实现可在`node`分支中找到，由[dunklesToast](https://github.com/dunklesToast)开发。目前不建议使用。

## 捐赠

[![使用Liberapay捐赠](https://liberapay.com/assets/widgets/donate.svg)](https://github.com/librespeed/speedtest)
[使用PayPal捐赠](https://github.com/librespeed/speedtest)

## 许可证

版权所有 (C) 2016-2024 Federico Dossena

本程序是自由软件：您可以重新分发和/或修改
它根据GNU Lesser General Public License的条款，由自由软件基金会发布，
无论是版本3的许可证，还是（根据您的选择）任何更高版本。

本程序的分发是希望它起到一定作用，
但不做任何保证；也没有暗示的保证适销性或特定用途的适用性。
请参阅GNU通用公共许可证以获取更多详细信息。

您应该已经与本程序一起收到了GNU Lesser General Public License的副本
。如果没有，请参阅<https://www.gnu.org/licenses/lgpl>。
