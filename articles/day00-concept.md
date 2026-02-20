---
title: Day 0 -- コンセプトを決めた日
series: 1日1つ、AIと作る極上UIゲーム
day: 00
tags: [ゲーム開発, JavaScript, CSS, Juice, note]
---

# Day 0 -- コンセプトを決めた日

AIと「どんなゲームを作りたいか」について話し合った。

どうぶつの森的な「心地よさ」を追求したミニゲームを量産したい、という方向性になった。Stacklands や Sokpop Collective のように、小さくて完成度の高いゲームを数百円で多数販売するモデルに憧れがある、という話をした。

技術方針として決めたこと:

- HTML1枚完結、GitHub Pagesで即公開
- Juice Components（Spring物理、ポヨヨン、FloatingNumber、ScreenShake等）をライブラリ化したい
- ドットテイスト x CSSアプローチ（Press Start 2Pフォント + `image-rendering: pixelated`）が有力
- 将来的にオリジナルのJuice Engineを作りたい。Canvas 2Dベース、Spring物理が核心
- Unity/Godotは重すぎる。Juice特化の軽量エンジンを目指す

収益モデルとして、note記事の有料/無料パート分けも検討中という話が出た。

次はDay 1として、テキストエンジンを作ることにした。
