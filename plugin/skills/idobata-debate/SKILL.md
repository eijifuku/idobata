---
name: idobata-debate
description: 議題を分析し、最適な議論プロトコル（hegelian / red-blue / six-hats / three-hats / rational-choice / pugh-matrix / delphi / multi-agent-debate）を 1 つ選んで自動的に起動する meta-router コマンド。議論手法選びに迷ったらこれ。
---

以下の議題に最適な議論プロトコルを **あなた（lead）が選択**し、`Skill` ツールで該当 skill を起動してください。あなたは router であり、議論そのものは行いません。

# 議題

$ARGUMENTS

# 強制指定（エスケープハッチ）

議題本文の冒頭に以下のいずれかが含まれる場合、ルーティング判定をスキップして即座に該当 skill を起動する:

- `--force=hegelian` → `idobata:hegelian`
- `--force=red-blue` → `idobata:red-blue`
- `--force=six-hats` → `idobata:six-hats`
- `--force=three-hats` → `idobata:three-hats`
- `--force=rational-choice` → `idobata:rational-choice`
- `--force=pugh-matrix` → `idobata:pugh-matrix`
- `--force=delphi` → `idobata:delphi`
- `--force=multi-agent-debate` → `idobata:multi-agent-debate`
- `--force=brainstorm` → `idobata:brainstorm`
- `--force=scamper` → `idobata:scamper`
- `--force=morphological` → `idobata:morphological`

`--force=` フラグは議題本文から除去してから子 skill に渡す。

# ルーティング手順（強制指定が無い場合）

## Step 1: 議題の要約

議題を 1〜2 文で要約し、議論の **目的タイプ** を以下から 1 つ判定する:

- **A. 二者択一の是非判定** （X すべき / すべきでない、賛成 / 反対）
- **B. 具体提案の堅牢性検証** （既にある計画・設計・PR を叩いて穴を探したい）
- **C. 複数候補からの選定** （3 つ以上の選択肢から 1 つを選ぶ）
- **D. 多面的探索・発散** （まだ立場が定まっていない、視点を広げたい）
- **E. 数値予測・専門家集約** （売上予測、リスク確率、定量見積もり）
- **F. 客観的に正解がある問題** （数学・論理・コードの正誤）
- **G. アイデアの発散・大量生成** （まだ案そのものが無い、量を出したい）
- **H. 既存物の改良・派生案生成** （既にあるものを変形して派生を作りたい）
- **I. 組合せ空間の網羅的探索** （次元 × 値の構造化された大空間からサンプルしたい、サービス設計・実験計画・制度設計など）

## Step 2: プロトコル判定

| 目的タイプ | 第一候補 | 補足 |
|---|---|---|
| A. 二者択一 | `idobata:hegelian` | 立場が真っ二つに割れる構造のとき |
| B. 提案の堅牢性検証 | `idobata:red-blue` | 既に具体物がある（仕様書・PR・設計書） |
| C. 複数候補選定（軽量・反復前提） | `idobata:pugh-matrix` | 候補が成長する・現状維持を datum にしたい |
| C. 複数候補選定（重み付き定量評価） | `idobata:rational-choice` | 評価軸の重みが意思決定に効く |
| D. 多面的探索（フル） | `idobata:six-hats` | 5 視点必要、コスト高め |
| D. 多面的探索（軽量） | `idobata:three-hats` | pragmatist / idealist / risk-analyst の 3 視点で素早く |
| E. 数値予測 | `idobata:delphi` | 匿名 N 専門家で追従バイアス排除 |
| F. 正解がある問題 | `idobata:multi-agent-debate` | 独立解答 → 相互参照で精度向上 |
| G. アイデア発散（ゼロから） | `idobata:brainstorm` | 異質 lens の ideator 並列。critic なし |
| H. 既存物の変形 | `idobata:scamper` | SCAMPER 7 操作で既存物を変形 |
| I. 組合せ空間探索 | `idobata:morphological` | 次元 × 値の Zwicky Box を構築・サンプル |

判定が割れる場合の優先ルール:
- 議題に「どっちが良い」「すべきか」が含まれ候補が 2 つ → hegelian
- 議題に「レビューして」「穴は」「リスクは」が含まれる → red-blue
- 議題に「予測」「何%」「いつ頃」が含まれる → delphi
- 議題に「アイデアを出して」「案を考えて」「発想して」が含まれ、既存物が無い → brainstorm
- 議題に「改良」「派生」「バリエーション」「varying X」が含まれ、既存物が 1 つある → scamper
- 議題に「組合せ」「網羅的に」「パラメータ空間」「次元」「全パターン」が含まれる → morphological
- 迷ったら **three-hats**（軽量で外しにくい）

## Step 3: 判定の明示

ユーザに対し、以下を **必ず** 出力してから skill を起動する:

```
議題要約: <1〜2 文>
目的タイプ: <A〜F のいずれか>
選択プロトコル: idobata:<skill名>
判定根拠: <なぜこれを選んだか 1〜2 文>
```

## Step 4: skill 起動

`Skill` ツールを以下の形で 1 回だけ呼ぶ:

- `skill`: 選択した skill 名（例 `idobata:hegelian`）
- `args`: 元の議題本文（`--force=` フラグは除去済み）

skill 起動後はあなた自身は議論に介入しない。子 skill 側の lead がプロトコルを進行する。

# 重要な制約

- ルーティング判定の根拠は **必ず明示** する。沈黙して skill を起動しない。
- 子 skill を **複数並列** で起動しない。1 議題 = 1 プロトコル。
- 議題が空または曖昧すぎる場合は、skill を起動せずユーザに議題の補足を求める。
