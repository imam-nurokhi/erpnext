# NexERP — Panduan Pengguna Modul Manufacturing

- **Versi dokumen:** 1.0 · **Tanggal:** 6 November 2026
- **Sistem:** NexERP (berbasis ERPNext / Frappe) — `https://nexerp.nexoratech.co/desk/manufacturing`
- **Sumber analisa:** kode repository `imam-nurokhi/erpnext` (branch rebrand NexERP) + tampilan halaman Manufacturing di situs live.
- **Ruang lingkup:** hanya modul **Manufacturing** (termasuk keterkaitan langsung ke Stock, Subcontracting, dan Quality).

---

## Daftar Isi

1. Pendahuluan
2. Peran Pengguna & Hak Akses
3. Mengenal Halaman Manufacturing
4. Konsep & Alur Proses Produksi
5. Setup Awal (Langkah demi Langkah)
6. Bill of Materials (BOM)
7. Eksekusi Produksi: Work Order, Job Card, Stock Entry
8. Perencanaan Material: Forecast, MPS, MRP, Production Plan
9. Subcontracting (Maklon)
10. Laporan & Dashboard
11. Contoh Skenario End-to-End
12. Troubleshooting & Pesan Error Umum
13. Status Saat Ini, Issue & Gap
14. Next Action Development (Roadmap)
15. Lampiran

---

## 1. Pendahuluan

Modul Manufacturing di NexERP dipakai untuk mengelola seluruh siklus produksi:

- **Definisi produk**: struktur bahan (BOM), operasi, mesin/workstation, routing.
- **Perencanaan**: peramalan permintaan, Master Production Schedule (MPS), Material Requirements Planning (MRP), Production Plan.
- **Eksekusi**: Work Order, Job Card (pencatatan waktu kerja per operasi), mutasi stok produksi.
- **Pengendalian**: downtime mesin, quality inspection, process loss, biaya produksi, laporan.

Semua transaksi produksi otomatis terhubung ke **Stock** (mutasi barang) dan **Accounting** (nilai persediaan & biaya), sehingga Harga Pokok Produksi terbentuk tanpa input manual tambahan.

### 1.1 Istilah Penting

| Istilah | Arti |
|---|---|
| **Raw Material (RM)** | Bahan baku yang dikonsumsi dalam produksi |
| **Sub-assembly / Semi-Finished Good** | Barang setengah jadi yang punya BOM sendiri |
| **Finished Good (FG)** | Barang jadi hasil produksi |
| **BOM** | Bill of Materials — resep/daftar bahan & operasi untuk membuat 1 batch barang |
| **Work Order (WO)** | Perintah produksi untuk sejumlah qty barang tertentu |
| **Job Card** | Kartu kerja per operasi per WO; tempat mencatat waktu & qty selesai |
| **Workstation** | Mesin / lini / meja kerja tempat operasi dilakukan |
| **WIP Warehouse** | Gudang Work-in-Progress (bahan yang sedang diproses) |
| **Backflush** | Pemotongan stok bahan baku otomatis saat barang jadi dicatat |
| **Process Loss** | Susut proses (qty yang hilang selama produksi) |
| **Secondary Items** | Scrap / co-product / by-product yang muncul dari produksi |
| **MPS / MRP** | Master Production Schedule / Material Requirements Planning |

---

## 2. Peran Pengguna & Hak Akses

| Role | Hak utama |
|---|---|
| **Manufacturing Manager** | Membuat & submit BOM, mengubah Manufacturing Settings, approve perencanaan |
| **Manufacturing User** | Membuat Work Order, Production Plan, Job Card, BOM |
| **Stock User** | Membuat Work Order & Stock Entry (transfer/manufacture) |
| **Operator** | Akses halaman Visual Plant Floor (start/complete job) |
| **System Manager** | Semua, termasuk Job Card |

> **Rekomendasi:** pisahkan role *Planner* (Production Plan/MPS), *Supervisor* (Work Order), *Operator* (Job Card saja) dan *Gudang* (Stock Entry). Lihat Gap G-11.

---

## 3. Mengenal Halaman Manufacturing

URL: `/desk/manufacturing`

### 3.1 Sidebar Kiri

| Menu | Isi |
|---|---|
| **Home** | Halaman workspace (chart, kartu angka, shortcut) |
| **Dashboard** | Dashboard Manufacturing (chart analitik lebih lengkap) |
| **BOM** | Daftar Bill of Materials |
| **Work Order** | Daftar perintah produksi |
| **Job Card** | Daftar kartu kerja operasi |
| **Stock Entry** | Mutasi stok (transfer bahan, manufacture) |
| **Material Planning ▸** | Item Lead Time, Production Plan, Forecasting, Master Production Schedule, Sales Forecast, Production Planning Report |
| **Tools ▸** | BOM Creator, BOM Update Tool, BOM Comparison Tool, Downtime Entry |
| **Reports ▸** | 10 laporan produksi (lihat Bab 10) |
| **Setup ▸** | Item, Warehouse, Operation, Workstation, Workstation Type, Plant Floor, Routing |
| **Settings** | Manufacturing Settings |
| **Getting Started** | Panel onboarding 7 langkah |

