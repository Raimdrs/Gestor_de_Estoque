# 02. Especificação de Requisitos Funcionais e Não Funcionais

**Projeto:** Japa em Casa — Sistema Profissional de Gestão de Estoque para Restaurantes  
**Padrão de Engenharia:** IEEE 830 / ISO/IEC/IEEE 29148  
**Idioma e Localização:** Português Brasileiro (`pt-BR`), Moeda Real (`R$ / BRL`), Fuso Horário Padrão `America/Sao_Paulo`.

---

## 1. Requisitos Funcionais (RF)

Os Requisitos Funcionais estão organizados por módulo e classificados por prioridade (**Essencial**, **Importante**, **Desejável**).

### Módulo 01: Autenticação, Usuários e Controle de Acesso (RBAC)

| Código | Requisito | Descrição Detalhada | Prioridade |
| :--- | :--- | :--- | :--- |
| **RF-AUTH-001** | Autenticação JWT com Refresh Token | Permitir login via e-mail e senha com emissão de *Access Token* de curta duração (15 min) e *Refresh Token* seguro (7 dias, armazenado em cookie `HttpOnly` com rotação automática). | Essencial |
| **RF-AUTH-002** | Bloqueio por Tentativas Inválidas | Bloquear temporariamente a conta ou aplicar *rate limiting* progressivo após 5 tentativas consecutivas de login malsucedidas. | Essencial |
| **RF-USER-001** | Gestão de Usuários | Permitir ao Administrador cadastrar, listar, editar, ativar/inativar usuários e redefinir credenciais de acesso. | Essencial |
| **RF-USER-002** | Controle de Acesso Baseado em Papéis (RBAC) | Suportar os perfis pré-configurados (**Administrador**, **Gerente**, **Gestor de Estoque**, **Estoquista**) e permitir que o Administrador configure permissões granulares por módulo/ação (`resource:action`). | Essencial |
| **RF-AUD-001** | Trilha de Auditoria (Audit Log) | Registrar automaticamente todas as ações sensíveis (criação/edição de produtos, estornos, aprovação de compras, ajustes de inventário, alterações de permissão) com ID do usuário, IP, timestamp, dados anteriores (`before`) e novos dados (`after`). | Essencial |

---

### Módulo 02: Cadastro de Produtos, Categorias, Unidades e Locais

| Código | Requisito | Descrição Detalhada | Prioridade |
| :--- | :--- | :--- | :--- |
| **RF-CAT-001** | Categorias e Subcategorias Pré-definidas e Customizáveis | Disponibilizar as 12 categorias iniciais do setor alimentício (*Carnes e aves, Peixes e frutos do mar, Hortifrúti, Laticínios e derivados, Grãos e cereais, Massas e farináceos, Temperos e condimentos, Bebidas, Produtos congelados, Produtos de limpeza, Embalagens e descartáveis, Outros insumos*) e permitir criar/editar categorias e subcategorias personalizadas. | Essencial |
| **RF-UOM-001** | Unidades de Medida e Conversões | Gerenciar unidades do setor alimentício (`KG`, `G`, `L`, `ML`, `UN`, `CX`, `FD`, `PCT`, `PORÇÃO`) e fatores de conversão entre unidade de compra/embalagem e unidade base de estoque. | Essencial |
| **RF-LOC-001** | Locais de Armazenamento | Cadastrar e gerenciar múltiplos locais físicos de armazenamento (ex: *Câmara Fria 01 - Peixes*, *Freezer Horizontal - Congelados*, *Estoque Seco*, *Adega/Bebidas*, *Garde Manger / Sushibar*) indicando faixa de temperatura recomendada. | Essencial |
| **RF-PROD-001** | Cadastro Completo de Produtos | Cadastrar produtos contendo: Nome, SKU (gerado automaticamente ou manual), Código de Barras (EAN-13/GTIN ou interno), Descrição, Categoria, Subcategoria, Unidade de Medida Base, Marca, Fornecedor Principal, Custo Unitário Atual, Custo Médio Ponderado (CMP), Estoque Atual, Estoque Mínimo, Estoque Máximo, Estoque de Segurança, Ponto de Reposição, Local de Armazenamento Padrão, Prazo de Validade Padrão (dias fechado e dias após aberto), Condição Térmica (*Ambiente, Refrigerado, Congelado*), Rendimento Padrão (%) e Status (*Ativo/Inativo*). | Essencial |
| **RF-PROD-002** | Pesquisa Avançada, Filtros e Leitura de Código de Barras | Permitir busca instantânea por nome, SKU ou código de barras (incluindo leitura via câmera do smartphone/tablet ou leitor USB), além de filtros combinados por categoria, fornecedor, local de armazenamento, condição térmica e situação do estoque. | Essencial |
| **RF-PROD-003** | Bloqueio de Produtos Inativos | Impedir que produtos com status *Inativo* sejam selecionados em novas solicitações de compra, entradas ou saídas, preservando todo o histórico passado para relatórios e auditoria. | Essencial |

