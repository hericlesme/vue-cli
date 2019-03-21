# UI Plugin Info

Quando usado na interface do usuário, seu plug-in pode mostrar informações adicionais para torná-lo mais detectável e reconhecível.

## Logo

Você pode colocar um arquivo `logo.png` no diretório raiz da pasta que será publicada no npm. Ele será exibido em vários lugares:
 - Ao procurar por um plugin para instalar
 - Na lista de plugins instalados

![Plugins](/plugins.png)

O logotipo deve ser uma imagem quadrada e não transparente (idealmente 84x84).

## Descoberta

Para melhor descoberta quando um usuário procura por seu plugin, coloque palavras-chave descrevendo seu plugin no campo `description` do arquivo `package.json`.

Exemplo:

```json
{
  "name": "vue-cli-plugin-apollo",
  "version": "0.7.7",
  "description": "plugin vue-cli para adicionar Apollo e GraphQL"
}
```

Você deve adicionar o url ao site ou repositório do plugin no campo `homepage` ou `repository` para que um botão 'More info' seja exibido na descrição do seu plugin:

```json
{
  "repository": {
    "type": "git",
    "url": "git+https://github.com/Akryum/vue-cli-plugin-apollo.git"
  },
  "homepage": "https://github.com/Akryum/vue-cli-plugin-apollo#readme"
}
```

![Item de pesquisa de plug-in](/plugin-search-item.png)