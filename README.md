<!DOCTYPE html>

<html lang="en">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>Keep — My Notes</title>
<link href="https://fonts.googleapis.com/css2?family=Product+Sans:wght@400;500;700&family=Google+Sans:wght@400;500&display=swap" rel="stylesheet">
<style>
  :root {
    --yellow: #fbbc04;
    --bg: #f5f5f5;
    --surface: #ffffff;
    --border: #e0e0e0;
    --text-primary: #202124;
    --text-secondary: #5f6368;
    --accent: #1a73e8;
    --hover: rgba(0,0,0,0.06);
    --shadow: 0 1px 3px rgba(0,0,0,0.12), 0 1px 2px rgba(0,0,0,0.08);
    --shadow-hover: 0 4px 12px rgba(0,0,0,0.15);
    --radius: 8px;
  }
  * { box-sizing: border-box; margin: 0; padding: 0; }
  body {
    font-family: 'Google Sans', 'Product Sans', sans-serif;
    background: var(--bg);
    color: var(--text-primary);
    min-height: 100vh;
  }

/* HEADER */
header {
position: sticky; top: 0; z-index: 100;
background: var(–surface);
border-bottom: 1px solid var(–border);
display: flex; align-items: center;
padding: 0 16px; height: 64px;
gap: 12px;
}
.header-logo {
display: flex; align-items: center; gap: 8px;
text-decoration: none;
}
.header-logo svg { width: 36px; height: 36px; }
.header-logo span {
font-size: 22px; color: var(–text-secondary);
font-weight: 400; letter-spacing: -0.3px;
}
.header-search {
flex: 1; max-width: 720px; margin: 0 auto;
position: relative;
}
.header-search input {
width: 100%; padding: 10px 16px 10px 44px;
background: #f1f3f4; border: none; border-radius: 24px;
font-size: 16px; font-family: inherit; color: var(–text-primary);
outline: none; transition: background 0.2s, box-shadow 0.2s;
}
.header-search input:focus {
background: white;
box-shadow: 0 2px 8px rgba(0,0,0,0.2);
}
.header-search .search-icon {
position: absolute; left: 12px; top: 50%; transform: translateY(-50%);
color: var(–text-secondary);
}
.header-actions { display: flex; gap: 4px; }
.icon-btn {
width: 40px; height: 40px; border-radius: 50%;
border: none; background: transparent; cursor: pointer;
display: flex; align-items: center; justify-content: center;
color: var(–text-secondary); transition: background 0.15s;
}
.icon-btn:hover { background: var(–hover); }
.sync-status {
font-size: 12px; color: var(–text-secondary);
display: flex; align-items: center; gap: 4px;
white-space: nowrap;
}
.sync-dot {
width: 8px; height: 8px; border-radius: 50%;
background: #34a853; transition: background 0.3s;
}
.sync-dot.syncing { background: var(–yellow); animation: pulse 1s infinite; }
.sync-dot.error { background: #ea4335; }
@keyframes pulse { 0%,100%{opacity:1} 50%{opacity:0.4} }

/* MAIN LAYOUT */
main { max-width: 1200px; margin: 0 auto; padding: 24px 16px; }

/* NOTE COMPOSER */
.composer-wrapper {
max-width: 600px; margin: 0 auto 32px;
}
.composer {
background: var(–surface);
border: 1px solid var(–border);
border-radius: var(–radius);
box-shadow: var(–shadow);
overflow: hidden;
transition: box-shadow 0.2s;
}
.composer:focus-within { box-shadow: var(–shadow-hover); }
.composer-title {
padding: 12px 16px 4px;
font-size: 16px; font-weight: 500;
border: none; background: transparent; width: 100%;
outline: none; font-family: inherit; color: var(–text-primary);
display: none;
}
.composer-title.visible { display: block; }
.composer-body {
padding: 12px 16px;
font-size: 14px; line-height: 1.6;
border: none; background: transparent; width: 100%;
outline: none; font-family: inherit; color: var(–text-primary);
resize: none; min-height: 48px; max-height: 80vh;
overflow-y: auto;
}
.composer-actions {
display: none; padding: 4px 8px 8px;
justify-content: space-between; align-items: center;
}
.composer-actions.visible { display: flex; }
.btn {
padding: 8px 24px; border-radius: 4px; border: none;
font-family: inherit; font-size: 14px; font-weight: 500;
cursor: pointer; transition: background 0.15s;
}
.btn-close {
background: transparent; color: var(–text-secondary);
}
.btn-close:hover { background: var(–hover); }

/* COLOR PICKER */
.color-row {
display: flex; gap: 6px; padding: 4px 8px;
flex-wrap: wrap;
}
.color-swatch {
width: 28px; height: 28px; border-radius: 50%;
cursor: pointer; border: 2px solid transparent;
transition: transform 0.15s, border-color 0.15s;
position: relative;
}
.color-swatch:hover { transform: scale(1.15); }
.color-swatch.selected { border-color: #444; }
.color-swatch.default { background: white; border-color: #ddd; }
.color-swatch.default::after {
content: ‘✕’; position: absolute; top: 50%; left: 50%;
transform: translate(-50%,-50%); font-size: 10px; color: #bbb;
}

/* NOTES GRID */
.section-label {
font-size: 11px; font-weight: 500; letter-spacing: 0.8px;
color: var(–text-secondary); text-transform: uppercase;
margin-bottom: 12px; padding: 0 4px;
}
.notes-grid {
columns: 5 180px; column-gap: 12px;
}
@media(max-width:600px){ .notes-grid { columns: 2 140px; } }

/* NOTE CARD */
.note-card {
break-inside: avoid; margin-bottom: 12px;
background: var(–surface);
border: 1px solid var(–border);
border-radius: var(–radius);
box-shadow: var(–shadow);
transition: box-shadow 0.2s, transform 0.15s;
position: relative; cursor: default;
animation: noteIn 0.2s ease;
}
@keyframes noteIn {
from { opacity:0; transform: scale(0.96) translateY(4px); }
to   { opacity:1; transform: scale(1) translateY(0); }
}
.note-card:hover {
box-shadow: var(–shadow-hover);
transform: translateY(-1px);
}
.note-card:hover .note-actions { opacity: 1; }
.note-title {
padding: 12px 12px 4px;
font-size: 14px; font-weight: 500;
word-break: break-word;
}
.note-body {
padding: 4px 12px 12px;
font-size: 13px; line-height: 1.6;
color: var(–text-secondary);
word-break: break-word;
white-space: pre-wrap;
}
.note-body.expanded { max-height: none; }
.note-footer {
padding: 4px 4px 4px;
display: flex; justify-content: space-between; align-items: center;
}
.note-actions {
opacity: 0; display: flex; gap: 2px;
transition: opacity 0.15s;
}
.note-time {
font-size: 11px; color: #bbb;
padding: 0 8px;
}
.note-pin {
position: absolute; top: 6px; right: 6px;
background: transparent; border: none;
cursor: pointer; padding: 4px; border-radius: 50%;
opacity: 0; transition: opacity 0.15s;
color: var(–text-secondary);
}
.note-card:hover .note-pin,
.note-card.pinned .note-pin { opacity: 1; }
.note-card.pinned .note-pin { color: var(–text-primary); }
.note-card.pinned { border-top: 3px solid var(–yellow); }

/* MODAL */
.modal-overlay {
position: fixed; inset: 0;
background: rgba(0,0,0,0.5);
z-index: 200; display: flex;
align-items: center; justify-content: center;
padding: 16px;
animation: fadeIn 0.15s ease;
}
@keyframes fadeIn { from{opacity:0} to{opacity:1} }
.modal {
background: var(–surface);
border-radius: var(–radius);
width: 100%; max-width: 600px;
max-height: 90vh; overflow-y: auto;
box-shadow: 0 8px 32px rgba(0,0,0,0.25);
animation: slideUp 0.2s ease;
display: flex; flex-direction: column;
}
@keyframes slideUp { from{transform:translateY(20px);opacity:0} to{transform:translateY(0);opacity:1} }
.modal-title {
padding: 16px 16px 8px;
font-size: 18px; font-weight: 500;
border: none; background: transparent; width: 100%;
outline: none; font-family: inherit; color: var(–text-primary);
resize: none;
}
.modal-body {
padding: 8px 16px 16px;
font-size: 15px; line-height: 1.7;
border: none; background: transparent; width: 100%;
outline: none; font-family: inherit; color: var(–text-primary);
resize: none; flex: 1; min-height: 120px;
}
.modal-footer {
padding: 8px 12px;
display: flex; justify-content: space-between; align-items: center;
border-top: 1px solid var(–border);
gap: 8px; flex-wrap: wrap;
}

/* COPY DIALOG */
.copy-dialog {
position: fixed; inset: 0;
background: rgba(0,0,0,0.4);
z-index: 300; display: flex;
align-items: center; justify-content: center;
padding: 16px;
}
.copy-box {
background: white; border-radius: 12px;
padding: 24px; max-width: 340px; width: 100%;
box-shadow: 0 8px 32px rgba(0,0,0,0.25);
text-align: center;
animation: slideUp 0.2s ease;
}
.copy-box h3 { font-size: 18px; margin-bottom: 8px; }
.copy-box p { font-size: 14px; color: var(–text-secondary); margin-bottom: 20px; }
.copy-options { display: flex; flex-direction: column; gap: 10px; }
.copy-opt-btn {
padding: 12px; border-radius: 8px; border: 1px solid var(–border);
background: white; font-family: inherit; font-size: 14px;
font-weight: 500; cursor: pointer; display: flex;
align-items: center; gap: 10px; transition: background 0.15s;
}
.copy-opt-btn:hover { background: #f1f3f4; }
.copy-opt-btn.primary { background: var(–accent); color: white; border-color: var(–accent); }
.copy-opt-btn.primary:hover { background: #1557b0; }
.copy-cancel {
margin-top: 12px; background: transparent; border: none;
font-family: inherit; font-size: 14px; color: var(–text-secondary);
cursor: pointer; padding: 8px;
}

/* EMPTY STATE */
.empty-state {
text-align: center; padding: 60px 20px;
color: var(–text-secondary);
}
.empty-state svg { opacity: 0.3; margin-bottom: 16px; }
.empty-state p { font-size: 16px; }

/* TOAST */
.toast {
position: fixed; bottom: 24px; left: 50%; transform: translateX(-50%);
background: #323232; color: white; padding: 12px 24px;
border-radius: 4px; font-size: 14px; z-index: 400;
animation: toastIn 0.3s ease;
pointer-events: none;
}
@keyframes toastIn { from{opacity:0;transform:translateX(-50%) translateY(8px)} to{opacity:1;transform:translateX(-50%) translateY(0)} }

/* LABEL / COLOR MAP */
.note-label {
display: inline-block; padding: 2px 8px;
border-radius: 12px; font-size: 11px;
background: #f1f3f4; color: var(–text-secondary);
margin: 4px 12px;
}

/* Scrollbar */
::-webkit-scrollbar { width: 6px; }
::-webkit-scrollbar-track { background: transparent; }
::-webkit-scrollbar-thumb { background: #ccc; border-radius: 3px; }
</style>

</head>
<body>

<!-- HEADER -->

<header>
  <div class="header-logo">
    <svg viewBox="0 0 36 36" fill="none">
      <rect width="36" height="36" rx="4" fill="#fbbc04"/>
      <rect x="9" y="9" width="18" height="3" rx="1.5" fill="white"/>
      <rect x="9" y="15" width="18" height="3" rx="1.5" fill="white"/>
      <rect x="9" y="21" width="12" height="3" rx="1.5" fill="white"/>
    </svg>
    <span>Keep</span>
  </div>
  <div class="header-search">
    <svg class="search-icon" width="20" height="20" viewBox="0 0 24 24" fill="currentColor">
      <path d="M21 21l-4.35-4.35M17 11A6 6 0 1 1 5 11a6 6 0 0 1 12 0z" stroke="currentColor" stroke-width="2" fill="none" stroke-linecap="round"/>
    </svg>
    <input type="text" id="searchInput" placeholder="Search your notes">
  </div>
  <div class="header-actions">
    <div class="sync-status" id="syncStatus">
      <div class="sync-dot" id="syncDot"></div>
      <span id="syncLabel">Synced</span>
    </div>
    <button class="icon-btn" onclick="syncNotes()" title="Sync now">
      <svg width="20" height="20" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round">
        <path d="M23 4v6h-6M1 20v-6h6"/>
        <path d="M3.51 9a9 9 0 0 1 14.85-3.36L23 10M1 14l4.64 4.36A9 9 0 0 0 20.49 15"/>
      </svg>
    </button>
  </div>
</header>

<main>
  <!-- COMPOSER -->
  <div class="composer-wrapper">
    <div class="composer" id="composer">
      <input type="text" class="composer-title" id="composerTitle" placeholder="Title" autocomplete="off">
      <textarea class="composer-body" id="composerBody" placeholder="Take a note…" rows="1"></textarea>
      <div class="color-row" id="composerColors"></div>
      <div class="composer-actions" id="composerActions">
        <div></div>
        <button class="btn btn-close" onclick="closeComposer()">Close</button>
      </div>
    </div>
  </div>

  <!-- PINNED -->

  <div id="pinnedSection" style="display:none">
    <div class="section-label">Pinned</div>
    <div class="notes-grid" id="pinnedGrid"></div>
  </div>

  <!-- OTHER -->

  <div id="othersSection">
    <div class="section-label" id="othersLabel" style="display:none">Others</div>
    <div class="notes-grid" id="notesGrid"></div>
    <div class="empty-state" id="emptyState">
      <svg width="120" height="120" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="1">
        <path d="M12 20h9M16.5 3.5a2.121 2.121 0 0 1 3 3L7 19l-4 1 1-4L16.5 3.5z"/>
      </svg>
      <p>Notes you add appear here</p>
    </div>
  </div>
</main>

<!-- EDIT MODAL -->

<div class="modal-overlay" id="editModal" style="display:none" onclick="handleModalClick(event)">
  <div class="modal" id="modalBox">
    <textarea class="modal-title" id="modalTitle" placeholder="Title" rows="1"></textarea>
    <textarea class="modal-body" id="modalBody" placeholder="Note…"></textarea>
    <div class="color-row" id="modalColors"></div>
    <div class="modal-footer">
      <div style="display:flex;gap:4px" id="modalActionBtns">
        <button class="icon-btn" onclick="openCopyDialog(currentEditId)" title="Copy">
          <svg width="18" height="18" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round">
            <rect x="9" y="9" width="13" height="13" rx="2"/><path d="M5 15H4a2 2 0 0 1-2-2V4a2 2 0 0 1 2-2h9a2 2 0 0 1 2 2v1"/>
          </svg>
        </button>
        <button class="icon-btn" onclick="deleteNote(currentEditId)" title="Delete">
          <svg width="18" height="18" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round">
            <polyline points="3 6 5 6 21 6"/><path d="M19 6l-1 14a2 2 0 0 1-2 2H8a2 2 0 0 1-2-2L5 6"/><path d="M10 11v6M14 11v6"/><path d="M9 6V4h6v2"/>
          </svg>
        </button>
      </div>
      <button class="btn btn-close" onclick="closeModal()">Close</button>
    </div>
  </div>
</div>

<!-- COPY DIALOG -->

<div class="copy-dialog" id="copyDialog" style="display:none">
  <div class="copy-box">
    <h3>📋 Copy Note</h3>
    <p>How would you like to copy this note?</p>
    <div class="copy-options">
      <button class="copy-opt-btn primary" onclick="copyToClipboard()">
        <svg width="20" height="20" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2"><rect x="9" y="9" width="13" height="13" rx="2"/><path d="M5 15H4a2 2 0 0 1-2-2V4a2 2 0 0 1 2-2h9a2 2 0 0 1 2 2v1"/></svg>
        Copy to Clipboard
      </button>
      <button class="copy-opt-btn" onclick="saveAsFile('txt')">
        <svg width="20" height="20" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2"><path d="M14 2H6a2 2 0 0 0-2 2v16a2 2 0 0 0 2 2h12a2 2 0 0 0 2-2V8z"/><polyline points="14 2 14 8 20 8"/><line x1="16" y1="13" x2="8" y2="13"/><line x1="16" y1="17" x2="8" y2="17"/></svg>
        Save as .txt File
      </button>
      <button class="copy-opt-btn" onclick="saveAsFile('md')">
        <svg width="20" height="20" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2"><path d="M14 2H6a2 2 0 0 0-2 2v16a2 2 0 0 0 2 2h12a2 2 0 0 0 2-2V8z"/><polyline points="14 2 14 8 20 8"/></svg>
        Save as .md File
      </button>
    </div>
    <button class="copy-cancel" onclick="closeCopyDialog()">Cancel</button>
  </div>
</div>

<script>
// ─── CONFIG ───────────────────────────────────────────────────────────────
const STORAGE_KEY = 'drake_keep_notes_v1';
const SYNC_KEY    = 'drake_keep_sync_v1';

// ─── STATE ────────────────────────────────────────────────────────────────
let notes = [];
let composerColor = null;
let currentEditId = null;
let copyTargetId  = null;
let composerOpen  = false;
let autoSaveTimer = null;

const COLORS = [
  { id: null,      label: 'Default', hex: null },
  { id: 'yellow',  label: 'Yellow',  hex: '#fff9c4' },
  { id: 'green',   label: 'Green',   hex: '#ccff90' },
  { id: 'teal',    label: 'Teal',    hex: '#a7ffeb' },
  { id: 'blue',    label: 'Blue',    hex: '#aecbfa' },
  { id: 'purple',  label: 'Purple',  hex: '#d7aefb' },
  { id: 'pink',    label: 'Pink',    hex: '#fbcfe8' },
  { id: 'red',     label: 'Red',     hex: '#f28b82' },
  { id: 'orange',  label: 'Orange',  hex: '#fbbc04' },
  { id: 'gray',    label: 'Gray',    hex: '#e8eaed' },
];

// ─── INIT ─────────────────────────────────────────────────────────────────
window.addEventListener('DOMContentLoaded', () => {
  buildColorPickers();
  loadNotes();
  setupComposer();
  setupSearch();
  setupAutoSync();
});

// ─── STORAGE / SYNC ───────────────────────────────────────────────────────
function saveNotes() {
  try {
    const data = { notes, updatedAt: Date.now() };
    // Use window.name as cross-tab shared store fallback
    // Primary: IndexedDB for large notes
    saveToIDB(data);
  } catch(e) { console.warn('save error', e); }
}

function loadNotes() {
  loadFromIDB().then(data => {
    if (data && data.notes) {
      notes = data.notes;
      renderAll();
    }
    setSyncStatus('synced');
  }).catch(() => {
    notes = [];
    renderAll();
  });
}

// IndexedDB for unlimited storage
function openDB() {
  return new Promise((res, rej) => {
    const req = indexedDB.open('KeepDB', 1);
    req.onupgradeneeded = e => {
      e.target.result.createObjectStore('notes', { keyPath: 'id' });
      e.target.result.createObjectStore('meta', { keyPath: 'id' });
    };
    req.onsuccess  = e => res(e.target.result);
    req.onerror    = e => rej(e);
  });
}

async function saveToIDB(data) {
  const db = await openDB();
  const tx = db.transaction('meta', 'readwrite');
  tx.objectStore('meta').put({ id: STORAGE_KEY, ...data });
  setSyncStatus('syncing');
  await new Promise(r => tx.oncomplete = r);
  setSyncStatus('synced');
}

async function loadFromIDB() {
  const db = await openDB();
  return new Promise((res, rej) => {
    const tx  = db.transaction('meta', 'readonly');
    const req = tx.objectStore('meta').get(STORAGE_KEY);
    req.onsuccess = e => res(e.target.result);
    req.onerror   = e => rej(e);
  });
}

// BroadcastChannel for same-device cross-tab sync
const bc = typeof BroadcastChannel !== 'undefined' ? new BroadcastChannel('keep_sync') : null;
if (bc) {
  bc.onmessage = e => {
    if (e.data && e.data.type === 'NOTES_UPDATED') {
      notes = e.data.notes;
      renderAll();
      setSyncStatus('synced');
    }
  };
}

function broadcastSync() {
  if (bc) bc.postMessage({ type: 'NOTES_UPDATED', notes });
}

function syncNotes() {
  setSyncStatus('syncing');
  saveToIDB({ notes, updatedAt: Date.now() }).then(() => {
    broadcastSync();
    showToast('Notes synced!');
  });
}

function setupAutoSync() {
  // Every 30 sec
  setInterval(() => saveToIDB({ notes, updatedAt: Date.now() }), 30000);
}

function setSyncStatus(state) {
  const dot   = document.getElementById('syncDot');
  const label = document.getElementById('syncLabel');
  dot.className = 'sync-dot' + (state === 'syncing' ? ' syncing' : state === 'error' ? ' error' : '');
  label.textContent = state === 'syncing' ? 'Syncing…' : state === 'error' ? 'Offline' : 'Synced';
}

// ─── NOTE CRUD ────────────────────────────────────────────────────────────
function createNote(title, body, color) {
  if (!title.trim() && !body.trim()) return;
  const note = {
    id:        Date.now().toString(),
    title:     title.trim(),
    body:      body.trim(),
    color:     color || null,
    pinned:    false,
    createdAt: Date.now(),
    updatedAt: Date.now(),
  };
  notes.unshift(note);
  saveNotes();
  broadcastSync();
  renderAll();
  return note;
}

function updateNote(id, changes) {
  const i = notes.findIndex(n => n.id === id);
  if (i < 0) return;
  notes[i] = { ...notes[i], ...changes, updatedAt: Date.now() };
  saveNotes();
  broadcastSync();
  renderAll();
}

function deleteNote(id) {
  notes = notes.filter(n => n.id !== id);
  saveNotes();
  broadcastSync();
  renderAll();
  closeModal();
  showToast('Note deleted');
}

function togglePin(id) {
  const note = notes.find(n => n.id === id);
  if (!note) return;
  updateNote(id, { pinned: !note.pinned });
}

// ─── RENDER ───────────────────────────────────────────────────────────────
function renderAll(filter = '') {
  const q       = filter.toLowerCase();
  const visible = q
    ? notes.filter(n => (n.title + n.body).toLowerCase().includes(q))
    : notes;
  const pinned  = visible.filter(n => n.pinned);
  const others  = visible.filter(n => !n.pinned);

  const pSec = document.getElementById('pinnedSection');
  const oLbl = document.getElementById('othersLabel');
  const pGrid = document.getElementById('pinnedGrid');
  const nGrid = document.getElementById('notesGrid');
  const empty = document.getElementById('emptyState');

  pSec.style.display = pinned.length ? '' : 'none';
  oLbl.style.display = pinned.length && others.length ? '' : 'none';

  pGrid.innerHTML = pinned.map(noteCard).join('');
  nGrid.innerHTML = others.map(noteCard).join('');
  empty.style.display = others.length === 0 && pinned.length === 0 ? '' : 'none';
}

function noteCard(n) {
  const bg    = n.color ? COLORS.find(c => c.id === n.color)?.hex || '' : '';
  const style = bg ? `background:${bg};border-color:${bg};` : '';
  const time  = relTime(n.updatedAt);
  return `
  <div class="note-card${n.pinned ? ' pinned':''}" id="card-${n.id}" style="${style}"
       onclick="openNote('${n.id}')">
    <button class="note-pin" onclick="event.stopPropagation();togglePin('${n.id}')" title="${n.pinned?'Unpin':'Pin'}">
      <svg width="16" height="16" viewBox="0 0 24 24" fill="${n.pinned?'currentColor':'none'}" stroke="currentColor" stroke-width="2">
        <path d="M12 2l3 7h5l-4 5 1.5 7L12 17l-5.5 4L8 14 4 9h5z"/>
      </svg>
    </button>
    ${n.title ? `<div class="note-title">${esc(n.title)}</div>` : ''}
    <div class="note-body">${esc(n.body)}</div>
    <div class="note-footer">
      <div class="note-actions">
        <button class="icon-btn" onclick="event.stopPropagation();openCopyDialog('${n.id}')" title="Copy">
          <svg width="16" height="16" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round">
            <rect x="9" y="9" width="13" height="13" rx="2"/><path d="M5 15H4a2 2 0 0 1-2-2V4a2 2 0 0 1 2-2h9a2 2 0 0 1 2 2v1"/>
          </svg>
        </button>
        <button class="icon-btn" onclick="event.stopPropagation();deleteNote('${n.id}')" title="Delete">
          <svg width="16" height="16" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2">
            <polyline points="3 6 5 6 21 6"/><path d="M19 6l-1 14a2 2 0 0 1-2 2H8a2 2 0 0 1-2-2L5 6"/><path d="M10 11v6M14 11v6"/>
          </svg>
        </button>
      </div>
      <span class="note-time">${time}</span>
    </div>
  </div>`;
}

function esc(s) {
  return (s || '').replace(/&/g,'&amp;').replace(/</g,'&lt;').replace(/>/g,'&gt;').replace(/"/g,'&quot;');
}

function relTime(ts) {
  const diff = Date.now() - ts;
  if (diff < 60000)     return 'Just now';
  if (diff < 3600000)   return Math.floor(diff/60000) + 'm ago';
  if (diff < 86400000)  return Math.floor(diff/3600000) + 'h ago';
  if (diff < 604800000) return Math.floor(diff/86400000) + 'd ago';
  return new Date(ts).toLocaleDateString();
}

// ─── COMPOSER ─────────────────────────────────────────────────────────────
function setupComposer() {
  const body    = document.getElementById('composerBody');
  const title   = document.getElementById('composerTitle');
  const actions = document.getElementById('composerActions');

  body.addEventListener('focus', () => {
    composerOpen = true;
    title.classList.add('visible');
    actions.classList.add('visible');
    body.style.minHeight = '80px';
  });

  // Auto-resize
  [body, title].forEach(el => el.addEventListener('input', () => autoResize(el)));
}

function autoResize(el) {
  el.style.height = 'auto';
  el.style.height = el.scrollHeight + 'px';
}

function closeComposer() {
  const title  = document.getElementById('composerTitle').value;
  const body   = document.getElementById('composerBody').value;
  if (title.trim() || body.trim()) {
    createNote(title, body, composerColor);
  }
  document.getElementById('composerTitle').value = '';
  document.getElementById('composerBody').value  = '';
  document.getElementById('composerTitle').classList.remove('visible');
  document.getElementById('composerActions').classList.remove('visible');
  document.getElementById('composerBody').style.minHeight = '48px';
  composerColor = null;
  composerOpen  = false;
  updateColorPicker('composerColors', composerColor, (c) => { composerColor = c; });
}

// ─── COLOR PICKERS ────────────────────────────────────────────────────────
function buildColorPickers() {
  updateColorPicker('composerColors', null, (c) => { composerColor = c; });
  updateColorPicker('modalColors',    null, (c) => {
    if (currentEditId) updateNote(currentEditId, { color: c });
  });
}

function updateColorPicker(containerId, selected, onSelect) {
  const wrap = document.getElementById(containerId);
  if (!wrap) return;
  wrap.innerHTML = COLORS.map(c => `
    <div class="color-swatch ${c.id ? '' : 'default'}${selected === c.id ? ' selected' : ''}"
         style="${c.hex ? 'background:'+c.hex : ''}"
         title="${c.label}"
         onclick="handleColorPick('${containerId}','${c.id||''}')">
    </div>
  `).join('');
  wrap._onSelect = onSelect;
}

function handleColorPick(containerId, colorId) {
  const wrap = document.getElementById(containerId);
  const id   = colorId || null;
  wrap.querySelectorAll('.color-swatch').forEach((el, i) => {
    el.classList.toggle('selected', COLORS[i].id === id);
  });
  if (wrap._onSelect) wrap._onSelect(id);

  // Update modal background
  if (containerId === 'modalColors' && currentEditId) {
    const hex = COLORS.find(c => c.id === id)?.hex || '';
    document.getElementById('modalBox').style.background = hex || 'white';
    updateNote(currentEditId, { color: id });
  }
}

// ─── EDIT MODAL ───────────────────────────────────────────────────────────
function openNote(id) {
  const note = notes.find(n => n.id === id);
  if (!note) return;
  currentEditId = id;

  const mTitle = document.getElementById('modalTitle');
  const mBody  = document.getElementById('modalBody');
  mTitle.value = note.title;
  mBody.value  = note.body;

  const hex = note.color ? COLORS.find(c => c.id === note.color)?.hex || '' : '';
  document.getElementById('modalBox').style.background = hex || 'white';

  updateColorPicker('modalColors', note.color, (c) => {
    if (currentEditId) {
      const h = COLORS.find(x => x.id === c)?.hex || '';
      document.getElementById('modalBox').style.background = h || 'white';
      updateNote(currentEditId, { color: c });
    }
  });

  document.getElementById('editModal').style.display = 'flex';

  // Auto-save on type
  [mTitle, mBody].forEach(el => {
    el.addEventListener('input', scheduleModalSave);
    autoResize(el);
    el.addEventListener('input', () => autoResize(el));
  });

  mBody.focus();
}

function scheduleModalSave() {
  clearTimeout(autoSaveTimer);
  autoSaveTimer = setTimeout(() => {
    if (!currentEditId) return;
    const title = document.getElementById('modalTitle').value;
    const body  = document.getElementById('modalBody').value;
    updateNote(currentEditId, { title, body });
  }, 600);
}

function closeModal() {
  // Final save
  if (currentEditId) {
    const title = document.getElementById('modalTitle').value;
    const body  = document.getElementById('modalBody').value;
    updateNote(currentEditId, { title, body });
  }
  document.getElementById('editModal').style.display = 'none';
  currentEditId = null;
  document.getElementById('modalTitle').removeEventListener('input', scheduleModalSave);
  document.getElementById('modalBody').removeEventListener('input', scheduleModalSave);
}

function handleModalClick(e) {
  if (e.target === document.getElementById('editModal')) closeModal();
}

// ─── COPY / SAVE ──────────────────────────────────────────────────────────
function openCopyDialog(id) {
  copyTargetId = id;
  document.getElementById('copyDialog').style.display = 'flex';
}

function closeCopyDialog() {
  document.getElementById('copyDialog').style.display = 'none';
  copyTargetId = null;
}

function getNoteText(id) {
  const note = notes.find(n => n.id === id || n.id === copyTargetId);
  if (!note) return '';
  let text = '';
  if (note.title) text += note.title + '\n\n';
  text += note.body;
  return text;
}

function copyToClipboard() {
  const text = getNoteText(copyTargetId);
  navigator.clipboard.writeText(text).then(() => {
    closeCopyDialog();
    showToast('Copied to clipboard!');
  }).catch(() => {
    // fallback
    const ta = document.createElement('textarea');
    ta.value = text; document.body.appendChild(ta);
    ta.select(); document.execCommand('copy');
    document.body.removeChild(ta);
    closeCopyDialog();
    showToast('Copied!');
  });
}

function saveAsFile(ext) {
  const note = notes.find(n => n.id === copyTargetId);
  if (!note) return;
  let text = '';
  if (ext === 'md') {
    if (note.title) text += `# ${note.title}\n\n`;
    text += note.body;
  } else {
    if (note.title) text += note.title + '\n' + '─'.repeat(note.title.length) + '\n\n';
    text += note.body;
  }
  const blob = new Blob([text], { type: 'text/plain' });
  const url  = URL.createObjectURL(blob);
  const a    = document.createElement('a');
  const safe = (note.title || 'note').replace(/[^a-z0-9]/gi, '_').substring(0, 40);
  a.href     = url;
  a.download = `${safe}.${ext}`;
  document.body.appendChild(a);
  a.click();
  document.body.removeChild(a);
  URL.revokeObjectURL(url);
  closeCopyDialog();
  showToast(`Saved as ${safe}.${ext}`);
}

// ─── SEARCH ───────────────────────────────────────────────────────────────
function setupSearch() {
  let timer;
  document.getElementById('searchInput').addEventListener('input', e => {
    clearTimeout(timer);
    timer = setTimeout(() => renderAll(e.target.value), 150);
  });
}

// ─── TOAST ────────────────────────────────────────────────────────────────
let toastTimer;
function showToast(msg) {
  clearTimeout(toastTimer);
  const existing = document.querySelector('.toast');
  if (existing) existing.remove();
  const t = document.createElement('div');
  t.className = 'toast'; t.textContent = msg;
  document.body.appendChild(t);
  toastTimer = setTimeout(() => t.remove(), 2500);
}

// ─── KEYBOARD ─────────────────────────────────────────────────────────────
document.addEventListener('keydown', e => {
  if (e.key === 'Escape') {
    if (document.getElementById('copyDialog').style.display !== 'none') { closeCopyDialog(); return; }
    if (document.getElementById('editModal').style.display  !== 'none') { closeModal(); return; }
    if (composerOpen) closeComposer();
  }
});

// Click outside composer
document.addEventListener('click', e => {
  const composer = document.getElementById('composer');
  if (composerOpen && !composer.contains(e.target)) {
    closeComposer();
  }
});
</script>

</body>
</html>