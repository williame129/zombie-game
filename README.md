<!DOCTYPE html>
<html lang="zh-TW">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no">
    <title>屍境重構：廢墟決戰 v21.0 (畫質選單與傷害倍增版)</title>
    <style>
        * { box-sizing: border-box; }
        html, body { width: 100%; height: 100%; margin: 0; padding: 0; overflow: hidden; font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif; user-select: none; background: #000; }
        
        #ui { position: absolute; top: 12px; left: 12px; color: #fff; text-shadow: 2px 2px 4px #000; font-size: 16px; pointer-events: none; z-index: 10; max-width: 80vw; }
        .pause-btn { pointer-events: auto; background: rgba(255, 255, 255, 0.2); border: 1px solid #fff; color: white; padding: 3px 8px; border-radius: 4px; cursor: pointer; font-size: 12px; margin-bottom: 5px; }
        .pause-btn:hover { background: #00ffff; color: #000; }

        #crosshair { position: absolute; top: 50%; left: 50%; width: 14px; height: 14px; border: 2px solid rgba(255,255,255,0.8); border-radius: 50%; transform: translate(-50%, -50%); pointer-events: none; z-index: 10; }
        #crosshair::after { content: ''; position: absolute; top: 4px; left: 4px; width: 2px; height: 2px; background: red; }
        
        #minimap-container {
            position: absolute; bottom: 12px; left: 12px; width: 160px; height: 160px;
            max-width: 28vw; max-height: 28vw; background: rgba(10, 15, 25, 0.85);
            border: 2px solid #00ffff; border-radius: 8px; box-shadow: 0 0 15px rgba(0, 255, 255, 0.4);
            pointer-events: none; z-index: 10; overflow: hidden;
        }
        #minimap { width: 100%; height: 100%; display: block; }

        #difficulty-screen {
            position: absolute; top: 0; left: 0; width: 100vw; height: 100vh;
            background: rgba(5, 5, 10, 0.95); color: white; display: flex;
            flex-direction: column; justify-content: center; align-items: center;
            z-index: 50; text-align: center;
        }

        #pause-menu {
            position: absolute; top: 50%; left: 50%; transform: translate(-50%, -50%);
            background: rgba(10, 10, 20, 0.95); color: white; padding: 25px; border-radius: 12px;
            display: none; text-align: center; border: 2px solid #00ffff; box-shadow: 0 0 25px rgba(0,255,255,0.5);
            z-index: 40; width: 90%; max-width: 400px;
        }

        #shop { position: absolute; top: 50%; left: 50%; transform: translate(-50%, -50%); background: rgba(15, 15, 25, 0.95); color: white; padding: 20px; border-radius: 12px; display: none; text-align: center; border: 2px solid #ff4444; box-shadow: 0 0 20px rgba(255,68,68,0.4); z-index: 20; max-height: 85vh; width: 95%; max-width: 600px; overflow-y: auto; }
        
        #game-over-screen { position: absolute; top: 50%; left: 50%; transform: translate(-50%, -50%); background: rgba(20, 0, 0, 0.95); color: white; padding: 25px; border-radius: 15px; display: none; text-align: center; border: 3px solid #ff0000; box-shadow: 0 0 30px rgba(255,0,0,0.8); z-index: 30; width: 90%; max-width: 400px; }
        .stats-box { background: rgba(255,255,255,0.08); margin: 15px 0; padding: 15px; border-radius: 8px; text-align: left; line-height: 1.8; font-size: 15px; }

        .btn { background: #222; color: white; border: 1px solid #ff4444; padding: 8px 12px; margin: 4px 0; cursor: pointer; font-size: 14px; border-radius: 6px; transition: 0.2s; width: 100%; }
        .btn:hover { background: #ff4444; color: black; font-weight: bold; }
        .btn:disabled { background: #444; border-color: #666; color: #aaa; cursor: not-allowed; }

        .quality-btn { border: 1px solid #00ffff; color: #00ffff; background: #002233; margin: 5px 0; padding: 10px; font-weight: bold; }
        .quality-btn.active { background: #00ffff; color: #000; }
        
        .weapon-row { display: flex; align-items: center; justify-content: space-between; margin-bottom: 8px; background: rgba(255,255,255,0.05); padding: 8px 10px; border-radius: 6px; gap: 6px; }
        .elem-btn { padding: 4px 6px; font-size: 11px; border-radius: 4px; cursor: pointer; white-space: nowrap; font-weight: bold; }
        .elem-fire { border: 1px solid #ff4400; color: #ff6622; background: #2a0a00; }
        .elem-fire:hover { background: #ff4400; color: #fff; }
        .elem-bomb { border: 1px solid #ffaa00; color: #ffcc00; background: #2a1a00; }
        .elem-bomb:hover { background: #ffaa00; color: #000; }
        .elem-elec { border: 1px solid #00ffff; color: #00ffff; background: #002233; }
        .elem-elec:hover { background: #00ffff; color: #000; }

        .diff-btn { font-size: 18px; padding: 15px 30px; width: 320px; margin: 10px; border-radius: 8px; font-weight: bold; }
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
        <button class="btn diff-btn diff-easy" onclick="selectDifficulty('easy')">🟢 簡單 (Easy)<br><span style="font-size:12px; font-weight:normal;">Boss傷害:20 | 小怪傷害:1.0/0.1s | 💰2倍</span></button>
        <button class="btn diff-btn diff-medium" onclick="selectDifficulty('medium')">🟡 中等 (Medium)<br><span style="font-size:12px; font-weight:normal;">Boss傷害:40 | 小怪傷害:2.0/0.1s | 💰1.4倍</span></button>
        <button class="btn diff-btn diff-hard" onclick="selectDifficulty('hard')">🔴 困難 (Hard)<br><span style="font-size:12px; font-weight:normal;">Boss傷害:60 | 小怪傷害:3.0/0.1s | 💰0.9倍</span></button>
    </div>

    <div id="damage-flash"></div>
    <div id="ui">
        <button class="pause-btn" onclick="togglePauseMenu()">⏸️ 暫停 / 畫質設定</button>
        <div>❤️ 血量: <span id="hp" style="color: #ff5555; font-weight: bold;">100</span></div>
        <div>💰 金幣: <span id="gold" style="color: #ffd700; font-weight: bold;">0</span></div>
        <div>🌊 當前波次: <span id="wave" style="color: #00ffff; font-weight: bold;">1</span></div>
        <div>👾 剩餘敵人: <span id="zombie-count" style="color: #ff4444; font-weight: bold;">0</span></div>
        <div>🎯 當前難度: <span id="difficulty-tag" style="font-weight: bold;">簡單</span></div>
        <div>🔫 武器: <span id="weapon">戰術手槍</span> (<span id="ammo">12/12</span>)</div>
        <div style="font-size:12px; color:#aaa; margin-top:4px;">[WASD] 移動 | [滑鼠] 水平旋轉視角 | [空白鍵 Space] 跳躍 | [左鍵] 連射 | [1-6] 切換武器 | [R] 換彈 | [E] 商店</div>
    </div>

    <!-- ⏸️ 暫停與畫質設定選單 -->
    <div id="pause-menu">
        <h2 style="margin-top:0; color:#00ffff;">⏸️ 遊戲暫停</h2>
        <p style="color:#ccc; font-size:14px;">調整畫質設定：</p>
        <button class="btn quality-btn" id="q-low" onclick="setQuality('low')">低畫質 (無特效、流暢首選)</button>
        <button class="btn quality-btn active" id="q-mid" onclick="setQuality('mid')">中畫質 (預設特效)</button>
        <button class="btn quality-btn" id="q-high" onclick="setQuality('high')">高畫質 (火燒、炸彈爆炸、電擊連線)</button>
        <br><br>
        <button class="btn" style="background: #0088cc; font-size: 16px; padding: 10px;" onclick="togglePauseMenu()">▶️ 繼續遊戲</button>
    </div>
    
    <div id="minimap-container">
        <canvas id="minimap" width="160" height="160"></canvas>
    </div>

    <div id="crosshair"></div>
    <div id="msg">戰鬥準備開始！</div>
    <div id="drop-msg">🎁 獲得幸運升級！</div>

    <div id="shop">
        <h2 style="font-size: 20px; margin-top:0;">🛒 軍火庫與屬性升級 (按 E 關閉)</h2>
        <div id="weapon-shop" style="margin-bottom: 10px; border-bottom: 1px solid #555; padding-bottom: 10px;">
            <h3>武器購買與屬性升級 (火/炸彈/電)</h3>
            
            <script>
                for(let i=1; i<=6; i++) {
                    document.write(`
                    <div class="weapon-row">
                        <button class="btn" id="buy-w${i}" style="flex:2.2;" onclick="buyWeapon(${i})">載入中...</button>
                        <button class="elem-btn elem-fire" onclick="upgradeElement(${i}, 'fire')">🔥火 Lv.<span id="lv-fire-${i}">0</span><br>(💰<span id="c-fire-${i}">500</span>)</button>
                        <button class="elem-btn elem-bomb" onclick="upgradeElement(${i}, 'bomb')">💥炸 Lv.<span id="lv-bomb-${i}">0</span><br>(💰<span id="c-bomb-${i}">500</span>)</button>
                        <button class="elem-btn elem-elec" onclick="upgradeElement(${i}, 'elec')">⚡電 Lv.<span id="lv-elec-${i}">0</span><br>(💰<span id="c-elec-${i}">500</span>)</button>
                    </div>
                    `);
                }
            </script>
        </div>

        <div>
            <h3>基礎能力強化</h3>
            <button class="btn" id="btn-up-hp" onclick="buyUpgrade('hp')">提升血量上限 (+25 HP) - 💰 <span id="cost-hp">50</span></button>
            <button class="btn" id="btn-up-dmg" onclick="buyUpgrade('dmg')">提升整體傷害 (+20%) - 💰 <span id="cost-dmg">75</span></button>
            <button class="btn" id="btn-up-speed" onclick="buyUpgrade('speed')">戰術機動加速 (+15%) - 💰 <span id="cost-speed">60</span></button>
            <button class="btn" id="btn-up-firerate" onclick="buyUpgrade('firerate')">⚡ 射速提升 (+15%) - 💰 <span id="cost-firerate">80</span></button>
            <button class="btn" id="btn-up-ammo" onclick="buyUpgrade('ammo')">📦 彈藥上限擴充 (+25%) - 💰 <span id="cost-ammo">70</span></button>
            <button class="btn" id="btn-up-autoReload" onclick="buyUpgrade('autoReload')">🤖 自動補彈系統 - 💰 3000</button>
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
let scene, camera, renderer, dirLight;
let hp = 100, maxHp = 100, gold = 0, wave = 1;
let totalZombiesInWave = 0, killedZombiesInWave = 0;
let moveSpeed = 0.18, damageMult = 1.0;
let fireRateMult = 1.0;

let graphicsQuality = 'mid'; // 'low', 'mid', 'high'

let difficulty = 'easy';
// ⚙️ 設定各難度的 Boss 傷害與小怪傷害 (以 0.1 秒算一次) — 已全部乘以 2
let diffMult = { hp: 1.0, speed: 1.0, initialZombies: 10, goldMult: 2.0, bossDmg: 20, normalDmg: 1.0, label: "🟢 簡單", color: "#00ff66" };

let yVelocity = 0;
const gravity = 0.015;
const jumpStrength = 0.38;
let isGrounded = true;

let lastReloadTime = 0;
let hasAutoReload = false;

let zombies = [], bullets = [], enemyBullets = [], particles = [], medkits = [], lightningBeams = [], expRings = [];
let keys = {};
let isPaused = false, isGameOver = false, isPauseMenuOpen = false;
let waveTransitioning = false;
let isMouseDown = false;
const MAP_SIZE = 40;

let minimapCanvas, minimapCtx;
let totalKills = 0, totalGoldEarned = 0;

const upgradeCosts = { hp: 50, dmg: 75, speed: 60, firerate: 80, ammo: 70 };

const weapons = {
    1: { name: "戰術手槍", maxAmmo: 12, ammo: 12, dmg: 35, baseFireRate: 250, fireRate: 250, auto: false, unlocked: true, price: 0, bulletColor: 0xffff00, elem: { fire: { lv: 0, cost: 500 }, bomb: { lv: 0, cost: 500 }, elec: { lv: 0, cost: 500 } } },
    2: { name: "戰術散彈槍", maxAmmo: 6, ammo: 6, dmg: 25, count: 6, baseFireRate: 750, fireRate: 750, auto: false, unlocked: false, price: 150, bulletColor: 0xffa500, elem: { fire: { lv: 0, cost: 500 }, bomb: { lv: 0, cost: 500 }, elec: { lv: 0, cost: 500 } } },
    3: { name: "突擊步槍", maxAmmo: 30, ammo: 30, dmg: 40, baseFireRate: 110, fireRate: 110, auto: true, unlocked: false, price: 300, bulletColor: 0xff4500, elem: { fire: { lv: 0, cost: 500 }, bomb: { lv: 0, cost: 500 }, elec: { lv: 0, cost: 500 } } },
    4: { name: "戰術衝鋒槍", maxAmmo: 45, ammo: 45, dmg: 22, baseFireRate: 60, fireRate: 60, auto: true, unlocked: false, price: 600, bulletColor: 0xffff00, elem: { fire: { lv: 0, cost: 500 }, bomb: { lv: 0, cost: 500 }, elec: { lv: 0, cost: 500 } } },
    5: { name: "離子電漿毀滅者", maxAmmo: 25, ammo: 25, dmg: 100, count: 2, baseFireRate: 140, fireRate: 140, auto: true, unlocked: false, price: 1200, bulletColor: 0x00ffff, elem: { fire: { lv: 0, cost: 500 }, bomb: { lv: 0, cost: 500 }, elec: { lv: 0, cost: 500 } } },
    6: { name: "快速離子電漿毀滅者", maxAmmo: 40, ammo: 40, dmg: 110, count: 2, baseFireRate: 93, fireRate: 93, auto: true, unlocked: false, price: 2500, bulletColor: 0x00ffaa, elem: { fire: { lv: 0, cost: 500 }, bomb: { lv: 0, cost: 500 }, elec: { lv: 0, cost: 500 } } }
};
let currentWeaponKey = 1;

// 🎯 選擇難度並載入對應傷害參數 (全部乘 2)
function selectDifficulty(diff) {
    difficulty = diff;
    if (diff === 'easy') {
        diffMult = { hp: 1.0, speed: 1.0, initialZombies: 10, goldMult: 2.0, bossDmg: 20, normalDmg: 1.0, label: "🟢 簡單", color: "#00ff66" };
    } else if (diff === 'medium') {
        diffMult = { hp: 1.5, speed: 1.1, initialZombies: 20, goldMult: 1.4, bossDmg: 40, normalDmg: 2.0, label: "🟡 中等", color: "#ffaa00" };
    } else if (diff === 'hard') {
        diffMult = { hp: 2.25, speed: 1.2, initialZombies: 30, goldMult: 0.9, bossDmg: 60, normalDmg: 3.0, label: "🔴 困難", color: "#ff3333" };
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

    dirLight = new THREE.DirectionalLight(0xffaa66, 1.5);
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
    
    document.body.addEventListener('click', (e) => {
        if (!isPaused && !isGameOver && e.target.tagName !== 'BUTTON') document.body.requestPointerLock();
    });
    
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

function setQuality(q) {
    graphicsQuality = q;
    document.querySelectorAll('.quality-btn').forEach(btn => btn.classList.remove('active'));
    document.getElementById(`q-${q}`).classList.add('active');

    if (q === 'low') {
        renderer.shadowMap.enabled = false;
        scene.fog = null;
        if (dirLight) dirLight.castShadow = false;
    } else {
        renderer.shadowMap.enabled = true;
        scene.fog = new THREE.FogExp2(0x0a0a12, 0.02);
        if (dirLight) dirLight.castShadow = true;
    }
}

function togglePauseMenu() {
    if (isGameOver) return;
    isPauseMenuOpen = !isPauseMenuOpen;
    isPaused = isPauseMenuOpen;
    isMouseDown = false;

    document.getElementById('pause-menu').style.display = isPauseMenuOpen ? 'block' : 'none';
    if (isPauseMenuOpen) {
        document.exitPointerLock();
    } else {
        document.body.requestPointerLock();
    }
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

function createZombieMesh(color, scale, eyeColor = 0x00ffcc, transparent = false, opacity = 1.0) {
    const group = new THREE.Group();
    const bodyMat = new THREE.MeshStandardMaterial({ color: color, roughness: 0.5, transparent: transparent, opacity: opacity });
    const eyeMat = new THREE.MeshBasicMaterial({ color: eyeColor, transparent: transparent, opacity: opacity });

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
    let isExploder = false;
    let isRanged = false;
    let transparent = false;
    let opacity = 1.0;

    let waveScalingHp = 1 + (wave - 1) * 0.08;
    let evoChance = Math.min(0.8, (wave - 1) * 0.08);

    if (forceBoss || (wave % 5 === 0 && Math.random() < 0.2)) {
        color = 0x800080; eyeColor = 0xff0055; baseSpeed = 0.022; baseHp = 600; scale = 2.4; isBoss = true; isRanged = true;
    } else if (Math.random() < evoChance) {
        let evos = ['speed', 'tank', 'toxic'];
        if (wave >= 8) evos.push('void');
        if (wave >= 9) evos.push('exploder');
        if (wave >= 11) evos.push('ghost');
        if (wave >= 13) evos.push('titan');
        if (wave >= 14) evos.push('plasma');
        if (wave >= 17) evos.push('plague');

        let chosenEvo = evos[Math.floor(Math.random() * evos.length)];

        if (chosenEvo === 'speed') {
            color = 0xffaa00; eyeColor = 0xffff00; baseSpeed = 0.11 + (wave * 0.003); baseHp = 45; scale = 0.95;
        } else if (chosenEvo === 'tank') {
            color = 0x334455; eyeColor = 0x00ffff; baseSpeed = 0.03; baseHp = 180; scale = 1.5;
        } else if (chosenEvo === 'toxic') {
            color = 0x00ff55; eyeColor = 0xff00ff; baseSpeed = 0.07; baseHp = 70; scale = 1.1;
        } else if (chosenEvo === 'void') {
            color = 0x8800ff; eyeColor = 0xffffff; baseSpeed = 0.10; baseHp = 250; scale = 1.3;
        } else if (chosenEvo === 'exploder') {
            color = 0xff3300; eyeColor = 0xffff00; baseSpeed = 0.09; baseHp = 80; scale = 1.05; isExploder = true;
        } else if (chosenEvo === 'ghost') {
            color = 0xaaaaaa; eyeColor = 0x00ffff; baseSpeed = 0.12; baseHp = 110; scale = 0.9; transparent = true; opacity = 0.35;
        } else if (chosenEvo === 'titan') {
            color = 0x111122; eyeColor = 0xff0000; baseSpeed = 0.02; baseHp = 500; scale = 2.0;
        } else if (chosenEvo === 'plasma') {
            color = 0x00e5ff; eyeColor = 0xffffff; baseSpeed = 0.13; baseHp = 160; scale = 1.1;
        } else if (chosenEvo === 'plague') {
            color = 0x4a0e4e; eyeColor = 0x00ff00; baseSpeed = 0.04; baseHp = 450; scale = 1.8; isRanged = true;
        }
    } else {
        baseSpeed += Math.min(0.04, wave * 0.001);
    }

    let speed = baseSpeed * diffMult.speed;
    let zHp = baseHp * diffMult.hp * waveScalingHp;

    const mesh = createZombieMesh(color, scale, eyeColor, transparent, opacity);
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
        mesh, hp: zHp, maxHp: zHp, speed, scale, color, isBoss, isExploder, isRanged,
        height: 1.8 * scale,
        strafeDir: (Math.random() < 0.5 ? 1 : -1),
        strafeTimer: Math.floor(Math.random() * 60),
        lastShootTime: Date.now(),
        burnTimer: 0,
        burnDmg: 0
    });
}

function zombieShoot(zombie) {
    const startPos = zombie.mesh.position.clone().add(new THREE.Vector3(0, 1.2 * zombie.scale, 0));
    const targetPos = camera.position.clone();
    const dir = new THREE.Vector3().subVectors(targetPos, startPos).normalize();

    const geo = new THREE.SphereGeometry(zombie.isBoss ? 0.3 : 0.2);
    const mat = new THREE.MeshBasicMaterial({ color: zombie.isBoss ? 0xff0055 : 0x00ff00 });
    const bullet = new THREE.Mesh(geo, mat);
    bullet.position.copy(startPos);

    enemyBullets.push({ mesh: bullet, dir, speed: zombie.isBoss ? 0.35 : 0.28, life: 120, damage: zombie.isBoss ? diffMult.bossDmg : (diffMult.bossDmg * 0.5) });
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
    let count = diffMult.initialZombies;
    for (let i = 1; i < wave; i++) {
        count = Math.floor(count * 1.17 + 3);
    }
    totalZombiesInWave = count;
    killedZombiesInWave = 0;
    updateUI();

    let spawned = 0;
    if (wave % 5 === 0) {
        spawnZombie(true);
        spawned++;
    }

    let maxTotalSpawnTime = 20000;
    let baseInterval = Math.max(120, 700 - (wave * 30));
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

function createHitParticles(pos, colorHex, count = 6) {
    if (graphicsQuality === 'low') return;
    for (let i = 0; i < count; i++) {
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

// ⚡ 高畫質閃電連結雷射
function createLightningBeam(pos1, pos2) {
    const distance = pos1.distanceTo(pos2);
    const geo = new THREE.CylinderGeometry(0.05, 0.05, distance, 8);
    const mat = new THREE.MeshBasicMaterial({ color: 0x00ffff });
    const beam = new THREE.Mesh(geo, mat);

    const midPoint = new THREE.Vector3().addVectors(pos1, pos2).multiplyScalar(0.5);
    beam.position.copy(midPoint);

    beam.lookAt(pos2);
    beam.rotateX(Math.PI / 2);

    scene.add(beam);
    lightningBeams.push({ mesh: beam, life: 6 });
}

// 💥 高畫質爆炸光環效果
function createExplosionRing(pos, maxRadius) {
    const geo = new THREE.RingGeometry(0.1, 0.2, 32);
    const mat = new THREE.MeshBasicMaterial({ color: 0xffaa00, side: THREE.DoubleSide, transparent: true, opacity: 0.8 });
    const ring = new THREE.Mesh(geo, mat);
    ring.rotation.x = -Math.PI / 2;
    ring.position.copy(pos);
    ring.position.y = 0.1;

    scene.add(ring);
    expRings.push({ mesh: ring, radius: 0.1, maxRadius: maxRadius, opacity: 0.8 });
}

function shoot() {
    const w = weapons[currentWeaponKey];
    if (!w.unlocked) return;
    const now = Date.now();
    if (now - (w.lastShot || 0) < w.fireRate || w.ammo <= 0) return;
    
    w.lastShot = now;
    w.ammo--;
    updateUI();

    let bColor = w.bulletColor;
    if (w.elem.elec.lv > 0) bColor = 0x00ffff;
    else if (w.elem.bomb.lv > 0) bColor = 0xffaa00;
    else if (w.elem.fire.lv > 0) bColor = 0xff3300;

    const createBullet = (dirOffset = new THREE.Vector3()) => {
        const geo = new THREE.SphereGeometry((currentWeaponKey === 5 || currentWeaponKey === 6) ? 0.2 : 0.08);
        const mat = new THREE.MeshBasicMaterial({ color: bColor });
        const bullet = new THREE.Mesh(geo, mat);
        bullet.position.copy(camera.position).add(new THREE.Vector3(0, -0.2, 0));

        const dir = new THREE.Vector3(0, 0, -1).applyQuaternion(camera.quaternion).add(dirOffset).normalize();
        bullets.push({ 
            mesh: bullet, 
            dir: dir, 
            damage: w.dmg * damageMult, 
            life: 60, 
            color: bColor, 
            elem: {
                fireLv: w.elem.fire.lv,
                bombLv: w.elem.bomb.lv,
                elecLv: w.elem.elec.lv
            }
        });
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
        let bonusGold = Math.floor((200 + wave * 50) * diffMult.goldMult);
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
    if (isGameOver || isPauseMenuOpen) return;
    isPaused = !isPaused;
    isMouseDown = false;
    document.getElementById('shop').style.display = isPaused ? 'block' : 'none';
    if (isPaused) {
        document.exitPointerLock();
        updateShopUI();
    } else {
        document.body.requestPointerLock();
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

function upgradeElement(key, type) {
    const w = weapons[key];
    if (!w.unlocked) {
        alert("請先購買解鎖該武器！");
        return;
    }

    const elemData = w.elem[type];
    if (gold >= elemData.cost) {
        gold -= elemData.cost;
        elemData.lv++;
        elemData.cost = Math.floor(elemData.cost * 1.45);

        let typeName = type === 'fire' ? '🔥 燃燒' : (type === 'bomb' ? '💥 爆炸' : '⚡ 電擊');
        showMsg(`⚡ ${w.name} 的 ${typeName} 屬性升級至 Lv.${elemData.lv}！`);
        updateUI();
        updateShopUI();
    } else {
        alert("金幣不足以升級該屬性！");
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
        upgradeCosts[type] = Math.floor(cost * 1.45);
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
                btn.innerText = `購買 ${weapons[k].name} - 💰 ${weapons[k].price}`;
                btn.disabled = gold < weapons[k].price;
            }
        }

        ['fire', 'bomb', 'elec'].forEach(type => {
            const elemData = weapons[k].elem[type];
            const lvSpan = document.getElementById(`lv-${type}-${k}`);
            const costSpan = document.getElementById(`c-${type}-${k}`);
            if(lvSpan) lvSpan.innerText = elemData.lv;
            if(costSpan) costSpan.innerText = elemData.cost;
        });
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
    let elemTags = [];
    if(w.elem.fire.lv > 0) elemTags.push(`🔥Lv.${w.elem.fire.lv}`);
    if(w.elem.bomb.lv > 0) elemTags.push(`💥Lv.${w.elem.bomb.lv}`);
    if(w.elem.elec.lv > 0) elemTags.push(`⚡Lv.${w.elem.elec.lv}`);

    let str = elemTags.length > 0 ? ` [${elemTags.join(' ')}]` : '';
    document.getElementById('weapon').innerText = w.name + str;
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

function triggerElementEffect(hitZombie, bullet) {
    if (bullet.elem.elecLv > 0) {
        let maxTargets = 2 + bullet.elem.elecLv;
        let chainDmg = bullet.damage * (0.4 + bullet.elem.elecLv * 0.15);
        let count = 0;
        zombies.forEach(z => {
            if (z !== hitZombie && count < maxTargets && z.mesh.position.distanceTo(hitZombie.mesh.position) < (5.0 + bullet.elem.elecLv)) {
                z.hp -= chainDmg;
                createHitParticles(z.mesh.position, 0x00ffff);
                
                // ⚡ 高畫質模式下的閃電連線效果
                if (graphicsQuality === 'high') {
                    createLightningBeam(hitZombie.mesh.position.clone().add(new THREE.Vector3(0, 1, 0)), z.mesh.position.clone().add(new THREE.Vector3(0, 1, 0)));
                }
                count++;
            }
        });
    }

    if (bullet.elem.bombLv > 0) {
        let radius = 3.0 + (bullet.elem.bombLv * 0.8);
        let bombDmg = bullet.damage * (0.3 + bullet.elem.bombLv * 0.2);
        createHitParticles(bullet.mesh.position, 0xffaa00, 15);
        
        // 💥 高畫質模式下的擴散爆炸光環
        if (graphicsQuality === 'high') {
            createExplosionRing(bullet.mesh.position.clone(), radius);
        }

        zombies.forEach(z => {
            if (z.mesh.position.distanceTo(bullet.mesh.position) < radius) {
                z.hp -= bombDmg;
            }
        });
    }

    if (bullet.elem.fireLv > 0) {
        hitZombie.burnTimer = 3 + bullet.elem.fireLv;
        hitZombie.burnDmg = bullet.damage * (0.1 + bullet.elem.fireLv * 0.08);
    }
}

function handleZombieDeath(z) {
    if (z.isExploder) {
        createHitParticles(z.mesh.position, 0xff3300, 20);
        if (graphicsQuality === 'high') createExplosionRing(z.mesh.position.clone(), 4.0);
        if (camera.position.distanceTo(z.mesh.position) < 4.0) {
            hp -= diffMult.normalDmg * 20;
            showMsg("💥 遭受自爆範圍傷害！");
        }
    }

    scene.remove(z.mesh);
    killedZombiesInWave++;
    totalKills++;
    
    let baseEarned = z.isBoss ? 250 : (25 + wave * 2);
    let earned = Math.floor(baseEarned * diffMult.goldMult);
    gold += earned;
    totalGoldEarned += earned;

    if (Math.random() < 0.01) {
        triggerLuckyDrop();
    }

    updateUI();
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

    // 清理高畫質電擊雷射 beam
    for (let i = lightningBeams.length - 1; i >= 0; i--) {
        let beam = lightningBeams[i];
        beam.life--;
        if (beam.life <= 0) {
            scene.remove(beam.mesh);
            lightningBeams.splice(i, 1);
        }
    }

    // 清理高畫質爆炸環
    for (let i = expRings.length - 1; i >= 0; i--) {
        let ring = expRings[i];
        ring.radius += (ring.maxRadius - ring.radius) * 0.2;
        ring.mesh.scale.set(ring.radius, ring.radius, 1);
        ring.opacity -= 0.08;
        ring.mesh.material.opacity = ring.opacity;

        if (ring.opacity <= 0) {
            scene.remove(ring.mesh);
            expRings.splice(i, 1);
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
                triggerElementEffect(z, b);
                
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
                    handleZombieDeath(z);
                    zombies.splice(j, 1);
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

        if (z.burnTimer > 0) {
            z.burnTimer -= 0.05;
            z.hp -= z.burnDmg * 0.05;
            
            // 🔥 高畫質模式下的燃燒冒煙/火花效果
            if (graphicsQuality === 'high') {
                createHitParticles(z.mesh.position.clone().add(new THREE.Vector3(0, Math.random() * z.scale, 0)), 0xff3300, 2);
            } else if (graphicsQuality === 'mid' && Math.random() < 0.3) {
                createHitParticles(z.mesh.position, 0xff3300, 1);
            }

            if (z.hp <= 0) {
                handleZombieDeath(z);
                zombies.splice(i, 1);
                continue;
            }
        }
        
        let forward = new THREE.Vector3().subVectors(camera.position, z.mesh.position);
        let horizontalDist = Math.sqrt(forward.x * forward.x + forward.z * forward.z);
        let playerFootY = camera.position.y - 1.7;
        let isPlayerAboveEnemy = playerFootY > (z.height - 0.2);

        forward.y = 0;
        forward.normalize();

        if (z.isRanged && Date.now() - z.lastShootTime > (z.isBoss ? 2500 : 3500)) {
            z.lastShootTime = Date.now();
            zombieShoot(z);
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

        // ⚔️ 碰撞傷害： Boss 或小怪扣除對應數量的傷害
        if (horizontalDist <= attackRange + 0.3 && !isPlayerAboveEnemy) {
            let dmgPerFrame = z.isBoss ? (diffMult.bossDmg / 6) : (diffMult.normalDmg / 6);
            hp -= dmgPerFrame;
            updateUI();
            
            const flash = document.getElementById('damage-flash');
            flash.style.display = 'block';
            setTimeout(() => flash.style.display = 'none', 30);

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
