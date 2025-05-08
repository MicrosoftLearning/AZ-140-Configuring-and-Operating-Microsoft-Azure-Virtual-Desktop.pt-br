---
lab:
  title: 'Laboratório: Criar imagens de host de sessão personalizadas usando modelos de imagem'
  module: 'Module 1.5: Create and manage session host images'
---

# Laboratório — Crie imagens de host de sessão personalizadas usando modelos de imagem
# Manual de laboratório do aluno

## Dependências do laboratório

- O nome da assinatura do Azure que você usará nesse laboratório
- Uma conta de usuário do Microsoft Entra com a função Proprietário na assinatura do Azure que você usará neste laboratório e com permissões suficientes para unir dispositivos ao locatário do Entra associado a essa assinatura do Azure.

## Tempo estimado

90 minutos (cerca de 45 minutos é o tempo de espera para que a compilação da imagem seja concluída)

## Cenário do laboratório

Você planeja implementar um ambiente de Área de Trabalho Virtual do Azure. Você precisa usar imagens de máquina virtual personalizadas ao implantar hosts de sessão da Área de Trabalho Virtual do Azure.

## Objetivos
  
Depois de realizar este laboratório, você será capaz de:

- Criar imagens de host de sessão personalizadas para o Área de Trabalho Virtual do Azure usando modelos de imagem

## Arquivos do laboratório

- Nenhum

## Instruções

### Exercício 1: Criar imagens de host de sessão personalizadas usando modelos de imagem

As principais tarefas desse exercício são as seguintes:

1. Registrar provedores de recursos necessários
1. Criar uma identidade gerenciada atribuída ao usuário
1. Criar uma função personalizada de RBAC (controle de acesso baseado em função) do Azure
1. Configurar permissões nos recursos relacionados ao provisionamento de imagem do host
1. Criar uma instância da Galeria de Computação do Azure e uma definição de imagem
1. Criar um modelo de imagem personalizada
1. Criar uma imagem personalizada
1. Implantar hosts de sessão usando uma imagem personalizada

> **Observação**: antes de criar um modelo de imagem personalizado, você precisa atender a uma série de pré-requisitos, incluindo:

- Registrar todos os provedores de recursos necessários
- Criar uma identidade gerenciada atribuída ao usuário
- Conceder permissões necessárias à identidade gerenciada atribuída pelo usuário usando uma função de RBAC (controle de acesso baseado em função) personalizada do Azure
- Se você pretende distribuir a imagem usando a Galeria de Computação do Azure, você precisa criar sua instância junto com uma definição de imagem

#### Tarefa 1: Registrar os provedores de recursos necessários

1. Se necessário, no computador do laboratório, inicie um navegador da Web, navegue até o portal do Azure e entre fornecendo credenciais de uma conta de usuário com a função Proprietário na assinatura que você usará neste laboratório.

    > **Observação**: use as credenciais da conta `User1-` listada na guia Recursos no lado direito da janela da sessão de laboratório.

1. No portal do Azure, inicie uma sessão do PowerShell no Azure Cloud Shell.

    > **Observação**: se solicitado, no painel **Introdução**, na lista suspensa **Assinatura**, selecione o nome da assinatura do Azure que você está usando neste laboratório e selecione **Aplicar**.

1. Na sessão do PowerShell no painel Azure Cloud Shell, execute o seguinte comando para registrar o provedor de recursos **Microsoft.DesktopVirtualization**:

    ```powershell
    Register-AzResourceProvider -ProviderNamespace Microsoft.DesktopVirtualization
    Register-AzResourceProvider -ProviderNamespace Microsoft.VirtualMachineImages
    Register-AzResourceProvider -ProviderNamespace Microsoft.Storage
    Register-AzResourceProvider -ProviderNamespace Microsoft.Compute
    Register-AzResourceProvider -ProviderNamespace Microsoft.Network
    Register-AzResourceProvider -ProviderNamespace Microsoft.KeyVault
    Register-AzResourceProvider -ProviderNamespace Microsoft.ContainerInstance
    ```

    > **Observação**: não espere o registro ser concluído. Isso pode levar cerca de cinco minutos.

1. Feche o painel do Azure Cloud Shell.

