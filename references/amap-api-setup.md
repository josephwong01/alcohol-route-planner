# 高德 API 配置

## 需要开通的能力

在高德开放平台创建 Web 服务应用并申请 Key，至少启用：

- 地理编码/逆地理编码
- 关键词搜索或周边搜索
- 步行路径规划

## Windows PowerShell 配置

在本机 PowerShell 中设置当前用户环境变量：

```powershell
[Environment]::SetEnvironmentVariable("AMAP_API_KEY", "你的高德Key", "User")
```

关闭并重新打开 Codex 或终端，使环境变量生效。验证时只检查是否存在，不要打印 Key：

```powershell
if ($env:AMAP_API_KEY) { "AMAP_API_KEY is configured" } else { "AMAP_API_KEY is missing" }
```

如只想临时用于当前窗口：

```powershell
$env:AMAP_API_KEY = "你的高德Key"
```

## 安全规则

- 不要把 Key 粘贴到聊天中。
- 不要把 Key 写进 `.json`、`.md`、`.py`、`.env` 或图片。
- 不要提交 `.env` 文件；若项目需要，使用 `.env.example`，内容只写变量名。
- 高德控制台中应限制 Key 的服务范围、来源 IP 或调用额度（按高德平台当前设置为准）。
- 发生泄露时立即在高德控制台禁用并重新生成 Key。
