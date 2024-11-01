Este desafio não trata de um sistema real. Trata se de um sistema ficticio com o objetivo de demonstrar o meu conhecimento sobre Arquitetura de Soluções.
Não existe portanto neste desafio a dinamica real de um arquiteto de soluções.
O Arquiteto precisa ter uma visão holistica do projeto de preferencia tendo acesso a todos os envolvidos.
Tambem é necessario conhecer bem a infraestrutura disponivel e a capacidade de crescimento desta. As ferramentas, middlewares, etc 

Quando eu vejo este documento do desafio algumas perguntas vem a minha cabeça.

- Existe um sistema previo? Se não existe um sistema previo existe alguma forma de controle manual?
- O Sistema precisa ser onpremises? ou Pode ser na nuvem?
- É melhor que seja onpremises ou na nuvem?
- Qual a disponibilidade de investimento?
- Possui condição de um investimento alto inicial ou precisa que o investimento seja mais continuo (opex vs capex)
- O documento de requisitos fala apenas do controle de lançamentos e exibição de relatorio existe alem desses requisitos algum outro requisito de integração com outros sistemas?
- Como é a infraestrutura do local onde vai ser executada a aplicação? é um unico local ou são varios locais?
- Como é o acesso a internet neste local? Possuir links de diferentes operadores com diferentes troncos (redundancia de links)
- Como é o clima deste local?
- Possui uma rede eletrica robusta com nobreaks e geradores no caso de falta de energia?
- Com qual velocidade o caixa precisa operar, é como um caixa de supermercado que precisa passar varios items rapidamente? Se for talvez seja indicado o uso de interfaces mais performaticas no lado client
- O saldo diario consolidado é baseado em 24 horas de operação ou desde a abertura da loja ate o fechamento?
- Ja existe algum software para controle de versão na empresa?
- Existe algum processo Devops configurado?
- Qual a perspectiva de futuro para este software? Ele deve crescer no uso? Havera necessidade de integrações no futuro?
- Quão critico é o sistema? Qual o prejuizo financeiro que ele pode causar se estiver fora do ar? 
- Por quanto tempo o software deve guardar as informações? Em quanto tempo elas podem ser arquivadas em discos frios(mais baratos) ou até mesmo apagados para sempre.
- Qual direcionamento corporativo para as soluções de arquitetura? Existem regras de dados, segurança ou qualquer outra restrição?

Por não ter como responder as questões acima vou tomar algumas premissas hipoteticas para poder realizar este documento.

## Processo Devops


Hoje é padrão na industria a utilização do git como software para controle de versão. 
No entanto cada empresa usa plataformas e ferramentas diferentes de acordo com sua necessidade.
Recentemente fiz um estudo sobre este assunto e a minha conclusão foi que o Github é a melhor plataforma neste momento.
O maior motivo para isso são os Github Actions. Os actions são como pedaços de codigo pre prontos para utilização nas esteiras.
Os actions tornam a construção das esteiras muito mais rapidas e a manutenção é mais facil porque a comunidade esta sempre atualizando os actions.
O gitlab seria minha segunda escolha. O AWS CodeCommit tambem pode ser uma opção se toda a infraestrutura for na AWS por facilitar alguns pipelines.

A estrutura das branches depende muito do tamanho da equipe de desenvolvimento e do tamanho do software. A capacidade tecnica da equipe em relação ao uso de git tambem deve ser levada em consideração.
Algumas estruturas são padrão no mercado
Trunk Based - Neste caso todo o codigo é colocado sempre numa mesma branch. Inviavel na maioria dos casos, porque gera muitos conflitos na hora de fazer o merge
Github Flow - É como o trunk based porem neste caso existem tambem as feature branches. Uma boa opção, mas pode ser um problema para times/softwares maiores, pois não possui branches dedicadas para correções de bugs por exemplo, e o codigo do desenvolvimento fica junto com o codigo que esta em produção isso pode dificultar bastante o merge em algumas situações e tambem pode ser um problema para criar uma estretegia de Continuos Deployment (Entrega continua)
Feature Branching only - é parecido com o trunk porem neste caso é como se houvessem multiplas trunks. Multiplas equipes trabalhando em features diferentes cada uma utilizando sua propria trunk. No final voce pode ser bastante complexo integrar todo o codigo
Git flow - Por fim temos o git flow que é o que vou utilizar neste desafio.

