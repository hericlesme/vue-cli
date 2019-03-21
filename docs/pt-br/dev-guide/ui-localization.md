# Localização da interface do usuário

## Traduza a interface do usuário padrão

Para facilitar a colaboração e a sincronização, a localidade de origem em inglês da ramificação `dev` é automaticamente importada para o [Transifex](https://www.transifex.com/vuejs/vue-cli/dashboard/), uma plataforma para tradução colaborativa.

Para idiomas existentes, você pode [inscrever-se como tradutor](https://www.transifex.com/vuejs/vue-cli/dashboard/).
Para novos idiomas, você pode [solicitar o novo idioma](https://www.transifex.com/vuejs/vue-cli/dashboard/) depois de se inscrever.

Em ambos os casos, você poderá traduzir as chaves à medida que forem adicionadas ou alteradas na localidade de origem.

## Traduza seu plugin

Você pode colocar arquivos locais compatíveis com [vue-i18n](https://github.com/kazupon/vue-i18n) em uma pasta `locales` na raiz do seu plugin. Eles serão carregados automaticamente no cliente quando o projeto for aberto. Você pode usar `$t` para traduzir strings em seus componentes e outros helpers vue-i18n. Além disso, as strings usadas na API da  interface do usuário (como `describeTask`) também passarão pelo vue-i18n para que você possa localizá-las.

Exemplo de pasta `locales`:

```
vue-cli-plugin/locales/en.json
vue-cli-plugin/locales/fr.json
```

Exemplo de uso na API:

```js
api.describeConfig ({
  // vue-i18n path
  description: 'com.my-name.my-plugin.config.foo'
})
```

::: danger
Certifique-se de namespace o id corretamente, uma vez que deve ser exclusivo em todos os plugins. É recomendado usar a [notação de nome de domínio reverso](https://en.wikipedia.org/wiki/Reverse_domain_name_notation).
:::

Exemplo de uso em componentes:

```html
<VueButton>{{$t('com.my-name.my-plugin.actions.bar')}}</VueButton>
```

Você também pode carregar os arquivos de local em um addon de cliente, se preferir, usando o `ClientAddonApi`:

```js
// Carrega os arquivos de local (uses vue-i18n)
const locales = require.context('./locales', true, /[a-z0-9]+\.json$/i)
locales.keys().forEach(key => {
  const locale = key.match(/([a-z0-9]+)\./i)[1]
  ClientAddonApi.addLocalization(locale, locales(key))
})
```