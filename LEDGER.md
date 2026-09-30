# LEDGER — Projeto MESA

Registro permanente do projeto. Nada é apagado: correções entram como entrada nova.
Entradas numeradas E001, E002…; hipóteses seguem a numeração da seção 6 do plano (H1, H2…).

## Contadores

- Hipóteses pré-registradas: 16 (H1, H1b, H2, H3, H4, H4b, H5, H7 e H8, mortas; H6: cripto morta, Nasdaq V1 confirmada pela regra do E017 mas sem G1, com EV negativo, E027; H9 morta por 0,002R no critério (b), E034; H10 morta, sem efeito nem antes dos custos, E037; H11 funil de 29 padrões, morta na triagem, E041; H12 cascatas em cripto, morta na triagem, E045; H13 e H14 aprovadas, rodando, E047)
- Variantes testadas (total de tentativas): 118 (H1: 30; H1b: 4; H2: 16; H3: 4; H4: 8; H4b: 1; H5: 8; H6: 4; H7: 2; H8: 2; H9: 1; H10: 1; H11: 29; H12: 8)
- Usos da validação: 3 (H2, 2 combinações; H4b, 1 variante; H6, 1 variante)
- **Estado: F3. H13 (filtro de regime pelo VIX na H9) e H14 (desconto + absorção em cripto) aprovadas e rodando (E047). Sem gasto.**
- Aberturas do cofre: 0
- Gasto acumulado: $0

## Entradas

### E001 — 2026-09-27 — Planejamento

- Plano mestre v0.5 concluído no chat, com o kit do projeto (CLAUDE.md, config.yaml, LEDGER, setup_ambiente.py).
- Próximo passo: F0 (ambiente) no Claude Code.

### E002 — 2026-09-28 — F0 concluída

- `scripts/setup_ambiente.py` rodado uma vez para preparar e depois com `--check`: **AMBIENTE PRONTO** (exit 0). A segunda rodada não alterou nada (idempotente).
- Criados: pastas `dados/`, `resultados/`, `testes/`, `src/`, `src/estrategias/`, `src/executor/` (com `__init__.py`), `.venv` e repositório git local (sem commits).
- Python do sistema e do `.venv`: **3.14.3** (Windows 11).
- Pacotes no `.venv`: pandas 3.0.6, numpy 2.5.3, pyarrow 25.0.1, requests 2.34.2, pyyaml 6.0.3, matplotlib 3.11.2, pytest 9.1.1, python-dotenv 1.2.3, tzdata 2026.4. Nenhum pacote fora do `requirements.txt`.
- Conferido: fuso America/New_York carrega; `config.yaml` válido (saldo 10000, DD 3%); `pytest` roda (ainda sem testes); `.env` ausente (esperado até a F4); variável `ANTHROPIC_API_KEY` não definida (o Claude Code usa a assinatura).
- Observação para a F1/F2: pandas 3.0 tem copy-on-write e tipo string por padrão; o código novo deve ser escrito para essa versão.
- Próximo passo: F1 (dados), aguardando o ok do Alison.

### E003 — 2026-09-28 — Versões dos pacotes travadas

- Decisão do Alison: travar as versões para que o mesmo backtest dê o mesmo resultado em qualquer reinstalação.
- `requirements.txt` passou de mínimos (`>=`) para versões exatas (`==`): os 9 pacotes usados diretamente e as 17 dependências deles, iguais às do E002. Nenhum pacote novo; nenhuma versão mudou.
- `scripts/setup_ambiente.py`: a checagem de pacotes agora também compara as versões instaladas com as travadas (estado `VERSAO` quando divergem; sem `--check`, reinstala a versão travada).
- Testado: `--check` com o ambiente atual dá AMBIENTE PRONTO; a rodada completa não reinstala nada; um arquivo de teste com versão errada e pacote ausente foi apontado corretamente.
- Regra daqui em diante: atualizar ou incluir pacote é decisão do Alison, com os testes rodados e o motivo registrado aqui.
- Próximo passo: F1 (dados), aguardando o ok do Alison.

### E004 — 2026-09-28 — Revisão antes da F1 (fatos verificados)

- `GET /v1/markets` da MyFundedPerps é **público** (sem chave): 121 mercados, 105 na Binance e 16 na Hyperliquid. Retrato salvo em `resultados/2026-09-28_mercados_mfp.json`.
- **Não existe NAS100.** Índices: `xyz:SP500` e `xyz:XYZ100` (Hyperliquid, alavancagem máx. 12); há também `QQQUSDT` (Binance, desde 2026-04-06). AAPL está na Hyperliquid (`xyz:AAPL`); GOOGL na Binance (`GOOGLUSDT`, desde 2026-03-26).
- Cripto principal na Binance, o mesmo venue da execução: BTC desde 2019-09-08, ETH desde 2019-11-27, SOL desde 2020-09-14 (alavancagem máx. 10/10/8).
- Hyperliquid: a API pública devolve só os ~5.000 candles mais recentes (1m ≈ 3,5 dias; 15m ≈ 52 dias; 1h ≈ 7 meses). Ações e commodities da Binance foram listadas entre jan. e abr. de 2026. Só cripto tem histórico longo no venue certo.
- `xyz:SP500` e `xyz:XYZ100` negociam 24/7, inclusive no fim de semana (últimos 60 dias: volume por hora no fim de semana ≈ 13% e 18% do volume do pregão de NY). O plano (seção 3) diz que índices têm horário de fechamento: falta confirmar se a plataforma bloqueia operação fora do pregão.
- A documentação não diz qual preço (mark ou último negócio) mede a equity para piso, perda diária e meta, nem se há regra de inatividade ou restrição de país.
- Disco livre: 29,5 GB. Nenhuma regra do `config.yaml` foi alterada.
- Pontos de protocolo levantados para decisão do Alison (N mínimo, critério do G1, barra por número de tentativas, contaminação do cofre na H1, compra na promoção): ver resposta da sessão de 2026-09-28; entram no plano só depois do ok.

### E005 — 2026-09-28 — Decisões do Alison e fatos novos (plano v0.7)

- **Decisões do Alison:**
  - índices só no pregão regular de NY (9h30–16h ET), zerando antes do fechamento;
  - Nasdaq entra no projeto (o mercado da plataforma é o `xyz:XYZ100`);
  - critério de amostra delegado ao Claude, que adotou: limite pessimista do edge > 0, barra corrigida por Bonferroni pelo número de variantes e piso de 300 trades (seção 5 do plano);
  - não há regras do "print": a H1 usa regras de MACD da literatura (pesquisa em `resultados/2026-09-28_pesquisa_macd.md`);
  - seguir com tudo o que não exige a conta;
  - comprar na promoção se o dinheiro entrar.
- **Fatos verificados na documentação:**
  - a avaliação falha após **30 dias corridos sem trade executado**, e a financiada é encerrada após **7 dias**; login, ordem não executada ou cancelada não contam;
  - a perda diária é a equity do início do dia (com P&L aberto) menos 3% do saldo inicial;
  - não diz se a equity usa mark ou último negócio.
- `XYZ100` acompanha o Nasdaq-100. No pregão, o preço de referência é o próprio índice; fora dele, vem do futuro NQ da CME. O contrato é perpétuo (sem vencimento nem rolagem). Preço em 28/09: XYZ100 30.291,5 e SP500 7.684,75.
- Dukascopy oferece Nasdaq-100 e S&P 500 (CFD) em 1m desde ~2012, grátis e sem cadastro. Candidato a proxy.
- **Pendente de ok do Alison:**
  - incluir a inatividade no `config.yaml` (o config atual diz só "sem prazo");
  - acrescentar ao G1 "EV acima do EV do baseline aleatório";
  - a proposta de download da F1.
- Variantes testadas: 0. Nenhum backtest rodado.

### E006 — 2026-09-28 — Como a mesa mede a perda diária e o piso

Dúvida do Alison: perda diária intradiária ou no fim do dia? Resposta na documentação oficial:
- **Intradiária e contínua.** O monitoramento dos limites de perda roda o tempo todo enquanto a conta está ativa, inclusive depois de cada trade e de cada cobrança de swap. Não existe checagem de fim de dia no Select; o drawdown é estático e não acompanha máximas intradiárias nem de fim de dia.
- **Medida pelo mark.** Pelo glossário, a equity é o valor da conta com as posições abertas avaliadas pelo mark, e o mark é o preço de referência da plataforma usado na avaliação da conta e nas checagens de risco. O stop dispara pelo último negócio (basta um negócio tocar o nível); os limites de perda e a liquidação usam outras referências de preço.
- **Âncora diária** = equity à meia-noite de NY, incluindo o P&L aberto naquele momento; limite = âncora − $300.
- **Tocar já quebra:** antes do 1º saque, atingir o piso quebra a conta, e o limite diário também vale no valor exato. Depois da trava do piso, ficar exatamente em $10.000 é permitido e abaixo disso quebra.
- O stop não garante saída antes da quebra: ele executa a mercado, com preço novo.
- **Consequência para o motor (F2):** checar piso e perda diária a cada candle de 1m pelo pior mark do candle, e disparar o stop pelo pior preço negociado. Para isso, a F1 passa a propor baixar também o mark de 1m da Binance (público: arquivos mensais de 2020-01 a 2026-08, com checksum).
- Fontes: `guides/glossary.md`, `guides/challenges-and-rules.md`, `guides/trading-guide.md`, `guides/challenge-lifecycle/evaluation-progress.md` (docs.myfundedperpetuals.com, lidos em 2026-09-28).

### E007 — 2026-09-28 — Saques, split e histórico da mesa

- **Split:** o padrão é 80/20. O 90/10 é um adicional pago escolhido só no checkout e não pode ser incluído depois. O `config.yaml` diz 0,9 "a confirmar": corrigir para 0,8 depende do ok do Alison (regra 5).
- **Saques:** abertos só em 2026-09-11 para todas as contas, então não há histórico de pagamentos da MyFundedPerps. A mesa é do grupo MyFunded (MyFundedFutures, que tem bom histórico de pagamentos), com tecnologia da Funding Perpetuals.
- **Trava no saque:** a operação fica travada enquanto o pedido está em análise, e a análise pode passar de um dia. A documentação não diz se a janela de inatividade de 7 dias da financiada pausa nesse período. Pergunta para o suporte.
- Parte do saque pode virar crédito da loja com bônus de 10% (expira em 45 dias; não vira dinheiro). Para o M5, contar só a parte em dinheiro.
- A plataforma **simula** as execuções com preços das bolsas e não envia ordens a elas. O preço de execução segue fórmula publicada (teto de slippage sobre o mark), então o backtest pode reproduzir o custo quase exatamente.
- Fontes: `guides/challenge-lifecycle/payouts.md`, `buying-a-challenge.md`, `becoming-funded.md`, `guides/glossary.md`; busca sobre a reputação (proptradingvibes.com, 2026-09-10).

### E008 — 2026-09-28 — F1 iniciada: divisão gravada e código de dados

- **Orientação do Alison:**
  - a mesa é a MyFundedPerps; MNQ e EOD foram só exemplos;
  - o foco é achar uma estratégia consistente e vencedora e validá-la nos backtests;
  - operar todo dia, a qualquer hora, sempre que houver EV; uns 2 trades por dia seria bom, sem expectativa fixa;
  - a armadilha do saque e o PC ligado ficam para depois.
- **`config.yaml` atualizado com as regras verificadas (E004–E007):**
  - inatividade de 30 e 7 dias;
  - equity pelo mark, monitorada continuamente, e tocar o limite quebra;
  - fórmula da ordem a mercado;
  - split padrão 0,8 (a confirmar na compra);
  - índices só no pregão.
- **Divisão gravada antes de abrir qualquer dado** (UTC, inclusiva):
  - desenvolvimento até 2023-11-30;
  - validação de 2023-12-01 a 2025-04-30;
  - cofre de 2025-05-01 a 2026-08-31;
  - forward a partir de 2026-09-01, também trancado.
  - Cripto começa em 2020-01 (início do arquivo público da Binance), o que dá ≈ 59/21/20. Índices começam em 2012-01, com as mesmas datas de corte.
- **Transparência:** na sondagem do formato dos arquivos, a primeira linha de ago/2026 (período do cofre) do BTC apareceu na tela: um preço de abertura de 1 minuto, sem retorno nem análise. Depois disso a trava foi implementada e as sondagens passaram a usar só datas do desenvolvimento.
- **`src/dados.py`:**
  - download idempotente e retomável, com checksum SHA-256 na Binance e verificação LZMA na Dukascopy;
  - Parquet mensal em UTC;
  - checagem estrutural (buracos, duplicatas, OHLC inválido, fuso, cobertura do pregão) gravada em manifesto no download;
  - relatório que lê só os manifestos;
  - `carregar()` com a trava do cofre.
- `pytest.ini` criado (`pythonpath = .`) para os testes importarem `src`. Nenhum pacote novo.
- **Testes:** `testes/test_dados.py`, 19 passando.
- A Dukascopy não responde por HTTPS nesta rede; o download usa HTTP (dados públicos; a descompactação LZMA confere a integridade).
- Coletor da Hyperliquid rodado: XYZ100 e SP500 em 1m, 15m e 1h.
- Variantes testadas: 0. Nenhum backtest rodado.

### E009 — 2026-09-28 — F1: Binance concluída; buracos do arquivo preenchidos pela API

- Binance, 6 mercados × (último negócio + mark), 2020-01 a 2026-08: 12 séries completas, 1,6 GB. Uma segunda rodada não baixou nada (idempotente).
- O arquivo mensal público tinha dias inteiros faltando: SOL e XRP (5 dias em 2022) no último negócio e vários dias no mark de todos. Novo `completar_binance()` preenche pela API (`fapi/v1/klines` e `markPriceKlines`) só os minutos ausentes. Para achar os buracos, lê só a coluna de tempo, e o arquivo mensal tem precedência. Com isso, o último negócio ficou 100% completo e o mark ficou com ≤ 0,003% faltando (dois trechos de 29 e 24 min em 2020, nas mesmas datas em todos os mercados).
- Dukascopy: a primeira rodada caiu duas vezes por bloqueio do servidor (HTTP 503) e não perdeu nada, porque retoma de onde parou. Agora faz 1 pedido por vez, com pausa, espera até 10 min entre tentativas e reinicia sozinha; ainda em andamento.
- Testes: 21 no `test_dados.py` (novos: espera em 503 e preenchimento pela API).

### E010 — 2026-09-28 — F2 concluída: motor, risco e simulador

- Ok do Alison para começar a F2 em paralelo ao download dos índices (a F2 não depende de dados reais).
- **`src/motor.py` (M2):**
  - execução em candles de 1m (último negócio + mark), estratégia em tempo gráfico maior;
  - sinal executa no minuto seguinte ao fechamento;
  - ordem a mercado no teto de slippage sobre o mark;
  - stop pelo último negócio, com saída na abertura se o minuto já abrir além do stop;
  - alvo como ordem limitada, que exige o mark;
  - stop e alvo no mesmo minuto: stop;
  - comissão por execução e swap por hora cheia;
  - modo "conta" com a classe `Conta`: piso inclusivo, trava no 1º saque, perda diária com âncora à meia-noite de NY incluindo o P&L aberto, aprovação ao tocar a meta, inatividade;
  - modo "trades" para o M5;
  - saídas S1–S6 (S5 pelo sinal); janela de pregão; custos de estresse 1,5×.
- **`src/risco.py` (M4):** margem viva, R = margem/K, tamanho = R/distância, faixa [d_min, d_max], arredondamento para baixo, custo em R e risco somado por grupo de correlação.
- **`src/simulador_mesa.py` (M5):**
  - Monte Carlo com reamostragem em blocos de dias;
  - avaliação: meta, piso, perda diária, inatividade, pior e melhor momento do trade;
  - financiada de 90 dias com política de colchão, trava do piso, split e saque mínimo;
  - EV e comparação de políticas de risco (margem, fixo, reduz perto da meta).
- **Baselines:** `baseline_sempre_long.py` e `baseline_aleatorio.py`.
- **`config.yaml`:** custos por mercado (classe, faixa de OI conservadora, casas decimais do tamanho), `R_min_usd`, `R_nominal_backtest`, grupos de correlação e teto de risco por grupo (provisórios).
- **Testes:** 64 passando (todos os obrigatórios do CLAUDE.md incluídos).
- **Validação** (`resultados/F2_validacao.md`, gerado por `scripts/validacao_f2.py`, só dados sintéticos):
  - sanidade: passeio aleatório sem custos aprova **24,7%** (exigência 25% ± 2 p.p.); com custo de 0,05R cai para 2,8%;
  - a tabela da seção 8 do plano foi reproduzida com diferença de 0,5 a 4,5 p.p.;
  - o motor não tem viés num passeio aleatório (R médio +0,007 ± 0,037 e −0,009 ± 0,040);
  - motor: 5 s para 7 anos de minutos; simulador: 4 s para 10 mil avaliações.
- **Correção do que eu disse no chat:** eu havia dito que "EV > 0" poderia ser atingido sem vantagem. A simulação mostrou que, com preço de $40 e split de 80%, o EV sem vantagem fica negativo em todos os K testados (−$9 a −$27). Só perto do limite (promoção de $32 + split de 90% + K = 4) chegaria a ~zero. A proposta de G1 continua de pé como proteção, mas o problema é menor do que eu havia dito.
- Variantes testadas em dados reais: 0. Nenhum backtest em dados reais.

