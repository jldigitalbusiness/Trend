# Trend Science: Integração das Landing Pages com o GHL

Oct 20, 2026 · @Jhonathan

## Resumo

**Atualização de 21/10/2026.** Não há validação de CRM nem conferência de receita antes do pagamento; a receita é coletada após o pagamento, na etapa Receita Pendente do funil Pedidos D9Pro. Onde este documento cita "validação do CRM", tarefa "Validar CRM", tags `crm-pendente`, `crm-validado` e `crm-invalido`, campo Status do CRM, etapa "CRM validado" ou "Receita em conferência", vale o que está no [Guia de execução](https://claude.ai/artifact/MLGHnHALCPa4dbu8sHhoFa): o POP é enviado em até 5 minutos após o cadastro, o P4 tem oito etapas (Conversa iniciada, Pediu contato, Em qualificação, Em negociação, Pedido criado, Ganho, Perdido, Retorno futuro), e o campo Status do CRM e as três tags podem ser apagados.

Todo lead das duas landing pages entra na subconta Trend Science do GHL (location `M2QUwbxxOXs69NSt476w`) como contato, com origem, campanha e criativo gravados, deduplicado por telefone, e cria uma oportunidade na etapa certa do funil. A LP 1 (POP) envia por formulário; a LP 2 (oferta direta) envia por clique para WhatsApp, com o formulário de retorno como alternativa.

**O que a conta já tem e é reaproveitado.** Dez campos personalizados de contato (entre eles `fornecedor_atual`, `prazo_de_entrega_do_concorrente`, `perfil_de_compra`, `resumo_da_conversa_ia`, `d9_order_id`, `clientcpf`) e quatro funis: P1 Lista Completa, P2 Captação POP Regulatório (disparo para a lista), P3 Inbound Captação POP Regulatório (lead que chega pelo formulário) e Pedidos D9Pro.

**O que falta e este documento especifica.** Campos de identificação profissional (CRM, UF do CRM, especialidade), campos de atribuição (origem, página, campanha, criativo, UTMs), campos operacionais da captação (status da validação do CRM, data de envio do POP, melhor horário, consentimento), um funil de entrada para a oferta direta inbound, e as regras de tag, deduplicação e automação.

A copy e os campos visíveis estão em [Copy final das Landing Pages](https://claude.ai/code/artifact/40cbf6a8-7e14-462d-bf98-043714ec8273); as campanhas e UTMs seguem o [Plano de Mídia](https://claude.ai/code/artifact/21dcd637-5f2b-42b9-8829-3c1a593ebbb0).

## Campos do contato

**Campos padrão do GHL usados.** Não criar campo personalizado para o que já é padrão: o relatório, a deduplicação e o WhatsApp dependem deles.

| Campo padrão | Chave na API | Recebe |
| --- | --- | --- |
| Nome | `firstName` + `lastName` | Nome completo do formulário, dividido no primeiro espaço |
| Telefone | `phone` | WhatsApp em E.164 (`+5511900000000`) |
| E-mail | `email` | Só quando coletado (não está nos formulários v2) |
| Estado | `state` | UF informada |
| País | `country` | `BR` fixo |
| Origem | `source` | `lp1_pop` ou `lp2_oferta` |
| Tags | `tags` | Lista da seção Funil e tags |
| Tipo | `type` | `lead` na entrada; `customer` após o primeiro pedido pago |

**Campos que já existem e são reaproveitados.** A conta tem CRM, UF e nome do prescritor no nível da oportunidade, criados pela integração com o D9: `opportunity.registro_prescritor`, `opportunity.uf_prescritor` e `opportunity.nome_prescritor`. Eles continuam sendo a referência do pedido. O problema é que o lead chega antes de existir oportunidade, e a deduplicação e a validação do CRM acontecem no contato. Por isso o CRM e a UF entram também no contato, e o workflow copia os valores para os campos da oportunidade quando ela é criada, para que o D9 e os relatórios de pedido não mudem.

| Já existe | Modelo | Uso nesta integração |
| --- | --- | --- |
| `opportunity.registro_prescritor` | Oportunidade | Recebe cópia de `contact.crm` ao criar a oportunidade |
| `opportunity.uf_prescritor` | Oportunidade | Recebe cópia de `contact.uf_do_crm` |
| `opportunity.nome_prescritor` | Oportunidade | Recebe nome completo do contato |
| `contact.fornecedor_atual`, `contact.prazo_de_entrega_do_concorrente`, `contact.preo_praticado_pelo_concorrente`, `contact.perfil_de_compra` | Contato | Preenchidos pela Luíza Sales na qualificação, nunca pela landing page |
| `contact.resumo_da_conversa_ia` | Contato | Handoff |
| `contact.clientcpf`, `contact.client_email`, `contact.cliente_id`, `contact.d9_order_id` | Contato | Integração D9; não tocar |

Especialidade não foi encontrada pela API em nenhum modelo. Se existir em outro lugar da conta (campo padrão renomeado ou pasta não exposta), reaproveitar; senão, criar conforme a tabela abaixo. Confirmar antes de criar.

**Campos personalizados a criar no contato.** Grupo "Captação e atribuição". A chave é a que o GHL gera a partir do nome; conferir após criar.

| Nome no GHL | Chave esperada | Tipo | Opções ou formato | Para quê |
| --- | --- | --- | --- | --- |
| CRM | `contact.crm` | TEXT | Somente dígitos | Identificação e validação antes de existir oportunidade; copiado para `opportunity.registro_prescritor` |
| UF do CRM | `contact.uf_do_crm` | SINGLE\_OPTIONS | 27 UFs | Validação; copiado para `opportunity.uf_prescritor` |
| Especialidade | `contact.especialidade` | TEXT | Livre | Só se não existir; coletada na conversa |
| Status do CRM | `contact.status_do_crm` | SINGLE\_OPTIONS | Pendente, Validado, Inválido | Gate do envio do POP e do contato comercial |
| Origem do lead | `contact.origem_do_lead` | SINGLE\_OPTIONS | Lista importada, Meta Ads, Indicação, Orgânico, Evento, Outro | Leitura por origem no modelo de meta |
| Página de entrada | `contact.pgina_de_entrada` | SINGLE\_OPTIONS | LP1 POP, LP2 Oferta direta, Nenhuma | Atribuição por LP |
| Código de campanha | `contact.codigo_de_campanha` | TEXT | Ex.: `LP2-OFERTA` | Lido da mensagem do WhatsApp ou do formulário |
| UTM source | `contact.utm_source` | TEXT | `meta`, `google`, `direct` | Atribuição |
| UTM medium | `contact.utm_medium` | TEXT | `paid`, `organic`, `referral` | Atribuição |
| UTM campaign | `contact.utm_campaign` | TEXT | `aquecimento`, `isca_pop`, `oferta_direta`, `avanco` | Atribuição |
| UTM content | `contact.utm_content` | TEXT | Identificador do criativo | Leitura por criativo |
| UTM term | `contact.utm_term` | TEXT | Público (`lal1`, `lista_sem_resposta`) | Leitura por público |
| Data de captura | `contact.data_de_captura` | DATE | AAAA-MM-DD | Aging e janelas |
| Consentimento de contato | `contact.consentimento_de_contato` | SINGLE\_OPTIONS | Sim, Não | Registro do aceite |
| Data do consentimento | `contact.data_do_consentimento` | DATE | AAAA-MM-DD | LGPD |
| Melhor horário | `contact.melhor_horario` | SINGLE\_OPTIONS | Manhã (8h às 12h), Tarde (12h às 18h), Fim do dia (18h às 20h) | Formulário alternativo da LP 2 |
| POP enviado em | `contact.pop_enviado_em` | DATE | AAAA-MM-DD | Marco do funil da isca |
| Versão do POP | `contact.versao_do_pop` | TEXT | Ex.: `2026.1` | Reenvio em nova edição |

**Criados em 21/10/2026 via API, 17 campos.** Chaves reais geradas pelo GHL (usar estas, não as esperadas): `contact.crm`, `contact.uf_do_crm`, `contact.especialidade`, `contact.status_do_crm`, `contact.origem_do_lead`, `contact.pgina_de_entrada` (sem o acento; diferente do previsto), `contact.codigo_de_campanha`, `contact.utm_source`, `contact.utm_medium`, `contact.utm_campaign`, `contact.utm_content`, `contact.utm_term`, `contact.data_de_captura`, `contact.consentimento_de_contato`, `contact.data_do_consentimento`, `contact.melhor_horario`, `contact.pop_enviado_em`, `contact.versao_do_pop`. A API não cria pasta para o objeto Contato: os campos ficaram na pasta padrão e podem ser agrupados em "Captação e atribuição" pela interface.

## Mapeamento campo a campo

O atributo `name` de cada input é o que o webhook envia; a coluna "Campo no GHL" é onde o valor é gravado. Validação acontece no navegador e de novo no workflow de entrada; o que falhar no workflow vai para a fila de exceções, nunca é descartado.

### LP 1 (POP): formulário do hero

| Campo visível | `name` | Obrigatório | Validação no navegador | Normalização antes de gravar | Campo no GHL |
| --- | --- | --- | --- | --- | --- |
| Nome completo | `nome` | Sim | Mínimo 2 palavras, 5 caracteres | Capitalizar; primeira palavra em `firstName`, restante em `lastName` | `firstName`, `lastName` |
| CRM | `crm` | Sim | 4 a 7 dígitos | Remover tudo que não for dígito | `contact.crm` |
| UF | `uf` | Sim | Uma das 27 UFs | Maiúsculas | `contact.uf_do_crm` e `state` |
| WhatsApp | `whatsapp` | Sim | 10 ou 11 dígitos com DDD | E.164: `+55` + DDD + número | `phone` |

### LP 2 (oferta direta): formulário alternativo

| Campo visível | `name` | Obrigatório | Validação no navegador | Normalização antes de gravar | Campo no GHL |
| --- | --- | --- | --- | --- | --- |
| Nome completo | `nome` | Sim | Mínimo 2 palavras | Como na LP 1 | `firstName`, `lastName` |
| CRM | `crm` | Sim | 4 a 7 dígitos | Só dígitos | `contact.crm` |
| UF | `uf` | Sim | 27 UFs | Maiúsculas | `contact.uf_do_crm` e `state` |
| WhatsApp | `whatsapp` | Sim | 10 ou 11 dígitos | E.164 | `phone` |
| Melhor horário | `melhor_horario` | Não | Uma das 3 opções | Texto exato da opção | `contact.melhor_horario` |

### LP 2: clique para WhatsApp (sem formulário)

Não há campos digitados. O contato é criado pelo próprio GHL quando a mensagem chega ao número conectado; `phone` e o nome do perfil vêm do WhatsApp. O restante é extraído pelo workflow a partir da mensagem pré-preenchida (seção seguinte). CRM e UF são coletados pela Luíza Sales na primeira troca, antes de qualquer informação de portfólio.

**Regras comuns.** O `phone` é a chave de deduplicação: o webhook usa a opção de upsert por telefone, nunca cria contato novo quando o número já existe. Campo vazio não sobrescreve valor existente. `source` só é gravado na primeira captura; capturas seguintes vão para `contact.pgina_de_entrada` e para as tags, para preservar a origem original.

## Parâmetros ocultos

Todo formulário carrega campos `hidden` preenchidos por script na abertura da página, a partir da URL e de valores fixos da página. Sem eles o lead chega sem atribuição e o modelo de meta não fecha por origem.

| `name` do hidden | Valor | De onde vem | Campo no GHL |
| --- | --- | --- | --- |
| `pagina` | `LP1 POP` ou `LP2 Oferta direta` | Fixo na página | `contact.pgina_de_entrada` |
| `source` | `lp1_pop` ou `lp2_oferta` | Fixo na página | `source` (só na primeira captura) |
| `codigo_campanha` | `LP1-POP` ou `LP2-OFERTA` | Fixo na página | `contact.codigo_de_campanha` |
| `utm_source` | ex.: `meta` | Query string; vazio vira `direct` | `contact.utm_source` |
| `utm_medium` | ex.: `paid` | Query string | `contact.utm_medium` |
| `utm_campaign` | ex.: `isca_pop` | Query string | `contact.utm_campaign` |
| `utm_content` | ex.: `pop_carrossel_v2` | Query string | `contact.utm_content` |
| `utm_term` | ex.: `lal1` | Query string | `contact.utm_term` |
| `fbclid` | id do clique | Query string | Guardar para a API de Conversões; não precisa de campo visível |
| `data_captura` | AAAA-MM-DD | Relógio do navegador, fuso de São Paulo | `contact.data_de_captura` |
| `consentimento` | `Sim` | Marcado ao enviar (o texto miúdo é o aceite) | `contact.consentimento_de_contato` e `contact.data_do_consentimento` |
| `origem_lead` | `Meta Ads` se houver `utm_source=meta`; senão `Orgânico` | Derivado | `contact.origem_do_lead` (só se vazio) |

**Padrão de UTM nos anúncios.** `utm_source=meta&utm_medium=paid&utm_campaign=<aquecimento|isca_pop|oferta_direta|avanco>&utm_content=<criativo>&utm_term=<publico>`. Minúsculas, sem espaço, sem acento. O Meta substitui `{{ad.name}}` e `{{adset.name}}` automaticamente se os nomes seguirem o mesmo padrão.

**Clique para WhatsApp (LP 2).** A mensagem pré-preenchida termina com `[LP2-OFERTA]`. O workflow de entrada de conversa lê a primeira mensagem recebida com uma condição "contém `[LP2-`" e grava: `contact.codigo_de_campanha` = texto entre colchetes; `contact.pagina_de_entrada` = LP2 Oferta direta; `source` = `lp2_oferta` se vazio; `contact.origem_do_lead` = Meta Ads se vazio; tag `lp2-oferta`. Variações por criativo usam sufixo (`[LP2-OFERTA-V2]`), e o workflow grava o sufixo em `contact.utm_content`. Os links de WhatsApp dos anúncios de clique direto (sem passar pela LP) usam o mesmo padrão com `[ADS-OFERTA-<criativo>]`.

## Funil, etapa de entrada, tags e deduplicação

A entrada depende de o contato já existir ou não. Contato novo cria oportunidade no funil inbound; contato da lista muda de etapa no funil onde já está e não ganha oportunidade duplicada.

| Entrada | Contato novo | Contato já na lista |
| --- | --- | --- |
| LP 1, formulário | Oportunidade em **P3 Inbound Captação POP Regulatório**, etapa **Lead Gerado** (`bf743d20...`) | Se está em P2: mover para **Preencheu Form** (`0e23b14b...`). Se está em P1: manter, adicionar tag `lp1-pop` e criar tarefa para o Representante. Cadência de disparo pausada. |
| LP 2, clique para WhatsApp | Oportunidade em **P4 Inbound Oferta Direta** (funil a criar), etapa **Conversa iniciada** | Mover a oportunidade existente para **Interagiu** (P1) ou **Em negociação** se já estava qualificado; tag `lp2-oferta`; cadência pausada |
| LP 2, formulário de retorno | Oportunidade em **P4**, etapa **Pediu contato**, com tarefa para o Representante no horário escolhido | Igual à linha acima, com a tarefa no horário escolhido |

**Funil P4 Inbound Oferta Direta (a criar).** Etapas: Conversa iniciada; Pediu contato; CRM validado; Em qualificação; Em negociação; Receita em conferência; Ganho; Perdido; Retorno futuro. O P1 continua sendo o funil da lista; o P4 isola o que o Meta Ads trouxe, para a leitura por origem.

**Tags.** Uma tag por dimensão, minúsculas, com hífen:

| Tag | Quando |
| --- | --- |
| `lp1-pop` ou `lp2-oferta` | Na captura, pela página |
| `origem-meta`, `origem-lista`, `origem-indicacao`, `origem-organico` | Na captura, derivada da UTM ou da import |
| `crm-pendente`, depois `crm-validado` ou `crm-invalido` | Workflow de validação |
| `pop-enviado` | Quando o PDF licenciado sai |
| `conversa-ativa` | Enquanto houver conversa aberta ou negociação; suspende disparos e campanhas de recompra |
| `opt-out` | Pedido explícito; bloqueia toda cadência |

**Deduplicação.** Chave primária `phone` em E.164; chave secundária `contact.crm` + `contact.uf_do_crm` (mesmo CRM e UF com telefone diferente é o mesmo médico com número novo: mesclar, manter o telefone mais recente como principal). Nunca criar oportunidade nova em P3 ou P4 para contato que já tem oportunidade aberta em qualquer funil; mover a existente.

## Payload do webhook e automações

As páginas enviam um POST JSON para o webhook de entrada do GHL (Inbound Webhook de um workflow) ou, se forem construídas no próprio GHL, usam o formulário nativo com os mesmos nomes de campo. Exemplo da LP 1:

```json
{
  "pagina": "LP1 POP",
  "source": "lp1_pop",
  "codigo_campanha": "LP1-POP",
  "nome": "Ana Paula Ribeiro",
  "crm": "123456",
  "uf": "SP",
  "whatsapp": "+5511987654321",
  "utm_source": "meta",
  "utm_medium": "paid",
  "utm_campaign": "isca_pop",
  "utm_content": "pop_carrossel_v2",
  "utm_term": "lal1",
  "fbclid": "IwAR...",
  "data_captura": "2026-11-03",
  "consentimento": "Sim",
  "origem_lead": "Meta Ads",
  "pagina_url": "https://.../pop?utm_source=meta..."
}
```

A LP 2 envia o mesmo formato com `pagina`, `source` e `codigo_campanha` da oferta direta e o campo `melhor_horario` a mais.

**Workflow de entrada (um por LP), na ordem:**

1. Ao criar a oportunidade (passo 3), copiar `contact.crm`, `contact.uf_do_crm` e o nome completo para `opportunity.registro_prescritor`, `opportunity.uf_prescritor` e `opportunity.nome_prescritor`, para manter a compatibilidade com o D9.
2. Upsert do contato por `phone`; gravar campos padrão e personalizados conforme o mapeamento; não sobrescrever campo preenchido com valor vazio.
3. Aplicar tags de página e origem; `crm-pendente`.
4. Verificar oportunidade aberta em qualquer funil. Existe: mover de etapa conforme a tabela. Não existe: criar em P3 (LP 1) ou P4 (LP 2).
5. Pausar qualquer cadência de disparo ativa para o contato; adicionar `conversa-ativa`.
6. LP 1: criar tarefa "Validar CRM" para o responsável, prazo 4 horas úteis. LP 2 formulário: criar tarefa "Ligar/chamar no horário" com o `melhor_horario`. LP 2 clique: nenhuma tarefa, a Luíza Sales assume na hora.
7. Enviar o evento `Lead` à API de Conversões da Meta com telefone (hash), `fbclid` e `utm_content`.
8. Mensagem de confirmação ao médico, pelo WhatsApp, dentro de 1 minuto (LP 1: "cadastro recebido, CRM em validação"; LP 2 formulário: "recebemos, o Representante chama no horário indicado").

**Workflow de validação do CRM.** Quando `contact.status_do_crm` muda para Validado: tag `crm-validado`; LP 1 envia o PDF licenciado, grava `pop_enviado_em` e `versao_do_pop`, move para **Envio de Doc Licenciado**; LP 2 libera a apresentação de portfólio na conversa e move para **Em qualificação**. Quando muda para Inválido: tag `crm-invalido`, oportunidade para Perdido com motivo "CRM não validado", nenhuma informação de produto enviada.

**Eventos de volta para a Meta (API de Conversões).** `Lead` na captura; `Contact` quando a conversa é respondida; `Qualified` ao entrar em qualificação; `Negotiation` ao entrar em negociação; `Purchase` com valor quando o pedido no D9 é pago. Todos com telefone e e-mail em hash e o `fbclid` guardado.

## Checklist de implantação e teste

- [ ] Confirmar onde está o campo Especialidade; criar os 16 ou 17 campos do grupo "Captação e atribuição" e conferir as chaves geradas
- [ ] Criar o funil P4 Inbound Oferta Direta com as nove etapas
- [ ] Criar as tags da tabela
- [ ] Construir o workflow de entrada da LP 1 e o da LP 2 com os sete passos
- [ ] Construir o workflow de leitura da mensagem `[LP2-` no número conectado
- [ ] Construir o workflow de validação do CRM
- [ ] Ajustar a Luíza Sales para coletar CRM e UF antes de qualquer informação de portfólio quando `status_do_crm` for Pendente
- [ ] Conectar a API de Conversões com os cinco eventos
- [ ] Substituir o número placeholder dos botões da LP 2 pelo número conectado ao GHL

**Teste de aceite, antes do primeiro real em mídia:**

- [ ] Lead novo pela LP 1 com UTMs: contato criado com todos os campos, oportunidade em P3 Lead Gerado, tags corretas, tarefa de validação criada, confirmação no WhatsApp em até 1 minuto, evento Lead na Meta
- [ ] Mesmo telefone enviado de novo pela LP 1: nenhum contato duplicado, nenhuma oportunidade nova
- [ ] Contato da lista (em P2 Disparo) preenche a LP 1: move para Preencheu Form, cadência pausada, `source` original preservado
- [ ] Clique para WhatsApp da LP 2 com `[LP2-OFERTA]`: código, página e origem gravados, oportunidade em P4 Conversa iniciada, Luíza pede CRM antes de falar de portfólio
- [ ] Formulário de retorno da LP 2: tarefa no horário escolhido, oportunidade em P4 Pediu contato
- [ ] CRM marcado como Inválido: nada de produto enviado, oportunidade em Perdido com motivo
- [ ] Campo vazio no reenvio não apaga valor existente
- [ ] Pedido pago no D9 de um lead de origem Meta aparece no relatório por origem e dispara `Purchase`
