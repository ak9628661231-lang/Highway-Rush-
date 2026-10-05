<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8">
<meta name="viewport"
content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no">

<title>Highway Rush</title>

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
  background:#07111d;
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

#hint{
  position:absolute;
  bottom:18px;
  left:50%;
  transform:translateX(-50%);
  color:white;
  opacity:.7;
  font-size:14px;
  z-index:5;
}

#gameOver{
  display:none;
  position:absolute;
  inset:0;
  background:rgba(0,0,0,.68);
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
</style>
</head>

<body>

<div id="game">

<canvas id="canvas"></canvas>

<div id="score">Score: 0</div>
<div id="coins">🪙 0</div>

<div id="hint">Swipe ← → to move</div>

<div id="gameOver">
  <h1>GAME OVER</h1>
  <p id="finalScore">Score: 0</p>
  <button onclick="restartGame()">PLAY AGAIN</button>
</div>

</div>

<script>

const canvas=document.getElementById("canvas");
const ctx=canvas.getContext("2d");

let W,H;
let dpr=Math.min(window.devicePixelRatio||1,2);

function resize(){

  W=window.innerWidth;
  H=window.innerHeight;

  canvas.width=W*dpr;
  canvas.height=H*dpr;

  canvas.style.width=W+"px";
  canvas.style.height=H+"px";

  ctx.setTransform(dpr,0,0,dpr,0,0);
}

window.addEventListener("resize",resize);
resize();

/* =========================
GAME VARIABLES
========================= */

let running=true;

let score=0;
let coinCount=0;

/* ROAD SPEED SLOW */
let speed=.20;

let roadOffset=0;

let playerLane=1;

let playerX=0;
let targetX=0;

/* FAST PLAYER MOVEMENT */
let playerMoveSpeed=0.45;

let objects=[];
let scenery=[];

let spawnTimer=0;
let sceneryTimer=0;

let lastTime=performance.now();

let swipeStartX=0;
let swipeStartY=0;

/* =========================
ROAD
========================= */

function roadWidth(y){

  let horizon=H*.28;

  let t=(y-horizon)/(H-horizon);

  t=Math.max(0,Math.min(1,t));

  return 100+t*W*.92;
}

function roadCenter(){
  return W/2;
}

function laneX(lane,y){

  let rw=roadWidth(y);

  let laneW=rw/3;

  return roadCenter()-rw/2+
         laneW*(lane+.5);
}

/* =========================
BACKGROUND
========================= */

function drawBackground(){

  /* SKY */

  let sky=ctx.createLinearGradient(
    0,0,0,H*.55
  );

  sky.addColorStop(0,"#174b91");
  sky.addColorStop(.55,"#43a8d1");
  sky.addColorStop(1,"#9bd6dc");

  ctx.fillStyle=sky;
  ctx.fillRect(0,0,W,H*.60);

  /* SUN */

  ctx.beginPath();

  ctx.arc(
    W*.82,
    H*.13,
    35,
    0,
    Math.PI*2
  );

  ctx.fillStyle="#ffe58a";
  ctx.fill();

  /* GROUND */

  ctx.fillStyle="#4c9b45";
  ctx.fillRect(0,H*.43,W,H*.57);

  /* DISTANT TREES */

  drawTree(60,H*.37,.65);
  drawTree(W-60,H*.37,.7);

  drawTree(W*.25,H*.39,.5);
  drawTree(W*.75,H*.39,.5);
}

/* =========================
TREE
========================= */

function drawTree(x,y,s){

  ctx.save();

  ctx.translate(x,y);
  ctx.scale(s,s);

  /* trunk */

  ctx.fillStyle="#70401f";
  ctx.fillRect(-7,0,14,70);

  /* leaves */

  ctx.fillStyle="#176b35";

  ctx.beginPath();
  ctx.arc(0,-5,32,0,Math.PI*2);
  ctx.fill();

  ctx.beginPath();
  ctx.arc(-25,15,25,0,Math.PI*2);
  ctx.fill();

  ctx.beginPath();
  ctx.arc(25,15,25,0,Math.PI*2);
  ctx.fill();

  ctx.restore();
}

