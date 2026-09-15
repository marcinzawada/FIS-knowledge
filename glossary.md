# Słownik FIS / SICS / ReInsurance

**Status:** `Financial record types` potwierdzone w IDA Wiki; zachowanie dat potwierdzone w aktualnym kodzie FIS

**Ostatnia weryfikacja:** 2026-09-15

`FinanceService` pośredniczy między SICS, w którym rejestrowane są contracts, premiums i claims, a Oracle, do którego trafiają dane księgowe. Definicje `Financial record types` poniżej pochodzą z wiki i są źródłem nadrzędnym. Opisy dat odnoszą się do aktualnej implementacji FIS, ponieważ strona wiki nie definiuje ich reguł.

## Financial record types

### Booking

`Booking` oznacza zarejestrowanie financial event, na przykład należnego premium albo zobowiązania do wypłaty claim. Sam `Booking` nie oznacza jeszcze, że doszło do przepływu gotówki.

### Pairing

`Pairing` jest dopasowaniem kwot w tabeli SICS `GL_P`. Jeden event `Pairing` może utworzyć `TechnicalSettlement` lub `ClaimSettlement`, `OffsetAdjustment` albo `NettingSettlement` — zależnie od typów sparowanych balances oraz strony match.

### Transaction

`Transaction` to podstawowy building block: pojedynczy financial event zarejestrowany w SICS, na przykład earned premium albo expected claim payment. Rekord zawiera context contract i line of business oraz informację, czy kwota jest actual czy estimate. Na tym etapie pieniądze nie muszą jeszcze zmienić właściciela.

W FIS rekord z SICS jest reprezentowany przez `GLTransaction` i zapisywany jako `Transaction` w tabeli `Transactions`.

### Settlement

`Settlement` jest wspólnym określeniem business side payment `Pairing`. Gdy `REMITTANCE` zostaje dopasowane do otwartych business balances, FIS tworzy `TechnicalSettlement` albo `ClaimSettlement`; obu towarzyszy money side w postaci `OffsetAdjustment`.

### Technical Settlement (`TechnicalSettlement`)

`TechnicalSettlement` opisuje business side `Pairing`, w którym rzeczywista payment (`REMITTANCE`) została dopasowana do otwartych premium-side amounts. Pokazuje więc, które premium bookings zostały opłacone; jeden `Pairing` może wygenerować kilka takich rekordów — po jednym dla każdego rozliczonego `Booking`. Rekord ma pełną atrybucję contract i line of business.

W aktualnym FIS ten przypadek rozpoznaje `GLSettlement` z `Balance1Type = REMITTANCE` i `Balance2Type = TECHNICAL`.

### Claim Settlement (`ClaimSettlement`)

`ClaimSettlement` ma tę samą rolę, lecz payment jest dopasowana do otwartych claim amounts. Opisuje więc, które zobowiązania claim zostały wypłacone. FIS wzbogaca te rekordy o dodatkowe dane claim pobierane osobno z SICS.

W aktualnym FIS ten przypadek rozpoznaje `GLSettlement` z `Balance1Type = REMITTANCE` i `Balance2Type = CLAIM`.

### Offset Adjustment (`OffsetAdjustment`)

`OffsetAdjustment` opisuje money side tego samego payment `Pairing`. Podczas gdy settlements wskazują premium albo claim bookings, które zostały rozliczone, `OffsetAdjustment` zapisuje payment amount oraz balance entries dla FX differences i write-offs. Razem z settlement records zapewnia zbilansowanie obu stron `Pairing`.

Rekord jest tworzony na poziomie `Pairing`, bez atrybucji do contract ani line of business, i jest oznaczony `ActivityCode`: `ORA-REMIT`, `ORA-WRTO`, `ORA-FXADJ`, `ORA-REMITWO` albo `ORA-WRTO-WO`.

### Netting Settlement (`NettingSettlement`)

`NettingSettlement` opisuje `Pairing` bez payment: dwa otwarte business amounts — `TECHNICAL` i/lub `CLAIM` — są anulowane bezpośrednio względem siebie. Nie powstaje money side, więc taki `Pairing` nie generuje `OffsetAdjustment`.

Przykład: partner jest winien company premium, a company jest winna partnerowi claim payment; zamiast wykonywać dwa payments, kwoty są `netted`.

## Daty

Poniższe definicje opisują aktualne reguły FIS. Wymagają ponownej weryfikacji po zmianie `MappingProvider`.

### Inception Date (`InceptionDate`)

W `Business Hub` wartość `InceptionDate` dla `Business` albo `Program` jest wyznaczana jako najwcześniejszy `StartDate` spośród aktywnych `InsuredPeriods`. Jest to data początku najwcześniejszego aktywnego okresu, a nie data importu ani `BookingDate`.

Źródło implementacji: `IDA/hubs/src/IDA.BusinessHub.DataFunction/Services/BusinessHubDataService.cs`.

### Insured Period Start (`InsuredPeriodStart`)

`InsuredPeriodStart` pochodzi z danych SICS dla `Transaction` i `Settlement`. FIS używa go między innymi do zbudowania `CompoPolicyReference`. Dla `UPR/DAC` oraz `accruals` z kodem `L4` rozpoczynającym się od `INA` albo `OUTA` stanowi on `ExchangeRateDate` zamiast `BookingDate`.

Źródło implementacji: `IDA/integrations/src/IDA.FinanceService/Mappings/MappingProvider.cs`.

### Booking Date (`BookingDate`, `booked date`)

