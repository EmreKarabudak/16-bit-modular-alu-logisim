# 16-Bit Modüler ALU (Logisim)

> 🇬🇧 [English Documentation (README.md)](README.md)

Bu depo, **Logisim** üzerinde geliştirilen **16-bitlik Aritmetik Mantık Birimi (ALU)** tasarımını ve aşamalı gelişim sürecini içerir. Proje; temel aritmetik ve karşılaştırma çekirdeğinden (`ALU_v1`), Mantık Birimi (`LU`) içeren 8 işlemli ALU'ya (`ALU_v2`) ve `Shifter` alt devresi (`SLL`, `SRL`, `SRA`), 4-bit `OpCode` ile 16 girişli `MUX` ekleyen `ALU_v3` sürümüne geçişi modüler alt devrelerle gösterir.

---

## Proje Sürümleri ve Gelişim Süreci

### 🔹 `ALU_v1` — Aritmetik ve Karşılaştırma Çekirdeği
* Veri yolu genişletilmeden önce doğrulanmış temel referans sürüm olarak korunmuştur.
* **Arithmetic Unit (`AU`)** alt devresi üzerinden **4 temel işlemi** (`ADD`, `SUB`, `SLT`, `SLTU`) destekler.
* İkiye tümleyen çıkarma kontrolü (`Sub` ve `Arit_Mux`) `OpCode bit[0]` (LSB) üzerinden sürülür.

### 🔹 `ALU_v2` — 8 İşlemli Tam ALU
* Mevcut `AU` yapısının yanına **Logic Unit (`LU`)** (`AND`, `OR`, `XOR`, `NOR`) entegre edilmiştir.
* Çıkış seçimi 3-bit `OpCode` ile kontrol edilen **8 girişli Çoklayıcı (`MUX`)** yapısına genişletilmiştir.
* **Kontrol Mimarisi Değişikliği:** Çıkarma ve karşılaştırma işlemlerini mantıksal işlemlerden düzenli biçimde ayırmak için `Sub` ve `Arit_Mux` kontrolü `bit[0]` yerine **`bit[2]` (`1xx` grubu)** üzerine taşınmıştır.

### 🔹 `ALU_v3` — Kaydırma Birimli ve 16 Girişli MUX ALU (Güncel)
* `Input 1` üzerinden **1-bit `SLL`**, **1-bit `SRL`** ve **1-bit `SRA`** işlemlerini üreten **`Shifter`** alt devresi eklenmiştir.
* `OpCode` genişliği **4 bite** çıkarılmış ve çıkış seçimi **16 girişli `MUX`** üzerinden sağlanmıştır.
* `Sub` ve `Arit_mux` kontrol hatları 4-bitlik `Splitter` üzerinden `OpCode bit[2]` ile sürülmüştür.

---

## Mimari Genel Bakış (`ALU_v3`)

* **16-Bit Veri Yolu (`Input 1`, `Input 2`):** İki adet 16-bit giriş, Logisim tünelleri aracılığıyla alt devrelere temiz bir şekilde dağıtılır.
* **Arithmetic Unit (`AU`):**
  * **Toplama (`ADD`):** Standart 16-bit ikili toplama (`A + B`) yapar.
  * **İkiye Tümleyen Çıkarma (`SUB`):** `bit[2] = 1` olduğunda `Arit_mux`, `NOT` kapısından geçen terslenmiş `Input 2` (`~B`) hattını seçer; aynı anda `Sub` sinyali `C in = 1` vererek `A + (~B) + 1` işlemini gerçekleştirir.
  * **SLTU (Set Less Than Unsigned):** Çıkarma sırasında en anlamlı bit (MSB) aşamasından çıkan elde bitinin tersi (`~Cout`) üzerinden işaretsiz `A < B` sonucunu üretir.
  * **SLT (Set Less Than Signed):** İşaretli sayılarda çıkarma sonucunun işaret biti ile taşma bayrağını (`MSB XOR Overflow`) kullanarak `A < B` sonucunu üretir.
  * **Bit Genişletme:** 1-bitlik `Slt` ve `Sltu` çıkışları `Bit Extender` (`0..16 extend`) ile 16-bite genişletilerek ana `MUX`'a bağlanır.
* **Logic Unit (`LU`):**
  * `Input 1` ve `Input 2` girişleri üzerinden paralel olarak 16-bit bit bazında `AND`, `OR`, `XOR` ve `NOR` işlemlerini hesaplar.
* **Kaydırma Birimi (`Shifter` Alt Devresi):**
  * `Input 1` üzerinde Logisim `Splitter` bileşenleri kullanılarak paralel üç adet 16-bit çıkış üretilir:
    * **`SLL`:** Bitleri 1 bit sola kaydırır (`0..14` -> `1..15`) ve `0.` bite sabit `0` yerleştirir.
    * **`SRL`:** Bitleri 1 bit mantıksal olarak sağa kaydırır (`1..15` -> `0..14`) ve `15.` bite sabit `0` yerleştirir.
    * **`SRA`:** Bitleri 1 bit aritmetik olarak sağa kaydırır (`1..15` -> `0..14`) ve işaret bitini (`Input 1[15]`) `15.` bitte korur.

---

## OpCode Tablosu (`ALU_v3`)

