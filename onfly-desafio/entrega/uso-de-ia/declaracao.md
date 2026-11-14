# Declaração de uso de IA

## Ferramenta e modelo

- **Ferramenta:** Claude Code (aplicativo desktop do Claude, aba Code)
- **Modelo:** Claude Opus 5.5 (`claude-opus-5-5`)
- **Sessão:** uma única sessão, exportada em `sessao.jsonl` (formato nativo do Claude Code). Antes de publicar, removi dados pessoais que o aplicativo injeta automaticamente no contexto e que não têm relação com o desafio: e-mail, listagem de arquivos da pasta home, nomes de outros repositórios e memórias de outras sessões. Cada trecho removido está marcado como `[... removido ...]`. As mensagens e as respostas da conversa não foram alteradas.

## O que eu fiz

- Instalei o ambiente e rodei `pytest -q` para ver as falhas iniciais.
- Investiguei e corrigi os defeitos de câmbio, contagem de noites, regra de status e ordenação do ranking de hotéis: li cada teste que falhava, localizei a função em `src/onfly_report/services/` e comparei com as regras do `README.md`.
- Rodei o CLI no dataset completo e percebi que ele processava só 200 reservas, de um arquivo de 300.
- Removi a condição de parada do loop de leitura.
- Rodei os testes e o CLI depois de cada correção.
- Validei o CLI e o `pytest` no Python 3.11.18, usando `uv`.
- Fiz todos os commits e o push.

## O que a IA fez

- **Leitura e conferência:** leu o README e o código do CLI e conferiu as correções que eu já tinha feito no início.
- **Validação independente:** escreveu um cálculo separado, direto das regras do README, e comparou com a saída do CLI na amostra de validação e no dataset completo.
- **Códigos de saída:** verificou o comportamento do CLI com entradas inválidas.
- **Defeito do limite de 200 linhas:** quando eu relatei que o CLI mostrava 200 reservas, apontou o `if len(rows) == 200: break` como causa e explicou por que remover a condição inteira, em vez de trocar o número.
- **Documentação:** redigiu o `entrega/README-ENTREGA.md` a partir das explicações que eu fui passando sobre cada correção, e preencheu a seção "Como validar".
- **Entrega:** montou a checklist do que faltava entregar, exportou esta sessão e redigiu esta declaração.

A IA não alterou o código-fonte nem os testes. Todas as alterações em `src/` foram feitas por mim.
