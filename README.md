# YOLOv8 ile Masaüstü Nesne Tespiti Proje Raporu

## 1. Proje Özet Tablosu

| Alan | Açıklama / Öğrenci Yanıtı |
|---|---|
| **Project Title** | YOLOv8 ile Masaüstü Nesne Tespiti (Desktop Object Detection) |
| **Problem / Task** | Nesne Tespiti (Object Detection) |
| **Dataset** | COCO128 (COCO veri setinin 80 sınıf içeren 128 görüntülük alt kümesi) |
| **Dataset URL** | https://cocodataset.org |
| **Model Architecture** | YOLOv8n (Nano) - COCO ön eğitimli & COCO128 fine-tuned |
| **Input Size** | 640 x 640 x 3 |
| **Hyperparameters** | Epochs: 15, Batch Size: 16, Initial LR (lr0): 0.001 |
| **Optimizer** | AdamW |
| **Activation / Output Function** | SiLU (Gizli Katmanlar), Sigmoid (Sınıf Skorları) |
| **Loss Function** | Box Loss (CIoU) + Classification Loss (BCE) + Distribution Focal Loss (DFL) |
| **Evaluation Metrics** | IoU, Precision, Recall, mAP@0.5, mAP@0.5:0.95 |
| **Training Environment** | Google Colab (Tesla T4 GPU), PyTorch, Ultralytics |
| **Best Performance** | mAP@0.5: 0.885 | mAP@0.5:0.95: 0.724 | Precision: 0.862 | Recall: 0.841 |
| **Inference Time** | Tahmin Süresi: ~6.5 ms / görsel (GPU) \| Eğitim Süresi: ~95 saniye |
| **Deployment Technology** | Gradio (Web Interactive UI) |
| **Repository** | https://github.com/abdulsametkaratas/yolov8-desktop-object-detection |

---

## 2. Veri Seti Notları & Sınırlılıklar

* **COCO128 Sınırlılığı:** COCO128 veri setinde bağımsız bir doğrulama (validation) kümesi bulunmamaktadır; eğitim ve doğrulama aynı 128 görsel üzerinde yürütülmüştür. Bu durum elde edilen yüksek mAP değerlerinin modelin ezberleme yatkınlığını tam yansıtmayabileceğini gösterir.
* **Genelleştirme Testi:** Modelin gerçek hayat başarısını sınamak ve over-fitting durumunu gözlemlemek amacıyla model, 8. maddede belirtildiği üzere kendi akıllı telefonumla çektiğim 5 özgün masaüstü fotoğrafı ile test edilmiştir.

---

## 3. Gerçek Görüntü Testi (5 Telefon Fotoğrafı)

| # | Fotoğraf (Masada Ne Var) | Gerçekte Olanlar | Modelin Bulduğu (Gradio Çıktısı) | Doğru mu? | Yorum / Analiz |
|---|---|---|---|---|---|
| **1** | Kupa Bardak | Kupa / Bardak (Mug) | `bottle (0.60)`, `laptop (0.26)` | **Yanlış (Hatalı Sınıflandırma)** | Model kupa bardağı yanlışlıkla %60 ihtimalle `bottle` (şişe) olarak sınıflandırdı. Ayrıca arka plandaki koyu renk gölgeleri ve standı %26 ihtimalle `laptop` olarak tespit etti. Bardağın silindirik gövde yapısı şişe özniteliklerine benzediği için model karıştırmıştır. |
| **2** | Siyah Defter / Ajanda | Defter / Kitap (Book) | *Nesne bulunamadı* | **Başarısız (Missed Detection)** | Masadaki siyah defter model tarafından tespit edilemedi. COCO veri setindeki kitap (`book`) örnekleri genellikle renkli kapaklar veya açık sayfalar içerdiğinden, düz siyah kapaklı kapalı defter dokusu modelin eşik değerini (conf=0.25) geçememiştir. |
| **3** | Laptop ve Kablosuz Fare | Laptop, Mouse | `car (0.62)`, `laptop (0.32)`, `refrigerator (0.25)` | **Kısmen Doğru / Hatalı Sınıflandırma** | Model laptop ekranını %32 güvenle `laptop` olarak buldu. Ancak masadaki siyah kablosuz fareyi oval formu ve mat yapısı nedeniyle %62 olasılıkla `car` (araba) olarak yanlış tespit etti. Ayrıca arka plandaki priz/duvar kısmını %25 ihtimalle `refrigerator` zannetti. |
| **4** | Defter ve Tükenmez Kalem | Defter, Kalem | *Nesne bulunamadı* | **Başarısız (Sınıf Dışı & Eşik Altı)** | Görseldeki kalem, COCO veri setinin 80 sınıfı arasında yer almadığı için model tarafından doğal olarak atlanmıştır. Siyah defter ise 2. fotoğraftaki gibi belirgin öznitelik sunamadığından tespit edilememiştir. |
| **5** | Defter, Şişe ve Kalem | Defter, Su Şişesi, Kalem | `bottle (0.98)` | **Kısmen Doğru (Başarılı Şişe Tespiti)** | Şeffaf su şişesi %98 gibi yüksek bir güven skoruyla doğru bir şekilde `bottle` olarak tespit edilmiştir. Kalem (COCO'da tanımlı değil) ve siyah defter (doku/kontrast eksikliği) tespit edilememiştir. |

---

## 4. Gerçek Görüntülerle Değerlendirme (Başarı ve Başarısızlık Analizi)

### Başarılı Olduğu Durumlar:
1. **Belirgin Nesne Öznitelikleri:** Model, şeffaf/plastik dokuya ve net hatlara sahip pet su şişesini %98 gibi çok yüksek bir güven skoruyla doğru sınıflamış ve sınır kutusunu (bounding box) hassas biçimde çizmiştir.
2. **Temel Form Tespiti:** Laptop ekranı gibi COCO veri setinde sıkça temsil edilen nesnelerin genel kütlesini ve ekrandaki parlaklık kontrastını başarıyla tespit edebilmiştir.

### Başarısız Olduğu Durumlar ve Nedenleri:
1. **Sınıf Dışı Nesneler (Out-of-Vocabulary):** Kalem (`pen`) nesnesi COCO’nun 80 temel sınıfı arasında tanımlı bir kategori olmadığından modelin bu nesneyi tespit etmesi mümkün olmamıştır.
2. **Doku ve Kontrast Eksikliği (False Negatives):** Kapalı siyah ajanda/defter, COCO veri setindeki tipik `book` örneklerinden (renkli kitap kapakları, açık sayfalar) farklı olduğu ve masa yüzeyiyle düşük kontrast oluşturduğu için öznitelik haritasında (feature map) eşik değerini geçememiştir.
3. **Biçimsel Yanılsama ve Yanlış Sınıflandırma (False Positives):**
   * Mat siyah oval kablosuz fare, model tarafından ölçek farkından ötürü küçük bir otomobil siluetine benzetilerek %62 güven skoruyla `car` olarak etiketlenmiştir.
   * Kupa bardak, üst kısmı daralan silindirik yapısından dolayı model tarafından %60 ihtimalle `bottle` sınıfı ile karıştırılmıştır.