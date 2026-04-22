<!DOCTYPE html>
<html lang="zh-TW">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no">
    <title>Readmoo 折扣比價</title>
    <style>
        :root {
            --page-bg: #e8e6df;
            --app-bg: #f8f7f2;
            --display-bg: #f2f1eb;
            --key-bg: #d6c9c7; /* 指定的淺色按鈕顏色 */ 
            --key-text: #ffffff;
            --gold: #b3a58e;
            --milktea: #917c79; /* 計算按鈕顏色 */
            --text-main: #6b6a66;
            --card-bg: #f2f1eb;
            --shadow: rgba(0,0,0,0.1);
        }

        [data-theme="dark"] {
            --page-bg: #1a1b1d;
            --app-bg: #212224;
            --display-bg: #2a2c2f;
            --key-bg: #3a3d42; 
            --key-text: #fefefe;
            --gold: #d4b889;
            --milktea: #917c79;
            --text-main: #fefefe;
            --card-bg: #2a2c2f;
            --shadow: rgba(0,0,0,0.4);
        }

        body {
            font-family: -apple-system, "Noto Sans CJK TC", sans-serif;
            background-color: var(--page-bg);
            margin: 0; padding: 40px 20px;
            display: flex; justify-content: center; align-items: flex-start;
            min-height: 100vh;
            transition: 0.3s;
        }

        .theme-switch-wrap { position: fixed; top: 20px; right: 20px; z-index: 100; }
        .theme-btn {
            background: var(--app-bg); border: 1px solid var(--gold);
            color: var(--gold); border-radius: 20px;
            padding: 6px 15px; cursor: pointer; font-size: 0.8rem;
            box-shadow: 0 2px 10px var(--shadow);
        }

        .app-container {
            width: 100%; max-width: 450px;
            background-color: var(--app-bg);
            border-radius: 40px; 
            padding: 35px 25px;
            box-shadow: 0 20px 50px var(--shadow);
            border: 8px solid var(--display-bg);
        }

	/* --- 1. 金額顯示區 (Display) --- */
        .display-area {
            background-color: var(--display-bg);
            border-radius: 20px; padding: 25px;
            text-align: right; margin-bottom: 20px;
        }
        .display-area .label { color: var(--gold); font-size: 0.9rem; text-align: left; margin-bottom: 10px; font-weight: bold;}
        .display-area .price-symbol { color: var(--gold); font-size: 1.8rem; margin-right: 5px; }
        .display-area .price-val { color: var(--text-main); font-size: 4.5rem; font-weight: 300; }

	/* --- 2. 數字鍵盤 (Keypad) --- */
        .keypad {
            display: grid; grid-template-columns: repeat(4, 1fr);
            gap: 12px; margin-bottom: 25px;
        }
        .key {
            background-color: var(--key-bg); color: var(--key-text);
            border: none; border-radius: 12px; height: 60px; 
            font-size: 1.5rem; cursor: pointer; font-weight: 500;
            transition: transform 0.1s;
        }
        .key:active { transform: scale(0.95); opacity: 0.8; }

	/* 特殊按鈕 */
        .key-c { color: var(--milktea); background-color: var(--display-bg); font-weight: bold; border: 1px solid var(--key-bg); }
        .key-calc { 
            grid-row: span 2; height: 132px;
            background-color: var(--milktea); color: white;
            font-size: 1.2rem; font-weight: bold;  font-weight: 500;
        }

        .results-grid {
            display: grid; grid-template-columns: repeat(3, 1fr); gap: 10px;
            display: none;
        }

	/* --- 3. 折扣卡片區 (Coupon Grid) --- */
        .coupon-card {
            background-color: var(--card-bg); border-radius: 15px;
            padding: 18px 5px; text-align: center; border: 1.5px solid transparent;
            display: flex; flex-direction: column; justify-content: center;
            transition: 0.3s;
        }
        
        .coupon-card.best { border-color: var(--gold); position: relative; }
        .coupon-card.disabled { opacity: 0.4; filter: grayscale(1); }
        
        .best-tag {
            font-size: 0.6rem; color: var(--gold);
            border: 1px solid var(--gold); border-radius: 10px;
            padding: 2px 8px; align-self: center; margin-bottom: 8px;
        }

        .coupon-name { font-size: 0.85rem; color: var(--gold); margin-bottom: 8px; font-weight: bold; }
        .coupon-price { font-size: 1.8rem; color: var(--text-main); font-weight: bold; margin-bottom: 5px; }
        .coupon-save { font-size: 0.7rem; color: var(--gold); margin-bottom: 4px; height: 1rem; }
        .coupon-desc { font-size: 0.6rem; color: var(--text-main); opacity: 0.7; line-height: 1.2; }

        .tooltip-area {
            margin-top: 25px; padding-top: 20px;
            border-top: 1px solid var(--gold);
            display: none;
        }
        .tooltip-text {
            color: var(--text-main); font-size: 0.85rem; line-height: 1.6;
            padding-left: 15px; border-left: 3px solid var(--milktea);
        }

        .init-view { text-align: center; color: var(--gold); margin-top: 50px; }
        .init-icon { font-size: 2.5rem; opacity: 0.4; margin-bottom: 15px; display: block; }
    </style>
</head>
<body>

<div class="theme-switch-wrap">
    <button class="theme-btn" onclick="toggleTheme()">切換模式</button>
</div>

