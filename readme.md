Este desafio não trata de um sistema real. Trata-se de um sistema fictício com o objetivo de demonstrar o meu conhecimento sobre Arquitetura de Soluções. Não existe portanto neste desafio a dinâmica real de um arquiteto de soluções. O arquiteto precisa ter uma visão holística do projeto de preferência tendo acesso a todos os envolvidos. Também é necessário conhecer bem a infraestrutura disponível e a capacidade de crescimento desta. As ferramentas, middlewares, etc

Quando eu vejo este documento do desafio algumas perguntas vêm à minha cabeça.

- Existe um sistema prévio? Se não existe um sistema prévio existe alguma forma de controle manual?
- O Sistema precisa ser on premises, ou pode ser na nuvem?
- É melhor que seja on premises ou na nuvem?
- Qual a disponibilidade de investimento?
- Possui condição de um investimento alto inicial ou precisa que o investimento seja mais contínuo (opex vs capex)
- O documento de requisitos fala apenas do controle de lançamentos e exibição de relatório, existe, além desses requisitos, algum outro requisito de integração com outros sistemas?
- Como é a infraestrutura do local onde vai ser executada a aplicação? É um único local ou são vários?
- Como é o acesso a internet neste local? Possui links de diferentes operadores com diferentes troncos? (redundância de links)
- Como é o clima deste local?
- Possui uma rede elétrica robusta, com nobreaks e geradores, no caso de falta de energia?
- Com qual velocidade o caixa precisa operar, é como um caixa de supermercado que precisa passar vários itens rapidamente? Se for, talvez seja indicado o uso de interfaces mais performáticas no lado cliente.
- O saldo diário consolidado é baseado em 24 horas de operação ou desde a abertura da loja até o fechamento?
- Já existe algum software para controle de versão na empresa?
- Existe algum processo Devops configurado?
- Qual a perspectiva de futuro para este software? Ele deve crescer no uso? Haverá necessidade de integrações no futuro?
- Quão crítico é o sistema? Qual o prejuízo financeiro que ele pode causar se estiver fora do ar?
- Por quanto tempo o software deve guardar as informações? Em quanto tempo elas podem ser arquivadas em discos frios(mais baratos) ou até mesmo apagadas para sempre.
- Qual direcionamento corporativo para as soluções de arquitetura? Existem regras de dados, segurança ou qualquer outra restrição?

Por não ter como responder as questões acima vou tomar algumas premissas hipotéticas para poder realizar este documento.

## Processo Devops

Hoje é padrão na indústria a utilização do Github como software para controle de versão. No entanto, cada empresa usa plataformas e ferramentas diferentes de acordo com sua necessidade. Recentemente fiz um estudo sobre este assunto e a minha conclusão foi que o Github é a melhor plataforma neste momento. O maior motivo para isso são os Github Actions. Os Actions são como pedaços de código pré prontos para utilização nas esteiras. Os Actions tornam a construção das esteiras muito mais rápidas e a manutenção é mais fácil porque a comunidade está sempre atualizando-os. 
O Gitlab seria minha segunda escolha, tem um funcionamento bem parecido apesar de não ter os Actions. O Gitlab inclusive foi o primeiro sistema que integrou tudo em uma unica ferramenta.
O AWS CodeCommit também pode ser uma opção se toda a infraestrutura for na AWS por facilitar alguns pipelines.

A estrutura das branches depende muito do tamanho da equipe de desenvolvimento e do tamanho do software. A capacidade técnica da equipe em relação ao uso de Git também deve ser levada em consideração. 
### Algumas estruturas são padrão no mercado 
- Trunk Based - Neste caso, todo o código é colocado sempre numa mesma branch. Inviável na maioria dos casos, porque gera muitos conflitos na hora de fazer o Merge.
- Github Flow - É como o trunk-based, porém neste caso, existem também as feature branches. Uma boa opção, mas pode ser um problema para times/softwares maiores, pois não possui branches dedicadas para correções de bugs, por exemplo, e o código do desenvolvimento fica junto com o código que está em produção. Isso pode dificultar bastante o merge em algumas situações e também pode ser um problema para criar uma estratégia de Continuous Deployment (Entrega Contínua). 
- Feature Branching Only - é parecido com o trunk, porém neste caso, é como se houvessem múltiplas trunks. Múltiplas equipes trabalhando em features diferentes, cada uma utilizando sua própria trunk. No final, pode ser bastante complexo integrar todo o código. 
- Git Flow - Por fim, temos o Git Flow - que vou utilizar neste desafio.