`BookingDate` jest datą financial event w `General Ledger` SICS. FIS przenosi ją do swoich encji i eksportuje do Oracle jako `TRANSACTION_DATE`. Dla booked premiums i deductions źródłem tej daty jest timestamp zamknięcia `SICS worksheet`.

Dla `OffsetAdjustment` FIS ustawia `BookingDate` z `PairingDatetime`.

Źródła implementacji: `IDA/hubs/src/IDA.Utils/Sics/GLTransaction.cs`; `IDA/hubs/src/IDA.Utils/Sics/GLSettlement.cs`; `IDA/integrations/src/IDA.FinanceService/Mappings/MappingProvider.cs`; `IDA/integrations/src/IDA.FinanceService/Services/OracleExportService.cs`.

### Accounting Date (`AccountingDate`, `accounted date`)

`AccountingDate` jest datą wyliczaną przez FIS na podstawie `BookingDate` i daty przetworzenia. Jeżeli `BookingDate` należy do bieżącego albo bezpośrednio poprzedniego miesiąca, FIS zachowuje `BookingDate`. W pozostałych przypadkach — rekord ma co najmniej dwa miesiące, przekracza rok albo ma future date — FIS ustawia pierwszy dzień bieżącego miesiąca.

FIS eksportuje tę wartość do pola Oracle `ACCOUNTING_DATE`.

Źródło implementacji: `IDA/integrations/src/IDA.FinanceService/Mappings/MappingProvider.cs` — `DetermineAccountingDate`; `IDA/integrations/src/IDA.FinanceService/Services/OracleExportService.cs`.

### Ledger Transfer Date (`LedgerTransferDate`)

`LedgerTransferDate` jest watermarkiem używanym przez FIS do incremental fetch danych SICS dla `Transactions`, `Settlements`, `NettingSettlements` i `OffsetAdjustments`. Pobieranie rozpoczyna się od `Max(LedgerTransferDate)`, a records z granicznego transfer batch są ponownie odczytywane, aby batch z tym samym timestampem nie został częściowo zaimportowany.

Źródło implementacji: `IDA/integrations/src/IDA.FinanceService/Processors/FinanceSicsExportCreatedEventProcessor.cs`; `IDA/integrations/src/IDA.FinanceService/Services/SicsDataFetcher.cs`.

### Pairing Date (`PairingDate` / `PairingDatetime`)

`PairingDate` pochodzi z SICS dla `Settlement` i `NettingSettlement`; `PairingDatetime` pełni tę rolę dla `OffsetAdjustment`. W przypadku `OffsetAdjustment` jest także źródłem `BookingDate` i — z wyjątkiem aktywności `REMIT` — `ExchangeRateDate`.

`PairingDate` nie zastępuje daty kursu pierwotnego `Transaction` przy rozliczaniu `Settlement`, ponieważ prowadziłoby to do wyceny pierwotnej pozycji po kursie miesiąca rozliczenia.

Źródło implementacji: `IDA/hubs/src/IDA.Utils/Sics/GLSettlement.cs`; `IDA/hubs/src/IDA.Utils/Sics/GLOffsetAdjustment.cs`; `IDA/integrations/src/IDA.FinanceService/Mappings/MappingProvider.cs`.

### Settlement Value Date (`SettlementValueDate`)

`SettlementValueDate` to data, w której gotówka została otrzymana albo wypłacona dla balance `REMITTANCE`. Jest pusta dla `NettingSettlement`. Dla zwykłego `Settlement` FIS używa jej jako `ExchangeRateDate`; jeśli jej nie ma, stosuje `BookingDate`.

Źródło implementacji: `IDA/hubs/src/IDA.Utils/Sics/GLSettlement.cs`; `IDA/integrations/src/IDA.FinanceService/Mappings/MappingProvider.cs` — `DetermineSettlementExchangeRateDate`.

### Original Booking Date (`OriginalBookingDate`)

`OriginalBookingDate` to `BookingDate` pierwotnego `Transaction` rozliczanego przez `Settlement`: timestamp zamknięcia `worksheet` stojącego za sparowanym `ledger detail`. Wartość może być pusta tylko wtedy, gdy ten `worksheet` jest nadal otwarty.

FIS używa tej daty do odtworzenia `ExchangeRateDate` pierwotnego `Transaction`, gdy nie ma go już w lokalnej tabeli `Transactions`.

Źródło implementacji: `IDA/hubs/src/IDA.Utils/Sics/GLSettlement.cs`; `IDA/integrations/src/IDA.FinanceService/Mappings/MappingProvider.cs` — `DetermineOriginalExchangeRateDate`.

### Exchange Rate Date (`ExchangeRateDate`)

`ExchangeRateDate` jest datą wyboru kursu. Dla `Transaction` FIS używa `InsuredPeriodStart` dla `UPR/DAC` oraz `accruals` (`L4` zaczynające się od `INA` lub `OUTA`); dla pozostałych przypadków używa `BookingDate`.

Przy `Settlement` normalna noga zachowuje datę kursu pierwotnego `Transaction`, natomiast event `realised-FX` używa `SettlementValueDate` albo `BookingDate`. Różnica między tymi nogami reprezentuje `FX gain/loss`.

Źródło implementacji: `IDA/integrations/src/IDA.FinanceService/Mappings/MappingProvider.cs`.

## Źródła nadrzędne

- IDA Wiki, `Welcome to the IDA Wiki / Knowledge Base / FAIR / Data / Financial record types` — definicje `Booking`, `Pairing`, `Transaction`, `TechnicalSettlement`, `ClaimSettlement`, `OffsetAdjustment` i `NettingSettlement`.
