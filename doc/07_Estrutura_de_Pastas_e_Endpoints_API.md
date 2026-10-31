# 07. Estrutura de Pastas e Especificação dos Endpoints da API REST

**Projeto:** Japa em Casa — Sistema Profissional de Gestão de Estoque para Restaurantes  
**Padrão de API:** RESTful JSON + OpenAPI 3.0 (Swagger)  
**Versionamento:** `/api/v1/...`

---

## 1. Estrutura de Pastas do Projeto (Monorepo ou Repositórios Coordenados)

Abaixo está a organização profissional de diretórios do **Backend (NestJS — Modular Monolith)** e do **Frontend (Next.js App Router — PWA)**:

```text
japa-em-casa-erp/
├── docker-compose.yml                  # Orquestração Postgres, Redis, Backend e Frontend
├── .github/workflows/ci-cd.yml         # Pipeline GitHub Actions (Lint, Test, Build)
├── doc/                                # Documentação Completa de Engenharia e Proposta Comercial
│
├── backend/                            # API NestJS (Modular Monolith + Clean Architecture)
│   ├── prisma/
│   │   ├── schema.prisma               # Modelagem Relacional Completa
│   │   ├── migrations/                 # Migrations SQL versionadas + Constraints CHECK
│   │   └── seed.ts                     # Seed das 12 Categorias, Unidades, Roles e Admin
│   ├── src/
│   │   ├── main.ts                     # Bootstrap, Swagger, Helmet, ValidationPipe, CORS
│   │   ├── app.module.ts               # Módulo Raiz
│   │   ├── common/                     # Recursos Transversais (Cross-Cutting Concerns)
│   │   │   ├── decorators/             # @CurrentUser(), @RequirePermissions()
│   │   │   ├── filters/                # GlobalExceptionFilter (Padronização RFC 7807)
│   │   │   ├── guards/                 # JwtAuthGuard, PermissionsGuard (RBAC)
│   │   │   ├── interceptors/           # AuditLogInterceptor, TransformResponseInterceptor
│   │   │   └── database/               # PrismaService & TransactionManager (Unit of Work)
│   │   │
│   │   └── modules/                    # Os 14 Módulos de Domínio Independentes
│   │       ├── auth/                   # 1. Authentication Module
│   │       ├── users/                  # 2. Users & RBAC Module
│   │       ├── categories/             # 3. Categories Module
│   │       ├── products/               # 4. Products & Units of Measure Module
│   │       │   ├── controllers/
│   │       │   ├── application/        # Use Cases & DTOs
│   │       │   ├── domain/             # Product Entity, CMP Calculator, Threshold Validator
│   │       │   └── infrastructure/     # PrismaProductRepository
│   │       ├── inventory/              # 5. Inventory Module (Saldos Consolidados e Locais)
│   │       ├── lots/                   # 6. Lots and Expiration Module (Algoritmo FEFO)
│   │       ├── stock-movements/        # 7. Stock Movements Module (Entradas, Saídas, Estornos)
│   │       ├── suppliers/              # 8. Suppliers Module
│   │       ├── purchases/              # 9. Purchases & Goods Receipt Module
│   │       ├── waste/                  # 10. Waste Management Module (Perdas e Desperdícios)
│   │       ├── inventory-counting/     # 11. Inventory Counting Module (Inventário Físico Cego)
│   │       ├── notifications/          # 12. Notifications & Alerts Module (BullMQ + Adapters)
│   │       ├── reports/                # 13. Reports Module (13 Relatórios + PDF/Excel)
│   │       └── audit/                  # 14. Audit Module
│   └── test/                           # Testes Unitários (Jest) e Integração (Supertest)
│
└── frontend/                           # Aplicação Web Responsiva PWA (Next.js + TypeScript)
    ├── public/
    │   ├── manifest.json               # Web App Manifest (Ícones, Standalone, Theme Color)
    │   └── icons/                      # Ícones PWA (192x192, 512x512, Maskable)
    ├── src/
    │   ├── sw.ts                       # Service Worker (Cache Offline & Background Sync)
    │   ├── app/                        # Next.js App Router
    │   │   ├── (auth)/login/           # Tela de Login
    │   │   └── (dashboard)/            # Layout Protegido (Sidebar + Topbar + Central Alertas)
    │   │       ├── page.tsx            # 3.1. Dashboard Gerencial
    │   │       ├── produtos/           # 3.2. Catálogo, Detalhes e Níveis de Estoque
    │   │       ├── lotes-validade/     # 3.4. Painel FEFO e Controle de Vencimentos
    │   │       ├── movimentacoes/      # 3.5. Entradas, Saídas Rápidas (Mobile) e Estornos
    │   │       ├── compras/            # 3.6. Sugestões, Pedidos de Compra e Recebimento NF
    │   │       ├── fornecedores/       # 3.7. Gestão de Fornecedores e Histórico de Preços
    │   │       ├── perdas/             # 3.8. Registro e Análise Financeira de Desperdícios
    │   │       ├── inventarios/        # 3.9. Contagem Física (Modo Cego) e Divergências
    │   │       ├── relatorios/         # 3.10. Central dos 13 Relatórios (PDF / Excel)
    │   │       ├── alertas/            # 4. Central de Notificações e Preferências
    │   │       └── configuracoes/      # 5. Usuários, Perfis RBAC, Locais e Auditoria
    │   ├── components/
    │   │   ├── ui/                     # Componentes base shadcn/ui
    │   │   ├── barcode-scanner/        # Leitor de código de barras via câmera
    │   │   ├── charts/                 # Gráficos Recharts do Dashboard
    │   │   └── stock-badges/           # Indicadores visuais de status e validade
    │   ├── hooks/                      # Custom Hooks TanStack Query (useProducts, useFEFO...)
    │   ├── lib/                        # Cliente HTTP Axios/Fetch com Refresh Token Interceptor
    │   └── schemas/                    # Esquemas de validação Zod em PT-BR
    └── e2e/                            # Testes End-to-End com Playwright
```

