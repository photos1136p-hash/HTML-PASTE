<!DOCTYPE html>

<html lang="en">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>APEX DRIVE</title>
<style>
  * { margin: 0; padding: 0; box-sizing: border-box; }
  body { background: #000; overflow: hidden; font-family: 'Courier New', monospace; }
  canvas { display: block; }
  #hud {
    position: fixed; top: 0; left: 0; width: 100%; height: 100%;
    pointer-events: none; z-index: 10;
  }
  #speedometer {
    position: absolute; bottom: 30px; right: 40px;
    background: rgba(0,0,0,0.7);
    border: 1px solid rgba(255,180,0,0.4);
    border-radius: 16px;
    padding: 20px 28px;
    text-align: center;
    backdrop-filter: blur(10px);
  }
  #speed-val {
    font-size: 52px; font-weight: 900; color: #FFB400;
    line-height: 1; letter-spacing: -2px;
    text-shadow: 0 0 20px rgba(255,180,0,0.6);
  }
  #speed-unit { font-size: 13px; color: rgba(255,180,0,0.6); letter-spacing: 4px; margin-top: 4px; }
  #gear-display {
    position: absolute; bottom: 30px; right: 230px;
    background: rgba(0,0,0,0.7);
    border: 1px solid rgba(255,180,0,0.3);
    border-radius: 14px;
    padding: 16px 22px;
    text-align: center;
    backdrop-filter: blur(10px);
  }
  #gear-val { font-size: 40px; font-weight: 900; color: #fff; line-height:1; }
  #gear-label { font-size: 11px; color: rgba(255,255,255,0.4); letter-spacing: 3px; margin-top: 4px; }
  #title-banner {
    position: absolute; top: 24px; left: 50%; transform: translateX(-50%);
    font-size: 13px; letter-spacing: 8px; color: rgba(255,180,0,0.7);
    font-weight: 700; text-transform: uppercase;
  }
  #controls-hint {
    position: absolute; bottom: 30px; left: 40px;
    background: rgba(0,0,0,0.65);
    border: 1px solid rgba(255,255,255,0.1);
    border-radius: 14px;
    padding: 16px 20px;
    backdrop-filter: blur(10px);
  }
  #controls-hint div { font-size: 12px; color: rgba(255,255,255,0.55); line-height: 1.9; }
  #controls-hint span { color: #FFB400; font-weight: bold; }
  #rpm-bar-wrap {
    position: absolute; bottom: 155px; right: 40px;
    width: 260px;
  }
  #rpm-label { font-size: 10px; color: rgba(255,180,0,0.5); letter-spacing: 4px; text-align: right; margin-bottom: 6px; }
  #rpm-bar-bg {
    width: 100%; height: 8px; background: rgba(255,255,255,0.08);
    border-radius: 4px; overflow: hidden;
  }
  #rpm-bar { height: 100%; width: 0%; background: linear-gradient(90deg, #FFB400, #FF4400); border-radius: 4px; transition: width 0.05s; }
  #minimap {
    position: absolute; top: 20px; right: 30px;
    width: 130px; height: 130px;
    background: rgba(0,0,0,0.65);
    border: 1px solid rgba(255,180,0,0.25);
    border-radius: 10px;
    overflow: hidden;
  }
  #minimap canvas { width: 100%; height: 100%; }
  .key-badge {
    display: inline-block;
    background: rgba(255,255,255,0.12);
    border-radius: 4px;
    padding: 1px 6px;
    font-size: 11px;
  }
</style>
</head>
<body>
<div id="hud">
  <div id="title-banner">APEX DRIVE — FREE ROAM</div>
  <div id="minimap"><canvas id="mm" width="130" height="130"></canvas></div>
  <div id="rpm-bar-wrap">
    <div id="rpm-label">RPM</div>
    <div id="rpm-bar-bg"><div id="rpm-bar"></div></div>
  </div>
  <div id="gear-display">
    <div id="gear-val">N</div>
    <div id="gear-label">GEAR</div>
  </div>
  <div id="speedometer">
    <div id="speed-val">0</div>
    <div id="speed-unit">KM/H</div>
  </div>
  <div id="controls-hint">
    <div><span class="key-badge">W/↑</span> Accelerate</div>
    <div><span class="key-badge">S/↓</span> Brake / Reverse</div>
    <div><span class="key-badge">A/D ←/→</span> Steer</div>
    <div><span class="key-badge">SPACE</span> Handbrake</div>
    <div><span class="key-badge">R</span> Reset Car</div>
  </div>
