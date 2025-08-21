<html>
<head>
  <meta charset="UTF-8">
  <title>Transaction Tracker</title>
  <style>
    body {
      font-family: Arial, sans-serif;
      margin: 0;
      padding: 0;
      background: #f5f7fb;
      color: #333;
    }
    header {
      background: #4a90e2;
      color: white;
      padding: 15px;
      display: flex;
      justify-content: space-between;
      align-items: center;
      box-shadow: 0 2px 6px rgba(0,0,0,0.2);
    }
    header h2 { margin: 0; }
    #notification { font-size: 18px; cursor: pointer; position: relative; }
    #notification span {
      position: absolute;
      top: -8px;
      right: -8px;
      background: red;
      color: white;
      font-size: 12px;
      padding: 2px 6px;
      border-radius: 50%;
    }
    .summary {
      display: flex;
      justify-content: space-around;
      margin: 20px;
    }
    .box {
      flex: 1;
      margin: 0 10px;
      padding: 15px;
      border-radius: 10px;
      text-align: center;
      box-shadow: 0 2px 8px rgba(0,0,0,0.15);
      color: white;
      font-weight: bold;
    }
    .lent { background: #4CAF50; }
    .borrowed { background: #E53935; }
    .balance { background: #6A5ACD; }
    .balance-interest { background: #FF9800; }
    table {
      width: 95%;
      margin: 20px auto;
      border-collapse: collapse;
      background: white;
      box-shadow: 0 2px 8px rgba(0,0,0,0.15);
      border-radius: 10px;
      overflow: hidden;
    }
    th, td {
      border: 1px solid #ddd;
      padding: 12px;
      text-align: center;
    }
    th {
      background: #4a90e2;
      color: white;
    }
    .actions button {
      border: none;
      background: none;
      cursor: pointer;
      font-size: 18px;
    }
    #addBtn, #borrowedBtn {
      position: fixed;
      bottom: 20px;
      right: 20px;
      font-size: 24px;
      padding: 15px;
      border: none;
      border-radius: 50%;
      cursor: pointer;
      color: white;
      box-shadow: 0 2px 6px rgba(0,0,0,0.3);
    }
    #addBtn { background: #4a90e2; margin-bottom: 70px; }
    #borrowedBtn { background: #E53935; }
    #formContainer {
      display: none;
      position: fixed;
      top: 50%;
      left: 50%;
      transform: translate(-50%,-50%);
      background: white;
      padding: 20px;
      border-radius: 10px;
      box-shadow: 0 2px 10px rgba(0,0,0,0.3);
      z-index: 10;
    }
    #formContainer input, #formContainer select {
      width: 100%;
      margin: 8px 0;
      padding: 10px;
      border-radius: 6px;
      border: 1px solid #ccc;
    }
    #formContainer button {
      background: #4a90e2;
      color: white;
      border: none;
      padding: 10px;
      margin-top: 10px;
      border-radius: 6px;
      cursor: pointer;
    }
  </style>
</head>
<body>

<header>
  <h2>Transaction Tracker</h2>
  <div id="notification">🔔<span id="notifyCount" style="display:none">1</span></div>
</header>

<div class="summary">
  <div class="box lent" id="totalLent">Total Lent: ₹0</div>
  <div class="box borrowed" id="totalBorrowed">Total Borrowed: ₹0</div>
  <div class="box balance" id="netBalance">Net Balance: ₹0</div>
  <div class="box balance-interest" id="netBalanceInterest">Net Bal (with Interest): ₹0</div>
</div>

<table>
  <thead>
    <tr>
      <th>Nos</th>
      <th>Name</th>
      <th>Type</th>
      <th>Amount</th>
      <th>Interest %</th>
      <th>Date From</th>
      <th>Until Date</th>
      <th>Reason</th>
      <th>Net Balance</th>
      <th>Action</th>
    </tr>
  </thead>
  <tbody id="transactionTable"></tbody>
</table>

