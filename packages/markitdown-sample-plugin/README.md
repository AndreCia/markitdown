# Plugin de exemplo do MarkItDown

[![PyPI](https://img.shields.io/pypi/v/markitdown-sample-plugin.svg)](https://pypi.org/project/markitdown-sample-plugin/)
![PyPI - Downloads](https://img.shields.io/pypi/dd/markitdown-sample-plugin)
[![Built by AutoGen Team](https://img.shields.io/badge/Built%20by-AutoGen%20Team-blue)](https://github.com/microsoft/autogen)


Este projeto mostra como criar um plugin de exemplo para o MarkItDown. As partes mais importantes são as seguintes:

Primeiro, implemente o seu DocumentConverter personalizado:

```python
from typing import BinaryIO, Any
from markitdown import MarkItDown, DocumentConverter, DocumentConverterResult, StreamInfo, PRIORITY_SPECIFIC_FILE_FORMAT

class RtfConverter(DocumentConverter):

    def __init__(
        self, priority: float = PRIORITY_SPECIFIC_FILE_FORMAT
    ):
        super().__init__(priority=priority)

    def accepts(
        self,
        file_stream: BinaryIO,
        stream_info: StreamInfo,
        **kwargs: Any,
    ) -> bool:

        # Implemente a lógica para verificar se o fluxo de arquivo é um arquivo RTF
        # ...
        raise NotImplementedError()


    def convert(
        self,
        file_stream: BinaryIO,
        stream_info: StreamInfo,
        **kwargs: Any,
    ) -> DocumentConverterResult:

        # Implemente a lógica para converter o fluxo de arquivo em Markdown
        # ...
        raise NotImplementedError()
```

Em seguida, garanta que o seu pacote implemente e exporte o seguinte:

```python
# A versão da interface de plugin que este plugin utiliza.
# A única versão suportada por enquanto é a 1.
__plugin_interface_version__ = 1

# O ponto de entrada principal do plugin. É chamado sempre que instâncias do MarkItDown são criadas.
def register_converters(markitdown: MarkItDown, **kwargs):
    """
    Chamado durante a construção de instâncias do MarkItDown para registrar os conversores fornecidos pelos plugins.
    """

    # Basta criar e anexar uma instância de RtfConverter
    markitdown.register_converter(RtfConverter())
```


Por fim, crie um entry point no arquivo `pyproject.toml`:

```toml
[project.entry-points."markitdown.plugin"]
sample_plugin = "markitdown_sample_plugin"
```

Aqui, o valor de `sample_plugin` pode ser qualquer chave, mas idealmente deve ser o nome do plugin. O valor é o nome totalmente qualificado do pacote que implementa o plugin.


## Instalação

Para usar o plugin com o MarkItDown, ele precisa estar instalado. Para instalar o plugin a partir do diretório atual, use:

```bash
pip install -e .
```

Depois que o pacote do plugin estiver instalado, verifique se ele está disponível para o MarkItDown executando:

```bash
markitdown --list-plugins
```

Para usar o plugin em uma conversão, utilize a flag `--use-plugins`. Por exemplo, para converter um arquivo RTF:

```bash
markitdown --use-plugins path-to-file.rtf
```

Em Python, os plugins podem ser ativados assim:

```python
from markitdown import MarkItDown

md = MarkItDown(enable_plugins=True)
result = md.convert("path-to-file.rtf")
print(result.text_content)
```

## Marcas registradas

Este projeto pode conter marcas registradas ou logotipos de projetos, produtos ou serviços. O uso autorizado de marcas
registradas ou logotipos da Microsoft está sujeito e deve seguir as
[Diretrizes de Marca Registrada e Marca da Microsoft](https://www.microsoft.com/en-us/legal/intellectualproperty/trademarks/usage/general).
O uso de marcas registradas ou logotipos da Microsoft em versões modificadas deste projeto não deve causar confusão nem sugerir patrocínio da Microsoft.
Qualquer uso de marcas registradas ou logotipos de terceiros está sujeito às políticas desses terceiros.
