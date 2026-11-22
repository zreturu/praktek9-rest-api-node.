# Praktek 9 – REST API Suhu dengan Node-RED

Flow Node-RED (`flows.json`) untuk REST API suhu, lengkap dengan lima tugas bonus.

| Tugas bonus | Implementasi |
|---|---|
| `PUT /api/threshold` + notifikasi | Set batas suhu. Jika suhu melewati batas, dikirim pesan `alert` lewat WebSocket (banner + Notification di dashboard) dan `node.warn` di debug Node-RED |
| Grafik real-time Chart.js | `GET /dashboard`: grafik suhu dan garis batas, update langsung tanpa refresh |
| Autentikasi sederhana | Semua endpoint `/api/*` wajib header `X-API-Key` (401 bila salah) |
| WebSocket | Listener `/ws/suhu` mem-broadcast data baru ke dashboard, tanpa polling |
| History ke database | SQLite (`suhu.db`), tabel `readings`; `GET /api/history` membaca dari DB, bukan flow context |

## Instalasi

1. Install node SQLite: `cd ~/.node-red && npm install node-red-node-sqlite` (atau lewat *Manage palette*).
2. (Opsional) set API key: `API_KEY=rahasiaku node-red` (default `secret123`).
3. Di Node-RED: menu → Import → pilih `flows.json` → Deploy.
4. Buka `http://localhost:1880/dashboard`, isi API Key, klik *Simpan & muat*.

Sensor disimulasikan tiap 2 detik. Sensor asli bisa mengirim lewat `POST /api/suhu`.

## Endpoint

| Method | Path | Keterangan |
|---|---|---|
| GET | `/api/suhu` | Data terakhir |
| POST | `/api/suhu` | Kirim data `{"temp":31.2,"hum":60}` |
| GET | `/api/history?limit=50` | History dari SQLite |
| GET | `/api/threshold` | Batas suhu saat ini |
| PUT | `/api/threshold` | Body `{"threshold":32}` |
| GET | `/dashboard` | Dashboard Chart.js (publik; data tetap butuh API key) |

## Contoh uji

```bash
curl -H "X-API-Key: secret123" localhost:1880/api/suhu
curl -X PUT -H "X-API-Key: secret123" -H "Content-Type: application/json" \
     -d '{"threshold":25}' localhost:1880/api/threshold
curl localhost:1880/api/suhu            # -> 401 tanpa API key
```

## Catatan
- Threshold disimpan di flow context (hilang saat restart; default 30 °C). Data history tersimpan permanen di SQLite.
- WebSocket hanya mengirim data (read-only) sehingga tidak memakai API key.
