# pyromarkt--minigame<!doctype html>
<html lang="nl">
<head> 
  <meta charset="utf-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1, viewport-fit=cover">
  <title>Pyromarkt — Rotterdam Portal</title>
  <script src="https://telegram.org/js/telegram-web-app.js"></script>
  <style>
    html,body{margin:0;height:100%;background:#05060a;overflow:hidden;font-family:Arial,sans-serif}
    canvas{display:block}

    .hud{
      position:fixed;left:12px;right:12px;top:12px;display:flex;gap:10px;align-items:center;justify-content:space-between;
      pointer-events:none; z-index:5;
    }
    .chip{
      pointer-events:none;
      background:rgba(20,20,25,.72);border:1px solid rgba(255,255,255,.10);
      padding:10px 12px;border-radius:14px;color:#fff;font-size:14px;
      backdrop-filter: blur(8px); box-shadow:0 10px 30px rgba(0,0,0,.35);
      display:flex;gap:8px;align-items:center;max-width:78vw;
    }
    .bar{width:140px;height:10px;border-radius:999px;background:rgba(255,255,255,.14);overflow:hidden;border:1px solid rgba(255,255,255,.10)}
    .bar>div{height:100%;width:0%;background:linear-gradient(90deg,#ff2a2a,#ffb000)}

    .bottom{
      position:fixed;left:12px;right:12px;bottom:12px;display:flex;gap:10px;justify-content:space-between;flex-wrap:wrap;
      pointer-events:auto; z-index:6;
    }
    .btn{
      border:0;border-radius:14px;padding:12px 12px;font-size:13px;font-weight:900;color:#fff;cursor:pointer;
      background:rgba(255,255,255,.10);border:1px solid rgba(255,255,255,.14);
      box-shadow:0 10px 30px rgba(0,0,0,.25);
      user-select:none;
    }
    .btn.active{background:linear-gradient(90deg,#ff2a2a,#ffb000);border:0}
    .btn.small{padding:10px 12px;font-size:12px;border-radius:12px}

    .panel{
      position:fixed;inset:0;display:none;align-items:center;justify-content:center;padding:16px;
      background:rgba(0,0,0,.45);backdrop-filter:blur(8px);
      z-index:20;
    }
    .card{
      width:min(650px,92vw);background:rgba(15,15,18,.90);border:1px solid rgba(255,255,255,.12);
      border-radius:18px;padding:16px;color:#fff;box-shadow:0 18px 60px rgba(0,0,0,.55);
    }
    .row{display:flex;justify-content:space-between;align-items:center;gap:10px}
    .title{font-size:18px;font-weight:900}
    .sub{opacity:.88;font-size:14px;line-height:1.35;margin-top:8px}
    .grid{display:grid;grid-template-columns:1fr;gap:10px;margin-top:12px}
    @media(min-width:560px){.grid{grid-template-columns:1fr 1fr}}
    .item{background:rgba(255,255,255,.06);border:1px solid rgba(255,255,255,.10);border-radius:14px;padding:12px}
    .item b{display:block;margin-bottom:6px}
    .meta{opacity:.85;font-size:13px;line-height:1.25}
    .buyrow{display:flex;justify-content:space-between;align-items:center;gap:10px;margin-top:10px}

    .toast{
      position:fixed;left:50%;top:74px;transform:translateX(-50%);
      background:rgba(20,20,25,.78);border:1px solid rgba(255,255,255,.12);
      color:#fff;padding:10px 12px;border-radius:14px;font-size:13px;opacity:0;pointer-events:none;
      transition:opacity .2s ease;backdrop-filter:blur(8px);
      z-index:30;
    }

    /* Touch joystick */
    .joy{
      position:fixed;left:12px;bottom:72px;width:140px;height:140px;pointer-events:auto;
      background:rgba(255,255,255,.06);border:1px solid rgba(255,255,255,.10);
      border-radius:999px;backdrop-filter:blur(8px);
      display:flex;align-items:center;justify-content:center;
      z-index:6;
    }
    .joyDot{
      width:54px;height:54px;border-radius:999px;
      background:rgba(255,255,255,.12);border:1px solid rgba(255,255,255,.18);
      transform:translate(0,0);
    }

    /* Logo watermark */
    .wm{
      position:fixed;top:10px;right:10px;width:56px;opacity:.85;pointer-events:none;
      filter: drop-shadow(0 0 12px rgba(255,120,0,.55));
      z-index:10;
    }

    /* Start screen */
    .start{
      position:fixed; inset:0; z-index:999;
      background:#000;
      display:flex; align-items:center; justify-content:center;
      flex-direction:column; gap:18px;
      padding:24px;
      text-align:center;
    }
    .start img{
      width:280px; max-width:75vw;
      filter: drop-shadow(0 0 35px rgba(255,120,0,.55));
      border-radius:18px;
    }
    .start .enter{
      padding:14px 22px;
      font-size:16px; font-weight:900;
      border-radius:12px; border:0; cursor:pointer;
      background:linear-gradient(90deg,#ff2a2a,#ffb000);
      color:#fff;
      box-shadow:0 18px 55px rgba(0,0,0,.6);
    }
    .start .small{
      opacity:.85; color:#fff; font-size:13px; line-height:1.35;
      max-width:560px;
    }
  </style>
</head>
<body>

<!-- Start screen with logo -->
<div class="start" id="startScreen">
  <img src="logo.png" alt="Pyromarkt Phoenix Logo">
  <button class="enter" id="enterBtn">ENTER PORTAAL</button>
  <div class="small">
    Loop door <b>Rotterdam</b>, pak vuurwerk en steek af.<br>
    (Anoniem zichtbaar — geen namen in de app.)
  </div>
</div>

<canvas id="c"></canvas>
<img class="wm" src="logo.png" alt="logo">

<div class="hud">
  <div class="chip">
    🏙️ <b>Rotterdam</b>
    <span style="opacity:.7">•</span>
    💰 <b>Coins</b>: <span id="coins">0</span>
    <span style="opacity:.7">•</span>
    🎒 <b>Items</b>: <span id="inv">0</span>
    <span style="opacity:.7">•</span>
    ⭐ <b>Score</b>: <span id="score">0</span>
  </div>
  <div class="chip" style="justify-content:flex-end;">
    <span style="opacity:.85">Unlock</span>
    <div class="bar"><div id="fill"></div></div>
    <span id="need" style="opacity:.85">0%</span>
  </div>
</div>

<div class="toast" id="toast"></div>

<div class="joy" id="joy">
  <div class="joyDot" id="joyDot"></div>
</div>

<div class="bottom">
  <button class="btn active" id="useBtn">🎆 Steek af</button>
  <button class="btn" id="cycleBtn">🔁 Wissel</button>
  <button class="btn small" id="shopBtn">🛒 Shop</button>
  <button class="btn small" id="howBtn">❓ Uitleg</button>
</div>

<div class="panel" id="howPanel">
  <div class="card">
    <div class="row">
      <div class="title">Uitleg</div>
      <button class="btn small" onclick="closePanel('howPanel')">Sluit</button>
    </div>
    <div class="sub">
      - Loop met <b>WASD/pijltjes</b> of joystick<br>
      - Pak pickups: 🚀 pijlen, 🧁 potten, 🧨 knallers, 🎇 singleshots<br>
      - “Steek af” gebruikt je geselecteerde item<br>
      - Coins → upgrades in de shop<br>
      - Bij 100% unlock: ga door in bot chat
    </div>
  </div>
</div>

<div class="panel" id="shopPanel">
  <div class="card">
    <div class="row">
      <div class="title">Shop</div>
      <button class="btn small" onclick="closePanel('shopPanel')">Sluit</button>
    </div>
    <div class="sub">Koop upgrades met coins (geen echt geld in de game).</div>
    <div class="grid" id="shopGrid"></div>
  </div>
</div>

<div class="panel" id="unlockPanel">
  <div class="card">
    <div class="row">
      <div class="title">✅ Unlock</div>
      <button class="btn small" onclick="closePanel('unlockPanel')">Sluit</button>
    </div>
    <div class="sub">Je hebt genoeg score. Volgende stap: verificatie in bot chat.</div>
    <div style="display:flex;gap:10px;flex-wrap:wrap;margin-top:12px">
      <button class="btn active" id="toBotBtn">Ga naar bot chat</button>
      <button class="btn" onclick="closePanel('unlockPanel')">Oké</button>
    </div>
  </div>
</div>

<script>
  const tg = window.Telegram?.WebApp;
  if (tg) tg.expand();

  const c = document.getElementById('c'), ctx = c.getContext('2d');
  let W=0,H=0,dpr=1;
  function resize(){
    dpr = Math.max(1, Math.min(2, window.devicePixelRatio || 1));
    W = innerWidth|0; H = innerHeight|0;
    c.width = (W*dpr)|0; c.height = (H*dpr)|0;
    c.style.width = W+'px'; c.style.height = H+'px';
    ctx.setTransform(dpr,0,0,dpr,0,0);
  }
  addEventListener('resize', resize); resize();

  const coinsEl = document.getElementById('coins');
  const invEl = document.getElementById('inv');
  const scoreEl = document.getElementById('score');
  const fillEl = document.getElementById('fill');
  const needEl = document.getElementById('need');
  const toastEl = document.getElementById('toast');

  const howBtn = document.getElementById('howBtn');
  const shopBtn = document.getElementById('shopBtn');
  const useBtn = document.getElementById('useBtn');
  const cycleBtn = document.getElementById('cycleBtn');

  const shopGrid = document.getElementById('shopGrid');
  const toBotBtn = document.getElementById('toBotBtn');

  function openPanel(id){ document.getElementById(id).style.display='flex'; }
  function closePanel(id){ document.getElementById(id).style.display='none'; }
  window.closePanel = closePanel;

  // Start button
  document.getElementById('enterBtn').onclick = () => {
    document.getElementById('startScreen').style.display = 'none';
  };

  // Toast
  let tt=null;
  function toast(msg){
    toastEl.textContent = msg;
    toastEl.style.opacity = 1;
    clearTimeout(tt);
    tt = setTimeout(()=>toastEl.style.opacity=0, 1200);
  }

  // Persistence
  const KEY="pyromarkt_rtm_v2";
  const TARGET=260; // unlock score
  const defaultState=()=>({
    coins: 0,
    score: 0,
    inv: { rocket:0, cake:0, banger:0, single:0 },
    selected: "rocket",
    upgrades: { biggerBoom:false, doubleCoins:false, speedBoost:false, extraPickups:false }
  });
  let S = load();
  function load(){
    try{
      const raw = localStorage.getItem(KEY);
      if(!raw) return defaultState();
      const s = JSON.parse(raw);
      return {...defaultState(), ...s, inv:{...defaultState().inv, ...(s.inv||{})}, upgrades:{...defaultState().upgrades, ...(s.upgrades||{})}};
    }catch{ return defaultState(); }
  }
  function save(){ localStorage.setItem(KEY, JSON.stringify(S)); }

  function invCount(){ return S.inv.rocket+S.inv.cake+S.inv.banger+S.inv.single; }
  function updateUI(){
    coinsEl.textContent = S.coins;
    invEl.textContent = invCount();
    scoreEl.textContent = S.score;

    const pct = Math.min(100, Math.floor((S.score/TARGET)*100));
    fillEl.style.width = pct+'%';
    needEl.textContent = pct+'%';
    save();
    if(pct>=100) openPanel('unlockPanel');
  }
  updateUI();

  // Shop
  const shopItems = [
    {id:"biggerBoom", name:"Bigger Boom", desc:"Grotere knal, meer particles.", price:180},
    {id:"doubleCoins", name:"Double Coins", desc:"+1 coin extra bij elke afsteek.", price:220},
    {id:"speedBoost", name:"Speed Boost", desc:"Je loopt sneller door Rotterdam.", price:160},
    {id:"extraPickups", name:"Extra Pickups", desc:"Er spawnen vaker nieuwe items.", price:200},
  ];
  function renderShop(){
    shopGrid.innerHTML="";
    for(const it of shopItems){
      const owned = !!S.upgrades[it.id];
      const canBuy = S.coins >= it.price && !owned;
      const el = document.createElement('div');
      el.className="item";
      el.innerHTML = `
        <b>${it.name} ${owned?"✅":""}</b>
        <div class="meta">${it.desc}</div>
        <div class="buyrow">
          <div class="meta">Prijs: <b>${it.price}</b> coins</div>
          <button class="btn small ${owned?'':'active'}" ${owned?'disabled':''} data-id="${it.id}">
            ${owned ? "In bezit" : (canBuy ? "Kopen" : "Te weinig")}
          </button>
        </div>
      `;
      shopGrid.appendChild(el);
    }
    shopGrid.querySelectorAll("button[data-id]").forEach(b=>{
      b.onclick=()=>{
        const id=b.getAttribute("data-id");
        const it=shopItems.find(x=>x.id===id);
        if(!it || S.upgrades[id]) return;
        if(S.coins<it.price) return toast("Te weinig coins.");
        S.coins-=it.price; S.upgrades[id]=true;
        updateUI(); renderShop(); toast("Gekocht ✅");
      };
    });
  }

  // Map (Rotterdam vibe)
  const world = { w: 2400, h: 1600 };
  const water = { x: 0, y: 520, w: 2400, h: 320 }; // Maas
  const bridge = { x: 1120, y: 520, w: 160, h: 320 }; // Erasmusbrug

  const poi = [
    {x: 380, y: 340, text:"MARKTHAL"},
    {x: 1760, y: 260, text:"KOP VAN ZUID"},
    {x: 1140, y: 480, text:"ERASMUSBRUG"},
  ];

  const buildings = [];
  function addBlock(x,y,w,h,label){ buildings.push({x,y,w,h,label}); }
  addBlock(200,140,320,220,"Centrum");
  addBlock(620,120,260,200,"Winkels");
  addBlock(980,160,300,220,"Toren");
  addBlock(1460,140,340,240,"WTC");
  addBlock(1900,160,360,220,"Blokken");
  addBlock(260,980,380,240,"Haven");
  addBlock(760,980,360,220,"Loods");
  addBlock(1320,980,420,260,"Zuid");
  addBlock(1880,980,360,240,"Kade");

  // Player
  const player = { x: 420, y: 420, r: 16, vx:0, vy:0 };
  let cam = { x:0, y:0 };

  // Pickups
  const pickups = [];
  function rand(a,b){return a+Math.random()*(b-a);}
  function spawnPickup(type, x, y){
    pickups.push({type, x, y, r: 14, alive:true});
  }
  function sprinkle(count){
    for(let i=0;i<count;i++){
      const t = ["rocket","cake","banger","single"][Math.floor(Math.random()*4)];
      spawnPickup(t, rand(140, world.w-140), rand(140, world.h-140));
    }
  }
  if (pickups.length===0) sprinkle(18);

  // Input keyboard
  const keys = {};
  addEventListener('keydown', e=>{ keys[e.key.toLowerCase()] = true; });
  addEventListener('keyup', e=>{ keys[e.key.toLowerCase()] = false; });

  // Touch joystick
  const joy = document.getElementById('joy');
  const joyDot = document.getElementById('joyDot');
  let joyActive=false, joyDx=0, joyDy=0, joyCx=0, joyCy=0;
  function joyPos(ev){
    const t = ev.touches ? ev.touches[0] : ev;
    return {x:t.clientX, y:t.clientY};
  }
  function setJoy(ev){
    const p=joyPos(ev);
    const rect=joy.getBoundingClientRect();
    joyCx = rect.left + rect.width/2;
    joyCy = rect.top + rect.height/2;
    let dx = p.x - joyCx, dy = p.y - joyCy;
    const max=48;
    const mag=Math.hypot(dx,dy);
    if(mag>max){ dx=dx/mag*max; dy=dy/mag*max; }
    joyDx = dx/max; joyDy = dy/max;
    joyDot.style.transform = `translate(${dx}px,${dy}px)`;
  }
  joy.addEventListener('touchstart', e=>{ joyActive=true; setJoy(e); }, {passive:false});
  joy.addEventListener('touchmove', e=>{ if(!joyActive) return; setJoy(e); }, {passive:false});
  joy.addEventListener('touchend', ()=>{ joyActive=false; joyDx=0; joyDy=0; joyDot.style.transform='translate(0,0)'; });

  // Firework particles
  const parts=[];
  function boom(x,y, power, hue){
    const count = Math.floor(70*power);
    for(let i=0;i<count;i++){
      const a = Math.random()*Math.PI*2;
      const sp = rand(60, 230)*power;
      const life = rand(26, 70);
      parts.push({
        x,y, vx: Math.cos(a)*sp, vy: Math.sin(a)*sp,
        life, max: life,
        hue: (hue + rand(-25,25))%360,
        s: rand(1.6,3.8)*power,
        drag: rand(0.86,0.92),
        g: rand(0.55,0.95)
      });
    }
  }
  function trail(x1,y1,x2,y2,hue){
    const steps=12;
    for(let i=0;i<steps;i++){
      const t=i/(steps-1);
      const life = rand(10, 20);
      parts.push({
        x: x1+(x2-x1)*t + rand(-2,2),
        y: y1+(y2-y1)*t + rand(-2,2),
        vx: rand(-20,20), vy: rand(-20,20),
        life, max: life, hue, s: rand(1.0,2.0),
        drag: 0.85, g: 0.35
      });
    }
  }

  // Collision
  function rectCircleColl(rx,ry,rw,rh,cx,cy,cr){
    const nx = Math.max(rx, Math.min(cx, rx+rw));
    const ny = Math.max(ry, Math.min(cy, ry+rh));
    const dx = cx-nx, dy = cy-ny;
    return (dx*dx+dy*dy) <= cr*cr;
  }
  function clamp(v,a,b){return Math.max(a,Math.min(b,v));}

  // Item cycle/use
  const order=["rocket","cake","banger","single"];
  const label = {rocket:"🚀 Pijl", cake:"🧁 Pot", banger:"🧨 Knaller", single:"🎇 Singleshot"};
  function nextItem(){
    let i=order.indexOf(S.selected);
    for(let tries=0;tries<4;tries++){
      i=(i+1)%4;
      const k=order[i];
      if(S.inv[k]>0){ S.selected=k; toast("Item: "+label[k]); return; }
    }
    toast("Geen items. Loop en pak pickups.");
  }
  cycleBtn.onclick = nextItem;

  useBtn.onclick = ()=>{
    const k=S.selected;
    if(S.inv[k]<=0) return toast("Geen "+label[k]+". Pak pickups!");
    S.inv[k]--;

    let addScore=0, addCoins=0;
    const x=player.x, y=player.y;
    const big = S.upgrades.biggerBoom ? 1.25 : 1.0;

    if(k==="rocket"){
      addScore=10; addCoins=2;
      const tx=x, ty=y-190;
      trail(x,y,tx,ty, 35);
      boom(tx,ty, 1.0*big, 35);
    } else if(k==="cake"){
      addScore=14; addCoins=3;
      for(let n=0;n<7;n++){
        const tx = x + rand(-70,70);
        const ty = y - rand(90,220);
        trail(x,y,tx,ty, rand(0,360));
        boom(tx,ty, 0.55*big, rand(0,360));
      }
    } else if(k==="banger"){
      addScore=7; addCoins=1;
      boom(x,y, 0.8*big, 0);
    } else {
      addScore=18; addCoins=4;
      boom(x,y-90, 1.2*big, rand(0,360));
    }

    if(S.upgrades.doubleCoins) addCoins += 1;

    S.score += addScore;
    S.coins += addCoins;
    updateUI();
    toast(`+${addScore} score • +${addCoins} coins`);
  };

  howBtn.onclick = ()=>openPanel('howPanel');
  shopBtn.onclick = ()=>{ renderShop(); openPanel('shopPanel'); };

  toBotBtn.onclick = ()=>{
    if(tg) tg.showAlert("Stuur je verificatie in de bot chat. Daarna krijg je betaalinstructies.");
    else alert("Open via Telegram om door te gaan.");
  };

  function pickUp(p){
    p.alive=false;
    S.inv[p.type] += 1;
    S.coins += 1;
    // auto-select if empty before
    if (invCount()===1) S.selected = p.type;
    updateUI();
    toast(`Gepakt: ${label[p.type]} (+1 coin)`);
  }

  // Periodically respawn pickups
  let respawnT=0;
  function maybeRespawn(dt){
    respawnT += dt;
    const interval = S.upgrades.extraPickups ? 2.8 : 4.8;
    if(respawnT >= interval){
      respawnT = 0;
      const aliveCount = pickups.filter(p=>p.alive).length;
      if(aliveCount < 18){
        const t = ["rocket","cake","banger","single"][Math.floor(Math.random()*4)];
        spawnPickup(t, rand(140, world.w-140), rand(140, world.h-140));
      }
    }
  }

  // Main loop
  let last=performance.now();
  function loop(t){
    const dt = Math.min(0.033, (t-last)/1000);
    last=t;

    maybeRespawn(dt);

    // movement input
    let ax=0, ay=0;
    if(keys['w']||keys['arrowup']) ay-=1;
    if(keys['s']||keys['arrowdown']) ay+=1;
    if(keys['a']||keys['arrowleft']) ax-=1;
    if(keys['d']||keys['arrowright']) ax+=1;
    ax += joyDx; ay += joyDy;

    const mag = Math.hypot(ax,ay) || 1;
    ax/=mag; ay/=mag;

    const baseSpeed = S.upgrades.speedBoost ? 245 : 195;
    player.vx = ax*baseSpeed;
    player.vy = ay*baseSpeed;

    let nx = player.x + player.vx*dt;
    let ny = player.y + player.vy*dt;

    nx = clamp(nx, player.r, world.w-player.r);
    ny = clamp(ny, player.r, world.h-player.r);

    // buildings collision
    for(const b of buildings){
      if(rectCircleColl(b.x,b.y,b.w,b.h,nx,ny,player.r)){
        const tryX = clamp(player.x + player.vx*dt, player.r, world.w-player.r);
        const tryY = clamp(player.y + player.vy*dt, player.r, world.h-player.r);
        if(!rectCircleColl(b.x,b.y,b.w,b.h,tryX,player.y,player.r)) nx=tryX, ny=player.y;
        else if(!rectCircleColl(b.x,b.y,b.w,b.h,player.x,tryY,player.r)) nx=player.x, ny=tryY;
        else { nx=player.x; ny=player.y; }
      }
    }

    // water non-walkable except bridge
    const inWater = (nx>water.x && nx<water.x+water.w && ny>water.y && ny<water.y+water.h);
    const onBridge = (nx>bridge.x && nx<bridge.x+bridge.w && ny>bridge.y && ny<bridge.y+bridge.h);
    if(inWater && !onBridge){ nx=player.x; ny=player.y; }

    player.x=nx; player.y=ny;

    // pickup collisions
    for(const p of pickups){
      if(!p.alive) continue;
      const dx=player.x-p.x, dy=player.y-p.y;
      if(dx*dx+dy*dy < (player.r+p.r)*(player.r+p.r)) pickUp(p);
    }

    // camera
    cam.x = clamp(player.x - W/2, 0, world.w-W);
    cam.y = clamp(player.y - H/2, 0, world.h-H);

    // draw screen bg
    ctx.fillStyle="#070812";
    ctx.fillRect(0,0,W,H);

    // draw world
    ctx.save();
    ctx.translate(-cam.x, -cam.y);

    // roads grid
    ctx.fillStyle="#10121b";
    for(let x=0;x<world.w;x+=240){ ctx.fillRect(x,0,36,world.h); }
    for(let y=0;y<world.h;y+=220){ ctx.fillRect(0,y,world.w,34); }

    // water + bridge
    ctx.fillStyle="rgba(20,80,140,.55)";
    ctx.fillRect(water.x, water.y, water.w, water.h);

    ctx.fillStyle="rgba(210,210,220,.55)";
    ctx.fillRect(bridge.x, bridge.y, bridge.w, bridge.h);
    ctx.fillStyle="rgba(255,255,255,.35)";
    ctx.fillRect(bridge.x+18, bridge.y, 6, bridge.h);
    ctx.fillRect(bridge.x+bridge.w-24, bridge.y, 6, bridge.h);

    // buildings
    for(const b of buildings){
      ctx.fillStyle="rgba(255,255,255,.07)";
      ctx.fillRect(b.x,b.y,b.w,b.h);
      ctx.strokeStyle="rgba(255,255,255,.12)";
      ctx.strokeRect(b.x,b.y,b.w,b.h);
      ctx.fillStyle="rgba(255,255,255,.55)";
      ctx.font="12px Arial";
      ctx.fillText(b.label, b.x+10, b.y+20);
    }

    // POI labels
    ctx.fillStyle="rgba(255,255,255,.75)";
    ctx.font="16px Arial";
    for(const p of poi){
      ctx.fillText(p.text, p.x, p.y);
    }

    // pickups
    for(const p of pickups){
      if(!p.alive) continue;
      ctx.beginPath();
      ctx.fillStyle="rgba(255,255,255,.09)";
      ctx.arc(p.x,p.y,p.r+6,0,Math.PI*2);
      ctx.fill();

      ctx.beginPath();
      ctx.fillStyle="rgba(255,255,255,.14)";
      ctx.arc(p.x,p.y,p.r,0,Math.PI*2);
      ctx.fill();

      ctx.fillStyle="rgba(255,255,255,.9)";
      ctx.font="16px Arial";
      const icon = p.type==="rocket"?"🚀":p.type==="cake"?"🧁":p.type==="banger"?"🧨":"🎇";
      ctx.fillText(icon, p.x-8, p.y+6);
    }

    // player
    ctx.beginPath();
    ctx.fillStyle="rgba(255,180,0,.18)";
    ctx.arc(player.x,player.y,player.r+10,0,Math.PI*2);
    ctx.fill();

    ctx.beginPath();
    ctx.fillStyle="rgba(255,255,255,.85)";
    ctx.arc(player.x,player.y,player.r,0,Math.PI*2);
    ctx.fill();

    // selected item tag
    ctx.fillStyle="rgba(0,0,0,.55)";
    ctx.fillRect(player.x-44, player.y-44, 90, 20);
    ctx.fillStyle="rgba(255,255,255,.92)";
    ctx.font="12px Arial";
    ctx.fillText(label[S.selected] || "—", player.x-40, player.y-30);

    // particles
    for(let i=parts.length-1;i>=0;i--){
      const p=parts[i];
      p.vx*=p.drag;
      p.vy=p.vy*p.drag + p.g;
      p.x += p.vx*dt;
      p.y += p.vy*dt;
      p.life -= 1;
      const tLife = Math.max(0, p.life/p.max);
      const a = Math.min(1, tLife*1.2);
      ctx.beginPath();
      ctx.fillStyle = `hsla(${p.hue}, 100%, 62%, ${a})`;
      ctx.arc(p.x,p.y,p.s*(0.6+0.6*(1-tLife)),0,Math.PI*2);
      ctx.fill();
      if(p.life<=0) parts.splice(i,1);
    }

    ctx.restore();
    requestAnimationFrame(loop);
  }
  requestAnimationFrame(loop);

  // First item tip
  if(invCount()===0){
    toast("Loop rond en pak pickups 🎒");
  } else {
    toast("Item: " + label[S.selected]);
  }
</script>
</body>
</html>

