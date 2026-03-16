# 🌤️ Bot do Clima — Telegram + N8N + OpenWeatherMap

Chatbot no Telegram desenvolvido com **N8N** que informa a temperatura atual de qualquer cidade do Brasil em tempo real, consultando a API do [OpenWeatherMap](https://openweathermap.org/).

---

## 📋 Visão Geral

O usuário envia o nome de uma cidade no formato `Cidade,UF` diretamente no Telegram. O bot consulta a API do OpenWeatherMap e responde com a temperatura atual de forma clara e amigável.

**Exemplo de interação:**

```
Usuário:  São Paulo,SP
Bot:      🌤️ A temperatura em São Paulo é de 28°C.

Usuário:  Grifnória
Bot:      ❌ Cidade não encontrada. Use o formato Cidade,UF (ex.: São Paulo,SP).
```

---

## 🏗️ Arquitetura do Workflow

```
Telegram Trigger
      │
      ▼
Formatar Entrada         ← captura chatId e normaliza o texto para "Cidade,UF,BR"
      │
      ▼
OpenWeather API          ← GET /data/2.5/weather?q=...&units=metric&lang=pt_br&appid=...
      │
      ▼
Cidade Encontrada?       ← IF: cod == 200
     ╱ ╲
   Sim   Não
    │      │
    ▼      ▼
Formatar  Telegram: ❌ Cidade não encontrada...
Mensagem
    │
    ▼
Telegram: 🌤️ A temperatura em X é de Y°C.
```

---

## 🔧 Pré-requisitos

- Conta no [N8N](https://n8n.io/) (self-hosted ou cloud)
- Chave de API gratuita do [OpenWeatherMap](https://openweathermap.org/api)
- Bot no Telegram criado via [@BotFather](https://t.me/botfather)

---

## 🔑 Variáveis e Credenciais Necessárias

| Variável | Onde obter | Como usar no N8N |
|---|---|---|
| `TELEGRAM_BOT_TOKEN` | [@BotFather](https://t.me/botfather) → `/newbot` | Credencial do tipo **Telegram API** |
| `OPENWEATHER_API_KEY` | [openweathermap.org/api](https://openweathermap.org/api) → sua conta | Query param `appid` no nó HTTP Request |

Obs: Como o n8n injeta a credencial do OpenWeatherMap — ele pode estar enviando a chave como header em vez de query param appid, e a API rejeita por isso. Neste caso jeito o mais seguro é não usar a credencial predefinida do OpenWeatherMap e colocar o appid diretamente na query string.

---

## 📥 Passo a Passo: Importar o Workflow no N8N

### 1. Baixe o arquivo do workflow

Faça o download do arquivo [`Bot do Clima - Telegram-fred.json`](./Bot do Clima - Telegram-fred.json) deste repositório.

### 2. Importe no N8N

1. Acesse sua instância do N8N
2. No menu lateral, clique em **Workflows**
3. Clique no botão **⋮** (três pontos) no canto superior direito
4. Selecione **Import from file**
5. Escolha o arquivo `Bot do Clima - Telegram-fred.json`
6. O workflow será importado com todos os nós e conexões configurados

---

## 🔐 Configurando as Credenciais

### Telegram Bot Token (`TELEGRAM_BOT_TOKEN`)

1. Abra o Telegram e converse com [@BotFather](https://t.me/botfather)
2. Digite `/newbot` e siga as instruções para criar seu bot
3. Copie o **token** fornecido (formato: `123456789:ABCDefgh...`)
4. No N8N, vá em **Settings → Credentials → Add Credential**
5. Selecione o tipo **Telegram API**
6. Cole o token no campo **Access Token**
7. Salve com o nome `BotFather - Weather API FRED` (ou o nome que preferir)
8. Nos nós **Telegram Trigger**, **Telegram - Enviar Temperatura** e **Telegram - Enviar Erro**, selecione essa credencial

Obs: O bot criado neste exercício está disponível em t.me/weather_fred_bot

### OpenWeather API Key (`OPENWEATHER_API_KEY`)

1. Crie uma conta em [openweathermap.org](https://openweathermap.org/)
2. Acesse **API Keys** no seu perfil e copie sua chave
3. No workflow importado, clique no nó **OpenWeather API**
4. Em **Query Parameters**, localize o parâmetro `appid`
5. Substitua o valor pelo sua chave: `SUA_OPENWEATHER_API_KEY`
6. Salve o nó

> ⚠️ **Nota:** A chave gratuita do OpenWeatherMap pode levar até 2 horas para ser ativada após o cadastro.

---

## ▶️ Executando o Chatbot

### Ativar o workflow

1. Com o workflow aberto no N8N, clique no toggle **Inactive** no canto superior direito
2. O status mudará para **Active** — o bot estará escutando mensagens

### Testar o bot

1. Abra o Telegram e encontre o seu bot pelo nome que definiu no BotFather
2. Inicie uma conversa clicando em **Start**
3. Envie uma mensagem no formato `Cidade,UF`:

| Mensagem enviada | Resposta esperada |
|---|---|
| `Curitiba,PR` | `🌤️ A temperatura em Curitiba é de 18°C.` |
| `São Paulo,SP` | `🌤️ A temperatura em São Paulo é de 28°C.` |
| `Salvador,BA` | `🌤️ A temperatura em Salvador é de 30°C.` |
| `Grifnória` | `❌ Cidade não encontrada. Use o formato Cidade,UF (ex.: São Paulo,SP).` |

---

## 🛠️ Estrutura dos Nós

| Nó | Tipo | Função |
|---|---|---|
| **Telegram Trigger** | `telegramTrigger` | Recebe mensagens de texto do usuário |
| **Formatar Entrada** | `set` | Normaliza o texto e extrai o `chatId` |
| **OpenWeather API** | `httpRequest` | Consulta a temperatura via API REST |
| **Cidade Encontrada?** | `if` | Verifica se `cod == 200` na resposta |
| **Formatar Mensagem Sucesso** | `set` | Monta a string final com nome e temperatura |
| **Telegram - Enviar Temperatura** | `telegram` | Envia a resposta de sucesso ao usuário |
| **Telegram - Enviar Erro** | `telegram` | Envia mensagem de erro ao usuário |

---

## 🌐 Endpoint da API Utilizado

```
GET https://api.openweathermap.org/data/2.5/weather
```

| Parâmetro | Valor |
|---|---|
| `q` | `Cidade,UF,BR` (ex.: `Curitiba,PR,BR`) |
| `units` | `metric` (temperatura em °C) |
| `lang` | `pt_br` (descrições em português) |
| `appid` | Sua `OPENWEATHER_API_KEY` |

---

## 📁 Arquivos do Repositório

```
.
├── README.md                      # Este arquivo
└── Bot do Clima - Telegram-fred.json     # Workflow N8N para importação
```

---

## 📄 Licença

Projeto desenvolvido para fins educacionais.
