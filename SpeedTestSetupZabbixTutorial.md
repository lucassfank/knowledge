# Velocidade da Internet com SpeedTest no Linux + Configuração para monitoramento pelo Zabbix <a name="introduction"></a>

Os planos e pacotes Internet vendidos pelos provedores aumentam em um ritmo que busca manter o alinhamento com a demanda dos serviços e usuários, que sobe cada dia mais. Além de usar os dados contratados, é importante monitorar se o plano contratado está sendo fornecido dentro das definições do contrato e da legislação que concerne estas definições. A análise dos serviços oferecidos pelos provedores é ainda mais importantes em um ambiente empresarial, que em sua maioria possui um Acordo de Nível de Serviço (SLA, do acrônimo em Inglês *Service Level Agreement*) atrelado ao contrato para ter uma segurança a mais sobre o servio contratado. 

Atualmente existem diversos sites e ferramentas para fazer a verificação de diversas métricas e indicadores de desempenho da rede. Qualquer usuário consegue pesquisar e encontrar um teste que em um clique apresenta o resultado de maneira gráfica e textual no seu dispositivo, como o [SpeedTest](https://www.speedtest.net). Inclusive alguns serviços incluem um teste na sua aplicação, se esse for um fator crítico para uma boa experiência, por exemplo o [Fast da Netflix](https://fast.com/pt/). Porém existem cenários em que é preciso manter registro destas medições, em um banco de dados ou em *logs*, e possívelmente gerar gráficos a partir destes dados. nestas situações é preciso que esses testes sejam executados e os seus resultados sejam exportados de maneira a manter estes registros.

# Sumário
1. [Plataformas de testes](#introduction)
2. [Plataformas de testes](#platforms)
    1. [Instalação e configurações do SpeedTest](#installtest)
    2. [Executando o teste](#exectest)
    3. [Porque não usar o speedtest-cli?](#cli)
3. [ Configuração para monitorar a conexão pelo Zabbix](#configmonitor)
    1. [Preparando o teste para ser usado com o Zabbix](#preptest)
    2. [Automatizar os testes](#autotest)
    3. [Instalar e configurar o agente Zabbix](#installagent)
    4. [Testar se o Zabbix server está recebendo os dados do Agente](#testagent)
    5. [Configurando o Host no Zabbix](#hostconfig)
        1. [Host](#host)
        2. [Itens](#itens)
        3. [Triggers](#triggers)
    6. [Monitorar os gráficos dos resultados no Zabbix](#monitor)
4. [Considerações Finais](#end)

## Plataformas de testes <a name="platforms"></a>

Um dos serviços mais populares é o Speedtest, que oferece [testes multiplataforma](https://www.speedtest.net/apps), incluindo Android, Apple TV, CLI, Google Chrome, iOS, macOS e Windows. A versão CLI, command line client, é dividida em quatro plataformas, para macOS, Windows, Linux e FreeBSD. As etapas na sequência deste tuturial irão utilizar a versão CLI para Linux. Já cabe destacar que não é a versão possivelmente disponibilizada pelo repositório de sua distribuição de escolha, mas a versão disponibilizada no site oficial da Ookla, mais sobre isso em seguida. A escolha do SpeedTest se deu pois a plataforma oferece inúmeros servidores em diversos países para receber a conexão do teste, oferece vários recursos de configuração do seu funcionamento, diversos parãmentros de medição, e, principalmente, a forma como os resultados são apresentados, permitindo exportar os dados dos testes realizados de maneira (relativamente) simples.

### Instalação e configurações do SpeedTest <a name="installtest"></a>

Antes de iniciar a mexer com os arquivos, certifique-se de possuir os seguintes pacotes instalados, se for seguir os comandos apresentados, ou outros similares para realiozar o precesso pela interface gráfica, se desejar, pois eles serão dependências para algumas partes do processo. Os comandos e cenários apresentados aqui são voltados para distribuições baseadas em Debian/Ubuntu, mas que podem ser facilmente adaptadas para a sua distribuição de preferência.

```
sudo apt install wget tar nano vi
```

O SpeedTest CLI a ser usado não precisa ser instalado, por assim dizer, já que ele é um executável. Mas o que faremos é criar um diretório para os arquivos a serem utilizados, dessa forma mantendo a organização dos diversos arquivos a serem manipulados. Nesta demonstração uso o diretório **Home** como localização dos arquivos, para o usuário *Administrador*. Dentro deste diretório definido, crie um novo, com nome de `speedtest` e depois acesse-o. Caso caso utilize a linha de comando, sigas os mostrados abaixo.

```
mkdir speedtest
```

```
cd speedtest
```

Agora, já dentro do diretório criado, será necessário baixar o SpeedTest para GNU/Linux do repositório do projeto. Para isso utilize o site, se estiver com interface gráfica instalada, ou o `wget` para fazer o download pelo terminal, como mostra o comando abaixo. Note que a arquitetura do dispositivo a ser instaldo influencia nesse link, dessa forma, selecione de acordo com o indicado no site ou use o comando abaixo de acordo com a arquitetura do seu dispositivo. 

Para arquitetura baseada em Intel x86 use **i386**:

```
wget https://bintray.com/ookla/download/download_file?file_path=ookla-speedtest-1.0.0-i386-linux.tgz
```

Para AMD e Intel de 64 bits (AMD64) use **x86_64**

```
wget https://bintray.com/ookla/download/download_file?file_path=ookla-speedtest-1.0.0-x86_64-linux.tgz
```

Para ARM use **arm**

```
wget https://bintray.com/ookla/download/download_file?file_path=ookla-speedtest-1.0.0-arm-linux.tgz
```

Para ARM com FPU em hardware use **armhf**

```
wget https://bintray.com/ookla/download/download_file?file_path=ookla-speedtest-1.0.0-armhf-linux.tgz
```

Para AArch64 ou ARM64 (ARM 64bit) use **aarch64**.

```
wget https://bintray.com/ookla/download/download_file?file_path=ookla-speedtest-1.0.0-aarch64-linux.tgz
```

Após ter feito o download, de acordo com a arquitetura do dispositivo, é necessário extrair o SpeedTest do arquivo. Antes de extrair é uma boa ideia renomear o arquivo, se fizer pelo terminal pode ser como mostra o comando abaixo ou de acordo com sua preferência, e mais uma vez se atente ao nome do arquivo, com base na arquitetura.

```
mv download_file\?file_path\=ookla-speedtest-1.0.0-x86_64-linux.tgz speedtest/speedtest-1.0.0-x86_64-linux.tgz
```

```
tar zxvf speedtest-1.0.0-x86_64-linux.tgz
```

### Executando o teste <a name="exectest"></a>

O arquivo compactado possui três outros arquivos, o _speedtest.md_ e o _speedtest.5_ com instruções informativas e de uso, além do próprio executável. Para fazer uso do executável, primeiro é preciso aplicar permissão para execução ao arquivo. Para atribuir permissão de execução, se definir as propriedades individouas de acesso, utilize o comando a seguir ou marque ele como executável, pela interface gráfica.

```
sudo chmod +x speedtest
```

Agora o arquivo já deve estar promto para ser executado e realizar o teste, para isso use algum dos comandos abaixo, de acordo com a preferência ou o que funcionar melhor. Na primeira vez será necessário aceitar a licensa do Speedtest, podendo ser respondendo confirmando à pergunta que será feita na primeira execução ou adicionando _--accept-gdp_ após o comando.

```
speedtest
```

```
./speedtest
```

```
/home/administrador/speedtest/speedtest
```

```
speedtest --accept-gdpr
```

Se o teste retornou um resultado semelhante ao seguinte, também disponível nos arquivos _speedtest.md_ e o _speedtest.5_ para base de comparação, ocorreu tudo certo.

```
$ speedtest
    Speedtest by Ookla

     Server: Speedtest.net - New York, NY (id = 10390)
        ISP: Comcast Cable
    Latency:    57.81 ms   (3.65 ms jitter)
   Download:    76.82 Mbps (data used: 80.9 MB)
     Upload:    37.58 Mbps (data used: 65.3 MB)
Packet Loss:     0.0%
 Result URL: https://www.speedtest.net/result/c/8ae1200c-e639-45e5-8b55-41421a079250

```


### Porque não usar o speedtest-cli? <a name="cli"></a>

Por mais que seja mais simples de instalar o speedtest-cli, já que ele provavelmente estará no repositório da sua distribuição escolhida, o que faz com que ele não seja utilizável para esta aplicação está justamente no ponto principal em ser feito o teste, a velocidade da conexão, pois esta versão traz resultados que não condizem com a realidade da conexão e resultados de outras plataformas, inclusive contradizendo as outras versões do speedtest. Outro ponto a ser considerado é a quantidade de informações fornecidas no resultado, já que o essa versão do repositório da distro apenas informa taxas de Download, Upload e Latência (nomeado Ping nessa versão).

Abaixo veja a diferença nos resultados dos dois teste que realizei (o parâmetro `--simple` exibe somente o resultado e o `--share` disponibiliza o link). Vale destacar que os servidores são diferentes nos diferentes teste, onde no `speedtest` temos os mesmos servidores da interface Web, enquanto que, pelo que percebi, no `speedtest-cli` estes mesmos servidores não estão disponíveis. Você pode pegar estas URLs mostradas para ver os resultado pelo navegador ([speedtest](https://www.speedtest.net/result/c/ba149b29-f88f-47b2-907f-5626c5f7c89c) e [speedtest-cli](http://www.speedtest.net/result/10352340490.png))

```
$  ./speedtest
    Speedtest by Ookla

     Server: Avato - Santa Maria (id = 11844)
        ISP: Vivo
    Latency:     1.61 ms   (0.05 ms jitter)
   Download:   101.41 Mbps (data used: 51.0 MB)                               
     Upload:    52.83 Mbps (data used: 24.5 MB)                               
Packet Loss:     0.0%
 Result URL: https://www.speedtest.net/result/c/ba149b29-f88f-47b2-907f-5626c5f7c89c

$  speedtest-cli --simple --share
Ping: 44.204 ms
Download: 75.33 Mbit/s
Upload: 51.93 Mbit/s
Share results: http://www.speedtest.net/result/10352340490.png

```

## Configuração para monitorar a conexão pelo Zabbix <a name="configmonitor"></a>

Se você chegou até aqui e não pretende usar o Zabbix para tratar estes resultados, o conteúdo específico do Speedtest com objetivo de somente realizar o teste termina aqui. É importante deixar claro que os passos e configurações a seguir não são para ensinar a usar o Zabbix, mas sim fazer essa configuração específica, portanto é preciso que já haja conhecimento prévio da ferramenta.

### Preparando o teste para ser usado com o Zabbix <a name="preptest"></a>

Com o teste funcionando e gerando os resultados corretamente, já é possível iniciar a configurar o script de execução e armazenamento dos dados coletados, para automatizar os testes e manter registro das coletas. O objetivo deste script será de realizar o teste, salvar no formato apresentado acima, transformar este resultado em uma sequência de dados tratada para manter a consistência do arquivo e facilitar a leitura pelo Zabbix, além de incrementar o resultado em um registro de histórico dos testes, em forma de *log* . Logo adiante as partes do script serão explicadas. 

Para criar o script, crie um arquivo (de texto) simples, ou utilize o comando a seguir, e cole as linhas da sequência, adaptando de acordo com a estrutura de diretórios que estiver usando.

```
nano runspeedtest
```

```
/home/administrador/speedtest/speedtest > /home/administrador/speedtest/speedtest.txt &&
cat /home/administrador/speedtest/speedtest.txt | sed -n '/Latency/s/ \+/ /gp' > /home/administrador/speedtest/speedtestzabbix.txt &&
cat /home/administrador/speedtest/speedtest.txt | sed -n '/Download/s/ \+/ /gp' >> /home/administrador/speedtest/speedtestzabbix.txt &&
cat /home/administrador/speedtest/speedtest.txt | sed -n '/Upload/s/ \+/ /gp' >> /home/administrador/speedtest/speedtestzabbix.txt &&
cat /home/administrador/speedtest/speedtest.txt | sed -n '/Loss/s/ \+/ /gp' >> /home/administrador/speedtest/speedtestzabbix.txt ;
date "+%Y-%m-%d--%H:%M" >> /home/administrador/speedtest/speedtestlog.txt &&
cat /home/administrador/speedtest/speedtest.txt >> /home/administrador/speedtest/speedtestlog.txt &&
echo "-----------------------------------" >> /home/administrador/speedtest/speedtestlog.txt
```

Na utilização do teste durante o uso do sistema e na linha 1 do script, caso não queira utilizar o comando no seu caminho absoluto, você pode adicionar ele aos binários do sistema, com o comando abaixo, que irá criar um link simbólico relativo do arquivo, lembrando que pode variar de acordo com a distribuição. Dessa forma basta digitar `speedtest` no terminal e ele irá executar. Mas para um script sempre é recomentado usar o caminho absoluto, não relativo, já que é preciso ter cuidado se o usuário de execução do script será o mesmo de criação e se ambos possuem todas as variáveis de sistema iguais.

```
sudo ln -sr /home/administrador/speedtest/speedtest /usr/bin/
```

Da linha 2 até a linha 4 do script, está sendo usado o comando `sed` para copiar apenas as linhas que contês as palavras chaves definidas, além de remover os espaços desnecessários do texto, já a quantidade de espaços na saída do teste varia dependedo dos números do resultado, dessa forma é preciso transformar essa saída em uma sequência controlada e igual todas as vezes, para que a leitura dos dados pelo zabbix funcione todas as vezes, independente dos valores gerados.

A linha 5 dos script irá salvar a data e hora atual para o arquivo de log, para em seguida, na linha 6 copiar o resultado do teste para o arquivo de log, seguido de um divisor, na linha 7.

Note que antes de iniciar as três últimas linhas, não é usado o operador `&&` (AND), isso se deve ao fato de este operador somente executar o comando na sequência se o imediantamente anterior foi bem sucedido ("A && B" executa B se A suceder), dessa forma, usando o operador `;` se o teste der certo ou não, de qualquer forma no log será registrado que o script foi executado com ou sem resultado, já que o operador `;` executa o próximo comando independente do resultado do comando anterior ("A ; B" executa A e depois B).

Com o script salvo, agora é preciso dar a permissão de execução para este script e também pode ser adicionado aos binários do sistema.

```
sudo chmod +x runspeedtest
```

```
sudo ln -sr /home/administrador/speedtest/runspeedtest /usr/bin/
```

### Automatizar os testes <a name="autotest"></a>

Com o script pronto, agora é possível agendar a execução do testes automaticamente, para que a coleta dos dados ocorra sem necessidade de execução manual. Para automatizar a execução, pode ser usado o Crontab. No Crontab devem ser definidos as tempos de execução e o comando a ser executado. Abra ele com o comando a seguir, com usuários normal, não root, dessa forma não é necessário permissão de administrador para agendar, além de não envolver permissões especiais para os arquivos e logs gerados. No crontab do seu usuário, siga um dos modelos abaixo, onde o teste é executado a cada 10 minutos, com uma das duas formas, a cada 10 minutos ou a cada 10 minutos pré definidos, o que não for escolhodo não precisa ser adicionado ou pode ficar comentado, usando `#` no início da linha.

```
crontab -e
```

```
#speedtest a cada 10 minutos
# m h  dom mon dow   command
#*/10 * * * * /home/administrador/speedtest/runspeedtest #a cada 10 minutos
00,10,20,30,40,50 * * * * /home/administrador/speedtest/runspeedtest #a cada 10 minutos especificos
```

O benefício em utilizar os minutos específicos definidos se justifica no caso de ser feito testes em outros dispositivos ou outras conexões na mesma rede, dessa forma eles podem ser programados para não executar ao mesmo tempo, por exemplo, um executa nos minutos terminados em 0 e outros nos terminados em 5, dessa forma os dois ocorrem a cada 10 minutos, mas nunca ao mesmo tempo, podendo evitar saturar a rede. Me deparei com essa situação quando tive que implantar testes em dois provedores diferentes, que pelo agendamento de 10 minutos livres, os testes conflitavam em horários, trazemdo incosistências.

### Instalar e configurar o agente Zabbix <a name="installagent"></a>

O SpeedTest não possui um template ou alguma configuração oficial para fazer a captura dos resultados dos testes. Para realizar isso, devem ser configurados parâmetros do usuário para esse processo funcionar. Inicialmente, se ainda não tiver, instale o agente zabbix no dispositivo que realizará os testes A instalação pode ser feita com o comando a seguir.

```
sudo apt install zabbix-agent
```

Para que o servidor Zabbix se comunique com este agente, é necessário adicionar as configurações de conexão e também os parâmetros de coleta. Abra o arquivo de configuração do agente zabbix com o seu editor de texto preferido, ou como o comando a seguir.

```
sudo nano /etc/zabbix/zabbix_agentd.conf
```

Dentro do arquivo, há quatro configurações básicas que devem ser feitas para a comunicação com o servidor, além dos parâmentros do teste. Dentro do arquivo, procure (**Ctrl** + **W** se estiver usando no editor nano) pela linha com os seguintes termos:

1. _Server=_, dentro de _Option: Server_ e insira o endereço IP do servidor Zabbix (use o comando `ip -a` no terminal do servidor para descobrir).


```
...
Server=IPDoServidorZabbix
```


2. _ListenPort=_, dentro de _Option: ListenPort_ e insira a porta que o Zabbix usa para se comunicar, por padrão é 10050.


```
...
ListenPort=10050
...
```

3. _ServerActive=_, dentro de _Active checks related_ e insira o endereço IP do servidor Zabbix.

```
...
ServerActive=IPDoServidorZabbix
...
```

4. _HostnameItem=_, dentro de _Option: Hostname_ e insira manualmente o nome do host ou use o parâmetro `system.hostname` para que seja usado o nome de host definido no próprio sistema. Lembrando que esse é o nome não deve conter espaços nem caracteres especiais. Também deverá ser colocado exatamente igual na configuração de Host no servidor Zabbix.

```
...
HostnameItem=system.hostname
...
```

5. _UserParameter=_, dentro de _Option: UserParameter_. Por fim, devem ser definidos os parâmetros usado para que o servidor Zabbix colete os dados dos testes. Para cada tipo de dado que o SpeedTest gera, deve ser adicionada uma chave para o Item. Dessa forma, Declare o parâmentro (UserParameter=), defina o nome da chave (download[\*]), informe o dado, nesse caso, o dado deve ser buscado no arquivo em que foi salvo pelo script `runspeedtest` (onde eram filtradas as linhas relevantes e retirados os espaços), com o comando `cat` exportando o arquivo e o comando `grep` buscando pela palavra de filtro, de acordo com a chave do parâmetro, e por fim, com o comando `cut` capturando o texto (numero do resultado) após uma determinada quantidade de marcadores a partir da palavra pesquisada (o marcador usado é o espaço, por isso a importância de remover os espaços desnecessários previamente - o número após o parâmentro -f define a quantidade de "casas" serem avançadas). Os parâmtros a seguir devem funcionar se o cenário foi reproduzido de acordo com o descrito nos passos anteriores, caso não funcione, ajuste o número de marcadores a ser usado. Salve o arquivo após as alterações.

```
...
UserParameter=latency[*],cat /home/administrador/speedtest/speedtestzabbix.txt | grep "Latency:" | cut -d " " -f3
UserParameter=jitter[*],cat /home/administrador/speedtest/speedtestzabbix.txt | grep "ms" | cut -d "(" -f2 | cut -d " " -f1
UserParameter=download[*],cat /home/administrador/speedtest/speedtestzabbix.txt | grep "Download:" | cut -d " " -f3
UserParameter=upload[*],cat /home/administrador/speedtest/speedtestzabbix.txt | grep "Upload:" | cut -d " " -f3
UserParameter=loss[*],cat /home/administrador/speedtest/speedtestzabbix.txt | grep "Loss:" | cut -d " " -f3 | cut -d "%" -f1
...
```

Com o agente configurado, é preciso reiniciar o serviço do agente Zabbix, e habilitar ele, para que ele seja iniciado automaticamente, em caso de reinicialização do sistema.

```
sudo systemctl restart zabbix-agent.service
```

```
sudo systemctl enable zabbix-agent.service
```


### Testar se o Zabbix server está recebendo os dados do Agente <a name="testagent"></a>

Para saber se a configuração funcionou, acesse o terminal do zervidor Zabbix e execute os comandos a seguir para capturar os dados de cada item. Forneça o endereço IP do agente configurado (use o comando `ip -a` no terminal do agente para descobrir) e a chave a ser buscada. Se os dados retornados forem os dados que constam no arquivo, estará tudo funcionando.

```
zabbix_get -s IpDoAgente -p 10050 -k "latency"
```

```
zabbix_get -s IpDoAgente -p 10050 -k "jitter"
```

```
zabbix_get -s IpDoAgente -p 10050 -k "download"
```

```
zabbix_get -s IpDoAgente -p 10050 -k "upload"
```

```
zabbix_get -s IpDoAgente -p 10050 -k "loss"
```

### Configurando o Host no Zabbix <a name="hostconfig"></a>

A seguir os campos mais importantes a serem preenchidos na criação junto com a representação do resultado final.

#### Host <a name="host"></a>

Na interface Web do seu servidor Zabbix, com acesso a privilégios administrativos, acesse **Configurações > Hosts > Criar host** e nos campos apresentados preencha os seguintes campos e veja o exemplo na imagem abaixo.

* **Nome do host (Host name)** : Deve ser igual ao configurado no agente, em _HostnameItem_ .

* **Nome visível (Visible name)**: Pode ser um nome diferente (com espaços, por exemplo) e é o que irá substituir o Nome do host na visualização da interface Web.

* **Grupos (Groups)**: Caso queira inserir em um grupo de servidores, por exemplo, *Linux Servers* ou *SpeedTest* .

* **Interfaces do agente (Agent interfaces)**: Insira o Endereço IP do agente e/ou Nome DNS (Será usado de acordo com o que estiver selecionado em "Connectado a"), a Porta definida na configuração e se é a Interface Padrão ou não.

* **Descrição (Description)**: É possível adicionar um complemento ou anotação.

* **Ativo (Enabled)**: Define se ele irá operar ou não.

As demais abas são opcionar, se você criar um template desta e outras configurações, você pode selecionar ele na aba Templates, ou qualquer outra configuração, terminando de preencher os campos clique em Adicionar.

#### Itens <a name="itens"></a>

Com o host criado, é necessario configurar os itens, para coletar os dados, para isso acesse o **Configurações > Hosts > Nome do host > Itens > Criar item** e nos campos apresentados preencha os seguintes, também veja o exemplo na imagem abaixo.

* **Nome (Name)**: Defina um nome do para o Item, sugiro usar a chave e o nome do provedor de Internet.

* **Tipo (Type)**: Escolha a opção Agente Zabbix.

* **Chave (Key)**: Preencha com a chave definida nos parâmetros do agente.

* **Interface do host (Host interface)**: Selecione a interface ou uma das interfaces cadastradas.

* **Tipo de informação (Type of information)**: Deve ser definido como Numérico (inteiro sem sinal) - *float* - pois os resultados apresentam casas decimais.

* **Unidades (Units)**: Pode ser preenchida com **Mbit/s** para Download e Upload, **ms** para Jitter e Latency, **%** para Loss.

* **Intervalo de atualização (Update interval)**: Use o mesmo intervalo que foi configurado no crontab, neste caso 10m.

* **Período de retenção do histórico (Hystory storage period)**: Este tempo varia da sua necesidade ou recursos, 180d por exemplo.

* **Período de retenção das estatísticas (Trend storage period)**: Este tempo varia da sua necesidade ou recursos, 365d por exemplo.

* **Aplicações (Aplications)**: Pode definir como SpeedTest, para poder filtrar itens e gráficos com esse marcador.

* **Ativo (Enabled)**: Define se ele irá capturar estes dados ou não.

#### Triggers <a name="triggers"></a>

Caso queira configurar um trigeer para avisar quando um dos itens não tem resultado satisfatório, acesse o **Configurações > Hosts > Nome do host > Triggers > Criar trigger** e nos campos apresentados preencha os seguintes campos, e veja o exemplo na imagem abaixo.

* **Nome (Name)**: Defina um nome do para a Trigger, sugiro ser semelhante ao nome do Item que ela faz referência.

* **Severidade (Severity)**: Se for definir um trigger para disparar quando não há coleta de dados por deperminado período de tempo, pode ser Alta, e Média se for para informar que os valores estão baixos, por exemplo.

* **Expressão (Expression)**: Por exemplo para informar que não há coleta de dados use `{NomeDoHost:NomeDoItem.last(0)}=0` e para informar que nos últimos 30 minutos as taxas de algum item está abaixo de um nível use `{NomeDoHost:download.min(30m)}<15 or {NomeDoHost:upload.min(30m)}<15`, onde 15 é a taxa definida.

* **Geração de eventos OK (OK event gereration)**: Mantenha a opção padrão selecionada, no caso, Expressão, a menos que tenha necessidade de não fazer assim.

* **Modo de geração de eventos de INCIDENTE (PROBLEM event generation mode)**: Também mantenha a opção padrão selecionada, no caso, Simples, a menos que tenha necessidade de não fazer assim.

* **Fechamentos de eventos OK (OK event closes)**: Também mantenha a opção padrão selecionada, no caso, Todos os incidentes, a menos que tenha necessidade de não fazer assim.

* **Permitir fechamento manual (Allow manual close)**: Pode ser marcado caso queira poder encerrar um incidente, para que ele não fique sendo apresentado no Dashboard.

* **Descrição (Description)**: É possível adicionar um complemento ou anotação.

* **Ativo (Enabled)**: Define se ele irá operar ou não.

### Monitorar os gráficos dos resultados no Zabbix <a name="monitor"></a>

Na interface Web do seu servidor Zabbix acesse **Monitoramento > Dados recentes** e filtre pelos itens recém configurados. Se você definiu o Campo **Aplicações** nos seus itens, você pode preencher o campo **Nome** com o que foi definido no item, neste caso **SpeedTest**, depois clicar em **Aplicar**. Dessa forma serão exibidos os dados de destas coletas, como mostra a imagem a seguir, para ver os gráficos clique em **Gráficos** no final da linha dos itens que deseja visualizar e veja um exemplo na imagem abaixo. Note que será necessário o Zabbix ter coletado pelo menos um valor para começar a exibir este gráfico.

## Considerações finais <a name="end"></a>

Pronto :grinning: ! Agora deve estar funcionando e com os principais recursos configurados. Espero que possa ajudar alguém com estas informações, ou pelo menos ter passado um pouco de conhecimento.
