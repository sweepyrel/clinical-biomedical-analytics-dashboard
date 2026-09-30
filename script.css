/* ═══════════════════════════════════════════════════════════
   Clinical & Biomedical Data Analytics Dashboard
   script.js — All data-driven from healthcare_data.json
   ═══════════════════════════════════════════════════════════ */

'use strict';

// ── Chart.js global defaults ──────────────────────────────
Chart.defaults.color = '#94a3b8';
Chart.defaults.borderColor = '#1e293b';
Chart.defaults.font.family = "'Inter', -apple-system, sans-serif";
Chart.defaults.font.size = 11;

// ── Color palettes ────────────────────────────────────────
const PALETTE = {
  cyan:    'rgba(6,182,212,',
  violet:  'rgba(139,92,246,',
  emerald: 'rgba(16,185,129,',
  amber:   'rgba(245,158,11,',
  rose:    'rgba(244,63,94,',
  sky:     'rgba(56,189,248,',
  indigo:  'rgba(99,102,241,',
  teal:    'rgba(20,184,166,',
};

const CONDITION_COLORS = {
  Arthritis:   '#06b6d4',
  Asthma:      '#8b5cf6',
  Cancer:      '#f43f5e',
  Diabetes:    '#10b981',
  Hypertension:'#f59e0b',
  Obesity:     '#38bdf8',
};

const ADMISSION_COLORS = {
  Emergency: '#f43f5e',
  Elective:  '#10b981',
  Urgent:    '#f59e0b',
};

const TEST_COLORS = {
  Normal:      '#10b981',
  Abnormal:    '#f43f5e',
  Inconclusive:'#94a3b8',
};

// ── State ─────────────────────────────────────────────────
let allData = [];
let filteredData = [];
let charts = {};

// ── Helpers ───────────────────────────────────────────────
function losFromRecord(r) {
  const a = new Date(r.date_of_admission);
  const d = new Date(r.discharge_date);
  const diff = Math.round((d - a) / (1000 * 60 * 60 * 24));
  return diff < 0 ? 0 : diff;
}

function groupBy(arr, key) {
  return arr.reduce((acc, item) => {
    const k = item[key];
    acc[k] = (acc[k] || 0) + 1;
    return acc;
  }, {});
}

function groupAvg(arr, groupKey, valueKey) {
  const sums = {}, counts = {};
  arr.forEach(item => {
    const k = item[groupKey];
    sums[k] = (sums[k] || 0) + item[valueKey];
    counts[k] = (counts[k] || 0) + 1;
  });
  const result = {};
  Object.keys(sums).forEach(k => { result[k] = sums[k] / counts[k]; });
  return result;
}

function getAgeGroup(age) {
  if (age <= 17) return '0–17';
  if (age <= 35) return '18–35';
  if (age <= 55) return '36–55';
  if (age <= 70) return '56–70';
  return '71+';
}

function fmtMoney(n) {
  return '$' + n.toLocaleString('en-US', { minimumFractionDigits: 0, maximumFractionDigits: 0 });
}

function destroyChart(id) {
  if (charts[id]) { charts[id].destroy(); delete charts[id]; }
}

// ── Tooltip plugin shared config ──────────────────────────
function tooltipStyle() {
  return {
    backgroundColor: '#1e293b',
    borderColor: '#334155',
    borderWidth: 1,
    titleColor: '#f1f5f9',
    bodyColor: '#94a3b8',
    padding: 10,
    cornerRadius: 8,
    displayColors: true,
    boxWidth: 10,
    boxHeight: 10,
  };
}

// ── Load Data ─────────────────────────────────────────────
async function loadData() {
  const res = await fetch('healthcare_data.json');
  allData = await res.json();
  // Pre-compute LOS
  allData = allData.map(r => ({ ...r, los: losFromRecord(r) }));
  filteredData = [...allData];

  const d = new Date();
  document.getElementById('last-updated').textContent =
    `Updated: ${d.toLocaleDateString('en-US', { month: 'short', day: 'numeric', year: 'numeric' })}`;

  applyFilters();
}

