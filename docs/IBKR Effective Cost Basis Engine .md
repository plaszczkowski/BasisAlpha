# Projekt: IBKR Effective Cost Basis Engine  
### Aplikacja .NET 8 — Portfolio Analytics / Wheel Strategy / Option Income Engine

---

## 1. Cel projektu

Zbudować aplikację w **.NET 8**, która:

- łączy się z **Interactive Brokers (IBKR API)**  
- pobiera pozycje akcyjne i transakcje opcyjne  
- wylicza **rzeczywisty (economic) cost basis** pozycji akcyjnych  
- uwzględnia **premie opcyjne, rolling, assignment, fees**  
- prezentuje **Effective Cost Basis**, **Net Premium Earned**, **Strategy-Level PnL**, **True Entry Price after Option Income**

Aplikacja wspiera strategie:

- Covered Calls  
- Cash Secured Puts  
- Wheel Strategy  
- Rolling opcji  
- Assignment-aware cost basis tracking  

**Problem:** Broker pokazuje średnią cenę akcji i PnL opcji osobno.  
**Aplikacja ma pokazać:**

- Effective Cost Basis  
- Net Premium Earned  
- Strategy-Level PnL  
- True Entry Price after Option Income  

---

## 2. Główny wzór biznesowy

### Effective Cost Basis

NetCostBasis = (StockCost - OptionProfits + Fees + AssignmentAdjustments) / Shares


Gdzie:

- **StockCost** — suma kosztu akcji  
- **OptionProfits** — wszystkie premie netto  
- **Fees** — prowizje  
- **AssignmentAdjustments** — korekty po assignment  

---

## 3. Stack technologiczny

**Backend:**

- .NET 8  
- ASP.NET Core Web API  
- Worker Service  
- Entity Framework Core  

**Integracje:**

