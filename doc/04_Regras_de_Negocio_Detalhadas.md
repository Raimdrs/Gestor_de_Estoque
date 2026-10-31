# 04. Especificação Detalhada das Regras de Negócio

**Projeto:** Japa em Casa — Sistema Profissional de Gestão de Estoque para Restaurantes  
**Camada Arquitetural:** Domain Layer (Entidades, Value Objects e Domain Services)

---

## 1. Catálogo de Regras de Negócio Obrigatórias e Complementares

Abaixo estão detalhadas as **14 Regras de Negócio Obrigatórias** exigidas na especificação master, acrescidas de **4 Regras de Negócio Complementares de Food Service** identificadas na análise crítica, com suas respectivas **fórmulas matemáticas**, **validações de domínio** e **comportamento transacional**.

---

### RN-01: Bloqueio Estrito de Estoque Negativo
- **Definição:** Nenhuma operação de saída (`SAIDA_PRODUCAO`, `SAIDA_CONSUMO`, `SAIDA_PERDA`, `SAIDA_VENCIMENTO`), transferência ou ajuste negativo poderá resultar em saldo físico ou saldo de lote menor que zero (`< 0,0000`).
- **Mecanismo Técnico:**
  1. Na camada de Domínio (`StockLot.deduct(quantity)`), se `quantity > this.availableQuantity`, lança-se a exceção `InsufficientStockException`.
  2. No Banco de Dados (PostgreSQL), as tabelas `stocks` e `stock_lots` possuem restrição física `CONSTRAINT chk_positive_balance CHECK (quantity >= 0)`.
- **Exceção Auditada:** Caso o restaurante configure a permissão especial de contingência (`stock:allow_negative_override` restrita exclusivamente a Administrador/Gerente em casos extremos em que a mercadoria física já entrou na cozinha antes de a Nota Fiscal ser lançada), o sistema exige senha de confirmação, justificativa obrigatória e grava um alerta **Crítico** de regularização pendente em `AuditLog`. Por padrão de fábrica, essa flag vem **desabilitada (`false`)**.

---

### RN-02: Consistência Atômica de Saldos nas Movimentações
- **Definição:** Toda movimentação deve atualizar simultaneamente o saldo do lote (`StockLot`), o saldo consolidado do local (`Stock`) e o saldo consolidado do produto (`Product.currentStock`), gravando na própria linha de `StockMovement` o `previous_balance` (saldo anterior) e o `resulting_balance` (saldo posterior).
- **Invariante de Integridade:** A qualquer instante $t$, para qualquer produto $P$:
  $$\text{Saldo Total}(P) = \sum_{i=1}^{n} \text{StockLot}_i.\text{available\_quantity} \quad (\text{onde status} \in \{\text{ACTIVE}, \text{EXPIRED}\})$$
  $$\text{Estoque Disponível}(P) = \sum_{i=1}^{n} \text{StockLot}_i.\text{available\_quantity} \quad (\text{onde status} = \text{ACTIVE e } \text{expiration\_date} \ge \text{Hoje}) - \text{ReservedQuantity}(P)$$

---

### RN-03: Rastreabilidade Obrigatória na Origem das Entradas
- **Definição:** Nenhuma entrada de mercadoria pode existir "solta".
  - `ENTRADA_COMPRA`: Exige vínculo obrigatório com um Fornecedor (`supplier_id`), Documento Fiscal/Referência (`document_number`, ex: NF-e) e Pedido de Compra ou Recebimento Avulso autorizado, além da criação ou vinculação de um Lote (`StockLot`) com data de validade.
  - `ENTRADA_DEVOLUCAO`: Exige vínculo com a saída original de produção que está retornando intacta ao estoque, mantendo o lote e a validade originais.

---

### RN-04: Motivo e Responsável Obrigatórios nas Saídas
- **Definição:** Toda saída de estoque exige identificação do usuário autenticado (`user_id` extraído do token JWT, nunca enviado livremente pelo cliente sem validação), tipo de saída, centro de custo/destino (quando aplicável) e motivo estruturado.