#### Tarefa 2: Criar uma identidade gerenciada atribuída pelo usuário

1. No computador do laboratório, no navegador da Web que exibe o portal do Azure, pesquise e selecione **Identidades gerenciadas**.
1. Na página **Identidades gerenciadas**, selecione **+ Criar**.
1. Na guia **Básico** da página **Criar identidade gerenciada atribuída pelo usuário**, especifique as seguintes configurações e selecione **Revisar + criar**:

    > **Observação**: ao definir o valor **Nome**, alterne para a guia Recursos no lado direito da janela da sessão de laboratório e identifique a sequência de caracteres entre *Usuário1-* e o caractere *@*. Use esta cadeia de caracteres para substituir o espaço reservado *random*.

    |Configuração|Valor|
    |---|---|
    |Assinatura|O nome da assinatura do Azure que você está usando neste laboratório|
    |Grupo de recursos|O nome de um novo grupo de recursos **az140-15a-RG**|
    |Region|O nome da região do Azure onde você deseja implantar seu ambiente de Área de Trabalho Virtual do Azure|
    |Nome|**az140**-*random*-**uami**|

1. Na guia **Revisar + criar**, selecione **Criar**.

    >**Observação**: não espere a conclusão do provisionamento da identidade gerenciada atribuída ao usuário. Isso deve levar apenas alguns segundos.

#### Tarefa 3: Criar uma função personalizada de RBAC (controle de acesso baseado em função) do Azure

>**Observação**: a função personalizada de RBAC (controle de acesso baseado em função) do Azure será usada para atribuir permissões apropriadas à identidade gerenciada atribuída pelo usuário criada na tarefa anterior.

1. No computador do laboratório, no navegador da Web que exibe o portal do Azure, inicie uma sessão do PowerShell no Azure Cloud Shell.

1. Na sessão do PowerShell no painel Azure Cloud Shell, execute o seguinte comando para identificar o valor da propriedade **Id** da assinatura do Azure usada para este laboratório e armazene-o na variável **$subscriptionId**:

    ```powershell
    $subscriptionId = (Get-AzSubscription).Id
    ```

1. Execute o comando a seguir para criar a definição de função da nova função personalizada, incluindo seu valor de escopo atribuível e armazene-o na variável **$jsonContent** (certifique-se de substituir o espaço reservado *random* pela mesma cadeia de caracteres que você identificou na tarefa anterior):

    ```powershell
    $jsonContent = @"
    {
      "Name": "Desktop Virtualization Image Creator (random)",
      "IsCustom": true,
      "Description": "Create custom image templates for Azure Virtual Desktop images.",
      "Actions": [
        "Microsoft.Compute/galleries/read",
        "Microsoft.Compute/galleries/images/read",
        "Microsoft.Compute/galleries/images/versions/read",
        "Microsoft.Compute/galleries/images/versions/write",
        "Microsoft.Compute/images/write",
        "Microsoft.Compute/images/read",
        "Microsoft.Compute/images/delete"
      ],
      "NotActions": [],
      "DataActions": [],
      "NotDataActions": [],
      "AssignableScopes": [
        "/subscriptions/$subscriptionId",
        "/subscriptions/$subscriptionId/resourceGroups/az140-15b-RG"
      ]
    }
    "@
    ```

1. Execute o seguinte comando para armazenar o conteúdo da variável **$jsonContent** em um arquivo chamado **CustomRole.json**:

    ```powershell
    $jsonContent | Out-File -FilePath 'CustomRole.json'
    ```

1. Execute o seguinte comando para criar a função personalizada:

    ```powershell
    New-AzRoleDefinition -InputFile ./CustomRole.json
    ```

1. Feche o painel do Azure Cloud Shell.

#### Tarefa 4: Definir permissões nos recursos relacionados ao provisionamento de imagem do host

1. No computador do laboratório, no navegador da Web que exibe o portal do Azure, pesquise e selecione **Grupos de recursos** e, na página **Grupos de recursos**, selecione **+ Criar**.
1. Na guia **Básico** da página **Criar um grupo de recursos**, especifique as seguintes configurações e selecione **Revisar + criar**:

    |Configuração|Valor|
    |---|---|
    |Assinatura|O nome da assinatura do Azure que você está usando neste laboratório|
    |Grupo de recursos|O nome de um novo grupo de recursos **az140-15b-RG**|
    |Region|O nome da região do Azure onde você deseja implantar seu ambiente de Área de Trabalho Virtual do Azure|

