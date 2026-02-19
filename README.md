<!DOCTYPE html>
<html>
<head>
<meta charset="UTF-8">
<title>Inventory Dashboard</title>

<style>

/* ===== GLOBAL ===== */

body {
  margin:0;
  background:#0b0f14;
  color:#e6edf3;
  font-family:system-ui,Segoe UI,Arial;
}

* { box-sizing:border-box; }

/* ===== LAYOUT ===== */

.app {
  display:flex;
  height:100vh;
}

/* ===== SIDEBAR ===== */

.sidebar {
  width:260px;
  background:#11161d;
  padding:20px;
  border-right:1px solid #1f2933;
}

.logo {
  font-size:20px;
  font-weight:bold;
  margin-bottom:20px;
}

.sectionTitle {
  margin-top:18px;
  font-size:13px;
  color:#8b949e;
}

input, button {
  width:100%;
  padding:10px;
  margin-top:8px;
  border-radius:6px;
  border:none;
  font-size:14px;
}

button {
  background:#238636;
  color:white;
  cursor:pointer;
}

.pins {
  margin-top:10px;
  font-size:13px;
}

.pinItem {
  background:#1c2128;
  padding:6px;
  margin:4px 0;
  border-radius:5px;
}

/* ===== MAIN ===== */

.main {
  flex:1;
  display:flex;
  flex-direction:column;
}

/* ===== TOP BAR ===== */

.topbar {
  padding:12px 20px;
  background:#0d1117;
  border-bottom:1px solid #1f2933;
  display:flex;
  gap:10px;
}

.topbar input {
  max-width:320px;
}

/* ===== STATS ===== */

.stats {
  padding:12px 20px;
  display:flex;
  gap:18px;
  font-size:14px;
  border-bottom:1px solid #1f2933;
}

/* ===== GRID ===== */

.grid {
  padding:20px;
  overflow:auto;
  display:grid;
  grid-template-columns:repeat(auto-fill, minmax(190px,1fr));
  gap:16px;
}

.card {
  background:#161b22;
  padding:10px;
  border-radius:12px;
  position:relative;
  transition:.15s;
}

.card:hover {
  transform:translateY(-3px);
  box-shadow:0 8px 20px rgba(0,0,0,.6);
}

.card.pinned {
  outline:2px solid gold;
}

.thumb {
  width:100%;
  aspect-ratio:1;
  object-fit:contain;
  background:#000;
  border-radius:8px;
  cursor:pointer;
}

.name { font-weight:600; margin-top:6px; }
.meta { font-size:12px; color:#8b949e; }

.pin {
  position:absolute;
  top:6px;
  right:8px;
  font-size:18px;
  cursor:pointer;
}

/* ===== ZOOM ===== */

#zoom {
  display:none;
  position:fixed;
  inset:0;
  background:rgba(0,0,0,.92);
  justify-content:center;
  align-items:center;
  z-index:999;
}

#zoom img {
  max-width:90%;
  max-height:90%;
  border-radius:10px;
}

#closeZoom {
  position:absolute;
  top:20px;
  right:30px;
  font-size:28px;
  cursor:pointer;
}

</style>
</head>

<body>

<div class="app">

  <!-- SIDEBAR -->
  <div class="sidebar">
    <div class="logo">Inventory Dashboard</div>
    <div class="sectionTitle">Fetch</div>
    <input id="uid" placeholder="User ID">
    <input id="type" placeholder="Asset Type (40)">
    <button onclick="load()">Fetch Inventory</button>
    <div class="sectionTitle">Pinned Creators</div>
    <div id="pins" class="pins"></div>

  </div>
  <!-- MAIN -->
  <div class="main">
    <!-- TOPBAR -->
    <div class="topbar">
      <input id="search" placeholder="Search assets..." oninput="render()">
    </div>
    <!-- STATS -->
    <div class="stats">
      <div>Total: <span id="count">0</span></div>
      <div>Pinned Creators: <span id="pcount">0</span></div>
    </div>
    <!-- GRID -->
    <div id="grid" class="grid"></div>

  </div>

</div>


<!-- ZOOM VIEW -->
<div id="zoom" onclick="closeZoom()">
  <span id="closeZoom">✕</span>
  <img id="zoomImg">
</div>


<script>

// 🔧 YOUR WORKER URL
const WORKER = "https://jolly-frog-e1eb.devrahsanko.workers.dev/";

// DATA
let assets = [];
let pinned = JSON.parse(localStorage.getItem("pins") || "[]");


// ===== PIN SYSTEM =====

function togglePin(name) {
  if (!name) return;

  if (pinned.includes(name))
    pinned = pinned.filter(n => n !== name);
  else
    pinned.push(name);

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


// ===== FETCH =====

async function load() {

  const uid = document.getElementById("uid").value.trim();
  const type = document.getElementById("type").value.trim() || "40";

  if (!uid) return alert("Enter User ID");

  const res = await fetch(
    `${WORKER}?userId=${encodeURIComponent(uid)}&type=${encodeURIComponent(type)}`
  );

  assets = await res.json();
  render();
}


// ===== RENDER =====

function render() {

  const grid = document.getElementById("grid");
  const q = document.getElementById("search").value.toLowerCase();

  let list = assets.filter(a =>
    !q ||
    a.name.toLowerCase().includes(q) ||
    String(a.id).includes(q) ||
    (a.creatorName || "").toLowerCase().includes(q)
  );

  // pinned first
  list.sort((a,b) =>
    pinned.includes(b.creatorName) - pinned.includes(a.creatorName)
  );

  grid.innerHTML = "";

  list.forEach(a => {

    const card = document.createElement("div");
    card.className = "card";

    if (pinned.includes(a.creatorName))
      card.classList.add("pinned");

    card.innerHTML = `
      <div class="pin" onclick="togglePin('${escape(a.creatorName)}')">
        ${pinned.includes(a.creatorName) ? "★" : "☆"}
      </div>

      <img class="thumb"
           src="${a.image}"
           loading="lazy"
           onclick="zoom('${a.image}')">

      <div class="name">${escape(a.name)}</div>
      <div class="meta">ID: ${a.id}</div>
      <div class="meta">${escape(a.creatorName || "Unknown")}</div>
    `;

    grid.appendChild(card);
  });

  document.getElementById("count").textContent = list.length;
}


// ===== ZOOM =====

function zoom(src) {
  document.getElementById("zoomImg").src = src;
  document.getElementById("zoom").style.display = "flex";
}

function closeZoom() {
  document.getElementById("zoom").style.display = "none";
}


// ===== UTILS =====

function escape(t) {
  return t ? t.replace(/[&<>"']/g,m=>({
    "&":"&amp;","<":"&lt;",">":"&gt;",
    '"':"&quot;","'":"&#039;"
  })[m]) : "";
}


// INIT
renderPins();

</script>

</body>
</html>
