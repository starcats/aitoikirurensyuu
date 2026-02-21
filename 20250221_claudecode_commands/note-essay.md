---
description: 思考・対話系セッションからnote向け読み物エッセイを自動生成
allowed-tools: [Bash, Read, Write, Glob]
---

# /note-essay — 思考対話エッセイジェネレーター

現在のプロジェクトの会話ログを読み込み、対話形式を一切使わず、
起承転結のある「読み物エッセイ」として再構成する。

## 手順

### 1. プロジェクトハッシュの特定

カレントディレクトリのパスをハッシュ形式に変換する。
Bash で以下を実行:
```bash
pwd | sed 's|/|-|g' | sed 's|^-||'
```

### 2. 最新JONLファイルの特定

以下のパターンで最新ファイルを取得（subagentsディレクトリは除外）:
```bash
ls -t ~/.claude/projects/{ハッシュ}/*.jsonl 2>/dev/null | head -1
```

ファイルが見つからない場合は「会話ログが見つかりません」と報告して終了。

### 3. 全発言テキストを時系列で抽出

ユーザー発言・AI返答の両方を時系列順に取得する。
以下をスキップ:
- `{` で始まる（システムJSON・チームメッセージ）
- `#` で始まる長いテキスト（スキル展開）
- `<` で始まる（XMLタグ）
- 30文字以下

```bash
python3 -c "
import json
messages = []
entries = []
for line in open('{JONLパス}'):
    try:
        entries.append(json.loads(line))
    except: pass

for obj in entries:
    role = obj.get('type', '')
    if role not in ('user', 'assistant'):
        continue
    for block in obj.get('message', {}).get('content', []):
        if isinstance(block, dict) and block.get('type') == 'text':
            t = block['text'].strip()
            if (len(t) > 30
                and not t.startswith('{')
                and not t.startswith('#')
                and not t.startswith('<')):
                messages.append(f'[{role.upper()}] {t[:800]}')

print('\n\n'.join(messages))
"
```

### 4. エッセイの生成

抽出した全発言テキストを読み込み、以下の方針でエッセイを作成する。

**テーマの確認:**
ユーザーが `/note-themes` の結果から特定のテーマを指定している場合は、そのテーマに絞ってエッセイを生成する。
指定がない場合は、会話全体から最も核心的なテーマを1つ選んで生成する。

**分析フェーズ（内部処理）:**
- 対象テーマの中心的な問いを特定する
- そのテーマに関連する思考の展開・転換を抽出する
- 印象的・示唆的なフレーズを5〜10個ピックアップする
- 最終的な着地点・気づきを確認する

**執筆方針:**
- 対話形式は一切使わない。「〇〇と聞いた」「AIが答えた」も書かない
- 一人の書き手が思考を辿るような語り口で書く
- 起承転結の構成にする（セクション分けしてよい）
- ピックした印象的フレーズはblockquoteで活かす
- 読んで面白い・気づきがある記事にする
- 文体：ですます調 or だ・である調、どちらでも自然な方を選ぶ
- 長さ：800〜1500字程度

**フォーマット:**
```markdown
---
title: {記事タイトル（キャッチーに、問いかけや逆説を活かす）}
date: {今日の日付 YYYY-MM-DD}
tags: [AI, {会話テーマに応じたタグ}]
---

# {タイトル}

{リード文：何について考えた記事か、2〜3文}

---

{本文：起承転結で展開、セクション見出しあり}

> {印象的なフレーズ引用}

{続き}

---

{締め：余韻のある結び}
```

### 5. ファイルへの書き込み

- 出力先: `{カレントディレクトリ}/articles/`（なければ作成）
- ファイル名: `{YYYYMMDD}_{タイトルをスネークケース}.md`
- 日付は今日の日付を使用

### 6. 完了報告

生成したファイルのパスと、記事のタイトル・文字数を報告する。
