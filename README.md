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

<img width="276" height="140" alt="Screenshot 2025-11-27 191025" src="https://github.com/user-attachments/assets/9b23d0f1-1530-4652-99b0-6dec7fe7c233" /> <img width="257" height="143" alt="image" src="https://github.com/user-attachments/assets/922b44c6-3f9f-46b8-b98c-34aea6a89c61" /> <img width="295" height="160" alt="image" src="https://github.com/user-attachments/assets/0c19c8ea-f15c-44a5-87fb-6f3d902df071" /> <img width="282" height="146" alt="image" src="https://github.com/user-attachments/assets/90de5181-65e3-4d7f-acd5-01bba73bffb8" />





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

---

## 🔐 Web Hacking: SQL Injection & Password Cracking
**Target:** DVWA (Security Level: Medium)
**Objective:** Mengambil alih akun administrator dengan mencuri dan memecahkan password hash database.

### 1. Attack Phase (Red Team)

**Tantangan (Security Obstacle):**
Pada level Medium, aplikasi menerapkan filter `mysql_real_escape_string()` yang memblokir karakter tanda petik (`'`). Hal ini mencegah serangan SQL Injection standar berbasis teks.

**Teknik Bypass:**
Ditemukan bahwa parameter `id` tidak divalidasi sebagai angka (Integer). Dengan menggunakan **Burp Suite**, payload disuntikkan langsung ke dalam paket HTTP tanpa menggunakan tanda petik, sehingga melewati filter aplikasi.

**Langkah Eksploitasi:**

1.  **Enumerasi Sistem (Reconnaissance):**
    Memastikan injeksi berhasil dengan memanggil fungsi sistem database.
    * **Payload:** `999 UNION SELECT user(),database() #`
    * **Hasil:** Database merespon dengan user: `root@localhost` dan nama DB: `dvwa`.

2.  **Pencurian Data (Data Exfiltration):**
    Menggunakan teknik *UNION SELECT* untuk menggabungkan hasil query asli dengan data dari tabel `users`. ID diubah menjadi `999` (invalid) agar aplikasi menampilkan data curian kita di baris pertama.
    * **Payload (Burp Suite):**
    * <img width="650" height="216" alt="Screenshot 2025-11-27 201219" src="https://github.com/user-attachments/assets/d49b1008-5cb8-45e8-933b-7d97473d642c" />

      ```text
      id=999+UNION+SELECT+user,password+FROM+users+%23
      ```
    * **Hasil Dump:**
      * User: `admin`
      * Password Hash: `5f4dcc3b5aa765d61d8327deb882cf99`


<img width="798" height="619" alt="Screenshot 2025-11-27 202147" src="https://github.com/user-attachments/assets/2aac09b6-9990-41ac-9fb7-501cf9873446" />

### 2. Post-Exploitation (Password Cracking)

Password yang didapatkan masih dalam bentuk enkripsi satu arah (MD5 Hash). Untuk mendapatkan password asli, dilakukan serangan *offline cracking* menggunakan tool **John The Ripper**.

**Command:**
```bash
echo "5f4dcc3b5aa765d61d8327deb882cf99" > admin_pass.txt
john --format=Raw-MD5 admin_pass.txt
```
<img width="632" height="158" alt="Screenshot 2025-11-27 201447" src="https://github.com/user-attachments/assets/d02e0121-abbe-4ee7-b476-7569d21e6704" />

<img width="636" height="299" alt="Screenshot 2025-11-27 201726" src="https://github.com/user-attachments/assets/ad47692f-bad0-4c02-b4c0-b4ea996604de" />

##**Defense & Remediation (Blue Team)**

Analisis Kerentanan: Meskipun input string sudah disanitasi, developer gagal memvalidasi Tipe Data. Input id seharusnya hanya menerima angka murni, namun aplikasi membiarkan perintah SQL masuk.

Kode Rentan (Vulnerable Code):

PHP
```bash
$id = $_GET['id']; // Tidak ada pengecekan apakah ini angka atau bukan
$id = mysql_real_escape_string($id);
$query = "SELECT first_name, last_name FROM users WHERE user_id = $id";
```
Solusi Perbaikan (Secure Code): Gunakan Casting untuk memaksa input menjadi Integer, atau gunakan Prepared Statements.

```
// Solusi 1: Integer Casting
$id = (int)$_GET['id'];

// Solusi 2: Prepared Statements (PDO)
$stmt = $pdo->prepare('SELECT first_name, last_name FROM users WHERE user_id = :id');
$stmt->execute(['id' => $id]);
