--- K U M A S G I ----
<html lang="en">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>Transaction Tracker</title>
  <style>
    body {
      font-family: Arial, sans-serif;
      margin: 0;
      padding: 0;
      background: #f5f7fa;
      color: #333;
    }
    header {
      background: linear-gradient(135deg, #4e73df, #1cc88a);
      color: white;
      text-align: center;
      padding: 20px;
      font-size: 24px;
      font-weight: bold;
      position: relative;
      box-shadow: 0 4px 6px rgba(0,0,0,0.2);
    }
    header span {
      font-size: 14px;
      display: block;
      margin-top: 5px;
    }
    header .bell {
      position: absolute;
      right: 15px;
      top: 20px;
      font-size: 20px;
      cursor: pointer;
    }
    .summary {
      display: grid;
      grid-template-columns: repeat(auto-fit, minmax(180px, 1fr));
      gap: 15px;
      margin: 20px;
    }
    .box {
      background: white;
      padding: 15px;
      border-radius: 12px;
      box-shadow: 0 4px 6px rgba(0,0,0,0.15);
      text-align: center;
    }
    .box h3 {
      margin: 0;
      font-size: 18px;
      color: #444;
    }
    .box p {
      font-size: 16px;
      margin: 5px 0 0;
      font-weight: bold;
    }
    .green { background: #d4edda; }
    .red { background: #f8d7da; }
    .blue { background: #d1ecf1; }
    .center-title {
      text-align: center;
      background: #e2e6ea;
      padding: 10px;
      font-weight: bold;
      border-radius: 8px;
      margin: 15px;
      box-shadow: inset 0 2px 4px rgba(0,0,0,0.1);
    }
    table {
      width: 95%;
      margin: 15px auto;
      border-collapse: collapse;
      background: white;
      box-shadow: 0 4px 6px rgba(0,0,0,0.15);
      border-radius: 10px;
      overflow: hidden;
    }
    table th, table td {
      padding: 10px;
      text-align: center;
      border-bottom: 1px solid #ddd;
    }
    table th {
      background: #4e73df;
      color: white;
    }
    .actions button {
      border: none;
      background: none;
      cursor: pointer;
      font-size: 18px;
      color: #e74a3b;
    }
    .floating-btn {
      position: fixed;
      bottom: 20px;
      width: 50px;
      height: 50px;
      border-radius: 50%;
      border: none;
      color: white;
      font-size: 22px;
      cursor: pointer;
      box-shadow: 0 4px 6px rgba(0,0,0,0.3);
    }
    #addBtn {
      right: 20px;
      background: #1cc88a;
    }
    #borrowedBtn {
      right: 80px;
      background: #e74a3b;
    }
    #formContainer {
      display: none;
      position: fixed;
      bottom: 80px;
      right: 20px;
      background: white;
      padding: 15px;
      border-radius: 12px;
      box-shadow: 0 4px 8px rgba(0,0,0,0.25);
      width: 260px;
    }
    #formContainer input {
      width: 100%;
      padding: 6px;
      margin: 6px 0;
      border: 1px solid #ccc;
      border-radius: 6px;
    }
    #formContainer button {
      margin-top: 8px;
      padding: 8px;
      width: 100%;
      border: none;
      background: #4e73df;
      color: white;
      border-radius: 6px;
      cursor: pointer;
    }
  </style>
</head>
<body>

<header>
  Transaction Tracker
  <span id="today"></span>
  <div class="bell" id="notificationBell">🔔</div>
</header>

<div class="summary">
  <div class="box green">
    <h3>Total Lent</h3>
    <p id="totalLent">₹0</p>
  </div>
  <div class="box red">
    <h3>Total Borrowed</h3>
    <p id="totalBorrowed">₹0</p>
  </div>
  <div class="box blue">
    <h3>Net Balance</h3>
    <p id="netBalance">₹0</p>
  </div>
  <div class="box blue">
    <h3>Net Balance w/ Interest</h3>
    <p id="netBalanceWithInterest">₹0</p>
  </div>
</div>

<div class="center-title">Transaction Details</div>

<table id="transactionTable">
  <thead>
    <tr>
      <th>#</th>
      <th>Name</th>
      <th>Amount</th>
      <th>Interest %</th>
      <th>From</th>
      <th>Until</th>
      <th>Type</th>
      <th>Action</th>
    </tr>
  </thead>
  <tbody></tbody>
