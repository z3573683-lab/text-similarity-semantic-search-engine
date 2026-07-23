# 📖 شرح كامل للنوتبوك | Full Notebook Walkthrough
### Production-Grade Two-Stage Semantic Search Engine

شرح كل خطوة فعلتها في النوتبوك بالعربي والإنجليزي - إيه اللي بيحصل، ليه، والنتيجة.
An explanation of every step in the notebook - what happens, why, and the outcome.

---

## 📁 Phase 1: Data Engineering | هندسة البيانات

### Step 1.1: Data Loading & Initial Exploration | تحميل واستكشاف البيانات
- **EN:** Mounted Google Drive, extracted the Quora dataset ZIP, and loaded `questions.csv` (404,351 pairs, 6 columns: `id`, `qid1`, `qid2`, `question1`, `question2`, `is_duplicate`). Checked for missing values (found 3 total) and inspected class balance.
- **AR:** ربطت جوجل درايف، فككت ملف الداتا المضغوط، وحمّلت `questions.csv` (404,351 زوج، 6 أعمدة). فحصت القيم المفقودة (لقيت 3 بس) وشفت توازن الفئات.
- **النتيجة | Result:** 63.08% غير متطابق (Class 0) مقابل 36.92% متطابق (Class 1) — عدم توازن متوسط بيحتم استخدام F1 بدل Accuracy.

### Step 1.2: Handling Missing Values | معالجة القيم المفقودة
- **EN:** Dropped 3 rows with `NaN` in `question1`/`question2`, then re-verified zero missing values remain.
- **AR:** حذفت 3 صفوف فيها قيم فاضية، وأكدت إن مفيش أي قيمة فاضية باقية.
- **النتيجة:** 404,348 صف نظيف.

### Step 1.3: Handling Duplicate Rows & Swapped Pairs | التكرارات والأزواج المعكوسة
- **EN:** Checked for exact duplicate rows AND swapped pairs `(q1,q2)` vs `(q2,q1)` using a normalized min/max tuple trick — important because a naive duplicate check would miss reversed pairs.
- **AR:** فحصت الصفوف المكررة تماماً + الأزواج المعكوسة (سؤال أ مع ب، وسؤال ب مع أ) باستخدام حيلة ترتيب (min/max) — مهم لأن الفحص العادي مش هيمسك الأزواج المعكوسة.
- **النتيجة:** 0 تكرار مباشر، 0 تكرار معكوس — الداتا كانت نضيفة أصلاً من الناحية دي.

### Step 1.4: Advanced Text Cleaning & Normalization | التنظيف المتقدم
- **EN:** Lowercased text, expanded contractions (`can't` → `cannot`), removed URLs/emails/HTML tags/non-alphanumeric characters. Found 16+9 rows that became empty strings after cleaning (pure symbol/link questions) and dropped them.
- **AR:** تصغير الحروف، فك الاختصارات، إزالة الروابط والإيميلات ووسوم HTML والرموز الخاصة. لقيت 16+9 صف بقوا فاضيين تماماً بعد التنظيف (أسئلة كانت رموز/روابط بس) وحذفتهم.
- **النتيجة:** 404,324 صف نهائي بعد التنظيف الكامل.

### Step 2.1: Text Length Analysis & Outlier Handling | تحليل الأطوال والقيم الشاذة
- **EN:** Computed word/char counts. Mean ≈11 words, 75th percentile ≤13 words, extreme tail up to 247 words. Filtered to keep only `[2, 50]` word-count range, preserving 99.5%+ of the data.
- **AR:** حسبت عدد الكلمات والحروف. المتوسط ~11 كلمة، وفيه Outliers توصل لـ247 كلمة. فلترت للاحتفاظ بالأسئلة من 2 لـ50 كلمة بس.
- **النتيجة:** إزالة الأسئلة القصيرة جداً (بدون سياق كافي) والطويلة جداً (شاذة) بدون خسارة جزء كبير من الداتا.