</div>
<script src="https://cdnjs.cloudflare.com/ajax/libs/three.js/r128/three.min.js"></script>
<script>
// ─── SCENE SETUP ───────────────────────────────────────────────────────────
const scene = new THREE.Scene();
scene.background = new THREE.Color(0x87CEEB);
scene.fog = new THREE.Fog(0x87CEEB, 120, 600);

const renderer = new THREE.WebGLRenderer({ antialias: true });
renderer.setSize(window.innerWidth, window.innerHeight);
renderer.shadowMap.enabled = true;
renderer.shadowMap.type = THREE.PCFSoftShadowMap;
renderer.setPixelRatio(Math.min(window.devicePixelRatio, 2));
document.body.appendChild(renderer.domElement);

const camera = new THREE.PerspectiveCamera(60, window.innerWidth / window.innerHeight, 0.1, 1000);

window.addEventListener(‘resize’, () => {
camera.aspect = window.innerWidth / window.innerHeight;
camera.updateProjectionMatrix();
renderer.setSize(window.innerWidth, window.innerHeight);
});

// ─── LIGHTING ──────────────────────────────────────────────────────────────
const ambientLight = new THREE.AmbientLight(0xffeedd, 0.6);
scene.add(ambientLight);

const sunLight = new THREE.DirectionalLight(0xfff5e0, 1.4);
sunLight.position.set(200, 300, 100);
sunLight.castShadow = true;
sunLight.shadow.mapSize.set(2048, 2048);
sunLight.shadow.camera.near = 0.5;
sunLight.shadow.camera.far = 800;
sunLight.shadow.camera.left = -300;
sunLight.shadow.camera.right = 300;
sunLight.shadow.camera.top = 300;
sunLight.shadow.camera.bottom = -300;
sunLight.shadow.bias = -0.001;
scene.add(sunLight);

const fillLight = new THREE.DirectionalLight(0x6699ff, 0.3);
fillLight.position.set(-100, 50, -100);
scene.add(fillLight);

// ─── TERRAIN & MAP ─────────────────────────────────────────────────────────
const WORLD = 600;

// Ground
const groundGeo = new THREE.PlaneGeometry(WORLD, WORLD, 80, 80);
const groundMat = new THREE.MeshLambertMaterial({ color: 0x4a7c3f });
const ground = new THREE.Mesh(groundGeo, groundMat);
ground.rotation.x = -Math.PI / 2;
ground.receiveShadow = true;
scene.add(ground);

// Road network helper
function makeRoad(x1,z1,x2,z2,w=10) {
const dx = x2-x1, dz = z2-z1;
const len = Math.sqrt(dx*dx+dz*dz);
const roadGeo = new THREE.PlaneGeometry(w, len);
const roadMat = new THREE.MeshLambertMaterial({ color: 0x333333 });
const road = new THREE.Mesh(roadGeo, roadMat);
road.rotation.x = -Math.PI/2;
road.position.set((x1+x2)/2, 0.01, (z1+z2)/2);
road.rotation.z = -Math.atan2(dz, dx) + Math.PI/2;
road.receiveShadow = true;
scene.add(road);

// white center dashes
const dashCount = Math.floor(len / 20);
for(let i=0;i<dashCount;i++) {
const t = (i+0.5)/dashCount;
const dGeo = new THREE.PlaneGeometry(0.6, 6);
const dMat = new THREE.MeshLambertMaterial({ color: 0xffffff });
const dash = new THREE.Mesh(dGeo, dMat);
dash.rotation.x = -Math.PI/2;
dash.position.set(x1+dx*t, 0.02, z1+dz*t);
dash.rotation.z = -Math.atan2(dz,dx)+Math.PI/2;
scene.add(dash);
}
}

