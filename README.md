<!DOCTYPE html>
<html lang="zh-CN">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no">
    <title>Cyber Drift: Armed</title>
    <style>
        @import url('https://fonts.googleapis.com/css2?family=Orbitron:wght@400;700;900&display=swap');

        :root {
            --neon-blue: #00f3ff;
            --neon-pink: #bc13fe;
            --neon-green: #0aff0a;
            --neon-red: #ff003c;
            --neon-yellow: #fcee0a;
            --bg-color: #050510;
        }

        body {
            margin: 0;
            padding: 0;
            overflow: hidden;
            background-color: var(--bg-color);
            font-family: 'Orbitron', sans-serif;
            color: white;
            user-select: none;
            touch-action: none;
        }

        #game-container {
            position: relative;
            width: 100vw;
            height: 100vh;
            display: flex;
            justify-content: center;
            align-items: center;
        }

        canvas {
            box-shadow: 0 0 50px rgba(0, 243, 255, 0.1);
            border: 1px solid rgba(0, 243, 255, 0.3);
        }

        /* UI 层 */
        .ui-layer {
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            pointer-events: none;
            display: flex;
            flex-direction: column;
            justify-content: space-between;
            padding: 20px;
            box-sizing: border-box;
        }

        /* 顶部 HUD */
        .hud-top {
            display: flex;
            justify-content: space-between;
            align-items: flex-start;
        }

        .score-box {
            text-shadow: 0 0 10px var(--neon-blue);
        }
        
        .score-label {
            font-size: 14px;
            color: #888;
        }

        .score-value {
            font-size: 28px;
            font-weight: 700;
            color: var(--neon-blue);
        }

        /* 状态栏 (血条 & 护盾) */
        .status-bar {
            display: flex;
            flex-direction: column;
            gap: 8px;
            width: 200px;
        }

        .bar-container {
            width: 100%;
            height: 12px;
            background: rgba(255, 255, 255, 0.1);
            border: 1px solid rgba(255, 255, 255, 0.2);
            position: relative;
            transform: skewX(-20deg);
            overflow: hidden;
        }

        .bar-fill {
            height: 100%;
            width: 100%;
            transition: width 0.2s ease-out;
            box-shadow: 0 0 10px currentColor;
        }

        .hp-bar .bar-fill { background: var(--neon-red); color: var(--neon-red); }
        .shield-bar .bar-fill { background: var(--neon-blue); color: var(--neon-blue); }
        .weapon-bar .bar-fill { background: var(--neon-yellow); color: var(--neon-yellow); }

        .shield-icon {
            position: absolute;
            right: -25px;
            top: -5px;
            font-size: 20px;
            color: var(--neon-blue);
            opacity: 0;
            transition: opacity 0.3s;
        }
        
        .shield-active .shield-icon { opacity: 1; }

        /* 连击显示 */
        .combo-box {
            position: absolute;
            top: 80px;
            left: 20px;
            opacity: 0;
            transform: scale(0.8);
            transition: all 0.1s;
        }

        .combo-box.active {
            opacity: 1;
            transform: scale(1.2);
        }

        .combo-count {
            font-size: 40px;
            font-weight: 900;
            color: var(--neon-yellow);
            font-style: italic;
            text-shadow: 2px 2px 0px var(--neon-pink);
        }

        .combo-label {
            font-size: 12px;
            color: var(--neon-yellow);
            letter-spacing: 2px;
        }

        /* 菜单屏幕 */
        .screen {
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            background: rgba(5, 5, 16, 0.9);
            backdrop-filter: blur(8px);
            display: flex;
            flex-direction: column;
            justify-content: center;
            align-items: center;
            z-index: 10;
            transition: opacity 0.3s;
        }

        .hidden {
            opacity: 0;
            pointer-events: none;
        }

        h1 {
            font-size: 60px;
            margin: 0 0 20px 0;
            background: linear-gradient(90deg, var(--neon-blue), var(--neon-pink));
            -webkit-background-clip: text;
            -webkit-text-fill-color: transparent;
            text-shadow: 0 0 30px rgba(188, 19, 254, 0.5);
            letter-spacing: 5px;
            text-transform: uppercase;
            text-align: center;
        }

        .instructions {
            display: flex;
            gap: 20px;
            margin-bottom: 40px;
        }

        .instruction-item {
            display: flex;
            flex-direction: column;
            align-items: center;
            font-size: 14px;
            color: #aaa;
        }

        .key-icon {
            width: 40px;
            height: 40px;
            border: 1px solid #555;
            border-radius: 5px;
            display: flex;
            justify-content: center;
            align-items: center;
            margin-bottom: 8px;
            font-size: 20px;
            color: white;
            box-shadow: 0 0 10px rgba(255,255,255,0.1);
        }

        button {
            background: transparent;
            color: var(--neon-blue);
            font-family: 'Orbitron', sans-serif;
            font-size: 24px;
            padding: 15px 50px;
            border: 2px solid var(--neon-blue);
            cursor: pointer;
            transition: all 0.2s;
            text-transform: uppercase;
            letter-spacing: 2px;
            position: relative;
            overflow: hidden;
            clip-path: polygon(10% 0, 100% 0, 100% 70%, 90% 100%, 0 100%, 0 30%);
        }

        button:hover {
            background: var(--neon-blue);
            color: var(--bg-color);
            box-shadow: 0 0 30px rgba(0, 243, 255, 0.6);
            transform: translateY(-2px);
        }

        button:active {
            transform: translateY(1px);
        }

        /* 受伤红屏闪烁 */
        #damageOverlay {
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            background: radial-gradient(circle, transparent 50%, rgba(255, 0, 60, 0.6) 100%);
            opacity: 0;
            pointer-events: none;
            transition: opacity 0.1s;
            z-index: 5;
        }

        /* 音频开关 */
        .audio-toggle {
            position: absolute;
            bottom: 20px;
            right: 20px;
            width: 40px;
            height: 40px;
            background: rgba(0,0,0,0.5);
            border: 1px solid var(--neon-blue);
            border-radius: 50%;
            display: flex;
            justify-content: center;
            align-items: center;
            cursor: pointer;
            pointer-events: auto;
            z-index: 20;
            color: var(--neon-blue);
            font-size: 20px;
            transition: all 0.2s;
        }

        .audio-toggle:hover {
            background: var(--neon-blue);
            color: var(--bg-color);
        }

        /* 故障特效动画 */
        .glitch {
            position: relative;
        }
        
        .glitch::before, .glitch::after {
            content: attr(data-text);
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            background: var(--bg-color);
        }

        .glitch::before {
            left: 2px;
            text-shadow: -1px 0 red;
            clip: rect(24px, 550px, 90px, 0);
            animation: glitch-anim-2 3s infinite linear alternate-reverse;
        }

        .glitch::after {
            left: -2px;
            text-shadow: -1px 0 blue;
            clip: rect(85px, 550px, 140px, 0);
            animation: glitch-anim 2.5s infinite linear alternate-reverse;
        }

        @keyframes glitch-anim {
            0% { clip: rect(10px, 9999px, 30px, 0); }
            20% { clip: rect(80px, 9999px, 100px, 0); }
            40% { clip: rect(10px, 9999px, 50px, 0); }
            60% { clip: rect(40px, 9999px, 80px, 0); }
            80% { clip: rect(20px, 9999px, 60px, 0); }
            100% { clip: rect(60px, 9999px, 90px, 0); }
        }

        @keyframes glitch-anim-2 {
            0% { clip: rect(60px, 9999px, 90px, 0); }
            20% { clip: rect(10px, 9999px, 40px, 0); }
            40% { clip: rect(80px, 9999px, 100px, 0); }
            60% { clip: rect(30px, 9999px, 60px, 0); }
            80% { clip: rect(50px, 9999px, 70px, 0); }
            100% { clip: rect(90px, 9999px, 90px, 0); }
        }
    </style>
