# 📚 API Documentation - Sistem Perizinan Pondok Darussalamah

## 🌐 Base URL
```
https://project.linier.my.id/api
```

## 🔐 Authentication
Sistem menggunakan **Laravel Sanctum** untuk authentication. Setelah login, gunakan Bearer token untuk mengakses protected endpoints.

### Headers Required
```
Content-Type: application/json
Accept: application/json
Authorization: Bearer {your_token}  // Untuk protected routes
```

---

## 📋 **1. AUTHENTICATION ENDPOINTS**

### **Login**
```http
POST https://project.linier.my.id/api/login
```

**Request Body:**
```json
{
    "no_hp": "08123456789",
    "password": "password"
}
```

**Response Success:**
```json
{
    "success": true,
    "data": {
        "user": {
            "id": 1,
            "name": "Pak Budi",
            "email": "wali@example.com",
            "role": "wali"
        },
        "token": "1|abcdef123456789..."
    },
    "message": "Login berhasil"
}
```

**Response Error:**
```json
{
    "success": false,
    "message": "Nomor HP atau password salah",
    "errors": {
        "email": ["The provided credentials are incorrect."]
    }
}
```

---

### **Register**
```http
POST https://project.linier.my.id/api/register
```

**Request Body:**
```json
{
    "name": "Pak Ahmad",
    "no_hp": "08121211212",
    "password": "password123",
    "password_confirmation": "password123"
}
```

**Response Success:**
```json
{
    "success": true,
    "data": {
        "user": {
            "id": 2,
            "name": "Pak Ahmad",
            "no_hp": "08121211212"
        },
        "token": "2|xyz789456123..."
    },
    "message": "Registrasi berhasil"
}
```

---

### **Get Current User**
```http
GET https://project.linier.my.id/api/user
```

**Headers:**
```
Authorization: Bearer {your_token}
```

**Response:**
```json
{
    "success": true,
    "data": {
        "id": 1,
        "name": "Pak Budi",
        "email": "wali@example.com",
        "role": "wali",
        "wali": {
            "id": 1,
            "alamat": "Jl. Merdeka No. 123",
            "no_hp": "08123456789"
        }
    }
}
```

---

### **Logout**
```http
POST https://project.linier.my.id/api/logout
```

**Headers:**
```
Authorization: Bearer {your_token}
```

**Response:**
```json
{
    "success": true,
    "message": "Logout berhasil"
}
```

---

## 📝 **2. IZIN MANAGEMENT ENDPOINTS**

### **Get All Izin (Role-based)**
```http
GET https://project.linier.my.id/api/izin
```

**Headers:**
```
Authorization: Bearer {your_token}
```

**Response (Wali):**
```json
{
    "success": true,
    "data": [
        {
            "id": 1,
            "santri_id": 1,
            "keperluan": "Pulang kampung",
            "tanggal_keluar": "2025-07-28",
            "sampai_tanggal": "2025-07-30",
            "status": "approved",
            "created_at": "2025-07-27T10:00:00Z",
            "santri": {
                "id": 1,
                "nama": "Ahmad Santri",
                "wali": {
                    "id": 1,
                    "user": {
                        "name": "Pak Budi"
                    }
                },
                "mustahiq": {
                    "id": 1,
                    "user": {
                        "name": "Ustadz Ali"
                    }
                }
            }
        }
    ],
    "message": "Data izin anak asuh berhasil diambil"
}
```

**Access Control:**
- **Wali**: Hanya izin santri yang menjadi anak asuhnya
- **Mustahiq**: Hanya izin santri di bawah binaannya
- **Satpam**: Semua izin
- **Admin**: Semua izin

---

### **Create New Izin**
```http
POST https://project.linier.my.id/api/create-izin
```

**Headers:**
```
Authorization: Bearer {your_token}
```

**Request Body:**
```json
{
    "santri_id": 1,
    "keperluan": "Berobat ke rumah sakit",
    "tanggal_keluar": "2025-07-29",
    "sampai_tanggal": "2025-07-29",
    "keterangan": "Kontrol rutin ke dokter"
}
```

**Response Success:**
```json
{
    "success": true,
    "data": {
        "id": 2,
        "santri_id": 1,
        "keperluan": "Berobat ke rumah sakit",
        "tanggal_keluar": "2025-07-29",
        "sampai_tanggal": "2025-07-29",
        "status": "pending",
        "keterangan": "Kontrol rutin ke dokter",
        "created_at": "2025-07-28T15:30:00Z"
    },
    "message": "Permohonan izin berhasil dibuat"
}
```

