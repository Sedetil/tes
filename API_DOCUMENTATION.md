# API Documentation - Telur API

## Base URL
```
http://127.0.0.1:8000/api
```

## Web Dashboard Access

### User Dashboard
```
http://127.0.0.1:8000/dashboard
```
**Features:**
- View available products and stock
- Make purchases (transactions)
- View transaction history
- Manage delivery addresses
- Track shipments with real-time status
- Write and manage product reviews
- Update user profile

### Admin Dashboard
```
http://127.0.0.1:8000/admin
```
**Features:**
- Complete system management interface
- Real-time transaction monitoring
- Product inventory management
- User management and role assignment
- Shipment tracking and status updates
- Transaction status management
- System analytics and reports

**Dashboard Authentication:**
- Use the same login credentials as API
- Automatic role-based interface (admin/user)
- Real-time updates without page refresh
- Mobile-responsive design

## Authentication
API ini menggunakan Laravel Sanctum untuk autentikasi. Setelah login, gunakan token yang diberikan di header:
```
Authorization: Bearer {your_token}
```

## Endpoints

### 1. Authentication

#### Register
```http
POST /api/register
Content-Type: application/json

{
    "name": "John Doe",
    "email": "john@example.com",
    "password": "password123",
    "password_confirmation": "password123"
}
```

#### Login
```http
POST /api/login
Content-Type: application/json

{
    "email": "admin@telur.com",
    "password": "password"
}
```

**Response:**
```json
{
    "success": true,
    "message": "Login successful",
    "data": {
        "user": {
            "id": 1,
            "name": "Admin",
            "email": "admin@telur.com",
            "role": "admin"
        },
        "token": "your_access_token",
        "token_type": "Bearer",
        "permissions": {
            "can_delete_transactions": true,
            "can_manage_users": true,
            "can_manage_products": true,
            "can_update_transaction_status": true,
            "is_admin": true
        }
    }
}
```

#### Logout
```http
POST /api/logout
Authorization: Bearer {token}
```

#### Get User Info
```http
GET /api/me
Authorization: Bearer {token}
```

**Response:**
```json
{
    "success": true,
    "message": "User profile retrieved successfully",
    "data": {
        "user": {
            "id": 1,
            "name": "Admin",
            "email": "admin@telur.com",
            "role": "admin"
        },
        "permissions": {
            "can_delete_transactions": true,
            "can_manage_users": true,
            "can_manage_products": true,
            "can_update_transaction_status": true,
            "is_admin": true
        }
    }
}
```

### 2. Telur Management

#### Get All Telur (Public untuk authenticated users)
```http
GET /api/telur
Authorization: Bearer {token}
```

#### Get Specific Telur
```http
GET /api/telur/{id}
Authorization: Bearer {token}
```

#### Create Telur (Admin Only)
```http
POST /api/telur
Authorization: Bearer {admin_token}
Content-Type: application/json

{
    "jenis_telur": "Telur Ayam Kampung",
    "harga_per_kilo": 35000,
    "stok_kg": 50.5
}
```

#### Update Telur (Admin Only)
```http
PUT /api/telur/{id}
Authorization: Bearer {admin_token}
Content-Type: application/json

{
    "jenis_telur": "Telur Ayam Kampung Premium",
    "harga_per_kilo": 40000,
    "stok_kg": 45.0
}
```

#### Delete Telur (Admin Only)
```http
DELETE /api/telur/{id}
Authorization: Bearer {admin_token}
```

### 3. Payment Management

#### Get Payment Methods (Public)
```http
GET /api/payment-methods
```

**Response:**
```json
{
    "success": true,
    "data": [
        {
            "id": "cod",
            "name": "COD (Bayar di Tempat)",
            "description": "Pembayaran akan dilakukan saat barang diterima",
            "icon": "💵",
            "enabled": true
        },
        {
            "id": "transfer_bank",
            "name": "Transfer Bank",
            "description": "Transfer ke rekening bank yang tersedia",
            "icon": "🏦",
            "enabled": true
        }
    ]
}
```

#### Get Payment Details (Public)
```http
GET /api/payment-details/{method}
```

**Available Methods:** `cod`, `transfer_bank`

**Response for Transfer Bank:**
```json
{
    "success": true,
    "data": {
        "type": "bank_transfer",
        "bank_name": "BNI",
        "account_number": "1234567890",
        "account_name": "PT Telur Ayam Segar",
        "instructions": "Silakan transfer ke rekening berikut dan konfirmasi via WhatsApp",
        "additional_info": "Pastikan nominal transfer sesuai dengan total pesanan",
        "whatsapp": {
            "number": "6285158862314",
            "message_template": "Halo admin, saya sudah transfer pembayaran pesanan saya.\n\n- Nama: {user_name}\n- Email: {user_email}\n- ID Transaksi: #{transaction_id}\n- Produk: {product_name} {quantity}kg\n- Total: Rp{total_amount}\n- Bukti Transfer: [lampiran]"
        },
        "steps": [
            "Transfer sesuai nominal yang tertera",
            "Simpan bukti transfer",
            "Klik tombol konfirmasi WhatsApp",
            "Kirim bukti transfer via WhatsApp",
            "Tunggu konfirmasi dari admin"
        ]
    }
}
```

