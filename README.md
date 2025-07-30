<!DOCTYPE html>
<html lang="it">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>Mutuo Smart - Antonino Bonfiglio</title>

  <!-- Icona App -->
  <link rel="icon" href="https://upload.wikimedia.org/wikipedia/commons/thumb/5/57/Home_icon.svg/1200px-Home_icon.svg.png">
  <link rel="manifest" href="manifest.json">
  <meta name="theme-color" content="#f57c00">

  <!-- PDF Tool -->
  <script src="https://cdnjs.cloudflare.com/ajax/libs/html2pdf.js/0.10.1/html2pdf.bundle.min.js"></script>

  <style>
    body { font-family: sans-serif; background: #f5f5f5; padding: 1em; margin: 0; }
    .container { max-width: 600px; margin: auto; background: white; padding: 2em; border-radius: 10px; box-shadow: 0 0 10px rgba(0,0,0,0.2); }
    input, select, button { width: 100%; padding: 0.8em; margin-top: 1em; font-size: 1em; }
    table { width: 100%; border-collapse: collapse; margin-top: 1em; }
    th, td { border: 1px solid #ddd; padding: 0.5em; text-align: center; }
    footer { margin-top: 2em; text-align: center; font-size: 0.9em; color: #777; }
  </style>
</head>
<body>
  <div class="container">
    <h2>🏠 Mutuo Smart</h2>

    <label>Importo (€):</label>
    <input type="number" id="amount">

    <label>Durata (anni):</label>
    <input type="number" id="years">

    <label>Tipo di tasso:</label>
    <select id="type">
      <option value="fisso">Tasso Fisso</option>
      <option value="variabile">Tasso Variabile</option>
    </select>

    <label>Tasso iniziale (%):</label>
    <input type="number" step="0.01" id="interest">

    <button onclick="calcolaMutuo()">Calcola Mutuo</button>
    <button onclick="window.print()">🖨️ Stampa</button>
    <button onclick="esportaPDF()">📄 Esporta in PDF</button>

    <h3 id="rata"></h3>
    <div id="ammortamento"></div>

    <footer>Creato da <strong>Antonino Bonfiglio</strong><br>Aggiungilo alla <strong>Schermata Home</strong> dal menu del browser!</footer>
  </div>

  <!-- Service Worker -->
  <script>
    if ('serviceWorker' in navigator) {
      navigator.serviceWorker.register('sw.js')
        .then(reg => console.log('✅ Service Worker registrato'))
        .catch(err => console.error('❌ Errore nel Service Worker:', err));
    }

    function calcolaMutuo() {
      const amount = parseFloat(document.getElementById('amount').value);
      const years = parseInt(document.getElementById('years').value);
      const type = document.getElementById('type').value;
      let interest = parseFloat(document.getElementById('interest').value) / 100;
      const months = years * 12;

      if (!amount || !interest || !years) {
        document.getElementById('rata').textContent = "⚠️ Valori non validi.";
        document.getElementById('ammortamento').innerHTML = "";
        return;
      }

      let rate = interest / 12;
      let monthly = (amount * rate) / (1 - Math.pow(1 + rate, -months));
      document.getElementById('rata').textContent = `Rata Mensile: € ${monthly.toFixed(2)}`;

      let output = "<table><tr><th>Mese</th><th>Rata</th><th>Interessi</th><th>Capitale</th><th>Residuo</th></tr>";
      let balance = amount;

      for (let i = 1; i <= months; i++) {
        if (type === "variabile" && i % 12 === 0) rate += 0.0005;
        const interestPart = balance * rate;
        const capitalPart = monthly - interestPart;
        balance -= capitalPart;
        output += `<tr><td>${i}</td><td>€ ${monthly.toFixed(2)}</td><td>€ ${interestPart.toFixed(2)}</td><td>€ ${capitalPart.toFixed(2)}</td><td>€ ${balance.toFixed(2)}</td></tr>`;
      }

      output += "</table>";
      document.getElementById('ammortamento').innerHTML = output;
    }

    function esportaPDF() {
      const element = document.getElementById("ammortamento");
      const opt = {
        margin: 0.5,
        filename: 'Piano_Mutuo_Antonino.pdf',
        image: { type: 'jpeg', quality: 0.98 },
        html2canvas: { scale: 2 },
        jsPDF: { unit: 'cm', format: 'a4', orientation: 'portrait' }
      };
      html2pdf().set(opt).from(element).save();
    }
  </script>
</body>
</html>
