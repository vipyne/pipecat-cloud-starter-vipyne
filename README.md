# 🤖 My Pipecat Cloud Starter Project

[![Docs](https://img.shields.io/badge/Documentation-blue)](https://docs.pipecat.daily.co) [![Discord](https://img.shields.io/discord/1217145424381743145)](https://discord.gg/dailyco)

A template voice agent for [Pipecat Cloud](https://www.daily.co/products/pipecat-cloud/) that demonstrates building and deploying a conversational AI agent.

## Prerequisites

- Python 3.10+
- Linux, MacOS
- [Docker](https://www.docker.com) and a Docker repository (e.g., [Docker Hub](https://hub.docker.com))
- A Docker Hub account (or other container registry account)
- [Pipecat Cloud](https://pipecat.daily.co) account

> **Note**: If you haven't installed Docker yet, follow the official installation guides for your platform ([Linux](https://docs.docker.com/engine/install/), [Mac](https://docs.docker.com/desktop/setup/install/mac-install/), [Windows](https://docs.docker.com/desktop/setup/install/windows-install/)). For Docker Hub, [create a free account](https://hub.docker.com/signup) and log in via terminal with `docker login`.

## Getting Started

### 1. Clone this repository

```bash
git clone https://github.com/daily-co/pipecat-cloud-starter
cd pipecat-cloud-starter
```

### 2. Set up Python environment
We *highly* recommend using a virtual environment to manage your Python dependencies.

```bash
python3.12 -m venv venv
source venv/bin/activate
pip install -r requirements.txt
pip install pipecatcloud
```

- just checking
```bash
pcc --help
```

### 3. Authenticate with Pipecat Cloud

```bash
pcc auth login
```

### 4. Acquire required API keys

This starter requires the following API keys:

- **OpenAI API Key**: Get from [platform.openai.com/api-keys](https://platform.openai.com/api-keys)
- **Cartesia API Key**: Get from [play.cartesia.ai/keys](https://play.cartesia.ai/keys)
- **Daily API Key**: Automatically provided through your Pipecat Cloud account

## Deploy and run

### 1. export your variables for convenience
```bash
# docker related variables
export PCC_DOCKER_REPOSITORY="<my-docker-repo>"
export PCC_DOCKER_USERNAME="${PCC_DOCKER_REPOSITORY}"
export PCC_IMAGE_CREDENTIALS="${PCC_DOCKER_REPOSITORY}"
export PCC_IMAGE_VERSION="0.1"

# custom strings (can be whatever you like)
export PCC_AGENT_NAME="my-first-agent"
export PCC_PULL_SECRET="my-first-pull-secret"
export PCC_SECRET_SET="my-first-secret-set"
export PCC_ORG_KEY="my-first-organization-key"
```

### 2. setup secrets

#### a. voice ai agent secrets
```bash
# Copy the example env file
cp example.env .env

# CARTESIA_API_KEY="${CARTESIA_API_KEY}"
# DAILY_API_KEY="${DAILY_API_KEY}"
# OPENAI_API_KEY="${OPENAI_API_KEY}"

# Create a secret set from your .env file
pcc secrets set "${PCC_SECRET_SET}" --file .env
```

#### b. docker secret (only necessary if docker image is private)
- if docker repo image is private:
```bash
# set docker image pull secret
pcc secrets image-pull-secret "${PCC_PULL_SECRET}" https://index.docker.io/v1/
```
then pass in your docker username and password.

> for example:
> ```bash
> $ pipecat secrets image-pull-secret my-pull-secret https://index.docker.io/v1/
> ? Username for image repository 'https://index.docker.io/v1/' ${PCC_DOCKER_USERNAME}
> ? Password for image repository 'https://index.docker.io/v1/' *********************
> ```

#### c. check secrets
> secret values will not be shown
```bash
pcc secrets list
```

> important!
update `secret_set` name in `pcc-deploy.toml`.  It must be a string literal (not an env var)

### 3. Test with a local run of agent
- check config
```bash
pipecat --config
```

- check that things are working as expected before deploying
- this will open a browser window
```bash
source .env
LOCAL_RUN=1 python bot.py
```

> Reminder: `DAILY_API_KEY` value can be found at [https://pipecat.daily.co](https://pipecat.daily.co) Under the `Settings` menu of your agent, in the `Daily` tab.

## Deploy & Run
### 1. Build and push your Docker image

> ensure Docker is running. also, this may take a minute aka coffee break opportunity.

```bash
docker build --platform linux/arm64 -t "${PCC_AGENT_NAME}" .

docker tag "${PCC_AGENT_NAME}":latest \
"${PCC_DOCKER_REPOSITORY}/${PCC_AGENT_NAME}:${PCC_IMAGE_VERSION}"

docker push "${PCC_DOCKER_REPOSITORY}/${PCC_AGENT_NAME}:${PCC_IMAGE_VERSION}"
```

#### 2. deploy
```bash
pcc deploy "${PCC_AGENT_NAME}" \
"${PCC_DOCKER_REPOSITORY}/${PCC_AGENT_NAME}:${PCC_IMAGE_VERSION}" \
--credentials "${PCC_PULL_SECRET}"
```

- for example:
```bash
$ pipecat deploy my-first-agent \
dockerhub_name/my-first-agent:0.1 \
--credentials my-pull-secret
```

#### 3. Prepare to run your agent
```bash
pcc agent logs "${PCC_AGENT_NAME}"
pcc agent status "${PCC_AGENT_NAME}"

pcc organizations keys create
pcc organizations keys use
```

- for example:
```bash
pcc organizations keys create
? Enter human readable name for API key e.g. 'Pipecat Key'
=> my-first-organization-key
pcc organizations keys use
```

### 4. Start your agent

```bash
# Start a session with your agent in a Daily room
pcc agent start "${PCC_AGENT_NAME}" --use-daily
```

- monitor, check logs
```bash
pcc agent status "${PCC_AGENT_NAME}"
pcc agent logs "${PCC_AGENT_NAME}"
```

### 5. Delete your agent
- assuming you are just testing things out. leaving an agent deployed incurs $$.
```bash
pcc agent delete "${PCC_AGENT_NAME}"
```

## Documentation

For more details on Pipecat Cloud and its capabilities:

- [Pipecat Cloud Documentation](https://docs.pipecat.daily.co)
- [Pipecat Project Documentation](https://docs.pipecat.ai)
- [Pipecat Repo](https://github.com/pipecat-ai/pipecat)

## Support

Join our [Discord community](https://discord.gg/dailyco) for help and discussions.
