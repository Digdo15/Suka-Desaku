<!DOCTYPE html>
<html lang="id">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Karya Makmur Pro - Server Jual Beli</title>
    <script src="https://cdn.tailwindcss.com"></script>
    <link rel="stylesheet" href="https://cdnjs.cloudflare.com/ajax/libs/font-awesome/6.4.0/css/all.min.css">
    <style>
        @import url('https://fonts.googleapis.com/css2?family=Plus+Jakarta+Sans:wght@400;600;700&display=swap');
        body { font-family: 'Plus Jakarta Sans', sans-serif; background: #f8fafc; }
    </style>
</head>
<body class="p-4 md:p-8">

    <div class="max-w-7xl mx-auto">
        <div class="flex flex-col md:flex-row justify-between items-center mb-10 gap-4">
            <div class="flex items-center gap-4">
                <div class="bg-blue-600 p-3 rounded-2xl shadow-lg text-white">
                    <i class="fas fa-server fa-lg"></i>
                </div>
                <div>
                    <h1 class="text-3xl font-bold text-slate-800">Karya Makmur <span class="text-blue-600 font-normal">SaaS</span></h1>
                    <p id="lastUpdate" class="text-xs text-slate-500 font-medium italic">Menghubungkan ke Server...</p>
                </div>
            </div>
            <div id="status" class="flex items-center gap-2 px-5 py-2 rounded-full bg-slate-200 text-slate-600 font-bold text-xs uppercase tracking-tighter transition-all">
                Offline
            </div>
        </div>

        <div class="grid grid-cols-1 lg:grid-cols-12 gap-8">
            <div class="lg:col-span-4">
                <div class="bg-white p-6 rounded-3xl border border-slate-200 shadow-sm sticky top-8">
                    <h3 class="font-bold text-slate-800 mb-6 flex items-center gap-2 text-lg">
                        <i class="fas fa-cart-plus text-blue-600"></i> Kelola Transaksi
                    </h3>
                    <input type="hidden" id="rowIndex">
                    <input type="hidden" id="currentID">

                    <div class="space-y-4">
                        <div class="bg-slate-50 p-4 rounded-2xl">
                            <label class="text-[10px] font-bold text-slate-400 uppercase mb-2 block tracking-widest">Identitas Mitra</label>
                            <input type="text" id="namaToko" placeholder="Nama Toko" class="w-full p-3 border rounded-xl text-sm mb-2 outline-none focus:ring-2 focus:ring-blue-500">
                            <input type="text" id="pemilik" placeholder="Nama Pemilik" class="w-full p-3 border rounded-xl text-sm mb-2 outline-none">
                            <input type="text" id="nomorHp" placeholder="Nomor WA (628...)" class="w-full p-3 border rounded-xl text-sm outline-none">
                        </div>
                        
                        <div class="bg-blue-50/50 p-4 rounded-2xl border border-blue-100">
                            <label class="text-[10px] font-bold text-blue-400 uppercase mb-2 block tracking-widest">Detail Barang & Harga</label>
                            <input type="text" id="barang" placeholder="Nama Barang" class="w-full p-3 border rounded-xl text-sm mb-2 outline-none">
                            <div class="grid grid-cols-2 gap-2 mb-2">
                                <input type="number" id="hargaModal" placeholder="Harga Modal" class="p-3 border rounded-xl text-sm outline-none">
                                <input type="number" id="hargaJual" placeholder="Harga Jual" class="p-3 border rounded-xl text-sm outline-none">
                            </div>
                            <div class="grid grid-cols-2 gap-2">
                                <input type="text" id="jenis" placeholder="Kategori" class="p-3 border rounded-xl text-sm outline-none">
                                <input type="number" id="terjual" placeholder="Qty Terjual" class="p-3 border rounded-xl text-sm outline-none">
                            </div>
                        </div>

                        <textarea id="alamat" placeholder="Alamat Toko Lengkap" rows="2" class="w-full p-3 bg-slate-50 border rounded-2xl text-sm outline-none"></textarea>

                        <button onclick="simpan()" id="btnSimpan" class="w-full bg-blue-600 hover:bg-blue-700 text-white py-4 rounded-2xl font-bold shadow-lg shadow-blue-100 transition-all flex justify-center items-center gap-3 active:scale-95">
                            SIMPAN KE SERVER
                        </button>
                    </div>
                </div>
            </div>

            <div class="lg:col-span-8">
                <div class="bg-white rounded-3xl border border-slate-200 shadow-sm overflow-hidden">
                    <div class="p-5 border-b flex justify-between items-center bg-white">
                        <span class="text-xs font-bold text-slate-400 uppercase tracking-widest">Database Transaksi</span>
                        <button onclick="loadData()" class="text-blue-600 p-2 hover:bg-blue-50 rounded-lg transition-all"><i class="fas fa-sync-alt"></i> Refresh</button>
                    </div>
                    <div class="overflow-x-auto">
                        <table class="w-full text-left text-sm">
                            <thead class="bg-slate-50 text-slate-400 text-[10px] font-bold uppercase tracking-widest border-b">
                                <tr>
                                    <th class="p-5">Barang</th>
                                    <th class="p-5">Toko</th>
                                    <th class="p-5 text-center">Terjual</th>
                                    <th class="p-5 text-right">Harga Jual</th>
                                    <th class="p-5 text-right">Aksi</th>
                                </tr>
                            </thead>
                            <tbody id="dataTable" class="divide-y divide-slate-100 italic">
                                <tr><td colspan="5" class="p-20 text-center text-slate-400">Memuat data dari server...</td></tr>
                            </tbody>
                        </table>
                    </div>
                </div>
            </div>
        </div>
    </div>

    <script>
        // MASUKKAN URL /exec ANDA DI SINI
        const API_URL = "https://script.google.com/macros/s/AKfycbzYGra7liFKJm0e1XOz9zG28dih034kV8uPP26OA_Ac07zrfVg598vYIjTt01ACfRMfaw/exec";

        async function loadData() {
            const statusEl = document.getElementById("status");
            try {
                const res = await fetch(API_URL);
                const data = await res.json();
                
                statusEl.innerText = "Server: Online";
                statusEl.className = "px-5 py-2 rounded-full bg-emerald-50 text-emerald-600 font-bold text-xs uppercase tracking-tighter shadow-sm border border-emerald-100 transition-all";
                document.getElementById("lastUpdate").innerText = "Update: " + new Date().toLocaleTimeString();

                const tbody = document.getElementById("dataTable");
                tbody.innerHTML = "";

                data.slice().reverse().forEach((d) => {
                    if(!d["Barang dijual"]) return;
                    tbody.innerHTML += `
                    <tr class="hover:bg-blue-50/50 transition">
                        <td class="p-5">
                            <div class="font-bold text-slate-800">${d["Barang dijual"]}</div>
                            <div class="text-[10px] text-slate-400 italic">ID: ${d["ID"] || '-'}</div>
                        </td>
                        <td class="p-5 font-semibold text-slate-600">${d["Nama Toko"] || '-'}</td>
                        <td class="p-5 text-center"><span class="bg-blue-100 text-blue-600 px-3 py-1 rounded-full text-[10px] font-bold">${d["Terjual"] || 0} Unit</span></td>
                        <td class="p-5 text-right font-bold text-emerald-600">Rp ${Number(d["Harga jual"] || 0).toLocaleString()}</td>
                        <td class="p-5 text-right">
                            <button class="bg-slate-100 p-2 rounded-lg text-slate-400 hover:bg-blue-600 hover:text-white transition shadow-sm"
                            onclick='editData(${JSON.stringify(d)})'><i class="fas fa-edit"></i></button>
                        </td>
                    </tr>`;
                });
            } catch (e) {
                statusEl.innerText = "Server: Offline";
                statusEl.className = "px-5 py-2 rounded-full bg-red-50 text-red-600 font-bold text-xs uppercase tracking-tighter border border-red-100 transition-all";
            }
        }

        async function simpan() {
            const btn = document.getElementById("btnSimpan");
            
            // Mengumpulkan data dari form
            const payload = {
                row_index: document.getElementById("rowIndex").value,
                id: document.getElementById("currentID").value,
                nama_toko: document.getElementById("namaToko").value,
                pemilik: document.getElementById("pemilik").value,
                nomor_hp: document.getElementById("nomorHp").value,
                barang: document.getElementById("barang").value,
                harga_modal: document.getElementById("hargaModal").value,
                harga_jual: document.getElementById("hargaJual").value,
                jenis: document.getElementById("jenis").value,
                terjual: document.getElementById("terjual").value,
                alamat: document.getElementById("alamat").value
            };

            if (!payload.nama_toko || !payload.barang) return alert("Nama toko & barang wajib diisi!");

            btn.disabled = true; 
            btn.innerHTML = `<i class="fas fa-spinner fa-spin"></i> Processing...`;

            try {
                // MENGGUNAKAN MODE NO-CORS AGAR TIDAK BLOKIR BROWSER
                await fetch(API_URL, {
                    method: "POST",
                    mode: "no-cors",
                    headers: { "Content-Type": "application/json" },
                    body: JSON.stringify(payload)
                });

                // Karena no-cors tidak mengembalikan respon JSON, kita beri jeda lalu refresh
                alert("Data Berhasil Dikirim ke Antrean Server!");
                resetForm();
                setTimeout(loadData, 2000); // Tunggu 2 detik agar Google Sheet memproses data
            } catch (e) {
                alert("Gagal koneksi ke server");
            } finally {
                btn.disabled = false; 
                btn.innerText = "SIMPAN KE SERVER";
            }
        }

        function editData(d) {
            document.getElementById("rowIndex").value = d.row_index;
            document.getElementById("currentID").value = d["ID"];
            document.getElementById("namaToko").value = d["Nama Toko"] || "";
            document.getElementById("pemilik").value = d["Nama Penjual"] || d["pemilik"] || "";
            document.getElementById("nomorHp").value = d["nomor hp pemilik toko"] || d["nomor_hp"] || "";
            document.getElementById("barang").value = d["Barang dijual"] || "";
            document.getElementById("hargaModal").value = d["Harga modal"] || "";
            document.getElementById("hargaJual").value = d["Harga jual"] || "";
            document.getElementById("jenis").value = d["Jenis"] || "";
            document.getElementById("terjual").value = d["Terjual"] || "";
            document.getElementById("alamat").value = d["Alamat toko"] || d["alamat"] || "";
            
            window.scrollTo({ top: 0, behavior: "smooth" });
        }

        function resetForm() {
            document.getElementById("rowIndex").value = "";
            document.getElementById("currentID").value = "";
            document.querySelectorAll("input, textarea").forEach(e => e.value = "");
        }

        // Load data saat pertama kali dibuka
        loadData();
    </script>
</body>
</html>