### 3.2 Area Utama (Home)

| Widget | Sumber data | Arti |
|---|---|---|
| **Produced Quantity** (chart garis) | Work Order submit, field `produced_qty` | Total qty diproduksi per periode (filter: Last Year, Monthly) |
| **Open Work Orders** | Work Order status *Not Started* | WO yang sudah disubmit tapi belum mulai |
| **WIP Work Orders** | Work Order status *In Process* | WO yang sedang berjalan |
| **Manufactured Items Value** | Stock Entry purpose *Manufacture*, bulan ini | Nilai (Rp) barang jadi yang masuk gudang bulan ini |
| **Kartu Link** | Bill of Materials, Production, Subcontracting, Reports, Tools, Settings | Akses cepat ke dokumen terkait |

### 3.3 Getting Started (Onboarding)

| # | Langkah | Aksi |
|---|---|---|
| 1 | Create Raw Material | Buat Item bahan baku |
| 2 | Create Finished Good | Buat Item barang jadi |
| 3 | Create Operation | Buat master Operation |
| 4 | Create Bill of Materials | Buat BOM |
| 5 | Create Work Order | Buat Work Order |
| 6 | View Work Order Summary | Buka laporan Work Order Summary |
| 7 | Review Manufacturing Settings | Cek & simpan Manufacturing Settings |

Progres saat ini di situs live: **0/7 (0%)**.

---

## 4. Konsep & Alur Proses Produksi

### 4.1 Alur Besar

```
 PERENCANAAN                                   EKSEKUSI
 ───────────                                   ────────
 Sales Forecast ──► Master Production ──► MRP Report
                    Schedule (MPS)            │
                                              ▼
 Sales Order / ──────────────────────► Production Plan ──► Material Request ──► Purchase Order
 Material Request                             │
                                              ▼
 BOM + Routing ─────────────────────────► Work Order
 (Operation, Workstation)                     │
                                              ├──► Job Card (per operasi: waktu, qty, QC)
                                              │
                                              ├──► Stock Entry: Material Transfer for Manufacture
                                              │        (Gudang Bahan ──► WIP)
                                              │
                                              └──► Stock Entry: Manufacture
                                                       (WIP dikurangi, Gudang FG bertambah,
                                                        scrap ke Scrap Warehouse)
```

### 4.2 Siklus Status Work Order

```
Draft ──submit──► Not Started ──transfer/JC mulai──► In Process ──produced = qty──► Completed
                      │                                 │
                      └──► Stopped ◄────────────────────┘  (bisa Resume)
                      └──► Closed (ditutup manual, sisa tidak diproduksi)
                      └──► Cancelled
```

Status tambahan: **Stock Reserved / Stock Partially Reserved** (jika reservasi stok aktif).

### 4.3 Siklus Status Job Card

`Open → Work In Progress → (On Hold) → Material Transferred → Completed`, lalu Submitted / Cancelled.

### 4.4 Dampak Akuntansi (ringkas)

| Transaksi | Debit | Kredit |
|---|---|---|
| Transfer bahan ke WIP | Persediaan WIP | Persediaan Bahan Baku |
| Manufacture | Persediaan Barang Jadi | Persediaan WIP / Bahan Baku + Biaya Operasi (dibebankan) |

Biaya operasi berasal dari *hour rate* Workstation × waktu operasi (BOM atau Job Card aktual).

---

## 5. Setup Awal (Langkah demi Langkah)

Urutan yang **wajib** diikuti sebelum transaksi pertama:

```
1. Manufacturing Settings ─► 2. Warehouse ─► 3. Item ─► 4. Workstation Type / Workstation / Plant Floor
                                                    ─► 5. Operation / Routing ─► 6. BOM ─► 7. Work Order uji coba
```

### 5.1 Manufacturing Settings

Menu: **Settings** (sidebar paling bawah). Hanya Manufacturing Manager/System Manager.

#### Tab *BOM and Production*

| Pengaturan | Default | Rekomendasi awal | Penjelasan |
|---|---|---|---|
| Backflush Raw Materials Based On | BOM | **BOM** | *BOM*: bahan dipotong sesuai resep. *Material Transferred for Manufacture*: dipotong sesuai yang benar-benar ditransfer ke WIP |
| Allow Continuous Material Consumption | Off | Off | Jika On, bahan bisa dikonsumsi bertahap sebelum barang jadi dicatat |
| Update BOM Cost Automatically | Off | **On** | Scheduler memperbarui biaya BOM dari harga terbaru |
| Make Serial No / Batch from Work Order | Off | Sesuai kebutuhan | Serial/batch barang jadi dibuat otomatis saat WO disubmit |
| Add Corrective Operation Cost in FG Valuation | Off | On (jika ada rework) | Biaya rework ikut menambah nilai barang jadi |
| Validate Components and Quantities Per BOM | Off | **On** (disiplin) | Stock Entry ditolak jika tidak sesuai BOM |
| Transfer Extra Raw Materials to WIP (%) | 0 | 5–10% | Toleransi transfer bahan lebih |
| Allow Editing of Items and Quantities in Work Order | Off | Off | Izinkan ubah bahan di WO |
| Set Operating Cost / Secondary Items From Sub-assemblies | Off | Sesuai kebutuhan | Biaya operasi & scrap sub-assembly ikut ke FG tanpa Job Card |
| Overproduction % for Sales Order / Work Order | 0 | 0–5% | Toleransi produksi melebihi pesanan |