/* =========================
HOUSE
========================= */

function drawHouse(x,y,s,side){

  ctx.save();

  ctx.translate(x,y);
  ctx.scale(s,s);

  /* shadow */

  ctx.fillStyle="rgba(0,0,0,.2)";
  ctx.fillRect(-42,48,84,10);

  /* wall */

  ctx.fillStyle="#f1c27d";
  ctx.fillRect(-40,0,80,52);

  /* roof */

  ctx.fillStyle="#a83e2b";

  ctx.beginPath();

  ctx.moveTo(-50,0);
  ctx.lineTo(0,-38);
  ctx.lineTo(50,0);

  ctx.closePath();

  ctx.fill();

  /* door */

  ctx.fillStyle="#70401f";
  ctx.fillRect(-9,25,18,27);

  /* windows */

  ctx.fillStyle="#7ed5e8";

  ctx.fillRect(-31,14,16,15);
  ctx.fillRect(15,14,16,15);

  /* window lines */

  ctx.strokeStyle="#315b65";
  ctx.lineWidth=2;

  ctx.beginPath();
  ctx.moveTo(-23,14);
  ctx.lineTo(-23,29);
  ctx.moveTo(-31,21.5);
  ctx.lineTo(-15,21.5);
  ctx.moveTo(23,14);
  ctx.lineTo(23,29);
  ctx.moveTo(15,21.5);
  ctx.lineTo(31,21.5);
  ctx.stroke();

  ctx.restore();
}

/* =========================
PERSON
========================= */

function drawPerson(x,y,s){

  ctx.save();

  ctx.translate(x,y);
  ctx.scale(s,s);

  /* head */

  ctx.fillStyle="#d99a68";

  ctx.beginPath();
  ctx.arc(0,-22,7,0,Math.PI*2);
  ctx.fill();

  /* body */

  ctx.fillStyle="#2474c8";
  ctx.fillRect(-7,-14,14,22);

  /* legs */

  ctx.strokeStyle="#222";
  ctx.lineWidth=5;

  ctx.beginPath();

  ctx.moveTo(-3,8);
  ctx.lineTo(-8,22);

  ctx.moveTo(3,8);
  ctx.lineTo(8,22);

  ctx.stroke();

  /* arms */

  ctx.beginPath();

  ctx.moveTo(-6,-8);
  ctx.lineTo(-15,3);

  ctx.moveTo(6,-8);
  ctx.lineTo(15,3);

  ctx.stroke();

  ctx.restore();
}

/* =========================
RAILWAY TRACK
========================= */

function drawRailway(){

  let horizon=H*.30;

  let leftX=W*.05;
  let rightX=W*.25;

  /* sleepers */

  for(let y=horizon;y<H;y+=42){

    let t=(y-horizon)/(H-horizon);

    let x1=leftX-t*20;
    let x2=rightX+t*20;

    ctx.strokeStyle="#65432c";
    ctx.lineWidth=Math.max(3,t*12);

    ctx.beginPath();

    ctx.moveTo(x1,y);
    ctx.lineTo(x2,y);

    ctx.stroke();
  }

  /* rails */

  ctx.strokeStyle="#b9b9b9";
  ctx.lineWidth=4;

  ctx.beginPath();

  ctx.moveTo(leftX,horizon);
  ctx.lineTo(leftX-20,H);

  ctx.moveTo(rightX,horizon);
  ctx.lineTo(rightX+20,H);

  ctx.stroke();

  /* second pair */

  ctx.strokeStyle="#777";
  ctx.lineWidth=3;

  ctx.beginPath();

  ctx.moveTo(leftX+18,horizon);
  ctx.lineTo(leftX,H);

  ctx.moveTo(rightX+18,horizon);
  ctx.lineTo(rightX+38,H);

  ctx.stroke();
}

/* =========================
SIDE SCENERY
========================= */

