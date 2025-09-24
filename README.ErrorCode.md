# FIA Documentation (Error Code)

List of all possible error codes and their details.


| Kode Error | HTTP Status | Status Code | Pesan Error | Deskripsi |
|------------|-------------|-------------|-------------|------------|
| 511.00 | 500 | Internal Server Error | general error | Error umum sistem |
| 511.01 | 400 | Bad Request | gateway does not exists | Gateway tidak ditemukan |
| 511.02 | 400 | Bad Request | merchant is inactive | Merchant tidak aktif |
| 511.03 | 400 | Bad Request | gateway is inactive | Gateway tidak aktif |
| 511.04 | 400 | Bad Request | credential not provided | Kredensial tidak disediakan |
| 511.05 | 401 | Unauthorized | User identifier not provided | Identifier pengguna tidak disediakan |
| 511.06 | 401 | Unauthorized | Rate limit exceeded | Batas rate limit terlampaui |
| 511.07 | 403 | Forbidden | OTP invalid | OTP tidak valid |
| 511.08 | 403 | Forbidden | OTP expired | OTP sudah kedaluwarsa |
| 511.09 | 403 | Forbidden | OTP has been verified | OTP sudah diverifikasi |
| 511.10 | 403 | Forbidden | fia not activate | FIA tidak aktif |
| 511.11 | 404 | Not Found | transaction not found | Transaksi tidak ditemukan |
| 511.12 | 404 | Not Found | initiate not found | Data inisiasi tidak ditemukan |
| 511.13 | 405 | Method Not Allowed | provider does not exists | Provider tidak ditemukan |
| 511.14 | 405 | Method Not Allowed | otp type does not exists | Tipe OTP tidak ditemukan |
| 511.15 | 405 | Method Not Allowed | pricing does not exists | Pricing tidak ditemukan |
| 511.16 | 405 | Method Not Allowed | operator does not exists | Operator tidak ditemukan |
| 511.17 | 402 | Payment Required | the balance is not sufficient | Saldo tidak mencukupi |
| 511.18 | 422 | Unprocessable Entity | Invalid payload | Payload tidak valid |
| 511.19 | 422 | Unprocessable Entity | Oops! You must enter either the OTP or the url | Harus memasukkan OTP atau URL |
| 511.20 | 422 | Unprocessable Entity | The number you entered is not a valid number | Nomor yang dimasukkan tidak valid |
| 511.21 | 422 | Unprocessable Entity | The number you entered is not a valid number by country check | Nomor tidak valid berdasarkan pengecekan negara |
| 511.22 | 422 | Unprocessable Entity | you are not allowed to send for this provider | Tidak diizinkan mengirim untuk provider ini |
| 511.23 | 422 | Unprocessable Entity | Parameters is required, incomplete parameters or parameters not in the correct format | Parameter diperlukan, tidak lengkap atau format salah |
| 511.24 | 422 | Unprocessable Entity | OTP in your payload is invalid | OTP dalam payload tidak valid |
| 511.25 | 422 | Unprocessable Entity | operator not supported | Operator tidak didukung |
| 511.26 | 500 | Internal Server Error | create record failed | Gagal membuat record |
| 511.27 | 501 | Not Implemented | otp not allowed | OTP tidak diizinkan |
| 511.28 | 503 | Service Unavailable | Service thirdparty unavailable | Layanan pihak ketiga tidak tersedia |
| 511.29 | 503 | Service Unavailable | Parse response data of service | Gagal parsing data response layanan |
| 511.30 | 503 | Service Unavailable | base url is empty | Base URL kosong |
| 511.31 | 503 | Service Unavailable | failed to parse url | Gagal parsing URL |
| 511.32 | 503 | Service Unavailable | Vendor is down | Vendor sedang down |
| 511.33 | 503 | Service Unavailable | Operator for this channel is down | Operator untuk channel ini sedang down |
| 511.34 | 503 | Service Unavailable | channel error: invalid url provider | Error channel: URL provider tidak valid |
| 511.35 | 429 | Too Many Requests | You have failed to verify your number. Please try verifying your number again in 24 hours. | Gagal verifikasi nomor, coba lagi dalam 24 jam |
| 511.36 | 429 | Too Many Requests | Your POC Quota has been exceeded. Please contact your account manager. | Kuota POC terlampaui, hubungi account manager |
| 511.37 | 404 | Not Found | WhatsApp initiate data not found | Data inisiasi WhatsApp tidak ditemukan |
| 511.38 | 404 | Not Found | OTP verification data not found | Data verifikasi OTP tidak ditemukan |
| 511.39 | 404 | Not Found | Header Enrichment verification data not found | Data verifikasi Header Enrichment tidak ditemukan |
| 511.40 | 500 | Internal Server Error | Failed to retrieve merchant data | Gagal mengambil data merchant |
| 511.41 | 500 | Internal Server Error | Failed to get FIA log activity from Keypaz | Gagal mendapatkan log aktivitas FIA dari Keypaz |
| 511.42 | 500 | Internal Server Error | Failed to retrieve merchant app data | Gagal mengambil data aplikasi merchant |
| 511.43 | 500 | Internal Server Error | Failed to check FIA blocking status | Gagal mengecek status blocking FIA |
| 511.44 | 500 | Internal Server Error | Failed to get device binding | Gagal mendapatkan device binding |
| 511.45 | 500 | Internal Server Error | Failed to get FIA gateways | Gagal mendapatkan gateway FIA |
| 511.46 | 500 | Internal Server Error | Failed to get gateway by ID | Gagal mendapatkan gateway berdasarkan ID |
| 511.47 | 500 | Internal Server Error | Failed to get last FIA OTP gateway | Gagal mendapatkan gateway OTP FIA terakhir |
| 511.48 | 500 | Internal Server Error | Failed to get FIA digit count | Gagal mendapatkan jumlah digit FIA |
| 511.49 | 500 | Internal Server Error | Failed to generate OTP | Gagal generate OTP |
| 511.50 | 500 | Internal Server Error | Failed to get gateway by key and merchant | Gagal mendapatkan gateway berdasarkan key dan merchant |
| 511.51 | 500 | Internal Server Error | Failed to initiate user merchant | Gagal inisiasi user merchant |
| 511.52 | 500 | Internal Server Error | Failed to request Header Enrichment | Gagal request Header Enrichment |
| 511.53 | 500 | Internal Server Error | Failed to send OTP | Gagal mengirim OTP |
| 511.54 | 500 | Internal Server Error | Failed to block FIA user | Gagal memblokir user FIA |
| 511.55 | 500 | Internal Server Error | Failed to get WhatsApp initiate data | Gagal mendapatkan data inisiasi WhatsApp |
| 511.56 | 500 | Internal Server Error | Failed to get OTP verification status | Gagal mendapatkan status verifikasi OTP |
| 511.57 | 500 | Internal Server Error | Failed to get Header Enrichment verification status | Gagal mendapatkan status verifikasi Header Enrichment |
| 511.58 | 500 | Internal Server Error | Failed to decrease POC quota | Gagal mengurangi kuota POC |
| 511.59 | 500 | Internal Server Error | Phone number is not valid | Nomor Telepon Tidak Valid |

Last updated at: 24 Sep 2025 02:59:44