**Response for COD:**
```json
{
    "success": true,
    "data": {
        "type": "cash_on_delivery",
        "instructions": "Pembayaran akan dilakukan saat pengiriman",
        "additional_info": "Pastikan menyiapkan uang pas sesuai total pesanan",
        "steps": [
            "Pesanan akan diproses",
            "Kurir akan menghubungi Anda",
            "Siapkan uang pas",
            "Bayar saat barang diterima"
        ]
    }
}
```

#### Get WhatsApp Confirmation URL (Public)
```http
POST /api/whatsapp-confirmation
Content-Type: application/json

{
    "transaction_id": 1,
    "customer_name": "John Doe",
    "customer_email": "john@example.com",
    "product_name": "Telur Ayam",
    "quantity": 2.5,
    "total_price": 75000
}
```

**Response:**
```json
{
    "success": true,
    "data": {
        "whatsapp_url": "https://wa.me/6285158862314?text=Halo%20admin%2C%20saya%20sudah%20transfer%20pembayaran%20pesanan%20saya.%0A%0A-%20Nama%3A%20John%20Doe%0A-%20Email%3A%20john%40example.com%0A-%20ID%20Transaksi%3A%20%231%0A-%20Produk%3A%20Telur%20Ayam%202.5kg%0A-%20Total%3A%20Rp75.000%0A-%20Bukti%20Transfer%3A%20%5Blampiran%5D",
        "whatsapp_number": "6285158862314",
        "message": "Halo admin, saya sudah transfer pembayaran pesanan saya.\n\n- Nama: John Doe\n- Email: john@example.com\n- ID Transaksi: #1\n- Produk: Telur Ayam 2.5kg\n- Total: Rp75.000\n- Bukti Transfer: [lampiran]"
    }
}
```

### 4. File Upload Management

#### Upload Single File
```http
POST /api/upload
Authorization: Bearer {token}
Content-Type: multipart/form-data

file: [binary file]
type: product|payment|review
```

**Response:**
```json
{
    "success": true,
    "message": "File uploaded successfully",
    "data": {
        "filename": "1640995200_abc123.jpg",
        "path": "uploads/product/1640995200_abc123.jpg",
        "url": "http://127.0.0.1:8000/storage/uploads/product/1640995200_abc123.jpg"
    }
}
```

#### Upload Multiple Files
```http
POST /api/upload/multiple
Authorization: Bearer {token}
Content-Type: multipart/form-data

files[]: [binary file 1]
files[]: [binary file 2]
type: product|payment|review
```

**Response:**
```json
{
    "success": true,
    "message": "Files uploaded successfully",
    "data": [
        {
            "filename": "1640995200_abc123.jpg",
            "path": "uploads/review/1640995200_abc123.jpg",
            "url": "http://127.0.0.1:8000/storage/uploads/review/1640995200_abc123.jpg"
        },
        {
            "filename": "1640995201_def456.jpg",
            "path": "uploads/review/1640995201_def456.jpg",
            "url": "http://127.0.0.1:8000/storage/uploads/review/1640995201_def456.jpg"
        }
    ]
}
```

#### Delete File
```http
DELETE /api/upload
Authorization: Bearer {token}
Content-Type: application/json

{
    "path": "uploads/product/1640995200_abc123.jpg"
}
```

**Response:**
```json
{
    "success": true,
    "message": "File deleted successfully"
}
```

**File Upload Rules:**
- Allowed file types: jpeg, png, jpg, gif
- Maximum file size: 2MB per file
- Maximum files per upload: 5 files
- File types:
  - `product`: For product photos
  - `payment`: For payment proof images
  - `review`: For review photos

### 5. Review Management

#### Get Product Reviews
```http
GET /api/telur/{telur_id}/reviews
Authorization: Bearer {token}
```

**Response:**
```json
{
    "success": true,
    "data": {
        "current_page": 1,
        "data": [
            {
                "id": 1,
                "user_id": 2,
                "telur_id": 1,
                "transaksi_id": 5,
                "rating": 5,
                "comment": "Telur sangat segar dan berkualitas!",
                "photos": [
                    "http://127.0.0.1:8000/storage/uploads/review/photo1.jpg",
                    "http://127.0.0.1:8000/storage/uploads/review/photo2.jpg"
                ],
                "is_verified": true,
                "helpful_count": 0,
                "created_at": "2024-01-01T10:00:00.000000Z",
                "user": {
                    "id": 2,
                    "name": "John Doe"
                },
                "transaksi": {
                    "id": 5,
                    "total_harga": 75000
                }
            }
        ],
        "per_page": 10,
        "total": 1
    }
}
```