---

### Módulo 03: Controle Inteligente de Estoque, Lotes e Validade (FEFO)

| Código | Requisito | Descrição Detalhada | Prioridade |
| :--- | :--- | :--- | :--- |
| **RF-STK-001** | Parametrização de Níveis de Estoque por Produto | Gerenciar individualmente por produto os parâmetros: Estoque Mínimo, Estoque de Segurança, Ponto de Reposição e Estoque Máximo. | Essencial |
| **RF-STK-002** | Cálculo de Estoque Disponível Real | Calcular o `Estoque Disponível = Saldo Total de Lotes Ativos e Não Vencidos - Quantidade Reservada`. Lotes vencidos ou bloqueados para quarentena são automaticamente deduzidos do saldo disponível para uso. | Essencial |
| **RF-STK-003** | Cálculo Automático de Custo Médio Ponderado (CMP) | Recalcular automaticamente o CMP do produto a cada entrada por compra: `Novo CMP = ((Saldo Atual * CMP Atual) + (Qtd Recebida * Custo Unitário de Entrada)) / (Saldo Atual + Qtd Recebida)`. | Essencial |
| **RF-LOT-001** | Rastreabilidade Completa por Lote | Registrar para cada lote: Produto, Código do Lote (do fabricante ou gerado internamente no formato `LOT-AAAAMMDD-XXXX`), Data de Recebimento, Data de Fabricação (opcional), Data de Validade, Quantidade Recebida, Quantidade Disponível, Fornecedor de Origem, Local de Armazenamento e Status (`ATIVO`, `ESGOTADO`, `VENCIDO`, `BLOQUEADO`). | Essencial |
| **RF-LOT-002** | Algoritmo FEFO Automático (*First Expired, First Out*) | Ao registrar uma saída de estoque (para produção ou consumo), sugerir e alocar automaticamente a retirada a partir do(s) lote(s) com data de validade mais próxima, permitindo múltiplos lotes se a quantidade requisitada exceder o primeiro lote. | Essencial |
| **RF-LOT-003** | Bloqueio de Uso de Lotes Vencidos | Impedir que lotes com `data_validade < data_atual` sejam consumidos em saídas normais de produção/consumo, exigindo que sejam baixados exclusivamente como *Saída por Vencimento / Perda*. | Essencial |
| **RF-LOT-004** | Painel de Vencimentos e Alertas Escalonados | Disponibilizar tela dedicada listando lotes vencidos e próximos do vencimento, com disparos de alertas configuráveis em **30 dias**, **15 dias**, **7 dias**, **3 dias** antes e **no dia do vencimento**. | Essencial |
| **RF-LOT-005** | Fracionamento / Abertura de Insumo (Validade Secundária) | Permitir registrar a abertura/manipulação de um lote fechado, recalculando a data de validade da fração aberta conforme o prazo de validade pós-abertura do produto. | Importante |

---

### Módulo 04: Movimentações de Estoque (Entradas, Saídas, Transferências e Estornos)

