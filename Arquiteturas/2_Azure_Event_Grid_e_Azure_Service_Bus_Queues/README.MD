# Lab 2 - Publicar_e_assinar_eventos_do_Event_Grid_e processar_mensagens_de_forma_assíncrona_usando_o_Azure_Service_Bus_Queues


## Diagrama de Arquitetura

![image](../../Arquiteturas/images/Lab2imersao.png)


## Tarefa 1

#### Passo 1: Abra o portal do Azure

1. Abra seu navegador.

2. Na janela do navegador aberta, navegue até o portal do Azure (<https://portal.azure.com>), e entre com a conta que você usará para este laboratório.

    > **Observação**: se esta for a primeira vez que você entra no portal do Azure, será oferecido um tour pelo portal. Selecione **Começar** para pular o tour e começar a usar o portal.

#### Passo 2: Abra o Azure Cloud Shell

 1. No portal do Azure, selecione o ícone do **Cloud Shell** ![Cloud Shell icon](./../images/az204_lab_CloudShell.png) para abrir uma nova sessão do Bash. Se o padrão do Cloud Shell for uma sessão do PowerShell, selecione **PowerShell** e, no menu suspenso, selecione **Bash** .

    > **Observação**: se esta for a primeira vez que você está iniciando o **Cloud Shell** , quando solicitado a selecionar **Bash** ou **PowerShell** , selecione **Bash** . Quando você receber a mensagem Você não tem armazenamento montado , selecione a assinatura que você está usando neste laboratório e, em seguida, selecione Criar armazenamento .

 2. No portal do Azure, no prompt de comando do **Cloud Shell** , execute o seguinte comando para obter a versão da ferramenta Azure Command-Line Interface (Azure CLI):

    ```bash
    az --version
    ```

#### Passo 3: Revise o registro do provedor Microsoft.EventGrid

1. No **painel CLoud Shell**, execute o seguinte comando para obter uma lista de subgrupos e comandos no nível raiz da CLI do Azure:

    ```bash
    az --help
    ```

2. No **painel CLoud Shell**, execute o seguinte comando para obter uma lista dos comandos disponíveis para provedores de recursos:

    ```bash
    az provider --help
    ```

3. No **painel CLoud Shell**, execute o seguinte comando para listar todos os provedores registrados atualmente:

    ```bash
    az provider list
    ```

4. No **painel CLoud Shell**, execute o seguinte comando para listar apenas os namespaces dos provedores registrados no momento:

    ```bash
    az provider list --query "[].namespace"
    ```

5. No **painel CLoud Shell**, revise a lista de provedores registrados no momento. Observe que o provedor **Microsoft.EventGrid** está atualmente incluído na lista de provedores.

6. Feche o **painel Cloud Shell**.

#### Passo 4: Criar um tópico EventGrid personalizado
1. Criar Grupo de Recursos


    **Cloud Shell**

    ```bash
    az group create --name "PubSubEvents" --location "eastus"
    ```
2. Criar tópico do EventGrid

    **Cloud Shell**
    ```bash
    az eventgrid topic create -g "PubSubEvents" --name "hrttopic<yourname>" -l "eastus" --public-network-access enabled 
    ```

    Capture o valor do id do recurso como mostra a figura abaixo. Você usará esse valor posteriormente no laboratório.


    ![image](../../Arquiteturas/images/idtopicoeventgridlab2.png)

3. Criar Plano de Serviço de Aplicativo

    **Cloud Shell**
    ```bash
    az appservice plan create -g "PubSubEvents" -n EventPlan --is-linux --sku P1v2
    ```
3. Criar Aplicativo Web

    **Cloud Shell**
    ```bash
    az webapp create -g "PubSubEvents" -p EventPlan -n "eventviewer<yourname>" -i microsoftlearning/azure-event-grid-viewer:latest
    ```
    > **Observação**: Aguarde o Azure concluir a criação do aplicativo Web antes de continuar com o laboratório.

    Capture o valor do link da URL como mostra a figura abaixo. Você usará esse valor posteriormente no laboratório.

    ![image](../../Arquiteturas/images/urlwebservicelab2.png)

#### Passo 5: Criar uma assinatura do Event Grid

**Cloud Shell**
```bash
    az eventgrid event-subscription create --name "basicsub" --source-resource-id "/subscriptions/<subscriptionID>/resourceGroups/PubSubEvents/providers/Microsoft.EventGrid/topics/hrttopic<yourname>" --endpoint "https://eventviewer<yourname>.azurewebsites.net/api/updates" --endpoint-type "webhook" 
```

+ No parâmetro **--source-resource-id** , insira o valor do ID do tópico do EventGrid que você guardou anteriormente.
    
+ No parâmetro **endpoint** , insira o valor de URL do aplicativo Web que você guardou anteriormente, certifique-se de que ele usa um prefixo https:// , e adicione o sufixo /api/updates. Por exemplo, se o valor da URL do seu aplicativo Web for http://eventviewerstudent.azurewebsites.net/, seu endpoint será https://eventviewerstudent.azurewebsites.net/api/updates.
    
#### Passo 6: Guarde as credenciais da assinatura

1. No painel de navegação do portal do Azure, selecione **Resource groups**.

2. Na folha **Resource groups**, selecione o grupo de recursos **PubSubEvents** que você criou anteriormente neste laboratório.

3. Na folha **PubSubEvents**, selecione o tópico Event Grid  **hrtopic**_[yourname]_  que você criou anteriormente neste laboratório.

4. Na folha **Event Grid Topic**, guarde o valor do campo **Topic Endpoint**. Você usará esse valor posteriormente no laboratório.

5. Na categoria **Settings**, selecione o link **Access keys**.

6. Na seção **Access keys**, registre o valor da caixa de texto **Key 1**. Você usará esse valor posteriormente no laboratório.

#### Passo 7: Publicar eventos do Event Grid do .NET

_Crie um Projeto .NET_

1. Crie uma pasta em seu computador chamada **work** em C:

2. Abra o **Visual Studio Code** em seu computador, e abra a pasta.

3. Abra o powershell no VScode e faça git clone do projeto.

    ```
    git clone https://github.com/luizjrlopes/ComunidadeCloudExpertSchool.git 
    ```
4. Finalizado o git clone, no terminal, entre na pasta **EventPublisher** em _(..\eventgrid\Starter\EventPublisher\)_ como na figura abaixo.


    ![image](../../Arquiteturas/images/vcscodetela1.png)

5. Para rodar os comandos a seguir é necessário possuir o dotnet core 3.1. Caso não tenha instalado, baixe e instale a partir do link: [Download .NET Core 3.1](https://dotnet.microsoft.com/en-us/download/dotnet).

    >**Observação**: Caso tenha instalado o .NET 6.0 poderá haver erro devido a mudança de bibliotecas. Desistale e instale o .NET Core 3.1.

6. De volta ao terminal do VScode, rode o seguinte comando para criar um novo projeto .NET chamado **EventPublisher** na pasta atual:

    ```powershell
    dotnet new console --name EventPublisher --output .
    ```

    > **Observação**: o comando **dotnet new** criará um novo projeto de **console** em uma pasta com o mesmo nome do projeto.

7. Execute o seguinte comando para importar a versão 4.1.0 do **Azure.Messaging.EventGrid** do NuGet:

    ```powershell
    dotnet add package Azure.Messaging.EventGrid --version 4.1.0
    ```

    > **Observação**: o comando **dotnet add package** adicionará o pacote **Microsoft.Azure.EventGrid** do NuGet. Para mais informações, acesse [Azure.Messaging.EventGrid](https://www.nuget.org/packages/Azure.Messaging.EventGrid/4.1.0).

8. Execute o seguinte comando para compilar o aplicativo Web .NET:

    ```powershell
    dotnet build
    ```

9. Selecione **Kill Terminal** ou o ícone **Recycle Bin** para fechar o terminal atualmente aberto e quaisquer processos associados.

#### Passo 7: Modifique a classe Program para se conectar ao Event Grid

1. No painel **Explorer** da janela **Visual Studio Code**, abra o arquivo **Program.cs**.

2. Na guia do editor de código do arquivo **Program.cs** , exclua todo o código do arquivo existente.

3. Adicione a seguinte linha de código para importar os namespaces **Azure** e **Azure.Messaging.EventGrid** do pacote **Azure.Messaging.EventGrid** importado do NuGet:

    ```csharp
    using Azure;
    using Azure.Messaging.EventGrid;
    ```

4. Adicione as seguintes linhas de código para adicionar a diretiva **using** para os namespaces internos que serão usados ​​neste arquivo:

    ```csharp
    using System;
    using System.Threading.Tasks;
    ```

5. Digite o seguinte código para criar uma nova classe **Program**:

    ```csharp
    public class Program
    {
    }
    ```

6. Na classe **Program**, insira a seguinte linha de código para criar uma nova constante de string chamada **topicEndpoint**:

    ```csharp
    private const string topicEndpoint = "";
    ```

7. Atualize a constante de string **topicEndpoint** definindo seu valor para o **Topic Endpoint** do tópico Event Grid que você guardou anteriormente neste laboratório.

8. Na classe **Program**, insira a seguinte linha de código para criar uma nova constante de string chamada **topicKey**:

    ```csharp
    private const string topicKey = "";
    ```

9. Atualize a constante de string **topicKey** definindo seu valor como a **Key** do tópico Event Grid que você guardou anteriormente neste laboratório.

10. Na classe **Program**, insira o código a seguir para criar um novo método assíncrono **Main**:

    ```csharp
    public static async Task Main(string[] args)
    {
    }
    ```

11. O arquivo **Program.cs**, ao final deve possuir as seguintes linhas de código:

    ```csharp
    using System;
    using System.Threading.Tasks;
    using Azure;
    using Azure.Messaging.EventGrid;
    public class Program
    {
        private const string topicEndpoint = "<topic-endpoint>";
        private const string topicKey = "<topic-key>";
        public static async Task Main(string[] args)
        {
        }
    }
    ```

#### Passo 8: Publicar novos eventos

1. No método **Main**, execute as seguintes ações para publicar uma lista de eventos em seu **topic Endpoint**:

    a. Adicione a seguinte linha de código para criar uma nova variável chamada **endpoint** do tipo **Uri**, usando a constante string **topicEndpoint** como um parâmetro de construtor:

    ```csharp
    Uri endpoint = new Uri(topicEndpoint); 
    ```

    b. Adicione a seguinte linha de código para criar uma nova variável chamada **credential** do tipo **[AzureKeyCredential](https://docs.microsoft.com/dotnet/api/azure.azurekeycredential)**, usando a constante string **topicKey** como um parâmetro de construtor:

    ```csharp
    AzureKeyCredential credential = new AzureKeyCredential(topicKey);
    ```

    c. Adicione a seguinte linha de código para criar uma nova variável chamada **client** do tipo **[EventGridPublisherClient](https://docs.microsoft.com/dotnet/api/azure.messaging.eventgrid.eventgridpublisherclient)**, usando as variáveis ​​**endpoint** e **credential** como parâmetros do construtor:

    ```csharp
    EventGridPublisherClient client = new EventGridPublisherClient(endpoint, credential);
    ```

    d. Adicione o seguinte bloco de código para criar uma nova variável chamada **firstEvent** do tipo **[EventGridEvent](https://docs.microsoft.com/dotnet/api/azure.messaging.eventgrid.eventgridevent)** e preencha essa variável com dados de amostra:

    ```csharp
    EventGridEvent firstEvent = new EventGridEvent(
        subject: $"New Employee: Alba Sutton",
        eventType: "Employees.Registration.New",
        dataVersion: "1.0",
        data: new
        {
            FullName = "Alba Sutton",
            Address = "4567 Pine Avenue, Edison, WA 97202"
        }
    );
    ```

    e. Adicione o seguinte bloco de código para criar uma nova variável chamada **secondEvent** do tipo **[EventGridEvent](https://docs.microsoft.com/dotnet/api/azure.messaging.eventgrid.eventgridevent)** e preencha essa variável com dados de amostra:

    ```csharp
        EventGridEvent secondEvent = new EventGridEvent(
            subject: $"New Employee: Alexandre Doyon",
            eventType: "Employees.Registration.New",
            dataVersion: "1.0",
            data: new
            {
                FullName = "Alexandre Doyon",
                Address = "456 College Street, Bow, WA 98107"
            }
        );
    ```

    f. Adicione a seguinte linha de código para invocar de forma assíncrona o **[EventGridPublisherClient.SendEventAsync](https://docs.microsoft.com/dotnet/api/azure.messaging.eventgrid.eventgridpublisherclient.sendeventasync)** usando a variável **firstEvent** como parâmetro:

    ```csharp
    await client.SendEventAsync(firstEvent);
    ```

    g. Adicione a seguinte linha de código para renderizar a mensagem **"Primeiro evento publicado"** no console:

    ```csharp
    Console.WriteLine("Primeiro evento publicado");
    ```

    h. Adicione a seguinte linha de código para invocar de forma assíncrona o **[EventGridPublisherClient.SendEventAsync](https://docs.microsoft.com/dotnet/api/azure.messaging.eventgrid.eventgridpublisherclient.sendeventasync)** usando a variável **secondEvent** como parâmetro:

    ```csharp
    await client.SendEventAsync(secondEvent);
    ```

    i. Adicione a seguinte linha de código para renderizar a mensagem **"Segundo evento publicado"** no console:

    ```csharp
    Console.WriteLine("Segundo evento publicado");
    ```

1. Revise o método **Main**, que agora deve incluir:

    ```csharp
    public static async Task Main(string[] args)
    {
        Uri endpoint = new Uri(topicEndpoint);
        AzureKeyCredential credential = new AzureKeyCredential(topicKey);
        EventGridPublisherClient client = new EventGridPublisherClient(endpoint, credential);
        EventGridEvent firstEvent = new EventGridEvent(
            subject: $"New Employee: Alba Sutton",
            eventType: "Employees.Registration.New",
            dataVersion: "1.0",
            data: new
            {
                FullName = "Alba Sutton",
                Address = "4567 Pine Avenue, Edison, WA 97202"
            }
        );
        EventGridEvent secondEvent = new EventGridEvent(
            subject: $"New Employee: Alexandre Doyon",
            eventType: "Employees.Registration.New",
            dataVersion: "1.0",
            data: new
            {
                FullName = "Alexandre Doyon",
                Address = "456 College Street, Bow, WA 98107"
            }
        );
        await client.SendEventAsync(firstEvent);
        Console.WriteLine("Primeiro evento publicado");
        await client.SendEventAsync(secondEvent);
        Console.WriteLine("Segundo evento publicado");
    }
    ```

1. Salve o arquivo **Program.cs** .

1. No terminal do **Visual Studio Code**, execute o seguinte comando para executar o aplicativo Web .NET:

    ```powershell
    dotnet run
    ```

    >**Observação**: se houver algum erro de compilação, revise o arquivo **Program.cs** na pasta **\Solution\\EventPublisher** .

1. Observe a saída da mensagem de sucesso do aplicativo de console em execução no momento.

1. Selecione **Kill Terminal** ou o ícone **Recycle Bin** para fechar o terminal atualmente aberto e quaisquer processos associados.

#### Passo 9: Observe published events

1. Retorne à janela do navegador com o aplicativo Web **Azure Event Grid Viewer**.

1. Revise os eventos **Employees.Registration.New** que foram criados pelo seu aplicativo de console.

1. Selecione qualquer um dos eventos e revise seu conteúdo JSON.

1. Retorne ao portal do Azure.

