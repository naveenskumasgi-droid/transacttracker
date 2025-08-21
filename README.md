<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>Transaction Tracker</title>
  <style>
    body { font-family: Arial, sans-serif; margin:0; padding:0; background:#f5f5f5; color:#333; }
    header { display:flex; justify-content:space-between; align-items:center; padding:1rem; background:#fff; box-shadow:0 2px 4px rgba(0,0,0,0.1); position:relative; }
    header h1 { margin:0 auto; }
    .notification { position:absolute; right:1rem; top:1rem; cursor:pointer; }
    #reminder { display:block; font-size:0.8rem; color:red; }
    .summary { display:flex; gap:1rem; justify-content:space-around; margin:1rem; }
    .box { flex:1; padding:1rem; border-radius:8px; box-shadow:0 2px 5px rgba(0,0,0,0.2); text-align:center; font-weight:bold; }
    .green { background:#d4edda; color:#155724; }
    .red { background:#f8d7da; color:#721c24; }
    .neutral { background:#e2e3e5; color:#383d41; }
    table { width:100%; border-collapse:collapse; margin:1rem 0; background:#fff; }
    th,td { border:1px solid #ddd; padding:0.5rem; text-align:center; }
    tr.lent { background:#eaf9ea; }
    tr.borrowed { background:#fceaea; }
    .form-section { display:none; padding:1rem; background:#fff; margin:1rem; border-radius:8px; box-shadow:0 2px 4px rgba(0,0,0,0.2); }
    .form-section input, .form-section select, .form-section button { display:block; margin:0.5rem 0; padding:0.5rem; width:100%; }
    .controls { display:flex; justify-content:center; gap:1rem; margin:1rem; }
    .controls button { padding:0.8rem 1.2rem; font-size:1.2rem; border:none; border-radius:50%; cursor:pointer; }
    #filterBorrowed { background:#ffcccc; }
    #showForm { background:#cce5ff; }
  </style>
</head>
<body>
  <header>
    <h1>💰 Transaction Tracker</h1>
    <div class="date" id="currentDate"></div>
    <div class="notification">🔔 <span id="reminder"></span></div>
  </header>

  <section class="summary">
    <div class="box green">Total Lent: <span id="totalLent">0</span></div>
    <div class="box red">Total Borrowed: <span id="totalBorrowed">0</span></div>
    <div class="box neutral">Net Balance: <span id="netBalance">0</span></div>
    <div class="box neutral">Net Balance w/ Interest: <span id="netBalanceInterest">0</span></div>
  </section>

  <table id="transactionTable">
    <thead>
      <tr>
        <th>No</th><th>Name</th><th>Type</th><th>Amount</th>
        <th>Interest %</th><th>Start Date</th><th>Until Date</th>
        <th>Days Passed</th><th>Interest</th><th>Net Balance</th><th>Action</th>
      </tr>
    </thead>
    <tbody id="tableBody"></tbody>
  </table>

  <section class="form-section" id="formSection">
    <h2 id="formTitle">Add Transaction</h2>
    <form id="transactionForm">
      <input type="text" id="name" placeholder="Name" required>
      <select id="type" required>
        <option value="Lent">Lent</option>
        <option value="Borrowed">Borrowed</option>
      </select>
      <input type="number" id="amount" placeholder="Amount" required>
      <input type="number" step="0.01" id="interest" placeholder="Interest %">
      <input type="date" id="startDate" required>
      <input type="date" id="untilDate" required>
      <button type="submit">Save</button>
    </form>
  </section>

  <div class="controls">
    <button id="filterBorrowed">B</button>
    <button id="showForm">+</button>
  </div>

  <script>
    const form = document.getElementById('transactionForm');
    const tableBody = document.getElementById('tableBody');
    const filterBtn = document.getElementById('filterBorrowed');
    const showFormBtn = document.getElementById('showForm');
    const formSection = document.getElementById('formSection');
    const reminder = document.getElementById('reminder');
    const formTitle = document.getElementById('formTitle');

    let transactions = JSON.parse(localStorage.getItem('transactions')) || [];
    let showBorrowedOnly = false;
    let editIndex = null; // track editing row

    function updateDate() {
      document.getElementById('currentDate').innerText = new Date().toDateString();
    }
    updateDate();

    function calculateDays(start) {
      const s = new Date(start);
      const now = new Date();
      return Math.floor((now - s) / (1000 * 60 * 60 * 24));
    }

    function checkReminders() {
      const today = new Date().toISOString().split("T")[0];
      const due = transactions.filter(t => t.untilDate === today && t.type === "Borrowed");
      reminder.innerText = due.length > 0 ? `⚠️ ${due.length} due today` : "";
    }

    function renderTransactions() {
      tableBody.innerHTML = '';
      let totalLent = 0, totalBorrowed = 0, netBalance = 0, netWithInterest = 0;

      transactions.forEach((t, i) => {
        if (showBorrowedOnly && t.type !== 'Borrowed') return;

        const row = document.createElement('tr');
        row.className = t.type.toLowerCase();

        const daysPassed = calculateDays(t.startDate);
        const months = daysPassed / 30;
        const interestAmount = (t.amount * (parseFloat(t.interest) / 100)) * months;
        const net = t.type === 'Lent'
          ? t.amount + interestAmount
          : -(t.amount + interestAmount);

        if (t.type === 'Lent') totalLent += t.amount;
        else totalBorrowed += t.amount;

        netBalance += t.type === 'Lent' ? t.amount : -t.amount;
        netWithInterest += net;

        row.innerHTML = `
          <td>${i + 1}</td><td>${t.name}</td><td>${t.type}</td><td>${t.amount}</td>
          <td>${t.interest}</td><td>${t.startDate}</td><td>${t.untilDate}</td>
          <td>${daysPassed}</td><td>${interestAmount.toFixed(2)}</td>
          <td>${net.toFixed(2)}</td>
          <td>
            <button onclick="editTransaction(${i})">✏️</button>
            <button onclick="deleteTransaction(${i})">❌</button>
          </td>
        `;

        tableBody.appendChild(row);
      });

      document.getElementById('totalLent').innerText = totalLent;
      document.getElementById('totalBorrowed').innerText = totalBorrowed;
      document.getElementById('netBalance').innerText = netBalance.toFixed(2);
      document.getElementById('netBalanceInterest').innerText = netWithInterest.toFixed(2);

      localStorage.setItem('transactions', JSON.stringify(transactions));
      checkReminders();
    }

    function defaultSubmitHandler(e) {
      e.preventDefault();
      const data = {
        name: document.getElementById('name').value,
        type: document.getElementById('type').value,
        amount: parseFloat(document.getElementById('amount').value),
        interest: parseFloat(document.getElementById('interest').value || 0),
        startDate: document.getElementById('startDate').value,
        untilDate: document.getElementById('untilDate').value
      };
      if (editIndex !== null) {
        transactions[editIndex] = data; editIndex = null; formTitle.innerText = "Add Transaction";
      } else {
        transactions.push(data);
      }
      renderTransactions();
      form.reset(); formSection.style.display = 'none';
    }
    form.onsubmit = defaultSubmitHandler;

    function editTransaction(index) {
      const t = transactions[index];
      document.getElementById('name').value = t.name;
      document.getElementById('type').value = t.type;
      document.getElementById('amount').value = t.amount;
      document.getElementById('interest').value = t.interest;
      document.getElementById('startDate').value = t.startDate;
      document.getElementById('untilDate').value = t.untilDate;
      formSection.style.display = 'block';
      formTitle.innerText = "Edit Transaction";
      editIndex = index;
    }

    function deleteTransaction(index) { transactions.splice(index, 1); renderTransactions(); }

    filterBtn.addEventListener('click', () => { showBorrowedOnly = !showBorrowedOnly; renderTransactions(); });
    showFormBtn.addEventListener('click', () => { formSection.style.display = formSection.style.display === 'block' ? 'none' : 'block'; });

    renderTransactions();
  </script>
</body>
</html>
