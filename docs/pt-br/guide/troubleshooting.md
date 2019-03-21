# Solução de problemas

Este documento aborda alguns problemas comuns da Vue CLI e como resolvê-los. Você deve sempre seguir estas etapas antes de abrir um novo problema.

## Executando a instalação com o `sudo` ou como` root`

Se você instalar `@vue/cli-service` como usuário `root` ou com `sudo`, pode haver problemas ao executar os scripts `postinstall` do pacote.

Este é um recurso de segurança do npm. Você deve sempre evitar executar o npm com privilégios de root, pois os scripts de instalação podem ser intencionalmente maliciosos.

Se você precisar, no entanto, você pode solucionar esse erro configurando o sinalizador `--unsafe-perm` do npm. Isso pode ser feito prefixando o comando com uma variável de ambiente, ou seja,

```bash
npm_config_unsafe_perm = true vue create my-project
```

## Links simbólicos em `node_modules`

Se houver dependências instaladas pelo `npm link` ou `yarn link` o ESLint (e às vezes o Babel também) pode não funcionar corretamente para as dependências com links simbólicos. É porque o [webpack define links simbólicos para seus locais reais por padrão](https://webpack.js.org/configuration/resolve/#resolvesymlinks), portanto interrompe a pesquisa de configuração do ESLint / Babel.

Uma solução para esse problema é desabilitar manualmente a resolução de links simbólicos no webpack:

```js
// vue.config.js
module.exports = {
  chainWebpack: (config) => {
    config.resolve.symlinks(false)
  }
}
```

::: warning
Desativar `resolve.symlinks` pode interromper o recarregamento do módulo ativo se suas dependências forem instaladas por clientes npm de terceiros que utilizaram links simbólicos, como `cnpm` ou `pnpm`.