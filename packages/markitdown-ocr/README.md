# Plugin de OCR do MarkItDown

Plugin de LLM Vision para o MarkItDown que extrai texto de imagens incorporadas em arquivos PDF, DOCX, PPTX e XLSX.

Usa o mesmo padrão `llm_client` / `llm_model` que o MarkItDown já suporta para descrições de imagens, sem exigir novas bibliotecas de aprendizado de máquina nem dependências binárias.

## Recursos

- **Conversor de PDF aprimorado**: extrai texto de imagens dentro de PDFs, com OCR de página inteira como alternativa para documentos digitalizados
- **Conversor de DOCX aprimorado**: OCR para imagens em documentos do Word
- **Conversor de PPTX aprimorado**: OCR para imagens em apresentações do PowerPoint
- **Conversor de XLSX aprimorado**: OCR para imagens em planilhas do Excel
- **Preservação de contexto**: mantém a estrutura e o fluxo do documento ao inserir o texto extraído

## Instalação

```bash
pip install markitdown-ocr
```

O plugin usa qualquer cliente compatível com a API da OpenAI que você já tenha. Instale um caso ainda não tenha:

```bash
pip install openai
```

## Uso

### Linha de comando

```bash
markitdown document.pdf --use-plugins --llm-client openai --llm-model gpt-4o
```

### API Python

Informe `llm_client` e `llm_model` ao `MarkItDown()` exatamente como você faria para descrições de imagens:

```python
from markitdown import MarkItDown
from openai import OpenAI

md = MarkItDown(
    enable_plugins=True,
    llm_client=OpenAI(),
    llm_model="gpt-4o",
)

result = md.convert("document_with_images.pdf")
print(result.text_content)
```

Se nenhum `llm_client` for fornecido, o plugin ainda é carregado, mas o OCR é silenciosamente ignorado, recorrendo ao conversor padrão embutido.

### Prompt personalizado

Substitua o prompt de extração padrão para documentos especializados:

```python
md = MarkItDown(
    enable_plugins=True,
    llm_client=OpenAI(),
    llm_model="gpt-4o",
    llm_prompt="Extract all text from this image, preserving table structure.",
)
```

### Qualquer cliente compatível com a API da OpenAI

Funciona com qualquer cliente que siga a API da OpenAI:

```python
from openai import AzureOpenAI

md = MarkItDown(
    enable_plugins=True,
    llm_client=AzureOpenAI(
        api_key="...",
        azure_endpoint="https://your-resource.openai.azure.com/",
        api_version="2024-02-01",
    ),
    llm_model="gpt-4o",
)
```

## Como funciona

Quando `MarkItDown(enable_plugins=True, llm_client=..., llm_model=...)` é chamado:

1. O MarkItDown descobre o plugin pelo grupo de entry point `markitdown.plugin`
2. Ele chama `register_converters()`, repassando todos os kwargs, incluindo `llm_client` e `llm_model`
3. O plugin cria um `LLMVisionOCRService` a partir desses kwargs
4. Quatro conversores com OCR são registrados com **prioridade -1.0**, antes dos conversores embutidos, que têm prioridade 0.0

Quando um arquivo é convertido:

1. O conversor com OCR aceita o arquivo
2. Ele extrai as imagens incorporadas no documento
3. Cada imagem é enviada ao LLM com um prompt de extração
4. O texto retornado é inserido no fluxo do texto, preservando a estrutura do documento
5. Se a chamada ao LLM falhar, a conversão continua sem o texto daquela imagem

## Formatos de arquivo suportados

### PDF

- As imagens incorporadas são extraídas por posição (via `page.images` / XObjects da página) e passam por OCR no fluxo do texto, intercaladas com o conteúdo ao redor na ordem vertical de leitura.
- **PDFs digitalizados** (páginas sem texto extraível) são detectados automaticamente: cada página é renderizada a 300 DPI e enviada ao LLM como imagem de página inteira.
- **PDFs malformados** que o pdfplumber/pdfminer não consegue abrir (por exemplo, com EOF truncado) são reprocessados com a renderização de páginas do PyMuPDF, de modo que o conteúdo ainda é recuperado.

### DOCX

- As imagens são extraídas pelas relações das partes do documento (`doc.part.rels`).
- O OCR é executado antes do pipeline DOCX→HTML→Markdown: tokens de marcação são injetados no HTML para que o conversor de markdown não escape os marcadores de OCR, e os marcadores finais são substituídos pelos blocos formatados `*[Image OCR]...[End OCR]*` após a conversão.
- O fluxo do documento (títulos, parágrafos, tabelas) é totalmente preservado ao redor dos blocos de OCR.

### PPTX

- Formas de imagem, formas de espaço reservado com imagens e imagens dentro de grupos são todas suportadas.
- As formas são processadas na ordem de leitura de cima para a esquerda em cada slide.
- Se um `llm_client` estiver configurado, o LLM é consultado primeiro para gerar uma descrição; o OCR é usado como alternativa quando nenhuma descrição é retornada.

### XLSX

- As imagens incorporadas nas planilhas (`sheet._images`) são extraídas por planilha.
- A posição da célula é calculada a partir das coordenadas de ancoragem da imagem (coluna/linha → notação de letras do Excel).
- As imagens são listadas em uma seção `### Images in this sheet:` após a tabela de dados da planilha, e não são intercaladas nas linhas da tabela.

### Formato de saída

Todo bloco de OCR extraído é delimitado assim:

```text
*[Image OCR]
<texto extraído>
[End OCR]*
```

## Solução de problemas

### O texto do OCR não aparece na saída

A causa mais provável é a ausência de `llm_client` ou `llm_model`. Verifique:

```python
from openai import OpenAI
from markitdown import MarkItDown

md = MarkItDown(
    enable_plugins=True,
    llm_client=OpenAI(),   # obrigatório
    llm_model="gpt-4o",    # obrigatório
)
```

### O plugin não está carregando

Confirme que o plugin está instalado e foi descoberto:

```bash
markitdown --list-plugins   # deve exibir: ocr
```

### Erros de API

O plugin propaga os erros da API do LLM como avisos e continua a conversão. Verifique sua chave de API, sua cota e se o modelo escolhido suporta entradas de imagem.

## Desenvolvimento

### Executando os testes

```bash
cd packages/markitdown-ocr
pytest tests/ -v
```

### Construindo a partir do código-fonte

```bash
git clone https://github.com/microsoft/markitdown.git
cd markitdown/packages/markitdown-ocr
pip install -e .
```

## Contribuindo

Contribuições são bem-vindas! Veja o [repositório do MarkItDown](https://github.com/microsoft/markitdown) para as diretrizes.

## Licença

MIT, veja [LICENSE](LICENSE).

## Registro de mudanças

### 0.1.0 (versão inicial)

- OCR com LLM Vision para PDF, DOCX, PPTX e XLSX
- OCR de página inteira como alternativa para PDFs digitalizados
- Inserção de texto no fluxo do documento com preservação de contexto
- Substituição de conversores baseada em prioridade (sem necessidade de alterar código)
