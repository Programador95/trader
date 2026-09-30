# PLANO MESTRE — Projeto MESA (nome provisório)

**Versão:** v0.8 — 29/09/2026
**Status:** **projeto em pausa** — F0 a F2 concluídas; F3 testou 7 hipóteses (71 variantes) sem nenhuma aprovada; gasto $0 (seção 13)

> Documento-fonte do projeto. Toda sessão nova (Claude Code ou chat) começa lendo este arquivo e o `LEDGER.md`.
> As regras da plataforma foram verificadas na documentação oficial da MyFundedPerps em 27/09/2026. Reconferir antes de qualquer gasto, porque elas podem mudar.

---

## 0. Em uma frase

Construir um robô em Python que opere a conta **Select de $10k da MyFundedPerps** com regras 100% pré-definidas, validado de graça antes de qualquer gasto, com teto de **uma conta por mês** saindo do próprio bolso.

## 1. Papéis e princípios

- **Alison:** operador e auditor. Aprova hipóteses, decide portões e dá o "go" em qualquer gasto.
- **Claude:** arquiteto, executor e auditor do código.
- **Por que robô:** a experiência anterior no Nasdaq mostrou que o problema foram decisões tomadas no calor (stop longo demais, curto demais, não realizar o alvo querendo mais, operação que inverte). O robô decide tudo antes. Durante um trade não existe decisão humana, só o botão de emergência (kill switch).
- **Sem remendos:** toda ideia nova entra pela fila de hipóteses (seção 6) e pelo ledger. Nada vai direto para o código de produção.

## 2. Função-objetivo: o que estamos maximizando

**Não é taxa de acerto.** É o EV por conta:

```
EV = P(passar) × E(saques líquidos | financiada) − custo da conta
```

- `P(passar)` e `E(saques)` saem do **Simulador da Mesa (M5)**, alimentado pelos trades do backtest.
- A taxa de acerto é **restrição**, não objetivo: acerto maior encurta as sequências de perda, e isso importa com apenas 3% de margem.
- Por que não maximizar acerto: alvo de +0,3% com stop de −3% acerta cerca de 91% dos trades e, depois das taxas, perde dinheiro.
- Referência sem edge: um passeio aleatório entre −3% e +9% passa em cerca de 25% das vezes antes de custos. As taxas derrubam esse número. Todo lucro precisa vir de edge real.

## 3. Regras do jogo (Select $10k), verificadas em 27/09/2026

### Avaliação

| Item | Regra |
|---|---|
| Preço | $40 (promoção de $32 com o código PERPS até ~30/09/2026) |
| Meta | +9% = **$900**, medida na equity (inclui P&L aberto) |
| Drawdown máximo | 3% **estático**, piso em **$9.700**. Não acompanha máximas intradiárias nem de fim de dia. Monitorado **o tempo todo** (intradiário), e **tocar** o piso já quebra a conta |
| Perda diária | $300 (3% do saldo inicial), medida a partir da equity à meia-noite de NY (com o P&L aberto naquele momento). Monitorada **o tempo todo** (intradiário, não no fim do dia); tocar o limite já quebra |
| Preço que mede a equity | o **mark** (preço de referência da plataforma), não o último negócio. O stop dispara pelo último negócio. Verificado em 28/09/2026 (glossário e *Challenges and Rules*) |
| Reset diário | meia-noite de Nova York: **01h de Brasília** agora, **02h** a partir de 01/11/2026 (fim do horário de verão dos EUA) |
| Prazo / dias mínimos | nenhum prazo para passar, **mas** a avaliação falha se ficar **30 dias corridos sem um trade executado** (login, ordem não executada ou cancelada não contam). Verificado em 28/09/2026 |
| Regra de consistência | nenhuma |
| Comissão por execução | cripto **0,03%**; ações/índices/commodities **0,005%**; forex 0,0025% (entrada e saída pagam) |
| Swap (custo de carregar posição) | cripto 0,03%/dia; ações/índices 0,015%/dia; cobrado de hora em hora, long e short |
| Ordem a mercado | executa no melhor preço do book, nunca melhor que o mark; slippage máximo publicado: índices/commodities 0,25 bps (até $100k), cripto de 0,8 bps (mercados com OI > $500M) a 6 bps (OI < $10M) |
| Ordem limitada | executa inteira, exatamente no preço limite, depois que o book atinge o preço em duas checagens seguidas; nunca parcial |
| Stop loss | fica no servidor, dispara no último preço negociado (um negócio basta) e executa a mercado; **não garante** saída antes de estourar uma regra |
| Tamanho da posição | em USD ou unidades do ativo, arredondado ao incremento do mercado. NAS100, SP500, ouro, prata e petróleo também aceitam modo contrato (mini/micro), que é só atalho de tamanho |
| Aprovação | ao atingir a meta, a conta passa e as posições são fechadas automaticamente |
| Robôs | permitidos: a plataforma é anunciada para agentes e algoritmos (API, SDK Python, MCP). O proibido são técnicas de abuso (HFT, latência, exploração de feed/execução, tamanho fora de padrão) |
| Acesso | o robô roda do PC próprio, no Brasil. VPS, VPN ou proxy só depois de confirmar com o suporte (o FAQ tem regras sobre VPN, proxy e IP compartilhado, e várias mesas proíbem VPS) |
| Limites por mercado | alavancagem máxima e teto de exposição por mercado (página *Market Exposure Caps*), além de orçamento horário de abertura por conta e mercado |
| Proibido | gambling de avaliação e tentativas repetidas de risco máximo, contas sacrificiais, HFT, arbitragem de latência, exploração de feed/execução/bug, hedge cruzado, spoofing |
| Ações e índices | os da Hyperliquid (`xyz:`) negociam 24/7, inclusive fim de semana (verificado em 28/09/2026); fora do pregão o preço de referência vem do futuro da CME. **Decisão do Alison (28/09):** o projeto opera índices só no pregão regular de NY (9h30–16h ET) e zera antes do fechamento |
| Mercados de índice | não existe "NAS100": o Nasdaq-100 é o **`xyz:XYZ100`** e o S&P 500 é o `xyz:SP500`, ambos na Hyperliquid (contrato perpétuo, sem vencimento nem rolagem). Lista pública em `GET /v1/markets` |

