---
title: "From Zero to bnaGPT (Part 0)"
meta_title: "How to Build Your Own ChatGPT from Scratch - Step-by-Step Guide"
description: "Article teaching how to create a ChatGPT from scratch"
reading_time: "5"
date: 2024-06-03T05:00:00Z
# image: "/images/image-placeholder.png"
categories: ["LLM"]
author: "Gabriel Pinto"
tags: ["Coding"]
draft: false
---

A new series by bna.dev has started, where we will teach you how to create your own ChatGPT!

## From Zero to bnaGPT

As mentioned, the idea of this series is to help you build a ChatGPT for yourself.

"Okay, but why would I do that?"

1. Thirst for knowledge, masters;
2. We've already spent a lot of time on YouTube and on ChatGPT itself trying to understand the best ways or practices to create a chat from scratch;
3. Better understand what ChatGPT is, how it works, and what you can replicate in other places;
4. It can be fun depending on the choices you've made in your life;
5. To have a cheaper and faster version of ChatGPT (hopefully).

The idea here is, in each "episode," to build one of ChatGPT's functionalities until we actually have a bnaGPT as good as or better than ChatGPT.

## Requirements

In general, to replicate what we will do here, you will need the following:
1. A computer or at least access to Google Colab or some other tool that allows you to run Python code;
2. Install Python on your machine (it's easy to do -> you can follow a tutorial on YouTube -> eventually, we will have an article on how to set up a machine for Python as well). Feel free to ask us any questions;
3. Basic knowledge of some programming language, but at this point, there's a world where you can manage with ChatGPT as well;
4. A credit card to access (not everyone will have to pay) some of the tools we will use;
5. Determination!

Other things that can help you but are not mandatory:
1. Knowing how to use Git/Github to clone our repositories;
2. Docker if you want to deploy the chat on a website;
3. Streamlit library if you want to change the layout we created.

## ChatGPT Features We Will Replicate
ChatGPT can be summarized as a UI with various AI features (not just Large Language Models). These are:

- [ ] Chat = you send a message and it responds :sleeping:
- [ ] Memory = you send multiple messages and it responds considering previous messages :sleeping:
- [ ] Save Conversations = you can reopen past conversations and continue them :unamused:
- [ ] Search the Internet = it searches your question on the internet before responding :confused:
- [ ] Read Documents = it reads a document with context about your question (or not, I don't know what you send) before responding :neutral_face:
- [ ] Read Images = you send a message with a document containing images or an image and it responds :relieved:
- [ ] Run Code = the agent can run some code it created itself :grinning:
- [ ] GPT/Assistant = you can create GPTs for specific functions :anguished:
- [ ] Generate Images = it creates images based on some context :open_mouth:
- [ ] Receive Audio/Generate Audio = it understands your audios and responds in audio :scream:
- [ ] Washes Your Dishes :skull:

Bonus: JSON mode = responds only in JSON (this is very helpful if you are a dev and using AI in some product).

Let us know if there are any features missing!

## Part 0 - **_Aesthetics_** and Simple Chat

Let's go!

In this series, we will try to divide into two codes/projects as much as possible:
- Beginner: here we will try to keep everything within a single code. It may become unfeasible after some features, but at least this way you will have a single file to run and it will work;
- Devs: here is the creation of a project thinking a little more long-term, using MVC architecture (we will explain), so that it is a clean code, easy to maintain, and easy to implement improvements.

Python installed? Can you run a hello world?

For those interested, all codes will be here: https://github.com/bna-dev-public/bnaGPT. In the 101s folder, we have the files for beginners and everything else is for "devs."

### Libraries and Tools Used

#### **_Aesthetics_**

We will use the Streamlit library.

Why Streamlit?

In our humble opinion, it is the library where you can most easily achieve two main objectives:
1. Ease of creation;
2. Your site won't look like a Windows 98 application.

There are other options, such as:
1. Django -> don't do this to yourself, we've done it to ourselves. It is an architecture that requires many files, so even for simple things (in our case) you will have to create/write a considerable number of files;
2. Shiny -> it is a good alternative, but your site will look like Windows 98 unless you spend a lot of time adjusting the **_Aesthetics_** of your page, and we don't want that. Or if you are going to use it for Data Science projects that need charts;
3. Bokeh -> same as Shiny;
4. Solara -> very similar to Streamlit, I think it's worth a try. However, it is a bit more verbose and still doesn't have much support or a large community, so it's hard to find good examples that apply to what you need.

There are some others, but they will fit into one of the two problems. If you have one that you particularly think fits well, let us know! (we might switch to Solara in some other projects).

> **ONLY FOR DEVS:** if you are going to use this in production and want a site running a chatbot or similar things. We would divide it like this:
>
> _Front-end:_ Streamlit if you don't mind using templates and have to use Python, but it would be preferable to use JS frameworks like Vanilla JS, React, Angular. We have struggled a few times trying to make a great front-end using Python.
>
> _Back-end:_ Flask or Django (it irritates me, but it would work) if you prefer using Python, but I would try to use a framework that communicates more with your front, like Node.js.
>
> The choice of JS is more about the ease of finding people who program in JS, but other languages would work well.

#### LLM Provider and LLM Models Used

For now, everything will work with Groq and the model used will be Llama 3 70b.

Why Groq and Llama 3?
- Since the idea is to create a ChatGPT, we want to avoid using OpenAI as much as possible. Only when there is a significant advantage;
- The combination of Groq and Llama 3 is extremely fast and has a quality similar to GPT-4 or GPT-4o;
- See our article [LLMs - 101](https://www.bna.dev.br/blog/post-1/) to understand more.

To make everything work, you only need an API key from Groq:
- **Groq Configuration**. You can create an API key for free on their [website](https://console.groq.com/)

### Step-by-Step for Beginners

1. Install the libraries:

```pip install streamlit groq```

2. Import the libraries:
```
from groq import Groq
import streamlit as st
```

3. Create the main configurations and methods for Groq:

    3.1. Instantiate the client to make requests to Groq:

    Important! For the security of your API key, it is preferable to use environment variables. You can access them with the following:
    ```
    import os
    GROQ_API_KEY = os.environ["GROQ_API_KEY"]
    ```

    ```
    # input you API key here
    GROQ_API_KEY = <YOUR API KEY>
    # Create your Groq client
    client = Groq(
        api_key=GROQ_API_KEY,
    )
    # set the model we're going to use
    MODEL_NAME = "llama3-70b-8192"
    ```

    3.2. Method to use streaming responses in Groq:
    ```
    def parse_groq_stream(stream):
        for chunk in stream:
            if chunk.choices:
                if chunk.choices[0].delta.content is not None:
                    yield chunk.choices[0].delta.content
    ```

4. Set the page title:
```
# Page title
st.title("bnaGPT - Chat Simples")
```
5. Initialize the session variable "messages" that will store the exchanged messages. FYI: session variables are cleared when you restart your app
```
# to save and output the messages
if "messages" not in st.session_state:
    st.session_state.messages = []
```
6. Print all messages stored in st.session_state.messages if you want to start with some default message.
```
for message in st.session_state.messages:
    with st.chat_message(message["role"]):
        st.markdown(message["content"])
```
7. Implement what happens when the user sends a message::

    7.1 Add a new message to "messages"
    ```
    # receving the input from the user
    if prompt := st.chat_input("What is up?"):
        st.session_state.messages.append({"role": "user", "content": prompt})
    ```
    7.2 Print the user's message
    ```
    # printing the user message
    with st.chat_message("user"):
        st.markdown(prompt)
    ```
    7.3 Create and print the chatbot's response
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
8. Run in your terminal:
```
streamlit run {nome_do_arquivo}
```

Done :D

Just a reminder that the complete code is in the 101s folder of our repository::
https://github.com/bna-dev-public/bnaGPT

### Step-by-Step for Devs

If you've jumped to this section, you probably have some programming experience and know that as the project grows, the complexity of organizing the code also increases. Therefore, we will take a different approach to make it easier to add features to our project in the future.

It may seem like overkill, but believe us, it will be worth it.

We will divide the project using the Model-View-Controller architecture (MVC - [a good example in Python here](https://realpython.com/lego-model-view-controller-python/)). This means:
- Controller -> the chatbot features that interpret what the user sends us;
- Model -> the models of how the controller can respond to the user (i.e., the messages, but eventually will be images, codes...)
- View -> how the interface with the user is made

Thus, our project will follow the following architecture:
```
project
│   README.md # project explanation
│   settings.py # some global settings
│   requeriments.txt
│   SimpleChatbot.py # where the main of our simple chatbot will be
│
└─── controller  # where the files with each of the project features will be
│   │   feature1.py
│   │   feature2.py
│
└─── view # where the files related to the interface will be
│   │   interface1.py
│   │   interface2.py
│
└─── models # where the files related to models will be
│   │   message.py
│   │   image.py
│
└─── pages # Streamlit requires this directory with all other project pages
│   │   page1.py
│   │   page2.py
│
└─── utils  # directory to create auxiliary classes and functions for the project, for example:
│   │   llama.py
│
└─── .streamlit # directory to customize elements and themes of Streamlit
    │   config.toml # config file
```

Now let's go through the most important functions in detail, but the entire project will be on [GitHub](https://github.com/bna-dev-public/bnaGPT).

Remember to install the project requirements with:

```
pip install -r requirements.txt
```

1. Build the view:

    1.1 Create the SimpleChatbotView class that will be initialized with the page variables:

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

    > Note that for the view implementation, we need the load_image method (just to load a page) and the SimpleChatEngine classes (our page controller) and StreamingMessage/Message (our models that will probably be used on all pages).

    1.2. Methods to set the page configuration (build_config), render the body/sidebar (build_body and build_sidebar), initialize the session variables (initialize_session), and the event handler for sending a message (handle_chat_input):

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

2. Build the controller to make the request to Groq and send the answer to your question. For now, it is simple because we only have one feature, answering a question (SimpleChatEngine).

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

3. Build the Controller <> Groq interface (or other API providers). For this, we will use LlamaIndex.

    LlamaIndex is a framework for LLM applications, just like LangChain.

    Why LlamaIndex?

    Our opinion LangChain vs LlamaIndex is as follows: LangChain is very abstract and verbose (many lines for few things). Besides, as we will later build a RAG (augmented retrieval generation - a technique to increase the context passed to the LLM - which we will delve into in the next chapters) to interact with documents, images, etc., Llama has many cool features for all of that;

    For a simple chat, any model's API would do. But here we are looking at the long-term project and will use n different models from n different APIs, so using a library like LlamaIndex helps a lot when we make changes.

    3.1. Build the Llama class:

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

    3.2. Method to instantiate an LLM:

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

    3.3. Method to instantiate a chat engine (Llama abstraction to manage the chat) with the LLM:
    ```
    def create_simple_chat_engine(self):
        llm = self.create_llm()
        simple_chat_engine = SimpleChatEngine.from_defaults(llm=llm)
        return simple_chat_engine
    ```

    3.4. Method to ask a question to the chat using the created chat engine:
    ```
    def simple_chat(self, user_input):
        simple_chat_engine = self.create_simple_chat_engine()
        stream = simple_chat_engine.stream_chat(user_input)
        return stream
    ```

4. Build the models
    4.1. Message Class to standardize messages between AI and user:
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

    4.2. StreamingMessage Class to standardize streaming messages:
    ```
    @dataclass(slots=True)
    class StreamingMessage:
        streaming_response: StreamingAgentChatResponse

        def stream_message(self):
            for token in self.streaming_response.response_gen:
                yield token + ""
    ```

5. Add some auxiliary files such as:
    - utils/loaders.py to load images
    - view/general.py -> general front-end functionalities, for example: sending error messages
    - settings.py with some global variables

6. Static archives in the directory `static`.

7. Finally, the main page file: Simple_Chatbot.py:
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

8. Run in your terminal:
```
streamlit run {Simple_Chatbot.py}
```

#### Result

Now you've completed step 0!

![models_quality](/images/bnaGPT_1.gif)
_bnaGPT versão 0_

---