### E011 — 2026-09-28 — F1 concluída; calendário da NYSE

- **Dukascopy completa:** Nasdaq-100 (USATECHIDXUSD) e S&P 500 (USA500IDXUSD), 1m de 2012-01 a 2026-08, 176 meses cada. O download foi interrompido duas vezes por bloqueio (503) e uma vez pelo reinício da sessão, sem perda nenhuma. Uma segunda rodada não baixou nada. Total em disco: 1,8 GB.
- **Achado:** o CFD da Dukascopy (como o futuro) negocia nos feriados da NYSE até 13h. Pela regra antiga, o robô trataria esses dias como pregão.
  - **Correção:** novo `src/calendario.py`, com as regras da NYSE (feriados, fechamentos antecipados às 13h e fechamentos especiais de 2012, 2018 e 2025) e testes conferidos com os calendários oficiais de 2024 a 2026.
  - O motor, com janela, não entra nos feriados e zera às 12h59 nos fechamentos antecipados. Novo `operacao.pregao_fim_antecipado` no config.
- **Qualidade no pregão** (`resultados/F1_relatorio_dados.md`, calculado só com os horários dos candles):
  - Nasdaq-100: de 2015 em diante falta < 0,5% dos minutos; em 2013 houve 12 pregões sem nenhum dado e 5,8% faltando.
  - S&P 500: 2012–2014 com 6–8% faltando; 2017–2019 com 1–4%; de 2020 em diante ≤ 0,2%.
  - Os buracos são quase todos de 1 a 5 minutos: minutos sem negociação, que o download descarta por não trazerem informação (o preço não mudou). Falhas de mais de 30 minutos acontecem de 0 a 2 vezes por ano, só até 2014.
- **Proposta ao Alison:** nos índices, pesquisar a partir de 2015. O desenvolvimento fica com ~9 anos (2015 a nov/2023), sem as falhas de 2012–2014.
- **Binance:** completa (E009). Coletor da Hyperliquid rodado de novo nesta sessão.
- **Testes:** 71 passando.
- **F1 pronta:** dados validados, divisão gravada e LEDGER atualizado. Próximo: ok do Alison para a F3.
- Variantes testadas em dados reais: 0.

### E012 — 2026-09-29 — Decisões do Alison ("ok geral") e início da F3

- **F3 liberada**, com a grade da H1 de `resultados/2026-09-28_H1_proposta_grade.md`.
- **Filtro de custo máximo em R desligado na etapa 1** do laboratório. Os custos continuam integrais em todo trade, com estresse de 1,5×. O limite de custo (d_min) será testado nas sobreviventes, como variante contada. No `config.yaml`: `custo_max_em_R: null`.
- **Nova redação do G1 aprovada** (seção 9 do plano).
- **Índices a partir de 2015** na pesquisa (`dados.indices_inicio_pesquisa`).
- **Motor:** as séries de minuto passaram para arrays compactos (~4× menos memória; o PC tem 7,7 GB e o Chrome e o app do Claude ocupam boa parte), e o `backtest` aceita séries já preparadas, para reaproveitá-las entre variantes. Os 71 testes continuam passando.

### H1 — Cruzamento de MACD — pré-registro — 2026-09-29
- **Família e ideia:** tendência/momento. Cruzamentos do MACD (EMA rápida − EMA lenta; linha de sinal = EMA do MACD), com regras clássicas da literatura e parâmetros fixos, sem otimização. Base: `resultados/2026-09-28_pesquisa_macd.md`.
- **Mercados e tempo gráfico:**
  - Cripto (perpétuos da Binance, 24h: BTC, ETH, SOL, XRP, BNB, DOGE): 15m, 1h e 4h.
  - Índices (Nasdaq-100 e S&P 500, proxy Dukascopy, desde 2015-01-01): 15m e 1h. Os candles são só do pregão regular da NYSE (calendário oficial), com o 1h alinhado às 9h30; o robô zera no último minuto do pregão.
- **Regra de entrada (codificável):** sinal no fechamento do candle t, execução no minuto seguinte.
  - **R1:** o MACD cruza a linha de sinal (para cima: compra; para baixo: venda).
  - **R2:** o MACD cruza o zero.
  - **R3:** R1, mas só a favor do lado do zero (compra com MACD > 0; venda com MACD < 0).
  - Só entra zerado; sinais com posição aberta são ignorados.
  - Aquecimento: sem sinal nos primeiros 3 × (lenta + sinal) candles.
- **Stop e saídas:**
  - Etapa 1: S2 (stop a 1,5 ATR(14) do tempo gráfico; alvo de 2R).
  - Etapa 2, só para sobreviventes:
    - S1: cripto com stop de 1% e alvo de 2%; índices com 0,25% e 0,5%;
    - S3: stop de 1,5 ATR, com stop móvel a 3 ATR do melhor preço;
    - S4: stop de 1,5 ATR, sai depois de 12 candles;
    - S5: stop de 1,5 ATR, sai no cruzamento contrário;
    - S6: stop de 1,5 ATR, alvo de 2R, empate depois de +1R.
- **Parâmetros livres e grade:** regra (R1, R2, R3) × conjunto (12/26/9 e 8/17/9) × tempo gráfico. Nenhum parâmetro é otimizado.
- **Número de variantes:** etapa 1 = **30** (18 de cripto + 12 de índices); etapa 2 = 5 por sobrevivente.
- **Custos:** comissão, swap por hora e slippage no teto publicado; rodadas com custos 1× e 1,5×.
- **Aprovação na etapa 1** (período de desenvolvimento, com os trades de todos os mercados do grupo somados; a variante precisa cumprir tudo):
  - (a) ≥ 300 trades;
  - (b) limite pessimista do R líquido médio > 0 com z ≥ 2,94 (5% ÷ 30 variantes, unilateral). O erro-padrão é agrupado por dia de NY, porque trades do mesmo dia em mercados correlacionados não são independentes;
  - (c) R líquido médio > 0 com custos 1,5×;
  - (d) R líquido médio acima dos dois baselines, com a mesma saída: sempre comprado, e aleatório com a mesma frequência de sinais (média de 3 sementes);
  - (e) R médio > 0 em pelo menos 4 das 6 criptos, ou nos 2 índices.
  - A estabilidade ano a ano (walk-forward com parâmetros fixos) é reportada.
- **Depois da etapa 1:** as sobreviventes vão para a etapa 2 no desenvolvimento, com a barra corrigida pelo total de variantes. Depois há uma única confirmação na validação (limite pessimista > 0 com z ≥ 1,645) e o simulador M5.
- **O que espero e por quê:** que a H1 morra na etapa 1. Os testes em larga escala da literatura não acharam vantagem no cruzamento puro depois dos custos, e no 15m em cripto o custo em R é alto.
- **Critério de morte:** nenhuma das 30 variantes passa na etapa 1, ou nenhuma sobrevivente se confirma na validação.
- **Grade aprovada pelo Alison em:** 2026-09-29 ("ok geral", resposta à proposta de 2026-09-28).

### E013 — H1 — resultado — 2026-09-29
- **Períodos usados:** só o desenvolvimento (cripto de 2020-01 a 2023-11; índices de 2015-01 a 2023-11, só pregão). Validação e cofre não foram tocados. Relatório em `resultados/2026-09-29_H1_v1.md`; trades em `resultados/2026-09-29_H1_v1_trades.parquet`; `scripts/lab_h1.py` (23 min).
- **Variantes rodadas:** 30 (etapa 1). Total acumulado: 30.
- **Trades, acerto e edge líquido em R:**
  - melhor de cripto, R2 12/26/9 4h: 1.174 trades, acerto de 37,6%, **+0,085R ± 0,052** (limite pessimista −0,066);
  - melhor de índices, R1 8/17/9 1h: 2.411 trades, acerto de 50,6%, +0,019R ± 0,018 (limite −0,034);
  - todas as 12 variantes de 15m são negativas: cripto de −0,11 a −0,13R, com custo de ~0,09R por trade; índices de −0,01 a −0,06R;
  - em cripto, o 1h é negativo em todas as regras.
- **Contra os baselines:** a melhor de cripto bate o sempre comprado (+0,018) e o aleatório (−0,040), e a melhor de índices bate os dois (−0,011 e −0,043). Mas nenhuma é estatisticamente significativa.
- **Simulador (M5), só informativo:** dentro da amostra e pegando a melhor de 30, o que infla o resultado (viés de seleção).
  - Cripto: P(passar) 67% contra 21% do aleatório; EV +$11 contra −$33.
  - Índices: P(passar) 64%, mas com mediana de 610 dias; EV −$33.
- **Robustez:** a R2 12/26/9 4h fica positiva nas 6 criptos (SOL +0,36, BNB +0,14; BTC e DOGE ≈ 0) e nos 4 anos (+0,07, +0,09, +0,14, +0,02). A melhor de índices é positiva nos 2 índices, mas instável ano a ano.
- **Veredito: MORRE.** Pelo critério pré-registrado, nenhuma variante passa em (b), o limite pessimista > 0 com z ≥ 2,94. Isso bate com a expectativa registrada antes de rodar.
- **Observação para as próximas hipóteses** (não é resgate da H1):
  - tendência lenta em cripto (cruzamento EMA12 × EMA26 no 4h) mostrou sinal positivo e consistente, porém fraco demais para a barra de 30 variantes. A H2 (Donchian) e a H3 (Supertrend) testam a mesma família com regras independentes, e a confirmação só vale na validação e no cofre;
  - o 15m não paga os custos com saída de alvo fixo.

### H2 — Rompimento de Donchian — pré-registro — 2026-09-29
- **Família e ideia:** tendência. Rompimento do canal de Donchian (sistemas clássicos dos Turtles), com saída pelo canal contrário para deixar o lucro correr. Proposta em `resultados/2026-09-29_H2_proposta_grade.md`.
- **Transparência:** duas lições da H1 (E013) entraram no desenho: sem 15m em cripto (custo em R previsível pela tabela e confirmado na H1) e saída de tendência no lugar do alvo fixo. A validação e o cofre são os juízes.
- **Mercados e tempo gráfico:** cripto (BTC, ETH, SOL, XRP, BNB, DOGE) em 1h e 4h; índices (Nasdaq-100 e S&P 500, desde 2015, só pregão da NYSE, 1h alinhado às 9h30, zerando no último minuto) em 15m e 1h.
- **Regra de entrada (codificável):** no fechamento do candle t, compra se o fechamento > máxima dos N candles anteriores (t−N a t−1); venda se o fechamento < mínima dos N anteriores. Execução no minuto seguinte. Só entra zerado. Sem sinal nos primeiros N candles.
- **Stop e saídas:**
  - Etapa 1: stop inicial de 2 ATR(20) e saída S5. O comprado sai quando o fechamento fica abaixo da mínima dos M candles anteriores; o vendido, quando fica acima da máxima dos M anteriores, com M = N/2 arredondado para baixo (10 e 27). Sem alvo e sem stop móvel.
  - Etapa 2, só para sobreviventes: S2 (alvo 2R), S3 (stop móvel a 3 ATR), S4 (sai depois de 12 candles) e S6 (empate depois de +1R), sempre com o stop inicial de 2 ATR(20).
- **Parâmetros livres e grade:** N ∈ {20, 55} × tempo gráfico (cripto: 1h e 4h; índices: 15m e 1h). Nenhum parâmetro otimizado.
- **Número de variantes:** etapa 1 = **8**; etapa 2 = 4 por sobrevivente.
- **Custos:** integrais, com estresse de 1,5×; filtro de custo máximo desligado (E012).
- **Baselines:** sempre comprado e aleatório (mesma frequência de sinais, 3 sementes), ambos com a mesma saída da variante: stop de 2 ATR(20) e o mesmo canal de saída de M candles.
- **Aprovação na etapa 1** (desenvolvimento; trades do grupo somados; tudo precisa valer):
  - (a) ≥ 300 trades;
  - (b) limite pessimista do R líquido médio > 0 com **z ≥ 2,50** (5% ÷ 8, unilateral), com erro-padrão agrupado por dia de NY;
  - (c) R médio > 0 com custos 1,5×;
  - (d) acima dos dois baselines;
  - (e) positivo em pelo menos 4 das 6 criptos, ou nos 2 índices.
  - A estabilidade ano a ano é reportada.
- **Depois da etapa 1:** as sobreviventes vão para a etapa 2 e, depois, para uma única confirmação na validação (limite > 0 com z ≥ 1,645), mais o simulador M5.
- **O que espero e por quê:** algum sinal positivo em cripto no 4h, porque 2020–2023 teve tendências fortes; é incerto se passa na barra. Pouco nos índices, porque zerar todo dia corta as tendências.
- **Critério de morte:** nenhuma das 8 passa na etapa 1, ou nenhuma sobrevivente se confirma na validação.
- **Grade aprovada pelo Alison em:** 2026-09-29 ("H2 OK").

### E014 — H2 — resultado da etapa 1 — 2026-09-29
- **Período:** só o desenvolvimento. Relatório em `resultados/2026-09-29_H2_v1.md`; trades em `..._trades.parquet`; `scripts/lab_h2.py` (6 min).
- **Variantes rodadas:** 8. Total acumulado: 38.
- **Sobreviventes: 2 de 8, ambas em cripto:**
  - **Donchian 20 4h:** 1.298 trades, acerto de 32,7%, **+0,373R ± 0,124** (limite +0,063); +0,347R com custos 1,5×; sempre comprado +0,127 e aleatório +0,157; positiva nas 6 criptos (de +0,19 a +0,64) e nos 4 anos (+0,64, +0,53, +0,11, +0,29).
  - **Donchian 55 4h:** 643 trades, acerto de 24,9%, **+0,555R ± 0,208** (limite +0,036); positiva nas 6 criptos.
- **Mortas:** Donchian 20/55 1h em cripto (limite < 0) e as 4 variantes de índices (todas negativas, 0 ou 1 de 2 mercados).
- **Diagnóstico de robustez** (informativo; não muda o critério pré-registrado):
  - **Donchian 20 4h:**
    - agrupando o erro por semana ou mês, o limite continua positivo (+0,034 e +0,035);
    - a vantagem sobre o aleatório com a mesma saída é de +0,217R, com t = 1,40, **não significativa**;
    - comprado **+0,712R** contra vendido **+0,001R**;
    - os 5% melhores trades somam 152% do resultado (o maior deu 73R).
  - **Donchian 55 4h:** agrupando por semana ou mês, o limite fica **negativo** (−0,138 e −0,160); vantagem sobre o aleatório com t = 0,99; comprado +1,143R contra vendido −0,134R.
  - **Leitura:** no desenvolvimento, o sistema se comportou como "comprar tendências de alta" num período de forte alta em cripto. A própria saída de tendência já rende com entradas aleatórias (+0,157R). O valor da entrada Donchian sobre o acaso ainda não está demonstrado.
- **Simulador (M5), informativo** (dentro da amostra, melhor de 8): Donchian 20 4h teve P(passar) 81% e EV +$493; o aleatório com a mesma saída, 66% e EV +$150. Isso confirma que o período favorece qualquer saída de tendência.

### E015 — H2 — esclarecimentos antes da etapa 2 (registrados antes de rodá-la) — 2026-09-29
Dois pontos do pré-registro estavam ambíguos. Ficam definidos assim, antes de ver qualquer resultado da etapa 2 ou da validação:
1. **As saídas da etapa 2 substituem o canal (S5), com o stop inicial de 2 ATR(20):**
   - S2: alvo de 2R;
   - S3: stop móvel a 3 ATR(20) do melhor preço, atualizado no fechamento de cada candle;
   - S4: sai depois de 12 candles (48 h);
   - S6 é um modificador: é o canal (S5) mais o empate (stop na entrada) depois de +1R.
   - São 8 combinações novas (2 sobreviventes × 4), e o total da H2 vai a 16 variantes. A barra das combinações novas é **z = 2,73** (5% ÷ 16). Os critérios (a) a (e) valem, com os baselines usando a mesma saída de cada combinação.
2. **Seleção para a validação:**
   - por entrada sobrevivente (N = 20 e N = 55), vai à validação só a combinação de maior limite pessimista entre as que passaram (a S5 original ou as novas da etapa 2), ou seja, **no máximo 2 testes**;
   - como são 2 testes, a barra da validação sobe de z = 1,645 para **z = 1,96** (5% ÷ 2), o que é mais rigoroso que o pré-registro;
   - a validação é rodada uma única vez, pelo mesmo script que faz a etapa 2, sem intervenção manual;
   - baselines, comprado × vendido e M5 são reportados.

### E016 — H2 — resultado da etapa 2 e da validação — 2026-09-29
- **Arquivos:** relatório em `resultados/2026-09-29_H2_v2_etapa2_validacao.md`; trades em `..._etapa2_trades.parquet` e `..._validacao_trades.parquet`; `scripts/lab_h2_etapa2.py`.
- **Etapa 2** (desenvolvimento, 8 combinações, z = 2,73): só **Donchian 20 4h S6** (canal + empate após +1R) passou: +0,259R ± 0,093, limite +0,005. As demais ficaram com limite negativo, entre −0,011 e −0,090, embora todas com R médio positivo e acima dos baselines. Total acumulado: 46 variantes.
- **Seleção pela regra do E015:** foram as duas S5 originais, com limites maiores que a S6. Foram à validação: Donchian 20 4h e Donchian 55 4h (z = 1,96).
- **Validação** (dez/2023 a abr/2025, rodada uma única vez):
  - **Donchian 20 4h:** 522 trades, acerto de 31,6%, **+0,170R ± 0,180** (limite −0,183), **não confirma**. Com custos 1,5×, +0,144R; sempre comprado +0,076 e aleatório −0,059; positiva nas 6 criptos. Comprado +0,371R contra vendido −0,064R. Os 5% melhores trades somam 235% do resultado. No M5 (informativo): P(passar) 63,5% contra 13,2% do aleatório; EV +$455 contra −$22.
  - **Donchian 55 4h:** 246 trades, **+0,457R ± 0,491** (limite −0,506), **não confirma**. Positiva em 4 de 6; comprado +1,214R contra vendido −0,419R.
