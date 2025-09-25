# Dinâmica: Design Estratégico do Projeto

## Objetivo

Identificar os subdomínios do projeto, classificá-los (Core, Supporting, Generic) e desenhar os bounded contexts, incluindo suas interações. Esse exercício ajudará a criar uma visão clara e estratégica do domínio.

---

## 1. Nome do Projeto

**DIVIDI**

---

## 2. Objetivo Principal do Projeto

**Organizar a divisão de despesas em grupo seja em viagens, moradias compartilhadas ou eventos.**

---

## 3. Identificação dos Subdomínios

Liste os subdomínios do sistema e classifique-os como **Core Domain**, **Supporting Subdomain** ou **Generic Subdomain**.

| **Subdomínio**             | **Descrição**                                                                                                                                                                                                                      | **Tipo**    |
| -------------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ----------- |
| Gestão de Grupos           | Criação e administração de grupos e membros; registro de despesas e participações (quem entra em cada despesa); emite eventos para cálculo de rateio e quitação.                                                                   | Core Domain |
| Motor de Rateio            | Aplica estratégias de divisão (igual, por item, percentual, participação) e ajustes; valida que a soma das cotas = total; produz cotas por membro.                                                                                 | Core Domain |
| Liquidação via PIX First   | Concilia pagamentos Pix (txid/endToEndId/valor/referência) com expectativas de pagamento; registra quitações com idempotência; não custodial; integra com PSP via ACL.                                                             | Core Domain |
| Saldos e Liquidações       | Mantém o _ledger_ (entradas imutáveis) e projeta saldos por grupo; registra quitações/reversões vindas da liquidação; garante soma-zero e auditabilidade do histórico.                                                             | Core Domain |
| Notificações e Lembretes   | Define políticas e textos “neutros/gentis”, cadência e janelas silenciosas; orquestra lembretes (WhatsApp-first) e acompanha confirmações de envio/leitura.                                                                        | Core Domain |
| Recorrências e Fechamento  | Gera despesas recorrentes (moradia) conforme agenda/regras; controla fechamento/reabertura de períodos e gatilhos de cobrança.                                                                                                     | Core Domain |
| Compliance e Auditoria     | LGPD, retenção e descarte de dados, trilhas de auditoria e governança; suporte a investigações e atendimento a solicitações de titulares.                                                                                          | Supporting  |
| Relatórios e Exportações   | Relatórios de grupo/período, prestação de contas e exportações (CSV/PDF); consome projeções do ledger e status de quitação.                                                                                                        | Supporting  |
| Observabilidade            | Logs, métricas, tracing, alertas e SLOs; saúde e performance do sistema; suporte a diagnósticos e capacidade.                                                                                                                      | Supporting  |
| Onboarding                 | Fluxos de primeiro uso, templates por tipo de grupo (viagem/moradia/evento), dicas/guias; acelera ativação e entendimento do produto.                                                                                              | Supporting  |
| Plano de Assinatura        | Gestão de planos (Free/Pro), limites e direito de uso (entitlements); integração de billing com provedores; controle de feature flags por plano.                                                                                   | Supporting  |
| Armazenamento de Arquivos  | Upload e storage de anexos/comprovantes e imagens (CDN/e-mail transacional quando necessário); uso de serviços de mercado.                                                                                                         | Generic     |
| Analytics de Produto       | Telemetria de uso, funis de ativação e retenção; instrumentação de eventos e dashboards via ferramentas SaaS.                                                                                                                      | Generic     |
| OCR/Leitura de Recibos     | Extração de itens/valores de recibos para sugerir despesas; serviço de visão computacional de terceiros encapsulado por ACL.                                                                                                       | Generic     |
| Autenticação e Autorização | Gerencia identidade e acesso (IAM): cadastro/login, recuperação de senha, MFA/SSO, emissão/validação de tokens, gestão de papéis (owner, co-admin, membro) e permissões/escopos por grupo, além de políticas de sessão e segurança | Generic     |

---

## 4. Desenho dos Bounded Contexts

Liste e descreva os bounded contexts identificados no projeto. Explique a responsabilidade de cada um.

