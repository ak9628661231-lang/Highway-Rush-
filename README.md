<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width,initial-scale=1.0,user-scalable=no">
<title>Village Road Runner</title>

<style>
*{
  margin:0;
  padding:0;
  box-sizing:border-box;
  -webkit-tap-highlight-color:transparent;
}

html,body{
  width:100%;
  height:100%;
  overflow:hidden;
  background:#77c95b;
  font-family:Arial,sans-serif;
}

#game{
  position:relative;
  width:100%;
  height:100%;
  overflow:hidden;
}

canvas{
  width:100%;
  height:100%;
  display:block;
}

#ui{
  position:absolute;
  top:15px;
  left:0;
  width:100%;
  display:flex;
  justify-content:space-between;
  padding:0 18px;
  color:white;
  font-size:20px;
  font-weight:bold;
  text-shadow:0 2px 4px #000;
  pointer-events:none;
}

#startScreen,#gameOver{
  position:absolute;
  inset:0;
  display:flex;
  align-items:center;
  justify-content:center;
  flex-direction:column;
  background:rgba(0,30,10,.55);
  color:white;
  text-align:center;
}

#gameOver{
  display:none;
}

.title{
  font-size:38px;
  font-weight:900;
  color:#ffe02f;
  text-shadow:3px 3px 0 #e34b17,0 5px 12px #000;
}

.subtitle{
  margin-top:8px;
  font-size:18px;
}

button{
  margin-top:25px;
  border:0;
  border-radius:30px;
  padding:14px 38px;
  font-size:20px;
  font-weight:bold;
  color:white;
  background:linear-gradient(#ffb900,#ff5b00);
  box-shadow:0 5px 0 #a52c00;
}

#controls{
  position:absolute;
  bottom:22px;
  left:0;
  width:100%;
  display:flex;
  justify-content:space-between;
  padding:0 20px;
  pointer-events:none;
}

.ctrl{
  pointer-events:auto;
  width:65px;
  height:65px;
  border-radius:50%;
  display:flex;
  justify-content:center;
  align-items:center;
  font-size:32px;
  color:white;
  background:rgba(0,0,0,.4);
  border:2px solid rgba(255,255,255,.5);
}
</style>
</head>

<body>

<div id="game">

<canvas id="canvas"></canvas>

<div id="ui">
  <div>⭐ <span id="score">0</span></div>
  <div>🪙 <span id="coins">0</span></div>
</div>

<div id="startScreen">
  <div class="title">VILLAGE RUN</div>
  <div class="subtitle">Green Fields Runner</div>
  <div class="subtitle">Swipe or use the buttons</div>
  <button id="startBtn">START GAME</button>
</div>

<div id="gameOver">
  <div class="title">GAME OVER</div>
  <div class="subtitle">
    Score: <span id="finalScore">0</span>
  </div>
  <button id="restartBtn">PLAY AGAIN</button>
</div>

<div id="controls">
  <div class="ctrl" id="left">◀</div>
  <div class="ctrl" id="right">▶</div>
</div>

</div>

<script>

const canvas=document.getElementById("canvas");
const ctx=canvas.getContext("2d");

let W,H;

let running=false;
let score=0;
let coins=0;

/* SLOW ENEMY */
let speed=0.32;

let roadOffset=0;
let spawnTimer=0;
let coinTimer=0;
let lastTime=0;

let player={
  lane:1,
  x:0,
  targetX:0,
  y:0,
  jump:0,
  jumping:false,
  jumpVelocity:0
};

let obstacles=[];
let coinItems=[];


/* RESIZE */

function resize(){

  W=canvas.width=window.innerWidth;
  H=canvas.height=window.innerHeight;

  player.y=H*.77;
  player.x=laneX(player.lane,player.y);
  player.targetX=player.x;
}

window.addEventListener("resize",resize);
resize();


/* LANE */

function laneX(lane,y){

  const center=W/2;
  const perspective=y/H;
  const laneWidth=W*(0.12+perspective*0.16);

  return center+(lane-1)*laneWidth;
}

