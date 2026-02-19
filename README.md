<!DOCTYPE html>
<html>
<head>
<meta charset="UTF-8">
<title>Inventory Dashboard</title>
<style>
/* ===== GLOBAL ===== */
body { margin:0; font-family:system-ui,Segoe UI,Arial; background:#0b0f14; color:#e6edf3; }
* { box-sizing:border-box; }

/* ===== LAYOUT ===== */
.app { display:flex; height:100vh; }

/* ===== SIDEBAR ===== */
.sidebar {
  width:280px;
  background:#11161d;
  padding:20px;
  border-right:1px solid #1f2933;
  display:flex;
  flex-direction:column;
}

.logo { font-size:20px; font-weight:bold; margin-bottom:20px; }

.nav-btn {
  background:none;
  border:none;
  color:#e6edf3;
  font-size:14px;
  padding:8px 12px;
  margin:2px 0;
  text-align:left;
  cursor:pointer;
  border-radius:6px;
  transition:.15s;
}

.nav-btn:hover { background:#1c2128; }

.nav-btn.active { background:#238636; }

/* ===== SIDEBAR CONTENT ===== */
.sidebar-content { margin-top:12px; flex:1; overflow:auto; }

/* ===== MAIN ===== */
.main { flex:1; display:flex; flex-direction:column; }

/* ===== TOPBAR ===== */
.topbar { padding:12px 20px; background:#0d1117; border-bottom:1px solid #1f2933; display:flex; gap:10px; }
.topbar input { max-width:320px; width:100%; padding:8px; border-radius:6px; border:none; }

/* ===== STATS ===== */
.stats { padding:12px 20px; display:flex; gap:18px; font-size:14px; border-bottom:1px solid #1f2933; }

/* ===== GRID ===== */
.grid { padding:20px; overflow:auto; display:grid; grid-template-columns:repeat(auto-fill, minmax(190px,1fr)); gap:16px; }

.card { background:#161b22; padding:10px; border-radius:12px; position:relative; transition:.15s; }
.card:hover { transform:translateY(-3px); box-shadow:0 8px 20px rgba(0,0,0,.6); }
.card.pinned { outline:2px solid gold; }

.thumb { width:100%; aspect-ratio:1; object-fit:contain; background:#000; border-radius:8px; cursor:pointer; }

.name { font-weight:600; margin-top:6px; }
.meta { font-size:12px; color:#8b949e; }

.pin { position:absolute; top:6px; right:8px; font-size:18px; cursor:pointer; }

/* ===== ZOOM ===== */
#zoom { display:none; position:fixed; inset:0; background:rgba(0,0,0,.92); justify-content:center; align-items:center; z-index:999; }
#zoom img { max-width:90%; max-height:90%; border-radius:10px; }
#closeZoom { position:absolute; top:20px; right:30px; font-size:28px; cursor:pointer; }

/* ===== HOME ===== */
.home-content { padding:20px; overflow:auto; }
.home-content h2 { margin-top:0; }
.home-content ul { margin:6px 0 6px 20px; }
</style>
</head>

<body>

<div class="app">

  <!-- SIDEBAR -->
  <div class="sidebar">
    <div class="logo">Inventory Dashboard</div>
    <button class="nav-btn active" onclick="showTab('home')">🏠 Home</button>
    <button class="nav-btn" onclick="showTab('inventory')">📦 Inventory</button>
    <div class="sidebar-content">
      <!-- Pinned creators list -->
      <div id="pinsContainer">
        <div class="sectionTitle">Pinned Creators</div>
        <div id="pins" class="pins"></div>
      </div>
    </div>
  </div>

  <!-- MAIN -->
  <div class="main">
    <!-- HOME TAB -->
    <div id="home" class="tab">
      <div class="home-content">
        <h2>Welcome to Inventory Dashboard</h2>
        <p>This dashboard lets you fetch Roblox inventories and view assets with thumbnails, metadata, and pinned creators.</p>
        <ul>
          <li>Fetch inventory from a user ID using your Cloudflare Worker backend.</li>
          <li>Click thumbnails to zoom full screen.</li>
          <li>Pin creators manually or by star to highlight and sort their assets.</li>
          <li>Search assets by name, ID, or creator.</li>
          <li>All pins are saved locally in your browser.</li>
        </ul>
        <p>Technologies used:</p>
        <ul>
          <li>HTML, CSS, JS for frontend dashboard</li>
          <li>Cloudflare Worker backend to fetch inventory and rbxg leaks asset info</li>
          <li>LocalStorage for pinned creators</li>
          <li>Modern CSS Grid and Flexbox for responsive UI</li>
        </ul>
      </div>
    </div>
    <!-- INVENTORY TAB -->
    <div id="inventory" class="tab" style="display:none;">
      <div class="topbar">
        <input id="search" placeholder="Search assets..." oninput="render()">
      </div>
      <div class="stats">
        <div>Total: <span id="count">0</span></div>
        <div>Pinned Creators: <span id="pcount">0</span></div>
      </div>
      <div style="padding:12px 20px;">
        <input id="uid" placeholder="User ID">
        <input id="type" placeholder="Asset Type (default 40)">
        <button onclick="load()">Fetch Inventory</button>
        <div style="margin-top:6px;">
          <input id="manualPin" placeholder="Add creator manually">
          <button onclick="addManualPin()">Add Pin</button>
        </div>
      </div>
      <div id="grid" class="grid"></div>
    </div>
  </div>

</div>

<!-- Zoom Modal -->
<div id="zoom" onclick="closeZoom()">
  <span id="closeZoom">✕</span>
  <img id="zoomImg">
</div>

<script>
// ---------------- CONFIG ----------------
const WORKER = "https://jolly-frog-e1eb.devrahsanko.workers.dev/";

// ---------------- DATA ----------------
let assets = [];
let pinned = JSON.parse(localStorage.getItem("pins") || "[]");

// ---------------- NAVIGATION ----------------
function showTab(tab) {
  document.querySelectorAll(".tab").forEach(t => t.style.display = "none");
  document.getElementById(tab).style.display = "block";
  document.querySelectorAll(".nav-btn").forEach(b => b.classList.remove("active"));
  event.currentTarget.classList.add("active");
}

// ---------------- PIN SYSTEM ----------------
function togglePin(name) {
  if (!name) return;
  if (pinned.includes(name)) pinned = pinned.filter(n => n !== name);
  else pinned.push(name);
  localStorage.setItem("pins", JSON.stringify(pinned));
  renderPins();
  render();
}

function renderPins() {
  const p = document.getElementById("pins");
  p.innerHTML = "";
  pinned.forEach(n => {
    const d = document.createElement("div");
    d.className = "pinItem";
    d.textContent = n;
    p.appendChild(d);
  });
  document.getElementById("pcount").textContent = pinned.length;
}

function addManualPin() {
  const input = document.getElementById("manualPin");
  const name = input.value.trim();
  if (!name) return alert("Enter a creator name!");
  if (!pinned.includes(name)) pinned.push(name);
  localStorage.setItem("pins", JSON.stringify(pinned));
  renderPins();
  render();
  input.value = "";
}

// ---------------- FETCH INVENTORY ----------------
async function load() {
  const uid = document.getElementById("uid").value.trim();
  const type = document.getElementById("type").value.trim() || "40";
  if (!uid) return alert("Enter User ID");

  const res = await fetch(`${WORKER}?userId=${encodeURIComponent(uid)}&type=${encodeURIComponent(type)}`);
  assets = await res.json();
  render();
}

// ---------------- RENDER GRID ----------------
function render() {
  const grid = document.getElementById("grid");
  const q = document.getElementById("search").value.toLowerCase();

  let list = assets.filter(a =>
    !q ||
    a.name.toLowerCase().includes(q) ||
    String(a.id).includes(q) ||
    (a.creatorName || "").toLowerCase().includes(q)
  );

  list.sort((a,b) => pinned.includes(b.creatorName) - pinned.includes(a.creatorName));

  grid.innerHTML = "";

  list.forEach(a => {
    const card = document.createElement("div");
    card.className = "card";
    if (pinned.includes(a.creatorName)) card.classList.add("pinned");

    card.innerHTML = `
      <div class="pin" onclick="togglePin('${escape(a.creatorName)}')">
        ${pinned.includes(a.creatorName) ? "★" : "☆"}
      </div>

      <img class="thumb" src="${a.image}" loading="lazy" onclick="zoom('${a.image}')">
      <div class="name">${escape(a.name)}</div>
      <div class="meta">ID: ${a.id}</div>
      <div class="meta">${escape(a.creatorName || "Unknown")}</div>
    `;
    grid.appendChild(card);
  });

  document.getElementById("count").textContent = list.length;
  renderPins();
}

// ---------------- ZOOM ----------------
function zoom(src) { document.getElementById("zoomImg").src = src; document.getElementById("zoom").style.display="flex"; }
function closeZoom() { document.getElementById("zoom").style.display="none"; }

// ---------------- ESCAPE ----------------
function escape(t){ return t?t.replace(/[&<>"']/g,m=>({"&":"&amp;","<":"&lt;",">":"&gt;",'"':"&quot;","'":"&#039;"})[m]) : ""; }

// ---------------- INIT ----------------
renderPins();
</script>
</body>
</html>
