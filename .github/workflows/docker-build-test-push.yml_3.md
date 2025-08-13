Para criar um fluxo de trabalho no GitHub Actions que construa a imagem Docker, faça os testes e publique a imagem tanto no **Docker Hub** quanto no **GitHub Container Registry (GHCR)**, podemos configurar dois passos de push, um para cada plataforma. Abaixo está um exemplo completo para o seu repositório.

### Fluxo de Trabalho para o GitHub Actions: Build, Teste e Push para Docker Hub e GHCR

```yaml
name: Build, Test and Push Docker Image to Docker Hub and GHCR

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

      # 5. Logar no GitHub Container Registry (GHCR)
      - name: Log in to GHCR
        uses: docker/login-action@v2
        with:
          username: ${{ github.actor }}
          password: ${{ secrets.GITHUB_TOKEN }}

      # 6. Construir a imagem Docker
      - name: Build Docker image
        run: |
          docker build -t ${{ secrets.DOCKER_USERNAME }}/ffmpeg-docker:latest .
          docker build -t ghcr.io/${{ github.repository_owner }}/ffmpeg-docker:latest .

      # 7. Testar a imagem Docker
      - name: Test Docker image
        run: |
          # Testar a imagem criada rodando um comando básico, ex: ffmpeg -version
          docker run --rm ${{ secrets.DOCKER_USERNAME }}/ffmpeg-docker:latest ffmpeg -version
          docker run --rm ghcr.io/${{ github.repository_owner }}/ffmpeg-docker:latest ffmpeg -version

      # 8. Enviar a imagem para o Docker Hub
      - name: Push Docker image to Docker Hub
        run: |
          docker push ${{ secrets.DOCKER_USERNAME }}/ffmpeg-docker:latest

      # 9. Enviar a imagem para o GitHub Container Registry (GHCR)
      - name: Push Docker image to GHCR
        run: |
          docker push ghcr.io/${{ github.repository_owner }}/ffmpeg-docker:latest
```

### Explicação dos Passos:

1. **Evento `push` e `pull_request`**:

   * O fluxo de trabalho é acionado sempre que há um `push` ou um `pull_request` na branch `main` (ou qualquer outra branch configurada).

2. **Passos do Job**:

   * **Checkout do repositório**: O código é recuperado do repositório para que o Dockerfile e outros arquivos possam ser utilizados na construção da imagem.
   * **Configuração do Docker Buildx**: O `docker/setup-buildx-action` é usado para configurar uma construção Docker otimizada e que suporta multi-plataformas.
   * **Cache de camadas Docker**: O cache de camadas do Docker ajuda a acelerar o build, reutilizando as camadas não modificadas.
   * **Login no Docker Hub**: O login no Docker Hub é feito com as credenciais configuradas como segredos no GitHub (`DOCKER_USERNAME` e `DOCKER_PASSWORD`).
   * **Login no GitHub Container Registry (GHCR)**: O login no GHCR é feito com o `GITHUB_TOKEN` fornecido automaticamente pelo GitHub, que é utilizado para autenticação. O `github.actor` é o nome de usuário do GitHub.
   * **Construção da Imagem Docker**: O comando `docker build` é utilizado para construir a imagem Docker. A imagem é criada e tagueada tanto para o Docker Hub quanto para o GHCR:

     * Docker Hub: `docker build -t ${{ secrets.DOCKER_USERNAME }}/ffmpeg-docker:latest .`
     * GitHub Container Registry (GHCR): `docker build -t ghcr.io/${{ github.repository_owner }}/ffmpeg-docker:latest .`
   * **Teste da Imagem Docker**: O teste é feito rodando `ffmpeg -version` dentro da imagem Docker para garantir que ela funciona como esperado.
   * **Push para o Docker Hub**: A imagem é enviada para o Docker Hub usando o comando `docker push`.
   * **Push para o GitHub Container Registry (GHCR)**: A imagem é enviada para o GHCR usando o comando `docker push`.

### Como Configurar os Segredos no GitHub:

Para que o GitHub Actions tenha as credenciais necessárias para acessar tanto o Docker Hub quanto o GHCR, você precisa configurar alguns segredos:

1. **DOCKER\_USERNAME**: Seu nome de usuário do Docker Hub.
2. **DOCKER\_PASSWORD**: Sua senha do Docker Hub ou um token de acesso (recomendado usar um token de acesso).
3. **GITHUB\_TOKEN**: Este token já está disponível por padrão e não precisa ser configurado manualmente. Ele é utilizado para login no GitHub Container Registry.

### Acesso às Imagens:

1. **Docker Hub**: Após o push, a imagem estará disponível no Docker Hub sob o nome `DOCKER_USERNAME/ffmpeg-docker:latest`.

   * Exemplo de comando para puxar a imagem:

     ```bash
     docker pull DOCKER_USERNAME/ffmpeg-docker:latest
     ```

2. **GitHub Container Registry (GHCR)**: Após o push, a imagem estará disponível no GitHub Container Registry sob o nome `ghcr.io/${{ github.repository_owner }}/ffmpeg-docker:latest`.

   * Exemplo de comando para puxar a imagem:

     ```bash
     docker pull ghcr.io/${{ github.repository_owner }}/ffmpeg-docker:latest
     ```

### Benefícios de Usar Ambos:

* **Docker Hub**: Amplamente usado e bem suportado. Ideal se você precisa que outras pessoas possam acessar facilmente suas imagens.
* **GitHub Container Registry**: Ideal para manter tudo no ecossistema GitHub, com autenticação fácil e controle de acesso mais integrado ao seu repositório GitHub.

### Fluxo Completo:

Com esse fluxo de trabalho, você garante que a imagem será construída, testada e empurrada para dois registros diferentes: o **Docker Hub** e o **GitHub Container Registry**. Isso proporciona mais flexibilidade no uso e distribuição das imagens Docker.

Se precisar de mais algum ajuste ou tiver dúvidas, estou à disposição!
