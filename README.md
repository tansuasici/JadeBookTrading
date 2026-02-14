# JADE Book Trading - Coklu Ajan Kitap Ticaret Sistemi

JADE (Java Agent DEvelopment Framework) kullanilarak gelistirilmis, otonom yazilim ajanlarinin birbirleriyle pazarlik yaparak kitap alip sattigini gosteren bir **coklu ajan sistemi** (Multi-Agent System) projesidir.

## Proje Hakkinda

Bu proje, klasik bir web uygulamasi **degildir**. Burada merkezi bir sunucu yoktur; bunun yerine, birbirinden bagimsiz calisan **otonom ajanlar** FIPA (Foundation for Intelligent Physical Agents) standartlarina uygun mesajlasma protokolleri ile iletisim kurar, pazarlik yapar ve ticaret gerceklestirir.

### Ne Ogrenirsiniz?

- **Ajan tabanli programlama** (Agent-Oriented Programming) temellerini
- **JADE Framework** kullanimi ve ajan yasam dongusu
- **FIPA ACL** (Agent Communication Language) mesajlasma protokolunu
- **Davranis modelleri** (Behaviour): `TickerBehaviour`, `CyclicBehaviour`, `OneShotBehaviour`
- **Sari Sayfa servisleri** (Directory Facilitator) ile servis kesfini
- **Pazarlik protokolu**: CFP (Call for Proposal) → PROPOSE → ACCEPT_PROPOSAL → INFORM

## Mimari

```
┌─────────────────────────────────────────────────────────────┐
│                    JADE Platform (RMA)                       │
│                                                              │
│  ┌──────────────┐    FIPA ACL     ┌──────────────────────┐  │
│  │ BookBuyer    │◄──────────────►│ BookSeller            │  │
│  │ Agent        │                │ Agent                  │  │
│  │              │   1. CFP       │                        │  │
│  │  Hedef:      │──────────────►│  Katalog:              │  │
│  │  "Java 101"  │                │  ┌──────────┬───────┐ │  │
│  │              │   2. PROPOSE   │  │ Kitap    │ Fiyat │ │  │
│  │              │◄──────────────│  ├──────────┼───────┤ │  │
│  │              │                │  │ Java 101 │  50   │ │  │
│  │              │  3. ACCEPT     │  │ JADE Pro │  75   │ │  │
│  │              │──────────────►│  └──────────┴───────┘ │  │
│  │              │                │                        │  │
│  │              │   4. INFORM    │  ┌──────────────────┐  │  │
│  │              │◄──────────────│  │   Seller GUI     │  │  │
│  └──────────────┘                │  │  [Kitap] [Fiyat] │  │  │
│                                  │  │     [Ekle]       │  │  │
│         ┌─────────────────┐      │  └──────────────────┘  │  │
│         │ Directory       │      └──────────────────────┘  │
│         │ Facilitator     │                                 │
│         │ (Sari Sayfalar) │                                 │
│         └─────────────────┘                                 │
└─────────────────────────────────────────────────────────────┘
```

## Pazarlik Protokolu (Adim Adim)

Sistemin calisma mantigi su sekildedir:

```
Buyer Agent                    Directory Facilitator              Seller Agent(s)
     │                                │                                │
     │  1. "book-selling" ara          │                                │
     │───────────────────────────────►│                                │
     │                                │                                │
     │  2. Satici listesi don          │                                │
     │◄───────────────────────────────│                                │
     │                                                                 │
     │  3. CFP: "Java 101 var mi?"                                     │
     │────────────────────────────────────────────────────────────────►│
     │                                                                 │
     │  4a. PROPOSE: "Fiyat: 50 TL"  (kitap varsa)                    │
     │◄────────────────────────────────────────────────────────────────│
     │  4b. REFUSE: "Yok"            (kitap yoksa)                     │
     │◄────────────────────────────────────────────────────────────────│
     │                                                                 │
     │  [En dusuk fiyati sec]                                          │
     │                                                                 │
     │  5. ACCEPT_PROPOSAL: "Java 101 aliyorum"                       │
     │────────────────────────────────────────────────────────────────►│
     │                                                                 │
     │  6a. INFORM: "Satis tamamlandi"                                 │
     │◄────────────────────────────────────────────────────────────────│
     │  6b. FAILURE: "Baskasina satildi"                               │
     │◄────────────────────────────────────────────────────────────────│
```

## Proje Yapisi

```
BookTrading/
├── src/booktrading/
│   ├── BookBuyerAgent.java      # Alici ajan - kitap arar ve satin alir
│   ├── BookSellerAgent.java     # Satici ajan - katalog tutar, satis yapar
│   ├── BookSellerGui.java       # Satici icin Swing GUI arayuzu
│   └── BookTrading.java         # Ana sinif (giris noktasi)
├── dist/
│   ├── BookTrading.jar          # Derlenmmis JAR dosyasi
│   └── lib/
│       ├── jade.jar             # JADE framework kutuphanesi
│       └── commons-codec-1.3.jar
├── build.xml                    # Apache Ant build dosyasi
└── nbproject/                   # NetBeans IDE yapilandirmasi
```

## Sinif Detaylari

### BookBuyerAgent

Otonom olarak kitap arayan ve satin alan ajan.

| Ozellik | Aciklama |
|---------|----------|
| **Davranis** | `TickerBehaviour` - her 60 saniyede satici arar |
| **Strateji** | Tum saticilara CFP gonderir, en ucuz teklifi kabul eder |
| **Giris Parametresi** | Aranacak kitap adi (ornegin `"Java 101"`) |
| **Sonlanma** | Basarili alisveristen sonra otomatik kapanir |

