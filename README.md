<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no">

<title>Ocean Runner</title>

<style>
*{
  margin:0;
  padding:0;
  box-sizing:border-box;
  touch-action:none;
}

html,body{
  width:100%;
  height:100%;
  overflow:hidden;
  background:#05070d;
  font-family:Arial,sans-serif;
}

#game{
  position:relative;
  width:100%;
  height:100%;
  overflow:hidden;
  background:#07152b;
}

canvas{
  width:100%;
  height:100%;
  display:block;
}

#score{
  position:absolute;
  top:18px;
  left:18px;
  color:white;
  font-size:22px;
  font-weight:bold;
  text-shadow:2px 2px 4px #000;
  z-index:5;
}

#coins{
  position:absolute;
  top:50px;
  left:18px;
  color:#ffd83d;
  font-size:20px;
  font-weight:bold;
  text-shadow:2px 2px 4px #000;
  z-index:5;
}

#gameOver{
  display:none;
  position:absolute;
  inset:0;
  background:rgba(0,0,0,.65);
  color:white;
  align-items:center;
  justify-content:center;
  flex-direction:column;
  z-index:20;
}

#gameOver h1{
  font-size:42px;
  margin-bottom:12px;
}

#gameOver p{
  font-size:20px;
  margin-bottom:22px;
}

button{
  border:0;
  border-radius:14px;
  padding:14px 30px;
  font-size:20px;
  font-weight:bold;
  background:#ffd52e;
  color:#111;
}

#hint{
  position:absolute;
  bottom:20px;
  left:50%;
  transform:translateX(-50%);
  color:white;
  opacity:.7;
  font-size:14px;
  z-index:5;
}
</style>
</head>

<body>

<div id="game">

<canvas id="canvas"></canvas>

<div id="score">Score: 0</div>
<div id="coins">🪙 0</div>

<div id="hint">
Swipe ← → to move
</div>

<div id="gameOver">
  <h1>GAME OVER</h1>
  <p id="finalScore">Score: 0</p>
  <button onclick="restartGame()">PLAY AGAIN</button>
</div>

</div>

<script>

const canvas = document.getElementById("canvas");
const ctx = canvas.getContext("2d");

let W,H;
let dpr = Math.min(window.devicePixelRatio || 1,2);

function resize(){

  W = window.innerWidth;
  H = window.innerHeight;

  canvas.width = W*dpr;
  canvas.height = H*dpr;

  canvas.style.width = W+"px";
  canvas.style.height = H+"px";

  ctx.setTransform(dpr,0,0,dpr,0,0);
}

window.addEventListener("resize",resize);
resize();


/* =========================
   GAME VARIABLES
========================= */

let running = true;

let score = 0;
let coinCount = 0;


/* ROAD SPEED AUR SLOW */
let speed = 0.20;

let roadOffset = 0;

let playerLane = 1;
let playerX = 0;

let targetX = 0;

let objects = [];

let spawnTimer = 0;

let lastTime = performance.now();

let swipeStartX = 0;
let swipeStartY = 0;


/* =========================
   ROAD
========================= */

function roadWidth(y){

  let horizon = H*0.28;

  let t = (y-horizon)/(H-horizon);

  t = Math.max(0,Math.min(1,t));

  return 100 + t*W*0.92;
}

function roadCenter(){

  return W/2;
}

function laneX(lane,y){

  let rw = roadWidth(y);
  let laneW = rw/3;

  return roadCenter() - rw/2 + laneW*(lane+0.5);
}


/* =========================
   BACKGROUND
========================= */

function drawBackground(){

  let sky = ctx.createLinearGradient(0,0,0,H*.55);

  sky.addColorStop(0,"#061b4c");
  sky.addColorStop(.55,"#1764a5");
  sky.addColorStop(1,"#27b7d4");

  ctx.fillStyle = sky;
  ctx.fillRect(0,0,W,H*.58);


  let sea = ctx.createLinearGradient(0,H*.35,0,H);

  sea.addColorStop(0,"#087e9b");
  sea.addColorStop(1,"#031d36");

  ctx.fillStyle = sea;
  ctx.fillRect(0,H*.35,W,H);


  ctx.beginPath();
  ctx.arc(W*.82,H*.12,32,0,Math.PI*2);
  ctx.fillStyle="rgba(255,255,220,.85)";
  ctx.fill();


  ctx.fillStyle="#06435b";

  ctx.beginPath();

  ctx.moveTo(0,H*.43);
  ctx.lineTo(W*.12,H*.34);
  ctx.lineTo(W*.23,H*.42);
  ctx.lineTo(W*.35,H*.35);
  ctx.lineTo(W*.50,H*.43);
  ctx.lineTo(W*.66,H*.34);
  ctx.lineTo(W*.82,H*.42);
  ctx.lineTo(W,H*.34);
  ctx.lineTo(W,H*.60);
  ctx.lineTo(0,H*.60);

  ctx.closePath();
  ctx.fill();


  drawPalm(55,H*.39,.8);
  drawPalm(W-60,H*.38,.9);

  drawDolphin(90,H*.38,1);
  drawDolphin(W-100,H*.38,.9);
}


