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
      --good: #98ffb0;
      --bad: #ff9fa7;
    }
    * { box-sizing: border-box; }
    body {
      margin: 0;
      min-height: 100vh;
      background: radial-gradient(circle at top, #2a2340 0%, var(--bg) 52%);
      color: var(--text);
      font-family: "Courier New", monospace;
      display: grid;
      place-items: center;
      padding: 1rem;
    }
    .wrapper { width: min(1080px, 100%); display: grid; gap: .75rem; }
    .hud {
      background: var(--panel);
      border: 3px solid var(--panel-border);
      padding: .7rem;
      display: grid;
      grid-template-columns: repeat(5, minmax(0, 1fr));
      gap: .4rem;
      box-shadow: 0 8px 0 #0d0a14;
      image-rendering: pixelated;
    }
    .hud > div {
      background: #171322;
      border: 2px solid #39304f;
      min-height: 52px;
      padding: .35rem;
      display: grid;
      gap: .1rem;
    }
    .label { color: #b6abd1; font-size: .72rem; text-transform: uppercase; }
    .value { color: var(--accent); font-weight: 700; font-size: .95rem; }
    .canvas-wrap { position: relative; border: 4px solid #6f6094; box-shadow: 0 12px 0 #191322; }
    canvas { width: 100%; height: auto; display: block; image-rendering: pixelated; background: #0f1220; }

    .overlay {
      position: absolute; inset: 0; display: none; align-items: center; justify-content: center;
      background: rgba(11, 8, 18, 0.8); padding: 1rem;
    }
    .overlay.show { display: flex; }
    .panel {
      width: min(760px, 100%);
      background: #191428;
      border: 3px solid #6d5b94;
      box-shadow: 0 8px 0 #0d0a14;
      padding: 1rem;
    }
    .panel h2 { margin: 0 0 .5rem; color: #beecff; }
    .panel p { margin: .25rem 0 .7rem; color: #d5cfff; }
    .btn-row { display: flex; flex-wrap: wrap; gap: .45rem; margin-top: .6rem; }
    button {
      border: 2px solid #4a3e66; background: #2a2040; color: #eff5ff;
      font-family: inherit; padding: .45rem .7rem; cursor: pointer;
    }
    button:hover { background: #3a2c57; }
    .shop-grid { display: grid; grid-template-columns: repeat(2, minmax(0, 1fr)); gap: .45rem; }
    .shop-item { background: #151126; border: 2px solid #3b3055; padding: .45rem; }
    .shop-item b { color: #95f3ff; }
    .shop-item small { color: #baaed7; display: block; margin-top: .2rem; }
    .ok { color: var(--good); }
    .warn { color: #ffd983; }
    .bad { color: var(--bad); }
    .controls { background: var(--panel); border: 3px solid var(--panel-border); padding: .75rem; font-size: .9rem; }
    @media (max-width: 900px) {
      .hud { grid-template-columns: repeat(2, minmax(0, 1fr)); }
      .shop-grid { grid-template-columns: 1fr; }
    }
  </style>
</head>
<body>
  <main class="wrapper">
    <section class="hud">
      <div><span class="label">HP</span><span id="health" class="value">100 / 100</span></div>
      <div><span class="label">Level</span><span id="level" class="value">1</span></div>
      <div><span class="label">XP</span><span id="xp" class="value">0 / 30</span></div>
      <div><span class="label">Gold</span><span id="gold" class="value">0</span></div>
      <div><span class="label">Crystals</span><span id="crystals" class="value">0</span></div>
      <div><span class="label">Weapon</span><span id="weapon" class="value">Rusty Starter</span></div>
      <div><span class="label">Ability</span><span id="ability" class="value">None</span></div>
      <div><span class="label">Map</span><span id="map" class="value">-</span></div>
      <div><span class="label">Monsters Left</span><span id="monsters" class="value">0</span></div>
      <div><span class="label">Status</span><span id="status" class="value">Press Start in menu</span></div>
    </section>

    <section class="canvas-wrap">
      <canvas id="game" width="1024" height="576" aria-label="Pixel RPG"></canvas>

      <div id="menuOverlay" class="overlay show">
        <div class="panel">
          <h2>⚔ Pixel Blade Hunt</h2>
          <p>A fast 2D pixel RPG with sword combos, blood splashes, loot, and map progression.</p>
          <p><span class="warn">Menu GUI:</span> Start here, then level up to open the item shop screen.</p>
          <div class="btn-row">
            <button id="startBtn">Start Run (Enter)</button>
          </div>
        </div>
      </div>

      <div id="shopOverlay" class="overlay">
        <div class="panel">
          <h2>🛒 Crystal Armory</h2>
          <p id="shopInfo">Spend crystals from level-ups to buy stronger gear and abilities.</p>
          <div id="shopGrid" class="shop-grid"></div>
          <div class="btn-row">
            <button id="continueBtn">Continue Battle (Esc)</button>
          </div>
        </div>
      </div>
    </section>

    <section class="controls">
      <b>Controls:</b> Move with WASD / Arrow keys • Attack with Space • Potion with Q • Open Shop with B (if you have crystals) • Enter to start.
    </section>
  </main>

  <script>
    (() => {
      const canvas = document.getElementById("game");
      const ctx = canvas.getContext("2d");
      ctx.imageSmoothingEnabled = false;

      const overlays = {
        menu: document.getElementById("menuOverlay"),
        shop: document.getElementById("shopOverlay")
      };
      const shopGrid = document.getElementById("shopGrid");
      const shopInfo = document.getElementById("shopInfo");

      const hud = {
        health: document.getElementById("health"), level: document.getElementById("level"), xp: document.getElementById("xp"),
        gold: document.getElementById("gold"), crystals: document.getElementById("crystals"), weapon: document.getElementById("weapon"),
        ability: document.getElementById("ability"), map: document.getElementById("map"), monsters: document.getElementById("monsters"),
        status: document.getElementById("status")
      };

      const swords = [
        { id: "rusty", name: "Rusty Starter", damage: 18, cooldown: 380, range: 44, color: "#b3a58a", owned: true },
        { id: "iron", name: "Iron Fang", damage: 27, cooldown: 330, range: 50, color: "#bad8ff", owned: false },
        { id: "knight", name: "Knight Splitter", damage: 37, cooldown: 295, range: 56, color: "#8cedff", owned: false },
        { id: "dragon", name: "Dragon Reaver", damage: 52, cooldown: 255, range: 63, color: "#ffa861", owned: false }
      ];

      const maps = [
        ["Gloom Meadow", ["#2f5930", "#35633a"], "#1f3621", 7],
        ["Moss Hollow", ["#2c5f4a", "#387257"], "#1f3f34", 9],
        ["Moon Marsh", ["#2f4f63", "#345b72"], "#21394a", 10],
        ["Ashen Catacombs", ["#4b4561", "#5a5372"], "#302a43", 12],
        ["Crimson Vault", ["#5b3648", "#70445a"], "#3e2230", 13],
        ["Frozen Ruins", ["#38506a", "#45617d"], "#26384d", 15],
        ["Storm Citadel", ["#3f416b", "#52548a"], "#2b2d49", 16],
        ["Void Garden", ["#3f2f59", "#4f3b73"], "#291c3e", 18],
        ["Inferno Gate", ["#69362e", "#7e463b"], "#4b251f", 20],
        ["Nebula Crown", ["#332658", "#423072"], "#241841", 22]
      ].map(([name, floor, wall, count], i) => ({
        name, floor, wall, count,
        tint: `rgba(${70 + i * 10}, ${70 + ((i * 17) % 90)}, ${90 + ((i * 23) % 120)}, 0.18)`
      }));

      const shopItems = [
        { id: "buy_iron", name: "Unlock Iron Fang", desc: "+Damage and faster swings", cost: 1, once: true, action: () => unlockSword("iron") },
        { id: "buy_knight", name: "Unlock Knight Splitter", desc: "High damage slash", cost: 2, once: true, action: () => unlockSword("knight") },
        { id: "buy_dragon", name: "Unlock Dragon Reaver", desc: "Huge power weapon", cost: 3, once: true, action: () => unlockSword("dragon") },
        { id: "fire", name: "Fire Slash Ability", desc: "Attacks ignite enemies for burn damage", cost: 2, once: true, action: () => state.player.fireSlash = true },
        { id: "maxhp", name: "Vital Core", desc: "+20 Max HP (repeatable)", cost: 1, once: false, action: () => { state.player.maxHp += 20; state.player.hp += 20; } },
        { id: "speed", name: "Dash Boots", desc: "+18 Move Speed (repeatable)", cost: 1, once: false, action: () => { state.player.speed += 18; } }
      ];

      const state = {
        screen: "menu",
        mapIndex: 0,
        keys: new Set(),
        lastTime: 0,
        cameraShake: 0,
        statusUntil: 0,
        monsters: [], drops: [], particles: [],
        player: {
          x: canvas.width / 2, y: canvas.height / 2, radius: 13, speed: 170,
          hp: 100, maxHp: 100, level: 1, xp: 0, xpToNext: 30, gold: 0, crystals: 0,
          facing: 0, attackT: 0, lastAttackAt: -999, potions: 0, hitFlash: 0,
          ownedSwords: new Set(["rusty"]), equippedSword: "rusty", fireSlash: false
        }
      };

      const rand = (a, b) => Math.random() * (b - a) + a;
      const now = () => performance.now();

      const currentSword = () => swords.find((s) => s.id === state.player.equippedSword) || swords[0];
      const circleHit = (a, b, ext = 0) => ((a.x - b.x) ** 2 + (a.y - b.y) ** 2) < (a.radius + b.radius + ext) ** 2;

      function unlockSword(id) {
        state.player.ownedSwords.add(id);
        const sword = swords.find((s) => s.id === id);
        if (sword) {
          sword.owned = true;
          state.player.equippedSword = id;
          status(`Unlocked ${sword.name}`);
        }
      }

      function spawnMonsters(count) {
        state.monsters = [];
        for (let i = 0; i < count; i++) {
          const elite = Math.random() < Math.min(0.08 + state.mapIndex * 0.03, 0.3);
          state.monsters.push({
            x: rand(40, canvas.width - 40), y: rand(40, canvas.height - 40),
            radius: elite ? 14 : 11,
            hp: elite ? 105 + state.mapIndex * 14 : 58 + state.mapIndex * 10,
            maxHp: elite ? 105 + state.mapIndex * 14 : 58 + state.mapIndex * 10,
            speed: elite ? 86 + state.mapIndex * 3 : 66 + state.mapIndex * 2,
            damage: elite ? 17 + Math.floor(state.mapIndex / 2) : 10 + Math.floor(state.mapIndex / 3),
            elite,
            burn: 0,
            hitFlash: 0,
            attackCd: rand(0.15, 0.6)
          });
        }
      }

      function setMap(i) {
        state.mapIndex = i;
        state.player.x = canvas.width / 2;
        state.player.y = canvas.height / 2;
        state.player.hp = Math.min(state.player.maxHp, state.player.hp + 25);
        state.drops = [];
        state.particles = [];
        spawnMonsters(maps[i].count);
        status(`Entered ${maps[i].name}`);
      }

      function status(text, ms = 2200) {
        hud.status.textContent = text;
        state.statusUntil = now() + ms;
      }

      function spawnBlood(x, y, count = 16, color = "#a6111c") {
        for (let i = 0; i < count; i++) {
          const angle = rand(0, Math.PI * 2);
          const speed = rand(30, 220);
          state.particles.push({
            x, y,
            vx: Math.cos(angle) * speed,
            vy: Math.sin(angle) * speed,
            life: rand(0.35, 0.8),
            maxLife: 1,
            size: rand(2, 4),
            color
          });
        }
      }

      function gainXp(amount) {
        const p = state.player;
        p.xp += amount;
        while (p.xp >= p.xpToNext) {
          p.xp -= p.xpToNext;
          p.level += 1;
          p.crystals += 1;
          p.maxHp += 8;
          p.hp = Math.min(p.maxHp, p.hp + 18);
          p.xpToNext = Math.floor(p.xpToNext * 1.3);
          openShop(`Level ${p.level}! Spend your crystal now.`);
        }
      }

      function spawnLoot(x, y) {
        const r = Math.random();
        let type = "coin";
        if (r > 0.82) type = "potion";
        else if (r > 0.58) type = "shard";
        state.drops.push({ x: x + rand(-9, 9), y: y + rand(-9, 9), radius: 8, type, life: 14 });
      }

      function openShop(msg) {
        if (state.screen === "gameover" || state.screen === "victory") return;
        state.screen = "shop";
        overlays.shop.classList.add("show");
        shopInfo.textContent = msg || "Spend crystals from level-ups.";
        renderShop();
      }

      function closeShop() {
        if (state.screen !== "shop") return;
        overlays.shop.classList.remove("show");
        state.screen = "playing";
      }

      function renderShop() {
        shopGrid.innerHTML = "";
        shopItems.forEach((item) => {
          const bought = !!item.bought;
          const ownedSword = item.id.startsWith("buy_") && state.player.ownedSwords.has(item.id.replace("buy_", ""));
          const done = bought || ownedSword;
          const card = document.createElement("div");
          card.className = "shop-item";
          const canBuy = !done && state.player.crystals >= item.cost;
          card.innerHTML = `<b>${item.name}</b> <span class="${canBuy ? "ok" : "warn"}">(${item.cost} crystal)</span><small>${item.desc}</small>`;
          const btn = document.createElement("button");
          btn.textContent = done ? "Owned" : canBuy ? "Buy" : "Need More Crystals";
          btn.disabled = done || !canBuy;
          btn.onclick = () => {
            if (state.player.crystals < item.cost) return;
            state.player.crystals -= item.cost;
            item.action();
            if (item.once) item.bought = true;
            renderShop();
          };
          card.appendChild(btn);

          if (!done && item.id.startsWith("buy_") && state.player.ownedSwords.has(item.id.replace("buy_", ""))) {
            item.bought = true;
          }

          shopGrid.appendChild(card);
        });

        const owned = swords.filter((s) => state.player.ownedSwords.has(s.id));
        const equipCard = document.createElement("div");
        equipCard.className = "shop-item";
        const options = owned.map((s) => `<option value="${s.id}" ${state.player.equippedSword === s.id ? "selected" : ""}>${s.name}</option>`).join("");
        equipCard.innerHTML = `<b>Equip Weapon</b><small>Switch to an owned weapon.</small><select id="equipSel">${options}</select>`;
        shopGrid.appendChild(equipCard);
        setTimeout(() => {
          const sel = document.getElementById("equipSel");
          if (sel) sel.onchange = () => { state.player.equippedSword = sel.value; };
        }, 0);
      }

      function startGame() {
        state.screen = "playing";
        overlays.menu.classList.remove("show");
        setMap(0);
      }

      function update(dt, t) {
        const p = state.player;
        if (state.screen !== "playing") return;

        let mx = 0, my = 0;
        if (state.keys.has("a") || state.keys.has("ArrowLeft")) mx -= 1;
        if (state.keys.has("d") || state.keys.has("ArrowRight")) mx += 1;
        if (state.keys.has("w") || state.keys.has("ArrowUp")) my -= 1;
        if (state.keys.has("s") || state.keys.has("ArrowDown")) my += 1;

        if (mx || my) {
          const len = Math.hypot(mx, my);
          mx /= len; my /= len;
          p.facing = Math.atan2(my, mx);
          p.x += mx * p.speed * dt;
          p.y += my * p.speed * dt;
        }

        p.x = Math.max(16, Math.min(canvas.width - 16, p.x));
        p.y = Math.max(16, Math.min(canvas.height - 16, p.y));
        p.attackT = Math.max(0, p.attackT - dt * 4.4);
        p.hitFlash = Math.max(0, p.hitFlash - dt);

        const sword = currentSword();
        if (state.keys.has(" ") && t - p.lastAttackAt > sword.cooldown) {
          p.lastAttackAt = t;
          p.attackT = 1;
          state.cameraShake = 6;
          const aa = p.facing;
          state.monsters.forEach((m) => {
            const dx = m.x - p.x, dy = m.y - p.y;
            const dist = Math.hypot(dx, dy);
            const ang = Math.atan2(dy, dx);
            const off = Math.atan2(Math.sin(ang - aa), Math.cos(ang - aa));
            if (dist < sword.range + m.radius && Math.abs(off) < 0.9) {
              const crit = Math.random() < 0.14;
              const dmg = sword.damage + (crit ? sword.damage * 0.5 : 0);
              m.hp -= dmg;
              m.hitFlash = 0.12;
              if (p.fireSlash) m.burn = Math.max(m.burn, 1.6);
              m.x += Math.cos(ang) * 12;
              m.y += Math.sin(ang) * 12;
              spawnBlood(m.x, m.y, crit ? 22 : 14, p.fireSlash ? "#ff5e1f" : "#a6111c");
            }
          });
        }

        state.monsters.forEach((m) => {
          const dx = p.x - m.x, dy = p.y - m.y;
          const d = Math.hypot(dx, dy) || 1;
          m.x += (dx / d) * m.speed * dt;
          m.y += (dy / d) * m.speed * dt;
          m.hitFlash = Math.max(0, m.hitFlash - dt);
          m.attackCd -= dt;

          if (m.burn > 0) {
            m.burn -= dt;
            m.hp -= 9 * dt;
            if (Math.random() < 0.3) spawnBlood(m.x, m.y, 1, "#ff7a2e");
          }

          if (circleHit(p, m) && m.attackCd <= 0) {
            m.attackCd = m.elite ? 0.78 : 0.95;
            p.hp -= m.damage;
            p.hitFlash = 0.2;
            state.cameraShake = 8;
            if (p.hp <= 0) {
              p.hp = 0;
              state.screen = "gameover";
              status("You were defeated. Refresh to retry.", 7000);
            }
          }
        });

        const killed = state.monsters.filter((m) => m.hp <= 0);
        if (killed.length) {
          killed.forEach((m) => {
            gainXp(m.elite ? 22 : 11);
            p.gold += m.elite ? 18 : 8;
            spawnLoot(m.x, m.y);
            if (Math.random() < (m.elite ? 0.65 : 0.26)) spawnLoot(m.x + 8, m.y + 6);
          });
          state.monsters = state.monsters.filter((m) => m.hp > 0);
        }

        state.drops.forEach((d) => {
          d.life -= dt;
          if (circleHit(p, d, 3)) {
            if (d.type === "coin") p.gold += 5;
            if (d.type === "shard") gainXp(12);
            if (d.type === "potion") p.potions += 1;
            d.life = 0;
          }
        });
        state.drops = state.drops.filter((d) => d.life > 0);

        if (state.keys.has("q") && p.potions > 0 && p.hp < p.maxHp) {
          p.potions -= 1;
          p.hp = Math.min(p.maxHp, p.hp + 35);
          status("Potion used +35 HP", 1200);
          state.keys.delete("q");
        }

        state.particles.forEach((k) => {
          k.life -= dt;
          k.x += k.vx * dt;
          k.y += k.vy * dt;
          k.vx *= 0.92;
          k.vy = k.vy * 0.92 + 420 * dt;
        });
        state.particles = state.particles.filter((k) => k.life > 0);

        if (state.monsters.length === 0) {
          if (state.mapIndex < maps.length - 1) setMap(state.mapIndex + 1);
          else {
            state.screen = "victory";
            status("All maps cleared. You are the Blade Champion!", 999999);
          }
        }

        state.cameraShake = Math.max(0, state.cameraShake - dt * 20);
      }

      function drawWorld() {
        const map = maps[state.mapIndex] || maps[0];
        const tile = 32;
        for (let y = 0; y < canvas.height; y += tile) {
          for (let x = 0; x < canvas.width; x += tile) {
            ctx.fillStyle = map.floor[((x / tile + y / tile) & 1)];
            ctx.fillRect(x, y, tile, tile);
          }
        }
        ctx.fillStyle = map.wall;
        ctx.fillRect(0, 0, canvas.width, 12);
        ctx.fillRect(0, canvas.height - 12, canvas.width, 12);
        ctx.fillRect(0, 0, 12, canvas.height);
        ctx.fillRect(canvas.width - 12, 0, 12, canvas.height);
        ctx.fillStyle = map.tint;
        ctx.fillRect(0, 0, canvas.width, canvas.height);
      }

      function drawPlayer() {
        const p = state.player;
        const s = currentSword();
        ctx.fillStyle = p.hitFlash > 0 ? "#ffdbdb" : "#80b9ff";
        ctx.fillRect((p.x - 10) | 0, (p.y - 12) | 0, 20, 24);
        ctx.fillStyle = "#273f6f";
        ctx.fillRect((p.x - 7) | 0, (p.y - 17) | 0, 14, 6);

        let swingOffset = 0;
        if (p.attackT > 0) {
          const curve = 1 - (1 - p.attackT) ** 2;
          swingOffset = -1.2 + curve * 2.4;
        }
        const ang = p.facing + swingOffset;
        const sx = p.x + Math.cos(ang) * 18;
        const sy = p.y + Math.sin(ang) * 18;

        ctx.save();
        ctx.translate(sx, sy);
        ctx.rotate(ang);
        ctx.fillStyle = s.color;
        ctx.fillRect(0, -3, 27, 6);
        ctx.fillStyle = "#2f2d3f";
        ctx.fillRect(-4, -4, 5, 8);
        if (p.attackT > 0) {
          ctx.strokeStyle = state.player.fireSlash ? "rgba(255,150,66,0.7)" : "rgba(255,255,255,0.5)";
          ctx.lineWidth = 3;
          ctx.beginPath();
          ctx.arc(0, 0, 42, -0.8, 0.8);
          ctx.stroke();
        }
        ctx.restore();
      }

      function drawMobs() {
        state.monsters.forEach((m) => {
          ctx.fillStyle = m.hitFlash > 0 ? "#ffd0d0" : m.elite ? "#bd77ff" : "#7cd567";
          ctx.fillRect((m.x - m.radius) | 0, (m.y - m.radius) | 0, m.radius * 2, m.radius * 2);
          ctx.fillStyle = "#220f2b";
          ctx.fillRect((m.x - 6) | 0, (m.y - 4) | 0, 4, 4);
          ctx.fillRect((m.x + 2) | 0, (m.y - 4) | 0, 4, 4);
          const w = 24, ratio = Math.max(0, m.hp / m.maxHp);
          ctx.fillStyle = "#171717";
          ctx.fillRect((m.x - w / 2) | 0, (m.y - m.radius - 8) | 0, w, 4);
          ctx.fillStyle = m.burn > 0 ? "#ff8d36" : m.elite ? "#d89aff" : "#a7ff91";
          ctx.fillRect((m.x - w / 2) | 0, (m.y - m.radius - 8) | 0, w * ratio, 4);
        });
      }

      function drawDropsAndParticles() {
        state.drops.forEach((d) => {
          ctx.fillStyle = d.type === "coin" ? "#ffd74d" : d.type === "shard" ? "#67f2ff" : "#ff6d8a";
          ctx.fillRect((d.x - 4) | 0, (d.y - 4) | 0, 8, 8);
        });
        state.particles.forEach((p) => {
          const alpha = Math.max(0, p.life / p.maxLife);
          ctx.fillStyle = p.color + Math.floor(alpha * 255).toString(16).padStart(2, "0");
          ctx.fillRect(p.x | 0, p.y | 0, p.size, p.size);
        });
      }

      function render() {
        ctx.clearRect(0, 0, canvas.width, canvas.height);
        const sx = state.cameraShake ? rand(-state.cameraShake, state.cameraShake) : 0;
        const sy = state.cameraShake ? rand(-state.cameraShake, state.cameraShake) : 0;
        ctx.save();
        ctx.translate(sx, sy);
        drawWorld();
        drawDropsAndParticles();
        drawMobs();
        drawPlayer();

        if (state.screen === "gameover") {
          ctx.fillStyle = "rgba(20,0,0,.6)";
          ctx.fillRect(0, 0, canvas.width, canvas.height);
          ctx.fillStyle = "#ff9f9f";
          ctx.font = "28px Courier New";
          ctx.fillText("You fell in battle", canvas.width / 2 - 130, canvas.height / 2);
        }
        if (state.screen === "victory") {
          ctx.fillStyle = "rgba(0,22,10,.56)";
          ctx.fillRect(0, 0, canvas.width, canvas.height);
          ctx.fillStyle = "#a6ffcc";
          ctx.font = "27px Courier New";
          ctx.fillText("Victory! All 10 maps cleared.", canvas.width / 2 - 190, canvas.height / 2);
        }
        ctx.restore();
      }

      function updateHud() {
        const p = state.player;
        hud.health.textContent = `${Math.ceil(p.hp)} / ${p.maxHp} (${p.potions} pot)`;
        hud.level.textContent = String(p.level);
        hud.xp.textContent = `${Math.floor(p.xp)} / ${p.xpToNext}`;
        hud.gold.textContent = String(p.gold);
        hud.crystals.textContent = String(p.crystals);
        const s = currentSword();
        hud.weapon.textContent = `${s.name} (${s.damage} dmg)`;
        hud.ability.textContent = p.fireSlash ? "Fire Slash" : "None";
        hud.map.textContent = maps[state.mapIndex]?.name || "-";
        hud.monsters.textContent = String(state.monsters.length);
        if (now() > state.statusUntil && state.screen === "playing") hud.status.textContent = "Clear monsters, level up, shop upgrades.";
      }

      function loop(ts) {
        if (!state.lastTime) state.lastTime = ts;
        const dt = Math.min((ts - state.lastTime) / 1000, 0.033);
        state.lastTime = ts;
        update(dt, ts);
        render();
        updateHud();
        requestAnimationFrame(loop);
      }

      window.addEventListener("keydown", (e) => {
        const k = e.key.length === 1 ? e.key.toLowerCase() : e.key;
        state.keys.add(k);
        if (["ArrowUp", "ArrowDown", "ArrowLeft", "ArrowRight", " "].includes(e.key)) e.preventDefault();

        if (e.key === "Enter" && state.screen === "menu") startGame();
        if ((k === "b") && state.screen === "playing" && state.player.crystals > 0) openShop("Manual shop open.");
        if (e.key === "Escape" && state.screen === "shop") closeShop();
      });
      window.addEventListener("keyup", (e) => {
        const k = e.key.length === 1 ? e.key.toLowerCase() : e.key;
        state.keys.delete(k);
      });

      document.getElementById("startBtn").onclick = startGame;
      document.getElementById("continueBtn").onclick = closeShop;

      renderShop();
      requestAnimationFrame(loop);
    })();
  </script>
</body>
</html>
