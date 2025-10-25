# Skincare Compatibility ML API

Proyek Machine Learning untuk memprediksi apakah dua produk skincare cocok digunakan bersama berdasarkan kesamaan bahan aktifnya.

## Features v1.3.0

- **Compatibility Check** - Cek kompatibilitas 2 produk sekaligus
- **Auto-Recommendations** - Rekomendasi otomatis muncul saat produk tidak cocok
- **Smart Recommendations** - Dapatkan rekomendasi produk yang cocok (optimized)
- **Educational Explanations** - Penjelasan detail WHY compatible/not
- **Ingredient Info** - Database pengetahuan tentang bahan skincare
- **Usage Tips** - Tips penggunaan untuk setiap kombinasi
- **Organized Structure** - Model & data di folder `model_training/`

### What's New in Latest Update:

- ⚡ **Performance Optimization**: `/recommend` endpoint 18-30x lebih cepat (90s → 3-5s)
- 🎯 **Alternative Recommendations**: Otomatis muncul saat produk tidak cocok
- 📦 **Batch Prediction**: Predict 2,610 produk sekaligus untuk response cepat

---

## Cara Menjalankan (untuk tim backend)

### 1. Instal dependensi

Pastikan sudah menginstal semua library yang diperlukan:

```bash
pip install -r requirements.txt
```

### 2. Jalankan API

```bash
python app.py
```

API akan berjalan di:

- **Local:** http://localhost:5000
- **Network:** http://0.0.0.0:5000

---

## Endpoint API

### 1. POST /predict

Mengecek kompatibilitas antara dua produk skincare.

**FITUR BARU:** Jika produk tidak cocok, otomatis muncul rekomendasi alternatif!

**Request JSON:**

```json
{
  "product1": "cerave cream",
  "product2": "ordinary hyaluronic acid"
}
```

**Response JSON (jika COCOK):**

```json
{
  "status": "ok",
  "product1": "CeraVe Moisturising Cream 454g",
  "product2": "The Ordinary Hyaluronic Acid 2% + B5",
  "result": "Cocok digunakan bersama",
  "confidence": 0.87,
  "shared_ingredients_count": 5,
  "shared_ingredients": ["hyaluronic acid", "glycerin", "ceramide"],
  "explanation": "Kedua produk dapat digunakan bersama karena...",
  "tips": "💡 Tips: Aplikasikan dari tekstur ringan ke berat..."
}
```

**Response JSON (jika TIDAK COCOK):**

```json
{
  "status": "ok",
  "product1": "Retinol Serum",
  "product2": "Vitamin C Serum",
  "result": "Tidak disarankan dipakai bersama",
  "confidence": 0.85,
  "explanation": "Kombinasi Retinol dan Vitamin C dapat menyebabkan iritasi...",
  "tips": "⚠️ Gunakan pada waktu berbeda (pagi/malam)",

  "alternative_recommendations": {
    "note": "Karena kedua produk tidak cocok, berikut rekomendasi alternatif:",
    "replace_product1": {
      "info": "Ganti 'Retinol Serum' dengan produk yang cocok dengan 'Vitamin C Serum':",
      "recommendations": [
        {
          "product_name": "Niacinamide Serum",
          "confidence": 0.95,
          "shared_ingredients_count": 2
        }
      ]
    },
    "replace_product2": {
      "info": "Atau ganti 'Vitamin C Serum' dengan produk yang cocok dengan 'Retinol Serum':",
      "recommendations": [
        {
          "product_name": "Ceramide Cream",
          "confidence": 0.98,
          "shared_ingredients_count": 4
        }
      ]
    }
  }
}
```

### 2. POST /recommend

Mendapatkan rekomendasi produk yang kompatibel dengan produk input.

**OPTIMIZED:** Response time dipercepat dari 90 detik ke 3-5 detik!

**Request JSON:**

```json
{
  "product": "cerave cream",
  "top_n": 5
}
```

**Response JSON:**

```json
{
  "status": "ok",
  "product": "CeraVe Moisturising Cream 454g",
  "total_recommendations": 5,
  "recommendations": [
    {
      "product_name": "The Ordinary Hyaluronic Acid 2% + B5",
      "confidence": 0.92,
      "shared_ingredients_count": 5
    },
    {
      "product_name": "Cetaphil Gentle Skin Cleanser",
      "confidence": 0.88,
      "shared_ingredients_count": 4
    },
    {
      "product_name": "Neutrogena Hydro Boost",
      "confidence": 0.85,
      "shared_ingredients_count": 3
    }
  ],
  "note": "Produk teratas memiliki 5 bahan yang sama...",
  "general_tip": "💡 Untuk hasil terbaik, lakukan patch test terlebih dahulu"
}
```

**Performance:**

- Dataset size: 2,610 produk
- Response time: ~3-5 detik (optimized dengan batch prediction)
- Accuracy: Sama dengan versi sebelumnya

---

### 3. POST /ingredient-info

Mendapatkan informasi edukatif tentang bahan skincare populer.

**Request JSON:**

```json
{
  "ingredient": "niacinamide"
}
```

**Response:** Informasi lengkap tentang bahan (benefits, suitable for, combinations, tips)

---

## Performance

| Endpoint           | Response Time | Dataset Size  |
| ------------------ | ------------- | ------------- |
| `/predict`         | ~0.5-1s       | 2,610 produk  |
| `/recommend`       | ~3-5s         | 2,610 produk  |
| `/ingredient-info` | <0.1s         | 8 ingredients |

**Optimizations:**

- Batch prediction untuk `/recommend` (18-30x lebih cepat)
- Vectorization untuk feature engineering
- Efficient filtering dengan NumPy

---

## Technical Details

- **Model:** RandomForestClassifier (200 trees, max_depth=8)
- **Features:** TF-IDF vectors, cosine similarity, shared ingredients
- **Dataset:** 2,610 skincare products
- **Accuracy:** High compatibility prediction
- **Python Version:** 3.10+
- **Framework:** Flask 3.0+ with CORS support

---
