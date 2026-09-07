# Arsitektur & Cara Kerja Sistem GitHub Profile README

Dokumen ini menjelaskan secara menyeluruh bagaimana sistem profil GitHub (`Adikafazaa/Adikafazaa`) bekerja, mulai dari mekanisme core GitHub, komponen SVG dinamis, alasan error layanan eksternal dan solusinya, hingga panduan perawatan serta kustomisasi mandiri.

---

## 1. Konsep Dasar GitHub Profile README

GitHub memiliki fitur khusus bernama **Special Repository** (Personal Profile README):

* **Aturan Penamaan**: Nama repositori harus **persis sama** dengan username akun GitHub (`Adikafazaa/Adikafazaa`).
* **Visibilitas**: Wajib berstatus **Public**.
* **Entry Point**: File `README.md` yang berada di root direktori branch default (`main`) akan otomatis di-render oleh GitHub di halaman depan profil (`https://github.com/Adikafazaa`), tepat di atas daftar pinned repositories dan contribution graph.

```
github.com/Adikafazaa
       │
       └──> Mendeteksi repo: Adikafazaa/Adikafazaa (Public)
                 │
                 └──> Mengambil: /README.md
                           │
                           └──> Di-render langsung di beranda profil
```

---

## 2. Bedah Arsitektur Komponen Profil

Profil yang telah dibuat menggunakan arsitektur modular berbasis Markdown, HTML wrapper, dan dynamic SVG services:

```
README.md
├── [1] Header Section (Waving Banner + Typing Animation + Social Badges)
├── [2] About Me (YAML format + Direct bullet points)
├── [3] Tech Stack & Systems (Linux Distros, Languages, DevOps)
├── [4] Selected Projects (Markdown table showcase)
└── [5] Dynamic Statistics (GitHub Stats, Top Languages, Streak Stats)
```

### A. Waving Header Banner (`capsule-render`)
* **Endpoint**: `https://capsule-render.vercel.app/api`
* **Cara Kerja**: Serverless function yang me-render SVG animasi gelombang secara real-time.
* **Parameter Utama**:
  * `type=waving`: Pola animasi ombak fluid di latar belakang.
  * `color=gradient&customColorList=14,24,35`: Skema warna gradasi gelap (selaras dark mode).
  * `theme=tokyonight`: Preset warna palet Tokyo Night.
  * `text=...` & `desc=...`: Nama utama dan sub-teks deskripsi.

### B. Animated Typing SVG (`readme-typing-svg`)
* **Endpoint**: `https://readme-typing-svg.demolab.com`
* **Cara Kerja**: Memanfaatkan CSS animation `@keyframes` di dalam file SVG statis. Teks diketik karakter demi karakter lalu berganti secara berulang (looping) tanpa memerlukan JavaScript di browser klien.
* **Parameter Utama**:
  * `font=Fira+Code`: Font monospace bernuansa developer.
  * `lines=Line1;Line2;Line3`: Kalimat-kalimat yang akan diketik bergantian (dipisahkan tanda titik-koma `;`).
  * `pause=1200`: Waktu tunggu (dalam milidetik) sebelum kalimat berganti.
  * `color=61AFEF`: Warna teks (hex code biru cerah).

