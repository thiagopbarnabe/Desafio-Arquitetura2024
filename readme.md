Este desafio não trata de um sistema real. Trata se de um sistema ficticio com o objetivo de demonstrar o meu conhecimento sobre Arquitetura de Soluções.
Não existe portanto neste desafio a dinamica real de um arquiteto de soluções.
O Arquiteto precisa ter uma visão holistica do projeto de preferencia tendo acesso com todos os envolvidos.
Tambem é necessario conhecer bem a infraestrutura disponivel e a capacidade de crescimento desta. As ferramentas, middlewares, etc 

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
Ja existe algum software para controle de versão na empresa?
Existe algum processo Devops configurado?
Qual a perspectiva de futuro para este software? Ele deve crescer no uso? Havera necessidade de integrações no futuro?
Quão critico é o sistema? Qual o prejuizo financeiro que ele pode causar se estiver fora do ar? 
Por quanto tempo o software deve guardar as informações? Em quanto tempo elas podem ser arquivadas em discos frios(mais baratos) ou até mesmo apagados para sempre.
Qual direcionamento corporativo para as soluções de arquitetura? Existem regras de dados, segurança ou qualquer outra restrição?


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

![image](https://github.com/user-attachments/assets/a47e9c91-3d7e-45e6-a2c9-e599fbef5b0a)






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

