jarvis<!DOCTYPE html>
<html lang="de">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>J.A.R.V.I.S. MARK VII</title>
    <style>
        :root {
            --cyan: #00f0ff;
            --amber: #ff9d00;
            --bg: #02060e;
        }

        * {
            box-sizing: border-box;
            user-select: none;
        }

        body {
            background-color: var(--bg);
            color: var(--cyan);
            font-family: 'Segoe UI', Roboto, -apple-system, sans-serif;
            display: flex;
            flex-direction: column;
            align-items: center;
            justify-content: space-between;
            height: 100vh;
            margin: 0;
            padding: 20px;
            overflow: hidden;
        }

        .header {
            text-align: center;
            margin-top: 10px;
        }

        h1 { 
            font-size: 24px; 
            letter-spacing: 5px; 
            text-shadow: 0 0 15px var(--cyan);
            margin: 0;
        }

        .sub-header {
            font-size: 10px;
            letter-spacing: 3px;
            opacity: 0.6;
            margin-top: 4px;
        }

        /* Arc-Reaktor Visualisierung */
        .arc-container {
            position: relative;
            width: 180px;
            height: 180px;
            display: flex;
            align-items: center;
            justify-content: center;
        }

        .arc-reactor {
            width: 140px;
            height: 140px;
            border: 3px solid var(--cyan);
            border-radius: 50%;
            box-shadow: 0 0 30px var(--cyan), inset 0 0 30px var(--cyan);
            display: flex;
            flex-direction: column;
            align-items: center;
            justify-content: center;
            transition: all 0.3s ease;
            background: rgba(0, 240, 255, 0.03);
        }

        .arc-ring {
            position: absolute;
            width: 175px;
            height: 175px;
            border: 2px dashed var(--cyan);
            border-radius: 50%;
            opacity: 0.4;
            animation: spin 18s linear infinite;
        }

        @keyframes spin {
            100% { transform: rotate(360deg); }
        }

        /* Status Zustände */
        .listening .arc-reactor {
            border-color: var(--amber);
            box-shadow: 0 0 40px var(--amber), inset 0 0 30px var(--amber);
            color: var(--amber);
        }

        .listening .arc-ring {
            border-color: var(--amber);
            animation-duration: 5s;
        }

        .speaking .arc-reactor {
            transform: scale(1.08);
            box-shadow: 0 0 50px var(--cyan), inset 0 0 40px var(--cyan);
        }

        #status {
            font-size: 12px;
            font-weight: bold;
            letter-spacing: 2px;
        }

        /* Chat Display */
        .chat-box {
            width: 100%;
            max-width: 500px;
            height: 220px;
            background: rgba(0, 240, 255, 0.03);
            border: 1px solid rgba(0, 240, 255, 0.25);
            border-radius: 12px;
            padding: 15px;
            overflow-y: auto;
            font-size: 13px;
            box-shadow: inset 0 0 20px rgba(0, 0, 0, 0.9);
            backdrop-filter: blur(5px);
        }

        .chat-entry {
            margin-bottom: 10px;
            line-height: 1.4;
            word-wrap: break-word;
        }

        .chat-entry.sys { opacity: 0.5; font-size: 11px; }
        .chat-entry b { letter-spacing: 1px; }

        /* Button Control */
        .btn-talk {
            background: rgba(0, 240, 255, 0.08);
            color: var(--cyan);
            border: 2px solid var(--cyan);
            width: 100%;
            max-width: 500px;
            padding: 16px;
            font-size: 16px;
            font-weight: bold;
            letter-spacing: 2px;
            border-radius: 30px;
            box-shadow: 0 0 20px rgba(0, 240, 255, 0.2);
            cursor: pointer;
            transition: all 0.2s;
            margin-bottom: 10px;
        }

        .btn-talk:active {
            transform: scale(0.98);
        }

        .btn-active {
            background: rgba(255, 157, 0, 0.15);
            border-color: var(--amber);
            color: var(--amber);
            box-shadow: 0 0 20px rgba(255, 157, 0, 0.3);
        }
    </style>
</head>
<body>

    <div class="header">
        <h1>J.A.R.V.I.S.</h1>
        <div class="sub-header">MARK VII - MOBILE OS</div>
    </div>

    <div class="arc-container" id="arcContainer">
        <div class="arc-ring"></div>
        <div class="arc-reactor">
            <span id="status">OFFLINE</span>
        </div>
    </div>

    <div class="chat-box" id="chat">
        <div class="chat-entry sys">System bereit. Tippe auf "SYSTEM STARTEN".</div>
    </div>

    <button class="btn-talk" id="btnTalk">SYSTEM STARTEN</button>

