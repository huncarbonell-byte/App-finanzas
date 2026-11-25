<!DOCTYPE html>
<html lang="es">
<head>
  <meta charset="UTF-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no, viewport-fit=cover" />
  <title>Bóveda Q — Finanzas Personales</title>
  <meta name="description" content="Control de gastos, ingresos, cuentas y pagos fijos en Quetzales." />
  <meta name="theme-color" content="#0F172A" />
  <meta name="apple-mobile-web-app-capable" content="yes" />
  <meta name="apple-mobile-web-app-status-bar-style" content="black-translucent" />
  <meta name="apple-mobile-web-app-title" content="Bóveda Q" />
  <meta name="mobile-web-app-capable" content="yes" />

  <script src="https://cdn.tailwindcss.com"></script>
  <script src="https://unpkg.com/lucide@latest"></script>
  <link rel="preconnect" href="https://fonts.googleapis.com">
  <link rel="preconnect" href="https://fonts.gstatic.com" crossorigin>
  <link href="https://fonts.googleapis.com/css2?family=Inter:wght@400;500;600;700;800&display=swap" rel="stylesheet">

  <style>
    :root {
      --sat: env(safe-area-inset-top, 0px);
      --sab: env(safe-area-inset-bottom, 0px);
    }
    * {
      box-sizing: border-box;
      -webkit-tap-highlight-color: transparent;
      font-family: 'Inter', -apple-system, BlinkMacSystemFont, sans-serif;
    }
    body {
      margin: 0;
      background: #0F172A;
      color: #0F172A;
      min-height: 100dvh;
      display: flex;
      justify-content: center;
      align-items: stretch;
    }
    #app-shell {
      width: 100%;
      max-width: 480px;
      min-height: 100dvh;
      background: #F8FAFC;
      display: flex;
      flex-direction: column;
      position: relative;
      overflow-x: hidden;
      box-shadow: 0 25px 60px rgba(0, 0, 0, 0.45);
    }
    header.app-header {
      padding-top: calc(14px + var(--sat));
    }
    nav.bottom-nav {
      padding-bottom: calc(10px + var(--sab));
    }
    .no-scrollbar::-webkit-scrollbar { display: none; }
    .no-scrollbar { -ms-overflow-style: none; scrollbar-width: none; }
    .modal-backdrop {
      animation: fadeIn 0.18s ease-out forwards;
    }
    .modal-sheet {
      animation: slideUp 0.24s cubic-bezier(0.16, 1, 0.3, 1) forwards;
      padding-bottom: calc(20px + var(--sab));
    }
    @keyframes fadeIn {
      from { opacity: 0; }
      to { opacity: 1; }
    }
    @keyframes slideUp {
      from { transform: translateY(100%); }
      to { transform: translateY(0); }
    }
  </style>
