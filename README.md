# checkponit2
Telegram - node-RED - IBM Watson
Telegram receiver
Primerio contêiner que temos é o do telegram receiver, ele funciona quase que como um ouvido. Ele é responsável por ouvir os inputs do usuário e repassa adiante.

Seus dois fios conduzem a mensagens distintas:

Um vai para a função "Save context" e um debug.
Outro vai direto para "Not authorized user" — porém, não há condição de filtro, logo, ele sempre será executado.
Function: Save context
Aqui, o nó guarda no contexto do fluxo o ID do chat e o tipo da mensagem, para usá-los depois. Em seguida, ele limpa o conteúdo da mensagem, mantendo apenas o content, que será enviado ao Watson Assistant. É como guardar a armadura antes de entrar no templo da sabedoria artificial.

context.flow.chatId = msg.payload.chatId;
context.flow.type = msg.payload.type;
msg.payload = msg.payload.content;
return msg;
Watson-assistant-v2
Este é o oráculo. O fluxo envia a mensagem ao IBM Watson Assistant, que responde baseado nos diálogos treinados no assistente com o ID especificado.

Retorna contexto para manter a conversa coerente.
Sai com duas conexões: uma para debug e outra para reconstruir a resposta.
Function: Restore context
Aqui, recupera-se o que foi armazenado anteriormente: chatId e type. Depois, extrai-se a primeira resposta textual do Watson (índice [0]) e a mensagem é pronta para ser enviada de volta ao Telegram.

msg.payload.chatId = context.flow.chatId;
msg.payload.type = context.flow.type;
msg.payload.content = msg.payload.output.generic[0].text;
return msg;
Function: Not authorized user
Define a resposta como “Você não é autorizado”. Porém, como mencionado, este bloco está sendo sempre chamado. O ideal seria haver um filtro condicional.

msg.payload.content = "You're not authorized user";
return msg;
Debug (1 e 2)
O primeiro mostra o que chega do telegram.
O segundo mostra a resposta do Watson antes da reconstrução final.
Telegram sender
Retorna a resposta ao usuário no Telegram. Recebe mensagens já prontas para envio, com chatId, type e content.

Então para resurmir o fluxo:

Mensagem chega pelo Telegram.

É capturada e os dados são salvos.

O conteúdo é enviado ao Watson.

A resposta do Watson é reconstruída com os dados originais.

A resposta é enviada de volta ao Telegram.

Flow
Arquivo .json do fluxo de Node-RED.

cpmentoria_bot
Arquivo .json dialog skill.
