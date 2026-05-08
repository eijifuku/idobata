# idobata

1 つの議題を、立場の違う複数の AI に並列で議論させる Claude Code プラグイン。

`/idobata-six-hats モノリスをマイクロサービスに分割すべきか` のようにスラッシュコマンドを叩くと、メインセッション（lead）が役割の異なる subagent を Agent Teams で並列 spawn し、各々に独立した立場で意見を出させたうえで、lead が争点ごとに裁定・統合する。

単一モデルに「賛否両論を挙げて」と頼むと中庸な平均意見に収束しがちだが、本プラグインは **役割を構造的に分離** することで、批判役が遠慮なく批判し、楽観役が遠慮なく擁護し、感情役が理由付け抜きで反応する、といった **意図的に偏った視点の応酬** を引き出す。

これにより、単一エージェントでは構造的に発生しやすい以下のバイアスを軽減することを狙う:

- **同調・追従バイアス**: 先に出た意見に引きずられて全員が同じ方向に収束する。Delphi の匿名集約や Multi-Agent Debate の独立解答ラウンドで、他者意見を見る前に立場を確定させて排除する。
- **中庸バイアス（平均への回帰）**: 「両論併記」を求めるとリスクも利点も均等に薄まり、結局は無難な結論になる。役割を強制分離することで、各 teammate が自分の lens の中では極端な立場を取り切ることを保証する。
- **確証バイアス・自己擁護**: 提案者が自分の案の弱点を過小評価する。critic / red-team / sensitivity-auditor のように **反対側に立つことが任務** の役割を別 teammate として spawn し、提案者と同じプロンプト空間を共有させない。
- **観点の単一化**: 単一視点で見落とす論点が出る。Six Hats の 5 視点、形態分析の N 次元、ブレストの異質 lens（ペルソナ / 制約 / 類推 / 反転 / ランダム刺激）といった **意図的に直交する複数視点** を並列に走らせ、意見の多様性そのものを構造的に担保する。

 1 体を「立場の違う N 体」に分解することで、**バイアスの排除** と **意見の多様性** を仕組みとして確保する。

**収束系（意思決定・裁定・合意形成）と発散系（アイデア生成・組合せ探索）の両方** に対応している:

- **収束系**: Six Thinking Hats、Hegelian 弁証法、Red-Blue 攻防、Pugh Matrix、Rational Choice Debate、Multi-Agent Debate (Du et al. 2023)、Delphi Method など。争点ごとに勝者を決める／推奨案を出す方向に lead が議論を畳む。
- **発散系**: Brainstorm（Osborn + Crazy 8s）、SCAMPER、Morphological Analysis。critic 役を**置かず**、判断保留で大量のアイデア・組合せを生成する。

合計 12 形式。議題を渡すと自動で最適な形式を選ぶ `/idobata-debate` meta-router もある。

## 提供する議論形式

| コマンド | 説明 |
| --- | --- |
| `/idobata-debate` | **meta-router**。議題を分析し、最適な議論プロトコルを自動選択して起動する。手法選びに迷ったらこれ。`--force=<protocol>` で強制指定も可能 |
| `/idobata-six-hats` | de Bono の Six Thinking Hats 正統プロトコル。白（事実）/ 赤（感情）/ 黒（批判）/ 黄（楽観）/ 緑（創造）の 5 hat と Blue Hat（lead）で議題タイプ別の順序で議論 |
| `/idobata-three-hats` | Six Hats 簡易版（pragmatist / idealist / risk-analyst の 3 視点）。争点ごとに裁定する |
| `/idobata-hegelian` | proposer（thesis） と critic（antithesis / devil's advocate） の議論を踏まえ、lead が統合案（synthesis）を出す弁証法型プロトコル |
| `/idobata-red-blue` | blue-team（擁護・防衛）と red-team（攻撃・破壊）の応酬を、judge（lead）が見て採否を判定する赤チーム・青チーム型プロトコル |
| `/idobata-rational-choice` | 候補 × 評価軸のマトリクスを機械的に組み立て、複数 lens の criteria-proposer / cell-scorer / sensitivity-auditor を経て、重み付き総合スコアと感度を考慮し意思決定する Rational Choice Debate |
| `/idobata-multi-agent-debate` | Du, Li, Torralba et al. 2023 の手法。N 個の対称 solver が独立解答 → 互いの解答を全文共有して再解答、を収束まで繰り返し、lead が最終統合。事実推論・数学・ファクトチェック向け |
| `/idobata-delphi` | RAND 1950 年代の Delphi Method。N 個の delphi-expert が独立意見 → lead が**匿名集約**してフィードバック → 再考を繰り返す。匿名性で追従バイアスを構造的に排除。予測・評価・優先順位付け向け |
| `/idobata-pugh-matrix` | Stuart Pugh 1981 の Pugh Concept Selection。datum（基準候補）と他候補を +/-/S の 3 値で相対比較し、弱点候補を pugh-improver で改良して反復。技術選定の初期スクリーニング・繰り返し検討向け |
| `/idobata-brainstorm` | Osborn 古典ブレスト + Crazy 8s 風。N 個の ideator を異質 lens（ペルソナ / 制約 / 類推 / 反転 / ランダム刺激）で並列 spawn し、判断保留で大量のアイデアを生成。critic なし。発散専用 |
| `/idobata-scamper` | Bob Eberle 1971 の SCAMPER。既存物に Substitute / Combine / Adapt / Modify / Put to other use / Eliminate / Reverse の 7 操作を 7 個の transformer で並列実行し、改良・派生案を大量生成 |
| `/idobata-morphological` | Fritz Zwicky 1969 の Morphological Analysis。お題を直交 N 次元に分解し、各次元の値候補を value-generator が並列生成、組合せ空間から異質組合せをサンプリング。サービス設計・実験計画・制度設計など組合せ空間が膨大な問題に汎用適用 |