- **Veredito: a H2 MORRE**, pelo critério de morte pré-registrado: nenhuma sobrevivente se confirma na validação. O cofre não foi tocado.
- **Leitura honesta:**
  - O Donchian 20 4h manteve na validação a direção do desenvolvimento: positivo nas 6 criptos, acima dos baselines e dentro de ~1,1 erro-padrão da estimativa de desenvolvimento.
  - A confirmação falhou por **falta de poder estatístico**: 17 meses de um sistema de caudas gordas dão um erro-padrão de ~0,18R, e só um edge acima de ~0,35R confirmaria.
  - "Não confirmado" não é o mesmo que "provado sem valor". Mesmo assim, a regra foi escrita antes e vale: nada de resgate.
  - Lição de protocolo (para decidir antes da próxima hipótese, e só para hipóteses novas): o critério atual de validação tem baixo poder para sistemas de tendência com poucos trades grandes.
- **Operacional:** a rodada levou 7 h em vez de ~15 min porque o Windows entrou em modo de espera às 02h47 e só voltou às 09h31 (eventos Kernel-Power 506/42). O cálculo não muda, mas na F4 o robô exige desligar a suspensão automática do PC.

### E017 — 2026-09-29 — Decisões do Alison: ordem das hipóteses e regra de validação
- **Nova ordem:** H4 (abertura de NY) → H5 (reversão) → H3 (Supertrend). A H3 vai para o fim por ser da mesma família de tendência da H2.
- **Regra de validação para hipóteses novas** (da H4 em diante; H1 e H2 continuam mortas pelas regras delas). Uma sobrevivente da etapa 1 ou 2 se confirma se, ao mesmo tempo:
  - (i) somando os trades do desenvolvimento e da validação, o limite pessimista > 0 com a **mesma barra de Bonferroni** da hipótese (erro agrupado por dia);
  - (ii) na validação sozinha, o R líquido médio ≥ 0;
  - (iii) no conjunto, o R médio fica acima dos dois baselines.
  - A validação continua rodando uma única vez, e o cofre continua sendo o teste final e independente (G1).
  - Motivo: com a regra antiga, a validação sozinha (17 meses) não tinha poder para sistemas de caudas gordas (E016).
- O G1 e a seção 5 do plano foram ajustados para essa regra.

### H4 — Abertura de NY (ORB) — pré-registro — 2026-09-29
- **Família e ideia:** horário. O movimento logo depois do sino de abertura de NY (9h30) tende a continuar durante o dia; é a observação do Alison no Nasdaq. Regras da literatura (Zarattini e Aziz; Zarattini, Barbon e Aziz), sem otimização. Proposta em `resultados/2026-09-29_H4_proposta_grade.md`.
- **Mercados e tempo gráfico:**
  - Índices: Nasdaq-100 e S&P 500 (proxy Dukascopy), desde 2015-01-01.
  - Cripto: BTC e ETH (Binance).
  - Os dois grupos operam só nos minutos do pregão regular da NYSE e só nos dias de pregão (calendário oficial), em candles de 1 minuto. O sinal no fechamento do minuto t executa no minuto t+1.
- **Regras de entrada (codificáveis):**
  - **R1, direção do 1º candle:** a direção do candle das 9h30–9h35 (fechamento das 9h34 contra a abertura das 9h30; alta = compra, baixa = venda, igual = sem trade). Entrada às 9h35, com stop no extremo oposto desse candle (mínima para compra, máxima para venda).
  - **R2(X), rompimento da faixa, X ∈ {5, 15, 30} minutos:** a faixa é a máxima e a mínima dos minutos das 9h30 até 9h30 + X. Vale o primeiro fechamento de 1 minuto acima da máxima (compra) ou abaixo da mínima (venda), a partir de 9h30 + X e antes das 12h00. O stop fica no lado oposto da faixa.
  - Um trade por dia.
- **Saída da etapa 1 ("base"):** alvo de 10R ou o último minuto do pregão (15h59; 12h59 nos fechamentos antecipados), o que vier primeiro.
- **Variantes:** 4 regras × 2 grupos = **8** na etapa 1; barra z = 2,50.
- **Baselines** (mesma saída e mesma lógica de stop):
  - **sempre comprado:** compra todo dia no mesmo horário da regra (R1: 9h35; R2: logo depois da faixa), com stop na mínima do candle ou da faixa;
  - **aleatório:** a cada dia, com a mesma frequência diária de trades da regra, direção sorteada. Na R1 às 9h35, com stop no extremo oposto do 1º candle; na R2 num minuto sorteado da janela de entrada, com stop à distância da largura da faixa. Três sementes.
- **Custos:** integrais, com estresse de 1,5×; filtro de custo máximo desligado (E012).
- **Etapa 1 (desenvolvimento; trades do grupo somados):**
  - (a) ≥ 300 trades;
  - (b) limite pessimista > 0 com z = 2,50 e erro agrupado por dia de NY;
  - (c) R médio > 0 com custos 1,5×;
  - (d) acima dos dois baselines;
  - (e) R médio > 0 nos 2 mercados do grupo.
- **Etapa 2** (só para sobreviventes, no desenvolvimento). Cada sobrevivente é testada com:
  - **S2:** alvo de 2R ou fim do pregão;
  - **S4:** sai depois de 60 minutos ou no fim do pregão;
  - **S6:** alvo de 10R, empate depois de +1R e fim do pregão.
  - Critérios (a) a (e), com z = Bonferroni sobre o total da H4 (8 + 3 × sobreviventes).
- **Seleção para a validação:** por sobrevivente (regra × grupo), a combinação de maior limite pessimista entre as que passaram (a base ou as da etapa 2).
- **Confirmação** (regra do E017, validação rodada uma única vez, pelo mesmo script e sem intervenção manual):
  - (i) somando desenvolvimento e validação, limite pessimista > 0 com o z do total da H4;
  - (ii) na validação sozinha, R médio ≥ 0;
  - (iii) no conjunto, R médio acima dos dois baselines.
  - M5 reportado.
- **O que espero e por quê:**
  - índices perto de zero, talvez levemente positivos na R1: a literatura mostra um ganho frágil que cabe no custo, e o nosso custo é baixo;
  - cripto negativo, porque a comissão de 0,03% contra stops curtos dá ~0,15 a 0,30R por trade.
- **Critério de morte:** nenhuma das 8 passa na etapa 1, ou nenhuma sobrevivente se confirma.
- **Grade aprovada pelo Alison em:** 2026-09-29 ("OK geral").

### E018 — H4 — resultado — 2026-09-29
- **Arquivos:** relatório em `resultados/2026-09-29_H4_v1.md`; trades em `..._etapa1_trades.parquet`; `scripts/lab_h4.py` (procedimento completo, com o PC mantido acordado).
- **Variantes rodadas:** 8. Total acumulado: 54. Nenhuma sobreviveu, então a etapa 2 e a validação não rodaram, e a validação não foi usada.
- **Índices** (2015–2023, ~4.300–4.500 trades por variante): todas entre −0,010 e −0,022R, com limite entre −0,08 e −0,13. A R1 (direção do 1º candle, regra do estudo do QQQ) deu −0,016R ± 0,046. O efeito da literatura não aparece no nosso proxy com custos. A R2(30) é negativa em 7 de 9 anos.
- **Cripto** (BTC e ETH, só no pregão de NY, 2020–2023):
  - **R2(30): +0,112R ± 0,053**, limite −0,022 (t ≈ 2,1 contra a barra de 2,50). Positiva no BTC (+0,120) e no ETH (+0,104) e nos 4 anos (+0,14, +0,05, +0,17, +0,09); custo médio de 0,079R; bem acima do aleatório (−0,068) e do sempre comprado (−0,234).
  - **R2(15):** +0,116R ± 0,065.
  - **R2(5):** +0,073R.
  - **R1:** −0,227R, com custo de ~0,28R, que mata os stops curtos.
- **M5, informativo** (melhor de 8, dentro da amostra): cripto R2(30) teve P(passar) 70% e EV +$141, contra 13% e −$32 do aleatório.
- **Veredito: a H4 MORRE** (nenhuma variante passa em (b)). A expectativa registrada se confirmou nos índices, mas não em cripto: esperei resultado negativo pelo custo, e o custo não matou a R2(30).
- **Padrão entre as hipóteses** (observação, não resgate):
  - os três sinais "promissores mas não significativos" foram todos em **cripto, na família de momento/tendência**: H1 R2 4h (t ≈ 1,6), H2 Donchian 20 4h (t ≈ 3,0 no desenvolvimento, sem confirmar na validação) e H4 ORB 30 min (t ≈ 2,1);
  - os índices não mostraram nada em nenhuma hipótese;
  - como o mesmo desenvolvimento já foi visto três vezes, uma hipótese nova que junte essas pistas teria de ser julgada com cuidado redobrado; o cofre seria a proteção real.

### E019 — 2026-09-29 — Auditoria do laboratório (pergunta do Alison: "o que mais pode ter passado batido?")
- **Origem:** o Alison perguntou se o MACD foi bem testado. A H1 testou 30 variantes (3 regras × 2 parâmetros × tempos gráficos × mercados), mas todas com a mesma saída (alvo fixo de 2R), que prejudica sinais de tendência. A H2 mostrou o tamanho do efeito: Donchian 20 4h deu +0,07R com alvo de 2R e +0,37R com saída de tendência. Eu deveria ter apontado isso antes de rodar. Decisão: a ordem fica H5 → H1b (MACD com saída de tendência, 4h em cripto, poucas variantes) → H3.
- **Diagnósticos medidos** (`resultados/2026-09-29_auditoria.md`, `scripts/auditoria_20260929.py`; só o desenvolvimento, sem seleção nem hipótese nova):
  1. **Funding:** conferido na documentação ("venue funding rates do not change your balance"). Só há swap, como o motor faz. ✔
  2. **Execução do stop sob estresse:**
     - H2 Donchian 20 4h (stop largo): +0,373 no nível, +0,350 no meio do caminho, +0,327 no pior preço do minuto. Robusta.
     - H1 R2 4h: +0,085 no nível, +0,062 no meio, +0,039 no extremo.
     - **H4 R2(30) cripto (stop curto): +0,112 no nível, +0,067 no meio, +0,022 no extremo. Frágil.** Em estratégias de stop curto, a qualidade da saída no stop decide.
  3. **Erro agrupado por dia, semana ou mês:** nenhuma decisão tomada muda. Para trades de vários dias, agrupar por dia é otimista (a H2 Donchian 55 4h passou só com o erro agrupado por dia).
  4. **Vantagem sobre o aleatório:** o critério (d) exigia só "acima", sem significância. Com erro agrupado por mês: H1 t = 1,93; H2 t = 1,40; H4 R2(30) cripto t = 3,26, a única entrada que bate o acaso com folga.
- **Mudanças já feitas** (não mudam critério nenhum; 91 testes):
  - motor com `execucao_stop` ("nivel", "meio", "extremo");
  - o laboratório roda toda variante também com o stop pessimista ("meio") e reporta sempre: o limite com erro por semana, a vantagem sobre o aleatório com o t, e comprado × vendido;
  - na validação, os indicadores são aquecidos com os dias anteriores ao período, e os trades antes do início do período são descartados.
- **Proposta ao Alison** (critérios "v2", só para hipóteses novas, a partir da H5; mais rigorosos):
  - (b) erro agrupado por **semana**;
  - (c) positivo com custos 1,5× **e** com o stop saindo no meio do caminho;
  - (d) acima dos baselines **e** com vantagem sobre o aleatório significativa (t ≥ 1,645, erro por mês).
- **Compromisso de processo:**
  - toda proposta de hipótese passa a ter a seção "O que pode distorcer este teste (a favor ou contra)";
  - todo relatório passa a trazer os diagnósticos acima automaticamente.
- **Pendências conhecidas, ainda não resolvidas:**
  - o M5 processa em sequência posições simultâneas em mercados correlacionados; um simulador de carteira com o teto por grupo é obrigatório antes de qualquer decisão de G1;
  - a comparação do proxy dos índices (Dukascopy) com o venue real (Hyperliquid) aguarda dados de forward;
  - os tetos de exposição e o orçamento horário de abertura só podem ser vistos com a conta (F4);
  - viés de sobrevivência das criptos escolhidas (compensado em parte pelo comprado × vendido no relatório).

### E020 — 2026-09-29 — Decisões do Alison: critérios v2 e grade da H5
- **Critérios "v2"** aprovados para todas as hipóteses novas, da H5 em diante:
  - (b) erro agrupado por semana;
  - (c) R > 0 com custos 1,5× e com o stop saindo no meio do caminho;
  - (d) acima dos dois baselines e com vantagem sobre o aleatório de t ≥ 1,645 (erro por mês).
- Na confirmação (regra do E017), o limite do conjunto desenvolvimento + validação também usa o erro agrupado por semana.
- **Grade da H5** aprovada (`resultados/2026-09-29_H5_proposta_grade.md`).

### H5 — Reversão à média em mercado lateral — pré-registro — 2026-09-29
- **Família e ideia:** reversão. Depois de um exagero de curto prazo, com o mercado lateral, o preço volta à média. Regras da literatura, sem otimização. A seção de distorções está na proposta.
- **Mercados e tempo gráfico:**
  - Cripto: BTC e ETH, 24h, em 1h e 4h.
  - Índices: Nasdaq-100 e S&P 500, desde 2015, só pregão da NYSE, zerando no fim, em 5m e 15m. Os indicadores seguem de um dia para o outro sobre os candles do pregão.
- **Regras de entrada** (sinal no fechamento do candle t, execução no minuto seguinte; só entra zerado):
  - **M1 (Connors):** compra com RSI(2) (Wilder) < 10; vende com RSI(2) > 90.
  - **M2 (Bollinger 20, 2):** compra com o fechamento abaixo da banda inferior; vende com o fechamento acima da superior.
  - **Filtro nas duas:** só entra com ADX(14) (Wilder) < 20.
  - Aquecimento: sem sinal nos primeiros 100 candles.
- **Saída da etapa 1 ("base"):**
  - stop de 2 ATR(14);
  - saída na média: M1 sai quando o fechamento cruza a média simples de 5 (comprado: fechamento > média; vendido: fechamento < média); M2 faz o mesmo com a média de 20;
  - nos índices, também no fim do pregão.
- **Variantes:** 2 regras × 2 tempos gráficos × 2 grupos = **8**.
- **Baselines:** sempre comprado e aleatório (mesma frequência de sinais, 3 sementes), com a mesma saída (stop de 2 ATR e a mesma média de saída).
- **Etapa 1** (desenvolvimento; critérios v2; z = 2,50; erro por semana):
  - (a) ≥ 300 trades;
  - (b) limite > 0;
  - (c) R > 0 com custos 1,5× e com o stop no meio do caminho;
  - (d) acima dos baselines e vantagem sobre o aleatório com t ≥ 1,645;
  - (e) positivo nos 2 mercados do grupo.
- **Etapa 2** (só sobreviventes; as saídas substituem a saída na média, com o mesmo stop de 2 ATR(14)):
  - S2: alvo de 1R;
  - S4: sai depois de 12 candles;
  - S6 (modificador): saída na média mais empate depois de +1R.
  - Critérios v2, com z pelo total da H5 (8 + 3 × sobreviventes).
- **Seleção:** por sobrevivente, a combinação de maior limite (erro por semana) entre as que passaram.
- **Confirmação** (E017, validação uma única vez, pelo mesmo script, com 30 dias de aquecimento dos indicadores):
  - (i) limite do conjunto desenvolvimento + validação > 0, com o z do total e o erro por semana;
  - (ii) R médio da validação ≥ 0;
  - (iii) o conjunto acima dos dois baselines.
  - M5 reportado.
- **O que espero e por quê:** cripto negativo, porque as hipóteses anteriores indicam momento em cripto, não reversão; índices incertos, talvez levemente positivos (reversão intradiária e custo baixo), mas sensíveis à execução do stop no 5m.
- **Critério de morte:** nenhuma das 8 passa na etapa 1, ou nenhuma sobrevivente se confirma.
- **Grade aprovada pelo Alison em:** 2026-09-29 ("Pode seguir com as suas recomendações").

### E021 — H5 — resultado — 2026-09-29
- **Arquivos:** relatório em `resultados/2026-09-29_H5_v1.md`; trades em `..._etapa1_trades.parquet`; `scripts/lab_h5.py`, que usa `laboratorio.procedimento_completo`, com critérios v2.
- **Variantes rodadas:** 8. Total acumulado: 62. Nenhuma sobreviveu, então a validação não foi usada.
- **Cripto** (BTC e ETH): todas negativas.
  - M1 1h −0,078R; M1 4h −0,103R; M2 1h −0,146R; M2 4h −0,038R.
  - Acerto de 51–60%, mas as perdas são maiores que os ganhos.
  - Negativas nos 2 mercados e em todos os anos.
  - **Piores que o aleatório** com a mesma saída em M1 4h (t = −2,82) e M2 1h (t = −2,53).
