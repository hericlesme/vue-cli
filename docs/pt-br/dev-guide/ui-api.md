# UI API

O cli-ui possui uma API que permite aumentar as configurações e tarefas do projeto, além de compartilhar dados e se comunicar com outros processos.

![UI - Arquitetura de Plugin](/vue-cli-ui-schema.png)

## UI files

Dentro de cada plug-in vue-cli instalado, o cli-ui tentará carregar um arquivo `ui.js` opcional na pasta raiz do plugin. Note que você também pode usar pastas (por exemplo, `ui/index.js`).

O arquivo deve exportar uma função que obtenha o objeto api como argumento:

```js
module.exports = api => {
  // Use a API aqui...
}
```

**⚠️ Os arquivos serão recarregados quando buscar a lista de plugins na visualização 'Plugins de Projeto'. Para aplicar as alterações, clique no botão 'Plugins de projeto' na barra lateral de navegação à esquerda na interface do usuário.**

Aqui está um exemplo de estrutura de pastas para um plugin vue-cli usando a API de interface do usuário:

```
- vue-cli-plugin-test
  - package.json
  - index.js
  - generator.js
  - prompts.js
  - ui.js
  - logo.png
```

### Plugins locais do projeto

Se você precisa acessar a API do plugin em seu projeto e não quer criar um plugin completo para isso, você pode usar a opção `vuePlugins.ui` no seu arquivo `package.json`:

```json
{
  "vuePlugins": {
    "ui": ["my-ui.js"]
  }
}
```

Cada arquivo precisará exportar uma função usando a API do plug-in como o primeiro argumento.

## Modo de desenvolvimento

Ao construir seu plugin, você pode querer rodar o cli-ui no modo Dev, então ele irá gerar logs úteis para você:

```
vue ui --dev
```

Ou:

```
vue ui -D
```

## Configurações do projeto

![Configuração - UI](/config-ui.png)

Você pode adicionar uma configuração de projeto com o método `api.describeConfig`.

Primeiro você precisa passar algumas informações:

```js
api.describeConfig({
  // ID exclusivo pra a configuração
  id: 'org.vue.eslintrc',
  // Nome de exibição
  name: 'ESLint configuration',
  // Exibido abaixo do nome
  description: 'Error checking & Code quality',
  // Link para mais informações
  link: 'https://eslint.org'
})
```

