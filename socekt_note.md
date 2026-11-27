# 主进程与子进程的 Socket 通信

主进程和 namespace 子进程通过 Unix `SOCK_SEQPACKET` 交换两条消息。

1. 主进程创建 listener，并通过 `--control-socket` 把路径交给子进程。子进程主动连接，主进程接受连接并校验对端 UID。 [创建 listener](daemon.go#L80) · [子进程连接](namespace.go#L63) · [接受连接](daemon.go#L264)
2. 子进程完成网络初始化后，用 `sendProxyFiles` 发送 4 字节大端 MTU，通过 `SCM_RIGHTS` 附带 TUN FD。开启 FakeDNS 时，第二个 FD 是 DNS socket。主进程用 `receiveProxyFiles` 接收并校验。 [发送 FD](namespace.go#L288) · [接收 FD](daemon.go#L224)
3. 主进程初始化代理后发送一个字节 `0x01`，子进程用 `waitReady` 等待并校验。 [发送 READY](daemon.go#L260) · [等待 READY](namespace.go#L298)
4. 双方关闭控制连接。子进程关闭本地网络 FD 副本，再通过 `unix.Exec` 执行目标命令。主进程继续运行代理并等待目标命令退出。 [主进程关闭连接](daemon.go#L202) · [子进程关闭连接](namespace.go#L253) · [执行目标命令](namespace.go#L272)
