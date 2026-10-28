# Omoda / Jaecoo → Home Assistant

🌐 [English](README.md) · **Polski** · [Changelog](CHANGELOG.md)

## Informacje o forku i lokalizacji

To repozytorium jest forkiem tłumaczącym i lokalizującym oryginalną, włoską integrację autorstwa [@Caslinovich](https://github.com/Caslinovich/omoda9-ha). Interfejs integracji jest dostępny po angielsku i po polsku — Home Assistant wybiera język automatycznie na podstawie ustawień profilu użytkownika.

### Ważne wskazówki
* **Konto delegowane (zalecane):** aby jednocześnie zachować dostęp do oficjalnej aplikacji mobilnej, zdecydowanie zaleca się założenie drugiego konta (wymaga zapasowego numeru telefonu), przekazanie mu dostępu do pojazdu i używanie tego drugiego konta w tej integracji. Logowanie tym samym kontem w Home Assistant i w oficjalnej aplikacji powoduje, że sesje/tokeny raz po raz wylogowują Cię z oficjalnej aplikacji.

### Zgodność z modelami
* **Omoda E5 oraz Omoda / Jaecoo**: przetestowane i potwierdzone z **Omoda E5** (w Wielkiej Brytanii) oraz z **Omoda / Jaecoo** (według autora oryginału).
* **Wszystkie modele Omoda/Jaecoo**: integracja powinna działać z każdym modelem obsługiwanym przez oficjalną aplikację **Omoda / Jaecoo**. Warstwa telemetrii nie zależy od rodzaju napędu (korzysta zamiennie z nazw pól BEV/PHEV i ukrywa atrybuty paliwa w czystym elektryku), a katalog poleceń jest wspólny, więc nieobsługiwane polecenie po prostu zwraca błąd i jest pomijane. Mimo to możesz chcieć wyłączyć kilka encji, których Twój samochód nie ma.
* **Liczy się to, jakiej aplikacji używasz.** W tym ekosystemie istnieją trzy różne backendy, a ta integracja obsługuje **tylko pierwszy**:
  * **Omoda / Jaecoo (bramka „legend”)** — tej używa integracja. Działa w wielu regionach (patrz niżej).
  * **Aplikacja z marką Chery** — *inna* platforma (inne BFF, osobna usługa logowania UAA i inny sposób podpisywania żądań). Warstwa telemetrii/MQTT jest wspólna, ale logowanie już nie, więc pojazdy z aplikacji Chery **nie** są obsługiwane w obecnej postaci.
  * **CarLinko** (popularna w Malezji, Indonezji, Uzbekistanie, ZEA) — zupełnie inny backend (Hangzhou Huijie, płaskie ścieżki REST, WebSockety zamiast MQTT). **Nie** jest obsługiwana; wymagałaby osobnej integracji.

### Regiony (wiele rynków)
Domyślnym regionem jest **Europa**, ale parametry regionu (hosty BFF / TSP / MQTT oraz kod tenanta) są dostępne w opcjach, a certyfikaty mutual-TLS do telemetrii dla **ok. 40 regionów** są już dołączone (wyodrębnione z zasobów samej aplikacji — Australia, Nowa Zelandia, Tajlandia, Brazylia, Meksyk, Arabia Saudyjska, ZEA, Singapur i wiele innych). Dodanie regionu Omoda/Jaecoo sprowadza się więc głównie do **hosta BFF** (tenant często jest wspólny).

* **Potwierdzone działanie:** **Europa** (`legend-oj.omodaauto.nl`) i **Meksyk** (`legend-oj.chirey.mx`). Meksyk używa identycznej bramki i akceptuje **ten sam tenant `300006`** — różni się tylko host.
* **Aby wypróbować inny region:** ustaw host BFF w **Ustawienia → Urządzenia oraz usługi → Omoda / Jaecoo → Konfiguruj** (np. `https://legend-oj.chirey.mx/api` dla Meksyku). Jeśli wspólny tenant `300006` zostanie odrzucony, dany region używa innego kodu tenanta.
* **Jak znaleźć wartości dla regionu:** każdy rynek ma własną wersję aplikacji; dokładne `BFF_SERVICE` i `TENANT_CODE` znajdują się w jednym pliku w APK danego rynku (`assets/flutter_assets/env/.env.prod`). Strona logowania jest też dostępna w sieci pod `https://<domena-bff>/h5`, więc można ją otworzyć w przeglądarce (z DevTools) i podejrzeć wywołania logowania.
* Zrzuty komunikacji spoza UE są bardzo mile widziane — zobacz narzędzie [sandbox](#sandbox-api-dla-deweloperów), które używa tego samego logowania poza Home Assistant i wypisuje surowy JSON.

---

Połącz swój samochód **Omoda / Jaecoo** z **Home Assistant**: stan pojazdu,
lokalizacja i polecenia — tak jak w oficjalnej aplikacji, ale zintegrowane z HA.

> ✅ **Gotowe do użycia.** Na start potrzebujesz tylko **e-maila i PIN-u** konta
> Omoda/Jaecoo (oraz **jednorazowego kodu OTP** otrzymanego e-mailem przy pierwszym
> logowaniu). VIN i certyfikaty są wykrywane i instalowane **automatycznie**.
> Pakiet **nie zawiera danych osobowych**: tokeny i dane logowania zostają wyłącznie
> w *Twoim* Home Assistant.

> ⚠️ **NIEOFICJALNE oprogramowanie**, powstałe metodą inżynierii wstecznej. Niezwiązane
> z Omoda / Jaecoo / Chery. Dostarczane „tak jak jest”, używasz na własne ryzyko
> i wyłącznie z własnym pojazdem. Zobacz [`LICENSE`](LICENSE).



## Co możesz zrobić

- **Stan samochodu** — drzwi, maska, klapa bagażnika, ciśnienie/temperatura/ostrzeżenia
  opon, silnik, klimatyzacja oraz podgrzewanie/wentylacja foteli — wszystko jako
  natywne encje HA.
- **Sterowanie w sposób typowy dla HA:**
  - **Zamek** (encja lock), **Bagażnik** i **Szyberdach** (cover otwórz/zamknij),
    **Szyby** (select z 3 stanami: Closed / Ventilate / Open, czyli zamknięte / uchylone / otwarte).
  - **Przełączniki** dla wszystkiego, co się włącza i wyłącza — ładowanie, ładowanie
    zaplanowane, odmrażanie szyb, podgrzewanie kierownicy, podgrzewanie/wentylacja
    każdego fotela oraz makra „schłodź/ogrzej wszystko”.
  - Encja **Klimatyzacja** z ustawianą temperaturą docelową (zakres odczytywany z Twojego auta).
- **Bateria, zasięg, ładowanie** — % baterii, szacowany zasięg (elektryczny + WLTP),
  czujnik **Ładowanie**, moc ładowania, stan ładowania, przebieg, prędkość. Zasięg
  i bateria aktualizują się **podczas jazdy lub ładowania**; przy starcie wykonywane
  jest odświeżenie tylko do odczytu, więc wartości są dostępne także na postoju.
- **Lokalizacja / GPS** — `device_tracker` z pełną dokładnością oraz przycisk lokalizacji.
- **Dopasowanie do Twojego auta** — rodzaj napędu, czysty elektryk (BEV) lub hybryda
  plug-in (PHEV), jest wykrywany z Twojego konta, więc widoczne są tylko pasujące
  czujniki (żadnych pustych wskaźników paliwa w BEV), a klimatyzacja używa
  rzeczywistego zakresu temperatur Twojego auta.
- **Karta dashboardu** — dołączona, dopracowana karta (zobacz [Karta dashboardu](#karta-dashboardu)).
- **Ponowne uwierzytelnienie** — gdy sesja wygaśnie (np. po otwarciu oficjalnej
  aplikacji), HA wyświetla standardowy komunikat „wymaga ponownego uwierzytelnienia”,
  aby zalogować się ponownie nowym kodem.
- **Powiadomienia** — opcjonalny blueprint z alertem, gdy polecenie się nie powiedzie.

> **Języki: angielski i polski.** Nazwy encji, kreator konfiguracji i karta dashboardu
> są przetłumaczone na polski; niektóre komunikaty i stany generowane w kodzie mogą
> pozostać po angielsku. Limit ładowania / docelowy SoC celowo **nie** jest dostępny:
> backend OMODA/Jaecoo nie ma do tego API (ustawia się go na ekranie samego samochodu).
> Szczegóły w [CHANGELOG](CHANGELOG.md).

## Instalacja

1. **HACS → menu ⋮ → Niestandardowe repozytoria** → dodaj adres URL tego repozytorium,
   kategoria **Integracja**.
2. Wyszukaj **Omoda / Jaecoo** → **Pobierz** → **uruchom ponownie Home Assistant**.
3. **Ustawienia → Urządzenia oraz usługi → Dodaj integrację → Omoda / Jaecoo**.

## Pierwsze logowanie

Wszystko odbywa się **wewnątrz Home Assistant**, bez zewnętrznych narzędzi:

1. Wpisz **e-mail** i **PIN** konta (endpointy regionalne są opcjonalne,
   domyślnie Europa). HA wyśle **kod OTP** na Twój e-mail.
2. Wpisz **kod OTP** → HA utworzy sesję i wykryje Twoje pojazdy.
3. Jeśli masz kilka samochodów, wybierz **VIN**; jeśli jest tylko jeden, zostanie
   dodany od razu, ze wszystkimi encjami.

Jeśli sesja później wygaśnie (zwykle dlatego, że otwarto oficjalną aplikację),
użyj przycisków **„Diagnostyka: wyślij kod OTP” / „Diagnostyka: potwierdź OTP”**
(razem z encją tekstową „Diagnostyka: kod OTP”), aby zalogować się ponownie bez
rekonfiguracji. W angielskim interfejsie są to „Request OTP code” / „Confirm OTP”.

## Codzienne użytkowanie

- **Nie otwieraj oficjalnej aplikacji**, gdy integracja jest aktywna: to samo konto →
  wzajemnie się rozłączają (i może być potrzebny nowy kod OTP).
- Wiele encji ma stan `unknown`, gdy samochód jest w **trybie czuwania** (to normalne);
  po restarcie HA pokazują ostatnią znaną wartość.
- Bateria, prędkość i przebieg aktualizują się tylko **podczas jazdy lub ładowania**.
  Aby od razu uzyskać odczyt na postoju, użyj przycisku **„Diagnostyka: odśwież pełny
  status”**, który na ok. 1 minutę włącza klimatyzację, aby wybudzić samochód,
  a następnie ją wyłącza.

## Karta dashboardu

Integracja zawiera własną kartę i **ładuje ją automatycznie** — po restarcie pojawia się
w **Dodaj kartę → Niestandardowe → „Omoda/Jaecoo Card”** (nie trzeba ręcznie dodawać
zasobu). To dopracowane podsumowanie: nazwa pojazdu, % baterii, stan ładowania, szacowany
zasięg oraz ostrzeżenia (opony, niska bateria, offline) wyświetlane **tylko wtedy, gdy
coś jest nie tak**. Karta wyświetla się po polsku lub po angielsku, zgodnie z językiem HA.

Minimalna konfiguracja po prostu działa:

```yaml
type: custom:omoda-card
```

Opcje (wszystkie opcjonalne):

| Opcja | Co robi |
|---|---|
| `title` | Tytuł nagłówka (domyślnie: nazwa pojazdu) |
| `image` | URL zdjęcia w nagłówku (nadpisuje zdjęcie ustawione w opcjach integracji) |
| `show_all: true` | Wyświetla też wszystkie pozostałe encje, pogrupowane |
| `entities: [...]` | Dodaje Twoje własne wiersze (identyfikatory encji) |

**Zdjęcie samochodu:** ustaw **URL zdjęcia pojazdu** w **Ustawienia → Urządzenia oraz
usługi → Omoda / Jaecoo → Konfiguruj**, a karta automatycznie użyje go w nagłówku —
bez edycji karty.

**Karta pokazuje „Błąd konfiguracji” w aplikacji mobilnej (a na komputerze działa)?**
Integracja ładuje kartę automatycznie, ale aplikacja mobilna nie zawsze to wychwytuje.
**Dodaj zasób ręcznie jeden raz** i będzie działać: **Ustawienia → Dashboardy → ⋮ (prawy
górny róg) → Zasoby → + Dodaj zasób**, URL `/omoda_jaecoo_card/omoda-card.js`, typ
**Moduł JavaScript**. Następnie całkowicie zamknij i ponownie otwórz aplikację (jeśli
nadal widać stary stan, najpierw **Konfiguracja aplikacji → Debugowanie → Zresetuj pamięć
podręczną frontendu**). Zasób wystarczy dodać raz; działa na wszystkich dashboardach.

## Aktualizacja

Gdy pojawi się nowa wersja: **HACS → Omoda / Jaecoo → Aktualizuj → uruchom ponownie
Home Assistant** (**pełny** restart, nie przeładowanie). Wycofane lub przemianowane
encje są automatycznie usuwane przy ładowaniu. Historia zmian jest w [CHANGELOG](CHANGELOG.md).

## Powiadomienia o nieudanym poleceniu (opcjonalnie)

Integracja dostarcza tylko encje: **sama nie wysyła powiadomień**. Jeśli chcesz
**komunikat, gdy polecenie wysłane do samochodu się nie powiedzie** (pojazd zajęty,
nieosiągalny, wygasła sesja…), zaimportuj dołączony blueprint:

[![Zaimportuj blueprint do Home Assistant](https://my.home-assistant.io/badges/blueprint_import.svg)](https://my.home-assistant.io/redirect/blueprint_import/?blueprint_url=https%3A%2F%2Fgithub.com%2FJackRonan%2Fomoda-jaecoo-ha%2Fblob%2Fmaster%2Fblueprints%2Fautomation%2Fomoda_jaecoo%2Ffailed_command.yaml)

Następnie **Ustawienia → Automatyzacje i sceny → Utwórz automatyzację → Z blueprintu →
_Omoda / Jaecoo — Failed command alert_**. Rozpoznaje wyłącznie prawdziwe błędy
(ignoruje ✅ i ⏳), więc nie ma fałszywych alarmów.

## Gdy coś nie działa

1. **Diagnostyka (zalecane):** **Ustawienia → Urządzenia oraz usługi → Omoda /
   Jaecoo → ⋮ → Pobierz diagnostykę**. Plik jest **już zanonimizowany** (e-mail, PIN,
   VIN, tUserId i GPS ukryte; tokeny/certyfikaty pokazują tylko „present:
   yes/no”) → można go bezpiecznie dołączyć do
   [zgłoszenia](https://github.com/JackRonan/omoda-jaecoo-ha/issues).
2. **Szczegółowe logi:** ta sama strona → **⋮ → Włącz rejestrowanie debugowania** →
   odtwórz problem → **Wyłącz rejestrowanie debugowania**: HA pobierze log. PIN, OTP
   i tokeny **nigdy nie trafiają do logów**; jedyną potencjalnie wrażliwą wartością jest
   **VIN** (diagnostyka z kroku 1 już go ukrywa).

## Wymagania

- Home Assistant 2024.1.0+ z HACS.
- Konto Omoda/Jaecoo z przypisanym pojazdem (właściciel).
- Lokalny broker MQTT **nie** jest potrzebny: integracja **sama** łączy się
  z chmurą samochodu.

---

# Jak to działa (szczegóły techniczne)

Wszystko poniżej dzieje się **automatycznie**: opis służy tylko do zrozumienia
przepływu, debugowania lub przeniesienia integracji do regionu, który nie jest jeszcze
obsługiwany. W zwykłej instalacji **niczego nie trzeba uruchamiać ręcznie**.

### 1. Logowanie i token (OTP)

Pierwsze logowanie tworzy **token sesji** dla konta na podstawie e-maila, PIN-u i OTP.
Łańcuch jest sterowany przez kreator konfiguracji (kod w
`custom_components/omoda_jaecoo/core/`):

| Krok | Moduł | Co robi |
|---|---|---|
| wysłanie OTP | `login_omoda.py invia <email>` | rozwiązuje captchę bramki (§2) i wysyła kod **e-mailem** |
| utworzenie tokenu | `prova_token.py <email> <code>` | wywołuje `/auth/oauth2/token`, odtwarzając działanie aplikacji (szyfrowanie SM4), i zapisuje token |
| orkiestracja | `session.py` | udostępnia `request_otp()` / `confirm_otp(code)` / `check()` / `refresh()` |

Token trafia do **`<config>/omoda_jaecoo_<VIN>_token.json`** (nigdy do repozytorium).
Dopóki **refresh_token** jest ważny, `session.refresh()` odnawia sesję **bez** nowego
OTP. Nowy OTP jest potrzebny tylko wtedy, gdy wygasną zarówno token, jak i refresh —
typowy przypadek: **otwarcie oficjalnej aplikacji** (po stronie chmury działa tylko
jedna sesja).

### 2. Captcha (suwak) — rozwiązywana w Home Assistant

Wysłanie OTP jest chronione **captchą z suwakiem**. `captcha_solver.py` rozwiązuje ją
**w procesie HA**, używając **wyłącznie `numpy` i `Pillow`** (korelacja wzajemna
i morfologia zaimplementowane od zera, **bez OpenCV**): dzięki temu działa nawet na
**Home Assistant OS** (musllinux, gdzie `opencv-python-headless` nie ma paczki wheel).
Bez udziału użytkownika i bez ciężkich zależności.

### 3. Certyfikaty mutual-TLS dla MQTT — automatyczna instalacja

Telemetria łączy się z brokerem **EMQX** samochodu przez **mutual-TLS**. Certyfikaty
klienta (`ca.pem`, `client.pem`, `client.key`) są **uniwersalnymi stałymi dla
regionu** — **identycznymi dla wszystkich użytkowników**, pochodzącymi z **publicznych**
zasobów APK — a **nie** danymi konta: oddzielenie kont zapewniają login/hasło MQTT
i uprawnienia do tematów (ACL), dokładnie tak jak w oficjalnej aplikacji.

Przy pierwszym uruchomieniu `coordinator.async_provision_certs()` odszyfrowuje
certyfikaty z paczki (`custom_components/omoda_jaecoo/certs/store.json`) i zapisuje je
w **`<config>/omoda_jaecoo_<VIN>_certs/`**. Ręczne nadpisanie: pole **`certs_src`**
w kreatorze konfiguracji. Dla regionu, którego **nie ma** w paczce, start kończy się
komunikatem wskazującym, gdzie umieścić certyfikaty.

### 4. Przygotowanie do wysyłania poleceń (car_token)

Wysyłanie poleceń wymaga **car_token** dla danego pojazdu (nie `userToken` z BFF).
Łańcuch odtworzony z aplikacji, obsługiwany przez `commands.py`:

```
getTuserId → loginTSP (= car_token) → queryList → setVecDefault(vin)
           → checkPassword(PIN, scene) → command   (Authorization = car_token)
```

**PIN** należy do konta. ⚠️ **Błędny** PIN grozi **zablokowaniem** konta: nie wolno
go zgadywać. VIN musi być na liście autoryzowanych pojazdów (`authorizeType`
2 = właściciel, 0 = delegat). `provision.py` udostępnia **diagnostykę tylko do
odczytu** (`diagnose()`), która sprawdza przypisanie pojazdu i `authorizeType`
**bez wysyłania czegokolwiek do samochodu**.

### Generowane pliki (w Twoim HA, nigdy w repozytorium)

- `<config>/omoda_jaecoo_<VIN>_token.json` — token sesji konta.
- `<config>/omoda_jaecoo_<VIN>_certs/` — certyfikaty mutual-TLS dla brokera MQTT.

Są objęte `.gitignore` i nigdy nie opuszczają Twojej instalacji.

### Ręczne przygotowanie / logowanie (zaawansowane, poza HA)

Do debugowania możesz użyć skryptów CLI z `custom_components/omoda_jaecoo/core/`
z Pythonem, który ma zainstalowane `requirements` z manifestu, konfigurując
środowisko przez zmienne (zobacz [`omoda_jaecoo.env.example`](omoda_jaecoo.env.example)):

```bash
# 1) wyślij kod OTP e-mailem (rozwiązuje captchę)
python3 login_omoda.py invia <email>

# 2) utwórz token i zapisz go w $OMODA_TOKEN_PATH (domyślnie ./token.json)
python3 prova_token.py <email> <code>

# 3) (opcjonalnie) diagnostyka pojazdu/autoryzacji — TYLKO DO ODCZYTU
python3 provision.py
```

Token utworzony w ten sposób to **ten sam** plik, który czyta integracja: ustawiając
`OMODA_TOKEN_PATH` na `<config>/omoda_jaecoo_<VIN>_token.json`, możesz odblokować
konfigurację nawet bez ponownego przechodzenia przez OTP w kreatorze.

### Sandbox API (dla deweloperów)

Folder [`sandbox/`](sandbox/) zawiera samodzielne, interaktywne CLI, które używa
prawdziwego mechanizmu logowania i podpisywania integracji, aby odpytywać API **poza
Home Assistant** i wypisywać **surowy JSON** — przydatne do sprawdzania, co raportuje
samochód, lub do testowania nowych endpointów. Wykrywa VIN tak samo jak kreator
konfiguracji i trzyma swój token oraz konfigurację **poza** repozytorium. Zobacz
[`sandbox/README.md`](sandbox/README.md) (po angielsku).

## Licencja

[MIT](LICENSE). Niezależny, nieoficjalny projekt.