### Sandbox (o demo oficial)

- Vem **incluído na compra** de qualquer desafio e fica ativo por **90 dias** a partir de cada compra.
- Usa o mesmo motor, os mesmos endpoints e as mesmas regras de avaliação, com dados reais de mercado. As contas podem quebrar.
- As contas são simuladas de **$100.000**, com limite de 10 contas na vida.
- Chaves de teste (`fp_test_...`) funcionam só em `sandbox.myfundedperpetuals.com`. Para ir para a conta real, basta trocar a chave e o host (`developers.myfundedperpetuals.com`), sem outra mudança de código.
- A API é REST, WebSocket ou MCP, com SDK Python, e está **em beta**: pode mudar sem aviso, então o executor precisa ser robusto a erros.

### Conta financiada

- **Também é simulada** ($10k). O único dinheiro real é o saque, pago via Rise.
- Passar na avaliação **não garante** o financiamento: há verificação de identidade, contrato e possível revisão.
- Regras do saque:
  - mínimo de **$100 bruto**;
  - **um pedido por dia** (reset à meia-noite de Nova York);
  - split **padrão de 80%**; o de **90%** é um adicional pago que **só pode ser escolhido no checkout** e não pode ser incluído depois (verificado em 28/09/2026);
  - parte do saque pode ser convertida em crédito da loja com bônus de 10% (expira em 45 dias; não vira dinheiro);
  - sem consistência, sem dias mínimos e sem teto;
  - a conta precisa estar **zerada** (sem posição nem ordem);
  - revisão manual, com a operação travada enquanto o pedido está pendente; a análise pode demorar mais que um dia.
- **Histórico da mesa:** os saques da MyFundedPerps só foram abertos em **11/09/2026**, então ainda não há histórico de pagamentos. Ela é do grupo MyFunded (a MyFundedFutures, de futuros, tem bom histórico de pagamentos).
- **Inatividade na financiada:** a conta é **encerrada** se ficar **7 dias corridos sem um trade executado**. A estratégia (ou o robô) precisa operar pelo menos uma vez por semana; o M5 e o M7 levam isso em conta.
- **Trava do piso:** no primeiro saque, o piso passa de $9.700 para **$10.000, para sempre**. O lucro que você deixar retido vira colchão e **não pode ser sacado** em ciclos seguintes.
- **Na prática:** os primeiros ~$300 de lucro na financiada reconstroem o colchão, e o excedente é sacável. A política de saque (manter o colchão ou sacar tudo) será otimizada no M5.
- **Imposto:** o saque é rendimento do exterior. Confirmar o tratamento (carnê-leão) com um contador.

## 4. Arquitetura: um sistema, oito módulos

```
 [M1 Dados] ──► [M3 Estratégias]  ──► [M4 Risco]
                 (só geram SINAL)      (decide TAMANHO e STOP)
                        │
                        ▼
               [M2 Motor de backtest]   ← custos e regras da mesa embutidos
                        │
                        ▼
               [M5 Simulador da Mesa]   → P(passar), E(saques), EV
                        │
                        ▼
 [M7 Guardião] ──► [M6 Executor: papel → sandbox → conta real]
                        │
                        ▼
                   [M8 Ledger]
```

### Princípios de arquitetura

1. **Um caminho só.** O mesmo código de sinal e risco roda no backtest, no sandbox e na conta real. Só o executor muda.
2. **Separação de funções.** A estratégia não sabe de dinheiro, e o risco não sabe de indicador. Cada peça é testável sozinha.
3. **Tudo em % da conta.** Assim, os $100k do sandbox equivalem exatamente à conta de $10k.
4. **Nada entra no robô sem passar pelo ledger.**

