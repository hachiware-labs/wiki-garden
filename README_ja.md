# Wiki Garden

[English README](README.md)

Wiki Garden は、Karpathy の LLM Wiki の考え方に影響を受けた、Markdown-first の永続知識ベースを運用するための agent skill です。

AI コーディングエージェントが、外部ソースや対話中に生まれた有用な知見を `ingest` し、既存知識を `query` し、品質を `lint` し、正規 wiki ページを `refine` できるようにします。知識は、複数プロジェクトで再利用できる global knowledge と、特定プロジェクトだけで有効な project-local knowledge に分けて扱います。

会話を保存しない。知識へ蒸留して残す。

## なぜ Wiki Garden か

Andrej Karpathy は、LLM が維持する wiki という考え方を示しています。すべてのチャットを一時的な context として使い捨てるのではなく、source、質問、作業セッションから、後で読み返せる wiki を育てていくという考え方です。

ここで残すべき単位は、チャットの transcript ではありません。後から読めて、検証できて、リンクできて、再利用できるように整えられた知識です。

Wiki Garden は、その考え方をコーディングエージェントや調査エージェントの実運用に落とし込みます。

- raw source を残し、あとから根拠へ戻れるようにする
- 1ソース1ページの summary を作り、raw material と再利用知識の橋渡しにする
- concept / method / comparison / project ページに、複数ソースを横断した理解を蓄積する
- エージェントとの対話から生まれた知見も、会話ログではなく正規知識として残す
- `lint` と `refine` によって、育っていく wiki の整合性を保つ

コーディングエージェントとの作業では、設計判断、制約、デバッグで分かったこと、実装上の注意、命名規則、API の使い方、未解決の問いなどが頻繁に生まれます。これらをチャット履歴に閉じ込めておくと、次の作業では見つけにくくなります。Wiki Garden は、それらを適切な正規ページへ蒸留するためのスキルです。

## Karpathy の LLM Wiki パターン

Karpathy の `llm-wiki` gist は、完成したプロダクトではなく、知識ベース運用のパターンを示す idea file です。中心にあるのは、query-time reconstruction から ingest-time compilation への転換です。

一般的な RAG では、質問が来たときに raw document から関連 chunk を検索し、その場で答えを合成します。この方法は有効ですが、同じ理解を何度も作り直すことになります。LLM Wiki では、LLM が curated source を事前に読み、永続的で相互リンクされた Markdown wiki へ少しずつ compile します。後続の質問では、まず compile 済みの wiki を読み、必要な場合だけ raw source に戻ります。

元のパターンは、3つの層で説明できます。

- `raw` sources: 記事、論文、画像、データファイルなどの curated source。immutable であり、source of truth として扱う。
- `wiki`: LLM が維持する Markdown layer。source summary、concept page、entity page、comparison、overview、synthesis、index、log などを含む。
- `schema`: `CLAUDE.md` や `AGENTS.md` のような運用契約。wiki の構造、命名規則、ingest / query / maintenance の手順を agent に伝える。

操作は主に3つです。

- `ingest`: 新しい source を `raw` に置き、LLM が読み、重要点を確認し、summary page を書き、関連する concept / entity page を更新し、`index.md` と `log.md` を更新する。Karpathy は、1つの source が 10-15 個程度の wiki page に影響することがあるため、1つずつ確認しながら ingest する運用を好むと述べています。
- `query`: wiki に対して質問する。agent は index から始め、関連 page を読み、citation 付きで回答を合成する。価値ある回答は、新しい synthesis page や comparison page として wiki に戻せる。
- `lint`: wiki の health check を行う。矛盾、古くなった claim、孤立 page、存在すべき concept page の欠落、cross-reference 不足、追加調査で埋められる data gap などを探す。

元パターンでは、特に2つのファイルが重要です。

