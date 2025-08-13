Explicação do Fluxo de Trabalho:
Evento push e pull_request:

O fluxo de trabalho é acionado sempre que há um push ou pull_request na branch main (ou qualquer outra branch que você queira configurar).

Passos do Job:

Checkout do repositório: O código é recuperado do repositório para que o Dockerfile e outros arquivos sejam utilizados na construção da imagem.

Configuração do Docker Buildx: O docker/setup-buildx-action é utilizado para permitir a construção de imagens Docker de forma eficiente e suportando várias plataformas.

Cache de camadas Docker: O cache das camadas do Docker é utilizado para acelerar builds subsequentes, reutilizando camadas que não mudaram entre as execuções.

Login no GitHub Container Registry: O login é feito com o GITHUB_TOKEN padrão do GitHub, que permite autenticação no GHCR. O github.actor é utilizado como o nome de usuário, pois representa o nome do usuário ou organização que está executando a ação.

Construção da Imagem Docker: O comando docker build é utilizado para construir a imagem Docker a partir do Dockerfile presente no repositório. A imagem é tagueada com o nome de repositório e o nome de usuário do GitHub, seguido de ffmpeg-docker:latest.

Teste da Imagem Docker: Um teste básico é executado rodando ffmpeg -version dentro da imagem para garantir que a imagem criada é funcional.

Push para o GitHub Container Registry (GHCR): A imagem construída é empurrada para o GitHub Container Registry (GHCR) usando o comando docker push. O nome da imagem segue o formato ghcr.io/${{ github.repository_owner }}/ffmpeg-docker:latest, onde ${{ github.repository_owner }} é o nome da organização ou usuário dono do repositório.

Configuração do GITHUB_TOKEN:
O GitHub cria automaticamente um token chamado GITHUB_TOKEN para cada repositório. Este token tem permissões adequadas para autenticação no GitHub Container Registry e não requer configuração adicional. Ele é utilizado diretamente no fluxo de trabalho para login no GHCR.

Como Acessar a Imagem no GHCR:
Após o fluxo de trabalho ser executado e a imagem ser enviada ao GHCR, a URL da imagem estará no formato:

bash
Copiar
Editar
ghcr.io/${{ github.repository_owner }}/ffmpeg-docker:latest
Se você quiser usar a imagem em outro lugar, pode referenciá-la dessa maneira.

Como Visualizar e Acessar Imagens no GHCR:
Vá para a página do seu repositório no GitHub.

Acesse a aba Packages (no menu lateral do repositório), onde você verá a imagem Docker que foi publicada.

Para acessar a imagem diretamente do terminal, você pode usar o comando docker pull:

bash
Copiar
Editar
docker pull ghcr.io/${{ github.repository_owner }}/ffmpeg-docker:latest
Fluxo Completo:
Esse fluxo de trabalho ajuda a garantir que a construção e o push da imagem Docker aconteçam automaticamente sempre que houver mudanças na branch main (ou qualquer outra branch configurada). Além disso, a imagem é publicada no GitHub Container Registry (GHCR), o que é conveniente se você já estiver utilizando o GitHub como a plataforma de gerenciamento de código-fonte.

Se precisar de mais ajustes ou tiver alguma dúvida, é só falar!