今後、別形式（steelman、council 等）を追加予定。

## 使う teammate (subagents)

| name | 役割 |
| --- | --- |
| `idobata:idobata-white-hat` | 事実・データ・客観的情報のみを扱う（six-hats 白） |
| `idobata:idobata-red-hat` | 感情・直感・第一印象（理由付け禁止）を扱う（six-hats 赤） |
| `idobata:idobata-black-hat` | 論理的批判・リスク・失敗モードを扱う（six-hats 黒） |
| `idobata:idobata-yellow-hat` | 論理的楽観・利益・実現可能性を扱う（six-hats 黄） |
| `idobata:idobata-green-hat` | 創造的代替案・突拍子もないアイデアを扱う（six-hats 緑） |
| `idobata:idobata-pragmatist` | 実装可能性・コスト・運用負荷・既存制約から評価する現実主義者（three-hats） |
| `idobata:idobata-idealist` | 理論的正しさ・長期的健全性・あるべき姿から評価する理想主義者（three-hats） |
| `idobata:idobata-risk-analyst` | 失敗モード・最悪シナリオ・見落とされた前提を探すリスク分析家（three-hats） |
| `idobata:idobata-proposer` | 「X すべき」と現時点の最良案を提示する thesis 役（hegelian） |
| `idobata:idobata-critic` | proposer に対し常に反対側に立つ devil's advocate / antithesis 役（hegelian） |
| `idobata:idobata-blue-team` | 提案を擁護・防衛する側。red-team の攻撃に応答する（red-blue） |
| `idobata:idobata-red-team` | 提案を攻撃・破壊する側。失敗シナリオ・前提崩壊・敵対的悪用を提示する（red-blue） |
| `idobata:idobata-criteria-proposer` | 指定 lens から評価軸を提案する。複数 lens 並列 spawn 想定（rational-choice） |
| `idobata:idobata-cell-scorer` | 候補 × 軸セルを軸定義に従い採点・根拠付与（rational-choice） |
| `idobata:idobata-sensitivity-auditor` | 完成マトリクスの重み感度・スコア感度・欠損軸・直交性違反を攻撃（rational-choice） |
| `idobata:idobata-solver` | 独立解答 → 他者解答全文を見て再解答する対称エージェント（multi-agent-debate） |
| `idobata:idobata-delphi-expert` | 匿名集約のみを参照して意見を維持・修正する専門家。追従バイアス排除（delphi） |
| `idobata:idobata-pugh-comparator` | datum と候補 X を軸ごとに +/-/S で相対比較する（pugh-matrix） |
| `idobata:idobata-pugh-improver` | 弱点候補を他候補の長所で改良した A' を生成。反復改良の本質を担う（pugh-matrix） |
| `idobata:idobata-ideator` | 割り当てられた lens（ペルソナ / 制約 / 類推 / 反転 / ランダム刺激）で大量のアイデアを判断保留で出す発散役（brainstorm） |
| `idobata:idobata-transformer` | SCAMPER 7 操作のうち 1 つを担当し、既存物を変形して派生案を大量生成する（scamper） |
| `idobata:idobata-dimension-proposer` | お題を直交する N 次元（パラメータ軸）に分解する形態分析の前段役（morphological） |
| `idobata:idobata-value-generator` | 形態分析で 1 つの次元に対する値候補を大量・多様に生成する（morphological） |

各 teammate は前提 / 主張 / 根拠 / 撤回条件のフォーマットで意見を返し、撤回条件を書けない場合は「価値観の差で議論で解けない」と表明する。

## インストール

Claude Code 内で以下を実行:

```
/plugin marketplace add eijifuku/idobata
/plugin install idobata@idobata-local
/reload-plugins
```

`/plugin marketplace add` は `eijifuku/idobata` のような `owner/repo` 形式または完全 URL `https://github.com/eijifuku/idobata` を受け取る。後者を使う場合:

```
/plugin marketplace add https://github.com/eijifuku/idobata
```

ローカル開発のためにリポジトリを clone して使いたい場合は、clone 後の絶対パスを指定:

