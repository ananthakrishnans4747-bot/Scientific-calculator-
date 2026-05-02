let currentExp = "";
let cursorPosition = 0;
let history = [];
let historyIndex = -1;
let isShiftActive = false;
let isAlphaActive = false;

const expEl = document.getElementById("expression");
const resEl = document.getElementById("result");
const indShift = document.getElementById("ind-shift");
const indAlpha = document.getElementById("ind-alpha");

function updateDisplay() {
    // Insert a blinking cursor span at cursorPosition
    let displayStr = currentExp.substring(0, cursorPosition) + '<span class="cursor">|</span>' + currentExp.substring(cursorPosition);
    expEl.innerHTML = displayStr + (isShiftActive ? " [S]" : "") + (isAlphaActive ? " [A]" : "");
}

function appendExp(val) {
    if (resEl.innerText !== "0" && resEl.innerText !== "Error" && currentExp === "") {
        // If there's a previous result and we press an operator, start with Ans
        if (['+', '-', '*', '/', '^'].includes(val)) {
            currentExp = "Ans" + val;
            cursorPosition = currentExp.length;
        } else {
            currentExp = val;
            cursorPosition = val.length;
            resEl.innerText = "0";
        }
    } else {
        currentExp = currentExp.slice(0, cursorPosition) + val + currentExp.slice(cursorPosition);
        cursorPosition += val.length;
    }
    
    // Reset modifiers
    isShiftActive = false;
    isAlphaActive = false;
    updateIndicators();
    updateDisplay();
}

function appendFunc(funcName) {
    let toAppend = "";
    if (isShiftActive) {
        toAppend = "a" + funcName + "("; // asin, acos, atan
    } else if (isAlphaActive) {
        toAppend = funcName + "(";
    } else {
        toAppend = funcName + "(";
    }
    
    currentExp = currentExp.slice(0, cursorPosition) + toAppend + currentExp.slice(cursorPosition);
    cursorPosition += toAppend.length;
    
    isShiftActive = false;
    isAlphaActive = false;
    updateIndicators();
    updateDisplay();
}

function deleteLast() {
    if (cursorPosition > 0) {
        currentExp = currentExp.slice(0, cursorPosition - 1) + currentExp.slice(cursorPosition);
        cursorPosition--;
        updateDisplay();
    }
}

function clearAll() {
    currentExp = "";
    cursorPosition = 0;
    resEl.innerText = "0";
    isShiftActive = false;
    isAlphaActive = false;
    updateIndicators();
    updateDisplay();
}

function toggleShift() {
    isShiftActive = !isShiftActive;
    if (isShiftActive) isAlphaActive = false;
    updateIndicators();
}

function toggleAlpha() {
    isAlphaActive = !isAlphaActive;
    if (isAlphaActive) isShiftActive = false;
    updateIndicators();
}

function updateIndicators() {
    indShift.className = isShiftActive ? "" : "hidden";
    indAlpha.className = isAlphaActive ? "" : "hidden";
}

function moveCursor(dir) {
    if (dir === -1 && cursorPosition > 0) cursorPosition--;
    if (dir === 1 && cursorPosition < currentExp.length) cursorPosition++;
    updateDisplay();
}

function moveHistory(dir) {
    if (history.length === 0) return;
    
    if (dir === -1) { // up (older history)
        if (historyIndex < history.length - 1) historyIndex++;
    } else { // down (newer history)
        if (historyIndex > -1) historyIndex--;
    }
    
    if (historyIndex === -1) {
        currentExp = "";
        cursorPosition = 0;
        resEl.innerText = "0";
    } else {
        currentExp = history[historyIndex].exp;
        cursorPosition = currentExp.length;
        resEl.innerText = history[historyIndex].res;
    }
    updateDisplay();
}