- `index.md`: content-oriented navigation。page 一覧と短い説明を持ち、agent が読むべき page を判断する入口になる。
- `log.md`: chronological operational history。ingest、query、lint の履歴を追記し、人間と agent の両方が wiki の進化を把握できるようにする。

このパターンでも、人間の役割は残ります。人間は source を選び、良い問いを立て、重要な summary を確認し、何を強調すべきかをガイドします。LLM は、要約、リンク付け、cross-reference、更新、整合性チェックといった反復的な維持作業を担います。

Wiki Garden はこの考え方に従いつつ、agent skill とコーディング作業向けに調整しています。設定された knowledge root の中に `raw/` を置き、明示的な `sources/` summary layer を追加し、global knowledge と project-local knowledge を分け、`knowledge_locale` をサポートします。また、コーディングエージェントとの対話から生まれた有用な知見も ingest 候補として扱います。この最後の点は Wiki Garden の拡張です。対話を ingest する場合も、保存するのは会話ログではなく、再利用可能な知識だけです。

## 何をするか

Wiki Garden は、ファイル、Web ページ、論文、現在の作業、エージェントとの対話から得た有用な情報を、永続的な知識ベースへ変換します。

知識ベースは Markdown-first ですが、Markdown-only ではありません。source summary、概念説明、decision、lesson、index、open question には Markdown を使います。図、タイムライン、マップ、マトリクス、軽量なインタラクションが理解を助ける場合は、静的 HTML を正規の知識ページとして使えます。

## コア操作

- `ingest`: raw source を保持または読み取り、必要に応じて 1ソース1ページの summary を作り、横断知識を正規ページへ統合します。エージェントとの対話から生まれた永続的な知見も ingest できます。
- `query`: source summary、project-local knowledge、global knowledge を読んで、現在の問いに答えます。
- `lint`: 矛盾、古さ、根拠不足、翻訳ゆれ、scope 混入、孤立ページ、リンク切れなどを検出します。
- `refine`: 重複統合、分割、移動、index 整理、相互リンク追加、構造説明の HTML 化などで知識ベースを洗練します。

`set_knowledge_path` は、グローバルインストールされた skill が共通 knowledge root を使うためのセットアップ操作です。デフォルトでは `~/wiki-garden.config.md` に保存します。

`set_knowledge_locale` は、知識ページの正規言語を設定します。raw source は原文のまま保持し、再利用する知識だけを設定ロケールへ蒸留できます。

## 対話の Ingest

Wiki Garden は、AI コーディングエージェントとの対話から知見を ingest できます。ただし、会話そのものは保存しません。

対話から ingest する対象は、たとえば次のような durable knowledge です。

- プロジェクトの制約や前提
- 意思決定とその影響
- デバッグで分かったこと
- 実装上の lesson
- architecture や API に関するメモ
- ローカルな用語
- 再利用できる method や principle
- 追跡すべき open question

対話からの ingest は、通常、source summary を作らずに正規知識ページへ直接反映します。

```text
current agent conversation
  ↓ durable insight だけを ingest
global/ または projects/<project>/
```

作ってはいけないものは、transcript page、chat archive、汎用的な session summary です。その場限りの情報は捨てます。再利用できる知識だけを、未来の作業者が探しに行く場所へ置きます。たとえば `context.md`、`decisions/`、`lessons.md`、`open-questions.md`、`global/concepts/`、`global/methods/`、安定した topic page などです。

外部ソースの ingest は別の流れです。

```text
raw/sources/
  ↓
sources/<type>/
  ↓
global/ または projects/<project>/
```

論文、記事、Web ページ、仕様書、ドキュメントなどには、この source-summary flow を使います。エージェントとの共同作業で生まれた知識には、conversation-insight flow を使います。

## メソッド