// ── Filter Logic ──────────────────────────────────────────
function getActiveFilters() {
  return {
    gender:    document.getElementById('filter-gender').value,
    age:       document.getElementById('filter-age').value,
    condition: document.getElementById('filter-condition').value,
    admission: document.getElementById('filter-admission').value,
    test:      document.getElementById('filter-test').value,
    medication:document.getElementById('filter-medication').value,
  };
}

function applyFilters() {
  const f = getActiveFilters();
  filteredData = allData.filter(r => {
    if (f.gender && r.gender !== f.gender) return false;
    if (f.condition && r.medical_condition !== f.condition) return false;
    if (f.admission && r.admission_type !== f.admission) return false;
    if (f.test && r.test_results !== f.test) return false;
    if (f.medication && r.medication !== f.medication) return false;
    if (f.age) {
      const ag = getAgeGroup(r.age);
      const mapAg = { '0-17':'0–17', '18-35':'18–35', '36-55':'36–55', '56-70':'56–70', '71+':'71+' };
      if (ag !== mapAg[f.age]) return false;
    }
    return true;
  });

  const count = filteredData.length;
  document.getElementById('filter-count').textContent =
    count === allData.length ? `Showing all ${count} records` : `Showing ${count} of ${allData.length} records`;

  updateKPIs();
  updateAllCharts();
  updateTable();
}

// ── KPIs ──────────────────────────────────────────────────
function updateKPIs() {
  const n = filteredData.length;
  if (n === 0) {
    ['kpi-total','kpi-age','kpi-los','kpi-records','kpi-billing'].forEach(id => {
      document.getElementById(id).textContent = '—';
    });
    return;
  }

  const avgAge  = filteredData.reduce((s,r) => s + r.age, 0) / n;
  const avgLos  = filteredData.reduce((s,r) => s + r.los, 0) / n;
  const avgBill = filteredData.reduce((s,r) => s + r.billing_amount, 0) / n;

  document.getElementById('kpi-total').textContent   = n.toLocaleString();
  document.getElementById('kpi-age').textContent     = avgAge.toFixed(1);
  document.getElementById('kpi-los').textContent     = avgLos.toFixed(1);
  document.getElementById('kpi-records').textContent = n.toLocaleString();
  document.getElementById('kpi-billing').textContent = fmtMoney(avgBill);
}

// ── Render All Charts ─────────────────────────────────────
function updateAllCharts() {
  renderConditionChart();
  renderAdmissionChart();
  renderTestChart();
  renderAgeChart();
  renderGenderConditionChart();
  renderMedicationChart();
  renderBillingConditionChart();
  renderLosChart();
  renderInsuranceChart();
  renderBloodChart();
  renderTrendChart();
  renderTestAdmissionChart();
  renderScatterChart();
}

// ── 1. Medical Condition Distribution (Doughnut) ──────────
function renderConditionChart() {
  destroyChart('condition');
  const counts = groupBy(filteredData, 'medical_condition');
  const labels = Object.keys(counts);
  const data   = labels.map(l => counts[l]);
  const colors = labels.map(l => CONDITION_COLORS[l] || '#475569');

  const ctx = document.getElementById('chart-condition').getContext('2d');
  charts['condition'] = new Chart(ctx, {
    type: 'doughnut',
    data: {
      labels,
      datasets: [{ data, backgroundColor: colors.map(c => c + 'cc'), borderColor: colors, borderWidth: 2, hoverOffset: 6 }]
    },
    options: {
      responsive: true, maintainAspectRatio: true,
      cutout: '65%',
      plugins: {
        legend: { position: 'right', labels: { padding: 12, boxWidth: 10, boxHeight: 10 } },
        tooltip: { callbacks: { label: ctx => ` ${ctx.label}: ${ctx.parsed} (${((ctx.parsed/filteredData.length)*100).toFixed(1)}%)` }, ...tooltipStyle() }
      }
    }
  });
}

