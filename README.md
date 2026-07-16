# Chatbot Sewa Jas — Flask + Ollama (LLM lokal)

Chatbot ini jalan **100% offline** di laptop kamu. Backend pakai Flask (Python),
tampilan pakai HTML/CSS/JS custom, dan otaknya adalah LLM lokal lewat Ollama.
Tidak pakai model ML klasifikasi intent — pertanyaan pelanggan langsung dijawab
oleh LLM, dengan data toko (`knowledge_base.json`) dan contoh gaya bicara
(`intents.json`) ditaruh sebagai konteks, supaya jawabannya selalu sesuai data
toko yang sebenarnya.

## Kenapa model segini, cocok RAM 8GB?

Laptop dengan RAM 8GB dipakai bareng OS + browser + aplikasi lain, jadi model
LLM yang jalan sebaiknya **quantized dan kecil (2–3 miliar parameter)**:

| Model                    | Ukuran RAM (kira-kira) | Catatan                                          |
|---------------------------|-------------------------|---------------------------------------------------|
| `llama3.2:3b` (default)   | ~2 GB                  | Rekomendasi utama, Bahasa Indonesia cukup bagus    |
| `qwen2.5:3b-instruct`     | ~2 GB                  | Alternatif, kuat multibahasa                       |
| `qwen2.5:1.5b`            | ~1 GB                  | Kalau `3b` masih berat/lemot                       |
| `gemma2:2b`               | ~1.5 GB                | Alternatif ringan lainnya                          |

Hindari model 7B ke atas — butuh RAM 6–8GB+ hanya untuk model-nya saja, akan
sangat lambat atau gagal jalan di RAM 8GB total.

## 1. Install Ollama

Download & install dari **https://ollama.com/download** (Windows/macOS/Linux).
Setelah terinstall, Ollama otomatis jalan sebagai service di background
(`http://localhost:11434`).

## 2. Tarik (download) model

```bash
ollama pull llama3.2:3b
```

Kalau nanti laptop terasa berat, ganti ke model yang lebih kecil:

```bash
ollama pull qwen2.5:1.5b
```

lalu isi nama model itu di kolom **"Model Ollama"** pada panel pengaturan (ikon
gear di pojok kanan atas aplikasi).

## 3. Install dependency Python

Pastikan Python 3.9+ sudah terpasang. Di folder project ini:

```bash
pip install -r requirements.txt
```

## 4. Jalankan aplikasi

```bash
python app.py
```

Buka browser ke **http://localhost:5000**.

## Struktur folder

```
chatbot-sewa-jas-flask/
├── app.py                 # Backend Flask + integrasi Ollama
├── requirements.txt
├── intents.json           # Contoh Q&A per intent (referensi gaya bicara)
├── knowledge_base.json    # Data toko (harga, ukuran, kebijakan, dll)
├── templates/
│   └── index.html         # Halaman chat
└── static/
    ├── css/style.css      # Tampilan (tema navy & brass, terinspirasi tailor shop)
    └── js/chat.js          # Logic kirim pesan & streaming respons
```

## Tips performa di RAM 8GB

- Tutup aplikasi berat lain (banyak tab browser, dsb) saat chatbot jalan.
- Kalau jawaban lambat/laptop nge-lag, turunkan `NUM_CTX` di `app.py` dari `4096`
  menjadi `2048`.
- Kalau masih berat, pindah ke model yang lebih kecil (`qwen2.5:1.5b` atau
  `gemma2:2b`) lewat panel pengaturan.
- Riwayat chat disimpan di `localStorage` browser (bukan server) — klik
  **"Reset percakapan"** di panel pengaturan kalau sudah lama chatting dan mulai
  berat.

## Cara kerja singkat

1. `knowledge_base.json` diringkas jadi teks (harga, ukuran, warna, kebijakan, dll)
   saat server Flask start.
2. Beberapa contoh Q&A dari `intents.json` disertakan sebagai referensi gaya
   bicara (santai, ramah, ala admin online shop).
3. Keduanya digabung jadi **system prompt**, dikirim ke model Ollama di setiap
   giliran chat lewat endpoint `/api/chat`, yang men-streaming jawaban token demi
   token ke browser (`fetch` + `ReadableStream` di `chat.js`).
4. Kalau `knowledge_base.json` diubah (misal harga naik), tinggal restart
   `python app.py` dan jawaban chatbot otomatis ikut update — tanpa training
   ulang apa pun.

## Kalau ingin ganti isi data toko

Edit langsung `knowledge_base.json` (harga, ukuran, kebijakan, dll) dan
`intents.json` (contoh pertanyaan & gaya jawaban), lalu restart `python app.py`.

## Troubleshooting

- **"Tidak bisa terhubung ke Ollama"** → pastikan aplikasi Ollama sudah jalan,
  atau jalankan manual: `ollama serve`.
- **Error model not found** → pastikan sudah `ollama pull <nama_model>` sesuai
  yang diisi di kolom "Model Ollama".
- **Port 5000 sudah dipakai** → ubah baris terakhir `app.py`:
  `app.run(host="0.0.0.0", port=5001, debug=True)` lalu buka
  `http://localhost:5001`.
