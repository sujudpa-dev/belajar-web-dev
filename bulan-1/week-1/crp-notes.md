# Critical Rendering Path — Notes Day 1

## 1. DOM Construction (Membangun Rangka HTML)

Saat pertama kali memuat sebuah halaman web, browser menerima data mentah berupa urutan angka (bytes). Browser kemudian membaca kode HTML ini dari atas ke bawah dan mengubahnya menjadi **DOM (Document Object Model)**. Ibaratnya, browser sedang membuat silsilah pohon keluarga untuk seluruh isi website—mulai dari `<html>` sebagai akar utama, lalu memiliki anak `<head>` dan `<body>`, dan seterusnya.
_Catatan penting:_ Jika browser bertemu dengan tag `<script>`, ia akan berhenti membaca HTML untuk memproses dan menjalankan script itu terlebih dahulu (_parser-blocking_). Itulah sebabnya script lebih baik diletakkan di bagian paling bawah atau menggunakan atribut `defer`.

## 2. CSSOM Construction (Membangun Rangka Desain)

Bersamaan dengan membuat DOM, browser juga membaca file CSS dan membuat struktur pohonnya sendiri yang disebut **CSSOM (CSS Object Model)**. Pohon ini berisi semua aturan visual seperti warna dan ukuran. CSS bersifat _render-blocking_—artinya browser menolak menampilkan apa pun ke layar sebelum seluruh CSS selesai diproses. Tujuannya agar pengguna tidak sempat melihat halaman HTML mentah yang berantakan sebelum desainnya dimuat.

## 3. Render Tree (Menggabungkan Rangka & Desain)

Sekarang browser memiliki pohon DOM (struktur/isi) dan CSSOM (desain visual). Keduanya kemudian digabung menjadi **Render Tree**. Aturan utamanya sangat sederhana: Render Tree _hanya_ memedulikan elemen yang benar-benar akan terlihat di layar. Jika ada elemen dengan kode CSS `display: none`, elemen itu langsung dibuang dan tidak masuk ke Render Tree. Namun, jika menggunakan `visibility: hidden`, elemen tersebut tetap masuk karena walaupun tidak terlihat secara visual, ia tetap memakan ruang (spasi) di halaman.

## 4. Layout / Reflow (Menghitung Posisi & Ukuran)

Setelah tahu elemen apa saja yang akan ditampilkan beserta desainnya, browser mulai menghitung matematika spasialnya: _"Berapa pixel lebar tombol ini? Di mana tepatnya posisi gambar ini pada layar?"_ Proses perhitungan geometri ini disebut **Layout** atau **Reflow**. Proses ini sangat "mahal" (memakan banyak tenaga komputer). Setiap kali kita mengubah ukuran layar atau memodifikasi lebar/margin menggunakan JavaScript, browser terpaksa harus menghitung ulang letak semuanya dari awal.

## 5. Paint & Compositing (Mewarnai & Menggabungkan)

Setelah letak dan ukurannya pasti, browser mulai "mewarnai" pixel-pixel di layar dengan warna, teks, bayangan, atau gambar yang sesuai. Tahap ini disebut **Paint**. Browser sering kali melukis elemen-elemen ini ke dalam beberapa lapisan (layers) yang terpisah. Terakhir, pada tahap **Compositing**, semua lapisan gambar tersebut ditumpuk menjadi satu kesatuan visual akhir yang utuh dan ditampilkan ke layar monitormu.

## 💡 Insight Paling Penting

**Kenapa membuat animasi menggunakan properti `transform` jauh lebih cepat dan mulus dibandingkan menggunakan properti `left`, `top`, atau `width`?**
Mengubah properti seperti `width` atau `left` akan memaksa browser untuk mengulang seluruh proses **Layout** dan **Paint** yang sangat membebani kinerja. Browser harus menghitung ulang letak elemen tersebut beserta elemen-elemen lain di sekitarnya. Akibatnya, animasi akan terasa berat dan patah-patah (_jank_).
Sebaliknya, jika kita menggunakan properti `transform` atau `opacity`, browser cukup cerdas untuk mendelegasikan tugas tersebut langsung ke GPU (kartu grafis) pada tahap **Compositing**. Browser akan sepenuhnya melompati tahap Layout dan Paint. Hasilnya: animasi diproses jauh lebih ringan dan berjalan sangat mulus di 60fps!
