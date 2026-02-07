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
