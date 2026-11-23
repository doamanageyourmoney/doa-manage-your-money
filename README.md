
<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>DOA | Manage Your Money</title>
<style>
*{box-sizing:border-box}
body{margin:0;font-family:Arial,sans-serif;background:#f3f5f9;color:#17243a}
header{background:#101d32;color:white;padding:22px}
header h1{margin:0;color:#e6bd65;font-size:27px}
header p{margin:7px 0 0;color:#c6ced9}
main{max-width:1050px;margin:auto;padding:18px}
h2{font-size:19px}
.grid{display:grid;grid-template-columns:repeat(2,1fr);gap:12px}
.card{background:white;padding:17px;border-radius:13px;box-shadow:0 2px 10px #0000000a}
.card small{color:#718096}
.amount{font-size:23px;font-weight:bold;margin-top:9px}
.gold{color:#bd913b}
.green{color:#168451}
.red{color:#d64b4b}
input,select,button{width:100%;padding:13px;margin:6px 0;border:1px solid #dce1e9;border-radius:8px;font-size:15px}
button{background:#14243c;color:white;border:0;font-weight:bold;cursor:pointer}
button:hover{background:#263c5e}
.record{border-bottom:1px solid #eee;padding:13px 0;display:flex;justify-content:space-between;gap:10px}
.record small{color:#718096}
.section{margin-top:22px}
footer{text-align:center;padding:25px;color:#778397;font-size:12px}
@media(max-width:500px){main{padding:12px}.amount{font-size:19px}}
</style>
</head>
<body>

<header>
<h1>DOA</h1>
<p>Manage Your Money | Personal Financial Assistant</p>
</header>

<main>
<h2>Financial Overview</h2>

<div class="grid">
<div class="card">
<small>Total Balance</small>
<div class="amount gold" id="balance">UGX 0</div>
</div>
<div class="card">
<small>Total Income</small>
<div class="amount green" id="income">UGX 0</div>
</div>
<div class="card">
<small>Total Expenditure</small>
<div class="amount red" id="expense">UGX 0</div>
</div>
<div class="card">
<small>Transactions</small>
<div class="amount" id="count">0</div>
</div>
</div>

<div class="section card">
<h2>Record a Transaction</h2>

<form id="transactionForm">
<select id="type" required>
<option value="income">Income</option>
<option value="expense">Expenditure</option>
</select>

<select id="account" required>
<option value="Personal">Personal Finance</option>
<option value="Business">Business Finance</option>
</select>

<input id="description" placeholder="Description" required>

<input id="amount" type="number" min="1" step="any"
placeholder="Amount in UGX" required>

<select id="category">
<option>Salary</option>
<option>Business</option>
<option>Sales</option>
<option>Food</option>
<option>Transport</option>
<option>Rent</option>
<option>Education</option>
<option>Medical</option>
<option>Shopping</option>
<option>Utilities</option>
<option>Other</option>
</select>

<input id="date" type="date" required>

<button type="submit">Save Transaction</button>
</form>
</div>

<div class="section card">
<h2>Transaction History</h2>
<div id="transactions">No transactions recorded yet.</div>
</div>

<div class="section card">
<h2>DOA Financial Assistant</h2>
<p>Your financial management starts here. Record your income and expenditure to build your financial history.</p>
<p class="gold"><b>DOA — Your Money. Your Control.</b></p>
</div>

</main>

<footer>
DOA Personal Financial Assistant | Version 1.0
</footer>

<script>
const storageKey = "doa_transactions";
let transactions = JSON.parse(localStorage.getItem(storageKey) || "[]");

const money = value => "UGX " + new Intl.NumberFormat("en-UG").format(value);

function save(){
localStorage.setItem(storageKey,JSON.stringify(transactions));
render();
}

function render(){
let income = transactions
.filter(t=>t.type==="income")
.reduce((s,t)=>s+t.amount,0);

let expense = transactions
.filter(t=>t.type==="expense")
.reduce((s,t)=>s+t.amount,0);

document.getElementById("balance").textContent=money(income-expense);
document.getElementById("income").textContent=money(income);
document.getElementById("expense").textContent=money(expense);
document.getElementById("count").textContent=transactions.length;

let list=document.getElementById("transactions");
list.innerHTML="";

if(!transactions.length){
list.textContent="No transactions recorded yet.";
return;
}

[...transactions].reverse().forEach(t=>{
let div=document.createElement("div");
div.className="record";

let details=document.createElement("div");
let title=document.createElement("b");
title.textContent=t.description;
let info=document.createElement("small");
info.textContent=t.account+" | "+t.category+" | "+t.date;
details.append(title,document.createElement("br"),info);

let value=document.createElement("b");
value.className=t.type==="income"?"green":"red";
value.textContent=(t.type==="income"?"+":"-")+money(t.amount);

div.append(details,value);
list.append(div);
});
}

document.getElementById("date").value=
new Date().toISOString().slice(0,10);

document.getElementById("transactionForm")
.addEventListener("submit",function(e){
e.preventDefault();

let transaction={
type:document.getElementById("type").value,
account:document.getElementById("account").value,
description:document.getElementById("description").value,
amount:Number(document.getElementById("amount").value),
category:document.getElementById("category").value,
date:document.getElementById("date").value
};

transactions.push(transaction);
save();
this.reset();
document.getElementById("date").value=
new Date().toISOString().slice(0,10);
});

render();
</script>
</body>
</html>

