# MarkItDown-MCP

> [!IMPORTANT]
> O pacote MarkItDown-MCP foi feito para **uso local**, com agentes locais confiáveis. Em especial, ao executar o servidor MCP com Streamable HTTP ou SSE, ele se vincula a `localhost` por padrão e não fica exposto a outras máquinas da rede ou da internet. Nessa configuração, ele serve como alternativa direta ao transporte STDIO, o que pode ser mais conveniente em alguns casos. NÃO vincule o servidor a outras interfaces a menos que você compreenda as [implicações de segurança](#considerações-de-segurança) dessa decisão.


[![PyPI](https://img.shields.io/pypi/v/markitdown-mcp.svg)](https://pypi.org/project/markitdown-mcp/)
![PyPI - Downloads](https://img.shields.io/pypi/dd/markitdown-mcp)
[![Built by AutoGen Team](https://img.shields.io/badge/Built%20by-AutoGen%20Team-blue)](https://github.com/microsoft/autogen)

O pacote `markitdown-mcp` oferece um servidor MCP leve com suporte a STDIO, Streamable HTTP e SSE para chamar o MarkItDown.

Ele expõe uma única ferramenta: `convert_to_markdown(uri)`, em que uri pode ser qualquer URI `http:`, `https:`, `file:` ou `data:`.

## Instalação

Para instalar o pacote, use o pip:

```bash
pip install markitdown-mcp
```

## Uso

Para executar o servidor MCP usando STDIO (padrão), use o comando a seguir:


```bash
markitdown-mcp
```

Para executar o servidor MCP usando Streamable HTTP e SSE, use o comando a seguir:

```bash
markitdown-mcp --http --host 127.0.0.1 --port 3001
```

## Executando no Docker

Para executar o `markitdown-mcp` no Docker, construa a imagem Docker usando o Dockerfile fornecido:
```bash
docker build -t markitdown-mcp:latest .
```

E execute-a com:
```bash
docker run -it --rm markitdown-mcp:latest
```
Isso é suficiente para URIs remotas. Para acessar arquivos locais, você precisa montar o diretório local dentro do contêiner. Por exemplo, se você quiser acessar arquivos em `/home/user/data`, execute:

```bash
docker run -it --rm -v /home/user/data:/workdir markitdown-mcp:latest
```

Uma vez montado, todos os arquivos sob data ficarão acessíveis em `/workdir` no contêiner. Por exemplo, se você tem um arquivo `example.txt` em `/home/user/data`, ele estará acessível no contêiner em `/workdir/example.txt`.

## Acessando pelo Claude Desktop

Recomenda-se usar a imagem Docker ao executar o servidor MCP para o Claude Desktop.

Siga [estas instruções](https://modelcontextprotocol.io/quickstart/user#for-claude-desktop-users) para acessar o arquivo `claude_desktop_config.json` do Claude.

Edite-o para incluir a seguinte entrada JSON:

```json
{
  "mcpServers": {
    "markitdown": {
      "command": "docker",
      "args": [
        "run",
        "--rm",
        "-i",
        "markitdown-mcp:latest"
      ]
    }
  }
}
```

Se você quiser montar um diretório, ajuste conforme necessário:

```json
{
  "mcpServers": {
    "markitdown": {
      "command": "docker",
      "args": [
        "run",
        "--rm",
        "-i",
        "-v",
        "/home/user/data:/workdir",
        "markitdown-mcp:latest"
      ]
    }
  }
}
```

## Depuração

Para depurar o servidor MCP, você pode usar a ferramenta `MCP Inspector`.

```bash
npx @modelcontextprotocol/inspector
```

Em seguida, conecte-se ao inspector pelo host e pela porta especificados (por exemplo, `http://localhost:5173/`).

Se estiver usando STDIO:
* selecione `STDIO` como tipo de transporte,
* informe `markitdown-mcp` como comando e
* clique em `Connect`

Se estiver usando Streamable HTTP:
* selecione `Streamable HTTP` como tipo de transporte,
* informe `http://127.0.0.1:3001/mcp` como URL e
* clique em `Connect`

Se estiver usando SSE:
* selecione `SSE` como tipo de transporte,
* informe `http://127.0.0.1:3001/sse` como URL e
* clique em `Connect`

Por fim:
* clique na aba `Tools`,
* clique em `List Tools`,
* clique em `convert_to_markdown` e
* execute a ferramenta com qualquer URI válida.

## Considerações de segurança

O servidor não oferece suporte a autenticação e roda com os privilégios do usuário que o executa. Por esse motivo, ao rodar em modo SSE ou Streamable HTTP, o servidor se vincula por padrão a `localhost`. Ainda assim, é importante reconhecer que o servidor pode ser acessado por qualquer processo ou usuário na mesma máquina local, e que a ferramenta `convert_to_markdown` pode ser usada para ler qualquer arquivo ao qual o usuário do servidor tenha acesso, ou quaisquer dados vindos da rede. Se você precisa de segurança adicional, considere executar o servidor em um ambiente isolado, como uma máquina virtual ou contêiner, e garanta que as permissões do usuário estejam configuradas de forma a limitar o acesso a arquivos sensíveis e a segmentos de rede. Acima de tudo, NÃO vincule o servidor a outras interfaces (fora do localhost) a menos que você compreenda as implicações de segurança dessa decisão.

## Marcas registradas

Este projeto pode conter marcas registradas ou logotipos de projetos, produtos ou serviços. O uso autorizado de marcas
registradas ou logotipos da Microsoft está sujeito e deve seguir as
[Diretrizes de Marca Registrada e Marca da Microsoft](https://www.microsoft.com/en-us/legal/intellectualproperty/trademarks/usage/general).
O uso de marcas registradas ou logotipos da Microsoft em versões modificadas deste projeto não deve causar confusão nem sugerir patrocínio da Microsoft.
Qualquer uso de marcas registradas ou logotipos de terceiros está sujeito às políticas desses terceiros.
