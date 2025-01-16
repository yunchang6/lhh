#运常
<!DOCTYPE html>
<html lang="zh-cn">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>浪漫烟花效果</title>
    <style>
        body {
            margin: 0;
            overflow: hidden;
            background-color: #000; /* 背景为黑色 */
            display: flex;
            justify-content: center;
            align-items: center;
            height: 100vh;
        }
        canvas {
            display: block;
        }
        #toggleButton {
            position: absolute;
            bottom: 20px;
            right: 20px;
            padding: 10px 20px;
            background-color: #ff4081;
            color: white;
            border: none;
            border-radius: 5px;
            box-shadow: 0 4px 6px rgba(0, 0, 0, 0.1);
            cursor: pointer;
            z-index: 10;
            font-size: 16px;
        }
        #toggleButton:hover {
            background-color: #ff79a6;
        }
    </style>
</head>
<body>
    <canvas id="fireworksCanvas"></canvas>
    <audio id="fireworkSound" src="firework.mp3"></audio>
    <button id="toggleButton">开始烟花</button>
    <script>
        const canvas = document.getElementById('fireworksCanvas');
        const ctx = canvas.getContext('2d');
        const fireworkSound = document.getElementById('fireworkSound');
        const toggleButton = document.getElementById('toggleButton');
        let fireworks = [];
        let particles = [];
        const colors = ['#FF1461', '#18FF92', '#5A87FF', '#FBF38C'];
        let fireworkCount = 0;
        let isFireworksActive = false;

        canvas.width = window.innerWidth;
        canvas.height = window.innerHeight;

        toggleButton.addEventListener('click', () => {
            isFireworksActive = !isFireworksActive;
            toggleButton.textContent = isFireworksActive ? '停止烟花' : '开始烟花';
        });

        class Firework {
            constructor(x, y, targetX, targetY) {
                this.x = x;
                this.y = y;
                this.targetX = targetX;
                this.targetY = targetY;
                this.distanceToTarget = this.calculateDistance(x, y, targetX, targetY);
                this.distanceTraveled = 0;
                this.coordinates = [];
                this.coordinateCount = 3;
                while (this.coordinateCount--) {
                    this.coordinates.push([this.x, this.y]);
                }
                this.angle = Math.atan2(targetY - y, targetX - x);
                this.speed = 1; // 调整速度更慢
                this.acceleration = 1.02; // 调整加速度
                this.brightness = Math.random() * 50 + 50;
                this.targetRadius = 1;
            }

            calculateDistance(x1, y1, x2, y2) {
                const xDistance = x2 - x1;
                const yDistance = y2 - y1;
                return Math.sqrt(Math.pow(xDistance, 2) + Math.pow(yDistance, 2));
            }

            update(index) {
                this.coordinates.pop();
                this.coordinates.unshift([this.x, this.y]);

                if (this.targetRadius < 8) {
                    this.targetRadius += 0.3;
                } else {
                    this.targetRadius = 1;
                }

                this.speed *= this.acceleration;

                const vx = Math.cos(this.angle) * this.speed;
                const vy = Math.sin(this.angle) * this.speed;
                this.distanceTraveled = this.calculateDistance(this.x, this.y, this.x + vx, this.y + vy);

                if (this.distanceTraveled >= this.distanceToTarget) {
                    createTextParticles(this.targetX, this.targetY, "欢欢");
                    fireworks.splice(index, 1);
                    fireworkSound.currentTime = 0;
                    fireworkSound.play();
                } else {
                    this.x += vx;
                    this.y += vy;
                }
            }

            draw() {
                ctx.beginPath();
                ctx.moveTo(this.coordinates[this.coordinates.length - 1][0], this.coordinates[this.coordinates.length - 1][1]);
                ctx.lineTo(this.x, this.y);
                ctx.strokeStyle = `hsl(${Math.random() * 360}, 100%, ${this.brightness}%)`;
                ctx.stroke();

                ctx.beginPath();
                ctx.arc(this.targetX, this.targetY, this.targetRadius, 0, Math.PI * 2);
                ctx.stroke();
            }
        }

        class Particle {
            constructor(x, y, hue) {
                this.x = x;
                this.y = y;
                this.coordinates = [];
                this.coordinateCount = 5;
                while (this.coordinateCount--) {
                    this.coordinates.push([this.x, this.y]);
                }
                this.angle = Math.random() * Math.PI * 2;
                this.speed = Math.random() * 10 + 1;
                this.friction = 0.95;
                this.gravity = 1;
                this.hue = hue;
                this.brightness = Math.random() * 80 + 20;
                this.alpha = 1;
                this.decay = Math.random() * 0.03 + 0.01;
            }

            update(index) {
                this.coordinates.pop();
                this.coordinates.unshift([this.x, this.y]);

                this.speed *= this.friction;
                this.x += Math.cos(this.angle) * this.speed;
                this.y += Math.sin(this.angle) * this.speed + this.gravity;
                this.alpha -= this.decay;

                if (this.alpha <= this.decay) {
                    particles.splice(index, 1);
                }
            }

            draw() {
                ctx.beginPath();
                ctx.moveTo(this.coordinates[this.coordinates.length - 1][0], this.coordinates[this.coordinates.length - 1][1]);
                ctx.lineTo(this.x, this.y);
                ctx.strokeStyle = `hsla(${this.hue}, 100%, ${this.brightness}%, ${this.alpha})`;
                ctx.stroke();
            }
        }

        function createTextParticles(x, y, text) {
            ctx.font = "30px Arial";
            ctx.fillStyle = "white";
            ctx.fillText(text, x, y);
            const textWidth = ctx.measureText(text).width;
            const imageData = ctx.getImageData(x, y - 30, textWidth, 30);
            ctx.clearRect(x, y - 30, textWidth, 30);

            for (let i = 0; i < imageData.data.length; i += 4) {
                if (imageData.data[i + 3] > 128) {
                    const particleX = x + (i / 4) % textWidth;
                    const particleY = y - 30 + Math.floor((i / 4) / textWidth);
                    particles.push(new Particle(particleX, particleY, Math.random() * 360));
                }
            }
        }

        function loop() {
            requestAnimationFrame(loop);

            ctx.globalCompositeOperation = 'destination-out';
            ctx.fillStyle = 'rgba(0, 0, 0, 0.5)';
            ctx.fillRect(0, 0, canvas.width, canvas.height);
            ctx.globalCompositeOperation = 'lighter';

            let i = fireworks.length;
            while (i--) {
                fireworks[i].draw();
                fireworks[i].update(i);
            }

            let j = particles.length;
            while (j--) {
                particles[j].draw();
                particles[j].update(j);
            }

            if (isFireworksActive && Math.random() < 0.05) { // 调整发射频率
                const startX = Math.random() * canvas.width; // 随机化发射点
                const startY = canvas.height;
                const targetX = Math.random() * canvas.width;
                const targetY = Math.random() * canvas.height / 2; // 确保目标位置在页面上半部分
                fireworks.push(new Firework(startX, startY, targetX, targetY));
                fireworkCount++; // 增加烟花数量
            }
        }

        window.onload = loop;
        window.onresize = () => {
            canvas.width = window.innerWidth;
            canvas.height = window.innerHeight;
        };
    </script>
</body>
</html>