**Validation Rules:**
- `santri_id`: required, exists in santris table
- `keperluan`: required, string, max 255 chars
- `tanggal_keluar`: required, date, today or future
- `sampai_tanggal`: required, date, >= tanggal_keluar
- `keterangan`: optional, string

---

### **Get Izin Detail**
```http
GET https://project.linier.my.id/api/izin/{id}
```

**Example:**
```http
GET https://project.linier.my.id/api/izin/1
```

**Headers:**
```
Authorization: Bearer {your_token}
```

**Response:**
```json
{
    "success": true,
    "data": {
        "id": 1,
        "santri_id": 1,
        "keperluan": "Pulang kampung",
        "tanggal_keluar": "2025-07-28",
        "sampai_tanggal": "2025-07-30",
        "status": "picked_up",
        "pickup_time": "2025-07-28T10:30:00Z",
        "verified_by": "Satpam Ahmad",
        "verification_notes": "Identitas wali sesuai dengan foto database",
        "santri": {
            "id": 1,
            "nama": "Ahmad Santri",
            "wali": {
                "id": 1,
                "alamat": "Jl. Merdeka No. 123",
                "user": {
                    "name": "Pak Budi",
                    "no_hp": "08123456789"
                }
            }
        }
    },
    "message": "Data izin berhasil diambil"
}
```

---

## 🛡️ **3. SATPAM VERIFICATION ENDPOINTS**

### **Get Pickup Data (Satpam Only)**
```http
GET https://project.linier.my.id/api/izin/{id}/pickup-data
```

**Example:**
```http
GET https://project.linier.my.id/api/izin/1/pickup-data
```

**Headers:**
```
Authorization: Bearer {satpam_token}
```

**Purpose**: Ambil data wali dan foto database untuk verifikasi manual

**Response:**
```json
{
    "success": true,
    "data": {
        "izin_id": 1,
        "tanggal_keluar": "2025-07-28",
        "keperluan": "Pulang kampung",
        "santri": {
            "nama": "Ahmad Santri",
            "wali": {
                "id": 1,
                "user": {
                    "name": "Pak Budi",
                    "no_hp": "08123456789"
                },
                "alamat": "Jl. Merdeka No. 123",
                "image_url": [
                    "https://project.linier.my.id/storage/wali/pak_budi_1.jpg",
                    "https://project.linier.my.id/storage/wali/pak_budi_2.jpg",
                    "https://project.linier.my.id/storage/wali/pak_budi_3.jpg"
                ]
            }
        }
    },
    "message": "Data pickup berhasil diambil"
}
```

**Requirements:**
- Hanya accessible oleh user dengan role Satpam
- Izin harus berstatus `approved`
- Tanggal keluar harus hari ini

---

### **Pickup Verification (Manual by Satpam)**
```http
POST https://project.linier.my.id/api/izin/{id}/pickup-verification
```

**Example:**
```http
POST https://project.linier.my.id/api/izin/1/pickup-verification
```

**Headers:**
```
Authorization: Bearer {satpam_token}
```

**Purpose**: Submit hasil verifikasi identitas manual oleh satpam

**Request Body (Identity Verified):**
```json
{
    "identity_verified": true,
    "wali_live_photo": "data:image/jpeg;base64,/9j/4AAQSkZJRgABAQ...",
    "santri_pickup_photo": "data:image/jpeg;base64,/9j/4AAQSkZJRgABAQ...",
    "verification_notes": "Identitas wali sesuai dengan foto database foto ke-2, santri dalam kondisi baik"
}
```

**Request Body (Identity NOT Verified):**
```json
{
    "identity_verified": false,
    "wali_live_photo": "data:image/jpeg;base64,/9j/4AAQSkZJRgABAQ...",
    "rejection_reason": "Identitas wali tidak sesuai dengan foto database, wajah berbeda"
}
```

**Response Success (Approved):**
```json
{
    "success": true,
    "data": {
        "izin_id": 1,
        "status": "picked_up",
        "pickup_time": "2025-07-28T10:30:00Z",
        "verified_by": "Satpam Ahmad",
        "verification_notes": "Identitas wali sesuai dengan foto database foto ke-2, santri dalam kondisi baik"
    },
    "message": "Verifikasi pickup berhasil"
}
```

**Response Failed (Rejected):**
```json
{
    "success": false,
    "data": {
        "izin_id": 1,
        "status": "rejected",
        "rejection_time": "2025-07-28T10:30:00Z",
        "rejected_by": "Satpam Ahmad",
        "rejection_reason": "Identitas wali tidak sesuai dengan foto database, wajah berbeda"
    },
    "message": "Verifikasi gagal - izin ditolak"
}
```

