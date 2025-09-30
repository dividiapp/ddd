
# 📚 Design Estratégico do Projeto

## 📌 Aula 1: Introdução ao Domain-Driven Design (DDD)

### **1️⃣ Revisão da Aula**
- O que é **Domain-Driven Design (DDD)**?
- Diferença entre **Complexidade Essencial vs. Complexidade Acidental**.
- **Subdomínios**: Core Domain, Supporting Subdomains e Generic Subdomains.
- **Bounded Contexts**: Separando conceitos e linguagens dentro do domínio.

### **2️⃣ Identificação dos Subdomínios (Projeto: DIVIDI)**
| **Subdomínio**             | **Descrição**                                                                                                                                                                                                                      | **Tipo**         |
|----------------------------|------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|------------------|
| Gestão de Grupos           | Criação e administração de grupos e membros; registro de despesas e participações (quem entra em cada despesa); emite eventos para cálculo de rateio e quitação.                                                                   | Core Domain      |
| Motor de Rateio            | Aplica estratégias de divisão (igual, por item, percentual, participação) e ajustes; valida que a soma das cotas = total; produz cotas por membro.                                                                                 | Core Domain      |
| Liquidação via PIX First   | Concilia pagamentos Pix (txid/endToEndId/valor/referência) com expectativas de pagamento; registra quitações com idempotência; não custodial; integra com PSP via ACL.                                                             | Core Domain      |
| Saldos e Liquidações       | Mantém o _ledger_ (entradas imutáveis) e projeta saldos por grupo; registra quitações/reversões vindas da liquidação; garante soma-zero e auditabilidade do histórico.                                                             | Core Domain      |
| Notificações e Lembretes   | Define políticas e textos “neutros/gentis”, cadência e janelas silenciosas; orquestra lembretes (WhatsApp-first) e acompanha confirmações de envio/leitura.                                                                        | Core Domain      |
| Recorrências e Fechamento  | Gera despesas recorrentes (moradia) conforme agenda/regras; controla fechamento/reabertura de períodos e gatilhos de cobrança.                                                                                                     | Core Domain      |
| Compliance e Auditoria     | LGPD, retenção e descarte de dados, trilhas de auditoria e governança; suporte a investigações e atendimento a solicitações de titulares.                                                                                          | Supporting       |
| Relatórios e Exportações   | Relatórios de grupo/período, prestação de contas e exportações (CSV/PDF); consome projeções do ledger e status de quitação.                                                                                                        | Supporting       |
| Observabilidade            | Logs, métricas, tracing, alertas e SLOs; saúde e performance do sistema; suporte a diagnósticos e capacidade.                                                                                                                      | Supporting       |
| Onboarding                 | Fluxos de primeiro uso, templates por tipo de grupo (viagem/moradia/evento), dicas/guias; acelera ativação e entendimento do produto.                                                                                              | Supporting       |
| Plano de Assinatura        | Gestão de planos (Free/Pro), limites e direito de uso (entitlements); integração de billing com provedores; controle de feature flags por plano.                                                                                   | Supporting       |
| Armazenamento de Arquivos  | Upload e storage de anexos/comprovantes e imagens (CDN/e-mail transacional quando necessário); uso de serviços de mercado.                                                                                                         | Generic          |
| Analytics de Produto       | Telemetria de uso, funis de ativação e retenção; instrumentação de eventos e dashboards via ferramentas SaaS.                                                                                                                      | Generic          |
| OCR/Leitura de Recibos     | Extração de itens/valores de recibos para sugerir despesas; serviço de visão computacional de terceiros encapsulado por ACL.                                                                                                       | Generic          |
| Autenticação e Autorização | Gerencia identidade e acesso (IAM): cadastro/login, MFA/SSO, emissão/validação de tokens, gestão de papéis (owner, co-admin, membro) e permissões/escopos por grupo, além de políticas de sessão e segurança.                     | Generic          |

---

## 📌 Aula 2: Mapeamento de Contextos (Context Mapping)

