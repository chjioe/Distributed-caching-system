# 分布式缓存系统

一个基于c++实现的高性能分布式缓存系统，采用一致性哈希算法进行数据分片，支持HTTP和gRPC双协议访问。

## 🚀 特性

- **分布式架构**: 支持多节点集群部署
- **一致性哈希**: 自动数据分片和负载均衡
- **双协议支持**: HTTP REST API + gRPC内部通信
- **容器化部署**: 支持Docker和Docker Compose
- **高性能**: 基于c++和gRPC构建
- **易于扩展**: 模块化设计，便于添加新功能

## 📋 系统要求

- c++
- Docker 
- Docker Compose 

## 🏗️ 架构设计

### 核心组件

```
┌─────────────────┐    ┌─────────────────┐    ┌─────────────────┐
│    Client       │    │    Client       │    │    Client       │
└─────────┬───────┘    └─────────┬───────┘    └─────────┬───────┘
          │ HTTP                 │ HTTP                 │ HTTP
          ▼                      ▼                      ▼
┌─────────────────┐    ┌─────────────────┐    ┌─────────────────┐
│   Server 1      │◄──►│   Server 2      │◄──►│   Server 3      │
│   Port: 9527    │gRPC│   Port: 9528    │gRPC│   Port: 9529    │
│                 │    │                 │    │                 │
│ ┌─────────────┐ │    │ ┌─────────────┐ │    │ ┌─────────────┐ │
│ │   Storage   │ │    │ │   Storage   │ │    │ │   Storage   │ │
│ └─────────────┘ │    │ └─────────────┘ │    │ └─────────────┘ │
└─────────────────┘    └─────────────────┘    └─────────────────┘
```

### 技术栈

- **gRPC**: 高性能RPC通信
- **一致性哈希**: 数据分片算法
- **Docker**: 容器化部署

## 📦 安装部署

### 方式一：Docker Compose (推荐)

1. 克隆项目
```bash
git clone <repository-url>
cd demo
```

2. 启动集群
```bash
docker-compose up --build
```

3. 验证服务
```bash
# 检查服务状态
curl http://localhost:9527/health
curl http://localhost:9528/health
curl http://localhost:9529/health
```


## 🔧 配置说明

### 节点配置

| 节点 | HTTP端口 | gRPC端口 | 主机名 |
|------|----------|----------|--------|
| server1 | 9527 | 50051 | server1 |
| server2 | 9528 | 50052 | server2 |
| server3 | 9529 | 50053 | server3 |

### 环境变量

- `NODE_ID`: 节点标识符 (server1/server2/server3)

## 📚 API 使用

### HTTP API

#### 设置缓存
```bash
curl -X POST http://localhost:9527 \
  -H "Content-Type: application/json" \
  -d '{"mykey": "myvalue"}'
```

#### 获取缓存
```bash
curl http://localhost:9527/mykey
```

#### 删除缓存
```bash
curl -X DELETE http://localhost:9527/mykey
```

### 响应格式

#### 成功设置
```json
{"success": true}
```

#### 成功获取
```json
{"mykey": "myvalue"}
```

#### 删除结果
```
1
```

#### 错误响应
```json
{"detail": "Not Found"}
```

## 🔍 工作原理

### 一致性哈希

1. **虚拟节点**: 每个物理节点创建100个虚拟节点
2. **哈希环**: 使用MD5算法将key映射到环上
3. **节点选择**: 顺时针查找最近的虚拟节点
4. **负载均衡**: 虚拟节点确保数据均匀分布

### 请求流程

1. 客户端发送HTTP请求到任意节点
2. 节点根据key计算哈希值确定负责节点
3. 如果是本地节点，直接处理请求
4. 如果是远程节点，通过gRPC转发请求
5. 返回处理结果给客户端

## 🧪 测试

### 功能测试

```bash
# 设置数据
curl -X POST http://localhost:9527 -H "Content-Type: application/json" -d '{"user:1": {"name": "Alice", "age": 25}}'
# 获取数据
curl http://localhost:9527/user:1
# 删除数据
curl -X DELETE http://localhost:9527/user:1
# 验证删除
curl http://localhost:9527/user:1
```

### 分布式测试

```bash
# 在不同节点设置数据
curl -X POST http://localhost:9527 -H "Content-Type: application/json" -d '{"key1": "value1"}'
curl -X POST http://localhost:9528 -H "Content-Type: application/json" -d '{"key2": "value2"}'
curl -X POST http://localhost:9529 -H "Content-Type: application/json" -d '{"key3": "value3"}'

# 从任意节点获取数据
curl http://localhost:9527/key2  # 可能会转发到server2
curl http://localhost:9528/key1  # 可能会转发到server1
curl http://localhost:9529/key3  # 本地处理
```

## 🤝 贡献
欢迎提交Issue和Pull Request来改进项目！

## 📄 许可证
MIT License

## 📞 联系
如有问题或建议，请创建Issue或联系维护者。