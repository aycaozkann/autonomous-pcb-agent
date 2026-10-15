# Akıllı Enerji Yönetimi ve Karbon Takip Sistemi

## Amaç

Prize takılı ancak kullanılmayan (bekleme modundaki / vampir güç çeken)
elektronik cihazların gereksiz akım tüketimini tespit eden, bu israfı anlık
ölçen ve gerektiğinde röle üzerinden fiziksel olarak gücü kesen entegre bir
akıllı sistem. Ayrıca tasarruf edilen enerjinin önlenen CO2 emisyonu
cinsinden karşılığını hesaplayıp kullanıcıya gösterir (görselleştirme —
uygulama/bulut tarafı, bu kart yalnızca ölçüm + kesme + Wi-Fi veri aktarımını
yapar).

## Ana Bileşenler

| Bileşen | Fonksiyon |
|---|---|
| ESP32-WROOM-32 | MCU, Wi-Fi |
| ACS712-05B (5A) | AC akım sensörü — bekleme-modu/vampir güç ölçümü için çözünürlük öncelikli (185mV/A) |
| SRD-05VDC-SL-C röle modülü | Optokuplörlü, 5V bobin, fiziksel güç kesme |
| HLK-PM01 | 220V AC → 5V DC izoleli güç kaynağı |
| AMS1117-3.3 | 5V → 3.3V LDO (ESP32 beslemesi) |
| 2 pin 5.08mm terminal blok × 2 | AC Giriş, AC Çıkış/Yük |
| Decoupling kapasitörleri | ESP32 için 10uF + 100nF, AMS1117 giriş/çıkış |

## Netlist / Mantıksal Bağlantılar

**AC Giriş:** AC Giriş Terminali (L, N) → HLK-PM01 AC giriş pinleri.

**DC Güç Dağıtımı:**
- HLK-PM01 5V çıkış → Röle VCC ve ACS712 VCC
- HLK-PM01 5V çıkış → AMS1117-3.3 → ESP32 3V3
- Tüm GND'ler ortak.

**Yüksek Gerilim Anahtarlama ve Ölçüm Yolu:**
- AC Giriş (L) → Röle COM
- Röle NO → ACS712 IP+
- ACS712 IP− → AC Çıkış (L)
- AC Giriş (N) → AC Çıkış (N) doğrudan

**Veri Sinyalleri:**
- ESP32 GPIO_12 (veya boş bir GPIO) → Röle Sinyal (IN)
- ACS712 VOUT → ESP32 ADC1_CH0 (GPIO_36 veya ADC özellikli benzer pin)

## Routing / DRC Kısıtları (KRİTİK)

- **İzolasyon (creepage/clearance):** Yüksek gerilim (220V AC) bölümü ile
  düşük gerilim (3.3V/5V DC) bölümü arasında en az 2.5mm (tercihen 3mm)
  boşluk.
- **İz genişliği:** AC güç izleri ve yük yolu en az 2.0mm genişlikte.
- **Milling:** Röle ve HLK-PM01'in altına izolasyon yarıkları (Edge.Cuts /
  milling layer).
- **Bakır döküm:** GND poligonu yalnızca düşük gerilim DC bölgesinde; yüksek
  gerilim AC bölgesinde bakır döküm YOK.

## Kullanıcı Kararları (2026-09-24)

1. **ACS712 aralığı:** 5A versiyon (ACS712-05B) — bekleme-modu akımları
   (genelde <1A) için çözünürlük öncelikli.
2. **Üretim yöntemi: ELLE LEHİM (perfboard/prototip).** Bu, klasör adının
   `-prototip` son ekiyle işaretlenmesine ve
   `pcb-agent/tools/scripts/sch/hand_solder_gate.py`'nin bu proje için
   otomatik olarak (folder-hint ile, `--force` gerekmeden) çalışmasına yol
   açtı. Kalıcı kural (global `CLAUDE.md`): her sinyal kaynağından hedefine
   kesintisiz, görünür bir **tel** ile çizilmeli; etiket telin yerine
   geçemez. İstisna yalnız çok-noktalı ortak düğümler (GND/VCC → güç sembolü
   + kısa tel).
   - **Güvenlik notu (agent → kullanıcı):** Bu kart 220V AC şebeke hattı
     taşıyor. Elle lehimlenen bir perfboard üzerinde şebeke gerilimi
     bulunması, fabrikasyon bir PCB'ye göre daha yüksek risk taşır (yalıtım
     kalitesi, iz/tel mesafesi, lehim köprüsü riski elle üretimde daha az
     kontrollü). Şematik ve yerleşim aşamasında creepage/clearance ve
     yalıtım gereksinimleri **azaltılmayacak**, aksine perfboard'a özgü ek
     notlarla (yüksek gerilim bölgesinin fiziksel olarak ayrılması, kablo
     kalınlığı, konnektör gerilim değeri) desteklenecek.

## Açık / Varsayılan Bırakılan Sorular

3. Şebeke gerilimi bölgesi: kullanıcı spesifikasyonunda "220V" açıkça
   belirtilmiş → **220V/50Hz varsayılıyor**, teyit gerekirse Faz 1'de
   sorulacak.
4. Kutu/gövde (enclosure) kısıtı: henüz belirtilmedi → **varsayım: yüksek
   gerilim bölümü kapalı, iletken olmayan bir muhafazada olacak** (perfboard
   + 220V kombinasyonunda dokunulabilir yüzey bırakmamak için zorunlu kabul
   ediliyor). Kullanıcı aksini belirtmezse bu varsayımla ilerlenecek.
5. Görselleştirme/CO2 hesaplama: **bu kartın kapsamı dışında** varsayılıyor
   (donanım yalnızca ölçüm + kesme + Wi-Fi veri aktarımı yapar; CO2
   hesaplama/görselleştirme ayrı bir yazılım/bulut bileşenidir).

## Montaj Notu (Donanım — BOM Dışı)

Kartta 2 adet M3 montaj deliği var (`MOUNTING_HOLES`, `scripts/gen_pcb.py`
— (12, 102) ve (158, 8), 5mm keepout ile işaretli). Bu delikleri
sabitlemek için gereken **2× M3 vida + 2× M3 somun** bir elektronik
bileşen olmadığı için şematikte/BOM'da hiç görünmez — kartı kutuya/şaseye
monte ederken ayrıca temin edilmeli.

## Ortam Notu

- `mcp__kicad__*` MCP sunucusu bu oturumda bağlanamadı (CONNECTION_CLOSED).
  Bu depoda (`aycaozkann/pcb-agent`) şematik üretimi zaten MCP'ye bağımlı
  değil — `pcb-agent/tools/scripts/sch/` altındaki script tabanlı
  (`sch_wire.py`, `wireify.py`, `sch_route.py`, proje-yerel
  `scripts/common_sch.py`) boru hattı kullanılacak.
- `kicad-cli.exe` bulundu: `/c/Program Files/KiCad/10.0/bin/kicad-cli.exe`.

## Durum

Proje klasörü oluşturuldu (`akilli-enerji-karbon-takip-sistemi-prototip`).
Sıradaki adım: Faz 1 — datasheet analizi ve pin bağlama tablosu.
