<!DOCTYPE html>
<html lang="mr">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>मंदिर रनर गेम</title>
    <style>
        body {
            margin: 0;
            padding: 0;
            background: #111;
            color: #fff;
            font-family: Arial, sans-serif;
            display: flex;
            flex-direction: column;
            align-items: center;
            justify-content: center;
            height: 100vh;
            overflow: hidden;
        }
        canvas {
            background: linear-gradient(to bottom, #87CEEB, #DEB887);
            border: 4px solid #fff;
            box-shadow: 0 0 20px rgba(255, 255, 255, 0.5);
        }
        .ui {
            margin-top: 10px;
            font-size: 20px;
            font-weight: bold;
        }
    </style>
</head>
<body>

    <div class="ui">स्कोअर: <span id="score">0</span></div>
    <canvas id="gameCanvas" width="400" height="600"></canvas>

<script>
    const canvas = document.getElementById("gameCanvas");
    const ctx = canvas.getContext("2d");

    let score = 0;
    let isGameOver = false;

    // खेळाडू (Player)
    let player = {
        x: 180,
        y: 480,
        width: 40,
        height: 60,
        speed: 5,
        lane: 1 // 0: डावी, 1: मध्य, 2: उजवी
    };

    const lanes = [90, 180, 270]; // रस्त्याचे तीन मार्ग

    let coins = [];
    let obstacles = [];

    // नाणी आणि अडथळे तयार करणे
    function spawnItems() {
        if (Math.random() < 0.03) {
            let laneIndex = Math.floor(Math.random() * 3);
            coins.push({ x: lanes[laneIndex], y: -50, radius: 15 });
        }
        if (Math.random() < 0.02) {
            let laneIndex = Math.floor(Math.random() * 3);
            obstacles.push({ x: lanes[laneIndex], y: -50, width: 40, height: 40 });
        }
    }

    // कीबोर्ड कंट்ரோल्स (किंवा मोबाईल टचसाठी)
    window.addEventListener("keydown", (e) => {
        if (e.key === "ArrowLeft" && player.lane > 0) {
            player.lane--;
        }
        if (e.key === "ArrowRight" && player.lane < 2) {
            player.lane++;
        }
    });

    // मोबाईल टच कंट்ரோल्स
    let touchStartX = 0;
    window.addEventListener("touchstart", (e) => {
        touchStartX = e.touches[0].clientX;
    });
    window.addEventListener("touchend", (e) => {
        let touchEndX = e.changedTouches[0].clientX;
        if (touchEndX < touchStartX - 30 && player.lane > 0) {
            player.lane--; // डावीकडे स्वाइप
        } else if (touchEndX > touchStartX + 30 && player.lane < 2) {
            player.lane++; // उजवीकडे स्वाइप
        }
    });

    function update() {
        if (isGameOver) return;

        // खेळाडूची जागा ठरवणे
        let targetX = lanes[player.lane];
        player.x += (targetX - player.x) * 0.2;

        spawnItems();

        // नाण्यांची हालचाल
        for (let i = coins.length - 1; i >= 0; i--) {
            coins[i].y += 4;

            // टक्कर तपासणी (Coin Collection)
            let dx = coins[i].x - (player.x + player.width / 2);
            let dy = coins[i].y - (player.y + player.height / 2);
            let distance = Math.sqrt(dx * dx + dy * dy);

            if (distance < 30) {
                score += 10;
                document.getElementById("score").innerText = score;
                coins.splice(i, 1);
            } else if (coins[i].y > canvas.height) {
                coins.splice(i, 1);
            }
        }

        // अडथळ्यांची हालचाल
        for (let i = obstacles.length - 1; i >= 0; i--) {
            obstacles[i].y += 4;

            // टक्कर तपासणी (Collision with Obstacle)
            if (
                player.x < obstacles[i].x + obstacles[i].width &&
                player.x + player.width > obstacles[i].x &&
                player.y < obstacles[i].y + obstacles[i].height &&
                player.y + player.height > obstacles[i].y
            ) {
                isGameOver = true;
                alert("गेम संपला! तुमचा स्कोअर: " + score);
                document.location.reload();
            } else if (obstacles[i].y > canvas.height) {
                obstacles.splice(i, 1);
            }
        }
    }

    function draw() {
        // पार्श्वभूमी (Background) साफ करणे
        ctx.clearRect(0, 0, canvas.width, canvas.height);

        // रस्ता काढणे (Temple path style)
        ctx.fillStyle = "#8B4513";
        ctx.fillRect(50, 0, 300, canvas.height);

        ctx.strokeStyle = "#FFD700";
        ctx.lineWidth = 4;
        ctx.beginPath();
        ctx.moveTo(140, 0); ctx.lineTo(140, canvas.height);
        ctx.moveTo(260, 0); ctx.lineTo(260, canvas.height);
        ctx.stroke();

        // नाणी काढणे (Coins)
        ctx.fillStyle = "gold";
        for (let c of coins) {
            ctx.beginPath();
            ctx.arc(c.x, c.y, c.radius, 0, Math.PI * 2);
            ctx.fill();
            ctx.stroke();
        }

        // अडथळे काढणे
        ctx.fillStyle = "red";
        for (let o of obstacles) {
            ctx.fillRect(o.x - 20, o.y, o.width, o.height);
        }

        // खेळाडू काढणे (Player)
        ctx.fillStyle = "#FF4500";
        ctx.fillRect(player.x - 20, player.y, player.width, player.height);
    }

    function loop() {
        update();
        draw();
        if (!isGameOver) {
            requestAnimationFrame(loop);
        }
    }

    loop();
</script>

</body>
</html> 