::: danger
Certifique-se de namespace o id corretamente, uma vez que deve ser exclusivo em todos os plugins. Recomenda-se usar a [notação de nome de domínio reverso](https://en.wikipedia.org/wiki/Reverse_domain_name_notation).
:::

### Ícone de configuração

Pode ser um código [Material icon](https://material.io/tools/icons) ou uma imagem personalizada (consulte [Arquivos estáticos públicos](#arquivos-estaticos-publicos)):

```js
api.describeConfig({
  /* ... */
  // Ícone de configuração
  icon: 'application_settings'
})
```

Se você não especificar um ícone, o logotipo do plug-in será exibido, se houver algum (consulte [Logo](./ui-info.md#logo)).

### Arquivos de configuração

Por padrão, uma UI de configuração pode ler e gravar em um ou mais arquivos de configuração, por exemplo, `.eslintrc.js` e `vue.config.js`.

Você pode fornecer quais são os arquivos possíveis a serem detectados no projeto do usuário:

```js
api.describeConfig({
  /* ... */
  // Todos os arquivos possíveis para esta configuração
  files: {
    // eslintrc.js
    eslint: {
      js: ['.eslintrc.js'],
      json: ['.eslintrc', '.eslintrc.json'],
      // Vai ler de `package.json`
      package: 'eslintConfig'
    },
    // vue.config.js
    vue: {
      js: ['vue.config.js']
    }
  },
})
```

Tipos de arquivos suportados: `json`, `yaml`, `js`, `package`. A ordem é importante: o primeiro nome de arquivo na lista será usado para criar o arquivo de configuração, se ele não existir.

### Exibe prompts de configuração

Use o gancho `onRead` para retornar uma lista de prompts a serem exibidos para a configuração:

```js
api.describeConfig({
  /* ... */
  onRead: ({ data, cwd }) => ({
    prompts: [
      // Prompt objects
    ]
  })
})
```

Esses prompts do serão exibidos no painel de detalhes da configuração.

Veja [Prompts](#prompts) para mais informações.

O objeto `data` contém o resultado JSON de cada conteúdo do arquivo de configuração.

Por exemplo, digamos que o usuário tenha o seguinte `vue.config.js` em seu projeto:

```js
module.exports = {
  lintOnSave: false
}
```

Nós declaramos o arquivo de configuração em nosso plugin como este:

```js
api.describeConfig({
  /* ... */
  // Todos os arquivos possíveis para esta configuração
  files: {
    // vue.config.js
    vue: {
      js: ['vue.config.js']
    }
  },
})
```

Então o objeto `data` será:

```js
{
  // Arquivo
  vue: {
    // Dados do arquivo
    lintOnSave: false
  }
}
```

Exemplo de vários arquivos: se adicionarmos o seguinte arquivo `eslintrc.js` no projeto do usuário:

```js
module.exports = {
  root: true,
  extends: [
    'plugin:vue/essential',
    '@vue/standard'
  ]
}
```

E mude a opção `files` no nosso plugin para isto:

```js
api.describeConfig({
  /* ... */
  // Todos os arquivos possíveis para esta configuração
  files: {
    // eslintrc.js
    eslint: {
      js: ['.eslintrc.js'],
      json: ['.eslintrc', '.eslintrc.json'],
      // Vai ler de `package.json`
      package: 'eslintConfig'
    },
    // vue.config.js
    vue: {
      js: ['vue.config.js']
    }
  },
})
```

Então o objeto `data` será:

```js
{
  eslint: {
    root: true,
    extends: [
      'plugin:vue/essential',
      '@vue/standard'
    ]
  },
  vue: {
    lintOnSave: false
  }
}
```

### Configuration tabs

Você pode organizar os prompts em várias guias:

```js
api.describeConfig({
  /* ... */
  onRead: ({ data, cwd }) => ({
    tabs: [
      {
        id: 'tab1',
        label: 'My tab',
        // Opcional
        icon: 'application_settings',
        prompts: [
          // Objetos de Prompt
        ]
      },
      {
        id: 'tab2',
        label: 'My other tab',
        prompts: [
          // Objetos de Prompt
        ]
      }
    ]
  })
})
```

### Salvar alterações de configuração

Use o gancho `onWrite` para gravar os dados no arquivo de configuração (ou executar qualquer código nodejs):

```js
api.describeConfig({
  /* ... */
  onWrite: ({ prompts, answers, data, files, cwd, api }) => {
    // ...
  }
})
```

Argumentos:

- `prompts`: objetos de prompts atuais em tempo de execução (veja abaixo)
- `answers`: responde aos dados das entradas do usuário
- `data`: dados iniciais somente leitura lidos a partir dos arquivos de configuração
- `files`: descritores dos arquivos encontrados (`{type: 'json', path: '...'}`)
- `cwd`: diretório de trabalho atual
- `api`: `onWrite API` (veja abaixo)

Solicita objetos de tempo de execução:

```js
{
  id: data.name,
  type: data.type,
  name: data.short || null,
  message: data.message,
  group: data.group || null,
  description: data.description || null,
  link: data.link || null,
  choices: null,
  visible: true,
  enabled: true,
  // Valor atual (não filtrado)
  value: null,
  // true se alterado pelo usuário
  valueChanged: false,
  error: null,
  tabId: null,
  // Objeto do prompt de pergunta original
  raw: data
}
```

`onWrite` API:

- `assignData (fileId, newData)`: use `Object.assign` para atualizar os dados de configuração antes de gravar.
- `setData (fileId, newData)`: cada chave de `newData` será profundamente ajustada (ou removida se o valor` undefined`) para os dados de configuração antes da escrita.
- `async getAnswer (id, mapper)`: recupera a resposta para uma dada id de prompt e a mapeia através da função `mapper` se fornecida (por exemplo,` JSON.parse`).

Exemplo (do plugin ESLint):

```js
api.describeConfig({
  // ...

  onWrite: async ({ api, prompts }) => {
    // Atualizar regras do ESLint
    const result = {}
    for (const prompt of prompts) {
      result[`rules.${prompt.id}`] = await api.getAnswer(prompt.id, JSON.parse)
    }
    api.setData('eslint', result)
  }
})
```

## Tarefas do projeto

![UI - Tarefas](/tasks-ui.png)

As tarefas são geradas a partir do campo `scripts` no arquivo `package.json` do projeto.

Você pode 'aumentar' as tarefas com informações adicionais e ganchos graças ao método `api.describeTask`:

```js
api.describeTask({
  // RegExp executado em comandos de script para selecionar qual tarefa será descrita aqui
  match: /vue-cli-service serve/,
  description: 'Compiles and hot-reloads for development',
  // Link para mais informações
  link: 'https://github.com/vuejs/vue-cli/blob/dev/docs/cli-service.md#serve'
})
```

Você também pode usar uma função para `match`:

```js
api.describeTask({
  match: (command) => /vue-cli-service serve/.test(command),
})
```

### Ícone da tarefa

Pode ser um código [Material icon](https://material.io/tools/icons) ou uma imagem personalizada (consulte [Arquivos estáticos públicos](#arquivos-estaticos-publicos)):

```js
api.describeTask({
  /* ... */
  // Task icon
  icon: 'application_settings'
})
```

Se você não especificar um ícone, o logotipo do plug-in será exibido, se houver algum (consulte [Logo](./ui-info.md#logo)).

### Parâmetros de tarefas

Você pode adicionar prompts para modificar os argumentos do comando. Eles serão exibidos em um modal de 'Parâmetros'.

Exemplo:

```js
api.describeTask({
  // ...

  // Parâmetros opcionais (solicitações do prompt)
  prompts: [
    {
      name: 'open',
      type: 'confirm',
      default: false,
      description: 'Open browser on server start'
    },
    {
      name: 'mode',
      type: 'list',
      default: 'development',
      choices: [
        {
          name: 'development',
          value: 'development'
        },
        {
          name: 'production',
          value: 'production'
        },
        {
          name: 'test',
          value: 'test'
        }
      ],
      description: 'Specify env mode'
    }
  ]
})
```

Consulte [Prompts](#prompts) para mais informações.

### Ganchos de tarefas

Vários ganchos estão disponíveis:

- `onBeforeRun`
- `onRun`
- `onExit`

Por exemplo, você pode usar as respostas para os prompts (veja acima) para adicionar novos argumentos ao comando:

```js
api.describeTask({
  // ...

  // Hooks
  // Modifique os argumentos aqui
  onBeforeRun: async ({ answers, args }) => {
    // Args
    if (answers.open) args.push('--open')
    if (answers.mode) args.push('--mode', answers.mode)
    args.push('--dashboard')
  },
  // Imediatamente após executar a tarefa
  onRun: async ({ args, child, cwd }) => {
    // child: node child process
    // cwd: process working directory
  },
  onExit: async ({ args, child, cwd, code, signal }) => {
    // code: exit code
    // signal: kill signal used if any
  }
})
```

### Exibições de tarefas

Você pode exibir exibições personalizadas no painel de detalhes da tarefa usando a API `ClientAddon`:

```js
api.describeTask({
  // ...

  // Visualizações adicionais (por exemplo, o painel do webpack)
  // Por padrão, há a exibição 'output' que exibe a saída do terminal
  views: [
    {
      // ID Exclusivo
      id: 'vue-webpack-dashboard-client-addon',
      // Etiqueta do botão (label)
      label: 'Dashboard',
      // Ícone do botão
      icon: 'dashboard',
      // Componente dinâmico para carregar (veja a seção 'Client addon' abaixo)
      component: 'vue-webpack-dashboard'
    }
  ],
  // Visualização padrão selecionada ao exibir os detalhes da tarefa (por padrão, é a saída)
  defaultView: 'vue-webpack-dashboard-client-addon'
})
```

Consulte [Client addon](#client-addon) para mais informações.


### Adicionar novas tarefas

Você também pode adicionar tarefas inteiramente novas que não estão nos scripts `package.json` com `api.addTask` em vez de `api.describeTask`. Essas tarefas só aparecerão na interface do usuário.

**Você precisa fornecer uma opção `command` ao invés de` match`.**

Exemplo:

```js
api.addTask({
  // Obrigatório
  name: 'inspect',
  command: 'vue-cli-service inspect',
  // Opcional
  // O resto é como `describeTask` sem a opção` match`
  description: '...',
  link: 'https://github.com/vuejs/vue-cli/...',
  prompts: [ /* ... */ ],
  onBeforeRun: () => {},
  onRun: () => {},
  onExit: () => {},
  views: [ /* ... */ ],
  defaultView: '...'
})
```

**⚠️ O `comando` executará um contexto node. Isso significa que você pode chamar os comandos node executáveis como você normalmente faria nos scripts `package.json`.**

## Prompts

Os objetos de prompt devem ser objetos [inquirer](https://github.com/SBoudrias/Inquirer.js) válidos.

No entanto, você pode adicionar os seguintes campos adicionais (que são opcionais e usados apenas pela interface do usuário):

```js
{
  /* ... */
  // Usado para agrupar prompts em seções
  group: 'Strongly recommended',
  // Descrição Adicional
  description: 'Enforce attribute naming style in template (`my-prop` or `myProp`)',
  // Link para mais informações
  link: 'https://github.com/vuejs/eslint-plugin-vue/blob/master/docs/rules/attribute-hyphenation.md',
}
```

Tipos de questionários suportados: `checkbox`, `confirm`, `input`, `password`, `list`, `rawlist`.

Tipos de pergunta suportados Além desses, a interface do usuário suporta tipos especiais que só funcionam com ela:

- `color`: Exibe um seletor de cores.

### Exemplo de Switch

```js
{
  name: 'open',
  type: 'confirm',
  default: false,
  description: 'Open the app in the browser'
}
```

### Exemplo de Select

```js
{
  name: 'mode',
  type: 'list',
  default: 'development',
  choices: [
    {
      name: 'Development mode',
      value: 'development'
    },
    {
      name: 'Production mode',
      value: 'production'
    },
    {
      name: 'Test mode',
      value: 'test'
    }
  ],
  description: 'Build mode',
  link: 'https://link-to-docs'
}
```

### Exemplo de Input

```js
{
  name: 'host',
  type: 'input',
  default: '0.0.0.0',
  description: 'Host for the development server'
}
```

### Exemplo de Checkbox

Exibe vários switches.

```js
{
  name: 'lintOn',
  message: 'Pick additional lint features:',
  when: answers => answers.features.includes('linter'),
  type: 'checkbox',
  choices: [
    {
      name: 'Lint on save',
      value: 'save',
      checked: true
    },
    {
      name: 'Lint and fix on commit' + (hasGit() ? '' : chalk.red(' (requires Git)')),
      value: 'commit'
    }
  ]
}
```

### Exemplo de seletor de cores

```js
{
  name: 'themeColor',
  type: 'color',
  message: 'Theme color',
  description: 'This is used to change the system UI color around the app',
  default: '#4DBA87'
}
```

### Prompts para invocação

Em seu plugin vue-cli, você pode já ter um arquivo `prompts.js` que faz algumas perguntas ao usuário ao instalar o plugin (com a CLI ou a UI). Você também pode adicionar os campos adicionais da interface do usuário (veja acima) a esses objetos de prompt para que eles forneçam mais informações se o usuário estiver usando a interface do usuário.

**⚠️ Atualmente, os tipos de pergunta que não são suportados (veja acima) não funcionarão corretamente na interface do usuário.**

## Client addon

Um addon Client é um pacote JS que é carregado dinamicamente no cli-ui. É útil carregar componentes personalizados e rotas.

### Crie um client addon

A maneira recomendada de criar um addon Client é criar um novo projeto usando o vue cli. Você pode fazer isso em uma subpasta do seu plugin ou em um pacote npm diferente.

Instale o `@vue/cli-ui` como uma dependência dev.

Em seguida, adicione um arquivo `vue.config.js` com o seguinte conteúdo:

```js
const { clientAddonConfig } = require('@vue/cli-ui')

module.exports = {
  ...clientAddonConfig({
    id: 'org.vue.webpack.client-addon',
    // Porta de desenvolvimento (default 8042)
    port: 8042
  })
}
```

O método `clientAddonConfig` gerará a configuração necessária do vue-cli. Entre outras coisas, ele desativa a extração CSS e gera o código `./dist/index.js` na pasta addon do cliente.

::: danger
Certifique-se de namespace o id corretamente, uma vez que deve ser exclusivo em todos os plugins. Recomenda-se usar a [notação de nome de domínio reverso](https://en.wikipedia.org/wiki/Reverse_domain_name_notation).
:::

Em seguida, modifique o arquivo `.eslintrc.json` para adicionar alguns objetos globais permitidos:

```json
{
  // ...
  "globals": {
    "ClientAddonApi": false,
    "mapSharedData": false,
    "Vue": false
  }
}
```

Agora você pode executar o script `serve` em desenvolvimento e o `build` quando estiver pronto para publicar seu plugin.

### ClientAddonApi

Abra o arquivo `main.js` nas fontes do client addon e remova todo o código.

**⚠️ Não importe o Vue nas fontes addon do cliente, use o objeto global `Vue` da janela` do browser`.**

Aqui está um exemplo de código para `main.js`:

```js
import VueProgress from 'vue-progress-path'
import WebpackDashboard from './components/WebpackDashboard.vue'
import TestView from './components/TestView.vue'

// Você pode instalar plugins vue adicionais
// usando a variável global 'Vue'
Vue.use(VueProgress, {
  defaultShape: 'circle'
})

// Registrar um componente personalizado
// (funciona como 'Vue.component')
ClientAddonApi.component('org.vue.webpack.components.dashboard', WebpackDashboard)

// Adicione rotas ao vue-router sob uma rota pai /addon/<id>.
// Por exemplo, addRoutes ('foo', [{path: ''}, {path: 'bar'}])
// adicionará as rotas /addon/foo e /addon/foo/bar ao vue-router.
// Aqui criamos uma nova rota '/addon/vue-webpack/' com o nome 'test-webpack-route'
ClientAddonApi.addRoutes('org.vue.webpack', [
  { path: '', name: 'org.vue.webpack.routes.test', component: TestView }
])

// Você pode traduzir seus componentes de plugins
// Carrega os arquivos de local (usa vue-i18n)
const locales = require.context('./locales', true, /[a-z0-9]+\.json$/i)
locales.keys().forEach(key => {
  const locale = key.match(/([a-z0-9]+)\./i)[1]
  ClientAddonApi.addLocalization(locale, locales(key))
})
```

::: danger
Certifique-se de nomear os IDs corretamente, pois eles devem ser exclusivos em todos os plug-ins. É recomendado usar a [notação de nome de domínio reverso](https://en.wikipedia.org/wiki/Reverse_domain_name_notation).
:::

O cli-ui registra `Vue` e `ClientAddonApi` como variáveis globais no escopo `window`.

Em seus componentes, você pode usar todos os componentes e as classes CSS de [@vue/ui](https://github.com/vuejs/ui) e [@vue/cli-ui](https://github.com/vuejs/vue-cli/tree/dev/packages/%40vue/cli-ui/src/components)) para manter a aparência consistente. Você também pode traduzir as strings com [vue-i18n](https://github.com/kazupon/vue-i18n), que está incluído.

### Registre o client addon

De volta ao arquivo `ui.js`, use o método `api.addClientAddon` com uma consulta require para a pasta interna:

```js
api.addClientAddon({
  id: 'org.vue.webpack.client-addon',
  // Pasta contendo os arquivos JS criados
  path: '@vue/cli-ui-addon-webpack/dist'
})
```

Isso usará a API node_s `require.resolve` para localizar a pasta e servir o arquivo `index.js` construído a partir do addon do cliente.

Ou especifique uma URL ao desenvolver o plugin (de preferência você quer fazer isso no arquivo `vue-cli-ui.js` no seu projeto test vue):

```js
// Útil para dev
// Irá substituir o caminho se já estiver definido em um plugin
api.addClientAddon({
  id: 'org.vue.webpack.client-addon',
  // Use a mesma porta que você configurou anteriormente
  url: 'http://localhost:8042/index.js'
})
```

### Use o addon do cliente

Agora você pode usar o complemento do cliente nas visualizações. Por exemplo, você pode especificar uma exibição em uma tarefa descrita:

```js
api.describeTask({
  /* ... */
  // Visualizações adicionais (por exemplo, o painel do webpack)
  // Por padrão, há a exibição 'output' que exibe a saída do terminal
  views: [
    {
      // ID Exclusivo
      id: 'org.vue.webpack.views.dashboard',
      // Etiqueta do botão (label)
      label: 'Dashboard',
      // Ícone do botão (material-icons)
      icon: 'dashboard',
      // Componente dinâmico para carregar, registrado usando ClientAddonApi
      component: 'org.vue.webpack.components.dashboard'
    }
  ],
  // Visualização selecionada padrão ao exibir os detalhes da tarefa (por padrão, é a saída)
  defaultView: 'org.vue.webpack.views.dashboard'
})
```

Aqui está o código addon do cliente que registra o componente `'org.vue.webpack.components.dashboard'` (como vimos anteriormente):

```js
/* No `main.js` */
// Importe o componente
import WebpackDashboard from './components/WebpackDashboard.vue'
// Registra um componente personalizado
// (funciona como 'Vue.component')
ClientAddonApi.component('org.vue.webpack.components.dashboard', WebpackDashboard)
```

![Exemplo de visualização de tarefa](/task-view.png)

## Visualizações personalizadas

Você pode adicionar uma nova visão abaixo dos padrões 'Project plugins', 'Project configuration' e 'Project tasks' usando o método `api.addView`:

```js
api.addView({
  // ID exlusivo
  id: 'org.vue.webpack.views.test',

  // Nome da rota (do vue-router)
  // Use o mesmo nome usado no método 'ClientAddonApi.addRoutes' (veja acima na seção Client addon)
  name: 'org.vue.webpack.routes.test',

  // Ícone do botão (material-icons)
  icon: 'pets',
  // You can also specify a custom image (see Public static files section below):
  // icon: 'http://localhost:4000/_plugin/%40vue%2Fcli-service/webpack-icon.svg',

  // Sugestão do botão
  tooltip: 'Test view from webpack addon'
})
```

Aqui está o código no addon do cliente que registra o `'org.vue.webpack.routes.test'` (como vimos anteriormente):

```js
/* No `main.js` */
// Importe o componente
import TestView from './components/TestView.vue'
// Adicione rotas ao vue-router sob uma rota pai /addon/<id>.
// Por exemplo, addRoutes ('foo', [{path: ''}, {path: 'bar'}])
// adicionará as rotas /addon/foo e /addon/foo/bar ao vue-router.
// Aqui criamos uma nova rota '/addon/vue-webpack/' com o nome 'test-webpack-route'
ClientAddonApi.addRoutes('org.vue.webpack', [
  { path: '', name: 'org.vue.webpack.routes.test', component: TestView }
])
```

![Exemplo de visualização personalizada](/custom-view.png)

## Dados compartilhados

Use dados compartilhados para comunicar informações com componentes personalizados de maneira fácil.

Por exemplo, o painel do Webpack compartilha as estatísticas de compilação entre o cliente de UI e o servidor de UI usando esta API.

No plugin `ui.js` (nodejs):

```js
// Define ou atualiza
api.setSharedData('com.my-name.my-variable', 'some-data')

// Pega
const sharedData = api.getSharedData('com.my-name.my-variable')
if (sharedData) {
  console.log(sharedData.value)
}

// Remova
api.removeSharedData('com.my-name.my-variable')

// Observa alterações
const watcher = (value, id) => {
  console.log(value, id)
}
api.watchSharedData('com.my-name.my-variable', watcher)
// Para de observar alterações
api.unwatchSharedData('com.my-name.my-variable', watcher)

// Versões com namespaces
const {
  setSharedData,
  getSharedData,
  removeSharedData,
  watchSharedData,
  unwatchSharedData
} = api.namespace('com.my-name.')
```

::: danger
Certifique-se de nomear os IDs corretamente, pois eles devem ser exclusivos em todos os plug-ins. É recomendado usar a [notação de nome de domínio reverso](https://en.wikipedia.org/wiki/Reverse_domain_name_notation).
:::

In the custom component:

```js
// Componente Vue
export default {
  // Sincroniza dados compartilhados
  sharedData () {
    return {
      // Você pode usar `myVariable` no template
      myVariable: 'com.my-name.my-variable'
      // Você também pode mapear dados compartilhados com namespace
      ...mapSharedData('com.my-name.', {
        myVariable2: 'my-variable2'
      })
    }
  },

  // Métodos manuais
  async created () {
    const value = await this.$getSharedData('com.my-name.my-variable')

    this.$watchSharedData(`com.my-name.my-variable`, value => {
      console.log(value)
    })

    await this.$setSharedData('com.my-name.my-variable', 'new-value')
  }
}
```

Se você usar a opção `sharedData`, os dados compartilhados poderão ser atualizados atribuindo um novo valor à propriedade correspondente.

```html
<template>
  <VueInput v-model="message"/>
</template>

<script>
export default {
  sharedData: {
    // Will sync the 'my-message' shared data on the server
    message: 'com.my-name.my-message'
  }
}
</script>
```

Isso é muito útil se você criar um componente de configurações, por exemplo.

## Ações de plug-in

Ações de plug-in são chamadas enviadas entre o cli-ui (navegador) e plugins (nodejs).

> Por exemplo, você pode ter um botão em um componente personalizado (consulte [Client addon](#client-addon)), que chama algum código de nodejs no servidor usando esta API.

No arquivo `ui.js` no plugin (nodejs), você pode usar dois métodos do `PluginApi`:

```js
// Invoca uma ação
api.callAction('com.my-name.other-action', { foo: 'bar' }).then(results => {
  console.log(results)
}).catch(errors => {
  console.error(errors)
})
```

```js
// Espera por uma ação
api.onAction('com.my-name.test-action', params => {
  console.log('test-action called', params)
})
```

::: danger
Certifique-se de nomear os IDs corretamente, pois eles devem ser exclusivos em todos os plug-ins. É recomendado usar a [notação de nome de domínio reverso](https://en.wikipedia.org/wiki/Reverse_domain_name_notation).
:::

Você pode usar versões com namespaces com o `api.namespace` (semelhante ao Shared data):

```js
const { onAction, callAction } = api.namespace('com.my-name.')
```

Nos componentes addon do cliente (navegador), você tem acesso a `$onPluginActionCalled`, `$onPluginActionResolved` e `$callPluginAction`:

```js
// Componente Vue
export default {
  created () {
    this.$onPluginActionCalled(action => {
      // Quando a ação é chamada
      // antes de ser executada
      console.log('called', action)
    })
    this.$onPluginActionResolved(action => {
      // Depois que a ação é executada e concluída
      console.log('resolved', action)
    })
  },

  methods: {
    testPluginAction () {
      // Invoca uma ação de plugin
      this.$callPluginAction('com.my-name.test-action', {
        meow: 'meow'
      })
    }
  }
}
```

## Comunicação entre processos (IPC)

IPC significa Inter-Process Communication. Este sistema permite que você envie facilmente mensagens de processos filhos (por exemplo, tarefas!). E é muito rápido e leve.

> Para exibir os dados na UI do painel do webpack, os comandos `serve` e `build` do `@vue/cli-service` enviam mensagens IPC para o servidor cli-ui nodejs quando o argumento` --dashboard` é passado.

Em seu código de processo (que pode ser um plugin Webpack ou um script de tarefa nodejs), você pode usar a classe `IpcMessenger` de `@vue/cli-shared-utils`:

```js
const { IpcMessenger } = require('@vue/cli-shared-utils')

// Cria uma nova instância IpcMessenger
const ipc = new IpcMessenger()

function sendMessage (data) {
  // Envia uma mensagem para o servidor cli-ui
  ipc.send({
    'com.my-name.some-data': {
      type: 'build',
      value: data
    }
  })
}

function messageHandler (data) {
  console.log(data)
}

// Espera por mensagens
ipc.on(messageHandler)

// Para de esperar mensagens
ipc.off(messageHandler)

function cleanup () {
  // Desconecta da rede IPC
  ipc.disconnect()
}
```

Manual connection:

```js
const ipc = new IpcMessenger({
  autoConnect: false
})

// Esta mensagem será enfileirada
ipc.send({ ... })

ipc.connect()
```

Desconexão automática em modo inativo (após algum tempo sem enviar nenhuma mensagem):

```js
const ipc = new IpcMessenger({
  disconnectOnIdle: true,
  idleTimeout: 3000 // Padrão
})

ipc.send({ ... })

setTimeout(() => {
  console.log(ipc.connected) // false
}, 3000)
```

Conectar com outra rede IPC:

```js
const ipc = new IpcMessenger({
  networkId: 'com.my-name.my-ipc-network'
})
```

Em um arquivo vue-cli `ui.js`, você pode usar os métodos `ipcOn`, `ipcOff` e `ipcSend`:

```js
function onWebpackMessage ({ data: message }) {
  if (message['com.my-name.some-data']) {
    console.log(message['com.my-name.some-data'])
  }
}

// Espera por alguma mensagem IPC
api.ipcOn(onWebpackMessage)

// Ṕara de esperar pelas mensagens IPC
api.ipcOff(onWebpackMessage)

// Envie uma mensagem para todas as instâncias do IpcMessenger conectadas
api.ipcSend({
  webpackDashboardMessage: {
    foo: 'bar'
  }
})
```

## Armazenamento local

Um plug-in pode salvar e carregar dados do banco de dados local [lowdb](https://github.com/typicode/lowdb) usado pelo servidor da interface do usuário.

```js
// Armazenar um valor no banco de dados local
api.storageSet('com.my-name.an-id', { some: 'value' })

// Recuperar um valor do banco de dados local
console.log(api.storageGet('com.my-name.an-id'))

// Instância completa do lowdb
api.db.get('posts')
  .find({ title: 'low!' })
  .assign({ title: 'hi!'})
  .write()

// Auxiliares com nomes
const { storageGet, storageSet } = api.namespace('my-plugin.')
```

::: danger
Certifique-se de nomear os IDs corretamente, pois eles devem ser exclusivos em todos os plug-ins. É recomendado usar a [notação de nome de domínio reverso](https://en.wikipedia.org/wiki/Reverse_domain_name_notation).
:::

## Notificação

Você pode exibir notificações usando o sistema de notificação do sistema operacional do usuário:

```js
api.notify({
  title: 'Some title',
  message: 'Some message',
  icon: 'path-to-icon.png'
})
```

Existem alguns ícones embutidos:

- `'done'`
- `'error'`

## tela de progresso

Você pode exibir uma tela de progresso com algum texto e uma barra de progresso:

```js
api.setProgress({
  status: 'Upgrading...',
  error: null,
  info: 'Step 2 of 4',
  progress: 0.4 // from 0 to 1, -1 significa barra de progresso oculta
})
```

Remova a tela de progresso:

```js
api.removeProgress()
```

## Ganchos

Ganchos permitem reagir a certos eventos cli-ui.

### onProjectOpen

Chamado quando o plug-in é carregado pela primeira vez para o projeto atual.

```js
api.onProjectOpen((project, previousProject) => {
  // Redefine dados
})
```

### onPluginReload

Chamado quando o plugin é recarregado.

```js
api.onPluginReload((project) => {
  console.log('plugin reloaded')
})
```

### onConfigRead

Chamado quando uma tela de configuração é aberta ou atualizada.

```js
api.onConfigRead(({ config, data, onReadData, tabs, cwd }) => {
  console.log(config.id)
})
```

### onConfigWrite

Chamado quando o usuário salva em uma tela de configuração.

```js
api.onConfigWrite(({ config, data, changedFields, cwd }) => {
  // ...
})
```

### onTaskOpen

Chamado quando o usuário abre um painel de detalhes da tarefa.

```js
api.onTaskOpen(({ task, cwd }) => {
  console.log(task.id)
})
```

### onTaskRun

Chamado quando o usuário executa uma tarefa.

```js
api.onTaskRun(({ task, args, child, cwd }) => {
  // ...
})
```

### onTaskExit

Chamado quando existe uma tarefa. Pode ser chamado tanto em caso de sucesso quanto falha.

```js
api.onTaskExit(({ task, args, child, signal, code, cwd }) => {
  // ...
})
```

### onViewOpen

Chamado quando os usuários abrem uma visualização (como 'Plugins', 'Configurations' ou 'Tasks').

```js
api.onViewOpen(({ view, cwd }) => {
  console.log(view.id)
})
```

## Sugestões

Sugestões são botões destinados a propor uma ação ao usuário. Eles são exibidos na barra superior. Por exemplo, podemos ter um botão que sugira instalar o vue-router se o pacote não for detectado no aplicativo.

```js
api.addSuggestion({
  id: 'com.my-name.my-suggestion',
  type: 'action', // Obrigatório (mais tipos no futuro)
  label: 'Add vue-router',
  // Isso será exibido em um modal de detalhes
  message: 'A longer message for the modal',
  link: 'http://link-to-docs-in-the-modal',
  // Imagem opcional
  image: '/_plugin/my-package/screenshot.png',
  // Função chamada quando a sugestão é ativada pelo usuário
  async handler () {
    // ...
    return {
      // Por padrão remove o botão
      keep: false
    }
  }
})
```

::: danger
Certifique-se de namespace o id corretamente, uma vez que deve ser exclusivo em todos os plugins. É recomendado usar a [notação de nome de domínio reverso](https://en.wikipedia.org/wiki/Reverse_domain_name_notation).
:::

![UI - Sugestão](/suggestion.png)

Então você pode remover a sugestão:

```js
api.removeSuggestion('com.my-name.my-suggestion')
```

Você também pode abrir uma página quando o usuário ativar a sugestão com o `actionLink`:

```js
api.addSuggestion({
  id: 'com.my-name.my-suggestion',
  type: 'action', // Obrigatório
  label: 'Add vue-router',
  // Abre uma nova aba
  actionLink: 'https://vuejs.org/'
})
```

Normalmente, você usará ganchos para exibir a sugestão no contexto certo:

```js
const ROUTER = 'vue-router-add'

api.onViewOpen(({ view }) => {
  if (view.id === 'vue-project-plugins') {
    if (!api.hasPlugin('vue-router')) {
      api.addSuggestion({
        id: ROUTER,
        type: 'action',
        label: 'org.vue.cli-service.suggestions.vue-router-add.label',
        message: 'org.vue.cli-service.suggestions.vue-router-add.message',
        link: 'https://router.vuejs.org/',
        async handler () {
          await install(api, 'vue-router')
        }
      })
    }
  } else {
    api.removeSuggestion(ROUTER)
  }
})
```

Neste exemplo, exibimos apenas a sugestão do vue-router na visualização de plugins e se o projeto já não tiver o vue-router instalado.

Nota: `addSuggestion` e `removeSuggestion` podem ser namespaced com `api.namespace()`.

## Widgets

Você pode registrar um widget para o painel do projeto no seu arquivo ui do plugin:

```js
registerWidget({
  // ID Exclusivo
  id: 'org.vue.widgets.news',
  // Informações Básicas
  title: 'org.vue.widgets.news.title',
  description: 'org.vue.widgets.news.description',
  icon: 'rss_feed',
  // Componente principal usado para renderizar o widget
  component: 'org.vue.widgets.components.news',
  // (Opcional) Componente secundário para visualização de widgets 'tela cheia'
  detailsComponent: 'org.vue.widgets.components.news',
  // Tamanho
  minWidth: 2,
  minHeight: 1,
  maxWidth: 6,
  maxHeight: 6,
  defaultWidth: 2,
  defaultHeight: 3,
  // (Opcional) Limite o número máximo desse widget no painel
  maxCount: 1,
  // (Opcional) Adicione um botão de 'tela cheia' no cabeçalho do widget
  openDetailsButton: true,
  // (Opcional) Configuração padrão para o widget
  defaultConfig: () => ({
    url: 'https://vuenews.fireside.fm/rss'
  }),
  // (Opcional) Requer que o usuário configure o widget quando adicionado
  // Você não deve usar `defaultConfig` com isso
  needsUserConfig: true,
  // (Opcional) Exibir prompts para configurar o widget
  onConfigOpen: async ({ context }) => {
    return {
      prompts: [
        {
          name: 'url',
          type: 'input',
          message: 'org.vue.widgets.news.prompts.url',
          validate: input => !!input // Required
        }
      ]
    }
  }
})
```

Nota: `registerWidget` pode ser namespaced com` api.namespace () `.

## Outros métodos

### hasPlugin

Retorna `true` se o projeto usa o plugin.

```js
api.hasPlugin('eslint')
api.hasPlugin('apollo')
api.hasPlugin('vue-cli-plugin-apollo')
```

### getCwd

Recupera o diretório de trabalho atual.

```js
api.getCwd()
```

### resolve

Resolve um arquivo dentro do projeto atual.

```js
api.resolve('src/main.js')
```

### getProject

Pega um projeto atualmente aberto.

```js
api.getProject()
```

### requestRoute

Alterna o usuário em uma rota específica no Web client.

```js
api.requestRoute({
  name: 'foo',
  params: {
    id: 'bar'
  }
})

api.requestRoute('/foobar')
```

## Arquivos estáticos públicos

Pode ser necessário expor alguns arquivos estáticos sobre o servidor HTTP integrado (normalmente, se você quiser especificar um ícone para uma exibição customizada).

Qualquer arquivo em uma pasta `ui-public` opcional na raiz da pasta do pacote plugin será exposto na rota HTTP `/_plugin/:id/*`.

Por exemplo, se você colocar um arquivo `my-logo.png` na pasta `vue-cli-plugin-hello/ui-public/`, ele estará disponível com o `/_plugin/vue-cli-plugin-hello/my-logo.png` URL quando o cli-ui carrega o plugin.

```js
api.describeConfig({
  /* ... */
  // Imagem personalizada
  icon: '/_plugin/vue-cli-plugin-hello/my-logo.png'
})
```
