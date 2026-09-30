# Product Requirements Document (PRD) — Kebun Sawit

## 1. Executive Summary
> Ringkasan singkat: visi, cakupan, dan outcome utama.

**Overview**

**Product:** Sistem Pengelolaan Kebun Sawit Keluarga  
**Platform:** Progressive Web App (PWA) — mobile-first, desktop-enhanced  
**Deployment:** Cloudflare Pages (frontend) + Cloudflare Workers (API) + Cloudflare D1 (serverless SQLite database)  
**Hosting Cost:** Rp0/bulan (Cloudflare Free plan)

Sistem ini memungkinkan sebuah keluarga petani sawit kecil (~1 ha, 3 kebun) untuk mencatat panen dan perawatan kebun secara digital, menghitung hasil bersih secara otomatis, dan melihat laporan keuangan per kebun — tanpa biaya hosting bulanan dan tanpa PC rumah yang menyala 24/7.

**Problem Statement**

Keluarga petani sawit kecil saat ini mengandalkan spreadsheet atau pencatatan manual (melet) untuk mencatat hasil panen dan pengeluaran perawatan. Proses ini ribet, rawan kesalahan perhitungan, dan menghasilkan laporan yang sulit dianalisis. Mereka butuh solusi sederhana yang:

- Bisa diakses lewat HP (karena Ibu, user utama, hanya memiliki ponsel).
- Bekeras offline atau dengan koneksi internet yang sering putus.
- Menyimpan semua data secara lokal di browser hingga berhasil disinkronkan ke backend.
- Menghitung hasil bersih otomatis tanpa perlu perhitungan manual.
- Menyajikan laporan keuangan per kebun di desktop.
- Memungkinkan ekspor data ke CSV/JSON untuk backup portabel.
- Berjalan dengan biaya Rp0/bulan.

**Target Users**

- **Ibu (Primary User — Mobile Entry):** Non-teknis, akses utama melalui HP. Bertugas mencatat panen (berat, harga/kg, upah, biaya tambahan) dan biaya perawatan setiap hari.
- **Ayah / Keluarga (Secondary User — Desktop Review):** Menggunakan desktop untuk mereview laporan keuangan per kebun, membandingkan performa antar kebun, dan mengekspor data ke CSV untuk analisis lanjutan.

**Goals**

- Memungkinkan pencatatan satu entri panen di HP dalam < 30 detik tanpa error perhitungan.
- Hasil bersih tercatat 100% konsisten dengan perhitungan manual keluarga.
- Laporan keuangan per kebun dapat dilihat di desktop dalam < 10 detik.
- Ekspor CSV bulanan untuk kebun dengan ≤ 365 entri selesai dalam < 5 detik.
- Biaya hosting tetap Rp0/bulan untuk keluarga dengan 3 kebun dan 12 bulan data.
- Backup portabel tersedia lewat ekspor database JSON.

**Non-Goals**

- AI prediksi hasil panen.
- Pengenalan karakter (OCR) atau pemindaian dokumen.
- Integrasi GPS, IoT sensor, atau perangkat keras.
- Sistem ERP/akuntansi penuh.
- Manajemen peran/pengguna multi-akun (hanya 1 akun keluarga bersama untuk MVP).
- Otomatisasi perhitungan upah pemanen.

**Business Context**

- **User segment:** Keluarga petani sawit kecil di Indonesia, tipikal 0.5–1 ha dengan 2–3 kebun.
- **Revenue model:** Gratis (Rp0/bulan). Aplikasi sumber tertutup, tidak komersial.
- **Technical constraint:** Harus zero-cost hosting. Tidak boleh memerlukan PC rumah atau server sendiri.
- **Regulatory:** Data disimpan di Cloudflare D1 (serverless SQLite). Session cookie Secure + HttpOnly. Rate-limiting login (5 percobaan gagal → cooling period).
- **Success criteria:** Kesesuaian angka dengan perhitungan manual, zero hosting cost, dan kemampuan export backup portabel.

**Main User Flows**

**Flow 1 — Ibu: Catat Panen (Mobile PWA)**

1. Ibu membuka PWA di HP.
2. Memilih kebun dari daftar.
3. Memilih tanggal (default: hari ini).
4. Memasukkan berat (kg) + harga/kg (atau memilih dari histori).
5. Memasukkan upah pemanen + biaya tambahan.
6. Sistem otomatis menghitung hasil bersih = (berat × harga/kg) − upah − biaya tambahan.
7. Menyentuh tombol Save (sticky). Data disimpan ke localStorage / pending sync queue jika offline.
8. Saat kembali online, data disinkronkan otomatis ke D1 backend.

