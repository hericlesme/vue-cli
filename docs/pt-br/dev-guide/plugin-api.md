# Plugin API

## getCwd

- **Uso**:
Retorna um diretório de trabalho atual

## resolve

- **Argumentos**
  - `{string} path` - caminho relativo da raiz do projeto

- **Retorna**
  - `{string}` - o caminho absoluto resolvido

- **Uso**:
Resolver um caminho para o projeto atual

## hasPlugin

- **Argumentos**
  - `{string} id` - id do plugin, pode omitir o prefixo (@vue/vue-|@scope/vue)-cli-plugin-

- **Retorna**
  - `{boolean}`

- **Uso**:
Verifica se o projeto tem um plugin com o ID fornecido

## registerCommand

- **Argumentos**
  - `{string} name`
  - `{object} [opts]`
  ```js
  {
  description: string,
  usage: string,
  options: { [string]: string }
  }
  ```
  - `{function} fn`
  ```js
  (args: { [string]: string }, rawArgs: string[]) => ?Promise
  ```

- **Uso**:
Registre um comando que ficará disponível como `vue-cli-service [name]`.

## chainWebpack

- **Argumentos**
  - `{function} fn`

- **Uso**:
Registre uma função que receberá uma configuração de webpack em cadeia. Esta função é preguiçosa e não será chamada até que o `resolveWebpackConfig` seja chamado.


## configureWebpack

- **Argumentos**
  - `{object | function} fn`

- **Uso**:
Registra um objeto de configuração do webpack que será mesclado na configuração **OU** uma função que receberá a configuração bruta do webpack. A função pode alterar a configuração diretamente ou retornar um objeto
que será mesclado na configuração do webpack.

## configureDevServer

- **Argumentos**
  - `{object | function} fn`

- **Uso**:
Registra uma função de configuração do dev serve. Ele irá receber a instância expressa do `app` do servidor dev.

## resolveWebpackConfig

- **Argumentos**
  - `{ChainableWebpackConfig} [chainableConfig]`
- **Retorna**
  - `{object}` - config raw do webpack

- **Uso**:
Resolva a configuração final do webpack, que será passada para o webpack.

## resolveChainableWebpackConfig

- **Retorna**
  - `{ChainableWebpackConfig}`

- **Uso**:
Define uma instância intermediária de configuração do webpack, que pode ser mais ajustada antes de gerar a configuração final do webpack bruto. Você pode chamar isso várias vezes para gerar diferentes ramificações da configuração base do webpack.

Veja [https://github.com/mozilla-neutrino/webpack-chain](https://github.com/mozilla-neutrino/webpack-chain)

## genCacheConfig

- **Argumentos**
  - `id`
  - `partialIdentifier`
  - `configFiles`
- **Retorna**
  - `{object}`
  ```js
  {
  cacheDirectory: string,
  cacheIdentifier: string }
  ```
- **Uso**:
Gere um identificador de cache a partir de um número de variáveis.