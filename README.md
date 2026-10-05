<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width,initial-scale=1.0,maximum-scale=1.0,user-scalable=no">

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

/* =========================
LEFT RIGHT BUTTONS
========================= */

#controls{
  position:absolute;
  bottom:25px;
  left:0;
  width:100%;
  display:flex;
  justify-content:space-between;
  padding:0 25px;
  z-index:10;
  pointer-events:none;
}

.controlBtn{
  width:75px;
  height:75px;
  border-radius:50%;
  border:3px solid rgba(255,255,255,.8);
  background:rgba(0,0,0,.45);
  color:white;
  font-size:42px;
  font-weight:bold;
  display:flex;
  align-items:center;
  justify-content:center;
  user-select:none;
  pointer-events:auto;
  box-shadow:0 4px 12px rgba(0,0,0,.4);
}

.controlBtn:active{
  transform:scale(.90);
  background:rgba(255,255,255,.35);
}

#hint{
  position:absolute;
  bottom:108px;
  left:50%;
  transform:translateX(-50%);
  color:white;
  opacity:.75;
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

<div id="hint">Swipe or use buttons</div>


<!-- LEFT RIGHT BUTTONS -->

<div id="controls">

  <div
    class="controlBtn"
    id="leftBtn">
    ◀
  </div>

  <div
    class="controlBtn"
    id="rightBtn">
    ▶
  </div>

</div>


<div id="gameOver">

<h1>GAME OVER</h1>

<p id="finalScore">
Score: 0
</p>

<button onclick="restartGame()">
PLAY AGAIN
</button>

</div>

</div>


<script>

const canvas=
document.getElementById("canvas");

const ctx=
canvas.getContext("2d");

let W,H;

let dpr=
Math.min(
  window.devicePixelRatio||1,
  2
);


function resize(){

  W=window.innerWidth;
  H=window.innerHeight;

  canvas.width=W*dpr;
  canvas.height=H*dpr;

  canvas.style.width=W+"px";
  canvas.style.height=H+"px";

  ctx.setTransform(
    dpr,0,0,dpr,0,0
  );
}

window.addEventListener(
  "resize",
  resize
);

resize();


/* =========================
GAME VARIABLES
========================= */

let running=true;

let score=0;

let coinCount=0;

let speed=.20;

let roadOffset=0;

let playerLane=1;

let playerX=0;

let targetX=0;


/* FAST PLAYER */

let playerMoveSpeed=.78;


let objects=[];

let spawnTimer=0;

let lastTime=
performance.now();


/* SWIPE */

let swipeStartX=0;
let swipeStartY=0;


/* TRAIN */

let trainX=-600;

let trainSpeed=.11;

let trainY=H*.40;


/* =========================
ROAD
========================= */

function roadWidth(y){

  let horizon=H*.28;

  let t=
  (y-horizon)/
  (H-horizon);

  t=
  Math.max(
    0,
    Math.min(1,t)
  );

  return 100+t*W*.92;
}


function laneX(lane,y){

  let rw=
  roadWidth(y);

  let laneW=
  rw/3;

  return W/2-rw/2+
  laneW*(lane+.5);
}


/* =========================
BACKGROUND
========================= */

function drawBackground(){

  let sky=
  ctx.createLinearGradient(
    0,0,0,H*.55
  );

  sky.addColorStop(
    0,
    "#1660a0"
  );

  sky.addColorStop(
    .55,
    "#55b5d0"
  );

  sky.addColorStop(
    1,
    "#b5e0d1"
  );

  ctx.fillStyle=sky;

  ctx.fillRect(
    0,0,W,H*.60
  );


  /* SUN */

  ctx.beginPath();

  ctx.arc(
    W*.82,
    H*.13,
    38,
    0,
    Math.PI*2
  );

  ctx.fillStyle="#ffe99a";

  ctx.fill();


  /* GROUND */

  ctx.fillStyle="#62a94b";

  ctx.fillRect(
    0,H*.43,
    W,H*.57
  );


  /* MOUNTAINS */

  ctx.fillStyle="#43835a";

  ctx.beginPath();

  ctx.moveTo(
    0,H*.43
  );

  ctx.lineTo(
    W*.12,H*.34
  );

  ctx.lineTo(
    W*.23,H*.42
  );

  ctx.lineTo(
    W*.38,H*.33
  );

  ctx.lineTo(
    W*.52,H*.42
  );

  ctx.lineTo(
    W*.70,H*.34
  );

  ctx.lineTo(
    W*.84,H*.42
  );

  ctx.lineTo(
    W,H*.34
  );

  ctx.lineTo(
    W,H*.52
  );

  ctx.lineTo(
    0,H*.52
  );

  ctx.closePath();

  ctx.fill();


  drawTree(
    55,
    H*.40,
    .65
  );

  drawTree(
    W-55,
    H*.40,
    .7
  );
}


