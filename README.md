# safe-password-generator
<!DOCTYPE html>
<html lang="hi">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Safe Password Generator | मजबूत पासवर्ड बनाएं</title>
    <style>
        * {
            box-sizing: border-box;
            margin: 0;
            padding: 0;
            font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
        }
        body {
            background-color: #0f172a;
            color: #f8fafc;
            display: flex;
            justify-content: center;
            align-items: center;
            min-height: 100vh;
            padding: 20px;
        }
        .container {
            background-color: #1e293b;
            padding: 30px;
            border-radius: 16px;
            box-shadow: 0 10px 25px rgba(0,0,0,0.3);
            width: 100%;
            max-width: 450px;
        }
        h1 {
            font-size: 24px;
            margin-bottom: 20px;
            text-align: center;
            color: #38bdf8;
        }
        .result-container {
            background-color: #0f172a;
            display: flex;
            justify-content: space-between;
            align-items: center;
            padding: 12px 16px;
            border-radius: 8px;
            margin-bottom: 20px;
            border: 1px solid #334155;
            position: relative;
        }
        #password {
            background: transparent;
            border: none;
            color: #fff;
            font-size: 18px;
            font-family: monospace;
            width: 80%;
            outline: none;
        }
        .btn-copy {
            background-color: #38bdf8;
            color: #0f172a;
            border: none;
            padding: 8px 12px;
            border-radius: 6px;
            cursor: pointer;
            font-weight: bold;
            transition: 0.2s;
        }
        .btn-copy:hover {
            background-color: #0ea5e9;
        }
        .setting {
            display: flex;
            justify-content: space-between;
            align-items: center;
            margin-bottom: 15px;
        }
        label {
            color: #cbd5e1;
            font-size: 15px;
        }
        input[type="checkbox"] {
            width: 18px;
            height: 18px;
            cursor: pointer;
            accent-color: #38bdf8;
        }
        input[type="range"] {
            width: 60%;
            accent-color: #38bdf8;
        }
        .btn-generate {
            width: 100%;
            background-color: #10b981;
            color: white;
            border: none;
            padding: 14px;
            border-radius: 8px;
            font-size: 16px;
            font-weight: bold;
            cursor: pointer;
            margin-top: 10px;
            transition: 0.2s;
        }
        .btn-generate:hover {
            background-color: #059669;
        }
        .strength-meter {
            height: 6px;
            background-color: #334155;
            border-radius: 3px;
            margin-top: -10px;
            margin-bottom: 20px;
            overflow: hidden;
        }
        #strength-bar {
            height: 100%;
            width: 0%;
            transition: 0.3s;
        }
    </style>
</head>
<body>

<div class="container">
    <h1>Safe Password Generator</h1>
    
    <div class="result-container">
        <input type="text" id="password" readonly placeholder="यहाँ पासवर्ड दिखेगा">
        <button class="btn-copy" id="copy-btn">Copy</button>
    </div>

    <div class="strength-meter">
        <div id="strength-bar"></div>
    </div>

    <div class="setting">
        <label>पासवर्ड की लंबाई (<span id="length-val">12</span>)</label>
        <input type="range" id="length" min="6" max="30" value="12">
    </div>

    <div class="setting">
        <label>बड़े अक्षर शामिल करें (A-Z)</label>
        <input type="checkbox" id="uppercase" checked>
    </div>

    <div class="setting">
        <label>छोटे अक्षर शामिल करें (a-z)</label>
        <input type="checkbox" id="lowercase" checked>
    </div>

    <div class="setting">
        <label>नंबर शामिल करें (0-9)</label>
        <input type="checkbox" id="numbers" checked>
    </div>

    <div class="setting">
        <label>स्पेशल कैरेक्टर शामिल करें (!@#$%)</label>
        <input type="checkbox" id="symbols" checked>
    </div>

    <button class="btn-generate" id="generate-btn">नया पासवर्ड बनाएं</button>
</div>

<script>
    const passwordEl = document.getElementById('password');
    const lengthEl = document.getElementById('length');
    const lengthValEl = document.getElementById('length-val');
    const uppercaseEl = document.getElementById('uppercase');
    const lowercaseEl = document.getElementById('lowercase');
    const numbersEl = document.getElementById('numbers');
    const symbolsEl = document.getElementById('symbols');
    const generateBtn = document.getElementById('generate-btn');
    const copyBtn = document.getElementById('copy-btn');
    const strengthBar = document.getElementById('strength-bar');

    const randomFunc = {
        lower: () => String.fromCharCode(Math.floor(Math.random() * 26) + 97),
        upper: () => String.fromCharCode(Math.floor(Math.random() * 26) + 65),
        number: () => String.fromCharCode(Math.floor(Math.random() * 10) + 48),
        symbol: () => {
            const symbols = '!@#$%^&*()_+~`|}{[]:;?><,./-=';
            return symbols[Math.floor(Math.random() * symbols.length)];
        }
    };

    lengthEl.addEventListener('input', (e) => {
        lengthValEl.innerText = e.target.value;
    });

    generateBtn.addEventListener('click', () => {
        const length = +lengthEl.value;
        const hasLower = lowercaseEl.checked;
        const hasUpper = uppercaseEl.checked;
        const hasNumber = numbersEl.checked;
        const hasSymbol = symbolsEl.checked;

        passwordEl.value = generatePassword(hasLower, hasUpper, hasNumber, hasSymbol, length);
        updateStrength(length, hasLower, hasUpper, hasNumber, hasSymbol);
    });

    copyBtn.addEventListener('click', () => {
        const password = passwordEl.value;
        if(!password) return;
        
        navigator.clipboard.writeText(password);
        copyBtn.innerText = "Copied!";
        copyBtn.style.backgroundColor = "#10b981";
        
        setTimeout(() => {
            copyBtn.innerText = "Copy";
            copyBtn.style.backgroundColor = "#38bdf8";
        }, 2000);
    });

    function generatePassword(lower, upper, number, symbol, length) {
        let generatedPassword = '';
        const typesCount = lower + upper + number + symbol;
        const typesArr = [{lower}, {upper}, {number}, {symbol}].filter(item => Object.values(item)[0]);
        
        if(typesCount === 0) return '';

        for(let i = 0; i < length; i += typesCount) {
            typesArr.forEach(type => {
                const funcName = Object.keys(type)[0];
                generatedPassword += randomFunc[funcName]();
            });
        }

        return generatedPassword.slice(0, length).split('').sort(() => Math.random() - 0.5).join('');
    }

    function updateStrength(length, lower, upper, number, symbol) {
        const typesCount = lower + upper + number + symbol;
        let score = length * 2 + typesCount * 10;
        
        if (score < 35) {
            strengthBar.style.width = '30%';
            strengthBar.style.backgroundColor = '#ef4444'; // Red
        } else if (score < 50) {
            strengthBar.style.width = '60%';
            strengthBar.style.backgroundColor = '#f59e0b'; // Yellow
        } else {
            strengthBar.style.width = '100%';
            strengthBar.style.backgroundColor = '#10b981'; // Green
        }
    }

    // Auto generate on load

    generateBtn.click();
</script>

</body>
</html>