**Flow 2 — Ayah: Review Laporan (Desktop)**

1. Ayah login di desktop.
2. Membuka sidebar navigasi.
3. Membuka dashboard kebun.
4. Melihat ringkasan pendapatan, pengeluaran, dan hasil bersih per kebun.
5. Membandingkan 3 kebun secara side-by-side (fitur P1).
6. Melihat badge Δ harga dan Δ berat (fitur P1).
7. Mengekspor data ke CSV (bulanan) atau JSON (full backup).

**Flow 3 — Backup & Export**

1. Dari menu desktop, klik "Export CSV" → unduh file dengan kolom: tanggal, berat_kg, harga_per_kg, upah, biaya_tambahan, hasil_bersih.
2. Klik "Export JSON" → unduh database lengkap (tabel: panen, biaya, kebun, pekerja).
3. File dapat disimpan sebagai backup portabel di perangkat lokal.

## 2. MVP / P0 Scope

P0 (MVP) mencakup:
- Pencatatan panen / setoran di HP (mobile PWA)
- Berat panen (kg) input manual
- Harga jual per kg input manual
- Total penjualan (berat × harga/kg) dihitung otomatis
- Upah pemanen diinput manual (tidak ada perhitungan otomatis)
- Biaya tambahan (input manual)
- Hasil bersih = (berat × harga/kg) − upah − biaya tambahan (auto-hitung)
- Histori dasar harga per setoran (disimpan per entry)
- Histori dasar berat per setoran (disimpan per entry)
- Biaya perawatan: pemupukan, semprot/pembersihan, biaya lain (lainnya)
- Laporan / ringkasan per kebun (pendapatan, pengeluaran, hasil bersih)
- Akun keluarga sederhana (passcode/PIN, 1 shared account)
- Export ke CSV (bulanan per kebun) dan JSON (full database backup)
- Offline-first: form persist localStorage, queue sync saat online kembali
- Mobile-first UX (PWA, 48px touch target, decimal keypad, sticky save)
- Desktop UX (sidebar nav, per-kebun report)

## 3. P1 / Future Scope

P1 (fitur fase berikutnya) mencakup:
- Badge Δ harga (±RpX/kg vs setoran sebelumnya)
- Badge Δ berat (±X kg vs setoran sebelumnya)
- Garis waktu (line chart) tren harga dan berat per setoran
- Penjelasan sederhana: pendapatan turun/tambah karena [harga | berat | keduanya]
- Perbandingan 3 kebun secara side-by-side di dashboard

## 4. Functional Requirements

- **FR-001**: System MUST allow recording a harvest entry with weight (kg), price per kg, worker wage, and extra cost, and auto-calculate net result = (weight × price/kg) − wage − extra cost.
- **FR-002**: System MUST persist the harvest form state in browser local storage so data survives accidental tab close.
- **FR-003**: System MUST display harvest history and offer prior weight + price as defaults for the next entry.
- **FR-004**: System MUST preserve a history of sale price per kg for each harvest entry (P0/MVP).
- **FR-005**: System MUST preserve a history of harvested weight (kg) per harvest/setoran entry (P0/MVP).
- **FR-006**: System MUST allow recording maintenance expenses categorized as fertilizer (pemupukan), spraying (semprot), or other (lainnya).
- **FR-007**: System MUST display a summary report per plot showing total revenue, total expenses, and net result for the selected period.
- **FR-008**: System MUST support a single shared family account protected by a PIN/passcode, with session cookies marked Secure and HttpOnly.
- **FR-009**: System MUST rate-limit login attempts (5 failed attempts → cooling period).
- **FR-010**: System MUST export per-plot harvest and maintenance data to CSV.
- **FR-011**: System MUST export the full database to a portable JSON file.
- **FR-012**: System MUST queue offline form submissions and sync them to the backend on connectivity return.
- **FR-013**: System MUST support mobile form factors with 48px touch targets and decimal numeric keypad on numeric inputs.
- **FR-014**: System MUST support desktop layouts with sidebar navigation.
- **FR-015**: System MUST display Δ price badge (±RpX/kg vs prior entries) and Δ weight badge (±X kg) in the desktop dashboard (P1).
- **FR-016**: System MUST display a line chart of price and weight trends across harvest entries in the desktop dashboard (P1).
- **FR-017**: System MUST show a simple explanation of revenue change caused by price, weight, or both when displaying Δ badges (P1).