// Road circuit + grid streets
const roads = [
// Outer ring road
[-200,-200, 200,-200],
[200,-200, 200,200],
[200,200, -200,200],
[-200,200, -200,-200],
// Inner ring
[-100,-100, 100,-100],
[100,-100, 100,100],
[100,100, -100,100],
[-100,100, -100,-100],
// Cross roads
[-200,0, 200,0],
[0,-200, 0,200],
// Diagonal connector
[-200,-200, -100,-100],
[200,-200, 100,-100],
[200,200, 100,100],
[-200,200, -100,100],
// Spurs
[-200,0, -240,60],
[200,0, 240,60],
[0,200, 60,240],
[0,-200, 60,-240],
];
roads.forEach(r => makeRoad(r[0],r[1],r[2],r[3], 12));

// Intersections (wider pads)
[[-200,-200],[200,-200],[200,200],[-200,200],
[-100,-100],[100,-100],[100,100],[-100,100],
[0,0],[-200,0],[200,0],[0,-200],[0,200]].forEach(([x,z]) => {
const geo = new THREE.PlaneGeometry(18,18);
const mat = new THREE.MeshLambertMaterial({ color: 0x2e2e2e });
const m = new THREE.Mesh(geo, mat);
m.rotation.x = -Math.PI/2; m.position.set(x,0.015,z); m.receiveShadow=true;
scene.add(m);
});

// ─── BUILDINGS ─────────────────────────────────────────────────────────────
const buildingColors = [0x8899aa, 0x99aabb, 0x7a8fa0, 0xb0c4d4, 0x6b8096, 0xd4c5a9, 0xc9b99a];
const rng = (a,b) => a + Math.random()*(b-a);

function addBuilding(x,z,w,d,h,color) {
// Body
const geo = new THREE.BoxGeometry(w,h,d);
const mat = new THREE.MeshLambertMaterial({ color });
const b = new THREE.Mesh(geo, mat);
b.position.set(x, h/2, z);
b.castShadow = true; b.receiveShadow = true;
scene.add(b);
// Roof detail
if(h>15) {
const rGeo = new THREE.BoxGeometry(w*0.6, 2, d*0.6);
const rMat = new THREE.MeshLambertMaterial({ color: 0x555566 });
const roof = new THREE.Mesh(rGeo, rMat);
roof.position.set(x, h+1, z);
scene.add(roof);
}
// Windows (simple grid texture via lines)
const wRows = Math.floor(h/4);
const wCols = Math.floor(w/4);
for(let r=0;r<wRows;r++) {
for(let c=0;c<wCols;c++) {
if(Math.random()<0.7) {
const wg = new THREE.PlaneGeometry(1.2,1.6);
const wm = new THREE.MeshLambertMaterial({ color: Math.random()<0.4?0xffffcc:0x334455 });
const win = new THREE.Mesh(wg, wm);
win.position.set(x-w/2+2+c*4, 3+r*4, z+d/2+0.01);
scene.add(win);
const win2 = win.clone();
win2.position.set(x-w/2+2+c*4, 3+r*4, z-d/2-0.01);
win2.rotation.y = Math.PI;
scene.add(win2);
}
}
}
}

// City blocks
const blocks = [
// NW quadrant
{x:-145,z:-145},{x:-165,z:-125},{x:-130,z:-165},
{x:-145,z:-50},{x:-165,z:-30},{x:-130,z:-70},
{x:-50,z:-145},{x:-30,z:-165},{x:-70,z:-130},
// NE quadrant
{x:145,z:-145},{x:165,z:-125},{x:130,z:-165},
{x:145,z:-50},{x:165,z:-30},{x:130,z:-70},
{x:50,z:-145},{x:30,z:-165},{x:70,z:-130},
// SW
{x:-145,z:145},{x:-165,z:125},{x:-130,z:165},
{x:-145,z:50},{x:-165,z:30},{x:-130,z:70},
{x:-50,z:145},{x:-30,z:165},{x:-70,z:130},
// SE
{x:145,z:145},{x:165,z:125},{x:130,z:165},
{x:145,z:50},{x:165,z:30},{x:130,z:70},
{x:50,z:145},{x:30,z:165},{x:70,z:130},
// Outer
{x:240,z:-40},{x:240,z:40},{x:-240,z:-40},{x:-240,z:40},
{x:-40,z:240},{x:40,z:240},{x:-40,z:-240},{x:40,z:-240},
];

