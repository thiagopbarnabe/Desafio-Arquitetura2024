Este desafio não trata de um sistema real. Trata se de um sistema ficticio com o objetivo de demonstrar o meu conhecimento sobre Arquitetura de Soluções.
Não existe portanto neste desafio a dinamica real de um arquiteto de soluções.
O Arquiteto precisa ter uma visão holistica do projeto de preferencia tendo acesso com todos os envolvidos.

Quando eu vejo este documento algumas perguntas vem a minha cabeça.

Existe um sistema previo? Se não existe um sistema previo existe alguma forma de controle manual?
O Sistema precisa ser onpremises? ou Pode ser na nuvem?
É melhor que seja onpremises ou na nuvem?
Qual a disponibilidade de investimento?
Possui condição de um investimento alto inicial ou precisa que o investimento seja mais continuo (opex vs capex)
O documento de requisitos fala apenas do controle de lançamentos e exibição de relatorio existe alem desses requisitos algum outro requisito de integração com outros sistemas?
Como é a infraestrutura do local onde vai ser executada a aplicação? é um unico local ou são varios locais?
Como é o acesso a internet neste local? Possuir links de diferentes operadores com diferentes troncos (redundancia de links)
Como é o clima deste local?
Possui uma rede eletrica robusta com nobreaks e geradores no caso de falta de energia?
Com qual velocidade o caixa precisa operar, é como um caixa de supermercado que precisa passar varios items rapidamente? Se for talvez seja indicado o uso de interfaces mais performaticas no lado client
O saldo diario consolidado é baseado em 24 horas de operação ou desde a abertura da loja ate o fechamento?

Para que eu possa ser

Ja existe algum software para controle de versão na empresa?
Existe algum processo Devops configurado?

Hoje é padrão na industria a utilização do git como software para controle de versão. 
No entanto cada empresa usa plataformas e ferramentas diferentes de acordo com sua necessidade.
Recentemente fiz um estudo sobre este assunto e a minha conclusão foi que o Github é a melhor plataforma neste momento.
O maior motivo para isso são os Github Actions. Os actions são como pedaços de codigo pre prontos para utilização nas esteiras.
Os actions tornam a construção das esteiras muito mais rapidas e a manutenção é mais facil porque a comunidade esta sempre atualizando os actions.
O gitlab seria minha segunda escolha. 

A estrutura das branches depende muito do tamanho da equipe de desenvolvimento e do tamanho do software. A capacidade tecnica da equipe em relação ao uso de git tambem deve ser levada em consideração.
Algumas estruturas são padrão no mercado
Trunk Based - Neste caso todo o codigo é colocado sempre numa mesma branch. Inviavel na maioria dos casos, porque gera muitos conflitos na hora de fazer o merge
Github Flow - É como o trunk based porem neste caso existem tambem as feature branches. Uma boa opção, mas pode ser um problema para times/softwares maiores. Não possui 
Git flow
Feature Branching only

A estrutura ficaria da seguinte forma
main
develop
release
hotfix
[feature branches]



melhores praticas devops
branches, esteiras, sonarqube, elastic search, 
sistemas de backup

sistema de fila para as operações de deposito e retirada

kubernetes para elasticidade

2 tipos de pods 1 para analytics e outro para o processo
cdc

sistemas de dados separados
data warehouse para analytics

diferentes propostas de arquitetura pelo menos 3 (uma barata, uma media, e uma grande)

