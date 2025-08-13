Aqui está um exemplo de sintaxe de fluxo de trabalho para o GitHub Actions que faz o build de um contêiner Docker e o envia para o Docker Hub.

### Exemplo de fluxo de trabalho no GitHub Actions (`.github/workflows/docker-image.yml`):

```yaml
name: Build and Push Docker Image

on:
  push:
    branches:
      - main  # Ou qualquer outra branch que deseja que o workflow seja executado

jobs:
  build:
    runs-on: ubuntu-latest

    steps:
      - name: Check out repository
        uses: actions/checkout@v2

      - name: Set up Docker Buildx
        uses: docker/setup-buildx-action@v2

      - name: Cache Docker layers
        uses: actions/cache@v2
        with:
          path: /tmp/.buildx-cache
          key: ${{ runner.os }}-buildx-${{ github.sha }}
          restore-keys: |
            ${{ runner.os }}-buildx-

      - name: Log in to DockerHub
        uses: docker/login-action@v2
        with:
          username: ${{ secrets.DOCKER_USERNAME }}
          password: ${{ secrets.DOCKER_PASSWORD }}

      - name: Build and push Docker image
        uses: docker/build-push-action@v3
        with:
          context: .
          push: true
          tags: ${{ secrets.DOCKER_USERNAME }}/my-image-name:latest

```

### Explicação do Fluxo de Trabalho:

1. **`on: push`**: O fluxo de trabalho é acionado sempre que há um `push` para a branch `main`. Você pode alterar a branch ou usar outros eventos de trigger conforme necessário.

2. **`jobs.build`**: A job de build é definida para rodar em `ubuntu-latest`.

3. **`actions/checkout@v2`**: Este passo faz o checkout do repositório para que o Dockerfile e os arquivos necessários possam ser usados para construir a imagem Docker.

4. **`docker/setup-buildx-action@v2`**: Configura o Buildx para construir imagens Docker de forma otimizada.

5. **`actions/cache@v2`**: Faz cache das camadas do Docker para acelerar o processo de build nas execuções subsequentes.

6. **`docker/login-action@v2`**: Faz login no Docker Hub com as credenciais armazenadas nos segredos do GitHub. As credenciais (`DOCKER_USERNAME` e `DOCKER_PASSWORD`) precisam ser configuradas como segredos no repositório (Settings > Secrets).

7. **`docker/build-push-action@v3`**: Constrói e envia a imagem Docker para o Docker Hub. A imagem será tagueada como `DOCKER_USERNAME/my-image-name:latest` (substitua `my-image-name` pelo nome da sua imagem).

### Como configurar os segredos no GitHub:

1. Vá até o repositório no GitHub.
2. Acesse **Settings** > **Secrets and variables** > **Actions**.
3. Adicione os segredos:

   * **DOCKER\_USERNAME**: Seu nome de usuário do Docker Hub.
   * **DOCKER\_PASSWORD**: Sua senha do Docker Hub ou um token de acesso.

Agora, sempre que você fizer um `push` para a branch especificada (por exemplo, `main`), o GitHub Actions irá construir a imagem Docker e empurrá-la para o Docker Hub automaticamente.

Se precisar de mais ajustes ou detalhes, é só avisar!
