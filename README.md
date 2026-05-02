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
