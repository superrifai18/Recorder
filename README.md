<!DOCTYPE html>
<html lang="id">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no">
    <title>Remote Monitor Pro - Fixed</title>
    <!-- Library PeerJS -->
    <script src="https://unpkg.com"></script>
    <style>
        :root { --primary: #007aff; --success: #28a745; --bg: #000; --card: #1c1c1e; }
        body { margin: 0; background: var(--bg); color: #fff; font-family: -apple-system, sans-serif; display: flex; flex-direction: column; align-items: center; padding: 20px; box-sizing: border-box; }
        
        .card { width: 100%; max-width: 450px; background: var(--card); padding: 20px; border-radius: 15px; margin-bottom: 15px; text-align: center; border: 1px solid #333; box-sizing: border-box; }
        
        #my-id { font-family: monospace; font-size: 16px; color: #fbff00; background: #000; padding: 12px; border-radius: 8px; margin: 10px 0; word-break: break-all; border: 1px dashed #555; min-height: 20px; }
        
        video { width: 100%; max-width: 450px; border-radius: 12px; background: #111; margin: 10px 0; border: 1px solid #444; }
        
        button { width: 100%; padding: 16px; margin: 6px 0; border: none; border-radius: 10px; font-weight: bold; cursor: pointer; transition: 0.2s; font-size: 14px; }
        button:active { transform: scale(0.98); opacity: 0.8; }
        
        .btn-copy { background: #444; color: white; }
        .btn-stream { background: var(--success); color: white; }
        .btn-view { background: var(--primary); color: white; }
        
        input { width: 100%; padding: 15px; border-radius: 10px; border: 1px solid #444; background: #2c2c2e; color: white; box-sizing: border-box; margin-bottom: 10px; text-align: center; }
        .label { font-size: 11px; color: #888; text-transform: uppercase; letter-spacing: 1px; display: block; margin-bottom: 5px; }
    </style>
</head>
<body>

    <div class="card">
        <span class="label">ID PERANGKAT ANDA</span>
        <div id="my-id">Menghubungkan...</div>
        <button class="btn-copy" onclick="copyID()">Salin ID</button>
    </div>

    <video id="display" autoplay playsinline muted></video>

    <div class="card">
        <span class="label">PENGIRIM (HP TARGET)</span>
        <button class="btn-stream" onclick="initMedia('user')">Aktifkan Kamera</button>
        <button class="btn-stream" onclick="initMedia('screen')">Aktifkan Layar</button>
    </div>

    <div class="card">
        <span class="label">PENONTON (LIHAT HP LAIN)</span>
        <input type="text" id="inputTargetId" placeholder="Tempel ID Target di sini">
        <button class="btn-view" onclick="startViewing()">LIHAT SEKARANG</button>
    </div>

    <script>
        // 1. Inisialisasi Global (Mencegah Initialization Error)
        let localStream = null;
        const peer = new Peer();
        const videoElement = document.getElementById('display');

        // 2. Event saat ID Peer dibuat
        peer.on('open', (id) => {
            document.getElementById('my-id').innerText = id;
        });

        // 3. Fungsi Salin ID
        function copyID() {
            const idText = document.getElementById('my-id').innerText;
            if(idText === "Menghubungkan...") return;
            navigator.clipboard.writeText(idText).then(() => {
                alert("ID Disalin! Kirim ke HP Penonton.");
            });
        }

        // 4. Inisialisasi Kamera/Layar (Sisi Target)
        async function initMedia(type) {
            try {
                if (localStream) {
                    localStream.getTracks().forEach(track => track.stop());
                }

                if (type === 'screen') {
                    localStream = await navigator.mediaDevices.getDisplayMedia({ video: true });
                } else {
                    localStream = await navigator.mediaDevices.getUserMedia({ 
                        video: { facingMode: type }, 
                        audio: true 
                    });
                }
                
                videoElement.srcObject = localStream;
                alert("Media Aktif! ID Anda sekarang bisa dihubungi.");
            } catch (err) {
                alert("Gagal: " + err.message);
            }
        }

        // 5. Menjawab Panggilan Otomatis (Sisi Target)
        peer.on('call', (call) => {
            if (localStream) {
                call.answer(localStream);
            } else {
                alert("Panggilan masuk ditolak: Aktifkan kamera/layar dulu!");
            }
        });

        // 6. Menghubungkan ke Perangkat Lain (Sisi Penonton)
        function startViewing() {
            const targetId = document.getElementById('inputTargetId').value;
            if (!targetId) return alert("Masukkan ID Target!");

            // Melakukan panggilan pancingan
            const call = peer.call(targetId, new MediaStream());
            call.on('stream', (remoteStream) => {
                videoElement.srcObject = remoteStream;
                videoElement.muted = false; // Aktifkan suara
            });
        }
    </script>
</body>
</html>
        #my-id { font-family: monospace; font-size: 18px; color: #fbff00; background: #000; padding: 10px; border-radius: 8px; margin: 10px 0; word-break: break-all; border: 1px dashed #555; }
        
        video { width: 100%; border-radius: 12px; background: #000; box-shadow: 0 5px 20px rgba(0,0,0,0.5); margin: 10px 0; }
        
        button { width: 100%; padding: 15px; margin: 5px 0; border: none; border-radius: 10px; font-weight: bold; cursor: pointer; transition: 0.2s; font-size: 14px; }
        button:active { transform: scale(0.98); opacity: 0.8; }
        
        .btn-copy { background: #444; color: white; margin-top: 5px; }
        .btn-stream { background: var(--success); color: white; }
        .btn-view { background: var(--primary); color: white; }
        
        input { width: 100%; padding: 15px; border-radius: 10px; border: 1px solid #444; background: #2c2c2e; color: white; box-sizing: border-box; margin-bottom: 10px; }
        .status { font-size: 12px; color: #888; text-transform: uppercase; letter-spacing: 1px; }
    </style>
</head>
<body>

    <div class="container">
        <!-- SEKSI ID SAYA -->
        <div class="card">
            <span class="status">ID PERANGKAT ANDA</span>
            <div id="my-id">Menghubungkan ke server...</div>
            <button class="btn-copy" onclick="copyID()">Salin ID Saya</button>
        </div>

        <video id="mainDisplay" autoplay playsinline muted></video>

        <!-- SEKSI PENGIRIM (TARGET) -->
        <div class="card">
            <span class="status">MODE PENGIRIM (HP TARGET)</span>
            <button class="btn-stream" onclick="startSource('user')">Aktifkan Kamera Selfie</button>
            <button class="btn-stream" onclick="startSource('screen')">Aktifkan Layar HP</button>
        </div>

        <!-- SEKSI PENONTON -->
        <div class="card">
            <span class="status">MODE PENONTON (LIHAT HP LAIN)</span>
            <input type="text" id="targetPeerId" placeholder="Tempel ID HP Target di sini">
            <button class="btn-view" onclick="connectToTarget()">LIHAT LIVE SEKARANG</button>
        </div>
    </div>

    <script>
        const peer = new Peer();
        let localStream = null;

        // Mendapatkan ID saat koneksi ke server PeerJS sukses
        peer.on('open', id => {
            document.getElementById('my-id').innerText = id;
        });

        // Fungsi Salin ID ke Clipboard
        function copyID() {
            const idText = document.getElementById('my-id').innerText;
            navigator.clipboard.writeText(idText).then(() => {
                alert("ID Berhasil Disalin! Kirimkan ke HP Penonton.");
            });
        }

        // Aktifkan Kamera atau Layar (HP Target)
        async function startSource(type) {
            try {
                if (type === 'screen') {
                    localStream = await navigator.mediaDevices.getDisplayMedia({ video: true });
                } else {
                    localStream = await navigator.mediaDevices.getUserMedia({ 
                        video: { facingMode: type }, 
                        audio: true 
                    });
                }
                document.getElementById('mainDisplay').srcObject = localStream;
                alert("Berhasil! Sekarang biarkan halaman ini terbuka & minta HP Penonton hubungkan ID Anda.");
            } catch (err) {
                alert("Gagal akses: " + err.message + "\nPastikan menggunakan HTTPS!");
            }
        }

        // Otomatis balas jika ada panggilan masuk (Sisi Target)
        peer.on('call', call => {
            if (localStream) {
                call.answer(localStream);
            } else {
                alert("Ada koneksi masuk, tapi kamera Anda belum aktif!");
            }
        });

        // Menghubungkan ke Target (Sisi Penonton)
        function connectToTarget() {
            const targetId = document.getElementById('targetPeerId').value;
            if (!targetId) return alert("Masukkan ID Target terlebih dahulu!");

            // Melakukan panggilan dengan media stream kosong untuk memancing respon
            const call = peer.call(targetId, new MediaStream());
            call.on('stream', remoteStream => {
                const video = document.getElementById('mainDisplay');
                video.srcObject = remoteStream;
                video.muted = false; // Aktifkan suara dari HP target
            });
        }
    </script>
</body>
</html>
        input { width: 90%; padding: 15px; border-radius: 10px; border: 1px solid #444; background: #333; color: white; margin-bottom: 10px; }
        #my-id { color: #fbff00; font-weight: bold; font-size: 18px; word-break: break-all; }
    </style>
</head>
<body>

    <div class="box">
        <p>ID PERANGKAT INI:</p>
        <div id="my-id">Menghubungkan...</div>
    </div>

    <video id="mainVideo" autoplay playsinline muted></video>

    <div class="box">
        <p><strong>LANGKAH 1:</strong> HP Target Klik Ini</p>
        <button class="btn-green" onclick="setupStream('user')">Aktifkan Kamera</button>
        <button class="btn-green" onclick="setupStream('screen')">Aktifkan Layar</button>
    </div>

    <div class="box">
        <p><strong>LANGKAH 2:</strong> HP Penonton Masukkan ID Target</p>
        <input type="text" id="remoteId" placeholder="Tempel ID Target di sini">
        <button class="btn-blue" onclick="callTarget()">HUBUNGKAN & LIHAT</button>
    </div>

    <script>
        const peer = new Peer();
        let localMediaStream = null;

        // Mendapatkan ID unik perangkat ini
        peer.on('open', id => {
            document.getElementById('my-id').innerText = id;
        });

        // Menyiapkan Kamera/Layar di HP Target
        async function setupStream(type) {
            try {
                if (type === 'screen') {
                    localMediaStream = await navigator.mediaDevices.getDisplayMedia({ video: true });
                } else {
                    localMediaStream = await navigator.mediaDevices.getUserMedia({ video: { facingMode: type }, audio: true });
                }
                document.getElementById('mainVideo').srcObject = localMediaStream;
                alert("Kamera Aktif! Silakan hubungkan dari HP lain menggunakan ID di atas.");
            } catch (err) {
                alert("Gagal akses: " + err.message);
            }
        }

        // Menjawab panggilan otomatis (Sisi HP Target)
        peer.on('call', call => {
            if (localMediaStream) {
                call.answer(localMediaStream);
                console.log("Mengirim stream ke penonton...");
            } else {
                alert("Error: Aktifkan kamera dulu di HP ini sebelum dihubungkan!");
            }
        });

        // Menghubungkan ke HP Target (Sisi HP Penonton)
        function callTarget() {
            const id = document.getElementById('remoteId').value;
            if (!id) return alert("Isi ID Target!");

            // Melakukan panggilan kosong untuk memancing respon stream
            const call = peer.call(id, new MediaStream());
            call.on('stream', remoteStream => {
                document.getElementById('mainVideo').srcObject = remoteStream;
                document.getElementById('mainVideo').muted = false; // Aktifkan suara dari target
            });
        }
    </script>
</body>
</html>
        #peer-id-display { margin: 10px; font-family: monospace; color: #fbff00; }
    </style>
</head>
<body>

    <h2>Remote Monitor</h2>
    <div id="peer-id-display">Menghubungkan ke server...</div>

    <video id="remoteVideo" autoplay playsinline></video>

    <div class="controls">
        <p>--- MODE PENGIRIM (HP Target) ---</p>
        <button class="btn-send" onclick="startStreaming('user')">Kirim Kamera Depan</button>
        <button class="btn-send" onclick="startStreaming('screen')">Kirim Layar HP</button>
        
        <p>--- MODE PENONTON (HP Anda) ---</p>
        <input type="text" id="targetId" placeholder="Masukkan ID HP Target" style="padding:10px; width:75%; border-radius:5px;">
        <button class="btn-view" onclick="connectToPeer()">Lihat Perangkat Lain</button>
    </div>

    <script>
        const peer = new Peer(); // Membuat ID unik otomatis
        let localStream;

        // Menampilkan ID perangkat ini
        peer.on('open', (id) => {
            document.getElementById('peer-id-display').innerText = "ID PERANGKAT INI: " + id;
        });

        // Menangani panggilan masuk (ketika penonton terhubung)
        peer.on('call', (call) => {
            call.answer(localStream);
            call.on('stream', (stream) => {
                document.getElementById('remoteVideo').srcObject = stream;
            });
        });

        // Fungsi Mengirim Video (Kamera/Layar)
        async function startStreaming(type) {
            try {
                if (type === 'screen') {
                    localStream = await navigator.mediaDevices.getDisplayMedia({ video: true });
                } else {
                    localStream = await navigator.mediaDevices.getUserMedia({ video: { facingMode: type }, audio: true });
                }
                document.getElementById('remoteVideo').srcObject = localStream;
                alert("Siap! Sekarang masukkan ID ini di HP Penonton.");
            } catch (err) {
                alert("Gagal akses media: " + err);
            }
        }

        // Fungsi Menonton Perangkat Lain
        function connectToPeer() {
            const targetId = document.getElementById('targetId').value;
            if (!targetId) return alert("Masukkan ID dulu!");
            
            // Melakukan panggilan kosong untuk memicu pertukaran stream
            const call = peer.call(targetId, new MediaStream()); 
            call.on('stream', (stream) => {
                document.getElementById('remoteVideo').srcObject = stream;
                document.getElementById('status').innerText = "MENONTON LIVE...";
            });
        }
    </script>
</body>
</html>

        /* Preview Video */
        .preview-area { flex: 1; position: relative; display: flex; align-items: center; justify-content: center; background: #111; }
        video { width: 100%; height: 100%; object-fit: contain; }

        /* Controls Panel */
        .controls { background: #1c1c1e; padding: 30px 20px; border-top: 1px solid #333; display: grid; grid-template-columns: 1fr 1fr; gap: 15px; border-radius: 25px 25px 0 0; }
        
        button { padding: 16px; border: none; border-radius: 12px; font-weight: bold; font-size: 14px; cursor: pointer; transition: 0.3s; background: #3a3a3c; color: white; }
        button:active { opacity: 0.7; transform: scale(0.95); }
        .active { background: var(--primary) !important; box-shadow: 0 0 15px rgba(0,122,255,0.4); }
        
        .btn-screen { grid-column: span 2; background: #2c2c2e; border: 1px solid #444; }
        .btn-record { grid-column: span 2; background: var(--danger); height: 60px; font-size: 16px; margin-top: 5px; }
        .btn-record:disabled { background: #555; opacity: 0.5; }
    </style>
</head>
<body>

    <div class="header">
        <div id="timer">00:10</div>
        <div id="status">PILIH SUMBER PERANGKAT</div>
    </div>

    <div class="preview-area">
        <video id="mainVideo" autoplay muted playsinline></video>
    </div>

    <div class="controls">
        <button id="btnUser" onclick="startCamera('user', this)">Selfie Cam</button>
        <button id="btnEnv" onclick="startCamera('environment', this)">Kamera Belakang</button>
        <button id="btnScreen" class="btn-screen" onclick="startScreen(this)">Bagikan Layar Android</button>
        <button id="btnRecord" class="btn-record" onclick="startRecording()" disabled>MULAI REKAM (10s)</button>
    </div>

    <script>
        let currentStream = null;
        let recorder = null;
        let chunks = [];
        let countdown = null;
        
        const videoEl = document.getElementById('mainVideo');
        const timerEl = document.getElementById('timer');
        const statusEl = document.getElementById('status');
        const recordBtn = document.getElementById('btnRecord');

        function stopAllTracks() {
            if (currentStream) currentStream.getTracks().forEach(t => t.stop());
            document.querySelectorAll('button').forEach(b => b.classList.remove('active'));
            clearInterval(countdown);
            timerEl.style.display = 'none';
        }

        async function startCamera(mode, btn) {
            stopAllTracks();
            try {
                currentStream = await navigator.mediaDevices.getUserMedia({ 
                    video: { facingMode: mode }, audio: true 
                });
                setupPreview(btn, mode === 'user' ? "KAMERA DEPAN AKTIF" : "KAMERA BELAKANG AKTIF");
            } catch (e) { alert("Izin Kamera Ditolak!"); }
        }

        async function startScreen(btn) {
            stopAllTracks();
            try {
                currentStream = await navigator.mediaDevices.getDisplayMedia({ video: true });
                setupPreview(btn, "BERBAGI LAYAR AKTIF");
            } catch (e) { console.log("Batal berbagi layar"); }
        }

        function setupPreview(btn, msg) {
            videoEl.srcObject = currentStream;
            btn.classList.add('active');
            statusEl.innerText = msg;
            recordBtn.disabled = false;
        }

        function startRecording() {
            chunks = [];
            recorder = new MediaRecorder(currentStream);
            
            recorder.ondataavailable = e => chunks.push(e.data);
            recorder.onstop = saveFile;

            recorder.start();
            recordBtn.disabled = true;
            timerEl.style.display = 'block';
            statusEl.innerText = "SEDANG MEREKAM...";

            let timeLeft = 10;
            timerEl.innerText = `00:${timeLeft < 10 ? '0'+timeLeft : timeLeft}`;

            countdown = setInterval(() => {
                timeLeft--;
                timerEl.innerText = `00:${timeLeft < 10 ? '0'+timeLeft : timeLeft}`;
                if (timeLeft <= 0) {
                    recorder.stop();
                    clearInterval(countdown);
                }
            }, 1000);
        }

        function saveFile() {
            const blob = new Blob(chunks, { type: 'video/webm' });
            const url = URL.createObjectURL(blob);
            const a = document.createElement('a');
            a.href = url;
            a.download = `Rekaman_${new Date().getTime()}.webm`;
            a.click();
            
            statusEl.innerText = "REKAMAN TERSIMPAN";
            timerEl.style.display = 'none';
            recordBtn.disabled = false;
        }
    </script>
</body>
</html>
