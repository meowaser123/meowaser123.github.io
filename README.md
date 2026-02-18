<!DOCTYPE html>
<html>
<head>
<meta charset="UTF-8">
<title>Roblox Inventory Fetch (CORS Fixed)</title>
<style>
  body { background:#111; color:#eee; font-family:Arial; text-align:center; }
  .item { background:#222; margin:10px auto; padding:10px; width:80%; border-radius:8px; }
</style>
</head>
<body>

<h1>Inventory Viewer — CORS Enabled</h1>
<button onclick="load()">Fetch Once</button>

<div id="out"></div>

<script>
const proxy = "https://jolly-frog-e1eb.devrahsanko.workers.dev/?url=";

async function load() {

  const out = document.getElementById("out");
  out.textContent = "Loading...";

  const api =
  "https://inventory.roblox.com/v2/users/205430552/inventory/40?limit=100&sortOrder=Desc";

  try {

    const res = await fetch(proxy + encodeURIComponent(api));
    const data = await res.json();

    if (!data.data) {
      out.textContent = "No data or private inventory.";
      return;
    }

    out.innerHTML = "";

    data.data.forEach(item => {
      const d = document.createElement("div");
      d.className = "item";
      d.innerHTML = `
        <b>${item.name}</b><br>
        Asset ID: ${item.assetId}<br>
        Created: ${item.created}
      `;
      out.appendChild(d);
    });

  } catch (e) {
    out.textContent = "Fetch failed.";
    console.error(e);
  }
}
</script>

</body>
</html>
