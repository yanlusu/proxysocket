proxysocket
===========
English: [README.md](./README.md)

用于代理客户端支持的跨平台 C 语言库，可作为 `connect()` 的即插即用替代方案。

说明
----
该库用于通过代理建立 TCP 连接，支持跨平台。

支持的连接方式：
 - 不使用代理（可选绑定本地地址和/或端口）
 - HTTP 代理：仅支持 CONNECT 方法，仅支持无认证或 Basic 认证
 - SOCKS4/SOCKS4A：不支持 IDENT 功能
 - SOCKS5（RFC 1928）：仅支持用户名/密码认证或无认证

特性：
 - 当前仅支持 IPv4 TCP 连接
 - 返回标准操作系统 `SOCKET`，可使用系统 `send()`、`recv()` 等函数处理
 - 可选在代理端执行域名解析
 - 支持多级代理链（daisy-chaining）
 - 也可用于直连（不走代理），并可选绑定本地地址和/或端口
 - 包含 `ipify.org` 示例程序，用于检测公网出口 IP
 - 支持 Windows、Linux、macOS
 
目标
----
在 Windows、Linux、macOS 上保持可移植性。

返回的 `SOCKET` 与系统 `socket()` 返回句柄兼容，便于低成本替换 `socket()` + `connect()` 流程。

仅支持 IPv4 TCP；除所列代理协议外，不内置 HTTP/SSL 等上层协议支持。

依赖
----
无

许可证
------
proxysocket 采用 MIT License，详见 `LICENSE.txt`。


使用 CMake 构建
---------------
```bash
cmake -S . -B build
cmake --build build
```

安装：
```bash
cmake --install build --prefix /usr/local
```

可选构建参数：
 - `-DBUILD_SHARED_LIBS=ON|OFF`
 - `-DPROXYSOCKET_BUILD_TOOLS=ON|OFF`
 - `-DPROXYSOCKET_DEBUG_POSTFIX=d`
 - `-DPROXYSOCKET_RELWITHDEBINFO_POSTFIX=rd`

构建类型示例：
```bash
cmake -S . -B build-debug -DCMAKE_BUILD_TYPE=Debug -DBUILD_SHARED_LIBS=ON
cmake -S . -B build-release -DCMAKE_BUILD_TYPE=Release -DBUILD_SHARED_LIBS=ON
cmake -S . -B build-rd -DCMAKE_BUILD_TYPE=RelWithDebInfo -DBUILD_SHARED_LIBS=ON
```

在其他 CMake 项目中使用
-----------------------
安装后可这样引用：
```cmake
find_package(proxysocket CONFIG REQUIRED)
target_link_libraries(your_target PRIVATE proxysocket::proxysocket)
```

如果 proxysocket 安装在非系统前缀，请设置：
```bash
cmake -S . -B build-consumer -DCMAKE_PREFIX_PATH=/path/to/proxysocket/install
```