**Total FR:** 17 (FR-001 — FR-017)

## 5. Business Rules

- Hanya boleh ada 1 akun keluarga untuk MVP (shared account).
- Semua data untuk setiap entry disimpan sebagai raw record (bukan hanya total Rupiah).
- P0 menyimpan record mentah saja; fitur Δ-badge dan grafik tren baru tersedia di P1.
- Export periode bulanan = kalender bulan ini (user dapat menyesuaikan start/end tanggal).
- Rate-limiting login: 5 percobaan gagal → cooling period.
- Session cookie harus Secure + HttpOnly.
- Ekspor CSV/JSON hanya tersedia saat ada data; jika tidak ada, tombol dinonaktifkan atau toast "Tidak ada data untuk diekspor".
- Jika belum ada kebun yang dibuat, tampilkan empty state dengan prompt "Tambah Kebun".
- Jika plot tidak ada entry pada periode yang dipilih, laporan menampilkan nol dan pesan kosong.

## 6. Panen / Setoran Rules

- Setiap entry panen mensyimpan: tanggal, berat_kg, harga_per_kg, upah, biaya_tambahan, hasil_bersih, catatan.
- Hasil bersih dihitung otomatis: (berat_kg × harga_per_kg) − upah − biaya_tambahan.
- Histori harga per setoran disimpan per entry (FR-004).
- Histori berat per setoran disimpan per entry (FR-005).
- Nilai historis (harga/kg dan berat) dapat dipakai sebagai default untuk entry berikutnya pada plot yang sama (FR-003).
- Offline Save: entry disimpan ke localStorage / pending sync queue; otomatis sync ke backend saat online kembali.

## 7. Manual Worker Wage Rules

- Upah pemanen diinput manual oleh Ibu/pengguna pada form panen.
- Sistem TIDAK menghitung upah otomatis — tidak ada formula per kg, per hari, atau persentase.
- Upah disebutkan sebagai "otomatisasi upah" pada daftar Ditolak — tidak akan diimplementasikan.
- Worker dipilih dari daftar pekerja manual per keluarga (pekerja dikelola manual).

## 8. Price & Weight History

- Setiap setoran panen menyimpan berat_kg dan harga_per_kg secara terpisah (bukan hanya total Rupiah).
- Histori harga per kg disimpan per entry → FR-004 (P0).
- Histori berat per kg disimpan per entry → FR-005 (P0).
- Histori ini menjadi dasar default nilai untuk entry berikutnya (FR-003).
- Histori harga & berat digunakan untuk menghitung badge Δ harga, badge Δ berat, dan line chart tren di P1.

## 9. Data Requirements / Conceptual Entities

Entitas konseptual yang disetujui:

- **app_user** — satu akun keluarga; atribut: id, family_id, auth_pin_hash, created_at.
- **kebun** (plot) — atribut: id, family_id, nama, luas_m2, created_at.
- **panen** (harvest) — atribut: id, kebun_id, tanggal, berat_kg, harga_per_kg, upah, biaya_tambahan, hasil_bersih, catatan, created_at.
- **biaya** (expense) — atribut: id, kebun_id, tanggal, jenis (pemupukan/semprot/lainnya), deskripsi, nominal, created_at.
- **pekerja** (worker) — atribut: id, nama, family_id.
- **panen_pekerja** — junction table hubungkan panen → pekerja (many-to-many).

Semua entitas menyimpan data mentah per entry. Tidak ada agregasi yang hilang setelah entry disimpan.

## 10. Reports & Reporting Requirements

Laporan yang harus tersedia:

- **Daily/Monthly/Yearly Report per Kebun:**
  - Pendapatan total (jumlah berat × harga/kg per entry)
  - Total pengeluaran (biaya perawatan: pemupukan, semprot, lainnya)
  - Hasil bersih (pendapatan − pengeluaran)
  - Periode: daily, bulanan, tahunan (bisa dipilih rentang tanggal)

- **Histori Aktivitas:**
  - Daftar semua panen dan biaya per kebun, diurutkan tanggal
  - Dapat difilter per periode dan kategori biaya

- **Histori Harga & Berat:**
  - Riwayat harga_per_kg dan berat_kg setiap setoran panen
  - Tersedia di dashboard mobile dan desktop
  - Digunakan sebagai sumber badge Δ harga, Δ berat, dan line chart (P1)

- **Export Report:**
  - CSV: kolom tanggal, berat_kg, harga_per_kg, upah, biaya_tambahan, hasil_bersih
  - JSON: database lengkap (tabel panen, biaya, kebun, pekerja)

