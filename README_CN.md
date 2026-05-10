# paper-fetch — 按 DOI 自动下载论文 PDF 📄

[English](README.md) | [Unpaywall](https://unpaywall.org) | [Semantic Scholar](https://www.semanticscholar.org) | [bioRxiv API](https://api.biorxiv.org)

## 功能简介

**DOI（或标题）→ PDF**
- 7 源回退链：[Unpaywall](https://unpaywall.org) → [Semantic Scholar](https://www.semanticscholar.org) → [arXiv](https://arxiv.org) → [PubMed Central](https://pmc.ncbi.nlm.nih.gov) → [bioRxiv](https://www.biorxiv.org)/[medRxiv](https://www.medrxiv.org) → 出版商直链 *（机构模式启用）* → [Sci-Hub](https://www.sci-hub.pub) 镜像 *（默认开启的兜底）*
- 仅有标题时通过 `--title` 解析 — Crossref + Semantic Scholar 双路回退，附置信度标记
- 自动命名：`{第一作者}_{年份}_{期刊简称}_{简短标题}.pdf`

**批量 + Agent 友好**
- `--batch dois.txt` 或 `--batch -`（stdin）批量下载
- `--idempotency-key` 重试时直接复用原信封，无任何网络 I/O
- `--stream` 在每个 DOI 解析完成时立即输出一行 NDJSON
- 已下载文件默认跳过；`--overwrite` 强制覆盖

**内置正确性保障**
- stdout 稳定 JSON 信封，stderr NDJSON 进度，机器可读的 `schema` 子命令
- `--format` 自动识别 TTY,退出码分类（`0`/`1`/`3`/`4`)便于 orchestrator 路由
- 每次外部抓取都执行 SSRF 防护 + `%PDF` 魔数校验 + 50 MB 体积上限
- 零运行时依赖 — 纯 Python 标准库

兼容 Claude Code、Codex、Hermes、OpenClaw、ClawHub、pi-mono、SkillsMP — 所有支持 [Agent Skills](https://agentskills.io) 格式的 Agent。

## 学科覆盖

本 skill **学科无关**,不限于生命科学或 CS。

| 来源 | 学科范围 |
|---|---|
| Unpaywall | ✅ 全学科(覆盖 Crossref 所有 DOI — 人文、社科、物理、化学、经济等) |
| Semantic Scholar | ✅ 全学科(跨领域学术图谱) |
| arXiv | 物理、数学、CS、统计、定量金融、经济学、EE |
| PubMed Central | 仅生物医学 |
| bioRxiv / medRxiv | 仅生物 / 医学预印本 |
| Sci-Hub | ✅ 全学科(兜底) |

实际使用中，**仅 Unpaywall + Semantic Scholar 两个源就足以覆盖化学、材料、经济、心理学、人文社科等任何领域的 OA 论文** — 它们会自动命中机构知识库、SSRN、RePEc 以及出版商自托管的 OA 版本。

## 对比

### vs 原生 agent(无 skill)

| 功能 | 原生 agent | 本 skill |
|---|---|---|
| DOI → PDF | 临时网络搜索 | 确定性 7 源链 |
| 标题 → DOI | 手动 | `--title`(Crossref + S2 回退,附置信度标记) |
| 批量下载 | ❌ | ✅ `--batch dois.txt` 或 `--batch -` |
| 一致的文件命名 | ❌ | ✅ `author_year_journal_title.pdf` |
| 机器可读 schema | ❌ | ✅ `fetch.py schema` |
| 结构化输出 | ❌ | ✅ JSON 信封 + NDJSON 进度 |
| 幂等重试 | ❌ | ✅ `--idempotency-key` |
| 退出码分类 | ❌ | ✅ `0`/`1`/`3`/`4` |
| SSRF + `%PDF` + 体积上限 | ❌ | ✅ 强制启用 |

## 环境要求

- `python3`(3.8+,仅标准库,无需 `pip install`)
- (推荐)[Unpaywall 联系邮箱](https://unpaywall.org/products/api):

  ```bash
  export UNPAYWALL_EMAIL=you@example.com
  ```

未设置时跳过 Unpaywall,其余 6 个源仍正常工作。

## 安装

```bash
# 任意 Agent(Claude Code、Cursor、Copilot 等)
npx skills add Agents365-ai/365-skills -g

# 仅 Claude Code
> /plugin marketplace add Agents365-ai/365-skills
> /plugin install paper-fetch
```

也已发布到 [SkillsMP](https://skillsmp.com/) 与 [ClawHub](https://clawhub.ai/),它们各自的市场负责更新。

## 使用

直接用自然语言告诉 Agent:

```
> 把 AlphaFold2 论文的 PDF 下载到 ~/papers

> 帮我下 DOI 10.1038/s41586-020-2649-2

> 把 dois.txt 里的全部 DOI 批量下载

> 找 "Attention Is All You Need" 这篇论文的 PDF 并保存

> 不下载,只预览 10.1126/science.abj8754 解析出的 PDF 链接
```

或直接调用脚本:

```bash
# 单个 DOI
python skills/paper-fetch/scripts/fetch.py 10.1038/s41586-021-03819-2

# 仅有标题(经 Crossref + S2 回退解析为 DOI 后下载)
python skills/paper-fetch/scripts/fetch.py --title "Highly accurate protein structure prediction with AlphaFold"

# 干跑预览(不下载)
python skills/paper-fetch/scripts/fetch.py 10.1038/s41586-020-2649-2 --dry-run

# 批量 + 幂等键
python skills/paper-fetch/scripts/fetch.py --batch dois.txt --out ~/papers \
    --idempotency-key monday-review-batch

# 从其他工具用管道传 DOI
echo 10.1038/s41586-021-03819-2 | python skills/paper-fetch/scripts/fetch.py --batch -

# Agent 自描述
python skills/paper-fetch/scripts/fetch.py schema --pretty
```

完整参数与 JSON 信封 schema 见 [`skills/paper-fetch/SKILL.md`](skills/paper-fetch/SKILL.md)。

## 机构访问(可选)

如所在机构有出版商订阅,设 `PAPER_FETCH_INSTITUTIONAL=1` 启用出版商直链回退。授权由你的 IP / cookies / EZproxy 完成;为避免触发出版商 ToS, skill 会自动加 1 req/s 限速。

```bash
export PAPER_FETCH_INSTITUTIONAL=1
```

详见 [`plan/institutional-access.md`](plan/institutional-access.md)。

## 已知限制

- **部分出版商重定向**会返回 HTML 落地页,`%PDF` 魔数校验会拒绝
- **不做浏览器自动化** — 不解 CAPTCHA、不用 Playwright、不做反指纹绕过
- **SSRF 防护**会拒绝私网 IP、非 http(s) 协议、非 80/443 端口、云元数据主机
- **每个 PDF 体积上限 50 MB**

## License

MIT

## 社区

加入交流群获取帮助、提问和最新动态:

- **Discord:** https://discord.gg/79JF5Atuk
- **微信:** 扫描下方二维码

<p align="center">
  <img src="https://raw.githubusercontent.com/Agents365-ai/images_payment/main/qrcode/agents365ai_wechat_1.png" width="200" alt="微信交流群">
</p>

## 支持作者

如果这个 skill 对你有帮助,欢迎打赏支持作者:

<table>
  <tr>
    <td align="center">
      <img src="https://raw.githubusercontent.com/Agents365-ai/images_payment/main/qrcode/wechat-pay.png" width="180" alt="微信支付">
      <br>
      <b>微信支付</b>
    </td>
    <td align="center">
      <img src="https://raw.githubusercontent.com/Agents365-ai/images_payment/main/qrcode/alipay.png" width="180" alt="支付宝">
      <br>
      <b>支付宝</b>
    </td>
    <td align="center">
      <img src="https://raw.githubusercontent.com/Agents365-ai/images_payment/main/qrcode/buymeacoffee.png" width="180" alt="Buy Me a Coffee">
      <br>
      <b>Buy Me a Coffee</b>
    </td>
    <td align="center">
      <img src="https://raw.githubusercontent.com/Agents365-ai/images_payment/main/awarding/award.gif" width="180" alt="打赏">
      <br>
      <b>打赏</b>
    </td>
  </tr>
</table>

## 作者

**Agents365-ai**

- Bilibili: https://space.bilibili.com/441831884
- GitHub: https://github.com/Agents365-ai
