<!DOCTYPE html>
<html lang="zh-CN">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>Word Climber — 四级单词冒险记</title>
<style>
* { margin: 0; padding: 0; box-sizing: border-box; }
body { background: #1a1a2e; overflow: hidden; font-family: 'Microsoft YaHei', sans-serif; }
canvas { display: block; margin: 0 auto; }
#ui-overlay {
  position: fixed; top: 0; left: 0; width: 100%; height: 100%;
  pointer-events: none; z-index: 10;
}
#ui-overlay.active { pointer-events: all; }
#quiz-modal {
  position: fixed; top: 50%; left: 50%; transform: translate(-50%, -50%);
  background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
  border-radius: 20px; padding: 30px; width: 500px; max-width: 90vw;
  box-shadow: 0 20px 60px rgba(0,0,0,0.5); display: none; z-index: 20;
  border: 3px solid #fff;
}
#quiz-modal.show { display: block; }
#quiz-modal h2 { color: #fff; text-align: center; margin-bottom: 10px; font-size: 24px; }
#quiz-meaning { color: #ffd700; text-align: center; font-size: 18px; margin-bottom: 25px; }
.quiz-btn {
  display: block; width: 100%; padding: 14px; margin: 10px 0;
  border: 2px solid #fff; border-radius: 12px; background: rgba(255,255,255,0.15);
  color: #fff; font-size: 18px; cursor: pointer; transition: all 0.2s;
}
.quiz-btn:hover { background: rgba(255,255,255,0.35); transform: scale(1.02); }
.quiz-btn.correct { background: #00c853 !important; }
.quiz-btn.wrong { background: #ff1744 !important; }
#quiz-feedback { text-align: center; font-size: 20px; margin-top: 15px; font-weight: bold; }
#hud {
  position: fixed; top: 10px; left: 10px; color: #fff; font-size: 14px; z-index: 5;
  text-shadow: 1px 1px 2px #000;
}
#hud .hp { color: #ff5252; }
#hud .master { color: #69f0ae; }
#start-screen {
  position: fixed; top: 0; left: 0; width: 100%; height: 100%;
  background: linear-gradient(180deg, #0f0c29, #302b63, #24243e);
  display: flex; flex-direction: column; align-items: center; justify-content: center; z-index: 30;
}
#start-screen h1 { font-size: 48px; color: #fff; margin-bottom: 10px; text-shadow: 0 0 20px #667eea; }
#start-screen p { color: #aaa; font-size: 18px; margin-bottom: 40px; }
#start-btn {
  padding: 16px 50px; font-size: 22px; border: 3px solid #667eea; border-radius: 50px;
  background: transparent; color: #fff; cursor: pointer; transition: all 0.3s;
}
#start-btn:hover { background: #667eea; box-shadow: 0 0 30px #667eea; }
#mobile-controls {
  position: fixed; bottom: 20px; left: 0; width: 100%; display: none; z-index: 5;
  justify-content: space-between; padding: 0 20px;
}
.mobile-btn {
  width: 60px; height: 60px; border-radius: 50%; background: rgba(255,255,255,0.2);
  color: #fff; font-size: 24px; border: 2px solid #fff; display: flex;
  align-items: center; justify-content: center; user-select: none;
}
@media (max-width: 768px) { #mobile-controls { display: flex; } }
</style>
</head>
<body>

<div id="start-screen">
  <h1>🗼 Word Climber</h1>
  <p>四级单词冒险记 — 用单词的力量攀登词汇之塔</p>
  <button id="start-btn">开始冒险</button>
</div>

<canvas id="game"></canvas>

<div id="hud">
  <div class="hp">HP: ♥♥♥</div>
  <div class="master">掌握: <span id="master-count">0</span>/<span id="total-count">0</span></div>
  <div>第 <span id="chapter-num">1</span> 章 — 层 <span id="floor-num">1</span></div>
</div>

<div id="quiz-modal">
  <h2>⚔️ 单词挑战！</h2>
  <div id="quiz-meaning"></div>
  <div id="quiz-options"></div>
  <div id="quiz-feedback"></div>
</div>

<div id="mobile-controls">
  <div style="display:flex; gap:10px;">
    <div class="mobile-btn" id="btn-left">←</div>
    <div class="mobile-btn" id="btn-right">→</div>
    <div class="mobile-btn" id="btn-jump">↑</div>
  </div>
</div>

<script>
// ============ 单词数据（示例，可扩展） ============
const WORDS = [
  {word:"abandon", phonetic:"/əˈbændən/", meaning:"v.放弃，抛弃", chapter:1},
  {word:"ability", phonetic:"/əˈbɪləti/", meaning:"n.能力，才能", chapter:1},
  {word:"able", phonetic:"/ˈeɪbl/", meaning:"adj.能够的", chapter:1},
  {word:"abnormal", phonetic:"/æbˈnɔːrml/", meaning:"adj.反常的", chapter:1},
  {word:"aboard", phonetic:"/əˈbɔːrd/", meaning:"adv.在船上", chapter:1},
  {word:"absence", phonetic:"/ˈæbsəns/", meaning:"n.缺席", chapter:1},
  {word:"absolute", phonetic:"/ˈæbsəluːt/", meaning:"adj.绝对的", chapter:1},
  {word:"absorb", phonetic:"/əbˈsɔːrb/", meaning:"v.吸收", chapter:1},
  {word:"abstract", phonetic:"/ˈæbstrækt/", meaning:"adj.抽象的", chapter:1},
  {word:"abuse", phonetic:"/əˈbjuːs/", meaning:"v.滥用", chapter:1},
  {word:"academic", phonetic:"/ˌækəˈdemɪk/", meaning:"adj.学术的", chapter:1},
  {word:"accelerate", phonetic:"/əkˈseləreɪt/", meaning:"v.加速", chapter:1},
  {word:"access", phonetic:"/ˈækses/", meaning:"n.通道，访问", chapter:1},
  {word:"accommodate", phonetic:"/əˈkɑːmədeɪt/", meaning:"v.容纳，提供住宿", chapter:1},
  {word:"accompany", phonetic:"/əˈkʌmpəni/", meaning:"v.陪伴", chapter:1},
];

// ============ 游戏引擎 ============
const canvas = document.getElementById('game');
const ctx = canvas.getContext('2d');
canvas.width = Math.min(window.innerWidth, 1200);
canvas.height = Math.min(window.innerHeight, 680);

// 适配
window.addEventListener('resize', () => {
  canvas.width = Math.min(window.innerWidth, 1200);
  canvas.height = Math.min(window.innerHeight, 680);
});

// ============ 状态 ============
let gameState = 'start'; // start, playing, quiz
let keys = {};
let memoryLevels = {}; // word -> level 0-4
WORDS.forEach(w => memoryLevels[w.word] = 0);

// ============ 音频（Web Audio API 合成发音） ============
const audioCtx = new (window.AudioContext || window.webkitAudioContext)();
function playTone(freq, dur, type='sine') {
  const osc = audioCtx.createOscillator();
  const gain = audioCtx.createGain();
  osc.type = type; osc.frequency.value = freq;
  osc.connect(gain); gain.connect(audioCtx.destination);
  osc.start(); gain.gain.exponentialRampToValueAtTime(0.0001, audioCtx.currentTime + dur);
  osc.stop(audioCtx.currentTime + dur);
}
function sfx_correct() { playTone(523, 0.15); setTimeout(()=>playTone(659, 0.15), 100); setTimeout(()=>playTone(784, 0.2), 200); }
function sfx_wrong() { playTone(300, 0.3, 'sawtooth'); }
function sfx_hit() { playTone(150, 0.2, 'square'); }
function sfx_jump() { playTone(400, 0.1); setTimeout(()=>playTone(500, 0.1), 80); }

// ============ 玩家 ============
const player = {
  x: 50, y: 400, w: 32, h: 48,
  vx: 0, vy: 0,
  speed: 4, jumpPower: -13, facing: 1,
  hp: 3, onGround: false,
  update(platforms) {
    // 重力
    this.vy += 0.6;
    if (this.vy > 14) this.vy = 14;

    // 水平
    this.x += this.vx;
    for (const p of platforms) {
      if (this._collide(p)) {
        if (this.vx > 0) this.x = p.x - this.w;
        else if (this.vx < 0) this.x = p.x + p.w;
      }
    }

    // 垂直
    this.onGround = false;
    this.y += this.vy;
    for (const p of platforms) {
      if (this._collide(p)) {
        if (this.vy > 0) { this.y = p.y - this.h; this.onGround = true; this.vy = 0; }
        else if (this.vy < 0) { this.y = p.y + p.h; this.vy = 0; }
      }
    }

    // 边界
    if (this.x < 0) this.x = 0;
    if (this.x > canvas.width - this.w) this.x = canvas.width - this.w;
  },
  _collide(p) { return this.x < p.x + p.w && this.x + this.w > p.x && this.y < p.y + p.h && this.y + this.h > p.y; },
  draw() {
    // 赛璐璐风格：蓝+黑边
    ctx.fillStyle = '#3d7eff';
    ctx.fillRect(this.x, this.y, this.w, this.h);
    ctx.strokeStyle = '#000'; ctx.lineWidth = 2;
    ctx.strokeRect(this.x, this.y, this.w, this.h);
    // 眼睛
    const ex = this.facing > 0 ? this.x + 22 : this.x + 8;
    ctx.fillStyle = '#fff'; ctx.beginPath(); ctx.arc(ex, this.y + 14, 5, 0, Math.PI*2); ctx.fill();
    ctx.fillStyle = '#000'; ctx.beginPath(); ctx.arc(ex, this.y + 14, 2.5, 0, Math.PI*2); ctx.fill();
    // 头发
    ctx.fillStyle = '#1a1a2e';
    ctx.fillRect(this.x + 4, this.y - 2, this.w - 8, 10);
  }
};

// ============ 平台 ============
const platforms = [
  {x: 0, y: canvas.height - 40, w: canvas.width, h: 40},
  {x: 150, y: canvas.height - 130, w: 120, h: 18},
  {x: 350, y: canvas.height - 200, w: 120, h: 18},
  {x: 550, y: canvas.height - 140, w: 120, h: 18},
  {x: 750, y: canvas.height - 220, w: 120, h: 18},
  {x: 900, y: canvas.height - 160, w: 120, h: 18},
];

// ============ 单词敌人 ============
class WordEnemy {
  constructor(x, y, wordData) {
    this.x = x; this.y = y; this.w = 38; this.h = 38;
    this.word = wordData;
    this.vx = (Math.random() > 0.5 ? 1 : -1) * 1.2;
    this.triggered = false;
    this.dead = false;
    this.bobOffset = Math.random() * Math.PI * 2;
  }
  update(platforms) {
    if (this.dead) return;
    this.x += this.vx;
    let hit = false;
    for (const p of platforms) {
      if (this.x < p.x + p.w && this.x + this.w > p.x && this.y < p.y + p.h && this.y + this.h > p.y) {
        hit = true; break;
      }
    }
    if (hit) this.vx *= -1;
    // 站在平台上
    let onPlat = false;
    for (const p of platforms) {
      if (this.x + this.w > p.x && this.x < p.x + p.w) {
        if (this.y + this.h <= p.y + 5 && this.y + this.h + 5 >= p.y) {
          onPlat = true; this.y = p.y - this.h;
        }
      }
    }
    if (!onPlat) this.y += 0.5;
  }
  draw() {
    if (this.dead) return;
    const bob = Math.sin(Date.now()/300 + this.bobOffset) * 3;
    // 赛璐璐红方块
    ctx.fillStyle = this.triggered ? '#ff9800' : '#ff5252';
    ctx.fillRect(this.x, this.y + bob, this.w, this.h);
    ctx.strokeStyle = '#000'; ctx.lineWidth = 2;
    ctx.strokeRect(this.x, this.y + bob, this.w, this.h);
    // 单词首字母
    ctx.fillStyle = '#fff'; ctx.font = 'bold 18px Microsoft YaHei';
    ctx.textAlign = 'center';
    ctx.fillText(this.word.word[0].toUpperCase(), this.x + this.w/2, this.y + bob + 24);
    ctx.textAlign = 'left';
  }
}

let enemies = [];

function spawnEnemies() {
  enemies = [];
  const chapterWords = WORDS.filter(w => w.chapter === 1);
  const shuffled = [...chapterWords].sort(() => Math.random() - 0.5).slice(0, 5);
  shuffled.forEach((w, i) => {
    enemies.push(new WordEnemy(200 + i * 170, canvas.height - 180, w));
  });
}

// ============ 单词挑战 ============
let currentQuiz = null;
let quizResult = null; // 'correct' / 'wrong'
let quizTimer = 0;

function startQuiz(enemy) {
  gameState = 'quiz';
  enemy.triggered = true;
  enemy.dead = true;
  sfx_hit();

  // 选目标词 + 3个干扰项
  const target = enemy.word;
  const others = WORDS.filter(w => w.word !== target.word);
  const distractors = [...others].sort(() => Math.random() - 0.5).slice(0, 3);
  const options = [target, ...distractors].sort(() => Math.random() - 0.5);

  currentQuiz = { target, options };
  quizResult = null;

  document.getElementById('quiz-meaning').textContent = `释义：${target.meaning}`;
  const optsDiv = document.getElementById('quiz-options');
  optsDiv.innerHTML = '';
  options.forEach(opt => {
    const btn = document.createElement('button');
    btn.className = 'quiz-btn';
    btn.textContent = opt.word;
    btn.onclick = () => answerQuiz(btn, opt);
    optsDiv.appendChild(btn);
  });
  document.getElementById('quiz-feedback').textContent = '';
  document.getElementById('quiz-modal').classList.add('show');
}

function answerQuiz(btn, chosen) {
  if (quizResult) return;
  const buttons = document.querySelectorAll('.quiz-btn');
  buttons.forEach(b => b.style.pointerEvents = 'none');

  if (chosen.word === currentQuiz.target.word) {
    quizResult = 'correct';
    btn.classList.add('correct');
    sfx_correct();
    memoryLevels[chosen.word] = Math.min(4, (memoryLevels[chosen.word] || 0) + 1);
    document.getElementById('quiz-feedback').textContent = '✓ 正确！记忆度+1';
    document.getElementById('quiz-feedback').style.color = '#69f0ae';
  } else {
    quizResult = 'wrong';
    btn.classList.add('wrong');
    // 高亮正确答案
    buttons.forEach(b => {
      if (b.textContent === currentQuiz.target.word) b.classList.add('correct');
    });
    sfx_wrong();
    memoryLevels[chosen.word] = Math.max(0, (memoryLevels[chosen.word] || 0) - 1);
    document.getElementById('quiz-feedback').textContent = `✗ 错误！正确答案是：${currentQuiz.target.word}`;
    document.getElementById('quiz-feedback').style.color = '#ff5252';
    player.hp -= 1;
    if (player.hp <= 0) {
      setTimeout(() => {
        alert('冒险失败！单词之力不足，重新来过吧。');
        location.reload();
      }, 800);
    }
  }

  updateHUD();
  setTimeout(() => {
    document.getElementById('quiz-modal').classList.remove('show');
    gameState = 'playing';
  }, 1200);
}

// ============ HUD ============
function updateHUD() {
  const mastered = Object.values(memoryLevels).filter(v => v >= 3).length;
  document.getElementById('master-count').textContent = mastered;
  document.getElementById('total-count').textContent = WORDS.length;
  document.querySelector('#hud .hp').textContent = 'HP: ' + '♥'.repeat(Math.max(0, player.hp));
}

// ============ 输入 ============
document.addEventListener('keydown', e => { keys[e.key] = true; keys[e.code] = true; });
document.addEventListener('keyup', e => { keys[e.key] = false; keys[e.code] = false; });

// 移动端
let touchLeft = false, touchRight = false, touchJump = false;
document.getElementById('btn-left').addEventListener('touchstart', e => { e.preventDefault(); touchLeft = true; });
document.getElementById('btn-left').addEventListener('touchend', e => { e.preventDefault(); touchLeft = false; });
document.getElementById('btn-right').addEventListener('touchstart', e => { e.preventDefault(); touchRight = true; });
document.getElementById('btn-right').addEventListener('touchend', e => { e.preventDefault(); touchRight = false; });
document.getElementById('btn-jump').addEventListener('touchstart', e => { e.preventDefault(); touchJump = true; });
document.getElementById('btn-jump').addEventListener('touchend', e => { e.preventDefault(); touchJump = false; });

// ============ 开始 ============
document.getElementById('start-btn').addEventListener('click', () => {
  document.getElementById('start-screen').style.display = 'none';
  gameState = 'playing';
  spawnEnemies();
  updateHUD();
  audioCtx.resume();
  gameLoop();
});

// ============ 背景装饰 ============
function drawBackground() {
  // 渐变天空
  const grd = ctx.createLinearGradient(0, 0, 0, canvas.height);
  grd.addColorStop(0, '#1a1a2e');
  grd.addColorStop(0.5, '#16213e');
  grd.addColorStop(1, '#0f3460');
  ctx.fillStyle = grd;
  ctx.fillRect(0, 0, canvas.width, canvas.height);

  // 星星
  ctx.fillStyle = '#fff';
  for (let i = 0; i < 30; i++) {
    const sx = (i * 137 + Date.now()/100) % canvas.width;
    const sy = (i * 97) % (canvas.height / 2);
    const sa = 0.3 + 0.7 * Math.sin(Date.now()/500 + i);
    ctx.globalAlpha = sa * 0.5;
    ctx.fillRect(sx, sy, 2, 2);
  }
  ctx.globalAlpha = 1;

  // 塔的轮廓（远景）
  ctx.fillStyle = 'rgba(100, 100, 150, 0.15)';
  ctx.fillRect(canvas.width * 0.7, canvas.height * 0.1, 80, canvas.height * 0.9);
  ctx.fillRect(canvas.width * 0.72, canvas.height * 0.15, 60, canvas.height * 0.85);
}

// ============ 主循环 ============
function gameLoop() {
  if (gameState === 'playing') {
    // 玩家输入
    player.vx = 0;
    if (keys['a'] || keys['ArrowLeft'] || touchLeft) { player.vx = -player.speed; player.facing = -1; }
    if (keys['d'] || keys['ArrowRight'] || touchRight) { player.vx = player.speed; player.facing = 1; }
    if ((keys['w'] || keys['ArrowUp'] || keys['Space'] || touchJump) && player.onGround) {
      player.vy = player.jumpPower;
      sfx_jump();
      touchJump = false;
    }

    player.update(platforms);

    // 敌人
    enemies.forEach(e => e.update(platforms));

    // 碰撞检测
    enemies.forEach(e => {
      if (!e.dead && !e.triggered &&
          player.x < e.x + e.w && player.x + player.w > e.x &&
          player.y < e.y + e.h && player.y + player.h > e.y) {
        startQuiz(e);
      }
    });

    // 渲染
    drawBackground();

    // 平台
    platforms.forEach(p => {
      ctx.fillStyle = '#4fc3a0';
      ctx.fillRect(p.x, p.y, p.w, p.h);
      ctx.strokeStyle = '#000'; ctx.lineWidth = 2;
      ctx.strokeRect(p.x, p.y, p.w, p.h);
      // 草地顶部
      ctx.fillStyle = '#81c784';
      ctx.fillRect(p.x, p.y, p.w, 5);
    });

    enemies.forEach(e => e.draw());
    player.draw();
  }

  requestAnimationFrame(gameLoop);
}

// 初始渲染背景
drawBackground();
updateHUD();
</script>
</body>
</html>
