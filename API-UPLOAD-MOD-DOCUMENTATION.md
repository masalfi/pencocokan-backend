# API UPLOAD MOD DOCUMENTATION

## Autentikasi

Semua API endpoint memerlukan token permanen untuk autentikasi. Token ini harus disertakan di header request:

```
Authorization: Bearer YOUR_PERMANENT_TOKEN
```

Untuk mendapatkan token permanen, gunakan endpoint:

```
GET /api/admin/generate-token
```

## API Endpoints

### 1. Upload File

Endpoint ini digunakan untuk mengupload file secara terpisah dan mendapatkan URL-nya.

```
POST /api/mods/upload
```

#### Headers
- Content-Type: multipart/form-data
- Authorization: Bearer YOUR_PERMANENT_TOKEN

#### Parameters
| Nama | Tipe | Wajib | Deskripsi |
|------|------|-------|-----------|
| file | File | Ya | File yang akan diupload (maksimal 10MB) |
| type | String | Ya | Tipe file: "image", "mod", atau "livery" |

#### Response Success (200 OK)
```json
{
    "success": true,
    "message": "File berhasil diupload",
    "url": "https://yourdomain.com/storage/mods/images/example.jpg"
}
```

#### Response Error (422 Unprocessable Entity)
```json
{
    "success": false,
    "message": "Validasi gagal",
    "errors": {
        "file": [
            "File wajib diisi"
        ],
        "type": [
            "Tipe file tidak valid"
        ]
    }
}
```

### 2. Tambah Mod Baru

Endpoint ini digunakan untuk menambahkan mod baru dengan opsi upload file atau menggunakan URL.

```
POST /api/mods/store
```

#### Headers
- Content-Type: multipart/form-data
- Authorization: Bearer YOUR_PERMANENT_TOKEN

#### Parameters
| Nama | Tipe | Wajib | Deskripsi |
|------|------|-------|-----------|
| category | String | Ya | Kategori mod |
| title | String | Ya | Judul mod |
| image | String | Ya* | URL gambar mod (wajib jika image_file tidak disertakan) |
| link_mods | String | Tidak | URL file mod (opsional jika mod_file disertakan) |
| link_livery | String | Tidak | URL file livery (opsional jika livery_file disertakan) |
| image_file | File | Tidak | File gambar mod (maksimal 5MB, format: jpeg, png, jpg, gif) |
| mod_file | File | Tidak | File mod (maksimal 10MB, format: zip, bussidmod, bussidvehicle) |
| livery_file | File | Tidak | File livery (maksimal 10MB, format: zip, png, jpg, jpeg) |

*Catatan: Anda dapat menyertakan URL atau file. Jika keduanya disertakan, file yang diupload akan diutamakan.

#### Response Success (201 Created)
```json
{
    "success": true,
    "message": "Mod berhasil ditambahkan",
    "data": {
        "id": 1,
        "category": "Bus",
        "title": "Contoh Mod Bus",
        "image": "https://yourdomain.com/storage/mods/images/bus.jpg",
        "link_mods": "https://yourdomain.com/storage/mods/files/bus.bussidmod",
        "link_livery": "https://yourdomain.com/storage/mods/livery/bus_livery.png",
        "likes": 0,
        "created_at": "2023-06-15T10:00:00.000000Z",
        "updated_at": "2023-06-15T10:00:00.000000Z"
    }
}
```

#### Response Error (422 Unprocessable Entity)
```json
{
    "success": false,
    "message": "Validasi gagal",
    "errors": {
        "category": [
            "Kategori wajib diisi"
        ],
        "title": [
            "Judul wajib diisi"
        ]
    }
}
```

## Contoh Penggunaan

### 1. Upload Gambar (JavaScript Fetch API)

```javascript
// Upload gambar
const uploadImage = async (imageFile) => {
    const formData = new FormData();
    formData.append('file', imageFile);
    formData.append('type', 'image');
    
    const response = await fetch('https://yourdomain.com/api/mods/upload', {
        method: 'POST',
        headers: {
            'Authorization': 'Bearer YOUR_PERMANENT_TOKEN'
        },
        body: formData
    });
    
    const result = await response.json();
    return result.url; // URL gambar yang diupload
};
```

### 2. Tambah Mod Baru dengan File (JavaScript Fetch API)

