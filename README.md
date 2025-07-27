<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1.0"/>
  <title>Wellness Check-In Chatbot</title>
  <style>
    body {
      font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
      background: #e8f0fe;
      margin: 0;
      padding: 0;
      display: flex;
      flex-direction: column;
      align-items: center;
      min-height: 100vh;
    }
    #chat {
      background: #fff;
      border-radius: 10px;
      box-shadow: 0 8px 20px rgba(0,0,0,0.1);
      width: 100%;
      max-width: 500px;
      padding: 20px;
      margin: 30px auto 10px;
      box-sizing: border-box;
      overflow-y: auto;
      flex-grow: 1;
    }
    .bot, .user {
      margin: 10px 0;
      padding: 12px 16px;
      border-radius: 20px;
      max-width: 80%;
      line-height: 1.5;
      word-wrap: break-word;
    }
    .bot {
      background: #f0f4ff;
      color: #333;
      align-self: flex-start;
      border-bottom-left-radius: 4px;
    }
    .user {
      background: #cce5ff;
      color: #003366;
      align-self: flex-end;
      margin-left: auto;
      text-align: right;
      border-bottom-right-radius: 4px;
    }
    .options {
      display: flex;
      flex-wrap: wrap;
      gap: 10px;
      margin-top: 10px;
    }
    button {
      background-color: #4285f4;
      color: white;
      border: none;
      padding: 10px 16px;
      border-radius: 20px;
      font-size: 14px;
      cursor: pointer;
      transition: background-color 0.2s ease;
    }
    button:hover {
      background-color: #3367d6;
    }
    #downloadBtn {
      margin-bottom: 20px;
      display: none;
    }
    @media screen and (max-width: 600px) {
      #chat {
        margin: 10px;
      }
      .bot, .user {
        max-width: 100%;
      }
      button {
        flex: 1 1 100%;
      }
    }
  </style>
</head>
<body>
  <div id="chat">
    <div class="bot">🤖 Hi! I'm your wellness assistant. May I ask a few quick questions about how you're feeling lately?</div>
    <div class="options">
      <button onclick="startChat()">Yes</button>
      <button onclick="endChat()">No</button>
    </div>
  </div>
  <button id="downloadBtn" onclick="downloadTranscript()">Download Chat Transcript</button>

  <script>
    const phq9 = [
      "Little interest or pleasure in doing things?",
      "Feeling down, depressed, or hopeless?",
      "Trouble falling or staying asleep, or sleeping too much?",
      "Feeling tired or having little energy?",
      "Poor appetite or overeating?",
      "Feeling bad about yourself — or that you are a failure or have let yourself or your family down?",
      "Trouble concentrating on things?",
      "Moving or speaking slowly or being fidgety/restless?",
      "Thoughts that you would be better off dead or hurting yourself?"
    ];
    const gad7 = [
      "Feeling nervous, anxious, or on edge?",
      "Not being able to stop or control worrying?",
      "Worrying too much about different things?",
      "Trouble relaxing?",
      "Being so restless that it's hard to sit still?",
      "Becoming easily annoyed or irritable?",
      "Feeling afraid as if something awful might happen?"
    ];
    let currentQ = 0;
    let mode = "PHQ";
    let score = 0;
    let chatLog = [];

    function startChat() {
      clearOptions();
      document.getElementById("downloadBtn").style.display = "block";
      askNextQuestion();
    }

    function askNextQuestion() {
      const questionSet = (mode === "PHQ") ? phq9 : gad7;
      if (currentQ < questionSet.length) {
        addBotMessage(${currentQ + 1}. ${questionSet[currentQ]});
        showOptions();
      } else {
        if (mode === "PHQ") {
          evaluate(score, "PHQ-9");
          score = 0;
          currentQ = 0;
          mode = "GAD";
          setTimeout(() => {
            addBotMessage("Now let's continue with the GAD-7 anxiety check.");
            askNextQuestion();
          }, 1500);
        } else {
          evaluate(score, "GAD-7");
          addBotMessage("💚 You're not alone. Reach out anytime.");
          clearOptions();
        }
      }
    }

    function showOptions() {
      const opts = ["0 – Not at all", "1 – Several days", "2 – More than half the days", "3 – Nearly every day"];
      const optDiv = document.createElement("div");
      optDiv.className = "options";
      opts.forEach((opt, val) => {
        const btn = document.createElement("button");
        btn.textContent = opt;
        btn.onclick = () => handleAnswer(val);
        btn.setAttribute("aria-label", Answer: ${opt});
        optDiv.appendChild(btn);
      });
      document.getElementById("chat").appendChild(optDiv);
    }

    function handleAnswer(val) {
      const labels = ["Not at all", "Several days", "More than half the days", "Nearly every day"];
      score += val;
      addUserMessage(labels[val]);
      currentQ++;
      clearOptions();
      askNextQuestion();
    }

    function evaluate(score, type) {
      let msg = `Your ${type} score is ${score}. `;
      if (score <= 4) {
        msg += "✅ Minimal symptoms. Keep up your self-care! Try short meditations.";
      } else if (score <= 9) {
        msg += "🌱 Mild symptoms. Journaling, mindful walking, or breathing exercises may help.";
      } else if (score <= 14) {
        msg += "⚠ Moderate symptoms. Consider talking to a counselor or therapist.";
      } else {
        msg += "🚨 High symptoms. Please speak with a mental health professional or use a hotline.";
      }
      addBotMessage(msg);
      if (score > 14) {
        addBotMessage("📞 Crisis Hotline (US): Call or text 988. | UK: 116 123 (Samaritans)");
        addBotMessage("📖 Book: The Anxiety and Phobia Workbook by Edmund Bourne");
      } else {
        addBotMessage("🧘 Tip: Try Insight Timer or Calm for beginner-friendly meditations.");
      }
    }

    function endChat() {
      clearOptions();
      addBotMessage("No worries. I'm here anytime you want to check in.");
    }

    function addBotMessage(msg) {
      const div = document.createElement("div");
      div.className = "bot";
      div.textContent = "🤖 " + msg;
      document.getElementById("chat").appendChild(div);
      chatLog.push("Bot: " + msg);
      scrollToBottom();
    }

    function addUserMessage(val) {
      const div = document.createElement("div");
      div.className = "user";
      div.textContent = You: ${val};
      document.getElementById("chat").appendChild(div);
      chatLog.push("You: " + val);
      scrollToBottom();
    }

    function clearOptions() {
      const opts = document.querySelectorAll(".options");
      opts.forEach(el => el.remove());
    }

    function scrollToBottom() {
      const chat = document.getElementById("chat");
      chat.scrollTop = chat.scrollHeight;
    }

    function downloadTranscript() {
      const blob = new Blob([chatLog.join("\n")], { type: "text/plain" });
      const url = URL.createObjectURL(blob);
      const a = document.createElement("a");
      a.href = url;
      a.download = "Wellness_Chat_Transcript.txt";
      document.body.appendChild(a);
      a.click();
      document.body.removeChild(a);
      URL.revokeObjectURL(url);
    }
  </script>
</body>
</html>
