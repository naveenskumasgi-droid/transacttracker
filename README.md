------- K U M A S G I ----------
<html lang="en">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>Transaction Tracker</title>
  <style>
    body {
      font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
      background: #f5f7fb;
      margin: 0;
      padding: 0;
    }

    header {
      background: #6c9ff5;
      color: white;
      padding: 20px;
      text-align: center;
      box-shadow: 0 2px 8px rgba(0,0,0,0.15);
      position: sticky;
      top: 0;
      z-index: 1000;
    }

    header h1 {
      margin: 0;
      font-size: 26px;
      display: flex;
      align-items: center;
      justify-content: center;
      gap: 8px;
    }

    .summary {
      display: flex;
      justify-content: space-around;
      margin: 15px;
      gap: 15px;
    }

    .summary-box {
      background: white;
      padding: 15px;
      border-radius: 10px;
      box-shadow: 0 3px 10px rgba(0,0,0,0.1);
      flex: 1;
      text-align: center;
    }

    .summary-box h3 {
      margin: 0;
      font-size: 15px;
      color: #444;
    }

    .summary-box p {
      font-size: 20px;
      font-weight: bold;
      margin: 8px 0 0;
      color: #222;
    }

    .container {
      display: flex;
      gap: 20px;
      padding: 20px;
    }

    .table-container {
      flex: 2;
      background: white;
      padding: 15px;
      border-radius: 10px;
      box-shadow: 0 2px 8px rgba(0,0,0,0.1);
      overflow-x: auto;
    }

    table {
      width: 100%;
      border-collapse: collapse;
      margin-top: 15px;
      background: #fdfdfd;
    }

    table thead {
      background: #e9f0ff;
    }

    table th, table td {
      padding: 10px;
      text-align: center;
      border: 1px solid #ddd;
    }

    table th {
      font-size: 14px;
      font-weight: bold;
      color: #333;
    }

    table td {
      font-size: 13px;
    }

    .lent {
      background: #d9fcd9;
      padding: 6px 10px;
      border-radius: 6px;
      color: #1b5e20;
      font-weight: bold;
    }

    .borrowed {
      background: #ffe0e0;
      padding: 6px 10px;
      border-radius: 6px;
      color: #b71c1c;
      font-weight: bold;
    }

    .action-btn {
      cursor: pointer;
      color: #d9534f;
      font-size: 16px;
    }

    .form-container {
      margin: 20px;
      background: white;
      padding: 15px;
      border-radius: 10px;
      box-shadow: 0 2px 8px rgba(0,0,0,0.1);
    }

    .form-container h2 {
      margin-top: 0;
      text-align: center;
      font-size: 20px;
      color: #333;
    }

    .form-group {
      margin-bottom: 12px;
    }

    .form-group label {
      display: block;
      font-size: 14px;
      margin-bottom: 5px;
    }

    .form-group input, 
    .form-group select {
      width: 100%;
      padding: 8px;
      border: 1px solid #ccc;
      border-radius: 6px;
    }

    button {
      background: #6c9ff5;
      color: white;
      border: none;
      padding: 10px 15px;
      border-radius: 6px;
      cursor: pointer;
      font-size: 14px;
    }

    button:hover {
      background: #5184db;
    }

    .chart-container {
      flex: 1;
      background: white;
      padding: 15px;
      border-radius: 10px;
      box-shadow: 0 2px 8px rgba(0,0,0,0.1);
    }
  </style>
