
## Explicação do Código Docker para Iniciar um Servidor de CS2 e um Servidor Web
O processo começa com a execução de dois contêineres Docker: um que hospeda o servidor de CS2, utilizando a imagem configurada para este propósito, e outro que hospeda um servidor web para gerar as informações das partidas. Em seguida, um script é executado dentro do contêiner do CS2 para inicializar o servidor, e, por fim, o sistema está pronto para carregar partidas a partir de uma URL gerada pelo servidor web.

### 1. Executando os Contêineres Docker

O processo de inicialização envolve a execução de dois contêineres: um para o servidor de CS2 e outro para o servidor web que gera as informações das partidas.

#### Iniciando o contêiner do servidor CS2:

```bash
docker run -d -p 27015:27015 --name servidor-cs2-01 -u cs2user -it filipess01/cs2-server /bin/bash
```

- **-d**: O container é executado em segundo plano.
- **-p 27015:27015 -p 27020:27020**: Mapeia as portas do servidor CS2.
- **--name servidor-cs2-01**: Nome amigável para o contêiner.
- **-u cs2user**: Usuário que roda o container, para segurança.
- **-it**: Terminal interativo para comandos dentro do container.
- **filipess01/eng-sports-cs2:1.0**: Imagem Docker do servidor CS2.
- **/bin/bash**: Abre o terminal Bash no contêiner.

#### Iniciando o contêiner do servidor web:

```bash
docker run -d -p 3000:3000 --name web-match-cs2 filipess01/match_information_cs2:1.0
```

- **-d**: O container é executado em segundo plano.
- **-p 3000:3000**: Mapeia a porta 3000 do host para o container web.
- **--name web-match-cs2**: Nome amigável para o container do servidor web.
- **filipess01/match_information_cs2:1.0**: Imagem Docker do servidor web que gera as informações das partidas.

Com esses dois contêineres, o servidor CS2 estará pronto para rodar partidas e o servidor web estará ativo para coletar as informações necessárias para gerar o JSON de cada partida.

### 2. Executando o Script de Inicialização do Servidor

Depois que o container está rodando, usamos o comando `docker exec` para entrar no ambiente do container e rodar o script de inicialização:

```bash
docker exec -it servidor-cs2-01 /bin/bash -c "/home/cs2user/start.sh"
```

- Esse comando executa o script `/home/cs2user/start.sh`, que contém as instruções para iniciar o servidor CS2 dentro do container já em execução.

Nesse ponto, o servidor CS2 está ativo e pronto para receber partidas, mas para carregar as partidas com as configurações desejadas, utilizamos o comando `matchzy_loadmatch_url`.

### 3. Carregar uma Partida com JSON

O sistema utiliza um comando especial para carregar uma partida a partir de uma URL que contém os dados da partida em formato JSON. Esses dados são gerados por uma aplicação HTML e processados via JavaScript.

```bash
matchzy_loadmatch_url URL
```

O comando `matchzy_loadmatch_url` carrega uma partida a partir de um link (URL), onde essa URL contém os detalhes da partida no formato JSON. Esse JSON inclui informações como os nomes e jogadores das equipes, os mapas a serem jogados, os espectadores, entre outros parâmetros.

