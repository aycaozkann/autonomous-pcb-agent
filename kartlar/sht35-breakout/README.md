# SHT35 Breakout (Rev. 4)

Sensirion SHT35-DIS sıcaklık/nem sensörü için 2 katmanlı, 1×6 header'lı breakout kartı:
+5V besleme (LDO yok), I²C pull-up'ları, ADDR kartta 10 kΩ ile GND'ye sabitli (adres 0x44),
NRESET ve ALERT header'a çıkıyor.

## Doğrulama (2026-10-15, kicad-cli 10.0.4)

| Kontrol | Sonuç | Rapor |
|---|---|---|
| DRC | 0 ihlal, 0 bağlantısız | `reports/drc_2026-10-15.json` |
| ERC | 0 ihlal | `reports/erc_2026-10-15.json` |

## İçerik

- `SHT35_Breakout.kicad_{pro,sch,pcb}` — KiCad 10 kaynakları
- `gerbers/` — Gerber + delik dosyası (`SHT35_Breakout.drl`), bu depodaki kartın güncel hâlinden 2026-10-15'te üretildi
- `bom/`, `pnp/` — BOM ve yerleşim (CPL) dosyaları
- `docs/` — şematik PDF, kart görünümü PDF, STEP modeli
- `pin_baglanti_tablosu.md` — tasarım notları (içindeki `DATASHEETS/…` yolları depoda yoktur, aşağıdaki bağlantılara bakın)

## KiCad kütüphane notu

Bu kart KiCad 10 ile çizildi ve **standart KiCad sembol/footprint kütüphanelerini** kullanır
(kartla birlikte dağıtılmaz). Projeyi açarken KiCad'in kendi kütüphanelerinin kurulu olması
gerekir. Kütüphane tabloları (`fp-lib-table`) Windows'taki varsayılan kurulum yolunu
(`C:/Program Files/KiCad/10.0/...`) gösteriyor olabilir; başka bir sistemde
*Tercihler → Footprint Kütüphanelerini Yönet* ile kendi yolunuza çevirin.
KiCad kütüphaneleri kendi lisansları (CC-BY-SA 4.0 + istisna) altındadır; bu deponun
CERN-OHL-P-2.0 lisansı yalnızca kartın kendi tasarım dosyalarını kapsar.

## Datasheet bağlantıları

Üretici belgeleri bu depoya **kopyalanmadı**; güncel sürümler üreticilerin sitelerinde:

- Sensirion SHT35-DIS-B: <https://sensirion.com/products/catalog/SHT35-DIS-B>
- Yageo (CC0603KRX7R9BB104, RC0603FR-074K7L, RC0603FR-0710KL): <https://www.yageo.com/>

## Lisans

CERN-OHL-P-2.0 — bkz. kökteki `LICENSE`.