**Validation Rules:**
- `identity_verified`: required, boolean
- `wali_live_photo`: required, base64 image string
- `santri_pickup_photo`: optional (jika identity_verified=true), base64 image string
- `verification_notes`: optional, string
- `rejection_reason`: required if identity_verified=false, string

---

### **Return Confirmation (Manual by Satpam)**
```http
POST https://project.linier.my.id/api/izin/{id}/return-confirmation
```

**Example:**
```http
POST https://project.linier.my.id/api/izin/1/return-confirmation
```

**Headers:**
```
Authorization: Bearer {satpam_token}
```

**Purpose**: Konfirmasi manual return santri ke pondok

**Request Body:**
```json
{
    "santri_return_photo": "data:image/jpeg;base64,/9j/4AAQSkZJRgABAQ...",
    "return_notes": "Santri kembali dalam kondisi sehat dan tepat waktu"
}
```

**Response:**
```json
{
    "success": true,
    "data": {
        "izin_id": 1,
        "status": "completed",
        "return_time": "2025-07-28T18:00:00Z",
        "confirmed_by": "Satpam Ahmad"
    },
    "message": "Return confirmation berhasil"
}
```

**Requirements:**
- Hanya accessible oleh user dengan role Satpam
- Izin harus berstatus `picked_up`

**Validation Rules:**
- `santri_return_photo`: required, base64 image string
- `return_notes`: optional, string

---

## 📊 **4. DASHBOARD & STATISTICS**

### **Dashboard Statistics (Role-based)**
```http
GET https://project.linier.my.id/api/dashboard/stats
```

**Headers:**
```
Authorization: Bearer {your_token}
```

**Response (Satpam):**
```json
{
    "success": true,
    "data": {
        "today_pickups": 5,
        "today_rejections": 1,
        "currently_out": 12,
        "overdue_returns": 2,
        "pending_verifications": 3
    }
}
```

**Response (Wali):**
```json
{
    "success": true,
    "data": {
        "total_santri": 2,
        "active_izin": 1,
        "pending_izin": 0,
        "this_month_izin": 3
    }
}
```

**Response (Mustahiq):**
```json
{
    "success": true,
    "data": {
        "total_santri": 15,
        "pending_reviews": 2,
        "approved_today": 3,
        "this_month_reviews": 12
    }
}
```

---

### **Today's Pickup List (Satpam Only)**
```http
GET https://project.linier.my.id/api/izin/today-pickups
```

**Headers:**
```
Authorization: Bearer {satpam_token}
```

**Response:**
```json
{
    "success": true,
    "data": [
        {
            "id": 1,
            "santri": {
                "nama": "Ahmad Santri",
                "wali": {
                    "user": {"name": "Pak Budi"}
                }
            },
            "keperluan": "Pulang kampung",
            "status": "approved",
            "tanggal_keluar": "2025-07-28"
        },
        {
            "id": 2,
            "santri": {
                "nama": "Fatimah Santri",
                "wali": {
                    "user": {"name": "Bu Siti"}
                }
            },
            "keperluan": "Berobat",
            "status": "approved",
            "tanggal_keluar": "2025-07-28"
        }
    ],
    "message": "Data pickup hari ini berhasil diambil"
}
```

---

### **Currently Out List (Satpam Only)**
```http
GET https://project.linier.my.id/api/izin/currently-out
```

**Headers:**
```
Authorization: Bearer {satpam_token}
```

**Response:**
```json
{
    "success": true,
    "data": [
        {
            "id": 1,
            "santri": {
                "nama": "Ahmad Santri"
            },
            "keperluan": "Pulang kampung",
            "status": "picked_up",
            "pickup_time": "2025-07-28T10:30:00Z",
            "sampai_tanggal": "2025-07-30",
            "is_overdue": false
        },
        {
            "id": 3,
            "santri": {
                "nama": "Yusuf Santri"
            },
            "keperluan": "Acara keluarga",
            "status": "picked_up",
            "pickup_time": "2025-07-26T09:00:00Z",
            "sampai_tanggal": "2025-07-27",
            "is_overdue": true
        }
    ],
    "message": "Data santri yang sedang keluar berhasil diambil"
}
```

---

## 🚨 **5. ERROR RESPONSES**

### **Common Error Formats:**

**401 Unauthorized:**
```json
{
    "success": false,
    "message": "Unauthenticated."
}
```

**403 Forbidden:**
```json
{
    "success": false,
    "message": "Unauthorized"
}
```

**404 Not Found:**
```json
{
    "success": false,
    "message": "Data izin tidak ditemukan"
}
```

**422 Validation Error:**
```json
{
    "success": false,
    "message": "The given data was invalid.",
    "errors": {
        "santri_id": ["The santri id field is required."],
        "tanggal_keluar": ["The tanggal keluar must be a date after or equal to today."]
    }
}
```

