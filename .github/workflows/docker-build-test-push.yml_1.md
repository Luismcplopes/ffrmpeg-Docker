Explicação do Fluxo de Trabalho:
Evento push e pull_request:

O fluxo de trabalho é executado sempre que há um push ou um pull_request para a branch main (ou qualquer outra branch configurada).

Passos do Job:

Checkout do repositório: O código é recuperado do repositório.

Configuração do Docker Buildx: Utiliza o docker/setup-buildx-action para configurar o Buildx, que é uma ferramenta mais eficiente para builds Docker avançados.

Cache de camadas Docker: O cache de camadas ajuda a acelerar builds subsequentes, já que reusa as camadas do Docker que não foram modificadas.

Login no Docker Hub: O login no Docker Hub é feito usando segredos configurados no GitHub (DOCKER_USERNAME e DOCKER_PASSWORD).

Construção da Imagem Docker: A imagem Docker é construída com o comando docker build e é tagueada com o nome de usuário do Docker Hub (${{ secrets.DOCKER_USERNAME }}/ffmpeg-docker:latest).

Testar a Imagem Docker: O teste é feito rodando o comando ffmpeg -version dentro do contêiner para garantir que a imagem está funcionando corretamente.

Push para o Docker Hub: A imagem é enviada para o Docker Hub com o comando docker push.

Como Configurar os Segredos no GitHub:
Para garantir que o fluxo de trabalho possa acessar suas credenciais do Docker Hub de forma segura, é necessário configurar os segredos no GitHub:

Vá até o repositório no GitHub.

Acesse Settings > Secrets and variables > Actions.

Adicione os seguintes segredos:

DOCKER_USERNAME: Seu nome de usuário do Docker Hub.

DOCKER_PASSWORD: Sua senha do Docker Hub ou um token de acesso (recomendado usar um token de acesso para maior segurança).

Testando a Imagem:
No passo de Testar a Imagem Docker, utilizamos o comando docker run para executar a imagem gerada e testar se o comando ffmpeg está funcionando. Esse teste pode ser ajustado conforme o seu caso de uso. Por exemplo, se você tiver um teste mais específico para o seu contêiner, é só adaptar o comando que executa dentro da imagem.

Fluxo de Trabalho Completo:
Esse fluxo de trabalho garante que, sempre que você fizer um push ou abrir um pull request na branch main, a imagem será construída, testada e empurrada para o Docker Hub automaticamente.

Se precisar de mais alguma modificação ou ajuda adicional, é só avisar!
