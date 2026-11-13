---
name: organizar-pasta
description: Organiza uma pasta bagunçada (Downloads por padrão) lendo o conteúdo de cada arquivo, não o nome. Mostra um plano em tabela antes de mexer, nunca apaga nada, manda duplicados e arquivos estragados para Revisar e renomeia com nomes descritivos. Use quando o usuário pedir para organizar, arrumar ou separar a pasta Downloads, Desktop ou outra pasta cheia de arquivos soltos como Scan, Documento, IMG ou Unknown.
---

# Organizar pasta pelo conteúdo

Arruma uma pasta abrindo cada arquivo e decidindo pelo que ele contém. O nome engana: um "Apólice.pdf" pode ser a fatura do cartão e a apólice de verdade pode estar num arquivo chamado "files". Por isso o nome do arquivo nunca decide sozinho para onde ele vai.

Fluxo: **ler → plano em tabela → aprovação → mover e renomear → script de desfazer.**

## Regras

- **Nunca apagar nada.** Nem duplicado, nem arquivo vazio, nem lixo aparente. Tudo que parecer descartável vai para `Revisar/`, com o motivo na tabela.
- **Nada se move antes do "pode fazer".** O usuário confere a tabela primeiro.
- **Só a pasta pedida.** Padrão: `~/Downloads`. Não sair dela nem seguir symlinks para fora.
- **Nível de cima apenas.** Organize os arquivos soltos na raiz. Pastas já existentes (projetos, repositórios, pastas com `.git`, `node_modules`, `package.json`) ficam onde estão, a menos que o usuário peça.
- **Nunca sobrescrever.** Use `mv -n`. Se o nome de destino já existir, acrescente ` (2)`, ` (3)`.
- **Credenciais e documentos sensíveis** (recovery codes, senhas, chaves, `.env`, passaporte, IRPF) não são abertos além do necessário para classificar e o conteúdo não aparece na resposta. Credencial vai para `Credenciais (mover para cofre)/` e o usuário é avisado.

## 1. Ler cada arquivo

Liste os arquivos soltos da raiz com tamanho e data (`ls -la`, `stat`). Depois abra um por um:

| Tipo | Como ler |
|---|---|
| PDF | Read com `pages: "1-2"`; se for escaneado sem texto, a imagem da página basta |
| Imagem (png, jpg, heic) | Read direto; HEIC: converter uma cópia em `sips -s format png` no scratchpad |
| docx, xlsx, pptx | `textutil -convert txt -stdout` para docx; skill xlsx/pptx ou `unzip -p` para os demais |
| txt, md, csv, json, sql | `head -40` |
| Áudio e vídeo | Não transcrever todos. Classifique pelo nome e metadados (`mdls`, `ffprobe`). Só transcreva se o usuário pedir, sem apagar o original |
| zip | `unzip -l` (não extrair) |
| Instalador (dmg, pkg, exe) | Pelo nome e tamanho |

Para marcar como `Revisar/`:
- **Duplicado:** mesmo `shasum -a 256`. Mantenha o mais antigo ou o de nome mais claro no destino; as outras cópias vão para Revisar.
- **Estragado:** não abre, tem 0 bytes ou `file` não reconhece o formato.
- **Não identificado:** abriu, mas não dá para saber do que se trata.

Com mais de ~30 arquivos, divida a leitura em lotes e mostre progresso em uma linha por lote.

## 2. Montar o plano

Antes de criar pastas, veja quais categorias já existem na raiz e reaproveite. Só crie pasta nova quando nenhuma existente servir. Categorias que costumam aparecer:

`Financeiro`, `Pessoal`, `Clientes/<nome do cliente>`, `Reuniões e transcrições`, `Conteúdo e imagens`, `Vídeos e gravações`, `Áudios do WhatsApp`, `Código, logs e exports`, `Instaladores e fontes`, `Revisar`.

Nome novo para cada arquivo: descreve o conteúdo, em português, com a data do documento (não a do download) quando houver. Ex.: `Fatura cartão Nubank - 2026-09.pdf`, `Escala plantão - 2026-10.pdf`, `Apólice seguro auto Porto - 2026.pdf`. Mantenha a extensão. Se o nome atual já for bom, não troque.

Mostre a tabela inteira:

| # | Arquivo atual | O que é (pelo conteúdo) | Destino | Nome novo |
|---|---|---|---|---|
| 1 | Apolice.pdf | Fatura do cartão, setembro/2026 | Financeiro/ | Fatura cartão - 2026-09.pdf |
| 2 | files.pdf | Apólice do seguro do carro | Pessoal/Seguros/ | Apólice seguro auto - 2026.pdf |
| 3 | Scan (1).pdf | Cópia idêntica do Scan.pdf | Revisar/ | Scan (1).pdf |

Abaixo da tabela, resuma em poucas linhas: quantos arquivos, pastas novas que serão criadas, quantos vão para Revisar e por quê (ex.: "2 duplicados, 3 que não abrem"). Destaque os casos em que o nome enganava, porque são os que o usuário precisa conferir.

Pare e espere. Se o usuário corrigir linhas, ajuste e mostre só as linhas alteradas.

## 3. Executar

Depois do "pode fazer":

1. Gere o script de desfazer **antes** de mover, em `Revisar/desfazer-organizacao-<AAAAMMDD-HHMMSS>.sh`, com uma linha `mv -n '<destino>' '<origem>'` por arquivo (`#!/bin/zsh` no topo, `chmod +x`).
2. Crie as pastas (`mkdir -p`) e mova com `mv -n`, já com o nome novo.
3. Confira: cada destino existe e a raiz não tem mais os arquivos movidos. Liste o que falhou, se algo falhou.

## 4. Relatório final

Curto: quantos arquivos foram movidos, pastas criadas, o que está em `Revisar/` com o motivo de cada um, e o caminho do script de desfazer. Lembre que nada foi apagado e que a limpeza de `Revisar/` fica com o usuário.
