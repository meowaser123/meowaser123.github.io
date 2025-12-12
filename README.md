updated check
<html lang="en">
<head>
<meta charset="UTF-8">
<title>Roblox Advanced Scanner</title>
<style>
body{font-family:'Segoe UI',sans-serif;background:#0a0a0a;color:white;padding:20px;}
h1{color:#ff416c;text-align:center;}
textarea{width:100%;height:150px;border-radius:12px;padding:15px;margin-bottom:15px;background:#111;color:white;border:2px solid #333;}
button{padding:12px 25px;border:none;border-radius:12px;background:linear-gradient(90deg,#ff416c,#ff4b2b);color:white;font-weight:bold;cursor:pointer;margin-bottom:15px;transition:0.3s;}
button:hover{opacity:0.85;}
pre{
    background:#111;       /* fully dark box */
    color:white;           /* white text */
    padding:15px;
    border-radius:12px;
    max-height:500px;
    overflow:auto;
    border:2px solid #333;
    font-family: 'Courier New', monospace;
    white-space: pre-wrap;        
    word-wrap: break-word;
}
.container{max-width:900px;margin:0 auto;}
label{margin-right:20px;font-weight:bold;color:#aaa;}
input[type=checkbox]{margin-right:5px;}
</style>
</head>
<body>
<div class="container">
<h1>Roblox Advanced Async Scanner</h1>
<textarea id="ids" placeholder="Paste Asset/Place/DevProduct IDs here (comma or newline)"></textarea>
<br>
<label><input type="checkbox" id="asset" checked> Asset</label>
<label><input type="checkbox" id="place" checked> Game</label>
<label><input type="checkbox" id="devproducts" checked> DevProducts</label>
<label><input type="checkbox" id="badges"> Badges</label>
<label><input type="checkbox" id="gamepasses"> GamePasses</label>
<br><br>
<button onclick="runScan()">Scan</button>
<pre id="out"></pre>
</div>

<script>
async function runScan(){
    const ids = document.getElementById('ids').value.split(/[\n,]+/).map(x=>x.trim()).filter(Boolean)
    const endpoints=[]
    if(document.getElementById('asset').checked) endpoints.push('asset')
    if(document.getElementById('place').checked) endpoints.push('place')
    if(document.getElementById('devproducts').checked) endpoints.push('devproducts')
    if(document.getElementById('badges').checked) endpoints.push('badges')
    if(document.getElementById('gamepasses').checked) endpoints.push('gamepasses')

    if(ids.length===0){document.getElementById('out').textContent="No IDs provided"; return;}

    document.getElementById('out').textContent="Scanning..."

    try{
        const res = await fetch('https://summer-hat-b3c2.devrahsanko.workers.dev',{
            method:'POST',
            headers:{'Content-Type':'application/json'},
            body:JSON.stringify({ids,endpoints})
        })
        const data = await res.json()
        document.getElementById('out').textContent = JSON.stringify(data,null,2)
    }catch(e){
        document.getElementById('out').textContent = "Error: "+e.message
    }
}
</script>
</body>
</html>
