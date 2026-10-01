<!DOCTYPE html>
<html lang="ja">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no">
<title>富士ストライク — Fuji Strike</title>
<style>
  * { margin: 0; padding: 0; box-sizing: border-box; user-select: none; -webkit-tap-highlight-color: transparent; }
  html, body {
    width: 100%; height: 100%;
    background: radial-gradient(circle at 50% 30%, #1a0a2e, #050208);
    font-family: 'Yu Mincho', 'Hiragino Mincho ProN', 'Noto Serif JP', serif;
    color: #fff; overflow: hidden; touch-action: manipulation;
  }
  #game {
    position: relative; width: 100vw; height: 100vh;
    max-width: 500px; max-height: 900px; margin: 0 auto;
    overflow: hidden; display: flex; flex-direction: column;
  }
  canvas { display: block; width: 100%; height: auto; touch-action: none; }
  #topBar {
    position: absolute; top: 6px; left: 6px; right: 6px;
    display: flex; justify-content: space-between;
    align-items: flex-start; pointer-events: none; z-index: 5;
    flex-wrap: wrap; gap: 4px;
  }
  #stats {
    background: rgba(0,0,0,0.75); border: 2px solid #d4a017;
    border-radius: 10px; padding: 4px 8px; font-size: 10px;
    font-weight: bold; color: #ffe0f0; display: flex; gap: 6px;
  }
  .currencies { display: flex; gap: 4px; flex-wrap: wrap; }
  .currency {
    background: rgba(0,0,0,0.75); border: 2px solid #ffd966;
    border-radius: 10px; padding: 3px 7px; font-size: 10px;
    font-weight: bold; color: #ffd966;
  }
  .currency.gem { border-color: #7ee0ff; color: #7ee0ff; }
  .currency.energy { border-color: #ff6ba6; color: #ff6ba6; }
  #sideButtons {
    position: absolute; right: 6px; top: 48px;
    display: flex; flex-direction: column; gap: 3px; z-index: 6;
    max-height: calc(100% - 130px); overflow-y: auto;
  }
  .side-btn {
    background: linear-gradient(#d4a017, #a06010);
    border: 2px solid #ffd966; border-radius: 10px;
    color: #fff; font-family: inherit; font-size: 10px;
    font-weight: bold; padding: 5px 6px; cursor: pointer;
    box-shadow: 0 2px 0 #604010; min-width: 62px;
    text-align: center; position: relative; min-height: 30px;
  }
  .side-btn:active { transform: translateY(1px); }
  .side-btn.demon { background: linear-gradient(#8e44ad, #4a1060); border-color: #ff6bff; }
  .side-btn.gacha { background: linear-gradient(#ff9e20, #c06010); }
  .side-btn.daily { background: linear-gradient(#c9a0ff, #6a2090); border-color: #ff9eff; }
  .side-btn.album { background: linear-gradient(#ff6b9e, #8e2050); }
  .side-btn.achievement { background: linear-gradient(#ffd966, #c09010); color: #1a0f0a; }
  .side-btn.training { background: linear-gradient(#7ee0ff, #2070c0); }
  .side-btn.event { background: linear-gradient(#ff9e20, #c04010); }
  .side-btn.auto { background: linear-gradient(#27ae60, #1a6030); }
  .side-btn.auto.on { background: linear-gradient(#ff4a4a, #a02020); animation: pulse 1s infinite alternate; }
  @keyframes pulse { from { box-shadow: 0 0 5px #ff4a4a; } to { box-shadow: 0 0 20px #ff4a4a; } }
  .side-btn .badge {
    position: absolute; top: -5px; right: -5px;
    background: #ff4a4a; color: #fff; border-radius: 10px;
    padding: 1px 5px; font-size: 9px;
    border: 2px solid #fff; font-weight: bold;
  }
  #touchControls {
    position: absolute; bottom: 10px; left: 0; right: 0;
    display: none; justify-content: space-between;
    padding: 0 12px; z-index: 7; pointer-events: none;
  }
  #touchControls.visible { display: flex; }
  .touch-group { display: flex; gap: 8px; pointer-events: auto; }
  .touch-btn {
    width: 58px; height: 58px; border-radius: 50%;
    background: rgba(212, 160, 23, 0.75);
    border: 3px solid #ffd966; color: #fff;
    font-size: 22px; font-weight: bold;
    display: flex; align-items: center; justify-content: center;
    cursor: pointer; touch-action: none;
  }
  .touch-btn:active { transform: scale(0.9); }
  .touch-btn.attack { background: rgba(255, 107, 158, 0.75); border-color: #ff9ec7; }
  .touch-btn.block { background: rgba(126, 224, 255, 0.75); border-color: #7ee0ff; }
  .touch-btn.special { background: rgba(255, 217, 102, 0.9); border-color: #ffd966; color: #1a0f0a; }
  .touch-btn.special:disabled { opacity: 0.3; }
  #quickFightBtn {
    position: absolute; bottom: 90px; left: 50%;
    transform: translateX(-50%);
    background: linear-gradient(#ff6b9e, #c0392b);
    border: 3px solid #ff9ec7; border-radius: 50px;
    padding: 14px 40px; font-size: 16px; font-family: inherit;
    font-weight: bold; color: #fff; cursor: pointer;
    box-shadow: 0 6px 0 #7a1010, 0 0 30px rgba(255,107,158,0.6);
    z-index: 8; text-shadow: 2px 2px 0 rgba(0,0,0,0.5);
  }
  .overlay {
    position: absolute; inset: 0; background: rgba(5, 2, 10, 0.97);
    display: flex; flex-direction: column; justify-content: flex-start;
    align-items: center; text-align: center; padding: 16px 12px;
    z-index: 10; overflow-y: auto;
  }
  .overlay h1 { font-size: 22px; color: #d4a017; text-shadow: 0 0 20px #d4a017; margin-bottom: 4px; }
  .overlay h2 { font-size: 12px; color: #ff6b9e; margin-bottom: 8px; }
  .overlay p { font-size: 11px; color: #ffe0f0; margin-bottom: 8px; line-height: 1.6; max-width: 95%; }
  .btn {
    background: linear-gradient(#d4a017, #a06010);
    color: #fff; border: 2px solid #ffd966;
    border-radius: 22px; padding: 10px 24px;
    font-size: 13px; font-family: inherit;
    font-weight: bold; cursor: pointer;
    box-shadow: 0 4px 0 #604010; margin: 4px; min-height: 42px;
  }
  .btn:active { transform: translateY(2px); }
  .btn.purple { background: linear-gradient(#8e44ad, #4a1060); border-color: #ff6bff; }
  .btn.green { background: linear-gradient(#27ae60, #1a6030); }
  .btn.pink { background: linear-gradient(#ff6b9e, #c0392b); }
  .btn.blue { background: linear-gradient(#4aa0ff, #2060a0); }
  .hidden { display: none !important; }
  .progress-bar {
    width: 95%; max-width: 320px; height: 14px;
    background: rgba(0,0,0,0.7); border: 2px solid #ffd966;
    border-radius: 8px; overflow: hidden; margin: 4px 0; position: relative;
  }
  .progress-bar .fill {
    height: 100%; background: linear-gradient(90deg, #27ae60, #ffd966, #ff6b9e);
    width: 0%; transition: width 0.4s;
  }
  .progress-bar .label {
    position: absolute; inset: 0; display: flex;
    justify-content: center; align-items: center;
    font-size: 9px; font-weight: bold; color: #fff;
  }
  .demon-grid, .roll-area {
    display: flex; gap: 6px; margin: 6px 0;
    flex-wrap: wrap; justify-content: center; max-width: 100%;
  }
  .demon-card {
    width: 46%; max-width: 155px; padding: 6px;
    background: linear-gradient(#2a0a2a, #100510);
    border: 2px solid #8e44ad; border-radius: 12px;
    cursor: pointer; font-size: 9px; font-weight: bold;
    position: relative;
  }
  .demon-card.today { border-color: #ffd966; box-shadow: 0 0 15px #ffd966; }
  .demon-card .demon-emoji { font-size: 26px; }
  .demon-card .demon-name-jp { font-size: 12px; color: #ff6bff; }
  .demon-card .demon-day { font-size: 9px; color: #ffd966; background: rgba(0,0,0,0.6); border-radius: 6px; padding: 1px 6px; display: inline-block; margin: 2px 0; }
  .demon-card .today-badge {
    position: absolute; top: -6px; right: -6px;
    background: #ffd966; color: #1a0f0a;
    border-radius: 8px; padding: 1px 6px;
    font-size: 8px; border: 2px solid #fff;
  }
  .demon-card .defeated-badge {
    position: absolute; top: -6px; left: -6px;
    background: #27ae60; color: #fff;
    border-radius: 50%; width: 20px; height: 20px;
    display: flex; align-items: center; justify-content: center;
    font-size: 11px; border: 2px solid #fff;
  }
  .card {
    width: 76px; height: 108px; border-radius: 10px;
    border: 2px solid #fff; background: linear-gradient(#2a1a3a, #100510);
    display: flex; flex-direction: column;
    align-items: center; justify-content: center;
    font-size: 9px; font-weight: bold;
    padding: 4px; text-align: center; animation: pop 0.4s;
  }
  @keyframes pop {
    0% { transform: scale(0.3) rotate(-10deg); opacity: 0; }
    100% { transform: scale(1) rotate(0); opacity: 1; }
  }
  .card .emoji { font-size: 26px; }
  .card .name-jp { font-size: 9px; color: #ffd966; }
  .card .stars { font-size: 8px; color: #ffd966; }
  .card.rare-1 { border-color: #9a9a9a; }
  .card.rare-2 { border-color: #27ae60; }
  .card.rare-3 { border-color: #4aa0ff; }
  .card.rare-4 { border-color: #8e44ad; }
  .card.rare-5 { border-color: #ffd966; box-shadow: 0 0 15px #ffd966; }
  .album-grid {
    display: grid; grid-template-columns: repeat(3, 1fr);
    gap: 6px; justify-content: center;
    max-width: 100%; margin: 8px 0; width: 95%;
  }
  .album-slot {
    aspect-ratio: 0.72; border-radius: 10px;
    border: 2px dashed #666; background: rgba(20, 10, 30, 0.5);
    display: flex; flex-direction: column;
    align-items: center; justify-content: center;
    font-size: 20px; color: #444;
  }
  .album-slot.filled {
    border: 2px solid #ffd966;
    background: linear-gradient(#2a1a3a, #100510); color: #fff;
  }
  .album-slot .name { font-size: 8px; color: #ffd966; margin-top: 2px; }
  .streak-bar { display: flex; gap: 2px; justify-content: center; margin: 6px 0; flex-wrap: wrap; }
  .streak-day {
    width: 40px; height: 48px; border-radius: 8px;
    background: linear-gradient(#2a1a3a, #100510);
    border: 2px solid #444; display: flex; flex-direction: column;
    align-items: center; justify-content: center;
    font-size: 7px; font-weight: bold; color: #666;
  }
  .streak-day.active { border-color: #27ae60; color: #6bff9e; }
  .streak-day.today { border-color: #ffd966; color: #ffd966; }
  .streak-day.claimed { border-color: #27ae60; background: linear-gradient(#1a4028, #0a2010); color: #6bff9e; }
  .streak-day .day-emoji { font-size: 14px; }
  .mission-list {
    display: flex; flex-direction: column;
    gap: 5px; max-width: 100%; width: 95%; margin: 6px 0;
  }
  .mission {
    background: linear-gradient(#2a1a3a, #100510);
    border: 2px solid #8e44ad; border-radius: 10px;
    padding: 6px 10px; font-size: 10px; font-weight: bold;
    color: #ffe0f0; display: flex;
    justify-content: space-between; align-items: center; gap: 6px;
  }
  .mission.done { border-color: #27ae60; color: #6bff9e; }
  .mission button {
    background: #ffd966; color: #1a0f0a;
    border: none; border-radius: 6px;
    padding: 4px 10px; font-size: 10px;
    font-weight: bold; cursor: pointer; min-height: 26px;
  }
  .time-slot {
    background: linear-gradient(#2a1a3a, #100510);
    border: 2px solid #7ee0ff; border-radius: 10px;
    padding: 6px 12px; font-size: 10px; color: #7ee0ff;
    cursor: pointer; display: inline-block; margin: 3px; min-height: 34px;
  }
  .time-slot.claimed { border-color: #444; color: #666; cursor: not-allowed; }
  #toast {
    position: absolute; top: 60px; left: 50%;
    transform: translateX(-50%); background: rgba(0,0,0,0.95);
    border: 2px solid #d4a017; border-radius: 12px;
    padding: 8px 16px; font-size: 11px; font-weight: bold;
    color: #ffd966; z-index: 20; opacity: 0;
    transition: 0.3s; pointer-events: none;
    max-width: 90%; text-align: center;
  }
  #toast.show { opacity: 1; transform: translateX(-50%) translateY(6px); }
  .demon-hp-bar {
    position: absolute; top: 50px; left: 50%;
    transform: translateX(-50%); width: 80%; max-width: 380px;
    height: 18px; background: rgba(0,0,0,0.8);
    border: 2px solid #ff6bff; border-radius: 10px;
    z-index: 6; overflow: hidden;
  }
  .demon-hp-bar .fill {
    height: 100%; background: linear-gradient(#ff6bff, #8e44ad);
    width: 100%; transition: width 0.2s;
  }
  .demon-hp-bar .label {
    position: absolute; inset: 0; display: flex;
    justify-content: center; align-items: center;
    font-size: 10px; font-weight: bold; color: #fff;
  }
  #shareBtn {
    position: absolute; top: 90px; left: 6px;
    background: linear-gradient(#27ae60, #1a6030);
    border: 2px solid #6bff9e; border-radius: 12px;
    padding: 6px 10px; font-size: 11px; font-weight: bold;
    color: #fff; cursor: pointer; z-index: 6;
    box-shadow: 0 3px 0 #0a3018;
  }
  #musicBtn {
    position: absolute; top: 130px; left: 6px;
    background: linear-gradient(#8e44ad, #4a1060);
    border: 2px solid #ff6bff; border-radius: 12px;
    padding: 6px 10px; font-size: 14px; font-weight: bold;
    color: #fff; cursor: pointer; z-index: 6;
    box-shadow: 0 3px 0 #2a0840;
  }
  #langBtn {
    position: absolute; top: 50px; left: 6px;
    background: linear-gradient(#4aa0ff, #2060a0);
    border: 2px solid #7ee0ff; border-radius: 12px;
    padding: 6px 10px; font-size: 12px; font-weight: bold;
    color: #fff; cursor: pointer; z-index: 6;
    box-shadow: 0 3px 0 #103060;
  }
</style>
</head>
<body><div id="game">
  <canvas id="canvas" width="500" height="500"></canvas>
  <div id="topBar">
    <div id="stats">
      <span>💗 <span id="hp">100</span></span>
      <span>⚔️ <span id="kills">0</span>/<span id="maxKills">6</span></span>
      <span>🔥 <span id="streakHud">0</span></span>
    </div>
    <div class="currencies">
      <div class="currency">🪙 <span id="coins">0</span></div>
      <div class="currency gem">💎 <span id="gems">100</span></div>
      <div class="currency energy">⚡ <span id="energy">5</span></div>
    </div>
  </div>
  <div id="demonHpBar" class="demon-hp-bar hidden">
    <div class="fill" id="demonHpFill"></div>
    <div class="label" id="demonHpLabel">👹 鬼</div>
  </div>
  <button id="langBtn" onclick="toggleLang()">🌐 JP</button>
  <button id="shareBtn" class="hidden" onclick="shareResult()">📤 シェア</button>
  <button id="musicBtn" class="hidden" onclick="toggleMusic()">🔊</button>
  <div id="sideButtons" class="hidden">
    <button class="side-btn demon" onclick="openDemons()">👹 鬼</button>
    <button class="side-btn daily" onclick="openDaily()">📅 日課</button>
    <button class="side-btn gacha" onclick="openGacha()">🎰 引</button>
    <button class="side-btn album" onclick="openAlbum()">🎴 図鑑</button>
    <button class="side-btn achievement" onclick="openAchievements()">🏆 実績</button>
    <button class="side-btn training" onclick="openTraining()">🥋 道場</button>
    <button class="side-btn event" onclick="openEvent()">🎋 祭</button>
    <button class="side-btn auto" id="autoBtn" onclick="toggleAutoBattle()">🤖 自動</button>
  </div>
  <div id="touchControls">
    <div class="touch-group">
      <div class="touch-btn" id="btnLeft">◀</div>
      <div class="touch-btn" id="btnRight">▶</div>
    </div>
    <div class="touch-group">
      <div class="touch-btn block" id="btnBlock">🛡</div>
      <div class="touch-btn special" id="btnSpecial" disabled>必</div>
      <div class="touch-btn attack" id="btnAttack">⚔</div>
    </div>
  </div>
  <button id="quickFightBtn" class="hidden" onclick="quickFight()">⚡ 今すぐ戦う</button>
  <div id="level-info" class="hidden">レベル 1</div>
  <div id="startOverlay" class="overlay">
    <h1 id="titleTxt">富士ストライク</h1>
    <h2 id="subtitleTxt">🌸 曜日の鬼 🌸</h2>
    <p id="startText">富士山の麓に、曜日の鬼たちが現れた。<br>毎日ログインして、七人の鬼を打ち倒せ！</p>
    <button class="btn" onclick="startGame()" id="startBtn">▶ 開始</button>
    <button class="btn green" onclick="startGame(true)" id="quickBtn">⚡ クイック</button>
    <button class="btn pink" onclick="openQuizFromStart()" id="quizBtn">🎯 今日のクイズ</button>
  </div>
  <div id="gachaOverlay" class="overlay hidden">
    <h1 id="gachaTitle">🎰 引</h1>
    <div class="progress-bar">
      <div class="fill" id="pityFill"></div>
      <div class="label" id="pityLabel">天井: 0/20</div>
    </div>
    <div class="roll-area" id="gachaResults"></div>
    <div>
      <button class="btn green" onclick="rollGacha(1, true)" id="freeGachaBtn">🎁 無料</button>
      <button class="btn" onclick="rollGacha(1)">🎲 一回 💎20</button>
      <button class="btn purple" onclick="rollGacha(10)">🎰 十回 💎180</button>
    </div>
    <button class="btn" onclick="closeOverlay('gachaOverlay')">閉じる</button>
  </div>
  <div id="demonsOverlay" class="overlay hidden">
    <h1 id="demonsTitle">👹 曜日の鬼</h1>
    <div class="demon-grid" id="demonGrid"></div>
    <button class="btn" onclick="closeOverlay('demonsOverlay')">閉じる</button>
  </div>
  <div id="dailyOverlay" class="overlay hidden">
    <h1 id="dailyTitle">📅 日課</h1>
    <div class="streak-bar" id="streakBar"></div>
    <div>
      <div class="time-slot" id="morningSlot" onclick="claimTimeBonus('morning')">🌅 朝 ⚡+1</div>
      <div class="time-slot" id="eveningSlot" onclick="claimTimeBonus('evening')">🌙 夜 🪙+200</div>
    </div>
    <button class="btn" onclick="closeOverlay('dailyOverlay')">閉じる</button>
  </div>
  <div id="albumOverlay" class="overlay hidden">
    <h1 id="albumTitle">🎴 図鑑</h1>
    <div class="progress-bar">
      <div class="fill" id="albumFill"></div>
      <div class="label" id="albumLabel">0/10</div>
    </div>
    <div class="album-grid" id="albumGrid"></div>
    <button class="btn" onclick="closeOverlay('albumOverlay')">閉じる</button>
  </div>
  <div id="achievementsOverlay" class="overlay hidden">
    <h1 id="achTitle">🏆 実績</h1>
    <div class="mission-list" id="achList"></div>
    <button class="btn" onclick="closeOverlay('achievementsOverlay')">閉じる</button>
  </div>
  <div id="trainingOverlay" class="overlay hidden">
    <h1 id="trainingTitle">🥋 道場</h1>
    <p id="trainingText">自由に練習できます。</p>
    <button class="btn green" onclick="startTraining()">▶ 練習開始</button>
    <button class="btn" onclick="closeOverlay('trainingOverlay')">閉じる</button>
  </div>
  <div id="eventOverlay" class="overlay hidden">
    <h1 id="eventTitle">🎋 祭</h1>
    <div id="eventContent"></div>
    <button class="btn" onclick="closeOverlay('eventOverlay')">閉じる</button>
  </div>
  <div id="quizOverlay" class="overlay hidden">
    <h1 id="quizTitle">🎯 今日のクイズ</h1>
    <h2 id="quizSubtitle">今日の鬼はどれ？</h2>
    <div class="quiz-options" id="quizOptions"></div>
    <button class="btn" onclick="closeOverlay('quizOverlay')">閉じる</button>
  </div>
  <div id="loginGiftOverlay" class="overlay hidden">
    <h1 id="giftTitle">🎁 ログインボーナス</h1>
    <div class="login-gift">
      <div style="font-size:70px;">🎁</div>
      <div style="font-size:18px;color:#ffd966;font-weight:bold;margin:10px 0;" id="loginGiftText">+100 🪙</div>
    </div>
    <button class="btn green" onclick="claimLoginGift()">受取</button>
  </div>
  <div id="levelOverlay" class="overlay hidden"></div>
  <div id="toast"></div>
</div>
<script>// ============================================
// ЯЗЫКИ
// ============================================
const LANGS = ['jp', 'ru', 'en'];
let currentLang = localStorage.getItem('fuji_lang') || 'jp';

const TEXTS = {
  jp: {
    title: '富士ストライク', subtitle: '🌸 曜日の鬼 🌸',
    startText: '富士山の麓に、曜日の鬼たちが現れた。<br>毎日ログインして、七人の鬼を打ち倒せ！',
    start: '▶ 開始', quick: '⚡ クイック', quiz: '🎯 今日のクイズ',
    gacha: '🎰 引', demons: '👹 曜日の鬼', daily: '📅 日課',
    album: '🎴 図鑑', achievements: '🏆 実績', training: '🥋 道場',
    event: '🎋 祭', close: '閉じる', auto: '🤖 自動',
    days: ['月','火','水','木','金','土','日'],
    demonNames: ['月曜鬼','火曜鬼','水曜蛇','木曜頭','金曜鬼','土曜龍','日曜大魔王']
  },
  ru: {
    title: 'Фудзи Страйк', subtitle: '🌸 Демоны Дней 🌸',
    startText: 'У подножия Фудзи появились демоны дней недели.<br>Заходи каждый день и победи всех семерых!',
    start: '▶ Начать', quick: '⚡ Быстро', quiz: '🎯 Квиз дня',
    gacha: '🎰 Гача', demons: '👹 Демоны', daily: '📅 Ежедневка',
    album: '🎴 Альбом', achievements: '🏆 Достижения', training: '🥋 Тренировка',
    event: '🎋 Ивенты', close: 'Закрыть', auto: '🤖 Авто',
    days: ['Пн','Вт','Ср','Чт','Пт','Сб','Вс'],
    demonNames: ['Гэцуёби-Они','Каёби-Они','Суйёби-Хэби','Мокуёби-Гасира','Кинъёби-Они','Дойёби-Рю','Нитиёби-Даймао']
  },
  en: {
    title: 'Fuji Strike', subtitle: '🌸 Demons of Days 🌸',
    startText: 'Demons of the days appeared at Mount Fuji.<br>Log in daily and defeat all seven!',
    start: '▶ Start', quick: '⚡ Quick', quiz: '🎯 Daily Quiz',
    gacha: '🎰 Gacha', demons: '👹 Demons', daily: '📅 Daily',
    album: '🎴 Album', achievements: '🏆 Achievements', training: '🥋 Training',
    event: '🎋 Events', close: 'Close', auto: '🤖 Auto',
    days: ['Mon','Tue','Wed','Thu','Fri','Sat','Sun'],
    demonNames: ['Getsuyobi-Oni','Kayobi-Oni','Suiyobi-Hebi','Mokuyobi-Gashira','Kinyobi-Oni','Doyobi-Ryu','Nichiyobi-Daimao']
  }
};

function toggleLang() {
  const idx = LANGS.indexOf(currentLang);
  currentLang = LANGS[(idx + 1) % LANGS.length];
  localStorage.setItem('fuji_lang', currentLang);
  updateLang();
  document.getElementById('langBtn').textContent = '🌐 ' + currentLang.toUpperCase();
  showToast('🌐 ' + currentLang.toUpperCase());
}

function updateLang() {
  const t = TEXTS[currentLang];
  document.getElementById('titleTxt').textContent = t.title;
  document.getElementById('subtitleTxt').textContent = t.subtitle;
  document.getElementById('startText').innerHTML = t.startText;
  document.getElementById('startBtn').textContent = t.start;
  document.getElementById('quickBtn').textContent = t.quick;
  document.getElementById('quizBtn').textContent = t.quiz;
  document.getElementById('gachaTitle').textContent = t.gacha;
  document.getElementById('demonsTitle').textContent = t.demons;
  document.getElementById('dailyTitle').textContent = t.daily;
  document.getElementById('albumTitle').textContent = t.album;
  document.getElementById('achTitle').textContent = t.achievements;
  document.getElementById('trainingTitle').textContent = t.training;
  document.getElementById('eventTitle').textContent = t.event;
}

// ============================================
// КАНВАС
// ============================================
const canvas = document.getElementById('canvas');
const ctx = canvas.getContext('2d');
const W = canvas.width, H = canvas.height;
const GROUND_Y = H - 80;

let state = {
  coins: 0, gems: 100, energy: 5, maxEnergy: 5,
  hp: 100, maxHp: 100, kills: 0,
  ownedCards: {}, demonDefeated: {},
  gachaPity: 0, totalRolls: 0,
  freeGachaDate: null, streak: 0, lastLoginDate: null,
  morningClaimed: false, eveningClaimed: false,
  lastMorningDate: null, lastEveningDate: null,
  totalKills: 0, loginGiftDate: null, quizDate: null,
  autoBattle: false, combo: 0, maxCombo: 0, specialCharge: 0
};

const DEMONS = [
  { id:'monday', day:1, emoji:'💙', hp:80, dmg:8, speed:0.8, reward:{coins:500,gems:30,energy:1} },
  { id:'tuesday', day:2, emoji:'🔴', hp:70, dmg:10, speed:1.2, reward:{coins:300,gems:15} },
  { id:'wednesday', day:3, emoji:'🐍', hp:80, dmg:9, speed:1.5, reward:{coins:400,gems:20} },
  { id:'thursday', day:4, emoji:'💀', hp:100, dmg:11, speed:1.3, reward:{coins:500,gems:25} },
  { id:'friday', day:5, emoji:'🔥', hp:120, dmg:13, speed:1.6, reward:{coins:800,gems:50} },
  { id:'saturday', day:6, emoji:'🐉', hp:160, dmg:16, speed:1.5, reward:{coins:1500,gems:100} },
  { id:'sunday', day:0, emoji:'👑', hp:220, dmg:20, speed:1.8, reward:{coins:3000,gems:200} }
];

const GACHA_POOL = [
  { id:'s1', nameJp:'若侍', emoji:'🗡️', rarity:1 },
  { id:'s2', nameJp:'木刀', emoji:'🪵', rarity:1 },
  { id:'s3', nameJp:'侍見習', emoji:'⚔️', rarity:2 },
  { id:'s4', nameJp:'刀', emoji:'🗾', rarity:2 },
  { id:'s5', nameJp:'剣聖', emoji:'🥷', rarity:3 },
  { id:'s6', nameJp:'狐守', emoji:'🦊', rarity:4 },
  { id:'s7', nameJp:'富士霊', emoji:'🗻', rarity:5 },
  { id:'s8', nameJp:'天狗', emoji:'🦅', rarity:3 },
  { id:'s9', nameJp:'鬼武者', emoji:'👹', rarity:4 },
  { id:'s10', nameJp:'桜姫', emoji:'🌸', rarity:5 }
];
const RARITY_STARS = {1:'⭐',2:'⭐⭐',3:'⭐⭐⭐',4:'⭐⭐⭐⭐',5:'⭐⭐⭐⭐⭐'};

const LEVELS = {
  1:{ spawnDelay:100, hp:30, dmg:8, speed:1.2, maxKills:6 },
  2:{ spawnDelay:70, hp:50, dmg:12, speed:1.8, maxKills:10 },
  3:{ spawnDelay:50, hp:80, dmg:18, speed:2.4, maxKills:14 }
};

let keys = {};
let gameState = 'menu';
let animTime = 0;
let level = 1;
let enemies = [];
let particles = [];
let enemySpawnTimer = 0;
let trainingMode = false;
let currentDemon = null;

const player = {
  x:150, y:0, w:50, h:75, vx:0,
  speed:4, facing:1,
  attackTimer:0, blockTimer:0, cooldown:0,
  hitFlash:0, invuln:0
};

let stars = [];
for (let i=0;i<40;i++) stars.push({ x:Math.random()*W, y:Math.random()*250, size:1+Math.random()*2, phase:Math.random()*Math.PI*2 });

function saveProgress() { try { localStorage.setItem('fuji_v6', JSON.stringify(state)); } catch(e) {} }
function loadProgress() { try { const s = localStorage.getItem('fuji_v6'); if (s) Object.assign(state, JSON.parse(s)); } catch(e) {} }
loadProgress();
updateLang();
document.getElementById('langBtn').textContent = '🌐 ' + currentLang.toUpperCase();

// ЗВУК
let audioCtx = null;
function initAudio() {
  if (!audioCtx) { try { audioCtx = new (window.AudioContext || window.webkitAudioContext)(); } catch(e) {} }
}
function playTone(freq, dur, type='sine', vol=0.12) {
  if (!audioCtx) return;
  if (audioCtx.state === 'suspended') audioCtx.resume();
  try {
    const osc = audioCtx.createOscillator();
    const gain = audioCtx.createGain();
    osc.type = type; osc.frequency.value = freq;
    gain.gain.setValueAtTime(vol, audioCtx.currentTime);
    gain.gain.exponentialRampToValueAtTime(0.001, audioCtx.currentTime + dur);
    osc.connect(gain); gain.connect(audioCtx.destination);
    osc.start(); osc.stop(audioCtx.currentTime + dur);
  } catch(e) {}
}
function sSlash() { playTone(800, 0.08, 'sawtooth', 0.1); }
function sHit() { playTone(150, 0.12, 'square', 0.15); }
function sWin() { [523,659,784,1046].forEach((f,i)=>setTimeout(()=>playTone(f,0.2,'triangle',0.12),i*100)); }
function sGacha() { for(let i=0;i<6;i++) setTimeout(()=>playTone(600+i*100,0.05,'square',0.07),i*50); }
function sLegend() { [523,659,784,1046,1318].forEach((f,i)=>setTimeout(()=>playTone(f,0.25,'triangle',0.15),i*120)); }

// МУЗЫКА
let musicCtx = null;
let musicPlaying = false;
let musicMuted = false;
function initMusic() {
  if (!musicCtx) { try { musicCtx = new (window.AudioContext || window.webkitAudioContext)(); } catch(e) {} }
}
function playTaiko(time, freq = 80, vol = 0.3) {
  if (!musicCtx || musicMuted) return;
  try {
    const osc = musicCtx.createOscillator();
    const gain = musicCtx.createGain();
    osc.type = 'sine';
    osc.frequency.setValueAtTime(freq, time);
    osc.frequency.exponentialRampToValueAtTime(30, time + 0.3);
    gain.gain.setValueAtTime(vol, time);
    gain.gain.exponentialRampToValueAtTime(0.001, time + 0.4);
    osc.connect(gain); gain.connect(musicCtx.destination);
    osc.start(time); osc.stop(time + 0.4);
  } catch(e) {}
}
function playBattleMusic() {
  if (!musicCtx || musicPlaying) return;
  musicPlaying = true;
  const bpm = 100;
  const beat = 60 / bpm;
  let barTime = musicCtx.currentTime + 0.1;
  function playBar() {
    if (!musicPlaying) return;
    const t = barTime;
    playTaiko(t, 80, 0.35);
    playTaiko(t + beat, 90, 0.3);
    playTaiko(t + beat * 2, 85, 0.35);
    playTaiko(t + beat * 3, 75, 0.25);
    barTime += beat * 4;
    setTimeout(playBar, (barTime - musicCtx.currentTime - 0.5) * 1000);
  }
  playBar();
}
function stopMusic() { musicPlaying = false; }
function toggleMusic() {
  musicMuted = !musicMuted;
  if (musicMuted) stopMusic();
  else if (gameState === 'playing') playBattleMusic();
  document.getElementById('musicBtn').textContent = musicMuted ? '🔇' : '🔊';
}

// УПРАВЛЕНИЕ
document.addEventListener('keydown', e => {
  keys[e.code] = true;
  if (['ArrowLeft','ArrowRight','Space','ShiftLeft'].includes(e.code)) e.preventDefault();
  if (e.code === 'Space' && gameState === 'playing') playerAttack();
});
document.addEventListener('keyup', e => keys[e.code] = false);

function bindTouch(id, key, action) {
  const el = document.getElementById(id);
  if (!el) return;
  const start = e => { e.preventDefault(); keys[key] = true; if (action) action(); };
  const end = e => { e.preventDefault(); keys[key] = false; };
  el.addEventListener('touchstart', start);
  el.addEventListener('touchend', end);
  el.addEventListener('mousedown', start);
  el.addEventListener('mouseup', end);
  el.addEventListener('mouseleave', end);
}
bindTouch('btnLeft', 'ArrowLeft');
bindTouch('btnRight', 'ArrowRight');
bindTouch('btnAttack', 'Space', () => playerAttack());
bindTouch('btnBlock', 'ShiftLeft');
bindTouch('btnSpecial', 'KeyQ', () => useSpecial());

function startGame(quick = false) {
  initAudio(); initMusic(); playBattleMusic();
  document.getElementById('startOverlay').classList.add('hidden');
  document.getElementById('sideButtons').classList.remove('hidden');
  document.getElementById('level-info').classList.remove('hidden');
  document.getElementById('touchControls').classList.add('visible');
  document.getElementById('shareBtn').classList.remove('hidden');
  document.getElementById('musicBtn').classList.remove('hidden');
  checkDailyLogin();
  checkLoginGift();
  resetLevel(1);
  gameState = 'playing';
  if (quick) quickFight();
  showToast('戦いが始まる！');
}

function quickFight() { resetLevel(1); gameState = 'playing'; }

function resetLevel(n) {
  level = n;
  const cfg = LEVELS[n];
  enemies = []; particles = [];
  state.kills = 0;
  enemySpawnTimer = 30;
  state.hp = Math.min(state.maxHp, state.hp + 20);
  player.x = 150; player.vx = 0;
  updateHUD();
}

function updateHUD() {
  document.getElementById('hp').textContent = Math.max(0, Math.round(state.hp));
  document.getElementById('kills').textContent = state.kills;
  document.getElementById('maxKills').textContent = LEVELS[level]?.maxKills || 0;
  document.getElementById('coins').textContent = state.coins;
  document.getElementById('gems').textContent = state.gems;
  document.getElementById('energy').textContent = state.energy;
  document.getElementById('streakHud').textContent = state.streak;
}

function playerAttack() {
  if (player.cooldown > 0) return;
  player.attackTimer = 14; player.cooldown = 22;
  sSlash();
  const range = 80;
  let hit = false;
  enemies.forEach(e => {
    if (e.dead) return;
    const ex = e.x + e.w / 2;
    const px = player.x + player.w / 2;
    if (Math.abs(ex - px) < range + e.w/2 && Math.sign(ex - px) === player.facing) {
      e.hp -= 20 + Math.random() * 10;
      e.hitFlash = 8; sHit();
      spawnParticles(ex, GROUND_Y - e.h/2, '#ff6ba6', 8);
      hit = true;
      if (e.hp <= 0) {
        e.dead = true;
        state.kills++; state.totalKills++;
        state.coins += 5 + level * 3;
        updateHUD();
        spawnParticles(ex, GROUND_Y - e.h/2, '#ffd966', 15);
      }
    }
  });
  if (hit) { state.combo++; state.specialCharge = Math.min(100, state.specialCharge + 10); if (state.combo > state.maxCombo) state.maxCombo = state.combo; }
  else state.combo = 0;
  updateSpecialButton();
}

function updateSpecialButton() {
  const btn = document.getElementById('btnSpecial');
  if (btn) btn.disabled = state.specialCharge < 100;
}

function useSpecial() {
  if (state.specialCharge < 100) return;
  state.specialCharge = 0;
  enemies.forEach(e => {
    if (e.dead) return;
    e.hp -= 100; e.hitFlash = 15;
    if (e.hp <= 0) { e.dead = true; state.kills++; state.totalKills++; state.coins += 10; }
  });
  updateHUD(); updateSpecialButton();
}

function spawnParticles(x, y, color, count) {
  for (let i=0;i<count;i++) particles.push({ x, y, vx:(Math.random()-0.5)*8, vy:(Math.random()-0.5)*8-2, life:35, maxLife:35, color, size:3+Math.random()*3 });
}

function spawnEnemy() {
  const cfg = LEVELS[level];
  const side = Math.random()<0.5?-1:1;
  const x = side===1?W+20:-50;
  enemies.push({ x, y:GROUND_Y-60, w:44, h:60, hp:cfg.hp, maxHp:cfg.hp, vx:side*-cfg.speed*(0.8+Math.random()*0.5), dmg:cfg.dmg, attackTimer:0, hitFlash:0, dead:false, bobPhase:Math.random()*Math.PI*2 });
}

function toggleAutoBattle() {
  state.autoBattle = !state.autoBattle;
  const t = TEXTS[currentLang];
  document.getElementById('autoBtn').textContent = state.autoBattle ? '🤖 ON' : t.auto;
  document.getElementById('autoBtn').classList.toggle('on', state.autoBattle);
  saveProgress();
}

function autoBattleUpdate() {
  if (!state.autoBattle || gameState !== 'playing') return;
  const nearest = enemies.find(e => !e.dead);
  if (nearest) {
    const px = player.x + player.w/2;
    const ex = nearest.x + nearest.w/2;
    if (Math.abs(px - ex) > 60) {
      if (ex > px) { player.vx = player.speed; player.facing = 1; }
      else { player.vx = -player.speed; player.facing = -1; }
      player.x += player.vx;
      player.x = Math.max(20, Math.min(W-player.w-20, player.x));
    } else if (player.cooldown <= 0) playerAttack();
  }
}

function update() {
  animTime++;
  if (gameState !== 'playing') return;
  autoBattleUpdate();
  if (!state.autoBattle) {
    player.vx = 0;
    if (keys['ArrowLeft']) { player.vx = -player.speed; player.facing = -1; }
    if (keys['ArrowRight']) { player.vx = player.speed; player.facing = 1; }
    player.x += player.vx;
    player.x = Math.max(20, Math.min(W-player.w-20, player.x));
  }
  if (player.attackTimer>0) player.attackTimer--;
  if (player.cooldown>0) player.cooldown--;
  if (player.hitFlash>0) player.hitFlash--;
  if (player.invuln>0) player.invuln--;
  if (keys['ShiftLeft']) player.blockTimer = 5;
  else player.blockTimer = Math.max(0, player.blockTimer-1);
  if (!trainingMode) {
    const cfg = LEVELS[level];
    enemySpawnTimer--;
    if (enemySpawnTimer<=0 && state.kills<cfg.maxKills) { spawnEnemy(); enemySpawnTimer = cfg.spawnDelay; }
  }
  enemies.forEach(e => {
    if (e.dead) return;
    e.x += e.vx;
    if (e.hitFlash>0) e.hitFlash--;
    const px = player.x + player.w/2;
    const ex = e.x + e.w/2;
    if (Math.abs(px-ex) < 50 && e.attackTimer<=0) {
      e.attackTimer = 60;
      let dmg = e.dmg;
      if (player.blockTimer>0) dmg = Math.floor(dmg*0.3);
      if (player.invuln<=0) {
        state.hp -= dmg; player.hitFlash = 8; player.invuln = 30; state.combo = 0; sHit();
        spawnParticles(px, GROUND_Y-40, '#ff4a4a', 8); updateHUD();
        if (state.hp<=0) { state.hp = 0; gameOver(); }
      }
    }
    if (e.attackTimer>0) e.attackTimer--;
    if (e.x<-100 || e.x>W+100) e.dead = true;
  });
  enemies = enemies.filter(e => !e.dead);
  particles.forEach(p => { p.x += p.vx; p.y += p.vy; p.vy += 0.3; p.life--; });
  particles = particles.filter(p => p.life>0);
  if (currentDemon && enemies.length === 0) defeatDemon();
  if (!trainingMode && !currentDemon && state.kills >= LEVELS[level].maxKills && enemies.length === 0) {
    if (level<3) levelUp(); else winGame();
  }
}

function startDemonFight(d) {
  closeOverlay('demonsOverlay');
  currentDemon = d;
  enemies = [{ x: W-120, y: GROUND_Y-60, w: 44, h: 60, hp: d.hp, maxHp: d.hp, vx: -0.5, dmg: d.dmg, attackTimer: 0, hitFlash: 0, dead: false, bobPhase: 0 }];
  document.getElementById('demonHpBar').classList.remove('hidden');
  document.getElementById('demonHpLabel').textContent = d.emoji + ' ' + TEXTS[currentLang].demonNames[DEMONS.indexOf(d)];
  gameState = 'playing';
}

function defeatDemon() {
  if (!currentDemon) return;
  sWin();
  const d = currentDemon;
  state.coins += d.reward.coins || 0;
  state.gems += d.reward.gems || 0;
  if (d.reward.energy) state.energy = Math.min(state.maxEnergy, state.energy + d.reward.energy);
  state.demonDefeated[d.id] = new Date().toDateString();
  currentDemon = null;
  document.getElementById('demonHpBar').classList.add('hidden');
  updateHUD(); saveProgress();
  const ov = document.getElementById('levelOverlay');
  ov.classList.remove('hidden');
  ov.innerHTML = `<h1>🎉 撃破！</h1><p>🪙 +${d.reward.coins||0} · 💎 +${d.reward.gems||0}</p><button class="btn" onclick="closeLevelOverlay()">OK</button>`;
}

function drawBackground() {
  const grad = ctx.createLinearGradient(0,0,0,H);
  grad.addColorStop(0, '#1a0a2e'); grad.addColorStop(0.6, '#3a1a5e'); grad.addColorStop(1, '#6a2a5e');
  ctx.fillStyle = grad; ctx.fillRect(0,0,W,H);
  stars.forEach((s,i) => {
    const tw = 0.3 + Math.sin(animTime*0.05 + s.phase)*0.5 + 0.5;
    ctx.globalAlpha = tw;
    ctx.fillStyle = i%5===0?'#ffd966':'#fff';
    ctx.beginPath(); ctx.arc(s.x, s.y, s.size, 0, Math.PI*2); ctx.fill();
  });
  ctx.globalAlpha = 1;
  ctx.fillStyle = 'rgba(255,240,200,0.9)';
  ctx.beginPath(); ctx.arc(400, 80, 35, 0, Math.PI*2); ctx.fill();
  ctx.fillStyle = '#1a0a2e';
  ctx.beginPath(); ctx.arc(388, 70, 32, 0, Math.PI*2); ctx.fill();
  const mGrad = ctx.createLinearGradient(0, 100, 0, GROUND_Y);
  mGrad.addColorStop(0, '#9a7ac0'); mGrad.addColorStop(0.5, '#5a3a7e'); mGrad.addColorStop(1, '#2a1a4e');
  ctx.fillStyle = mGrad;
  ctx.beginPath();
  ctx.moveTo(0, GROUND_Y); ctx.lineTo(W/2 - 80, 140); ctx.lineTo(W/2, 100); ctx.lineTo(W/2 + 80, 140); ctx.lineTo(W, GROUND_Y);
  ctx.closePath(); ctx.fill();
  ctx.fillStyle = '#fff';
  ctx.beginPath();
  ctx.moveTo(W/2-60, 160); ctx.lineTo(W/2-30, 150); ctx.lineTo(W/2-15, 160); ctx.lineTo(W/2, 100);
  ctx.lineTo(W/2+15, 160); ctx.lineTo(W/2+30, 150); ctx.lineTo(W/2+60, 160); ctx.lineTo(W/2+40, 180);
  ctx.quadraticCurveTo(W/2, 160, W/2-40, 180); ctx.closePath(); ctx.fill();
  ctx.fillStyle = '#1a0a2a'; ctx.fillRect(0, GROUND_Y, W, H-GROUND_Y);
  ctx.fillStyle = '#3a2a5a'; ctx.fillRect(0, GROUND_Y, W, 4);
}

function drawSamurai(x, y) {
  const facing = player.facing;
  const bob = Math.sin(animTime*0.1)*2;
  const cx = x + player.w/2;
  const baseY = y + bob;
  ctx.fillStyle = 'rgba(0,0,0,0.4)';
  ctx.beginPath(); ctx.ellipse(cx, GROUND_Y+5, 26, 7, 0, 0, Math.PI*2); ctx.fill();
  ctx.fillStyle = player.hitFlash>0?'#fff':'#2c4a80';
  ctx.beginPath();
  ctx.moveTo(cx-18, baseY+50); ctx.lineTo(cx+18, baseY+50); ctx.lineTo(cx+22, baseY+72); ctx.lineTo(cx-22, baseY+72);
  ctx.closePath(); ctx.fill();
  ctx.fillStyle = '#d4a017'; ctx.fillRect(cx-20, baseY+58, 40, 6);
  const headY = baseY + 22;
  ctx.fillStyle = player.hitFlash>0?'#fff':'#ffdcc0';
  ctx.beginPath(); ctx.arc(cx, headY, 24, 0, Math.PI*2); ctx.fill();
  ctx.fillStyle = '#0a0a1a';
  ctx.beginPath(); ctx.arc(cx, headY-4, 24, Math.PI, 0); ctx.fill();
  ctx.beginPath();
  ctx.moveTo(cx-24, headY-4); ctx.lineTo(cx-12, headY+4); ctx.lineTo(cx-4, headY-6); ctx.lineTo(cx+4, headY+4);
  ctx.lineTo(cx+12, headY-6); ctx.lineTo(cx+24, headY-4); ctx.lineTo(cx+24, headY-10); ctx.lineTo(cx-24, headY-10);
  ctx.closePath(); ctx.fill();
  const eyeY = headY + 4;
  ctx.fillStyle = '#fff';
  ctx.beginPath(); ctx.arc(cx-9, eyeY, 6, 0, Math.PI*2); ctx.arc(cx+9, eyeY, 6, 0, Math.PI*2); ctx.fill();
  ctx.fillStyle = '#000';
  ctx.beginPath(); ctx.arc(cx-9+facing, eyeY, 2.5, 0, Math.PI*2); ctx.arc(cx+9+facing, eyeY, 2.5, 0, Math.PI*2); ctx.fill();
  const swordX = facing===1?cx+20:cx-20;
  if (player.attackTimer>0) {
    const progress = 1 - player.attackTimer/14;
    ctx.save();
    ctx.translate(cx, baseY+55);
    ctx.rotate(facing * (progress*Math.PI - Math.PI/2));
    ctx.strokeStyle = 'rgba(255,255,255,0.9)';
    ctx.lineWidth = 5; ctx.lineCap = 'round';
    ctx.beginPath(); ctx.moveTo(0,0); ctx.lineTo(facing*60, -5); ctx.stroke();
    ctx.restore();
  } else {
    ctx.fillStyle = '#c0c0d0'; ctx.fillRect(swordX, baseY+40, 5*facing, 40);
    ctx.fillStyle = '#d4a017'; ctx.fillRect(swordX - (facing===1?0:5), baseY+35, 5, 8);
  }
  if (player.blockTimer>0) {
    ctx.strokeStyle = 'rgba(126,224,255,0.9)';
    ctx.lineWidth = 4;
    ctx.beginPath(); ctx.arc(cx, baseY+45, 50, 0, Math.PI*2); ctx.stroke();
  }
}

function drawEnemy(e) {
  const cx = e.x + e.w/2;
  const bob = Math.sin(animTime*0.12 + e.bobPhase)*4;
  const baseY = e.y + e.h/2 + bob;
  ctx.fillStyle = 'rgba(0,0,0,0.4)';
  ctx.beginPath(); ctx.ellipse(cx, GROUND_Y+5, 22, 6, 0, 0, Math.PI*2); ctx.fill();
  ctx.fillStyle = e.hitFlash>0?'#fff':'#c0392b';
  ctx.beginPath(); ctx.arc(cx, baseY, 26, 0, Math.PI*2); ctx.fill();
  ctx.fillStyle = e.hitFlash>0?'#fff':'#e67e22';
  ctx.beginPath(); ctx.ellipse(cx, baseY+10, 18, 10, 0, 0, Math.PI*2); ctx.fill();
  ctx.fillStyle = '#fff';
  ctx.beginPath();
  ctx.moveTo(cx-12, baseY-18); ctx.lineTo(cx-18, baseY-32); ctx.lineTo(cx-5, baseY-20); ctx.closePath();
  ctx.moveTo(cx+12, baseY-18); ctx.lineTo(cx+18, baseY-32); ctx.lineTo(cx+5, baseY-20); ctx.closePath();
  ctx.fill();
  ctx.fillStyle = '#ffd966';
  ctx.beginPath(); ctx.arc(cx-18, baseY-32, 3, 0, Math.PI*2); ctx.arc(cx+18, baseY-32, 3, 0, Math.PI*2); ctx.fill();
  const eyeY = baseY - 4;
  ctx.fillStyle = '#fff';
  ctx.beginPath(); ctx.arc(cx-8, eyeY, 6, 0, Math.PI*2); ctx.arc(cx+8, eyeY, 6, 0, Math.PI*2); ctx.fill();
  ctx.fillStyle = '#000';
  ctx.beginPath(); ctx.arc(cx-8, eyeY, 3.5, 0, Math.PI*2); ctx.arc(cx+8, eyeY, 3.5, 0, Math.PI*2); ctx.fill();
  const hpW = 40;
  ctx.fillStyle = '#000'; ctx.fillRect(cx-hpW/2, e.y-15, hpW, 5);
  ctx.fillStyle = '#ff4a4a'; ctx.fillRect(cx-hpW/2, e.y-15, hpW*(e.hp/e.maxHp), 5);
}

function loop() {
  update();
  drawBackground();
  drawSamurai(player.x, GROUND_Y-player.h);
  enemies.forEach(e => { if (!e.dead) drawEnemy(e); });
  particles.forEach(p => {
    ctx.globalAlpha = p.life/p.maxLife;
    ctx.fillStyle = p.color;
    ctx.beginPath(); ctx.arc(p.x, p.y, p.size, 0, Math.PI*2); ctx.fill();
  });
  ctx.globalAlpha = 1;
  requestAnimationFrame(loop);
}
loop();

function showToast(msg) {
  const t = document.getElementById('toast');
  t.textContent = msg;
  t.classList.add('show');
  setTimeout(() => t.classList.remove('show'), 2000);
}
function closeOverlay(id) { document.getElementById(id).classList.add('hidden'); }
function closeLevelOverlay() { document.getElementById('levelOverlay').classList.add('hidden'); }

function gameOver() {
  gameState = 'gameover'; stopMusic();
  const ov = document.getElementById('levelOverlay');
  ov.classList.remove('hidden');
  ov.innerHTML = `<h1 style="color:#ff4a4a;">💔 敗北</h1><button class="btn" onclick="restart()">再挑戦</button>`;
  saveProgress();
}
function levelUp() {
  gameState = 'levelup';
  const ov = document.getElementById('levelOverlay');
  ov.classList.remove('hidden');
  ov.innerHTML = `<h1>レベル ${level} クリア</h1><button class="btn" onclick="nextLevel()">次へ</button>`;
}
function nextLevel() { closeLevelOverlay(); resetLevel(level+1); gameState = 'playing'; }
function winGame() {
  gameState = 'win'; stopMusic();
  const ov = document.getElementById('levelOverlay');
  ov.classList.remove('hidden');
  ov.innerHTML = `<h1 style="color:#ffd966;">🌸 勝利 🌸</h1><button class="btn" onclick="restart()">もう一度</button>`;
  saveProgress();
}
function restart() { closeLevelOverlay(); state.hp = state.maxHp; resetLevel(1); gameState = 'playing'; playBattleMusic(); }

function openGacha() { document.getElementById('gachaResults').innerHTML = ''; updatePityBar(); document.getElementById('gachaOverlay').classList.remove('hidden'); }
function updatePityBar() {
  const pct = (state.gachaPity / 20) * 100;
  document.getElementById('pityFill').style.width = pct + '%';
  document.getElementById('pityLabel').textContent = `天井: ${state.gachaPity}/20`;
}
function rollGacha(count, free = false) {
  initAudio();
  if (free) {
    const today = new Date().toDateString();
    if (state.freeGachaDate === today) { showToast('使用済み'); return; }
    state.freeGachaDate = today;
  } else {
    const cost = count === 10 ? 180 : 20;
    if (state.gems < cost) { showToast('💎 が足りない！'); return; }
    state.gems -= cost;
  }
  sGacha();
  const res = document.getElementById('gachaResults');
  res.innerHTML = '';
  for (let i=0;i<count;i++) {
    let rarity = 1;
    if (state.gachaPity >= 19) rarity = 5;
    else {
      const r = Math.random();
      let acc = 0;
      const ch = [{r:1,c:0.60},{r:2,c:0.25},{r:3,c:0.10},{r:4,c:0.04},{r:5,c:0.01}];
      for (const x of ch) { acc += x.c; if (r < acc) { rarity = x.r; break; } }
    }
    const pool = GACHA_POOL.filter(c => c.rarity === rarity);
    const card = pool[Math.floor(Math.random()*pool.length)];
    state.ownedCards[card.id] = (state.ownedCards[card.id]||0)+1;
    if (rarity >= 3) state.gachaPity = 0;
    else state.gachaPity = Math.min(20, state.gachaPity+1);
    setTimeout(() => {
      const div = document.createElement('div');
      div.className = `card rare-${card.rarity}`;
      div.innerHTML = `<div class="emoji">${card.emoji}</div><div class="name-jp">${card.nameJp}</div><div class="stars">${RARITY_STARS[card.rarity]}</div>`;
      res.appendChild(div);
      if (card.rarity >= 4) sLegend();
    }, i * 150);
  }
  state.totalRolls += count;
  updateHUD(); updatePityBar(); saveProgress();
}

function openDemons() {
  const today = new Date().getDay();
  const t = TEXTS[currentLang];
  const grid = document.getElementById('demonGrid');
  grid.innerHTML = '';
  DEMONS.forEach((d, idx) => {
    const isToday = d.day === today;
    const defeated = state.demonDefeated[d.id];
    const div = document.createElement('div');
    div.className = 'demon-card' + (isToday?' today':'');
    div.innerHTML = `
      ${isToday?'<div class="today-badge">今日</div>':''}
      ${defeated?'<div class="defeated-badge">✓</div>':''}
      <div class="demon-emoji">${d.emoji}</div>
      <div class="demon-name-jp">${t.demonNames[idx]}</div>
      <div class="demon-day">${t.days[d.day === 0 ? 6 : d.day - 1]}</div>
    `;
    div.onclick = () => { if (isToday) startDemonFight(d); else showToast(t.days[d.day === 0 ? 6 : d.day - 1]); };
    grid.appendChild(div);
  });
  document.getElementById('demonsOverlay').classList.remove('hidden');
}

function checkDailyLogin() {
  const today = new Date().toDateString();
  if (state.lastLoginDate === today) return;
  const yesterday = new Date(Date.now()-86400000).toDateString();
  if (state.lastLoginDate === yesterday) state.streak++; else state.streak = 1;
  state.lastLoginDate = today;
  state.morningClaimed = false; state.eveningClaimed = false;
  updateHUD(); saveProgress();
}
function checkLoginGift() {
  const today = new Date().toDateString();
  if (state.loginGiftDate === today) return;
  const gift = 100 + state.streak * 50;
  document.getElementById('loginGiftText').textContent = `+${gift} 🪙 +${5 + state.streak} 💎`;
  document.getElementById('loginGiftOverlay').classList.remove('hidden');
  state.pendingGift = gift;
}
function claimLoginGift() {
  const gift = state.pendingGift || 100;
  state.coins += gift; state.gems += 5 + state.streak;
  state.loginGiftDate = new Date().toDateString();
  updateHUD(); saveProgress();
  closeOverlay('loginGiftOverlay');
  showToast(`🎁 +${gift} 🪙`);
}
function openDaily() {
  const t = TEXTS[currentLang];
  const bar = document.getElementById('streakBar');
  bar.innerHTML = '';
  const emojis = ['💙','🔴','🐍','💀','🔥','🐉','👑'];
  for (let i=0;i<7;i++) {
    const div = document.createElement('div');
    div.className = 'streak-day';
    if (i < state.streak) div.classList.add('claimed');
    if (i === state.streak-1) div.classList.add('today');
    div.innerHTML = `<div class="day-emoji">${emojis[i]}</div><div>${t.days[i]}</div>`;
    bar.appendChild(div);
  }
  document.getElementById('morningSlot').className = 'time-slot' + (state.morningClaimed?' claimed':'');
  document.getElementById('eveningSlot').className = 'time-slot' + (state.eveningClaimed?' claimed':'');
  document.getElementById('dailyOverlay').classList.remove('hidden');
}
function claimTimeBonus(type) {
  const today = new Date().toDateString();
  if (type === 'morning') {
    if (state.morningClaimed && state.lastMorningDate === today) return;
    state.energy = Math.min(state.maxEnergy, state.energy+1);
    state.morningClaimed = true; state.lastMorningDate = today;
  } else {
    if (state.eveningClaimed && state.lastEveningDate === today) return;
    state.coins += 200; state.eveningClaimed = true; state.lastEveningDate = today;
  }
  updateHUD(); saveProgress(); openDaily();
}

function openAlbum() {
  const grid = document.getElementById('albumGrid');
  grid.innerHTML = '';
  let count = 0;
  GACHA_POOL.forEach(card => {
    const owned = state.ownedCards[card.id];
    if (owned) count++;
    const div = document.createElement('div');
    div.className = 'album-slot' + (owned?' filled':'');
    div.innerHTML = owned ? `<div style="font-size:24px;">${card.emoji}</div><div class="name">${card.nameJp}</div>` : `<div style="font-size:24px;">?</div><div class="name">???</div>`;
    grid.appendChild(div);
  });
  document.getElementById('albumFill').style.width = (count/GACHA_POOL.length*100)+'%';
  document.getElementById('albumLabel').textContent = `${count}/${GACHA_POOL.length}`;
  document.getElementById('albumOverlay').classList.remove('hidden');
}

const ACHIEVEMENTS = [
  { id:'first_kill', name:'初勝利', check: () => state.totalKills >= 1 },
  { id:'kill_10', name:'鬼狩り', check: () => state.totalKills >= 10 },
  { id:'kill_50', name:'百鬼夜行', check: () => state.totalKills >= 50 },
  { id:'combo_10', name:'コンボ名人', check: () => state.maxCombo >= 10 },
  { id:'gacha_10', name:'ガチャ中毒', check: () => state.totalRolls >= 10 },
  { id:'streak_7', name:'常連', check: () => state.streak >= 7 },
  { id:'all_cards', name:'コレクター', check: () => Object.keys(state.ownedCards).length >= GACHA_POOL.length }
];
function openAchievements() {
  const list = document.getElementById('achList');
  list.innerHTML = '';
  ACHIEVEMENTS.forEach(a => {
    const done = a.check();
    const div = document.createElement('div');
    div.className = 'mission' + (done?' done':'');
    div.innerHTML = `<div>${done?'✓':'○'} ${a.name}</div>`;
    list.appendChild(div);
  });
  document.getElementById('achievementsOverlay').classList.remove('hidden');
}

function openTraining() { document.getElementById('trainingOverlay').classList.remove('hidden'); }
function startTraining() {
  closeOverlay('trainingOverlay');
  trainingMode = true;
  enemies = []; state.kills = 0; state.hp = state.maxHp; player.x = 150;
  updateHUD(); gameState = 'playing';
}
function openEvent() {
  const month = new Date().getMonth();
  const content = document.getElementById('eventContent');
  let html = `<p style="color:#ffe0f0;">Ивентов пока нет.</p>`;
  if (month === 0) html = `<div style="background:rgba(255,107,158,0.2);border:2px solid #ff6b9e;border-radius:12px;padding:10px;"><h3>🎍 正月</h3><button class="btn pink" onclick="claimEvent('newyear')">受取</button></div>`;
  if (month === 3) html = `<div style="background:rgba(255,107,158,0.2);border:2px solid #ff9ec7;border-radius:12px;padding:10px;"><h3>🌸 花祭り</h3><button class="btn pink" onclick="claimEvent('hanami')">受取</button></div>`;
  if (month === 4) html = `<div style="background:rgba(255,158,32,0.2);border:2px solid #ff9e20;border-radius:12px;padding:10px;"><h3>🎏 GW</h3><button class="btn" onclick="claimEvent('gw')">受取</button></div>`;
  if (month === 7) html = `<div style="background:rgba(142,68,173,0.2);border:2px solid #8e44ad;border-radius:12px;padding:10px;"><h3>🏮 お盆</h3><button class="btn purple" onclick="claimEvent('obon')">受取</button></div>`;
  content.innerHTML = html;
  document.getElementById('eventOverlay').classList.remove('hidden');
}
function claimEvent(type) {
  const today = new Date().toDateString();
  if (state['event_'+type] === today) return;
  if (type === 'newyear') { state.coins += 1000; state.gems += 100; }
  if (type === 'obon') { state.gems += 80; state.coins += 500; }
  if (type === 'hanami') { state.gems += 50; }
  if (type === 'gw') { state.coins += 2000; state.gems += 150; }
  state['event_'+type] = today;
  updateHUD(); saveProgress();
  closeOverlay('eventOverlay');
}

function openQuizFromStart() { document.getElementById('startOverlay').classList.add('hidden'); openQuiz(); }
function openQuiz() {
  const today = new Date().toDateString();
  if (state.quizDate === today) { showToast('回答済み'); return; }
  const todayDay = new Date().getDay();
  const correct = DEMONS.find(d => d.day === todayDay);
  const others = DEMONS.filter(d => d.day !== todayDay).sort(() => Math.random()-0.5).slice(0,2);
  const options = [correct, ...others].sort(() => Math.random()-0.5);
  const el = document.getElementById('quizOptions');
  el.innerHTML = '';
  const t = TEXTS[currentLang];
  options.forEach(opt => {
    const idx = DEMONS.indexOf(opt);
    const div = document.createElement('div');
    div.className = 'quiz-option';
    div.innerHTML = `<div class="opt-emoji">${opt.emoji}</div><div>${t.demonNames[idx]}</div>`;
    div.onclick = () => {
      if (opt.id === correct.id) {
        div.classList.add('correct');
        state.coins += 300; state.gems += 20;
        state.quizDate = today;
        updateHUD(); saveProgress();
        showToast('🎉 +300🪙 +20💎');
      } else div.classList.add('wrong');
      setTimeout(() => closeOverlay('quizOverlay'), 1500);
    };
    el.appendChild(div);
  });
  document.getElementById('quizOverlay').classList.remove('hidden');
}

function shareResult() {
  const text = `富士ストライク ${state.kills}体撃破！`;
  if (navigator.share) navigator.share({ title: 'Fuji Strike', text }).catch(() => {});
  else showToast('📤');
  state.coins += 100;
  updateHUD(); saveProgress();
}

console.log('🗻 v6 ready!');
</script>
</body>
</html>
