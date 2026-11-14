# Entrega — Consolidador de reservas Onfly

## Defeitos corrigidos

### 1. Conversão de câmbio
- **Arquivo:** `src/onfly_report/services/currency_converter.py`
- **Sintoma:** `tests/test_currency.py:15` falhava com `assert '118.45' == '642.00'` (reservas de 100.00 BRL + 100.00 USD).
- **Causa raiz:** o conversor dividia o valor pela taxa (e passava por `float`) em vez de multiplicar.
- **Correção:** `amount * rate`, retornando `Decimal`.
- **Commit:** `dc28747`

## Como investiguei

### Câmbio
1. Rodei `pytest -q` e o `test_currency_conversion_to_brl` falhou: esperado `642.00`, obtido `118.45`.
2. O teste diz que o valor deve ser multiplicado pela taxa, sem inverter a operação. A taxa usada (`USD = 5.42`) está em `tests/helpers.py`.
3. Conferi a conta: `100 + 100 × 5.42 = 642.00` (esperado), enquanto `100 + 100 ÷ 5.42 ≈ 118.45` (obtido), o que mostra que o valor estava sendo dividido.
4. Procurei onde a taxa era usada com `grep -rn "rate" src/`, que levou à função `convert_to_brl`.
5. Troquei a divisão por multiplicação e passei a retornar `Decimal`, sem passar por `float`.

## Como validar

<!-- a preencher -->

## Resultado do pytest

<!-- a preencher -->
