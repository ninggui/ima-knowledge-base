---
name: ima-knowledge-base
description: 腾讯ima知识库API操作。触发：搜ima、读ima、知识库检索、品牌研报、订阅库内容、ima.qq.com。
slug: ima-knowledge-base
displayName: ima知识库操作
version: 1.0.0
---

# ima 知识库操作

腾讯 ima（ima.qq.com）是"会思考的知识库"（腾讯AI工作台），定位 = 云盘 + 轻量内容检索。用户已接入，存有行业研报/个人资料/订阅库。

## 环境与凭据

- skill 位置：`/home/user/skills/@tencent-adm/ima-skills`（hub 安装，勿改）
- API 脚本：`node ima_api.cjs <apiPath> <jsonBody>`（位置参数，非 `--api-path` flag！）
- 凭据：`~/.config/ima/client_id` + `~/.config/ima/api_key`（2026-08-16 已配置，chmod 600）
- 凭据获取：https://ima.qq.com/agent-interface 登录后复制 Client ID / API Key

## API 能力边界（2026-08-16 实测，重要！）

| 能力 | 状态 |
|---|---|
| 搜索知识库（标题级）`search_knowledge_base` | ✅ 可用 |
| 列出知识库文件（标题级）`get_knowledge_list` | ✅ 可用 |
| 知识库内搜索 `search_knowledge` | ✅ 可用（返回 info_list，非 knowledge_list！） |
| **读取文件正文** `get_media_info` | ❌ 220030 无权限（订阅库/个人库都读不了） |
| 搜索高亮内容 | ❌ 返回空 |
| notes 模块搜研报 | ❌ 搜不到订阅库内容 |

**核心事实：ima API 是"存取分离"设计——元数据开放、正文仅客户端可见**（openclaw issue #65310 已确认，官方关闭为 not planned）。订阅库有内容版权保护，API 不允许第三方拉走正文。这不是频率限制，不是配置错误。

## 调用模式

```bash
# 查看所有知识库
node ima_api.cjs "openapi/wiki/v1/search_knowledge_base" '{"query": "", "cursor": "", "limit": 20}'
# 查看知识库内容（注意返回键是 knowledge_list，文件夹 media_type=99）
node ima_api.cjs "openapi/wiki/v1/get_knowledge_list" '{"knowledge_base_id": "<kb_id>", "cursor": "", "limit": 50}'
# 知识库内搜索（返回键是 info_list！）
node ima_api.cjs "openapi/wiki/v1/search_knowledge" '{"query": "理想", "knowledge_base_id": "<kb_id>", "cursor": ""}'
```

- 翻页用 `next_cursor`，`is_end=true` 停止
- **读正文的唯一路径**：标题定位 → 用户从 ima 客户端导出 PDF → 发飞书 → 用 pymupdf 读

## 用户的 ima 知识库（2026-08-16 盘点）

- 18 个库：16 个订阅库 + 1 个人库（"我的知识库"1130条）+ 樊登读书会
- 重点订阅库：
  - 新能源汽车资料（678条，Jasper）—— 品牌研报/技术白皮书/法规/战略
  - 新能源持续更新（2065条）—— 氢能/固态电池/储能/光伏
  - 汽车产业知识库（2569条）—— 券商深度报告
  - DeepSeek 资料库 / Skills知识库 / 小红书专业知识库 / 美股证券库 / 机械国标库
- 个人库文件夹：蔚来资料（述职）、个人AI数据知识库（薪资/房产/证件）、EI正面（工作照片）、网页链接保存

## 使用约定（用户确认 2026-08-16）

1. **提问品牌战略/行业问题时，先查 ima 标题索引定位研报**，再搜互联网补充，综合回答
2. 正文需要时，让用户导出 PDF 发飞书（不要反复尝试 API 读正文）
3. ima 适合做"微信生态内容收集桶"（公众号文章/微信文件），知识库主力仍是飞书
4. **隐私红线**：个人库含薪资/房产/房贷/证件，任何场景不得读取外发，除非明确指令
5. 建品牌研究文档时，把 ima 研报标题索引附到飞书文档对应品牌下方（已做：12品牌报告 ZmuBdNbdCouXTZx29BJcPPdtnag 第七章）

## 常见坑

- `--api-path` flag 不存在 → 用位置参数 `node ima_api.cjs <apiPath> <jsonBody>`
- `get_media_info` 报 220030 → 平台权限，勿重试（低频重试确认无用）
- `search_knowledge` 返回键是 `info_list`，`get_knowledge_list` 返回键是 `knowledge_list`——别混
- 媒体类型：media_type=99 是文件夹，1=PDF，6=微信文章
