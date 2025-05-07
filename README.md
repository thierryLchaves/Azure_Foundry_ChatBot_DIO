<img src="https://media.licdn.com/dms/image/v2/D5612AQG6Mf0a5ezeag/article-cover_image-shrink_600_2000/B56ZT.izGkGUAU-/0/1739437323394?e=2147483647&v=beta&t=bWArTk4rhC2ycAywx87tHYPDX7w50Nu4cscwyZm-rBY" alt="selo" width="80" align="left" />

&nbsp;&nbsp;&nbsp;&nbsp;

# Azure Foundry - Chatbot

Chatbot baseado em conteúdos PDFs utilizando o Azure Foundry


O documento a seguir descreve o processo de criação de um chatbot utilizando a plaforma **Azure Foundry**, desde a configuração dos componentes necessários, para aplicação e treinamento do modelo, bem como demonstração de uso e aplicação do modelo na prática. 

## Índice

- [1. Workspace e Recursos](#1-workspace-e-recursos)
  - [1.1 Sobre Recursos utilizados](#11-recursos-utilizados)
  - [1.2 Configuração do Azure AI Search](#12-azure-ai-search)
  - [1.3 Configuração do Azure AI Foundry](#13-azure-ai-foundry)
- [2. Azure AI Foundry Studio](#2-azure-ai-foundry-studio)
  - [2.1 Criando um novo Projeto](#21-criação-do-projeto)
  - [2.2 Criando um agente](#22-criando--agente)
  - [2.3 Criando um novo Projeto](#23-implantando-modelos)
    - [2.3.1 Adicionando dados no Projeto](#231-adicionando-os-dados)    
    - [2.3.2 Contextualizando o chat](#232-contextualizando-o-chat)    
- [3. Avaliando o modelo](#3-avaliando-o-modelo)
    - [3.1 Avaliando Manualmente](#31-avaliacao-manual)    
    - [3.2 Avaliando De forma Automatizada](#32-avaliacao-aAvaliação)    
- [Funcionalidades](#funcionalidades)
- [Licença](#licenca)


## 1. Workspace-e-Recursos

**Para o início do projeto, foi criada uma workspace nomeada como `proj-02-Dio`:**

Para evitar problemas com "recursos indisponíveis", tanto o workspace quanto os demais recursos foram provisionados com a localidade **East US**.
Ps: Durante a configuração do projeto foi visualizado que o modelo de linguagem gpt-4o,não possuia quotas disponíveis para implantação como Global Standard, então foi adotado a implementação de outro modelo disponível, e visando a celeridade de processamento do processo foi selecionado a localidade mencionada.

![Criação de Workspace](inputs/imgs/conf_workspace.png)  

![Workspace vazio](inputs/imgs/empty_workspace.png)  

---

### 1.1 Recursos-Utilizados

Antes de iniciar a criação do projeto de chatbot que será utilizado posteriormente, é necessário que os recursos complementares estejam devidamente criados e prontos para uso no workspace.  

Além do recurso principal **Azure AI Foundry**, é essencial que o serviço **Azure AI Search** também esteja presente no workspace criado, pois é por meio dele que será possível configurar o modelo utilizado, permitindo que este busque os arquivos fornecidos e os converta em uma base de conhecimento para o agente.  

Sendo assim, foram criados os seguintes recursos:


---

### 1.2 Azure AI Search

Para que o chatbot a ser criado tenha um viés baseado nos documentos fornecidos, é necessário criar o recurso **[Azure AI Search](#azure-ai-search)**. Com este recurso em nosso workspace, será possível realizar o upload de arquivos PDF que servirão como fonte de busca do modelo durante sua configuração.  
![Marketplace Azure AI Search](inputs/imgs/mkp_AzureaiSearch.png)  

As seguintes especificações foram utilizadas na criação do recurso:

#### Informações do Serviço Azure AI Search

| **Configurações Básicas**     |                          |                                                                 |
|-------------------------------|--------------------------|-----------------------------------------------------------------|
| **Nome do Serviço**           | **Localização**          | **Tipo de Serviço**                                             |
| `proj-2-azureaisearch-dio`    |  East US                 | Básico (15 GB/partição, máx. 3 réplicas, máx. 3 partições, até 9 unidades de pesquisa) |

|                               | **Escala**               |                                                                 |
|-------------------------------|--------------------------|-----------------------------------------------------------------|
| **Unidades de Pesquisa**      | **Custo Estimado por Mês** | **Réplicas e Partições**                                        |
| 2/9                           | US$ 150,29               | 2 réplicas e 1 partição                                         |

![Configuração básica Azure AI Search](inputs\imgs\conf_basic_aas.png)  

![Configuração de escala Azure AI Search](inputs/imgs/conf_escala_aas.png)  

As configurações de rede e marcas do serviço foram mantidas como padrão.  
![Configuração de rede Azure AI Search](inputs/imgs/conf_rede_aas.png)  

![Configuração de marcas Azure AI Search](inputs/imgs/conf_marcas_ass.png)  

Com as configurações devidamente realizadas, o serviço foi criado e integrado ao workspace.  
![Implantação bem-sucedida Azure AI Search](inputs/imgs/impl_aas.png)  

---

### 1.3 Azure AI Foundry

Com o serviço **Azure AI Search** instalado e configurado, foi realizada a implantação do serviço **Azure AI Foundry**.  
![Marketplace Azure AI Foundry](inputs/imgs/mkp_AzureAIFoundry.png)  

Ao iniciar a criação do serviço, a plataforma direciona automaticamente para a criação de um **HUB**, responsável por armazenar os projetos que serão desenvolvidos futuramente.

As seguintes especificações foram utilizadas:

#### Informações do Serviço Azure AI Foundry

| **Configurações Básicas**      |            |                 |
|--------------------------------|------------|-----------------|
| **Nome do Serviço**            | **Localização** | **Grupo de Recursos** |
| `proj-2-azureaifoundry-dio`    | East US    | proj-02-dio     |

![Configuração básica Azure AI Foundry](inputs/imgs/conf_basic_aaf.png)  

As demais configurações foram mantidas com os valores padrão.  
![Resumo das configurações Azure AI Foundry](inputs/imgs/conf_resum_aaf.png)  

![Implantação bem-sucedida Azure AI Foundry](inputs/imgs/impl_aaf.png)  

Com os recursos primários devidamente implantados, dá-se início ao processo de criação e treinamento do chatbot.

## 2 Azure AI Foundry Studio
Dentro do ambiente do **Azure AI Foundry**, é apresentada a tela de Overview, na qual são exibidos todos os projetos criados para aquele hub.
![Overview Azure AI Foundry](inputs/imgs/aaf_ovwv_studio.png)  
Nesse menu, é possível não apenas criar um novo projeto para o hub, como também visualizar todos os projetos já existentes. O projeto que será estudado foi nomeado como **"tcchat"**.
![Overview Azure AI Foundry](inputs/imgs/aaf_ovwv_studio_2.png)  

### 2.1 Criação do projeto
Dentro desse menu de Overview, ao clicar na opção de "Novo Projeto", será apresentada uma tela para nomear o novo projeto a ser desenvolvido.
![Criando um novo projeto](inputs/imgs/aaf_create_proj.png)  
Após a criação, o usuário é direcionado ao centro de gerenciamento do projeto, onde será feita a configuração propriamente dita. Nessa mesma tela, já é possível visualizar, além dos endpoints disponíveis, quais são os recursos conectados ao projeto.
![Centro de Gerenciamento de Projeto](inputs/imgs/aaf_gerencia_proj.png)

### 2.2 Criando o agente
Para criar um novo agente para o projeto, devemos acessar a opção de "Models + Endpoints" correspondentes ao projeto desejado.
#### Criando um novo modelo 
Dentro da opção "Models + Endpoints", são apresentados todos os modelos já implantados, bem como os pontos de acesso daquele projeto. Também é apresentada a opção de criar um novo modelo, onde é possível selecionar se o novo modelo a ser criado será um modelo básico ou um modelo ajustado.  

    (Para o projeto em questão, será criado apenas um modelo básico.)

![Model + Endpoint](inputs/imgs/aaf_model_endpoint.png)

![Opções de modelos e endpoints](inputs/imgs/aaf_model_endpoint_2.png)  

### 2.3 Implantando modelos  
Após selecionar o tipo de modelo a ser implantado será apresentado a tela para seleção de modelos de agente disponiveis 
 ![Lista de modelos](inputs/imgs/aff_lista_de_modelos.png)  

**Observação** Durante a implantação do modelo é importante se atentar a quantia de quotas diposniveis, é possível se consultar em QUOTA
![Consulta de cota Azure AI Foundry](inputs\imgs\aff_quota.png)  

No projeto implantado foi utilizado 2 modelos de lingaguem para criação do chat em questão sendo eles `gpt-4o-mini` e `text-embedding-3-large`.
Com os modelos criados e disponíveis para o projeto, enfim pode ser dado inicio ao projeto de criação, no acesso do projeto criado, podemos da inicio a configuração do chatbot. 
Acessando a opção de menu **"Playground"**, é possível inicializar a configuração do modelo do chat a ser utilizado. 
![Visão Geral de projeto Azure AI Foundry](inputs\imgs\aff_visao_geral_proj.png)

### Configurando o chatbot
Na área de Playground, do seu projeto pode ser acessado a opção de playground de chat, nessa área será configurado o comportamento do chat, seu contexto, sua base de pesquisa e realizado os testes de do chat. 

![Playground de chat Azure AI Foundry](inputs\imgs\aaf_playground_chat.png)

Antes de realizar a enganharia de prompt para o chatbot, a ser feito irei selecionar a base de dados PDFs nos quais o chat irá se embasar, para realizar sua pesquisa "inviesada".
### 2.3.1 Adicionando os dados 
selecionando a opção de `Adicionar seus dados`,é possível indicar ao chat qual será a fonte de dados de pesquisa, como a ideia do projeto e realizar um chat que gere insights para um TCC, a ser realizado. Selecionado apção em questão será possível é apresentado a lista de fontes possíveis de pesquisa, no caso será escolhida opção de carregar arquivos. 

![Fonte de dados](inputs\imgs\aff_escolha_de_dados.png)

Com os dados devidamente carregados será solicitado então a configuração do índice, e é nesse paso que utilizaremos o recurso previamente criado de [Serviço de pesquisa](#12-azure-ai-search), foi para isso que o serviço anteriormente criado foi adicionado ao workspace,nesta mesma opção de configuração ainda  é possível que seja escolhido o tipo de computação a ser utilizada para tal.  

![Configuração de índice](inputs\imgs\aff_conf_indice.png)  

No ultimo passo de configuração adicionaremos então o outro modelo criado para o projeto o **"text-embedding-3-large"**

![Configuração de pesquisa](inputs\imgs\aff_conf_pesquisa.png)  

Ainda na configurações definir os métodos de pesquisa na base de dados, se será uma busca vetorizada, palavra chave, semânticas, híbrido ( palavra-chave + busca vetorizada), ou híbrida + semântica. Ainda é possíveis configurações adicionais para o chat a ser criado como quantia de mensagens anteriores, quantidade de respostas máxima por gasto de token, 
temperatura e p, para o caso do chat como desejo que o chat me de insigths de um tcc deixei limitado em 0.56, que dará uma certa liberdade criativa porém não muito.  

![Parâmetros de chat](inputs\imgs\aaf_param_chat.png)  

Após as configurações adicionaremos o contexto para o chat e outras informações nas `instruções e contexto de modelo`


### 2.3.2 Contextualizando o chat
Para a personalização do chat implementado foram utilizada algumas técnicas de enenharia de prompt, RAG(feito no passo anterior, ao inserir os PDFs para formar base de conhecimentos), também foi realizado exemplos de uso mesmo que primários, e algumas váriaveis de uso 
  #### Contexto e instruções de comportamento
  "Identidade e Propósito:
    Você é um assistente de Inteligência Artificial chamado "TcChat!". Seu principal objetivo é ajudar alunos a obter insights, esclarecimentos e informações relevantes sobre trabalhos de conclusão de curso (TCC).

    Cursos Atendidos:

      - Você só deve atender dúvidas relacionadas aos seguintes cursos:

      - Análise e Desenvolvimento de Sistemas

        - Sistemas de Informação

      - Técnico em Sistemas da Informação

      - Se um usuário apresentar dúvidas fora desses cursos, informe gentilmente que seu propósito é auxiliar exclusivamente nas áreas mencionadas.

    Tom e Linguagem:

    - Sua linguagem padrão deve ser técnica e formal.

    - É permitido adotar uma linguagem mais leve e informal apenas para explicações de dúvidas, e somente se o usuário demonstrar abertura para isso durante a conversa.

    - Caso o tom da conversa permita, você pode fazer essa transição de forma gradual.

    Referências:

    - Sempre que fornecer uma explicação técnica, inclua as referências utilizadas, dando preferência a sua base de conhecimento, se possível as referencie com:   (Nome da apostila e o capítulo que contem o tema).

    Respostas Genéricas:

    - Quando receber uma pergunta genérica, ajude o usuário a refinar sua dúvida para que ela fique mais clara, fornecendo contexto ou orientações adicionais para guiar o entendimento.

    Auxílio com Ideias de TCC:

    - Ao ser solicitado para ajudar com ideias de TCC, siga os passos abaixo:

    - Pergunte se o usuário já tem um tema em mente.

    - Se não tiver, pergunte qual é sua área de formação.

    Informe que você atua nas seguintes áreas:

    - Análise e Desenvolvimento de Sistemas

    - Sistemas de Informação

    - Técnico em Sistemas da Informação

    - Se o usuário desejar discutir uma área diferente, verifique se a dúvida continua dentro de sua base de conhecimento técnica.

    - Caso não esteja, reforce educadamente sua área de atuação e limite de propósito."
  
![Parâmetros de chat](inputs\imgs\aff_context_chat.png)  

#### Variáveis e Exemplos de conversa  

Para otimização de alguns casos, principalmente para ínicio de conversação foram configurados tanto exemplos de conversação como váriaveis de utilização. 
![Exemplos de conversa e váriaveis](inputs\imgs\aff_var_exemp.png)  

  #### Exemplo 1:
    #Customer:
      - Olá chat!
    #System:
    - Olá eu sou {{chat}}, estou aqui para lhe ajudar com informações sobre os temas: {{cursos}}. Sobre qual dessas áreas você deseja saber mais?

  #### Exemplo 2:
    #Customer:
      - Olá tcchat!
    #System:
    - Olá estou aqui para lhe ajudar com informações sobre os temas: {{cursos}}. Sobre qual dessas áreas você deseja saber mais?

  #### Exemplo 3:
    #Customer:
      - Queria uma ideia sobre como faço um tcc
    #System:
    - Sobre qual dessas áreas você deseja saber mais? 
    {{cursos}}

### 2.3.4 Exemplos de converssação 
Após a configuração do projeto foi realizado algumas interações com o chat diretamente do playground antes de realizar sua avaliação. 
![Exemplos de interação](inputs\imgs\aaf_conversa_chat.png)  
![Exemplos de interação](inputs\imgs\aaf_conversa_chat_2.png)  
![Exemplos de interação](inputs\imgs\aaf_conversa_chat_3.png)  

## 3. Avaliando o Modelo
Para avaliação do modelo a ser implantado é possível realizar dois tipos de avaliações, manual ou automatizada, para melhor exploração do modelo foram realizadas os 2 tipos avaliações. 

### 3.1 Avaliação manual
Dentro da avaliação manual do modelo a ser implantado consiste em inserir os parâmetros básicos do chat, como os descritos em [Na contextutalização do chat](#232-contextualizando-o-chat), e posteiormente ir inserindo possíveis perguntas manuais conforme o usuário o faria, e em seguida realizar a resposta esperada, nos moldes atuais do projeto, o sistema está com uma grande acuracia. 
![Avaliação de Manual](inputs\imgs\aaf_aval_manual.png)  

### 3.2 Avaliação Automatizada
Ao ser selecionado a `avaliação automatizada`, o sistema irá criar um especie de **"JOB"** para realizar a avaliação do modelo que será implantado. O mesmo é consultado através das avaliações na barra lateral esquerda do sistema. 
![Avaliação de Automatizada](inputs\imgs\aaf_aval_autom.png)  
Com esse serviço é possivél visualizar nos seus inputs quais foram as perguntas e resposta geradas 
![trabalho gerados pela automatizada](inputs\imgs\aff_trabalhos_gerados.png)  
No modelo atual foram alcançadas médias de 4 pontos percentuais, e uma coerência de 5 pontos conforme podemos visualizar nos gráficos gerados pelo trabalho
![Gráficos gerados pela automatizada](inputs\imgs\aaf_graf_autom.png) 


<table style="text-align: center; width: 100%;"> 
 <caption><b>Project skils</b></caption>
  <tr>
    <td style="text-align: center;">
      <img alt="Markdown" src="https://img.shields.io/badge/markdown-%23000000.svg?style=for-the-badge&logo=markdown&logoColor=white"/>
    </td>
    <td style="text-align: center;">
      <img alt="HTML5" src="https://img.shields.io/badge/html5-%23E34F26.svg?style=for-the-badge&logo=html5&logoColor=white"/>
    </td>
    <td style="text-align: center;">
      <img alt="Visual Studio Code" src="https://img.shields.io/badge/VisualStudioCode-0078d7.svg?style=for-the-badge&logo=visual-studio-code&logoColor=white"/>
    </td>
    <td style="text-align: center;">
      <img alt="Git" src="https://img.shields.io/badge/git-%23F05033.svg?style=for-the-badge&logo=git&logoColor=white"/>
    </td>
    <td style="text-align: center;">
      <img alt="GitHub" src="https://img.shields.io/badge/github-%23121011.svg?style=for-the-badge&logo=github&logoColor=white"/>
    </td>
    <td style="text-align: center;">
      <img alt="Azure" src="https://img.shields.io/badge/azure-%230072C6.svg?style=for-the-badge&logo=azure-devops&logoColor=white"/>
    </td>
  </tr>
</table>

