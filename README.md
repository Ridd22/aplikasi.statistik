<!DOCTYPE html>
<html lang="id">
<head>
  <meta charset="UTF-8">
  <title>📊 Aplikasi Statistika</title>
  <meta name="viewport" content="width=device-width, initial-scale=1">
  <script src="https://cdn.jsdelivr.net/npm/chart.js"></script>
  <script src="https://cdnjs.cloudflare.com/ajax/libs/html2pdf.js/0.10.1/html2pdf.bundle.min.js"></script>
  <style>
    body {
      font-family: 'Segoe UI', sans-serif;
      margin: 0;
      background: #f7f9fc;
      color: #333;
    }
    header {
      background: linear-gradient(45deg, #007bff, #00bfff);
      color: white;
      padding: 20px;
      text-align: center;
      box-shadow: 0 2px 5px rgba(0,0,0,0.1);
    }
    header h1 {
      margin: 0;
    }
    .container {
      max-width: 900px;
      margin: auto;
      padding: 30px;
      background: #ffffff;
      margin-top: 30px;
      margin-bottom: 50px;
      border-radius: 12px;
      box-shadow: 0 0 15px rgba(0,0,0,0.05);
    }
    input {
      width: 100%;
      padding: 12px;
      font-size: 16px;
      margin-top: 10px;
      margin-bottom: 20px;
      border: 1px solid #ccc;
      border-radius: 8px;
    }
    .btn-group {
      display: flex;
      gap: 10px;
      flex-wrap: wrap;
      margin-bottom: 20px;
    }
    button {
      flex: 1;
      padding: 12px;
      font-size: 16px;
      background: #007bff;
      color: white;
      border: none;
      border-radius: 8px;
      cursor: pointer;
      transition: 0.2s ease-in-out;
    }
    button:hover {
      background: #0056b3;
    }
    table {
      width: 100%;
      border-collapse: collapse;
      margin-top: 20px;
    }
    th, td {
      text-align: left;
      padding: 12px;
      border-bottom: 1px solid #e0e0e0;
    }
    th {
      background-color: #f0f0f0;
    }
    #myChart {
      margin-top: 30px;
    }
    footer {
      text-align: center;
      padding: 20px;
      background: #f0f0f0;
      color: #555;
      font-size: 14px;
    }
  </style>
</head>
<body>

<header>
  <h1>📊 Aplikasi Statistika Interaktif</h1>
  <p>Visualisasi & Analisis Data Secara Real-Time</p>
</header>

<div class="container" id="pdfArea">
  <label for="dataInput"><strong>Masukkan data (pisahkan dengan koma):</strong></label>
  <input type="text" id="dataInput" placeholder="Contoh: 5, 3, 8, 5, 10">

  <div class="btn-group">
    <button onclick="hitungStatistika()">Hitung</button>
    <button onclick="downloadPDF()">Export ke PDF</button>
    <button onclick="resetData()">Reset</button>
  </div>

  <div id="output"></div>
  <canvas id="myChart"></canvas>
</div>

<footer>
  &copy; 2025 Aplikasi Statistika oleh Anda. Dibuat dengan ❤️ untuk edukasi.
</footer>

<script>
function hitungStatistika() {
  const input = document.getElementById('dataInput').value;
  let data = input.split(',').map(x => parseFloat(x.trim())).filter(x => !isNaN(x));
  if (data.length === 0) {
    alert("Masukkan data numerik yang valid.");
    return;
  }

  let original = [...data];
  let sorted = [...data].sort((a, b) => a - b);
  let n = sorted.length;

  let mean = data.reduce((a, b) => a + b, 0) / n;
  let median = (n % 2 === 0) ? (sorted[n/2 - 1] + sorted[n/2]) / 2 : sorted[Math.floor(n/2)];

  let freq = {};
  sorted.forEach(n => freq[n] = (freq[n] || 0) + 1);
  let maxFreq = Math.max(...Object.values(freq));
  let modus = Object.keys(freq).filter(k => freq[k] === maxFreq);
  modus = maxFreq === 1 ? ["(Tidak ada modus)"] : modus;

  let variance = data.reduce((sum, x) => sum + Math.pow(x - mean, 2), 0) / n;
  let stdDev = Math.sqrt(variance);
  let range = sorted[n - 1] - sorted[0];

  function medianOf(arr) {
    let len = arr.length;
    return len % 2 === 0 ? (arr[len/2 - 1] + arr[len/2]) / 2 : arr[Math.floor(len/2)];
  }
  let lowerHalf = sorted.slice(0, Math.floor(n/2));
  let upperHalf = sorted.slice(Math.ceil(n/2));
  let q1 = medianOf(lowerHalf);
  let q3 = medianOf(upperHalf);
  let iqr = q3 - q1;
  let outliers = data.filter(x => x < q1 - 1.5 * iqr || x > q3 + 1.5 * iqr);

  document.getElementById('output').innerHTML = `
    <h3>📑 Ringkasan Statistika</h3>
    <table>
      <tr><th>Jenis</th><th>Nilai</th></tr>
      <tr><td>Data Asli</td><td>${original.join(', ')}</td></tr>
      <tr><td>Data Terurut</td><td>${sorted.join(', ')}</td></tr>
      <tr><td>Mean</td><td>${mean.toFixed(2)}</td></tr>
      <tr><td>Median</td><td>${median}</td></tr>
      <tr><td>Modus</td><td>${modus.join(', ')}</td></tr>
      <tr><td>Range</td><td>${range}</td></tr>
      <tr><td>Varians</td><td>${variance.toFixed(2)}</td></tr>
      <tr><td>Standar Deviasi</td><td>${stdDev.toFixed(2)}</td></tr>
      <tr><td>Kuartil 1 (Q1)</td><td>${q1}</td></tr>
      <tr><td>Kuartil 3 (Q3)</td><td>${q3}</td></tr>
      <tr><td>IQR (Q3 - Q1)</td><td>${iqr}</td></tr>
      <tr><td>Outlier</td><td>${outliers.length > 0 ? outliers.join(', ') : '(Tidak ada)'}</td></tr>
    </table>
  `;

  tampilkanChart(freq);
}

function tampilkanChart(frequencies) {
  const labels = Object.keys(frequencies);
  const values = Object.values(frequencies);
  const ctx = document.getElementById('myChart').getContext('2d');
  if (window.barChart) window.barChart.destroy();

  window.barChart = new Chart(ctx, {
    type: 'bar',
    data: {
      labels: labels,
      datasets: [{
        label: 'Frekuensi Data',
        data: values,
        backgroundColor: '#00bfff'
      }]
    },
    options: {
      responsive: true,
      plugins: { legend: { display: false } },
      scales: {
        y: {
          beginAtZero: true,
          ticks: { stepSize: 1 }
        }
      }
    }
  });
}

function downloadPDF() {
  const element = document.getElementById('pdfArea');
  html2pdf().from(element).save('statistika.pdf');
}

function resetData() {
  document.getElementById('dataInput').value = '';
  document.getElementById('output').innerHTML = '';
  if (window.barChart) window.barChart.destroy();
}
</script>

</body>
</html>