/* =========================
   PALM TREE
========================= */

function drawPalm(x,y,s){

  ctx.save();

  ctx.translate(x,y);
  ctx.scale(s,s);

  ctx.strokeStyle="#59351d";
  ctx.lineWidth=8;

  ctx.beginPath();
  ctx.moveTo(0,80);
  ctx.quadraticCurveTo(-8,30,0,0);
  ctx.stroke();

  ctx.strokeStyle="#1b8d59";
  ctx.lineWidth=6;

  for(let i=0;i<6;i++){

    let a = i*Math.PI/3;

    ctx.beginPath();

    ctx.moveTo(0,0);

    ctx.quadraticCurveTo(
      Math.cos(a)*35,
      Math.sin(a)*20,
      Math.cos(a)*65,
      Math.sin(a)*35
    );

    ctx.stroke();
  }

  ctx.restore();
}


/* =========================
   DOLPHIN
========================= */

function drawDolphin(x,y,s){

  ctx.save();

  ctx.translate(x,y);
  ctx.scale(s,s);

  ctx.strokeStyle="rgba(150,245,255,.8)";
  ctx.lineWidth=5;

  ctx.beginPath();

  ctx.moveTo(-55,10);

  ctx.quadraticCurveTo(
    0,-40,
    55,5
  );

  ctx.quadraticCurveTo(
    10,28,
    -25,5
  );

  ctx.stroke();

  ctx.beginPath();
  ctx.moveTo(0,-4);
  ctx.lineTo(15,-28);
  ctx.stroke();

  ctx.restore();
}


/* =========================
   ROAD
========================= */

function drawRoad(){

  let horizon = H*.28;

  let bottomWidth = W*1.05;


  ctx.fillStyle="#182d3d";

  ctx.beginPath();

  ctx.moveTo(W/2-50,horizon);
  ctx.lineTo(W/2+50,horizon);

  ctx.lineTo(W/2+bottomWidth/2,H);
  ctx.lineTo(W/2-bottomWidth/2,H);

  ctx.closePath();
  ctx.fill();


  let tileSize = 55;

  for(let y=horizon; y<H; y+=tileSize){

    let yy = y + (roadOffset % tileSize);

    if(yy<horizon) continue;

    let rw = roadWidth(yy);

    let left = W/2-rw/2;
    let lane = rw/3;

    for(let i=0;i<3;i++){

      ctx.fillStyle =
        ((Math.floor((yy-roadOffset)/tileSize)+i)%2===0)
        ? "#164454"
        : "#1b5260";

      ctx.fillRect(
        left+i*lane,
        yy,
        lane-2,
        tileSize-2
      );
    }
  }


  ctx.strokeStyle="#6ed5dc";
  ctx.lineWidth=4;

  for(let i=0;i<=3;i++){

    ctx.beginPath();

    ctx.moveTo(
      W/2-50+(100/3)*i,
      horizon
    );

    ctx.lineTo(
      W/2-bottomWidth/2+(bottomWidth/3)*i,
      H
    );

    ctx.stroke();
  }


  drawRoadLight(-1);
  drawRoadLight(1);
}


function drawRoadLight(side){

  let horizon=H*.28;

  for(let i=0;i<6;i++){

    let t=(i/6 + (roadOffset*.0007))%1;

    let y=horizon+t*(H-horizon);

    let rw=roadWidth(y);

    let x=W/2+side*(rw/2+25);

    let size=5+t*8;

    ctx.fillStyle="#54e8ff";

    ctx.beginPath();

    ctx.arc(x,y,size,0,Math.PI*2);

    ctx.fill();
  }
}


/* =========================
   PLAYER
========================= */

