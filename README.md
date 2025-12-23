<!DOCTYPE html>
<html lang="id">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Pasar Digital Karya Makmur</title>
    <script src="https://cdnjs.cloudflare.com/ajax/libs/PapaParse/5.3.0/papaparse.min.js"></script>
    
    <style>
        :root {
            --primary: #2ecc71; --dark: #2c3e50; --bg: #f8f9fa;
        }
        body { font-family: 'Segoe UI', sans-serif; background: var(--bg); margin: 0; padding-bottom: 80px; }
        
        /* Header */
        header { background: linear-gradient(135deg, #27ae60, #2980b9); color: white; padding: 20px; border-radius: 0 0 20px 20px; position: sticky; top: 0; z-index: 10; box-shadow: 0 4px 10px rgba(0,0,0,0.1); }
        .search-box input { width: 100%; padding: 12px; border-radius: 25px; border: none; margin-top: 10px; box-sizing: border-box; }
        
        /* Kategori Tabs */
        .category-scroll { display: flex; overflow-x: auto; padding: 15px 20px; gap: 10px; -webkit-overflow-scrolling: touch; }
        .category-scroll::-webkit-scrollbar { display: none; }
        .cat-chip { background: white; padding: 8px 16px; border-radius: 20px; white-space: nowrap; font-size: 0.9rem; border: 1px solid #ddd; cursor: pointer; transition: 0.3s; }
        .cat-chip.active { background: var(--dark); color: white; border-color: var(--dark); }

        /* Shop Grid */
        .shop-container { padding: 10px 20px; display: grid; grid-template-columns: repeat(auto-fill, minmax(150px, 1fr)); gap: 15px; }
        .shop-card { background: white; border-radius: 12px; overflow: hidden; box-shadow: 0 2px 8px rgba(0,0,0,0.1); cursor: pointer; position: relative; }
        .shop-card:active { transform: scale(0.98); }
        .shop-img { height: 100px; display: flex; align-items: center; justify-content: center; font-size: 40px; background: #ecf0f1; }
        .shop-info { padding: 10px; }
        .shop-name { font-weight: bold; font-size: 0.9rem; color: var(--dark); margin-bottom: 5px;}
        .shop-tag { font-size: 0.7rem; background: #eee; padding: 2px 6px; border-radius: 4px; color: #555; }
        .badge-closed { position: absolute; top: 10px; right: 10px; background: red; color: white; padding: 2px 8px; font-size: 10px; border-radius: 10px; }

        /* Loader */
        #loader { text-align: center; padding: 20px; color: #777; }

        /* Modal Styles */
        .modal { display: none; position: fixed; z-index: 100; left: 0; top: 0; width: 100%; height: 100%; background-color: rgba(0,0,0,0.5); align-items: flex-end; justify-content: center; }
        .modal-content { background-color: #fff; width: 100%; max-width: 500px; border-radius: 20px 20px 0 0; padding: 20px; max-height: 85vh; overflow-y: auto; animation: slideUp 0.3s; }
        @keyframes slideUp { from {transform: translateY(100%);} to {transform: translateY(0);} }
        
        /* Item Row dengan Gambar */
        .item-row { display: flex; align-items: center; margin-bottom: 15px; border-bottom: 1px dashed #eee; padding-bottom: 10px; gap: 12px; }
        
        /* Thumbnail Produk */
        .prod-thumb { width: 60px; height: 60px; border-radius: 8px; object-fit: cover; background: #eee; }
        .prod-thumb-placeholder { width: 60px; height: 60px; border-radius: 8px; background: #e0e0e0; display: flex; align-items: center; justify-content: center; font-weight: bold; color: #888; font-size: 20px; }

        .item-info { flex: 1; }
        .item-controls { display: flex; align-items: center; gap: 8px; }
        .btn-qty { width: 30px; height: 30px; border-radius: 50%; border: 1px solid #ddd; background: white; cursor: pointer; font-weight: bold; }
        
        .checkout-bar { margin-top: 20px; background: #2c3e50; color: white; padding: 15px; border-radius: 10px; display: flex; justify-content: space-between; align-items: center; cursor: pointer; }
    </style>
</head>
<body>

    <header>
        <h3>Desa Karya Makmur 🇮🇩</h3>
        <div class="search-box">
            <input type="text" id="searchInput" placeholder="Cari barang atau toko..." onkeyup="filterShops()">
        </div>
    </header>

    <div class="category-scroll">
        <div class="cat-chip active" onclick="filterCategory('Semua', this)">Semua</div>
        <div class="cat-chip" onclick="filterCategory('Makanan', this)">🍲 Makanan</div>
        <div class="cat-chip" onclick="filterCategory('Minuman', this)">🥤 Minuman</div>
        <div class="cat-chip" onclick="filterCategory('Hasil Alam', this)">🌾 Hasil Alam</div>
        <div class="cat-chip" onclick="filterCategory('Otomotif', this)">🔧 Otomotif</div>
    </div>

    <div id="loader">Sedang mengambil data terbaru... 🔄</div>
    <div class="shop-container" id="shopList"></div>

    <div id="productModal" class="modal">
        <div class="modal-content">
            <div style="display:flex; justify-content:space-between; align-items:center;">
                <h3 id="modalShopName" style="margin:0;">Nama Toko</h3>
                <span onclick="closeModal()" style="font-size:24px; cursor:pointer;">&times;</span>
            </div>
            <p id="modalShopDesc" style="font-size:12px; color:#777; margin-bottom:15px;"></p>
            
            <div id="productList"></div>

            <div class="checkout-bar" onclick="processCheckout()">
                <div><small>Total:</small><br><strong id="totalPrice">Rp 0</strong></div>
                <div style="font-weight: bold;">Pesan & Antar 🛵 ></div>
            </div>
        </div>
    </div>

    <script>
        // --- LINK CSV GOOGLE SHEET ---
        // Pastikan Anda sudah menambah kolom 'foto_produk' di Google Sheet!
        const googleSheetURL = "https://docs.google.com/spreadsheets/d/e/2PACX-1vQjssXRMg7zBG3i7gXEwFBHUtGMb0DrvmSWaYnMaY6gmPFs9gNn---LoUx7S_jBwmErpYj5vXJdL3XV/pub?output=csv"; 

        let allShops = [];
        let groupedShops = {};
        let currentCart = {};
        let activeShopId = null;

        function loadData() {
            Papa.parse(googleSheetURL, {
                download: true, header: true,
                complete: function(results) { processData(results.data); },
                error: function(err) { document.getElementById('loader').innerText = "Gagal koneksi."; }
            });
        }

        function processData(data) {
            groupedShops = {};
            data.forEach(row => {
                if(row.id_toko && row.nama_toko) {
                    const id = row.id_toko;
                    if (!groupedShops[id]) {
                        groupedShops[id] = {
                            id: id,
                            name: row.nama_toko,
                            category: row.kategori_segmen || "Lainnya",
                            desc: row.deskripsi_toko || "",
                            isOpen: (row.status_buka && row.status_buka.toLowerCase() === 'true'),
                            emoji: row.emoji || "🏪",
                            products: []
                        };
                    }
                    if(row.nama_produk) {
                        groupedShops[id].products.push({
                            name: row.nama_produk,
                            price: parseInt(row.harga_produk) || 0,
                            image: row.foto_produk || "" // Membaca kolom foto_produk
                        });
                    }
                }
            });
            allShops = Object.values(groupedShops);
            document.getElementById('loader').style.display = 'none';
            renderShops(allShops);
        }

        // --- RENDER TOKO ---
        const shopContainer = document.getElementById('shopList');
        function renderShops(data) {
            shopContainer.innerHTML = '';
            if(data.length === 0) { shopContainer.innerHTML = '<p style="text-align:center;">Data kosong.</p>'; return; }
            data.forEach(shop => {
                const opacity = shop.isOpen ? '1' : '0.6';
                const badge = shop.isOpen ? '' : '<div class="badge-closed">TUTUP</div>';
                const clickAction = shop.isOpen ? `onclick="openShop('${shop.id}')"` : `onclick="alert('Toko Tutup')"` ;
                shopContainer.innerHTML += `
                    <div class="shop-card" style="opacity: ${opacity};" ${clickAction}>
                        ${badge}
                        <div class="shop-img">${shop.emoji}</div>
                        <div class="shop-info">
                            <div class="shop-name">${shop.name}</div>
                            <span class="shop-tag">${shop.category}</span>
                        </div>
                    </div>
                `;
            });
        }

        function filterCategory(cat, el) {
            document.querySelectorAll('.cat-chip').forEach(e => e.classList.remove('active'));
            el.classList.add('active');
            renderShops(cat === 'Semua' ? allShops : allShops.filter(s => s.category.toLowerCase().includes(cat.toLowerCase())));
        }

        function filterShops() {
            const q = document.getElementById('searchInput').value.toLowerCase();
            renderShops(allShops.filter(s => s.name.toLowerCase().includes(q) || s.products.some(p => p.name.toLowerCase().includes(q))));
        }

        // --- LOGIC PRODUK & GAMBAR ---
        const modal = document.getElementById('productModal');
        const productList = document.getElementById('productList');

        function openShop(id) {
            const shop = groupedShops[id];
            activeShopId = id;
            document.getElementById('modalShopName').innerText = shop.name;
            document.getElementById('modalShopDesc').innerText = shop.desc;
            currentCart = {}; 
            renderProducts(shop);
            updateTotal();
            modal.style.display = "flex";
        }
        function closeModal() { modal.style.display = "none"; }

        function renderProducts(shop) {
            productList.innerHTML = '';
            shop.products.forEach(prod => {
                const qty = currentCart[prod.name] || 0;
                
                // Cek ada gambar atau tidak
                let imgHTML = '';
                if(prod.image && prod.image.length > 5) {
                    imgHTML = `<img src="${prod.image}" class="prod-thumb" alt="${prod.name}">`;
                } else {
                    // Jika tidak ada gambar, pakai inisial huruf
                    imgHTML = `<div class="prod-thumb-placeholder">${prod.name.charAt(0)}</div>`;
                }

                productList.innerHTML += `
                    <div class="item-row">
                        ${imgHTML}
                        <div class="item-info">
                            <div style="font-weight:600;">${prod.name}</div>
                            <div style="color:#2ecc71;">Rp ${prod.price.toLocaleString()}</div>
                        </div>
                        <div class="item-controls">
                            <button class="btn-qty" onclick="updateQty('${prod.name}', ${prod.price}, -1)">-</button>
                            <span style="width:20px; text-align:center; font-weight:bold;">${qty}</span>
                            <button class="btn-qty" onclick="updateQty('${prod.name}', ${prod.price}, 1)">+</button>
                        </div>
                    </div>
                `;
            });
        }

        function updateQty(name, price, change) {
            if (!currentCart[name]) currentCart[name] = 0;
            currentCart[name] += change;
            if (currentCart[name] < 0) currentCart[name] = 0;
            renderProducts(groupedShops[activeShopId]); // Re-render untuk update UI
            updateTotal();
        }

        function updateTotal() {
            const s = groupedShops[activeShopId];
            let total = 0;
            if(s) s.products.forEach(p => total += (currentCart[p.name]||0) * p.price);
            document.getElementById('totalPrice').innerText = "Rp " + total.toLocaleString();
        }

        function processCheckout() {
            const shop = groupedShops[activeShopId];
            let orderText = `Halo ${shop.name}, saya mau pesan:%0A`;
            let weight = 0; let total = 0; let hasItem = false;

            shop.products.forEach(p => {
                const qty = currentCart[p.name] || 0;
                if (qty > 0) {
                    orderText += `- ${p.name} (${qty}x)%0A`;
                    total += qty * p.price;
                    // Estimasi berat
                    if(p.name.toLowerCase().includes("kg") || p.name.toLowerCase().includes("liter")) weight += qty;
                    else if (shop.category.toLowerCase().includes("otomotif")) weight += (qty * 0.5);
                    else weight += (qty * 0.2);
                    hasItem = true;
                }
            });

            if (!hasItem) { alert("Pilih barang dulu!"); return; }
            orderText += `%0ATotal Barang: Rp ${total.toLocaleString()}`;
            
            const params = new URLSearchParams();
            params.append("pesanan", orderText);
            params.append("berat", Math.ceil(weight));
            window.location.href = `index.html?${params.toString()}`;
        }

        window.onclick = function(e) { if (e.target == modal) closeModal(); }
        loadData();
    </script>
</body>
</html>
