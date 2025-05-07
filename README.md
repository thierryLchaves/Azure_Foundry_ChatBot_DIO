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
- [Funcionalidades](#funcionalidades)
- [Licença](#licenca)


## 1. Workspace-e-Recursos

**Para o início do projeto, foi criada uma workspace nomeada como `proj-02-dio`:**

Para evitar problemas com "recursos indisponíveis", tanto o workspace quanto os demais recursos foram provisionados com a localidade **Brazil South**.  
Essa escolha tornou o processo um pouco mais lento, porém possibilitou o acesso a uma maior disponibilidade de recursos.  
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
| `proj-2-azureaisearch-dio`    |  Brazil South             | Básico (15 GB/partição, máx. 3 réplicas, máx. 3 partições, até 9 unidades de pesquisa) |

|                               | **Escala**               |                                                                 |
|-------------------------------|--------------------------|-----------------------------------------------------------------|
| **Unidades de Pesquisa**      | **Custo Estimado por Mês** | **Réplicas e Partições**                                        |
| 2/9                           | US$ 150,29               | 2 réplicas e 1 partição                                         |

![Configuração básica Azure AI Search](inputs/imgs/conf_basic_aas.png)  

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
| `proj-2-azureaifoundry-dio`    | Brazil South    | proj-02-DIO     |

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

### 2.3 Configurando o modelo 
Após selecionar o tipo de modelo a ser implantado será apresentado a tela para seleção de modelos de agente disponiveis 
 ![Lista de modelos](inputs/imgs/aff_lista_de_modelos.png)  
    Neste projeto serão utilizados 2 tipos de modelos diferentes sendo eles:
      gpt-4o & 

  


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

