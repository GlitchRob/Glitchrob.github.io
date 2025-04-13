<!DOCTYPE html>
<html lang="es">
<head>
  <meta charset="UTF-8"/>
  <meta name="viewport" content="width=device-width, initial-scale=1"/>
  <title>Transacciones - Ingresos con fecha seleccionada, enumeración de cuotas y total general</title>
  
  <!-- Material Icons -->
  <link href="https://fonts.googleapis.com/icon?family=Material+Icons" rel="stylesheet"/>
  
  <!-- Chart.js -->
  <script src="https://cdn.jsdelivr.net/npm/chart.js"></script>
  
  <style>
    body { margin:0; font-family:Arial,sans-serif; background:#000; }
    .transaction-list { padding:10px; margin-bottom:60px; }
    .transaction-card {
      background:#fff; padding:10px; margin-bottom:10px; border-radius:4px;
      box-shadow:0 2px 4px rgba(0,0,0,0.1); cursor:pointer;
    }
    .floating-btn {
      position:fixed; bottom:80px; right:20px; background:#6200ee; color:#fff;
      width:56px; height:56px; border-radius:50%; display:flex; justify-content:center;
      align-items:center; font-size:36px; cursor:pointer; z-index:20;
    }
    /* Botón flotante para información general */
    .general-info-btn {
      position: fixed;
      bottom: 80px;
      left: 20px;
      background: #00796b;
      color: #fff;
      width: 56px;
      height: 56px;
      border-radius: 50%;
      display: flex;
      justify-content: center;
      align-items: center;
      font-size: 36px;
      cursor: pointer;
      z-index: 20;
    }
    .modal {
      display:none; 
      position:fixed; 
      top:0; left:0; right:0; bottom:0;
      background:rgba(0,0,0,0.5); 
      justify-content:center; 
      align-items:center; 
      z-index:30;
    }
    .modal-content {
      box-shadow:0 2px 4px rgba(9,2,0,4.1);
      background:black; 
      color:white; 
      padding:20px; 
      border-radius:4px; 
      width:65%; 
      max-width:500px; 
      position:relative;
    }
    .close-modal {
      position:absolute; 
      top:10px; 
      right:15px; 
      cursor:pointer; 
      font-size:20px;
    }
    .form-group { margin-bottom:10px; }
    label { display:block; margin-bottom:5px; }
    input, select { width:100%; padding:8px; box-sizing:border-box; }
    button[type="submit"], .modal-content button {
      background:#6200ee; 
      color:#fff; 
      border:none; 
      padding:10px 15px;
      cursor:pointer; 
      border-radius:4px; 
      margin-right:5px;
    }
    .group-header {
      margin-bottom:40px;
      color: white;
      /* Se definirá el background de forma dinámica en el JS */
      padding:10px; 
      margin-bottom:5px; 
      border-radius:4px;
      cursor: pointer;
      position: relative;
    }
    .info-btn {
      position: absolute;
      right: 10px;
      top: 10px;
      cursor: pointer;
      font-size: 24px;
      color: white;
    }
    .amount { display:block; text-align:right; }
    .payment-date-separator {
      background:#eee; 
      padding:5px 10px; 
      margin:10px 0; 
      border-left:5px solid #6200ee; 
      font-weight:bold;
      cursor: pointer;
    }
    /* Estilo para el total general fijo en la parte superior */
    #globalTotals {
      position: fixed;
      top: 15px;
      right: 0;
      z-index:40;
      text-align: center;
      color: white;
      background: rgba(0,0,0,0.5);
      padding: 5px;
      border-radius: 4px;
      width: 100%;
    }
    /* Botón para agregar cuenta */
    #addAccountBtn {
      margin: 15px;
      background: #009688;
      color: white;
      border: none;
      padding: 10px 15px;
      border-radius: 4px;
      cursor: pointer;
    }
    /* Contenedor de la gráfica */
    #chartContainer {
      width: 350px;
      height: 350px;
      margin: 35px auto 0 auto;
      margin-top:80px;
    }
    b { color:green; }
    c { color:red; }
    totales { display:inline-block; width:120px; height:15px; }
  </style>
