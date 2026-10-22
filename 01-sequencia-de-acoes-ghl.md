# Trend Science: Sequência de ações da integração com o GHL

Oct 20, 2026 · @Jhonathan

Instruções para quem vai executar a integração das duas landing pages da Trend Science com o GHL, na subconta **Trend Science** (`M2QUwbxxOXs69NSt476w`). Dez ações, na ordem em que uma depende da anterior. O detalhe de cada workflow, com gatilho, condições e ações, está no [Guia de execução](https://claude.ai/artifact/MLGHnHALCPa4dbu8sHhoFa); o mapeamento de campos, payload e regras de deduplicação, na [Especificação de integração](https://claude.ai/code/artifact/78cc6122-fedc-46c7-9e95-aea733b578c2).

## O que já está pronto na conta

- **16 campos personalizados de contato**, criados em 21/10: `contact.crm`, `contact.uf_do_crm`, `contact.especialidade`, `contact.origem_do_lead`, `contact.pgina_de_entrada` (sem acento), `contact.codigo_de_campanha`, `contact.utm_source`, `contact.utm_medium`, `contact.utm_campaign`, `contact.utm_content`, `contact.utm_term`, `contact.data_de_captura`, `contact.consentimento_de_contato`, `contact.data_do_consentimento`, `contact.melhor_horario`, `contact.pop_enviado_em`, `contact.versao_do_pop`.
- **9 tags**: `lp1-pop`, `lp2-oferta`, `origem-meta`, `origem-lista`, `origem-indicacao`, `origem-organico`, `pop-enviado`, `conversa-ativa`, `opt-out`.
- **Funis existentes que a integração usa**: P1 Lista Completa (`7snawjtgxFhWhRl52L5c`), P2 Captação POP Regulatório (`F310RLIIi4NGYMio9rCH`), P3 Inbound Captação POP Regulatório (`6AWiM9u2sNIQqS2KPc1i`), Pedidos D9Pro (`XKK9MK2z7mtRMmrvH3ci`).
- **Campos de oportunidade existentes** que recebem cópia dos dados do contato ao criar a oportunidade: `opportunity.registro_prescritor`, `opportunity.uf_prescritor`, `opportunity.nome_prescritor`.

## Regras de negócio que valem em tudo

- Não há validação de CRM. O CRM é coletado para identificação e para os campos do pedido; nenhum fluxo espera aprovação.
- A receita não é pedida antes do pagamento. Ela é coletada depois, na etapa Receita Pendente do funil Pedidos D9Pro, como já funciona.
- Telefone em E.164 (`+55` + DDD + número) é a chave de deduplicação. Upsert sempre; nunca criar contato novo para telefone existente. Campo vazio não sobrescreve valor preenchido.
- Contato com oportunidade aberta em qualquer funil não ganha oportunidade nova: a existente é movida.
- `source` só é gravado na primeira captura; capturas seguintes vão para Página de entrada e tags.

## Sequência de ações

### Funil e motivos de perda

1. **Criar o funil P4 - Inbound Oferta Direta** em Oportunidades, Funis, com oito etapas nesta ordem: Conversa iniciada; Pediu contato; Em qualificação; Em negociação; Pedido criado; Ganho; Perdido; Retorno futuro. Marcar "mostrar no funil" e "mostrar no gráfico". Anotar o ID do funil e das oito etapas e enviar a Jhonathan.
2. **Cadastrar os motivos de perda** em Configurações, Oportunidades: Sem interesse; Já tem fornecedor; Não trabalha com manipulados; Pediu para não receber contato; Preço; Prazo; Sem resposta após cadência; Não é médico.

### Workflows de entrada

3. **Workflow "LP1 POP · Entrada"**, gatilho Inbound Webhook. Upsert do contato por telefone com todos os campos do payload; tags `lp1-pop` e de origem; se houver oportunidade aberta, mover (P2 para Preencheu Form; P1 mantém e cria tarefa), senão criar em P3 Lead Gerado copiando CRM, UF e nome para os campos de prescritor; remover de workflows de cadência e aplicar `conversa-ativa`; enviar o PDF licenciado pelo WhatsApp em até 5 minutos, gravar POP enviado em e Versão do POP, tag `pop-enviado`, mover para Envio de Doc Licenciado; aguardar 2 dias úteis e criar tarefa de primeiro contato se não houver resposta; enviar evento Lead à Meta. Salvar e copiar a URL do webhook: ela vai no formulário da LP 1.
4. **Workflow "LP2 Oferta · Entrada formulário"**, duplicado do anterior: Inbound Webhook próprio; tag `lp2-oferta`; oportunidade em P4 Pediu contato; tarefa para o Representante Credenciado no horário escolhido (`contact.melhor_horario`); mensagem de confirmação; sem envio de PDF. Copiar a URL do webhook: ela vai no formulário de retorno da LP 2.
5. **Workflow "LP2 Oferta · Entrada WhatsApp"**, gatilho Customer Replied com filtro "mensagem contém `[LP2-`" (e um segundo gatilho com `[ADS-`), com condição inicial "tag `lp2-oferta` não existe" para não reprocessar. Gravar código de campanha (texto entre colchetes, via Custom Code ou um ramo por código), Página de entrada, Origem e consentimento se vazios; tags `lp2-oferta`, `origem-meta`, `conversa-ativa`; oportunidade em P4 Conversa iniciada ou mover a existente; remover de cadências; evento Contact à Meta. Sem tarefa e sem mensagem automática: a Luíza Sales responde.
6. **Em todos os três**, ao criar oportunidade, copiar `contact.crm`, `contact.uf_do_crm` e nome completo para `opportunity.registro_prescritor`, `opportunity.uf_prescritor` e `opportunity.nome_prescritor`. Qualquer falha de passo (telefone inválido, CRM fora do padrão) vai para um ramo que aplica a tag `revisar-entrada` (criar) e abre tarefa; nada é descartado.

### Agente de IA

7. **Ajustar a Luíza Sales** (Conversation AI): na primeira troca com contato novo, pedir nome completo, CRM e UF em uma pergunta e gravar nos campos; ignorar e não repetir o código entre colchetes da primeira mensagem; não pedir receita antes do pagamento e informar que ela é solicitada após a confirmação; encerrar com cordialidade quem não é médico, com motivo de perda Não é médico. Habilitar nas Ações a atualização de CRM, UF do CRM e Especialidade e a movimentação em P4. Incluir CRM e UF no Summary de handoff.

### Rastreamento e páginas

8. **Conectar a API de Conversões da Meta** com cinco eventos: Lead (captura por formulário), Contact (conversa iniciada ou respondida), Qualified (entrada em Em qualificação), Negotiation (entrada em Em negociação), Purchase (Ganho ou pedido pago no D9, com valor em BRL). Telefone e e-mail em SHA-256, `fbc` a partir do `fbclid`, `event_id` único por contato, evento e data.
9. **Configurar os formulários das páginas**: campos visíveis com `name` = `nome`, `crm`, `uf`, `whatsapp` (e `melhor_horario` na LP 2); hidden fields `pagina`, `source`, `codigo_campanha`, `utm_source`, `utm_medium`, `utm_campaign`, `utm_content`, `utm_term`, `fbclid`, `data_captura`, `consentimento`, preenchidos por script a partir da URL (com persistência em localStorage por 30 dias); `action` = URL do webhook do workflow correspondente, POST JSON. Nos botões de WhatsApp da LP 2, mensagem pré-preenchida terminando em `[LP2-OFERTA]` (com sufixo do criativo quando houver `utm_content`) e o número oficial conectado ao GHL.

### Teste de aceite

10. **Rodar os oito casos** do Guia de execução com número e CRM de teste: lead novo pela LP 1; duplicado; contato da lista; clique para WhatsApp com código; formulário de retorno; não médico; campo vazio no reenvio; pedido pago com origem. Nenhum anúncio é ligado antes de os oito passarem. Registrar a data de conclusão e devolver a Jhonathan: IDs do P4, URLs dos dois webhooks, prints dos oito testes.
