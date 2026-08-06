# Portocork Mockup — Resumo da Sessão

Resumo do trabalho feito nesta conversa sobre o mockup
`settings-factory-calendar.html` (Configurações · Calendário da Fábrica —
Portocork). Ver também `CONVENTIONS.md` (regras transversais) e
`VIEWS.md` (lógica por página) para referência mais detalhada e
duradoura — este ficheiro é sobretudo um registo cronológico do que foi
pedido e feito.

## Contexto

Mockup estático (HTML/CSS/JS, sem framework/build) para a Portocork,
reutilizando os padrões visuais e de domínio dos outros planeadores do
grupo Amorim (Lamas, AD, DS). Dois fluxos alternáveis no topo:
**Distribuição** e **NDTech**. Publicado como Claude Artifact privado
(link mantido estável ao longo de toda a sessão via republicação no
mesmo ficheiro).

## 1. Reconciliação do registo de alertas

- "Gestão de Alertas" foi estabelecida como **lista mestre** de tipos de
  alerta — todas as outras páginas (Alertas, Encomendas por Planear,
  Carteira de Encomendas) passaram a derivar ícone/cor a partir dela em
  vez de manter listas paralelas.
- Removidos definitivamente da Gestão de Alertas (e de todas as páginas
  que os usavam): "OV Inexistente", "Artigo sem Componente de Marcação",
  "Stock de Reprocessamento Disponível", "Alerta de Calendário", "Artigos
  sem Lavação Definida" — alguns destes já tinham sido pedidos para
  remover antes e voltaram a aparecer por engano; foram removidos de
  vez, incluindo do cartão de Alertas da Homepage (lista independente que
  tinha ficado desatualizada).
- Ícones trocados: Laboratório ↔ Tratamento (estavam invertidos).

## 2. STOs — reformulação da página

- Estado STO passou a distinguir **Por Planear** / **Planeada** (em vez
  de um "Pendente" genérico) além de Alteração/Expedido.
- Coluna "Progresso" substituída por **"Estado de Produção do
  Fornecedor"** (Por Embalar / Em Controlo / Aprovada / Rejeitada), com
  badge extra "Via Verde · Controlo PTK" — só visível em Distribuição.
- "OVs Afetadas" restringido a linhas em Alteração, com detalhe completo
  (OV, Item, Data Planeada, Nova Data Prevista) em vez de uma lista de
  texto simples.
- Coluna "Ações" passou a abrir um menu de 3 opções (Alterar Data /
  Alterar Quantidade / Eliminar) em vez de um formulário combinado.
- Nova coluna "Contactar Fornecedor" (email não automático, exige
  validação explícita antes de enviar); "Histórico de Datas" renomeada
  para "Alteração de Datas" e só mostra o ícone quando há alteração
  real.
- Colunas "Alertas" e "Ações" (na tabela semanal de Carga Capacidade,
  separadamente) removidas por não fazerem sentido.

## 3. Dados reais

A pedido explícito do utilizador (avisado do trade-off de usar dados
reais num Artifact hospedado fora do ambiente local), populámos as 4
tabelas principais partilhadas entre fluxos com dados reais extraídos
dos excels em `PortoCork Distribuição/` e `PortoCork NDTECH/`:

- **Encomendas por Planear**, **Carteira de Encomendas**, **STOs** e
  **Cumprimento do Plano** — dezenas de linhas reais cada, mantendo
  algumas linhas originais para preservar demonstrações de
  funcionalidades (alertas, via verde, OVs afetadas) que os dados reais
  não cobriam sozinhos.
- Descoberta importante: estas 4 tabelas **não são filtradas por
  fluxo** — são partilhadas, por isso não existe uma "versão NDTech"
  separada a popular.
- Ajustes de qualidade feitos depois: destaque de cor a amarelo em vez
  de vermelho (Carteira, STOs, Encomendas por Planear — só quando o
  badge de STO está mesmo crítico), e variedade de ícones de alerta em
  Encomendas por Planear (deixou de ser quase todos "camião").

## 4. Carga Capacidade

- Ordem dos centros invertida (último processo do fluxo primeiro).
- Limite de sobrecarga corrigido de 100% para **90%** (cores, linha de
  referência e legenda).
