# Lead Intake & CRM Synchronization System

## Ringkasan
Workflow asinkron untuk menerima lead, menyimpan data secara persisten, lalu menyinkronkannya ke CRM secara terkontrol. Penerimaan lead dipisahkan dari proses pengiriman agar proses intake tidak bergantung langsung pada ketersediaan CRM.

Sistem menggunakan buffer, worker, retry, dan recovery untuk menangani kegagalan serta hasil request yang tidak pasti.

## Masalah
Jika banyak lead masuk dalam waktu singkat, pengiriman langsung dari webhook ke CRM dapat membuat proses penerimaan bergantung pada ketersediaan CRM. Rate limit, server error, atau timeout juga dapat membuat status sinkronisasi sulit dipastikan.

## Alur solusi
1. Webhook menerima lead.
2. Data dinormalisasi dan divalidasi. Nomor telepon dinormalisasi ke format Indonesia dan digunakan sebagai `lead_id`.
3. Data valid disimpan ke PostgreSQL sebagai buffer.
4. Worker mengambil batch lead berstatus `pending` menggunakan atomic claiming.
5. Worker mengirim lead ke Mock CRM melalui REST API.
6. Status buffer diperbarui berdasarkan respons.
7. Jika hasil request tidak pasti, proses recovery melakukan lookup ke CRM sebelum menentukan apakah lead berhasil atau perlu dicoba kembali.

## Komponen dan teknologi
- **n8n:** orkestrasi workflow.
- **PostgreSQL/Supabase:** penyimpanan payload dan status operasional.
- **REST API / Mock CRM:** simulasi endpoint CRM untuk pengujian.
- **SQLite:** penyimpanan record pada Mock CRM.

## Keputusan teknis
- Worker menggunakan `FOR UPDATE SKIP LOCKED` untuk mengklaim pekerjaan secara atomik di database.
- Status operasional meliputi `pending`, `processing`, `success`, `unknown`, dan `failure_data`.
- HTTP 400 diarahkan ke `failure_data`.
- HTTP 500 dapat dicoba kembali dengan batas percobaan.
- HTTP 429 mengikuti `Retry-After`.
- Timeout masuk ke status `unknown`, karena CRM mungkin sudah menyimpan record meskipun respons tidak diterima.
- Recovery melakukan lookup berdasarkan `lead_id` untuk merekonsiliasi status buffer dengan kondisi record pada CRM.

Atomic claiming mengoordinasikan worker di database, tetapi bukan transaksi atomik antara database dan CRM.

## Pengujian dan hasil
Pengujian dilakukan menggunakan Mock CRM, bukan CRM produksi.

| Metrik | Hasil |
|---|---:|
| Total request | 1.000 |
| Lead baru | 780 |
| Request duplikat | 200 |
| Input invalid | 20 |
| Record CRM | 780 |
| Duplikasi terdeteksi | 0 |
| Waktu kuras antrean | 541 detik |
| Latensi p50 | 107,3 detik |
| Latensi p95 | 154,4 detik |
| Latensi maksimum | 195,6 detik |

Pada konfigurasi pengujian tersebut, 780 lead baru menghasilkan 780 record CRM. Rekonsiliasi menunjukkan tidak ada lead hilang, duplikasi yang terdeteksi, backlog, atau status yang belum terselesaikan. Angka ini berlaku untuk beban dan konfigurasi pengujian tersebut, bukan klaim kapasitas maksimum atau SLA produksi.

## Batasan dan pengembangan berikutnya
- Mock CRM digunakan sebagai pengganti CRM produksi. Autentikasi, kontrak API, rate limit, dan dukungan idempotency pada CRM target masih perlu divalidasi.
- Identitas lead menggunakan nomor telepon. Beberapa inquiry dari nomor yang sama dapat tergabung atau saling memperbarui konteks.
- Batas total percobaan retry 429 lintas siklus antrean dan kebijakan alert jangka panjang masih perlu dipertegas.
- State yang terlalu lama berada di `processing` perlu dipulihkan tanpa mengambil alih pekerjaan yang masih berjalan. Lease atau token kepemilikan claim dapat dipertimbangkan.

## Dokumentasi lengkap
Lihat [dokumentasi proyek](https://github.com/Rifkimaulanaaji/ai-automation-portfolio/blob/main/ai-workflow-portfolio/lead-intake-crm-sync/docs/lead-intake-crm-synchronization-system.pdf).