blocks.forEach(({x,z}) => {
const w = rng(12,28), d = rng(12,28), h = rng(8,55);
addBuilding(x+rng(-5,5), z+rng(-5,5), w, d, h, buildingColors[Math.floor(Math.random()*buildingColors.length)]);
});

// ─── TREES ─────────────────────────────────────────────────────────────────
function addTree(x,z) {
const trunk = new THREE.Mesh(
new THREE.CylinderGeometry(0.3,0.4,3,6),
new THREE.MeshLambertMaterial({ color: 0x5c3d1e })
);
trunk.position.set(x,1.5,z); trunk.castShadow=true; scene.add(trunk);
const foliage = new THREE.Mesh(
new THREE.SphereGeometry(rng(2.5,4),7,7),
new THREE.MeshLambertMaterial({ color: 0x2d6a1f })
);
foliage.position.set(x,5.5+rng(0,2),z); foliage.castShadow=true; scene.add(foliage);
}

// Scatter trees in open areas
for(let i=0;i<200;i++) {
const x = rng(-280,280), z = rng(-280,280);
const r = Math.sqrt(x*x+z*z);
if(r>220 || (Math.abs(x)>30 && Math.abs(z)>30)) addTree(x,z);
}

// Park in center
for(let i=0;i<30;i++) {
const angle = Math.random()*Math.PI*2, dist = rng(15,55);
addTree(Math.cos(angle)*dist, Math.sin(angle)*dist);
}

// Park grass circle
const parkGeo = new THREE.CircleGeometry(60,32);
const parkMat = new THREE.MeshLambertMaterial({ color: 0x5a9e42 });
const park = new THREE.Mesh(parkGeo, parkMat);
park.rotation.x = -Math.PI/2; park.position.set(0,0.005,0); park.receiveShadow=true;
scene.add(park);

// ─── STREET LIGHTS ─────────────────────────────────────────────────────────
function addLight(x,z) {
const pole = new THREE.Mesh(
new THREE.CylinderGeometry(0.12,0.15,8,6),
new THREE.MeshLambertMaterial({ color: 0x888888 })
);
pole.position.set(x,4,z); scene.add(pole);
const head = new THREE.Mesh(
new THREE.SphereGeometry(0.4,6,6),
new THREE.MeshBasicMaterial({ color: 0xffffaa })
);
head.position.set(x,8.2,z); scene.add(head);
const pLight = new THREE.PointLight(0xffffcc, 0.8, 40);
pLight.position.set(x,8,z); scene.add(pLight);
}

[[-195,-6],[-195,6],[195,-6],[195,6],
[-6,-195],[6,-195],[-6,195],[6,195],
[-195,-100],[-195,100],[195,-100],[195,100],
[-100,-195],[100,-195],[-100,195],[100,195]].forEach(([x,z])=>addLight(x,z));

// ─── CAR ──────────────────────────────────────────────────────────────────
const carGroup = new THREE.Group();

// Car body
const bodyGeo = new THREE.BoxGeometry(2.2, 0.7, 4.5);
const bodyMat = new THREE.MeshLambertMaterial({ color: 0xcc2211 });
const carBody = new THREE.Mesh(bodyGeo, bodyMat);
carBody.position.y = 0.5; carBody.castShadow=true;
carGroup.add(carBody);

// Cabin
const cabinGeo = new THREE.BoxGeometry(1.8, 0.65, 2.2);
const cabinMat = new THREE.MeshLambertMaterial({ color: 0xaa1a0e });
const cabin = new THREE.Mesh(cabinGeo, cabinMat);
cabin.position.set(0, 1.15, -0.2); cabin.castShadow=true;
carGroup.add(cabin);

// Windshields
const wGeo = new THREE.PlaneGeometry(1.6, 0.55);
const wMat = new THREE.MeshLambertMaterial({ color: 0x88ccff, transparent:true, opacity:0.6 });
const windshield = new THREE.Mesh(wGeo, wMat);
windshield.position.set(0,1.2,0.91); windshield.rotation.x=0.3;
carGroup.add(windshield);
const rearshield = windshield.clone();
rearshield.position.set(0,1.2,-1.31); rearshield.rotation.x=-0.3;
rearshield.rotation.y=Math.PI;
carGroup.add(rearshield);

