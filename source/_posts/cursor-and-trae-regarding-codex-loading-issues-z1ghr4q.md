---
title: Cursor与Trae关于Codex loading问题
date: '2025-11-05 10:27:41'
updated: '2025-11-05 13:25:37'
excerpt: Codex插件问题处理过程
tags:
  - AI
  - Agent
categories:
  - AI实践
permalink: /post/cursor-and-trae-regarding-codex-loading-issues-z1ghr4q.html
comments: true
toc: true
---





> [!highlight]
>
> 前些天ChatGPT免费送了一个月Plus，这次问题解决的十分顺利少不了Plus的帮助，通过德区Paypal实现绑定，绑定验证大概花了一周左右的时间，听说小概率被限制账户，但是我没有，这次的问题解决完全依靠ChatGPT5精准且高效的回答。本次的问题核心其实就是类VSCode IDE（如Cursor、Trae）版本迭代升级与插件的兼容性问题，当然了也包含了WSL的问题。

‍

# 问题产生

随着Cursor与Trae的高频迭代，越来越多人遇到了Codex插件无法被加载的问题，可见[Codex not loading on latest Cursor version (1.6.27)](https://forum.cursor.com/t/codex-not-loading-on-latest-cursor-version-1-6-27/134088/12)，事实也证明了Cursor社区确实比Trae活跃，起码我在Trae简单查了一下，没有关于Codex问题的描述。上一次去社区查问题最多的还是思源笔记。

‍

![](https://cdn.jsdelivr.net/gh/luommy/myblogimg@img/nullimage-20251105105037-k8pezsm.png "无法加载Codex")

‍

# 解决动机

其实这个一开始就大概猜到了问题所在，就是配置或者环境出现了问题，但不论是哪个官方，等了一两周一直没有修复（再不修复Plus就过期了，用不了Codex了 ：）），觉得是时候自己动手解决了。

![image](https://cdn.jsdelivr.net/gh/luommy/myblogimg@img/20251105105322.png)

# 解决过程

解决的方式很简单，把当前配置、环境、报错信息全部丢给GPT5，用不了几轮对话就给你精准定位信息了。

本质上就以下几点原因：

- Codex 插件的子进程启动失败（缺少 bash 环境）：Codex 的「App Server」运行在一个 Node.js + sandbox 子环境中。  
  Windows 下如果没有正确的 shell 解释器（如 Git Bash / WSL / MSYS2），Codex 无法执行内部命令，因而报错/usr/bin/bash: not found
- Cursor（Codex）读取的是错误的 config.toml
- WLS环境异常：我使用的是Docker虚拟环境

‍

## 详细步骤

### 1️⃣ 安装 Ubuntu 发行版

在 **管理员权限 PowerShell** 里执行：

```powershell
wsl --install -d Ubuntu
```

系统会自动下载并安装 Ubuntu。  
安装完毕后会提示重启电脑，请照做。

---

### 2️⃣ 初始化 Ubuntu 环境

重启后，打开命令行：

```powershell
wsl
```

第一次运行会要求你设置用户名和密码。  
然后在 Ubuntu 终端里执行：

```bash
bash --version
```

如果能输出版本号（例如 `GNU bash, version 5.x.x`），说明 bash 已就绪。

---

### 3️⃣ 设置为默认 WSL 发行版

在 PowerShell 中执行：

```powershell
wsl --set-default Ubuntu
```

验证：

```powershell
wsl --list --verbose
```

应显示：

```
  NAME        STATE           VERSION
* Ubuntu      Running         2
  docker-desktop Stopped      2
```

星号 (\*) 代表 Ubuntu 是默认环境。

---

### 4️⃣ 测试 bash 可调用性

执行：

```powershell
C:\Windows\System32\wsl.exe bash -l -c "echo CodexOK"
```

✅ 若输出 `CodexOK`，说明 Codex 能成功调用 WSL bash。

---

### 5️⃣ 确认 config.toml 位置与内容

路径必须是：

```
C:\Users\jiangxd\.cursor\.codex\config.toml
```

内容如下（保持标准 ASCII 引号）：

```toml
model = "gpt-5-codex"
approval_policy = "on-request"
sandbox_mode = "workspace-write"

[sandbox]
shell_path = "C:\\Windows\\System32\\wsl.exe"
shell_args = ["bash", "-l", "-c"]

[projects."C:/Users/XXXX/Projects/XXXXX"]
trust_level = "trusted"

[profiles.auto]
model = "gpt-5-codex"
sandbox_mode = "workspace-write"
approval_policy = "never"

[profiles.safe]
model = "gpt-5"
sandbox_mode = "read-only"
approval_policy = "on-request"
```

---

### 6️⃣ 重启 Cursor 并加载 Code

‍

# 关于Plus的赠送

作为三年牢用户，始终没开过Plus，对于这次赠送并不感到惊讶（Gemini也送了）。事实证明，付费有付费的道理。


![17612773408324](https://cdn.jsdelivr.net/gh/luommy/myblogimg@img/null%E4%BC%81%E4%B8%9A%E5%BE%AE%E4%BF%A1%E6%88%AA%E5%9B%BE_17612773408324-20251105104005-qd0wix6.png)

‍

![17612765104262](https://cdn.jsdelivr.net/gh/luommy/myblogimg@img/null%E4%BC%81%E4%B8%9A%E5%BE%AE%E4%BF%A1%E6%88%AA%E5%9B%BE_17612765104262-20251105103658-sceloca.png)

# 参考链接

[OpenAI-Running Codex on Windows](https://developers.openai.com/codex/windows)

[Cursor社区](https://forum.cursor.com/)

[Trae社区](https://www.reddit.com/r/Trae_ai/)

‍
