--------- K U M A S G I -----------
<html>
<head>
  <meta charset="UTF-8">
  <title>Transaction Tracker</title>
  <style>
    body {
      font-family: Arial, sans-serif;
      margin: 0;
      background: #f4f4f9;
      color: #333;
      transition: background 0.3s, color 0.3s;
    }
    body.dark {
      background: #181818;
      color: #eee;
    }
    header {
      display: flex;
      justify-content: space-between;
      align-items: center;
      padding: 15px;
      background: #6200ea;
      color: white;
      font-size: 20px;
      box-shadow: 0 2px 6px rgba(0,0,0,0.3);
    }
    header .right {
      display: flex;
      gap: 15px;
      align-items: center;
    }
    .summary {
      display: flex;
      justify-content: space-around;
      margin: 20px;
      gap: 15px;
      flex-wrap: wrap;
    }
    .box {
      flex: 1;
      min-width: 200px;
      padding: 15px;
      border-radius: 12px;
      box-shadow: 0 4px 10px rgba(0,0,0,0.2);
      text-align: center;
      font-weight: bold;
      color: white;
    }
    .lent { background: #2e7d32; }
    .borrowed { background: #c62828; }
    .net { background: #1565c0; }
    .table-container {
      margin: 20px;
      overflow-x: auto;
    }
    table {
      width: 100%;
      border-collapse: collapse;
      border-radius: 12px;
      overflow: hidden;
      box-shadow: 0 4px 10px rgba(0,0,0,0.1);
    }
    th, td {
      padding: 12px;
      text-align: center;
      border-bottom: 1px solid #ddd;
    }
    tr:nth-child(even) { background: #f9f9f9; }
    body.dark tr:nth-child(even) { background: #2a2a2a; }
    th {
      background: #6200ea;
      color: white;
      cursor: pointer;
    }
    .actions button {
      background: none;
      border: none;
      cursor: pointer;
      font-size: 18px;
    }
    .fab {
      position: fixed;
      bottom: 20px;
      right: 20px;
      display: flex;
      flex-direction: column;
      gap: 10px;
    }
    .fab button {
      width: 50px;
      height: 50px;
      border-radius: 50%;
      border: none;
      background: #6200ea;
      color: white;
      font-size: 24px;
      cursor: pointer;
      box-shadow: 0 4px 10px rgba(0,0,0,0.3);
    }
    canvas { margin: 20px auto; display: block; max-width: 600px; }
  </style>
</head>
<body>
  <header>
    <div>📊 Transaction Tracker</div>
    <div class="right">
      <div id="today"></div>
      <button onclick="toggleDark()">🌙</button>
      <span id="notification">🔔</span>
    </div>
  </header>

  <div class="summary">
    <div class="box lent">Total Lent: ₹<span id="totalLent">0</span></div>
    <div class="box borrowed">Total Borrowed: ₹<span id="totalBorrowed">0</span></div>
    <div class="box net">Net Balance: ₹<span id="netBalance">0</span></div>
  </div>

  <div class="table-container">
    <table id="transactionTable">
      <thead>
        <tr>
          <th>#</th>
          <th>Name</th>
          <th>Type</th>
          <th>Amount</th>
          <th>Interest %</th>
          <th>From</th>
          <th>Until</th>
          <th>Reason</th>
          <th>Action</th>
        </tr>
      </thead>
      <tbody></tbody>
    </table>
  </div>

  <canvas id="lineChart"></canvas>

  <div class="fab">
    <button onclick="showBorrowed()">B</button>
    <button onclick="addTransaction()">＋</button>
  </div>

  <script src="https://cdn.jsdelivr.net/npm/chart.js"></script>
  <script>
    let transactions = JSON.parse(localStorage.getItem('transactions') || '[]');
    let dark = false;

    function toggleDark() {
      document.body.classList.toggle('dark');
      dark = !dark;
    }

    function renderTable(list = transactions) {
      let tbody = document.querySelector("#transactionTable tbody");
      tbody.innerHTML = "";
      let totalLent = 0, totalBorrowed = 0;
      list.forEach((t, i) => {
        let interestAmount = t.amount * (t.interest/100) * monthsBetween(t.from, t.until);
        let row = `<tr>
          <td>${i+1}</td>
          <td>${t.name}</td>
          <td>${t.type}</td>
          <td>₹${t.amount.toFixed(2)}</td>
          <td>${t.interest}%</td>
          <td>${t.from}</td>
          <td>${t.until}</td>
          <td>${t.reason}</td>
          <td class="actions"><button onclick="deleteTx(${i})">🗑️</button></td>
        </tr>`;
        tbody.innerHTML += row;
        if(t.type==="Lent") totalLent += t.amount + interestAmount;
        if(t.type==="Borrowed") totalBorrowed += t.amount + interestAmount;
      });
      document.getElementById("totalLent").textContent = totalLent.toFixed(2);
      document.getElementById("totalBorrowed").textContent = totalBorrowed.toFixed(2);
      document.getElementById("netBalance").textContent = (totalLent-totalBorrowed).toFixed(2);
      drawChart();
    }

    function monthsBetween(start, end) {
      let d1 = new Date(start), d2 = new Date(end);
      return Math.max(0,(d2.getFullYear()-d1.getFullYear())*12+(d2.getMonth()-d1.getMonth()));
    }

    function addTransaction() {
      let name = prompt("Name:");
      let type = prompt("Type (Lent/Borrowed):");
      let amount = parseFloat(prompt("Amount:"));
      let interest = parseFloat(prompt("Interest % (e.g. 5.6):")) || 0;
      let from = prompt("From Date (YYYY-MM-DD):");
      let until = prompt("Until Date (YYYY-MM-DD):");
      let reason = prompt("Reason:");
      transactions.push({name,type,amount,interest,from,until,reason});
      localStorage.setItem('transactions',JSON.stringify(transactions));
      renderTable();
    }

    function deleteTx(i) {
      transactions.splice(i,1);
      localStorage.setItem('transactions',JSON.stringify(transactions));
      renderTable();
    }

    function showBorrowed() {
      let borrowed = transactions.filter(t=>t.type==="Borrowed");
      renderTable(borrowed);
      setTimeout(()=>renderTable(),3000); // auto reset after 3 sec
    }

    function drawChart() {
      let ctx = document.getElementById("lineChart").getContext("2d");
      if(window.myChart) window.myChart.destroy();
      let labels = transactions.map((t,i)=>i+1);
      let lentData = transactions.map(t=>t.type==="Lent"?t.amount:0);
      let borrowedData = transactions.map(t=>t.type==="Borrowed"?t.amount:0);
      window.myChart = new Chart(ctx,{
        type:"line",
        data:{
          labels,
          datasets:[
            {label:"Lent",data:lentData,borderColor:"green",fill:false},
            {label:"Borrowed",data:borrowedData,borderColor:"red",fill:false}
          ]
        }
      });
    }

    function showToday() {
      let d = new Date();
      document.getElementById("today").textContent = d.toDateString();
      transactions.forEach(t=>{
        if(new Date(t.until).toDateString()===d.toDateString()) {
          document.getElementById("notification").textContent = "🔔 Return due: "+t.name;
        }
      });
    }

    showToday();
    renderTable();
  </script>
</body>
</html>
