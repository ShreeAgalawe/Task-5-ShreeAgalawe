# Task-5-ShreeAgalawe
<!DOCTYPE HTML>
<html>
<head>
  <title>Smart Pet Bowl Monitor</title>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1">
  <style>
    :root {
      --primary-text-color: #2c3e50;
      --secondary-text-color: #7f8c8d;
      --accent-color: #1abc9c;
      --status-ok-color: #2ecc71;
      --status-alert-color: #e74c3c;
      --shadow-color: rgba(0, 0, 0, 0.12);
      --overlay-color: rgba(244, 247, 246, 0.85);
      --card-bg-color: rgba(255, 255, 255, 0.95);
    }
    body {
      font-family: -apple-system, BlinkMacSystemFont, 'Segoe UI', Roboto, Helvetica, Arial, sans-serif;
      color: var(--primary-text-color);
      margin: 0; padding: 20px; min-height: 100vh;
      display: flex; flex-direction: column; align-items: center; justify-content: center;
      position: relative; overflow-x: hidden;
      background-image: url("https://via.placeholder.com/1920x1080/cfe8fc/7f8c8d?text=PetFeederBackground");
      background-size: cover; background-position: center; background-attachment: fixed;
    }
    body::before {
      content: ''; position: absolute; top: 0; left: 0; right: 0; bottom: 0;
      background-color: var(--overlay-color); z-index: -1;
    }
    .header { text-align: center; margin-bottom: 40px; z-index: 1; }
    .header h1 { font-size: 2.8em; font-weight: 700; margin: 0; }
    .card-container { display: flex; justify-content: center; gap: 25px; z-index: 1; }
    .card {
      background-color: var(--card-bg-color); backdrop-filter: blur(5px);
      border-radius: 12px; box-shadow: 0 6px 20px var(--shadow-color);
      padding: 25px 30px; min-width: 280px; text-align: left;
      border-top: 5px solid var(--accent-color);
    }
    .card-header { display: flex; align-items: center; gap: 15px; }
    .card-header .icon { width: 32px; height: 32px; fill: var(--accent-color); }
    .card-header h2 { margin: 0; font-size: 1.3em; font-weight: 500; color: var(--secondary-text-color); }
    .reading-container { text-align: center; margin-top: 25px; }
    .reading { font-size: 4em; font-weight: 700; }
    .unit { font-size: 1.3em; color: var(--secondary-text-color); margin-left: 8px; }
    .controls { text-align: center; margin-top: 40px; z-index: 1; }
    #alertStatus {
      padding: 15px 35px; font-size: 1.1em; font-weight: 600; color: #fff;
      border: none; border-radius: 8px;
      display: inline-flex; align-items: center; gap: 10px;
      box-shadow: 0 4px 12px rgba(0,0,0,0.15);
      transition: background-color 0.3s ease;
    }
    #alertStatus.ok { background-color: var(--status-ok-color); }
    #alertStatus.alert { background-color: var(--status-alert-color); }
    #alertStatus .icon { width: 22px; height: 22px; fill: #fff; }
    @keyframes highlight {
      from { color: var(--accent-color); transform: scale(1.05); }
      to { color: var(--primary-text-color); transform: scale(1); }
    }
    .updated { animation: highlight 0.5s ease; }
  </style>
</head>
<body>

  <div class="header"><h1>🐾 Smart Pet Bowl Monitor</h1></div>
  
  <div class="card-container">
    <div class="card">
      <div class="card-header">
        <svg class="icon" xmlns="http://www.w3.org/2000/svg" viewBox="0 0 24 24"><path d="M12 2.69l5.31 5.31c.39.39.39 1.02 0 1.41-.39.39-1.02.39-1.41 0L13 6.81V15c0 .55-.45 1-1 1s-1-.45-1-1V6.81l-2.89 2.6c-.39.39-1.02.39-1.41 0-.39-.39-.39-1.02 0-1.41L12 2.69zM4 16c0-.55-.45-1-1-1s-1 .45-1 1v4c0 1.1.9 2 2 2h14c1.1 0 2-.9 2-2v-4c0-.55-.45-1-1-1s-1 .45-1 1v3H5v-3z"/></svg>
        <h2>Bowl Status</h2>
      </div>
      <div class="reading-container">
        <span id="distance" class="reading">...</span>
        <span class="unit" style="display: none;">cm</span>
      </div>
    </div>
  </div>

  <div class="controls">
    <div id="alertStatus" class="ok">
      <svg class="icon" xmlns="http://www.w3.org/2000/svg" viewBox="0 0 24 24"><path d="M12 2C6.48 2 2 6.48 2 12s4.48 10 10 10 10-4.48 10-10S17.52 2 12 2zm-2 15l-5-5 1.41-1.41L10 14.17l7.59-7.59L19 8l-9 9z"/></svg>
      <span id="alertText">Status: OK</span>
    </div>
  </div>
    
  <script type="module">
    import { initializeApp } from "https://www.gstatic.com/firebasejs/9.22.0/firebase-app.js";
    import { getDatabase, ref, onValue } from "https://www.gstatic.com/firebasejs/9.22.0/firebase-database.js";
    
    // Your Firebase configuration object (SENSITIVE DATA REMOVED)
    const firebaseConfig = {
      apiKey: "YOUR_API_KEY",
      authDomain: "YOUR_PROJECT_ID.firebaseapp.com",
      databaseURL: "https://YOUR_PROJECT_ID-default-rtdb.REGION.firebasedatabase.app",
      projectId: "YOUR_PROJECT_ID",
      storageBucket: "YOUR_PROJECT_ID.firebasestorage.app",
      messagingSenderId: "YOUR_MESSAGING_SENDER_ID",
      appId: "YOUR_APP_ID",
      measurementId: "YOUR_MEASUREMENT_ID"
    };

    const app = initializeApp(firebaseConfig);
    const database = getDatabase(app);

    const distanceEl = document.getElementById('distance');
    const unitEl = document.querySelector('.unit');
    const alertStatusEl = document.getElementById('alertStatus');
    const alertTextEl = document.getElementById('alertText');
    
    // Listen for the Water Level from Firebase
    const levelRef = ref(database, '/PetFeeder/Level_cm');
    onValue(levelRef, (snapshot) => {
      const data = snapshot.val();
      if (data !== null) {
        
        // Logic to display words instead of numbers
        let statusText = '';
        
        // If distance is small, bowl is Full. If large, it's Empty.
        if (data < 15) { // Threshold is 15cm, adjust if needed
            statusText = "Full";
        } else {
            statusText = "Empty";
        }

        // Update the display with the status text
        distanceEl.innerHTML = statusText;
        distanceEl.style.fontSize = "3em"; // Adjust font size for words
        unitEl.style.display = "none"; // Hide the "cm" unit

        // Animate the update
        distanceEl.classList.add('updated');
        setTimeout(() => distanceEl.classList.remove('updated'), 500);
      }
    });

    // Listen for the Alert Status from Firebase
    const alertRef = ref(database, '/PetFeeder/LowAlert');
    onValue(alertRef, (snapshot) => {
        const isAlertActive = snapshot.val();
        if (isAlertActive === true) {
            alertStatusEl.classList.remove('ok');
            alertStatusEl.classList.add('alert');
            alertTextEl.textContent = 'Alert: ON';
        } else {
            alertStatusEl.classList.remove('alert');
            alertStatusEl.classList.add('ok');
            alertTextEl.textContent = 'Status: OK';
        }
    });
  </script>

</body>
</html>
