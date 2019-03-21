# API do gerador

## resolve

- **Argumentos**
  - `{string} _path` - caminho relativo da raiz do projeto

- **Retorna**
  - `{string}` - o caminho absoluto definido

- **Uso**:
Define um caminho para o projeto atual

## hasPlugin

- **Argumentos**
  - `{string} id` - id do plugin, pode omitir o prefixo (@vue/|vue-|@scope/vue)-cli-plugin-

- **Retorna**
  - `{boolean}`

- **Uso**:
Verifica se o projeto tem um plugin com o ID fornecido

## addConfigTransform

- **Argumentos**
  - `{string} key` - chave de configuração em package.json
  - `{object} options` - opções
  - `{object} options.file` - descritor de arquivo. Usado para procurar pelo arquivo existente. Cada chave é um tipo de arquivo (valores possíveis: ['js', 'json', 'yaml', 'lines']). O valor é uma lista de nomes de arquivos.
  Exemplo:
  ```js
  {
    js: ['.eslintrc.js'],
    json: ['.eslintrc.json', '.eslintrc']
  }
  ```
  Por padrão, o primeiro nome de arquivo será usado para criar o arquivo de configuração.

- **Retorna**
  - `{boolean}`

- **Uso**:
Configura como os arquivos de configuração são extraídos.

## extendPackage

- **Argumentos**
  - `{object | () => object} fields` - campos para mesclar

- **Uso**:
Estenda o `package.json` do projeto. Os campos aninhados são mesclados em profundidade, a menos que "{merge: false}" seja passado. Também resolve conflitos de dependência entre plugins. Os campos de configuração da ferramenta podem ser extraídos em arquivos independentes antes que os arquivos sejam gravados no disco.

## render

- **Argumentos**
  - `{string | object | FileMiddleware} source` - pode ser
    - um caminho relativo para um diretório;
    - um objeto hash de mapeamentos `{sourceTemplate: targetFile}`;
    - uma função de middleware de arquivo personalizado
  - `{object} [additionalData]` - dados adicionais disponíveis para modelos
  - `{object} [ejsOptions]` - opções para ejs

- **Uso**:
Renderiza arquivos de modelo no objeto da árvore de arquivos virtuais.

## postProcessFiles

- **Argumentos**
  - `{FileMiddleware} cb` - middleware de arquivo

- **Uso**:
Empurra um middleware de arquivo que será aplicado após todos os middlewares de arquivos normais terem sido aplicados.

## onCreateComplete

- **Argumentos**
  - `{function} cb`

- **Uso**:
Envia um retorno de chamada para ser chamado quando os arquivos tiverem sido gravados no disco.

## exitLog

- **Argumentos**
  - `{} msg` - string ou valor a ser impresso após a conclusão da geração;
  - `{('log' | 'info' | 'done' | 'warn' | 'error')} [type = 'log']` - tipo da mensagem.

- **Uso**:
Adiciona uma mensagem a ser impressa quando o gerador sair (depois de qualquer outra mensagem padrão).

## genJSConfig

- **Argumentos**
  - `{any} value`

- **Uso**:
Método de conveniência para gerar um arquivo de configuração JS de JSON

## injectImports

- **Argumentos**
  - `{string} file` - arquivo de destino para adicionar importações
  - `{string | [string]} imports` - importa de string/array

- **Uso**:
Adiciona instruções de importação a um arquivo.

## injectRootOptions

- **Argumentos**
  - `{string} file` - arquivo de destino para adicionar opções
  - `{string | [string]} options` - opções string/array

- **Uso**:
Adiciona opções à instância raiz do Vue (detectada pelo `new Vue`).

## entryFile

- **Retorna**
  - `{('src/main.ts' | 'src/main.js')}`

- **Uso**:
Obtenha o arquivo de entrada levando em conta o typescript.

## invocando

- **Retorna**
  - `{boolean}`

- **Uso**:
Verifica se o plug-in está sendo chamado.