/* =========================
TREE
========================= */

function drawTree(x,y,s){

  ctx.save();

  ctx.translate(x,y);

  ctx.scale(s,s);

  ctx.fillStyle="#704321";

  ctx.fillRect(
    -7,0,14,70
  );

  ctx.fillStyle="#1b7137";

  ctx.beginPath();

  ctx.arc(
    0,-5,
    34,
    0,
    Math.PI*2
  );

  ctx.fill();

  ctx.beginPath();

  ctx.arc(
    -27,15,
    25,
    0,
    Math.PI*2
  );

  ctx.fill();

  ctx.beginPath();

  ctx.arc(
    27,15,
    25,
    0,
    Math.PI*2
  );

  ctx.fill();

  ctx.restore();
}


/* =========================
HOUSE
========================= */

function drawHouse(x,y,s){

  ctx.save();

  ctx.translate(x,y);

  ctx.scale(s,s);


  ctx.fillStyle=
  "rgba(0,0,0,.18)";

  ctx.beginPath();

  ctx.ellipse(
    0,58,
    55,10,
    0,0,Math.PI*2
  );

  ctx.fill();


  /* WALL */

  ctx.fillStyle="#f0c27b";

  ctx.fillRect(
    -43,0,
    86,55
  );


  /* ROOF */

  ctx.fillStyle="#9e3c27";

  ctx.beginPath();

  ctx.moveTo(-54,0);

  ctx.lineTo(0,-43);

  ctx.lineTo(54,0);

  ctx.closePath();

  ctx.fill();


  /* DOOR */

  ctx.fillStyle="#70401f";

  ctx.fillRect(
    -10,25,
    20,30
  );


  /* WINDOWS */

  ctx.fillStyle="#75d1e8";

  ctx.fillRect(
    -34,15,
    18,16
  );

  ctx.fillRect(
    16,15,
    18,16
  );


  ctx.restore();
}


/* =========================
PEOPLE
========================= */

function drawPerson(
  x,y,s,shirt,skin
){

  ctx.save();

  ctx.translate(x,y);

  ctx.scale(s,s);


  /* SHADOW */

  ctx.fillStyle=
  "rgba(0,0,0,.25)";

  ctx.beginPath();

  ctx.ellipse(
    0,25,
    13,5,
    0,0,Math.PI*2
  );

  ctx.fill();


  /* HEAD */

  ctx.fillStyle=skin;

  ctx.beginPath();

  ctx.arc(
    0,-25,
    10,
    0,
    Math.PI*2
  );

  ctx.fill();


  /* HAIR */

  ctx.fillStyle="#211810";

  ctx.beginPath();

  ctx.arc(
    0,-30,
    10,
    Math.PI,
    Math.PI*2
  );

  ctx.fill();


  /* BODY */

  ctx.fillStyle=shirt;

  ctx.beginPath();

  ctx.roundRect(
    -10,-14,
    20,30,
    5
  );

  ctx.fill();


  /* ARMS */

  ctx.strokeStyle=skin;

  ctx.lineWidth=6;

  ctx.beginPath();

  ctx.moveTo(-8,-7);

  ctx.lineTo(-19,8);

  ctx.moveTo(8,-7);

  ctx.lineTo(19,8);

  ctx.stroke();


  /* LEGS */

  ctx.strokeStyle="#222";

  ctx.lineWidth=7;

  ctx.beginPath();

  ctx.moveTo(-4,14);

  ctx.lineTo(-10,31);

  ctx.moveTo(4,14);

  ctx.lineTo(10,31);

  ctx.stroke();


  ctx.restore();
}