### 4. Geração do JSON via Formulário HTML
![alt text](https://cdn.discordapp.com/attachments/753744531085852683/1286169904420225125/image.png?ex=66ecee91&is=66eb9d11&hm=0a70a9d66e6257bb62de659a5fade3c42a570a3113dedfe156ec4853df1e94a7&)

A aplicação HTML serve como interface para que os dados da partida sejam inseridos manualmente. O usuário preenche o formulário com as informações dos dois times, jogadores, mapas e espectadores. O formulário coleta essas informações e, ao ser submetido, um script JavaScript transforma os dados em um JSON no formato adequado.

Para acessar o servidor web que gera as informações das partidas, você pode seguir os seguintes passos:

1. **Acessando Localmente**: 
   - Se o contêiner estiver rodando na mesma máquina que você está utilizando para o acesso, basta inserir `localhost:3000` no navegador. Isso irá conectar diretamente à aplicação web na porta 3000. O mesmo serve para o servidor de cs2.

2. **Acessando Externamente**:
   - Se o contêiner estiver rodando em outra máquina (por exemplo, um servidor remoto), será necessário identificar o **IP público** da máquina onde o contêiner está sendo executado. Após identificar o IP, acesse o servidor web utilizando `http://<IP_Público>:3000`.
   
   - Para garantir que o acesso externo funcione, é fundamental que as **portas apropriadas estejam liberadas no firewall** da máquina remota:
     - A **porta 3000** precisa estar aberta para permitir o acesso ao servidor web.
     - Além disso, para permitir que os jogadores se conectem ao servidor de CS2, você deve liberar as portas **27015** e **27020**.
   
   - Caso o firewall da máquina ou da rede esteja bloqueando essas portas, o acesso externo não será possível, e será necessário habilitar as portas manualmente.

#### Resumo de Portas para Liberação:
- **3000**: Porta do servidor web (geração de informações das partidas).
- **27015 e 27020**: Portas do servidor de CS2 para conectar os jogadores.

Isso permitirá que tanto o servidor web quanto o servidor de CS2 sejam acessíveis externamente, garantindo o funcionamento adequado dos dois serviços.

### 5. Estrutura do JSON

A URL que será passada para o comando `matchzy_loadmatch_url` deve enviar um JSON com a seguinte estrutura:

```json
{
  "matchid": 27,
  "team1": {
    "name": "Astralis",
    "players": {
      "76561197990682262": "Xyp9x",
      "76561198010511021": "gla1ve",
      "76561197979669175": "K0nfig",
      "76561198028458803": "BlameF",
      "76561198024248129": "farlig"
    }
  },
  "team2": {
    "name": "NaVi",
    "players": {
      "76561198034202275": "s1mple",
      "76561198044045107": "electronic",
      "76561198246607476": "b1t",
      "76561198121220486": "Perfecto",
      "76561198040577200": "sdy"
    }
  },
  "num_maps": 3,
  "maplist": [
    "de_mirage",
    "de_overpass",
    "de_inferno"
  ],
  "map_sides": [
    "team1_ct",
    "team2_ct",
    "knife"
  ],
  "spectators": {
    "players": {
      "76561198264582285": "Anders Blume"
    }
  },
  "clinch_series": true,
  "players_per_team": 5,
  "cvars": {
    "hostname": "MatchZy: Astralis vs NaVi #27",
    "mp_friendlyfire": "0"
  }
}
```

Este JSON define as configurações da partida:
- **matchid**: Um identificador único da partida.
- **team1** e **team2**: Informações das equipes, incluindo os Steam IDs e nomes dos jogadores.
- **num_maps**: Número de mapas a serem jogados.
- **maplist**: Lista dos mapas selecionados.
- **map_sides**: Definição dos lados que cada equipe começa em cada mapa.
- **spectators**: Lista de espectadores e seus Steam IDs.
- **clinch_series (Partida Decisiva)**:
  - Se estiver definido como `true`, isso significa que essa partida ou mapa pode definir o vencedor da série. Por exemplo, se for uma melhor de 3 mapas (Bo3), o time que vencer dois mapas vence a série. Portanto, se um time já venceu um mapa e vence o próximo, a série termina, porque ele atingiu a quantidade necessária de vitórias.

  - Se for `false`, isso significa que a partida atual não decide a série. Esse cenário pode ser usado em partidas que não têm valor decisivo para o placar geral, ou em fases de grupo onde todas as partidas devem ser jogadas independentemente do resultado.
- **players_per_team**: Número de jogadores por equipe.
- **cvars**: Configurações do servidor, como nome do servidor e configuração de "friendly fire".

### 6. Submissão do JSON

Quando o formulário HTML é enviado, ele cria uma URL que contém o JSON com todos os detalhes da partida. Essa URL é então usada com o comando `matchzy_loadmatch_url` para carregar a partida no servidor CS2.