</head>
<body>

    <div id="game-container">
        <canvas id="gameCanvas"></canvas>
        <div id="damageOverlay"></div>
        
        <div class="ui-layer">
            <div class="hud-top">
                <div class="status-bar">
                    <div class="bar-container hp-bar">
                        <div class="bar-fill" id="hpBar"></div>
                    </div>
                    <div class="bar-container shield-bar" id="shieldBarContainer">
                        <div class="bar-fill" id="shieldBar"></div>
                        <div class="shield-icon">🛡️</div>
                    </div>
                    <div class="bar-container weapon-bar">
                        <div class="bar-fill" id="weaponBar"></div>
                    </div>
                </div>

                <div class="score-box">
                    <div class="score-label">SCORE</div>
                    <div class="score-value" id="scoreDisplay">0</div>
                    <div class="score-label" style="margin-top: 5px;">BEST: <span id="highScoreDisplay">0</span></div>
                </div>
            </div>

            <div class="combo-box" id="comboBox">
                <div class="combo-count" id="comboCount">x0</div>
                <div class="combo-label">COMBO</div>
            </div>
        </div>

        <!-- 开始屏幕 -->
        <div id="startScreen" class="screen">
            <h1 class="glitch" data-text="CYBER DRIFT">CYBER DRIFT</h1>
            <div class="instructions">
                <div class="instruction-item">
                    <div class="key-icon">↔</div>
                    <span>移动</span>
                </div>
                <div class="instruction-item">
                    <div class="key-icon" style="color:var(--neon-yellow)">⚡</div>
                    <span>武器</span>
                </div>
                <div class="instruction-item">
                    <div class="key-icon" style="color:var(--neon-blue)">▲</div>
                    <span>护盾</span>
                </div>
                <div class="instruction-item">
                    <div class="key-icon" style="color:var(--neon-red)">♥</div>
                    <span>回血</span>
                </div>
            </div>
            <button id="startBtn">初始化系统</button>
        </div>

        <!-- 游戏结束屏幕 -->
        <div id="gameOverScreen" class="screen hidden">
            <h1 class="glitch" data-text="SYSTEM FAILURE">SYSTEM FAILURE</h1>
            <p style="font-size: 24px; color: #ccc;">最终得分: <span id="finalScore" style="color:var(--neon-blue); font-weight: bold;">0</span></p>
            <button id="restartBtn">重启系统</button>
        </div>

        <!-- 音频开关 -->
        <div class="audio-toggle" id="audioToggle">♪</div>
    </div>

    <script>
        /**
         * CYBER DRIFT: ARMED - AUDIO ENGINE
         */
        
        const AudioContext = window.AudioContext || window.webkitAudioContext;
        let audioCtx;
        let bgmOscillators = [];
        let bgmGainNode;
        let isMuted = false;
        let isAudioInitialized = false;

        function initAudio() {
            if (isAudioInitialized) return;
            
            try {
                audioCtx = new AudioContext();
                bgmGainNode = audioCtx.createGain();
                bgmGainNode.connect(audioCtx.destination);
                bgmGainNode.gain.value = 0.08; 
                isAudioInitialized = true;
            } catch (e) {
                console.warn('Web Audio API not supported');
            }
        }

        function playSound(type) {
            if (!isAudioInitialized || isMuted) return;
            if (audioCtx.state === 'suspended') audioCtx.resume();

            const osc = audioCtx.createOscillator();
            const gainNode = audioCtx.createGain();
            
            osc.connect(gainNode);
            gainNode.connect(audioCtx.destination);

            const now = audioCtx.currentTime;

            switch (type) {
                case 'move': 
                    osc.type = 'sine';
                    osc.frequency.setValueAtTime(300, now);
                    osc.frequency.exponentialRampToValueAtTime(500, now + 0.1);
                    gainNode.gain.setValueAtTime(0.05, now);
                    gainNode.gain.exponentialRampToValueAtTime(0.01, now + 0.1);
                    osc.start(now);
                    osc.stop(now + 0.1);
                    break;
                
                case 'shoot':
                    osc.type = 'square';
                    osc.frequency.setValueAtTime(800, now);
                    osc.frequency.exponentialRampToValueAtTime(200, now + 0.1);
                    gainNode.gain.setValueAtTime(0.05, now);
                    gainNode.gain.exponentialRampToValueAtTime(0.01, now + 0.1);
                    osc.start(now);
                    osc.stop(now + 0.1);
                    break;

                case 'collect': 
                    osc.type = 'triangle';
                    osc.frequency.setValueAtTime(800, now);
                    osc.frequency.setValueAtTime(1200, now + 0.05);
                    gainNode.gain.setValueAtTime(0.1, now);
                    gainNode.gain.exponentialRampToValueAtTime(0.01, now + 0.2);
                    osc.start(now);
                    osc.stop(now + 0.2);
                    
                    const osc2 = audioCtx.createOscillator();
                    const gain2 = audioCtx.createGain();
                    osc2.connect(gain2);
                    gain2.connect(audioCtx.destination);
                    osc2.type = 'sine';
                    osc2.frequency.setValueAtTime(1200, now + 0.05);
                    osc2.frequency.exponentialRampToValueAtTime(1800, now + 0.15);
                    gain2.gain.setValueAtTime(0.1, now + 0.05);
                    gain2.gain.exponentialRampToValueAtTime(0.01, now + 0.2);
                    osc2.start(now + 0.05);
                    osc2.stop(now + 0.2);
                    break;

                case 'hit': 
                    osc.type = 'sawtooth';
                    osc.frequency.setValueAtTime(100, now);
                    osc.frequency.exponentialRampToValueAtTime(10, now + 0.3);
                    gainNode.gain.setValueAtTime(0.2, now);
                    gainNode.gain.exponentialRampToValueAtTime(0.01, now + 0.3);
                    osc.start(now);
                    osc.stop(now + 0.3);
                    break;

                case 'gameover': 
                    osc.type = 'sawtooth';
                    osc.frequency.setValueAtTime(200, now);
                    osc.frequency.exponentialRampToValueAtTime(10, now + 1.5);
                    gainNode.gain.setValueAtTime(0.3, now);
                    gainNode.gain.exponentialRampToValueAtTime(0.01, now + 1.5);
                    osc.start(now);
                    osc.stop(now + 1.5);
                    break;
                    
                case 'powerup': 
                    osc.type = 'square';
                    osc.frequency.setValueAtTime(400, now);
                    osc.frequency.linearRampToValueAtTime(600, now + 0.5);
                    gainNode.gain.setValueAtTime(0.05, now);
                    gainNode.gain.linearRampToValueAtTime(0, now + 0.5);
                    osc.start(now);
                    osc.stop(now + 0.5);
                    break;
                
                case 'shield': 
                    osc.type = 'sine';
                    osc.frequency.setValueAtTime(600, now);
                    osc.frequency.linearRampToValueAtTime(1200, now + 0.3);
                    gainNode.gain.setValueAtTime(0.1, now);
                    gainNode.gain.linearRampToValueAtTime(0, now + 0.3);
                    osc.start(now);
                    osc.stop(now + 0.3);
                    break;

                case 'heal':
                    osc.type = 'triangle';
                    osc.frequency.setValueAtTime(400, now);
                    osc.frequency.linearRampToValueAtTime(800, now + 0.2);
                    osc.frequency.linearRampToValueAtTime(600, now + 0.4);
                    gainNode.gain.setValueAtTime(0.1, now);
                    gainNode.gain.linearRampToValueAtTime(0, now + 0.4);
                    osc.start(now);
                    osc.stop(now + 0.4);
                    break;

                case 'combo':
                    const pitch = 400 + (game.combo * 50);
                    osc.type = 'square';
                    osc.frequency.setValueAtTime(pitch, now);
                    osc.frequency.exponentialRampToValueAtTime(pitch * 1.5, now + 0.1);
                    gainNode.gain.setValueAtTime(0.05, now);
                    gainNode.gain.exponentialRampToValueAtTime(0.01, now + 0.1);
                    osc.start(now);
                    osc.stop(now + 0.1);
                    break;
            }
        }

        let bgmInterval;
        function startBGM() {
            if (!isAudioInitialized || isMuted) return;
            stopBGM();

            let noteIndex = 0;
            const notes = [110, 110, 220, 110, 130.81, 110, 164.81, 98]; // A2, A3, C3, E3, G2
            
            bgmInterval = setInterval(() => {
                if (isMuted || !game.active) return;
                
                const osc = audioCtx.createOscillator();
                const gain = audioCtx.createGain();
                
                osc.connect(gain);
                gain.connect(bgmGainNode);
                
                osc.type = 'sawtooth';
                osc.frequency.value = notes[noteIndex];
                
                const now = audioCtx.currentTime;
                gain.gain.setValueAtTime(0.1, now);
                gain.gain.exponentialRampToValueAtTime(0.001, now + 0.4);
                
                osc.start(now);
                osc.stop(now + 0.4);
                
                noteIndex = (noteIndex + 1) % notes.length;
            }, 250);
        }

        function stopBGM() {
            if (bgmInterval) clearInterval(bgmInterval);
        }

        const audioToggle = document.getElementById('audioToggle');
        audioToggle.addEventListener('click', () => {
            isMuted = !isMuted;
            audioToggle.style.opacity = isMuted ? '0.3' : '1';
            audioToggle.innerText = isMuted ? '✕' : '♪';
            
            if (isMuted) {
                stopBGM();
            } else if (game.active) {
                startBGM();
            }
        });

        /**
         * CYBER DRIFT: ARMED - GAME ENGINE
         */
        
        const canvas = document.getElementById('gameCanvas');
        const ctx = canvas.getContext('2d');
        
        // UI 元素
        const scoreEl = document.getElementById('scoreDisplay');
        const highScoreEl = document.getElementById('highScoreDisplay');
        const finalScoreEl = document.getElementById('finalScore');
        const hpBar = document.getElementById('hpBar');
        const shieldBar = document.getElementById('shieldBar');
        const shieldBarContainer = document.getElementById('shieldBarContainer');
        const weaponBar = document.getElementById('weaponBar');
        const comboBox = document.getElementById('comboBox');
        const comboCount = document.getElementById('comboCount');
        const damageOverlay = document.getElementById('damageOverlay');
        
        const startScreen = document.getElementById('startScreen');
        const gameOverScreen = document.getElementById('gameOverScreen');
        const startBtn = document.getElementById('startBtn');
        const restartBtn = document.getElementById('restartBtn');

        // 游戏状态
        let game = {
            active: false,
            width: 0,
            height: 0,
            score: 0,
            highScore: localStorage.getItem('cyberDriftHighScore') || 0,
            speed: 2.0, 
            frame: 0,
            shake: 0,
            combo: 0,
            comboTimer: 0
        };

        highScoreEl.innerText = game.highScore;

        function resize() {
            game.width = window.innerWidth;
            game.height = window.innerHeight;
            canvas.width = game.width;
            canvas.height = game.height;
        }
        window.addEventListener('resize', resize);
        resize();

        // 输入控制
        const input = {
            x: 0,
            y: 0
        };

        window.addEventListener('keydown', e => {
            if (!game.active) return;
            switch(e.key) {
                case 'ArrowLeft': 
                    if (input.x !== -1) playSound('move');
                    input.x = -1; 
                    break;
                case 'ArrowRight': 
                    if (input.x !== 1) playSound('move');
                    input.x = 1; 
                    break;
                case 'ArrowUp': 
                    if (input.y !== -1) playSound('move');
                    input.y = -1; 
                    break;
                case 'ArrowDown': 
                    if (input.y !== 1) playSound('move');
                    input.y = 1; 
                    break;
            }
        });

        window.addEventListener('keyup', e => {
            switch(e.key) {
                case 'ArrowLeft': 
                case 'ArrowRight': input.x = 0; break;
                case 'ArrowUp': 
                case 'ArrowDown': input.y = 0; break;
            }
        });

        // 触摸控制
        let touchStartX = 0;
        let touchStartY = 0;
        
        window.addEventListener('touchstart', e => {
            touchStartX = e.touches[0].clientX;
            touchStartY = e.touches[0].clientY;
        }, {passive: false});

        window.addEventListener('touchmove', e => {
            if (!game.active) return;
            e.preventDefault();
            const touchX = e.touches[0].clientX;
            const touchY = e.touches[0].clientY;
            
            const diffX = touchX - touchStartX;
            const diffY = touchY - touchStartY;
            
            if (Math.abs(diffX) > 10 || Math.abs(diffY) > 10) {
                if (Math.abs(diffX) > Math.abs(diffY)) {
                    if (input.x !== (diffX > 10 ? 1 : -1)) playSound('move');
                    input.x = diffX > 10 ? 1 : -1;
                    input.y = 0;
                } else {
                    if (input.y !== (diffY > 10 ? 1 : -1)) playSound('move');
                    input.y = diffY > 10 ? 1 : -1;
                    input.x = 0;
                }
            }
        }, {passive: false});

        window.addEventListener('touchend', () => {
            input.x = 0;
            input.y = 0;
        });

        // 玩家类
        class Player {
            constructor() {
                this.size = 30;
                this.x = game.width / 2;
                this.y = game.height - 100;
                this.speed = 7;
                this.color = '#00f3ff';
                this.trail = [];
                this.engineParticles = [];
                
                // 状态属性
                this.maxHp = 100;
                this.hp = 100;
                this.hasShield = false;
                this.invincible = false;
                this.invincibleTimer = 0;
                
                // 武器属性
                this.weaponLevel = 1; // 1: 单发, 2: 双发, 3: 散射
                this.shootTimer = 0;
                this.shootInterval = 15; // 射击间隔（帧）
            }

            update() {
                // 移动
                this.x += input.x * this.speed;
                this.y += input.y * this.speed;

                // 边界限制
                if (this.x < this.size) this.x = this.size;
                if (this.x > game.width - this.size) this.x = game.width - this.size;
                if (this.y < this.size) this.y = this.size;
                if (this.y > game.height - this.size) this.y = game.height - this.size;

                // 拖尾效果
                this.trail.push({x: this.x, y: this.y, alpha: 1.0});
                if (this.trail.length > 10) this.trail.shift();
                this.trail.forEach(t => t.alpha -= 0.1);

                // 引擎粒子效果
                if (game.frame % 2 === 0) {
                    this.engineParticles.push({
                        x: this.x + (Math.random() - 0.5) * 10,
                        y: this.y + this.size/2,
                        vx: (Math.random() - 0.5) * 2,
                        vy: Math.random() * 3 + 2,
                        life: 1.0,
                        size: Math.random() * 3 + 1
                    });
                }

                // 更新引擎粒子
                for (let i = this.engineParticles.length - 1; i >= 0; i--) {
                    const p = this.engineParticles[i];
                    p.x += p.vx;
                    p.y += p.vy;
                    p.life -= 0.05;
                    if (p.life <= 0) this.engineParticles.splice(i, 1);
                }

                // 自动射击
                if (this.shootTimer <= 0) {
                    this.shoot();
                    this.shootTimer = this.shootInterval;
                } else {
                    this.shootTimer--;
                }

                // 无敌时间处理
                if (this.invincible) {
                    this.invincibleTimer--;
                    if (this.invincibleTimer <= 0) this.invincible = false;
                }
            }

            shoot() {
                playSound('shoot');
                
                // 根据武器等级发射不同类型的子弹
                if (this.weaponLevel === 1) {
                    // 单发
                    bullets.push(new Bullet(this.x, this.y - this.size/2, 0, -10));
                } else if (this.weaponLevel === 2) {
                    // 双发
                    bullets.push(new Bullet(this.x - 10, this.y - this.size/2, 0, -10));
                    bullets.push(new Bullet(this.x + 10, this.y - this.size/2, 0, -10));
                } else {
                    // 散射 (3发)
                    bullets.push(new Bullet(this.x, this.y - this.size/2, 0, -10));
                    bullets.push(new Bullet(this.x - 10, this.y - this.size/2, -2, -9));
                    bullets.push(new Bullet(this.x + 10, this.y - this.size/2, 2, -9));
                }
            }

            upgradeWeapon() {
                if (this.weaponLevel < 3) {
                    this.weaponLevel++;
                    playSound('powerup');
                    createExplosion(this.x, this.y, '#fcee0a', 20);
                    updateUI();
                } else {
                    // 满级时加分
                    addScore(200);
                }
            }

            draw() {
                // 绘制引擎粒子
                this.engineParticles.forEach(p => {
                    ctx.fillStyle = `rgba(0, 243, 255, ${p.life})`;
                    ctx.fillRect(p.x, p.y, p.size, p.size);
                });

                // 绘制拖尾
                this.trail.forEach(t => {
                    ctx.fillStyle = `rgba(0, 243, 255, ${t.alpha * 0.5})`;
                    ctx.fillRect(t.x - this.size/2, t.y - this.size/2, this.size, this.size);
                });

                // 绘制护盾
                if (this.hasShield) {
                    ctx.beginPath();
                    ctx.arc(this.x, this.y, this.size * 0.8, 0, Math.PI * 2);
                    ctx.strokeStyle = `rgba(0, 243, 255, ${0.5 + Math.sin(game.frame * 0.1) * 0.2})`;
                    ctx.lineWidth = 2;
                    ctx.stroke();
                    
                    ctx.beginPath();
                    ctx.arc(this.x, this.y, this.size * 0.8, 0, Math.PI * 2);
                    ctx.fillStyle = `rgba(0, 243, 255, 0.1)`;
                    ctx.fill();
                }

                // 绘制玩家
                ctx.shadowBlur = 20;
                ctx.shadowColor = this.invincible ? '#ffffff' : this.color;
                ctx.fillStyle = this.invincible ? '#ffffff' : this.color;
                
                ctx.save();
                ctx.translate(this.x, this.y);
                ctx.rotate(game.frame * 0.1);
                ctx.fillRect(-this.size/2, -this.size/2, this.size, this.size);
                ctx.restore();
                
                ctx.shadowBlur = 0;
            }

            takeDamage(amount) {
                if (this.invincible) return false;

                if (this.hasShield) {
                    this.hasShield = false;
                    playSound('shield');
                    createExplosion(this.x, this.y, '#00f3ff', 10);
                    updateUI();
                    return false; 
                }

                this.hp -= amount;
                game.shake = 15;
                playSound('hit');
                
                // 受伤红屏闪烁
                damageOverlay.style.opacity = '1';
                setTimeout(() => { damageOverlay.style.opacity = '0'; }, 100);

                if (this.hp <= 0) {
                    this.hp = 0;
                    return true; 
                }

                // 受伤后短暂无敌
                this.invincible = true;
                this.invincibleTimer = 90; 
                updateUI();
                return false;
            }

            heal(amount) {
                this.hp = Math.min(this.hp + amount, this.maxHp);
                playSound('heal');
                createExplosion(this.x, this.y, '#ff003c', 10);
                updateUI();
            }
        }

        // 子弹类
        class Bullet {
            constructor(x, y, vx, vy) {
                this.x = x;
                this.y = y;
                this.vx = vx;
                this.vy = vy;
                this.size = 4;
                this.color = '#fcee0a';
                this.active = true;
            }

            update() {
                this.x += this.vx;
                this.y += this.vy;

                // 超出屏幕移除
                if (this.y < -10 || this.x < -10 || this.x > game.width + 10) {
                    this.active = false;
                }
            }

            draw() {
                ctx.shadowBlur = 10;
                ctx.shadowColor = this.color;
                ctx.fillStyle = this.color;
                
                ctx.beginPath();
                ctx.arc(this.x, this.y, this.size, 0, Math.PI * 2);
                ctx.fill();
                
                ctx.shadowBlur = 0;
            }
        }

        // 敌人类
        class Enemy {
            constructor() {
                this.size = 25;
                this.x = Math.random() * (game.width - this.size * 2) + this.size;
                this.y = -50;
                this.speed = Math.random() * 1.5 + game.speed;
                this.angle = 0;
                this.isHoming = Math.random() > 0.9;
                this.color = '#bc13fe';
                
                // 敌人血量：普通怪1血，精英怪3血
                this.isElite = Math.random() > 0.8;
                this.hp = this.isElite ? 3 : 1;
                this.maxHp = this.hp;
            }

            update(player) {
                if (this.isHoming) {
                    if (this.x < player.x) this.x += 0.8;
                    else this.x -= 0.8;
                }
                
                this.y += this.speed;
                this.angle += 0.05;
            }

            draw() {
                ctx.save();
                ctx.translate(this.x, this.y);
                ctx.rotate(this.angle);
                
                ctx.shadowBlur = 15;
                ctx.shadowColor = this.color;
                ctx.strokeStyle = this.color;
                ctx.lineWidth = this.isElite ? 5 : 3;
                
                // 绘制菱形
                ctx.beginPath();
                ctx.moveTo(0, -this.size/2);
                ctx.lineTo(this.size/2, 0);
                ctx.lineTo(0, this.size/2);
                ctx.lineTo(-this.size/2, 0);
                ctx.closePath();
                ctx.stroke();
                
                // 精英怪内部填充
                if (this.isElite) {
                    ctx.fillStyle = 'rgba(188, 19, 254, 0.3)';
                    ctx.fill();
                }
                
                ctx.restore();
            }

            takeDamage(amount) {
                this.hp -= amount;
                if (this.hp <= 0) {
                    return true; // 死亡
                }
                return false; // 存活
            }
        }

        const PowerUpType = {
            INVINCIBLE: 'invincible',
            SHIELD: 'shield',
            HEALTH: 'health',
            WEAPON: 'weapon' // 新增武器升级道具
        };

        class PowerUp {
            constructor() {
                this.size = 20;
                this.x = Math.random() * (game.width - this.size * 2) + this.size;
                this.y = -50;
                this.speed = game.speed;
                
                const rand = Math.random();
                if (rand < 0.5) {
                    this.type = PowerUpType.SHIELD;
                    this.color = '#00f3ff'; 
                } else if (rand < 0.85) {
                    this.type = PowerUpType.HEALTH;
                    this.color = '#ff003c'; 
                } else if (rand < 0.95) {
                    this.type = PowerUpType.WEAPON; // 武器升级
                    this.color = '#fcee0a'; 
                } else {
                    this.type = PowerUpType.INVINCIBLE;
                    this.color = '#0aff0a'; 
                }
                
                this.angle = 0;
            }

            update() {
                this.y += this.speed;
                this.angle += 0.05;
            }

            draw() {
                ctx.save();
                ctx.translate(this.x, this.y);
                ctx.rotate(this.angle);
                
                ctx.shadowBlur = 15;
                ctx.shadowColor = this.color;
                ctx.fillStyle = this.color;
                
                const pulse = 1 + Math.sin(game.frame * 0.1) * 0.2;
                ctx.scale(pulse, pulse);
                
                ctx.beginPath();
                if (this.type === PowerUpType.SHIELD) {
                    ctx.moveTo(0, -this.size/2);
                    ctx.lineTo(this.size/2, this.size/2);
                    ctx.lineTo(-this.size/2, this.size/2);
                    ctx.closePath();
                } else if (this.type === PowerUpType.HEALTH) {
                    ctx.arc(0, 0, this.size/2, 0, Math.PI * 2);
                } else if (this.type === PowerUpType.WEAPON) {
                    // 闪电形状
                    ctx.moveTo(0, -this.size/2);
                    ctx.lineTo(this.size/3, -this.size/6);
                    ctx.lineTo(-this.size/3, -this.size/6);
                    ctx.lineTo(0, this.size/2);
                    ctx.lineTo(this.size/3, this.size/6);
                    ctx.lineTo(-this.size/3, this.size/6);
                    ctx.closePath();
                } else {
                    ctx.moveTo(0, -this.size/2);
                    ctx.lineTo(this.size/2, 0);
                    ctx.lineTo(0, this.size/2);
                    ctx.lineTo(-this.size/2, 0);
                    ctx.closePath();
                }
                ctx.fill();
                
                ctx.restore();
            }
        }

        class Particle {
            constructor(x, y, color) {
                this.x = x;
                this.y = y;
                this.color = color;
                this.size = Math.random() * 3 + 1;
                this.speedX = (Math.random() - 0.5) * 6;
                this.speedY = (Math.random() - 0.5) * 6;
                this.life = 1.0;
            }

            update() {
                this.x += this.speedX;
                this.y += this.speedY;
                this.life -= 0.03;
            }

            draw() {
                ctx.globalAlpha = this.life;
                ctx.fillStyle = this.color;
                ctx.fillRect(this.x, this.y, this.size, this.size);
                ctx.globalAlpha = 1.0;
            }
        }

        let player;
        let bullets = [];
        let enemies = [];
        let powerUps = [];
        let particles = [];
        let stars = []; 

        function initStars() {
            stars = [];
            for(let i=0; i<100; i++) {
                stars.push({
                    x: Math.random() * game.width,
                    y: Math.random() * game.height,
                    size: Math.random() * 2,
                    speed: Math.random() * 0.5 + 0.1
                });
            }
        }

        function init() {
            player = new Player();
            bullets = [];
            enemies = [];
            powerUps = [];
            particles = [];
            game.score = 0;
            game.speed = 2.0;
            game.frame = 0;
            game.combo = 0;
            game.comboTimer = 0;
            
            scoreEl.innerText = '0';
            updateUI();
            initStars();
        }

        function checkCollision(rect1, rect2) {
            return (
                rect1.x - rect1.size/2 < rect2.x + rect2.size/2 &&
                rect1.x + rect1.size/2 > rect2.x - rect2.size/2 &&
                rect1.y - rect1.size/2 < rect2.y + rect2.size/2 &&
                rect1.y + rect1.size/2 > rect2.y - rect2.size/2
            );
        }

        function createExplosion(x, y, color, count = 15) {
            for (let i = 0; i < count; i++) {
                particles.push(new Particle(x, y, color));
            }
        }

        function addScore(amount) {
            game.combo++;
            game.comboTimer = 120;
            
            const multiplier = 1 + (game.combo * 0.1);
            const finalScore = Math.floor(amount * multiplier);
            
            game.score += finalScore;
            scoreEl.innerText = game.score;
            
            if (game.combo > 1) {
                playSound('combo');
            }
            
            updateUI();
        }

        function updateUI() {
            const hpPercent = (player.hp / player.maxHp) * 100;
            hpBar.style.width = `${hpPercent}%`;
            
            if (player.hasShield) {
                shieldBar.style.width = '100%';
                shieldBarContainer.classList.add('shield-active');
            } else {
                shieldBar.style.width = '0%';
                shieldBarContainer.classList.remove('shield-active');
            }
            
            // 更新武器等级条
            const weaponPercent = (player.weaponLevel / 3) * 100;
            weaponBar.style.width = `${weaponPercent}%`;
            
            if (game.combo > 1) {
                comboBox.classList.add('active');
                comboCount.innerText = `x${game.combo}`;
            } else {
                comboBox.classList.remove('active');
            }
        }

        function loop() {
            if (!game.active) return;

            ctx.fillStyle = 'rgba(5, 5, 16, 0.3)';
            ctx.fillRect(0, 0, game.width, game.height);

            if (game.shake > 0) {
                const dx = (Math.random() - 0.5) * game.shake;
                const dy = (Math.random() - 0.5) * game.shake;
                ctx.save();
                ctx.translate(dx, dy);
                game.shake *= 0.9;
                if (game.shake < 0.5) game.shake = 0;
            }

            ctx.strokeStyle = 'rgba(0, 243, 255, 0.1)';
            ctx.lineWidth = 1;
            const gridSize = 50;
            const offsetY = (game.frame * game.speed) % gridSize;
            
            for (let i = 0; i <= game.width; i += gridSize) {
                ctx.beginPath();
                ctx.moveTo(i, 0);
                ctx.lineTo(i, game.height);
                ctx.stroke();
            }
            
            for (let i = 0; i <= game.height; i += gridSize) {
                ctx.beginPath();
                ctx.moveTo(0, i + offsetY - gridSize);
                ctx.lineTo(game.width, i + offsetY - gridSize);
                ctx.stroke();
            }

            ctx.fillStyle = 'white';
            stars.forEach(star => {
                star.y += star.speed + (game.speed * 0.1);
                if (star.y > game.height) star.y = 0;
                ctx.globalAlpha = Math.random() * 0.5 + 0.3;
                ctx.fillRect(star.x, star.y, star.size, star.size);
            });
            ctx.globalAlpha = 1.0;

            if (game.comboTimer > 0) {
                game.comboTimer--;
                if (game.comboTimer <= 0) {
                    game.combo = 0;
                    updateUI();
                }
            }

            player.update();
            player.draw();

            // 更新和绘制子弹
            for (let i = bullets.length - 1; i >= 0; i--) {
                const bullet = bullets[i];
                bullet.update();
                bullet.draw();
                
                if (!bullet.active) {
                    bullets.splice(i, 1);
                }
            }

            // 调整敌人生成频率
            const spawnRate = Math.max(40, 100 - Math.floor(game.speed * 10)); 
            if (game.frame % spawnRate === 0) {
                enemies.push(new Enemy());
            }

            // 更新和绘制敌人
            for (let i = enemies.length - 1; i >= 0; i--) {
                const enemy = enemies[i];
                enemy.update(player);
                enemy.draw();

                // 子弹与敌人碰撞检测
                for (let j = bullets.length - 1; j >= 0; j--) {
                    const bullet = bullets[j];
                    if (checkCollision(bullet, enemy)) {
                        // 子弹击中敌人
                        createExplosion(bullet.x, bullet.y, '#fcee0a', 5);
                        bullet.active = false;
                        
                        // 敌人扣血
                        const isDead = enemy.takeDamage(1);
                        if (isDead) {
                            createExplosion(enemy.x, enemy.y, '#bc13fe', 15);
                            playSound('hit');
                            enemies.splice(i, 1);
                            addScore(enemy.isElite ? 30 : 10);
                        }
                        break; // 一颗子弹只打一个敌人
                    }
                }

                // 玩家与敌人碰撞检测
                if (checkCollision(player, enemy)) {
                    if (player.invincible) {
                        createExplosion(enemy.x, enemy.y, '#bc13fe', 10);
                        playSound('hit');
                        enemies.splice(i, 1);
                        addScore(50);
                    } else {
                        const isDead = player.takeDamage(34);
                        createExplosion(player.x, player.y, '#00f3ff', 15);
                        createExplosion(enemy.x, enemy.y, '#bc13fe', 15);
                        
                        if (isDead) {
                            playSound('gameover');
                            gameOver();
                        }
                        
                        enemies.splice(i, 1);
                    }
                }

                // 移除超出屏幕的敌人
                if (enemy.y > game.height + 50) {
                    enemies.splice(i, 1);
                    addScore(5); // 躲避敌人得分降低
                }
            }

            // 调整道具生成频率
            if (game.frame % 300 === 0) {
                powerUps.push(new PowerUp());
            }

            // 更新和绘制奖励
            for (let i = powerUps.length - 1; i >= 0; i--) {
                const p = powerUps[i];
                p.update();
                p.draw();

                if (checkCollision(player, p)) {
                    createExplosion(p.x, p.y, p.color, 20);
                    
                    if (p.type === PowerUpType.INVINCIBLE) {
                        playSound('powerup');
                        player.invincible = true;
                        player.invincibleTimer = 300; 
                        addScore(100);
                    } else if (p.type === PowerUpType.SHIELD) {
                        playSound('shield');
                        player.hasShield = true;
                        addScore(50);
                    } else if (p.type === PowerUpType.HEALTH) {
                        player.heal(30);
                        addScore(30);
                    } else if (p.type === PowerUpType.WEAPON) {
                        player.upgradeWeapon();
                    }
                    
                    powerUps.splice(i, 1);
                } else if (p.y > game.height + 50) {
                    powerUps.splice(i, 1);
                }
            }

            // 更新和绘制粒子
            for (let i = particles.length - 1; i >= 0; i--) {
                const p = particles[i];
                p.update();
                p.draw();
                if (p.life <= 0) particles.splice(i, 1);
            }

            // 调整难度增加速度
            if (game.frame % 600 === 0) {
                game.speed += 0.1;
            }

            if (game.shake > 0) ctx.restore();

            game.frame++;
            requestAnimationFrame(loop);
        }

        function gameOver() {
            game.active = false;
            game.shake = 20; 
            stopBGM();
            
            if (game.score > game.highScore) {
                game.highScore = game.score;
                localStorage.setItem('cyberDriftHighScore', game.highScore);
                highScoreEl.innerText = game.highScore;
            }
            
            finalScoreEl.innerText = game.score;
            gameOverScreen.classList.remove('hidden');
        }

        function startGame() {
            initAudio();
            init();
            game.active = true;
            startScreen.classList.add('hidden');
            gameOverScreen.classList.add('hidden');
            startBGM();
            loop();
        }

        startBtn.addEventListener('click', startGame);
        restartBtn.addEventListener('click', startGame);

        function drawStaticBackground() {
            ctx.fillStyle = '#050510';
            ctx.fillRect(0, 0, game.width, game.height);
            initStars();
            stars.forEach(star => {
                ctx.fillStyle = 'white';
                ctx.globalAlpha = Math.random() * 0.5 + 0.3;
                ctx.fillRect(star.x, star.y, star.size, star.size);
            });
            ctx.globalAlpha = 1.0;
        }
        drawStaticBackground();

    </script>
</body>
</html>
