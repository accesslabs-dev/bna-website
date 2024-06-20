---
title: "Do zero ao bnaGPT (Parte 1)"
meta_title: "How to Build Your Own ChatGPT from Scratch - Step-by-Step Guide"
description: "Artigo ensinando a como criar um chatGPT do zero"
reading_time: "5 min"
date: 2024-06-17T05:00:00Z
# image: "/images/image-placeholder.png"
categories: ["LLM"]
author: "Gabriel Pinto"
tags: ["Coding"]
draft: false
---

Parte 1 do nosso bnaGPT. Te ensinando a montar um chatGPT do zero!

## Parte 1

Na parte 0 fizemos um chat básico e a UI do nosso bnaGPT.

Hoje continuaremos a adicionar mais funcionalidades. Vamos focar em:
- Memória
- Salvar conversas

Lista de funcionalidade até agora:

- [x] Chat = você manda uma mensagem e ele te responde :sleeping:
- [ ] Memória = você manda várias mensagens e ele te responde levando em consideração as mensagens passadas :sleeping:
- [ ] Salvar conversas = você consegue reabrir conversas passadas e recomeçá-las :unamused:
- [ ] Buscar na internet = procura a sua pergunta na internet antes de responder :confused:
- [ ] Ler documentos = lê um documento com contexto sobre a sua pergunta (ou não né, sei lá o que vocês mandam) antes de responder :neutral_face:
- [ ] Ler imagens = você manda uma mensagem com algum documento com imagens ou imagem e ele te responde :relieved:
- [ ] Rodar códigos = o agente consegue rodar algum código que ele mesmo criou :grinning:
- [ ] GPT / assistente = você consegue criar GPTs para funções específicas :anguished:
- [ ] Gerar imagens = ele cria imagens a partir de algum contexto :open_mouth:
- [ ] Recebe áudio / gera áudio = entende seus áudios e responde em áudio :scream:
- [ ] Lava a sua louça :skull:

- [ ] Bônus: JSON mode = te responder somente em JSON (isso ajuda muito se você é um dev e está usando IA em algum produto).

## Parte 1 - Memória e salvar conversas

### Passo a Passo para iniciantes

Nesta seção vamos documentar as mudanças feitas no código para adicionar essas novas funcionalidades de uma maneira mais simples.

1. Função para salvar conversa -> aqui você deve designar um nome de arquivo JSON para salvar as conversas e salvar o nome na variável `CONVERSATIONS_FILE`

Exemplo:

```python
CONVERSATIONS_FILE = "conversas.json"
```

Código para salvar uma conversa no arquivo criado.

```python
# Function to save all conversations to a single file
def save_conversation(convo_name):
    convo_file = CONVERSATIONS_FILE
    if os.path.exists(convo_file):
        with open(convo_file, "r") as f:
            all_conversations = json.load(f)
    else:
        all_conversations = {}

    all_conversations[convo_name] = st.session_state.messages

    with open(convo_file, "w") as f:
        json.dump(all_conversations, f)

    st.session_state.saved_conversations = list(all_conversations.keys())
    st.success(f"Conversation '{convo_name}' saved!")
```

2. Função para listar todas as conversar salvas:
```python
# Function to list all saved conversations
def list_conversations():
    convo_file = CONVERSATIONS_FILE
    if os.path.exists(convo_file):
        try:
            with open(convo_file, "r") as f:
                all_conversations = json.load(f)
            return list(all_conversations.keys())
        except Exception:
            return []
    return []
```

3. Função para carregar alguma conversa específica:
```python
# Function to load a conversation from the file
def load_conversation(convo_name):
    convo_file = CONVERSATIONS_FILE
    if os.path.exists(convo_file):
        with open(convo_file, "r") as f:
            all_conversations = json.load(f)
        if convo_name in all_conversations:
            st.session_state.messages = all_conversations[convo_name]
            st.success(f"Conversation '{convo_name}' loaded!")
        else:
            st.error(f"No conversation found with the name '{convo_name}'.")
    else:
        st.error("No conversations file found.")
```

