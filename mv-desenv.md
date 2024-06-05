# Dockerfile Explicado

1. **Frontend (Node.js)**

    ```dockerfile
    FROM node:20-alpine AS frontend
    ```
    - Usa a imagem base do Node.js versão 20 em Alpine Linux e nomeia este estágio de construção como `frontend`.

    ```dockerfile
    RUN mkdir -p /home/node/app/node_modules && chown -R node:node /home/node/app
    ```
    - Cria o diretório `/home/node/app/node_modules` e define a propriedade do diretório e seus conteúdos para o usuário `node`.

    ```dockerfile
    WORKDIR /home/node/app
    ```
    - Define o diretório de trabalho para `/home/node/app`.

    ```dockerfile
    COPY ./frontend/package*.json ./
    ```
    - Copia os arquivos `package.json` e `package-lock.json` do diretório `frontend` do host para o diretório de trabalho no contêiner.

    ```dockerfile
    USER node
    ```
    - Troca o usuário para `node` para executar os próximos comandos com privilégios reduzidos.

    ```dockerfile
    RUN npm ci
    ```
    - Instala as dependências do Node.js listadas nos arquivos `package.json` e `package-lock.json`.

    ```dockerfile
    COPY --chown=node:node ./frontend/ ./frontend
    ```
    - Copia o conteúdo do diretório `frontend` do host para o contêiner com a propriedade definida para o usuário `node`.

    ```dockerfile
    COPY --chown=node:node ./static/ ./static
    ```
    - Copia o conteúdo do diretório `static` do host para o contêiner com a propriedade definida para o usuário `node`.

    ```dockerfile
    WORKDIR /home/node/app/frontend
    ```
    - Define o diretório de trabalho para `/home/node/app/frontend`.

    ```dockerfile
    RUN NODE_OPTIONS=--max_old_space_size=8192 npm run build
    ```
    - Define uma opção de configuração do Node.js para aumentar o tamanho máximo da memória heap e executa o comando `npm run build` para construir o aplicativo frontend.

2. **Backend (Python)**

    ```dockerfile
    FROM python:3.11-alpine
    ```
    - Usa a imagem base do Python versão 3.11 em Alpine Linux.

    ```dockerfile
    RUN apk add --no-cache --virtual .build-deps \  
        build-base \  
        libffi-dev \  
        openssl-dev \  
        curl \  
        && apk add --no-cache \  
        libpq
    ```
    - Instala pacotes necessários para compilar e rodar a aplicação Python, incluindo ferramentas de construção, bibliotecas de desenvolvimento e `libpq` (biblioteca do PostgreSQL).

    ```dockerfile
    COPY requirements.txt /usr/src/app/
    ```
    - Copia o arquivo `requirements.txt` para o diretório de trabalho `/usr/src/app` no contêiner.

    ```dockerfile
    RUN pip install --no-cache-dir -r /usr/src/app/requirements.txt \  
        && rm -rf /root/.cache
    ```
    - Instala as dependências do Python listadas em `requirements.txt` e remove o cache do `pip` para reduzir o tamanho da imagem.

    ```dockerfile
    COPY . /usr/src/app/
    ```
    - Copia todos os arquivos do diretório atual do host para o diretório de trabalho `/usr/src/app` no contêiner.

    ```dockerfile
    COPY --from=frontend /home/node/app/static /usr/src/app/static/
    ```
    - Copia o diretório `static` do estágio de construção `frontend` para o diretório `/usr/src/app/static` no estágio atual.

    ```dockerfile
    WORKDIR /usr/src/app
    ```
    - Define o diretório de trabalho para `/usr/src/app`.

    ```dockerfile
    EXPOSE 80
    ```
    - Expõe a porta 80 para permitir o tráfego HTTP.

    ```dockerfile
    CMD ["gunicorn", "-b", "0.0.0.0:80", "app:app"]
    ```
    - Define o comando padrão para iniciar o servidor Gunicorn, que serve a aplicação Python na porta 80.

# Casos de Teste
# Introdução
Abaixo estão alguns fluxos a serem seguidos para garantir que as mudanças de código sejam testadas de forma eficaz e eficiente. O objetivo é garantir que isso seja repetível e possa ser feito por qualquer pessoa para evitar regressões.

## Fluxos de Casos de Teste
- Com dados, streaming
- Com dados, sem streaming
- Sem dados, streaming
- Sem dados, sem streaming
- Todos os casos acima com e sem histórico de chat

### Com Dados, Streaming
As seguintes variáveis de ambiente são necessárias para executar este caso de teste:

`AZURE_SEARCH_SERVICE`  
`AZURE_SEARCH_INDEX`  
`AZURE_SEARCH_KEY`

`AZURE_OPENAI_STREAM` deve ser configurado como `true`

### Com Dados, Streaming, Histórico de Chat
Mantenha as mesmas variáveis de ambiente acima, mas adicione o seguinte:

`AZURE_COSMOSDB_DATABASE`  
`AZURE_COSMOSDB_ACCOUNT`  
`AZURE_COSMOSDB_CONVERSATIONS_CONTAINER`  
`AZURE_COSMOSDB_ACCOUNT_KEY`

### Com Dados, Sem Streaming
As seguintes variáveis de ambiente são necessárias para executar este caso de teste:

`AZURE_SEARCH_SERVICE`  
`AZURE_SEARCH_INDEX`  
`AZURE_SEARCH_KEY`

`AZURE_OPENAI_STREAM` deve ser configurado como `false`

### Com Dados, Sem Streaming, Histórico de Chat
Mantenha as mesmas variáveis de ambiente acima, mas adicione o seguinte:

`AZURE_COSMOSDB_DATABASE`  
`AZURE_COSMOSDB_ACCOUNT`  
`AZURE_COSMOSDB_CONVERSATIONS_CONTAINER`  
`AZURE_COSMOSDB_ACCOUNT_KEY`

### Sem Dados, Streaming
As seguintes variáveis de ambiente **não** devem ser configuradas:

`AZURE_SEARCH_SERVICE`  
`AZURE_SEARCH_INDEX`  
`AZURE_SEARCH_KEY`

`AZURE_OPENAI_STREAM` deve ser configurado como `true`

### Sem Dados, Streaming, Histórico de Chat
Mantenha as mesmas variáveis de ambiente acima, mas adicione o seguinte:

`AZURE_COSMOSDB_DATABASE`  
`AZURE_COSMOSDB_ACCOUNT`  
`AZURE_COSMOSDB_CONVERSATIONS_CONTAINER`  
`AZURE_COSMOSDB_ACCOUNT_KEY`

### Sem Dados, Sem Streaming
As seguintes variáveis de ambiente **não** devem ser configuradas:

`AZURE_SEARCH_SERVICE`  
`AZURE_SEARCH_INDEX`  
`AZURE_SEARCH_KEY`

`AZURE_OPENAI_STREAM` deve ser configurado como `false`

### Sem Dados, Sem Streaming, Histórico de Chat
Mantenha as mesmas variáveis de ambiente acima, mas adicione o seguinte:

`AZURE_COSMOSDB_DATABASE`  
`AZURE_COSMOSDB_ACCOUNT`  
`AZURE_COSMOSDB_CONVERSATIONS_CONTAINER`  
`AZURE_COSMOSDB_ACCOUNT_KEY`

# Script de inicialização start.sh
# Dockerfile Explicado

1. **Frontend (Node.js)**

    ```dockerfile
    FROM node:20-alpine AS frontend
    ```
    - Usa a imagem base do Node.js versão 20 em Alpine Linux e nomeia este estágio de construção como `frontend`.

    ```dockerfile
    RUN mkdir -p /home/node/app/node_modules && chown -R node:node /home/node/app
    ```
    - Cria o diretório `/home/node/app/node_modules` e define a propriedade do diretório e seus conteúdos para o usuário `node`.

    ```dockerfile
    WORKDIR /home/node/app
    ```
    - Define o diretório de trabalho para `/home/node/app`.

    ```dockerfile
    COPY ./frontend/package*.json ./
    ```
    - Copia os arquivos `package.json` e `package-lock.json` do diretório `frontend` do host para o diretório de trabalho no contêiner.

    ```dockerfile
    USER node
    ```
    - Troca o usuário para `node` para executar os próximos comandos com privilégios reduzidos.

    ```dockerfile
    RUN npm ci
    ```
    - Instala as dependências do Node.js listadas nos arquivos `package.json` e `package-lock.json`.

    ```dockerfile
    COPY --chown=node:node ./frontend/ ./frontend
    ```
    - Copia o conteúdo do diretório `frontend` do host para o contêiner com a propriedade definida para o usuário `node`.

    ```dockerfile
    COPY --chown=node:node ./static/ ./static
    ```
    - Copia o conteúdo do diretório `static` do host para o contêiner com a propriedade definida para o usuário `node`.

    ```dockerfile
    WORKDIR /home/node/app/frontend
    ```
    - Define o diretório de trabalho para `/home/node/app/frontend`.

    ```dockerfile
    RUN NODE_OPTIONS=--max_old_space_size=8192 npm run build
    ```
    - Define uma opção de configuração do Node.js para aumentar o tamanho máximo da memória heap e executa o comando `npm run build` para construir o aplicativo frontend.