- **Índices:** todas negativas, de −0,068 a −0,088R; 0 de 2 mercados; negativas em todos os 9 anos. M1 15m pior que o aleatório (t = −2,79).
- **Veredito: a H5 MORRE.** Bate com a expectativa em cripto; nos índices o resultado foi pior que o "incerto" esperado.
- **Leitura:** apostar contra o movimento de curto prazo perde, e em cripto perde mais que o acaso. É o espelho do padrão de momento/tendência em cripto (E018).
- **Correção de apresentação:** o cabeçalho do relatório padrão dizia "erro agrupado por dia", e a coluna do limite mostrava o limite por dia, embora o critério v2 decida pelo limite por semana. O relatório agora mostra a coluna que decide. O veredito não muda, e a regeneração releu os trades gravados, sem rodar de novo.

### H1b — MACD com saída de tendência — pré-registro — 2026-09-29
- **Ideia:** a H1 só testou o MACD com alvo fixo de 2R, que corta as tendências (pergunta do Alison, E019). Aqui o MACD roda como sistema de tendência clássico. Proposta em `resultados/2026-09-29_H1b_H3_proposta_grade.md`.
- **Transparência:** o desenho foi feito depois dos resultados da H1, H2, H4 e H5 no mesmo desenvolvimento (caminhos que se bifurcam); o cofre é a proteção real.
- **Mercados e tempo gráfico:** 6 criptos (BTC, ETH, SOL, XRP, BNB, DOGE), 4h, desenvolvimento de 2020 a nov/2023.
- **Entradas:** MACD 12/26/9, com as mesmas definições e o mesmo aquecimento da H1.
  - **R1:** cruzamento da linha de sinal.
  - **R2:** cruzamento do zero.
  - Sinal no fechamento, execução no minuto seguinte.
- **Saídas:**
  - **S5:** sai no sinal contrário da mesma regra e entra do outro lado (o motor só entra zerado, então a inversão acontece no mesmo minuto), com stop inicial de 2 ATR(14);
  - **S3:** stop inicial de 2 ATR(14) e stop móvel a 3 ATR(14) do melhor preço, atualizado no fechamento de cada candle; sinais contrários são ignorados enquanto a posição está aberta.
- **Variantes: 4** (2 regras × 2 saídas). Barra z = 2,24. **Sem etapa 2.**
- **Baselines:** sempre comprado e aleatório (mesma frequência de sinais, 3 sementes), com a mesma saída de cada variante.
- **Critérios v2 (E020):**
  - (a) ≥ 300 trades;
  - (b) limite > 0 com erro por semana;
  - (c) R > 0 com custos 1,5× e com o stop no meio do caminho;
  - (d) acima dos baselines e vantagem sobre o aleatório com t ≥ 1,645;
  - (e) R > 0 em pelo menos 4 das 6 criptos.
- **Seleção e confirmação:** pela regra do E017, com validação única e 30 dias de aquecimento.
- **O que espero:** R médio positivo no desenvolvimento, mas é incerto se passa no (d), porque a vantagem pode vir só da saída num período de alta.
- **Critério de morte:** nenhuma das 4 passa, ou a sobrevivente não se confirma.
- **Grade aprovada pelo Alison em:** 2026-09-29.

### H3 — Supertrend — pré-registro — 2026-09-29
- **Ideia:** a linha do Supertrend acompanha o preço a m × ATR(n) e vira quando o fechamento a cruza; ela funciona como stop móvel. Parâmetros clássicos, sem otimização.
- **Mercados e tempo gráfico:** 6 criptos em 1h e 4h. Índices fora (recomendação aprovada: tendência intradiária nos índices não mostrou nada na H1, H2 e H4).
- **Regra:** Supertrend padrão.
  - Banda inferior = (máxima + mínima)/2 − m × ATR(n); banda superior = (máxima + mínima)/2 + m × ATR(n).
  - As bandas só apertam, a menos que o fechamento anterior as rompa.
  - A tendência vira para alta quando o fechamento passa da banda superior anterior e para baixa quando fica abaixo da banda inferior anterior.
  - A virada para alta compra, com stop inicial na banda inferior; a virada para baixa vende, com stop inicial na banda superior.
  - Sai e inverte na virada contrária.
  - Sem sinal nos primeiros 50 candles.
- **Parâmetros:** n = 10; m ∈ {3, 2}.
- **Variantes: 4** (m × tempo gráfico). z = 2,24. Sem etapa 2, porque a saída faz parte da regra.
- **Baselines:** mesma saída; o stop inicial é m × ATR(10) a partir da entrada, porque os baselines não têm a linha.
- **Critérios, seleção e confirmação:** iguais aos da H1b; em (e), pelo menos 4 das 6 criptos.
- **O que espero:** positivo no 4h no desenvolvimento, com o (d) incerto, como na H1b.
- **Critério de morte:** nenhuma das 4 passa, ou a sobrevivente não se confirma.
- **Grade aprovada pelo Alison em:** 2026-09-29.

### E022 — H1b e H3 — resultados — 2026-09-29
- **Arquivos:** relatórios em `resultados/2026-09-29_H1b_v1.md` e `resultados/2026-09-29_H3_v1.md`; `scripts/lab_h1b.py` e `scripts/lab_h3.py`. Variantes: 4 + 4; total acumulado de 70. Nenhuma sobreviveu, então a validação não foi usada.
- **H1b** (MACD 4h com saída de tendência, 6 criptos):
  - **R2 S5** (sai e inverte no cruzamento do zero): +0,313R ± 0,121, limite por semana +0,023; passa em (a), (b), (c) e (e). **Falha em (d):** vantagem sobre o aleatório com a mesma saída de +0,119R, t = 0,80 (o aleatório deu +0,195R). Comprado +0,641R, vendido −0,015R.
  - As outras três (R1 S5 +0,047; R1 S3 +0,075; R2 S3 +0,062) têm limite negativo e t de 0,45 a 0,95.
  - **Veredito: MORRE.**
- **H3** (Supertrend, 6 criptos):
  - **10/2 4h:** +0,180R, limite por semana +0,009, aleatório +0,195 (t = −0,13).
  - **10/3 4h:** +0,223R, aleatório +0,269 (t = −0,31).
  - **10/3 1h** e **10/2 1h:** +0,072 e +0,025, com t de 0,72 e 1,14.
  - Comprado de +0,09 a +0,52R; vendido ≈ −0,05R.
  - **Veredito: MORRE**, pois nenhuma entrada bate o acaso.
- **Defeito encontrado no baseline "sempre comprado":** com saída "no sinal contrário", ele nunca recebe sinal contrário. Vira compra-e-segura até o stop inicial e dá poucos trades gigantes (+24 a +34R por trade em 2020–2021), o que torna a comparação por trade sem sentido. **Não mudou nenhum veredito**, porque todas as variantes falham também contra o aleatório. Para hipóteses futuras, proponho redefinir o "sempre comprado" como "compra nos mesmos momentos da estratégia", o que isola a informação da direção; isso depende de aprovação.
- **Conclusão da família de tendência em cripto** (H1, H1b, H2, H3): no desenvolvimento (2020–2023), **entrar ao acaso e sair com saída de tendência rendeu tanto quanto** MACD, Donchian ou Supertrend. O ganho vem da saída (cortar perdas e deixar os ganhos correrem) e da alta do período: os comprados ganham e os vendidos ficam ≈ 0. **Nenhuma entrada de tendência mostrou informação própria.**
- **Única entrada com informação sobre o acaso:** H4 R2(30) em cripto, t = 3,26 contra o aleatório (E019). Ela falhou na barra absoluta (t ≈ 2,1 < 2,50) e é sensível à execução do stop (+0,112 no nível, +0,067 no meio do caminho, +0,022 no pior preço do minuto).
- **Atenção à contaminação** (lembrete do E008): eu vi um único preço do BTC do período do cofre (ago/2026). Não uso isso para desenhar nada, mas registro o risco de viés no meu julgamento sobre sistemas que dependem de alta.
- **A fila de hipóteses do plano (H1–H5 e a H1b) acabou sem nenhuma aprovada.** Pela seção 9 do plano, sem G1 o projeto pausa sem gastar. As próximas decisões são do Alison.

### E023 — 2026-09-29 — Decisões do Alison depois do balanço
- **Teste final (H4b):** o ORB de 30 min em cripto, uma variante, testado só na validação. Se falhar, o projeto pausa (seção 9).
- **Compra da conta:** não comprar agora. Nenhuma estratégia passou no G1, e o plano prevê a compra só depois dele.
- **Baseline "sempre comprado" redefinido** para hipóteses futuras: "compra nos mesmos momentos da estratégia, com a mesma saída". Isso isola a informação da direção e evita o compra-e-segura degenerado do E022. Onde a lógica de stop da estratégia não permitir essa definição (como no ORB), vale o baseline próprio pré-registrado.

### H4b — ORB de 30 min em cripto, teste fora da amostra — pré-registro — 2026-09-29
- **Origem e transparência:** a hipótese nasceu da H4 (E018) e da auditoria (E019). A R2(30) em cripto foi a melhor de 8 variantes no desenvolvimento (+0,112R, t ≈ 2,1) e a única entrada que bateu o acaso com folga (t = 3,26). Como foi escolhida olhando o desenvolvimento, **o desenvolvimento não conta como evidência**: o teste é só na validação, que o ORB nunca viu. A validação passa a ter sido usada 2 vezes (H2 e H4b).
- **Regra (idêntica à R2(30) da H4):**
  - BTC e ETH, só no pregão regular da NYSE e só nos dias de pregão, em candles de 1 minuto;
  - a faixa é a máxima e a mínima das 9h30 às 10h00;
  - entrada no primeiro fechamento de 1 minuto além da faixa, antes das 12h00, um trade por dia;
  - stop no lado oposto da faixa; alvo de 10R ou o último minuto do pregão.
- **Baselines:** os mesmos da H4. O sempre comprado compra às 10h00 com stop na mínima da faixa; o aleatório tem a mesma frequência diária, direção sorteada, minuto sorteado da janela e stop à largura da faixa (3 sementes).
- **Período:** só a validação (2023-12-01 a 2025-04-30). Rodada **uma única vez**, protegida contra reexecução.
- **Critérios v2 (E020), sobre a validação sozinha, com 1 variante (z = 1,645):**
  - (a) ≥ 300 trades;
  - (b) limite pessimista > 0, com erro agrupado por semana;
  - (c) R > 0 com custos 1,5× e com o stop saindo no meio do caminho;
  - (d) acima dos dois baselines e vantagem sobre o aleatório com t ≥ 1,645;
  - (e) R > 0 no BTC e no ETH.
  - M5 reportado.
- **Se passar:** vai ao cofre só com o ok explícito do Alison, uma única vez, e depois ao G1, que exige o simulador de carteira.
- **Se falhar:** a H4b morre e o projeto pausa sem gasto.
- **O que espero:** é incerto. O efeito no desenvolvimento foi pequeno e sensível à execução do stop. Com ~650–700 trades na validação, só um R médio acima de ~0,09 passaria em (b).
- **Aprovado pelo Alison em:** 2026-09-29 ("Pode seguir com as suas recomendações").

### E024 — H4b — resultado — e pausa do projeto — 2026-09-29
- **Arquivos:** relatório em `resultados/2026-09-29_H4b_v1.md`; trades em `..._validacao_trades.parquet`; `scripts/lab_h4b.py`. Rodado uma única vez.
- **Validação** (dez/2023 a abr/2025, BTC e ETH, pregão de NY):
  - 663 trades, acerto de 43,3%, **+0,047R ± 0,070**, limite por semana −0,059;
  - +0,010 com custos 1,5× e +0,016 com o stop pessimista;
  - sempre comprado −0,273 e aleatório +0,018;
  - **vantagem sobre o aleatório de +0,029R, t = 0,40**;
  - positiva no BTC (+0,026) e no ETH (+0,068); comprado +0,012, vendido +0,079;
  - trimestres instáveis, de −0,257 a +0,526.
- **Critérios:** (a) S, (b) n, (c) S, (d) n, (e) S. **Não passou.**
- **Veredito: a H4b MORRE.** A informação da entrada sobre o acaso (t = 3,26 no desenvolvimento) não se repetiu fora da amostra (t = 0,40): foi efeito da seleção da melhor de 8, e a validação cumpriu o papel dela.
- **M5, informativo:** estratégia com P(passar) de 53,6% e EV de +$46; o **aleatório com a mesma saída deu 61,4% e EV de +$67**. Isso confirma na prática o que a F2 apontou: com certas saídas (alvo distante, acerto baixo), a assimetria da mesa pode dar EV positivo **sem vantagem nenhuma**. Por isso o G1 exige EV acima do aleatório, e não só EV > 0.
- **PROJETO EM PAUSA** (seção 9 do plano; decisão do E023). Resumo da F3:
  - 7 hipóteses e 71 variantes, nenhuma aprovada;
  - cofre intacto (0 aberturas); validação usada 2 vezes; gasto de $0.
- **O que fica pronto para uma retomada:** dados (F1), motor, risco e simulador (F2), laboratório com procedimento único auditado (`laboratorio.procedimento_completo`), critérios v2 e 97 testes.
- **Pendências antes de qualquer G1 futuro:**
  - simulador de carteira com o teto de risco por grupo;
  - comparação do proxy dos índices com o venue real;
  - tetos de exposição e orçamento horário da conta (F4).

