<!DOCTYPE html>
<html lang="bn">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Draggable Signal Overlay with Groq AI Brain</title>
    <style>
        body, html {
            margin: 0;
            padding: 0;
            width: 100%;
            height: 100%;
            overflow: hidden;
            font-family: Arial, sans-serif;
        }
        /* ব্যাকগ্রাউন্ডে ওয়েবসাইট দেখানোর জন্য iframe */
        .website-frame {
            width: 100%;
            height: 100%;
            border: none;
        }

        /* ওভারলে কন্টেইনার */
        .overlay-container {
            position: fixed;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            pointer-events: none;
            display: flex;
            justify-content: center;
            align-items: flex-start;
            padding-top: 40px;
            z-index: 99999;
            display: none; /* পাসওয়ার্ড সঠিক দিলে শো করবে */
        }

        /* পাসওয়ার্ড লক পপ-আপ স্টাইল */
        .lock-popup-container {
            position: fixed;
            top: 20px;
            right: 20px;
            width: 160px;
            z-index: 100000;
            cursor: grab;
            user-select: none;
            touch-action: none;
        }

        .lock-box {
            background: #000;
            border: 2px solid #ff0066;
            border-radius: 12px;
            padding: 15px;
            text-align: center;
            box-shadow: 0 0 15px rgba(255, 0, 102, 0.9);
            color: #fff;
        }

        .lock-icon {
            background: #0b1315;
            border: 1px solid #00ffcc;
            width: 35px;
            height: 35px;
            margin: 0 auto 10px auto;
            border-radius: 8px;
            display: flex;
            justify-content: center;
            align-items: center;
            font-size: 16px;
        }

        .lock-title {
            font-size: 11px;
            font-weight: bold;
            letter-spacing: 0.5px;
            color: #00ffcc;
        }

        .lock-subtitle {
            font-size: 8px;
            font-weight: bold;
            color: #ff0066;
            margin-bottom: 10px;
            letter-spacing: 0.5px;
        }

        .lock-input {
            width: 90%;
            padding: 6px;
            border: 1px solid #00ffcc;
            border-radius: 6px;
            background: #0b1315;
            color: #fff;
            font-size: 11px;
            text-align: center;
            margin-bottom: 12px;
            outline: none;
        }

        .btn-group {
            display: flex;
            justify-content: space-between;
            gap: 6px;
        }

        .btn {
            flex: 1;
            padding: 6px;
            border-radius: 6px;
            font-weight: bold;
            cursor: pointer;
            font-size: 10px;
        }

        .btn-execute {
            background: #ff0066;
            color: #fff;
            border: 1px solid #ff0066;
        }

        .btn-hide {
            background: #0b1315;
            color: #00ffcc;
            border: 1px solid #00ffcc;
        }

        /* FH বল (প্রথমে লুকিয়ে থাকবে, পাসওয়ার্ড বক্স হাইড করলে আসবে) */
        .floating-fh-ball {
            position: fixed;
            top: 20px;
            right: 20px;
            width: 50px;
            height: 50px;
            background: #000;
            border: 2px solid #ff0066;
            border-radius: 50%;
            display: none; /* প্রথমে লুকিয়ে থাকবে */
            flex-direction: column;
            justify-content: center;
            align-items: center;
            box-shadow: 0 0 15px rgba(255, 0, 102, 0.9);
            z-index: 100001;
            cursor: grab;
            user-select: none;
            touch-action: none;
            pointer-events: auto;
        }

        /* EI বল (সিগন্যাল বক্স টগল করার জন্য) */
        .floating-ei-ball {
            position: fixed;
            top: 20px;
            right: 20px;
            width: 50px;
            height: 50px;
            background: #000;
            border: 2px solid #00ffcc;
            border-radius: 50%;
            display: none; /* প্রথমে লুকিয়ে থাকবে */
            flex-direction: column;
            justify-content: center;
            align-items: center;
            box-shadow: 0 0 15px rgba(0, 255, 204, 0.9);
            z-index: 100001;
            cursor: grab;
            user-select: none;
            touch-action: none;
            pointer-events: auto;
        }

        .ball-text {
            font-size: 11px;
            font-weight: bold;
            color: #fff;
        }
        .ball-bar {
            width: 25px;
            height: 2px;
            background: #00ffcc;
            margin-top: 2px;
        }

        /* EI বলের নিচে নতুন 'H' গোল বাটন */
        .floating-history-ball {
            position: fixed;
            top: 80px;
            right: 30px;
            width: 30px;
            height: 30px;
            background: #000;
            border: 2px solid #ff0066;
            border-radius: 50%;
            display: none; /* প্রথমে লুকিয়ে থাকবে */
            justify-content: center;
            align-items: center;
            box-shadow: 0 0 12px rgba(255, 0, 102, 0.8);
            z-index: 100001;
            user-select: none;
            pointer-events: auto;
            color: #00ffcc;
            font-size: 15px;
            font-weight: bold;
            cursor: pointer;
        }

        /* হিস্ট্রি দেখানোর বড় পপ-আপ মডাল */
        .history-modal {
            position: fixed;
            top: 50%;
            left: 50%;
            transform: translate(-50%, -50%);
            width: 340px;
            height: 380px;
            background: #000;
            border: 2px solid #ff0066;
            border-radius: 12px;
            z-index: 100005;
            display: none;
            flex-direction: column;
            box-shadow: 0 0 20px rgba(255, 0, 102, 0.8);
            color: #fff;
            overflow: hidden;
            font-family: Arial, sans-serif;
        }
        .history-header {
            background: #00ffcc;
            color: #000;
            font-weight: bold;
            font-size: 12px;
            display: flex;
            justify-content: space-between;
            padding: 8px 12px;
            align-items: center;
            flex-shrink: 0;
        }
        .history-close {
            cursor: pointer;
            font-size: 14px;
            font-weight: bold;
        }
        .history-table-container {
            width: 100%;
            height: calc(380px - 33px);
            overflow-y: auto;
        }
        .history-table {
            width: 100%;
            border-collapse: collapse;
            table-layout: fixed;
        }
        .history-table th {
            background: #ffcc00;
            color: #000;
            font-size: 10px;
            padding: 6px 2px;
            text-align: center;
            position: sticky;
            top: 0;
            z-index: 2;
        }
        .history-table th:nth-child(1), .history-table td:nth-child(1) {
            width: 140px;
            text-align: center;
            padding-left: 4px;
        }
        .history-table td {
            font-size: 10px;
            height: 32px;
            padding: 4px 2px;
            text-align: center;
            border-bottom: 1px solid #222;
            white-space: nowrap;
            overflow: hidden;
            text-overflow: ellipsis;
        }

        /* মূল সিগন্যাল র‍্যাপার */
        .signal-box-wrapper {
            position: relative;
            width: 340px;
            height: 200px;
            pointer-events: auto;
            display: flex;
            justify-content: center;
        }

        /* কমন বক্স স্টাইল */
        .box {
            position: absolute;
            background: #0b1315;
            border: 2px solid #00ffcc;
            border-radius: 10px;
            color: #fff;
            text-align: center;
            box-shadow: 0 0 12px rgba(0, 255, 204, 0.4);
        }

        .box-title {
            font-size: 8px;
            letter-spacing: 1px;
            color: #8ceade;
            margin-bottom: 2px;
            font-weight: bold;
        }

        /* ১. টেমার বক্স */
        .timer-box {
            top: 0px;
            left: 70%;
            transform: translateX(-50%);
            width: 90px;
            padding: 6px;
        }
        .timer-value {
            font-size: 13px;
            font-weight: bold;
            color: #fff;
        }
        .timer-bar-container {
            width: 100%;
            height: 3px;
            background: #222;
            margin-top: 4px;
            border-radius: 2px;
            overflow: hidden;
        }
        .timer-bar {
            height: 100%;
            background: #00ffcc;
            width: 100%;
            transition: width 1s linear;
        }

        /* নতুন নিউ পিরিয়ড বক্স */
        .new-period-box {
            top: 0px;
            left: 25%;
            transform: translateX(-50%);
            width: 115px;
            padding: 6px 2px;
        }
        .new-period-value {
            font-size: 11px;
            font-weight: bold;
            color: #ffcc00;
            margin-top: 2px;
            letter-spacing: 0.5px;
        }

        /* ৫. টেমার বক্সের নিচের ছোট বক্স (NUMBER সহ) */
        .sub-timer-box {
            top: 55px;
            left: 50%;
            transform: translateX(-50%);
            width: 50px;
            padding: 7px 4px;
        }
        .sub-timer-value {
            font-size: 18px;
            font-weight: bold;
            margin-top: 2px;
        }

        /* ২. সিগন্যাল বক্স */
        .signal-left {
            top: 55px;
            left: 15px;
            width: 100px;
            padding: 8px 4px;
        }
        .signal-value {
            font-size: 14px;
            font-weight: bold;
        }

        /* ৩. কালার বক্স */
        .signal-right {
            top: 55px;
            right: 15px;
            width: 95px;
            padding: 8px 4px;
        }
        .color-value {
            font-size: 14px;
            font-weight: bold;
            margin-top: 3px;
        }

        /* ৪. মাঝখানের গোল HIDE লোগো বক্স */
        .center-circle {
            position: absolute;
            top: 110px;
            left: 50%;
            transform: translateX(-50%);
            width: 55px;
            height: 55px;
            background: #000;
            border: 3px solid #ff0066;
            border-radius: 50%;
            display: flex;
            flex-direction: column;
            justify-content: center;
            align-items: center;
            box-shadow: 0 0 15px rgba(255, 0, 102, 0.9);
            z-index: 20;
            cursor: pointer;
        }
        .circle-text {
            font-size: 10px;
            font-weight: bold;
            color: #fff;
        }
        .circle-bar {
            width: 25px;
            height: 2px;
            background: #00ffcc;
            margin-top: 2px;
        }
    </style>
