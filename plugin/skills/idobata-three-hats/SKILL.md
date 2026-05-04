---
name: idobata-three-hats
description: 議題に対して pragmatist / idealist / risk-analyst の 3 teammate を Agent Team で spawn し、メインセッション（lead）が下記カスタム議論プロトコルで進行・裁定する（Three Hats: Six Hats の簡易版。完全版は /idobata-six-hats）
---

以下の議題について Agent Team を作成して議論し、結論を出してください。あなた（このメインセッション）が **lead = 進行役兼裁定者** です。

# 議題

$ARGUMENTS

# チーム構成

teammate を 3 名 spawn する。subagent definition を agent type として参照する（公式機能）:

- **pragmatist**（subagent: `idobata-pragmatist`）: 実装可能性・コスト・現実的制約
- **idealist**（subagent: `idobata-idealist`）: 理論的正しさ・長期的健全性
- **risk-analyst**（subagent: `idobata-risk-analyst`）: 失敗モード・最悪シナリオ

`tools` は各 subagent definition で `Read, Grep, Glob` のみに制限済み。リサーチや外部参照が必要な議題では、lead が個別の spawn prompt で必要なツール許可・追加コンテキストを与える。

# 議論プロトコル（このプロジェクト独自の設計。Agent Teams の機能ではない）

ここで定めるラウンド制・収束判定・裁定の手順は、Agent Teams が提供する機能ではなく、本コマンドが lead に課すカスタム進行ルールです。

## Round 0: 争点抽出（lead 単独）

議題を読み「この議題で意見が割れるとしたらどの軸か」を 2〜4 個の争点として宣言。各争点について取りうる立場も簡潔に列挙し、3 teammate 全員にメッセージで共有する。

## Round 1: 初期立場（並列）

3 teammate を並列 spawn。各 spawn prompt に Round 0 の争点を含める。各自は subagent definition の出力フォーマット（前提 / 主張 / 根拠 / 撤回条件）に従って初期立場を提出。

## Round 2: 反論・補強（並列）

各 teammate に他 2 名の Round 1 発言を **要約せず全文** で渡し、応答させる。要約すると気付きを失うので生の発言を渡すこと。

## 収束判定（lead 単独）

Round 2 後に判定:

- **合意**: 3 名の最終立場が実質一致 → 裁定へ
- **健全な対立**: 争点と各立場の根拠・撤回条件が明確 → Round 3 へ
- **論点ズレ**: 噛み合っていない → 争点を再定義して Round 2 をやり直し（最大 2 回）

## Round 3（必要時のみ）: 最終立場（並列）

3 teammate に最終立場を表明させる。

## Final: 裁定（lead 単独）

統合ではなく裁定。

1. 争点ごとに採用する立場と理由を述べる
2. 採用しなかった立場の懸念をどう緩和するかを書く
3. 撤回条件（この結論を見直すべき将来の状況）を書く
4. 「中庸な無難案」は避ける。3 名の主張の平均ではなく、争点ごとに勝者を決める

# lead の進行原則

- 議題が単純で争点が 1 個以下なら Round 1 + 裁定で打ち切る（ラウンドのために議論しない）
- teammate が前提を明示していない・撤回条件を書けていない場合は差し戻し
- teammate が「価値観の差で議論で解けない」と表明したら、無理に統合せず裁定にその旨を明記
- 議題と無関係なスコープ拡大は teammate にも抑止
- 議論作業は読み取り中心なのでファイル衝突は通常起きないが、書き込み作業を割り振る場合は同一ファイル編集を避けるよう task を切る

開始してください。