### Módulos

- **M1 Dados.** Baixa e guarda o histórico.
  - Cripto: Binance, vários anos, candles de 1m/15m/1h.
  - Conferir pela API da MyFundedPerps qual venue (Binance, Hyperliquid ou Bybit) cada mercado usa, e usar os dados desse venue.
  - Índices intradiários (Nasdaq-100 = `xyz:XYZ100`, SP500 = `xyz:SP500`, na Hyperliquid): a API da Hyperliquid só guarda os ~5.000 candles mais recentes, então o histórico longo vem de um proxy do índice (candidato: Dukascopy, 1m desde ~2012). No pregão regular, o preço de referência do XYZ100 é o próprio índice, o que torna o proxy razoável nesse horário. Um coletor guarda os candles da Hyperliquid daqui em diante para comparar com o proxy.
  - Validar buracos, duplicatas e fuso horário.
- **M2 Motor de backtest.** Simula candle a candle sem olhar o futuro. Aplica comissão, swap por hora e slippage pessimista, além das regras da mesa (perda diária e piso). A conta quebra no backtest exatamente como quebraria lá.
- **M3 Estratégias.** Cada estratégia é uma função que recebe dados e devolve sinais (entrar long, entrar short, sair), com **no máximo 3 parâmetros livres**.
- **M4 Risco.**
  - Risco por trade (R) = margem viva ÷ K (ver seção 8).
  - Distância do stop definida por volatilidade (ATR) ou por regra técnica.
  - `tamanho = risco em $ ÷ distância do stop`.
  - Limite de posições simultâneas e de correlação: 10 posições long em cripto são, na prática, uma aposta só.
- **M5 Simulador da Mesa.** Recebe a lista de trades do backtest e roda um Monte Carlo com reamostragem em blocos. Devolve:
  - P(passar) e distribuição de dias até passar;
  - P(quebrar);
  - saques esperados em 90 dias de conta financiada;
  - EV contra o custo da conta;
  - comparação entre políticas de saque.
  - **Teste de sanidade obrigatório:** uma estratégia aleatória sem custos precisa dar cerca de 25% de aprovação, e com custos o número tem que cair.
- **M6 Executor.** Interface única com três implementações: `PapelCaseiro`, `Sandbox` e `Real`. Entre elas, só trocam a chave e o host.
- **M7 Guardião.** Independente da estratégia:
  - para o dia ao consumir a fração máxima da margem viva definida na seção 8;
  - fecha tudo se a margem até o piso ficar crítica;
  - bloqueia novas ordens se os dados estiverem atrasados;
  - oferece um kill switch manual.
- **M8 Ledger.** `LEDGER.md` mais CSVs. Registra:
  - toda hipótese, pré-registrada antes de testar;
  - todo teste, inclusive os que falharam;
  - todo trade do sandbox e da conta real;
  - a contagem acumulada de tentativas.

## 5. Protocolo de pesquisa (anti-autoengano)

1. **Divisão dos dados em três partes:** desenvolvimento (~60%), validação (~20%) e **cofre** (~20% mais recente). O cofre é aberto **uma única vez** por hipótese finalista.
2. **Walk-forward** dentro das partes de desenvolvimento e validação.
3. **Baselines obrigatórios:** "sempre long" e entradas aleatórias com as mesmas saídas. Estratégia que não bate os dois está morta. Desde 29/09/2026 (E023), o "sempre long" das hipóteses novas é "comprar nos mesmos momentos da estratégia", o que isola a informação da direção. Onde a lógica de stop não permitir essa definição, vale o baseline próprio pré-registrado.
4. **No máximo 3 parâmetros livres.** Preferir regiões estáveis, em que os vizinhos também funcionam, a um pico isolado.
5. **Critério estatístico (v0.7, substitui "N ≥ 100"):** o edge líquido médio por trade (em R) só conta se o **limite pessimista** do intervalo de confiança for **maior que zero**. Piso de **300 trades** no backtest; 30 ou mais no forward (G2). Referência: com trades de +2R/−1R e 40% de acerto, 100 trades dão margem de erro de ±0,29R; confirmar +0,15R exige ~370 trades e +0,05R, ~3.300. Um edge forte se prova com menos trades; um fraco precisa de milhares. O M5 usa o limite pessimista, não a média.
6. **Custos sempre ligados,** com slippage pessimista. Cenário de estresse com custos 1,5×.
7. **Toda variante testada entra no ledger, e a barra sobe com elas:** o nível de confiança do item 5 é corrigido pelo número de variantes da hipótese (Bonferroni: significância de 5% dividida pelo número de variantes). Exemplo: 1 variante exige z ≥ 1,64; 20 variantes, z ≥ 2,88; 100 variantes, z ≥ 3,29.
8. **Hipótese se escreve antes de rodar:** o que se espera, por quê, e o critério de morte.
8a. **Confirmação na validação (desde 29/09/2026, E017):** a sobrevivente se confirma se (i) o limite pessimista do conjunto desenvolvimento + validação for > 0 com a mesma barra de Bonferroni da hipótese; (ii) a validação sozinha tiver R médio ≥ 0; e (iii) o conjunto ficar acima dos dois baselines. A validação roda uma única vez; o cofre segue como teste final.
9. **Grade pré-registrada.** Ativos, horários, estratégias e saídas testados são definidos antes de rodar, e toda combinação conta como tentativa no ledger. Testar é de graça em dinheiro, mas cada combinação a mais aumenta a chance de achar coincidência.
10. **Robustez entre ativos.** Estratégia que só funciona num ativo é suspeita; a que funciona em vários ativos parecidos, com os mesmos parâmetros, é candidata.

