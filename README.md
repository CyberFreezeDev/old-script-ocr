
# 🧠 Train Tesseract OCR (v5.x) from Scratch  
### Old Hindi Script ➝ Modern Hindi Characters

This guide will help you build a custom OCR model using **Tesseract 5.x**, trained to recognize old Hindi printed text and output modern Devanagari characters.

---

## ⚙️ Platform Support

| Platform | Training Recommended | Notes |
|----------|----------------------|-------|
| Ubuntu   | ✅ Best supported     | Native tools and packages available |
| macOS    | ✅ Fully supported    | Use Homebrew to install dependencies |
| Windows  | ⚠️ Use WSL or Docker | Native Windows builds lack training tools |

---

## 📦 Prerequisites

### ✅ Ubuntu / WSL

```bash
sudo apt update
sudo apt install -y tesseract-ocr tesseract-ocr-dev libleptonica-dev \\
  libicu-dev libpango1.0-dev libcairo2-dev pkg-config python3-pip \\
  g++ make automake autoconf git fonts-deva
```

Clone training repo:
```bash
git clone https://github.com/tesseract-ocr/tesstrain.git
cd tesstrain
```

---

### ✅ macOS (via Homebrew)

Install Homebrew if needed:
```bash
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"
```

Install dependencies:
```bash
brew install tesseract automake autoconf pkg-config icu4c pango cairo leptonica git make gnu-sed
git clone https://github.com/tesseract-ocr/tesstrain.git
cd tesstrain
```

---

### ⚠️ Windows (Use WSL or Docker)

1. Install **WSL2 + Ubuntu**:  
   https://learn.microsoft.com/en-us/windows/wsl/install

2. Open Ubuntu terminal and follow Ubuntu steps above

---

## 📁 Step 1: Prepare Ground Truth

```plaintext
tesstrain/data/oldhindi-ground-truth/
├── image1.tif
├── image1.gt.txt
├── image2.tif
├── image2.gt.txt
```

- `.tif`: 300 DPI scanned image (grayscale preferred)
- `.gt.txt`: UTF-8 modern Hindi equivalent of image text

---

## 🖼️ Step 2 (Optional): Generate Synthetic Data

```bash
text2image --text=yourtext.txt \\
           --outputbase=youroutput \\
           --font="Lohit Devanagari" \\
           --fonts_dir=/usr/share/fonts/truetype
```

> macOS font location: `~/Library/Fonts/` or use `fc-list :lang=hi`

---

## 🧠 Step 3: Train Your Model

```bash
make training MODEL_NAME=oldhindi \\
  START_MODEL=hin \\
  TESSDATA_PREFIX=/usr/share/tesseract-ocr/5/tessdata \\
  GROUND_TRUTH_DIR=./data/oldhindi-ground-truth
```

> Remove `START_MODEL` if training from scratch.

---

## 🛠️ Step 4: Combine & Finalize

```bash
combine_tessdata -e oldhindi.traineddata oldhindi.lstm
combine_tessdata -o oldhindi_final.traineddata oldhindi
```

Copy model to Tesseract directory:

```bash
# Linux/WSL
sudo cp oldhindi_final.traineddata /usr/share/tesseract-ocr/5/tessdata/

# macOS
cp oldhindi_final.traineddata /opt/homebrew/Cellar/tesseract/*/share/tessdata/
```

---

## ✅ Step 5: Use the Model

```bash
tesseract yourimage.tif output.txt -l oldhindi
```

---

## 📊 Step 6: Evaluate Accuracy

```bash
lstmeval --model oldhindi.lstm \\
         --traineddata oldhindi.traineddata \\
         --eval_listfile eval.txt
```

---

## 🔁 Optional: Post-OCR Modernization

Tesseract won't modernize spellings. Use:

- 🧠 A word-level mapping (`dict_old_to_new`)
- 📖 Or train a Transformer model (e.g., MarianMT or BERT2BERT) for normalization

---

## 📂 Suggested Folder Layout

```plaintext
project/
├── tesstrain/
├── data/
│   └── oldhindi-ground-truth/
│       ├── *.tif
│       └── *.gt.txt
├── models/
│   └── oldhindi.traineddata
└── scripts/
    └── modernization.py
```

---

## 🔗 References

- [Tesseract OCR GitHub](https://github.com/tesseract-ocr/tesseract)
- [Tesstrain Tool](https://github.com/tesseract-ocr/tesstrain)
- [WSL Setup Guide](https://learn.microsoft.com/en-us/windows/wsl/install)
- [Indic OCR](https://github.com/indic-dict/indic-ocr)

---

## 🧰 Need Help?

- Want a `Dockerfile`?
- GUI tools like `jTessBoxEditor`?
- Modernization model script?

Raise an issue or discussion!
```
