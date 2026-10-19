
<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width,initial-scale=1,maximum-scale=1,user-scalable=no">
<title>Highway Racer 3D</title>
<style>
*{box-sizing:border-box}
html,body{margin:0;width:100%;height:100%;overflow:hidden;background:#101820;font-family:Arial,sans-serif;touch-action:none}
canvas{display:block;width:100vw;height:100dvh}
#hud{position:fixed;top:12px;left:12px;right:12px;display:flex;justify-content:space-between;gap:8px;color:white;font-weight:800;text-shadow:0 2px 5px #000;pointer-events:none}
.pill{background:#07131dcc;border:1px solid #ffffff55;border-radius:12px;padding:9px 12px;font-size:14px}
#overlay{position:fixed;inset:0;display:flex;align-items:center;justify-content:center;background:#06101ab8;color:white;text-align:center;padding:22px}
.panel{width:min(440px,100%);background:linear-gradient(145deg,#142a3c,#08121d);border:1px solid #52eaff88;border-radius:22px;padding:24px 20px;box-shadow:0 0 30px #00d9ff30}
h1{font-size:34px;margin:0 0 8px;color:#62efff;text-transform:uppercase}
p{line-height:1.5;color:#d7e5ee}
button{border:0;border-radius:14px;padding:14px 20px;font-size:17px;font-weight:800;color:#06131c;background:linear-gradient(90deg,#67f4ff,#65ffa6);cursor:pointer}
.small{font-size:12px;color:#a9c1d0}
#controls{position:fixed;left:0;right:0;bottom:18px;display:flex;justify-content:space-between;padding:0 20px;pointer-events:none}
.ctrl{pointer-events:auto;width:76px;height:66px;border-radius:18px;background:#07131dcc;color:white;border:2px solid #ffffff80;font-size:27px;display:grid;place-items:center;user-select:none}
#rightControls{display:flex;gap:12px}
@media(min-width:800px){.ctrl{width:90px;height:72px}}
</style>
</head>
<body>
<canvas id="game"></canvas>

<div id="hud">
  <div class="pill">🏁 SCORE <span id="score">0</span></div>
  <div class="pill">⚡ <span id="speed">0</span> km/h</div>
  <div class="pill">🪙 <span id="coins">0</span></div>
</div>

<div id="controls">
  <button class="ctrl" id="left">◀</button>
  <div id="rightControls">
    <button class="ctrl" id="brake">▼</button>
    <button class="ctrl" id="right">▶</button>
  </div>
</div>

<div id="overlay">
  <div class="panel">
    <h1>Highway Racer</h1>
    <p>Avoid traffic, collect coins and survive as long as you can.</p>
    <p>Use ◀ ▶ buttons or tap the left/right side of the screen to steer.</p>
    <button id="start">START RACING</button>
    <p class="small">Mobile-friendly single-file game</p>
  </div>
</div>

<script>
(()=>{
const canvas=document.getElementById('game');
const ctx=canvas.getContext('2d');

const scoreEl=document.getElementById('score');
const speedEl=document.getElementById('speed');
const coinsEl=document.getElementById('coins');

let W=0,H=0,dpr=1;
let running=false,last=0,elapsed=0;
let score=0,coins=0,speed=.48;
let playerX=0,steer=0,brake=false;
let traffic=[],pickups=[],spawn=0,coinSpawn=0,roadOffset=0;

const colors=['#ff4757','#ffbd2e','#29a8ff','#a66bff','#f4f7fb','#36d399'];

function resize(){
  dpr=Math.min(devicePixelRatio||1,2);
  W=innerWidth;
  H=innerHeight;
  canvas.width=Math.floor(W*dpr);
  canvas.height=Math.floor(H*dpr);
  ctx.setTransform(dpr,0,0,dpr,0,0);
}
addEventListener('resize',resize);
resize();

function roadAt(y){
  let t=(y-H*.22)/(H*.82);
  t=Math.max(0,Math.min(1,t));
  return {
    w:W*(.12+.78*t),
    cx:W*.5+Math.sin(elapsed*.12+t*2)*W*.025*t,
    t
  };
}

function poly(points,color){
  ctx.fillStyle=color;
  ctx.beginPath();
  ctx.moveTo(points[0][0],points[0][1]);
  for(let i=1;i<points.length;i++)ctx.lineTo(points[i][0],points[i][1]);
  ctx.closePath();
  ctx.fill();
}

function drawCar(x,y,w,h,color,player=false){
  ctx.fillStyle='#0008';
  ctx.beginPath();
  ctx.ellipse(x,y+h*.42,w*.62,h*.14,0,0,Math.PI*2);
  ctx.fill();

  poly([
    [x-w*.48,y+h*.35],
    [x-w*.42,y-h*.25],
    [x-w*.26,y-h*.48],
    [x+w*.26,y-h*.48],
    [x+w*.42,y-h*.25],
    [x+w*.48,y+h*.35]
  ],color);

  poly([
    [x-w*.29,y-h*.22],
    [x-w*.19,y-h*.42],
    [x+w*.19,y-h*.42],
    [x+w*.29,y-h*.22]
  ],'#a9e8ff');

  poly([
    [x-w*.34,y+h*.02],
    [x+w*.34,y+h*.02],
    [x+w*.39,y+h*.25],
    [x-w*.39,y+h*.25]
  ],player?'#142c42':'#263746');

  ctx.fillStyle='#12151b';
  ctx.fillRect(x-w*.53,y-h*.05,w*.12,h*.32);
  ctx.fillRect(x+w*.41,y-h*.05,w*.12,h*.32);

  ctx.fillStyle=player?'#dfffff':'#ffedaa';
  ctx.fillRect(x-w*.36,y+h*.23,w*.17,h*.055);
  ctx.fillRect(x+w*.19,y+h*.23,w*.17,h*.055);

  if(player){
    ctx.fillStyle='#00eaff';
    ctx.fillRect(x-w*.035,y-h*.05,w*.07,h*.26);
  }
}

function drawCoin(x,y,r){
  ctx.fillStyle='#ffb800';
  ctx.beginPath();
  ctx.arc(x,y,r,0,Math.PI*2);
  ctx.fill();

  ctx.strokeStyle='#fff2a6';
  ctx.lineWidth=Math.max(1,r*.18);
  ctx.stroke();

  ctx.fillStyle='#fff3ae';
  ctx.font=`bold ${r*1.25}px Arial`;
  ctx.textAlign='center';
  ctx.textBaseline='middle';
  ctx.fillText('$',x,y+1);
}

function background(){
  const g=ctx.createLinearGradient(0,0,0,H);
  g.addColorStop(0,'#6cc9ed');
  g.addColorStop(.46,'#d5f2ee');
  g.addColorStop(.47,'#397e38');
  g.addColorStop(1,'#174f28');
  ctx.fillStyle=g;
  ctx.fillRect(0,0,W,H);

  poly([
    [0,H*.42],[W*.12,H*.29],[W*.25,H*.39],
    [W*.43,H*.25],[W*.58,H*.38],[W*.78,H*.28],
    [W,H*.4],[W,H*.55],[0,H*.55]
  ],'#589b68');

  for(let side of [-1,1]){
    for(let i=0;i<13;i++){
      const y=((i/13*H*.9+roadOffset*H*.35)%(H*.88))+H*.08;
      const t=Math.max(.1,(y-H*.2)/(H*.8));
      const r=7+t*20;
      const road=roadAt(y);
      const x=road.cx+side*(road.w*.58+r*1.2);

      ctx.fillStyle='#69452b';
      ctx.fillRect(x-r*.13,y-r*.05,r*.26,r*.7);

      ctx.fillStyle=i%2?'#146b37':'#1b843e';
      ctx.beginPath();
      ctx.arc(x,y-r*.35,r*.62,0,Math.PI*2);
      ctx.fill();
    }
  }
}

function drawRoad(){
  const top=H*.2,bottom=H*1.08;
  let a=roadAt(top),b=roadAt(bottom);

  poly([
    [a.cx-a.w*.56,top],[a.cx+a.w*.56,top],
    [b.cx+b.w*.56,bottom],[b.cx-b.w*.56,bottom]
  ],'#414852');

  poly([
    [a.cx-a.w*.62,top],[a.cx-a.w*.56,top],
    [b.cx-b.w*.56,bottom],[b.cx-b.w*.62,bottom]
  ],'#e9e9d9');

  poly([
    [a.cx+a.w*.56,top],[a.cx+a.w*.62,top],
    [b.cx+b.w*.62,bottom],[b.cx+b.w*.56,bottom]
  ],'#e9e9d9');

  for(let i=0;i<14;i++){
    const f=((i/14+roadOffset)%1);
    const y=top+f*(bottom-top);
    const r=roadAt(y);
    const dashH=5+f*22;

    for(let lane=1;lane<=2;lane++){
      const x=r.cx-r.w*.56+r.w*1.12*lane/3;
      ctx.fillStyle='#f5f5dc';
      ctx.fillRect(x-(1+f*2),y,2+f*4,dashH);
    }

    if(i%2===0){
      ctx.fillStyle='#ffffff';
      ctx.fillRect(r.cx-r.w*.56,y,Math.max(2,f*5),dashH*.55);
      ctx.fillRect(r.cx+r.w*.56-Math.max(2,f*5),y,Math.max(2,f*5),dashH*.55);
    }
  }
}

function spawnTraffic(){
  const lane=Math.floor(Math.random()*3)-1;
  traffic.push({
    lane,
    color:colors[Math.floor(Math.random()*colors.length)],
    z:0
  });
}

function spawnCoin(){
  const lane=Math.floor(Math.random()*3)-1;
  pickups.push({lane,z:0,got:false});
}

function project(lane,z){
  const y=H*.2+z*H*.8;
  const r=roadAt(y);
  return {
    x:r.cx+lane*r.w*.34,
    y,
    w:18+z*27,
    h:32+z*44,
    r
  };
}

function reset(){
  elapsed=0;
  score=0;
  coins=0;
  speed=.48;
  playerX=0;
  traffic=[];
  pickups=[];
  spawn=.5;
  coinSpawn=1;
  roadOffset=0;
  running=true;

  document.getElementById('overlay').style.display='none';
  last=performance.now();
  requestAnimationFrame(loop);
}

function endGame(){
  running=false;
  document.getElementById('overlay').style.display='flex';
  document.querySelector('#overlay h1').textContent='CRASH!';
  document.querySelector('#overlay p').innerHTML=
    `Score: <b>${Math.floor(score)}</b> &nbsp; Coins: <b>${coins}</b><br>Try again and beat your score!`;
  document.getElementById('start').textContent='RACE AGAIN';
}

function loop(now){
  if(!running)return;

  const dt=Math.min((now-last)/1000,.04);
  last=now;
  elapsed+=dt;

  if(brake)speed=Math.max(.25,speed-.8*dt);
  else speed=Math.min(.92,speed+.08*dt);

  roadOffset=(roadOffset+dt*speed*.42)%1;
  score+=dt*speed*20;

  playerX+=steer*dt*1.55;
  playerX=Math.max(-.76,Math.min(.76,playerX));

  spawn-=dt;
  if(spawn<=0){
    spawnTraffic();
    spawn=Math.max(.45,1.2-speed*.65)+Math.random()*.35;
  }

  coinSpawn-=dt;
  if(coinSpawn<=0){
    spawnCoin();
    coinSpawn=1.1+Math.random()*1.2;
  }

  traffic.forEach(o=>o.z+=dt*(.19+speed*.7));
  pickups.forEach(o=>o.z+=dt*(.19+speed*.7));

  traffic=traffic.filter(o=>o.z<1.15);
  pickups=pickups.filter(o=>o.z<1.08);

  const px=W*.5+playerX*W*.34;
  const py=H*.82;
  const pw=Math.min(W*.15,68);
  const ph=Math.min(H*.17,95);

  for(const o of traffic){
    const p=project(o.lane,o.z);
    if(o.z>.78&&o.z<.99&&Math.abs(p.x-px)<(p.w+pw)*.38){
      endGame();
      return;
    }
  }

  for(const o of pickups){
    const p=project(o.lane,o.z);
    if(!o.got&&o.z>.78&&o.z<.98&&Math.abs(p.x-px)<pw*.55){
      o.got=true;
      coins++;
      score+=50;
    }
  }
  pickups=pickups.filter(o=>!o.got);

  background();
  drawRoad();

  const objs=[
    ...traffic.map(o=>({type:'car',o})),
    ...pickups.map(o=>({type:'coin',o}))
  ].sort((a,b)=>a.o.z-b.o.z);

  for(const item of objs){
    const o=item.o,p=project(o.lane,o.z);
    if(item.type==='car')drawCar(p.x,p.y,p.w,p.h,o.color);
    else drawCoin(p.x,p.y,5+o.z*13);
  }

  drawCar(px,py,pw,ph,'#12c9e8',true);

  scoreEl.textContent=Math.floor(score);
  speedEl.textContent=Math.floor(speed*220);
  coinsEl.textContent=coins;

  requestAnimationFrame(loop);
}

function setSteer(v){steer=v}

[['left',-1],['right',1]].forEach(([id,v])=>{
  const b=document.getElementById(id);
  b.addEventListener('pointerdown',e=>{
    e.preventDefault();
    setSteer(v);
  });
  ['pointerup','pointercancel','pointerleave'].forEach(ev=>{
    b.addEventListener(ev,()=>{
      if(steer===v)setSteer(0);
    });
  });
});

const brakeBtn=document.getElementById('brake');
brakeBtn.addEventListener('pointerdown',e=>{
  e.preventDefault();
  brake=true;
});
['pointerup','pointercancel','pointerleave'].forEach(ev=>{
  brakeBtn.addEventListener(ev,()=>brake=false);
});

canvas.addEventListener('pointerdown',e=>{
  if(!running)return;
  setSteer(e.clientX<W/2?-1:1);
});
canvas.addEventListener('pointerup',()=>setSteer(0));
canvas.addEventListener('pointercancel',()=>setSteer(0));

document.getElementById('start').addEventListener('click',reset);

background();
drawRoad();
drawCar(W/2,H*.82,Math.min(W*.15,68),Math.min(H*.17,95),'#12c9e8',true);
})();
</script>
</body>
</html>

