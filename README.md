# Voice LLM API

API de chat em Node.js que integra um LLM a um pipeline completo de áudio: recebe voz, transcreve, consulta o modelo e devolve a resposta em áudio.

Arquitetura em camadas (MVC + service layer + repositories), autenticação JWT com roles, rate limiting, filtro de conteúdo, validação contra NoSQL injection e testes de integração.

> Tem também um modo `sick`, em que o assistente responde como se estivesse gripado — trocadilho com "DeepSeek". É uma feature de humor; o resto do projeto é sério.

---

## 🚀 Funcionalidades

### Core Features
* [x] **Chat baseado em LLM**: Integração com DeepSeek para respostas contextuais inteligentes
* [x] **Modo "Sick"**: IA responde como se estivesse doente (modo sátira/humor) 🤧
* [x] **Áudio para Texto**: Receba mensagens em áudio e converta para texto (Google Speech-to-Text)
* [x] **Texto para Áudio**: Respostas da IA convertidas em áudio (Google Text-to-Speech)
* [x] **Áudio para Áudio**: Pipeline completo - envie áudio, receba áudio como resposta
* [x] **Histórico de Conversas**: Todas as mensagens são persistidas no MongoDB
* [x] **Sistema de Autenticação**: JWT com roles (admin/user)

### Segurança e Validações
* [x] **Filtro de Palavras Proibidas**: Bloqueia conteúdo ofensivo e comandos maliciosos
* [x] **Rate Limiting**: Proteção contra spam (10 mensagens a cada 10 minutos)
* [x] **Autenticação JWT**: Tokens seguros com expiração configurável
* [x] **Sanitização de Áudio**: Remove emojis, markdown e caracteres especiais para síntese de voz
* [x] **Validação de Entrada**: Prevenção de SQL/NoSQL injection

---

## 💪 Instruções para rodar localmente

### Pré-requisitos

* Node.js >= 22
* MongoDB (local ou remoto)
* Conta Google Cloud com APIs habilitadas:
  * Cloud Speech-to-Text API
  * Cloud Text-to-Speech API
* Chave de API do DeepSeek

### Passos para rodar

1. Clone o repositório:

   ```bash
   git clone https://github.com/LucasFrts/voice-llm-api.git
   cd voice-llm-api
   ```

2. Instale as dependências:

   ```bash
   npm install
   ```

3. Configure as variáveis de ambiente (crie um arquivo `.env`):

   ```env
   # Servidor
   APP_PORT=3000
   NODE_ENV=development

   # MongoDB
   DATABASE_CONNECTION=mongodb://localhost:27017/deepsick
   DATABASE_CONNECTION_TEST=mongodb://localhost:27017/deepsick_test

   # JWT
   JWT_SECRET=seu_secret_super_seguro_aqui
   JWT_EXPIRES_IN=7d

   # DeepSeek API
   DEEPSEEK_API_KEY=sua_chave_deepseek_aqui
   DEEPSEEK_API_URL=https://api.deepseek.com/v1

   # Google Cloud
   GOOGLE_APPLICATION_CREDENTIALS=./path/to/google-credentials.json
   ```

