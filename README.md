# WargaKepo - Riwayat Teman Mabar

Userscript yang mencatat **siapa saja yang pernah satu tim denganmu** di match squad **Absolute**, lalu menampilkan labelnya di halaman sesi. Sekali lihat kamu tahu pemain di lobi itu **sudah pernah** main bareng atau **belum**.

Semuanya berjalan otomatis di latar belakang. Tidak perlu membuka profil atau halaman match satu per satu.

> Skrip tidak resmi, tidak berafiliasi dengan WargaKepo.

---

## Fitur

- **Pencatatan otomatis.** Membaca riwayat match dan roster tiap match lewat API situs, tanpa iframe dan tanpa membaca tampilan halaman.
- **Label di halaman sesi (`/slots/<id>`).** Di samping username pemain muncul:
  - 🟢 **Sudah 3x · 2 hari lalu** — sudah pernah satu tim, beserta berapa kali dan kapan terakhir.
  - 🟠 **Belum** — belum pernah satu tim.
- **Panel riwayat** lewat tombol berlogo WK:
  - Ringkasan: jumlah orang, jumlah pertandingan unik, dan waktu terakhir dipindai.
  - Cari nama, urutkan **Terbaru / Terbanyak / A-Z**.
  - Tiap baris punya badge jumlah main (biru 1x, hijau 2-3x, ungu 4x ke atas) dan tautan ke profil.
- **Username terdeteksi otomatis** dari tautan "Profil" di menu situs.
- **Tombol bisa digeser** ke mana saja dan posisinya diingat.
- **Penghitungan bersih.** Satu pertandingan hanya dihitung sekali walau diunggah lebih dari sekali, dan username ganda dalam satu roster dibuang.

---

## Pemasangan

1. Pasang pengelola userscript di browser:
   - **Android:** Tampermonkey atau Violentmonkey di browser yang mendukung ekstensi (mis. Kiwi Browser, Firefox).
   - **Desktop:** Tampermonkey atau Violentmonkey.
2. Buat script baru, hapus isi bawaannya, lalu tempel seluruh isi `wk-riwayat.user.js`. Simpan.
3. Buka **wargakepo.id** dalam keadaan sudah login, lalu tunggu sekitar 5-10 detik.

Kalau sebelumnya sudah memakai versi lama, cukup ganti isinya. Data yang sudah tersimpan tetap dipakai.

---

## Cara pakai

1. Buka halaman wargakepo.id mana saja. Pemindaian pertama berjalan sendiri dan membaca **seluruh** riwayat match Absolute-mu.
2. Ketuk tombol logo WK untuk membuka panel. Bar status di panel menampilkan progres.
3. Buka halaman sesi untuk melihat label **Sudah / Belum** di samping pemain.

### Panel

| Elemen | Fungsi |
|---|---|
| Tombol logo WK | Ketuk untuk membuka panel. Geser untuk memindahkan. Cincin berputar berarti sedang memindai. |
| ⟳ (header) | Pindai ulang seluruh riwayat sekarang. |
| Kolom cari | Saring daftar berdasarkan nama. |
| Terbaru / Terbanyak / A-Z | Urutan daftar. Pilihan diingat. |
| Pengaturan | Ubah username manual dan hapus semua data. |

---

## Cara kerja

Skrip memanggil dua alamat API milik situs dengan sesi login-mu sendiri (same-origin):

1. `/api/u/<username>/mabar-history?offset=N&pageSize=20` untuk daftar match. Disaring berdasarkan `squad.name` = Absolute.
2. `/api/mabar-submissions/<id>` untuk roster satu match. Daftar teman diambil dari `member_username`.

Aturan pemindaian:

- **Pemindaian pertama** (atau setelah tombol ⟳) membaca seluruh riwayat, maksimal 1000 match.
- **Pemindaian rutin** tiap 10 menit hanya membaca halaman terbaru dan berhenti begitu semua match di halaman itu sudah tercatat.
- Kalau ada match yang gagal diambil, percobaan ulang dilakukan 1 menit kemudian, dan pemindaian berikutnya membaca seluruh riwayat sampai lengkap.
- Match yang sudah tercatat tidak diambil lagi.

Aturan penghitungan:

- Hanya **teman satu tim** yang dihitung, bukan lawan.
- Pemain yang akunnya belum tertaut di roster (tanpa username) tidak terhitung.
- Dihitung per `battle_id`, jadi satu pertandingan yang diunggah dua kali tetap terhitung satu.

---

## Pengaturan di kode

Ubah di bagian atas `wk-riwayat.user.js`:

| Konstanta | Bawaan | Arti |
|---|---|---|
| `SQUAD` | `'Absolute'` | Hanya match dari squad ini yang dicatat. |
| `PAGE_SIZE` | `20` | Jumlah match per permintaan riwayat. |
| `MAX_PAGES` | `50` | Batas pengaman halaman yang dibaca (50 x 20 = 1000 match). |
| `RESCAN_MS` | 10 menit | Jeda pemindaian rutin. |
| `RETRY_MS` | 1 menit | Jeda percobaan ulang setelah gagal. |

---

## Data dan privasi

- Semua data tersimpan **lokal** di pengelola userscript browser-mu (`GM_setValue`). Tidak ada yang dikirim ke server lain.
- Satu-satunya permintaan jaringan adalah ke `wargakepo.id` memakai sesi login-mu sendiri.
- Tombol **Hapus semua** di Pengaturan menghapus seluruh riwayat yang tersimpan.

---

## Pemecahan masalah

Pesan di bar status panel menunjukkan penyebabnya:

| Pesan | Artinya | Yang dilakukan |
|---|---|---|
| `Username belum terdeteksi` | Tautan "Profil" di menu tidak ditemukan. | Buka Pengaturan, tap **Ubah username**, isi username tanpa `@`. |
| `Error: HTTP 401` atau `403` | Belum login atau sesi habis. | Login ulang di wargakepo.id. |
| `Error: HTTP 404` | Alamat API berubah atau username salah. | Periksa username. Kalau benar, situs mungkin mengubah API-nya. |
| `... tapi tidak ada match Absolute` | Riwayat terbaca, tapi tidak ada match squad itu. | Periksa nilai `SQUAD` di kode. |
| `... N gagal (roster kosong ...)` | Roster sebagian match tidak terbaca. | Akan dicoba ulang otomatis. Atau tap ⟳. |

Label tidak muncul di halaman sesi? Label hanya tampil di alamat `/slots/<angka>` dan saat daftar tim sudah termuat. Tunggu sebentar, atau muat ulang halaman.

---

## Keterbatasan

- Memakai API situs yang tidak terdokumentasi. Kalau situs mengubahnya, skrip perlu disesuaikan.
- Label di halaman sesi dicari lewat teks di layar ("Team (x/y)", "In team"). Perubahan tampilan situs bisa memengaruhinya.
- Hanya satu squad yang dicatat (`SQUAD`).

---

## Riwayat versi

- **4.2** — Tombol memakai logo WK.
- **4.1** — Tampilan baru: panel bottom sheet, urutan, badge, tombol bisa digeser, warna label "Belum" jadi oranye, perbaikan ketuk di HP.
- **4.0** — Beralih penuh ke API, semua match Absolute dicatat, perhitungan per `battle_id`, username ganda dibuang, username terdeteksi otomatis.
- **2.0** — Versi awal berbasis iframe dan pembacaan tampilan halaman.
