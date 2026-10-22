# Trend Science: Guia de execução das próximas etapas no GHL

Oct 21, 2026 · @Jhonathan

## Situação

Na subconta Trend Science (`M2QUwbxxOXs69NSt476w`) já estão criados os 17 campos personalizados de contato e as 12 tags. Faltam cinco etapas, todas feitas na interface do GHL porque a API não cria funis nem workflows. Este guia descreve cada uma no formato do construtor, na ordem de dependência.

| Item | Status | Onde |
| --- | --- | --- |
| 17 campos personalizados de contato | Feito em 21/10 | Configurações, Campos personalizados (Status do CRM pode ser apagado) |
| 12 tags | Feito em 21/10 | Configurações, Tags (crm-pendente, crm-validado e crm-invalido podem ser apagadas) |
| Funil P4 Inbound Oferta Direta | A fazer | Oportunidades, Funis |
| Workflow Entrada LP 1 POP | A fazer | Automação, Workflows |
| Workflow Entrada LP 2 formulário | A fazer | Automação, Workflows |
| Workflow Entrada LP 2 WhatsApp | A fazer | Automação, Workflows |
| Workflow Validação do CRM (removido em 21/10: não há validação) | A fazer | Automação, Workflows |
| Ajuste da Luíza Sales | A fazer | Conversation AI |
| API de Conversões da Meta | A fazer | Integrações, Facebook e Instagram |
| Hidden fields nas páginas | A fazer | Sites, Formulários ou código da LP |
| Teste de aceite | A fazer | Tudo acima |

**Chaves dos campos criados**, para copiar nos workflows: `contact.crm`, `contact.uf_do_crm`, `contact.especialidade`, `contact.status_do_crm`, `contact.origem_do_lead`, `contact.pgina_de_entrada` (sem acento), `contact.codigo_de_campanha`, `contact.utm_source`, `contact.utm_medium`, `contact.utm_campaign`, `contact.utm_content`, `contact.utm_term`, `contact.data_de_captura`, `contact.consentimento_de_contato`, `contact.data_do_consentimento`, `contact.melhor_horario`, `contact.pop_enviado_em`, `contact.versao_do_pop`.

**IDs que os workflows usam.** Funil P2 Captação POP: `F310RLIIi4NGYMio9rCH`, etapa Preencheu Form `0e23b14b-4cf5-46bf-81bb-971ee2ee1dfb`, etapa Envio de Doc Licenciado `5e0f4524-4144-4035-b50c-eae7529dd694`. Funil P3 Inbound POP: `6AWiM9u2sNIQqS2KPc1i`, etapa Lead Gerado `bf743d20-81d3-4a2f-81fa-8334d58cc4e6`, etapa Envio de Doc Licenciado `eb49af24-3c43-436c-b9e0-e0c966f36567`. Funil P1 Lista Completa: `7snawjtgxFhWhRl52L5c`, etapa Interagiu `6920427a-5d14-44f2-bf2e-6441b853bf93`, Em negociação `f053fdec-2077-4654-8a85-df35a50a60b0`. Na interface o construtor mostra os nomes, não os IDs; eles servem para conferir e para o webhook.

## Etapa 1. Criar o funil P4 Inbound Oferta Direta

Em Oportunidades, Funis, Criar novo funil. Nome: **P4 - Inbound Oferta Direta**. Marcar "Mostrar no funil" e "Mostrar no gráfico". Criar as etapas nesta ordem:

| Posição | Etapa | Entra quando | Probabilidade sugerida |
| --- | --- | --- | --- |
| 1 | Conversa iniciada | Clique para WhatsApp com código `[LP2-` | 15% |
| 2 | Pediu contato | Formulário de retorno da LP 2 | 20% |
| 3 | Em qualificação | Luíza Sales em trilha de qualificação | 40% |
| 4 | Em negociação | Portfólio e condições apresentados | 65% |
| 5 | Pedido criado | Pedido aberto no D9, aguardando pagamento | 85% |
| 6 | Ganho | Pedido pago no D9 (a receita é coletada depois, no funil Pedidos D9Pro) | 100% |
| 7 | Perdido | Motivo padronizado | 0% |
| 8 | Retorno futuro | Médico pediu para voltar depois | 10% |

Depois de salvar, abrir cada etapa e anotar o ID (aparece na URL ao editar a etapa, ou pela API com `get-pipelines`). Enviar os oito IDs para atualizar a especificação. Os motivos de perda a cadastrar em Configurações, Oportunidades, Motivos de perda: Sem interesse; Já tem fornecedor; Não trabalha com manipulados; Pediu para não receber contato; Preço; Prazo; Sem resposta após cadência; Não é médico.

