# Akıllı Enerji ve Karbon Takip Sistemi (el-lehim prototipi)

ESP32-WROOM-32E tabanlı, şebeke gerilimli (AC giriş/çıkış) enerji ölçüm prototipi: ACS712 akım
modülü, 5 V röle modülü, HLK-PM01 besleme, sigortalı AC girişi. Delikli (THT) yapıdadır ve
**elle lehimlenmek üzere** tasarlandı: şematikte her bağlantı etiketle değil çizili telle gösterilir.

> **Güvenlik:** Kartta şebeke gerilimi bölgesi vardır. Yalnızca yetkin kişilerce, uygun
> yalıtım ve koruma ile kullanılmalıdır.

## Doğrulama (2026-10-15, kicad-cli 10.0.4)

| Kontrol | Sonuç | Rapor |
|---|---|---|
| DRC | 0 ihlal, 0 bağlantısız | `reports/drc_2026-10-15.json` |
| ERC | 0 ihlal | `reports/erc_2026-10-15.json` |

## İçerik

- `board.kicad_{pro,sch,pcb}`, `block_diagram.kicad_sch` — KiCad 10 kaynakları
- `gerbers/`, `bom/`, `pnp/` — üretim çıktıları (2026-09-28; kart dosyası bundan eskidir, 2026-09-25). Bu kart için yeniden üretilmiş bir set yoktur.
- `docs/` — blok diyagram, veri sayfası ve mühendislik hesapları (xlsx/csv)
- `reports/` — DRC/ERC çıktıları
- `pin_baglanti_tablosu.md`, `PROJE_GEREKSINIMLERI.md` — tasarım notları

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

- Espressif ESP32-WROOM-32E: <https://www.espressif.com/sites/default/files/documentation/esp32-wroom-32e_esp32-wroom-32ue_datasheet_en.pdf>
- Allegro ACS712: <https://www.allegromicro.com/en/products/sense/current-sensor-ics/zero-to-fifty-amp-integrated-conductor-sensor-ics/acs712>
- Hi-Link HLK-PM01: <https://www.hlktech.net/>
- Songle SRD-05VDC-SL-C: <https://www.songlerelay.com/>
- Bulgin FX0456 sigorta yuvası: <https://www.bulgin.com/>
- TDK (FK26X7R1H105KRE06): <https://www.tdk-electronics.tdk.com/>
- MDD 1N5819: <https://www.mdd.com.tw/>

## Lisans

CERN-OHL-P-2.0 — bkz. kökteki `LICENSE`.