function roadWidth(y){
  return W*(0.30+(y/H)*0.48);
}


/* SKY */

function drawSky(){

  const g=ctx.createLinearGradient(0,0,0,H);

  g.addColorStop(0,"#69c9ff");
  g.addColorStop(.45,"#b9ecff");
  g.addColorStop(1,"#c9f59b");

  ctx.fillStyle=g;
  ctx.fillRect(0,0,W,H);

  /* SUN */

  ctx.fillStyle="#ffe45c";

  ctx.beginPath();
  ctx.arc(W*.82,H*.13,35,0,Math.PI*2);
  ctx.fill();

  /* CLOUDS */

  ctx.fillStyle="rgba(255,255,255,.7)";

  for(let i=0;i<4;i++){

    const x=(i*W*.32)-30;
    const y=H*(.15+i*.045);

    ctx.beginPath();

    ctx.arc(x,y,25,0,Math.PI*2);
    ctx.arc(x+28,y-12,32,0,Math.PI*2);
    ctx.arc(x+65,y,24,0,Math.PI*2);

    ctx.fill();
  }
}


/* DISTANT HILLS */

function drawHills(){

  ctx.fillStyle="#66ad4e";

  ctx.beginPath();

  ctx.moveTo(0,H*.37);

  for(let x=0;x<=W;x+=40){

    const y=
      H*.34+
      Math.sin(x*.012)*18+
      Math.sin(x*.027)*10;

    ctx.lineTo(x,y);
  }

  ctx.lineTo(W,H*.50);
  ctx.lineTo(0,H*.50);

  ctx.closePath();
  ctx.fill();

  /* FAR TREES */

  for(let i=0;i<18;i++){

    const x=i*(W/17);

    drawTree(
      x,
      H*.42+Math.sin(i)*5,
      .35
    );
  }
}


/* GREEN GROUND */

function drawGround(){

  ctx.fillStyle="#59b847";
  ctx.fillRect(0,H*.40,W,H*.60);

  /* field strips */

  for(let side=0;side<2;side++){

    const startX=
      side===0 ? 0 : W*.72;

    const width=W*.28;

    ctx.strokeStyle="rgba(24,120,35,.25)";
    ctx.lineWidth=3;

    for(let y=H*.45;y<H;y+=18){

      ctx.beginPath();
      ctx.moveTo(startX,y);
      ctx.lineTo(startX+width,y+5);
      ctx.stroke();
    }
  }

  /* crop lines */

  ctx.strokeStyle="rgba(20,105,30,.35)";
  ctx.lineWidth=2;

  for(let y=H*.48;y<H;y+=30){

    ctx.beginPath();
    ctx.moveTo(0,y);
    ctx.lineTo(W*.35,y+10);
    ctx.stroke();

    ctx.beginPath();
    ctx.moveTo(W*.65,y+10);
    ctx.lineTo(W,y);
    ctx.stroke();
  }
}


/* TALAB */

function drawPond(x,y,w,h){

  ctx.fillStyle="#4f9f37";

  ctx.beginPath();
  ctx.ellipse(x,y,w/2+8,h/2+8,0,0,Math.PI*2);
  ctx.fill();

  ctx.fillStyle="#38a9d5";

  ctx.beginPath();
  ctx.ellipse(x,y,w/2,h/2,0,0,Math.PI*2);
  ctx.fill();

  /* water shine */

  ctx.strokeStyle="rgba(255,255,255,.55)";
  ctx.lineWidth=2;

  for(let i=0;i<4;i++){

    ctx.beginPath();

    ctx.moveTo(
      x-w*.3,
      y-h*.2+i*9
    );

    ctx.quadraticCurveTo(
      x,
      y-h*.3+i*9,
      x+w*.3,
      y-h*.2+i*9
    );

    ctx.stroke();
  }

  /* lotus */

  ctx.fillStyle="#ff91b6";

  ctx.beginPath();
  ctx.arc(x-20,y,6,0,Math.PI*2);
  ctx.fill();

  ctx.beginPath();
  ctx.arc(x-14,y-4,6,0,Math.PI*2);
  ctx.fill();

  ctx.fillStyle="#68b842";

  ctx.beginPath();
  ctx.arc(x+25,y+5,9,0,Math.PI*2);
  ctx.fill();
}


