# 部署指南

## 环境要求

### 系统要求

- **操作系统**：Windows 10+, macOS 10.15+, Linux (Ubuntu 20.04+)
- **Node.js**：16.0.0 或更高版本
- **内存**：至少 2GB RAM
- **磁盘空间**：至少 500MB 可用空间

### 软件依赖

- **Node.js**：https://nodejs.org/
- **Git**：https://git-scm.com/
- **SQLite**：自动安装（通过better-sqlite3）
- **Redis**：可选（用于L2缓存）
- **Obsidian**：可选（用于知识管理可视化）

## 安装步骤

### 1. 克隆项目

```bash
git clone https://github.com/your-username/openclaw-advanced-memory.git
cd openclaw-advanced-memory
```

### 2. 安装依赖

```bash
npm install
```

### 3. 配置环境变量（可选）

创建 `.env` 文件：

```env
# 数据库路径
DB_PATH=./memory.db

# Redis配置（可选）
REDIS_HOST=localhost
REDIS_PORT=6379
REDIS_PASSWORD=

# Obsidian配置（可选）
OBSIDIAN_VAULT_PATH=C:\OpenClaw图书馆\OpenClaw-Memory
OBSIDIAN_API_PORT=27124

# 日志配置
LOG_LEVEL=info
LOG_FILE=./logs/system.log
```

### 4. 初始化数据库

```bash
npm run init
```

### 5. 启动系统

```bash
npm start
```

## 配置选项

### 数据库配置

在 `start-advanced-memory-system.js` 中配置：

```javascript
const system = new AdvancedMemorySystem({
  dbPath: './memory.db',          // 数据库路径
  cacheSize: 1000,                // L1缓存大小
  writeBufferSize: 50,            // 写入缓冲区大小
  writeInterval: 5000,            // 写入间隔（毫秒）
});
```

### 缓存配置

```javascript
await system.initCache({
  l1Size: 1000,                   // L1缓存大小
  l2Enabled: true,                // 是否启用L2缓存（Redis）
  l2Host: 'localhost',            // Redis主机
  l2Port: 6379,                   // Redis端口
});
```

### 后台任务配置

```javascript
await system.startBackgroundTasks({
  compressInterval: 24 * 60 * 60 * 1000,  // 记忆压缩间隔
  decayInterval: 60 * 60 * 1000,           // 衰减重算间隔
  cleanupInterval: 60 * 60 * 1000,         // 缓存清理间隔
});
```

### Obsidian同步配置

```javascript
await system.syncToObsidian({
  vaultPath: 'C:\\OpenClaw图书馆\\OpenClaw-Memory',
  apiPort: 27124,
  syncInterval: 15 * 60 * 1000,  // 15分钟
});
```

## 运行模式

### 开发模式

```bash
npm run dev
```

### 生产模式

```bash
npm run prod
```

### 后台运行（Linux/macOS）

```bash
nohup npm start > output.log 2>&1 &
```

### 后台运行（Windows）

使用 PM2：

```bash
npm install -g pm2
pm2 start start-advanced-memory-system.js --name openclaw-memory
pm2 save
pm2 startup
```

## 验证安装

### 1. 检查系统状态

访问 `http://localhost:3000/status`（如果启用了Web界面）

### 2. 查看日志

```bash
tail -f logs/system.log
```

### 3. 测试数据库

```bash
sqlite3 memory.db "SELECT COUNT(*) FROM conversations;"
```

### 4. 测试缓存

```bash
redis-cli ping  # 如果启用了Redis
```

## 常见问题

### Q: better-sqlite3 安装失败？

**A**: 使用预编译的二进制文件：

```bash
npm install better-sqlite3 --build-from-source
```

### Q: Redis 连接失败？

**A**: 检查Redis是否运行：

```bash
redis-cli ping
```

如果未运行，启动Redis：

```bash
# Windows
redis-server.exe

# Linux/macOS
redis-server
```

### Q: Obsidian同步失败？

**A**: 确保已安装Local REST API插件并配置端口：

1. 打开Obsidian设置
2. 安装"Local REST API"插件
3. 配置端口为27124
4. 重启Obsidian

### Q: 内存占用过高？

**A**: 调整缓存大小：

```javascript
const system = new AdvancedMemorySystem({
  cacheSize: 500,  // 减小缓存大小
});
```

## 升级指南

### 从 v0.x 升级到 v1.0

1. 备份数据库：

```bash
cp memory.db memory.db.backup
```

2. 更新代码：

```bash
git pull origin main
npm install
```

3. 运行迁移脚本：

```bash
npm run migrate
```

4. 重启系统：

```bash
npm start
```

## 卸载

### 1. 停止系统

```bash
pm2 stop openclaw-memory
pm2 delete openclaw-memory
```

### 2. 删除数据

```bash
# 删除数据库
rm memory.db

# 删除日志
rm -rf logs/

# 删除配置
rm .env
```

### 3. 卸载依赖

```bash
npm uninstall
```

## 生产环境建议

### 1. 使用进程管理器

推荐使用 PM2：

```bash
npm install -g pm2
pm2 start start-advanced-memory-system.js --name openclaw-memory
pm2 save
pm2 startup
```

### 2. 配置日志轮转

使用 logrotate：

```bash
# /etc/logrotate.d/openclaw-memory
/path/to/logs/*.log {
    daily
    rotate 7
    compress
    delaycompress
    missingok
    notifempty
}
```

### 3. 定期备份

使用 cron：

```bash
# 每天凌晨2点备份
0 2 * * * cp /path/to/memory.db /path/to/backups/memory-$(date +\%Y\%m\%d).db
```

### 4. 监控系统

使用监控工具（如Prometheus + Grafana）监控系统状态。

### 5. 安全加固

- 限制数据库文件权限
- 使用环境变量存储敏感信息
- 定期更新依赖包
- 启用HTTPS（如果暴露Web接口）

## 故障排查

### 日志位置

- 系统日志：`logs/system.log`
- 错误日志：`logs/error.log`
- 性能日志：`logs/performance.log`

### 调试模式

启用调试日志：

```bash
LOG_LEVEL=debug npm start
```

### 性能分析

使用性能分析工具：

```bash
npm run profile
```

## 获取帮助

- **文档**：查看项目Wiki
- **Issues**：提交GitHub Issue
- **讨论**：加入GitHub Discussions
- **邮件**：发送邮件至支持团队

## 许可证

MIT License