### C. Shield Badges (`shields.io`)
* **Endpoint**: `https://img.shields.io/badge/<LABEL>-<COLOR>?style=...&logo=...`
* **Struktur Badge**:
  * Menggunakan logo resmi dari [Simple Icons](https://simpleicons.org/).
  * `style=for-the-badge`: Format badge berukuran tebal dan tegas.
  * Parameter `logoColor=white` / `black` memastikan kontras ikon terhadap warna latar badge.
* **Kategori Terpasang**:
  * **Linux Distros**: Linux, Debian, Ubuntu, Pop!_OS, Linux Mint.
  * **Languages & DB**: Python, TypeScript, JavaScript, SQL, GNU Bash.
  * **DevOps & Tools**: Docker, Git, GitHub Actions, Postman, VS Code.

### D. Dynamic Stats Cards & Masalah 503 Vercel

Bagian statistik menggunakan kartu dinamis yang mengambil data commit, PR, stars, dan bahasa langsung dari GitHub GraphQL / REST API:

#### Mengapa Layanan Lama (`github-readme-stats.vercel.app`) Sempat Hilang/Broken?
1. Domain publik `github-readme-stats.vercel.app` dikunjungi puluhan juta pengguna GitHub setiap hari.
2. Serverless instance Vercel milik developernya mengalami limit kuota bandwidth bulanan, sehingga Vercel memutus deployment dengan error:
   ```http
   HTTP/1.1 503 Service Unavailable
   X-Vercel-Error: DEPLOYMENT_PAUSED
   ```
3. Akibatnya, browser menampilkan icon gambar rusak (*broken image*).

#### Solusi yang Diterapkan:
Kami mengalihkan endpoint ke **`github-stats-extended.vercel.app`**:
* Merupakan fork/successor aktif yang mengimplementasikan caching lebih baik dan memiliki deployment aktif.
* Menampilkan metrik:
  * **GitHub Stats**: Total Stars, Total Commits, Pull Requests, Issues, Contributed repositories.
  * **Top Languages**: Persentase penggunaan bahasa pemrograman dihitung otomatis dari baris kode repositori (`Python 44%`, `TypeScript 41%`, dll.).
  * **GitHub Streak Stats** (`github-readme-streak-stats.herokuapp.com`): Melacak konsistensi commit harian berturut-turut.

---

## 3. Alur Otomasi CLI & Akses Repository

Pengelolaan repositori profil ini dilakukan langsung melalui terminal lokal menggunakan integrasi **Git** dan **GitHub CLI (`gh`)**:

1. **Autentikasi GitHub CLI**:
   Komputer kamu sudah login via `gh auth login` dengan token akses berizin:
   * `repo`: Akses baca-tulis penuh ke semua repositori publik & privat.
   * `workflow`: Akses integrasi CI/CD.
   * `gist`: Akses manajemen snippet.

2. **Siklus Pembaruan (Git Workflow)**:
   ```bash
   # Masuk ke direktori repositori profil
   cd C:\Users\Adika\Adikafazaa

   # Edit file README.md
   # (lakukan perubahan teks, badge, atau link)

   # Commit perubahan
   git add README.md
   git commit -m "update: perbarui konten profil"

   # Push langsung ke GitHub
   git push origin main
   ```
   Begitu perintah `git push` selesai, halaman `github.com/Adikafazaa` akan langsung menampilkan versi terbaru.

---

## 4. GitHub Camo & Cache Busting

GitHub tidak langsung menampilkan URL gambar eksternal ke browser pengunjung. Demi alasan keamanan dan privasi, GitHub menggunakan reverse-proxy caching bernama **Camo**:
* URL asli: `https://github-stats-extended.vercel.app/api?...`
* URL yang di-render GitHub: `https://camo.githubusercontent.com/<hash>`

### Jika Gambar Tidak Berubah Setelah Di-Push:
* Browser pengunjung biasanya meng-cache hasil gambar Camo selama beberapa jam.
* **Solusi**: Lakukan **Hard Refresh** di browser menggunakan shortcut:
  * Windows/Linux: `Ctrl + F5` atau `Ctrl + Shift + R`
  * Mac: `Cmd + Shift + R`

---

## 5. Panduan Kustomisasi Cepat (Cheat Sheet)

### A. Mengubah Tema Warna Kartu Statistik
Ganti parameter `theme=tokyonight` pada URL kartu statistik dengan tema lain, contoh:
* `dracula`
* `nord`
* `radical`
* `gruvbox`
* `onedark`
* `catppuccin_mocha`

### B. Menambah Distro Linux atau Tool Baru di Tech Stack
Cukup tambahkan baris badge Markdown baru dengan pola:
```markdown
![NamaDistro](https://img.shields.io/badge/NamaDistro-HEXCOLOR?style=for-the-badge&logo=NAMA_LOGO&logoColor=white)
```
*Daftar nama logo dan kode hex resmi dapat dicari di https://simpleicons.org/.*

### C. Menambah Proyek Baru di Tabel "Featured Projects"
Tambahkan baris baru pada tabel Markdown:
```markdown
| **[Nama-Repo](https://github.com/Adikafazaa/Nama-Repo)** | Deskripsi singkat fungsi proyek kamu. | `Tech1` `Tech2` |
```

---

## 6. Ringkasan Lokasi Berkas Lokal

* **Repositori Profil Lokal**: `C:\Users\Adika\Adikafazaa\`
* **File Profil Utama**: `C:\Users\Adika\Adikafazaa\README.md`
* **Remote Git Origin**: `https://github.com/Adikafazaa/Adikafazaa.git`
* **Dokumentasi Pengetahuan**: File ini (`knowledge/github-profile-system.md`)