## 6. Fila de hipóteses (v0.1)

| ID | Família | Ideia | Mercados iniciais | Por quê |
|---|---|---|---|---|
| H1 | Tendência | Cruzamento de MACD com regras clássicas da literatura e parâmetros fixos (não há regras do print; ver `resultados/2026-09-28_pesquisa_macd.md`) | BTC, ETH, SOL, Nasdaq-100 (XYZ100), SP500 | Indicador mais usado; régua barata para a família de tendência. A literatura prevê que o cruzamento puro morre com os custos |
| H2 | Tendência | Donchian: rompimento da máxima ou mínima de N períodos | idem | Família clássica, 1 ou 2 parâmetros |
| H3 | Tendência | Supertrend (baseado em ATR) | idem | Tendência com stop embutido |
| H4 | Horário | Rompimento da faixa de abertura de NY (ORB): faixa dos primeiros X minutos, entrada no rompimento | Nasdaq-100 (XYZ100)/SP500 e BTC/ETH | Observação do Alison no Nasdaq: arrancadas fortes nos primeiros minutos |
| H5 | Reversão | RSI ou Bollinger em extremos, só em regime lateral | BTC, ETH, índices | Complementa tendência, com acerto tipicamente maior |
| FR | Filtro | Regime de mercado (média de 200, ADX) aplicado sobre H1–H5 | — | Evita operar tendência em mercado lateral |
| FV | Filtro | Volume acima da média no rompimento | — | Confirma força do movimento |

- **Indicadores são ingredientes.** ATR serve para stop e tamanho, volume para confirmação, VWAP como referência intradiária da H4. Nenhum indicador entra sem um papel definido.
- **Ordem de execução:** H1 → H2 (as duas testadas e mortas em 29/09/2026) → **H4 → H5 → H3** (ordem alterada pelo Alison em 29/09, E017). Os filtros só entram depois que alguma família sobreviver.
- **Horários da abertura de NY:** 9h30 ET, que é **10h30 de Brasília** agora e **11h30** a partir de 01/11/2026.

## 7. Saídas (stop e alvo): uma dimensão testada à parte

Toda estratégia sobrevivente é testada com o mesmo cardápio de saídas:

| ID | Saída |
|---|---|
| S1 | Stop e alvo fixos em % |
| S2 | Stop por ATR com alvo fixo em múltiplos do risco (1R, 1,5R, 2R) |
| S3 | Stop por ATR com saída móvel (trailing): deixa o lucro correr |
| S4 | Saída por tempo (N candles ou fim da sessão) |
| S5 | Alvo técnico definido por regra (ex.: máxima/mínima dos últimos N candles, banda oposta do Donchian, VWAP) |
| S6 | Stop movido a favor por regra: para o empate ou para +X R depois de o trade andar +Y R (ex.: empate após +1R) |

- **"Técnico" precisa virar regra.** Stop e alvo técnicos só entram no robô se puderem ser escritos como código (ex.: mínima dos últimos N candles, 1,5 × ATR). "Suporte" ou "resistência" sem definição objetiva não é testável.

- As saídas são comparadas por **EV e P(passar)**, com taxa de acerto e maior sequência de perdas ao lado.
- **Regras fixas, que não se testam:**
  - o risco em dólar é definido antes da entrada;
  - o stop **nunca** anda contra a posição; a favor, só por regra pré-definida (S3 ou S6), nunca no calor;
  - não existe intervenção manual durante o trade.

## 8. Risco: tudo medido pela margem até o piso (corrigido na v0.2)

A v0.1 tinha números errados: guardião diário de 1,5% e risco fixo de $25–50 por trade. Com piso estático de 3%, dois dias ruins de 1,5% matariam a conta, e $50 por trade dá só 6 perdas seguidas de margem. O único orçamento que importa é a distância até o piso.

