# 📘 Machine Learning, Deep Learning та формати моделей

Цей документ пояснює відмінності між класичними алгоритмами ML, сучасними нейронними мережами (DL), а також форматами збереження моделей (**інференс** vs **донавчання**).  

---

## 🔹 1. Класичне ML (алгоритми + ознаки)

### Алгоритми **SVM, kNN, RandomForest**:
- Це **класичні ML-алгоритми**, які не вчаться витягувати ознаки самостійно.  
- Для роботи їм треба подати **features (ознаки)**, які попередньо витягуються з зображення.  

### Приклади ознак:
- **SIFT, SURF, ORB, BRISK, FREAK** → локальні дескриптори ключових точок.  
- **HOG** → глобальний дескриптор (градієнти орієнтовані по гістограмам).  

### OpenCV і пошук ознак
У Python за допомогою **OpenCV** можна легко отримати ці дескриптори:  

```python
import cv2

# Завантаження зображення у відтінках сірого
img = cv2.imread("image.jpg", cv2.IMREAD_GRAYSCALE)

# --- Локальні дескриптори ---
# SIFT
sift = cv2.SIFT_create()
kp, des = sift.detectAndCompute(img, None)

# ORB (безкоштовна альтернатива SIFT/SURF)
orb = cv2.ORB_create()
kp, des = orb.detectAndCompute(img, None)

# BRISK
brisk = cv2.BRISK_create()
kp, des = brisk.detectAndCompute(img, None)

# --- Глобальний дескриптор ---
# HOG
hog = cv2.HOGDescriptor()
des = hog.compute(img)
```

> ⚠️ Заувага:  
> - **SIFT** і **SURF** знаходяться в модулі `opencv-contrib-python`, бо мають патенти (раніше були платними).  
> - **ORB, BRISK, FREAK, HOG** доступні у звичайному OpenCV.  


### Залежність:
```
Зображення → (екстрактор ознак: SIFT/HOG/...) → Вектори ознак → (SVM/kNN/RandomForest) → Класифікація
```

---

## 🔹 2. Deep Learning (нейромережі)

Моделі **YOLO, Faster R-CNN**:
- Це **архітектури нейронних мереж** для задач детекції об’єктів.  
- Вони **самі вчаться витягувати ознаки** з даних (end-to-end learning).  
- Можуть бути збережені у власних форматах (PyTorch `.pt/.pth`, TensorFlow `.h5/SavedModel`) або сконвертовані у **ONNX** для використання на інших платформах.  

---

## 🔹 3. ONNX (Open Neural Network Exchange)

- **ONNX** = відкритий універсальний формат для обміну нейромережами.  
- Дозволяє експортувати модель з TensorFlow / PyTorch і виконувати інференс у будь-якому середовищі (CPU, GPU, мобільні пристрої, спеціальні inference-рушії).  
- **Обмеження**: ONNX підтримує **тільки inference (передбачення)**. Тренувати модель у цьому форматі не можна.  

---

## 🔹 4. Інференс-модель vs Тренувальна модель

### 📌 Інференс-модель
- Використовується лише для **передбачення** (`predict()`).  
- Містить **архітектуру + ваги**, але **не зберігає стан оптимізатора**.  
- Продовжити навчання після завантаження такої моделі не можна.  

**Приклади форматів:**
- TensorFlow → `.pb`, `.tflite`, `.onnx`  
- PyTorch → TorchScript (`.pt`), `.onnx`  
- ONNX → `.onnx` (лише inference)  

### 📌 Тренувальна модель (для донавчання)
- Містить **ваги + архітектуру + стан оптимізатора**.  
- Дозволяє відновити навчання «з того ж місця», де воно зупинилося.  

**TensorFlow:**
- `SavedModel` (директорія) або `.h5` (HDF5 файл) → можна `load_model()` і продовжити `fit()`  
- Checkpoint (`tf.train.Checkpoint`) → зберігає ваги та оптимізатор, але для відновлення потрібна та сама архітектура вручну  

**PyTorch:**
- `.pt` / `.pth` збережені через `torch.save({'model_state': ..., 'optimizer_state': ...})`  
- Відновлення: `model.load_state_dict(...)` + `optimizer.load_state_dict(...)` → далі `train()`  

---

## 🔹 5. Чому важливо зберігати стан оптимізатора
- Якщо зберегти тільки ваги → оптимізатор почне «з нуля».  
- Якщо зберегти ще й **стан оптимізатора** → тренування продовжується так, ніби його ніколи не зупиняли.  

---

## 🔹 6. Схематично

```
Класичний ML:
Зображення → Features (SIFT/HOG/...) → SVM/kNN/RandomForest → Класифікація

DL (нейронні мережі):
Зображення → Мережа (YOLO/Faster R-CNN) → ONNX (для інференсу)

Формати:
- Інференс: .pb, .tflite, .onnx, TorchScript (.pt)
- Тренування: SavedModel, .h5 (TF), .pt/.pth (PyTorch, зі state_dict)
- ONNX → тільки inference
```

# 🔹 7. Що можна тренувати у TensorFlow і PyTorch

## TensorFlow / Keras
Підтримує тренування більшості **класичних і мобільних моделей**:  
- Класифікація: VGG, ResNet, DenseNet, Inception, MobileNet, EfficientNet  
- Сегментація: U-Net, DeepLab, Mask R-CNN, Faster R-CNN (через TF Object Detection API)  
- NLP: BERT, GPT-2, T5, XLNet (через HuggingFace)  
- Аудіо: RNN, LSTM, Tacotron, DeepSpeech  

## PyTorch
Домінує у **нових SOTA-моделях (state of the art)**:  
- CV: YOLOv5–v9 (Ultralytics), DETR, Segment Anything (SAM), Stable Diffusion, ResNet, ViT, ConvNeXt  
- NLP: GPT-2/3, LLaMA, Falcon, Mistral, HuggingFace Transformers  
- Генеративні моделі: Stable Diffusion, StyleGAN, CycleGAN  
- Аудіо: Whisper, Wav2Vec2, HuBERT  

## Обидва (TensorFlow + PyTorch)
Завдяки HuggingFace і TF Hub можна тренувати в обох фреймворках:  
- ResNet, EfficientNet, MobileNet, Inception, VGG  
- Vision Transformers (ViT, Swin, DeiT)  
- BERT, T5, DistilBERT, RoBERTa  
- U-Net, DeepLab  

---

# 🔹 8. Ultralytics YOLO

**Ultralytics** — компанія, яка створила YOLOv5 і розвиває сучасні YOLO (v5–v9).  
Вони перенесли YOLO з Darknet (C/CUDA) у **PyTorch**, зробили бібліотеку `ultralytics` (pip), яка підтримує:  

- Тренування: `yolo train ...`  
- Інференс: `yolo predict ...`  
- Оцінка: `yolo val ...`  
- Експорт: `yolo export ...` (ONNX, TFLite, TensorRT, CoreML, SavedModel і т.д.)  

👉 Важливо: **Ultralytics YOLO можна тренувати тільки в PyTorch**.  
В TensorFlow модель можна експортувати, але **лише для інференсу**.  

---