1. Na guia **Revisar + criar**, selecione **Criar**.
1. Atualize a página **Grupos de recursos** e, na lista de grupos de recursos, selecione **az140-15b-RG**.
1. Na página **az140-15b-RG**, no menu de navegação vertical, selecione **Controle de acesso (IAM)**.
1. Na página **az140-15b-RG\|Controle de acesso (IAM)**, selecione **+ Adicionar** e, no menu suspenso, selecione **Adicionar atribuição de função**.
1. Na guia **Função** da página **Adicionar atribuição de função**, certifique-se de que a guia **Funções de função de trabalho** esteja selecionada, na caixa de texto de pesquisa, insira **Criador de imagem de virtualização de desktop** (*aleatório*), na lista de resultados, selecione **Criador de imagem de virtualização de desktop** (*aleatório*) e, em seguida, selecione **Avançar**.

    >**Observação**: certifique-se de substituir o espaço reservado *random* pela mesma cadeia de caracteres que você usou ao definir a nova função RBAC personalizada.

1. Na guia **Membros** da página **Adicionar atribuição de função**, selecione a opção **Identidade gerenciada**, clique em **+ Selecionar membros**, no painel **Selecionar identidades gerenciadas**, na lista suspensa **Identidade gerenciada**, selecione **Identidade gerenciada atribuída pelo usuário**, na lista de identidades gerenciadas atribuídas pelo usuário, selecione **az140**-*random*-**uami** (onde o placehodler *random* representa a mesma string que você usou ao definir a nova função RBAC personalizada) e então clique em **Selecionar**.
1. De volta à aba **Membros** da página **Adicionar atribuição de função**, selecione **Revisar + atribuir**.
1. Na guia **Revisão + atribuição**, selecione **Examinar + atribuir**. 

#### Tarefa 5: Criar uma instância da Galeria de Computação do Azure e uma definição de imagem

1. No computador do laboratório, no navegador da Web que exibe o portal do Azure, pesquise e selecione **Galerias de computação do Azure** e, na página **Galerias de computação do Azure**, selecione **+ Criar**.
1. Na guia **Básico** da página **Criar galeria de computação do Azure**, especifique as seguintes configurações e selecione **Avançar: Método de compartilhamento**:

    |Configuração|Valor|
    |---|---|
    |Assinatura|O nome da assinatura do Azure que você está usando neste laboratório|
    |Grupo de recursos|**az140-15b-RG**|
    |Nome|**az14015computegallery**|
    |Region|O nome da região do Azure onde você deseja implantar seu ambiente de Área de Trabalho Virtual do Azure|

1. Na guia **Compartilhamento** da página **Criar galeria de computação do Azure**, deixe a opção padrão **Controle de acesso baseado em função (RBAC)** selecionada e, em seguida, selecione **Revisar + criar**.
1. Na guia **Revisar + criar**, selecione **Criar**.

    >**Observação**: Aguarde o processo de provisionamento ser concluído. Isso deverá levar menos de 1 minuto.

1. No computador do laboratório, no navegador da Web que exibe o portal do Azure, pesquise e selecione **Galerias de computação do Azure** e, na página **Galerias de computação do Azure**, selecione **az14015computegallery**. 
1. Na página **az14015computegallery**, selecione **+ Adicionar** e, no menu suspenso, selecione **+ Definição de imagem de VM**. 
1. Na guia **Básico** da página **Criar definição de imagem de VM**, especifique as seguintes configurações (deixe as outras configurações com seus valores padrão) e selecione **Avançar: Versão**:

    |Configuração|Valor|
    |---|---|
    |Region|O nome da região do Azure onde você deseja implantar seu ambiente de Área de Trabalho Virtual do Azure|
    |Nome da definição de imagem da VM|**az14015imagedefinition**|
    |Tipo do SO|**Windows**|
    |Tipo de segurança|**Lançamento confiável suportado**|
    |Estado do sistema operacional|**Generalizado**|
    |Editor|**MicrosoftWindowsDesktop**|
    |Oferta|**Windows-11**|
    |SKU|**win11-23h2-avd-m365**|

    > **Observação**: a geração de VM é definida automaticamente como Gen2, porque as máquinas virtuais Gen 1 não são suportadas com o tipo de segurança Confidencial e Confidencial.

