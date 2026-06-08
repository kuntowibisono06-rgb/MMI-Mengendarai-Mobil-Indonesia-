<!DOCTYPE html>
<html lang="id">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0, user-scalable=no">
    <title>Realistic Driving Manual - Transmisi & Kontrol Mobil</title>
    <style>
        body {
            margin: 0;
            overflow: hidden;
            font-family: 'Segoe UI', 'Arial', sans-serif;
        }
        #info {
            position: absolute;
            top: 20px;
            left: 20px;
            background: rgba(0,0,0,0.75);
            color: white;
            padding: 12px 20px;
            border-radius: 12px;
            backdrop-filter: blur(8px);
            pointer-events: none;
            z-index: 10;
            border-left: 5px solid #ff5500;
            box-shadow: 0 4px 15px rgba(0,0,0,0.3);
            font-size: 14px;
        }
        #dashboard {
            position: absolute;
            bottom: 20px;
            left: 20px;
            right: 20px;
            background: rgba(0,0,0,0.85);
            color: #0ff;
            padding: 12px 20px;
            border-radius: 20px;
            backdrop-filter: blur(10px);
            display: flex;
            justify-content: space-between;
            font-family: monospace;
            font-weight: bold;
            font-size: 1.5rem;
            letter-spacing: 1px;
            z-index: 10;
            border-top: 2px solid #ffaa33;
            border-bottom: 2px solid #ffaa33;
            pointer-events: none;
            text-shadow: 0 0 3px black;
        }
        .panel {
            background: #1e1e1ecc;
            padding: 6px 16px;
            border-radius: 12px;
            text-align: center;
        }
        .speed-value {
            color: #ffaa33;
            font-size: 2rem;
            font-weight: bold;
        }
        .gear-value {
            color: #ff5500;
            font-size: 2.2rem;
            font-weight: bold;
            background: #000000aa;
            padding: 0 12px;
            border-radius: 12px;
        }
        .controls-hint {
            position: absolute;
            bottom: 20px;
            right: 20px;
            background: rgba(0,0,0,0.6);
            color: #ccc;
            padding: 10px 15px;
            border-radius: 12px;
            font-size: 12px;
            font-family: monospace;
            text-align: right;
            pointer-events: none;
            z-index: 10;
            backdrop-filter: blur(4px);
        }
        @media (max-width: 700px) {
            .panel { font-size: 0.8rem; padding: 2px 8px; }
            .speed-value { font-size: 1.3rem; }
            .gear-value { font-size: 1.5rem; }
            #dashboard { padding: 8px 12px; }
            .controls-hint { font-size: 9px; }
        }
        .warning {
            color: #ff8888;
            font-size: 0.8rem;
        }
    </style>