4. Inicializar as novas variáveis de sessão usadas:
    - show_confirm_save: variável inicializada como `False` e se torna `True` quando o usuário clica para salvar uma nova conversa e um aparece um _textbox_ para o usuário colocar um nome para essa conversa e confirmar o salvamento;
    ```python
    # show_confirm_save = variable that will store whether the button of
    # confirm save should be seen
    if "show_confirm_save" not in st.session_state:
        st.session_state.show_confirm_save = False
    ```
    - saved_conversations: variável com todas as conversas já salvas no arquivo JSON. Ela é inicializada com as conversas salvas pela função list_conversation()
    ```python
    # saved_conversations = variable to store all conversations
    if "saved_conversations" not in st.session_state:
        st.session_state.saved_conversations = list_conversations()
    ```

5. Adicionar elementos de UI na sidebar. O comando `with st.sidebar:` faz com que todos os elementos criados dentro deste with sejam colocados na sidebar:
    - Botão para salvar conversa atual
    ```python
    # Sidebar to load saved conversations
    with st.sidebar:
        if st.button("Save Current Conversation"):
            st.session_state.show_confirm_save = True
    ```
    - O que acontece quando botão de salvar conversa é clicado -> aparece o _textbox_ e o botão de confirmar salvamento
    ```python
        if st.session_state.show_confirm_save:
            save_convo_name = st.text_input("Enter a name to save the conversation:")
            if st.button("Confirm Save") and save_convo_name:
                save_conversation(save_convo_name)
                st.session_state.show_confirm_save = False
    ```
    - Listar conversas salvas
    ```python
        # UI for saving a conversation
        st.subheader("Saved Conversations")
        for convo_name in st.session_state.saved_conversations:
            if st.sidebar.button(convo_name):
                load_conversation(convo_name)
    ```

6. Alterar a função de chat do Groq anterior para levar em consideração todas as mensagens já enviadas na conversa:
```python
# Now creating the assistant response
with st.chat_message("assistant"):
    # Using groq to get the response
    stream = client.chat.completions.create(
        model=MODEL_NAME,
        messages=st.session_state.messages,
        # Setting stream as True to get the response little by little
        # Set as False if you want response at once
        stream=True,
    )
    response = st.write_stream(parse_groq_stream(stream))
st.session_state.messages.append({"role": "assistant", "content":response})
```

