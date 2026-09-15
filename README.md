<!DOCTYPE html>
<html lang="zh-TW">
<head>
    <meta charset="UTF-8">
    <title>屍境重構：廢墟決戰 v7.1</title>
    <style>
        body { margin: 0; overflow: hidden; font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif; user-select: none; background: #000; }
        #ui { position: absolute; top: 15px; left: 15px; color: #fff; text-shadow: 2px 2px 4px #000; font-size: 18px; pointer-events: none; z-index: 10; }
        #crosshair { position: absolute; top: 50%; left: 50%; width: 14px; height: 14px; border: 2px solid rgba(255,255,255,0.8); border-radius: 50%; transform: translate(-50%, -50%); pointer-events: none; z-index: 10; }
        #crosshair::after { content: ''; position: absolute; top: 4px; left: 4px; width: 2px; height: 2px; background: red; }
        
        /* 左下角矩形全地圖雷達 */
        #minimap-container {
            position: absolute;
            bottom: 20px;
            left: 20px;
            width: 180px;
            height: 180px;
            background: rgba(10, 15, 25, 0.85);
            border: 2px solid #00ffff;
            border-radius: 8px;
            box-shadow: 0 0 15px rgba(0, 255, 255, 0.4);
            pointer-events: none;
            z-index: 10;
            overflow: hidden;
        }
        #minimap {
            width: 100%;
            height: 100%;
        }

        /* 商店樣式 */
        #shop { position: absolute; top: 50%; left: 50%; transform: translate(-50%, -50%); background: rgba(15, 15, 25, 0.95); color: white; padding: 25px; border-radius: 12px; display: none; text-align: center; border: 2px solid #ff4444; box-shadow: 0 0 20px rgba(255,68,68,0.4); z-index: 20; max-height: 80vh; overflow-y: auto; }
        
        /* 遊戲結束結算畫面 */
        #game-over-screen { position: absolute; top: 50%; left: 50%; transform: translate(-50%, -50%); background: rgba(20, 0, 0, 0.95); color: white; padding: 35px; border-radius: 15px; display: none; text-align: center; border: 3px solid #ff0000; box-shadow: 0 0 30px rgba(255,0,0,0.8); z-index: 30; min-width: 320px; }
        .stats-box { background: rgba(255,255,255,0.08); margin: 15px 0; padding: 15px; border-radius: 8px; text-align: left; line-height: 1.8; }
        
        .btn { background: #222; color: white; border: 1px solid #ff4444; padding: 10px 20px; margin: 5px; cursor: pointer; font-size: 15px; border-radius: 6px; transition: 0.2s; }
        .btn:hover { background: #ff4444; color: black; font-weight: bold; }
        .btn:disabled { background: #444; border-color: #666; color: #aaa; cursor: not-allowed; }
        #msg { position: absolute; top: 25%; width: 100%; text-align: center; color: #ffeb3b; font-size: 32px; font-weight: bold; text-shadow: 3px 3px 6px #000; pointer-events: none; display: none; z-index: 10; }
        #damage-flash { position: absolute; top: 0; left: 0; width: 100vw; height: 100vh; background: rgba(255,0,0,0.3); pointer-events: none; display: none; z-index: 5; }
    </style>
    <script src="https://cdnjs.cloudflare.com/ajax/libs/three.js/r128/three.min.js"></script>
</head>
<body>

    <div id="damage-flash"></div>
    <div id="ui">
        <div>❤️ 血量: <span id="hp" style="color: #ff5555; font-weight: bold;">100</span></div>
        <div>💰 金幣: <span id="gold" style="color: #ffd700; font-weight: bold;">0</span></div>
        <div>🌊 當前波次: <span id="wave" style="color: #00ffff; font-weight: bold;">1</span></div>
        <div>👾 剩餘敵人: <span id="zombie-count" style="color: #ff4444; font-weight: bold;">0</span></div>
        <div>🔫 武器: <span id="weapon">戰術手槍</span> (<span id="ammo">12/12</span>)</div>
        <div style="font-size:13px; color:#aaa; margin-top:8px;">[WASD] 移動 | [按住左鍵] 自動連射 | [1-5] 切換武器 | [R] 換彈 | [E] 商店</div>
    </div>
    
    <!-- 左下角矩形小地圖 -->
    <div id="minimap-container">
        <canvas id="minimap" width="180" height="180"></canvas>
    </div>

    <div id="crosshair"></div>
    <div id="msg">戰鬥準備開始！</div>

    <!-- 商店 UI -->
    <div id="shop">
        <h2>🛒 軍火庫與技能升級 (按 E 關閉)</h2>
        <div id="weapon-shop" style="margin-bottom: 15px; border-bottom: 1px solid #555; padding-bottom: 15px;">
            <h3>解鎖新武器</h3>
            <button class="btn" id="buy-w2" onclick="buyWeapon(2)">購買 戰術散彈槍 - 💰 150</button><br>
            <button class="btn" id="buy-w3" onclick="buyWeapon(3)">購買 突擊步槍 - 💰 300</button><br>
            <button class="btn" id="buy-w4" onclick="buyWeapon(4)">購買 戰術衝鋒槍 (極高射速) - 💰 600</button><br>
            <button class="btn" id="buy-w5" onclick="buyWeapon(5)">購買 離子電漿毀滅者 - 💰 1200</button>
        </div>
        <div>
            <h3>能力強化 (每級費用增加 50%)</h3>
            <button class="btn" id="btn-up-hp" onclick="buyUpgrade('hp')">提升血量上限 (+25 HP) - 💰 <span id="cost-hp">50</span></button><br>
            <button class="btn" id="btn-up-dmg" onclick="buyUpgrade('dmg')">提升整體傷害 (+20%) - 💰 <span id="cost-dmg">75</span></button><br>
            <button class="btn" id="btn-up-speed" onclick="buyUpgrade('speed')">戰術機動加速 (+15%) - 💰 <span id="cost-speed">60</span></button>
        </div>
        <br>
        <button class="btn" onclick="toggleShop()" style="background: #444;">返回戰場</button>
    </div>

    <!-- 遊戲結束結算畫面 UI -->
    <div id="game-over-screen">
        <h1 style="color: #ff3333; margin: 0;">💀 陣亡通知</h1>
        <p>你已經被怪物大軍吞噬...</p>
        <div class="stats-box">
            📊 <b>本局最終數據統計：</b><br>
            🌊 生存波次: <span id="stat-wave" style="color:#00ffff;">0</span><br>
            👾 總擊殺數: <span id="stat-kills" style="color:#ff5555;">0</span><br>
            💰 累計獲得金幣: <span id="stat-gold" style="color:#ffd700;">0</span><br>
            🎯 總輸出傷害: <span id="stat-damage" style="color:#ff9900;">0</span>
        </div>
        <button class="btn" onclick="location.reload()" style="font-size: 18px; padding: 12px 30px; background: #ff4444; color: white;">🔄 重新開始遊戲</button>
    </div>

<script>
let scene, camera, renderer;
let hp = 100, maxHp = 100, gold = 0, wave = 1;
let totalZombiesInWave = 0, killedZombiesInWave = 0;
let moveSpeed = 0.18, damageMult = 1.0;
let zombies = [], bullets = [], particles = [], medkits = [];
let keys = {};
let isPaused = false, isGameOver = false;
let waveTransitioning = false;
let isMouseDown = false;
const MAP_SIZE = 40;

// 小地圖 Canvas
let minimapCanvas, minimapCtx;

// 統計數據
let totalKills = 0, totalGoldEarned = 0, totalDamageDealt = 0;

// 升級價格與遞增係數
const upgradeCosts = { hp: 50, dmg: 75, speed: 60 };

// 武器庫設定
const weapons = {
    1: { name: "戰術手槍", maxAmmo: 12, ammo: 12, dmg: 35, fireRate: 250, auto: false, unlocked: true, price: 0, bulletColor: 0xffff00 },
    2: { name: "戰術散彈槍", maxAmmo: 6, ammo: 6, dmg: 25, count: 6, fireRate: 750, auto: false, unlocked: false, price: 150, bulletColor: 0xffa500 },
    3: { name: "突擊步槍", maxAmmo: 30, ammo: 30, dmg: 40, fireRate: 110, auto: true, unlocked: false, price: 300, bulletColor: 0xff4500 },
    4: { name: "戰術衝鋒槍", maxAmmo: 45, ammo: 45, dmg: 22, fireRate: 60, auto: true, unlocked: false, price: 600, bulletColor: 0xffff00 },
    5: { name: "離子電漿毀滅者", maxAmmo: 25, ammo: 25, dmg: 100, count: 2, fireRate: 140, auto: true, unlocked: false, price: 1200, bulletColor: 0x00ffff }
};
let currentWeaponKey = 1;

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

    // 初始化小地圖 Canvas
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
        keys[e.key.toLowerCase()] = true;
        if(e.key === 'e' || e.key === 'E') toggleShop();
        if(['1','2','3','4','5'].includes(e.key)) switchWeapon(parseInt(e.key));
        if(e.key.toLowerCase() === 'r') reloadAmmo();
    });
    document.addEventListener('keyup', (e) => keys[e.key.toLowerCase()] = false);
    
    document.body.addEventListener('click', () => {
        if (!isPaused && !isGameOver) document.body.requestPointerLock();
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

    // 定時在隨機位置生成醫藥包 (每 12 秒檢測一次，上限 3 個)
    setInterval(spawnMedkit, 12000);
    spawnMedkit(); // 遊戲開始先生成 1 個

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

// 建立 3D 醫藥包模型
function createMedkitMesh() {
    const group = new THREE.Group();
    
    // 白盒本體
    const boxGeo = new THREE.BoxGeometry(0.8, 0.5, 0.6);
    const boxMat = new THREE.MeshStandardMaterial({ color: 0xffffff, roughness: 0.3 });
    const box = new THREE.Mesh(boxGeo, boxMat);
    box.position.y = 0.25;
    box.castShadow = true;
    group.add(box);

    // 綠十字標誌
    const crossMat = new THREE.MeshBasicMaterial({ color: 0x00ff66 });
    const c1 = new THREE.Mesh(new THREE.BoxGeometry(0.4, 0.52, 0.12), crossMat);
    const c2 = new THREE.Mesh(new THREE.BoxGeometry(0.12, 0.52, 0.4), crossMat);
    c1.position.y = 0.25;
    c2.position.y = 0.25;
    group.add(c1); group.add(c2);

    return group;
}

// 在地圖上隨機生成醫藥包 (上限最多 3 個)
function spawnMedkit() {
    if (medkits.length >= 3 || isGameOver) return;

    const mesh = createMedkitMesh();
    const rx = (Math.random() - 0.5) * (MAP_SIZE * 2 - 10);
    const rz = (Math.random() - 0.5) * (MAP_SIZE * 2 - 10);
    mesh.position.set(rx, 0, rz);

    scene.add(mesh);
    // 固定回復 20 血量
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

function spawnZombie() {
    let type = Math.random();
    let color = 0x2d5a27;
    let eyeColor = 0x00ffcc;
    let speed = 0.045;
    let zHp = 60 + (wave * 15);
    let scale = 1.0;
    let isBoss = false;

    if (wave >= 5 && Math.random() < 0.15) {
        color = 0x4b0082; eyeColor = 0xff00ff; speed = 0.02; zHp = 500 + (wave * 80); scale = 2.0; isBoss = true;
    } else if (wave >= 3 && type < 0.3) {
        color = 0xccff00; eyeColor = 0xff0000; speed = 0.12; zHp = 50 + (wave * 10); scale = 0.9;
    } else if (type < 0.5) {
        color = 0x8b0000; speed = 0.095; zHp = 45 + (wave * 10);
    } else if (type < 0.75) {
        color = 0x223355; speed = 0.025; zHp = 200 + (wave * 40); scale = 1.4;
    }

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
        strafeDir: (Math.random() < 0.5 ? 1 : -1),
        strafeTimer: Math.floor(Math.random() * 60)
    });
}

function startNextWaveCountdown() {
    waveTransitioning = true;
    let countdown = 3;
    showMsg(`🎉 本波清掃完畢！ ${countdown} 秒後開始第 ${wave} 波`);
    
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
    totalZombiesInWave = 5 + wave * 4;
    killedZombiesInWave = 0;
    updateUI();

    let spawned = 0;
    let timer = setInterval(() => {
        if (spawned < totalZombiesInWave && !isPaused && !isGameOver) {
            spawnZombie();
            spawned++;
        } else if (spawned >= totalZombiesInWave) {
            clearInterval(timer);
        }
    }, 700);
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
    if (now - w.lastShot < w.fireRate || w.ammo <= 0) return;
    
    w.lastShot = now;
    w.ammo--;
    updateUI();

    const createBullet = (dirOffset = new THREE.Vector3()) => {
        const geo = new THREE.SphereGeometry(currentWeaponKey === 5 ? 0.2 : 0.08);
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

function reloadAmmo() {
    const w = weapons[currentWeaponKey];
    if (w.unlocked) {
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
        currentWeaponKey = key;
        showMsg(`解鎖新武器: ${w.name}！`);
        updateUI();
        updateShopUI();
    } else if (gold < w.price) {
        alert("金幣不足！");
    }
}

function buyUpgrade(type) {
    const cost = upgradeCosts[type];
    if (gold >= cost) {
        gold -= cost;
        if (type === 'hp') {
            maxHp += 25; hp += 25;
        } else if (type === 'dmg') {
            damageMult += 0.20;
        } else if (type === 'speed') {
            moveSpeed += 0.03;
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

// 繪製左下角矩形小地圖雷達
function drawMinimap() {
    const width = minimapCanvas.width;
    const height = minimapCanvas.height;
    const padding = 10;
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

    // 繪製醫藥包（綠點）
    medkits.forEach(m => {
        const p = mapToCanvas(m.mesh.position.x, m.mesh.position.z);
        minimapCtx.fillStyle = '#00ff66';
        minimapCtx.beginPath();
        minimapCtx.arc(p.x, p.y, 3.5, 0, Math.PI * 2);
        minimapCtx.fill();
    });

    // 繪製敵人紅點
    zombies.forEach(z => {
        const p = mapToCanvas(z.mesh.position.x, z.mesh.position.z);
        minimapCtx.fillStyle = z.isBoss ? '#ff00ff' : '#ff3333';
        minimapCtx.beginPath();
        minimapCtx.arc(p.x, p.y, z.isBoss ? 4.5 : 2.5, 0, Math.PI * 2);
        minimapCtx.fill();
    });

    // 繪製玩家位置與朝向
    const pPlayer = mapToCanvas(camera.position.x, camera.position.z);

    minimapCtx.fillStyle = '#ffffff';
    minimapCtx.beginPath();
    minimapCtx.arc(pPlayer.x, pPlayer.y, 3.5, 0, Math.PI * 2);
    minimapCtx.fill();

    const forward = new THREE.Vector3(0, 0, -1).applyQuaternion(camera.quaternion);
    const lineLen = 14;
    minimapCtx.strokeStyle = '#ffffff';
    minimapCtx.lineWidth = 2;
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
    document.getElementById('stat-damage').innerText = Math.floor(totalDamageDealt);
    
    document.getElementById('game-over-screen').style.display = 'block';
}

function animate() {
    requestAnimationFrame(animate);
    if (isPaused || isGameOver) return;

    if (isMouseDown && weapons[currentWeaponKey].auto) {
        shoot();
    }

    // 1. 移動與邊界限制
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

    // 2. 醫藥包動態與觸碰拾取 (+20 血量)
    for (let i = medkits.length - 1; i >= 0; i--) {
        let m = medkits[i];
        m.mesh.rotation.y += 0.02;
        m.mesh.position.y = 0.2 + Math.sin(Date.now() * 0.003) * 0.1;

        if (camera.position.distanceTo(m.mesh.position) < 1.5) {
            hp = Math.min(maxHp, hp + m.healAmount);
            showMsg(`💚 拾取醫藥包，回復了 ${m.healAmount} 點血量！`);
            updateUI();
            
            scene.remove(m.mesh);
            medkits.splice(i, 1);
        }
    }

    // 3. 子彈擊中判定
    for (let i = bullets.length - 1; i >= 0; i--) {
        let b = bullets[i];
        b.mesh.position.addScaledVector(b.dir, 0.9);
        b.life--;

        for (let j = zombies.length - 1; j >= 0; j--) {
            let z = zombies[j];
            if (b.mesh.position.distanceTo(z.mesh.position.clone().add(new THREE.Vector3(0, 1, 0))) < 0.9 * z.scale) {
                z.hp -= b.damage;
                totalDamageDealt += b.damage;
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
                    
                    let earned = z.isBoss ? 150 : 25 + wave * 2;
                    gold += earned;
                    totalGoldEarned += earned;
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

    // 4. 粒子更新
    for (let i = particles.length - 1; i >= 0; i--) {
        let p = particles[i];
        p.mesh.position.add(p.vel);
        p.life--;
        if (p.life <= 0) {
            scene.remove(p.mesh);
            particles.splice(i, 1);
        }
    }

    // 5. 殭屍 AI
    for (let i = zombies.length - 1; i >= 0; i--) {
        let z = zombies[i];
        
        let forward = new THREE.Vector3().subVectors(camera.position, z.mesh.position);
        forward.y = 0;
        let distToPlayer = forward.length();
        forward.normalize();

        z.strafeTimer--;
        if (z.strafeTimer <= 0) {
            z.strafeDir = Math.random() < 0.5 ? 1 : -1;
            z.strafeTimer = 30 + Math.floor(Math.random() * 60);
        }
        let sideDir = new THREE.Vector3(-forward.z, 0, forward.x).multiplyScalar(z.strafeDir);
        let attackRange = 1.8 * z.scale;

        if (distToPlayer > attackRange) {
            let moveDir = new THREE.Vector3()
                .addScaledVector(forward, 0.85)
                .addScaledVector(sideDir, 0.3)
                .normalize();

            z.mesh.position.addScaledVector(moveDir, z.speed);
        }
        
        z.mesh.lookAt(camera.position.x, 0, camera.position.z);

        if (distToPlayer <= attackRange + 0.3) {
            hp -= z.isBoss ? 1.2 : 0.4;
            updateUI();
            
            const flash = document.getElementById('damage-flash');
            flash.style.display = 'block';
            setTimeout(() => flash.style.display = 'none', 50);

            if (hp <= 0) {
                gameOver();
            }
        }
    }

    // 6. 波次推進
    if (killedZombiesInWave >= totalZombiesInWave && totalZombiesInWave > 0 && !waveTransitioning) {
        wave++;
        startNextWaveCountdown();
    }

    // 7. 即時繪製小地圖
    drawMinimap();

    renderer.render(scene, camera);
}

window.addEventListener('resize', () => {
    camera.aspect = window.innerWidth / window.innerHeight;
    camera.updateProjectionMatrix();
    renderer.setSize(window.innerWidth, window.innerHeight);
});

init();
</script>
</body>
</html>
