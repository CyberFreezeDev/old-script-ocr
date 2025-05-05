# 🧠 Train Tesseract OCR on macOS (v5.x) from Scratch  
### Convert Old Hindi Script ➝ Modern Devanagari Characters

This guide will walk you through training a custom OCR model using **Tesseract 5.x** on **macOS**, tailored for recognizing old Hindi printed scripts and outputting modern Unicode Hindi text.

---

## 🍏 macOS: System Requirements

- macOS Ventura / Monterey / Big Sur
- Homebrew installed (https://brew.sh)
- Xcode Command Line Tools installed

---

## 📦 Step 1: Install Tesseract and Dependencies

### ✅ Install Homebrew (if not installed)
```bash
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"
```

### ✅ Install Tesseract and other tools
```bash
brew install tesseract automake autoconf pkg-config icu4c pango cairo leptonica make gnu-sed
```

---

## 📂 Step 2: Clone Tesstrain Repository

```bash
git clone https://github.com/tesseract-ocr/tesstrain.git
cd tesstrain
```

This repo contains scripts to automate the training process.

---

## 🖋️ Step 3: Prepare Ground Truth Data

Create this structure under `tesstrain/data/oldhindi-ground-truth/`:

```plaintext
tesstrain/data/oldhindi-ground-truth/
├── image1.tif
├── image1.gt.txt
├── image2.tif
├── image2.gt.txt
```

- `.tif`: 300 DPI scanned image (grayscale recommended)
- `.gt.txt`: Modern Hindi text in UTF-8 format matching the content of the image

---

## 🖼️ Step 4 (Optional): Generate Synthetic Training Images

You can generate consistent training data using a Hindi font like Lohit or Noto Sans Devanagari:

### 🔍 Find installed fonts

```bash
fc-list :lang=hi
```

### ✅ Run `text2image`
```bash
text2image --text=yourtext.txt \\
           --outputbase=synthetic \\
           --font="Lohit Devanagari" \\
           --fonts_dir=/System/Library/Fonts
```

> Replace `"Lohit Devanagari"` with an available Hindi font on your system.

---

## 🧠 Step 5: Train Your Custom Model

Use the following command inside the `tesstrain` directory:

```bash
make training MODEL_NAME=oldhindi \\
  START_MODEL=hin \\
  TESSDATA_PREFIX=/opt/homebrew/Cellar/tesseract/5.*/share/tessdata \\
  GROUND_TRUTH_DIR=./data/oldhindi-ground-truth
```

- `START_MODEL=hin`: Uses base Hindi model as a starting point
- Remove `START_MODEL` to train from scratch

---

## 🛠️ Step 6: Combine & Finalize the Model

```bash
combine_tessdata -e oldhindi.traineddata oldhindi.lstm
combine_tessdata -o oldhindi_final.traineddata oldhindi
```

### ✅ Copy model to Tesseract tessdata path

```bash
cp oldhindi_final.traineddata /opt/homebrew/Cellar/tesseract/*/share/tessdata/
```

---

## ✅ Step 7: Test Your Model

Run inference using:

```bash
tesseract yourimage.tif output.txt -l oldhindi
```

---

## 📊 Step 8: Evaluate Accuracy (Optional)

```bash
lstmeval --model oldhindi.lstm \\
         --traineddata oldhindi.traineddata \\
         --eval_listfile eval.txt
```

---

## 🔁 Bonus: Post-OCR Modernization

Tesseract doesn't normalize older spellings. To convert outputs to **modern Devanagari**, consider:

- 🔠 A rule-based substitution script (`dict_old_to_new`)
- 🤖 Training a small seq2seq model (e.g., MarianMT, BERT2BERT)

---

## 📁 Recommended Folder Layout

```plaintext
project/
├── tesstrain/
├── data/
│   └── oldhindi-ground-truth/
│       ├── image1.tif
│       ├── image1.gt.txt
├── models/
│   └── oldhindi.traineddata
└── scripts/
    └── modernization.py
```

---

## 🔗 References

- [Tesseract OCR GitHub](https://github.com/tesseract-ocr/tesseract)
- [Tesstrain](https://github.com/tesseract-ocr/tesstrain)
- [Fonts for Devanagari](https://fonts.google.com/noto/specimen/Noto+Sans+Devanagari)
- [Indic OCR Dataset](https://github.com/indic-dict/indic-ocr)

---

## 💬 Need Help?

- Need a `Dockerfile` version?
- Want to automate spell modernization?
- OCR is producing poor results?

Create an issue or contact us!
