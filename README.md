<!DOCTYPE html>
<html lang="id">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Karya Makmur Super App</title>
    
    <script src="https://cdnjs.cloudflare.com/ajax/libs/PapaParse/5.3.0/papaparse.min.js"></script>
    <link rel="stylesheet" href="https://unpkg.com/leaflet@1.9.4/dist/leaflet.css" />
    <link rel="stylesheet" href="https://unpkg.com/leaflet-routing-machine/dist/leaflet-routing-machine.css" />

    <style>
        :root { --p: #2ecc71; --d: #2c3e50; --bg: #f8f9fa; }
        body { font-family: 'Segoe UI', sans-serif; margin: 0; background: var(--bg); }
        
        /* Navigasi Bawah */
        .bottom-nav { position: fixed; bottom: 0; width: 100%; background: white; display: flex; border-top: 1px solid #ddd; z-index: 1000; }
        .nav-item { flex: 1; text-align: center; padding: 12px; font-size: 11px; cursor: pointer; color: #888; }
        .nav-item.active { color: var(--p); font-weight: bold; }

        /* Halaman */
        .page { display: none; padding: 15px; padding-bottom: 80px; max-width: 500px; margin: 0 auto; }
        .page.active { display: block; animation: fadeIn 0.3s; }
        @keyframes fadeIn { from {opacity: 0;} to {opacity: 1;} }

        header { background: var(--p); color: white; padding: 20px; border-radius: 0 0 20px 20px; text-align: center; margin: -15px -15px 15px -15px; }
        
        /* Produk */
        .item-grid { display: grid; grid-template-columns: 1fr 1fr; gap: 10px; }
        .item-card { background: white; border-radius: 10px; overflow: hidden; border: 1px solid #eee; display: flex; flex-direction: column; }
        .item-img { width: 100%; height: 110px; object-fit: cover; background: #eee; }
        .item-info { padding: 8px; }
        
        /* Form & Button */
        input, select, button { width: 100%; padding: 12px; margin-top: 8px; border-radius: 8px; border: 1px solid #ddd; box-sizing: border-box; }
        .btn-p { background: var(--p); color: white; border: none; font-weight: bold; cursor: pointer; }
        .btn-b { background: #3498db; color: white; border: none; }
        
        #map { height: 250px; width: 100%; border-radius: 10px; margin-top: 10px; }
        .summary-box { background: #eef2f3; padding: 10px; border-radius: 8px; font-size: 13px; margin-top: 10px; }
    </style>
</head>
<body>

    <div id="home" class="page active">
        <header>
            <h3 style="margin:0">Karya Makmur Market</h3>
            <p style="font-size:11px">Belanja Mudah dari Rumah</p>
        </header>
        <div id="loader" style="text-align:center; padding:30px;">Memuat data...</div>
        <div id="itemList" class="item-grid"></div>
    </div>

    <div id="delivery" class="page">
        <h3>🛵 Pengiriman</h3>
        <div class="summary-box" id="cartText">Keranjang belanja kosong.</div>
        <button class="btn-b" onclick="getLocation()">📍 Klik Deteksi Lokasi</button>
        <div id="map"></div>
        <div id="ongkirRes" class="summary-box" style="display:none; background:#fff3cd;"></div>
        <button id="btnWA" class="btn-p" style="background:#25D366; margin-top:15px;" onclick="sendToWA()" disabled>Pesan via WhatsApp</button>
    </div>

    <div id="admin" class="page">
        <h3>⚙️ Tambah Barang Baru</h3>
        <div style="background:white; padding:15px; border-radius:10px;">
            <input type="text" id="adm_toko" placeholder="Nama Toko">
            <select id="adm_kat">
                <option value="Hasil Alam">Hasil Alam</option>
                <option value="Makanan">Makanan</option>
                <option value="Minuman">Minuman</option>
                <option value="Otomotif">Otomotif</option>
            </select>
            <input type="text" id="adm_nama" placeholder="Nama Produk">
            <input type="number" id="adm_harga" placeholder="Harga (Misal: 25000)">
            <input type="text" id="adm_foto" placeholder="Link Foto Produk">
            <button id="btnSave" class="btn-p" onclick="saveToSheet()">Simpan ke Google Sheet</button>
        </div>
    </div>

    <div class="bottom-nav">
        <div class="nav-item active" onclick="nav('home', this)">Beranda</div>
        <div class="nav-item" onclick="nav('delivery', this)">Antar</div>
        <div class="nav-item" onclick="nav('admin', this)">Admin</div>
    </div>

    <script src="https://unpkg.com/leaflet@1.9.4/dist/leaflet.js"></script>
    <script src="https://unpkg.com/leaflet-routing-machine/dist/leaflet-routing-machine.js"></script>

    <script>
        // --- KONFIGURASI ---
        const csvURL = "https://docs.google.com/spreadsheets/d/e/2PACX-1vQjssXRMg7zBG3i7gXEwFBHUtGMb0DrvmSWaYnMaY6gmPFs9gNn---LoUx7S_jBwmErpYj5vXJdL3XV/pub?output=csv";
        const scriptURL = "https://script.google.com/macros/s/AKfycbwrPd8M7PaKYIiqdLcP-RAQ5EN3M-2uyN-y_K4uiWGCr-FvTabnng9XNgVxni7l1dYI/exec";
        const myWA = "6285841889968";
        const tokoPos = [-2.9723, 104.7634]; // Ganti koordinat toko Anda

        let cart = { msg: "", total: 0, berat: 0 };
        let userPos = null;
        let finalOngkir = 0;

        // Ambil Data
        function loadItems() {
            Papa.parse(csvURL, {
                download: true, header: true,
                complete: function(res) {
                    const list = document.getElementById('itemList');
                    list.innerHTML = '';
                    res.data.forEach(item => {
                        if(!item.nama_produk) return;
                        list.innerHTML += `
                            <div class="item-card">
                                <img src="${item.foto_produk || 'https://via.placeholder.com/150'}" class="item-img">
                                <div class="item-info">
                                    <div style="font-size:13px; font-weight:bold;">${item.nama_produk}</div>
                                    <div style="color:green; font-size:12px;">Rp ${parseInt(item.harga_produk).toLocaleString()}</div>
                                    <button onclick="add('${item.nama_produk}', ${item.harga_produk})" style="padding:5px; font-size:10px; background:#2ecc71; color:white; border:none; border-radius:4px; margin-top:5px; cursor:pointer;">Pilih</button>
                                </div>
                            </div>
                        `;
                    });
                    document.getElementById('loader').style.display = 'none';
                }
            });
        }

        function add(name, price) {
            cart.msg += `- ${name} (1x)\n`;
            cart.total += parseInt(price);
            cart.berat += 0.5; // Estimasi 0.5kg per item
            document.getElementById('cartText').innerText = cart.msg + "\nSubtotal Barang: Rp " + cart.total.toLocaleString();
            alert(name + " dipilih!");
        }

        function nav(id, el) {
            document.querySelectorAll('.page').forEach(p => p.classList.remove('active'));
            document.querySelectorAll('.nav-item').forEach(n => n.classList.remove('active'));
            document.getElementById(id).classList.add('active');
            el.classList.add('active');
            if(id === 'delivery') setTimeout(() => map.invalidateSize(), 200);
        }

        // Peta
        const map = L.map('map').setView(tokoPos, 14);
        L.tileLayer('https://{s}.tile.openstreetmap.org/{z}/{x}/{y}.png').addTo(map);
        L.marker(tokoPos).addTo(map).bindPopup("Toko");

        function getLocation() {
            navigator.geolocation.getCurrentPosition(p => {
                userPos = [p.coords.latitude, p.coords.longitude];
                L.Routing.control({
                    waypoints: [L.latLng(tokoPos), L.latLng(userPos)],
                    addWaypoints: false, draggableWaypoints: false, show: false
                }).on('routesfound', function(e) {
                    let dist = e.routes[0].summary.totalDistance / 1000;
                    finalOngkir = (Math.ceil(dist) * 5000) + (cart.berat * 500);
                    const res = document.getElementById('ongkirRes');
                    res.style.display = 'block';
                    res.innerHTML = `Jarak: ${dist.toFixed(1)} km <br> Ongkir: Rp ${finalOngkir.toLocaleString()} <br> <b>Total Bayar: Rp ${(cart.total + finalOngkir).toLocaleString()}</b>`;
                    document.getElementById('btnWA').disabled = false;
                }).addTo(map);
            });
        }

        function sendToWA() {
            const maps = `https://www.google.com/maps?q=${userPos[0]},${userPos[1]}`;
            const text = `*PESANAN BARU*%0A${cart.msg}%0A---%0ATotal Barang: Rp ${cart.total.toLocaleString()}%0AOngkir: Rp ${finalOngkir.toLocaleString()}%0A*TOTAL BAYAR: Rp ${(cart.total + finalOngkir).toLocaleString()}*%0A%0ALokasi: ${maps}`;
            window.open(`https://wa.me/${myWA}?text=${text}`);
        }

        // Simpan ke Sheet via Apps Script
        function saveToSheet() {
            const btn = document.getElementById('btnSave');
            btn.innerText = "Mengirim..."; btn.disabled = true;

            const data = {
                id_toko: Date.now(),
                nama_toko: document.getElementById('adm_toko').value,
                kategori_segmen: document.getElementById('adm_kat').value,
                nama_produk: document.getElementById('adm_nama').value,
                harga_produk: document.getElementById('adm_harga').value,
                foto_produk: document.getElementById('adm_foto').value
            };

            fetch(scriptURL, {
                method: "POST",
                body: JSON.stringify(data)
            })
            .then(() => {
                alert("Berhasil! Data akan muncul dalam beberapa menit.");
                location.reload();
            })
            .catch(() => alert("Gagal mengirim data."));
        }

        loadItems();
    </script>
</body>
</html>
