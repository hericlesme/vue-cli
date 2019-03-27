---
sidebar: auto
---

# Referência de configuração

<Bit/>

## Configuração global da CLI

Algumas configurações globais para o `@vue/cli`, como seu gerenciador de pacotes preferido e suas predefinições salvas localmente, são armazenadas em um arquivo JSON chamado `.vuerc` em seu diretório inicial. Você pode editar este arquivo diretamente com o editor de sua escolha para alterar as opções salvas.

Você também pode usar o comando `vue config` para inspecionar ou modificar a configuração global da CLI.

## Navegadores de Destino

Veja a seção [Compatibilidade do Navegador](../guide/browser-compatibility.md#browserslist) no guia.

## vue.config.js

`vue.config.js` é um arquivo de configuração opcional que será automaticamente carregado pelo `@vue/cli-service` se estiver presente na raiz do seu projeto (próximo a `package.json`). Você também pode usar o campo `vue` em `package.json`, mas observe que, nesse caso, você estará limitado apenas a valores compatíveis com JSON.

O arquivo deve exportar um objeto contendo opções:

``` js
// vue.config.js
module.exports = {
  // opções...
}
```

### baseUrl

Depreciado desde o Vue CLI 3.3, por favor use [`publicPath`](#publicPath).

### publicPath

- Tipo: `string`
- Padrão: `'/'`

  O URL base do seu pacote de aplicativos será implementado em (conhecido como `baseUrl` antes do Vue CLI 3.3). Este é o equivalente do `output.publicPath` do webpack, mas o Vue CLI também precisa deste valor para outros propósitos, então você deve **sempre usar `publicPath` ao invés de modificar o pacote de saída `output.publicPath`**.

  Por padrão, a Vue CLI assume que seu aplicativo será implantado na raiz de um domínio, por exemplo, `https://www.my-app.com/`. Se seu aplicativo for implantado em um subcaminho, você precisará especificar esse subcaminho usando essa opção. Por exemplo, se seu aplicativo for implementado em `https://www.foobar.com/my-app/`, defina `publicPath` como `'/ my-app/'`.

  O valor também pode ser definido para uma string vazia (`''`) ou um caminho relativo (`./`) Para que todos os ativos sejam vinculados usando caminhos relativos. Isso permite que o pacote incorporado seja implantado em qualquer caminho público ou usado em um ambiente baseado em sistema de arquivos, como um aplicativo híbrido Cordova.

  ::: warning Limitações do publicPath relativo
  O publicPath relativo tem algumas limitações e deve ser evitado     quando:

  - Você está usando o roteamento `history.pushState` do HTML5;

  - Você está usando a opção `pages` para construir um aplicativo     multi-paginado.
  :::

  Este valor também é respeitado durante o desenvolvimento. Se        você quiser que seu servidor dev seja servido na raiz, você         pode usar um valor condicional:

  ``` js
  module.exports = {
    publicPath: process.env.NODE_ENV === 'production'
      ? '/production-sub-path/'
      : '/'
  }
  ```

### outputDir

- Tipo: `string`
- Padrão: `'dist'`

  O diretório onde os arquivos de compilação de produção serão gerados ao executar o `vue-cli-service build`. Observe que o diretório de destino será removido antes da construção (esse comportamento pode ser desabilitado passando `--no-clean` ao construir).

  ::: tip
  Sempre use `outputDir` em vez de modificar o webpack `output.path`.
  :::

### assetsDir

- Tipo: `string`
- Padrão: `''`

  Um diretório (relativo a `outputDir`) para aninhar recursos estáticos gerados (js, css, img, fonts).
  
  ::: tip
  `assetsDir` é ignorado ao sobrescrever o nome do arquivo ou o chunkFilename dos recursos gerados.
  :::

### indexPath

- Tipo: `string`
- Padrão: `'index.html'`

  Especifique o caminho de saída para o `index.html` gerado (relativo ao `outputDir`). Também pode ser um caminho absoluto.

### filenameHashing

- Tipo: `boolean`
- Padrão: `true`

  Por padrão, os ativos estáticos gerados contêm hashes em seus nomes de arquivos para melhor controle de armazenamento em cache. No entanto, isso requer que o HTML de índice seja gerado automaticamente pela Vue CLI. Se você não puder usar o HTML de índice gerado pelo Vue CLI, poderá desabilitar o hash de nome de arquivo definindo esta opção como `false`.

### pages

- Tipo: `Object`
- Padrão: `undefined`

  Construa o aplicativo com múltiplas páginas. Cada "página" deve ter um arquivo de entrada JavaScript correspondente. O valor deve ser um objeto em que a chave é o nome da entrada e o valor é:

  - Um objeto que especifica seu `entry`,` template`, `filename`,` title` e `chunks` (todos opcionais, exceto` entry`). Quaisquer outras propriedades adicionadas ao lado também serão passadas diretamente para o `html-webpack-plugin`, permitindo ao usuário personalizar o referido plugin;
  
  - Ou uma string especificando sua `entrada`.

  ``` js
  module.exports = {
    pages: {
      index: {
        // entrada para a página
        entry: 'src/index/main.js',
        // o template de origem
        template: 'public/index.html',
        // saída como dist/index.html
        filename: 'index.html',
        // ao usar a opção de título,
        // a tag de título do template precisa ser <title><%=htmlWebpackPlugin.options.title%></title>
        title: 'Index Page',
        // chunks para incluir nessa página, por padrão, inclui
        // extracted common chunks e vendor chunks.
        chunks: ['chunk-vendors', 'chunk-common', 'index']
      },
        // ao usar o formato de string somente de entrada,
        // template é inferido como `public/subpage.html`
        // e volta para `public/index.html` se não for encontrado.
        // O nome do arquivo de saída é inferido como `subpage.html`.
      subpage: 'src/subpage/main.js'
    }
  }
  ```

  ::: tip
  Ao construir no modo de múltiplas páginas, a configuração do webpack conterá plugins diferentes (haverá várias instâncias do  `plugin-html-webpack` e `plugin-webpack do pré-pacote`).   Certifique-se de executar o `vue inspect` se você está tentando  modificar as opções para esses plugins.
  :::

### lintOnSave

- Tipo: `boolean | 'error'`
- Padrão: `true`

  O lint-on-save deve ser executado durante o desenvolvimento usando [eslint-loader](https://github.com/webpack-contrib/eslint-loader). Este valor é respeitado apenas quando o [`@vue/cli-plugin-eslint`](https://github.com/vuejs/vue-cli/tree/dev/packages/%40vue/cli-plugin-eslint) estiver instalado .

  Quando definido como `true`, o `eslint-loader` emitirá erros de lint como avisos. Por padrão, os avisos são registrados apenas no terminal e não falham na compilação.

  Para que os erros de lint apareçam na sobreposição do navegador,  você pode usar o `lintOnSave: 'error'`. Isso forçará o `eslint-loader` a sempre emitir erros. Isso também significa que os erros de lint agora farão com que a compilação falhe.

  Como alternativa, você pode configurar a sobreposição para exibir avisos e erros:

  ``` js
  // vue.config.js
  module.exports = {
    devServer: {
      overlay: {
        warnings: true,
        errors: true
      }
    }
  }
  ```

  Quando `lintOnSave` é um valor verdadeiro, o `eslint-loader` será aplicado tanto no desenvolvimento quanto na produção. Se você quiser desabilitar o `eslint-loader` durante a compilação de produção, você pode usar a seguinte configuração:

  ``` js
  // vue.config.js
  module.exports = {
    lintOnSave: process.env.NODE_ENV !== 'production'
  }
  ```

### runtimeCompiler

- Tipo: `boolean`
- Padrão: `false`

  Se deve usar a compilação do núcleo Vue que inclui o compilador de tempo de execução. A configuração para 'true' permitirá que você use a opção `template` nos componentes do Vue, mas incorrerá em torno de uma carga extra de 10kb para seu aplicativo.

  Veja também: [Runtime + Compiler vs. Runtime only](https://vuejs.org/v2/guide/installation.html#Runtime-Compiler-vs-Runtime-only).

### transpileDependencies

- Tipo: `Array<string | RegExp>`
- Padrão: `[]`

  Por padrão o `babel-loader` ignora todos os arquivos dentro de` node_modules`. Se você quiser explicitamente transpilar uma dependência com o Babel, poderá listá-lo nessa opção.

::: warning Jest config
Esta opção não é respeitada pelo [plugin cli-unit-jest](#jest), porque em jest, nós não temos que transpilar o código de `/node_modules` a menos que ele use recursos não-padrão - o Node > 8.11 já suporta o mais recentes funcionalidades ECMAScript.

No entanto, jest às vezes tem que transformar o conteúdo de node_modules se esse código usar a sintaxe ES6 `import`/`export`. Nesse caso, use a opção `tranformIgnorePatterns` em `jest.config.js`.

See [o README do plugin](https://github.com/vuejs/vue-cli/blob/dev/packages/%40vue/cli-plugin-unit-jest/README.md) para mais informações.
:::

### productionSourceMap

- Tipo: `boolean`
- Padrão: `true`

  Definir isso como 'false' pode acelerar as construções de produção se você não precisar de mapas de origem para produção.

### crossorigin

- Tipo: `string`
- Padrão: `undefined`

  Configure o atributo `crossorigin` nas tags `<link rel="stylesheet">` e `<script>` no HTML gerado.

  Note que isto afeta somente as tags injetadas pelo `html-webpack-plugin` - as tags adicionadas diretamente no template fonte (`public/index.html`) não são afetadas.

  Veja também: [Atributos de configurações do CORS](https://developer.mozilla.org/en-US/docs/Web/HTML/CORS_settings_attributes)

### integrity

- Tipo: `boolean`
- Padrão: `false`

  Defina como `true` para ativar [Subresource Integrity](https://developer.mozilla.org/en-US/docs/Web/Security/Subresource_Integrity) (SRI) em `<link rel="stylesheet">` e `Tags <script>` no HTML gerado. Se você estiver hospedando seus arquivos construídos em um CDN, é uma boa ideia ativá-lo para segurança adicional.

  Note que isto afeta somente os tags injetados pelo `html-webpack-plugin` - as tags adicionadas diretamente no template fonte (`public/index.html`) não são afetadas.

  Além disso, quando o SRI está habilitado, as dicas de recurso de pré-carregamento são desativadas devido a um [bug no Chrome](https://bugs.chromium.org/p/chromium/issues/detail?id=677022) que faz com que os recursos sejam baixados duas vezes.

### configureWebpack

- Tipo: `Object | Function`

  Se o valor for um Object, ele será mesclado na configuração final usando [webpack-merge](https://github.com/survivejs/webpack-merge).

  Se o valor for uma função, ele receberá a configuração resolvida como o argumento. A função pode alterar a configuração e retornar nada OU retornar uma versão clonada ou mesclada da configuração.

  Veja também: [Trabalhando com Webpack > Configuração Simples](../guide/webpack.md#configuracao-simples)

### chainWebpack

- Tipo: `Function`

  Uma função que receberá uma instância de `ChainableConfig` ativada por [webpack-chain](https://github.com/mozilla-neutrino/webpack-chain). Permite uma modificação mais detalhada da configuração interna do webpack.

  Veja também: [Trabalhando Webpack > Chaining](../guide/webpack.md#chaining-avancado)

### css.modules

- Tipo: `boolean`
- Padrão: `false`

  Por padrão, somente os arquivos que terminam com `*.module.[ext]` são tratados como módulos CSS. Configurar isso para 'true' permitirá que você remova `.module` nos nomes dos arquivos e trate todos os arquivos `*.(css|scss|sass|less|styl(us)?)` como módulos CSS.

  Veja Também: [Trabalhando CSS > Módulos CSS](../guide/css.md#modulos-css)

### css.extract

- Tipo: `boolean | Object`
- Padrão: `true` em producão, `false` em desenvolvimento

  Se extrai CSS em seus componentes em arquivos CSS autônomos (em vez de embutidos em JavaScript e injetados dinamicamente).

  Isso sempre é desativado ao compilar como componentes da web (os estilos são embutidos e injetados no shadowRoot).

  Ao construir como uma biblioteca, você também pode definir isso como "false" para evitar que seus usuários precisem importar o CSS.

  A extração do CSS é desativada por padrão no modo de desenvolvimento, pois é incompatível com o recarregamento a quente do CSS. No entanto, você ainda pode impor a extração em todos os casos definindo explicitamente o valor como 'true'.

### css.sourceMap

- Tipo: `boolean`
- Padrão: `false`

  Se deseja ativar os mapas de origem para CSS. Definir isso como `true` pode afetar o desempenho de criação.

### css.loaderOptions

- Tipo: `Object`
- Padrão: `{}`

  Passa opções para carregadores relacionados ao CSS. Por exemplo:

  ``` js
  module.exports = {
    css: {
      loaderOptions: {
        css: {
          // opções aqui serão passadas para o css-loader
        },
        postcss: {
          // opções aqui serão passadas para o postcss-loader
        }
      }
    }
  }
  ```

  Loaders suportados:

  - [css-loader](https://github.com/webpack-contrib/css-loader)
  - [postcss-loader](https://github.com/postcss/postcss-loader)
  - [sass-loader](https://github.com/webpack-contrib/sass-loader)
  - [less-loader](https://github.com/webpack-contrib/less-loader)
  - [stylus-loader](https://github.com/shama/stylus-loader)

  Veja Também: [Opções de passagem para carregadores de pré-processador](../guide/css.md#opcoes-de-passagem-para-carregadores-de-pre-processador)

  ::: tip
  Isso é preferível ao tocar manualmente em carregadores específicos usando o `chainWebpack`, porque essas opções precisam ser aplicadas em vários locais onde o carregador correspondente é usado.
  :::

### devServer

- Tipo: `Object`

  [Todas as opções para `webpack-dev-server`](https://webpack.js.org/configuration/dev-server/) são suportadas. Note que:

  - Alguns valores como `host`, `port` e `https` podem ser sobrescritos por sinalizadores de linha de comando.

  - Alguns valores como `publicPath` e `historyApiFallback` não devem ser modificados, pois precisam ser sincronizados com [publicPath](#baseurl) para que o servidor dev funcione corretamente.

### devServer.proxy

- Tipo: `string | Object`

  Se o aplicativo de front-end e o servidor de API de back-end não estiverem em execução no mesmo host, você precisará fazer proxy nas solicitações de API para o servidor de API durante o desenvolvimento. Isto é configurável através da opção `devServer.proxy` no `vue.config.js`.

  `devServer.proxy` pode ser uma string apontando para o servidor da API de desenvolvimento:
  
  ``` js
  module.exports = {
    devServer: {
      proxy: 'http://localhost:4000'
    }
  }
  ```

  Isso dirá ao servidor dev para fazer proxy de quaisquer solicitações desconhecidas (solicitações que não correspondem a um arquivo estático) para `http://localhost:4000`.

  Se você quer ter mais controle sobre o comportamento do proxy, você também pode usar um objeto com pares `path: options`. Consulte [http-proxy-middleware](https://github.com/chimurai/http-proxy-middleware#proxycontext-config) para opções completas:

  ``` js
  module.exports = {
    devServer: {
      proxy: {
        '^/api': {
          target: '<url>',
          ws: true,
          changeOrigin: true
        },
        '^/foo': {
          target: '<other_url>'
        }
      }
    }
  }
  ```

### parallel

- Tipo: `boolean`
- Padrão: `require('os').cpus().length > 1`

  Se usar o `thread-loader` para a transpilação de Babel ou TypeScript. Isso é ativado para construções de produção quando o sistema tem mais de um núcleo de CPU.

### pwa

- Tipo: `Object`

  Passa as opções para o [PWA Plugin](https://github.com/vuejs/vue-cli/tree/dev/packages/%40vue/cli-plugin-pwa).

### pluginOptions

- Tipo: `Object`

  Este é um objeto que não passa por nenhuma validação de esquema, por isso pode ser usado para passar opções arbitrárias para plugins de terceiros. Por exemplo:

  ``` js
  module.exports = {
    pluginOptions: {
      foo: {
        // plugins podem acessar essas opções como
        // `options.pluginOptions.foo`.
      }
    }
  }
  ```

## Babel

O Babel pode ser configurado via `babel.config.js`.

::: tip
O Vue CLI usa o `babel.config.js` que é um novo formato de configuração no Babel 7. Ao contrário do `babelrc` ou do campo `babel` no `package.json`, este arquivo de configuração não usa uma resolução baseada no local do arquivo, e é aplicado consistentemente a qualquer arquivo sob o projeto raiz, incluindo dependências dentro de `node_modules`. Recomenda-se sempre usar o `babel.config.js` em vez de outros formatos nos projetos do Vue CLI.
:::

Todos os aplicativos Vue CLI usam `@ vue/babel-preset-app`, que inclui `babel-preset-env`, suporte a JSX e configuração otimizada para sobrecarga mínima de tamanho de pacote. Consulte [sua documentação](https://github.com/vuejs/vue-cli/tree/dev/packages/%40vue/babel-preset-app) para obter detalhes e opções predefinidas.

Veja também a seção [Polyfills](../guide/guide-compatibility.md#polyfills) no guia.

## ESLint

O ESLint pode ser configurado através do campo `.eslintrc` ou `eslintConfig` em `package.json`.

Veja [@vue/cli-plugin-eslint](https://github.com/vuejs/vue-cli/tree/dev/packages/%40vue/cli-plugin-eslint) para mais detalhes.

## TypeScript

TypeScript pode ser configurado via `tsconfig.json`.

Veja [@vue/cli-plugin-typescript](https://github.com/vuejs/vue-cli/tree/dev/packages/%40vue/cli-plugin-typescript) para mais detalhes.

## Unit Testing

### Jest

Veja [@vue/cli-plugin-unit-jest](https://github.com/vuejs/vue-cli/tree/dev/packages/%40vue/cli-plugin-unit-jest) para mais detalhes.

### Mocha (via `mocha-webpack`)

Veja [@vue/cli-plugin-unit-mocha](https://github.com/vuejs/vue-cli/tree/dev/packages/%40vue/cli-plugin-unit-mocha) para mais detalhes.

## E2E Testing

### Cypress

Veja [@vue/cli-plugin-e2e-cypress](https://github.com/vuejs/vue-cli/tree/dev/packages/%40vue/cli-plugin-e2e-cypress) para mais detalhes.

### Nightwatch

Veja [@vue/cli-plugin-e2e-nightwatch](https://github.com/vuejs/vue-cli/tree/dev/packages/%40vue/cli-plugin-e2e-nightwatch) para mais detalhes.