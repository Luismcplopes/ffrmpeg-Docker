Para o repositório que você mencionou ([https://github.com/Luismcplopes/ffrmpeg-Docker](https://github.com/Luismcplopes/ffrmpeg-Docker)), podemos criar um fluxo de trabalho do GitHub Actions para:

1. **Construir** a imagem Docker.
2. **Testar** a imagem Docker.
3. **Publicar** a imagem no Docker Hub.

Aqui está um exemplo de sintaxe para o fluxo de trabalho (`.github/workflows/docker-build-test-push.yml`):

### Fluxo de Trabalho: Build, Test e Push para Docker Hub

```yaml
name: Build, Test and Push Docker Image

on:
  push:
    branches:
      - main  # Ou outra branch desejada
  pull_request:
    branches:
      - main  # Garante que os testes também sejam realizados para PRs

jobs:
  build:
    runs-on: ubuntu-latest

    steps:
      # 1. Fazer checkout do repositório
      - name: Check out repository
        uses: actions/checkout@v2

      # 2. Set up Docker Buildx (para uma build eficiente e multi-plataforma)
      - name: Set up Docker Buildx
        uses: docker/setup-buildx-action@v2

      # 3. Cache Docker layers para aceleração de build
      - name: Cache Docker layers
        uses: actions/cache@v2
        with:
          path: /tmp/.buildx-cache
          key: ${{ runner.os }}-buildx-${{ github.sha }}
          restore-keys: |
            ${{ runner.os }}-buildx-

      # 4. Logar no Docker Hub
      - name: Log in to DockerHub
        uses: docker/login-action@v2
        with:
          username: ${{ secrets.DOCKER_USERNAME }}
          password: ${{ secrets.DOCKER_PASSWORD }}

      # 5. Construir a imagem Docker
      - name: Build Docker image
        run: |
          docker build -t ${{ secrets.DOCKER_USERNAME }}/ffmpeg-docker:latest .

      # 6. Testar a imagem Docker
      - name: Test Docker image
        run: |
          # Testar a imagem criada rodando um comando básico, ex: ffmpeg -version
          docker run --rm ${{ secrets.DOCKER_USERNAME }}/ffmpeg-docker:latest ffmpeg -version

      # 7. Enviar a imagem para o Docker Hub
      - name: Push Docker image to Docker Hub
        run: |
          docker push ${{ secrets.DOCKER_USERNAME }}/ffmpeg-docker:latest
```

### Explicação do Fluxo de Trabalho:

1. **Evento `push` e `pull_request`**:

   * O fluxo de trabalho é executado sempre que há um `push` ou um `pull_request` para a branch `main` (ou qualquer outra branch configurada).

2. **Passos do Job**:

   * **Checkout do repositório**: O código é recuperado do repositório.
   * **Configuração do Docker Buildx**: Utiliza o `docker/setup-buildx-action` para configurar o Buildx, que é uma ferramenta mais eficiente para builds Docker avançados.
   * **Cache de camadas Docker**: O cache de camadas ajuda a acelerar builds subsequentes, já que reusa as camadas do Docker que não foram modificadas.
   * **Login no Docker Hub**: O login no Docker Hub é feito usando segredos configurados no GitHub (`DOCKER_USERNAME` e `DOCKER_PASSWORD`).
   * **Construção da Imagem Docker**: A imagem Docker é construída com o comando `docker build` e é tagueada com o nome de usuário do Docker Hub (`${{ secrets.DOCKER_USERNAME }}/ffmpeg-docker:latest`).
   * **Testar a Imagem Docker**: O teste é feito rodando o comando `ffmpeg -version` dentro do contêiner para garantir que a imagem está funcionando corretamente.
   * **Push para o Docker Hub**: A imagem é enviada para o Docker Hub com o comando `docker push`.

### Como Configurar os Segredos no GitHub:

Para garantir que o fluxo de trabalho possa acessar suas credenciais do Docker Hub de forma segura, é necessário configurar os segredos no GitHub:

1. Vá até o repositório no GitHub.
2. Acesse **Settings** > **Secrets and variables** > **Actions**.
3. Adicione os seguintes segredos:

   * **DOCKER\_USERNAME**: Seu nome de usuário do Docker Hub.
   * **DOCKER\_PASSWORD**: Sua senha do Docker Hub ou um token de acesso (recomendado usar um token de acesso para maior segurança).

### Testando a Imagem:

No passo de **Testar a Imagem Docker**, utilizamos o comando `docker run` para executar a imagem gerada e testar se o comando `ffmpeg` está funcionando. Esse teste pode ser ajustado conforme o seu caso de uso. Por exemplo, se você tiver um teste mais específico para o seu contêiner, é só adaptar o comando que executa dentro da imagem.

### Fluxo de Trabalho Completo:

Esse fluxo de trabalho garante que, sempre que você fizer um `push` ou abrir um `pull request` na branch `main`, a imagem será construída, testada e empurrada para o Docker Hub automaticamente.

Se precisar de mais alguma modificação ou ajuda adicional, é só avisar!
