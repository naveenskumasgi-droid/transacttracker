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
      background: #f2f5f9;
    }
    header {
      display: flex;
      justify-content: center;
      align-items: center;
      padding: 15px;
      background: #4a90e2;
      color: white;
      font-size: 20px;
      font-weight: bold;
      position: relative;
    }
    header span {
      margin-left: 10px;
    }
    header .bell {
      position: absolute;
      right: 20px;
      font-size: 20px;
      cursor: pointer;
    }
    .summary {
      display: flex;
      justify-content: space-around;
      flex-wrap: wrap;
      margin: 15px;
      gap: 10px;
    }
    .box {
      flex: 1;
      min-width: 180px;
      padding: 15px;
      border-radius: 12px;
      color: white;
      box-shadow: 0 4px 8px rgba(0,0,0,0.15);
      text-align: center;
    }
    .lent { background: #2ecc71; }
    .borrowed { background: #e74c3c; }
    .net { background: #9b59b6; }
    .without-int { background: #3498db; }

    table {
      width: 95%;
      margin: 20px auto;
      border-collapse: collapse;
      background: white;
      border-radius: 12px;
      overflow: hidden;
      box-shadow: 0 4px 8px rgba(0,0,0,0.1);
    }
    th, td {
      padding: 12px;
      border: 1px solid #ddd;
      text-align: center;
    }
    th {
      background: #4a90e2;
      color: white;
    }
    .actions button {
      background: none;
      border: none;
      cursor: pointer;
      font-size: 16px;
    }

    /* Floating buttons */
    .fab {
      position: fixed;
      bottom: 20px;
      right: 20px;
      background: #4a90e2;
      color: white;
      border-radius: 50%;
      width: 55px;
      height: 55px;
      display: flex;
      justify-content: center;
      align-items: center;
      font-size: 26px;
      box-shadow: 0 4px 8px rgba(0,0,0,0.3);
      cursor: pointer;
      margin: 10px 0;
    }
    .fab.b-btn { bottom: 90px; background: #e74c3c; }

    /* Modal */
    .modal {
      display: none;
      position: fixed;
      left: 0; top: 0;
      width: 100%; height: 100%;
      background: rgba(0,0,0,0.5);
      justify-content: center;
      align-items: center;
    }
    .modal-content {
      background: white;
      padding: 20px;
      border-radius: 12px;
      width: 320px;
    }
    .modal-content input, .modal-content select {
      width: 100%;
      margin: 8px 0;
      padding: 8px;
      border-radius: 6px;
      border: 1px solid #ccc;
    }
    .modal-content button {
      width: 100%;
      padding: 10px;
      background: #4a90e2;
      color: white;
      border: none;
      border-radius: 8px;
      cursor: pointer;
      margin-top: 10px;
    }
  </style>
</head>
<body>
  <header>
    <span>📊 Transaction Tracker</span>
    <span class="bell" id="notifyBell">🔔</span>
  </header>

  <div class="summary">
    <div class="box lent">Total Lent<br><span id="totalLent">0</span></div>
    <div class="box without-int">Lent w/o Interest<br><span id="totalWithoutInterest">0</span></div>
    <div class="box borrowed">Total Borrowed<br><span id="totalBorrowed">0</span></div>
    <div class="box net">Net Balance (w/ Int)<br><span id="netBalance">0</span></div>
  </div>

  <table>
    <thead>
      <tr>
        <th>Nos.</th>
        <th>Name</th>
        <th>Type</th>
        <th>Amount</th>
        <th>Rate (%)</th>
        <th>From</th>
        <th>Until</th>
        <th>Days</th>
        <th>Reason</th>
        <th>Return</th>
        <th>Net Balance</th>
        <th>Action</th>
      </tr>
    </thead>
    <tbody id="transactionTable"></tbody>
  </table>

  <!-- Floating buttons -->
  <div class="fab" id="addBtn">+</div>
  <div class="fab b-btn" id="borrowBtn">B</div>

  <!-- Modal -->
  <div class="modal" id="modal">
    <div class="modal-content">
      <h3>Add Transaction</h3>
      <input type="text" id="name" placeholder="Name">
      <select id="type">
        <option value="lent">Lent</option>
        <option value="borrowed">Borrowed</option>
      </select>
      <input type="number" id="amount" placeholder="Amount">
      <input type="number" step="0.1" id="rate" placeholder="Interest % (optional)">
      <input type="date" id="fromDate">
      <input type="date" id="untilDate">
      <input type="text" id="reason" placeholder="Reason">
      <button onclick="addTransaction()">Save</button>
    </div>
  </div>

  <script>
    let transactions = JSON.parse(localStorage.getItem("transactions")) || [];

    function saveData() {
      localStorage.setItem("transactions", JSON.stringify(transactions));
    }

    function renderTransactions(filter="all") {
      const tbody = document.getElementById("transactionTable");
      tbody.innerHTML = "";
      let totalLent = 0, totalWithoutInt = 0, totalBorrowed = 0;

      transactions.forEach((t, i) => {
        if (filter !== "all" && t.type !== filter) return;
        const today = new Date();
        const from = new Date(t.fromDate);
        const until = new Date(t.untilDate);
        const daysPassed = Math.floor((today - from)/(1000*60*60*24));
        const months = Math.max(0, (until - from)/(1000*60*60*24*30));
        let netBalance = parseFloat(t.amount);
        if (t.rate) {
          netBalance += (t.amount * (t.rate/100) * months);
        }

        if (t.type === "lent") {
          totalLent += netBalance;
          totalWithoutInt += parseFloat(t.amount);
        } else {
          totalBorrowed += netBalance;
        }

        tbody.innerHTML += `
          <tr>
            <td>${i+1}</td>
            <td>${t.name}</td>
            <td>${t.type}</td>
            <td>${t.amount}</td>
            <td>${t.rate || "-"}</td>
            <td>${t.fromDate}</td>
            <td>${t.untilDate}</td>
            <td>${daysPassed} days</td>
            <td>${t.reason}</td>
            <td>${t.untilDate}</td>
            <td>${netBalance.toFixed(2)}</td>
            <td class="actions"><button onclick="deleteTransaction(${i})">🗑️</button></td>
          </tr>
        `;
      });

      document.getElementById("totalLent").innerText = totalLent.toFixed(2);
      document.getElementById("totalWithoutInterest").innerText = totalWithoutInt.toFixed(2);
      document.getElementById("totalBorrowed").innerText = totalBorrowed.toFixed(2);
      document.getElementById("netBalance").innerText = (totalLent-totalBorrowed).toFixed(2);
    }

    function addTransaction() {
      const t = {
        name: document.getElementById("name").value,
        type: document.getElementById("type").value,
        amount: parseFloat(document.getElementById("amount").value),
        rate: parseFloat(document.getElementById("rate").value) || null,
        fromDate: document.getElementById("fromDate").value,
        untilDate: document.getElementById("untilDate").value,
        reason: document.getElementById("reason").value
      };
      transactions.push(t);
      saveData();
      renderTransactions();
      closeModal();
    }

    function deleteTransaction(i) {
      transactions.splice(i,1);
      saveData();
      renderTransactions();
    }

    function closeModal() {
      document.getElementById("modal").style.display = "none";
    }
    document.getElementById("addBtn").onclick = () => {
      document.getElementById("modal").style.display = "flex";
    };

    document.getElementById("borrowBtn").onclick = () => {
      renderTransactions("borrowed");
    };

    document.getElementById("notifyBell").onclick = () => {
      const today = new Date().toISOString().split("T")[0];
      let due = transactions.find(t => t.untilDate === today);
      if (due) alert(`Reminder: ${due.name} should return money today!`);
      else alert("No reminders today 🎉");
    };

    renderTransactions();
  </script>
</body>
</html>