#### Get User's Reviews
```http
GET /api/reviews/user
Authorization: Bearer {token}
```

#### Get Product Rating Statistics
```http
GET /api/telur/{telur_id}/reviews/stats
Authorization: Bearer {token}
```

**Response:**
```json
{
    "success": true,
    "data": {
        "product": {
            "id": 1,
            "jenis_telur": "Telur Ayam Kampung",
            "harga_per_kilo": 35000,
            "stok_kg": 50.5
        },
        "total_reviews": 15,
        "average_rating": 4.3,
        "rating_distribution": {
            "1": {"count": 0, "percentage": 0},
            "2": {"count": 1, "percentage": 6.7},
            "3": {"count": 2, "percentage": 13.3},
            "4": {"count": 5, "percentage": 33.3},
            "5": {"count": 7, "percentage": 46.7}
        }
    }
}
```

#### Create Review
```http
POST /api/reviews
Authorization: Bearer {token}
Content-Type: application/json

{
    "telur_id": 1,
    "transaksi_id": 5,
    "rating": 5,
    "comment": "Telur sangat segar dan berkualitas!",
    "photos": [
        "http://127.0.0.1:8000/storage/uploads/review/photo1.jpg",
        "http://127.0.0.1:8000/storage/uploads/review/photo2.jpg"
    ]
}
```

**Response:**
```json
{
    "success": true,
    "message": "Review created successfully",
    "data": {
        "id": 1,
        "user_id": 2,
        "telur_id": 1,
        "transaksi_id": 5,
        "rating": 5,
        "comment": "Telur sangat segar dan berkualitas!",
        "photos": [
            "http://127.0.0.1:8000/storage/uploads/review/photo1.jpg"
        ],
        "is_verified": true,
        "created_at": "2024-01-01T10:00:00.000000Z",
        "user": {
            "id": 2,
            "name": "John Doe"
        },
        "telur": {
            "id": 1,
            "jenis_telur": "Telur Ayam Kampung"
        }
    }
}
```

#### Get Specific Review
```http
GET /api/reviews/{id}
Authorization: Bearer {token}
```

#### Update Review (Owner Only)
```http
PUT /api/reviews/{id}
Authorization: Bearer {token}
Content-Type: application/json

{
    "rating": 4,
    "comment": "Updated review comment",
    "photos": [
        "http://127.0.0.1:8000/storage/uploads/review/new_photo.jpg"
    ]
}
```

#### Delete Review (Owner Only)
```http
DELETE /api/reviews/{id}
Authorization: Bearer {token}
```

#### Toggle Helpful Vote
```http
POST /api/reviews/{id}/helpful
Authorization: Bearer {token}
```

#### Report Review
```http
POST /api/reviews/{id}/report
Authorization: Bearer {token}
Content-Type: application/json

{
    "reason": "Inappropriate content"
}
```

**Review Rules:**
- Only users who purchased the product can review
- One review per transaction per product
- Rating must be between 1-5
- Maximum 5 photos per review
- Maximum 1000 characters for comment
- Reviews are auto-verified for purchased products

### 6. Address Management

#### Get User Addresses
```http
GET /api/addresses
Authorization: Bearer {token}
```

**Response:**
```json
{
    "success": true,
    "data": [
        {
            "id": 1,
            "user_id": 2,
            "label": "Rumah",
            "recipient_name": "John Doe",
            "phone": "081234567890",
            "address": "Jl. Merdeka No. 123, RT 01/RW 02",
            "city": "Jakarta Pusat",
            "province": "DKI Jakarta",
            "postal_code": "10110",
            "latitude": -6.200000,
            "longitude": 106.816666,
            "is_default": true,
            "created_at": "2024-01-01T10:00:00.000000Z",
            "updated_at": "2024-01-01T10:00:00.000000Z"
        }
    ]
}
```

#### Create Address
```http
POST /api/addresses
Authorization: Bearer {token}
Content-Type: application/json

{
    "label": "Kantor",
    "recipient_name": "John Doe",
    "phone": "081234567890",
    "address": "Jl. Sudirman No. 456, Lantai 10",
    "city": "Jakarta Selatan",
    "province": "DKI Jakarta",
    "postal_code": "12190",
    "latitude": -6.225000,
    "longitude": 106.800000,
    "is_default": false
}
```

**Response:**
```json
{
    "success": true,
    "message": "Address created successfully",
    "data": {
        "id": 2,
        "user_id": 2,
        "label": "Kantor",
        "recipient_name": "John Doe",
        "phone": "081234567890",
        "address": "Jl. Sudirman No. 456, Lantai 10",
        "city": "Jakarta Selatan",
        "province": "DKI Jakarta",
        "postal_code": "12190",
        "latitude": -6.225000,
        "longitude": 106.800000,
        "is_default": false,
        "created_at": "2024-01-01T10:00:00.000000Z",
        "updated_at": "2024-01-01T10:00:00.000000Z"
    }
}
```

