# 🤖 Yapay Zeka (AI) Destekli Geliştirme Süreci (AI.md)

Bu dosya, **2D Canvas Survival Game** projesinin geliştirilme aşamalarında yapay zeka (AI) ile yapılan teknik fikir alışverişlerini, karşılaşılan problemlerin çözüm süreçlerini ve mühendislik kararlarının arkasındaki mantığı belgelemek amacıyla hazırlanmıştır.

---

## 📌 Aşamalar ve Yapay Zekaya Yöneltilen Sorular

### Soru 1: HTML5 Canvas üzerinde sonsuz harita hissi veren bir kamera sistemini harici kütüphane kullanmadan nasıl kurabilirim?

**Yapay Zeka Yanıtı:**
Sonsuz harita hissi yaratmanın en performanslı yolu, oyuncuyu ekranın tam merkezinde sabit tutmak ve dünyadaki diğer tüm nesneleri (düşmanlar, yemler, mermiler) oyuncunun hareket miktarının tersi yönünde kaydırarak çizmektir (Camera Offset).

Bunun için oyuncunun konumuna bağlı olarak bir kamera ofseti hesaplayan fonksiyonlar yazılmalıdır:
```javascript
function camX() { return player.x - canvas.width / 2; }
function camY() { return player.y - canvas.height / 2; }
```
Çizim yaparken, nesnelerin gerçek dünyadaki x ve y koordinatlarından bu kamera değerleri çıkarılır:
```javascript
// Örnek: Düşmanı kameraya göre çizme
ctx.arc(m.x - camX(), m.y - camY(), 12, 0, Math.PI * 2);
```
Böylece oyuncu dünyada hareket ettikçe, kamera onu takip ediyormuş gibi bir illüzyon oluşur.

### Soru 2: JavaScript'te, oyuncu ateş ettiğinde haritadaki en yakın düşmanı otomatik bulup o yöne doğru mermi fırlatacak algoritmayı nasıl yazarım?
**Yapay Zeka Yanıtı:**
En yakın düşmanı bulmak için tüm aktif düşmanların listesini (mobs dizisi) tarayıp her birinin oyuncuya olan uzaklığını Öklid Mesafesi formülüyle hesaplamamız gerekir. JavaScript'teki Array.prototype.reduce() fonksiyonu bu tür bir "en küçüğü bulma" işlemi için oldukça uygundur.

Çözüm Algoritması:

İki nokta arasındaki mesafeyi bulan yardımcı fonksiyon:

```javascript
function dist(a, b) {
    let dx = a.x - b.x, dy = a.y - b.y;
    return Math.sqrt(dx * dx + dy * dy);
}
```
Ateş etme anında en yakın hedefi seçme ve mermiyi o yöndeki birim vektöre göre hızlandırma:

```javascript
let target = mobs.reduce((a, b) => dist(player, a) < dist(player, b) ? a : b);

let dx = target.x - player.x;
let dy = target.y - player.y;
let d = Math.sqrt(dx * dx + dy * dy); // Mesafe

dx /= d; // X yönündeki birim vektör
dy /= d; // Y yönündeki birim vektör

bullets.push({
    x: player.x,
    y: player.y,
    vx: dx * 8, // Mermi hızı çarpanı
    vy: dy * 8,
    life: 100
});
```
### Soru 3: Düşmanların (mobs) oyuncunun tam üzerinde doğmasını engellemek ve ekranın dışındaki rastgele yönlerden gelmelerini sağlamak için nasıl bir matematiksel yaklaşım izlemeliyiz?
**Yapay Zeka Yanıtı:**
Düşmanları doğrudan rastgele X ve Y koordinatlarında doğurursanız, bazen oyuncunun çok yakınında doğup haksız ölümlere yol açabilirler. Bunun yerine Kutupsal Koordinat Sistemi (Polar Coordinates) kullanarak oyuncunun etrafında hayali bir güvenli çember şeridi oluşturmalıyız.

Uygulama:

Math.random() * Math.PI * 2 ile 0-360 derece arasında rastgele bir açı (a) seçilir.

150 + Math.random() * 100 ile oyuncudan en az 150, en fazla 250 piksel uzaklıkta bir mesafe (d) belirlenir.

Trigonometrik fonksiyonlar (Math.cos ve Math.sin) kullanılarak bu açı ve mesafe, kartezyen koordinatlara dönüştürülüp oyuncunun konumuna eklenir:

```javascript
function spawnMob() {
    let a = Math.random() * Math.PI * 2; // Rastgele yön
    let d = 150 + Math.random() * 100;   // Güvenli mesafe şeridi
    mobs.push({
        x: player.x + Math.cos(a) * d,
        y: player.y + Math.sin(a) * d,
        hp: 3
    });
}
```
### Soru 4: Oyunda her 5 yem toplandığında açılan bir "şans çarkı (Roll)" mekaniği kurmak istiyorum. State (durum) yönetimini ve buff/debuff sistemini oyunu bozmadan nasıl entegre edebilirim?
**Yapay Zeka Yanıtı:**
Oyunun akışını kontrol etmek için bir gameState değişkeni (State Machine) kullanmalıyız. Durumlar: "menu", "playing" ve "roll" olabilir.

Durum Değişimi: Oyuncu 5 yem topladığında gameState = "roll" yapılır.

Oyun Döngüsü Kontrolü: update() fonksiyonunun başına şu kontrol eklenerek durum "playing" değilse arka plandaki hareketlerin ve çarpışmaların durması sağlanır:

```javascript
if (gameState !== "playing") return;
```
Buff Uygulama: Rastgele seçilen bir renge göre oyuncunun hızı yüzdesel olarak artırılır/azaltılır veya mermi gücü değiştirilir:

```javascript
function applyBuff(c) {
    player.color = c;
    if(c === "green") player.speed *= 1.1;  // %10 Hız artışı
    if(c === "red") player.speed *= 0.92;   // %8 Hız düşüşü
    if(c === "blue") bulletPower++;         // Mermi gücü +1
    if(c === "purple") bulletPower = Math.max(1, bulletPower - 1);
}
```
### Soru 5: Oyuncu oyunu kapatıp açtığında en yüksek skorunun (High Score) silinmemesini nasıl sağlarım? Veritabanı kurmaya gerek var mı?
**Yapay Zeka Yanıtı:**
Küçük çaplı tarayıcı oyunlarında bu işlem için karmaşık bir veritabanı (SQL/NoSQL) kurmaya gerek yoktur. Tarayıcının yerel hafıza çözümü olan localStorage bu iş için biçilmiş kaftandır. localStorage, verileri anahtar-değer (key-value) ilişkisiyle kullanıcının tarayıcısında kalıcı olarak saklar.

Oyun başlarken skoru hafızadan okuma:

```javascript
let highScore = localStorage.getItem("highScore") || 0;
```
Oyun bittiğinde eğer yeni skor eskisinden büyükse hafızaya kaydetme:

```javascript
if (score > highScore) {
    highScore = score;
    localStorage.setItem("highScore", highScore);
}
```
