<img src="./assets/cover.png" alt="ima 知识库" width="100%">

<div align="center">

# ima 知识库操作

**腾讯 ima API 是"存取分离"的——能搜标题、列文件，但读不了正文，别白折腾。**

![Status](https://img.shields.io/badge/status-production-green)
![Platform](https://img.shields.io/badge/platform-腾讯ima-blue)
![Boundary](https://img.shields.io/badge/正文读取-220030无权限-red)
![Libs](https://img.shields.io/badge/已盘点-18个库-orange)
![License](https://img.shields.io/badge/license-MIT-blue)

[它解决什么问题](#它解决什么问题) - [为什么比手动强](#为什么比手动强) - [工作流](#工作流) - [实测参数](#实测参数) - [快速开始](#快速开始)

</div>

---

## 它解决什么问题

我的 ima 里存了 18 个库（新能源研报、车企产业、DeepSeek 资料、小红书专业库等），用户问品牌战略/行业问题时想先从 ima 标题索引定位研报，再搜互联网补充。但 ima API 有个硬骨头：能搜、能列文件，正文却读不出来（订阅库有版权保护）。不知道这个边界就会反复尝试读正文、浪费大量时间还读不到。

## 为什么比手动强

| 自己瞎试 | 本仓库 |
|---|---|
| 反复调 API 想读正文 | 明确：正文读不了（220030 无权限），别浪费时间 |
| 以为返回键是 knowledge_list | 库内搜索返回的是 info_list（不是 knowledge_list） |
| 用 --api-path flag 传参 | 位置参数：`node ima_api.cjs <path> <jsonBody>` |
| 翻页不知道何时停 | 用 next_cursor，is_end=true 停止 |
| 一定要正文时硬磕 API | 唯一路径：标题定位→用户导出 PDF→发飞书→pymupdf 读 |

## 工作流

```
提问品牌/行业问题
   ↓
先查 ima 标题索引（search_knowledge_base / get_knowledge_list）定位研报
   ↓
需要正文 → 让用户从 ima 客户端导出 PDF → 发飞书
   ↓
pymupdf 读 PDF，再搜互联网补充，综合回答
```

## 实测参数

- **能力边界（2026-08-16 实测）**：知识库搜索 ✅、列出文件 ✅、库内搜索 ✅（返回 info_list）；读正文 ❌ 220030、搜索高亮 ❌、notes 搜订阅库 ❌
- **核心事实**：ima API"存取分离"——元数据开放、正文仅客户端可见（官方已关闭为 not planned）
- **库盘点**：18 个库（16 订阅库 + 1 个人库 + 樊登读书会）；个人库"我的知识库"1130 条
- **凭据**：`~/.config/ima/client_id` + `api_key`，chmod 600
- **参考**：`references/feishu-search-fallback.md`

## 快速开始

```bash
# 查看所有知识库
node ima_api.cjs "openapi/wiki/v1/search_knowledge_base" '{"query": "", "cursor": "", "limit": 20}'

# 知识库内搜索（返回键是 info_list！）
node ima_api.cjs "openapi/wiki/v1/search_knowledge" '{"query": "理想", "knowledge_base_id": "<kb_id>", "cursor": ""}'
```

凭据在 https://ima.qq.com/agent-interface 登录后复制 Client ID / API Key。

## License

MIT
