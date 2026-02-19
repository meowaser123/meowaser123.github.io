<!DOCTYPE html>
<html>
<head>
<meta charset="UTF-8">
<title>Inventory Viewer</title>
<style>
  body { background:#0f0f0f; color:#eee; font-family:Arial; text-align:center; }
  .grid { display:flex; flex-wrap:wrap; gap:12px; justify-content:center; }
  .card { background:#1c1c1c; padding:10px; width:180px; border-radius:10px; }
  img { width:150px; height:150px; object-fit:contain; }
</style>
</head>
<body>

<h1>Inventory Viewer</h1>
<input id="uid" placeholder="User ID">
<button onclick="load()">Fetch</button>

<div class="grid" id="grid"></div>

<script>

const worker = "https://jolly-frog-e1eb.devrahsanko.workers.dev/";

async function load() {

  const uid = document.getElementById("uid").value.trim();
  const grid = document.getElementById("grid");

  grid.innerHTML = "Loading...";

  const res = await fetch(worker + "?userId=" + uid);
  const data = await res.json();

  grid.innerHTML = "";

  data.forEach(a => {

    const d = document.createElement("div");
    d.className = "card";

    d.innerHTML = `
      <img src="${a.image}">
      <div><b>${a.name}</b></div>
      <div>ID: ${a.id}</div>
      <div>${a.assetType}</div>
      <div>${a.creator || ""}</div>
    `;

    grid.appendChild(d);
  });
}

</script>

</body>
</html>
