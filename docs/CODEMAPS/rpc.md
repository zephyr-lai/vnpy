# RPC Codemap

**Last Updated:** 2026-09-30
**Total Files:** 4
**Total Lines:** ~327

## Entry Points

- `vnpy/rpc/__init__.py` — 导出 `RpcServer`、`RpcClient`
- `examples/simple_rpc/` — 服务端/客户端示例
- `examples/client_server/` — 分布式部署示例

## Architecture

```
rpc/
├── server.py   # RpcServer：REP 路由 + PUB 事件广播
├── client.py   # RpcClient：REQ 调用 + SUB 订阅回调
└── common.py   # 默认地址约定
```

## Key Modules

| File | Lines | 职责 |
|------|-------|------|
| `vnpy/rpc/client.py` | 169 | `RpcClient`：`__getattr__` 代理远程方法为同步调用；`subscribe(topic, callback)` 收推送 |
| `vnpy/rpc/server.py` | 140 | `RpcServer`：`register(event, function)` 暴露方法；`rep_router` 处理 REQ；`publish(event, data)` 广播 |
| `vnpy/rpc/common.py` | 10 | `REP_ADDRESS` / `PUB_ADDRESS` 默认地址 |

## Data Flow

```
Client ──REQ(方法名+args)──▶ Server.REP 路由 ──▶ 本地函数 ──返回值──▶ Client
Server ──PUB(topic, data)────────▶ Client.SUB ──▶ callback(data)
```

## 典型用途

把 `MainEngine`/`OmsEngine` 注册进 `RpcServer`，远程客户端即可像本地对象一样调 `main_engine.get_all_orders()` —— VeighNa 分布式部署的传输层。

## External Dependencies

`pyzmq`（ZMQ_REQ/REP + PUB/SUB）。
