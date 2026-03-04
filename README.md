<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1.0" />
  <title>Pixel Blade Hunt</title>
  <style>
    :root {
      --bg: #121018;
      --panel: #1d1a2a;
      --panel-border: #4b3f62;
      --text: #f7f4ff;
      --accent: #70e0ff;
    }

    * { box-sizing: border-box; }

    body {
      margin: 0;
      min-height: 100vh;
      background: radial-gradient(circle at top, #2a2340 0%, var(--bg) 52%);
      color: var(--text);
      font-family: "Courier New", Courier, monospace;
      display: grid;
      place-items: center;
      padding: 1rem;
    }

    .wrapper {
      width: min(1000px, 100%);
      display: grid;
      gap: 0.75rem;
    }

    .hud {
      background: var(--panel);
      border: 3px solid var(--panel-border);
      border-radius: 0;
      padding: 0.75rem;
      display: grid;
      grid-template-columns: repeat(4, minmax(0, 1fr));
      gap: 0.5rem;
      box-shadow: 0 8px 0 #0d0a14;
      image-rendering: pixelated;
    }

    .hud > div {
      background: #171322;
      border: 2px solid #39304f;
      padding: 0.4rem;
      min-height: 52px;
      display: grid;
      gap: 0.15rem;
    }

    .label { color: #b6abd1; font-size: 0.78rem; text-transform: uppercase; }
    .value { color: var(--accent); font-weight: 700; letter-spacing: 0.4px; }

    .canvas-wrap {
      border: 4px solid #6f6094;
      box-shadow: 0 12px 0 #191322;
      background: #0d0b13;
    }

    canvas {
      width: 100%;
      height: auto;
      display: block;
      image-rendering: pixelated;
      image-rendering: crisp-edges;
      background: #0f1220;
    }

    .controls {
      background: var(--panel);
      border: 3px solid var(--panel-border);
      padding: 0.75rem;
      font-size: 0.88rem;
      line-height: 1.35;
    }

    .win {
      color: #9dff9d;
      text-shadow: 0 0 7px #31a331;
    }

    @media (max-width: 820px) {
      .hud { grid-template-columns: repeat(2, minmax(0, 1fr)); }
    }
  </style>
</head>
<body>
  <main class="wrapper">
    <section class="hud">
      <div><span class="label">Health</span><span id="health" class="value">100 / 100</span></div>
      <div><span class="label">Level</span><span id="level" class="value">1</span></div>
      <div><span class="label">XP</span><span id="xp" class="value">0 / 30</span></div>
      <div><span class="label">Gold</span><span id="gold" class="value">0</span></div>
      <div><span class="label">Sword</span><span id="sword" class="value">Rusty Starter</span></div>
      <div><span class="label">Map</span><span id="map" class="value">Gloom Meadow</span></div>
      <div><span class="label">Monsters Left</span><span id="monsters" class="value">0</span></div>
      <div><span class="label">Status</span><span id="status" class="value">Survive and clear each map.</span></div>
    </section>

    <section class="canvas-wrap">
      <canvas id="game" width="960" height="540" aria-label="Pixel Blade Hunt game"></canvas>
    </section>

    <section class="controls">
      <strong>Controls:</strong> Move with <b>WASD</b> or <b>Arrow Keys</b>, swing with <b>Space</b>, drink potion with <b>Q</b>.<br />
      Defeat monsters to gain XP and loot. New swords unlock at levels 2, 4, and 6.
      Clear a map to warp to the next arena. Beat all maps to win.
    </section>
  </main>

  <script>
    (() => {
      const canvas = document.getElementById("game");
      const ctx = canvas.getContext("2d");
      ctx.imageSmoothingEnabled = false;

      const hud = {
        health: document.getElementById("health"),
        level: document.getElementById("level"),
        xp: document.getElementById("xp"),
        gold: document.getElementById("gold"),
        sword: document.getElementById("sword"),
        map: document.getElementById("map"),
        monsters: document.getElementById("monsters"),
        status: document.getElementById("status")
      };

      const TILE = 32;
      const WORLD_W = canvas.width;
      const WORLD_H = canvas.height;

      const swords = [
        { name: "Rusty Starter", unlockLevel: 1, damage: 20, cooldown: 380, range: 44, color: "#b3a58a" },
        { name: "Frost Fang", unlockLevel: 2, damage: 28, cooldown: 325, range: 50, color: "#9dd8ff" },
        { name: "Ember Cleaver", unlockLevel: 4, damage: 38, cooldown: 280, range: 54, color: "#ff9f69" },
        { name: "Nebula Edge", unlockLevel: 6, damage: 50, cooldown: 240, range: 58, color: "#c192ff" }
      ];

      const maps = [
        { name: "Gloom Meadow", floor: ["#2f5930", "#35633a"], wall: "#1f3621", monsterCount: 8, tint: "rgba(70, 130, 90, 0.18)" },
        { name: "Ashen Catacombs", floor: ["#48435f", "#534f6f"], wall: "#2c2840", monsterCount: 11, tint: "rgba(150, 80, 90, 0.2)" },
        { name: "Arcane Rift", floor: ["#3c2f5a", "#4a3d70"], wall: "#221834", monsterCount: 14, tint: "rgba(90, 80, 180, 0.22)" }
      ];

      const state = {
        keys: new Set(),
        lastTime: 0,
        mapIndex: 0,
        gameWon: false,
        messageUntil: 0,
        cameraShake: 0,
        player: {
          x: WORLD_W / 2,
          y: WORLD_H / 2,
          radius: 13,
          speed: 170,
          hp: 100,
          maxHp: 100,
          level: 1,
          xp: 0,
          xpToNext: 30,
          gold: 0,
          facing: 0,
          attackT: 0,
          lastAttackAt: -999,
          hitFlash: 0,
          potions: 0
        },
        monsters: [],
        drops: [],
        particles: []
      };

      function rand(min, max) {
        return Math.random() * (max - min) + min;
      }

      function currentSword() {
        const unlocked = swords.filter((s) => state.player.level >= s.unlockLevel);
        return unlocked[unlocked.length - 1];
      }

      function spawnMonsters(count) {
        state.monsters = [];
        for (let i = 0; i < count; i++) {
          const edge = Math.floor(Math.random() * 4);
          let x = rand(40, WORLD_W - 40);
          let y = rand(40, WORLD_H - 40);
          if (edge === 0) x = rand(0, 100);
          if (edge === 1) x = rand(WORLD_W - 100, WORLD_W);
          if (edge === 2) y = rand(0, 100);
          if (edge === 3) y = rand(WORLD_H - 100, WORLD_H);

          const eliteChance = Math.random() < Math.min(0.1 + state.mapIndex * 0.08, 0.35);
          state.monsters.push({
            x,
            y,
            radius: eliteChance ? 14 : 11,
            hp: eliteChance ? 95 + state.mapIndex * 15 : 55 + state.mapIndex * 12,
            maxHp: eliteChance ? 95 + state.mapIndex * 15 : 55 + state.mapIndex * 12,
            speed: eliteChance ? 78 + state.mapIndex * 6 : 62 + state.mapIndex * 5,
            damage: eliteChance ? 16 + state.mapIndex * 2 : 10 + state.mapIndex,
            hitFlash: 0,
            attackCooldown: rand(0.1, 0.45),
            elite: eliteChance
          });
        }
      }

      function setMap(index) {
        state.mapIndex = index;
        const map = maps[index];
        state.drops = [];
        state.particles = [];
        spawnMonsters(map.monsterCount);
        state.player.x = WORLD_W / 2;
        state.player.y = WORLD_H / 2;
        state.player.hp = Math.min(state.player.hp + 20, state.player.maxHp);
        flashStatus(`Entered ${map.name}`);
      }

      function flashStatus(text, ms = 2200) {
        hud.status.textContent = text;
        state.messageUntil = performance.now() + ms;
      }

      function gainXp(amount) {
        state.player.xp += amount;
        while (state.player.xp >= state.player.xpToNext) {
          state.player.xp -= state.player.xpToNext;
          state.player.level += 1;
          state.player.maxHp += 10;
          state.player.hp = Math.min(state.player.maxHp, state.player.hp + 20);
          state.player.xpToNext = Math.floor(state.player.xpToNext * 1.32);
          flashStatus(`Level up! You are now level ${state.player.level}.`, 2600);
        }
      }

      function spawnLoot(x, y) {
        const roll = Math.random();
        let type = "coin";
        if (roll > 0.78) type = "potion";
        else if (roll > 0.56) type = "shard";

        state.drops.push({
          x: x + rand(-8, 8),
          y: y + rand(-8, 8),
          radius: 8,
          type,
          life: 12
        });
      }

      function spawnBlood(x, y, count = 16) {
        for (let i = 0; i < count; i++) {
          const angle = rand(0, Math.PI * 2);
          const speed = rand(40, 220);
          state.particles.push({
            x,
            y,
            vx: Math.cos(angle) * speed,
            vy: Math.sin(angle) * speed,
            size: rand(2, 4),
            life: rand(0.35, 0.85),
            maxLife: 1,
            color: Math.random() < 0.22 ? "#500910" : "#a6111c"
          });
        }
      }

      function circleHit(a, b, extra = 0) {
        const dx = a.x - b.x;
        const dy = a.y - b.y;
        return dx * dx + dy * dy < (a.radius + b.radius + extra) ** 2;
      }

      function update(dt, now) {
        const p = state.player;
        if (p.hp <= 0 || state.gameWon) return;

        let mx = 0;
        let my = 0;
        if (state.keys.has("ArrowLeft") || state.keys.has("a")) mx -= 1;
        if (state.keys.has("ArrowRight") || state.keys.has("d")) mx += 1;
        if (state.keys.has("ArrowUp") || state.keys.has("w")) my -= 1;
        if (state.keys.has("ArrowDown") || state.keys.has("s")) my += 1;

        if (mx !== 0 || my !== 0) {
          const len = Math.hypot(mx, my);
          mx /= len;
          my /= len;
          p.facing = Math.atan2(my, mx);
          p.x += mx * p.speed * dt;
          p.y += my * p.speed * dt;
        }

        p.x = Math.max(16, Math.min(WORLD_W - 16, p.x));
        p.y = Math.max(16, Math.min(WORLD_H - 16, p.y));

        const sword = currentSword();
        if (p.attackT > 0) p.attackT = Math.max(0, p.attackT - dt * 3.9);

        if (state.keys.has(" ") && now - p.lastAttackAt >= sword.cooldown) {
          p.lastAttackAt = now;
          p.attackT = 1;
          state.cameraShake = 6;
          const attackAngle = p.facing;

          state.monsters.forEach((m) => {
            const dx = m.x - p.x;
            const dy = m.y - p.y;
            const dist = Math.hypot(dx, dy);
            const angle = Math.atan2(dy, dx);
            const offset = Math.atan2(Math.sin(angle - attackAngle), Math.cos(angle - attackAngle));

            if (dist < sword.range + m.radius && Math.abs(offset) < 0.9) {
              const crit = Math.random() < 0.14;
              const damage = sword.damage + (crit ? sword.damage * 0.55 : 0);
              m.hp -= damage;
              m.hitFlash = 0.12;
              m.x += Math.cos(angle) * 14;
              m.y += Math.sin(angle) * 14;
              spawnBlood(m.x, m.y, crit ? 26 : 14);
            }
          });
        }

        state.monsters.forEach((m) => {
          const dx = p.x - m.x;
          const dy = p.y - m.y;
          const d = Math.hypot(dx, dy) || 1;
          m.x += (dx / d) * m.speed * dt;
          m.y += (dy / d) * m.speed * dt;
          m.hitFlash = Math.max(0, m.hitFlash - dt);
          m.attackCooldown -= dt;

          if (circleHit(p, m) && m.attackCooldown <= 0) {
            m.attackCooldown = m.elite ? 0.75 : 0.95;
            p.hp -= m.damage;
            p.hitFlash = 0.2;
            state.cameraShake = 9;
            if (p.hp <= 0) {
              p.hp = 0;
              flashStatus("You were defeated. Refresh to retry.", 6000);
            }
          }
        });

        const killed = state.monsters.filter((m) => m.hp <= 0);
        if (killed.length) {
          killed.forEach((m) => {
            gainXp(m.elite ? 20 : 10);
            state.player.gold += m.elite ? 16 : 7;
            spawnLoot(m.x, m.y);
            if (Math.random() < (m.elite ? 0.6 : 0.25)) spawnLoot(m.x + 10, m.y + 8);
          });
          state.monsters = state.monsters.filter((m) => m.hp > 0);
        }

        state.drops.forEach((drop) => {
          drop.life -= dt;
          if (circleHit(p, drop, 2)) {
            if (drop.type === "coin") p.gold += 5;
            if (drop.type === "shard") gainXp(12);
            if (drop.type === "potion") p.potions += 1;
            drop.life = 0;
          }
        });
        state.drops = state.drops.filter((d) => d.life > 0);

        if (state.keys.has("q") && p.potions > 0 && p.hp < p.maxHp) {
          p.potions -= 1;
          p.hp = Math.min(p.maxHp, p.hp + 35);
          spawnBlood(p.x, p.y, 10);
          flashStatus("Potion chugged. +35 HP", 1400);
          state.keys.delete("q");
        }

        state.particles.forEach((part) => {
          part.life -= dt;
          part.x += part.vx * dt;
          part.y += part.vy * dt;
          part.vx *= 0.92;
          part.vy = part.vy * 0.92 + 420 * dt;
        });
        state.particles = state.particles.filter((p2) => p2.life > 0);

        if (state.monsters.length === 0) {
          if (state.mapIndex < maps.length - 1) {
            setMap(state.mapIndex + 1);
          } else {
            state.gameWon = true;
            hud.status.innerHTML = "<span class='win'>All maps cleared! You are the Blade Champion.</span>";
          }
        }

        state.cameraShake = Math.max(0, state.cameraShake - dt * 20);
      }

      function drawGrid(map) {
        for (let y = 0; y < WORLD_H; y += TILE) {
          for (let x = 0; x < WORLD_W; x += TILE) {
            const i = ((x / TILE + y / TILE) & 1);
            ctx.fillStyle = map.floor[i];
            ctx.fillRect(x, y, TILE, TILE);
          }
        }

        ctx.fillStyle = map.wall;
        ctx.fillRect(0, 0, WORLD_W, 12);
        ctx.fillRect(0, WORLD_H - 12, WORLD_W, 12);
        ctx.fillRect(0, 0, 12, WORLD_H);
        ctx.fillRect(WORLD_W - 12, 0, 12, WORLD_H);

        ctx.fillStyle = map.tint;
        ctx.fillRect(0, 0, WORLD_W, WORLD_H);
      }

      function drawPlayer() {
        const p = state.player;
        const sword = currentSword();

        ctx.save();
        if (p.hitFlash > 0) {
          ctx.fillStyle = "#ffdbdb";
        } else {
          ctx.fillStyle = "#80b9ff";
        }
        ctx.fillRect(Math.floor(p.x - 10), Math.floor(p.y - 12), 20, 24);
        ctx.fillStyle = "#243f7a";
        ctx.fillRect(Math.floor(p.x - 7), Math.floor(p.y - 17), 14, 6);
        ctx.restore();

        const t = p.attackT;
        let swingOffset = 0;
        if (t > 0) {
          const curve = 1 - (1 - t) ** 2;
          swingOffset = (-1.2 + curve * 2.4);
        }
        const angle = p.facing + swingOffset;
        const sx = p.x + Math.cos(angle) * 18;
        const sy = p.y + Math.sin(angle) * 18;

        ctx.save();
        ctx.translate(sx, sy);
        ctx.rotate(angle);
        ctx.fillStyle = sword.color;
        ctx.fillRect(0, -3, 26, 6);
        ctx.fillStyle = "#31313d";
        ctx.fillRect(-4, -4, 5, 8);
        if (t > 0) {
          ctx.strokeStyle = "rgba(255,255,255,0.55)";
          ctx.lineWidth = 3;
          ctx.beginPath();
          ctx.arc(0, 0, 42, -0.8, 0.8);
          ctx.stroke();
        }
        ctx.restore();
      }

      function drawMonsters() {
        state.monsters.forEach((m) => {
          ctx.fillStyle = m.hitFlash > 0 ? "#ffd0d0" : m.elite ? "#b970ff" : "#7ad264";
          ctx.fillRect(Math.floor(m.x - m.radius), Math.floor(m.y - m.radius), m.radius * 2, m.radius * 2);
          ctx.fillStyle = "#220f2b";
          ctx.fillRect(Math.floor(m.x - 6), Math.floor(m.y - 4), 4, 4);
          ctx.fillRect(Math.floor(m.x + 2), Math.floor(m.y - 4), 4, 4);

          const barW = 24;
          const hpRatio = Math.max(0, m.hp / m.maxHp);
          ctx.fillStyle = "#1b1b1b";
          ctx.fillRect(Math.floor(m.x - barW / 2), Math.floor(m.y - m.radius - 8), barW, 4);
          ctx.fillStyle = m.elite ? "#d794ff" : "#a5ff8d";
          ctx.fillRect(Math.floor(m.x - barW / 2), Math.floor(m.y - m.radius - 8), barW * hpRatio, 4);
        });
      }

      function drawDrops() {
        state.drops.forEach((d) => {
          if (d.type === "coin") ctx.fillStyle = "#ffd74d";
          if (d.type === "shard") ctx.fillStyle = "#67f2ff";
          if (d.type === "potion") ctx.fillStyle = "#ff6d8a";
          ctx.fillRect(Math.floor(d.x - d.radius / 2), Math.floor(d.y - d.radius / 2), d.radius, d.radius);
        });
      }

      function drawParticles() {
        state.particles.forEach((part) => {
          const alpha = Math.max(0, part.life / part.maxLife);
          ctx.fillStyle = part.color + Math.floor(alpha * 255).toString(16).padStart(2, "0");
          ctx.fillRect(Math.floor(part.x), Math.floor(part.y), part.size, part.size);
        });
      }

      function render() {
        const map = maps[state.mapIndex];
        const shakeX = state.cameraShake > 0 ? rand(-state.cameraShake, state.cameraShake) : 0;
        const shakeY = state.cameraShake > 0 ? rand(-state.cameraShake, state.cameraShake) : 0;

        ctx.save();
        ctx.clearRect(0, 0, WORLD_W, WORLD_H);
        ctx.translate(shakeX, shakeY);

        drawGrid(map);
        drawDrops();
        drawParticles();
        drawMonsters();
        drawPlayer();

        if (state.player.hp <= 0) {
          ctx.fillStyle = "rgba(20, 0, 0, 0.62)";
          ctx.fillRect(0, 0, WORLD_W, WORLD_H);
          ctx.fillStyle = "#ff9f9f";
          ctx.font = "28px Courier New";
          ctx.fillText("You fell in battle", WORLD_W / 2 - 130, WORLD_H / 2);
        }

        if (state.gameWon) {
          ctx.fillStyle = "rgba(0, 20, 10, 0.58)";
          ctx.fillRect(0, 0, WORLD_W, WORLD_H);
          ctx.fillStyle = "#a6ffcc";
          ctx.font = "28px Courier New";
          ctx.fillText("Victory! Refresh to replay.", WORLD_W / 2 - 165, WORLD_H / 2);
        }

        ctx.restore();
      }

      function updateHud(now) {
        const p = state.player;
        hud.health.textContent = `${Math.ceil(p.hp)} / ${p.maxHp} (${p.potions} potion${p.potions === 1 ? "" : "s"})`;
        hud.level.textContent = String(p.level);
        hud.xp.textContent = `${Math.floor(p.xp)} / ${p.xpToNext}`;
        hud.gold.textContent = String(p.gold);
        hud.sword.textContent = `${currentSword().name} (${currentSword().damage} dmg)`;
        hud.map.textContent = maps[state.mapIndex].name;
        hud.monsters.textContent = String(state.monsters.length);
        if (!state.gameWon && p.hp > 0 && now > state.messageUntil) {
          hud.status.textContent = "Hunt all monsters to unlock the next map.";
        }
      }

      function loop(ts) {
        if (!state.lastTime) state.lastTime = ts;
        const dt = Math.min((ts - state.lastTime) / 1000, 0.033);
        state.lastTime = ts;

        update(dt, ts);
        render();
        updateHud(ts);
        requestAnimationFrame(loop);
      }

      window.addEventListener("keydown", (e) => {
        const key = e.key.length === 1 ? e.key.toLowerCase() : e.key;
        state.keys.add(key);
        if (["ArrowUp", "ArrowDown", "ArrowLeft", "ArrowRight", " "].includes(e.key)) e.preventDefault();
      });

      window.addEventListener("keyup", (e) => {
        const key = e.key.length === 1 ? e.key.toLowerCase() : e.key;
        state.keys.delete(key);
      });

      setMap(0);
      requestAnimationFrame(loop);
    })();
  </script>
</body>
</html>
