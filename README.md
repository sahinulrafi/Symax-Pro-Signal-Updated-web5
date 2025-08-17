<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="utf-8"/>
  <meta name="viewport" content="width=device-width, initial-scale=1"/>
  <title>Quotex Pair Signal Simulator (Currency + OTC only)</title>
  <style>
    body { background:#111; color:#ddd; font-family: monospace; padding:20px; }
    .pair-btn { display:block; margin:5px 0; padding:8px; background:#222; border:none; color:#ddd; cursor:pointer; width:100%; max-width:300px; text-align:left; }
    #signal { margin-top:20px; font-size:1.2em; }
    .up { color:#0f0; }
    .down { color:#f00; }
    .avoid { color:#ff0; }
  </style>
</head>
<body>
  <h2>Quotex Pair Signal Simulator</h2>
  <div id="pairs"></div>

  <div id="signal">Select a pair above…</div>
  <button id="gen" disabled>Generate Direction (1 min lock)</button>

  <script>
    // Currency pairs — common majors, minors, a couple exotic
    const basePairs = [
      'EURUSD','GBPUSD','USDJPY','USDCHF','AUDUSD','USDCAD','NZDUSD',
      'EURGBP','EURJPY','GBPJPY','AUDJPY','EURCHF','AUDCAD','NZDJPY'
    ];
    const includeOTC = true;
    const pairs = includeOTC
      ? basePairs.reduce((a,p) => a.concat(p, p+'-OTC'), [])
      : basePairs.slice();

    const container = document.getElementById('pairs');
    const signalDiv = document.getElementById('signal');
    const genBtn = document.getElementById('gen');
    let selected = null;
    let lockedUntil = 0, timerId = null;

    pairs.forEach(p => {
      const btn = document.createElement('button');
      btn.textContent = p;
      btn.className = 'pair-btn';
      btn.onclick = () => {
        selected = p;
        signalDiv.textContent = `Selected: ${p}`;
        genBtn.disabled = Date.now() < lockedUntil;
      };
      container.appendChild(btn);
    });

    genBtn.onclick = () => {
      if (!selected || Date.now() < lockedUntil) return;
      const r = Math.random();
      let dirText, dirClass;
      if (r < 0.45) { dirText = 'UP'; dirClass = 'up'; }
      else if (r < 0.90) { dirText = 'DOWN'; dirClass = 'down'; }
      else { dirText = 'AVOID'; dirClass = 'avoid'; }

      signalDiv.innerHTML = `Pair: ${selected} → <span class="${dirClass}">${dirText}</span>`;
      lockedUntil = Date.now() + 60 * 1000;
      genBtn.disabled = true;

      if (timerId) clearInterval(timerId);
      timerId = setInterval(() => {
        const rem = Math.max(0, lockedUntil - Date.now());
        const sec = Math.ceil(rem / 1000);
        if (rem <= 0) {
          clearInterval(timerId);
          genBtn.disabled = false;
          signalDiv.textContent = `Pair: ${selected} → <span class="${dirClass}">${dirText}</span>`;
        } else {
          genBtn.textContent = `Wait ${sec}s...`;
        }
      }, 200);

      genBtn.textContent = 'Wait 60s...';
    };
  </script>
</body>
</html>