---

## 2. Especificação dos Principais Endpoints da API REST (`/api/v1`)

Todos os endpoints (exceto `/auth/login` e `/health`) exigem header `Authorization: Bearer <access_token>` e validam permissões RBAC. Listagens suportam paginação padronizada (`?page=1&limit=20&search=...&sortBy=...&sortOrder=asc|desc`).

### 2.1. Autenticação e Usuários (`AuthModule` & `UsersModule`)
| Método | Endpoint | Permissão | Descrição |
| :---: | :--- | :--- | :--- |
| `POST` | `/api/v1/auth/login` | Público | Autentica usuário e retorna `accessToken` + cookie `refreshToken` (`HttpOnly`). |
| `POST` | `/api/v1/auth/refresh` | Público (Cookie) | Rotaciona o `refreshToken` e emite novo `accessToken`. |
| `POST` | `/api/v1/auth/logout` | Autenticado | Revoga o `refreshToken` atual no banco. |
| `GET` | `/api/v1/auth/me` | Autenticado | Retorna dados do usuário logado, papel e lista de permissões ativas. |
| `GET` | `/api/v1/users` | `users:manage` | Lista usuários com filtro por papel e status. |
| `POST` | `/api/v1/users` | `users:manage` | Cria novo usuário atribuindo perfil RBAC. |
| `PATCH` | `/api/v1/users/:id` | `users:manage` | Atualiza dados, papel ou status (`isActive`) do usuário. |
| `GET` | `/api/v1/roles` | `users:manage` | Lista papéis e suas permissões configuradas. |
| `PUT` | `/api/v1/roles/:id/permissions` | `users:manage` | Atualiza matriz de permissões de um perfil. |

---

