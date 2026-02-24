# teest
<!DOCTYPE html>
<html lang="fr">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Sound Data Transfer</title>
    <style>
        body { font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif; max-width: 600px; margin: 40px auto; padding: 20px; line-height: 1.6; background: #f4f7f6; }
        .card { background: white; padding: 20px; border-radius: 8px; box-shadow: 0 2px 10px rgba(0,0,0,0.1); margin-bottom: 20px; }
        textarea { width: 100%; height: 80px; margin: 10px 0; border: 1px solid #ddd; border-radius: 4px; padding: 10px; box-sizing: border-box; }
        button { padding: 10px 20px; background: #007bff; color: white; border: none; border-radius: 4px; cursor: pointer; font-weight: bold; }
        button:hover { background: #0056b3; }
        button.stop { background: #dc3545; }
        #status { font-weight: bold; color: #007bff; }
        #receivedText { background: #e9ecef; padding: 15px; border-radius: 4px; min-height: 50px; word-break: break-all; margin-top: 10px; border-left: 5px solid #007bff; }
    </style>
</head>
<body>
    <h1>📟 Modem Audio JS</h1>
    
    <div class="card">
        <h2>Émetteur</h2>
        <textarea id="textToSend" placeholder="Message à transformer en son..."></textarea>
        <button onclick="sendText()">🔊 Émettre le son</button>
    </div>

    <div class="card">
        <h2>Récepteur</h2>
        <p>Statut : <span id="status">En attente...</span></p>
        <button onclick="startListening()">🎤 Activer le micro</button>
        <button class="stop" onclick="stopListening()">⏹️ Stop</button>
        <div id="received">
            <strong>Texte décodé :</strong>
            <div id="receivedText">-</div>
        </div>
    </div>

    <script>
        const FREQ_0 = 1200; // Fréquence pour le bit '0'
        const FREQ_1 = 1800; // Fréquence pour le bit '1'
        const DURATION = 150; // Durée d'un bit en ms
        
        let audioCtx = null;
        let isListening = false;
        let streamSource = null;

        // --- PARTIE ÉMISSION ---
        function sendText() {
            if (!audioCtx) audioCtx = new (window.AudioContext || window.webkitAudioContext)();
            const text = document.getElementById('textToSend').value;
            if (!text) return;

            // Conversion texte -> Binaire (8 bits par caractère)
            const binary = text.split('').map(char => 
                char.charCodeAt(0).toString(2).padStart(8, '0')
            ).join('');

            console.log("Envoi de :", binary);
            playBit(binary, 0);
        }

        function playBit(binary, index) {
            if (index >= binary.length) return;

            const osc = audioCtx.createOscillator();
            const gain = audioCtx.createGain();
            
            osc.frequency.value = binary[index] === '1' ? FREQ_1 : FREQ_0;
            osc.type = 'sine';
            
            osc.connect(gain);
            gain.connect(audioCtx.destination);
            
            const now = audioCtx.currentTime;
            gain.gain.setValueAtTime(0, now);
            gain.gain.linearRampToValueAtTime(0.2, now + 0.01);
            gain.gain.linearRampToValueAtTime(0, now + (DURATION/1000) - 0.01);
            
            osc.start(now);
            osc.stop(now + DURATION/1000);

            setTimeout(() => playBit(binary, index + 1), DURATION);
        }

        // --- PARTIE RÉCEPTION ---
        let binAccumulator = "";
        let lastBitTime = 0;

        async function startListening() {
            if (!audioCtx) audioCtx = new (window.AudioContext || window.webkitAudioContext)();
            isListening = true;
            document.getElementById('status').innerText = "À l'écoute...";
            
            const stream = await navigator.mediaDevices.getUserMedia({ audio: true });
            streamSource = audioCtx.createMediaStreamSource(stream);
            const analyser = audioCtx.createAnalyser();
            analyser.fftSize = 2048;
            streamSource.connect(analyser);

            const bufferLength = analyser.frequencyBinCount;
            const dataArray = new Uint8Array(bufferLength);

            function checkAudio() {
                if (!isListening) return;
                analyser.getByteFrequencyData(dataArray);

                // On cherche quelle fréquence domine entre FREQ_0 et FREQ_1
                const idx0 = Math.round(FREQ_0 / (audioCtx.sampleRate / analyser.fftSize));
                const idx1 = Math.round(FREQ_1 / (audioCtx.sampleRate / analyser.fftSize));

                const val0 = dataArray[idx0];
                const val1 = dataArray[idx1];

                const now = Date.now();
                if ((val0 > 150 || val1 > 150) && (now - lastBitTime > DURATION)) {
                    const bit = val1 > val0 ? "1" : "0";
                    binAccumulator += bit;
                    lastBitTime = now;
                    decodeBuffer();
                }
                requestAnimationFrame(checkAudio);
            }
            checkAudio();
        }

        function decodeBuffer() {
            if (binAccumulator.length % 8 === 0) {
                let text = "";
                for (let i = 0; i < binAccumulator.length; i += 8) {
                    const byte = binAccumulator.substr(i, 8);
                    text += String.fromCharCode(parseInt(byte, 2));
                }
                document.getElementById('receivedText').innerText = text;
            }
        }

        function stopListening() {
            isListening = false;
            document.getElementById('status').innerText = "Arrêté.";
            if (streamSource) streamSource.mediaStream.getTracks().forEach(t => t.stop());
        }
    </script>
</body>
</html>