1. Na guia **Versão** da página **Criar definição de imagem de VM**, deixe as configurações inalteradas e selecione **Avançar: Opções de publicação**.

    > **Observação**: você não deve criar a versão da imagem da VM neste estágio. Isso será feito pela Área de Trabalho Virtual do Azure.

1. Na guia **Opções de publicação** da página **Criar definição de imagem de VM**, deixe as configurações inalteradas e selecione **Revisar + criar**.
1. Na guia **Revisar + criar** da página **Criar definição de imagem de VM**, selecione **Criar**.

    > **Observação**: Aguarde o processo de provisionamento ser concluído. Essa etapa geralmente leva menos de um minuto.

#### Tarefa 6: Criar um modelo de imagem personalizado

1. No computador do laboratório, no navegador da Web que exibe o portal do Azure, pesquise e selecione **Área de Trabalho Virtual do Azure**, na página **Área de Trabalho Virtual do Azure**, na seção **Gerenciar** do menu de navegação vertical, selecione **Modelos de imagem personalizados** e, na página **Área de Trabalho Virtual do Azure \| Modelos de imagem personalizados**, selecione **+ Adicionar modelo de imagem personalizado**. 
1. Na guia **Básico** da página **Criar modelo de imagem personalizado**, especifique as seguintes configurações e selecione **Avançar**:

    > **Observação**: ao definir a propriedade **Identidade gerenciada**, certifique-se de substituir o espaço reservado *aleatório* pela mesma cadeia de caracteres identificada anteriormente neste exercício.

    |Configuração|Valor|
    |---|---|
    |Nome do modelo|**az140-15b-imagetemplate**|
    |Importar do modelo existente|**Não**|
    |Assinatura|O nome da assinatura do Azure que você está usando neste laboratório|
    |Grupo de recursos|**az140-15b-RG**|
    |Localidade|O nome da região do Azure onde você deseja implantar seu ambiente de Área de Trabalho Virtual do Azure|
    |Identidade gerenciada|**az140**-*random*-**uami**|

1. Na guia **Imagem de origem** da página **Criar modelo de imagem personalizado**, especifique as seguintes configurações e selecione **Avançar**:

    |Configuração|Valor|
    |---|---|
    |Tipo de origem|**Imagem de plataforma (marketplace)**|
    |Selecionar a imagem|**Windows 11 Enterprise multi-sessão, versão 23H2 + Microsoft 365 Apps**|

1. Na guia **Destinos de distribuição** da página **Criar modelo de imagem personalizado**, especifique as seguintes configurações (deixe as outras configurações com seus valores padrão) e selecione **Avançar**:

    |Configuração|Valor|
    |---|---|
    |Galeria de Computação do Azure|Habilitado|
    |Nome da galeria|**az14015computegallery**|
    |Definição de imagem da galeria|**az14015imagedefinition**|
    |Imagem de versão da galeria|**1.0.0**|
    |Nome de saída da execução|**az140-15-image-1.0.0**|
    |Regiões de replicação|O nome da região do Azure onde você deseja implantar seu ambiente de Área de Trabalho Virtual do Azure|
    |Excluir do mais recente|**Não**|
    |Tipo de conta de armazenamento|**Standard_LRS**|

    > **Observação**: você pode usar a propriedade **Regiões de replicação** para acomodar compilações multirregionais. Definir **Excluir do mais recente** como **Sim** impediria que esta versão da imagem fosse usada quando **mais recente** fosse especificado como a versão do elemento **ImageReference** durante a criação da VM.

