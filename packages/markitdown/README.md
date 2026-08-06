# MarkItDown

> [!TIP]
> O MarkItDown é um pacote Python e um utilitário de linha de comando para converter diversos tipos de arquivo em Markdown (por exemplo, para indexação, análise de texto, etc).
>
> Para mais informações e a documentação completa, veja o [README.md](https://github.com/microsoft/markitdown) do projeto no GitHub.

> [!IMPORTANT]
> O MarkItDown realiza operações de entrada e saída com os privilégios do processo atual. Assim como `open()` ou `requests.get()`, ele acessa os recursos que o próprio processo consegue acessar. Sanitize suas entradas em ambientes não confiáveis e chame a função `convert_*` mais restrita que atenda ao seu caso de uso (por exemplo, `convert_stream()` ou `convert_local()`). Veja a seção [Security Considerations](https://github.com/microsoft/markitdown#security-considerations) da documentação para mais informações.

## Instalação

A partir do PyPI:

```bash
pip install 'markitdown[all]'
```

A partir do código-fonte:

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

### API Python

```python
from markitdown import MarkItDown

md = MarkItDown()
result = md.convert("test.xlsx")
print(result.text_content)
```

### Mais informações

Para mais informações e a documentação completa, veja o [README.md](https://github.com/microsoft/markitdown) do projeto no GitHub.

## Marcas registradas

Este projeto pode conter marcas registradas ou logotipos de projetos, produtos ou serviços. O uso autorizado de marcas
registradas ou logotipos da Microsoft está sujeito e deve seguir as
[Diretrizes de Marca Registrada e Marca da Microsoft](https://www.microsoft.com/en-us/legal/intellectualproperty/trademarks/usage/general).
O uso de marcas registradas ou logotipos da Microsoft em versões modificadas deste projeto não deve causar confusão nem sugerir patrocínio da Microsoft.
Qualquer uso de marcas registradas ou logotipos de terceiros está sujeito às políticas desses terceiros.
