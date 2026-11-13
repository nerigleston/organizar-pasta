# organizar-pasta

Skill para o [Claude Code](https://claude.com/claude-code) que organiza uma pasta bagunçada (a pasta Downloads, por padrão) abrindo cada arquivo e decidindo pelo conteúdo, não pelo nome.

O nome do arquivo engana. Um `Apólice.pdf` pode ser a fatura do cartão, e a apólice de verdade pode estar num arquivo chamado `files.pdf`. Se a organização seguir o nome, vai tudo para o lugar errado.

## O que ela faz

1. **Lê cada arquivo:** PDF, imagem (inclusive HEIC), docx/xlsx, texto, zip. Também acha duplicados por hash e arquivos que não abrem.
2. **Mostra um plano em tabela antes de mexer em qualquer coisa:** arquivo atual, o que ele é, para onde vai e o nome novo.
3. **Espera você conferir e dizer "pode fazer".** Só então cria as pastas e renomeia com nomes descritivos, como `Fatura cartão - 2026-09.pdf` ou `Escala plantão - 2026-10.pdf`.

Regras que ela segue:

- Nunca apaga nada. Duplicados, arquivos estragados e o que não deu para identificar vão para uma pasta `Revisar/`, com o motivo na tabela.
- Nunca sobrescreve (`mv -n`).
- Reaproveita as pastas que você já tem antes de criar novas.
- Não mexe em pastas de projeto (com `.git`, `node_modules` etc.).
- Antes de mover, gera um script de desfazer em `Revisar/` que devolve cada arquivo ao lugar original.
- Senhas, chaves e recovery codes vão para uma pasta separada, e o conteúdo deles não aparece na conversa.

Feita para macOS (usa `sips`, `textutil`, `mdls`). A lógica também serve em Linux, trocando esses comandos.

## Instalação

```bash
git clone https://github.com/nerigleston/organizar-pasta ~/.claude/skills/organizar-pasta
```

## Uso

No Claude Code, digite `/organizar-pasta` ou peça em linguagem natural:

> organiza minha pasta Downloads

> arruma a pasta ~/Desktop pelo conteúdo dos arquivos

Antes de usar, vale ligar o acesso só à pasta que você quer organizar, e não ao computador inteiro.

## Licença

MIT
