---
title: "From Zero to bnaGPT (Part 1)"
meta_title: "How to Build Your Own ChatGPT from Scratch - Step-by-Step Guide"
description: "Article teaching how to create a ChatGPT from scratch"
reading_time: "5 min"
medium_link: ""
date: 2024-06-17T05:00:00Z
categories: ["LLM"]
author: "Gabriel Pinto"
tags: ["Coding"]
draft: false
---

Part 1 of our bnaGPT. Teaching you how to build a ChatGPT from scratch!

## Part 1

In part 0, we made a basic chat and the UI for our bnaGPT.

Today we will continue to add more functionalities. We will focus on:
- Memory
- Saving conversations

List of functionalities so far:

- [x] Chat = you send a message, and it responds :sleeping:
- [ ] Memory = you send multiple messages, and it responds considering past messages :sleeping:
- [ ] Save conversations = you can reopen past conversations and restart them :unamused:
- [ ] Search the internet = it searches your question on the internet before responding :confused:
- [ ] Read documents = it reads a document with context about your question (or not, who knows what you send) before responding :neutral_face:
- [ ] Read images = you send a message with a document containing images or just an image, and it responds :relieved:
- [ ] Run code = the agent can run some code it created :grinning:
- [ ] GPT/assistant = you can create GPTs for specific functions :anguished:
- [ ] Generate images = it creates images from some context :open_mouth:
- [ ] Receive audio/generate audio = it understands your audio and responds in audio :scream:
- [ ] Wash your dishes :skull:

- [ ] Bonus: JSON mode = it responds only in JSON (this is very helpful if you are a dev using AI in some product).

## Part 1 - Memory and Saving Conversations

### Step-by-Step for Beginners

In this section, we will document the code changes to add these new features in a simpler way.

1. Function to save conversation -> here you should designate a JSON filename to save the conversations and save the name in the `CONVERSATIONS_FILE` variable

Example:
```python
CONVERSATIONS_FILE = "conversations.json"
```

Code to save a conversation in the created file.

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

2. Function to list all saved conversations:

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

3. Function to load a specific conversation:
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

4. Initialize the new session variables used:
    - show_confirm_save: variable initialized as False and becomes True when the user clicks to save a new conversation and a textbox appears for the user to name this conversation and confirm the save;
    ```python
    # show_confirm_save = variable that will store whether the button of
    # confirm save should be seen
    if "show_confirm_save" not in st.session_state:
        st.session_state.show_confirm_save = False
    ```
    - saved_conversations: variable with all conversations already saved in the JSON file. It is initialized with the conversations saved by the function list_conversation()
    ```python
    # saved_conversations = variable to store all conversations
    if "saved_conversations" not in st.session_state:
        st.session_state.saved_conversations = list_conversations()
    ```

5. Add UI elements in the sidebar. The with st.sidebar: command ensures all elements created within this block are placed in the sidebar:
    - Button to save the current conversation
    ```python
    # Sidebar to load saved conversations
    with st.sidebar:
        if st.button("Save Current Conversation"):
            st.session_state.show_confirm_save = True
    ```
    - What happens when the save conversation button is clicked -> a textbox and confirm save button appear
    ```python
        if st.session_state.show_confirm_save:
            save_convo_name = st.text_input("Enter a name to save the conversation:")
            if st.button("Confirm Save") and save_convo_name:
                save_conversation(save_convo_name)
                st.session_state.show_confirm_save = False
    ```
    - List saved conversations
    ```python
        # UI for saving a conversation
        st.subheader("Saved Conversations")
        for convo_name in st.session_state.saved_conversations:
            if st.sidebar.button(convo_name):
                load_conversation(convo_name)
    ```
6. Modify the previous Groq chat function to consider all messages already sent in the conversation:
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

Final code in our [repository.](https://github.com/bna-dev-public/bnaGPT/blob/main/101s/chatbot_with_memory.py)

### Step-by-Step for Devs

Now let's create the step-by-step guide for our project for devs. Remember we are using an MVC (model-view-controller) architecture, meaning we have separate interfaces and files for:

- model -> how we structure our data = our messages
- view -> how we structure our front-end = streamlit functions
- controller -> how we structure our back-end = interface with llama-index, LLMs, embedding models, and vector bases (soon).

1. Pull from our [repository](https://github.com/bna-dev-public/bnaGPT) :)

Now all the changes we made:

2. settings.py: added a CONVERSATIONS_FILE variable and updated the LLM variables to simplify a bit
3. view/chatbot_with_memory.py: entire interface for saving, loading, and deleting saved conversations.
    - new sidebar with save and confirm new save buttons
    - list all conversations with options to load or delete the saved conversation
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
    - new session variable initialization
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
    - modify chatbot interface. Now we will use an agent (I will go into details when we get into the controller modifications)
    ```python
    # now creating the assistant response
    with st.chat_message(MessageRole.ASSISTANT):
        # getting the streaming response from controller
        streaming_response = StreamingMessage(
            st.session_state.agent.stream_chat(chat_input)
        )
    ```
4. Changes in utils/llama.py for new controller functions
    - create an openAI agent. Even though the class is called OpenAIAgent, it is actually an implementation of Llama-Index based on how the OpenAI agent works. This means: it has chat with memory functionality and supports various functions that we can implement (e.g., making database queries, interacting with APIs, performing calculations, running code). For now, we will have only memory, and in the next articles, we will add more things.
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
5. New controller for the new page in controller/chatbot_with_memory.py
    - create memory from past messages
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
    - create the OpenAI agent
    ```python
    def create_openAI_agent(
        self, memory: ChatMemoryBuffer = ChatMemoryBuffer.from_defaults()
    ) -> OpenAIAgent:
        openAI_agent = self.llama.create_openAI_agent(memory)
        return openAI_agent
    ```
    - chat with the agent:
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
    - functions to save, delete, read, and list saved conversations
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
6. New model to format messages to work with the agent:
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
7. Create new page with new implementation in pages/1_Chatbot_With_Memory.py
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

## Conclusion

Now you have a free local bnaGPT to use whenever you want! Still without all the functionalities of ChatGPT, but 90% of the features are already there!