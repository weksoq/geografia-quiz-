<!-- <!DOCTYPE html> -->
<html lang="pl">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>GeoMaster Polska | Quiz Topograficzny</title>
    <link rel="stylesheet" href="https://unpkg.com/leaflet@1.9.4/dist/leaflet.css" />
    <link href="https://fonts.googleapis.com/css2?family=Inter:wght@400;600;800&display=swap" rel="stylesheet">
    <style>
        :root {
            --primary: #6366f1;
            --primary-hover: #4f46e5;
            --success: #10b981;
            --danger: #ef4444;
            --bg: #f1f5f9;
            --text: #1e293b;
        }

        body {
            font-family: 'Inter', sans-serif;
            background: radial-gradient(circle at top right, #e2e8f0, #cbd5e1);
            color: var(--text);
            margin: 0;
            display: flex;
            justify-content: center;
            align-items: center;
            min-height: 100vh;
        }

        /* LOGOWANIE */
        #login-card {
            background: white;
            padding: 3rem;
            border-radius: 2rem;
            box-shadow: 0 25px 50px -12px rgba(0, 0, 0, 0.15);
            text-align: center;
            width: 100%;
            max-width: 400px;
            border: 1px solid rgba(255,255,255,0.7);
        }

        /* KONTENER GRY */
        #game-container {
            display: none;
            width: 95%;
            max-width: 1300px;
            padding: 20px;
            animation: slideUp 0.5s ease-out;
        }

        @keyframes slideUp { from { opacity: 0; transform: translateY(20px); } to { opacity: 1; transform: translateY(0); } }

        .grid {
            display: grid;
            grid-template-columns: 1fr 380px;
            gap: 25px;
        }

        @media (max-width: 1024px) { .grid { grid-template-columns: 1fr; } }

        .panel {
            background: rgba(255, 255, 255, 0.9);
            backdrop-filter: blur(10px);
            padding: 25px;
            border-radius: 1.5rem;
            box-shadow: 0 10px 15px -3px rgba(0, 0, 0, 0.1);
        }

        /* NAG艁脫WEK */
        .header {
            display: flex;
            justify-content: space-between;
            align-items: center;
            margin-bottom: 20px;
            background: white;
            padding: 20px;
            border-radius: 1rem;
            box-shadow: inset 0 2px 4px rgba(0,0,0,0.05);
        }

        #target-city {
            font-size: 2.2rem;
            font-weight: 800;
            color: var(--primary);
            margin: 0;
            letter-spacing: -1px;
        }

        #timer {
            font-family: 'Courier New', monospace;
            font-size: 1.8rem;
            font-weight: 700;
            color: #475569;
        }

        /* MAPA */
        #map {
            height: 650px;
            width: 100%;
            border-radius: 1rem;
            border: 5px solid white;
            box-shadow: 0 4px 6px -1px rgba(0,0,0,0.1);
            cursor: crosshair;
        }

        /* PRZYCISKI */
        .btn {
            background: var(--primary);
            color: white;
            border: none;
            padding: 12px 24px;
            border-radius: 12px;
            font-weight: 600;
            cursor: pointer;
            transition: all 0.2s;
        }

        .btn:hover { background: var(--primary-hover); transform: translateY(-2px); }
        .btn-start { background: var(--success); font-size: 1.1rem; margin-bottom: 15px; }

        /* RANKING */
        table { width: 100%; border-collapse: separate; border-spacing: 0 8px; }
        th { text-align: left; font-size: 0.75rem; color: #64748b; text-transform: uppercase; padding: 0 10px; }
        td { padding: 14px 10px; background: white; font-size: 0.95rem; transition: transform 0.2s; }
        tr:hover td { transform: scale(1.02); background: #f8fafc; }
        td:first-child { border-radius: 10px 0 0 10px; font-weight: 600; }
        td:last-child { border-radius: 0 10px 10px 0; text-align: right; }
        .score-cell { color: var(--primary); font-weight: 800; text-align: center; }

        .spinning { animation: rotate 1s linear infinite; display: inline-block; }
        @keyframes rotate { from { transform: rotate(0deg); } to { transform: rotate(360deg); } }

        input {
            width: 100%;
            padding: 12px;
            margin: 10px 0;
            border: 2px solid #e2e8f0;
            border-radius: 10px;
            font-size: 1rem;
            box-sizing: border-box;
        }
    </style>
</head>
<body>

<div id="login-card">
    <div style="font-size: 4rem; margin-bottom: 10px;">馃實</div>
    <h2 style="margin: 0 0 20px 0;">GeoMaster Polska</h2>
    <input type="text" id="nick-input" placeholder="Tw贸j Nick">
    <input type="password" id="pass-input" placeholder="Has艂o dost臋pu" onkeydown="if(event.key==='Enter') checkAccess()">
    <button class="btn" style="width: 100%; margin-top: 10px;" onclick="checkAccess()">Zaloguj i Graj</button>
</div>

<div id="game-container">
    <div class="grid">
        <div class="panel">
            <div class="header">
                <div>
                    <h1 id="target-city">GOTOWY?</h1>
                    <div id="timer">00:00</div>
                </div>
                <div style="text-align: right;">
                    <div id="stats" style="font-size: 1.2rem; font-weight: 700;">Pkt: 0 | Pozosta艂o: 0</div>
                    <small id="player-display" style="color: var(--primary); font-weight: 600;"></small>
                </div>
            </div>
            
            <button class="btn btn-start" onclick="startGame()">馃殌 Rozpocznij Quiz</button>
            <div id="map"></div>
        </div>

        <div id="side-bar">
            <div class="panel">
                <div style="display: flex; justify-content: space-between; align-items: center; margin-bottom: 15px;">
                    <h3 style="margin: 0;">馃弳 Ranking Top 10</h3>
                    <button onclick="fetchRanking()" style="background:none; border:none; cursor:pointer; font-size:1.4rem;" id="refresh-btn">馃攧</button>
                </div>
                <table>
                    <thead>
                        <tr>
                            <th>Gracz</th>
                            <th style="text-align: center;">Pkt</th>
                            <th style="text-align: right;">Czas</th>
                        </tr>
                    </thead>
                    <tbody id="rank-body">
                        </tbody>
                </table>
                <div id="rank-status" style="font-size: 0.8rem; color: #94a3b8; margin-top: 10px; text-align: center;"></div>
            </div>
        </div>
    </div>
</div>

<script src="https://unpkg.com/leaflet@1.9.4/dist/leaflet.js"></script>
<script>
    // --- KONFIGURACJA ---
    const SCRIPT_URL = "https://script.google.com/macros/s/AKfycbwxQwIOPMsDltiUXNvkS_0JIp8D43NldCw9WJKTyn-QaYs43zm5OJoy75IqIQRMrYQ9/exec"; 
    const secret = "UG9sc2thMjAyNA=="; // Has艂o: Polska2024

    let currentPlayer = "", score = 0, remainingCities = [], markers = [], currentCity = null, startTime, timerInterval, finalTimeStr;

    const cities = [
        {n: "Szczecin", lat: 53.4285, lon: 14.5528}, {n: "S艂upsk", lat: 54.4641, lon: 17.0285},
        {n: "Ko艂obrzeg", lat: 54.1757, lon: 15.5833}, {n: "Gda艅sk", lat: 54.3520, lon: 18.6466},
        {n: "Gdynia", lat: 54.5189, lon: 18.5305}, {n: "Sopot", lat: 54.4416, lon: 18.5601},
        {n: "Elbl膮g", lat: 54.1561, lon: 19.4045}, {n: "Olsztyn", lat: 53.7784, lon: 20.4801},
        {n: "Suwa艂ki", lat: 54.1115, lon: 22.9307}, {n: "Bia艂ystok", lat: 53.1325, lon: 23.1688},
        {n: "P艂ock", lat: 52.5464, lon: 19.7065}, {n: "Warszawa", lat: 52.2297, lon: 21.0122},
        {n: "Radom", lat: 51.4027, lon: 21.1471}, {n: "Bia艂a Podlaska", lat: 52.0333, lon: 23.1167},
        {n: "Lublin", lat: 51.2465, lon: 22.5684}, {n: "Zamo艣膰", lat: 50.7231, lon: 23.2520},
        {n: "Tarnobrzeg", lat: 50.5744, lon: 21.6811}, {n: "Rzesz贸w", lat: 50.0413, lon: 21.9990},
        {n: "Kielce", lat: 50.8661, lon: 20.6286}, {n: "Krak贸w", lat: 50.0647, lon: 19.9450},
        {n: "Nowy S膮cz", lat: 49.6218, lon: 20.6973}, {n: "Cz臋stochowa", lat: 50.8118, lon: 19.1203},
        {n: "Katowice", lat: 50.2649, lon: 19.0238}, {n: "Bielsko-Bia艂a", lat: 49.8225, lon: 19.0444},
        {n: "Opole", lat: 50.6751, lon: 17.9213}, {n: "Wroc艂aw", lat: 51.1079, lon: 17.0385},
        {n: "Wa艂brzych", lat: 50.7718, lon: 16.2842}, {n: "Jelenia G贸ra", lat: 50.9044, lon: 15.7276},
        {n: "Legnica", lat: 51.2070, lon: 16.1553}, {n: "Zielona G贸ra", lat: 51.9355, lon: 15.5062},
        {n: "Gorz贸w Wlkp.", lat: 52.7325, lon: 15.2369}, {n: "Pozna艅", lat: 52.4064, lon: 16.9252},
        {n: "Kalisz", lat: 51.7611, lon: 18.0910}, {n: "Konin", lat: 52.2234, lon: 18.2512},
        {n: "Bydgoszcz", lat: 53.1235, lon: 18.0084}, {n: "Toru艅", lat: 53.0138, lon: 18.5984},
        {n: "W艂oc艂awek", lat: 52.6484, lon: 19.0678}, {n: "Skierniewice", lat: 51.9559, lon: 20.1456},
        {n: "Piotrk贸w Tryb.", lat: 51.4052, lon: 19.7032}, {n: "Sieradz", lat: 51.5960, lon: 18.7303},
        {n: "艁贸d藕", lat: 51.7592, lon: 19.4560}, {n: "Pabianice", lat: 51.6635, lon: 19.3571}, 
        {n: "Zgierz", lat: 51.8588, lon: 19.4058}, {n: "Ozork贸w", lat: 51.9616, lon: 19.2882}, 
        {n: "Aleksandr贸w 艁贸dzki", lat: 51.8153, lon: 19.3039}, {n: "Konstantyn贸w 艁贸dzki", lat: 51.7479, lon: 19.3248},
        {n: "Gliwice", lat: 50.2945, lon: 18.6714}, {n: "Sosnowiec", lat: 50.2868, lon: 19.1040},
        {n: "Bytom", lat: 50.3481, lon: 18.9328}, {n: "Zabrze", lat: 50.3081, lon: 18.7857},
        {n: "Ruda 艢l膮ska", lat: 50.2575, lon: 18.8550}, {n: "Tychy", lat: 50.1231, lon: 18.9857}
    ];

    let map = L.map('map', { zoomControl: false }).setView([52.1, 19.3], 6);
    L.tileLayer('https://{s}.basemaps.cartocdn.com/light_nolabels/{z}/{x}/{y}{r}.png').addTo(map);
    L.control.zoom({ position: 'bottomright' }).addTo(map);

    function checkAccess() {
        const pass = document.getElementById('pass-input').value;
        const nick = document.getElementById('nick-input').value.trim();
        if (btoa(pass) === secret && nick !== "") {
            currentPlayer = nick;
            document.getElementById('login-card').style.display = 'none';
            document.getElementById('game-container').style.display = 'block';
            document.getElementById('player-display').innerText = "Zalogowano: " + nick;
            fetchRanking();
            setTimeout(() => map.invalidateSize(), 400);
        } else { alert("B艂膮d logowania!"); }
    }

    function startGame() {
        clearInterval(timerInterval);
        markers.forEach(m => map.removeLayer(m));
        markers = []; score = 0;
        remainingCities = [...cities].sort(() => Math.random() - 0.5);
        startTime = Date.now();
        timerInterval = setInterval(() => {
            let elapsed = Math.floor((Date.now() - startTime) / 1000);
            finalTimeStr = Math.floor(elapsed/60).toString().padStart(2,'0') + ":" + (elapsed%60).toString().padStart(2,'0');
            document.getElementById('timer').innerText = finalTimeStr;
        }, 1000);
        nextCity();
    }

    function nextCity() {
        if (remainingCities.length === 0) {
            clearInterval(timerInterval);
            document.getElementById('target-city').innerHTML = "KONIEC!";
            submitScore(currentPlayer, score, finalTimeStr);
            return;
        }
        currentCity = remainingCities.pop();
        document.getElementById('target-city').innerHTML = currentCity.n;
        document.getElementById('stats').innerHTML = `Pkt: ${score} | Pozosta艂o: ${remainingCities.length}`;
    }

    map.on('click', function(e) {
        if (!currentCity) return;
        const dist = map.distance(e.latlng, [currentCity.lat, currentCity.lon]) / 1000;
        const hit = dist < 30;
        const m = L.circleMarker([currentCity.lat, currentCity.lon], { 
            radius: 9, fillColor: hit ? "#10b981" : "#ef4444", color: "white", weight: 3, fillOpacity: 1 
        }).addTo(map);
        markers.push(m);
        if (hit) score++;
        nextCity();
    });

    async function fetchRanking() {
        const btn = document.getElementById('refresh-btn');
        btn.classList.add('spinning');
        try {
            const r = await fetch(SCRIPT_URL);
            const data = await r.json();
            renderRanking(data);
        } catch(e) { console.error("B艂膮d rankingu"); }
        finally { setTimeout(() => btn.classList.remove('spinning'), 600); }
    }

    async function submitScore(n, s, t) {
        document.getElementById('rank-status').innerText = "Wysy艂anie wyniku...";
        try {
            const r = await fetch(SCRIPT_URL, { 
                method: 'POST', body: JSON.stringify({ name: n, score: s, time: t }) 
            });
            const data = await r.json();
            renderRanking(data);
            document.getElementById('rank-status').innerText = "Wynik zapisany!";
        } catch(e) { document.getElementById('rank-status').innerText = "B艂膮d zapisu."; }
    }

    function renderRanking(d) {
        const body = document.getElementById('rank-body');
        if (!Array.isArray(d)) return;
        body.innerHTML = d.map((r, i) => `
            <tr>
                <td>${i+1}. ${r.name.split(',')}</td>
                <td class="score-cell">${r.score}</td>
                <td style="text-align: right;">${r.time}</td>
            </tr>
        `).join('');
    }
</script>
</body>
</html>
