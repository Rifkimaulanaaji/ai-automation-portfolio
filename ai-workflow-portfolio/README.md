# AI Automation Portfolio

Portofolio ini berisi dua proyek automasi yang saya kerjakan menggunakan n8n, integrasi API, database, dan AI.

## Proyek

### 1. Lead Intake & CRM Synchronization System
Workflow asinkron untuk menerima lead, menyimpan data secara persisten, lalu menyinkronkannya ke CRM melalui buffer dan worker. Fokus proyek ini adalah reliability: atomic claiming, retry, error handling, dan recovery ketika hasil request tidak pasti.

- **Teknologi:** n8n, PostgreSQL/Supabase, REST API, SQLite, Mock CRM
- **Dokumentasi:** [Buka proyek](./lead-intake-crm-sync/README.md)

### 2. RAG Customer Service Bot
Workflow customer service berbasis retrieval yang menggunakan knowledge base SOP untuk membantu menjawab pertanyaan pelanggan. Tersedia jalur eskalasi ke admin dan fallback ketika AI Agent mengalami error.

- **Teknologi:** n8n, Google Gemini 2.5 Flash, Supabase Vector Store, WAHA, Telegram
- **Dokumentasi:** [Buka proyek](./rag-customer-service-bot/README.md)

## Catatan
Dokumentasi menjelaskan konfigurasi dan bukti yang tersedia pada masing-masing proyek. Status implementasi dan pengujian ditulis sesuai artefak yang tersedia; tidak semua workflow telah diuji end-to-end pada lingkungan produksi.