#### Get Specific Address
```http
GET /api/addresses/{id}
Authorization: Bearer {token}
```

#### Update Address
```http
PUT /api/addresses/{id}
Authorization: Bearer {token}
Content-Type: application/json

{
    "label": "Rumah Utama",
    "recipient_name": "John Doe",
    "phone": "081234567890",
    "address": "Jl. Merdeka No. 123, RT 01/RW 02",
    "city": "Jakarta Pusat",
    "province": "DKI Jakarta",
    "postal_code": "10110",
    "latitude": -6.200000,
    "longitude": 106.816666,
    "is_default": true
}
```

#### Delete Address
```http
DELETE /api/addresses/{id}
Authorization: Bearer {token}
```

#### Set Default Address
```http
PATCH /api/addresses/{id}/default
Authorization: Bearer {token}
```

**Response:**
```json
{
    "success": true,
    "message": "Default address updated successfully",
    "data": {
        "id": 1,
        "is_default": true
    }
}
```

#### Calculate Shipping Cost
```http
POST /api/addresses/calculate-shipping
Authorization: Bearer {token}
Content-Type: application/json

{
    "address_id": 1,
    "weight_kg": 2.5,
    "telur_id": 1
}
```

**Response:**
```json
{
    "success": true,
    "data": {
        "shipping_cost": 15000,
        "estimated_delivery": "1-2 hari kerja",
        "courier": "Kurir Internal",
        "distance_km": 12.5
    }
}
```

**Address Rules:**
- Users can have multiple addresses
- Only one address can be set as default
- Setting a new default address will unset the previous one
- Latitude and longitude are optional for GPS coordinates
- Phone number is required for delivery contact
- Postal code must be valid Indonesian postal code format

### 7. Transaksi Management

#### Get Transaksi
- Admin: Melihat semua transaksi
- User: Melihat transaksi miliknya saja
```http
GET /api/transaksi
Authorization: Bearer {token}
```

**Response:**
```json
{
    "success": true,
    "message": "Data transaksi berhasil diambil",
    "user_role": "admin",
    "can_delete": true,
    "data": [
        {
            "id": 1,
            "user_id": 2,
            "telur_id": 1,
            "jumlah_kg": 2.5,
            "total_harga": 75000,
            "metode_pembayaran": "cod",
            "catatan_pembayaran": "Bayar saat barang sampai",
            "status": "pending",
            "created_at": "2024-01-01T10:00:00.000000Z",
            "telur": {
                "id": 1,
                "jenis_telur": "Telur Ayam",
                "harga_per_kilo": 30000,
                "stok_kg": 47.5
            }
        }
    ]
}
```

#### Get Specific Transaksi
```http
GET /api/transaksi/{id}
Authorization: Bearer {token}
```

#### Create Transaksi (Beli Telur)
```http
POST /api/transaksi
Authorization: Bearer {token}
Content-Type: application/json

{
    "telur_id": 1,
    "jumlah_kg": 2.5,
    "metode_pembayaran": "cod",
    "catatan_pembayaran": "Bayar saat barang sampai"
}
```

**Payment Methods**:
- `cod`: Cash on Delivery (Bayar di Tempat)
- `transfer_bank`: Bank Transfer

**Note**: `catatan_pembayaran` is optional

#### Update Status Transaksi (Admin Only)
```http
PATCH /api/transaksi/{id}/status
Authorization: Bearer {admin_token}
Content-Type: application/json

{
  "status": "success"
}
```

**Available Status Values**:
- `pending`: Transaksi menunggu konfirmasi
- `success`: Transaksi berhasil/selesai
- `cancel`: Transaksi dibatalkan

#### Delete Transaksi (Admin Only)
```http
DELETE /api/transaksi/{id}
Authorization: Bearer {admin_token}
```

**Success Response:**
```json
{
    "success": true,
    "message": "Transaksi berhasil dihapus dan stok telur dikembalikan",
    "data": {
        "deleted_transaction_id": 1,
        "restored_stock": 2.5,
        "product_name": "Telur Ayam",
        "new_stock": 50.0
    }
}
```

**Error Responses:**

*Insufficient Permissions (403):*
```json
{
    "success": false,
    "message": "Anda tidak memiliki akses untuk menghapus transaksi",
    "error_code": "INSUFFICIENT_PERMISSIONS",
    "data": {
        "user_role": "user",
        "required_role": "admin",
        "can_delete": false
    }
}
```

*Transaction Not Found (404):*
```json
{
    "success": false,
    "message": "Data transaksi tidak ditemukan",
    "error_code": "TRANSACTION_NOT_FOUND",
    "data": {
        "transaction_id": "1",
        "user_role": "admin"
    }
}
```