| **Bounded Context**                              | **Responsabilidade**                                                                                                                                                         | **Subdomínios Relacionados** |
| ------------------------------------------------ | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ---------------------------- |
| **Contexto de Grupos e Despesas**                | Gerir grupos, membros, despesas e participações; emitir eventos de criação/fechamento de despesas; base para cálculo de rateio e posterior quitação.                         | Gestão de Grupos             |
| **Contexto de Rateio**                           | Aplicar estratégias de divisão (igual, por item, percentual, participação) e ajustes; garantir que a soma das cotas = total; produzir cotas por membro.                      | Motor de Rateio              |
| **Contexto de Liquidação Pix (Conciliação)**     | Conciliação **Pix-first**: mapear pagamentos (txid/endToEndId/valor/referência) para expectativas de pagamento; registrar quitações com idempotência; operar sem custódia.   | Liquidação via PIX First     |
| **Contexto de Ledger e Saldos**                  | Manter o **ledger** (entradas imutáveis) e projetar saldos por grupo; registrar quitações/reversões vindas da liquidação; garantir **soma-zero** e auditabilidade.           | Saldos e Liquidações         |
| **Contexto de Notificações e Lembretes**         | Definir políticas e textos “neutros/gentis”, cadência e janelas silenciosas; orquestrar lembretes (WhatsApp-first) e acompanhar confirmação/entrega/leitura.                 | Notificações e Lembretes     |
| **Contexto de Recorrências e Fechamento**        | Gerar despesas recorrentes (moradia) conforme agenda/regras; controlar fechamento/reabertura de períodos e disparar gatilhos de cobrança/ajustes.                            | Recorrências e Fechamento    |
| **Contexto de Autenticação e Autorização (IAM)** | Gerenciar identidade e acesso: cadastro/login, MFA/SSO, emissão/validação de tokens; papéis (owner, co-admin, membro) e permissões por grupo; políticas de sessão/segurança. | Autenticação e Autorização   |
| **Contexto de Compliance e Auditoria**           | LGPD (consentimento, retenção, descarte), trilhas de auditoria e governança; suporte a investigações e atendimento a solicitações de titulares de dados.                     | Compliance e Auditoria       |
| **Contexto de Relatórios e Exportações**         | Geração de relatórios/CSV/PDF de prestação de contas; consumo de projeções do ledger e status de quitação; filtros por grupo/período/categoria.                              | Relatórios e Exportações     |
| **Contexto de Observabilidade**                  | Logs, métricas e tracing; alertas e SLOs; saúde e performance do sistema; suporte a diagnósticos e planejamento de capacidade.                                               | Observabilidade              |
| **Contexto de Onboarding e Templates**           | Fluxos de primeiro uso; templates por tipo de grupo (viagem/moradia/evento); dicas/guias para ativação; auxílio na configuração inicial de regras/recorrências.              | Onboarding                   |
| **Contexto de Planos e Entitlements**            | Gestão de planos (Free/Pro), limites e direitos de uso; integração com billing; controle de feature flags por plano e auditoria de consumo.                                  | Plano de Assinatura          |
| **Contexto de Armazenamento de Arquivos**        | Upload e storage de anexos/comprovantes/imagens; integração com CDN e e-mail transacional quando necessário; versionamento/expiração de objetos.                             | Armazenamento de Arquivos    |
| **Contexto de Analytics de Produto**             | Telemetria de uso, funis de ativação/retensão e dashboards; instrumentação de eventos de produto via ferramentas SaaS; export/ETL quando aplicável.                          | Analytics de Produto         |
| **Contexto de OCR e Leitura de Recibos**         | Extração de itens/valores de recibos para sugerir despesas; processamento síncrono/assíncrono; encapsulamento de provedores de visão computacional através de ACL.           | OCR/Leitura de Recibos       |

---

## 5. Comunicação entre os Bounded Contexts

Explique como os bounded contexts vão se comunicar. Use os padrões de comunicação, como:

- **Mensageria/Eventos (desacoplado):** Ex.: O Contexto de Consultas emite um evento "Consulta Finalizada", consumido pelo Contexto de Pagamentos.
- **APIs (síncrono):** Ex.: O Contexto de Pagamentos consulta informações de preços no Contexto de Consultas.

