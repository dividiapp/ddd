
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
- **Liquidação Pix (Conciliação)**
- **Ledger & Saldos**
- **Notificações & Lembretes**
- **Recorrências & Fechamento**
- **IAM (Autenticação & Autorização)**
- **Compliance & Auditoria**
- **Relatórios & Exportações**
- **Observabilidade**
- **Onboarding & Templates**
- **Planos & Entitlements**
- **Armazenamento de Arquivos**
- **Analytics de Produto**
- **OCR & Leitura de Recibos**

#### **2.2 Padrões de Context Mapping utilizados**
- **Customer–Supplier**
- **Conformist**
- **Shared Kernel** *(não aplicado neste mapeamento para manter fronteiras bem isoladas)*
- **Anticorruption Layer (ACL)**
- **Published Language** (eventos e contratos estáveis)
- **Open-Host Service** (API pública com contrato claro)
- **Partnership** (codependência colaborativa e alinhamento frequente)
- **Separate Ways** (baixo acoplamento; evoluem em paralelo)
- **Upstream–Downstream** (quem define contrato/linguagem e quem consome)

#### **2.3 Relações entre Contextos**
| **Origem**                           | **Destino**                           | **Tipo de Relacionamento**                                | **Explicação** |
|-------------------------------------|---------------------------------------|-----------------------------------------------------------|----------------|
| **Grupos & Despesas**               | **Rateio**                            | **Customer–Supplier + Published Language**                | Despesas criadas/atualizadas em Grupos geram eventos (`DespesaCriada`, `DespesaAtualizada`) consumidos por Rateio para calcular cotas. |
| **Rateio**                          | **Ledger & Saldos**                   | **Upstream–Downstream + Published Language**              | Rateio publica `CotasCalculadas`; Ledger materializa lançamentos mantendo soma-zero e auditabilidade. |
| **Liquidação Pix (Conciliação)**    | **Ledger & Saldos**                   | **Upstream–Downstream + Published Language**              | Liquidação emite `PagamentoLiquidado`/`PagamentoRevertido`; Ledger registra entradas imutáveis. |
| **Recorrências & Fechamento**       | **Rateio**                            | **Customer–Supplier + Published Language**                | Fechamento dispara ciclos (`CicloEncerrado`) e/ou gera despesas/ajustes; Rateio recalcula. |
| **Recorrências & Fechamento**       | **Liquidação Pix (Conciliação)**      | **Customer–Supplier**                                     | Dispara geração de cobranças do período (comandos síncronos). |
| **Grupos & Despesas**               | **Liquidação Pix (Conciliação)**      | **Customer–Supplier**                                     | Solicita quitação de despesa/saldo; Liquidação orquestra conciliação Pix. |
| **Liquidação Pix (Conciliação)**    | **Notificações & Lembretes**          | **Upstream–Downstream + Published Language**              | Evento `PagamentoConfirmado` aciona lembretes/avisos aos participantes. |
| **Ledger & Saldos**                 | **Relatórios & Exportações**          | **Customer–Supplier + Published Language**                | Relatórios consomem projeções/saldos e eventos `LancamentoRegistrado` para gerar documentos. |
| **Relatórios & Exportações**        | **Armazenamento de Arquivos**         | **Customer–Supplier (Open-Host Service)**                 | Envia artefatos (CSV/PDF) a uma API/SDK estável de storage e recebe ponteiro/URL. |
| **Armazenamento de Arquivos**       | **Relatórios & Exportações**          | **Published Language**                                     | Evento `ArquivoDisponivel { fileId, url }` notifica conclusão de upload. |
| **OCR & Leitura de Recibos**        | **Grupos & Despesas**                 | **Customer–Supplier + Published Language**                | Evento `ReciboInterpretado` cria *drafts* de despesa para confirmação pelo usuário. |
| **Onboarding & Templates**          | **Grupos & Despesas**                 | **Customer–Supplier (Open-Host Service)**                 | Criação de grupos a partir de *templates* via API clara e estável. |
| **IAM (AuthN/AuthZ)**               | **Todos os Contextos**                | **Open-Host Service + Conformist + Published Language**   | JWT/claims (`tenantId`, `scopes`) padronizados; os demais contextos **conformam-se**. |
| **Planos & Entitlements**           | **Todos os Contextos**                | **Open-Host Service + Conformist**                        | Checagem de direitos/limites e feature flags; consumidores seguem o contrato do provedor. |
| **Compliance & Auditoria**          | **Todos os Contextos**                | **Upstream–Downstream + Published Language**              | Todos publicam `Audit.EventoRegistrado` com `actorId`, `ação`, `recurso`, `traceId`. |
| **Observabilidade**                 | **Todos os Contextos**                | **Separate Ways + Published Language (telemetria)**       | Telemetria (OpenTelemetry) é coletada sem contaminar o domínio. |
| **Analytics de Produto**            | **Todos os Contextos**                | **Separate Ways + Published Language (eventos de produto)**| Coleta de eventos de uso/engajamento separados do domínio de negócio. |
| **Rateio** ↔ **Liquidação Pix**     | **Partnership + Published Language**                      | **Alinhamento fino** (ex.: arredondamentos/centavos) para garantir que o valor cobrado é exatamente o valor calculado. |
| **Liquidação Pix (Conciliação)**    | **PSPs/SPI (externo)**                | **Anticorruption Layer (ACL)**                            | Adapters protegem o modelo do domínio de variações dos provedores Pix. |
| **OCR & Leitura de Recibos**        | **Provedor OCR (externo)**            | **Anticorruption Layer (ACL)**                            | Tradução de formatos/ruídos do provedor para o modelo interno. |

#### **2.4 Diagrama no Draw.io (LucidChart)**

- [DRAWIO]([https://miro.com/](https://lucid.app/lucidchart/46c1c0f6-7610-4f50-b6bd-768f6dff1bbb/edit?viewport_loc=58%2C-340%2C3186%2C1572%2C0_0&invitationId=inv_8a3f5ef5-0312-4d76-9fe2-2efb6464d44e))  

#### **2.5 Justificativas das escolhas**
- **Published Language (PL):** eventos padronizados (DespesaCriada, PagamentoLiquidado, etc.) reduzem ambiguidade e permitem evolução independente.

- **Customer–Supplier / Upstream–Downstream:** clareza sobre quem fornece e quem consome contratos.

- **Open-Host Service:** IAM, Entitlements e Storage expõem APIs estáveis; consumidores se conformam.

- **ACL:** protege o domínio de variações externas (PSPs Pix, OCR).

- **Separate Ways:** Observabilidade e Analytics evoluem em paralelo, sem poluir o modelo de negócio.

- **Partnership:** Rateio e Liquidação mantêm alinhamento próximo para evitar inconsistências financeiras.


## 📌 Aula 3: Próximos Passos  
Na próxima aula, vamos explorar **Design Tático**, abordando:  
🔹 **Entidades vs. Value Objects** – Como diferenciar e modelar corretamente.  
🔹 **Agregados** – Como definir o agregado raiz e garantir consistência.  
🔹 **Repositórios** – Como separar persistência da lógica de domínio.  

📌 **Prepare-se!** Tente aplicar **Context Mapping** no seu projeto antes da próxima aula.  

---

**📢 Bom trabalho! Nos vemos na próxima aula! 🚀**  