- IBKR TWS API (C# IBApi)

**Baza danych:**

- PostgreSQL

**Logging:**

- Serilog

**Testy:**

- xUnit  
- FluentAssertions  

**Containerization:**

- Docker  

---

## 4. Architektura systemu

### Docelowy przepływ:

IBKR API → Sync Worker → Normalization Layer → Domain Engine → PostgreSQL → ASP.NET API → Dashboard / Reporting


### Architektura warstwowa (Clean Architecture):

Solution:

- **Portfolio.Api**
- **Portfolio.Application**
- **Portfolio.Domain**
- **Portfolio.Infrastructure**
- **Portfolio.Tests**

---

## 5. Model domenowy

### StockPosition

- Symbol  
- Quantity  
- AverageCost  
- TotalCost  

### OptionTrade

- Symbol  
- OptionType (Call/Put)  
- Strike  
- Expiry  
- PremiumCollected  
- Contracts  
- Fees  
- Status (Expired / Assigned / BoughtBack)

### AssignmentEvent

- AssignedShares  
- StrikePrice  
- AssignmentDate  
- AssignmentType  

### PortfolioState

- StockPosition  
- OptionTrades  
- AssignmentEvents  
- CashFlows  

---

## 6. Core kalkulatory

### EffectiveCostBasisCalculator

Liczy:

- adjusted cost basis  
- per-share effective entry  

### PremiumIncomeCalculator

- total premium earned  
- annualized option yield  

### AssignmentAdjustmentCalculator

Obsługuje:

- covered call assignment  
- put assignment  
- adjusted stock basis  

### StrategyPnlCalculator

- stock pnl  
- options pnl  
- net strategy pnl  

---

## 7. Etapy implementacji (Roadmap)

### **Faza 1 — MVP**

Taski:

1. Skonfiguruj solution .NET 8  
2. Dodaj projekty warstwowe  
3. Dodaj integrację z IBKR API  
4. Pobieraj pozycje akcyjne  
5. Pobieraj executions opcyjne  
6. Znormalizuj dane do modelu domenowego  
7. Napisz EffectiveCostBasisCalculator  
8. Endpoint: `GET /portfolio/effective-cost`  

**Output MVP:**

- effective cost basis  
- total premiums earned  

**Definition of Done:**  
Działa dla jednej pozycji (IBM)

---

### **Faza 2 — Production Core**

Taski:

9. PostgreSQL persistence  
10. Trade repository  
11. Sync worker z IBKR  
12. Assignment handling  
13. Fees handling  
14. Rolling trades support  
15. Audit trail  

Nowe endpointy:

- `/portfolio/summary`  
- `/portfolio/costbasis`  
- `/portfolio/premiums`  

---

### **Faza 3 — Strategy Analytics**

Taski:

16. Income yield dashboard  
17. Premium by month  
18. Cost basis trend over time  
19. Strategy IRR  
20. Option cycle analytics  
21. Assignment impact reports  

Metryki:

- Premium Yield %  
- Effective Entry Price  
- Return on Cost Basis  
- Income vs Capital Gain Split  

---

## 8. Zadania dla AI (Claude)

### Task 1  
Wygeneruj strukturę solution zgodnie z Clean Architecture.

### Task 2  
Napisz modele domenowe (rich domain model).

### Task 3  
Napisz EffectiveCostBasisCalculator + unit tests (90% coverage).

### Task 4  
Integracja z IBKR API (connect, reqPositions, reqExecutions).

### Task 5  
Background sync worker (polling, reconnect, error handling).

### Task 6  
Persistence: EF Core + PostgreSQL + migracje.

### Task 7  
REST API:  
- `GET /effective-cost-basis`  
- `GET /premium-income`  
- `GET /strategy-pnl`  

### Task 8  
Serilog logging.

### Task 9  
Docker + docker-compose + postgres.

### Task 10  
Testy: unit, integration, calculators, assignments.

---

## 9. Scenariusze testowe

### Scenario A  
200 akcji IBM po 226.15  
Premie opcyjne: 4900 PLN  
Policz effective cost basis.

### Scenario B  
Covered call assignment — sprawdź adjustment.

### Scenario C  
Rolling:  
- otwarcie  
- odkupienie  
- nowy strike  
Sprawdź cumulative premium.

### Scenario D  
Cash secured put assigned — sprawdź blended basis.

---

## 10. Wymagania architektoniczne

- SOLID  
- Clean Architecture  
- Dependency Injection  
- Repository Pattern  
- Domain Driven Design (lite)  
- Fail-safe reconnect do IBKR  
- Idempotent sync  
- Testability first  

Unikać:

- fat controllers  
- business logic w API layer  
- direct IBKR calls z kontrolerów  
- god classes  

---

## 11. Struktura folderów

Portfolio.Domain/
Entities/
ValueObjects/
Services/

Portfolio.Application/
Interfaces/
UseCases/
Calculators/

Portfolio.Infrastructure/
Ibkr/
Persistence/
Repositories/

Portfolio.Api/
Controllers/
Workers/
Configuration/


---

## 12. Stretch goals (opcjonalne)

- Greeks snapshot  
- Delta-adjusted exposure  
- Covered call optimizer  
- Wheel strategy recommendations  
- Tax lot support  
- Multi-position support  
- UI dashboard  

---

## 13. Przykładowe prompt’y dla Claude

- "Implement EffectiveCostBasisCalculator with assignment adjustments and unit tests."  
- "Refactor IBKR integration using resilient reconnect patterns."  
- "Design domain model for covered call assignment affecting stock cost basis."  
- "Generate production-grade .NET worker consuming IBKR executions stream."  
- "Add repository and EF mapping for option trade ledger."  

---

## 14. Kolejność realizacji

1. Domain model  
2. Calculator engine  
3. IBKR integration  
4. Persistence  
5. API  
6. Worker sync  
7. Tests  
8. Docker  
9. Analytics  
10. UI  

---

## 15. Definicja sukcesu projektu

Projekt jest ukończony, gdy aplikacja potrafi:

- pobrać pozycję IBM z IBKR  
- pobrać wszystkie premie z opcji  
- policzyć effective cost basis  
- uwzględnić assignments  
- pokazać strategy-level PnL  
- działać automatycznie po sync z IBKR  

**Finalny output:**  
"Ile naprawdę kosztuje mnie moja pozycja IBM po wszystkich premiach opcyjnych?"

---

## 16. MVP Backlog (20% pracy → 80% wartości)

- IBKR connector  
- Position importer  
- Option premium importer  
- Effective cost basis calculator  
- Simple REST endpoint  

---

## 17. Dodatkowy moduł: Cost Basis Timeline

Pokazuje trend obniżania cost basis:

- Styczeń: 226.15  
- Luty: 223.80  
- Marzec: 220.40  
- Kwiecień: 217.90  

---

## 18. Final instruction dla Claude

Budować projekt jak **production-grade financial analytics tool**, nie demo.

Priorytety:

1. poprawność obliczeń  
2. assignment handling  
3. resilience IBKR integration  
4. test coverage  
5. clean code  

Optymalizować pod maintainability.  
Najpierw dostarczać działające pionowe slice end-to-end, potem rozszerzać.

