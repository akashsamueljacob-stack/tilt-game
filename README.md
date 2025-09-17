<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="utf-8" />
<meta name="viewport" content="width=device-width,initial-scale=1,maximum-scale=1,user-scalable=no" />
<title>Tilt Ball — Mobile</title>
<meta name="theme-color" content="#0f1220" />
<style>
  :root {
    --bg: #0f1220; --fg:#e9eefb; --muted:#9aa7c7; --accent:#7ec8ff; --good:#65e890; --bad:#ff6584; --warn:#ffd166;
    --wall:#2b3355; --mover:#3f4977; --holeglow:#ff658433; --border:rgba(255,255,255,0.18);
  }
  /* Light theme */
  .theme-light {
    --bg:#f6f8ff; --fg:#0d1533; --muted:#516190; --accent:#216cff; --good:#18a563; --bad:#e03b5b; --warn:#c38b00;
    --wall:#caddff; --mover:#b4c9ff; --holeglow:#e03b5b33; --border:rgba(5,17,54,0.15);
  }
  /* Neon theme */
  .theme-neon {
    --bg:#090b10; --fg:#e6f7ff; --muted:#96d8ff; --accent:#69f0ff; --good:#00ffa3; --bad:#ff4d9a; --warn:#ffe066;
    --wall:#1b1f3a; --mover:#262e66; --holeglow:#ff4d9a44; --border:rgba(105,240,255,0.3);
  }

  html, body { height:100%; margin:0; background:var(--bg); color:var(--fg); font-family: system-ui, -apple-system, Segoe UI, Roboto, Helvetica, Arial, sans-serif; }
  #wrap { position:fixed; inset:0; display:grid; grid-template-rows:auto 1fr auto; }
  header, footer { padding:10px 12px; display:flex; align-items:center; justify-content:space-between; gap:10px; }
  header { backdrop-filter: blur(6px); background: rgba(10,12,22,0.5); border-bottom: 1px solid rgba(255,255,255,0.06); }
  footer { backdrop-filter: blur(6px); background: rgba(10,12,22,0.5); border-top: 1px solid rgba(255,255,255,0.06); }
  #hud { display:flex; gap:14px; align-items:center; font-weight:600; }
  .tag { padding:6px 10px; border-radius:999px; background:rgba(255,255,255,0.06); color:var(--fg); font-size:13px; }
  .btn { -webkit-tap-highlight-color: transparent; padding:10px 14px; border-radius:10px; background: rgba(255,255,255,0.08);
    color: var(--fg); border: 1px solid rgba(255,255,255,0.12); font-weight: 600; font-size: 14px; cursor: pointer; }
  .btn:active { transform: translateY(1px); }
  .btn.primary { background: linear-gradient(180deg, #2a6bff, #2052d6); border-color: rgba(0,0,0,0.15); }
  .btn.good { background: linear-gradient(180deg, #3bd480, #2bad68); }
  .btn.bad { background: linear-gradient(180deg, #ff6a88, #e35270); }
  select { padding:8px 10px; border-radius:10px; border:1px solid rgba(255,255,255,0.18); background: rgba(255,255,255,0.06); color: var(--fg); font-weight:600; }
  #canvas { width:100%; height:100%; display:block; touch-action:none; }
  #overlay { position:absolute; inset:0; display:none; place-items:center; padding:18px;
    background: radial-gradient(1200px 800px at 50% 40%, rgba(255,255,255,0.06), rgba(0,0,0,0.75)); text-align:center; }
  #overlay.show { display:grid; }
  #card { max-width:580px; background: rgba(8,10,18,0.75); border: 1px solid rgba(255,255,255,0.1);
    padding:24px; border-radius:16px; box-shadow: 0 10px 50px rgba(0,0,0,0.45); }
  h1 { margin:0 0 6px; font-size:26px; }
  p { margin:8px 0; color:var(--muted); line-height:1.45; }
  .row { display:flex; gap:10px; justify-content:center; flex-wrap:wrap; margin-top:14px; }
  .grid { display:grid; grid-template-columns: 1fr 1fr; gap:10px; margin-top:10px; }
  .kbd { font-family: ui-monospace, SFMono-Regular, Menlo, Consolas, monospace; background: rgba(255,255,255,0.08); padding:2px 6px; border-radius:6px; border: 1px solid rgba(255,255,255,0.12); }
</style>
</head>
<body>
<div id="wrap" class="">
  <header>
    <div id="hud">
      <span class="tag">Pack <span id="packTag">Starter</span></span>
      <span class="tag">Level <span id="levelTag">1</span></span>
      <span class="tag">Best <span id="bestTag">1</span></span>
      <span class="tag">Attempts <span id="triesTag">0</span></span>
    </div>
    <div style="display:flex; gap:8px; align-items:center;">
      <select id="selPack" title="Level Pack">
        <option value="starter">Starter</option>
        <option value="trickster">Trickster</option>
        <option value="endless">Endless</option>
      </select>
      <select id="selTheme" title="Theme">
        <option value="dark">Dark</option>
        <option value="light">Light</option>
        <option value="neon">Neon</option>
      </select>
      <button id="btnPerm" class="btn primary">Enable Gyro</button>
      <button id="btnCal" class="btn">Calibrate</button>
      <button id="btnPause" class="btn">Pause</button>
      <button id="btnReset" class="btn">Reset</button>
    </div>
  </header>

  <main style="position:relative;">
    <canvas id="canvas"></canvas>
    <div id="overlay" class="show" aria-live="polite">
      <div id="card">
        <h1>Tilt Ball</h1>
        <p>Tilt to roll the ball <span style="color:var(--accent);">●</span> into the green ring <span style="color:var(--good);">◎</span>. Avoid pink holes and walls. Each level gets trickier.</p>
        <div class="grid">
          <div>
            <label>Level Pack</label><br/>
            <select id="selPackStart">
              <option value="starter">Starter</option>
              <option value="trickster">Trickster</option>
              <option value="endless">Endless</option>
            </select>
          </div>
          <div>
            <label>Theme</label><br/>
            <select id="selThemeStart">
              <option value="dark">Dark</option>
              <option value="light">Light</option>
              <option value="neon">Neon</option>
            </select>
          </div>
        </div>
        <p style="margin-top:8px;"><strong>iOS:</strong> tap <em>Enable Gyro</em> and allow motion access. Desktop: use <span class="kbd">WASD</span> or arrows.</p>
        <div class="row">
          <button id="btnPlay" class="btn good">Play</button>
          <button id="btnHow" class="btn">How to Play</button>
        </div>
      </div>
    </div>
  </main>

  <footer>
    <div style="font-size:12px; color:var(--muted);">Tap Calibrate to set your neutral tilt. Settings persist. </div>
    <div style="display:flex; gap:8px;">
      <button id="btnMute" class="btn">Sound On</button>
    </div>
  </footer>
</div>

<script>
(() => {
  const $ = sel => document.querySelector(sel);
  const canvas = $('#canvas'); const ctx = canvas.getContext('2d', { alpha:false });
  const overlay = $('#overlay');

  // Controls
  const btnPerm  = $('#btnPerm');
  const btnCal   = $('#btnCal');
  const btnPause = $('#btnPause');
  const btnReset = $('#btnReset');
  const btnPlay  = $('#btnPlay');
  const btnHow   = $('#btnHow');
  const btnMute  = $('#btnMute');
  const selPackStart = $('#selPackStart');
  const selThemeStart = $('#selThemeStart');
  const selPack = $('#selPack');
  const selTheme = $('#selTheme');

  // HUD
  const packTag  = $('#packTag');
  const levelTag = $('#levelTag');
  const bestTag  = $('#bestTag');
  const triesTag = $('#triesTag');

  // Audio
  let audioCtx; let muted = false;
  function ping(freq=660, dur=0.08, gain=0.04) {
    if (muted) return;
    try {
      if (!audioCtx) audioCtx = new (window.AudioContext || window.webkitAudioContext)();
      const t0 = audioCtx.currentTime;
      const o = audioCtx.createOscillator();
      const g = audioCtx.createGain();
      o.type = 'sine'; o.frequency.value = freq;
      g.gain.value = gain; o.connect(g); g.connect(audioCtx.destination);
      o.start(t0); o.stop(t0 + dur);
    } catch {}
  }
  btnMute.addEventListener('click', () => {
    muted = !muted;
    btnMute.textContent = muted ? 'Sound Off' : 'Sound On';
    if (!muted) ping(880,0.06,0.03);
  });

  // DPR fit
  function fit() {
    const dpr = Math.max(1, Math.min(2, window.devicePixelRatio || 1));
    const w = canvas.clientWidth, h = canvas.clientHeight;
    canvas.width = Math.round(w*dpr); canvas.height = Math.round(h*dpr);
    ctx.setTransform(dpr,0,0,dpr,0,0);
  }
  window.addEventListener('resize', fit, { passive:true }); fit();

  // Packs & themes
  const THEMES = {
    dark:   '',
    light:  'theme-light',
    neon:   'theme-neon'
  };
  const PACKS = {
    starter: {
      name: 'Starter',
      wallK: n => Math.min(3 + Math.floor(n*0.5), 12),
      holeK: n => Math.min(1 + Math.floor(n*0.4), 8),
      moverK: n => Math.min(Math.floor(n/4), 4),
      goalR: n => Math.max(26 - n, 12),
      ballR: n => 10 + Math.max(0, 3 - Math.floor(n/6)),
      gravityK: n => 0.012 + Math.min(n*0.0016, 0.016),
      damping: n => 0.991 - Math.min(n*0.0005, 0.007),
      maxSpeed: n => 6 + Math.min(n*0.25, 5),
    },
    trickster: {
      name: 'Trickster',
      wallK: n => Math.min(4 + Math.floor(n*0.8), 20),
      holeK: n => Math.min(2 + Math.floor(n*0.7), 14),
      moverK: n => Math.min(Math.floor(n/3), 8),
      goalR: n => Math.max(24 - n, 10),
      ballR: n => 10 + Math.max(0, 2 - Math.floor(n/5)),
      gravityK: n => 0.013 + Math.min(n*0.0022, 0.02),
      damping: n => 0.990 - Math.min(n*0.0008, 0.009),
      maxSpeed: n => 6 + Math.min(n*0.35, 7),
    },
    endless: {
      name: 'Endless',
      wallK: n => Math.min(3 + Math.floor(n*0.7), 30),
      holeK: n => Math.min(1 + Math.floor(n*0.8), 26),
      moverK: n => Math.min(1 + Math.floor(n*0.5), 12),
      goalR: n => Math.max(26 - Math.floor(n*1.2), 8),
      ballR: n => Math.max(12 - Math.floor(n/7), 7),
      gravityK: n => 0.014 + Math.min(n*0.0026, 0.025),
      damping: n => 0.990 - Math.min(n*0.0010, 0.010),
      maxSpeed: n => 7 + Math.min(n*0.45, 9),
    }
  };

  // Persisted state
  const store = {
    theme: localStorage.getItem('tilt_theme') || 'dark',
    pack:  localStorage.getItem('tilt_pack')  || 'starter',
    level: parseInt(localStorage.getItem('tilt_level') || '1',10),
    best:  parseInt(localStorage.getItem('tilt_best')  || '1',10),
  };

  // Game state
  const state = {
    running:false, paused:true, tries:0,
    width: () => canvas.clientWidth, height: () => canvas.clientHeight,
    ball:{x:60,y:0,r:10,vx:0,vy:0},
    goal:{x:0,y:0,r:24},
    walls:[], holes:[], movers:[],
    accel:{x:0,y:0}, gravityK:0.015, damping:0.992, maxSpeed:8, boundsPad:6,
    input:{left:false,right:false,up:false,down:false},
    calibrated:{beta:0,gamma:0}, hasPermission:false,
  };

  // UI bind
  function applyTheme(th=store.theme) {
    document.getElementById('wrap').className = THEMES[th] || '';
    localStorage.setItem('tilt_theme', th);
    selTheme.value = th; selThemeStart.value = th;
  }
  function applyPack(pk=store.pack) {
    packTag.textContent = PACKS[pk]?.name || 'Starter';
    localStorage.setItem('tilt_pack', pk);
    selPack.value = pk; selPackStart.value = pk;
    // reset level counters per pack key (keep separate best per pack if you want)
  }
  function syncHUD() {
    levelTag.textContent = store.level;
    bestTag.textContent  = store.best;
    triesTag.textContent = state.tries;
    packTag.textContent  = PACKS[store.pack].name;
  }
  applyTheme(store.theme); applyPack(store.pack); syncHUD();

  selTheme.addEventListener('change', e => { store.theme = e.target.value; applyTheme(store.theme); });
  selPack.addEventListener('change', e => { store.pack = e.target.value; applyPack(store.pack); resetLevel(false,true); });

  // Permission (iOS)
  async function requestMotionPermission() {
    const any = window.DeviceMotionEvent || window.DeviceOrientationEvent;
    if (!any) { alert('Motion sensors not available on this device.'); return false; }
    const needs = typeof DeviceMotionEvent !== 'undefined' && typeof DeviceMotionEvent.requestPermission === 'function';
    try {
      if (needs) {
        const res = await DeviceMotionEvent.requestPermission();
        state.hasPermission = (res === 'granted');
      } else { state.hasPermission = true; }
    } catch { state.hasPermission = false; }
    if (!state.hasPermission) alert('Motion access was not granted. Keyboard controls are available.');
    return state.hasPermission;
  }
  btnPerm.addEventListener('click', async ()=>{ await requestMotionPermission(); if (state.hasPermission) { ping(900,0.06,0.03); overlay.classList.remove('show'); state.paused=false; state.running=true; } });

  // Orientation + keyboard
  let lastBeta=0, lastGamma=0;
  window.addEventListener('deviceorientation', e => {
    lastBeta  = (e.beta  ?? 0) - state.calibrated.beta;
    lastGamma = (e.gamma ?? 0) - state.calibrated.gamma;
  });
  function onKey(e, down) {
    const k=e.key.toLowerCase();
    if(['arrowleft','a'].includes(k)) state.input.left=down;
    if(['arrowright','d'].includes(k)) state.input.right=down;
    if(['arrowup','w'].includes(k)) state.input.up=down;
    if(['arrowdown','s'].includes(k)) state.input.down=down;
    if(down && k===' ') resetLevel();
    if(down && k==='p') togglePause();
  }
  window.addEventListener('keydown', e=>onKey(e,true));
  window.addEventListener('keyup',   e=>onKey(e,false));

  btnCal.addEventListener('click', ()=>{ state.calibrated.beta=lastBeta; state.calibrated.gamma=lastGamma; ping(700,0.05,0.03); });
  btnPause.addEventListener('click', togglePause);
  function togglePause() {
    state.paused = !state.paused;
    overlay.classList.toggle('show', state.paused);
  }

  btnReset.addEventListener('click', ()=> resetLevel(false));
  btnHow.addEventListener('click', ()=> alert(
`How to play:
• Tilt your phone to roll the ball into the green ring.
• Avoid holes and walls (moving bars are dangerous).
• Tap "Enable Gyro" (iOS) and allow motion access.
• Tap "Calibrate" while holding your neutral tilt.
• Choose Level Pack and Theme from the header or start screen.
• Keyboard: W/A/S/D or Arrows, Space to reset, P to pause.`
  ));
  btnPlay.addEventListener('click', ()=>{
    store.pack = selPackStart.value; applyPack(store.pack);
    store.theme = selThemeStart.value; applyTheme(store.theme);
    overlay.classList.remove('show'); state.paused=false; state.running=true; ping(760,0.05,0.03);
  });

  // RNG helpers
  const rand = (a,b)=>Math.random()*(b-a)+a;
  const irand = (a,b)=>Math.floor(rand(a,b));
  const clamp = (v,a,b)=>Math.max(a,Math.min(b,v));

  // Level generation (pack-aware)
  function generateLevel(n) {
    const w=state.width(), h=state.height(), pad=40;
    const pack = PACKS[store.pack];

    state.ball.r = pack.ballR(n);
    state.ball.x = pad; state.ball.y = h/2; state.ball.vx = state.ball.vy = 0;

    state.goal.r = pack.goalR(n);
    state.goal.x = w - pad; state.goal.y = h/2;

    state.gravityK = pack.gravityK(n);
    state.damping  = pack.damping(n);
    state.maxSpeed = pack.maxSpeed(n);

    // safe corridor to avoid unwinnable spawns
    const safeHalf = Math.max(36 - Math.floor(n/2), 16);
    const safeY0 = h/2 - safeHalf, safeY1 = h/2 + safeHalf;

    // Walls
    state.walls = [];
    for (let i=0;i<pack.wallK(n);i++){
      let tries=0;
      while(tries++<60){
        const ww=irand(40,160), hh=irand(14,32);
        const x=irand(pad*2, w - pad*2 - ww);
        const y=irand(20, h - 20 - hh);
        if(!(y<safeY1 && y+hh>safeY0)) { state.walls.push({x,y,w:ww,h:hh}); break; }
      }
    }

    // Holes
    state.holes=[];
    const minHoleR = Math.max(10, 18 - Math.floor(n/4));
    const maxHoleR = Math.max(minHoleR+2, 22 - Math.floor(n/5));
    for (let i=0;i<pack.holeK(n);i++){
      let tries=0;
      while(tries++<60){
        const r=rand(minHoleR,maxHoleR);
        const x=rand(pad*1.2 + r, w - pad*1.2 - r);
        const y=rand(20 + r, h - 20 - r);
        if(y>safeY1 + r || y<safeY0 - r){ state.holes.push({x,y,r}); break; }
      }
    }

    // Movers (vertical bars)
    state.movers=[];
    for (let i=0;i<pack.moverK(n);i++){
      const ww=irand(20,26), hh=irand(70,130);
      const leftSide = Math.random()<0.5;
      const x = leftSide ? irand(pad*1.5, w/3) : irand(2*w/3, w-pad*1.5 - ww);
      const baseY = (Math.random()<0.5) ? irand(20, safeY0 - hh - 10) : irand(safeY1 + 10, h - 20 - hh);
      const amp = irand(20, 90);
      const speed = rand(0.6, 1.8) * (Math.random()<0.5?-1:1);
      state.movers.push({x, y: baseY, w: ww, h: hh, baseY, amp, speed, t: rand(0, Math.PI*2)});
    }

    localStorage.setItem('tilt_level', String(store.level));
    if (store.level > store.best) { store.best = store.level; localStorage.setItem('tilt_best', String(store.best)); }
    syncHUD();
  }

  function circleRectCollide(cx,cy,cr, rx,ry,rw,rh){
    const nearestX=clamp(cx,rx,rx+rw); const nearestY=clamp(cy,ry,ry+rh);
    const dx=cx-nearestX, dy=cy-nearestY;
    return (dx*dx+dy*dy)<=cr*cr;
  }
  function resolveCircleRect(ball, rect){
    const bx=ball.x, by=ball.y, br=ball.r;
    const nearestX=clamp(bx,rect.x,rect.x+rect.w); const nearestY=clamp(by,rect.y,rect.y+rect.h);
    const dx=bx-nearestX, dy=by-nearestY; const d2=dx*dx+dy*dy;
    if(d2>br*br) return false;
    const d=Math.sqrt(d2) || 0.00001, overlap=br-d, nx=dx/d, ny=dy/d;
    ball.x += nx*overlap; ball.y += ny*overlap;
    const dot=ball.vx*nx + ball.vy*ny;
    if(dot<0){ ball.vx -= 1.4*dot*nx; ball.vy -= 1.4*dot*ny; } else { ball.vx*=0.98; ball.vy*=0.98; }
    return true;
  }

  function resetLevel(incrementTries=true, keepLevel=false){
    if(incrementTries) state.tries++;
    if(!keepLevel) store.level = Math.max(1, store.level);
    generateLevel(store.level);
    ping(500,0.05,0.03); syncHUD();
  }

  // Update & draw
  let lastTime=performance.now();
  function update(dt){
    const pack = PACKS[store.pack];

    const targetAx = (0 + (lastGamma||0)) * pack.gravityK(store.level);
    const targetAy = (0 + (lastBeta ||0)) * pack.gravityK(store.level);
    state.accel.x = state.accel.x*0.7 + targetAx*0.3;
    state.accel.y = state.accel.y*0.7 + targetAy*0.3;

    const kBoost=0.22;
    if(state.input.left)  state.accel.x -= kBoost * pack.gravityK(store.level) * 8;
    if(state.input.right) state.accel.x += kBoost * pack.gravityK(store.level) * 8;
    if(state.input.up)    state.accel.y -= kBoost * pack.gravityK(store.level) * 8;
    if(state.input.down)  state.accel.y += kBoost * pack.gravityK(store.level) * 8;

    state.ball.vx = (state.ball.vx + state.accel.x * dt) * Math.pow(pack.damping(store.level), dt/16);
    state.ball.vy = (state.ball.vy + state.accel.y * dt) * Math.pow(pack.damping(store.level), dt/16);

    const sp=Math.hypot(state.ball.vx,state.ball.vy), maxSp=pack.maxSpeed(store.level);
    if(sp>maxSp){ const s=maxSp/sp; state.ball.vx*=s; state.ball.vy*=s; }

    state.ball.x += state.ball.vx; state.ball.y += state.ball.vy;

    const pad=state.boundsPad;
    if(state.ball.x < pad + state.ball.r){ state.ball.x = pad + state.ball.r; state.ball.vx *= -0.6; ping(300,0.02,0.02); }
    if(state.ball.x > state.width()-pad - state.ball.r){ state.ball.x = state.width()-pad - state.ball.r; state.ball.vx *= -0.6; ping(300,0.02,0.02); }
    if(state.ball.y < pad + state.ball.r){ state.ball.y = pad + state.ball.r; state.ball.vy *= -0.6; ping(300,0.02,0.02); }
    if(state.ball.y > state.height()-pad - state.ball.r){ state.ball.y = state.height()-pad - state.ball.r; state.ball.vy *= -0.6; ping(300,0.02,0.02); }

    for (const m of state.movers){ m.t += (dt/1000)*m.speed; m.y = m.baseY + Math.sin(m.t)*m.amp; }
    for (const r of state.walls)  resolveCircleRect(state.ball, r);
    for (const r of state.movers) resolveCircleRect(state.ball, r);

    for (const ho of state.holes){
      const dx=state.ball.x-ho.x, dy=state.ball.y-ho.y, d=Math.hypot(dx,dy);
      if(d < ho.r*2.2){ const pull=(1 - d/(ho.r*2.2))*0.5; state.ball.vx -= (dx/d)*pull; state.ball.vy -= (dy/d)*pull; }
      if(d < ho.r-2){ ping(220,0.2,0.05); resetLevel(true,true); return; }
    }

    const dg=Math.hypot(state.ball.x - state.goal.x, state.ball.y - state.goal.y);
    if(dg < state.goal.r){
      store.level++;
      localStorage.setItem('tilt_level', String(store.level));
      if(store.level>store.best){ store.best=store.level; localStorage.setItem('tilt_best', String(store.best)); }
      syncHUD();
      ping(880,0.08,0.035); setTimeout(()=>ping(1040,0.08,0.035),90); setTimeout(()=>ping(1320,0.1,0.035),180);
      generateLevel(store.level);
    }
  }

  function draw(){
    const w=canvas.clientWidth, h=canvas.clientHeight;
    ctx.fillStyle = getComputedStyle(document.documentElement).getPropertyValue('--bg')||'#0f1220';
    ctx.fillRect(0,0,w,h);

    ctx.save(); ctx.globalAlpha=0.15; ctx.beginPath(); const step=28;
    for(let x=0;x<=w;x+=step){ ctx.moveTo(x,0); ctx.lineTo(x,h); }
    for(let y=0;y<=h;y+=step){ ctx.moveTo(0,y); ctx.lineTo(w,y); }
    ctx.strokeStyle='#ffffff'; ctx.lineWidth=1; ctx.stroke(); ctx.restore();

    // Goal
    ctx.beginPath(); ctx.lineWidth=5; ctx.strokeStyle=getComputedStyle(document.documentElement).getPropertyValue('--good')||'#65e890';
    ctx.arc(state.goal.x, state.goal.y, state.goal.r, 0, Math.PI*2); ctx.stroke();

    // Walls
    ctx.fillStyle = getComputedStyle(document.documentElement).getPropertyValue('--wall')||'#2b3355';
    for(const r of state.walls) ctx.fillRect(r.x,r.y,r.w,r.h);

    // Movers
    ctx.fillStyle = getComputedStyle(document.documentElement).getPropertyValue('--mover')||'#3f4977';
    for(const r of state.movers) ctx.fillRect(r.x,r.y,r.w,r.h);

    // Holes
    const holeStroke = getComputedStyle(document.documentElement).getPropertyValue('--bad')||'#ff6584';
    for(const ho of state.holes){
      const g=ctx.createRadialGradient(ho.x,ho.y,2,ho.x,ho.y,ho.r);
      g.addColorStop(0,'#000'); g.addColorStop(1,getComputedStyle(document.documentElement).getPropertyValue('--holeglow')||'#ff658433');
      ctx.fillStyle=g; ctx.beginPath(); ctx.arc(ho.x,ho.y,ho.r,0,Math.PI*2); ctx.fill();
      ctx.strokeStyle=holeStroke; ctx.globalAlpha=0.5; ctx.lineWidth=2; ctx.stroke(); ctx.globalAlpha=1;
    }

    // Ball
    ctx.beginPath(); ctx.fillStyle=getComputedStyle(document.documentElement).getPropertyValue('--accent')||'#7ec8ff';
    ctx.arc(state.ball.x,state.ball.y,state.ball.r,0,Math.PI*2); ctx.fill();

    // Border
    ctx.strokeStyle=getComputedStyle(document.documentElement).getPropertyValue('--border')||'rgba(255,255,255,0.18)';
    ctx.lineWidth=2; ctx.strokeRect(4,4,w-8,h-8);
  }

  function loop(t){
    const dt=Math.min(40, t - lastTime); lastTime=t;
    if(state.running && !state.paused) update(dt);
    draw(); requestAnimationFrame(loop);
  }
  requestAnimationFrame(loop);

  // Init a level
  function firstInit(){
    overlay.classList.add('show'); state.paused=true; state.running=false;
    generateLevel(store.level);
  }
  firstInit();

  // Orientation/layout changes
  window.addEventListener('pointerdown', ()=>{ if (!state.running && !overlay.classList.contains('show')) state.running=true; }, { passive:true });
  window.addEventListener('orientationchange', ()=> setTimeout(()=>{ fit(); generateLevel(store.level); }, 250));
})();
</script>
</body>
</html>
