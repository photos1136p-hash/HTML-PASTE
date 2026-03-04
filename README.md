<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8">
<title>Pixel RPG Sword Demo</title>
<style>
  html, body {
    margin: 0;
    background: #111;
    color: #eee;
    font-family: monospace;
    overflow: hidden;
  }
  #gameContainer {
    display: flex;
    flex-direction: column;
    align-items: center;
    margin-top: 8px;
  }
  canvas {
    image-rendering: pixelated;
    border: 2px solid #444;
    background: #1a1a1a;
  }
  #ui {
    margin-top: 6px;
    font-size: 14px;
    text-align: center;
    white-space: pre;
  }
  #help {
    margin-top: 4px;
    font-size: 12px;
    color: #aaa;
  }
</style>
</head>
<body>
<div id="gameContainer">
  <canvas id="game" width="256" height="144"></canvas>
  <div id="ui"></div>
  <div id="help">
    Move: WASD or Arrows | Attack: J or Space | Pickup: E<br>
    Kill all monsters to unlock next map. Loot can drop swords & gold.
  </div>
</div>

<script>
(() => {
  const canvas = document.getElementById('game');
  const ctx = canvas.getContext('2d');

  const SCALE = 4; // visual scale if you want to upscale via CSS instead
  // (we already use small resolution canvas, so this is mostly conceptual)

  // --- INPUT ---
  const keys = {};
  window.addEventListener('keydown', e => {
    keys[e.key.toLowerCase()] = true;
  });
  window.addEventListener('keyup', e => {
    keys[e.key.toLowerCase()] = false;
  });

  // --- UTIL ---
  function randRange(a, b) {
    return a + Math.random() * (b - a);
  }
  function randInt(a, b) {
    return Math.floor(randRange(a, b + 1));
  }
  function clamp(v, a, b) {
    return v < a ? a : v > b ? b : v;
  }
  function dist2(a, b) {
    const dx = a.x - b.x;
    const dy = a.y - b.y;
    return dx * dx + dy * dy;
  }

  // --- GAME DATA ---
  const swords = [
    { name: "Rusty Sword", damage: 4, cooldown: 0.45, range: 18, color: "#c0c0c0" },
    { name: "Iron Blade", damage: 7, cooldown: 0.35, range: 20, color: "#e0e0ff" },
    { name: "Crimson Edge", damage: 11, cooldown: 0.30, range: 22, color: "#ff4040" },
    { name: "Void Katana", damage: 16, cooldown: 0.24, range: 24, color: "#a040ff" },
  ];

  const maps = [
    {
      name: "Grassy Plains",
      color: "#204020",
      wallColor: "#102010",
      monsterCount: 5,
      monsterHP: 16,
      monsterDamage: 4,
      lootChanceSword: 0.15,
      lootChanceGold: 0.7,
    },
    {
      name: "Dark Forest",
      color: "#102018",
      wallColor: "#05090a",
      monsterCount: 7,
      monsterHP: 22,
      monsterDamage: 5,
      lootChanceSword: 0.2,
      lootChanceGold: 0.8,
    },
    {
      name: "Ancient Ruins",
      color: "#302828",
      wallColor: "#1a1414",
      monsterCount: 9,
      monsterHP: 30,
      monsterDamage: 6,
      lootChanceSword: 0.25,
      lootChanceGold: 0.85,
    },
  ];

  // --- ENTITIES ---
  const player = {
    x: 128,
    y: 72,
    w: 6,
    h: 8,
    speed: 55,
    vx: 0,
    vy: 0,
    hp: 40,
    maxHp: 40,
    facing: 1, // 1 right, -1 left
    swordIndex: 0,
    attackTimer: 0,
    gold: 0,
    invulnTimer: 0,
  };

  let monsters = [];
  let loot = [];
  let particles = [];
  let levelIndex = 0;
  let levelTransitionTimer = 0;
  let gameOver = false;

  function spawnLevel(idx) {
    const map = maps[idx];
    monsters = [];
    loot = [];
    particles = [];
    player.x = 128;
    player.y = 72;
    player.vx = player.vy = 0;
    player.attackTimer = 0;
    player.invulnTimer = 0;
    if (idx === 0) {
      player.hp = player.maxHp;
      player.swordIndex = 0;
      player.gold = 0;
    }

    for (let i = 0; i < map.monsterCount; i++) {
      monsters.push({
        x: randRange(20, 236),
        y: randRange(20, 124),
        w: 7,
        h: 7,
        hp: map.monsterHP,
        maxHp: map.monsterHP,
        vx: 0,
        vy: 0,
        knockbackTimer: 0,
      });
    }
  }

  function spawnBlood(x, y, count, color = "#ff4040") {
    for (let i = 0; i < count; i++) {
      const ang = Math.random() * Math.PI * 2;
      const spd = randRange(20, 60);
      particles.push({
        x,
        y,
        vx: Math.cos(ang) * spd,
        vy: Math.sin(ang) * spd,
        life: randRange(0.2, 0.6),
        maxLife: 0.6,
        color,
        size: randInt(1, 2),
      });
    }
  }

  function spawnLoot(x, y, map) {
    // gold
    if (Math.random() < map.lootChanceGold) {
      loot.push({
        x,
        y,
        type: "gold",
        amount: randInt(3, 10),
        color: "#ffd040",
      });
    }
    // sword
    if (Math.random() < map.lootChanceSword) {
      const maxIndex = Math.min(swords.length - 1, player.swordIndex + 1 + randInt(0, 1));
      const idx = randInt(player.swordIndex, maxIndex);
      loot.push({
        x,
        y,
        type: "sword",
        swordIndex: idx,
        color: swords[idx].color,
      });
    }
  }

  // --- COLLISION HELPERS ---
  function rectOverlap(a, b) {
    return (
      a.x < b.x + b.w &&
      a.x + a.w > b.x &&
      a.y < b.y + b.h &&
      a.y + a.h > b.y
    );
  }

  // --- UPDATE ---
  let lastTime = performance.now();

  function update(dt) {
    if (gameOver) return;

    const map = maps[levelIndex];

    // Input
    let moveX = 0, moveY = 0;
    if (keys["a"] || keys["arrowleft"]) moveX -= 1;
    if (keys["d"] || keys["arrowright"]) moveX += 1;
    if (keys["w"] || keys["arrowup"]) moveY -= 1;
    if (keys["s"] || keys["arrowdown"]) moveY += 1;

    const len = Math.hypot(moveX, moveY) || 1;
    moveX /= len;
    moveY /= len;

    player.vx = moveX * player.speed;
    player.vy = moveY * player.speed;

    if (moveX !== 0) player.facing = moveX > 0 ? 1 : -1;

    // Attack
    const attackKey = keys[" "] || keys["j"];
    const sword = swords[player.swordIndex];
    if (player.attackTimer > 0) {
      player.attackTimer -= dt;
    }
    if (attackKey && player.attackTimer <= 0) {
      player.attackTimer = sword.cooldown;
      // Attack hitbox
      const range = sword.range;
      const hitbox = {
        x: player.x + player.facing * (player.w / 2),
        y: player.y - 1,
        w: range * player.facing,
        h: 10,
      };
      if (hitbox.w < 0) {
        hitbox.x += hitbox.w;
        hitbox.w = -hitbox.w;
      }

      // Check monsters
      for (const m of monsters) {
        const mBox = { x: m.x - m.w / 2, y: m.y - m.h / 2, w: m.w, h: m.h };
        if (rectOverlap(hitbox, mBox)) {
          m.hp -= sword.damage;
          const kbDir = Math.sign(m.x - player.x) || player.facing;
          m.vx = kbDir * 80;
          m.vy = -20;
          m.knockbackTimer = 0.15;
          spawnBlood(m.x, m.y, randInt(4, 8));
        }
      }
    }

    // Move player
    player.x += player.vx * dt;
    player.y += player.vy * dt;

    // Clamp to map bounds
    const margin = 8;
    player.x = clamp(player.x, margin, canvas.width - margin);
    player.y = clamp(player.y, margin, canvas.height - margin);

    // Invulnerability timer
    if (player.invulnTimer > 0) {
      player.invulnTimer -= dt;
    }

    // Monsters AI
    for (const m of monsters) {
      if (m.knockbackTimer > 0) {
        m.knockbackTimer -= dt;
      } else {
        const dx = player.x - m.x;
        const dy = player.y - m.y;
        const d = Math.hypot(dx, dy) || 1;
        const speed = 25;
        m.vx = (dx / d) * speed;
        m.vy = (dy / d) * speed;
      }

      m.x += m.vx * dt;
      m.y += m.vy * dt;

      // simple friction
      m.vx *= 0.9;
      m.vy *= 0.9;

      // clamp
      m.x = clamp(m.x, margin, canvas.width - margin);
      m.y = clamp(m.y, margin, canvas.height - margin);

      // Damage player
      const mBox = { x: m.x - m.w / 2, y: m.y - m.h / 2, w: m.w, h: m.h };
      const pBox = { x: player.x - player.w / 2, y: player.y - player.h / 2, w: player.w, h: player.h };
      if (rectOverlap(mBox, pBox) && player.invulnTimer <= 0) {
        player.hp -= map.monsterDamage;
        player.invulnTimer = 0.6;
        spawnBlood(player.x, player.y, randInt(5, 10), "#ff8080");
        if (player.hp <= 0) {
          player.hp = 0;
          gameOver = true;
        }
      }
    }

    // Remove dead monsters & spawn loot
    for (let i = monsters.length - 1; i >= 0; i--) {
      if (monsters[i].hp <= 0) {
        spawnLoot(monsters[i].x, monsters[i].y, map);
        monsters.splice(i, 1);
      }
    }

    // Loot pickup
    const pickupKey = keys["e"];
    if (pickupKey) {
      const pBox = { x: player.x - 4, y: player.y - 4, w: 8, h: 8 };
      for (let i = loot.length - 1; i >= 0; i--) {
        const l = loot[i];
        const lBox = { x: l.x - 3, y: l.y - 3, w: 6, h: 6 };
        if (rectOverlap(pBox, lBox)) {
          if (l.type === "gold") {
            player.gold += l.amount;
          } else if (l.type === "sword") {
            if (l.swordIndex > player.swordIndex) {
              player.swordIndex = l.swordIndex;
            }
          }
          loot.splice(i, 1);
        }
      }
    }

    // Particles
    for (const p of particles) {
      p.x += p.vx * dt;
      p.y += p.vy * dt;
      p.vx *= 0.9;
      p.vy += 40 * dt; // gravity-ish
      p.life -= dt;
    }
    particles = particles.filter(p => p.life > 0);

    // Level progression
    if (monsters.length === 0 && levelTransitionTimer <= 0 && !gameOver) {
      levelTransitionTimer = 1.5;
    }
    if (levelTransitionTimer > 0) {
      levelTransitionTimer -= dt;
      if (levelTransitionTimer <= 0) {
        if (levelIndex < maps.length - 1) {
          levelIndex++;
          spawnLevel(levelIndex);
        } else {
          // loop last level or just stay
          levelTransitionTimer = 0;
        }
      }
    }
  }

  // --- DRAW ---
  function draw() {
    const map = maps[levelIndex];

    // Background
    ctx.fillStyle = map.color;
    ctx.fillRect(0, 0, canvas.width, canvas.height);

    // Simple border walls
    ctx.fillStyle = map.wallColor;
    ctx.fillRect(0, 0, canvas.width, 6);
    ctx.fillRect(0, canvas.height - 6, canvas.width, 6);
    ctx.fillRect(0, 0, 6, canvas.height);
    ctx.fillRect(canvas.width - 6, 0, 6, canvas.height);

    // Loot
    for (const l of loot) {
      ctx.fillStyle = l.color;
      ctx.fillRect(l.x - 3, l.y - 3, 6, 6);
      if (l.type === "sword") {
        ctx.fillStyle = "#000";
        ctx.fillRect(l.x - 1, l.y - 2, 2, 4);
      }
    }

    // Monsters
    for (const m of monsters) {
      ctx.fillStyle = "#702020";
      ctx.fillRect(m.x - m.w / 2, m.y - m.h / 2, m.w, m.h);
      // tiny HP bar
      const hpRatio = m.hp / m.maxHp;
      ctx.fillStyle = "#000000";
      ctx.fillRect(m.x - 4, m.y - m.h / 2 - 3, 8, 2);
      ctx.fillStyle = "#ff4040";
      ctx.fillRect(m.x - 4, m.y - m.h / 2 - 3, 8 * hpRatio, 2);
    }

    // Player
    const flicker = player.invulnTimer > 0 && Math.floor(player.invulnTimer * 20) % 2 === 0;
    if (!flicker) {
      ctx.fillStyle = "#40a0ff";
      ctx.fillRect(player.x - player.w / 2, player.y - player.h / 2, player.w, player.h);
      // head
      ctx.fillStyle = "#ffe0c0";
      ctx.fillRect(player.x - 2, player.y - player.h / 2 - 3, 4, 3);
    }

    // Sword swing (simple arc-ish rectangle)
    if (player.attackTimer > 0) {
      const sword = swords[player.swordIndex];
      const t = 1 - player.attackTimer / sword.cooldown; // 0..1
      const swingOffsetY = Math.sin(t * Math.PI) * 6;
      const baseX = player.x + player.facing * (player.w / 2 + 2);
      const baseY = player.y - 1 + swingOffsetY;
      ctx.fillStyle = sword.color;
      const length = sword.range;
      const thickness = 3;
      ctx.fillRect(
        baseX,
        baseY - thickness / 2,
        player.facing * length,
        thickness
      );
    }

    // Particles (blood)
    for (const p of particles) {
      const alpha = p.life / p.maxLife;
      ctx.fillStyle = p.color;
      ctx.globalAlpha = alpha;
      ctx.fillRect(p.x, p.y, p.size, p.size);
      ctx.globalAlpha = 1;
    }

    // Level transition text
    if (levelTransitionTimer > 0 && !gameOver) {
      ctx.fillStyle = "rgba(0,0,0,0.5)";
      ctx.fillRect(0, canvas.height / 2 - 10, canvas.width, 20);
      ctx.fillStyle = "#ffffff";
      ctx.textAlign = "center";
      ctx.font = "8px monospace";
      ctx.fillText("Cleared! Preparing next area...", canvas.width / 2, canvas.height / 2 + 3);
    }

    // Game over
    if (gameOver) {
      ctx.fillStyle = "rgba(0,0,0,0.7)";
      ctx.fillRect(0, 0, canvas.width, canvas.height);
      ctx.fillStyle = "#ff4040";
      ctx.textAlign = "center";
      ctx.font = "10px monospace";
      ctx.fillText("YOU DIED", canvas.width / 2, canvas.height / 2 - 4);
      ctx.font = "7px monospace";
      ctx.fillStyle = "#ffffff";
      ctx.fillText("Refresh page to restart", canvas.width / 2, canvas.height / 2 + 8);
    }

    // UI text
    const ui = document.getElementById('ui');
    const sword = swords[player.swordIndex];
    ui.textContent =
      `Area: ${map.name} (Level ${levelIndex + 1}/${maps.length})` +
      ` | HP: ${player.hp}/${player.maxHp}` +
      ` | Sword: ${sword.name}` +
      ` | DMG: ${sword.damage}` +
      ` | Gold: ${player.gold}`;
  }

  // --- MAIN LOOP ---
  function loop(now) {
    const dt = Math.min(0.05, (now - lastTime) / 1000);
    lastTime = now;
    update(dt);
    draw();
    requestAnimationFrame(loop);
  }

  // Start
  spawnLevel(levelIndex);
  requestAnimationFrame(loop);
})();
</script>
</body>
</html>
