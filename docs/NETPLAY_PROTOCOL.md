# 联机协议摘要（基于 DAxZE 客户端源码）

参考：https://github.com/usaginya/DAxZE （Apache-2.0），源码版本 `212051a`。
本文件只记录从客户端源码读出的内容。服务端行为是推断，需要实际测试确认。

## 组件

| 组件 | 位置（DAxZE 源码） | 作用 |
| --- | --- | --- |
| WebSocket 客户端 | `DAxZE/WebsocketCore.cs` | 连接 `wss://<服务器>`，申请远端地址 |
| 本地 UDP 转发 | `DAxZE/UDPForwarder.cs` | 在 `127.0.0.1` 上收发游戏的 UDP 数据 |
| 握手帧 | `CreateThNetworkStream` | 告诉服务端本地端口与远端端口 |

## 已从源码确认的内容

1. 客户端默认在 `127.0.0.1:10800` 上监听（`Local_port` 默认值）。
2. WebSocket 收到的文本消息按分隔符拆分，其中一段格式为 `IP:端口`，即服务端分配的远端地址。
3. 握手帧为 9 字节，所有整数为大端序：

   | 偏移 | 长度 | 内容 |
   | --- | --- | --- |
   | 0 | 2 | 本地端口（uint16） |
   | 2 | 2 | 远端端口（uint16） |
   | 4 | 2 | 帧长度，固定为 9 |
   | 6 | 3 | 零填充 |

4. 本地 UDP 收到的游戏数据，经 WebSocket 发送到远端。
5. 心跳：WebSocket 每 8 秒自动发送一次 ping。客户端还会对服务器地址执行 ICMP ping 以显示延迟。
6. 断线检测：`UDPForwarder` 连续 7 次未收到数据进入等待状态，连续 60 次则停止。

## 待测试的推断

- 服务端分配的 IP 与端口，是否就是游戏对端实际连接的地址。
- 9 字节帧之后，WebSocket 的二进制消息是否直接就是游戏的 UDP 载荷。
- 不同游戏版本（如 TH12.3）是否使用相同端口和协议。
- 服务端如何处理多个房间与并发连接。

## 移植说明

- 字节布局已用 `netplay/src/com/th123/netplay/ThNetworkStream.java` 实现，并由 `netplay/test/com/th123/netplay/ThNetworkStreamCheck.java` 检查通过。
- UDP 转发和 WebSocket 收发尚未移植，需要在 Android 上验证。
- 本项目的服务端如需兼容，只能依据本文件描述的客户端行为实现，不能依赖 MyCard 的服务端。
