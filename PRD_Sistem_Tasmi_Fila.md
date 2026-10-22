# Product Requirements Document (PRD)
**Nama Produk:** Tasmi Fila 
**Platform:** Web Application
**Tech Stack:** Next.js (Frontend & API Routes), Supabase (Database, Auth, Storage)

---

## 1. Ringkasan Eksekutif
Aplikasi ini bertujuan untuk mendigitalisasi proses input nilai ujian Tasmi' hafalan Al-Qur'an, memonitoring progres siswa berdasarkan masing-masing Guru, dan mengotomatisasi pembuatan sertifikat. Sistem memisahkan hak akses antara Guru (input data terpandu saat sesi dibuka) dan Admin (kendali penuh atas pengguna, verifikasi data, pencetakan, laporan ekspor Excel, dan pengaturan sistem).

---

## 2. Autentikasi & Manajemen Pengguna (User Management)
Sistem menggunakan autentikasi tertutup berbasis *Role-Based Access Control* (RBAC) tanpa fitur pendaftaran mandiri (register) untuk publik.

*   **Akun Default Admin:** 
    *   Sistem diinisialisasi dengan satu akun super-admin secara *default*.
    *   **Email/Username:** `finsani52@gmail.com`
    *   **Password:** `123456fiblm`
    *   *Catatan:* Admin memiliki fitur **Ubah Password** di dalam *dashboard* untuk mengganti kata sandi *default* ini.
*   **Pembuatan Akun Guru:** Admin memiliki kewenangan penuh untuk membuatkan akun bagi Guru Al-Qur'an (input Nama, Username, Role, dan Password awal). Admin juga dapat mereset *password* guru, serta menghapus atau menonaktifkan akun guru.
*   **Penyimpanan Sesi (Session):** Menggunakan basis *Cookies* bawaan sistem (Supabase Auth) agar status *login* (baik Admin maupun Guru) tetap tersimpan persisten meskipun browser ditutup.

---

## 3. Fitur Utama & Spesifikasi

### 3.1. Kontrol Sistem Admin (Toggle Akses Input)
*   **Toggle Aktivitas Input Guru:** Admin memiliki tombol sakelar (*toggle button*) di *dashboard* untuk membuka atau menutup masa pengisian data Tasmi'. 
    *   **Jika ON:** Seluruh akun Guru dapat mengakses form dan mengirim data siswa.
    *   **Jika OFF:** Halaman form input bagi Guru akan terkunci (menampilkan pesan bahwa sesi pengisian telah ditutup oleh Admin), sehingga Guru tidak bisa lagi menambah atau mengedit data.

### 3.2. Form Input Data Tasmi' (Akses Guru)
Formulir ini hanya dapat diakses saat Admin mengaktifkan *Toggle Input*.
*   **Nama Lengkap:** Input teks.
*   **Tanggal & Waktu Tasmi':** Menggunakan *Date Picker* (Masehi) dan input Waktu.
*   **Kelas:** *Dropdown* bersumber dari *Master Data Kelas*.
*   **Capaian Juz & Status Mutqin:**
    *   Pemilihan Juz (misal: Juz 30).
    *   Terdapat *checkbox/flagging* **"Mutqin"**. Jika aktif, Guru dapat menginput lebih dari 1 juz sekaligus (misal: Juz 28, 29, 30).
*   **Input Nilai (Fashohah, Tajwid, Kelancaran, Kompilasi):**
    *   **Validasi Range:** Sistem menolak input angka di luar rentang **86 hingga 95**.
    *   **Otomatisasi Predikat:** Sistem mengonversi nilai menjadi keterangan:
        *   **86 - 89:** Jayyid
        *   **90 - 92:** Jayyid Jiddan
        *   **93 - 95:** Mumtaz