/* TREES */

function drawTree(x,y,s){

  /* trunk */

  ctx.fillStyle="#704529";

  ctx.fillRect(
    x-5*s,
    y,
    10*s,
    35*s
  );

  /* leaves */

  ctx.fillStyle="#198b35";

  ctx.beginPath();
  ctx.arc(x,y-8*s,25*s,0,Math.PI*2);
  ctx.arc(x-20*s,y+2*s,20*s,0,Math.PI*2);
  ctx.arc(x+20*s,y+2*s,20*s,0,Math.PI*2);
  ctx.fill();

  ctx.fillStyle="#39a943";

  ctx.beginPath();
  ctx.arc(x-5*s,y-18*s,16*s,0,Math.PI*2);
  ctx.fill();
}


/* SUGARCANE / CROPS */

function drawCrops(){

  const positions=[
    W*.07,W*.14,W*.22,
    W*.78,W*.86,W*.94
  ];

  positions.forEach((x,i)=>{

    for(let j=0;j<7;j++){

      const y=H*.47+j*18;

      ctx.strokeStyle="#187e32";
      ctx.lineWidth=3;

      ctx.beginPath();
      ctx.moveTo(x,y+15);
      ctx.lineTo(x+Math.sin(j+i)*5,y-5);
      ctx.stroke();

      ctx.strokeStyle="#6fc84c";

      ctx.beginPath();
      ctx.moveTo(x+2,y);
      ctx.lineTo(x-8,y-6);
      ctx.stroke();

      ctx.beginPath();
      ctx.moveTo(x+2,y+5);
      ctx.lineTo(x+10,y-2);
      ctx.stroke();
    }
  });
}


/* VILLAGE HOUSES */

function drawHouse(x,y,s){

  ctx.fillStyle="#f3d59b";

  ctx.fillRect(
    x-35*s,
    y-35*s,
    70*s,
    55*s
  );

  /* roof */

  ctx.fillStyle="#b74b32";

  ctx.beginPath();

  ctx.moveTo(x-45*s,y-35*s);
  ctx.lineTo(x,y-70*s);
  ctx.lineTo(x+45*s,y-35*s);

  ctx.closePath();
  ctx.fill();

  /* door */

  ctx.fillStyle="#68452f";

  ctx.fillRect(
    x-9*s,
    y-5*s,
    18*s,
    25*s
  );

  /* window */

  ctx.fillStyle="#7fd6e8";

  ctx.fillRect(
    x+18*s,
    y-20*s,
    15*s,
    13*s
  );
}


/* BACKGROUND */

function drawBackground(){

  drawSky();
  drawGround();
  drawHills();

  /* ponds */

  drawPond(
    W*.13,
    H*.56,
    W*.20,
    H*.08
  );

  drawPond(
    W*.87,
    H*.61,
    W*.18,
    H*.075
  );

  /* houses */

  drawHouse(W*.06,H*.50,.45);
  drawHouse(W*.94,H*.48,.40);

  /* trees */

  drawTree(W*.04,H*.54,1);
  drawTree(W*.27,H*.49,.75);

  drawTree(W*.73,H*.50,.75);
  drawTree(W*.96,H*.54,1);

  drawCrops();
}


/* ROAD */

