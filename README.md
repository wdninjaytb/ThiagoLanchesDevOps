Thiago Lanches — Infraestrutura DevOps
Infraestrutura de desenvolvimento do Thiago Lanches, organizada com Docker Compose para executar MySQL, Backend, Frontend e um Nginx atuando como proxy reverso.

Este repositório contém a configuração de infraestrutura. O código das aplicações fica em repositórios separados.
Estrutura do projeto
Os três repositórios devem ser clonados como pastas irmãs, dentro do mesmo diretório:
BIM4/
├── ThiagoLanchesBackEnd/
│   ├── Dockerfile
│   └── .env                  # configuração local do Backend (não versionar)
├── ThiagoLanchesFrontEnd/
│   └── Dockerfile
└── ThiagoLanchesDevOps/
    ├── docker-compose.yml
    ├── nginx.conf
    ├── .env.example
    ├── .env                  # configuração local do Compose (não versionar)
    └── README.md
O Compose utiliza caminhos relativos (../ThiagoLanchesBackEnd e ../ThiagoLanchesFrontEnd). Mantenha esses nomes e a posição das pastas, ou ajuste os caminhos no docker-compose.yml.
Serviços
Serviço	Tecnologia	Função
mysql	MySQL 8.4	Persistência dos dados
backend	Node.js / Express	API e acesso ao banco
frontend	React / Vite, servido por Nginx	Interface web
nginx	Nginx	Proxy reverso e entrada da aplicação


O proxy encaminha:
- / → frontend:80
- /api/ → backend:3000/ (removendo o prefixo /api/)
Os serviços se comunicam pela rede Docker thiago_internal. O Backend utiliza o hostname mysql para acessar o banco dentro da rede.
Pré-requisitos
- Docker Desktop com Docker Compose disponível.
- Repositórios do Backend e do Frontend nas pastas indicadas acima.
- Arquivos de ambiente locais devidamente configurados.
Configuração do ambiente
1. Na pasta ThiagoLanchesDevOps, copie .env.example para .env.
2. Preencha as variáveis MYSQL_DATABASE, MYSQL_USER, MYSQL_PASSWORD e MYSQL_ROOT_PASSWORD com os valores do seu ambiente.
3. Configure também o arquivo .env do repositório ThiagoLanchesBackEnd, conforme as variáveis exigidas pela aplicação. As credenciais de acesso ao MySQL devem corresponder às configuradas no Compose. O Compose já define DB_HOST=mysql e PORT=3000 para o serviço Backend.
4. Nunca envie arquivos .env reais para o Git.
Atenção: em um volume MySQL já inicializado, alterar as variáveis do Compose não redefine automaticamente os usuários, as senhas ou os dados existentes.
Executar o projeto
Abra um terminal na pasta ThiagoLanchesDevOps.

Validar a configuração:
docker compose config --quiet
Construir e iniciar os serviços:
docker compose up -d --build
Verificar os contêineres:
docker compose ps
Endereços para teste:
- Interface web: http://localhost:8080
- API do cardápio pelo proxy: http://localhost:8080/api/menu
Consultar logs:
docker compose logs -f
Parar os serviços, preservando os volumes:
docker compose down
Não execute docker compose down -v se quiser preservar os dados do MySQL. O volume persistente está nomeado como thiagolanches_mysql_data.
Acesso ao banco durante o desenvolvimento
A configuração atual publica o MySQL em 127.0.0.1:3306, permitindo o uso de ferramentas locais como o DBeaver. O Backend utiliza mysql:3306 dentro da rede Docker.

Isso é uma configuração de desenvolvimento: no estado atual, não é correto afirmar que apenas o Nginx possui porta publicada. O isolamento de redes e a política final de exposição serão revisados separadamente, conforme a rubrica completa do projeto.
Banco de dados em outro computador
O volume thiagolanches_mysql_data é local a cada ambiente Docker. Clonar este repositório não copia automaticamente as tabelas nem os produtos de outro computador.

Em uma instalação nova, o MySQL poderá criar o banco configurado por MYSQL_DATABASE, mas a criação das tabelas e a carga inicial de dados dependem de um esquema SQL, migrações ou procedimento de inicialização do projeto. Esse processo ainda precisa ser documentado e validado antes de afirmar que o ambiente sobe completo do zero.
Estado da integração
Já foram testados no ambiente de desenvolvimento:
- Inicialização dos quatro serviços pelo Compose.
- Interface web através do Nginx em localhost:8080.
- Rota localhost:8080/api/menu, com encaminhamento do Nginx ao Backend e consulta ao MySQL.
Pendente: revisar e validar as chamadas da aplicação React para usar URLs relativas (/api/...) no lugar de referências diretas a localhost:3000, incluindo os fluxos aplicáveis de autenticação. Isso deve ser coordenado com quem estiver trabalhando no Frontend.
Segurança e versionamento
- .env e outros arquivos de credenciais locais devem permanecer fora do Git.
- .env.example contém apenas valores de exemplo e pode ser versionado.
- O projeto usa caminhos relativos para facilitar a execução pelos integrantes da equipe.
- Não remova volumes Docker para resolver problemas de configuração sem antes verificar o impacto nos dados.