---

### RN-05: Bloqueio Automático de Produtos e Lotes Vencidos
- **Definição:** Produtos cujos lotes atingiram `expiration_date < CURRENT_DATE` são considerados **impróprios para consumo** pela legislação sanitária (ANVISA).
- **Comportamento no Sistema:**
  1. Lotes vencidos têm seu saldo imediatamente excluído do cálculo de `Estoque Disponível` (embora permaneçam no `Saldo Físico Total` até que o descarte físico seja realizado e registrado).
  2. Lotes vencidos **não podem** ser selecionados em operações de `SAIDA_PRODUCAO` ou `SAIDA_CONSUMO`.
  3. A única saída permitida para um lote vencido é `SAIDA_VENCIMENTO` (que gera automaticamente um registro no módulo de **Perdas e Desperdícios**).

---

### RN-06: Baixa de Estoque pelo Método FEFO (*First Expired, First Out*)
- **Definição:** Nas saídas de estoque para produção ou consumo, o sistema deve priorizar os lotes cuja data de validade vence mais cedo.
- **Algoritmo de Alocação FEFO:**
  Dado um produto $P$ e uma quantidade requisitada $Q_{\text{req}} > 0$:
  1. Buscar lotes de $P$ no local de origem onde `status = 'ACTIVE'`, `available_quantity > 0` e `expiration_date >= CURRENT_DATE`.
  2. Ordenar os lotes por:
     - `expiration_date ASC` (vence primeiro vem primeiro);
     - `received_at ASC` (desempate por FIFO — entrou primeiro vem primeiro).
  3. Iterar sobre os lotes ordenados alocando $\min(Q_{\text{restante}}, \text{Lote}_i.\text{available\_quantity})$ até que $Q_{\text{restante}} = 0$.
  4. Se após percorrer todos os lotes válidos $Q_{\text{restante}} > 0$, abortar a operação por saldo disponível insuficiente (RN-01 e RN-05).

---

### RN-07: Imutabilidade do Histórico e Correção Exclusiva por Estorno/Ajuste
- **Definição:** A tabela `stock_movements` é um livro-razão contábil (*append-only ledger*). Não existem endpoints de `PUT /movements/:id` ou `DELETE /movements/:id`.
- **Regra de Estorno:** Quando um erro operacional ocorre, a operação de estorno:
  1. Marca a movimentação original com `is_reversed = true`;
  2. Insere uma nova linha em `stock_movements` com quantidade inversa, referenciando `reversed_from_id = original.id`, `user_id` de quem autorizou o estorno e `reason` obrigatório;
  3. Registra a operação em `audit_logs`.

---

### RN-08: Cálculo do Custo Médio Ponderado (CMP / Moving Average Cost)
- **Definição:** O valor financeiro do estoque é avaliado pelo método do **Custo Médio Ponderado Móvel**, recalculado a cada nova entrada por compra (`ENTRADA_COMPRA`).
- **Fórmula Matemática:**
  Seja:
  - $Q_{\text{atual}}$ = Saldo total do produto antes da entrada (na unidade base)
  - $CMP_{\text{atual}}$ = Custo Médio Ponderado atual do produto (por unidade base)
  - $Q_{\text{entrada}}$ = Quantidade recebida na nova compra (convertida para a unidade base)
  - $C_{\text{entrada}}$ = Custo unitário líquido da nova compra (convertido para a unidade base)

  $$\text{Novo CMP} = \frac{(Q_{\text{atual}} \times CMP_{\text{atual}}) + (Q_{\text{entrada}} \times C_{\text{entrada}})}{Q_{\text{atual}} + Q_{\text{entrada}}}$$

