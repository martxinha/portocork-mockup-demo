# Portocork Mockup — Convenções e Restrições

Resumo das regras e decisões acumuladas ao longo do desenvolvimento do mockup
`settings-factory-calendar.html`. Serve como referência rápida para continuar
o trabalho sem repetir correções já feitas.

## Contexto do projeto

- Mockup estático (HTML/CSS/JS puro, sem framework, sem build step) para a
  Portocork (PTK), a mais recente unidade de cortiça do grupo Amorim.
- Reutiliza a linguagem visual (Vuetify/Material Design) e os padrões de UI
  dos outros planeadores do grupo (Amorim Lamas, Amorim AD, Amorim DS), mas
  com processos e terminologia próprios da Portocork.
- Dois fluxos operacionais alternáveis no topo da página: **Distribuição** e
  **NDTech** (`state.globalFlow`). A maioria das tabelas principais
  (Encomendas por Planear, Carteira de Encomendas, STOs, Cumprimento do
  Plano) é **partilhada entre os dois fluxos** — não filtram por
  `globalFlow`. Só `GESTAO_ALERTAS_ROWS` / `ALERTAS_ROWS` e as tabelas de
  Máquinas são específicas de cada fluxo.
- Português de Portugal em toda a interface (terminologia dos planeadores).

## Dados

- Depois de uma fase inicial com dados fictícios, o mockup passou a usar
  **dados reais** extraídos dos excels em `PortoCork Distribuição/` e
  `PortoCork NDTECH/` (decisão explícita do utilizador, ciente de que o
  Artifact é hospedado fora do ambiente local).
- Volume esperado por tabela: dezenas de linhas reais (não milhares) — o
  mockup é uma demonstração, não uma réplica do dataset de produção.
- Ao popular uma tabela com dados reais, manter algumas das linhas
  originais que demonstram funcionalidades específicas (alertas, OVs
  afetadas, via verde, etc.) quando os dados reais não cobrem esse caso.
- **Gestão de Alertas é a lista mestre.** Qualquer `tipo`/alerta mostrado
  noutra página (Alertas, Encomendas por Planear, Carteira de Encomendas)
  tem de corresponder exatamente a uma `descricao` em `GESTAO_ALERTAS_ROWS`
  (ver `gestaoAlertaByDescricao()`). Nunca inventar um tipo de alerta numa
  página consumidora sem primeiro o adicionar à Gestão de Alertas. Antes de
  remover/readicionar um alerta, verificar se já não foi explicitamente
  removido antes.
- Nunca usar o mesmo ícone para dois alertas diferentes.
- Tipos de lavação canónicos: `N101`, `Light`, `Prelight`, `Nature`
  (`LAVACAO_TIPOS`) — não usar "Natural/Clássica/Rosé" nem outros nomes
  inventados.

## Convenções de tabela

- Sempre que uma célula combinaria "código · nome" (ex.: "PS001 ·
  Pesagem"), separar em duas colunas: **Centro de Trabalho** (código) e
  **Processo** (nome). Se uma linha tiver vários códigos separados por
  "/", separar em várias linhas (uma por código).
- Tabelas genéricas construídas com `renderStaticTable(...)` têm sempre,
  na coluna Ações, um ícone de lápis (editar) **e** um ícone de lixo
  (eliminar) lado a lado — não só o lápis.
- Em tabelas `table-tight` (muitas colunas), os cabeçalhos podem quebrar em
  duas linhas (`white-space: normal` nos `<th>`) para a tabela caber sem
  scroll horizontal — preferir isso a scroll lateral sempre que possível.
- Filtros/campos de pesquisa têm sempre uma legenda acima (usar
  `labeledSelect`/`labeledInput`), nunca um `<select>`/`<input>` nu.
- Campos de filtro (`.filter-select`) são "largos" por defeito (min-width
  180px) — não voltar a estreitar globalmente.

## Cores e destaque

- Destaque de linha por alerta/atenção usa **amarelo/âmbar**
  (`var(--evt-holiday-bg)`), não vermelho — vermelho (`--evt-stop-bg`) fica
  reservado para estados realmente "parados/rejeitados", não para simples
  chamadas de atenção. Aplica-se a: Carteira de Encomendas (linhas com
  alerta), STOs (linhas em Alteração). Em Encomendas por Planear, o
  destaque só deve aparecer quando o badge de STO está mesmo a vermelho
  (`pct >= 100`), não em todas as linhas com `sto.alert`.
- "Via Verde · Controlo PTK" (STOs) só existe no fluxo Distribuição — nunca
  mostrar em NDTech independentemente do valor do campo.
- Checkboxes que representam um valor **por defeito, editável** (ex.:
  Controlo de Qualidade em Acompanhamento da Produção) devem ter uma cor
  diferente do estado "confirmado/final" usado noutro lado da app — usam
  `var(--evt-holiday)` (âmbar), não `var(--evt-closure)` (verde).
- Hover em itens de menu/lista nunca deve usar a mesma cor do fundo do
  próprio painel — verificar sempre o contraste antes de aplicar uma
  variável CSS de hover.

## Coisas já removidas (não voltar a adicionar sem pedido explícito)

- Aba/rota "Simulação" (Macro Planeamento).
- Botão "Continuar Fluxo" (Criar OFs) — o modal subjacente ficou como
  código morto, propositadamente.
- Separadores "Embalagem"/"Semanais" em Regras de Fecho OFs — passou a ser
  um único conjunto de regras, sem abas.
- Coluna "Ações" na tabela semanal de Carga Capacidade.
- Coluna "Alertas" na tabela de STOs.
- Alerta "Artigos sem Lavação Definida" (tanto na Gestão de Alertas como no
  cartão de Alertas da Homepage).

## Notas técnicas

- Sem backend nem framework — tudo em `var`/funções dentro de uma única
  IIFE. Ordem de declaração importa (ex.: inicializações que dependem de
  `CLIENTES_ROWS` têm de vir depois da declaração desse array, não junto de
  `PLANO_ROWS`).
- Publicado como Claude Artifact privado; o ficheiro `.html` em si é
  autónomo e pode ser aberto/enviado directamente sem o Artifact.
