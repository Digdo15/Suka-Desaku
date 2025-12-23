<!DOCTYPE html>
<html lang="id">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Aplikasi Super Karya Makmur</title>
    
    <script src="https://cdnjs.cloudflare.com/ajax/libs/PapaParse/5.3.0/papaparse.min.js"></script>
    <link rel="stylesheet" href="https://unpkg.com/leaflet@1.9.4/dist/leaflet.css" />
    <link rel="stylesheet" href="https://unpkg.com/leaflet-routing-machine/dist/leaflet-routing-machine.css" />

    <style>
        :root { --p: #2ecc71; --d: #2c3e50; }
        body { font-family: sans-serif; margin: 0; background: #f4f4f4; }
        
        /* Navigasi Bawah */
        .bottom-nav { position: fixed; bottom: 0; width: 100%; background: white; display: flex; box-shadow: 0 -2px 10px rgba(0,0,0,0.1); z-index: 1000; }
        .nav-item { flex: 1; text-align: center; padding: 15px; font-size: 12px; cursor: pointer; color: #777; }
        .nav-item.active { color: var(--p); font-weight: bold; }

        /* Halaman */
        .page { display: none; padding: 20px; padding-bottom: 80px; }
        .page.active { display: block; }

        /* Card & UI */
        header { background: var(--p); color: white; padding: 20px; border-radius: 0 0 20px 20px; margin-bottom: 10px; }
        .shop-grid { display: grid; grid-template-columns: 1fr 1fr; gap: 10px; }
        .card { background: white; padding: 10px; border-radius: 10px; box-shadow: 0 2px 5px rgba(0,0,0,0.05); }
        #map { height: 300px; width: 100%; border-radius: 10px; margin-top: 10px; }
        
        /* Form Edit */
        input, select, button { width: 100%; padding: 12px; margin-top: 10px; border-radius: 8px; border: 1px solid #ddd; box-sizing: border-box; }
        .btn-main { background: var(--p); color: white; border: none; font-weight: bold; cursor: pointer; }
    </style>
</head>
<body>

    <div id="home" class="page active">
        <header>
            <h2 style="margin:0">Pasar Karya Makmur</h2>
            <p style="font-size:12px">Beli Ikan, Jajanan & Kebutuhan</p>
        </header>
        <div id="loader">Memuat barang...</div>
        <div id="shopList" class="shop-grid"></div>
    </div>

    <div id="delivery" class="page">
        <h3>📍 Detail Pengantaran</h3>
        <div class="card">
            <label>Ringkasan Pesanan:</label>
            <div id="orderSummary" style="font-size: 13px; color: #555; background: #eee; padding: 10px; border-radius: 5px;">Belum ada pesanan.</div>
            <button class="btn-main" onclick="getLocation()" style="background: #3498db;">Deteksi Lokasi Saya</button>
            <div id="map"></div>
            <div id="ongkirDetail"></div>
            <button id="btnWA" class="btn-main" onclick="sendToWA()" disabled>Pesan via WhatsApp</button>
        </div>
    </div>

    <div id="admin" class="page">
        <h3>⚙️ Kelola Barang</h3>
        <p style="font-size:12px; color:red">Khusus Admin: Tambahkan barang langsung ke Google Sheet</p>
        <div class="card">
            <input type="text" id="add_nama_toko" placeholder="Nama Toko">
            <select id="add_kategori">
                <option value="Hasil Alam">Hasil Alam (Ikan/Sayur)</option>
                <option value="Makanan">Makanan (Seblak/Jajanan)</option>
                <option value="Otomotif">Otomotif</option>
            </select>
            <input type="text" id="add_nama_produk" placeholder="Nama Barang">
            <input type="number" id="add_harga" placeholder="Harga (Contoh: 25000)">
            <input type="text" id="add_foto" placeholder="Link Foto (URL)">
            <button class="btn-main" onclick="saveToSheet()">Simpan ke Google Sheet</button>
        </div>
    </div>

    <div class="bottom-nav">
        <div class="nav-item active" onclick="showPage('home')">🏠 Beranda</div>
        <div class="nav-item" onclick="showPage('delivery')">🛵 Antar</div>
        <div class="nav-item" onclick="showPage('admin')">⚙️ Edit</div>
    </div>

    <script src="https://unpkg.com/leaflet@1.9.4/dist/leaflet.js"></script>
    <script src="https://unpkg.com/leaflet-routing-machine/dist/leaflet-routing-machine.js"></script>

    <script>
        // --- KONFIGURASI ---
        const googleSheetCSV = "LINK_CSV_ANDA"; 
        const scriptURL = "LINK_WEB_APP_APPS_SCRIPT_ANDA"; // Dari Langkah 1
        const adminNum = "6285841889968";

        // State Aplikasi
        let cart = { items: "", totalHarga: 0, berat: 0 };
        let userPos = null;

        // 1. Fungsi Navigasi Halaman
        function showPage(pageId) {
            document.querySelectorAll('.page').forEach(p => p.classList.remove('active'));
            document.querySelectorAll('.nav-item').forEach(n => n.classList.remove('active'));
            document.getElementById(pageId).classList.add('active');
            event.currentTarget.classList.add('active');
        }

        // 2. Fungsi Ambil Data (Sama seperti sebelumnya)
        function loadData() {
            Papa.parse(googleSheetCSV, {
                download: true, header: true,
                complete: function(res) { renderShops(res.data); }
            });
        }

        function renderShops(data) {
            const list = document.getElementById('shopList');
            list.innerHTML = '';
            document.getElementById('loader').style.display = 'none';
            // Logic render toko + tombol beli (Singkatnya: Saat beli, update object cart & pindah ke page delivery)
            // ... (Kode render sama dengan sebelumnya) ...
        }

        // 3. Fungsi EDIT BARANG (Simpan ke Google Sheet)
        function saveToSheet() {
            const data = {
                id_toko: Date.now(), // ID Unik dari waktu
                nama_toko: document.getElementById('add_nama_toko').value,
                kategori_segmen: document.getElementById('add_kategori').value,
                deskripsi_toko: "Toko Baru di Desa",
                emoji: "🏪",
                nama_produk: document.getElementById('add_nama_produk').value,
                harga_produk: document.getElementById('add_harga').value,
                foto_produk: document.getElementById('add_foto').value
            };

            fetch(scriptURL, {
                method: 'POST',
                body: JSON.stringify(data)
            })
            .then(res => alert("Tersimpan! Harap tunggu 5 menit sampai Google memperbarui datanya."))
            .catch(err => alert("Gagal Simpan"));
        }

        // 4. Fungsi Peta (Sama seperti sebelumnya di file index.html)
        // ... (Fungsi getLocation, routing, calculate total ongkir, sendToWA) ...

        loadData();
    </script>
</body>
</html>
