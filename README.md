-------- Kumasgi---------
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