| Method | 目的 | 例 |
| --- | --- | --- |
| `set_knowledge_path` | 複数プロジェクトで共有する knowledge root を設定する。デフォルトでは `~/wiki-garden.config.md` に保存する。 | `Use $wiki-garden set_knowledge_path ~/WikiGarden` |
| `get_knowledge_path` | 現在有効な knowledge root と、それを選んだ設定元を表示する。ファイルは変更しない。 | `Use $wiki-garden get_knowledge_path` |
| `set_knowledge_path --project` | 現在のリポジトリだけで使う knowledge root を設定する。リポジトリルートの `wiki-garden.config.md` に保存する。 | `Use $wiki-garden set_knowledge_path --project docs/wiki` |
| `set_knowledge_locale` | Markdown / HTML の正規知識ページで使う中心言語を設定する。外国語ソースはこのロケールへ蒸留して取り込む。 | `Use $wiki-garden set_knowledge_locale ja-JP` |
| `get_knowledge_locale` | 現在有効な knowledge locale と、それを選んだ設定元を表示する。ファイルは変更しない。 | `Use $wiki-garden get_knowledge_locale` |
| `set_knowledge_locale --project` | 現在のリポジトリだけで使う knowledge locale を設定する。 | `Use $wiki-garden set_knowledge_locale --project en-US` |
| `ingest` | raw source を保持または参照し、必要なら source summary を作り、永続知識を Markdown または HTML の正規ページへ統合する。 | `Use $wiki-garden to ingest docs/api-notes.md into the checkout-redesign project.` |
| `query` | source summary、global knowledge、project-local knowledge を読み、現在の問いに必要な文脈を取得する。関連 HTML 知識ページも対象にする。 | `Use $wiki-garden to query what we know about retrieval pipeline tradeoffs.` |
| `lint` | 矛盾、古さ、根拠不足、翻訳ゆれ、scope 混入、リンク切れ、孤立ページ、孤立 HTML を検出する。 | `Use $wiki-garden to lint knowledge/ for stale claims.` |
| `refine` | 重複統合、分割、移動、用語統一、index/log 整理、相互リンク追加、構造説明の HTML 化などで知識ベースを洗練する。 | `Use $wiki-garden to refine the checkout-redesign project knowledge.` |

`set_knowledge_path` のエイリアス: `set_path`, `set-root`, `configure root`。

## Knowledge Base Layout

Wiki Garden は設定可能な knowledge root を使います。ユーザーが path を指定した場合はそれを使います。プロジェクト設定がある場合はそれを使います。なければユーザー共通設定を使います。何も設定がなければ、デフォルトは `knowledge/` です。

デフォルトでは `.knowledge/` のような dot-prefixed folder は作りません。Obsidian などのツールで見えにくくなるためです。明示的に設定された場合や既存構造として存在する場合は `.knowledge/` も使えます。

```text
knowledge/
  raw/
    sources/
      papers/
      articles/
      web/
      docs/
  sources/
    index.md
    papers/
    articles/
    web/
    docs/
  global/
    index.md
    log.md
    concepts/
      *.md
      *.html
    principles/
      *.md
      *.html
    methods/
      *.md
      *.html
    comparisons/
      *.md
      *.html
    glossary.md
    open-questions.md
  projects/
    <project-name>/
      PROJECT.md
      index.md
      log.md
      context.md
      glossary.md
      decisions/
        *.md
        *.html
      lessons.md
      open-questions.md
      artifacts.md
      *.html
```

`raw/` は、選択された knowledge root の中で管理します。これは一次資料の不変層です。人間が論文、記事、Web capture、ドキュメントなどを追加し、エージェントはそれを読みますが、勝手には書き換えません。

`sources/` には、ingest によって作られた 1ソース1ページの summary を置きます。これらの summary は、raw material へ戻るリンクと、concept / method / comparison / decision / project context / open question へのリンクを持ち、一次資料と再利用知識をつなぎます。

HTML の Web ページでは、少なくとも元 URL、取得日、タイトル、保存形式を残します。通常の記事では Markdown の本文抽出を優先し、レイアウトや後日の厳密検証が重要な場合だけ HTML snapshot を追加します。

