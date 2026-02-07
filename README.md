<!doctype html>
<html lang="de">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>5M Leitstelle</title>

<style>
:root{
  --bg:#0b1220;
  --text:#ffffff;
  --border:rgba(255,255,255,.12);
  --table:#0f172a;
  --cell:#020617;
}
body{
  margin:0;
  font-family:ui-sans-serif,system-ui,sans-serif;
  background:
    radial-gradient(1200px 700px at 20% 10%, rgba(96,165,250,.22), transparent 55%),
    radial-gradient(900px 600px at 85% 35%, rgba(94,234,212,.20), transparent 50%),
    radial-gradient(900px 600px at 55% 95%, rgba(251,113,133,.14), transparent 45%),
    var(--bg);
  color:var(--text);
  min-height:100vh;
  display:flex;
  flex-direction:column;
}
header{
  position:sticky;
  top:0;
  backdrop-filter:blur(14px);
  background:rgba(11,18,32,.6);
  border-bottom:1px solid var(--border);
  padding:18px;
  display:flex;
  align-items:center;
  gap:14px;
}
.logo{
  width:44px;height:44px;border-radius:16px;
  background:linear-gradient(135deg,#5eead4,#60a5fa);
  display:grid;place-items:center;
  color:#020617;font-weight:900;
}
main{
  flex:1;
  display:flex;
  flex-direction:column;
  align-items:center;
  padding:25px;
}
input{
  padding:12px;
  border-radius:14px;
  border:1px solid var(--border);
  background:#020617;
  color:white;
  outline:none;
  width:220px;
  margin-bottom:12px;
}
button{
  padding:12px 14px;
  border-radius:16px;
  border:0;
  cursor:pointer;
  font-weight:700;
  font-size:14px;
  color:#020617;
  background:linear-gradient(135deg,#5eead4,#60a5fa);
  margin:6px;
  width:240px;
}
table{
  border-collapse:collapse;
  margin-top:20px;
  width:99%;
  background:var(--table);
}
th,td{
  border:1px solid var(--border);
  padding:6px;
  text-align:center;
}
th{
  background:#020617;
  font-size:13px;
}
input.cell{
  width:100%;
  background:transparent;
  border:none;
  color:white;
  outline:none;
  padding:6px;
  font-size:13px;
}
.status{
  font-weight:700;
}
</style>
</head>

<body>

<header>
  <div class="logo">5M</div>
  <h2>Leitstelle</h2>
</header>

<main>

<div id="loginBox">
  <input id="pw" type="password" placeholder="Passwort">
  <button onclick="login()">Einloggen</button>
</div>

<div id="app" style="display:none;">
  <h3 id="jobTitle"></h3>

  <button onclick="addRow()">Zeile hinzufügen</button>
  <button onclick="clearTable()">Tabelle löschen</button>

  <table id="table"></table>
</div>

</main>

<script>
let job="";
let data=[];

const headers=[
"CALL ID","CODE","PRIORITY","ADDRESS","UNIT",
"STATUS","INFO","TIME","CHANNEL","NOTES"
];

function login(){
  const pw=document.getElementById("pw").value.trim();

  if(pw==="01") job="pd";
  else if(pw==="02") job="fd";
  else if(pw==="03") job="md";
  else{ alert("Falsches Passwort"); return; }

  document.getElementById("loginBox").style.display="none";
  document.getElementById("app").style.display="block";
  document.getElementById("jobTitle").innerText=job.toUpperCase()+" DISPATCH";

  loadData();
  buildTable();
}

function loadData(){
  data=JSON.parse(localStorage.getItem("leitstelle_"+job)||"[]");
  if(data.length===0){
    for(let i=0;i<10;i++){
      data.push(new Array(10).fill(""));
    }
  }
}

function save(){
  localStorage.setItem("leitstelle_"+job,JSON.stringify(data));
}

function statusColor(val,el){
  val=val.toUpperCase();

  el.style.background="";
  if(val==="AVAILABLE") el.style.background="#16a34a";
  if(val==="EN ROUTE") el.style.background="#2563eb";
  if(val==="ON SCENE") el.style.background="#ea580c";
  if(val==="BUSY") el.style.background="#dc2626";
  if(val==="TRANSPORT") el.style.background="#9333ea";
  if(val==="OFF DUTY") el.style.background="#6b7280";
}

function buildTable(){
  const table=document.getElementById("table");
  table.innerHTML="";

  const head=document.createElement("tr");
  headers.forEach(h=>{
    const th=document.createElement("th");
    th.innerText=h;
    head.appendChild(th);
  });
  table.appendChild(head);

  data.forEach((row,r)=>{
    const tr=document.createElement("tr");

    for(let c=0;c<10;c++){
      const td=document.createElement("td");
      const input=document.createElement("input");
      input.className="cell";
      input.value=row[c]||"";

      input.addEventListener("input",()=>{
        data[r][c]=input.value;
        save();
        if(c===5) statusColor(input.value,input);
      });

      if(c===5) statusColor(input.value,input);

      td.appendChild(input);
      tr.appendChild(td);
    }
    table.appendChild(tr);
  });
}

function addRow(){
  data.push(new Array(10).fill(""));
  save();
  buildTable();
}

function clearTable(){
  if(confirm("Alles löschen?")){
    localStorage.removeItem("leitstelle_"+job);
    data=[];
    loadData();
    buildTable();
  }
}
</script>

</body>
</html>

