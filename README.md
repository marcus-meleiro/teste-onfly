# Teste técnico Onfly — Consolidador de reservas de hotel

Solução do desafio técnico da Onfly: encontrar e corrigir os defeitos de um CLI em Python que gera um relatório JSON consolidado de reservas de hotel.

## Estrutura do repositório

```
teste-onfly/
├── README.md                  # este arquivo
├── entrega/                   # cópia da entrega na raiz, para facilitar o acesso
│   ├── README-ENTREGA.md
│   └── uso-de-ia/
│       ├── sessao.jsonl
│       └── declaracao.md
└── onfly-desafio/             # projeto do desafio
    ├── README.md              # enunciado e regras de negócio originais
    ├── src/onfly_report/      # código do CLI (onde estão as correções)
    ├── tests/                 # testes fornecidos, sem alteração
    ├── data/                  # CSVs de reservas, câmbio e amostra de validação
    ├── entrega/               # entrega no local pedido pelo enunciado
    └── output/                # relatório gerado pelo CLI
```

As duas pastas `entrega/` têm o mesmo conteúdo. A que fica dentro de `onfly-desafio/` segue a estrutura pedida no enunciado; a da raiz é uma cópia para quem abre o repositório.

- **`README-ENTREGA.md`:** defeitos corrigidos, causa raiz, investigação, como validar e resultado do `pytest`.
- **`uso-de-ia/sessao.jsonl`:** exportação da sessão com o assistente de IA, com dados pessoais não relacionados ao desafio removidos.
- **`uso-de-ia/declaracao.md`:** ferramenta e modelo usados, e o que foi feito por mim e pela IA.

## Como fiz

1. Instalei o ambiente e rodei `pytest -q`: os 5 testes fornecidos falhavam.
2. Para cada teste que falhava, li a mensagem de erro e a docstring do teste, localizei a função responsável em `src/onfly_report/` e comparei o comportamento dela com as regras de negócio do `onfly-desafio/README.md`.
3. Corrigi um defeito por vez, rodando os testes depois de cada correção.
4. Com os testes passando, rodei o CLI no dataset completo e encontrei um defeito que os testes não cobriam: a leitura parava em 200 de 300 reservas.
5. Validei o resultado na amostra de validação do enunciado, no dataset completo e no Python 3.11.
6. Documentei tudo em `entrega/README-ENTREGA.md`.

### Defeitos corrigidos

| # | Defeito | Arquivo |
|-|-|-|
| 1 | O câmbio dividia pela taxa em vez de multiplicar (e usava `float`) | `services/currency_converter.py` |
| 2 | As noites eram calculadas pelo dia do mês, quebrando na virada de mês/ano | `services/stay_calculator.py` |
| 3 | `NO_SHOW` entrava nos totais (só `CONFIRMED` é faturável) | `services/status_policy.py` |
| 4 | O ranking de hotéis estava em ordem crescente | `services/report_builder.py` |
| 5 | A leitura do CSV parava em 200 linhas | `repositories/csv_reservation_repository.py` |

Os detalhes de cada um estão em [`entrega/README-ENTREGA.md`](entrega/README-ENTREGA.md).

## Como rodar

```bash
cd onfly-desafio
python3.11 -m venv .venv
source .venv/bin/activate
pip install -r requirements-dev.txt

pytest -q

PYTHONPATH=src python -m onfly_report \
  --input data/reservas_hoteis.csv \
  --rates data/taxas_cambio.csv \
  --output output/report.json
```

Resultado esperado:

- **pytest:** `5 passed`
- **CLI:** `Reservas: 300 | Confirmadas: 235 | Noites: 1347 | Total BRL: 396170.00`

## Agradecimento

Agradeço ao time da Onfly pela oportunidade de participar deste processo seletivo. Gostei de trabalhar no desafio: investigar números errados a partir de testes e regras de negócio bem definidas é o tipo de problema que me motiva.

Tenho muito interesse em fazer parte da Onfly, contribuir com o produto e crescer junto com o time. Fico à disposição para conversar sobre a solução e sobre os próximos passos.
