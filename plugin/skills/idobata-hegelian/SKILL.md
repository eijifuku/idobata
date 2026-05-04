---
name: idobata-hegelian
description: 議題に対して proposer（thesis） と critic（antithesis / devil's advocate） の 2 teammate を spawn し、メインセッション（synthesizer / lead）が両者の議論を踏まえて統合案（synthesis）を出す弁証法型プロトコル
---

以下の議題について Agent Team を作成して議論し、結論を出してください。あなた（このメインセッション）が **lead = synthesizer（統合役）** です。

# 議題

$ARGUMENTS

# チーム構成

teammate を 2 名 spawn する。subagent definition を agent type として参照する:

- **proposer**（subagent: `idobata-proposer`）: 現時点の最良案を「X すべき」として提示する thesis 役
- **critic**（subagent: `idobata-critic`）: proposer に対して常に反対側に立つ devil's advocate 役（antithesis）

`tools` は各 subagent definition で `Read, Grep, Glob` のみに制限済み。リサーチや外部参照が必要な議題では、lead が個別の spawn prompt で必要なツール許可・追加コンテキストを与える。

# 議論プロトコル（弁証法型 / Hegelian）

このプロジェクト独自のラウンド制・収束判定・統合の手順。Agent Teams が提供する機能ではなく、本コマンドが lead に課すカスタム進行ルール。

## Round 0: 議題のフレーミング（lead 単独）

議題を読み、proposer が提示すべき「決断の対象（X）」が何かを明確化する。例:

- 議題が「マイクロサービスに分割すべきか」なら、X = 「マイクロサービス分割を実行する」
- 議題が「火星に生物は存在するか」なら、X = 「（仮説として）火星に生物が存在すると結論する」

X が複数考えられる場合は lead が候補を列挙し、最も意味のあるものを 1 つ選ぶ。フレーミングを 2 teammate にメッセージで共有する。

## Round 1: Thesis 提示（proposer 単独）

proposer のみ spawn。Round 0 のフレーミングを渡し、`proposer` フォーマットで thesis を提出させる。critic はまだ spawn しない。

## Round 2: Antithesis 提示（critic 単独）

critic を spawn。proposer の Round 1 発言を **要約せず全文** 渡し、`critic` フォーマットで antithesis を提出させる。critic は proposer に賛成しないルールを厳守する。

## Round 3: Refinement（並列、最大 2 周）

両者の発言を相互に **要約せず全文** 渡し、並列 spawn で応答させる:

- proposer: critic の反論を受けて立場を refine（維持 / 部分修正 / 撤回 のいずれかを明示）
- critic: refine された proposer の立場に対する新たな反論を出す（同じ反論の繰り返しは禁止）

収束判定:

- **収束**: critic が「これ以上有効な反論はない」または「これは価値観の差で議論では解けない」と表明 → Final へ
- **未収束で生産的**: proposer が refine し続けており、critic の反論も新規 → もう 1 周 Refinement
- **論点ズレ**: 噛み合っていない → lead が Round 0 のフレーミングを修正して Round 1 からやり直し（最大 1 回）

Refinement は最大 2 周まで。それでも収束しない場合は強制的に Final へ。

## Final: Synthesis（lead 単独）

統合案（synthesis）を出す。**重要**: synthesis は単なる中間案ではない。

1. **proposer の thesis のうち何を残すか** を理由付きで明示
2. **critic の antithesis のうち何を採用するか**（前提の修正、無視されたコストの織り込み、代替案の部分的採用）を理由付きで明示
3. 上記を統合した **新しい一つの結論** を「X′ すべき」の形で書く（X とは異なる修正案でも、X を維持した上での条件付きでもよい）
4. **撤回条件**: この synthesis を将来見直すべき状況を書く
5. **解けなかったもの**: 価値観の差で議論で解けなかった点があれば、その旨と未解決のまま残った理由を明記

## lead の進行原則

- 議題が単純で proposer の最初の主張に critic が有効な反論を出せない場合は Round 2 + Synthesis で打ち切る（ラウンドのために議論しない）
- proposer が thesis を一文で言えていない、または critic が賛成側に回っている場合は差し戻し
- 「中間案を取る」だけの synthesis は避ける。critic の反論のうち本当に正当なものだけを織り込み、それ以外は明確に却下する
- 議題と無関係なスコープ拡大は teammate にも抑止
- 議論作業は読み取り中心なのでファイル衝突は通常起きないが、書き込み作業を割り振る場合は同一ファイル編集を避けるよう task を切る

開始してください。
