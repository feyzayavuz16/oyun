# 🕹️ 2D Canvas Survival Game

Bu proje, **HTML5 Canvas** ve **Pure (Vanilla) JavaScript** kullanılarak geliştirilmiş, tarayıcı tabanlı bir 2D hayatta kalma ve nişancı oyunudur. Harici hiçbir oyun motoru veya kütüphane kullanılmadan, tamamen asgari web teknolojileri ve matematiksel algoritmalarla inşa edilmiştir.

## 🚀 Öne Çıkan Özellikler

* **Dinamik Kamera (Camera Tracking):** Oyuncuyu ekranın merkezinde tutan ve dünyayı oyuncunun hareketine göre kaydıran dinamik kamera sistemi.
* **Akıllı Hedefleme Sistemi:** Ateş tetiklendiğinde en yakındaki düşmanı Öklid bağıntısı kullanarak bulan ve mermiyi o yöne yönlendiren otomatik nişan mekanizması.
* **Roguelike Roll (Şans) Sistemi:** Her 5 yem toplandığında tetiklenen, oyuncuya pozitif veya negatif etkiler (Buff/Debuff) veren rastgele ödül mekanizması.
* **Kalıcı Yüksek Skor (High Score):** Tarayıcının `localStorage` API'si kullanılarak oyuncunun en yüksek skorunu hafızada tutma yeteneği.
* **Asenkron Ses Yönetimi:** Arka plan müziği ve oyun içi aksiyonlara (ateş, isabet, toplama) duyarlı ses efektleri.

## 🎮 Oynanış ve Kontroller

* **Hareket:** Fareyi (Mouse) ekran üzerinde hareket ettirerek karakterinizi yönlendirebilirsiniz. Karakteriniz farenin olduğu konuma doğru ilerler.
* **Ateş Etme:** Ekrana **Sol Tıklayarak (Left Click)** en yakındaki düşmana ateş edebilirsiniz.
* **Amaç:** Kahverengi düşmanlar size ulaşmadan önce onları vurun, öldüklerinde bıraktıkları pembe yemleri toplayarak skorunuzu artırın ve hayatta kalın!

### 🎲 Şans Çarkı Etkileri (Roll)
Her 5 yem topladığınızda karşınıza çıkan çarktan şu özellikler gelebilir:
* 🟢 **Yeşil Kare:** Hızınız %10 artar.
* 🔴 **Kırmızı Kare:** Hızınız %8 azalır.
* 🔵 **Mavi Kare:** Mermi gücünüz +1 artar.
* 🟣 **Mor Kare:** Mermi gücünüz -1 azalır (Minimum 1).

## 🛠️ Teknik Detaylar & Algoritmalar

### En Yakın Düşman Tespiti (Vektör Matematiği)
Oyuncu ateş ettiğinde, tüm aktif düşman dizisi (`mobs`) `Array.prototype.reduce()` fonksiyonu ile taranarak oyuncuya en kısa mesafedeki düşman belirlenir:

```javascript
let target = mobs.reduce((a, b) => {
    return dist(player, a) < dist(player, b) ? a : b;
});
