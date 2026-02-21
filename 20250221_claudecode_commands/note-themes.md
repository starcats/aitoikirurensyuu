---
description: 会話ログからnote記事のテーマ候補を分析・提案する
allowed-tools: [Bash, Read]
---

# /note-themes — テーマ分析・提案

現在のプロジェクトの会話ログを読み込み、
note記事として書けそうなテーマを3〜5個提案する。

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

### 4. テーマの分析・提案

抽出した全発言テキストを読み込み、以下の方針でテーマを提案する。

**分析フェーズ（内部処理）:**
- 会話全体の中で独立した「問い」や「気づき」のまとまりを探す
- 思考が転換・深化したポイントを特定する
- 一本のエッセイとして成立しそうなテーマを絞る
- 各テーマの「主張の核」（一文で言えること）を確認する

**提案フォーマット:**
番号付きリストで3〜5テーマを提示する。各テーマに以下を含める：

```
**{番号}. {テーマタイトル（キャッチーに）}**
{そのテーマで言えること・問い・気づきを2〜3文で説明}
想定タグ: [{関連タグ}]
```

**提案後のメッセージ:**
テーマ一覧を出力した後、以下を添える：

> テーマ番号を選んで `/note-essay` を呼ぶと、そのテーマでエッセイを生成します。
> 複数選ぶこともできます（例：「1と3で」）。
