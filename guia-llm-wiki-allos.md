# LLM Wiki — Guia Prático para o Canal da Allos

## Visão Geral

```
YouTube Video → yt-dlp (áudio) → Whisper (transcrição) → Claude Code (wiki) → Obsidian (visualização)
```

---

## Etapa 1: Instalar Dependências

### 1.1 — yt-dlp (baixar áudio do YouTube)

```bash
# Instala via pip
pip install yt-dlp

# Verifica
yt-dlp --version
```

### 1.2 — ffmpeg (necessário pro yt-dlp extrair áudio)

```bash
# Se não tiver instalado
sudo apt install ffmpeg

# Verifica
ffmpeg -version
```

### 1.3 — Whisper (transcrição de áudio → texto)

Você tem duas opções. Recomendo a **opção A** pra começar:

**Opção A: whisper.cpp (mais rápido, roda em CPU)**

```bash
# Clona e compila
git clone https://github.com/ggerganov/whisper.cpp.git
cd whisper.cpp
cmake -B build
cmake --build build --config Release

# Baixa o modelo medium (bom equilíbrio qualidade/velocidade pro português)
./models/download-ggml-model.sh medium

# Testa
./build/bin/whisper-cli -m models/ggml-medium.bin -l pt -f caminho/para/audio.wav
```

**Opção B: OpenAI Whisper (Python, mais simples mas mais lento em CPU)**

```bash
pip install openai-whisper

# Uso:
whisper audio.wav --language pt --model medium --output_format txt
```

> **Nota sobre modelos:** `medium` é o sweet spot pro português. `small` é 2x mais rápido mas perde qualidade. `large-v3` é melhor mas bem mais lento sem GPU.

### 1.4 — Obsidian

```bash
# Opção 1: Flatpak (recomendado no Linux)
flatpak install flathub md.obsidian.Obsidian

# Opção 2: AppImage (baixar do site)
# https://obsidian.md/download → Linux → AppImage
# chmod +x Obsidian-*.AppImage && ./Obsidian-*.AppImage

# Opção 3: Snap
sudo snap install obsidian --classic
```

---

## Etapa 2: Estrutura do Projeto

Cria essa estrutura de pastas no lugar que você preferir:

```bash
mkdir -p ~/allos-wiki/{raw/{sources,audio,transcripts},wiki/{sources,concepts,entities,comparisons,meta}}

# Resultado:
# ~/allos-wiki/
# ├── raw/                    # Fontes brutas (imutáveis)
# │   ├── sources/            # Artigos, PDFs, materiais
# │   ├── audio/              # Áudios extraídos dos vídeos
# │   └── transcripts/        # Transcrições do Whisper
# ├── wiki/                   # Wiki gerada pelo LLM
# │   ├── sources/            # Resumos de cada fonte (1 por vídeo)
# │   ├── concepts/           # Páginas de conceitos (ex: aliança-terapeutica.md)
# │   ├── entities/           # Páginas de entidades (ex: pessoas, autores, abordagens)
# │   ├── comparisons/        # Análises comparativas
# │   └── meta/               # index.md, log.md, overview.md
# └── CLAUDE.md               # Schema pro Claude Code
```

```bash
# Inicializa os arquivos meta
touch ~/allos-wiki/wiki/meta/index.md
touch ~/allos-wiki/wiki/meta/log.md
touch ~/allos-wiki/wiki/meta/overview.md
```

Depois, abre o Obsidian → "Open folder as vault" → aponta pra `~/allos-wiki/wiki/`. O Obsidian vai ler a pasta `wiki/` e montar o grafo automaticamente.

---

## Etapa 3: Pipeline de Transcrição (teste com 1 vídeo)

### 3.1 — Escolha um vídeo do canal

Vai no canal https://www.youtube.com/@associacaoallos e copia a URL de um vídeo. Idealmente escolha um com conteúdo educacional rico (aula, palestra, discussão clínica) em vez de um anúncio curto.

### 3.2 — Baixar o áudio

```bash
cd ~/allos-wiki/raw/audio

# Baixa apenas o áudio em formato wav (melhor pro Whisper)
yt-dlp -x --audio-format wav -o "%(title)s.%(ext)s" "URL_DO_VIDEO"

# Se quiser ver os vídeos disponíveis antes:
yt-dlp --flat-playlist --print "%(id)s %(title)s" "https://www.youtube.com/@associacaoallos/videos"
```

### 3.3 — Transcrever

```bash
cd ~/allos-wiki/raw/transcripts

# Com whisper.cpp (ajuste o caminho do binário):
~/whisper.cpp/build/bin/whisper-cli \
  -m ~/whisper.cpp/models/ggml-medium.bin \
  -l pt \
  -f ../audio/"NOME_DO_ARQUIVO.wav" \
  --output-txt \
  -of "nome-do-video"

# OU com OpenAI Whisper:
whisper ../audio/"NOME_DO_ARQUIVO.wav" \
  --language pt \
  --model medium \
  --output_format txt \
  --output_dir .
```

