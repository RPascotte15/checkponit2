# checkponit2
Telegram Receiver
Primeiro contêiner que temos é o do Telegram Receiver, ele funciona quase que como um ouvido.
É responsável por ouvir os inputs do usuário e repassar adiante.

Seus dois fios conduzem a mensagens distintas:

Um vai para a função "Save context" e um nó de debug.

Outro vai para "Not authorized user" — porém, não há condição de filtro, logo, ele sempre será executado.
→ Corrigido com verificação de autorização.

🔐 Function: Save context
Aqui, o nó guarda no contexto do fluxo o ID do chat e o tipo da mensagem, para usá-los depois.
Em seguida, ele limpa o conteúdo da mensagem, mantendo apenas o content, que será enviado ao Watson Assistant.

É como guardar a armadura antes de entrar no templo da sabedoria artificial.

// === Verificação de segurança inicial ====================================
if (!msg?.payload) {
    node.warn('msg.payload não encontrado.');
    return null;
}

const { chatId, type, content, options } = msg.payload;

// === Armazena dados úteis no contexto de fluxo ===========================
Object.assign(context.flow, { chatId, type });

// === Prepara payload para envio final ====================================
// Se houver opções (teclado inline), mantemos como objeto com reply_markup.
// Se não houver, envia apenas o conteúdo como texto simples.
msg.payload = options ? { chatId, type, content, options } : content;

return msg;

🧙 Watson-assistant-v2
Este é o oráculo.
O fluxo envia a mensagem ao IBM Watson Assistant, que responde baseado nos diálogos treinados no assistente com o ID especificado.

O nó retorna o context para manter a conversa coerente.

Sai com duas conexões:

uma para debug

outra para reconstruir a resposta

🧩 Function: Restore context
Aqui, recupera-se o que foi armazenado anteriormente: chatId e type.
Depois, extrai-se a primeira resposta textual do Watson (índice [0]) e a mensagem é preparada para ser enviada de volta ao Telegram.


// === Metadados vindos do contexto =======================================
const { chatId, type } = context.flow;

// === Primeira resposta genérica do Watson ===============================
const watsonResponse = msg?.payload?.output?.generic?.[0];

if (!watsonResponse) {
    node.warn('Nenhuma resposta genérica retornada pelo Watson.');
    return null;
}

// === Constrói o payload conforme o tipo de resposta =====================
switch (watsonResponse.response_type) {

    // ---------- Texto simples --------------------------------------------
    case 'text':
        msg.payload = {
            chatId,
            type,
            content: watsonResponse.text
        };
        break;

    // ---------- Opções com botões ----------------------------------------
    case 'option':
        msg.payload = {
            chatId,
            type: 'message', // telegram exige “message” para botões
            content: watsonResponse.title,
            options: {
                reply_markup: {
                    inline_keyboard: watsonResponse.options.map(({ label, value }) => [
                        {
                            text: label,
                            callback_data: value?.input?.text ?? label // fallback seguro
                        }
                    ])
                }
            }
        };
        break;

    // ---------- Qualquer outro tipo --------------------------------------
    default:
        node.warn(`Tipo de resposta não tratado: ${watsonResponse.response_type}`);
        return null;
}

return msg;

🚀 Telegram Sender
Retorna a resposta ao usuário no Telegram.
Recebe mensagens já prontas para envio, com chatId, type e content.

🔁 Para resumir o fluxo:
A mensagem chega pelo Telegram.

É capturada e os dados são salvos.

O conteúdo é enviado ao Watson.

A resposta do Watson é reconstruída com os dados originais.

A resposta é enviada de volta ao Telegram.

📁 Arquivos usados:
Flow: Arquivo: json do fluxo de Node-RED

cpmentoria_bot: Arquivo .json dialog skill.
