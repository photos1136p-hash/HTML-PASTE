<!DOCTYPE html>

<html lang="en">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>APEX DRIVE</title>
<style>
  *{margin:0;padding:0;box-sizing:border-box}
  body{background:#000;overflow:hidden;font-family:'Courier New',monospace}
  canvas{display:block;position:absolute;top:0;left:0}
  #hud{position:fixed;top:0;left:0;width:100%;height:100%;pointer-events:none;z-index:5}
  #speedo{position:absolute;bottom:28px;right:36px;background:rgba(0,0,0,.75);border:1px solid rgba(255,180,0,.45);border-radius:14px;padding:18px 26px;text-align:center}
  #spd{font-size:54px;font-weight:900;color:#FFB400;line-height:1;letter-spacing:-2px;text-shadow:0 0 18px rgba(255,180,0,.7)}
  #sunit{font-size:12px;color:rgba(255,180,0,.55);letter-spacing:4px;margin-top:3px}
  #gearbox{position:absolute;bottom:28px;right:220px;background:rgba(0,0,0,.75);border:1px solid rgba(255,180,0,.3);border-radius:12px;padding:14px 20px;text-align:center}
  #gval{font-size:42px;font-weight:900;color:#fff;line-height:1}
  #glabel{font-size:10px;color:rgba(255,255,255,.4);letter-spacing:3px;margin-top:3px}
  #rpmwrap{position:absolute;bottom:148px;right:36px;width:250px}
  #rpmlbl{font-size:10px;color:rgba(255,180,0,.5);letter-spacing:4px;text-align:right;margin-bottom:5px}
  #rpmbg{width:100%;height:7px;background:rgba(255,255,255,.08);border-radius:4px;overflow:hidden}
  #rpmfill{height:100%;width:0%;background:linear-gradient(90deg,#FFB400,#FF4400);border-radius:4px;transition:width .05s}
  #hints{position:absolute;bottom:28px;left:36px;background:rgba(0,0,0,.7);border:1px solid rgba(255,255,255,.1);border-radius:12px;padding:14px 18px}
  #hints div{font-size:12px;color:rgba(255,255,255,.5);line-height:2}
  #hints b{color:#FFB400}
  #mmwrap{position:absolute;top:18px;right:26px;width:130px;height:130px;background:rgba(0,0,0,.7);border:1px solid rgba(255,180,0,.25);border-radius:10px;overflow:hidden}
  #banner{position:absolute;top:22px;left:50%;transform:translateX(-50%);font-size:12px;letter-spacing:8px;color:rgba(255,180,0,.7);font-weight:700;white-space:nowrap}
  #loading{position:fixed;top:0;left:0;width:100%;height:100%;background:#000;display:flex;flex-direction:column;align-items:center;justify-content:center;z-index:99;color:#FFB400;font-family:'Courier New',monospace}
  #loading h1{font-size:36px;letter-spacing:10px;margin-bottom:20px}
  #loading p{font-size:13px;letter-spacing:4px;color:rgba(255,180,0,.6);animation:blink 1s infinite}
  @keyframes blink{0%,100%{opacity:1}50%{opacity:0.3}}
</style>
</head>
<body>
<div id="loading"><h1>APEX DRIVE</h1><p>LOADING WORLD...</p></div>
<canvas id="gc"></canvas>
<div id="hud">
  <div id="banner">APEX DRIVE — FREE ROAM</div>
  <div id="mmwrap"><canvas id="mm" width="130" height="130"></canvas></div>
  <div id="rpmwrap"><div id="rpmlbl">RPM</div><div id="rpmbg"><div id="rpmfill"></div></div></div>
  <div id="gearbox"><div id="gval">N</div><div id="glabel">GEAR</div></div>
  <div id="speedo"><div id="spd">0</div><div id="sunit">KM/H</div></div>
  <div id="hints">
    <div><b>W / ↑</b>&nbsp; Accelerate</div>
    <div><b>S / ↓</b>&nbsp; Brake / Reverse</div>
    <div><b>A D / ← →</b>&nbsp; Steer</div>
    <div><b>SPACE</b>&nbsp; Handbrake</div>
    <div><b>R</b>&nbsp; Reset Car</div>
  </div>
</div>

<script>
// ── Canvas setup ──────────────────────────────────────────────────────────
var gc = document.getElementById('gc');
var ctx = gc.getContext('2d');
var mm = document.getElementById('mm');
var mc = mm.getContext('2d');

function resize() { gc.width = window.innerWidth; gc.height = window.innerHeight; }
resize();
window.addEventListener('resize', resize);

// ── World dimensions ──────────────────────────────────────────────────────
var WS = 2400, HS = 2400;

// ── Helpers ───────────────────────────────────────────────────────────────
function lerp(a, b, t) { return a + (b - a) * t; }
function rnd(a, b) { return a + Math.random() * (b - a); }
function ri(a, b) { return Math.floor(rnd(a, b)); }
function preRand(x, y) { return (((Math.sin(x * 127.1 + y * 311.7) * 43758.5453) % 1) + 1) % 1; }

function shadeColor(hex, amt) {
  var n = parseInt(hex.replace('#',''), 16);
  var r = Math.min(255, Math.max(0, ((n >> 16) & 0xff) + amt));
  var g = Math.min(255, Math.max(0, ((n >> 8) & 0xff) + amt));
  var b = Math.min(255, Math.max(0, (n & 0xff) + amt));
  return 'rgb('+r+','+g+','+b+')';
}

function rrect(ctx2, x, y, w, h, r) {
  ctx2.beginPath();
  ctx2.moveTo(x + r, y);
  ctx2.lineTo(x + w - r, y); ctx2.arcTo(x+w, y, x+w, y+r, r);
  ctx2.lineTo(x + w, y + h - r); ctx2.arcTo(x+w, y+h, x+w-r, y+h, r);
  ctx2.lineTo(x + r, y + h); ctx2.arcTo(x, y+h, x, y+h-r, r);
  ctx2.lineTo(x, y + r); ctx2.arcTo(x, y, x+r, y, r);
  ctx2.closePath();
}

// ── World data ────────────────────────────────────────────────────────────
var roads = [
  [200,200,2200,200,58],[2200,200,2200,2200,58],[2200,2200,200,2200,58],[200,2200,200,200,58],
  [600,600,1800,600,52],[1800,600,1800,1800,52],[1800,1800,600,1800,52],[600,1800,600,600,52],
  [200,1200,2200,1200,52],[1200,200,1200,2200,52],
  [200,200,600,600,38],[2200,200,1800,600,38],[2200,2200,1800,1800,38],[200,2200,600,1800,38],
  [600,600,200,400,32],[1800,600,2200,400,32],[600,1800,200,2000,32],[1800,1800,2200,2000,32],
  [1200,200,1350,50,30],[1200,2200,1350,2380,30],[200,1200,50,1350,30],[2200,1200,2380,1350,30],
];

var buildingColors = ['#8899aa','#99aabb','#7a8fa0','#b0c4d4','#6b8096','#d4c5a9','#c9b99a','#a0b0c0'];
var bPositions = [
  {x:690,y:680},{x:830,y:680},{x:970,y:680},{x:1110,y:680},
  {x:690,y:820},{x:830,y:820},{x:970,y:820},
  {x:1310,y:680},{x:1450,y:680},{x:1590,y:680},{x:1690,y:680},
  {x:1310,y:820},{x:1450,y:820},{x:1590,y:820},
  {x:690,y:1310},{x:830,y:1310},{x:970,y:1310},
  {x:690,y:1450},{x:830,y:1450},{x:970,y:1450},
  {x:1310,y:1310},{x:1450,y:1310},{x:1590,y:1310},
  {x:1310,y:1450},{x:1450,y:1450},{x:1590,y:1450},
  {x:285,y:280},{x:390,y:280},{x:285,y:390},
  {x:1990,y:280},{x:2090,y:280},{x:2090,y:390},
  {x:285,y:1990},{x:390,y:2090},{x:285,y:2090},
  {x:1990,y:1990},{x:2090,y:1990},{x:2090,y:2090},
  {x:285,y:800},{x:285,y:960},{x:285,y:1120},
  {x:2090,y:800},{x:2090,y:960},{x:2090,y:1120},
  {x:800,y:285},{x:960,y:285},{x:1120,y:285},
  {x:800,y:2090},{x:960,y:2090},{x:1120,y:2090},
];

var buildings = [];
bPositions.forEach(function(p) {
  buildings.push({
    x: p.x + rnd(-12,12), y: p.y + rnd(-12,12),
    w: rnd(48,105), h: rnd(48,105),
    color: buildingColors[ri(0, buildingColors.length)],
    sh: rnd(8,28)
  });
});

// Pre-generate window states
buildings.forEach(function(b) {
  b.windows = [];
  var cols = Math.floor(b.w / 18);
  var rows = Math.floor(b.h / 20);
  for (var r = 0; r < rows; r++) {
    for (var c = 0; c < cols; c++) {
      b.windows.push(preRand(b.x + c*77, b.y + r*77) < 0.65 ?
        (preRand(b.x+c*13,b.y+r*17) < 0.35 ? '#ffffaa' : '#253040') : null);
    }
  }
  b.winCols = cols;
  b.winRows = rows;
});

var trees = [];
for (var i = 0; i < 300; i++) {
  trees.push({ x: rnd(60,WS-60), y: rnd(60,HS-60), r: rnd(10,20) });
}
for (var j = 0; j < 60; j++) {
  var a = Math.random() * Math.PI * 2, d = rnd(30,200);
  trees.push({ x: 1200 + Math.cos(a)*d, y: 1200 + Math.sin(a)*d, r: rnd(10,18) });
}

var intersections = [
  [200,200],[2200,200],[2200,2200],[200,2200],
  [600,600],[1800,600],[1800,1800],[600,1800],
  [1200,1200],[200,1200],[2200,1200],[1200,200],[1200,2200],
  [600,1200],[1800,1200],[1200,600],[1200,1800],
];

// ── Car state ─────────────────────────────────────────────────────────────
var car = {
  x: 1200, y: 1430,
  vx: 0, vy: 0,
  angle: 0,
  steer: 0,
};

var CFG = {
  maxSpeed: 8.0,
  acc: 0.18,
  brk: 0.28,
  rev: 0.08,
  steerMax: 0.048,
  steerSpeed: 0.15,
  steerReturn: 0.12,
  grip: 0.82,
  gripHB: 0.96,
  drag: 0.976,
  revDrag: 0.94,
};

// ── Input ─────────────────────────────────────────────────────────────────
var keys = {};
window.addEventListener('keydown', function(e) {
  keys[e.code] = true;
  if(['Space','ArrowUp','ArrowDown','ArrowLeft','ArrowRight'].indexOf(e.code) >= 0)
    e.preventDefault();
});
window.addEventListener('keyup', function(e) { keys[e.code] = false; });

// ── Camera ────────────────────────────────────────────────────────────────
var cam = { x: 1200, y: 1430, sx: 1200, sy: 1430, scale: 1.4 };

// ── Physics ───────────────────────────────────────────────────────────────
function updatePhysics(dt) {
  dt = Math.min(dt, 0.05);

  var thr = (keys['KeyW'] || keys['ArrowUp']) ? 1 : 0;
  var brk = (keys['KeyS'] || keys['ArrowDown']) ? 1 : 0;
  var sl  = (keys['KeyA'] || keys['ArrowLeft']) ? 1 : 0;
  var sr  = (keys['KeyD'] || keys['ArrowRight']) ? 1 : 0;
  var hb  = keys['Space'] ? 1 : 0;

  if (keys['KeyR']) {
    car.x=1200; car.y=1430; car.vx=0; car.vy=0; car.angle=0; car.steer=0;
  }

  var spd = Math.sqrt(car.vx*car.vx + car.vy*car.vy);
  var fx = Math.sin(car.angle), fy = -Math.cos(car.angle);
  var lx = Math.cos(car.angle), ly = Math.sin(car.angle);

  var fwdV = car.vx*fx + car.vy*fy;
  var latV = car.vx*lx + car.vy*ly;

  // Steering
  var targetSteer = (sl - sr) * CFG.steerMax * Math.min(1, spd / 1.5);
  car.steer += (targetSteer - car.steer) * CFG.steerSpeed * (dt * 60);
  car.steer += (0 - car.steer) * CFG.steerReturn * (dt * 60) * (1 - Math.abs(sl - sr));

  // Rotate car
  if (spd > 0.05) car.angle += car.steer * (fwdV >= 0 ? 1 : -1);

  // Engine force
  if (thr) {
    fwdV = Math.min(fwdV + CFG.acc, CFG.maxSpeed);
  } else if (brk) {
    if (fwdV > 0.1) fwdV = Math.max(0, fwdV - CFG.brk);
    else fwdV = Math.max(-CFG.rev * CFG.maxSpeed, fwdV - CFG.rev);
  } else {
    fwdV *= Math.pow(CFG.drag, dt * 60);
  }

  // Lateral grip
  var gripFactor = hb ? (1 - CFG.gripHB) : (1 - CFG.grip);
  latV *= Math.pow(1 - (hb ? 0.04 : 0.85), dt * 60);

  // Rebuild velocity
  car.vx = fx * fwdV + lx * latV;
  car.vy = fy * fwdV + ly * latV;

  // Move
  car.x += car.vx;
  car.y += car.vy;

  // World boundary
  car.x = Math.max(60, Math.min(WS-60, car.x));
  car.y = Math.max(60, Math.min(HS-60, car.y));

  // Camera — smooth follow with lookahead
  var lookX = car.x + fx * 80;
  var lookY = car.y + fy * 80;
  cam.sx = lerp(cam.sx, lookX, 7 * dt);
  cam.sy = lerp(cam.sy, lookY, 7 * dt);
  cam.x = cam.sx; cam.y = cam.sy;
  var targetScale = lerp(1.65, 1.05, spd / CFG.maxSpeed);
  cam.scale = lerp(cam.scale, targetScale, 2.5 * dt);

  // HUD
  var kmh = Math.abs(fwdV) * 28;
  document.getElementById('spd').textContent = Math.round(kmh);
  var rPct = Math.min(spd / CFG.maxSpeed, 1);
  document.getElementById('rpmfill').style.width = (rPct * 100) + '%';
  document.getElementById('rpmfill').style.background =
    rPct > .85 ? 'linear-gradient(90deg,#FF4400,#FF0000)' :
    rPct > .60 ? 'linear-gradient(90deg,#FFB400,#FF6600)' :
                 'linear-gradient(90deg,#FFB400,#FF4400)';

  var gear = 'N';
  if (fwdV < -0.1) gear = 'R';
  else if (spd < 0.1) gear = 'N';
  else if (kmh < 25) gear = 1;
  else if (kmh < 55) gear = 2;
  else if (kmh < 90) gear = 3;
  else if (kmh < 130) gear = 4;
  else gear = 5;
  document.getElementById('gval').textContent = gear;
}

// ── Draw functions ────────────────────────────────────────────────────────
function drawSky() {
  var g = ctx.createLinearGradient(0, 0, 0, gc.height * 0.4);
  g.addColorStop(0, '#3a7db5');
  g.addColorStop(1, '#87CEEB');
  ctx.fillStyle = g;
  ctx.fillRect(0, 0, gc.width, gc.height * 0.4);
}

function drawScene() {
  var cx = gc.width / 2 - cam.x * cam.scale;
  var cy = gc.height / 2 - cam.y * cam.scale;
  var sc = cam.scale;

  ctx.save();
  ctx.translate(cx, cy);
  ctx.scale(sc, sc);

  // Ground
  ctx.fillStyle = '#4a8c3f';
  ctx.fillRect(0, 0, WS, HS);

  // Outer boundary road edge
  ctx.strokeStyle = '#1a1a1a';
  ctx.lineWidth = 6;
  ctx.strokeRect(180, 180, 2040, 2040);

  // Water feature
  ctx.fillStyle = '#3a7fbf';
  ctx.beginPath(); ctx.ellipse(360, 360, 130, 90, 0.4, 0, Math.PI*2); ctx.fill();
  ctx.fillStyle = '#5a9ac0';
  ctx.beginPath(); ctx.ellipse(360, 360, 80, 52, 0.4, 0, Math.PI*2); ctx.fill();
  ctx.strokeStyle = '#2a6a9a'; ctx.lineWidth = 3;
  ctx.beginPath(); ctx.ellipse(360, 360, 130, 90, 0.4, 0, Math.PI*2); ctx.stroke();

  // Park
  ctx.fillStyle = '#5a9e42';
  ctx.beginPath(); ctx.arc(1200, 1200, 215, 0, Math.PI*2); ctx.fill();
  ctx.strokeStyle = '#3a7a2a'; ctx.lineWidth = 5;
  ctx.beginPath(); ctx.arc(1200, 1200, 215, 0, Math.PI*2); ctx.stroke();
  // Park path
  ctx.strokeStyle = '#d4c5a9'; ctx.lineWidth = 8; ctx.setLineDash([20,15]);
  ctx.beginPath(); ctx.arc(1200, 1200, 140, 0, Math.PI*2); ctx.stroke();
  ctx.setLineDash([]);

  // Road shadows
  ctx.lineCap = 'round';
  for (var i = 0; i < roads.length; i++) {
    var r = roads[i];
    ctx.strokeStyle = 'rgba(0,0,0,0.18)';
    ctx.lineWidth = r[4] + 10;
    ctx.beginPath(); ctx.moveTo(r[0], r[1]); ctx.lineTo(r[2], r[3]); ctx.stroke();
  }

  // Roads
  ctx.lineCap = 'square';
  for (var i = 0; i < roads.length; i++) {
    var r = roads[i];
    // asphalt
    ctx.strokeStyle = '#2d2d2d';
    ctx.lineWidth = r[4];
    ctx.beginPath(); ctx.moveTo(r[0], r[1]); ctx.lineTo(r[2], r[3]); ctx.stroke();
    // edge lines
    var dx = r[2]-r[0], dy = r[3]-r[1];
    var len = Math.sqrt(dx*dx+dy*dy);
    var nx = -dy/len, ny = dx/len;
    var off = r[4]/2 - 4;
    ctx.strokeStyle = 'rgba(200,200,200,0.25)';
    ctx.lineWidth = 2;
    ctx.beginPath(); ctx.moveTo(r[0]+nx*off,r[1]+ny*off); ctx.lineTo(r[2]+nx*off,r[3]+ny*off); ctx.stroke();
    ctx.beginPath(); ctx.moveTo(r[0]-nx*off,r[1]-ny*off); ctx.lineTo(r[2]-nx*off,r[3]-ny*off); ctx.stroke();
    // center dashes
    ctx.strokeStyle = '#ffffff';
    ctx.lineWidth = 2.5;
    ctx.setLineDash([28, 22]);
    ctx.beginPath(); ctx.moveTo(r[0], r[1]); ctx.lineTo(r[2], r[3]); ctx.stroke();
    ctx.setLineDash([]);
  }

  // Intersections
  for (var i = 0; i < intersections.length; i++) {
    var ix = intersections[i][0], iy = intersections[i][1];
    ctx.fillStyle = '#2a2a2a';
    ctx.fillRect(ix - 38, iy - 38, 76, 76);
  }

  // Trees (back)
  for (var i = 0; i < trees.length; i++) {
    var t = trees[i];
    ctx.fillStyle = 'rgba(0,0,0,0.15)';
    ctx.beginPath(); ctx.ellipse(t.x+5, t.y+5, t.r, t.r*0.65, 0, 0, Math.PI*2); ctx.fill();
    ctx.fillStyle = '#5c3d1e';
    ctx.fillRect(t.x-3, t.y-3, 6, 6);
    ctx.fillStyle = '#2d6a1f';
    ctx.beginPath(); ctx.arc(t.x, t.y, t.r, 0, Math.PI*2); ctx.fill();
    ctx.fillStyle = '#3d8a2a';
    ctx.beginPath(); ctx.arc(t.x - t.r*0.3, t.y - t.r*0.3, t.r*0.5, 0, Math.PI*2); ctx.fill();
  }

  // Buildings
  for (var i = 0; i < buildings.length; i++) {
    var b = buildings[i];
    var sh = b.sh;
    // drop shadow
    ctx.fillStyle = 'rgba(0,0,0,0.22)';
    ctx.fillRect(b.x + sh, b.y + sh, b.w, b.h);
    // right face
    ctx.fillStyle = shadeColor(b.color, -45);
    ctx.beginPath();
    ctx.moveTo(b.x+b.w, b.y);
    ctx.lineTo(b.x+b.w+sh, b.y-sh);
    ctx.lineTo(b.x+b.w+sh, b.y+b.h-sh);
    ctx.lineTo(b.x+b.w, b.y+b.h);
    ctx.fill();
    // top face
    ctx.fillStyle = shadeColor(b.color, 25);
    ctx.beginPath();
    ctx.moveTo(b.x, b.y);
    ctx.lineTo(b.x+sh, b.y-sh);
    ctx.lineTo(b.x+b.w+sh, b.y-sh);
    ctx.lineTo(b.x+b.w, b.y);
    ctx.fill();
    // front face
    ctx.fillStyle = b.color;
    ctx.fillRect(b.x, b.y, b.w, b.h);
    // windows
    var wIdx = 0;
    for (var row = 0; row < b.winRows; row++) {
      for (var col = 0; col < b.winCols; col++) {
        var wc = b.windows[wIdx++];
        if (wc) {
          ctx.fillStyle = wc;
          ctx.fillRect(b.x + 5 + col*18, b.y + 5 + row*20, 10, 13);
        }
      }
    }
    ctx.strokeStyle = 'rgba(0,0,0,0.35)';
    ctx.lineWidth = 1;
    ctx.strokeRect(b.x, b.y, b.w, b.h);
  }

  // Street lights
  var lpos = [
    [195,600],[195,800],[195,1000],[195,1400],[195,1600],[195,1800],
    [2205,600],[2205,800],[2205,1000],[2205,1400],[2205,1600],[2205,1800],
    [600,195],[800,195],[1000,195],[1400,195],[1600,195],[1800,195],
    [600,2205],[800,2205],[1000,2205],[1400,2205],[1600,2205],[1800,2205]
  ];
  for (var i = 0; i < lpos.length; i++) {
    var lx = lpos[i][0], ly = lpos[i][1];
    ctx.strokeStyle = '#777'; ctx.lineWidth = 4;
    ctx.beginPath(); ctx.moveTo(lx, ly+20); ctx.lineTo(lx, ly); ctx.stroke();
    ctx.fillStyle = 'rgba(255,255,150,0.18)';
    ctx.beginPath(); ctx.arc(lx, ly, 28, 0, Math.PI*2); ctx.fill();
    ctx.fillStyle = 'rgba(255,255,150,0.9)';
    ctx.beginPath(); ctx.arc(lx, ly, 6, 0, Math.PI*2); ctx.fill();
  }

  // Car
  drawCar();

  ctx.restore();
}

function drawCar() {
  ctx.save();
  ctx.translate(car.x, car.y);
  ctx.rotate(car.angle);

  var L = 38, W2 = 18;
  var hb = keys['Space'];
  var brk = keys['KeyS'] || keys['ArrowDown'];

  // Shadow
  ctx.fillStyle = 'rgba(0,0,0,0.2)';
  ctx.beginPath(); ctx.ellipse(4, 4, W2+2, L*0.35, 0, 0, Math.PI*2); ctx.fill();

  // Wheels (back first)
  drawWheel(-W2/2-2,  L/2-8, 0, false);
  drawWheel( W2/2+2,  L/2-8, 0, false);
  drawWheel(-W2/2-2, -L/2+8, car.steer * 12, false);
  drawWheel( W2/2+2, -L/2+8, car.steer * 12, false);

  // Body
  ctx.fillStyle = '#cc2211';
  rrect(ctx, -W2/2, -L/2, W2, L, 4); ctx.fill();

  // Hood detail
  ctx.fillStyle = '#b51d10';
  rrect(ctx, -W2/2+2, -L/2+2, W2-4, L*0.38, 3); ctx.fill();

  // Cabin
  ctx.fillStyle = '#aa1a0e';
  rrect(ctx, -W2/2+2, -L/2+L*0.22, W2-4, L*0.42, 3); ctx.fill();

  // Windshield
  ctx.fillStyle = 'rgba(140,210,255,0.55)';
  rrect(ctx, -W2/2+3.5, -L/2+L*0.23, W2-7, L*0.13, 2); ctx.fill();

  // Rear window
  ctx.fillStyle = 'rgba(140,210,255,0.45)';
  rrect(ctx, -W2/2+3.5, -L/2+L*0.58, W2-7, L*0.09, 2); ctx.fill();

  // Headlights
  ctx.fillStyle = 'rgba(255,255,200,0.95)';
  ctx.fillRect(-W2/2+1.5, -L/2+2, 5, 4);
  ctx.fillRect(W2/2-6.5,  -L/2+2, 5, 4);

  // Glow
  ctx.fillStyle = 'rgba(255,255,150,0.18)';
  ctx.beginPath(); ctx.arc(-W2/2+4, -L/2+3, 10, 0, Math.PI*2); ctx.fill();
  ctx.beginPath(); ctx.arc( W2/2-4, -L/2+3, 10, 0, Math.PI*2); ctx.fill();

  // Taillights
  ctx.fillStyle = brk ? '#ff5500' : '#cc0000';
  ctx.fillRect(-W2/2+1.5, L/2-6, 5, 4);
  ctx.fillRect(W2/2-6.5,  L/2-6, 5, 4);

  // Spoiler
  ctx.fillStyle = '#111';
  ctx.fillRect(-W2/2+1, L/2-4, W2-2, 3);

  // Body outline
  ctx.strokeStyle = 'rgba(0,0,0,0.55)'; ctx.lineWidth = 1;
  rrect(ctx, -W2/2, -L/2, W2, L, 4); ctx.stroke();

  ctx.restore();
}

function drawWheel(ox, oy, steerDeg, spinning) {
  ctx.save();
  ctx.translate(ox, oy);
  ctx.rotate(steerDeg * Math.PI / 180);
  ctx.fillStyle = '#111';
  ctx.fillRect(-4, -7, 8, 14);
  ctx.fillStyle = '#888';
  ctx.fillRect(-2, -5, 4, 10);
  ctx.fillStyle = '#555';
  ctx.fillRect(-0.5, -6, 1, 12);
  ctx.restore();
}

function drawMinimap() {
  mc.clearRect(0, 0, 130, 130);
  mc.fillStyle = 'rgba(0,18,0,0.9)';
  mc.fillRect(0, 0, 130, 130);
  var s = 130 / WS;

  mc.fillStyle = 'rgba(90,158,66,0.35)';
  mc.beginPath(); mc.arc(1200*s, 1200*s, 215*s, 0, Math.PI*2); mc.fill();

  mc.strokeStyle = '#3a3a3a'; mc.lineWidth = 2;
  for (var i = 0; i < roads.length; i++) {
    var r = roads[i];
    mc.beginPath(); mc.moveTo(r[0]*s, r[1]*s); mc.lineTo(r[2]*s, r[3]*s); mc.stroke();
  }

  mc.save();
  mc.translate(car.x*s, car.y*s);
  mc.rotate(car.angle);
  mc.fillStyle = '#FFB400';
  mc.fillRect(-3, -5, 6, 10);
  mc.restore();

  mc.strokeStyle = 'rgba(255,180,0,0.3)'; mc.lineWidth = 1;
  mc.strokeRect(0, 0, 130, 130);
}

// ── Main loop ─────────────────────────────────────────────────────────────
var prev = performance.now();
var started = false;

function loop(now) {
  requestAnimationFrame(loop);
  var dt = (now - prev) / 1000;
  prev = now;

  updatePhysics(dt);

  ctx.clearRect(0, 0, gc.width, gc.height);
  drawSky();
  drawScene();
  drawMinimap();

  if (!started) {
    started = true;
    document.getElementById('loading').style.display = 'none';
  }
}

requestAnimationFrame(loop);
</script>

</body>
</html>