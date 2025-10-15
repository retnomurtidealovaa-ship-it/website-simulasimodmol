# website-simulasimodmol[index.html](https://github.com/user-attachments/files/22931579/index.html)
<!DOCTYPE html>
<html lang="id">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Simulasi Model Molekul</title>
    <link href="https://fonts.googleapis.com/css2?family=Inter:wght@300;400;600;800&display=swap" rel="stylesheet">
    <script src="https://cdnjs.cloudflare.com/ajax/libs/three.js/r128/three.min.js"></script>
    <script src="https://cdn.jsdelivr.net/npm/three@0.128.0/examples/js/controls/OrbitControls.min.js"></script>
    <style>
        body {
            margin: 0;
            padding: 0;
            font-family: 'Inter', 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
            background: linear-gradient(135deg, #071426 0%, #0b3451 40%, #07202e 100%);
            color: white;
            overflow: hidden;
            height: 100vh;
            display: flex;
            flex-direction: column;
        }
        
        .header {
            display: flex;
            align-items: center;
            justify-content: space-between;
            gap: 12px;
            padding: 12px 20px;
            background: linear-gradient(180deg, rgba(0,0,0,0.35), rgba(0,0,0,0.25));
            box-shadow: 0 6px 20px rgba(3,10,18,0.6);
            backdrop-filter: blur(6px);
        }
        
        h1 {
            margin: 0;
            font-size: 1.25rem;
            font-weight: 700;
            letter-spacing: -0.5px;
        }
        
        .subtitle {
            font-size: 0.85rem;
            color: rgba(255,255,255,0.85);
        }

        /* Top toolbar */
        .toolbar {
            display: flex;
            gap: 8px;
            align-items: center;
        }
        .tool-btn {
            display: inline-flex;
            align-items: center;
            gap: 8px;
            background: rgba(255,255,255,0.06);
            border: 1px solid rgba(255,255,255,0.06);
            padding: 8px 10px;
            border-radius: 8px;
            color: white;
            cursor: pointer;
            font-size: 0.9rem;
            transition: all 160ms ease;
        }
        .tool-btn:hover { transform: translateY(-3px); background: rgba(255,255,255,0.08);} 
        .tool-btn.secondary { background: rgba(255,255,255,0.03); border-color: rgba(255,255,255,0.04); }
        .tool-toggle { padding: 6px 8px; border-radius: 6px; }
        .volume-slider { width: 110px; }
        
        .container {
            display: flex;
            flex: 1;
            overflow: hidden;
        }
        
        .controls {
            width: 250px;
            background-color: rgba(0, 0, 0, 0.5);
            padding: 20px;
            overflow-y: auto;
            box-shadow: 3px 0 10px rgba(0, 0, 0, 0.3);
        }

        /* Responsive: collapse controls on small screens */
        @media (max-width: 900px) {
            .controls { width: 220px; }
        }
        @media (max-width: 700px) {
            .container { flex-direction: column-reverse; }
            .controls { width: 100%; height: 260px; box-shadow: 0 -6px 20px rgba(0,0,0,0.5); }
            .viewer { height: calc(100vh - 320px); }
        }
        
        .molecule-list {
            margin-bottom: 25px;
        }
        
        .molecule-list h3 {
            border-bottom: 2px solid rgba(255, 255, 255, 0.3);
            padding-bottom: 8px;
            margin-top: 0;
        }
        
        .molecule-btn {
            display: block;
            width: 100%;
            padding: 12px;
            margin: 8px 0;
            background: rgba(255, 255, 255, 0.1);
            border: 1px solid rgba(255, 255, 255, 0.3);
            border-radius: 5px;
            color: white;
            font-size: 1rem;
            cursor: pointer;
            transition: all 0.3s ease;
        }
        
        .molecule-btn:hover {
            background: rgba(255, 255, 255, 0.2);
            transform: translateY(-2px);
        }
        
        .molecule-btn.active {
            background: rgba(0, 150, 255, 0.4);
            border-color: rgba(0, 150, 255, 0.7);
        }
        
        .info-panel {
            margin-top: 20px;
            background: rgba(0, 0, 0, 0.3);
            padding: 15px;
            border-radius: 8px;
        }
        
        .info-panel h3 {
            margin-top: 0;
        }
        
        .atom-legend {
            margin-top: 20px;
        }
        
        .atom-item {
            display: flex;
            align-items: center;
            margin: 8px 0;
        }
        
        .atom-color {
            width: 20px;
            height: 20px;
            border-radius: 50%;
            margin-right: 10px;
            border: 1px solid rgba(255, 255, 255, 0.5);
        }
        
        .viewer {
            flex: 1;
            position: relative;
        }
        
        #molecule-container {
            width: 100%;
            height: 100%;
        }
        
        .instructions {
            position: absolute;
            bottom: 20px;
            left: 20px;
            background: rgba(0, 0, 0, 0.6);
            padding: 10px 15px;
            border-radius: 5px;
            font-size: 0.9rem;
        }

        /* subtle loading */
        .loading {
            position: absolute;
            top: 50%; left: 50%; transform: translate(-50%,-50%);
            background: rgba(0,0,0,0.45);
            padding: 12px 18px; border-radius: 10px; font-size: 0.95rem;
            display: flex; gap: 10px; align-items: center; z-index: 10;
        }
        .dot { width:8px;height:8px;border-radius:50%;background:#4fd1c5;animation:pop 1s infinite;}
        .dot:nth-child(2){animation-delay:0.15s}.dot:nth-child(3){animation-delay:0.3s}
        @keyframes pop{0%{transform:scale(1);opacity:0.4}50%{transform:scale(1.6);opacity:1}100%{transform:scale(1);opacity:0.4}}

        /* dark theme adjustments */
        body.light-theme { color: #062a3b; }
        body.light-theme .header { background: rgba(255,255,255,0.85); color: #062a3b }
        body.light-theme .controls { background: rgba(255,255,255,0.98); color: #062a3b }
        
        .footer {
            text-align: center;
            padding: 10px;
            background-color: rgba(0, 0, 0, 0.4);
            font-size: 0.9rem;
        }
    </style>
</head>
<body>
    <div class="header">
        <div>
            <h1>Simulasi Model Molekul 3D</h1>
            <div class="subtitle">Visualisasi struktur molekul dalam ruang 3 dimensi</div>
        </div>

        <div class="toolbar">
            <button class="tool-btn" id="btn-audio-toggle">Play Music</button>
            <input type="range" id="volume" class="volume-slider" min="0" max="1" step="0.01" value="0.35" aria-label="volume">
            <button class="tool-btn" id="btn-autospin">Auto-rotate: Off</button>
            <button class="tool-btn" id="btn-screenshot">Screenshot</button>
            <button class="tool-btn" id="btn-fullscreen">Fullscreen</button>
            <button class="tool-btn secondary" id="btn-theme">Toggle Theme</button>
        </div>
    </div>

    <div class="loading" id="loading" style="display:none;">
        <div class="dot"></div>
        <div class="dot"></div>
        <div class="dot"></div>
        <div>Memuat...</div>
    </div>
    
    <div class="container">
        <div class="controls">
            <div class="molecule-list">
                <h3>Pilih Molekul</h3>
                <button class="molecule-btn active" data-molecule="water">Air (H₂O)</button>
                <button class="molecule-btn" data-molecule="methane">Metana (CH₄)</button>
                <button class="molecule-btn" data-molecule="ammonia">Amonia (NH₃)</button>
                <button class="molecule-btn" data-molecule="carbonDioxide">Karbon Dioksida (CO₂)</button>
                <button class="molecule-btn" data-molecule="benzene">Benzena (C₆H₆)</button>
                <button class="molecule-btn" data-molecule="glucose">Glukosa (C₆H₁₂O₆)</button>
            </div>
            
            <div class="info-panel">
                <h3>Informasi Molekul</h3>
                <div id="molecule-info">
                    <p><strong>Nama:</strong> Air</p>
                    <p><strong>Rumus:</strong> H₂O</p>
                    <p><strong>Massa Molar:</strong> 18.015 g/mol</p>
                    <p><strong>Struktur:</strong> Bentuk V dengan sudut 104.5°</p>
                    <p><strong>Deskripsi:</strong> Molekul air terdiri dari satu atom oksigen dan dua atom hidrogen.</p>
                </div>
            </div>
            
            <div class="atom-legend">
                <h3>Legenda Atom</h3>
                <div class="atom-item">
                    <div class="atom-color" style="background-color: #FF0D0D;"></div>
                    <span>Oksigen (O)</span>
                </div>
                <div class="atom-item">
                    <div class="atom-color" style="background-color: #1E90FF;"></div>
                    <span>Hidrogen (H)</span>
                </div>
                <div class="atom-item">
                    <div class="atom-color" style="background-color: #30A030;"></div>
                    <span>Karbon (C)</span>
                </div>
                <div class="atom-item">
                    <div class="atom-color" style="background-color: #B0B0B0;"></div>
                    <span>Nitrogen (N)</span>
                </div>
            </div>
        </div>
        
        <div class="viewer">
            <div id="molecule-container"></div>
            <div class="instructions">
                Gunakan mouse untuk memutar, menggeser, dan memperbesar model molekul
            </div>
        </div>
    </div>
    
    <div class="footer">
        Simulasi Model Molekul 3D &copy; 2023 - Visualisasi Kimia Interaktif
    </div>
    <!-- Audio untuk latar (ikan memilih royalty-free short loop) -->
    <audio id="bg-audio" loop crossorigin="anonymous">
        <source src="https://cdn.jsdelivr.net/gh/mdn/webaudio-examples/audio/pinknoise.mp3" type="audio/mpeg">
        Your browser does not support the audio element.
    </audio>

    <script>
        // Inisialisasi scene Three.js
        let scene, camera, renderer, controls;
        let currentMolecule = null;
        let autoRotate = false;
        let audioCtx, bgAudioElem, audioSource, gainNode;
        
        // Informasi molekul
        const moleculeData = {
            water: {
                name: "Air",
                formula: "H₂O",
                molarMass: "18.015 g/mol",
                structure: "Bentuk V dengan sudut 104.5°",
                description: "Molekul air terdiri dari satu atom oksigen dan dua atom hidrogen. Molekul ini bersifat polar karena perbedaan elektronegativitas antara oksigen dan hidrogen.",
                atoms: [
                    { element: "O", x: 0, y: 0, z: 0 },
                    { element: "H", x: 0.757, y: 0.586, z: 0 },
                    { element: "H", x: -0.757, y: 0.586, z: 0 }
                ],
                bonds: [
                    { from: 0, to: 1 },
                    { from: 0, to: 2 }
                ]
            },
            methane: {
                name: "Metana",
                formula: "CH₄",
                molarMass: "16.04 g/mol",
                structure: "Tetrahedral dengan sudut 109.5°",
                description: "Metana adalah hidrokarbon paling sederhana dengan satu atom karbon dan empat atom hidrogen. Molekul ini memiliki bentuk tetrahedral sempurna.",
                atoms: [
                    { element: "C", x: 0, y: 0, z: 0 },
                    { element: "H", x: 0, y: 1, z: 0 },
                    { element: "H", x: 0.9428, y: -0.3333, z: 0 },
                    { element: "H", x: -0.4714, y: -0.3333, z: 0.8165 },
                    { element: "H", x: -0.4714, y: -0.3333, z: -0.8165 }
                ],
                bonds: [
                    { from: 0, to: 1 },
                    { from: 0, to: 2 },
                    { from: 0, to: 3 },
                    { from: 0, to: 4 }
                ]
            },
            ammonia: {
                name: "Amonia",
                formula: "NH₃",
                molarMass: "17.031 g/mol",
                structure: "Piramida trigonal dengan sudut 107°",
                description: "Amonia terdiri dari satu atom nitrogen dan tiga atom hidrogen. Molekul ini memiliki bentuk piramida trigonal dengan pasangan elektron bebas pada nitrogen.",
                atoms: [
                    { element: "N", x: 0, y: 0, z: 0 },
                    { element: "H", x: 0, y: 1, z: 0 },
                    { element: "H", x: 0.9397, y: -0.3333, z: 0 },
                    { element: "H", x: -0.4698, y: -0.3333, z: 0.8138 }
                ],
                bonds: [
                    { from: 0, to: 1 },
                    { from: 0, to: 2 },
                    { from: 0, to: 3 }
                ]
            },
            carbonDioxide: {
                name: "Karbon Dioksida",
                formula: "CO₂",
                molarMass: "44.01 g/mol",
                structure: "Linear dengan sudut 180°",
                description: "Karbon dioksida terdiri dari satu atom karbon dan dua atom oksigen. Molekul ini linear dan non-polar.",
                atoms: [
                    { element: "C", x: 0, y: 0, z: 0 },
                    { element: "O", x: 1.16, y: 0, z: 0 },
                    { element: "O", x: -1.16, y: 0, z: 0 }
                ],
                bonds: [
                    { from: 0, to: 1 },
                    { from: 0, to: 2 }
                ]
            },
            benzene: {
                name: "Benzena",
                formula: "C₆H₆",
                molarMass: "78.11 g/mol",
                structure: "Cincin heksagonal planar",
                description: "Benzena adalah senyawa aromatik dengan enam atom karbon membentuk cincin dan enam atom hidrogen. Strukturnya planar dengan ikatan rangkap terkonjugasi.",
                atoms: [
                    { element: "C", x: 0, y: 0, z: 0 },
                    { element: "C", x: 1.21, y: 0, z: 0 },
                    { element: "C", x: 1.81, y: 1.05, z: 0 },
                    { element: "C", x: 1.21, y: 2.1, z: 0 },
                    { element: "C", x: 0, y: 2.1, z: 0 },
                    { element: "C", x: -0.6, y: 1.05, z: 0 },
                    { element: "H", x: -0.6, y: -1.05, z: 0 },
                    { element: "H", x: 1.81, y: -1.05, z: 0 },
                    { element: "H", x: 2.42, y: 1.05, z: 0 },
                    { element: "H", x: 1.81, y: 3.15, z: 0 },
                    { element: "H", x: -0.6, y: 3.15, z: 0 },
                    { element: "H", x: -1.21, y: 1.05, z: 0 }
                ],
                bonds: [
                    { from: 0, to: 1 }, { from: 1, to: 2 }, { from: 2, to: 3 },
                    { from: 3, to: 4 }, { from: 4, to: 5 }, { from: 5, to: 0 },
                    { from: 0, to: 6 }, { from: 1, to: 7 }, { from: 2, to: 8 },
                    { from: 3, to: 9 }, { from: 4, to: 10 }, { from: 5, to: 11 }
                ]
            },
            glucose: {
                name: "Glukosa",
                formula: "C₆H₁₂O₆",
                molarMass: "180.16 g/mol",
                structure: "Monosakarida dengan 6 atom karbon",
                description: "Glukosa adalah gula sederhana dan sumber energi utama bagi sel. Molekul ini memiliki struktur cincin dengan gugus hidroksil.",
                atoms: [
                    { element: "C", x: 0, y: 0, z: 0 },
                    { element: "C", x: 1.42, y: 0, z: 0 },
                    { element: "C", x: 1.94, y: 1.23, z: 0 },
                    { element: "C", x: 1.42, y: 2.46, z: 0 },
                    { element: "C", x: 0, y: 2.46, z: 0 },
                    { element: "C", x: -0.52, y: 1.23, z: 0 },
                    { element: "O", x: -1.42, y: 1.23, z: 0 },
                    { element: "O", x: 2.46, y: -0.82, z: 0 },
                    { element: "O", x: 2.46, y: 3.28, z: 0 },
                    { element: "H", x: -0.52, y: -0.82, z: 0 },
                    { element: "H", x: 2.46, y: -1.84, z: 0 },
                    { element: "H", x: 2.46, y: 4.3, z: 0 },
                    { element: "H", x: -0.52, y: 3.28, z: 0 },
                    { element: "H", x: -1.94, y: 1.23, z: 0 },
                    { element: "H", x: 0, y: -0.82, z: 0 },
                    { element: "H", x: 1.42, y: -0.82, z: 0 },
                    { element: "H", x: 1.94, y: -0.82, z: 0 },
                    { element: "H", x: 1.42, y: 3.28, z: 0 }
                ],
                bonds: [
                    { from: 0, to: 1 }, { from: 1, to: 2 }, { from: 2, to: 3 },
                    { from: 3, to: 4 }, { from: 4, to: 5 }, { from: 5, to: 0 },
                    { from: 5, to: 6 }, { from: 1, to: 7 }, { from: 3, to: 8 },
                    { from: 0, to: 9 }, { from: 7, to: 10 }, { from: 8, to: 11 },
                    { from: 4, to: 12 }, { from: 6, to: 13 }, { from: 0, to: 14 },
                    { from: 1, to: 15 }, { from: 2, to: 16 }, { from: 3, to: 17 }
                ]
            }
        };
        
        // Warna untuk setiap jenis atom
        const atomColors = {
            "H": 0x1E90FF,  // Biru untuk Hidrogen
            "C": 0x30A030,  // Hijau untuk Karbon
            "O": 0xFF0D0D,  // Merah untuk Oksigen
            "N": 0xB0B0B0   // Abu-abu untuk Nitrogen
        };
        
        // Ukuran untuk setiap jenis atom (radius)
        const atomSizes = {
            "H": 0.3,
            "C": 0.6,
            "O": 0.66,
            "N": 0.65
        };
        
        // Inisialisasi scene Three.js
        function init() {
            // Buat scene
            scene = new THREE.Scene();
            scene.background = new THREE.Color(0x0a0a2a);
            
            // Buat camera
            camera = new THREE.PerspectiveCamera(75, window.innerWidth / window.innerHeight, 0.1, 1000);
            camera.position.z = 5;
            
            // Buat renderer
            renderer = new THREE.WebGLRenderer({ antialias: true });
            // set pixel ratio untuk layar high-DPI
            renderer.setPixelRatio(window.devicePixelRatio || 1);
            renderer.setSize(document.getElementById('molecule-container').offsetWidth, 
                            document.getElementById('molecule-container').offsetHeight);
            document.getElementById('molecule-container').appendChild(renderer.domElement);
            // pastikan ukuran benar setelah element terpasang
            onWindowResize();
            
            // Tambahkan kontrol orbit
            controls = new THREE.OrbitControls(camera, renderer.domElement);
            controls.enableDamping = true;
            controls.dampingFactor = 0.05;
            
            // Tambahkan pencahayaan
            const ambientLight = new THREE.AmbientLight(0xffffff, 0.6);
            scene.add(ambientLight);
            
            const directionalLight = new THREE.DirectionalLight(0xffffff, 0.8);
            directionalLight.position.set(1, 1, 1);
            scene.add(directionalLight);
            
            // Tampilkan molekul pertama (air)
            showMolecule('water');

            // Setup audio
            bgAudioElem = document.getElementById('bg-audio');
            trySetupAudio();

            // Toolbar actions
            document.getElementById('btn-audio-toggle').addEventListener('click', toggleAudio);
            document.getElementById('volume').addEventListener('input', e => setVolume(parseFloat(e.target.value)));
            document.getElementById('btn-screenshot').addEventListener('click', takeScreenshot);
            document.getElementById('btn-fullscreen').addEventListener('click', toggleFullscreen);
            document.getElementById('btn-autospin').addEventListener('click', () => {
                autoRotate = !autoRotate;
                document.getElementById('btn-autospin').textContent = `Auto-rotate: ${autoRotate ? 'On' : 'Off'}`;
            });
            document.getElementById('btn-theme').addEventListener('click', toggleTheme);
            
            // Event listener untuk tombol molekul
            document.querySelectorAll('.molecule-btn').forEach(button => {
                button.addEventListener('click', function() {
                    document.querySelectorAll('.molecule-btn').forEach(btn => btn.classList.remove('active'));
                    this.classList.add('active');
                    showMolecule(this.getAttribute('data-molecule'));
                });
            });
            
            // Handle resize window
            window.addEventListener('resize', onWindowResize);
            
            // Mulai animasi
            animate();
        }
        
        // Fungsi untuk menampilkan molekul
        function showMolecule(moleculeKey) {
            // Hapus molekul sebelumnya jika ada
            if (currentMolecule) {
                scene.remove(currentMolecule);
            }
            document.getElementById('loading').style.display = 'flex';
            setTimeout(() => { document.getElementById('loading').style.display = 'none'; }, 350);
            
            // Dapatkan data molekul
            const molecule = moleculeData[moleculeKey];
            
            // Buat grup untuk molekul
            currentMolecule = new THREE.Group();
            
            // Buat atom-atom
            molecule.atoms.forEach((atom, index) => {
                const geometry = new THREE.SphereGeometry(atomSizes[atom.element], 32, 32);
                const material = new THREE.MeshPhongMaterial({ 
                    color: atomColors[atom.element],
                    shininess: 100
                });
                const sphere = new THREE.Mesh(geometry, material);
                sphere.position.set(atom.x, atom.y, atom.z);
                currentMolecule.add(sphere);
            });
            
            // Buat ikatan antar atom
            molecule.bonds.forEach(bond => {
                const atom1 = molecule.atoms[bond.from];
                const atom2 = molecule.atoms[bond.to];
                
                const direction = new THREE.Vector3().subVectors(
                    new THREE.Vector3(atom2.x, atom2.y, atom2.z),
                    new THREE.Vector3(atom1.x, atom1.y, atom1.z)
                );
                
                const length = direction.length();
                const cylinderGeometry = new THREE.CylinderGeometry(0.1, 0.1, length, 8);
                cylinderGeometry.translate(0, length/2, 0);
                cylinderGeometry.rotateX(Math.PI/2);
                
                const cylinderMaterial = new THREE.MeshPhongMaterial({ color: 0xCCCCCC });
                const cylinder = new THREE.Mesh(cylinderGeometry, cylinderMaterial);
                
                cylinder.position.set(
                    (atom1.x + atom2.x) / 2,
                    (atom1.y + atom2.y) / 2,
                    (atom1.z + atom2.z) / 2
                );
                
                cylinder.lookAt(new THREE.Vector3(atom2.x, atom2.y, atom2.z));
                currentMolecule.add(cylinder);
            });
            
            // Tambahkan molekul ke scene
            scene.add(currentMolecule);
            
            // Update informasi molekul
            updateMoleculeInfo(molecule);
        }
        
        // Update informasi molekul di panel
        function updateMoleculeInfo(molecule) {
            document.getElementById('molecule-info').innerHTML = `
                <p><strong>Nama:</strong> ${molecule.name}</p>
                <p><strong>Rumus:</strong> ${molecule.formula}</p>
                <p><strong>Massa Molar:</strong> ${molecule.molarMass}</p>
                <p><strong>Struktur:</strong> ${molecule.structure}</p>
                <p><strong>Deskripsi:</strong> ${molecule.description}</p>
            `;
        }
        
        // Handle resize window
        function onWindowResize() {
            camera.aspect = document.getElementById('molecule-container').offsetWidth / 
                           document.getElementById('molecule-container').offsetHeight;
            camera.updateProjectionMatrix();
            renderer.setSize(document.getElementById('molecule-container').offsetWidth, 
                           document.getElementById('molecule-container').offsetHeight);
        }
        
        // Fungsi animasi
        function animate() {
            requestAnimationFrame(animate);
            if (autoRotate && currentMolecule) {
                currentMolecule.rotation.y += 0.005;
            }
            controls.update();
            renderer.render(scene, camera);
        }

        // Audio helpers
        function trySetupAudio() {
            try {
                audioCtx = new (window.AudioContext || window.webkitAudioContext)();
                audioSource = audioCtx.createMediaElementSource(bgAudioElem);
                gainNode = audioCtx.createGain();
                audioSource.connect(gainNode).connect(audioCtx.destination);
                setVolume(parseFloat(document.getElementById('volume').value));
            } catch (e) {
                console.warn('Audio context not available:', e);
            }
        }

        function toggleAudio() {
            if (!bgAudioElem) return;
            // resume context if needed
            if (audioCtx && audioCtx.state === 'suspended') audioCtx.resume();
            if (bgAudioElem.paused) {
                bgAudioElem.play();
                document.getElementById('btn-audio-toggle').textContent = 'Pause Music';
            } else {
                bgAudioElem.pause();
                document.getElementById('btn-audio-toggle').textContent = 'Play Music';
            }
        }

        function setVolume(v) {
            if (gainNode) gainNode.gain.value = v;
            if (bgAudioElem) bgAudioElem.volume = v; // fallback
        }

        // Screenshot
        function takeScreenshot() {
            try {
                const dataURL = renderer.domElement.toDataURL('image/png');
                const a = document.createElement('a');
                a.href = dataURL;
                a.download = `molecule-${Date.now()}.png`;
                a.click();
            } catch (e) { console.error('Screenshot failed', e); }
        }

        // Fullscreen
        function toggleFullscreen() {
            const el = document.documentElement;
            if (!document.fullscreenElement) {
                if (el.requestFullscreen) el.requestFullscreen();
            } else {
                if (document.exitFullscreen) document.exitFullscreen();
            }
        }

        // Theme toggle
        function toggleTheme() {
            document.body.classList.toggle('light-theme');
        }
        
        // Inisialisasi saat halaman dimuat
        window.onload = init;
    </script>
</body>
</html>