- **Stop, risco e tamanho são três decisões separadas:**
  - o **stop** fica onde a ideia do trade está errada (regra técnica codificável);
  - o **risco R** é quanto se perde se o stop for atingido;
  - o **tamanho** liga os dois: `tamanho = R ÷ distância do stop`.
  Nunca se escolhe o stop pelo valor em dólar: stop por dólar cai no ruído (curto demais) ou fica largo demais. Como o tamanho é contínuo (USD ou unidades), qualquer stop técnico cabe no R escolhido.
  - Exemplo: R = $20 e stop técnico a 1,5% do preço dão posição de ~$1.333; stop a 0,4% dá ~$5.000.
- **Faixa permitida de distância do stop** [d_min, d_max], testada no laboratório. O robô só entra quando o stop técnico cai dentro dela:
  - **d_min:** abaixo disso a posição fica grande demais, as taxas passam do limite em R (ex.: 0,1R) ou a alavancagem/exposição estoura o teto do mercado;
  - **d_max:** acima disso o trade exige um movimento grande demais para o alvo, ou a posição fica abaixo do incremento mínimo.
- **Sem contratos inteiros.** O robô dimensiona em USD/unidades. No NAS100, 1 mini (NQ) = $20/ponto: um stop de 15 pontos já consome os $300 de margem. 1 micro (MNQ) = $2/ponto é viável, mas discreto demais para acertar o R exato.
- **Margem viva** = equity − $9.700 (antes do 1º saque) ou equity − $10.000 (depois). Começa em $300 e cresce com o lucro.
- **Risco por trade (R)** = margem viva ÷ K, em que K é o número de perdas seguidas que a conta aguenta. K é escolhido pelo M5 a partir do edge medido, estimado de forma conservadora, e não por achismo. Exemplo: K = 12 dá R = $25 no início.
- Com R atrelado à margem, o risco sobe quando a conta ganha e cai sozinho quando perde. O M5 compara esse modelo com R fixo.
- **Guardião diário** = fração da margem viva (ex.: no máximo 1/4 por dia), nunca um % fixo da conta. Se R ficar abaixo do tamanho mínimo de ordem, o robô para.
- A perda diária de 3% da mesa quase nunca é a restrição ativa no começo; o piso estático é.
- Posições correlacionadas contam como uma só no risco. Limite de posições simultâneas calibrado no M5.
- **Sem alvo diário.** O alvo é $900 no total, sem prazo. O robô só opera quando há sinal; meta diária força trade ruim.
- **Risco dependente do estado da conta (lógica de satélite/ICM).** Na avaliação, o objetivo é chegar a +9% antes de −3%, não maximizar lucro: perto da meta, proteger a margem pode valer mais que o EV do trade. O M5 testa políticas como reduzir R ou travar lucro quando a equity se aproxima da meta, e mede o efeito em P(passar). Na conta financiada o jogo vira "cash game" e volta a valer o EV puro.
- **Stop em lucro não é lucro garantido.** O stop executa a mercado; em salto de preço por notícia, a saída pode ser bem pior que o nível do stop.
- **Taxas medidas em R.** Custo de ida e volta ≈ 0,06% ÷ distância do stop em cripto, ou ≈ 0,01% ÷ distância do stop em ações e índices. Em cripto, stop de 2% custa ~0,03R; stop de 0,5%, ~0,12R; stop de 0,2%, ~0,3R. O custo em R não depende do tamanho da posição, só da distância do stop. Swap de cripto soma ~0,03% ÷ distância do stop por dia de posição aberta.
- **Trade-off central** (simulação de 27/09/2026, 4 mil contas por célula; modelo de 40% de acerto, +2R/−1R, com o edge líquido indicado depois das taxas):

| Edge líquido por trade | R = $50 | R = $25 | R = $10 |
|---|---|---|---|
| +0,15R | 63% (mediana ~64 trades) | 83% (~190) | 99% (~560) |
| +0,05R | 38% | 50% | 76% |
| zero | 23% | 25% | 25% |
| −0,05R | 17% | 9% | 1% |

  Leitura: com edge real, risco menor aproxima a aprovação de 100%, ao custo de mais trades e mais tempo. Sem edge, dá ~25% qualquer que seja o risco. Com edge negativo, risco pequeno só prolonga a morte. É gestão de banca de poker: com vantagem, stakes baixos em relação à banca; sem vantagem, stake nenhum salva.
- **Nunca** tentar a meta em poucos trades grandes. Além de ser -EV, é o padrão de "gambling de avaliação" que a mesa proíbe.

## 9. Fases e portões

