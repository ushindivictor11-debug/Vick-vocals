<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Vick Vocals Pro</title>
    <meta name="mobile-web-app-capable" content="yes">
    <meta name="apple-mobile-web-app-status-bar-style" content="black-translucent">
    <style>
        :root {
            --neon-green: #39ff14;
            --neon-blue: #00f3ff;
            --neon-pink: #ff00ff;
            --dark-bg: #050505;
        }

        /* BRIGHT RHYTHMIC EDGE LIGHTING */
        body::before {
            content: "";
            position: fixed;
            top: 0; left: 0; right: 0; bottom: 0;
            border: 12px solid transparent; /* Thicker for more brightness */
            z-index: 9999;
            pointer-events: none;
        }

        body.playing::before {
            border-image: conic-gradient(from var(--angle), var(--neon-blue), var(--neon-pink), var(--neon-green), var(--neon-blue)) 1;
            animation: rotateBorder 2s linear infinite, glowPulse 1s ease-in-out infinite alternate;
            filter: blur(2px) brightness(2); /* Makes it pop like a club */
        }

        @property --angle { syntax: '<angle>'; initial-value: 0deg; inherits: false; }
        @keyframes rotateBorder { to { --angle: 360deg; } }
        @keyframes glowPulse { from { opacity: 0.6; } to { opacity: 1; } }

        body {
            background-color: var(--dark-bg);
            color: white;
            font-family: 'Segoe UI', sans-serif;
            margin: 0;
            display: flex;
            flex-direction: column;
            align-items: center;
            min-height: 100vh;
            overflow: hidden;
        }

        .container {
            width: 85%;
            max-width: 380px;
            text-align: center;
            padding: 30px;
            margin-top: 40px;
            border-radius: 50px;
            background: rgba(10, 10, 10, 0.95);
            border: 1px solid #333;
            box-shadow: 0 0 50px rgba(0, 243, 255, 0.2);
            z-index: 10;
        }

        h1 { color: var(--neon-blue); text-shadow: 0 0 20px var(--neon-blue); margin-bottom: 5px; }

        /* NEON PROGRESS LINE */
        .progress-box { width: 100%; margin: 20px 0; }
        #seek-bar {
            width: 100%;
            accent-color: var(--neon-green);
            filter: drop-shadow(0 0 8px var(--neon-green));
            cursor: pointer;
        }
        .time-labels {
            display: flex;
            justify-content: space-between;
            font-size: 11px;
            color: var(--neon-blue);
            font-weight: bold;
            margin-top: 5px;
        }

        /* VINYL CLUB VISUALS */
        .visualizer {
            position: relative;
            width: 200px; height: 200px;
            margin: 20px auto;
            display: flex; justify-content: center; align-items: center;
        }
        .beams {
            position: absolute; width: 160%; height: 160%;
            background: conic-gradient(from 0deg, transparent, var(--neon-blue), transparent, var(--neon-pink), transparent, var(--neon-green), transparent);
            filter: blur(40px); opacity: 0; animation: rotate 4s linear infinite;
        }
        .vinyl {
            position: relative; width: 170px; height: 170px; border-radius: 50%;
            background: radial-gradient(circle, #444 2%, #111 25%, #000 100%);
            border: 4px solid #1a1a1a; box-shadow: 0 0 30px var(--neon-green);
            z-index: 5;
        }
        
        .spinning { animation: rotate 3s linear infinite; }
        .shaking { animation: bass 0.1s infinite; }
        @keyframes rotate { to { transform: rotate(360deg); } }
        @keyframes bass { 0% { transform: scale(1); } 50% { transform: scale(1.05); } 100% { transform: scale(1); } }

        /* DASHBOARD */
        .dashboard { display: grid; grid-template-columns: repeat(3, 1fr); gap: 10px; margin-bottom: 20px; }
        .dash-btn { background: #111; border: 1px solid #333; color: #fff; padding: 10px; border-radius: 12px; font-size: 9px; cursor: pointer; }
        .play-btn { width: 70px; height: 70px; border-radius: 50%; background: white; border: none; font-size: 26px; cursor: pointer; }
    </style>
</head>
<body>

<div class="container">
    <h1>VICK VOCALS</h1>
    <div id="track-title" style="font-size: 12px; color: #777;">Load your club tracks</div>

    <div class="visualizer">
        <div class="beams" id="beams"></div>
        <div class="vinyl" id="vinyl"></div>
    </div>

    <div class="progress-box">
        <input type="range" id="seek-bar" value="0" step="0.1">
        <div class="time-labels">
            <span id="cur-time">0:00</span>
            <span id="rem-time">-0:00</span>
        </div>
    </div>

    <div class="dashboard">
        <button class="dash-btn" onclick="document.getElementById('file-in').click()">➕ ADD</button>
        <button class="dash-btn" onclick="setTimer()">⏰ TIMER</button>
        <button class="dash-btn" onclick="shuffle()">🔀 SHUFFLE</button>
        <button class="dash-btn" onclick="toggleLoop()" id="lpBtn">🔁 LOOP: OFF</button>
        <button class="dash-btn" onclick="clearAll()" style="color:red">🗑 CLEAR</button>
        <button class="dash-btn">🎙 RECORD</button>
    </div>

    <input type="file" id="file-in" hidden multiple accept="audio/*">
    <audio id="player"></audio>

    <div style="display: flex; justify-content: space-around; align-items: center; width: 100%;">
        <button class="dash-btn" onclick="prev()" style="font-size: 20px;">⏮</button>
        <button class="play-btn" onclick="playPause()" id="mPlay">▶</button>
        <button class="dash-btn" onclick="next()" style="font-size: 20px;">⏭</button>
    </div>
</div>

<script>
    let list = []; let cur = 0;
    const audio = document.getElementById('player');
    const mPlay = document.getElementById('mPlay');
    const vinyl = document.getElementById('vinyl');
    const beams = document.getElementById('beams');
    const seek = document.getElementById('seek-bar');

    document.getElementById('file-in').onchange = (e) => {
        list = [...list, ...Array.from(e.target.files)];
        if(audio.paused && !audio.src) load(0);
    };

    function load(i) {
        cur = i; audio.src = URL.createObjectURL(list[cur]);
        document.getElementById('track-title').innerText = list[cur].name;
        audio.play(); state(true);
    }

    audio.ontimeupdate = () => {
        if(!isNaN(audio.duration)) {
            seek.max = audio.duration; seek.value = audio.currentTime;
            document.getElementById('cur-time').innerText = fmt(audio.currentTime);
            document.getElementById('rem-time').innerText = "-" + fmt(audio.duration - audio.currentTime);
        }
    };

    seek.oninput = () => audio.currentTime = seek.value;

    function fmt(s) {
        let m = Math.floor(s/60); let sec = Math.floor(s%60);
        return `${m}:${sec < 10 ? '0'+sec : sec}`;
    }

    function playPause() {
        if(list.length === 0) return;
        audio.paused ? (audio.play(), state(true)) : (audio.pause(), state(false));
    }

    function state(p) {
        mPlay.innerText = p ? "⏸" : "▶";
        if(p) { vinyl.classList.add('spinning', 'shaking'); beams.style.opacity = 1; document.body.classList.add('playing'); }
        else { vinyl.classList.remove('spinning', 'shaking'); beams.style.opacity = 0; document.body.classList.remove('playing'); }
    }

    function next() { cur = (cur+1)%list.length; load(cur); }
    function prev() { cur = (cur-1+list.length)%list.length; load(cur); }
    audio.onended = next;

    function clearAll() { if(confirm("Clear Vick Vocals?")) { list=[]; audio.pause(); audio.src=""; state(false); } }
    function toggleLoop() { audio.loop = !audio.loop; document.getElementById('lpBtn').innerText = audio.loop ? "🔂 LOOP: ON" : "🔁 LOOP: OFF"; }
    function shuffle() { list.sort(() => Math.random() - 0.5); load(0); }
    function setTimer() { let m = prompt("Stop in mins:", "30"); if(m) setTimeout(() => { audio.pause(); state(false); }, m*60000); }
</script>
</body>
</html>
