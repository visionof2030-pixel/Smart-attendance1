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
      margin: 30px 0;
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
      margin-top: 10px;
      border-radius: 8px;
      border: none;
      cursor: pointer;
    }
    .start {
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
  <button class="start" onclick="initMicrophone()">ابدأ التحضير</button>
</div>

<script>
  const students = ["أحمد محمد", "سارة علي", "خالد عبدالله"];
  let index = 0;
  let countdown;
  let recognition;
  let stream;

  async function initMicrophone() {
    try {
      stream = await navigator.mediaDevices.getUserMedia({ audio: true });
      startAttendance();
    } catch (e) {
      alert("يجب السماح بالوصول إلى المايكروفون");
    }
  }

  function speak(text) {
    const u = new SpeechSynthesisUtterance(text);
    u.lang = "ar-SA";
    u.rate = 0.9;
    speechSynthesis.speak(u);
  }

  function startAttendance() {
    index = 0;
    nextStudent();
  }

  function nextStudent() {
    if (index >= students.length) {
      document.getElementById("status").innerText = "انتهى التحضير";
      document.getElementById("timer").innerText = "";
      return;
    }

    document.getElementById("studentName").innerText = students[index];
    document.getElementById("status").innerText = "بانتظار الرد...";
    speak(students[index]);

    setTimeout(() => {
      startListening();
      startTimer(5);
    }, 800);
  }

  function startTimer(seconds) {
    let remaining = seconds;
    document.getElementById("timer").innerText = remaining + " ثواني";
    countdown = setInterval(() => {
      remaining--;
      document.getElementById("timer").innerText = remaining + " ثواني";
      if (remaining <= 0) {
        clearInterval(countdown);
        stopListening();
        markAbsent();
      }
    }, 1000);
  }

  function startListening() {
    recognition = new (window.SpeechRecognition || window.webkitSpeechRecognition)();
    recognition.lang = "ar-SA";
    recognition.continuous = true;
    recognition.interimResults = false;

    recognition.onresult = function(event) {
      for (let i = event.resultIndex; i < event.results.length; i++) {
        const text = event.results[i][0].transcript.trim().toLowerCase();
        if (text.includes("حاضر") || text.includes("present")) {
          clearInterval(countdown);
          stopListening();
          markPresent();
          return;
        }
        if (text.includes("غائب")) {
          clearInterval(countdown);
          stopListening();
          markAbsent();
          return;
        }
      }
    };

    recognition.onerror = function() {
      stopListening();
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
    moveNext();
  }

  function markAbsent() {
    document.getElementById("status").innerText = "غائب";
    document.getElementById("status").className = "status absent";
    save("absent");
    moveNext();
  }

  function moveNext() {
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