- **Regras de Borda:**
  - Se $Q_{\text{atual}} = 0$, então $\text{Novo CMP} = C_{\text{entrada}}$.
  - Saídas de estoque (`SAIDA_PRODUCAO`, `SAIDA_PERDA`, etc.) **não alteram** o CMP unitário; elas saem valoradas pelo CMP vigente no momento da baixa, reduzindo proporcionalmente o Valor Financeiro Total do Estoque.
  - O campo `last_Purchase_cost` (Último Custo de Compra) também é atualizado paralelamente para fins de comparação de inflação de insumos.

---

### RN-09: Motor Reativo de Alertas e Níveis de Estoque
- **Definição:** Após qualquer movimentação de estoque ou execução do Job diário de validades, o sistema avalia as regras de alerta do produto afetado.
- **Parametrização e Hierarquia Lógica dos Limites:**
  Para todo produto, o cadastro valida a seguinte inequação lógica:
  $$0 \le \text{Estoque de Segurança} \le \text{Estoque Mínimo} \le \text{Ponto de Reposição} < \text{Estoque Máximo}$$
- **Fórmulas de Referência Sugeridas pelo Sistema (Auxílio ao Gestor):**
  - $\text{Estoque de Segurança} = \text{Consumo Médio Diário} \times \text{Dias de Margem de Segurança}$
  - $\text{Ponto de Reposição} = (\text{Consumo Médio Diário} \times \text{Lead Time do Fornecedor em Dias}) + \text{Estoque de Segurança}$
- **Tabela de Disparo de Alertas:**

| Condição Detectada | Tipo de Alerta | Prioridade | Ação Automática |
| :--- | :--- | :---: | :--- |
| $\text{Estoque Disponível} = 0$ | `OUT_OF_STOCK` (Produto Esgotado) | **CRÍTICO** | Notifica no topo do Dashboard + inclui na lista urgente de Compras |
| $0 < \text{Estoque Disponível} \le \text{Estoque de Segurança}$ | `BELOW_SAFETY_STOCK` | **CRÍTICO** | Notifica Gestor e Gerente imediatamente |
| $\text{Lote com } \text{expiration\_date} < \text{Hoje}$ e $\text{saldo} > 0$ | `LOT_EXPIRED` (Lote Vencido) | **CRÍTICO** | Bloqueia saldo disponível do lote + exige baixa por perda |
| $\text{Estoque de Segurança} < \text{Estoque Disponível} \le \text{Estoque Mínimo}$ | `BELOW_MIN_STOCK` | **ALTO** | Gera alerta de alta prioridade + sugere pedido de compra |
| $\text{Lote vence em } \le 7 \text{ dias}$ (configurável: 7, 3, 0 dias) | `EXPIRING_SOON_HIGH` | **ALTO** | Destaca para uso imediato na cozinha (prioridade FEFO) |
| $\text{Estoque Mínimo} < \text{Estoque Disponível} \le \text{Ponto de Reposição}$ | `REORDER_POINT_REACHED` | **MÉDIO** | Insere produto na fila de "Sugestão de Compras" |
| $\text{Lote vence em } 15 \text{ ou } 30 \text{ dias}$ | `EXPIRING_SOON_MEDIUM` | **MÉDIO** | Alerta preventivo na Central de Notificações |
| $\text{Consumo Diário} > 1,5 \times \text{Média Histórica (30 dias)}$ | `ABNORMAL_CONSUMPTION` | **MÉDIO** | Alerta Gestor sobre pico de demanda ou possível desvio |
| $\text{Estoque Disponível} > \text{Estoque Máximo}$ | `ABOVE_MAX_STOCK` | **INFORMATIVO** | Alerta sobre capital imobilizado excessivo / risco de perda |

---

### RN-10: Isolamento Transacional (ACID) em Operações Críticas
- **Definição:** Operações que envolvem múltiplas tabelas (Recebimento de Compra, Saída FEFO multi-lote, Registro de Perda, Aprovação de Inventário e Estorno) devem executar dentro de `prisma.$transaction()` com bloqueio pessimista (`SELECT ... FOR UPDATE`) nas linhas de saldo. Se qualquer etapa falhar, ocorre `ROLLBACK` completo.