#### Tab *Job Card and Capacity Planning*

| Pengaturan | Default | Penjelasan |
|---|---|---|
| Disable Capacity Planning | Off | Matikan jika tidak ingin penjadwalan otomatis per workstation |
| Capacity Planning For (Days) | 30 | Horizon penjadwalan operasi |
| Time Between Operations (Mins) | 10 | Jeda antar operasi |
| Allow Overtime | Off | Boleh menjadwalkan di luar jam kerja workstation |
| Allow Production on Holidays | Off | Boleh menjadwalkan di hari libur (Holiday List) |
| Allow Excess Material Transfer | Off | Transfer bahan di Job Card setelah kebutuhan terpenuhi |
| Get Raw Materials Cost from Consumption Entry | Off | Biaya RM dari entri konsumsi |
| Enforce Time Logs | Off | Time log Job Card wajib From/To Time |

Default Warehouse (WIP, FG, Scrap) juga diatur di sini dan dipakai otomatis oleh Work Order baru.

### 5.2 Warehouse

Buat minimal gudang berikut (per Company):

| Gudang | Fungsi |
|---|---|
| **Stores / Gudang Bahan Baku** | Sumber bahan baku |
| **Work In Progress** | Bahan yang sedang diproses |
| **Finished Goods** | Hasil produksi |
| **Scrap** (opsional) | Sisa produksi / scrap |

### 5.3 Item

| Jenis | Setting penting |
|---|---|
| Bahan Baku | `Maintain Stock` = Ya, Item Group (mis. *Raw Material*), UOM, Valuation Rate awal, Default Warehouse |
| Sub-assembly | `Maintain Stock` = Ya, nanti diberi BOM |
| Barang Jadi | `Maintain Stock` = Ya, Default BOM (otomatis terisi saat BOM default disubmit) |
| Serial / Batch | Centang *Has Serial No* / *Has Batch No* jika perlu ketelusuran |
| Subkontrak | Centang *Is Sub Contracted Item* untuk bahan yang dikerjakan vendor |

### 5.4 Workstation Type, Workstation, Plant Floor

1. **Workstation Type** — kelompok mesin (mis. *Mesin CNC*, *Meja Assembly*). Isi *Hour Rate* & komponen biaya (listrik, sewa, operator).
2. **Workstation** — mesin spesifik:
   - *Workstation Type*, *Plant Floor*, *Warehouse*
   - *Production Capacity* (jumlah job paralel)
   - *Working Hours* (jam kerja per hari) + *Holiday List*
   - *Workstation Costs* → menghasilkan **Hour Rate** (Rp/jam)
   - *Status*: Production / Off / Idle / Problem / Maintenance / Setup
3. **Plant Floor** — denah lantai produksi. Di form Plant Floor tampil **Visual Plant Floor**: status semua workstation, Job Card aktif, stok di gudang lantai, serta tombol Start/Complete job dan buat Stock Entry.

### 5.5 Operation & Routing

- **Operation**: nama proses (Potong, Rakit, Cat, QC).
  - *Default Workstation*, *Sub Operations* (langkah detail + waktu)
  - *Batch Size* + *Create Job Card based on Batch Size*
  - *Is Corrective Operation* (untuk rework)
  - *Quality Inspection Template*
- **Routing**: urutan operasi standar (Operation + Workstation + waktu menit + batch size). Satu Routing bisa dipakai banyak BOM; biaya operasi dihitung otomatis.

---

## 6. Bill of Materials (BOM)

### 6.1 Membuat BOM (cara standar)

1. Menu **BOM → + Add BOM**.
2. Isi **Item** (barang jadi/sub-assembly) dan **Quantity** (qty hasil per BOM, mis. 1 atau 100).
3. Bagian **Raw Materials**: tambah item bahan, qty, UOM. Rate terisi otomatis sesuai *Rate Of Materials Based On*:
   - *Valuation Rate*, *Price List*, *Last Purchase Rate*, atau *Manual*.
4. (Opsional) Centang **With Operations** → pilih **Routing** atau isi tabel **Operations** (Operation, Workstation, Time in mins).
5. (Opsional) **Secondary Items** — scrap/co-product beserta % alokasi biaya (total alokasi FG + secondary harus 100%).
6. (Opsional) **Process Loss %**, **Inspection Required**, **Allow Alternative Item**, **Project**.
7. **Save → Submit**. Centang **Is Active** dan **Is Default** untuk BOM utama.

**Aturan sistem:**
- Hanya **satu BOM default** per item; BOM default otomatis tercatat di Item.
- BOM yang dipakai BOM lain **tidak bisa dinonaktifkan/dicancel** (gunakan BOM Update Tool → Replace).
- Mengubah BOM yang sudah submit = **Amend/Copy** menjadi versi baru (penomoran `BOM-ITEM-001`, `-002`, …).
- Biaya total = Raw Material Cost + Operating Cost (mendukung multi currency).

