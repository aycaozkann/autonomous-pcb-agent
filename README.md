# pcb-agent — kart vitrini

**KiCad üzerinde çalışan, yapay zeka destekli PCB tasarım ajanı. Her adımı ölçer, eksiğini raporlar.**

Bu depo ajanın ürettiği **kartları ve sonuçlarını** gösterir. Ajanın kendisi (araç seti) burada yok.

> **Neden private:** Araç seti, üçüncü taraf üretici belgeleri ve ekip çalışmasından gelen geçmiş içerdiği için şimdilik private; kartlar ve sonuçları açık.

## Kartlar

| Kart | Tür | DRC / ERC (2026-10-15, kicad-cli 10.0.4) |
|---|---|---|
| [`kartlar/sht35-breakout`](kartlar/sht35-breakout) | 2 katmanlı SMD breakout, Sensirion SHT35 | 0 ihlal, 0 bağlantısız / 0 ihlal |
| [`kartlar/akilli-enerji-karbon-takip`](kartlar/akilli-enerji-karbon-takip) | Elle lehimlenen THT prototip, ESP32 + AC ölçüm | 0 ihlal, 0 bağlantısız / 0 ihlal |

Her kart klasöründe KiCad kaynakları, Gerber/delik, BOM, CPL, DRC/ERC raporları ve (varsa) şematik/blok
diyagram/hesap belgeleri bulunur. Rapor dosyaları klasörlerin `reports/` altındadır; kendiniz de
`kicad-cli pcb drc` / `kicad-cli sch erc` ile yeniden üretebilirsiniz.

## Araç seti ne yapar (kısaca)

- Şematik ve PCB üzerinde ERC/DRC (`kicad-cli` sarmalayıcısı) ile ek geometri, DFM ve IPC kontrollerini kapı olarak koşar.
- Çalışmayan veya kapsam dışı kalan bir kontrolü "geçti" saymaz; `NO_COVERAGE` olarak raporlar.
- Elle lehimlenecek şematiklerde, bağlantının etiketle değil gerçek telle çizilmesini zorlar.
- Gerber, BOM, CPL ve STEP çıktıları üretir.
- Test paketi: 2026-10-15 tam koşumunda 4022 test: 3930 geçti, 0 başarısız, 0 hata, 92 atlandı (atlananlar ortam bağımlı: canlı KiCad IPC, `pcbnew`, symlink izni vb. — nedenleri tek tek kaydedilmedi). Windows 10, Python 3.14, KiCad 10.0.4.

Ajan bir LLM oturumu (Claude Code) tarafından sürülür; araç seti her adımı ölçer. Kararlar ve
fiziksel doğrulama (montaj, bring-up) insana aittir.

## Sınırlamalar

Bilinen sorunlar (düzeltilmedi, olduğu gibi kayıtlı):

1. **`--schematic-parity` boş PASS verebilir.** DRC kapısı geçici klasöre yalnızca `.kicad_pcb`
   (+ `.kicad_pro`) kopyalıyor, `.kicad_sch`'yi kopyalamıyor. Şematik olmadan KiCad parity listesini
   boş döndürüyor; yani "0 parity farkı — PASS" şematik–PCB uyumunun kanıtı **değildir**. Aynı kartta
   `kicad-cli` şematikle 35 uyarı verirken yalnız PCB ile 0 veriyor.
2. **El-lehim tel kapısı alt sayfalardaki etiket bağlantısını görmüyor.** `etiketle-baglama`
   hata fikstürü, kapıya `--force` verilince **GREEN** çıkıyor (tel=0, etiket=0): kapı yalnızca kök
   sayfayı ölçüyor. Bu fikstür beklenen RED'i göstermez.

Diğer sınırlar:

- Araç seti tek başına kart üretmez; kart-özel betikleri bir LLM oturumu yazar. Karmaşık kartlarda
  yönlendirme ihlalsiz kapanmayabilir (döngü bu durumda "insan gerekli" der).
- Empedans/stackup çözücü sign-off değildir.
- JLCPCB'nin online DFM kontrolü otomatik değil, manuel adımdır.
- Yukarıdaki iki kart basittir ve fiziksel olarak üretilip ölçülmüş değildir (bring-up kaydı yok).

## Lisans

Kart tasarımları **CERN-OHL-P-2.0** altındadır (`LICENSE`). KiCad kütüphane notu ve üretici belge
bağlantıları her kartın kendi README'sindedir.

## İletişim

📬 [ozkanayca8@gmail.com](mailto:ozkanayca8@gmail.com) · [LinkedIn](https://www.linkedin.com/in/ay%C3%A7a-%C3%B6zkan-0a2265195)