// ── 2. Admission Type (Doughnut) ──────────────────────────
function renderAdmissionChart() {
  destroyChart('admission');
  const counts = groupBy(filteredData, 'admission_type');
  const labels = ['Emergency','Elective','Urgent'].filter(l => counts[l]);
  const data   = labels.map(l => counts[l] || 0);
  const colors = labels.map(l => ADMISSION_COLORS[l]);

  const ctx = document.getElementById('chart-admission').getContext('2d');
  charts['admission'] = new Chart(ctx, {
    type: 'doughnut',
    data: {
      labels,
      datasets: [{ data, backgroundColor: colors.map(c => c + 'cc'), borderColor: colors, borderWidth: 2, hoverOffset: 6 }]
    },
    options: {
      responsive: true, maintainAspectRatio: true,
      cutout: '65%',
      plugins: {
        legend: { position: 'right', labels: { padding: 12, boxWidth: 10, boxHeight: 10 } },
        tooltip: { callbacks: { label: ctx => ` ${ctx.label}: ${ctx.parsed} (${((ctx.parsed/filteredData.length)*100).toFixed(1)}%)` }, ...tooltipStyle() }
      }
    }
  });
}

// ── 3. Test Results (Doughnut) ────────────────────────────
function renderTestChart() {
  destroyChart('test');
  const counts = groupBy(filteredData, 'test_results');
  const labels = ['Normal','Abnormal','Inconclusive'].filter(l => counts[l]);
  const data   = labels.map(l => counts[l] || 0);
  const colors = labels.map(l => TEST_COLORS[l]);

  const ctx = document.getElementById('chart-test').getContext('2d');
  charts['test'] = new Chart(ctx, {
    type: 'doughnut',
    data: {
      labels,
      datasets: [{ data, backgroundColor: colors.map(c => c + 'cc'), borderColor: colors, borderWidth: 2, hoverOffset: 6 }]
    },
    options: {
      responsive: true, maintainAspectRatio: true,
      cutout: '65%',
      plugins: {
        legend: { position: 'right', labels: { padding: 12, boxWidth: 10, boxHeight: 10 } },
        tooltip: { callbacks: { label: ctx => ` ${ctx.label}: ${ctx.parsed} (${((ctx.parsed/filteredData.length)*100).toFixed(1)}%)` }, ...tooltipStyle() }
      }
    }
  });
}

// ── 4. Age Group Distribution (Bar) ──────────────────────
function renderAgeChart() {
  destroyChart('age');
  const groups = ['0–17','18–35','36–55','56–70','71+'];
  const counts = {};
  groups.forEach(g => { counts[g] = 0; });
  filteredData.forEach(r => { counts[getAgeGroup(r.age)]++; });

  const ctx = document.getElementById('chart-age').getContext('2d');
  charts['age'] = new Chart(ctx, {
    type: 'bar',
    data: {
      labels: groups,
      datasets: [{
        label: 'Patients',
        data: groups.map(g => counts[g]),
        backgroundColor: [
          'rgba(6,182,212,0.7)', 'rgba(139,92,246,0.7)', 'rgba(16,185,129,0.7)',
          'rgba(245,158,11,0.7)', 'rgba(244,63,94,0.7)'
        ],
        borderColor: ['#06b6d4','#8b5cf6','#10b981','#f59e0b','#f43f5e'],
        borderWidth: 1.5,
        borderRadius: 6,
      }]
    },
    options: {
      responsive: true, maintainAspectRatio: false,
      plugins: { legend: { display: false }, tooltip: { ...tooltipStyle() } },
      scales: {
        x: { grid: { display: false }, ticks: { color: '#64748b' } },
        y: { grid: { color: '#1e293b' }, ticks: { color: '#64748b', stepSize: 1 } }
      }
    }
  });
}

