<!DOCTYPE html>

<html lang="en">
<head>
<meta charset="UTF-8">
<title>APEX DRIVE 3D</title>
<style>
*{margin:0;padding:0;box-sizing:border-box}
body{background:#000;overflow:hidden;font-family:'Courier New',monospace}
#c{display:block;width:100vw;height:100vh}
#hud{position:fixed;top:0;left:0;width:100%;height:100%;pointer-events:none;z-index:5}
#speedo{position:absolute;bottom:28px;right:36px;background:rgba(0,0,0,.8);border:1px solid rgba(255,180,0,.5);border-radius:14px;padding:18px 26px;text-align:center}
#spd{font-size:54px;font-weight:900;color:#FFB400;line-height:1;letter-spacing:-2px;text-shadow:0 0 20px rgba(255,180,0,.8)}
#sunit{font-size:12px;color:rgba(255,180,0,.55);letter-spacing:4px;margin-top:3px}
#gearbox{position:absolute;bottom:28px;right:230px;background:rgba(0,0,0,.8);border:1px solid rgba(255,180,0,.3);border-radius:12px;padding:14px 20px;text-align:center}
#gval{font-size:42px;font-weight:900;color:#fff;line-height:1}
#glabel{font-size:10px;color:rgba(255,255,255,.4);letter-spacing:3px;margin-top:3px}
#rpmwrap{position:absolute;bottom:150px;right:36px;width:250px}
#rpmlbl{font-size:10px;color:rgba(255,180,0,.5);letter-spacing:4px;text-align:right;margin-bottom:5px}
#rpmbg{width:100%;height:7px;background:rgba(255,255,255,.08);border-radius:4px;overflow:hidden}
#rpmfill{height:100%;width:0%;border-radius:4px;transition:width .04s}
#hints{position:absolute;bottom:28px;left:36px;background:rgba(0,0,0,.75);border:1px solid rgba(255,255,255,.1);border-radius:12px;padding:14px 18px}
#hints div{font-size:12px;color:rgba(255,255,255,.5);line-height:2}
#hints b{color:#FFB400}
#mmwrap{position:absolute;top:18px;right:26px;width:130px;height:130px;background:rgba(0,0,0,.75);border:1px solid rgba(255,180,0,.25);border-radius:10px;overflow:hidden}
#banner{position:absolute;top:22px;left:50%;transform:translateX(-50%);font-size:12px;letter-spacing:8px;color:rgba(255,180,0,.8);font-weight:700;white-space:nowrap;text-shadow:0 0 10px rgba(255,180,0,.4)}
#err{display:none;position:fixed;top:0;left:0;width:100%;height:100%;background:#000;color:#FFB400;font-family:Courier New;display:flex;flex-direction:column;align-items:center;justify-content:center;text-align:center;padding:40px;z-index:99}
#loading{position:fixed;top:0;left:0;width:100%;height:100%;background:#000;display:flex;flex-direction:column;align-items:center;justify-content:center;z-index:98;color:#FFB400;font-family:'Courier New',monospace}
#loading h1{font-size:36px;letter-spacing:10px;margin-bottom:20px}
#loading p{font-size:13px;letter-spacing:4px;color:rgba(255,180,0,.6);animation:blink 1s infinite}
@keyframes blink{0%,100%{opacity:1}50%{opacity:.3}}
</style>
</head>
<body>
<div id="loading"><h1>APEX DRIVE</h1><p>INITIALIZING 3D ENGINE...</p></div>
<canvas id="c"></canvas>
<div id="hud">
  <div id="banner">APEX DRIVE 3D — FREE ROAM</div>
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
'use strict';
// ─── WebGL Setup ─────────────────────────────────────────────────────────
const canvas = document.getElementById('c');
let gl;
try {
  gl = canvas.getContext('webgl', {antialias:true, alpha:false}) ||
       canvas.getContext('experimental-webgl', {antialias:true, alpha:false});
} catch(e){}
if (!gl) {
  document.getElementById('loading').style.display='none';
  const e=document.getElementById('err');
  e.style.display='flex';
  e.innerHTML='<h2>WebGL Not Available</h2><p style="margin-top:16px;opacity:.6">Please enable hardware acceleration in Chrome settings:<br>chrome://settings/system → Use hardware acceleration</p>';
}

// ─── Resize ───────────────────────────────────────────────────────────────
function resize() {
  canvas.width  = window.innerWidth;
  canvas.height = window.innerHeight;
  gl.viewport(0, 0, canvas.width, canvas.height);
}
resize();
window.addEventListener('resize', resize);

// ─── Shaders ──────────────────────────────────────────────────────────────
const VS = `
attribute vec3 aPos;
attribute vec3 aNorm;
attribute vec3 aCol;
uniform mat4 uMVP;
uniform mat4 uModel;
uniform vec3 uSunDir;
uniform vec3 uBaseColor;
uniform float uUseVertColor;
varying vec3 vColor;
varying float vLight;
void main(){
  vec3 worldNorm = normalize(mat3(uModel) * aNorm);
  float diff = max(dot(worldNorm, normalize(uSunDir)), 0.0);
  vLight = 0.38 + diff * 0.62;
  vColor = uUseVertColor > 0.5 ? aCol : uBaseColor;
  gl_Position = uMVP * vec4(aPos, 1.0);
}`;

const FS = `
precision mediump float;
varying vec3 vColor;
varying float vLight;
void main(){
  gl_FragColor = vec4(vColor * vLight, 1.0);
}`;

function mkShader(type, src) {
  const s = gl.createShader(type);
  gl.shaderSource(s, src); gl.compileShader(s);
  return s;
}
const prog = gl.createProgram();
gl.attachShader(prog, mkShader(gl.VERTEX_SHADER, VS));
gl.attachShader(prog, mkShader(gl.FRAGMENT_SHADER, FS));
gl.linkProgram(prog); gl.useProgram(prog);

const aPos  = gl.getAttribLocation(prog,'aPos');
const aNorm = gl.getAttribLocation(prog,'aNorm');
const aCol  = gl.getAttribLocation(prog,'aCol');
const uMVP  = gl.getUniformLocation(prog,'uMVP');
const uModel= gl.getUniformLocation(prog,'uModel');
const uSun  = gl.getUniformLocation(prog,'uSunDir');
const uBase = gl.getUniformLocation(prog,'uBaseColor');
const uUseV = gl.getUniformLocation(prog,'uUseVertColor');

gl.enable(gl.DEPTH_TEST); gl.enable(gl.CULL_FACE);
gl.clearColor(0.53, 0.81, 0.92, 1.0);

// ─── Math helpers ─────────────────────────────────────────────────────────
function mat4(){return new Float32Array(16)}
function identity(m){m[0]=m[5]=m[10]=m[15]=1;m[1]=m[2]=m[3]=m[4]=m[6]=m[7]=m[8]=m[9]=m[11]=m[12]=m[13]=m[14]=0;return m}

function perspective(m,fov,asp,near,far){
  const f=1/Math.tan(fov/2),d=near-far;
  m[0]=f/asp;m[1]=0;m[2]=0;m[3]=0;
  m[4]=0;m[5]=f;m[6]=0;m[7]=0;
  m[8]=0;m[9]=0;m[10]=(far+near)/d;m[11]=-1;
  m[12]=0;m[13]=0;m[14]=2*far*near/d;m[15]=0;
  return m;
}

function lookAt(m, ex,ey,ez, tx,ty,tz){
  let fx=tx-ex,fy=ty-ey,fz=tz-ez;
  let fl=Math.sqrt(fx*fx+fy*fy+fz*fz); fx/=fl;fy/=fl;fz/=fl;
  let rx=fy*0-fz*1,ry=fz*0-fx*0,rz=fx*1-fy*0; // up=(0,1,0)
  let rl=Math.sqrt(rx*rx+ry*ry+rz*rz); rx/=rl;ry/=rl;rz/=rl;
  let ux=ry*fz-rz*fy,uy=rz*fx-rx*fz,uz=rx*fy-ry*fx;
  m[0]=rx;m[1]=ux;m[2]=-fx;m[3]=0;
  m[4]=ry;m[5]=uy;m[6]=-fy;m[7]=0;
  m[8]=rz;m[9]=uz;m[10]=-fz;m[11]=0;
  m[12]=-(rx*ex+ry*ey+rz*ez);
  m[13]=-(ux*ex+uy*ey+uz*ez);
  m[14]= (fx*ex+fy*ey+fz*ez);
  m[15]=1;
  return m;
}

function rotY(m,a){
  const c=Math.cos(a),s=Math.sin(a);
  identity(m);
  m[0]=c;m[2]=s;m[8]=-s;m[10]=c;
  return m;
}

function mul(out,a,b){
  for(let i=0;i<4;i++)for(let j=0;j<4;j++){
    out[i*4+j]=0;
    for(let k=0;k<4;k++) out[i*4+j]+=a[i*4+k]*b[k*4+j];
  }
  return out;
}

function translate(m,x,y,z){
  identity(m);
  m[12]=x;m[13]=y;m[14]=z;
  return m;
}

// ─── Mesh builder ─────────────────────────────────────────────────────────
function makeMesh(verts, norms, colors) {
  // verts: flat float array xyz, norms: flat float array xyz, colors: flat rgb
  const buf = {
    v: gl.createBuffer(), n: gl.createBuffer(), c: gl.createBuffer(),
    count: verts.length/3
  };
  gl.bindBuffer(gl.ARRAY_BUFFER, buf.v);
  gl.bufferData(gl.ARRAY_BUFFER, new Float32Array(verts), gl.STATIC_DRAW);
  gl.bindBuffer(gl.ARRAY_BUFFER, buf.n);
  gl.bufferData(gl.ARRAY_BUFFER, new Float32Array(norms), gl.STATIC_DRAW);
  gl.bindBuffer(gl.ARRAY_BUFFER, buf.c);
  gl.bufferData(gl.ARRAY_BUFFER, new Float32Array(colors), gl.STATIC_DRAW);
  return buf;
}

function drawMesh(buf, mvp, model, baseColor, useVertColor) {
  gl.bindBuffer(gl.ARRAY_BUFFER, buf.v);
  gl.enableVertexAttribArray(aPos);
  gl.vertexAttribPointer(aPos, 3, gl.FLOAT, false, 0, 0);
  gl.bindBuffer(gl.ARRAY_BUFFER, buf.n);
  gl.enableVertexAttribArray(aNorm);
  gl.vertexAttribPointer(aNorm, 3, gl.FLOAT, false, 0, 0);
  gl.bindBuffer(gl.ARRAY_BUFFER, buf.c);
  gl.enableVertexAttribArray(aCol);
  gl.vertexAttribPointer(aCol, 3, gl.FLOAT, false, 0, 0);
  gl.uniformMatrix4fv(uMVP, false, mvp);
  gl.uniformMatrix4fv(uModel, false, model);
  gl.uniform3fv(uBase, baseColor);
  gl.uniform1f(uUseV, useVertColor ? 1.0 : 0.0);
  gl.drawArrays(gl.TRIANGLES, 0, buf.count);
}

// ─── Geometry helpers ─────────────────────────────────────────────────────
function pushQuad(verts,norms,nx,ny,nz, x0,y0,z0, x1,y1,z1, x2,y2,z2, x3,y3,z3){
  // two triangles
  verts.push(x0,y0,z0, x1,y1,z1, x2,y2,z2, x0,y0,z0, x2,y2,z2, x3,y3,z3);
  for(let i=0;i<6;i++) norms.push(nx,ny,nz);
}

function buildBox(w,h,d, cr,cg,cb){
  const verts=[], norms=[], cols=[];
  const hw=w/2, hh=h/2, hd=d/2;
  // +Y top
  pushQuad(verts,norms,0,1,0, -hw,hh,-hd, hw,hh,-hd, hw,hh,hd, -hw,hh,hd);
  // -Y bottom
  pushQuad(verts,norms,0,-1,0, -hw,-hh,hd, hw,-hh,hd, hw,-hh,-hd, -hw,-hh,-hd);
  // +Z front
  pushQuad(verts,norms,0,0,1, -hw,-hh,hd, hw,-hh,hd, hw,hh,hd, -hw,hh,hd);
  // -Z back
  pushQuad(verts,norms,0,0,-1, hw,-hh,-hd, -hw,-hh,-hd, -hw,hh,-hd, hw,hh,-hd);
  // +X right
  pushQuad(verts,norms,1,0,0, hw,-hh,hd, hw,-hh,-hd, hw,hh,-hd, hw,hh,hd);
  // -X left
  pushQuad(verts,norms,-1,0,0, -hw,-hh,-hd, -hw,-hh,hd, -hw,hh,hd, -hw,hh,-hd);
  for(let i=0;i<verts.length/3;i++) cols.push(cr,cg,cb);
  return makeMesh(verts,norms,cols);
}

function buildFlatQuad(w,d, cr,cg,cb){
  const verts=[], norms=[], cols=[];
  const hw=w/2, hd=d/2;
  pushQuad(verts,norms,0,1,0, -hw,0,-hd, hw,0,-hd, hw,0,hd, -hw,0,hd);
  for(let i=0;i<6;i++){norms.push(0,1,0);cols.push(cr,cg,cb);}
  return makeMesh(verts,norms,cols);
}

// ─── World Meshes ─────────────────────────────────────────────────────────
// Ground
const groundMesh = buildFlatQuad(600, 600, 0.29, 0.55, 0.25);

// Road segment mesh cache (reuse per road)
const roadMeshes = [];
const roadData = [
  // [x1,z1, x2,z2]  (y=0, width auto 8)
  [-200,-200, 200,-200, 8],
  [ 200,-200, 200, 200, 8],
  [ 200, 200,-200, 200, 8],
  [-200, 200,-200,-200, 8],
  [-100,-100, 100,-100, 7],
  [ 100,-100, 100, 100, 7],
  [ 100, 100,-100, 100, 7],
  [-100, 100,-100,-100, 7],
  [-200,   0, 200,   0, 7],
  [   0,-200,   0, 200, 7],
  [-200,-200,-100,-100, 5],
  [ 200,-200, 100,-100, 5],
  [ 200, 200, 100, 100, 5],
  [-200, 200,-100, 100, 5],
  [-200,   0,-240,  50, 5],
  [ 200,   0, 240,  50, 5],
  [   0, 200,  50, 240, 5],
  [   0,-200,  50,-240, 5],
];

roadData.forEach(function(r){
  const x1=r[0],z1=r[1],x2=r[2],z2=r[3],hw=r[4];
  const dx=x2-x1, dz=z2-z1;
  const len=Math.sqrt(dx*dx+dz*dz);
  const angle=Math.atan2(dx,dz);
  // build a flat quad for this road
  const verts=[], norms=[], cols=[];
  pushQuad(verts,norms,0,1,0, -hw,0.02,-len/2, hw,0.02,-len/2, hw,0.02,len/2, -hw,0.02,len/2);
  for(let i=0;i<6;i++){cols.push(0.20,0.20,0.20);}
  const m=makeMesh(verts,norms,cols);
  roadMeshes.push({mesh:m, cx:(x1+x2)/2, cz:(z1+z2)/2, angle:angle, len:len});
});

// Buildings
const rnd=(a,b)=>a+Math.random()*(b-a);
const ri=(a,b)=>Math.floor(rnd(a,b));

const bColors=[
  [0.53,0.60,0.67],[0.60,0.67,0.73],[0.48,0.56,0.63],
  [0.69,0.77,0.83],[0.42,0.50,0.60],[0.83,0.77,0.66],
  [0.77,0.72,0.61],[0.40,0.50,0.58],
];

const buildingInstances=[];
const bPos=[
  {x:-145,z:-145},{x:-165,z:-125},{x:-130,z:-165},
  {x:-145,z:-50},{x:-165,z:-30},{x:-130,z:-70},
  {x:-50,z:-145},{x:-30,z:-165},{x:-70,z:-130},
  {x:145,z:-145},{x:165,z:-125},{x:130,z:-165},
  {x:145,z:-50},{x:165,z:-30},{x:130,z:-70},
  {x:50,z:-145},{x:30,z:-165},{x:70,z:-130},
  {x:-145,z:145},{x:-165,z:125},{x:-130,z:165},
  {x:-145,z:50},{x:-165,z:30},{x:-130,z:70},
  {x:-50,z:145},{x:-30,z:165},{x:-70,z:130},
  {x:145,z:145},{x:165,z:125},{x:130,z:165},
  {x:145,z:50},{x:165,z:30},{x:130,z:70},
  {x:50,z:145},{x:30,z:165},{x:70,z:130},
  {x:240,z:-40},{x:240,z:40},{x:-240,z:-40},{x:-240,z:40},
  {x:-40,z:240},{x:40,z:240},{x:-40,z:-240},{x:40,z:-240},
];

bPos.forEach(function(p){
  const w=rnd(10,24), d=rnd(10,24), h=rnd(8,55);
  const col=bColors[ri(0,bColors.length)];
  buildingInstances.push({
    x:p.x+rnd(-4,4), z:p.z+rnd(-4,4), w:w, h:h, d:d,
    mesh: buildBox(w,h,d, col[0],col[1],col[2]),
    col: col
  });
});

// Trees
const treeTrunkMesh = buildBox(0.4, 2.5, 0.4, 0.36,0.24,0.12);
const treeTopMesh   = buildBox(3.2, 3.5, 3.2, 0.18,0.45,0.12);
const treeInstances=[];
function addTree(x,z){
  treeInstances.push({x,z});
}
for(let i=0;i<160;i++){
  const angle=Math.random()*Math.PI*2, dist=rnd(15,55);
  addTree(Math.cos(angle)*dist, Math.sin(angle)*dist);
}
for(let i=0;i<80;i++){
  const x=rnd(-280,280),z=rnd(-280,280);
  if(Math.sqrt(x*x+z*z)>220) addTree(x,z);
}
// roadside trees
[-180,-120,-60,0,60,120,180].forEach(function(v){
  addTree(-205,v); addTree(205,v);
  addTree(v,-205); addTree(v,205);
});

// Park grass disc (flat)
function buildDisc(r,cr,cg,cb){
  const verts=[],norms=[],cols=[];
  const segs=32;
  for(let i=0;i<segs;i++){
    const a0=i/segs*Math.PI*2, a1=(i+1)/segs*Math.PI*2;
    verts.push(0,0.01,0, Math.cos(a0)*r,0.01,Math.sin(a0)*r, Math.cos(a1)*r,0.01,Math.sin(a1)*r);
    for(let j=0;j<3;j++) norms.push(0,1,0);
    for(let j=0;j<3;j++) cols.push(cr,cg,cb);
  }
  return makeMesh(verts,norms,cols);
}
const parkMesh = buildDisc(55, 0.35,0.62,0.26);

// Intersection pads
function buildIntersectionPad(s){
  const verts=[],norms=[],cols=[];
  pushQuad(verts,norms,0,1,0,-s,0.015,-s,s,0.015,-s,s,0.015,s,-s,0.015,s);
  for(let i=0;i<6;i++){cols.push(0.18,0.18,0.18);}
  return makeMesh(verts,norms,cols);
}
const iPadMesh = buildIntersectionPad(12);
const interPos=[
  [-200,-200],[200,-200],[200,200],[-200,200],
  [-100,-100],[100,-100],[100,100],[-100,100],
  [0,0],[-200,0],[200,0],[0,-200],[0,200]
];

// Skybox dome
function buildSkyDome(){
  const verts=[],norms=[],cols=[];
  const R=480,stacks=8,slices=16;
  for(let i=0;i<stacks;i++){
    const phi0=i/stacks*Math.PI/2, phi1=(i+1)/stacks*Math.PI/2;
    for(let j=0;j<slices;j++){
      const th0=j/slices*Math.PI*2, th1=(j+1)/slices*Math.PI*2;
      const t=(i/stacks);
      const cr=lerp3(0.53,0.36,t), cg=lerp3(0.81,0.60,t), cb=lerp3(0.92,0.80,t);
      const pts=[
        [Math.cos(th0)*Math.cos(phi0)*R, Math.sin(phi0)*R, Math.sin(th0)*Math.cos(phi0)*R],
        [Math.cos(th1)*Math.cos(phi0)*R, Math.sin(phi0)*R, Math.sin(th1)*Math.cos(phi0)*R],
        [Math.cos(th1)*Math.cos(phi1)*R, Math.sin(phi1)*R, Math.sin(th1)*Math.cos(phi1)*R],
        [Math.cos(th0)*Math.cos(phi1)*R, Math.sin(phi1)*R, Math.sin(th0)*Math.cos(phi1)*R],
      ];
      verts.push(...pts[0],...pts[1],...pts[2],...pts[0],...pts[2],...pts[3]);
      for(let k=0;k<6;k++){norms.push(0,-1,0);cols.push(cr,cg,cb);}
    }
  }
  return makeMesh(verts,norms,cols);
}
function lerp3(a,b,t){return a+(b-a)*t;}
const skyDome=buildSkyDome();

// ─── Car Mesh ──────────────────────────────────────────────────────────────
// Build car from multiple boxes
const carParts=[];
function addPart(w,h,d,ox,oy,oz,cr,cg,cb){
  carParts.push({mesh:buildBox(w,h,d,cr,cg,cb),ox,oy,oz,cr,cg,cb});
}
// Main body
addPart(2.1, 0.65, 4.4,  0, 0.5, 0,    0.80,0.13,0.07);
// Cabin
addPart(1.75,0.60, 2.1,  0, 1.15,-0.15, 0.67,0.10,0.05);
// Spoiler
addPart(2.0, 0.07, 0.4,  0, 1.1,-2.1,   0.08,0.08,0.08);
// Spoiler posts
addPart(0.1, 0.35, 0.1, -0.85,0.9,-2.1, 0.08,0.08,0.08);
addPart(0.1, 0.35, 0.1,  0.85,0.9,-2.1, 0.08,0.08,0.08);
// Front bumper
addPart(2.1, 0.25, 0.3,  0, 0.22, 2.3,  0.08,0.08,0.08);
// Rear bumper
addPart(2.1, 0.25, 0.3,  0, 0.22,-2.3,  0.08,0.08,0.08);
// Headlights
addPart(0.5,0.22,0.15, -0.65,0.5,2.25,  0.98,0.97,0.80);
addPart(0.5,0.22,0.15,  0.65,0.5,2.25,  0.98,0.97,0.80);
// Taillights (index 9, 10 - we animate these)
addPart(0.55,0.22,0.15,-0.65,0.5,-2.25, 0.80,0.05,0.05);
addPart(0.55,0.22,0.15, 0.65,0.5,-2.25, 0.80,0.05,0.05);

// Wheels (separate so we can rotate them)
const wheelMesh = buildBox(0.35,0.72,0.72, 0.12,0.12,0.12);
const rimMesh   = buildBox(0.36,0.44,0.44, 0.75,0.75,0.75);
const wheelPositions=[
  {x:-1.10,z: 1.45,front:true},
  {x: 1.10,z: 1.45,front:true},
  {x:-1.10,z:-1.45,front:false},
  {x: 1.10,z:-1.45,front:false},
];

// ─── Physics ──────────────────────────────────────────────────────────────
const car={x:0,z:-130,angle:0,vx:0,vz:0,steer:0,wheelRot:0,speed:0};
const CFG={maxSpeed:7.5,acc:0.18,brk:0.28,rev:0.08,
           steerMax:0.048,steerSpeed:0.15,steerReturn:0.12,
           grip:0.82,gripHB:0.96,drag:0.978};

const keys={};
window.addEventListener('keydown',function(e){keys[e.code]=true;
  if(['Space','ArrowUp','ArrowDown','ArrowLeft','ArrowRight'].indexOf(e.code)>=0)e.preventDefault();});
window.addEventListener('keyup',function(e){keys[e.code]=false;});

function lerp(a,b,t){return a+(b-a)*t;}

// ─── Camera ───────────────────────────────────────────────────────────────
const cam={x:0,y:6,z:-145, lx:0,ly:1,lz:-130, sx:0,sy:6,sz:-145, slx:0,sly:1,slz:-130};

// ─── Matrices ─────────────────────────────────────────────────────────────
const mProj=mat4(), mView=mat4(), mModel=mat4(), mTemp=mat4(), mMVP=mat4();
const mIdent=identity(mat4());

function updatePhysics(dt){
  dt=Math.min(dt,0.05);
  const thr=(keys['KeyW']||keys['ArrowUp'])?1:0;
  const brk=(keys['KeyS']||keys['ArrowDown'])?1:0;
  const sl =(keys['KeyA']||keys['ArrowLeft'])?1:0;
  const sr =(keys['KeyD']||keys['ArrowRight'])?1:0;
  const hb = keys['Space']?1:0;

  if(keys['KeyR']){car.x=0;car.z=-130;car.vx=0;car.vz=0;car.angle=0;car.steer=0;}

  const spd=Math.sqrt(car.vx*car.vx+car.vz*car.vz);
  const fx=Math.sin(car.angle), fz=Math.cos(car.angle);
  const lx=Math.cos(car.angle), lz=-Math.sin(car.angle);

  let fwdV=car.vx*fx+car.vz*fz;
  let latV=car.vx*lx+car.vz*lz;

  // Steering
  const tSt=(sl-sr)*CFG.steerMax*Math.min(1,spd/1.5);
  car.steer+=(tSt-car.steer)*CFG.steerSpeed*(dt*60);
  car.steer+=(0-car.steer)*CFG.steerReturn*(dt*60)*(1-Math.abs(sl-sr));

  if(spd>0.05) car.angle+=car.steer*(fwdV>=0?1:-1);

  if(thr)      fwdV=Math.min(fwdV+CFG.acc, CFG.maxSpeed);
  else if(brk){ if(fwdV>0.1)fwdV=Math.max(0,fwdV-CFG.brk); else fwdV=Math.max(-CFG.rev*CFG.maxSpeed,fwdV-CFG.rev); }
  else         fwdV*=Math.pow(CFG.drag,dt*60);

  latV*=Math.pow(hb?0.97:0.15, dt*60);

  car.vx=fx*fwdV+lx*latV;
  car.vz=fz*fwdV+lz*latV;
  car.x+=car.vx; car.z+=car.vz;
  car.x=Math.max(-290,Math.min(290,car.x));
  car.z=Math.max(-290,Math.min(290,car.z));

  car.wheelRot+=fwdV*0.6;
  car.speed=fwdV;

  // Smooth camera behind car
  const CDIST=10, CHEIGHT=4.8, CLAG=6;
  const desX=car.x-Math.sin(car.angle)*CDIST;
  const desY=0.38+CHEIGHT;
  const desZ=car.z-Math.cos(car.angle)*CDIST;
  cam.sx=lerp(cam.sx,desX,CLAG*dt);
  cam.sy=lerp(cam.sy,desY,CLAG*dt);
  cam.sz=lerp(cam.sz,desZ,CLAG*dt);
  const laX=car.x+Math.sin(car.angle)*5;
  const laY=0.38+1.2;
  const laZ=car.z+Math.cos(car.angle)*5;
  cam.slx=lerp(cam.slx,laX,CLAG*dt);
  cam.sly=lerp(cam.sly,laY,CLAG*dt);
  cam.slz=lerp(cam.slz,laZ,CLAG*dt);

  // HUD
  const kmh=Math.abs(fwdV)*28;
  document.getElementById('spd').textContent=Math.round(kmh);
  const rPct=Math.min(spd/CFG.maxSpeed,1);
  const rf=document.getElementById('rpmfill');
  rf.style.width=(rPct*100)+'%';
  rf.style.background=rPct>.85?'linear-gradient(90deg,#FF4400,#FF0000)':
    rPct>.60?'linear-gradient(90deg,#FFB400,#FF6600)':'linear-gradient(90deg,#FFB400,#FF4400)';
  let gear='N';
  if(fwdV<-0.1)gear='R'; else if(spd<0.1)gear='N';
  else if(kmh<25)gear=1; else if(kmh<55)gear=2;
  else if(kmh<90)gear=3; else if(kmh<130)gear=4; else gear=5;
  document.getElementById('gval').textContent=gear;
}

// ─── Render helpers ───────────────────────────────────────────────────────
function drawAt(mesh, x, y, z, angleY, baseCol, useVC) {
  translate(mModel, x, y, z);
  if(angleY) rotY(mTemp, angleY), mul(mModel, mModel, mTemp);
  mul(mMVP, mView, mModel);
  mul(mMVP, mProj, mMVP);
  drawMesh(mesh, mMVP, mModel, baseCol||[1,1,1], useVC||false);
}

// ─── Minimap ──────────────────────────────────────────────────────────────
const mm2=document.getElementById('mm');
const mc=mm2.getContext('2d');
function drawMinimap(){
  mc.clearRect(0,0,130,130);
  mc.fillStyle='rgba(0,18,0,0.9)';mc.fillRect(0,0,130,130);
  const s=130/600, off=65;
  mc.strokeStyle='#3a3a3a';mc.lineWidth=2;
  roadData.forEach(function(r){
    mc.beginPath();mc.moveTo(r[0]*s+off,r[1]*s+off);mc.lineTo(r[2]*s+off,r[3]*s+off);mc.stroke();
  });
  mc.fillStyle='rgba(90,158,66,0.35)';
  mc.beginPath();mc.arc(off,off,55*s,0,Math.PI*2);mc.fill();
  mc.save();mc.translate(car.x*s+off,car.z*s+off);mc.rotate(car.angle);
  mc.fillStyle='#FFB400';mc.fillRect(-3,-5,6,10);mc.restore();
  mc.strokeStyle='rgba(255,180,0,0.3)';mc.lineWidth=1;mc.strokeRect(0,0,130,130);
}

// ─── Main render ──────────────────────────────────────────────────────────
const sunDir=[0.55,0.82,0.35];
let prevTime=performance.now();
let started=false;

function render(now){
  requestAnimationFrame(render);
  const dt=(now-prevTime)/1000; prevTime=now;
  updatePhysics(dt);

  gl.clear(gl.COLOR_BUFFER_BIT|gl.DEPTH_BUFFER_BIT);

  const asp=canvas.width/canvas.height;
  perspective(mProj, 1.05, asp, 0.1, 900);
  lookAt(mView, cam.sx,cam.sy,cam.sz, cam.slx,cam.sly,cam.slz);
  gl.uniform3fv(uSun, sunDir);

  // Sky dome (follow camera, no depth write)
  gl.depthMask(false);
  drawAt(skyDome, cam.sx, cam.sy, cam.sz, 0, [1,1,1], true);
  gl.depthMask(true);

  // Ground
  drawAt(groundMesh, 0, 0, 0, 0, [0.29,0.55,0.25], false);

  // Park
  drawAt(parkMesh, 0, 0, 0, 0, [1,1,1], true);

  // Roads
  roadMeshes.forEach(function(r){
    drawAt(r.mesh, r.cx, 0, r.cz, r.angle, [0.2,0.2,0.2], true);
  });

  // Intersection pads
  interPos.forEach(function(p){
    drawAt(iPadMesh, p[0], 0, p[1], 0, [0.18,0.18,0.18], false);
  });

  // Buildings
  buildingInstances.forEach(function(b){
    drawAt(b.mesh, b.x, b.h/2, b.z, 0, b.col, false);
  });

  // Trees
  treeInstances.forEach(function(t){
    drawAt(treeTrunkMesh, t.x, 1.25, t.z, 0, [0.36,0.24,0.12], false);
    drawAt(treeTopMesh,   t.x, 4.5,  t.z, 0, [0.18,0.45,0.12], false);
  });

  // ── Car ────────────────────────────────────────────────────────────────
  const isBraking=(keys['KeyS']||keys['ArrowDown']);
  carParts.forEach(function(p,idx){
    // Body roll
    const latV=car.vx*Math.cos(car.angle)-car.vz*Math.sin(car.angle);
    const bodyRoll=-latV*0.04;

    // taillights color
    let cr=p.cr,cg=p.cg,cb=p.cb;
    if(idx===9||idx===10){ cr=isBraking?1.0:0.8; cg=isBraking?0.3:0.05; cb=0.05; }

    // compose: car position + rotation + part offset
    const cosA=Math.cos(car.angle), sinA=Math.sin(car.angle);
    const wx=car.x + cosA*p.ox + sinA*p.oz;
    const wy=0.38 + p.oy + Math.sin(car.wheelRot*0.3)*0.01; // subtle bounce
    const wz=car.z - sinA*p.ox + cosA*p.oz;

    translate(mModel, wx, wy, wz);
    rotY(mTemp, car.angle); mul(mModel, mModel, mTemp);

    mul(mMVP, mView, mModel);
    mul(mMVP, mProj, mMVP);
    drawMesh(p.mesh, mMVP, mModel, [cr,cg,cb], false);
  });

  // Wheels
  wheelPositions.forEach(function(wp){
    const cosA=Math.cos(car.angle), sinA=Math.sin(car.angle);
    const wx=car.x + cosA*wp.x + sinA*wp.z;
    const wy=0.38+0.36;
    const wz=car.z - sinA*wp.x + cosA*wp.z;

    const steerA=wp.front?car.steer*10:0;

    translate(mModel, wx, wy, wz);
    // car yaw
    rotY(mTemp, car.angle+steerA); mul(mModel,mModel,mTemp);
    // wheel spin (around X)
    const spinM=mat4(); identity(spinM);
    const sc=Math.cos(car.wheelRot),ss=Math.sin(car.wheelRot);
    spinM[5]=sc;spinM[6]=ss;spinM[9]=-ss;spinM[10]=sc;
    mul(mModel,mModel,spinM);

    mul(mMVP,mView,mModel); mul(mMVP,mProj,mMVP);
    drawMesh(wheelMesh,mMVP,mModel,[0.12,0.12,0.12],false);

    // rim (same transform, slightly different matrix)
    mul(mMVP,mView,mModel); mul(mMVP,mProj,mMVP);
    drawMesh(rimMesh,mMVP,mModel,[0.75,0.75,0.75],false);
  });

  drawMinimap();

  if(!started){
    started=true;
    document.getElementById('loading').style.display='none';
  }
}

requestAnimationFrame(render);
</script>

</body>
</html>