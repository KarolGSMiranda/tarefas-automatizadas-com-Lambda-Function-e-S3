# Desafio DIO: Tarefas Automatizadas com Amazon S3 e AWS Lambda

Este repositório documenta minha jornada de aprendizado e a implementação prática do desafio "Tarefas automatizadas com Amazon S3 e Lambda", como parte da Formação AWS Cloud Foundations da DIO (Digital Innovation One).

O objetivo principal deste projeto é consolidar meu conhecimento em arquitetura serverless e orientada a eventos, criando uma automação onde o upload de um arquivo em um bucket S3 dispara uma função Lambda para processamento e registro.

---

## Objetivos do Desafio

Meu foco neste laboratório foi aplicar e documentar o processo de criação de uma esteira de dados automatizada. O entregável é este repositório, que serve como meu material de apoio e um diário técnico dos insights que adquiri durante a prática.

## A Arquitetura da Solução: O Padrão Ouro do Serverless

O núcleo deste desafio é um dos padrões de arquitetura mais comuns e poderosos da AWS: **S3 -> Lambda -> DynamoDB**.

Meu entendimento desse fluxo é o seguinte:

1.  **O Gatilho (Trigger):** Um usuário ou sistema faz o upload de um arquivo (um objeto) em um bucket específico no **Amazon S3**.
2.  **A Notificação do Evento:** O S3, que não é apenas um "disco na nuvem", mas um serviço inteligente, detecta essa ação (ex: `s3:ObjectCreated:*`) e gera uma notificação de evento.
3.  **A Computação (Compute):** Essa notificação é configurada para invocar (chamar) uma função **AWS Lambda**. O Lambda recebe um payload (um JSON) contendo informações sobre o evento, como o nome do bucket e a chave (o caminho) do arquivo que acabou de chegar.
4.  **O Processamento (Process):** Minha função Lambda contém a lógica de negócio. Ela pode, por exemplo, ler os metadados do arquivo, validar seu conteúdo, ou (como no desafio) simplesmente registrar sua chegada.
5.  **O Registro (Storage):** Após o processamento, a função Lambda se conecta a uma tabela no **Amazon DynamoDB** (um banco de dados NoSQL) e grava um registro, como o nome do arquivo, a data do upload e o status do processamento.

O que eu achei mais impressionante nesse modelo é a eficiência: se nenhum arquivo for enviado, o custo é praticamente zero. A arquitetura "acorda" sob demanda, executa o trabalho e "dorme" novamente, tudo em milissegundos.

## O Desafio do "Como Testar?": Minha Experiência com o LocalStack

Uma das maiores descobertas deste módulo para mim foi o **LocalStack**.

**O Problema:** Como eu poderia testar esse fluxo complexo? O método tradicional seria:
1.  Escrever o código da Lambda.
2.  Fazer o deploy na AWS.
3.  Configurar o S3 na AWS.
4.  Fazer um upload.
5.  Verificar os logs no CloudWatch.
6.  Encontrou um bug? Volte ao passo 1.

Esse ciclo de feedback é lento, custoso e depende de uma conexão constante com a internet.

**A Solução (LocalStack):** O LocalStack é um emulador dos serviços da AWS que roda inteiramente na minha máquina local (dentro de um container Docker).

Minha experiência com ele foi uma virada de chave. Eu pude simular o ambiente da AWS *inteiro* localmente. Meu processo de desenvolvimento mudou para:

1.  **Configurar:** Iniciar o LocalStack (geralmente com um `docker-compose.yml`) para "ligar" os serviços S3, Lambda e DynamoDB locais.
2.  **Criar Recursos:** Usar a `awslocal` CLI (uma versão da AWS CLI que aponta para o LocalStack) para criar meu bucket, minha tabela no DynamoDB e fazer o deploy da minha função Lambda, tudo no meu notebook.
3.  **Testar:** Usar o comando `awslocal s3 cp meu-arquivo.txt s3://meu-bucket/` para simular o upload.
4.  **Verificar:** Imediatamente, eu podia verificar os logs do container do LocalStack ou usar o `awslocal dynamodb scan ...` para ver se o registro apareceu na minha tabela local.

Esse ciclo de "codar-testar-depurar" passou de minutos para segundos.

---

## Lições Aprendidas e Insights Pessoais

* **S3 é mais que um disco:** Eu passei a ver o S3 menos como um repositório de arquivos e mais como um *hub de eventos* para a nuvem. Ele é um ponto de partida para inúmeras automações.
* **Lambda é o "canivete suíço":** A função do Lambda como "cola" (glue code) entre serviços é incrivelmente poderosa. Ela é a peça que conecta tudo.
* **A importância de um ambiente local:** O LocalStack não é apenas uma conveniência; eu o vejo como uma ferramenta essencial para o desenvolvimento serverless profissional. Ele permite testes de integração reais sem tocar na nuvem pública.
* **Infraestrutura como Código (implícito):** Embora o desafio pudesse ser feito manualmente, eu percebi que criar os recursos locais (S3, Lambda, DynamoDB) com scripts (`.sh`) já é o primeiro passo para a Infraestrutura como Código (IaC). O próximo passo lógico seria gerenciar esses recursos com CloudFormation ou Terraform.

## Fontes e Recursos de Estudo

Para aprofundar os conceitos vistos neste desafio, utilizei como base as aulas da DIO e as seguintes documentações oficiais:

* **Documentação Oficial do Amazon S3:** O guia principal sobre buckets, objetos e, crucialmente, sobre a configuração de Notificações de Eventos.
    * *aws.amazon.com/pt/s3/getting-started/*
    * *docs.aws.amazon.com/pt_br/AmazonS3/latest/userguide/NotificationHowTo.html*
* **Documentação Oficial do AWS Lambda:** O guia do desenvolvedor, essencial para entender o modelo de programação, os runtimes e como os gatilhos (triggers) funcionam.
    * *aws.amazon.com/pt/lambda/getting-started/*
    * *docs.aws.amazon.com/pt_br/lambda/latest/dg/welcome.html*
* **Documentação Oficial do LocalStack:** O guia de "como começar" e a referência da CLI `awslocal`.
    * *docs.localstack.cloud/getting-started/*