4. Configure as credenciais do Google Cloud:

   * Acesse o [Google Cloud Console](https://console.cloud.google.com)
   * Crie um projeto e habilite as APIs necessárias
   * Baixe o arquivo JSON de credenciais
   * Salve o arquivo e configure o caminho em `GOOGLE_APPLICATION_CREDENTIALS`

5. Inicie o MongoDB (se estiver rodando localmente):

   ```bash
   mongod
   ```

6. Rode a aplicação em desenvolvimento:

   ```bash
   npm run dev
   ```

7. Ou rode em produção:

   ```bash
   npm start
   ```

8. Acesse a API:

   ```
   http://localhost:3000
   ```

---

## 🧪 Executando os Testes

A aplicação possui cobertura de testes de integração para os principais endpoints:

```bash
# Rodar os testes em modo watch
npm test

# Rodar linter
npm run lint

# Formatar código
npm run format
```

### Casos de Teste Implementados

* ✅ Cadastro de usuário com sucesso
* ✅ Cadastro com email inválido
* ✅ Cadastro com dados nulos
* ✅ Consulta de usuários autenticados
* ✅ Consulta com token inválido
* ✅ Edição de usuário
* ✅ Edição de usuário inexistente
* ✅ Proteção de rotas sem autenticação

---

## 📡 Endpoints da API

### Autenticação

#### `POST /login`
Autentica um usuário e retorna um token JWT.

**Request Body:**
```json
{
  "email": "usuario@example.com",
  "password": "senha123"
}
```

**Response:**
```json
{
  "statusCode": 200,
  "data": {
    "user": {
      "_id": "...",
      "name": "Nome do Usuário",
      "email": "usuario@example.com",
      "role": "user"
    },
    "token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9..."
  }
}
```

---

### Usuários

#### `POST /users`
Cadastra um novo usuário.

**Request Body:**
```json
{
  "name": "João Silva",
  "email": "joao@example.com",
  "password": "senha123",
  "language": "pt-BR"
}
```

#### `GET /users`
Lista usuários (admins veem todos, usuários veem apenas seus dados).

**Headers:**
```
Authorization: Bearer {token}
```

#### `GET /users/:id`
Busca um usuário específico.

#### `PUT /users/:id`
Atualiza dados do usuário.

#### `DELETE /users/:id`
Remove um usuário.

---

### Mensagens (Chat)

Todas as rotas de mensagens requerem autenticação via token JWT.

#### `POST /messages`
Envia uma mensagem de texto para a IA.

**Headers:**
```
Authorization: Bearer {token}
```

**Request Body:**
```json
{
  "message": "Olá, como você está?"
}
```

**Response:**
```json
{
  "statusCode": 200,
  "data": {
    "assistant": "Olá! Estou bem, obrigado por perguntar. Como posso ajudá-lo?",
    "history": [
      {
        "role": "user",
        "content": "Olá, como você está?"
      },
      {
        "role": "assistant",
        "content": "Olá! Estou bem, obrigado por perguntar..."
      }
    ]
  }
}
```

#### `POST /messages/sick`
Envia uma mensagem para a IA em "modo doente" 🤒.

**Request Body:**
```json
{
  "message": "Como você está se sentindo?"
}
```

**Response (exemplo):**
```json
{
  "statusCode": 200,
  "data": {
    "assistant": "*cof cof* 🤧 tô... mal... cabeeça dói muitooo... *espirro* achuu! friooooo... preciso... descansar... 🤒",
    "history": [...]
  }
}
```

#### `POST /messages/from-audio`
Envia um arquivo de áudio, converte para texto e processa a mensagem.

**Headers:**
```
Authorization: Bearer {token}
Content-Type: multipart/form-data
```

**Form Data:**
```
audio: [arquivo de áudio]
```

**Response:**
```json
{
  "statusCode": 200,
  "data": {
    "assistant": "Resposta da IA baseada no áudio transcrito",
    "history": [...]
  }
}
```

#### `POST /messages/to-audio`
Envia mensagem de texto e recebe resposta em áudio.

**Request Body:**
```json
{
  "message": "Conte-me uma história"
}
```

**Response:**
```
Arquivo MP3 para download
```

#### `POST /messages/from-audio/to-audio`
Pipeline completo: envia áudio, recebe resposta em áudio.

**Form Data:**
```
audio: [arquivo de áudio]
```

**Response:**
```
Arquivo MP3 para download
```

#### `GET /messages`
Retorna todo o histórico de conversas do usuário.

#### `GET /messages/:id`
Busca uma mensagem específica por ID.

---

## 🏗️ Arquitetura do Projeto

A aplicação segue o padrão **MVC** com **camadas de serviço** e **repositórios**:

```
server/
├── app.js                      # Configuração do Express
├── bin/
│   └── www.js                  # Entry point da aplicação
├── config/
│   ├── database.js             # Configuração MongoDB
│   └── badwords.js             # Lista de palavras proibidas
├── controllers/                # Camada de Controllers (MVC)
│   ├── auth-controller.js
│   ├── message-controller.js
│   └── user-controller.js
├── models/                     # Modelos Mongoose
│   ├── message.js
│   └── user.js
├── services/                   # Camada de Lógica de Negócio
│   ├── message-service.js
│   ├── audio-to-text-service.js
│   ├── text-to-audio-service.js
│   ├── message-validator-service.js
│   └── user-service.js
├── repository/                 # Camada de Acesso a Dados
│   ├── message-repository.js
│   └── user-repository.js
├── middlewares/
│   └── auth.js                 # Middleware JWT
├── helpers/
│   ├── deepseek-sdk.js         # Cliente DeepSeek
│   └── http-response.js
├── exceptions/                 # Exceções customizadas
│   ├── forbidden-word-error.js
│   ├── rate-limit-exceed-error.js
│   ├── not-processable-audio-exception.js
│   ├── unauthorized-error-exception.js
│   └── validation-error.js
└── routes/                     # Definição de rotas
    ├── index.js
    ├── messages.js
    └── users.js
```

### Princípios Arquiteturais

* **Separação de Responsabilidades**: Controllers, Services, Repositories
* **Injeção de Dependência**: Instâncias passadas via construtor
* **Orientação a Objetos**: Classes ES6+ com auto-binding
* **Tratamento de Erros**: Exceções customizadas com códigos HTTP apropriados
* **Modularidade**: Cada camada tem responsabilidade bem definida

---

## 🛠️ Tecnologias Utilizadas

### Backend
* **Node.js** (>= 22) - Runtime JavaScript
* **Express 5** - Framework web
* **MongoDB + Mongoose** - Banco de dados NoSQL

### Autenticação e Segurança
* **JWT (jsonwebtoken)** - Autenticação stateless
* **bcryptjs** - Hash de senhas
* **Rate Limiting** - Proteção contra spam

### Inteligência Artificial
* **OpenAI SDK** - Cliente para DeepSeek API
* **DeepSeek Chat** - Modelo de linguagem generativa

### Processamento de Áudio
* **@google-cloud/speech** - Speech-to-Text
* **@google-cloud/text-to-speech** - Text-to-Speech
* **fluent-ffmpeg** - Conversão e processamento de áudio
* **ffmpeg-static** - Binário FFmpeg embarcado

### Upload e Manipulação
* **multer** - Upload de arquivos multipart
* **emoji-regex** - Sanitização de texto para áudio

### Utilidades
* **winston** - Logging estruturado
* **dayjs** - Manipulação de datas
* **dotenv** - Variáveis de ambiente

### Desenvolvimento
* **Jest + Supertest** - Testes de integração
* **nodemon** - Hot reload em desenvolvimento
* **ESLint + Prettier** - Qualidade de código

---

## 🧐 Processo de Desenvolvimento

O projeto foi desenvolvido como um MVP (Minimum Viable Product) com foco em demonstrar:

1. **Arquitetura Limpa**: Separação clara de responsabilidades em camadas
2. **Integração com Serviços Externos**: DeepSeek API e Google Cloud
3. **Processamento de Mídia**: Pipeline completo de áudio (entrada/saída)
4. **Segurança**: Autenticação, autorização e validação de entrada
5. **Testabilidade**: Testes de integração cobrindo casos críticos
6. **Boas Práticas**: ES6+ Modules, async/await, error handling

### Desafios Técnicos

* **Integração Google Cloud**: Configuração de credenciais e conversão de formatos de áudio
* **Conversão de Áudio**: FFmpeg para normalizar entrada de áudio para Speech-to-Text
* **Sanitização para TTS**: Remover emojis e formatação markdown para síntese natural
* **Rate Limiting**: Implementação de janela deslizante com MongoDB
* **Modo "Sick"**: Prompt engineering para respostas humorísticas consistentes

---

## 🌟 Funcionalidade Especial: Modo "Sick"

O diferencial do projeto é o **modo "sick"** (`POST /messages/sick`), onde a IA responde como se estivesse doente. O prompt foi cuidadosamente elaborado para:

* Usar digitação irregular e trêmula
* Incluir onomatopeias (cof cof, achuu!)
* Repetir letras para ênfase (friooooo, cabeeça)
* Adicionar pausas com "..."
* Inserir emojis temáticos (🤒🤧)
* Simular sintomas físicos (tosse, febre, voz rouca)

### Exemplo de Interação

**Input:**
```
Como fazer um bolo de chocolate?
```

**Output (modo normal):**
```
Para fazer um bolo de chocolate, você vai precisar de:
- 2 xícaras de farinha
- 1 xícara de chocolate em pó
...
```

**Output (modo sick):**
```
*cof cof* 🤧 ai... bolo... né... tá... 
precisa de fariiinha... *espirro* achuu!
e chocolateee em pó... 2 xícaras acho...
tô com muitooo sono pra... explicar direito... 🤒
mistura tudooo e... *tosse* põe no forno... 180 graus...
me desculpa tô... malzão... 😷
```

---

## ⚠️ Limitações e Melhorias Futuras

### Limitações Conhecidas

* ⚠️ Sem paginação nos endpoints de listagem
* ⚠️ Histórico de conversas não tem limite de tamanho
* ⚠️ Arquivos de áudio não são deletados periodicamente
* ⚠️ Testes cobrem apenas endpoints de usuários
* ⚠️ Sem documentação Swagger/OpenAPI
* ⚠️ Logs não estruturados para produção

### Roadmap de Melhorias

* [ ] Implementar Swagger/OpenAPI para documentação interativa
* [ ] Adicionar paginação e filtros nos endpoints
* [ ] Sistema de limpeza automática de arquivos temporários
* [ ] Implementar WebSockets para chat em tempo real
* [ ] Adicionar suporte a múltiplos idiomas
* [ ] Dashboard web para visualizar conversas
* [ ] Métricas e observabilidade (Prometheus/Grafana)
* [ ] Implementar cache Redis para histórico recente
* [ ] Suporte a imagens (visão computacional)
* [ ] Testes unitários para todas as camadas
* [ ] CI/CD com GitHub Actions
* [ ] Containerização com Docker

---

## 📊 Estrutura de Dados

### Model: User
```javascript
{
  name: String,        // Nome do usuário
  email: String,       // Email único (índice)
  password: String,    // Hash bcrypt
  role: String,        // 'user' | 'admin'
  language: String,    // 'pt-BR' | 'en-US' | etc
  createdAt: Date,
  updatedAt: Date
}
```

### Model: Message
```javascript
{
  userId: ObjectId,    // Referência ao User
  role: String,        // 'user' | 'assistant'
  content: String,     // Conteúdo da mensagem
  createdAt: Date
}
```

---

## 🔒 Segurança

### Medidas Implementadas

1. **Autenticação JWT**: Tokens assinados com secret forte
2. **Hash de Senhas**: bcrypt com salt rounds = 10
3. **Validação de Entrada**: Middleware de validação em todos os endpoints
4. **Filtro de Palavras**: Lista de palavras proibidas (ofensivas e comandos maliciosos)
5. **Rate Limiting**: 10 mensagens por 10 minutos por usuário
6. **CORS**: Configurável via variáveis de ambiente
7. **Proteção NoSQL Injection**: Lista de palavras proíbe operadores MongoDB ($ne, $where, etc)

### Boas Práticas

* Nunca commitar `.env` ou credenciais
* Usar HTTPS em produção
* Configurar CORS adequadamente
* Rotacionar tokens regularmente
* Monitorar logs para atividades suspeitas
* Limitar tamanho de uploads (configurado no multer)

---

## 📝 Variáveis de Ambiente

```env
# Servidor
APP_PORT=3000                    # Porta da aplicação
NODE_ENV=development             # development | production | test

# MongoDB
DATABASE_CONNECTION=mongodb://localhost:27017/deepsick
DATABASE_CONNECTION_TEST=mongodb://localhost:27017/deepsick_test

# JWT
JWT_SECRET=seu_secret_aqui       # Deve ser uma string aleatória forte
JWT_EXPIRES_IN=7d                # Tempo de expiração do token

# DeepSeek
DEEPSEEK_API_KEY=sk-...          # Chave da API DeepSeek
DEEPSEEK_API_URL=https://api.deepseek.com/v1

# Google Cloud
GOOGLE_APPLICATION_CREDENTIALS=./credentials.json  # Caminho para o JSON
```

---

## 🤝 Contribuindo

Contribuições são bem-vindas! Para contribuir:

1. Fork o projeto
2. Crie uma branch para sua feature (`git checkout -b feature/MinhaFeature`)
3. Commit suas mudanças (`git commit -m 'feat: Adiciona MinhaFeature'`)
4. Push para a branch (`git push origin feature/MinhaFeature`)
5. Abra um Pull Request

### Padrões de Código

* Use ES6+ Modules (`import`/`export`)
* Siga o estilo do ESLint configurado
* Formate com Prettier antes de commitar
* Escreva testes para novas funcionalidades
* Documente funções complexas com JSDoc

---

## 📄 Licença

Este projeto é open source e está disponível sob a licença MIT.

---

## 👨‍💻 Autor

Desenvolvido como projeto acadêmico para demonstrar habilidades em:
* Arquitetura de software backend
* Integração com APIs de terceiros
* Processamento de áudio
* Testes automatizados
* Boas práticas de desenvolvimento

---

## 🙏 Agradecimentos

* **DeepSeek**: Pela API de LLM generativo
* **Google Cloud**: Pelos serviços de Speech e Text-to-Speech
* **MongoDB**: Pelo excelente banco de dados NoSQL
* **Comunidade Node.js**: Pelas bibliotecas incríveis

---

## 📞 Suporte

Para dúvidas, sugestões ou reportar bugs:

* Abra uma [issue no GitHub](https://github.com/LucasFrts/voice-llm-api/issues)
* Entre em contato via email (se aplicável)

---

**Voice LLM API** — chat por voz, ponta a ponta.
