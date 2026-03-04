<!DOCTYPE html>

<html lang="en">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>PIXEL QUEST — Blade of Eternity</title>
<style>
  @import url('https://fonts.googleapis.com/css2?family=Press+Start+2P&display=swap');
  * { margin: 0; padding: 0; box-sizing: border-box; }
  body {
    background: #0a0a0f;
    display: flex; flex-direction: column;
    align-items: center; justify-content: center;
    min-height: 100vh;
    font-family: 'Press Start 2P', monospace;
    overflow: hidden;
  }
  #gameWrapper {
    position: relative;
    border: 3px solid #3a3a5c;
    box-shadow: 0 0 40px #7b68ee55, 0 0 80px #ff6b3520;
  }
  canvas { display: block; image-rendering: pixelated; }
  #ui {
    position: absolute; top: 0; left: 0; right: 0;
    padding: 8px 12px;
    display: flex; align-items: flex-start; justify-content: space-between;
    pointer-events: none;
  }
  .stat-bar { display: flex; flex-direction: column; gap: 4px; }
  .bar-row { display: flex; align-items: center; gap: 6px; }
  .bar-label { font-size: 6px; color: #aaa; width: 14px; }
  .bar-bg { width: 90px; height: 8px; background: #1a1a2e; border: 1px solid #333; }
  .bar-fill { height: 100%; transition: width 0.2s; }
  .hp-fill { background: linear-gradient(90deg, #ff3333, #ff6666); }
  .xp-fill { background: linear-gradient(90deg, #7b68ee, #b39ddb); }
  #levelDisplay { font-size: 7px; color: #ffd700; text-shadow: 0 0 8px #ffd70088; }
  #goldDisplay { font-size: 7px; color: #ffd700; }
  #mapDisplay { font-size: 6px; color: #7b9dce; }
  #inventory {
    position: absolute; bottom: 0; left: 0; right: 0;
    padding: 6px 12px;
    display: flex; align-items: center; justify-content: space-between;
    pointer-events: none;
  }
  #swordSlots { display: flex; gap: 6px; align-items: center; }
  .sword-slot {
    width: 32px; height: 32px;
    background: #111122;
    border: 2px solid #333;
    display: flex; align-items: center; justify-content: center;
    font-size: 14px;
    position: relative;
  }
  .sword-slot.active { border-color: #ffd700; box-shadow: 0 0 8px #ffd70088; }
  .sword-slot.locked { opacity: 0.4; }
  .sword-label { font-size: 5px; color: #888; margin-top: 2px; }
  #killCount { font-size: 6px; color: #ff6b6b; }
  #controls {
    position: absolute; right: -170px; top: 0;
    width: 160px; padding: 10px;
    background: #0d0d1a;
    border: 2px solid #2a2a4a;
    font-size: 5px; color: #888; line-height: 2;
  }
  #controls h3 { color: #7b68ee; font-size: 6px; margin-bottom: 8px; }
  #overlay {
    position: absolute; inset: 0;
    background: rgba(0,0,0,0.85);
    display: flex; flex-direction: column;
    align-items: center; justify-content: center;
    gap: 16px;
  }
  #overlay h1 { font-size: 14px; color: #ffd700; text-shadow: 0 0 20px #ffd700; letter-spacing: 2px; }
  #overlay p { font-size: 7px; color: #aaa; }
  .menu-btn {
    font-family: 'Press Start 2P', monospace;
    font-size: 8px; padding: 10px 20px;
    background: #1a1a3e; color: #ffd700;
    border: 2px solid #7b68ee;
    cursor: pointer;
    transition: all 0.2s;
    pointer-events: all;
  }
  .menu-btn:hover { background: #7b68ee; color: #fff; transform: scale(1.05); }
  #lootPopup {
    position: absolute; top: 50%; left: 50%;
    transform: translate(-50%, -50%);
    background: #0d0d1a; border: 2px solid #ffd700;
    padding: 12px; font-size: 6px; color: #ffd700;
    pointer-events: none; opacity: 0;
    transition: opacity 0.3s;
    text-align: center; min-width: 100px;
    z-index: 10;
  }
  #lootPopup.show { opacity: 1; }
  #notification {
    position: absolute; top: 60px; left: 50%;
    transform: translateX(-50%);
    font-size: 7px; color: #7fffff;
    text-shadow: 0 0 10px #7fffff;
    pointer-events: none; opacity: 0;
    transition: opacity 0.5s;
    white-space: nowrap;
  }
  #notification.show { opacity: 1; }
</style>
</head>
<body>
<div id="gameWrapper">
  <canvas id="gameCanvas" width="640" height="480"></canvas>

  <div id="ui">
    <div class="stat-bar">
      <div id="levelDisplay">LVL 1</div>
      <div class="bar-row">
        <span class="bar-label">HP</span>
        <div class="bar-bg"><div class="bar-fill hp-fill" id="hpBar" style="width:100%"></div></div>
        <span style="font-size:6px;color:#ff6666" id="hpText">100/100</span>
      </div>
      <div class="bar-row">
        <span class="bar-label">XP</span>
        <div class="bar-bg"><div class="bar-fill xp-fill" id="xpBar" style="width:0%"></div></div>
        <span style="font-size:6px;color:#b39ddb" id="xpText">0/100</span>
      </div>
      <div id="goldDisplay">💰 0 GOLD</div>
    </div>
    <div style="text-align:right">
      <div id="mapDisplay">MAP: VERDANT FOREST</div>
      <div id="killCount" style="margin-top:4px">KILLS: 0</div>
    </div>
  </div>

  <div id="inventory">
    <div>
      <div style="font-size:5px;color:#666;margin-bottom:4px">WEAPONS [1-4]</div>
      <div id="swordSlots"></div>
    </div>
    <div style="text-align:right;font-size:5px;color:#555">
      WASD/ARROWS: MOVE<br>SPACE/CLICK: ATTACK<br>1-4: SWITCH SWORD
    </div>
  </div>

  <div id="lootPopup"></div>
  <div id="notification"></div>

  <div id="overlay">
    <h1>⚔ PIXEL QUEST ⚔</h1>
    <p style="color:#7b68ee">BLADE OF ETERNITY</p>
    <p>Slay monsters, collect loot, unlock legendary swords!</p>
    <button class="menu-btn" id="startBtn">▶ BEGIN QUEST</button>
    <p style="font-size:5px;color:#555">WASD/ARROWS to move • SPACE or click to attack</p>
  </div>

  <div id="controls">
    <h3>CONTROLS</h3>
    WASD / ↑↓←→ Move<br>
    SPACE / Click Attack<br>
    1-4 Switch Sword<br>
    <br>
    <span style="color:#ffd700">SWORDS:</span><br>
    🗡 Iron — start<br>
    ⚔ Steel — Lv3<br>
    🔥 Flame — Lv6<br>
    💜 Soul — Lv10<br>
    <br>
    <span style="color:#ffd700">MAPS:</span><br>
    🌲 Forest — Lv1<br>
    💀 Dungeon — Lv5<br>
    🌋 Volcano — Lv9<br>
  </div>
</div>

<script>
const canvas = document.getElementById('gameCanvas');
const ctx = canvas.getContext('2d');
ctx.imageSmoothingEnabled = false;

// ─── CONSTANTS ────────────────────────────────────────────────
const TILE = 32;
const COLS = 20, ROWS = 15;
const W = COLS * TILE, H = ROWS * TILE;

// ─── GAME STATE ───────────────────────────────────────────────
let state = 'menu';
let keys = {};
let mousePos = {x:0,y:0};
let mouseClick = false;

// ─── SWORDS ───────────────────────────────────────────────────
const SWORDS = [
  { id:0, name:'IRON BLADE',   emoji:'🗡', color:'#aaaaaa', dmg:15, range:40, speed:12, unlockLv:1,  swingFrames:6,  knockback:3 },
  { id:1, name:'STEEL SWORD',  emoji:'⚔', color:'#88aaff', dmg:28, range:52, speed:10, unlockLv:3,  swingFrames:5,  knockback:5 },
  { id:2, name:'FLAME SWORD',  emoji:'🔥', color:'#ff6600', dmg:45, range:58, speed:8,  unlockLv:6,  swingFrames:4,  knockback:7, trail:true },
  { id:3, name:'SOUL REAPER',  emoji:'💜', color:'#cc44ff', dmg:80, range:70, speed:6,  unlockLv:10, swingFrames:3,  knockback:10, aoe:true },
];

// ─── MONSTERS ─────────────────────────────────────────────────
const MONSTER_TYPES = [
  { type:'slime',   color:'#44ff88', size:12, hp:30,  dmg:8,  speed:0.6, xp:15,  gold:[1,5],   loot:['Slime Gel','Health Potion'], emoji:'🟢' },
  { type:'goblin',  color:'#88cc44', size:14, hp:55,  dmg:15, speed:0.9, xp:30,  gold:[3,10],  loot:['Goblin Tooth','Iron Shard','Small Potion'], emoji:'👺' },
  { type:'orc',     color:'#886633', size:18, hp:120, dmg:25, speed:0.5, xp:60,  gold:[8,20],  loot:['Orc Hide','Steel Fragment','Elixir'], emoji:'👹' },
  { type:'demon',   color:'#ff4444', size:16, hp:90,  dmg:35, speed:1.1, xp:80,  gold:[10,30], loot:['Demon Horn','Flame Essence','Mega Potion'], emoji:'😈' },
  { type:'dragon',  color:'#ff8800', size:24, hp:300, dmg:50, speed:0.4, xp:200, gold:[30,80], loot:['Dragon Scale','Dragon Claw','Soul Crystal'], emoji:'🐉', boss:true },
];

// ─── MAPS ─────────────────────────────────────────────────────
const MAPS = [
  { name:'VERDANT FOREST', bgColor:'#1a2e1a', tileColors:['#2d4a2d','#3d5a3d','#253525'], treeColor:'#1a3a1a', monsterTypes:[0,1], ambientColor:'#2a4a2a', minLv:1 },
  { name:'DARK DUNGEON',   bgColor:'#1a1a2a', tileColors:['#2a2a3a','#252535','#303040'], treeColor:'#1a1a2a', monsterTypes:[1,2,3], ambientColor:'#1a1a3a', minLv:5 },
  { name:'VOLCANO PEAK',   bgColor:'#2a1a0a', tileColors:['#3a2010','#4a2a15','#2a1508'], treeColor:'#ff4400', monsterTypes:[2,3,4], ambientColor:'#3a1a0a', minLv:9 },
];

// ─── PLAYER ───────────────────────────────────────────────────
let player = {
  x: W/2, y: H/2,
  vx: 0, vy: 0,
  hp: 100, maxHp: 100,
  xp: 0, xpNeeded: 100,
  level: 1,
  gold: 0,
  kills: 0,
  sword: 0,
  unlockedSwords: [0],
  facing: 1,
  walkFrame: 0,
  walkTimer: 0,
  invincible: 0,
  swinging: false,
  swingAngle: 0,
  swingTimer: 0,
  swingDir: 1,
  damageFlash: 0,
  speed: 2.2,
};

let monsters = [];
let particles = [];
let floaters = [];
let lootItems = [];
let mapTiles = [];
let trees = [];
let currentMap = 0;
let spawnTimer = 0;
let spawnInterval = 180;
let gameTimer = 0;
let kills = 0;
let lootPopupTimer = 0;
let notifTimer = 0;

// ─── GENERATE MAP ─────────────────────────────────────────────
function generateMap(mapIdx) {
  const m = MAPS[mapIdx];
  mapTiles = [];
  trees = [];
  for (let r = 0; r < ROWS; r++) {
    mapTiles[r] = [];
    for (let c = 0; c < COLS; c++) {
      const v = Math.random();
      mapTiles[r][c] = v < 0.6 ? 0 : v < 0.85 ? 1 : 2;
    }
  }
  // Random trees/obstacles
  for (let i = 0; i < 22; i++) {
    const tx = TILE + Math.random() * (W - TILE*2);
    const ty = TILE + Math.random() * (H - TILE*2);
    const dist = Math.hypot(tx - W/2, ty - H/2);
    if (dist > 80) {
      trees.push({ x: tx, y: ty, size: 12 + Math.random()*14, variant: Math.floor(Math.random()*3) });
    }
  }
  monsters = [];
  particles = [];
  floaters = [];
  lootItems = [];
  spawnTimer = 0;
}

// ─── SPAWN MONSTER ────────────────────────────────────────────
function spawnMonster() {
  const m = MAPS[currentMap];
  const typeIdx = m.monsterTypes[Math.floor(Math.random() * m.monsterTypes.length)];
  const mt = MONSTER_TYPES[typeIdx];
  // Spawn at edge
  const edge = Math.floor(Math.random()*4);
  let x, y;
  if (edge===0) { x=Math.random()*W; y=20; }
  else if (edge===1) { x=W-20; y=Math.random()*H; }
  else if (edge===2) { x=Math.random()*W; y=H-20; }
  else { x=20; y=Math.random()*H; }
  const scaleLv = 1 + (player.level-1)*0.15;
  monsters.push({
    ...mt,
    x, y,
    maxHp: Math.floor(mt.hp * scaleLv),
    hp: Math.floor(mt.hp * scaleLv),
    dmg: Math.floor(mt.dmg * scaleLv),
    vx: 0, vy: 0,
    hitFlash: 0,
    aiTimer: Math.random()*60,
    wanderAngle: Math.random()*Math.PI*2,
    attackCooldown: 0,
    wobble: 0,
    deathTimer: -1,
    id: Math.random(),
  });
}

// ─── PARTICLES ────────────────────────────────────────────────
function spawnBlood(x, y, count, color) {
  for (let i = 0; i < count; i++) {
    const angle = Math.random() * Math.PI * 2;
    const speed = 1 + Math.random() * 4;
    const size = 2 + Math.random() * 4;
    particles.push({
      type: 'blood',
      x, y,
      vx: Math.cos(angle) * speed,
      vy: Math.sin(angle) * speed - 1,
      size,
      color,
      life: 30 + Math.random()*30,
      maxLife: 60,
      gravity: 0.18,
    });
  }
}

function spawnSwordTrail(x, y, color) {
  for (let i = 0; i < 3; i++) {
    particles.push({
      type: 'trail',
      x: x + (Math.random()-0.5)*10,
      y: y + (Math.random()-0.5)*10,
      vx: (Math.random()-0.5)*1,
      vy: (Math.random()-0.5)*1 - 0.5,
      size: 3+Math.random()*4,
      color,
      life: 15+Math.random()*10,
      maxLife: 25,
      gravity: 0,
    });
  }
}

function spawnSpark(x, y, color, count=6) {
  for (let i = 0; i < count; i++) {
    const angle = Math.random() * Math.PI * 2;
    const speed = 2 + Math.random() * 3;
    particles.push({
      type: 'spark',
      x, y,
      vx: Math.cos(angle)*speed,
      vy: Math.sin(angle)*speed,
      size: 2,
      color,
      life: 12+Math.random()*8,
      maxLife: 20,
      gravity: 0.1,
    });
  }
}

function spawnAOE(x, y, radius, color) {
  for (let i = 0; i < 30; i++) {
    const angle = (i/30)*Math.PI*2;
    const r = radius * (0.5+Math.random()*0.5);
    particles.push({
      type: 'aoe',
      x: x+Math.cos(angle)*r,
      y: y+Math.sin(angle)*r,
      vx: Math.cos(angle)*1.5,
      vy: Math.sin(angle)*1.5,
      size: 4+Math.random()*6,
      color,
      life: 20+Math.random()*15,
      maxLife: 35,
      gravity: 0,
    });
  }
}

function addFloater(x, y, text, color) {
  floaters.push({ x, y, text, color, life: 60, vy: -1.2 });
}

// ─── DRAW FUNCTIONS ───────────────────────────────────────────
function drawPixelRect(x, y, w, h, color) {
  ctx.fillStyle = color;
  ctx.fillRect(Math.floor(x), Math.floor(y), w, h);
}

function drawPlayer() {
  const px = Math.floor(player.x - 12);
  const py = Math.floor(player.y - 20);
  const flash = player.invincible > 0 && Math.floor(player.invincible/4)%2 === 0;
  if (flash) return;
  const dmgFlash = player.damageFlash > 0;
  
  ctx.save();
  if (dmgFlash) { ctx.globalAlpha = 0.85; }
  
  // Shadow
  ctx.fillStyle = 'rgba(0,0,0,0.3)';
  ctx.beginPath();
  ctx.ellipse(player.x, player.y+2, 10, 4, 0, 0, Math.PI*2);
  ctx.fill();
  
  const walkBob = Math.sin(player.walkFrame * 0.8) * (player.vx !== 0 || player.vy !== 0 ? 2 : 0);
  const py2 = py + walkBob;
  
  // Body
  drawPixelRect(px+3, py2+8, 18, 12, dmgFlash ? '#ff4444' : '#e8c47a'); // torso skin
  // Shirt
  drawPixelRect(px+3, py2+10, 18, 8, dmgFlash ? '#ff2222' : '#4466cc');
  // Belt
  drawPixelRect(px+3, py2+17, 18, 2, '#885500');
  // Pants
  drawPixelRect(px+4, py2+19, 7, 6, '#223366');
  drawPixelRect(px+13, py2+19, 7, 6, '#223366');
  // Boots
  drawPixelRect(px+3, py2+24, 8, 4, '#442200');
  drawPixelRect(px+12, py2+24, 8, 4, '#442200');
  // Head
  drawPixelRect(px+5, py2, 14, 12, dmgFlash ? '#ff6666' : '#f5c78c');
  // Hair
  drawPixelRect(px+5, py2, 14, 4, '#332200');
  // Eyes
  ctx.fillStyle = '#ffffff';
  const eyeX = player.facing > 0 ? px+14 : px+5;
  drawPixelRect(eyeX, py2+5, 4, 3, '#ffffff');
  drawPixelRect(eyeX + (player.facing>0?1:-1), py2+6, 2, 2, '#222244');
  // Arms
  drawPixelRect(px, py2+10, 4, 10, dmgFlash ? '#ff6666' : '#f5c78c');
  drawPixelRect(px+20, py2+10, 4, 10, dmgFlash ? '#ff6666' : '#f5c78c');
  
  ctx.restore();
  
  // Draw sword swing
  if (player.swinging) {
    drawSwordSwing();
  }
  
  player.damageFlash = Math.max(0, player.damageFlash - 1);
}

function drawSwordSwing() {
  const sw = SWORDS[player.sword];
  const progress = player.swingTimer / (sw.swingFrames * 3);
  const baseAngle = player.facing > 0 ? -0.3 : Math.PI + 0.3;
  const swingArc = 2.2;
  const angle = baseAngle + (progress * swingArc * player.swingDir) - swingArc*0.3;
  
  const armX = player.x + player.facing * 8;
  const armY = player.y - 8;
  
  ctx.save();
  ctx.translate(armX, armY);
  ctx.rotate(angle);
  
  // Sword glow for special swords
  if (sw.trail) {
    ctx.shadowColor = sw.color;
    ctx.shadowBlur = 12;
    spawnSwordTrail(armX + Math.cos(angle)*sw.range*0.7, armY + Math.sin(angle)*sw.range*0.7, sw.color);
  }
  if (sw.aoe) {
    ctx.shadowColor = sw.color;
    ctx.shadowBlur = 20;
  }
  
  // Blade
  ctx.fillStyle = sw.color;
  ctx.fillRect(0, -2, sw.range, 4);
  // Blade edge highlight
  ctx.fillStyle = '#ffffff88';
  ctx.fillRect(0, -2, sw.range, 1);
  // Guard
  ctx.fillStyle = '#885500';
  ctx.fillRect(-2, -6, 6, 12);
  // Handle
  ctx.fillStyle = '#663300';
  ctx.fillRect(-10, -3, 10, 6);
  
  // Flame effect
  if (sw.trail) {
    for (let i = 0; i < 5; i++) {
      ctx.fillStyle = `hsla(${20+i*8},100%,${50+i*10}%,${0.6-i*0.1})`;
      ctx.fillRect(sw.range*0.3+i*5, -3-i, sw.range*0.4-i*5, 2+i*2);
    }
  }
  if (sw.aoe) {
    ctx.fillStyle = `hsla(280,100%,70%,0.5)`;
    ctx.beginPath();
    ctx.arc(sw.range, 0, 10, 0, Math.PI*2);
    ctx.fill();
  }
  
  ctx.restore();
}

function drawMonster(m) {
  if (m.deathTimer >= 0) return;
  ctx.save();
  
  const flash = m.hitFlash > 0 && Math.floor(m.hitFlash/2)%2 === 0;
  const wobble = Math.sin(m.wobble * 0.5) * 2;
  
  ctx.translate(Math.floor(m.x), Math.floor(m.y));
  if (flash) ctx.globalAlpha = 0.6;
  
  const s = m.size;
  
  // Shadow
  ctx.fillStyle = 'rgba(0,0,0,0.3)';
  ctx.beginPath();
  ctx.ellipse(0, s/2+2, s*0.7, s*0.25, 0, 0, Math.PI*2);
  ctx.fill();
  
  if (m.type === 'slime') {
    // Slime body
    ctx.fillStyle = flash ? '#ffffff' : m.color;
    ctx.beginPath();
    ctx.ellipse(wobble, 0, s, s*0.75, 0, 0, Math.PI*2);
    ctx.fill();
    ctx.fillStyle = m.color+'88';
    ctx.beginPath();
    ctx.ellipse(wobble-3, -4, s*0.4, s*0.3, -0.4, 0, Math.PI*2);
    ctx.fill();
    // Eyes
    ctx.fillStyle = '#000';
    ctx.fillRect(-4, -4, 3, 3);
    ctx.fillRect(3, -4, 3, 3);
    ctx.fillStyle = '#fff';
    ctx.fillRect(-3, -5, 1, 1);
    ctx.fillRect(4, -5, 1, 1);
  } else if (m.type === 'goblin') {
    // Body
    ctx.fillStyle = flash ? '#ffffff' : m.color;
    ctx.fillRect(-s/2, -s/2, s, s*1.2);
    // Ears
    ctx.fillRect(-s/2-4, -s/2, 5, 7);
    ctx.fillRect(s/2, -s/2, 5, 7);
    // Eyes
    ctx.fillStyle = '#ff2200';
    ctx.fillRect(-5, -s/2+2, 4, 4);
    ctx.fillRect(2, -s/2+2, 4, 4);
    // Mouth
    ctx.fillStyle = '#222';
    ctx.fillRect(-4, s/2-8, 8, 3);
    // Weapon
    ctx.fillStyle = '#888';
    ctx.fillRect(s/2+2, -s, 3, s*1.5);
  } else if (m.type === 'orc') {
    // Big body
    ctx.fillStyle = flash ? '#ffffff' : m.color;
    ctx.fillRect(-s/2, -s/2, s, s);
    // Shoulders
    ctx.fillRect(-s/2-5, -s/2, 8, s*0.6);
    ctx.fillRect(s/2-3, -s/2, 8, s*0.6);
    // Head
    ctx.fillRect(-s/2+2, -s*0.9, s-4, s*0.55);
    // Tusks
    ctx.fillStyle = '#ffffaa';
    ctx.fillRect(-5, -s*0.5, 3, 8);
    ctx.fillRect(3, -s*0.5, 3, 8);
    // Eyes
    ctx.fillStyle = '#ff4400';
    ctx.fillRect(-6, -s*0.75, 5, 4);
    ctx.fillRect(2, -s*0.75, 5, 4);
  } else if (m.type === 'demon') {
    // Body
    ctx.fillStyle = flash ? '#ffffff' : m.color;
    ctx.fillRect(-s/2, -s/2, s, s);
    // Wings
    ctx.fillStyle = '#880000';
    ctx.beginPath();
    ctx.moveTo(-s/2, -s/2);
    ctx.lineTo(-s/2-14, -s*0.9-wobble);
    ctx.lineTo(-s/2-8, s/2);
    ctx.closePath();
    ctx.fill();
    ctx.beginPath();
    ctx.moveTo(s/2, -s/2);
    ctx.lineTo(s/2+14, -s*0.9-wobble);
    ctx.lineTo(s/2+8, s/2);
    ctx.closePath();
    ctx.fill();
    // Horns
    ctx.fillStyle = '#440000';
    ctx.fillRect(-6, -s*0.85, 4, -8-wobble);
    ctx.fillRect(3, -s*0.85, 4, -8-wobble);
    // Eyes
    ctx.fillStyle = '#ffff00';
    ctx.fillRect(-7, -s*0.5, 5, 5);
    ctx.fillRect(3, -s*0.5, 5, 5);
  } else if (m.type === 'dragon') {
    // Big dragon
    ctx.fillStyle = flash ? '#ffffff' : m.color;
    ctx.fillRect(-s/2, -s/3, s, s);
    // Neck
    ctx.fillRect(-s/4, -s, s/2, s*0.8);
    // Head
    ctx.fillRect(-s/3, -s*1.4, s*0.66, s*0.6);
    // Snout
    ctx.fillRect(-s/4, -s*1.1, s/3, s*0.25);
    // Wings big
    ctx.fillStyle = '#cc4400';
    ctx.beginPath();
    ctx.moveTo(-s/2, -s/4);
    ctx.lineTo(-s/2-28, -s*1.2-wobble);
    ctx.lineTo(-s/2-18, s/2);
    ctx.closePath();
    ctx.fill();
    ctx.beginPath();
    ctx.moveTo(s/2, -s/4);
    ctx.lineTo(s/2+28, -s*1.2-wobble);
    ctx.lineTo(s/2+18, s/2);
    ctx.closePath();
    ctx.fill();
    // Eyes
    ctx.fillStyle = '#ffffff';
    ctx.fillRect(-s/4, -s*1.2, 6, 6);
    ctx.fillRect(s/4-4, -s*1.2, 6, 6);
    ctx.fillStyle = '#ff2200';
    ctx.fillRect(-s/4+1, -s*1.2+1, 4, 4);
    ctx.fillRect(s/4-3, -s*1.2+1, 4, 4);
    // Tail
    ctx.fillStyle = m.color;
    ctx.fillRect(s/2, s/4, s/2+wobble*2, s/3);
    ctx.fillRect(s, s/3, s/3, s/4);
  }
  
  // HP bar above monster
  const barW = s * 1.5;
  const hpPct = m.hp / m.maxHp;
  ctx.fillStyle = '#220000';
  ctx.fillRect(-barW/2, -s - 10, barW, 4);
  ctx.fillStyle = hpPct > 0.5 ? '#44ff44' : hpPct > 0.25 ? '#ffaa00' : '#ff2200';
  ctx.fillRect(-barW/2, -s - 10, barW * hpPct, 4);
  
  ctx.restore();
}

function drawParticles() {
  particles.forEach(p => {
    const alpha = p.life / p.maxLife;
    ctx.save();
    ctx.globalAlpha = alpha;
    if (p.type === 'blood') {
      ctx.fillStyle = p.color;
      ctx.shadowColor = p.color;
      ctx.shadowBlur = 2;
      // Pixel blood drops
      ctx.fillRect(Math.floor(p.x)-p.size/2, Math.floor(p.y)-p.size/2, Math.ceil(p.size), Math.ceil(p.size));
      if (p.size > 3) {
        ctx.fillRect(Math.floor(p.x)-1, Math.floor(p.y)+p.size/2, 2, 2);
      }
    } else if (p.type === 'trail') {
      ctx.fillStyle = p.color;
      ctx.shadowColor = p.color;
      ctx.shadowBlur = 8;
      ctx.fillRect(Math.floor(p.x)-p.size/2, Math.floor(p.y)-p.size/2, Math.ceil(p.size), Math.ceil(p.size));
    } else if (p.type === 'spark') {
      ctx.strokeStyle = p.color;
      ctx.lineWidth = 2;
      ctx.beginPath();
      ctx.moveTo(p.x, p.y);
      ctx.lineTo(p.x - p.vx*3, p.y - p.vy*3);
      ctx.stroke();
    } else if (p.type === 'aoe') {
      ctx.fillStyle = p.color;
      ctx.shadowColor = p.color;
      ctx.shadowBlur = 12;
      ctx.beginPath();
      ctx.arc(p.x, p.y, p.size*alpha, 0, Math.PI*2);
      ctx.fill();
    } else if (p.type === 'loot') {
      ctx.fillStyle = p.color;
      ctx.shadowColor = p.color;
      ctx.shadowBlur = 6;
      ctx.fillRect(Math.floor(p.x)-3, Math.floor(p.y)-3, 6, 6);
    }
    ctx.restore();
  });
}

function drawLootItems() {
  lootItems.forEach(l => {
    const bob = Math.sin(gameTimer * 0.05 + l.phase) * 3;
    ctx.save();
    ctx.shadowColor = l.color;
    ctx.shadowBlur = 8 + Math.sin(gameTimer*0.08)*4;
    ctx.font = '16px serif';
    ctx.textAlign = 'center';
    ctx.fillText(l.emoji, l.x, l.y + bob);
    ctx.restore();
    // Glow ring
    ctx.save();
    ctx.globalAlpha = 0.3 + Math.sin(gameTimer*0.08)*0.1;
    ctx.strokeStyle = l.color;
    ctx.lineWidth = 2;
    ctx.beginPath();
    ctx.arc(l.x, l.y+bob+4, 12, 0, Math.PI*2);
    ctx.stroke();
    ctx.restore();
  });
}

function drawFloaters() {
  floaters.forEach(f => {
    ctx.save();
    ctx.globalAlpha = Math.min(1, f.life/20);
    ctx.fillStyle = f.color;
    ctx.shadowColor = f.color;
    ctx.shadowBlur = 6;
    ctx.font = '7px "Press Start 2P"';
    ctx.textAlign = 'center';
    ctx.fillText(f.text, f.x, f.y);
    ctx.restore();
  });
}

function drawMap() {
  const m = MAPS[currentMap];
  // Background
  ctx.fillStyle = m.bgColor;
  ctx.fillRect(0, 0, W, H);
  
  // Tiles
  for (let r = 0; r < ROWS; r++) {
    for (let c = 0; c < COLS; c++) {
      const v = mapTiles[r][c];
      ctx.fillStyle = m.tileColors[v];
      ctx.fillRect(c*TILE, r*TILE, TILE, TILE);
      // Subtle grid
      ctx.fillStyle = 'rgba(0,0,0,0.08)';
      ctx.fillRect(c*TILE, r*TILE, 1, TILE);
      ctx.fillRect(c*TILE, r*TILE, TILE, 1);
    }
  }
  
  // Trees/rocks
  trees.forEach(t => {
    ctx.save();
    ctx.translate(t.x, t.y);
    if (currentMap === 0) {
      // Tree
      ctx.fillStyle = '#5a3a1a';
      ctx.fillRect(-4, 0, 8, t.size*0.7);
      ctx.fillStyle = m.treeColor;
      ctx.beginPath();
      ctx.moveTo(0, -t.size*1.2);
      ctx.lineTo(-t.size*0.7, t.size*0.1);
      ctx.lineTo(t.size*0.7, t.size*0.1);
      ctx.closePath();
      ctx.fill();
      ctx.fillStyle = '#2a5a2a';
      ctx.beginPath();
      ctx.moveTo(0, -t.size*0.9);
      ctx.lineTo(-t.size*0.55, t.size*0.2);
      ctx.lineTo(t.size*0.55, t.size*0.2);
      ctx.closePath();
      ctx.fill();
    } else if (currentMap === 1) {
      // Dungeon pillar
      ctx.fillStyle = '#333344';
      ctx.fillRect(-t.size/2, -t.size*0.8, t.size, t.size*1.5);
      ctx.fillStyle = '#444455';
      ctx.fillRect(-t.size/2, -t.size*0.8, t.size*0.3, t.size*1.5);
      // Cracks
      ctx.fillStyle = '#111';
      ctx.fillRect(-2, -t.size*0.4, 1, t.size*0.8);
    } else {
      // Volcano rock
      ctx.fillStyle = '#331100';
      ctx.beginPath();
      ctx.arc(0, 0, t.size*0.7, 0, Math.PI*2);
      ctx.fill();
      ctx.fillStyle = '#ff440022';
      ctx.beginPath();
      ctx.arc(0, 0, t.size*0.4, 0, Math.PI*2);
      ctx.fill();
    }
    ctx.restore();
  });
  
  // Ambient light overlay
  const ambientGrad = ctx.createRadialGradient(player.x, player.y, 0, player.x, player.y, 260);
  ambientGrad.addColorStop(0, 'rgba(0,0,0,0)');
  ambientGrad.addColorStop(1, 'rgba(0,0,0,0.45)');
  ctx.fillStyle = ambientGrad;
  ctx.fillRect(0, 0, W, H);
  
  // Volcano lava glow
  if (currentMap === 2) {
    ctx.save();
    ctx.globalAlpha = 0.15 + Math.sin(gameTimer*0.03)*0.05;
    const lavaGrad = ctx.createRadialGradient(W/2, H+50, 50, W/2, H+50, H*1.2);
    lavaGrad.addColorStop(0, '#ff4400');
    lavaGrad.addColorStop(1, 'rgba(0,0,0,0)');
    ctx.fillStyle = lavaGrad;
    ctx.fillRect(0, 0, W, H);
    ctx.restore();
  }
}

// ─── ATTACK LOGIC ────────────────────────────────────────────
function doAttack() {
  if (player.swinging) return;
  player.swinging = true;
  player.swingTimer = 0;
  player.swingDir = (Math.random() > 0.5 ? 1 : -1);
  
  const sw = SWORDS[player.sword];
  
  // Determine hit area
  const attackX = player.x + player.facing * sw.range * 0.6;
  const attackY = player.y - 8;
  const hitRadius = sw.range * 0.55;
  
  if (sw.aoe) {
    spawnAOE(player.x, player.y - 8, sw.range, sw.color);
  }
  
  let hit = false;
  monsters.forEach(m => {
    if (m.deathTimer >= 0) return;
    const dist = Math.hypot(m.x - attackX, m.y - attackY);
    if (dist < hitRadius + m.size) {
      hit = true;
      hitMonster(m, sw);
    }
  });
  
  if (!hit) {
    // Miss spark
    const mx = attackX + (Math.random()-0.5)*20;
    const my = attackY + (Math.random()-0.5)*20;
    spawnSpark(mx, my, '#ffffff', 3);
  }
  
  return hit;
}

function hitMonster(m, sw) {
  const dmgMult = 1 + (player.level - 1) * 0.1;
  const dmg = Math.floor(sw.dmg * dmgMult * (0.85 + Math.random()*0.3));
  m.hp -= dmg;
  m.hitFlash = 12;
  m.wobble = 0;
  
  // Knockback
  const dx = m.x - player.x;
  const dy = m.y - player.y;
  const dist = Math.hypot(dx, dy) || 1;
  m.x += (dx/dist) * sw.knockback * 3;
  m.y += (dy/dist) * sw.knockback * 3;
  m.x = Math.max(m.size, Math.min(W-m.size, m.x));
  m.y = Math.max(m.size, Math.min(H-m.size, m.y));
  
  // Blood
  const bloodColor = m.type === 'slime' ? '#44ff88' : m.type === 'goblin' ? '#cc4422' : '#cc2222';
  spawnBlood(m.x, m.y, 6 + (sw.aoe ? 12 : 0), bloodColor);
  spawnSpark(m.x, m.y, sw.color, 4);
  
  addFloater(m.x, m.y - m.size - 10, `-${dmg}`, '#ff4444');
  
  if (m.hp <= 0) {
    killMonster(m);
  }
}

function killMonster(m) {
  m.deathTimer = 0;
  spawnBlood(m.x, m.y, 18, m.type === 'slime' ? '#44ff88' : '#cc1111');
  spawnBlood(m.x, m.y - m.size/2, 12, '#880000');
  
  // XP
  gainXP(m.xp);
  
  // Gold
  const goldAmt = m.gold[0] + Math.floor(Math.random() * (m.gold[1] - m.gold[0]));
  player.gold += goldAmt;
  addFloater(m.x, m.y - m.size - 20, `+${goldAmt}💰`, '#ffd700');
  
  // Loot drop
  if (Math.random() < 0.65) {
    const lootName = m.loot[Math.floor(Math.random() * m.loot.length)];
    const lootColor = lootColors[lootName] || '#aaaaff';
    const lootEmoji = lootEmojis[lootName] || '📦';
    lootItems.push({
      x: m.x + (Math.random()-0.5)*20,
      y: m.y + (Math.random()-0.5)*20,
      name: lootName,
      color: lootColor,
      emoji: lootEmoji,
      phase: Math.random()*Math.PI*2,
      timer: 0,
    });
  }
  
  player.kills++;
  kills++;
  document.getElementById('killCount').textContent = `KILLS: ${player.kills}`;
  
  // Check map progression
  checkMapUnlock();
}

const lootColors = {
  'Slime Gel':'#44ff88','Health Potion':'#ff4488','Small Potion':'#ff6699',
  'Goblin Tooth':'#ffffaa','Iron Shard':'#aaaaaa','Elixir':'#ff88ff',
  'Orc Hide':'#886633','Steel Fragment':'#88aaff','Mega Potion':'#ff2266',
  'Demon Horn':'#ff4444','Flame Essence':'#ff8800','Soul Crystal':'#cc44ff',
  'Dragon Scale':'#ff8800','Dragon Claw':'#ff4400',
};
const lootEmojis = {
  'Slime Gel':'💚','Health Potion':'🧪','Small Potion':'🍶','Goblin Tooth':'🦷',
  'Iron Shard':'🔩','Elixir':'⚗️','Orc Hide':'🟫','Steel Fragment':'⚙️',
  'Mega Potion':'❤️‍🔥','Demon Horn':'📯','Flame Essence':'🔥','Soul Crystal':'💜',
  'Dragon Scale':'🐉','Dragon Claw':'🗝️',
};

function gainXP(amount) {
  player.xp += amount;
  addFloater(player.x, player.y - 35, `+${amount}XP`, '#b39ddb');
  while (player.xp >= player.xpNeeded) {
    player.xp -= player.xpNeeded;
    player.level++;
    player.xpNeeded = Math.floor(player.xpNeeded * 1.4);
    player.maxHp += 20;
    player.hp = Math.min(player.hp + 30, player.maxHp);
    player.speed += 0.05;
    showNotification(`★ LEVEL UP! LVL ${player.level} ★`);
    // Unlock swords
    SWORDS.forEach(sw => {
      if (sw.unlockLv === player.level && !player.unlockedSwords.includes(sw.id)) {
        player.unlockedSwords.push(sw.id);
        showNotification(`⚔ UNLOCKED: ${sw.name}!`);
        spawnAOE(player.x, player.y, 80, sw.color);
      }
    });
    updateSwordSlots();
  }
  updateUI();
}

function checkMapUnlock() {
  if (player.level >= 9 && currentMap < 2) {
    // Will transition to volcano
  } else if (player.level >= 5 && currentMap < 1) {
    // Will transition to dungeon
  }
}

function changeMap(idx) {
  currentMap = idx;
  generateMap(idx);
  player.x = W/2; player.y = H/2;
  showNotification(`📍 ${MAPS[idx].name}`);
  document.getElementById('mapDisplay').textContent = `MAP: ${MAPS[idx].name}`;
}

// ─── UPDATE UI ────────────────────────────────────────────────
function updateUI() {
  const hpPct = (player.hp / player.maxHp) * 100;
  const xpPct = (player.xp / player.xpNeeded) * 100;
  document.getElementById('hpBar').style.width = hpPct + '%';
  document.getElementById('xpBar').style.width = xpPct + '%';
  document.getElementById('hpText').textContent = `${Math.ceil(player.hp)}/${player.maxHp}`;
  document.getElementById('xpText').textContent = `${player.xp}/${player.xpNeeded}`;
  document.getElementById('levelDisplay').textContent = `LVL ${player.level}`;
  document.getElementById('goldDisplay').textContent = `💰 ${player.gold} GOLD`;
}

function updateSwordSlots() {
  const container = document.getElementById('swordSlots');
  container.innerHTML = '';
  SWORDS.forEach((sw, i) => {
    const div = document.createElement('div');
    div.className = 'sword-slot' +
      (player.sword === i ? ' active' : '') +
      (!player.unlockedSwords.includes(i) ? ' locked' : '');
    div.innerHTML = `<div>${sw.emoji}</div><div class="sword-label">${i+1}</div>`;
    if (player.unlockedSwords.includes(i)) {
      div.title = sw.name;
      div.style.cursor = 'pointer';
      div.addEventListener('click', () => switchSword(i));
    } else {
      div.title = `${sw.name} — Unlock at Lv${sw.unlockLv}`;
    }
    container.appendChild(div);
  });
}

function switchSword(idx) {
  if (!player.unlockedSwords.includes(idx)) return;
  player.sword = idx;
  updateSwordSlots();
  showNotification(`⚔ ${SWORDS[idx].name}`);
}

function showNotification(msg) {
  const el = document.getElementById('notification');
  el.textContent = msg;
  el.classList.add('show');
  clearTimeout(window._notifTimeout);
  window._notifTimeout = setTimeout(() => el.classList.remove('show'), 2000);
}

function showLootPopup(name, emoji, color) {
  const el = document.getElementById('lootPopup');
  el.innerHTML = `${emoji}<br>${name}`;
  el.style.color = color;
  el.style.borderColor = color;
  el.classList.add('show');
  clearTimeout(window._lootTimeout);
  window._lootTimeout = setTimeout(() => el.classList.remove('show'), 1500);
}

// ─── GAME LOOP ───────────────────────────────────────────────
function update() {
  if (state !== 'playing') return;
  gameTimer++;
  
  // Player movement
  let dx = 0, dy = 0;
  if (keys['ArrowLeft'] || keys['KeyA']) dx -= 1;
  if (keys['ArrowRight'] || keys['KeyD']) dx += 1;
  if (keys['ArrowUp'] || keys['KeyW']) dy -= 1;
  if (keys['ArrowDown'] || keys['KeyS']) dy += 1;
  
  if (dx !== 0 || dy !== 0) {
    const len = Math.hypot(dx, dy);
    player.vx = (dx/len) * player.speed;
    player.vy = (dy/len) * player.speed;
    player.walkTimer++;
    if (player.walkTimer > 6) { player.walkFrame++; player.walkTimer = 0; }
    if (dx !== 0) player.facing = dx > 0 ? 1 : -1;
  } else {
    player.vx *= 0.8;
    player.vy *= 0.8;
  }
  
  player.x += player.vx;
  player.y += player.vy;
  player.x = Math.max(16, Math.min(W-16, player.x));
  player.y = Math.max(20, Math.min(H-16, player.y));
  
  // Attack
  if ((keys['Space'] || mouseClick) && !player.swinging) {
    doAttack();
  }
  mouseClick = false;
  
  // Keyboard sword switching
  if (keys['Digit1']) switchSword(0);
  if (keys['Digit2']) switchSword(1);
  if (keys['Digit3']) switchSword(2);
  if (keys['Digit4']) switchSword(3);
  
  // Swing timer
  if (player.swinging) {
    player.swingTimer++;
    const sw = SWORDS[player.sword];
    if (player.swingTimer >= sw.swingFrames * 3) {
      player.swinging = false;
    }
  }
  
  player.invincible = Math.max(0, player.invincible - 1);
  
  // Map transitions
  if (player.level >= 9 && currentMap < 2 && player.kills > 0 && player.kills % 20 === 0) {
    changeMap(2);
  } else if (player.level >= 5 && currentMap < 1 && player.kills > 0 && player.kills % 15 === 0) {
    changeMap(1);
  }
  
  // Spawn monsters
  spawnTimer++;
  const maxMonsters = 6 + player.level * 2;
  if (spawnTimer >= spawnInterval && monsters.filter(m=>m.deathTimer<0).length < maxMonsters) {
    spawnTimer = 0;
    spawnInterval = Math.max(60, 200 - player.level * 8);
    // Chance for boss on higher levels
    if (player.level >= 8 && Math.random() < 0.15 && !monsters.find(m=>m.boss&&m.deathTimer<0)) {
      const mt = {...MONSTER_TYPES[4]};
      mt.hp = mt.maxHp = Math.floor(mt.hp * (1 + player.level * 0.2));
      mt.x = 50; mt.y = 50;
      mt.vx = 0; mt.vy = 0; mt.hitFlash = 0;
      mt.aiTimer = 0; mt.wanderAngle = 0; mt.attackCooldown = 0;
      mt.wobble = 0; mt.deathTimer = -1; mt.id = Math.random();
      monsters.push(mt);
      showNotification('🐉 BOSS INCOMING!');
    } else {
      spawnMonster();
    }
  }
  
  // Monster AI
  monsters.forEach(m => {
    if (m.deathTimer >= 0) {
      m.deathTimer++;
      return;
    }
    m.wobble += 1;
    m.hitFlash = Math.max(0, m.hitFlash - 1);
    m.attackCooldown = Math.max(0, m.attackCooldown - 1);
    m.aiTimer++;
    
    const dx = player.x - m.x;
    const dy = player.y - m.y;
    const dist = Math.hypot(dx, dy);
    
    if (dist < 250) {
      // Chase
      const nx = dx/dist, ny = dy/dist;
      m.vx = nx * m.speed;
      m.vy = ny * m.speed;
    } else {
      // Wander
      if (m.aiTimer % 90 === 0) m.wanderAngle = Math.random()*Math.PI*2;
      m.vx = Math.cos(m.wanderAngle) * m.speed * 0.4;
      m.vy = Math.sin(m.wanderAngle) * m.speed * 0.4;
    }
    
    m.x += m.vx;
    m.y += m.vy;
    m.x = Math.max(m.size, Math.min(W-m.size, m.x));
    m.y = Math.max(m.size, Math.min(H-m.size, m.y));
    
    // Attack player
    if (dist < m.size + 20 && m.attackCooldown === 0 && player.invincible === 0) {
      m.attackCooldown = 80;
      player.hp -= m.dmg;
      player.invincible = 60;
      player.damageFlash = 8;
      addFloater(player.x, player.y-40, `-${m.dmg}`, '#ff2222');
      spawnSpark(player.x, player.y, '#ff4444', 8);
      updateUI();
      if (player.hp <= 0) {
        player.hp = 0;
        updateUI();
        state = 'dead';
        showDeadScreen();
      }
    }
  });
  
  // Clean dead monsters
  monsters = monsters.filter(m => m.deathTimer < 0 || m.deathTimer < 40);
  
  // Loot pickup
  lootItems.forEach((l, i) => {
    l.timer++;
    const dist = Math.hypot(l.x - player.x, l.y - player.y);
    if (dist < 25) {
      showLootPopup(l.name, l.emoji, l.color || '#aaaaff');
      addFloater(l.x, l.y-20, l.name, l.color || '#aaaaff');
      lootItems.splice(i, 1);
      // Heal from potions
      if (l.name.includes('Potion') || l.name.includes('Elixir')) {
        const healAmt = l.name.includes('Mega') ? 50 : l.name.includes('Health') ? 30 : 20;
        player.hp = Math.min(player.maxHp, player.hp + healAmt);
        addFloater(player.x, player.y-50, `+${healAmt}HP`, '#44ff88');
        updateUI();
      }
    }
    if (l.timer > 600) lootItems.splice(i, 1);
  });
  
  // Update particles
  particles.forEach(p => {
    p.x += p.vx;
    p.y += p.vy;
    p.vy += p.gravity;
    p.life--;
    if (p.type === 'blood') p.vx *= 0.92;
    if (p.type === 'blood') p.size = Math.max(1, p.size - 0.04);
  });
  particles = particles.filter(p => p.life > 0);
  
  // Update floaters
  floaters.forEach(f => {
    f.y += f.vy;
    f.life--;
  });
  floaters = floaters.filter(f => f.life > 0);
}

function draw() {
  ctx.clearRect(0, 0, W, H);
  drawMap();
  drawLootItems();
  drawParticles();
  monsters.forEach(m => drawMonster(m));
  drawPlayer();
  drawFloaters();
}

function showDeadScreen() {
  const overlay = document.getElementById('overlay');
  overlay.innerHTML = `
    <h1 style="color:#ff4444;text-shadow:0 0 20px #ff4444">YOU DIED</h1>
    <p style="color:#ff8888">LEVEL ${player.level} • ${player.kills} KILLS • ${player.gold} GOLD</p>
    <button class="menu-btn" id="restartBtn">↺ TRY AGAIN</button>
    <button class="menu-btn" id="menuBtn2" style="font-size:6px">⌂ MAIN MENU</button>
  `;
  overlay.style.display = 'flex';
  document.getElementById('restartBtn').onclick = startGame;
  document.getElementById('menuBtn2').onclick = () => {
    overlay.style.display = 'flex';
    overlay.innerHTML = `
      <h1>⚔ PIXEL QUEST ⚔</h1>
      <p style="color:#7b68ee">BLADE OF ETERNITY</p>
      <p>Slay monsters, collect loot, unlock legendary swords!</p>
      <button class="menu-btn" id="startBtn2">▶ BEGIN QUEST</button>
    `;
    document.getElementById('startBtn2').onclick = startGame;
  };
}

function startGame() {
  player = {
    x: W/2, y: H/2,
    vx: 0, vy: 0,
    hp: 100, maxHp: 100,
    xp: 0, xpNeeded: 100,
    level: 1, gold: 0, kills: 0,
    sword: 0, unlockedSwords: [0],
    facing: 1, walkFrame: 0, walkTimer: 0,
    invincible: 0, swinging: false, swingAngle: 0,
    swingTimer: 0, swingDir: 1, damageFlash: 0,
    speed: 2.2,
  };
  currentMap = 0;
  kills = 0;
  gameTimer = 0;
  generateMap(0);
  updateUI();
  updateSwordSlots();
  document.getElementById('killCount').textContent = 'KILLS: 0';
  document.getElementById('mapDisplay').textContent = 'MAP: VERDANT FOREST';
  document.getElementById('overlay').style.display = 'none';
  state = 'playing';
  
  // Initial monsters
  for (let i = 0; i < 4; i++) spawnMonster();
}

function gameLoop() {
  update();
  draw();
  requestAnimationFrame(gameLoop);
}

// ─── INPUT ────────────────────────────────────────────────────
window.addEventListener('keydown', e => {
  keys[e.code] = true;
  e.preventDefault();
});
window.addEventListener('keyup', e => { keys[e.code] = false; });

canvas.addEventListener('mousemove', e => {
  const rect = canvas.getBoundingClientRect();
  const scaleX = W / rect.width;
  const scaleY = H / rect.height;
  mousePos.x = (e.clientX - rect.left) * scaleX;
  mousePos.y = (e.clientY - rect.top) * scaleY;
});

canvas.addEventListener('click', e => {
  if (state !== 'playing') return;
  const rect = canvas.getBoundingClientRect();
  const scaleX = W / rect.width;
  const mx = (e.clientX - rect.left) * scaleX;
  player.facing = mx > player.x ? 1 : -1;
  mouseClick = true;
});

document.getElementById('startBtn').onclick = startGame;

// ─── START ────────────────────────────────────────────────────
updateSwordSlots();
gameLoop();
</script>

</body>
</html>