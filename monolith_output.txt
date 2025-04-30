# Monolithic Code File

---
=====================================
# popup.css
=====================================

/* popup.css */
body {
  font-family: system-ui, sans-serif;
  padding: 12px;
  width: 320px;
  background: #1e1e1e;
  color: #e0e0e0;
}

h1 {
  font-size: 18px;
  margin-bottom: 8px;
  color: #ffffff;
}

#rulesArea {
  width: 100%;
  height: 200px;
  font-family: monospace;
  font-size: 13px;
  padding: 8px;
  box-sizing: border-box;
  background: #2e2e2e;
  color: #e0e0e0;
  border: 1px solid #555555;
  border-radius: 4px;
}

.buttons {
  margin-top: 8px;
  display: flex;
  gap: 8px;
}

button {
  flex: 1;
  padding: 6px 0;
  font-size: 14px;
  cursor: pointer;
  border: none;
  border-radius: 4px;
  background: #4b5563;
  color: #e0e0e0;
  transition: background .2s ease;
}

button:hover {
  background: #374151;
}

#status {
  margin-top: 8px;
  font-size: 12px;
  color: #90ee90;
}

---
=====================================
# popup.html
=====================================

<!-- src/popup/popup.html -->
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8" />
  <title>Extension Settings</title>
  <link rel="stylesheet" href="popup.css" />
</head>
<body>
  <h1>Scraping Rules</h1>
  <textarea id="rulesArea" placeholder="Paste JSON rules here…"></textarea>
  <div class="buttons">
    <button id="saveBtn">Save</button>
    <button id="resetBtn">Reset to defaults</button>
  </div>
  <div id="status"></div>
  <script type="module" src="popup.js"></script>
</body>
</html>

---
=====================================
# popup.js
=====================================

const DEFAULT_RULES = chrome.runtime.getURL('rules.json');

async function loadDefaults() {
  const resp = await fetch(DEFAULT_RULES);
  return resp.json();
}

function showStatus(msg, success = true) {
  const el = document.getElementById('status');
  el.textContent = msg;
  el.style.color = success ? 'green' : 'red';
  setTimeout(() => (el.textContent = ''), 2000);
}

document.addEventListener('DOMContentLoaded', async () => {
  const area = document.getElementById('rulesArea');
  // Load saved or fallback to defaults
  chrome.storage.local.get({ rules: null }, async data => {
    let rulesText;
    if (data.rules) {
      rulesText = JSON.stringify(data.rules, null, 2);
    } else {
      const defs = await loadDefaults();
      rulesText = JSON.stringify(defs, null, 2);
    }
    area.value = rulesText;
  });

  document.getElementById('saveBtn').addEventListener('click', () => {
    try {
      const parsed = JSON.parse(area.value);
      chrome.storage.local.set({ rules: parsed }, () => {
        showStatus('Saved!');
      });
    } catch (e) {
      showStatus('Invalid JSON', false);
    }
  });

  document.getElementById('resetBtn').addEventListener('click', async () => {
    const defs = await loadDefaults();
    const text = JSON.stringify(defs, null, 2);
    document.getElementById('rulesArea').value = text;
    chrome.storage.local.set({ rules: defs }, () => {
      showStatus('Reset to defaults');
    });
  });
});

