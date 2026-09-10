---
name: frame-conexa-templates
description: Crie, revise, edite e gerencie templates de WhatsApp do Frame Conexa via MCP, incluindo submissão e acompanhamento da análise da Meta. Use para templates da conta; não para disparos ou campanhas.
---

# Templates do Frame Conexa

Use as ferramentas do MCP `https://frameconexa.com/mcp`, autenticado com a chave `fc_` da conta em `Authorization: Bearer`. A chave vem do modal **Templates → Criar com IA** ou de **Configurações → API**. Não use tokens de sessão do painel ou credenciais da Meta. Nunca envie a chave ao baixar esta skill ou consultar documentação pública.

As chaves novas do modal de templates expiram em **1 hora** por padrão; a opção **Gerar chave de longa duração** emite uma chave válida por **2 meses de calendário**. O prompt informa o vencimento absoluto. Depois dele, a API/MCP recusa a chave e ela some do painel. Remova a credencial e a configuração desse MCP dos seus arquivos locais quando detectar o vencimento, somente se ainda usarem a chave vencida; peça uma nova chave ao usuário, sem tentar renovar o acesso por conta própria. A expiração invalida o segredo, mas não apaga mensagens antigas do histórico da conversa. Uma chave existente informada manualmente mantém seu prazo original.

## Conectar e identificar a conta

Use a configuração pessoal do cliente e preserve os outros servidores. O transporte é Streamable HTTP, sem OAuth próprio. Codex aceita `mcp_servers.frame_conexa` com `url` e `http_headers.Authorization` ou `bearer_token_env_var`; Claude Code aceita um servidor HTTP no escopo pessoal com header Bearer. Guarde segredos fora de repositórios e de logs. A chave da API tem acesso à conta, não apenas aos templates; um filtro de ferramentas no cliente não restringe a credencial no servidor.

Uma configuração salva não prova conexão. Descubra as ferramentas com `tools/list`, execute `list_instances` e depois `list_templates`. Se o cliente exigir recarregar a conversa, diga qual etapa falta. Se ele não suportar MCP remoto com Bearer, explique a limitação sem inventar uma conexão funcional.

IDs de instância são UUIDs opacos. IDs de template são strings numéricas da Meta. Confirme o número e a conta antes de escrever. No Frame Conexa, criar, editar ou excluir templates exige uma instância `kind="marketing"`, mesmo para templates de utilidade. A leitura funciona também em instâncias normais. Templates pertencem à WABA: números que compartilham a mesma WABA veem as alterações. Não troque de WABA silenciosamente.

## Preparar uma mensagem útil

Entenda a finalidade, o destinatário, a relação existente e o que muda em cada envio. Use variáveis para dados concretos como nome, data e referência de pedido. Mantenha o propósito explícito no texto fixo; não esconda ofertas em variáveis ou transforme o corpo inteiro em placeholders.

- **UTILITY:** atualização específica de uma solicitação, transação ou serviço existente. Uma confirmação de consulta deve identificar a consulta. Não acrescente venda cruzada ou promoção.
- **MARKETING:** ofertas, promoções, convites comerciais e reengajamento. Não prometa aprovação como utilidade ou menor cobrança por reformular artificialmente uma promoção.
- **AUTHENTICATION:** códigos de autenticação têm estrutura própria na Meta. O builder atual do Frame Conexa só aceita o formato simples sem variáveis, cabeçalho, rodapé ou botões nesta categoria; não presuma suporte completo a OTP. Explique limitações antes de submeter.

A Meta decide a categoria e a aprovação. Texto válido no Frame Conexa ainda pode ser rejeitado. Confira a política oficial vigente se houver dúvida sobre um caso específico; não prometa aprovação, prazo ou preço.

## Contrato das ferramentas

Consulte o schema real das ferramentas antes de montar o payload. Atualmente o builder aceita:

