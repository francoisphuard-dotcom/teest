<!DOCTYPE html>
<html lang="fr">
<head>
    <meta charset="UTF-8">
    <title>Modem Optique Flash - TOP SECRET</title>
    <style>
        body { font-family: 'Courier New', monospace; background: #000; color: #ff00ff; text-align: center; padding: 20px; }
        .card { border: 2px solid #440044; padding: 20px; border-radius: 15px; background: #0a0a0a; margin-bottom: 20px; box-shadow: 0 0 20px #ff00ff22; }
        textarea { width: 100%; height: 60px; background: #000; color: #ff00ff; border: 1px solid #440044; padding: 10px; box-sizing: border-box; font-weight: bold; }
        button { width: 100%; padding: 15px; margin-top: 10px; background: #220022; color: #ff00ff; border: 1px solid #ff00ff; cursor: pointer; font-weight: bold; text-transform: uppercase; }
        button:hover { background: #ff00ff; color: #000; box-shadow: 0 0 15px #ff00ff; }
        #receivedText { font-size: 1.8em; color: #fff; margin-top: 20px; min-height: 50px; background: #111; border-radius: 10px; padding: 10px; border: 1px solid #440044; }
        #monitor { display: block; margin: 10px auto; border: 1px solid #333; background: #000; width: 100%; max-width: 300px; border-radius: 10px; }
        .status { font-size: 0.8em; color: #666; margin-bottom: 5px; }
    </style>
</head>
<body>

    <div class="card">
        <h2>🔦 ÉMETTEUR (FLASH LED)</h2>
        <textarea id="textToSend">NILS</textarea>
        <button onclick="sendWithFlash()">🚀 LANCER LA TRANSMISSION</button>
        <p class="status">Note: Fonctionne sur Android/Chrome. Sur PC, l'écran flashera à la place.</p>
    </div>

    <div class="card">
        <h2>👁️ RÉCEPTEUR (CAPTEUR)</h2>
        <div class="status">CANAL OPTIQUE : <span id="recv-status">HORS LIGNE</span></div>
        <canvas id="monitor"></canvas>
        <div id="receivedText">_</div>
        <button onclick="startListening()">ACTIVER LA CAMÉRA</button>
    </div>

<script>
    const BIT_TIME = 150; // Vitesse de chaque flash (ms)

    // --- LOGIQUE ÉMETTEUR ---
    async function sendWithFlash() {
        const text = document.getElementById('textToSend').value;
        let binary = "";
        for (let char of text) {
            // On ajoute un '1' de synchro devant chaque lettre
            binary += "1" + char.charCodeAt(0).toString(2).padStart(8, '0');
        }

        try {
            const stream = await navigator.mediaDevices.getUserMedia({ video: { facingMode: 'environment' } });
            const track = stream.getVideoTracks()[0];
            
            for (let bit of binary) {
                // Tente d'allumer la torche (Flash arrière)
                const settings = track.getCapabilities();
                if (settings.torch) {
                    await track.applyConstraints({ advanced: [{ torch: (bit === "1") }] });
                } else {
                    // Fallback : Si pas de flash, on fait flasher l'écran en blanc
                    document.body.style.backgroundColor = (bit === "1") ? "#ffffff" : "#000000";
                }
                await new Promise(r => setTimeout(r, BIT_TIME));
            }

            // Reset
            if (settings.torch) await track.applyConstraints({ advanced: [{ torch: false }] });
            document.body.style.backgroundColor = "#000000";
            track.stop();
        } catch (e) {
            alert("Erreur accès caméra/flash : " + e.message);
        }
    }

    // --- LOGIQUE RÉCEPTEUR ---
    let isReading = false;
    async function startListening() {
        const canvas = document.getElementById('monitor');
        const ctx = canvas.getContext('2d');
        const output = document.getElementById('receivedText');
        const status = document.getElementById('recv-status');

        const stream = await navigator.mediaDevices.getUserMedia({ video: { facingMode: 'user' } });
        const video = document.createElement('video');
        video.srcObject = stream;
        video.play();
        
        isReading = true;
        status.innerText = "ANALYSE EN COURS...";
        status.style.color = "#00ff00";

        let bitBuffer = "";

        // Boucle de lecture cadencée
        setInterval(() => {
            if (!isReading) return;
            
            // On dessine la vidéo dans le canvas pour analyser les pixels
            ctx.drawImage(video, 0, 0, canvas.width, canvas.height);
            const d = ctx.getImageData(0, 0, canvas.width, canvas.height).data;
            
            // Calcul de la luminosité moyenne
            let brightness = 0;
            for (let i = 0; i < d.length; i += 4) {
                brightness += (d[i] + d[i+1] + d[i+2]) / 3;
            }
            let avg = brightness / (d.length / 4);
            
            // Seuil de détection (ajustable selon la lumière ambiante)
            let val = (avg > 140) ? "1" : "0";
            bitBuffer += val;
            
            // On cherche le "1" de départ pour décoder l'octet
            if (bitBuffer.length >= 9) {
                let decodedStr = "";
                for(let i = 0; i <= bitBuffer.length - 9; i++) {
                    if (bitBuffer[i] === "1") {
                        let byte = bitBuffer.substr(i + 1, 8);
                        decodedStr += String.fromCharCode(parseInt(byte, 2));
                        i += 8; // On saute l'octet lu
                    }
                }
                if (decodedStr.length > 0) output.innerText = decodedStr;
                
                // On garde un petit buffer pour la fluidité
                if (bitBuffer.length > 100) bitBuffer = bitBuffer.substring(50);
            }
        }, BIT_TIME);
    }
</script>
</body>
</html>
