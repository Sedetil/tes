# IzinController Documentation

## Overview
`IzinController` adalah controller yang menangani operasi CRUD (Create, Read, Update, Delete) untuk sistem perizinan santri di Pondok Darussalamah. Controller ini mengimplementasikan role-based access control untuk memastikan setiap user hanya dapat mengakses data sesuai dengan perannya.

## Namespace & Dependencies
```php
namespace App\Http\Controllers;

use App\Http\Requests\StoreIzinRequest;
use App\Http\Requests\UpdateIzinRequest;
use App\Models\Izin;
use Illuminate\Support\Facades\Auth;
```

## Role-based Access Control

Controller ini mengimplementasikan 4 level akses berdasarkan role:

1. **Admin**: Akses penuh ke semua data izin
2. **Mustahiq** (Pembimbing): Akses ke izin santri bimbingannya
3. **Wali** (Orang Tua): Akses ke izin anak santrinya
4. **Satpam**: Akses read-only untuk monitoring

## Methods

### Helper Methods

#### `getUserOrUnauthorized()`
```php
private function getUserOrUnauthorized()
```
**Purpose**: Mengambil user yang sedang login dan memuat relasi yang diperlukan.

**Returns**: 
- `User` object dengan relasi `mustahiq`, `wali`, `satpam` jika authenticated
- `JsonResponse` dengan status 401 jika tidak authenticated

**Usage**: Digunakan oleh semua API methods untuk validasi authentication.

#### `izinWithRelations()`
```php
private function izinWithRelations()
```
**Purpose**: Mendefisikan relasi yang dimuat saat query Izin.

**Returns**: Array berisi `['santri.mustahiq.user', 'santri.wali.user']`

**Usage**: Memastikan data relasi tersedia untuk response API.

### Web Methods (Empty Stubs)

#### Standard Resource Methods
```php
public function index() {}
public function create() {}
public function store(StoreIzinRequest $request) {}
public function show(Izin $izin) {}
public function edit(Izin $izin) {}
public function update(UpdateIzinRequest $request, Izin $izin) {}
public function destroy(Izin $izin) {}
```

**Note**: Method-method ini kosong karena aplikasi menggunakan Filament untuk web interface dan API untuk mobile/frontend interaction.

### API Methods

#### `apiIndex()`
```php
public function apiIndex()
```

**Purpose**: Mengambil daftar izin sesuai dengan role user.

**Authentication**: Required

**Authorization**:
- **Admin/Satpam**: Semua data izin
- **Mustahiq**: Izin dari santri bimbingannya
- **Wali**: Izin dari anak santrinya

**Response**:
```json
{
    "success": true,
    "data": [
        {
            "id": 1,
            "santri_id": 1,
            "keperluan": "Pulang kampung",
            "tanggal_keluar": "2025-07-28",
            "tanggal_kembali": "2025-07-30",
            "status": "pending",
            "santri": {
                "id": 1,
                "nama": "Ahmad Santri",
                "mustahiq": {...},
                "wali": {...}
            }
        }
    ],
    "message": "Data izin berhasil diambil"
}
```

**Error Responses**:
- `401`: User tidak authenticated
- `403`: Role tidak dikenali
- `404`: Data mustahiq/wali tidak ditemukan
- `500`: Server error

#### `apiStore(StoreIzinRequest $request)`
```php
public function apiStore(StoreIzinRequest $request)
```

**Purpose**: Membuat permohonan izin baru.

**Authentication**: Required

**Authorization**: Admin, Mustahiq, atau Wali

**Request Body**:
```json
{
    "santri_id": 1,
    "keperluan": "Pulang kampung",
    "tanggal_keluar": "2025-07-28",
    "tanggal_kembali": "2025-07-30",
    "keterangan": "Menghadiri acara keluarga"
}
```

**Response**:
```json
{
    "success": true,
    "data": {
        "id": 1,
        "santri_id": 1,
        "keperluan": "Pulang kampung",
        "tanggal_keluar": "2025-07-28",
        "tanggal_kembali": "2025-07-30",
        "status": "pending",
        "santri": {...}
    },
    "message": "Permohonan izin berhasil dibuat"
}
```

**Error Responses**:
- `401`: User tidak authenticated
- `403`: User tidak memiliki permission
- `422`: Validation error
- `500`: Server error

#### `apiShow($id)`
```php
public function apiShow($id)
```

**Purpose**: Mengambil detail izin berdasarkan ID.

**Authentication**: Required

**Authorization**: 
- **Admin/Satpam**: Semua izin
- **Mustahiq**: Hanya izin santri bimbingannya
- **Wali**: Hanya izin anak santrinya