// ── 5. Gender Distribution by Condition (Grouped Bar) ─────
function renderGenderConditionChart() {
  destroyChart('gender-condition');
  const conditions = ['Arthritis','Asthma','Cancer','Diabetes','Hypertension','Obesity'];
  const males = conditions.map(c => filteredData.filter(r => r.medical_condition === c && r.gender === 'Male').length);
  const females = conditions.map(c => filteredData.filter(r => r.medical_condition === c && r.gender === 'Female').length);

  const ctx = document.getElementById('chart-gender-condition').getContext('2d');
  charts['gender-condition'] = new Chart(ctx, {
    type: 'bar',
    data: {
      labels: conditions,
      datasets: [
        { label: 'Male', data: males, backgroundColor: 'rgba(56,189,248,0.7)', borderColor: '#38bdf8', borderWidth: 1.5, borderRadius: 4 },
        { label: 'Female', data: females, backgroundColor: 'rgba(244,63,94,0.55)', borderColor: '#f43f5e', borderWidth: 1.5, borderRadius: 4 }
      ]
    },
    options: {
      responsive: true, maintainAspectRatio: false,
      plugins: { legend: { position: 'top', labels: { boxWidth: 10, boxHeight: 10, padding: 12 } }, tooltip: { ...tooltipStyle() } },
      scales: {
        x: { grid: { display: false }, ticks: { color: '#64748b', maxRotation: 30 } },
        y: { grid: { color: '#1e293b' }, ticks: { color: '#64748b', stepSize: 1 } }
      }
    }
  });
}

// ── 6. Medication Frequency (Horizontal Bar) ──────────────
function renderMedicationChart() {
  destroyChart('medication');
  const counts = groupBy(filteredData, 'medication');
  const sorted = Object.entries(counts).sort((a,b) => b[1]-a[1]);
  const labels = sorted.map(e => e[0]);
  const data   = sorted.map(e => e[1]);
  const colors = ['rgba(6,182,212,0.75)','rgba(139,92,246,0.75)','rgba(16,185,129,0.75)','rgba(245,158,11,0.75)','rgba(244,63,94,0.75)','rgba(56,189,248,0.75)'];

  const ctx = document.getElementById('chart-medication').getContext('2d');
  charts['medication'] = new Chart(ctx, {
    type: 'bar',
    data: {
      labels,
      datasets: [{
        label: 'Prescriptions',
        data,
        backgroundColor: colors.slice(0, labels.length),
        borderRadius: 6,
        borderSkipped: false,
      }]
    },
    options: {
      indexAxis: 'y',
      responsive: true, maintainAspectRatio: false,
      plugins: { legend: { display: false }, tooltip: { ...tooltipStyle() } },
      scales: {
        x: { grid: { color: '#1e293b' }, ticks: { color: '#64748b' } },
        y: { grid: { display: false }, ticks: { color: '#94a3b8' } }
      }
    }
  });
}

// ── 7. Avg Billing by Condition (Bar) ────────────────────
function renderBillingConditionChart() {
  destroyChart('billing-condition');
  const conditions = ['Arthritis','Asthma','Cancer','Diabetes','Hypertension','Obesity'];
  const avgs = conditions.map(c => {
    const sub = filteredData.filter(r => r.medical_condition === c);
    if (!sub.length) return 0;
    return sub.reduce((s,r) => s + r.billing_amount, 0) / sub.length;
  });

  const ctx = document.getElementById('chart-billing-condition').getContext('2d');
  charts['billing-condition'] = new Chart(ctx, {
    type: 'bar',
    data: {
      labels: conditions,
      datasets: [{
        label: 'Avg Billing (USD)',
        data: avgs.map(v => Math.round(v)),
        backgroundColor: conditions.map(c => (CONDITION_COLORS[c] || '#475569') + 'bb'),
        borderColor: conditions.map(c => CONDITION_COLORS[c] || '#475569'),
        borderWidth: 1.5,
        borderRadius: 6,
      }]
    },
    options: {
      responsive: true, maintainAspectRatio: false,
      plugins: {
        legend: { display: false },
        tooltip: { callbacks: { label: ctx => ` Avg: $${ctx.parsed.y.toLocaleString()}` }, ...tooltipStyle() }
      },
      scales: {
        x: { grid: { display: false }, ticks: { color: '#64748b', maxRotation: 30 } },
        y: {
          grid: { color: '#1e293b' },
          ticks: { color: '#64748b', callback: v => '$' + (v/1000).toFixed(0) + 'k' }
        }
      }
    }
  });
}