<!-- Add Form -->
<div id="formContainer">
  <h3>Add Transaction</h3>
  <input type="text" id="name" placeholder="Name">
  <select id="type">
    <option value="Lent">Lent</option>
    <option value="Borrowed">Borrowed</option>
  </select>
  <input type="number" id="amount" placeholder="Amount">
  <input type="number" step="0.1" id="interest" placeholder="Interest % (e.g. 5.6)">
  <input type="date" id="dateFrom">
  <input type="date" id="untilDate">
  <input type="text" id="reason" placeholder="Reason">
  <button onclick="saveTransaction()">Save</button>
</div>

<!-- Floating Buttons -->
<button id="addBtn" onclick="showForm()">＋</button>
<button id="borrowedBtn" onclick="filterBorrowed()">B</button>

<script>
  let transactions = JSON.parse(localStorage.getItem("transactions")) || [];

  function updateSummary() {
    let lent = 0, borrowed = 0, balanceWithInterest = 0;
    transactions.forEach(t => {
      if(t.type === "Lent") lent += t.amount;
      else borrowed += t.amount;
      balanceWithInterest += t.netBalance;
    });
    document.getElementById("totalLent").innerText = "Total Lent: ₹" + lent;
    document.getElementById("totalBorrowed").innerText = "Total Borrowed: ₹" + borrowed;
    document.getElementById("netBalance").innerText = "Net Balance: ₹" + (lent - borrowed);
    document.getElementById("netBalanceInterest").innerText = "Net Bal (with Interest): ₹" + balanceWithInterest.toFixed(2);
  }

  function renderTable(filtered=false) {
    const table = document.getElementById("transactionTable");
    table.innerHTML = "";
    transactions.forEach((t, index) => {
      if(filtered && t.type !== "Borrowed") return;
      const row = `<tr>
        <td>${index+1}</td>
        <td>${t.name}</td>
        <td>${t.type}</td>
        <td>₹${t.amount}</td>
        <td>${t.interest}%</td>
        <td>${t.dateFrom}</td>
        <td>${t.untilDate}</td>
        <td>${t.reason}</td>
        <td>₹${t.netBalance.toFixed(2)}</td>
        <td class="actions"><button onclick="deleteTransaction(${index})">🗑️</button></td>
      </tr>`;
      table.innerHTML += row;
    });
    updateSummary();
  }

  function showForm() {
    document.getElementById("formContainer").style.display = "block";
  }
  function hideForm() {
    document.getElementById("formContainer").style.display = "none";
  }

  function saveTransaction() {
    const name = document.getElementById("name").value;
    const type = document.getElementById("type").value;
    const amount = parseFloat(document.getElementById("amount").value);
    const interest = parseFloat(document.getElementById("interest").value) || 0;
    const dateFrom = document.getElementById("dateFrom").value;
    const untilDate = document.getElementById("untilDate").value;
    const reason = document.getElementById("reason").value;

    if(!name || !amount || !dateFrom || !untilDate) { alert("Fill all fields"); return; }

    let months = (new Date(untilDate).getFullYear()-new Date(dateFrom).getFullYear())*12 + 
                 (new Date(untilDate).getMonth()-new Date(dateFrom).getMonth());
    let netBalance = type==="Lent" ? amount + (amount*interest*months/100) : 
                                     amount + (amount*interest*months/100);

    transactions.push({name, type, amount, interest, dateFrom, untilDate, reason, netBalance});
    localStorage.setItem("transactions", JSON.stringify(transactions));
    hideForm();
    renderTable();
  }

  function deleteTransaction(index) {
    transactions.splice(index, 1);
    localStorage.setItem("transactions", JSON.stringify(transactions));
    renderTable();
  }

  function filterBorrowed() {
    renderTable(true);
    setTimeout(()=>renderTable(), 4000);
  }

  function checkNotifications() {
    let today = new Date().toISOString().split("T")[0];
    let notify = transactions.some(t => t.untilDate === today);
    document.getElementById("notifyCount").style.display = notify ? "block" : "none";
  }

  renderTable();
  checkNotifications();
</script>
</body>
</html>

