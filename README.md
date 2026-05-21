# Waste Next 🗑️

Pokazuje **następny wywóz śmieci** w jednej encji Home Assistant — z ikoną
i kolorem zależnym od frakcji odpadów.

Dane pobiera z kalendarza tworzonego przez integrację
[Waste Collection Schedule](https://github.com/mampfes/hacs_waste_collection_schedule).

---

## Co dostajesz

Jedną encję `sensor.nastepny_wywoz_smieci`:

| Pole | Przykład | Opis |
|---|---|---|
| stan | `BIO` | nazwa najbliższej frakcji |
| `dni` | `2` | ile dni do wywozu |
| `data` | `2026-05-22` | data wywozu |
| `opis` | `Za 2 dni` | gotowy tekst do wyświetlenia |
| `icon_color` | `light-green` | nazwa koloru Home Assistant |
| ikona | `mdi:flower` | ikona zależna od frakcji |

Po godzinie **16:00** dzisiejszy wywóz jest pomijany — sensor pokazuje
od razu kolejny termin.

---

## Wymagania

- Home Assistant OS
- Integracja Waste Collection Schedule z **włączonym kalendarzem** (`calendar.*`)

---

## Instalacja ręczna

Ten projekt to **pakiet** (*package*) Home Assistant — osobny plik
konfiguracyjny. Nie wklejasz nic do `configuration.yaml` poza jedną
linią (jednorazowo).

**1. Włącz pakiety (raz na zawsze).** W `configuration.yaml` dodaj:

```yaml
homeassistant:
  packages: !include_dir_named packages
```

Jeśli masz już sekcję `homeassistant:` — dopisz tylko linię `packages:`.

**2. Utwórz folder `packages`** obok pliku `configuration.yaml`.

**3. Wrzuć plik `packages/waste_next.yaml`** z tego repozytorium
do folderu `packages` na swoim HA.

**4. Dostosuj plik** — patrz sekcja *Konfiguracja* niżej.

**5.** Developer Tools → YAML → **Sprawdź konfigurację**, następnie
**Restart** Home Assistant (pierwsze dodanie pakietu wymaga restartu).

---

## Automatyczna synchronizacja (dodatek Git pull)

Jeśli chcesz, żeby zmiany z GitHub trafiały na HA automatycznie
(bez ręcznego kopiowania pliku), użyj oficjalnego dodatku **Git pull**.

### Jednorazowa konfiguracja

**1.** Zainstaluj dodatek:
Ustawienia → Dodatki → Sklep → wyszukaj **Git pull** → Zainstaluj.

**2.** Skonfiguruj dodatek (zakładka *Konfiguracja*):

```yaml
repository: https://github.com/wignaczewski/ha-waste-next
auto_restart: false
repeat:
  active: true
  interval: 300
```

`interval: 300` oznacza sprawdzanie co 5 minut. Możesz zmienić np. na `3600`
(co godzinę).

**3.** Uruchom dodatek → zakładka *Dziennik* — powinien pojawić się komunikat
o pomyślnym pobraniu repozytorium.

Po tym kroku plik `packages/waste_next.yaml` trafi automatycznie
do `/config/packages/` na Twoim HA.

**4.** Jeśli jeszcze nie masz włączonych pakietów, dodaj do `configuration.yaml`:

```yaml
homeassistant:
  packages: !include_dir_named packages
```

**5.** Zrestartuj HA (pierwsze uruchomienie pakietu wymaga restartu).

### Testowanie zmian po pushu

Po każdym pushu na GitHub wystarczy:

1. Odczekać do 5 minut (lub wejść w dodatek Git pull → **Uruchom ponownie**,
   żeby wymusić natychmiastowe pobranie).
2. Developer Tools → YAML → **Przeładuj szablony** (nie trzeba restartować HA).
3. Sprawdzić encję `sensor.nastepny_wywoz_smieci` w Developer Tools → States.

---

## Konfiguracja

Otwórz `packages/waste_next.yaml` i zmień dwie rzeczy:

1. **Nazwa kalendarza.** Zamień `calendar.wywoz_smieci` na nazwę swojego
   kalendarza (Developer Tools → States). Występuje w **4 miejscach**.

2. **Nazwy frakcji.** Słowniki ikon i kolorów mają klucze `BIO`, `Papier`,
   `Tworzywa`, `Zmieszane`, `Pozostałe`. Muszą się zgadzać **co do znaku**
   z nazwami wydarzeń w Twoim kalendarzu. Sprawdzisz je tak:
   Developer Tools → Actions → `calendar.get_events` (target: Twój
   kalendarz, `duration: hours: 168`) → *Wykonaj* → spójrz na pole `summary`.

---

## Wyświetlanie na dashboardzie

⚠️ Sam atrybut `icon_color` **nie pokoloruje** ikony — standardowe karty
Home Assistant nie czytają tego atrybutu. Potrzebujesz karty, która umie
czytać atrybuty, np. [Mushroom](https://github.com/piitaya/lovelace-mushroom):

```yaml
type: custom:mushroom-template-card
entity: sensor.nastepny_wywoz_smieci
primary: "{{ states('sensor.nastepny_wywoz_smieci') }}"
secondary: "{{ state_attr('sensor.nastepny_wywoz_smieci','opis') }}"
icon: "{{ state_icon('sensor.nastepny_wywoz_smieci') }}"
icon_color: "{{ state_attr('sensor.nastepny_wywoz_smieci','icon_color') }}"
```

Gotowy przykład znajdziesz w [`examples/mushroom-card.yaml`](examples/mushroom-card.yaml).

---

## Czy to działa przez HACS?

Nie. HACS instaluje integracje (Python) oraz karty (JavaScript) — nie
instaluje pakietów YAML. Ten projekt instaluje się ręcznie lub przez
dodatek Git pull (patrz wyżej).

---

## Licencja

MIT — patrz [LICENSE](LICENSE).