function drawSideScenery(){

  /* houses */

  drawHouse(
    W*.08,
    H*.58,
    .8,
    "left"
  );

  drawHouse(
    W*.90,
    H*.60,
    .75,
    "right"
  );

  drawHouse(
    W*.16,
    H*.72,
    .65,
    "left"
  );

  drawHouse(
    W*.83,
    H*.76,
    .65,
    "right"
  );

  /* people */

  drawPerson(
    W*.30,
    H*.66,
    .75
  );

  drawPerson(
    W*.70,
    H*.70,
    .75
  );

  drawPerson(
    W*.11,
    H*.83,
    .9
  );

  drawPerson(
    W*.89,
    H*.84,
    .9
  );

  /* trees */

  drawTree(
    W*.04,
    H*.68,
    .65
  );

  drawTree(
    W*.96,
    H*.70,
    .65
  );
}

/* =========================
ROAD
========================= */

function drawRoad(){

  let horizon=H*.28;

  let bottomWidth=W*1.05;

  /* road */

  ctx.fillStyle="#293238";

  ctx.beginPath();

  ctx.moveTo(
    W/2-50,
    horizon
  );

  ctx.lineTo(
    W/2+50,
    horizon
  );

  ctx.lineTo(
    W/2+bottomWidth/2,
    H
  );

  ctx.lineTo(
    W/2-bottomWidth/2,
    H
  );

  ctx.closePath();

  ctx.fill();

  /* road edges */

  ctx.strokeStyle="#e7e7e7";
  ctx.lineWidth=5;

  ctx.beginPath();

  ctx.moveTo(W/2-50,horizon);
  ctx.lineTo(W/2-bottomWidth/2,H);

  ctx.moveTo(W/2+50,horizon);
  ctx.lineTo(W/2+bottomWidth/2,H);

  ctx.stroke();

  /* lane markings */

  let dash=55;

  for(
    let y=horizon+(roadOffset%dash)-dash;
    y<H;
    y+=dash
  ){

    let t=(y-horizon)/(H-horizon);

    let rw=roadWidth(y);
    let laneW=rw/3;

    ctx.fillStyle="#eeeeee";

    for(let i=1;i<3;i++){

      let x=
        W/2-rw/2+
        laneW*i;

      let markH=18+t*30;

      ctx.fillRect(
        x-3,
        y,
        6,
        markH
      );
    }
  }
}

/* =========================
PLAYER CAR
========================= */

function drawPlayer(){

  let y=H*.76;

  /* FAST SMOOTH MOVEMENT */

  playerX +=
    (targetX-playerX)*
    playerMoveSpeed;

  ctx.save();

  ctx.translate(playerX,y);

  /* shadow */

  ctx.fillStyle="rgba(0,0,0,.35)";

  ctx.beginPath();

  ctx.ellipse(
    0,
    48,
    34,
    10,
    0,
    0,
    Math.PI*2
  );

  ctx.fill();

  /* car body */

  ctx.fillStyle="#e52e35";

  ctx.beginPath();

  ctx.roundRect(
    -25,
    -45,
    50,
    90,
    10
  );

  ctx.fill();

  /* roof/window */

  ctx.fillStyle="#172b3d";

  ctx.beginPath();

  ctx.roundRect(
    -17,
    -30,
    34,
    35,
    7
  );

  ctx.fill();

  /* windshield */

  ctx.fillStyle="#65b8d1";

  ctx.beginPath();

  ctx.moveTo(-14,-25);
  ctx.lineTo(14,-25);
  ctx.lineTo(13,-8);
  ctx.lineTo(-13,-8);

  ctx.closePath();

  ctx.fill();

  /* lights */

  ctx.fillStyle="#fff4a3";

  ctx.fillRect(-21,-39,9,7);
  ctx.fillRect(12,-39,9,7);

  /* back lights */

  ctx.fillStyle="#ff2525";

  ctx.fillRect(-21,32,9,7);
  ctx.fillRect(12,32,9,7);

  /* wheels */

  ctx.fillStyle="#111";

  ctx.fillRect(-29,-28,7,22);
  ctx.fillRect(22,-28,7,22);

  ctx.fillRect(-29,18,7,22);
  ctx.fillRect(22,18,7,22);

  ctx.restore();
}

/* =========================
OBSTACLE
========================= */

