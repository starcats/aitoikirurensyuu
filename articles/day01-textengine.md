---
title: Day 1 -- テキストエンジンができた
series: 1日1つ、AIと作る極上UIゲーム
day: 01
tags: [ゲーム開発, JavaScript, CSS, Juice, テキストエンジン, note]
---

# Day 1 -- テキストエンジンができた

AIと一緒に、HTML1枚完結のテキストアドベンチャーエンジンを作った。「潮風の島」という5シーンの短い物語が動くところまでできた。

ファイル: `games/day01-text-adventure/index.html`（330行）

できたもの:

- 1文字ずつ表示するテキストエンジン（BASE_DELAY 45ms、句読点で追加ウェイト）
- 選択肢ボタン（ホバーでポヨン、決定でpopBounce）
- シーン遷移（白フェード）
- すりガラス風メッセージウィンドウ
- シナリオはJSONオブジェクトで定義、データとエンジンが分離している

---

<!-- ここから有料パート -->

コード全文（`games/day01-text-adventure/index.html`）:

```html
<!DOCTYPE html>
<html lang="ja">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>潮風の島</title>
<style>
  *,*::before,*::after{margin:0;padding:0;box-sizing:border-box}

  body{
    font-family:'Hiragino Kaku Gothic ProN','Hiragino Sans',sans-serif;
    min-height:100vh;
    display:flex;
    align-items:center;
    justify-content:center;
    background:linear-gradient(135deg,#e8f5e9 0%,#e3f2fd 50%,#fff8e1 100%);
    background-size:400% 400%;
    animation:bgShift 20s ease infinite;
    overflow:hidden;
  }

  @keyframes bgShift{
    0%{background-position:0% 50%}
    50%{background-position:100% 50%}
    100%{background-position:0% 50%}
  }

  #game{
    width:min(640px,90vw);
    display:flex;
    flex-direction:column;
    gap:20px;
    opacity:0;
    animation:fadeIn .8s ease forwards;
  }

  @keyframes fadeIn{to{opacity:1}}

  #message-window{
    position:relative;
    background:rgba(255,255,255,.72);
    backdrop-filter:blur(14px);
    -webkit-backdrop-filter:blur(14px);
    border-radius:16px;
    padding:28px 32px;
    min-height:160px;
    box-shadow:0 4px 24px rgba(0,0,0,.06);
    border:1px solid rgba(255,255,255,.8);
    display:flex;
    align-items:flex-start;
  }

  #text-display{
    font-size:1.05rem;
    line-height:1.9;
    color:#3e4a40;
    letter-spacing:.04em;
    white-space:pre-wrap;
  }

  #text-display .cursor{
    display:inline-block;
    width:2px;
    height:1.1em;
    background:#7cb342;
    vertical-align:text-bottom;
    animation:blink .7s step-end infinite;
    margin-left:2px;
  }

  @keyframes blink{50%{opacity:0}}

  #choices{
    display:flex;
    flex-direction:column;
    gap:12px;
    min-height:60px;
  }

  .choice-btn{
    appearance:none;
    border:none;
    background:rgba(255,255,255,.78);
    backdrop-filter:blur(10px);
    -webkit-backdrop-filter:blur(10px);
    border-radius:12px;
    padding:14px 24px;
    font-family:inherit;
    font-size:.95rem;
    color:#4e6e52;
    cursor:pointer;
    box-shadow:0 2px 12px rgba(0,0,0,.05);
    border:1px solid rgba(200,230,200,.5);
    transition:transform .35s cubic-bezier(.34,1.56,.64,1),
               box-shadow .3s ease,
               background .3s ease;
    transform:scale(1);
    opacity:0;
    animation:choiceFadeIn .4s ease forwards;
  }

  .choice-btn:nth-child(1){animation-delay:.05s}
  .choice-btn:nth-child(2){animation-delay:.15s}
  .choice-btn:nth-child(3){animation-delay:.25s}

  @keyframes choiceFadeIn{
    from{opacity:0;transform:translateY(8px) scale(.97)}
    to{opacity:1;transform:translateY(0) scale(1)}
  }

  .choice-btn:hover{
    transform:scale(1.05);
    background:rgba(255,255,255,.92);
    box-shadow:0 4px 18px rgba(0,0,0,.08);
  }

  .choice-btn:active{
    transform:scale(.97);
  }

  .choice-btn.pop{
    animation:popBounce .5s cubic-bezier(.34,1.56,.64,1) forwards;
    pointer-events:none;
  }

  @keyframes popBounce{
    0%{transform:scale(1)}
    20%{transform:scale(1.2)}
    45%{transform:scale(.9)}
    70%{transform:scale(1.05)}
    100%{transform:scale(1)}
  }

  #overlay{
    position:fixed;
    inset:0;
    background:rgba(255,255,255,.95);
    opacity:0;
    pointer-events:none;
    transition:opacity .4s ease;
    z-index:100;
  }
  #overlay.active{
    opacity:1;
    pointer-events:all;
  }
</style>
</head>
<body>

<div id="overlay"></div>

<div id="game">
  <div id="message-window">
    <div id="text-display"></div>
  </div>
  <div id="choices"></div>
</div>

<script>
const scenario = {
  start: {
    text: "目を覚ます。\n潮の音が、やさしく耳に届く。",
    choices: [
      { label: "起き上がる", next: "stand" },
      { label: "もう少し寝る", next: "sleep" }
    ]
  },
  stand: {
    text: "砂浜だ。\n白い砂が、朝の光にきらきら輝いている。\n遠くに椰子の木が見える。",
    choices: [
      { label: "海を見る", next: "sea" },
      { label: "森に向かう", next: "forest" }
    ]
  },
  sleep: {
    text: "もう少し眠った。\n波の音を子守唄にして……。\n気づくと、空は茜色に染まっていた。",
    choices: [
      { label: "起き上がる", next: "stand" }
    ]
  },
  sea: {
    text: "水平線がどこまでも続いている。\n潮風が頬を撫でる。\nなんだか、懐かしい気持ちになった。\n\n――この島の物語は、まだはじまったばかり。",
    choices: []
  },
  forest: {
    text: "木漏れ日の中を歩く。\n小鳥のさえずりが、あちこちから聞こえる。\n奥の方に、小さな小屋が見えた。\n\n――この島の物語は、まだはじまったばかり。",
    choices: []
  }
};

const textDisplay = document.getElementById("text-display");
const choicesEl  = document.getElementById("choices");
const overlay    = document.getElementById("overlay");

const BASE_DELAY   = 45;
const PUNCT_DELAYS = { "。": 90, "、": 70, "！": 90, "？": 90, "…": 60, "―": 30 };

let typingId = null;
let isTyping = false;
let currentFullText = "";

function typeText(text) {
  return new Promise((resolve) => {
    currentFullText = text;
    textDisplay.innerHTML = "";
    isTyping = true;
    let i = 0;

    function tick() {
      if (i < text.length) {
        const ch = text[i];
        if (ch === "\n") {
          textDisplay.innerHTML += "<br>";
        } else {
          textDisplay.innerHTML += ch;
        }
        i++;
        const extra = PUNCT_DELAYS[ch] || 0;
        typingId = setTimeout(tick, BASE_DELAY + extra);
      } else {
        isTyping = false;
        addCursor();
        resolve();
      }
    }

    tick();
  });
}

function skipTyping() {
  if (!isTyping) return;
  clearTimeout(typingId);
  isTyping = false;
  let html = "";
  for (const ch of currentFullText) {
    html += ch === "\n" ? "<br>" : ch;
  }
  textDisplay.innerHTML = html;
  addCursor();
}

function addCursor() {
  textDisplay.innerHTML += '<span class="cursor"></span>';
}

document.getElementById("message-window").addEventListener("click", () => {
  if (isTyping) skipTyping();
});

function showChoices(choices) {
  choicesEl.innerHTML = "";
  if (choices.length === 0) return;

  choices.forEach((c) => {
    const btn = document.createElement("button");
    btn.className = "choice-btn";
    btn.textContent = c.label;
    btn.addEventListener("click", () => onChoiceSelect(btn, c.next));
    choicesEl.appendChild(btn);
  });
}

function onChoiceSelect(btn, nextId) {
  if (isTyping) {
    skipTyping();
  }

  choicesEl.querySelectorAll(".choice-btn").forEach((b) => {
    b.style.pointerEvents = "none";
  });

  btn.classList.add("pop");

  setTimeout(() => {
    transitionTo(nextId);
  }, 550);
}

async function transitionTo(sceneId) {
  overlay.classList.add("active");
  await wait(420);

  textDisplay.innerHTML = "";
  choicesEl.innerHTML = "";

  const scene = scenario[sceneId];
  if (!scene) return;

  overlay.classList.remove("active");
  await wait(350);

  await typeText(scene.text);
  showChoices(scene.choices);
}

function wait(ms) {
  return new Promise((r) => setTimeout(r, ms));
}

(async () => {
  await wait(300);
  const first = scenario["start"];
  await typeText(first.text);
  showChoices(first.choices);
})();
</script>
</body>
</html>
```
