欢欢新年快乐！
<html lang="zh-cn">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>浪漫烟花效果</title>
    <style>
        @import url('https://fonts.googleapis.com/css2?family=Zhi+Mang+Xing&display=swap'); /* 引入行楷字体 */
        
        body {
            margin: 0;
            overflow: hidden;
            background-color: #000; /* 背景为黑色 */
            display: flex;
            justify-content: center;
            align-items: center;
            height: 100vh;
            position: relative;
            flex-direction: column;
        }
        canvas {
            display: block;
            width: 100%;
            height: 100%;
        }
        #controls {
            position: absolute;
            bottom: 20px;
            display: flex;
            gap: 10px;
        }
        .controlButton {
            padding: 8px 16px;
            background-color: #ff4081;
            color: white;
            border: none;
            border-radius: 20px;
            box-shadow: 0 2px 4px rgba(0, 0, 0, 0.1);
            cursor: pointer;
            z-index: 10;
            font-size: 14px;
            font-family: 'Zhi Mang Xing', cursive;
            transition: background-color 0.3s, transform 0.3s;
        }
        .controlButton:hover {
            background-color: #ff79a6;
            transform: scale(1.1);
        }
        #backgroundText, #newYearText {
            position: absolute;
            left: 50%;
            transform: translate(-50%, -50%);
            font-size: 10vw; /* 使用相对单位以适应不同屏幕 */
            font-family: 'Zhi Mang Xing', cursive; /* 使用行楷字体 */
            background: linear-gradient(45deg, #ff0000, #ff7f00, #ffff00, #00ff00, #0000ff, #4b0082, #8b00ff);
            -webkit-background-clip: text;
            color: transparent;
            z-index: 1;
            text-align: center;
            opacity: 0;
            transition: opacity 2s ease-in-out; /* 添加淡入淡出效果 */
        }
        #backgroundText {
            top: 40%;
        }
        #newYearText {
            top: 60%;
        }
    </style>
</head>
<body>
    <div id="backgroundText">欢欢</div>
    <div id="newYearText">新年快乐</div>
    <canvas id="fireworksCanvas"></canvas>
    <audio id="fireworkSound" src="firework.mp3"></audio>
    <div id="controls">
        <button class="controlButton" id="toggleButton">开始烟花</button>
        <button class="controlButton" id="type1Button">普通烟花</button>
        <button class="controlButton" id="type2Button">大烟花</button>
        <button class="controlButton" id="type3Button">小烟花</button>
        <button class="controlButton" id="type4Button">特殊烟花</button>
    </div>
    <script>
        const canvas = document.getElementById('fireworksCanvas');
        const ctx = canvas.getContext('2d');
        const fireworkSound = document.getElementById('fireworkSound');
        const toggleButton = document.getElementById('toggleButton');
        const type1Button = document.getElementById('type1Button');
        const type2Button = document.getElementById('type2Button');
        const type3Button = document.getElementById('type3Button');
        const type4Button = document.getElementById('type4Button');
        const backgroundText = document.getElementById('backgroundText');
        const newYearText = document.getElementById('newYearText');
        let fireworks = [];
        let particles = [];
        const colors = ['#FF1461', '#18FF92', '#5A87FF', '#FBF38C'];
        let fireworkCount = 0;
        let isFireworksActive = false;
        let hue = 0;
        let fireworkType = 0; // 当前烟花类型

        canvas.width = window.innerWidth;
        canvas.height = window.innerHeight;

        toggleButton.addEventListener('click', () => {
            isFireworksActive = !isFireworksActive;
            toggleButton.textContent = isFireworksActive ? '停止烟花' : '开始烟花';
            if (isFireworksActive) {
                setTimeout(() => {
                    backgroundText.style.opacity = 1;
                }, 3000); // 3秒后显示“欢欢”
                setTimeout(() => {
                    newYearText.style.opacity = 1;
                }, 9000); // 9秒后显示“新年快乐”
            } else {
                backgroundText.style.opacity = 0;
                newYearText.style.opacity = 0;
            }
        });

        type1Button.addEventListener('click', () => fireworkType = 0);
        type2Button.addEventListener('click', () => fireworkType = 1);
        type3Button.addEventListener('click', () => fireworkType = 2);
        type4Button.addEventListener('click', () => fireworkType = 3);

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
                    createParticles(this.targetX, this.targetY, fireworkType);
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

        function createParticles(x, y, type) {
            let particleCount;
            switch (type) {
                case 0: // 普通烟花
                    particleCount = 100;
                    break;
                case 1: // 大烟花
                    particleCount = 200;
                    break;
                case 2: // 小烟花
                    particleCount = 50;
                    break;
                case 3: // 特殊烟花
                    particleCount = 150;
                    break;
                default:
                    particleCount = 100;
            }

            while (particleCount--) {
                particles.push(new Particle(x, y, Math.random() * 360));
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

            if (isFireworksActive && Math.random() < 0.1) { // 增加发射频率
                const startX = Math.random() * canvas.width; // 随机化发射点
                const startY = canvas.height;
                const targetX = Math.random() * canvas.width;
                const targetY = Math.random() * canvas.height / 2; // 确保目标位置在页面上半部分
                fireworks.push(new Firework(startX, startY, targetX, targetY));
                fireworkCount++; // 增加烟花数量
            }

            // 更新背景文字颜色，颜色变化速度减缓
            if (isFireworksActive) {
                hue += 0.5; // 调整颜色变化速度
                const gradient = `linear-gradient(45deg, hsl(${hue}, 100%, 50%), hsl(${(hue + 60) % 360}, 100%, 50%), hsl(${(hue + 120) % 360}, 100%, 50%), hsl(${(hue + 180) % 360}, 100%, 50%), hsl(${(hue + 240) % 360}, 100%, 50%), hsl(${(hue + 300) % 360}, 100%, 50%))`;
                backgroundText.style.background = gradient;
                backgroundText.style.webkitBackgroundClip = 'text';
                backgroundText.style.color = 'transparent';
                newYearText.style.background = gradient;
                newYearText.style.webkitBackgroundClip = 'text';
                newYearText.style.color = 'transparent';
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
