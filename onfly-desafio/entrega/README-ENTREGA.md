# Entrega — Consolidador de reservas Onfly

## Defeitos corrigidos

### 1. Conversão de câmbio
- **Arquivo:** `src/onfly_report/services/currency_converter.py`
- **Sintoma:** `tests/test_currency.py:15` falhava com `assert '118.45' == '642.00'` (reservas de 100.00 BRL + 100.00 USD).
- **Causa raiz:** o conversor dividia o valor pela taxa (e passava por `float`) em vez de multiplicar.
- **Correção:** `amount * rate`, retornando `Decimal`.
- **Commit:** `dc28747`

### 2. Contagem de noites
- **Arquivo:** `src/onfly_report/services/stay_calculator.py`
- **Sintoma:** `tests/test_nights.py:19` falhava com `assert -28 == 3` (check-in 2025-01-30, check-out 2025-02-02).
- **Causa raiz:** a função subtraía só o dia do mês (`checkout.day - checkin.day`), ignorando a virada de mês/ano.
- **Correção:** `(checkout - checkin).days * rooms`, fórmula do README.

### 3. Regra de status faturável
- **Arquivo:** `src/onfly_report/services/status_policy.py`
- **Sintoma:** `tests/test_status_rules.py:20` falhava com `assert '120.00' == '100.00'` (100 CONFIRMED + 50 CANCELLED + 20 NO_SHOW).
- **Causa raiz:** `is_billable` usava `status != "CANCELLED"`, então `NO_SHOW` entrava nos totais.
- **Correção:** `status == "CONFIRMED"`, conforme a regra do README ("Só `CONFIRMED` é faturável").

### 4. Ordenação do ranking de hotéis
- **Arquivo:** `src/onfly_report/services/report_builder.py`
- **Sintoma:** `tests/test_top_hotels.py:18` falhava com `['H04', 'H03', 'H02'] == ['H01', 'H02', 'H03']`.
- **Causa raiz:** a chave de ordenação `(amount, hotel_id)` ordenava o valor em ordem crescente, colocando os hotéis de menor valor primeiro.
- **Correção:** chave `(-amount, hotel_id)`: valor decrescente e desempate por `hotel_id` crescente.

### 5. Leitura limitada a 200 reservas
- **Arquivo:** `src/onfly_report/repositories/csv_reservation_repository.py`
- **Sintoma:** o CLI no dataset real mostrava `Reservas: 200`, mas `data/reservas_hoteis.csv` tem 300 reservas. Nenhum teste fornecido pegava isso, porque os testes usam poucas linhas.
- **Causa raiz:** o loop de leitura tinha um `if len(rows) == 200: break` que interrompia a leitura do CSV.
- **Correção:** removi a condição de parada; o `for values in reader` já termina no fim do arquivo.

## Como investiguei

### Abordagem geral
Para cada teste que falhava no `pytest -q`, li a mensagem de erro e a docstring do teste, localizei em `src/onfly_report/services/` a função responsável por aquele cálculo e comparei o comportamento dela com as regras de negócio do `README.md`. Corrigi um defeito por vez e rodei os testes de novo depois de cada correção.

### Câmbio
1. Rodei `pytest -q` e o `test_currency_conversion_to_brl` falhou: esperado `642.00`, obtido `118.45`.
2. O teste diz que o valor deve ser multiplicado pela taxa, sem inverter a operação. A taxa usada (`USD = 5.42`) está em `tests/helpers.py`.
3. Conferi a conta: `100 + 100 × 5.42 = 642.00` (esperado), enquanto `100 + 100 ÷ 5.42 ≈ 118.45` (obtido), o que mostra que o valor estava sendo dividido.
4. Procurei onde a taxa era usada com `grep -rn "rate" src/`, que levou à função `convert_to_brl`.
5. Troquei a divisão por multiplicação e passei a retornar `Decimal`, sem passar por `float`.

