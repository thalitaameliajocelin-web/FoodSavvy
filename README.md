# FoodSavvy
​FoodSavvy—Aplikasi perencana makanan cerdas berbasis AI untuk mengurangi sampah makanan (zero-waste), mengelola stok pantry otomatis, serta melacak anggaran dan nutrisi harian.
```html
<!DOCTYPE html>
<html lang="id">
<head>
  <meta charset="UTF-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1.0" />
  <title>FoodSavvy — AI Zero-Waste Pantry & Meal Planner</title>
  <!-- Tailwind CSS -->
  <script src="https://cdn.tailwindcss.com"></script>
  <!-- Google Fonts: Inter -->
  <link href="https://fonts.googleapis.com/css2?family=Inter:wght@400;500;600;700;800&display=swap" rel="stylesheet">
  <!-- Font Awesome untuk Ikon Bersih -->
  <link rel="stylesheet" href="https://cdnjs.cloudflare.com/ajax/libs/font-awesome/6.4.0/css/all.min.css" />
  <style>
    body {
      font-family: 'Inter', sans-serif;
      background-color: #F8FAFC;
      color: #0F172A;
    }
    .glass-card {
      background: rgba(255, 255, 255, 0.95);
      backdrop-filter: blur(12px);
      border: 1px solid rgba(226, 232, 240, 0.9);
    }
    .tab-active {
      background-color: #10B981;
      color: #FFFFFF;
      box-shadow: 0 4px 12px -2px rgba(16, 185, 129, 0.4);
    }
    @keyframes scanline {
      0% { top: 4%; }
      50% { top: 94%; }
      100% { top: 4%; }
    }
    .scanner-line {
      position: absolute;
      left: 0;
      right: 0;
      height: 3px;
      background: linear-gradient(90deg, transparent, #10B981, transparent);
      box-shadow: 0 0 14px #10B981;
      animation: scanline 2.4s infinite ease-in-out;
    }
    /* Custom scrollbar untuk tabel & navigasi mobile */
    ::-webkit-scrollbar {
      width: 6px;
      height: 6px;
    }
    ::-webkit-scrollbar-track {
      background: #F1F5F9;
    }
    ::-webkit-scrollbar-thumb {
      background: #CBD5E1;
      border-radius: 9999px;
    }
  </style>
</head>
<body class="min-h-screen flex flex-col selection:bg-emerald-100 selection:text-emerald-900">

  <!-- Top Navigation Bar -->
  <header class="sticky top-0 z-30 glass-card border-b border-slate-200/80 px-4 sm:px-6 lg:px-8 py-3.5">
    <div class="max-w-7xl mx-auto flex flex-col md:flex-row md:items-center justify-between gap-3">
      <!-- Brand Logo -->
      <div class="flex items-center justify-between">
        <div class="flex items-center gap-3">
          <div class="w-10 h-10 rounded-xl bg-gradient-to-br from-emerald-500 to-teal-600 flex items-center justify-center text-white font-extrabold text-xl shadow-md">
            <i class="fa-solid fa-leaf"></i>
          </div>
          <div>
            <h1 class="text-xl font-extrabold tracking-tight text-slate-900">Food<span class="text-emerald-600">Savvy</span></h1>
            <p class="text-[11px] text-slate-500 font-medium">AI Zero-Waste Pantry & Meal Planner</p>
          </div>
        </div>
        <!-- Reset Demo Data Button (Mobile/Desktop) -->
        <button onclick="resetDemoData()" title="Reset Data Demo ke Awal" class="md:hidden text-xs font-semibold text-slate-500 hover:text-emerald-600 bg-slate-100 px-3 py-1.5 rounded-lg">
          <i class="fa-solid fa-rotate-right mr-1"></i> Reset
        </button>
      </div>

      <!-- Navigation Tabs -->
      <div class="flex items-center gap-2 overflow-x-auto pb-1 md:pb-0">
        <nav class="flex items-center bg-slate-100 p-1 rounded-xl gap-1 shrink-0">
          <button onclick="switchTab('dashboard')" id="nav-dashboard" class="tab-btn tab-active px-3.5 py-2 rounded-lg text-xs sm:text-sm font-semibold transition-all flex items-center gap-1.5">
            <i class="fa-solid fa-chart-pie"></i> Beranda
          </button>
          <button onclick="switchTab('scanner')" id="nav-scanner" class="tab-btn text-slate-600 hover:text-slate-900 px-3.5 py-2 rounded-lg text-xs sm:text-sm font-semibold transition-all flex items-center gap-1.5">
            <i class="fa-solid fa-camera"></i> Pantry Scanner
          </button>
          <button onclick="switchTab('planner')" id="nav-planner" class="tab-btn text-slate-600 hover:text-slate-900 px-3.5 py-2 rounded-lg text-xs sm:text-sm font-semibold transition-all flex items-center gap-1.5">
            <i class="fa-solid fa-wand-magic-sparkles"></i> Smart Meal Planner
          </button>
          <button onclick="switchTab('shopping')" id="nav-shopping" class="tab-btn text-slate-600 hover:text-slate-900 px-3.5 py-2 rounded-lg text-xs sm:text-sm font-semibold transition-all flex items-center gap-1.5">
            <i class="fa-solid fa-cart-shopping"></i> Auto-Shopping
            <span id="shopping-badge" class="bg-amber-500 text-white text-[11px] font-bold px-1.5 py-0.5 rounded-full">0</span>
          </button>
        </nav>
        <button onclick="resetDemoData()" title="Reset Data Demo" class="hidden md:flex items-center gap-1.5 text-xs font-semibold text-slate-500 hover:text-emerald-700 bg-slate-100 hover:bg-emerald-50 px-3 py-2.5 rounded-xl transition shrink-0">
          <i class="fa-solid fa-rotate-right"></i> Reset
        </button>
      </div>
    </div>
  </header>

  <!-- Main Content -->
  <main class="flex-1 max-w-7xl w-full mx-auto px-4 sm:px-6 lg:px-8 py-6 space-y-6">

    <!-- 2. EXPIRY ALERT BANNER (Dinamis berdasarkan isi kulkas) -->
    <div id="expiry-alert" class="bg-amber-50 border border-amber-200 rounded-2xl p-4 flex flex-col sm:flex-row items-start sm:items-center justify-between gap-3 shadow-sm transition-all">
      <div class="flex items-start sm:items-center gap-3.5">
        <div class="w-10 h-10 rounded-xl bg-amber-100 text-amber-600 flex items-center justify-center font-bold text-lg shrink-0">
          <i class="fa-solid fa-triangle-exclamation"></i>
        </div>
        <div>
          <p class="text-sm font-bold text-amber-950">Peringatan Kadaluarsa Dini (Expiry Alert — 24–48 Jam)</p>
          <p id="expiry-alert-text" class="text-xs text-amber-800 mt-0.5 leading-relaxed">
            Memeriksa stok bahan kritis di kulkas...
          </p>
        </div>
      </div>
      <button onclick="switchTab('planner')" class="px-4 py-2.5 bg-amber-600 hover:bg-amber-700 text-white text-xs font-bold rounded-xl transition shrink-0 shadow-sm">
        Lihat Resep Prioritas <i class="fa-solid fa-arrow-right ml-1"></i>
      </button>
    </div>

    <!-- ===================================================================== -->
    <!-- TAB 1: BERANDA (HOME DASHBOARD) -->
    <!-- ===================================================================== -->
    <section id="tab-dashboard" class="space-y-6">
      <!-- KPI Cards -->
      <div class="grid grid-cols-1 lg:grid-cols-3 gap-5">
        
        <!-- Card 1: Target Kalori & Makronutrisi Harian -->
        <div class="glass-card rounded-2xl p-5 shadow-sm flex flex-col justify-between">
          <div>
            <div class="flex items-center justify-between mb-3">
              <span class="text-xs font-bold uppercase tracking-wider text-slate-400">Target Kalori & Gizi Harian</span>
              <span id="cal-status-badge" class="text-xs font-semibold bg-emerald-50 text-emerald-700 px-2.5 py-1 rounded-full">On Track</span>
            </div>
            <div class="flex items-baseline justify-between">
              <div class="flex items-baseline gap-1.5">
                <span id="cal-consumed" class="text-3xl font-extrabold text-slate-900">1.250</span>
                <span class="text-sm font-medium text-slate-500">/ 2.000 kkal</span>
              </div>
              <button onclick="resetDailyNutrition()" class="text-[11px] font-semibold text-slate-400 hover:text-emerald-600">Reset Hari</button>
            </div>
            <div class="w-full bg-slate-100 h-2.5 rounded-full mt-3 overflow-hidden">
              <div id="cal-bar" class="bg-emerald-500 h-full rounded-full transition-all duration-500" style="width: 62%"></div>
            </div>
          </div>

          <div class="grid grid-cols-4 gap-2 mt-4 pt-3.5 border-t border-slate-100 text-center">
            <div class="bg-slate-50 rounded-xl p-2">
              <p class="text-[10px] font-medium text-slate-400">Protein</p>
              <p id="macro-protein" class="text-xs font-extrabold text-emerald-700 mt-0.5">68g</p>
            </div>
            <div class="bg-slate-50 rounded-xl p-2">
              <p class="text-[10px] font-medium text-slate-400">Karbo</p>
              <p id="macro-carbs" class="text-xs font-extrabold text-slate-800 mt-0.5">135g</p>
            </div>
            <div class="bg-slate-50 rounded-xl p-2">
              <p class="text-[10px] font-medium text-slate-400">Lemak</p>
              <p id="macro-fat" class="text-xs font-extrabold text-slate-800 mt-0.5">38g</p>
            </div>
            <div class="bg-slate-50 rounded-xl p-2">
              <p class="text-[10px] font-medium text-slate-400">Serat</p>
              <p id="macro-fiber" class="text-xs font-extrabold text-teal-700 mt-0.5">18g</p>
            </div>
          </div>
        </div>

        <!-- Card 2: Pelacak Anggaran Belanja Bulanan & Penghematan -->
        <div class="glass-card rounded-2xl p-5 shadow-sm flex flex-col justify-between">
          <div>
            <div class="flex items-center justify-between mb-3">
              <span class="text-xs font-bold uppercase tracking-wider text-slate-400">Anggaran Belanja Bulanan</span>
              <span class="text-xs font-semibold bg-blue-50 text-blue-700 px-2.5 py-1 rounded-full">Efisien</span>
            </div>
            <div class="flex items-baseline gap-1.5">
              <span id="budget-spent" class="text-3xl font-extrabold text-slate-900">Rp 840.000</span>
              <span class="text-sm font-medium text-slate-500">/ Rp 1,5 Jt</span>
            </div>
            <div class="w-full bg-slate-100 h-2.5 rounded-full mt-3 overflow-hidden">
              <div id="budget-bar" class="bg-blue-500 h-full rounded-full transition-all duration-500" style="width: 56%"></div>
            </div>
          </div>

          <div class="mt-4 pt-3.5 border-t border-slate-100 flex items-center justify-between">
            <div>
              <p class="text-[11px] text-slate-500">Total Penghematan Zero-Waste:</p>
              <p id="zero-waste-savings" class="text-sm font-extrabold text-emerald-600">+Rp 235.000</p>
            </div>
            <div class="w-9 h-9 rounded-xl bg-emerald-50 text-emerald-600 flex items-center justify-center">
              <i class="fa-solid fa-piggy-bank"></i>
            </div>
          </div>
        </div>

        <!-- Card 3: Ringkasan Status Stok Kulkas -->
        <div class="glass-card rounded-2xl p-5 shadow-sm flex flex-col justify-between">
          <div>
            <div class="flex items-center justify-between mb-3">
              <span class="text-xs font-bold uppercase tracking-wider text-slate-400">Status Stok Kulkas</span>
              <button onclick="switchTab('scanner')" class="text-xs font-bold text-emerald-600 hover:text-emerald-700 bg-emerald-50 px-2.5 py-1 rounded-lg transition">
                + Tambah Stok
              </button>
            </div>
            <div class="flex items-baseline gap-2">
              <span id="total-items-count" class="text-3xl font-extrabold text-slate-900">0</span>
              <span class="text-sm font-medium text-slate-500">Bahan Tersimpan</span>
            </div>
            <p class="text-xs text-slate-400 mt-1">Dipantau otomatis dengan kalibrasi masa simpan</p>
          </div>

          <div class="grid grid-cols-3 gap-2 mt-4 pt-3.5 border-t border-slate-100 text-center text-xs font-bold">
            <div class="px-2 py-2 rounded-xl bg-emerald-50 text-emerald-700" id="count-green">0 Aman</div>
            <div class="px-2 py-2 rounded-xl bg-amber-50 text-amber-700" id="count-yellow">0 Segera</div>
            <div class="px-2 py-2 rounded-xl bg-rose-50 text-rose-700" id="count-red">0 Kritis</div>
          </div>
        </div>
      </div>

      <!-- Papan Manajemen Stok Pantry (Pantry Management) -->
      <div class="glass-card rounded-2xl p-5 sm:p-6 shadow-sm">
        <div class="flex flex-col sm:flex-row sm:items-center justify-between gap-3 mb-5">
          <div>
            <h2 class="text-lg font-extrabold text-slate-900">Papan Manajemen Stok Dapur (Pantry Management)</h2>
            <p class="text-xs text-slate-500 mt-0.5">Kelola kesegaran bahan makanan, sisa masa simpan, dan panduan penyimpanan optimal.</p>
          </div>
          <div class="flex items-center gap-2">
            <label for="filter-status" class="text-xs font-semibold text-slate-500">Filter:</label>
            <select id="filter-status" onchange="renderPantry()" class="text-xs font-bold bg-slate-100 border border-slate-200 rounded-xl px-3.5 py-2.5 text-slate-700 focus:outline-none focus:border-emerald-500">
              <option value="all">Semua Status</option>
              <option value="red">Kritis / Hampir Rusak (≤1 Hari)</option>
              <option value="yellow">Segera Masak (2–3 Hari)</option>
              <option value="green">Aman (>3 Hari)</option>
            </select>
          </div>
        </div>

        <div class="overflow-x-auto">
          <table class="w-full text-left border-collapse min-w-[720px]">
            <thead>
              <tr class="border-b border-slate-200 text-[11px] font-bold uppercase tracking-wider text-slate-400">
                <th class="pb-3.5">Nama Bahan</th>
                <th class="pb-3.5">Kategori</th>
                <th class="pb-3.5">Jumlah</th>
                <th class="pb-3.5">Sisa Masa Simpan</th>
                <th class="pb-3.5">Status</th>
                <th class="pb-3.5">Tips Penyimpanan</th>
                <th class="pb-3.5 text-right">Aksi</th>
              </tr>
            </thead>
            <tbody id="pantry-table-body" class="divide-y divide-slate-100 text-sm">
              <!-- Diisi oleh JavaScript -->
            </tbody>
          </table>
        </div>
      </div>
    </section>

    <!-- ===================================================================== -->
    <!-- TAB 2: PANTRY SCANNER (OCR & AI VISION) -->
    <!-- ===================================================================== -->
    <section id="tab-scanner" class="hidden space-y-6">
      <div class="grid grid-cols-1 lg:grid-cols-2 gap-6">
        
        <!-- Kamera / Upload Struk OCR & AI Vision -->
        <div class="glass-card rounded-2xl p-6 shadow-sm flex flex-col justify-between">
          <div>
            <div class="flex items-center justify-between">
              <h2 class="text-lg font-extrabold text-slate-900">Pantry Scanner (OCR & AI Vision)</h2>
              <span class="text-[11px] font-bold bg-emerald-50 text-emerald-700 px-2.5 py-1 rounded-lg">AI Powered</span>
            </div>
            <p class="text-xs text-slate-500 mt-1">Unggah foto struk belanja/bahan makanan atau jalankan simulasi pemindaian otomatis ke stok dapur.</p>
          </div>

          <!-- Area Viewfinder Scanner -->
          <div class="relative my-5 h-64 rounded-2xl bg-slate-900 overflow-hidden flex flex-col items-center justify-center text-center p-6 border-2 border-dashed border-emerald-500/60">
            <img id="scanner-preview-img" class="hidden absolute inset-0 w-full h-full object-cover opacity-45" alt="Preview Scan" />
            <div class="scanner-line"></div>
            <div class="relative z-10 flex flex-col items-center">
              <div id="scanner-icon-box" class="w-12 h-12 rounded-2xl bg-emerald-500/20 text-emerald-400 flex items-center justify-center mb-3 text-xl">
                <i class="fa-solid fa-expand"></i>
              </div>
              <p class="text-sm font-bold text-white" id="scanner-status-text">Kamera & OCR Siap Memindai</p>
              <p class="text-xs text-slate-300 mt-1 max-w-xs" id="scanner-sub-text">Pilih foto struk/kulkas dari perangkat Anda atau gunakan tombol simulasi instan di bawah.</p>
              
              <!-- Input File Gambar Nyata -->
              <label class="mt-4 cursor-pointer bg-white/15 hover:bg-white/25 backdrop-blur-md text-white text-xs font-bold px-4 py-2.5 rounded-xl border border-white/20 transition">
                <i class="fa-solid fa-upload mr-1.5"></i> Unggah Foto Struk / Bahan
                <input type="file" accept="image/*" onchange="handleImageUpload(event)" class="hidden" />
              </label>
            </div>
          </div>

          <div class="grid grid-cols-1 sm:grid-cols-2 gap-3">
            <button onclick="simulateScanReceipt()" class="bg-emerald-600 hover:bg-emerald-700 text-white font-bold text-xs sm:text-sm py-3 px-4 rounded-xl transition shadow-sm flex items-center justify-center gap-2">
              <i class="fa-solid fa-receipt"></i> Simulasi OCR Struk Belanja
            </button>
            <button onclick="simulateScanFridge()" class="bg-slate-800 hover:bg-slate-900 text-white font-bold text-xs sm:text-sm py-3 px-4 rounded-xl transition shadow-sm flex items-center justify-center gap-2">
              <i class="fa-solid fa-snowflake"></i> Simulasi Vision Isi Kulkas
            </button>
          </div>
        </div>

        <!-- Form Input Manual dengan Kalibrasi Shelf-Life Otomatis -->
        <div class="glass-card rounded-2xl p-6 shadow-sm">
          <h2 class="text-lg font-extrabold text-slate-900">Input Manual & Kalibrasi Masa Simpan</h2>
          <p class="text-xs text-slate-500 mt-1">Pilih kategori pangan untuk mengkalibrasi estimasi masa simpan (*shelf-life*) secara otomatis.</p>

          <form onsubmit="addManualItem(event)" class="mt-5 space-y-4">
            <div>
              <label class="block text-xs font-bold text-slate-700 mb-1.5">Nama Bahan Makanan</label>
              <input type="text" id="item-name" required placeholder="Contoh: Brokoli Segar, Ikan Salmon, Tahu Putih" class="w-full px-3.5 py-2.5 rounded-xl border border-slate-200 text-sm focus:outline-none focus:border-emerald-500" />
            </div>

            <div class="grid grid-cols-1 sm:grid-cols-2 gap-4">
              <div>
                <label class="block text-xs font-bold text-slate-700 mb-1.5">Kategori Pangan</label>
                <select id="item-category" onchange="autoCalibrateShelfLife()" class="w-full px-3.5 py-2.5 rounded-xl border border-slate-200 text-sm focus:outline-none focus:border-emerald-500 bg-white">
                  <option value="Sayur">Sayur (Kalibrasi: 4 Hari)</option>
                  <option value="Daging">Daging & Unggas (Kalibrasi: 2 Hari)</option>
                  <option value="Buah">Buah (Kalibrasi: 5 Hari)</option>
                  <option value="Bumbu">Bumbu & Bahan Kering (Kalibrasi: 14 Hari)</option>
                </select>
              </div>
              <div>
                <label class="block text-xs font-bold text-slate-700 mb-1.5">Jumlah / Berat</label>
                <input type="text" id="item-qty" required placeholder="Contoh: 250 gr / 2 ikat" class="w-full px-3.5 py-2.5 rounded-xl border border-slate-200 text-sm focus:outline-none focus:border-emerald-500" />
              </div>
            </div>

            <div>
              <div class="flex items-center justify-between mb-1.5">
                <label class="block text-xs font-bold text-slate-700">Estimasi Sisa Masa Simpan (Hari)</label>
                <span id="shelf-life-hint" class="text-[11px] font-semibold text-emerald-600">Otomatis sesuai kategori Sayur</span>
              </div>
              <input type="number" id="item-days" min="1" max="90" value="4" required class="w-full px-3.5 py-2.5 rounded-xl border border-slate-200 text-sm focus:outline-none focus:border-emerald-500" />
            </div>

            <button type="submit" class="w-full bg-emerald-600 hover:bg-emerald-700 text-white font-bold text-sm py-3 rounded-xl transition shadow-sm mt-2">
              <i class="fa-solid fa-plus mr-1.5"></i> Simpan ke Stok Pantry
            </button>
          </form>
        </div>

      </div>
    </section>

    <!-- ===================================================================== -->
    <!-- TAB 3: SMART MEAL PLANNER (AI ZERO-WASTE) -->
    <!-- ===================================================================== -->
    <section id="tab-planner" class="hidden space-y-6">
      <div class="glass-card rounded-2xl p-5 shadow-sm flex flex-col md:flex-row md:items-center justify-between gap-4">
        <div>
          <div class="flex items-center gap-2">
            <h2 class="text-lg font-extrabold text-slate-900">Smart Meal Planner (AI Zero-Waste)</h2>
            <span class="text-[11px] font-bold bg-emerald-100 text-emerald-800 px-2.5 py-0.5 rounded-lg">Masak &lt; 30 Menit</span>
          </div>
          <p class="text-xs text-slate-500 mt-1">
            Memprioritaskan bahan berstatus <span class="font-bold text-rose-600">Kritis</span> & <span class="font-bold text-amber-600">Segera Masak</span> di kulkas Anda lengkap dengan rincian gizi & biaya per porsi.
          </p>
        </div>

        <!-- Tombol Generate Resep AI dengan Gemini API -->
        <button id="btn-generate-ai" onclick="generateAIRecipeFromPantry()" class="bg-gradient-to-r from-emerald-600 to-teal-600 hover:from-emerald-700 hover:to-teal-700 text-white font-bold text-xs sm:text-sm px-4 py-3 rounded-xl shadow-md transition flex items-center justify-center gap-2 shrink-0">
          <i class="fa-solid fa-wand-magic-sparkles"></i>
          <span>Generate Resep AI dari Stok Kulkas</span>
        </button>
      </div>

      <!-- Grid Kartu Resep -->
      <div id="recipe-grid" class="grid grid-cols-1 md:grid-cols-2 lg:grid-cols-3 gap-6">
        <!-- Diisi oleh JavaScript -->
      </div>
    </section>

    <!-- ===================================================================== -->
    <!-- TAB 4: AUTO-SHOPPING LIST -->
    <!-- ===================================================================== -->
    <section id="tab-shopping" class="hidden space-y-6">
      <div class="glass-card rounded-2xl p-6 shadow-sm max-w-3xl mx-auto">
        <div class="flex flex-col sm:flex-row sm:items-center justify-between gap-3 mb-5">
          <div>
            <h2 class="text-lg font-extrabold text-slate-900">Auto-Shopping List</h2>
            <p class="text-xs text-slate-500 mt-0.5">Hanya mencatat kekurangan bahan dari resep pilihan agar belanja lebih hemat & tanpa duplikasi.</p>
          </div>
          <button onclick="clearCheckedShopping()" class="text-xs font-bold text-rose-600 hover:text-rose-700 bg-rose-50 px-3 py-2 rounded-xl transition self-start">
            <i class="fa-solid fa-trash-can mr-1"></i> Bersihkan yang Terbeli
          </button>
        </div>

        <!-- Form Tambah Item Belanja Manual -->
        <form onsubmit="addManualShoppingItem(event)" class="flex gap-2 mb-5">
          <input type="text" id="shopping-manual-input" required placeholder="Tambah item belanja manual (misal: Minyak Zaitun 250ml)..." class="flex-1 px-3.5 py-2.5 rounded-xl border border-slate-200 text-sm focus:outline-none focus:border-emerald-500" />
          <button type="submit" class="bg-emerald-600 hover:bg-emerald-700 text-white font-bold text-xs sm:text-sm px-4 py-2.5 rounded-xl transition shrink-0">
            + Tambah
          </button>
        </form>

        <ul id="shopping-list-container" class="divide-y divide-slate-100">
          <!-- Diisi oleh JavaScript -->
        </ul>
      </div>
    </section>

  </main>

  <!-- Toast Notification -->
  <div id="toast" class="fixed bottom-5 right-5 z-50 hidden bg-slate-900 text-white text-xs font-semibold px-4 py-3.5 rounded-xl shadow-xl border border-slate-700 flex items-center gap-2.5 transition-all">
    <i class="fa-solid fa-circle-check text-emerald-400 text-sm"></i>
    <span id="toast-message">Notifikasi</span>
  </div>

  <script>
    // =========================================================================
    // STATE MANAGEMENT & DATA AWAL (Tersimpan di localStorage)
    // =========================================================================
    const STORAGE_KEY = "foodsavvy_app_state_v1";

    const defaultState = {
      pantryItems: [
        { id: 1, name: "Dada Ayam Fillet", category: "Daging", qty: "300 gr", daysLeft: 1, tip: "Simpan di chiller suhu 0–4°C, segera olah dalam 24 jam." },
        { id: 2, name: "Bayam Segar", category: "Sayur", qty: "1 ikat", daysLeft: 2, tip: "Bungkus tisu dapur kering di dalam wadah tertutup." },
        { id: 3, name: "Tomat Merah", category: "Sayur", qty: "3 buah", daysLeft: 2, tip: "Simpan dengan posisi tangkai menghadap ke bawah." },
        { id: 4, name: "Telur Ayam", category: "Daging", qty: "6 butir", daysLeft: 10, tip: "Letakkan di rak tengah kulkas agar suhu stabil." },
        { id: 5, name: "Bawang Putih", category: "Bumbu", qty: "5 siung", daysLeft: 14, tip: "Simpan di wadah terbuka bersirkulasi udara kering." },
        { id: 6, name: "Pisang Cavendish", category: "Buah", qty: "4 buah", daysLeft: 5, tip: "Bungkus pangkal tandan dengan plastik wrap." }
      ],
      recipes: [
        {
          id: 101,
          title: "Tumis Dada Ayam Bayam Bawang Putih",
          priorityTag: "Prioritas Zero-Waste (Ayam & Bayam)",
          time: "20 Menit",
          costPerServing: "Rp 14.500",
          savingsValue: 18000,
          calories: 380,
          protein: 42,
          carbs: 12,
          fat: 16,
          fiber: 4,
          usedPantryIds: [1, 2],
          ingredientsOwned: ["Dada Ayam Fillet (300 gr)", "Bayam Segar (1 ikat)", "Bawang Putih (3 siung)"],
          missingIngredients: ["Saus Tiram (1 sdm)"]
        },
        {
          id: 102,
          title: "Orak-Arik Telur Tomat Segar",
          priorityTag: "Prioritas Zero-Waste (Tomat Merah)",
          time: "15 Menit",
          costPerServing: "Rp 9.000",
          savingsValue: 12000,
          calories: 290,
          protein: 18,
          carbs: 14,
          fat: 18,
          fiber: 3,
          usedPantryIds: [3],
          ingredientsOwned: ["Telur Ayam (3 butir)", "Tomat Merah (3 buah)", "Bawang Putih (2 siung)"],
          missingIngredients: ["Daun Bawang (1 batang)"]
        },
        {
          id: 103,
          title: "Smoothie Pisang Protein Ringan",
          priorityTag: "Menu Sarapan Cepat",
          time: "10 Menit",
          costPerServing: "Rp 11.000",
          savingsValue: 10000,
          calories: 240,
          protein: 10,
          carbs: 44,
          fat: 3,
          fiber: 5,
          usedPantryIds: [6],
          ingredientsOwned: ["Pisang Cavendish (2 buah)"],
          missingIngredients: ["Susu Rendah Lemak (200 ml)"]
        }
      ],
      shoppingList: [
        { id: 1, name: "Saus Tiram (1 botol kecil)", recipe: "Tumis Dada Ayam Bayam", checked: false },
        { id: 2, name: "Daun Bawang (1 ikat)", recipe: "Orak-Arik Telur Tomat", checked: false }
      ],
      nutrition: {
        calories: 1250,
        protein: 68,
        carbs: 135,
        fat: 38,
        fiber: 18
      },
      budget: {
        monthlyLimit: 1500000,
        spent: 840000,
        savings: 235000
      }
    };

    let appState = loadState();

    function loadState() {
      try {
        const saved = localStorage.getItem(STORAGE_KEY);
        if (saved) return JSON.parse(saved);
      } catch (e) {
        // Fallback ke defaultState jika storage diblokir
      }
      return JSON.parse(JSON.stringify(defaultState));
    }

    function saveState() {
      try {
        localStorage.setItem(STORAGE_KEY, JSON.stringify(appState));
      } catch (e) {}
    }

    function resetDemoData() {
      appState = JSON.parse(JSON.stringify(defaultState));
      saveState();
      renderAll();
      showToast("Data demo berhasil dikembalikan ke pengaturan awal.");
    }

    // =========================================================================
    // KALIBRASI MASA SIMPAN & STATUS KESEHATAN BAHAN
    // =========================================================================
    const categoryCalibration = {
      "Sayur": { days: 4, tip: "Bungkus tisu dapur kering di laci sayur kulkas." },
      "Daging": { days: 2, tip: "Simpan di chiller suhu 0–4°C dalam wadah tertutup rapat." },
      "Buah": { days: 5, tip: "Pisahkan buah penghasil etilen (pisang/apel) dari buah lain." },
      "Bumbu": { days: 14, tip: "Simpan di tempat kering sejuk dan terhindar dari sinar matahari." }
    };

    function autoCalibrateShelfLife() {
      const cat = document.getElementById("item-category").value;
      const info = categoryCalibration[cat] || { days: 4 };
      document.getElementById("item-days").value = info.days;
      document.getElementById("shelf-life-hint").textContent = `Otomatis sesuai kategori ${cat} (${info.days} hari)`;
    }

    function getStatusBadge(days) {
      if (days <= 1) return { key: "red", label: `Kritis (${days} Hari)`, cls: "bg-rose-100 text-rose-700 border border-rose-200" };
      if (days <= 3) return { key: "yellow", label: `Segera Masak (${days} Hari)`, cls: "bg-amber-100 text-amber-800 border border-amber-200" };
      return { key: "green", label: `Aman (${days} Hari)`, cls: "bg-emerald-100 text-emerald-700 border border-emerald-200" };
    }

    // =========================================================================
    // RENDER DASHBOARD, EXPIRY ALERT, PANTRY, RECIPES & SHOPPING LIST
    // =========================================================================
    function renderExpiryAlert() {
      const urgentItems = appState.pantryItems.filter(i => i.daysLeft <= 2);
      const alertBanner = document.getElementById("expiry-alert");
      const alertText = document.getElementById("expiry-alert-text");

      if (urgentItems.length === 0) {
        alertBanner.classList.add("hidden");
        return;
      }

      alertBanner.classList.remove("hidden");
      const names = urgentItems.map(i => `<span class="font-bold underline">${i.name} (${i.qty})</span>`).join(", ");
      alertText.innerHTML = `${names} akan melewati masa simpan dalam 24–48 jam. AI telah menyiapkan resep prioritas untuk menyelamatkannya!`;
    }

    function renderKPIs() {
      // Nutrisi
      const { calories, protein, carbs, fat, fiber } = appState.nutrition;
      document.getElementById("cal-consumed").textContent = calories.toLocaleString("id-ID");
      document.getElementById("macro-protein").textContent = `${protein}g`;
      document.getElementById("macro-carbs").textContent = `${carbs}g`;
      document.getElementById("macro-fat").textContent = `${fat}g`;
      document.getElementById("macro-fiber").textContent = `${fiber}g`;

      const calPct = Math.min(100, Math.round((calories / 2000) * 100));
      document.getElementById("cal-bar").style.width = `${calPct}%`;

      // Anggaran & Penghematan
      const { monthlyLimit, spent, savings } = appState.budget;
      document.getElementById("budget-spent").textContent = `Rp ${spent.toLocaleString("id-ID")}`;
      document.getElementById("zero-waste-savings").textContent = `+Rp ${savings.toLocaleString("id-ID")}`;
      const budgetPct = Math.min(100, Math.round((spent / monthlyLimit) * 100));
      document.getElementById("budget-bar").style.width = `${budgetPct}%`;
    }

    function renderPantry() {
      const tbody = document.getElementById("pantry-table-body");
      const filter = document.getElementById("filter-status").value;
      tbody.innerHTML = "";

      let greenCount = 0, yellowCount = 0, redCount = 0;

      // Urutkan dari yang paling cepat kedaluwarsa
      const sortedItems = [...appState.pantryItems].sort((a, b) => a.daysLeft - b.daysLeft);

      sortedItems.forEach(item => {
        const st = getStatusBadge(item.daysLeft);
        if (st.key === "green") greenCount++;
        if (st.key === "yellow") yellowCount++;
        if (st.key === "red") redCount++;

        if (filter !== "all" && filter !== st.key) return;

        const tr = document.createElement("tr");
        tr.className = "hover:bg-slate-50/90 transition";
        tr.innerHTML = `
          <td class="py-3.5 font-bold text-slate-900">${item.name}</td>
          <td class="py-3.5 text-slate-600 text-xs font-medium">${item.category}</td>
          <td class="py-3.5 text-slate-700 font-semibold text-xs">${item.qty}</td>
          <td class="py-3.5 text-slate-800 font-bold text-xs">${item.daysLeft} hari lagi</td>
          <td class="py-3.5">
            <span class="px-2.5 py-1 rounded-lg text-[11px] font-bold ${st.cls}">${st.label}</span>
          </td>
          <td class="py-3.5 text-xs text-slate-500 max-w-xs leading-relaxed">${item.tip}</td>
          <td class="py-3.5 text-right">
            <button onclick="markItemUsed(${item.id})" class="text-xs font-bold text-emerald-700 hover:text-white bg-emerald-50 hover:bg-emerald-600 px-3 py-1.5 rounded-lg transition">
              <i class="fa-solid fa-check mr-1"></i> Tandai Terpakai
            </button>
          </td>
        `;
        tbody.appendChild(tr);
      });

      if (tbody.children.length === 0) {
        tbody.innerHTML = `<tr><td colspan="7" class="py-8 text-center text-xs text-slate-400">Tidak ada bahan makanan untuk filter status ini.</td></tr>`;
      }

      document.getElementById("total-items-count").textContent = appState.pantryItems.length;
      document.getElementById("count-green").textContent = `${greenCount} Aman`;
      document.getElementById("count-yellow").textContent = `${yellowCount} Segera`;
      document.getElementById("count-red").textContent = `${redCount} Kritis`;

      renderExpiryAlert();
    }

    function renderRecipes() {
      const grid = document.getElementById("recipe-grid");
      grid.innerHTML = "";

      appState.recipes.forEach(r => {
        const missingList = r.missingIngredients && r.missingIngredients.length > 0
          ? r.missingIngredients.join(", ")
          : "Semua bahan sudah lengkap di kulkas!";

        const firstMissing = r.missingIngredients && r.missingIngredients.length > 0
          ? r.missingIngredients[0]
          : "";

        const card = document.createElement("div");
        card.className = "glass-card rounded-2xl p-5 shadow-sm flex flex-col justify-between hover:shadow-md transition";
        card.innerHTML = `
          <div>
            <div class="flex items-center justify-between gap-2 mb-3">
              <span class="text-[11px] font-bold bg-amber-100 text-amber-800 px-2.5 py-1 rounded-lg">${r.priorityTag}</span>
              <span class="text-xs font-bold text-slate-500 shrink-0"><i class="fa-regular fa-clock mr-1"></i>${r.time}</span>
            </div>
            <h3 class="text-base font-extrabold text-slate-900 leading-snug">${r.title}</h3>
            <p class="text-xs text-slate-500 mt-1">Estimasi Biaya: <span class="font-bold text-emerald-700">${r.costPerServing} / porsi</span></p>

            <!-- Rincian Makronutrisi Per Porsi -->
            <div class="grid grid-cols-5 gap-1.5 bg-slate-50 border border-slate-200/70 rounded-xl p-2.5 my-4 text-center">
              <div>
                <p class="text-[10px] text-slate-400">Kalori</p>
                <p class="text-xs font-extrabold text-slate-900">${r.calories}</p>
              </div>
              <div>
                <p class="text-[10px] text-slate-400">Protein</p>
                <p class="text-xs font-bold text-emerald-700">${r.protein}g</p>
              </div>
              <div>
                <p class="text-[10px] text-slate-400">Karbo</p>
                <p class="text-xs font-bold text-slate-800">${r.carbs}g</p>
              </div>
              <div>
                <p class="text-[10px] text-slate-400">Lemak</p>
                <p class="text-xs font-bold text-slate-800">${r.fat}g</p>
              </div>
              <div>
                <p class="text-[10px] text-slate-400">Serat</p>
                <p class="text-xs font-bold text-teal-700">${r.fiber}g</p>
              </div>
            </div>

            <div class="space-y-2.5 text-xs">
              <div class="bg-emerald-50/60 p-2.5 rounded-xl border border-emerald-100">
                <p class="font-bold text-emerald-900 mb-0.5"><i class="fa-solid fa-check-circle text-emerald-600 mr-1"></i> Bahan Tersedia di Kulkas:</p>
                <p class="text-emerald-800">${r.ingredientsOwned.join(", ")}</p>
              </div>
              <div class="bg-amber-50/60 p-2.5 rounded-xl border border-amber-100">
                <p class="font-bold text-amber-900 mb-0.5"><i class="fa-solid fa-basket-shopping text-amber-600 mr-1"></i> Bahan Kurang:</p>
                <p class="text-amber-800">${missingList}</p>
              </div>
            </div>
          </div>

          <div class="mt-5 pt-4 border-t border-slate-100 flex gap-2">
            <button onclick="cookRecipe(${r.id})" class="flex-1 bg-emerald-600 hover:bg-emerald-700 text-white font-bold text-xs py-2.5 px-3 rounded-xl transition shadow-sm">
              <i class="fa-solid fa-fire-burner mr-1"></i> Masak & Catat Kalori
            </button>
            ${firstMissing ? `
              <button onclick="addMissingToShopping('${firstMissing.replace(/'/g, "\\'")}', '${r.title.replace(/'/g, "\\'")}')" class="px-3 py-2.5 bg-slate-100 hover:bg-slate-200 text-slate-700 font-bold text-xs rounded-xl transition shrink-0">
                + Belanja
              </button>
            ` : ""}
          </div>
        `;
        grid.appendChild(card);
      });
    }

    function renderShoppingList() {
      const container = document.getElementById("shopping-list-container");
      container.innerHTML = "";

      if (appState.shoppingList.length === 0) {
        container.innerHTML = `<li class="py-8 text-center text-xs text-slate-400">Daftar belanja kosong. Semua kebutuhan dapur Anda terpenuhi!</li>`;
      } else {
        appState.shoppingList.forEach(item => {
          const li = document.createElement("li");
          li.className = "py-3.5 flex items-center justify-between gap-3";
          li.innerHTML = `
            <label class="flex items-center gap-3 cursor-pointer flex-1">
              <input type="checkbox" ${item.checked ? "checked" : ""} onchange="toggleShopping(${item.id})" class="w-4 h-4 rounded text-emerald-600 focus:ring-emerald-500 cursor-pointer" />
              <div>
                <p class="text-sm font-bold ${item.checked ? "line-through text-slate-400" : "text-slate-800"}">${item.name}</p>
                <p class="text-[11px] text-slate-500">Sumber: ${item.recipe}</p>
              </div>
            </label>
            <span class="text-[11px] font-bold px-2.5 py-1 rounded-lg shrink-0 ${item.checked ? "bg-emerald-50 text-emerald-700" : "bg-amber-50 text-amber-700"}">
              ${item.checked ? "Sudah Dibeli" : "Perlu Dibeli"}
            </span>
          `;
          container.appendChild(li);
        });
      }

      const pendingCount = appState.shoppingList.filter(i => !i.checked).length;
      document.getElementById("shopping-badge").textContent = pendingCount;
    }

    function renderAll() {
      renderKPIs();
      renderPantry();
      renderRecipes();
      renderShoppingList();
    }

    // =========================================================================
    // INTERAKSI FITUR: PANTRY SCANNER, COOKING, SHOPPING & GEMINI AI
    // =========================================================================
    function handleImageUpload(event) {
      const file = event.target.files[0];
      if (!file) return;

      const previewImg = document.getElementById("scanner-preview-img");
      previewImg.src = URL.createObjectURL(file);
      previewImg.classList.remove("hidden");

      document.getElementById("scanner-status-text").textContent = "Menganalisis Gambar dengan OCR & AI Vision...";
      document.getElementById("scanner-sub-text").textContent = `Memproses "${file.name}"...`;

      setTimeout(() => {
        appState.pantryItems.unshift(
          { id: Date.now(), name: "Paprika Merah Segar (Scan)", category: "Sayur", qty: "2 buah", daysLeft: 4, tip: categoryCalibration["Sayur"].tip },
          { id: Date.now() + 1, name: "Fillet Ikan Dori (Scan)", category: "Daging", qty: "250 gr", daysLeft: 2, tip: categoryCalibration["Daging"].tip }
        );
        saveState();
        renderAll();
        document.getElementById("scanner-status-text").textContent = "2 Bahan Berhasil Diekstrak!";
        document.getElementById("scanner-sub-text").textContent = "Paprika Merah & Fillet Ikan Dori masuk ke stok dapur.";
        showToast("Pemindaian foto selesai! 2 bahan ditambahkan ke Pantry.");
        switchTab("dashboard");
      }, 1100);
    }

    function simulateScanReceipt() {
      document.getElementById("scanner-status-text").textContent = "Memindai Teks Struk Belanja (OCR)...";
      setTimeout(() => {
        appState.pantryItems.unshift(
          { id: Date.now(), name: "Brokoli Hijau (OCR Struk)", category: "Sayur", qty: "400 gr", daysLeft: 4, tip: categoryCalibration["Sayur"].tip },
          { id: Date.now() + 1, name: "Tahu Putih Sutra (OCR Struk)", category: "Daging", qty: "2 kotak", daysLeft: 2, tip: "Rendam dalam air matang bersih di wadah tertutup." }
        );
        saveState();
        renderAll();
        document.getElementById("scanner-status-text").textContent = "OCR Struk Selesai!";
        showToast("OCR Berhasil: Brokoli & Tahu ditambahkan ke Pantry!");
        switchTab("dashboard");
      }, 800);
    }

    function simulateScanFridge() {
      document.getElementById("scanner-status-text").textContent = "Mendeteksi Objek Makanan (AI Vision)...";
      setTimeout(() => {
        appState.pantryItems.unshift(
          { id: Date.now(), name: "Wortel Organik (AI Vision)", category: "Sayur", qty: "3 buah", daysLeft: 5, tip: "Potong daun bagian atas agar kelembapan terjaga." }
        );
        saveState();
        renderAll();
        document.getElementById("scanner-status-text").textContent = "Objek Kulkas Terdeteksi!";
        showToast("AI Vision: Wortel Organik ditambahkan ke Pantry!");
        switchTab("dashboard");
      }, 800);
    }

    function addManualItem(e) {
      e.preventDefault();
      const name = document.getElementById("item-name").value.trim();
      const category = document.getElementById("item-category").value;
      const qty = document.getElementById("item-qty").value.trim();
      const daysLeft = parseInt(document.getElementById("item-days").value, 10);

      appState.pantryItems.unshift({
        id: Date.now(),
        name,
        category,
        qty,
        daysLeft,
        tip: categoryCalibration[category]?.tip || "Simpan dalam wadah kedap udara."
      });

      e.target.reset();
      autoCalibrateShelfLife();
      saveState();
      renderAll();
      showToast(`"${name}" ditambahkan ke stok dapur!`);
      switchTab("dashboard");
    }

    function markItemUsed(id) {
      const target = appState.pantryItems.find(i => i.id === id);
      appState.pantryItems = appState.pantryItems.filter(i => i.id !== id);
      if (target) {
        appState.budget.savings += 7500;
        showToast(`"${target.name}" ditandai terpakai (+Rp 7.500 hemat)!`);
      }
      saveState();
      renderAll();
    }

    function cookRecipe(recipeId) {
      const recipe = appState.recipes.find(r => r.id === recipeId);
      if (!recipe) return;

      // Tambah kalori & makronutrisi harian
      appState.nutrition.calories += recipe.calories;
      appState.nutrition.protein += recipe.protein;
      appState.nutrition.carbs += recipe.carbs;
      appState.nutrition.fat += recipe.fat;
      appState.nutrition.fiber += recipe.fiber;

      // Tambah penghematan zero-waste
      appState.budget.savings += (recipe.savingsValue || 15000);

      // Kurangi stok pantry yang digunakan resep
      if (recipe.usedPantryIds && recipe.usedPantryIds.length > 0) {
        appState.pantryItems = appState.pantryItems.filter(item => !recipe.usedPantryIds.includes(item.id));
      }

      saveState();
      renderAll();
      showToast(`Memasak "${recipe.title}" (+${recipe.calories} kkal tercatat & stok diperbarui)!`);
      switchTab("dashboard");
    }

    function resetDailyNutrition() {
      appState.nutrition = { calories: 0, protein: 0, carbs: 0, fat: 0, fiber: 0 };
      saveState();
      renderKPIs();
      showToast("Catatan kalori & gizi harian telah direset ke 0.");
    }

    function addMissingToShopping(itemName, recipeTitle) {
      if (!itemName) return;
      appState.shoppingList.unshift({
        id: Date.now(),
        name: itemName,
        recipe: recipeTitle,
        checked: false
      });
      saveState();
      renderShoppingList();
      showToast(`"${itemName}" dikirim ke Auto-Shopping List!`);
    }

    function addManualShoppingItem(e) {
      e.preventDefault();
      const input = document.getElementById("shopping-manual-input");
      const val = input.value.trim();
      if (!val) return;

      appState.shoppingList.unshift({
        id: Date.now(),
        name: val,
        recipe: "Input Manual",
        checked: false
      });
      input.value = "";
      saveState();
      renderShoppingList();
      showToast(`"${val}" ditambahkan ke daftar belanja.`);
    }

    function toggleShopping(id) {
      appState.shoppingList = appState.shoppingList.map(i => i.id === id ? { ...i, checked: !i.checked } : i);
      saveState();
      renderShoppingList();
    }

    function clearCheckedShopping() {
      const before = appState.shoppingList.length;
      appState.shoppingList = appState.shoppingList.filter(i => !i.checked);
      saveState();
      renderShoppingList();
      if (before !== appState.shoppingList.length) {
        showToast("Item belanja yang sudah dibeli telah dibersihkan.");
      }
    }

    // =========================================================================
    // INTEGRASI GEMINI API: GENERATE RESEP AI DARI STOK KULKAS TERKINI
    // =========================================================================
    async function generateAIRecipeFromPantry() {
      const btn = document.getElementById("btn-generate-ai");
      const originalHTML = btn.innerHTML;
      btn.disabled = true;
      btn.innerHTML = `<i class="fa-solid fa-spinner fa-spin"></i> <span>AI Meracik Resep...</span>`;

      const pantrySummary = appState.pantryItems
        .map(i => `${i.name} (${i.qty}, sisa ${i.daysLeft} hari)`)
        .join(", ");

      const prompt = `Berdasarkan stok kulkas berikut: [${pantrySummary}]. Buatkan 1 resep masakan Indonesia/sehat yang praktis (durasi masak di bawah 30 menit) dengan memprioritaskan bahan yang sisa harinya paling sedikit (zero-waste).`;

      const apiKey = "";
      const apiUrl = `https://generativelanguage.googleapis.com/v1beta/models/gemini-3-flash-preview:generateContent?key=${apiKey}`;

      const payload = {
        contents: [{ parts: [{ text: prompt }] }],
        generationConfig: {
          responseMimeType: "application/json",
          responseSchema: {
            type: "OBJECT",
            properties: {
              title: { type: "STRING" },
              priorityTag: { type: "STRING" },
              time: { type: "STRING" },
              costPerServing: { type: "STRING" },
              calories: { type: "INTEGER" },
              protein: { type: "INTEGER" },
              carbs: { type: "INTEGER" },
              fat: { type: "INTEGER" },
              fiber: { type: "INTEGER" },
              ingredientsOwned: { type: "ARRAY", items: { type: "STRING" } },
              missingIngredients: { type: "ARRAY", items: { type: "STRING" } }
            },
            required: ["title", "priorityTag", "time", "costPerServing", "calories", "protein", "carbs", "fat", "fiber", "ingredientsOwned", "missingIngredients"]
          }
        }
      };

      try {
        let response = null;
        let delay = 1000;
        for (let attempt = 0; attempt < 3; attempt++) {
          response = await fetch(apiUrl, {
            method: "POST",
            headers: { "Content-Type": "application/json" },
            body: JSON.stringify(payload)
          });
          if (response.ok) break;
          await new Promise(res => setTimeout(res, delay));
          delay *= 2;
        }

        if (!response || !response.ok) throw new Error("API Fallback");

        const result = await response.json();
        const rawJson = result?.candidates?.[0]?.content?.parts?.[0]?.text;
        const aiRecipe = JSON.parse(rawJson);

        appState.recipes.unshift({
          id: Date.now(),
          ...aiRecipe,
          savingsValue: 15000,
          usedPantryIds: []
        });

        saveState();
        renderRecipes();
        showToast(`Resep AI baru "${aiRecipe.title}" berhasil dibuat!`);
      } catch (err) {
        // Fallback cerdas berbasis stok kulkas aktual jika berjalan offline
        const urgent = [...appState.pantryItems].sort((a, b) => a.daysLeft - b.daysLeft).slice(0, 2);
        const ownedNames = urgent.length > 0 ? urgent.map(u => `${u.name} (${u.qty})`) : ["Telur Ayam (2 butir)"];
        const mainTitle = urgent.length > 0 ? `Tumis Sehat ${urgent.map(u => u.name.split(" ")[0]).join(" & ")}` : "Omelet Sayur Kilat";

        appState.recipes.unshift({
          id: Date.now(),
          title: mainTitle,
          priorityTag: "Rekomendasi AI Zero-Waste",
          time: "18 Menit",
          costPerServing: "Rp 12.000",
          savingsValue: 15000,
          calories: 320,
          protein: 24,
          carbs: 18,
          fat: 14,
          fiber: 5,
          usedPantryIds: urgent.map(u => u.id),
          ingredientsOwned: ownedNames,
          missingIngredients: ["Lada Hitam & Kecap Asin (1 sdm)"]
        });

        saveState();
        renderRecipes();
        showToast(`Resep AI "${mainTitle}" diracik dari stok kulkas Anda!`);
      } finally {
        btn.disabled = false;
        btn.innerHTML = originalHTML;
      }
    }

    // =========================================================================
    // NAVIGASI TAB & NOTIFIKASI TOAST
    // =========================================================================
    function switchTab(tabName) {
      ["dashboard", "scanner", "planner", "shopping"].forEach(t => {
        document.getElementById(`tab-${t}`).classList.add("hidden");
        const btn = document.getElementById(`nav-${t}`);
        btn.classList.remove("tab-active");
        btn.classList.add("text-slate-600");
      });

      document.getElementById(`tab-${tabName}`).classList.remove("hidden");
      const activeBtn = document.getElementById(`nav-${tabName}`);
      activeBtn.classList.add("tab-active");
      activeBtn.classList.remove("text-slate-600");
    }

    let toastTimer;
    function showToast(msg) {
      const toast = document.getElementById("toast");
      document.getElementById("toast-message").textContent = msg;
      toast.classList.remove("hidden");
      clearTimeout(toastTimer);
      toastTimer = setTimeout(() => toast.classList.add("hidden"), 3400);
    }

    // Jalankan inisialisasi saat halaman siap
    window.onload = function () {
      renderAll();
    };
  </script>
</body>
</html>
```