*Server Error (500):*
```json
{
    "success": false,
    "message": "Terjadi kesalahan saat menghapus transaksi",
    "error_code": "DELETE_TRANSACTION_FAILED",
    "data": {
        "transaction_id": "1",
        "error_details": "Database connection failed",
        "user_role": "admin"
    }
}
```

### 5. User Management (Admin Only)

#### Get All Users
```http
GET /api/users
Authorization: Bearer {admin_token}
```

#### Get Specific User
```http
GET /api/users/{id}
Authorization: Bearer {admin_token}
```

#### Update User
```http
PUT /api/users/{id}
Authorization: Bearer {admin_token}
Content-Type: application/json

{
    "name": "Updated Name",
    "email": "updated@example.com",
    "role": "admin",
    "password": "newpassword123"
}
```

#### Delete User
```http
DELETE /api/users/{id}
Authorization: Bearer {admin_token}
```

#### Update User Role
```http
PATCH /api/users/{id}/role
Authorization: Bearer {admin_token}
Content-Type: application/json

{
    "role": "admin"
}
```

## Default Users (Seeder Data)

### Admin
- Email: `admin@telur.com`
- Password: `password`
- Role: `admin`

### Regular Users
1. Email: `budi@gmail.com`, Password: `password`
2. Email: `siti@gmail.com`, Password: `password`
3. Email: `ahmad@gmail.com`, Password: `password`

## Sample Data

### Telur Types
1. Telur Ayam - Rp 30,000/kg

## Response Format

### Success Response
```json
{
    "success": true,
    "message": "Success message",
    "data": {},
    "user_role": "admin",
    "can_delete": true
}
```

### Error Response
```json
{
    "success": false,
    "message": "Error message",
    "error_code": "ERROR_CODE",
    "data": {
        "additional_info": "value"
    },
    "errors": {}
}
```

## Error Codes

API menggunakan error codes yang konsisten untuk memudahkan handling di mobile:

- `INSUFFICIENT_PERMISSIONS`: User tidak memiliki permission yang diperlukan
- `TRANSACTION_NOT_FOUND`: Transaksi tidak ditemukan
- `DELETE_TRANSACTION_FAILED`: Gagal menghapus transaksi karena error server
- `VALIDATION_ERROR`: Data input tidak valid
- `UNAUTHORIZED`: Token tidak valid atau expired
- `FORBIDDEN`: Akses ditolak

## Permissions System

API menyediakan informasi permissions yang dapat digunakan untuk:
- Menampilkan/menyembunyikan tombol di UI mobile
- Validasi akses sebelum melakukan request
- Menentukan fitur yang tersedia untuk user

**Available Permissions:**
- `can_delete_transactions`: Dapat menghapus transaksi
- `can_manage_users`: Dapat mengelola user
- `can_manage_products`: Dapat mengelola produk telur
- `can_update_transaction_status`: Dapat mengubah status transaksi
- `is_admin`: User adalah admin

## Mobile Implementation Guidelines

### 1. Authentication Flow
1. Login dengan endpoint `/api/login`
2. Simpan `token` dan `permissions` dari response
3. Gunakan `permissions` untuk mengatur UI (show/hide buttons)
4. Set Authorization header untuk semua request selanjutnya

### 2. Permission-Based UI
```javascript
// Contoh implementasi di mobile
if (user.permissions.can_delete_transactions) {
    showDeleteButton();
} else {
    hideDeleteButton();
}
```

### 3. Error Handling
```javascript
// Handle error berdasarkan error_code
switch (response.error_code) {
    case 'INSUFFICIENT_PERMISSIONS':
        showPermissionDeniedDialog();
        break;
    case 'TRANSACTION_NOT_FOUND':
        showNotFoundMessage();
        break;
    case 'DELETE_TRANSACTION_FAILED':
        showServerErrorMessage();
        break;
}
```

### 4. Payment Management
- Gunakan `/api/payment-methods` untuk mendapatkan metode pembayaran yang tersedia
- Gunakan `/api/payment-details/{method}` untuk mendapatkan detail pembayaran
- Untuk transfer bank, gunakan `/api/whatsapp-confirmation` untuk generate URL WhatsApp
- Implementasikan UI yang menampilkan langkah-langkah pembayaran sesuai metode yang dipilih

### 5. Transaction Management
- Gunakan `user_role` dan `can_delete` dari response `/api/transaksi`
- Untuk delete, pastikan user adalah admin sebelum menampilkan opsi
- Handle success response untuk update UI (refresh list, show success message)

### 6. Best Practices
- Selalu check permissions sebelum menampilkan action buttons
- Gunakan error_code untuk consistent error handling
- Cache permissions data untuk menghindari request berulang
- Implement proper loading states dan error messages

## Database Schema

### Addresses Table
The addresses table stores user delivery addresses with the following structure:

```sql
-- Migration: 2025_07_30_065434_create_addresses_table.php
CREATE TABLE addresses (
    id BIGINT UNSIGNED AUTO_INCREMENT PRIMARY KEY,
    user_id BIGINT UNSIGNED NOT NULL,
    label VARCHAR(50) NULL COMMENT 'Address label like Home, Office, etc',
    recipient_name VARCHAR(100) NOT NULL,
    phone VARCHAR(20) NOT NULL,
    address TEXT NOT NULL,
    city VARCHAR(100) NOT NULL,
    province VARCHAR(100) NOT NULL,
    postal_code VARCHAR(10) NOT NULL,
    latitude DECIMAL(10,8) NULL COMMENT 'GPS latitude coordinate',
    longitude DECIMAL(11,8) NULL COMMENT 'GPS longitude coordinate',
    is_default BOOLEAN DEFAULT FALSE COMMENT 'Default address for user',
    created_at TIMESTAMP NULL,
    updated_at TIMESTAMP NULL,
    
    FOREIGN KEY (user_id) REFERENCES users(id) ON DELETE CASCADE,
    INDEX idx_user_id (user_id),
    INDEX idx_is_default (is_default)
);
```

**Table Relationships:**
- `user_id`: Foreign key to `users` table (one-to-many)
- Cascade delete when user is deleted

**Key Features:**
- Support for multiple addresses per user
- GPS coordinates for precise location
- Default address functionality
- Flexible address labeling
- Complete Indonesian address format support

**Indexes:**
- Primary key on `id`
- Index on `user_id` for efficient user address queries
- Index on `is_default` for quick default address lookup

**Data Validation:**
- `latitude`: Must be between -90 and 90 degrees
- `longitude`: Must be between -180 and 180 degrees
- `postal_code`: Indonesian postal code format (5 digits)
- `phone`: Indonesian phone number format
- Only one default address per user (enforced by application logic)

**Usage Examples:**
```sql
-- Get user's default address
SELECT * FROM addresses WHERE user_id = 1 AND is_default = TRUE;

-- Get all addresses for a user, ordered by default first
SELECT * FROM addresses WHERE user_id = 1 ORDER BY is_default DESC, created_at DESC;

-- Find addresses within a city
SELECT * FROM addresses WHERE city = 'Jakarta Pusat';

-- Calculate distance using GPS coordinates (requires spatial functions)
SELECT *, 
    ST_Distance_Sphere(
        POINT(longitude, latitude),
        POINT(106.816666, -6.200000)
    ) as distance_meters
FROM addresses 
WHERE latitude IS NOT NULL AND longitude IS NOT NULL;
```

### 8. Shipment Management

#### Get All Shipments (Admin Only)
```http
GET /api/shipments
Authorization: Bearer {admin_token}
```

**Response:**
```json
{
    "success": true,
    "message": "Data shipments berhasil diambil",
    "data": [
        {
            "id": 1,
            "transaksi_id": 1,
            "address_id": 1,
            "tracking_number": "TLR20241230001",
            "status": "delivered",
            "shipping_cost": 15000,
            "distance_km": 12.5,
            "estimated_delivery": "2024-01-03T10:00:00.000000Z",
            "shipped_at": "2023-12-28T10:00:00.000000Z",
            "delivered_at": "2023-12-30T14:30:00.000000Z",
            "courier_name": "Budi Kurniawan",
            "courier_phone": "081234567890",
            "notes": "Paket telah diterima dengan baik",
            "tracking_history": [
                {
                    "status": "pending",
                    "timestamp": "2023-12-27T10:00:00.000000Z",
                    "description": "Pesanan sedang diproses"
                },
                {
                    "status": "shipped",
                    "timestamp": "2023-12-28T10:00:00.000000Z",
                    "description": "Paket telah dikirim oleh kurir"
                },
                {
                    "status": "delivered",
                    "timestamp": "2023-12-30T14:30:00.000000Z",
                    "description": "Paket telah diterima"
                }
            ],
            "transaksi": {
                "id": 1,
                "user_id": 2,
                "total_harga": 75000,
                "user": {
                    "id": 2,
                    "name": "John Doe"
                },
                "telur": {
                    "id": 1,
                    "jenis_telur": "Telur Ayam Kampung"
                }
            },
            "address": {
                "id": 1,
                "label": "Rumah",
                "recipient_name": "John Doe",
                "address": "Jl. Merdeka No. 123",
                "city": "Jakarta Pusat"
            }
        }
    ]
}
```

#### Get Specific Shipment (Admin Only)
```http
GET /api/shipments/{id}
Authorization: Bearer {admin_token}
```

#### Create Shipment (Admin Only)
```http
POST /api/shipments
Authorization: Bearer {admin_token}
Content-Type: application/json

{
    "transaksi_id": 1,
    "address_id": 1,
    "shipping_cost": 15000,
    "distance_km": 12.5,
    "courier_name": "Budi Kurniawan",
    "courier_phone": "081234567890"
}
```