### Noites
1. O `test_nights_across_month_boundary` falhou: esperado `3`, obtido `-28`.
2. Procurei em `src/onfly_report/services/` a função que calculava as noites e encontrei `calculate_room_nights`, com `return (checkout.day - checkin.day) * rooms`.
3. Testei como as datas se comportam no Python:
   ```bash
   python3 -c "from datetime import date; d = date(2025,2,2) - date(2025,1,30); print(d, '|', d.days)"
   ```
   A subtração de datas inteiras dá 3 dias, mas o código subtraía só o dia do mês: `2 - 30 = -28`, ignorando a troca de mês.
4. Usei a fórmula do README (`(checkout_date - checkin_date).days * rooms`) com os nomes de variáveis da função, e o teste passou.

### Status
1. O `test_status_rules_exclude_cancelled_and_no_show` falhou: as contagens por status estavam certas, mas `total_amount_brl` deu `120.00` em vez de `100.00`.
2. A diferença de 20.00 é exatamente o valor da reserva `NO_SHOW`, o que indicou que ela estava entrando nos totais.
3. Em `services/status_policy.py`, a função `is_billable` retornava `status != "CANCELLED"`, deixando passar qualquer status diferente de cancelado, inclusive `NO_SHOW`.
4. O README diz que só `CONFIRMED` é faturável, então troquei para `status == "CONFIRMED"`.

### Ranking de hotéis
1. O `test_top_hotels_are_ranked_descending` falhou: o ranking veio na ordem inversa (`H04, H03, H02`).
2. Em `report_builder.py`, o `sorted` usava a chave `(item[1]["amount"], item[0])`, ou seja, ordem crescente de valor.
3. O README pede `total_brl` decrescente com desempate por `hotel_id` **ascendente**. Usar `reverse=True` inverteria a tupla inteira e deixaria o desempate decrescente também.
4. Por isso neguei só o primeiro item da tupla, `(-item[1]["amount"], item[0])`, invertendo o valor e mantendo o desempate crescente.

### Limite de 200 reservas
1. Com os 5 testes passando, rodei o CLI no dataset real:
   ```
   Reservas: 200 | Confirmadas: 153 | Noites: 909 | Total BRL: 249543.79
   ```
2. O arquivo `data/reservas_hoteis.csv` tem 300 reservas (301 linhas com o cabeçalho), então 100 estavam sendo ignoradas.
3. Em `repositories/csv_reservation_repository.py`, o loop de leitura tinha `if len(rows) == 200: break`.
4. Removi a condição inteira em vez de trocar o número, porque o relatório precisa ler o arquivo todo, qualquer que seja o tamanho. Resultado depois da correção:
   ```
   Reservas: 300 | Confirmadas: 235 | Noites: 1347 | Total BRL: 396170.00
   ```

## Como validar

Na pasta `onfly-desafio/`:

```bash
python3 -m venv .venv
source .venv/bin/activate
pip install -r requirements-dev.txt
```

1. **Testes fornecidos** (sem alteração):
   ```bash
   pytest -q
   ```
   Esperado: `5 passed`.

2. **Amostra de validação** (comparar com a tabela do `README.md`):
   ```bash
   PYTHONPATH=src python -m onfly_report \
     --input data/exemplo-validacao.csv \
     --rates data/taxas_cambio.csv \
     --output output/exemplo.json
   ```
   Esperado: `Reservas: 12 | Confirmadas: 9 | Noites: 28 | Total BRL: 7566.00`, com `average_nightly_rate_brl` = `270.21` e `by_month`/`top_hotels` iguais às tabelas do README.

3. **Dataset completo:**
   ```bash
   PYTHONPATH=src python -m onfly_report \
     --input data/reservas_hoteis.csv \
     --rates data/taxas_cambio.csv \
     --output output/report.json
   ```
   Esperado: `Reservas: 300 | Confirmadas: 235 | Noites: 1347 | Total BRL: 396170.00`, com `average_nightly_rate_brl` = `294.11`.

## Resultado do pytest

```
$ pytest -q
.....                                                                    [100%]
5 passed in 0.51s
```
