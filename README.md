<!DOCTYPE html>

<html lang="en">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>Keep — My Notes</title>
<link href="https://fonts.googleapis.com/css2?family=Google+Sans:wght@400;500;700&display=swap" rel="stylesheet">
<script src="https://www.gstatic.com/firebasejs/9.22.0/firebase-app-compat.js"></script>
<script src="https://www.gstatic.com/firebasejs/9.22.0/firebase-database-compat.js"></script>
<style>
  :root{
    --yellow:#fbbc04;--bg:#f5f5f5;--surface:#fff;--border:#e0e0e0;
    --text:#202124;--muted:#5f6368;--accent:#1a73e8;
    --hover:rgba(0,0,0,.06);
    --s1:0 1px 3px rgba(0,0,0,.12),0 1px 2px rgba(0,0,0,.08);
    --s2:0 4px 16px rgba(0,0,0,.18);--r:8px;
  }
  *{box-sizing:border-box;margin:0;padding:0;}
  body{font-family:'Google Sans',sans-serif;background:var(--bg);color:var(--text);min-height:100vh;}

/* SETUP */
.setup{position:fixed;inset:0;background:#fff;z-index:600;display:flex;align-items:center;justify-content:center;padding:24px;}
.setup-box{max-width:440px;width:100%;text-align:center;}
.setup-logo{width:60px;height:60px;margin:0 auto 20px;}
.setup-box h2{font-size:26px;margin-bottom:8px;}
.setup-box p{font-size:14px;color:var(–muted);margin-bottom:24px;line-height:1.7;}
.setup-box strong{color:var(–text);}
.setup-input{
width:100%;padding:12px 16px;border:1px solid var(–border);border-radius:8px;
font-size:16px;font-family:inherit;outline:none;margin-bottom:12px;
transition:border-color .2s,box-shadow .2s;
}
.setup-input:focus{border-color:var(–accent);box-shadow:0 0 0 3px rgba(26,115,232,.15);}
.setup-btn{
width:100%;padding:13px;background:var(–accent);color:#fff;border:none;
border-radius:8px;font-size:16px;font-weight:500;font-family:inherit;cursor:pointer;
}
.setup-btn:hover{background:#1557b0;}
.setup-hint{font-size:12px;color:var(–muted);margin-top:14px;line-height:1.6;}

/* HEADER */
header{
position:sticky;top:0;z-index:100;background:var(–surface);
border-bottom:1px solid var(–border);display:flex;align-items:center;
padding:0 16px;height:64px;gap:12px;
}
.logo{display:flex;align-items:center;gap:8px;}
.logo svg{width:36px;height:36px;flex-shrink:0;}
.logo-text{font-size:20px;color:var(–muted);font-weight:400;white-space:nowrap;overflow:hidden;text-overflow:ellipsis;max-width:200px;}
.search-wrap{flex:1;max-width:680px;margin:0 auto;position:relative;}
.search-wrap input{
width:100%;padding:10px 16px 10px 44px;background:#f1f3f4;border:none;
border-radius:24px;font-size:16px;font-family:inherit;outline:none;
transition:background .2s,box-shadow .2s;
}
.search-wrap input:focus{background:#fff;box-shadow:0 2px 8px rgba(0,0,0,.2);}
.si{position:absolute;left:12px;top:50%;transform:translateY(-50%);color:var(–muted);}
.hright{display:flex;align-items:center;gap:8px;flex-shrink:0;}
.sync-pill{
display:flex;align-items:center;gap:6px;padding:4px 12px;
border-radius:20px;font-size:12px;font-weight:500;color:var(–muted);
border:1px solid var(–border);white-space:nowrap;
}
.dot{width:8px;height:8px;border-radius:50%;background:#34a853;flex-shrink:0;}
.dot.pulsing{background:var(–yellow);animation:pulse 1s infinite;}
.dot.off{background:#ea4335;}
@keyframes pulse{0%,100%{opacity:1}50%{opacity:.3}}
.ib{
width:40px;height:40px;border-radius:50%;border:none;background:transparent;
cursor:pointer;display:flex;align-items:center;justify-content:center;
color:var(–muted);transition:background .15s;flex-shrink:0;
}
.ib:hover{background:var(–hover);}

/* MAIN */
main{max-width:1200px;margin:0 auto;padding:24px 16px;}
.cw{max-width:600px;margin:0 auto 32px;}
.composer{
background:#fff;border:1px solid var(–border);border-radius:var(–r);
box-shadow:var(–s1);overflow:hidden;transition:box-shadow .2s;
}
.composer:focus-within{box-shadow:var(–s2);}
.ct{
padding:12px 16px 4px;font-size:16px;font-weight:500;border:none;
background:transparent;width:100%;outline:none;font-family:inherit;
color:var(–text);display:none;
}
.ct.show{display:block;}
.cb{
padding:12px 16px;font-size:14px;line-height:1.6;border:none;background:transparent;
width:100%;outline:none;font-family:inherit;color:var(–text);resize:none;min-height:48px;
}
.cf{display:none;padding:6px 8px 8px;justify-content:space-between;align-items:center;}
.cf.show{display:flex;}
.btn{padding:8px 24px;border-radius:4px;border:none;font-family:inherit;font-size:14px;font-weight:500;cursor:pointer;}
.ghost{background:transparent;color:var(–muted);}
.ghost:hover{background:var(–hover);}

/* COLORS */
.crow{display:flex;gap:6px;padding:4px 8px;flex-wrap:wrap;}
.sw{
width:28px;height:28px;border-radius:50%;cursor:pointer;
border:2px solid transparent;transition:transform .15s,border-color .15s;
position:relative;flex-shrink:0;
}
.sw:hover{transform:scale(1.15);}
.sw.sel{border-color:#555;}
.sw.none{background:#fff;border-color:#ddd;}
.sw.none::after{content:‘✕’;position:absolute;top:50%;left:50%;transform:translate(-50%,-50%);font-size:10px;color:#bbb;}

.slbl{font-size:11px;font-weight:500;letter-spacing:.8px;color:var(–muted);text-transform:uppercase;margin-bottom:12px;padding:0 4px;}
.grid{columns:5 180px;column-gap:12px;}
@media(max-width:600px){.grid{columns:2 140px;}}

/* CARD */
.card{
break-inside:avoid;margin-bottom:12px;background:#fff;
border:1px solid var(–border);border-radius:var(–r);box-shadow:var(–s1);
transition:box-shadow .2s,transform .15s;position:relative;cursor:pointer;
animation:fi .2s ease;
}
@keyframes fi{from{opacity:0;transform:scale(.96) translateY(4px)}to{opacity:1;transform:none}}
.card:hover{box-shadow:var(–s2);transform:translateY(-1px);}
.card:hover .ca{opacity:1;}
.card.pin{border-top:3px solid var(–yellow);}
.ct2{padding:12px 32px 4px 12px;font-size:14px;font-weight:500;word-break:break-word;}
.cb2{padding:4px 12px 12px;font-size:13px;line-height:1.6;color:var(–muted);word-break:break-word;white-space:pre-wrap;}
.cft{padding:4px;display:flex;justify-content:space-between;align-items:center;}
.ca{opacity:0;display:flex;gap:2px;transition:opacity .15s;}
.ctm{font-size:11px;color:#bbb;padding:0 8px;}
.pb{
position:absolute;top:6px;right:6px;background:transparent;border:none;
cursor:pointer;padding:4px;border-radius:50%;opacity:0;transition:opacity .15s;color:var(–muted);
}
.card:hover .pb,.card.pin .pb{opacity:1;}
.card.pin .pb{color:var(–text);}

/* MODAL */
.ov{
position:fixed;inset:0;background:rgba(0,0,0,.5);z-index:200;
display:flex;align-items:center;justify-content:center;padding:16px;
}
.modal{
background:#fff;border-radius:var(–r);width:100%;max-width:600px;
max-height:90vh;overflow-y:auto;box-shadow:var(–s2);
animation:su .2s ease;display:flex;flex-direction:column;
}
@keyframes su{from{transform:translateY(20px);opacity:0}to{transform:none;opacity:1}}
.mt{padding:16px 16px 8px;font-size:18px;font-weight:500;border:none;background:transparent;width:100%;outline:none;font-family:inherit;color:var(–text);resize:none;}
.mb{padding:8px 16px 16px;font-size:15px;line-height:1.7;border:none;background:transparent;width:100%;outline:none;font-family:inherit;color:var(–text);resize:none;flex:1;min-height:120px;}
.mf{padding:8px 12px;display:flex;justify-content:space-between;align-items:center;border-top:1px solid var(–border);gap:8px;flex-wrap:wrap;}

/* COPY */
.cop{position:fixed;inset:0;background:rgba(0,0,0,.45);z-index:300;display:flex;align-items:center;justify-content:center;padding:16px;}
.copb{background:#fff;border-radius:16px;padding:28px;max-width:340px;width:100%;box-shadow:var(–s2);text-align:center;animation:su .2s ease;}
.copb h3{font-size:18px;margin-bottom:6px;}
.copb p{font-size:13px;color:var(–muted);margin-bottom:20px;}
.copts{display:flex;flex-direction:column;gap:10px;}
.cob{
padding:12px;border-radius:10px;border:1px solid var(–border);background:#fff;
font-family:inherit;font-size:14px;font-weight:500;cursor:pointer;
display:flex;align-items:center;gap:10px;transition:background .15s;
}
.cob:hover{background:#f1f3f4;}
.cob.pr{background:var(–accent);color:#fff;border-color:var(–accent);}
.cob.pr:hover{background:#1557b0;}
.ccan{margin-top:12px;background:transparent;border:none;font-family:inherit;font-size:14px;color:var(–muted);cursor:pointer;padding:8px;}

/* EMPTY */
.empty{text-align:center;padding:60px 20px;color:var(–muted);}
.empty svg{opacity:.2;margin-bottom:16px;}
.empty p{font-size:16px;}

/* TOAST */
.toast{
position:fixed;bottom:24px;left:50%;transform:translateX(-50%);
background:#323232;color:#fff;padding:12px 24px;border-radius:4px;
font-size:14px;z-index:500;pointer-events:none;animation:ti .3s ease;
}
@keyframes ti{from{opacity:0;transform:translateX(-50%) translateY(8px)}to{opacity:1;transform:translateX(-50%) translateY(0)}}
::-webkit-scrollbar{width:6px;}
::-webkit-scrollbar-thumb{background:#ccc;border-radius:3px;}
</style>

</head>
<body>

<!-- SETUP -->

<div class="setup" id="setup">
  <div class="setup-box">
    <svg class="setup-logo" viewBox="0 0 60 60" fill="none">
      <rect width="60" height="60" rx="10" fill="#fbbc04"/>
      <rect x="15" y="15" width="30" height="6" rx="3" fill="white"/>
      <rect x="15" y="26" width="30" height="6" rx="3" fill="white"/>
      <rect x="15" y="37" width="20" height="6" rx="3" fill="white"/>
    </svg>
    <h2>Welcome to Keep</h2>
    <p>
      Choose a <strong>Room Name</strong> — any word or phrase.<br>
      Every device that uses the <strong>same room name</strong> sees<br>
      the exact same notes, updated live in real-time.
    </p>
    <input class="setup-input" id="roomInput" placeholder="e.g.  my-secret-notes-2025" autocomplete="off" spellcheck="false">
    <button class="setup-btn" onclick="joinRoom()">Open My Notes →</button>
    <p class="setup-hint">
      💡 Tip: type the same room name on your phone and Chromebook<br>
      — notes you write on one will instantly appear on the other.
    </p>
  </div>
</div>

<!-- APP -->

<header id="hdr" style="display:none">
  <div class="logo">
    <svg viewBox="0 0 36 36" fill="none">
      <rect width="36" height="36" rx="4" fill="#fbbc04"/>
      <rect x="9" y="9" width="18" height="3" rx="1.5" fill="white"/>
      <rect x="9" y="15" width="18" height="3" rx="1.5" fill="white"/>
      <rect x="9" y="21" width="12" height="3" rx="1.5" fill="white"/>
    </svg>
    <span class="logo-text" id="roomLbl">Keep</span>
  </div>
  <div class="search-wrap">
    <svg class="si" width="20" height="20" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round"><circle cx="11" cy="11" r="6"/><path d="m21 21-4.35-4.35"/></svg>
    <input type="text" id="searchInput" placeholder="Search your notes">
  </div>
  <div class="hright">
    <div class="sync-pill"><div class="dot" id="dot"></div><span id="slbl">Live</span></div>
    <button class="ib" onclick="changeRoom()" title="Change room">
      <svg width="18" height="18" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round"><circle cx="12" cy="12" r="3"/><path d="M19.07 4.93a10 10 0 0 1 0 14.14M4.93 4.93a10 10 0 0 0 0 14.14"/></svg>
    </button>
  </div>
</header>

<main id="app" style="display:none">
  <div class="cw">
    <div class="composer">
      <input type="text" class="ct" id="cTitle" placeholder="Title" autocomplete="off">
      <textarea class="cb" id="cBody" placeholder="Take a note…" rows="1"></textarea>
      <div class="crow" id="cColors"></div>
      <div class="cf" id="cFoot">
        <div></div>
        <button class="btn ghost" onclick="closeComposer()">Close</button>
      </div>
    </div>
  </div>

  <div id="pSec" style="display:none">
    <div class="slbl">Pinned</div>
    <div class="grid" id="pGrid"></div>
  </div>
  <div id="oSec">
    <div class="slbl" id="oLbl" style="display:none">Others</div>
    <div class="grid" id="nGrid"></div>
    <div class="empty" id="emptyEl">
      <svg width="96" height="96" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="1"><path d="M12 20h9M16.5 3.5a2.121 2.121 0 0 1 3 3L7 19l-4 1 1-4z"/></svg>
      <p>Notes you add appear here</p>
    </div>
  </div>
</main>

<!-- EDIT MODAL -->

<div class="ov" id="editModal" style="display:none" onclick="ovClick(event)">
  <div class="modal" id="modalBox">
    <textarea class="mt" id="mTitle" placeholder="Title" rows="1"></textarea>
    <textarea class="mb" id="mBody" placeholder="Note…"></textarea>
    <div class="crow" id="mColors"></div>
    <div class="mf">
      <div style="display:flex;gap:4px">
        <button class="ib" onclick="openCopy(curId)" title="Copy">
          <svg width="18" height="18" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round"><rect x="9" y="9" width="13" height="13" rx="2"/><path d="M5 15H4a2 2 0 0 1-2-2V4a2 2 0 0 1 2-2h9a2 2 0 0 1 2 2v1"/></svg>
        </button>
        <button class="ib" onclick="delNote(curId)" title="Delete">
          <svg width="18" height="18" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2"><polyline points="3 6 5 6 21 6"/><path d="M19 6l-1 14a2 2 0 0 1-2 2H8a2 2 0 0 1-2-2L5 6"/><path d="M10 11v6M14 11v6"/></svg>
        </button>
      </div>
      <button class="btn ghost" onclick="closeModal()">Close</button>
    </div>
  </div>
</div>

<!-- COPY DIALOG -->

<div class="cop" id="copyDlg" style="display:none">
  <div class="copb">
    <h3>📋 Copy Note</h3>
    <p>How do you want to copy this note?</p>
    <div class="copts">
      <button class="cob pr" onclick="doCopyClip()">
        <svg width="18" height="18" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2"><rect x="9" y="9" width="13" height="13" rx="2"/><path d="M5 15H4a2 2 0 0 1-2-2V4a2 2 0 0 1 2-2h9a2 2 0 0 1 2 2v1"/></svg>
        Copy to Clipboard
      </button>
      <button class="cob" onclick="doSaveFile('txt')">
        <svg width="18" height="18" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2"><path d="M14 2H6a2 2 0 0 0-2 2v16a2 2 0 0 0 2 2h12a2 2 0 0 0 2-2V8z"/><polyline points="14 2 14 8 20 8"/><line x1="16" y1="13" x2="8" y2="13"/><line x1="16" y1="17" x2="8" y2="17"/></svg>
        Save as .txt → Files
      </button>
      <button class="cob" onclick="doSaveFile('md')">
        <svg width="18" height="18" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2"><path d="M14 2H6a2 2 0 0 0-2 2v16a2 2 0 0 0 2 2h12a2 2 0 0 0 2-2V8z"/><polyline points="14 2 14 8 20 8"/></svg>
        Save as .md → Files
      </button>
    </div>
    <button class="ccan" onclick="closeCopy()">Cancel</button>
  </div>
</div>

<script>
// ─── FIREBASE ─────────────────────────────────────────────────────────────
// Free public Realtime Database — notes stored by room name you choose
const FB = {
  apiKey:"AIzaSyC2NeNOxl43K3POJuYJEGQByvGpZPkFxRM",
  authDomain:"keepclone-realtime.firebaseapp.com",
  databaseURL:"https://keepclone-realtime-default-rtdb.firebaseio.com",
  projectId:"keepclone-realtime",
};
firebase.initializeApp(FB);
const db = firebase.database();

// ─── COLORS ───────────────────────────────────────────────────────────────
const COLORS=[
  {id:null,hex:null},{id:'y',hex:'#fff9c4'},{id:'g',hex:'#ccff90'},
  {id:'t',hex:'#a7ffeb'},{id:'b',hex:'#aecbfa'},{id:'p',hex:'#d7aefb'},
  {id:'pk',hex:'#fbcfe8'},{id:'r',hex:'#f28b82'},{id:'o',hex:'#fbbc04'},
  {id:'gr',hex:'#e8eaed'},
];
const hex=id=>COLORS.find(c=>c.id===id)?.hex||'';

// ─── STATE ────────────────────────────────────────────────────────────────
let notes={}, room=null, curId=null, copyId=null, cColor=null;
let composerOpen=false, saveT=null;

// ─── STARTUP ──────────────────────────────────────────────────────────────
window.onload=()=>{
  const saved=localStorage.getItem('keep_room')||'';
  if(saved) document.getElementById('roomInput').value=saved;
  document.getElementById('roomInput').onkeydown=e=>{ if(e.key==='Enter') joinRoom(); };
  buildSw('cColors',null,c=>{ cColor=c; });
};

// ─── ROOM ─────────────────────────────────────────────────────────────────
function joinRoom(){
  const raw=document.getElementById('roomInput').value.trim();
  if(!raw){ showToast('Enter a room name first'); return; }
  room=raw.replace(/[.#$[\]/]/g,'-');
  localStorage.setItem('keep_room',raw);
  document.getElementById('setup').style.display='none';
  document.getElementById('hdr').style.display='flex';
  document.getElementById('app').style.display='block';
  document.getElementById('roomLbl').textContent='/ '+room;
  startListen();
  setupComposer();
  document.getElementById('searchInput').oninput=e=>{
    let t; clearTimeout(t); t=setTimeout(()=>render(e.target.value),150);
  };
}

function changeRoom(){
  if(!confirm('Switch to a different room?')) return;
  db.ref('rooms/'+room).off();
  notes={};
  document.getElementById('hdr').style.display='none';
  document.getElementById('app').style.display='none';
  document.getElementById('setup').style.display='flex';
}

// ─── FIREBASE LISTENER ────────────────────────────────────────────────────
function startListen(){
  setStat('pulsing','Connecting…');
  db.ref('rooms/'+room).on('value', snap=>{
    notes=snap.val()||{};
    render();
    setStat('','Live');
  }, ()=>setStat('off','Offline'));
}

function push(n){ setStat('pulsing','Saving…'); db.ref('rooms/'+room+'/'+n.id).set(n); }
function drop(id){ db.ref('rooms/'+room+'/'+id).remove(); }

// ─── CRUD ─────────────────────────────────────────────────────────────────
function newNote(title,body,color){
  if(!title.trim()&&!body.trim()) return;
  push({id:Date.now().toString(),title:title.trim(),body:body.trim(),color:color||null,pinned:false,ts:Date.now()});
}
function updNote(id,ch){
  if(!id) return;
  push({...(notes[id]||{}), ...ch, id, ts:Date.now()});
}
function delNote(id){ drop(id); closeModal(); showToast('Note deleted'); }
function pinNote(id){ updNote(id,{pinned:!(notes[id]?.pinned)}); }

// ─── RENDER ───────────────────────────────────────────────────────────────
function render(q=''){
  const all=Object.values(notes).sort((a,b)=>b.ts-a.ts);
  const vis=q?all.filter(n=>(n.title+n.body).toLowerCase().includes(q.toLowerCase())):all;
  const P=vis.filter(n=>n.pinned), O=vis.filter(n=>!n.pinned);
  document.getElementById('pSec').style.display=P.length?'':'none';
  document.getElementById('oLbl').style.display=P.length&&O.length?'':'none';
  document.getElementById('pGrid').innerHTML=P.map(cardHtml).join('');
  document.getElementById('nGrid').innerHTML=O.map(cardHtml).join('');
  document.getElementById('emptyEl').style.display=!vis.length?'':'none';
}

function cardHtml(n){
  const h=hex(n.color);
  const st=h?`style="background:${h};border-color:${h}"`:'';
  return`<div class="card${n.pinned?' pin':''}" ${st} onclick="openNote('${n.id}')">
  <button class="pb" onclick="event.stopPropagation();pinNote('${n.id}')" title="${n.pinned?'Unpin':'Pin'}">
    <svg width="16" height="16" viewBox="0 0 24 24" fill="${n.pinned?'currentColor':'none'}" stroke="currentColor" stroke-width="2">
      <path d="M12 2l3 7h5l-4 5 1.5 7L12 17l-5.5 4L8 14 4 9h5z"/>
    </svg>
  </button>
  ${n.title?`<div class="ct2">${esc(n.title)}</div>`:''}
  <div class="cb2">${esc(n.body)}</div>
  <div class="cft">
    <div class="ca">
      <button class="ib" onclick="event.stopPropagation();openCopy('${n.id}')" title="Copy">
        <svg width="15" height="15" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round"><rect x="9" y="9" width="13" height="13" rx="2"/><path d="M5 15H4a2 2 0 0 1-2-2V4a2 2 0 0 1 2-2h9a2 2 0 0 1 2 2v1"/></svg>
      </button>
      <button class="ib" onclick="event.stopPropagation();delNote('${n.id}')" title="Delete">
        <svg width="15" height="15" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2"><polyline points="3 6 5 6 21 6"/><path d="M19 6l-1 14a2 2 0 0 1-2 2H8a2 2 0 0 1-2-2L5 6"/></svg>
      </button>
    </div>
    <span class="ctm">${relT(n.ts)}</span>
  </div>
</div>`;
}

function esc(s){return(s||'').replace(/&/g,'&amp;').replace(/</g,'&lt;').replace(/>/g,'&gt;').replace(/\n/g,'<br>');}
function relT(ts){const d=Date.now()-ts;if(d<60000)return'Just now';if(d<3600000)return Math.floor(d/60000)+'m ago';if(d<86400000)return Math.floor(d/3600000)+'h ago';return new Date(ts).toLocaleDateString();}

// ─── COMPOSER ────────────────────────────────────────────────────────────
function setupComposer(){
  const b=document.getElementById('cBody');
  const t=document.getElementById('cTitle');
  b.onfocus=()=>{
    composerOpen=true;
    document.getElementById('cTitle').classList.add('show');
    document.getElementById('cFoot').classList.add('show');
    b.style.minHeight='80px';
  };
  [b,t].forEach(el=>el.oninput=()=>aH(el));
}
function aH(el){el.style.height='auto';el.style.height=el.scrollHeight+'px';}
function closeComposer(){
  const t=document.getElementById('cTitle').value;
  const b=document.getElementById('cBody').value;
  newNote(t,b,cColor);
  document.getElementById('cTitle').value='';
  document.getElementById('cBody').value='';
  document.getElementById('cTitle').classList.remove('show');
  document.getElementById('cFoot').classList.remove('show');
  document.getElementById('cBody').style.minHeight='48px';
  cColor=null; composerOpen=false;
  buildSw('cColors',null,c=>{cColor=c;});
}

// ─── SWATCHES ─────────────────────────────────────────────────────────────
function buildSw(id,sel,cb){
  const w=document.getElementById(id);
  if(!w)return;
  w.innerHTML=COLORS.map(c=>`<div class="sw${c.id?'':' none'}${sel===c.id?' sel':''}" style="${c.hex?'background:'+c.hex:''}" title="${c.id||'Default'}" onclick="pickSw('${id}','${c.id||''}')"></div>`).join('');
  w._cb=cb;
}
function pickSw(id,colorId){
  const w=document.getElementById(id);
  const cid=colorId||null;
  w.querySelectorAll('.sw').forEach((el,i)=>el.classList.toggle('sel',COLORS[i].id===cid));
  if(w._cb) w._cb(cid);
  if(id==='mColors'&&curId){
    document.getElementById('modalBox').style.background=hex(cid)||'#fff';
    updNote(curId,{color:cid});
  }
}

// ─── MODAL ────────────────────────────────────────────────────────────────
function openNote(id){
  const n=notes[id]; if(!n)return;
  curId=id;
  document.getElementById('mTitle').value=n.title||'';
  document.getElementById('mBody').value=n.body||'';
  document.getElementById('modalBox').style.background=hex(n.color)||'#fff';
  buildSw('mColors',n.color,c=>{
    document.getElementById('modalBox').style.background=hex(c)||'#fff';
    if(curId) updNote(curId,{color:c});
  });
  document.getElementById('editModal').style.display='flex';
  [document.getElementById('mTitle'),document.getElementById('mBody')].forEach(el=>{
    aH(el); el.oninput=()=>{aH(el);schedSave();};
  });
  document.getElementById('mBody').focus();
}
function schedSave(){
  clearTimeout(saveT);
  saveT=setTimeout(()=>{
    if(!curId)return;
    updNote(curId,{title:document.getElementById('mTitle').value,body:document.getElementById('mBody').value});
  },700);
}
function closeModal(){
  if(curId) updNote(curId,{title:document.getElementById('mTitle').value,body:document.getElementById('mBody').value});
  document.getElementById('editModal').style.display='none';
  curId=null;
}
function ovClick(e){if(e.target.id==='editModal')closeModal();}

// ─── COPY ─────────────────────────────────────────────────────────────────
function openCopy(id){copyId=id;document.getElementById('copyDlg').style.display='flex';}
function closeCopy(){document.getElementById('copyDlg').style.display='none';copyId=null;}
function getNoteText(){const n=notes[copyId];if(!n)return'';return(n.title?n.title+'\n\n':'')+(n.body||'');}

function doCopyClip(){
  const txt=getNoteText();
  navigator.clipboard.writeText(txt).catch(()=>{const ta=document.createElement('textarea');ta.value=txt;document.body.appendChild(ta);ta.select();document.execCommand('copy');document.body.removeChild(ta);});
  closeCopy();showToast('Copied to clipboard!');
}
function doSaveFile(ext){
  const n=notes[copyId];if(!n)return;
  const txt=ext==='md'?(n.title?`# ${n.title}\n\n`:'')+(n.body||''):(n.title?n.title+'\n'+'─'.repeat(n.title.length)+'\n\n':'')+(n.body||'');
  const blob=new Blob([txt],{type:'text/plain'});
  const url=URL.createObjectURL(blob);
  const a=document.createElement('a');
  const name=(n.title||'note').replace(/[^a-z0-9]/gi,'_').substring(0,40);
  a.href=url;a.download=`${name}.${ext}`;
  document.body.appendChild(a);a.click();document.body.removeChild(a);URL.revokeObjectURL(url);
  closeCopy();showToast(`Saved ${name}.${ext} to Downloads`);
}

// ─── SYNC STATUS ──────────────────────────────────────────────────────────
function setStat(cls,txt){
  document.getElementById('dot').className='dot'+(cls?' '+cls:'');
  document.getElementById('slbl').textContent=txt;
}

// ─── TOAST ────────────────────────────────────────────────────────────────
let tT;
function showToast(m){
  clearTimeout(tT);document.querySelectorAll('.toast').forEach(e=>e.remove());
  const t=document.createElement('div');t.className='toast';t.textContent=m;
  document.body.appendChild(t);tT=setTimeout(()=>t.remove(),2500);
}

// ─── KEYBOARD ─────────────────────────────────────────────────────────────
document.addEventListener('keydown',e=>{
  if(e.key!=='Escape')return;
  if(document.getElementById('copyDlg').style.display!=='none'){closeCopy();return;}
  if(document.getElementById('editModal').style.display!=='none'){closeModal();return;}
  if(composerOpen)closeComposer();
});
document.addEventListener('click',e=>{
  const c=document.getElementById('cBody')?.closest('.composer');
  if(composerOpen&&c&&!c.contains(e.target))closeComposer();
});
</script>

</body>
</html>