A estrutura ficaria da seguinte forma: 
- Main - Código em produção, código já entregue; 
- Develop - As feature branches são integradas primeiramente nesta branch;
- Feature branches - Código com as últimas features sendo desenvolvidas;
- Release - Código pronto, empacotado para ser enviado para produção;
- Hotfix - Código com bugs de produção corrigidos.

A grande vantagem desta estrutura é a facilidade de criar estratégias de CI/CD. 
Podemos criar uma pipeline para produção com a branch main, uma pipeline de homologação com a branch release, e uma pipeline para testes com a branch develop. 
Uma vez definida a estratégia de branchs basta criar as pipelines. As pipelines nada mais são do que scripts que rodam em uma máquina determinada e fazem todo o processo de compilação, testes unitários e também o próprio deploy. 
No caso do Github, e também do Gitlab, é utilizada uma máquina chamada GitRunner. Ambas soluções possuem versões na nuvem e on premises. A decisão entre nuvem e on premises deve ser feita de acordo com a estrutura e regras da empresa. No GitRunner precisam estar todas as dependências para a aplicação ser compilada.


#### Uma proposta de pipeline seria a seguinte:
- Preenche os secrets do código (como senhas de banco de dados ou recursos externos de rede, etc) preferencialmente utilizando uma ferramenta externa como hashicorp vault, é possível usar também as features de secrets do Gitlab e do Github também, mas estas não são tão completas). 
- Compila (neste momento todas as dependências necessária para a compilação do sistema já devem estar instaladas no Gitrunner) 
- Executa testes (unitários e de integração) 
- Quality Gate - análise de qualidade por meio de um software, como sonarqube, por exemplo. Existem outros mas o sonarqube é o mais utilizado por ser mais completo.
- Deploy - enviar diretamente para o servidor no qual estará hospedada esta versão. Seja desenvolvimento, produção e homologação, ou até outras branches caso necessário.

É muito importante também a implantação de alguma ferramenta para buscar dentro do código, como o sourcegraph ou até o elastic seach, pode ser configurado de uma forma que permita ser usada para pesquisar termos dentro do código. Ferramentas de buscas são ainda mais importantes quando o desenvolvimento não é inhouse, e sim gerado por um fornecedor externo.

# Arquitetura

## Breve Resumo
Este documento de arquitetura é referente ao projeto de um sistema de controle de fluxo de caixa diário com lançamentos.

## Requisitos Funcionais
- Controle de lançamentos
  - Débito
  - Crédito
- Consolidação diária

## Requisitos não Funcionais
O serviço de controle de lançamento não deve ficar indisponível se o sistema
de consolidado diário cair. Em dias de picos, o serviço de consolidado diário
recebe 50 requisições por segundo, com no máximo 5% de perda de
requisições.

## Layout de arquitetura
Para desenhar utilizei o Diagrama de Deployment da Uml. Utilizei o software Visual Paradigm.
O motivo de usar o diagrama de deployment é por este diagrama ser, na minha opinião, aquele que mais se aproxima da visão de infraestrutura da aplicação. Uma visão bem de arquitetura. 
O C4, por exemplo, não permite alguns ajustes e deixa o diagrama pobre. 

Por não ter todas as informações, conforme questões já acima citadas, optei por criar três arquiteturas diferentes. Todas elas devem atender aos requisitos. 
Nesta situação, eu assumi um papel parecido com o de um arquiteto de uma empresa que esteja querendo fornecer um serviço/produto para outra empresa.


