# Índice Geral da Documentação (Fase 1 — Planejamento e Engenharia)

**Projeto:** Japa em Casa — Sistema Profissional de Gestão de Estoque para Restaurantes (ERP Food Service)  
**Diretório:** `/doc`  
**Status Atual:** Fase 1 Concluída — Aguardando Validação para Início da Fase 2 (Implementação)

---

## Mapa dos 15 Entregáveis Obrigatórios + Documento Comercial

Todos os entregáveis técnicos, arquiteturais, funcionais e comerciais foram elaborados e encontram-se organizados nos arquivos abaixo:

| # Entregável (Prompt Master) | Arquivo Correspondente na Pasta `doc/` | Conteúdo Principal |
| :--- | :--- | :--- |
| **1. Visão Geral do Produto + Análise Crítica** | [01_Visao_Geral_do_Produto.md](file:///C:/Users/raiim/Desktop/Japa%20em%20casa/doc/01_Visao_Geral_do_Produto.md) | Proposta de valor, perfis de uso, análise crítica de lacunas (conversão de unidades, validade pós-abertura, tolerância na doca, inventário cego, concorrência) e KPIs de sucesso. |
| **2. Requisitos Funcionais e Não Funcionais** | [02_Requisitos_Funcionais_e_Nao_Funcionais.md](file:///C:/Users/raiim/Desktop/Japa%20em%20casa/doc/02_Requisitos_Funcionais_e_Nao_Funcionais.md) | Catálogo completo codificado (`RF-AUTH`, `RF-PROD`, `RF-STK`, `RF-LOT`, `RF-MOV`, `RF-SUP`, `RF-PUR`, `RF-WST`, `RF-INV`, `RF-DSH`, `RF-NOT`, `RF-REP`) e SLAs mensuráveis (`RNF`). |
| **3. Identificação dos Atores e Matriz RBAC**<br>**4. Diagrama de Casos de Uso UML**<br>**5. Descrição Textual dos Casos de Uso** | [03_Atores_e_Casos_de_Uso_UML.md](file:///C:/Users/raiim/Desktop/Japa%20em%20casa/doc/03_Atores_e_Casos_de_Uso_UML.md) | Definição dos 4 atores humanos + Worker sistêmico, Matriz de Permissões RBAC granular, Diagrama de Casos de Uso UML em Mermaid e fluxos principais/alternativos/exceção detalhados. |
| **6. Regras de Negócio Detalhadas** | [04_Regras_de_Negocio_Detalhadas.md](file:///C:/Users/raiim/Desktop/Japa%20em%20casa/doc/04_Regras_de_Negocio_Detalhadas.md) | Especificação matemática e transacional das 14 regras obrigatórias + 4 regras complementares (fórmula do CMP, algoritmo FEFO, invariantes de saldo, matriz de prioridades de alertas). |
| **7. Diagrama de Arquitetura e Stack** | [05_Arquitetura_de_Software_e_Stack.md](file:///C:/Users/raiim/Desktop/Japa%20em%20casa/doc/05_Arquitetura_de_Software_e_Stack.md) | Arquitetura de Monólito Modular (14 módulos NestJS), Clean Architecture em 4 camadas, eventos internos de domínio, filas BullMQ/Redis e justificativa de toda a stack tecnológica. |
| **8. Diagrama Entidade-Relacionamento (DER)**<br>**9. Modelo Lógico do Banco de Dados** | [06_Modelagem_Banco_de_Dados_DER_e_Logico.md](file:///C:/Users/raiim/Desktop/Japa%20em%20casa/doc/06_Modelagem_Banco_de_Dados_DER_e_Logico.md) | DER completo em Mermaid, 24 entidades em 3FN, Schema Prisma completo com tipos `Decimal(12,4)`, índices compostos FEFO e constraints SQL `CHECK`. |
| **10. Estrutura de Pastas (Frontend/Backend)**<br>**11. Especificação dos Endpoints da API** | [07_Estrutura_de_Pastas_e_Endpoints_API.md](file:///C:/Users/raiim/Desktop/Japa%20em%20casa/doc/07_Estrutura_de_Pastas_e_Endpoints_API.md) | Árvore de diretórios do Backend (NestJS) e Frontend (Next.js PWA) e catálogo de endpoints REST (`/api/v1/...`) com verbos, permissões e payloads. |
| **12. Fluxos das Principais Operações**<br>**13. Definição das Telas e Navegação (UX/UI)** | [08_Fluxos_Operacionais_e_Definicao_de_Telas_UX.md](file:///C:/Users/raiim/Desktop/Japa%20em%20casa/doc/08_Fluxos_Operacionais_e_Definicao_de_Telas_UX.md) | Diagramas de sequência (Recebimento com Divergência + CMP e Baixa FEFO Multi-Lote), Máquina de Estados de Compras e especificação das 11 telas responsivas (Desktop, Tablet e Smartphone). |
| **14. Estratégia de Testes**<br>**15. Plano de Implantação e Manutenção** | [09_Estrategia_de_Testes_Seguranca_e_Implantacao.md](file:///C:/Users/raiim/Desktop/Japa%20em%20casa/doc/09_Estrategia_de_Testes_Seguranca_e_Implantacao.md) | Pirâmide de testes (Jest, Supertest com concorrência e Playwright), segurança OWASP/LGPD, `docker-compose.yml`, pipeline GitHub Actions e política de Backup/DRP. |
| **⭐ EXTRA: Documento Comercial para Clientes** | [10_Proposta_Comercial_e_Apresentacao_Clientes.md](file:///C:/Users/raiim/Desktop/Japa%20em%20casa/doc/10_Proposta_Comercial_e_Apresentacao_Clientes.md) | Apresentação executiva e comercial em linguagem de negócios pronta para enviar a clientes, sócios e donos de restaurantes. |