</table>

<div id="formContainer">
  <input type="text" id="name" placeholder="Name">
  <input type="number" id="amount" placeholder="Amount">
  <input type="number" step="0.1" id="interest" placeholder="Interest %">
  <input type="date" id="fromDate">
  <input type="date" id="untilDate">
  <select id="type">
    <option value="Lent">Lent</option>
    <option value="Borrowed">Borrowed</option>
  </select>
  <button onclick="addTransaction()">Add Transaction</button>
</div>

<button id="addBtn" class="floating-btn">+</button>
<button id="borrowedBtn" class="floating-btn">B</button>

<script>
  let transactions = JSON.parse(localStorage.getItem("transactions")) || [];
  let showingBorrowed = false;

  function renderTransactions(filterBorrowed = false) {
    const tbody = document.querySelector("#transactionTable tbody");
    tbody.innerHTML = "";
    let totalLent = 0, totalBorrowed = 0, netInterest = 0;

    transactions.forEach((t, index) => {
      if (filterBorrowed && t.type !== "Borrowed") return;

      let row = document.createElement("tr");
      let interestEarned = 0;

      if (t.interest > 0) {
        let from = new Date(t.fromDate);
        let until = new Date(t.untilDate);
        let today = new Date();
        let months = Math.max(0, (Math.min(today, until) - from) / (1000*60*60*24*30));
        interestEarned = (t.amount * (t.interest / 100)) * months;
      }

      if (t.type === "Lent") totalLent += t.amount + interestEarned;
      else totalBorrowed += t.amount + interestEarned;

      netInterest += interestEarned;

      row.innerHTML = `
        <td>${index + 1}</td>
        <td>${t.name}</td>
        <td>₹${t.amount}</td>
        <td>${t.interest}%</td>
        <td>${t.fromDate}</td>
        <td>${t.untilDate}</td>
        <td>${t.type}</td>
        <td class="actions"><button onclick="deleteTransaction(${index})">🗑</button></td>
      `;
      tbody.appendChild(row);
    });

    document.getElementById("totalLent").innerText = "₹" + totalLent.toFixed(2);
    document.getElementById("totalBorrowed").innerText = "₹" + totalBorrowed.toFixed(2);
    document.getElementById("netBalance").innerText = "₹" + (totalLent - totalBorrowed).toFixed(2);
    document.getElementById("netBalanceWithInterest").innerText = "₹" + (totalLent - totalBorrowed + netInterest).toFixed(2);
  }

  function addTransaction() {
    let name = document.getElementById("name").value;
    let amount = parseFloat(document.getElementById("amount").value);
    let interest = parseFloat(document.getElementById("interest").value) || 0;
    let fromDate = document.getElementById("fromDate").value;
    let untilDate = document.getElementById("untilDate").value;
    let type = document.getElementById("type").value;

    if (!name || !amount || !fromDate || !untilDate) return;

    transactions.push({ name, amount, interest, fromDate, untilDate, type });
    localStorage.setItem("transactions", JSON.stringify(transactions));
    document.getElementById("formContainer").style.display = "none";
    renderTransactions(showingBorrowed);
  }

  function deleteTransaction(index) {
    transactions.splice(index, 1);
    localStorage.setItem("transactions", JSON.stringify(transactions));
    renderTransactions(showingBorrowed);
  }

  document.getElementById("addBtn").addEventListener("click", () => {
    let form = document.getElementById("formContainer");
    form.style.display = form.style.display === "block" ? "none" : "block";
  });

  document.getElementById("borrowedBtn").addEventListener("click", () => {
    showingBorrowed = !showingBorrowed;
    renderTransactions(showingBorrowed);
  });

  document.getElementById("today").innerText = new Date().toLocaleDateString('en-US', { weekday:'long', year:'numeric', month:'short', day:'numeric' });

  function checkNotifications() {
    let today = new Date().toISOString().split("T")[0];
    let due = transactions.find(t => t.untilDate === today);
    if (due) alert("Reminder: " + due.name + " should return ₹" + due.amount + " today!");
  }

  document.getElementById("notificationBell").addEventListener("click", checkNotifications);

  renderTransactions();
</script>
</body>
</html>