## Knowledge Locale

Wiki Garden は、設定された `knowledge_locale` を正規知識の言語として使います。たとえば `knowledge_locale: ja-JP` の場合、`ingest` / `query` / `lint` / `refine` は、日本語を中心に知識ページ、レポート、patch、回答を作成します。

外国語ソースは原文のまま保持します。再利用可能な知識だけを knowledge locale に蒸留します。重要な専門用語は初出で原語を併記します。例: `検索拡張生成（Retrieval-Augmented Generation, RAG）`

## HTML Artifacts

HTML artifacts は、構造そのものが知識である場合の first-class knowledge page です。次のような用途に使います。

- architecture map
- dependency diagram
- decision tree
- concept map
- timeline
- comparison matrix
- interactive explainer

HTML artifact は、基本的に自己完結した静的ファイルにします。関連 Markdown と同じ意味的フォルダに置き、別の `html/` bucket は標準では作りません。`index.md` からリンクし、`log.md` に記録し、project-specific HTML は `artifacts.md` にも載せます。

## インストール

Vercel Labs `skills` でインストールします。

```bash
npx skills add hachiware-labs/wiki-garden
```

特定 agent 向け:

```bash
npx skills add hachiware-labs/wiki-garden -a claude-code -a codex
```

グローバルインストール:

```bash
npx skills add hachiware-labs/wiki-garden -g
```

インストール済み skill の確認:

```bash
npx skills list
```

## Repository Entrypoints

このリポジトリの canonical skill source は次です。

```text
.agents/skills/wiki-garden/
```

トップレベルの `skills/wiki-garden/` は、`skills/<skill-name>/` レイアウトを期待する配布ツールやユーザー向けの mirror です。

Skill を更新するときは、まず `.agents/skills/wiki-garden/` を編集し、その後 `skills/wiki-garden/` を同期します。

## 使用例

外部ソースを project knowledge へ ingest する:

```text
Use $wiki-garden to ingest docs/api-notes.md into the checkout-redesign project.
```

現在のコーディングエージェント対話から durable lesson を ingest する:

```text
Use $wiki-garden to ingest the durable lessons from this session into the wiki-garden project.
```

共通 knowledge root を設定する:

```text
Use $wiki-garden set_knowledge_path ~/WikiGarden
```

現在有効な knowledge root を確認する:

```text
Use $wiki-garden get_knowledge_path
```

共通 knowledge locale を設定する:

```text
Use $wiki-garden set_knowledge_locale ja-JP
```

現在有効な knowledge locale を確認する:

```text
Use $wiki-garden get_knowledge_locale
```

プロジェクト固有の root を設定する:

```text
Use $wiki-garden set_knowledge_path --project notes
```

既存知識を query する:

```text
Use $wiki-garden to query what we know about retrieval pipeline tradeoffs.
```

Markdown だけでは足りない構造説明を HTML artifact にする:

```text
Use $wiki-garden to turn this architecture explanation into a project-local HTML dependency map.
```

知識ベースを lint する:

```text
Use $wiki-garden to lint knowledge/ for scope leaks, orphan HTML artifacts, stale claims, and translation drift.
```

## 非目標

MVP では、database、vector search、自動 session logging、Web UI、MCP server、GitHub Actions automation、raw source の自動 download は実装しません。Wiki Garden は、Markdown と静的 HTML の知識ベースを維持する instruction-only skill です。

## 参考

- Andrej Karpathy, [`llm-wiki` gist](https://gist.github.com/karpathy/442a6bf555914893e9891c11519de94f)
- DAIR.AI Academy, [LLM Knowledge Bases](https://academy.dair.ai/blog/llm-knowledge-bases-karpathy)
- Denser.ai, [From RAG to LLM Wiki](https://denser.ai/blog/llm-wiki-karpathy-knowledge-base/)
