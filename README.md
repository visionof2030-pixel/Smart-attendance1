<!DOCTYPE html>
<html lang="ar" dir="rtl">
<head>
<meta charset="UTF-8">
<title>Voice Attendance AI</title>
<meta name="viewport" content="width=device-width, initial-scale=1.0">

<style>
body{font-family:Arial;background:#f4f6f8;padding:20px}
h1{text-align:center}
.container{max-width:600px;margin:auto;background:#fff;border-radius:10px;padding:20px;box-shadow:0 2px 10px rgba(0,0,0,.1)}
.student-name{font-size:32px;text-align:center;margin:25px 0}
.timer{text-align:center;font-size:22px}
.status{text-align:center;font-size:20px;margin-top:15px}
.present{color:green}
.absent{color:red}
button{width:100%;padding:15px;font-size:18px;margin-top:15px;border-radius:8px;border:none;background:#007bff;color:#fff}
</style>
</head>

<body>

<h1>التحضير الصوتي التفاعلي</h1>

<div class="container">
  <div id="studentName" class="student-name">—</div>
  <div id="timer" class="timer">—</div>
  <div id="status" class="status">اضغط لبدء التحضير</div>
  <button id="startBtn">ابدأ التحضير</button>
</div>

<script>
const students=["أحمد محمد","سارة علي","خالد عبدالله"];
let index=0;
let timer=null;
let recognition=null;
let audioContext=null;
let audioReady=false;

document.getElementById("startBtn").addEventListener("click", async ()=>{

  if(!audioContext){
    audioContext=new (window.AudioContext||window.webkitAudioContext)();
  }
  await audioContext.resume();
  audioReady=true;

  const u=new SpeechSynthesisUtterance("تم بدء التحضير");
  u.lang="ar-SA";
  speechSynthesis.cancel();
  speechSynthesis.speak(u);

  await navigator.mediaDevices.getUserMedia({audio:true});
  startAttendance();
});

function startAttendance(){
  index=0;
  nextStudent();
}

function nextStudent(){
  clearInterval(timer);
  stopListening();

  if(index>=students.length){
    document.getElementById("studentName").innerText="—";
    document.getElementById("status").innerText="انتهى التحضير";
    document.getElementById("timer").innerText="";
    return;
  }

  const name=students[index];
  document.getElementById("studentName").innerText=name;
  document.getElementById("status").innerText="بانتظار الرد...";
  document.getElementById("timer").innerText="5";

  speak(name);

  setTimeout(()=>{
    startListening();
    startTimer(5);
  },1500);
}

function speak(text){
  if(!audioReady)return;
  const utter=new SpeechSynthesisUtterance(text);
  utter.lang="ar-SA";
  utter.rate=0.85;
  utter.volume=1;
  speechSynthesis.cancel();
  speechSynthesis.speak(utter);
}

function startTimer(sec){
  let t=sec;
  timer=setInterval(()=>{
    t--;
    document.getElementById("timer").innerText=t;
    if(t<=0){
      clearInterval(timer);
      stopListening();
      markAbsent();
    }
  },1000);
}

function startListening(){
  const SR=window.SpeechRecognition||window.webkitSpeechRecognition;
  if(!SR){
    markAbsent();
    return;
  }

  recognition=new SR();
  recognition.lang="ar-SA";
  recognition.continuous=false;
  recognition.interimResults=false;

  recognition.onresult=e=>{
    const text=e.results[0][0].transcript.toLowerCase();
    clearInterval(timer);
    stopListening();
    if(text.includes("حاضر")||text.includes("present")){
      markPresent();
    }else{
      markAbsent();
    }
  };

  recognition.onerror=()=>{
    clearInterval(timer);
    stopListening();
    markAbsent();
  };

  recognition.start();
}

function stopListening(){
  if(recognition){
    recognition.stop();
    recognition=null;
  }
}

function markPresent(){
  document.getElementById("status").innerText="حاضر";
  document.getElementById("status").className="status present";
  save("present");
  moveNext();
}

function markAbsent(){
  document.getElementById("status").innerText="غائب";
  document.getElementById("status").className="status absent";
  save("absent");
  moveNext();
}

function moveNext(){
  setTimeout(()=>{
    index++;
    nextStudent();
  },1200);
}

function save(status){
  fetch("https://your-backend-on-render.com/attendance",{
    method:"POST",
    headers:{"Content-Type":"application/json"},
    body:JSON.stringify({
      student:students[index],
      status,
      source:"voice",
      timestamp:new Date().toISOString()
    })
  });
}
</script>

</body>
</html>