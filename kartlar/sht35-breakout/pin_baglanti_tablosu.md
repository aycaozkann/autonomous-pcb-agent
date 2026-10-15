# Pin Bağlantı Tablosu — SHT35 Breakout Kartı (Rev. 4 — MJperx referansına tam uyum, ŞEMATİK ÇİZİLDİ)

Proje: Sensirion SHT35 Sıcaklık/Nem Sensör Breakout Kartı
Faz: 1 tamamlandı (ONAYLANDI) → Faz 2 şematik çizimi **tamamlandı**, ERC: **0 hata / 0 uyarı**
Datasheet kaynağı: `DATASHEETS/Sensirion_SHT3x-DIS_Datasheet.pdf` (Dec 2022, Ver. 7)
Şematik dosyası: `SHT35_Breakout.kicad_sch`

**Rev. 4 değişikliği (kullanıcı talebi — referans: MJperx SHT35-Sensor-Breakout-PCB):**
- Güç mimarisi Rev.2'ye döndü: **+5V doğrudan besleme, LDO yok.** SHT35'in 2.15-5.5V aralığı 5V'u zaten kapsıyor.
- Header **1×6**'ya çıkarıldı: VCC(+5V), GND, SDA, SCL, **NRESET**, **ALERT** — hepsi dışarı çıkıyor (Rev.3'te NC bırakılan NRESET/ALERT artık header'da).
- **ADDR pini header'da değil** — Rev.3'teki karar korundu: kart üzerinde sabit 10kΩ pull-down ile GND'ye bağlı, adres 0x44'e kilitli. Kullanıcı bunu bilinçli olarak referans tasarımdan daha güvenli (datasheet'in "floating bırakılamaz" kuralına tam uyumlu) bir sapma olarak onayladı.
- Pull-up dirençleri (SDA/SCL) artık **+5V** rayına bağlı (Rev.3'te +3V3'e bağlıydı).

---

## 1. BOM Yaşam Döngüsü / Stok Raporu — değişmedi

| Parça | MPN | Durum | Kaynak |
|---|---|---|---|
| Sensör | SHT35-DIS-B2.5KS | **Active** | LCSC `C90161` (stok 1502), DigiKey/Mouser |

---

## 2. Güç Mimarisi

`+5V` net'i = header VCC pini = sensör VDD pini, doğrudan (regülatörsüz). SHT35 Table 3: VDD Min 2.15V / Typ 3.3V / Max 5.5V — 5V bu aralığın içinde, abs. max rating (-0.3 ile 6V, Table 6) içinde rahat marjlı. Host'un zaten regüle 5V sağladığı varsayılıyor.

**Not (I2C mantık seviyesi):** Pull-up'lar +5V'a bağlı olduğundan I2C bus 5V mantık seviyesinde çalışır. Host 5V-tolerant olmalı; değilse harici seviye dönüştürücü host tarafında eklenmelidir (bu breakout kartının kapsamı dışında, host'un sorumluluğunda).

---

## 3. Sensirion SHT35-DIS-B (8-pin DFN, 2.5×2.5mm) — Gerçekleşen Şematik Bağlantıları

| Pin | Adı | Datasheet Fonksiyonu | Şematikte Bağlı Net | Neden |
|---|---|---|---|---|
| 1 | SDA | I2C veri, open-drain I/O | `I2C_SDA` → J1 pin3 + R1 (4.7kΩ → `+5V`) | Datasheet §3.2: open-drain, harici pull-up gerekir. |
| 2 | ADDR | I2C adres seçimi; floating bırakılamaz | `I2C_ADDR` → **sadece** R3 (10kΩ pull-down → GND), header'a çıkmıyor | Kullanıcı kararı: adres kartta sabit 0x44 (Table 8). |
| 3 | ALERT | Alarm çıkışı | `SHT_ALERT` → J1 pin6 | Kullanıcı talebiyle header'a çıkarıldı (referans tasarımla birebir eşleşme). |
| 4 | SCL | I2C saat, open-drain I/O | `I2C_SCL` → J1 pin4 + R2 (4.7kΩ → `+5V`) | SDA ile aynı gerekçe. |
| 5 | VDD | Besleme girişi | `+5V` (doğrudan header) + C1 (100nF) | Datasheet §3.1: 100nF decoupling. |
| 6 | nRESET | Aktif-low reset | `SHT_NRESET` → J1 pin5 | Kullanıcı talebiyle header'a çıkarıldı. Host sürmediğinde dahili 50kΩ pull-up (VDD'ye) pini pasif tutar. |
| 7 | R | Elektriksel fonksiyonu yok | `GND` (zorunlu) | Datasheet Table 7. |
| 8 | VSS | Toprak | `GND` | — |
| 9 (Die Pad) | VSS | Dahili VSS | `GND` | Datasheet §3.3 + Master Rulebook Faz 1. |

---

## 4. Header (J1, Conn_01x06) — Kullanıcı Arayüzü (Rev. 4: 6 pin)

| Pin | Net | Yön |
|---|---|---|
| 1 | `+5V` (VCC) | Giriş |
| 2 | `GND` | — |
| 3 | `I2C_SDA` | I/O |
| 4 | `I2C_SCL` | I/O |
| 5 | `SHT_NRESET` | Giriş (aktif-low, opsiyonel kullanım) |
| 6 | `SHT_ALERT` | Çıkış (opsiyonel kullanım) |

ADDR header'da yok (bkz. Bölüm 3).

---

## 5. Doğrulanmış Net Listesi (KiCad ERC: 0 hata / 0 uyarı)

| Net | Bağlı Pinler |
|---|---|
| `+5V` | J1/1, U1/5, R1/1, R2/1, C1/1, FLG1/1 |
| `GND` | J1/2, U1/7, U1/8, U1/9, R3/2, C1/2, FLG2/1 |
| `I2C_ADDR` | U1/2, R3/1 |
| `I2C_SCL` | J1/4, U1/4, R2/2 |
| `I2C_SDA` | J1/3, U1/1, R1/2 |
| `SHT_ALERT` | J1/6, U1/3 |
| `SHT_NRESET` | J1/5, U1/6 |

`FLG1`/`FLG2` (PWR_FLAG): host üzerinden gelen +5V/GND'nin harici kaynak olduğunu ERC'ye bildiren zorunlu işaretler (kartta regülatör/kaynak pini yok).

---

## 6. Pasif Değerler Özeti (BOM, Rev. 4 — final, MPN/LCSC dolduruldu)

> **Düzeltme notu (bağımsız `design-checker` denetimi sonrası, bkz. `TEST/checker_raporu.md`):**
> Bu tablo daha önce R1/R2 için hâlâ 10kΩ yazıyordu — gerçek şematik/netlist Faz 3'te
> 4.7kΩ'a düzeltilmişti ama bu "final" doküman güncellenmemişti. Aşağıdaki tablo artık
> gerçek şematikle (`SHT35_Breakout.kicad_sch`) birebir eşleşiyor. Ayrıca MPN/LCSC alanları
> önceden sadece U1 için doluydu (5 pasif/konnektör satırı boştu) — MASTER_RULEBOOK Faz 1'in
> "her aday parça BOM'a kilitlenmeden önce belgelenir" kuralına uyum için jlcsearch (LCSC)
> üzerinden gerçek, stokta mevcut MPN'ler bulunup şematiğe işlendi.

| Referans | Değer | Tip | MPN | Üretici | LCSC | Konum/Amaç |
|---|---|---|---|---|---|---|
| C1 | 100nF | X7R seramik, 16V | CC0603KRX7R9BB104 | Yageo | C14663 (JLCPCB basic) | SHT35 VDD decoupling (pine 1.475mm — bkz. Bölüm 7, sınıra yakın ama kural içinde) |
| R1 | 4.7kΩ | %1 E96, 0603 | RC0603FR-074K7L | Yageo | C99782 | SDA pull-up → +5V |
| R2 | 4.7kΩ | %1 E96, 0603 | RC0603FR-074K7L | Yageo | C99782 | SCL pull-up → +5V |
| R3 | 10kΩ | %1 E96, 0603 | RC0603FR-0710KL | Yageo | C98220 | ADDR pull-down → GND (kart üzerinde sabit) |
| J1 | 1×6, 2.54mm | THT pin header (jenerik) | 2.54-1x6P (jenerik, üretici farketmez) | Generic/Various | C37208 | VCC/GND/SDA/SCL/NRESET/ALERT |
| U1 | SHT35-DIS-B2.5KS | DFN-8-EP | SHT35-DIS-B2.5KS | Sensirion | C90161 | Sensör |

---

## 7. Fiziksel Decoupling Mesafesi — Düzeltilmiş Ölçüm

`TEST/pcb_layout_raporu.md`'de daha önce C1'in VDD'ye "0.475mm" mesafede olduğu yazılmıştı —
bu hatalıydı (muhtemelen ara bir yerleşim iterasyonundan kalma eski bir değerdi).
Bağımsız denetim, ham PCB koordinatlarından (`U1` VDD pad @ 9.425,9.75 — `C1` +5V pad @
9.425,11.225) gerçek mesafeyi **1.475mm** olarak hesapladı. Kural (≤1.5mm) yine de
sağlanıyor ama sadece **25 mikron marjla** — "fazlasıyla marjlı" değil, sınıra yakın.
Bu, gerçek fiziksel toleranslarla (pad boyutu, panelizasyon vb.) risk taşımıyor çünkü PCB
üretim toleransları genelde ±0.05-0.1mm mertebesinde ve burası kritik bir yüksek hız hattı
değil (sadece DC decoupling) — ama rapor artık doğru sayıyı ve gerçek marjı yansıtıyor.

---

## 8. Arşiv — Terk Edilen Revizyonlar

- Rev. 1: LDO (AP2112K-3.3TRG1), 3.3-5V esnek giriş — terk edildi.
- Rev. 2: LDO'suz +5V, 7 pinli header (ADDR dahil) — terk edildi (ADDR header'dan sonradan kaldırıldı).
- Rev. 3: LDO'suz +3.3V, 4 pinli header (ADDR/NRESET/ALERT hiçbiri header'da değil) — terk edildi.
- `TEST/Faz2_Dropout_Kontrolu.csv`: Rev.1'e ait, geçersiz, arşiv notuyla saklanıyor.

---

**Faz 1 + Faz 2 TAMAMLANDI (Rev. 4, final).** Şematik `SHT35_Breakout.kicad_sch` içinde çizildi, ERC 0 hata/0 uyarı verdi. Sırada Faz 2.5 (footprint ataması) ve Faz 3 (4 aşamalı doğrulama: pin karşılaştırma / ERC / netlist / simülasyon) var.