// Spoiler
const spoilerGeo = new THREE.BoxGeometry(2, 0.08, 0.4);
const spoilerMat = new THREE.MeshLambertMaterial({ color: 0x111111 });
const spoiler = new THREE.Mesh(spoilerGeo, spoilerMat);
spoiler.position.set(0,1.15,-2.1);
carGroup.add(spoiler);
const sPost1 = new THREE.Mesh(new THREE.BoxGeometry(0.1,0.4,0.1), spoilerMat);
sPost1.position.set(-0.85,0.95,-2.1); carGroup.add(sPost1);
const sPost2 = sPost1.clone(); sPost2.position.set(0.85,0.95,-2.1); carGroup.add(sPost2);

// Headlights
const hlGeo = new THREE.SphereGeometry(0.22,8,8);
const hlMat = new THREE.MeshBasicMaterial({ color: 0xffffee });
[-0.7,0.7].forEach(x=>{
const hl = new THREE.Mesh(hlGeo,hlMat);
hl.position.set(x,0.5,2.28); hl.scale.z=0.3; carGroup.add(hl);
const tl = new THREE.Mesh(hlGeo, new THREE.MeshBasicMaterial({color:0xff2200}));
tl.position.set(x,0.5,-2.28); tl.scale.z=0.3; carGroup.add(tl);
});

// Wheels
const wheelGeo = new THREE.CylinderGeometry(0.38, 0.38, 0.35, 12);
const wheelMat = new THREE.MeshLambertMaterial({ color: 0x111111 });
const rimGeo = new THREE.CylinderGeometry(0.22,0.22,0.36,8);
const rimMat = new THREE.MeshLambertMaterial({ color: 0xcccccc });

const wheelPositions = [
{ x:-1.15, z:1.5, front:true },
{ x: 1.15, z:1.5, front:true },
{ x:-1.15, z:-1.5, front:false },
{ x: 1.15, z:-1.5, front:false },
];

const wheels = [];
wheelPositions.forEach((wp,i)=>{
const wg = new THREE.Group();
const w = new THREE.Mesh(wheelGeo, wheelMat);
w.rotation.z = Math.PI/2;
wg.add(w);
const rim = new THREE.Mesh(rimGeo, rimMat);
rim.rotation.z = Math.PI/2;
wg.add(rim);
// Spokes
for(let s=0;s<5;s++){
const spoke = new THREE.Mesh(new THREE.BoxGeometry(0.06,0.28,0.04), rimMat);
spoke.rotation.z = (s/5)*Math.PI*2;
wg.add(spoke);
}
wg.position.set(wp.x, 0.38, wp.z);
carGroup.add(wg);
wheels.push({ group:wg, front:wp.front });
});

carGroup.position.set(0, 0.38, -130);
scene.add(carGroup);

// ─── PHYSICS STATE ─────────────────────────────────────────────────────────
const physics = {
speed: 0,
steerAngle: 0,
heading: 0,       // radians
vx: 0, vz: 0,    // velocity in world space
lat_vel: 0,
spin: 0,          // angular velocity
onGround: true,
wheelRot: 0,
handbrake: false,
};

const CAR_CONFIG = {
maxSpeed: 55,
acceleration: 18,
braking: 30,
reverseSpeed: 12,
steerSpeed: 2.2,
steerReturn: 3.5,
maxSteer: 0.55,
friction: 8,
lateralFriction: 14,
lateralFrictionHB: 3,
downforce: 12,
drag: 0.5,
};

// ─── INPUT ─────────────────────────────────────────────────────────────────
const keys = {};
window.addEventListener(‘keydown’, e => { keys[e.code] = true; e.preventDefault(); });
window.addEventListener(‘keyup’, e => { keys[e.code] = false; });

// ─── SMOOTH CAMERA ─────────────────────────────────────────────────────────
const camState = {
pos: new THREE.Vector3(0, 6, -140),
target: new THREE.Vector3(0, 1, -130),
currentLookAt: new THREE.Vector3(),
};

// ─── MINIMAP ───────────────────────────────────────────────────────────────
const mm = document.getElementById(‘mm’);
const mmCtx = mm.getContext(‘2d’);

