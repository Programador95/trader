# CLAUDE.md — Projeto MESA

Este arquivo é carregado automaticamente pelo Claude Code em toda sessão. Ele define **como** trabalhar. O **quê** e o **porquê** estão no `PLANO_MESTRE.md`.

## Papéis

- **Alison:** operador e auditor. Decide os portões, aprova as hipóteses e autoriza qualquer gasto. Responda sempre em português do Brasil.
- **Claude (você):** executor. Escreve, roda e testa o código e registra tudo. Propõe, mas não decide portões.

## Início de toda sessão (obrigatório, nesta ordem)

1. Ler a seção 13 (Estado atual) do `PLANO_MESTRE.md` e a última entrada do `LEDGER.md`.
2. Rodar `python scripts/setup_ambiente.py --check`.
3. Responder em até 5 linhas: fase atual, último passo concluído, próximo passo, pendências e se o ambiente está OK.
4. Esperar a instrução do Alison.

## Regras permanentes

1. **Uma fase por vez.** Nunca avance de fase sem "ok" explícito do Alison. Dentro de uma fase, execute os passos sem pedir licença para cada um.
2. **Verificar antes de fazer.** Se algo já existe e passa na verificação, pule e registre "já estava pronto". Todo script deve ser idempotente: rodar duas vezes não estraga nada.
3. **Dinheiro e contas.** Nunca compre, cadastre, envie ordem real ou use chave real. Nas fases F4 e F5, peça ao Alison para fazer o passo manual.
4. **Segredos.** Chaves de API ficam só no `.env`, criado pelo Alison a partir do `.env.example`. Nunca imprima, registre, versione ou peça a chave no chat.
5. **Regras da mesa só no `config.yaml`.** Nenhum número da mesa fixo no código. Se a documentação oficial contradizer o config, pergunte antes de mudar.
6. **Ambiente Python.** Use sempre o Python do `.venv` do projeto (Windows: `.venv\Scripts\python.exe`; Mac/Linux: `.venv/bin/python`). Nada de pip global. Pacote novo vai para o `requirements.txt`, com o motivo registrado no LEDGER.
7. **Pré-registro.** Nenhum backtest roda sem a hipótese pré-registrada no LEDGER e a grade aprovada pelo Alison.
8. **Tudo no LEDGER.** Cada teste entra no LEDGER, inclusive os que falham, com o número de variantes. Nunca apague entradas; correções viram entrada nova. Atualize os contadores no topo.
9. **Cofre trancado.** Os dados do período do cofre (datas no `config.yaml`) nunca são carregados, exceto por comando explícito do Alison para uma hipótese finalista, uma única vez. A trava fica no código de carga de dados, não na boa vontade.
10. **Sem olhar o futuro.** Sinal calculado no fechamento do candle t só executa a partir do candle t+1.
11. **Conservadorismo.** Custos sempre ligados, slippage no teto publicado. Se stop e alvo caem no mesmo candle, assuma o stop.
12. **Fim de sessão.** Atualize a seção 13 do `PLANO_MESTRE.md` e o LEDGER, e entregue um resumo com o próximo passo.
13. **Na dúvida, pergunte.** Não invente regra da plataforma nem preencha lacuna com suposição silenciosa.

## Comandos

- Checar o ambiente: `python scripts/setup_ambiente.py --check`
- Preparar o ambiente: `python scripts/setup_ambiente.py`
- Rodar os testes: `<python do .venv> -m pytest testes -q`

## Convenções

- **Horários:** armazenar em UTC, exibir no horário de Brasília. As regras da mesa usam America/New_York.
- **Dados:** Parquet em `dados/<fonte>/<mercado>/<tempo_grafico>/`.
- **Resultados:** em `resultados/`, com data e versão no nome (ex.: `2026-10-02_H1_v1.md`).
- **Nomes e estrutura:** conforme a seção 11 do plano. Comentários e relatórios em português.
- **Testes:** todo módulo em `src/` tem teste em `testes/`.

## Fases: checklist executável

### F0 — Ambiente

1. Rodar `python scripts/setup_ambiente.py`. Ele pula o que já existe e só faz o que falta.
2. Rodar `--check` até aparecer AMBIENTE PRONTO.
3. Registrar no LEDGER "F0 concluída", com as versões do Python e dos pacotes.

**Pronto quando:** o `--check` retorna AMBIENTE PRONTO. Parar e pedir ok para a F1.

### F1 — Dados (M1)

1. Antes de baixar qualquer coisa, propor ao Alison: mercados, tempos gráficos, período, fonte e tamanho estimado em disco. Proposta padrão:
   - **Cripto:** BTC, ETH, SOL e mais 3 a 5 mercados líquidos, futuros perpétuos USDT-M da Binance (dados públicos, sem cadastro), candles de 1 minuto desde o início disponível. Tempos gráficos maiores são agregados a partir deles.
   - **Índices (NAS100, SP500):** pesquisar fontes gratuitas de histórico intradiário e apresentar as opções com prós e contras. Não baixar sem ok.
