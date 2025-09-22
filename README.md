# lgs-<!doctype html>
<html lang="tr">
<head>
<meta charset="utf-8" />
<meta name="viewport" content="width=device-width,initial-scale=1" />
<title>Defne Yılmaz | LGS Günlüğüm</title>
<style>
:root{
  --bg:#f6f7fb; --card:#ffffff; --muted:#444; --accent:#6c63ff; --glass: rgba(255,255,255,0.6); --radius:12px;
  --good:#4CAF50; --medium:#FFC107; --bad:#F44336;
}
[data-theme="dark"]{
  --bg:#0b0b0f; --card:#1a1d23; --muted:#c9d1d9; --accent:#8b84ff; --glass: rgba(255,255,255,0.05);
}
*{box-sizing:border-box}
body{margin:0;font-family:Arial,sans-serif;background:var(--bg);color:var(--muted);}
.wrap{max-width:900px;margin:20px auto;padding:20px;}
header{display:flex;justify-content:space-between;align-items:center;margin-bottom:20px}
.brand{display:flex;align-items:center;gap:12px}
.avatar{width:60px;height:60px;border-radius:50%;background:var(--accent);color:white;display:flex;align-items:center;justify-content:center;font-size:22px;font-weight:bold;}
h1{margin:0;font-size:20px}
.card{background:var(--card);padding:16px;border-radius:var(--radius);margin-bottom:16px;box-shadow:0 3px 8px rgba(0,0,0,0.1)}
h2{margin-top:0;color:var(--accent)}
.project{background:var(--glass);padding:10px;border-radius:8px;margin-top:10px;position:relative}
.delete-btn{position:absolute;top:6px;right:6px;background:red;color:white;border:none;border-radius:4px;padding:2px 6px;cursor:pointer;font-size:12px;}
.score-tag{display:inline-block;padding:2px 6px;border-radius:6px;font-size:12px;font-weight:bold;margin-left:6px;}
canvas{background:#fff;border-radius:8px;margin-top:16px;max-width:100%;height:auto;}
table{width:100%;border-collapse:collapse;margin-top:16px;}
th,td{padding:8px;text-align:center;border:1px solid #ccc;}
th{background:var(--accent);color:white;}
.bar{height:16px;border-radius:6px;}
footer{text-align:center;font-size:12px;margin-top:20px;color:var(--muted)}
.theme-toggle{cursor:pointer;padding:6px 10px;border:1px solid var(--accent);border-radius:8px;background:none;color:var(--accent)}
form{display:flex;flex-direction:column;gap:10px;margin-top:10px}
input,textarea,select{padding:8px;border:1px solid #ccc;border-radius:6px;font-family:inherit;font-size:14px}
button{padding:8px;border:none;border-radius:6px;background:var(--accent);color:white;font-weight:bold;cursor:pointer}
</style>
</head>
<body>
<div class="wrap">
<header>
  <div class="brand">
    <div class="avatar">D</div>
    <div>
      <h1>Defne Yılmaz</h1>
      <p>8. Sınıf Öğrencisi • LGS Yolculuğu</p>
    </div>
  </div>
  <button class="theme-toggle" id="themeBtn">🌙/☀️</button>
</header>

<article class="card" id="about">
  <h2>Hakkımda</h2>
  <p>Benim adım <strong>Defne Yılmaz</strong>. 8. sınıf öğrencisiyim ve şu an <strong>LGS’ye hazırlanıyorum</strong>.
  Bu web sayfasında LGS sürecimi aktaracak ve her gün neler yaptığımı yazacağım.</p>
</article>

<article class="card" id="lgs-diary">
<h2>LGS Günlüğüm</h2>
<p>Her gün çalışmalarımla ilgili kısa notlarımı buraya ekleyeceğim:</p>

<div id="entries"></div>

<h3>Yeni Günlük Ekle</h3>
<form id="diaryForm">
  <input type="date" id="dateInput" required />
  <textarea id="textInput" rows="3" placeholder="Bugün neler yaptığını yaz..." required></textarea>
  <label>Başarı/Puan Durumu:
    <select id="scoreInput">
      <option value="">Seçiniz</option>
      <option value="good">İyi</option>
      <option value="medium">Orta</option>
      <option value="bad">Geliştir</option>
    </select>
  </label>
  <button type="submit">Ekle</button>
</form>
<p style="font-size:12px;color:gray;">Not: Yazılar tarayıcına kaydedilir ve tarihe göre sıralanır. Silmek için her yazının sağ üstündeki kırmızı butonu kullanabilirsin.</p>

<h3>Haftalık Özet</h3>
<canvas id="weeklyChart" width="800" height="300"></canvas>

<h3>Aylık İstatistikler</h3>
<table id="monthlyTable">
<thead>
<tr><th>Ay</th><th>Toplam Gün</th><th>İyi</th><th>Orta</th><th>Geliştir</th><th>İyi %</th></tr>
</thead>
<tbody></tbody>
</table>

</article>

<footer>
<p>© 2025 Defne Yılmaz — LGS Günlüğüm</p>
</footer>
</div>

<script>
// Tema
const btn=document.getElementById("themeBtn");
btn.addEventListener("click",()=>{
  if(document.documentElement.getAttribute("data-theme")==="dark"){
    document.documentElement.removeAttribute("data-theme");
  } else {
    document.documentElement.setAttribute("data-theme","dark");
  }
});

// Günlükler
const form=document.getElementById("diaryForm");
const entries=document.getElementById("entries");
const monthlyTableBody=document.querySelector("#monthlyTable tbody");

function loadEntries(){
  const saved=localStorage.getItem("lgsDiary");
  entries.innerHTML="";
  if(saved){
    let arr=JSON.parse(saved);
    arr.sort((a,b)=> new Date(b.date)-new Date(a.date));
    arr.forEach((e,index)=>{
      const div=document.createElement("div");
      div.className="project";
      let scoreTag="";
      if(e.score){
        const color=e.score==="good"? "var(--good)" : e.score==="medium"? "var(--medium)" : "var(--bad)";
        const text=e.score==="good"? "İyi" : e.score==="medium"? "Orta" : "Geliştir";
        scoreTag=`<span class="score-tag" style="background:${color};color:white">${text}</span>`;
      }
      div.innerHTML=`<strong>${e.date}</strong>${scoreTag}<p>${e.text}</p>`;
      const delBtn=document.createElement("button");
      delBtn.textContent="Sil";
      delBtn.className="delete-btn";
      delBtn.addEventListener("click",()=>{
        arr.splice(index,1);
        localStorage.setItem("lgsDiary",JSON.stringify(arr));
        loadEntries();
      });
      div.appendChild(delBtn);
      entries.appendChild(div);
    });
    drawWeeklyChart(arr);
    drawMonthlyTable(arr);
  } else {
    drawWeeklyChart([]);
    drawMonthlyTable([]);
  }
}

form.addEventListener("submit",function(e){
  e.preventDefault();
  const date=document.getElementById("dateInput").value;
  const text=document.getElementById("textInput").value;
  const score=document.getElementById("scoreInput").value;

  if(date && text){
    const saved=localStorage.getItem("lgsDiary");
    const arr=saved?JSON.parse(saved):[];
    arr.push({date,text,score});
    localStorage.setItem("lgsDiary",JSON.stringify(arr));
    loadEntries();
    form.reset();
  }
});

loadEntries();

// Haftalık grafik
function drawWeeklyChart(entriesArr){
  const canvas=document.getElementById("weeklyChart");
  const ctx=canvas.getContext("2d");
  ctx.clearRect(0,0,canvas.width,canvas.height);

  const weeks={};
  entriesArr.forEach(e=>{
    const d=new Date(e.date);
    const year=d.getFullYear();
    const week=Math.ceil((((d-new Date(year,0,1))/86400000)+1)/7);
    const key=year+"-W"+week;
    if(!weeks[key]) weeks[key]={good:0,medium:0,bad:0};
    if(e.score) weeks[key][e.score]++;
  });

  const keys=Object.keys(weeks).sort();
  const barWidth=50;
  const gap=20;
  keys.forEach((k,i)=>{
    const startX=i*(barWidth+gap)+50;
    let yBase=canvas.height-20;
    ["bad","medium","good"].forEach(score=>{
      const val=weeks[k][score];
      const height=val*20;
      if(height>0){
        ctx.fillStyle=score==="good"? "var(--good)": score==="medium"? "var(--medium)": "var(--bad)";
        ctx.fillRect(startX, yBase-height, barWidth, height);
        yBase-=height;
      }
    });
    ctx.fillStyle="#000";
    ctx.font="12px Arial";
    ctx.fillText(k,startX,canvas.height-5);
  });

  ctx.strokeStyle="#000";
  ctx.beginPath();
  ctx.moveTo(40,0);
  ctx.lineTo(40,canvas.height-20);
  ctx.stroke();
}

// Aylık tablo
function drawMonthlyTable(entriesArr){
  const monthly={};
  entriesArr.forEach(e=>{
    const d=new Date(e.date);
    const key=d.toLocaleString('tr-TR',{month:'long',year:'numeric'});
    if(!monthly[key]) monthly[key]={good:0,medium:0,bad:0};
    if(e.score) monthly[key][e.score]++;
  });

  monthlyTableBody.innerHTML="";
  Object.keys(monthly).sort().forEach(key=>{
    const data=monthly[key];
    const total=data.good+data.medium+data.bad;
    const goodPercent=total?Math.round(data.good/total*100):0;
    const barHTML=`<div class="bar" style="width:${goodPercent}%;background:var(--good)"></div>`;
    const row=document.createElement("tr");
    row.innerHTML=`<td>${key}</td><td>${total}</td><td>${data.good
