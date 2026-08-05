# Barbearia do Mestre — Sistema de Agendamento

Aplicação web mobile-first para agendamento de horários, com back-end em Node.js/Express/Sequelize/PostgreSQL e integração automática com o Google Agenda. Pronta para deploy no **Render**.

## Stack técnica

- **Back-end:** Node.js, Express, Sequelize, Express Handlebars, CORS, dotenv
- **Banco de dados:** PostgreSQL (banco gerenciado gratuito do Render)
- **Front-end:** HTML + CSS + JavaScript puro (sem framework)
- **Integração:** Google Calendar API

## Estrutura do projeto

```
barbearia-agendamento/
├── config/
│   ├── database.js         # Conexão Sequelize/Postgres (usa DATABASE_URL no Render)
│   ├── servicos.js         # Lista de serviços e duração (fonte única)
│   └── googleCalendar.js   # Integração com o Google Agenda
├── models/
│   └── Agendamento.js
├── controllers/
│   └── agendamentoController.js
├── routes/
│   └── agendamentoRoutes.js
├── views/
│   ├── layouts/main.handlebars
│   ├── home.handlebars       # Tela 1 — início
│   ├── agendar.handlebars    # Tela 2 — novo agendamento
│   └── consultar.handlebars  # Tela 3 — consulta
├── public/
│   ├── css/style.css
│   └── js/ (utils.js, agendar.js, consultar.js)
├── server.js
├── render.yaml              # Blueprint de deploy automático no Render
├── package.json
└── .env.example
```

## Deploy no Render (recomendado)

### 1. Suba o projeto para o GitHub
Crie um repositório e faça o push desta pasta (o `.gitignore` já protege `.env` e `config/credentials.json`).

### 2. Crie o Blueprint no Render
1. Acesse [dashboard.render.com](https://dashboard.render.com/) → **New → Blueprint**.
2. Conecte o repositório do GitHub. O Render vai ler o `render.yaml` automaticamente e propor:
   - um **Web Service** (Node) chamado `barbearia-agendamento`
   - um **banco PostgreSQL** gratuito chamado `barbearia-db`, já conectado ao serviço via `DATABASE_URL`
3. Clique em **Apply** — o deploy começa sozinho. As tabelas são criadas automaticamente na primeira subida (via `sequelize.sync()`).

### 3. Configure o Google Agenda (variáveis de ambiente)
Depois do primeiro deploy, vá em **seu serviço → Environment** e adicione:

- `GOOGLE_CREDENTIALS_JSON` → cole o conteúdo **inteiro** do JSON da conta de serviço do Google, em uma linha só (veja o passo a passo abaixo)
- `GOOGLE_CALENDAR_ID` → e-mail do Gmail do barbeiro (dono da agenda)

Salvar essas variáveis já dispara um novo deploy automaticamente.

> Sem essas variáveis, o site funciona normal — os agendamentos são salvos no banco assim mesmo, só não cria o evento no Google Agenda.

### Passo a passo para gerar o `GOOGLE_CREDENTIALS_JSON`
1. Crie um projeto no [Google Cloud Console](https://console.cloud.google.com/).
2. Ative a **Google Calendar API** (APIs e Serviços → Biblioteca).
3. Vá em **Credenciais → Criar credenciais → Conta de serviço**.
4. Na conta criada → aba **Chaves → Adicionar chave → Criar nova chave → JSON**. Um arquivo será baixado, parecido com isto:
   ```json
   {
     "type": "service_account",
     "project_id": "seu-projeto",
     "private_key": "-----BEGIN PRIVATE KEY-----...",
     "client_email": "xxxx@seu-projeto.iam.gserviceaccount.com"
   }
   ```
5. Abra esse arquivo, copie o conteúdo inteiro e cole no campo `GOOGLE_CREDENTIALS_JSON` no Render (como uma linha só).
6. No Google Agenda do barbeiro → **Configurações do calendário → Compartilhar com pessoas específicas** → cole o `client_email` do JSON → permissão **"Fazer alterações nos eventos"**.

## Rodando localmente (opcional, para testar antes de subir)

Requer Node.js 18+ e PostgreSQL instalado.

```bash
npm install
cp .env.example .env
```

Preencha `DB_HOST`, `DB_USER`, `DB_PASSWORD`, `DB_NAME` no `.env` com os dados do seu Postgres local (ou aponte para o Postgres do Render — a URL de conexão externa fica em **seu banco → Info → External Database URL** — e cole em `DATABASE_URL`).

Para o Google Agenda em dev, é mais prático colocar o arquivo baixado em `config/credentials.json` (em vez de usar a variável de ambiente).

```bash
npm run dev    # com nodemon
npm start      # sem reload automático
```

Acesse `http://localhost:3000`.

## Rotas da API

| Método | Rota | Descrição |
|---|---|---|
| GET | `/` | Página inicial |
| GET | `/agendar` | Formulário de novo agendamento |
| GET | `/consultar` | Formulário de consulta |
| POST | `/api/agendamentos` | Cria um agendamento + evento no Google Agenda |
| GET | `/api/agendamentos?telefone=&nome=` | Busca agendamentos por telefone e/ou nome |
| GET | `/api/agendamentos/horarios-disponiveis?dia=YYYY-MM-DD` | Lista horários já ocupados em um dia |

## Personalizando

- **Serviços e duração:** edite apenas `config/servicos.js`.
- **Horários de atendimento:** array `HORARIOS` em `public/js/agendar.js`.
- **Cores e identidade visual:** variáveis CSS no topo de `public/css/style.css`.
- **Endereço/telefone no rodapé:** `views/home.handlebars`.

## Decisões tomadas

- **PostgreSQL em vez de MySQL:** o Render não tem MySQL gerenciado gratuito, só Postgres — e o Sequelize abstrai o dialeto, então models/controllers/rotas não mudam nada.
- **Credencial do Google via variável de ambiente:** o `credentials.json` não pode ir para o GitHub (fica no `.gitignore` por segurança), então em produção ele é lido a partir de `GOOGLE_CREDENTIALS_JSON`. Em dev local, o arquivo continua funcionando normalmente.
- **Um dia + horário só pode ter um agendamento:** garantido por um índice único no banco, evitando choque mesmo se dois clientes enviarem ao mesmo tempo.
- **Telefone salvo só com dígitos:** evita que formatações diferentes impeçam a consulta de encontrar o agendamento.

## Observação sobre o plano gratuito do Render

O serviço web gratuito "dorme" depois de ~15 minutos sem acesso e demora uns 30-60 segundos para acordar na próxima visita. Para uma barbearia com poucos agendamentos por dia isso é aceitável, mas se incomodar, o plano pago (a partir de ~US$7/mês) mantém o serviço sempre ativo.

## Sugestões de evolução

- Painel do barbeiro para listar/cancelar os agendamentos do dia
- Cancelamento de agendamento pelo próprio cliente (usando o `googleEventId` salvo)
- Confirmação automática por WhatsApp
