<!doctype html>
<html lang="en">
<head>
  <meta charset="utf-8" />
  <meta name="viewport" content="width=device-width,initial-scale=1" />
  <title>Free Drive Playground</title>
  <style>
    :root{
      --bg1:#060814; --bg2:#121a3b;
      --hud: rgba(0,0,0,.35);
      --txt:#eaf2ff;
      --accent:#7cf6ff;
      --coin:#ffe66d;
      --boost:#64ff9f;
      --rock:#a9b0c7;
      --tree:#2cf0a0;
      --danger:#ff4d6d;
    }
    html,body{height:100%; margin:0; background: radial-gradient(1200px 800px at 50% 35%, var(--bg2), var(--bg1)); color:var(--txt); font-family:system-ui,Segoe UI,Roboto,Arial,sans-serif;}
    canvas{display:block; width:100vw; height:100vh;}
    .hud{
      position:fixed; left:14px; top:14px; right:14px;
      display:flex; gap:12px; align-items:flex-start; justify-content:space-between;
      pointer-events:none;
    }
    .panel{
      pointer-events:none;
      background:var(--hud); backdrop-filter: blur(8px);
      border:1px solid rgba(255,255,255,.12);
      border-radius:12px; padding:10px 12px;
      box-shadow: 0 12px 30px rgba(0,0,0,.35);
      max-width: 520px;
    }
    .title{font-weight:800; letter-spacing:.3px;}
    .tiny{opacity:.9; font-size:12px; line-height:1.35;}
    .row{display:flex; gap:10px; flex-wrap:wrap; margin-top:6px; font-size:13px; opacity:.95;}
    .tag{padding:4px 8px; border-radius:999px; border:1px solid rgba(255,255,255,.14); background:rgba(255,255,255,.06);}
    .kbd{font-family: ui-monospace, SFMono-Regular, Menlo, Monaco, Consolas, "Liberation Mono", monospace; font-size:12px; padding:2px 6px; border-radius:7px; background:rgba(255,255,255,.10); border:1px solid rgba(255,255,255,.12);}
    .centerOverlay{
      position:fixed; inset:0; display:grid; place-items:center;
      pointer-events:none;
    }
    .card{
      pointer-events:auto;
      text-align:center;
      width:min(560px, calc(100vw - 40px));
      background:rgba(0,0,0,.55);
      border:1px solid rgba(255,255,255,.14);
      border-radius:16px; padding:18px 18px 14px;
      box-shadow: 0 18px 50px rgba(0,0,0,.5);
    }
    button{
      pointer-events:auto;
      margin-top:12px;
      cursor:pointer;
      border-radius:12px;
      border:1px solid rgba(255,255,255,.18);
      background:linear-gradient(180deg, rgba(124,246,255,.24), rgba(124,246,255,.08));
      color:var(--txt);
      padding:10px 14px;
      font-weight:700;
    }
    button:hover{filter:brightness(1.08);}
  </style>
</head>
<body>
  <canvas id="c"></canvas>

  <div class="hud">
    <div class="panel">
      <div class="title">Free Drive Playground</div>
      <div class="tiny">Huge open area. Grab coins, hit boost pads, and cruise.</div>
      <div class="row">
        <div class="tag"><span class="kbd">WASD</span> / <span class="kbd">↑↓←→</span> drive</div>
        <div class="tag"><span class="kbd">Space</span> handbrake</div>
        <div class="tag"><span class="kbd">Shift</span> turbo (uses energy)</div>
        <div class="tag"><span class="kbd">R</span> reset</div>
        <div class="tag"><span class="kbd">M</span> map</div>
      </div>
    </div>

    <div class="panel" id="stats"></div>
  </div>

  <div class="centerOverlay" id="overlay">
    <div class="card">
      <div style="font-size:22px; font-weight:900; letter-spacing:.3px;">🚗 Welcome to the Open Space</div>
      <div class="tiny" style="margin-top:8px;">
        Drive freely in a big world. Coins give points. Boost pads refill turbo energy.
        Avoid rocks (they slow you) and trees (they bounce you). Press <span class="kbd">M</span> anytime for a mini-map.
      </div>
      <button id="startBtn">Start Driving</button>
    </div>
  </div>

