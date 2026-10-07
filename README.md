<div align="center">

# 中国行政诉讼案例数据集

**635 个行政案件 · 425 篇裁判文书 · 417 篇全文**

从《行政诉讼法》（第三版，何海波）书末《案例索引》整理，并逐案在北大法宝检索到对应裁判文书全文。
Web-scale Chinese administrative litigation case corpus for legal NLP and empirical legal research.

[![License: CC BY-NC 4.0](https://img.shields.io/badge/License-CC%20BY--NC%204.0-lightgrey.svg)](https://creativecommons.org/licenses/by-nc/4.0/)
[![Cases](https://img.shields.io/badge/cases-635-blue.svg)](#数据概览)
[![Judgments](https://img.shields.io/badge/judgments-425-brightgreen.svg)](#数据概览)
[![Full text](https://img.shields.io/badge/full--text-417-orange.svg)](#数据概览)
[![Format](https://img.shields.io/badge/format-JSONL%20%7C%20Markdown-informational.svg)](#文件说明)
[![Language](https://img.shields.io/badge/language-%E4%B8%AD%E6%96%87-red.svg)](#)

</div>

---

## 目录

- [这是什么](#这是什么)
- [数据概览](#数据概览)
- [文件说明](#文件说明)
- [快速开始](#快速开始)
- [字段说明](#字段说明)
- [典型用途](#典型用途)
- [数据来源与生成方法](#数据来源与生成方法)
- [匹配规则](#匹配规则)
- [已知局限](#已知局限)
- [引用](#引用)
- [许可与免责声明](#许可与免责声明)

## 这是什么

一个**中国行政诉讼（民告官）案例语料库**。书中《案例索引》是作者精选的行政审判代表性案例，本仓库把这些案件的**书内出处**与**裁判文书原文**对应起来，可直接用于：

- 法律检索与类案研究
- 计算法学 / LegalNLP 语料（判决书全文、结构化元数据）
- 行政审判实证研究（年份、法院层级、文书类型、参照级别均可统计）

**为什么值得用**：中文裁判文书数据集不少，但**按权威教材索引组织、且带原书页码定位**的很少。你可以用 `book_pages` 直接跳回书中的分析，形成"教材论述 ↔ 原始判决"的双向检索。

## 数据概览

| 分类（依原书） | 条目数 | 命中裁判文书 | 命中率 |
| --- | ---: | ---: | ---: |
| （一）中国法院诉讼案例 | 614 | 417 | 68% |
| （二）非诉执行案件 | 8 | 6 | 75% |
| （三）清末民国案例 | 4 | 0 | 0% |
| （四）外国法院与国际机构案例 | 9 | 2 | 22% |
| **合计** | **635** | **425** | **67%** |

其中 **417 篇**已取得判决书全文（最短 452 字，中位 4,444 字，最长 23,134 字）。

**文书质量分布**：

| 参照级别 | 篇数 | | 文书类型 | 篇数 |
| --- | ---: | --- | --- | ---: |
| 普通案例 | 185 | | 二审行政判决书 | 165 |
| 经典案例 | 154 | | 一审行政判决书 | 73 |
| 公报案例 | 40 | | 再审行政裁定书 | 59 |
| 评析案例 | 16 | | 二审行政裁定书 | 30 |
| 法宝推荐 | 13 | | 一审行政裁定书 | 28 |
| 典型案例 | 8 | | 再审行政判决书 | 19 |
| 指导性案例 | 4 | | 执行行政裁定书 | 9 |
| 优秀案例 | 3 | | 其他 | 42 |

裁判年份跨度 **1986–2026**。

## 文件说明

| 文件 | 说明 |
| --- | --- |
| `data/judgments.jsonl` | **判决书全文与元数据**，一行一个案件（JSON Lines，约 6.4 MB） |
| `cases/` | 635 个独立 Markdown（与原书索引同序，编号一一对应），可直接在网页阅读 |
| `data/pkulaw_results.json` | 全 635 案的检索元数据：命中数、最佳匹配、匹配分、是否取得全文、被否决的候选 |
| `cases.md` | 书末《案例索引》全文，按原书分类与字母顺序 |
| `data/cases.csv` / `data/cases.json` | 案件索引结构化数据 |
| `raw/case_index_ocr.txt` | 原书索引页 OCR 原文（供核对） |

## 快速开始

**Python：**

```python
import json

with open("data/judgments.jsonl", encoding="utf-8") as f:
    cases = [json.loads(line) for line in f]

hits = [c for c in cases if c["fulltext"]]
print(len(cases), "个案件，", len(hits), "篇有全文")

# 按关键词检索判决书正文
def search(kw):
    return [c for c in hits if kw in c["fulltext"]]

for c in search("比例原则")[:3]:
    print(c["case_name"], "->", c["matched_title"])
```

**命令行（jq）—— 导出为纯文本：**

```bash
jq -r 'select(.fulltext != "") | "### \(.case_name)\n\(.fulltext)\n"' data/judgments.jsonl > all.txt
```

**统计各类文书数量：**

```bash
jq -r 'select(.doc_type != "") | .doc_type' data/judgments.jsonl | grep -oE '(一审|二审|再审)?行政(判决书|裁定书)' | sort | uniq -c | sort -rn
```

## 字段说明

`data/judgments.jsonl` 每行的字段：

| 字段 | 类型 | 含义 |
| --- | --- | --- |
| `case_name` | string | 书中案件名（索引缩略名） |
| `category` | string | 原书分类 |
| `book_pages` | string | 该案在原书**正文**中出现的页码（可据此回查书中分析） |
| `matched_title` | string | 在北大法宝匹配到的裁判文书正式标题 |
| `pkulaw_url` | string | 该文书在北大法宝的链接 |
| `doc_type` | string | 文书类型、法院、案号、审结日期 |
| `match_score` | float | 名称匹配置信度（0–1） |
| `fulltext` | string | 判决书全文（未取到者为空串） |

## 典型用途

**1. 教材 ↔ 判决双向检索** —— 用 `book_pages` 从书中论述找到原始判决，或反向定位。

**2. 构建 LegalNLP 语料** —— 417 篇带完整元数据的行政判决书，可按年份/法院/文书类型切分。

**3. 类案与裁判规则研究** —— `matched_title` 中的"经典案例""公报案例""指导性案例"标注可直接用作高质量样本筛选。

**4. 行政审判实证分析** —— 案由、地域、审级、年份分布均可统计。

## 数据来源与生成方法

**1. 案件索引** —— 原书《案例索引》位于原书第 736–756 页（PDF 第 763–783 页，二者相差 27 页）。该书为**扫描版 PDF、无文本层**（仅有一层引流水印文字），使用 macOS Vision 框架（`zh-Hans`，Accurate 模式）OCR，并依据识别框坐标还原多栏与换行排版。

**2. 判决书全文** —— 通过本机 Chrome（CDP 协议自动化）访问北大法宝司法案例库检索抓取。全程限速、多标签页并行、单账号常态访问。

## 匹配规则

书中案件名是**索引缩略名**（如"苏州爱利电器公司诉苏州市工商局沧浪分局行政处罚案"），与裁判文书正式标题常有差异。为在**召回**与**准确**之间取舍，采用如下规则：

1. 使用**标题检索**（而非全文检索）
2. **原告名完整出现且后接边界字**（诉/与/不服/状告/等/申请…），**或**与标题的最长公共子串 ≥4 字且不含通用词
3. 结果**必须是行政案件**（排除同名当事人的民事案件）
4. 文书详情页**必须从检索结果点击进入**（直接跳转 URL 会被拦截返回空壳）

整体策略是**宁可漏、不可错配**——67% 的命中率是这一取舍的结果。

## 已知局限

- 210 个案件未匹配到裁判文书，多为年代较早、未公开或名称差异过大者。`data/pkulaw_results.json` 中保留了被否决的候选（`rejected_match`），可供人工复核。
- 少数结果可能仍为同名当事人的其他案件；`match_score`、`matched_title`、`doc_type` 可供核验。
- OCR 可能存在个别形近字误差。
- `book_pages` 是**原书正文页码**，不是 PDF 页码。
- 数据采集于 2026 年 10 月，未跟踪后续文书更新。

## 引用

如果本数据集对你的研究有帮助，欢迎引用：

```bibtex
@misc{xzss_case_index,
  title  = {中国行政诉讼案例数据集：《行政诉讼法》(何海波 第三版) 案件索引与裁判文书全文},
  author = {thu-lawyer},
  year   = {2026},
  url    = {https://github.com/thu-lawyer/xingzheng-susongfa-case-index}
}
```

原始索引来自：何海波《行政诉讼法》（第三版）。

## 许可与免责声明

- 本仓库的**整理成果**（案件索引、结构化元数据）采用 [CC BY-NC 4.0](https://creativecommons.org/licenses/by-nc/4.0/) 许可，限非商业研究与学习使用。
- 其中**纯索引与结构化元数据字段**（案名、分类、原书页码、案号、法院、审结日期、文书类型、检索元数据）另以 **[CC0 1.0](https://creativecommons.org/publicdomain/zero/1.0/)** 释放，见 [`LICENSE-METADATA`](LICENSE-METADATA)——可无摩擦地并入任何语料库或数据库，无需署名。**如果你只想复用索引结构，用这一层。**
- **裁判文书本身**依《中华人民共和国著作权法》第五条（具有立法、行政、司法性质的文件）不属于著作权保护对象；但本仓库的裁判文书文本采集自北大法宝，其服务条款可能限制再分发，**商业使用请自行取得授权**。
- 案件名称、裁判文书等权利归原权利人所有。如涉版权问题请联系删除。
- 本仓库为个人学习研究用途的数据整理，不构成任何法律意见。
