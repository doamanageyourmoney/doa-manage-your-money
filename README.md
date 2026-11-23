
<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>DOA | Personal Financial Assistant</title>
<style>
*{box-sizing:border-box}
body{margin:0;font-family:Arial,sans-serif;background:#f3f5f8;color:#172033}
header{background:#102b46;color:white;padding:24px 18px}
header h1{margin:0;font-size:28px;letter-spacing:2px}
header p{margin:7px 0 0;color:#c9d6e3}
main{max-width:900px;margin:auto;padding:18px}
h2{font-size:19px}
.cards{display:grid;grid-template-columns:repeat(3,1fr);gap:10px}
.card{background:white;padding:16px 12px;border-radius:12px;box-shadow:0 2px 9px #0000000c}
.card small{color:#64748b}
.card strong{display:block;margin-top:10px;font-size:18px;overflow-wrap:anywhere}
.green{color:#13834b}
.red{color:#d33b42}
section{background:white;padding:18px;border-radius:12px;margin-top:18px}
input,select,textarea{width:100%;padding:13px;margin:7px 0 13px;border:1px solid #ccd5df;border-radius:8px;font-size:15px}
button{background:#123c60;color:white;border:0;padding:13px 18px;border-radius:8px;font-weight:bold;cursor:pointer}
button:hover{background:#20577f}
.delete{background:#b8323b;padding:7px 10px;font-size:12px}
table{width:100%;border-collapse:collapse;font-size:13px}
th,td{text-align:left;padding:11px 6px;border-bottom:1px solid #e5eaf0}
th{color:#64748b}
.empty{text-align:center;color:#7a8797;padding:20px}
footer{text-align:center;color:#8290a0;padding:25px}
@media(max-width:520px){
.cards{grid-template-columns:1fr}
.card strong{font-size:20px}
table{font-size:12px}
}
</style>
</head>
<body>
<header>
<h1>DOA</h1>
<p>Personal Financial Assistant</p>
</header>

<main>
<h2>Financial Overview</h2>
<div class="cards">
<div class="card">
<small>Total Income</small>
<strong class="green" id="income">UGX 0</strong>
</div>
<div class="card">
<small>Total Expenses</small>
<strong class="red" id="expenses">UGX 0</strong>
</div>
<div class="card">
<small>Available Balance</small>
<strong id="balance">UGX 0</strong>
</div>
</div>

<section>
<h2>Record a Transaction</h2>
<form id="transactionForm">
<label>Transaction type</label>
<select id="type" required>
<option value="income">Income</option>
<option value="expense">Expense</option>
</select>

<label>Amount (UGX)</label>
<input type="number" id="amount" min="1" step="1" placeholder="Enter amount" required>

<label>Category</label>
<select id="category" required>
<option>Salary</option>
<option>Business</option>
<option>Sales</option>
<option>Food</option>
<option>Transport</option>
<option>Rent</option>
<option>Utilities</option>
<option>Education</option>
<option>Health</option>
<option>Shopping</option>
<option>Other</option>
</select>

<label>Description</label>
<input type="text" id="description" placeholder="What was this transaction for?">

<label>Date</label>
<input type="date" id="date" required>

<button type="submit">Save Transaction</button>
</form>
</section>

<section>
<h2>Transaction History</h2>
<div style="overflow-x:auto">
<table>
<thead>
<tr>
<th>Date</th>
<th>Type</th>
<th>Category</th>
<th>Amount</th>
<th>Action</th>
</tr>
</thead>
<tbody id="records">
<tr><td colspan="5" class="empty">No transactions recorded yet.</td></tr>
</tbody>
</table>
</div>
</section>

<section>
<h2>Financial Report</h2>
<p id="report">Start recording transactions to see your financial report.</p>
<button onclick="exportData()">Export Records (CSV)</button>
</section>

</main>

<footer>DOA Personal Financial Assistant · Manage your money wisely.</footer>

<script>
let transactions = [];
try {
 transactions = JSON.parse(localStorage.getItem("doa_transactions") || "[]");
 if (!Array.isArray(transactions)) transactions = [];
} catch(e) { transactions = []; }

const money = n => "UGX " + Math.round(n).toLocaleString("en-UG");

function save() {
 localStorage.setItem("doa_transactions", JSON.stringify(transactions));
 render();
}

function render() {
 const income = transactions.filter(t=>t.type==="income")
 .reduce((s,t)=>s+t.amount,0);
 const expenses = transactions.filter(t=>t.type==="expense")
 .reduce((s,t)=>s+t.amount,0);

 document.getElementById("income").textContent=money(income);
 document.getElementById("expenses").textContent=money(expenses);
 document.getElementById("balance").textContent=money(income-expenses);

 const records=document.getElementById("records");
 records.innerHTML="";

 if(!transactions.length) {
  records.innerHTML='<tr><td colspan="5" class="empty">No transactions recorded yet.</td></tr>';
 } else {
  [...transactions].reverse().forEach(t=>{
   const tr=document.createElement("tr");
   const values=[t.date,t.type,t.category,
    (t.type==="income"?"+":"-")+money(t.amount)];
   values.forEach((v,i)=>{
    const td=document.createElement("td");
    td.textContent=v;
    if(i===3) td.style.color=t.type==="income"?"#13834b":"#d33b42";
    tr.appendChild(td);
   });
   const action=document.createElement("td");
   const btn=document.createElement("button");
   btn.className="delete";
   btn.textContent="Delete";
   btn.onclick=()=>removeTransaction(t.id);
   action.appendChild(btn);
   tr.appendChild(action);
   records.appendChild(tr);
  });
 }

 document.getElementById("report").textContent=
 "You have recorded "+transactions.length+
 " transactions. Your current balance is "+money(income-expenses)+
 ". Your total income is "+money(income)+
 " and your total expenses are "+money(expenses)+".";
}

document.getElementById("date").value=
 new Date().toISOString().slice(0,10);

document.getElementById("transactionForm").addEventListener("submit",e=>{
 e.preventDefault();
 const amount=Number(document.getElementById("amount").value);
 if(!Number.isFinite(amount)||amount<=0) {
  alert("Enter a valid amount.");
  return;
 }
 transactions.push({
  id:Date.now()+Math.random(),
  type:document.getElementById("type").value,
  amount:amount,
  category:document.getElementById("category").value,
  description:document.getElementById("description").value,
  date:document.getElementById("date").value
 });
 save();
 e.target.reset();
 document.getElementById("date").value=
  new Date().toISOString().slice(0,10);
 alert("Transaction saved successfully.");
});

function removeTransaction(id) {
 if(confirm("Delete this transaction?")) {
  transactions=transactions.filter(t=>t.id!==id);
  save();
 }
}

function exportData() {
 const rows=[["Date","Type","Category","Description","Amount"]];
 transactions.forEach(t=>rows.push([
  t.date,t.type,t.category,t.description,t.amount
 ]));
 const csv=rows.map(r=>r.map(v=>
  '"'+String(v??"").replace(/"/g,'""')+'"'
 ).join(",")).join("\n");
 const blob=new Blob([csv],{type:"text/csv"});
 const url=URL.createObjectURL(blob);
 const a=document.createElement("a");
 a.href=url;
 a.download="DOA-financial-records.csv";
 a.click();
 URL.revokeObjectURL(url);
}

function startVoiceEntry() {
  const Speech = window.SpeechRecognition || window.webkitSpeechRecognition;
  const status = document.getElementById("voiceStatus");

  if (!Speech) {
    status.textContent = "Voice recognition is not supported. Please use Chrome.";
    return;
  }

  const language = document.getElementById("voiceLanguage")?.value || "en-US";
  const recognition = new Speech();

  recognition.lang = language;
  recognition.interimResults = false;
  recognition.maxAlternatives = 1;

  status.textContent = "Listening... Tell me about your transaction.";

  recognition.onresult = function(event) {
    const spoken = event.results[0][0].transcript;
    const amountMatch = spoken.match(/\d[\d,]*(?:\.\d+)?/);

    if (!amountMatch) {
      status.textContent = "I heard: " + spoken + ". Please enter the amount manually, then try again.";
      document.getElementById("description").value = spoken;
      return;
    }

    const amount = amountMatch[0].replace(/,/g, "");
    const words = spoken.toLowerCase();

    const type = /\b(income|received|earned|salary|sold)\b/i.test(words)
      ? "income" : "expense";

    document.getElementById("type").value = type;
    document.getElementById("description").value = spoken;
    document.getElementById("amount").value = amount;

    if (/food|lunch|dinner|meal/i.test(words))
      document.getElementById("category").value = "Food";
    else if (/transport|taxi|fuel/i.test(words))
      document.getElementById("category").value = "Transport";
    else if (/rent/i.test(words))
      document.getElementById("category").value = "Rent";

    status.textContent =
      "I heard: " + spoken + ". Say YES to save or NO to cancel.";

    const confirmSpeech = new Speech();
    confirmSpeech.lang = language;
    confirmSpeech.interimResults = false;

    confirmSpeech.onresult = function(answer) {
      const reply = answer.results[0][0].transcript.toLowerCase();

      if (/\b(yes|yeah|correct|save|okay|ok)\b/i.test(reply)) {
        const form = document.getElementById("recordForm");

        if (form) {
          form.requestSubmit();
          status.textContent = "Your transaction has been submitted.";
        } else {
          status.textContent = "Please use the Save button to save this transaction.";
        }
      } else {
        status.textContent = "Cancelled. Your transaction was not saved.";
      }
    };

    confirmSpeech.onerror = function() {
      status.textContent = "I couldn't hear your confirmation. Please use the Save button.";
    };

    setTimeout(function() {
      confirmSpeech.start();
    }, 700);
  };

  recognition.onerror = function() {
    status.textContent = "I couldn't hear you. Please try again.";
  };

  recognition.start();
}

 render();
</script>
</body>
</html>