### 6.2 Multi-level BOM

Jika bahan adalah sub-assembly yang punya BOM, field **BOM No** di baris bahan terisi otomatis. Tabel **Exploded Items** menampilkan kebutuhan bahan baku sampai level terbawah. Di Work Order, centang **Use Multi-Level BOM** untuk langsung memakai bahan level terbawah (tanpa WO sub-assembly).

**Phantom BOM**: BOM yang tidak diproduksi tersendiri — bahannya langsung "diledakkan" ke BOM induk.

### 6.3 BOM Creator (membuat BOM bertingkat sekaligus)

Menu **Tools → BOM Creator**:
1. Isi Finished Good, Qty, Company, *Rate Based On*.
2. Pada tampilan pohon, tambah bahan; klik bahan → **Add Sub Assembly** untuk membuat level di bawahnya.
3. **Submit** → sistem membuat semua BOM (dari level terbawah) di background. Status: *In Progress → Completed* (atau *Failed* dengan Error Log).

### 6.4 BOM Update Tool

Menu **Tools → BOM Update Tool**:
- **Replace BOM**: pilih *Current BOM* dan *New BOM* → ganti di semua BOM induk. Dijalankan sebagai background job.
- **Update latest price in all BOMs**: hitung ulang biaya seluruh BOM level demi level.
- Riwayat & status job tercatat di **BOM Update Log** (Queued, In Progress, Completed, Failed).

### 6.5 BOM Comparison Tool

Menu **Tools → BOM Comparison Tool**: pilih dua BOM → tampil perbedaan item, qty, rate, dan operasi. Berguna untuk audit perubahan resep.

---

## 7. Eksekusi Produksi

### 7.1 Work Order — Langkah demi Langkah

1. **Work Order → + Add** (atau dari Sales Order / Production Plan / Material Request).
2. Isi:
   - **Item To Manufacture**, **BOM No** (default terisi otomatis), **Qty To Manufacture**
   - **Source / WIP / Target (FG) / Scrap Warehouse**
   - **Planned Start Date**, **Expected Delivery Date**
   - Opsi: *Use Multi-Level BOM*, *Skip Material Transfer to WIP*, *Reserve Stock*, *Project*, *Sales Order*
3. Tabel **Required Items** (bahan) dan **Operations** terisi dari BOM.
4. **Submit** → status **Not Started**. Jika ada operasi & capacity planning aktif, jadwal operasi dihitung dan **Job Card** dapat dibuat.
5. **Start** → membuat Stock Entry *Material Transfer for Manufacture* (Stores → WIP).
   *(Dilewati bila "Skip Material Transfer" aktif.)*
6. Kerjakan operasi melalui **Job Card** (7.2).
7. **Finish** → membuat Stock Entry *Manufacture*: bahan dikonsumsi (backflush), barang jadi masuk FG Warehouse, scrap masuk Scrap Warehouse.
8. Saat Produced Qty = Qty, status **Completed**.

**Tombol/aksi lain di Work Order:**

| Aksi | Fungsi |
|---|---|
| Create Job Card | Membuat Job Card per operasi |
| Create Pick List | Daftar ambil bahan untuk gudang |
| Stop / Re-open | Menghentikan sementara WO |
| Close | Menutup WO walaupun belum selesai (sisa tidak diproduksi) |
| Disassemble | Membongkar barang jadi kembali menjadi komponen |
| Return Components | Mengembalikan sisa bahan dari WIP ke gudang |
| Reserve / Unreserve Stock | Reservasi stok bahan untuk WO |
| Create BOM | Membuat BOM dari WO (jika belum ada) |

### 7.2 Job Card

Dibuat per **operasi** dari Work Order.

1. Buka Job Card → **Start Job** (pilih Employee). Timer berjalan.
2. **Pause / Resume** jika ada jeda (status *On Hold*).
3. **Complete Job** → isi *Completed Qty* (dan *Process Loss Qty* jika ada).
4. Jika operasi butuh bahan khusus, gunakan **Material Request** atau **Material Transfer** dari Job Card.
5. Jika *Inspection Required*, buat **Quality Inspection** sebelum submit.
6. **Submit** → completed qty & waktu aktual diperbarui ke Work Order; biaya operasi aktual = hour rate × waktu aktual.

Fitur lanjutan:
- **Sub Operations** — checklist langkah dalam satu operasi.
- **Corrective Job Card** — untuk rework (operasi berflag *Is Corrective Operation*).
- **Subcontracting PO per operasi** — operasi tertentu dikerjakan vendor.
- **Semi-Finished Goods tracking** — hasil per operasi dicatat sebagai barang setengah jadi.
- Validasi **overlap jadwal workstation** (workstation tidak bisa dipakai dua job melebihi kapasitas).

### 7.3 Stock Entry untuk Produksi

| Purpose | Kapan dipakai |
|---|---|
| Material Transfer for Manufacture | Memindahkan bahan dari gudang ke WIP |
| Material Consumption for Manufacture | Konsumsi bahan bertahap (jika Continuous Consumption aktif) |
| Manufacture | Mencatat barang jadi & konsumsi bahan |
| Disassemble | Membongkar barang jadi |
| Material Transfer (return) | Mengembalikan sisa bahan |