### **1️⃣ Objetivo da Aula**
Nesta aula, vamos:
✅ Explorar como **Bounded Contexts** se relacionam entre si.  
✅ Aplicar **Context Mapping** para visualizar dependências entre contextos.  
✅ Criar um **diagrama de Context Mapping** para um projeto.  

---

### **2️⃣ Atividade Prática: Context Mapping no Projeto (DIVIDI)**

#### **2.1 Bounded Contexts do DIVIDI**
- **Grupos & Despesas**
- **Rateio**
- **Liquidação Pix**
- **Ledger & Saldos**
- **Notificações & Lembretes**
- **Recorrências & Fechamento**
- **OCR & Recibos**
- **Provedor OCR (Externo)**
- **PSPs / SPI Pix (Externo)**

#### **2.2 Padrões de Context Mapping utilizados**
- **Customer–Supplier**
- **Conformist**
- **Anticorruption Layer (ACL)**
- **Partnership** (codependência colaborativa e alinhamento frequente)
- **Separate Ways** (baixo acoplamento; evoluem em paralelo)
- **Upstream–Downstream** (quem define contrato/linguagem e quem consome)

#### **2.3 Relações entre Contextos**
| **Origem**                     | **Destino**                   | **Tipo de Relacionamento**        | **Explicação** |
|--------------------------------|-------------------------------|-----------------------------------|----------------|
| **OCR & Recibos**              | **Grupos & Despesas**         | **Customer–Supplier**             | OCR envia evento `ReciboInterpretado` que gera drafts de despesas. |
| **OCR & Recibos**              | **Provedor OCR (Externo)**    | **Anticorruption Layer (ACL)**    | Tradução do resultado externo para o modelo interno. |
| **Grupos & Despesas**          | **Rateio**                    | **Customer–Supplier**             | Eventos de criação/atualização de despesas (`DespesaCriada`) são consumidos por Rateio. |
| **Grupos & Despesas**          | **Liquidação Pix**            | **Customer–Supplier**             | Solicita quitação de despesa/saldo. |
| **Rateio**                     | **Ledger & Saldos**           | **Upstream–Downstream**           | Rateio gera `CotasCalculadas` consumidas pelo Ledger. |
| **Liquidação Pix**             | **Ledger & Saldos**           | **Upstream–Downstream**           | Liquidação emite `PagamentoLiquidado` que atualiza Ledger. |
| **Liquidação Pix**             | **Notificações & Lembretes**  | **Upstream–Downstream**           | Liquidação confirma pagamento e dispara `PagamentoConfirmado`. |
| **Liquidação Pix**             | **PSPs / SPI Pix (Externo)**  | **Anticorruption Layer (ACL)**    | Protege o domínio de integrações com provedores Pix. |
| **Recorrências & Fechamento**  | **Rateio**                    | **Customer–Supplier**             | Fechamento dispara ciclos (`CicloEncerrado`). |
| **Recorrências & Fechamento**  | **Liquidação Pix**            | **Customer–Supplier**             | Gera cobranças para liquidação. |

#### **2.4 Diagrama no Draw.io (LucidChart)**

- [Acessar diagrama no LucidChart](https://lucid.app/lucidchart/46c1c0f6-7610-4f50-b6bd-768f6dff1bbb/edit?viewport_loc=58%2C-340%2C3186%2C1572%2C0_0&invitationId=inv_8a3f5ef5-0312-4d76-9fe2-2efb6464d44e)  

#### **2.5 Justificativas das escolhas**
- **Customer–Supplier:** usado quando um contexto depende do outro para executar sua função (ex.: Grupos envia despesas para Rateio).  
- **Upstream–Downstream:** o contexto upstream define os contratos/eventos que o downstream consome (ex.: Rateio → Ledger).  
- **ACL:** usado nos limites com sistemas externos (OCR e PSPs Pix).  
- **Partnership:** pode ser aplicado em pontos de alinhamento crítico (ex.: Rateio ↔ Liquidação, se houver dependência forte de valores).  
- **Separate Ways:** reservado para contextos de telemetria/analytics (não incluídos no diagrama simplificado).  
- **Conformist:** pode aparecer em integrações onde o consumidor adota integralmente o modelo do fornecedor, mas não foi evidenciado no fluxograma atual.  