2. **Backend (Python)**

    ```dockerfile
    FROM python:3.11-alpine
    ```
    - Usa a imagem base do Python versão 3.11 em Alpine Linux.

    ```dockerfile
    RUN apk add --no-cache --virtual .build-deps \  
        build-base \  
        libffi-dev \  
        openssl-dev \  
        curl \  
        && apk add --no-cache \  
        libpq
    ```
    - Instala pacotes necessários para compilar e rodar a aplicação Python, incluindo ferramentas de construção, bibliotecas de desenvolvimento e `libpq` (biblioteca do PostgreSQL).

    ```dockerfile
    COPY requirements.txt /usr/src/app/
    ```
    - Copia o arquivo `requirements.txt` para o diretório de trabalho `/usr/src/app` no contêiner.

    ```dockerfile
    RUN pip install --no-cache-dir -r /usr/src/app/requirements.txt \  
        && rm -rf /root/.cache
    ```
    - Instala as dependências do Python listadas em `requirements.txt` e remove o cache do `pip` para reduzir o tamanho da imagem.

    ```dockerfile
    COPY . /usr/src/app/
    ```
    - Copia todos os arquivos do diretório atual do host para o diretório de trabalho `/usr/src/app` no contêiner.

    ```dockerfile
    COPY --from=frontend /home/node/app/static /usr/src/app/static/
    ```
    - Copia o diretório `static` do estágio de construção `frontend` para o diretório `/usr/src/app/static` no estágio atual.

    ```dockerfile
    WORKDIR /usr/src/app
    ```
    - Define o diretório de trabalho para `/usr/src/app`.

    ```dockerfile
    EXPOSE 80
    ```
    - Expõe a porta 80 para permitir o tráfego HTTP.

    ```dockerfile
    CMD ["gunicorn", "-b", "0.0.0.0:80", "app:app"]
    ```
    - Define o comando padrão para iniciar o servidor Gunicorn, que serve a aplicação Python na porta 80.

# Explicação do Script start.sh

Este script automatiza o processo de preparação e inicialização de um aplicativo que inclui um frontend e um backend (em Python usando o Quart). Abaixo está a explicação linha por linha do script.

```bash
#!/bin/bash
```
- **#!/bin/bash**: Especifica que este script deve ser executado usando o shell Bash.

```bash
export NODE_OPTIONS=--max_old_space_size=8192
```
- **export NODE_OPTIONS=--max_old_space_size=8192**: Define a variável de ambiente `NODE_OPTIONS` para aumentar o tamanho máximo da memória heap do Node.js para 8192 MB.

```bash
echo ""
echo "Restoring frontend npm packages"
echo ""
```
- **echo ""**: Imprime uma linha em branco para a saída.
- **echo "Restoring frontend npm packages"**: Imprime uma mensagem indicando que o processo de restauração dos pacotes npm do frontend está começando.

```bash
cd frontend
```
- **cd frontend**: Muda o diretório de trabalho para o diretório `frontend`.

```bash
npm install
if [ $? -ne 0 ]; then
    echo "Failed to restore frontend npm packages"
    exit $?
fi
```
- **npm install**: Instala os pacotes npm listados no `package.json`.
- **if [ $? -ne 0 ]; then**: Verifica se o último comando (`npm install`) foi bem-sucedido. `$?` contém o código de saída do último comando executado.
- **echo "Failed to restore frontend npm packages"**: Se `npm install` falhar, imprime uma mensagem de erro.
- **exit $?**: Sai do script com o código de saída do comando `npm install`.

```bash
echo ""
echo "Building frontend"
echo ""
```
- **echo ""**: Imprime uma linha em branco para a saída.
- **echo "Building frontend"**: Imprime uma mensagem indicando que o processo de construção do frontend está começando.

```bash
npm run build
if [ $? -ne 0 ]; then
    echo "Failed to build frontend"
    exit $?
fi
```
- **npm run build**: Executa o script de construção do frontend definido no `package.json`.
- **if [ $? -ne 0 ]; then**: Verifica se o comando `npm run build` foi bem-sucedido.
- **echo "Failed to build frontend"**: Se `npm run build` falhar, imprime uma mensagem de erro.
- **exit $?**: Sai do script com o código de saída do comando `npm run build`.

```bash
cd ..
```
- **cd ..**: Volta para o diretório pai (a raiz do projeto).

```bash
. ./scripts/loadenv.sh
```
- **. ./scripts/loadenv.sh**: Executa o script `loadenv.sh` localizado no diretório `scripts` para carregar variáveis de ambiente necessárias.

```bash
echo ""
echo "Starting backend"
echo ""
```
- **echo ""**: Imprime uma linha em branco para a saída.
- **echo "Starting backend"**: Imprime uma mensagem indicando que o processo de inicialização do backend está começando.