function drawRoad(){

  const topY=H*.36;
  const bottomY=H;

  const topWidth=W*.12;
  const bottomWidth=W*.88;

  ctx.fillStyle="#5c5c58";

  ctx.beginPath();

  ctx.moveTo(W/2-topWidth/2,topY);
  ctx.lineTo(W/2+topWidth/2,topY);
  ctx.lineTo(W/2+bottomWidth/2,bottomY);
  ctx.lineTo(W/2-bottomWidth/2,bottomY);

  ctx.closePath();
  ctx.fill();

  /* road edge */

  ctx.strokeStyle="#e8d89c";
  ctx.lineWidth=6;

  ctx.beginPath();
  ctx.moveTo(W/2-topWidth/2,topY);
  ctx.lineTo(W/2-bottomWidth/2,bottomY);
  ctx.stroke();

  ctx.beginPath();
  ctx.moveTo(W/2+topWidth/2,topY);
  ctx.lineTo(W/2+bottomWidth/2,bottomY);
  ctx.stroke();

  /* lane lines */

  ctx.strokeStyle="rgba(255,255,255,.45)";
  ctx.lineWidth=3;

  for(let lane=1;lane<=2;lane++){

    ctx.beginPath();

    const topX=
      W/2+(lane-1.5)*topWidth;

    const bottomX=
      W/2+(lane-1.5)*bottomWidth;

    ctx.moveTo(topX,topY);
    ctx.lineTo(bottomX,bottomY);

    ctx.stroke();
  }

  /* road dashes */

  for(let i=0;i<18;i++){

    let p=((i/18)+roadOffset)%1;

    const y=
      topY+
      Math.pow(p,1.8)*(H-topY);

    const half=
      roadWidth(y)/2;

    ctx.strokeStyle="rgba(255,255,255,.55)";
    ctx.lineWidth=4;

    ctx.beginPath();

    ctx.moveTo(W/2-half,y);
    ctx.lineTo(W/2+half,y);

    ctx.stroke();
  }
}


/* PLAYER */

function drawPlayer(){

  let x=player.x;
  let y=player.y-player.jump;

  /* shadow */

  ctx.fillStyle="rgba(0,0,0,.35)";

  ctx.beginPath();

  ctx.ellipse(
    x,
    player.y+12,
    25,
    8,
    0,
    0,
    Math.PI*2
  );

  ctx.fill();

  /* legs */

  ctx.strokeStyle="#f7b28c";
  ctx.lineWidth=8;
  ctx.lineCap="round";

  ctx.beginPath();
  ctx.moveTo(x-8,y+28);
  ctx.lineTo(x-12,y+58);
  ctx.stroke();

  ctx.beginPath();
  ctx.moveTo(x+8,y+28);
  ctx.lineTo(x+13,y+58);
  ctx.stroke();

  /* shoes */

  ctx.fillStyle="#e84d55";

  ctx.beginPath();
  ctx.ellipse(x-14,y+61,10,5,0,0,Math.PI*2);
  ctx.fill();

  ctx.beginPath();
  ctx.ellipse(x+14,y+61,10,5,0,0,Math.PI*2);
  ctx.fill();

  /* dress */

  ctx.fillStyle="#ff82ae";

  ctx.beginPath();

  ctx.moveTo(x,y-8);
  ctx.lineTo(x-27,y+37);
  ctx.lineTo(x+27,y+37);

  ctx.closePath();
  ctx.fill();

  /* shirt */

  ctx.fillStyle="#fff1f5";
  ctx.fillRect(x-13,y-15,26,30);

  /* arms */

  ctx.strokeStyle="#f7b28c";
  ctx.lineWidth=7;

  ctx.beginPath();
  ctx.moveTo(x-12,y-7);
  ctx.lineTo(x-31,y+15);
  ctx.stroke();

  ctx.beginPath();
  ctx.moveTo(x+12,y-7);
  ctx.lineTo(x+31,y+15);
  ctx.stroke();

  /* head */

  ctx.fillStyle="#f7b28c";

  ctx.beginPath();
  ctx.arc(x,y-34,17,0,Math.PI*2);
  ctx.fill();

  /* hair */

  ctx.fillStyle="#d83f45";

  ctx.beginPath();
  ctx.arc(x,y-42,19,Math.PI,Math.PI*2);
  ctx.fill();

  ctx.beginPath();
  ctx.arc(x-13,y-38,7,0,Math.PI*2);
  ctx.fill();

  ctx.beginPath();
  ctx.arc(x+13,y-38,7,0,Math.PI*2);
  ctx.fill();

  /* eyes */

  ctx.fillStyle="#222";

  ctx.beginPath();
  ctx.arc(x-6,y-35,2,0,Math.PI*2);
  ctx.fill();

  ctx.beginPath();
  ctx.arc(x+6,y-35,2,0,Math.PI*2);
  ctx.fill();
}