// ── 8. Avg LOS by Condition (Bar) ────────────────────────
function renderLosChart() {
  destroyChart('los');
  const conditions = ['Arthritis','Asthma','Cancer','Diabetes','Hypertension','Obesity'];
  const avgLos = conditions.map(c => {
    const sub = filteredData.filter(r => r.medical_condition === c);
    if (!sub.length) return 0;
    return sub.reduce((s,r) => s + r.los, 0) / sub.length;
  });

  const ctx = document.getElementById('chart-los').getContext('2d');
  charts['los'] = new Chart(ctx, {
    type: 'bar',
    data: {
      labels: conditions,
      datasets: [{
        label: 'Avg LOS (Days)',
        data: avgLos.map(v => parseFloat(v.toFixed(1))),
        backgroundColor: 'rgba(16,185,129,0.55)',
        borderColor: '#10b981',
        borderWidth: 1.5,
        borderRadius: 6,
      }]
    },
    options: {
      responsive: true, maintainAspectRatio: false,
      plugins: {
        legend: { display: false },
        tooltip: { callbacks: { label: ctx => ` Avg LOS: ${ctx.parsed.y} days` }, ...tooltipStyle() }
      },
      scales: {
        x: { grid: { display: false }, ticks: { color: '#64748b', maxRotation: 30 } },
        y: { grid: { color: '#1e293b' }, ticks: { color: '#64748b' } }
      }
    }
  });
}

// ── 9. Insurance Provider (Polar Area) ───────────────────
function renderInsuranceChart() {
  destroyChart('insurance');
  const counts = groupBy(filteredData, 'insurance_provider');
  const sorted = Object.entries(counts).sort((a,b) => b[1]-a[1]);
  const labels = sorted.map(e => e[0]);
  const data   = sorted.map(e => e[1]);
  const colors = ['rgba(6,182,212,0.75)','rgba(139,92,246,0.75)','rgba(16,185,129,0.75)','rgba(245,158,11,0.75)','rgba(244,63,94,0.75)','rgba(56,189,248,0.75)'];

  const ctx = document.getElementById('chart-insurance').getContext('2d');
  charts['insurance'] = new Chart(ctx, {
    type: 'polarArea',
    data: {
      labels,
      datasets: [{ data, backgroundColor: colors.slice(0, labels.length), borderWidth: 0 }]
    },
    options: {
      responsive: true, maintainAspectRatio: false,
      plugins: {
        legend: { position: 'right', labels: { boxWidth: 10, boxHeight: 10, padding: 10 } },
        tooltip: { ...tooltipStyle() }
      },
      scales: { r: { grid: { color: '#1e293b' }, ticks: { color: '#64748b', backdropColor: 'transparent' } } }
    }
  });
}

// ── 10. Blood Type (Bar) ──────────────────────────────────
function renderBloodChart() {
  destroyChart('blood');
  const order = ['O+','O-','A+','A-','B+','B-','AB+','AB-'];
  const counts = groupBy(filteredData, 'blood_type');
  const data = order.map(b => counts[b] || 0);
  const colors = [
    'rgba(6,182,212,0.75)','rgba(6,182,212,0.45)',
    'rgba(244,63,94,0.75)','rgba(244,63,94,0.45)',
    'rgba(16,185,129,0.75)','rgba(16,185,129,0.45)',
    'rgba(139,92,246,0.75)','rgba(139,92,246,0.45)',
  ];

  const ctx = document.getElementById('chart-blood').getContext('2d');
  charts['blood'] = new Chart(ctx, {
    type: 'bar',
    data: {
      labels: order,
      datasets: [{
        label: 'Patients',
        data,
        backgroundColor: colors,
        borderColor: colors.map(c => c.replace('0.75','1').replace('0.45','0.9')),
        borderWidth: 1.5,
        borderRadius: 6,
      }]
    },
    options: {
      responsive: true, maintainAspectRatio: false,
      plugins: { legend: { display: false }, tooltip: { ...tooltipStyle() } },
      scales: {
        x: { grid: { display: false }, ticks: { color: '#64748b' } },
        y: { grid: { color: '#1e293b' }, ticks: { color: '#64748b', stepSize: 1 } }
      }
    }
  });
}