- Parênteses em "Carga Atual" passaram a mostrar o valor **planeado**
  (uma percentagem), não uma banda de "objetivo".
- Coluna de Ações removida da tabela semanal.
- NDTech > Pesagem: centro de trabalho corrigido para "Por definir" (não
  tem PS001 — esse é da Distribuição).

## 5. Máquinas

- NDTech: cadência, lead time e OEE corrigidos para os valores reais
  fornecidos (com correção posterior: cadência é `un/h`, não `ML/h`,
  como se pensou inicialmente); nova coluna "Abertura (h)" (24h para
  NDTech, 16h para as restantes etapas).
- Distribuição: adicionada a mesma coluna "Abertura (h)", calculada como
  `turnos × 8h` (sem fonte real específica — assinalado ao utilizador).
- Restrições (`Configurações > Recursos > Restrições`): confirmada/anotada
  a restrição de Lavação NDTech (Máq 1: Light/Nature/N101; Máq 2:
  Light/N101; CL0 não ocupa máquina) e adicionada a restrição em falta
  "só a EMB002 processa calibres de comprimento > 49" (Escolha,
  Distribuição).

## 6. Outras correções de UI ao longo da sessão

- Hover do submenu lateral corrigido (usava a mesma cor do fundo do
  painel, por isso não se via nada ao passar o rato).
- Checkbox de Controlo de Qualidade (Acompanhamento da Produção) passou
  a âmbar em vez de verde, para sinalizar "valor por defeito, editável"
  em vez de "confirmado".
- Removidos: aba "Simulação", sub-abas "Embalagem"/"Semanais" em Regras
  de Fecho OFs, coluna "Nome" em Criar OFs (redundante com "Processo"),
  item "Data Prevista de Conclusão" em Prioridades de Planeamento >
  Sequenciamento (Distribuição).
- Tipos de lavação da Homepage corrigidos para os canónicos (N101/
  Light/Prelight/Nature), em vez de nomes inventados.
- Correção de um bug vivo: `renderEliminarBody()` ainda referenciava
  variáveis de estado já removidas (`eliminarState`), o que rebentaria
  ao abrir essa página.

## 7. Reconciliação com export mais recente do Artifact

Numa continuação da sessão (fora deste repositório git), o mockup avançou
mais e foi exportado de novo a partir do Artifact para
`Portocork-Mockup (1).html` (~660KB vs ~464KB da cópia sincronizada até
então). Copiámos esse ficheiro para `settings-factory-calendar.html` e
`Portocork-Mockup.html`, confirmando primeiro que era um superset seguro
(todas as `key:` de página anteriores mantidas, nenhuma removida) antes de
substituir. Novidades identificadas e documentadas em `VIEWS.md`:

- Nova página **Report de Planeamento** (menu Acompanhamento): histórico
  do 1.º planeamento ao último sem Stand By por OV/Item, com âmbito
  seguindo os filtros/seleção de Encomendas por Planear.
- **Carteira de Encomendas** ganhou 3 tabs (Encomendas/Amostras/
  Provisionais — amostras OV `1425*` e motivo ZP3 saem do sequenciamento e
  capacidade), coluna **Controlo de Qualidade** (pill de 4 estados,
  clicável) e coluna auditável "Motivo Ordem".
- Novo **Alerta de Orientação** (corpo/topo/ambos na marcação a laser,
  só Distribuição) — devidamente registado em `GESTAO_ALERTAS_ROWS` como
  lista mestre, sem duplicar ícone.
- **Configurações > Recursos > Parametrizações**: novas tabelas Ocupação
  Máxima (limite de carga por semana do horizonte) e Partição de OFs.
- **Critérios de Liberação de OFs** ganhou coluna Tipo de Encomenda —
  regra dura: Provisional nunca libera OF.

## Ficheiros produzidos nesta sessão

- `settings-factory-calendar.html` — ficheiro de trabalho (também
  publicado como Artifact).
- `Portocork-Mockup.html` — cópia exportável, sempre sincronizada com o
  ficheiro de trabalho após cada alteração.
- `CONVENTIONS.md` — regras e restrições transversais.
- `VIEWS.md` — lógica e restrições por página.
- `SESSION-SUMMARY.md` — este ficheiro.