/* COIN */

function drawCoin(c){

  const x=laneX(c.lane,c.y);
  const s=.45+(c.y/H)*.9;

  ctx.save();

  ctx.translate(x,c.y);
  ctx.scale(s,s);

  ctx.fillStyle="#ffad00";
  ctx.strokeStyle="#ffe34b";
  ctx.lineWidth=4;

  ctx.beginPath();
  ctx.arc(0,0,15,0,Math.PI*2);
  ctx.fill();
  ctx.stroke();

  ctx.fillStyle="#ffe34b";
  ctx.font="bold 18px Arial";
  ctx.textAlign="center";
  ctx.textBaseline="middle";

  ctx.fillText("$",0,1);

  ctx.restore();
}


/* ENEMY */

function drawObstacle(o){

  const x=laneX(o.lane,o.y);
  const s=.4+(o.y/H)*.9;

  ctx.save();

  ctx.translate(x,o.y);
  ctx.scale(s,s);

  ctx.fillStyle="#e94b3c";

  ctx.beginPath();

  ctx.roundRect(
    -24,-30,48,55,8
  );

  ctx.fill();

  ctx.fillStyle="#ffcc28";

  ctx.fillRect(-17,-20,34,7);
  ctx.fillRect(-17,-3,34,7);

  ctx.fillStyle="#222";

  ctx.fillRect(-20,20,12,8);
  ctx.fillRect(8,20,12,8);

  ctx.restore();
}


/* SPAWN */

function spawnObstacle(){

  obstacles.push({
    lane:Math.floor(Math.random()*3),
    y:H*.35
  });
}

function spawnCoin(){

  coinItems.push({
    lane:Math.floor(Math.random()*3),
    y:H*.35
  });
}


/* FAST PLAYER */

function moveLeft(){

  if(!running)return;

  if(player.lane>0){

    player.lane--;

    player.targetX=
      laneX(player.lane,player.y);

    player.x=
      player.targetX;
  }
}


function moveRight(){

  if(!running)return;

  if(player.lane<2){

    player.lane++;

    player.targetX=
      laneX(player.lane,player.y);

    player.x=
      player.targetX;
  }
}


/* JUMP */

function jump(){

  if(!running || player.jumping)return;

  player.jumping=true;
  player.jumpVelocity=17;
}


/* GAME OVER */

function gameOver(){

  running=false;

  document.getElementById("finalScore")
    .textContent=Math.floor(score);

  document.getElementById("gameOver")
    .style.display="flex";
}


/* RESET */

function resetGame(){

  score=0;
  coins=0;

  speed=.32;

  roadOffset=0;
  spawnTimer=0;
  coinTimer=0;

  obstacles=[];
  coinItems=[];

  player.lane=1;
  player.jump=0;
  player.jumping=false;
  player.jumpVelocity=0;

  player.y=H*.77;

  player.x=
    laneX(1,player.y);

  player.targetX=
    player.x;

  document.getElementById("score")
    .textContent="0";

  document.getElementById("coins")
    .textContent="0";
}


/* START */

function startGame(){

  resetGame();

  running=true;

  document.getElementById("startScreen")
    .style.display="none";

  document.getElementById("gameOver")
    .style.display="none";

  lastTime=performance.now();

  requestAnimationFrame(loop);
}


/* UPDATE */