### Step 2.2: Visualization & Audit | التصوير البصري والتدقيق
- **EN:** Generated Word Clouds for both question columns (top terms: `best`, `india`, `difference`, `quora`), and re-checked class balance before/after cleaning (63.1% / 36.9%, unchanged) to confirm cleaning didn't introduce label bias.
- **AR:** عملت Word Cloud للأسئلة، وأعدت فحص توازن الفئات قبل وبعد التنظيف (النسبة اتحافظت عليها) للتأكد إن التنظيف مأثرش على توزيع الفئة المستهدفة.

---

## 🔒 Phase 3: Data Splitting & Test Set Isolation | تقسيم البيانات وعزل الاختبار

- **EN:** Performed a **single, final, stratified split**: 70% Train / 15% Validation / 15% Test based on `is_duplicate`. The Test set's indices are treated as permanently locked from this point forward — no future operation is allowed to re-sample or touch them.
- **AR:** عملت **تقسيم واحد نهائي فقط** (Stratified) بنسبة 70% تدريب / 15% تحقق / 15% اختبار. أندكسات مجموعة الاختبار بقيت "مقفولة" من هنا وبعد كده، ومفيش أي خطوة تانية بتلمسها.
- **ليه ده مهم | Why this matters:** ده هو أهم فرق بين النسخة دي والنسخة القديمة — في النسخة الأولى كان بيتعمل split تاني على الداتا الكاملة بعد كده، وده كان يعني إن أجزاء من الـ Test بتدخل في التدريب من غير قصد (Data Leakage). هنا اتمنع تماماً.

---

## 🚀 Phase 4: Bi-Encoder Fine-Tuning & Threshold Calibration | تدريب Bi-Encoder ومعايرة العتبة

### Step 4.1: Zero-Shot Baseline Evaluation | تقييم الموديل الجاهز قبل التدريب
- **EN:** Evaluated the raw pretrained `all-MiniLM-L6-v2` (before any fine-tuning) on the Validation set using cosine similarity at the default threshold (0.5).
- **AR:** قيّمت الموديل الجاهز (`all-MiniLM-L6-v2`) قبل أي تدريب على مجموعة التحقق باستخدام Cosine Similarity عند العتبة الافتراضية (0.5).
- **النتيجة:** F1 = 0.6491 (Precision: 0.4810, Recall: 0.9981) — Recall عالي جداً وPrecision ضعيف، معناه الموديل الجاهز بيصنف كل حاجة تقريباً "متطابقة" عند العتبة الافتراضية. ده بيثبت الحاجة الفعلية للـ Fine-Tuning والمعايرة.

### Step 4.2: Bi-Encoder Fine-Tuning with Contrastive Loss | تدريب الموديل
- **EN:** Fine-tuned the model using `OnlineContrastiveLoss` (pulls duplicate-question embeddings closer, pushes non-duplicates apart) — trained **strictly on the Train set only** (282,258 rows), 1 epoch, LR=2e-5, 10% warmup.
- **AR:** درّبت الموديل باستخدام `OnlineContrastiveLoss` — التدريب كان على مجموعة التدريب بس، دورة واحدة، معدل تعلم 2e-5.

### Step 4.3: Threshold Calibration on Validation | معايرة العتبة
- **EN:** Swept cosine similarity thresholds from 0.10 to 0.90 (81 steps) on the Validation set, selecting the threshold that maximizes F1 — instead of guessing a fixed number.
- **AR:** جربت عتبات من 0.10 لـ0.90 (81 قيمة) على مجموعة التحقق واخترت العتبة اللي بتدي أعلى F1 — بدل ما أحط رقم عشوائي.
- **النتيجة:** τ* = 0.79، F1 = 0.8302 على الـ Validation (تحسّن +18.1% عن الـ Baseline).

### Step 4.5: Final Evaluation on the Locked Test Set | التقييم النهائي على الاختبار المحجوز
- **EN:** Applied the calibrated threshold (0.79) — derived only from Validation — to the never-touched Test set.
- **AR:** طبّقت العتبة المعايرة (0.79) — اللي اتحسبت من التحقق بس — على مجموعة الاختبار اللي متلمستش خالص قبل كده.
- **النتيجة:** F1 = 0.8337, Accuracy = 86.63%, Recall (Class 1) = 90.51% — قريبة جداً من نتيجة الـ Validation، يعني الموديل مش بيعمل Overfitting وقادر يعمم كويس.

