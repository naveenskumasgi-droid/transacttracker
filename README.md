<!--
Paste this entire block into your README.md (inside ```html ... ```),
or save as index.html. It’s fully self-contained (HTML+CSS+JS).
-->
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1" />
  <title>Transaction Tracker</title>
  <style>
    :root{
      --bg: #f5f7fb;
      --card: #ffffff;
      --ink: #0f172a;
      --muted: #475569;
      --brand: #2563eb;
      --brand-2:#22c55e;
      --danger:#ef4444;
      --shadow: 0 8px 24px rgba(15,23,42,.08);
      --radius:16px;
    }
    *{box-sizing:border-box}
    html,body{margin:0;padding:0;background:var(--bg);color:var(--ink);font:16px/1.5 -apple-system,BlinkMacSystemFont,"Segoe UI",Roboto,Inter,system-ui,Arial}
    header{
      position:sticky; top:0; z-index:10;
      background:linear-gradient(135deg,#60a5fa,#22d3ee);
      color:#fff; padding:16px 20px; box-shadow:var(--shadow)
    }
    .row{display:flex; align-items:center; justify-content:space-between; gap:12px; flex-wrap:wrap}
    h1{margin:0; font-size:1.4rem; letter-spacing:.2px}
    .hdr-right{display:flex; align-items:center; gap:8px}
    #today{font-weight:600; background:rgba(255,255,255,.2); padding:6px 10px; border-radius:999px}
    #btnBorrowed{
      border:0; width:36px; height:36px; border-radius:999px; cursor:pointer;
      background:#fff; color:#1d4ed8; font-weight:800; box-shadow:var(--shadow)
    }
    .wrap{max-width:1100px; margin:18px auto; padding:0 14px}
    .grid{
      display:grid; gap:12px; grid-template-columns:repeat(3,minmax(0,1fr));
    }
    @media (max-width:900px){ .grid{grid-template-columns:1fr} }
    .widget{
      background:var(--card); border-radius:var(--radius); padding:16px; box-shadow:var(--shadow); position:relative; overflow:hidden
    }
    .widget h3{margin:0 0 6px; color:var(--muted); font-size:.95rem; font-weight:600}
    .widget p{margin:0; font-size:1.35rem; font-weight:800}
    .widget--blue{border-left:6px solid #2563eb}
    .widget--green{border-left:6px solid #22c55e}
    .widget--violet{border-left:6px solid #8b5cf6}

    .note{display:none; margin:14px 0; padding:12px 14px; border-radius:12px; background:#fde68a; color:#7c2d12; font-weight:600; box-shadow:var(--shadow)}
    .note.show{display:block}

    table{width:100%; border-collapse:collapse; margin-top:12px; background:var(--card); border-radius:14px; overflow:hidden; box-shadow:var(--shadow)}
    thead th{background:#e2e8f0; color:#0f172a; font-weight:700; font-size:.9rem}
    th,td{padding:12px 10px; text-align:center; border-bottom:1px solid #eef2f7}
    tbody tr:hover{background:#f8fafc}
    .chip{display:inline-block; padding:4px 8px; border-radius:999px; font-weight:700; font-size:.8rem}
    .chip.lent{background:#dcfce7; color:#166534}
    .chip.borrowed{background:#fee2e2; color:#991b1b}

    .fab{
      position:fixed; right:18px; bottom:18px; width:58px; height:58px; border-radius:999px; border:0;
      background:var(--brand-2); color:#fff; font-size:28px; cursor:pointer; box-shadow:var(--shadow)
    }

    /* Modal */
    .modal{position:fixed; inset:0; display:none; place-items:center; background:rgba(15,23,42,.45); padding:16px}
    .modal.show{display:grid}
    .card{
      width:min(460px,100%); background:var(--card); border-radius:18px; box-shadow:var(--shadow); padding:18px
    }
    .card h2{margin:0 0 8px; font-size:1.2rem}
    .grid-2{display:grid; grid-template-columns:1fr 1fr; gap:10px}
    .field{display:flex; flex-direction:column; gap:6px}
    .field label{font-size:.85rem; color:var(--muted); font-weight:600}
    input[type="text"], input[type="number"], input[type="date"], select, textarea{
      width:100%; padding:10px 12px; border-radius:12px; border:1px solid #e5e7eb; background:#fff; outline:none; transition:border .2s, box-shadow .2s;
      font:inherit
    }
    input:focus, select:focus, textarea:focus{border-color:#93c5fd; box-shadow:0 0 0 4px rgba(59,130,246,.15)}
    .actions{display:flex; gap:10px; margin-top:12px}
    .btn{flex:1; border:0; padding:12px; border-radius:12px; font-weight:700; cursor:pointer}
    .btn.primary{background:var(--brand); color:#fff}
    .btn.ghost{background:#f1f5f9}

    .del{border:0; background:var(--danger); color:#fff; padding:6px 10px; border-radius:10px; cursor:pointer}
    .muted{color:var(--muted)}
  </style>
</head>
<body>
  <header>
    <div class="row">
      <h1>💰 Transaction Tracker</h1>
      <div class="hdr-right">
        <span id="today" title="Today"></span>
        <button id="btnBorrowed" title="Toggle Borrowed view">B</button>
      </div>
    </div>
  </header>

  <div class="wrap">
    <!-- Summary widgets -->
    <div class="grid">
      <div class="widget widget--blue">
        <h3>Total Net Balance (with interest)</h3>
        <p id="sumWith">₹0.00</p>
      </div>
      <div class="widget widget--green">
        <h3>Total Net Balance (without interest)</h3>
        <p id="sumWithout">₹0.00</p>
      </div>
      <div class="widget widget--violet">
        <h3 class="muted">Hint</h3>
        <p class="muted">Click <strong>B</strong> to view only Borrowed list. Click again to return.</p>
      </div>
    </div>

    <!-- One-time notification (due today) -->
    <div id="notify" class="note"></div>

    <!-- Records table -->
    <table id="tbl">
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
          <th>Interest Accrued</th>
          <th>Total w/ Interest</th>
          <th>Action</th>
        </tr>
      </thead>
      <tbody id="tbody"></tbody>
    </table>
  </div>

  <!-- Add button -->
  <button class="fab" id="fab" aria-label="Add transaction">＋</button>

  <!-- Modal -->
  <div class="modal" id="modal">
    <div class="card">
      <h2>Add Transaction</h2>
      <form id="form">
        <div class="grid-2">
          <div class="field">
            <label for="name">Name</label>
            <input id="name" type="text" placeholder="Person / Entity" required />
          </div>
          <div class="field">
            <label for="type">Type</label>
            <select id="type" required>
              <option value="Lent">Lent</option>
              <option value="Borrowed">Borrowed</option>
            </select>
          </div>
          <div class="field">
            <label for="amount">Amount (₹)</label>
            <input id="amount" type="number" inputmode="decimal" step="0.01" min="0" placeholder="e.g., 1500.00" required />
          </div>
          <div class="field">
            <label for="rate">Interest % (monthly, decimals allowed)</label>
            <input id="rate" type="number" inputmode="decimal" step="0.1" min="0" placeholder="e.g., 5.6" />
          </div>
          <div class="field">
            <label for="from">From date</label>
            <input id="from" type="date" required />
          </div>
          <div class="field">
            <label for="until">Until date</label>
            <input id="until" type="date" />
          </div>
        </div>
        <div class="field" style="margin-top:6px">
          <label for="reason">Reason (optional)</label>
          <input id="reason" type="text" placeholder="Notes / purpose" />
        </div>
        <div class="actions">
          <button type="button" class="btn ghost" id="cancel">Cancel</button>
          <button type="submit" class="btn primary">Save</button>
        </div>
      </form>
    </div>
  </div>

  <script>
    // ===== State & storage =====
    const tbody = document.getElementById('tbody');
    const sumWith = document.getElementById('sumWith');
    const sumWithout = document.getElementById('sumWithout');
    const notify = document.getElementById('notify');
    const btnBorrowed = document.getElementById('btnBorrowed');
    const fab = document.getElementById('fab');
    const modal = document.getElementById('modal');
    const form = document.getElementById('form');
    const cancelBtn = document.getElementById('cancel');

    const todayLabel = document.getElementById('today');
    const today = new Date();
    const dayFmt = new Intl.DateTimeFormat('en-IN', { weekday:'long', day:'2-digit', month:'short', year:'numeric' });
    todayLabel.textContent = dayFmt.format(today);

    let data = JSON.parse(localStorage.getItem('transactions_v2') || '[]');
    let showBorrowedOnly = false;

    // One-time notification per day key
    const notifKey = 'notif_shown_' + today.toISOString().slice(0,10);

    // ===== Utils =====
    function monthDiff(a, b){
      // count whole months difference, never negative
      const start = new Date(a), end = new Date(b);
      if (isNaN(start) || isNaN(end)) return 0;
      let m = (end.getFullYear()-start.getFullYear())*12 + (end.getMonth()-start.getMonth());
      return Math.max(0, m);
    }
    const fmtC = n => '₹' + Number(n).toFixed(2);

    function calcInterest(amount, rate, from, until){
      if (!rate || !until) return 0;
      const months = monthDiff(from, until);
      return (amount * (rate/100)) * months; // simple monthly interest
    }

    function persist(){ localStorage.setItem('transactions_v2', JSON.stringify(data)); }

    // ===== Rendering =====
    function render(){
      tbody.innerHTML = '';
      let netWith = 0, netWithout = 0;
      let dueTodayName = null;

      data.forEach((t, idx) => {
        if (showBorrowedOnly && t.type !== 'Borrowed') return;

        const interest = calcInterest(t.amount, t.rate, t.from, t.until);
        const totalWith = t.amount + interest;

        if (t.type === 'Lent'){ netWith += totalWith; netWithout += t.amount; }
        else { netWith -= totalWith; netWithout -= t.amount; }

        const tr = document.createElement('tr');
        tr.innerHTML = `
          <td>${idx+1}</td>
          <td>${escapeHtml(t.name)}</td>
          <td><span class="chip ${t.type==='Lent'?'lent':'borrowed'}">${t.type}</span></td>
          <td>${fmtC(t.amount)}</td>
          <td>${(t.rate || 0).toFixed(1)}%</td>
          <td>${t.from || '-'}</td>
          <td>${t.until || '-'}</td>
          <td>${escapeHtml(t.reason || '-')}</td>
          <td>${fmtC(interest)}</td>
          <td>${fmtC(totalWith)}</td>
          <td><button class="del" aria-label="Delete" onclick="delRow(${idx})">Delete</button></td>
        `;
        tbody.appendChild(tr);

        // Due today notification (Borrowed only)
        if (!localStorage.getItem(notifKey) && t.type==='Borrowed' && t.until){
          const d = new Date(t.until);
          if (d.toDateString() === today.toDateString() && !dueTodayName){
            dueTodayName = t.name;
          }
        }
      });

      sumWith.textContent = fmtC(netWith);
      sumWithout.textContent = fmtC(netWithout);

      if (dueTodayName){
        notify.textContent = `Reminder: ${dueTodayName} is due today.`;
        notify.classList.add('show');
        // mark shown once per day
        localStorage.setItem(notifKey,'1');
      } else {
        // show previously-shown note? keep hidden unless there is a new one
        notify.classList.remove('show');
      }
    }

    // escape HTML for safe injection
    function escapeHtml(s){
      return String(s).replace(/[&<>"']/g, m => ({'&':'&amp;','<':'&lt;','>':'&gt;','"':'&quot;',"'":'&#39;'}[m]));
    }

    // ===== Interactions =====
    window.delRow = function(i){
      data.splice(i,1);
      persist(); render();
    }

    fab.addEventListener('click', ()=> modal.classList.add('show'));
    cancelBtn.addEventListener('click', ()=> modal.classList.remove('show'));
    modal.addEventListener('click', (e)=>{ if(e.target===modal) modal.classList.remove('show'); });

    btnBorrowed.addEventListener('click', ()=>{
      showBorrowedOnly = !showBorrowedOnly;
      btnBorrowed.style.background = showBorrowedOnly ? '#1d4ed8' : '#fff';
      btnBorrowed.style.color = showBorrowedOnly ? '#fff' : '#1d4ed8';
      render();
    });

    form.addEventListener('submit', (e)=>{
      e.preventDefault();
      const name = document.getElementById('name').value.trim();
      const type = document.getElementById('type').value;
      const amount = parseFloat(document.getElementById('amount').value) || 0;
      const rate = parseFloat(document.getElementById('rate').value); // allow decimals like 5.6
      const from = document.getElementById('from').value || '';
      const until = document.getElementById('until').value || '';
      const reason = document.getElementById('reason').value.trim();

      data.push({name, type, amount, rate: isNaN(rate)?0:rate, from, until, reason});
      persist();
      form.reset();
      modal.classList.remove('show');
      render();
    });

    // Init
    render();
  </script>
</body>
</html>
