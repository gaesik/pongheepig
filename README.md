<!DOCTYPE html>
<html lang="ko">
<head>
    <meta charset="UTF-8">
    <title>대충 방대방 편하게</title>
    <style>
        /* 🚨 전역 박스 모델 초기화: 테두리와 패딩을 너비 계산에 포함시켜 칼정렬 보증 */
        * {
            box-sizing: border-box;
            margin: 0;
            padding: 0;
        }

        body {
            font-family: 'Malgun Gothic', sans-serif;
            display: flex;
            flex-direction: column;
            align-items: center;
            background-color: #f5f6f7;
            padding-bottom: 50px;
        }

        /* 상단 헤더 */
        header {
            width: 100%;
            background-color: #fff;
            border-bottom: 1px solid #e3e5e8;
            display: flex;
            justify-content: center;
            box-shadow: 0 2px 5px rgba(0,0,0,0.05);
            position: sticky;
            top: 0;
            z-index: 100;
        }
        .nav-container {
            width: 610px;
            display: flex;
            justify-content: flex-start;
        }
        .nav-item {
            padding: 15px 20px;
            font-size: 16px;
            font-weight: bold;
            color: #666;
            cursor: pointer;
            border-bottom: 3px solid transparent;
            transition: all 0.2s;
        }
        .nav-item:hover { color: #03cf5d; }
        .nav-item.active { color: #03cf5d; border-bottom-color: #03cf5d; }

        /* 게임 화면 컨테이너 */
        .main-content { margin-top: 30px; }
        .game-wrapper {
            display: none; 
            background: white;
            padding: 30px;
            border-radius: 8px;
            box-shadow: 0 4px 10px rgba(0,0,0,0.1);
            text-align: center;
            width: 600px; /* 전체 기준 너비 통일 */
        }
        .game-wrapper.active { display: block; }

        /* 공통 스타일 */
        h2 { margin-bottom: 10px; font-size: 24px; color: #333; }
        .info-text { color: #03cf5d; font-size: 15px; font-weight: bold; margin-bottom: 15px; min-height: 20px; }
        .control-btn { border: none; padding: 10px 24px; font-size: 15px; font-weight: bold; border-radius: 4px; cursor: pointer; margin: 0 5px; }
        .start-btn { background-color: #03cf5d; color: white; }
        .start-btn:hover { background-color: #02b34f; }
        .reset-btn { background-color: #333; color: white; }
        .reset-btn:hover { background-color: #111; }
        button:disabled { background-color: #ccc !important; cursor: not-allowed; }

        /* 1️⃣ 사다리타기 정렬 특화 스타일 */
        .setup-area { margin-bottom: 20px; display: flex; align-items: center; justify-content: center; gap: 15px; }
        .count-btn { background-color: #fff; border: 1px solid #ccc; color: #333; width: 35px; height: 35px; font-size: 18px; cursor: pointer; border-radius: 4px; display: flex; align-items: center; justify-content: center; }
        .count-btn:hover { background-color: #f0f0f0; }
        .count-display { font-size: 18px; font-weight: bold; }
        
        /* 🎯 캔버스 너비와 100% 일치시켜 부모 영역 오차 제거 */
        .inputs { 
            display: flex; 
            width: 540px; 
            margin: 0 auto; 
        }
        
        /* 🎯 자바스크립트로 생성된 인원수 칸들이 균등 비율(N등분)을 완벽히 나눠 갖도록 설계 */
        .input-wrapper {
            flex: 1;
            display: flex;
            justify-content: center;
            align-items: center;
            padding: 0 4px; 
        }

        .player-box, .reward-box { 
            width: 100%; 
            max-width: 80px; /* 인원이 적을 때 양옆으로 너무 뚱뚱해지지 않도록 최대치 제한 */
            text-align: center; 
            border-radius: 4px; 
            font-weight: bold; 
            font-size: 14px;
            height: 36px;
        }
        .player-box { border: 1px solid #ddd; background-color: #fff; padding: 5px; }
        .player-box.ready-to-click { border: 2px solid #03cf5d; cursor: pointer; transition: all 0.2s; }
        .player-box.ready-to-click:hover { background-color: #f0fff5; transform: translateY(-3px); }
        .player-box.done { background-color: #e9ecef; border-color: #ccc; color: #6c757d; cursor: default; transform: none; }
        .reward-box { border: 1px solid #ddd; background-color: #fff; padding: 5px; }
        
        /* 🎯 캔버스 중앙 정렬 및 여백 보정 (너비를 540으로 고정 배정) */
        canvas { 
            background-color: #fff; 
            margin: 10px auto; 
            display: block; 
        }
        .btn-group { margin-top: 20px; }

        /* 2️⃣ 팀원 섞기 스타일 */
        .team-container { text-align: left; }
        .team-input-grid { display: grid; grid-template-columns: 1fr 1fr; gap: 20px; margin-bottom: 15px; }
        .team-input-block { display: flex; flex-direction: column; }
        .textarea-label { font-weight: bold; display: block; margin-bottom: 8px; font-size: 14px; color: #333; }
        .team1-border { border: 1px solid #ff4d4d !important; }
        .team2-border { border: 1px solid #4d79ff !important; }
        textarea { width: 100%; height: 150px; padding: 10px; box-sizing: border-box; border-radius: 4px; resize: none; font-size: 14px; }
        .team-results { margin-top: 20px; display: grid; grid-template-columns: repeat(2, 1fr); gap: 20px; }
        .team-card { padding: 15px; border-radius: 6px; box-shadow: 0 2px 5px rgba(0,0,0,0.02); }
        .team-card.t1 { background: #fff5f5; border: 1px solid #ff4d4d; }
        .team-card.t1 h4 { margin: 0 0 10px 0; color: #d63031; border-bottom: 1px solid #ffcccc; padding-bottom: 5px; }
        .team-card.t2 { background: #f5f7ff; border: 1px solid #4d79ff; }
        .team-card.t2 h4 { margin: 0 0 10px 0; color: #0984e3; border-bottom: 1px solid #ccccff; padding-bottom: 5px; }
        .team-card ol, .tier-card ol { margin: 0; padding-left: 20px; font-size: 14px; line-height: 1.8; font-weight: bold; color: #444; }

        /* 3️⃣ 티어 순서 정하기 전용 스타일 */
        .tier-border { border: 1px solid #9b59b6 !important; }
        .tier-card { background: #fbf5ff; border: 1px solid #9b59b6; padding: 20px; border-radius: 6px; text-align: left; margin-top: 20px; }
        .tier-card h4 { margin: 0 0 12px 0; color: #8e44ad; border-bottom: 1px solid #e8d4f7; padding-bottom: 8px; font-size: 16px; }
        .tier-card ol li { font-size: 15px; margin-bottom: 5px; color: #2c3e50; }
        .tier-card ol li span { color: #8e44ad; font-weight: 900; margin-right: 5px; }
    </style>
</head>
<body>

<header>
    <div class="nav-container">
        <div class="nav-item active" onclick="switchGame('ladder', this)">사다리타기</div>
        <div class="nav-item" onclick="switchGame('team', this)">올 랜덤 올킬전</div>
        <div class="nav-item" onclick="switchGame('tier', this)">티어 출전 순서</div>
    </div>
</header>

<div class="main-content">
    
    <!-- 1️⃣ 사다리타기 게임 영역 -->
    <div id="game-ladder" class="game-wrapper active">
        <h2>사다리 타기</h2>
        <div class="setup-area">
            <span>참여 인원</span>
            <button id="minus-btn" class="count-btn" onclick="changeCount(-1)">-</button>
            <span id="count-txt" class="count-display">4</span>
            <button id="plus-btn" class="count-btn" onclick="changeCount(1)">+</button>
        </div>
        <p id="info-msg" class="info-text">설정을 완료한 후 '사다리 시작' 버튼을 눌러주세요.</p>
        
        <!-- 상단 이름 입력 영역 -->
        <div id="player-inputs" class="inputs"></div>
        
        <!-- 중앙 사다리 캔버스 (N등분 나눗셈이 딱 떨어지도록 너비를 540으로 조정) -->
        <canvas id="ladderCanvas" width="540" height="400"></canvas>
        
        <!-- 하단 결과 입력 영역 -->
        <div id="reward-inputs" class="inputs"></div>
        
        <div class="btn-group">
            <button id="start-game-btn" class="control-btn start-btn" onclick="lockAndStart()">사다리 시작</button>
            <button class="control-btn reset-btn" onclick="resetLadder()">새 판하기 (리셋)</button>
        </div>
    </div>

    <!-- 2️⃣ 팀별 순서 섞기 게임 영역 -->
    <div id="game-team" class="game-wrapper">
        <h2>순서 랜덤 올킬전</h2>
        <p class="info-text" style="color: #4d79ff;">팀 간 멤버 이동 없이, 각 팀 내부의 순서만 무작위로 섞습니다.</p>
        
        <div class="team-container">
            <div class="team-input-grid">
                <div class="team-input-block">
                    <span class="textarea-label" style="color: #d63031;">🔴 1팀 명단 입력</span>
                    <textarea id="team1-members" class="team1-border" placeholder="줄바꿈이나 콤마로 구분&#13;&#10;예시:&#13;&#10;퐁희&#13;&#10;늙은이&#13;&#10;돼지"></textarea>
                </div>
                <div class="team-input-block">
                    <span class="textarea-label" style="color: #0984e3;">🔵 2팀 명단 입력</span>
                    <textarea id="team2-members" class="team2-border" placeholder="줄바꿈이나 콤마로 구분&#13;&#10;예시:&#13;&#10;퐁희&#13;&#10;멧돼지&#13;&#10;틀딱"></textarea>
                </div>
            </div>
            <div style="text-align: center; margin-top: 20px;">
                <button class="control-btn start-btn" style="background-color: #4d79ff;" onclick="shuffleEachTeam()">각자 순서 섞기 시작!</button>
                <button class="control-btn reset-btn" onclick="resetTeams()">초기화</button>
            </div>
            <div id="team-results-box" class="team-results"></div>
        </div>
    </div>

    <!-- 3️⃣ 티어 순서 정하기 게임 영역 -->
    <div id="game-tier" class="game-wrapper">
        <h2>방대방 출전 티어 순서</h2>
        <p class="info-text" style="color: #9b59b6;">경기에 먼저 나갈 티어의 우선순위를 무작위로 추첨합니다.</p>
        
        <div class="team-container">
            <span class="textarea-label" style="color: #8e44ad;">🎖️ 대상 티어 목록 입력</span>
            <textarea id="tier-list" class="tier-border" style="height: 120px;" placeholder="줄바꿈이나 콤마로 티어를 구분하여 적어주세요.&#13;&#10;예시:&#13;&#10;1티어&#13;&#10;2티어&#13;&#10;3티어&#13;&#10;4티어"></textarea>
            
            <div style="text-align: center; margin-top: 20px;">
                <button class="control-btn start-btn" style="background-color: #9b59b6;" onclick="shuffleTiers()">티어 순서 섞기!</button>
                <button class="control-btn reset-btn" onclick="resetTiers()">초기화</button>
            </div>
            <div id="tier-results-box"></div>
        </div>
    </div>

</div>

<script>
    /* -------------------------------------------
       🌐 헤더 메뉴 전환 로직
    ------------------------------------------- */
    function switchGame(gameId, element) {
        if (typeof isDrawing !== 'undefined' && isDrawing) {
            alert("현재 사다리 타기가 진행 중입니다. 끝난 후 이동해 주세요!");
            return;
        }
        document.querySelectorAll('.nav-item').forEach(item => item.classList.remove('active'));
        element.classList.add('active');
        document.querySelectorAll('.game-wrapper').forEach(wrapper => wrapper.classList.remove('active'));
        document.getElementById('game-' + gameId).classList.add('active');
    }

    // 공통 믹스 기능 함수들
    function shuffleArray(array) {
        for (let i = array.length - 1; i > 0; i--) {
            const j = Math.floor(Math.random() * (i + 1));
            [array[i], array[j]] = [array[j], array[i]];
        }
        return array;
    }
    function parseNames(text) {
        return text.split(/[\s,\n]+/).filter(name => name.trim() !== "");
    }

    /* -------------------------------------------
       👥 2번 메뉴: 팀별 독립 순서 섞기 로직
    ------------------------------------------- */
    function shuffleEachTeam() {
        const t1Text = document.getElementById('team1-members').value.trim();
        const t2Text = document.getElementById('team2-members').value.trim();
        if (!t1Text && !t2Text) { alert("최소 한 개 팀 이상의 명단을 입력해 주세요."); return; }

        let team1Members = parseNames(t1Text);
        let team2Members = parseNames(t2Text);

        team1Members = shuffleArray(team1Members);
        team2Members = shuffleArray(team2Members);

        const resultsBox = document.getElementById('team-results-box');
        resultsBox.innerHTML = ''; 

        if (team1Members.length > 0) {
            let t1HTML = `<div class="team-card t1"><h4>🔴 1팀 섞기 결과 (${team1Members.length}명)</h4><ol>`;
            team1Members.forEach(name => { t1HTML += `<li>${name}</li>`; });
            t1HTML += `</ol></div>`;
            resultsBox.innerHTML += t1HTML;
        } else { resultsBox.innerHTML += `<div></div>`; }

        if (team2Members.length > 0) {
            let t2HTML = `<div class="team-card t2"><h4>🔵 2팀 섞기 결과 (${team2Members.length}명)</h4><ol>`;
            team2Members.forEach(name => { t2HTML += `<li>${name}</li>`; });
            t2HTML += `</ol></div>`;
            resultsBox.innerHTML += t2HTML;
        }
    }
    function resetTeams() {
        document.getElementById('team1-members').value = '';
        document.getElementById('team2-members').value = '';
        document.getElementById('team-results-box').innerHTML = '';
    }

    /* -------------------------------------------
       🎖️ 3번 메뉴: 티어 순서 섞기 로직
    ------------------------------------------- */
    function shuffleTiers() {
        const tierText = document.getElementById('tier-list').value.trim();
        if (!tierText) { alert("순서를 정할 티어 목록을 입력해 주세요."); return; }

        let tiers = parseNames(tierText);
        tiers = shuffleArray(tiers);

        const tierResultBox = document.getElementById('tier-results-box');
        let resultHTML = `<div class="tier-card"><h4>🔮 티어별 출전 순서 추첨 결과</h4><ol>`;
        tiers.forEach((tierName, index) => {
            resultHTML += `<li><span>[${index + 1}순위]</span> ${tierName}</li>`;
        });
        resultHTML += `</ol></div>`;
        tierResultBox.innerHTML = resultHTML;
    }
    function resetTiers() {
        document.getElementById('tier-list').value = '';
        document.getElementById('tier-results-box').innerHTML = '';
    }

    /* -------------------------------------------
       🪜 1번 메뉴: 사다리타기 로직 (칼정렬 고도화)
    ------------------------------------------- */
    const canvas = document.getElementById('ladderCanvas');
    const ctx = canvas.getContext('2d');
    const height = 10; let count = 4; let stepX, stepY, lines;
    let isGameStarted = false; let isDrawing = false;     

    const defaultPlayers = ["철수", "영희", "민수", "짱구", "훈이", "맹구"];
    const defaultRewards = ["통과", "꽝", "통과", "간식", "꽝", "독식"];

    function init() {
        stepX = canvas.width / count; 
        stepY = canvas.height / (height + 1);
        const playerArea = document.getElementById('player-inputs');
        const rewardArea = document.getElementById('reward-inputs');
        playerArea.innerHTML = ''; rewardArea.innerHTML = '';
        
        // 🎯 정렬 핵심: 부모 flex 컨테이너 안에서 각 칸을 전용 wrapper div로 감싸 픽셀 비틀림 완벽 차단
        for(let i=0; i<count; i++) {
            playerArea.innerHTML += `<div class="input-wrapper"><input type="text" class="player-box" value="${defaultPlayers[i]}" onclick="selectPlayer(${i}, this)"></div>`;
            rewardArea.innerHTML += `<div class="input-wrapper"><input type="text" class="reward-box" value="${defaultRewards[i]}"></div>`;
        }
        lines = [];
        for (let h = 0; h < height; h++) {
            let row = [];
            for (let w = 0; w < count - 1; w++) {
                if (w > 0 && row[w-1]) { row.push(false); } 
                else { row.push(Math.random() > 0.4); }
            }
            lines.push(row);
        }
        drawBaseLadder();
    }

    function changeCount(amount) {
        if (isGameStarted) return; 
        let newCount = count + amount;
        if (newCount < 2 || newCount > 6) return; 
        count = newCount;
        document.getElementById('count-txt').innerText = count;
        document.getElementById('minus-btn').disabled = (count === 2);
        document.getElementById('plus-btn').disabled = (count === 6);
        init(); 
    }

    function drawBaseLadder() {
        ctx.clearRect(0, 0, canvas.width, canvas.height);
        ctx.strokeStyle = '#e0e0e0'; ctx.lineWidth = 4; ctx.lineCap = 'round';
        for (let i = 0; i < count; i++) {
            const x = stepX * i + (stepX / 2);
            ctx.beginPath(); ctx.moveTo(x, 0); ctx.lineTo(x, canvas.height); ctx.stroke();
        }
        for (let h = 0; h < height; h++) {
            const y = stepY * (h + 1);
            for (let w = 0; w < count - 1; w++) {
                if (lines[h][w]) {
                    const x1 = stepX * w + (stepX / 2);
                    const x2 = stepX * (w + 1) + (stepX / 2);
                    ctx.beginPath(); ctx.moveTo(x1, y); ctx.lineTo(x2, y); ctx.stroke();
                }
            }
        }
    }

    function lockAndStart() {
        isGameStarted = true;
        document.getElementById('info-msg').innerText = "💡 확인하고 싶은 사람의 이름을 클릭하세요!";
        document.getElementById('start-game-btn').disabled = true;
        document.getElementById('minus-btn').disabled = true;
        document.getElementById('plus-btn').disabled = true;
        document.querySelectorAll('#player-inputs input').forEach(input => { input.readOnly = true; input.classList.add('ready-to-click'); });
        document.querySelectorAll('#reward-inputs input').forEach(input => input.readOnly = true);
    }

    function selectPlayer(index, element) {
        if (!isGameStarted || isDrawing || element.classList.contains('done')) return;
        isDrawing = true;
        element.classList.remove('ready-to-click'); element.classList.add('done'); 
        const colors = ['#ff4d4d', '#4d79ff', '#2ecc71', '#f1c40f', '#9b59b6', '#e67e22'];
        animatePlayer(index, colors[index % colors.length]);
    }

    function animatePlayer(playerIndex, color) {
        let currentXIdx = playerIndex; let currentYIdx = 0;
        let currentX = stepX * currentXIdx + (stepX / 2); let currentY = 0;
        ctx.strokeStyle = color; ctx.lineWidth = 6;
        function move() {
            if (currentYIdx >= height) {
                ctx.beginPath(); ctx.moveTo(currentX, currentY); ctx.lineTo(currentX, canvas.height); ctx.stroke();
                const rewards = document.querySelectorAll('#reward-inputs input');
                const finalResult = rewards[currentXIdx].value;
                const playerName = document.querySelectorAll('#player-inputs input')[playerIndex].value;
                setTimeout(() => { alert(`${playerName}님의 결과: [ ${finalResult} ]`); isDrawing = false; }, 200);
                return;
            }
            let nextY = stepY * (currentYIdx + 1);
            ctx.beginPath(); ctx.moveTo(currentX, currentY); ctx.lineTo(currentX, nextY); ctx.stroke();
            currentY = nextY;
            setTimeout(() => {
                if (currentXIdx < count - 1 && lines[currentYIdx][currentXIdx]) {
                    let nextX = stepX * (currentXIdx + 1) + (stepX / 2);
                    ctx.beginPath(); ctx.moveTo(currentX, currentY); ctx.lineTo(nextX, currentY);
                    ctx.stroke(); currentX = nextX; currentXIdx++;
                } else if (currentXIdx > 0 && lines[currentYIdx][currentXIdx - 1]) {
                    let nextX = stepX * (currentXIdx - 1) + (stepX / 2);
                    ctx.beginPath(); ctx.moveTo(currentX, currentY); ctx.lineTo(nextX, currentY);
                    ctx.stroke(); currentX = nextX; currentXIdx--;
                }
                currentYIdx++; setTimeout(move, 60); 
            }, 60);
        }
        move();
    }

    function resetLadder() {
        if (isDrawing) return; 
        isGameStarted = false;
        document.getElementById('info-msg').innerText = "설정을 완료한 후 '사다리 시작' 버튼을 눌러주세요.";
        document.getElementById('start-game-btn').disabled = false;
        document.getElementById('minus-btn').disabled = (count === 2);
        document.getElementById('plus-btn').disabled = (count === 6);
        document.querySelectorAll('#player-inputs input').forEach(input => { input.readOnly = false; input.classList.remove('ready-to-click', 'done'); });
        document.querySelectorAll('#reward-inputs input').forEach(input => input.readOnly = false);
        init();
    }

    window.onload = init;
</script>

</body>
</html>