// ── 11. Admissions by Year (Line) ────────────────────────
function renderTrendChart() {
  destroyChart('trend');
  const yearCounts = {};
  filteredData.forEach(r => {
    const yr = r.date_of_admission.substring(0, 4);
    yearCounts[yr] = (yearCounts[yr] || 0) + 1;
  });
  const years = Object.keys(yearCounts).sort();
  const data  = years.map(y => yearCounts[y]);

  const ctx = document.getElementById('chart-trend').getContext('2d');
  charts['trend'] = new Chart(ctx, {
    type: 'line',
    data: {
      labels: years,
      datasets: [{
        label: 'Admissions',
        data,
        borderColor: '#06b6d4',
        backgroundColor: 'rgba(6,182,212,0.08)',
        borderWidth: 2.5,
        pointBackgroundColor: '#06b6d4',
        pointBorderColor: '#0f172a',
        pointRadius: 5,
        pointHoverRadius: 7,
        fill: true,
        tension: 0.35,
      }]
    },
    options: {
      responsive: true, maintainAspectRatio: false,
      plugins: { legend: { display: false }, tooltip: { ...tooltipStyle() } },
      scales: {
        x: { grid: { display: false }, ticks: { color: '#64748b' } },
        y: { grid: { color: '#1e293b' }, ticks: { color: '#64748b', stepSize: 1 } }
      }
    }
  });
}

// ── 12. Test Result by Admission Type (Stacked Bar) ───────
function renderTestAdmissionChart() {
  destroyChart('test-admission');
  const admTypes = ['Emergency','Elective','Urgent'];
  const testTypes = ['Normal','Abnormal','Inconclusive'];
  const colors = { Normal: 'rgba(16,185,129,0.75)', Abnormal: 'rgba(244,63,94,0.75)', Inconclusive: 'rgba(148,163,184,0.55)' };

  const datasets = testTypes.map(test => ({
    label: test,
    data: admTypes.map(adm =>
      filteredData.filter(r => r.admission_type === adm && r.test_results === test).length
    ),
    backgroundColor: colors[test],
    borderColor: colors[test].replace('0.75','1').replace('0.55','0.9'),
    borderWidth: 1,
    borderRadius: 4,
  }));

  const ctx = document.getElementById('chart-test-admission').getContext('2d');
  charts['test-admission'] = new Chart(ctx, {
    type: 'bar',
    data: { labels: admTypes, datasets },
    options: {
      responsive: true, maintainAspectRatio: false,
      plugins: {
        legend: { position: 'top', labels: { boxWidth: 10, boxHeight: 10, padding: 12 } },
        tooltip: { ...tooltipStyle() }
      },
      scales: {
        x: { stacked: true, grid: { display: false }, ticks: { color: '#64748b' } },
        y: { stacked: true, grid: { color: '#1e293b' }, ticks: { color: '#64748b' } }
      }
    }
  });
}

