# nwn-ee-l10n-data

[NVN:EE 汉化工作台](https://github.com/bokiHong/nwn-ee-l10n)（`nwn-ee-l10n`）的**公开数据包**。

代码在那边，数据在这边 —— 拆开是因为两者版权风险不同（见下）。

---

## 里面有什么

| 文件 | 条数 | 大小 | 说明 |
| --- | ---: | ---: | --- |
| `data/term_active.json` | 6,581 | 1.0 MB | **正式术语表**（生效中的译名，含「必守」标记） |
| `data/term_candidate.json` | 6,832 | 0.95 MB | **候选术语**（从规则书语料挖掘出来的候选译名池） |
| `data/tm.jsonl` | 59,546 | 21.4 MB | **翻译记忆库**（英文原文 ↔ 中文译文，一行一条） |

数据来源：PRC8 全量汉化 + **Swordflight Chapter One ~ Five**（Ch3/Ch4/Ch5 于 2026-10-11
新译 15,591 条），在真实项目上逐步积累而成。

---

## 怎么用

```bash
# 1) 两个仓库都要克隆
git clone https://github.com/bokiHong/nwn-ee-l10n.git
git clone https://github.com/bokiHong/nwn-ee-l10n-data.git

# 2) 建库 + 导入（幂等，可反复跑）
cd nwn-ee-l10n
python tools/bootstrap.py --data ../nwn-ee-l10n-data/data

# 3) 验收
python -m l10n.selftest
```

只想看条数、不写库：

```bash
python tools/bootstrap.py --data ../nwn-ee-l10n-data/data --dry-run
```

---

## 数据格式

### `term_active.json` / `term_candidate.json`

JSON 数组，每条一个对象：

```json
[
  {
    "en": "Aranea",
    "zh": "变形蛛",
    "type": "monster",
    "required": 1,
    "seq": 1204,
    "src": "manual",
    "note": "规则书译名"
  }
]
```

| 字段 | 说明 |
| --- | --- |
| `en` / `zh` | 英文 / 中文译名 |
| `type` | 分类：`name` `place` `deity` `class` `spell` `item` `feat` `monster` `race` `skill` `psionic` `martial` `domain` `mantle` `prestige_class` `term` `misc` |
| `required` | `1` = **必守**（译文里必须用这个译名，否则报违规）；`0` = 参考 |
| `seq` | 原始导入顺序（导出 `glossary.json` 时要靠它保持逐字节等价） |
| `src` | 溯源：`glossary` / `override` / `draft` / `manual` |
| `note` | 备注（维护者的判断依据） |

### `tm.jsonl`

JSON Lines，一行一条：

```json
{"key": "0002208ae74cc5b21eff5711ced25d9e", "src": "Hexblade Protection from Law", "dst": "魔剑客防护秩序", "mod": "PRC8", "status": "manual", "note": "规则书术语统一(C-变体)"}
```

| 字段 | 说明 |
| --- | --- |
| `key` | **原文内容哈希**（不是序号）—— 这是跨模组复用的关键 |
| `src` / `dst` | 英文原文 / 中文译文 |
| `mod` | 来源模组（`PRC8` / `Swordflight - Chapter One` / …） |
| `status` | `auto`（机器译）/ `manual`（人工译）/ `fallback` |
| `note` | 可选的溯源备注 |

> ⚠️ **`key` 必须原样使用，不要重算**：历史上两条流程的 key 算法不完全一致
> （`norm` 与 `raw`，见 `l10n/keys.py`），目前有 119 条是按 `raw` 算的。
> 重算会让同一句英文变成两条。`tools/bootstrap.py` 已经处理好了。

### 导入后不会有的东西

`unit` / `resource` / `container` 三张表**故意不导出** ——
它们记录「某台机器上某个模组的某个位置」，路径因人而异。
要建它们，自己跑：

```bash
python -m l10n scan    --project mymod --mod <你的.mod>
python -m l10n extract --project mymod --mod <你的.mod>
```

---

## 为什么单独一个仓库

数据的版权风险与代码不同：

* **术语表**是词组级的译名对照（`Aranea` → `变形蛛`），属于事实性数据。
* **翻译记忆库**是整句对照，其中**英文原文来自第三方模组**（PRC8、Swordflight 等），
  权利归各自作者；**中文译文**由本项目维护者创作。

本项目**不主张**对英文原文的任何权利，也**不分发**模组本体文件。
如果你需要这些原文，请自行准备合法的模组文件。

拆成两个仓库的好处：万一有权利人提出异议，**只需归档这个数据仓库**，
工具仓库完全不受影响。

---

## 许可

本数据采用 **Creative Commons Attribution-ShareAlike 4.0 International
(CC BY-SA 4.0)** 许可。

你可以自由地共享、演绎（包括商业用途），惟须**署名**并以**相同方式共享**。

完整文本：https://creativecommons.org/licenses/by-sa/4.0/legalcode.zh-Hans

若你是权利人，认为这些数据侵犯了你的权益，请开 Issue，我们会立即移除。