function drawMinimap() {
mmCtx.clearRect(0,0,130,130);
mmCtx.fillStyle = ‘rgba(0,20,0,0.85)’;
mmCtx.fillRect(0,0,130,130);
const scale = 130 / WORLD;
const cx = 65, cy = 65;

// roads
mmCtx.strokeStyle = ‘#555’; mmCtx.lineWidth = 3;
roads.forEach(([x1,z1,x2,z2]) => {
mmCtx.beginPath();
mmCtx.moveTo(cx+x1*scale, cy+z1*scale);
mmCtx.lineTo(cx+x2*scale, cy+z2*scale);
mmCtx.stroke();
});

// car dot
const carX = cx + carGroup.position.x * scale;
const carZ = cy + carGroup.position.z * scale;
mmCtx.save();
mmCtx.translate(carX, carZ);
mmCtx.rotate(physics.heading);
mmCtx.fillStyle = ‘#FFB400’;
mmCtx.fillRect(-3,-5,6,10);
mmCtx.restore();

// border
mmCtx.strokeStyle = ‘rgba(255,180,0,0.3)’; mmCtx.lineWidth = 1;
mmCtx.strokeRect(0,0,130,130);
}

// ─── UPDATE ────────────────────────────────────────────────────────────────
let prev = performance.now();
const RESET_POS = new THREE.Vector3(0, 0.38, -130);