```javascript
// Tambah mod baru dengan file
const addNewMod = async (category, title, imageFile, modFile, liveryFile) => {
    const formData = new FormData();
    formData.append('category', category);
    formData.append('title', title);
    
    // Upload file-file
    if (imageFile) {
        formData.append('image_file', imageFile);
    }
    
    if (modFile) {
        formData.append('mod_file', modFile);
    }
    
    if (liveryFile) {
        formData.append('livery_file', liveryFile);
    }
    
    const response = await fetch('https://yourdomain.com/api/mods/store', {
        method: 'POST',
        headers: {
            'Authorization': 'Bearer YOUR_PERMANENT_TOKEN'
        },
        body: formData
    });
    
    return await response.json();
};
```

### 3. Tambah Mod Baru dengan URL (JavaScript Fetch API)

```javascript
// Tambah mod baru dengan URL
const addNewModWithUrl = async (category, title, imageUrl, modUrl, liveryUrl) => {
    const formData = new FormData();
    formData.append('category', category);
    formData.append('title', title);
    formData.append('image', imageUrl);
    
    if (modUrl) {
        formData.append('link_mods', modUrl);
    }
    
    if (liveryUrl) {
        formData.append('link_livery', liveryUrl);
    }
    
    const response = await fetch('https://yourdomain.com/api/mods/store', {
        method: 'POST',
        headers: {
            'Authorization': 'Bearer YOUR_PERMANENT_TOKEN'
        },
        body: formData
    });
    
    return await response.json();
};
```

### 4. Tambah Mod Baru (Contoh dengan PHP)

```php
<?php
// Contoh penggunaan dengan PHP dan cURL

// Fungsi untuk menambahkan mod baru
function addNewMod($token, $category, $title, $imagePath = null, $modPath = null, $liveryPath = null, $imageUrl = null, $modUrl = null, $liveryUrl = null) {
    $curl = curl_init();
    
    $postFields = [
        'category' => $category,
        'title' => $title,
    ];
    
    // Tambahkan URL jika disediakan
    if ($imageUrl) $postFields['image'] = $imageUrl;
    if ($modUrl) $postFields['link_mods'] = $modUrl;
    if ($liveryUrl) $postFields['link_livery'] = $liveryUrl;
    
    // Tambahkan file jika disediakan
    if ($imagePath) $postFields['image_file'] = new CURLFile($imagePath);
    if ($modPath) $postFields['mod_file'] = new CURLFile($modPath);
    if ($liveryPath) $postFields['livery_file'] = new CURLFile($liveryPath);
    
    curl_setopt_array($curl, [
        CURLOPT_URL => 'https://yourdomain.com/api/mods/store',
        CURLOPT_RETURNTRANSFER => true,
        CURLOPT_ENCODING => '',
        CURLOPT_MAXREDIRS => 10,
        CURLOPT_TIMEOUT => 0,
        CURLOPT_FOLLOWLOCATION => true,
        CURLOPT_HTTP_VERSION => CURL_HTTP_VERSION_1_1,
        CURLOPT_CUSTOMREQUEST => 'POST',
        CURLOPT_POSTFIELDS => $postFields,
        CURLOPT_HTTPHEADER => [
            'Authorization: Bearer ' . $token
        ],
    ]);
    
    $response = curl_exec($curl);
    curl_close($curl);
    
    return json_decode($response, true);
}

// Contoh penggunaan
$token = 'YOUR_PERMANENT_TOKEN';
$result = addNewMod(
    $token,
    'Bus',
    'Bus Baru',
    '/path/to/image.jpg',  // Path file gambar
    '/path/to/mod.bussidmod',  // Path file mod
    '/path/to/livery.png'  // Path file livery
);

print_r($result);
```

## Informasi Tambahan

1. Semua file yang diupload akan disimpan di direktori:
   - Gambar: `storage/app/public/mods/images/`
   - Mod: `storage/app/public/mods/files/`
   - Livery: `storage/app/public/mods/livery/`

2. Pastikan direktori `storage/app/public` telah terhubung dengan symlink ke `public/storage` menggunakan perintah:
   ```
   php artisan storage:link
   ```

3. Ukuran maksimum file:
   - Gambar:
   - Mod: 
   - Livery: 

4. Format file yang didukung:
   - Gambar: jpeg, png, jpg, gif
   - Mod: zip, bussidmod, bussidvehicle
   - Livery: zip, png, jpg, jpeg 