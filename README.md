<!DOCTYPE html>
<html lang="id">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no">
    <title>Multi-Media Recorder Pro</title>
    <style>
        :root { --primary: #007aff; --danger: #ff3b30; --bg: #000; }
        body { margin: 0; background: var(--bg); color: white; font-family: -apple-system, sans-serif; display: flex; flex-direction: column; height: 100vh; overflow: hidden; }
        
        /* Header & Status */
        .header { padding: 20px; text-align: center; background: rgba(255,255,255,0.05); z-index: 10; }
        #timer { font-size: 24px; font-weight: bold; color: var(--danger); display: none; margin-bottom: 5px; }
        #status { font-size: 12px; text-transform: uppercase; letter-spacing: 1.5px; color: #888; }

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
