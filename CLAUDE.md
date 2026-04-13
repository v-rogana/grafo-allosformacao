# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# Allos Wiki — Schema

## Sobre este projeto

Este é um wiki de conhecimento sobre o conteúdo educacional do canal da Associação Allos
(@associacaoallos no YouTube). O wiki é mantido pelo Claude Code e visualizado no Obsidian.

A Allos é uma clínica-escola de psicologia em Belo Horizonte que forma psicólogos através
de prática clínica supervisionada, oferecendo atendimento acessível.

## Estrutura de diretórios

- `raw/sources/` — fontes brutas (artigos, PDFs). Somente leitura.
- `raw/audio/` — áudios extraídos do YouTube. Somente leitura.
- `raw/transcripts/` — transcrições geradas pelo Whisper. Somente leitura.
- `wiki/sources/` — resumo de cada fonte processada (1 arquivo por vídeo/artigo).
- `wiki/concepts/` — páginas de conceitos teóricos (ex: alianca-terapeutica.md).
- `wiki/entities/` — páginas de entidades: pessoas, autores, abordagens, instituições.
- `wiki/comparisons/` — análises comparativas geradas por queries.
- `wiki/meta/index.md` — catálogo de todas as páginas do wiki.
- `wiki/meta/log.md` — registro cronológico de operações.
- `wiki/meta/overview.md` — síntese geral do que o wiki cobre.

## Convenções

### Nomes de arquivo
- Usar kebab-case: `alianca-terapeutica.md`, `carl-rogers.md`
- Sem acentos nos nomes de arquivo: `analise-do-comportamento.md`
- Acentos normais dentro do conteúdo dos arquivos

### Links internos (Obsidian)
- Usar `[[nome-do-arquivo]]` para links entre páginas
- Usar `[[nome-do-arquivo|Texto visível]]` quando o texto do link difere do nome
- Todo conceito mencionado que tenha página própria deve ser linkado na primeira menção

### Frontmatter (YAML)
Toda página deve ter frontmatter:

```yaml
---
title: Nome Legível do Conceito
type: source | concept | entity | comparison
tags: [psicologia, tcc, aliança-terapêutica]
sources: [nome-do-video]
created: 2026-04-08
updated: 2026-04-08
---
```

### Estrutura de uma página de source (vídeo)

```markdown
---
title: "Título do Vídeo"
type: source
tags: [...]
url: https://youtube.com/watch?v=...
duration: "1h23m"
speakers: [Nome1, Nome2]
created: 2026-04-08
updated: 2026-04-08
---

# Título do Vídeo

## Resumo
2-3 parágrafos com as ideias principais.

## Pontos-chave
- Ponto 1 com [[link-para-conceito]]
- Ponto 2 ...

## Conceitos abordados
Lista de [[conceitos]] discutidos no vídeo.

## Citações notáveis
Trechos relevantes (com timestamp se disponível).

## Conexões
Como este conteúdo se relaciona com outros no wiki.
```

## Workflows

### INGEST (processar nova fonte)

Quando eu disser "ingest [arquivo]" ou "processa [arquivo]":

1. Leia o arquivo de transcrição em `raw/transcripts/`
2. Discuta comigo os principais takeaways
3. Crie a página de source em `wiki/sources/`
4. Para cada conceito importante mencionado:
   - Se já existe página em `wiki/concepts/`: atualize com novas informações
   - Se não existe: crie nova página
5. Para cada entidade (pessoa, autor, abordagem) mencionada:
   - Se já existe página em `wiki/entities/`: atualize
   - Se não existe: crie nova página
6. Atualize `wiki/meta/index.md` com as novas páginas
7. Atualize `wiki/meta/log.md` com entrada datada
8. Atualize `wiki/meta/overview.md` se necessário
9. Apresente um resumo do que foi criado/atualizado

### QUERY (consultar o wiki)

Quando eu fizer uma pergunta sobre o conteúdo:

1. Leia `wiki/meta/index.md` para encontrar páginas relevantes
2. Leia as páginas relevantes
3. Sintetize a resposta com [[links]] para as fontes
4. Se a resposta gerar insight novo valioso, pergunte se devo salvá-la como página

### LINT (manutenção)

Quando eu disser "lint" ou "health check":

1. Verifique links quebrados (referências a páginas que não existem)
2. Encontre páginas órfãs (sem links de entrada)
3. Identifique conceitos mencionados mas sem página própria
4. Note contradições entre páginas
5. Sugira lacunas de conhecimento e possíveis novas fontes
6. Reporte o estado geral do wiki

### Estrutura de uma página de concept

```markdown
---
title: Nome Legível do Conceito
type: concept
tags: [psicologia, ...]
sources: [nome-do-video]
created: 2026-04-08
updated: 2026-04-08
---

# Nome Legível do Conceito

Definição e explicação do conceito (2-3 parágrafos).

## Contexto teórico
Origem, escola de pensamento, autores associados com [[links]].

## Aplicação clínica
Como o conceito aparece na prática clínica.

## Fontes
- [[nome-do-video]] — contexto em que aparece
```

### Estrutura de uma página de entity

```markdown
---
title: Nome da Entidade
type: entity
tags: [autor | abordagem | instituição, ...]
created: 2026-04-08
updated: 2026-04-08
---

# Nome da Entidade

Descrição breve.

## Conceitos associados
- [[conceito-1]]
- [[conceito-2]]

## Mencionado em
- [[nome-do-video]]
```

## Ferramentas

### Transcrição via Groq API
`tools/transcribe-groq.py` — transcreve áudios usando Whisper Large V3 via Groq API.

```bash
python tools/transcribe-groq.py
```

O script espera arquivos de áudio particionados (`part1.mp3`, `part2.mp3`, `part3.mp3`) em `raw/audio/`.
Para vídeos longos, particione o áudio antes de rodar (limite da API Groq: 25 MB por arquivo).
Saída vai para `raw/transcripts/`.

> **ATENÇÃO:** Este script contém uma API key hardcoded. Nunca comitar em repositório público.

### Publicação web com Quartz
O diretório `quartz/` contém uma instância do [Quartz 4](https://quartz.jzhao.xyz/) configurada para publicar o wiki como site estático.
- Config em `quartz/quartz.config.ts` — título "Wiki Allos", locale pt-BR
- Deploy target: `v-rogana.github.io/grafo-allosformacao`
- Suporta Obsidian-flavored markdown, wikilinks, grafo interativo, busca
- O conteúdo do wiki precisa ser copiado ou linkado em `quartz/content/` para publicação
- Requer Node >= 22

```bash
# Build do site estático
cd quartz && npx quartz build

# Dev server com hot reload
cd quartz && npx quartz serve

# Verificar tipos e formatação
cd quartz && npm run check
```

## Pipeline completo

```
YouTube → yt-dlp (áudio .wav) → Groq/Whisper (transcrição .txt) → Claude Code (wiki .md) → Obsidian (visualização) / Quartz (publicação web)
```

## Domínio

Conceitos que provavelmente vão aparecer com frequência:

- Abordagens terapêuticas: TCC, ACT, psicanálise, humanismo, gestalt, etc.
- Conceitos clínicos: aliança terapêutica, manejo, enquadre, transferência, etc.
- Formação: supervisão, prática clínica, estudo de caso, ética profissional
- Pesquisa: psicologia baseada em evidências, metodologia, escalas, instrumentos
- Pessoas: autores, pesquisadores, teóricos referenciados nos vídeos