### 2.2. Produtos, Categorias, Unidades e Locais (`ProductsModule`, `CategoriesModule`, `InventoryModule`)
| Método | Endpoint | Permissão | Descrição |
| :---: | :--- | :--- | :--- |
| `GET` | `/api/v1/categories` | `products:read` | Lista árvore de categorias e subcategorias (as 12 nativas + personalizadas). |
| `POST` | `/api/v1/categories` | `catalog:manage` | Cria categoria ou subcategoria personalizada. |
| `GET` | `/api/v1/units` | `products:read` | Lista unidades de medida (`KG`, `G`, `L`, `ML`, `UN`, `CX`, `FD`, `PCT`). |
| `GET` | `/api/v1/locations` | `products:read` | Lista locais de armazenamento (Câmaras Frias, Estoque Seco, Sushibar, etc.). |
| `POST` | `/api/v1/locations` | `catalog:manage` | Cadastra novo local de armazenamento e faixa térmica. |
| `GET` | `/api/v1/products` | `products:read` | Lista produtos com filtros por categoria, status de estoque (`CRITICAL`, `BELOW_MIN`, `REORDER`, `OK`), fornecedor, condição térmica e busca por Nome/SKU/Código de Barras. |
| `GET` | `/api/v1/products/:id` | `products:read` | Detalhes completos do produto: saldos por local, lotes ativos (ordenados por FEFO), CMP, fornecedores vinculados e últimas movimentações. |
| `GET` | `/api/v1/products/barcode/:code` | `products:read` | Busca rápida por leitura de código de barras (câmera/scanner). |
| `POST` | `/api/v1/products` | `products:write` | Cadastra novo produto com parâmetros de estoque mínimo, máximo, segurança e ponto de reposição. |
| `PATCH` | `/api/v1/products/:id` | `products:write` | Atualiza dados cadastrais ou limites de estoque do produto (gera `AuditLog`). |

---

### 2.3. Controle de Lotes e Validade — FEFO (`LotsModule`)
| Método | Endpoint | Permissão | Descrição |
| :---: | :--- | :--- | :--- |
| `GET` | `/api/v1/lots` | `products:read` | Lista lotes com filtros por janela de vencimento (`EXPIRED`, `NEXT_3_DAYS`, `NEXT_7_DAYS`, `NEXT_15_DAYS`, `NEXT_30_DAYS`), local e produto. |
| `GET` | `/api/v1/lots/fefo-preview` | `movements:create_out` | **Preview FEFO:** Recebe `?productId=...&quantity=8.5` e retorna exatamente quais lotes (e em quais quantidades) serão consumidos segundo a regra FEFO. |
| `POST` | `/api/v1/lots/:id/open` | `movements:create_out` | Registra abertura/manipulação de embalagem fechada, ajustando a validade secundária conforme `afterOpenShelfLifeDays`. |

---

### 2.4. Movimentações de Estoque (`StockMovementsModule`)
| Método | Endpoint | Permissão | Descrição |
| :---: | :--- | :--- | :--- |
| `GET` | `/api/v1/movements` | `products:read` | Histórico paginado de movimentações com filtros por período, tipo, produto, lote, usuário e local. |
| `POST` | `/api/v1/movements/out` | `movements:create_out` | Registra **Saída para Produção** ou **Saída por Consumo** aplicando alocação automática FEFO. |
| `POST` | `/api/v1/movements/return` | `movements:create_in` | Registra **Entrada por Devolução** de insumo intacto da cozinha para o estoque. |
| `POST` | `/api/v1/movements/transfer` | `movements:transfer` | Registra **Transferência entre Locais de Armazenamento** mantendo lote e validade. |
| `POST` | `/api/v1/movements/:id/reverse` | `movements:reverse` | Realiza **Estorno Auditável** de uma movimentação, exigindo motivo obrigatório e recompondo saldos. |

#### Exemplo de Payload: `POST /api/v1/movements/out` (Saída com FEFO)
```json
{
  "productId": "550e8400-e29b-41d4-a716-446655440000",
  "type": "SAIDA_PRODUCAO",
  "quantity": 6.5,
  "originLocationId": "7c9e6679-7425-40de-944b-e07fc1f90ae7",
  "costCenter": "Sushibar",
  "reason": "Preparação do turno da noite (Rodízio e Delivery)",
  "useAutomaticFefo": true
}
```