<script>
(() => {
  const canvas = document.getElementById('c');
  const ctx = canvas.getContext('2d', { alpha: false });
  const statsEl = document.getElementById('stats');
  const overlay = document.getElementById('overlay');
  const startBtn = document.getElementById('startBtn');

  // ----- Resize handling (HiDPI) -----
  function resize() {
    const dpr = Math.max(1, Math.min(2, window.devicePixelRatio || 1));
    canvas.width = Math.floor(innerWidth * dpr);
    canvas.height = Math.floor(innerHeight * dpr);
    canvas.style.width = innerWidth + 'px';
    canvas.style.height = innerHeight + 'px';
    ctx.setTransform(dpr, 0, 0, dpr, 0, 0);
  }
  addEventListener('resize', resize);
  resize();

  // ----- Input -----
  const keys = new Set();
  addEventListener('keydown', (e) => {
    if (["ArrowUp","ArrowDown","ArrowLeft","ArrowRight"," "].includes(e.key)) e.preventDefault();
    keys.add(e.key.toLowerCase());
  }, { passive:false });
  addEventListener('keyup', (e) => keys.delete(e.key.toLowerCase()));

  // ----- World -----
  const WORLD = { w: 7000, h: 7000 };  // lots of space
  const rand = (a,b)=>a+Math.random()*(b-a);
  const clamp = (v,a,b)=>Math.max(a, Math.min(b,v));
  const dist2 = (ax,ay,bx,by)=> {
    const dx=ax-bx, dy=ay-by; return dx*dx+dy*dy;
  };

  // Entities
  const coins = [];
  const boosts = [];
  const rocks = [];
  const trees = [];

  const COIN_COUNT = 220;
  const BOOST_COUNT = 28;
  const ROCK_COUNT = 70;
  const TREE_COUNT = 90;

  function scatter() {
    coins.length = boosts.length = rocks.length = trees.length = 0;

    // Keep center a bit more open for comfort
    const safeR = 260;

    function pickPos() {
      let x,y;
      do {
        x = rand(-WORLD.w/2, WORLD.w/2);
        y = rand(-WORLD.h/2, WORLD.h/2);
      } while (x*x + y*y < safeR*safeR);
      return {x,y};
    }

    for (let i=0;i<COIN_COUNT;i++){
      const p = pickPos();
      coins.push({x:p.x, y:p.y, r:10, taken:false, bob:rand(0,Math.PI*2)});
    }
    for (let i=0;i<BOOST_COUNT;i++){
      const p = pickPos();
      boosts.push({x:p.x, y:p.y, r:18, t:rand(0,Math.PI*2)});
    }
    for (let i=0;i<ROCK_COUNT;i++){
      const p = pickPos();
      rocks.push({x:p.x, y:p.y, r:rand(22,40)});
    }
    for (let i=0;i<TREE_COUNT;i++){
      const p = pickPos();
      trees.push({x:p.x, y:p.y, r:rand(18,34)});
    }
  }

  // ----- Player car physics -----
  const car = {
    x: 0, y: 0,
    vx: 0, vy: 0,
    angle: -Math.PI/2,
    angVel: 0,
    turbo: 100, // energy
    score: 0,
    coins: 0,
    best: 0
  };

  function resetCar() {
    car.x = 0; car.y = 0;
    car.vx = 0; car.vy = 0;
    car.angle = -Math.PI/2;
    car.angVel = 0;
    car.turbo = 100;
  }

  // ----- Camera -----
  const cam = { x: 0, y: 0 };

  // ----- Game loop -----
  let last = performance.now();
  let running = false;
  let showMap = false;

  startBtn.addEventListener('click', () => {
    overlay.style.display = 'none';
    running = true;
  });

  // Toggle map
  addEventListener('keydown', (e)=>{
    const k = e.key.toLowerCase();
    if (k === 'm') showMap = !showMap;
    if (k === 'r') resetCar();
  });

  scatter();

  function update(dt) {
    // Controls
    const up = keys.has('w') || keys.has('arrowup');
    const down = keys.has('s') || keys.has('arrowdown');
    const left = keys.has('a') || keys.has('arrowleft');
    const right = keys.has('d') || keys.has('arrowright');
    const handbrake = keys.has(' ');
    const turboKey = keys.has('shift');

    // Physics tuning
    const baseAccel = 520;
    const reverseAccel = 360;
    const steerPower = 3.2;
    const grip = handbrake ? 1.2 : 3.4;   // less grip -> more drift
    const drag = 0.985;
    const maxSpeed = handbrake ? 780 : 920;

    // Turbo
    let turboMul = 1;
    if (turboKey && car.turbo > 0.5) {
      turboMul = 1.65;
      car.turbo -= 35 * dt;
    } else {
      car.turbo += 12 * dt; // slow regen
    }
    car.turbo = clamp(car.turbo, 0, 100);

    // Steering depends a bit on speed
    const speed = Math.hypot(car.vx, car.vy);
    const steer = (left ? -1 : 0) + (right ? 1 : 0);
    const steerFactor = steerPower * (0.35 + clamp(speed/900, 0, 1));
    car.angVel += steer * steerFactor * dt;

    // Throttle
    const forward = { x: Math.cos(car.angle), y: Math.sin(car.angle) };
    if (up) {
      car.vx += forward.x * baseAccel * turboMul * dt;
      car.vy += forward.y * baseAccel * turboMul * dt;
    }
    if (down) {
      car.vx -= forward.x * reverseAccel * dt;
      car.vy -= forward.y * reverseAccel * dt;
    }

    // Integrate angle
    car.angle += car.angVel * dt;
    car.angVel *= 0.90;

    // Lateral grip: pull velocity toward forward direction
    const velDotF = car.vx*forward.x + car.vy*forward.y;
    const targetVx = forward.x * velDotF;
    const targetVy = forward.y * velDotF;

    car.vx += (targetVx - car.vx) * clamp(grip*dt, 0, 1);
    car.vy += (targetVy - car.vy) * clamp(grip*dt, 0, 1);

    // Drag & speed cap
    car.vx *= drag;
    car.vy *= drag;
    const newSpeed = Math.hypot(car.vx, car.vy);
    if (newSpeed > maxSpeed) {
      const s = maxSpeed / newSpeed;
      car.vx *= s; car.vy *= s;
    }

    // Move
    car.x += car.vx * dt;
    car.y += car.vy * dt;

    // World bounds: soft pushback
    const hx = WORLD.w/2, hy = WORLD.h/2;
    if (car.x < -hx) { car.x = -hx; car.vx *= -0.35; }
    if (car.x >  hx) { car.x =  hx; car.vx *= -0.35; }
    if (car.y < -hy) { car.y = -hy; car.vy *= -0.35; }
    if (car.y >  hy) { car.y =  hy; car.vy *= -0.35; }

    // Interactions
    // Coins
    for (const c of coins) {
      if (c.taken) continue;
      const rr = (c.r + 18);
      if (dist2(car.x, car.y, c.x, c.y) < rr*rr) {
        c.taken = true;
        car.coins++;
        car.score += 50;
      }
    }

    // Boost pads: refill turbo + score tick
    for (const b of boosts) {
      const rr = (b.r + 18);
      if (dist2(car.x, car.y, b.x, b.y) < rr*rr) {
        car.turbo = Math.min(100, car.turbo + 80 * dt);
        car.score += 10 * dt;
      }
    }

    // Rocks: slow-down bump
    for (const r of rocks) {
      const rr = (r.r + 18);
      const d2 = dist2(car.x, car.y, r.x, r.y);
      if (d2 < rr*rr) {
        // Push out
        const d = Math.sqrt(Math.max(d2, 0.0001));
        const nx = (car.x - r.x)/d, ny=(car.y - r.y)/d;
        const push = (rr - d) * 0.9;
        car.x += nx * push;
        car.y += ny * push;
        // Slow
        car.vx *= 0.65;
        car.vy *= 0.65;
        car.score = Math.max(0, car.score - 20 * dt);
      }
    }

    // Trees: bouncy
    for (const t of trees) {
      const rr = (t.r + 18);
      const d2 = dist2(car.x, car.y, t.x, t.y);
      if (d2 < rr*rr) {
        const d = Math.sqrt(Math.max(d2, 0.0001));
        const nx = (car.x - t.x)/d, ny=(car.y - t.y)/d;
        const push = (rr - d) * 1.0;
        car.x += nx * push;
        car.y += ny * push;
        // Bounce
        const dot = car.vx*nx + car.vy*ny;
        car.vx -= 1.6 * dot * nx;
        car.vy -= 1.6 * dot * ny;
        car.score = Math.max(0, car.score - 35 * dt);
      }
    }

    // Score from speed (fun cruising reward)
    car.score += (Math.min(900, Math.hypot(car.vx, car.vy)) / 220) * dt;
    car.best = Math.max(car.best, Math.floor(car.score));

    // Camera follow with smoothing
    cam.x += (car.x - cam.x) * (1 - Math.pow(0.0006, dt)); // dt-stable smoothing
    cam.y += (car.y - cam.y) * (1 - Math.pow(0.0006, dt));
  }

  // ----- Drawing helpers -----
  function drawBackground() {
    // Spacey grid + subtle stars
    const w = innerWidth, h = innerHeight;

    // Starfield
    ctx.fillStyle = "#070914";
    ctx.fillRect(0,0,w,h);

    // radial glow
    const g = ctx.createRadialGradient(w*0.5, h*0.35, 40, w*0.5, h*0.5, Math.max(w,h));
    g.addColorStop(0, "rgba(18,26,59,0.95)");
    g.addColorStop(1, "rgba(6,8,20,1)");
    ctx.fillStyle = g;
    ctx.fillRect(0,0,w,h);

    // sprinkle stars
    ctx.globalAlpha = 0.35;
    ctx.fillStyle = "#ffffff";
    for (let i=0;i<60;i++){
      const x = (i*997) % w;
      const y = (i*577) % h;
      ctx.fillRect(x, y, 2, 2);
    }
    ctx.globalAlpha = 1;
  }

  function worldToScreen(wx, wy) {
    const sx = (wx - cam.x) + innerWidth/2;
    const sy = (wy - cam.y) + innerHeight/2;
    return {x:sx, y:sy};
  }

  function drawWorld() {
    const w = innerWidth, h = innerHeight;

    // draw ground grid in world space
    const grid = 140;
    const left = cam.x - w/2, right = cam.x + w/2;
    const top = cam.y - h/2, bottom = cam.y + h/2;

    // faint grid lines
    ctx.strokeStyle = "rgba(255,255,255,0.06)";
    ctx.lineWidth = 1;

    const x0 = Math.floor(left / grid) * grid;
    const x1 = Math.ceil(right / grid) * grid;
    const y0 = Math.floor(top / grid) * grid;
    const y1 = Math.ceil(bottom / grid) * grid;

    ctx.beginPath();
    for (let x = x0; x <= x1; x += grid) {
      const a = worldToScreen(x, top);
      const b = worldToScreen(x, bottom);
      ctx.moveTo(a.x, a.y); ctx.lineTo(b.x, b.y);
    }
    for (let y = y0; y <= y1; y += grid) {
      const a = worldToScreen(left, y);
      const b = worldToScreen(right, y);
      ctx.moveTo(a.x, a.y); ctx.lineTo(b.x, b.y);
    }
    ctx.stroke();

    // world boundary rectangle (so you know the edge exists)
    ctx.strokeStyle = "rgba(124,246,255,0.18)";
    ctx.lineWidth = 2;
    const hx = WORLD.w/2, hy = WORLD.h/2;
    const p1 = worldToScreen(-hx, -hy);
    const p2 = worldToScreen(hx, hy);
    ctx.strokeRect(p1.x, p1.y, (p2.x - p1.x), (p2.y - p1.y));

    // draw boosts
    for (const b of boosts) {
      const s = worldToScreen(b.x, b.y);
      if (s.x < -60 || s.y < -60 || s.x > w+60 || s.y > h+60) continue;

      const pulse = 0.75 + 0.25*Math.sin(performance.now()/250 + b.t);
      ctx.beginPath();
      ctx.fillStyle = `rgba(100,255,159,${0.22 + 0.18*pulse})`;
      ctx.strokeStyle = "rgba(100,255,159,0.85)";
      ctx.lineWidth = 2;
      ctx.arc(s.x, s.y, b.r + 10*pulse, 0, Math.PI*2);
      ctx.fill(); ctx.stroke();

      ctx.beginPath();
      ctx.fillStyle = "rgba(100,255,159,0.9)";
      ctx.arc(s.x, s.y, 5, 0, Math.PI*2);
      ctx.fill();
    }

    // draw coins
    for (const c of coins) {
      if (c.taken) continue;
      const s = worldToScreen(c.x, c.y);
      if (s.x < -50 || s.y < -50 || s.x > w+50 || s.y > h+50) continue;

      const bob = 3*Math.sin(performance.now()/300 + c.bob);
      ctx.beginPath();
      ctx.strokeStyle = "rgba(255,230,109,0.95)";
      ctx.fillStyle = "rgba(255,230,109,0.25)";
      ctx.lineWidth = 2;
      ctx.ellipse(s.x, s.y + bob, c.r, c.r*0.72, 0, 0, Math.PI*2);
      ctx.fill(); ctx.stroke();
      ctx.globalAlpha = 0.8;
      ctx.fillStyle = "rgba(255,230,109,0.55)";
      ctx.fillRect(s.x-1, s.y-6 + bob, 2, 12);
      ctx.globalAlpha = 1;
    }

    // draw rocks
    for (const r of rocks) {
      const s = worldToScreen(r.x, r.y);
      if (s.x < -80 || s.y < -80 || s.x > w+80 || s.y > h+80) continue;
      ctx.beginPath();
      ctx.fillStyle = "rgba(169,176,199,0.35)";
      ctx.strokeStyle = "rgba(169,176,199,0.85)";
      ctx.lineWidth = 2;
      ctx.arc(s.x, s.y, r.r, 0, Math.PI*2);
      ctx.fill(); ctx.stroke();
    }

    // draw trees
    for (const t of trees) {
      const s = worldToScreen(t.x, t.y);
      if (s.x < -80 || s.y < -80 || s.x > w+80 || s.y > h+80) continue;
      ctx.beginPath();
      ctx.fillStyle = "rgba(44,240,160,0.18)";
      ctx.strokeStyle = "rgba(44,240,160,0.85)";
      ctx.lineWidth = 2;
      ctx.arc(s.x, s.y, t.r, 0, Math.PI*2);
      ctx.fill(); ctx.stroke();

      // trunk dot
      ctx.beginPath();
      ctx.fillStyle = "rgba(255,255,255,0.28)";
      ctx.arc(s.x, s.y, 4, 0, Math.PI*2);
      ctx.fill();
    }

    // center marker
    const center = worldToScreen(0,0);
    ctx.globalAlpha = 0.8;
    ctx.strokeStyle = "rgba(124,246,255,0.85)";
    ctx.lineWidth = 2;
    ctx.beginPath();
    ctx.moveTo(center.x-12, center.y); ctx.lineTo(center.x+12, center.y);
    ctx.moveTo(center.x, center.y-12); ctx.lineTo(center.x, center.y+12);
    ctx.stroke();
    ctx.globalAlpha = 1;
  }

  function drawCar() {
    const s = worldToScreen(car.x, car.y);
    ctx.save();
    ctx.translate(s.x, s.y);
    ctx.rotate(car.angle);

    // car shadow
    ctx.globalAlpha = 0.35;
    ctx.fillStyle = "#000";
    ctx.beginPath();
    ctx.roundRect(-12, -18, 24, 38, 8);
    ctx.fill();
    ctx.globalAlpha = 1;

    // body
    const speed = Math.hypot(car.vx, car.vy);
    const glow = clamp(speed / 900, 0, 1);
    ctx.fillStyle = `rgba(124,246,255,${0.25 + 0.15*glow})`;
    ctx.strokeStyle = "rgba(124,246,255,0.95)";
    ctx.lineWidth = 2.2;
    ctx.beginPath();
    ctx.roundRect(-13, -20, 26, 42, 10);
    ctx.fill();
    ctx.stroke();

    // cabin
    ctx.fillStyle = "rgba(234,242,255,0.28)";
    ctx.beginPath();
    ctx.roundRect(-9, -12, 18, 18, 7);
    ctx.fill();

    // headlights
    ctx.fillStyle = "rgba(255,255,255,0.85)";
    ctx.fillRect(-10, -22, 6, 4);
    ctx.fillRect(4, -22, 6, 4);

    // trail flames if turbo
    if ((keys.has('shift') && car.turbo > 0.5) || speed > 820) {
      ctx.globalAlpha = 0.9;
      ctx.fillStyle = "rgba(255,230,109,0.75)";
      ctx.beginPath();
      ctx.moveTo(-7, 22);
      ctx.quadraticCurveTo(0, 36, 7, 22);
      ctx.quadraticCurveTo(0, 26, -7, 22);
      ctx.fill();
      ctx.globalAlpha = 1;
    }

    ctx.restore();
  }

  // Mini-map (top-right overlay)
  function drawMap() {
    const pad = 14;
    const w = 200, h = 200;
    const x = innerWidth - w - pad;
    const y = innerHeight - h - pad;

    ctx.save();
    ctx.globalAlpha = 0.95;
    ctx.fillStyle = "rgba(0,0,0,0.45)";
    ctx.strokeStyle = "rgba(255,255,255,0.18)";
    ctx.lineWidth = 1;
    ctx.beginPath();
    ctx.roundRect(x, y, w, h, 14);
    ctx.fill(); ctx.stroke();

    // map area
    const mx = x+12, my=y+12, mw=w-24, mh=h-24;

    // world -> map
    const hx = WORLD.w/2, hy = WORLD.h/2;
    const wxToMapX = (wx)=> mx + ( (wx + hx) / (2*hx) ) * mw;
    const wyToMapY = (wy)=> my + ( (wy + hy) / (2*hy) ) * mh;

    // boundary
    ctx.strokeStyle = "rgba(124,246,255,0.25)";
    ctx.strokeRect(mx, my, mw, mh);

    // dots
    // coins
    ctx.fillStyle = "rgba(255,230,109,0.75)";
    for (const c of coins) {
      if (c.taken) continue;
      const px = wxToMapX(c.x), py = wyToMapY(c.y);
      ctx.fillRect(px, py, 2, 2);
    }
    // boosts
    ctx.fillStyle = "rgba(100,255,159,0.85)";
    for (const b of boosts) {
      const px = wxToMapX(b.x), py = wyToMapY(b.y);
      ctx.fillRect(px, py, 3, 3);
    }
    // obstacles
    ctx.fillStyle = "rgba(169,176,199,0.55)";
    for (const r of rocks) {
      const px = wxToMapX(r.x), py = wyToMapY(r.y);
      ctx.fillRect(px, py, 2, 2);
    }
    ctx.fillStyle = "rgba(44,240,160,0.55)";
    for (const t of trees) {
      const px = wxToMapX(t.x), py = wyToMapY(t.y);
      ctx.fillRect(px, py, 2, 2);
    }

    // player
    const px = wxToMapX(car.x), py = wyToMapY(car.y);
    ctx.save();
    ctx.translate(px, py);
    ctx.rotate(car.angle);
    ctx.fillStyle = "rgba(124,246,255,1)";
    ctx.beginPath();
    ctx.moveTo(0, -5);
    ctx.lineTo(3.5, 5);
    ctx.lineTo(-3.5, 5);
    ctx.closePath();
    ctx.fill();
    ctx.restore();

    // label
    ctx.fillStyle = "rgba(234,242,255,0.85)";
    ctx.font = "12px system-ui,Segoe UI,Roboto,Arial";
    ctx.fillText("Map (M)", x+14, y+18);

    ctx.restore();
  }

  // roundRect polyfill for older browsers
  if (!CanvasRenderingContext2D.prototype.roundRect) {
    CanvasRenderingContext2D.prototype.roundRect = function(x,y,w,h,r){
      r = Math.min(r, w/2, h/2);
      this.beginPath();
      this.moveTo(x+r, y);
      this.arcTo(x+w, y, x+w, y+h, r);
      this.arcTo(x+w, y+h, x, y+h, r);
      this.arcTo(x, y+h, x, y, r);
      this.arcTo(x, y, x+w, y, r);
      this.closePath();
      return this;
    }
  }

  function drawHUD() {
    const speed = Math.hypot(car.vx, car.vy);
    const remainingCoins = coins.reduce((a,c)=>a + (c.taken ? 0 : 1), 0);

    statsEl.innerHTML = `
      <div class="title" style="display:flex; gap:10px; align-items:center; justify-content:space-between;">
        <span>Stats</span>
        <span style="font-size:12px; opacity:.9;">Press <span class="kbd">M</span> for map</span>
      </div>
      <div class="row" style="margin-top:8px;">
        <div class="tag">Speed: <b>${Math.round(speed)}</b></div>
        <div class="tag">Score: <b>${Math.floor(car.score)}</b></div>
        <div class="tag">Best: <b>${car.best}</b></div>
        <div class="tag">Coins: <b>${car.coins}</b> <span style="opacity:.75;">(left ${remainingCoins})</span></div>
      </div>
      <div style="margin-top:10px;">
        <div style="font-size:12px; opacity:.9; margin-bottom:6px;">Turbo energy</div>
        <div style="height:10px; border-radius:999px; background:rgba(255,255,255,.10); overflow:hidden; border:1px solid rgba(255,255,255,.12);">
          <div style="height:100%; width:${car.turbo}%; background:linear-gradient(90deg, rgba(100,255,159,.9), rgba(124,246,255,.9));"></div>
        </div>
        <div class="tiny" style="margin-top:6px;">Tip: Boost pads refill turbo fast.</div>
      </div>
    `;
  }

  function loop(now) {
    const dt = Math.min(0.033, (now - last) / 1000);
    last = now;

    drawBackground();

    if (running) update(dt);

    drawWorld();
    drawCar();
    if (showMap) drawMap();
    drawHUD();

    requestAnimationFrame(loop);
  }
  requestAnimationFrame(loop);

  // Click anywhere to start too
  addEventListener('pointerdown', () => {
    if (!running) {
      overlay.style.display = 'none';
      running = true;
    }
  });

})();
</script>
</body>
</html>