// ── 13. Scatter: Age vs Billing ───────────────────────────
function renderScatterChart() {
  destroyChart('scatter');
  const conditionList = ['Arthritis','Asthma','Cancer','Diabetes','Hypertension','Obesity'];

  const datasets = conditionList.map(c => ({
    label: c,
    data: filteredData.filter(r => r.medical_condition === c).map(r => ({ x: r.age, y: Math.round(r.billing_amount) })),
    backgroundColor: (CONDITION_COLORS[c] || '#475569') + '99',
    borderColor: CONDITION_COLORS[c] || '#475569',
    borderWidth: 1,
    pointRadius: 5,
    pointHoverRadius: 7,
  }));

  const ctx = document.getElementById('chart-scatter').getContext('2d');
  charts['scatter'] = new Chart(ctx, {
    type: 'scatter',
    data: { datasets },
    options: {
      responsive: true, maintainAspectRatio: false,
      plugins: {
        legend: { position: 'right', labels: { boxWidth: 8, boxHeight: 8, padding: 8, font: { size: 10 } } },
        tooltip: {
          callbacks: {
            label: ctx => ` ${ctx.dataset.label}: Age ${ctx.parsed.x}, $${ctx.parsed.y.toLocaleString()}`
          },
          ...tooltipStyle()
        }
      },
      scales: {
        x: { title: { display: true, text: 'Age', color: '#64748b' }, grid: { color: '#1e293b' }, ticks: { color: '#64748b' } },
        y: { title: { display: true, text: 'Billing (USD)', color: '#64748b' }, grid: { color: '#1e293b' }, ticks: { color: '#64748b', callback: v => '$' + (v/1000).toFixed(0) + 'k' } }
      }
    }
  });
}

// ── Patient Table ─────────────────────────────────────────
function updateTable() {
  const tbody = document.getElementById('table-body');
  const count = document.getElementById('table-count');
  count.textContent = `${filteredData.length} records`;

  if (filteredData.length === 0) {
    tbody.innerHTML = `<tr><td colspan="11" class="table-td text-center text-slate-500 py-8">No matching records found</td></tr>`;
    return;
  }

  // Show up to 100 rows
  const rows = filteredData.slice(0, 100);
  tbody.innerHTML = rows.map((r, i) => {
    const admBadge = { Emergency: 'badge-emergency', Elective: 'badge-elective', Urgent: 'badge-urgent' };
    const testBadge = { Normal: 'badge-normal', Abnormal: 'badge-abnormal', Inconclusive: 'badge-inconclusive' };
    return `
      <tr class="table-row" style="animation: fadeIn 0.25s ease ${(i % 20) * 0.02}s both;">
        <td class="table-td font-medium text-slate-200">${r.name}</td>
        <td class="table-td">${r.age}</td>
        <td class="table-td">${r.gender}</td>
        <td class="table-td"><span class="font-mono text-xs">${r.blood_type}</span></td>
        <td class="table-td">${r.medical_condition}</td>
        <td class="table-td text-slate-400">${r.date_of_admission}</td>
        <td class="table-td"><span class="badge ${admBadge[r.admission_type] || ''}">${r.admission_type}</span></td>
        <td class="table-td text-center">${r.los}</td>
        <td class="table-td">${r.medication}</td>
        <td class="table-td"><span class="badge ${testBadge[r.test_results] || ''}">${r.test_results}</span></td>
        <td class="table-td text-right font-mono">$${r.billing_amount.toLocaleString('en-US', { minimumFractionDigits: 2, maximumFractionDigits: 2 })}</td>
      </tr>`;
  }).join('');
}

// ── Event Listeners ───────────────────────────────────────
document.addEventListener('DOMContentLoaded', () => {
  // Filter listeners
  ['filter-gender','filter-age','filter-condition','filter-admission','filter-test','filter-medication']
    .forEach(id => document.getElementById(id).addEventListener('change', applyFilters));

  // Reset button
  document.getElementById('reset-filters').addEventListener('click', () => {
    ['filter-gender','filter-age','filter-condition','filter-admission','filter-test','filter-medication']
      .forEach(id => { document.getElementById(id).value = ''; });
    applyFilters();
  });

  // Load data
  loadData().catch(err => {
    console.error('Failed to load healthcare_data.json:', err);
    document.querySelector('main').innerHTML =
      `<div class="text-center py-20 text-rose-400">
        <p class="text-lg font-semibold">Failed to load data</p>
        <p class="text-sm text-slate-500 mt-2">Please ensure healthcare_data.json is in the same directory as index.html and served via a local server.</p>
      </div>`;
  });
});
