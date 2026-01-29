---
summary: "使用 Amazon Bedrock (Converse API) 模型"
read_when:
  - 您想使用 Amazon Bedrock 模型
  - 您需要为模型调用设置 AWS 凭证/区域
---
# Amazon Bedrock

Moltbot 可以通过 pi-ai 的 **Bedrock Converse** 流式提供商使用 **Amazon Bedrock** 模型。Bedrock 认证使用 **AWS SDK 默认凭证链**，而不是 API 密钥。

## pi-ai 支持的内容

- 提供商：`amazon-bedrock`
- API：`bedrock-converse-stream`
- 认证：AWS 凭证（环境变量、共享配置或实例角色）
- 区域：`AWS_REGION` 或 `AWS_DEFAULT_REGION`（默认：`us-east-1`）

## 自动模型发现

如果检测到 AWS 凭证，Moltbot 可以自动发现支持**流式传输**和**文本输出**的 Bedrock 模型。发现过程使用 `bedrock:ListFoundationModels` 并会被缓存（默认：1 小时）。

配置选项位于 `models.bedrockDiscovery` 下：

```json5
{
  models: {
    bedrockDiscovery: {
      enabled: true,
      region: "us-east-1",
      providerFilter: ["anthropic", "amazon"],
      refreshInterval: 3600,
      defaultContextWindow: 32000,
      defaultMaxTokens: 4096
    }
  }
}
```

注意：
- 当存在 AWS 凭证时，`enabled` 默认为 `true`。
- `region` 默认为 `AWS_REGION` 或 `AWS_DEFAULT_REGION`，然后是 `us-east-1`。
- `providerFilter` 匹配 Bedrock 提供商名称（例如 `anthropic`）。
- `refreshInterval` 是秒数；设置为 `0` 可禁用缓存。
- `defaultContextWindow`（默认：`32000`）和 `defaultMaxTokens`（默认：`4096`）
  用于发现的模型（如果您知道模型限制，可以覆盖）。

## 设置（手动）

1) 确保 AWS 凭证在 **网关主机** 上可用：

```bash
export AWS_ACCESS_KEY_ID="AKIA..."
export AWS_SECRET_ACCESS_KEY="..."
export AWS_REGION="us-east-1"
# 可选：
export AWS_SESSION_TOKEN="..."
export AWS_PROFILE="your-profile"
# 可选（Bedrock API 密钥/不记名令牌）：
export AWS_BEARER_TOKEN_BEDROCK="..."
```

2) 在配置中添加 Bedrock 提供商和模型（不需要 `apiKey`）：

```json5
{
  models: {
    providers: {
      "amazon-bedrock": {
        baseUrl: "https://bedrock-runtime.us-east-1.amazonaws.com",
        api: "bedrock-converse-stream",
        auth: "aws-sdk",
        models: [
          {
            id: "anthropic.claude-opus-4-5-20251101-v1:0",
            name: "Claude Opus 4.5 (Bedrock)",
            reasoning: true,
            input: ["text", "image"],
            cost: { input: 0, output: 0, cacheRead: 0, cacheWrite: 0 },
            contextWindow: 200000,
            maxTokens: 8192
          }
        ]
      }
    }
  },
  agents: {
    defaults: {
      model: { primary: "amazon-bedrock/anthropic.claude-opus-4-5-20251101-v1:0" }
    }
  }
}
```

## EC2 实例角色

当在附加了 IAM 角色的 EC2 实例上运行 Moltbot 时，AWS SDK
将自动使用实例元数据服务 (IMDS) 进行认证。
但是，Moltbot 的凭证检测目前只检查环境变量，而不是 IMDS 凭证。

**解决方法：** 设置 `AWS_PROFILE=default` 以表示 AWS 凭证可用。
实际认证仍通过 IMDS 使用实例角色。

```bash
# 添加到 ~/.bashrc 或您的 shell 配置文件
export AWS_PROFILE=default
export AWS_REGION=us-east-1
```

EC2 实例角色**所需的 IAM 权限**：
- `bedrock:InvokeModel`
- `bedrock:InvokeModelWithResponseStream`
- `bedrock:ListFoundationModels`（用于自动发现）

或附加托管策略 `AmazonBedrockFullAccess`。

**快速设置：**

```bash
# 1. 创建 IAM 角色和实例配置文件
aws iam create-role --role-name EC2-Bedrock-Access \
  --assume-role-policy-document '{
    "Version": "2012-10-17",
    "Statement": [{
      "Effect": "Allow",
      "Principal": {"Service": "ec2.amazonaws.com"},
      "Action": "sts:AssumeRole"
    }]
  }'

aws iam attach-role-policy --role-name EC2-Bedrock-Access \
  --policy-arn arn:aws:iam::aws:policy/AmazonBedrockFullAccess

aws iam create-instance-profile --instance-profile-name EC2-Bedrock-Access
aws iam add-role-to-instance-profile \
  --instance-profile-name EC2-Bedrock-Access \
  --role-name EC2-Bedrock-Access

# 2. 附加到您的 EC2 实例
aws ec2 associate-iam-instance-profile \
  --instance-id i-xxxxx \
  --iam-instance-profile Name=EC2-Bedrock-Access

# 3. 在 EC2 实例上，启用发现
moltbot config set models.bedrockDiscovery.enabled true
moltbot config set models.bedrockDiscovery.region us-east-1

# 4. 设置解决方法环境变量
echo 'export AWS_PROFILE=default' >> ~/.bashrc
echo 'export AWS_REGION=us-east-1' >> ~/.bashrc
source ~/.bashrc

# 5. 验证模型已发现
moltbot models list
```

## 注意事项

- Bedrock 要求在您的 AWS 账户/区域中启用**模型访问权限**。
- 自动发现需要 `bedrock:ListFoundationModels` 权限。
- 如果您使用配置文件，请在网关主机上设置 `AWS_PROFILE`。
- Moltbot 按以下顺序提供凭证来源：`AWS_BEARER_TOKEN_BEDROCK`，
  然后 `AWS_ACCESS_KEY_ID` + `AWS_SECRET_ACCESS_KEY`，然后 `AWS_PROFILE`，然后是
  默认 AWS SDK 链。
- 推理支持取决于模型；查看 Bedrock 模型卡片了解当前功能。
- 如果您更喜欢托管密钥流程，也可以在 Bedrock 前面放置一个 OpenAI 兼容
  代理，并将其配置为 OpenAI 提供商。
