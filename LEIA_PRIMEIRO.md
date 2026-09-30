# LEIA PRIMEIRO — Projeto MESA

São só três passos manuais. O resto o Claude Code faz sozinho, pulando o que já estiver pronto.

## 1. Ter o Claude Code

Se já tiver, pule para o passo 2.

- **Jeito mais fácil:** o app Claude Desktop, que permite usar o Claude Code sem terminal.
- **Pelo terminal (PowerShell):** `irm https://claude.ai/install.ps1 | iex`. Depois, abra um terminal novo e confira com `claude --version`.
- Exige assinatura **Pro ou Max** do Claude.
- Se o PC tiver a variável de ambiente `ANTHROPIC_API_KEY`, o Claude Code usa essa chave (cobrada à parte) em vez da assinatura.
- O Git for Windows é opcional.

## 2. Colocar a pasta do projeto no PC

Descompacte a pasta `mesa/` onde preferir (por exemplo, em Documentos).

## 3. Abrir o projeto e dar a partida

Abra o Claude Code dentro da pasta `mesa/` e mande:

> Vamos começar. Execute a Fase 0.

Pronto. O `CLAUDE.md` é lido automaticamente e diz ao Claude Code como trabalhar em cada fase.

**Sobre o Python:** você já tem. O script de ambiente confere a versão. Se ela for anterior à 3.10, instale a atual em python.org, marcando a opção "Add python.exe to PATH".
