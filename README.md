# Online_quiz
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1.0"/>
  <title>Online Quiz</title>
  <link href="https://fonts.googleapis.com/css2?family=Poppins:wght@400;600&display=swap" rel="stylesheet">
  <style>
    :root {
      --bg: #ffffff;
      --text: #222;
      --card: #f9f9f9;
      --accent: #007BFF;
    }
    body.dark {
      --bg: #1e1e1e;
      --text: #eee;
      --card: #2c2c2c;
      --accent: #00bfff;
    }

    body {
      font-family: 'Poppins', sans-serif;
      background-color: var(--bg);
      color: var(--text);
      margin: 0;
      padding: 20px;
      transition: background 0.4s, color 0.4s;
    }

    .container {
      max-width: 700px;
      margin: auto;
    }

    .toggle-mode {
      text-align: right;
      margin-bottom: 10px;
    }

    .card {
      background-color: var(--card);
      padding: 25px;
      border-radius: 16px;
      box-shadow: 0 10px 25px rgba(0, 0, 0, 0.1);
      perspective: 1000px;
      transition: background 0.4s;
      margin-top: 20px;
    }

    .card-content {
      transition: transform 0.8s;
      transform-style: preserve-3d;
      position: relative;
    }

    .card.flip .card-content {
      transform: rotateY(180deg);
    }

    .front, .back {
      backface-visibility: hidden;
      position: absolute;
      top: 0; left: 0; right: 0; bottom: 0;
      padding: 20px;
    }

    .back {
      transform: rotateY(180deg);
    }

    h2 {
      font-size: 22px;
      margin-bottom: 20px;
    }

    label {
      display: block;
      margin-bottom: 10px;
    }

    input[type="text"], input[type="password"] {
      padding: 8px;
      width: 100%;
      margin-top: 4px;
      margin-bottom: 16px;
      border-radius: 6px;
      border: 1px solid #ccc;
    }

    button {
      padding: 10px 20px;
      background-color: var(--accent);
      color: white;
      border: none;
      border-radius: 6px;
      cursor: pointer;
      transition: background 0.3s;
    }

    button:hover {
      background-color: #0056b3;
    }

    .options label {
      margin: 10px 0;
      display: block;
      cursor: pointer;
    }

    .navigation-buttons {
      display: flex;
      justify-content: space-between;
      margin-top: 20px;
    }

    .submit-button {
      width: 100%;
      margin-top: 20px;
      background-color: #28a745;
    }

    .submit-button:hover {
      background-color: #218838;
    }

    .progress-bar {
      height: 10px;
      background-color: #ccc;
      border-radius: 5px;
      overflow: hidden;
      margin: 20px 0;
    }

    .progress-bar div {
      height: 100%;
      background-color: var(--accent);
      width: 0%;
      transition: width 0.4s ease-in-out;
    }

    .result { margin-top: 30px; }
    .correct { color: green; }
    .incorrect { color: red; }

    #quizCard { display: none; }
  </style>
