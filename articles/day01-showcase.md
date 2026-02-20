---
title: Day 1: Claude Code でゲームエンジンとチームを同時に作った
date: 2026-02-20
tags: [Claude Code, ゲーム開発, AI]
---

# Day 1: Claude Code でゲームエンジンとチームを同時に作った

Claude Code に指示を出しながら、HTML1枚のテキストアドベンチャーエンジンと、それを支えるエージェントチームを作った記録。

---

## やったこと

**「チーム機能で一人初期を作って、常に .md ファイルにまとめる更新作業する人も作っておいて。noteでの連載『1日1つ、AIと作る極上UIゲーム』のベースとなる、ミニマルで手触りの良いゲームエンジン（HTML1枚完結）を開発したい」**

→ Claude Code がチーム `game-engine-dev` を立ち上げ、`initializer`（実装担当）と `doc-writer`（ドキュメント担当）の2エージェントを同時に起動した。

---

**initializer エージェントが `index.html` を実装。doc-writer は完了通知を受けて待機。**

→ 約2分で `index.html` が生成された。仕様は以下：

- テキストエンジン: 1文字ずつ表示、句読点（。、！？）でウェイト変化（BASE_DELAY 45ms）
- ボタンホバー時 `cubic-bezier(.34, 1.56, .64, 1)` によるSpring風拡大
- 決定時 `popBounce` アニメーション（scale 1→1.2→0.9→1.05→1）
- シナリオJSON形式のデータドリブン設計（無人島サンプル、5シーン）
- 背景グラデーション 20秒周期アニメーション、白半透明ウィンドウ＋backdrop-filter

---

**doc-writer が `index.html` を読み込んで `DESIGN.md` を自動生成。**

→ `DESIGN.md` として全8セクションが出力された。アーキテクチャ図、アニメーション設計の詳細、シナリオJSON仕様、note記事用の解説ポイント3本まで含む。

---

**「放置ゲーム、クリッカーゲームとか好きで、Juiceでパーツをいくつか作って、それらを統合してミニマルなゲームをたくさん作ってみたい。それらをログとしてnoteで記事化して、どっかで有料販売とかできたらいいな」**

→ Claude Code が以下のコンポーネント構成案を提示：

```
JuiceButton    - クリック時にポヨヨン＋パーティクル
FloatingNumber - +100 が跳ねながら消える
ProgressBar    - スムーズに溜まる、満タンで光る
ComboDisplay   - 連続クリックでコンボ演出
ScreenShake    - 大きい出来事でカメラ揺れ
IdleCounter    - 数字が滑らかにインクリメント
```

---

**「Stacklandsとかのインディゲームの人とか結構理想系、数百円でいろんなゲーム作ってる」**

→ Sokpop Collective モデルを日本語Web版に置き換えた比較表を提示。Steam → note有料記事（¥300〜500）、Unity/Godot → HTML1枚＋Juice Components という対応関係。

---

**「まずはドットテイストで心地よいを追求みたいなのはどうなんだろう」**

→ CSSピクセルスタイル（`Press Start 2P` フォント＋`border-radius: 0`＋`image-rendering: pixelated`）がコスパ最高という回答。実ドット絵なしでドット感が出せ、Springアニメーションとの相性も良いと整理。

---

**「Unity / Godot は触ってるけど、ちょっとリッチすぎるから、オリジナルjuice特化なゲームエンジン的なのも作りたい」**

→ 「Juice Engine」コンセプトのロードマップが提示された：

```
Phase 1: Juice Primitives（Spring / Tween / Shake / Freeze / Particles）
Phase 2: Canvas 2D Renderer（pixelated scaling＋パレット制限）
Phase 3: Game Loop + Entity
Phase 4: ミニゲーム第1弾で検証
Phase 5: note で公開・フィードバック
```

---

**「こういう会話も要約して、ベットそのままnoteの記事にできるように記事作成メンバー常に作るのどうだろう」**

→ `article-writer` エージェントをチームに追加。initializer・doc-writer の完了を受けて記事を並行生成するワークフローが構成された。

---

**「とりあえず最初はgithub pagesでいいかな。毎回リポジトリ変えようか同じリポジトリでURLだけ変えていくのとどっちがいいかな」**

→ 同一リポジトリのモノレポ構成を推奨：

```
38_Game/
  index.html                 ← ポータル
  juice/juice.js             ← 共通Juiceライブラリ
  games/day01-text-adventure/index.html
  articles/
  DESIGN.md
```

---

## 最終的にできたもの

- `index.html` — ゲームポータル
- `games/day01-text-adventure/index.html` — どうぶつの森風テキストアドベンチャーエンジン（HTML1枚完結、330行）
- `juice/juice.js` — 共通Juiceライブラリ（スケルトン）
- `DESIGN.md` — アーキテクチャ・アニメーション設計・シナリオJSON仕様
- `articles/day00-concept.md` — シリーズコンセプト記事
- `articles/day01-textengine.md` — テキストエンジン実装解説記事
- チーム `game-engine-dev`（initializer / doc-writer / article-writer）の常駐体制
- `.claude/commands/game-team.md` — `/game-team` コマンド（次回から一発起動）
