<!DOCTYPE html>

<html lang="en">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>APEX CHASE</title>
<style>
  *{margin:0;padding:0;box-sizing:border-box}
  body{background:#000;overflow:hidden;font-family:'Courier New',monospace}
  canvas{display:block;position:absolute;top:0;left:0}
  #hud{position:fixed;top:0;left:0;width:100%;height:100%;pointer-events:none;z-index:5}

/* Speedometer */
#speedo{position:absolute;bottom:28px;right:36px;background:rgba(0,0,0,.8);border:1px solid rgba(255,180,0,.5);border-radius:14px;padding:16px 24px;text-align:center}
#spd{font-size:50px;font-weight:900;color:#FFB400;line-height:1;letter-spacing:-2px;text-shadow:0 0 18px rgba(255,180,0,.7)}
#sunit{font-size:11px;color:rgba(255,180,0,.5);letter-spacing:4px;margin-top:3px}

/* Gear */
#gearbox{position:absolute;bottom:28px;right:220px;background:rgba(0,0,0,.8);border:1px solid rgba(255,180,0,.3);border-radius:12px;padding:12px 18px;text-align:center}
#gval{font-size:40px;font-weight:900;color:#fff;line-height:1}
#glabel{font-size:10px;color:rgba(255,255,255,.4);letter-spacing:3px;margin-top:3px}

/* RPM */
#rpmwrap{position:absolute;bottom:148px;right:36px;width:240px}
#rpmlbl{font-size:10px;color:rgba(255,180,0,.5);letter-spacing:4px;text-align:right;margin-bottom:5px}
#rpmbg{width:100%;height:7px;background:rgba(255,255,255,.08);border-radius:4px;overflow:hidden}
#rpmfill{height:100%;width:0%;border-radius:4px;transition:width .04s}

/* Controls */
#hints{position:absolute;bottom:28px;left:36px;background:rgba(0,0,0,.75);border:1px solid rgba(255,255,255,.1);border-radius:12px;padding:12px 16px}
#hints div{font-size:11px;color:rgba(255,255,255,.45);line-height:1.9}
#hints b{color:#FFB400}

/* Minimap */
#mmwrap{position:absolute;top:18px;right:26px;width:130px;height:130px;background:rgba(0,0,0,.75);border:1px solid rgba(255,180,0,.25);border-radius:10px;overflow:hidden}

/* Banner */
#banner{position:absolute;top:22px;left:50%;transform:translateX(-50%);font-size:12px;letter-spacing:8px;color:rgba(255,180,0,.8);font-weight:700;white-space:nowrap}

/* Wanted level */
#wanted{position:absolute;top:18px;left:36px;background:rgba(0,0,0,.8);border:1px solid rgba(255,50,50,.4);border-radius:12px;padding:12px 18px}
#wlabel{font-size:10px;color:rgba(255,80,80,.7);letter-spacing:4px;margin-bottom:8px}
#stars{display:flex;gap:5px}
.star{font-size:18px;color:rgba(255,255,255,.15);transition:color .2s,text-shadow .2s}
.star.on{color:#FFD700;text-shadow:0 0 8px #FFD700}

/* Cop proximity warning */
#copwarn{position:absolute;top:50%;left:50%;transform:translate(-50%,-50%);font-size:14px;letter-spacing:6px;color:#ff3333;font-weight:900;text-shadow:0 0 20px #ff0000;opacity:0;transition:opacity .2s;pointer-events:none;text-align:center}

/* BUSTED overlay */
#busted{display:none;position:fixed;top:0;left:0;width:100%;height:100%;background:rgba(0,0,0,.85);z-index:50;flex-direction:column;align-items:center;justify-content:center}
#busted-title{font-size:88px;font-weight:900;color:#ff2200;letter-spacing:12px;text-shadow:0 0 40px #ff0000,0 0 80px #ff0000;animation:bust-pulse 0.5s infinite alternate}
#busted-sub{font-size:16px;letter-spacing:6px;color:rgba(255,255,255,.6);margin-top:16px}
#busted-timer{font-size:28px;color:#FFB400;margin-top:24px;letter-spacing:4px}
@keyframes bust-pulse{from{transform:scale(1)}to{transform:scale(1.06)}}

/* Loading */
#loading{position:fixed;top:0;left:0;width:100%;height:100%;background:#000;display:flex;flex-direction:column;align-items:center;justify-content:center;z-index:99;color:#FFB400}
#loading h1{font-size:36px;letter-spacing:10px;margin-bottom:18px}
#loading p{font-size:13px;letter-spacing:4px;color:rgba(255,180,0,.6);animation:blink 1s infinite}
@keyframes blink{0%,100%{opacity:1}50%{opacity:.3}}

/* Siren flash */
#siren{position:fixed;top:0;left:0;width:100%;height:100%;pointer-events:none;z-index:4;opacity:0;transition:opacity .08s}
</style>

</head>
<body>
<div id="loading"><h1>APEX CHASE</h1><p>LOADING WORLD...</p></div>
<div id="siren"></div>
<canvas id="gc"></canvas>
<div id="hud">
  <div id="banner">APEX CHASE</div>
  <div id="mmwrap"><canvas id="mm" width="130" height="130"></canvas></div>
  <div id="rpmwrap"><div id="rpmlbl">RPM</div><div id="rpmbg"><div id="rpmfill"></div></div></div>
  <div id="gearbox"><div id="gval">N</div><div id="glabel">GEAR</div></div>
  <div id="speedo"><div id="spd">0</div><div id="sunit">KM/H</div></div>
  <div id="wanted">
    <div id="wlabel">WANTED</div>
    <div id="stars">
      <span class="star" id="s0">★</span>
      <span class="star" id="s1">★</span>
      <span class="star" id="s2">★</span>
      <span class="star" id="s3">★</span>
      <span class="star" id="s4">★</span>
    </div>
  </div>
  <div id="copwarn">⚠ COP CLOSE ⚠</div>
  <div id="hints">
    <div><b>W/↑</b> Accelerate</div>
    <div><b>S/↓</b> Brake/Reverse</div>
    <div><b>A D/← →</b> Steer</div>
    <div><b>SPACE</b> Handbrake</div>
    <div><b>R</b> Reset</div>
  </div>
</div>
<div id="busted">
  <div id="busted-title">BUSTED!</div>
  <div id="busted-sub">YOU WERE CAUGHT BY THE POLICE</div>
  <div id="busted-timer">RESTARTING IN <span id="btcount">3</span>...</div>
</div>

<script>
'use strict';
// ── Canvas ────────────────────────────────────────────────────────────────
const gc = document.getElementById('gc');
const ctx = gc.getContext('2d');
const mm = document.getElementById('mm');
const mc = mm.getContext('2d');
function resize(){ gc.width=window.innerWidth; gc.height=window.innerHeight; }
resize(); window.addEventListener('resize',resize);

const WS=2400,HS=2400;
function lerp(a,b,t){return a+(b-a)*t;}
function rnd(a,b){return a+Math.random()*(b-a);}
function ri(a,b){return Math.floor(rnd(a,b));}
function dist2(ax,ay,bx,by){const dx=ax-bx,dy=ay-by;return dx*dx+dy*dy;}
function rrect(c,x,y,w,h,r){c.beginPath();c.moveTo(x+r,y);c.lineTo(x+w-r,y);c.arcTo(x+w,y,x+w,y+r,r);c.lineTo(x+w,y+h-r);c.arcTo(x+w,y+h,x+w-r,y+h,r);c.lineTo(x+r,y+h);c.arcTo(x,y+h,x,y+h-r,r);c.lineTo(x,y+r);c.arcTo(x,y,x+r,y,r);c.closePath();}
function shadeColor(hex,amt){const n=parseInt(hex.replace('#',''),16);const r=Math.min(255,Math.max(0,((n>>16)&0xff)+amt));const g=Math.min(255,Math.max(0,((n>>8)&0xff)+amt));const b=Math.min(255,Math.max(0,(n&0xff)+amt));return`rgb(${r},${g},${b})`;}
function preRand(x,y){return(((Math.sin(x*127.1+y*311.7)*43758.5453)%1)+1)%1;}
function angleDiff(a,b){let d=b-a;while(d>Math.PI)d-=2*Math.PI;while(d<-Math.PI)d+=2*Math.PI;return d;}

// ── Roads ─────────────────────────────────────────────────────────────────
const roads=[
  [200,200,2200,200,58],[2200,200,2200,2200,58],[2200,2200,200,2200,58],[200,2200,200,200,58],
  [600,600,1800,600,52],[1800,600,1800,1800,52],[1800,1800,600,1800,52],[600,1800,600,600,52],
  [200,1200,2200,1200,52],[1200,200,1200,2200,52],
  [200,200,600,600,38],[2200,200,1800,600,38],[2200,2200,1800,1800,38],[200,2200,600,1800,38],
  [600,600,200,400,32],[1800,600,2200,400,32],[600,1800,200,2000,32],[1800,1800,2200,2000,32],
  [1200,200,1350,50,30],[1200,2200,1350,2380,30],[200,1200,50,1350,30],[2200,1200,2380,1350,30],
];

// Waypoints cops use to navigate
const waypointGrid=[
  [200,200],[1200,200],[2200,200],
  [200,600],[600,600],[1200,600],[1800,600],[2200,600],
  [200,1200],[600,1200],[1200,1200],[1800,1200],[2200,1200],
  [200,1800],[600,1800],[1200,1800],[1800,1800],[2200,1800],
  [200,2200],[1200,2200],[2200,2200],
  [600,200],[1800,200],[600,2200],[1800,2200],
];

// ── Buildings ─────────────────────────────────────────────────────────────
const buildingColors=['#8899aa','#99aabb','#7a8fa0','#b0c4d4','#6b8096','#d4c5a9','#c9b99a','#a0b0c0'];
const bPositions=[
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
const buildings=[];
bPositions.forEach(p=>{
  const w=rnd(48,105),h=rnd(48,105);
  const col=buildingColors[ri(0,buildingColors.length)];
  const sh=rnd(8,26);
  const wCols=Math.floor(w/18),wRows=Math.floor(h/20);
  const wins=[];
  for(let r=0;r<wRows;r++)for(let c=0;c<wCols;c++)
    wins.push(preRand(p.x+c*77,p.y+r*77)<0.65?(preRand(p.x+c*13,p.y+r*17)<0.35?'#ffffaa':'#253040'):null);
  buildings.push({x:p.x+rnd(-12,12),y:p.y+rnd(-12,12),w,h,col,sh,wins,wCols,wRows});
});

// ── Trees ─────────────────────────────────────────────────────────────────
const trees=[];
for(let i=0;i<300;i++) trees.push({x:rnd(60,WS-60),y:rnd(60,HS-60),r:rnd(10,20)});
for(let i=0;i<60;i++){const a=Math.random()*Math.PI*2,d=rnd(30,200);trees.push({x:1200+Math.cos(a)*d,y:1200+Math.sin(a)*d,r:rnd(10,18)});}

const intersections=[[200,200],[2200,200],[2200,2200],[200,2200],[600,600],[1800,600],[1800,1800],[600,1800],[1200,1200],[200,1200],[2200,1200],[1200,200],[1200,2200],[600,1200],[1800,1200],[1200,600],[1200,1800]];

// ── Physics config ────────────────────────────────────────────────────────
const CFG={maxSpeed:8.0,acc:0.18,brk:0.28,rev:0.08,steerMax:0.048,steerSpeed:0.15,steerReturn:0.12,grip:0.82,gripHB:0.96,drag:0.976};
const COP_CFG={maxSpeed:7.2,acc:0.16,steerMax:0.052,grip:0.78,drag:0.974};

// ── Player ────────────────────────────────────────────────────────────────
let player={x:1200,y:1430,vx:0,vy:0,angle:0,steer:0};
const keys={};
window.addEventListener('keydown',e=>{keys[e.code]=true;if(['Space','ArrowUp','ArrowDown','ArrowLeft','ArrowRight'].indexOf(e.code)>=0)e.preventDefault();});
window.addEventListener('keyup',e=>{keys[e.code]=false;});

// ── Camera ────────────────────────────────────────────────────────────────
let cam={x:1200,y:1430,sx:1200,sy:1430,scale:1.4};

// ── Cops ──────────────────────────────────────────────────────────────────
const COP_COLORS=['#1a3a8c','#1a3a8c','#0d2a6e'];
const COP_SPAWN_POINTS=[
  [400,400],[2000,400],[400,2000],[2000,2000],
  [1200,400],[400,1200],[2000,1200],[1200,2000],
];

function makeCop(spawnIdx){
  const sp=COP_SPAWN_POINTS[spawnIdx%COP_SPAWN_POINTS.length];
  return{
    x:sp[0],y:sp[1],vx:0,vy:0,angle:0,steer:0,
    state:'chase',// chase, intercept
    wpIdx:0,
    sirenPhase:Math.random()*Math.PI*2,
    active:true,
    catchTimer:0,
  };
}

let cops=[];
let wantedLevel=0;
let wantedTimer=0; // time at current wanted level
let gameState='playing'; // playing, busted
let bustedCountdown=3;
let bustedTimer=0;

// spawn initial cops
function initCops(){
  cops=[];
  // start with 1 cop, more appear as wanted rises
  cops.push(makeCop(0));
}

function resetGame(){
  player={x:1200,y:1430,vx:0,vy:0,angle:0,steer:0};
  cam={x:1200,y:1430,sx:1200,sy:1430,scale:1.4};
  wantedLevel=1;
  wantedTimer=0;
  gameState='playing';
  bustedCountdown=3;
  document.getElementById('busted').style.display='none';
  document.getElementById('siren').style.opacity=0;
  initCops();
  updateStars();
}

function updateStars(){
  for(let i=0;i<5;i++){
    document.getElementById('s'+i).className='star'+(i<wantedLevel?' on':'');
  }
}

// ── Cop AI ────────────────────────────────────────────────────────────────
function updateCop(cop,dt,px,py,pangle){
  if(!cop.active)return;

  // Target: always try to reach player
  let tx=px,ty=py;

  // Predict player position slightly ahead for interception
  const predTime=0.8;
  const pspd=Math.sqrt(player.vx*player.vx+player.vy*player.vy);
  if(pspd>1){
    tx=px+player.vx*predTime*18;
    ty=py+player.vy*predTime*18;
  }

  // Angle to target
  const dx=tx-cop.x, dy=ty-cop.y;
  const targetAngle=Math.atan2(dx,-dy);
  const dAngle=angleDiff(cop.angle,targetAngle);

  // Steer toward target
  const steerTarget=Math.max(-COP_CFG.steerMax,Math.min(COP_CFG.steerMax,dAngle*0.7));
  cop.steer=lerp(cop.steer,steerTarget,0.15*(dt*60));

  const copSpd=Math.sqrt(cop.vx*cop.vx+cop.vy*cop.vy);
  if(copSpd>0.05) cop.angle+=cop.steer*(1);

  const fx=Math.sin(cop.angle),fy=-Math.cos(cop.angle);
  const lx=Math.cos(cop.angle),ly=Math.sin(cop.angle);
  let fwdV=cop.vx*fx+cop.vy*fy;
  let latV=cop.vx*lx+cop.vy*ly;

  // Always accelerate toward player
  const distToPlayer=Math.sqrt(dist2(cop.x,cop.y,px,py));

  if(distToPlayer>30){
    fwdV=Math.min(fwdV+COP_CFG.acc,COP_CFG.maxSpeed);
  } else {
    // slow near player to avoid overshoot
    fwdV*=Math.pow(0.92,dt*60);
  }

  // Grip
  latV*=Math.pow(0.12,dt*60);

  cop.vx=fx*fwdV+lx*latV;
  cop.vy=fy*fwdV+ly*latV;
  cop.x+=cop.vx;
  cop.y+=cop.vy;
  cop.x=Math.max(60,Math.min(WS-60,cop.x));
  cop.y=Math.max(60,Math.min(HS-60,cop.y));

  // Siren phase
  cop.sirenPhase+=dt*8;
}

// ── Game logic ────────────────────────────────────────────────────────────
function updateWanted(dt){
  wantedTimer+=dt;
  // Escalate wanted level over time
  if(wantedLevel<5 && wantedTimer>15){
    wantedLevel=Math.min(5,wantedLevel+1);
    wantedTimer=0;
    updateStars();
    // spawn more cops
    if(cops.length<wantedLevel+1){
      cops.push(makeCop(cops.length));
    }
  }

  // Check if any cop caught player
  if(gameState!=='playing')return;
  let closestDist=Infinity;
  cops.forEach(cop=>{
    if(!cop.active)return;
    const d=Math.sqrt(dist2(cop.x,cop.y,player.x,player.y));
    if(d<closestDist)closestDist=d;
    if(d<28){ // caught!
      triggerBusted();
    }
  });

  // Warning
  const warn=document.getElementById('copwarn');
  const siren=document.getElementById('siren');
  if(closestDist<200){
    warn.style.opacity='1';
    const sirenIntensity=Math.max(0,(200-closestDist)/200)*0.12;
    const sirenColor=Math.sin(Date.now()*0.015)>0?`rgba(255,50,50,${sirenIntensity})`:`rgba(50,50,255,${sirenIntensity})`;
    siren.style.background=sirenColor;
    siren.style.opacity='1';
  } else {
    warn.style.opacity='0';
    siren.style.opacity='0';
    siren.style.background='transparent';
  }
}

function triggerBusted(){
  if(gameState==='busted')return;
  gameState='busted';
  document.getElementById('busted').style.display='flex';
  bustedCountdown=3;
  document.getElementById('btcount').textContent=bustedCountdown;
  const iv=setInterval(()=>{
    bustedCountdown--;
    document.getElementById('btcount').textContent=Math.max(0,bustedCountdown);
    if(bustedCountdown<=0){clearInterval(iv);resetGame();}
  },1000);
}

// ── Player physics ────────────────────────────────────────────────────────
function updatePlayer(dt){
  if(gameState!=='playing')return;
  dt=Math.min(dt,0.05);
  const thr=(keys['KeyW']||keys['ArrowUp'])?1:0;
  const brk=(keys['KeyS']||keys['ArrowDown'])?1:0;
  const sl=(keys['KeyA']||keys['ArrowLeft'])?1:0;
  const sr=(keys['KeyD']||keys['ArrowRight'])?1:0;
  const hb=keys['Space']?1:0;

  if(keys['KeyR']){player.x=1200;player.y=1430;player.vx=0;player.vy=0;player.angle=0;player.steer=0;}

  const spd=Math.sqrt(player.vx*player.vx+player.vy*player.vy);
  const fx=Math.sin(player.angle),fy=-Math.cos(player.angle);
  const lx=Math.cos(player.angle),ly=Math.sin(player.angle);
  let fwdV=player.vx*fx+player.vy*fy;
  let latV=player.vx*lx+player.vy*ly;

  const tSt=(sl-sr)*CFG.steerMax*Math.min(1,spd/1.5);
  player.steer+=(tSt-player.steer)*CFG.steerSpeed*(dt*60);
  player.steer+=(0-player.steer)*CFG.steerReturn*(dt*60)*(1-Math.abs(sl-sr));
  if(spd>0.05) player.angle+=player.steer*(fwdV>=0?1:-1);

  if(thr)fwdV=Math.min(fwdV+CFG.acc,CFG.maxSpeed);
  else if(brk){if(fwdV>0.1)fwdV=Math.max(0,fwdV-CFG.brk);else fwdV=Math.max(-CFG.rev*CFG.maxSpeed,fwdV-CFG.rev);}
  else fwdV*=Math.pow(CFG.drag,dt*60);

  latV*=Math.pow(hb?0.97:0.15,dt*60);
  player.vx=fx*fwdV+lx*latV;
  player.vy=fy*fwdV+ly*latV;
  player.x+=player.vx; player.y+=player.vy;
  player.x=Math.max(60,Math.min(WS-60,player.x));
  player.y=Math.max(60,Math.min(HS-60,player.y));

  // camera
  const lookX=player.x+Math.sin(player.angle)*80;
  const lookY=player.y-Math.cos(player.angle)*80;
  cam.sx=lerp(cam.sx,lookX,7*dt);
  cam.sy=lerp(cam.sy,lookY,7*dt);
  cam.x=cam.sx; cam.y=cam.sy;
  const ts=lerp(1.65,1.05,spd/CFG.maxSpeed);
  cam.scale=lerp(cam.scale,ts,2.5*dt);

  // HUD
  const kmh=Math.abs(fwdV)*28;
  document.getElementById('spd').textContent=Math.round(kmh);
  const rPct=Math.min(spd/CFG.maxSpeed,1);
  const rf=document.getElementById('rpmfill');
  rf.style.width=(rPct*100)+'%';
  rf.style.background=rPct>.85?'linear-gradient(90deg,#FF4400,#FF0000)':rPct>.60?'linear-gradient(90deg,#FFB400,#FF6600)':'linear-gradient(90deg,#FFB400,#FF4400)';
  let gear='N';
  if(fwdV<-0.1)gear='R';else if(spd<0.1)gear='N';
  else if(kmh<25)gear=1;else if(kmh<55)gear=2;else if(kmh<90)gear=3;else if(kmh<130)gear=4;else gear=5;
  document.getElementById('gval').textContent=gear;
}

// ── Draw ──────────────────────────────────────────────────────────────────
function drawSky(){
  const g=ctx.createLinearGradient(0,0,0,gc.height*0.4);
  g.addColorStop(0,'#3a7db5');g.addColorStop(1,'#87CEEB');
  ctx.fillStyle=g; ctx.fillRect(0,0,gc.width,gc.height*0.4);
}

function drawScene(){
  const cx=gc.width/2-cam.x*cam.scale;
  const cy=gc.height/2-cam.y*cam.scale;
  const sc=cam.scale;
  ctx.save();
  ctx.translate(cx,cy);
  ctx.scale(sc,sc);

  // Ground
  ctx.fillStyle='#4a8c3f'; ctx.fillRect(0,0,WS,HS);

  // Water
  ctx.fillStyle='#3a7fbf'; ctx.beginPath(); ctx.ellipse(360,360,130,90,0.4,0,Math.PI*2); ctx.fill();
  ctx.fillStyle='#5a9ac0'; ctx.beginPath(); ctx.ellipse(360,360,80,52,0.4,0,Math.PI*2); ctx.fill();
  ctx.strokeStyle='#2a6a9a'; ctx.lineWidth=3; ctx.beginPath(); ctx.ellipse(360,360,130,90,0.4,0,Math.PI*2); ctx.stroke();

  // Park
  ctx.fillStyle='#5a9e42'; ctx.beginPath(); ctx.arc(1200,1200,215,0,Math.PI*2); ctx.fill();
  ctx.strokeStyle='#3a7a2a'; ctx.lineWidth=5; ctx.beginPath(); ctx.arc(1200,1200,215,0,Math.PI*2); ctx.stroke();
  ctx.strokeStyle='#d4c5a9'; ctx.lineWidth=8; ctx.setLineDash([20,15]);
  ctx.beginPath(); ctx.arc(1200,1200,140,0,Math.PI*2); ctx.stroke(); ctx.setLineDash([]);

  // Road shadows
  ctx.lineCap='round';
  roads.forEach(r=>{
    ctx.strokeStyle='rgba(0,0,0,0.18)'; ctx.lineWidth=r[4]+10;
    ctx.beginPath(); ctx.moveTo(r[0],r[1]); ctx.lineTo(r[2],r[3]); ctx.stroke();
  });

  // Roads
  ctx.lineCap='square';
  roads.forEach(r=>{
    ctx.strokeStyle='#2d2d2d'; ctx.lineWidth=r[4];
    ctx.beginPath(); ctx.moveTo(r[0],r[1]); ctx.lineTo(r[2],r[3]); ctx.stroke();
    const dx=r[2]-r[0],dy=r[3]-r[1],len=Math.sqrt(dx*dx+dy*dy);
    const nx=-dy/len,ny=dx/len,off=r[4]/2-4;
    ctx.strokeStyle='rgba(200,200,200,0.22)'; ctx.lineWidth=2;
    ctx.beginPath(); ctx.moveTo(r[0]+nx*off,r[1]+ny*off); ctx.lineTo(r[2]+nx*off,r[3]+ny*off); ctx.stroke();
    ctx.beginPath(); ctx.moveTo(r[0]-nx*off,r[1]-ny*off); ctx.lineTo(r[2]-nx*off,r[3]-ny*off); ctx.stroke();
    ctx.strokeStyle='#ffffff'; ctx.lineWidth=2.5; ctx.setLineDash([28,22]);
    ctx.beginPath(); ctx.moveTo(r[0],r[1]); ctx.lineTo(r[2],r[3]); ctx.stroke(); ctx.setLineDash([]);
  });

  // Intersections
  intersections.forEach(([ix,iy])=>{
    ctx.fillStyle='#2a2a2a'; ctx.fillRect(ix-38,iy-38,76,76);
  });

  // Trees
  trees.forEach(t=>{
    ctx.fillStyle='rgba(0,0,0,0.14)'; ctx.beginPath(); ctx.ellipse(t.x+5,t.y+5,t.r,t.r*0.65,0,0,Math.PI*2); ctx.fill();
    ctx.fillStyle='#5c3d1e'; ctx.fillRect(t.x-3,t.y-3,6,6);
    ctx.fillStyle='#2d6a1f'; ctx.beginPath(); ctx.arc(t.x,t.y,t.r,0,Math.PI*2); ctx.fill();
    ctx.fillStyle='#3d8a2a'; ctx.beginPath(); ctx.arc(t.x-t.r*.3,t.y-t.r*.3,t.r*.5,0,Math.PI*2); ctx.fill();
  });

  // Buildings
  buildings.forEach(b=>{
    ctx.fillStyle='rgba(0,0,0,0.2)'; ctx.fillRect(b.x+b.sh,b.y+b.sh,b.w,b.h);
    ctx.fillStyle=shadeColor(b.col,-45);
    ctx.beginPath(); ctx.moveTo(b.x+b.w,b.y); ctx.lineTo(b.x+b.w+b.sh,b.y-b.sh); ctx.lineTo(b.x+b.w+b.sh,b.y+b.h-b.sh); ctx.lineTo(b.x+b.w,b.y+b.h); ctx.fill();
    ctx.fillStyle=shadeColor(b.col,25);
    ctx.beginPath(); ctx.moveTo(b.x,b.y); ctx.lineTo(b.x+b.sh,b.y-b.sh); ctx.lineTo(b.x+b.w+b.sh,b.y-b.sh); ctx.lineTo(b.x+b.w,b.y); ctx.fill();
    ctx.fillStyle=b.col; ctx.fillRect(b.x,b.y,b.w,b.h);
    let wi=0;
    for(let r=0;r<b.wRows;r++) for(let c=0;c<b.wCols;c++){
      const wc=b.wins[wi++];
      if(wc){ctx.fillStyle=wc; ctx.fillRect(b.x+5+c*18,b.y+5+r*20,10,13);}
    }
    ctx.strokeStyle='rgba(0,0,0,0.35)'; ctx.lineWidth=1; ctx.strokeRect(b.x,b.y,b.w,b.h);
  });

  // Street lights
  [[195,600],[195,800],[195,1000],[195,1400],[195,1600],[195,1800],
   [2205,600],[2205,800],[2205,1000],[2205,1400],[2205,1600],[2205,1800],
   [600,195],[800,195],[1000,195],[1400,195],[1600,195],[1800,195],
   [600,2205],[800,2205],[1000,2205],[1400,2205],[1600,2205],[1800,2205]].forEach(([lx,ly])=>{
    ctx.strokeStyle='#777'; ctx.lineWidth=4; ctx.beginPath(); ctx.moveTo(lx,ly+20); ctx.lineTo(lx,ly); ctx.stroke();
    ctx.fillStyle='rgba(255,255,150,0.18)'; ctx.beginPath(); ctx.arc(lx,ly,28,0,Math.PI*2); ctx.fill();
    ctx.fillStyle='rgba(255,255,150,0.9)'; ctx.beginPath(); ctx.arc(lx,ly,6,0,Math.PI*2); ctx.fill();
  });

  // Draw cops
  cops.forEach(cop=>{ if(cop.active) drawCopCar(cop); });

  // Draw player
  drawPlayerCar();

  ctx.restore();
}

function drawPlayerCar(){
  ctx.save();
  ctx.translate(player.x,player.y);
  ctx.rotate(player.angle);
  const L=38,W2=18;
  const brk=keys['KeyS']||keys['ArrowDown'];
  ctx.fillStyle='rgba(0,0,0,0.2)'; ctx.beginPath(); ctx.ellipse(4,4,W2+2,L*.35,0,0,Math.PI*2); ctx.fill();
  drawWheel(ctx,-W2/2-2,L/2-8,0);drawWheel(ctx,W2/2+2,L/2-8,0);
  drawWheel(ctx,-W2/2-2,-L/2+8,player.steer*12);drawWheel(ctx,W2/2+2,-L/2+8,player.steer*12);
  ctx.fillStyle='#cc2211'; rrect(ctx,-W2/2,-L/2,W2,L,4); ctx.fill();
  ctx.fillStyle='#b51d10'; rrect(ctx,-W2/2+2,-L/2+2,W2-4,L*.38,3); ctx.fill();
  ctx.fillStyle='#aa1a0e'; rrect(ctx,-W2/2+2,-L/2+L*.22,W2-4,L*.42,3); ctx.fill();
  ctx.fillStyle='rgba(140,210,255,0.55)'; rrect(ctx,-W2/2+3.5,-L/2+L*.23,W2-7,L*.13,2); ctx.fill();
  ctx.fillStyle='rgba(140,210,255,0.45)'; rrect(ctx,-W2/2+3.5,-L/2+L*.58,W2-7,L*.09,2); ctx.fill();
  ctx.fillStyle='rgba(255,255,200,0.95)'; ctx.fillRect(-W2/2+1.5,-L/2+2,5,4); ctx.fillRect(W2/2-6.5,-L/2+2,5,4);
  ctx.fillStyle='rgba(255,255,150,0.15)'; ctx.beginPath(); ctx.arc(-W2/2+4,-L/2+3,10,0,Math.PI*2); ctx.fill(); ctx.beginPath(); ctx.arc(W2/2-4,-L/2+3,10,0,Math.PI*2); ctx.fill();
  ctx.fillStyle=brk?'#ff5500':'#cc0000'; ctx.fillRect(-W2/2+1.5,L/2-6,5,4); ctx.fillRect(W2/2-6.5,L/2-6,5,4);
  ctx.fillStyle='#111'; ctx.fillRect(-W2/2+1,L/2-4,W2-2,3);
  ctx.strokeStyle='rgba(0,0,0,0.5)'; ctx.lineWidth=1; rrect(ctx,-W2/2,-L/2,W2,L,4); ctx.stroke();
  ctx.restore();
}

function drawCopCar(cop){
  ctx.save();
  ctx.translate(cop.x,cop.y);
  ctx.rotate(cop.angle);
  const L=38,W2=18;
  // Siren light glow
  const sirenOn=Math.sin(cop.sirenPhase)>0;
  if(sirenOn){
    ctx.fillStyle='rgba(0,50,255,0.18)';
    ctx.beginPath(); ctx.arc(0,0,40,0,Math.PI*2); ctx.fill();
  } else {
    ctx.fillStyle='rgba(255,30,30,0.18)';
    ctx.beginPath(); ctx.arc(0,0,40,0,Math.PI*2); ctx.fill();
  }

  ctx.fillStyle='rgba(0,0,0,0.2)'; ctx.beginPath(); ctx.ellipse(4,4,W2+2,L*.35,0,0,Math.PI*2); ctx.fill();
  drawWheel(ctx,-W2/2-2,L/2-8,0);drawWheel(ctx,W2/2+2,L/2-8,0);
  drawWheel(ctx,-W2/2-2,-L/2+8,cop.steer*12);drawWheel(ctx,W2/2+2,-L/2+8,cop.steer*12);

  // Cop car body - dark blue/white
  ctx.fillStyle='#1a3a8c'; rrect(ctx,-W2/2,-L/2,W2,L,4); ctx.fill();
  // White stripe
  ctx.fillStyle='#ffffff'; ctx.fillRect(-W2/2+1,-L*.05,W2-2,L*.18);
  ctx.fillStyle='#1530a0'; rrect(ctx,-W2/2+2,-L/2+L*.22,W2-4,L*.42,3); ctx.fill();
  ctx.fillStyle='rgba(140,210,255,0.55)'; rrect(ctx,-W2/2+3.5,-L/2+L*.23,W2-7,L*.13,2); ctx.fill();
  ctx.fillStyle='rgba(140,210,255,0.45)'; rrect(ctx,-W2/2+3.5,-L/2+L*.58,W2-7,L*.09,2); ctx.fill();

  // Siren bar on top
  ctx.fillStyle='#222'; ctx.fillRect(-W2/2+2,-L/2+L*.22+2,W2-4,5);
  // Siren lights alternating
  ctx.fillStyle=sirenOn?'#ff2200':'#333';
  ctx.beginPath(); ctx.arc(-W2/2+6,-L/2+L*.22+4.5,3.5,0,Math.PI*2); ctx.fill();
  ctx.fillStyle=sirenOn?'#333':'#2244ff';
  ctx.beginPath(); ctx.arc(W2/2-6,-L/2+L*.22+4.5,3.5,0,Math.PI*2); ctx.fill();

  // Headlights
  ctx.fillStyle='rgba(255,255,200,0.95)'; ctx.fillRect(-W2/2+1.5,-L/2+2,5,4); ctx.fillRect(W2/2-6.5,-L/2+2,5,4);
  ctx.fillStyle='rgba(255,255,150,0.2)'; ctx.beginPath(); ctx.arc(0,-L/2-8,14,0,Math.PI*2); ctx.fill();
  ctx.fillStyle='#cc0000'; ctx.fillRect(-W2/2+1.5,L/2-6,5,4); ctx.fillRect(W2/2-6.5,L/2-6,5,4);

  // POLICE text
  ctx.save();
  ctx.fillStyle='rgba(255,255,255,0.9)';
  ctx.font='bold 6px Courier New';
  ctx.textAlign='center';
  ctx.fillText('POLICE',0,3);
  ctx.restore();

  ctx.strokeStyle='rgba(0,0,0,0.5)'; ctx.lineWidth=1; rrect(ctx,-W2/2,-L/2,W2,L,4); ctx.stroke();
  ctx.restore();
}

function drawWheel(c,ox,oy,steerDeg){
  c.save(); c.translate(ox,oy); c.rotate(steerDeg*Math.PI/180);
  c.fillStyle='#111'; c.fillRect(-4,-7,8,14);
  c.fillStyle='#888'; c.fillRect(-2,-5,4,10);
  c.fillStyle='#555'; c.fillRect(-0.5,-6,1,12);
  c.restore();
}

function drawMinimap(){
  mc.clearRect(0,0,130,130);
  mc.fillStyle='rgba(0,18,0,0.9)'; mc.fillRect(0,0,130,130);
  const s=130/WS,off=0;
  mc.fillStyle='rgba(90,158,66,0.35)'; mc.beginPath(); mc.arc(1200*s,1200*s,215*s,0,Math.PI*2); mc.fill();
  mc.strokeStyle='#3a3a3a'; mc.lineWidth=2;
  roads.forEach(r=>{mc.beginPath();mc.moveTo(r[0]*s,r[1]*s);mc.lineTo(r[2]*s,r[3]*s);mc.stroke();});

  // Cops on minimap (red dots)
  cops.forEach(cop=>{
    if(!cop.active)return;
    mc.fillStyle=Math.sin(cop.sirenPhase)>0?'#ff3333':'#3355ff';
    mc.beginPath(); mc.arc(cop.x*s,cop.y*s,4,0,Math.PI*2); mc.fill();
  });

  // Player
  mc.save(); mc.translate(player.x*s,player.y*s); mc.rotate(player.angle);
  mc.fillStyle='#FFB400'; mc.fillRect(-3,-5,6,10); mc.restore();
  mc.strokeStyle='rgba(255,180,0,0.3)'; mc.lineWidth=1; mc.strokeRect(0,0,130,130);
}

// ── Main loop ─────────────────────────────────────────────────────────────
let prev=performance.now(), started=false;
function loop(now){
  requestAnimationFrame(loop);
  const dt=(now-prev)/1000; prev=now;

  updatePlayer(dt);
  if(gameState==='playing'){
    cops.forEach(cop=>updateCop(cop,dt,player.x,player.y,player.angle));
    updateWanted(dt);
  }

  ctx.clearRect(0,0,gc.width,gc.height);
  drawSky();
  drawScene();
  drawMinimap();

  if(!started){started=true;document.getElementById('loading').style.display='none';}
}

// ── Init ──────────────────────────────────────────────────────────────────
wantedLevel=1;
initCops();
updateStars();
requestAnimationFrame(loop);
</script>

</body>
</html>