**500 Server Error:**
```json
{
    "success": false,
    "message": "Terjadi kesalahan: Database connection failed"
}
```

---

## 🔄 **6. STATUS FLOW & BUSINESS LOGIC**

### **Izin Status Flow:**
```
pending → approved → picked_up → completed
    ↓         ↓
rejected   rejected
```

### **Status Descriptions:**
- **`pending`**: Izin baru diajukan, menunggu review mustahiq
- **`approved`**: Disetujui mustahiq, menunggu pickup
- **`picked_up`**: Santri sudah di-pickup wali (verified by satpam)
- **`rejected`**: Ditolak (bisa di level mustahiq atau satpam)
- **`completed`**: Santri sudah kembali ke pondok

### **Role Permissions:**
- **Wali**: Create izin, view own izin
- **Mustahiq**: Review & approve/reject izin
- **Satpam**: Pickup verification, return confirmation
- **Admin**: Full access to all data

---

## 📱 **7. MOBILE APP INTEGRATION EXAMPLES**

### **Login Flow:**
```javascript
// Login request
const loginResponse = await fetch('https://project.linier.my.id/api/login', {
    method: 'POST',
    headers: {
        'Content-Type': 'application/json',
        'Accept': 'application/json'
    },
    body: JSON.stringify({
        email: 'satpam@example.com',
        password: 'password123'
    })
});

const loginData = await loginResponse.json();
const token = loginData.data.token;

// Store token for subsequent requests
localStorage.setItem('auth_token', token);
```

### **Pickup Verification Flow:**
```javascript
// 1. Get pickup data
const pickupDataResponse = await fetch('https://project.linier.my.id/api/izin/1/pickup-data', {
    headers: {
        'Authorization': `Bearer ${token}`,
        'Accept': 'application/json'
    }
});

const pickupData = await pickupDataResponse.json();
// Display wali photos for manual comparison

// 2. Capture live photo and submit verification
const verificationResponse = await fetch('https://project.linier.my.id/api/izin/1/pickup-verification', {
    method: 'POST',
    headers: {
        'Authorization': `Bearer ${token}`,
        'Content-Type': 'application/json',
        'Accept': 'application/json'
    },
    body: JSON.stringify({
        identity_verified: true, // Manual decision by satpam
        wali_live_photo: base64PhotoData,
        santri_pickup_photo: base64SantriPhoto,
        verification_notes: 'Identitas sesuai dengan database'
    })
});
```

---

## 🛠️ **8. TESTING WITH CURL**

### **Login Test:**
```bash
curl -X POST https://project.linier.my.id/api/login \
  -H "Content-Type: application/json" \
  -H "Accept: application/json" \
  -d '{
    "email": "satpam@example.com",
    "password": "password123"
  }'
```

### **Get Pickup Data Test:**
```bash
curl -X GET https://project.linier.my.id/api/izin/1/pickup-data \
  -H "Authorization: Bearer YOUR_TOKEN_HERE" \
  -H "Accept: application/json"
```

### **Pickup Verification Test:**
```bash
curl -X POST https://project.linier.my.id/api/izin/1/pickup-verification \
  -H "Authorization: Bearer YOUR_TOKEN_HERE" \
  -H "Content-Type: application/json" \
  -H "Accept: application/json" \
  -d '{
    "identity_verified": true,
    "wali_live_photo": "data:image/jpeg;base64,/9j/4AAQ...",
    "santri_pickup_photo": "data:image/jpeg;base64,/9j/4AAQ...",
    "verification_notes": "Identitas wali sesuai dengan foto database"
  }'
```

---

## 🎯 **9. IMPLEMENTATION NOTES**

### **Image Handling:**
- Semua foto dikirim sebagai Base64 encoded string
- Server akan convert dan simpan ke storage
- URL foto dikembalikan sebagai full URL dengan domain

### **Manual Verification Process:**
1. **Tidak ada AI/ML**: Semua keputusan dibuat manual oleh satpam
2. **Side-by-side comparison**: Mobile app menampilkan foto database vs live photo
3. **Human decision**: Satpam memutuskan SESUAI/TIDAK SESUAI
4. **Audit trail**: Semua keputusan tercatat dengan timestamp dan catatan

### **Security Features:**
- Token-based authentication (Laravel Sanctum)
- Role-based access control
- Time-based validation (pickup hanya di tanggal yang sesuai)
- Photo evidence untuk semua transaksi
- Complete audit trail

---

**📞 Support Contact:**
- Email: support@project.linier.my.id
- Documentation: https://project.linier.my.id/docs
- API Status: https://project.linier.my.id/api/health
