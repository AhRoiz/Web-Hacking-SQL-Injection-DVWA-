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

<img width="795" height="554" alt="Screenshot 2025-11-27 184816" src="https://github.com/user-attachments/assets/eada0db3-e3fc-4be3-afb3-651aed07fe9f" />

**Hasil:**
Berhasil mem-bypass restriksi UI dan filter keamanan. Database menampilkan seluruh daftar user dan password hash.

<img width="276" height="140" alt="Screenshot 2025-11-27 191025" src="https://github.com/user-attachments/assets/9b23d0f1-1530-4652-99b0-6dec7fe7c233" /> <img width="257" height="143" alt="image" src="https://github.com/user-attachments/assets/922b44c6-3f9f-46b8-b98c-34aea6a89c61" /> <img width="295" height="160" alt="image" src="https://github.com/user-attachments/assets/0c19c8ea-f15c-44a5-87fb-6f3d902df071" /> <img width="287" height="148" alt="image" src="https://github.com/user-attachments/assets/e3566215-9e7d-4589-8ac0-24848076c331" />




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
