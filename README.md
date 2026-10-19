
<!DOCTYPE html>
<html lang="hi">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1">
<title>Netaji vs CID - Comedy Run</title>
<style>
*{box-sizing:border-box}
body{margin:0;background:#78d5ff;font-family:Arial,sans-serif;text-align:center;color:#172033}
h2{margin:12px 0}
#info{font-weight:bold;padding:8px}
#game{display:block;width:100%;max-width:800px;height:65vh;min-height:350px;margin:auto;background:#a9e6ff;border:3px solid #fff}
button{font-size:18px;font-weight:bold;border:0;border-radius:12px;padding:14px;margin:7px;background:#ffca28;color:#222}
#msg{font-weight:bold;background:white;padding:12px;margin:10px auto;max-width:800px;border-radius:12px}
</style>
</head>
<body>
<h2>🏃 Netaji vs CID 🚔</h2>
<div id="info">Score: <span id="score">0</span> | Best: <span id="best">0</span></div>
<canvas id="game"></canvas>
<div id="msg">CID: Pakdo ise! Iski speed se toh hamari jeep bhi sharma gayi!</div>
<div>
<button onclick="jump()">⬆️ JUMP</button>
<button onclick="duck()">⬇️ DUCK</button>
<button onclick="pauseGame()">⏸️ PAUSE</button>
</div>
<button id="start" onclick="startGame()">🎮 GAME START</button>
<p>Mobile par buttons dabao. Computer par Space se jump aur ↓ se duck karo.</p>
<p><small>Yeh ek kalpanik comedy/parody game hai.</small></p>
<script>
const c=document.getElementById("game");
const ctx=c.getContext("2d");
const scoreBox=document.getElementById("score");
const bestBox=document.getElementById("best");
const msg=document.getElementById("msg");
const startBtn=document.getElementById("start");

let W=800,H=450,ground=380;
let player,obstacles,score,best,speed,active,paused,ducking;
let last=0,spawn=0,frame=0;
let lines=[
'😂 CID: Pakdo ise! Iski speed se toh hamari jeep bhi sharma gayi!',
'🚔 CID: Sahab, bhaagna band! Sawalon ka jawab dekar jaana!',
'🤣 CID: Aapki speed se toh hamari jeep bhi sharma gayi!',
'🕵️ CID: Game abhi khatam nahi hua, poochtaach baaki hai!',
'😎 CID: Ruk jaiye sahab, CID peeche lagi hai!'
];
try{best=Number(localStorage.getItem("cidBest")||0)}catch(e){best=0}
bestBox.textContent=best;

function resize(){
 const r=c.getBoundingClientRect();
 W=Math.max(300,r.width);
 H=Math.max(350,r.height);
 c.width=W*devicePixelRatio;
 c.height=H*devicePixelRatio;
 ctx.setTransform(devicePixelRatio,0,0,devicePixelRatio,0,0);
 ground=H-55;
 if(player && player.onGround)player.y=ground-player.h;
 draw();
}
window.addEventListener("resize",resize);

function reset(){
 player={x:55,y:ground-55,w:35,h:55,vy:0,onGround:true};
 obstacles=[];score=0;speed=5;spawn=60;frame=0;ducking=0;
 scoreBox.textContent=0;
}
function startGame(){
 reset();active=true;paused=false;
 startBtn.textContent="🔄 RESTART";
 msg.textContent=lines[0];
 last=performance.now();
 requestAnimationFrame(loop);
}
function jump(){
 if(active&&!paused&&player.onGround&&!ducking){
  player.vy=-13;player.onGround=false;
 }
}
function duck(){
 if(active&&!paused)ducking=25;
}
function pauseGame(){
 if(!active)return;
 paused=!paused;
 msg.textContent=paused?"Game pause hai, bhai!":"CID: Pakdo ise!";
 if(!paused){last=performance.now();requestAnimationFrame(loop)}
}
window.addEventListener("keydown",e=>{
 if(e.code==="Space"||e.code==="ArrowUp"){e.preventDefault();jump()}
 if(e.code==="ArrowDown"){e.preventDefault();duck()}
});
function rounded(x,y,w,h,r,color){
 ctx.fillStyle=color;ctx.beginPath();
 ctx.roundRect(x,y,w,h,r);ctx.fill();
}
function drawPerson(x,y,cid=false){
 ctx.save();
 ctx.strokeStyle=cid?"#182b49":"#28354a";
 ctx.lineWidth=6;ctx.lineCap="round";
 ctx.beginPath();ctx.moveTo(x+12,y+38);ctx.lineTo(x+8,y+51);ctx.stroke();
 ctx.beginPath();ctx.moveTo(x+24,y+38);ctx.lineTo(x+29,y+51);ctx.stroke();
 rounded(x+5,y+19,27,25,7,cid?"#182b49":"#f28c28");
 ctx.fillStyle="#b97955";
 ctx.beginPath();ctx.arc(x+18,y+12,12,0,Math.PI*2);ctx.fill();
 ctx.fillStyle=cid?"#17243a":"#f4f0d2";
 ctx.fillRect(x+6,y+2,24,6);
 ctx.fillStyle="#222";
 ctx.fillRect(x+11,y+12,5,2);ctx.fillRect(x+21,y+12,5,2);
 if(cid){
  ctx.fillStyle="white";ctx.font="bold 8px Arial";
  ctx.fillText("CID",x+10,y+34);
 }
 ctx.restore();
}
function draw(){
 if(!W||!H)return;
 ctx.clearRect(0,0,W,H);
 let sky=ctx.createLinearGradient(0,0,0,H);
 sky.addColorStop(0,"#63c8ff");sky.addColorStop(1,"#e0f8ff");
 ctx.fillStyle=sky;ctx.fillRect(0,0,W,H);
 // Clouds
 ctx.fillStyle="white";
 for(let i=0;i<4;i++){
  let x=(i*180-frame*0.4)%(W+80);
  ctx.beginPath();ctx.arc(x,65+i%2*30,16,0,7);
  ctx.arc(x+20,57+i%2*30,20,0,7);
  ctx.arc(x+42,66+i%2*30,15,0,7);ctx.fill();
 }
 // Buildings
 for(let i=0;i<8;i++){
  let x=(i*115-frame*1.2)%(W+120);
  ctx.fillStyle=i%2?"#91b7c5":"#b0cbd3";
  ctx.fillRect(x,ground-100-(i%3)*20,55,100+(i%3)*20);
  ctx.fillStyle="#fff2b0";
  for(let yy=ground-85;yy<ground-15;yy+=20)
   for(let xx=x+8;xx<x+48;xx+=18)ctx.fillRect(xx,yy,7,8);
 }
 ctx.fillStyle="#38a653";ctx.fillRect(0,ground,W,H-ground);
 ctx.fillStyle="#e5d3ad";ctx.fillRect(0,ground+12,W,25);
 ctx.fillStyle="white";
 for(let x=-(frame*2%75);x<W;x+=75)ctx.fillRect(x,ground+22,35,4);
 // CID chasing
 drawPerson(12,ground-55,true);
 // Obstacles
 for(const o of obstacles){
  if(o.type===0){
   ctx.fillStyle="#ff6b24";ctx.beginPath();
   ctx.moveTo(o.x+15,o.y);ctx.lineTo(o.x+30,o.y+35);
   ctx.lineTo(o.x,o.y+35);ctx.fill();
   ctx.fillStyle="white";ctx.fillRect(o.x+7,o.y+21,16,4);
  }else{
   rounded(o.x,o.y,48,27,4,"#e74747");
   ctx.fillStyle="white";
   for(let k=4;k<45;k+=16){
    ctx.beginPath();ctx.moveTo(o.x+k,o.y+26);
    ctx.lineTo(o.x+k+10,o.y+2);ctx.lineTo(o.x+k+15,o.y+2);
    ctx.lineTo(o.x+k+5,o.y+26);ctx.fill();
   }
  }
 }
 // Runner
 if(player){
  if(ducking){
   rounded(player.x,player.y+20,40,28,8,"#f28c28");
   ctx.fillStyle="#b97955";ctx.beginPath();
   ctx.arc(player.x+30,player.y+22,10,0,7);ctx.fill();
  }else drawPerson(player.x,player.y,false);
 }
}
function endGame(){
 active=false;
 if(score>best){
  best=score;bestBox.textContent=best;
  try{localStorage.setItem("cidBest",best)}catch(e){}
 }
 msg.textContent="😂 CID: Sahab, ab daud khatam… ab hisaab shuru! Score: "+score;
 startBtn.textContent="🔄 PHIR SE KHELO";
 draw();
}
function loop(now){
 if(!active)return;
 if(paused)return;
 let dt=Math.min((now-last)/16.67,2);
 last=now;frame+=dt;
 score+=Math.max(1,Math.floor(dt));
 scoreBox.textContent=score;
 speed=Math.min(10,5+score/300);
 player.vy+=0.65*dt;
 player.y+=player.vy*dt;
 if(player.y>=ground-player.h){
  player.y=ground-player.h;player.vy=0;player.onGround=true;
 }
 if(ducking>0)ducking-=dt;
 spawn-=dt;
 if(spawn<=0){
  obstacles.push({x:W+20,y:ground-(Math.random()<0.5?35:27),
   type:Math.random()<0.5?0:1,w:30,h:35});
  spawn=65+Math.random()*45;
 }
 for(const o of obstacles)o.x-=speed*dt;
 obstacles=obstacles.filter(o=>o.x>-60);
 const ph=ducking>0?35:player.h;
 const py=ducking>0?player.y+20:player.y;
 for(const o of obstacles){
  if(player.x+30>o.x+4&&player.x+5<o.x+(o.type===0?30:48)
   &&py+ph>o.y+3&&py<o.y+(o.type===0?35:27)){
   endGame();return;
  }
 }
 if(Math.floor(frame)%300===0)msg.textContent=lines[Math.floor(frame/300)%lines.length];
 draw();
 requestAnimationFrame(loop);
}
resize();
</script>
</body>
</html>
