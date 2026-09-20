
https://www.xiaolincoding.com/network/2_http/http_rpc.html#http-%E5%92%8C-rpc-%E6%9C%89%E4%BB%80%E4%B9%88%E5%8C%BA%E5%88%AB
# 问题汇总

## 1. Consul 和 etcd 有什么区别？

**核心定位不同**：etcd 是专注的**分布式键值存储**，Consul 是**端到端的服务发现与配置管理框架**。

| 维度 | etcd | Consul |
| :--- | :--- | :--- |
| 核心定位 | 强一致 KV 存储，单一职责 | 服务发现与配置管理框架 |
| KV 存储 | 强项，支持 MVCC、可靠 Watch、迷你事务，适合百万级键值 | 弱项，规模小（数百 MB），无 MVCC |
| 服务发现 | 需自行构建 | 内置核心功能，DNS/HTTP 直接查询 |
| 健康检查 | 基础租约机制 | 功能丰富，分布式本地 Agent 执行 |
| 多数据中心 | 不支持原生多 DC | 原生支持，突出优势 |
| 接口 | HTTP/gRPC | HTTP、DNS、Web UI |
| 读延迟 | 网络 RTT | 网络 RTT + fsync |

**选择建议**：要纯高性能 KV 选 etcd（K8s 核心组件）；要开箱即用的服务发现 + 多数据中心选 Consul。

---

## 2. 字节跳动内部 RPC 框架用的是什么？

主要是 **Kitex**（Go）和 **Volo**（Rust），均属其开源的 **CloudWeGo** 中间件集合。

- **Kitex**：字节内部 Go 微服务 RPC 核心，承载上万微服务，内置自研高性能网络库 **Netpoll**，主要支持 **Thrift** 协议。
- **Volo**：面向 Rust 的新一代高性能 RPC 框架。
- **澄清**：Kratos 并非字节自研，通常被认为是 B 站开源项目。

---

## 3. Kitex 底层用什么实现的？

核心是**自研高性能网络库 Netpoll**，而非 Go 原生 net 库。

- **网络层**：基于 Linux **epoll**，采用**主从 Reactor 模式**（MainReactor 接连接，SubReactor 管读写，事件提交协程池）。自研 EpollWait 带来约 **10%** 吞吐提升。
- **内存层**：**Nocopy Buffer（LinkBuffer）**，链表结构由多个 `[]byte` block 串联，读写分别操作头尾指针实现无锁并行；扩缩容无需拷贝，block 引用计数 + 对象池管理，大幅减少 GC。
- **协议层**：默认支持 Thrift，结合 Nocopy Buffer 实现 **Nocopy Thrift** 零拷贝编解码。

---

## 4. Kitex 和 Consul 有关系吗？

有关系，是 **"框架"与"组件"** 的关系。Kitex 是 RPC 框架，Consul 是服务注册中心。

Kitex 通过抽象的 `Registry` / `Resolver` 接口实现注册中心插件化：

- **服务端注册**：`kitex-contrib/registry-consul` 的 `NewConsulRegister` + `server.WithRegistry(r)`
- **客户端发现**：同包的 `NewConsulResolver` + `client.WithResolver(r)`
- **配置中心**：`kitex-contrib/config-consul` 可动态管理限流、重试、熔断等

两者是可灵活搭配的"好搭档"，Kitex 专注通信性能，Consul 专注服务治理基础设施。

---

## 5. 字节内部是不是就是 Kitex + Consul？

**不是简单的固定组合。**

- 字节从 **2016 年**启动 K8s 上线时就开始基于 **Consul 实现服务发现**，Consul 在字节基础设施中确实扮演过重要角色。
- 但 **Kitex 的注册中心是插件化的**，通过统一接口对接 Consul、etcd、Nacos 等，Consul 只是其中之一。
- 真实情况更可能是：**Kitex 作为 RPC 框架是统一的，底层注册中心因业务线、历史阶段和基础设施不同而有差异。**

---

## 6. 介绍一下 Thrift 协议

Thrift 是 Facebook 开发、后进入 Apache 的**跨语言 RPC 框架**，核心是 **IDL 驱动**。

- **IDL 驱动**：用中立语言描述接口和数据结构，编译器生成目标语言代码（含序列化、网络通信逻辑）。
- **协议栈四层**：Transport（传输）、Protocol（编解码）、Processor（请求处理）、Server（运行模型）。Transport 和 Protocol 可自由组合。
- **通信流程**：客户端序列化 → Transport 发送 → 服务端反序列化 → Processor 分发 → 执行后回传。
- **与 gRPC 对比**：Thrift 自有 IDL、自定义二进制、性能极致但流式较弱；gRPC 基于 Protobuf + HTTP/2，原生支持双向流。
- **Kitex 选它的原因**：字节存量服务大量基于 Thrift；其分层设计给 Kitex 留出优化空间——Transport 层换 Netpoll，Protocol 层结合 Nocopy Buffer 实现零拷贝。

---

## 7. .pyi 是什么文件？

`.pyi` 是 **Python 的类型存根文件（Type Stub File）**，为类型检查工具（mypy、pyright）和 IDE 提供类型信息。

- **只写签名不写实现**：函数体用 `...` 占位
- **优先级高于 `.py`**：同目录下类型检查工具优先读 `.pyi`
- **不影响运行时**：解释器完全忽略
- **常见场景**：为第三方库补类型、C/C++ 扩展模块、标准库（typeshed）、项目内部接口隔离

```python
# math_utils.pyi
def add(a: int, b: int) -> int: ...
```

简单说，`.pyi` 就是 Python 世界的 **"头文件"**，只不过服务于类型检查器而非编译器。