> **Tempo estimado:** Um vídeo de 1h leva ~5-15min no `medium` com CPU razoável. Com GPU CUDA é ~1-2min.

### 3.4 — Conferir a transcrição

```bash
# Dá uma olhada rápida no resultado
head -50 ~/allos-wiki/raw/transcripts/nome-do-video.txt
wc -w ~/allos-wiki/raw/transcripts/nome-do-video.txt  # conta palavras
```

---

## Etapa 4: O Schema (CLAUDE.md)

Esse é o arquivo mais importante — ele diz pro Claude Code como operar o wiki. Cria em `~/allos-wiki/CLAUDE.md`:

```markdown
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
sources: [nome-do-video]  # quais fontes sustentam esta página
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

## Domínio

Conceitos que provavelmente vão aparecer com frequência:

- Abordagens terapêuticas: TCC, ACT, psicanálise, humanismo, gestalt, etc.
- Conceitos clínicos: aliança terapêutica, manejo, enquadre, transferência, etc.
- Formação: supervisão, prática clínica, estudo de caso, ética profissional
- Pesquisa: psicologia baseada em evidências, metodologia, escalas, instrumentos
- Pessoas: autores, pesquisadores, teóricos referenciados nos vídeos
```

---

## Etapa 5: Primeiro Ingest com Claude Code

Com tudo pronto, a sequência é:

```bash
# 1. Abre o Claude Code na raiz do projeto
cd ~/allos-wiki
claude

# 2. Dentro do Claude Code, diga:
# "Leia o CLAUDE.md pra entender o projeto.
#  Depois processa o arquivo raw/transcripts/nome-do-video.txt como primeiro ingest."
```

O Claude Code vai:
- Ler o schema
- Ler a transcrição
- Discutir com você os pontos principais
- Criar as páginas no wiki
- Atualizar index e log

Você acompanha em tempo real no Obsidian — as páginas vão aparecendo, o grafo vai se formando.

---

## Dicas Extras

### Obsidian — plugins recomendados

Depois de instalar, ative estes plugins da comunidade:
- **Graph View** (já vem built-in) — o grafo de conhecimento
- **Dataview** — queries dinâmicas sobre o frontmatter das páginas
- **Tag Wrangler** — gerenciar tags
- **Backlinks** (built-in) — ver quem linka pra cada página

### Otimizações futuras

- **Batch ingest:** script bash que baixa + transcreve todos os vídeos do canal de uma vez
- **qmd:** search engine local pra quando o wiki crescer além de ~100 páginas
- **Marp:** gerar apresentações a partir do wiki (você já usa Remotion, pode ser complementar)
- **Git:** versionar o wiki com git pra ter histórico de evolução

### Script auxiliar: download + transcrição em um comando

```bash
#!/bin/bash
# Salvar como ~/allos-wiki/tools/ingest-video.sh
# Uso: ./tools/ingest-video.sh "URL_DO_VIDEO"

set -e

URL="$1"
AUDIO_DIR="$HOME/allos-wiki/raw/audio"
TRANSCRIPT_DIR="$HOME/allos-wiki/raw/transcripts"

echo "📥 Baixando áudio..."
cd "$AUDIO_DIR"
FILENAME=$(yt-dlp --print filename -o "%(title)s" "$URL")
yt-dlp -x --audio-format wav -o "%(title)s.%(ext)s" "$URL"

echo "🎙️ Transcrevendo..."
cd "$TRANSCRIPT_DIR"

# Ajuste o caminho do whisper conforme sua instalação:
whisper "$AUDIO_DIR/${FILENAME}.wav" \
  --language pt \
  --model medium \
  --output_format txt \
  --output_dir .

echo "✅ Transcrição salva em: $TRANSCRIPT_DIR/${FILENAME}.txt"
echo ""
echo "Próximo passo: abra o Claude Code em ~/allos-wiki e diga:"
echo "  ingest raw/transcripts/${FILENAME}.txt"
```

```bash
chmod +x ~/allos-wiki/tools/ingest-video.sh
```

---

## Checklist Rápido

- [ ] Instalar yt-dlp (`pip install yt-dlp`)
- [ ] Verificar ffmpeg (`ffmpeg -version`)
- [ ] Instalar Whisper (whisper.cpp ou openai-whisper)
- [ ] Instalar Obsidian
- [ ] Criar estrutura de pastas (`~/allos-wiki/...`)
- [ ] Criar `CLAUDE.md` na raiz
- [ ] Criar `index.md`, `log.md`, `overview.md` em `wiki/meta/`
- [ ] Escolher um vídeo de teste
- [ ] Baixar áudio com yt-dlp
- [ ] Transcrever com Whisper
- [ ] Abrir Obsidian apontando para `~/allos-wiki/wiki/`
- [ ] Abrir Claude Code em `~/allos-wiki/`
- [ ] Rodar o primeiro ingest!