**Parameters**:
- `$id` (integer): ID izin yang akan diambil

**Response**:
```json
{
    "success": true,
    "data": {
        "id": 1,
        "santri_id": 1,
        "keperluan": "Pulang kampung",
        "tanggal_keluar": "2025-07-28",
        "tanggal_kembali": "2025-07-30",
        "status": "approved",
        "santri": {
            "id": 1,
            "nama": "Ahmad Santri",
            "mustahiq": {...},
            "wali": {...}
        }
    },
    "message": "Data izin berhasil diambil"
}
```

**Error Responses**:
- `401`: User tidak authenticated
- `403`: User tidak memiliki akses ke data ini
- `404`: Data izin tidak ditemukan
- `500`: Server error

#### `apiUpdateStatus($id, UpdateIzinRequest $request)`
```php
public function apiUpdateStatus($id, UpdateIzinRequest $request)
```

**Purpose**: Mengubah status izin (approve/reject).

**Authentication**: Required

**Authorization**: Admin atau Mustahiq

**Parameters**:
- `$id` (integer): ID izin yang akan diupdate

**Request Body**:
```json
{
    "status": "approved",
    "keterangan": "Izin disetujui"
}
```

**Response**:
```json
{
    "success": true,
    "data": {
        "id": 1,
        "status": "approved",
        "keterangan": "Izin disetujui",
        "santri": {...}
    },
    "message": "Status izin berhasil diubah"
}
```

**Error Responses**:
- `401`: User tidak authenticated
- `403`: User tidak memiliki permission untuk mengubah status
- `404`: Data izin tidak ditemukan
- `422`: Validation error
- `500`: Server error

## Request Validation

### StoreIzinRequest
Validasi untuk membuat izin baru:
- `santri_id`: required, exists in santris table
- `keperluan`: required, string
- `tanggal_keluar`: required, date
- `tanggal_kembali`: required, date, after tanggal_keluar
- `keterangan`: optional, string

### UpdateIzinRequest  
Validasi untuk update status izin:
- `status`: required, in:pending,approved,rejected
- `keterangan`: optional, string

## Error Handling

Semua method API menggunakan try-catch block untuk menangani error:

```php
try {
    // Logic here
} catch (\Exception $e) {
    return response()->json([
        'success' => false,
        'message' => 'Terjadi kesalahan: ' . $e->getMessage(),
    ], 500);
}
```

## Security Features

1. **Authentication Check**: Semua API method memeriksa apakah user authenticated
2. **Role-based Authorization**: Setiap role hanya dapat mengakses data yang sesuai
3. **Data Ownership**: Mustahiq dan Wali hanya dapat mengakses data santri mereka
4. **Request Validation**: Menggunakan Form Request untuk validasi input
5. **Relationship Loading**: Memuat relasi yang diperlukan untuk authorization check

## API Routes

```php
// routes/api.php
Route::get('/izin', [IzinController::class, 'apiIndex']);
Route::post('/izin', [IzinController::class, 'apiStore']);
Route::get('/izin/{id}', [IzinController::class, 'apiShow']);
Route::put('/izin/{id}/status', [IzinController::class, 'apiUpdateStatus']);
```

## Usage Examples

### Get All Izin (Admin)
```bash
curl -X GET "http://localhost:8000/api/izin" \
  -H "Authorization: Bearer {token}" \
  -H "Accept: application/json"
```

### Create New Izin
```bash
curl -X POST "http://localhost:8000/api/izin" \
  -H "Authorization: Bearer {token}" \
  -H "Content-Type: application/json" \
  -d '{
    "santri_id": 1,
    "keperluan": "Pulang kampung",
    "tanggal_keluar": "2025-07-28",
    "tanggal_kembali": "2025-07-30"
  }'
```

### Update Izin Status
```bash
curl -X PUT "http://localhost:8000/api/izin/1/status" \
  -H "Authorization: Bearer {token}" \
  -H "Content-Type: application/json" \
  -d '{
    "status": "approved",
    "keterangan": "Izin disetujui"
  }'
```

## Integration with Filament

Web interface untuk CRUD izin dihandle oleh Filament Admin Panel di `/admin/izins`. Controller ini fokus pada API endpoints untuk mobile app atau frontend JavaScript.

## Notes

- Controller menggunakan relationship-based role detection (bukan field 'role')
- Semua response menggunakan format JSON konsisten
- Error messages dalam Bahasa Indonesia untuk user experience yang lebih baik
- Implement proper HTTP status codes sesuai REST API standards
