# 9Router Patronum

A local AI routing gateway with provider fallback and token-saving features. Fork of [decolua/9router](https://github.com/decolua/9router) — dengan patch tambahan untuk deployment OCI ARM + akses model yang belum dirilis ke semua akun.

## Patch yang ada di versi ini

1. **Qoder model fallback** (`open-sse/executors/qoder.js`)
   - Model yang belum ada di catalog akun (contoh: `qd/cmodel` / Cantus) otomatis pakai config dari model sibling (`qmodel_38max`).
   - Efek: `qd/cmodel` bisa langsung dipakai walau akun Qoder belum dapat entitlemen resmi. Upstream menerima config clone (verified).
   - Tambah model lain: edit `FALLBACK_CONFIG_MAP` di `buildQoderRequestBody`.

2. **Docker build tanpa buildx cache mount** (`Dockerfile`)
   - `npm ci` tanpa `--mount=type=cache` — kompatibel dengan environment tanpa BuildKit penuh (OCI ARM).

3. **Opencode executor khas OCI** (`open-sse/executors/opencode-go.js`)
   - Executor opencode yang dipakai di setup 9router OCI. Pertahankan saat sync upstream.

## Cara pakai

### Opsi 1: Docker (disarankan)

```bash
git clone https://github.com/jonijonna23-source/9router-patronum.git
cd 9router-patronum

# build image
docker build -t 9router-patronum:latest .

# running
mkdir -p 9router-data
docker run -d \
  --name 9router \
  -p 20128:20128 \
  -v 9router-data:/app/data \
  -e DATA_DIR=/app/data \
  -e PORT=20128 \
  -e HOSTNAME=0.0.0.0 \
  -e NODE_ENV=production \
  -e JWT_SECRET=<generate-with-openssl-rand-hex-32> \
  -e INITIAL_PASSWORD=<your-dashboard-password> \
  -e API_KEY_SECRET=<generate-with-openssl-rand-hex-32> \
  -e MACHINE_ID_SALT=<generate-with-openssl-rand-hex-32> \
  9router-patronum:latest
```

Dashboard: `http://localhost:20128/dashboard`

### Opsi 2: Manual (Node.js 22+)

```bash
git clone https://github.com/jonijonna23-source/9router-patronum.git
cd 9router-patronum
cp .env.example .env
# isi JWT_SECRET, INITIAL_PASSWORD, API_KEY_SECRET, MACHINE_ID_SALT
npm install
npm run build
npm run start
```

## Setelah install: tambah model custom (misal `qd/cmodel`)

1. Buka dashboard → Providers → Qoder
2. **Add Custom Model** → isi `cmodel`
3. Simpan → model muncul di `/v1/models` sebagai `qd/cmodel`
4. Langsung bisa dipakai — fallback config di executor otomatis aktif

## Sinkronisasi dengan upstream

```bash
git fetch upstream
git merge upstream/master
# re-apply patch kalau konflik (qoder.js, Dockerfile, opencode-go.js)
```

Patch di source langsung — **tidak perlu script patch otomatis** setelah install dari repo ini.

## Info

- Upstream: [decolua/9router](https://github.com/decolua/9router)
- License: MIT (sama dengan upstream)