| Fase | O quê | Custo | Portão para avançar |
|---|---|---|---|
| F0 | Setup: Claude Code, pasta do projeto, ambiente Python, este plano e o LEDGER | $0 | Ambiente rodando |
| F1 | Dados (M1) | $0 | Histórico baixado e validado |
| F2 | Motor, Risco e Simulador (M2, M4, M5) | $0 | Sanidade: aleatório sem custo dá ~25% de aprovação; com custo, cai |
| F3 | Laboratório: H1–H5 × S1–S6 | $0 | **G1** (redação aprovada em 29/09/2026): a finalista passa no critério estatístico da seção 5 no desenvolvimento e na confirmação (desenvolvimento + validação somados, com a validação sozinha não negativa; E017); bate os dois baselines no cofre; tem EV positivo **e** acima do EV do baseline aleatório, mesmo com custos 1,5×; e cumpre as regras operacionais (pelo menos 1 trade por semana, por causa da inatividade de 7 dias da financiada, e respeito à alavancagem máxima e ao incremento mínimo de cada mercado) |
| F4 | Compra da conta; robô ao vivo no sandbox (M6, M7) | $32–40 | **G2:** 30 trades ou mais e 4 semanas ou mais, com resultado dentro da faixa de 80% prevista pelo M5 |
| F5 | Avaliação de 10k com o mesmo código | $0 | Passar |
| F6 | Conta financiada com a política de saque definida no M5 | $0 | — |

### O que dá para fazer sem conta e o que exige a conta

- **Sem conta, de graça (F0 a F3):** Claude Code, estrutura do projeto, download do histórico, motor, simulador da mesa, laboratório completo e, se útil, papel caseiro ao vivo com dados públicos.
- **Exige a conta (F4 em diante):** chaves de API (teste e real), sandbox com o motor oficial, limites exatos do mercado para a conta (tamanho máximo, incremento mínimo), avaliação, verificação de identidade, contrato e Rise.
- A compra só acontece depois do G1. Se nada passar no G1, o projeto para com gasto zero.

### Critérios de morte

- **Nenhuma hipótese passa no G1:** o projeto pausa **sem gastar nada**.
- **Sandbox diverge do backtest (G2 falha):** volta para a F3. A conta de 10k pode ficar parada, porque não tem prazo, **desde que** receba um trade executado a cada 30 dias (regra de inatividade).
- **Avaliação quebra:** volta para a F3. Nova conta só no **mês seguinte**, e só depois de um novo G2.
- **Teto do bolso:** uma conta por mês. Contas adicionais só com dinheiro de saque.

## 10. Decisões

### Decididas pelo laboratório (F3), com dados, não por achismo

1. **Mercados:** cripto, índices (Nasdaq-100/XYZ100, SP500; só no pregão regular de NY) ou os dois. Depende também da qualidade do proxy de histórico dos índices na F1.
2. **Horários:** 24h, só abertura de NY, ou janelas específicas.
3. **Tempo gráfico e rotina:** candles de 1h/4h (o robô acorda poucas vezes por dia) ou intradiário (PC ligado no horário de NY). **Preferência do Alison (28/09):** operar todo dia, a qualquer hora, sempre que houver trade com EV positivo; algo como 2 trades por dia seria bom, sem expectativa fixa. O laboratório decide com os dados.
4. **Estratégia e saída:** qual combinação H × S sobrevive ao protocolo da seção 5.
5. **K (tamanho do risco) e guardião diário:** escolhidos pelo M5 com o edge medido.

### Do Alison (não dependem de dados)

6. **Momento da compra:** na promoção (até ~30/09, $32, o que antecipa o início dos 90 dias de sandbox) ou só na F4 ($40). **Decisão do Alison (28/09):** comprar na promoção se o dinheiro entrar. Consequência: a avaliação precisa de um trade executado a cada 30 dias corridos até o robô assumir, ou falha por inatividade.
7. **Split do Select:** 80% padrão; 90% só com adicional pago escolhido no checkout (decidir na hora da compra).
8. **Nome definitivo do projeto.**

## 11. Estrutura do projeto (Claude Code)

```
mesa/
  LEIA_PRIMEIRO.md       ← passos manuais do Alison (instalar e abrir)
  CLAUDE.md              ← como o Claude Code trabalha (lido sozinho a cada sessão)
  PLANO_MESTRE.md        ← este arquivo: o quê e o porquê
  LEDGER.md              ← hipóteses, testes, trades
  config.yaml            ← regras da mesa e parâmetros de risco (fonte única)
  requirements.txt       ← pacotes Python
  .gitignore
  .env.example           ← modelo para as chaves de API (o .env real nunca é versionado)
  scripts/
    setup_ambiente.py    F0: verifica e prepara o ambiente, pulando o que já existe
  dados/                 ← histórico baixado (não versionar)
  src/
    dados.py             M1
    motor.py             M2
    estrategias/         M3 (baselines, h1_macd.py, h2_donchian.py, ...)
    risco.py             M4
    simulador_mesa.py    M5
    executor/            M6 (papel.py, sandbox.py, real.py)
    guardiao.py          M7
  testes/                ← testes de cada módulo
  resultados/            ← relatórios de cada fase, backtests e simulações
```

- Chaves de API ficam só no `.env`, **nunca** no código, no ledger ou no chat.
- Toda regra da mesa vive no `config.yaml`. Se a mesa mudar uma regra, muda-se num lugar só.
- O `CLAUDE.md` tem o passo a passo executável de cada fase e as regras de trabalho do Claude Code.

