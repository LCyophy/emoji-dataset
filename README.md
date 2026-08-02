# Emoji Dataset

这是一个面向搜索与聊天应用的公开表情包数据仓库，包含经过尺寸规范化和压缩的图片，以及统一的 OCR、视觉理解、来源与分类索引。处理工具源码见 [universal-emoji-compressor](https://github.com/LCyophy/universal-emoji-compressor)。

## 数据概览

- 数据版本：schema v1
- 图片数量：11,396
- 集合数量：6
- 入口索引：[\`index.json\`](index.json)
- 集合统计：[\`catalog.json\`](catalog.json)

| collection | 来源 | 图片数 | 大小 |
|---|---|---:|---:|
| \`acfun-emoji\` | [AcFun-Emoji](https://github.com/ikutarian/AcFun-Emoji) | 349 | 8.4 MiB |
| \`chinese-bqb\` | [ChineseBQB](https://github.com/zhaoolee/ChineseBQB) | 5,840 | 212.6 MiB |
| \`doro\` | [doro](https://github.com/1143520/doro) | 3,276 | 405.3 MiB |
| \`nailong\` | [nailong-meme-generator](https://github.com/lewischn/nailong-meme-generator) | 80 | 0.2 MiB |
| \`phoebe-hub\` | [Phoebe-Hub](https://github.com/Kato-Shoko705/Phoebe-Hub) | 755 | 84.1 MiB |
| \`opossum-work-memes\` | — | 1,096 | 2.4 MiB |

## 使用索引

\`index.json\` 的 \`data\` 数组包含每张图片。应用可以在 \`search_text\` 中做简单全文匹配，再读取记录中的 \`url\`。长期集成建议保存相对 \`path\`，以便仓库迁移时自行拼接 Raw URL。

\`\`\`python
import json
from urllib.request import urlopen

url = "https://raw.githubusercontent.com/LCyophy/emoji-dataset/main/index.json"
index = json.load(urlopen(url))
results = [
    row for row in index["data"]
    if all(word.lower() in row["search_text"].lower() for word in ["doro", "激动"])
]
for row in results[:10]:
    print(row["id"], row["url"])
\`\`\`

主要字段：

- \`collection\`：稳定的来源集合标识。
- \`path\` / \`url\`：仓库相对路径与便利 Raw URL。
- \`original_path\` / \`original_paths\`：有来源记录的集合会保留相对来源路径；未公开来源的集合省略。
- \`category_path\`：原始路径父目录数组。
- \`ocr_text\`：以简体中文为主的原图 OCR 索引。
- \`description\`、\`emotion\`、\`objects\`、\`tags\`：Qwen3-VL-4B-Instruct 本地生成的视觉索引。
- \`search_text\`：由文件名、目录、集合、OCR 与视觉语义合并的搜索文本。
- \`content_sha256\`：公开压缩文件的 SHA-256。

## 数据处理

图片使用 OCR-first 尺寸算法：原图 OCR 用于索引，最终编码后的 160px OCR 作为物理可读性基准；无文字时使用复杂度阈值 0.236 / 0.373 / 0.456 选择 64 / 96 / 128 / 160。GIF 保留 loop 与总播放时长，损坏但可恢复的帧会跳过并转移时长。

本仓库只发布最终消费数据，不包含 SQLite、内部 JSONL、OCR 检测框、复杂度明细、模型耗时、日志、错误堆栈或模型文件。视觉理解可能存在识别错误或标签偏差，请勿将其作为事实判断依据。

## 来源与权利说明

素材来自以下公开仓库：

- [ikutarian/AcFun-Emoji](https://github.com/ikutarian/AcFun-Emoji)
- [zhaoolee/ChineseBQB](https://github.com/zhaoolee/ChineseBQB)
- [1143520/doro](https://github.com/1143520/doro)
- [lewischn/nailong-meme-generator](https://github.com/lewischn/nailong-meme-generator)
- [Kato-Shoko705/Phoebe-Hub](https://github.com/Kato-Shoko705/Phoebe-Hub)

\`opossum-work-memes\` 是不公开外部来源信息的负鼠主题集合，因此索引不包含来源仓库、来源网址、源文件路径或源内容哈希。

本仓库不对图片素材重新授权。图片、角色、商标及其他内容的权利归原作者或相应权利人；请根据来源仓库说明及使用场景确认许可。权利人可通过 GitHub Issue 请求更正来源或移除内容。

## 更新方式

本次数据由统一算法和 Qwen3-VL-4B-Instruct 完整重建，并以单一干净历史发布。后续应用应始终以索引中的 \`path\` 为权威定位字段。