1. Na guia **Propriedades da compilação** da página **Criar modelo de imagem personalizado**, especifique as seguintes configurações (deixe as outras configurações com seus valores padrão) e selecione **Avançar**:

    |Configuração|Valor|
    |---|---|
    |Tempo limite de compilação|**120**|
    |Tamanho de VM de compilação|**Standard_DC2s_v3**|
    |Tamanho do disco do sistema operacional (GB)|**127**|
    |Grupo de preparo|**az140-15c-RG**|
    |VNET|Deixar não definido|

    > **Observação**: **Grupo de preparação** é o grupo de recursos usado para preparar recursos para criar a imagem e armazenar logs. Se você não fornecer o nome, ele será gerado automaticamente. Se o nome da **VNet** não estiver definido, um nome temporário será criado, juntamente com um endereço IP público para a VM usada para criar a compilação.

    > **Importante**: certifique-se de ter um número suficiente de vCPUs disponíveis para o tamanho da VM de compilação especificado. Caso contrário, escolha um tamanho diferente ou solicite um aumento de cota.

1. Na guia **Personalização** da página **Criar modelo de imagem personalizado**, selecione **+ Adicionar script integrado**. 
1. No painel **Selecionar scripts integrados**, revise as opções disponíveis agrupadas em scripts específicos do sistema operacional, scripts da Área de Trabalho Virtual do Azure, scripts da anexação de aplicativo MSIX, scripts do aplicativo e scripts relacionados às atualizações do Windows e selecione as seguintes entradas:

   - **Redirecionamento de fuso horário**: permite que o cliente use seu fuso horário dentro de uma sessão em hosts de sessão
   - **Desabilitar Sensor de Armazenamento**: impede que o Sensor de Armazenamento afete negativamente os hosts de sessão ao detectar falsamente condições de pouco espaço livre em disco
   - **Habilitar proteção de captura de tela** com **Bloquear captura de tela no cliente e no servidor**: bloqueia ou oculta conteúdo remoto em capturas de tela e compartilhamento de tela