Código final em nosso [repositório.](https://github.com/bna-dev-public/bnaGPT/blob/main/101s/chatbot_with_memory.py)

### Passo a passo para devs

Agora vamos criar o passo a passo do nosso projeto para devs. Lembrando que estamos utilizando uma arquitetura MVC (model-view-controller), ou seja, temos interfaces e arquivos separados para tudo que seja:
- model -> como estruturamos nossos dados = nossas mensagens
- view -> como estruturamos nosso front-end = funções do streamlit
- controller -> como estruturamos nosso back-end = interface com o llama-index, LLMs, modelos de embedding e bases vetoriais (em breve).

1. Pull no nosso [repositório](https://github.com/bna-dev-public/bnaGPT) :)

Agora todas as mudanças que fizemos:

2. settings.py: adicionamos uma variável CONVERSATIONS_FILE e atualizamos as variáveis de LLM para facilitar um pouco
3. view/chatbot_with_memory.py: toda a interface de salvar, carregar e deletar conversas salvas.
    - nova sidebar com UI botões de salvar e confirmar novo salvamento
    - listar todas as conversas com as opções de carregar ou deletar a conversa salva
    ```python
    # build the page sidebar
    def build_sidebar(self):
        with st.sidebar:
            image = load_image(self.logo_filename)
            st.logo(image)
            st.markdown("#### Click here to save current conversation!")
            if st.button("Save", use_container_width=True, type="primary"):
                st.session_state.show_confirm_save = True
            if st.session_state.show_confirm_save:
                save_convo_name = st.text_input(
                    "Enter a name to save the conversation:"
                )
                if st.button("Confirm Save") and save_convo_name:
                    conversation_names = save_conversation(
                        save_convo_name, st.session_state.messages
                    )
                    st.session_state.show_confirm_save = False
                    st.session_state.saved_conversations = list(conversation_names)
                    st.success(f"Conversation '{save_convo_name}' saved!")
            st.markdown("#")
            st.markdown("Saved Conversations")

            # printing every saved conversation
            for convo_name in st.session_state.saved_conversations:
                with st.popover(
                    convo_name,
                    use_container_width=True,
                ):
                    load_col, delete_col = st.columns([1, 1])
                    with load_col:
                        if st.button(
                            "Load",
                            key=f"load_{convo_name}",
                            use_container_width=True,
                            type="primary",
                        ):
                            messages, status = load_conversation(convo_name)
                            if status == "success":
                                st.session_state.messages = messages
                                # formatted_messages =
                                memory = chatbot.create_memory_from_historical_messages(
                                    st.session_state.messages
                                )
                                st.session_state.agent = chatbot.create_openAI_agent(
                                    memory=memory
                                )
                                st.experimental_rerun()
                            elif status == "wrong convo name":
                                st.error("This conversation could not be found")
                            else:
                                st.error("There is no historical conversations saved")
                    with delete_col:
                        if st.button(
                            "Delete",
                            key=f"delete_{convo_name}",
                            use_container_width=True,
                        ):
                            status, text = delete_conversation(convo_name)
                            if status == 200:
                                st.success(f"Conversation '{convo_name}' deleted!")
                                st.session_state.saved_conversations = (
                                    list_conversations()
                                )
                                st.experimental_rerun()
                            elif text == "no conversation found":
                                st.error(
                                    f"""No conversation found with the
                                    name '{convo_name}'"""
                                )
                            else:
                                st.error("No conversations file found")
    ```
    - nova inicialização de variáveis da sessão
    ```python
    # Streamlit has a session element that you can create and save variables in it
    # So we need to update it once our main runs in a loop
    def initialize_session(self):

        # show_confirm_save = variable that will store whether the button of
        # confirm save should be seen
        if "show_confirm_save" not in st.session_state:
            st.session_state.show_confirm_save = False

        # saved_conversations = variable to store all conversations
        if "saved_conversations" not in st.session_state:
            st.session_state.saved_conversations = list_conversations()

        # we can create a variable called messages
        # to save and output the messages
        if "messages" not in st.session_state:
            st.session_state.messages = []

        # agent = variable to store openAIAgent
        if "agent" not in st.session_state:
            memory = chatbot.create_memory_from_historical_messages(
                st.session_state.messages
            )
            st.session_state.agent = chatbot.create_openAI_agent(memory=memory)

        # output messages where:
        # role = user or assistant
        # content = content of the message
        for message in st.session_state.messages:
            with st.chat_message(message.role.value):
                st.markdown(message.content)
    ```
    - alterar interface com chatbot. Agora usaremos um agente (vou entrar em detalhes quando entrarmos nas modificações no controller)
    ```python
    # now creating the assistant response
    with st.chat_message(MessageRole.ASSISTANT):
        # getting the streaming response from controller
        streaming_response = StreamingMessage(
            st.session_state.agent.stream_chat(chat_input)
        )
    ```
4. Alterações em utils/llama.py para novas funções do controller
    - criar um agente openAI. Por mais que a class se chame OpenAIAgent, ela na verdade é uma implementação do Llama-Index com base em como o agente da OpenAI funciona. Isso quer dizer: tem a funcionalidade de chat com memória e suporta diversas funções que podemos implementar (por exemplo: fazer queries em banco de dados, interagir com APIs, fazer cálculos, rodar códigos). Por agora, teremos somente a memória e nos próximos artigos incluíremos mais coisas.
    ```python
    def create_openAI_agent(
        self,
        memory: ChatMemoryBuffer = ChatMemoryBuffer.from_defaults(),
    ) -> OpenAIAgent:
        llm = self.create_llm()
        agent = OpenAIAgent.from_tools(
            llm=llm,
            memory=memory,
        )
        return agent
    ```
5. Novo controller para a nova página em controller/chatbot_with_memory.py
    - criar memória atráves as mensagens passadas
    ```python
    @staticmethod
    def create_memory_from_historical_messages(
        messages: List[Message],
    ) -> ChatMemoryBuffer:
        chat_history = []
        for message in messages:
            chat_history.append(message.convert_message_to_chat_message())
        memory = ChatMemoryBuffer.from_defaults(chat_history=chat_history)
        return memory
    ```
    - criar o agente OpenAI
    ```python
    def create_openAI_agent(
        self, memory: ChatMemoryBuffer = ChatMemoryBuffer.from_defaults()
    ) -> OpenAIAgent:
        openAI_agent = self.llama.create_openAI_agent(memory)
        return openAI_agent
    ```
    - conversar com o agente:
    ```python
    @staticmethod
    def chat_with_openAI_agent(openAI_agent, user_input) -> StreamingAgentChatResponse:
        if openAI_agent is not None:
            try:
                streaming_response = openAI_agent.stream_chat(user_input)
                return streaming_response
            except Exception as e:
                raise ValueError(str(e))
        else:
            raise ValueError("No OpenAI agent was found")
    ```
    - funções para salvar, deletar, ler e listar conversas salvas
    ```python
    # Function to list all saved conversations
    def list_conversations():
        convo_file = CONVERSATIONS_FILE
        if os.path.exists(convo_file):
            try:
                with open(convo_file, "r") as f:
                    all_conversations = json.load(f)
                return list(all_conversations.keys())
            except Exception:
                return []
        return []


    # Function to save all conversations to a single file
    def save_conversation(convo_name, messages: List[Message]):
        convo_file = CONVERSATIONS_FILE
        if os.path.exists(convo_file):
            with open(convo_file, "r") as f:
                all_conversations = json.load(f)
        else:
            all_conversations = {}
        formatted_messages = []
        for message in messages:
            formatted_messages.append(message.create_message())
        all_conversations[convo_name] = formatted_messages
        with open(convo_file, "w") as f:
            json.dump(all_conversations, f)
        return list(all_conversations.keys())


    # Function to load a conversation from the file
    def load_conversation(convo_name) -> Tuple[List[Message], str]:
        convo_file = CONVERSATIONS_FILE
        if os.path.exists(convo_file):
            with open(convo_file, "r") as f:
                all_conversations = json.load(f)
            if convo_name in all_conversations:
                messages = []
                print(all_conversations[convo_name])
                for message in all_conversations[convo_name]:
                    if message.get("role", "") == "user":
                        messages.append(
                            Message(MessageRole.USER, message.get("content", ""))
                        )
                    if message.get("role", "") == "assistant":
                        messages.append(
                            Message(MessageRole.ASSISTANT, message.get("content",   ""))
                        )
                return messages, "success"
            else:
                return [], "wrong convo name"
        else:
            return [], "no file found"


    # Function to delete a conversation from the file
    def delete_conversation(convo_name) -> Tuple[int, str]:
        convo_file = CONVERSATIONS_FILE
        if os.path.exists(convo_file):
            with open(convo_file, "r") as f:
                all_conversations = json.load(f)
            if convo_name in all_conversations:
                del all_conversations[convo_name]
                with open(convo_file, "w") as f:
                    json.dump(all_conversations, f)
                return 200, "success"
            else:
                return 400, "no conversation found"
        else:
            return 400, "no conversations file found"
    ```
6. Novo model para formatar as mensagens para funcionar com o agente:
```python
from llama_index.core.base.llms.types import ChatMessage, MessageRole


@dataclass(slots=True)
class Message:
    role: MessageRole
    content: str

    def create_message(self):
        message = {"role": self.role.value, "content": self.content}
        return message

    def convert_message_to_chat_message(self) -> ChatMessage:
        chat_message = ChatMessage(role=self.role, content=self.content)
        return chat_message

```
7. Criar nova página com nova implementação em pages/1_Chatbot_With_Memory.py
```python
from view.chatbot_with_memory import ChatbotWithMemoryView


# main function of the Simple Chatbot
def main():
    sc = ChatbotWithMemoryView()
    sc.build_config()
    chat_input = sc.build_body()
    sc.initialize_session()
    sc.build_sidebar()
    sc.handle_chat_input(chat_input)


if __name__ == "__main__":
    main()

```

## Conclusão

Agora você tem um bnaGPT gratuito local para usar quando quiser! Ainda sem todas as funcionalidade do chatGPT, mas 90% das funcionalidades já estão aí!
