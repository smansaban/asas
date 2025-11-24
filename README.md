# Repo untuk deploy ke Vercel + notifikasi Google Apps Script

Repo ini men-deploy situs statis ke Vercel yang menampilkan URL repo saat ini (`public/link.json`).
Setiap push ke repo akan:
- memperbarui `public/link.json` ke `https://github.com/<owner>/<repo>`
- meng-POST JSON `{"repo":"https://github.com/<owner>/<repo>"}` ke Google Apps Script URL:
  `https://script.google.com/macros/s/AKfycbw2ycTrj5Q3w8S6EH7syGwuzu1hP-C9MZkJX2uLKuk/dev`

## Cara pakai
1. Buat repo baru di GitHub, copy semua file di atas ke repo.
2. Connect repo ke Vercel (Vercel → New Project → pilih repo).
3. Pastikan branch `main` (atau branch default Anda) berisi file.
4. (Opsional) Edit `public/link.json` default jika mau.
5. Push perubahan. Workflow akan otomatis:
   - update `public/link.json`
   - commit & push jika ada perubahan
   - POST ke Google Apps Script URL yang tercantum

## Catatan penting
- Endpoint Google Apps Script **harus** menerima POST JSON. Contoh Apps Script simple:
```javascript
function doPost(e) {
  var payload = JSON.parse(e.postData.contents);
  // simpan payload.repo ke PropertiesService, Spreadsheet, dsb.
  return ContentService.createTextOutput(JSON.stringify({status:"ok", repo: payload.repo})).setMimeType(ContentService.MimeType.JSON);
}

## Otomatis Deploy ke Vercel saat `public/link.json` berubah

Untuk memastikan Vercel melakukan deploy ulang saat Anda mengubah `public/link.json`, gunakan Vercel Deploy Hook dan GitHub Actions:

- Di Vercel: buka project → Settings → Git → create "Deploy Hook". Salin URL hook.
- Di GitHub repo: buka Settings → Secrets and variables → Actions → New repository secret. Buat secret `VERCEL_DEPLOY_HOOK` berisi URL hook tadi.
- Workflow yang sudah ditambahkan akan memanggil hook setiap ada push yang mengubah `public/link.json`.

File workflow berada di `.github/workflows/deploy-on-link-change.yml`.

Contoh penggunaan:

1. Edit `public/link.json` lalu push ke `main`.
2. GitHub Actions mendeteksi perubahan pada path `public/link.json` dan POST ke Deploy Hook.
3. Vercel menerima POST dan memicu deploy baru, sehingga perubahan file akan langsung terlihat di situs.

Catatan: Deploy Hook harus dibuat pada project Vercel yang terhubung ke repo ini. Jangan menaruh URL hook secara publik — gunakan GitHub Secrets.
