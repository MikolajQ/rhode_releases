# LineageOS 23.2 dla Motorola Moto G52 (rhode)

Nieoficjalny, spersonalizowany build LineageOS 23.2 (Android 16 QPR2) dla **Motorola Moto G52** (`rhode`,
Snapdragon 680 / SM6225), zbudowany od zera z bieżących źródeł. Bazuje na tym samym drzewie co
zoptymalizowane buildy [Tomoms](https://github.com/tomoms) dla tego urządzenia, ale dokłada własną warstwę
funkcji i zamienia część komponentów tam, gdzie ich upstream przestał być dostępny lub nie odpowiadał
naszym potrzebom (root, kontrola rodzicielska, prywatność sieciowa).

**Pobierz:** [najnowszy release](../../releases/latest) — zip ROM-u + `boot.img`, `dtbo.img`, `vendor_boot.img`.
**Jak zbudować własną kopię:** przepis źródłowy w [rhode-los23.2](https://github.com/MikolajQ/rhode-los23.2).

---

## Czego tu nie znajdziesz z domyślnego LineageOS

Standardowy LineageOS jest celowo minimalistyczny — czysty AOSP plus garść ulepszeń Lineage (Trebuchet,
Aperture, motyw). Ten build dokłada do tego trzy grupy rzeczy, których w oficjalnych buildach nie ma
w ogóle: **root**, **usługi Google w wersji okrojonej i naprawionej** oraz **blokowanie treści wbudowane
w system**. Do tego dziedziczy po drzewie źródłowym solidny pakiet optymalizacji wydajności i baterii,
opisany niżej.

### Root: KernelSU-Next

Oficjalny LineageOS nie ma roota. Ten build ma wkompilowane w jądro wsparcie dla
**[KernelSU-Next](https://github.com/KernelSU-Next/KernelSU-Next)** — nowocześniejszej gałęzi popularnego
KernelSU, działającej na poziomie jądra (nie jako modyfikacja `/system` jak starsze rozwiązania). Manager
KernelSU-Next **instaluje się osobno po flashu** — to jedyny element, który nie mieści się w samym obrazie
systemu z przyczyn technicznych (wymogi podpisu APK na Androidzie 16); root działa identycznie jak przy
instalacji systemowej. Jądro rozpoznaje manager po certyfikacie podpisu, więc obie oficjalne wersje z
[releases KernelSU-Next v3.3.0](https://github.com/KernelSU-Next/KernelSU-Next/releases/tag/v3.3.0) działają:

- `KernelSU_Next_v3.3.0_33214-release.apk` — standardowa,
- `KernelSU_Next_v3.3.0-spoofed_33214-release.apk` — **zalecana**: ta sama funkcjonalność i ten sam podpis,
  ale ukryta prawdziwa nazwa pakietu, żeby proste sprawdzenia roota w aplikacjach bankowych (szukające po
  nazwie pakietu managera) jej nie wykryły. Nie omija zaawansowanej weryfikacji integralności (Play
  Integrity) — to załatwia osobno wbudowany PIF.

### Usługi Google: lekki zestaw, ale z działającą Kontrolą rodzicielską i Android Auto

Zamiast pełnego pakietu Google Apps (setki MB Asystenta, Google TV, Wellbeing, itd.) build zawiera **okrojony
zestaw w stylu NikGapps core** — tylko to, co potrzebne żeby telefon normalnie działał z kontem Google:

- Google Play Services (GmsCore), Sklep Play, Google Services Framework,
- synchronizacja kontaktów i kalendarza z kontem Google,
- **Android Auto z pełną listą uprawnień** (71 uprawnień systemowych zamiast okrojonych 19 z typowych paczek
  GApps) — bezprzewodowe Android Auto działa od razu, bez grzebania w ustawieniach dewelopera,
- **w pełni działająca Kontrola rodzicielska Google (Family Link)** — naprawiony powszechny błąd, przez
  który konto dziecka wisi w nieskończoność na ekranie „Przygotowuję kolejne kroki" (przyczyna: usługa
  nadzoru instaluje się jako zwykła, nieuprzywilejowana aplikacja; ten build dostarcza ją jako
  uprzywilejowaną aplikację systemową, więc Google Play może ją poprawnie zaktualizować).

Z pełnego pakietu GApps świadomie wycięto (oszczędność ~385 MB): Asystenta Google, usługi mowy Google,
TalkBack, Digital Wellbeing, kopię zapasową Google Restore, integrację z kontaktami Exchange i dialerem
Google. Nic z tego nie jest potrzebne do normalnego działania konta Google i można to doinstalować ręcznie,
jeśli komuś zależy.

### Wbudowany bloker treści — dwie niezależne warstwy

W przeciwieństwie do zwykłych ROM-ów, gdzie blokowanie reklam/treści dla dorosłych wymaga osobnej aplikacji
(zwykle działającej jako VPN, zajmującej pamięć i slot VPN), tutaj filtrowanie jest wbudowane w system i nie
zużywa żadnych dodatkowych zasobów:

1. **Domyślny prywatny DNS (DoT) ustawiony na AdGuard DNS Family** — blokuje reklamy i trackery, treści dla
   dorosłych, wymusza bezpieczne wyszukiwanie i tryb ograniczony YouTube. Konfigurowalny w Ustawieniach
   (Sieć i internet → Prywatny DNS) — można zmienić na inny serwer albo wyłączyć.
2. **Lista `/system/etc/hosts` wpieczona w obraz systemu** — niezależna od ustawień DNS, działa nawet gdy
   ktoś wyłączy prywatny DNS. Blokuje strony dla dorosłych i media społecznościowe (Instagram, TikTok,
   Facebook, Snapchat, Reddit, Twitter/X…) oraz popularne komunikatory poza WhatsAppem i Signalem (Telegram
   w wersji web, Discord, Viber, Skype). **WhatsApp i Signal celowo pozostają w pełni funkcjonalne** — łącznie
   z transferem mediów, który inaczej skonfigurowana lista blokowałaby przypadkowo. Listę można rozszerzyć
   po zainstalowaniu roota (np. aplikacją AdAway).

### Droid-ify zamiast F-Droida

Zamiast oficjalnego klienta F-Droid (ciężki, toporny interfejs) build ma wbudowany
**[Droid-ify](https://github.com/Droid-ify/client)** — lżejszy, szybszy klient tych samych repozytoriów.
Od razu skonfigurowane trzy dodatkowe repozytoria — nie trzeba ich dodawać ręcznie:

- **[IzzyOnDroid](https://apt.izzysoft.de/fdroid/)** — duży katalog aplikacji spoza głównego repozytorium F-Droid,
- **[NewPipe](https://newpipe.net/)** — oficjalne repozytorium klienta YouTube/mediów bez reklam i śledzenia,
- **[IronFox](https://ironfoxoss.org/)** — przeglądarka mobilna skupiona na prywatności.

Droid-ify nie ma odpowiednika Privileged Extension (cichej instalacji jako uprzywilejowana aplikacja
systemowa) — aktualizacje instaluje przez uprawnienia roota (KernelSU-Next), więc przy pierwszym użyciu
trzeba mu przyznać dostęp w managerze roota.

Systemowy WebView (silnik, którego używają wszystkie aplikacje pokazujące strony internetowe wewnątrz
siebie) to standardowy, oficjalny prebuilt LineageOS — bez własnych modyfikacji.

### Własny kanał aktualizacji

Build ma niezależny kanał OTA (Ustawienia → Aktualizator) — aktualizacje przychodzą z tego repozytorium, nie
z buildów Tomomsa, więc dostajesz dokładnie tę konfigurację, nie inną.

---

## Rodzaje zastosowanych optymalizacji

Ten build dziedziczy po drzewie źródłowym szeroki zestaw optymalizacji wydajności, baterii i pamięci,
wykraczający poza to, co oferuje czysty AOSP/LineageOS. Poniżej — pogrupowane według warstwy systemu.

### Kompilator i jądro

- **ThinLTO** (Link-Time Optimization) dla jądra Linux — kompilator optymalizuje kod na poziomie całego
  jądra naraz, nie pojedynczych plików, co daje szybszy i mniejszy kod wynikowy.
- **Polly** — zaawansowany framework optymalizacji pętli (autovektoryzacja, poliedryczna optymalizacja
  planowania instrukcji) włączony w kompilacji jądra.
- **CFI (Control Flow Integrity)** — sprzętowe/kompilatorowe zabezpieczenie przed przejęciem kontroli nad
  przepływem programu w jądrze (ochrona przed konkretną klasą exploitów).
- **Simple LMK** zamiast standardowego `lmkd` — lżejszy, szybszy mechanizm zabijania procesów przy niskiej
  pamięci.
- **TEO (Timer Events Oriented)** — governor cpuidle dobierający głębokość uśpienia rdzeni CPU na podstawie
  nadchodzących zdarzeń czasowych, zamiast prostych heurystyk.
- Strojenie schedulera WALT: progi migracji zadań między rdzeniami wydajnymi/oszczędnymi (`sched_upmigrate`/
  `sched_downmigrate`), krzywe governora `schedutil` (`hispeed_freq`, `hispeed_load`) dobrane osobno dla
  wariantu SoC tego urządzenia.

### System i runtime aplikacji (ART)

- Aktualizacje środowiska uruchomieniowego ART (maszyna wirtualna Androida) prosto z gałęzi rozwojowej
  AOSP, wyprzedzające to, co trafia do oficjalnych wydań.
- Usunięte instrukcje debugowania/śledzenia z ART niepotrzebne na buildzie produkcyjnym — mniejszy narzut
  na każde uruchomienie aplikacji.
- Zoptymalizowane rutyny biblioteki `bionic` (libc Androida) — m.in. szybsza implementacja `fmodf` i
  natywna obsługa właściwości systemowych zamiast pośrednictwa przez wolniejsze warstwy.
- **jemalloc** jako domyślny alokator pamięci systemowej zamiast standardowego — zazwyczaj mniejsza
  fragmentacja pamięci i szybsze alokacje przy typowym obciążeniu aplikacjami mobilnymi.
- Strojenie dexopt (wstępnej kompilacji aplikacji) i wyłączenie generowania zbędnych metadanych debugowania
  przy pakowaniu obrazu systemowego.
- Masowe wycięcie zbędnego logowania (`logspam`) w kluczowych usługach systemowych (SystemUI, AppOps,
  menedżer procesów, silnik renderowania `hwui`, harmonogram zadań) — mniej pracy CPU na pisanie do bufora
  logów, którego i tak nikt nie czyta na co dzień.

### Wyświetlacz i bateria

- Adaptacyjne odświeżanie 90 Hz ze schodzeniem do 60 Hz po 500 ms bezczynności ekranu — płynność przy
  interakcji, oszczędność baterii przy statycznym obrazie.
- Zarządzanie energią radia Wi-Fi (tryby oszczędzania IMPS/BMPS) aktywne domyślnie.
- `zram` jako skompresowana pamięć wymiany (70% RAM) — więcej efektywnej pamięci operacyjnej bez fizycznego
  jej dokładania, kosztem niewielkiego obciążenia CPU przy kompresji.
- Strojenie wyprzedzającego odczytu (`readahead`) z pamięci masowej, dobrane pod konkretny typ nośnika w
  tym urządzeniu.

### Prywatność sieciowa (domyślne ustawienia)

- Serwery NTP, SUPL (asysta GPS) i URL-e portalu przechwytującego przełączone z Google na alternatywy
  szanujące prywatność (m.in. pool.ntp.org, serwery GrapheneOS) — telefon nie odpytuje infrastruktury
  Google przy każdym starcie i połączeniu z siecią, nawet bez konta Google.
- Wbudowana obsługa wielu dostawców prywatnego DNS (DNS-over-TLS) do wyboru w ustawieniach.
- **Play Integrity BASIC/DEVICE** przez wbudowany profil PropImitationHooks (świeży, nie-beta fingerprint
  Pixela — starsze, betowe profile Google od pewnego czasu odrzuca nawet na poziomie DEVICE). Poziom
  **STRONG** (sprzętowa atestacja) wymaga własnego, prawdziwego `keybox.xml` w Ustawieniach → Lineage
  Extras — bez niego apki wymagające STRONG mogą nie przechodzić certyfikacji.

---

## Instalacja

> **Build 20260918 nie nadaje się do instalacji** — pętla rozruchu (błędna nazwa pakietu WebView w konfiguracji;
> poprawione w źródłach, czeka na kolejny build). Zostawiony jako pre-release do celów historycznych.

Urządzenie ma Virtual A/B — **kolejność kroków ma znaczenie** (jest zgodna z [wiki LineageOS](https://wiki.lineageos.org/devices/devon/install/)):

1. Telefon w bootloaderze: `fastboot boot boot.img` (z tego samego release'u co zip) → uruchomi się recovery tego builda.
2. **Factory Reset → Format data / factory reset** — *przed* sideloadem. Zrobiony *po* sideloadzie kasuje stan
   snapshotów (`/metadata/ota`) i nowy slot nigdy nie wstanie (ląduje w recovery).
3. **Apply Update → Apply from ADB**, na komputerze `adb -d sideload lineage-…-signed.zip`. Pytanie o dodatki: **No**.
4. **Reboot system now** — bez żadnego dodatkowego resetu. Pierwszy rozruch trwa dłużej (scalanie snapshotów w tle).
5. Zainstaluj ręcznie manager KernelSU-Next (patrz sekcja Root) — nie ma go w obrazie.

> **Uwaga:** ten build jest podpisany własnymi kluczami, różnymi od kluczy oficjalnego LineageOS i od kluczy
> testowych. Instalacja na urządzeniu z innym ROM-em (w tym z buildów Tomoms) wymaga pełnego wyczyszczenia danych —
> Android nie pozwala nadpisać danych aplikacji podpisanych innym kluczem.

Aparat: w obrazie są **Aperture** (LineageOS) i **Moto Camera stockowa** (MotCamera4 + MotCamera3AI + MotoSignature,
wyciągnięte z oficjalnego firmware'u Motoroli) — wszystkie obiektywy, portret, noc, wideo HEVC działają;
jedyny brak to tryb Ultra-Res 50 MP (wymaga mechanizmu z cameraservice Motoroli, którego LineageOS nie ma).
Lepsze zdjęcia w słabym świetle daje Google Camera z ręki, ale **tylko porty z rodziny 8.4–8.6** —
Snapdragon 680 (Cortex-A73, ARMv8.0) nie wykonuje bibliotek HDR+ z GCam 8.7+/9.x (instrukcje FP16 → `SIGILL`).
Sprawdzony zestaw „jedna aplikacja do zdjęć i wideo": **MGC 8.6.263 (BSG)** + konfiguracja `MGC86-G52-final.xml`
(strojenie z Redmi Note 11 — ten sam SoC i sensor JN1 — bez kluczy strumieni/wideo Xiaomi, które psują HAL Motoroli).
Pułapka: wpis „Moto G52 → sahabulfinal.xml" na liście Hasli to konfig z Moto G72 (MediaTek) — zielone zdjęcia.

## Podziękowania

Ten build nie powstałby bez pracy zespołu [LineageOS](https://lineageos.org/), maintainera drzewa źródłowego
[Tomoms](https://github.com/tomoms), projektów [KernelSU-Next](https://github.com/KernelSU-Next/KernelSU-Next),
[Droid-ify](https://github.com/Droid-ify/client), [MindTheGapps](https://gitlab.com/MindTheGapps),
[NikGapps](https://nikgapps.com/), oraz społeczności [StevenBlack/hosts](https://github.com/StevenBlack/hosts) i
[AdGuard](https://adguard-dns.io/).

---

*Przepis buildu (dla chcących zbudować własną kopię lub prześledzić dokładnie, co i jak zostało zmienione):
[rhode-los23.2](https://github.com/MikolajQ/rhode-los23.2).*
