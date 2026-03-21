# 贡献指南

感谢您考虑为 OpenClaw Advanced Memory System 贡献！

## 如何贡献

### 报告问题

如果您发现了bug或有功能建议：

1. 检查是否已有相似的Issue
2. 创建新Issue，详细描述问题或建议
3. 提供重现步骤（如果是bug）
4. 附上相关日志和系统信息

### 提交代码

1. Fork 项目
2. 创建功能分支 (`git checkout -b feature/AmazingFeature`)
3. 提交更改 (`git commit -m 'Add some AmazingFeature'`)
4. 推送到分支 (`git push origin feature/AmazingFeature`)
5. 开启 Pull Request

### 代码规范

- 使用 ESLint 进行代码检查
- 遵循现有代码风格
- 添加必要的注释
- 更新相关文档

### 测试

- 为新功能添加测试
- 确保所有测试通过
- 测试不同环境（Windows/macOS/Linux）

## 开发环境设置

### 安装开发依赖

```bash
npm install --save-dev
```

### 运行测试

```bash
npm test
```

### 代码检查

```bash
npm run lint
```

### 格式化代码

```bash
npm run format
```

## Pull Request 流程

### PR 标题格式

- `feat:` 新功能
- `fix:` 修复bug
- `docs:` 文档更新
- `style:` 代码格式调整
- `refactor:` 代码重构
- `test:` 测试相关
- `chore:` 构建/工具相关

### PR 描述模板

```markdown
## 变更类型
- [ ] Bug修复
- [ ] 新功能
- [ ] 代码重构
- [ ] 文档更新
- [ ] 性能优化

## 变更说明
简要描述本次PR的内容

## 相关Issue
关闭 #issue_number

## 测试
描述测试过程和结果

## 截图（如适用）
上传相关截图

## 检查清单
- [ ] 代码遵循项目规范
- [ ] 已添加测试
- [ ] 测试通过
- [ ] 文档已更新
```

## 社区规范

- 尊重所有贡献者
- 建设性讨论
- 接受反馈并改进
- 帮助新贡献者

## 获取帮助

如果您有任何问题：

- 查看 [文档](ARCHITECTURE.md)
- 搜索已有的 [Issues](https://github.com/your-username/openclaw-advanced-memory/issues)
- 在 [Discussions](https://github.com/your-username/openclaw-advanced-memory/discussions) 中提问

## 许可证

通过贡献代码，您同意您的贡献将使用 MIT 许可证。
