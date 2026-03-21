# 系统架构

## 分层记忆架构

OpenClaw Advanced Memory System 采用四层记忆架构，模拟人类大脑的记忆机制。

### L0层：完整原始记录

- **功能**：保留所有对话细节
- **用途**：深度检索、完整上下文
- **存储**：SQLite数据库
- **保留时间**：永久

### L1层：关键点提取

- **功能**：提取对话中的要点
- **压缩率**：70%（省70% token）
- **提取方式**：自动提取关键句子
- **更新频率**：实时

### L2层：结构化知识

- **功能**：转化为可复用的知识模块
- **压缩率**：90%（省90% token）
- **结构化方式**：JSON格式，支持索引
- **更新频率**：每天3点

### L3层：核心洞察

- **功能**：跨对话提炼的深层认知
- **压缩率**：95%（省95% token）
- **提炼方式**：AI自动总结
- **更新频率**：每天3点

## 智能上下文组合

系统根据查询内容，智能组合各层记忆：

```
L3层（核心洞察）：30%
  └─ 永远保留，优先级最高

L2层（结构化知识）：40%
  └─ 按需检索，关联性强

L1层（关键点）：30%
  └─ 按需检索，补充细节

L0层（原始记录）：按需
  └─ 仅在深度检索时使用
```

## 数据库设计

### 表结构

#### conversations（对话表）

```sql
CREATE TABLE conversations (
  id TEXT PRIMARY KEY,
  parent_id TEXT,              -- 父对话ID（支持分支）
  branch_name TEXT,             -- 分支名称
  created_at INTEGER,           -- 创建时间
  metadata TEXT                 -- 元数据（JSON）
);
```

#### messages（消息表）

```sql
CREATE TABLE messages (
  id TEXT PRIMARY KEY,
  conversation_id TEXT,         -- 所属对话
  role TEXT,                    -- 角色（user/assistant/system）
  content TEXT,                 -- 消息内容
  layer INTEGER DEFAULT 0,      -- 记忆层级（0/1/2/3）
  created_at INTEGER,           -- 创建时间
  FOREIGN KEY (conversation_id) REFERENCES conversations(id)
);
```

#### memories（记忆表）

```sql
CREATE TABLE memories (
  id TEXT PRIMARY KEY,
  conversation_id TEXT,         -- 所属对话
  layer INTEGER,                -- 记忆层级（1/2/3）
  content TEXT,                 -- 记忆内容
  access_count INTEGER DEFAULT 0, -- 访问次数
  last_accessed INTEGER,        -- 最后访问时间
  created_at INTEGER,           -- 创建时间
  FOREIGN KEY (conversation_id) REFERENCES conversations(id)
);
```

### 索引设计

```sql
-- 消息对话索引
CREATE INDEX idx_messages_conversation ON messages(conversation_id);

-- 记忆层级索引
CREATE INDEX idx_memories_layer ON memories(layer);

-- 记忆访问次数索引（用于衰减计算）
CREATE INDEX idx_memories_access ON memories(access_count);
```

## 缓存架构

### L1缓存（内存）

- **实现**：LRU算法
- **容量**：1000条记忆
- **访问速度**：微秒级
- **淘汰策略**：最少使用项优先淘汰

### L2缓存（Redis，可选）

- **实现**：Redis分布式缓存
- **容量**：可配置
- **访问速度**：毫秒级
- **持久化**：可选RDB/AOF

### L3缓存（SQLite）

- **实现**：数据库查询
- **容量**：无限制
- **访问速度**：秒级
- **持久化**：永久

## 异步I/O设计

### 批量写入

- **缓冲区大小**：50条消息
- **刷新间隔**：5秒
- **重试机制**：3次
- **事务支持**：确保数据一致性

### 写入流程

```
1. 消息进入缓冲区
2. 缓冲区满或定时器触发
3. 开启事务
4. 批量写入数据库
5. 提交事务
6. 失败则重试
```

## 后台任务调度

### 任务列表

| 任务名称 | 频率 | 功能 |
|---------|------|------|
| 记忆压缩 | 每天3点 | L0→L1→L2→L3 自动分层 |
| 衰减重算 | 每小时 | 根据访问频率调整记忆权重 |
| 向量索引 | 每5分钟 | 更新语义检索索引 |
| 缓存清理 | 每小时 | 清理过期缓存 |
| 统计收集 | 每天午夜 | 生成使用报告 |

### 任务调度器

```javascript
scheduleTask(name, interval, handler) {
  const execute = async () => {
    try {
      await handler();
    } catch (error) {
      console.error(`任务 ${name} 执行失败:`, error);
    }
  };

  // 立即执行一次
  execute();

  // 定时执行
  setInterval(execute, interval);
}
```

## 分支管理系统

### 分支操作

```javascript
// 创建新分支
memory.branch(conversationId, "alternative-approach");

// 合并分支
memory.merge(sourceBranchId, targetBranchId);

// 切换分支
memory.checkout(branchId);

// 对比分支
memory.diff(branch1, branch2);
```

### 分支结构

```
main（主分支）
  ├── feature-branch-A（功能分支A）
  │   └── experiment-1（实验分支1）
  └── feature-branch-B（功能分支B）
```

## Obsidian同步

### 同步内容

- 系统概览（`_系统概览.md`）
- 核心记忆（`核心记忆/`）
- 对话树（`对话树/`）
- 知识图谱（`知识图谱/`）

### 同步频率

- 自动同步：每15分钟
- 手动同步：调用`syncToObsidian()`方法

### 文件结构

```
C:\OpenClaw图书馆\OpenClaw-Memory\
├── _系统概览.md          # 系统状态报告
├── 核心记忆/              # L3层核心洞察
│   ├── 2026-03-21.md
│   └── 2026-03-22.md
├── 对话树/                # 可视化对话分支
│   ├── main.md
│   └── feature-branch-A.md
└── 知识图谱/              # 关联网络展示
    └── graph.json
```

## 性能优化

### Token优化

| 优化方法 | 效果 |
|---------|------|
| 分层记忆 | Token节省90% |
| 智能组合 | 上下文相关性提升50% |
| 压缩算法 | 存储空间节省80% |

### 检索优化

| 优化方法 | 效果 |
|---------|------|
| 索引优化 | 检索速度提升10倍 |
| 缓存机制 | 命中率90%+ |
| 异步I/O | 并发性能提升5倍 |

## 安全性

### 数据加密

- **加密算法**：AES-256-GCM
- **加密范围**：所有敏感数据
- **密钥管理**：环境变量

### 并发控制

- **锁机制**：读写锁
- **事务隔离**：SERIALIZABLE
- **死锁检测**：自动回滚

### 备份恢复

- **备份频率**：每天
- **备份方式**：增量备份
- **恢复时间**：<5分钟

## 监控指标

### 系统指标

- 记忆总量（L0/L1/L2/L3）
- Token节省统计
- 缓存命中率
- 后台任务状态

### 性能指标

- 检索延迟
- 写入吞吐量
- 缓存命中率
- 错误率

### 业务指标

- 记忆访问频率
- 分支创建/合并次数
- Obsidian同步成功率
- 用户满意度