### 7.4 Downtime Entry

Menu **Tools → Downtime Entry**: catat Workstation, From/To Time, Operator, **Stop Reason** (Excessive Machine set up time, Unplanned machine maintenance, On-machine press checks, Machine operator errors, Machine malfunction, Electricity down, Other), Remarks. Data dipakai laporan **Downtime Analysis**.

---

## 8. Perencanaan Material

### 8.1 Item Lead Time

Data waktu per item untuk MPS/MRP: Manufacturing Time (menit/unit), No. of Shift, Shift Time, No. of Workstations, Daily Yield, Capacity per Day, Purchase Time, Buffer Time.

### 8.2 Sales Forecast

1. **Sales Forecast → + Add**: Company, Posting Date, Parent Warehouse, From Date, Frequency, Demand Number.
2. Pilih item → **Generate Demand** (exponential smoothing dari histori Sales Order) atau isi manual.
3. Submit → status *Planned*; tombol **Create MPS** → status *MPS Generated*.

### 8.3 Master Production Schedule (MPS)

1. Isi Company, Parent Warehouse, From/To Date.
2. Ambil demand dari **Sales Forecast**, **Sales Orders** (Get Sales Orders), **Material Requests**.
3. **Get Actual Demand** → sistem menghitung kebutuhan per item per tanggal dengan lead time kumulatif.
4. Submit → MRP dihitung (background); lihat **Material Requirements Planning Report**.

### 8.4 Production Plan

1. **Production Plan → + Add**: *Get Items From* = Sales Order atau Material Request.
2. Filter (tanggal, customer, project, item) → **Get Sales Orders** / **Get Material Requests** → **Get Finished Goods for Manufacture**.
3. **Get Sub Assembly Items** (opsi: combine, skip yang stoknya tersedia, gudang sub-assembly).
4. **Get Raw Materials for Purchase** (opsi: include safety stock, consider minimum order qty, ignore existing ordered qty, include non-stock / subcontracted items, warehouse tertentu).
5. **Submit** →
   - **Make Material Request** → bahan yang kurang (Purchase/Transfer)
   - **Make Work Order** → WO untuk FG & sub-assembly (atau Subcontracting PO)
   - **Reserve Stock** (opsional)
6. Pantau status: *Not Started → Material Requested → In Process → Completed / Closed*. Laporan: **Production Plan Summary**.

Tombol **Download Materials Required** → file Excel kebutuhan bahan.

### 8.5 Blanket Order

Kontrak jangka panjang (dengan customer/supplier) berisi item, qty & rate yang disepakati dalam periode tertentu; Sales/Purchase Order dibuat dari Blanket Order dan qty-nya dipantau.

---

## 9. Subcontracting (Maklon)

Alur:
```
Subcontracting BOM ─► Purchase Order (service) ─► Subcontracting Order ─► Kirim bahan (Stock Entry Send to Subcontractor)
                                                                     ─► Subcontracting Receipt (terima barang jadi)
```
Laporan: **Subcontract Order Summary**, **Subcontracted Raw Materials To Be Transferred**, **Subcontracted Item To Be Received**.
Subcontracting per operasi juga bisa dibuat dari **Job Card**.

---

## 10. Laporan & Dashboard

### 10.1 Laporan di Menu

| Laporan | Tujuan | Filter utama |
|---|---|---|
| Production Planning Report | Cek kecukupan bahan untuk WO / SO terbuka | Company, Based On, Warehouse |
| Material Requirements Planning | Kebutuhan bahan hasil MPS | MPS, Company |
| Work Order Summary | Rekap WO: status, umur, qty | Company, tanggal, status |
| Production Analytics | Tren WO per periode | Company, range |
| Job Card Summary | Rekap Job Card & waktu | Company, tanggal, WO |
| Downtime Analysis | Downtime per alasan & workstation | Tanggal, workstation |
| Quality Inspection Summary | Rekap QC lulus/gagal | Tanggal |
| BOM Search | Cari BOM yang memakai item tertentu | Item |
| BOM Stock Report | Kecukupan stok untuk sebuah BOM | BOM, Warehouse, Qty (**lihat Issue I-01**) |
| BOM Operations Time | Waktu operasi per BOM | Item, BOM, Workstation |
| Work Order Consumed Materials | Bahan aktual terpakai per WO | Company, WO |

### 10.2 Laporan Tersedia tetapi Tidak Ada di Menu

Akses lewat Search (⌘K / Ctrl+K) dengan nama laporan:

| Laporan | Tujuan |
|---|---|
| BOM Explorer | Pohon BOM lengkap multi level |
| BOM Stock Analysis | Analisa stok terhadap BOM |
| BOM Variance Report | Selisih bahan aktual vs BOM |
| Process Loss Report | Susut proses per WO |
| Cost of Poor Quality Report | Biaya rework / corrective operation |
| Exponential Smoothing Forecasting | Peramalan permintaan |
| Completed / Open Work Orders, Work Orders in Progress | Daftar WO per status |
| Issued Items Against Work Order | Bahan yang dikeluarkan per WO |
| Work Order Stock Report | Stok vs kebutuhan WO |
| Production Plan Summary | Rekap progres Production Plan |