function update(dt){

  score+=dt*.012;

  document.getElementById("score")
    .textContent=Math.floor(score);

  /* FAST CONTROL */

  player.x=player.targetX;

  /* JUMP */

  if(player.jumping){

    player.jump+=player.jumpVelocity;

    player.jumpVelocity-=.8;

    if(player.jump<=0){

      player.jump=0;
      player.jumping=false;
      player.jumpVelocity=0;
    }
  }

  /* ROAD */

  roadOffset+=speed*dt*.001;

  if(roadOffset>1)
    roadOffset-=1;

  /* ENEMY SPAWN */

  spawnTimer+=dt;

  if(spawnTimer>1300){

    spawnTimer=0;

    spawnObstacle();
  }

  /* COIN */

  coinTimer+=dt;

  if(coinTimer>500){

    coinTimer=0;

    if(Math.random()<.9)
      spawnCoin();
  }

  /* ENEMIES SLOW */

  for(let i=obstacles.length-1;i>=0;i--){

    let o=obstacles[i];

    o.y+=speed*dt*.65;

    if(
      o.lane===player.lane &&
      Math.abs(o.y-player.y)<48 &&
      player.jump<35
    ){

      gameOver();
      return;
    }

    if(o.y>H+80)
      obstacles.splice(i,1);
  }

  /* COINS */

  for(let i=coinItems.length-1;i>=0;i--){

    let c=coinItems[i];

    c.y+=speed*dt*.65;

    if(
      c.lane===player.lane &&
      Math.abs(c.y-player.y)<50
    ){

      coins++;

      document.getElementById("coins")
        .textContent=coins;

      score+=10;

      coinItems.splice(i,1);

      continue;
    }

    if(c.y>H+60)
      coinItems.splice(i,1);
  }
}


/* DRAW */

function draw(){

  ctx.clearRect(0,0,W,H);

  drawBackground();
  drawRoad();

  obstacles
    .slice()
    .sort((a,b)=>a.y-b.y)
    .forEach(drawObstacle);

  coinItems
    .slice()
    .sort((a,b)=>a.y-b.y)
    .forEach(drawCoin);

  drawPlayer();
}


/* LOOP */

function loop(now){

  if(!running)return;

  const dt=
    Math.min(now-lastTime,40);

  lastTime=now;

  update(dt);
  draw();

  requestAnimationFrame(loop);
}


/* KEYBOARD */

document.addEventListener("keydown",e=>{

  if(e.key==="ArrowLeft" || e.key==="a")
    moveLeft();

  if(e.key==="ArrowRight" || e.key==="d")
    moveRight();

  if(
    e.key==="ArrowUp" ||
    e.key==="w" ||
    e.key===" "
  )
    jump();
});


/* MOBILE BUTTONS */

document.getElementById("left")
.addEventListener(
  "touchstart",
  e=>{
    e.preventDefault();
    moveLeft();
  },
  {passive:false}
);

document.getElementById("right")
.addEventListener(
  "touchstart",
  e=>{
    e.preventDefault();
    moveRight();
  },
  {passive:false}
);

document.getElementById("left")
.addEventListener("click",moveLeft);

document.getElementById("right")
.addEventListener("click",moveRight);


/* SWIPE */

let startX=0;
let startY=0;

canvas.addEventListener(
  "touchstart",
  e=>{

    startX=e.touches[0].clientX;
    startY=e.touches[0].clientY;

  },
  {passive:true}
);

canvas.addEventListener(
  "touchend",
  e=>{

    const endX=e.changedTouches[0].clientX;
    const endY=e.changedTouches[0].clientY;

    const dx=endX-startX;
    const dy=endY-startY;

    if(Math.abs(dx)>35){

      if(dx<0)
        moveLeft();
      else
        moveRight();

    }else if(dy<-50){

      jump();
    }

  },
  {passive:true}
);


/* BUTTONS */

document.getElementById("startBtn")
.addEventListener("click",startGame);

document.getElementById("restartBtn")
.addEventListener("click",startGame);


/* INITIAL */

draw();

</script>

</body>
</html>