| Campo | Regra |
| --- | --- |
| `name` | 1–512 caracteres: letras minúsculas sem acento, números e `_` |
| `category` | `UTILITY`, `MARKETING` ou `AUTHENTICATION` |
| `language` | Código da Meta; use `pt_BR` para português brasileiro |
| `body` | Obrigatório, até 1024 caracteres |
| `bodyExamples` | Um exemplo fictício por variável do corpo, na ordem |
| `headerText` | Até 60 caracteres; no máximo uma variável `{{1}}` |
| `headerExample` | Obrigatório quando o cabeçalho tem variável |
| `footer` | Até 60 caracteres, sem variáveis |
| `buttons` | Até 10, agrupados por tipo `QUICK_REPLY` ou `URL`; rótulo até 25 caracteres |
| `buttons[].url` | HTTP(S), até 2000 caracteres; uma variável `{{1}}` apenas no final |
| `buttons[].urlExample` | URL completa de exemplo quando a URL é variável |
| `allowCategoryChange` | Na criação, padrão `true`; permite recategorização pela Meta |

Variáveis do corpo são `{{1}}`, `{{2}}`, etc., sem saltos. A numeração do cabeçalho e da URL é independente. Exemplos devem conter valores finais, nunca novos placeholders ou dados pessoais reais. Não envie componentes de mídia ou tipos de botão que o schema não suporta.

Exemplo de conteúdo para uma consulta já agendada, a completar com o `instanceId` retornado pela conta:

```json
{
  "name": "lembrete_consulta",
  "category": "UTILITY",
  "language": "pt_BR",
  "body": "Olá, {{1}}. Sua consulta na Clínica Exemplo está agendada para {{2}}, às {{3}}. Você confirma sua presença?",
  "bodyExamples": ["Ana", "12/10/2026", "14h"],
  "buttons": [
    { "type": "QUICK_REPLY", "text": "Confirmar presença" },
    { "type": "QUICK_REPLY", "text": "Preciso reagendar" }
  ]
}
```

## Criar, editar e acompanhar

1. Leia `list_templates` na instância correta para identificar duplicatas, idiomas, IDs, status e conteúdo atual.
2. Prepare os campos e mostre uma prévia com exemplos preenchidos. Informe a categoria e o efeito da próxima ação. Se a pessoa pediu apenas um rascunho, entregue o rascunho. Se pediu submissão, respeite a autorização existente; peça informação apenas quando faltar algo necessário.
3. `create_template` cria e já envia para análise da Meta; não existe uma ferramenta separada de “publicar rascunho”. `update_template` recebe `instanceId`, `templateId` e **todos** os campos do conteúdo final: é substituição completa, não patch. Preserve cabeçalho, rodapé, exemplos e botões que a pessoa não pediu para remover, reconstruindo-os dos componentes retornados pela leitura.
4. Após criar ou editar, consulte `list_templates` para verificar o conteúdo e status. A edição volta à análise. Diferencie aceitação da solicitação, `PENDING` e aprovação `APPROVED`. Reporte o status real; não fique consultando indefinidamente esperando aprovação.
5. Se houver rejeição, leia o motivo devolvido, explique o ajuste proposto e preserve a intenção original. Não transforme conteúdo promocional em falsa utilidade para contornar a decisão.

`delete_template` remove uma tradução pelo ID da Meta, preservando outros idiomas. É irreversível; confirme o alvo e a intenção antes de excluir, conforme as permissões do cliente. Não exclua e recrie automaticamente para contornar erros ou restrições de edição. O nome pode ficar indisponível para reutilização na Meta.

Em timeout ou erro ambíguo de escrita, consulte o estado remoto antes de repetir para não duplicar a operação. Respeite `429` e `Retry-After`. `401` indica chave ausente, inválida ou revogada; não tente fabricar uma chave. O erro `instancia_normal` exige escolher uma instância de marketing autorizada, não mudar artificialmente o tipo do número.

Conectar o MCP não autoriza enviar mensagens a contatos. Criação de templates e submissão à Meta são diferentes de `send_message` ou `create_campaign`. Não use disparos para testar a conexão e não desative a confirmação de envio da conta.

## Referências

- [MCP do Frame Conexa](https://frameconexa.com/docs/guides/mcp.md): transporte, autenticação e erros.
- [Contrato da API](https://frameconexa.com/docs/openapi.json): consulte os caminhos de templates para detalhes HTTP.
- [MCP no Codex](https://developers.openai.com/codex/mcp).
- [MCP no Claude Code](https://code.claude.com/docs/en/mcp).