| De (Origem)                                  | Para (Destino)                           | Forma de Comunicação | Exemplo de Evento/Chamada                                             |
| -------------------------------------------- | ---------------------------------------- | -------------------- | --------------------------------------------------------------------- |
| Contexto de Grupos e Despesas                | Contexto de Rateio                       | Evento               | `Grupos.DespesaCriada { expenseId, groupId, valores, participantes }` |
| Contexto de Grupos e Despesas                | Contexto de Rateio                       | Evento               | `Grupos.DespesaAtualizada { expenseId, … }`                           |
| Contexto de Rateio                           | Contexto de Ledger e Saldos              | Evento               | `Rateio.SaldoRecalculado { groupId, saldos[] }`                       |
| Contexto de Rateio                           | Contexto de Liquidação Pix (Conciliação) | API síncrona         | `POST /liquidacoes { groupId, payerId, payees[], valores[] }`         |
| Contexto de Liquidação Pix (Conciliação)     | Contexto de Ledger e Saldos              | Evento               | `Pix.PagamentoLiquidado { settlementId, payerId, payeeId, valor }`    |
| Contexto de Liquidação Pix (Conciliação)     | Contexto de Notificações e Lembretes     | Evento               | `Pix.PagamentoConfirmado { payerId, payeeId, valor }`                 |
| Contexto de Ledger e Saldos                  | Contexto de Relatórios e Exportações     | Evento               | `Ledger.LancamentoRegistrado { lancamentoId, tipo, valor }`           |
| Contexto de Ledger e Saldos                  | Contexto de Grupos e Despesas            | API síncrona         | `GET /grupos/{id}/saldos`                                             |
| Contexto de Recorrências e Fechamento        | Contexto de Rateio                       | Evento               | `Fechamento.CicloEncerrado { groupId, periodo }`                      |
| Contexto de Recorrências e Fechamento        | Contexto de Liquidação Pix (Conciliação) | API síncrona         | `POST /cobrancas/gerar { groupId, periodo }`                          |
| Contexto de Recorrências e Fechamento        | Contexto de Notificações e Lembretes     | Evento               | `Fechamento.CobrancasGeradas { groupId, cobrancas[] }`                |
| Contexto de Notificações e Lembretes         | Usuário final                            | Push/Email/SMS       | “Você tem R$ X a pagar no grupo Y”                                    |
| Contexto de Autenticação e Autorização (IAM) | Todos os Contextos                       | API síncrona / JWT   | `GET /me/roles` ou claims JWT (`tenantId`, `scopes`)                  |
| Contexto de Planos e Entitlements            | Todos os Contextos                       | API síncrona         | `POST /check-feature { userId, feature }`                             |
| Contexto de IAM                              | Contexto de Planos e Entitlements        | Evento               | `IAM.AssinaturaAlterada { userId, plano }`                            |
| Contexto de Compliance e Auditoria           | Todos os Contextos                       | Evento               | `Audit.EventoRegistrado { actorId, ação, recurso, traceId }`          |
| Contexto de Observabilidade                  | Todos os Contextos                       | Telemetria           | `otel.trace`, `metric.rateio.latency`                                 |
| Contexto de Relatórios e Exportações         | Contexto de Armazenamento de Arquivos    | API síncrona         | `POST /arquivos { tipo: "relatorio", conteúdo }`                      |
| Contexto de Armazenamento de Arquivos        | Contexto de Relatórios e Exportações     | Evento               | `Arquivos.ArquivoDisponivel { fileId, url }`                          |
| Contexto de OCR e Leitura de Recibos         | Contexto de Grupos e Despesas            | Evento               | `OCR.ReciboInterpretado { expenseDraftId, itens[], total }`           |
| Contexto de Grupos e Despesas                | Contexto de OCR e Leitura de Recibos     | API síncrona         | `POST /ocr/extrair { fileId }`                                        |
| Contexto de Onboarding e Templates           | Contexto de Grupos e Despesas            | API síncrona         | `POST /grupos { templateId }`                                         |
| Contexto de Analytics de Produto             | Todos os Contextos                       | Evento               | `Analytics.EventoProduto { userId, evento, propriedades }`            |