<div class="app-container">
    <div class="display-area">
        <div class="label">書本售價</div>
        <div>
            <span class="price-symbol">$</span>
            <span id="displayVal" class="price-val">0</span>
        </div>
    </div>

    <div class="keypad">
        <button class="key" onclick="press('7')">7</button>
        <button class="key" onclick="press('8')">8</button>
        <button class="key" onclick="press('9')">9</button>
        <button class="key" style="color: var(--gold); font-size: 1.2rem;" onclick="back()">⌫</button>

        <button class="key" onclick="press('4')">4</button>
        <button class="key" onclick="press('5')">5</button>
        <button class="key" onclick="press('6')">6</button>
        <button class="key key-c" onclick="cls()">C</button>

        <button class="key" onclick="press('1')">1</button>
        <button class="key" onclick="press('2')">2</button>
        <button class="key" onclick="press('3')">3</button>
        <button class="key key-calc" onclick="calculate()">計算</button>

        <button class="key" onclick="press('0')">0</button>
        <button class="key" onclick="press('00')">00</button>
    </div>

    <div id="initInfo" class="init-view">
        <span class="init-icon">📖</span>
        <div style="font-size: 0.9rem; letter-spacing: 1px;">輸入書價，按「計算」看結果</div>
    </div>

    <div id="results" class="results-grid"></div>

    <div id="tooltip" class="tooltip-area">
        <div id="bestNote" class="tooltip-text"></div>
    </div>
</div>

<script>
    let currentInput = "0";
    const COST_PER_POINT = 166.5; // 每點領書額度成本

    function toggleTheme() {
        const body = document.body;
        const current = body.getAttribute('data-theme');
        body.setAttribute('data-theme', current === 'dark' ? 'light' : 'dark');
    }

    function press(n) {
        if (currentInput === "0") currentInput = n;
        else if (currentInput.length < 6) currentInput += n;
        update();
    }

    function cls() { 
        currentInput = "0"; update(); 
        document.getElementById('results').style.display = 'none'; 
        document.getElementById('tooltip').style.display = 'none'; 
        document.getElementById('initInfo').style.display = 'block'; 
    }

    function back() { 
        currentInput = currentInput.slice(0, -1); 
        if (currentInput === "") currentInput = "0"; 
        update(); 
    }

    function update() { document.getElementById('displayVal').innerText = currentInput; }

    function calculate() {
        const p = parseInt(currentInput);
        if (p === 0) return;

        document.getElementById('initInfo').style.display = 'none';
        document.getElementById('results').style.display = 'grid';
        document.getElementById('tooltip').style.display = 'block';

        // 領書額度：(含)250元以下 1 點，超過每 250 元進位 1 點
        const pointsNeeded = Math.ceil(p / 250);
        const creditCost = pointsNeeded * COST_PER_POINT;

        const schemes = [
            { id: "unlimited50", name: "不限額折50", threshold: 0, calc: (p) => Math.max(0, p - 50), desc: "不限金額折$50" },
            { name: "滿250折50", threshold: 250, calc: (p) => p - 50, desc: "滿$250可用" },
            { name: "滿150折30", threshold: 150, calc: (p) => p - 30, desc: "滿$150可用" },
            { name: "滿100折25", threshold: 100, calc: (p) => p - 25, desc: "滿$100可用" },
            { name: "7折券", threshold: 0, calc: (p) => Math.round(p * 0.7), desc: "7折優惠券" },
            { name: "75折券", threshold: 0, calc: (p) => Math.round(p * 0.75), desc: "3本以上75折" },
            { name: "8折券", threshold: 0, calc: (p) => Math.round(p * 0.8), desc: "單本8折" },
            { id: "credit", name: "領書額度", threshold: 0, calc: (p) => creditCost, desc: `消耗 ${pointsNeeded} 點` },
            { name: "原價", threshold: 0, calc: (p) => p, desc: "無折扣" }
        ];

        let processed = schemes.map(s => {
            const isActive = p >= s.threshold;
            const finalPrice = isActive ? s.calc(p) : p;
            return {
                ...s,
                isActive,
                finalPrice,
                save: p - finalPrice
            };
        });

        processed.sort((a, b) => {
            if (a.isActive !== b.isActive) return b.isActive - a.isActive;
            
            // 邏輯優化：書價 200(含) 以下時，優先推薦「不限額折50」
            if (p <= 200) {
                if (a.id === "unlimited50") return -1;
                if (b.id === "unlimited50") return 1;
            }
            
            return a.finalPrice - b.finalPrice;
        });

        const resDiv = document.getElementById('results');
        resDiv.innerHTML = processed.map((s, i) => {
            const isBest = s.isActive && i === 0;
            // 格式化價格：領書額度若有小數點則顯示，折價券則為整數
            const priceLabel = s.id === "credit" ? s.finalPrice.toString() : s.finalPrice;
            
            return `
                <div class="coupon-card ${isBest ? 'best' : ''} ${!s.isActive ? 'disabled' : ''}">
                    ${isBest ? '<span class="best-tag">最划算</span>' : ''}
                    <div class="coupon-name">${s.name}</div>
                    <div class="coupon-price">$${s.isActive ? priceLabel : '-'}</div>
                    <div class="coupon-save">${s.isActive && s.save > 0 ? '省 $' + Math.floor(s.save) : ''}</div>
                    <div class="coupon-desc">${s.isActive ? s.desc : '未達門檻'}</div>
                </div>
            `;
        }).join('');

        const best = processed.find(s => s.isActive);
        let note = best ? `${best.name}最划算，換算成本約 $${best.finalPrice}。` : "";
        if (p <= 50) note += " 金額在 $50 以下，用券直接 $0 領書！";
        if (pointsNeeded > 1) note += ` 本書售價較高，需消耗 ${pointsNeeded} 點領書額度。`;
        
        document.getElementById('bestNote').innerText = note;
    }

</script>

</body>
</html>