A estrutura ficaria da seguinte forma
main - Codigo em produção, codigo ja entregue.
[feature branches]
develop - codigo com as ultimas features sendo desenvolvidas. As feature branches são integradas primeiramente nesta branch.
release - codigo pronto, empacotado para ser enviado para produção
hotfix - codigo com bugs de produção corrigidos

A grande vantagem desta estrutura é a facilidade de criar estrategias de CI/CD. Podemos criar uma pipeline para produção com a branch main, uma pipeline de homologação com a branch release, e uma pipeline para testes com a branch develop.
Uma vez definida a estrategia de branchs basta criar as pipelines. As pipelines nada mais são do que scripts que rodam em uma maquina determinada e fazem todo o processo de compilação, testes unitarios e tambem o proprio deploy. 
No caso do github e tambem do gitlab é utilizada uma maquina chamada gitrunner. Ambas soluções possuem versões na nuvem e onpremises. A decisão entre nuvem e onpremises deve ser feita de acordo com a estrutura e regras da empresa. No gitrunner precisam estar todas as dependencias para ser compilada a aplicação.

Uma proposta de pipeline seria a seguinte abaixo
Preenche os secrets do codigo(como senhas de banco de dados ou recursos externos de rede, etc) preferencialmente utilizando uma ferramenta externa como hashicorp vault, é possivel usar tambem as features de secrets do gitlab e do github tambem mas não são tão completas)
Compila (neste momento todas as dependencias necessaria para a compilação do sistema ja devem estar instaladas no gitrunner)
Executa testes (unitarios e de integração)
Passar em algum software para analise de qualidade como sonarqube por exemplo(existem outros mas o sonarqube é o mais utilizado por ser mais completo)
Deploy (enviar diretamente para o servidor onde estara hospedada esta versão seja desenvolvimento, produção e homologação ou até outras branchs caso necessario)

É muito importante tambem a implantação de alguma ferramenta para buscar dentro do codigo, como o sourcegraph ou até o elastic seach pode ser configurado de uma forma que posssa ser usada para pesquisar termos dentro do codigo. 
Ferramentas de buscas são ainda mais importantes quando o desenvolvimento não é inhouse e sim gerado por um fornecedor externo.

# Arquitetura

## Breve Resumo
Este documento de arquitetura é referente ao projeto de um sistema de controle de fluxo de caixa diario com lanaçamentos.

## Requisitos Funcionais
- Controle de lançamentos
  - Debito
  - Credito
- Consolidação diaria

## Requisitos não Funcionais
O serviço de controle de lançamento não deve ficar indisponível se o sistema
de consolidado diário cair. Em dias de picos, o serviço de consolidado diário
recebe 50 requisições por segundo, com no máximo 5% de perda de
requisições.

## Layout de arquitetura
Para desenhar utilizei o Diagrama de Deployment da Uml. Utilizei o software Visual Paradigm (o qual não tenho muita pratica pois no meu trabalho uso enterprise architect porem o EA não tem versão gratuita/comumnity)
O motivo de usar o diagrama de deployment é por este diagrama ser na minha opinião aquele que mais se aproxima da visão de infraestrutura da aplicação. Uma visão bem de arquitetura. O C4 por exemplo não permite alguns ajustes e deixa o diagrama pobre. 
Por não ter todas as informações conforme questões ja acima citadas optei por criar 3 arquiteturas diferentes. Todas elas devem atender aos requisitos. Nesta situação eu assumi um papel parecido com o de um arquiteto de uma empresa que esteja querendo fornecer um serviço/produto para outra empresa.