## 12. Como retomar

- **Claude Code:** o `CLAUDE.md` é carregado automaticamente em toda sessão e manda ler o estado atual (seção 13) e o LEDGER antes de qualquer coisa. Basta abrir o projeto e dizer o que quer.
- **Chat comum:** anexar este arquivo e o LEDGER.md e dizer: "Estamos na fase X. Próximo passo: Y."

Ao fim de cada sessão, atualizar a seção 13 e o LEDGER.

### Primeira mensagem no Claude Code

> Vamos começar. Execute a Fase 0.

## 13. Estado atual

- **Fase: F3, aguardando a decisão do Alison sobre o rumo** (LEDGER E045). Nenhuma hipótese chegou ao G1. **Gasto: $0.** A conta não foi comprada (E023), e o cofre segue fechado (E030).
- **Concluído:**
  - F0 (ambiente, versões travadas);
  - F1 (dados: 6 criptos da Binance com último negócio e mark, 2020–2026; Nasdaq-100 e S&P 500 da Dukascopy, 2012–2026; calendário da NYSE);
  - F2 (motor, risco, simulador; sanidade 24,7%). Em 29/09, o motor ganhou a saída parcial (E032).
- **F1 ampliada (E035–E037):** ouro, prata, Brent, euro e iene (Dukascopy, jan/2020 a abr/2025; cofre não baixado), checados em `resultados/2026-09-29_H10_dados.md`.
- **F3 (laboratório):** 14 hipóteses e 118 variantes, nenhuma com G1. Detalhes e números no LEDGER (E012 a E045).
  - H1 e H1b (MACD), H2 (Donchian) e H3 (Supertrend): em cripto, as entradas de tendência não batem entradas aleatórias com a mesma saída; o ganho do período 2020–2023 veio da saída de tendência e da alta do mercado.
  - H4 e H4b (abertura de NY): índices sem efeito; em cripto, a vantagem da entrada sumiu fora da amostra.
  - H5 (reversão à média): negativa em tudo.
  - Vídeos do YouTube indicados pelo Alison:
    - **H6** (DEMA + Supertrend): cripto morta. O Nasdaq, zerando no fim do pregão, teve uma vantagem pequena (+0,05R), mas empatou com o acaso na validação sozinha, e o EV é negativo em todo K.
    - **H7** (rompimento com candle de momento): poucos trades e negativa.
    - **H8** (MACD + EMA 200, "86%"): −0,27R em cripto, pior que o acaso; nula no Nasdaq.
  - TradeX Labs e Ross Cameron: nada novo testável (E028, E029).
  - **H9** (momento intradiário com área de ruído, base acadêmica):
    - **o primeiro efeito real:** +3,2 bps por trade antes dos custos, como na pesquisa; bate o acaso com t = 3,18; positivo nos 2 índices em 2020–2023;
    - fino demais para a mesa: +0,008R líquido, com o limite do (b) em −0,002 (morreu);
    - EV no M5 perto de zero (+$9 só no K = 2, sem estresse);
    - replicação externa relata colapso em 2025–2026 (E033, E034).
  - **H10** (a regra da H9 em ouro, prata, Brent, euro e iene, no pregão de NY): sem efeito nenhum, nem antes dos custos (E037).
  - **H11** (funil de 29 padrões intradiários nos índices: gap com pullback, momento, VWAP, falso rompimento, MACD diário): nenhum passou na triagem.
    - O melhor foi o gap com pullback do Alison, segurando até o fechamento: t = 1,41, mas todo o resultado vem de 10 dias de tendência.
    - Trades curtos com stop apertado pagam 13–15% do risco em custo (E041).
  - Cofre intacto (0 aberturas); validação usada 3 vezes (H2, H4b e H6).
- **Protocolo vigente** para hipóteses novas:
  - critérios v2 (E020);
  - confirmação desenvolvimento + validação (E017);
  - baseline "sempre comprado" nos mesmos momentos da estratégia (E023);
  - seção obrigatória "O que pode distorcer este teste" em toda proposta (E019);
  - regras de estratégia tiradas da fonte primária (E025).
- **Régua da mesa (E042, `resultados/2026-09-30_meta_ev.md`):**
  - EV > 0 exige, com os trades bem-comportados, ~+0,20R líquido por trade com 2 trades por semana, ~+0,07R com 5, **~+0,05R com 10 (2 por dia)** e ~+0,02R com 20–40 no estilo "scalp" (acerto alto).
  - Com 1 trade por semana, nada serve.
  - Sem vantagem (μ = 0), o EV é sempre negativo.
  - Com trades reais (caudas, sequências), a meta é mais alta.