function calculate() {
    if (!currentExp) return;

    try {
        // Preprocess expression for mathjs
        let toEval = currentExp
            .replace(/×/g, '*')
            .replace(/÷/g, '/')
            .replace(/Ans/g, resEl.innerText === "Error" ? "0" : resEl.innerText)
            .replace(/π/g, 'pi')
            .replace(/ln\(/g, 'f_ln(')       // Temp placeholder for ln
            .replace(/log\(/g, 'log10(')    // Casio log is base 10
            .replace(/f_ln\(/g, 'log(')     // mathjs log is base e (ln)
            .replace(/√/g, 'sqrt')
            .replace(/nPr\(/g, 'permutations(')
            .replace(/nCr\(/g, 'combinations(');

        // Evaluate using math.js
        const result = math.evaluate(toEval);
        
        // Format result nicely
        let finalRes = math.format(result, { precision: 10 });
        
        resEl.innerText = finalRes;
        
        // Save to history
        history.unshift({ exp: currentExp, res: finalRes });
        if (history.length > 10) history.pop();
        historyIndex = -1; // Reset history navigation
        
        currentExp = ""; // Reset expression for next calculation
        cursorPosition = 0;
        updateDisplay();
    } catch (error) {
        resEl.innerText = "Syntax ERROR";
    }
}

// Keyboard support
document.addEventListener('keydown', (e) => {
    const key = e.key;
    if (/[0-9]/.test(key) || ['+', '-', '*', '/', '.', '(', ')', '^'].includes(key)) {
        appendExp(key);
    } else if (key === 'Enter' || key === '=') {
        calculate();
    } else if (key === 'Backspace') {
        deleteLast();
    } else if (key === 'Escape') {
        clearAll();
    } else if (key === 'ArrowLeft') {
        moveCursor(-1);
    } else if (key === 'ArrowRight') {
        moveCursor(1);
    } else if (key === 'ArrowUp') {
        moveHistory(-1);
    } else if (key === 'ArrowDown') {
        moveHistory(1);
    }
});

// Initial display
updateDisplay();
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Scientific Calculator - Casio Style</title>
    <link rel="preconnect" href="https://fonts.googleapis.com">
    <link rel="preconnect" href="https://fonts.gstatic.com" crossorigin>
    <link href="https://fonts.googleapis.com/css2?family=Share+Tech+Mono&family=Inter:wght@400;600;700&display=swap" rel="stylesheet">
    <link rel="stylesheet" href="style.css">
</head>
<body>
    <div class="calculator-chassis">
        <div class="calc-header">
            <div class="brand-section">
                <span class="brand-name">CASIO</span>
                <span class="model-name">fx-991MS <small>S-V.P.A.M.</small></span>
            </div>
            <div class="solar-panel">
                <div class="cell"></div><div class="cell"></div><div class="cell"></div><div class="cell"></div>
            </div>
        </div>

        <div class="screen-wrapper">
            <div class="screen">
                <div class="indicators">
                    <span id="ind-shift" class="hidden">S</span>
                    <span id="ind-alpha" class="hidden">A</span>
                    <span id="ind-math">MATH</span>
                </div>
                <div class="display-lines">
                    <div class="expression-line" id="expression"></div>
                    <div class="result-line" id="result">0</div>
                </div>
            </div>
        </div>

        <div class="keypad">
            <!-- Top Controls Row -->
            <div class="key-row top-controls">
                <div class="key-group">
                    <div class="key-label shift-label">SHIFT</div>
                    <button class="key key-small key-grey" id="btn-shift" onclick="toggleShift()">SHIFT</button>
                </div>
                <div class="key-group">
                    <div class="key-label alpha-label">ALPHA</div>
                    <button class="key key-small key-grey" id="btn-alpha" onclick="toggleAlpha()">ALPHA</button>
                </div>
                
                <div class="nav-cluster">
                    <button class="key key-nav" onclick="moveHistory(-1)">↑ Hist</button>
                    <div class="nav-row">
                        <button class="key key-nav" onclick="moveCursor(-1)">← Cur</button>
                        <button class="key key-nav" onclick="moveCursor(1)">Cur →</button>
                    </div>
                    <button class="key key-nav" onclick="moveHistory(1)">↓ Hist</button>
                </div>

                <div class="key-group">
                    <div class="key-label">MODE</div>
                    <button class="key key-small key-grey" onclick="clearAll()">MODE</button>
                </div>
                <div class="key-group">
                    <div class="key-label">ON</div>
                    <button class="key key-small key-grey" onclick="clearAll()">ON</button>
                </div>
            </div>

            <!-- Scientific Keys Area -->
            <div class="scientific-keys">
                <!-- Row 1 -->
                <div class="key-row">
                    <div class="key-group"><div class="key-label shift-label">x!</div><button class="key key-sci" onclick="appendExp('^-1')">x⁻¹</button></div>
                    <div class="key-group"><div class="key-label shift-label">nCr</div><button class="key key-sci" onclick="appendExp('nPr(')">nCr</button></div>
                    <div class="key-group"><div class="key-label shift-label">Pol(</div><button class="key key-sci" onclick="appendExp('abs(')">Abs</button></div>
                    <div class="key-group"><div class="key-label shift-label">x³</div><button class="key key-sci" onclick="appendExp('^3')">x³</button></div>
                    <div class="key-group"><div class="key-label shift-label">d/dx</div><button class="key key-sci" onclick="appendExp('derivative(')">∫dx</button></div>
                    <div class="key-group"><div class="key-label shift-label">10^x</div><button class="key key-sci" onclick="appendExp('log(')">log</button></div>
                </div>
                <!-- Row 2 -->
                <div class="key-row">
                    <div class="key-group"><div class="key-label shift-label">d/c</div><button class="key key-sci" onclick="appendExp('/')">a b/c</button></div>
                    <div class="key-group"><div class="key-label shift-label">³√</div><button class="key key-sci" onclick="appendExp('sqrt(')">√</button></div>
                    <div class="key-group"><div class="key-label">x²</div><button class="key key-sci" onclick="appendExp('^2')">x²</button></div>
                    <div class="key-group"><div class="key-label shift-label">x√</div><button class="key key-sci" onclick="appendExp('^')">^</button></div>
                    <div class="key-group"><div class="key-label shift-label">e^x</div><button class="key key-sci" onclick="appendExp('ln(')">ln</button></div>
                </div>
                <!-- Row 3 -->
                <div class="key-row">
                    <div class="key-group"><div class="key-label alpha-label">A</div><button class="key key-sci" onclick="appendExp('-')">(-)</button></div>
                    <div class="key-group"><div class="key-label alpha-label">B</div><button class="key key-sci" onclick="appendExp(' deg')">°' "</button></div>
                    <div class="key-group"><div class="key-label alpha-label">C</div><button class="key key-sci" onclick="appendExp('sinh(')">hyp</button></div>
                    <div class="key-group"><div class="key-label shift-label">sin⁻¹</div><div class="key-label alpha-label right">D</div><button class="key key-sci" id="btn-sin" onclick="appendFunc('sin')">sin</button></div>
                    <div class="key-group"><div class="key-label shift-label">cos⁻¹</div><div class="key-label alpha-label right">E</div><button class="key key-sci" id="btn-cos" onclick="appendFunc('cos')">cos</button></div>
                    <div class="key-group"><div class="key-label shift-label">tan⁻¹</div><div class="key-label alpha-label right">F</div><button class="key key-sci" id="btn-tan" onclick="appendFunc('tan')">tan</button></div>
                </div>
                <!-- Row 4 -->
                <div class="key-row">
                    <div class="key-group"><div class="key-label shift-label">STO</div><button class="key key-sci" onclick="appendExp('!')">!</button></div>
                    <div class="key-group"><div class="key-label shift-label">←</div><button class="key key-sci" onclick="appendExp('%')">%</button></div>
                    <div class="key-group"><div class="key-label alpha-label">X</div><button class="key key-sci" onclick="appendExp('(')">(</button></div>
                    <div class="key-group"><div class="key-label alpha-label">Y</div><button class="key key-sci" onclick="appendExp(')')">)</button></div>
                    <div class="key-group"><div class="key-label alpha-label">M</div><button class="key key-sci" onclick="appendExp(',')">,</button></div>
                    <div class="key-group"><div class="key-label shift-label">M-</div><button class="key key-sci" onclick="appendExp('pi')">π</button></div>
                </div>
            </div>

            <!-- Numpad Area -->
            <div class="numpad-area">
                <div class="key-row">
                    <button class="key key-num" onclick="appendExp('7')">7</button>
                    <button class="key key-num" onclick="appendExp('8')">8</button>
                    <button class="key key-num" onclick="appendExp('9')">9</button>
                    <div class="key-group"><div class="key-label shift-label">INS</div><button class="key key-del" onclick="deleteLast()">DEL</button></div>
                    <div class="key-group"><div class="key-label shift-label">OFF</div><button class="key key-ac" onclick="clearAll()">AC</button></div>
                </div>
                <div class="key-row">
                    <button class="key key-num" onclick="appendExp('4')">4</button>
                    <button class="key key-num" onclick="appendExp('5')">5</button>
                    <button class="key key-num" onclick="appendExp('6')">6</button>
                    <button class="key key-num key-op" onclick="appendExp('*')">×</button>
                    <button class="key key-num key-op" onclick="appendExp('/')">÷</button>
                </div>
                <div class="key-row">
                    <button class="key key-num" onclick="appendExp('1')">1</button>
                    <button class="key key-num" onclick="appendExp('2')">2</button>
                    <button class="key key-num" onclick="appendExp('3')">3</button>
                    <button class="key key-num key-op" onclick="appendExp('+')">+</button>
                    <button class="key key-num key-op" onclick="appendExp('-')">-</button>
                </div>
                <div class="key-row">
                    <button class="key key-num" onclick="appendExp('0')">0</button>
                    <button class="key key-num" onclick="appendExp('.')">.</button>
                    <button class="key key-num" onclick="appendExp('e')">EXP</button>
                    <button class="key key-num" onclick="appendExp('Ans')">Ans</button>
                    <button class="key key-num key-eq" onclick="calculate()">=</button>
                </div>
            </div>
        </div>
    </div>

    <!-- Math.js for safe mathematical evaluations -->
    <script src="https://cdnjs.cloudflare.com/ajax/libs/mathjs/11.8.0/math.js"></script>
    <script src="script.js"></script>
</body>
</html>
:root {
    --bg-main: #2a2e35;
    --bg-screen: #9ea79c;
    --key-dark: #3a3f47;
    --key-light: #d1d5db;
    --key-del: #e27d60;
    --key-ac: #e27d60;
    --text-light: #f3f4f6;
    --text-dark: #1f2937;
    --text-shift: #eab308; /* Yellow/Orange for Shift */
    --text-alpha: #ec4899; /* Pink/Red for Alpha */
    --shadow-dark: #1a1c20;
    --shadow-light: #3e444d;
}
* {
    box-sizing: border-box;
    margin: 0;
    padding: 0;
}
body {
    background-color: #111827;
    display: flex;
    justify-content: center;
    align-items: center;
    min-height: 100vh;
    font-family: 'Inter', sans-serif;
}
.calculator-chassis {
    background: linear-gradient(145deg, #323740, #25282f);
    border-radius: 20px 20px 40px 40px;
    padding: 25px 20px;
    width: 380px;
    box-shadow: 
        0 30px 50px rgba(0,0,0,0.6),
        inset 0 2px 4px rgba(255,255,255,0.1),
        inset 0 -5px 15px rgba(0,0,0,0.4);
    border: 1px solid #4a505c;
    position: relative;
    overflow: hidden;
}
/* Header & Solar Panel */
.calc-header {
    display: flex;
    justify-content: space-between;
    align-items: flex-end;
    margin-bottom: 20px;
    padding: 0 10px;
}
.brand-section {
    display: flex;
    flex-direction: column;
}
.brand-name {
    color: #e5e7eb;
    font-weight: 700;
    font-size: 1.2rem;
    letter-spacing: 2px;
}
.model-name {
    color: #eab308;
    font-size: 0.8rem;
    font-style: italic;
    font-weight: 600;
    margin-top: 2px;
}
.model-name small {
    color: #9ca3af;
    font-size: 0.6rem;
    font-style: normal;
}
.solar-panel {
    background: #1e1e1e;
    width: 100px;
    height: 30px;
    border: 2px solid #111;
    border-radius: 2px;
    display: flex;
    gap: 2px;
    padding: 2px;
    box-shadow: inset 0 2px 5px rgba(0,0,0,0.8);
}
.solar-panel .cell {
    flex: 1;
    background: linear-gradient(to bottom, #4a2b2b, #2a1b1b);
    border-left: 1px solid #555;
}
/* Screen */
.screen-wrapper {
    background: #111;
    padding: 10px;
    border-radius: 8px;
    margin-bottom: 25px;
    box-shadow: inset 0 3px 10px rgba(0,0,0,0.8), 0 2px 0 rgba(255,255,255,0.1);
}
.screen {
    background-color: var(--bg-screen);
    border-radius: 4px;
    padding: 5px 10px;
    box-shadow: inset 0 2px 8px rgba(0,0,0,0.3);
    font-family: 'Share Tech Mono', monospace;
    position: relative;
    height: 75px;
    display: flex;
    flex-direction: column;
    justify-content: space-between;
    text-shadow: 1px 1px 0px rgba(255,255,255,0.1);
}
.indicators {
    font-size: 0.6rem;
    display: flex;
    gap: 10px;
    color: #111;
    height: 12px;
    font-weight: bold;
}
.hidden {
    opacity: 0;
}
.display-lines {
    display: flex;
    flex-direction: column;
    align-items: flex-end;
}
.expression-line {
    font-size: 1.1rem;
    color: #111;
    min-height: 20px;
    letter-spacing: 1px;
    word-break: break-all;
    text-align: left;
    width: 100%;
}
.result-line {
    font-size: 1.8rem;
    color: #000;
    font-weight: bold;
    min-height: 30px;
    letter-spacing: 1px;
}
/* Keypad General */
.keypad {
    display: flex;
    flex-direction: column;
    gap: 15px;
}
.key-row {
    display: flex;
    justify-content: space-between;
    gap: 10px;
}
.key-group {
    display: flex;
    flex-direction: column;
    align-items: center;
    position: relative;
    width: 100%;
}
.key-label {
    font-size: 0.55rem;
    font-weight: 600;
    margin-bottom: 3px;
    position: absolute;
    top: -12px;
    width: 100%;
    text-align: left;
    left: 2px;
}
.key-label.right {
    text-align: right;
    right: 2px;
    left: auto;
}
.shift-label {
    color: var(--text-shift);
}
.alpha-label {
    color: var(--text-alpha);
}
.key {
    border: none;
    border-radius: 4px;
    cursor: pointer;
    font-family: 'Inter', sans-serif;
    font-weight: 600;
    display: flex;
    justify-content: center;
    align-items: center;
    transition: all 0.1s ease;
    width: 100%;
    position: relative;
    user-select: none;
}
.key:active {
    transform: translateY(2px);
}
/* Key Types */
.key-small {
    height: 25px;
    font-size: 0.65rem;
    border-radius: 12px;
}
.key-sci {
    height: 28px;
    background: var(--key-dark);
    color: var(--text-light);
    font-size: 0.8rem;
    box-shadow: 0 4px 0 #1a1c20, 0 5px 5px rgba(0,0,0,0.4);
    border: 1px solid #4a505c;
    border-radius: 4px 4px 8px 8px;
}
.key-sci:active {
    box-shadow: 0 2px 0 #1a1c20, 0 2px 3px rgba(0,0,0,0.4);
}
.key-grey {
    background: #565d68;
    color: #fff;
    box-shadow: 0 3px 0 #2d3138, 0 4px 4px rgba(0,0,0,0.4);
}
.key-grey:active {
    box-shadow: 0 1px 0 #2d3138, 0 2px 2px rgba(0,0,0,0.4);
}
.key-num, .key-op, .key-del, .key-ac, .key-eq {
    height: 38px;
    font-size: 1.1rem;
    box-shadow: 0 4px 0 #9ca3af, 0 5px 5px rgba(0,0,0,0.3);
    border-radius: 4px 4px 10px 10px;
    background: var(--key-light);
    color: var(--text-dark);
    border: 1px solid #fff;
}
.key-num:active, .key-op:active, .key-del:active, .key-ac:active, .key-eq:active {
    box-shadow: 0 2px 0 #9ca3af, 0 2px 3px rgba(0,0,0,0.3);
}
.key-del, .key-ac {
    background: var(--key-del);
    color: #fff;
    box-shadow: 0 4px 0 #b3563b, 0 5px 5px rgba(0,0,0,0.3);
    border: 1px solid #ff9a7c;
}
.key-del:active, .key-ac:active {
    box-shadow: 0 2px 0 #b3563b, 0 2px 3px rgba(0,0,0,0.3);
}
/* Specific Layout Adjustments */
.top-controls {
    align-items: flex-end;
    margin-bottom: 5px;
}
.scientific-keys {
    display: flex;
    flex-direction: column;
    gap: 16px;
    margin-bottom: 20px;
}
.numpad-area {
    display: flex;
    flex-direction: column;
    gap: 15px;
    background: #2a2e35;
    padding: 15px 12px;
    border-radius: 10px;
    margin: 0 -10px -15px -10px;
    box-shadow: inset 0 2px 5px rgba(0,0,0,0.4);
}
/* Navigation Cluster */
.nav-cluster {
    display: flex;
    flex-direction: column;
    align-items: center;
    gap: 5px;
    background: #3e444d;
    padding: 5px;
    border-radius: 8px;
    box-shadow: inset 0 2px 4px rgba(0,0,0,0.5);
    margin: -10px 5px 0 5px;
}
.nav-row {
    display: flex;
    gap: 20px;
}
.key-nav {
    background: #565d68;
    color: #fff;
    font-size: 0.55rem;
    padding: 4px 8px;
    height: 22px;
    box-shadow: 0 2px 0 #2d3138, 0 3px 3px rgba(0,0,0,0.4);
    border-radius: 4px;
}
.key-nav:active {
    box-shadow: 0 1px 0 #2d3138, 0 1px 2px rgba(0,0,0,0.4);
    transform: translateY(1px);
}
.cursor {
    font-weight: bold;
    color: #111;
    animation: blink 1s step-end infinite;
    display: inline-block;
    width: 2px;
}
@keyframes blink {
    0%, 100% { opacity: 1; }
    50% { opacity: 0; }
}