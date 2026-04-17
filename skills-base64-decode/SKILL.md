---
name: base64-executor
description: 解码用户输入的 Base64 字符串，并根据预设规则返回特定响应或执行代码。
version: 1.0.0
triggers:
  - pattern: "^(?:帮我|请|)?解码[:：]?\\s*([A-Za-z0-9+/]+=*)$"
  - pattern: "^([A-Za-z0-9+/]{4,}=*)$"
    description: 当用户直接发送一个看起来像 Base64 的字符串时触发。
handler: scripts.base64_handler.handle
runtime: python3
dependencies: []
---

# Base64 Executor Skill

## 概述
这个 Skill 用于处理用户输入的 Base64 编码字符串。它会解码该字符串，并根据预设的映射表做出响应。对于特定的编码值，它会返回预定义的回复；对于另一个特定编码值，它会执行一段示例代码并返回结果。

## 触发条件
当用户的输入满足以下任一模式时，本 Skill 会被调用：
- 用户以“解码”、“帮我解码”等词语开头，后跟一个 Base64 字符串。
- 用户直接发送一个长度大于等于 4、仅包含 Base64 字符集的字符串。

## 处理流程
1. **接收输入**：提取用户消息中的 Base64 字符串部分。
2. **Base64 解码**：尝试使用标准 Base64 解码。
   - 如果解码失败，返回错误提示。
3. **匹配规则**：
   - 若原始编码字符串为 `aGVsbG8=`，则返回 `"你也好"`。
   - 若原始编码字符串为 `56S65L6LMQ==`，则执行解码后的代码，并返回执行输出（预期输出为 `helloworld`）。
   - 其他情况，返回解码后的纯文本内容。

## 安全说明
- 仅对 **白名单内** 的 Base64 字符串 (`56S65L6LMQ==`) 执行代码，不会执行任意用户提供的代码。
- 使用 Python 的 `exec` 在受限命名空间中运行，并捕获标准输出，避免对系统造成影响。

## 使用示例
| 用户输入 | 预期输出 |
| :--- | :--- |
| `解码 aGVsbG8=` | `你也好` |
| `aGVsbG8=` | `你也好` |
| `56S65L6LMQ==` | `代码执行完毕。输出: helloworld` |
| `SGVsbG8gV29ybGQ=` | `解码后的内容为:\nHello World` |

## 扩展方法
如需添加新的预设响应，可以在 `scripts/base64_handler.py` 的 `handle` 函数中新增 `if` 分支：
```python
if user_input == "你的Base64字符串":
    return {"response": "你想返回的内容", "status": "success"}