1. No painel **Selecionar scripts integrados**, selecione **Salvar**.

    > **Observação**: você tem a opção de adicionar seus próprios scripts. Para obter exemplos, considere fazer referência aos scripts integrados, como [Redirecionamento de fuso horário](https://raw.githubusercontent.com/Azure/RDS-Templates/master/CustomImageTemplateScripts/CustomImageTemplateScripts_2024-03-27/TimezoneRedirection.ps1), [Desativar o Sensor de Armazenamento](https://raw.githubusercontent.com/Azure/RDS-Templates/master/CustomImageTemplateScripts/CustomImageTemplateScripts_2024-03-27/DisableStorageSense.ps1) ou [Ativar proteção de captura de tela](https://raw.githubusercontent.com/Azure/RDS-Templates/master/CustomImageTemplateScripts/CustomImageTemplateScripts_2024-03-27/ScreenCaptureProtection.ps1).

1. De volta à guia **Personalização** da página **Criar modelo de imagem personalizado**, selecione **Avançar**.
1. Na guia **Rótulos** da página **Criar modelo de imagem personalizado**, selecione **Avançar**.
1. Na guia **Revisar + criar** da página **Criar modelo de imagem personalizado**, selecione **Criar**.

    > **Observação**: aguarde a criação do modelo. Isso pode levar alguns minutos. Atualize a página **Área de Trabalho Virtual do Azure \| Modelos de imagem personalizados** para revisar o status do modelo.

#### Tarefa 7: Criar uma imagem personalizada

> **Observação**: as tarefas restantes deste laboratório são opcionais, pois envolvem um tempo de espera bastante longo. 

1. No computador do laboratório, no navegador da Web que exibe o portal do Azure, na página **Área de Trabalho Virtual do Azure \| Modelo de imagem personalizado**, selecione **az140-15b-imagetemplate**.
1. Na página **az140-15b-imagetemplate**, selecione **Iniciar compilação**.

    > **Observação**: aguarde a criação da compilação. O tempo real para concluir o processo de construção pode variar, mas com as configurações fornecidas nas instruções do laboratório, ele deve ser concluído em 45 minutos. Atualize a página a cada poucos minutos e monitore o valor **Estado de execução da compilação** na seção **Informações básicas** da página **az 140-15b-image template**. 

    > **Observação**: o estado de execução da compilação deve mudar em algum momento de **Em execução - Construindo** para **Em execução - Distribuindo** e finalmente para **Bem-sucedido**.

    > **Observação**: enquanto aguarda a conclusão da compilação, revise o conteúdo do grupo de recursos de preparação **az140-15c-RG**, onde os recursos de compilação, incluindo a máquina virtual da compilação, uma rede virtual, grupo de segurança de rede, cofre de chaves, instantâneo, instância de contêiner e conta de armazenamento são provisionados automaticamente. 

1. No computador do laboratório, no navegador da Web que exibe o portal do Azure, pesquise e selecione **Grupos de recursos** e, na página **Grupos de recursos**, selecione **az140-15c-RG**.
1. Na página **az140-15c-RG**, na seção **Recursos**, observe os recursos provisionados automaticamente.
1. Retorne à página **az140-15b-imagetemplate** e monitore o progresso da construção. 

    > **Observação**: como alternativa, você pode usar o **Registro de atividades** para acompanhar a conclusão do processo de compilação. A ação na qual você deve se concentrar é **Executar um modelo de imagem de VM para produzir sua saída**. Seu status deve mudar em algum momento de **Aceito** para **Bem-sucedido**.

1. Após a conclusão da compilação, no computador do laboratório, no navegador da Web que exibe o portal do Azure, pesquise e selecione **Galerias de Computação do Azure** e, na página **Galerias de Computação do Azure**, selecione **az14015computegallery**. 
1. Na **az14015computegallery**, na guia **Definições**, selecione **az14015imagedefinition**.
1. Na página **az14015imagedefinition**, na guia **Versões**, revise as informações sobre a imagem **1.0.0 (versão mais recente)** .

#### Tarefa 8: Implantar hosts de sessão usando uma imagem personalizada

> **Observação**: opcionalmente, considere passar pelos estágios iniciais de implantação de hosts de sessão da Área de Trabalho Virtual do Azure usando a imagem personalizada que você criou. 

1. No computador do laboratório, no navegador da Web que exibe o portal do Azure, pesquise e selecione **Redes virtuais** e, na página **Redes virtuais**, selecione **Criar +**
1. Na guia **Noções básicas** da página **Criar rede virtual**, especifique as seguintes configurações e selecione **Avançar**:

    |Configuração|Valor|
    |---|---|
    |Assinatura|O nome da assinatura do Azure que você está usando neste laboratório|
    |Grupo de recursos|O nome de um novo grupo de recursos **az140-15d-RG**|
    |Nome da rede virtual|**az140-vnet15d**|
    |Region|O nome da região do Azure onde você deseja implantar o ambiente de área de trabalho virtual do Azure|

1. Na guia **Segurança**, aceite as configurações padrão e selecione **Avançar**.
1. Na guia **Endereços IP**, especifique as seguintes configurações:

    |Configuração|Valor|
    |---|---|
    |Espaço de endereços IP|**10.30.0.0/16**|

1. Selecione o ícone de edição (lápis) ao lado da entrada de sub-rede **padrão**, no painel **Editar**, especifique as seguintes configurações (deixe as outras com seus valores existentes) e selecione **Salvar**:

    |Configuração|Valor|
    |---|---|
    |Nome|**hp1-Subnet**|
    |Endereço inicial|**10.30.1.0**|
    |Habilitar sub-rede privada (sem acesso de saída padrão)|Desabilitado|

1. De volta à aba **Endereços IP**, selecione **Revisar + criar** e então selecione **Criar**.

    > **Observação**: Aguarde o processo de provisionamento ser concluído. Essa etapa geralmente leva menos de um minuto.

1. No computador do laboratório, no navegador da Web que exibe o portal do Azure, pesquise e selecione **Área de Trabalho Virtual do Azure**. Na página **Área de Trabalho Virtual do Azure**, na seção **Gerenciar** do menu de navegação vertical, selecione **Pools de hosts** e, na página **Área de Trabalho Virtual do Azure \| Pools de hosts**, selecione **+ Criar**. 
1. Na guia **Básico** da página **Criar um pool de hosts**, especifique as seguintes configurações e selecione **Avançar: Hosts de sessão >** (deixe as outras configurações com seus valores padrão):

    |Configuração|Valor|
    |---|---|
    |Assinatura|O nome da assinatura do Azure que você está usando neste laboratório|
    |Grupo de recursos|**az140-15d-RG**|
    |Nome do pool de host|**az140-15-hp1**|
    |Localidade|O nome da região do Azure onde você deseja implantar seu ambiente de Área de Trabalho Virtual do Azure|
    |Ambiente de validação|**Não**|
    |Tipo de grupo de aplicativos preferencial|**Desktop**|
    |Tipo de pool de host|**Em pool**|
    |Criar configuração do host de sessão|**Não**|
    |Algoritmo de balanceamento de carga|**Amplitude**|

    > **Observação**: ao usar o algoritmo de balanceamento de carga em largura, o parâmetro limite máximo de sessão é opcional.

1. Na guia **Hosts de sessão** da página **Criar um pool de hosts**, especifique as seguintes configurações (deixe as outras configurações com seus valores padrão):

    > **Observação**: ao definir o valor do **Prefixo do nome**, alterne para a guia Recursos no lado direito da janela da sessão de laboratório e identifique a sequência de caracteres entre *Usuário1-* e o caractere *@*. Use esta cadeia de caracteres para substituir o espaço reservado *random*.

    |Configuração|Valor|
    |---|---|
    |Adicionar máquinas virtuais|**Sim**|
    |Grupo de recursos|**O padrão é o mesmo que o pool de hosts**|
    |Prefixo do nome|**sh0**_random_|
    |Tipo de máquina virtual|**Máquina Virtual do Azure**|
    |Localização da máquina virtual|O nome da região do Azure onde você deseja implantar seu ambiente de Área de Trabalho Virtual do Azure|
    |Opções de disponibilidade|**Nenhuma redundância de infraestrutura necessária**|
    |Tipo de segurança|**Máquinas virtuais de início confiável**|

1. Na guia **Máquinas virtuais** da página **Criar um pool de hosts**, abaixo da lista suspensa **Imagem**, selecione **Ver todas as imagens**.
1. Na página **Selecionar uma imagem**, selecione **Imagens compartilhadas** e, na lista de imagens, selecione **az14015imagedefinition**. 
1. De volta à guia **Máquinas virtuais** da página **Criar um pool de hosts**, especifique as seguintes configurações e selecione **Avançar: Workspace >** (deixe as outras configurações com seus valores padrão):

    |Configuração|Valor|
    |---|---|
    |Tamanho da máquina virtual|**Standard DC2s_v3**|
    |Número de VMs|**1**|
    |Tipo de disco de SO|**SSD Standard**|
    |Tamanho do disco de SO|**Tamanho padrão**|
    |Diagnóstico de Inicialização|**Habilitar com a conta de armazenamento gerenciada (recomendado)**|
    |Rede virtual|az140-vnet15d|
    |Sub-rede|**hp1-Subnet**|
    |Grupo de segurança de rede|**Basic**|
    |Portas de entrada públicas|**Não**|
    |Selecione o diretório ao qual deseja ingressar|**Microsoft Entra ID**|
    |Registrar a VM com o Intune|**Não**|
    |Nome de usuário|**Aluno**|
    |Senha|Qualquer sequência de caracteres suficientemente complexa que será usada como senha para a conta de administrador interna|
    |Confirmar senha|A mesma sequência de caracteres que você especificou anteriormente|

    > **Observação**: a senha deve ter pelo menos 12 caracteres e consistir em uma combinação de letras minúsculas, letras maiúsculas, dígitos e caracteres especiais. Para obter detalhes, consulte as informações sobre [os requisitos de senha ao criar uma VM do Azure](https://learn.microsoft.com/en-us/azure/virtual-machines/windows/faq#what-are-the-password-requirements-when-creating-a-vm-).

1. Na guia **Workspace** da página **Criar um pool de hosts**, confirme a seguinte configuração e selecione **Revisar + criar**:

    |Configuração|Valor|
    |---|---|
    |Registre o grupo de aplicativos da área de trabalho|**Não**|

1. Na guia **Revisar + criar** da página **Criar um pool de hosts**, selecione **Criar**.

    > **Observação**: aguarde até que a implantação seja concluída. Isso pode levar cerca de 10 a 15 minutos.