</head>
<body>
    <div id="info">
        🚗 SIMULATOR MANUAL TRANSMISI | Realistis: Gigi mempengaruhi kecepatan maks & akselerasi<br>
        🎮 <strong>Stir (Arah)</strong>: ← → atau A/D &nbsp;&nbsp;|&nbsp;&nbsp;
        🚀 <strong>Gas</strong>: ↑ atau W &nbsp;&nbsp;|&nbsp;&nbsp;
        🛑 <strong>Rem</strong>: ↓ atau S <br>
        ⚙️ <strong>Naik Gigi</strong>: E &nbsp;&nbsp;|&nbsp;&nbsp;
        ⚙️ <strong>Turun Gigi</strong>: Q &nbsp;&nbsp;|&nbsp;&nbsp;
        🔄 <strong>Mundur (R)</strong>: Tekan R &nbsp;&nbsp;|&nbsp;&nbsp;
        ⚪ <strong>Netral (N)</strong>: Tekan N
    </div>
    <div id="dashboard">
        <div class="panel">🏎️ SPEED <br><span id="speedValue" class="speed-value">0</span> <span style="font-size:1rem;">km/h</span></div>
        <div class="panel">⚙️ TRANSMISI <br><span id="gearDisplay" class="gear-value">1</span></div>
        <div class="panel">📊 RPM <br><span id="rpmValue" style="color:#88ff88;">0</span> %</div>
    </div>
    <div class="controls-hint">
        [ E ] Naik Gigi &nbsp;&nbsp;| [ Q ] Turun Gigi <br>
        [ R ] Mundur &nbsp;&nbsp;| [ N ] Netral <br>
        Batas kecepatan per gigi: <span id="gearLimitHint">1:30 | 2:70 | 3:120 | 4:170 | 5:220 km/h</span>
    </div>

    <!-- Import Three.js core dan add-ons -->
    <script type="importmap">
        {
            "imports": {
                "three": "https://unpkg.com/three@0.128.0/build/three.module.js",
                "three/addons/": "https://unpkg.com/three@0.128.0/examples/jsm/"
            }
        }
    </script>

    <script type="module">
        import * as THREE from 'three';
        import { OrbitControls } from 'three/addons/controls/OrbitControls.js';
        import { CSS2DRenderer, CSS2DObject } from 'three/addons/renderers/CSS2DRenderer.js';

        // --- Inisialisasi Scene, Camera, Renderers ---
        const scene = new THREE.Scene();
        scene.background = new THREE.Color(0x071a3b);
        scene.fog = new THREE.FogExp2(0x071a3b, 0.003);
        
        // Kamera utama (Third Person)
        const camera = new THREE.PerspectiveCamera(55, window.innerWidth / window.innerHeight, 0.1, 1000);
        camera.position.set(0, 3.5, -8);
        
        // Renderer WebGL
        const renderer = new THREE.WebGLRenderer({ antialias: true });
        renderer.setSize(window.innerWidth, window.innerHeight);
        renderer.shadowMap.enabled = true; // bayangan realistis
        renderer.shadowMap.type = THREE.PCFSoftShadowMap;
        document.body.appendChild(renderer.domElement);
        
        // CSS2 Renderer untuk teks statis
        const labelRenderer = new CSS2DRenderer();
        labelRenderer.setSize(window.innerWidth, window.innerHeight);
        labelRenderer.domElement.style.position = 'absolute';
        labelRenderer.domElement.style.top = '0px';
        labelRenderer.domElement.style.left = '0px';
        labelRenderer.domElement.style.pointerEvents = 'none';
        document.body.appendChild(labelRenderer.domElement);
        
        // --- Pencahayaan Realistis ---
        // Ambient light
        const ambientLight = new THREE.AmbientLight(0x404060);
        scene.add(ambientLight);
        // Sun light utama
        const dirLight = new THREE.DirectionalLight(0xfff5e0, 1.2);
        dirLight.position.set(10, 20, 5);
        dirLight.castShadow = true;
        dirLight.receiveShadow = true;
        dirLight.shadow.mapSize.width = 1024;
        dirLight.shadow.mapSize.height = 1024;
        dirLight.shadow.camera.near = 0.5;
        dirLight.shadow.camera.far = 30;
        dirLight.shadow.camera.left = -10;
        dirLight.shadow.camera.right = 10;
        dirLight.shadow.camera.top = 10;
        dirLight.shadow.camera.bottom = -10;
        scene.add(dirLight);
        // Fill light belakang
        const backLight = new THREE.PointLight(0x4466cc, 0.4);
        backLight.position.set(-3, 2, -4);
        scene.add(backLight);
        // Rim light
        const rimLight = new THREE.PointLight(0xffaa66, 0.5);
        rimLight.position.set(2, 3, -3);
        scene.add(rimLight);
        
        // Ground / jalan dengan grid dan referensi
        const gridHelper = new THREE.GridHelper(200, 40, 0x88aaff, 0x335588);
        gridHelper.position.y = -0.05;
        gridHelper.material.transparent = true;
        gridHelper.material.opacity = 0.65;
        scene.add(gridHelper);
        
        // Lantai reflektif sederhana untuk bayangan
        const groundPlane = new THREE.Mesh(
            new THREE.PlaneGeometry(150, 150),
            new THREE.ShadowMaterial({ opacity: 0.4, color: 0x000000, transparent: true, side: THREE.DoubleSide })
        );
        groundPlane.rotation.x = -Math.PI / 2;
        groundPlane.position.y = -0.1;
        groundPlane.receiveShadow = true;
        scene.add(groundPlane);
        
        // Beberapa pohon / marker sederhana agar terasa kecepatan
        const addTree = (x, z) => {
            const group = new THREE.Group();
            const trunk = new THREE.Mesh(new THREE.CylinderGeometry(0.4, 0.5, 1.2), new THREE.MeshStandardMaterial({ color: 0x8B5A2B }));
            trunk.position.y = 0.6;
            trunk.castShadow = true;
            const foliage = new THREE.Mesh(new THREE.ConeGeometry(0.7, 1.2, 6), new THREE.MeshStandardMaterial({ color: 0x5C9E5E }));
            foliage.position.y = 1.2;
            foliage.castShadow = true;
            group.add(trunk, foliage);
            group.position.set(x, 0, z);
            scene.add(group);
        };
        for (let i = -40; i <= 40; i += 12) {
            for (let j = -30; j <= 30; j += 14) {
                if (Math.abs(i) < 8 && Math.abs(j) < 10) continue; // hindari spawn di depan mobil
                addTree(i + (Math.random() - 0.5) * 3, j + (Math.random() - 0.5) * 3);
            }
        }
        // garis tengah jalan sederhana dengan titik-titik
        const lineMat = new THREE.MeshStandardMaterial({ color: 0xffdd99, emissive: 0x442200 });
        for (let z = -60; z <= 60; z += 3) {
            const dash = new THREE.Mesh(new THREE.BoxGeometry(0.3, 0.05, 1.8), lineMat);
            dash.position.set(0, 0, z);
            dash.receiveShadow = true;
            scene.add(dash);
        }
        
        // --- Model Mobil (sederhana namun estetik) ---
        const carGroup = new THREE.Group();
        // Body
        const bodyGeo = new THREE.BoxGeometry(1.5, 0.5, 3.2);
        const bodyMat = new THREE.MeshStandardMaterial({ color: 0xdd3333, roughness: 0.3, metalness: 0.7 });
        const body = new THREE.Mesh(bodyGeo, bodyMat);
        body.castShadow = true;
        body.receiveShadow = true;
        body.position.y = 0.25;
        carGroup.add(body);
        // Kabin
        const cabinGeo = new THREE.BoxGeometry(1.2, 0.4, 1.4);
        const cabinMat = new THREE.MeshStandardMaterial({ color: 0x88aaff, roughness: 0.2, metalness: 0.1 });
        const cabin = new THREE.Mesh(cabinGeo, cabinMat);
        cabin.position.y = 0.55;
        cabin.position.z = -0.3;
        cabin.castShadow = true;
        carGroup.add(cabin);
        // Atap
        const roofGeo = new THREE.BoxGeometry(1.15, 0.2, 1.3);
        const roofMat = new THREE.MeshStandardMaterial({ color: 0xccccdd });
        const roof = new THREE.Mesh(roofGeo, roofMat);
        roof.position.y = 0.85;
        roof.position.z = -0.3;
        roof.castShadow = true;
        carGroup.add(roof);
        
        // Roda
        const wheelGeo = new THREE.CylinderGeometry(0.4, 0.4, 0.5, 24);
        const wheelMat = new THREE.MeshStandardMaterial({ color: 0x222222, roughness: 0.6, metalness: 0.8 });
        const positions = [[-0.9, 0.2, 1.1], [0.9, 0.2, 1.1], [-0.9, 0.2, -1.1], [0.9, 0.2, -1.1]];
        const wheels = [];
        positions.forEach(pos => {
            const wheel = new THREE.Mesh(wheelGeo, wheelMat);
            wheel.rotation.z = Math.PI / 2;
            wheel.position.set(pos[0], pos[1], pos[2]);
            wheel.castShadow = true;
            carGroup.add(wheel);
            wheels.push(wheel);
        });
        // Lampu depan & belakang
        const lightMat = new THREE.MeshStandardMaterial({ color: 0xffaa66, emissive: 0xff4411 });
        const frontLight = new THREE.Mesh(new THREE.SphereGeometry(0.15, 8, 8), lightMat);
        frontLight.position.set(0, 0.2, 1.65);
        carGroup.add(frontLight);
        const rearLight = new THREE.Mesh(new THREE.SphereGeometry(0.15, 8, 8), new THREE.MeshStandardMaterial({ color: 0xcc3300, emissive: 0x441100 }));
        rearLight.position.set(0, 0.2, -1.65);
        carGroup.add(rearLight);
        
        scene.add(carGroup);
        
        // --- Parameter Fisika & Kontrol ---
        // Keadaan mobil
        let velocity = 0;        // m/s (positif maju, negatif mundur)
        let angle = 0;           // orientasi mobil (radian)
        let steeringAngle = 0;   // sudut stir (rad) maks ±0.7 rad (40 derajat)
        
        // Kontrol input
        const keyState = {
            ArrowUp: false, ArrowDown: false, ArrowLeft: false, ArrowRight: false,
            KeyW: false, KeyS: false, KeyA: false, KeyD: false
        };
        
        let throttle = 0;    // 0..1
        let brake = 0;       // 0..1
        
        // Sistem Transmisi Manual
        let currentGearMode = "1"; // "R", "N", "1","2","3","4","5"
        const gears = {
            "R": { maxSpeed: 25, accelFactor: 0.55, name: "R" },    // mundur
            "N": { maxSpeed: 0, accelFactor: 0, name: "N" },
            "1": { maxSpeed: 30, accelFactor: 1.0, name: "1" },
            "2": { maxSpeed: 70, accelFactor: 0.85, name: "2" },
            "3": { maxSpeed: 120, accelFactor: 0.72, name: "3" },
            "4": { maxSpeed: 170, accelFactor: 0.62, name: "4" },
            "5": { maxSpeed: 220, accelFactor: 0.55, name: "5" }
        };
        
        // Parameter fisis
        const MAX_STEER = 0.75;         // rad (43 derajat)
        const STEER_SPEED = 2.2;        // kecepatan balik stir
        const ACC_FORCE = 14.0;         // percepatan maks m/s^2
        const BRAKE_FORCE = 10.0;        // pengereman m/s^2
        const DRAG_COEFF = 0.98;         // drag & rolling resistance (per detik)
        const REVERSE_ACC_MULT = 0.65;    // akselerasi mundur lebih lembut
        
        // Update steering dari input
        let steerInput = 0;
        
        // UI Elements
        const speedElem = document.getElementById('speedValue');
        const gearDisplayElem = document.getElementById('gearDisplay');
        const rpmElem = document.getElementById('rpmValue');
        const gearLimitHint = document.getElementById('gearLimitHint');
        
        function updateUI() {
            const spdKMH = Math.abs(velocity * 3.6);
            speedElem.innerText = Math.floor(spdKMH);
            let displayGear = currentGearMode;
            if (displayGear === "R") displayGear = "R";
            else if (displayGear === "N") displayGear = "N";
            else displayGear = displayGear;
            gearDisplayElem.innerText = displayGear;
            
            // simulasi RPM berdasarkan rasio kecepatan terhadap maxSpeed gigi (kecuali netral/mundur)
            let rpmPercent = 0;
            if (currentGearMode !== "N" && currentGearMode !== "R") {
                const maxSp = gears[currentGearMode].maxSpeed;
                const spd = Math.abs(velocity * 3.6);
                if (maxSp > 0) {
                    rpmPercent = Math.min(100, (spd / maxSp) * 110);
                    if (throttle > 0.1 && spd < maxSp * 0.95) rpmPercent = Math.min(95, rpmPercent + throttle * 15);
                } else rpmPercent = 0;
            } else if (currentGearMode === "R") {
                const maxR = gears.R.maxSpeed;
                const spd = Math.abs(velocity * 3.6);
                rpmPercent = Math.min(75, (spd / maxR) * 80);
            } else {
                rpmPercent = 0;
            }
            rpmElem.innerText = Math.floor(rpmPercent);
            
            // update hint batas
            let limits = "";
            if (currentGearMode === "1") limits = "Max 30 km/h | Aksel tinggi";
            else if (currentGearMode === "2") limits = "Max 70 km/h";
            else if (currentGearMode === "3") limits = "Max 120 km/h";
            else if (currentGearMode === "4") limits = "Max 170 km/h";
            else if (currentGearMode === "5") limits = "Max 220 km/h | Kecepatan tertinggi";
            else if (currentGearMode === "R") limits = "Mundur | Max 25 km/h";
            else limits = "Netral - Gas tidak berpengaruh";
            gearLimitHint.innerText = `Gigi ${currentGearMode}: ${limits}`;
        }
        
        // Fungsi untuk mengubah gigi dengan validasi
        function shiftGearUp() {
            const order = ["R", "N", "1", "2", "3", "4", "5"];
            let idx = order.indexOf(currentGearMode);
            if (idx !== -1 && idx < order.length - 1) {
                // cek khusus: jika di R atau N, next ke 1
                let next = order[idx + 1];
                currentGearMode = next;
                // Efek suara real (opsional: batasi kecepatan jika overspeed)
                clampSpeedToGearLimit();
            }
        }
        
        function shiftGearDown() {
            const order = ["R", "N", "1", "2", "3", "4", "5"];
            let idx = order.indexOf(currentGearMode);
            if (idx !== -1 && idx > 0) {
                currentGearMode = order[idx - 1];
                clampSpeedToGearLimit();
            }
        }
        
        function setReverse() {
            if (currentGearMode !== "R") {
                currentGearMode = "R";
                clampSpeedToGearLimit();
            }
        }
        
        function setNeutral() {
            if (currentGearMode !== "N") {
                currentGearMode = "N";
                // netral: kecepatan bisa berkurang secara natural, tidak ada akselerasi
            }
        }
        
        function clampSpeedToGearLimit() {
            const gear = gears[currentGearMode];
            if (!gear) return;
            const maxSpeedMps = gear.maxSpeed / 3.6;
            if (currentGearMode !== "R" && currentGearMode !== "N") {
                if (velocity > maxSpeedMps) velocity = maxSpeedMps;
                if (velocity < 0 && currentGearMode !== "R") velocity = 0; // maju netral mundur diatur terpisah
            } else if (currentGearMode === "R") {
                if (velocity < -maxSpeedMps) velocity = -maxSpeedMps;
                if (velocity > 0) velocity = 0; // kalau posisif saat R, set ke 0
            } else if (currentGearMode === "N") {
                // tidak ada limitasi percepatan hanya friksi
            }
        }
        
        // --- Fisika update per frame ---
        let lastTimestamp = 0;
        let deltaTime = 1/60; // default
        
        function updatePhysics(dt) {
            if (dt > 0.033) dt = 0.033;
            
            // 1. Baca throttle & brake dari input
            let rawThrottle = (keyState.ArrowUp || keyState.KeyW) ? 1 : 0;
            let rawBrake = (keyState.ArrowDown || keyState.KeyS) ? 1 : 0;
            
            throttle = rawThrottle;
            brake = rawBrake;
            
            // 2. Hitung gaya akselerasi berdasarkan gigi & arah
            let acceleration = 0;
            const gearData = gears[currentGearMode];
            const currentSpeedAbs = Math.abs(velocity);
            const maxGearSpeedMps = gearData ? (gearData.maxSpeed / 3.6) : 0;
            
            if (currentGearMode !== "N") {
                let allowAccel = true;
                let maxReachable = maxGearSpeedMps;
                // Untuk gigi maju (1-5) cek jika kecepatan melebihi batas gigi, gas tidak nambah percepatan
                if (currentGearMode !== "R" && currentGearMode !== "N") {
                    if (velocity > maxGearSpeedMps) allowAccel = false;
                    if (velocity < 0) allowAccel = false; // tidak bisa akselerasi maju jika mundur
                } 
                else if (currentGearMode === "R") {
                    if (velocity < -maxGearSpeedMps) allowAccel = false;
                    if (velocity > 0) allowAccel = false;
                    maxReachable = maxGearSpeedMps;
                }
                
                if (allowAccel && throttle > 0) {
                    let force = ACC_FORCE * (gearData.accelFactor || 0.7);
                    if (currentGearMode === "R") force *= REVERSE_ACC_MULT;
                    // arah: maju untuk gigi 1-5, mundur untuk R
                    let direction = (currentGearMode === "R") ? -1 : 1;
                    acceleration += direction * force * throttle;
                }
            }
            
            // 3. Pengereman (rem selalu bekerja)
            if (brake > 0) {
                let brakeDecel = BRAKE_FORCE * brake;
                if (velocity > 0) acceleration -= brakeDecel;
                else if (velocity < 0) acceleration += brakeDecel;
            }
            
            // 4. Drag & rolling friction
            let drag = velocity * 0.85 * DRAG_COEFF * dt; // efek sederhana
            let frictionForce = -velocity * 1.2 * dt;
            acceleration += frictionForce;
            if (Math.abs(velocity) < 0.2 && Math.abs(acceleration) < 0.5 && throttle === 0 && brake === 0) {
                velocity = 0;
            } else {
                velocity += acceleration * dt;
            }
            
            // Batasan kecepatan berdasarkan gigi (final clamp)
            if (currentGearMode !== "N") {
                const maxSpMps = gears[currentGearMode].maxSpeed / 3.6;
                if (currentGearMode === "R") {
                    if (velocity < -maxSpMps) velocity = -maxSpMps;
                    if (velocity > 0) velocity = 0;
                } else if (currentGearMode !== "N") {
                    if (velocity > maxSpMps) velocity = maxSpMps;
                    if (velocity < 0 && currentGearMode !== "R") velocity = 0;
                }
            } else {
                // netral: hanya drag, dan tidak boleh lebih dari 10 km/h atau berkurang
                if (Math.abs(velocity) > 5) velocity *= 0.99;
            }
            
            // 5. Kemudi (steering) - pengaruh kecepatan
            let steerTarget = 0;
            if (keyState.ArrowLeft || keyState.KeyA) steerTarget = 1;
            if (keyState.ArrowRight || keyState.KeyD) steerTarget = -1;
            // smoothing steering
            steeringAngle += (steerTarget * MAX_STEER - steeringAngle) * dt * 6;
            steeringAngle = Math.min(MAX_STEER, Math.max(-MAX_STEER, steeringAngle));
            
            // Belok berdasarkan kecepatan & steeringAngle
            let turnFactor = 0;
            const spdForTurn = Math.abs(velocity);
            if (spdForTurn > 0.5) {
                turnFactor = steeringAngle * (spdForTurn / 18.0); // makin cepat belok makin sensitif
                turnFactor = Math.min(0.9, Math.max(-0.9, turnFactor));
            }
            angle += turnFactor * dt * 2.2;
            
            // Update posisi mobil
            const moveDistance = velocity * dt;
            const dx = Math.sin(angle) * moveDistance;
            const dz = Math.cos(angle) * moveDistance;
            carGroup.position.x += dx;
            carGroup.position.z += dz;
            
            // Rotasi mobil
            carGroup.rotation.y = angle;
            
            // Animasi roda berputar berdasarkan kecepatan
            const wheelRotation = velocity * dt * 12;
            wheels.forEach(wheel => {
                wheel.rotateX(wheelRotation);
            });
            
            // Efek kemudi pada roda depan (visual)
            // (opsional: roda depan sedikit berputar)
            const frontLeftWheel = wheels[0];
            const frontRightWheel = wheels[1];
            if (frontLeftWheel && frontRightWheel) {
                frontLeftWheel.rotation.z = steeringAngle * 0.6;
                frontRightWheel.rotation.z = steeringAngle * 0.6;
            }
            
            // Batas dunia (soft boundary)
            const bound = 85;
            if (Math.abs(carGroup.position.x) > bound) carGroup.position.x *= 0.99;
            if (Math.abs(carGroup.position.z) > bound) carGroup.position.z *= 0.99;
            
            // Update UI
            updateUI();
        }
        
        // --- Kamera follow (third person dinamis) ---
        function updateCamera() {
            // offset relatif terhadap mobil
            const targetOffset = new THREE.Vector3(-Math.sin(angle) * 5.5, 2.2, -Math.cos(angle) * 7.2);
            const desiredPos = carGroup.position.clone().add(targetOffset);
            camera.position.lerp(desiredPos, 0.08);
            camera.lookAt(carGroup.position);
        }
        
        // --- Event listeners keyboard ---
        window.addEventListener('keydown', (e) => {
            const code = e.code;
            if (code === 'ArrowUp' || code === 'ArrowDown' || code === 'ArrowLeft' || code === 'ArrowRight' ||
                code === 'KeyW' || code === 'KeyS' || code === 'KeyA' || code === 'KeyD') {
                e.preventDefault();
                if (keyState.hasOwnProperty(code)) keyState[code] = true;
            }
            // Shift gigi
            if (code === 'KeyE') {
                e.preventDefault();
                shiftGearUp();
            }
            if (code === 'KeyQ') {
                e.preventDefault();
                shiftGearDown();
            }
            if (code === 'KeyR') {
                e.preventDefault();
                setReverse();
            }
            if (code === 'KeyN') {
                e.preventDefault();
                setNeutral();
            }
            // Tombol 1-5 langsung set gigi (opsional)
            if (code === 'Digit1') { e.preventDefault(); currentGearMode = "1"; clampSpeedToGearLimit(); }
            if (code === 'Digit2') { e.preventDefault(); currentGearMode = "2"; clampSpeedToGearLimit(); }
            if (code === 'Digit3') { e.preventDefault(); currentGearMode = "3"; clampSpeedToGearLimit(); }
            if (code === 'Digit4') { e.preventDefault(); currentGearMode = "4"; clampSpeedToGearLimit(); }
            if (code === 'Digit5') { e.preventDefault(); currentGearMode = "5"; clampSpeedToGearLimit(); }
        });
        
        window.addEventListener('keyup', (e) => {
            const code = e.code;
            if (keyState.hasOwnProperty(code)) keyState[code] = false;
            if (code === 'ArrowUp' || code === 'ArrowDown' || code === 'ArrowLeft' || code === 'ArrowRight' ||
                code === 'KeyW' || code === 'KeyS' || code === 'KeyA' || code === 'KeyD') {
                e.preventDefault();
            }
        });
        
        // Touch/Mobile sederhana tidak wajib, tapi agar fokus, mouse click prevent default
        window.addEventListener('contextmenu', (e) => e.preventDefault());
        
        // --- Animasi loop ---
        let lastTime = performance.now();
        function animate() {
            const now = performance.now();
            let dt = Math.min(0.033, (now - lastTime) / 1000);
            if (dt <= 0) dt = 1/60;
            lastTime = now;
            
            updatePhysics(dt);
            updateCamera();
            
            renderer.render(scene, camera);
            labelRenderer.render(scene, camera);
            requestAnimationFrame(animate);
        }
        
        // Reset posisi awal
        carGroup.position.set(0, 0.1, 0);
        angle = 0;
        velocity = 0;
        currentGearMode = "1";
        updateUI();
        
        // Mulai game
        animate();
        
        // Resize handler
        window.addEventListener('resize', onWindowResize, false);
        function onWindowResize() {
            camera.aspect = window.innerWidth / window.innerHeight;
            camera.updateProjectionMatrix();
            renderer.setSize(window.innerWidth, window.innerHeight);
            labelRenderer.setSize(window.innerWidth, window.innerHeight);
        }
        
        // Tambahan sedikit efek partikel debu (opsional, tidak mengganggu performa)
        console.log("Game siap! Gunakan E/Q untuk transmisi, R/N untuk mundur/netral. Stir kiri/kanan, gas rem.");
    </script>
</body>
</html>