/* =========================
VILLAGE
========================= */

function drawVillage(){

  drawHouse(
    W*.08,
    H*.57,
    .85
  );

  drawHouse(
    W*.17,
    H*.72,
    .62
  );

  drawHouse(
    W*.91,
    H*.59,
    .80
  );

  drawHouse(
    W*.82,
    H*.74,
    .62
  );


  drawPerson(
    W*.27,
    H*.66,
    1.05,
    "#2674c8",
    "#c98251"
  );

  drawPerson(
    W*.72,
    H*.68,
    1.05,
    "#d14b58",
    "#b87345"
  );

  drawPerson(
    W*.12,
    H*.82,
    1.15,
    "#3a9b55",
    "#c98251"
  );

  drawPerson(
    W*.88,
    H*.84,
    1.15,
    "#d88a29",
    "#b87345"
  );


  drawTree(
    W*.035,
    H*.68,
    .7
  );

  drawTree(
    W*.965,
    H*.70,
    .7
  );
}


/* =========================
RAILWAY
========================= */

function drawRailway(){

  let horizon=H*.31;

  let rail1=W*.045;

  let rail2=W*.105;


  ctx.strokeStyle="#bcbcbc";

  ctx.lineWidth=4;

  ctx.beginPath();

  ctx.moveTo(
    rail1,
    horizon
  );

  ctx.lineTo(
    rail1-15,
    H
  );

  ctx.moveTo(
    rail2,
    horizon
  );

  ctx.lineTo(
    rail2+20,
    H
  );

  ctx.stroke();


  for(
    let y=horizon;
    y<H;
    y+=30
  ){

    let t=
    (y-horizon)/
    (H-horizon);

    let x1=
    rail1-t*15;

    let x2=
    rail2+t*20;

    ctx.strokeStyle="#65442d";

    ctx.lineWidth=
    4+t*8;

    ctx.beginPath();

    ctx.moveTo(
      x1-10,
      y
    );

    ctx.lineTo(
      x2+10,
      y
    );

    ctx.stroke();
  }
}


/* =========================
TRAIN
========================= */

function drawTrain(){

  let y=trainY;

  let engineW=170;

  let coachW=75;


  ctx.fillStyle=
  "rgba(0,0,0,.25)";

  ctx.fillRect(
    trainX,
    y+53,
    engineW+coachW*3,
    8
  );


  /* ENGINE */

  ctx.fillStyle="#d53a32";

  ctx.fillRect(
    trainX,
    y,
    engineW,
    48
  );


  /* CABIN */

  ctx.fillStyle="#b92e2b";

  ctx.fillRect(
    trainX+25,
    y-25,
    55,
    25
  );


  /* WINDOWS */

  ctx.fillStyle="#9de0ec";

  ctx.fillRect(
    trainX+35,
    y-18,
    17,14
  );

  ctx.fillRect(
    trainX+58,
    y-18,
    17,14
  );


  /* COACHES */

  for(
    let i=0;
    i<3;
    i++
  ){

    let bx=
    trainX+
    engineW+
    i*coachW;

    ctx.fillStyle=
    i%2===0
    ?"#e0b52e"
    :"#d79d25";

    ctx.fillRect(
      bx,
      y+3,
      coachW-5,
      45
    );


    ctx.fillStyle="#75c9df";

    for(
      let w=0;
      w<2;
      w++
    ){

      ctx.fillRect(
        bx+10+w*27,
        y+12,
        18,14
      );
    }
  }


  /* WHEELS */

  ctx.fillStyle="#202020";

  for(
    let i=0;
    i<10;
    i++
  ){

    let wx=
    trainX+
    15+
    i*40;

    ctx.beginPath();

    ctx.arc(
      wx,
      y+52,
      7,
      0,
      Math.PI*2
    );

    ctx.fill();
  }
}


function updateTrain(dt){

  trainX+=
  trainSpeed*dt;

  let trainWidth=
  170+75*3;

  if(
    trainX>W+100
  ){

    trainX=
    -trainWidth-100;
  }
}


