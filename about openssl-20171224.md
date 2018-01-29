### 关于OpenSSL社区
OpenSSL功能简介，OpenSSL外围包提供了如下三种功能：
1. 命令行工具，用来完成各种各样密码学相关的任务。例如，创建证书、解析证书、加密文件、加密算法测试等。
2. 全面的、可扩展的密码学库（libcrypto）。覆盖了大多数标准定义的加密算法，可用硬件扩展或者加速。
3. 符合SSL/TLS协议的加密通讯库（libssl）。提供客户端和服务端进行加密通讯的的能力。

### OpenSSL版本号规定
OpenSSL的版本号的格式为：n1.n2.n3x（例如OpenSSL-1.0.2k）。其中n1, n2, n3是数字，x是字母。OpenSSL的版本分为：大版本(Major Release)、小版本(Minor Release)、字母版本(Letter Release)。
* 大版本(Major Release)：n1或者n2变化，该版本的变化会导致OpenSSL相关的二进制兼容性问题，升级时要注意。
* 小版本(Minor Release)：n3变化，该版本变化是因为新增一些特性，但是这些特性不会导致OpenSSL二进制的兼容性有问题。
* 字母版本(Letter Release)：x变化，该版本变化是因为修复bug和安全问题（CVE修复），不包含任何新特性。

### OpenSSL发布计划
OpenSSL社区对1.0.0大版本高度认可，认为其对开发者和厂商都十分友好。OpenSSL项目的发布策略 如下：
* Version 1.1.0 will be supported until 2018-08-31.
* Version 1.0.2 will be supported until 2019-12-31 (LTS).
* Version 1.0.1 is no longer supported.
* Version 1.0.0 is no longer supported.
* Version 0.9.8 is no longer supported.

OpenSSL社区每四年会指定一个LTS的OpenSSL版本，OpenSSL社区对LTS版本的OpenSSL至少会支持5年。非LTS版本的OpenSSL设置至少会支持2年。OpenSSL社区支持的意思是超过支持年之后，社区只接纳CVE相关的commit。

### 关于OpenSSL的安全议题

### 关于OpenSSL使用方法

### 关于OpenSSL的软件架构

### 关于OpenSSL的Coding Style

### 关于OpenSSL源码须知

### 关于OpenSSL主要代码实现

### 关于OpenSSL的Iusse Tracking