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
#speedo{position:absolute;bottom:28px;right:36px;background:rgba(0,0,0,.82);border:1px solid rgba(255,180,0,.5);border-radius:14px;padding:16px 24px;text-align:center}
#spd{font-size:50px;font-weight:900;color:#FFB400;line-height:1;letter-spacing:-2px;text-shadow:0 0 18px rgba(255,180,0,.7)}
#sunit{font-size:11px;color:rgba(255,180,0,.5);letter-spacing:4px;margin-top:3px}
#gearbox{position:absolute;bottom:28px;right:222px;background:rgba(0,0,0,.82);border:1px solid rgba(255,180,0,.3);border-radius:12px;padding:12px 18px;text-align:center}
#gval{font-size:40px;font-weight:900;color:#fff;line-height:1}
#glabel{font-size:10px;color:rgba(255,255,255,.4);letter-spacing:3px;margin-top:3px}
#rpmwrap{position:absolute;bottom:148px;right:36px;width:240px}
#rpmlbl{font-size:10px;color:rgba(255,180,0,.5);letter-spacing:4px;text-align:right;margin-bottom:5px}
#rpmbg{width:100%;height:7px;background:rgba(255,255,255,.08);border-radius:4px;overflow:hidden}
#rpmfill{height:100%;width:0%;border-radius:4px;transition:width .04s}
#hints{position:absolute;bottom:28px;left:36px;background:rgba(0,0,0,.75);border:1px solid rgba(255,255,255,.1);border-radius:12px;padding:12px 16px}
#hints div{font-size:11px;color:rgba(255,255,255,.45);line-height:1.9}
#hints b{color:#FFB400}
#mmwrap{position:absolute;top:18px;right:26px;width:150px;height:150px;background:rgba(0,0,0,.75);border:1px solid rgba(255,180,0,.25);border-radius:10px;overflow:hidden}
#banner{position:absolute;top:22px;left:50%;transform:translateX(-50%);font-size:12px;letter-spacing:8px;color:rgba(255,180,0,.8);font-weight:700;white-space:nowrap}
#scorebox{position:absolute;top:18px;left:36px;background:rgba(0,0,0,.82);border:1px solid rgba(255,180,0,.35);border-radius:12px;padding:12px 20px;min-width:160px}
#score-val{font-size:32px;font-weight:900;color:#FFB400;line-height:1;text-shadow:0 0 12px rgba(255,180,0,.5)}
#score-lbl{font-size:10px;color:rgba(255,180,0,.45);letter-spacing:4px;margin-bottom:4px}
#bestscore{font-size:11px;color:rgba(255,255,255,.3);margin-top:5px;letter-spacing:2px}
#wanted{position:absolute;top:110px;left:36px;background:rgba(0,0,0,.8);border:1px solid rgba(255,50,50,.4);border-radius:12px;padding:10px 16px}
#wlabel{font-size:10px;color:rgba(255,80,80,.7);letter-spacing:4px;margin-bottom:6px}
#stars{display:flex;gap:4px}
.star{font-size:17px;color:rgba(255,255,255,.12);transition:color .2s}
.star.on{color:#FFD700;text-shadow:0 0 8px #FFD700}
#copwarn{position:absolute;top:50%;left:50%;transform:translate(-50%,-50%) translateY(-60px);font-size:13px;letter-spacing:6px;color:#ff3333;font-weight:900;text-shadow:0 0 20px #ff0000;opacity:0;transition:opacity .15s;pointer-events:none;text-align:center}
#siren{display:none;position:fixed;top:0;left:0;width:100%;height:100%;pointer-events:none;z-index:4}
#busted{display:none;position:fixed;top:0;left:0;width:100%;height:100%;background:rgba(0,0,0,.88);z-index:50;flex-direction:column;align-items:center;justify-content:center}
#busted-title{font-size:90px;font-weight:900;color:#ff2200;letter-spacing:12px;text-shadow:0 0 40px #ff0000,0 0 80px #ff0000;animation:bust .45s infinite alternate}
#busted-sub{font-size:15px;letter-spacing:5px;color:rgba(255,255,255,.55);margin-top:14px}
#busted-score{font-size:24px;color:#FFB400;margin-top:18px;letter-spacing:3px}
#busted-best{font-size:14px;color:rgba(255,255,255,.35);margin-top:8px;letter-spacing:3px}
#busted-timer{font-size:22px;color:rgba(255,255,255,.5);margin-top:18px;letter-spacing:4px}
@keyframes bust{from{transform:scale(1) rotate(-1deg)}to{transform:scale(1.06) rotate(1deg)}}
#escape-wrap{position:absolute;bottom:90px;left:50%;transform:translateX(-50%);width:220px;opacity:0;transition:opacity .3s;pointer-events:none}
#escape-wrap.show{opacity:1}
#escape-lbl{font-size:10px;color:rgba(100,200,255,.7);letter-spacing:4px;text-align:center;margin-bottom:5px}
#escape-bg{width:100%;height:8px;background:rgba(255,255,255,.1);border-radius:4px;overflow:hidden}
#escape-fill{height:100%;width:0%;background:linear-gradient(90deg,#00aaff,#00ffcc);border-radius:4px;transition:width .1s}
#loading{position:fixed;top:0;left:0;width:100%;height:100%;background:#000;display:flex;flex-direction:column;align-items:center;justify-content:center;z-index:99;color:#FFB400}
#loading h1{font-size:34px;letter-spacing:10px;margin-bottom:16px}
#loading p{font-size:12px;letter-spacing:4px;color:rgba(255,180,0,.6);animation:bl 1s infinite}
@keyframes bl{0%,100%{opacity:1}50%{opacity:.3}}
</style>
</head>
<body>
<div id="loading"><h1>APEX CHASE</h1><p>BUILDING WORLD...</p></div>
<div id="siren"></div>
<canvas id="gc"></canvas>
<div id="hud">
  <div id="banner">APEX CHASE</div>
  <div id="mmwrap"><canvas id="mm" width="150" height="150"></canvas></div>
  <div id="rpmwrap"><div id="rpmlbl">RPM</div><div id="rpmbg"><div id="rpmfill"></div></div></div>
  <div id="gearbox"><div id="gval">N</div><div id="glabel">GEAR</div></div>
  <div id="speedo"><div id="spd">0</div><div id="sunit">KM/H</div></div>
  <div id="scorebox"><div id="score-lbl">SCORE</div><div id="score-val">0</div><div id="bestscore">BEST: 0</div></div>
  <div id="wanted"><div id="wlabel">WANTED</div><div id="stars"><span class="star" id="s0">★</span><span class="star" id="s1">★</span><span class="star" id="s2">★</span><span class="star" id="s3">★</span><span class="star" id="s4">★</span></div></div>
  <div id="copwarn">⚠ COP NEARBY ⚠</div>
  <div id="escape-wrap"><div id="escape-lbl">ESCAPING...</div><div id="escape-bg"><div id="escape-fill"></div></div></div>
  <div id="hints"><div><b>W/↑</b> Gas</div><div><b>S/↓</b> Brake/Rev</div><div><b>A D</b> Steer</div><div><b>SPACE</b> Handbrake</div><div><b>R</b> Reset pos</div></div>
</div>
<div id="busted">
  <div id="busted-title">BUSTED!</div>
  <div id="busted-sub">YOU WERE CAUGHT BY THE POLICE</div>
  <div id="busted-score"></div>
  <div id="busted-best"></div>
  <div id="busted-timer">RESTARTING IN <span id="btcount">3</span>...</div>
</div>

<script>
'use strict';
var gc=document.getElementById('gc'),ctx=gc.getContext('2d');
var mmC=document.getElementById('mm'),mc=mmC.getContext('2d');
function resize(){gc.width=window.innerWidth;gc.height=window.innerHeight;}
resize(); window.addEventListener('resize',resize);

var WS=4800,HS=4800;
function lerp(a,b,t){return a+(b-a)*t;}
function rnd(a,b){return a+Math.random()*(b-a);}
function ri(a,b){return Math.floor(rnd(a,b));}
function dist(ax,ay,bx,by){var dx=ax-bx,dy=ay-by;return Math.sqrt(dx*dx+dy*dy);}
function rrect(c,x,y,w,h,r){c.beginPath();c.moveTo(x+r,y);c.lineTo(x+w-r,y);c.arcTo(x+w,y,x+w,y+r,r);c.lineTo(x+w,y+h-r);c.arcTo(x+w,y+h,x+w-r,y+h,r);c.lineTo(x+r,y+h);c.arcTo(x,y+h,x,y+h-r,r);c.lineTo(x,y+r);c.arcTo(x,y,x+r,y,r);c.closePath();}
function shadeColor(hex,amt){var n=parseInt(hex.replace('#',''),16);return'rgb('+Math.min(255,Math.max(0,((n>>16)&255)+amt))+','+Math.min(255,Math.max(0,((n>>8)&255)+amt))+','+Math.min(255,Math.max(0,(n&255)+amt))+')';}
function preRand(x,y){return(((Math.sin(x*127.1+y*311.7)*43758.5453)%1)+1)%1;}
function angleDiff(a,b){var d=b-a;while(d>Math.PI)d-=Math.PI*2;while(d<-Math.PI)d+=Math.PI*2;return d;}

// ── Roads (4800x4800 world) ──────────────────────────────────────────────
// [x1,y1,x2,y2, halfWidth]
var roads=[
  // Outer ring
  [300,300,4500,300,60],[4500,300,4500,4500,60],[4500,4500,300,4500,60],[300,4500,300,300,60],
  // Second ring
  [900,900,3900,900,52],[3900,900,3900,3900,52],[3900,3900,900,3900,52],[900,3900,900,900,52],
  // Inner ring
  [1600,1600,3200,1600,48],[3200,1600,3200,3200,48],[3200,3200,1600,3200,48],[1600,3200,1600,1600,48],
  // Main cross
  [300,2400,4500,2400,52],[2400,300,2400,4500,52],
  // Corner diagonals
  [300,300,900,900,36],[4500,300,3900,900,36],[4500,4500,3900,3900,36],[300,4500,900,3900,36],
  // Spokes outer->second
  [900,900,1600,1600,36],[3900,900,3200,1600,36],[3900,3900,3200,3200,36],[900,3900,1600,3200,36],
  // Mid connectors
  [300,2400,900,2400,44],[4500,2400,3900,2400,44],[2400,300,2400,900,44],[2400,4500,2400,3900,44],
  [900,2400,1600,2400,40],[3900,2400,3200,2400,40],[2400,900,2400,1600,40],[2400,3900,2400,3200,40],
  // Inner cross
  [1600,2400,3200,2400,40],[2400,1600,2400,3200,40],
  // Downtown grid
  [2000,1600,2000,3200,34],[2800,1600,2800,3200,34],
  [1600,2000,3200,2000,34],[1600,2800,3200,2800,34],
  // Side streets
  [900,1600,1600,1600,32],[900,3200,1600,3200,32],[3200,1600,3900,1600,32],[3200,3200,3900,3200,32],
  [1600,900,1600,1600,32],[3200,900,3200,1600,32],[1600,3200,1600,3900,32],[3200,3200,3200,3900,32],
  // Suburbs top-left
  [300,900,900,900,36],[300,1600,900,1600,32],[300,3200,900,3200,32],
  [600,300,600,900,28],[600,900,600,1600,28],
  // Industrial top-right
  [3900,900,4500,900,36],[3900,1600,4500,1600,32],
  [4200,300,4200,900,28],[3600,300,3600,900,28],[4200,900,4200,1600,28],
  // Harbour bottom-left
  [300,3600,900,3600,36],[300,4200,900,4200,36],
  [600,3600,600,4200,28],[300,3900,900,3900,28],
  // SE district
  [3900,3200,4500,3200,32],[3900,3900,4500,3900,32],
  [4200,3200,4200,3900,28],[4200,3900,4200,4500,28],
  // Extra inner grid
  [2000,2400,2000,2000,30],[2800,2400,2800,2000,30],
  [2000,2400,2000,2800,30],[2800,2400,2800,2800,30],
];

var intersections=[
  [300,300],[4500,300],[4500,4500],[300,4500],
  [900,900],[3900,900],[3900,3900],[900,3900],
  [1600,1600],[3200,1600],[3200,3200],[1600,3200],
  [2400,2400],[300,2400],[4500,2400],[2400,300],[2400,4500],
  [900,2400],[3900,2400],[2400,900],[2400,3900],
  [1600,2400],[3200,2400],[2400,1600],[2400,3200],
  [2000,2000],[2800,2000],[2000,2800],[2800,2800],
  [600,900],[600,1600],[4200,900],[4200,1600],
  [600,3600],[600,4200],[300,3900],[900,3900],
  [4200,3200],[4200,3900],[3900,3200],[3900,3900],
];

// ── Buildings ─────────────────────────────────────────────────────────────
var bColors=['#8899aa','#99aabb','#7a8fa0','#b0c4d4','#6b8096','#d4c5a9','#c9b99a','#a0b0c0','#889977','#aabb88'];
var buildings=[];
var bRaw=[];
// Downtown (tall)
for(var i=0;i<40;i++) bRaw.push({x:rnd(1700,3100),y:rnd(1700,3100),big:true});
// NW suburb
for(var i=0;i<28;i++) bRaw.push({x:rnd(350,850),y:rnd(350,850),big:false});
// NE industrial
for(var i=0;i<28;i++) bRaw.push({x:rnd(3600,4400),y:rnd(350,850),big:false});
// SW harbour
for(var i=0;i<25;i++) bRaw.push({x:rnd(350,850),y:rnd(3600,4400),big:false});
// SE
for(var i=0;i<25;i++) bRaw.push({x:rnd(3600,4400),y:rnd(3600,4400),big:false});
// Mid ring blocks
for(var i=0;i<40;i++) bRaw.push({x:rnd(1000,1550),y:rnd(1000,1550),big:false});
for(var i=0;i<40;i++) bRaw.push({x:rnd(3250,3850),y:rnd(1000,1550),big:false});
for(var i=0;i<40;i++) bRaw.push({x:rnd(1000,1550),y:rnd(3250,3850),big:false});
for(var i=0;i<40;i++) bRaw.push({x:rnd(3250,3850),y:rnd(3250,3850),big:false});
// Outer strips
for(var i=0;i<30;i++) bRaw.push({x:rnd(1000,3800),y:rnd(360,760),big:false});
for(var i=0;i<30;i++) bRaw.push({x:rnd(1000,3800),y:rnd(4040,4440),big:false});
for(var i=0;i<30;i++) bRaw.push({x:rnd(360,760),y:rnd(1000,3800),big:false});
for(var i=0;i<30;i++) bRaw.push({x:rnd(4040,4440),y:rnd(1000,3800),big:false});

bRaw.forEach(function(p){
  var w=rnd(p.big?35:18,p.big?115:75);
  var h=rnd(p.big?35:18,p.big?115:75);
  var sh=rnd(8,26);
  var col=bColors[ri(0,bColors.length)];
  var wCols=Math.floor(w/18),wRows=Math.floor(h/20);
  var wins=[];
  for(var r=0;r<wRows;r++) for(var c=0;c<wCols;c++)
    wins.push(preRand(p.x+c*77,p.y+r*77)<0.65?(preRand(p.x+c*13,p.y+r*17)<0.35?'#ffffaa':'#253040'):null);
  buildings.push({x:p.x,y:p.y,w:w,h:h,col:col,sh:sh,wins:wins,wCols:wCols,wRows:wRows});
});

// ── Trees ─────────────────────────────────────────────────────────────────
var trees=[];
for(var i=0;i<700;i++) trees.push({x:rnd(80,WS-80),y:rnd(80,HS-80),r:rnd(10,22)});
for(var i=0;i<140;i++){var a=Math.random()*Math.PI*2,d=rnd(30,400);trees.push({x:2400+Math.cos(a)*d,y:2400+Math.sin(a)*d,r:rnd(10,21)});}
for(var v=500;v<4400;v+=200){trees.push({x:265,y:v,r:rnd(10,16)});trees.push({x:4535,y:v,r:rnd(10,16)});trees.push({x:v,y:265,r:rnd(10,16)});trees.push({x:v,y:4535,r:rnd(10,16)});}

// ── Parks & water ─────────────────────────────────────────────────────────
var parks=[
  {x:2400,y:2400,r:420,col:'#5a9e42',str:'#3a7a2a'},
  {x:600,y:600,r:180,col:'#5a9e42',str:'#3a7a2a'},
  {x:4200,y:600,r:160,col:'#5a9e42',str:'#3a7a2a'},
  {x:600,y:4200,r:160,col:'#5a9e42',str:'#3a7a2a'},
  {x:4200,y:4200,r:160,col:'#5a9e42',str:'#3a7a2a'},
  {x:1200,y:2400,r:120,col:'#5a9e42',str:'#3a7a2a'},
  {x:3600,y:2400,r:120,col:'#5a9e42',str:'#3a7a2a'},
  {x:2400,y:1200,r:120,col:'#5a9e42',str:'#3a7a2a'},
  {x:2400,y:3600,r:120,col:'#5a9e42',str:'#3a7a2a'},
];
var lakes=[
  {x:700,y:4100,rx:140,ry:90,rot:0.4},{x:4100,y:700,rx:130,ry:80,rot:0.2},
  {x:1100,y:1100,rx:100,ry:65,rot:0.7},{x:3700,y:3700,rx:120,ry:75,rot:0.3},
  {x:1100,y:3700,rx:95,ry:60,rot:1.1},{x:3700,y:1100,rx:95,ry:60,rot:0.9},
  {x:2400,y:700,rx:110,ry:70,rot:0.5},{x:2400,y:4100,rx:110,ry:70,rot:0.6},
];

// ── Street lights ─────────────────────────────────────────────────────────
var streetLights=[];
roads.forEach(function(r){
  var dx=r[2]-r[0],dy=r[3]-r[1],len=Math.sqrt(dx*dx+dy*dy);
  var steps=Math.floor(len/280);
  for(var i=1;i<steps;i++) streetLights.push({x:r[0]+dx*(i/steps),y:r[1]+dy*(i/steps)});
});

// ── Cop spawns (spread across entire map) ─────────────────────────────────
var COP_SPAWNS=[
  [500,500],[2400,400],[4300,500],[4400,2400],[4300,4300],[2400,4400],[500,4300],[400,2400],
  [1200,1200],[3600,1200],[3600,3600],[1200,3600],
  [1200,2400],[3600,2400],[2400,1200],[2400,3600],
  [800,2400],[4000,2400],[2400,800],[2400,4000],
];

// ── Physics ───────────────────────────────────────────────────────────────
var PCFG={maxSpeed:8.2,acc:0.20,brk:0.30,rev:0.08,steerMax:0.050,steerSpeed:0.16,steerReturn:0.13,drag:0.976};
var CCFG={maxSpeed:6.8,acc:0.13,steerMax:0.044,drag:0.970};

var player={x:2400,y:2600,vx:0,vy:0,angle:0,steer:0};
var keys={};
window.addEventListener('keydown',function(e){keys[e.code]=true;if(['Space','ArrowUp','ArrowDown','ArrowLeft','ArrowRight'].indexOf(e.code)>=0)e.preventDefault();});
window.addEventListener('keyup',function(e){keys[e.code]=false;});
var cam={x:2400,y:2600,sx:2400,sy:2600,scale:1.0};

var cops=[],wantedLevel=1,wantedTimer=0;
var score=0,bestScore=0,scoreTimer=0,escapeTimer=0;
var gameState='playing',sirenFlip=0;

// ── Cop factory ───────────────────────────────────────────────────────────
function makeCop(spawnIdx){
  var sp=COP_SPAWNS[spawnIdx%COP_SPAWNS.length];
  return{
    x:sp[0],y:sp[1],vx:0,vy:0,angle:Math.random()*Math.PI*2,steer:0,
    predictBias:rnd(0.15,0.80),   // imperfect prediction
    reactionDelay:rnd(0.08,0.38), // input lag
    steerNoise:rnd(0.0,0.018),    // steering wobble
    speedFactor:rnd(0.80,0.96),   // individual speed
    steerSmooth:0,
    sirenPhase:Math.random()*Math.PI*2,
    active:true
  };
}

function initCops(){
  cops=[];
  // Pick a spawn point far from player
  var best=0,bestD=0;
  COP_SPAWNS.forEach(function(sp,i){var d=dist(sp[0],sp[1],player.x,player.y);if(d>bestD){bestD=d;best=i;}});
  cops.push(makeCop(best));
}

function updateStars(){for(var i=0;i<5;i++)document.getElementById('s'+i).className='star'+(i<wantedLevel?' on':'');}

function resetGame(){
  player={x:2400,y:2600,vx:0,vy:0,angle:0,steer:0};
  cam={x:2400,y:2600,sx:2400,sy:2600,scale:1.0};
  if(score>bestScore){bestScore=score;document.getElementById('bestscore').textContent='BEST: '+Math.floor(bestScore);}
  score=0;wantedLevel=1;wantedTimer=0;scoreTimer=0;escapeTimer=0;gameState='playing';
  document.getElementById('busted').style.display='none';
  document.getElementById('siren').style.display='none';
  document.getElementById('score-val').textContent='0';
  updateStars();initCops();
}

// ── Cop AI ────────────────────────────────────────────────────────────────
function updateCop(cop,dt){
  if(!cop.active)return;
  var px=player.x,py=player.y;

  // Imperfect intercept prediction
  var predTime=cop.predictBias*1.4;
  var tx=px+player.vx*predTime*20;
  var ty=py+player.vy*predTime*20;

  var desired=Math.atan2(tx-cop.x,-(ty-cop.y));
  var dAngle=angleDiff(cop.angle,desired);
  dAngle+=(Math.random()-0.5)*cop.steerNoise*2.5;

  var steerTarget=Math.max(-CCFG.steerMax,Math.min(CCFG.steerMax,dAngle*0.62));
  cop.steerSmooth=lerp(cop.steerSmooth,steerTarget,(0.10+cop.reactionDelay)*(dt*60));

  var copSpd=Math.sqrt(cop.vx*cop.vx+cop.vy*cop.vy);
  if(copSpd>0.05)cop.angle+=cop.steerSmooth;

  var fx=Math.sin(cop.angle),fy=-Math.cos(cop.angle);
  var lx=Math.cos(cop.angle),ly=Math.sin(cop.angle);
  var fwdV=cop.vx*fx+cop.vy*fy;
  var latV=cop.vx*lx+cop.vy*ly;

  var dToPlayer=dist(cop.x,cop.y,px,py);
  var topSpd=CCFG.maxSpeed*cop.speedFactor;
  if(dToPlayer>40) fwdV=Math.min(fwdV+CCFG.acc,topSpd);
  else fwdV*=Math.pow(0.90,dt*60);

  // Less grip than player = slides more
  latV*=Math.pow(0.70,dt*60);

  cop.vx=fx*fwdV+lx*latV;
  cop.vy=fy*fwdV+ly*latV;
  cop.x=Math.max(80,Math.min(WS-80,cop.x+cop.vx));
  cop.y=Math.max(80,Math.min(HS-80,cop.y+cop.vy));
  cop.sirenPhase+=dt*9;
}

function triggerBusted(){
  if(gameState==='busted')return;
  gameState='busted';
  if(score>bestScore)bestScore=score;
  document.getElementById('busted').style.display='flex';
  document.getElementById('busted-score').textContent='SCORE: '+Math.floor(score);
  document.getElementById('busted-best').textContent='BEST: '+Math.floor(bestScore);
  document.getElementById('bestscore').textContent='BEST: '+Math.floor(bestScore);
  var cd=3;document.getElementById('btcount').textContent=cd;
  var iv=setInterval(function(){cd--;document.getElementById('btcount').textContent=Math.max(0,cd);if(cd<=0){clearInterval(iv);resetGame();}},1000);
}

// ── Player physics ────────────────────────────────────────────────────────
function updatePlayer(dt){
  if(gameState!=='playing')return;
  dt=Math.min(dt,0.05);
  var thr=(keys['KeyW']||keys['ArrowUp'])?1:0;
  var brk=(keys['KeyS']||keys['ArrowDown'])?1:0;
  var sl=(keys['KeyA']||keys['ArrowLeft'])?1:0;
  var sr=(keys['KeyD']||keys['ArrowRight'])?1:0;
  var hb=keys['Space']?1:0;
  if(keys['KeyR']){player.x=2400;player.y=2600;player.vx=0;player.vy=0;player.angle=0;player.steer=0;}

  var spd=Math.sqrt(player.vx*player.vx+player.vy*player.vy);
  var fx=Math.sin(player.angle),fy=-Math.cos(player.angle);
  var lx=Math.cos(player.angle),ly=Math.sin(player.angle);
  var fwdV=player.vx*fx+player.vy*fy;
  var latV=player.vx*lx+player.vy*ly;

  var tSt=(sl-sr)*PCFG.steerMax*Math.min(1,spd/1.5);
  player.steer+=(tSt-player.steer)*PCFG.steerSpeed*(dt*60);
  player.steer+=(0-player.steer)*PCFG.steerReturn*(dt*60)*(1-Math.abs(sl-sr));
  if(spd>0.05)player.angle+=player.steer*(fwdV>=0?1:-1);

  if(thr) fwdV=Math.min(fwdV+PCFG.acc,PCFG.maxSpeed);
  else if(brk){if(fwdV>0.1)fwdV=Math.max(0,fwdV-PCFG.brk);else fwdV=Math.max(-PCFG.rev*PCFG.maxSpeed,fwdV-PCFG.rev);}
  else fwdV*=Math.pow(PCFG.drag,dt*60);

  latV*=Math.pow(hb?0.97:0.15,dt*60);
  player.vx=fx*fwdV+lx*latV;
  player.vy=fy*fwdV+ly*latV;
  player.x=Math.max(80,Math.min(WS-80,player.x+player.vx));
  player.y=Math.max(80,Math.min(HS-80,player.y+player.vy));

  var lookX=player.x+Math.sin(player.angle)*100;
  var lookY=player.y-Math.cos(player.angle)*100;
  cam.sx=lerp(cam.sx,lookX,7*dt);cam.sy=lerp(cam.sy,lookY,7*dt);
  cam.x=cam.sx;cam.y=cam.sy;
  cam.scale=lerp(cam.scale,lerp(0.90,0.50,spd/PCFG.maxSpeed),2.5*dt);

  scoreTimer+=dt;
  if(scoreTimer>=0.5){scoreTimer=0;score+=wantedLevel*10;document.getElementById('score-val').textContent=Math.floor(score);}

  var kmh=Math.abs(fwdV)*30;
  document.getElementById('spd').textContent=Math.round(kmh);
  var rPct=Math.min(spd/PCFG.maxSpeed,1);
  var rf=document.getElementById('rpmfill');
  rf.style.width=(rPct*100)+'%';
  rf.style.background=rPct>.85?'linear-gradient(90deg,#FF4400,#FF0000)':rPct>.60?'linear-gradient(90deg,#FFB400,#FF6600)':'linear-gradient(90deg,#FFB400,#FF4400)';
  var gear='N';
  if(fwdV<-0.1)gear='R';else if(spd<0.1)gear='N';else if(kmh<25)gear=1;else if(kmh<55)gear=2;else if(kmh<90)gear=3;else if(kmh<130)gear=4;else gear=5;
  document.getElementById('gval').textContent=gear;
}

// ── Game logic ────────────────────────────────────────────────────────────
function updateGame(dt){
  if(gameState!=='playing')return;
  wantedTimer+=dt;sirenFlip+=dt;

  // Escalate every 20s
  if(wantedLevel<5&&wantedTimer>20){
    wantedLevel=Math.min(5,wantedLevel+1);
    wantedTimer=0;updateStars();
    // Spawn new cop from direction opposite to player heading
    var oppositeAngle=player.angle+Math.PI;
    var spawnDist=rnd(900,1500);
    var spawnX=player.x+Math.sin(oppositeAngle)*spawnDist;
    var spawnY=player.y-Math.cos(oppositeAngle)*spawnDist;
    // Find closest spawn point to that position
    var best=0,bestD=999999;
    COP_SPAWNS.forEach(function(sp,i){
      var d=dist(sp[0],sp[1],spawnX,spawnY);
      // Must not be too close to player
      var dToPlayer=dist(sp[0],sp[1],player.x,player.y);
      if(d<bestD&&dToPlayer>500){bestD=d;best=i;}
    });
    cops.push(makeCop(best));
  }

  // Respawn cops that fall too far behind
  cops.forEach(function(cop){
    if(!cop.active)return;
    var d=dist(cop.x,cop.y,player.x,player.y);
    if(d>1900){
      // Pick spawn from a different direction to player
      var angles=[];
      COP_SPAWNS.forEach(function(sp,i){
        var dToPlayer=dist(sp[0],sp[1],player.x,player.y);
        if(dToPlayer>500&&dToPlayer<1600)angles.push(i);
      });
      if(angles.length>0){
        var sp=COP_SPAWNS[angles[ri(0,angles.length)]];
        cop.x=sp[0];cop.y=sp[1];cop.vx=0;cop.vy=0;
        cop.predictBias=rnd(0.15,0.82);
        cop.reactionDelay=rnd(0.06,0.40);
        cop.steerNoise=rnd(0.0,0.020);
        cop.speedFactor=rnd(0.78,0.95);
      }
    }
  });

  // Catch check
  var closestD=Infinity;
  cops.forEach(function(cop){
    if(!cop.active)return;
    var d=dist(cop.x,cop.y,player.x,player.y);
    if(d<closestD)closestD=d;
    if(d<30)triggerBusted();
  });

  // Escape bar
  var ew=document.getElementById('escape-wrap');
  var ef=document.getElementById('escape-fill');
  if(closestD<350){
    escapeTimer=0;ew.classList.remove('show');
  } else if(closestD<900){
    escapeTimer+=dt;ew.classList.add('show');
    ef.style.width=Math.min(100,(escapeTimer/5)*100)+'%';
    if(escapeTimer>=5){
      escapeTimer=0;score+=wantedLevel*50;
      document.getElementById('score-val').textContent=Math.floor(score);
    }
  } else {
    escapeTimer=0;ew.classList.remove('show');
  }

  // Siren flash
  var warn=document.getElementById('copwarn');
  var siren=document.getElementById('siren');
  if(closestD<320){
    warn.style.opacity='1';
    siren.style.display='block';
    siren.style.background=sirenFlip%0.18<0.09?'rgba(255,40,40,0.07)':'rgba(40,40,255,0.07)';
  } else {
    warn.style.opacity='0';
    siren.style.display='none';
  }
}

// ── Draw ──────────────────────────────────────────────────────────────────
function drawSky(){
  var g=ctx.createLinearGradient(0,0,0,gc.height*0.38);
  g.addColorStop(0,'#2e6ea6');g.addColorStop(1,'#7dc4e8');
  ctx.fillStyle=g;ctx.fillRect(0,0,gc.width,gc.height*0.38);
}

function drawScene(){
  var offX=gc.width/2-cam.x*cam.scale;
  var offY=gc.height/2-cam.y*cam.scale;
  ctx.save();ctx.translate(offX,offY);ctx.scale(cam.scale,cam.scale);

  ctx.fillStyle='#48883c';ctx.fillRect(0,0,WS,HS);
  ctx.strokeStyle='#1a1a1a';ctx.lineWidth=10;ctx.strokeRect(280,280,WS-560,HS-560);

  // Parks
  parks.forEach(function(p){
    ctx.fillStyle=p.col;ctx.beginPath();ctx.arc(p.x,p.y,p.r,0,Math.PI*2);ctx.fill();
    ctx.strokeStyle=p.str;ctx.lineWidth=5;ctx.stroke();
  });
  ctx.strokeStyle='#d4c5a9';ctx.lineWidth=9;ctx.setLineDash([25,18]);
  parks.forEach(function(p){ctx.beginPath();ctx.arc(p.x,p.y,p.r*0.6,0,Math.PI*2);ctx.stroke();});
  ctx.setLineDash([]);

  // Lakes
  lakes.forEach(function(l){
    ctx.fillStyle='#3a7fbf';ctx.beginPath();ctx.ellipse(l.x,l.y,l.rx,l.ry,l.rot,0,Math.PI*2);ctx.fill();
    ctx.fillStyle='#5a9ac0';ctx.beginPath();ctx.ellipse(l.x,l.y,l.rx*.6,l.ry*.6,l.rot,0,Math.PI*2);ctx.fill();
    ctx.strokeStyle='#2a6a9a';ctx.lineWidth=3;ctx.beginPath();ctx.ellipse(l.x,l.y,l.rx,l.ry,l.rot,0,Math.PI*2);ctx.stroke();
  });

  // Road shadows
  ctx.lineCap='round';
  roads.forEach(function(r){ctx.strokeStyle='rgba(0,0,0,0.16)';ctx.lineWidth=r[4]*2+12;ctx.beginPath();ctx.moveTo(r[0],r[1]);ctx.lineTo(r[2],r[3]);ctx.stroke();});

  // Roads
  ctx.lineCap='square';
  roads.forEach(function(r){
    var rw=r[4]*2;
    ctx.strokeStyle='#2c2c2c';ctx.lineWidth=rw;ctx.beginPath();ctx.moveTo(r[0],r[1]);ctx.lineTo(r[2],r[3]);ctx.stroke();
    var dx=r[2]-r[0],dy=r[3]-r[1],len=Math.sqrt(dx*dx+dy*dy);
    var nx=-dy/len,ny=dx/len,off=r[4]-4;
    ctx.strokeStyle='rgba(200,200,200,0.20)';ctx.lineWidth=2;
    ctx.beginPath();ctx.moveTo(r[0]+nx*off,r[1]+ny*off);ctx.lineTo(r[2]+nx*off,r[3]+ny*off);ctx.stroke();
    ctx.beginPath();ctx.moveTo(r[0]-nx*off,r[1]-ny*off);ctx.lineTo(r[2]-nx*off,r[3]-ny*off);ctx.stroke();
    ctx.strokeStyle='#fff';ctx.lineWidth=2.5;ctx.setLineDash([30,24]);
    ctx.beginPath();ctx.moveTo(r[0],r[1]);ctx.lineTo(r[2],r[3]);ctx.stroke();ctx.setLineDash([]);
  });

  // Intersection pads
  intersections.forEach(function(p){ctx.fillStyle='#282828';ctx.fillRect(p[0]-44,p[1]-44,88,88);});

  // Trees
  trees.forEach(function(t){
    ctx.fillStyle='rgba(0,0,0,0.13)';ctx.beginPath();ctx.ellipse(t.x+5,t.y+5,t.r,t.r*.65,0,0,Math.PI*2);ctx.fill();
    ctx.fillStyle='#5c3d1e';ctx.fillRect(t.x-3,t.y-3,6,6);
    ctx.fillStyle='#2d6a1f';ctx.beginPath();ctx.arc(t.x,t.y,t.r,0,Math.PI*2);ctx.fill();
    ctx.fillStyle='#3d8a2a';ctx.beginPath();ctx.arc(t.x-t.r*.3,t.y-t.r*.3,t.r*.5,0,Math.PI*2);ctx.fill();
  });

  // Buildings
  buildings.forEach(function(b){
    ctx.fillStyle='rgba(0,0,0,0.18)';ctx.fillRect(b.x+b.sh,b.y+b.sh,b.w,b.h);
    ctx.fillStyle=shadeColor(b.col,-44);
    ctx.beginPath();ctx.moveTo(b.x+b.w,b.y);ctx.lineTo(b.x+b.w+b.sh,b.y-b.sh);ctx.lineTo(b.x+b.w+b.sh,b.y+b.h-b.sh);ctx.lineTo(b.x+b.w,b.y+b.h);ctx.fill();
    ctx.fillStyle=shadeColor(b.col,24);
    ctx.beginPath();ctx.moveTo(b.x,b.y);ctx.lineTo(b.x+b.sh,b.y-b.sh);ctx.lineTo(b.x+b.w+b.sh,b.y-b.sh);ctx.lineTo(b.x+b.w,b.y);ctx.fill();
    ctx.fillStyle=b.col;ctx.fillRect(b.x,b.y,b.w,b.h);
    var wi=0;
    for(var r=0;r<b.wRows;r++) for(var c=0;c<b.wCols;c++){var wc=b.wins[wi++];if(wc){ctx.fillStyle=wc;ctx.fillRect(b.x+5+c*18,b.y+5+r*20,10,13);}}
    ctx.strokeStyle='rgba(0,0,0,.3)';ctx.lineWidth=1;ctx.strokeRect(b.x,b.y,b.w,b.h);
  });

  // Street lights
  streetLights.forEach(function(l){
    ctx.strokeStyle='#777';ctx.lineWidth=3;ctx.beginPath();ctx.moveTo(l.x,l.y+18);ctx.lineTo(l.x,l.y);ctx.stroke();
    ctx.fillStyle='rgba(255,255,150,.16)';ctx.beginPath();ctx.arc(l.x,l.y,26,0,Math.PI*2);ctx.fill();
    ctx.fillStyle='rgba(255,255,150,.9)';ctx.beginPath();ctx.arc(l.x,l.y,5,0,Math.PI*2);ctx.fill();
  });

  // Cops
  cops.forEach(function(c){if(c.active)drawCopCar(c);});
  // Player
  drawPlayerCar();
  ctx.restore();
}

function drawPlayerCar(){
  ctx.save();ctx.translate(player.x,player.y);ctx.rotate(player.angle);
  var L=40,W2=19,brk=keys['KeyS']||keys['ArrowDown'];
  ctx.fillStyle='rgba(0,0,0,.2)';ctx.beginPath();ctx.ellipse(4,4,W2+2,L*.35,0,0,Math.PI*2);ctx.fill();
  drawWheel(ctx,-W2/2-2,L/2-9,0);drawWheel(ctx,W2/2+2,L/2-9,0);
  drawWheel(ctx,-W2/2-2,-L/2+9,player.steer*12);drawWheel(ctx,W2/2+2,-L/2+9,player.steer*12);
  ctx.fillStyle='#cc2211';rrect(ctx,-W2/2,-L/2,W2,L,4);ctx.fill();
  ctx.fillStyle='#b51d10';rrect(ctx,-W2/2+2,-L/2+2,W2-4,L*.38,3);ctx.fill();
  ctx.fillStyle='#aa1a0e';rrect(ctx,-W2/2+2,-L/2+L*.22,W2-4,L*.42,3);ctx.fill();
  ctx.fillStyle='rgba(140,210,255,.55)';rrect(ctx,-W2/2+3.5,-L/2+L*.23,W2-7,L*.13,2);ctx.fill();
  ctx.fillStyle='rgba(140,210,255,.45)';rrect(ctx,-W2/2+3.5,-L/2+L*.58,W2-7,L*.09,2);ctx.fill();
  ctx.fillStyle='rgba(255,255,200,.95)';ctx.fillRect(-W2/2+1.5,-L/2+2,5,4);ctx.fillRect(W2/2-6.5,-L/2+2,5,4);
  ctx.fillStyle='rgba(255,255,150,.18)';ctx.beginPath();ctx.arc(-W2/2+4,-L/2+3,11,0,Math.PI*2);ctx.fill();ctx.beginPath();ctx.arc(W2/2-4,-L/2+3,11,0,Math.PI*2);ctx.fill();
  ctx.fillStyle=brk?'#ff5500':'#cc0000';ctx.fillRect(-W2/2+1.5,L/2-7,5,4);ctx.fillRect(W2/2-6.5,L/2-7,5,4);
  ctx.fillStyle='#111';ctx.fillRect(-W2/2+1,L/2-4,W2-2,3);
  ctx.strokeStyle='rgba(0,0,0,.5)';ctx.lineWidth=1;rrect(ctx,-W2/2,-L/2,W2,L,4);ctx.stroke();
  ctx.restore();
}

function drawCopCar(cop){
  ctx.save();ctx.translate(cop.x,cop.y);ctx.rotate(cop.angle);
  var L=40,W2=19,sirenOn=Math.sin(cop.sirenPhase)>0;
  ctx.fillStyle=sirenOn?'rgba(0,60,255,.14)':'rgba(255,40,40,.14)';
  ctx.beginPath();ctx.arc(0,0,48,0,Math.PI*2);ctx.fill();
  ctx.fillStyle='rgba(0,0,0,.2)';ctx.beginPath();ctx.ellipse(4,4,W2+2,L*.35,0,0,Math.PI*2);ctx.fill();
  drawWheel(ctx,-W2/2-2,L/2-9,0);drawWheel(ctx,W2/2+2,L/2-9,0);
  drawWheel(ctx,-W2/2-2,-L/2+9,cop.steerSmooth*12);drawWheel(ctx,W2/2+2,-L/2+9,cop.steerSmooth*12);
  ctx.fillStyle='#1a3a8c';rrect(ctx,-W2/2,-L/2,W2,L,4);ctx.fill();
  ctx.fillStyle='#ffffff';ctx.fillRect(-W2/2+1,-4,W2-2,9);
  ctx.fillStyle='#1530a0';rrect(ctx,-W2/2+2,-L/2+L*.22,W2-4,L*.42,3);ctx.fill();
  ctx.fillStyle='rgba(140,210,255,.5)';rrect(ctx,-W2/2+3.5,-L/2+L*.23,W2-7,L*.13,2);ctx.fill();
  ctx.fillStyle='#111';ctx.fillRect(-W2/2+2,-L/2+L*.22+1,W2-4,6);
  ctx.fillStyle=sirenOn?'#ff2200':'#330000';ctx.beginPath();ctx.arc(-W2/2+7,-L/2+L*.22+4,3.5,0,Math.PI*2);ctx.fill();
  ctx.fillStyle=sirenOn?'#330055':'#2244ff';ctx.beginPath();ctx.arc(W2/2-7,-L/2+L*.22+4,3.5,0,Math.PI*2);ctx.fill();
  ctx.fillStyle='rgba(255,255,150,.2)';ctx.beginPath();ctx.arc(0,-L/2-11,16,0,Math.PI*2);ctx.fill();
  ctx.fillStyle='rgba(255,255,200,.95)';ctx.fillRect(-W2/2+1.5,-L/2+2,5,4);ctx.fillRect(W2/2-6.5,-L/2+2,5,4);
  ctx.fillStyle='#cc0000';ctx.fillRect(-W2/2+1.5,L/2-7,5,4);ctx.fillRect(W2/2-6.5,L/2-7,5,4);
  ctx.fillStyle='rgba(255,255,255,.85)';ctx.font='bold 6px Courier New';ctx.textAlign='center';ctx.fillText('POLICE',0,4);
  ctx.strokeStyle='rgba(0,0,0,.5)';ctx.lineWidth=1;rrect(ctx,-W2/2,-L/2,W2,L,4);ctx.stroke();
  ctx.restore();
}

function drawWheel(c,ox,oy,steerDeg){
  c.save();c.translate(ox,oy);c.rotate(steerDeg*Math.PI/180);
  c.fillStyle='#111';c.fillRect(-4,-7,8,14);
  c.fillStyle='#888';c.fillRect(-2,-5,4,10);
  c.restore();
}

function drawMinimap(){
  var S=150;mc.clearRect(0,0,S,S);
  mc.fillStyle='rgba(0,18,0,.92)';mc.fillRect(0,0,S,S);
  var s=S/WS;
  parks.forEach(function(p){mc.fillStyle='rgba(90,158,66,.28)';mc.beginPath();mc.arc(p.x*s,p.y*s,p.r*s,0,Math.PI*2);mc.fill();});
  mc.strokeStyle='#3a3a3a';mc.lineWidth=1.2;
  roads.forEach(function(r){mc.beginPath();mc.moveTo(r[0]*s,r[1]*s);mc.lineTo(r[2]*s,r[3]*s);mc.stroke();});
  cops.forEach(function(cop){
    if(!cop.active)return;
    mc.fillStyle=Math.sin(cop.sirenPhase)>0?'#ff3333':'#3355ff';
    mc.beginPath();mc.arc(cop.x*s,cop.y*s,3.5,0,Math.PI*2);mc.fill();
  });
  mc.save();mc.translate(player.x*s,player.y*s);mc.rotate(player.angle);
  mc.fillStyle='#FFB400';mc.fillRect(-3,-5,6,10);mc.restore();
  mc.strokeStyle='rgba(255,180,0,.3)';mc.lineWidth=1;mc.strokeRect(0,0,S,S);
}

// ── Loop ──────────────────────────────────────────────────────────────────
var prev=performance.now(),started=false;
function loop(now){
  requestAnimationFrame(loop);
  var dt=(now-prev)/1000;prev=now;
  updatePlayer(dt);
  if(gameState==='playing'){cops.forEach(function(c){updateCop(c,dt);});updateGame(dt);}
  ctx.clearRect(0,0,gc.width,gc.height);
  drawSky();drawScene();drawMinimap();
  if(!started){started=true;document.getElementById('loading').style.display='none';}
}
wantedLevel=1;initCops();updateStars();
requestAnimationFrame(loop);
</script>

</body>
</html>