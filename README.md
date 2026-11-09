<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>BHATT Clinic Management</title>
<style>
*{box-sizing:border-box}
body{margin:0;font-family:Arial,sans-serif;background:#f4f6fa;color:#202b3c}
button,input,select,textarea{font:inherit}
button{cursor:pointer;border:0;border-radius:8px;padding:11px 16px}
.primary{background:#2457ed;color:white}
.soft{background:#edf2ff;color:#2457ed}
.sidebar{position:fixed;left:0;top:0;bottom:0;width:245px;background:#101b2d;color:white;padding:22px 14px;overflow:auto}
.logo{font-size:23px;font-weight:bold;padding:10px;margin-bottom:25px}
.logo span{background:#2457ed;padding:8px 11px;border-radius:10px;margin-right:8px}
.nav-title{font-size:11px;color:#8290a5;padding:15px 12px 8px}
.nav{display:block;padding:13px 14px;margin:4px 0;color:#b9c5d6;border-radius:8px;cursor:pointer}
.nav:hover,.nav.active{background:#2457ed;color:white}
.main{margin-left:245px}
.topbar{height:68px;background:white;display:flex;align-items:center;justify-content:space-between;padding:0 28px;border-bottom:1px solid #e7ebf2}
.search{width:300px;padding:11px 15px;border:0;border-radius:25px;background:#f3f5f9}
main{padding:28px}
h1{font-size:26px;margin:0 0 8px}
h3{margin-top:0}
.muted{color:#8490a2}
.pagehead{display:flex;justify-content:space-between;align-items:center;margin-bottom:23px}
.stats{display:grid;grid-template-columns:repeat(4,1fr);gap:16px;margin-bottom:20px}
.card,.panel{background:white;border:1px solid #e9edf3;border-radius:13px;padding:19px;margin-bottom:18px;box-shadow:0 5px 18px #192c4908}
.stat-label{color:#758196;font-size:13px}
.stat-value{font-size:29px;font-weight:bold;margin:13px 0}
.stat-icon{float:right;padding:10px;border-radius:10px;background:#eeeaff;color:#7148db}
.dashboard-grid{display:grid;grid-template-columns:1.6fr 1fr;gap:18px}
.chart{height:200px;display:flex;align-items:end;gap:13px;padding:12px 5px;border-bottom:1px solid #ddd;background:repeating-linear-gradient(to bottom,transparent 0,transparent 48px,#edf0f5 49px)}
.bar-group{height:100%;flex:1;display:flex;align-items:end;gap:4px;position:relative}
.bar{width:13px;background:#2457ed;border-radius:5px 5px 0 0}
.bar.green{background:#35c994}
.bar-label{position:absolute;bottom:-23px;font-size:10px;color:#8490a2}
.form-grid{display:grid;grid-template-columns:repeat(4,minmax(0,1fr));gap:15px}
.field label{display:block;font-size:12px;font-weight:bold;margin-bottom:7px;color:#455267}
.field input,.field select,.field textarea{width:100%;padding:10px;border:1px solid #dce2eb;border-radius:6px;background:white}
.section-title{margin:25px 0 16px;color:#159b96;font-weight:bold;border-bottom:1px solid #dce7e9;padding-bottom:9px}
.wide{grid-column:span 2}
.actions{display:flex;justify-content:flex-end;gap:10px;margin-top:22px}
table{width:100%;border-collapse:collapse;white-space:nowrap}
th,td{text-align:left;padding:13px 10px;border-bottom:1px solid #edf0f5;font-size:13px}
th{background:#f5f7fa;color:#748198;font-size:11px}
.tablewrap{overflow:auto}
.status{background:#e3f8ed;color:#159563;padding:5px 9px;border-radius:20px;font-size:11px}
.login{min-height:100vh;display:grid;place-items:center;background:#14233b;padding:20px}
.loginbox{background:white;width:min(400px,100%);padding:32px;border-radius:16px}
.loginbox input{width:100%;padding:12px;border:1px solid #ddd;border-radius:8px;margin:8px 0 15px}
.loginbox button{width:100%}
.rxrow{display:grid;grid-template-columns:2fr 1fr 1fr auto;gap:10px;margin:10px 0}
@media(max-width:1000px){.stats{grid-template-columns:repeat(2,1fr)}.form-grid{grid-template-columns:repeat(2,1fr)}.dashboard-grid{grid-template-columns:1fr}}
@media(max-width:650px){.sidebar{width:65px;padding:10px 5px}.logo{font-size:0;padding:10px 4px}.logo span{font-size:20px}.nav-title,.nav-text{display:none}.nav{text-align:center;padding:14px 5px}.main{margin-left:65px}.topbar{padding:0 12px}main{padding:13px}.stats{gap:8px}.card{padding:12px}.stat-value{font-size:22px}.form-grid{grid-template-columns:1fr}.wide{grid-column:span 1}.pagehead{align-items:flex-start;gap:10px;flex-direction:column}.rxrow{grid-template-columns:1fr 1fr}}
</style>
</head>
<body>
<div id="app"></div>
<script>
const DB={
patients:"Bhatt_patients",
appointments:"Bhatt_appointments",
prescriptions:"Bhatt_prescriptions",
bills:"Bhatt_bills"
};

const users={
admin:{password:"1234",role:"Admin",name:"Clinic Admin"},
reception:{password:"1234",role:"Receptionist",name:"Receptionist"},
doctor:{password:"1234",role:"Doctor",name:"Dr. John Doe"}
};

let user=null;
let page="dashboard";

const get=k=>JSON.parse(localStorage.getItem(DB[k])||"[]");
const save=(k,v)=>localStorage.setItem(DB[k],JSON.stringify(v));

function safe(v){
return String(v??"").replace(/[&<>"']/g,c=>({
"&":"&amp;","<":"&lt;",">":"&gt;",'"':"&quot;","'":"&#39;"
}[c]));
}

function login(){
document.getElementById("app").innerHTML=`
<div class="login">
<div class="loginbox">
<h2>✚ BHATT's CLINIC</h2>
<h1>Welcome Back</h1>
<p class="muted">Clinic Management Portal</p>
<form id="loginform">
<label>Username</label>
<input name="username" required placeholder="Enter username">
<label>Password</label>
<input name="password" type="password" required placeholder="Enter password">
<button class="primary">Login</button>
<p id="error" style="color:red"></p>
</form>
<hr>
<p><b>Demo Login</b></p>
<p>Admin: admin / 1234</p>
<p>Reception: reception / 1234</p>
<p>Doctor: doctor / 1234</p>
</div>
</div>`;

document.getElementById("loginform").onsubmit=e=>{
e.preventDefault();
let f=new FormData(e.target);
let u=users[f.get("username")];
if(!u||u.password!==f.get("password")){
document.getElementById("error").textContent="Invalid login details";
return;
}
user={...u,username:f.get("username")};
page="dashboard";
render();
};
}

function nav(id,icon,label){
return `<div class="nav ${page===id?"active":""}" onclick="go('${id}')">
${icon} <span class="nav-text">${label}</span>
</div>`;
}

function shell(content){
document.getElementById("app").innerHTML=`
<div class="sidebar">
<div class="logo"><span>✚</span> MedAxis</div>
<div class="nav-title">CLINIC MANAGEMENT</div>
${nav("dashboard","▦","Dashboard")}
${nav("patients","♙","Patient Registration")}
${nav("appointments","▣","Appointments")}
${nav("prescriptions","✎","Digital Prescription")}
${nav("billing","₹","Payments & Billing")}
<div class="nav-title">WORKSPACE</div>
${nav("reports","▤","Reports")}
<div style="margin-top:60px;color:#8290a5;font-size:12px;padding:10px">
${user.name}<br><br>
<button class="soft" onclick="logout()">Logout</button>
</div>
</div>
<div class="main">
<header class="topbar">
<input class="search" placeholder="⌕ Search...">
<span>${user.name}　●</span>
</header>
<main>${content}</main>
</div>`;
}

function go(p){page=p;render()}
function logout(){user=null;login()}

function head(title,subtitle,button=""){
return `<div class="pagehead">
<div><h1>${title}</h1><span class="muted">${subtitle}</span></div>
${button}
</div>`;
}

function stat(label,value,icon){
return `<div class="card">
<span class="stat-icon">${icon}</span>
<div class="stat-label">${label}</div>
<div class="stat-value">${value}</div>
<span class="muted">Clinic overview</span>
</div>`;
}

function table(headers,rows){
return `<div class="tablewrap"><table>
<thead><tr>${headers.map(h=>`<th>${h}</th>`).join("")}</tr></thead>
<tbody>${rows.length?rows.join(""):`<tr><td colspan="${headers.length}">No records found</td></tr>`}</tbody>
</table></div>`;
}

function dashboard(){
let p=get("patients"),a=get("appointments"),r=get("prescriptions"),b=get("bills");
let revenue=b.reduce((s,x)=>s+Number(x.amount||0),0);

return head("Dashboard","Good morning, "+user.name+". Let's make today efficient.",
`<button class="primary" onclick="go('patients')">+ New Patient</button>`)
+`<div class="stats">
${stat("Total Patients",p.length,"♙")}
${stat("Appointments",a.length,"▣")}
${stat("Prescriptions",r.length,"✎")}
${stat("Total Revenue","₹"+revenue,"₹")}
</div>
<div class="dashboard-grid">
<section class="panel">
<h3>Patient Overview</h3>
<p class="muted">Monthly clinic activity</p>
<div class="chart">
${[40,65,50,80,55,90,60,100,75,85,60,95].map((v,i)=>`
<div class="bar-group">
<div class="bar" style="height:${v}%"></div>
<div class="bar green" style="height:${v*.6}%"></div>
<span class="bar-label">${["Jan","Feb","Mar","Apr","May","Jun","Jul","Aug","Sep","Oct","Nov","Dec"][i]}</span>
</div>`).join("")}
</div>
</section>
<section class="panel">
<h3>Quick Actions</h3>
<p><button class="soft" onclick="go('patients')">+ Register Patient</button></p>
<p><button class="soft" onclick="go('appointments')">+ Book Appointment</button></p>
<p><button class="soft" onclick="go('prescriptions')">+ Write Prescription</button></p>
<p><button class="soft" onclick="go('billing')">+ Create Bill</button></p>
</section>
</div>
<section class="panel">
<h3>Recent Patients</h3>
${table(["Patient ID","Name","Phone","Age","Gender"],
p.slice().reverse().slice(0,8).map(x=>`
<tr><td>${safe(x.id)}</td><td>${safe(x.name)}</td>
<td>${safe(x.phone)}</td><td>${safe(x.age)}</td><td>${safe(x.gender)}</td></tr>`))}
</section>
<section class="panel">
<h3>Recent Appointments</h3>
${table(["Patient","Doctor","Date","Time","Status"],
a.slice().reverse().slice(0,8).map(x=>`
<tr><td>${safe(x.patient)}</td><td>${safe(x.doctor)}</td>
<td>${safe(x.date)}</td><td>${safe(x.time)}</td>
<td><span class="status">${safe(x.status)}</span></td></tr>`))}
</section>`;
}

function field(label,name,placeholder="",type="text",required=false,options=""){
return `<div class="field">
<label>${label}${required?" *":""}</label>
${options?`<select name="${name}" ${required?"required":""}>${options}</select>`:
`<input name="${name}" type="${type}" placeholder="${placeholder}" ${required?"required":""}>`}
</div>`;
}

function patients(){
let p=get("patients");
return head("Patient Registration","Register and manage patient information.")
+`<form class="panel" id="patientForm">
<div class="section-title">Patient Details</div>
<div class="form-grid">
${field("Title","title","","text",false,"<option>Mr.</option><option>Mrs.</option><option>Ms.</option>")}
${field("First Name","name","Enter first name","text",true)}
${field("Middle Name","middle","Enter middle name")}
${field("Surname","surname","Enter surname")}
${field("Gender","gender","","text",true,"<option value=''>Select gender</option><option>Male</option><option>Female</option><option>Other</option>")}
${field("Date of Birth","dob","","date")}
${field("Age","age","Enter age","number",true)}
${field("Blood Group","blood","","text",false,"<option>Select blood group</option><option>A+</option><option>A-</option><option>B+</option><option>B-</option><option>AB+</option><option>AB-</option><option>O+</option><option>O-</option>")}
${field("Contact Number","phone","Mobile number","tel",true)}
${field("Email","email","Email address","email")}
${field("Alternate Number","alternate","Alternate number","tel")}
${field("Emergency Number","emergency","Emergency number","tel")}
${field("Father / Spouse Details","guardian","Enter name")}
</div>

<div class="section-title">Address Details</div>
<div class="form-grid">
${field("Nationality","nationality","","text",true,"<option>Indian</option><option>Other</option>")}
${field("State","state","Enter state")}
${field("City","city","Enter city","text",true)}
${field("Area","area","Enter area","text",true)}
${field("Address","address","Enter full address","text",true)}
${field("Pin Code","pin","Enter pin code")}
${field("ID Proof","proof","","text",false,"<option>Select proof</option><option>Aadhaar Card</option><option>Driving Licence</option><option>Passport</option><option>Voter ID</option>")}
${field("ID Proof Number","proofno","Proof number")}
</div>

<div class="section-title">Other Details</div>
<div class="form-grid">
${field("Payment Type","payment","","text",true,"<option>Cash</option><option>UPI</option><option>Card</option><option>Insurance</option>")}
${field("Registration Amount","amount","10","number",true)}
${field("Discount (%)","discount","0","number")}
${field("Referral Category","referral","","text",true,"<option>Walk-in</option><option>Online</option><option>Referral</option>")}
<div class="field wide"><label>Remarks</label><textarea name="remarks"></textarea></div>
</div>
<div class="actions">
<button type="reset" class="soft">Clear</button>
<button class="primary">Create Patient</button>
</div>
</form>
<section class="panel">
<h3>Registered Patients</h3>
${table(["Patient ID","Name","Phone","Age","Gender","City"],
p.slice().reverse().map(x=>`
<tr><td>${safe(x.id)}</td><td>${safe(x.name)}</td>
<td>${safe(x.phone)}</td><td>${safe(x.age)}</td>
<td>${safe(x.gender)}</td><td>${safe(x.city)}</td></tr>`))}
</section>`;
}

function appointments(){
let a=get("appointments");
return head("Appointments","Manage patient appointments.")
+`<form class="panel" id="appointmentForm">
<h3>Book New Appointment</h3>
<div class="form-grid" style="margin-top:20px">
${field("Patient Name","patient","Patient name","text",true)}
${field("Doctor Name","doctor","Doctor name","text",true)}
${field("Department","department","","text",false,"<option>General Medicine</option><option>Dental</option><option>Cardiology</option><option>Dermatology</option><option>Paediatrics</option>")}
${field("Appointment Date","date","","date",true)}
${field("Appointment Time","time","","time",true)}
${field("Visit Type","visit","","text",false,"<option>Consultation</option><option>Follow-up</option><option>Emergency</option>")}
</div>
<div class="actions"><button class="primary">Book Appointment</button></div>
</form>
<section class="panel"><h3>Appointment Records</h3>
${table(["Patient","Doctor","Department","Date","Time","Status"],
a.slice().reverse().map(x=>`
<tr><td>${safe(x.patient)}</td><td>${safe(x.doctor)}</td>
<td>${safe(x.department)}</td><td>${safe(x.date)}</td>
<td>${safe(x.time)}</td><td><span class="status">${safe(x.status)}</span></td></tr>`))}
</section>`;
}

function prescriptions(){
let r=get("prescriptions");
return head("Digital Prescription","Create and manage patient prescriptions.")
+`<form class="panel" id="rxForm">
<h3>Prescription Details</h3>
<div class="form-grid" style="margin-top:18px">
${field("Patient Name","patient","Patient name","text",true)}
${field("Patient ID","patientid","Patient ID")}
${field("Doctor Name","doctor","Doctor name","text",true)}
${field("Consultation Date","date","","date",true)}
<div class="field wide"><label>Diagnosis</label><input name="diagnosis" placeholder="Enter diagnosis"></div>
</div>
<div class="section-title">Medicines</div>
<div id="medicines"></div>
<button type="button" class="soft" onclick="addMedicine()">+ Add Medicine</button>
<div class="section-title">Instructions</div>
<div class="form-grid">
<div class="field wide"><label>Advice</label><textarea name="advice" rows="3"></textarea></div>
${field("Follow-up Date","followup","","date")}
</div>
<div class="actions"><button class="primary">Save Prescription</button></div>
</form>
<section class="panel"><h3>Saved Prescriptions</h3>
${table(["Patient","Doctor","Diagnosis","Medicines","Date"],
r.slice().reverse().map(x=>`
<tr><td>${safe(x.patient)}</td><td>${safe(x.doctor)}</td>
<td>${safe(x.diagnosis)}</td>
<td>${safe(x.medicines.map(m=>m.name).join(", "))}</td>
<td>${safe(x.date)}</td></tr>`))}
</section>`;
}

function addMedicine(){
let d=document.createElement("div");
d.className="rxrow";
d.innerHTML=`
<div class="field"><label>Medicine Name</label><input name="medicine" required></div>
<div class="field"><label>Dosage</label><input name="dosage" placeholder="1 tablet"></div>
<div class="field"><label>Duration</label><input name="duration" placeholder="5 days"></div>
<button type="button" class="soft" onclick="this.parentElement.remove()">Remove</button>`;
document.getElementById("medicines").appendChild(d);
}

function billing(){
let b=get("bills");
return head("Payments & Billing","Manage clinic invoices and payments.")
+`<form class="panel" id="billForm">
<h3>Create New Bill</h3>
<div class="form-grid" style="margin-top:20px">
${field("Patient Name","patient","Patient name","text",true)}
${field("Patient ID","patientid","Patient ID")}
${field("Service","service","Consultation fee","text",true)}
${field("Amount (₹)","amount","Amount","number",true)}
${field("Discount (₹)","discount","Discount","number")}
${field("Payment Mode","mode","","text",true,"<option>Cash</option><option>UPI</option><option>Card</option><option>Insurance</option>")}
${field("Date","date","","date",true)}
</div>
<div class="actions"><button class="primary">Save Bill</button></div>
</form>
<section class="panel"><h3>Billing History</h3>
${table(["Invoice ID","Patient","Service","Amount","Mode","Date"],
b.slice().reverse().map(x=>`
<tr><td>${safe(x.id)}</td><td>${safe(x.patient)}</td>
<td>${safe(x.service)}</td><td>₹${safe(x.amount)}</td>
<td>${safe(x.mode)}</td><td>${safe(x.date)}</td></tr>`))}
</section>`;
}

function reports(){
let p=get("patients"),a=get("appointments"),r=get("prescriptions"),b=get("bills");
let revenue=b.reduce((s,x)=>s+Number(x.amount||0),0);
return head("Clinic Reports","Summary of clinic activity.")
+`<div class="stats">
${stat("Patients",p.length,"♙")}
${stat("Appointments",a.length,"▣")}
${stat("Prescriptions",r.length,"✎")}
${stat("Revenue","₹"+revenue,"₹")}
</div>
<section class="panel">
<h3>Export Data</h3>
<p class="muted">Download a backup of records stored in this browser.</p>
<button class="primary" onclick="exportData()">Export JSON Backup</button>
</section>`;
}

function exportData(){
let data={
patients:get("patients"),
appointments:get("appointments"),
prescriptions:get("prescriptions"),
bills:get("bills")
};
let a=document.createElement("a");
a.href=URL.createObjectURL(new Blob([JSON.stringify(data,null,2)],{type:"application/json"}));
a.download="clinic-backup.json";
a.click();
}

function render(){
if(!user){login();return}
let pages={dashboard,patients,appointments,prescriptions,billing,reports};
shell(pages[page]());

let pf=document.getElementById("patientForm");
if(pf)pf.onsubmit=e=>{
e.preventDefault();
let x=Object.fromEntries(new FormData(pf));
x.id="PT-"+Date.now().toString().slice(-7);
save("patients",[...get("patients"),x]);
alert("Patient registered successfully");
go("patients");
};

let af=document.getElementById("appointmentForm");
if(af)af.onsubmit=e=>{
e.preventDefault();
let x=Object.fromEntries(new FormData(af));
x.status="Confirmed";
save("appointments",[...get("appointments"),x]);
alert("Appointment booked successfully");
go("appointments");
};

let rf=document.getElementById("rxForm");
if(rf){
addMedicine();
rf.onsubmit=e=>{
e.preventDefault();
let x=Object.fromEntries(new FormData(rf));
x.medicines=[...document.querySelectorAll(".rxrow")].map(row=>({
name:row.querySelector('[name="medicine"]').value,
dosage:row.querySelector('[name="dosage"]').value,
duration:row.querySelector('[name="duration"]').value
}));
x.id="RX-"+Date.now();
save("prescriptions",[...get("prescriptions"),x]);
alert("Prescription saved successfully");
go("prescriptions");
};
}

let bf=document.getElementById("billForm");
if(bf)bf.onsubmit=e=>{
e.preventDefault();
let x=Object.fromEntries(new FormData(bf));
x.id="INV-"+Date.now().toString().slice(-7);
save("bills",[...get("bills"),x]);
alert("Bill saved successfully");
go("billing");
};
}
login();
</script>
</body>
</html>
