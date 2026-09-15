# FIS Knowledge

To repozytorium jest `second brain` dla wiedzy domenowej i technicznej dotyczącej **FIS**, **SICS** oraz **ReInsurance**. Jego celem jest zachowanie kontekstu, który zwykle pozostaje rozproszony między kodem, dokumentacją, zadaniami, e-mailami i rozmowami — oraz powiązanie go tak, aby dało się go wykorzystać w następnym zadaniu.

## Cel

- gromadzić wiedzę odkrytą podczas pracy z repozytoriami i zadaniami;
- opisywać `business flows`, `systems`, `integrations`, `data models` i `technical decisions`;
- zachowywać uzasadnienia oraz zależności, a nie tylko końcowe rozwiązania;
- umożliwiać szybkie odnalezienie sprawdzonej informacji bez ponownego badania kodu.

Nie próbujemy od razu tworzyć idealnej dokumentacji. Najpierw zapisujemy przydatne odkrycie, następnie podłączamy je do właściwych tematów i uzupełniamy, gdy pojawi się nowy kontekst.

Pierwszym punktem wejścia do wspólnej terminologii jest [słownik FIS / SICS / ReInsurance](glossary.md).

## Język i terminologia

Treść zapisujemy po polsku. Nazwy systemów, komponentów, klas, tabel, endpointów, zdarzeń oraz terminy domenowe i techniczne pozostają po angielsku. Nie tłumaczymy nazw własnych ani terminów, które występują w kodzie lub dokumentacji źródłowej.

Przykład: „`ClaimSettlementLookup` pobiera `General Ledger pairings` dla wybranego `booking date`.”

## Źródła wiedzy

Każda istotna informacja powinna wskazywać źródło oraz poziom pewności:

| Źródło | Jak zapisywać |
| --- | --- |
| Kod | ścieżka do pliku, klasa/funkcja i, gdy ma to znaczenie, commit lub branch |
| Dokumentacja | link albo ścieżka oraz data dostępu |
| Zadanie | identyfikator Azure DevOps i krótki kontekst |
| E-mail lub rozmowa | data, autor/uczestnicy i zwięzłe podsumowanie; bez danych wrażliwych |
| Obserwacja | oznaczenie `Do weryfikacji` i opis sposobu weryfikacji |

Nie zapisujemy sekretów, danych produkcyjnych, danych osobowych ani pełnych treści poufnej korespondencji.

## Status informacji

Stosujemy następujące oznaczenia:

- `Potwierdzone` — zweryfikowane w kodzie, dokumentacji lub przez właściciela obszaru.
- `Do weryfikacji` — robocza hipoteza albo niepełna informacja.
- `Nieaktualne` — historyczna wiedza zachowana dla kontekstu, lecz nieprzeznaczona do użycia w nowych zmianach.

## Zasady second brain

- **Capture quickly** — nie czekamy na pełne zrozumienie. Krótka notatka z odnośnikiem do źródła jest lepsza niż utracone odkrycie.
- **One idea per note** — jedna notatka opisuje jeden `concept`, `flow`, `decision`, `integration` albo problem. Dzięki temu łatwo ją rozwijać i linkować.
- **Link context** — używamy względnych linków Markdown między powiązanymi notatkami, np. `[Claim flow](../processes/claim-flow.md)`. Link jest ważniejszy niż kopiowanie tej samej treści.
- **Preserve provenance** — przy faktach zostawiamy ślad do kodu, dokumentacji, zadania lub rozmowy. Wniosek bez źródła oznaczamy `Do weryfikacji`.
- **Keep it alive** — przy zmianie wiedzy aktualizujemy istniejącą notatkę albo oznaczamy ją jako `Nieaktualne`; nie tworzymy sprzecznych kopii.
- **Write for the future self** — zapis ma odpowiadać na pytanie: „co muszę wiedzieć, aby za trzy miesiące ruszyć z tym zadaniem bez ponownego śledztwa?”

## Proponowana struktura

```text
FIS-knowledge/
├── README.md
├── inbox/               # szybkie, nieuporządkowane odkrycia do późniejszego przetworzenia
├── fis/                 # moduły, flows i integrations związane z FIS
├── sics/                # procesy i komponenty SICS
├── reinsurance/         # domena i procesy ReInsurance
├── integrations/        # interfaces, messages, schedules i dependencies między systemami
├── data/                # data models, mappings, CSV files i database conventions
├── processes/           # end-to-end business flows i operational runbooks
├── decisions/           # Architecture Decision Records oraz ważne technical decisions
├── glossary.md          # wspólny słownik terminów
└── sources/             # indeks dokumentów, zadań i innych materiałów źródłowych
```

Katalogi utworzymy wtedy, gdy pojawi się w nich pierwsza wartościowa notatka — bez pustej struktury dla samej struktury. Wyjątkiem może być `inbox/`: służy do natychmiastowego przechwytywania wiedzy, zanim zdecydujemy, gdzie należy.

## Szablon notatki

```md
# [Temat]

**Status:** Potwierdzone | Do weryfikacji | Nieaktualne

**Obszar:** FIS | SICS | ReInsurance | Integration | Data
**Ostatnia weryfikacja:** RRRR-MM-DD

## Kontekst

Krótki opis problemu, procesu albo komponentu.

## Wiedza

Najważniejsze fakty, zależności i wyjątki.

## Źródła

- Kod: `ścieżka/do/pliku.cs` — `Class.Method`
- Dokumentacja: [nazwa](link-lub-ścieżka)
- Azure DevOps: `AB#12345` — krótki kontekst

## Otwarte pytania

- [ ] Pytanie lub element wymagający weryfikacji.
```

## Sposób pracy

Po każdym istotnym odkryciu z kodu, dokumentacji, zadania lub rozmowy dodajemy krótką notatkę w `inbox/` albo aktualizujemy istniejącą notatkę. Następnie zapisujemy link do źródła, rozdzielamy fakty od wniosków i dodajemy powiązania do pozostałej wiedzy. README jest tylko punktem wejścia — szczegółowa wiedza powinna trafiać do małych, tematycznych plików Markdown.
