# Frame Conexa Skills

Skills para operar o Frame Conexa com assistentes de IA.

## Templates de WhatsApp

[Leia a skill frame-conexa-templates](skills/frame-conexa-templates/SKILL.md).

A skill orienta criação, revisão, edição, exclusão e acompanhamento da análise de templates pela Meta. Ela usa o MCP remoto do Frame Conexa em `https://frameconexa.com/mcp`.

No painel, abra **Templates → Criar com IA**, escolha seu assistente, gere uma chave ou informe uma chave existente e copie o prompt. Cole-o no seu assistente para configurar a conexão e verificar os templates da conta.

Para instalar apenas as orientações, copie a pasta `skills/frame-conexa-templates` para o diretório pessoal de skills do cliente. Codex usa `~/.codex/skills`; Claude Code usa `~/.claude/skills`. A instalação da skill não configura a autenticação por si só.

O cliente precisa suportar MCP remoto com autenticação Bearer. Configurar um servidor durante uma conversa pode exigir recarregar o cliente. O agente deve confirmar a conexão por uma leitura autenticada.

A chave é privada e dá acesso à API da conta. Não a publique neste repositório. A Meta decide a aprovação dos templates; o Frame Conexa exige uma instância de marketing para criá-los, editá-los ou excluí-los.