---

## 🏗️ Phase 5: Retrieval Infrastructure & Two-Stage Search | البنية التحتية للاسترجاع

### Step 5.1: Building FAISS Index + Loading Cross-Encoder | بناء الفهرس وتحميل المُعيد ترتيب
- **EN:** Collected all unique questions from **Train + Validation only** (Test excluded) as the searchable Knowledge Base (>300K questions). Normalized embeddings (`faiss.normalize_L2`) so that `IndexFlatIP` (Inner Product) becomes mathematically equivalent to Cosine Similarity. Loaded a pretrained Cross-Encoder (`ms-marco-MiniLM-L-6-v2`) for re-ranking.
- **AR:** جمّعت كل الأسئلة الفريدة من التدريب والتحقق بس (مش الاختبار) كقاعدة معرفية للبحث. طبّقت L2 Normalization عشان الـ Inner Product يبقى مكافئ رياضياً للـ Cosine Similarity. حمّلت Cross-Encoder جاهز لإعادة الترتيب.

### Step 5.2: Two-Stage Search Function | دالة البحث بمرحلتين
- **EN:** Built `search_similar_questions()`: Stage 1 retrieves Top-K candidates via FAISS (fast, approximate relevance), Stage 2 re-scores each candidate with the Cross-Encoder (slow, precise relevance), then sorts by the Cross-Encoder score.
- **AR:** بنيت دالة بحث بمرحلتين: الأولى تجيب أقرب مرشحين بسرعة من FAISS، والتانية تعيد تقييمهم بدقة بالـ Cross-Encoder، وبعدين ترتبهم حسب سكور المرحلة التانية.

### Step 5.3: Latency Benchmarking | قياس زمن الاستجابة
- **EN:** Measured average latency over 50 runs: Stage 1 ≈ 88.79 ms, Stage 2 ≈ 19.57 ms, Total ≈ 108.36 ms — well within real-time search SLAs (<100-150ms typical).
- **AR:** قياس متوسط زمن الاستجابة على 50 تشغيلة: المرحلة الأولى ~89 ملي ثانية، الثانية ~20 ملي ثانية، الإجمالي ~108 ملي ثانية.

---

## 📊 Phase 6: Comprehensive System Evaluation | التقييم الشامل

### Step 6.1: Safety Assertions & Confusion Matrix | فحوصات الأمان ومصفوفة الالتباس
- **EN:** Programmatic assertions confirmed **zero index overlap** between Train/Val/Test (`assert` statements, not just a visual check). Plotted the Confusion Matrix on the Test set: 32,130 True Negatives, 20,269 True Positives, 5,960 False Positives, 2,126 False Negatives.
- **AR:** فحص برمجي (Assert) أكد عدم وجود أي تقاطع بين المجموعات التلاتة. رسمت مصفوفة الالتباس على الاختبار.

### Step 6.2: Retrieval Metrics — Hit Rate@K & MRR@K | مقاييس الاسترجاع
- **EN:** Sampled 1,000 duplicate pairs from the Test set and ran them through the full two-stage pipeline to measure **end-to-end search quality** (not just pairwise classification).
- **AR:** أخدت عينة 1000 زوج متطابق من الاختبار وشغّلتهم على الـ Pipeline كامل لقياس **جودة البحث الفعلية** (مش بس تصنيف الزوج).
- **النتيجة:**
  - Hit Rate@1 = 5.20% (52/1000)
  - Hit Rate@5 = 32.60% (326/1000)
  - Hit Rate@10 = 45.00% (450/1000)
  - MRR@10 = 0.1676
