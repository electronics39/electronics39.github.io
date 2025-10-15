<!DOCTYPE html>
<html lang="id">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Sistem Pendataan Barang - Teknik Elektronika Industri</title>
    <link href="https://cdn.jsdelivr.net/npm/tailwindcss@2.2.19/dist/tailwind.min.css" rel="stylesheet">
    <link href="https://cdnjs.cloudflare.com/ajax/libs/font-awesome/6.0.0/css/all.min.css" rel="stylesheet">
    <script src="https://cdn.jsdelivr.net/npm/qrcode@1.5.3/build/qrcode.min.js"></script>
    <script src="https://cdn.jsdelivr.net/npm/jsqr@1.4.0/dist/jsQR.js"></script>
    <script src="https://cdnjs.cloudflare.com/ajax/libs/xlsx/0.18.5/xlsx.full.min.js"></script>
    <script src="https://cdnjs.cloudflare.com/ajax/libs/jspdf/2.5.1/jspdf.umd.min.js"></script>
    <style>
        body {
            box-sizing: border-box;
        }
        
        .bg-primary { background-color: #1e40af; }
        .bg-secondary { background-color: #10b981; }
        .text-primary { color: #1e40af; }
        .text-secondary { color: #10b981; }
        .border-primary { border-color: #1e40af; }
        
        .notification {
            position: fixed;
            top: 20px;
            right: 20px;
            z-index: 1000;
            min-width: 300px;
            transform: translateX(400px);
            transition: transform 0.3s ease;
        }
        
        .notification.show {
            transform: translateX(0);
        }
        
        .scanner-container {
            position: relative;
            width: 100%;
            max-width: 400px;
            margin: 0 auto;
        }
        
        #scanner-video {
            width: 100%;
            height: 300px;
            object-fit: cover;
            border-radius: 8px;
        }
        
        .scanner-overlay {
            position: absolute;
            top: 50%;
            left: 50%;
            transform: translate(-50%, -50%);
            width: 200px;
            height: 200px;
            border: 2px solid #10b981;
            border-radius: 8px;
            pointer-events: none;
        }
        
        .hidden { display: none !important; }
        
        .card-hover:hover {
            transform: translateY(-2px);
            box-shadow: 0 10px 25px rgba(0,0,0,0.1);
        }
        
        .transition-all {
            transition: all 0.3s ease;
        }

        .login-container {
            min-height: 100vh;
            background: linear-gradient(135deg, #1e40af 0%, #10b981 100%);
        }

        .logo-preview {
            max-width: 200px;
            max-height: 100px;
            object-fit: contain;
        }

        .user-role-badge {
            font-size: 0.75rem;
            padding: 0.25rem 0.5rem;
            border-radius: 9999px;
        }

        .super-admin { background-color: #dc2626; color: white; }
        .admin { background-color: #1e40af; color: white; }
        .user { background-color: #10b981; color: white; }
    </style>
</head>
<body class="bg-gray-50">
    <!-- Login Screen -->
    <div id="login-screen" class="login-container flex items-center justify-center">
        <div class="bg-white rounded-lg shadow-xl p-8 w-full max-w-md mx-4">
            <div class="text-center mb-8">
                <div id="login-logo" class="mb-4">
                    <i class="fas fa-microchip text-4xl text-primary"></i>
                </div>
                <h1 class="text-2xl font-bold text-gray-800">Sistem Pendataan Barang</h1>
                <p class="text-gray-600">Teknik Elektronika Industri</p>
            </div>
            
            <form id="login-form" class="space-y-4">
                <div>
                    <label class="block text-sm font-medium text-gray-700 mb-2">Username</label>
                    <input type="text" id="login-username" required class="w-full px-4 py-2 border border-gray-300 rounded-lg focus:outline-none focus:border-primary">
                </div>
                
                <div>
                    <label class="block text-sm font-medium text-gray-700 mb-2">Password</label>
                    <input type="password" id="login-password" required class="w-full px-4 py-2 border border-gray-300 rounded-lg focus:outline-none focus:border-primary">
                </div>
                
                <button type="submit" class="w-full bg-primary text-white py-2 px-4 rounded-lg hover:bg-blue-700 transition-colors">
                    <i class="fas fa-sign-in-alt mr-2"></i>Login
                </button>
            </form>
            
            <div class="mt-6 text-center">
                <p class="text-sm text-gray-600">Demo Accounts:</p>
                <div class="text-xs text-gray-500 mt-2 space-y-1">
                    <p><strong>Super Admin:</strong> superadmin / admin123</p>
                    <p><strong>Admin:</strong> admin / admin123</p>
                    <p><strong>User:</strong> user / user123</p>
                </div>
            </div>
        </div>
    </div>

    <!-- Main App -->
    <div id="main-app" class="hidden">
        <!-- Notification Container -->
        <div id="notification" class="notification"></div>

        <!-- Header -->
        <header class="bg-primary text-white shadow-lg">
            <div class="container mx-auto px-4 py-4">
                <div class="flex items-center justify-between">
                    <div class="flex items-center space-x-3">
                        <div id="header-logo">
                            <i class="fas fa-microchip text-2xl"></i>
                        </div>
                        <div>
                            <h1 class="text-xl font-bold">Sistem Pendataan Barang</h1>
                            <p class="text-blue-200 text-sm">Teknik Elektronika Industri</p>
                        </div>
                    </div>
                    <div class="flex items-center space-x-4">
                        <div class="text-right">
                            <p class="text-sm font-medium" id="user-name">User</p>
                            <p class="text-xs text-blue-200" id="user-role">Role</p>
                        </div>
                        <span id="current-time" class="text-sm"></span>
                        <div class="relative">
                            <button onclick="toggleUserMenu()" class="flex items-center space-x-2 hover:bg-blue-600 rounded-lg px-2 py-1 transition-colors">
                                <i class="fas fa-user-circle text-2xl"></i>
                                <i class="fas fa-chevron-down text-sm"></i>
                            </button>
                            <div id="user-menu" class="absolute right-0 mt-2 w-48 bg-white rounded-lg shadow-lg py-2 hidden">
                                <button onclick="logout()" class="w-full text-left px-4 py-2 text-gray-700 hover:bg-gray-100">
                                    <i class="fas fa-sign-out-alt mr-2"></i>Logout
                                </button>
                            </div>
                        </div>
                    </div>
                </div>
            </div>
        </header>

        <!-- Navigation -->
        <nav class="bg-white shadow-md border-b-2 border-primary">
            <div class="container mx-auto px-4">
                <div class="flex space-x-0 overflow-x-auto">
                    <button onclick="showSection('dashboard')" class="nav-btn px-6 py-3 text-gray-600 hover:text-primary hover:bg-blue-50 border-b-2 border-transparent hover:border-primary transition-all whitespace-nowrap">
                        <i class="fas fa-tachometer-alt mr-2"></i>Dashboard
                    </button>
                    <button onclick="showSection('data-barang')" class="nav-btn px-6 py-3 text-gray-600 hover:text-primary hover:bg-blue-50 border-b-2 border-transparent hover:border-primary transition-all whitespace-nowrap">
                        <i class="fas fa-boxes mr-2"></i>Data Barang
                    </button>
                    <button onclick="showSection('input-barang')" class="nav-btn px-6 py-3 text-gray-600 hover:text-primary hover:bg-blue-50 border-b-2 border-transparent hover:border-primary transition-all whitespace-nowrap">
                        <i class="fas fa-plus-circle mr-2"></i>Input Barang
                    </button>
                    <button onclick="showSection('barang-masuk')" class="nav-btn px-6 py-3 text-gray-600 hover:text-primary hover:bg-blue-50 border-b-2 border-transparent hover:border-primary transition-all whitespace-nowrap">
                        <i class="fas fa-arrow-down mr-2"></i>Barang Masuk
                    </button>
                    <button onclick="showSection('barang-keluar')" class="nav-btn px-6 py-3 text-gray-600 hover:text-primary hover:bg-blue-50 border-b-2 border-transparent hover:border-primary transition-all whitespace-nowrap">
                        <i class="fas fa-arrow-up mr-2"></i>Barang Keluar
                    </button>
                    <button onclick="showSection('laporan')" class="nav-btn px-6 py-3 text-gray-600 hover:text-primary hover:bg-blue-50 border-b-2 border-transparent hover:border-primary transition-all whitespace-nowrap">
                        <i class="fas fa-chart-bar mr-2"></i>Laporan
                    </button>
                    <!-- Admin Only Sections -->
                    <button id="nav-users" onclick="showSection('users')" class="nav-btn px-6 py-3 text-gray-600 hover:text-primary hover:bg-blue-50 border-b-2 border-transparent hover:border-primary transition-all whitespace-nowrap hidden">
                        <i class="fas fa-users mr-2"></i>Kelola User
                    </button>
                    <!-- Super Admin Only Sections -->
                    <button id="nav-settings" onclick="showSection('settings')" class="nav-btn px-6 py-3 text-gray-600 hover:text-primary hover:bg-blue-50 border-b-2 border-transparent hover:border-primary transition-all whitespace-nowrap hidden">
                        <i class="fas fa-cog mr-2"></i>Pengaturan
                    </button>
                </div>
            </div>
        </nav>

        <!-- Main Content -->
        <main class="container mx-auto px-4 py-6">
            <!-- Dashboard Section -->
            <section id="dashboard" class="section">
                <div class="mb-6">
                    <h2 class="text-2xl font-bold text-gray-800 mb-2">Dashboard</h2>
                    <p class="text-gray-600">Ringkasan data inventori barang</p>
                </div>
                
                <div class="grid grid-cols-1 md:grid-cols-2 lg:grid-cols-4 gap-6 mb-8">
                    <div onclick="viewAllItems()" class="bg-white rounded-lg shadow-md p-6 card-hover transition-all cursor-pointer">
                        <div class="flex items-center justify-between">
                            <div>
                                <p class="text-gray-500 text-sm">Total Barang</p>
                                <p id="total-items" class="text-2xl font-bold text-primary">0</p>
                            </div>
                            <i class="fas fa-boxes text-3xl text-primary opacity-20"></i>
                        </div>
                    </div>
                    
                    <div onclick="viewAvailableItems()" class="bg-white rounded-lg shadow-md p-6 card-hover transition-all cursor-pointer">
                        <div class="flex items-center justify-between">
                            <div>
                                <p class="text-gray-500 text-sm">Barang Tersedia</p>
                                <p id="available-items" class="text-2xl font-bold text-secondary">0</p>
                            </div>
                            <i class="fas fa-check-circle text-3xl text-secondary opacity-20"></i>
                        </div>
                    </div>
                    
                    <div onclick="viewLowStockItems()" class="bg-white rounded-lg shadow-md p-6 card-hover transition-all cursor-pointer">
                        <div class="flex items-center justify-between">
                            <div>
                                <p class="text-gray-500 text-sm">Stok Menipis</p>
                                <p id="low-stock" class="text-2xl font-bold text-yellow-500">0</p>
                            </div>
                            <i class="fas fa-exclamation-triangle text-3xl text-yellow-500 opacity-20"></i>
                        </div>
                    </div>
                    
                    <div onclick="viewEmptyStockItems()" class="bg-white rounded-lg shadow-md p-6 card-hover transition-all cursor-pointer">
                        <div class="flex items-center justify-between">
                            <div>
                                <p class="text-gray-500 text-sm">Stok Habis</p>
                                <p id="empty-stock" class="text-2xl font-bold text-red-500">0</p>
                            </div>
                            <i class="fas fa-times-circle text-3xl text-red-500 opacity-20"></i>
                        </div>
                    </div>
                </div>

                <!-- Recent Activities -->
                <div class="bg-white rounded-lg shadow-md p-6">
                    <h3 class="text-lg font-semibold text-gray-800 mb-4">Aktivitas Terbaru</h3>
                    <div id="recent-activities" class="space-y-3">
                        <p class="text-gray-500 text-center py-4">Belum ada aktivitas</p>
                    </div>
                </div>
            </section>

            <!-- Data Barang Section -->
            <section id="data-barang" class="section hidden">
                <div class="mb-6">
                    <h2 class="text-2xl font-bold text-gray-800 mb-2">Data Barang</h2>
                    <p class="text-gray-600">Daftar semua barang dalam inventori</p>
                </div>
                
                <div class="bg-white rounded-lg shadow-md p-6">
                    <div class="flex flex-col md:flex-row justify-between items-center mb-4 space-y-2 md:space-y-0">
                        <div class="flex items-center space-x-2">
                            <input type="text" id="search-items" placeholder="Cari barang..." class="px-4 py-2 border border-gray-300 rounded-lg focus:outline-none focus:border-primary">
                            <button onclick="searchItems()" class="bg-primary text-white px-4 py-2 rounded-lg hover:bg-blue-700 transition-colors">
                                <i class="fas fa-search"></i>
                            </button>
                        </div>
                        <div class="flex items-center space-x-2">
                            <select id="category-filter" onchange="applyFilters()" class="px-4 py-2 border border-gray-300 rounded-lg focus:outline-none focus:border-primary">
                                <option value="">Semua Kategori</option>
                                <option value="Komponen">Komponen</option>
                                <option value="Alat">Alat</option>
                                <option value="Modul">Modul</option>
                                <option value="Sensor">Sensor</option>
                            </select>
                            <select id="stock-filter" onchange="applyFilters()" class="px-4 py-2 border border-gray-300 rounded-lg focus:outline-none focus:border-primary">
                                <option value="">Semua Stok</option>
                                <option value="available">Tersedia</option>
                                <option value="low">Stok Menipis</option>
                                <option value="empty">Stok Habis</option>
                            </select>
                            <select id="sort-filter" onchange="applyFilters()" class="px-4 py-2 border border-gray-300 rounded-lg focus:outline-none focus:border-primary">
                                <option value="name-asc">Nama A-Z</option>
                                <option value="name-desc">Nama Z-A</option>
                                <option value="stock-asc">Stok Terendah</option>
                                <option value="stock-desc">Stok Tertinggi</option>
                                <option value="date-asc">Terlama</option>
                                <option value="date-desc">Terbaru</option>
                            </select>
                        </div>
                    </div>
                    
                    <div class="overflow-x-auto">
                        <table class="w-full table-auto">
                            <thead class="bg-gray-50">
                                <tr>
                                    <th class="px-4 py-3 text-left text-xs font-medium text-gray-500 uppercase tracking-wider">Gambar</th>
                                    <th class="px-4 py-3 text-left text-xs font-medium text-gray-500 uppercase tracking-wider">Kode</th>
                                    <th class="px-4 py-3 text-left text-xs font-medium text-gray-500 uppercase tracking-wider">Nama Barang</th>
                                    <th class="px-4 py-3 text-left text-xs font-medium text-gray-500 uppercase tracking-wider">Kategori</th>
                                    <th class="px-4 py-3 text-left text-xs font-medium text-gray-500 uppercase tracking-wider">Stok</th>
                                    <th class="px-4 py-3 text-left text-xs font-medium text-gray-500 uppercase tracking-wider">QR Code</th>
                                    <th class="px-4 py-3 text-left text-xs font-medium text-gray-500 uppercase tracking-wider">Aksi</th>
                                </tr>
                            </thead>
                            <tbody id="items-table" class="bg-white divide-y divide-gray-200">
                                <tr>
                                    <td colspan="7" class="px-4 py-8 text-center text-gray-500">Belum ada data barang</td>
                                </tr>
                            </tbody>
                        </table>
                    </div>
                </div>
            </section>

            <!-- Input Barang Section -->
            <section id="input-barang" class="section hidden">
                <div class="mb-6">
                    <h2 class="text-2xl font-bold text-gray-800 mb-2">Input Barang Baru</h2>
                    <p class="text-gray-600">Tambahkan barang baru ke inventori</p>
                </div>
                
                <div class="grid grid-cols-1 lg:grid-cols-2 gap-6">
                    <!-- Manual Input Form -->
                    <div class="bg-white rounded-lg shadow-md p-6">
                        <h3 class="text-lg font-semibold text-gray-800 mb-4">Input Manual</h3>
                        <form id="add-item-form" class="space-y-4">
                            <div>
                                <label class="block text-sm font-medium text-gray-700 mb-2">Kode Barang</label>
                                <input type="text" id="item-code" required class="w-full px-4 py-2 border border-gray-300 rounded-lg focus:outline-none focus:border-primary">
                            </div>
                            
                            <div>
                                <label class="block text-sm font-medium text-gray-700 mb-2">Nama Barang</label>
                                <input type="text" id="item-name" required class="w-full px-4 py-2 border border-gray-300 rounded-lg focus:outline-none focus:border-primary">
                            </div>
                            
                            <div>
                                <label class="block text-sm font-medium text-gray-700 mb-2">Kategori</label>
                                <select id="item-category" required class="w-full px-4 py-2 border border-gray-300 rounded-lg focus:outline-none focus:border-primary">
                                    <option value="">Pilih Kategori</option>
                                    <option value="Komponen">Komponen</option>
                                    <option value="Alat">Alat</option>
                                    <option value="Modul">Modul</option>
                                    <option value="Sensor">Sensor</option>
                                </select>
                            </div>
                            
                            <div>
                                <label class="block text-sm font-medium text-gray-700 mb-2">Jumlah</label>
                                <input type="number" id="item-quantity" required min="1" class="w-full px-4 py-2 border border-gray-300 rounded-lg focus:outline-none focus:border-primary">
                            </div>
                            
                            <div>
                                <label class="block text-sm font-medium text-gray-700 mb-2">Tanggal Masuk</label>
                                <input type="date" id="item-date" required class="w-full px-4 py-2 border border-gray-300 rounded-lg focus:outline-none focus:border-primary">
                            </div>
                            
                            <div>
                                <label class="block text-sm font-medium text-gray-700 mb-2">Gambar Barang</label>
                                <input type="file" id="item-image" accept="image/*" class="w-full px-4 py-2 border border-gray-300 rounded-lg focus:outline-none focus:border-primary">
                            </div>
                            
                            <button type="submit" class="w-full bg-primary text-white py-2 px-4 rounded-lg hover:bg-blue-700 transition-colors">
                                <i class="fas fa-plus mr-2"></i>Tambah Barang
                            </button>
                        </form>
                    </div>
                    
                    <!-- Scanner Input -->
                    <div class="bg-white rounded-lg shadow-md p-6">
                        <h3 class="text-lg font-semibold text-gray-800 mb-4">Scan Barcode/QR Code</h3>
                        <div class="text-center">
                            <div class="scanner-container mb-4">
                                <video id="scanner-video" class="hidden"></video>
                                <canvas id="scanner-canvas" class="hidden"></canvas>
                                <div class="scanner-overlay hidden"></div>
                                <div id="scanner-placeholder" class="bg-gray-100 rounded-lg p-8 border-2 border-dashed border-gray-300">
                                    <i class="fas fa-qrcode text-4xl text-gray-400 mb-4"></i>
                                    <p class="text-gray-500">Klik tombol di bawah untuk memulai scan</p>
                                </div>
                            </div>
                            
                            <button id="start-scanner" onclick="startScanner()" class="bg-secondary text-white px-6 py-2 rounded-lg hover:bg-green-600 transition-colors mb-2">
                                <i class="fas fa-camera mr-2"></i>Mulai Scan
                            </button>
                            <button id="stop-scanner" onclick="stopScanner()" class="bg-red-500 text-white px-6 py-2 rounded-lg hover:bg-red-600 transition-colors mb-2 hidden">
                                <i class="fas fa-stop mr-2"></i>Stop Scan
                            </button>
                            
                            <div id="scan-result" class="mt-4 p-4 bg-green-50 border border-green-200 rounded-lg hidden">
                                <p class="text-green-800">Kode berhasil discan: <span id="scanned-code" class="font-mono font-bold"></span></p>
                            </div>
                        </div>
                    </div>
                </div>
            </section>

            <!-- Barang Masuk Section -->
            <section id="barang-masuk" class="section hidden">
                <div class="mb-6">
                    <h2 class="text-2xl font-bold text-gray-800 mb-2">Barang Masuk</h2>
                    <p class="text-gray-600">Catat barang yang masuk ke inventori</p>
                </div>
                
                <div class="bg-white rounded-lg shadow-md p-6">
                    <form id="stock-in-form" class="grid grid-cols-1 md:grid-cols-2 gap-4">
                        <div>
                            <label class="block text-sm font-medium text-gray-700 mb-2">Pilih Barang</label>
                            <select id="stock-in-item" required class="w-full px-4 py-2 border border-gray-300 rounded-lg focus:outline-none focus:border-primary">
                                <option value="">Pilih Barang</option>
                            </select>
                        </div>
                        
                        <div>
                            <label class="block text-sm font-medium text-gray-700 mb-2">Jumlah Masuk</label>
                            <input type="number" id="stock-in-quantity" required min="1" class="w-full px-4 py-2 border border-gray-300 rounded-lg focus:outline-none focus:border-primary">
                        </div>
                        
                        <div>
                            <label class="block text-sm font-medium text-gray-700 mb-2">Tanggal Masuk</label>
                            <input type="date" id="stock-in-date" required class="w-full px-4 py-2 border border-gray-300 rounded-lg focus:outline-none focus:border-primary">
                        </div>
                        
                        <div>
                            <label class="block text-sm font-medium text-gray-700 mb-2">Keterangan</label>
                            <input type="text" id="stock-in-notes" placeholder="Keterangan (opsional)" class="w-full px-4 py-2 border border-gray-300 rounded-lg focus:outline-none focus:border-primary">
                        </div>
                        
                        <div class="md:col-span-2">
                            <button type="submit" class="bg-secondary text-white py-2 px-6 rounded-lg hover:bg-green-600 transition-colors">
                                <i class="fas fa-arrow-down mr-2"></i>Catat Barang Masuk
                            </button>
                        </div>
                    </form>
                </div>
            </section>

            <!-- Barang Keluar Section -->
            <section id="barang-keluar" class="section hidden">
                <div class="mb-6">
                    <h2 class="text-2xl font-bold text-gray-800 mb-2">Barang Keluar</h2>
                    <p class="text-gray-600">Catat barang yang keluar dari inventori</p>
                </div>
                
                <div class="bg-white rounded-lg shadow-md p-6">
                    <form id="stock-out-form" class="grid grid-cols-1 md:grid-cols-2 gap-4">
                        <div>
                            <label class="block text-sm font-medium text-gray-700 mb-2">Pilih Barang</label>
                            <select id="stock-out-item" required class="w-full px-4 py-2 border border-gray-300 rounded-lg focus:outline-none focus:border-primary">
                                <option value="">Pilih Barang</option>
                            </select>
                        </div>
                        
                        <div>
                            <label class="block text-sm font-medium text-gray-700 mb-2">Jumlah Keluar</label>
                            <input type="number" id="stock-out-quantity" required min="1" class="w-full px-4 py-2 border border-gray-300 rounded-lg focus:outline-none focus:border-primary">
                        </div>
                        
                        <div>
                            <label class="block text-sm font-medium text-gray-700 mb-2">Tanggal Keluar</label>
                            <input type="date" id="stock-out-date" required class="w-full px-4 py-2 border border-gray-300 rounded-lg focus:outline-none focus:border-primary">
                        </div>
                        
                        <div>
                            <label class="block text-sm font-medium text-gray-700 mb-2">Keterangan</label>
                            <input type="text" id="stock-out-notes" placeholder="Keterangan (opsional)" class="w-full px-4 py-2 border border-gray-300 rounded-lg focus:outline-none focus:border-primary">
                        </div>
                        
                        <div class="md:col-span-2">
                            <button type="submit" class="bg-red-500 text-white py-2 px-6 rounded-lg hover:bg-red-600 transition-colors">
                                <i class="fas fa-arrow-up mr-2"></i>Catat Barang Keluar
                            </button>
                        </div>
                    </form>
                </div>
            </section>

            <!-- Laporan Section -->
            <section id="laporan" class="section hidden">
                <div class="mb-6">
                    <h2 class="text-2xl font-bold text-gray-800 mb-2">Laporan</h2>
                    <p class="text-gray-600">Export dan cetak laporan inventori</p>
                </div>
                
                <div class="grid grid-cols-1 md:grid-cols-2 gap-6">
                    <div class="bg-white rounded-lg shadow-md p-6">
                        <h3 class="text-lg font-semibold text-gray-800 mb-4">Export Data</h3>
                        <div class="space-y-3">
                            <button onclick="exportToExcel()" class="w-full bg-secondary text-white py-2 px-4 rounded-lg hover:bg-green-600 transition-colors">
                                <i class="fas fa-file-excel mr-2"></i>Export ke Excel
                            </button>
                            <button onclick="exportToPDF()" class="w-full bg-red-500 text-white py-2 px-4 rounded-lg hover:bg-red-600 transition-colors">
                                <i class="fas fa-file-pdf mr-2"></i>Export ke PDF
                            </button>
                        </div>
                    </div>
                    
                    <div class="bg-white rounded-lg shadow-md p-6">
                        <h3 class="text-lg font-semibold text-gray-800 mb-4">Statistik</h3>
                        <div class="space-y-3">
                            <div class="flex justify-between">
                                <span class="text-gray-600">Total Barang:</span>
                                <span id="stats-total" class="font-semibold">0</span>
                            </div>
                            <div class="flex justify-between">
                                <span class="text-gray-600">Total Stok:</span>
                                <span id="stats-stock" class="font-semibold">0</span>
                            </div>
                            <div class="flex justify-between">
                                <span class="text-gray-600">Kategori Terbanyak:</span>
                                <span id="stats-category" class="font-semibold">-</span>
                            </div>
                        </div>
                    </div>
                </div>
            </section>

            <!-- Users Management Section (Admin & Super Admin Only) -->
            <section id="users" class="section hidden">
                <div class="mb-6">
                    <h2 class="text-2xl font-bold text-gray-800 mb-2">Kelola User</h2>
                    <p class="text-gray-600">Manajemen pengguna sistem</p>
                </div>
                
                <div class="grid grid-cols-1 lg:grid-cols-2 gap-6">
                    <!-- Add User Form -->
                    <div class="bg-white rounded-lg shadow-md p-6">
                        <h3 class="text-lg font-semibold text-gray-800 mb-4">Tambah User Baru</h3>
                        <form id="add-user-form" class="space-y-4">
                            <div>
                                <label class="block text-sm font-medium text-gray-700 mb-2">Username</label>
                                <input type="text" id="new-username" required class="w-full px-4 py-2 border border-gray-300 rounded-lg focus:outline-none focus:border-primary">
                            </div>
                            
                            <div>
                                <label class="block text-sm font-medium text-gray-700 mb-2">Password</label>
                                <input type="password" id="new-password" required class="w-full px-4 py-2 border border-gray-300 rounded-lg focus:outline-none focus:border-primary">
                            </div>
                            
                            <div>
                                <label class="block text-sm font-medium text-gray-700 mb-2">Nama Lengkap</label>
                                <input type="text" id="new-fullname" required class="w-full px-4 py-2 border border-gray-300 rounded-lg focus:outline-none focus:border-primary">
                            </div>
                            
                            <div>
                                <label class="block text-sm font-medium text-gray-700 mb-2">Role</label>
                                <select id="new-role" required class="w-full px-4 py-2 border border-gray-300 rounded-lg focus:outline-none focus:border-primary">
                                    <option value="">Pilih Role</option>
                                    <option value="user">User</option>
                                    <option value="admin">Admin</option>
                                    <option id="superadmin-option" value="superadmin" class="hidden">Super Admin</option>
                                </select>
                            </div>
                            
                            <button type="submit" class="w-full bg-primary text-white py-2 px-4 rounded-lg hover:bg-blue-700 transition-colors">
                                <i class="fas fa-user-plus mr-2"></i>Tambah User
                            </button>
                        </form>
                    </div>
                    
                    <!-- Users List -->
                    <div class="bg-white rounded-lg shadow-md p-6">
                        <h3 class="text-lg font-semibold text-gray-800 mb-4">Daftar User</h3>
                        <div id="users-list" class="space-y-3">
                            <!-- Users will be loaded here -->
                        </div>
                    </div>
                </div>
            </section>

            <!-- Settings Section (Super Admin Only) -->
            <section id="settings" class="section hidden">
                <div class="mb-6">
                    <h2 class="text-2xl font-bold text-gray-800 mb-2">Pengaturan Sistem</h2>
                    <p class="text-gray-600">Konfigurasi sistem dan tampilan</p>
                </div>
                
                <div class="grid grid-cols-1 lg:grid-cols-2 gap-6">
                    <!-- Logo Settings -->
                    <div class="bg-white rounded-lg shadow-md p-6">
                        <h3 class="text-lg font-semibold text-gray-800 mb-4">Pengaturan Logo</h3>
                        
                        <div class="mb-4">
                            <label class="block text-sm font-medium text-gray-700 mb-2">Logo Saat Ini</label>
                            <div class="border border-gray-300 rounded-lg p-4 text-center">
                                <div id="current-logo-preview">
                                    <i class="fas fa-microchip text-4xl text-primary"></i>
                                </div>
                            </div>
                        </div>
                        
                        <form id="logo-form" class="space-y-4">
                            <div>
                                <label class="block text-sm font-medium text-gray-700 mb-2">Upload Logo Baru</label>
                                <input type="file" id="logo-upload" accept="image/*" class="w-full px-4 py-2 border border-gray-300 rounded-lg focus:outline-none focus:border-primary">
                                <p class="text-xs text-gray-500 mt-1">Format: JPG, PNG, SVG. Maksimal 2MB</p>
                            </div>
                            
                            <div class="flex space-x-2">
                                <button type="submit" class="flex-1 bg-primary text-white py-2 px-4 rounded-lg hover:bg-blue-700 transition-colors">
                                    <i class="fas fa-upload mr-2"></i>Update Logo
                                </button>
                                <button type="button" onclick="resetLogo()" class="flex-1 bg-gray-500 text-white py-2 px-4 rounded-lg hover:bg-gray-600 transition-colors">
                                    <i class="fas fa-undo mr-2"></i>Reset Default
                                </button>
                            </div>
                        </form>
                    </div>
                    
                    <!-- System Settings -->
                    <div class="bg-white rounded-lg shadow-md p-6">
                        <h3 class="text-lg font-semibold text-gray-800 mb-4">Pengaturan Sistem</h3>
                        
                        <form id="system-settings-form" class="space-y-4">
                            <div>
                                <label class="block text-sm font-medium text-gray-700 mb-2">Nama Sistem</label>
                                <input type="text" id="system-name" value="Sistem Pendataan Barang" class="w-full px-4 py-2 border border-gray-300 rounded-lg focus:outline-none focus:border-primary">
                            </div>
                            
                            <div>
                                <label class="block text-sm font-medium text-gray-700 mb-2">Nama Institusi</label>
                                <input type="text" id="institution-name" value="Teknik Elektronika Industri" class="w-full px-4 py-2 border border-gray-300 rounded-lg focus:outline-none focus:border-primary">
                            </div>
                            
                            <div>
                                <label class="block text-sm font-medium text-gray-700 mb-2">Minimum Stok Alert</label>
                                <input type="number" id="min-stock-alert" value="10" min="1" class="w-full px-4 py-2 border border-gray-300 rounded-lg focus:outline-none focus:border-primary">
                            </div>
                            
                            <button type="submit" class="w-full bg-secondary text-white py-2 px-4 rounded-lg hover:bg-green-600 transition-colors">
                                <i class="fas fa-save mr-2"></i>Simpan Pengaturan
                            </button>
                        </form>
                        
                        <hr class="my-6">
                        
                        <div class="space-y-3">
                            <h4 class="font-semibold text-gray-800">Data Management</h4>
                            <button onclick="exportAllData()" class="w-full bg-blue-500 text-white py-2 px-4 rounded-lg hover:bg-blue-600 transition-colors">
                                <i class="fas fa-download mr-2"></i>Backup Semua Data
                            </button>
                            <button onclick="clearAllData()" class="w-full bg-red-500 text-white py-2 px-4 rounded-lg hover:bg-red-600 transition-colors">
                                <i class="fas fa-trash mr-2"></i>Hapus Semua Data
                            </button>
                        </div>
                    </div>
                </div>
            </section>
        </main>
    </div>

    <!-- QR Code Modal -->
    <div id="qr-modal" class="fixed inset-0 bg-black bg-opacity-50 hidden items-center justify-center z-50">
        <div class="bg-white rounded-lg p-6 max-w-sm w-full mx-4">
            <div class="flex justify-between items-center mb-4">
                <h3 class="text-lg font-semibold">QR Code</h3>
                <button onclick="closeQRModal()" class="text-gray-500 hover:text-gray-700">
                    <i class="fas fa-times"></i>
                </button>
            </div>
            <div class="text-center">
                <canvas id="qr-canvas" class="mx-auto mb-4"></canvas>
                <button onclick="downloadQR()" class="bg-primary text-white px-4 py-2 rounded-lg hover:bg-blue-700 transition-colors">
                    <i class="fas fa-download mr-2"></i>Download
                </button>
            </div>
        </div>
    </div>

    <script>
        // Global variables
        let items = JSON.parse(localStorage.getItem('items')) || [];
        let transactions = JSON.parse(localStorage.getItem('transactions')) || [];
        let users = JSON.parse(localStorage.getItem('users')) || [
            { id: 1, username: 'superadmin', password: 'admin123', fullname: 'Super Administrator', role: 'superadmin', active: true },
            { id: 2, username: 'admin', password: 'admin123', fullname: 'Administrator', role: 'admin', active: true },
            { id: 3, username: 'user', password: 'user123', fullname: 'User Biasa', role: 'user', active: true }
        ];
        let currentUser = JSON.parse(localStorage.getItem('currentUser')) || null;
        let systemSettings = JSON.parse(localStorage.getItem('systemSettings')) || {
            systemName: 'Sistem Pendataan Barang',
            institutionName: 'Teknik Elektronika Industri',
            minStockAlert: 10,
            logo: null
        };
        
        let currentStream = null;
        let currentQRData = '';

        // Initialize app
        document.addEventListener('DOMContentLoaded', function() {
            // Save default users if not exists
            if (!localStorage.getItem('users')) {
                localStorage.setItem('users', JSON.stringify(users));
            }
            
            // Check if user is logged in
            if (currentUser) {
                showMainApp();
            } else {
                showLoginScreen();
            }
            
            updateDateTime();
            setInterval(updateDateTime, 1000);
            
            // Load system settings
            loadSystemSettings();
        });

        // Authentication functions
        function showLoginScreen() {
            document.getElementById('login-screen').classList.remove('hidden');
            document.getElementById('main-app').classList.add('hidden');
            loadLogo('login-logo');
        }

        function showMainApp() {
            document.getElementById('login-screen').classList.add('hidden');
            document.getElementById('main-app').classList.remove('hidden');
            
            // Set user info
            document.getElementById('user-name').textContent = currentUser.fullname;
            document.getElementById('user-role').textContent = getRoleDisplayName(currentUser.role);
            
            // Show/hide menu items based on role
            setupMenuPermissions();
            
            // Set default dates
            const today = new Date().toISOString().split('T')[0];
            document.getElementById('item-date').value = today;
            document.getElementById('stock-in-date').value = today;
            document.getElementById('stock-out-date').value = today;
            
            // Load data
            loadItems();
            updateDashboard();
            updateSelectOptions();
            loadLogo('header-logo');
            
            // Show dashboard by default
            showSection('dashboard');
            
            // Check for low stock notifications
            checkLowStock();
        }

        function setupMenuPermissions() {
            const navUsers = document.getElementById('nav-users');
            const navSettings = document.getElementById('nav-settings');
            const superadminOption = document.getElementById('superadmin-option');
            
            if (currentUser.role === 'superadmin') {
                navUsers.classList.remove('hidden');
                navSettings.classList.remove('hidden');
                superadminOption.classList.remove('hidden');
            } else if (currentUser.role === 'admin') {
                navUsers.classList.remove('hidden');
                navSettings.classList.add('hidden');
                superadminOption.classList.add('hidden');
            } else {
                navUsers.classList.add('hidden');
                navSettings.classList.add('hidden');
                superadminOption.classList.add('hidden');
            }
        }

        function getRoleDisplayName(role) {
            switch(role) {
                case 'superadmin': return 'Super Admin';
                case 'admin': return 'Admin';
                case 'user': return 'User';
                default: return 'Unknown';
            }
        }

        // Login form handler
        document.getElementById('login-form').addEventListener('submit', function(e) {
            e.preventDefault();
            
            const username = document.getElementById('login-username').value;
            const password = document.getElementById('login-password').value;
            
            const user = users.find(u => u.username === username && u.password === password && u.active);
            
            if (user) {
                currentUser = user;
                localStorage.setItem('currentUser', JSON.stringify(currentUser));
                showMainApp();
                showNotification('Login berhasil!');
            } else {
                showNotification('Username atau password salah!', 'error');
            }
        });

        function toggleUserMenu() {
            const menu = document.getElementById('user-menu');
            menu.classList.toggle('hidden');
        }

        function logout() {
            currentUser = null;
            localStorage.removeItem('currentUser');
            showLoginScreen();
            showNotification('Logout berhasil!');
        }

        // User Management Functions
        document.getElementById('add-user-form').addEventListener('submit', function(e) {
            e.preventDefault();
            
            if (currentUser.role !== 'admin' && currentUser.role !== 'superadmin') {
                showNotification('Anda tidak memiliki akses untuk menambah user!', 'error');
                return;
            }
            
            const username = document.getElementById('new-username').value;
            const password = document.getElementById('new-password').value;
            const fullname = document.getElementById('new-fullname').value;
            const role = document.getElementById('new-role').value;
            
            // Check if username already exists
            if (users.find(u => u.username === username)) {
                showNotification('Username sudah ada!', 'error');
                return;
            }
            
            // Only superadmin can create superadmin
            if (role === 'superadmin' && currentUser.role !== 'superadmin') {
                showNotification('Hanya Super Admin yang dapat membuat Super Admin!', 'error');
                return;
            }
            
            const newUser = {
                id: Date.now(),
                username: username,
                password: password,
                fullname: fullname,
                role: role,
                active: true,
                createdBy: currentUser.username,
                createdAt: new Date().toISOString()
            };
            
            users.push(newUser);
            localStorage.setItem('users', JSON.stringify(users));
            
            // Reset form
            this.reset();
            
            // Reload users list
            loadUsersList();
            
            showNotification('User berhasil ditambahkan!');
        });

        function loadUsersList() {
            const usersList = document.getElementById('users-list');
            
            if (users.length === 0) {
                usersList.innerHTML = '<p class="text-gray-500 text-center py-4">Belum ada user</p>';
                return;
            }
            
            usersList.innerHTML = users.map(user => `
                <div class="flex items-center justify-between p-3 bg-gray-50 rounded-lg">
                    <div class="flex items-center space-x-3">
                        <div class="w-10 h-10 rounded-full bg-primary text-white flex items-center justify-center">
                            <i class="fas fa-user"></i>
                        </div>
                        <div>
                            <p class="font-medium">${user.fullname}</p>
                            <p class="text-sm text-gray-500">@${user.username}</p>
                        </div>
                    </div>
                    <div class="flex items-center space-x-2">
                        <span class="user-role-badge ${user.role}">${getRoleDisplayName(user.role)}</span>
                        ${user.username !== 'superadmin' && (currentUser.role === 'superadmin' || (currentUser.role === 'admin' && user.role !== 'superadmin')) ? 
                            `<button onclick="toggleUserStatus(${user.id})" class="text-${user.active ? 'red' : 'green'}-500 hover:text-${user.active ? 'red' : 'green'}-700">
                                <i class="fas fa-${user.active ? 'ban' : 'check'}"></i>
                            </button>
                            <button onclick="deleteUser(${user.id})" class="text-red-500 hover:text-red-700">
                                <i class="fas fa-trash"></i>
                            </button>` : ''
                        }
                    </div>
                </div>
            `).join('');
        }

        function toggleUserStatus(userId) {
            const user = users.find(u => u.id === userId);
            if (user) {
                user.active = !user.active;
                localStorage.setItem('users', JSON.stringify(users));
                loadUsersList();
                showNotification(`User ${user.active ? 'diaktifkan' : 'dinonaktifkan'}!`);
            }
        }

        function deleteUser(userId) {
            if (confirm('Yakin ingin menghapus user ini?')) {
                users = users.filter(u => u.id !== userId);
                localStorage.setItem('users', JSON.stringify(users));
                loadUsersList();
                showNotification('User berhasil dihapus!');
            }
        }

        // Logo Management Functions
        function loadLogo(elementId) {
            const logoElement = document.getElementById(elementId);
            if (systemSettings.logo) {
                logoElement.innerHTML = `<img src="${systemSettings.logo}" alt="Logo" class="logo-preview">`;
            } else {
                logoElement.innerHTML = '<i class="fas fa-microchip text-4xl text-primary"></i>';
            }
        }

        document.getElementById('logo-form').addEventListener('submit', function(e) {
            e.preventDefault();
            
            if (currentUser.role !== 'superadmin') {
                showNotification('Hanya Super Admin yang dapat mengubah logo!', 'error');
                return;
            }
            
            const logoFile = document.getElementById('logo-upload').files[0];
            
            if (!logoFile) {
                showNotification('Pilih file logo terlebih dahulu!', 'error');
                return;
            }
            
            if (logoFile.size > 2 * 1024 * 1024) {
                showNotification('Ukuran file terlalu besar! Maksimal 2MB', 'error');
                return;
            }
            
            const reader = new FileReader();
            reader.onload = function(e) {
                systemSettings.logo = e.target.result;
                localStorage.setItem('systemSettings', JSON.stringify(systemSettings));
                
                // Update logo preview
                document.getElementById('current-logo-preview').innerHTML = 
                    `<img src="${systemSettings.logo}" alt="Logo" class="logo-preview mx-auto">`;
                
                // Update header logo
                loadLogo('header-logo');
                
                // Reset form
                document.getElementById('logo-form').reset();
                
                showNotification('Logo berhasil diupdate!');
            };
            reader.readAsDataURL(logoFile);
        });

        function resetLogo() {
            if (currentUser.role !== 'superadmin') {
                showNotification('Hanya Super Admin yang dapat mereset logo!', 'error');
                return;
            }
            
            if (confirm('Yakin ingin mereset logo ke default?')) {
                systemSettings.logo = null;
                localStorage.setItem('systemSettings', JSON.stringify(systemSettings));
                
                // Update logo displays
                document.getElementById('current-logo-preview').innerHTML = 
                    '<i class="fas fa-microchip text-4xl text-primary"></i>';
                loadLogo('header-logo');
                
                showNotification('Logo berhasil direset!');
            }
        }

        // System Settings Functions
        function loadSystemSettings() {
            document.getElementById('system-name').value = systemSettings.systemName;
            document.getElementById('institution-name').value = systemSettings.institutionName;
            document.getElementById('min-stock-alert').value = systemSettings.minStockAlert;
            
            if (systemSettings.logo) {
                document.getElementById('current-logo-preview').innerHTML = 
                    `<img src="${systemSettings.logo}" alt="Logo" class="logo-preview mx-auto">`;
            }
        }

        document.getElementById('system-settings-form').addEventListener('submit', function(e) {
            e.preventDefault();
            
            if (currentUser.role !== 'superadmin') {
                showNotification('Hanya Super Admin yang dapat mengubah pengaturan sistem!', 'error');
                return;
            }
            
            systemSettings.systemName = document.getElementById('system-name').value;
            systemSettings.institutionName = document.getElementById('institution-name').value;
            systemSettings.minStockAlert = parseInt(document.getElementById('min-stock-alert').value);
            
            localStorage.setItem('systemSettings', JSON.stringify(systemSettings));
            
            showNotification('Pengaturan sistem berhasil disimpan!');
        });

        function exportAllData() {
            if (currentUser.role !== 'superadmin') {
                showNotification('Hanya Super Admin yang dapat backup data!', 'error');
                return;
            }
            
            const allData = {
                items: items,
                transactions: transactions,
                users: users.map(u => ({ ...u, password: '***' })), // Hide passwords
                systemSettings: systemSettings,
                exportDate: new Date().toISOString()
            };
            
            const dataStr = JSON.stringify(allData, null, 2);
            const dataBlob = new Blob([dataStr], { type: 'application/json' });
            
            const link = document.createElement('a');
            link.href = URL.createObjectURL(dataBlob);
            link.download = `backup_${new Date().toISOString().split('T')[0]}.json`;
            link.click();
            
            showNotification('Backup data berhasil didownload!');
        }

        function clearAllData() {
            if (currentUser.role !== 'superadmin') {
                showNotification('Hanya Super Admin yang dapat menghapus semua data!', 'error');
                return;
            }
            
            if (confirm('PERINGATAN: Ini akan menghapus SEMUA data termasuk barang, transaksi, dan user (kecuali superadmin). Yakin ingin melanjutkan?')) {
                if (confirm('Konfirmasi sekali lagi: Semua data akan hilang permanen!')) {
                    // Clear all data except superadmin user
                    items = [];
                    transactions = [];
                    users = users.filter(u => u.username === 'superadmin');
                    
                    localStorage.setItem('items', JSON.stringify(items));
                    localStorage.setItem('transactions', JSON.stringify(transactions));
                    localStorage.setItem('users', JSON.stringify(users));
                    
                    // Reload displays
                    loadItems();
                    updateDashboard();
                    updateSelectOptions();
                    loadUsersList();
                    
                    showNotification('Semua data berhasil dihapus!');
                }
            }
        }

        // Navigation
        function showSection(sectionId) {
            // Check permissions
            if ((sectionId === 'users' && currentUser.role === 'user') ||
                (sectionId === 'settings' && currentUser.role !== 'superadmin')) {
                showNotification('Anda tidak memiliki akses ke halaman ini!', 'error');
                return;
            }
            
            // Hide all sections
            document.querySelectorAll('.section').forEach(section => {
                section.classList.add('hidden');
            });
            
            // Remove active state from all nav buttons
            document.querySelectorAll('.nav-btn').forEach(btn => {
                btn.classList.remove('text-primary', 'border-primary');
                btn.classList.add('text-gray-600', 'border-transparent');
            });
            
            // Show selected section
            document.getElementById(sectionId).classList.remove('hidden');
            
            // Add active state to clicked nav button (only if event exists)
            if (event && event.target) {
                event.target.classList.remove('text-gray-600', 'border-transparent');
                event.target.classList.add('text-primary', 'border-primary');
            } else {
                // Find and activate the corresponding nav button
                const navButton = document.querySelector(`[onclick="showSection('${sectionId}')"]`);
                if (navButton) {
                    navButton.classList.remove('text-gray-600', 'border-transparent');
                    navButton.classList.add('text-primary', 'border-primary');
                }
            }
            
            // Update data when switching sections
            if (sectionId === 'dashboard') {
                updateDashboard();
            } else if (sectionId === 'data-barang') {
                loadItems();
            } else if (sectionId === 'laporan') {
                updateStats();
            } else if (sectionId === 'users') {
                loadUsersList();
            } else if (sectionId === 'settings') {
                loadSystemSettings();
            }
        }

        // DateTime update
        function updateDateTime() {
            const now = new Date();
            const options = { 
                weekday: 'long', 
                year: 'numeric', 
                month: 'long', 
                day: 'numeric',
                hour: '2-digit',
                minute: '2-digit'
            };
            document.getElementById('current-time').textContent = now.toLocaleDateString('id-ID', options);
        }

        // Notifications
        function showNotification(message, type = 'success') {
            const notification = document.getElementById('notification');
            const bgColor = type === 'success' ? 'bg-green-500' : type === 'error' ? 'bg-red-500' : 'bg-yellow-500';
            
            notification.innerHTML = `
                <div class="${bgColor} text-white px-6 py-4 rounded-lg shadow-lg">
                    <div class="flex items-center justify-between">
                        <span>${message}</span>
                        <button onclick="hideNotification()" class="ml-4 text-white hover:text-gray-200">
                            <i class="fas fa-times"></i>
                        </button>
                    </div>
                </div>
            `;
            
            notification.classList.add('show');
            setTimeout(hideNotification, 5000);
        }

        function hideNotification() {
            document.getElementById('notification').classList.remove('show');
        }

        // Add Item Form
        document.getElementById('add-item-form').addEventListener('submit', function(e) {
            e.preventDefault();
            
            const code = document.getElementById('item-code').value;
            const name = document.getElementById('item-name').value;
            const category = document.getElementById('item-category').value;
            const quantity = parseInt(document.getElementById('item-quantity').value);
            const date = document.getElementById('item-date').value;
            const imageFile = document.getElementById('item-image').files[0];
            
            // Check if code already exists
            if (items.find(item => item.code === code)) {
                showNotification('Kode barang sudah ada!', 'error');
                return;
            }
            
            const item = {
                id: Date.now(),
                code: code,
                name: name,
                category: category,
                stock: quantity,
                dateAdded: date,
                image: null,
                addedBy: currentUser.username
            };
            
            // Handle image upload
            if (imageFile) {
                const reader = new FileReader();
                reader.onload = function(e) {
                    item.image = e.target.result;
                    saveItem(item, quantity);
                };
                reader.readAsDataURL(imageFile);
            } else {
                saveItem(item, quantity);
            }
        });

        function saveItem(item, quantity) {
            items.push(item);
            localStorage.setItem('items', JSON.stringify(items));
            
            // Add transaction record
            const transaction = {
                id: Date.now(),
                itemId: item.id,
                itemName: item.name,
                type: 'masuk',
                quantity: quantity,
                date: item.dateAdded,
                notes: 'Barang baru ditambahkan',
                processedBy: currentUser.username
            };
            
            transactions.push(transaction);
            localStorage.setItem('transactions', JSON.stringify(transactions));
            
            // Reset form
            document.getElementById('add-item-form').reset();
            document.getElementById('item-date').value = new Date().toISOString().split('T')[0];
            
            // Update displays
            loadItems();
            updateDashboard();
            updateSelectOptions();
            
            showNotification('Barang berhasil ditambahkan!');
        }

        // Load and display items
        function loadItems() {
            const tbody = document.getElementById('items-table');
            
            if (items.length === 0) {
                tbody.innerHTML = '<tr><td colspan="7" class="px-4 py-8 text-center text-gray-500">Belum ada data barang</td></tr>';
                return;
            }
            
            tbody.innerHTML = items.map(item => `
                <tr class="hover:bg-gray-50">
                    <td class="px-4 py-4">
                        ${item.image ? 
                            `<img src="${item.image}" alt="${item.name}" class="w-12 h-12 object-cover rounded-lg">` : 
                            '<div class="w-12 h-12 bg-gray-200 rounded-lg flex items-center justify-center"><i class="fas fa-image text-gray-400"></i></div>'
                        }
                    </td>
                    <td class="px-4 py-4 font-mono text-sm">${item.code}</td>
                    <td class="px-4 py-4 font-medium">${item.name}</td>
                    <td class="px-4 py-4">
                        <span class="px-2 py-1 text-xs font-semibold rounded-full bg-blue-100 text-blue-800">${item.category}</span>
                    </td>
                    <td class="px-4 py-4">
                        <span class="font-semibold ${item.stock < systemSettings.minStockAlert ? 'text-red-500' : 'text-green-600'}">${item.stock}</span>
                    </td>
                    <td class="px-4 py-4">
                        <button onclick="showQRCode('${item.code}')" class="text-primary hover:text-blue-700">
                            <i class="fas fa-qrcode text-lg"></i>
                        </button>
                    </td>
                    <td class="px-4 py-4">
                        ${currentUser.role !== 'user' ? 
                            `<button onclick="deleteItem(${item.id})" class="text-red-500 hover:text-red-700">
                                <i class="fas fa-trash"></i>
                            </button>` : ''
                        }
                    </td>
                </tr>
            `).join('');
        }

        // Delete item
        function deleteItem(id) {
            if (currentUser.role === 'user') {
                showNotification('Anda tidak memiliki akses untuk menghapus barang!', 'error');
                return;
            }
            
            if (confirm('Yakin ingin menghapus barang ini?')) {
                items = items.filter(item => item.id !== id);
                localStorage.setItem('items', JSON.stringify(items));
                loadItems();
                updateDashboard();
                updateSelectOptions();
                showNotification('Barang berhasil dihapus!');
            }
        }

        // Search items
        function searchItems() {
            applyFilters();
        }

        // Apply all filters and sorting
        function applyFilters() {
            let filteredItems = [...items];
            
            // Search filter
            const searchTerm = document.getElementById('search-items').value.toLowerCase();
            if (searchTerm) {
                filteredItems = filteredItems.filter(item => 
                    item.name.toLowerCase().includes(searchTerm) || 
                    item.code.toLowerCase().includes(searchTerm)
                );
            }
            
            // Category filter
            const category = document.getElementById('category-filter').value;
            if (category) {
                filteredItems = filteredItems.filter(item => item.category === category);
            }
            
            // Stock filter
            const stockFilter = document.getElementById('stock-filter').value;
            if (stockFilter === 'available') {
                filteredItems = filteredItems.filter(item => item.stock > systemSettings.minStockAlert);
            } else if (stockFilter === 'low') {
                filteredItems = filteredItems.filter(item => item.stock > 0 && item.stock <= systemSettings.minStockAlert);
            } else if (stockFilter === 'empty') {
                filteredItems = filteredItems.filter(item => item.stock === 0);
            }
            
            // Sort filter
            const sortFilter = document.getElementById('sort-filter').value;
            switch (sortFilter) {
                case 'name-asc':
                    filteredItems.sort((a, b) => a.name.localeCompare(b.name));
                    break;
                case 'name-desc':
                    filteredItems.sort((a, b) => b.name.localeCompare(a.name));
                    break;
                case 'stock-asc':
                    filteredItems.sort((a, b) => a.stock - b.stock);
                    break;
                case 'stock-desc':
                    filteredItems.sort((a, b) => b.stock - a.stock);
                    break;
                case 'date-asc':
                    filteredItems.sort((a, b) => new Date(a.dateAdded) - new Date(b.dateAdded));
                    break;
                case 'date-desc':
                    filteredItems.sort((a, b) => new Date(b.dateAdded) - new Date(a.dateAdded));
                    break;
            }
            
            displayFilteredItems(filteredItems);
        }

        function displayFilteredItems(filteredItems) {
            const tbody = document.getElementById('items-table');
            
            if (filteredItems.length === 0) {
                tbody.innerHTML = '<tr><td colspan="7" class="px-4 py-8 text-center text-gray-500">Tidak ada data yang sesuai</td></tr>';
                return;
            }
            
            tbody.innerHTML = filteredItems.map(item => `
                <tr class="hover:bg-gray-50">
                    <td class="px-4 py-4">
                        ${item.image ? 
                            `<img src="${item.image}" alt="${item.name}" class="w-12 h-12 object-cover rounded-lg">` : 
                            '<div class="w-12 h-12 bg-gray-200 rounded-lg flex items-center justify-center"><i class="fas fa-image text-gray-400"></i></div>'
                        }
                    </td>
                    <td class="px-4 py-4 font-mono text-sm">${item.code}</td>
                    <td class="px-4 py-4 font-medium">${item.name}</td>
                    <td class="px-4 py-4">
                        <span class="px-2 py-1 text-xs font-semibold rounded-full bg-blue-100 text-blue-800">${item.category}</span>
                    </td>
                    <td class="px-4 py-4">
                        <span class="font-semibold ${item.stock < systemSettings.minStockAlert ? 'text-red-500' : 'text-green-600'}">${item.stock}</span>
                    </td>
                    <td class="px-4 py-4">
                        <button onclick="showQRCode('${item.code}')" class="text-primary hover:text-blue-700">
                            <i class="fas fa-qrcode text-lg"></i>
                        </button>
                    </td>
                    <td class="px-4 py-4">
                        ${currentUser.role !== 'user' ? 
                            `<button onclick="deleteItem(${item.id})" class="text-red-500 hover:text-red-700">
                                <i class="fas fa-trash"></i>
                            </button>` : ''
                        }
                    </td>
                </tr>
            `).join('');
        }

        // QR Code functions
        function showQRCode(code) {
            currentQRData = code;
            const canvas = document.getElementById('qr-canvas');
            
            QRCode.toCanvas(canvas, code, {
                width: 200,
                margin: 2,
                color: {
                    dark: '#1e40af',
                    light: '#ffffff'
                }
            }, function(error) {
                if (error) {
                    showNotification('Error generating QR code', 'error');
                    return;
                }
                document.getElementById('qr-modal').classList.remove('hidden');
                document.getElementById('qr-modal').classList.add('flex');
            });
        }

        function closeQRModal() {
            document.getElementById('qr-modal').classList.add('hidden');
            document.getElementById('qr-modal').classList.remove('flex');
        }

        function downloadQR() {
            const canvas = document.getElementById('qr-canvas');
            const link = document.createElement('a');
            link.download = `qr-${currentQRData}.png`;
            link.href = canvas.toDataURL();
            link.click();
        }

        // Scanner functions
        function startScanner() {
            navigator.mediaDevices.getUserMedia({ video: { facingMode: 'environment' } })
                .then(function(stream) {
                    currentStream = stream;
                    const video = document.getElementById('scanner-video');
                    const canvas = document.getElementById('scanner-canvas');
                    const overlay = document.querySelector('.scanner-overlay');
                    const placeholder = document.getElementById('scanner-placeholder');
                    
                    video.srcObject = stream;
                    video.play();
                    
                    // Show scanner elements
                    video.classList.remove('hidden');
                    overlay.classList.remove('hidden');
                    placeholder.classList.add('hidden');
                    document.getElementById('start-scanner').classList.add('hidden');
                    document.getElementById('stop-scanner').classList.remove('hidden');
                    
                    // Start scanning
                    scanQRCode(video, canvas);
                })
                .catch(function(err) {
                    showNotification('Tidak dapat mengakses kamera: ' + err.message, 'error');
                });
        }

        function stopScanner() {
            if (currentStream) {
                currentStream.getTracks().forEach(track => track.stop());
                currentStream = null;
            }
            
            const video = document.getElementById('scanner-video');
            const overlay = document.querySelector('.scanner-overlay');
            const placeholder = document.getElementById('scanner-placeholder');
            
            video.classList.add('hidden');
            overlay.classList.add('hidden');
            placeholder.classList.remove('hidden');
            document.getElementById('start-scanner').classList.remove('hidden');
            document.getElementById('stop-scanner').classList.add('hidden');
            document.getElementById('scan-result').classList.add('hidden');
        }

        function scanQRCode(video, canvas) {
            const context = canvas.getContext('2d');
            
            function scan() {
                if (video.readyState === video.HAVE_ENOUGH_DATA) {
                    canvas.height = video.videoHeight;
                    canvas.width = video.videoWidth;
                    context.drawImage(video, 0, 0, canvas.width, canvas.height);
                    
                    const imageData = context.getImageData(0, 0, canvas.width, canvas.height);
                    const code = jsQR(imageData.data, imageData.width, imageData.height);
                    
                    if (code) {
                        document.getElementById('scanned-code').textContent = code.data;
                        document.getElementById('scan-result').classList.remove('hidden');
                        document.getElementById('item-code').value = code.data;
                        stopScanner();
                        showNotification('Kode berhasil discan!');
                        return;
                    }
                }
                
                if (currentStream) {
                    requestAnimationFrame(scan);
                }
            }
            
            scan();
        }

        // Stock In/Out functions
        function updateSelectOptions() {
            const stockInSelect = document.getElementById('stock-in-item');
            const stockOutSelect = document.getElementById('stock-out-item');
            
            const options = items.map(item => 
                `<option value="${item.id}">${item.name} (${item.code}) - Stok: ${item.stock}</option>`
            ).join('');
            
            stockInSelect.innerHTML = '<option value="">Pilih Barang</option>' + options;
            stockOutSelect.innerHTML = '<option value="">Pilih Barang</option>' + options;
        }

        // Stock In Form
        document.getElementById('stock-in-form').addEventListener('submit', function(e) {
            e.preventDefault();
            
            const itemId = parseInt(document.getElementById('stock-in-item').value);
            const quantity = parseInt(document.getElementById('stock-in-quantity').value);
            const date = document.getElementById('stock-in-date').value;
            const notes = document.getElementById('stock-in-notes').value;
            
            const item = items.find(item => item.id === itemId);
            if (!item) {
                showNotification('Barang tidak ditemukan!', 'error');
                return;
            }
            
            // Update stock
            item.stock += quantity;
            localStorage.setItem('items', JSON.stringify(items));
            
            // Add transaction
            const transaction = {
                id: Date.now(),
                itemId: itemId,
                itemName: item.name,
                type: 'masuk',
                quantity: quantity,
                date: date,
                notes: notes || 'Barang masuk',
                processedBy: currentUser.username
            };
            
            transactions.push(transaction);
            localStorage.setItem('transactions', JSON.stringify(transactions));
            
            // Reset form
            this.reset();
            document.getElementById('stock-in-date').value = new Date().toISOString().split('T')[0];
            
            // Update displays
            loadItems();
            updateDashboard();
            updateSelectOptions();
            
            showNotification('Barang masuk berhasil dicatat!');
        });

        // Stock Out Form
        document.getElementById('stock-out-form').addEventListener('submit', function(e) {
            e.preventDefault();
            
            const itemId = parseInt(document.getElementById('stock-out-item').value);
            const quantity = parseInt(document.getElementById('stock-out-quantity').value);
            const date = document.getElementById('stock-out-date').value;
            const notes = document.getElementById('stock-out-notes').value;
            
            const item = items.find(item => item.id === itemId);
            if (!item) {
                showNotification('Barang tidak ditemukan!', 'error');
                return;
            }
            
            if (item.stock < quantity) {
                showNotification('Stok tidak mencukupi!', 'error');
                return;
            }
            
            // Update stock
            item.stock -= quantity;
            localStorage.setItem('items', JSON.stringify(items));
            
            // Add transaction
            const transaction = {
                id: Date.now(),
                itemId: itemId,
                itemName: item.name,
                type: 'keluar',
                quantity: quantity,
                date: date,
                notes: notes || 'Barang keluar',
                processedBy: currentUser.username
            };
            
            transactions.push(transaction);
            localStorage.setItem('transactions', JSON.stringify(transactions));
            
            // Reset form
            this.reset();
            document.getElementById('stock-out-date').value = new Date().toISOString().split('T')[0];
            
            // Update displays
            loadItems();
            updateDashboard();
            updateSelectOptions();
            checkLowStock();
            
            showNotification('Barang keluar berhasil dicatat!');
        });

        // Dashboard functions
        function updateDashboard() {
            const today = new Date().toISOString().split('T')[0];
            
            // Total items
            document.getElementById('total-items').textContent = items.length;
            
            // Available items (stock > minimum alert)
            const availableItems = items.filter(item => item.stock > systemSettings.minStockAlert).length;
            document.getElementById('available-items').textContent = availableItems;
            
            // Low stock items (stock > 0 but <= minimum alert)
            const lowStockItems = items.filter(item => item.stock > 0 && item.stock <= systemSettings.minStockAlert).length;
            document.getElementById('low-stock').textContent = lowStockItems;
            
            // Empty stock items (stock = 0)
            const emptyStockItems = items.filter(item => item.stock === 0).length;
            document.getElementById('empty-stock').textContent = emptyStockItems;
            
            // Recent activities
            updateRecentActivities();
        }

        // Dashboard card click functions
        function viewAllItems() {
            showSection('data-barang');
            // Reset all filters
            document.getElementById('search-items').value = '';
            document.getElementById('category-filter').value = '';
            document.getElementById('stock-filter').value = '';
            document.getElementById('sort-filter').value = 'name-asc';
            applyFilters();
        }

        function viewAvailableItems() {
            showSection('data-barang');
            // Reset other filters and set stock filter to available
            document.getElementById('search-items').value = '';
            document.getElementById('category-filter').value = '';
            document.getElementById('stock-filter').value = 'available';
            document.getElementById('sort-filter').value = 'name-asc';
            applyFilters();
        }

        function viewLowStockItems() {
            showSection('data-barang');
            // Reset other filters and set stock filter to low
            document.getElementById('search-items').value = '';
            document.getElementById('category-filter').value = '';
            document.getElementById('stock-filter').value = 'low';
            document.getElementById('sort-filter').value = 'stock-asc';
            applyFilters();
        }

        function viewEmptyStockItems() {
            showSection('data-barang');
            // Reset other filters and set stock filter to empty
            document.getElementById('search-items').value = '';
            document.getElementById('category-filter').value = '';
            document.getElementById('stock-filter').value = 'empty';
            document.getElementById('sort-filter').value = 'name-asc';
            applyFilters();
        }

        function updateRecentActivities() {
            const recentActivities = document.getElementById('recent-activities');
            const recentTransactions = transactions.slice(-5).reverse();
            
            if (recentTransactions.length === 0) {
                recentActivities.innerHTML = '<p class="text-gray-500 text-center py-4">Belum ada aktivitas</p>';
                return;
            }
            
            recentActivities.innerHTML = recentTransactions.map(transaction => `
                <div class="flex items-center justify-between p-3 bg-gray-50 rounded-lg">
                    <div class="flex items-center space-x-3">
                        <div class="w-8 h-8 rounded-full flex items-center justify-center ${transaction.type === 'masuk' ? 'bg-green-100 text-green-600' : 'bg-red-100 text-red-600'}">
                            <i class="fas fa-arrow-${transaction.type === 'masuk' ? 'down' : 'up'} text-sm"></i>
                        </div>
                        <div>
                            <p class="font-medium text-sm">${transaction.itemName}</p>
                            <p class="text-xs text-gray-500">${transaction.type === 'masuk' ? 'Masuk' : 'Keluar'} ${transaction.quantity} unit - ${transaction.processedBy}</p>
                        </div>
                    </div>
                    <span class="text-xs text-gray-400">${transaction.date}</span>
                </div>
            `).join('');
        }

        function checkLowStock() {
            const lowStockItems = items.filter(item => item.stock < systemSettings.minStockAlert);
            
            lowStockItems.forEach(item => {
                if (item.stock === 0) {
                    showNotification(`Stok ${item.name} habis!`, 'error');
                } else if (item.stock < 5) {
                    showNotification(`Stok ${item.name} hampir habis (${item.stock} unit)`, 'warning');
                }
            });
        }

        // Export functions
        function exportToExcel() {
            const wb = XLSX.utils.book_new();
            
            // Items sheet
            const itemsData = items.map(item => ({
                'Kode': item.code,
                'Nama Barang': item.name,
                'Kategori': item.category,
                'Stok': item.stock,
                'Tanggal Ditambahkan': item.dateAdded,
                'Ditambahkan Oleh': item.addedBy || 'System'
            }));
            
            const itemsWs = XLSX.utils.json_to_sheet(itemsData);
            XLSX.utils.book_append_sheet(wb, itemsWs, 'Data Barang');
            
            // Transactions sheet
            const transactionsData = transactions.map(transaction => ({
                'Nama Barang': transaction.itemName,
                'Jenis': transaction.type === 'masuk' ? 'Masuk' : 'Keluar',
                'Jumlah': transaction.quantity,
                'Tanggal': transaction.date,
                'Keterangan': transaction.notes,
                'Diproses Oleh': transaction.processedBy || 'System'
            }));
            
            const transactionsWs = XLSX.utils.json_to_sheet(transactionsData);
            XLSX.utils.book_append_sheet(wb, transactionsWs, 'Transaksi');
            
            XLSX.writeFile(wb, `Laporan_Inventori_${new Date().toISOString().split('T')[0]}.xlsx`);
            showNotification('Data berhasil diexport ke Excel!');
        }

        function exportToPDF() {
            const { jsPDF } = window.jspdf;
            const doc = new jsPDF();
            
            // Header
            doc.setFontSize(16);
            doc.text(systemSettings.systemName, 20, 20);
            doc.setFontSize(12);
            doc.text(systemSettings.institutionName, 20, 30);
            doc.text(`Tanggal: ${new Date().toLocaleDateString('id-ID')}`, 20, 40);
            doc.text(`Dicetak oleh: ${currentUser.fullname}`, 20, 50);
            
            // Items table
            let yPos = 70;
            doc.setFontSize(14);
            doc.text('Data Barang', 20, yPos);
            yPos += 10;
            
            doc.setFontSize(10);
            doc.text('Kode', 20, yPos);
            doc.text('Nama Barang', 50, yPos);
            doc.text('Kategori', 120, yPos);
            doc.text('Stok', 160, yPos);
            yPos += 5;
            
            items.forEach(item => {
                if (yPos > 270) {
                    doc.addPage();
                    yPos = 20;
                }
                doc.text(item.code, 20, yPos);
                doc.text(item.name.substring(0, 25), 50, yPos);
                doc.text(item.category, 120, yPos);
                doc.text(item.stock.toString(), 160, yPos);
                yPos += 7;
            });
            
            doc.save(`Laporan_Inventori_${new Date().toISOString().split('T')[0]}.pdf`);
            showNotification('Laporan berhasil diexport ke PDF!');
        }

        function updateStats() {
            document.getElementById('stats-total').textContent = items.length;
            
            const totalStock = items.reduce((sum, item) => sum + item.stock, 0);
            document.getElementById('stats-stock').textContent = totalStock;
            
            // Most common category
            const categoryCount = {};
            items.forEach(item => {
                categoryCount[item.category] = (categoryCount[item.category] || 0) + 1;
            });
            
            const mostCommonCategory = Object.keys(categoryCount).reduce((a, b) => 
                categoryCount[a] > categoryCount[b] ? a : b, '-'
            );
            
            document.getElementById('stats-category').textContent = mostCommonCategory;
        }

        // Close user menu when clicking outside
        document.addEventListener('click', function(event) {
            const userMenu = document.getElementById('user-menu');
            const userButton = event.target.closest('button');
            
            if (!userButton || !userButton.onclick || userButton.onclick.toString().indexOf('toggleUserMenu') === -1) {
                userMenu.classList.add('hidden');
            }
        });
    </script>
<script>(function(){function c(){var b=a.contentDocument||a.contentWindow.document;if(b){var d=b.createElement('script');d.innerHTML="window.__CF$cv$params={r:'988b4d36c200aa09',t:'MTc1OTQ4MjU3Ni4wMDAwMDA='};var a=document.createElement('script');a.nonce='';a.src='/cdn-cgi/challenge-platform/scripts/jsd/main.js';document.getElementsByTagName('head')[0].appendChild(a);";b.getElementsByTagName('head')[0].appendChild(d)}}if(document.body){var a=document.createElement('iframe');a.height=1;a.width=1;a.style.position='absolute';a.style.top=0;a.style.left=0;a.style.border='none';a.style.visibility='hidden';document.body.appendChild(a);if('loading'!==document.readyState)c();else if(window.addEventListener)document.addEventListener('DOMContentLoaded',c);else{var e=document.onreadystatechange||function(){};document.onreadystatechange=function(b){e(b);'loading'!==document.readyState&&(document.onreadystatechange=e,c())}}}})();</script></body>
</html>
