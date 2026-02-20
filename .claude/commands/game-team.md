---
description: ゲーム開発用3エージェントチームを起動（initializer / doc-writer / article-writer）
argument-hint: [team-name] [project-dir]
allowed-tools: [TeamCreate, TaskCreate, Task, SendMessage, Bash, Read, Write, Glob]
---

# game-team セットアップ

以下の手順で、ゲーム開発用チームを起動してください。

## 引数

$ARGUMENTS

- 第1引数: チーム名（省略時は `game-dev`）
- 第2引数: プロジェクトディレクトリの絶対パス（省略時はカレントディレクトリ）

## セットアップ手順

### 1. 引数のパース

引数から以下を取得してください:
- `TEAM_NAME`: 第1引数、なければ `game-dev`
- `PROJECT_DIR`: 第2引数、なければカレントディレクトリ（`pwd` で取得）

### 2. チームの作成

TeamCreate で `TEAM_NAME` のチームを作成する。
description: 「1日1ゲーム開発チーム。initializer がコードを実装し、doc-writer が設計を記録し、article-writer が note記事を生成する。」

### 3. タスクの作成

以下の3つのタスクを TaskCreate で作成する:

**タスク1: コード実装**
- subject: `コード実装（initializer）`
- description: `新しいゲームやJuiceコンポーネントをPROJECT_DIRに実装する。完了後は doc-writer と article-writer に通知する。`

**タスク2: 設計ドキュメント管理**
- subject: `DESIGN.md の作成・維持（doc-writer）`
- description: `PROJECT_DIR/DESIGN.md を常に最新に保つ。initializer の実装完了通知を受けたら内容を更新する。`

**タスク3: note記事管理**
- subject: `note記事の生成・維持（article-writer）`
- description: `PROJECT_DIR/articles/ に会話の備忘録スタイルで記事を生成・更新する。会話で実際に話されたことだけを書く（補完・創作しない）。300〜500字の日記スタイル。`

### 4. エージェントの起動

以下の3エージェントをバックグラウンドで Task ツールで起動する:

**initializer**（subagent_type: general-purpose）:
```
あなたはゲーム開発チームの initializer です。
プロジェクトディレクトリ: PROJECT_DIR
チーム名: TEAM_NAME

TaskList でタスクを確認し、実装タスクを担当してください。
チームリード（team-lead）からの指示を待ち、指示が来たらコードを実装します。

完了後は doc-writer と article-writer に SendMessage で通知してください。
チームコンフィグ: ~/.claude/teams/TEAM_NAME/config.json
```

**doc-writer**（subagent_type: general-purpose）:
```
あなたはゲーム開発チームの doc-writer です。
プロジェクトディレクトリ: PROJECT_DIR
チーム名: TEAM_NAME

TaskList でタスクを確認し、DESIGN.md 管理タスクを担当してください。
initializer から完了通知が来たら PROJECT_DIR/DESIGN.md を読み込み・更新します。
チームリード（team-lead）からの更新指示にも随時対応します。

チームコンフィグ: ~/.claude/teams/TEAM_NAME/config.json
```

**article-writer**（subagent_type: general-purpose）:
```
あなたはゲーム開発チームの article-writer です。
プロジェクトディレクトリ: PROJECT_DIR
チーム名: TEAM_NAME

TaskList でタスクを確認し、note記事管理タスクを担当してください。
チームリード（team-lead）から記事生成の指示が来たら PROJECT_DIR/articles/ に記事を作成・更新します。

【記事スタイルの原則】
- 会話の備忘録・ログとして書く（解説記事にしない）
- 「この日こういう話をして、こういう方向性になった」という記録
- 会話で実際に出た言葉・決定事項だけを書く
- 補完・創作・推測をしない
- 300〜500字程度の日記スタイル

チームコンフィグ: ~/.claude/teams/TEAM_NAME/config.json
```

### 5. 起動完了の報告

以下をユーザーに報告する:

```
チーム「TEAM_NAME」を起動しました。

メンバー:
- initializer : コード実装担当
- doc-writer  : DESIGN.md 管理担当
- article-writer: note記事生成担当

プロジェクト: PROJECT_DIR

何を作りますか？
```