**Response:**
```json
{
    "success": true,
    "message": "Shipment berhasil dibuat",
    "data": {
        "id": 1,
        "transaksi_id": 1,
        "address_id": 1,
        "tracking_number": "TLR20241230001",
        "status": "pending",
        "shipping_cost": 15000,
        "distance_km": 12.5,
        "estimated_delivery": "2024-01-02T10:00:00.000000Z",
        "courier_name": "Budi Kurniawan",
        "courier_phone": "081234567890",
        "tracking_history": [
            {
                "status": "pending",
                "timestamp": "2024-01-01T10:00:00.000000Z",
                "description": "Pesanan sedang diproses"
            }
        ]
    }
}
```

#### Update Shipment Status (Admin Only)
```http
PATCH /api/shipments/{id}/status
Authorization: Bearer {admin_token}
Content-Type: application/json

{
    "status": "shipped",
    "notes": "Paket telah dikirim dengan kurir"
}
```

**Available Status Values:**
- `pending`: Pesanan sedang diproses
- `processing`: Pesanan sedang dikemas
- `shipped`: Paket telah dikirim
- `in_transit`: Paket sedang dalam perjalanan
- `delivered`: Paket telah diterima
- `cancelled`: Pengiriman dibatalkan

**Response:**
```json
{
    "success": true,
    "message": "Status shipment berhasil diupdate",
    "data": {
        "id": 1,
        "status": "shipped",
        "shipped_at": "2024-01-01T10:00:00.000000Z",
        "tracking_history": [
            {
                "status": "pending",
                "timestamp": "2023-12-31T10:00:00.000000Z",
                "description": "Pesanan sedang diproses"
            },
            {
                "status": "shipped",
                "timestamp": "2024-01-01T10:00:00.000000Z",
                "description": "Paket telah dikirim dengan kurir"
            }
        ]
    }
}
```

#### Get User Shipments
```http
GET /api/shipments/user
Authorization: Bearer {token}
```

**Response:**
```json
{
    "success": true,
    "message": "Data shipments user berhasil diambil",
    "data": [
        {
            "id": 1,
            "tracking_number": "TLR20241230001",
            "status": "delivered",
            "shipping_cost": 15000,
            "estimated_delivery": "2024-01-03T10:00:00.000000Z",
            "delivered_at": "2023-12-30T14:30:00.000000Z",
            "courier_name": "Budi Kurniawan",
            "courier_phone": "081234567890",
            "transaksi": {
                "id": 1,
                "total_harga": 75000,
                "telur": {
                    "jenis_telur": "Telur Ayam Kampung"
                }
            },
            "address": {
                "label": "Rumah",
                "recipient_name": "John Doe",
                "address": "Jl. Merdeka No. 123"
            }
        }
    ]
}
```

#### Track Shipment by Tracking Number
```http
GET /api/shipments/track/{trackingNumber}
Authorization: Bearer {token}
```

**Example:**
```http
GET /api/shipments/track/TLR20241230001
Authorization: Bearer {token}
```

**Response:**
```json
{
    "success": true,
    "message": "Data tracking berhasil diambil",
    "data": {
        "shipment": {
            "id": 1,
            "tracking_number": "TLR20241230001",
            "status": "delivered",
            "shipping_cost": 15000,
            "distance_km": 12.5,
            "estimated_delivery": "2024-01-03T10:00:00.000000Z",
            "delivered_at": "2023-12-30T14:30:00.000000Z",
            "courier_name": "Budi Kurniawan",
            "courier_phone": "081234567890",
            "notes": "Paket telah diterima dengan baik"
        },
        "tracking_history": [
            {
                "status": "pending",
                "timestamp": "2023-12-27T10:00:00.000000Z",
                "description": "Pesanan sedang diproses"
            },
            {
                "status": "processing",
                "timestamp": "2023-12-28T08:00:00.000000Z",
                "description": "Pesanan sedang dikemas"
            },
            {
                "status": "shipped",
                "timestamp": "2023-12-28T10:00:00.000000Z",
                "description": "Paket telah dikirim oleh kurir"
            },
            {
                "status": "in_transit",
                "timestamp": "2023-12-29T14:00:00.000000Z",
                "description": "Paket dalam perjalanan"
            },
            {
                "status": "delivered",
                "timestamp": "2023-12-30T14:30:00.000000Z",
                "description": "Paket telah diterima"
            }
        ],
        "current_status": "delivered",
        "estimated_delivery": "2024-01-03T10:00:00.000000Z"
    }
}
```