/* =========================
ROAD
========================= */

function drawRoad(){

  let horizon=H*.28;

  let bottomWidth=W*1.05;


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


  /* EDGES */

  ctx.strokeStyle="#eeeeee";

  ctx.lineWidth=5;

  ctx.beginPath();

  ctx.moveTo(
    W/2-50,
    horizon
  );

  ctx.lineTo(
    W/2-bottomWidth/2,
    H
  );

  ctx.moveTo(
    W/2+50,
    horizon
  );

  ctx.lineTo(
    W/2+bottomWidth/2,
    H
  );

  ctx.stroke();


  /* LANE MARKINGS */

  let dash=55;

  for(
    let y=
    horizon+
    (roadOffset%dash)-
    dash;

    y<H;

    y+=dash
  ){

    let t=
    (y-horizon)/
    (H-horizon);

    let rw=
    roadWidth(y);

    let laneW=
    rw/3;

    ctx.fillStyle="#eeeeee";

    for(
      let i=1;
      i<3;
      i++
    ){

      let x=
      W/2-rw/2+
      laneW*i;

      let markH=
      18+t*30;

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
PLAYER
========================= */

function drawPlayer(){

  let y=H*.76;


  /* FAST MOVEMENT */

  playerX +=
  (targetX-playerX)*
  playerMoveSpeed;


  ctx.save();

  ctx.translate(
    playerX,
    y
  );


  /* SHADOW */

  ctx.fillStyle=
  "rgba(0,0,0,.35)";

  ctx.beginPath();

  ctx.ellipse(
    0,48,
    34,10,
    0,0,Math.PI*2
  );

  ctx.fill();


  /* CAR */

  ctx.fillStyle="#e52e35";

  ctx.beginPath();

  ctx.roundRect(
    -25,-45,
    50,90,
    10
  );

  ctx.fill();


  /* ROOF */

  ctx.fillStyle="#172b3d";

  ctx.beginPath();

  ctx.roundRect(
    -17,-30,
    34,35,
    7
  );

  ctx.fill();


  /* GLASS */

  ctx.fillStyle="#65b8d1";

  ctx.beginPath();

  ctx.moveTo(-14,-25);

  ctx.lineTo(14,-25);

  ctx.lineTo(13,-8);

  ctx.lineTo(-13,-8);

  ctx.closePath();

  ctx.fill();


  /* LIGHTS */

  ctx.fillStyle="#fff4a3";

  ctx.fillRect(
    -21,-39,
    9,7
  );

  ctx.fillRect(
    12,-39,
    9,7
  );


  /* BACK LIGHT */

  ctx.fillStyle="#ff2525";

  ctx.fillRect(
    -21,32,
    9,7
  );

  ctx.fillRect(
    12,32,
    9,7
  );


  /* WHEELS */

  ctx.fillStyle="#111";

  ctx.fillRect(
    -29,-28,
    7,22
  );

  ctx.fillRect(
    22,-28,
    7,22
  );

  ctx.fillRect(
    -29,18,
    7,22
  );

  ctx.fillRect(
    22,18,
    7,22
  );

  ctx.restore();
}


/* =========================
ENEMY
========================= */

function drawObstacle(o){

  let y=o.y;

  let s=o.size;

  ctx.save();

  ctx.translate(
    o.x,
    y
  );

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


  ctx.fillStyle="#172b3d";

  ctx.fillRect(
    -s*.45,
    -s*.65,
    s*.9,
    s*.55
  );


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

  ctx.translate(
    o.x,
    o.y
  );

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
  Math.max(10,o.size)+
  "px Arial";

  ctx.textAlign="center";

  ctx.textBaseline="middle";

  ctx.fillText(
    "₹",
    0,
    1
  );

  ctx.restore();
}


/* =========================
SPAWN
========================= */

function spawnObject(){

  let lane=
  Math.floor(
    Math.random()*3
  );

  let type=
  Math.random()<.28
  ?"coin"
  :"car";


  objects.push({

    lane:lane,

    y:H*.28-50,

    x:laneX(
      lane,
      H*.28
    ),

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

    let o=
    objects[i];

    o.y+=
    speed*dt;

    o.x=
    laneX(
      o.lane,
      o.y
    );


    let t=
    (o.y-H*.28)/
    (H-H*.28);


    if(
      o.type==="coin"
    ){

      o.size=
      10+t*22;

    }else{

      o.size=
      14+t*32;
    }


    let playerY=
    H*.76;


    if(
      Math.abs(
        o.y-playerY
      )<45 &&
      o.lane===playerLane
    ){

      if(
        o.type==="coin"
      ){

        coinCount++;

        score+=10;

        objects.splice(
          i,
          1
        );

        updateUI();

        continue;

      }else{

        gameOver();

        return;
      }
    }


    if(
      o.y>H+80
    ){

      objects.splice(
        i,
        1
      );
    }
  }
}


/* =========================
MOVE
========================= */

function moveLeft(){

  if(
    playerLane>0
  ){

    playerLane--;

    updatePlayerPosition();
  }
}


function moveRight(){

  if(
    playerLane<2
  ){

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
BUTTON CONTROLS
========================= */

const leftBtn=
document.getElementById(
  "leftBtn"
);

const rightBtn=
document.getElementById(
  "rightBtn"
);


/* LEFT BUTTON */

leftBtn.addEventListener(
  "touchstart",
  function(e){

    e.preventDefault();

    moveLeft();

  },
  {passive:false}
);


leftBtn.addEventListener(
  "mousedown",
  function(e){

    e.preventDefault();

    moveLeft();

  }
);


/* RIGHT BUTTON */

rightBtn.addEventListener(
  "touchstart",
  function(e){

    e.preventDefault();

    moveRight();

  },
  {passive:false}
);


rightBtn.addEventListener(
  "mousedown",
  function(e){

    e.preventDefault();

    moveRight();

  }
);


/* =========================
SWIPE CONTROL
========================= */

canvas.addEventListener(
  "touchstart",
  function(e){

    let t=
    e.touches[0];

    swipeStartX=
    t.clientX;

    swipeStartY=
    t.clientY;

  },
  {passive:false}
);


canvas.addEventListener(
  "touchend",
  function(e){

    let t=
    e.changedTouches[0];

    let dx=
    t.clientX-
    swipeStartX;

    let dy=
    t.clientY-
    swipeStartY;


    if(
      Math.abs(dx)>20 &&
      Math.abs(dx)>
      Math.abs(dy)
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

    if(
      e.key==="ArrowLeft"
    ){

      moveLeft();
    }

    if(
      e.key==="ArrowRight"
    ){

      moveRight();
    }
  }
);


/* =========================
UI
========================= */

function updateUI(){

  document.getElementById(
    "score"
  ).textContent=
  "Score: "+
  Math.floor(score);


  document.getElementById(
    "coins"
  ).textContent=
  "🪙 "+
  coinCount;
}


/* =========================
GAME OVER
========================= */

function gameOver(){

  running=false;

  document.getElementById(
    "finalScore"
  ).textContent=
  "Score: "+
  Math.floor(score);

  document.getElementById(
    "gameOver"
  ).style.display="flex";
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

  trainX=-600;


  targetX=
  laneX(
    playerLane,
    H*.76
  );

  playerX=
  targetX;


  document.getElementById(
    "gameOver"
  ).style.display="none";


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

    roadOffset+=
    speed*dt;


    spawnTimer+=dt;


    if(
      spawnTimer>850
    ){

      spawnObject();

      spawnTimer=0;
    }


    updateObjects(dt);

    updateTrain(dt);


    score+=
    .015*dt;


    updateUI();
  }


  ctx.clearRect(
    0,0,W,H
  );


  drawBackground();

  drawRailway();

  drawTrain();

  drawVillage();

  drawRoad();


  for(
    let o of objects
  ){

    if(
      o.type==="coin"
    ){

      drawCoin(o);

    }else{

      drawObstacle(o);
    }
  }


  drawPlayer();


  requestAnimationFrame(
    gameLoop
  );
}


/* =========================
START
========================= */

targetX=
laneX(
  playerLane,
  H*.76
);

playerX=
targetX;

updateUI();

requestAnimationFrame(
  gameLoop
);

</script>

</body>
</html>