2. Criar `src/dados.py`: download idempotente e retomável, que pula o que já existe e checa a integridade. Salvar em Parquet, em UTC.
3. Validar buracos, duplicatas, OHLC inválido (máxima menor que mínima etc.) e fuso. Relatório em `resultados/F1_relatorio_dados.md`.
4. Propor a divisão desenvolvimento / validação / cofre por datas (~60/20/20, com o cofre no período mais recente) e gravar no `config.yaml` depois do ok. A partir daqui, o cofre fica trancado.
5. Testes em `testes/test_dados.py`.

**Pronto quando:** dados validados, divisão gravada e LEDGER atualizado.

### F2 — Motor, Risco e Simulador (M2, M4, M5)

1. **`src/motor.py`:** backtest candle a candle, com:
   - comissão por execução e swap por hora, lidos do config;
   - slippage no teto publicado;
   - stop disparando no preço negociado, com a regra do mesmo candle;
   - perda diária com reset à meia-noite de NY (incluindo a troca de horário de verão), calculada sobre a equity;
   - piso estático e aprovação ao atingir a meta.
2. **`src/risco.py`:** margem viva; R = margem ÷ K; tamanho = R ÷ distância do stop; faixa [d_min, d_max]; arredondamento; posições correlacionadas contam juntas.
3. **`src/simulador_mesa.py`:** Monte Carlo com reamostragem em blocos da lista de trades. Calcula P(passar), distribuição de trades e dias até passar, P(quebrar), saques esperados em 90 dias de conta financiada (com a trava do piso) e EV contra o preço da conta. Também compara políticas de risco dependentes do estado da conta (ex.: reduzir R ou travar lucro perto da meta) pelo efeito em P(passar).
4. **Baselines** em `src/estrategias/`: `baseline_sempre_long.py` e `baseline_aleatorio.py`.
5. **Testes obrigatórios**, com valores da documentação oficial:
   - comissão de $3 numa execução de $10.000 em cripto;
   - swap de $0,125 por hora sobre $10.000 em cripto;
   - piso em $9.700 e trava em $10.000 depois do primeiro saque;
   - reset diário correto na virada do horário de verão;
   - tamanho = R ÷ distância do stop.
6. **Teste de sanidade:** passeio aleatório sem custos, com alvo de +9% e piso de −3%, precisa dar aprovação de ~25% (±2 p.p. com 10 mil simulações). Com custos, o número tem que cair.

**Pronto quando:** todos os testes passam e a sanidade confere. Relatório em `resultados/F2_validacao.md`.

### F3 — Laboratório

Para cada hipótese da seção 6 do plano, na ordem H1 → H2 → H3 → H4 → H5:

1. Escrever o pré-registro no LEDGER (modelo no fim do arquivo), apresentar a grade ao Alison e esperar o ok.
2. Rodar nos períodos de desenvolvimento e validação, com walk-forward, usando as saídas S1–S6 da seção 7.
3. Comparar com os baselines, rodar o M5 e checar a robustez entre ativos.
4. Registrar o resultado no LEDGER (modelo), inclusive quando a hipótese morrer.
5. Uma finalista só vai ao cofre com ok explícito, uma única vez. Registrar a abertura.

Ao final, gerar `resultados/F3_relatorio.md` com a tabela comparativa, a recomendação de mercados, horários, tempo gráfico e K (seção 10 do plano) e a verificação do G1. **O G1 é decidido pelo Alison.**

### F4 — Sandbox (exige a conta; tem passos manuais do Alison)

1. Pedir ao Alison para comprar a conta, criar a chave de teste (ambiente Test/Sandbox) e colocá-la no `.env` como `MFP_API_KEY_TESTE`.
2. Criar `src/executor/` com uma interface única e três implementações (`papel.py`, `sandbox.py`, `real.py`), usando chaves de idempotência, tratamento de erros e limites de requisição. Ler a política da conta pela API antes de operar.
3. Criar `src/guardiao.py` (M7), independente da estratégia, com kill switch.
4. Operar no sandbox, registrar cada trade em CSV e gerar um relatório semanal comparando o resultado com a faixa prevista pelo M5.

**Pronto quando:** o G2 for atingido (seção 9). A decisão é do Alison.

### F5 — Avaliação

Só com ok explícito. O Alison cria a chave real e a coloca no `.env`. Trocar o host e a chave, sem nenhuma outra mudança de código. Acompanhar diariamente.

### F6 — Financiada

Aplicar a política de saque definida no M5. O saque é sempre feito manualmente pelo Alison no painel.

## Critérios de morte

Se um portão falhar (seção 9 do plano): pare, registre e volte à fase indicada. Nunca contorne um portão.
