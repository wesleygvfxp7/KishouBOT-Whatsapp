# Desenvolvendo o Kishou BOT

Este projeto é um bot de WhatsApp baseado em Baileys. O fluxo de entrada está em `dados/src/connect.js`: ele cria a conexão, recebe eventos e delega mensagens ao dispatcher em `dados/src/index.js`. O dispatcher normaliza prefixo e aliases, prepara o contexto (chat, usuário, grupo e permissões) e executa o `switch (command)`. Os menus exibidos no WhatsApp ficam em `dados/src/menus/` e seus módulos são registrados em `dados/src/menus/index.js`.

## Adicionar um comando

1. Escolha um nome curto e minúsculo. Procure no dispatcher para evitar colisões com comandos existentes e aliases.
2. Adicione um `case 'meucomando':` dentro do `switch (command)` em `dados/src/index.js`.
3. Use as variáveis já preparadas pelo dispatcher. Em geral, `q` contém os argumentos, `prefix` o prefixo configurado, `from` o chat, `sender` o usuário, `isGroup` o tipo de chat e `reply(text)` responde à mensagem original.
4. Para funções administrativas, confira o padrão de autorização existente no bloco próximo ao comando e valide permissões antes de efeitos colaterais.
5. Faça operações de rede e arquivos dentro de `try/catch`, valide argumentos antes de usá-los e evite bloquear o event loop com operações síncronas demoradas.
6. Acrescente o comando ao menu correspondente em `dados/src/menus/` para que as pessoas descubram a funcionalidade. Se criar uma categoria de menu, registre-a também em `dados/src/menus/index.js` e adicione seu alias ao switch que encaminha comandos de menu.

Exemplo mínimo dentro do switch:

```js
case 'eco': {
  if (!q) return reply(`Uso: ${prefix}eco <texto>`);
  return reply(`🌸 ${q}`);
}
```

### Comando de brincadeira com sorteio

Um comando de brincadeira deve ser previsível, evitar efeitos destrutivos e tratar argumento vazio. Por exemplo, para um comando `!sorte`:

```js
case 'sorte': {
  const alvos = q.trim();
  if (!alvos) return reply(`Uso: ${prefix}sorte <nome ou pergunta>`);
  const resultado = Math.floor(Math.random() * 100) + 1;
  return reply(`🍀 Sorte de *${alvos}*: *${resultado}%*`);
}
```

Adicione a descrição `sorte <nome ou pergunta>` ao menu de brincadeiras em `dados/src/menus/menubn.js`. Se a brincadeira envolve dois participantes, use `mentionedJid` ou o contexto de mensagem citado em vez de confiar em nomes digitados como identidade.

### Criar uma brincadeira sem alterar código

O dono pode criar respostas simples em tempo real com `addcmd`:

```text
!addcmd piada Por que o computador foi ao médico? Porque estava com um vírus! 😄
!piada
```

Também é possível usar argumentos e placeholders:

```text
!addcmd elogio [param:string:nome:required] 🌟 {nome}, hoje você está com energia de protagonista!
!elogio Akira
```

Para uma rolagem configurável, o comando personalizado pode receber e validar um número, mas a resposta salva é texto fixo; sorteios reais, placares, cooldowns ou estado de jogo devem ser implementados em JavaScript para evitar resultados falsos ou inconsistentes.

O prefixo pode ser diferente de `!`; consulte `dados/src/config.json`. Comandos personalizados são persistidos em `dados/database/customCommands.json` e podem ser editados/removidos pelos comandos `edcmd` e `delcmd`.

## Adicionar uma rota / endpoint

Kishou BOT não usa um servidor HTTP nem rotas REST no momento. Seus pontos de entrada são eventos do WhatsApp (mensagens, conexão e atualizações de grupo) registrados em `dados/src/connect.js`; comandos são roteados pelo dispatcher em `dados/src/index.js`.

Para uma integração HTTP futura:

1. Crie um módulo dedicado em `dados/src/routes/`, com uma função de registro que recebe a instância do servidor, configuração e dependências necessárias.
2. Escolha um framework HTTP e declare-o como dependência em `package.json`; mantenha o servidor isolado do ciclo de conexão do WhatsApp.
3. Registre as rotas a partir de um módulo de inicialização explícito. Valide esquema, autenticação e limites de requisição antes de executar ações.
4. Nunca exponha credenciais, arquivos de sessão, QR codes, dados de usuários ou funções administrativas sem autorização. Mantenha segredos em variáveis de ambiente, não no repositório.
5. Documente método, caminho, autenticação, parâmetros, respostas e erros. Ao adicionar uma rota, atualize esta seção e forneça exemplos reproduzíveis.

Não coloque rotas REST dentro do `switch (command)`: comandos WhatsApp e endpoints HTTP têm protocolos, ciclo de vida e autenticação diferentes.

## Onde cada mudança costuma ficar

| Mudança | Local |
| --- | --- |
| Comportamento de comando | `dados/src/index.js` |
| Texto e lista de comandos no menu | `dados/src/menus/*.js` |
| Registro de uma nova categoria de menu | `dados/src/menus/index.js` e dispatcher |
| Recepção/conexão WhatsApp | `dados/src/connect.js` |
| Persistência de configurações e dados | `dados/src/utils/database.js` |
| Utilitários reutilizáveis | `dados/src/utils/` ou `dados/src/funcs/utils/` |
| Configuração da instância | `dados/src/config.json` |
| Inicialização, QR e reconexão no terminal | `dados/src/.scripts/start.js` |

## Convenções

- Preserve nomes de configuração existentes para manter compatibilidade com instalações já configuradas.
- Reutilize helpers e validação do dispatcher em vez de criar um segundo fluxo de permissões.
- Não registre tokens, números privados, conteúdo de mensagens ou dados de autenticação em logs.
- Use o prefixo configurado nas mensagens de ajuda e descreva argumentos inválidos com um exemplo.
- Mantenha a documentação e o menu atualizados junto com a funcionalidade.
