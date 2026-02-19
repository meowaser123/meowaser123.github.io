<!DOCTYPE html>
<html>
<head>
<meta charset="UTF-8">
<title>Roblox Inventory Viewer</title>

<style>
  body {
    background:#0f0f0f;
    color:#eee;
    font-family:Arial;
    text-align:center;
  }

  h1 { margin-top:20px; }

  input, button {
    padding:10px;
    margin:6px;
    font-size:16px;
    border-radius:6px;
    border:none;
  }

  button {
    background:#2ea44f;
    color:white;
    cursor:pointer;
  }

  .grid {
    display:flex;
    flex-wrap:wrap;
    justify-content:center;
    gap:14px;
    margin-top:20px;
  }

  .card {
    background:#1c1c1c;
    padding:10px;
    width:190px;
    border-radius:12px;
    box-shadow:0 0 12px #000;
  }

  img {
    width:160px;
    height:160px;
    object-fit:contain;
    background:#000;
    border-radius:8px;
  }

  .name {
    font-weight:bold;
    margin-top:8px;
  }

  .meta {
    font-size:13px;
    color:#bbb;
    margin-top:3px;
  }
</style>
</head>

<body>

<h1>Inventory Viewer</h1>

<input id="uid" placeholder="Enter User ID">
<input id="type" placeholder="Asset Type (default 40)">
<button onclick="load()">Fetch Once</button>

<div id="status"></div>
<div class="grid" id="grid"></div>

<script>

// 🔧 PUT YOUR WORKER URL HERE
const WORKER = "https://jolly-frog-e1eb.devrahsanko.workers.dev/";


async function load() {

  const uid = document.getElementById("uid").value.trim();
  const type = document.getElementById("type").value.trim() || "40";

  const status = document.getElementById("status");
  const grid = document.getElementById("grid");

  if (!uid) {
    alert("Enter a User ID");
    return;
  }

  status.textContent = "Fetching inventory...";
  grid.innerHTML = "";

  try {

    const res = await fetch(
      `${WORKER}?userId=${encodeURIComponent(uid)}&type=${encodeURIComponent(type)}`
    );

    const data = await res.json();

    if (!Array.isArray(data)) {
      status.textContent = data.error || "No data.";
      return;
    }

    status.textContent = `Loaded ${data.length} assets`;

    for (const a of data) {

      const card = document.createElement("div");
      card.className = "card";

      card.innerHTML = `
        <img src="${a.image}" loading="lazy">
        <div class="name">${escapeHtml(a.name)}</div>
        <div class="meta">ID: ${a.id}</div>
        <div class="meta">Type: ${a.typeId}</div>
        <div class="meta">Creator: ${escapeHtml(a.creatorName || "Unknown")}</div>
        <div class="meta">${a.created ? new Date(a.created).toLocaleDateString() : ""}</div>
      `;

      grid.appendChild(card);
    }

  } catch (e) {
    status.textContent = "Request failed.";
    console.error(e);
  }
}


// 🔒 Prevent HTML injection
function escapeHtml(text) {
  if (!text) return "";
  return text.replace(/[&<>"']/g, m => ({
    "&": "&amp;",
    "<": "&lt;",
    ">": "&gt;",
    '"': "&quot;",
    "'": "&#039;"
  })[m]);
}

</script>

</body>
</html>