## Etapa 2. Workflow "Entrada LP 1 POP"

Em Automação, Workflows, Criar workflow, Começar do zero. Nome: **LP1 POP · Entrada**. Configurações do workflow: permitir reentrada (sim), fuso America/Sao\_Paulo.

**Gatilho:** Inbound Webhook. Salvar o workflow uma vez para o GHL gerar a URL do webhook; essa URL vai no formulário da LP 1. Enviar um payload de teste (o JSON da especificação) para o GHL mapear os campos.

**Ações, na ordem:**

1. **Criar ou atualizar contato** (ação "Create/Update Contact" do Inbound Webhook). Mapear: Nome = `{{inboundWebhookRequest.nome}}`; Telefone = `{{inboundWebhookRequest.whatsapp}}`; Estado = `{{inboundWebhookRequest.uf}}`; País = BR; Origem (source) = `lp1_pop`; CRM = `{{inboundWebhookRequest.crm}}`; UF do CRM = `{{inboundWebhookRequest.uf}}`; Página de entrada = LP1 POP; Código de campanha = `{{inboundWebhookRequest.codigo_campanha}}`; UTM source, medium, campaign, content, term = os cinco `{{inboundWebhookRequest.utm_*}}`; Data de captura = `{{inboundWebhookRequest.data_captura}}`; Consentimento de contato = Sim; Data do consentimento = `{{inboundWebhookRequest.data_captura}}`. Opção "não sobrescrever campos preenchidos" ligada para Origem e Origem do lead.
2. **Se/Senão: Origem do lead vazia.** Ramo Sim: Atualizar campo Origem do lead = Meta Ads se `utm_source` contiver `meta`; senão Orgânico. Ramo Não: seguir.
3. **Adicionar tags:** `lp1-pop` e `origem-meta` ou `origem-organico` conforme o passo 2.
4. **Se/Senão: o contato tem oportunidade aberta em algum funil.** Condição: "Oportunidade, status = aberta" em P1, P2 ou P3.
   - Ramo Sim, oportunidade em P2: ação "Atualizar oportunidade", funil P2, etapa Preencheu Form.
   - Ramo Sim, oportunidade em P1: não mover; criar tarefa "Lead da lista preencheu o POP" para o responsável da oportunidade.
   - Ramo Não: ação "Criar oportunidade", funil P3 Inbound Captação POP Regulatório, etapa Lead Gerado, nome `{{contact.name}} · POP`, origem `lp1_pop`. Nela, preencher os campos da oportunidade: registro prescritor = `{{contact.crm}}`, uf prescritor = `{{contact.uf_do_crm}}`, nome prescritor = `{{contact.name}}`.