| Código | Requisito | Descrição Detalhada | Prioridade |
| :--- | :--- | :--- | :--- |
| **RF-MOV-001** | Registro Padronizado de Movimentações | Suportar os 8 tipos oficiais de movimentação: `ENTRADA_COMPRA`, `ENTRADA_DEVOLUCAO`, `SAIDA_PRODUCAO`, `SAIDA_CONSUMO`, `SAIDA_PERDA`, `SAIDA_VENCIMENTO`, `AJUSTE_INVENTARIO`, `TRANSFERENCIA`. | Essencial |
| **RF-MOV-002** | Metadados Obrigatórios de Movimentação | Exigir e gravar em toda movimentação: Produto, Quantidade (na unidade base e unidade informada), Custo Unitário no momento da operação, Saldo Anterior, Saldo Posterior, Tipo, Data/Hora exata, Usuário Responsável, Motivo/Justificativa, Lote(s) afetado(s), Local de Origem/Destino e Documento de Referência (NF, Pedido, Inventário). | Essencial |
| **RF-MOV-003** | Imutabilidade do Histórico e Estorno Auditável | Proibir edição ou exclusão física (`UPDATE` / `DELETE`) de registros de movimentação já confirmados. Qualquer correção deve gerar uma nova movimentação compensatória de **Estorno** ou **Ajuste**, vinculada ao ID da movimentação original e exigindo motivo obrigatório. | Essencial |
| **RF-MOV-004** | Bloqueio de Estoque Negativo | Bloquear qualquer operação de saída ou transferência que resulte em saldo negativo no lote ou no produto (salvo se o usuário possuir permissão especial de Administrador/Gerente e assinar justificativa de auditoria). | Essencial |
| **RF-MOV-005** | Transferência entre Locais de Armazenamento | Permitir transferir total ou parcialmente o saldo de um lote de um local (ex: *Câmara Fria Principal*) para outro (ex: *Balcão Refrigerado Sushibar*), mantendo o mesmo lote e validade. | Essencial |

---

### Módulo 05: Gestão de Fornecedores e Compras

| Código | Requisito | Descrição Detalhada | Prioridade |
| :--- | :--- | :--- | :--- |
| **RF-SUP-001** | Cadastro Completo de Fornecedores | Cadastrar Razão Social, Nome Fantasia, CNPJ/CPF (com validação de dígitos verificadores), Nome do Contato, Telefone/WhatsApp, E-mail, Endereço completo, Condições de Pagamento padrão, Prazo Médio de Entrega (*Lead Time* em dias), Observações e Status (*Ativo/Inativo*). | Essencial |
| **RF-SUP-002** | Catálogo Produto x Fornecedor (`ProductSupplier`) | Relacionar múltiplos produtos a múltiplos fornecedores, armazenando por vínculo: código do produto no fornecedor, unidade de compra (ex: Caixa 10kg), fator de conversão, último preço praticado e se é o fornecedor preferencial. | Essencial |
| **RF-SUP-003** | Histórico e Avaliação de Fornecedores | Exibir histórico completo de compras por fornecedor, evolução de preços praticados, cumprimento de prazos de entrega e índice de divergências no recebimento. | Importante |
| **RF-PUR-001** | Identificação Automática de Necessidade de Compra | Listar automaticamente todos os produtos cujo `Estoque Disponível <= Ponto de Reposição`, calculando a **Sugestão de Quantidade de Compra** com base na fórmula: `Sugestão = Estoque Máximo - Estoque Disponível - Quantidade em Pedidos Abertos`. | Essencial |
| **RF-PUR-002** | Ciclo de Vida do Pedido de Compra | Gerenciar pedidos de compra através dos status: `RASCUNHO` -> `AGUARDANDO_APROVACAO` -> `APROVADO` -> `ENVIADO_FORNECEDOR` -> `PARCIALMENTE_RECEBIDO` -> `RECEBIDO` (ou `CANCELADO`). | Essencial |
| **RF-PUR-003** | Alçada de Aprovação de Compras | Exigir aprovação de usuário com perfil **Gerente** ou **Administrador** para transição do status `AGUARDANDO_APROVACAO` para `APROVADO`. | Essencial |
| **RF-PUR-004** | Conferência de Recebimento e Divergências | Registrar o recebimento físico informando Nota Fiscal, quantidades realmente entregues por item, número do lote e data de validade. Comparar automaticamente `Quantidade Solicitada vs. Quantidade Recebida` e `Preço Pedido vs. Preço NF`, gerando entrada no estoque **apenas da quantidade efetivamente aceita**. | Essencial |