### Arquitetura 1
![image](https://github.com/user-attachments/assets/a47e9c91-3d7e-45e6-a2c9-e599fbef5b0a)
Trata se de uma arquitetura mais voltada para um ambiente on premises mais simples. Não possui elasticidade mas possui capacidade de crescimento bastam adicionar mais servidores se houver a necessidade. Junto é claro com algum sistema para balancear a carga. Trata se da arquitetura mais simples dentre as 3 que eu estou propondo mas ainda pode ser mais simplificada conforme as anotações no desenho. 

### Arquitetura 2
![image](https://github.com/user-attachments/assets/6edf9f5f-d9b3-4564-b0a1-8be41b8e5dba)
Esta arquitetura pode funcionar on premises ou na nuvem. Bastando para isso trocar o Cluster Kubernetes por um cluzer na AWS ou na AZURE, os servidores de banco de dados e redis tambem podem ser hospedados na nuvem.  Essa solução inclui os pods que podem ser replicados conforme a necessidade. Portanto trata se de uma aplicação elastica pronta para atingir o resultado esperado.

### Arquitetura 3
![image](https://github.com/user-attachments/assets/a4f5fdd3-d750-448e-80dd-205053f2eb3c)
Para esta arquitetura imaginei o sistema totalmente implantado na AWS. Utilizando funções lambdas que são chamadas dinamicamente ou pelo API Gateway ou pelo Amazon MQ. Esta tambem é uma aplicação elastica e cresce rapidamente conforme o uso. A resposta ao aumento de requisições pode ser um pouco mais lenta do que no caso do kubernetes, neste caso é necessario estar monitorando e agindo conforme o necessario.



## Linguagens, Frameworks e tecnologias
Não esta no desenho da aplicação pois não enxerguei necessidade de deixar explicito uma vez que praticamente qualquer linguagem pode ser adotada. Mas tenho preferencia pelo .net por ja ter sido desenvolvedor por muitos anos usando .net. Mas trata se apenas de um gosto pessoal. Outras questões mais importantes como as ja citadas acima devem ser levadas em consideração primeiramente.

## Segurança
Todas as chamadas são protegidas pelo api gateway. Todas as chamadas devem ser atravez de https. O sistema se for apenas de uso interno deve ter seus servidores protegidos da internet atravez de fireewall. A autenticação ao servidor externo de oauth é apenas uma sugestão. No documento não especifica nenhuma tecnologia ou opção portanto coloquei OAUTH como sugestão. 

## Observabilidade
Para observabilidade recomendo a utilização de softwares como Dynatrace ou Datadog. Ambos são muito poderosos e podem ser utilizado para software on premises e tambem para softwares na nuvem. Existem outras opções mais "manuais" como salvar os logs da aplicação etc. Mas estas opções não vão ter a mesma praticidade e eficiencia do que essas grandes ferramentas de mercado citadas acima.

## Retenção de dados
As filas deve ter retenção minima de 1 semana. E os bancos de dados devem ter um backup ao menos uma vez por dia. (caso o sistema não precisar operar 24 horas escolher o horario em que não estiver sendo utilizado para realizar este backup)
Determinar quanto tempo que as informações precisam estar salvas no banco de dados. Elas podem ser apagadas depois de algum periodo de dias, meses ou anos? Se não puderem ser apagadas podem ser transfiridas para um outro tipo de storage com acesso mais lento porem com custo menor? Isso é importante não apenas para diminuir os custos com a aplicação mas tambem para garantir que o banco de dados não comece a apresentar problemas de performance depois de um certo tempo.


## Possibilidades Futuras
Podem ser incluidos novos serviços e apis para algum tipo de integração com outros softwares.
A informação que hoje se encontra no redis pode ser passada para um banco de dados tipo BigData como hadoop para analytics caso isso possa oferecer algum ganho para a empresa.

## Arquitetura de Transição
![image](https://github.com/user-attachments/assets/a3554c81-970f-4b3d-892a-4703e58bedb2)
Imaginando uma aplicação existente provavelmente um monolito que salva todas as informações no banco de dados de forma sincrona. Pensei em colocar uma aplicação de Change Data Capture neste banco de dados que capta todas as informações no momento em que acontecem e enviam para a fila. Este processo permite que ambas aplicações funciomem paralelamente durante um periodo até que os donos da aplicação se sintam seguros e com todos os bugs resolvidos para fazer a troca definitiva. O desenho contem apenas o processo que pertence a arquitetura de transição. As outras funções devem ser adicionadas conforme uma das 3 arquiteturas apresentadas previamente seja escolhida.


## Conclusão
O processo de arquitetura não possui resposta certa. Mas possui resposta errada. O arquiteto precisa de uma visão geral de todo o cenario para prover a melhor solução possivel. Isso envolve inclusive o conhecimento tecnico do proprio arquiteto. Neste desafio me esforcei para tentar responder todas as questões apresentadas no desafio dentro das informações que eu tinha disponivel.