### E025 — 2026-09-29 — Retomada da F3 com a estratégia de um vídeo (H6)
- **Decisão do Alison:** retomar ("vamos dar continuidade"), testando a estratégia de um vídeo que ele indicou: "Highly Profitable DEMA + SuperTrend Trading Strategy", do canal TradingLab (https://www.youtube.com/watch?v=g-PLctW8aU0), com "+130,14%" em 100 trades.
- **Fonte das regras:** a legenda do próprio vídeo, lida em 29/09/2026 no navegador do app, sem baixar arquivo. Um resumo de terceiros achado antes estava errado em dois pontos, e eu o repassei ao Alison antes de conferir:
  - o tempo gráfico é 15 minutos, não 1 hora;
  - não existe a regra "só sai no lucro ou no empate".
  - **Lição:** regra de estratégia vem da fonte primária antes de qualquer proposta.
- **Código, sem rodar nada em dado real:**
  - `src/estrategias/h6_dema_supertrend.py`, que reaproveita o Supertrend da H3;
  - `scripts/lab_h6.py`;
  - 5 testes novos com dados sintéticos, **102 no total**, todos passando.
  - O motor não precisou mudar: a saída na virada usa `sair_long`/`sair_short`.
- **Commit** das mudanças da sessão anterior: `9b86b3e`.
- **Contexto do pedido do Alison ("será que só a gente está cego?"):** registrei a evidência apresentada. Chague, De-Losso e Giovannetti (FGV/USP, SSRN 3423101) mostram que, entre os day traders de mini-índice que persistiram mais de 300 pregões, 97% perderam dinheiro e só 1,1% ganharam mais que um salário mínimo.

### H6 — DEMA 200 + Supertrend (vídeo do TradingLab) — pré-registro — 2026-09-29
- **Família e ideia:** tendência. Regras de um vídeo público, fiéis à legenda, sem ajuste aos nossos dados. Proposta completa, com a seção de distorções, em `resultados/2026-09-29_H6_proposta_grade.md`.
- **Mercados e tempo gráfico:** 6 criptos (BTC, ETH, SOL, XRP, BNB, DOGE), 24h, em 15m (o tempo gráfico dos exemplos do vídeo).
- **Regras** (sinal no fechamento do candle t, execução no minuto seguinte):
  - DEMA(200) no fechamento = 2 × EMA − EMA da EMA; Supertrend com ATR(12) de Wilder e multiplicador 3, igual ao da H3.
  - **V1 (original):** compra quando o Supertrend vira para alta e o fechamento está acima da DEMA; vende no espelho.
  - **V2 (1ª dica do vídeo):** compra no primeiro candle em que "Supertrend em alta e fechamento acima da DEMA" passa a valer, em qualquer ordem; vende no espelho.
  - **Stop:** a linha do Supertrend no candle do sinal, fixa.
  - **Saída:** na virada contrária do Supertrend (com ou sem DEMA). Se a virada cumprir a entrada do outro lado, inverte no mesmo minuto.
  - Sem sinal nos primeiros 1.000 candles.
  - A 2ª dica (Bandas de Bollinger de Fibonacci) fica de fora: o autor não a testou.
- **Variantes: 2.** z = 1,96. Sem etapa 2.
- **Baselines** (mesma saída):
  - **aleatório:** mesma frequência de sinais, 3 sementes, stop de 3 × ATR(12);
  - **sempre comprado (E023):** compra nos momentos dos sinais da estratégia; nos de compra, com a mesma linha como stop; nos de venda, com stop de 3 × ATR(12).
- **Critérios v2 (E020):**
  - (a) ≥ 300 trades;
  - (b) limite > 0 com erro por semana;
  - (c) R > 0 com custos 1,5× e com o stop no meio do caminho;
  - (d) acima dos baselines e vantagem sobre o aleatório com t ≥ 1,645;
  - (e) R > 0 em pelo menos 4 das 6 criptos.
- **Confirmação:** regra do E017; validação uma única vez, com 30 dias de aquecimento. M5 reportado.
- **O que espero e por quê:** que morra. A H3 mostrou que as entradas do Supertrend em cripto equivalem ao acaso. No 15m há mais ruído, e o custo pesa mais em R (estimativa pelo config: de 0,05 a 0,10R por trade).
- **Critério de morte:** nenhuma das 2 passa na etapa 1, ou a sobrevivente não se confirma.
- **Grade aprovada pelo Alison em:** 2026-09-29, com a inclusão do Nasdaq ("OK, tudo bem. mas tente incluir nasdaq nos testes também"). A grade final, que vale no lugar desta onde houver diferença, está no E026.

### E026 — 2026-09-29 — H6: grade final com o Nasdaq e correção no laboratório (antes de rodar)
- **Grade final da H6**, que prevalece sobre o pré-registro acima onde os dois diferem:
  - **Grupo novo "nasdaq":** Nasdaq-100 (proxy da Dukascopy, `USATECHIDXUSD`) desde 2015, em 15m. Só no pregão regular da NYSE, zerando no fim do pregão (decisão do Alison na v0.7 do plano). Os indicadores seguem de um dia para o outro sobre os candles do pregão, como na H5.
  - Mesmas regras (V1 e V2), mesmos baselines e mesmos critérios; em (e), R > 0 no Nasdaq.
  - **Variantes: 4** (V1 e V2 × cripto e Nasdaq). A barra sobe de z = 1,96 para **z = 2,24** (Bonferroni pelo total da hipótese), **também para as criptos**. Separar o Nasdaq numa hipótese à parte para manter a barra das criptos seria contornar a correção.
  - **Aquecimento da validação: 90 dias** (antes 30). Com 26 candles de 15m por pregão, 30 dias não cobririam os 1.000 candles de aquecimento no Nasdaq, e as primeiras semanas da validação ficariam sem sinal. Nas criptos, mudar não tem efeito além de carregar mais dados antes: os trades anteriores ao início da validação continuam descartados.
- **Defeito encontrado e corrigido, antes de qualquer uso:**
  - Em `laboratorio.rodar_grupo`, o "inicio" de um grupo (índices: 2015) valia até na validação.
  - Uma hipótese de índices que chegasse à validação carregaria também os dados do desenvolvimento e os contaria como validação.
  - **Nunca aconteceu:** nenhuma hipótese de índices chegou à validação; H4 e H5 morreram na etapa 1, e a H4b foi só cripto.
  - **Correção:** o início do grupo nunca puxa o período para trás.
  - Teste novo: **103 testes**, todos passando.

### E027 — H6 — resultado — 2026-09-29
- **Arquivos:** relatório em `resultados/2026-09-29_H6_v1.md`, log em `..._H6_v1_log.txt`, trades em `..._etapa1_trades.parquet` e `..._validacao_trades.parquet`.
- **Variantes rodadas:** 4. Total acumulado: 75. Validação usada pela 3ª vez, só no Nasdaq V1, com 90 dias de aquecimento carregados a partir de 2023-09-05 (a correção do E026 funcionou).
- **Cripto** (6 moedas, 15m, 24h, a regra do vídeo sem adaptação): as duas **morrem**.
  - V1: −0,026R (11.509 trades); V2: −0,019R (15.504).
  - Positivas em só 2 das 6 moedas; vendidos negativos.
  - Vantagem sobre o aleatório com t = 1,19 e 1,46.
  - Custo médio de 0,05R por trade. Bate com a expectativa.
- **Nasdaq, desenvolvimento 2015–2023:**
  - **V1: passou nos 5 critérios.** 958 trades, acerto de 51%, **+0,057R ± 0,024**, limite por semana **+0,006** (margem mínima).
  - Com custos 1,5×: +0,046; com o stop no meio do caminho: +0,055.
  - Sempre comprado −0,035; aleatório −0,018; vantagem com **t = 2,85**.
  - Comprado +0,053 e vendido +0,060.
  - Positivo em 7 de 9 anos (2016 −0,047; 2019 −0,023). Sem os 10 maiores trades: +0,027.
  - V2: +0,040R, limite −0,008. Falha só em (b).
- **Confirmação** (regra do E017, z = 2,24):
  - (i) limite de desenvolvimento + validação de +0,006 (1.103 trades, +0,054R);
  - (ii) validação sozinha +0,034R (145 trades);
  - (iii) acima dos baselines.
  - **Confirmada pela regra pré-registrada.**
- **Diagnósticos feitos depois**, informativos e não pré-registrados (procurar artefato antes de concluir):
  - **Na validação sozinha, não há vantagem sobre o acaso:**
    - estratégia +0,034 ± 0,059; aleatório +0,029 (365 trades); **vantagem +0,005, t = 0,07**;
    - sempre comprado −0,027;
    - trimestres de −0,205 a +0,285.
    - A confirmação passou porque o desenvolvimento (958 trades) domina o conjunto. **É o mesmo padrão da H4b:** a vantagem sobre o acaso do desenvolvimento não se repetiu fora da amostra.
  - **Mecanismo:**
    - 78% dos trades saem no fim do pregão, com +0,29R em média; as saídas pela virada dão −0,57R e os stops −1,02R.
    - Na prática, é "entrar na virada a favor da DEMA e segurar até o fechamento": parece momento intradiário.
    - **Zerar no fim do pregão é uma regra nossa, não do vídeo.** Na regra original (cripto, 24h, sem zerar), perdeu.
  - **Horários da Dukascopy conferidos:** o salto de volatilidade cai no minuto das 9h30 de NY (6,2 bps contra 2,0 às 9h29). Não há deslocamento de horário.
  - **M5 com dados de desenvolvimento + validação** (colchão de $300; 3.000 contas):
    - **EV negativo em todo K**: K = 2: −$32; K = 3: −$34; K = 4: −$36; K = 6: −$38; K = 12: −$40.
    - É sempre um pouco melhor que o aleatório (−$35 a −$40), mas não paga a conta.
    - No K = 12: P(passar) de 86%, mas com **mediana de ~1.090 dias**. Com cerca de 2 trades por semana a +0,05R, a vantagem é pequena e lenta demais para a mesa.
    - **Com custos 1,5×, o EV também fica negativo.**
  - **Correção múltipla no projeto inteiro:** pelo total de 75 variantes testadas no mesmo desenvolvimento, z ≈ 3,2. O limite ficaria em ≈ −0,02, ou seja, não passaria.
- **Veredito:**
  - A H6 cripto (a regra do vídeo) **morre**.
  - A H6 Nasdaq V1 é **confirmada pela regra do E017, mas não cumpre o G1**, porque o EV é negativo em todo K (o G1 exige EV positivo e acima do aleatório com custos 1,5×).
  - Pelo protocolo, a próxima etapa seria o cofre, e isso é decisão do Alison.
- **Recomendação: não abrir o cofre.**
  - Mesmo que confirme, o G1 falha pelo EV, então o resultado não muda nenhuma decisão.
  - O cofre de índices (mai/2025 a ago/2026) teria só ~140 trades. Com erro de ~0,06R, ele não distinguiria uma vantagem de +0,05R de zero.
  - Abrir o cofre gasta a única amostra limpa de índices que resta.
- **Decisão do Alison:** pendente.

### E028 — 2026-09-29 — Novas fontes indicadas pelo Alison: TradeX Labs e TradingLab (falsos rompimentos)
- **Pedido:** "Vamos buscar testar mais estratégias", com 3 abas do navegador. Material lido pelas legendas e pelas telas, sem baixar arquivo. Análise completa em `resultados/2026-09-29_H7_proposta_grade.md`.
- **TradeX Labs** (shorts sobre ORB automatizado no NQ para mesas proprietárias):
  - as regras das variações vencedoras não são públicas; o acesso é vendido;
  - a regra pública, o ORB clássico de 15 min com entrada no fechamento de 1 min fora da faixa, é a **H4 R2(15)**, já testada nos índices e morta (E018);
  - a tela do backtest deles mostra 80% de acerto, fator de lucro de 1,30, **comissão de 0%** e parâmetros otimizados no próprio histórico, sem teste fora da amostra.
  - **Nada novo para testar.**
- **TradingLab, "How to Avoid False Breakouts":** regras codificáveis, sem backtest no vídeo. Proposta como **H7**.
- **Resposta à pergunta do Alison, "essa é a que chegou mais perto?":** sim, a H6 Nasdaq V1 foi a única a passar no desenvolvimento e na confirmação. Mas ela empatou com o acaso na validação sozinha e tem EV negativo no M5, então continua longe do G1.

### H7 — Rompimento de consolidação com candle de momento (vídeo do TradingLab) — pré-registro — 2026-09-29
- **Família e ideia:** rompimento. Só entra quando o rompimento vem com "momento", para evitar os falsos rompimentos. Regras do vídeo codificadas com escolhas fixadas antes de ver dados. A proposta tem a seção de distorções.
- **Mercados e tempo gráfico** (o vídeo não diz):
  - 6 criptos em 1h, 24h, de 2020 a 2023;
  - Nasdaq-100 (proxy da Dukascopy) em 15m, só no pregão de NY e zerando no fim, de 2015 a 2023; os indicadores seguem de um dia para o outro sobre os candles do pregão.
- **Regras** (sinal no fechamento do candle, execução no minuto seguinte; só entra zerado):
  - **Consolidação:** nos 20 candles antes do rompimento, a resistência é a maior máxima e o suporte a menor mínima. Exige pelo menos 2 "visitas" separadas a cada linha (máxima ou mínima a menos de 0,2 × ATR(14) da linha).
  - **Candle de momento A:** fecha além da linha, com corpo ≥ 1 × ATR(14) e mais da metade do corpo além da linha.
  - **Candle de momento B:** o primeiro candle que fecha além da linha e os 2 seguintes são todos da cor do rompimento e fecham além da linha; entra no 3º.
  - A venda é o espelho (rompimento do suporte).
  - **Stop:** a linha rompida ∓ 0,1 × ATR(14).
  - **Saída:**
    - metade no alvo de +1,5R;
    - depois disso, o stop da outra metade vai para +1,5R, e ela sai quando o Chandelier Exit (22 candles, 2 × ATR(22), pelo fechamento) vira contra a posição;
    - antes do alvo, só o stop;
    - no Nasdaq, também no fim do pregão.
- **Variantes: 2** (cripto 1h; Nasdaq 15m). z = 1,96. Sem etapa 2.
- **Baselines** (mesma saída):
  - aleatório: mesma frequência de sinais, 3 sementes, stop de 1 × ATR(14);
  - sempre comprado (E023) nos mesmos momentos: stop da linha nos rompimentos de alta, 1 × ATR(14) nos de baixa.
- **Critérios v2 (E020):**
  - (a) ≥ 300 trades;
  - (b) limite > 0 com erro por semana;
  - (c) R > 0 com custos 1,5× e com o stop no meio do caminho;
  - (d) acima dos baselines e vantagem sobre o aleatório com t ≥ 1,645;
  - (e) R > 0 em pelo menos 4 das 6 criptos; no Nasdaq, R > 0.
- **Confirmação:** regra do E017; validação uma única vez, com 90 dias de aquecimento. M5 reportado com vários K.
- **Código necessário antes de rodar:** saída parcial no motor (metade no alvo, stop da outra metade movido, saída pelo sinal só depois da parcial), com testes.
- **O que espero e por quê:** que morra. Rompimento em cripto (H2) e ORB nos índices (H4) não mostraram informação da entrada; o filtro de momento é a novidade.
- **Critério de morte:** nenhuma das 2 passa na etapa 1, ou a sobrevivente não se confirma.
- **Grade aprovada pelo Alison em:** 2026-09-29 ("OK, vamos seguir as suas recomendações"; E030).

### E029 — 2026-09-29 — Mais duas fontes do Alison: MACD do TradingLab e MACD do Ross Cameron
- Abas lidas pelas legendas e pelas telas. Análise em `resultados/2026-09-29_H8_proposta_grade.md`.
- **Ross Cameron (Warrior Trading):**
  - usa o MACD de 1 minuto como filtro ("só compra com o MACD aberto") em **ações pequenas que disparam com notícia**, um mercado que a MyFundedPerps não tem;
  - o filtro em si é a H1 R1, já morta;
  - acordo de US$ 3 milhões com a FTC em 2022 por propaganda enganosa de ganhos; segundo a FTC, a maioria dos clientes perdeu dinheiro.
  - **Não se aplica.**
- **TradingLab, "BEST MACD Trading Strategy [86% Win Rate]":** compra o cruzamento do MACD **abaixo** do zero com o preço acima da EMA 200, stop atrás da EMA 200 e alvo de 1,5R. **É diferente da H1**, cuja R3 comprava acima do zero, sem EMA 200. Proposta como **H8**. O vídeo não mostra backtest dos "86%".

### H8 — MACD + EMA 200 (vídeo do TradingLab) — pré-registro — 2026-09-29
- **Família e ideia:** tendência com entrada na correção. Compra a virada do MACD abaixo do zero dentro de uma alta (preço acima da EMA 200); vende no espelho.
- **Mercados e tempo gráfico:**
  - 6 criptos em 15m, 24h, de 2020 a 2023 (o exemplo do vídeo é uma cripto em 15m);
  - Nasdaq-100 (proxy da Dukascopy) em 15m, só no pregão de NY e zerando no fim, de 2015 a 2023.
- **Regras** (sinal no fechamento, execução no minuto seguinte; só entra zerado):
  - MACD 12/26/9 e EMA 200, no fechamento.
  - **Compra:** o MACD cruza a linha de sinal para cima, as duas abaixo de zero, com o fechamento acima da EMA 200.
  - **Venda:** o espelho.
  - **Stop:** EMA 200 do candle do sinal ∓ 0,1 × ATR(14).
  - **Alvo:** 1,5R. No Nasdaq, também o fim do pregão.
  - Sem sinal nos primeiros 1.000 candles.
- **Variantes: 2.** z = 1,96. Sem etapa 2.
- **Baselines:**
  - aleatório: mesma frequência de sinais, 3 sementes, stop de 2 × ATR(14), alvo de 1,5R;
  - sempre comprado (E023) nos mesmos momentos: nas compras, o stop da estratégia; nas vendas, 2 × ATR(14).
- **Critérios v2:** (a) a (e) como na H7. **Confirmação:** E017, com 90 dias de aquecimento.
- **O que espero:** incerto. É uma ideia diferente da H1 (correção dentro da tendência), mas os cruzamentos de MACD não mostraram informação na H1.
- **Critério de morte:** nenhuma das 2 passa na etapa 1, ou a sobrevivente não se confirma.
- **Grade aprovada pelo Alison em:** 2026-09-29 ("OK, vamos seguir as suas recomendações"; E030).

### E030 — 2026-09-29 — Decisões do Alison: H7 e H8 aprovadas; cofre da H6 fechado
- "OK, vamos seguir as suas recomendações... Vamos um passo de cada vez por enquanto."
- **H7 e H8:** grades aprovadas como propostas. A H8 roda primeiro, porque o motor já a suporta; a H7 roda depois da saída parcial no motor.
- **Cofre da H6 Nasdaq V1: NÃO abrir** (recomendação do E027). O cofre continua intacto, com 0 aberturas.
- **Momento intradiário:** fica para depois ("se mesmo assim nada for lucrativo, a gente conversa mais tarde").

### E031 — H8 — resultado — 2026-09-29
- **Arquivos:** relatório em `resultados/2026-09-29_H8_v1.md` e log; `scripts/lab_h8.py`; `src/estrategias/h8_macd_ema.py`, com 3 testes novos.
- **Variantes rodadas:** 2. Total acumulado: 77. Nenhuma sobreviveu, e a validação não foi usada.
- **Cripto 15m:** **−0,268R ± 0,019** em 7.579 trades.
  - Acerto de 37% (alvo de 1,5R atingido em 37% dos trades).
  - Negativa nas 6 moedas e nos 4 anos.
  - **Pior que o aleatório** com a mesma saída: vantagem de −0,166R, t = −6,39.
  - O custo médio é de 0,171R por trade, porque o stop fica perto da EMA (mediana de 0,78% do preço).
  - **Diagnóstico** (o custo explica o resultado?): **não**.
    - Antes dos custos, a estratégia dá −0,097R, contra −0,033R do aleatório.
    - Por quartil de distância do stop, é negativa em todos, já antes dos custos: de −0,234 (stop mais curto) a −0,026 (stop mais longo).
    - A entrada é ruim por si; o stop colado na média só amplifica a perda.
- **Nasdaq 15m:** +0,014R ± 0,034 em 665 trades, limite −0,052, vantagem sobre o aleatório com t = 0,66. Antes dos custos, +0,050. Positiva em 5 de 9 anos. Não passa.
- **Veredito: a H8 MORRE.** Comprar a correção do MACD dentro da tendência perdeu dinheiro em cripto e não mostrou nada no Nasdaq. Os "86% de acerto" do título não aparecem: o acerto real foi de 37% em cripto.

### E032 — H7 — resultado — 2026-09-29
- **Código novo:**
  - **Saída parcial no motor** (`Saida.parcial_R`, `parcial_frac`, `stop_apos_parcial_R`, `sinal_so_apos_parcial`):
    - ordem limitada, como o alvo;
    - stop e parcial no mesmo minuto: assume o stop;
    - o novo stop vale a partir do minuto seguinte;
    - o registro do trade soma a parte realizada.
    - Sem a opção, o motor dá os mesmos resultados de antes.
  - `src/estrategias/h7_rompimento_momento.py`, com o Chandelier Exit igual ao do TradingView (fechamento, 22, 2 × ATR).
  - **115 testes**, todos passando: 4 da saída parcial no motor e 5 da H7.
  - **Detalhe de codificação:** as formas A e B só partem do 1º fechamento além da linha (o candle anterior não pode ser também um rompimento). Isso segue a descrição do vídeo e foi fixado antes de rodar.
- **Arquivos:** relatório em `resultados/2026-09-29_H7_v1.md` e log; `scripts/lab_h7.py`.
- **Variantes rodadas:** 2. Total acumulado: 79. Nenhuma sobreviveu, e a validação não foi usada.
- **Cripto 1h:**
  - **−0,027R ± 0,068** em só **358 trades** (cerca de 15 por ano por moeda), acerto de 41%, limite −0,157;
  - com custos 1,5×: −0,047; com o stop no meio do caminho: −0,067;
  - positiva em 3 de 6 moedas; por ano, de +0,116 (2020) a −0,268 (2023).
  - O aleatório com a mesma saída deu −0,187 (vantagem +0,160, t = 2,15), mas a estratégia em si fica negativa.
  - Custo de 0,053R por trade; antes dos custos, +0,026.
  - Critérios: S n n S n.
- **Nasdaq 15m:** −0,036R ± 0,099 em só **117 trades em 9 anos**. Falha até no (a).
- **Veredito: a H7 MORRE.**
- **Leitura:** a regra fixa de "consolidação com 2 toques de cada lado" dispara pouco; a expectativa era de centenas a milhares de trades, e vieram 358 e 117. Onde dispara, não ganha. Mesmo que ganhasse, 1 a 2 trades por semana seria lento demais para a mesa (a lição da H6). A limitação de fundo, registrada antes de rodar, continua: o olho humano escolhe outras consolidações, e isso o teste não mede.
- **Balanço das fontes do YouTube (H6, H7, H8):** nenhuma sobreviveu como estratégia para a mesa.
  - A regra do vídeo da H6 perdeu em cripto e só teve uma vantagem pequena, sem EV, no Nasdaq zerando no fim do pregão, que é uma regra nossa.
  - H7 e H8 morreram na etapa 1.
  - Os números dos títulos ("86%", "+130%") não se reproduziram com custos e com anos de dados.
- **Próximo passo:** conversar com o Alison ("se mesmo assim nada for lucrativo, a gente conversa mais tarde").

### E033 — 2026-09-29 — Alison escolhe a hipótese com base acadêmica (momento intradiário)
- "Pode seguir com o que conversamos, curioso para ver essa hipótese melhor."
- **Mercados da plataforma** (lista salva na F1, `resultados/2026-09-28_mercados_mfp.json`): 121 no total, incluindo:
  - ações dos EUA (NVDA, TSLA, META, AAPL etc.), QQQ, SP500 e XYZ100;
  - ouro, prata, petróleo (CL e BRENTOIL), gás natural e cobre;
  - EUR e JPY;
  - e dezenas de criptos.
- **Pesquisa:**
  - Gao, Han, Li e Zhou (2018, JFE; SPY 1993–2013);
  - Baltussen, Da, Lammers e Martens (2021, JFE; mais de 60 futuros de 1974 a 2020; mecanismo: proteção de quem vende opções);
  - Zarattini, Aziz e Barbon (2024, SSRN 4824172; "área de ruído"; Sharpe de 1,33 líquido no SPY, 2007–2024).
  - Regras exatas tiradas da replicação independente `github.com/codecat-ops/zarattini-2024-momentum-spy` (docs/SPEC.md), porque o PDF do SSRN bloqueou a leitura automática.
- **Informação externa sobre o período do cofre** (como no E008): a mesma replicação relata, no SPY, **2025: −4,9%** (Sharpe −0,27) e **2026 até agora: −12,9%** (Sharpe −1,91), com os futuros do S&P iguais (correlação de 0,97). No período 2020–2024, o Sharpe ficou entre 1,4 e 2,0 por ano.
  - Nenhum dado nosso foi aberto, mas agora sabemos o que outra pessoa mediu no período do nosso cofre para esta estratégia no SPY.
  - Isso vai registrado na proposta e no pré-registro.
- **Código pronto antes da aprovação:**
  - `src/estrategias/h9_momento_intradiario.py`, com faixas, VWAP e baselines próprios;
  - `scripts/lab_h9.py`;
  - 5 testes novos (4 de estratégia e 1 de ponta a ponta no laboratório).
  - Nada rodou em dado real.

### H9 — Momento intradiário com área de ruído (Zarattini, Aziz e Barbon, 2024) — pré-registro — 2026-09-29
- **Família e ideia:** momento intradiário nos índices de ações. Quando o preço sai da faixa normal de movimento do horário, a tendência do dia tende a continuar até o fechamento, pelo mecanismo de proteção de quem vende opções (Baltussen et al., 2021). Proposta com a seção de distorções em `resultados/2026-09-29_H9_proposta_grade.md`.
- **Mercados e tempo gráfico:** Nasdaq-100 e S&P 500 (proxies da Dukascopy), desde 2015, só no pregão regular de NY e zerando no fim, com candles de 30 min (checagens das 10h00 às 15h30).
- **Regras:**
  - sigma do horário t = média, nos 14 pregões anteriores (com pelo menos 10 disponíveis), de |fechamento em t ÷ abertura do dia − 1|;
  - faixa superior = máx(abertura, fechamento anterior) × (1 + sigma); faixa inferior = mín(abertura, fechamento anterior) × (1 − sigma);
  - **compra** se o fechamento da checagem passa da faixa superior; **venda** se fica abaixo da inferior; execução no minuto seguinte;
  - **saída** numa checagem: comprado com o fechamento abaixo de máx(VWAP, faixa superior); vendido com o fechamento acima de mín(VWAP, faixa inferior). Inverte no mesmo minuto se houver sinal contrário;
  - **stop de proteção** na faixa oposta do momento da entrada;
  - fim do pregão.
  - O VWAP é aproximado pelos candles de 30 min.
- **Variantes: 1** (um grupo com os 2 índices). z = 1,645. Sem etapa 2. Parâmetros do artigo, sem otimização.
- **Baselines:**
  - aleatório: nas checagens com faixa, mesma frequência, direção sorteada, stop a uma largura da faixa, mesma saída; 3 sementes;
  - sempre comprado (E023) nos mesmos momentos: nas compras, o mesmo stop; nas vendas, uma largura da faixa.
- **Critérios v2 (E020):**
  - (a) ≥ 300 trades;
  - (b) limite > 0 com erro por semana;
  - (c) R > 0 com custos 1,5× e com o stop no meio do caminho;
  - (d) acima dos baselines e vantagem sobre o aleatório com t ≥ 1,645;
  - (e) R > 0 nos 2 índices.
- **Confirmação:** E017; validação uma única vez, com 90 dias de aquecimento. M5 com vários K.
- **O que espero e por quê:**
  - no desenvolvimento, positiva antes dos custos; depois dos nossos custos (~1,5 bp por ida e volta, mais o swap, contra uma vantagem de ~2,6 bps por trade na replicação), pequena e no limite do (b);
  - se passar, o cofre provavelmente reprova, pelo colapso relatado em 2025–2026;
  - chance de G1 baixa.
- **Critério de morte:** não passa na etapa 1, ou não se confirma.
- **Grade aprovada pelo Alison em:** 2026-09-29 ("Beleza, pode rodar...").

### E034 — H9 — resultado — 2026-09-29
- **Arquivos:** relatório em `resultados/2026-09-29_H9_v1.md`, log e trades; `scripts/lab_h9.py`.
- **Variantes rodadas:** 1. Total acumulado: 80. Não sobreviveu, e a validação não foi usada.
- **Desenvolvimento** (Nasdaq-100 e S&P 500, 2015 a nov/2023):
  - 4.332 trades (cerca de 2,7 por pregão com trade), acerto de 37%, **+0,008R ± 0,007**;
  - limite por semana **−0,002**;
  - com custos 1,5×: +0,001; com o stop no meio do caminho: +0,008;
  - sempre comprado −0,012; aleatório −0,013;
  - **vantagem sobre o aleatório de +0,022R, t = 3,18**;
  - Nasdaq +0,014 e S&P +0,002; comprado +0,018, vendido −0,001.
  - **Critérios: S n S S S.** Falha só no (b), por 0,002R.
- **Diagnósticos** (informativos; procurar artefato antes de concluir):
  - **Tamanho do efeito:** **+3,2 bps por trade antes dos custos** (+0,019R), contra os +2,6 bps da replicação independente. O efeito aparece nos nossos dados no tamanho publicado. O custo da mesa leva ~1,1 bp (0,011R), e sobram ~2 bps.
  - **De onde vem o ganho:** saídas no fim do pregão (1.364 trades) dão +0,327R; saídas por sinal (2.944) dão −0,131R.
  - **Stop de proteção:** só 24 trades pararam nele (−1,02R); o acréscimo em relação ao artigo não distorce o resultado.
  - **Horários:** 6 entradas fora do horário das checagens (11h40, 12h31 etc.), por falhas de minutos na Dukascopy (o motor aceita até 30 min de atraso). Efeito desprezível.
  - **Por ano:**
    - 2015–2019 misturado (de −0,056 a +0,073 conforme o índice);
    - **2020–2023 positivo nos dois índices em todos os anos** (de +0,011 a +0,045);
    - bate com a replicação, que achou 2020–2024 bom.
  - **M5 no desenvolvimento** (colchão de $300; 3.000 contas):

    | K | EV da estratégia | EV com custos 1,5× | EV do aleatório |
    |---|---|---|---|
    | 2 | **+$9** (P(passar) de 29,5%) | −$9 | −$38 |
    | 3 | −$1 | −$18 | −$39 |
    | 4 | −$8 | −$24 | −$40 |
    | 6 | −$22 | −$33 | −$40 |
    | 12 | −$38 (P(passar) de 73%, mas ~1.050 dias) | −$39 | −$40 |

    É a primeira hipótese com EV perto de zero e muito acima do aleatório. Mas só dá EV positivo no K mais agressivo e sem estresse de custo, o que não cumpre o G1.
- **Veredito: a H9 MORRE** pelo critério pré-registrado (b).
- **Leitura:**
  - O momento intradiário é o **primeiro efeito real** que o laboratório encontrou: bate o acaso com folga, nos 2 índices, no tamanho publicado e consistente em 2020–2023.
  - Só que ele é **fino demais para os custos e as regras desta mesa**, e há o relato externo de colapso em 2025–2026 (E033).

### E035 — 2026-09-29 — Decisão do Alison: testar o momento intradiário em ouro, prata, Brent, euro e iene; dados novos
- "Testa aí, xauusd, esses bagulhos de moedas e tal que você citou" (opção (a) do E034).
- **Mercados novos** (proxies da Dukascopy, lado BID, candles de 1 min), cada um com seu mercado na plataforma:

  | Proxy | Mercado na plataforma | Classe de custo |
  |---|---|---|
  | XAUUSD | xyz:GOLD | commodities (a mesma dos índices) |
  | XAGUSD | xyz:SILVER | commodities |
  | BRENTCMDUSD | xyz:BRENTOIL | commodities |
  | EURUSD | xyz:EUR | forex |
  | USDJPY | xyz:JPY | forex; a cotação da plataforma pode ser a inversa, e o momento é simétrico |

  - As classes seguem as taxas publicadas: commodities pagam como índices; EUR e JPY como forex (comissão de 0,0025% e slippage de 0,05 bp).
- **Mudanças:**
  - **config:**
    - os 5 mercados em `custos.mercados`, com as casas decimais da lista da plataforma;
    - a lista `dados.mercados.dukascopy_macro`;
    - o fator de preço por mercado (`dukascopy_fator_por_mercado`: EURUSD 100.000; os outros 1.000).
    - Conferido num dia de desenvolvimento (04/03/2019): ouro 1.287,7; prata 15,14; Brent 65,46; EURUSD 1,134; USDJPY 111,88.
  - **`src/dados.py`:**
    - `fator_dukascopy()`;
    - opção `--desde` no download.
  - 1 teste novo; **121 testes**, todos passando.
- **Download:** jan/2015 a ago/2026, retomável, com recomeço automático; log em `dados/download_macro_log.txt`. O período do cofre é baixado, mas continua trancado na carga, como na F1.

### H10 — Momento intradiário (a regra da H9) em ouro, prata, Brent, euro e iene — pré-registro — 2026-09-29
- **Família e ideia:** a mesma da H9, em outras classes de ativos. Baltussen et al. (2021) acharam o momento intradiário também em commodities e moedas. É uma **amostra nova de mercados**, que o laboratório nunca viu. O pré-registro é escrito **antes** de os dados terminarem de baixar.
- **Regra:** a da H9, **sem nenhuma mudança** (`src/estrategias/h9_momento_intradiario.py`): área de ruído de 14 pregões, checagens a cada 30 min das 10h00 às 15h30, saída por VWAP/faixa, stop de proteção na faixa oposta, zerando às 16h00.
- **Pregão: o da NYSE (9h30–16h00 de NY, com o calendário de feriados) para os 5 mercados.** É uma escolha minha, fixada antes de ver dado, por simplicidade (a mesma infraestrutura da H9) e porque o robô operaria no horário dos EUA.
  - Contra: esses mercados têm outros horários de fechamento (ouro na COMEX às 13h30; Brent às 14h30; moedas sem fechamento), e o efeito pode se concentrar neles.
- **Mercados e período:** os 5 do E035, desenvolvimento de 2015 a nov/2023.
- **Variantes: 1** (um grupo com os 5 mercados). z = 1,645. Sem etapa 2.
- **Baselines:** os da H9 (aleatório nas checagens com stop de uma largura da faixa; sempre comprado do E023).
- **Critérios v2:**
  - (a) ≥ 300 trades;
  - (b) limite > 0 com erro por semana;
  - (c) R > 0 com custos 1,5× e com o stop no meio do caminho;
  - (d) acima dos baselines e vantagem sobre o aleatório com t ≥ 1,645;
  - (e) **R > 0 em pelo menos 3 dos 5 mercados**.
- **Confirmação:** E017; validação uma única vez, com 90 dias de aquecimento. M5 com vários K.
- **O que pode distorcer:**
  - **Contra:** a escolha do pregão de NY; o custo de commodities igual ao dos índices (no forex, a metade); o Brent é um CFD que rola de contrato (saltos entre dias, que só alargam a faixa no dia seguinte).
  - **A favor:** mais mercados dão mais trades e diversificação.
  - **Neutro:** o proxy é da Dukascopy (BID), e a plataforma opera pelo mark da Hyperliquid; a diferença ainda não foi medida. O relato de colapso em 2025–2026 (E033) é do SPY e não se sabe se vale para estes mercados.
- **O que espero:** incerto. A pesquisa dá base, mas o horário escolhido e os custos podem apagar o efeito. Se ele aparecer, a H9 (índices) e a H10 juntas seriam uma carteira para o M5, o que exige o simulador de carteira pendente.
- **Critério de morte:** não passa na etapa 1, ou não se confirma.
- **Grade aprovada pelo Alison em:** 2026-09-29 ("Testa aí...", sobre a proposta (a) do E034). Os detalhes (pregão de NY para todos; critério (e) em 3 de 5) foram fixados por mim antes de haver qualquer dado desses mercados, e ficam aqui para a auditoria dele.

### E036 — 2026-09-29 — H10: só o trecho recente dos dados (decisão do Alison, antes de rodar)
- O download completo (jan/2015 a ago/2026, 5 mercados) levaria ~4 horas. O Alison: "Não é melhor baixar apenas um trecho mais recente?"
- **Mudança na H10, antes de qualquer resultado** (só vi os preços de jan a mar/2015 do ouro, para conferir a escala):
  - **Desenvolvimento de jan/2020 a nov/2023** (antes: 2015 a nov/2023);
  - **validação** como sempre (dez/2023 a abr/2025);
  - **o cofre não foi baixado**: só será, se alguma variante chegar a ele.
  - O resto do pré-registro fica igual.
- **Por quê:** 4 anos em 5 mercados ainda dão milhares de trades. É também o período em que a replicação independente achou o efeito mais forte no SPY (2020–2024). **Contra:** menos anos para ver a estabilidade, e o período é favorável ao efeito; a validação e o cofre continuam sendo a proteção.
- **Download:** em duas filas paralelas (ouro, prata e Brent; euro e iene), com a opção nova `--ate` do `src/dados.py`. Logs em `dados/download_macro_fila1.txt` e `..._fila2.txt`. Os meses de 2015 do ouro que já tinham baixado ficam guardados e não entram no teste.

### E037 — H10 — dados e resultado — 2026-09-29
- **Download concluído** em cerca de 1h20, em 3 filas (o Brent numa 3ª fila; mais filas deixaram cada uma mais lenta).
- **Checagem dos dados** (`scripts/checar_macro.py` → `resultados/2026-09-29_H10_dados.md`):
  - jan/2020 a abr/2025, 64 meses por mercado;
  - 0 duplicatas, 0 OHLC inválido, 0 nulos;
  - minutos faltando no pregão de NY: 1,0–1,3% nos metais e no Brent, 0,1–0,2% nas moedas;
  - preços coerentes com a história.
  - **Saltos entre dias acima de 3%:** prata 58, Brent 115, ouro 4, EURUSD 0, USDJPY 2. Conferidos um a um nos maiores, **são reais**: Brent −22% em 09/03/2020 (guerra de preços) e −24% em 21/04/2020 (petróleo negativo); prata +11% em 01/02/2021 (onda do Reddit); crash da Covid. O salto persiste no 5º minuto, então não é tique errado. É a madrugada desses mercados, que negociam quase 24 horas.
- **Resultado** (desenvolvimento de 2020 a nov/2023; `resultados/2026-09-29_H10_v1.md`):
  - 4.641 trades, acerto de 36%, **−0,012R ± 0,006**, limite −0,021;
  - **negativa nos 5 mercados e nos 4 anos**;
  - igual ao aleatório (vantagem −0,001, t = −0,10).
  - Critérios: S n n n n.
  - O cabeçalho do relatório diz "índices de 2015" por ser o texto padrão; o grupo começou em 2020.
- **Diagnóstico** (o custo explica?): **não**.
  - Antes dos custos, −0,003R por trade (−0,7 bp); por mercado, de −4,8 bps (Brent) a +0,9 bp (prata). **Não há efeito nenhum**, mesmo sem custo.
  - Nos índices (H9) havia +3,2 bps antes dos custos e t = 3,18 contra o aleatório.
- **Variantes:** 1. Total acumulado: 81. A validação não foi usada.
- **Veredito: a H10 MORRE.**
- **Leitura:**
  - No pregão de NY, o momento intradiário aparece nos índices de ações (H9) e não em metais, petróleo e moedas. Isso bate com a explicação da proteção de quem vende opções, que é mais forte nas opções de índice.
  - **Ressalva registrada antes de rodar:** o horário de NY não é o horário de fechamento desses mercados, e o efeito poderia estar em outro horário. Não foi testado.

### E038 — 2026-09-29 — Vídeos do amigo do Alison e um vídeo de MACD em 3 sistemas (análise, sem teste)
- **Canal do amigo ("Aprenda Ser Livre"):**
  - **"Lucrar no daytrade é fácil, o desafio é ser consistente":**
    - opera **NQ/MNQ (futuros do Nasdaq) de forma manual**, em 5 contas de US$ 250 mil da PropShopTrader;
    - mostra US$ 66 mil líquidos em 34 dias nas 5 contas (107 mil de ganhos, 41 mil de perdas); numa conta, acerto de ~83%, fator de lucro de 3,36 e ganho médio de 0,67 da perda média;
    - o vídeo trata das **regras da mesa**: consistência de 30%, 3 dias de trade por semana, teto de saque semanal, colchão antes do 1º saque, limite de perda diária e alerta de "jogatina" ao mudar o tamanho da mão;
    - cita um "dia de fúria" que pôs uma conta em prejuízo;
    - a estratégia em si não é descrita ("zonas de operação");
    - há link de afiliado da mesa e venda de curso.
  - **"Gaste R$0 usando modelos gratuitos no OpenClaw":** configurar um agente de IA com modelos gratuitos numa VPS, para relatórios e alertas. Não traz estratégia.
    - **Cuidado de segurança para o projeto:** agentes com acesso à máquina inteira e às chaves não devem receber a chave da MyFundedPerps (regra 4 do CLAUDE.md).
- **"My Boring MACD Trading Strategy Just Hit 71% Win Rate This Month" (The Secret Mindset):**
  - **3 sistemas:**
    1. tendência: só compra com o MACD acima do zero (a regra da H1 R3), cruzamento "longe do zero" (±0,5, um valor que depende da escala do preço), esperando 2 ou 3 candles;
    2. reversão: divergência com gatilho no histograma;
    3. confirmação: 3 tempos gráficos (diário = viés, 4h = sinal, 1h = gatilho) e suporte e resistência.
  - **Gestão:** stop no último topo ou fundo, metade no alvo de 2R, stop no empate e o resto sai no cruzamento contrário.
  - **Não há backtest**; os "71%" são de um mês.
  - **O que é novo em relação às H1/H1b/H8:** o filtro de 3 tempos gráficos. O resto já foi testado ou depende de leitura visual.
- **Nenhum teste rodou.** Próximo passo proposto ao Alison: incluir as ideias codificáveis (o filtro de 3 tempos gráficos e padrões curtos de abertura) num funil nos índices, e pedir ao amigo as regras exatas.

### E039 — 2026-09-29 — Live do amigo ("Dia 10/30 operando ao vivo Nasdaq, Bitcoin e Forex", 1h42, 10/03/2026)
- **A live não tem legenda nenhuma** (nem automática). Foi analisada pelas imagens, com telas em 4, 15, 25, 40, 55, 70, 85 e 101 minutos. O que ele fala não foi ouvido.
- **Três mesas ao mesmo tempo:**
  - **YRM Prop (futuros):** conta de ~US$ 51 mil, **MNQ em 1 minuto** na DeepChart, com **perfil de volume e VWAP** (diário e semanal). Às 10h26, o dia estava em +US$ 237,70 e ele estava sem posição.
  - **The5%ers (forex/CFD, MT5):** conta de US$ 5 mil com o robô **"VWAP Reversal Pro"**. Os painéis mostram status, direção, posições, "preço médio", "fluxo atual" e "VWAP inverteu/inversões". Na lista de robôs aparecem os exemplos que vêm com o MT5 (ExpertMACD, ExpertMAPSAR etc.) e dois próprios ("SEU MADRUGA", "T-Killer v3.5"). Muitas posições pequenas em GBPUSD, USDCAD, AUDUSD e BTCUSD. No BTC, **três compras em preços cada vez mais baixos** (70.676 → 70.360 → 69.901), o que **parece** preço médio (aumentar a posição contra o movimento). O saldo foi de 5.014,87 para 5.001,29 durante a live, praticamente estável.
  - **E8 Markets (MT5):** conta de ~US$ 25 mil (saldo de 25.420); ouro em 1 min; **3 compras de AUDUSD em 2 segundos**. O saldo ficou estável.
- **O que dá para extrair:**
  - **O VWAP é a ferramenta central dele**, no manual (MNQ) e no robô (forex).
  - Várias contas baratas em mesas diferentes: a economia da mesa pesa tanto quanto a estratégia.
  - **Sinal de alerta:** preço médio e muitas posições somadas explicam acerto alto com ganho médio menor que a perda média (0,67 no outro vídeo). Isso funciona até um dia de tendência forte contra, o risco de cauda que acaba contas.
- **Não é codificável pelas imagens:** as regras do robô (o painel de configurações estava fechado) e o critério manual dele.
- **O dado mais valioso** seria o **histórico de trades exportado** das mesas (relatório do MT5 ou CSV do painel), sem senha nenhuma. Com centenas de trades reais, o laboratório mede a vantagem dele (acerto, payoff, cauda, comparação com o acaso) e roda o M5 com as regras da MyFundedPerps. Também seria útil o código (.mq5) ou as regras do "VWAP Reversal Pro".

### E040 — 2026-09-30 — Decisão do Alison: estratégia própria, sem pedir nada ao amigo; proposta do funil (H11)
- "Não vou pedir nada ao meu amigo, precisamos entender e descobrir nossa própria estratégia."
- **Proposta H11** (`resultados/2026-09-30_H11_funil_proposta.md`): funil de 29 padrões intradiários no Nasdaq-100 e no S&P 500.
  - **Famílias:**
    - A, gap com pullback (setup do Alison): 16;
    - B, momento do dia (Gao; Baltussen): 3;
    - C, VWAP (reversão e retomada): 4;
    - D, falso rompimento da faixa de abertura: 4;
    - E, filtro do MACD diário: 2.
  - **Etapas:**
    - **triagem** nos anos ímpares do desenvolvimento, com barra solta (t ≥ 2,0 com erro por semana, R > 0 nos 2 índices e com custos 1,5×, ≥ 100 trades; no máximo as 5 de maior t);
    - **confirmação** nos anos pares, com os critérios v2 e z corrigido pelo número de candidatas;
    - **validação** uma única vez (E017);
    - cofre com ok;
    - G1 com as confirmadas em carteira (exige o simulador de carteira).
- **Grade aprovada pelo Alison em 2026-09-30** ("Pode começar").

### H11 — Funil de padrões intradiários nos índices — pré-registro — 2026-09-30
- **Proposta:** `resultados/2026-09-30_H11_funil_proposta.md` (aprovada no E040).
- **Código:**
  - `src/estrategias/h11_intradiario.py`: famílias A–E, com baselines próprios;
  - `scripts/lab_h11.py`: o funil;
  - `laboratorio.rodar_grupo(..., baselines=False)` para a triagem;
  - 9 testes novos; **130 testes**, todos passando.
- **Detalhes fixados no código, antes de rodar:**
  - **Aquecimento:** 20 pregões (105 com o filtro do MACD diário); a validação carrega 160 dias antes.
  - **Horários:**
    - A: gatilho em candles de 5 min que começam de 9h30 + espera até antes das 12h00;
    - C: candles de 5 min que fecham entre 10h00 e 15h00;
    - D: 1º fechamento de 1 min fora da faixa antes das 12h00; volta em até 15 min.
  - **A:** o candle que fecha em 9h30 + espera tem de fechar do lado do gap, e o candle do gatilho também. Um trade por dia.
  - **D:** um trade por dia, só o 1º rompimento.
  - **Baselines da confirmação:**
    - **aleatório:** entradas sorteadas nos candles do horário da família, com a mesma frequência, direção sorteada, stop de 1,5 × ATR(14) do tempo gráfico e as mesmas saídas em R, tempo e fim do pregão. Nas variantes com alvo num nível de preço (C1 no VWAP; D no lado oposto da faixa), o aleatório usa alvo de 1R. É uma simplificação;
    - **sempre comprado (E023):** nos momentos da estratégia; nas compras, o stop e o alvo da estratégia; nas vendas, stop de 1,5 × ATR(14).
  - **Triagem:** só estratégia e custos 1,5×.
  - **Validação** (só as confirmadas, uma vez):
    - (i) limite dos anos pares + validação > 0, com o z da confirmação e erro por semana;
    - (ii) R médio da validação ≥ 0;
    - (iii) o conjunto acima dos dois baselines.
    - Os anos ímpares ficam fora do conjunto, porque foram usados para escolher.
- **Variantes: 29.**

### E041 — H11 — resultado — 2026-09-30
- **Arquivos:** relatório em `resultados/2026-09-30_H11_v1.md`, a tabela da triagem em `..._triagem.csv`, os trades e o log. A triagem rodou em ~6 min.
- **Triagem** (anos ímpares): **nenhuma das 29 passou**. A confirmação, a validação e o cofre não foram usados.
- **Melhor da triagem:** "A gap 0,6%, espera de 15 min, VWAP, fim do pregão" (o setup do Alison).
  - 455 trades, acerto de 21%, **+0,292R**, t = 1,41 (a barra era 2,0); +0,320 no Nasdaq e +0,249 no S&P; com custos 1,5×, +0,202.
  - **Todo o resultado vem de 10 trades** (dias de tendência seguros até o fechamento, de +13 a +30R). Sem eles, −0,143R.
  - As mesmas variantes com alvo de 2R ficam negativas (−0,09 a −0,21).
- **As outras:**
  - família A com EMA 20: −0,24 a −0,33R;
  - B (momento às 10h e às 15h30): −0,09 a −0,18R;
  - C1 (volta ao VWAP): −0,07 e −0,08R, com t de −3,7 e −5,7 (voltar ao VWAP perde, coerente com momento);
  - C2 (retomada do VWAP): cerca de −0,30R;
  - D (falso rompimento): −0,02 a −0,22R;
  - E (filtro do MACD): +0,01 e −0,09R.
- **Diagnósticos:**
  - **Custo dos trades curtos:** nas variantes com stop no extremo de um candle de 5 min (A com EMA, C2, D), a distância mediana do stop é de **0,08–0,11% do preço**, e o custo chega a **0,13–0,15R por trade**. Mesmo antes do custo, quase todas ficam negativas.
  - **B1 conferida contra bug:** direto nos dados, seguir o dia das 15h30 às 15h59 rende +1,4 bp (Nasdaq) e +0,6 bp (S&P) por dia, em 2015–2023 (correlação de +0,07 a +0,09). **O efeito da pesquisa existe**, mas é do tamanho do custo de ida e volta (~1,5 bp), e o stop de 1 ATR (11% dos trades) tira o resto.
    - O motor foi conferido em 2019: entradas às 15h30 e saídas às 15h59, como esperado; bruto de −0,1 bp e −0,05R depois dos custos.
- **Variantes:** 29. Total acumulado: **110**.
- **Veredito: a H11 MORRE** na triagem.
- **Leitura (H6, H9 e H11 juntas):**
  - No Nasdaq e no S&P, o que ganha dinheiro no intradiário são **poucos dias de tendência seguros até o fechamento**, e nenhuma regra testada consegue escolhê-los com antecedência.
  - Os efeitos que existem (momento intradiário, a última meia hora) são **do tamanho do custo da mesa**.
  - Trades de minutos com stop apertado pagam 13–15% do risco em custo a cada trade.

### E042 — 2026-09-30 — A "meta" da MyFundedPerps: vantagem e frequência necessárias para EV positivo
- **Pedido do Alison:** seguir com a proposta de medir a meta ("vamos seguir com a sua proposta agora").
- **Método** (`scripts/meta_ev.py` → `resultados/2026-09-30_meta_ev.md` e `.csv`):
  - M5 sobre **estratégias sintéticas**, sem dado de mercado; 3 anos de dias úteis; trades independentes com número diário sorteado; média exata μ (R líquido por trade).
  - **Três estilos** (perda de −1R): "tiro longo" (acerta 20%), "tendência" (40%) e "scalp" (60%).
  - Para cada caso, o melhor K entre 2 e 12; 3.000 contas.
  - EV = P(passar) × saques em 90 dias − US$ 40.
- **Resultados:**
  - **μ = 0 dá EV negativo em todos os casos** (−US$ 5 a −39). Sem vantagem, a conta é mau negócio.
  - **Com 1 trade por semana, nada dá EV positivo**, nem +0,30R.
  - **Fronteira do EV > 0** (melhor estilo):

    | Trades por semana | μ mínimo para EV > 0 | Para EV de ~US$ 50–100 |
    |---|---|---|
    | 2 | ~+0,20R | +0,30R dá só +US$ 35–38 |
    | 5 | ~+0,07R | ~+0,10R |
    | 10 | ~+0,05R | ~+0,07–0,10R |
    | 20 | ~+0,02R (scalp) ou ~+0,04R (tendência) | ~+0,05R |
    | 40 | ~+0,02R (scalp) | ~+0,05R |

  - **O estilo pesa:** com a mesma vantagem, "scalp" (acerto alto) > "tendência" > "tiro longo". O limite de perda de 3% pune a variância. Exemplo: μ = +0,10R a 10/semana dá +US$ 152 (scalp), +US$ 97 (tendência) e +US$ 35 (tiro longo).
- **Onde ficaram as melhores achadas** (μ líquido × frequência):

  | Hipótese | μ | Frequência | Situação |
  |---|---|---|---|
  | H9 (índices) | +0,008R | ~10/semana | precisaria de ~6 vezes mais |
  | H6 Nasdaq | +0,057 (dev) / +0,034 (val) | ~2/semana | precisaria de ~+0,20R |
  | H4b ORB cripto (validação) | +0,047R | ~9/semana | perto da fronteira, mas não bateu o acaso |
  | H11, gap do Alison (triagem) | +0,29R | ~2/semana | na fronteira, mas o resultado vem de 10 dias |

- **Ressalvas:** os trades sintéticos são independentes e bem-comportados. Trades reais têm caudas e sequências ruins, então a meta real é **mais alta**. O horizonte de 90 dias da financiada é conservador; ficar mais tempo aumenta o EV.
- **Leitura:** a alavanca principal é **a frequência**, com trades de acerto alto e várias fontes de vantagem somadas (carteira). A régua passa a valer para julgar qualquer ideia antes de testar: **com ~2 trades por dia, é preciso pelo menos +0,05R líquido por trade, de forma consistente.**

### E043 — 2026-09-30 — Plano de pesquisa guiado pela régua (proposta)
- **Pedido do Alison:** "Primeiro, bole um plano para a gente achar uma estratégia que se adeque a essas condições, depois execute os testes. Existem robôs por aí [...] que foram descobertos de alguma forma e estão dando certo."
- **Plano** (`resultados/2026-09-30_plano_de_pesquisa.md`), em 4 frentes, na ordem:
  - **(0)** simulador de carteira (pendência para qualquer G1);
  - **(1) H12:** reversão depois de liquidação em cascata em cripto (minutos). Tem razão econômica (fornecer liquidez a vendas forçadas), é o estilo de acerto alto que a régua favorece, o alvo é grande perto do custo e usa dados que já temos;
  - **(2) H13:** tendência diversificada em dezenas de mercados da plataforma, no diário. Exige dados diários novos, com ok;
  - **(3) H14:** modelo que combina muitos sinais (aprendizado de máquina). Exige o pacote scikit-learn, com ok e registro.
- **Regras mantidas:**
  - cada frente com pré-registro, distorções, grade aprovada, funil, validação única e cofre com ok;
  - o critério (d), bater o acaso, continua obrigatório;
  - a régua (EV no M5) entra em todo relatório.
- **Primeiro passo proposto:** a H12, com 8 variantes (k ∈ {5, 8} desvios-padrão; janela de 1 ou 5 min; volume ≥ 3 × a mediana de 24h; saída pelo repique de 50% com tempo de 60 min, ou só tempo de 30 min). Descoberta em 2021 e 2023; confirmação em 2020 e 2022.
- **Aprovado pelo Alison em 2026-09-30** ("Pode executar"): o simulador de carteira e a H12. As frentes 2 e 3 pedem ok próprio (dados e pacote novos).

### E044 — 2026-09-30 — Frente 0: simulador de carteira e funil reutilizável
- **`src/simulador_mesa.py`:**
  - `grupo_do_mercado`;
  - `aplicar_teto_grupos`: aceita um trade só se o risco aberto do grupo, a 1R por trade, cabe no `risco_max_grupo_em_R` do config (hoje 1R, ou seja, uma posição por grupo por vez);
  - `combinar_sobrepostos`: junta trades abertos ao mesmo tempo num bloco, somando resultado e pior momento, sem contar aprovação por pico no meio do bloco; é conservador;
  - `avaliar_carteira`.
- **`src/laboratorio.py`:**
  - `funil()` genérico (triagem → confirmação → validação única), a partir da H11;
  - `regua_m5()`: EV da carteira no melhor K, mostrado na confirmação e na validação;
  - `triagem_funil()`.
- **Testes:** 4 de carteira e 2 do funil de ponta a ponta, incluindo que a validação não roda duas vezes. **141 testes**, todos passando.
- Isto resolve a pendência "simulador de carteira com o teto de risco por grupo".

### H12 — Reversão depois de liquidação em cascata em cripto — pré-registro — 2026-09-30
- **Família e ideia:** fornecer liquidez a vendas e compras forçadas. Quando o preço dispara ou despenca em minutos com volume anormal (liquidações em cascata), o movimento tende a devolver uma parte. Tem o estilo que a régua favorece: acerto alto, vários eventos e alvo grande perto do custo. Plano em `resultados/2026-09-30_plano_de_pesquisa.md`.
- **Mercados e dados:** 6 criptos (BTC, ETH, SOL, XRP, BNB, DOGE), 1 min do último negócio, com o mark na execução, 24h.
- **Evento** (no fechamento do minuto t; `src/estrategias/h12_cascata.py`):
  - retorno logarítmico da janela (1 ou 5 min) além de k × σ, onde σ é o desvio de 1 min nas 24h anteriores, vezes a raiz da janela;
  - volume da janela ≥ 3 × a mediana do volume de 1 min das 24h anteriores, vezes a janela;
  - queda → compra; alta → venda.
- **Stop:** um movimento M além do extremo da janela, onde M = |fechamento em t − fechamento em t − janela|.
- **Saídas:**
  - (i) alvo de 50% de M de volta, com saída por tempo em 60 min;
  - (ii) só tempo, em 30 min.
- **Aquecimento:** 12h de dado (1.440 min para σ e a mediana, com metade como mínimo).
- **Variantes: 8** (k ∈ {5, 8} × janela ∈ {1, 5} min × saída ∈ {i, ii}).
- **Baselines** (confirmação):
  - **aleatório:** **os mesmos eventos com a direção sorteada** e a mesma geometria (a mesma distância do stop e do alvo), 3 sementes. Testa se a direção "reverter" tem informação;
  - **sempre comprado (E023):** compra em todos os eventos, com a mesma geometria.
- **Funil** (`laboratorio.funil`):
  - **triagem** em 2021 e 2023 (jan–nov): ≥ 100 trades, R > 0 em ≥ 4 das 6 moedas, R > 0 com custos 1,5×, t ≥ 2,0 (erro por semana); seguem até 5;
  - **confirmação** em 2020 e 2022: critérios v2 com z de Bonferroni pelo número de candidatas, (a) ≥ 150 trades, (e) ≥ 4 das 6 moedas;
  - **validação** uma única vez (dez/2023 a abr/2025, 3 dias de aquecimento): (i) limite da confirmação + validação > 0, (ii) R médio da validação ≥ 0, (iii) acima dos baselines;
  - a régua do M5 em carteira (uma posição cripto por vez) é reportada.
- **O que pode distorcer:**
  - **a favor:** o preenchimento em quedas rápidas (mark com o teto de slippage publicado; na vida real a fila pode ser pior). O estresse de stop "no meio do caminho" mede parte disso;
  - **contra:** o custo de cripto (~0,08% por ida e volta) e eventos raros em anos calmos;
  - **neutro:** eventos simultâneos nas 6 moedas (erro por semana; a carteira aceita uma posição por vez).
- **O que espero:** chance moderada. Há razão econômica e literatura (a reversão de curto prazo remunera quem fornece liquidez), mas cripto é muito disputado por robôs rápidos.
- **Critério de morte:** nenhuma candidata na triagem, ou nenhuma confirmada.
- **Grade aprovada pelo Alison em:** 2026-09-30 ("Pode executar", E043).

### E045 — H12 — resultado — 2026-09-30
- **Arquivos:** `resultados/2026-09-30_H12_v1.md`, a triagem em `..._triagem.csv`, os trades e o log. A triagem rodou em ~12 min.
- **Triagem** (2021 e 2023): **nenhuma das 8 passou.**
  - Todas negativas: de −0,100 a −0,206R, com t de −4,9 a −24,2, e **0 de 6 moedas positivas** em todas.
  - Acerto de 43–64% (alto nas saídas com repique, como esperado), mas as perdas pesam mais.
  - O custo (0,03–0,09R) não explica: antes dele, ainda fica de −0,07 a −0,12R.
- **Diagnóstico** (2021, k = 5, janela de 1 min):
  - **O repique existe no último negócio:** retorno em 30 min, na direção da reversão, a partir do fechamento do evento: **+1,8 bps (BTC, 1.355 eventos) e +26,3 bps (SOL, 1.090 eventos).**
  - **O mark do minuto seguinte está 8,2 bps (BTC) e 16,0 bps (SOL) "atrás"** do último negócio, na direção da reversão.
    - Como a mesa compra "nunca abaixo do mark" (e vende nunca acima), a entrada perde esse pedaço.
    - A partir do mark, o retorno cai para −6,6 bps (BTC) e +9,6 bps (SOL), antes do custo (~8 bps) e dos stops.
  - **Conclusão:** nesta plataforma, a reversão de curto prazo depois de movimentos rápidos é **estruturalmente desfavorecida**. A execução pelo mark come a parte do repique que existia.
- **Cuidado para o futuro:** o mesmo atraso do mark **favorece, no backtest, entradas a favor de movimentos rápidos** em cripto, porque compra-se "barato" depois de uma alta.
  - Resultados de momento de curto prazo em cripto (como o ORB da H4, t = 3,26 contra o acaso no desenvolvimento, morto na validação) podem ter tido essa ajuda. Qualquer teste futuro desse tipo deve medir a contribuição do atraso do mark.
  - Explorar deliberadamente atraso de preço costuma ser proibido pelas regras das mesas. Não é um caminho sem ler as regras e sem o ok do Alison.
- **Variantes:** 8. Total acumulado: **118**. A validação não foi usada.
- **Veredito: a H12 MORRE** na triagem.

### E046 — 2026-09-30 — Vídeo do campeão da Robbins World Cup (Chris Creamer, fluxo de ordens): análise
- **Vídeo:** "Trading WORLD CHAMPION Reveals the Orderflow Strategy That Won the Robbins Cup", canal IQCapital (uma mesa proprietária que vende desafios), 58 min, 1,86 milhão de visualizações.
  - Legenda lida inteira.
  - Imagens em ~10 pontos: é entrevista com **quadro branco**, sem operação ao vivo.
- **O método dele** (NQ/MNQ; primeira 1h30 depois da abertura de NY; gráficos de 5 min, olhando o de 1h e o de 15 min):
  1. **Ambiente:**
     - estrutura do gráfico maior (valor subindo em degraus);
     - **regime de gama das opções (GEX)**: em gama **positiva**, os formadores de mercado vendem as altas e compram as quedas, o que traz mercado travado e muitos rompimentos falsos; em gama **negativa**, eles compram as altas e vendem as quedas, o que amplifica a volatilidade;
     - tudo definido antes da abertura.
  2. **Localização:**
     - "desconto" abaixo da área de valor numa estrutura de alta, pelo Fibonacci de 0,705, 0,788 e 0,886 do último movimento, **fora** da área de valor;
     - de preferência numa região de pouco volume, por onde o preço passou rápido;
     - abaixo do 0,886, não opera.
  3. **Confirmação por fluxo de ordens:**
     - vendedores agressivos (delta negativo, volume concentrado no pavio de baixo) sendo **absorvidos** na zona;
     - espera a **virada de domínio**: compradores agressivos, desequilíbrios de 400% no bid × ask, e uma segunda tentativa vendedora que falha;
     - só então compra, com **stop logo abaixo da falha dos vendedores**.
  - **Alvos e gestão:** topo ou fundo anterior, o POC, ou "voltar para dentro da área de valor"; stop móvel acompanhando a agressão. Resultado típico de **1,5 a 2R**, **acerto de 60–65% e fator de lucro ~1,8** (palavras dele, sem dados mostrados).
  - **Filtros:** só com participação alta (≥ 20 mil contratos de MNQ por candle de 5 min); de 0 a 2 trades por dia; parada rígida depois de 1h30 e depois de 2 perdas seguidas.
  - **Lição principal dele:** a consistência vem de **cortar os trades ruins** ("jogo C": entrar antes da confirmação, perseguir o preço), não de achar mais setups.
  - Na mesa, 1,5R faz sentido porque a razão meta/perda máxima de uma conta de US$ 50 mil é 1,5.
- **Avaliação crítica:**
  - A Robbins Cup é uma competição real e auditada, mas **de 1 mês** e com risco agressivo (quem ganha é o extremo entre muitos participantes).
  - O método é discricionário ("não é receita de bolo").
  - O canal vende desafios de mesa (US$ 9 e US$ 1).
  - Os números de acerto e fator de lucro não são mostrados.
- **O que dá para testar com os nossos dados:**
  - **Regime de gama como filtro** (base: Baltussen et al., 2021, em que o momento intradiário é mais forte com gama negativa). Aplicar à única vantagem real que achamos (H9, índices). Exige um indicador de regime; o mais simples e gratuito é o VIX diário (Stooq ou CBOE), com o ok do Alison.
  - **Desconto + absorção em cripto:** os nossos dados da Binance têm o **volume agressor de compra por minuto** (`taker_buy_volume`), então dá para calcular o **delta** e detectar "venda agressiva absorvida + virada de domínio" em candles de 5 min. Nos índices não dá: a Dukascopy não tem o lado agressor.
  - **Filtros de participação e de horário:** operar só com volume alto; parar depois de 2 perdas.
- **Nenhum teste rodou.** As frentes antigas (tendência diversificada e modelo de sinais) passam a vir depois dessas duas.
- **Decisão do Alison (2026-09-30, "Pode prosseguir"):** baixar o VIX e preparar a H13 nova (filtro de regime na H9) e a H14 nova (desconto + absorção em cripto).

### E047 — 2026-09-30 — VIX baixado; H13 e H14 preparadas (nada rodou)
- **VIX diário** da CBOE (`dados/cboe/VIX_History.csv`, 1990 até hoje; fonte oficial e grátis).
  - Lido só por `dados.carregar_vix(fim)`, com a **mesma trava do cofre** de `carregar()`: nada a partir de 2025-05-01 sem ok.
  - Só olhei as estatísticas do desenvolvimento (2015 a nov/2023: mediana 16,8; mínimo 9,1; máximo 82,7).
- **Código:**
  - `src/estrategias/h13_regime_gama.py` e `h14_absorcao.py`;
  - `scripts/lab_h13.py` e `lab_h14.py`;
  - 5 testes novos, incluindo a trava do VIX e "sem olhar o futuro".
  - **146 testes**, todos passando.

### H13 — Momento intradiário só em regime de gama negativa (VIX) — pré-registro — 2026-09-30
- **Família e ideia:** a regra da H9 (área de ruído, Nasdaq-100 e S&P 500, 30 min, pregão de NY), sem mudança, **só nos dias de regime provável de gama negativa**.
  - Base: Baltussen et al. (2021), em que o momento intradiário é mais forte com gama negativa, e a camada "ambiente" do método do campeão da Robbins Cup (E046).
  - É um filtro sobre a única vantagem real achada (H9: +3,2 bps antes dos custos, t = 3,18 contra o acaso, fina demais para os custos).
- **Regime:** VIX de fechamento do pregão anterior (nada do próprio dia).
  - "vix_sobe": VIX ≥ a média de 20 pregões;
  - "vix_alto": VIX ≥ 20.
- **Variantes: 2.** z = 1,96. Sem etapa 2. A saída é a da H9.
- **Baselines:** os da H9, restritos aos dias do regime (aleatório nas mesmas checagens; sempre comprado do E023).
- **Critérios:** v2 no desenvolvimento (2015 a nov/2023), (e) R > 0 nos 2 índices. Confirmação pelo E017, com a validação (nunca usada para a H9) uma única vez e 90 dias de aquecimento. Régua no M5.
- **O que pode distorcer:**
  - **A favor (importante):** o desenvolvimento dos índices já foi visto muitas vezes, e a H9 inteira também. O filtro vem de fora (pesquisa e vídeo), mas escolhe um pedaço de um resultado já conhecido. A proteção real é a validação e o cofre.
  - **Contra:** o VIX é um termômetro indireto do gama (não há dados de opções); com menos dias, há menos trades.
- **O que espero:** a vantagem por trade deve subir nos dias de VIX alto ou subindo. É incerto se chega à régua (~+0,05R com ~2 trades por dia).
- **Grade aprovada pelo Alison em:** 2026-09-30 ("pode").

### H14 — Desconto + absorção em cripto (fluxo de agressão) — pré-registro — 2026-09-30
- **Família e ideia:** as camadas "localização" e "confirmação" do método do campeão (E046), em cripto.
  - Numa estrutura de alta, o preço volta à zona de "desconto" (Fibonacci de 0,705 a 0,886 do último movimento, abaixo da área de valor do dia anterior).
  - Vendedores agressivos são absorvidos (delta negativo com fechamento forte), e em seguida os compradores assumem.
  - Compra na virada; a venda é o espelho.
- **Mercados e dados:** 6 criptos, candles de 5 min, 24h. O delta vem do volume agressor de compra por minuto da Binance (`taker_buy_volume`).
- **Regras** (`src/estrategias/h14_absorcao.py`):
  - estrutura pela EMA de 240 candles contra a de 600 (~EMA 20 e 50 do 1h);
  - movimento de referência: nas 48 velas anteriores, com ≥ 3 ATR(14);
  - área de valor do dia UTC anterior (70% do volume);
  - absorção: delta/volume ≤ −limiar e fechamento na metade de cima;
  - virada nos 3 candles seguintes: candle de alta, delta > 0 e mínima acima da mínima da absorção;
  - stop 0,1 ATR além da absorção; alvo em R; saída por tempo em 4h; aquecimento de 700 candles.
- **Variantes: 4** (limiar do delta ∈ {0,10; 0,20} × alvo ∈ {1,5R; 2R}). z = 2,24. Sem etapa 2.
- **Baselines:**
  - aleatório: mesma frequência, stop de 1,5 × ATR(14), o mesmo alvo e o mesmo tempo;
  - sempre comprado (E023) nos mesmos momentos, com a mesma distância de stop.
- **Critérios:** v2 no desenvolvimento (2020 a nov/2023), (e) ≥ 4 das 6 moedas. Confirmação pelo E017, com a validação uma única vez. Régua no M5.
- **O que pode distorcer:**
  - **Contra:**
    - o fluxo por minuto é bem mais grosso que o "footprint" por preço que o campeão usa;
    - a execução pelo mark: a lição do E045. Aqui é menor, porque a entrada vem depois da virada;
    - o custo de cripto.
  - **A favor:** 2020–2021 em alta favorece compras. O critério (d) desconta.
  - **Diagnóstico planejado:** medir a contribuição do atraso do mark (o cuidado do E045).
- **O que espero:** incerto. É a primeira regra do laboratório que usa o fluxo de agressão, informação que nenhuma das anteriores teve.
- **Grade aprovada pelo Alison em:** 2026-09-30 ("pode").

---

## Modelos

### Pré-registro de hipótese (preencher ANTES de rodar)

```
### Hxx — <nome> — pré-registro — AAAA-MM-DD
- Família e ideia:
- Mercados e tempo gráfico:
- Regra de entrada (codificável):
- Stop (regra) e saídas a testar (S1–S6):
- Parâmetros livres (máx. 3) e grade:
- Número de variantes que serão testadas:
- O que espero e por quê:
- Critério de morte:
- Grade aprovada pelo Alison em:
```

### Resultado de teste

```
### Exxx — Hxx — resultado — AAAA-MM-DD
- Períodos usados (desenvolvimento / validação):
- Variantes rodadas:
- Trades (N), acerto, payoff médio, edge líquido em R:
- Contra os baselines (sempre long / aleatório):
- Simulador (M5): P(passar), mediana de trades até passar, P(quebrar), EV:
- Robustez entre ativos:
- Veredito: segue / morre / ajustar — e por quê:
```

### Abertura do cofre

```
### Exxx — Hxx — abertura do cofre — AAAA-MM-DD
- Autorizada pelo Alison em:
- Resultado no cofre:
- Veredito G1:
```

Trades ao vivo (sandbox e conta real) ficam em `resultados/trades_<ambiente>.csv`, com um resumo semanal aqui.