## 11. Authentication Requirements

- 1 akun keluarga bersama untuk MVP (single shared account)
- Credential dilindungi — PIN/passcode disimpan hashed di D1
- Session menggunakan cookie yang Secure + HttpOnly
- Logout yang jelas dan mudah diakses
- Rate-limiting login: 5 percobaan gagal → cooling period
- Tidak ada role/permission kompleks (multi-user/permission management ditolak)

## 12. Mobile UX Requirements

- Fokus utama: pencatatan panen / setoran — prioritas input aktivitas harian
- Desainer untuk pengguna non-teknis — langkah seminimal mungkin
- Semua input angka (berat, harga, upah, biaya) memakai numeric decimal keyboard
- Touch target minimal 48px untuk semua interaksi
- Error ditampilkan inline secara jelas dan mudah dipahami (bukan kode error)
- Form persisted otomatis ke localStorage; data tidak hilang jika tab ditutup
- Offline mode: data tersimpan lokal dan sync ke backend saat online kembali
- Desain mobile-first — bukan sekadar scaling-down desktop layout

## 13. Desktop UX Requirements

- Fokus utama: review laporan dan analisis data
- Sidebar navigasi: Dashboard, Kebun, Panen, Perawatan, Laporan
- Report per kebun: pendapatan, pengeluaran, hasil bersih
- Perbandingan hingga 3 kebun secara side-by-side (dashboard P1)
- Badge Δ harga dan Δ berat di dashboard (P1)
- Line chart tren harga & berat per setoran (P1)
- Penjelasan sederhana perubahan pendapatan: karena harga, berat, atau keduanya (P1)
- Export ke CSV (per kebun, per periode) dan JSON (full backup)

## 14. Edge Cases

- Pengguna membuka PWA tanpa ada kebun yang terdaftar → tampilkan empty state dengan prompt "Tambah Kebun".
- Device offline saat klik Save → entry disimpan ke localStorage pending queue; sync otomatis saat koneksi kembali.
- Device offline tanpa koneksi sebelumnya → simpan ke localStorage; queue sync diproses saat koneksi pertama tersedia.
- Plot tidak memiliki entry pada periode yang dipilih → laporan menampilkan nol dan pesan "tidak ada data".
- Export gagal karena tidak ada data → tombol Export dinonaktifkan atau toast "Tidak ada data untuk diekspor".

## 15. Assumptions

- Keluarga menggunakan 1 akun shared untuk MVP; sistem multi-user role management ditunda ke P1.
- Workers dipilih dari daftar pekerja manual per keluarga (daftar dikelola manual).
- Histori harga dan berat = setiap entry yang tersimpan (bukan rata-rata periode).
- Histori histori harga/berat dapat dipakai sebagai default untuk entry berikutnya pada plot yang sama.
- Periode export bulanan = kalender bulan ini (user dapat menyesuaikan start/end tanggal).
- P0 menyimpan raw record saja; fitur Δ-badge, line chart, dan perbandingan kebun tiba di P1.
- Upah pemanen selalu diinput manual — tidak ada formula otomatis.

## 16. Out of Scope

Fitur yang Secara Eksplisit DITOLAK / tidak akan dikembangkan:

- AI prediksi hasil panen (price/berat prediction)
- Pengenalan karakter (OCR) atau pemindaian dokumen
- Integrasi GPS / geolocation tracking
- IoT sensor (soil moisture, weather station, dll.)
- Sistem ERP / akuntansi kompleks
- Manajemen role & permission (multi-user, RBAC, dll.)
- Otomatisasi perhitungan upah pemanen (formula per kg / hari / persentase)
- PC rumah / server lokal yang harus menyala 24/7
- Integrasi akuntansi eksternal (QuickBooks, Xero, dll.)
- Fitur kolaborasi tim / multi-anggota keluarga login terpisah

## 17. Success Metrics

- **SC-001**: Families can record a harvest entry on mobile in under 30 seconds without calculation errors.
- **SC-002**: 100% of recorded harvest value + expenses reconcile to the same net-result figure the family calculates by hand.
- **SC-003**: Family can view a plot's month-to-date net result on desktop within 10 seconds.
- **SC-004**: Monthly CSV export for a plot with ≤ 365 entries completes in under 5 seconds.
- **SC-005**: Zero hosting cost is incurred for a family with 3 plots over a 12-month period.

## 18. Appendix

> Referensi tambahan bila diperlukan pada iterasi berikutnya.
