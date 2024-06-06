---
title: "Do zero ao bnaGPT (Parte 0)"
meta_title: "How to Build Your Own ChatGPT from Scratch - Step-by-Step Guide"
description: "Artigo ensinando a como criar um chatGPT do zero"
reading_time: "5 min"
date: 2024-06-03T05:00:00Z
# image: "/images/image-placeholder.png"
categories: ["LLM"]
author: "Gabriel Pinto"
tags: ["Coding"]
draft: false
---

Começou a nova série da bna.dev na qual te ensinaremos a criar o seu próprio chatGPT!

## Do zero ao bnaGPT

Como dito, a ideia dessa série é te ajudar a construir um chatGPT para você.

"Tá, mas por que eu faria isso?"

1. Sede pelo conhecimento, mestres;
2. Já gastamos muito tempo no YouTube e no próprio chatGPT tentando entender as melhores maneiras ou práticas de se criar um chat do zero;
3. Entender melhor o que é o chatGPT, como ele funciona e o que você consegue replicar em outros lugares;
4. Pode ser divertido dependendo das escolhas que você fez na sua vida;
5. Ter uma versão mais barata e rápida do chatGPT (assim espero).

A ideia aqui é a cada "episódio" construir uma das funcionalidades do chatGPT até que a gente realmente tenha um bnaGPT tão bom ou melhor do que o chatGPT.

## Requisitos

De modo geral, para conseguir replicar o que faremos aqui, você precisará do seguinte:
1. Um computador ou pelo menos acesso ao Google Colab ou alguma outra ferramenta que te permita rodar códigos em Python;
2. Instalar Python em sua máquina (é fácil de fazer -> dá para seguir um tutorial no Youtube -> eventualmente, teremos um artigo de como configurar uma máquina para Python também). Qualquer coisa nos perguntem;
3. Conhecimento básico em alguma linguagem de programação, mas no ponto que estamos existe um mundo que dá para se virar no chatGPT também;
4. Um cartão de crédito para ter acesso (nem todas vocês terão que pagar) às algumas das ferramentas que vamos usar;
5. Determinação!

Outras coisas que podem te facilitar, mas não são obrigatórias:
1. Saber usar Git / Github para clonar nossos repositórios;
2. Docker se quiser _deployar_ o chat em algum site;
3. Biblioteca Streamlit se quiser mudar o _layout_ que criamos.

## Funcionalidades do chatGPT que replicaremos
O chatGPT pode ser resumido em uma UI com diversas _features_ de IA (não só Large Language Models). São elas:

- [ ] Chat = você manda uma mensagem e ele te responde :sleeping:
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

Bônus: JSON mode = te responder somente em JSON (isso ajuda muito se você é um dev e está usando IA em algum produto).

Mandem para gente se está faltando alguma funcionalidade!

## Parte 0 - **_Aesthetics_** e chat simples

Vamos lá!

Nessa série vamos tentar ao máximo dividir em dois códigos/projetos:
- Iniciante: aqui tentaremos deixar tudo dentro de um único código. Pode ser que após algumas features fique inviável, mas pelo menos
dessa forma você terá um único arquivo para rodar e vai funcionar;
- Devs: aqui já é a criação de um projeto pensando um pouco mais longo prazo, utilizando arquitetura MVC (explicaremos), para que
seja um código limpo, de fácil manutenção e fácil de implementar melhorias.

Python instalado? Consegue rodar um hello world?

Para quem quiser, todos os códigos estarão aqui https://github.com/bna-dev-public/bnaGPT. Na pasta 101s temos os arquivos para iniciantes e
todo o resto é "devs".

### Bibliotecas e ferramentas utilizadas

#### **_Aesthetics_**

Vamos usar a biblioteca Streamlit.

Por que Streamlit?

Em nossa singela opinião, é a biblioteca na qual você consegue de maneira mais simples atingir dois objetivos principais:
1. Facilidade de criação;
2. O seu site não parecer um aplicativo de Windows 98.

Existem outras opções, como:
1. Django -> não faça isso com você, já fizemos conosco. É uma arquitetura que demanda muitos arquivos, então mesmo para coisas simples (nosso caso) você vai ter que criar/escrever
um número consideravel de arquivos;
2. Shiny -> é uma boa alternativa, mas seu site vai parecer o Windows 98, a não ser que você gaste bastante tempo ajustando os **_Aesthetics_** da sua página e não queremos isso. Ou se você vai usar para
projetos de Data Science que precisa de gráficos;
3. Bokeh -> same as Shiny;
4. Solara -> bem parecido com Streamlit, acho que vale a tentativa. No entanto, é um pouco mais verborrágico e ainda não tem tanto suporte ou uma comunidade tão grande, então fica difícil achar bons exemplos que se aplicam
ao que você precisa.