</head>
<body>
  <header>
    <h1>📊 Transaction Tracker</h1>
    <p id="todayDate"></p>
  </header>

  <div class="summary">
    <div class="summary-box">
      <h3>Total Lent</h3>
      <p id="totalLent">₹0</p>
    </div>
    <div class="summary-box">
      <h3>Total Borrowed</h3>
      <p id="totalBorrowed">₹0</p>
    </div>
    <div class="summary-box">
      <h3>Net Balance</h3>
      <p id="netBalance">₹0</p>
    </div>
  </div>

  <div class="container">
    <div class="table-container">
      <h2>Transaction Details</h2>
      <table id="transactionTable">
        <thead>
          <tr>
            <th>Nos</th>
            <th>Name</th>
            <th>Type</th>
            <th>Amount</th>
            <th>Interest %</th>
            <th>From Date</th>
            <th>Until Date</th>
            <th>Net Balance</th>
            <th>Reason</th>
            <th>Action</th>
          </tr>
        </thead>
        <tbody></tbody>
      </table>
    </div>

    <div class="chart-container">
      <h2>Overview</h2>
      <canvas id="lineChart"></canvas>
    </div>
  </div>

  <div class="form-container">
    <h2>Add Transaction</h2>
    <div class="form-group">
      <label>Name</label>
      <input type="text" id="name" required>
    </div>
    <div class="form-group">
      <label>Type</label>
      <select id="type">
        <option value="lent">Lent</option>
        <option value="borrowed">Borrowed</option>
      </select>
    </div>
    <div class="form-group">
      <label>Amount</label>
      <input type="number" id="amount" required>
    </div>
    <div class="form-group">
      <label>Interest %</label>
      <input type="number" id="interest" step="0.1" placeholder="e.g. 5.6">
    </div>
    <div class="form-group">
      <label>From Date</label>
      <input type="date" id="fromDate" required>
    </div>
    <div class="form-group">
      <label>Until Date</label>
      <input type="date" id="untilDate" required>
    </div>
    <div class="form-group">
      <label>Reason</label>
      <input type="text" id="reason">
    </div>
    <button onclick="addTransaction()">Add Transaction</button>
  </div>

  <script src="https://cdn.jsdelivr.net/npm/chart.js"></script>
  <script>
    const tableBody = document.querySelector("#transactionTable tbody");
    let transactions = JSON.parse(localStorage.getItem("transactions")) || [];

    function renderTransactions() {
      tableBody.innerHTML = "";
      let totalLent = 0, totalBorrowed = 0;

      transactions.forEach((t, index) => {
        const row = document.createElement("tr");
        const amountBox = t.type === "lent" ? 
          `<span class='lent'>₹${t.amount}</span>` : 
          `<span class='borrowed'>₹${t.amount}</span>`;

        const months = Math.max(1, Math.ceil((new Date(t.untilDate) - new Date(t.fromDate)) / (1000*60*60*24*30)));
        const netBalance = t.amount + (t.amount * (t.interest/100) * months);

        row.innerHTML = `
          <td>${index+1}</td>
          <td>${t.name}</td>
          <td>${t.type}</td>
          <td>${amountBox}</td>
          <td>${t.interest || 0}%</td>
          <td>${t.fromDate}</td>
          <td>${t.untilDate}</td>
          <td>₹${netBalance.toFixed(2)}</td>
          <td>${t.reason}</td>
          <td><span class="action-btn" onclick="deleteTransaction(${index})">🗑️</span></td>
        `;

        tableBody.appendChild(row);

        if(t.type === "lent") totalLent += netBalance;
        else totalBorrowed += netBalance;
      });

      document.getElementById("totalLent").innerText = "₹" + totalLent.toFixed(2);
      document.getElementById("totalBorrowed").innerText = "₹" + totalBorrowed.toFixed(2);
      document.getElementById("netBalance").innerText = "₹" + (totalLent - totalBorrowed).toFixed(2);

      updateChart();
      localStorage.setItem("transactions", JSON.stringify(transactions));
    }

    function addTransaction() {
      const name = document.getElementById("name").value;
      const type = document.getElementById("type").value;
      const amount = parseFloat(document.getElementById("amount").value);
      const interest = parseFloat(document.getElementById("interest").value) || 0;
      const fromDate = document.getElementById("fromDate").value;
      const untilDate = document.getElementById("untilDate").value;
      const reason = document.getElementById("reason").value;

      if(name && amount && fromDate && untilDate) {
        transactions.push({ name, type, amount, interest, fromDate, untilDate, reason });
        renderTransactions();
        document.querySelectorAll(".form-group input").forEach(i => i.value = "");
      }
    }

    function deleteTransaction(index) {
      transactions.splice(index, 1);
      renderTransactions();
    }

    let chart;
    function updateChart() {
      const ctx = document.getElementById("lineChart").getContext("2d");
      if(chart) chart.destroy();

      chart = new Chart(ctx, {
        type: 'line',
        data: {
          labels: transactions.map((t, i) => "T"+(i+1)),
          datasets: [{
            label: "Net Balance",
            data: transactions.map(t => {
              const months = Math.max(1, Math.ceil((new Date(t.untilDate) - new Date(t.fromDate)) / (1000*60*60*24*30)));
              return t.amount + (t.amount * (t.interest/100) * months);
            }),
            borderColor: "#6c9ff5",
            fill: false,
            tension: 0.3
          }]
        }
      });
    }

    document.getElementById("todayDate").innerText = new Date().toDateString();
    renderTransactions();
  </script>
</body>
</html>