```
/plugin marketplace add /path/to/cloned/idobata
```

## 使い方

```
/idobata-six-hats <議題>
```

例:

```
/idobata-six-hats モノリスをマイクロサービスに分割すべきか
/idobata-six-hats 火星に生物は存在するか
```

抽象的な議題や未確定の問題でも機能する。

## 前提: Agent Teams 機能を有効化する

本プラグインの議論プロトコルはすべて、メインセッション（lead）が **Agent Teams** 機能で複数の subagent (teammate) を spawn して並列議論させる仕組みに依存している。Agent Teams は Claude Code の experimental 機能のため、明示的に有効化が必要。

### 有効化方法

`.claude/settings.json`（プロジェクトスコープ）または `~/.claude/settings.json`（ユーザスコープ）に以下を追加:

```json
{
  "env": {
    "CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS": "1"
  }
}
```

本リポジトリは `.claude/settings.json` で既に有効化済み。プラグインを別プロジェクトにインストールして使う場合は、そのプロジェクトの `.claude/settings.json` か、または個人の `~/.claude/settings.json` に同じ env を設定する。

### 確認方法

設定後、Claude Code を起動し直し、`Agent` ツール（`TeamCreate` などを含む agent teams 関連ツール）が利用可能であることを確認する。各議論プロトコルは `Agent` ツールに依存しているため、未有効だと teammate を spawn できず動作しない。

### 注意

- **experimental flag**: 仕様変更や名称変更の可能性あり。Anthropic の公式ドキュメント・changelog を随時確認のこと
- 起動後の env 反映: settings.json 編集後は Claude Code セッションを再起動する必要がある
- マーケットプレイス経由でインストールしても env 設定は自動付与されない（プラグインは settings.json を書き換えない設計）

## ディレクトリ構成

```
idobata/
├── .claude-plugin/
│   └── marketplace.json     # ローカル marketplace 定義
└── plugin/
    ├── .claude-plugin/
    │   └── plugin.json       # プラグインメタデータ
    ├── skills/                              # 議論形式（Claude Code skill 形式。/idobata-<name> で起動）
    │   ├── idobata-debate/SKILL.md           # /idobata-debate（meta-router）
    │   ├── idobata-six-hats/SKILL.md         # de Bono 正統 6 hats
    │   ├── idobata-three-hats/SKILL.md       # 簡易 3 視点
    │   ├── idobata-hegelian/SKILL.md
    │   ├── idobata-red-blue/SKILL.md
    │   ├── idobata-rational-choice/SKILL.md
    │   ├── idobata-pugh-matrix/SKILL.md
    │   ├── idobata-multi-agent-debate/SKILL.md
    │   ├── idobata-delphi/SKILL.md
    │   ├── idobata-brainstorm/SKILL.md
    │   ├── idobata-scamper/SKILL.md
    │   └── idobata-morphological/SKILL.md
    └── agents/
        ├── idobata-white-hat.md        # six-hats
        ├── idobata-red-hat.md          # six-hats
        ├── idobata-black-hat.md        # six-hats
        ├── idobata-yellow-hat.md       # six-hats
        ├── idobata-green-hat.md        # six-hats
        ├── idobata-pragmatist.md       # three-hats
        ├── idobata-idealist.md         # three-hats
        ├── idobata-risk-analyst.md     # three-hats
        ├── idobata-proposer.md         # hegelian
        ├── idobata-critic.md           # hegelian
        ├── idobata-blue-team.md        # red-blue
        ├── idobata-red-team.md         # red-blue
        ├── idobata-criteria-proposer.md # rational-choice
        ├── idobata-cell-scorer.md       # rational-choice
        ├── idobata-sensitivity-auditor.md # rational-choice
        ├── idobata-solver.md            # multi-agent-debate
        ├── idobata-delphi-expert.md     # delphi
        ├── idobata-pugh-comparator.md   # pugh-matrix
        ├── idobata-pugh-improver.md     # pugh-matrix
        ├── idobata-ideator.md           # brainstorm
        ├── idobata-transformer.md       # scamper
        ├── idobata-dimension-proposer.md # morphological
        └── idobata-value-generator.md   # morphological
```

## 新しい議論形式を追加するには

1. `plugin/skills/idobata-<形式名>/SKILL.md` を追加（必要なら新しい teammate も `plugin/agents/idobata-<役割名>.md` に追加）。skill / agent 名は `idobata-` 接頭辞で統一する（Claude Code は plugin skill をスラッシュコマンドとして露出する際に namespace を付けないため、グローバル衝突を避ける目的）。
2. `/reload-plugins` で反映。
3. `/idobata-<形式名>` で使える（スラッシュコマンド形式。Skill ツール経由なら `idobata:idobata-<形式名>` で参照する）。

> 注: 旧形式 `plugin/commands/<形式名>.md` も後方互換でサポートされるが、Claude Code 公式は skill 形式を新規推奨。本リポジトリも skill 形式に統一済み。
