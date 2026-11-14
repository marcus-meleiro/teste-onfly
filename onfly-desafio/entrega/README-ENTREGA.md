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

## Como investiguei

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

## Como validar

<!-- a preencher -->

## Resultado do pytest

<!-- a preencher -->
