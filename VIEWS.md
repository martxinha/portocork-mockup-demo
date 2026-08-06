# Portocork Mockup — Lógica e Restrições por View

Complementa `CONVENTIONS.md` (regras transversais). Aqui descreve-se,
página a página, de onde vêm os dados, que filtros existem e que regras de
negócio/visuais são específicas dessa view. Nomes de função/variável
referem-se a `settings-factory-calendar.html`.

## Homepage (`home`)

- **Fontes de Dados**: lista estática de timestamps de sincronização
  (`DATA_SOURCES`) — só protótipo, sem lógica real.
- **Acompanhamento Geral**: "Cumprimento Oferta" é um valor fixo; "Cumprimento
  Produção (Calibre · Classe · Lavação)" (`CUMPRIMENTO_PRODUCAO`) usa sempre
  os 4 tipos de lavação canónicos — `N101`, `Light`, `Prelight`, `Nature`
  (nunca "Natural/Clássica/Rosé").
- **Alertas** (`HOME_ALERTS`): lista própria, independente da Gestão de
  Alertas — ao adicionar/remover um tipo de alerta no site, verificar
  também aqui (já aconteceu ficar desatualizado, ex.: "Artigos sem Lavação
  Definida" removido da Gestão de Alertas mas esquecido aqui).
- **Stocks vs Objetivos**: cartões estáticos, sem filtros.

## Macro Planeamento > Encomendas por Planear (`encomendasPlanear`)

- Tabela **partilhada** entre Distribuição/NDTech — `ENCOMENDAS_ROWS` não é
  filtrada por `state.globalFlow` (só a coluna "Classe" é `flow:'ndtech'`
  only).
- Filtros: Estado (Cumpre/Standby/Não Cumpre), Calibre, Classe — derivados
  dinamicamente dos dados (`encomendaCalibre`/`encomendaClasse`, regex sobre
  o texto do material).
- **Destaque de linha em âmbar só quando `sto.alert && sto.pct >= 100`**
  (badge de STO fica vermelho) — não em todas as linhas com `sto.alert`
  (esse era um bug corrigido nesta sessão: quase todas as linhas reais
  tinham `alert:true`, o que sobre-destacava a tabela).
- Coluna "Alertas": `encomendaAlertas()` combina o campo `r.alertas` com dois
  alertas computados automaticamente — `criacaoSto` (se `sto.alert`) e
  `clientePremium` (se o Sold To/Ship To corresponder a um cliente com
  `tipo:'Premium'` em `CLIENTES_ROWS`). Os ícones vêm de `ENC_ALERTA_DEFS`,
  que tem de estar em sincronia com `GESTAO_ALERTAS_ROWS`.
- Botão "Balanço de Stock" por linha abre o modal genérico
  `renderBalancoStockBody`.

## Macro Planeamento > Carteira de Encomendas (`carteiraEncomendas`)

- Também **partilhada** entre fluxos (`CARTEIRA_ROWS`). O que muda por
  fluxo é por-linha: `processoAtual` vs `processoAtualNdtech`,
  `alertas` vs `alertasNdtech` (ver `carteiraProcessoAtual`/
  `carteiraAlertasHtml`).
- **Destaque de linha em âmbar quando a linha tem qualquer alerta**
  (`temAlerta`, olhando para o array certo consoante o fluxo).
- Ícones de alerta (`CARTEIRA_ALERTA_DEFS`) têm de usar o mesmo ícone que
  `GESTAO_ALERTAS_ROWS` para o mesmo conceito — nunca duplicar ícone entre
  dois tipos de alerta diferentes. Inclui `orientacao` (ver "Alerta de
  Orientação" abaixo), com o mesmo ícone (`orientacao`) em `ENC_ALERTA_DEFS`
  e `CARTEIRA_ALERTA_DEFS`.
- Coluna "Status" (bolinha verde/vermelha) = `statusOk`, independente de
  `qtdFalta` (podem não coincidir — é assim nos dados reais também).
- Colunas visíveis configuráveis via dropdown (`carteiraColVisible`).
- **Três tabs** (`CARTEIRA_TABS`/`carteiraState.tab`): **Encomendas**,
  **Amostras**, **Provisionais** — classificação via `carteiraTipo(r)`
  (ordem fixa: `motivoOrdem === 'ZP3'` → Provisionais; OV `1425*` ou
  `carteiraAmostraManual[ov]` → Amostras; resto → Encomendas). Amostras e
  Provisionais ficam **fora do sequenciamento e da capacidade**
  (`carteiraForaDoPlano`), mas mantêm data desejada/alertas — a data
  continua a contar para risco de incumprimento. Colunas de execução
  (`plano: true`: OF, Plan, colunas por processo, Processo Atual) não
  aparecem nessas duas tabs.
  - Linhas OV `1425*`/`motivoOrdem === 'ZP3'` vêm classificadas pelo SAP
    (`carteiraTipoAutomatico`) e não se assinalam/retiram à mão. Outras
    amostras assinalam-se manualmente em Encomendas (botão "Assinalar como
    Amostra" / "Retirar de Amostras" — ausente na tab Provisionais).
  - Coluna "Motivo Ordem" (`motivoOrdem`, desligada por defeito em
    `carteiraColVisible`) é só para auditoria — mostra `ZP3`, `1425*` ou
    "Manual" conforme a origem da classificação.
- Coluna **"Controlo de Qualidade"** (`cq`, `CQ_DEFS`/`CQ_ORDEM`): resultado
  do laboratório por linha (`na`/`pendente`/`ok`/`naoOk`), distinto do
  requisito de CQ do Plano de Produção e do "Status" (cumprimento de data).
  `naoOk` acende automaticamente o alerta `laboratorio` existente em vez de
  introduzir ícone novo (`carteiraAlertasHtml`).
- **Alerta de Orientação** (`orientacao`, só Distribuição —
  `temAlertaOrientacao`): marca sem orientação definida (corpo/topo/ambos)
  — sem ela não se calcula a cadência do laser. Fonte no protótipo:
  `ORIENTACAO_POR_DEFINIR` (mapa por OV), presente tanto em Encomendas por
  Planear como em Carteira.

## Macro Planeamento > Carga Capacidade (`cargaCapacidade`)

- Duas visualizações: **Tabela Semanal** (`renderCapTable`, semanas em
  linha, centros em coluna, célula com %/quantidade/cumulativo) e
  **Visão em Gráfico** (`renderCapPorCentroTable`, sparkline por centro).
- Ordem dos centros = `CAPACITY_GROUPS` / `CAPACITY_GROUPS_NDTECH` —
  ordenados do **último processo do fluxo para o primeiro** (ex.
  Distribuição: Embalagem → Tratamento → Marcação Tinta/Laser/Fogo →
  Pesagem → Escolha). Não inverter sem pedido explícito.
- Coluna "Carga Atual" (visão por centro) mostra por baixo do valor atual
  uma única percentagem entre parênteses — o valor que estava **planeado**
  para esse centro (`capacityPlanned(groupIdx)`), não um "objetivo"/banda.
- **Limite de sobrecarga é 90%, não 100%** — `capacityColor()` e as cores
  das barras do sparkline (`renderCapSparkline`) ficam a vermelho a partir
  de `pct > 90`; a linha de referência tracejada e a legenda seguem o
  mesmo limite ("80–90%" azul escuro, "> 90% (sobrecarga)" vermelho,
  "Limite 90%"). Não voltar a usar 100% como limite sem pedido explícito.
- **Sem coluna de Ações** na tabela semanal (removida — não voltar a
  adicionar).
- `capState.view` alterna entre `'tabela'` e `'grafico'`; datas de
  início/fim da tabela semanal são selects independentes
  (`capInicioSelect`/`capFimSelect`).

## Sequenciamento (`sequenciamento`)

- Lista (`renderSeqList`) ou kanban (`renderSeqKanban`) das ordens
  agrupadas por centro de trabalho (`seqCentro`).
- Drag-and-drop para reordenar só é permitido quando **não há filtros
  aplicados** (`seqCanDrag()`) — com filtro ativo a ordem relativa ficaria
  ambígua, por isso a app mostra um aviso e desativa o arraste.
- Linhas com `stockOk === false` aparecem semitransparentes (falta de
  stock de consumo bloqueia visualmente a prioridade, mas não impede
  reordenar).

## Sequenciamento / Carga Capacidade — Marcação Laser (`CAP_LASER_TIPOS`)

- Encomendas de marcação a laser (Distribuição) têm um tipo de orientação —
  **Corpo** (cadência 14), **Topo** (cadência 12) ou **Corpo e topo**
  (cadência 8, a mais lenta) — que muda a cadência da máquina e por isso a
  carga/sequenciamento. No protótipo `capLaserTipo(ov)` atribui o tipo de
  forma pseudo-aleatória estável por OV (maioria corpo+topo, topo-só é
  minoria) — substituir pelos campos reais do ZPLAN. Liga-se ao mesmo
  conceito do "Alerta de Orientação" da Carteira/Encomendas por Planear:
  sem orientação definida, esta cadência não se pode calcular.

## Acompanhamento > Report de Planeamento (`reportPlaneamento`)

- Nova página sob o menu "Acompanhamento" (ao lado de "Acompanhamento da
  Produção"). Mostra, por OV/Item de `ENCOMENDAS_ROWS`, o histórico de
  planeamento **do 1.º planeamento ao último sem Stand By**
  (`resumoPlaneamento`/`renderPlanReportBody`): datas de 1.º e último
  planeamento sem Stand By, janela em dias, nº de replaneamentos, desvio
  (dias entre a 1ª e a última data prevista) e dias em Stand By.
- Âmbito das linhas (`planReportLinhas`) segue os **filtros ou a seleção
  atualmente aplicados em Encomendas por Planear** (`encState`/
  `encSelected`) — não tem filtros próprios; o cabeçalho mostra
  explicitamente se veio de "Seleção" ou de "Filtros".
- Linha expansível (`planReportExpanded`) mostra o histórico completo de
  planeamentos (data, utilizador, prevista, balanço, estado) por OV.
- Botão "Exportar Excel" é só protótipo (`showToast`, sem exportação real).

## Acompanhamento > Controlo de Qualidade (`controloQualidade`)

- Página dedicada ao laboratório de qualidade — fila de trabalho sobre a
  **mesma fonte de dados e o mesmo estado da Carteira de Encomendas**
  (`carteiraRows()`/`carteiraCq`/`CQ_DEFS`/`CQ_ORDEM`), não uma lista
  paralela: mudar o estado aqui ou na Carteira reflete-se em ambos
  (`carteiraCqAvancar`, extraído para ser partilhado pelos dois sítios).
- Filtro "Estado CQ" por defeito em **Pendente** (`qualidadeState.estadoFiltro`
  = `CQ_DEFS.pendente.label`) — é a fila do que falta testar; os outros
  estados (Não aplicável/OK/Não OK) servem para consulta/histórico.
  Filtros adicionais: Calibre, Classe, pesquisa livre.
- Coluna "Tipo" (`carteiraTipoLabel`) mostra Encomendas/Amostras/
  Provisionais para contexto, já que a fila não está limitada à tab
  "Encomendas" da Carteira — o laboratório testa qualquer linha,
  independentemente de estar dentro ou fora do plano.
- Pill de CQ clicável (mesmo componente da Carteira, `carteiraCqPillHtml`)
  — clicar cicla para o próximo estado; sem formulário de detalhe (valor
  medido/observações) nesta primeira versão.

## Acompanhamento da Produção (`producaoAcompanhamento`)

Duas sub-tabs (`PROD_SUB_TABS`), ambas sobre `PLANO_ROWS` **filtrada por
`state.globalFlow`** via `planoRowsFlow()` — ao contrário de Carteira/
Encomendas/STOs, aqui a linha inteira pertence a um único fluxo (`r.flow`),
porque o centro de trabalho real não é partilhado entre Distribuição e
NDTech (excepto Pesagem, `PS001`). Centros válidos por fluxo (ver
`NEC_CADEIA`):

- **Distribuição**: `EMA001` Escolha, `PS001` Pesagem, `MFO001`/`MLS001`/
  `MTI001` Marcação (Fogo/Laser/Tinta), `TRT001`/`TRT002` Tratamento,
  `EMB003` Embalagem.
- **NDTech**: `NDR001`/`NDC001` NDTech, `LAV001` Lavação, `PS001` Pesagem,
  `EET002` Escolha Eletrónica, `EMB002`/`EMB003` Embalagem (`EMB003` também
  é usado pelo NDTech para embalagem via prestação de serviço — confirmado
  em `BD_SP_Carteira` do `Análises_NDTech.xlsx`, coluna "Descritivo de
  Centro de Trabalho" = "Embalagem PA").
- Ao trocar de fluxo, se o filtro de Centro de Trabalho (ou, em Consumos e
  Produções, o de Processo) já selecionado não existir no novo fluxo, a
  página repõe-no para "Todos" em vez de mostrar uma tabela vazia.
- **Bug corrigido**: antes, `PLANO_ROWS` não tinha `flow` por linha e
  mostrava sempre todas as linhas em ambos os fluxos — em NDTech apareciam
  linhas com centros só de Distribuição (`EMA001`, `MFO001`/`MTI001`,
  `TRT001`/`TRT002`), que o NDTech não tem. Corrigido atribuindo `flow` a
  cada linha (a partir da origem real dos dados/artigo) e substituindo os
  centros que não existiam no fluxo correspondente pelo centro correto
  mais próximo.

- **Cumprimento do Plano** (`renderPlanoTab`): uma linha por OV/OF,
  produção vs objetivo por dia da semana, calculados via `genDay()`
  (pseudo-random determinístico a partir de `row.seed`+`row.objetivoBase`
  — não são dados reais dia-a-dia, só o OV/OF/artigo/cliente/processo é
  real). Linhas sem `objetivoBase` aparecem semitransparentes e sem
  cálculo.
  - Filtros: Tipo de Ordem (derivado do processo via `PLANO_TIPO_ORDEM`) e
    Centro de Trabalho (`planoProcessoCentro`, primeira parte do campo
    `processo`, formato `"CODE · Nome"`).
  - Checkbox de **Controlo de Qualidade** por linha: valor por defeito
    calculado uma única vez no arranque via `clienteRequerCQ(cliente,
    processo)` (compara `row.cliente` com `CLIENTES_ROWS`, que só tem ~14
    clientes curados). Clientes reais fora dessa lista ficam sempre por
    defeito **desmarcados** — decisão consciente, não corrigir sem pedido
    explícito (ver `CONVENTIONS.md`). Cor do checkbox marcado é âmbar
    (`--evt-holiday`), não verde — sinaliza "valor por defeito, editável".
- **Consumos e Produções** (`renderConsumosTab`): mesma tabela `PLANO_ROWS`,
  mas mostra produção + variação de consumo/produção por dia
  (`genDelta`), filtrada por Processo (etapa) e Centro de Trabalho.
- **Balanço de Stocks** (`prodState.tab === 'stocks'`, `renderStocksTab`):
  stock atual por material/setor (`STOCK_ROWS`) + gráfico de linhas
  (`renderStocksChart`).

## Gestão de OFs

Sub-secções sob o mesmo menu "Gestão de OFs":

- **OFs por Liberar** (`ofsPorLiberar`, só Distribuição): duas tabs —
  "possíveis" (`!r.alerta`) e todas. Linhas sem alerta ficam com fundo
  verde (`--evt-closure-bg`) a indicar que podem ser liberadas.
- **Criar OFs** (`ofsCriar`): tabela estática de OFs a criar na semana
  atual; botão "Gerar OFs" só aparece em NDTech; **sem botão "Continuar
  Fluxo"** (removido — o modal `continuarFluxoOverlay`/
  `openContinuarFluxoModal` ficou como código morto intencional, não
  reativar sem pedido).
- **Eliminar OFs** — título de secção "Validar Eliminação de OFs"
  (`ofsEliminar`): lista simples `ELIMINAR_ROWS`, **sem abas** (a antiga
  aba "Embalagem"/tabs foi removida porque só havia uma opção — não
  reintroduzir sub-abas sem pedido).
- **Configurações OFs** (`ofsConfiguracoes`): duas tabs — "Regras de
  Impressão OFs" e "Regras de Fecho OFs". Esta última usa um **único**
  conjunto de regras (`INATIVIDADE_FECHO_CONFIG`) — não tem sub-abas
  "Embalagem"/"Semanais" (removidas por não fazerem sentido). Em NDTech o
  rótulo do primeiro switch muda para "Quantidade de Embalagem Produzida".

## STOs

- **STOs** (`stos`): tabela **partilhada** entre fluxos (`STOS_ROWS`, sem
  filtro por `globalFlow`).
  - `estado` tem 4 valores: **Por Planear** / **Planeada** / **Alteração**
    / **Expedido**. Alteração só surge quando o utilizador confirma uma
    alteração de data/quantidade (`btnConfirmStoAltData`) — nunca gerado
    automaticamente a partir dos dados importados.
  - `estadoProducao` (Por Embalar/Em Controlo/Aprovada/Rejeitada) é
    independente do `estado` — substitui a antiga barra de "Progresso".
  - **"Via Verde · Controlo PTK" só existe em Distribuição** — mesmo que
    `controloPtkViaVerde` seja `true` numa linha, só se mostra quando
    `state.globalFlow === 'distribuicao'` (ver `stoEstadoProducaoBadge`).
  - **"OVs Afetadas" só pode ter conteúdo quando `estado === 'Alteração'`**
    — cada entrada tem `{ov, item, dataPlaneada, novaDataPrevista}`; para
    qualquer outro estado o array deve ficar vazio.
  - **"Alteração de Datas"** (coluna, ícone de calendário) só aparece
    (a vermelho) quando `estado === 'Alteração'`; caso contrário mostra
    "—" — não voltar a mostrar sempre o ícone.
  - **"Contactar Fornecedor"** (coluna, ícone de email) só aparece quando
    `estadoProducao` é `emControlo` ou `aprovada`; abre um modal de
    rascunho de email que exige clique explícito em "Validar e Enviar"
    — nunca enviar automaticamente.
  - Coluna "Ações" (ícone de engrenagem) abre um menu de 3 opções
    (Alterar Data / Alterar Quantidade / Eliminar), não um formulário
    combinado.
  - **Sem coluna "Alertas"** (removida).
- **Perfil de Fornecedores** (`perfilFornecedores`): KPIs por fornecedor
  (Lead Time Assumido, LT Médio de Cumprimento, OTIF, % Replaneamentos)
  com histórico de 8 semanas por métrica; filtrado por `globalFlow`
  (fornecedores têm `flow` opcional).

## Alertas (`alertas`)

- Lista simples `ALERTAS_ROWS`, filtrada por `globalFlow` quando a linha
  tem `flow` definido.
- **`tipo` de cada linha tem de corresponder exatamente a uma `descricao`
  em `GESTAO_ALERTAS_ROWS`** — a coluna "Tipo de Alerta" usa
  `gestaoAlertaByDescricao()` para ir buscar ícone/cor, com fallback a
  texto simples se não encontrar correspondência (não deveria acontecer:
  se acontecer, é sinal de que as duas listas divergiram).

## Máquinas (`maquinas`)

- Uma etapa (tab) por processo do fluxo ativo — etapas vêm de
  `maquinasEtapas(flowKey)`, que é só a lista de `etapa` únicos em
  `MAQUINAS_FLOW_ROWS[flowKey]` (Distribuição: `MAQUINAS_DISTRIBUICAO_ROWS`;
  NDTech: `MAQUINAS_NDTECH_ROWS`, já populado com dados reais de
  `Análises_NDTech.xlsx`).
- Em Distribuição, "Marcação Fogo" e "Marcação Tinta" têm tabelas próprias
  (`maquinasFogoTable`/`maquinasTintaTable`, campos diferentes:
  corpo/topo marca+2ª passagem vs cadência/OEE simples) — as outras
  etapas usam a tabela genérica (`maquinasGenericTable`).
- Botão "+ Adicionar Máquina" existe só na tab "Máquinas" de cada view
  (não na view de Turnos — distinção pedida explicitamente).

## Configurações > Operação (`configOperacao`, `CONFIG_OP_TABS`)

Tabs (algumas restritas a um fluxo via `flow`): Compatibilidades,
Parametrizações (NDTech), % Aprovação por Contrato, Critérios de
Liberação de OFs (Distribuição), Prioridades de Planeamento, Múltiplos de
Palete, Período de Congelamento (NDTech). `PERIODO_CONGELAMENTO_CONFIG` é
específico do fluxo NDTech (forma `{comCapacidade, semCapacidade, notas}`,
sem variante Distribuição).

- **Critérios de Liberação de OFs** (`renderCriteriosLiberacao`,
  `CRITERIOS_LIBERACAO_ROWS`): regras avaliadas por ordem — a primeira que
  encaixa decide (`decisao`: Liberar/Não Liberar). Regra dura e sempre
  primeira linha: `tipoEncomenda === 'Provisional'` nunca libera OF. A
  antecedência já vem calculada da data de início de produção da OF (que
  já reflete a quantidade) — não filtrar outra vez por ML aqui.

## Configurações > Recursos (`configRecursos`, `CONFIG_REC_TABS`)

Tabs: Turnos, Setups, Lead Times, Restrições, Centros de Trabalho,
Mapeamento de Depósitos, Parametrizações (`ocupacaoMaxima` — Distribuição
only). Notas:

- **Parametrizações** (tab `ocupacaoMaxima`, `renderOcupacaoMaxima`) tem
  duas tabelas:
  - **Ocupação Máxima** (`OCUPACAO_MAXIMA_ROWS`): limite de carga que o
    planeamento pode ocupar por semana do horizonte, a contar da semana
    corrente N (`semanaLabel`: "N", "N + 1", …). Cada linha aplica-se
    dessa semana em diante; sem regra definida o limite é 100%. Dados de
    protótipo decrescem de 100% (N) a 80% (N+4).
  - **Partição de OFs** (`PARTICAO_OFS_ROWS`): a partir de que quantidade
    uma OF se particiona (ex. `>= 1000` → 2 partições).
- Note: esta tab reutiliza o rótulo "Parametrizações", já usado noutro
  sítio para as tabs de Operação (NDTech) — são conceitos diferentes, não
  confundir ao editar.

- Turnos > Máquinas por grupo — sem botão de adicionar máquina aqui (só
  na view "Máquinas" do menu principal).
- Setups, Restrições, Centros de Trabalho: sempre com colunas separadas
  Centro de Trabalho / Processo (nunca "código · nome" combinado — ver
  `CONVENTIONS.md`).
- Todas as tabelas construídas via `renderStaticTable` têm lápis (editar)
  + lixo (eliminar) na coluna Ações.

## Configurações > Gerais (`configGerais`, `CONFIG_GERAIS_TABS`)

Tabs: Calendário, Calendário de Expedição, **Gestão de Alertas** (lista
mestre — ver `CONVENTIONS.md` e a secção "Alertas" acima), Gestão de
Utilizadores, Clientes, Fornecedores.

- **Clientes**: filtro único "Tipo de Cliente" à esquerda; botão
  "+ Adicionar Cliente" à direita; colunas de Controlo de Qualidade após
  Processo agrupadas por NDTech/Escolha Eletrónica; estes booleans
  (`cqPesagem`, `cqEscolha`, `cqEmbalagem`, `cqNdtech`,
  `cqEscolhaEletronica`) são a fonte da checkbox por defeito em
  Acompanhamento da Produção (`clienteRequerCQ`).
- **Fornecedores**: lista de fornecedores de STO por fluxo
  (`FORNECEDORES_STO_ROWS`), com botão "+ Adicionar Fornecedor".
