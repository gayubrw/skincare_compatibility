# Skincare Compatibility ML API

Proyek Machine Learning untuk memprediksi apakah dua produk skincare cocok digunakan bersama berdasarkan kesamaan bahan aktifnya.

---

## 🚀 Quick Start Guide (untuk yang baru clone)

### Prerequisites

- Python 3.10 atau lebih baru
- pip (Python package manager)
- Git

### Langkah-langkah Setup:

#### Clone Repository

```bash
git clone https://github.com/gayubrw/skincare_compatibility.git
cd skincare_compatibility
```

#### Buat Virtual Environment (Opsional tapi Direkomendasikan)

**Windows:**

```bash
python -m venv venv
venv\Scripts\activate
```

**macOS/Linux:**

```bash
python3 -m venv venv
source venv/bin/activate
```

#### Install Dependencies

```bash
pip install -r requirements.txt
```

**Dependencies yang akan diinstall:**

- Flask 3.0+ (Web framework)
- scikit-learn 1.6.1 (ML library)
- pandas 2.0+ (Data processing)
- numpy 1.24+ (Numerical computing)
- gunicorn 21.2+ (Production server)
- flask-cors (CORS support)

#### Verifikasi File Model & Dataset

Pastikan file-file ini ada di folder `model_training/`:

- `skincare_model.pkl` (94 KB) - Trained ML model
- `unified_cleaned_products.csv` (560 KB) - 2,610 produk
- `compatibility_rules.csv` (1 KB) - Rules untuk penjelasan

#### Jalankan API

```bash
python app.py
```

**Output yang benar:**

```
Compatibility rules loaded: 5 rules
Model loaded successfully!
Dataset loaded: 2610 products
Starting server on port 5000
Debug mode: True
 * Running on http://127.0.0.1:5000
```

#### Test API

**Test dengan browser:**
Buka: http://localhost:5000/health

**Test dengan Postman:**

- Method: POST
- URL: `http://localhost:5000/predict`
- Body (raw JSON):

```json
{
  "product1": "cerave cream",
  "product2": "ordinary hyaluronic acid"
}
```

---

## Troubleshooting

### Error: "No module named 'flask'"

**Solusi:** Install dependencies

```bash
pip install -r requirements.txt
```

### Error: "Model file not found"

**Solusi:** Pastikan file `model_training/skincare_model.pkl` ada

### Error: "Port 5000 already in use"

**Solusi:** Ubah port di `app.py` atau kill process yang pakai port 5000

```bash
# Windows
netstat -ano | findstr :5000
taskkill /PID <PID> /F

# macOS/Linux
lsof -i :5000
kill -9 <PID>
```

### Model Tidak Ditemukan

**Gejala:** Error "Model file not found"

**Solusi:**

```bash
# Pastikan model ada di folder yang benar
ls model_training/skincare_model.pkl

# Jika tidak ada, retrain model:
cd model_training
jupyter notebook Feature_Engineering_&_Model_Training.ipynb
# Run all cells (Cell → Run All)
```

### Port 5000 Sudah Dipakai

**Solusi:**

```bash
# Windows
netstat -ano | findstr :5000
taskkill /PID <PID> /F

# macOS/Linux
lsof -i :5000
kill -9 <PID>
```

### API Lambat / Timeout

**Solusi:**

- Model sudah di-optimasi, response time normal:
  - `/predict`: ~1s
  - `/recommend`: ~3-5s (batch prediction)
  - `/ingredient-info`: <1s
  - `/tips`: <1s

### Model Selalu Predict "Compatible"

**Status:** ✅ **FIXED in v2.0!**

Model baru menggunakan balanced training data, sehingga bisa predict BOTH compatible AND incompatible dengan akurat.

---

## 📦 Deployment ke Production

### Deploy ke Render (Recommended)