5. **Remover de todos os workflows de cadência** (ação "Remove from workflow", selecionar os workflows de disparo e follow-up da lista) e **adicionar tag** `conversa-ativa`.
6. **Enviar o PDF licenciado pelo WhatsApp** (número oficial), em até 5 minutos: "Cadastro recebido. Segue o manual POP regulatório licenciado em seu nome. Um Representante Credenciado fica disponível por aqui para dúvidas. Trend Science." Se o PDF for gerado por médico (marca d'água), a ação chama o gerador do Luiz Otavio por webhook e envia o link retornado. Atualizar POP enviado em = hoje, Versão do POP = `2026.1`, tag `pop-enviado`; mover a oportunidade para Envio de Doc Licenciado (P2 ou P3).
7. **Aguardar 2 dias úteis** e criar tarefa "Primeiro contato pós-POP" para o Representante Credenciado, salvo se o médico já tiver respondido (condição: última mensagem recebida depois do envio).
8. **Webhook de saída** para a API de Conversões (ou ação nativa "Facebook Conversion API" se disponível): evento `Lead`, `event_time` agora, `phone` e `email` em hash, `fbc` a partir de `{{inboundWebhookRequest.fbclid}}`, `event_source_url` = `{{inboundWebhookRequest.pagina_url}}`.

**Fila de exceções.** Qualquer passo que falhe (telefone inválido, CRM fora do padrão) vai para um ramo que adiciona a tag `revisar-entrada` (criar na hora) e abre tarefa para o Inside Sales. Nada é descartado.

## Etapa 3. Workflows da LP 2

### "LP2 Oferta · Entrada formulário"

Duplicar o workflow da LP 1 e alterar:

- Gatilho: Inbound Webhook próprio (nova URL, vai no formulário de retorno da LP 2).
- Mapeamento: Origem = `lp2_oferta`; Página de entrada = LP2 Oferta direta; Código de campanha = `LP2-OFERTA`; Melhor horário = `{{inboundWebhookRequest.melhor_horario}}`.
- Tag `lp2-oferta` no lugar de `lp1-pop`.
- Passo 4, ramo Não: criar oportunidade no funil **P4**, etapa **Pediu contato**. Ramo Sim com oportunidade em P1: mover para Interagiu, ou manter se já estiver em Em negociação ou além.
- Passos 6 e 7 substituídos por: tarefa "Chamar no horário: {{contact.melhor\_horario}}" para o Representante Credenciado, com vencimento no início da faixa escolhida (usar "Aguardar até" com condição de horário antes da tarefa, se quiser que ela apareça só na hora), e mensagem: "Recebemos o pedido de contato. Um Representante Credenciado da Trend Science chama no horário indicado."

### "LP2 Oferta · Entrada WhatsApp"

Workflow novo. Nome: **LP2 Oferta · Entrada WhatsApp**.

**Gatilho:** Customer Replied (mensagem recebida), canal WhatsApp, filtro "mensagem contém" `[LP2-`. Adicionar um segundo gatilho igual com `[ADS-` para os anúncios de clique direto.

**Ações:**

1. **Atualizar campos do contato** (só se vazios): Página de entrada = LP2 Oferta direta; Origem (source) = `lp2_oferta`; Origem do lead = Meta Ads; Data de captura = hoje; Consentimento de contato = Sim; Data do consentimento = hoje.
2. **Atualizar campo** Código de campanha com o texto entre colchetes. O construtor não extrai substring; usar uma ação "Custom Code" (JavaScript) que recebe `{{message.body}}` e devolve o conteúdo entre `[` e `]`, ou, se preferir sem código, um Se/Senão com um ramo por código conhecido (`[LP2-OFERTA]`, `[LP2-OFERTA-V2]`, `[ADS-OFERTA-V1]`), cada ramo gravando o valor fixo. Gravar também o sufixo em UTM content.
3. **Adicionar tags:** `lp2-oferta`, `origem-meta`, `conversa-ativa`.
4. **Se/Senão: oportunidade aberta.** Sim em P1: mover para Interagiu (ou manter se adiante). Sim em P2 ou P3: manter, adicionar nota "Chegou também pela oferta direta". Não: criar oportunidade em **P4**, etapa **Conversa iniciada**, com os campos de prescritor copiados quando a Luíza registrar CRM e UF.
5. **Remover dos workflows de cadência.**
6. **Webhook de saída** para a API de Conversões: evento `Contact`.
7. Nenhuma tarefa e nenhuma mensagem automática: a Luíza Sales responde dentro da janela de 72 horas.

**Cuidado com o gatilho.** O filtro "contém `[LP2-`" precisa valer só para a primeira mensagem da conversa; adicionar a condição "tag `lp2-oferta` não existe no contato" no início do workflow, para não reprocessar se o médico reenviar o texto.

## Etapa 4. Ajuste da Luíza Sales (Conversation AI)

Não há validação de CRM nem conferência de receita antes do pagamento. O CRM é coletado para identificação e para os campos do pedido; a receita é coletada depois do pagamento, na etapa Receita Pendente do funil Pedidos D9Pro, como já funciona hoje.

No prompt central, dentro do limite de 2.000 palavras, adicionar duas regras:

- **Identificação.** "Na primeira troca com um contato novo, antes de apresentar portfólio ou condições, peça nome completo, CRM e UF em uma única pergunta, com naturalidade, e registre nos campos CRM e UF do CRM. Se o contato não for médico ou não informar CRM, explique que o atendimento é exclusivo a prescritores e encerre com cordialidade; registre o motivo de perda Não é médico."
- **Código de campanha.** "Se a primeira mensagem do contato contiver um código entre colchetes, como \[LP2-OFERTA\], ignore o código na resposta, não o repita e não pergunte sobre ele. Trate a mensagem como interesse no plano de fornecimento e abra a qualificação."
- **Receita.** "Não peça receita antes do pagamento. Ao fechar o pedido, informe que as receitas serão solicitadas após a confirmação do pagamento, pelo mesmo canal, e que precisam corresponder ao total de ampolas do pedido."

Nas Ações da Luíza, habilitar a atualização dos campos CRM, UF do CRM e Especialidade, e a movimentação da oportunidade em P4 para Em qualificação e Em negociação. No Summary de handoff, incluir CRM e UF. Testar com três conversas: uma com código e contato novo, uma de contato da lista sem código, uma de não médico.

## Etapa 5. API de Conversões da Meta e hidden fields

### API de Conversões

Em Configurações, Integrações, conectar a conta do Facebook e selecionar o pixel da Trend Science. Se a conta tiver a ação nativa "Facebook Conversion API" nos workflows, usar essa; senão, ação Webhook para `https://graph.facebook.com/v21.0/<PIXEL_ID>/events?access_token=<TOKEN>` com o corpo em JSON.

| Evento | Disparado por | Dados |
| --- | --- | --- |
| Lead | Workflows de entrada das LPs (formulário) | telefone e e-mail em SHA-256, `fbc` do `fbclid`, URL da página |
| Contact | Workflow Entrada WhatsApp e primeira resposta do médico | telefone em hash |
| Qualified (evento personalizado) | Oportunidade entra em Em qualificação | telefone em hash |
| Negotiation (evento personalizado) | Oportunidade entra em Em negociação | telefone em hash, valor estimado |
| Purchase | Oportunidade entra em Ganho ou pedido pago no D9 | telefone em hash, `value` = valor do pedido, `currency` = BRL |

Todo evento leva `event_id` único (ID do contato mais nome do evento mais data) para desduplicar com o pixel. `action_source` = `website` para Lead e `chat` para os demais.

### Hidden fields nas páginas

Se as páginas forem construídas no construtor do GHL, o formulário nativo já captura UTMs automaticamente; basta adicionar campos ocultos `pagina`, `source` e `codigo_campanha` com valor fixo e mapear para os campos personalizados. Se forem páginas externas, incluir no `<head>` um script que: lê a query string; grava `utm_source`, `utm_medium`, `utm_campaign`, `utm_content`, `utm_term` e `fbclid` em campos `input type="hidden"` com esses mesmos `name`; preenche `data_captura` com a data local no formato AAAA-MM-DD; guarda os UTMs em `localStorage` por 30 dias para que o médico que volta sem parâmetros mantenha a atribuição; e, nos botões de WhatsApp da LP 2, acrescenta o sufixo do criativo ao código (`[LP2-OFERTA-<utm_content>]`) quando houver `utm_content`.

O `name` de cada campo visível segue a especificação: `nome`, `crm`, `uf`, `whatsapp`, `melhor_horario`. O `action` do formulário é a URL do Inbound Webhook do workflow correspondente, método POST, JSON.

## Etapa 6. Teste de aceite

Usar um número de WhatsApp de teste e um CRM fictício marcado como teste. Rodar cada caso, conferir o resultado e só então apagar o contato de teste. Nenhum real em mídia antes de os oito passarem.

| Caso | O que enviar | O que conferir |
| --- | --- | --- |
| 1. Lead novo pela LP 1 | Formulário com UTMs `meta/paid/isca_pop/teste_v1/lal1` | Contato criado com telefone em E.164, CRM, UF, Página, Código, cinco UTMs, Data de captura, Consentimento; tags `lp1-pop`, `origem-meta`, `conversa-ativa`, `pop-enviado`; oportunidade em P3 Envio de Doc Licenciado com campos de prescritor; PDF recebido no WhatsApp em até 5 minutos; evento Lead no Gerenciador de Eventos da Meta |
| 2. Duplicado | Mesmo telefone, novo envio pela LP 1 | Nenhum contato novo; nenhuma oportunidade nova; campos preenchidos mantidos; PDF não reenviado se `pop-enviado` já existe |
| 3. Contato da lista | Contato existente em P2 Disparo preenche a LP 1 | Oportunidade movida para Preencheu Form e depois Envio de Doc Licenciado; `source` original preservado; tag `lp1-pop` adicionada; removido dos workflows de cadência |
| 4. Clique para WhatsApp | Mensagem "Olá. Vi a página... \[LP2-OFERTA\]" de número novo | Código, Página e Origem gravados; oportunidade em P4 Conversa iniciada; Luíza pede nome, CRM e UF na primeira troca e não repete o código; evento Contact na Meta |
| 5. Formulário de retorno | Formulário da LP 2 com Melhor horário = Tarde | Oportunidade em P4 Pediu contato; tarefa para o Representante com o horário; WhatsApp de confirmação |
| 6. Não médico | Conversa pelo WhatsApp sem CRM, dizendo ser paciente | Luíza encerra com cordialidade, sem portfólio; oportunidade Perdida com motivo Não é médico |
| 7. Campo vazio | Reenviar o caso 1 com `utm_content` vazio | UTM content anterior mantido |
| 8. Pedido pago com origem | Marcar a oportunidade do caso 1 como Ganho com valor | Evento Purchase com valor na Meta; relatório por Origem do lead mostra o pedido em Meta Ads; pedido segue para Receita Pendente no funil Pedidos D9Pro |

Quando os oito passarem, registrar a data no documento de integração e liberar o primeiro real em mídia.
