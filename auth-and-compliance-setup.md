# Panduan Konfigurasi Autentikasi Google, Supabase, & Kepatuhan Google Play Console

Dokumen internal ini memuat langkah manual konfigurasi Google Cloud Platform (GCP), Supabase Dashboard, dan Google Play Console untuk aplikasi **BuildEra** (v1.2.3).

---

## 📌 1. Konfigurasi Google Cloud Platform (GCP)
> Proyek GCP: `caltax-caa38` (atau nama proyek GCP Anda)

### A. Layar Persetujuan OAuth (*OAuth Consent Screen*)
1. Buka GCP Console (https://console.cloud.google.com) ➔ **APIs & Services** ➔ **OAuth consent screen**.
2. Pilih jenis pengguna **External**.
3. Isikan data aplikasi:
   - **App name**: `BuildEra`
   - **User support email**: `wenyafi@gmail.com`
   - **Developer contact info**: `wenyafi@gmail.com`
   - **App domain links** (Boleh menggunakan domain gratis seperti GitHub Pages / Cloudflare Pages):
     - Homepage: `https://alyafi0899.github.io/BuildEra-Construction-Store-POS/`
     - Privacy Policy: `https://alyafi0899.github.io/BuildEra-Construction-Store-POS/privacy-policy/`
     - Terms of Service: `https://alyafi0899.github.io/BuildEra-Construction-Store-POS/terms/`
4. **Scopes**: Tambahkan `openid`, `https://www.googleapis.com/auth/userinfo.email`, dan `https://www.googleapis.com/auth/userinfo.profile`.
5. **Publish Status**: Ubah dari status *Testing* menjadi **In Production** agar semua akun Google pengguna dapat masuk tanpa dimasukkan manual ke test users.

### B. Pembuatan Kredensial OAuth (*OAuth Credentials*)
Buat 2 Kredensial pada GCP Console ➔ **APIs & Services** ➔ **Credentials**:

1. **Web Application Client ID** (Digunakan untuk `serverClientId` pada Flutter & Supabase):
   - Type: **Web application**
   - Name: `BuildEra Web Server Client`
   - Authorized redirect URIs: `https://<YOUR_SUPABASE_REF>.supabase.co/auth/v1/callback`
   - Salin **Client ID** dan **Client Secret**.

2. **Android Client ID**:
   - Type: **Android**
   - Name: `BuildEra Android App`
   - Package name: `com.solutivostudio.caltax`
   - **Wajib daftarkan TIGA SHA-1 Fingerprint**:
     - 🔹 **SHA-1 Debug**: `keytool -list -v -keystore ~/.android/debug.keystore`
     - 🔹 **SHA-1 Release / Upload Key**: SHA-1 dari `key.properties` / `release.jks`
     - 🔹 **SHA-1 Google Play App Signing**: Dapatkan dari **Google Play Console** ➔ **Setup** ➔ **App signing** ➔ **App signing key certificate**.
   - ⚠️ **PERINGATAN KRITIS**: Tanpa memasukkan SHA-1 Google Play App Signing, login Google akan **GAGAL TOTAL (Error Code 10 / ApiException)** ketika aplikasi diunduh pengguna dari Play Store!

---

## 📌 2. Konfigurasi Supabase Dashboard

1. Buka **Supabase Dashboard** ➔ **Authentication** ➔ **Providers** ➔ **Google**.
2. Ubah toggle menjadi **Enabled (ON)**.
3. **Client IDs**: Masukkan **Web Client ID** dan **Android Client ID** (pisahkan dengan koma jika diisi lebih dari satu).
4. **Client Secret**: Masukkan Client Secret dari Web Client ID.
5. Pastikan **Confirm email** aktif jika Anda mengharuskan verifikasi email untuk registrasi email/password.

---

## 📌 3. Pengisian Data Safety Google Play Console

Buka **Google Play Console** ➔ Pilih aplikasi **BuildEra** ➔ **App content** ➔ **Data safety**:

| Jenis Data | Dikumpulkan? | Dibagikan? | Tujuan Pengumpulan |
| --- | --- | --- | --- |
| **Name** (Nama Pemilik) | Ya | Tidak | Account Management, App Functionality |
| **Email address** | Ya | Tidak | Account Management, App Functionality |
| **User IDs** | Ya | Tidak | Account Management, App Functionality |
| **Financial Info (Purchases)** | Ya | Tidak | App Functionality, Analytics (Google Play Billing) |
| **Device or other IDs** | Ya | Ya | Advertising & Marketing (AdMob), Analytics |
| **App Feedback & Tech Info** | Ya (Opsional) | Tidak | App Functionality, Analytics, Troubleshooting |
| **Consent Logs** | Ya | Tidak | Compliance & Legal Requirement |

---

## 📌 4. Pengaturan Langganan Google Play Console
1. Buka **Play Console** ➔ **Monetize** ➔ **Subscriptions**.
2. **Grace Period**: Aktifkan *Grace Period* (mis. 7 atau 14 hari) agar pengguna diberikan kesempatan memperbarui pembayaran gagal tanpa langsung kehilangan akses.
3. **Account Hold**: Aktifkan *Account Hold* (mis. 30 hari) untuk membekukan fitur premium sementara saat pembayaran belum diselesaikan.
4. **Pajak & Harga**: Pastikan harga bulanan (`buildera_monthly`) dan tahunan (`buildera_yearly`) sudah termasuk pajak lokal jika diwajibkan oleh regulasi setempat.

---

## 📌 5. Migrasi Database Supabase
Jalankan perintah berikut untuk menerapkan migrasi tabel baru (`user_consents` & `app_feedback`):
```bash
supabase db push
```
Atau jalankan file migrasi SQL berikut di Supabase SQL Editor:
- `supabase/migrations/20261102000000_create_user_consents_table.sql`
- `supabase/migrations/20261102000001_create_app_feedback_table.sql`

**Query Verifikasi**:
```sql
SELECT count(*) FROM public.user_consents;
SELECT count(*) FROM public.app_feedback;
```

---

## 📌 6. Skenario Pengujian Kunci Sebelum Rilis
- [x] Skenario 1: Pendaftaran akun email baru ➔ verifikasi email ➔ `ConsentScreen` ➔ `StoreSetupWizard` ➔ `SetupPinPage` ➔ `MainShell`.
- [x] Skenario 2: Login Google akun baru ➔ `ConsentScreen` ➔ `StoreSetupWizard` ➔ `SetupPinPage` ➔ `MainShell`.
- [x] Skenario 3: Akun email lama masuk kembali ➔ `ConsentScreen` (pembaruan versi dokumen) ➔ `MainShell`.
- [x] Skenario 4: Email akun sudah ada via password, lalu mencoba "Lanjutkan dengan Google" ➔ menampilkan pesan penolakan ramah.
- [x] Skenario 5: Lupa PIN untuk akun Google ➔ autentikasi ulang Google via `ReauthService`.
- [x] Skenario 6: Kirim masukan privat (Feedback) max 5x sehari.