1. Push repository ke GitHub
2. Buat account di [Render.com](https://render.com)
3. Buat New Web Service
4. Connect repository GitHub
5. Konfigurasi:
   - **Build Command:** `pip install -r requirements.txt`
   - **Start Command:** `gunicorn app:app`
   - **Environment:** Python 3

### Deploy ke Heroku

```bash
heroku login
heroku create skincare-compatibility-api
git push heroku main
```

File `Procfile` dan `runtime.txt` sudah disediakan.

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

| Endpoint           | Response Time | Dataset Size  | Note                         |
| ------------------ | ------------- | ------------- | ---------------------------- |
| `/predict`         | ~0.5-1s       | 2,610 produk  | Single pair prediction       |
| `/recommend`       | ~3-5s         | 2,610 produk  | Batch prediction (optimized) |
| `/ingredient-info` | <0.1s         | 8 ingredients | Instant lookup               |
| `/tips`            | <0.1s         | Rule-based    | Instant response             |

**Optimizations Applied:**

- ✅ Batch prediction untuk `/recommend` (18-30x lebih cepat: 90s → 3-5s)
- ✅ Vectorization untuk feature engineering
- ✅ Efficient filtering dengan NumPy arrays
- ✅ Balanced model training (no bias, accurate predictions)

---

## 📊 Technical Details

### Model Specifications

**Algorithm:** RandomForestClassifier

**Parameters:**

- `n_estimators=200` (200 decision trees)
- `max_depth=8` (tree depth limit)
- `class_weight='balanced'` (handle any remaining imbalance)
- `random_state=42` (reproducibility)
- `n_jobs=-1` (parallel processing)

**Training Data:**

- **Total:** 4,000 balanced pairs
- **Compatible:** 2,000 pairs (50%)
- **Incompatible:** 2,000 pairs (50%)
- **Source:** 2,610 unique skincare products

**Features (4 total):**

1. `len_diff` - Difference in ingredient list length
2. `shared_ingredients` - Count of shared ingredients (**44% importance**)
3. `jaccard_similarity` - Set intersection metric (**46.3% importance**)
4. `cosine_similarity` - TF vector similarity (**9.6% importance**)

**Performance Metrics:**

- ✅ Train Accuracy: **100%**
- ✅ Test Accuracy: **100%**
- ✅ Can predict Compatible: **YES**
- ✅ Can predict Incompatible: **YES**
- ✅ Confusion Matrix: Perfect (no false positives/negatives)

**Conflict Detection:**

- Retinol + Vitamin C (L-Ascorbic Acid)
- AHA/BHA + Retinol
- Benzoyl Peroxide + Retinol

### Technology Stack

- **Python:** 3.10+
- **Framework:** Flask 3.0+ with CORS support
- **ML Library:** scikit-learn 1.6.1
- **Data Processing:** pandas 2.0+, numpy 1.24+
- **Production Server:** Gunicorn 21.2+
- **Training Environment:** Jupyter Notebook with matplotlib, seaborn

---

## 🗂️ Project Structure

```

ML-SkincareCompatibility/
├── app.py # Main Flask API (500+ lines)
├── requirements.txt # Python dependencies
├── runtime.txt # Python version for deployment
├── Procfile # Gunicorn configuration
├── README.md # Documentation (this file)
├── INSTALLATION.md # Detailed installation guide
├── MODEL*TRAINING_SUMMARY.md # Training process documentation
├── model_training/
│ ├── skincare_model.pkl # ✅ Trained ML model (94 KB)
│ ├── unified_cleaned_products.csv # Product database (2,610 products)
│ ├── compatibility_rules.csv # Compatibility rules (5 rules)
│ ├── master_ingredient_dictionary.csv # Ingredient reference
│ └── Feature_Engineering*&\_Model_Training.ipynb # ✅ Training notebook (balanced data)
└── **pycache**/ # Python cache (auto-generated)

```

**Important Files:**

- `Feature_Engineering_&_Model_Training.ipynb` ✅ **Use this for retraining** - Balanced data with visualizations

````

**Important Files:**

- `Feature_Engineering_&_Model_Training.ipynb` ✅ **Use this for retraining** - Balanced data with visualizations
- `app.py` - Production Flask API

---

## 🔄 Retraining Model (Advanced)

**Kapan perlu retrain:**

- Ada produk baru di dataset (> 10% increase)
- Update compatibility rules
- Model accuracy menurun
- Perubahan feature engineering

### **Cara Retrain:**

Gunakan Jupyter Notebook untuk training interaktif dengan visualisasi:

```bash
# Navigate to model_training folder
cd model_training

# Open notebook
jupyter notebook Feature_Engineering_&_Model_Training.ipynb
````

**Features:**

- ✅ **Balanced Training Data** - 50% compatible + 50% incompatible pairs
- ✅ **Conflict Detection** - Automatically detect conflicting ingredient pairs
- ✅ **Visualisasi** - Distribusi data, confusion matrix, feature importance
- ✅ **Step-by-step Explanation** - Clear documentation in each cell
- ✅ **Production-Ready Model** - Can predict BOTH compatible AND incompatible

**Cara Pakai:**

1. Run all cells dari atas ke bawah (Cell → Run All)
2. Model akan tersimpan di `model_training/skincare_model.pkl`
3. Restart Flask API: `python app.py`
4. Test di Postman - model sekarang bisa predict Compatible DAN Incompatible!

**Expected Output:**

```
✅ Loaded 2610 products
✅ Found 3 conflicting pairs
✅ Generated 2000 compatible pairs
✅ Generated 2000 incompatible pairs
⚖️ Dataset is BALANCED!
🎯 Accuracy: Train: 100.00%, Test: 100.00%
💾 Model saved to: skincare_model.pkl
```

**Verification:**

- ✅ Notebook telah di-test dan berjalan tanpa error
- ✅ Model yang dihasilkan dapat predict BOTH compatible dan incompatible
- ✅ API berjalan normal setelah menggunakan model baru
- ✅ Response time tetap optimal (~3-5s untuk `/recommend`)

---

## 👥 Contributing

Untuk kontribusi:

1. Fork repository
2. Create feature branch (`git checkout -b feature/AmazingFeature`)
3. Commit changes (`git commit -m 'Add some AmazingFeature'`)
4. Push to branch (`git push origin feature/AmazingFeature`)
5. Open Pull Request

**For Model Improvements:**

- Open notebook: `model_training/Feature_Engineering_&_Model_Training.ipynb`
- Experiment with parameters (n_estimators, max_depth, features)
- Document changes in notebook markdown cells
- Test accuracy before committing
- Update `MODEL_TRAINING_SUMMARY.md` with results

---

## 📝 License

This project is for educational purposes.

---

## 👨‍💻 Team

- **ML Team Lead:** Mas Gilang
- **ML Developer:** Gayu
- **Flutter Team:** 2 developers

---

## 📞 Support

Jika ada masalah:

1. Cek section [Troubleshooting](#-troubleshooting)
2. Lihat [Issues](https://github.com/gayubrw/skincare_compatibility/issues)
3. Hubungi tim ML

---

**Last Updated:** October 25, 2025
**Version:** 1.3.0