### 10.3 Dashboard Manufacturing

Chart: Produced Quantity, Completed Operation, Work Order Analysis, Work Order Qty Analysis, Pending Work Order (umur), Job Card Analysis, Quality Inspection Analysis, Last Month Downtime Analysis.
Number Card: Open WO, WIP WO, Manufactured Items Value, Monthly Total/Completed WO, Monthly Quality Inspection, Ongoing Job Card.

---

## 11. Contoh Skenario End-to-End

**Produk:** *Meja Kayu Jati 120cm* (FG). **Sub-assembly:** *Rangka Meja*. **Bahan:** Papan Jati, Kaki Meja, Sekrup, Cat Melamine.

| Langkah | Dokumen | Isi |
|---|---|---|
| 1 | Item | Buat 4 bahan baku, 1 sub-assembly, 1 FG |
| 2 | Workstation | *Mesin Potong* (Rp 50.000/jam), *Meja Rakit* (Rp 30.000/jam), *Booth Cat* (Rp 40.000/jam) |
| 3 | Operation | Potong, Rakit, Finishing Cat |
| 4 | Routing | *Routing Meja*: Potong 20' → Rakit 30' → Cat 25' |
| 5 | BOM Rangka | 2 Papan Jati + 4 Kaki + 16 Sekrup, operasi Potong & Rakit |
| 6 | BOM Meja | 1 Rangka + 1 Papan Jati (top) + 0,5 L Cat, operasi Cat; Secondary: serbuk kayu (scrap) |
| 7 | Sales Order | Customer pesan 10 meja |
| 8 | Production Plan | Dari SO → Get Sub Assembly → Get Raw Materials → Make MR (bahan kurang) + Make WO |
| 9 | Purchase | MR → PO → Purchase Receipt bahan |
| 10 | Work Order Rangka | Start (transfer) → Job Card Potong & Rakit → Finish |
| 11 | Work Order Meja | Start → Job Card Cat → Quality Inspection → Finish |
| 12 | Delivery | Delivery Note & Sales Invoice dari SO |
| 13 | Analisa | Work Order Summary, Job Card Summary, BOM Variance, Process Loss |

---

## 12. Troubleshooting & Pesan Error Umum

| Pesan | Penyebab | Solusi |
|---|---|---|
| *Work-in-Progress Warehouse is required before Submit* | WIP Warehouse kosong | Isi di WO atau set default di Manufacturing Settings |
| *Target Warehouse is required before Submit* | FG Warehouse kosong | Isi Target Warehouse |
| *Quantity to Manufacture must be greater than 0* | Qty 0 | Isi qty |
| *Work Order cannot be raised against a Item Template* | Item adalah template varian | Pilih item varian |
| *BOM {0} must be active / must be submitted* | BOM draft/nonaktif | Submit & aktifkan BOM |
| *BOM {0} does not belong to Item {1}* | Salah pilih BOM | Pilih BOM milik item tsb |
| *Cannot deactivate or cancel BOM as it is linked with other BOMs* | BOM dipakai BOM induk | Pakai BOM Update Tool → Replace |
| *Operations cannot be left blank* | With Operations dicentang tanpa operasi | Isi operasi / uncheck |
| *Operation Time must be greater than 0* | Waktu operasi 0 | Isi Time in mins |
| *Cost allocation between finished goods and secondary items should equal 100%* | Alokasi biaya tidak 100% | Sesuaikan % |
| *Completed Qty cannot be greater than 'Qty to Manufacture'* | Over produksi | Naikkan Overproduction % atau koreksi qty |
| *Stopped Work Order cannot be cancelled, Unstop it first* | WO status Stopped | Re-open lalu cancel |
| *Closed Work Order can not be stopped or Re-opened* | WO sudah Closed | Buat WO baru |
| *You can't make any changes to Job Card since Work Order is closed* | WO closed | — |
| *Row {0}: From time must be less than to time* | Time log salah | Perbaiki jam |
| *Capacity Planning Error...* | Workstation tanpa jam kerja / waktu 0 | Isi Working Hours, atau Disable Capacity Planning |
| Stok tidak cukup saat Manufacture | Bahan belum ditransfer / stok kosong | Cek Production Planning Report, transfer bahan dulu |

---

## 13. Status Saat Ini, Issue & Gap

### 13.1 Status Situs Live (6 Nov 2026)

| Aspek | Status |
|---|---|
| Onboarding | 0/7 langkah — belum ada master data produksi |
| Transaksi | Open WO = 0, WIP WO = 0, Manufactured Value = Rp 0 |
| Mata uang | Rupiah (Rp) sudah aktif |
| Branding | Kode sudah NexERP (`app_title = "NexERP"`), tetapi header sidebar situs live masih menampilkan **"ERPNext"** |

### 13.2 Issue (bug / konfigurasi salah)

