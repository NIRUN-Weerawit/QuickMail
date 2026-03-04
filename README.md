# Required Installations

To replicate this project, the following components must be installed and running locally:

- **Ollama** (local LLM server)
- **n8n** (workflow automation engine)
- **Docker** (recommended for clean setup)

# 🖥 System Requirements

Minimum:

- 16GB RAM recommended
- 10GB free disk space
- Linux / macOS / Windows (WSL)

# Docker Container

1. Install Docker
    
    ```jsx
    sudo apt update
    sudo apt install docker.io docker-compose -y
    sudo systemctl enable docker
    sudo systemctl start docker
    #verify
    docker --version 
    ```
    
2. Create docker image inside a project folder of your preferred name
    
    ```jsx
    mkdir quickmail
    cd quickmail
    nano docker-compose.yml
    ```
    
3. Paste:
    
    ```jsx
    services:
      ollama:
        image: ollama/ollama
        container_name: ollama
        ports:
          - "11434:11434"
        volumes:
          - ollama_data:/root/.ollama
    
      n8n:
        image: n8nio/n8n
        container_name: n8n
        ports:
          - "5678:5678"
        volumes:
          - n8n_data:/home/node/.n8n
        environment:
          - N8N_ENCRYPTION_KEY=supersecretkey123
        depends_on:
          - ollama
    
    volumes:
      ollama_data:
      n8n_data:
    ```
    
4. Run the Docker container 
    
    ```jsx
    docker compose up -d
    ```
    
    Remarks: If error relates to unable to locate ~/.n8n, then create it and grant it permissions:
    
    ```jsx
    mkdir -p ~/.n8n
    sudo chown -R $USER:$USER ~/.n8n
    # or sudo chown -R 1000:1000 ~/.n8n
    ```
    
    Then, run the Docker again
    
5. Pull the LLM model inside the container (this case: llama3)
    
    ```jsx
    docker exec -it ollama ollama pull llama3
    ```
    
    Verify if the model is already inside the container
    
    ```jsx
    docker exec -it ollama ollama list
    # it should return the model's name: llama3
    ```
    
6. Go to the following website to start building the automated workflow
    
    ```jsx
    http://localhost:5678
    ```
    

# 🧠 Architecture Summary

Final working architecture:

```
Docker Network
 ├── ollama (LLM inference server)
 └── n8n (workflow automation)
```

n8n communicates with Ollama internally using:

```
http://ollama:11434/api/generate
```

Next step is to setup all APIs authentication

# Configure Gmail API

1. Go to [Google Cloud Console](https://console.cloud.google.com/welcome?project=gmail-assistant-489104)
2. Create a project
3. Enable `Gmail API`  by : `APIs & Services → Library → Search -> Gmail API -> Enable` 

# Configure OAuth Consent Screen

1. `APIs & Services → OAuth consent screen` 
2. Finish all the neccessary steps
3. `APIs & Services → Credentials` 
4. Click  `Create Credentials → OAuth client ID`
5. Choose `Web Applicaiton` 
6. Choose `External` 
7. In the `Authorized Redirect URI` , if n8n runs locally or Docker, add  

```jsx
http://localhost:5678/rest/oauth2-credential/callback
```

1. Go to `Google Auth Platform → Audience → Publishing Status → Publish`  to make it public

# Use Credentials in n8n

1. Create Gmail Credential
2. Choose `OAuth2`
3. Enter:
    - `Client ID`
    - `Client Secret`
4. Click Connect

Sign in with Google → Approve → Done.

Next, setup the Discord Bot API

Version 1.x - 2.x uses Discord as the main communicator, which requires creating a Discord bot to send messages to a channel. 

# Create Application (a bot)

1. Access https://discord.com/developers/applications
2. Click `New Application`, then name your own bot
3. Go to `OAuth2` section, in `OAuth2 URL Generator`, select `bot` option
4. In `Bot permission`, select all that applies to what you want the bot to access: e.g. `send messages`, `Embed Links`, `Mention Everyone`, etc.
5. Copy the `Generated URL`, then paste it on any browser, which it will direct you to Discord app
6. Select the server you would like this bot to send messages to.

# Establish Credentials for Discord Bot API

1. Add `Discord` node : `Send a message` action in n8n
2. Choose `Connection Type` as `Bot Token`
3. In the `Application` page in `Discord Developer Portal`, go to `Bot` section
4. press `Reset Token`, then copy the token
5. Go back to n8n’s Discord node, for `Credentials for Discord Bot API,` select `Create new credentials`
6. Paste the copied token, then save
7. Copy the `Channel ID` from the Channel you would like to receive the message from (Server ID is the same as Channel ID)

