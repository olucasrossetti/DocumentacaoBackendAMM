# Backend Protocolos

Este backend foi desenvolvido para atender as necessidades de automação de gerenciamento de projetos públicos, com funcionalidades como criação de protocolos, upload de documentos, integração com AWS S3, e envio de notificações por email. Ele foi construído utilizando Node.js, Express e MongoDB, além de diversas outras tecnologias para otimizar o processo.

# Índice

1. [Backend Protocolos](#backend-protocolos)
2. [Requisitos](#requisitos)
3. [Instalação](#instalação)
   1. [1. Clonar o Repositório](#1-clonar-o-repositório)
   2. [2. Navegar para o Diretório](#2-navegar-para-o-diretório)
   3. [3. Instalar Dependências](#3-instalar-dependências)
   4. [4. Configurar Variáveis de Ambiente](#4-configurar-variáveis-de-ambiente)
4. [Iniciando a Aplicação](#iniciando-a-aplicação)
   1. [5.1 Configurar o Nginx](#51-configurar-o-nginx)
      1. [5.1.1. Abrir o Arquivo de Configuração do Nginx](#511-abrir-o-arquivo-de-configuração-do-nginx)
      2. [5.1.2. Testar a Configuração](#512-testar-a-configuração)
      3. [5.1.3. Reiniciar o Nginx](#513-reiniciar-o-nginx)
      4. [5.1.4. Verificar o Status do Nginx](#514-verificar-o-status-do-nginx)
   2. [5.2 Executar a Aplicação com screen](#52-executar-a-aplicação-com-screen)
   3. [5.3 Desanexando a sessão sem parar o APP](#53-desanexando-a-sessão-sem-parar-o-app)
   4. [5.4 Reconectar à Sessão screen](#54-reconectar-à-sessão-screen)
5. [Estrutura do Projeto](#estrutura-do-projeto)
6. [Dependências Principais](#dependências-principais)
7. [Explicação Detalhada do Código](#explicação-detalhada-do-código)
    1. [Middleware de Autenticação e Controle de Acesso](#1-middleware-de-autenticação-e-controle-de-acesso)
    2. [Tratamento de Erros](#2-tratamento-de-erros)
    3. [Modelos Mongoose](#3-modelos-mongoose)
    4. [Rotas de Protocolo](#4-rotas-de-protocolo)
    5. [Upload de arquivos para AWS S3](#5-upload-de-arquivos-para-aws-s3)
    6. [Notificações por Email](#6-notificações-por-email)
    7. [Estrutura de Rotas](#7-estrutura-de-rotas)
    8. [Configuração](#8-configuração)
    9. [Conclusão](#9-conclusão)
8. [Disclaimer](#disclaimer)
9. [Licença](#licença)
   1. [Direitos Autorais](#direitos-autorais)

## Requisitos

Certifique-se de ter as seguintes dependências instaladas no servidor:

- **Node.js** (>=14.0)
- **Nginx** (configurado como proxy reverso)
- **MongoDB** (para armazenar dados)
- **AWS S3** (para upload de arquivos)
- **screen** (para rodar o processo em background)

## Instalação

### 1. Clonar o Repositório

Primeiro, clone o repositório no seu servidor:

```bash
git clone https://github.com/olucasrossetti/BackendAMM.git
```

> No momento o repositório é privado, solicite as credenciais em 65 9 9638-6568.

### 2. Navegar para o Diretório

Entre no diretório do projeto:

```bash
cd BackendAMM
```

### 3. Instalar Dependências

Use o seguinte comando para instalar todas as dependências do projeto:

```bash
npm install
```

Esse comando irá instalar os pacotes listados no arquivo package.json, como express, mongoose, aws-sdk, multer, entre outros.

### 4. Configurar Variáveis de Ambiente

Crie um arquivo .env na raiz do projeto com as seguintes variáveis de ambiente:
Use o comando:

```bash
nano .env
```

Isso abrirá o editor de texto nano no terminal. Você pode inserir suas variáveis de ambiente no formato "CHAVE=VALOR"

```makefile
PORT=8080
AWS_REGION=us-east-2
S3_LOCATION=http://app-gerenciador-amm.s3.amazonaws.com/
S3_SECRET=solicitar
S3_KEY=solicitar
S3_BUCKET=app-gerenciador-amm
MONGO_URI=solicitar
SECRET_KEY=solicitar
EMAIL_USER=protocolocentraldeprojetos@amm.org.br
EMAIL_PASS=solicitar
```

Após inserir as variáveis, salve o arquivo:

Pressione `Ctrl + X` para sair.
Quando perguntado se deseja salvar, pressione `Y`.
Pressione `Enter` para confirmar o nome do arquivo como .env.

### 5 Iniciando a Aplicação

#### 5.1 Configurar o Nginx

Certifique-se de que o Nginx esteja configurado para servir sua aplicação como proxy reverso.
Aqui está um exemplo básico de configuração para `/etc/nginx/nginx.conf:`

##### 5.1.1. Abrir o Arquivo de Configuração do Nginx

O arquivo principal de configuração do Nginx geralmente está localizado em `/etc/nginx/nginx.conf`

Use o seguinte comando para editar o arquivo de configuração:

```bash
sudo nano /etc/nginx/nginx.conf
```

Você deve adicionar ou modificar a configuração para que o Nginx funcione como um proxy reverso para sua aplicação Node.js.
Navegue até as ultimas linhas do arquivo com as setas do teclado e adicione as seguintes informações na tag `server` como mostra a imagem:

![Arquivo de configuração do Nginx](https://i.imgur.com/ruGm5mI.png)

```nginx
server {
    if ($host = api.centraldeprojetosamm.com.br) {
        return 301 https://$host$request_uri;
    }
    listen 80;
    server_name api.centraldeprojetosamm.com.br;
    return 404;
}
```

Após ter feito todas as alterações, pressione `CTRL + O` para salvar, confirme com a tecla `Enter`.

##### 5.1.2. Testar a Configuração

Antes de reiniciar o Nginx, você pode verificar se o arquivo de configuração está correto:

```bash
sudo nginx -t
```

Se estiver tudo correto, você verá uma mensagem como `syntax is ok` e `test is successful`.

##### 5.1.3. Reiniciar o Nginx

Após editar e verificar o arquivo de configuração, reinicie o Nginx para aplicar as mudanças:

```bash
sudo systemctl restart nginx
```

##### 5.1.4. Verificar o Status do Nginx

Para garantir que o Nginx está rodando corretamente, você pode verificar o status com o seguinte comando:

```bash
sudo systemctl status nginx
```

Isso mostrará se o Nginx está ativo e funcionando.

#### 5.2 Executar a Aplicação com screen

Você pode usar o screen para garantir que a aplicação continue rodando mesmo após desconectar da sessão SSH:

```bash
screen -S amm
```

> O nome da sessão é `amm` mas pode ser qualquer outro que desejar.
> Certifique-se de estar no diretório `BackendAMM`.

Então inicie o servidor:

```bash
node index.js
```

#### 5.3 Desanexando a sessão sem parar o APP

Pressione `CTRL + A` seguido de `CTRL + D`.

#### 5.4. Reconectar à Sessão screen

Se você quiser reconectar à sessão de screen e verificar o status da aplicação:

```bash
screen -r amm
```

## Estrutura do Projeto

```perl
BACKENDAMM/
├── middleware/             # Middlewares para tratar requisições (por exemplo, autenticação)
├── models/                 # Modelos Mongoose para MongoDB
├── routes/                 # Definição de rotas da API
├── services/               # Serviços que fazem a lógica de negócio e interações com APIs externas
│   ├── userServices.js
├── tests/                  # Testes unitários e de integração
├── utils/                  # Funções utilitárias e helpers
│   ├── emailService.js     # Função para enviar notificações ao usuários
│   ├── twilioService.js    # Twilio no momento está em fase de implementação
├── .gitignore              # Arquivos e pastas ignorados pelo Git
├── config.js               # Arquivo de configuração da aplicação
├── discloud.config         # Arquivo de configuração para o Discloud (atualmente usado para testes, será descontinuado)
├── index.js                # Arquivo principal que inicia a aplicação
├── package-lock.json       # Informações sobre as versões exatas das dependências instaladas
├── package.json            # Lista de dependências e scripts do projeto
└── README.md               # Documentação do projeto
```

## Dependências Principais

- Express: Framework para construir a API REST.
- Mongoose: ODM para interação com o MongoDB.
- AWS SDK: Para interagir com o serviço de S3 da AWS.
- Multer: Middleware para upload de arquivos.
- bcryptjs: Para hash e verificação de senhas.
- jsonwebtoken: Para autenticação via tokens JWT.

## Explicação Detalhada do Código

### 1. Middleware de Autenticação e Controle de Acesso

Arquivo: `backend/middleware/auth.js`

Este middleware autentica as requisições usando **JWT (JSON Web Token)**. Ele verifica se um token de autenticação está presente no cabeçalho da requisição e, se válido, decodifica o token para identificar o usuário e anexar suas informações à requisição. Também há um middleware adicional para verificar se o usuário tem o papel de admin para acessar recursos restritos.

- **authMiddleware:** Verifica o token JWT, decodifica-o e atribui o usuário à requisição (`req.user`).
- **adminMiddleware:** Verifica se o usuário autenticado tem o papel de `admin`.

### 2. Tratamento de Erros

Arquivo: `backend/middleware/errorHandler.js`

Este middleware é responsável por capturar erros inesperados na aplicação e retornar uma resposta apropriada ao cliente. Ele garante que, em caso de erro, o servidor sempre retorne uma mensagem de erro apropriada e um status HTTP 500 (erro interno).

### 3. Modelos Mongoose

Arquivo: `backend/models/Protocolo.js`

Este modelo define a estrutura dos documentos de **Protocolos** no banco de dados **MongoDB**. A estrutura inclui campos para armazenar as informações dos usuários, detalhes do projeto, URLs de documentos enviados para AWS S3, e campos para gerenciar pendências (como pendências abertas, respostas e aprovações). Além disso, há uma enumeração para o status dos protocolos: `pendente`, `aprovado` ou `rejeitado`.
Para cada tipo de projeto, existe um `model` diferente, como levantamento topográfico, pavimentação, drenagem etc:

```js
const protocoloSchema = new mongoose.Schema(
  {
    numeroProtocolo: { type: String, required: true },
    tipo: { type: String, required: true },
    nomeObjeto: { type: String, required: true },
    oficioUrl: { type: String, required: true },
    // ...
    status: {
      type: String,
      enum: ["pendente", "aprovado", "rejeitado"],
      default: "pendente",
    },
  },
  { timestamps: true }
);
```

### 4. Rotas de Protocolo

Arquivo: `backend/routes/protocolo.js`

Da mesma maneira que existe um model para cada tipo de solicitação, acontece o mesmo com as rotas.
Aqui estão as rotas para gerenciar os protocolos.
As funcionalidades incluem:

- Criação de Protocolo: Processa uploads de arquivos para AWS S3 usando multer e cria um novo protocolo com os arquivos anexados.
- Listagem de Protocolos: Retorna todos os protocolos para administradores e permite a listagem de protocolos específicos por município para usuários.
- Pendências e Aprovações: Adiciona pendências a um protocolo e permite que administradores aprovem ou rejeitem protocolos.

```js
router.post(
  "/construcao-nova",
  authMiddleware,
  uploadConstrucaoNovaFields,
  async (req, res) => {
    // Lógica para criar um novo protocolo
  }
);
```

### 5. Upload de Arquivos para AWS S3

O sistema utiliza **AWS S3** para o armazenamento de arquivos, como documentos de protocolo. O multer-s3 é usado para processar os uploads diretamente para o bucket do S3, e o URL do arquivo carregado é armazenado no banco de dados.

```js
const upload = multer({
  storage: multerS3({
    s3: s3,
    bucket: config.s3Bucket,
    acl: "public-read",
    key: function (req, file, cb) {
      cb(null, `protocolos/civil/${uuidv4()}-${file.originalname}`);
    },
  }),
});
```

### 6. Notificações por Email

Arquivo: `backend/utils/emailService.js`

Este módulo utiliza **Nodemailer** para enviar notificações por email, como quando um protocolo é criado, aprovado ou quando uma pendência é adicionada. As notificações são enviadas para os usuários relevantes, com base no município, e para os administradores.

```js
const sendEmail = async (to, subject, text, html) => {
  const mailOptions = {
    from: config.email,
    to: to,
    subject: subject,
    text: text,
    html: html,
  };
  await transporter.sendMail(mailOptions);
};
```

### 7. Estrutura de Rotas

Cada funcionalidade principal (protocolos, uploads, notificações) possui uma rota separada. As rotas estão organizadas por tipo de requisição e são protegidas pelo middleware de autenticação.

- Rotas de Protocolo: Para gerenciar protocolos (criação, listagem, pendências).
- Rotas de Upload: Para lidar com uploads de arquivos relacionados aos protocolos.
- Rotas de Slide: Para gerenciar slides de apresentação na página de login.

### 8. Configuração

Arquivo: `backend/config.js`

A configuração sensível, como as credenciais de AWS e chaves secretas, são armazenadas em variáveis de ambiente (gerenciadas pelo `dotenv`). Estas configurações são carregadas do arquivo `.env` e exportadas para uso em todo o sistema.

```js
module.exports = {
  secretKey: process.env.SECRET_KEY,
  mongoURI: process.env.MONGO_URI,
  s3Bucket: process.env.S3_BUCKET,
  // ...
};
```

### 9. Conclusão

Este backend foi cuidadosamente arquitetado para atender às necessidades de gerenciamento de protocolos e fluxos de documentos em uma aplicação web robusta, escalável e segura. Ele usa uma combinação de tecnologias modernas como Node.js, Express, MongoDB, AWS S3 e Nodemailer para oferecer um serviço completo e eficiente para o gerenciamento de projetos públicos.

## Disclaimer

Este software é de propriedade exclusiva de **Lucas Rossetti de Souza** e foi licenciado exclusivamente para a **AMM - Associação Matogrossense dos Municípios**. O uso, modificação, distribuição ou reprodução do código sem a autorização expressa do autor é estritamente proibido. Todo o conteúdo, incluindo código, design e estrutura, é protegido por direitos autorais e outras leis de propriedade intelectual.

Qualquer uso não autorizado pode resultar em medidas legais.

## Licença

Este projeto é um software de código fechado, desenvolvido inicialmente para a **AMM - Associação Matogrossense dos Municípios**, com direitos exclusivos de uso concedidos à associação. No entanto, o autor **Lucas Rossetti de Souza** se reserva o direito de, futuramente, explorar e implantar o mesmo sistema ou sistemas similares em outras instituições, de acordo com suas necessidades e negociações específicas.

### Direitos Autorais

&copy; 2024 **Lucas Rossetti de Souza**. Todos os direitos reservados.