**Shipment Rules:**
- Tracking number format: `TLR{YYYYMMDD}{XXX}` (contoh: TLR20241230001)
- Shipment dibuat otomatis saat transaksi dikonfirmasi atau manual oleh admin
- Status tracking tersimpan dalam JSON history untuk audit trail
- Estimasi pengiriman default 2 hari kerja dari tanggal shipped
- Shipping cost dihitung berdasarkan jarak dan berat
- User hanya bisa melihat shipment dari transaksi mereka sendiri
- Admin dapat melihat dan mengelola semua shipment

**Tracking Number Format:**
- `TLR`: Prefix untuk Telur
- `YYYYMMDD`: Tanggal pembuatan shipment
- `XXX`: Sequential number (001, 002, dst.)

**Status Workflow:**
1. `pending` → Pesanan baru dibuat
2. `processing` → Pesanan sedang dikemas
3. `shipped` → Paket telah dikirim (set `shipped_at`)
4. `in_transit` → Paket dalam perjalanan
5. `delivered` → Paket diterima (set `delivered_at`)
6. `cancelled` → Pengiriman dibatalkan

## Status Codes
- 200: OK
- 201: Created
- 400: Bad Request
- 401: Unauthorized
- 403: Forbidden
- 404: Not Found
- 422: Validation Error
- 500: Internal Server Error

## 📋 API Changelog

### v2.1 - Enhanced API & UI Improvements (Latest)
- ✅ **Dashboard Integration**: Web interface untuk admin dan user
- ✅ **Real-time Updates**: Status transaksi dan pengiriman dapat diupdate secara real-time
- ✅ **Enhanced Error Handling**: 
  - Error codes yang konsisten (INSUFFICIENT_PERMISSIONS, TRANSACTION_NOT_FOUND, dll.)
  - Response format yang terstandarisasi
  - Detailed error messages untuk debugging
- ✅ **Permission-based API**: 
  - Response API menyertakan informasi permissions user
  - Mobile app dapat menggunakan permissions untuk show/hide UI elements
  - Role-based access control yang lebih granular
- ✅ **Mobile-ready Features**:
  - Optimized response format untuk mobile consumption
  - Consistent JSON structure across all endpoints
  - Proper HTTP status codes untuk mobile error handling
- ✅ **API Documentation Improvements**:
  - Comprehensive examples untuk setiap endpoint
  - Mobile implementation guidelines
  - Error handling best practices
  - Permission system documentation

### v2.0 - Shipment Management System
- ✅ **Shipment Tracking API**: 6 endpoint baru untuk manajemen pengiriman
- ✅ **Tracking Number System**: Format TLR{YYYYMMDD}{XXX}
- ✅ **Status Workflow**: pending → processing → shipped → in_transit → delivered → cancelled
- ✅ **Tracking History**: JSON-based audit trail untuk setiap perubahan status
- ✅ **Courier Management**: Informasi kurir dan kontak dalam shipment
- ✅ **GPS-based Shipping Cost**: Kalkulasi biaya berdasarkan koordinat alamat

### v1.5 - Enhanced Features
- ✅ **Review System**: Rating dan komentar produk dengan foto
- ✅ **Address Management**: CRUD alamat dengan koordinat GPS
- ✅ **File Upload System**: Upload foto produk, bukti pembayaran, review
- ✅ **Payment Gateway Integration**: WhatsApp confirmation untuk transfer bank
- ✅ **Multi-file Upload**: Support upload multiple files sekaligus

### v1.0 - Core Features
- ✅ **Authentication System**: Laravel Sanctum dengan role-based access
- ✅ **Product Management**: CRUD telur dengan stok dan harga
- ✅ **Transaction System**: Pembelian dengan validasi stok otomatis
- ✅ **User Management**: Admin dapat mengelola user dan role
- ✅ **Payment Methods**: COD dan Transfer Bank

## 🔄 Breaking Changes

### v2.1
- **Response Format**: Semua response sekarang menyertakan `user_role` dan permission flags
- **Error Codes**: Introduced standardized error codes (tidak breaking, hanya enhancement)

### v2.0
- **Transaction Status**: Status values berubah dari `completed/cancelled` menjadi `success/cancel`
- **Database Schema**: Added shipments table dengan foreign key ke transaksis

## 🚀 Upcoming Features (Roadmap)

### v2.2 - Planned
- 📋 **Notification System**: Push notifications untuk status updates
- 📋 **Inventory Alerts**: Notifikasi stok menipis
- 📋 **Analytics Dashboard**: Laporan penjualan dan statistik
- 📋 **Bulk Operations**: Import/export data dalam format CSV

### v2.3 - Planned
- 📋 **Multi-vendor Support**: Support multiple suppliers
- 📋 **Advanced Search**: Filter dan pencarian lanjutan
- 📋 **Loyalty Program**: Point system untuk pelanggan setia
- 📋 **API Rate Limiting**: Throttling untuk mencegah abuse

---

**Last Updated**: December 2024  
**API Version**: v2.1  
**Laravel Version**: 12.x  
**Documentation Maintained by**: Telur API Team