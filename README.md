# NoteLocation
learn every note on your Guitar

<!DOCTYPE html>
<html lang="de">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Note Location Pro</title>
    <style>
        :root {
            --bg: #ffffff;
            --text: #1c1c1e;
            --surface: #f2f2f7;
            --primary: #007aff;
            --border: #d1d1d6;
            --ease: cubic-bezier(0.4, 0, 0.2, 1);
        }

        [data-theme="dark"] {
            --bg: #1c1c1e;
            --text: #f2f2f7;
            --surface: #2c2c2e;
            --primary: #0a84ff;
            --border: #3a3a3c;
        }

        body {
            font-family: -apple-system, system-ui, sans-serif;
            background: var(--bg);
            color: var(--text);
            margin: 0;
            padding: 1rem;
            min-height: 100vh;
            display: flex;
            justify-content: center;
            align-items: center;
            transition: background 0.3s var(--ease);
        }

        .container {
            max-width: 500px;
            width: 100%;
            padding: 2rem;
            background: var(--surface);
            border-radius: 20px;
            box-shadow: 0 8px 24px rgba(0, 0, 0, 0.1);
        }

        h1 {
            font-size: 2rem;
            text-align: center;
            margin-bottom: 1.5rem;
            color: var(--primary);
        }

        .range-inputs {
            display: flex;
            gap: 1rem;
            margin-bottom: 1.5rem;
        }

        .range-inputs input {
            flex: 1;
            padding: 0.8rem;
            border: 1px solid var(--border);
            border-radius: 12px;
            font-size: 1rem;
            text-align: center;
            background: var(--bg);
            color: var(--text);
        }

        .strings-grid {
            display: grid;
            grid-template-columns: repeat(3, 1fr);
            gap: 0.8rem;
            margin-bottom: 1.5rem;
        }

        .string-btn {
            padding: 1rem;
            border: 1px solid var(--border);
            border-radius: 12px;
            background: var(--bg);
            color: var(--text);
            text-align: center;
            cursor: pointer;
            transition: all 0.2s var(--ease);
        }

        .string-btn.active {
            background: var(--primary);
            border-color: var(--primary);
            color: white;
        }

        .note-display {
            font-size: 3rem;
            text-align: center;
            margin: 2rem 0;
            font-weight: bold;
            opacity: 0;
            transform: translateY(10px);
            transition: all 0.3s var(--ease);
        }

        .note-display.visible {
            opacity: 1;
            transform: translateY(0);
        }

        .solution {
            text-align: center;
            margin-top: 1.5rem;
            opacity: 0;
            transform: translateY(10px);
            transition: all 0.3s var(--ease);
        }

        .solution.visible {
            opacity: 1;
            transform: translateY(0);
        }

        .buttons {
            display: flex;
            gap: 1rem;
            justify-content: center;
            margin-top: 2rem;
        }

        .btn {
            padding: 1rem 2rem;
            border: none;
            border-radius: 14px;
            background: var(--primary);
            color: white;
            font-size: 1rem;
            cursor: pointer;
            transition: transform 0.2s var(--ease);
        }

        .btn:hover {
            transform: translateY(-2px);
        }

        .theme-toggle {
            position: fixed;
            top: 1rem;
            right: 1rem;
            background: var(--surface);
            border: none;
            padding: 0.8rem;
            border-radius: 50%;
            cursor: pointer;
            box-shadow: 0 4px 12px rgba(0, 0, 0, 0.1);
        }
    </style>
