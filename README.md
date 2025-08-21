# transacttracker
a webiste to track ur transaction
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1.0" />
  <title>Transaction Tracker</title>
  <style>
    body {
      margin: 0;
      font-family: "Segoe UI", Roboto, Arial, sans-serif;
      background: linear-gradient(135deg, #f8fafc, #e0f2fe);
      color: #1e293b;
    }

    header {
      background: linear-gradient(90deg, #3b82f6, #06b6d4);
      color: white;
      padding: 1rem 2rem;
      text-align: center;
      border-radius: 0 0 20px 20px;
      box-shadow: 0 4px 10px rgba(0,0,0,0.1);
    }

    header h1 {
      margin: 0;
      font-size: 1.8rem;
    }

    .summary {
      display: flex;
      justify-content: space-around;
      margin: 1.5rem auto;
      max-width: 900px;
    }

    .card {
      flex: 1;
      margin: 0 0.5rem;
      background: white;
      border-radius: 16px;
      padding: 1rem;
      text-align: center;
      box-shadow: 0 4px 8px rgba(0,0,0,0.1);
      transition: transform 0.2s;
    }

    .card:hover {
      transform: translateY(-3px);
    }

    .card h2 {
      margin: 0.2rem 0;
      font-size: 1.2rem;
      color: #475569;
    }

    .card p {
      font-size: 1.3rem;
      font-weight: bold;
      color: #111827;
    }

    .container {
      max-width: 1000px;
      margin: auto;
      padding: 1rem;
    }

    .add-btn, .search-btn {
      position: fixed;
      bottom: 20px;
      width: 55px;
      height: 55px;
      border-radius: 50%;
      border: none;
      font-size: 24px;
      color: white;
      cursor: pointer;
      box-shadow: 0 4px 12px rgba(0,0,0,0.25);
    }

    .add-btn {
      right: 20px;
      background: #22c55e;
    }

    .search-btn {
      right: 90px;
      background: #3b82f6;
    }

    table {
      width: 100%;
      border-collapse: collapse;
      margin-top: 1rem;
      background: white;
      border-radius: 16px;
      overflow: hidden;
      box-shadow: 0 4px 8px rgba(0,0,0,0.08);
    }

    th, td {
      padding: 12px;
      text-align: center;
      border-bottom: 1px solid #e2e8f0;
    }

    th {
      background: #f1f5f9;
      color: #334155;
    }

    tr:hover {
      background: #f9fafb;
    }

    .delete-btn {
      background: #ef4444;
      border: none;
      color: white;
      padding: 6px 12px;
      border-radius: 8px;
      cursor: pointer;
      font-size: 0.9rem;
    }

    .modal {
      display: none;
      position: fixed;
      top: 0; left: 0;
      width: 100%; height: 100%;
      background: rgba(0,0,0,0.5);
      justify-content: center;
      align-items: center;
    }

    .modal-content {
      background: white;
      padding: 2rem;
      border-radius: 20px;
      width: 90%;
      max-width: 400px;
      box-shadow: 0 8px 16px rgba(0,0,0,0.2);
    }

    .modal-content h2 {
      margin-top: 0;
    }

    .modal-content input {
      width: 100%;
      padding: 0.8rem;
      margin: 0.6rem 0;
      border: 1px solid #cbd5e1;
      border-radius: 10px;
      font-size: 1rem;
    }

    .modal-content button {
      margin-top: 1rem;
      padding: 0.8rem;
      width: 100%;
      border: none;
      border-radius: 10px;
      background: #3b82f6;
      color: white;
      font-size: 1rem;
      cursor: pointer;
    }

    .modal-content button:hover {
      background: #2563eb;
    }
  </style>
</head>
<body>
  <header>
    <h1>📊 Transaction Tracker</h1>
    <p>Track, calculate interest & manage money with ease</p>
  </header>

  <section class="summary">
    <div class="card">
      <h2>Total Lent</h2>
      <p id="totalLent">₹0</p>
    </div>
    <div class="card">
      <h2>Total Borrowed</h2>
      <p id="totalBorrowed">₹0</p>
    </div>
    <div class="card">
      <h2>Net Balance</h2>
      <p id="netBalance">₹0</p>
    </div>
  </section>

  <div class="container">
    <table id="transactionTable">
      <thead>
        <tr>
          <th>Name</th>
          <th>Type</th>
          <th>Amount</th>
          <th>Interest</th>
          <th>From</th>
          <th>Until</th>
          <th>Reason</th>
          <th>Return (w/ Interest)</th>
          <th>Action</th>
        </tr>
      </thead>
      <tbody></tbody>
    </table>
  </div>

  <button class="search-btn">🔍</button>
  <button class="add-btn">＋</button>

  <!-- Modal -->
  <div class="modal" id="modal">
    <div class="modal-content">
      <h2>Add Transaction</h2>
      <input type="text" id="name" placeholder="Name" />
      <input type="number" id="amount" placeholder="Amount (₹)" />
      <input type="number" id="interest" placeholder="Interest (% per month)" />
      <input type="date" id="fromDate" />
      <input type="date" id="untilDate" />
      <input type="text" id="reason" placeholder="Reason" />
      <select id="type">
        <option value="lent">Lent</option>
        <option value="borrowed">Borrowed</option>
      </select>
      <button onclick="addTransaction()">Save</button>
    </div>
  </div>

  <script>
    const modal = document.getElementById("modal");
    const addBtn = document.querySelector(".add-btn");
    const tbody = document.querySelector("#transactionTable tbody");

    let transactions = JSON.parse(localStorage.getItem("transactions")) || [];

    function render() {
      tbody.innerHTML = "";
      let totalLent = 0, totalBorrowed = 0;

      transactions.forEach((t, i) => {
        let from = new Date(t.fromDate);
        let until = new Date(t.untilDate);
        let months = (until.getFullYear() - from.getFullYear())*12 + (until.getMonth() - from.getMonth());
        months = Math.max(months, 0);

        let interestAmount = t.interest > 0 ? (t.amount * (t.interest/100) * months) : 0;
        let returnAmount = t.amount + interestAmount;

        if (t.type === "lent") totalLent += returnAmount;
        else totalBorrowed += returnAmount;

        tbody.innerHTML += `
          <tr>
            <td>${t.name}</td>
            <td>${t.type}</td>
            <td>₹${t.amount}</td>
            <td>${t.interest}%</td>
            <td>${t.fromDate}</td>
            <td>${t.untilDate}</td>
            <td>${t.reason}</td>
            <td>₹${returnAmount.toFixed(2)}</td>
            <td><button class="delete-btn" onclick="deleteTransaction(${i})">Delete</button></td>
          </tr>`;
      });

      document.getElementById("totalLent").textContent = "₹" + totalLent.toFixed(2);
      document.getElementById("totalBorrowed").textContent = "₹" + totalBorrowed.toFixed(2);
      document.getElementById("netBalance").textContent = "₹" + (totalLent - totalBorrowed).toFixed(2);

      localStorage.setItem("transactions", JSON.stringify(transactions));
    }

    function addTransaction() {
      let t = {
        name: document.getElementById("name").value,
        amount: parseFloat(document.getElementById("amount").value),
        interest: parseFloat(document.getElementById("interest").value) || 0,
        fromDate: document.getElementById("fromDate").value,
        untilDate: document.getElementById("untilDate").value,
        reason: document.getElementById("reason").value,
        type: document.getElementById("type").value
      };
      transactions.push(t);
      modal.style.display = "none";
      render();
    }

    function deleteTransaction(i) {
      transactions.splice(i, 1);
      render();
    }

    addBtn.onclick = () => modal.style.display = "flex";
    window.onclick = e => { if (e.target == modal) modal.style.display = "none"; };

    render();
  </script>
</body>
</html>
