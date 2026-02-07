<!DOCTYPE html>
<html lang="id">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Fixed Remote Monitor</title>
    <script src="https://unpkg.com"></script>
    <style>
        body { margin: 0; background: #000; color: #fff; font-family: sans-serif; text-align: center; padding: 10px; }
        video { width: 100%; max-width: 500px; background: #222; border-radius: 12px; margin: 15px 0; border: 2px solid #444; }
        .box { background: #1a1a1a; padding: 20px; border-radius: 15px; margin-bottom: 10px; }
        button { padding: 15px; width: 100%; margin: 5px 0; border-radius: 10px; border: none; font-weight: bold; cursor: pointer; }
        .btn-green { background: #28a745; color: white; }
        .btn-blue { background: #007bff; color: white; }
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
