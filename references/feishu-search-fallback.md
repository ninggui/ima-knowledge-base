# ima 找不到 → 飞书 drive +search 全文搜索回退（2026-08-26 实测）

用户说"在ima里找一个文档"但 ima 标题搜索/知识库内搜索均空时，**不一定是没有**——先切飞书全文搜索。

## 触发场景

- 用户说"你在ima里面找一个文档内容里面包含[某句话]"（如"做好自己的事情，不给别人添麻烦"）
- ima 侧可能全空：`search_knowledge_base`（标题级）→ `search_knowledge`（库内）→ 遍历个人库文件夹全部无命中
- 记忆结论：ima 的正文/高亮搜索都不可用，只有标题级，正文片段搜不到是正常现象

## 正确路径

```bash
# 1. 飞书 drive +search 支持正文级全文检索（含 PPTX/PDF 内文）
lark-cli drive +search --query "不给别人添麻烦" --as user --format json
#   结果在 data.results[]，命中片段在 summary_highlighted（<h> 标记）

# 2. 搜索词用独特短语片段（5-10字），不要整句；同一文档多副本会出多条，用 result_meta.url 去重

# 3. 命中的文件类型（docx/wiki/file/...）：
#    - docx → lark-cli docs +fetch --api-version v2 --doc TOKEN --as bot
#    - file（pptx/docx 等）→ lark-cli drive +download --file-token TOKEN --output x.pptx --as user
#    - wiki → 先 wiki +node-get 拿 obj_token 再对应处理

# 4. 解析 PPTX：uv venv + pip install python-pptx（系统 python3 无该模块），
#    提取脚本走 write_file 落盘 .py 再执行（勿 python3 -c）
```

## 关键事实

- **ima API 正文级搜索能力缺失是设计使然**（标题级可用、高亮/正文不可用），别花时间重试。
- 用户记忆："个人说明书/工作准则"类文档可能在飞书而非 ima——ima 找不到时先问/先搜飞书，比反复翻 ima 文件夹高效。
- drive +search 对刚创建文档有索引延迟（几分钟），新文档搜不到是正常的，用已知 token 直取。