function drawObstacle(o){

  let y=o.y;
  let s=o.size;

  ctx.save();

  ctx.translate(o.x,y);

  /* car */

  ctx.fillStyle="#2468d8";

  ctx.beginPath();

  ctx.roundRect(
    -s*.75,
    -s,
    s*1.5,
    s*2,
    s*.2
  );

  ctx.fill();

  /* window */

  ctx.fillStyle="#172b3d";

  ctx.fillRect(
    -s*.45,
    -s*.65,
    s*.9,
    s*.55
  );

  /* lights */

  ctx.fillStyle="#ffe889";

  ctx.fillRect(
    -s*.58,
    -s*.85,
    s*.25,
    s*.18
  );

  ctx.fillRect(
    s*.33,
    -s*.85,
    s*.25,
    s*.18
  );

  ctx.restore();
}

/* =========================
COIN
========================= */

function drawCoin(o){

  ctx.save();

  ctx.translate(o.x,o.y);

  ctx.beginPath();

  ctx.arc(
    0,
    0,
    o.size,
    0,
    Math.PI*2
  );

  ctx.fillStyle="#ffd42a";
  ctx.fill();

  ctx.strokeStyle="#fff09b";
  ctx.lineWidth=3;
  ctx.stroke();

  ctx.fillStyle="#9a6a00";

  ctx.font=
    Math.max(10,o.size)+"px Arial";

  ctx.textAlign="center";
  ctx.textBaseline="middle";

  ctx.fillText("₹",0,1);

  ctx.restore();
}

/* =========================
SPAWN OBJECT
========================= */

function spawnObject(){

  let lane=
    Math.floor(Math.random()*3);

  let type=
    Math.random()<.28
    ?"coin"
    :"car";

  objects.push({

    lane:lane,

    y:H*.28-50,

    x:laneX(lane,H*.28),

    type:type,

    size:20
  });
}

/* =========================
UPDATE OBJECTS
========================= */

function updateObjects(dt){

  for(
    let i=objects.length-1;
    i>=0;
    i--
  ){

    let o=objects[i];

    o.y+=speed*dt;

    o.x=
      laneX(
        o.lane,
        o.y
      );

    let t=
      (o.y-H*.28)/
      (H-H*.28);

    if(o.type==="coin"){

      o.size=
        10+t*22;

    }else{

      o.size=
        14+t*32;
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

  targetX=
    laneX(
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

    let dx=
      t.clientX-swipeStartX;

    let dy=
      t.clientY-swipeStartY;

    if(
      Math.abs(dx)>25 &&
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

  document.getElementById("score")
  .textContent=
  "Score: "+Math.floor(score);

  document.getElementById("coins")
  .textContent=
  "🪙 "+coinCount;
}

/* =========================
GAME OVER
========================= */

function gameOver(){

  running=false;

  document.getElementById("finalScore")
  .textContent=
  "Score: "+Math.floor(score);

  document.getElementById("gameOver")
  .style.display="flex";
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

  targetX=
    laneX(
      playerLane,
      H*.76
    );

  playerX=targetX;

  document.getElementById("gameOver")
  .style.display="none";

  updateUI();

  lastTime=
    performance.now();
}

/* =========================
GAME LOOP
========================= */

function gameLoop(now){

  let dt=
    now-lastTime;

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

    /* speed very slowly increases */

    speed+=
      .000003*dt;

    score+=
      .015*dt;

    updateUI();
  }

  ctx.clearRect(
    0,
    0,
    W,
    H
  );

  drawBackground();

  /* railway */

  drawRailway();

  /* houses + people */

  drawSideScenery();

  /* road */

  drawRoad();

  /* objects */

  for(let o of objects){

    if(o.type==="coin"){

      drawCoin(o);

    }else{

      drawObstacle(o);
    }
  }

  /* player */

  drawPlayer();

  requestAnimationFrame(gameLoop);
}

/* =========================
START
========================= */

targetX=
  laneX(
    playerLane,
    H*.76
  );

playerX=targetX;

updateUI();

requestAnimationFrame(gameLoop);

</script>

</body>
</html>