---

## 6. Definição da Linguagem Ubíqua

Liste os termos principais da Linguagem Ubíqua do projeto. Explique brevemente cada termo.

| **Termo**          | **Descrição**                                                                                         |
| ------------------ | ----------------------------------------------------------------------------------------------------- |
| **Grupo**          | Conjunto de pessoas reunidas para dividir despesas em comum (ex.: viagem, moradia, evento).           |
| **Despesa**        | Valor registrado dentro de um grupo, representando um gasto a ser rateado entre os participantes.     |
| **Rateio**         | Regra de divisão de uma despesa entre os membros (igual, percentual, por item ou customizado).        |
| **Saldo**          | Posição individual de cada participante no grupo: quanto deve ou tem a receber.                       |
| **Liquidação Pix** | Ação de quitar uma despesa ou saldo via Pix, conciliando o pagamento com o sistema.                   |
| **Conciliação**    | Processo de verificar e marcar uma despesa ou saldo como pago (manual ou automática).                 |
| **Ledger**         | Registro histórico e auditável de todas as movimentações financeiras de um grupo.                     |
| **Recorrência**    | Despesas que se repetem periodicamente (ex.: aluguel, internet, contas fixas).                        |
| **Fechamento**     | Momento de consolidar as despesas de um período (mês, viagem) e gerar os saldos finais.               |
| **Lembrete**       | Notificação enviada (ex.: via WhatsApp) para lembrar os participantes de pagar ou confirmar quitação. |

---

## 7. Estratégia de Desenvolvimento

Para cada tipo de subdomínio, explique a abordagem para implementação:

- **Core Domain:** Desenvolver internamente com foco total.
- **Supporting Subdomain:** Desenvolver internamente ou parcialmente terceirizar.
- **Generic Subdomain:** Usar ferramentas ou serviços de mercado.

| **Subdomínio**             | **Estratégia**                                                   | **Ferramentas ou Serviços (se aplicável)**      |
| -------------------------- | ---------------------------------------------------------------- | ----------------------------------------------- |
| Gestão de Grupos           | Desenvolver internamente com foco total, pois é o núcleo.        |                                                 |
| Motor de Rateio            | Implementar internamente, garantindo regras flexíveis.           |                                                 |
| Liquidação via PIX First   | Construir internamente com integrações bancárias.                | SPI/Pix API do Banco Central, Open Banking APIs |
| Saldos e Liquidações       | Desenvolver internamente para consistência e controle.           | Confluente, AWS MSK                             |
| Notificações e Lembretes   | Interno ou terceirizado, conforme escala.                        | OneSignal, Twilio, SendGrid                     |
| Recorrências e Fechamento  | Desenvolver lógica própria de agendamento e ciclos.              |                                                 |
| Autenticação e Autorização | Usar soluções prontas de mercado para acelerar.                  | Clerk, Auth0, Keycloak, Cognito                 |
| Compliance e Auditoria     | Desenvolver integrações, aproveitando libs externas.             | Elastic (audit logs), Open Policy Agent (OPA)   |
| Relatórios e Exportações   | Interno, com apoio em libs de geração de relatórios/BI.          |                                                 |
| Observabilidade            | Usar stack consolidada de mercado.                               | Grafana, Prometheus, Datadog                    |
| Onboarding                 | Desenvolver internamente, apoiado em libs e frameworks de UX.    |                                                 |
| Plano de Assinatura        | Controle interno de planos, usando serviços de billing externos. | Stripe                                          |
| Armazenamento de Arquivos  | Usar serviços de nuvem consolidados.                             | AWS S3                                          |
| Analytics de Produto       | Híbrido: coleta interna e análise via SaaS.                      | PostHog                                         |
| OCR / Leitura de Recibos   | Usar soluções de mercado para reconhecimento.                    | AWS Textract, Tesseract OCR                     |

---

## 8. Diagrama Visual (Opcional, mas Recomendado)
<img width="3840" height="941" alt="Untitled diagram _ Mermaid Chart-2025-09-25-013158" src="https://github.com/user-attachments/assets/ddf33638-4005-4644-a879-011ed0315beb" />