</head>
<body>

    <!-- ব্যাকগ্রাউন্ড ওয়েবসাইট -->
    <iframe src="https://dkwin9.com/" class="website-frame"></iframe>

    <!-- পাসওয়ার্ড লক পপ-আপ -->
    <div class="lock-popup-container" id="lockPopup">
        <div class="lock-box">
            <div class="lock-icon">🔒</div>
            <div class="lock-title">ACCESS LOCKED</div>
            <div class="lock-subtitle">ENTER PASSWORD</div>
            <input type="password" id="accessKey" class="lock-input" placeholder="ACCESS KEY" value="12345">
            <div class="btn-group">
                <button class="btn btn-execute" onclick="checkPassword()">EXECUTE</button>
                <button class="btn btn-hide" onclick="hidePasswordLock()">HIDE</button>
            </div>
        </div>
    </div>

    <!-- FH বল (পাসওয়ার্ড পপ-আপ টগল করার জন্য) -->
    <div class="floating-fh-ball" id="fhBall" onclick="handleFhBallClick()">
        <div class="ball-text">FH</div>
        <div class="ball-bar"></div>
    </div>

    <!-- EI বল (সিগন্যাল বক্স টগল করার জন্য) -->
    <div class="floating-ei-ball" id="eiBall" onclick="handleEiBallClick()">
        <div class="ball-text">EI</div>
        <div class="ball-bar"></div>
    </div>

    <!-- EI বলের নিচে নতুন 'H' গোল বাটন -->
    <div class="floating-history-ball" id="historyBall" onclick="toggleHistoryModal()">H</div>

    <!-- হিস্ট্রি দেখানোর বড় পপ-আপ মডাল -->
    <div class="history-modal" id="historyModal">
        <div class="history-header">
            <span>Signal History</span>
            <span class="history-close" onclick="toggleHistoryModal()">✕</span>
        </div>
        <div class="history-table-container">
            <table class="history-table">
                <thead>
                    <tr>
                        <th>Period</th>
                        <th>Number</th>
                        <th>Status</th>
                    </tr>
                </thead>
                <tbody id="historyTableBody">
                    <!-- নিজস্ব জেনারেটেড হিস্ট্রি এখানে ডাইনামিক দেখাবে -->
                </tbody>
            </table>
        </div>
    </div>

    <!-- সিগন্যাল ওভারলে বক্সসমূহ -->
    <div class="overlay-container" id="signalOverlay">
        <div class="signal-box-wrapper" id="signalWrapper">
            
            <!-- New Period Box -->
            <div class="box new-period-box">
                <div class="box-title">NEW PERIOD</div>
                <div class="new-period-value" id="newPeriodDisplay">---</div>
            </div>

            <!-- Timer Box -->
            <div class="box timer-box">
                <div class="box-title">TIMER</div>
                <div class="timer-value" id="timerDisplay">30S</div>
                <div class="timer-bar-container">
                    <div class="timer-bar" id="timerBar"></div>
                </div>
            </div>

            <!-- Sub Timer Box (NUMBER সহ) -->
            <div class="box sub-timer-box">
                <div class="box-title" id="numBoxTitle">NUMBER</div>
                <div class="sub-timer-value" id="subTimerValue">0</div>
            </div>

            <!-- Signal Left Box -->
            <div class="box signal-left">
                <div class="box-title">SIGNAL</div>
                <div class="signal-value" id="signalValue">SMALL</div>
            </div>

            <!-- Colour Right Box -->
            <div class="box signal-right">
                <div class="box-title">COLOUR</div>
                <div class="color-value" id="colorValue">Green Violet</div>
            </div>

            <!-- Center HIDE Circle Box -->
            <div class="center-circle" onclick="toggleSignalBoxes()">
                <div class="circle-text">HIDE</div>
                <div class="circle-bar"></div>
            </div>

        </div>
    </div>

    <script>
        let isUnlocked = false;
        let selectedTime = 30;
        let selectedApiType = 'WinGo_30S';
        let timerInterval = null;
        let lastGeneratedCycle = -1;
        let apiCheckedThisCycle = false;

        let currentSignal = {
            number: 0,
            size: 'Small',
            targetFullPeriod: ''
        };

        // ব্রেন মেমোরি ও অ্যাপের নিজস্ব সিগন্যাল হিস্ট্রি অ্যারে
        let apiBrainMemory = [];
        let mySignalHistory = []; // জেনারেট করা সিগন্যাল ও রেজাল্ট সেভ রাখার জন্য

        function checkPassword() {
            let key = document.getElementById('accessKey').value;
            if (key === "12345") {
                isUnlocked = true;
                document.getElementById('lockPopup').style.display = 'none';
                document.getElementById('signalOverlay').style.display = 'flex';
                document.getElementById('signalWrapper').style.display = 'flex';
                document.getElementById('fhBall').style.display = 'none';
                document.getElementById('eiBall').style.display = 'none';
                document.getElementById('historyBall').style.display = 'none';
                startTimer(selectedTime);
                fetchAndUpdateNewPeriod(); 
            } else {
                alert("ভুল পাসওয়ার্ড! সঠিক পাসওয়ার্ড দিন।");
            }
        }

        function hidePasswordLock() {
            document.getElementById('lockPopup').style.display = 'none';
            document.getElementById('fhBall').style.display = 'flex';
        }

        function handleFhBallClick() {
            if (!hasMoved) {
                const lockPopup = document.getElementById('lockPopup');
                lockPopup.style.display = 'block';
                document.getElementById('fhBall').style.display = 'none';
            }
        }

        function handleEiBallClick() {
            if (!hasMoved) {
                toggleSignalBoxes();
            }
        }

        function toggleSignalBoxes() {
            const wrapper = document.getElementById('signalWrapper');
            const eiBall = document.getElementById('eiBall');
            const historyBall = document.getElementById('historyBall');
            
            if (wrapper.style.display === 'none') {
                wrapper.style.display = 'flex';
                eiBall.style.display = 'none';
                historyBall.style.display = 'none';
            } else {
                wrapper.style.display = 'none';
                eiBall.style.display = 'flex';
                historyBall.style.display = 'flex';
            }
        }

        function toggleHistoryModal() {
            const modal = document.getElementById('historyModal');
            if (modal.style.display === 'flex') {
                modal.style.display = 'none';
            } else {
                modal.style.display = 'flex';
                displayMyHistoryTable();
            }
        }

        // নিজস্ব জেনারেটেড হিস্ট্রি টেবিলে দেখানোর ফাংশন
        function displayMyHistoryTable() {
            const tbody = document.getElementById('historyTableBody');
            if (mySignalHistory.length === 0) {
                tbody.innerHTML = '<tr><td colspan="3">No History Yet</td></tr>';
                return;
            }

            let html = '';
            mySignalHistory.forEach(item => {
                let statusColor = item.status === 'Win' ? '#28a745' : '#dc3545';
                html += `<tr>
                    <td style="color:#fff; font-size:10px; text-align:center;">${item.period}</td>
                    <td style="font-weight:bold; font-size:13px; color:#fff;">${item.number}</td>
                    <td style="font-weight:bold; font-size:12px; color:${statusColor};">${item.status}</td>
                </tr>`;
            });
            tbody.innerHTML = html;
        }

        // API থেকে লেটেস্ট পিরিয়ড এনে নিউ পিরিয়ড বক্সে টার্গেট সেট করা
        async function fetchAndUpdateNewPeriod() {
            const url = `https://draw.ar-lottery01.com/WinGo/${selectedApiType}/GetHistoryIssuePage.json`;
            try {
                let response = await fetch(url, {
                    headers: {
                        "Origin": "https://dkwin9.com",
                        "Referer": "https://dkwin9.com/"
                    }
                });
                let data = await response.json();
                apiBrainMemory = data.data.list; // সব পুরনো ও নতুন রেজাল্ট সেভ রাখা হলো
                let latestItem = apiBrainMemory[0];
                let latestPeriod = latestItem.issueNumber || latestItem.gameNo || latestItem.period || '';
                
                if (latestPeriod) {
                    let nextPeriod = String(BigInt(latestPeriod) + 1n);
                    currentSignal.targetFullPeriod = nextPeriod; 
                    document.getElementById('newPeriodDisplay').textContent = nextPeriod; 
                } else {
                    document.getElementById('newPeriodDisplay').textContent = "---";
                }
            } catch (error) {
                console.log("Period Fetch Error: ", error);
            }
        }

        // টাইমার শেষে সঠিকভাবে API থেকে নতুন রেজাল্ট এনে মিল করার ফাংশন (ডিলে সহ)
        async function fetchAndCheckResult() {
            if (apiCheckedThisCycle) return;
            apiCheckedThisCycle = true;

            const url = `https://draw.ar-lottery01.com/WinGo/${selectedApiType}/GetHistoryIssuePage.json`;
            
            setTimeout(async () => {
                try {
                    let response = await fetch(url, {
                        headers: {
                            "Origin": "https://dkwin9.com",
                            "Referer": "https://dkwin9.com/"
                        }
                    });
                    let data = await response.json();
                    apiBrainMemory = data.data.list; // নতুন রেজাল্টসহ আপডেট করা হলো
                    let latestResult = apiBrainMemory[0];

                    let apiFullPeriod = latestResult.issueNumber || latestResult.gameNo || latestResult.period || '';
                    let apiNumber = parseInt(latestResult.number); 

                    let isPeriodMatch = (apiFullPeriod === currentSignal.targetFullPeriod);
                    let isMatch = isPeriodMatch && (apiNumber === currentSignal.number);

                    let statusText = isMatch ? "Win" : "Loss";
                    applyWinLossUI(isMatch);

                    mySignalHistory.unshift({
                        period: currentSignal.targetFullPeriod,
                        number: currentSignal.number,
                        status: statusText
                    });

                    if (document.getElementById('historyModal').style.display === 'flex') {
                        displayMyHistoryTable();
                    }

                    fetchAndUpdateNewPeriod(); 

                } catch (error) {
                    console.log("API Fetch Error: ", error);
                    applyWinLossUI(false);

                    mySignalHistory.unshift({
                        period: currentSignal.targetFullPeriod,
                        number: currentSignal.number,
                        status: "Loss"
                    });

                    if (document.getElementById('historyModal').style.display === 'flex') {
                        displayMyHistoryTable();
                    }
                }
            }, 1500);
        }

        function applyWinLossUI(isWin) {
            const subTimerValue = document.getElementById('subTimerValue');
            const signalValue = document.getElementById('signalValue');
            const colorValue = document.getElementById('colorValue');

            document.getElementById('numBoxTitle').textContent = "STATUS";
            subTimerValue.style.background = "none";
            subTimerValue.style.webkitTextFillColor = "initial";
            
            if (isWin) {
                subTimerValue.textContent = "Win";
                subTimerValue.style.color = "#28a745";
                signalValue.textContent = "Win";
                signalValue.style.color = "#28a745";
                colorValue.innerHTML = '<span style="color:#28a745;">Win</span>';
            } else {
                subTimerValue.textContent = "Loss";
                subTimerValue.style.color = "#dc3545";
                signalValue.textContent = "Loss";
                signalValue.style.color = "#dc3545";
                colorValue.innerHTML = '<span style="color:#dc3545;">Loss</span>';
            }
        }

        // Groq AI API ব্যবহার করে শক্তিশালী ব্রেন প্রেডিকশন লজিক
        async function generateNewSignal() {
            const subTimerValue = document.getElementById('subTimerValue');
            document.getElementById('numBoxTitle').textContent = "NUMBER";
            subTimerValue.textContent = "...";

            let predictedNum = 5; // ডিফল্ট ফলব্যাক

            try {
                // apiBrainMemory থেকে পূর্বের রেজাল্টগুলো প্রস্তুত করা
                let historyDataString = JSON.stringify(apiBrainMemory.slice(0, 15));

                const groqApiKey = "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";
                const response = await fetch("https://api.groq.com/openai/v1/chat/completions", {
                    method: "POST",
                    headers: {
                        "Authorization": `Bearer ${groqApiKey}`,
                        "Content-Type": "application/json"
                    },
                    body: JSON.stringify({
                        model: "openai/gpt-oss-20b",
                        messages: [
                            {
                                role: "system",
                                content: "You are an expert lottery and color prediction analyzer. Analyze the provided past game results array history, look for patterns, trends, frequency, and statistical probabilities. Output ONLY a single digit integer from 0 to 9 representing the predicted next number."
                            },
                            {
                                role: "user",
                                content: `Here is the past game results history: ${historyDataString}. Predict the exact next single digit number (0-9). Return ONLY the digit number, nothing else.`
                            }
                        ],
                        temperature: 0.7,
                        max_tokens: 5
                    })
                });

                let aiData = await response.json();
                let aiText = aiData.choices[0].message.content.trim();
                let parsedNum = parseInt(aiText.replace(/[^0-9]/g, ''));
                
                if (!isNaN(parsedNum) && parsedNum >= 0 && parsedNum <= 9) {
                    predictedNum = parsedNum;
                } else {
                    predictedNum = Math.floor(Math.random() * 10);
                }
            } catch (err) {
                console.error("Groq AI Error:", err);
                predictedNum = Math.floor(Math.random() * 10);
            }

            currentSignal.number = predictedNum;
            updateSignalUI(predictedNum);
        }

        function updateSignalUI(predictedNum) {
            const subTimerValue = document.getElementById('subTimerValue');
            const signalValue = document.getElementById('signalValue');
            const colorValue = document.getElementById('colorValue');

            subTimerValue.textContent = predictedNum;

            if (predictedNum === 0) {
                subTimerValue.style.background = "linear-gradient(to bottom, #ff3333 50%, #9933cc 50%)";
                subTimerValue.style.webkitBackgroundClip = "text";
                subTimerValue.style.webkitTextFillColor = "transparent";
            } else if (predictedNum === 5) {
                subTimerValue.style.background = "linear-gradient(to bottom, #28a745 50%, #9933cc 50%)";
                subTimerValue.style.webkitBackgroundClip = "text";
                subTimerValue.style.webkitTextFillColor = "transparent";
            } else if ([1, 3, 7, 9].includes(predictedNum)) {
                subTimerValue.style.background = "none";
                subTimerValue.style.webkitTextFillColor = "initial";
                subTimerValue.style.color = "#28a745";
            } else {
                subTimerValue.style.background = "none";
                subTimerValue.style.webkitTextFillColor = "initial";
                subTimerValue.style.color = "#dc3545";
            }

            if (predictedNum >= 0 && predictedNum <= 4) {
                currentSignal.size = "Small";
                signalValue.textContent = "Small";
                signalValue.style.color = "#5bc0de"; 
            } else {
                currentSignal.size = "Big";
                signalValue.textContent = "Big";
                signalValue.style.color = "#f0ad4e"; 
            }

            if (predictedNum === 0) {
                colorValue.innerHTML = '<span style="color:#ff3333;">Red</span> <span style="color:#9933cc;">Violet</span>';
            } else if (predictedNum === 5) {
                colorValue.innerHTML = '<span style="color:#28a745;">Green</span> <span style="color:#9933cc;">Violet</span>';
            } else if ([1, 3, 7, 9].includes(predictedNum)) {
                colorValue.innerHTML = '<span style="color:#28a745;">Green</span>';
            } else {
                colorValue.innerHTML = '<span style="color:#dc3545;">Red</span>';
            }
        }
        
        // টাইমার কোড শুরু
        function startTimer(totalSeconds) {
            if (timerInterval) clearInterval(timerInterval);
            
            const timerDisplay = document.getElementById('timerDisplay');
            const timerBar = document.getElementById('timerBar');

            async function updateDisplay() {
                let now = Math.floor(Date.now() / 1000 + 2); 
                let currentCycle = Math.floor(now / totalSeconds);
                let rawTimeLeft = totalSeconds - (now % totalSeconds);
                
                let timeLeft = rawTimeLeft === totalSeconds ? 0 : rawTimeLeft;

                if (currentCycle !== lastGeneratedCycle) {
                    lastGeneratedCycle = currentCycle;
                    apiCheckedThisCycle = false;
                    await fetchAndUpdateNewPeriod(); 
                    await generateNewSignal(); // AI দিয়ে নতুন সিগন্যাল জেনারেট করা
                }

                if (timeLeft === 3 || timeLeft === 2) {
                    fetchAndCheckResult();
                }

                let mins = Math.floor(timeLeft / 60);
                let secs = timeLeft % 60;
                let formattedTime = (mins < 10 ? "0" + mins : mins) + ":" + (secs < 10 ? "0" + secs : secs) + "M";

                if (totalSeconds === 30) {
                    timerDisplay.textContent = (timeLeft < 10 ? "0" : "") + timeLeft + "S";
                } else {
                    timerDisplay.textContent = formattedTime;
                }
                let percentage = (timeLeft / totalSeconds) * 100;
                timerBar.style.width = percentage + "%";
            }

            updateDisplay();

            timerInterval = setInterval(updateDisplay, 1000);
        }

        makeDraggable(document.getElementById('lockPopup'));
        makeDraggable(document.getElementById('fhBall'));
        makeDraggable(document.getElementById('eiBall'));

        let hasMoved = false;

        function makeDraggable(element) {
            let isDragging = false, startX, startY;

            element.addEventListener('mousedown', dragStart);
            document.addEventListener('mousemove', drag);
            document.addEventListener('mouseup', dragEnd);

            element.addEventListener('touchstart', dragStart);
            document.addEventListener('touchmove', drag);
            document.addEventListener('touchend', dragEnd);

            function dragStart(e) {
                isDragging = true;
                hasMoved = false;
                let clientX = e.type.includes('touch') ? e.touches[0].clientX : e.clientX;
                let clientY = e.type.includes('touch') ? e.touches[0].clientY : e.clientY;
                
                startX = clientX - element.offsetLeft;
                startY = clientY - element.offsetTop;
            }

            function drag(e) {
                if (!isDragging) return;
                hasMoved = true;
                let clientX = e.type.includes('touch') ? e.touches[0].clientX : e.clientX;
                let clientY = e.type.includes('touch') ? e.touches[0].clientY : e.clientY;

                let newX = clientX - startX;
                let newY = clientY - startY;

                let maxX = window.innerWidth - element.offsetWidth;
                let maxY = window.innerHeight - element.offsetHeight;

                newX = Math.max(0, Math.min(newX, maxX));
                newY = Math.max(0, Math.min(newY, maxY));

                element.style.left = newX + 'px';
                element.style.top = newY + 'px';
                element.style.right = 'auto';
            }

            function dragEnd() {
                isDragging = false;
            }
        }
    </script>

</body>
</html>