| ID | Severity | Issue | Bukti | Dampak |
|---|---|---|---|---|
| **I-01** | Tinggi | Link **BOM Stock Report** di menu Reports & kartu Reports kemungkinan rusak; di kode hanya ada report **BOM Stock Analysis** | Sidebar & workspace JSON menautkan "BOM Stock Report"; yang ada hanya folder `report/bom_stock_analysis` | User klik → "Not Found"/error (perlu konfirmasi di live) |
| **I-02** | Sedang | Header sidebar situs live masih "ERPNext" padahal kode sudah rebrand | Screenshot vs `hooks.py` | Inkonsistensi branding; kemungkinan belum `bench migrate`/`clear-cache`/build di server |
| **I-03** | Rendah | Duplikasi menu: *Production Planning Report* muncul di Material Planning dan Reports; *BOM* di sidebar & kartu Subcontracting | workspace_sidebar JSON | Navigasi membingungkan |
| **I-04** | Rendah | Chart *Last Month Downtime Analysis* & *Quality Inspection Analysis* menyimpan filter tanggal statis (2019/2020) di fixture | Fixture `dashboard_chart` | Chart bisa kosong bila tidak di-refresh oleh `dashboard_fixtures.py` |
| **I-05** | Rendah | Terminologi terjemahan Bahasa Indonesia belum diverifikasi untuk istilah manufaktur (mix EN/ID di UI) | Screenshot UI masih EN | Adopsi user lapangan lebih lambat |

### 13.3 Gap (fitur / proses yang belum ada atau belum terkonfigurasi)

| ID | Area | Gap | Rekomendasi |
|---|---|---|---|
| **G-01** | Master data | Belum ada Item, Warehouse produksi, Workstation, Operation, BOM | Jalankan setup Bab 5; template import (Data Import) |
| **G-02** | Settings | Manufacturing Settings belum di-review (default gudang, backflush, capacity planning) | Workshop keputusan kebijakan (lihat 5.1) |
| **G-03** | Navigasi | Laporan bernilai (BOM Explorer, Process Loss, Cost of Poor Quality, BOM Variance, Production Plan Summary) tidak ada di menu | Tambah ke sidebar Reports |
| **G-04** | Navigasi | **Visual Plant Floor** hanya bisa diakses dari form Plant Floor; Blanket Order & Quality Inspection tidak ada di menu Manufacturing | Tambah link / shortcut |
| **G-05** | Home | Workspace tidak punya **Shortcut** (shortcuts = kosong); hanya 1 chart & 3 number card | Tambah shortcut: New Work Order, Job Card terbuka, Production Plan, Plant Floor; tambah card *Ongoing Job Card* & *Monthly Completed WO* |
| **G-06** | Dokumentasi | Belum ada SOP/manual Bahasa Indonesia & materi training per role | Dokumen ini + video/slide per role |
| **G-07** | Lokal Indonesia | Belum ada format cetak (Print Format) Bahasa Indonesia untuk Work Order, Job Card, Surat Perintah Kerja, Bukti Pengeluaran Bahan | Buat Print Format ID |
| **G-08** | Shop floor | Belum ada mode operator sederhana (tablet/barcode) di luar Visual Plant Floor | Evaluasi Visual Plant Floor + barcode Job Card; atau custom page operator |
| **G-09** | Kualitas | Belum ada Quality Inspection Template & parameter per item | Definisikan template QC per produk/operasi |
| **G-10** | Biaya | Belum ada Workstation Operating Component yang dipetakan ke akun GL (listrik, tenaga kerja, overhead) | Mapping COA biaya produksi (Indonesia) |
| **G-11** | Akses | Role belum dipisah (Planner / Supervisor / Operator / Gudang); Operator butuh akses Job Card | Buat Role Profile & User Permission per plant/gudang |
| **G-12** | Notifikasi | Belum ada notifikasi/Assignment Rule (WO terlambat, material kurang, downtime) | Notification & Assignment Rule |
| **G-13** | Testing | Belum ada UAT script & data dummy untuk skenario manufaktur NexERP | Buat UAT per skenario Bab 11 |
| **G-14** | Integrasi | Belum ada integrasi mesin/IoT untuk status workstation & downtime otomatis | Fase lanjutan (API Workstation status) |
| **G-15** | Kustomisasi | Tidak ada custom field/script khusus NexERP di modul Manufacturing (masih 100% standar) | Kumpulkan kebutuhan industri klien |

---

## 14. Next Action Development (Roadmap)

### Fase 0 — Quick Fix (1 minggu)

| # | Aksi | Terkait | Output |
|---|---|---|---|
| 0.1 | Verifikasi link BOM Stock Report di live; perbaiki link sidebar & workspace ke **BOM Stock Analysis** | I-01 | Patch JSON workspace & sidebar |
| 0.2 | Deploy rebrand: `bench build`, `bench migrate`, `bench clear-cache` di server | I-02 | Header "NexERP" |
| 0.3 | Rapikan duplikasi menu | I-03 | Sidebar bersih |
| 0.4 | Tambah laporan tersembunyi & Visual Plant Floor ke menu | G-03, G-04 | Sidebar Reports lengkap |
| 0.5 | Tambah shortcut & number card di Home | G-05 | Home lebih informatif |

### Fase 1 — Setup & Konfigurasi (2–3 minggu)