| OpCode | Grup | İşlem | Açıklama |
| :--- | :--- | :--- | :--- |
| `0000` | `0xxx` | `ADD` | 16-bit toplama (`A + B`) |
| `0001` | `0xxx` | `AND` | Bit bazında VE (`A & B`) |
| `0010` | `0xxx` | `OR` | Bit bazında VEYA (`A \| B`) |
| `0011` | `0xxx` | `XOR` | Bit bazında Özel VEYA (`A ^ B`) |
| `0100` | `0xxx` | `SUB` | 16-bit ikiye tümleyen çıkarma (`A - B`) |
| `0101` | `0xxx` | `SLTU` | İşaretsiz küçüktür karşılaştırması (`A < B`) |
| `0110` | `0xxx` | `SLT` | İşaretli küçüktür karşılaştırması (`A < B`) |
| `0111` | `0xxx` | `NOR` | Bit bazında VEYA-DEĞİL (`~(A \| B)`) |
| `1000` | `1xxx` | `SLL` | `Input 1` için 1-bit mantıksal sola kaydırma |
| `1001` | `1xxx` | `SRL` | `Input 1` için 1-bit mantıksal sağa kaydırma |
| `1010` | `1xxx` | `SRA` | `Input 1` için 1-bit aritmetik sağa kaydırma |

---

## Test ve Doğrulama Senaryoları

`ALU_v1`, `ALU_v2` ve `ALU_v3` devreleri aritmetik, karşılaştırma, taşma (`Overflow`), mantıksal kapı ve kaydırma test senaryolarında doğrulandı:

| Senaryo | Girişler (`A`, `B`) | OpCode | Beklenen / Gözlemlenen Sonuç |
| :--- | :--- | :--- | :--- |
| **Pozitif Toplama** | `12`, `25` | `000` (`ADD`) / `0000` | `37` (`0x0025`), `Overflow = 0` |
| **İşaretli Toplama** | `-10`, `4` / `-10`, `-5` | `000` (`ADD`) / `0000` | `-6` (`0xFFFA`) / `-15` (`0xFFF1`), `Overflow = 0` |
| **İşaretli Taşma (Overflow) Sınırı** | `32767` (`0x7FFF`), `1` (`0x0001`) | `000` (`ADD`) / `0000` | `-32768` (`0x8000`), `Overflow = 1` |
| **Sıfır Çıkarma Sınır Durumu** | `0`, `0` | `100` (`SUB`) / `0100` | `0` (`0x0000`), `Overflow = 0` |
| **Küçükten Büyük Çıkarma** | `10`, `25` | `100` (`SUB`) / `0100` | `-15` (`0xFFF1`) |
| **İşaretli Çıkarma** | `49`, `-9` | `100` (`SUB`) / `0100` | `58` (`0x003A`) |
| **Eşit Sayı Karşılaştırması** | `20`, `20` | `101` (`SLTU`) / `110` (`SLT`) | `SLTU` ve `SLT` ikisi de `0` (`0x0000`) |
| **İşaretli vs. İşaretsiz Ayrımı** | `49` (`0x0031`), `-9` (`0xFFF7` / `65527`) | `110` (`SLT`) / `101` (`SLTU`) | `SLT = 0` (`49 < -9` yanlış), `SLTU = 1` (`49 < 65527` doğru) |
| **Bit Deseni Testi (`LU`)** | `0xFF00`, `0x0F0F` | `001` (`AND`), `010` (`OR`), `011` (`XOR`), `111` (`NOR`) | `AND = 0x0F00`, `OR = 0xFF0F`, `XOR = 0xF00F`, `NOR = 0x00F0` |
| **Uç Değerler (`LU`)** | `0x0000` / `0xFFFF` | `001`–`111` | `NOR` (`0x0000` -> `0xFFFF`, `0xFFFF` -> `0x0000`) dahil tüm kapılar doğrulandı |
| **Kaydırma Negatif MSB Testi (`ALU_v3`)** | `0x8006` (`1000 0000 0000 0110`) | `1000` (`SLL`), `1001` (`SRL`), `1010` (`SRA`) | `SLL = 0x000C`, `SRL = 0x4003`, `SRA = 0xC003` (`1` işaret biti korundu) |
| **Aritmetik Sağa Kaydırma Pozitif Testi (`ALU_v3`)** | `0x0C66` (`0000 1100 0110 0110`) | `1010` (`SRA`) | `0x0633` (`0000 0110 0011 0011`, `0` işaret biti korundu) |

---

## Çalıştırma ve İnceleme

1. **Logisim** uygulamasını açın.
2. Depodaki `.circ` proje dosyasını yükleyin.
3. Sol panelden `ALU_v1`, `ALU_v2` ve `ALU_v3` devreleri arasında geçiş yapın.
4. `Input 1`, `Input 2` ve `OpCode` pinlerini değiştirerek `Result` ve `Overflow` çıkışlarını gözlemleyin.

---

## Yol Haritası (`ALU_v4` — Planlanan)

* Çok kademeli kaydırma yapısıyla dışarıdan kontrollü kaydırma miktarı (`shamt`, 4-bit ile `0–15` arası) eklenmesi.
* Mevcut `SLT` / `SLTU` karşılaştırma çıkışlarını kullanan **`MAX` / `MIN`** seçim biriminin eklenmesi.