---

### Módulo 06: Controle de Perdas, Desperdícios e Inventário Físico

| Código | Requisito | Descrição Detalhada | Prioridade |
| :--- | :--- | :--- | :--- |
| **RF-WST-001** | Registro Detalhado de Perdas e Desperdícios | Registrar perdas contendo: Produto, Quantidade, Lote afetado, Classificação do Motivo (`VENCIMENTO`, `DETERIORACAO`, `ARMAZENAMENTO_INADEQUADO`, `ERRO_MANIPULACAO`, `EXCESSO_PRODUCAO`, `DANOS_TRANSPORTE`, `OUTROS`), Data/Hora, Usuário Responsável, Observação/Foto opcional e Valor Financeiro Estimado (calculado automaticamente via `Quantidade * CMP`). | Essencial |
| **RF-WST-002** | Análise de Impacto Financeiro de Desperdício | Gerar demonstrativos de perdas em R$ e percentual sobre o consumo total, agrupados por motivo, produto, categoria e período. | Essencial |
| **RF-INV-001** | Abertura de Inventário Físico (Completo ou Parcial) | Permitir abrir sessões de inventário filtrando por Local de Armazenamento, Categoria ou seleção manual de produtos (ex: inventário diário de proteínas nobres como salmão e atum, e inventário mensal geral). | Essencial |
| **RF-INV-002** | Contagem às Cegas e Mapa de Divergências | Capturar a quantidade física encontrada por lote/produto (sem exibir o saldo do sistema para o Estoquista), calcular automaticamente a divergência (`Qtd Física - Qtd Sistema`) e o impacto financeiro (`Divergência * CMP`). | Essencial |
| **RF-INV-003** | Aprovação de Inventário e Ajuste Automático | Exigir justificativa para cada item divergente e submeter à aprovação do Gestor/Gerente. Ao aprovar, gerar automaticamente as movimentações de `AJUSTE_INVENTARIO` (positivas ou negativas) em transação única. | Essencial |

---

### Módulo 07: Dashboard Gerencial, Alertas e Relatórios

| Código | Requisito | Descrição Detalhada | Prioridade |
| :--- | :--- | :--- | :--- |
| **RF-DSH-001** | Painel Principal de Indicadores (KPIs) | Apresentar no Dashboard: Total de Itens Ativos, Valor Financeiro Total do Estoque (R$), Itens Abaixo do Mínimo, Itens Próximos do Vencimento, Itens Vencidos, Itens Esgotados (Sem Estoque), Compras no Período (R$), Entradas e Saídas Recentes, Top 10 Produtos de Maior Consumo e Top 10 Produtos de Maior Desperdício. | Essencial |
| **RF-NOT-001** | Motor Centralizado de Alertas por Prioridade | Classificar e emitir alertas automáticos em 4 níveis: **Crítico** (produto esgotado, abaixo do estoque de segurança, lote vencido), **Alto** (abaixo do estoque mínimo, vencimento em <= 7 dias), **Médio** (atingiu ponto de reposição, vencimento entre 8 e 30 dias, consumo acima da média histórica), **Informativo** (estoque acima do máximo, pedido recebido). | Essencial |
| **RF-NOT-002** | Gestão de Notificações e Preferências | Permitir visualizar alertas no topo da interface e na Central de Alertas, marcar como lido, rastrear a resolução (ex: alerta fecha automaticamente quando o estoque é reposto) e configurar quais perfis/usuários recebem cada nível de alerta (com arquitetura plugável *Adapter Pattern* pronta para E-mail e WhatsApp). | Essencial |
| **RF-REP-001** | Geração dos 13 Relatórios Gerenciais Obrigatórios | Gerar com filtros por período, produto, categoria, local e fornecedor: (1) Posição Atual do Estoque, (2) Valor Financeiro do Estoque, (3) Histórico de Entradas e Saídas, (4) Produtos Abaixo do Mínimo, (5) Produtos Próximos do Vencimento, (6) Produtos Vencidos, (7) Consumo por Período, (8) Compras por Período, (9) Histórico de Preços de Aquisição, (10) Perdas e Desperdícios, (11) Produtos com Maior Consumo (Curva ABC), (12) Produtos com Menor Movimentação (Estoque Parado), (13) Divergências de Inventário. | Essencial |
| **RF-REP-002** | Exportação em PDF e Excel (.xlsx) | Permitir exportar qualquer relatório gerencial em formato **PDF** (formatado para impressão com cabeçalho do restaurante, data/hora e emissor) e **Excel (.xlsx)** para análises externas. | Essencial |