**Anahtar Davranislar:**
- `TickerBehaviour`: Periyodik olarak Directory Facilitator'dan satici listesini gunceller
- `RequestPerformer` (inner class): 4 adimli durum makinesi (state machine) ile pazarlik yurutur

### BookSellerAgent

Kitap katalogunu yoneten ve satis yapan ajan.

| Ozellik | Aciklama |
|---------|----------|
| **Katalog** | `Hashtable<String, Integer>` - kitap adi → fiyat eslesmesi |
| **Servis Kaydi** | Directory Facilitator'a `"book-selling"` olarak kaydolur |
| **GUI** | `BookSellerGui` ile kullanicidan kitap ekleme |

**Anahtar Davranislar:**
- `OfferRequestsServer` (`CyclicBehaviour`): CFP mesajlarina fiyat teklifi veya red ile cevap verir
- `PurchaseOrdersServer` (`CyclicBehaviour`): Satin alma siparislerini isler, kitabi katalogdan cikarir

### BookSellerGui

Satici ajan icin Swing tabanli grafik arayuz.

- Kitap adi ve fiyat giris alanlari
- "Add" butonu ile kataloga kitap ekleme
- Pencere kapatildiginda ajan sonlanir

## Gereksinimler

| Gereksinim | Surum |
|------------|-------|
| **Java JDK** | 1.8 veya ustu |
| **Apache Ant** | 1.8.0+ (derleme icin) |
| **JADE** | 4.5.0 (dahil edilmistir) |

## Kurulum ve Calistirma

### 1. Projeyi Derleyin

```bash
cd BookTrading
ant clean build
ant jar
```

### 2. JADE Platformunu Baslatin

```bash
cd BookTrading/dist

# Yontem 1: Satici ve alici ajanlari ayri ayri olusturun
java -cp "lib/jade.jar:lib/commons-codec-1.3.jar:BookTrading.jar" \
  jade.Boot -gui

# Yontem 2: Satici ve alici ajanlari komut satirindan baslatin
java -cp "lib/jade.jar:lib/commons-codec-1.3.jar:BookTrading.jar" \
  jade.Boot -gui \
  "satici1:booktrading.BookSellerAgent;satici2:booktrading.BookSellerAgent"
```

### 3. Alici Ajan Ekleyin

JADE RMA (Remote Management Agent) arayuzunden:

1. **Actions** → **Start New Agent** secin
2. Agent Name: `alici1`
3. Class Name: `booktrading.BookBuyerAgent`
4. Arguments: `Java 101` (aramak istediginiz kitap adi)

Ya da komut satirindan dogrudan:

```bash
java -cp "lib/jade.jar:lib/commons-codec-1.3.jar:BookTrading.jar" \
  jade.Boot -gui \
  "satici1:booktrading.BookSellerAgent;alici1:booktrading.BookBuyerAgent(Java 101)"
```

### 4. Satici GUI'den Kitap Ekleyin

Satici ajanin GUI penceresi acilacaktir:
1. **Book title** alanina kitap adini girin (ornegin `Java 101`)
2. **Price** alanina fiyat girin (ornegin `50`)
3. **Add** butonuna tiklayin

Alici ajan 60 saniye icerisinde bu kitabi bulacak ve otomatik olarak satin alma surecini baslatacaktir.

## Ornek Calisma Senaryosu

```
# Terminal Ciktisi:

Seller-agent satici1@192.168.0.100:1099/JADE is ready.
Java 101 inserted into catalogue. Price = 50

Hallo! Buyer-agent alici1@192.168.0.100:1099/JADE is ready.
Target book is Java 101
Trying to buy Java 101
Found the following seller agents:
satici1@192.168.0.100:1099/JADE
Java 101 successfully purchased from agent satici1@192.168.0.100:1099/JADE
Price = 50
Buyer-agent alici1@192.168.0.100:1099/JADE terminating.
```

## Temel Kavramlar Sozlugu

| Kavram | Aciklama |
|--------|----------|
| **Agent** | Otonom, hedef odakli yazilim varligi |
| **Behaviour** | Ajanin gerceklestirdigi gorev/davranis birimleri |
| **ACL Message** | Ajanlar arasi iletisim mesaji (FIPA standardi) |
| **CFP** | Call for Proposal - teklif cagrisi |
| **PROPOSE** | Fiyat teklifi yaniti |
| **ACCEPT_PROPOSAL** | Teklif kabulU |
| **INFORM** | Islem basarili bildirimi |
| **REFUSE** | Teklif reddi (urun yok) |
| **FAILURE** | Islem basarisiz (urun satilmis) |
| **Directory Facilitator (DF)** | Ajan servislerinin kayitli oldugu sari sayfa servisi |
| **RMA** | Remote Management Agent - JADE platform yonetim arayuzu |
| **FIPA** | Foundation for Intelligent Physical Agents - ajan standartlari |
| **MAS** | Multi-Agent System - coklu ajan sistemi |

## Kaynaklar

- [JADE Resmi Sitesi](https://jade.tilab.com/)
- [JADE Programlama Kilavuzu](https://jade.tilab.com/doc/programmersguide.pdf)
- [FIPA Standartlari](http://www.fipa.org/specifications/)

## Lisans

Bu proje JADE orneklerini temel almaktadir ve **GNU Lesser General Public License v2.1** altinda lisanslanmistir.