- **التفسير | Interpretation:** الفرق الكبير بين F1 عالي (0.83) للتصنيف الثنائي (زوج محدد سلفاً) مقابل Hit Rate@1 منخفض (5.2%) للبحث المفتوح، بيوضح إن "تصنيف هل زوجين متطابقين" أسهل بكتير من "إيجاد السؤال الصح وسط 300 ألف مرشح ووضعه في المركز الأول" — دي مشكلة استرجاع (Retrieval) مش مشكلة تصنيف.

---

## 🚀 Phase 7: Deployment & API Service | النشر وبناء الـ API

### Step 7.1: Saving Production Artifacts | حفظ مخرجات الإنتاج
- **EN:** Saved the fine-tuned model, FAISS index, Knowledge Base question list, and a `config.json` with the calibrated threshold + metadata — all needed to reload the system without retraining.
- **AR:** حفظت الموديل المدرب، فهرس FAISS، قايمة أسئلة قاعدة المعرفة، وملف إعدادات فيه العتبة المعايرة — كل ده عشان تقدر تشغل النظام تاني من غير إعادة تدريب.

### Step 7.2: `requirements.txt` Generation | توليد ملف المتطلبات
- **EN:** Documented all runtime dependencies (`fastapi`, `uvicorn`, `pydantic`, `sentence-transformers`, `faiss-cpu`, `torch`, etc.) — fixed from the earlier version which was missing the API-serving libraries.
- **AR:** وثّقت كل المكتبات المطلوبة للتشغيل — اتصلحت من النسخة القديمة اللي كانت ناقصة مكتبات تشغيل الـ API.

### Step 7.3: Building `app.py` (Gradio Interface) | بناء واجهة التطبيق
- **EN:** Wrote a standalone `app.py` that loads all artifacts once at startup and exposes a Gradio interface with adjustable Top-K sliders — this is the exact code deployed live to Hugging Face Spaces.
- **AR:** كتبت ملف `app.py` مستقل بيحمّل كل المخرجات مرة واحدة عند التشغيل، وبيعرض واجهة Gradio فيها سلايدرز لضبط عدد النتائج — ده هو نفس الكود المنشور فعلياً على Hugging Face.

### Step 7.4: Local FastAPI Testing | الاختبار المحلي
- **EN:** Used `TestClient` to hit the `/` (health check) and `/search` endpoints locally before deployment, confirming Status Code 200 on both and validating the JSON response shape.
- **AR:** استخدمت `TestClient` لاختبار الـ endpoints محلياً قبل النشر، وأكدت استجابة 200 على الاتنين وشكل الـ JSON صحيح.

### Final Step: Upload to Hugging Face Spaces | الرفع النهائي
- **EN:** Uploaded the `deployment_artifacts` folder, `app.py`, and `requirements.txt` to the Space using the `huggingface_hub` API.
- **AR:** رفعت مجلد المخرجات وملف التطبيق وملف المتطلبات على الـ Space باستخدام مكتبة `huggingface_hub`.
- ⚠️ **ملحوظة أمان | Security note:** الخلية دي فيها Token مكتوب صريح في الكود — لازم يتغيّر ويتحط بدل منه في Colab Secrets، زي ما اتكلمنا قبل كده.

---

## 🧾 خلاصة عامة | Overall Summary

- **EN:** The notebook covers a complete, methodologically sound ML lifecycle: cleaning → leak-free splitting → baseline comparison → fine-tuning → threshold calibration → two-stage retrieval architecture → multi-angle evaluation (classification + retrieval + latency) → real deployment. The main technical gap remaining is the low Hit Rate@1/MRR, which points to the Cross-Encoder needing its own fine-tuning rather than being used zero-shot.
- **AR:** النوتبوك بيغطي دورة حياة مشروع Machine Learning كاملة ومنهجياً سليمة: تنظيف ← تقسيم بدون تسريب ← مقارنة Baseline ← تدريب ← معايرة عتبة ← معمارية استرجاع بمرحلتين ← تقييم متعدد الجوانب ← نشر حقيقي. الفجوة التقنية الأساسية المتبقية هي انخفاض Hit Rate@1 و MRR، وده بيوضح إن الـ Cross-Encoder محتاج Fine-tuning خاص بيه بدل ما يفضل Zero-shot.
