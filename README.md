<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no">
    <title>Ghana Life 3D</title>
    <style>
        * { box-sizing: border-box; margin: 0; padding: 0; user-select: none; }
        body, html { width: 100%; height: 100%; overflow: hidden; font-family: 'Segoe UI', Tahoma, sans-serif; background: #000; color: #fff; }

        /* 3D Canvas Area */
        #game-canvas { width: 100vw; height: 100vh; display: block; }

        /* UI Overlays */
        .overlay { position: absolute; top: 0; left: 0; width: 100%; height: 100%; pointer-events: none; }
        .interactive { pointer-events: auto; }

        /* HUD Top Bar */
        #hud {
            position: absolute; top: 15px; left: 15px; right: 15px;
            display: flex; justify-content: space-between; align-items: center;
            background: rgba(0, 0, 0, 0.75); border: 2px solid #fcd116;
            padding: 12px 20px; border-radius: 12px; backdrop-filter: blur(5px);
        }
        .stat-box { font-size: 14px; font-weight: bold; color: #006b3f; }
        .stat-box span { color: #fcd116; }

        /* Virtual Phone (Bottom Right) */
        #phone-btn {
            position: absolute; bottom: 20px; right: 20px;
            width: 65px; height: 65px; background: #0066cc; border: 3px solid #fcd116;
            border-radius: 50%; display: flex; justify-content: center; align-items: center;
            font-size: 28px; cursor: pointer; box-shadow: 0 4px 15px rgba(0,0,0,0.5);
        }

        #phone-screen {
            position: absolute; bottom: 95px; right: 20px;
            width: 310px; height: 520px; background: #111; border: 4px solid #333;
            border-radius: 36px; display: none; flex-direction: column; overflow: hidden;
            box-shadow: 0 10px 30px rgba(0,0,0,0.8);
        }
        #phone-screen.open { display: flex; }
        .phone-header { background: #222; padding: 12px; text-align: center; font-weight: bold; color: #fcd116; border-bottom: 1px solid #333; }
        .phone-body { flex: 1; padding: 15px; overflow-y: auto; display: grid; grid-template-columns: repeat(3, 1fr); gap: 10px; align-content: start; }
        .app-icon { background: #222; border: 1px solid #444; border-radius: 12px; padding: 15px 5px; text-align: center; font-size: 11px; cursor: pointer; }
        .app-icon div { font-size: 22px; margin-bottom: 5px; }

        /* Auth Modal */
        #auth-modal {
            position: absolute; top: 0; left: 0; width: 100%; height: 100%;
            background: rgba(0,0,0,0.9); display: flex; justify-content: center; align-items: center;
        }
        .auth-card {
            background: #1a1a1a; border-top: 5px solid #ce1126; border-bottom: 5px solid #006b3f;
            padding: 30px; border-radius: 16px; width: 90%; max-width: 380px; text-align: center;
        }
        .auth-card h1 { color: #fcd116; margin-bottom: 15px; font-size: 24px; }
        .auth-card input { width: 100%; padding: 12px; margin: 8px 0; background: #2a2a2a; border: 1px solid #444; color: #fff; border-radius: 8px; }
        .auth-card button { width: 100%; padding: 12px; margin-top: 15px; background: #006b3f; color: #fff; border: none; border-radius: 8px; font-weight: bold; cursor: pointer; }
    </style>
    <!-- Three.js Library for 3D Graphics -->
    <script src="https://cdnjs.cloudflare.com/ajax/libs/three.js/r128/three.min.js"></script>
</head>
<body>

    <!-- 3D Canvas -->
    <canvas id="game-canvas"></canvas>

    <!-- UI Overlay -->
    <div class="overlay">
        <!-- Top HUD -->
        <div id="hud" class="interactive">
            <div class="stat-box">PLAYER: <span id="player-display">Swartz</span></div>
            <div class="stat-box">CASH: ₵<span id="cash-display">14,000</span></div>
            <div class="stat-box">CITY: <span id="city-display">Accra</span></div>
        </div>

        <!-- Phone Button & Screen -->
        <div id="phone-btn" class="interactive" onclick="togglePhone()">📱</div>
        <div id="phone-screen" class="interactive">
            <div class="phone-header">iPhone 17 - Ghana Life</div>
            <div class="phone-body" id="app-grid">
                <div class="app-icon" onclick="openApp('Call')"><div>📞</div>Call</div>
                <div class="app-icon" onclick="openApp('Messages')"><div>💬</div>Chat</div>
                <div class="app-icon" onclick="openApp('Jobs')"><div>💼</div>Jobs</div>
                <div class="app-icon" onclick="openApp('Cars')"><div>🚗</div>Cars</div>
                <div class="app-icon" onclick="openApp('Houses')"><div>🏠</div>Houses</div>
                <div class="app-icon" onclick="openApp('Travel')"><div>✈️</div>Travel</div>
            </div>
        </div>

        <!-- Login / Registration Screen -->
        <div id="auth-modal" class="interactive">
            <div class="auth-card">
                <h1>🇬🇭 GHANA LIFE 3D</h1>
                <input type="text" id="username" placeholder="Username" value="Swartz">
                <input type="text" id="fullname" placeholder="Full Name">
                <input type="password" id="password" placeholder="Password">
                <button onclick="startGame()">CONFIRM & ENTER GAME</button>
            </div>
        </div>
    </div>

    <script>
        // --- GAME STATE ---
        let player = { name: "Swartz", cash: 14000, city: "Accra" };

        // --- THREE.JS 3D ENGINE SETUP ---
        const canvas = document.getElementById('game-canvas');
        const scene = new THREE.Scene();
        scene.background = new THREE.Color(0x87ceeb); // Sky blue
        scene.fog = new THREE.FogExp2(0x87ceeb, 0.015);

        const camera = new THREE.PerspectiveCamera(60, window.innerWidth / window.innerHeight, 0.1, 1000);
        camera.position.set(0, 15, 25);
        camera.lookAt(0, 0, 0);

        const renderer = new THREE.WebGLRenderer({ canvas: canvas, antialias: true });
        renderer.setSize(window.innerWidth, window.innerHeight);
        renderer.shadowMap.enabled = true;

        // --- LIGHTING ---
        const ambientLight = new THREE.AmbientLight(0xffffff, 0.6);
        scene.add(ambientLight);

        const sunLight = new THREE.DirectionalLight(0xffffff, 0.8);
        sunLight.position.set(40, 60, 20);
        sunLight.castShadow = true;
        scene.add(sunLight);

        // --- 3D ENVIRONMENT (STREETS & BUILDINGS) ---
        // Ground / Road
        const groundGeo = new THREE.PlaneGeometry(200, 200);
        const groundMat = new THREE.MeshStandardMaterial({ color: 0x333333 });
        const ground = new THREE.Mesh(groundGeo, groundMat);
        ground.rotation.x = -Math.PI / 2;
        ground.receiveShadow = true;
        scene.add(ground);

        // Simple City Buildings Generator
        function createBuilding(x, z, color, height) {
            const geo = new THREE.BoxGeometry(8, height, 8);
            const mat = new THREE.MeshStandardMaterial({ color: color });
            const mesh = new THREE.Mesh(geo, mat);
            mesh.position.set(x, height / 2, z);
            mesh.castShadow = true;
            mesh.receiveShadow = true;
            scene.add(mesh);
        }

        // Generate City Blocks
        const colors = [0xce1126, 0xfcd116, 0x006b3f, 0x0066cc, 0x888888];
        for (let i = -40; i <= 40; i += 20) {
            for (let j = -40; j <= 40; j += 20) {
                if (Math.abs(i) > 10 || Math.abs(j) > 10) {
                    const h = 10 + Math.random() * 20;
                    const c = colors[Math.floor(Math.random() * colors.length)];
                    createBuilding(i, j, c, h);
                }
            }
        }

        // Player 3D Character Avatar
        const playerGeo = new THREE.CapsuleGeometry(0.8, 1.8, 4, 8);
        const playerMat = new THREE.MeshStandardMaterial({ color: 0xfcd116 });
        const playerMesh = new THREE.Mesh(playerGeo, playerMat);
        playerMesh.position.set(0, 1.7, 0);
        playerMesh.castShadow = true;
        scene.add(playerMesh);

        // --- ANIMATION LOOP ---
        function animate() {
            requestAnimationFrame(animate);
            // Slow environmental rotation for broadcasting camera effect
            scene.rotation.y += 0.002;
            renderer.render(scene, camera);
        }
        animate();

        // --- GAME FUNCTIONS ---
        function startGame() {
            const user = document.getElementById('username').value;
            if(user) player.name = user;
            
            document.getElementById('player-display').innerText = player.name;
            document.getElementById('cash-display').innerText = player.cash.toLocaleString();
            document.getElementById('auth-modal').style.display = 'none';
        }

        function togglePhone() {
            const phone = document.getElementById('phone-screen');
            phone.classList.toggle('open');
        }

        function openApp(appName) {
            alert(appName + " app opened on your iPhone 17!");
        }

        window.addEventListener('resize', () => {
            camera.aspect = window.innerWidth / window.innerHeight;
            camera.updateProjectionMatrix();
            renderer.setSize(window.innerWidth, window.innerHeight);
        });
    </script>
</body>
</html># Ghana-life-sim-