function drawPlayer(){

  let y=H*.76;


  /* PLAYER FAST MOVE */
  playerX += (targetX-playerX)*0.35;


  ctx.fillStyle="rgba(0,0,0,.4)";

  ctx.beginPath();

  ctx.ellipse(
    playerX,
    y+45,
    28,
    10,
    0,
    0,
    Math.PI*2
  );

  ctx.fill();


  ctx.strokeStyle="#ffd0b4";
  ctx.lineWidth=8;
  ctx.lineCap="round";

  ctx.beginPath();
  ctx.moveTo(playerX-8,y+22);
  ctx.lineTo(playerX-11,y+43);
  ctx.stroke();


  ctx.beginPath();
  ctx.moveTo(playerX+8,y+22);
  ctx.lineTo(playerX+11,y+43);
  ctx.stroke();


  ctx.fillStyle="#e84155";

  ctx.beginPath();

  ctx.ellipse(
    playerX-12,
    y+46,
    8,
    5,
    0,
    0,
    Math.PI*2
  );

  ctx.fill();


  ctx.beginPath();

  ctx.ellipse(
    playerX+12,
    y+46,
    8,
    5,
    0,
    0,
    Math.PI*2
  );

  ctx.fill();


  ctx.fillStyle="#fff0f5";

  ctx.beginPath();

  ctx.moveTo(playerX,y-35);
  ctx.lineTo(playerX-25,y+20);
  ctx.lineTo(playerX+25,y+20);

  ctx.closePath();

  ctx.fill();


  ctx.fillStyle="#ff91b5";

  ctx.beginPath();

  ctx.arc(playerX,y-3,9,0,Math.PI*2);

  ctx.fill();


  ctx.fillStyle="#ffd0b4";

  ctx.beginPath();

  ctx.arc(playerX,y-38,13,0,Math.PI*2);

  ctx.fill();


  ctx.fillStyle="#e85b55";

  ctx.beginPath();

  ctx.arc(playerX,y-43,16,0,Math.PI*2);

  ctx.fill();


  ctx.fillStyle="#c83e48";

  ctx.beginPath();

  ctx.arc(playerX-12,y-34,7,0,Math.PI*2);

  ctx.fill();


  ctx.beginPath();

  ctx.arc(playerX+12,y-34,7,0,Math.PI*2);

  ctx.fill();


  ctx.strokeStyle="#ffd0b4";
  ctx.lineWidth=6;

  ctx.beginPath();

  ctx.moveTo(playerX-12,y-22);
  ctx.lineTo(playerX-28,y-5);

  ctx.stroke();


  ctx.beginPath();

  ctx.moveTo(playerX+12,y-22);
  ctx.lineTo(playerX+28,y-5);

  ctx.stroke();
}


/* =========================
   COIN
========================= */

function drawCoin(o){

  let x=o.x;
  let y=o.y;
  let r=o.size;

  ctx.save();

  ctx.shadowColor="#ffd52e";
  ctx.shadowBlur=12;

  ctx.fillStyle="#ffd52e";

  ctx.beginPath();
  ctx.arc(x,y,r,0,Math.PI*2);
  ctx.fill();

  ctx.shadowBlur=0;

  ctx.strokeStyle="#fff0a0";
  ctx.lineWidth=2;

  ctx.beginPath();
  ctx.arc(x,y,r-2,0,Math.PI*2);
  ctx.stroke();

  ctx.fillStyle="#e39b00";

  ctx.font="bold "+Math.max(10,r)+"px Arial";

  ctx.textAlign="center";
  ctx.textBaseline="middle";

  ctx.fillText("★",x,y);

  ctx.restore();
}


/* =========================
   OBSTACLE
========================= */

function drawObstacle(o){

  let x=o.x;
  let y=o.y;
  let s=o.size;

  ctx.fillStyle="#e54848";

  ctx.beginPath();

  ctx.roundRect(
    x-s,
    y-s*.7,
    s*2,
    s*1.4,
    8
  );

  ctx.fill();


  ctx.fillStyle="#ffe36a";

  ctx.fillRect(
    x-s*.65,
    y-s*.35,
    s*1.3,
    s*.22
  );


  ctx.fillStyle="#222";

  ctx.beginPath();

  ctx.arc(
    x-s*.55,
    y+s*.55,
    s*.2,
    0,
    Math.PI*2
  );

  ctx.fill();


  ctx.beginPath();

  ctx.arc(
    x+s*.55,
    y+s*.55,
    s*.2,
    0,
    Math.PI*2
  );

  ctx.fill();
}