Tem algumas outras, mas vão se encaixar em algum dos dois problemas e se tiver alguma que você particulamente acha que se encaixa bem, manda para nóis!
(nós talvez troquemos para Solara em alguns outros projetos).

> **SOMENTE PARA DEVS:** se for usar isso em prod e quer tem um site rodando um chatbot ou coisas do tipo. Dividiriámos assim:
>
> _Front-end:_ streamlit se você não ligar em usar templates e tiver que usar Python, mas seria preferível usar os frameworks em JS mesmo, tipo: Vanilla JS, React, Angular. Já sofremos algumas
> vezes tentando fazer um front-end massa usando Python.
>
> _Back-end:_ Flask ou Django (ele me irrita, mas daria) se você preferir usar Python, mas tentaria usar algum framework que converse mais com o seu front, como Node.js
>
> A escolha por JS é mais sobre a facilidade de encontrar pessoas que programam em JS, mas outras linguagens funcionariam bem.

#### Provedor de LLM e modelos de LLM usados
Por enquanto tudo funcionará com Groq e o modelo utilizado será o Llama 3 70b.

Por que Groq e Llama 3?
- Como a ideia é criar um chatGPT, queremos evitar ao máximo usar OpenAI.
Somente quando realmente existir uma grande vantagem;
- A combinação Groq e Llama 3 é extremamente rápida e com qualidade similar ao
GPT4 ou GPT4o;
- Veja o nosso artigo [LLMs - 101](https://www.bna.dev.br/blog/post-1/) para entender mais.

Para tudo funcionar você só precisa uma API key do Groq:
- **Configuração do Groq**. Você pode criar uma API key gratuitamente no [site deles](https://console.groq.com/)

### Passo a Passo para iniciantes

1. Instalar as bibliotecas:

```pip install streamlit groq```

2. Importar as bibliotecas:
```
from groq import Groq
import streamlit as st
```

3. Criar as configurações e métodos principais do Groq:

    3.1. Instanciar cliente para fazer requests para o Groq:

    Importante! Para a segurança da sua API key, é preferível que você use variáveis de ambiente.
    Você pode ter acesso à elas com o seguinte:
    ```
    import os
    GROQ_API_KEY = os.environ["GROQ_API_KEY"]
    ```

    ```
    # coloque sua API key aqui
    GROQ_API_KEY = <YOUR API KEY>
    # Crie o seu cliente
    client = Groq(
        api_key=GROQ_API_KEY,
    )
    # setar o modelo que vamos usar
    MODEL_NAME = "llama3-70b-8192"
    ```

    3.2. Método para usar respostas com streaming no Groq:
    ```
    def parse_groq_stream(stream):
        for chunk in stream:
            if chunk.choices:
                if chunk.choices[0].delta.content is not None:
                    yield chunk.choices[0].delta.content
    ```

4. Colocar título na página:
```
# Page title
st.title("bnaGPT - Chat Simples")
```
5. Inicializar a variável da sessão "messages" que guardará as mensagens trocadas. FYI: variáveis de sessão são apagadas quando você
reiniciar seu app
```
# to save and output the messages
if "messages" not in st.session_state:
    st.session_state.messages = []
```
6. Imprimir todas as mensagens guardadas em st.session_state.messages caso queira iniciar com alguma mensagem padrão.
```
for message in st.session_state.messages:
    with st.chat_message(message["role"]):
        st.markdown(message["content"])
```
7. Implementar o que acontece quando o user manda uma mensagem:

    7.1 Adicionar nova mensagem a "messages"
    ```
    # receving the input from the user
    if prompt := st.chat_input("What is up?"):
        st.session_state.messages.append({"role": "user", "content": prompt})
    ```
    7.2 Imprimir a mensagem do usuário
    ```
    # printing the user message
    with st.chat_message("user"):
        st.markdown(prompt)
    ```
    7.3 Criar e imprimir a resposta do chatbot
    ```
    # now creating the assistant response
    with st.chat_message("assistant"):

        # using groq to get the response
        stream = client.chat.completions.create(
            model=MODEL_NAME,
            messages=[{"role": "user", "content": prompt}],
            # setting stream as True to get the response little by little
            # set as False if you want response at once
            stream=True,
        )
        response = st.write_stream(parse_groq_stream(stream))
    st.session_state.messages.append({"role": "assistant", "content": response})
    ```
8. Rodar em seu terminal:
```
streamlit run {nome_do_arquivo}
```

Pronto :D

Só lembrando que o código completo está na pasta 101s do nosso repositório:
https://github.com/bna-dev-public/bnaGPT

### Passo a passo para devs

Você que pulou para essa sessão já deve ter provavelmente alguma experiência com
programação e sabe que conforme o projeto cresce, a complexidade de organização do
código também aumenta. Por isso, teremos outra abordagem para que nosso projeto fique
mais fácil de adicionar _features_ futuramente.

Vai parecer uma bazuca em uma formiga, mas acreditem, vai valer a pena.

Vamos dividir o projeto usando a arquitetura Model-View-Controller
(MVC - [um bom exemplo em Python aqui](https://realpython.com/lego-model-view-controller-python/)). Isso significa:
- Controller -> as _features_ do chatbot que interpretam o que o usuário nos manda;
- Model -> os modelos de como o controller pode responder o usuário (ou seja, as mensagens, mas enventualmente serão imagens, códigos...)
- View -> como é feita a interface com o usuário

Desta maneira, o nosso projeto seguirá a seguinte arquitetura:
```
project
│   README.md # explicação do projeto
│   settings.py # algumas configurações globais
│   requeriments.txt
│   SimpleChatbot.py # onde estará a main do nosso chatbot simples
│
└─── controller # onde vão ficar os arquivos com cada uma das features do projeto
│   │   feature1.py
│   │   feature2.py
│
└─── view # onde vão ficar os arquivos relacionados à interface
│   │   interface1.py
│   │   interface2.py
│
└─── models # onde vão ficar os arquivos relacionados ao models
│   │   message.py
│   │   image.py
│
└─── pages # o streamlit obriga a ter esse diretório com todas as outras páginas do projeto
│   │   page1.py
│   │   page2.py
│
└─── utils # diretório para criar classes e funções auxiliares ao projeto, por exemplo:
│   │   llama.py
│
└─── .streamlit # diretório para customizar elementos e os temas do streamlit
    │   config.toml # arquivo de configuração
```

Agora vamos passar pelas as funções mais importantes em detalhes, mas o projeto inteiro estará no [github](https://github.com/bna-dev-public/bnaGPT).

Lembre-se de instalar os requirementos do projetos com:

```
pip install -r requirements.txt
```

1. Construção do view:

    1.1 Criar classe SimpleChatbotView que será inicializada com as variáveis da página:

    ```
    from dataclasses import dataclass, field
    import streamlit as st
    from utils.loaders import load_image
    from controller.simple_chatbot import SimpleChatEngine
    from models.messages import StreamingMessage, Message


    chat_engine = SimpleChatEngine()


    # This class represents a simple chatbot in Python.
    @dataclass(slots=True)
    class SimpleChatbotView:

        # default values for the page
        page_title: str = field(default="bnaGPT")
        page_icon: str = field(default=":desktop_computer:")
        logo_filename: str = field(default="static/images/logo.png")
        header_string: str = field(default="bnaGPT | Simple Chat")
        text_box_string: str = field(default="Talk to our agent and see how it works!")
    ```

    > Veja que para a implementação do view precisamos do método load_image (somente para carregar uma página) e das classes
    SimpleChatEngine (o nosso controller da página) e StreamingMessage/Message (nossos models que vão ser
    usados em todas as páginas provavelmente).

    1.2. Métodos para setar configuração da página (build_config), renderizar o body/sidebar
    (build_body e build_sidebar), inicializar as váriaveis da sessão (initialize_session) e o handler do
    evento de mandar uma mensagem (handle_chat_input):

    ```
    # build the page configuration
    def build_config(self):
        st.set_page_config(
            page_title=self.page_title,
            page_icon=self.page_icon,
        )

    # build the page body
    def build_body(self):
        # build the header
        st.header(self.header_string)
        # build the chat input
        chat_input = st.chat_input("What is up?")
        return chat_input

    # build the page sidebar
    def build_sidebar(self):
        with st.sidebar:
            image = load_image(self.logo_filename)
            st.logo(image)

    # Streamlit has a session element that you can create and save variables in it
    # So we need to update it once our main runs in a loop
    def initialize_session(self):

        # we can create a variable called messages
        # to save and output the messages
        if "messages" not in st.session_state:
            st.session_state.messages = []

        # output messages where:
        # role = user or assistant
        # content = content of the message
        for message in st.session_state.messages:
            with st.chat_message(message["role"]):
                st.markdown(message["content"])

    # get user input
    def handle_chat_input(self, chat_input):
        if prompt := chat_input:
            message = Message("user", prompt)
            st.session_state.messages.append(message.create_message())

            # printing the user message
            with st.chat_message("user"):
                st.markdown(prompt)

            # now creating the assistant response
            with st.chat_message("assistant"):
                # getting the streaming response from controller
                streaming_response = StreamingMessage(
                    chat_engine.get_simple_chat_response(prompt)
                )
                # printing response
                response = st.write_stream(streaming_response.stream_message())
            # saving response
            message = Message("assistant", str(response))
            st.session_state.messages.append({"role": "assistant", "content": response})

    ```

2. Construir o controller para realizar a consulta no Groq e mandar a resposta da sua pergunta. Por enquanto
é simples porque só temos uma funcionalidade, responder uma pergunta (SimpleChatEngine).

```
from dataclasses import dataclass, field
from utils.llama import Llama


@dataclass(slots=True)
class SimpleChatEngine:
    llama: Llama = field(default_factory=Llama)

    def get_simple_chat_response(self, user_input):
        stream = self.llama.simple_chat(user_input)
        return stream
```

3. Construir a interface Controller <> Groq (ou outros
API providers). Para isso vamos usar LlamaIndex.

    LlamaIndex é um framework para aplicações de LLM, assim como a Langchain.

    Por que LlamaIndex?

    Nossa opinião LangChain vs LlamaIndex é a seguinte: LangChain é muito abstrato
    e verborrágica (muitas linhas para poucas coisas). Além disso, como vamos depois construir
    um RAG (geração aumentada de recuperação - uma técnica para aumentar o contexto passado para a
    LLM - que vamos nos aprofundar nos próximos capítulos) para conversar com documentos, imagens e etc.. o Llama tem muitas coisas
    legais para tudo isso;

    Para um simples chat, a API de qualquer modelo serviria. Mas aqui estamos olhando
    para o longo prazo do projeto e vamos usar n diferentes modelos de n diferentes APIs,
    então usar uma lib como LlamaIndex ajuda bastante quando formos fazer mudanças.

    3.1. Construção da classe Llama:

    ```
    from dataclasses import dataclass, field
    from llama_index.llms.groq import Groq
    from llama_index.core.chat_engine import SimpleChatEngine
    from settings import DEFAULT_LLM_NAME, DEFAULT_LLM_PROVIDER, GROQ_API_KEY
    from view.general import print_error


    @dataclass(slots=True)
    class Llama:
        llm: dict = field(
            default_factory=lambda: {
                "llm_provider": DEFAULT_LLM_PROVIDER,
                "llm_name": DEFAULT_LLM_NAME,
            }
        )
    ```

    3.2. Método para instanciar uma LLM:

    ```
    def create_llm(self):
        if self.llm.get("llm_provider") == "groq":
            try:
                llm = Groq(model=self.llm.get("llm_name", ""), api_key=GROQ_API_KEY)
            except Exception as e:
                print_error("Error to create the llm: " + str(e))

        else:
            print_error("We only accept groq models right now")

        return llm
    ```

    3.3. Método para instanciar uma chat engine (abstração do Llama para gerenciar o chat) com a LLM:
    ```
    def create_simple_chat_engine(self):
        llm = self.create_llm()
        simple_chat_engine = SimpleChatEngine.from_defaults(llm=llm)
        return simple_chat_engine
    ```

    3.4. Método para fazer uma pergunta ao chat usando a chat engine criada:
    ```
    def simple_chat(self, user_input):
        simple_chat_engine = self.create_simple_chat_engine()
        stream = simple_chat_engine.stream_chat(user_input)
        return stream
    ```

4. Construção dos models.
    4.1. Classe Message para padronizar mensagens entre AI e user:
    ```
    from dataclasses import dataclass
    from llama_index.core.chat_engine.types import StreamingAgentChatResponse


    @dataclass(slots=True)
    class Message:
        role: str
        content: str

        def create_message(self):
            message = {"role": self.role, "content": self.content}
            return message
    ```

    4.2. Classe StreamingMessage para padronizar mensagens feitas com streaming:
    ```
    @dataclass(slots=True)
    class StreamingMessage:
        streaming_response: StreamingAgentChatResponse

        def stream_message(self):
            for token in self.streaming_response.response_gen:
                yield token + ""
    ```

5. Adicionar alguns arquivos secundários de auxílio como:
    - utils/loaders.py (para carregar imagens)
    - view/general.py (funcionalidades gerais do front-end,
    por exemplo: mandar mensagens de erro)
    - settings.py com variáveis globais

6. Arquivos estáticos no diretório `static`

7. Finalmente o arquivo principal da página: Simple_Chatbot.py:
```
from view.simple_chatbot import SimpleChatbotView


# main function of the Simple Chatbot
def main():
    sc = SimpleChatbotView()
    sc.build_config()
    chat_input = sc.build_body()
    sc.build_sidebar()
    sc.initialize_session()
    sc.handle_chat_input(chat_input)


if __name__ == "__main__":
    main()
```

8. Rode em seu terminal:
```
streamlit run {Simple_Chatbot.py}
```


#### Resultado

E assim você completou o passo 0!

![models_quality](/images/bnaGPT_1.gif)
_bnaGPT versão 0_