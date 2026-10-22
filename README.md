# Trend Science: integração das landing pages com o GHL

Documentação para implantar, na subconta Trend Science do GoHighLevel, a captação de leads das duas landing pages (POP regulatório e oferta direta), com atribuição por origem, deduplicação e rastreamento para a Meta.

## Leia nesta ordem

1. [Sequência de ações](docs/01-sequencia-de-acoes-ghl.md): o que já está pronto na conta, as regras de negócio e as dez ações restantes, em ordem de dependência. Comece por aqui.
2. [Guia de execução](docs/02-guia-de-execucao-ghl.md): cada etapa no formato do construtor do GHL, com gatilho, condições e ações de cada workflow, IDs dos funis e o teste de aceite.
3. [Especificação de integração](docs/03-especificacao-integracao-lps.md): mapeamento campo a campo, hidden fields, payload do webhook, tags e deduplicação.

## Regras que valem em tudo

- Não há validação de CRM. O CRM é coletado para identificação e para os campos do pedido.
- A receita não é pedida antes do pagamento; ela é coletada depois, no funil Pedidos D9Pro.
- Telefone em E.164 é a chave de deduplicação. Upsert sempre.
- Nenhum anúncio é ligado antes de os oito testes de aceite passarem.

## Estado em 22/10/2026

- 16 campos personalizados de contato criados.
- 9 tags criadas.
- Funil P4, workflows, ajuste da Luíza Sales, API de Conversões e testes: pendentes (ver a sequência de ações).

Nota: a especificação carrega uma atualização de 21/10 no topo, porque foi escrita antes da decisão de não validar CRM; onde houver conflito, vale o Guia de execução.