### 3.3. Verifikasi Data, Laporan, & Cetak (Akses Admin)
*   **Monitoring Data per Guru:** Admin dapat memantau tabel data siswa yang masuk, dilengkapi informasi **Nama Guru** yang mengirimkan data tersebut.
*   **Verifikasi & Status:** Admin dapat me-review data dan mengubah status siswa menjadi **"Telah Tertasmi"**. 
*   **Cetak Sertifikat:** Tombol *Generate Certificate* hanya akan aktif pada baris data siswa yang sudah berstatus "Telah Tertasmi".
*   **Export to Excel (Laporan Rapih):** Admin dapat mengunduh data tasmi' ke dalam format *Excel (.xlsx)*. Sistem harus men-generate file Excel dengan susunan kolom tabel yang rapi dan terstruktur, mencakup: 
    *   *(No, Nama Siswa, Kelas, Tanggal Tasmi', Waktu, Juz, Mutqin (Ya/Tidak), Nilai Fashohah, Nilai Tajwid, Nilai Kelancaran, Nilai Kompilasi, Predikat Rata-rata, Status Verifikasi, Nama Guru Penginput).*

### 3.4. Generator Sertifikat & Logika Visual
Sistem secara presisi mencetak sertifikat merujuk pada referensi visual dengan logika otomatis:
*   **Konversi Tanggal Hijriyah Otomatis:** Tanggal Masehi yang diinput Guru akan dirender pada sertifikat bersanding dengan otomatisasi konversi ke penanggalan Hijriyah (Contoh: *Bandar Lampung, 20 Juni 2026 / 05 Muharram 1447 Hijriyah*).
*   **Dinamika Warna Bingkai (Frame Color):** Warna dominan sertifikat berubah otomatis menyesuaikan Kelas siswa:
    *   Kelas 1: `#0e7b28`
    *   Kelas 2: `#2c389a`
    *   Kelas 3: `#b63333`
    *   Kelas 4: `#1fa6a9`
    *   Kelas 5: `#ff6c03`
    *   Kelas 6: `#8815a8`
    *   **Mutqin Override:** Jika siswa mendapatkan status Mutqin (di kelas manapun), warna frame WAJIB berubah menjadi hijau zaitun/olive dengan kode `#49501f`.
*   **Kustomisasi Cetak Admin:** Admin dapat mengunggah logo sekolah (tersimpan di *Cloud Storage*), merubah teks statis (Nomor Surat, Nama & NIK Kepala Sekolah / Koordinator), serta mengatur ukuran kertas (*A4/F4*) sebelum mencetak PDF.

---

## 4. Rekomendasi Arsitektur Teknis (Developer Guidelines)

**Frontend (Next.js):**
*   **Next.js App Router** dipadukan dengan **Tailwind CSS**.
*   **React Hook Form & Zod** untuk validasi strict (form angka 86-95).
*   **html2canvas + jsPDF** (atau *Server-side headless browser* API) untuk *render* cetak sertifikat beresolusi tinggi dengan *absolute positioning*.
*   **moment-hijri** atau **hijri-converter** untuk *helper* konversi penanggalan.
*   **xlsx** atau **exceljs** untuk *generate* file *export* laporan dengan penyesuaian lebar kolom otomatis (*auto-width*) agar rapi saat dibuka.

**Backend & Database (Supabase):**
*   **Supabase Admin API (Service Role Key):** Digunakan pada *API Routes* untuk *endpoint* pembuatan akun Guru oleh Admin.
*   **Supabase Storage:** *Bucket* khusus aset gambar logo dinamis.

**Struktur Tabel Database (Draft):**
1.  **`users_profiles`** (Extensi Supabase Auth): `id`, `full_name`, `role` (admin/guru).
2.  **`classes`**: `id`, `name`, `color_code`.
3.  **`system_settings`**: `id`, `is_input_session_active` (boolean toggle), `school_logo_url`, `headmaster_name`, `quran_coordinator_name`.
4.  **`tasmi_records`**: 
    *   `id`, `student_name`, `class_id`, `tasmi_date`, `tasmi_time`, `juz_details`, `is_mutqin`
    *   `score_fashohah`, `score_tajwid`, `score_kelancaran`, `score_kompilasi`
    *   `teacher_id` (FK ke profiles)
    *   `status` (varchar: 'pending', 'telah_tertasmi')