</head>
<body>
  <div id="app-shell">
    <!-- TOP BAR -->
    <header class="app-header bg-slate-900 text-white px-5 pb-4 sticky top-0 z-20 flex items-center justify-between">
      <div class="flex items-center gap-2.5">
        <div class="w-9 h-9 rounded-xl bg-emerald-500 flex items-center justify-center font-extrabold text-slate-950 text-lg shadow-md">
          Q
        </div>
        <div>
          <div class="flex items-center gap-1.5">
            <h1 class="text-base font-bold tracking-tight leading-none">Bóveda Q</h1>
            <span id="offline-badge" class="hidden text-[10px] font-semibold px-1.5 py-0.5 rounded bg-amber-500/20 text-amber-300">Offline</span>
          </div>
          <p class="text-[11px] text-slate-400 mt-0.5" id="header-subtitle">Control multicuenta</p>
        </div>
      </div>

      <div class="flex items-center gap-2">
        <button onclick="togglePrivacy()" title="Ocultar/Mostrar saldos" class="w-9 h-9 rounded-full bg-slate-800 hover:bg-slate-700 flex items-center justify-center text-slate-200 transition">
          <i id="privacy-icon" data-lucide="eye" class="w-4 h-4"></i>
        </button>
        <button onclick="openAlertsModal()" title="Alertas de pagos fijos" class="relative w-9 h-9 rounded-full bg-slate-800 hover:bg-slate-700 flex items-center justify-center text-slate-200 transition">
          <i data-lucide="bell" class="w-4 h-4"></i>
          <span id="alert-badge" class="hidden absolute -top-1 -right-1 min-w-[18px] h-[18px] px-1 rounded-full bg-rose-500 text-white text-[10px] font-bold flex items-center justify-center">0</span>
        </button>
      </div>
    </header>

    <!-- MAIN CONTENT -->
    <main id="main-content" class="flex-1 px-4 pt-4 pb-28 space-y-5 overflow-y-auto"></main>

    <!-- BOTTOM NAVIGATION -->
    <nav class="bottom-nav fixed bottom-0 w-full max-w-[480px] bg-white/95 backdrop-blur-md border-t border-slate-200 px-2 pt-2 z-30 grid grid-cols-5 items-center">
      <button onclick="switchTab('inicio')" data-tab="inicio" class="nav-btn flex flex-col items-center justify-center gap-0.5 text-emerald-600">
        <i data-lucide="home" class="w-5 h-5"></i>
        <span class="text-[10px] font-semibold">Inicio</span>
      </button>

      <button onclick="switchTab('movimientos')" data-tab="movimientos" class="nav-btn flex flex-col items-center justify-center gap-0.5 text-slate-400">
        <i data-lucide="arrow-left-right" class="w-5 h-5"></i>
        <span class="text-[10px] font-semibold">Historial</span>
      </button>

      <div class="flex justify-center">
        <button onclick="openTxModal()" class="-mt-7 w-14 h-14 rounded-full bg-emerald-500 hover:bg-emerald-600 active:scale-95 text-slate-950 shadow-lg shadow-emerald-500/30 flex items-center justify-center transition">
          <i data-lucide="plus" class="w-7 h-7 stroke-[2.5]"></i>
        </button>
      </div>

      <button onclick="switchTab('fijos')" data-tab="fijos" class="nav-btn relative flex flex-col items-center justify-center gap-0.5 text-slate-400">
        <i data-lucide="calendar-clock" class="w-5 h-5"></i>
        <span class="text-[10px] font-semibold">Fijos</span>
        <span id="nav-fijos-dot" class="hidden absolute top-0 right-4 w-2 h-2 rounded-full bg-rose-500"></span>
      </button>

      <button onclick="switchTab('ajustes')" data-tab="ajustes" class="nav-btn flex flex-col items-center justify-center gap-0.5 text-slate-400">
        <i data-lucide="smartphone" class="w-5 h-5"></i>
        <span class="text-[10px] font-semibold">App</span>
      </button>
    </nav>

    <!-- MODAL CONTAINER -->
    <div id="modal-root" class="hidden fixed inset-0 z-50 flex items-end justify-center">
      <div onclick="closeModal()" class="modal-backdrop fixed inset-0 bg-slate-950/60 backdrop-blur-[1px]"></div>
      <div id="modal-box" class="modal-sheet relative w-full max-w-[480px] bg-white rounded-t-3xl p-5 max-h-[90dvh] overflow-y-auto shadow-2xl z-10"></div>
    </div>

    <!-- TOAST -->
    <div id="toast" class="hidden fixed top-20 left-1/2 -translate-x-1/2 z-50 bg-slate-900 text-white text-xs font-medium px-4 py-2.5 rounded-full shadow-xl border border-slate-700"></div>
  </div>

  <script>
    // =========================================================
    // 1. CONFIGURACIÓN PWA (MANIFIESTO + ÍCONO + SERVICE WORKER)
    // =========================================================
    (function setupPWA() {
      try {
        const canvas = document.createElement('canvas');
        canvas.width = 512;
        canvas.height = 512;
        const ctx = canvas.getContext('2d');
        ctx.fillStyle = '#0F172A';
        ctx.fillRect(0, 0, 512, 512);
        ctx.fillStyle = '#10B981';
        ctx.beginPath();
        ctx.roundRect(48, 48, 416, 416, 96);
        ctx.fill();
        ctx.fillStyle = '#020617';
        ctx.font = '800 260px Inter, sans-serif';
        ctx.textAlign = 'center';
        ctx.textBaseline = 'middle';
        ctx.fillText('Q', 256, 272);

        const iconUrl = canvas.toDataURL('image/png');
        const appleTouch = document.createElement('link');
        appleTouch.rel = 'apple-touch-icon';
        appleTouch.href = iconUrl;
        document.head.appendChild(appleTouch);

        const manifest = {
          name: 'Bóveda Q — Control Financiero',
          short_name: 'Bóveda Q',
          description: 'Control personal de cuentas, gastos, ingresos y pagos mensuales fijos.',
          start_url: './',
          display: 'standalone',
          background_color: '#0F172A',
          theme_color: '#0F172A',
          orientation: 'portrait',
          icons: [
            { src: iconUrl, sizes: '192x192', type: 'image/png', purpose: 'any maskable' },
            { src: iconUrl, sizes: '512x512', type: 'image/png', purpose: 'any maskable' }
          ]
        };
        const manifestBlob = new Blob([JSON.stringify(manifest)], { type: 'application/json' });
        const manifestLink = document.createElement('link');
        manifestLink.rel = 'manifest';
        manifestLink.href = URL.createObjectURL(manifestBlob);
        document.head.appendChild(manifestLink);

        if ('serviceWorker' in navigator && window.location.protocol.startsWith('http')) {
          const swCode = `
            const CACHE = 'boveda-q-v1';
            self.addEventListener('install', e => {
              e.waitUntil(caches.open(CACHE).then(c => c.addAll(['./'])).catch(() => {}));
              self.skipWaiting();
            });
            self.addEventListener('activate', e => {
              e.waitUntil(self.clients.claim());
            });
            self.addEventListener('fetch', e => {
              if (e.request.method !== 'GET') return;
              e.respondWith(
                fetch(e.request).then(res => {
                  const copy = res.clone();
                  caches.open(CACHE).then(c => c.put(e.request, copy)).catch(() => {});
                  return res;
                }).catch(() => caches.match(e.request))
              );
            });
          `;
          const swBlob = new Blob([swCode], { type: 'application/javascript' });
          navigator.serviceWorker.register(URL.createObjectURL(swBlob)).catch(() => {});
        }
      } catch (err) {
        console.warn('PWA init:', err);
      }
    })();

    // =========================================================
    // 2. ESTADO INICIAL Y PERSISTENCIA LOCALSTORAGE
    // =========================================================
    const STORAGE_KEY = 'boveda_q_state_v1';

    const CATEGORIES = {
      gasto: [
        { id: 'comida', name: 'Comida', icon: 'utensils', color: '#F97316' },
        { id: 'transporte', name: 'Transporte / Gas', icon: 'car', color: '#3B82F6' },
        { id: 'servicios', name: 'Servicios / Recibos', icon: 'zap', color: '#EAB308' },
        { id: 'estudios', name: 'Estudios / U', icon: 'graduation-cap', color: '#8B5CF6' },
        { id: 'salud', name: 'Salud / Gym', icon: 'heart-pulse', color: '#EC4899' },
        { id: 'ocio', name: 'Ocio / Salidas', icon: 'film', color: '#14B8A6' },
        { id: 'otros', name: 'Otros', icon: 'package', color: '#64748B' }
      ],
      ingreso: [
        { id: 'salario', name: 'Salario / Pago', icon: 'briefcase', color: '#10B981' },
        { id: 'proyecto', name: 'Trabajo / Extra', icon: 'laptop', color: '#06B6D4' },
        { id: 'mesada', name: 'Apoyo / Mesada', icon: 'wallet', color: '#6366F1' },
        { id: 'otros_in', name: 'Otro ingreso', icon: 'plus-circle', color: '#22C55E' }
      ]
    };

    function getDefaultState() {
      const today = new Date();
      const soonDay = Math.min(28, today.getDate() + 2);
      const laterDay = Math.min(28, today.getDate() + 10);
      return {
        privacy: false,
        activeTab: 'inicio',
        filterAccount: 'todas',
        filterType: 'todos',
        searchQuery: '',
        accounts: [
          { id: 'acc_1', name: 'Banco Industrial (Débito)', type: 'debito', balance: 2450.00, limit: 0, color: 'from-slate-800 to-slate-950' },
          { id: 'acc_2', name: 'Efectivo / Billetera', type: 'efectivo', balance: 380.00, limit: 0, color: 'from-emerald-600 to-teal-800' },
          { id: 'acc_3', name: 'Cuenta Ahorros', type: 'ahorro', balance: 4200.00, limit: 0, color: 'from-blue-600 to-indigo-800' },
          { id: 'acc_4', name: 'Tarjeta Crédito BAC', type: 'credito', balance: 620.00, limit: 5000.00, color: 'from-rose-600 to-pink-800' }
        ],
        transactions: [
          { id: 'tx_1', type: 'ingreso', amount: 3200.00, category: 'salario', accountId: 'acc_1', toAccountId: null, note: 'Pago quincena', date: new Date().toISOString().slice(0, 10) },
          { id: 'tx_2', type: 'gasto', amount: 145.00, category: 'transporte', accountId: 'acc_2', toAccountId: null, note: 'Gasolina moto / carro', date: new Date().toISOString().slice(0, 10) },
          { id: 'tx_3', type: 'gasto', amount: 85.00, category: 'comida', accountId: 'acc_1', toAccountId: null, note: 'Almuerzo', date: new Date().toISOString().slice(0, 10) }
        ],
        fixedExpenses: [
          { id: 'fix_1', name: 'Internet Residencial', amount: 299.00, category: 'servicios', accountId: 'acc_2', dayOfMonth: soonDay, leadDays: 3, lastPaidMonth: '' },
          { id: 'fix_2', name: 'Plan Celular', amount: 150.00, category: 'servicios', accountId: 'acc_1', dayOfMonth: soonDay, leadDays: 5, lastPaidMonth: '' },
          { id: 'fix_3', name: 'Gimnasio Mensual', amount: 250.00, category: 'salud', accountId: 'acc_1', dayOfMonth: laterDay, leadDays: 3, lastPaidMonth: '' }
        ]
      };
    }

    let state = (() => {
      try {
        const raw = localStorage.getItem(STORAGE_KEY);
        if (!raw) return getDefaultState();
        return { ...getDefaultState(), ...JSON.parse(raw) };
      } catch {
        return getDefaultState();
      }
    })();

    function saveState() {
      localStorage.setItem(STORAGE_KEY, JSON.stringify(state));
      updateAlertIndicators();
    }

    // =========================================================
    // 3. UTILIDADES Y LÓGICA DE FECHAS / ALERTAS
    // =========================================================
    function formatQ(num) {
      if (state.privacy) return 'Q •••••';
      const val = Number(num || 0);
      return 'Q ' + val.toLocaleString('es-GT', { minimumFractionDigits: 2, maximumFractionDigits: 2 });
    }

    function currentMonthKey() {
      const d = new Date();
      return `${d.getFullYear()}-${String(d.getMonth() + 1).padStart(2, '0')}`;
    }

    function daysUntilBillingDay(dayOfMonth) {
      const now = new Date();
      const today = new Date(now.getFullYear(), now.getMonth(), now.getDate());
      const daysInCurrentMonth = new Date(now.getFullYear(), now.getMonth() + 1, 0).getDate();
      const targetDayCurrent = Math.min(dayOfMonth, daysInCurrentMonth);
      let targetDate = new Date(now.getFullYear(), now.getMonth(), targetDayCurrent);

      if (targetDate < today) {
        const nextMonth = now.getMonth() + 1;
        const nextYear = now.getFullYear() + (nextMonth > 11 ? 1 : 0);
        const normMonth = nextMonth % 12;
        const daysInNext = new Date(nextYear, normMonth + 1, 0).getDate();
        targetDate = new Date(nextYear, normMonth, Math.min(dayOfMonth, daysInNext));
      }
      return Math.round((targetDate - today) / (1000 * 60 * 60 * 24));
    }

    function getAccountAvailable(acc) {
      if (!acc) return 0;
      if (acc.type === 'credito') return Math.max(0, (acc.limit || 0) - (acc.balance || 0));
      return Number(acc.balance || 0);
    }

    function evaluateFixedExpenses() {
      const monthKey = currentMonthKey();
      return state.fixedExpenses.map(fix => {
        const acc = state.accounts.find(a => a.id === fix.accountId);
        const daysLeft = daysUntilBillingDay(Number(fix.dayOfMonth));
        const available = getAccountAvailable(acc);
        const isPaidThisMonth = fix.lastPaidMonth === monthKey;
        const isEmpty = !acc || available <= 0;
        const isInsufficient = !acc || available < Number(fix.amount);
        const shortage = Math.max(0, Number(fix.amount) - available);
        const withinLeadWindow = daysLeft <= Number(fix.leadDays || 3);
        const needsAlert = !isPaidThisMonth && withinLeadWindow && isInsufficient;

        return {
          ...fix,
          account: acc,
          daysLeft,
          available,
          isPaidThisMonth,
          isEmpty,
          isInsufficient,
          shortage,
          withinLeadWindow,
          needsAlert
        };
      }).sort((a, b) => a.daysLeft - b.daysLeft);
    }

    function updateAlertIndicators() {
      const alerts = evaluateFixedExpenses().filter(f => f.needsAlert);
      const badge = document.getElementById('alert-badge');
      const dot = document.getElementById('nav-fijos-dot');
      if (alerts.length > 0) {
        badge.textContent = alerts.length;
        badge.classList.remove('hidden');
        dot.classList.remove('hidden');
      } else {
        badge.classList.add('hidden');
        dot.classList.add('hidden');
      }
    }

    function showToast(msg) {
      const el = document.getElementById('toast');
      el.textContent = msg;
      el.classList.remove('hidden');
      clearTimeout(window._toastTimer);
      window._toastTimer = setTimeout(() => el.classList.add('hidden'), 2600);
    }

    function togglePrivacy() {
      state.privacy = !state.privacy;
      saveState();
      render();
    }

    function switchTab(tab) {
      state.activeTab = tab;
      saveState();
      render();
    }

    function findCategory(id) {
      return [...CATEGORIES.gasto, ...CATEGORIES.ingreso].find(c => c.id === id) || { name: 'General', icon: 'circle', color: '#64748B' };
    }

    // =========================================================
    // 4. VISTAS PRINCIPALES
    // =========================================================
    function render() {
      const privIcon = document.getElementById('privacy-icon');
      privIcon.setAttribute('data-lucide', state.privacy ? 'eye-off' : 'eye');

      document.querySelectorAll('.nav-btn').forEach(btn => {
        const active = btn.dataset.tab === state.activeTab;
        btn.classList.toggle('text-emerald-600', active);
        btn.classList.toggle('text-slate-400', !active);
      });

      const main = document.getElementById('main-content');
      if (state.activeTab === 'inicio') main.innerHTML = renderInicio();
      if (state.activeTab === 'movimientos') main.innerHTML = renderMovimientos();
      if (state.activeTab === 'fijos') main.innerHTML = renderFijos();
      if (state.activeTab === 'ajustes') main.innerHTML = renderAjustes();

      updateAlertIndicators();
      lucide.createIcons();
    }

    function renderInicio() {
      const liquidTotal = state.accounts
        .filter(a => a.type !== 'credito')
        .reduce((sum, a) => sum + Number(a.balance || 0), 0);

      const creditDebt = state.accounts
        .filter(a => a.type === 'credito')
        .reduce((sum, a) => sum + Number(a.balance || 0), 0);

      const monthPrefix = currentMonthKey();
      const monthTxs = state.transactions.filter(t => (t.date || '').startsWith(monthPrefix));
      const monthIncome = monthTxs.filter(t => t.type === 'ingreso').reduce((s, t) => s + Number(t.amount), 0);
      const monthExpense = monthTxs.filter(t => t.type === 'gasto').reduce((s, t) => s + Number(t.amount), 0);

      const evaluatedFixed = evaluateFixedExpenses();
      const activeAlerts = evaluatedFixed.filter(f => f.needsAlert);

      return `
        <!-- PATRIMONIO -->
        <section class="bg-slate-900 text-white rounded-3xl p-5 shadow-lg relative overflow-hidden">
          <div class="absolute -right-8 -top-8 w-36 h-36 rounded-full bg-emerald-500/15 blur-xl"></div>
          <p class="text-xs text-slate-400 font-medium">Saldo disponible total</p>
          <h2 class="text-3xl font-extrabold tracking-tight mt-1">${formatQ(liquidTotal)}</h2>
          ${creditDebt > 0 ? `<p class="text-xs text-rose-300 mt-1">Deuda en tarjetas de crédito: ${formatQ(creditDebt)}</p>` : ''}

          <div class="grid grid-cols-2 gap-3 mt-4 pt-4 border-t border-slate-800">
            <div class="bg-slate-800/80 rounded-2xl p-3">
              <div class="flex items-center gap-1.5 text-emerald-400 text-xs font-semibold">
                <i data-lucide="arrow-down-left" class="w-3.5 h-3.5"></i> Ingresos mes
              </div>
              <p class="text-sm font-bold mt-1">${formatQ(monthIncome)}</p>
            </div>
            <div class="bg-slate-800/80 rounded-2xl p-3">
              <div class="flex items-center gap-1.5 text-rose-400 text-xs font-semibold">
                <i data-lucide="arrow-up-right" class="w-3.5 h-3.5"></i> Gastos mes
              </div>
              <p class="text-sm font-bold mt-1">${formatQ(monthExpense)}</p>
            </div>
          </div>
        </section>

        <!-- ALERTAS DE PAGOS FIJOS -->
        ${activeAlerts.length > 0 ? `
          <section class="bg-rose-50 border border-rose-200 rounded-2xl p-4 space-y-3">
            <div class="flex items-center justify-between">
              <div class="flex items-center gap-2 text-rose-700 font-bold text-xs uppercase tracking-wide">
                <i data-lucide="alert-triangle" class="w-4 h-4"></i>
                <span>Aviso anticipado de pago (${activeAlerts.length})</span>
              </div>
              <button onclick="switchTab('fijos')" class="text-xs font-semibold text-rose-700 underline">Ver todos</button>
            </div>
            <div class="space-y-2.5">
              ${activeAlerts.map(item => `
                <div class="bg-white rounded-xl p-3 border border-rose-100 shadow-sm">
                  <div class="flex items-start justify-between gap-2">
                    <div>
                      <p class="font-bold text-sm text-slate-900">${item.name}</p>
                      <p class="text-xs text-slate-500 mt-0.5">
                        Cobra el día <b>${item.dayOfMonth}</b> (${item.daysLeft === 0 ? '¡Hoy!' : `en ${item.daysLeft} día${item.daysLeft === 1 ? '' : 's'}`}) · <b>${item.account ? item.account.name : 'Sin cuenta'}</b>
                      </p>
                      <p class="text-xs font-semibold text-rose-600 mt-1">
                        ${item.isEmpty ? 'Cuenta vacía (Q 0.00).' : `Saldo actual: ${formatQ(item.available)}.`} Faltan ${formatQ(item.shortage)}
                      </p>
                    </div>
                    <span class="font-extrabold text-sm text-slate-900">${formatQ(item.amount)}</span>
                  </div>
                  <div class="flex gap-2 mt-2.5">
                    <button onclick="openFundModal('${item.id}')" class="flex-1 py-1.5 px-3 rounded-lg bg-slate-900 text-white text-xs font-semibold hover:bg-slate-800 transition">
                      Fondear cuenta
                    </button>
                    <button onclick="payFixedNow('${item.id}')" class="py-1.5 px-3 rounded-lg bg-emerald-100 text-emerald-800 text-xs font-semibold hover:bg-emerald-200 transition">
                      Ya lo pagué
                    </button>
                  </div>
                </div>
              `).join('')}
            </div>
          </section>
        ` : ''}

        <!-- MIS CUENTAS -->
        <section class="space-y-2.5">
          <div class="flex items-center justify-between">
            <h3 class="text-sm font-bold text-slate-800">Mis cuentas</h3>
            <button onclick="openAccountModal()" class="text-xs font-semibold text-emerald-600 flex items-center gap-1">
              <i data-lucide="plus" class="w-3.5 h-3.5"></i> Nueva cuenta
            </button>
          </div>
          <div class="flex gap-3 overflow-x-auto no-scrollbar pb-1">
            ${state.accounts.map(acc => `
              <div onclick="openAccountModal('${acc.id}')" class="min-w-[205px] bg-gradient-to-br ${acc.color} text-white rounded-2xl p-4 shadow-md cursor-pointer active:scale-[0.98] transition shrink-0">
                <div class="flex items-center justify-between">
                  <span class="text-[10px] uppercase tracking-wider bg-white/15 px-2 py-0.5 rounded-full font-semibold">${acc.type}</span>
                  <i data-lucide="${acc.type === 'credito' ? 'credit-card' : acc.type === 'efectivo' ? 'banknote' : 'landmark'}" class="w-4 h-4 text-white/80"></i>
                </div>
                <p class="text-xs text-white/85 font-medium mt-3 truncate">${acc.name}</p>
                <p class="text-lg font-extrabold mt-0.5">${formatQ(acc.balance)}</p>
                ${acc.type === 'credito' ? `<p class="text-[11px] text-white/75 mt-1">Disponible: ${formatQ(getAccountAvailable(acc))}</p>` : ''}
              </div>
            `).join('')}
          </div>
        </section>

        <!-- ÚLTIMOS MOVIMIENTOS -->
        <section class="space-y-2.5">
          <div class="flex items-center justify-between">
            <h3 class="text-sm font-bold text-slate-800">Movimientos recientes</h3>
            <button onclick="switchTab('movimientos')" class="text-xs font-semibold text-slate-500">Ver historial</button>
          </div>
          <div class="bg-white rounded-2xl border border-slate-200/80 divide-y divide-slate-100">
            ${state.transactions.slice(0, 5).map(tx => renderTxRow(tx)).join('') || `
              <p class="p-6 text-center text-xs text-slate-400">Aún no hay movimientos registrados. Toca el botón + para empezar.</p>
            `}
          </div>
        </section>
      `;
    }

    function renderTxRow(tx) {
      const acc = state.accounts.find(a => a.id === tx.accountId);
      const toAcc = tx.toAccountId ? state.accounts.find(a => a.id === tx.toAccountId) : null;
      const cat = findCategory(tx.category);
      const isTransfer = tx.type === 'transferencia';
      const isIncome = tx.type === 'ingreso';

      return `
        <div class="p-3.5 flex items-center justify-between gap-3">
          <div class="flex items-center gap-3 min-w-0">
            <div class="w-10 h-10 rounded-xl flex items-center justify-center shrink-0" style="background:${isTransfer ? '#E0F2FE' : cat.color + '20'}; color:${isTransfer ? '#0284C7' : cat.color}">
              <i data-lucide="${isTransfer ? 'repeat' : cat.icon}" class="w-5 h-5"></i>
            </div>
            <div class="min-w-0">
              <p class="text-sm font-semibold text-slate-800 truncate">${tx.note || (isTransfer ? 'Transferencia' : cat.name)}</p>
              <p class="text-[11px] text-slate-400 truncate">
                ${isTransfer ? `${acc?.name || ''} → ${toAcc?.name || ''}` : `${acc?.name || 'Cuenta'} · ${tx.date}`}
              </p>
            </div>
          </div>
          <div class="text-right shrink-0">
            <p class="text-sm font-bold ${isTransfer ? 'text-sky-600' : isIncome ? 'text-emerald-600' : 'text-slate-900'}">
              ${isTransfer ? '' : isIncome ? '+' : '-'}${formatQ(tx.amount)}
            </p>
            <button onclick="deleteTx('${tx.id}')" class="text-[10px] text-slate-400 hover:text-rose-500">Eliminar</button>
          </div>
        </div>
      `;
    }

    function renderMovimientos() {
      const filtered = state.transactions.filter(tx => {
        if (state.filterAccount !== 'todas' && tx.accountId !== state.filterAccount && tx.toAccountId !== state.filterAccount) return false;
        if (state.filterType !== 'todos' && tx.type !== state.filterType) return false;
        if (state.searchQuery && !(tx.note || '').toLowerCase().includes(state.searchQuery.toLowerCase())) return false;
        return true;
      });

      return `
        <section class="space-y-3">
          <h2 class="text-lg font-extrabold text-slate-900">Historial de movimientos</h2>

          <input type="text" value="${state.searchQuery}" oninput="state.searchQuery=this.value; render();" placeholder="Buscar por nota o concepto..." class="w-full px-3.5 py-2.5 rounded-xl bg-white border border-slate-200 text-sm focus:outline-none focus:border-emerald-500" />

          <div class="flex gap-2 overflow-x-auto no-scrollbar">
            <select onchange="state.filterAccount=this.value; render();" class="px-3 py-2 rounded-xl bg-white border border-slate-200 text-xs font-semibold text-slate-700">
              <option value="todas" ${state.filterAccount === 'todas' ? 'selected' : ''}>Todas las cuentas</option>
              ${state.accounts.map(a => `<option value="${a.id}" ${state.filterAccount === a.id ? 'selected' : ''}>${a.name}</option>`).join('')}
            </select>

            <select onchange="state.filterType=this.value; render();" class="px-3 py-2 rounded-xl bg-white border border-slate-200 text-xs font-semibold text-slate-700">
              <option value="todos" ${state.filterType === 'todos' ? 'selected' : ''}>Todos los tipos</option>
              <option value="gasto" ${state.filterType === 'gasto' ? 'selected' : ''}>Gastos</option>
              <option value="ingreso" ${state.filterType === 'ingreso' ? 'selected' : ''}>Ingresos</option>
              <option value="transferencia" ${state.filterType === 'transferencia' ? 'selected' : ''}>Transferencias</option>
            </select>
          </div>

          <div class="bg-white rounded-2xl border border-slate-200 divide-y divide-slate-100">
            ${filtered.map(tx => renderTxRow(tx)).join('') || `<p class="p-8 text-center text-xs text-slate-400">No hay movimientos con estos filtros.</p>`}
          </div>
        </section>
      `;
    }

    function renderFijos() {
      const items = evaluateFixedExpenses();
      const totalMonthly = items.reduce((s, i) => s + Number(i.amount), 0);

      return `
        <section class="space-y-4">
          <div class="flex items-center justify-between">
            <div>
              <h2 class="text-lg font-extrabold text-slate-900">Gastos mensuales fijos</h2>
              <p class="text-xs text-slate-500">Total comprometido al mes: <b>${formatQ(totalMonthly)}</b></p>
            </div>
            <button onclick="openFixedModal()" class="px-3.5 py-2 rounded-xl bg-emerald-500 text-slate-950 font-bold text-xs flex items-center gap-1 shadow-sm">
              <i data-lucide="plus" class="w-4 h-4"></i> Agregar
            </button>
          </div>

          <div class="space-y-3">
            ${items.map(item => `
              <div class="bg-white rounded-2xl p-4 border ${item.needsAlert ? 'border-rose-300 ring-2 ring-rose-500/15' : 'border-slate-200'} shadow-sm space-y-3">
                <div class="flex items-start justify-between gap-2">
                  <div>
                    <div class="flex items-center gap-2">
                      <h3 class="font-bold text-sm text-slate-900">${item.name}</h3>
                      ${item.isPaidThisMonth
                        ? `<span class="text-[10px] font-bold px-2 py-0.5 rounded-full bg-emerald-100 text-emerald-700">Pagado este mes</span>`
                        : item.needsAlert
                          ? `<span class="text-[10px] font-bold px-2 py-0.5 rounded-full bg-rose-100 text-rose-700">Saldo insuficiente</span>`
                          : `<span class="text-[10px] font-bold px-2 py-0.5 rounded-full bg-slate-100 text-slate-600">En ${item.daysLeft}d</span>`
                      }
                    </div>
                    <p class="text-xs text-slate-500 mt-1">
                      Se descuenta de: <b>${item.account ? item.account.name : 'Cuenta eliminada'}</b>
                    </p>
                    <p class="text-[11px] text-slate-400 mt-0.5">
                      Cobro cada día <b>${item.dayOfMonth}</b> · Aviso <b>${item.leadDays} días antes</b>
                    </p>
                  </div>
                  <div class="text-right">
                    <p class="text-base font-extrabold text-slate-900">${formatQ(item.amount)}</p>
                    <div class="flex items-center justify-end gap-2 mt-1">
                      <button onclick="openFixedModal('${item.id}')" class="text-[11px] font-semibold text-slate-500 hover:text-slate-800">Editar</button>
                      <button onclick="deleteFixed('${item.id}')" class="text-[11px] font-semibold text-rose-500">Borrar</button>
                    </div>
                  </div>
                </div>

                ${!item.isPaidThisMonth ? `
                  <div class="pt-2.5 border-t border-slate-100 flex items-center justify-between gap-2">
                    <span class="text-[11px] ${item.isInsufficient ? 'text-rose-600 font-semibold' : 'text-emerald-600 font-medium'}">
                      ${item.isInsufficient ? `Faltan ${formatQ(item.shortage)} en cuenta` : `Saldo cubierto (${formatQ(item.available)})`}
                    </span>
                    <div class="flex gap-2">
                      ${item.isInsufficient ? `<button onclick="openFundModal('${item.id}')" class="px-2.5 py-1.5 rounded-lg bg-slate-900 text-white text-[11px] font-semibold">Fondear</button>` : ''}
                      <button onclick="payFixedNow('${item.id}')" class="px-2.5 py-1.5 rounded-lg bg-emerald-500 text-slate-950 text-[11px] font-bold">Registrar pago</button>
                    </div>
                  </div>
                ` : ''}
              </div>
            `).join('') || `<p class="bg-white rounded-2xl p-8 text-center text-xs text-slate-400 border border-slate-200">No tienes gastos fijos registrados aún.</p>`}
          </div>
        </section>
      `;
    }

    function renderAjustes() {
      const notifStatus = ('Notification' in window) ? Notification.permission : 'unsupported';
      return `
        <section class="space-y-4">
          <h2 class="text-lg font-extrabold text-slate-900">Instalación y respaldos</h2>

          <!-- GUÍA DE INSTALACIÓN -->
          <div class="bg-white rounded-2xl p-4 border border-slate-200 space-y-3">
            <div class="flex items-center gap-2 text-slate-900 font-bold text-sm">
              <i data-lucide="download" class="w-4 h-4 text-emerald-600"></i>
              <span>Instalar en tu teléfono (App nativa)</span>
            </div>
            <div class="text-xs text-slate-600 space-y-2 leading-relaxed">
              <p><b>En Android (Chrome):</b> Toca el menú <b>⋮</b> arriba a la derecha y elige <b>"Instalar aplicación"</b> o <b>"Agregar a pantalla principal"</b>.</p>
              <p><b>En iPhone (Safari):</b> Toca el botón <b>Compartir</b> (cuadro con flecha hacia arriba) y selecciona <b>"Agregar a inicio"</b>.</p>
            </div>
          </div>

          <!-- NOTIFICACIONES -->
          <div class="bg-white rounded-2xl p-4 border border-slate-200 space-y-3">
            <div class="flex items-center justify-between">
              <div>
                <h3 class="font-bold text-sm text-slate-900">Notificaciones del sistema</h3>
                <p class="text-xs text-slate-500">Estado actual: <b>${notifStatus}</b></p>
              </div>
              <button onclick="requestNotifPermission()" class="px-3 py-2 rounded-xl bg-slate-900 text-white text-xs font-semibold">
                Activar / Probar
              </button>
            </div>
          </div>

          <!-- RESPALDO JSON -->
          <div class="bg-white rounded-2xl p-4 border border-slate-200 space-y-3">
            <h3 class="font-bold text-sm text-slate-900">Copia de seguridad de tus datos</h3>
            <p class="text-xs text-slate-500">Tus registros se guardan localmente en este dispositivo. Exporta un respaldo cuando quieras.</p>
            <div class="grid grid-cols-2 gap-2.5">
              <button onclick="exportBackup()" class="py-2.5 px-3 rounded-xl bg-emerald-500 text-slate-950 font-bold text-xs flex items-center justify-center gap-1.5">
                <i data-lucide="download-cloud" class="w-4 h-4"></i> Exportar JSON
              </button>
              <label class="py-2.5 px-3 rounded-xl bg-slate-100 text-slate-800 font-bold text-xs flex items-center justify-center gap-1.5 cursor-pointer">
                <i data-lucide="upload-cloud" class="w-4 h-4"></i> Importar JSON
                <input type="file" accept=".json" onchange="importBackup(event)" class="hidden" />
              </label>
            </div>
          </div>
        </section>
      `;
    }

    // =========================================================
    // 5. MODALES Y ACCIONES
    // =========================================================
    function openModal(html) {
      document.getElementById('modal-box').innerHTML = html;
      document.getElementById('modal-root').classList.remove('hidden');
      lucide.createIcons();
    }

    function closeModal() {
      document.getElementById('modal-root').classList.add('hidden');
    }

    function openTxModal(defaultType = 'gasto') {
      openModal(`
        <div class="space-y-4">
          <div class="flex items-center justify-between">
            <h3 class="text-base font-extrabold text-slate-900">Nuevo movimiento</h3>
            <button onclick="closeModal()" class="p-1 text-slate-400"><i data-lucide="x" class="w-5 h-5"></i></button>
          </div>

          <div class="grid grid-cols-3 gap-1.5 bg-slate-100 p-1 rounded-xl">
            <button type="button" onclick="setTxType('gasto')" id="tab-gasto" class="tx-type-btn py-2 rounded-lg text-xs font-bold bg-white text-rose-600 shadow-sm">Gasto</button>
            <button type="button" onclick="setTxType('ingreso')" id="tab-ingreso" class="tx-type-btn py-2 rounded-lg text-xs font-bold text-slate-500">Ingreso</button>
            <button type="button" onclick="setTxType('transferencia')" id="tab-transferencia" class="tx-type-btn py-2 rounded-lg text-xs font-bold text-slate-500">Transferir</button>
          </div>

          <form onsubmit="submitTx(event)" class="space-y-3">
            <input type="hidden" id="tx-type" value="${defaultType}" />

            <div>
              <label class="text-xs font-semibold text-slate-600">Monto (Q)</label>
              <input id="tx-amount" type="number" step="0.01" min="0.01" required placeholder="0.00" class="w-full mt-1 px-3.5 py-2.5 rounded-xl border border-slate-200 text-lg font-extrabold focus:outline-none focus:border-emerald-500" />
            </div>

            <div>
              <label id="tx-acc-label" class="text-xs font-semibold text-slate-600">Cuenta</label>
              <select id="tx-account" class="w-full mt-1 px-3.5 py-2.5 rounded-xl border border-slate-200 text-sm bg-white">
                ${state.accounts.map(a => `<option value="${a.id}">${a.name} (${formatQ(a.balance)})</option>`).join('')}
              </select>
            </div>

            <div id="tx-to-wrapper" class="hidden">
              <label class="text-xs font-semibold text-slate-600">Cuenta destino</label>
              <select id="tx-to-account" class="w-full mt-1 px-3.5 py-2.5 rounded-xl border border-slate-200 text-sm bg-white">
                ${state.accounts.map((a, idx) => `<option value="${a.id}" ${idx === 1 ? 'selected' : ''}>${a.name}</option>`).join('')}
              </select>
            </div>

            <div id="tx-cat-wrapper">
              <label class="text-xs font-semibold text-slate-600">Categoría</label>
              <select id="tx-category" class="w-full mt-1 px-3.5 py-2.5 rounded-xl border border-slate-200 text-sm bg-white">
                ${CATEGORIES.gasto.map(c => `<option value="${c.id}">${c.name}</option>`).join('')}
              </select>
            </div>

            <div class="grid grid-cols-2 gap-2.5">
              <div>
                <label class="text-xs font-semibold text-slate-600">Fecha</label>
                <input id="tx-date" type="date" value="${new Date().toISOString().slice(0, 10)}" class="w-full mt-1 px-3 py-2 rounded-xl border border-slate-200 text-xs" />
              </div>
              <div>
                <label class="text-xs font-semibold text-slate-600">Nota (opcional)</label>
                <input id="tx-note" type="text" placeholder="Ej. Cena, gasolina..." class="w-full mt-1 px-3 py-2 rounded-xl border border-slate-200 text-xs" />
              </div>
            </div>

            <button type="submit" class="w-full py-3 rounded-xl bg-emerald-500 hover:bg-emerald-600 text-slate-950 font-extrabold text-sm shadow-md">
              Guardar movimiento
            </button>
          </form>
        </div>
      `);
    }

    function setTxType(type) {
      document.getElementById('tx-type').value = type;
      ['gasto', 'ingreso', 'transferencia'].forEach(t => {
        const btn = document.getElementById('tab-' + t);
        const active = t === type;
        btn.className = `tx-type-btn py-2 rounded-lg text-xs font-bold ${active ? 'bg-white shadow-sm ' + (t === 'gasto' ? 'text-rose-600' : t === 'ingreso' ? 'text-emerald-600' : 'text-sky-600') : 'text-slate-500'}`;
      });
      document.getElementById('tx-to-wrapper').classList.toggle('hidden', type !== 'transferencia');
      document.getElementById('tx-cat-wrapper').classList.toggle('hidden', type === 'transferencia');
      document.getElementById('tx-acc-label').textContent = type === 'transferencia' ? 'Cuenta origen' : 'Cuenta';

      if (type !== 'transferencia') {
        document.getElementById('tx-category').innerHTML = CATEGORIES[type].map(c => `<option value="${c.id}">${c.name}</option>`).join('');
      }
    }

    function submitTx(e) {
      e.preventDefault();
      const type = document.getElementById('tx-type').value;
      const amount = parseFloat(document.getElementById('tx-amount').value);
      const accountId = document.getElementById('tx-account').value;
      const toAccountId = document.getElementById('tx-to-account').value;
      const category = document.getElementById('tx-category').value;
      const date = document.getElementById('tx-date').value;
      const note = document.getElementById('tx-note').value.trim();

      if (type === 'transferencia' && accountId === toAccountId) {
        showToast('Elige dos cuentas diferentes');
        return;
      }

      applyBalanceChange(type, amount, accountId, toAccountId, 1);

      state.transactions.unshift({
        id: 'tx_' + Date.now(),
        type,
        amount,
        category: type === 'transferencia' ? 'transfer' : category,
        accountId,
        toAccountId: type === 'transferencia' ? toAccountId : null,
        date,
        note
      });

      saveState();
      closeModal();
      render();
      showToast('Movimiento registrado');
    }

    function applyBalanceChange(type, amount, accountId, toAccountId, dir) {
      const acc = state.accounts.find(a => a.id === accountId);
      if (acc) {
        if (type === 'gasto') {
          acc.balance = Number(acc.balance) + (acc.type === 'credito' ? amount * dir : -amount * dir);
        } else if (type === 'ingreso') {
          acc.balance = Number(acc.balance) + (acc.type === 'credito' ? -amount * dir : amount * dir);
        } else if (type === 'transferencia') {
          acc.balance = Number(acc.balance) - amount * dir;
          const dest = state.accounts.find(a => a.id === toAccountId);
          if (dest) {
            dest.balance = Number(dest.balance) + (dest.type === 'credito' ? -amount * dir : amount * dir);
          }
        }
      }
    }

    function deleteTx(id) {
      const idx = state.transactions.findIndex(t => t.id === id);
      if (idx === -1) return;
      const tx = state.transactions[idx];
      applyBalanceChange(tx.type, Number(tx.amount), tx.accountId, tx.toAccountId, -1);
      state.transactions.splice(idx, 1);
      saveState();
      render();
      showToast('Movimiento eliminado y saldo revertido');
    }

    function openFixedModal(editId = null) {
      const item = editId ? state.fixedExpenses.find(f => f.id === editId) : null;
      openModal(`
        <div class="space-y-4">
          <div class="flex items-center justify-between">
            <h3 class="text-base font-extrabold text-slate-900">${item ? 'Editar gasto fijo' : 'Nuevo gasto mensual fijo'}</h3>
            <button onclick="closeModal()" class="p-1 text-slate-400"><i data-lucide="x" class="w-5 h-5"></i></button>
          </div>
          <form onsubmit="submitFixed(event, '${editId || ''}')" class="space-y-3">
            <div>
              <label class="text-xs font-semibold text-slate-600">Nombre del pago</label>
              <input id="fix-name" required value="${item?.name || ''}" placeholder="Ej. Internet, Luz, Spotify, Renta" class="w-full mt-1 px-3.5 py-2.5 rounded-xl border border-slate-200 text-sm" />
            </div>
            <div class="grid grid-cols-2 gap-2.5">
              <div>
                <label class="text-xs font-semibold text-slate-600">Monto mensual (Q)</label>
                <input id="fix-amount" type="number" step="0.01" min="0.01" required value="${item?.amount || ''}" placeholder="0.00" class="w-full mt-1 px-3.5 py-2.5 rounded-xl border border-slate-200 text-sm font-bold" />
              </div>
              <div>
                <label class="text-xs font-semibold text-slate-600">Día de cobro (1-31)</label>
                <input id="fix-day" type="number" min="1" max="31" required value="${item?.dayOfMonth || 15}" class="w-full mt-1 px-3.5 py-2.5 rounded-xl border border-slate-200 text-sm font-bold" />
              </div>
            </div>
            <div>
              <label class="text-xs font-semibold text-slate-600">Cuenta de donde se descontará</label>
              <select id="fix-account" class="w-full mt-1 px-3.5 py-2.5 rounded-xl border border-slate-200 text-sm bg-white">
                ${state.accounts.map(a => `<option value="${a.id}" ${item?.accountId === a.id ? 'selected' : ''}>${a.name}</option>`).join('')}
              </select>
            </div>
            <div class="grid grid-cols-2 gap-2.5">
              <div>
                <label class="text-xs font-semibold text-slate-600">Avisar con anticipación</label>
                <select id="fix-lead" class="w-full mt-1 px-3.5 py-2.5 rounded-xl border border-slate-200 text-sm bg-white">
                  ${[1, 2, 3, 5, 7, 10].map(d => `<option value="${d}" ${(item?.leadDays || 3) === d ? 'selected' : ''}>${d} día${d === 1 ? '' : 's'} antes</option>`).join('')}
                </select>
              </div>
              <div>
                <label class="text-xs font-semibold text-slate-600">Categoría</label>
                <select id="fix-cat" class="w-full mt-1 px-3.5 py-2.5 rounded-xl border border-slate-200 text-sm bg-white">
                  ${CATEGORIES.gasto.map(c => `<option value="${c.id}" ${item?.category === c.id ? 'selected' : ''}>${c.name}</option>`).join('')}
                </select>
              </div>
            </div>
            <button type="submit" class="w-full py-3 rounded-xl bg-emerald-500 text-slate-950 font-extrabold text-sm">
              Guardar gasto fijo
            </button>
          </form>
        </div>
      `);
    }

    function submitFixed(e, editId) {
      e.preventDefault();
      const payload = {
        id: editId || 'fix_' + Date.now(),
        name: document.getElementById('fix-name').value.trim(),
        amount: parseFloat(document.getElementById('fix-amount').value),
        dayOfMonth: parseInt(document.getElementById('fix-day').value, 10),
        accountId: document.getElementById('fix-account').value,
        leadDays: parseInt(document.getElementById('fix-lead').value, 10),
        category: document.getElementById('fix-cat').value,
        lastPaidMonth: editId ? (state.fixedExpenses.find(f => f.id === editId)?.lastPaidMonth || '') : ''
      };

      if (editId) {
        const idx = state.fixedExpenses.findIndex(f => f.id === editId);
        if (idx !== -1) state.fixedExpenses[idx] = payload;
      } else {
        state.fixedExpenses.push(payload);
      }
      saveState();
      closeModal();
      render();
      showToast('Gasto fijo guardado');
    }

    function deleteFixed(id) {
      state.fixedExpenses = state.fixedExpenses.filter(f => f.id !== id);
      saveState();
      render();
      showToast('Gasto fijo eliminado');
    }

    function payFixedNow(id) {
      const fix = state.fixedExpenses.find(f => f.id === id);
      if (!fix) return;
      applyBalanceChange('gasto', Number(fix.amount), fix.accountId, null, 1);
      fix.lastPaidMonth = currentMonthKey();
      state.transactions.unshift({
        id: 'tx_' + Date.now(),
        type: 'gasto',
        amount: Number(fix.amount),
        category: fix.category || 'servicios',
        accountId: fix.accountId,
        toAccountId: null,
        date: new Date().toISOString().slice(0, 10),
        note: `Pago fijo: ${fix.name}`
      });
      saveState();
      closeModal();
      render();
      showToast(`Pago registrado: ${fix.name}`);
    }

    function openFundModal(fixId) {
      const item = evaluateFixedExpenses().find(f => f.id === fixId);
      if (!item) return;
      const otherAccounts = state.accounts.filter(a => a.id !== item.accountId && a.type !== 'credito');

      openModal(`
        <div class="space-y-4">
          <div class="flex items-center justify-between">
            <h3 class="text-base font-extrabold text-slate-900">Fondear cuenta para pago</h3>
            <button onclick="closeModal()" class="p-1 text-slate-400"><i data-lucide="x" class="w-5 h-5"></i></button>
          </div>
          <p class="text-xs text-slate-600">
            Para cubrir <b>${item.name}</b> (${formatQ(item.amount)}) en <b>${item.account?.name}</b>, te faltan <b>${formatQ(item.shortage)}</b>.
          </p>
          <form onsubmit="submitFund(event, '${item.accountId}', '${item.name}')" class="space-y-3">
            <div>
              <label class="text-xs font-semibold text-slate-600">Transferir desde</label>
              <select id="fund-from" class="w-full mt-1 px-3.5 py-2.5 rounded-xl border border-slate-200 text-sm bg-white">
                ${otherAccounts.map(a => `<option value="${a.id}">${a.name} (${formatQ(a.balance)})</option>`).join('')}
              </select>
            </div>
            <div>
              <label class="text-xs font-semibold text-slate-600">Monto a transferir (Q)</label>
              <input id="fund-amount" type="number" step="0.01" min="0.01" required value="${item.shortage.toFixed(2)}" class="w-full mt-1 px-3.5 py-2.5 rounded-xl border border-slate-200 text-base font-extrabold" />
            </div>
            <button type="submit" class="w-full py-3 rounded-xl bg-slate-900 text-white font-bold text-sm">
              Transferir ahora
            </button>
          </form>
        </div>
      `);
    }

    function submitFund(e, targetAccId, fixName) {
      e.preventDefault();
      const fromId = document.getElementById('fund-from').value;
      const amount = parseFloat(document.getElementById('fund-amount').value);
      if (!fromId) return;
      applyBalanceChange('transferencia', amount, fromId, targetAccId, 1);
      state.transactions.unshift({
        id: 'tx_' + Date.now(),
        type: 'transferencia',
        amount,
        category: 'transfer',
        accountId: fromId,
        toAccountId: targetAccId,
        date: new Date().toISOString().slice(0, 10),
        note: `Fondeo para ${fixName}`
      });
      saveState();
      closeModal();
      render();
      showToast('Cuenta fondeada con éxito');
    }

    function openAccountModal(editId = null) {
      const acc = editId ? state.accounts.find(a => a.id === editId) : null;
      openModal(`
        <div class="space-y-4">
          <div class="flex items-center justify-between">
            <h3 class="text-base font-extrabold text-slate-900">${acc ? 'Editar cuenta' : 'Nueva cuenta'}</h3>
            <button onclick="closeModal()" class="p-1 text-slate-400"><i data-lucide="x" class="w-5 h-5"></i></button>
          </div>
          <form onsubmit="submitAccount(event, '${editId || ''}')" class="space-y-3">
            <div>
              <label class="text-xs font-semibold text-slate-600">Nombre de la cuenta</label>
              <input id="acc-name" required value="${acc?.name || ''}" placeholder="Ej. Banrural, Efectivo, Tarjeta Visa" class="w-full mt-1 px-3.5 py-2.5 rounded-xl border border-slate-200 text-sm" />
            </div>
            <div class="grid grid-cols-2 gap-2.5">
              <div>
                <label class="text-xs font-semibold text-slate-600">Tipo</label>
                <select id="acc-type" class="w-full mt-1 px-3.5 py-2.5 rounded-xl border border-slate-200 text-sm bg-white">
                  <option value="debito" ${acc?.type === 'debito' ? 'selected' : ''}>Débito / Banco</option>
                  <option value="efectivo" ${acc?.type === 'efectivo' ? 'selected' : ''}>Efectivo</option>
                  <option value="ahorro" ${acc?.type === 'ahorro' ? 'selected' : ''}>Ahorros</option>
                  <option value="credito" ${acc?.type === 'credito' ? 'selected' : ''}>Tarjeta Crédito</option>
                </select>
              </div>
              <div>
                <label class="text-xs font-semibold text-slate-600">Saldo actual (Q)</label>
                <input id="acc-balance" type="number" step="0.01" required value="${acc ? acc.balance : '0'}" class="w-full mt-1 px-3.5 py-2.5 rounded-xl border border-slate-200 text-sm font-bold" />
              </div>
            </div>
            <div>
              <label class="text-xs font-semibold text-slate-600">Límite de crédito (solo si es tarjeta)</label>
              <input id="acc-limit" type="number" step="0.01" value="${acc?.limit || 0}" class="w-full mt-1 px-3.5 py-2.5 rounded-xl border border-slate-200 text-sm" />
            </div>
            <button type="submit" class="w-full py-3 rounded-xl bg-emerald-500 text-slate-950 font-extrabold text-sm">
              Guardar cuenta
            </button>
          </form>
        </div>
      `);
    }

    function submitAccount(e, editId) {
      e.preventDefault();
      const colors = ['from-slate-800 to-slate-950', 'from-emerald-600 to-teal-800', 'from-blue-600 to-indigo-800', 'from-purple-600 to-violet-900'];
      const payload = {
        id: editId || 'acc_' + Date.now(),
        name: document.getElementById('acc-name').value.trim(),
        type: document.getElementById('acc-type').value,
        balance: parseFloat(document.getElementById('acc-balance').value || 0),
        limit: parseFloat(document.getElementById('acc-limit').value || 0),
        color: editId ? (state.accounts.find(a => a.id === editId)?.color || colors[0]) : colors[state.accounts.length % colors.length]
      };
      if (editId) {
        const idx = state.accounts.findIndex(a => a.id === editId);
        if (idx !== -1) state.accounts[idx] = payload;
      } else {
        state.accounts.push(payload);
      }
      saveState();
      closeModal();
      render();
      showToast('Cuenta guardada');
    }

    function openAlertsModal() {
      const alerts = evaluateFixedExpenses().filter(f => f.needsAlert);
      openModal(`
        <div class="space-y-4">
          <div class="flex items-center justify-between">
            <h3 class="text-base font-extrabold text-slate-900">Centro de alertas</h3>
            <button onclick="closeModal()" class="p-1 text-slate-400"><i data-lucide="x" class="w-5 h-5"></i></button>
          </div>
          ${alerts.length === 0
            ? `<p class="text-xs text-slate-500 py-4 text-center">Todo en orden. Ningún cobro fijo próximo tiene saldo insuficiente.</p>`
            : alerts.map(a => `
              <div class="p-3.5 rounded-2xl bg-rose-50 border border-rose-200 space-y-2">
                <p class="font-bold text-sm text-rose-900">${a.name} — ${formatQ(a.amount)}</p>
                <p class="text-xs text-rose-700">Cobra el día ${a.dayOfMonth} (en ${a.daysLeft}d) en <b>${a.account?.name}</b>, pero faltan <b>${formatQ(a.shortage)}</b>.</p>
                <button onclick="openFundModal('${a.id}')" class="px-3 py-1.5 rounded-lg bg-slate-900 text-white text-xs font-semibold">Fondear cuenta</button>
              </div>
            `).join('')
          }
        </div>
      `);
    }

    function requestNotifPermission() {
      if (!('Notification' in window)) {
        showToast('Este navegador no soporta notificaciones push');
        return;
      }
      Notification.requestPermission().then(perm => {
        render();
        if (perm === 'granted') {
          new Notification('Bóveda Q — Alertas activas', {
            body: 'Te avisaremos antes de tus cobros fijos si alguna cuenta no tiene saldo suficiente.'
          });
          showToast('Notificaciones activadas');
        } else {
          showToast('Permiso de notificaciones denegado');
        }
      });
    }

    function exportBackup() {
      const blob = new Blob([JSON.stringify(state, null, 2)], { type: 'application/json' });
      const a = document.createElement('a');
      a.href = URL.createObjectURL(blob);
      a.download = `respaldo-boveda-q-${new Date().toISOString().slice(0, 10)}.json`;
      a.click();
      showToast('Respaldo descargado');
    }

    function importBackup(event) {
      const file = event.target.files[0];
      if (!file) return;
      const reader = new FileReader();
      reader.onload = e => {
        try {
          const parsed = JSON.parse(e.target.result);
          if (parsed.accounts && parsed.transactions) {
            state = { ...getDefaultState(), ...parsed };
            saveState();
            render();
            showToast('Respaldo restaurado correctamente');
          } else {
            showToast('Archivo de respaldo no válido');
          }
        } catch {
          showToast('Error al leer el archivo');
        }
      };
      reader.readAsText(file);
    }

    window.addEventListener('online', () => document.getElementById('offline-badge').classList.add('hidden'));
    window.addEventListener('offline', () => document.getElementById('offline-badge').classList.remove('hidden'));

    render();
  </script>
</body>
</html>