/* =========================
   SPAWN
========================= */

function spawnObject(){

  let lane=Math.floor(Math.random()*3);

  let type=Math.random()<0.72
    ? "coin"
    : "obstacle";


  objects.push({

    lane:lane,

    y:H*.29,

    type:type,

    size:type==="coin" ? 14 : 25,

    x:laneX(lane,H*.29)

  });
}


/* =========================
   OBJECT UPDATE
========================= */

function updateObjects(dt){

  for(let i=objects.length-1;i>=0;i--){

    let o=objects[i];

    o.y += speed*dt;

    o.x=laneX(o.lane,o.y);


    let t=(o.y-H*.28)/(H-H*.28);


    if(o.type==="coin"){

      o.size=10+t*22;

    }else{

      o.size=14+t*32;

    }


    let playerY=H*.76;


    if(
      Math.abs(o.y-playerY)<45 &&
      o.lane===playerLane
    ){

      if(o.type==="coin"){

        coinCount++;
        score+=10;

        objects.splice(i,1);

        updateUI();

        continue;

      }else{

        gameOver();

        return;
      }
    }


    if(o.y>H+80){

      objects.splice(i,1);
    }
  }
}


/* =========================
   MOVE PLAYER
========================= */

function moveLeft(){

  if(playerLane>0){

    playerLane--;

    updatePlayerPosition();
  }
}


function moveRight(){

  if(playerLane<2){

    playerLane++;

    updatePlayerPosition();
  }
}


function updatePlayerPosition(){

  targetX=laneX(
    playerLane,
    H*.76
  );
}


/* =========================
   TOUCH
========================= */

canvas.addEventListener(
  "touchstart",
  function(e){

    let t=e.touches[0];

    swipeStartX=t.clientX;
    swipeStartY=t.clientY;

  },
  {passive:false}
);


canvas.addEventListener(
  "touchend",
  function(e){

    let t=e.changedTouches[0];

    let dx=t.clientX-swipeStartX;
    let dy=t.clientY-swipeStartY;


    if(
      Math.abs(dx)>40 &&
      Math.abs(dx)>Math.abs(dy)
    ){

      if(dx<0){

        moveLeft();

      }else{

        moveRight();
      }
    }

  },
  {passive:false}
);


/* =========================
   KEYBOARD
========================= */

document.addEventListener(
  "keydown",
  function(e){

    if(e.key==="ArrowLeft"){
      moveLeft();
    }

    if(e.key==="ArrowRight"){
      moveRight();
    }

  }
);


/* =========================
   UI
========================= */

function updateUI(){

  document.getElementById("score").textContent =
    "Score: "+Math.floor(score);

  document.getElementById("coins").textContent =
    "🪙 "+coinCount;
}


/* =========================
   GAME OVER
========================= */

function gameOver(){

  running=false;

  document.getElementById("finalScore").textContent =
    "Score: "+Math.floor(score);

  document.getElementById("gameOver").style.display =
    "flex";
}


/* =========================
   RESTART
========================= */

function restartGame(){

  running=true;

  score=0;
  coinCount=0;

  speed=.20;

  playerLane=1;

  objects=[];

  spawnTimer=0;

  targetX=laneX(
    playerLane,
    H*.76
  );

  playerX=targetX;

  document.getElementById("gameOver").style.display =
    "none";

  updateUI();

  lastTime=performance.now();
}


/* =========================
   GAME LOOP
========================= */

function gameLoop(now){

  let dt=now-lastTime;

  lastTime=now;

  if(dt>50){
    dt=50;
  }


  if(running){

    roadOffset+=speed*dt;

    spawnTimer+=dt;


    if(spawnTimer>850){

      spawnObject();

      spawnTimer=0;
    }


    updateObjects(dt);


    /* SPEED BAHUT SLOWLY BADHEGI */
    speed+=0.000003*dt;


    score+=0.015*dt;

    updateUI();
  }


  ctx.clearRect(
    0,
    0,
    W,
    H
  );


  drawBackground();

  drawRoad();


  for(let o of objects){

    if(o.type==="coin"){

      drawCoin(o);

    }else{

      drawObstacle(o);
    }
  }


  drawPlayer();


  requestAnimationFrame(gameLoop);
}


/* =========================
   START
========================= */

targetX=laneX(
  playerLane,
  H*.76
);

playerX=targetX;

updateUI();

requestAnimationFrame(gameLoop);

</script>

</body>
</html>
