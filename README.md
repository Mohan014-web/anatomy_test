<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1.0" />
  <title>Anatomy Test Interface</title>
  <style>
    body { font-family: Arial, sans-serif; margin: 20px; }
    .hidden { display: none; }
    #timer { font-size: 18px; font-weight: bold; margin-bottom: 10px; }
    .question { margin-bottom: 15px; }
    table { border-collapse: collapse; margin-top: 15px; }
    th, td { border: 1px solid #333; padding: 6px 10px; }
  </style>
</head>
<body>
  <div id="login-section">
    <h2>Student Login</h2>
    <label>Name: <input type="text" id="studentName"></label><br><br>
    <label>Year: <input type="text" id="studentYear"></label><br><br>
    <label>Department: <input type="text" id="studentDept"></label><br><br>
    <button onclick="startTest()">Start Test</button>
    <br><br>
    <button onclick="showAdminLogin()">Admin Login</button>
  </div>

  <div id="admin-login" class="hidden">
    <h2>Admin Login</h2>
    <label>Password: <input type="password" id="adminPass"></label>
    <button onclick="adminAuth()">Login</button>
    <button onclick="goHome()">Home</button>
  </div>

  <div id="test-section" class="hidden">
    <div id="timer"></div>
    <form id="testForm"></form>
    <button type="button" onclick="submitTest()">Submit Test</button>
  </div>

  <div id="result-section" class="hidden"></div>

  <div id="admin-panel" class="hidden">
    <h2>Admin Panel</h2>
    <button onclick="downloadCSV()">Download Results (CSV)</button>
    <button onclick="clearResults()">Clear Records</button>
    <button onclick="goHome()">Home</button>
    <div id="records"></div>
  </div>

  <script>
    const questions = [
      { q: "Which bone is known as the collar bone?", options: ["Scapula", "Clavicle", "Humerus", "Femur"], ans: 1 },
      { q: "How many chambers are there in the human heart?", options: ["2", "3", "4", "5"], ans: 2 },
      { q: "What is the functional unit of the kidney?", options: ["Neuron", "Nephron", "Alveoli", "Glomerulus"], ans: 1 },
      { q: "Which part of the brain controls balance and coordination?", options: ["Cerebrum", "Cerebellum", "Medulla", "Pons"], ans: 1 },
      { q: "Which blood cells are responsible for clotting?", options: ["RBC", "WBC", "Platelets", "Plasma"], ans: 2 },
      { q: "Which is the longest bone in the human body?", options: ["Humerus", "Tibia", "Femur", "Fibula"], ans: 2 },
      { q: "The voice box in humans is called?", options: ["Trachea", "Pharynx", "Larynx", "Epiglottis"], ans: 2 },
      { q: "Which organ is called the ‘Master Gland’?", options: ["Thyroid", "Pituitary", "Adrenal", "Pancreas"], ans: 1 },
      { q: "Which type of joint is found in the shoulder?", options: ["Hinge joint", "Ball and socket", "Pivot joint", "Gliding joint"], ans: 1 },
      { q: "Which part of the eye is responsible for vision in dim light?", options: ["Cones", "Rods", "Cornea", "Lens"], ans: 1 },
      { q: "Which artery supplies blood to the heart muscle itself?", options: ["Aorta", "Pulmonary artery", "Coronary artery", "Carotid artery"], ans: 2 },
      { q: "Where does gas exchange occur in the lungs?", options: ["Bronchi", "Bronchioles", "Alveoli", "Pleura"], ans: 2 },
      { q: "Which cranial nerve is responsible for vision?", options: ["Olfactory", "Optic", "Facial", "Vagus"], ans: 1 },
      { q: "Which muscle is known as the heart muscle?", options: ["Smooth muscle", "Cardiac muscle", "Skeletal muscle", "Voluntary muscle"], ans: 1 },
      { q: "What is the structural and functional unit of the nervous system?", options: ["Neuron", "Glial cell", "Axon", "Dendrite"], ans: 0 }
    ];

    let timer;
    let timeLeft = 600; // 10 minutes
    let lockPeriod = 300; // 5 minutes

    function startTest() {
      const name = document.getElementById('studentName').value.trim();
      const year = document.getElementById('studentYear').value.trim();
      const dept = document.getElementById('studentDept').value.trim();
      if (!name || !year || !dept) {
        alert('Please fill all fields');
        return;
      }
      document.getElementById('login-section').classList.add('hidden');
      document.getElementById('test-section').classList.remove('hidden');
      loadQuestions();
      startTimer();
      window.onbeforeunload = function () {
        if (timeLeft > (600 - lockPeriod)) {
          return "You cannot leave during the first 5 minutes.";
        }
      };
    }

    function loadQuestions() {
      const form = document.getElementById('testForm');
      questions.forEach((qObj, index) => {
        const div = document.createElement('div');
        div.className = 'question';
        div.innerHTML = `<p>Q${index+1}. ${qObj.q}</p>` +
          qObj.options.map((opt, i) => {
            return `<label><input type='radio' name='q${index}' value='${i}'> ${opt}</label><br>`;
          }).join("");
        form.appendChild(div);
      });
    }

    function startTimer() {
      timer = setInterval(() => {
        if (timeLeft <= 0) {
          clearInterval(timer);
          submitTest();
        }
        document.getElementById('timer').textContent =
          `Time Remaining: ${Math.floor(timeLeft/60)}:${('0'+timeLeft%60).slice(-2)}`;
        timeLeft--;
      }, 1000);
    }

    function submitTest() {
      clearInterval(timer);
      const form = document.getElementById('testForm');
      let score = 0;
      questions.forEach((qObj, index) => {
        const ans = form.querySelector(`input[name='q${index}']:checked`);
        if (ans && parseInt(ans.value) === qObj.ans) {
          score += 2;
        }
      });

      // Save result
      const record = {
        name: document.getElementById('studentName').value,
        year: document.getElementById('studentYear').value,
        dept: document.getElementById('studentDept').value,
        score: score
      };
      let results = JSON.parse(localStorage.getItem('results') || '[]');
      results.push(record);
      localStorage.setItem('results', JSON.stringify(results));

      document.getElementById('test-section').classList.add('hidden');
      const resultDiv = document.getElementById('result-section');
      resultDiv.innerHTML = `<h2>Test Completed</h2>
        <p>Name: ${record.name}</p>
        <p>Year: ${record.year}</p>
        <p>Department: ${record.dept}</p>
        <p>Score: ${record.score} / ${questions.length * 2}</p>
        <p><em>The window will close automatically in 3 minutes.</em></p>`;
      resultDiv.classList.remove('hidden');

      // 3-minute delay before closing
      setTimeout(() => {
        try { window.close(); }
        catch (e) { document.body.innerHTML = "<h2>Test session closed.</h2>"; }
      }, 180000);
    }

    function showAdminLogin() {
      document.getElementById('login-section').classList.add('hidden');
      document.getElementById('admin-login').classList.remove('hidden');
    }

    function adminAuth() {
      const pass = document.getElementById('adminPass').value;
      if (pass === 'admin123') {
        document.getElementById('admin-login').classList.add('hidden');
        document.getElementById('admin-panel').classList.remove('hidden');
        loadRecords();
      } else {
        alert('Incorrect password');
      }
    }

    function loadRecords() {
      let results = JSON.parse(localStorage.getItem('results') || '[]');
      let html = '<table><tr><th>Name</th><th>Year</th><th>Department</th><th>Score</th></tr>';
      results.forEach(r => {
        html += `<tr><td>${r.name}</td><td>${r.year}</td><td>${r.dept}</td><td>${r.score}</td></tr>`;
      });
      html += '</table>';
      document.getElementById('records').innerHTML = html;
    }

    function downloadCSV() {
      let results = JSON.parse(localStorage.getItem('results') || '[]');
      if (results.length === 0) {
        alert('No records found');
        return;
      }
      let csv = 'Name,Year,Department,Score\n';
      results.forEach(r => {
        csv += `${r.name},${r.year},${r.dept},${r.score}\n`;
      });
      const blob = new Blob([csv], { type: 'text/csv' });
      const url = URL.createObjectURL(blob);
      const a = document.createElement('a');
      a.href = url;
      a.download = 'results.csv';
      a.click();
      URL.revokeObjectURL(url);
    }

    function clearResults() {
      if (confirm('Are you sure you want to clear all records?')) {
        localStorage.removeItem('results');
        loadRecords();
      }
    }

    function goHome() {
      document.getElementById('admin-login').classList.add('hidden');
      document.getElementById('admin-panel').classList.add('hidden');
      document.getElementById('result-section').classList.add('hidden');
      document.getElementById('test-section').classList.add('hidden');
      document.getElementById('login-section').classList.remove('hidden');
    }
  </script>
</body>
</html>
