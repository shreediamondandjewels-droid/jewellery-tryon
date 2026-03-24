<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1.0" />
  <title>Jewellery Try-On Studio Pro (Free)</title>
  <style>
    :root{--bg:#0f0f12;--card:#17171c;--accent:#d4af37;--muted:#9aa0a6}
    *{box-sizing:border-box}
    body{margin:0;font-family:Arial, Helvetica, sans-serif;background:linear-gradient(180deg,#0f0f12,#111217);color:#fff}
    header{padding:16px 20px;border-bottom:1px solid #222;display:flex;align-items:center;justify-content:space-between}
    h1{font-size:18px;margin:0;letter-spacing:.5px}
    .wrap{max-width:1100px;margin:20px auto;padding:0 16px}
    .grid{display:grid;grid-template-columns:280px 1fr;gap:16px}
    .card{background:var(--card);border:1px solid #222;border-radius:16px;padding:14px}
    .controls label{display:block;font-size:12px;color:var(--muted);margin:10px 0 6px}
    input[type="file"],input[type="range"],select{width:100%}
    .row{display:flex;gap:10px}
    button{background:#000;border:1px solid #333;color:#fff;border-radius:10px;padding:10px 12px;cursor:pointer}
    button.primary{background:var(--accent);color:#000;border:none}
    canvas{width:100%;height:auto;border-radius:16px;background:#fff}
    .hint{font-size:12px;color:var(--muted);margin-top:8px}
    footer{color:#7a7f86;text-align:center;padding:16px;font-size:12px}
  </style>
</head>
<body>
  <header>
    <div style="display:flex;align-items:center;gap:10px;">
      <img id="logoPreview" src="" alt="Logo" style="height:40px;display:none;border-radius:6px;"/>
      <h1>💎 Shree Diamond Try-On Studio</h1>
    </div>
    <div class="row">
      <button onclick="resetAll()">Reset</button>
      <button class="primary" onclick="downloadImage()">Download</button>
    </div>
  </header>

  <div class="wrap">
    <div class="grid">
      <!-- Controls -->
      <div class="card controls">
        <label>Upload Logo
        <input type="file" id="logoUpload" accept="image/*" />

        <label>Upload Model Image</label>
        <input type="file" id="modelUpload" accept="image/*" />

        <label>Upload Jewellery (PNG preferred)</label>
        <input type="file" id="jewelleryUpload" accept="image/*" />

        <label>Preset Position</label>
        <select id="preset">
          <option value="custom">Custom (drag)</option>
          <option value="necklace">Necklace</option>
          <option value="earring">Earring</option>
          <option value="ring">Ring</option>
          <option value="bracelet">Bracelet</option>
        </select>

        <label>Size</label>
        <input type="range" id="size" min="20" max="300" value="120" />

        <label>Rotate</label>
        <input type="range" id="rotate" min="-180" max="180" value="0" />

        <label>Opacity</label>
        <input type="range" id="opacity" min="0" max="100" value="100" />

        <div class="row">
          <button onclick="removeBg()">Auto Remove BG (basic)</button>
          <button onclick="centerItem()">Center</button>
        </div>
        <p class="hint">Tip: Use transparent PNG for best results. Drag on canvas to position.</p>
      </div>

      <!-- Canvas -->
      <div class="card">
        <canvas id="canvas" width="700" height="900"></canvas>
      </div>
    </div>
  </div>

  <footer>Free version • No AI used • Works offline</footer>

  <script>
    const canvas = document.getElementById('canvas');
    const ctx = canvas.getContext('2d');

    let modelImg = new Image();
    let jewelleryImg = new Image();

    let x = 250, y = 250, size = 120, angle = 0, opacity = 1;
    let dragging = false;

    const modelUpload = document.getElementById('modelUpload');
    const jewelleryUpload = document.getElementById('jewelleryUpload');
    const sizeInput = document.getElementById('size');
    const rotateInput = document.getElementById('rotate');
    const opacityInput = document.getElementById('opacity');
    const preset = document.getElementById('preset');
    const logoUpload = document.getElementById('logoUpload');
    const logoPreview = document.getElementById('logoPreview');

    modelUpload.onchange = e => loadImage(e, modelImg, true);
    logoUpload.onchange = e => {
      const file = e.target.files[0];
      if(!file) return;
      const reader = new FileReader();
      reader.onload = ev => {
        logoPreview.src = ev.target.result;
        logoPreview.style.display = 'block';
      };
      reader.readAsDataURL(file);
    };
    jewelleryUpload.onchange = e => loadImage(e, jewelleryImg, false);

    function loadImage(e, img, isModel){
      const file = e.target.files[0];
      if(!file) return;
      const reader = new FileReader();
      reader.onload = ev => { img.src = ev.target.result; };
      reader.readAsDataURL(file);
    }

    modelImg.onload = draw;
    jewelleryImg.onload = draw;

    sizeInput.oninput = () => { size = +sizeInput.value; draw(); };
    rotateInput.oninput = () => { angle = +rotateInput.value; draw(); };
    opacityInput.oninput = () => { opacity = opacityInput.value/100; draw(); };

    preset.onchange = () => applyPreset();

    function applyPreset(){
      if(!modelImg.src) return;
      switch(preset.value){
        case 'necklace': x = canvas.width/2 - size/2; y = canvas.height*0.45; break;
        case 'earring': x = canvas.width*0.55; y = canvas.height*0.28; break;
        case 'ring': x = canvas.width*0.6; y = canvas.height*0.65; break;
        case 'bracelet': x = canvas.width*0.6; y = canvas.height*0.7; break;
        default: return;
      }
      draw();
    }

    function draw(){
      ctx.clearRect(0,0,canvas.width,canvas.height);
      if(modelImg.src) ctx.drawImage(modelImg,0,0,canvas.width,canvas.height);
      if(jewelleryImg.src){
        ctx.save();
        ctx.globalAlpha = opacity;
        ctx.translate(x + size/2, y + size/2);
        ctx.rotate(angle * Math.PI/180);
        ctx.drawImage(jewelleryImg, -size/2, -size/2, size, size);
        ctx.restore();
      }
    }

    canvas.addEventListener('mousedown', ()=> dragging=true);
    canvas.addEventListener('mouseup', ()=> dragging=false);
    canvas.addEventListener('mousemove', e => {
      if(!dragging) return;
      const rect = canvas.getBoundingClientRect();
      x = e.clientX - rect.left - size/2;
      y = e.clientY - rect.top - size/2;
      draw();
    });

    function centerItem(){ x = canvas.width/2 - size/2; y = canvas.height/2 - size/2; draw(); }

    function resetAll(){
      x=250; y=250; size=120; angle=0; opacity=1;
      sizeInput.value=120; rotateInput.value=0; opacityInput.value=100; preset.value='custom';
      draw();
    }

    function removeBg(){
      // simple white background removal (basic, not perfect)
      const temp = document.createElement('canvas');
      const tctx = temp.getContext('2d');
      temp.width = jewelleryImg.width;
      temp.height = jewelleryImg.height;
      tctx.drawImage(jewelleryImg,0,0);
      const imgData = tctx.getImageData(0,0,temp.width,temp.height);
      for(let i=0;i<imgData.data.length;i+=4){
        const r=imgData.data[i], g=imgData.data[i+1], b=imgData.data[i+2];
        if(r>240 && g>240 && b>240){ imgData.data[i+3]=0; }
      }
      tctx.putImageData(imgData,0,0);
      jewelleryImg.src = temp.toDataURL();
    }

    function downloadImage(){
      const link = document.createElement('a');
      link.download = 'jewellery-tryon-pro.png';
      link.href = canvas.toDataURL('image/png');
      link.click();
    }
  </script>
</body>
</html>
