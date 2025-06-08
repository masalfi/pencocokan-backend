# Metode Pencocokan Package name 
- Metode ini mengandalkan aplikasi Flutter untuk mengirimkan nama paketnya ke backend setiap kali membuat request. Backend kemudian akan memvalidasinya.
  
1. Tambahkan package package_info_plus:
   # pubspec.yaml

```
  dependencies:
  package_info_plus: ^8.0.0 # Cek versi terbaru
```
2. Kirim Package Name di Header Request:

```
import 'package:dio/dio.dart'; // atau package http
import 'package:package_info_plus/package_info_plus.dart';

Future<void> fetchDataFromLaravel() async {
  PackageInfo packageInfo = await PackageInfo.fromPlatform();
  String packageName = packageInfo.packageName;

  var dio = Dio();
  try {
    Response response = await dio.get(
      'https://api.domain-anda.com/data-penting',
      options: Options(
        headers: {
          'Accept': 'application/json',
          'X-Package-Name': packageName, // Header kustom kita
        },
      ),
    );
    // proses data...
  } catch (e) {
    // tangani error...
  }
}
```
- Langkah di Backend Laravel:
1. Gunakan Middleware untuk mencegat dan memvalidasi setiap request yang masuk.
  ```
php artisan make:middleware VerifyPackageName
  ```
2. Edit Middleware (app/Http/Middleware/VerifyPackageName.php):
```
   <?php

namespace App\Http\Middleware;

use Closure;
use Illuminate\Http\Request;

class VerifyPackageName
{
    public function handle(Request $request, Closure $next)
    {
        // Ambil header dari request
        $requestPackageName = $request->header('X-Package-Name');

        // Dapatkan package name yang sah dari file .env
        $validPackageName = env('VALID_PACKAGE_NAME');

        // Jika header tidak ada atau tidak cocok, tolak request
        if (!$requestPackageName || $requestPackageName !== $validPackageName) {
            return response()->json(['error' => 'Unauthorized Access.'], 403);
        }

        // Jika cocok, lanjutkan request
        return $next($request);
    }
}
```
3. Tambahkan Package Name yang Sah di .env:
```
   # .env
VALID_PACKAGE_NAME=com.namaperusahaan.namaaplikasi
```
4. Daftarkan Middleware: Terapkan middleware ini pada route API Anda di app/Http/Kernel.php atau langsung di file routes/api.php.
```
// routes/api.php
Route::middleware(['auth:sanctum', \App\Http\Middleware\VerifyPackageName::class])->group(function () {
    Route::get('/data-penting', [DataController::class, 'index']);
    // route-route lain yang butuh proteksi
});
```
   