- **Plano de pesquisa em execução** (E043, `resultados/2026-09-30_plano_de_pesquisa.md`):
  - frente 0 pronta: simulador de carteira e funil reutilizável (E044);
  - frente 1, **H12** (reversão depois de liquidação em cascata em cripto): morta. A execução pelo mark come o repique (E045);
  - **próximo:**
    - frente 2, **H13** (tendência diversificada em dezenas de mercados, diário), que precisa de dados diários novos;
    - frente 3, **H14** (modelo que combina sinais), que precisa do pacote scikit-learn.
    - As duas aguardam o ok do Alison.
  - Os dados de forward (a partir de set/2026) vão se acumulando como amostra nova.
- **Pendências antes de qualquer G1:**
  - ~~simulador de carteira com o teto de risco por grupo de correlação~~ (feito, E044);
  - comparação do proxy dos índices com o venue real (coletor: `.venv\Scripts\python.exe -m src.dados hyperliquid`, opcional durante a pausa);
  - tetos de exposição e orçamento horário (só com a conta).
- **Operacional (se chegar à F4):**
  - desligar a suspensão automática do Windows para o robô;
  - perguntas ao suporte (E007): inatividade durante a análise de saque, VPS, pausa dos índices fora do pregão.

## 14. Custos do projeto

- **Dados históricos** (Binance e similares): grátis, sem cadastro.
- **Python e bibliotecas:** grátis.
- **API da MyFundedPerps:** a documentação não menciona cobrança; as chaves são criadas nas configurações da conta, e o sandbox vem incluído na compra.
- **Claude Code:** incluído nas assinaturas Pro e Max do Claude, compartilhando o mesmo limite de uso. Atenção: se existir a variável de ambiente `ANTHROPIC_API_KEY` no PC, o Claude Code cobra pela API em vez de usar a assinatura.
- **Único gasto previsto:** a conta ($32–40), no máximo uma por mês.

## 15. Fontes (consultadas em 27/09/2026)

- Documentação: https://docs.myfundedperpetuals.com/
- Regras: https://docs.myfundedperpetuals.com/guides/challenges-and-rules.md
- Saques: https://docs.myfundedperpetuals.com/guides/challenge-lifecycle/payouts.md
- Taxas: https://docs.myfundedperpetuals.com/guides/commissions-and-fees.md
- Sandbox: https://docs.myfundedperpetuals.com/sandbox.md
- Limites de exposição: https://docs.myfundedperpetuals.com/guides/asset-exposure-limits.md
- Índice completo: https://docs.myfundedperpetuals.com/llms.txt

## 16. Histórico de versões

- **v0.1 (27/09/2026):** plano inicial.
- **v0.2 (27/09/2026):** seção 8 reescrita (risco pela margem até o piso, K escolhido pelo M5, sem alvo diário, taxas em R, tabela de trade-off simulada); seção 10 dividida entre decisões do laboratório e do Alison; grade pré-registrada e robustez entre ativos no protocolo; seção de custos.
- **v0.3 (27/09/2026):** stop, risco e tamanho separados, com faixa permitida de distância do stop e dimensionamento em USD (seção 8); saída S5 e regra de que "técnico" precisa ser codificável (seção 7); detalhes de execução da plataforma na seção 3 (slippage publicado, ordem limitada, gatilho do stop, modo contrato, fechamento automático na aprovação); QQQ trocado por NAS100.
- **v0.4 (27/09/2026):** regras sobre robôs e acesso (PC próprio; VPS/VPN só com confirmação do suporte); separação entre o que dá para fazer sem conta e o que exige a conta (seção 9); primeira mensagem para o Claude Code (seção 12); pendências externas no estado atual.
- **v0.5 (27/09/2026):** kit do projeto criado (CLAUDE.md com o passo a passo executável de cada fase, LEIA_PRIMEIRO.md, config.yaml preenchido, LEDGER com modelos, requirements.txt, setup_ambiente.py idempotente); seções 11 e 12 atualizadas.
- **v0.8 (29/09/2026):** F3 executada (H1, H1b, H2, H3, H4, H4b e H5, todas mortas); ordem das hipóteses alterada pelo Alison; nova regra de confirmação na validação (seção 5, item 8a); critérios v2 e baseline "sempre comprado" nos mesmos momentos da estratégia (seção 5, item 3); nova redação do G1; seção 13 reescrita com o projeto em pausa.
- **v0.7 (28/09/2026):** mercados reais da plataforma (não existe NAS100: Nasdaq-100 = XYZ100 na Hyperliquid; índices negociam 24/7); índices só no pregão regular de NY (decisão do Alison); regra de inatividade (30 dias na avaliação, 7 na financiada); critério estatístico no lugar de "N ≥ 100", com a barra corrigida pelo número de variantes; H1 sem o print, com regras de MACD da literatura; decisão de compra na promoção.
- **v0.6 (28/09/2026):** saída S6 (stop movido a favor por regra); regra de que o stop só anda a favor por regra pré-definida; risco dependente do estado da conta (lógica de satélite/ICM) avaliado no M5; aviso de que stop em lucro pode sair pior em salto de preço.