function update(dt) {
dt = Math.min(dt, 0.05);

const throttle = (keys[‘KeyW’]||keys[‘ArrowUp’]) ? 1 : 0;
const brake    = (keys[‘KeyS’]||keys[‘ArrowDown’]) ? 1 : 0;
const steerL   = (keys[‘KeyA’]||keys[‘ArrowLeft’]) ? 1 : 0;
const steerR   = (keys[‘KeyD’]||keys[‘ArrowRight’]) ? 1 : 0;
const hb       = keys[‘Space’] ? 1 : 0;

if(keys[‘KeyR’]) {
carGroup.position.copy(RESET_POS);
physics.speed = 0; physics.vx = 0; physics.vz = 0;
physics.heading = 0; physics.steerAngle = 0; physics.spin = 0;
carGroup.rotation.y = 0;
}

const spd = Math.abs(physics.speed);
const lateralFric = hb ? CAR_CONFIG.lateralFrictionHB : CAR_CONFIG.lateralFriction;

// Steering
const targetSteer = (steerL - steerR) * CAR_CONFIG.maxSteer;
const steerBlend = Math.min(spd / 10, 1);
physics.steerAngle += (targetSteer - physics.steerAngle) * CAR_CONFIG.steerSpeed * dt;
physics.steerAngle *= 1 - CAR_CONFIG.steerReturn * dt * (1 - steerBlend * 0.5);

// Torque -> angular velocity based on speed and steer
const turnStrength = (physics.speed / CAR_CONFIG.maxSpeed) * physics.steerAngle * 1.8;
physics.spin += turnStrength * dt * 5;
physics.spin *= Math.pow(0.1, dt);
physics.heading += physics.spin * dt;

// Accelerate / brake
if(throttle) {
physics.speed += CAR_CONFIG.acceleration * dt;
} else if(brake) {
if(physics.speed > 0.5) {
physics.speed -= CAR_CONFIG.braking * dt;
} else {
physics.speed -= CAR_CONFIG.reverseSpeed * dt * 0.6;
}
}

// Friction / drag
const frictionForce = CAR_CONFIG.friction + spd * CAR_CONFIG.drag;
if(!throttle && !brake) {
physics.speed *= Math.pow(1 - CAR_CONFIG.friction * 0.04, dt * 60);
}
physics.speed = Math.max(-CAR_CONFIG.reverseSpeed, Math.min(CAR_CONFIG.maxSpeed, physics.speed));

// World velocity
const fwdX = Math.sin(physics.heading);
const fwdZ = Math.cos(physics.heading);
const latX = Math.cos(physics.heading);
const latZ = -Math.sin(physics.heading);

// Forward velocity
const fwdVel = physics.vx * fwdX + physics.vz * fwdZ;
// Lateral velocity (slip)
let latVel = physics.vx * latX + physics.vz * latZ;

// Target fwd velocity = speed
const targetFwdVel = physics.speed;
const newFwdVel = fwdVel + (targetFwdVel - fwdVel) * (1 - Math.pow(0.001, dt));

// Reduce lateral (grip)
latVel *= Math.pow(1 - lateralFric * dt, 1);

physics.vx = fwdX * newFwdVel + latX * latVel;
physics.vz = fwdZ * newFwdVel + latZ * latVel;

// Move car
carGroup.position.x += physics.vx * dt;
carGroup.position.z += physics.vz * dt;
carGroup.position.y = 0.38; // flat ground

// World bounds
carGroup.position.x = Math.max(-290, Math.min(290, carGroup.position.x));
carGroup.position.z = Math.max(-290, Math.min(290, carGroup.position.z));

// Rotate car body
carGroup.rotation.y = physics.heading;

// Rotate wheels
physics.wheelRot += physics.speed * dt * 2.2;
wheels.forEach(({group, front}) => {
group.rotation.x = physics.wheelRot;
if(front) group.rotation.y = physics.steerAngle * 0.8;
});

// Body roll & pitch
const latG = latVel * 0.015;
const fwdG = (throttle - brake) * 0.015;
carBody.rotation.z = -latG;
carBody.rotation.x = fwdG;
cabin.rotation.z = -latG * 0.5;

// Taillights when braking
const tails = carGroup.children.filter(c => c.material && c.material.color &&
c.material.color.getHex() === 0xff2200);
tails.forEach(t => t.material.color.set(brake ? 0xff5500 : 0xff2200));

// ─── CAMERA ──────────────────────────────────────────────────────────────
const CAM_DIST = 9.5;
const CAM_HEIGHT = 4.5;
const CAM_LAG = 5.0;

const desiredCamX = carGroup.position.x - Math.sin(physics.heading) * CAM_DIST;
const desiredCamY = carGroup.position.y + CAM_HEIGHT;
const desiredCamZ = carGroup.position.z - Math.cos(physics.heading) * CAM_DIST;

camState.pos.x += (desiredCamX - camState.pos.x) * CAM_LAG * dt;
camState.pos.y += (desiredCamY - camState.pos.y) * CAM_LAG * dt;
camState.pos.z += (desiredCamZ - camState.pos.z) * CAM_LAG * dt;

const lookAtX = carGroup.position.x + Math.sin(physics.heading) * 4;
const lookAtY = carGroup.position.y + 1.0;
const lookAtZ = carGroup.position.z + Math.cos(physics.heading) * 4;

camState.currentLookAt.x += (lookAtX - camState.currentLookAt.x) * CAM_LAG * dt;
camState.currentLookAt.y += (lookAtY - camState.currentLookAt.y) * CAM_LAG * dt;
camState.currentLookAt.z += (lookAtZ - camState.currentLookAt.z) * CAM_LAG * dt;

camera.position.copy(camState.pos);
camera.lookAt(camState.currentLookAt);

// ─── HUD UPDATE ──────────────────────────────────────────────────────────
const kmh = Math.abs(physics.speed) * 3.6;
document.getElementById(‘speed-val’).textContent = Math.round(kmh);

const rpmPct = Math.min(spd / CAR_CONFIG.maxSpeed, 1);
document.getElementById(‘rpm-bar’).style.width = (rpmPct * 100) + ‘%’;
document.getElementById(‘rpm-bar’).style.background =
rpmPct > 0.85 ? ‘linear-gradient(90deg,#FF4400,#FF0000)’ :
rpmPct > 0.6  ? ‘linear-gradient(90deg,#FFB400,#FF6600)’ :
‘linear-gradient(90deg,#FFB400,#FF4400)’;

// Gear
const gearBands = [0,8,18,30,42,55];
let gear = ‘N’;
if(physics.speed < -0.5) gear = ‘R’;
else if(spd < 1) gear = ‘N’;
else {
for(let g=1;g<gearBands.length;g++) {
if(spd*3.6 <= gearBands[g]*3.6 || g === gearBands.length-1) { gear = g; break; }
}
}
document.getElementById(‘gear-val’).textContent = gear;

drawMinimap();
}

// ─── ANIMATE ───────────────────────────────────────────────────────────────
function animate(now) {
requestAnimationFrame(animate);
const dt = (now - prev) / 1000;
prev = now;
update(dt);
renderer.render(scene, camera);
}
requestAnimationFrame(animate);
</script>

</body>
</html>