</head>
<body>
    <button class="theme-toggle" onclick="toggleTheme()">🌓</button>

    <div class="container">
        <h1>Note Location Pro</h1>
        <div class="range-inputs">
            <input type="number" id="minFret" placeholder="Start Bund" min="0" max="12" value="0">
            <input type="number" id="maxFret" placeholder="End Bund" min="1" max="12" value="12">
        </div>

        <div class="strings-grid">
            <div class="string-btn" data-string="E">E</div>
            <div class="string-btn" data-string="A">A</div>
            <div class="string-btn" data-string="D">D</div>
            <div class="string-btn" data-string="G">G</div>
            <div class="string-btn" data-string="B">B</div>
            <div class="string-btn" data-string="e">e</div>
        </div>

        <div class="note-display" id="noteDisplay"></div>
        <div class="solution" id="solution"></div>

        <div class="buttons">
            <button class="btn" onclick="generateNewNote()">Neue Note</button>
            <button class="btn" onclick="playNote()">Ton & Lösung</button>
        </div>
    </div>

    <script>
        const notes = ['C', 'C#', 'D', 'D#', 'E', 'F', 'F#', 'G', 'G#', 'A', 'A#', 'B'];
        const baseNotes = { E: 4, A: 9, D: 2, G: 7, B: 11, e: 4 };
        let currentNote = null;

        // Theme-Wechsel
        function toggleTheme() {
            document.body.setAttribute('data-theme',
                document.body.getAttribute('data-theme') === 'dark' ? 'light' : 'dark'
            );
        }

        // Tonerzeugung mit verbessertem Klang
        function playGuitarSound(freq) {
            const audioCtx = new (window.AudioContext || window.webkitAudioContext)();
            const oscillator = audioCtx.createOscillator();
            const gainNode = audioCtx.createGain();
            
            oscillator.type = 'square';
            oscillator.frequency.setValueAtTime(freq, audioCtx.currentTime);
            
            gainNode.gain.setValueAtTime(0.3, audioCtx.currentTime);
            gainNode.gain.exponentialRampToValueAtTime(0.001, audioCtx.currentTime + 1.5);
            
            oscillator.connect(gainNode);
            gainNode.connect(audioCtx.destination);
            
            oscillator.start();
            oscillator.stop(audioCtx.currentTime + 1.5);
        }

        // Neue Note generieren
        function generateNewNote() {
            const min = parseInt(document.getElementById('minFret').value);
            const max = parseInt(document.getElementById('maxFret').value);
            const selectedStrings = Array.from(document.querySelectorAll('.string-btn.active'))
                                      .map(btn => btn.dataset.string);

            if (selectedStrings.length === 0) {
                alert('Bitte wähle mindestens eine Saite aus!');
                return;
            }

            // Zufällige Note finden, die auf mindestens einer Saite existiert
            let attempts = 0;
            let noteData;
            do {
                const targetNote = notes[Math.floor(Math.random() * notes.length)];
                const positions = [];
                
                selectedStrings.forEach(string => {
                    const base = baseNotes[string];
                    for (let fret = min; fret <= max; fret++) {
                        if (notes[(base + fret) % 12] === targetNote) {
                            positions.push({ string, fret });
                        }
                    }
                });

                if (positions.length > 0) {
                    noteData = { note: targetNote, positions };
                }
            } while (!noteData && attempts++ < 50);

            if (!noteData) {
                alert('Keine Noten gefunden - erweitere den Bundbereich!');
                return;
            }

            currentNote = noteData;
            document.getElementById('noteDisplay').textContent = currentNote.note;
            document.getElementById('noteDisplay').classList.add('visible');
            document.getElementById('solution').classList.remove('visible');
        }

        // Ton abspielen und Lösung anzeigen
        function playNote() {
            if (!currentNote) return;

            // Ton abspielen (mittlere Oktave)
            const noteIndex = notes.indexOf(currentNote.note);
            const freq = 440 * Math.pow(2, (noteIndex - 9) / 12);
            playGuitarSound(freq);

            // Lösungen anzeigen
            const solutionText = currentNote.positions
                .map(pos => `${pos.string}-Saite · Bund ${pos.fret}`)
                .join('<br>');
            
            document.getElementById('solution').innerHTML = solutionText;
            document.getElementById('solution').classList.add('visible');
        }

        // Event-Listener für Saitenauswahl
        document.querySelectorAll('.string-btn').forEach(btn => {
            btn.addEventListener('click', () => {
                btn.classList.toggle('active');
            });
        });
    </script>
</body>
</html>
