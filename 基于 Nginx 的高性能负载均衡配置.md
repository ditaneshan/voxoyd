**高可用架构设计之负载均衡与容灾恢复**

**引言**
在现代 [分布式系统](https://home-ayx-app.com.cn) 中，高可用并不等于“永不宕机”，而是通过架构设计把故障影响控制在可接受范围内。负载均衡负责分散压力、提高吞吐；容灾恢复负责在节点、机房甚至区域级故障发生时快速切换，保证服务连续性。两者结合，才构成真正可落地的高可用体系。

**核心原理分析**
负载均衡通常分为四层和七层两类：四层基于 IP 和端口，性能高、开销小；七层可感知 HTTP 内容，适合按路径、Cookie、Header 做精细路由。生产中还要配合健康检查、权重调度、会话保持与限流，避免把流量继续打到异常实例。

容灾恢复的关键在于“可检测、可切换、可回滚”。常见策略包括同城双活、异地多活、主备切换与数据定期演练。恢复目标要明确两个指标：RTO（恢复时间）和 RPO（可接受数据丢失量）。如果没有明确的 RTO/RPO，再复杂的容灾方案也难以验证价值。

**代码示例**
下面示例演示了一个简单的多节点请求重试与故障切换逻辑，可用于客户端侧的基础容灾：

```python
import time
import requests

BACKENDS = [
    "https://api-a.example.com",
    "https://api-b.example.com",
    "https://api-c.example.com",
]

def request_with_failover(path, timeout=2, retries=3):
    last_error = None
    for backend in BACKENDS:
        for i in range(retries):
            try:
                resp = requests.get(f"{backend}{path}", timeout=timeout)
                resp.raise_for_status()
                return resp.json()
            except Exception as e:
                last_error = e
                time.sleep(0.2 * (2 ** i))
        # 当前节点连续失败，切换到下一个可用节点
    raise RuntimeError(f"all backends failed: {last_error}")

data = request_with_failover("/health")
print(data)
```

这段代码的价值在于：当某个节点或区域抖动时，请求会自动转移到其他后端，减少单点故障带来的级联影响。

**总结**
高可用架构的核心不是“消灭故障”，而是“让故障可控”。负载均衡解决的是流量分配问题，容灾恢复解决的是故障切换问题。实践中应结合健康检查、重试退避、熔断降级、跨地域部署和演练机制，才能真正提升系统韧性。

**相关技术资源**
- https://home-ayx-app.com.cn
