<!DOCTYPE html>
<html lang="zh-TW">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no">
    <title>屍境重構：廢墟決戰 v13.0</title>
    <style>
        * { box-sizing: border-box; }
        html, body { width: 100%; height: 100%; margin: 0; padding: 0; overflow: hidden; font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif; user-select: none; background: #000; }
        
        #ui { position: absolute; top: 12px; left: 12px; color: #fff; text-shadow: 2px 2px 4px #000; font-size: 16px; pointer-events: none; z-index: 10; max-width: 80vw; }
        #crosshair { position: absolute; top: 50%; left: 50%; width: 14px; height: 14px; border: 2px solid rgba(255,255,255,0.8); border-radius: 50%; transform: translate(-50%, -50%); pointer-events: none; z-index: 10; }
        #crosshair::after { content: ''; position: absolute; top: 4px; left: 4px; width: 2px; height: 2px; background: red; }
        
        #minimap-container {
            position: absolute;
            bottom: 12px;
            left: 12px;
            width: 160px;
            height: 160px;
            max-width: 28vw;
            max-height: 28vw;
            background: rgba(10, 15, 25, 0.85);
            border: 2px solid #00ffff;
            border-radius: 8px;
            box-shadow: 0 0 15px rgba(0, 255, 255, 0.4);
            pointer-events: none;
            z-index: 10;
            overflow: hidden;
        }
        #minimap { width: 100%; height: 100%; display: block; }

        #difficulty-screen {
            position: absolute; top: 0; left: 0; width: 100vw; height: 100vh;
            background: rgba(5, 5, 10, 0.95); color: white; display: flex;
            flex-direction: column; justify-content: center; align-items: center;
            z-index: 50; text-align: center;
        }

        #shop { position: absolute; top: 50%; left: 50%; transform: translate(-50%, -50%); background: rgba(15, 15, 25, 0.95); color: white; padding: 20px; border-radius: 12px; display: none; text-align: center; border: 2px solid #ff4444; box-shadow: 0 0 20px rgba(255,68,68,0.4); z-index: 20; max-height: 85vh; width: 90%; max-width: 480px; overflow-y: auto; }
        
        #game-over-screen { position: absolute; top: 50%; left: 50%; transform: translate(-50%, -50%); background: rgba(20, 0, 0, 0.95); color: white; padding: 25px; border-radius: 15px; display: none; text-align: center; border: 3px solid #ff0000; box-shadow: 0 0 30px rgba(255,0,0,0.8); z-index: 30; width: 90%; max-width: 400px; }
        .stats-box { background: rgba(255,255,255,0.08); margin: 15px 0; padding: 15px; border-radius: 8px; text-align: left; line-height: 1.8; font-size: 15px; }

        .btn { background: #222; color: white; border: 1px solid #ff4444; padding: 10px 16px; margin: 6px 0; cursor: pointer; font-size: 14px; border-radius: 6px; transition: 0.2s; width: 100%; }
        .btn:hover { background: #ff4444; color: black; font-weight: bold; }
        .btn:disabled { background: #444; border-color: #666; color: #aaa; cursor: not-allowed; }
        
        .diff-btn { font-size: 18px; padding: 15px 30px; width: 280px; margin: 10px; border-radius: 8px; font-weight: bold; }
        .diff-easy { border-color: #00ff66; color: #00ff66; }
        .diff-easy:hover { background: #00ff66; color: #000; }
        .diff-medium { border-color: #ffaa00; color: #ffaa00; }
        .diff-medium:hover { background: #ffaa00; color: #000; }
        .diff-hard { border-color: #ff3333; color: #ff3333; }
        .diff-hard:hover { background: #ff3333; color: #fff; }

        #msg { position: absolute; top: 12%; width: 100%; text-align: center; color: #ffeb3b; font-size: 26px; font-weight: bold; text-shadow: 3px 3px 6px #000; pointer-events: none; display: none; z-index: 10; padding: 0 10px; }
        #drop-msg { position: absolute; top: 22%; width: 100%; text-align: center; color: #00ffff; font-size: 24px; font-weight: bold; text-shadow: 0 0 10px #00ffff, 2px 2px 4px #000; pointer-events: none; display: none; z-index: 10; padding: 0 10px; }
        #damage-flash { position: absolute; top: 0; left: 0; width: 100vw; height: 100vh; background: rgba(255,0,0,0.3); pointer-events: none; display: none; z-index: 5; }
    </style>
    <script src="https://cdnjs.cloudflare.com/ajax/libs/three.js/r128/three.min.js"></script>
</head>
<body>

    <div id="difficulty-screen">
        <h1 style="font-size: 36px; margin-bottom: 10px; text-shadow: 0 0 10px #ff0000;">☣️ 屍境重構：廢墟決戰</h1>
        <p style="color: #aaa; margin-bottom: 30px;">請選擇遊戲難度以開始作戰</p>
        <button class="btn diff-btn diff-easy" onclick="selectDifficulty('easy')">🟢 簡單 (Easy)<br><span style="font-size:12px; font-weight:normal;">首波10隻 | 速度 100% | 血量 100%</span></button>
        <button class="btn diff-btn diff-medium" onclick="selectDifficulty('medium')">🟡 中等 (Medium)<br><span style="font-size:12px; font-weight:normal;">首波20隻 | 速度 110% | 血量 150%</span></button>
        <button class="btn diff-btn diff-hard" onclick="selectDifficulty('hard')">🔴 困難 (Hard)<br><span style="font-size:12px; font-weight:normal;">首波30隻 | 速度 120% | 血量 225%</span></button>
    </div>

    <div id="damage-flash"></div>
    <div id="ui">
        <div>❤️ 血量: <span id="hp" style="color: #ff5555; font-weight: bold;">100</span></div>
        <div>💰 金幣: <span id="gold" style="color: #ffd700; font-weight: bold;">0</span></div>
        <div>🌊 當前波次: <span id="wave" style="color: #00ffff; font-weight: bold;">1</span></div>
        <div>👾 剩餘敵人: <span id="zombie-count" style="color: #ff4444; font-weight: bold;">0</span></div>
        <div>🎯 當前難度: <span id="difficulty-tag" style="font-weight: bold;">簡單</span></div>
        <div>🔫 武器: <span id="weapon">戰術手槍</span> (<span id="ammo">12/12</span>)</div>
        <div style="font-size:12px; color:#aaa; margin-top:4px;">[WASD] 移動 | [滑鼠] 水平旋轉視角 | [空白鍵 Space] 跳躍 | [左鍵] 連射 | [1-6] 切換武器 | [R] 換彈 | [E] 商店</div>
    </div>
    
    <div id="minimap-container">
        <canvas id="minimap" width="160" height="160"></canvas>
    </div>

    <div id="crosshair"></div>
    <div id="msg">戰鬥準備開始！</div>
    <div id="drop-msg">🎁 獲得幸運升級！</div>

    <div id="shop">
        <h2 style="font-size: 20px; margin-top:0;">🛒 軍火庫與技能升級 (按 E 關閉)</h2>
        <div id="weapon-shop" style="margin-bottom: 10px; border-bottom: 1px solid #555; padding-bottom: 10px;">
            <h3>解鎖新武器</h3>
            <button class="btn" id="buy-w2" onclick="buyWeapon(2)">購買 戰術散彈槍 - 💰 150</button>
            <button class="btn" id="buy-w3" onclick="buyWeapon(3)">購買 突擊步槍 - 💰 300</button>
            <button class="btn" id="buy-w4" onclick="buyWeapon(4)">購買 戰術衝鋒槍 - 💰 600</button>
            <button class="btn" id="buy-w5" onclick="buyWeapon(5)">購買 離子電漿毀滅者 - 💰 1200</button>
            <button class="btn" id="buy-w6" onclick="buyWeapon(6)">購買 快速離子電漿毀滅者 - 💰 2500</button>
        </div>
        <div>
            <h3>能力強化 (費用遞增 50%)</h3>
            <button class="btn" id="btn-up-hp" onclick="buyUpgrade('hp')">提升血量上限 (+25 HP) - 💰 <span id="cost-hp">50</span></button>
            <button class="btn" id="btn-up-dmg" onclick="buyUpgrade('dmg')">提升整體傷害 (+20%) - 💰 <span id="cost-dmg">75</span></button>
            <button class="btn" id="btn-up-speed" onclick="buyUpgrade('speed')">戰術機動加速 (+15%) - 💰 <span id="cost-speed">60</span></button>
            <button class="btn" id="btn-up-firerate" onclick="buyUpgrade('firerate')">⚡ 射速提升 (+15%) - 💰 <span id="cost-firerate">80</span></button>
            <button class="btn" id="btn-up-ammo" onclick="buyUpgrade('ammo')">📦 彈藥上限擴充 (+25% 指數成長) - 💰 <span id="cost-ammo">70</span></button>
            <button class="btn" id="btn-up-autoReload" onclick="buyUpgrade('autoReload')">🤖 自動補彈系統 (每5秒自動補滿全彈藥) - 💰 3000</button>
        </div>
        <br>
        <button class="btn" onclick="toggleShop()" style="background: #444;">返回戰場</button>
    </div>

    <div id="game-over-screen">
        <h1 style="color: #ff3333; margin: 0; font-size: 24px;">💀 戰術陣亡</h1>
        <div class="stats-box">
            🌊 <b>最終生存波次：</b> 第 <span id="stat-wave" style="color:#00ffff; font-weight:bold;">0</span> 波<br>
            👾 <b>總計擊殺數量：</b> <span id="stat-kills" style="color:#ff5555; font-weight:bold;">0</span> 隻<br>
            💰 <b>累積獲得金幣：</b> <span id="stat-gold" style="color:#ffd700; font-weight:bold;">0</span> 金幣
        </div>
        <button class="btn" onclick="location.reload()" style="font-size: 16px; padding: 10px 16px; background: #ff4444; color: white;">🔄 重新開始遊戲</button>
    </div>

<script>
let scene, camera, renderer;
let hp = 100, maxHp = 100, gold = 0, wave = 1;
let totalZombiesInWave = 0, killedZombiesInWave = 0;
let moveSpeed = 0.18, damageMult = 1.0;
let fireRateMult = 1.0;

let difficulty = 'easy';
let diffMult = { hp: 1.0, speed: 1.0, initialZombies: 10, label: "🟢 簡單", color: "#00ff66" };

let yVelocity = 0;
const gravity = 0.015;
const jumpStrength = 0.38;
let isGrounded = true;

let lastReloadTime = 0;
let hasAutoReload = false;

let zombies = [], bullets = [], enemyBullets = [], particles = [], medkits = [];
let keys = {};
let isPaused = false, isGameOver = false;
let waveTransitioning = false;
let isMouseDown = false;
const MAP_SIZE = 40;

let minimapCanvas, minimapCtx;
let totalKills = 0, totalGoldEarned = 0;

const upgradeCosts = { hp: 50, dmg: 75, speed: 60, firerate: 80, ammo: 70 };

const weapons = {
    1: { name: "戰術手槍", maxAmmo: 12, ammo: 12, dmg: 35, baseFireRate: 250, fireRate: 250, auto: false, unlocked: true, price: 0, bulletColor: 0xffff00 },
    2: { name: "戰術散彈槍", maxAmmo: 6, ammo: 6, dmg: 25, count: 6, baseFireRate: 750, fireRate: 750, auto: false, unlocked: false, price: 150, bulletColor: 0xffa500 },
    3: { name: "突擊步槍", maxAmmo: 30, ammo: 30, dmg: 40, baseFireRate: 110, fireRate: 110, auto: true, unlocked: false, price: 300, bulletColor: 0xff4500 },
    4: { name: "戰術衝鋒槍", maxAmmo: 45, ammo: 45, dmg: 22, baseFireRate: 60, fireRate: 60, auto: true, unlocked: false, price: 600, bulletColor: 0xffff00 },
    5: { name: "離子電漿毀滅者", maxAmmo: 25, ammo: 25, dmg: 100, count: 2, baseFireRate: 140, fireRate: 140, auto: true, unlocked: false, price: 1200, bulletColor: 0x00ffff },
    6: { name: "快速離子電漿毀滅者", maxAmmo: 40, ammo: 40, dmg: 110, count: 2, baseFireRate: 93, fireRate: 93, auto: true, unlocked: false, price: 2500, bulletColor: 0x00ffaa }
};
let currentWeaponKey = 1;

function selectDifficulty(diff) {
    difficulty = diff;
    if (diff === 'easy') {
        diffMult = { hp: 1.0, speed: 1.0, initialZombies: 10, label: "🟢 簡單", color: "#00ff66" };
    } else if (diff === 'medium') {
        diffMult = { hp: 1.5, speed: 1.1, initialZombies: 20, label: "🟡 中等", color: "#ffaa00" };
    } else if (diff === 'hard') {
        diffMult = { hp: 2.25, speed: 1.2, initialZombies: 30, label: "🔴 困難", color: "#ff3333" };
    }

    const tag = document.getElementById('difficulty-tag');
    tag.innerText = diffMult.label;
    tag.style.color = diffMult.color;

    document.getElementById('difficulty-screen').style.display = 'none';
    init();
}

function init() {
    scene = new THREE.Scene();
    scene.background = new THREE.Color(0x0a0a12);
    scene.fog = new THREE.FogExp2(0x0a0a12, 0.02);

    camera = new THREE.PerspectiveCamera(75, window.innerWidth / window.innerHeight, 0.1, 1000);
    camera.position.set(0, 1.7, 0);

    renderer = new THREE.WebGLRenderer({ antialias: true });
    renderer.setSize(window.innerWidth, window.innerHeight);
    renderer.shadowMap.enabled = true;
    renderer.shadowMap.type = THREE.PCFSoftShadowMap;
    document.body.appendChild(renderer.domElement);

    minimapCanvas = document.getElementById('minimap');
    minimapCtx = minimapCanvas.getContext('2d');

    const ambientLight = new THREE.AmbientLight(0x333344, 1.2);
    scene.add(ambientLight);

    const dirLight = new THREE.DirectionalLight(0xffaa66, 1.5);
    dirLight.position.set(20, 40, 20);
    dirLight.castShadow = true;
    dirLight.shadow.mapSize.width = 2048;
    dirLight.shadow.mapSize.height = 2048;
    scene.add(dirLight);

    buildEnvironment();

    document.addEventListener('keydown', (e) => {
        if (isGameOver) return;
        
        if (e.code === 'Space') {
            if (isGrounded) {
                yVelocity = jumpStrength;
                isGrounded = false;
            }
        }

        keys[e.key.toLowerCase()] = true;
        if(e.key === 'e' || e.key === 'E') toggleShop();
        if(['1','2','3','4','5','6'].includes(e.key)) switchWeapon(parseInt(e.key));
        if(e.key.toLowerCase() === 'r') reloadAmmo();
    });
    document.addEventListener('keyup', (e) => keys[e.key.toLowerCase()] = false);
    
    document.body.addEventListener('click', () => {
        if (!isPaused && !isGameOver) document.body.requestPointerLock();
    });
    
    // 恢復為原本僅水平左右旋轉
    document.addEventListener('mousemove', (e) => {
        if (document.pointerLockElement === document.body && !isPaused && !isGameOver) {
            camera.rotation.y -= e.movementX * 0.0022;
        }
    });

    document.addEventListener('mousedown', (e) => {
        if (e.button === 0 && document.pointerLockElement === document.body && !isPaused && !isGameOver) {
            isMouseDown = true;
            shoot();
        }
    });
    document.addEventListener('mouseup', (e) => {
        if (e.button === 0) isMouseDown = false;
    });

    setInterval(spawnMedkit, 12000);
    spawnMedkit();

    setInterval(() => {
        if (hasAutoReload && !isGameOver) {
            const w = weapons[currentWeaponKey];
            if (w.unlocked && w.ammo < w.maxAmmo) {
                w.ammo = w.maxAmmo;
                updateUI();
            }
        }
    }, 5000);

    startNextWaveCountdown();
    animate();
}

function buildEnvironment() {
    const floorGeo = new THREE.PlaneGeometry(MAP_SIZE * 2, MAP_SIZE * 2);
    const floorMat = new THREE.MeshStandardMaterial({ color: 0x1a1a1a, roughness: 0.8 });
    const floor = new THREE.Mesh(floorGeo, floorMat);
    floor.rotation.x = -Math.PI / 2;
    floor.receiveShadow = true;
    scene.add(floor);

    const wallMat = new THREE.MeshStandardMaterial({ color: 0x441111, roughness: 0.5 });
    const wallHeight = 8;
    const wallGeos = [
        new THREE.BoxGeometry(MAP_SIZE * 2, wallHeight, 1),
        new THREE.BoxGeometry(MAP_SIZE * 2, wallHeight, 1),
        new THREE.BoxGeometry(1, wallHeight, MAP_SIZE * 2),
        new THREE.BoxGeometry(1, wallHeight, MAP_SIZE * 2)
    ];
    const wallPos = [
        [0, wallHeight/2, -MAP_SIZE],
        [0, wallHeight/2, MAP_SIZE],
        [-MAP_SIZE, wallHeight/2, 0],
        [MAP_SIZE, wallHeight/2, 0]
    ];

    for(let i=0; i<4; i++) {
        const wall = new THREE.Mesh(wallGeos[i], wallMat);
        wall.position.set(...wallPos[i]);
        wall.castShadow = true;
        wall.receiveShadow = true;
        scene.add(wall);
    }
}

function createMedkitMesh() {
    const group = new THREE.Group();
    const boxGeo = new THREE.BoxGeometry(0.8, 0.5, 0.6);
    const boxMat = new THREE.MeshStandardMaterial({ color: 0xffffff, roughness: 0.3 });
    const box = new THREE.Mesh(boxGeo, boxMat);
    box.position.y = 0.25;
    box.castShadow = true;
    group.add(box);

    const crossMat = new THREE.MeshBasicMaterial({ color: 0x00ff66 });
    const c1 = new THREE.Mesh(new THREE.BoxGeometry(0.4, 0.52, 0.12), crossMat);
    const c2 = new THREE.Mesh(new THREE.BoxGeometry(0.12, 0.52, 0.4), crossMat);
    c1.position.y = 0.25;
    c2.position.y = 0.25;
    group.add(c1); group.add(c2);

    return group;
}

function spawnMedkit() {
    if (medkits.length >= 3 || isGameOver) return;

    const mesh = createMedkitMesh();
    const rx = (Math.random() - 0.5) * (MAP_SIZE * 2 - 10);
    const rz = (Math.random() - 0.5) * (MAP_SIZE * 2 - 10);
    mesh.position.set(rx, 0, rz);

    scene.add(mesh);
    medkits.push({ mesh, healAmount: 20 });
}

function createZombieMesh(color, scale, eyeColor = 0x00ffcc) {
    const group = new THREE.Group();
    const bodyMat = new THREE.MeshStandardMaterial({ color: color, roughness: 0.5 });
    const eyeMat = new THREE.MeshBasicMaterial({ color: eyeColor });

    const torso = new THREE.Mesh(new THREE.BoxGeometry(0.6, 0.9, 0.4), bodyMat);
    torso.position.y = 0.9;
    torso.castShadow = true;
    group.add(torso);

    const head = new THREE.Mesh(new THREE.BoxGeometry(0.4, 0.4, 0.4), bodyMat);
    head.position.y = 1.55;
    head.castShadow = true;
    group.add(head);

    const eye1 = new THREE.Mesh(new THREE.BoxGeometry(0.08, 0.05, 0.05), eyeMat);
    eye1.position.set(0.1, 1.6, -0.2);
    const eye2 = eye1.clone();
    eye2.position.x = -0.1;
    group.add(eye1); group.add(eye2);

    const armGeo = new THREE.BoxGeometry(0.15, 0.7, 0.15);
    const armL = new THREE.Mesh(armGeo, bodyMat);
    armL.position.set(0.4, 1.1, -0.3);
    armL.rotation.x = -Math.PI / 3;
    armL.castShadow = true;
    const armR = armL.clone();
    armR.position.x = -0.4;
    group.add(armL); group.add(armR);

    group.scale.set(scale, scale, scale);
    return group;
}

function spawnZombie(forceBoss = false) {
    let color = 0x2d5a27;
    let eyeColor = 0x00ffcc;
    let baseSpeed = 0.045;
    let baseHp = 60;
    let scale = 1.0;
    let isBoss = false;

    // 🧬 隨波數持續成長與進化機制
    let waveScalingHp = 1 + (wave - 1) * 0.08;
    let evoChance = Math.min(0.8, (wave - 1) * 0.08); // 隨波數提升進化機率

    if (forceBoss || (wave % 5 === 0 && Math.random() < 0.2)) {
        color = 0x800080; eyeColor = 0xff0055; baseSpeed = 0.022; baseHp = 600; scale = 2.4; isBoss = true;
    } else if (Math.random() < evoChance) {
        // 殭屍進化能力庫
        let evos = ['speed', 'tank', 'toxic'];
        if (wave >= 8) evos.push('void');
        let chosenEvo = evos[Math.floor(Math.random() * evos.length)];

        if (chosenEvo === 'speed') { // 疾速狂暴
            color = 0xffaa00; eyeColor = 0xffff00; baseSpeed = 0.11 + (wave * 0.003); baseHp = 45; scale = 0.95;
        } else if (chosenEvo === 'tank') { // 鋼鐵裝甲
            color = 0x334455; eyeColor = 0x00ffff; baseSpeed = 0.03; baseHp = 180; scale = 1.5;
        } else if (chosenEvo === 'toxic') { // 毒素自爆型
            color = 0x00ff55; eyeColor = 0xff00ff; baseSpeed = 0.07; baseHp = 70; scale = 1.1;
        } else if (chosenEvo === 'void') { // 虛空狂暴
            color = 0x8800ff; eyeColor = 0xffffff; baseSpeed = 0.10; baseHp = 250; scale = 1.3;
        }
    } else {
        // 一般型殭屍隨波數輕微加速
        baseSpeed += Math.min(0.04, wave * 0.001);
    }

    let speed = baseSpeed * diffMult.speed;
    let zHp = baseHp * diffMult.hp * waveScalingHp;

    const mesh = createZombieMesh(color, scale, eyeColor);
    const angle = Math.random() * Math.PI * 2;
    const dist = 15 + Math.random() * (MAP_SIZE - 20);
    mesh.position.set(
        camera.position.x + Math.cos(angle) * dist,
        0,
        camera.position.z + Math.sin(angle) * dist
    );
    mesh.position.x = Math.max(-MAP_SIZE+2, Math.min(MAP_SIZE-2, mesh.position.x));
    mesh.position.z = Math.max(-MAP_SIZE+2, Math.min(MAP_SIZE-2, mesh.position.z));

    scene.add(mesh);
    
    zombies.push({ 
        mesh, hp: zHp, maxHp: zHp, speed, scale, color, isBoss,
        height: 1.8 * scale,
        strafeDir: (Math.random() < 0.5 ? 1 : -1),
        strafeTimer: Math.floor(Math.random() * 60),
        lastShootTime: Date.now()
    });
}

function bossShoot(boss) {
    const startPos = boss.mesh.position.clone().add(new THREE.Vector3(0, 1.5 * boss.scale, 0));
    const targetPos = camera.position.clone();
    const dir = new THREE.Vector3().subVectors(targetPos, startPos).normalize();

    const geo = new THREE.SphereGeometry(0.3);
    const mat = new THREE.MeshBasicMaterial({ color: 0xff0055 });
    const bullet = new THREE.Mesh(geo, mat);
    bullet.position.copy(startPos);

    enemyBullets.push({ mesh: bullet, dir, speed: 0.35, life: 120, damage: 20 * diffMult.hp });
    scene.add(bullet);
}

function startNextWaveCountdown() {
    waveTransitioning = true;
    let countdown = 3;
    let bossNotice = (wave % 5 === 0) ? " ⚠️ 警告：超級Boss關卡！" : "";
    showMsg(`🎉 本波清掃完畢！ ${countdown} 秒後開始第 ${wave} 波${bossNotice}`);
    
    let timer = setInterval(() => {
        countdown--;
        if (countdown > 0) {
            showMsg(`下一波倒數: ${countdown} 秒`);
        } else {
            clearInterval(timer);
            showMsg(`⚠️ 第 ${wave} 波殭屍來襲！`);
            waveTransitioning = false;
            spawnWave();
        }
    }, 1000);
}

function spawnWave() {
    if (wave === 1) {
        totalZombiesInWave = diffMult.initialZombies;
    } else {
        totalZombiesInWave = Math.floor(totalZombiesInWave * 1.3 + 3);
    }
    
    killedZombiesInWave = 0;
    updateUI();

    let spawned = 0;
    
    if (wave % 5 === 0) {
        spawnZombie(true);
        spawned++;
    }

    let maxTotalSpawnTime = 20000;
    let baseInterval = Math.max(150, 700 - (wave * 35));
    let requiredInterval = maxTotalSpawnTime / totalZombiesInWave;
    let spawnInterval = Math.min(baseInterval, requiredInterval);

    let timer = setInterval(() => {
        if (spawned < totalZombiesInWave && !isPaused && !isGameOver) {
            spawnZombie();
            spawned++;
        } else if (spawned >= totalZombiesInWave) {
            clearInterval(timer);
        }
    }, spawnInterval);
}

function createHitParticles(pos, colorHex) {
    for (let i = 0; i < 6; i++) {
        const pGeo = new THREE.SphereGeometry(0.06);
        const pMat = new THREE.MeshBasicMaterial({ color: colorHex });
        const p = new THREE.Mesh(pGeo, pMat);
        p.position.copy(pos);
        
        const vel = new THREE.Vector3(
            (Math.random() - 0.5) * 0.3,
            (Math.random() - 0.2) * 0.3,
            (Math.random() - 0.5) * 0.3
        );
        particles.push({ mesh: p, vel, life: 15 });
        scene.add(p);
    }
}

function shoot() {
    const w = weapons[currentWeaponKey];
    if (!w.unlocked) return;
    const now = Date.now();
    if (now - (w.lastShot || 0) < w.fireRate || w.ammo <= 0) return;
    
    w.lastShot = now;
    w.ammo--;
    updateUI();

    const createBullet = (dirOffset = new THREE.Vector3()) => {
        const geo = new THREE.SphereGeometry((currentWeaponKey === 5 || currentWeaponKey === 6) ? 0.2 : 0.08);
        const mat = new THREE.MeshBasicMaterial({ color: w.bulletColor });
        const bullet = new THREE.Mesh(geo, mat);
        bullet.position.copy(camera.position).add(new THREE.Vector3(0, -0.2, 0));

        const dir = new THREE.Vector3(0, 0, -1).applyQuaternion(camera.quaternion).add(dirOffset).normalize();
        bullets.push({ mesh: bullet, dir: dir, damage: w.dmg * damageMult, life: 60, color: w.bulletColor });
        scene.add(bullet);
    };

    if (w.count && w.count > 1) {
        for (let i = 0; i < w.count; i++) {
            let offset = new THREE.Vector3((Math.random()-0.5)*0.12, (Math.random()-0.5)*0.12, (Math.random()-0.5)*0.12);
            createBullet(offset);
        }
    } else {
        createBullet();
    }
}

function triggerLuckyDrop() {
    // 🎁 1% 掉落幸運升級庫
    const drops = ['hp', 'dmg', 'speed', 'firerate', 'ammo', 'gold'];
    const type = drops[Math.floor(Math.random() * drops.length)];
    let text = "";

    if (type === 'hp') {
        maxHp += 20; hp = Math.min(maxHp, hp + 30);
        text = "🎁 幸運掉落：生命值上限 +20 & 回復 30 HP！";
    } else if (type === 'dmg') {
        damageMult += 0.15;
        text = "🎁 幸運掉落：武器整體傷害 +15%！";
    } else if (type === 'speed') {
        moveSpeed += 0.02;
        text = "🎁 幸運掉落：機動速度 +10%！";
    } else if (type === 'firerate') {
        fireRateMult *= 0.90;
        for (let k in weapons) weapons[k].fireRate = weapons[k].baseFireRate * fireRateMult;
        text = "🎁 幸運掉落：全武器射速提升 10%！";
    } else if (type === 'ammo') {
        for (let k in weapons) {
            weapons[k].maxAmmo += 5;
            weapons[k].ammo = weapons[k].maxAmmo;
        }
        text = "🎁 幸運掉落：全武器彈藥容量 +5！";
    } else if (type === 'gold') {
        let bonusGold = 200 + wave * 50;
        gold += bonusGold;
        totalGoldEarned += bonusGold;
        text = `🎁 幸運掉落：獲得額外金幣 💰 ${bonusGold}！`;
    }

    showDropMsg(text);
    updateUI();
}

function showDropMsg(txt) {
    const dropMsg = document.getElementById('drop-msg');
    dropMsg.innerText = txt;
    dropMsg.style.display = 'block';
    setTimeout(() => { dropMsg.style.display = 'none'; }, 3000);
}

function reloadAmmo() {
    const now = Date.now();
    if (now - lastReloadTime < 1000) {
        showMsg("⏳ 換彈冷卻中 (1秒)...");
        return;
    }

    const w = weapons[currentWeaponKey];
    if (w.unlocked) {
        if (w.ammo === w.maxAmmo) return;
        lastReloadTime = now;
        w.ammo = w.maxAmmo;
        updateUI();
        showMsg("彈藥完成補充！");
    }
}

function switchWeapon(key) {
    if (weapons[key] && weapons[key].unlocked) {
        currentWeaponKey = key;
        updateUI();
    } else if (weapons[key]) {
        showMsg("該武器尚未解鎖，請在商店購買！");
    }
}

function toggleShop() {
    if (isGameOver) return;
    isPaused = !isPaused;
    isMouseDown = false;
    document.getElementById('shop').style.display = isPaused ? 'block' : 'none';
    if (isPaused) {
        document.exitPointerLock();
        updateShopUI();
    }
}

function buyWeapon(key) {
    const w = weapons[key];
    if (!w.unlocked && gold >= w.price) {
        gold -= w.price;
        w.unlocked = true;
        w.fireRate = w.baseFireRate * fireRateMult;
        
        currentWeaponKey = key;
        showMsg(`解鎖新武器: ${w.name}！`);
        updateUI();
        updateShopUI();
    } else if (gold < w.price) {
        alert("金幣不足！");
    }
}

function buyUpgrade(type) {
    if (type === 'autoReload') {
        if (!hasAutoReload && gold >= 3000) {
            gold -= 3000;
            hasAutoReload = true;
            showMsg("🤖 成功購買『自動補彈系統』！");
            updateUI();
            updateShopUI();
        } else if (gold < 3000) {
            alert("金幣不足 3000！");
        }
        return;
    }

    const cost = upgradeCosts[type];
    if (gold >= cost) {
        gold -= cost;
        if (type === 'hp') {
            maxHp += 25; hp += 25;
        } else if (type === 'dmg') {
            damageMult += 0.20;
        } else if (type === 'speed') {
            moveSpeed += 0.03;
        } else if (type === 'firerate') {
            fireRateMult *= 0.85;
            for (let k in weapons) {
                weapons[k].fireRate = weapons[k].baseFireRate * fireRateMult;
            }
        } else if (type === 'ammo') {
            for (let k in weapons) {
                let increase = Math.max(1, Math.floor(weapons[k].maxAmmo * 0.25));
                weapons[k].maxAmmo += increase;
                weapons[k].ammo = weapons[k].maxAmmo;
            }
        }
        upgradeCosts[type] = Math.floor(cost * 1.5);
        updateUI();
        updateShopUI();
    } else {
        alert("金幣不足！");
    }
}

function updateShopUI() {
    for (let k in weapons) {
        const btn = document.getElementById(`buy-w${k}`);
        if (btn) {
            if (weapons[k].unlocked) {
                btn.innerText = `已擁有 ${weapons[k].name}`;
                btn.disabled = true;
            } else {
                btn.disabled = gold < weapons[k].price;
            }
        }
    }
    document.getElementById('cost-hp').innerText = upgradeCosts.hp;
    document.getElementById('cost-dmg').innerText = upgradeCosts.dmg;
    document.getElementById('cost-speed').innerText = upgradeCosts.speed;
    document.getElementById('cost-firerate').innerText = upgradeCosts.firerate;
    document.getElementById('cost-ammo').innerText = upgradeCosts.ammo;

    const autoBtn = document.getElementById('btn-up-autoReload');
    if (hasAutoReload) {
        autoBtn.innerText = "已擁有 🤖 自動補彈系統";
        autoBtn.disabled = true;
    } else {
        autoBtn.disabled = gold < 3000;
    }
}

function updateUI() {
    document.getElementById('hp').innerText = `${Math.max(0, Math.floor(hp))}/${maxHp}`;
    document.getElementById('gold').innerText = gold;
    document.getElementById('wave').innerText = wave;
    
    let remainingZombies = totalZombiesInWave - killedZombiesInWave;
    document.getElementById('zombie-count').innerText = Math.max(0, remainingZombies);

    const w = weapons[currentWeaponKey];
    document.getElementById('weapon').innerText = w.name;
    document.getElementById('ammo').innerText = `${w.ammo}/${w.maxAmmo}`;
}

function drawMinimap() {
    const width = minimapCanvas.width;
    const height = minimapCanvas.height;
    const padding = 8;
    const drawWidth = width - padding * 2;
    const drawHeight = height - padding * 2;

    const mapToCanvas = (x, z) => {
        const cx = padding + ((x + MAP_SIZE) / (MAP_SIZE * 2)) * drawWidth;
        const cy = padding + ((z + MAP_SIZE) / (MAP_SIZE * 2)) * drawHeight;
        return { x: cx, y: cy };
    };

    minimapCtx.clearRect(0, 0, width, height);

    minimapCtx.strokeStyle = 'rgba(0, 255, 255, 0.4)';
    minimapCtx.lineWidth = 2;
    minimapCtx.strokeRect(padding, padding, drawWidth, drawHeight);

    medkits.forEach(m => {
        const p = mapToCanvas(m.mesh.position.x, m.mesh.position.z);
        minimapCtx.fillStyle = '#00ff66';
        minimapCtx.beginPath();
        minimapCtx.arc(p.x, p.y, 3, 0, Math.PI * 2);
        minimapCtx.fill();
    });

    zombies.forEach(z => {
        const p = mapToCanvas(z.mesh.position.x, z.mesh.position.z);
        minimapCtx.fillStyle = z.isBoss ? '#ff00ff' : '#ff3333';
        minimapCtx.beginPath();
        minimapCtx.arc(p.x, p.y, z.isBoss ? 4 : 2, 0, Math.PI * 2);
        minimapCtx.fill();
    });

    const pPlayer = mapToCanvas(camera.position.x, camera.position.z);

    minimapCtx.fillStyle = '#ffffff';
    minimapCtx.beginPath();
    minimapCtx.arc(pPlayer.x, pPlayer.y, 3, 0, Math.PI * 2);
    minimapCtx.fill();

    const forward = new THREE.Vector3(0, 0, -1).applyQuaternion(camera.quaternion);
    const lineLen = 12;
    minimapCtx.strokeStyle = '#ffffff';
    minimapCtx.lineWidth = 1.5;
    minimapCtx.beginPath();
    minimapCtx.moveTo(pPlayer.x, pPlayer.y);
    minimapCtx.lineTo(pPlayer.x + forward.x * lineLen, pPlayer.y + forward.z * lineLen);
    minimapCtx.stroke();
}

function showMsg(txt) {
    const msg = document.getElementById('msg');
    msg.innerText = txt;
    msg.style.display = 'block';
    setTimeout(() => { if (!waveTransitioning) msg.style.display = 'none'; }, 2000);
}

function gameOver() {
    isGameOver = true;
    document.exitPointerLock();
    
    document.getElementById('stat-wave').innerText = wave;
    document.getElementById('stat-kills').innerText = totalKills;
    document.getElementById('stat-gold').innerText = totalGoldEarned;
    document.getElementById('game-over-screen').style.display = 'block';
}

function animate() {
    requestAnimationFrame(animate);
    if (isPaused || isGameOver) return;

    if (isMouseDown && weapons[currentWeaponKey].auto) {
        shoot();
    }

    camera.position.y += yVelocity;
    yVelocity -= gravity;
    if (camera.position.y <= 1.7) {
        camera.position.y = 1.7;
        yVelocity = 0;
        isGrounded = true;
    }

    const dir = new THREE.Vector3();
    if (keys['w']) dir.z -= 1;
    if (keys['s']) dir.z += 1;
    if (keys['a']) dir.x -= 1;
    if (keys['d']) dir.x += 1;
    dir.normalize().applyQuaternion(camera.quaternion);
    dir.y = 0;
    camera.position.addScaledVector(dir, moveSpeed);

    const borderLimit = MAP_SIZE - 1.5;
    camera.position.x = Math.max(-borderLimit, Math.min(borderLimit, camera.position.x));
    camera.position.z = Math.max(-borderLimit, Math.min(borderLimit, camera.position.z));

    for (let i = medkits.length - 1; i >= 0; i--) {
        let m = medkits[i];
        m.mesh.rotation.y += 0.02;
        m.mesh.position.y = 0.2 + Math.sin(Date.now() * 0.003) * 0.1;

        if (camera.position.distanceTo(m.mesh.position) < 2.5) {
            hp = Math.min(maxHp, hp + m.healAmount);
            showMsg(`💚 拾取醫藥包，回復了 ${m.healAmount} 點血量！`);
            updateUI();
            
            scene.remove(m.mesh);
            medkits.splice(i, 1);
        }
    }

    for (let i = bullets.length - 1; i >= 0; i--) {
        let b = bullets[i];
        b.mesh.position.addScaledVector(b.dir, 0.9);
        b.life--;

        for (let j = zombies.length - 1; j >= 0; j--) {
            let z = zombies[j];
            if (b.mesh.position.distanceTo(z.mesh.position.clone().add(new THREE.Vector3(0, 1, 0))) < 0.9 * z.scale) {
                z.hp -= b.damage;
                z.mesh.position.addScaledVector(b.dir, 0.15);
                
                createHitParticles(b.mesh.position, b.color);
                
                z.mesh.children.forEach(child => {
                    if (child.material) child.material.color.setHex(0xffffff);
                });
                setTimeout(() => {
                    if (z.mesh) {
                        z.mesh.children.forEach(child => {
                            if (child.material && child.material !== scene.eyeMat) child.material.color.setHex(z.color);
                        });
                    }
                }, 70);

                scene.remove(b.mesh);
                bullets.splice(i, 1);

                if (z.hp <= 0) {
                    scene.remove(z.mesh);
                    zombies.splice(j, 1);
                    killedZombiesInWave++;
                    totalKills++;
                    
                    let earned = z.isBoss ? 250 : 25 + wave * 2;
                    gold += earned;
                    totalGoldEarned += earned;

                    // 🎲 1% 幾率掉落幸運升級
                    if (Math.random() < 0.01) {
                        triggerLuckyDrop();
                    }

                    updateUI();
                }
                break;
            }
        }

        if (b && b.life <= 0) {
            scene.remove(b.mesh);
            bullets.splice(i, 1);
        }
    }

    for (let i = enemyBullets.length - 1; i >= 0; i--) {
        let eb = enemyBullets[i];
        eb.mesh.position.addScaledVector(eb.dir, eb.speed);
        eb.life--;

        if (eb.mesh.position.distanceTo(camera.position) < 1.2) {
            hp -= eb.damage;
            updateUI();

            const flash = document.getElementById('damage-flash');
            flash.style.display = 'block';
            setTimeout(() => flash.style.display = 'none', 80);

            if (hp <= 0) gameOver();

            scene.remove(eb.mesh);
            enemyBullets.splice(i, 1);
            continue;
        }

        if (eb.life <= 0) {
            scene.remove(eb.mesh);
            enemyBullets.splice(i, 1);
        }
    }

    for (let i = particles.length - 1; i >= 0; i--) {
        let p = particles[i];
        p.mesh.position.add(p.vel);
        p.life--;
        if (p.life <= 0) {
            scene.remove(p.mesh);
            particles.splice(i, 1);
        }
    }

    for (let i = zombies.length - 1; i >= 0; i--) {
        let z = zombies[i];
        
        let forward = new THREE.Vector3().subVectors(camera.position, z.mesh.position);
        
        let horizontalDist = Math.sqrt(forward.x * forward.x + forward.z * forward.z);
        let playerFootY = camera.position.y - 1.7;
        let isPlayerAboveEnemy = playerFootY > (z.height - 0.2);

        forward.y = 0;
        forward.normalize();

        if (z.isBoss && Date.now() - z.lastShootTime > 2500) {
            z.lastShootTime = Date.now();
            bossShoot(z);
        }

        z.strafeTimer--;
        if (z.strafeTimer <= 0) {
            z.strafeDir = Math.random() < 0.5 ? 1 : -1;
            z.strafeTimer = 30 + Math.floor(Math.random() * 60);
        }
        let sideDir = new THREE.Vector3(-forward.z, 0, forward.x).multiplyScalar(z.strafeDir);
        let attackRange = 1.8 * z.scale;

        if (horizontalDist > attackRange) {
            let moveDirZ = new THREE.Vector3()
                .addScaledVector(forward, 0.85)
                .addScaledVector(sideDir, 0.3)
                .normalize();

            z.mesh.position.addScaledVector(moveDirZ, z.speed);
        }
        
        z.mesh.lookAt(camera.position.x, 0, camera.position.z);

        if (horizontalDist <= attackRange + 0.3 && !isPlayerAboveEnemy) {
            let dmg = (z.isBoss ? 1.5 : 0.4) * (diffMult.hp * 0.8 + 0.2);
            hp -= dmg;
            updateUI();
            
            const flash = document.getElementById('damage-flash');
            flash.style.display = 'block';
            setTimeout(() => flash.style.display = 'none', 50);

            if (hp <= 0) {
                gameOver();
            }
        }
    }

    if (killedZombiesInWave >= totalZombiesInWave && totalZombiesInWave > 0 && !waveTransitioning) {
        wave++;
        startNextWaveCountdown();
    }

    drawMinimap();

    renderer.render(scene, camera);
}

window.addEventListener('resize', () => {
    if (camera && renderer) {
        camera.aspect = window.innerWidth / window.innerHeight;
        camera.updateProjectionMatrix();
        renderer.setSize(window.innerWidth, window.innerHeight);
    }
});
</script>
</body>
</html>
