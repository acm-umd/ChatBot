# Building a Basic chatGPT clone

In this workshop we will walk through the process of building a very basic website with the basic functionality of chatgpt. 
We will build a backend with routing, a frontend for the backend to serve and hook up a basic conversational model to our backend 
to generate the responses, which will be displayed on a simple web page. 

## Table of Contents
1. [Prerequisites]
2. [Project Setup]
   - [Setting up Python environment]
   - [Installing Django]
   - [Creating a Django Project]
   - [Installing Transformers and Pytorch]
3. [Backend Development]
   - [Creating Django Views]
   - [Expanding our view with Transformers]
   - [Improving our View]
4. [Final Notes]
5. [Further Improvements]

---

## Prerequisites

Before starting, ensure you have **Python 3** installed on your machine. You can verify this by running `python3 --version` in your terminal.
- You can install python by visiting the following link: [Python Installation](https://www.python.org/downloads/)
- For windows users, it is advised to use the command prompt instead of powershell
- To create files use `touch [file_name]` on macOS/linux, and `notepad [file_name]` on windows 
  
For more detailed information on these topics, you can refer to the official documentation:
- [Python](https://docs.python.org/3/)
- [Django](https://www.djangoproject.com/)
- [Transformers](https://huggingface.co/transformers/)

---

## Project Setup

### Setting up Python environment
- Now that you have ensured that you have Python installed (`python3 --version`), create a directory called `ai_chatbot`(`mkdir ai_chatbot`) 
- change directory into it with `cd ai_chatbot`.
- Next, create and activate a virtual environment:
   - Virtual environments allow you to isolate your project's dependencies, ensuring they don't conflict with other projects on your machine.
   - Run: `python3 -m venv .venv`. This will create a virtual environment in your current directory under `.venv/`.
   - Activate the virtual environment:
     - On macOS and Linux, use: `source .venv/bin/activate`
     - On Windows, use: `.venv\Scripts\activate` (This may not work in powershell due to security settings)
       - If you are using a non posix compliant shell like fish, you may need to use `source .venv/bin/activate.fish` or something similar 
   
   You should now see the virtual environment activated in your terminal (usually the terminal prompt will start with `(.venv)`).

### Installing Django
- With the virtual environment activated, install **Django** using pip (Python's package manager):
  - Run: `pip install django`
  - This will install Django into your virtual environment, making it available to your project.

### Creating a Django Project
- Now that Django is installed, you can create a new Django project. Run the following command:
  - `django-admin startproject chatbot .`
  - This will create the necessary Django project files in the current directory, including the `manage.py` file and the project directory (`chatbot`).
  - You should see output like the following if you list the contents of your directory(`ls` for macOS/Linux, `dir` for windows):
  ```
   $ ls -al
   drwxr-xr-x   - user 15 Sep 16:34 .venv
   drwxr-xr-x   - user 15 Sep 16:43 chatbot
   .rwxr-xr-x 663 user 15 Sep 16:43 manage.py

  ```
  - We can test if django installed successfully by going to the directory manage.py is in and running `python manage.py runserver`
   - Then we navigate to `http://127.0.0.1:8000/` in our browser
   - If it installed successfully, we should see something like `The install worked successfully! Congratulations!` as a webpage on the site
   - press ctrl + c in the terminal to stop the execution of the webserver
  

### Installing Transformers and Pytorch
- Before proceeding to the AI part, we need to install Hugging Face's **Transformers** library along with **Pytorch**, which we'll use to load a pre-trained model.  
  - Run: `pip install transformers`
  - Run: `pip install torch`
  
  
- At this point, you should have Django, Transformers, and Torch installed. In the next steps, we will integrate these into the Django app and set up the chatbot logic.

---

## Backend Development

### Creating a django app
- Next, we'll start start a new app within the project that will handle the chatbot's functionality:
  - Run: `python manage.py startapp chat`
  - This will create a new app directory called `chat` where the views, models, and templates for your chatbot will reside.
  
- Once the app is created, we'll need to add it to our installed app which is in the django settings file
  - Open `chatbot/settings.py`, find the `INSTALLED_APPS` list, and add `'chat',` to the list. This ensures Django knows about your new app.
  ```python

   INSTALLED_APPS = [
       "chat", # Add this line
       "django.contrib.admin",
       "django.contrib.auth",
       "django.contrib.contenttypes",
       "django.contrib.sessions",
       "django.contrib.messages",
       "django.contrib.staticfiles",
   ]
  ```

### Creating Django Views
- The chatbot’s logic will live inside a **Django view**. A view is simply a Python function or class that handles user input and sends a response to the frontend.
  
- In your `chat` app, open `views.py`. You will create a function that handles user queries and processes them using the AI model.

- The view will:
  - Accept a message from the frontend via an HTTP request.
  - Pass the message to the transformer model to generate a response.
  - Return the AI-generated response to be rendered on the webpage.

- For now, we'll create a simple view that simply returns "Hello World"

```python
# chat/views.py
from django.shortcuts import render
from django.http import HttpResponse


def index(request):
    return HttpResponse("Hello, world!")
```
- To access it in a browser, we need to map it to a URL - and to do this, we need to define a URL configuration. 
- URL configurations are defined inside each Django app, within that app's urls.py.
- This file isn't created by default, so we have to create it under chat/
   - `cd chat`
   - `touch urls.py`

```python
# chat/urls.py
from django.urls import path

from . import views

urlpatterns = [
    path("", views.index, name="index"),
]
```

- The next step is configure the global url configurations file to include this URL configuration
- To do this, navigate to the chatbot directory and edit the urls.py file to the following:
```python

"""
URL configuration for chatbot project.

The `urlpatterns` list routes URLs to views. For more information please see:
    https://docs.djangoproject.com/en/5.1/topics/http/urls/
Examples:
Function views
    1. Add an import:  from my_app import views
    2. Add a URL to urlpatterns:  path('', views.home, name='home')
Class-based views
    1. Add an import:  from other_app.views import Home
    2. Add a URL to urlpatterns:  path('', Home.as_view(), name='home')
Including another URLconf
    1. Import the include() function: from django.urls import include, path
    2. Add a URL to urlpatterns:  path('blog/', include('blog.urls'))
"""
from django.contrib import admin
from django.urls import include, path

urlpatterns = [
    path("admin/", admin.site.urls),
    path("chat/", include("chat.urls")),
]
```

- Now that that is done, run `python manage.py runserver` and go to `127.0.0.1:8000/chat/` in your browser
- You should see "Hello World"

### Expanding our view with Transformers 
- Now we want to expand our view to be able to generate text for us 
- First we should decide on a model to use.
   - There is a tradeoff between performance and accuracy
- I've already picked one out that should run on most people's computers. 
   - Unfortunately this means that the model doesn't offer very detailed or informative responses

- Copy the following into views.py

```python
# chat/views.py
from django.shortcuts import render
from django.http import HttpResponse
from transformers import pipeline

# Load the model only once when the server starts, not on every request.
chat_model = pipeline("text-generation", model="distilgpt2")

def chat_response(request):
    # This allows us to get User Input from the Form we are going to create
    user_message = request.GET.get('message', '')  # Default to empty string if no input is provided.
    
   # We only want to generate a response with our model if the user provides a message
    if user_message:
        ai_response = chat_model(user_message, max_length=50)[0]['generated_text']
    else:
        ai_response = "Please provide a message."

    # Render the response on the page using an HTML template (we will create an HTML template next).
    return render(request, 'chat.html', {'response': ai_response, 'user_message': user_message})
```

- Now that we have removed the "index" function, we have to modify urls.py to route `chat/` to our `chat_response` view
- Modify urls.py to the following
```python
# chat/urls.py
   from django.urls import path

   from . import views

   urlpatterns = [
       path("", views.chat_response, name="chat_response"), # we are changing this line to route to chat_response instead of index
   ]
```

- Now we need to create the template we talked about
   - First we need to create a templates directory inside the chat app:
   - Run: mkdir templates
   - Inside that directory, create an HTML file named chat.html where we will write the HTML code.
   - We'll create a very simple template for our view
   ```html

   <form method="GET" action="">
       <input type="text" name="message" placeholder="Enter your message">
       <button type="submit">Send</button>
   </form>

   {% if response %}
       <p>Chatbot Response: {{ response }}</p>
   {% endif %}
   ```

- Now we should be able to navigate to `127.0.0.1:8000/chat` and converse with our model 

### Improving our view
- Now we should improve our user interface. Here is some code that will do that 
- Our new view:

```python
def chat_response(request):
    # Can be used to store conversation history in the future
    conversation = []

    if request.method == "POST":
        # Get the user's message from the POST request
        user_message = request.POST.get("message")

        # Generate a response using the transformer model
        response = chat_model(user_message)
        bot_message = response[0]["generated_text"]

        # Add the user message and bot response to the conversation history
        conversation = request.session.get("conversation", [])
        conversation.append(("You", user_message))
        conversation.append(("Bot", bot_message))

    return render(request, "chat.html", {"conversation": conversation})
```

- And this is the new template, with some style changes and preliminary support for conversation history
```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Chat Interface</title>
    <style>
        body {
            font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
            background-color: #f7f7f8;
            margin: 0;
            padding: 0;
            display: flex;
            justify-content: center;
            align-items: center;
            height: 100vh;
            background-color: #e5e5e5;
        }

        #chat-container {
            display: flex;
            flex-direction: column;
            justify-content: space-between;
            width: 60%;
            max-width: 800px;
            height: 90vh;
            background-color: #ffffff;
            border-radius: 15px;
            box-shadow: 0 4px 12px rgba(0, 0, 0, 0.1);
        }

        #messages {
            flex-grow: 1;
            padding: 20px;
            overflow-y: auto;
            background-color: #f7f7f7;
        }

        .message {
            margin: 10px 0;
            padding: 12px;
            border-radius: 16px;
            max-width: 75%;
            word-wrap: break-word;
            display: inline-block;
        }

        .user-message {
            background-color: #007bff;
            color: white;
            align-self: flex-end;
            text-align: right;
            margin-left: auto;
        }

        .bot-message {
            background-color: #f1f1f1;
            color: black;
            text-align: left;
            margin-right: auto;
        }

        #user-input {
            padding: 20px;
            background-color: #ffffff;
            border-top: 1px solid #e0e0e0;
            display: flex;
            justify-content: center;
            align-items: center;
        }

        #input-form {
            width: 100%;
            display: flex;
            align-items: center;
            background-color: #f7f7f7;
            border-radius: 30px;
            padding: 10px 15px;
            border: 1px solid #ddd;
        }

        #input-form input[type="text"] {
            flex-grow: 1;
            border: none;
            padding: 10px;
            font-size: 16px;
            border-radius: 30px;
            background-color: #fff;
            outline: none;
        }

        #input-form input[type="text"]::placeholder {
            color: #aaa;
        }

        #input-form button {
            background-color: #007bff;
            color: white;
            border: none;
            padding: 10px 20px;
            border-radius: 30px;
            font-size: 16px;
            cursor: pointer;
            margin-left: 10px;
        }

        #input-form button:hover {
            background-color: #0056b3;
        }

        /* Chat Bubble Alignment */
        .message-wrapper {
            display: flex;
        }

        .user-message-wrapper {
            justify-content: flex-end;
        }

        .bot-message-wrapper {
            justify-content: flex-start;
        }

    </style>
</head>
<body>
    <div id="chat-container">
        <div id="messages">
            {% if conversation %}
                {% for speaker, message in conversation %}
                    <div class="message-wrapper {% if speaker == 'You' %}user-message-wrapper{% else %}bot-message-wrapper{% endif %}">
                        <div class="message {% if speaker == 'You' %}user-message{% else %}bot-message{% endif %}">
                            <strong>{{ speaker }}:</strong> {{ message }}
                        </div>
                    </div>
                {% endfor %}
            {% else %}
                <p>No conversation yet. Start by typing a message!</p>
            {% endif %}
        </div>

        <div id="user-input">
            <form id="input-form" method="post" action="{% url 'chat_response' %}">
                {% csrf_token %}
                <input type="text" name="message" placeholder="Enter your prompt" required />
                <button type="submit">Submit</button>
            </form>
        </div>
    </div>
</body>
</html>
```

## Final Notes

At this point, you have built a basic AI chatbot using a transformer model, Django backend, and Django templates for the frontend. While this is a simple implementation, it provides the foundation for more complex chatbots and AI-driven web applications.

You can use this as a launch off point to build a similar website on your own or add AI integration to any website you want to build

---

## Further Improvements

Here are some ideas to extend the functionality of what we have built:

1. **Implement User Authentication**:
   - Add a login system using Django’s built-in authentication framework.
   This allows users to sign in and interact with the chatbot under their accounts, saving their chat history.

2. **Save Chat History**:
   - Store user queries and responses in a database so users can review their past interactions with the chatbot.

3. **Deploying to the Cloud**:
   - Once the chatbot is complete, you can deploy it to a cloud service like Heroku or AWS to make it accessible online. You’ll need to set up a production environment, including a proper database and static file handling.
   - This would also allow you to use more powerful models as they could scale with the hardware that is availible

4. **Frontend Enhancements**:
   - You can enhance the frontend experience by integrating a modern JavaScript framework like React or Vue.js 
   - This would allow you to create a more dynamic experience and enhance functionality

If desired, the ACM can have another workshop that can cover some of these improvements