<script>
    const chat = document.getElementById('chat');
    const status = document.getElementById('status');
    const arcContainer = document.getElementById('arcContainer');
    const btnTalk = document.getElementById('btnTalk');

    // GEDÄCHTNIS (Lokaler Speicher im Browser)
    let memory = JSON.parse(localStorage.getItem('jarvis_memory')) || {
        userName: 'Sir',
        history: []
    };

    function saveMemory() {
        localStorage.setItem('jarvis_memory', JSON.stringify(memory));
    }

    // AUDIO SYSTEM (Sci-Fi Signalton)
    const audioCtx = new (window.AudioContext || window.webkitAudioContext)();
    function playBeep(freq = 900) {
        if (audioCtx.state === 'suspended') audioCtx.resume();
        const osc = audioCtx.createOscillator();
        const gain = audioCtx.createGain();
        osc.frequency.value = freq;
        osc.connect(gain);
        gain.connect(audioCtx.destination);
        osc.start();
        gain.gain.exponentialRampToValueAtTime(0.00001, audioCtx.currentTime + 0.12);
        osc.stop(audioCtx.currentTime + 0.12);
    }

    // STIMMEN-AUSWAHL (Tiefe Jarvis-Stimme)
    let jarvisVoice = null;
    function loadVoices() {
        const voices = window.speechSynthesis.getVoices();
        jarvisVoice = voices.find(v => v.lang.includes('de') && (v.name.includes('Stefan') || v.name.includes('Google') || v.name.includes('Yannick'))) 
                      || voices.find(v => v.lang.includes('de'));
    }
    loadVoices();
    if (speechSynthesis.onvoiceschanged !== undefined) {
        speechSynthesis.onvoiceschanged = loadVoices;
    }

    // SPRACHAUSGABE
    function speak(text) {
        window.speechSynthesis.cancel();

        isSpeaking = true;
        status.innerText = 'SPRICHT...';
        arcContainer.classList.add('speaking');
        arcContainer.classList.remove('listening');

        // Mikrofon während der Sprachausgabe pausieren
        try { recognition.stop(); } catch(e){}

        const uttr = new SpeechSynthesisUtterance(text);
        uttr.lang = 'de-DE';
        uttr.pitch = 0.78; // Tiefer Jarvis-Klang
        uttr.rate = 0.95;
        if (jarvisVoice) uttr.voice = jarvisVoice;

        uttr.onend = () => {
            isSpeaking = false;
            arcContainer.classList.remove('speaking');
            if (isSystemActive) {
                status.innerText = 'LAUSCHE...';
                setTimeout(safeStartRecognition, 300);
            }
        };

        uttr.onerror = () => {
            isSpeaking = false;
            arcContainer.classList.remove('speaking');
            if (isSystemActive) safeStartRecognition();
        };

        window.speechSynthesis.speak(uttr);
    }

    function addMessage(sender, text, cls = '') {
        const div = document.createElement('div');
        div.className = `chat-entry ${cls}`;
        div.innerHTML = `<b>${sender}:</b> ${text}`;
        chat.appendChild(div);
        chat.scrollTop = chat.scrollHeight;
    }

    // INTERNET-SUCHE (Wikipedia Live Data)
    async function fetchInternetFact(query) {
        try {
            const res = await fetch(`https://de.wikipedia.org/api/rest_v1/page/summary/${encodeURIComponent(query)}`);
            if (res.ok) {
                const data = await res.json();
                if (data.extract) return data.extract;
            }
        } catch (e) {}
        return null;
    }

    // BEFEHLSVERARBEITUNG
    async function processCommand(command) {
        const lower = command.toLowerCase();
        let reply = "";

        if (lower.includes('mein name ist')) {
            const name = command.split('ist')[1].trim();
            memory.userName = name;
            saveMemory();
            reply = `Verstanden, ${name}. Ich habe Ihre Daten aktualisiert.`;
        }
        else if (lower.includes('wer bin ich')) {
            reply = `Sie sind ${memory.userName}, mein Schöpfer.`;
        }
        else if (lower.includes('uhrzeit') || lower.includes('wie viel uhr') || lower.includes('uhr')) {
            const time = new Date().toLocaleTimeString([], {hour: '2-digit', minute:'2-digit'});
            reply = `Es ist genau ${time} Uhr, ${memory.userName}.`;
        }
        else if (lower.includes('datum') || lower.includes('welcher tag')) {
            const options = { weekday: 'long', year: 'numeric', month: 'long', day: 'numeric' };
            const date = new Date().toLocaleDateString('de-DE', options);
            reply = `Heute ist ${date}.`;
        }
        else if (lower.includes('was ist') || lower.includes('wer ist') || lower.includes('suche nach')) {
            const term = command.replace(/was ist|wer ist|suche nach|jarvis/gi, '').trim();
            addMessage('SYSTEM', `Recherchiere '${term}' im Web...`, 'sys');
            
            const fact = await fetchInternetFact(term);
            if (fact) {
                reply = `Nach meinen Informationen: ${fact.substring(0, 220)}...`;
            } else {
                reply = `Ich habe nach ${term} gesucht, konnte aber keine präzisen Daten finden.`;
            }
        }
        else if (lower.includes('hallo') || lower.includes('hi') || lower.includes('servus')) {
            reply = `Guten Tag, ${memory.userName}. Wie kann ich Ihnen behilflich sein?`;
        }
        else if (lower.includes('status') || lower.includes('system')) {
            reply = `Alle Kernsysteme laufen stabil bei 100 Prozent Leistung, ${memory.userName}.`;
        }
        else {
            reply = `Befehl '${command}' wurde verarbeitet, ${memory.userName}.`;
        }

        addMessage('JARVIS', reply);
        speak(reply);
    }

    // SPRACHERKENNUNG & WATCHDOG SETUP
    const SpeechRecognition = window.SpeechRecognition || window.webkitSpeechRecognition;

    if (!SpeechRecognition) {
        addMessage('SYSTEM', 'Fehler: Nutzung in Google Chrome wird empfohlen.', 'sys');
    } else {
        const recognition = new SpeechRecognition();
        recognition.lang = 'de-DE';
        recognition.continuous = false;
        recognition.interimResults = false;

        let isSystemActive = false;
        let isSpeaking = false;
        let isRecognizing = false;
        let watchdogInterval = null;

        function safeStartRecognition() {
            if (!isSystemActive || isSpeaking || isRecognizing) return;
            try {
                recognition.start();
            } catch (e) {}
        }

        recognition.onstart = () => {
            isRecognizing = true;
            if (!isSpeaking) status.innerText = 'LAUSCHE...';
        };

        recognition.onresult = async (event) => {
            isRecognizing = false;
            const userText = event.results[0][0].transcript.trim();
            const lowerText = userText.toLowerCase();

            if (lowerText.includes('jarvis')) {
                playBeep(900);
                arcContainer.classList.add('listening');

                const cleanCommand = userText.replace(/jarvis/gi, '').trim();

                if (cleanCommand.length > 0) {
                    addMessage(memory.userName.toUpperCase(), cleanCommand);
                    await processCommand(cleanCommand);
                } else {
                    const reply = `Ja, ${memory.userName}?`;
                    addMessage('JARVIS', reply);
                    speak(reply);
                }

                setTimeout(() => arcContainer.classList.remove('listening'), 1200);
            } else {
                safeStartRecognition();
            }
        };

        recognition.onerror = () => {
            isRecognizing = false;
            if (isSystemActive && !isSpeaking) {
                setTimeout(safeStartRecognition, 400);
            }
        };

        recognition.onend = () => {
            isRecognizing = false;
            if (isSystemActive && !isSpeaking) {
                setTimeout(safeStartRecognition, 300);
            }
        };

        // BUTTON STEUERUNG
        btnTalk.addEventListener('click', () => {
            if (!isSystemActive) {
                isSystemActive = true;
                btnTalk.innerText = 'SYSTEM BEENDEN';
                btnTalk.classList.add('btn-active');
                status.innerText = 'LAUSCHE...';
                
                playBeep(600);
                addMessage('SYSTEM', `System gestartet. Willkommen, ${memory.userName}. Sagen Sie "Jarvis"...`, 'sys');
                safeStartRecognition();

                // WATCHDOG TIMER (Verhindert Einschlafen auf Smartphones)
                watchdogInterval = setInterval(() => {
                    if (isSystemActive && !isSpeaking && !isRecognizing) {
                        status.innerText = 'LAUSCHE...';
                        safeStartRecognition();
                    }
                }, 2000);

            } else {
                isSystemActive = false;
                clearInterval(watchdogInterval);
                window.speechSynthesis.cancel();
                try { recognition.stop(); } catch(e){}
                
                status.innerText = 'OFFLINE';
                btnTalk.innerText = 'SYSTEM STARTEN';
                btnTalk.classList.remove('btn-active');
                arcContainer.classList.remove('listening', 'speaking');
                addMessage('SYSTEM', 'System in Standby versetzt.', 'sys');
            }
        });
    }
</script>
</body>
</html>
