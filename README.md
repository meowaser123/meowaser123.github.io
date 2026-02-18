<!DOCTYPE html>
<html>
<head>
<meta charset="UTF-8">
<title>Roblox Inventory Type 40 Viewer</title>
<style>
  body { background:#111; color:#eee; font-family:Arial; text-align:center; }
  .item { background:#222; margin:10px auto; padding:10px; width:80%; border-radius:8px; }
</style>
</head>
<body>

<h1>User 205430552 — Asset Type 40</h1>
<button onclick="load()">Fetch Once</button>

<div id="out"></div>

<script>
async function load() {
  const out = document.getElementById("out");
  out.textContent = "Loading...";

  const url = "https://inventory.roblox.com/v2/users/205430552/inventory/40?cursor=&limit=100&sortOrder=Desc";

  try {
    const res = await fetch(url);
    const data = await res.json();

    if (!data.data || data.data.length === 0) {
      out.textContent = "No items or inventory private.";
      return;
    }

    out.innerHTML = "";

    data.data.forEach(item => {
      const div = document.createElement("div");
      div.className = "item";
      div.innerHTML = `
        <b>${item.name}</b><br>
        Asset ID: ${item.assetId}<br>
        Type: ${item.assetType}<br>
        Created: ${item.created}
      `;
      out.appendChild(div);
    });

  } catch (err) {
    out.textContent = "Request failed (CORS or private inventory).";
    console.error(err);
  }
}
</script>

</body>
</html>
