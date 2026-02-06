<!DOCTYPE html>
<html lang="id">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Monitoring Collection - BPR BKK Jateng</title>
    <link href="https://cdn.jsdelivr.net/npm/bootstrap@5.3.0/dist/css/bootstrap.min.css" rel="stylesheet">
    <link rel="stylesheet" href="https://cdn.jsdelivr.net/npm/bootstrap-icons@1.11.1/font/bootstrap-icons.css">
    <style>
        :root { --bkk-blue: #004a99; --bkk-gold: #ffcc00; }
        body { background-color: #f0f2f5; font-family: 'Segoe UI', sans-serif; }
        .header-bank { background: var(--bkk-blue); color: white; padding: 15px; border-bottom: 5px solid var(--bkk-gold); margin-bottom: 20px; display: flex; align-items: center; justify-content: center; gap: 15px; position: relative; }
        .logo-bank { width: 50px; background: white; padding: 5px; border-radius: 8px; }
        #loginOverlay { position: fixed; top: 0; left: 0; width: 100%; height: 100%; background: rgba(0,74,153,0.98); z-index: 9999; display: flex; align-items: center; justify-content: center; }
        .card { border: none; border-radius: 12px; box-shadow: 0 4px 12px rgba(0,0,0,0.05); }
        .sticky-table { max-height: 450px; overflow-y: auto; border: 1px solid #dee2e6; border-radius: 8px; }
        .table-fixed { table-layout: fixed; width: 100%; margin-bottom: 0; }
        .table-fixed th, .table-fixed td { overflow: hidden; text-overflow: ellipsis; white-space: nowrap; vertical-align: middle; }
        .amount-font { font-family: 'Courier New', monospace; font-weight: bold; }
        .alamat-truncate { font-size: 0.75rem; color: #666; white-space: normal; display: -webkit-box; -webkit-line-clamp: 2; -webkit-box-orient: vertical; overflow: hidden; }
        .img-table { width: 45px; height: 45px; object-fit: cover; border-radius: 6px; }
        #preview-foto { max-width: 100%; height: 200px; object-fit: cover; display: none; border-radius: 8px; margin-top: 10px; border: 2px solid #ddd; }
        .rekap-box { background: white; border-radius: 10px; padding: 15px; margin-bottom: 20px; border-left: 5px solid var(--bkk-gold); }
        .page-section { display: none; }
    </style>
</head>
<body>

<div id="loginOverlay">
    <div class="card p-4 text-center shadow-lg" style="max-width: 350px; width: 90%;">
        <img src="https://bankbanjarharjo.id/assets/upload/images/logo.png" width="70" class="mx-auto mb-3">
        <h4 class="fw-bold">Akses Kolektor</h4>
        <p class="small text-muted mb-3">Pilih nama dan masukkan PIN</p>
        
        <select id="userKolektor" class="form-select mb-2 text-center">
            <option value="" disabled selected>-- Pilih Nama --</option>
        </select>
        
        <input type="password" id="passKolektor" class="form-control mb-3 text-center" placeholder="PIN Akses">

        <button onclick="prosesLogin()" class="btn btn-primary w-100 fw-bold mb-3">MASUK</button>
        <hr>
        <button onclick="tarikMasterCloud()" id="btnSyncLogin" class="btn btn-sm btn-outline-success w-100">
            <i class="bi bi-cloud-arrow-down"></i> Sinkron Petugas & Data
        </button>
        <div id="syncStatusLogin" style="font-size: 10px;" class="mt-1 text-muted"></div>
    </div>
</div>

<div id="mainApp" style="display:none;">
    <div class="header-bank">
        <img src="https://bankbanjarharjo.id/assets/upload/images/logo.png" class="logo-bank">
        <div>
            <h5 class="fw-bold mb-0">BPR BKK JATENG</h5>
            <span id="labelPetugas" class="badge bg-warning text-dark">Petugas: -</span>
        </div>
        <button onclick="logout()" class="btn btn-sm btn-outline-light position-absolute top-0 end-0 m-2"><i class="bi bi-power"></i></button>
    </div>

    <div id="pageRencana" class="container-fluid page-section">
        <div class="card p-3 mb-4 border-success shadow-sm">
            <div class="d-flex justify-content-between align-items-center mb-3">
                <h6 class="fw-bold text-success mb-0"><i class="bi bi-calendar-check"></i> Rencana Kunjungan</h6>
                <button onclick="tarikMasterCloud()" id="btnTarikCloud" class="btn btn-sm btn-success fw-bold"><i class="bi bi-cloud-download"></i> TARIK DATA</button>
            </div>
            <div class="table-responsive sticky-table">
                <table class="table table-sm table-striped table-fixed">
                    <thead class="table-success small sticky-top">
                        <tr>
                            <th style="width: 45%">Nasabah</th>
                            <th style="width: 35%" class="text-end">Tagihan</th>
                            <th style="width: 20%" class="text-center">Aksi</th>
                        </tr>
                    </thead>
                    <tbody id="tbodyMasterExcel" class="small"></tbody>
                </table>
            </div>
        </div>
    </div>

    <div id="pageForm" class="container-fluid page-section">
        <div class="row">
            <div class="col-lg-5">
                <div class="card p-4 mb-4">
                    <div class="d-flex justify-content-between mb-3">
                        <h5 class="fw-bold text-primary mb-0">Laporan Lapangan</h5>
                        <button onclick="bukaHalaman('pageRencana')" class="btn btn-sm btn-secondary">Kembali</button>
                    </div>
                    <form id="collectionForm">
                        <input type="text" id="namaNasabah" class="form-control mb-2 fw-bold" readonly>
                        <textarea id="alamatNasabah" class="form-control mb-3 bg-light" rows="2" readonly></textarea>

                        <label class="small fw-bold">Bertemu Dengan:</label>
                        <select id="bertemuDengan" class="form-select mb-3" required>
                            <option value="" disabled selected>Pilih...</option>
                            <option value="Debitur">Debitur</option>
                            <option value="Pasangan">Pasangan</option>
                            <option value="Orang Tua">Orang Tua</option>
                            <option value="Anak">Anak</option>
                            <option value="Saudara">Saudara</option>
                            <option value="Tetangga">Tetangga</option>
                            <option value="Tidak Ada Orang">Tidak Ada Orang</option>
                        </select>

                        <label class="small fw-bold">Status Kunjungan:</label>
                        <select id="statusKunjungan" class="form-select mb-3" required onchange="cekStatusJanji()">
                            <option value="" disabled selected>Pilih...</option>
                            <option value="Janji Bayar">Janji Bayar</option>
                            <option value="Bayar Sebagian">Bayar Sebagian</option>
                            <option value="Titip Angsuran">Titip Angsuran</option>
                            <option value="Rumah Kosong">Rumah Kosong</option>
                        </select>

                        <div id="areaNominal" class="mb-3 p-3 border border-primary rounded shadow-sm bg-light" style="display: none;">
                            <div id="boxTglJanji" class="mb-2">
                                <label class="small fw-bold text-danger">TANGGAL JANJI BAYAR:</label>
                                <input type="date" id="tglJanjiBayar" class="form-control form-control-sm">
                            </div>
                            <div id="boxNominal">
                                <label class="small fw-bold text-primary">NOMINAL (Rp):</label>
                                <input type="text" id="nominalDisplay" class="form-control fw-bold text-primary" placeholder="Rp 0" onkeyup="formatRupiah(this)">
                                <input type="hidden" id="nominalAsli">
                            </div>
                        </div>

                        <label class="small fw-bold">Detail Usaha & Masalah:</label>
                        <select id="usahaDebitur" class="form-select mb-2" required>
                            <option value="" disabled selected>Usaha Debitur ...</option>
                            <option value="Perdagangan">Perdagangan</option>
                            <option value="Pertanian dan Perkebunan">Pertanian dan Perkebunan</option>
                            <option value="Peternakan">Peternakan</option>
                            <option value="Jasa">Jasa</option>
                            <option value="Industri dan Produksi">Industri dan Produksi</option>
                            <option value="Properti dan Kontruksi">Properti dan Kontruksi</option>
                            <option value="Kuliner dan F&B">Kuliner dan F&B</option>
                            <option value="Transportasi dan Logistik">Transportasi dan Logistik</option>
                            <option value="Teknologi dan Kreatif">Teknologi dan Kreatif</option>
                            <option value="Keuangan dan Lainnya">Keuangan dan Lainnya</option>
                            <option value="Sosial dan Lainnya">Sosial dan Lainnya</option>
                            <option value="Perikanan dan Kelautan">Perikanan dan Kelautan</option>
                            <option value="Pariwisata dan Hiburan">Pariwisata dan Hiburan</option>
                            <option value="Pendidikan dan kesehatan">Pendidikan dan kesehatan</option>
                            <option value="Energi dan Lingkungan">Energi dan Lingkungan</option>
                        </select>

                        <select id="penyebab" class="form-select mb-2" required>
                            <option value="" disabled selected>Penyebab Tidak Bayar ...</option>
                            <option value="Karakter/Itikad Bayar Lemah">Karakter/Itikad Bayar Lemah</option>
                            <option value="Penggunaan Dana Tidak Sesuai">Penggunaan Dana Tidak Sesuai</option>
                            <option value="Dana Dipakai Oranglain">Dana Dipakai Oranglain</option>
                            <option value="Omset Menurun/Usaha Sepi">Omset Menurun/Usaha Sepi</option>
                            <option value="Manajemen Usaha Lemah">Manajemen Usaha Lemah</option>
                            <option value="Banyak Hutang">Banyak Hutang</option>
                            <option value="Sakit">Sakit</option>
                            <option value="Meninggal">Meninggal</option>
                            <option value="Perceraian">Perceraian</option>
                            <option value="Konflik Keluarga">Konflik Keluarga</option>
                            <option value="Modal Untuk Konsumsi Pribadi">Modal Untuk Konsumsi Pribadi</option>
                            <option value="Monitoring dan Penagihan Tidak Optimal">Monitoring dan Penagihan Tidak Optimal</option>
                            <option value="Piutang Tidak Terbayar">Piutang Tidak Terbayar</option>
                            <option value="Fraud Internal">Fraud Internal</option>
                            <option value="Penipuan Rekan Bisnis">Penipuan Rekan Bisnis</option>
                            <option value="SKIP">SKIP</option>
                            <option value="Bencana">Bencana</option>
                            <option value="Gagal Panen">Gagal Panen</option>
                            <option value="Persaingan Usaha">Persaingan Usaha</option>
                            <option value="Usaha Tutup">Usaha Tutup</option>
                        </select>

                        <select id="kendala" class="form-select mb-3" required>
                            <option value="" disabled selected>Kendala Penyelesaian ...</option>
                            <option value="Debitur Tidak Kooperatif">Debitur Tidak Kooperatif</option>
                            <option value="Nomor HP Tidak Aktif">Nomor HP Tidak Aktif</option>
                            <option value="Debitur Pindah Alamat">Debitur Pindah Alamat Tanpa Pemberitahuan</option>
                            <option value="Janji Palsu">Debitur Hanya Memberikan Janji Palsu</option>
                            <option value="Omset Menurun">Omset/Pendapatan Debitur Menurun Signifikan</option>
                            <option value="Bangkrut">Usaha Tutup/Bangkrut</option>
                            <option value="PHK">Debitur Mengalami PHK</option>
                            <option value="Sakit Berat">Debitur Sakit Berat/Meninggal</option>
                            <option value="Sengketa Agunan">Agunan Dalam Sengketa/Belum Balik Nama</option>
                        </select>  

                        <input type="file" id="fotoKunjungan" class="form-control form-control-sm mb-2" accept="image/*" capture="camera" required>
                        <img id="preview-foto">
                        
                        <input type="hidden" id="lokasiGps">
                        <button type="submit" id="btnSimpan" class="btn btn-primary btn-lg w-100 fw-bold">SIMPAN & KIRIM</button>
                    </form>
                </div>
            </div>

            <div class="col-lg-7">
                <div class="rekap-box d-flex justify-content-around text-center shadow-sm">
                    <div><small>Laporan Hari Ini</small><h4 id="countTotal" class="fw-bold">0</h4></div>
                    <div class="text-success"><small>Ada Setoran</small><h4 id="countBayar" class="fw-bold">0</h4></div>
                </div>
                <div class="card p-3">
                    <div class="table-responsive">
                        <table class="table table-sm small">
                            <thead class="table-dark"><tr><th>Foto</th><th>Detail</th><th>Status</th><th>Aksi</th></tr></thead>
                            <tbody id="tbodyLaporan"></tbody>
                        </table>
                    </div>
                </div>
            </div>
        </div>
    </div>
</div>

<script>
    // --- KONFIGURASI ---
    const SCRIPT_URL = "https://script.google.com/macros/s/AKfycbxQy2XEQf67GiA8JVnzdiE32CNl2t2gMqSYkoO4Oq0_BLKAjYjWVHESAWDqM6d--mDz/exec"; 

    let dataLaporan = JSON.parse(localStorage.getItem('laporan_bkk')) || [];
    let petugasAktif = localStorage.getItem('petugas_aktif') || "";
    let daftarPetugas = JSON.parse(localStorage.getItem('daftar_petugas')) || [];
    let base64Foto = "";

    window.onload = () => {
        updateDropdownUI();
        if(petugasAktif) loginSukses();
        getGPS();
    };

    // --- FUNGSI LOGIN & MASTER ---
    function updateDropdownUI() {
        const sel = document.getElementById('userKolektor');
        sel.innerHTML = '<option value="" disabled selected>-- Pilih Nama --</option>';
        daftarPetugas.forEach(p => {
            let opt = document.createElement('option');
            opt.value = p.nama; opt.textContent = p.nama;
            sel.appendChild(opt);
        });
    }

    function prosesLogin() {
        const nama = document.getElementById('userKolektor').value;
        const pin = document.getElementById('passKolektor').value;
        const user = daftarPetugas.find(u => u.nama === nama);

        if(user && user.pin === pin) {
            localStorage.setItem('petugas_aktif', nama);
            petugasAktif = nama;
            loginSukses();
        } else {
            alert("Login Gagal! PIN salah atau belum pilih nama.");
        }
    }

    function loginSukses() {
        document.getElementById('loginOverlay').style.display = 'none';
        document.getElementById('mainApp').style.display = 'block';
        document.getElementById('labelPetugas').innerText = "Petugas: " + petugasAktif;
        bukaHalaman('pageRencana');
    }

    async function tarikMasterCloud() {
        const btnL = document.getElementById('btnSyncLogin');
        const btnM = document.getElementById('btnTarikCloud');
        if(btnL) btnL.innerText = "Syncing...";
        if(btnM) btnM.innerText = "Loading...";

        try {
            const res = await fetch(SCRIPT_URL);
            const data = await res.json();
            if(data.nasabah && data.petugas) {
                localStorage.setItem('rencana_kunjungan', JSON.stringify(data.nasabah));
                localStorage.setItem('daftar_petugas', JSON.stringify(data.petugas));
                daftarPetugas = data.petugas;
                updateDropdownUI();
                tampilkanMasterExcel();
                alert("Data Berhasil Sinkron!");
            }
        } catch(e) { alert("Gagal koneksi ke server!"); }
        finally {
            if(btnL) btnL.innerText = "Sinkron Petugas & Data";
            if(btnM) btnM.innerText = "TARIK DATA";
        }
    }

    // --- FUNGSI HALAMAN & FORM ---
    function bukaHalaman(id) {
        document.querySelectorAll('.page-section').forEach(p => p.style.display = 'none');
        document.getElementById(id).style.display = 'block';
        if(id === 'pageRencana') tampilkanMasterExcel();
        if(id === 'pageForm') tampilkanData();
    }

    function tampilkanMasterExcel() {
        const tbody = document.getElementById('tbodyMasterExcel');
        const data = JSON.parse(localStorage.getItem('rencana_kunjungan')) || [];
        tbody.innerHTML = "";
        data.forEach(item => {
            const row = tbody.insertRow();
            row.innerHTML = `
                <td><div class="fw-bold">${item.nama}</div><div class="alamat-truncate">${item.alamat}</div></td>
                <td class="text-end text-danger amount-font">${Number(item.tagihan).toLocaleString('id-ID')}</td>
                <td class="text-center"><button onclick="pilihKeForm('${item.nama}', '${item.alamat}')" class="btn btn-sm btn-primary">PILIH</button></td>
            `;
        });
    }

    function pilihKeForm(nama, alamat) {
        document.getElementById('namaNasabah').value = nama;
        document.getElementById('alamatNasabah').value = alamat;
        bukaHalaman('pageForm');
    }

    function formatRupiah(el) {
        let val = el.value.replace(/\D/g, "");
        document.getElementById('nominalAsli').value = val;
        el.value = val ? "Rp " + Number(val).toLocaleString('id-ID') : "";
    }

    function cekStatusJanji() {
        const s = document.getElementById('statusKunjungan').value;
        const area = document.getElementById('areaNominal');
        const boxTgl = document.getElementById('boxTglJanji');
        area.style.display = (s === "Rumah Kosong") ? "none" : "block";
        boxTgl.style.display = (s === "Janji Bayar") ? "block" : "none";
    }

    // --- FUNGSI FOTO & GPS ---
    function getGPS() {
    return new Promise((resolve, reject) => {
        if (!navigator.geolocation) {
            alert("GPS tidak didukung di browser ini.");
            resolve("-");
        }
        navigator.geolocation.getCurrentPosition(
            (p) => {
                const koordinat = p.coords.latitude + "," + p.coords.longitude;
                document.getElementById('lokasiGps').value = koordinat;
                resolve(koordinat);
            },
            (err) => {
                console.error("Gagal GPS:", err);
                resolve("-"); // Tetap lanjut meskipun GPS gagal
            },
            { 
                enableHighAccuracy: true, // Paksa akurasi tinggi (GPS Satelit)
                timeout: 10000,           // Tunggu maksimal 10 detik
                maximumAge: 0             // Jangan gunakan data lama
            }
        );
    });
}

    document.getElementById('fotoKunjungan').addEventListener('change', function() {
        const reader = new FileReader();
        reader.onload = e => {
            const img = new Image();
            img.src = e.target.result;
            img.onload = () => {
                const cvs = document.createElement('canvas');
                const ctx = cvs.getContext('2d');
                const maxWidth = 500; 
cvs.width = maxWidth; 
cvs.height = img.height * (maxWidth / img.width);

ctx.drawImage(img, 0, 0, cvs.width, cvs.height);
                ctx.drawImage(img, 0, 0, cvs.width, cvs.height);
                base64Foto = cvs.toDataURL('image/jpeg', 0.5);
                document.getElementById('preview-foto').src = base64Foto;
                document.getElementById('preview-foto').style.display = 'block';
            };
        };
        reader.readAsDataURL(this.files[0]);
    });

    // --- SIMPAN DATA ---
    document.getElementById('collectionForm').addEventListener('submit', async function(e) {
    e.preventDefault();
    const btn = document.getElementById('btnSimpan');
    btn.disabled = true; 
    btn.innerText = "MENGUNCI GPS..."; // Beri tahu user kalau sedang cari lokasi

    // Ambil GPS terbaru tepat saat tombol diklik
    const gpsTerbaru = await getGPS();

    btn.innerText = "MENGIRIM...";

    const data = {
        waktu: new Date().toLocaleString('id-ID'),
        petugas: petugasAktif,
        nama: document.getElementById('namaNasabah').value,
        alamat: document.getElementById('alamatNasabah').value,
        bertemu: document.getElementById('bertemuDengan').value,
        status: document.getElementById('statusKunjungan').value,
        janji_bayar: document.getElementById('tglJanjiBayar').value || "-",
        nominal: document.getElementById('nominalAsli').value || "0",
        usaha: document.getElementById('usahaDebitur').value,
        penyebab: document.getElementById('penyebab').value,
        kendala: document.getElementById('kendala').value,
        gps: gpsTerbaru, // Gunakan hasil koordinat terbaru
        foto: base64Foto
    };

    try {
        await fetch(SCRIPT_URL, {
            method: 'POST',
            mode: 'no-cors',
            body: JSON.stringify(data)
        });
        dataLaporan.unshift(data);
        localStorage.setItem('laporan_bkk', JSON.stringify(dataLaporan));
        alert("Terkirim ke Cloud!");
        this.reset();
        document.getElementById('preview-foto').style.display = 'none';
        base64Foto = "";
        bukaHalaman('pageRencana');
    } catch(e) { 
        alert("Gagal kirim ke Cloud, tapi tersimpan di HP."); 
    } finally { 
        btn.disabled = false; 
        btn.innerText = "SIMPAN & KIRIM"; 
    }
});

    function tampilkanData() {
        const table = document.getElementById('tbodyLaporan');
        table.innerHTML = ""; let cb = 0;
        dataLaporan.forEach((item, i) => {
            if(item.status.includes("Bayar") || item.status.includes("Titip")) cb++;
            const row = table.insertRow();
            row.innerHTML = `
                <td><img src="${item.foto}" class="img-table"></td>
                <td><b>${item.nama}</b><br><small>${item.waktu}</small></td>
                <td><span class="badge bg-primary">${item.status}</span></td>
                <td><button onclick="hapusData(${i})" class="btn btn-sm btn-link text-danger"><i class="bi bi-trash"></i></button></td>
            `;
        });
        document.getElementById('countTotal').innerText = dataLaporan.length;
        document.getElementById('countBayar').innerText = cb;
    }

    function hapusData(i) {
        if(confirm("Hapus riwayat di HP ini?")) {
            dataLaporan.splice(i, 1);
            localStorage.setItem('laporan_bkk', JSON.stringify(dataLaporan));
            tampilkanData();
        }
    }

    function logout() { localStorage.removeItem('petugas_aktif'); location.reload(); }
</script>
</body>
</html><!DOCTYPE html>
<html>
<head>

<title> Hello from Earth</title>
  <!--CSS cascading style sheets-->
<!-- <style> 
         body {
            background-color: black;
            color: white;
}
</style>   -->
</head>

<body>

          <h1> welcome to HTML 101:</h1>
          <h2> A beginners guide to coding</h2>
         <h3> Lists </h3>
  <ol> 
        <li> something in here </li>
        <li> make a second bullet point</li>
  </ol>

<ul>
       <li> point #1 </li>
        <li> point #2</li>
         <li> point #3</li>
</ul>
<!-- <script>
  alert("Hello world");
</script>
  src="test_javascripy.js"></script> -->
<a href="http://instagram.com" target="_blank">Click this link</a>
<a href="Untitled-2.html"> This goes to the second page</a>
<p>

   <b>Lorem ipsum dolor sit </b> amet consectetur adipisicing elit. 
   <i>Ab aperiam minima quia quas </i> laboriosam iure cupiditate labore, 
    assumenda laudantium sunt animi soluta minus, 
    <u>omnis aliquid sed voluptates nihil ea dolorum.</u>
</p>
<h3> This is an example of element nesting</h3>
<a href="Untitled-2.html">
<img 
src="C:\Users\Personal\Desktop\green_bird.jpg" 
width="500" 
height="100"

>
<br>
</a>

<p>
    <b> <i> <u>
    Lorem, ipsum dolor sit amet consectetur adipisicing elit. 
    Rem eveniet est delectus sequi beatae in? Reiciendis laudantium, 
    asperiores ratione sapiente facere tempore suscipit! Saepe autem animi quo,
     impedit maxime quidem.
     </u> </i> </b>
</p>
<hr>
<div>
    This has no special meaning.
</div>
</body>
</html>


</head>
