## 🌐 Web Hacking: SQL Injection (DVWA)

**Target:** DVWA (Damn Vulnerable Web App) pada Metasploitable 2.
**Objective:** Mengambil seluruh data user (Database Dump) dengan melewati proteksi level Medium.

### 1. Attack Phase (Red Team)

**Skenario Level Medium:**
Aplikasi mengubah input teks menjadi *Dropdown Menu*, membatasi user hanya bisa memilih angka 1-5. Selain itu, aplikasi menerapkan filter `mysql_real_escape_string()` untuk memblokir tanda petik (`'`).

**Teknik Eksploitasi:**
Menggunakan **Burp Suite (Intercepting Proxy)** untuk memanipulasi *HTTP Request* sebelum dikirim ke server.

* **Vulnerability:** Integer Based SQL Injection. Programmer tidak menggunakan tanda petik pada query SQL (`WHERE id = $id`), sehingga serangan dapat dilakukan tanpa menggunakan tanda petik yang difilter.
* **Payload:** `1 OR 1=1 #`
* **URL Encoded Payload (di Burp Suite):** `1+OR+1=1+%23`

![Screenshot Burp Suite](Link_Gambar_BurpSuite.png)

**Hasil:**
Berhasil mem-bypass restriksi UI dan filter keamanan. Database menampilkan seluruh daftar user dan password hash.

![Screenshot Hasil SQLi](Link_Gambar_Hasil_SQLi.png)

### 2. Defense Phase (Blue Team)

**Root Cause Analysis:**
Kode sumber level Medium hanya melakukan sanitasi input string (`mysql_real_escape_string`), namun gagal memvalidasi tipe data input.

**Rekomendasi Perbaikan:**
1.  **Input Validation:** Memaksa input agar selalu bertipe Integer (Angka).
    ```php
    $id = (int)$_GET['id'];
    ```
2.  **Prepared Statements:** Menggunakan parameterisasi query (PDO) agar input user tidak pernah dieksekusi sebagai perintah SQL.

---