</head>
<body>
  <div class="container">
    <div class="toggle-mode">
      <label><input type="checkbox" id="darkModeToggle"> Dark Mode</label>
    </div>

    <!-- Login Card -->
    <div class="card" id="loginCard">
      <h2>Login</h2>
      <label>Username</label>
      <input type="text" id="username" placeholder="Enter Username">
      <label>Password</label>
      <input type="password" id="password" placeholder="Enter Password">
      <button onclick="login()">Login</button>
    </div>

    <!-- Quiz Card -->
    <div class="card" id="quizCard">
      <div class="card-content">
        <div class="front">
          <h2 id="question">Question Text</h2>
          <div id="options" class="options"></div>
          <div class="progress-bar"><div id="progress"></div></div>
          <div class="navigation-buttons">
            <button id="prevBtn">Previous</button>
            <button id="nextBtn">Next</button>
          </div>
          <button class="submit-button" id="submitBtn">Submit Quiz</button>
        </div>
        <div class="back" id="result"></div>
      </div>
    </div>
  </div>

  <script>
    const quizData = [
      { question: "Capital of France?", options: ["Paris", "Berlin", "Rome", "London"], answer: "Paris" },
      { question: "HTML stands for?", options: ["HyperText Markup", "Home Tool", "Hyper Transfer", "Hyper Text Makeup"], answer: "HyperText Markup" },
      { question: "Largest planet?", options: ["Jupiter", "Saturn", "Mars", "Earth"], answer: "Jupiter" },
      { question: "JS keyword for variable?", options: ["int", "var", "define", "dim"], answer: "var" },
      { question: "Who painted Mona Lisa?", options: ["Da Vinci", "Picasso", "Rembrandt", "Van Gogh"], answer: "Da Vinci" }
    ];

    let currentQuestion = 0;
    const userAnswers = new Array(quizData.length).fill(null);
    const questionEl = document.getElementById("question");
    const optionsEl = document.getElementById("options");
    const prevBtn = document.getElementById("prevBtn");
    const nextBtn = document.getElementById("nextBtn");
    const submitBtn = document.getElementById("submitBtn");
    const resultEl = document.getElementById("result");
    const progress = document.getElementById("progress");

    function loadQuestion(index) {
      const q = quizData[index];
      questionEl.textContent = `${index + 1}. ${q.question}`;
      optionsEl.innerHTML = "";
      q.options.forEach(opt => {
        const label = document.createElement("label");
        const input = document.createElement("input");
        input.type = "radio";
        input.name = "answer";
        input.value = opt;
        if (userAnswers[index] === opt) input.checked = true;
        label.appendChild(input);
        label.appendChild(document.createTextNode(" " + opt));
        optionsEl.appendChild(label);
      });
      prevBtn.disabled = index === 0;
      nextBtn.disabled = index === quizData.length - 1;
      progress.style.width = ((index + 1) / quizData.length * 100) + "%";
    }

    function getSelected() {
      const opts = document.getElementsByName("answer");
      for (let o of opts) if (o.checked) return o.value;
      return null;
    }

    prevBtn.onclick = () => {
      const selected = getSelected();
      if (selected) userAnswers[currentQuestion] = selected;
      if (currentQuestion > 0) currentQuestion--;
      loadQuestion(currentQuestion);
    };

    nextBtn.onclick = () => {
      const selected = getSelected();
      if (selected) userAnswers[currentQuestion] = selected;
      if (currentQuestion < quizData.length - 1) currentQuestion++;
      loadQuestion(currentQuestion);
    };

    submitBtn.onclick = () => {
      const selected = getSelected();
      if (selected) userAnswers[currentQuestion] = selected;
      if (userAnswers.includes(null)) return alert("Please answer all questions.");
      let score = 0;
      let html = "<h2>Your Results</h2>";
      quizData.forEach((q, i) => {
        const userAns = userAnswers[i];
        const correct = userAns === q.answer;
        if (correct) score++;
        html += `<p><strong>${i + 1}. ${q.question}</strong><br>
        Your Answer: <span class="${correct ? 'correct' : 'incorrect'}">${userAns}</span><br>
        ${!correct ? `Correct: <span class="correct">${q.answer}</span>` : ""}</p>`;
      });
      html += `<h3>Score: ${score}/${quizData.length}</h3>`;
      resultEl.innerHTML = html;
      document.getElementById("quizCard").classList.add("flip");
    };

    function login() {
      const user = document.getElementById("username").value;
      const pass = document.getElementById("password").value;
      if (user === "admin" && pass === "1234") {
        document.getElementById("loginCard").style.display = "none";
        document.getElementById("quizCard").style.display = "block";
        loadQuestion(currentQuestion);
      } else {
        alert("Invalid credentials. Try again.");
      }
    }

    document.getElementById("darkModeToggle").addEventListener("change", (e) => {
      document.body.classList.toggle("dark", e.target.checked);
    });
  </script>
</body>
</html>