### Arquitetura 1
![image](https://github.com/user-attachments/assets/a47e9c91-3d7e-45e6-a2c9-e599fbef5b0a)
Trata-se de uma arquitetura mais voltada para um ambiente on premises mais simples. Não possui elasticidade, mas possui capacidade de crescimento, basta adicionar mais servidores se houver a necessidade. Junto, é claro, com algum sistema para balancear a carga. Trata-se da arquitetura mais simples dentre as 3 que eu estou propondo, mas ainda pode ser mais simplificada conforme as anotações no desenho.

### Arquitetura 2
![image](https://github.com/user-attachments/assets/6edf9f5f-d9b3-4564-b0a1-8be41b8e5dba)
Esta arquitetura pode funcionar on premises ou na nuvem. Bastando para isso trocar o Cluster Kubernetes por um Cluster na AWS ou na AZURE, os servidores de banco de dados e redis também podem ser hospedados na nuvem. Essa solução inclui os pods que podem ser replicados conforme a necessidade. Portanto, trata-se de uma aplicação elástica pronta para atingir o resultado esperado.

### Arquitetura 3
![image](https://github.com/user-attachments/assets/a4f5fdd3-d750-448e-80dd-205053f2eb3c)
Para esta arquitetura imaginei o sistema totalmente implantado na AWS. Utilizando funções lambdas que são chamadas dinamicamente ou pelo API Gateway ou pelo Amazon MQ. Esta também é uma aplicação elástica e cresce rapidamente conforme o uso. A resposta ao aumento de requisições pode ser um pouco mais lenta do que no caso dos kubernetes, neste caso é necessário estar monitorando e agindo conforme o necessário.


## Linguagens, Frameworks e tecnologias
Não está no desenho da aplicação, pois não enxerguei necessidade de deixar explícito uma vez que praticamente qualquer linguagem pode ser adotada. Mas tenho preferência pelo .net por já ter sido desenvolvedor por muitos anos usando .net. Mas trata-se apenas de um gosto pessoal. Outras questões mais importantes como as já citadas acima devem ser levadas em consideração primeiramente.
## Segurança
Todas as chamadas são protegidas pelo api gateway. Todas as chamadas devem ser através de https. O sistema se for apenas de uso interno deve ter seus servidores protegidos da internet através de firewall. A autenticação ao servidor externo de OAuth é apenas uma sugestão. No documento não há especificação de nenhuma tecnologia ou opção, portanto, coloquei OAuth como sugestão.
## Observabilidade
Para observabilidade, recomendo a utilização de softwares como Dynatrace ou Datadog. Ambos são muito poderosos e podem ser utilizados para software on premises e também para softwares na nuvem. Existem outras opções mais "manuais", como salvar os logs da aplicação etc., mas estas opções não vão ter a mesma praticidade e eficiência do que as grandes ferramentas de mercado citadas acima.
## Retenção de dados
As filas devem ter retenção mínima de 1 semana. E os bancos de dados devem ter um backup ao menos uma vez por dia. 
Caso o sistema não precise operar 24 horas, é possível escolher o horário em que não estiver sendo utilizado para realizar este backup. 
Determinar quanto tempo que as informações precisam estar salvas no banco de dados. Elas podem ser apagadas depois de algum período de dias, meses ou anos? Se não puderem ser apagadas podem ser transferidas para um outro tipo de storage com acesso mais lento porém com custo menor? 
Isso é importante não apenas para diminuir os custos com a aplicação, mas também para garantir que o banco de dados não comece a apresentar problemas de performance depois de um certo tempo.

## Possibilidades Futuras
Podem ser incluídos novos serviços e apis para algum tipo de integração com outros softwares. A informação que hoje se encontra no redis pode ser passada para um banco de dados tipo BigData como hadoop para analytics caso isso possa oferecer algum ganho para a empresa.


## Arquitetura de Transição
![image](https://github.com/user-attachments/assets/a3554c81-970f-4b3d-892a-4703e58bedb2)
Imaginando uma aplicação existente - provavelmente um monolito que salva todas as informações no banco de dados de forma síncrona - colocaria uma aplicação de Change Data Capture neste banco de dados, que capta todas as informações no momento em que acontecem e envia-as para a fila. O Change Data Capture é fornecido por diversas empresas que atuam no mercado de dados. Na minha experiencia de uso a Qlik é a ferramenta com maior qualidade de todas disponiveis no mercado. O processo de CDC é muito vantajoso nesta situação porque não vai onerar a base de dados da aplicação legada e permite que continue executando como se nada estivesse acontecendo. Este processo permite que ambas aplicações funcionem paralelamente durante um período até que os donos da aplicação se sintam seguros, e com todos os bugs resolvidos, para fazer a troca definitiva.
O desenho contém apenas o processo que pertence à arquitetura de transição. As outras funções devem ser adicionadas conforme uma das 3 arquiteturas apresentadas previamente seja escolhida.



## Conclusão
O processo de arquitetura não possui resposta certa, mas, sem dúvidas, possui resposta errada. O arquiteto precisa de uma visão geral de todo o cenário para prover a melhor solução possível. Isso envolve inclusive o conhecimento técnico do próprio arquiteto. 
Neste desafio, me esforcei para tentar responder a todas as questões apresentadas, dentro das informações que estavam disponíveis.

