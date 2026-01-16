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
  <button onclick="init()">ابدأ التحضير</button>
</div>

<script>
  const students = ["أحمد محمد", "سارة علي", "خالد عبدالله"];
  let currentIndex = 0;
  let countdownTimer = null;
  let recognition = null;
  let listening = false;

  function init() {
    if (!navigator.mediaDevices || !navigator.mediaDevices.getUserMedia) {
      alert("المتصفح لا يدعم المايكروفون");
      return;
    }
    navigator.mediaDevices.getUserMedia({ audio: true })
      .then(() => startAttendance())
      .catch(() => alert("يجب السماح بالوصول إلى المايكروفون"));
  }

  function startAttendance() {
    currentIndex = 0;
    nextStudent();
  }

  function nextStudent() {
    clearTimers();
    stopListening();

    if (currentIndex >= students.length) {
      document.getElementById("studentName").innerText = "—";
      document.getElementById("status").innerText = "انتهى التحضير";
      document.getElementById("timer").innerText = "";
      return;
    }

    const name = students[currentIndex];
    document.getElementById("studentName").innerText = name;
    document.getElementById("status").innerText = "بانتظار الرد...";
    document.getElementById("timer").innerText = "5";

    speak(name);

    setTimeout(() => {
      startListening();
      startCountdown(5);
    }, 1000);
  }

  function startCountdown(seconds) {
    let remaining = seconds;
    countdownTimer = setInterval(() => {
      remaining--;
      document.getElementById("timer").innerText = remaining;
      if (remaining <= 0) {
        clearTimers();
        stopListening();
        markAbsent();
      }
    }, 1000);
  }

  function speak(text) {
    const utterance = new SpeechSynthesisUtterance(text);
    utterance.lang = "ar-SA";
    utterance.rate = 0.9;
    speechSynthesis.cancel();
    speechSynthesis.speak(utterance);
  }

  function startListening() {
    const SpeechRecognition = window.SpeechRecognition || window.webkitSpeechRecognition;
    if (!SpeechRecognition) {
      alert("التعرف الصوتي غير مدعوم في هذا المتصفح");
      return;
    }

    recognition = new SpeechRecognition();
    recognition.lang = "ar-SA";
    recognition.continuous = false;
    recognition.interimResults = false;
    listening = true;

    recognition.onresult = function(event) {
      const transcript = event.results[0][0].transcript.trim().toLowerCase();

      clearTimers();
      stopListening();

      if (transcript.includes("حاضر") || transcript.includes("present")) {
        markPresent();
      } else if (transcript.includes("غائب")) {
        markAbsent();
      } else {
        markAbsent();
      }
    };

    recognition.onerror = function() {
      clearTimers();
      stopListening();
      markAbsent();
    };

    recognition.start();
  }

  function stopListening() {
    if (recognition && listening) {
      recognition.stop();
      recognition = null;
      listening = false;
    }
  }

  function clearTimers() {
    if (countdownTimer) {
      clearInterval(countdownTimer);
      countdownTimer = null;
    }
  }

  function markPresent() {
    document.getElementById("status").innerText = "حاضر";
    document.getElementById("status").className = "status present";
    saveAttendance("present");
    moveNext();
  }

  function markAbsent() {
    document.getElementById("status").innerText = "غائب";
    document.getElementById("status").className = "status absent";
    saveAttendance("absent");
    moveNext();
  }

  function moveNext() {
    setTimeout(() => {
      currentIndex++;
      nextStudent();
    }, 1200);
  }

  function saveAttendance(status) {
    fetch("https://your-backend-on-render.com/attendance", {
      method: "POST",
      headers: { "Content-Type": "application/json" },
      body: JSON.stringify({
        student: students[currentIndex],
        status: status,
        source: "voice",
        timestamp: new Date().toISOString()
      })
    });
  }
</script>

</body>
</html>