---

## 2. Requisitos Não Funcionais (RNF) e Metas Mensuráveis (SLAs)

| Código | Categoria | Requisito | Métrica / Critério de Aceitação Mensurável |
| :--- | :--- | :--- | :--- |
| **RNF-PERF-001** | Desempenho de API | Tempo de resposta das operações comuns de leitura e escrita (consulta de saldo, busca de produto, registro de entrada/saída). | Latência **p95 < 250ms** e **p99 < 500ms** sob carga de 100 requisições concorrentes. |
| **RNF-PERF-002** | Desempenho do Dashboard | Carregamento inicial do Dashboard Gerencial com gráficos e alertas. | Tempo total de renderização (*Largest Contentful Paint - LCP*) **< 1,8 segundos** em conexão 4G/Wi-Fi. |
| **RNF-PERF-003** | Processamento Assíncrono | Geração de relatórios pesados (PDF/Excel com > 10.000 linhas) e varredura de alertas de validade/consumo. | Executados em background via filas **BullMQ / Redis**, sem bloquear a thread principal da API HTTP. |
| **RNF-UX-001** | Usabilidade Operacional | Eficiência na execução das tarefas diárias do estoquista em dispositivos móveis. | Registro de baixa de produção ou perda concluído em **no máximo 3 etapas/telas** e **< 15 segundos**. |
| **RNF-COMP-001** | PWA e Responsividade | Compatibilidade multiplataforma e instalação como aplicativo nativo. | Layout fluido de `320px` (Smartphone) a `2560px` (Desktop), com `manifest.json`, Service Worker ativo e pontuação **Lighthouse PWA >= 90**. |
| **RNF-SEC-001** | Criptografia e Autenticação | Proteção de credenciais e transporte de dados. | Senhas com hash **Argon2id** (ou bcrypt cost >= 12), tráfego 100% sobre **HTTPS (TLS 1.3)**, headers de segurança via **Helmet** e CORS restrito. |
| **RNF-SEC-002** | Proteção contra Ataques Web | Prevenção contra SQL Injection, XSS, CSRF e força bruta. | Queries 100% parametrizadas via **Prisma ORM**, validação estrita de payload via **Zod / class-validator** e **Rate Limiting** (100 req/min por IP; 5 req/min no login). |
| **RNF-SEC-003** | Conformidade LGPD | Privacidade e proteção de dados pessoais (Lei nº 13.709/2018). | Coleta mínima de dados pessoais (apenas nome, e-mail profissional e CPF/CNPJ de fornecedores), mascaramento de dados em logs e direito de anonimização ao desligar usuário. |
| **RNF-INT-001** | Integridade Transacional | Consistência absoluta em saldos físicos e financeiros. | 100% das operações que alteram saldo utilizam transações ACID no PostgreSQL com bloqueio de linha (`SELECT FOR UPDATE`) e precisão numérica `DECIMAL` (nunca `FLOAT` para valores monetários ou quantidades). |
| **RNF-DISP-001** | Disponibilidade e Recuperação | Continuidade de negócios e proteção contra perda de dados. | Disponibilidade alvo de **99,5%**, backups automatizados diários do PostgreSQL com retenção de 30 dias (**RPO < 1 hora**, **RTO < 2 horas**) e endpoints de `/health` monitorados. |
| **RNF-MAN-001** | Manutenibilidade e Qualidade | Padronização de código, tipagem estrita e documentação viva. | 100% TypeScript (`strict: true`), documentação **OpenAPI 3.0 (Swagger)** gerada automaticamente e cobertura de testes automatizados **>= 80% nas regras de domínio**. |
