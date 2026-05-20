# AliveSafe-Guardian
Smart panic button and personal safety application with GPS tracking, emergency alerts, guardian mode, evidence capture, and PDF reporting.
<img width="405" height="793" alt="image" src="https://github.com/user-attachments/assets/c7e5157d-3630-4428-8862-2c42b006b4fa" />



<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8">
<title>AliveSafe Guardian Pro</title>
<meta name="viewport" content="width=device-width, initial-scale=1">

<link href="https://fonts.googleapis.com/css2?family=Poppins:wght@300;400;400;600&display=swap" rel="stylesheet">

<script src="https://cdnjs.cloudflare.com/ajax/libs/jspdf/2.5.1/jspdf.umd.min.js"></script>

<style>

/* RESET */
*{margin:0;padding:0;box-sizing:border-box;font-family:Poppins,sans-serif}

/* BACKGROUND */
body{
height:100vh;
background:linear-gradient(135deg,#02040f,#0a1a2f,#02040f);
display:flex;
justify-content:center;
align-items:center;
}

/* GLASS CARD */
.app{
width:390px;
max-width:95%;
padding:25px;
border-radius:25px;
background:rgba(255,255,255,0.05);
backdrop-filter:blur(20px);
border:1px solid rgba(255,255,255,0.1);
box-shadow:0 0 40px rgba(0,255,255,0.2);
color:white;
text-align:center;
overflow-y:auto;
max-height:95vh;
}

/* HEADERS */
h1{color:#00f7ff;margin-bottom:5px}
.tag{color:#aaa;font-size:13px;margin-bottom:15px}

/* INPUT */
input{
width:100%;
padding:11px;
margin:6px 0;
border-radius:12px;
border:none;
background:rgba(255,255,255,0.08);
color:white;
outline:none;
}

input::placeholder{color:#888}

/* BUTTON */
button{
width:100%;
padding:13px;
margin:7px 0;
border-radius:14px;
border:none;
font-weight:600;
cursor:pointer;
position:relative;
overflow:hidden;
transition:.3s;
}

button::before{
content:"";
position:absolute;
top:0;left:-100%;
width:100%;height:100%;
background:linear-gradient(120deg,transparent,rgba(255,255,255,.3),transparent);
transition:.6s;
}
button:hover::before{left:100%}

/* TYPES */
.save{background:#00f7ff;color:#000}
.safe{background:#2ecc71}
.guard{background:#f39c12}
.panic{background:#ff0033;animation:pulse 2s infinite}
.report{background:#9b59b6}

@keyframes pulse{
0%{box-shadow:0 0 0 0 rgba(255,0,50,.6)}
70%{box-shadow:0 0 0 18px rgba(255,0,50,0)}
100%{box-shadow:0 0 0 0 rgba(255,0,50,0)}
}

/* STATUS */
.status{
margin-top:10px;
padding:10px;
border-radius:12px;
background:rgba(255,255,255,0.08);
font-size:13px;
color:#ccc;
}

/* IMAGE */
#lastPhoto{
width:100%;
border-radius:12px;
margin-top:10px;
display:none;
border:2px solid #00f7ff;
}

hr{
border:0;
height:1px;
background:rgba(255,255,255,.1);
margin:15px 0;
}

</style>
</head>

<body>

<div class="app">

<h1>AliveSafe</h1>
<div class="tag">Guardian Protection System</div>

<!-- PROFILE -->
<input id="name" placeholder="Full Name">
<input id="c1" placeholder="WhatsApp Contact 1">
<input id="c2" placeholder="WhatsApp Contact 2">
<input id="pin" placeholder="Secret PIN">

<button class="save" onclick="saveProfile()">Save Profile</button>

<hr>

<!-- ACTIONS -->
<button class="safe" onclick="checkIn()">✅ I'm Safe</button>
<button class="guard" onclick="guardian()">🟠 Guardian Mode</button>
<button class="panic" onclick="panic()">🚨 Emergency</button>

<hr>

<!-- REPORTS -->
<button class="report" onclick="exportExcel()">📊 Export Excel</button>
<button class="report" onclick="exportPDF()">📄 Export PDF</button>

<hr>

<img id="lastPhoto">

<div class="status" id="status">Status: Protected</div>

</div>


<video id="video" autoplay playsinline hidden></video>
<canvas id="canvas" hidden></canvas>


<script>

let records = JSON.parse(localStorage.getItem("records")) || [];
let watchID=null;

/* SAVE */
function saveProfile(){
localStorage.setItem("name",name.value);
localStorage.setItem("c1",c1.value);
localStorage.setItem("c2",c2.value);
localStorage.setItem("pin",pin.value);
alert("Profile Saved ✅");
}

/* CHECK IN */
function checkIn(){
capture();
status.innerText="Status: Safe ✅";
}

/* GUARDIAN */
function guardian(){
status.innerText="Guardian Mode Active 🟠";
track();
}

/* PANIC */
function panic(){
capture(true);
sendAlert();
status.innerText="🚨 Emergency Activated";
}

/* CAMERA+GPS */
function capture(emergency=false){

navigator.mediaDevices.getUserMedia({video:true})
.then(stream=>{

video.srcObject=stream;

setTimeout(()=>{

canvas.width=video.videoWidth;
canvas.height=video.videoHeight;

canvas.getContext("2d")
.drawImage(video,0,0,canvas.width,canvas.height);

const photo=canvas.toDataURL("image/png");

stream.getTracks().forEach(t=>t.stop());

navigator.geolocation.getCurrentPosition(pos=>{

const lat=pos.coords.latitude.toFixed(6);
const lon=pos.coords.longitude.toFixed(6);

const rec={
name:localStorage.getItem("name"),
time:new Date().toLocaleString(),
lat,lon,
map:`https://maps.google.com/?q=${lat},${lon}`,
photo,
emergency
};

records.push(rec);
localStorage.setItem("records",JSON.stringify(records));

lastPhoto.src=photo;
lastPhoto.style.display="block";

});

},1500);

});
}

/* TRACK */
function track(){

if(!navigator.geolocation) return;

watchID=navigator.geolocation.watchPosition(p=>{

localStorage.setItem("lat",p.coords.latitude);
localStorage.setItem("lon",p.coords.longitude);

});
}

/* ALERT */
function sendAlert(){

const n=localStorage.getItem("name")||"User";
const lat=localStorage.getItem("lat");
const lon=localStorage.getItem("lon");

const msg=encodeURIComponent(
`🚨 EMERGENCY ALERT

${n} may be in danger.

Location:
${lat}, ${lon}

Respond immediately.`);

[c1.value,c2.value].forEach(x=>{

if(x){
window.open(`https://wa.me/${x}?text=${msg}`,"_blank");
}

});
}

/* EXCEL */
function exportExcel(){

let csv="Name,Time,Latitude,Longitude,Map,Emergency,Photo\n";

records.forEach(r=>{

csv+=`"${r.name}","${r.time}","${r.lat}","${r.lon}","${r.map}","${r.emergency}","${r.photo}"\n`;

});

const blob=new Blob([csv],{type:"text/csv"});

const a=document.createElement("a");
a.href=URL.createObjectURL(blob);
a.download="AliveSafe_Report.csv";
a.click();
}

/* PDF */
function exportPDF(){

const {jsPDF}=window.jspdf;
const doc=new jsPDF();

let y=15;

doc.setFontSize(18);
doc.text("AliveSafe Emergency Report",10,y); y+=10;

records.forEach((r,i)=>{

if(y>260){doc.addPage();y=15}

doc.setFontSize(13);
doc.text(`Record ${i+1}`,10,y); y+=6;

doc.setFontSize(11);
doc.text(`Name: ${r.name}`,10,y); y+=5;
doc.text(`Time: ${r.time}`,10,y); y+=5;
doc.text(`Location: ${r.lat}, ${r.lon}`,10,y); y+=5;
doc.text(`Emergency: ${r.emergency}`,10,y); y+=5;

if(r.photo){

doc.addImage(r.photo,"PNG",10,y,80,60);
y+=65;

}

y+=5;

});

doc.save("AliveSafe_Report.pdf");
}

</script>

</body>
</html>