| # | Aksi | Terkait |
|---|---|---|
| 1.1 | Workshop kebijakan Manufacturing Settings + dokumentasi keputusan | G-02 |
| 1.2 | Template Data Import: Item, BOM, Workstation, Operation, Routing | G-01 |
| 1.3 | Struktur gudang produksi per Company/plant | G-01 |
| 1.4 | Mapping biaya Workstation → COA | G-10 |
| 1.5 | Role Profile & User Permission | G-11 |
| 1.6 | Quality Inspection Template | G-09 |
| 1.7 | Selesaikan 7 langkah onboarding dengan data pilot | — |

### Fase 2 — Lokalisasi & Adopsi (2–4 minggu)

| # | Aksi | Terkait |
|---|---|---|
| 2.1 | Review & lengkapi terjemahan Bahasa Indonesia istilah manufaktur (`erpnext/locale/id.po`) | I-05 |
| 2.2 | Print Format ID: SPK/Work Order, Job Card, Bon Pengeluaran Bahan, Laporan Hasil Produksi | G-07 |
| 2.3 | Notifikasi & Assignment Rule | G-12 |
| 2.4 | Training per role + video singkat; distribusi manual ini | G-06 |
| 2.5 | UAT dengan skenario Bab 11, sign-off | G-13 |

### Fase 3 — Peningkatan (backlog)

| # | Aksi | Terkait |
|---|---|---|
| 3.1 | Halaman operator (tablet, scan barcode Job Card, start/stop) | G-08 |
| 3.2 | Dashboard KPI: OEE, yield, on-time WO, biaya per unit | — |
| 3.3 | Integrasi mesin/IoT untuk status & downtime otomatis | G-14 |
| 3.4 | Fix chart fixture tanggal statis | I-04 |
| 3.5 | Kustomisasi per industri klien (food: batch & expiry; garment: varian ukuran/warna; furniture: cutting list) | G-15 |
| 3.6 | Automated test untuk kustomisasi NexERP | G-13 |

### 14.1 Prioritas Ringkas

| Prioritas | Item |
|---|---|
| **P0** | I-01, I-02, G-01, G-02 |
| **P1** | G-03, G-04, G-05, G-10, G-11, G-09 |
| **P2** | I-03, I-05, G-06, G-07, G-12, G-13 |
| **P3** | I-04, G-08, G-14, G-15 |

---

## 15. Lampiran

### 15.1 Daftar DocType Modul Manufacturing

| DocType | Jenis | Keterangan |
|---|---|---|
| BOM | Submittable | Resep produk |
| BOM Creator | Submittable | Pembuat BOM bertingkat |
| BOM Update Tool | Single | Replace BOM / update biaya |
| BOM Update Log | Submittable | Log job BOM update |
| Blanket Order | Submittable | Kontrak jangka panjang |
| Downtime Entry | Normal | Catatan downtime |
| Job Card | Submittable | Kartu kerja operasi |
| Manufacturing Settings | Single | Pengaturan modul |
| Master Production Schedule | Normal | MPS |
| Operation | Normal | Master operasi |
| Plant Floor | Normal | Lantai produksi + visual |
| Production Plan | Submittable | Rencana produksi |
| Routing | Normal | Urutan operasi |
| Sales Forecast | Submittable | Peramalan permintaan |
| Work Order | Submittable | Perintah produksi |
| Workstation | Normal | Mesin / lini |
| Workstation Type | Normal | Kelompok mesin |
| Workstation Operating Component | Normal | Komponen biaya workstation |
| Item Lead Time *(modul Stock)* | Normal | Data lead time untuk MPS/MRP |

Child table: BOM Item, BOM Operation, BOM Explosion Item, BOM Secondary Item, BOM Creator Item, BOM Update Batch, BOM Website Item/Operation, Job Card Item/Operation/Time Log/Scheduled Time/Secondary Item, Production Plan Item/Sales Order/Material Request/Sub Assembly Item/Item Reference/Material Request Warehouse, Material Request Plan Item, Master Production Schedule Item, Sales Forecast Item, Sub Operation, Work Order Item/Operation, Workstation Cost/Working Hour, Workstation Operating Component Account, Blanket Order Item.

### 15.2 Halaman (Page)

| Page | Fungsi |
|---|---|
| bom-comparison-tool | Bandingkan dua BOM |
| visual-plant-floor | Monitor workstation (dipakai di form Plant Floor) |

### 15.3 Lokasi File Konfigurasi (untuk developer)

| Komponen | Path |
|---|---|
| Workspace Home | `erpnext/manufacturing/workspace/manufacturing/manufacturing.json` |
| Sidebar | `erpnext/workspace_sidebar/manufacturing.json` |
| Onboarding | `erpnext/manufacturing/module_onboarding/`, `onboarding_step/` |
| Number Card / Chart | `erpnext/manufacturing/number_card/`, `dashboard_chart/`, `dashboard_fixtures.py` |
| DocType | `erpnext/manufacturing/doctype/` |
| Report | `erpnext/manufacturing/report/` |
| Terjemahan ID | `erpnext/locale/id.po` |

---
*Dokumen ini disusun dari analisa kode sumber dan tampilan situs live. Item bertanda "perlu konfirmasi" harus diverifikasi langsung di server produksi.*
