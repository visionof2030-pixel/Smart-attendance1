<!DOCTYPE html>
<html lang="ar" dir="rtl">
<head>
  <meta charset="UTF-8">
  <title>Voice Attendance AI</title>
  <meta name="viewport" content="width=device-width, initial-scale=1.0">

  <style>
    body {
      font-family: Arial, sans-serif;
      background: #f4f6f8;
      padding: 20px;
    }
    h1 {
      text-align: center;
    }
    .container {
      max-width: 600px;
      margin: auto;
      background: #ffffff;
      border-radius: 10px;
      padding: 20px;
      box-shadow: 0 2px 10px rgba(0,0,0,0.1);
    }
    .student-name {
      font-size: 32px;
      text-align: center;
      margin: 25px 0;
    }
    .timer {
      text-align: center;
      font-size: 22px;
    }
    .status {
      text-align: center;
      font-size: 20px;
      margin-top: 15px;
    }
    .present { color: green; }
    .absent { color: red; }
    button {
      width: 100%;
      padding: 15px;
      font-size: 18px;
      margin-top: 15px;
      border-radius: 8px;
      border: none;
      cursor: pointer;
      background: #007bff;
      color: #fff;
    }
  </style>
</head>

<body>

<h1>التحضير الصوتي التفاعلي</h1>

<div class="container">
  <div id="studentName" class="student-name">—</div>
  <div id="timer" class="timer">—</div>
  <div id="status" class="status">اضغط لبدء التحضير</div>
  <button onclick="unlockAndStart()">ابدأ التحضير</button>
</div>

<script>
  const students = ["أحمد محمد", "سارة علي", "خالد عبدالله"];
  let index = 0;
  let countdown = null;
  let recognition = null;
  let audioUnlocked = false;

  async function unlockAndStart() {
    try {
      await navigator.mediaDevices.getUserMedia({ audio: true });

      const unlock = new SpeechSynthesisUtterance("تم بدء التحضير");
      unlock.lang = "ar-SA";
      unlock.volume = 1;
      speechSynthesis.cancel();
      speechSynthesis.speak(unlock);

      audioUnlocked = true;

      setTimeout(() => {
        startAttendance();
      }, 300);

    } catch {
      alert("يجب السماح باستخدام المايكروفون");
    }
  }

  function startAttendance() {
    index = 0;
    nextStudent();
  }

  function nextStudent() {
    clearInterval(countdown);
    stopListening();

    if (index >= students.length) {
      document.getElementById("studentName").innerText = "—";
      document.getElementById("status").innerText = "انتهى التحضير";
      document.getElementById("timer").innerText = "";
      return;
    }

    const name = students[index];
    document.getElementById("studentName").innerText = name;
    document.getElementById("status").innerText = "بانتظار الرد...";
    document.getElementById("timer").innerText = "5";

    speakName(name);

    setTimeout(() => {
      startListening();
      startTimer(5);
    }, 1200);
  }

  function speakName(name) {
    if (!audioUnlocked) return;

    speechSynthesis.cancel();
    const utter = new SpeechSynthesisUtterance(name);
    utter.lang = "ar-SA";
    utter.rate = 0.85;
    utter.volume = 1;
    speechSynthesis.speak(utter);
  }

  function startTimer(seconds) {
    let t = seconds;
    countdown = setInterval(() => {
      t--;
      document.getElementById("timer").innerText = t;
      if (t <= 0) {
        clearInterval(countdown);
        stopListening();
        markAbsent();
      }
    }, 1000);
  }

  function startListening() {
    const SR = window.SpeechRecognition || window.webkitSpeechRecognition;
    if (!SR) {
      markAbsent();
      return;
    }

    recognition = new SR();
    recognition.lang = "ar-SA";
    recognition.continuous = false;
    recognition.interimResults = false;

    recognition.onresult = (e) => {
      const text = e.results[0][0].transcript.trim().toLowerCase();
      clearInterval(countdown);
      stopListening();
      if (text.includes("حاضر") || text.includes("present")) {
        markPresent();
      } else {
        markAbsent();
      }
    };

    recognition.onerror = () => {
      clearInterval(countdown);
      stopListening();
      markAbsent();
    };

    recognition.start();
  }

  function stopListening() {
    if (recognition) {
      recognition.stop();
      recognition = null;
    }
  }

  function markPresent() {
    document.getElementById("status").innerText = "حاضر";
    document.getElementById("status").className = "status present";
    save("present");
    next();
  }

  function markAbsent() {
    document.getElementById("status").innerText = "غائب";
    document.getElementById("status").className = "status absent";
    save("absent");
    next();
  }

  function next() {
    setTimeout(() => {
      index++;
      nextStudent();
    }, 1200);
  }

  function save(status) {
    fetch("https://your-backend-on-render.com/attendance", {
      method: "POST",
      headers: { "Content-Type": "application/json" },
      body: JSON.stringify({
        student: students[index],
        status: status,
        source: "voice",
        timestamp: new Date().toISOString()
      })
    });
  }
</script>

</body>
</html>