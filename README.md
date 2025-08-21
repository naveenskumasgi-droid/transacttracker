<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>Transaction Tracker</title>
  <style>
    body {
      font-family: "Segoe UI", Tahoma, sans-serif;
      background: linear-gradient(120deg, #a1c4fd, #c2e9fb);
      margin: 0;
      padding: 0;
      color: #333;
    }
    header {
      background: #4a90e2;
      color: white;
      text-align: center;
      padding: 15px;
      font-size: 20px;
      font-weight: bold;
      position: relative;
    }
    header .date {
      font-size: 14px;
      margin-top: 5px;
      opacity: 0.9;
    }
    .container {
      width: 95%;
      max-width: 1100px;
      margin: 20px auto;
    }
    .summary {
      display: grid;
      grid-template-columns: repeat(auto-fit, minmax(180px, 1fr));
      gap: 15px;
      margin-bottom: 20px;
    }
    .card {
      background: white;
      border-radius: 12px;
      box-shadow: 0 3px 8px rgba(0,0,0,0.15);
      padding: 15px;
      text-align: center;
      font-weight: bold;
      transition: transform 0.2s;
    }
    .card:hover { transform: translateY(-3px); }
    .card h3 {
      margin: 8px 0;
      font-size: 16px;
      color: #555;
    }
    .card p {
      font-size: 18px;
      color: #000;
    }
    table {
      width: 100%;
      border-collapse: collapse;
      background: white;
      border-radius: 12px;
      overflow: hidden;
      box-shadow: 0 3px 8px rgba(0,0,0,0.1);
    }
    th, td {
      border-bottom: 1px solid #eee;
      padding: 12px;
      text-align: center;
      font-size: 14px;
    }
    th {
      background: #4a90e2;
      color: white;
      font-size: 15px;
    }
    tr:hover { background: #f9f9f9; }
    .floating-btn {
      position: fixed;
      bottom: 20px;
      right: 20px;
      display: flex;
      flex-direction: column;
      gap: 10px;
      z-index: 9999;
    }
    .btn {
      background: #4a90e2;
      color: white;
      padding: 15px;
      border-radius: 50%;
      border: none;
      font-size: 20px;
      cursor: pointer;
      box-shadow: 0 4px 8px rgba(0,0,0,0.2);
      transition: 0.3s;
    }
    .btn:hover { background: #357ABD; }
    .modal {
      display: none;
      position: fixed;
      z-index: 10000; /* Fixed overlap issue */
      left: 0;
      top: 0;
      width: 100%;
      height: 100%;
      overflow: auto;
      background: rgba(0,0,0,0.5);
    }
    .modal-content {
      background: white;
      margin: 10% auto;
      padding: 20px;
      border-radius: 12px;
      width: 90%;
      max-width: 500px;
    }
    .modal h2 {
      text-align: center;
      margin-bottom: 15px;
    }
    .modal input, .modal select {
      width: 100%;
      padding: 10px;
      margin: 8px 0;
      border: 1px solid #ccc;
      border-radius: 8px;
      font-size: 14px;
    }
    .modal button {
      width: 100%;
      padding: 10px;
      background: #4a90e2;
      border: none;
      color: white;
      font-size: 16px;
      border-radius: 8px;
      margin-top: 10px;
      cursor: pointer;
    }
    .close {
      float: right;
      font-size: 18px;
      cursor: pointer;
      color: red;
    }
    .notification {
      background: #ffcccc;
      padding: 10px;
      border-radius: 8px;
      text-align: center;
      margin-bottom: 10px;
      color: #b30000;
      font-weight: bold;
    }
  </style>
</head>
<body>
  <header>
    Transaction Tracker
    <div class="date" id="todayDate"></div>
  </header>

  <div class="container">
    <div id="notification"></div>
    <div class="summary">
      <div class="card"><h3>Total Net Balance</h3><p id="netBalance">₹0</p></div>
      <div class="card"><h3>Total Net Balance (With Interest)</h3><p id="netBalanceInterest">₹0</p></div>
      <div class="card"><h3>Total Borrowed</h3><p id="totalBorrowed">₹0</p></div>
      <div class="card"><h3>Total Lent</h3><p id="totalLent">₹0</p></div>
    </div>

    <table id="transactionTable">
      <thead>
        <tr>
          <th>Sr. No</th>
          <th>Name</th>
          <th>Type</th>
          <th>Amount</th>
          <th>Interest %</th>
          <th>Start Date</th>
          <th>Return Date</th>
          <th>Reason</th>
          <th>Interest Return</th>
          <th>Action</th>
        </tr>
      </thead>
      <tbody></tbody>
    </table>
  </div>

  <div class="floating-btn">
    <button class="btn" id="borrowedBtn">B</button>
    <button class="btn" id="addBtn">+</button>
  </div>

  <!-- Modal -->
  <div id="transactionModal" class="modal">
    <div class="modal-content">
      <span class="close" id="closeModal">&times;</span>
      <h2>Add Transaction</h2>
      <input type="text" id="name" placeholder="Name" required>
      <select id="type">
        <option value="lent">Lent</option>
        <option value="borrowed">Borrowed</option>
      </select>
      <input type="number" id="amount" placeholder="Amount" required>
      <input type="number" step="0.1" id="interest" placeholder="Interest Rate (%)">
      <input type="date" id="fromDate" required>
      <input type="date" id="untilDate" required>
      <input type="text" id="reason" placeholder="Reason">
      <button id="saveTransaction">Save</button>
    </div>
  </div>

  <script>
    let transactions = JSON.parse(localStorage.getItem("transactions")) || [];

    const today = new Date();
    const todayString = today.toLocaleDateString("en-IN", { weekday: 'long', year: 'numeric', month: 'long', day: 'numeric' });
    document.getElementById("todayDate").innerText = todayString;

    function saveTransactions() {
      localStorage.setItem("transactions", JSON.stringify(transactions));
    }

    function calculateInterest(amount, rate, from, until) {
      if (!rate) return 0;
      const start = new Date(from);
      const end = new Date(until);
      const months = (end.getFullYear() - start.getFullYear()) * 12 + (end.getMonth() - start.getMonth());
      return (amount * (rate / 100) * months).toFixed(2);
    }

    function updateTable(filterBorrowed = false) {
      const tbody = document.querySelector("#transactionTable tbody");
      tbody.innerHTML = "";
      let totalBorrowed = 0, totalLent = 0, netBalance = 0, netBalanceInterest = 0;

      transactions.forEach((t, index) => {
        if (filterBorrowed && t.type !== "borrowed") return;

        const interestReturn = calculateInterest(t.amount, parseFloat(t.interest), t.fromDate, t.untilDate);
        const row = `<tr>
          <td>${index + 1}</td>
          <td>${t.name}</td>
          <td>${t.type}</td>
          <td>₹${t.amount}</td>
          <td>${t.interest || 0}%</td>
          <td>${t.fromDate}</td>
          <td>${t.untilDate}</td>
          <td>${t.reason || "-"}</td>
          <td>₹${interestReturn}</td>
          <td><button onclick="deleteTransaction(${index})">❌</button></td>
        </tr>`;
        tbody.insertAdjacentHTML("beforeend", row);

        if (t.type === "borrowed") {
          totalBorrowed += parseFloat(t.amount);
          netBalance -= parseFloat(t.amount);
          netBalanceInterest -= (parseFloat(t.amount) + parseFloat(interestReturn));
        } else {
          totalLent += parseFloat(t.amount);
          netBalance += parseFloat(t.amount);
          netBalanceInterest += (parseFloat(t.amount) + parseFloat(interestReturn));
        }
      });

      document.getElementById("totalBorrowed").innerText = `₹${totalBorrowed}`;
      document.getElementById("totalLent").innerText = `₹${totalLent}`;
      document.getElementById("netBalance").innerText = `₹${netBalance}`;
      document.getElementById("netBalanceInterest").innerText = `₹${netBalanceInterest}`;
    }

    function deleteTransaction(index) {
      transactions.splice(index, 1);
      saveTransactions();
      updateTable();
    }

    document.getElementById("addBtn").onclick = () => {
      document.getElementById("transactionModal").style.display = "block";
    };
    document.getElementById("closeModal").onclick = () => {
      document.getElementById("transactionModal").style.display = "none";
    };

    document.getElementById("saveTransaction").onclick = () => {
      const t = {
        name: document.getElementById("name").value,
        type: document.getElementById("type").value,
        amount: parseFloat(document.getElementById("amount").value),
        interest: document.getElementById("interest").value,
        fromDate: document.getElementById("fromDate").value,
        untilDate: document.getElementById("untilDate").value,
        reason: document.getElementById("reason").value
      };
      transactions.push(t);
      saveTransactions();
      updateTable();
      document.getElementById("transactionModal").style.display = "none";
    };

    document.getElementById("borrowedBtn").onclick = () => {
      updateTable(true);
    };

    // Notifications for due returns
    transactions.forEach(t => {
      if (t.untilDate === today.toISOString().split('T')[0]) {
        document.getElementById("notification").innerHTML = `<div class="notification">${t.name} should return today (${t.untilDate})</div>`;
      }
    });

    updateTable();
  </script>
</body>
</html>