</head>
<body>
  <!-- Contenedor de la gráfica circular -->
  <div id="chartContainer">
    <canvas id="accountsChart"></canvas>
  </div>
  
  <!-- Encabezado global con fecha actual y fecha de último pago -->
  <h1 style="color:white;margin:15px;">
    Transacciones<br>
    <small style="font-size:14px;font-weight:bold; color:grey;">
      Fecha Actual: <span id="currentDate"></span>
    </small><br>
    <small style="font-size:14px;font-weight:bold; color:grey;">
      Último Pago: <span id="lastPaymentDate"></span>
    </small>
  </h1>
  
  <!-- Botón para agregar cuenta -->
  <button id="addAccountBtn">Nueva Cuenta</button>
  
  <!-- Total general fijo -->
  <div id="globalTotals">
    <strong>Deuda actual: 
    Total Mensual: $0</strong><br>
    <small>Total por Viernes: $0</small><br>
    <small>Total General Global: $0</small>
  </div>
  
  <div class="transaction-list" id="transactionList">Transacciones</div>

  <!-- Botón flotante para nueva transacción -->
  <div class="floating-btn" id="addTransactionBtn">
    <span class="material-icons">add</span>
  </div>

  <!-- Botón flotante para información general -->
  <div class="general-info-btn" id="generalInfoBtn">
    <span class="material-icons">info</span>
  </div>

  <!-- Modal para transacción -->
  <div class="modal" id="transactionModal">
    <div class="modal-content">
      <span class="close-modal" id="closeTransactionModal">&times;</span>
      <h2 id="transactionModalTitle">Nueva Transacción</h2>
      <form id="transactionForm">
        <div class="form-group">
          <label for="transCreationDate">Fecha de Creación</label>
          <input type="date" id="transCreationDate" required>
        </div>
        <div class="form-group" id="tipoField">
          <label for="transType">Tipo</label>
          <select id="transType" required>
            <option value="Gasto">Gasto</option>
            <option value="Ingreso">Ingreso</option>
          </select>
        </div>
        <div class="form-group">
          <label for="transAccount">Selecciona una Cuenta</label>
          <select id="transAccount" required></select>
        </div>
        <div class="form-group">
          <label for="transDescription">Descripción</label>
          <input type="text" value="Gasto" id="transDescription" required>
        </div>
        <div class="form-group" id="gastoFields">
          <label for="transCost">Costo</label>
          <input type="number" id="transCost" step="0.01">
        </div>
        <div class="form-group" id="cantidadField">
          <label for="transQuantity">Cantidad</label>
          <input type="number" id="transQuantity" value="1">
        </div>
        <div class="form-group" id="mesesField">
          <label for="transMonths">Meses a Pagar</label>
          <input type="number" id="transMonths" value="1" min="1">
        </div>
        <button type="submit" id="saveTransactionBtn">Guardar</button>
      </form>
    </div>
  </div>

  <!-- Modal para detalles de transacción -->
  <div class="modal" id="transactionDetailsModal">
    <div class="modal-content">
      <span class="close-modal" id="closeTransactionDetailsModal">&times;</span>
      <h2>Detalles de la Transacción</h2>
      <div id="transactionDetailsContent"></div>
      <div id="transactionActionButtons" style="margin-top:10px;"></div>
    </div>
  </div>

  <!-- Modal para información de cuenta -->
  <div class="modal" id="accountInfoModal">
    <div class="modal-content">
      <span class="close-modal" id="closeAccountInfoModal">&times;</span>
      <h2>Información de la Cuenta</h2>
      <div id="accountInfoContent"></div>
    </div>
  </div>

  <!-- Modal para información general -->
  <div class="modal" id="generalInfoModal">
    <div class="modal-content">
      <span class="close-modal" id="closeGeneralInfoModal">&times;</span>
      <h2>Información General</h2>
      <div id="generalInfoContent"></div>
    </div>
  </div>

  <!-- Modal para creación de cuenta -->
  <div class="modal" id="accountModal">
    <div class="modal-content">
      <span class="close-modal" id="closeAccountModal">&times;</span>
      <h2>Nueva Cuenta</h2>
      <form id="accountForm">
         <div class="form-group">
           <label for="accountName">Nombre de la Cuenta</label>
           <input type="text" id="accountName" required>
         </div>
         <div class="form-group">
           <label for="accountColor">Color</label>
           <input type="color" id="accountColor" value="#6200ee">
         </div>
         <div class="form-group">
           <label for="accountCutoffDate">Fecha de Corte</label>
           <input type="date" id="accountCutoffDate" required>
         </div>
         <div class="form-group">
           <label for="accountPaymentDate">Fecha de Pago</label>
           <input type="date" id="accountPaymentDate" required>
         </div>
         <button type="submit">Guardar Cuenta</button>
      </form>
    </div>
  </div>
  
  <!-- Dependencias JS -->
  <script src="https://code.jquery.com/jquery-3.5.1.slim.min.js"></script>
  <script src="https://cdn.jsdelivr.net/npm/popper.js@1.16.1/dist/umd/popper.min.js"></script>
  <script src="https://stackpath.bootstrapcdn.com/bootstrap/4.5.2/js/bootstrap.min.js"></script>
  
  <script>
    let transactions = [];
    let accounts = [];
    // Variable de filtro para múltiples cuentas
    let selectedAccountFilters = [];
    let selectedTransactionIndex = null;
    let isEditingTransaction = false;

    // --- Helpers de fechas ---
    function parseLocalDate(dateStr) {
      const [yyyy, mm, dd] = dateStr.split("-");
      return new Date(+yyyy, +mm - 1, +dd);
    }
    function formatDateForInput(date) {
      const y = date.getFullYear();
      const m = String(date.getMonth() + 1).padStart(2, "0");
      const d = String(date.getDate()).padStart(2, "0");
      return `${y}-${m}-${d}`;
    }
    // Formatea la fecha incluyendo día de la semana
    function formatFullDate(date) {
      return date.toLocaleDateString("es-MX", {
        weekday: "long",
        year: "numeric",
        month: "2-digit",
        day: "2-digit"
      });
    }
    // Obtiene el viernes actual (o el próximo viernes)
    function getCurrentFriday() {
      const today = new Date();
      const diff = (5 - today.getDay() + 7) % 7;
      const friday = new Date(today);
      friday.setDate(today.getDate() + diff);
      return friday;
    }

    // Cálculo de fecha de pago efectivo
    function getEffectivePaymentDate(creationStr, cutoffStr, paymentStr) {
      const creation = parseLocalDate(creationStr);
      const cutoffRef = parseLocalDate(cutoffStr);
      const paymentRef = parseLocalDate(paymentStr);
      const cutoffDay = cutoffRef.getDate();
      const paymentDay = paymentRef.getDate();
      let year = creation.getFullYear();
      let month = creation.getMonth();
      let cycleCutoff = new Date(year, month, cutoffDay);
      if (paymentDay < cutoffDay) {
        if (creation <= cycleCutoff) {
          month += 1;
        } else {
          month += 2;
        }
      } else {
        if (creation <= cycleCutoff) {
          // mismo mes
        } else {
          month += 1;
        }
      }
      while (month > 11) {
        month -= 12;
        year++;
      }
      return new Date(year, month, paymentDay);
    }

    // Para cuotas
    function computeEffectivePaymentDate(transaction, account) {
      let base = getEffectivePaymentDate(transaction.creationDate, account.cutoffDate, account.paymentDate);
      if (transaction.installmentOffset !== undefined) {
        let newDate = new Date(base);
        newDate.setMonth(newDate.getMonth() + transaction.installmentOffset);
        return newDate;
      }
      return base;
    }

    function countFridays(start, end) {
      let count = 0;
      let current = new Date(start);
      while (current < end) {
        if (current.getDay() === 5) count++;
        current.setDate(current.getDate() + 1);
      }
      return count;
    }

    // --- LocalStorage y manejo de cuentas ---
    function loadAccounts() {
      const stored = localStorage.getItem('cuentas');
      accounts = stored ? JSON.parse(stored) : [];
      populateAccountSelect();
      updateAccountsChart();
    }
    function saveAccounts() {
      localStorage.setItem('cuentas', JSON.stringify(accounts));
    }
    function populateAccountSelect() {
      const sel = document.getElementById('transAccount');
      sel.innerHTML = '';
      if (accounts.length === 0) {
        sel.disabled = true;
        let op = document.createElement('option');
        op.value = "";
        op.text = "No hay cuentas disponibles";
        sel.appendChild(op);
      } else {
        sel.disabled = false;
        accounts.forEach((acc, idx) => {
          let op = document.createElement('option');
          op.value = idx;
          op.text = acc.name;
          sel.appendChild(op);
        });
      }
    }

    // --- LocalStorage y manejo de transacciones ---
    function loadTransactions() {
      const stored = localStorage.getItem('transacciones');
      transactions = stored ? JSON.parse(stored) : [];
      renderTransactions();
      updateAccountsChart();
      updateLastPaymentHeader();
    }
    function saveTransactions() {
      localStorage.setItem('transacciones', JSON.stringify(transactions));
    }

    // Actualiza el encabezado principal con el último pago entre todas las transacciones
    function updateLastPaymentHeader() {
      let lastPayment = null;
      transactions.forEach(tr => {
        const acc = accounts[tr.accountIndex];
        if (acc) {
          const eff = computeEffectivePaymentDate(tr, acc);
          if (!lastPayment || eff > lastPayment) {
            lastPayment = eff;
          }
        }
      });
      if(lastPayment) {
        document.getElementById('lastPaymentDate').innerText = formatFullDate(lastPayment);
      } else {
        document.getElementById('lastPaymentDate').innerText = '-';
      }
    }

    // --- Renderizar transacciones ---
    function renderTransactions() {
      // Filtrar transacciones según las cuentas seleccionadas (si hay)
      let filteredTransactions = selectedAccountFilters.length > 0
        ? transactions.filter(tr => selectedAccountFilters.includes(tr.accountIndex))
        : transactions;

      filteredTransactions.sort((a, b) => {
        let accA = accounts[a.accountIndex];
        let accB = accounts[b.accountIndex];
        if (!accA || !accB) return 0;
        let da = computeEffectivePaymentDate(a, accA);
        let db = computeEffectivePaymentDate(b, accB);
        return da - db;
      });
      saveTransactions();

      const container = document.getElementById('transactionList');
      container.innerHTML = '';

      let globalDailyTotal = 0, globalDailyFriTotal = 0, globalGeneralTotal = 0;
      let groups = {};
      filteredTransactions.forEach((tr, idx) => {
        if (!groups[tr.accountIndex]) {
          groups[tr.accountIndex] = { transactions: [], indices: [] };
        }
        groups[tr.accountIndex].transactions.push(tr);
        groups[tr.accountIndex].indices.push(idx);
      });

      // Se obtiene la fecha del viernes actual
      const currentFriday = getCurrentFriday();
      const currentFridayStr = formatFullDate(currentFriday);

      for (let accountIndex in groups) {
        let group = groups[accountIndex];
        let account = accounts[accountIndex];
        if (!account) continue;

        const todayStr = formatDateForInput(new Date());
        const currentPaymentDate = getEffectivePaymentDate(todayStr, account.cutoffDate, account.paymentDate);
        const currentPaymentStr = formatFullDate(currentPaymentDate);

        const today = new Date();
        const diffTime = currentPaymentDate - today;
        const diffDays = Math.ceil(diffTime / (1000 * 60 * 60 * 24));
        const fridaysCount = countFridays(today, currentPaymentDate);

        let groupTransactions = group.transactions.filter(tr => {
          let tEff = computeEffectivePaymentDate(tr, account);
          return tEff.toDateString() === currentPaymentDate.toDateString();
        });
        if (groupTransactions.length === 0) continue;

        let dailyTotal = 0, dailyFriTotal = 0;
        groupTransactions.forEach(x => {
          let amount = (x.type === 'Gasto')
            ? (x.installmentOffset !== undefined ? x.total : (x.months > 1 ? x.total / x.months : x.total))
            : x.total;
          let fCount = countFridays(new Date(), computeEffectivePaymentDate(x, account));
          let portion = fCount > 0 ? amount / fCount : amount;
          if (x.type === 'Gasto') {
            dailyTotal += amount;
            dailyFriTotal += portion;
          } else {
            dailyTotal -= amount;
            dailyFriTotal -= portion;
          }
        });

        globalDailyTotal += dailyTotal;
        globalDailyFriTotal += dailyFriTotal;

        let generalTotal = 0;
        group.transactions.forEach(x => {
          let amount = (x.type === 'Gasto')
            ? (x.installmentOffset !== undefined ? x.total : (x.months > 1 ? x.total / x.months : x.total))
            : x.total;
          generalTotal += (x.type === 'Gasto' ? amount : -amount);
        });
        globalGeneralTotal += generalTotal;

        // Se determina la última fecha de pago (fecha de fin) para esta cuenta
        let lastPaymentForAccount = null;
        group.transactions.forEach(x => {
          const eff = computeEffectivePaymentDate(x, account);
          if (!lastPaymentForAccount || eff > lastPaymentForAccount) {
            lastPaymentForAccount = eff;
          }
        });
        let lastPaymentForAccountStr = lastPaymentForAccount ? formatFullDate(lastPaymentForAccount) : '-';

        // Encabezado del grupo con botón de información
        let header = document.createElement('div');
        header.className = 'group-header';
        header.style.background = account.color 
          ? `linear-gradient(to left, ${account.color}, black)` 
          : 'linear-gradient(to left, #7B5DFF, black)';
        header.innerHTML = `
          <strong style="font-size: 25px;">${account.name}</strong>
          <span class="material-icons info-btn" title="Ver información">info</span><br>
          <small style="font-weight: bold;">Fecha de Pago: ${currentPaymentStr} en ${diffDays} día(s)</small><br>
          <span style="margin-top:-15px;padding:5px;" class="amount">Deuda Actual:<br>
            <span style="margin-top:5px;padding:2px;background: rgba(0,0,0,0.5);color:red;font-size:30px;">
              $${Number(dailyTotal).toLocaleString("es-MX", { minimumFractionDigits: 0, maximumFractionDigits: 2 })}
            </span>
          </span><br>
          <span style="margin-top:-30px;" class="amount1">A pagar hoy: ${dailyFriTotal <= 0 ? "<b>Pagado</b>" : "<c>Pendiente</c>"}<br> 
            <span style="font-size:25px;color:green;font-weight:bold;">
              $${Number(dailyFriTotal).toLocaleString("es-MX", { minimumFractionDigits: 0, maximumFractionDigits: 2 })}
            </span> (${fridaysCount} viernes)<br>
            <small style="font-weight: bold; color: grey;">Límite de pago: ${currentFridayStr}</small><br>
          </span><br>
          <span style="margin-top:-15px;margin-right:-10px;font-size:20px" class="amount">
            Saldo: <span style="font-size:25px;background: black;padding:10px;border-radius:4px;">
              $${Number(generalTotal).toLocaleString("es-MX", { minimumFractionDigits: 0, maximumFractionDigits: 2 })}
            </span>
          </span><br>
          <small style="font-size:12px; color:grey;">Fecha final de pago: ${lastPaymentForAccountStr}</small>
        `;
        // Evitar colapso de grupo al pulsar el botón info
        header.addEventListener('click', (e) => {
          if(e.target.classList.contains('info-btn')) return;
          groupDiv.style.display = groupDiv.style.display === 'none' ? 'block' : 'none';
        });
        container.appendChild(header);

        // Evento para mostrar la información de la cuenta al hacer click en el botón info
        header.querySelector('.info-btn').addEventListener('click', (e) => {
          e.stopPropagation();
          showAccountInfo(parseInt(accountIndex), group.transactions, generalTotal);
        });

        let groupDiv = document.createElement('div');
        groupDiv.style.display = 'none';
        group.transactions.forEach((tr, i) => {
          let globalIdx = group.indices[i];
          let card = document.createElement('div');
          card.className = 'transaction-card';
          card.style.borderLeft = `5px solid ${tr.type === 'Gasto' ? '#f44336' : '#4caf50'}`;
          
          if (tr.type === 'Ingreso') {
            card.innerHTML = `
              <div style="background: linear-gradient(to right, #4c4c4c, black); color: white; padding: 10px; margin: -10px;">
                <small class="amount" style="margin-bottom: -15px;">${account.name}</small>
                <strong>${tr.description}</strong><br>
                <small style="font-weight: bold; font-size: 12px;color: gray;">
                  ${formatFullDate(parseLocalDate(tr.creationDate))}
                </small><br>
                <span class="amount" style="color: #4caf50; margin-top: -15px;">
                  +$${Number(tr.total).toLocaleString("es-MX", { minimumFractionDigits: 0, maximumFractionDigits: 2 })}
                </span>
              </div>
            `;
          } else {
            card.innerHTML = `
              <div style="background: linear-gradient(to right, #4C4C4C, black); color: white; padding: 10px; margin: -10px;">
                <small class="amount" style="margin-bottom: -15px;">${account.name}</small>
                <strong>${tr.description}</strong>
                <small style="border-radius:27px;color:#f44336;font-weight:bold;">
                  ${tr.installmentOffset !== undefined ? `(${tr.installmentOffset + 1} / ${tr.totalMonths})` : ''}
                </small>
                <br>
                <small style="font-weight: bold; font-size: 12px;color:gray;">
                  ${formatFullDate(parseLocalDate(tr.creationDate))}
                </small><br>
                <span class="amount" style="color: #f44336; margin-top: -15px;">
                  -$${Number(tr.total).toLocaleString("es-MX", { minimumFractionDigits: 0, maximumFractionDigits: 2 })}
                </span>
              </div>
            `;
          }
          card.addEventListener('click', () => {
            selectedTransactionIndex = globalIdx;
            showTransactionDetails(tr);
          });
          groupDiv.appendChild(card);
        });
        container.appendChild(groupDiv);
      }

      const globalDiv = document.getElementById('globalTotals');
      globalDiv.innerHTML = `
        <totales>Total Mensual:<br> <span style="color:red;font-size:20px;"> $${Number(globalDailyTotal).toLocaleString("es-MX", { minimumFractionDigits: 0, maximumFractionDigits: 2 })}</span></totales>
        <totales>Total por Viernes:<br> <span style="color:green;font-size:20px;">$${Number(globalDailyFriTotal).toLocaleString("es-MX", { minimumFractionDigits: 0, maximumFractionDigits: 2 })}</span></totales>
        <totales>Deuda Total:<br> <span style="font-size:20px;">$${Number(globalGeneralTotal).toLocaleString("es-MX", { minimumFractionDigits: 0, maximumFractionDigits: 2 })}</span></totales>
      `;
    }

    // Función para mostrar la información de la cuenta en un modal
    function showAccountInfo(accountIndex, accountTransactions, accountTotal) {
      const account = accounts[accountIndex];
      // Filtrar transacciones futuras (a partir de hoy) y agrupar por mes
      const today = new Date();
      let monthlyTotals = {};
      let lastPaymentDate = null;
      accountTransactions.forEach(tr => {
        const eff = computeEffectivePaymentDate(tr, account);
        if (eff >= today) {
          let key = eff.getFullYear() + '-' + String(eff.getMonth()+1).padStart(2, '0');
          if (!monthlyTotals[key]) monthlyTotals[key] = 0;
          let amount = (tr.type === 'Gasto')
            ? (tr.installmentOffset !== undefined ? tr.total : (tr.months > 1 ? tr.total / tr.months : tr.total))
            : tr.total;
          monthlyTotals[key] += (tr.type === 'Gasto' ? amount : -amount);
        }
        // Determinar último pago (fecha máxima)
        if (!lastPaymentDate || eff > lastPaymentDate) {
          lastPaymentDate = eff;
        }
      });
      let monthlyHtml = '';
      if(Object.keys(monthlyTotals).length > 0) {
        monthlyHtml += '<ul>';
        Object.keys(monthlyTotals).sort().forEach(key => {
          monthlyHtml += `<li><strong>${key}:</strong> $${Number(monthlyTotals[key]).toLocaleString("es-MX", { minimumFractionDigits: 0, maximumFractionDigits: 2 })}</li>`;
        });
        monthlyHtml += '</ul>';
      } else {
        monthlyHtml = 'No hay transacciones programadas para meses futuros.';
      }

      let html = `
        <p><strong>Cuenta:</strong> ${account.name}</p>
        <p><strong>Saldo Actual:</strong> $${Number(accountTotal).toLocaleString("es-MX", { minimumFractionDigits: 0, maximumFractionDigits: 2 })}</p>
        <p><strong>Totales Mensuales (próximos meses):</strong><br>${monthlyHtml}</p>
        <p><strong>Fecha de Fin (Último Pago):</strong> ${lastPaymentDate ? formatFullDate(lastPaymentDate) : '-'}</p>
      `;
      document.getElementById('accountInfoContent').innerHTML = html;
      document.getElementById('accountInfoModal').style.display = 'flex';
    }

    // Función para mostrar información general en un modal
    function showGeneralInfo() {
      // Para cada transacción se agrupan las futuras por mes y se obtiene el total global
      const today = new Date();
      let monthlyTotals = {};
      let globalTotal = 0;
      let lastPayment = null;
      transactions.forEach(tr => {
        const acc = accounts[tr.accountIndex];
        if (acc) {
          const eff = computeEffectivePaymentDate(tr, acc);
          if (eff >= today) {
            let key = eff.getFullYear() + '-' + String(eff.getMonth()+1).padStart(2, '0');
            if (!monthlyTotals[key]) monthlyTotals[key] = 0;
            let amount = (tr.type === 'Gasto')
              ? (tr.installmentOffset !== undefined ? tr.total : (tr.months > 1 ? tr.total / tr.months : tr.total))
              : tr.total;
            monthlyTotals[key] += (tr.type === 'Gasto' ? amount : -amount);
          }
          if (!lastPayment || eff > lastPayment) {
            lastPayment = eff;
          }
          globalTotal += (tr.type === 'Gasto'
            ? (tr.installmentOffset !== undefined ? tr.total : (tr.months > 1 ? tr.total / tr.months : tr.total))
            : -tr.total);
        }
      });
      let monthlyHtml = '';
      if(Object.keys(monthlyTotals).length > 0) {
        monthlyHtml += '<ul>';
        Object.keys(monthlyTotals).sort().forEach(key => {
          monthlyHtml += `<li><strong>${key}:</strong> $${Number(monthlyTotals[key]).toLocaleString("es-MX", { minimumFractionDigits: 0, maximumFractionDigits: 2 })}</li>`;
        });
        monthlyHtml += '</ul>';
      } else {
        monthlyHtml = 'No hay transacciones programadas para meses futuros.';
      }
      let html = `
        <p><strong>Deuda Total Global:</strong> $${Number(globalTotal).toLocaleString("es-MX", { minimumFractionDigits: 0, maximumFractionDigits: 2 })}</p>
        <p><strong>Totales Mensuales Globales (próximos meses):</strong><br>${monthlyHtml}</p>
        <p><strong>Último Pago Global:</strong> ${lastPayment ? formatFullDate(lastPayment) : '-'}</p>
      `;
      document.getElementById('generalInfoContent').innerHTML = html;
      document.getElementById('generalInfoModal').style.display = 'flex';
    }

    // --- Actualizar gráfica de cuentas ---
    let accountsChart;
    function updateAccountsChart() {
      const ctx = document.getElementById('accountsChart').getContext('2d');
      const labels = accounts.map(acc => acc.name);
      const data = accounts.map((acc, idx) => {
        let total = 0;
        transactions.filter(t => t.accountIndex == idx).forEach(t => {
          let amount = (t.type === 'Gasto')
            ? (t.installmentOffset !== undefined ? t.total : (t.months > 1 ? t.total / t.months : t.total))
            : t.total;
          total += (t.type === 'Gasto' ? amount : -amount);
        });
        return total;
      });
      const backgroundColors = accounts.map(acc => {
        if (acc.color) {
          return acc.color;
        } else {
          const r = Math.floor(Math.random() * 200) + 55;
          const g = Math.floor(Math.random() * 200) + 55;
          const b = Math.floor(Math.random() * 200) + 55;
          return `rgba(${r},${g},${b},0.7)`;
        }
      });
      if (accountsChart) accountsChart.destroy();
      accountsChart = new Chart(ctx, {
        type: 'pie',
        data: {
          labels: labels,
          datasets: [{
            data: data,
            backgroundColor: backgroundColors
          }]
        },
        options: {
          responsive: true,
          plugins: { legend: { position: 'bottom' } },
          onClick: (evt, elements) => {
            if (elements.length > 0) {
              const index = elements[0].index;
              if (selectedAccountFilters.includes(index)) {
                selectedAccountFilters = selectedAccountFilters.filter(i => i !== index);
              } else {
                selectedAccountFilters.push(index);
              }
              renderTransactions();
            }
          }
        }
      });
    }

    // --- Detalles de transacción ---
    function showTransactionDetails(tr) {
      let acc = accounts[tr.accountIndex];
      let html = `
        <p><strong>Fecha:</strong> ${formatFullDate(parseLocalDate(tr.creationDate))}</p>
        <p><strong>Descripción:</strong> ${tr.description}</p>
        <p><strong>Cuenta:</strong> ${acc ? acc.name : 'N/A'}</p>
      `;
      if (tr.type === 'Gasto') {
        html += `
          <p><strong>Costo:</strong>
            <span class="amount" style="font-size: 40px;">
              $${Number(tr.cost).toLocaleString("es-MX", { minimumFractionDigits: 0, maximumFractionDigits: 2 })} x ${tr.quantity}
            </span>
          </p>
          <p><strong>Pago mensual:</strong>
            <span class="amount" style="font-size: 40px;">
              $${Number(tr.total).toLocaleString("es-MX", { minimumFractionDigits: 0, maximumFractionDigits: 2 })}
            </span>
          </p>
        `;
        if (!tr.installmentOffset && tr.months > 1) {
          let pagoMensual = tr.total / tr.months;
          html += `
            <p><strong>Pago Mensual:</strong>
              <span class="amount">
                $${Number(pagoMensual).toLocaleString("es-MX", { minimumFractionDigits: 0, maximumFractionDigits: 2 })}
              </span>
            </p>
          `;
        }
        if (tr.installmentOffset !== undefined) {
          html += `<p><strong>Cuota:</strong> ${tr.installmentOffset + 1} de ${tr.totalMonths}</p>`;
        }
        let eff = computeEffectivePaymentDate(tr, acc);
        let fc = countFridays(new Date(), eff);
        let portion = (tr.installmentOffset !== undefined || tr.months === 1)
          ? tr.total
          : tr.total / tr.months;
        let totalViernes = fc > 0 ? portion / fc : portion;
        html += `
          <p><strong>Total por Viernes:</strong>
            <span class="amount" style="font-size: 40px;">
              $${Number(totalViernes).toLocaleString("es-MX", { minimumFractionDigits: 0, maximumFractionDigits: 2 })}
            </span>
          </p>
          <p><strong>Fecha de Pago:</strong> ${formatFullDate(eff)}</p>
        `;
      } else {
        html += `
          <p><strong>Total:</strong>
            <span class="amount" style="font-size: 40px;">
              $${Number(tr.total).toLocaleString("es-MX", { minimumFractionDigits: 0, maximumFractionDigits: 2 })}
            </span>
          </p>
        `;
      }
      document.getElementById('transactionDetailsContent').innerHTML = html;
      let btns = document.getElementById('transactionActionButtons');
      btns.innerHTML = `
        <button id="transEditBtn"><span class="material-icons">edit</span></button>
        <button id="transDeleteBtn"><span class="material-icons">delete</span></button>
      `;
      document.getElementById('transactionDetailsModal').style.display = 'flex';

      document.getElementById('transEditBtn').onclick = () => {
        isEditingTransaction = true;
        document.getElementById('transactionModalTitle').innerText = 'Editar Transacción';
        document.getElementById('transCreationDate').value = tr.creationDate;
        document.getElementById('transDescription').value = tr.description;
        document.getElementById('transAccount').value = tr.accountIndex;
        document.getElementById('transType').value = tr.type;
        document.getElementById('gastoFields').style.display = 'block';
        document.getElementById('cantidadField').style.display = 'block';
        document.getElementById('transCost').value = tr.cost;
        document.getElementById('transQuantity').value = tr.quantity;
        if (tr.type === 'Gasto') {
          if (tr.installmentOffset === undefined && tr.months > 1) {
            document.getElementById('mesesField').style.display = 'block';
            document.getElementById('transMonths').value = tr.months;
          } else {
            document.getElementById('mesesField').style.display = 'none';
          }
        } else {
          document.getElementById('mesesField').style.display = 'none';
        }
        document.getElementById('transactionModal').style.display = 'flex';
        document.getElementById('transactionDetailsModal').style.display = 'none';
      };

      document.getElementById('transDeleteBtn').onclick = () => {
        if (confirm("¿Está seguro de eliminar esta transacción?")) {
          let idx = transactions.indexOf(tr);
          if (idx !== -1) {
            transactions.splice(idx, 1);
            saveTransactions();
            renderTransactions();
          }
          document.getElementById('transactionDetailsModal').style.display = 'none';
        }
      };
    }

    // --- Cerrar modales ---
    document.getElementById('closeTransactionModal').onclick = () => {
      document.getElementById('transactionModal').style.display = 'none';
    };
    document.getElementById('closeTransactionDetailsModal').onclick = () => {
      document.getElementById('transactionDetailsModal').style.display = 'none';
    };
    document.getElementById('closeAccountModal').onclick = () => {
      document.getElementById('accountModal').style.display = 'none';
    };
    document.getElementById('closeAccountInfoModal').onclick = () => {
      document.getElementById('accountInfoModal').style.display = 'none';
    };
    document.getElementById('closeGeneralInfoModal').onclick = () => {
      document.getElementById('generalInfoModal').style.display = 'none';
    };
    document.querySelectorAll('.modal').forEach(m => {
      m.addEventListener('click', (e) => {
        if (e.target === m) m.style.display = 'none';
      });
    });

    // --- CREAR/EDITAR TRANSACCIONES ---
    document.getElementById('transactionForm').addEventListener('submit', (e) => {
      e.preventDefault();
      let creationDate = document.getElementById('transCreationDate').value;
      if (!creationDate) {
        creationDate = formatDateForInput(new Date());
      }
      const description = document.getElementById('transDescription').value;
      const accountIndex = parseInt(document.getElementById('transAccount').value);
      const type = document.getElementById('transType').value;
      let cost = parseFloat(document.getElementById('transCost').value) || 0;
      let quantity = parseFloat(document.getElementById('transQuantity').value) || 0;
      let months = (type === 'Gasto') ? parseInt(document.getElementById('transMonths').value) || 1 : 0;
      let baseTotal = cost * quantity;

      if (type === 'Ingreso') {
        const newIncome = {
          creationDate,
          description,
          accountIndex,
          type,
          cost,
          quantity,
          months: 0,
          total: baseTotal
        };
        if (isEditingTransaction && selectedTransactionIndex !== null) {
          transactions[selectedTransactionIndex] = newIncome;
        } else {
          transactions.push(newIncome);
        }
      } else {
        if (type === 'Gasto' && months >= 2) {
          for (let i = 0; i < months; i++) {
            let installmentTotal = baseTotal / months;
            let newRecord = {
              creationDate,
              description,
              accountIndex,
              type,
              cost,
              quantity,
              months: 1,
              total: installmentTotal,
              installmentOffset: i,
              totalMonths: months
            };
            transactions.push(newRecord);
          }
        } else {
          const newData = {
            creationDate,
            description,
            accountIndex,
            type,
            cost,
            quantity,
            months,
            total: baseTotal
          };
          if (isEditingTransaction && selectedTransactionIndex !== null) {
            transactions[selectedTransactionIndex] = newData;
          } else {
            transactions.push(newData);
          }
        }
      }

      saveTransactions();
      renderTransactions();
      updateAccountsChart();
      document.getElementById('transactionModal').style.display = 'none';
      isEditingTransaction = false;
      selectedTransactionIndex = null;
    });

    // --- Crear cuenta (Modal de cuenta) ---
    document.getElementById('accountForm').addEventListener('submit', (e) => {
      e.preventDefault();
      const name = document.getElementById('accountName').value;
      const color = document.getElementById('accountColor').value;
      const cutoffDate = document.getElementById('accountCutoffDate').value;
      const paymentDate = document.getElementById('accountPaymentDate').value;
      if (name && cutoffDate && paymentDate) {
        const newAccount = { name, color, cutoffDate, paymentDate };
        accounts.push(newAccount);
        saveAccounts();
        populateAccountSelect();
        updateAccountsChart();
        document.getElementById('accountModal').style.display = 'none';
      } else {
        alert("Completa todos los campos requeridos");
      }
    });

    // --- Botón para agregar cuenta ---
    document.getElementById('addAccountBtn').addEventListener('click', () => {
      document.getElementById('accountForm').reset();
      document.getElementById('accountModal').style.display = 'flex';
    });

    // --- Botón flotante para nueva transacción ---
    document.getElementById('addTransactionBtn').addEventListener('click', () => {
      isEditingTransaction = false;
      selectedTransactionIndex = null;
      document.getElementById('transactionForm').reset();
      document.getElementById('transCreationDate').value = formatDateForInput(new Date());
      document.getElementById('transType').value = 'Gasto';
      document.getElementById('transDescription').value = '';
      document.getElementById('gastoFields').style.display = 'block';
      document.getElementById('cantidadField').style.display = 'block';
      document.getElementById('mesesField').style.display = 'block';
      document.getElementById('transactionModal').style.display = 'flex';
    });

    // --- Botón flotante para información general ---
    document.getElementById('generalInfoBtn').addEventListener('click', () => {
      showGeneralInfo();
    });

    // --- Mostrar/ocultar campos según tipo de transacción ---
    document.getElementById('transType').addEventListener('change', (e) => {
      if (e.target.value === 'Ingreso') {
        document.getElementById('gastoFields').style.display = 'block';
        document.getElementById('cantidadField').style.display = 'block';
        document.getElementById('mesesField').style.display = 'none';
        document.getElementById('transDescription').value = 'Ingreso';
      } else {
        document.getElementById('gastoFields').style.display = 'block';
        document.getElementById('cantidadField').style.display = 'block';
        document.getElementById('mesesField').style.display = 'block';
        document.getElementById('transDescription').value = '';
      }
    });

    // --- Inicialización ---
    window.onload = function () {
      document.getElementById('currentDate').innerText = formatFullDate(new Date());
      loadAccounts();
      loadTransactions();
    };
  </script>
</body>
</html>
