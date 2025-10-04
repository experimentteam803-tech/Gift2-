<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>Redeem Code</title>

  <script type="module" src="https://www.gstatic.com/firebasejs/9.6.1/firebase-app.js"></script>
  <script type="module" src="https://www.gstatic.com/firebasejs/9.6.1/firebase-database.js"></script>
  <script type="module" src="https://www.gstatic.com/firebasejs/9.6.1/firebase-analytics.js"></script>

  <style>
    /* CSS Styles (Same as before) */
    body {
      font-family: "Segoe UI", Tahoma, Geneva, Verdana, sans-serif;
      background: linear-gradient(135deg, #6a11cb, #2575fc);
      display: flex;
      justify-content: center;
      align-items: center;
      height: 100vh;
      margin: 0;
      color: #fff;
    }
    .container {
      background: rgba(255, 255, 255, 0.12);
      padding: 35px 25px;
      border-radius: 18px;
      text-align: center;
      box-shadow: 0px 8px 25px rgba(0, 0, 0, 0.3);
      width: 90%;
      max-width: 420px;
      display: none;
      backdrop-filter: blur(8px);
    }
    h1 {
      margin-bottom: 18px;
      font-size: 26px;
      font-weight: 700;
    }
    .code-box {
      background: #fff;
      color: #000;
      padding: 14px;
      border-radius: 10px;
      font-size: 22px;
      font-weight: bold;
      margin: 18px 0;
      letter-spacing: 1px;
    }
    .btn {
      margin-top: 12px;
      border: none;
      padding: 12px 20px;
      border-radius: 10px;
      cursor: pointer;
      font-weight: 600;
      width: 100%;
      font-size: 16px;
      transition: all 0.3s ease;
      margin-bottom: 5px;
    }
    .copy-btn {
      background: #ffcc00; 
      color: #000;
    }
    .copy-btn:hover {
      background: #ffaa00;
      transform: translateY(-2px);
    }
    .msg {
      margin-top: 12px;
      font-size: 14px;
      color: #ffe600;
      display: none;
    }
    .intro {
      position: fixed;
      top: 0;
      left: 0;
      width: 100%;
      height: 100%;
      background: #000;
      display: flex;
      justify-content: center;
      align-items: center;
      color: #fff;
      font-size: 28px;
      font-weight: bold;
      flex-direction: column;
      z-index: 1000;
      animation: fadeOut 2s ease forwards;
      animation-delay: 2s;
    }
    @keyframes fadeOut {
      to {
        opacity: 0;
        visibility: hidden;
      }
    }
  </style>
</head>
<body>
  <div class="intro" id="introScreen">
    🎉 Congratulations 🎉
    <p style="font-size:18px; margin-top:10px;">You have unlocked your redeem code!</p>
  </div>

  <div class="container" id="mainContent">
    <h1>🎁 Your Redeem Code</h1>
    <div class="code-box" id="redeemCode">Loading Code...</div>
    <button class="btn copy-btn" onclick="copyAndTrack()">Copy Code</button> 
    <p class="msg" id="copyMsg">✅ Code copied to clipboard and tracked!</p>
  </div>

  <textarea id="hiddenInput" style="position:absolute; left:-9999px;"></textarea>

  <script type="module">
    // 3.1. Modular Firebase Imports
    import { initializeApp } from "https://www.gstatic.com/firebasejs/9.6.1/firebase-app.js";
    import { getDatabase, ref, set, get, child } from "https://www.gstatic.com/firebasejs/9.6.1/firebase-database.js";
    import { getAnalytics, logEvent } from "https://www.gstatic.com/firebasejs/9.6.1/firebase-analytics.js";
    
    // 3.2. NEW Firebase Configuration
    const firebaseConfig = {
      apiKey: "AIzaSyB0vsW1fivhOVec40Pif8Z_mDbnLH9vV80",
      authDomain: "gift-code-wellcome.firebaseapp.com",
      projectId: "gift-code-wellcome",
      storageBucket: "gift-code-wellcome.firebasestorage.app",
      messagingSenderId: "463243795581",
      appId: "1:463243795581:web:ac978a50a2798590e7a514",
      measurementId: "G-Y20QR716LV"
    };

    // Initialize Firebase and Database
    const app = initializeApp(firebaseConfig);
    const database = getDatabase(app);
    const analytics = getAnalytics(app);
    
    // 3.3. Database Tracking Function (Updated for Modular SDK)
    async function trackAndSaveCode(code) {
      const uniqueKey = Date.now(); // Simple unique ID
      const dbRef = ref(database); // Database reference
      
      try {
          await set(ref(database, 'redeem_code_clicks/' + uniqueKey), {
              code: code,
              timestamp: new Date().toISOString(),
              referrer: document.referrer || 'direct'
          });
          console.log("Code tracked successfully in Firebase!");
          // Optional: Log custom event to Analytics
          logEvent(analytics, 'code_copied', { code: code, redirect_url: "https://otieu.com/4/9966942" });
      } catch (error) {
          console.error("Firebase tracking failed:", error);
      }
    }

    // 3.4. Load Active Code Function (Updated for Modular SDK)
    async function loadActiveCode() {
        const codeDisplay = document.getElementById("redeemCode");
        const dbRef = ref(database); 
        
        try {
            // 'active_gift_code/code' path se value fetch karna
            const snapshot = await get(child(dbRef, 'active_gift_code/'));
            
            if (snapshot.exists() && snapshot.val().code) {
                const activeCode = snapshot.val().code;
                codeDisplay.textContent = activeCode;
            } else {
                codeDisplay.textContent = 'ERROR! Set Code in Admin Panel.';
            }
        } catch (error) {
            codeDisplay.textContent = 'ERROR LOADING CODE';
            console.error("Error fetching active code:", error);
        }
    }

    // 3.5. Main Action Function (Global function access ke liye ise window object mein rakha gaya hai)
    window.copyAndTrack = async function() {
      const code = document.getElementById("redeemCode").innerText;
      const hiddenInput = document.getElementById("hiddenInput");
      
      // 1. Code Copy Logic
      hiddenInput.value = code;
      hiddenInput.select();
      hiddenInput.setSelectionRange(0, 99999);
      document.execCommand("copy");
      
      // Show confirmation message
      document.getElementById("copyMsg").style.display = "block";
      setTimeout(() => {
        document.getElementById("copyMsg").style.display = "none";
      }, 2000);

      // 2. Track in Firebase 
      await trackAndSaveCode(code); 

      // 3. Redirect to the link
      setTimeout(() => {
        window.location.href = "https://otieu.com/4/9966942";
      }, 500); 
    };

    // 3.6. Initialization and Intro Screen Logic
    setTimeout(() => {
      document.getElementById("introScreen").style.opacity = 0;
      document.getElementById("introScreen").style.visibility = 'hidden';
      document.getElementById("mainContent").style.display = "block";
      
      // Active code load karna
      loadActiveCode(); 
    }, 2500);
  </script>
</body>
</html>
