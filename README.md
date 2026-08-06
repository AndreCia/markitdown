# MarkItDown

[![PyPI](https://img.shields.io/pypi/v/markitdown.svg)](https://pypi.org/project/markitdown/)
![PyPI - Downloads](https://img.shields.io/pypi/dd/markitdown)
[![Built by AutoGen Team](https://img.shields.io/badge/Built%20by-AutoGen%20Team-blue)](https://github.com/microsoft/autogen)

> [!IMPORTANT]
> O MarkItDown realiza operações de entrada e saída com os privilégios do processo atual. Assim como `open()` ou `requests.get()`, ele acessa os recursos que o próprio processo consegue acessar. Sanitize suas entradas em ambientes não confiáveis e chame a função `convert_*` mais restrita que atenda ao seu caso de uso (por exemplo, `convert_stream()` ou `convert_local()`). Veja a seção [Considerações de segurança](#considerações-de-segurança) da documentação para mais informações.

O MarkItDown é um utilitário Python leve para converter diversos tipos de arquivo em Markdown, voltado ao uso com LLMs e com pipelines de análise de texto. Nesse sentido, ele é mais comparável ao [textract](https://github.com/deanmalmgren/textract), mas com foco em preservar a estrutura e o conteúdo importantes do documento em Markdown (incluindo títulos, listas, tabelas, links, etc.). Embora o resultado seja quase sempre apresentável e legível para pessoas, ele foi pensado para ser consumido por ferramentas de análise de texto e pode não ser a melhor opção para conversões de alta fidelidade destinadas à leitura humana.

Atualmente, o MarkItDown suporta a conversão de:

- PDF
- PowerPoint
- Word
- Excel
- Imagens (metadados EXIF e OCR)
- Áudio (metadados EXIF e transcrição de fala)
- HTML
- Formatos baseados em texto (CSV, JSON, XML)
- Arquivos ZIP (percorre o conteúdo)
- URLs do YouTube
- EPubs
- ... e mais!

## Por que Markdown?

O Markdown é extremamente próximo do texto puro, com marcação e formatação mínimas, mas ainda assim
oferece uma forma de representar a estrutura importante de um documento. LLMs populares, como o
GPT-4o da OpenAI, "_falam_" Markdown nativamente e com frequência incorporam Markdown em suas
respostas sem que isso seja solicitado. Isso sugere que foram treinadas com enormes volumes de
texto formatado em Markdown e que o compreendem bem. Como benefício adicional, as convenções do
Markdown também são bastante eficientes em consumo de tokens.

## Pré-requisitos
O MarkItDown requer Python 3.10 ou superior. Recomenda-se usar um ambiente virtual para evitar conflitos de dependências.

Com a instalação padrão do Python, você pode criar e ativar um ambiente virtual com os comandos abaixo:

```bash
python -m venv .venv
source .venv/bin/activate
```

Se estiver usando `uv`, você pode criar um ambiente virtual com:

```bash
uv venv --python=3.12 .venv
source .venv/bin/activate
# OBSERVAÇÃO: use 'uv pip install' em vez de apenas 'pip install' para instalar pacotes neste ambiente virtual
```

Se estiver usando Anaconda, você pode criar um ambiente virtual com:

```bash
conda create -n markitdown python=3.12
conda activate markitdown
```

## Instalação

Para instalar o MarkItDown, use o pip: `pip install 'markitdown[all]'`. Como alternativa, você pode instalá-lo a partir do código-fonte:

```bash
git clone git@github.com:microsoft/markitdown.git
cd markitdown
pip install -e 'packages/markitdown[all]'
```

## Uso

### Linha de comando

```bash
markitdown path-to-file.pdf > document.md
```

Ou use `-o` para especificar o arquivo de saída:

```bash
markitdown path-to-file.pdf -o document.md
```

Você também pode enviar o conteúdo por pipe:

```bash
cat path-to-file.pdf | markitdown
```

### Dependências opcionais
O MarkItDown possui dependências opcionais que ativam o suporte a vários formatos de arquivo. Antes, neste documento, instalamos todas as dependências opcionais com a opção `[all]`. No entanto, você também pode instalá-las individualmente para ter mais controle. Por exemplo:

```bash
pip install 'markitdown[pdf, docx, pptx]'
```

instalará apenas as dependências para arquivos PDF, DOCX e PPTX.

No momento, as seguintes dependências opcionais estão disponíveis:

* `[all]` Instala todas as dependências opcionais
* `[pptx]` Instala as dependências para arquivos do PowerPoint
* `[docx]` Instala as dependências para arquivos do Word
* `[xlsx]` Instala as dependências para arquivos do Excel
* `[xls]` Instala as dependências para arquivos antigos do Excel
* `[pdf]` Instala as dependências para arquivos PDF
* `[outlook]` Instala as dependências para mensagens do Outlook
* `[az-doc-intel]` Instala as dependências para o Azure Document Intelligence
* `[az-content-understanding]` Instala as dependências para o Azure Content Understanding
* `[audio-transcription]` Instala as dependências para transcrição de áudio de arquivos wav e mp3
* `[youtube-transcription]` Instala as dependências para obter a transcrição de vídeos do YouTube

### Plugins

O MarkItDown também suporta plugins de terceiros. Os plugins vêm desativados por padrão. Para listar os plugins instalados:

```bash
markitdown --list-plugins
```

Para ativar os plugins, use:

```bash
markitdown --use-plugins path-to-file.pdf
```

Para encontrar plugins disponíveis, procure no GitHub pela hashtag `#markitdown-plugin`. Para desenvolver um plugin, veja `packages/markitdown-sample-plugin`.

#### Plugin markitdown-ocr

O plugin `markitdown-ocr` adiciona suporte a OCR aos conversores de PDF, DOCX, PPTX e XLSX, extraindo texto de imagens incorporadas por meio de LLM Vision, com o mesmo padrão `llm_client` / `llm_model` que o MarkItDown já usa para descrições de imagens. Não é necessária nenhuma biblioteca de aprendizado de máquina nova nem dependência binária.

**Instalação:**

```bash
pip install markitdown-ocr
pip install openai  # ou qualquer cliente compatível com a API da OpenAI
```

**Uso:**

Informe o mesmo `llm_client` e `llm_model` que você usaria para descrições de imagens:

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

Se nenhum `llm_client` for fornecido, o plugin ainda é carregado, mas o OCR é silenciosamente ignorado e o conversor padrão embutido é usado no lugar.

Veja [`packages/markitdown-ocr/README.md`](packages/markitdown-ocr/README.md) para a documentação detalhada.

### Azure Content Understanding

O [Azure Content Understanding](https://learn.microsoft.com/azure/ai-services/content-understanding/) oferece conversão de qualidade superior com extração estruturada de campos (front matter em YAML), suporte multimodal (documentos, imagens, áudio, vídeo) e analisadores configuráveis.

Instalação: `pip install 'markitdown[az-content-understanding]'`

#### Quando usar o Content Understanding

O Content Understanding é ideal quando você precisa de recursos que vão além do que os conversores embutidos ou o Document Intelligence oferecem:

- **Arquivos de áudio e vídeo:** o CU é a única opção para vídeo e a opção em nuvem de maior qualidade para áudio. Os conversores embutidos não têm suporte a vídeo e oferecem apenas transcrição básica de áudio.
- **Extração estruturada de campos:** analisadores [pré-construídos](https://learn.microsoft.com/azure/ai-services/content-understanding/concepts/prebuilt-analyzers) ou [personalizados](https://learn.microsoft.com/azure/ai-services/content-understanding/how-to/customize-analyzer-content-understanding-studio?tabs=portal) extraem campos específicos do domínio (valores de faturas, datas de recibos, cláusulas contratuais) serializados como front matter em YAML. Nem a versão embutida nem a integração com o Doc Intel expõem esses campos.
- **Extração de documentos com qualidade superior:** análise de layout e OCR na nuvem para PDFs digitalizados, tabelas complexas e documentos de várias páginas.
- **Uma única API para todas as modalidades:** um único `cu_endpoint` lida com documentos, imagens, áudio e vídeo, com roteamento automático de analisadores.

| Recurso | Conversores embutidos | Azure Document Intelligence | Azure Content Understanding |
|------------|---------------------|-----------------------------|-----------------------------|
| Conversão de documentos | Offline, extração específica por formato | Extração de layout na nuvem | Extração multimodal na nuvem |
| Campos estruturados | Não disponível | Não exposto por esta integração | Front matter em YAML a partir dos campos do analisador |
| Analisadores personalizados | Não disponível | Não configurável nesta integração | Suportado via `cu_analyzer_id` |
| Áudio e vídeo | Áudio básico, sem vídeo | Não suportado | Analisadores de áudio e vídeo |
| Custo | Apenas processamento local | Chamadas cobradas à API do Azure | Chamadas cobradas à API do Azure |

**Linha de comando:**

```bash
markitdown path-to-file.pdf --use-cu --cu-endpoint "<content_understanding_endpoint>"
```

**API Python:**

```python
from markitdown import MarkItDown

# Sem configuração: seleciona automaticamente o analisador por tipo de arquivo
md = MarkItDown(cu_endpoint="<content_understanding_endpoint>")
result = md.convert("report.pdf")   # documentos → prebuilt-documentSearch
result = md.convert("meeting.mp4")  # vídeo → prebuilt-videoSearch
result = md.convert("call.wav")     # áudio → prebuilt-audioSearch
print(result.markdown)
```

**Com um analisador personalizado** (para extração de campos específicos do domínio):

```python
md = MarkItDown(
    cu_endpoint="<content_understanding_endpoint>",
    cu_analyzer_id="my-invoice-analyzer",
)
result = md.convert("invoice.pdf")
print(result.markdown)
# A saída inclui front matter em YAML com os campos extraídos:
# ---
# contentType: document
# fields:
#   VendorName: CONTOSO LTD.
#   InvoiceDate: '2019-11-15'
# ---
# <!-- page 1 -->
# ...
```

Quando `cu_analyzer_id` é definido, o conversor restringe automaticamente seu uso aos tipos de arquivo compatíveis, com base na modalidade do analisador. Tipos incompatíveis (por exemplo, arquivos de áudio com um analisador de documentos) são roteados automaticamente para os analisadores pré-construídos padrão.

**Observação sobre custos:** cada chamada de `convert()` para um formato roteado ao CU é uma chamada cobrada à API do Azure. Use `cu_file_types` para restringir quais formatos são roteados ao CU:

```python
from markitdown.converters import ContentUnderstandingFileType

md = MarkItDown(
    cu_endpoint="<content_understanding_endpoint>",
    cu_file_types=[ContentUnderstandingFileType.PDF],  # apenas PDFs usam o CU
)
```

Mais informações sobre o Azure Content Understanding podem ser encontradas [aqui](https://learn.microsoft.com/azure/ai-services/content-understanding/).

### Azure Document Intelligence

Para usar o Microsoft Document Intelligence na conversão:

```bash
markitdown path-to-file.pdf -o document.md -d -e "<document_intelligence_endpoint>"
```

Mais informações sobre como configurar um recurso do Azure Document Intelligence podem ser encontradas [aqui](https://learn.microsoft.com/en-us/azure/ai-services/document-intelligence/how-to-guides/create-document-intelligence-resource?view=doc-intel-4.0.0)

### API Python

Uso básico em Python:

```python
from markitdown import MarkItDown

md = MarkItDown(enable_plugins=False) # Defina como True para ativar os plugins
result = md.convert("test.xlsx")
print(result.text_content)
```

Conversão com Document Intelligence em Python:

```python
from markitdown import MarkItDown

md = MarkItDown(docintel_endpoint="<document_intelligence_endpoint>")
result = md.convert("test.pdf")
print(result.text_content)
```

Para usar modelos de linguagem grandes na descrição de imagens (atualmente apenas para arquivos pptx e de imagem), informe `llm_client` e `llm_model`:

```python
from markitdown import MarkItDown
from openai import OpenAI

client = OpenAI()
md = MarkItDown(llm_client=client, llm_model="gpt-4o", llm_prompt="optional custom prompt")
result = md.convert("example.jpg")
print(result.text_content)
```

### Docker

```sh
docker build -t markitdown:latest .
docker run --rm -i markitdown:latest < ~/your-file.pdf > output.md
```

## Contribuindo

Este projeto recebe bem contribuições e sugestões. A maioria das contribuições exige que você concorde com um
Contrato de Licença de Contribuidor (CLA), declarando que você tem o direito de nos conceder, e de fato nos concede,
os direitos de uso da sua contribuição. Para detalhes, acesse https://cla.opensource.microsoft.com.

Quando você envia um pull request, um bot de CLA determina automaticamente se você precisa fornecer
um CLA e sinaliza o PR de forma apropriada (por exemplo, com uma verificação de status ou um comentário). Basta seguir as instruções
fornecidas pelo bot. Você só precisará fazer isso uma vez em todos os repositórios que usam nosso CLA.

Este projeto adotou o [Código de Conduta de Código Aberto da Microsoft](https://opensource.microsoft.com/codeofconduct/).
Para mais informações, veja o [FAQ do Código de Conduta](https://opensource.microsoft.com/codeofconduct/faq/) ou
entre em contato pelo e-mail [opencode@microsoft.com](mailto:opencode@microsoft.com) com quaisquer perguntas ou comentários adicionais.

### Como contribuir

Você pode ajudar analisando issues ou revisando PRs. Qualquer issue ou PR é bem-vindo, mas também marcamos alguns como 'open for contribution' e 'open for reviewing' para facilitar as contribuições da comunidade. São apenas sugestões, é claro, e você pode contribuir da forma que preferir.

<div align="center">

|            | Todos                                                          | Especialmente precisam de ajuda da comunidade                                                                                                      |
| ---------- | ------------------------------------------------------------ | ----------------------------------------------------------------------------------------------------------------------------------------------- |
| **Issues** | [Todas as issues](https://github.com/microsoft/markitdown/issues) | [Issues abertas para contribuição](https://github.com/microsoft/markitdown/issues?q=is%3Aissue+is%3Aopen+label%3A%22open+for+contribution%22) |
| **PRs**    | [Todos os PRs](https://github.com/microsoft/markitdown/pulls)     | [PRs abertos para revisão](https://github.com/microsoft/markitdown/pulls?q=is%3Apr+is%3Aopen+label%3A%22open+for+reviewing%22)              |

</div>

### Executando testes e verificações

- Vá até o pacote MarkItDown:

  ```sh
  cd packages/markitdown
  ```

- Instale o `hatch` no seu ambiente e execute os testes:

  ```sh
  pip install hatch  # Outras formas de instalar o hatch: https://hatch.pypa.io/dev/install/
  hatch shell
  hatch test
  ```

  (Alternativa) Use o Devcontainer, que já tem todas as dependências instaladas:

  ```sh
  # Reabra o projeto no Devcontainer e execute:
  hatch test
  ```

- Execute as verificações do pre-commit antes de enviar um PR: `pre-commit run --all-files`

### Considerações de segurança

O MarkItDown realiza operações de entrada e saída com os privilégios do processo atual. Assim como `open()` ou `requests.get()`, ele acessa os recursos que o próprio processo consegue acessar.

**Sanitize suas entradas:** não passe entradas não confiáveis diretamente ao MarkItDown. Se qualquer parte da entrada puder ser controlada por um usuário ou sistema não confiável, como em aplicações hospedadas ou do lado do servidor, ela precisa ser validada e restringida antes de chamar o MarkItDown. Dependendo do seu ambiente, isso pode incluir restringir caminhos de arquivo, limitar esquemas de URI e destinos de rede, e bloquear o acesso a endereços privados, de loopback, link-local ou de serviços de metadados.

**Chame apenas o método de conversão de que você precisa:** prefira a API de conversão mais restrita que atenda ao seu caso de uso. O método `convert()` do MarkItDown é intencionalmente permissivo e consegue lidar com arquivos locais, URIs remotas e fluxos de bytes. Se sua aplicação só precisa ler arquivos locais, chame `convert_local()` em vez dele. Se você precisa de mais controle sobre a busca de URIs, chame `requests.get()` por conta própria e passe o objeto de resposta para `convert_response()`. Para controle máximo, abra um fluxo para a entrada que deseja converter e chame `convert_stream()`.

### Contribuindo com plugins de terceiros

Você também pode contribuir criando e compartilhando plugins de terceiros. Veja `packages/markitdown-sample-plugin` para mais detalhes.

## Marcas registradas

Este projeto pode conter marcas registradas ou logotipos de projetos, produtos ou serviços. O uso autorizado de marcas
registradas ou logotipos da Microsoft está sujeito e deve seguir as
[Diretrizes de Marca Registrada e Marca da Microsoft](https://www.microsoft.com/en-us/legal/intellectualproperty/trademarks/usage/general).
O uso de marcas registradas ou logotipos da Microsoft em versões modificadas deste projeto não deve causar confusão nem sugerir patrocínio da Microsoft.
Qualquer uso de marcas registradas ou logotipos de terceiros está sujeito às políticas desses terceiros.