```bash
./.venv/bin/python -m quart run --port=50505 --host=127.0.0.1 --reload
if [ $? -ne 0 ]; then
    echo "Failed to start backend"
    exit $?
fi
```
- **./.venv/bin/python -m quart run --port=50505 --host=127.0.0.1 --reload**: Inicia o servidor backend usando Quart, configurando-o para escutar na porta 50505 e no host 127.0.0.1 com a opção de recarregar automaticamente as mudanças no código.
- **if [ $? -ne 0 ]; then**: Verifica se o comando para iniciar o backend foi bem-sucedido.
- **echo "Failed to start backend"**: Se o comando falhar, imprime uma mensagem de erro.
- **exit $?**: Sai do script com o código de saída do comando para iniciar o backend.
```

# SECURITY_PT_BR.md
# Segurança

A Microsoft leva a segurança de seus produtos de software e serviços a sério, o que inclui todos os repositórios de código-fonte gerenciados através de nossas organizações no GitHub, que incluem [Microsoft](https://github.com/microsoft), [Azure](https://github.com/Azure), [DotNet](https://github.com/dotnet), [AspNet](https://github.com/aspnet), [Xamarin](https://github.com/xamarin) e [nossas organizações no GitHub](https://opensource.microsoft.com/).
```
Se você acredita que encontrou uma vulnerabilidade de segurança em qualquer repositório de propriedade da Microsoft que atenda à [definição de vulnerabilidade de segurança da Microsoft](https://aka.ms/opensource/security/definition), por favor, reporte-a conforme descrito abaixo.

## Relatando Problemas de Segurança

**Por favor, não reporte vulnerabilidades de segurança através de issues públicas no GitHub.**

Em vez disso, reporte-as ao Centro de Resposta de Segurança da Microsoft (MSRC) em [https://msrc.microsoft.com/create-report](https://aka.ms/opensource/security/create-report).

Se você preferir enviar sem fazer login, envie um e-mail para [secure@microsoft.com](mailto:secure@microsoft.com). Se possível, criptografe sua mensagem com nossa chave PGP; por favor, faça o download na [página da chave PGP do Centro de Resposta de Segurança da Microsoft](https://aka.ms/opensource/security/pgpkey).

Você deve receber uma resposta dentro de 24 horas. Se por algum motivo você não receber, por favor, siga com um e-mail para garantir que recebemos sua mensagem original. Informações adicionais podem ser encontradas em [microsoft.com/msrc](https://aka.ms/opensource/security/msrc).

Por favor, inclua as informações solicitadas listadas abaixo (tanto quanto possível) para nos ajudar a entender melhor a natureza e o escopo do possível problema:

  * Tipo de problema (por exemplo, buffer overflow, SQL injection, cross-site scripting, etc.)
  * Caminhos completos dos arquivos de origem relacionados à manifestação do problema
  * A localização do código-fonte afetado (tag/branch/commit ou URL direto)
  * Qualquer configuração especial necessária para reproduzir o problema
  * Instruções passo a passo para reproduzir o problema
  * Código de prova de conceito ou de exploração (se possível)
  * Impacto do problema, incluindo como um invasor poderia explorar o problema

Essas informações nos ajudarão a priorizar seu relatório mais rapidamente.

Se você estiver relatando para um programa de recompensa por bugs, relatórios mais completos podem contribuir para uma recompensa maior. Por favor, visite nossa página do [Programa de Recompensas por Bugs da Microsoft](https://aka.ms/opensource/security/bounty) para mais detalhes sobre nossos programas ativos.

## Idiomas Preferidos

Preferimos que todas as comunicações sejam em inglês.

## Política

A Microsoft segue o princípio de [Divulgação Coordenada de Vulnerabilidades](https://aka.ms/opensource/security/cvd).
  

# Generate the translated explanation in a markdown file - README_azd.md

# (Pré-visualização) Aplicativo de Chat de Exemplo com AOAI

## Implantação com a CLI do Desenvolvedor Azure

**IMPORTANTE:** Para implantar e executar este exemplo, você precisará de uma **assinatura do Azure com acesso habilitado para o serviço Azure OpenAI**. Você pode solicitar acesso [aqui](https://aka.ms/oaiapply). Você também pode visitar [aqui](https://azure.microsoft.com/free/cognitive-search/) para obter alguns créditos gratuitos do Azure para começar. Sua conta do Azure deve ter permissões `Microsoft.Authorization/roleAssignments/write`, como [Administrador de Acesso ao Usuário](https://learn.microsoft.com/azure/role-based-access-control/built-in-roles#user-access-administrator) ou [Proprietário](https://learn.microsoft.com/azure/role-based-access-control/built-in-roles#owner).

**CUSTOS DE RECURSOS AZURE:** por padrão, este exemplo criará recursos do Azure App Service e do Azure Cognitive Search que têm um custo mensal, bem como um recurso do Form Recognizer que tem um custo por página de documento. Você pode alterná-los para versões gratuitas de cada um deles, se quiser evitar esse custo, alterando o arquivo de parâmetros na pasta infra (embora haja alguns limites a considerar; por exemplo, você pode ter até 1 recurso gratuito do Cognitive Search por assinatura, e o recurso gratuito do Form Recognizer só analisa as primeiras 2 páginas de cada documento).

## Abrindo o projeto

Este projeto tem suporte a [Contêineres de Desenvolvimento](https://code.visualstudio.com/docs/devcontainers/containers), então ele será configurado automaticamente se você abri-lo no Github Codespaces ou no VS Code local com a [extensão Dev Containers](https://marketplace.visualstudio.com/items?itemName=ms-vscode-remote.remote-containers).

Se você não estiver usando uma dessas opções para abrir o projeto, então você precisará:

1. Instalar os pré-requisitos:

    - [CLI do Desenvolvedor Azure](https://aka.ms/azure-dev/install)
    - [Python 3+](https://www.python.org/downloads/)
        - **Importante**: O Python e o gerenciador de pacotes pip devem estar no caminho no Windows para que os scripts de configuração funcionem.
    - [Node.js com npx](https://nodejs.org/) (apenas usado para o desenvolvimento local de clientes React)
    - [Git](https://git-scm.com/downloads)

1. Configurar suas variáveis de ambiente:
    - Execute `azd init` para inicializar o ambiente Azure Developer para este aplicativo.
    - Execute `azd env set AZURE_OPENAI_SERVICE {Nome do serviço OpenAI do Azure}` para definir o nome do serviço OpenAI do Azure.
    - Execute `azd env set AZURE_OPENAI_RESOURCE_GROUP {Nome do grupo de recursos}` para definir o nome do grupo de recursos do OpenAI.
    - Execute `azd env set AZURE_OPENAI_SKU_NAME {Nome do SKU do OpenAI}`. O padrão é `S0`.
    - Execute `azd env set AZURE_FORMRECOGNIZER_SERVICE {Nome do serviço Form Recognizer}` para definir o nome do serviço Form Recognizer.
    - Execute `azd env set AZURE_FORMRECOGNIZER_RESOURCE_GROUP {Nome do grupo de recursos do Form Recognizer}` para definir o nome do grupo de recursos do Form Recognizer.
    - Execute `azd env set AZURE_FORMRECOGNIZER_SKU_NAME {Nome do SKU do Form Recognizer}`. O padrão é `S0`.

1. Execute `azd auth login` para fazer login na sua conta Azure.
1. Execute `azd up` para provisionar recursos do Azure e implantar este exemplo nesses recursos. Isso também executa um script para criar o índice de pesquisa com base nos arquivos na pasta `./data`.
1. Após a aplicação ter sido implantada com &#8203;:citation[oaicite:0]{index=0}&#8203;

# Explicação do Script data_collection.py

Este script realiza a coleta de dados e processamento de pares de perguntas e respostas (QA) usando um módulo de aplicação (`app.py`). O script carrega variáveis de ambiente, lê dados de um arquivo JSON, processa as perguntas através de uma API e grava os resultados em um arquivo JSONL.

# Generate the translated explanation in a markdown file

translated_data_collection_explanation = """
# Explicação do Script data_collection.py

Este script realiza a coleta de dados e processamento de pares de perguntas e respostas (QA) usando um módulo de aplicação (`app.py`). O script carrega variáveis de ambiente, lê dados de um arquivo JSON, processa as perguntas através de uma API e grava os resultados em um arquivo JSONL.

## Importações

```python
import os
import sys
import asyncio
import json
from dotenv import load_dotenv
```

- os, sys: Utilizados para manipulação do sistema operacional e do caminho de arquivos.
- asyncio: Utilizado para programação assíncrona.
- json: Utilizado para manipulação de dados em formato JSON.
- load_dotenv: Utilizado para carregar variáveis de ambiente de um arquivo .env.

```python
sys.path.append(os.path.abspath(os.path.join(os.path.dirname(__file__), '..')))
```

- Adiciona o diretório pai ao sys.path para permitir a importação do módulo app.py.

```python
import app
```

- load_env_into_module: Carrega variáveis de ambiente de um arquivo .env e define essas variáveis no módulo especificado (app neste caso).

```python
app.SHOULD_STREAM = False
app.SHOULD_USE_DATA = app.should_use_data()
```

- Define algumas configurações necessárias no app.py.

```python
generated_data_path = r"path/to/qa_input_file.json"
```

- Define o caminho do arquivo JSON contendo os pares de perguntas e respostas.

```python
with open(generated_data_path, 'r') as file:
    data = json.load(file)
```

- Lê os dados do arquivo JSON e os carrega na variável data.

```python
async def process(data: list, file):
  for qa_pairs_obj in data:
      qa_pairs = qa_pairs_obj["qa_pairs"]
      for qa_pair in qa_pairs:
          question = qa_pair["question"]
          messages = [{"role":"user", "content":question}]

          print("processing question "+question)

          request = {"messages":messages, "id":"1"}

          response = await app.complete_chat_request(request)

          messages = response["choices"][0]["messages"]

          tool_message = None
          assistant_message = None

          for message in messages:
            if message["role"] == "tool":
              tool_message = message["content"]
            elif message["role"] == "assistant":
              assistant_message = message["content"]
            else:
              raise ValueError("unknown message role")

          user_message = {"role":"user", "content":question}
          assistant_message = {"role":"assistant", "content":assistant_message}

          citations = json.loads(tool_message)
          assistant_message["context"] = citations

          messages = []
          messages.append(user_message)
          messages.append(assistant_message)

          evaluation_data = {"messages":messages}

          file.write(json.dumps(evaluation_data)+"\n")
          file.flush()
```

- Lê os dados do arquivo JSON e os carrega na variável data.

```python
async def process(data: list, file):
  for qa_pairs_obj in data:
      qa_pairs = qa_pairs_obj["qa_pairs"]
      for qa_pair in qa_pairs:
          question = qa_pair["question"]
          messages = [{"role":"user", "content":question}]

          print("processing question "+question)

          request = {"messages":messages, "id":"1"}

          response = await app.complete_chat_request(request)

          messages = response["choices"][0]["messages"]

          tool_message = None
          assistant_message = None

          for message in messages:
            if message["role"] == "tool":
              tool_message = message["content"]
            elif message["role"] == "assistant":
              assistant_message = message["content"]
            else:
              raise ValueError("unknown message role")

          user_message = {"role":"user", "content":question}
          assistant_message = {"role":"assistant", "content":assistant_message}

          citations = json.loads(tool_message)
          assistant_message["context"] = citations

          messages = []
          messages.append(user_message)
          messages.append(assistant_message)

          evaluation_data = {"messages":messages}

          file.write(json.dumps(evaluation_data)+"\n")
          file.flush()
```

- process: Função assíncrona que processa uma lista de pares de perguntas e respostas, enviando as perguntas para a API e escrevendo as respostas no arquivo de saída.

```python
evaluation_data_file_path = r"path/to/output_file.jsonl"
```

- Define o caminho do arquivo JSONL onde os dados processados serão salvos.

```python
with open(evaluation_data_file_path, "w") as file:
  asyncio.run(process(data, file))
```

- Abre o arquivo de saída para escrita e executa a função process para processar os dados e escrever os resultados no arquivo.

# Preparação de dados

# Prepare dados localmente

Siga as instruções nesta seção para preparar seus dados localmente. Isso é mais fácil para pequenos conjuntos de dados. Para conjuntos muito maiores, consulte as [instruções para usar AML abaixo](#use-aml-to-prepare-data).

## Configurar
- Instale os pacotes necessários listados em requisitos.txt, por exemplo.

```linux
pip install --user -r requisitos.txt
```

## Configure

- Crie um arquivo .env semelhante ao arquivo .env.example. Preencha os valores das variáveis ​​de ambiente.
- Crie um arquivo de configuração como `config.json`. O formato deve ser uma lista de objetos JSON, com cada objeto especificando uma configuração de caminho de dados local e serviço de pesquisa de destino e índice.

```
[
    {
        "data_path": "<local path or blob URL>",
        "location": "<azure region, e.g. 'westus2'>", 
        "subscription_id": "<subscription id>",
        "resource_group": "<resource group name>",
        "search_service_name": "<search service name to use or create>",
        "index_name": "<index name to use or create>",
        "chunk_size": 1024, // definido como null para desabilitar o chunking antes da ingestão
        "token_overlap": 128 // number of tokens to overlap between chunks
        "semantic_config_name": "default",
        "language": "en" // configuração para definir o idioma dos seus documentos. Altere se seus documentos não estiverem em inglês. Procure em data_preparation.py por SUPPORTED_LANGUAGE_CODES,
        "vector_config_name": "default" // usado se adicionar vetores ao índice
    }
]
```

Observação: `data_path` pode ser um caminho para arquivos localizados localmente em sua máquina ou uma URL de Blob do Azure, por exemplo. do formato `"https://<nome da conta de armazenamento>.blob.core.windows.net/<nome do contêiner>/<caminho>/"`. Se uma URL de blob for usada, os dados serão baixados primeiro do Blob Storage para um diretório temporário em seu computador antes de prosseguir com a preparação dos dados.

## Criar índices e ingerir dados
Isenção de responsabilidade: certifique-se de que não haja páginas duplicadas em seus dados. Isso pode afetar negativamente a qualidade das respostas que você obtém.

- Execute o script de preparação de dados, passando seu arquivo de configuração. Você pode definir njobs para análise paralela de seus arquivos.

    ```
    python data_preparation.py --config config.json --njobs=4
    ```

### Criação de índices em lote
Consulte o script run_batch_create_index.py para criar vários índices em lote usando um script.

## Opcional: Use URL de Blob do Azure para preparar dados
Cada documento pode ser associado a uma URL que é armazenada com cada pedaço de documento no índice do Azure Cognitive Search no campo "url". Se seus documentos foram baixados da Web, você poderá especificar um prefixo de URL a ser usado para construir as URLs do documento ao ingerir seus dados. Seu arquivo de configuração deve ter um parâmetro `url_prefix` adicional como este:

```
[
    {
        "data_path": "<local path or blob URL>",
        "url_prefix": "https://<source website URL>.com/"
        "location": "<azure region, e.g. 'westus2'>", 
        "subscription_id": "<subscription id>",
        "resource_group": "<resource group name>",
        "search_service_name": "<search service name to use or create>",
        "index_name": "<index name to use or create>",
        "chunk_size": 1024, // set to null to disable chunking before ingestion
        "token_overlap": 128 // number of tokens to overlap between chunks
        "semantic_config_name": "default",
        "language": "en" // setting to set language of your documents. Change if your documents are not in English. Look in data_preparation.py for SUPPORTED_LANGUAGE_CODES,
        "vector_config_name": "default" // used if adding vectors to index
    }
]
```

Para cada documento, a URL armazenada com pedaços desse documento será `url_prefix` concatenada com o caminho relativo do documento em `data_path`. Por exemplo, se meu `data_path` for `mydata` contendo a seguinte estrutura:

```
└───mydata
    │   overview.html
    │
    └───examples
            example1.html
            example2.html
```

And `url_prefix` is `"https://my-wiki.com/"`, the resulting URLs will be:
|File| URL|
|---|---|
|overview.html | `"https://my-wiki.com/overview.html"`|
|example1.html | `"https://my-wiki.com/examples/example1.html"`|
|example2.html | `"https://my-wiki.com/examples/example2.html"`|

Essas URLs podem então ser usadas ​​na exibição de citações no aplicativo da web. Consulte o [README](../README.md#umbling-citation-display) para obter mais detalhes.

Para cada documento, a URL armazenada com pedaços desse documento será `url_prefix` concatenada com o caminho relativo do documento em `data_path`. Por exemplo, se meu `data_path` for `mydata` contendo a seguinte estrutura:

```
└───mydata
    │   overview.html
    │
    └───examples
            example1.html
            example2.html
```
And `url_prefix` is `"https://my-wiki.com/"`, the resulting URLs will be:
|File| URL|
|---|---|
|overview.html | `"https://my-wiki.com/overview.html"`|
|example1.html | `"https://my-wiki.com/examples/example1.html"`|
|example2.html | `"https://my-wiki.com/examples/example2.html"`|

Essas URLs podem então ser usadas ​​na exibição de citações no aplicativo da web. Consulte o [README](../README.md#umbling-citation-display) para obter mais detalhes.

Se você tiver documentos de vários sites de origem, poderá especificar vários caminhos e prefixos seguindo o exemplo em `config_multiple_url.json`.

```
[
    {
        "data_paths": [
            {
                "path": "data/source1",
                "url_prefix": "https://<URL for source 1>.com/"
            },
            {
                "path": "data/source2",
                "url_prefix": "https://<URL for source 2>.com/"
            }
        ],
        "subscription_id": "<subscription id>",
        "resource_group": "<resource group name>",
        "search_service_name": "<search service name to use or create>",
        "index_name": "<index name to use or create>",
        "chunk_size": 1024,
        "token_overlap": 128,
        "semantic_config_name": "default",
        "language": "<Language to support for example use 'en' for English. Checked supported languages here under lucene - https://learn.microsoft.com/en-us/azure/search/index-add-language-analyzers"
    }
]
```

O script de ingestão percorrerá cada caminho em `data_paths` e construirá as URLs do documento seguindo o mesmo padrão descrito acima, usando o prefixo de URL específico para cada caminho de dados.

Você pode modificar a lógica de construção da URL em `process_file()` em [data_utils.py](./data_utils.py):

```
url_path = None
rel_file_path = os.path.relpath(file_path, directory_path)
if url_prefix:
    url_path = url_prefix + rel_file_path
    url_path = convert_escaped_to_posix(url_path)
```

## Opcional: Adicionar incorporações de vetores
A Pesquisa Cognitiva do Azure dá suporte à pesquisa vetorial em versão prévia pública. Consulte [os documentos](https://learn.microsoft.com/en-us/azure/search/vector-search-overview) para obter mais informações.

Para adicionar vetores ao seu índice, primeiro você precisará de um [recurso Azure OpenAI](https://learn.microsoft.com/en-us/azure/ai-services/openai/overview) com uma [implantação de modelo de incorporação Ada] (https://learn.microsoft.com/en-us/azure/ai-services/openai/concepts/models#embeddings-models). O modelo `text-embedding-ada-002` é suportado.

- Obtenha o endpoint para incorporar a implantação do modelo. O endpoint geralmente terá o formato `https://<nome do recurso Azure Openai>.openai.azure.com/openai/deployments/<nome de implantação ada>/embeddings?api-version=2023-06-01-preview` .
- Execute o script de preparação de dados, passando seu arquivo de configuração e o endpoint e a chave de incorporação como argumentos extras:

    
      `python data_preparation.py --config config.json --embedding-model-endpoint "<embedding endpoint>"`

## Opcional: transformar PDFs em texto
Se seus dados estiverem no formato PDF, primeiro você precisará converter do formato PDF para o formato .txt. Você pode usar seu próprio script para isso ou usar o código de conversão fornecido aqui.

### Configuração para cracking de PDF

- Crie um [reconhecedor de formulário](https://learn.microsoft.com/en-us/azure/applied-ai-services/form-recognizer/create-a-form-recognizer-resource?view=form-recog-3.0.0) na sua assinatura 
- Certifique-se de ter o SDK do Reconhecedor de Formulários: `pip install azure-ai-formrecognizer`
- Execute o seguinte comando para obter uma chave de acesso para o seu recurso Form Recognizer:

```
az cognitiveservices account keys list --name "<form-rec-resource-name>" --resource-group "<resource-group-name>"
```

Copie uma das chaves retornadas por este comando.

### Crie índices e carrgue os dados de PDF com o Reconhecedor de Formulários
Passe o nome e a chave do recurso do Form Recognizer ao executar o script de preparação de dados:

```
python data_preparation.py --config config.json --njobs=4 --form-rec-resource <form-rec-resource-name> --form-rec-key <form-rec-key>
```

Isso usará o modelo de leitura do Reconhecedor de Formulários por padrão. 

Se seus documentos tiverem muitas tabelas e informações de layout relevantes, você poderá usar o modelo Form Recognizer Layout, que é mais caro e mais lento de executar, mas preservará as informações da tabela com melhor qualidade. O modelo Layout também ajudará a preservar algumas informações de formatação do documento, como títulos e subtítulos, o que tornará as citações mais legíveis. Para usar o modelo Layout em vez do modelo Read padrão, passe o argumento `--form-rec-use-layout`.

```
python data_preparation.py --config config.json --njobs=4 --form-rec-resource <form-rec-resource-name> --form-rec-key <form-rec-key> --form-rec-use-layout
```

# Use AML (Azure Machine Learn) para preparar dados
## Configurar
- Instalar [Azure ML CLI v2](https://learn.microsoft.com/en-us/azure/machine-learning/concept-v2?view=azureml-api-2)

## Pré-requisitos
- Azure Machine Learning (AML) Workspace associada a Keyvault
- Azure Cognitive Search (ACS) resource
- (Optional if processing PDF) [Azure AI Document Intelligence](https://learn.microsoft.com/en-us/azure/ai-services/document-intelligence/overview?view=doc-intel-3.1.0) resource
- (Opcional se adicionar embeddings para vector search) Azure OpenAI resource with Ada (text-embedding-ada-002) deployment
- (Optional) Azure Blob Storage account
## Configurar
- Criar chaves no keyvault AML para a chave de administração de recursos do Azure Cognitive Search, a chave de acesso do Document Intelligence e a chave de API do Azure OpenAI (se estiver usando)
- Crie um arquivo de configuração como `aml_config.json`. O formato pode ser um único objeto JSON ou uma lista deles, com cada objeto especificando uma configuração de chaves do Keyvault, configurações de chunking e configuração de índice.
```
{
        "chunk_size": 1024,
        "token_overlap": 128,
        "keyvault_url": "https://<keyvault name>.vault.azure.net/",
        "document_intelligence_secret_name": "myDocIntelligenceKey",
        "document_intelligence_endpoint": "https://<document intelligence resource name>.cognitiveservices.azure.com/",
        "embedding_key_secret_name": "myAzureOpenAIKey",
        "embedding_endpoint": "https:/<azure openai resource name>.openai.azure.com/openai/deployments/<Ada deployment name>/embeddings?api-version=2023-06-01-preview",
        "index_name": "<new index name>",
        "search_service_name": "<search service name>",
        "search_key_secret_name": "mySearchServiceKey"
}
```

## Opcional: crie um armazenamento de dados AML
Se os seus dados estiverem no Armazenamento de Blobs do Azure, você poderá primeiro criar um Datastore AML que será usado para se conectar aos seus dados durante o upload. Atualize `datastore.yml` com as informações da sua conta de armazenamento, incluindo a chave da conta. Em seguida, execute este comando usando o grupo de recursos e o nome do espaço de trabalho do seu espaço de trabalho AML:

```
az ml datastore create --resource-group <workspace resource group> --workspace-name <workspace name> --file datastore.yml
```
## Envie o "work" do pipeline de processamento de dados
Em `pipeline.yml`, atualize as entradas para apontar para o nome do arquivo de configuração e o armazenamento de dados que você criou. Se você estiver usando dados armazenados localmente, comente o caminho do armazenamento de dados e remova o comentário do caminho local, atualizando para apontar para o local dos dados locais. Em seguida, envie o trabalho do pipeline para seu espaço de trabalho AML:

```
az ml job create --resource-group <workspace resource group> --workspace-name <workspace name> --file pipeline.yml
```