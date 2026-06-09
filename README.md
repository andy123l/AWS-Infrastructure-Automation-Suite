# AWS Prompt the Planet Challenge - 提交指南

## 比赛信息

- **名称**: AWS Prompt the Planet Challenge
- **奖金**: $50,000 AWS Activate Credits (10位获奖者各$5,000)
- **截止**: 2026年6月10日 23:59
- **提交链接**: https://d1hhwkzyo8r24l.cloudfront.net/

## 评判标准

| 标准 | 权重 | 说明 |
|------|------|------|
| Clear & Actionable | 高 | 具体指令，每步清晰 |
| Production-Ready | 高 | 内置安全、监控、成本控制 |
| Well-Documented | 高 | 前提条件、示例、排查指南 |
| AWS Best Practices | 高 | 对齐 Well-Architected Framework |

## 我们的5个提交

| # | 文件名 | 主题 | 差异化 |
|---|--------|------|--------|
| 1 | `01-multi-account-security-baseline.md` | 多账户安全基线 | 现有示例偏应用层，安全架构方向空白 |
| 2 | `02-cost-optimization-automation.md` | 成本优化自动化 | 直接省钱，每个团队都需要 |
| 3 | `03-compliance-audit-automation.md` | 合规审计自动化 | 初创公司SOC2/HIPAA刚需 |
| 4 | `04-zero-downtime-blue-green.md` | 零停机蓝绿部署 | 每个团队都会遇到的场景 |
| 5 | `05-serverless-fullstack-scaffold.md` | Serverless全栈脚手架 | 覆盖面广，实用价值高 |

## 提交步骤

### 1. 打开提交链接
https://d1hhwkzyo8r24l.cloudfront.net/

### 2. 填写表单

对于每个提交：

1. **Prompt Title**: 复制文件中的 `## Prompt Title` 内容
2. **Category**: 选择 `Code Development`
3. **The Prompt**: 复制 `## The Prompt` 代码块中的完整内容（不包含 ``` 标记）
4. **Description & Use Case**: 复制 `## Description & Use Case` 部分的内容
5. **Example Output** (可选): 复制 `## Example Output` 部分的内容
6. **Recommended AI Model(s)**: 复制 `## Recommended AI Model(s)` 内容
7. **Tags**: 根据文件中的 `## Tags` 勾选对应标签
8. **Additional Notes** (可选): 复制 `## Additional Notes` 部分的内容
9. **勾选免责声明**: 阅读并同意
10. **点击 Submit**

### 3. 重复提交

- 比赛鼓励多次投稿，每个prompt独立提交
- 没有提交数量限制
- 每个prompt独立评审

## 质量检查清单

提交前检查：

- [ ] Prompt 是完整的、可复制粘贴的（不是描述性文字）
- [ ] 包含具体的参数值（不是占位符如 `{your-value}`）
- [ ] 包含前提条件（AWS账户、IAM权限、工具要求）
- [ ] 包含预期结果（运行后会得到什么）
- [ ] 包含排查建议（常见问题和解决方案）
- [ ] 对齐 AWS Well-Architected Framework
- [ ] 包含具体的AWS服务列表
- [ ] 没有包含敏感信息（API密钥、密码等）

## 提示词质量参考

### 好的示例 (✅)
```
Create a production-ready EKS cluster with the following specifications:
Use Karpenter for node provisioning with Spot instance support,
implement KEDA for workload-based autoscaling, configure AWS Load
Balancer Controller for ingress...
```

### 坏的示例 (❌)
```
A prompt that helps set up Kubernetes clusters
```

## AWS Bedrock Prompt Engineering 最佳实践

我们的prompt遵循以下最佳实践：

1. **结构化**: Role → Task → Context → Constraints → Output Format
2. **具体值**: 使用具体参数值，不用占位符
3. **Few-shot**: 包含示例输入输出
4. **明确约束**: 列出技术要求和限制条件
5. **输出格式**: 明确指定生成代码的结构

## 参考资源

- [AWS Well-Architected Framework](https://aws.amazon.com/architecture/well-architected/)
- [Amazon Bedrock Documentation](https://docs.aws.amazon.com/bedrock/)
- [AWS Startups Prompt Library](https://aws.amazon.com/startups/prompt-library/)
- [Prompt Engineering Best Practices](https://docs.aws.amazon.com/bedrock/latest/userguide/prompt-engineering-guidelines.html)

## 时间规划

| 日期 | 任务 |
|------|------|
| 6月8日 | 完成所有5个prompt的编写 |
| 6月9日 | 测试prompt、优化内容、提交 |
| 6月10日 | 最终检查、补充提交 |

## 注意事项

1. **不要包含敏感信息**: 无API密钥、密码、内部域名
2. **使用公开AWS服务**: 不依赖私有API或内部服务
3. **原创性**: 所有内容必须原创
4. **语言**: 使用英文提交（面向全球开发者）
5. **格式**: 保持格式清晰，便于阅读
