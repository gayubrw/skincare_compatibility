# Skincare Compatibility ML API 🧴✨

Proyek Machine Learning untuk memprediksi apakah dua produk skincare cocok digunakan bersama berdasarkan kesamaan bahan aktifnya.

## ✨ Features v1.3.0

- ✅ **Compatibility Check** - Cek kompatibilitas 2 produk sekaligus
- ✅ **Smart Recommendations** - Dapatkan rekomendasi produk yang cocok
- ✅ **Educational Explanations** - Penjelasan detail WHY compatible/not
- ✅ **Ingredient Info** - Database pengetahuan tentang bahan skincare
- ✅ **Usage Tips** - Tips penggunaan untuk setiap kombinasi
- ✅ **Organized Structure** - Model & data di folder `model_training/`

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

**Request JSON:**

```json
{
  "product1": "cerave cream",
  "product2": "ordinary hyaluronic acid"
}
```

**Response JSON:**

```json
{
  "status": "ok",
  "product1": "CeraVe Moisturising Cream 454g",
  "product2": "The Ordinary Hyaluronic Acid 2% + B5",
  "result": "✅ Cocok digunakan bersama",
  "confidence": 0.87,
  "shared_ingredients_count": 5
}
```

### 2. POST /recommend

Mendapatkan rekomendasi produk yang kompatibel dengan produk input.

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
    }
  ]
}
```

---

## Catatan

- Model dilatih menggunakan RandomForestClassifier dengan fitur:
  - Panjang nama produk (len_diff)
  - Jumlah bahan aktif yang sama (shared)
  - Jaccard similarity
  - Cosine similarity
- Dataset: unified_cleaned_products.csv
- Versi Python yang disarankan: Python 3.10+

---
