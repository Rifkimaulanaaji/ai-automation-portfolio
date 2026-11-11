# RAG Customer Service Bot

## Ringkasan
Workflow customer service berbasis retrieval untuk membantu menjawab pertanyaan pelanggan menggunakan knowledge base SOP. Workflow memisahkan tiga jalur: balasan normal, eskalasi ke admin, dan fallback ketika AI Agent mengalami error.

## Kebutuhan yang ditangani
Pertanyaan berulang mengenai ongkir, estimasi pengiriman, ukuran, bahan, retur, pembayaran, dan promo membutuhkan jawaban yang konsisten.

Pertanyaan mengenai stok belum dapat dijawab otomatis karena workflow tidak memiliki tool untuk memeriksa stok.

## Alur solusi
1. Pesan masuk diterima melalui webhook WAHA.
2. Payload dinormalisasi dan divalidasi.
3. Pesan yang lolos validasi diteruskan ke AI Agent.
4. Google Gemini 2.5 Flash menghasilkan jawaban dengan bantuan knowledge base SOP melalui Supabase Vector Store.
5. Workflow memisahkan respons menjadi:
   - **Normal:** menyiapkan balasan dan mengirimkannya melalui WAHA.
   - **Eskalasi:** menyiapkan balasan untuk pelanggan dan mengirim alert kepada admin melalui Telegram.
   - **Agent error:** mengirim fallback kepada pelanggan dan alert teknis melalui Telegram.

## Komponen dan teknologi
- **n8n:** orkestrasi workflow.
- **Google Gemini 2.5 Flash:** model pada AI Agent.
- **Supabase Vector Store:** retrieval knowledge base SOP.
- **WAHA:** penerimaan dan pengiriman pesan WhatsApp.
- **Telegram:** notifikasi eskalasi dan error.

## Keputusan teknis
- Tool `search_sop` diarahkan untuk digunakan saat menjawab pertanyaan kebijakan toko.
- Supabase Vector Store menggunakan tabel `documents`, query function `match_documents`, `topK = 6`, dan Gemini embeddings.
- Memory menggunakan `chatId` sebagai custom key dengan context window 16.
- Output eskalasi menggunakan format `ESCALATE||alasan||balasan_customer`, kemudian diperiksa untuk menentukan jalur eskalasi.

## Status dan batasan
Dokumentasi ini menjelaskan konfigurasi yang terlihat pada workflow JSON. Workflow berstatus nonaktif (`active: false`), pinned payload tersedia, dan execution log tidak disertakan. Karena itu, konfigurasi jalur bukan bukti bahwa workflow telah berhasil dijalankan end-to-end.

Hal yang masih perlu diperiksa sebelum penggunaan nyata:
- Validasi event type dan field `fromMe`. Field `fromMe` yang tidak tersedia dapat dinormalisasi menjadi `false`, sehingga perlu diperketat.
- Kredensial Supabase, WAHA, dan Telegram perlu dikonfigurasi di environment n8n.
- Retrieval SOP dan akurasi jawaban perlu diuji.
- Format output eskalasi perlu divalidasi sebelum parsing.
- Pengiriman pesan pada semua jalur perlu diuji untuk timeout dan HTTP error.
- Memory disebut menggunakan penyimpanan in-memory dan dapat hilang ketika n8n restart.
- Autentikasi atau validasi sumber request perlu dipertimbangkan.

## Dokumentasi lengkap
Lihat [dokumentasi proyek](https://github.com/Rifkimaulanaaji/ai-automation-portfolio/blob/main/ai-workflow-portfolio/rag-customer-service-bot/docs/rag-customer-service-bot.pdf).