---

### 2.5. Fornecedores e Gestão de Compras (`SuppliersModule` & `PurchasesModule`)
| Método | Endpoint | Permissão | Descrição |
| :---: | :--- | :--- | :--- |
| `GET` | `/api/v1/suppliers` | `products:read` | Lista fornecedores ativos/inativos. |
| `POST` | `/api/v1/suppliers` | `suppliers:manage` | Cadastra fornecedor com validação de CNPJ/CPF. |
| `POST` | `/api/v1/suppliers/:id/products` | `suppliers:manage` | Vincula produto ao catálogo do fornecedor informando unidade de compra e fator de conversão. |
| `GET` | `/api/v1/purchases/suggestions` | `purchases:create` | **Reposição Automática:** Lista produtos no Ponto de Reposição / Abaixo do Mínimo com sugestão automática de quantidade de compra e melhor fornecedor. |
| `GET` | `/api/v1/purchases` | `purchases:create` | Lista pedidos de compra filtrados por status e fornecedor. |
| `POST` | `/api/v1/purchases` | `purchases:create` | Cria Pedido de Compra (`RASCUNHO` ou `AGUARDANDO_APROVACAO`). |
| `POST` | `/api/v1/purchases/:id/approve` | `purchases:approve` | Gerente aprova Pedido de Compra (`APROVADO`). |
| `POST` | `/api/v1/purchases/:id/send` | `purchases:create` | Marca pedido como `ENVIADO_FORNECEDOR`. |
| `POST` | `/api/v1/purchases/:id/receive` | `movements:create_in` | **Recebimento Físico de Mercadoria:** Registra NF, lotes, datas de validade e quantidades reais entregues; calcula divergências, cria lotes, gera `ENTRADA_COMPRA` e atualiza o CMP. |

---

### 2.6. Perdas, Inventário, Alertas, Dashboard e Relatórios
| Método | Endpoint | Permissão | Descrição |
| :---: | :--- | :--- | :--- |
| `POST` | `/api/v1/waste` | `waste:create` | Registra perda/desperdício (`VENCIMENTO`, `DETERIORACAO`, `ERRO_MANIPULACAO`, etc.), baixa o lote e calcula impacto financeiro pelo CMP. |
| `GET` | `/api/v1/waste/analytics` | `waste:read_financial` | Retorna agregados financeiros de perdas por motivo, categoria e produto. |
| `POST` | `/api/v1/inventory-counts` | `inventory:open` | Abre sessão de inventário físico (completo ou parcial por categoria/local). |
| `POST` | `/api/v1/inventory-counts/:id/items` | `inventory:count` | Estoquista submete as quantidades físicas contadas (Inventário Cego). |
| `POST` | `/api/v1/inventory-counts/:id/approve` | `inventory:approve` | Gerente aprova conferência, justificando divergências e gerando ajustes automáticos no estoque. |
| `GET` | `/api/v1/dashboard/summary` | `dashboard:view_operational` | Retorna todos os KPIs, gráficos e alertas prioritários do Dashboard Gerencial. |
| `GET` | `/api/v1/notifications` | Autenticado | Lista alertas filtrados por prioridade (`CRITICO`, `ALTO`, `MEDIO`, `INFORMATIVO`) e status de leitura. |
| `PATCH` | `/api/v1/notifications/:id/read` | Autenticado | Marca notificação como lida. |
| `GET` | `/api/v1/reports/:reportType` | `reports:export` | Consulta dados ou exporta qualquer um dos **13 Relatórios Gerenciais** passando `?format=json|pdf|xlsx`. |
| `GET` | `/api/v1/audit-logs` | `audit:read` | Consulta trilha completa de auditoria do sistema. |
| `GET` | `/api/v1/health` | Público | Health check da aplicação (status do PostgreSQL, Redis e memória). |