---

### RN-11: Bloqueio de Produtos e Fornecedores Inativos
- **Definição:** Produtos ou Fornecedores com `is_active = false` não podem ser utilizados em novas compras, entradas ou saídas. Contudo, a exclusão física (`DELETE`) é proibida sempre que houver ao menos 1 registro histórico vinculado, preservando a integridade referencial de relatórios passados.

---

### RN-12: Recebimento Parcial e Atualização Estrita pelo Físico Recebido
- **Definição:** A aprovação ou envio de um Pedido de Compra (`PurchaseOrder`) **não altera** o saldo físico do estoque. O saldo só é incrementado no ato do **Recebimento de Mercadorias (`GoodsReceipt`)**, utilizando **exclusivamente a quantidade física conferida e aceita**.
  - Se o pedido solicitou `50 KG` e foram recebidos `35 KG`:
    1. O estoque aumenta em `+35 KG`;
    2. O item do pedido registra `received_quantity = 35 KG` e `pending_quantity = 15 KG`;
    3. O pedido assume o status `PARCIALMENTE_RECEBIDO` até que o restante seja entregue ou o saldo residual seja cancelado pelo comprador.

---

### RN-13: Auditoria Completa de Operações Sensíveis
- **Definição:** Toda operação que crie, altere ou estorne dados financeiros, parâmetros de estoque, permissões de usuário ou aprovações gera um registro imutável em `AuditLog` contendo: `actor_id`, `actor_role`, `action`, `entity_type`, `entity_id`, `old_values` (JSONB), `new_values` (JSONB), `ip_address`, `user_agent` e `created_at`.

---

### RN-14: Conversão Exata de Unidades de Medida
- **Definição:** Toda movimentação respeita a conversão matemática exata entre a unidade informada na operação e a **Unidade Base** do produto:
  $$\text{Quantidade Base} = \text{Quantidade Informada} \times \text{Fator de Conversão}$$
  $$\text{Custo Unitário Base} = \frac{\text{Custo Unitário da Embalagem}}{\text{Fator de Conversão}}$$
  *Exemplo Prático:* Produto `Shoyu Premium` tem Unidade Base = `L` (Litro). Na compra de `2 Galões` onde `1 Galão = 20 L` ao preço de `R$ 180,00 / Galão`:
  - Quantidade que entra no estoque = $2 \times 20 = 40,0000 \text{ L}$.
  - Custo por Litro para o cálculo do CMP = $180,00 / 20 = \text{R\$ } 9,0000 / \text{L}$.

---

### Regras Complementares de Food Service (RN-15 a RN-18)

- **RN-15 (Validade Secundária Pós-Abertura):** Ao acionar "Abrir/Manipular Lote" para uma fração $q$ de um lote fechado, a nova data de validade daquela fração passa a ser $\min(\text{Validade Original do Lote}, \text{Data Atual} + \text{Prazo Pós-Abertura em Dias})$.
- **RN-16 (Controle de Temperatura no Recebimento):** Produtos marcados com `storage_condition = 'REFRIGERATED'` (até 4°C a 7°C) ou `'FROZEN'` ($\le -12^\circ\text{C}$ a $-18^\circ\text{C}$) exibem campo para registro da temperatura aferida no ato da entrega na doca, emitindo alerta caso esteja fora da faixa sanitária segura.
- **RN-17 (Inventário Cego por Padrão):** Usuários com papel `STOCK_CLERK` (Estoquista) não visualizam a coluna `system_quantity` durante sessões de inventário com status `OPEN` ou `IN_PROGRESS`.
- **RN-18 (Cálculo Automático da Sugestão de Compra):** Para itens abaixo ou no Ponto de Reposição, a quantidade sugerida de compra considera pedidos já em andamento para evitar compras duplicadas:
  $$Q_{\text{sugerida}} = \max\left(0, \; \text{Estoque Máximo} - (\text{Estoque Disponível} + \text{Qtd em Pedidos Abertos})\right)$$
