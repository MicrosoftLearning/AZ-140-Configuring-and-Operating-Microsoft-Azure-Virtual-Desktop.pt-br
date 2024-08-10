---
lab:
  title: 'Laboratório: Preparar-se para a implantação da Área de Trabalho Virtual do Azure (Microsoft Entra DS)'
  module: 'Module 1: Plan an AVD Architecture'
---

# Laboratório – Preparar-se para a implantação da Área de Trabalho Virtual do Azure (Microsoft Entra DS)
# Manual de laboratório do aluno

## Dependências do laboratório

- Uma assinatura do Azure
- Uma conta Microsoft ou uma conta do Microsoft Entra com a função de administrador global no locatário do Microsoft Entra associado à assinatura do Azure e com a função de proprietário ou colaborador na assinatura

## Tempo estimado

150 minutos

>**Observação**: O provisionamento de um Microsoft Entra DS envolve cerca de 90 minutos de tempo de espera.

## Cenário do laboratório

Você precisa se preparar para a implantação da Área de Trabalho Virtual do Azure em um ambiente do Azure Active Directory Domain Services (Microsoft Entra DS)

## Objetivos
  
Depois de realizar este laboratório, você será capaz de:

- Implementar um domínio do Microsoft Entra DS
- Configurar o ambiente de domínio do Microsoft Entra DS

## Arquivos do laboratório

-  \\\\AZ-140\\AllFiles\\Labs\\01\\az140-11_azuredeploycl11a.json
-  \\\\AZ-140\\AllFiles\\Labs\\01\\az140-11_azuredeploycl11a.parameters.json

## Instruções

### Exercício 0: Aumentar o número de cotas de vCPU

As principais tarefas deste exercício são as seguintes:

1. Identificar o uso atual da vCPU
1. Solicitar aumento de cota de vCPU

#### Tarefa 1: Identificar o uso atual da vCPU

1. No seu computador de laboratório, inicie um navegador da Web, navegue até o [portal do Azure](https://portal.azure.com) e entre fornecendo credenciais de uma conta de usuário com a função Proprietário na assinatura que você usará nesse laboratório.
1. No portal do Azure, abra o painel do **Cloud Shell** selecionando o ícone da barra de ferramentas diretamente à direita da caixa de texto de pesquisa.
1. Se for solicitado que você selecione **Bash** ou **PowerShell**, selecione **PowerShell**. 

   >**Observação**: se esta é a primeira vez que você inicia o **Cloud Shell** e recebe a mensagem **Você não tem nenhum armazenamento montado**, selecione a assinatura que está usando neste laboratório e selecione **Criar armazenamento**. 

1. No portal do Azure, na sessão do PowerShell do **Cloud Shell**, execute o seguinte para registrar o provedor de recursos **Microsoft.Compute**, caso ele não esteja registrado:

   ```powershell
   Register-AzResourceProvider -ProviderNamespace 'Microsoft.Compute'
   ```

1. No portal do Azure, na sessão do PowerShell do **Cloud Shell**, execute o seguinte para verificar o status de registro do provedor de recursos **Microsoft.Compute** :

   ```powershell
   Get-AzResourceProvider -ListAvailable | Where-Object {$_.ProviderNamespace -eq 'Microsoft.Compute'}
   ```

   >**Observação**: Verifique se o status está listado como **Registrado**. Se não estiver, aguarde alguns minutos e repita essa etapa.

1. No portal do Azure, na sessão do PowerShell do **Cloud Shell**, execute o seguinte para criar uma variável do PowerShell com o nome de uma região do Azure (substitua o espaço reservado `<Azure_region>` pelo nome da região do Azure que você pretende usar para esse laboratório, como, por exemplo, `eastus`):

   ```powershell
   $location = '<Azure_region>'
   ```

   > **Observação**: Para identificar os nomes das regiões do Azure, no **Cloud Shell** no prompt do PowerShell, execute `(Get-AzLocation).Location`.
   
1. No portal do Azure, na sessão do PowerShell do **Cloud Shell**, execute o seguinte para identificar o uso atual de vCPUs e os limites correspondentes para as VMs do **Azure StandardDSv3Family** :

   ```powershell
   Get-AzVMUsage -Location $location | Where-Object {$_.Name.Value -eq 'StandardDSv3Family'}
   ```

1. Examine a saída do comando executado na etapa anterior e verifique se você tem pelo menos **20** vCPUs disponíveis na **Família DSv3 Standard de VMs do Azure** na região do Azure de destino. Se esse já for o caso, prossiga diretamente para o próximo exercício. Caso contrário, prossiga para a próxima tarefa desse exercício. 

#### Tarefa 2: Solicitar aumento de cota de vCPU

1. No portal do Azure, pesquise e selecione **Assinaturas** e, na folha **Assinaturas**, selecione a entrada que representa a assinatura do Azure que você pretende usar para esse laboratório.
1. No portal do Azure, na folha que exibe a assinatura do Azure, no menu vertical à esquerda, na seção **Configurações**, selecione **Uso + cotas**. 
1. Na folha  **Azure Pass – Patrocínio | Uso + cotas** , selecione as seguintes setas suspensas na barra de pesquisa superior:

   |**Configuração**|**Valor**|
   |---|---|
   |**Pesquisa**|**DSv3 Standard**|
   |**Todos os locais**|**Desmarque tudo**e, em seguida, verifique *sua localização*|
   |**Provedor de recursos** | **Microsoft.Compute** |
   
1. No item **VCPUs da Família DSv3 Standard** retornado, selecione o ícone de lápis, **Editar**.
1. Na folha **Detalhes da Cota**, na caixa de texto de coluna **Novo limite**, digite**30** e selecione **Salvar e continuar**.
1. Permita que a solicitação de cota seja concluída.  Após alguns instantes, a folha **Detalhes da Cota** irá especificar que a solicitação foi aprovada e a Cota aumentada. Feche a folha **Detalhes da Cota**.

    >**Observação**: Dependendo da escolha da região do Azure e da demanda atual, talvez seja necessário gerar uma solicitação de suporte. Para obter instruções sobre o processo de criação de solicitação de suporte, consulte [Criar uma solicitação de suporte do Azure](https://docs.microsoft.com/en-us/azure/azure-portal/supportability/how-to-create-azure-support-request).

### Exercício 1: Implementar um domínio do Azure Active Directory Domain Services (AD DS)

As principais tarefas desse exercício são as seguintes:

1. Criar e configurar uma conta de usuário do Microsoft Entra para administração do domínio do Microsoft Entra DS
1. Implantar uma instância do Microsoft Entra DS usando o portal do Azure
1. Definir as configurações de rede e identidade da implantação do Microsoft Entra DS

#### Tarefa 1: Criar e configurar uma conta de usuário do Microsoft Entra para administração do domínio do Microsoft Entra DS

1. No seu computador de laboratório, inicie um navegador da Web, navegue até o [portal do Azure](https://portal.azure.com) e entre fornecendo credenciais de uma conta de usuário com a função Proprietário na assinatura que você usará nesse laboratório e a função de Administrador Global no locatário do Microsoft Entra associado à assinatura do Azure.
1. No navegador da Web que exibe o portal do Azure, navegue até a folha **Visão geral** do locatário do Microsoft Entra e, no menu vertical à esquerda, na seção **Gerenciar**, clique em **Propriedades**.
1. Na folha **Propriedades** do locatário do Microsoft Entra, na parte inferior da folha, selecione o link **Gerenciar Padrões de Segurança**.
1. Na folha **Habilitar Segurança padrão**, se necessário, selecione **Não**, selecione a caixa de seleção **Minha organização está usando o Acesso Condicional** e selecione **Salvar**.
1. No portal do Azure, abra o painel do **Cloud Shell** selecionando no ícone da barra de ferramentas diretamente à direita da caixa de texto de pesquisa.
1. Se for solicitado que você selecione **Bash** ou **PowerShell**, selecione **PowerShell**. 

   >**Observação**: se esta é a primeira vez que você inicia o **Cloud Shell** e recebe a mensagem **Você não tem nenhum armazenamento montado**, selecione a assinatura que está usando neste laboratório e selecione **Criar armazenamento**. 

1. No painel do Cloud Shell, execute o seguinte para entrar no locatário do Microsoft Entra:

   ```powershell
   Connect-AzureAD
   ```

1. No painel do Cloud Shell, execute o seguinte para recuperar o nome de domínio DNS primário do locatário do Microsoft Entra associado à sua assinatura do Azure:

   ```powershell
   $aadDomainName = ((Get-AzureAdTenantDetail).VerifiedDomains)[0].Name
   $aadDomainName
   ```

1. No painel do Cloud Shell, execute o seguinte para criar usuários do Microsoft Entra que receberão privilégios elevados (substitua o espaço reservado `<password>` por uma senha aleatória e complexa):

   > **Observação**: Lembre-se da senha usada. Você precisará dela mais tarde neste e nos laboratórios subsequentes.

   ```powershell
   $passwordProfile = New-Object -TypeName Microsoft.Open.AzureAD.Model.PasswordProfile
   $passwordProfile.Password = '<password>'
   $passwordProfile.ForceChangePasswordNextLogin = $false
   New-AzureADUser -AccountEnabled $true -DisplayName 'aadadmin1' -PasswordProfile $passwordProfile -MailNickName 'aadadmin1' -UserPrincipalName "aadadmin1@$aadDomainName"
   New-AzureADUser -AccountEnabled $true -DisplayName 'wvdaadmin1' -PasswordProfile $passwordProfile -MailNickName 'wvdaadmin1' -UserPrincipalName "wvdaadmin1@$aadDomainName"
   ```

1. No painel do Cloud Shell, execute o seguinte para atribuir a função de Administrador Global ao primeiro dos usuários do Microsoft Entra recém-criados:

   ```powershell
   $aadUser = Get-AzureADUser -ObjectId "aadadmin1@$aadDomainName"
   $aadRole = Get-AzureADDirectoryRole | Where-Object {$_.displayName -eq 'Global administrator'}
   Add-AzureADDirectoryRoleMember -ObjectId $aadRole.ObjectId -RefObjectId $aadUser.ObjectId
   ```

   > **Observação**: O módulo do PowerShell do Azure AD refere-se à função de Administrador Global como Administrador da Empresa.

1. No painel do Cloud Shell, execute o seguinte para identificar o nome principal do usuário recém-criado do Microsoft Entra:

   ```powershell
   (Get-AzureADUser -Filter "MailNickName eq 'aadadmin1'").UserPrincipalName
   ```

   > **Observação**: Registre o nome da entidade de segurança do usuário. Você precisará dele posteriormente no exercício. 

1. Feche o painel do Cloud Shell.
1. No portal do Azure, pesquise e selecione **Assinaturas** e, na folha **Assinaturas**, selecione a assinatura do Azure que você está usando nesse laboratório. 
1. Na folha que exibe as propriedades de sua assinatura do Azure, selecione **Controle de acesso (IAM)**, selecione **Adicionar** e, em seguida, selecione **Adicionar atribuição de função**. 
1. Na folha **Adicionar atribuição de função**, selecione **Proprietário** e clique em **Avançar**
1. Clique no hiperlink **+Selecionar membros**.
1. Na folha **Selecionar Membros**, selecione o item **aadadmin1**, clique no botão **Selecionar** e, em seguida, clique em **Avançar**.
1. Na folha **Examinar + atribuir**, selecione o botão **Examinar + Atribuir**.

   > **Observação**: Você usará a conta **aadadmin1** para gerenciar sua assinatura do Azure e o locatário correspondente do Microsoft Entra de uma VM do Microsoft Entra DS de uma VM do Azure ingressada no Windows 10 mais tarde no laboratório. 


#### Tarefa 2: Implantar uma instância do Microsoft Entra DS usando o portal do Azure

1. No seu computador de laboratório, no portal do Azure, pesquise e selecione ** Microsoft Entra Domain Services** e, na folha **Microsoft Entra Domain Services**, selecione **+ Criar**. Isso irá abrir a folha **Criar Microsoft Entra Domain Services**.
1. Na guia **Noções básicas** da folha **Criar Microsoft Entra Domain Services **, especifique as seguintes configurações e selecione **Avançar** (deixe as outras com seus valores existentes):

   |Configuração|Valor|
   |---|---|
   |Assinatura|o nome da assinatura do Azure que você está usando neste laboratório|
   |Grupo de recursos|Selecione Criar novo **az140-11a-RG**|
   |Nome de domínio DNS|**adatum.com**|
   |Region|o nome da região em que você deseja hospedar sua implantação do AVD|
   |SKU|**Standard**|

   > **Observação**: Embora isso tecnicamente não seja necessário, em geral, você deve atribuir um nome de domínio do Microsoft Entra DS diferente de qualquer espaço de nome DNS existente ou local.

1. Na guia **Rede** da folha Criar **Microsoft Entra Domain Services**, ao lado da lista suspensa **Rede virtual**, selecione **Criar novo**.
1. Na folha **Criar rede virtual**, atribua as seguintes configurações e selecione **OK**:

   |Configuração|Valor|
   |---|---|
   |Nome|**az140-aadds-vnet11a**|
   |Intervalo de endereços|**10.10.0.0/16**|
   |Nome da sub-rede|**aadds-Subnet**|
   |Intervalo de endereços|**10.10.0.0/24**|

1. De volta à guia **Rede** da folha **Criar rede virtual**, selecione **Avançar** (deixe as outras com seus valores existentes).
1. Na guia **Administração** da folha **Criar Microsoft Entra Domain Services**, aceite as configurações padrão e selecione **Avançar**.
1. Na guia **Sincronização** da folha **Criar Microsoft Entra Domain Services**, verifique se **Todos** está selecionado e selecione **Avançar**.
1. Na guia **Configurações de Segurança** da folha **Criar Microsoft Entra Domain Services**, aceite as configurações padrão e selecione **Avançar**.
1. Na guia **Marcas** da folha **Criar Microsoft Entra Domain Services**, aceite as configurações padrão e selecione Avançar
2. Na guia **Revisar + criar** da folha **Criar Microsoft Entra Domain Services**, selecione **Criar**. 
3. Examine a notificação sobre as configurações que você não poderá alterar após a criação do domínio do Microsoft Entra DS e selecione **OK**.

   >**Observação**: As configurações que você não poderá alterar após o provisionamento de um domínio do Microsoft Entra DS incluem seu nome DNS, sua assinatura do Azure, seu grupo de recursos, a rede virtual e a sub-rede que hospeda seus controladores de domínio e o tipo de floresta.

   > **Observação**: Aguarde a conclusão da implantação antes de prosseguir para o próximo exercício. Isso pode levar cerca de 90 minutos. 

#### Tarefa 3: Definir as configurações de rede e identidade da implantação do Microsoft Entra DS

1. No seu computador de laboratório, no portal do Azure, pesquise e selecione ** Microsoft Entra Domain Services** e, na folha **Microsoft Entra Domain Services**, selecione a entrada **adatum.com** para navegar até a instância do Microsoft Entra DS recém-provisionada. 
1. Na folha **adatum.com** da instância do Microsoft Entra DS, clique no aviso que começa com **Problemas de configuração para seu domínio gerenciado**. 
1. Na folha **adatum.com | Diagnóstico de configuração**, clique em **Executar**.
1. Na seção **Validação**, expanda o painel **Registros DNS** e clique em **Corrigir**.
1. Na folha **Registros DNS**, clique em **Corrigir** novamente.
1. Navegue de volta para a folha **adatum.com** da instância do Microsoft Entra DS e, na seção **Etapas de configuração necessárias**, examine as informações sobre a sincronização de hash de senha do Microsoft Entra DS. 

   > **Observação**: Todos os usuários existentes somente na nuvem que precisam ser capazes de acessar computadores de domínio do Microsoft Entra DS e seus recursos devem alterar suas senhas ou redefini-las. Isso se aplica à conta **aadadmin1** que você criou anteriormente nesse laboratório.

1. No seu computador de laboratório, no portal do Azure, abra uma sessão do **PowerShell** no painel do **Cloud Shell**.
1. Na sessão do PowerShell no painel do Cloud Shell, execute o seguinte para identificar o atributo objectID da conta de usuário **aadadmin1** do Microsoft Entra:

   ```powershell
   Connect-AzureAD
   $objectId = (Get-AzureADUser -Filter "MailNickName eq 'aadadmin1'").ObjectId
   ```

1. Na sessão do PowerShell no painel do Cloud Shell, execute o seguinte para redefinir a senha da conta de usuário **aadadmin1**, cujo objectId você identificou na etapa anterior (substitua o espaço reservado `<password>` por uma senha aleatória e complexa):

   > **Observação**: Lembre-se da senha usada. Você precisará dela mais tarde neste e nos laboratórios subsequentes.

   ```powershell
   $password = ConvertTo-SecureString '<password>' -AsPlainText -Force
   Set-AzureADUserPassword -ObjectId $objectId -Password $password -ForceChangePasswordNextLogin $false
   ```

   > **Observação**: Em cenários reais, você normalmente definiria o valor de **-ForceChangePasswordNextLogin** como $true. Escolhemos **$false** nesse caso para simplificar as etapas do laboratório. 

1. Repita as duas etapas anteriores para redefinir a senha da conta de usuário **wvdaadmin1**.


### Exercício 2: Configurar o ambiente de domínio do Microsoft Entra DS
  
As principais tarefas desse exercício são as seguintes:

1. Implantar uma VM do Azure executando o Windows 10 usando um modelo de Início Rápido do Azure Resource Manager
1. Implantar o Azure Bastion
1. Examinar a configuração padrão do domínio do Microsoft Entra DS
1. Criar usuários e grupos do AD DS que serão sincronizados com o Microsoft Entra DS

#### Tarefa 1: Implantar uma VM do Azure executando o Windows 10 usando um modelo de Início Rápido do Azure Resource Manager

1. No seu computador de laboratório, no portal do Azure, na sessão do PowerShell no painel do Cloud Shell, execute o seguinte para adicionar uma sub-rede chamada **cl-Subnet** à rede virtual chamada **az140-aadds-vnet11a** criada na tarefa anterior:

   ```powershell
   $resourceGroupName = 'az140-11a-RG'
   $vnet = Get-AzVirtualNetwork -ResourceGroupName $resourceGroupName -Name 'az140-aadds-vnet11a'
   $subnetConfig = Add-AzVirtualNetworkSubnetConfig `
     -Name 'cl-Subnet' `
     -AddressPrefix 10.10.255.0/24 `
     -VirtualNetwork $vnet
   $vnet | Set-AzVirtualNetwork
   ```

1. No seu computador de laboratório, localize o **\\\\arquivo de parâmetros do AZ-140\\AllFiles\\Labs\\01\\az140-11_azuredeploycl11a.parameters.json** e abra o arquivo usando o Visual Studio Code.

1.  Na linha 21, localize o valor do parâmetro domainPassword. Atualize a senha existente no arquivo de parâmetro para usar a senha que você definiu anteriormente nesse laboratório para a conta de usuário **aadadmin1** e **Salve** o arquivo.

1. No portal do Azure, na barra de ferramentas do painel do Cloud Shell, selecione o ícone **Carregar/Baixar arquivos**, no menu suspenso, selecione **Carregar**e carregue os arquivos **\\\\AZ-140\\AllFiles\\Labs\\01\\az140-11_azuredeploycl11a.json** e **\\\\AZ-140\\AllFiles\\Labs\\01\\az140-11_azuredeploycl11a.parameters.json** no diretório inicial do Cloud Shell.
1. Na sessão do PowerShell no painel do Cloud Shell, execute o seguinte para implantar uma VM do Azure executando o Windows 10 que servirá como cliente da Área de Trabalho Virtual do Azure e ingresse-a no domínio do Microsoft Entra DS:

   ```powershell
   $resourceGroupName = 'az140-11a-RG'
   $location = (Get-AzResourceGroup -ResourceGroupName $resourceGroupName).Location
   New-AzResourceGroupDeployment `
     -ResourceGroupName $resourceGroupName `
     -Location $location `
     -Name az140lab0101vmDeployment `
     -TemplateFile $HOME/az140-11_azuredeploycl11a.json `
     -TemplateParameterFile $HOME/az140-11_azuredeploycl11a.parameters.json
   ```

   > **Observação**: A implantação pode levar cerca de 10 minutos. Aguarde a conclusão da implantação antes de prosseguir para a próxima tarefa. 


#### Tarefa 2: Implantar o Azure Bastion 

> **Observação**: O Azure Bastion permite a conexão com as VMs do Azure sem pontos de extremidade públicos implantados na tarefa anterior deste exercício, ao mesmo tempo em que fornece proteção contra explorações de força bruta direcionadas às credenciais de nível do sistema operacional.

> **Observação**: Verifique se o navegador tem a funcionalidade pop-up habilitada.

1. Na janela do navegador da Web que está exibindo o portal do Azure, abra outra guia e navegue até o [**portal do Azure**](https://portal.azure.com).
1. No portal do Azure, abra o painel do **Cloud Shell** selecionando no ícone da barra de ferramentas diretamente à direita da caixa de texto de pesquisa.
1. Na sessão do PowerShell no painel do Cloud Shell, execute o seguinte para adicionar uma sub-rede chamada **AzureBastionSubnet** à rede virtual chamada **az140-adds-vnet11** criada anteriormente nesse exercício:

   ```powershell
   $resourceGroupName = 'az140-11a-RG'
   $vnet = Get-AzVirtualNetwork -ResourceGroupName $resourceGroupName -Name 'az140-aadds-vnet11a'
   $subnetConfig = Add-AzVirtualNetworkSubnetConfig `
     -Name 'AzureBastionSubnet' `
     -AddressPrefix 10.10.254.0/24 `
     -VirtualNetwork $vnet
   $vnet | Set-AzVirtualNetwork
   ```

1. Feche o painel do Cloud Shell.
1. No portal do Azure, pesquise e selecione **Bastions** e, na folha **Bastions**, selecione **+ Criar**.
1. Na guia **Noções Básicas** da folha **Criar um Bastion**, especifique as seguintes configurações e selecione **Examinar + criar**:

   |Configuração|Valor|
   |---|---|
   |Assinatura|o nome da assinatura do Azure que você está usando neste laboratório|
   |Grupo de recursos|**az140-11a-RG**|
   |Nome|**az140-11a-bastion**|
   |Region|a mesma região do Azure na qual você implantou os recursos na tarefa anterior desse exercício|
   |Camada|**Basic**|
   |Rede virtual|**az140-aadds-vnet11a**|
   |Sub-rede|**AzureBastionSubnet (10.10.254.0/24)**|
   |Endereço IP público|**Criar novo**|
   |Nome do IP público|**az140-aadds-vnet11a-ip**|

1. Na guia **Revisar + criar** da folha **Criar um Bastion**, selecione **Criar**:

   > **Observação**: Aguarde a conclusão da implantação antes de prosseguir para a próxima tarefa desse exercício. A implantação pode levar cerca de cinco minutos.


#### Tarefa 3: Examinar a configuração padrão do domínio do Microsoft Entra DS

> **Observação**: Antes de entrar no computador recém-ingressado no Microsoft Entra DS, você precisa adicionar a conta de usuário com a qual pretende entrar no grupo **Administradores do AAD DC** do Microsoft Entra. Esse grupo do Microsoft Entra é criado automaticamente no locatário do Microsoft Entra associado à assinatura do Azure em que você provisionou a instância do Microsoft Entra DS.

> **Observação**: você tem a opção de preencher esse grupo com contas de usuário existentes do Microsoft Entra ao provisionar uma instância do Microsoft Entra DS.

1. No seu computador de laboratório, no portal do Azure, no painel do Cloud Shell, execute o seguinte para adicionar a conta de usuário **aadadmin1** do Microsoft Entra ao grupo **Administradores do AAD DC** do Microsoft Entra :

   ```powershell
   Connect-AzureAD
   $groupObjectId = (Get-AzureADGroup -Filter "DisplayName eq 'AAD DC Administrators'").ObjectId
   $userObjectId = (Get-AzureADUser -Filter "MailNickName eq 'aadadmin1'").ObjectId
   Add-AzureADGroupMember -ObjectId $groupObjectId -RefObjectId $userObjectId
   ```

1. Feche o painel do Cloud Shell.
1. No seu computador de laboratório, no portal do Azure, pesquise e selecione **Máquinas virtuais** e, na folha **Máquinas virtuais**, selecione a entrada **az140-cl-vm11a**. Isso irá abrir a folha **az140-cl-vm11a**.
1. Na folha **az140-cl-vm11a**, selecione **Conectar**, no menu suspenso, selecione **Bastion**, na guia **Bastion** do **az140-cl-vm11a**, forneça as seguintes credenciais e selecione **Conectar**:
1. Quando solicitado, entre como o usuário **aadadmin1** usando seu nome principal que você identificou anteriormente neste laboratório e a senha que você definiu para essa conta de usuário ao criá-la anteriormente no laboratório.
1. No Bastion para a VM do Azure **az140-cl-vm11a**, inicie o **ISE do Windows PowerShell** como Administrador e, a partir de **Administrador: Painel do ISE do Windows PowerShell**: execute o seguinte para instalar as Ferramentas de Administração de Servidor Remoto relacionadas ao Active Directory e DNS:

   ```powershell
   Add-WindowsCapability -Name Rsat.ActiveDirectory.DS-LDS.Tools~~~~0.0.1.0 -Online
   Add-WindowsCapability -Name Rsat.Dns.Tools~~~~0.0.1.0 -Online
   Add-WindowsCapability -Name Rsat.GroupPolicy.Management.Tools~~~~0.0.1.0 -Online
   Add-WindowsCapability -Name Rsat.ServerManager.Tools~~~~0.0.1.0 -Online
   ```

   > **Observação**: Aguarde até que a instalação seja concluída antes de prosseguir para a próxima etapa. Isso pode levar cerca de dois minutos.

1. No Bastion para a VM do Azure **az140-cl-vm11a**, no menu **Iniciar**, navegue até a pasta **Ferramentas Administrativas do Windows**, expanda-a e, na lista de ferramentas, inicie **Usuários e Computadores do Active Directory**. 
1. No console de **Usuários e Computadores do Active Directory**, examine a hierarquia padrão, incluindo as unidades organizacionais de **Computadores AADDC** e **Usuários do AADDC**. Observe que o primeiro inclui a conta de computador **az140-cl-vm11a** e este último inclui as contas de usuário sincronizadas do locatário do Microsoft Entra associado à assinatura do Azure que hospeda a implantação da instância do Microsoft Entra DS. A unidade organizacional **Usuários do AADDC** também inclui o grupo **Administradores do AADDC** sincronizado do mesmo locatário do Microsoft Entra, juntamente com sua associação de grupo. Essa associação não pode ser modificada diretamente no domínio do Microsoft Entra DS. Em vez disso, você precisa gerenciá-la dentro do locatário do Microsoft Entra DS. Todas as alterações são sincronizadas automaticamente com a réplica do grupo hospedado no domínio do Microsoft Entra DS. 

   **Dica:** Se a opção **Usuários e Computadores do Active Directory** não listar nenhum conteúdo relacionado ao domínio, clique com o botão direito do mouse em **Usuários e Computadores do Active Directory** e selecione** Alterar Domínio ** e escolha o domínio **Adatum **.

   > **Observação**: Atualmente, o grupo inclui apenas a conta de usuário **aadadmin1**.

1. No console **Usuários e Computadores do Active Directory**, na UO **Usuários do AADDC**, selecione a conta de usuário **aadadmin1**, exiba sua caixa de diálogo **Propriedades**, alterne para a guia **Contas** e observe que o sufixo do nome de entidade de usuário corresponde ao nome de domínio principal do Microsoft Entra DNS e não é modificável. 
1. No console **Usuários e Computadores do Active Directory**, examine o conteúdo da unidade organizacional **Controladores de Domínio** e observe que ela inclui contas de computador de dois controladores de domínio com nomes gerados aleatoriamente. 

#### Tarefa 4: Criar usuários e grupos do AD DS que serão sincronizados com o Microsoft Entra DS

1. No Bastion para a VM do Azure **az140-cl-vm11a**, inicie o Microsoft Edge, navegue até o [portal do Azure](https://portal.azure.com) e entre fornecendo o nome da entidade de usuário da conta de usuário **aadadmin1** com a senha definida anteriormente nesse laboratório como sua senha.
1. No portal do Azure, abra o **Cloud Shell**.
1. Quando solicitado a selecionar **Bash** ou **PowerShell**, selecione **PowerShell**. 

   >**Observação**: Como essa é a primeira vez que você está iniciando o **Cloud Shell** usando a conta de usuário **aadadmin1**, você precisará configurar seu diretório inicial do Cloud Shell. Quando aparecer a mensagem **Você não tem armazenamento montado**, selecione a assinatura que você está usando nesse laboratório e selecione **Criar armazenamento**. 

1. Na sessão do PowerShell no painel do Cloud Shell, execute o seguinte para entrar para autenticar no locatário do Microsoft Entra:

   ```powershell
   Connect-AzureAD
   ```

1. Na sessão do PowerShell no painel do Cloud Shell, execute o seguinte para recuperar o nome de domínio DNS primário do locatário do Microsoft Entra associado à sua assinatura do Azure:

   ```powershell
   $aadDomainName = ((Get-AzureAdTenantDetail).VerifiedDomains)[0].Name
   ```

1. Na sessão do PowerShell no painel do Cloud Shell, execute o seguinte para criar as contas de usuário do Microsoft Entra que você usará nos próximos laboratórios (substitua o espaço reservado `<password>` por uma senha aleatória e complexa):

   > **Observação**: Lembre-se da senha usada. Você precisará dela mais tarde neste e nos laboratórios subsequentes.

   ```powershell
   $passwordProfile = New-Object -TypeName Microsoft.Open.AzureAD.Model.PasswordProfile
   $passwordProfile.Password = '<password>'
   $passwordProfile.ForceChangePasswordNextLogin = $false
   $aadUserNamePrefix = 'aaduser'
   $userCount = 1..9
   foreach ($counter in $userCount) {
     New-AzureADUser -AccountEnabled $true -DisplayName "$aadUserNamePrefix$counter" -PasswordProfile $passwordProfile -MailNickName "$aadUserNamePrefix$counter" -UserPrincipalName "$aadUserNamePrefix$counter@$aadDomainName"
   } 
   ```

1. Na sessão do PowerShell no painel do Cloud Shell, execute o seguinte para criar um grupo do Microsoft Entra chamado **az140-wvd-aadmins** e adicione a ele as contas de usuário **aadadmin1** e **wvdaadmin1**:

   ```powershell
   $az140wvdaadmins = New-AzureADGroup -Description 'az140-wvd-aadmins' -DisplayName 'az140-wvd-aadmins' -MailEnabled $false -SecurityEnabled $true -MailNickName 'az140-wvd-aadmins'
   $userObjectId = (Get-AzureADUser -Filter "MailNickName eq 'aadadmin1'").ObjectId
   Add-AzureADGroupMember -ObjectId $az140wvdaadmins.ObjectId -RefObjectId $userObjectId
   $userObjectId = (Get-AzureADUser -Filter "MailNickName eq 'wvdaadmin1'").ObjectId
   Add-AzureADGroupMember -ObjectId $az140wvdaadmins.ObjectId -RefObjectId $userObjectId
   ```

1. No painel do Cloud Shell, repita a etapa anterior para criar grupos do Microsoft Entra para usuários que você irá usar nos próximos laboratórios e adicione contas de usuário do Microsoft Entra criadas anteriormente a eles:

   >**Observação**: Observação: Devido ao tamanho limitado da Área de Transferência na máquina virtual, nem todos os cmdlets listados serão copiados corretamente. Abra o Bloco de Notas na máquina virtual e copie todos os cmdlets para ele usando Digitar Texto, o constructo Digitar Texto da Área de Transferência que faz parte do controle do Lightning Bolt. Depois de garantir que todos os cmdlets estejam no Bloco de Notas, corte-os e cole-os em blocos no Cloud Shell e execute-os.

   ```powershell
   $az140wvdausers = New-AzureADGroup -Description 'az140-wvd-ausers' -DisplayName 'az140-wvd-ausers' -MailEnabled $false -SecurityEnabled $true -MailNickName 'az140-wvd-ausers'
   $userObjectId = (Get-AzureADUser -Filter "MailNickName eq 'aaduser1'").ObjectId
   Add-AzureADGroupMember -ObjectId $az140wvdausers.ObjectId -RefObjectId $userObjectId
   $userObjectId = (Get-AzureADUser -Filter "MailNickName eq 'aaduser2'").ObjectId
   Add-AzureADGroupMember -ObjectId $az140wvdausers.ObjectId -RefObjectId $userObjectId
   $userObjectId = (Get-AzureADUser -Filter "MailNickName eq 'aaduser3'").ObjectId
   Add-AzureADGroupMember -ObjectId $az140wvdausers.ObjectId -RefObjectId $userObjectId
   $userObjectId = (Get-AzureADUser -Filter "MailNickName eq 'aaduser4'").ObjectId
   Add-AzureADGroupMember -ObjectId $az140wvdausers.ObjectId -RefObjectId $userObjectId
   $userObjectId = (Get-AzureADUser -Filter "MailNickName eq 'aaduser5'").ObjectId
   Add-AzureADGroupMember -ObjectId $az140wvdausers.ObjectId -RefObjectId $userObjectId
   $userObjectId = (Get-AzureADUser -Filter "MailNickName eq 'aaduser6'").ObjectId
   Add-AzureADGroupMember -ObjectId $az140wvdausers.ObjectId -RefObjectId $userObjectId
   $userObjectId = (Get-AzureADUser -Filter "MailNickName eq 'aaduser7'").ObjectId
   Add-AzureADGroupMember -ObjectId $az140wvdausers.ObjectId -RefObjectId $userObjectId
   $userObjectId = (Get-AzureADUser -Filter "MailNickName eq 'aaduser8'").ObjectId
   Add-AzureADGroupMember -ObjectId $az140wvdausers.ObjectId -RefObjectId $userObjectId
   $userObjectId = (Get-AzureADUser -Filter "MailNickName eq 'aaduser9'").ObjectId
   Add-AzureADGroupMember -ObjectId $az140wvdausers.ObjectId -RefObjectId $userObjectId

   $az140wvdaremoteapp = New-AzureADGroup -Description "az140-wvd-aremote-app" -DisplayName "az140-wvd-aremote-app" -MailEnabled $false -SecurityEnabled $true -MailNickName "az140-wvd-aremote-app"
   $userObjectId = (Get-AzureADUser -Filter "MailNickName eq 'aaduser1'").ObjectId
   Add-AzureADGroupMember -ObjectId $az140wvdaremoteapp.ObjectId -RefObjectId $userObjectId
   $userObjectId = (Get-AzureADUser -Filter "MailNickName eq 'aaduser5'").ObjectId
   Add-AzureADGroupMember -ObjectId $az140wvdaremoteapp.ObjectId -RefObjectId $userObjectId
   $userObjectId = (Get-AzureADUser -Filter "MailNickName eq 'aaduser6'").ObjectId
   Add-AzureADGroupMember -ObjectId $az140wvdaremoteapp.ObjectId -RefObjectId $userObjectId

   $az140wvdapooled = New-AzureADGroup -Description "az140-wvd-apooled" -DisplayName "az140-wvd-apooled" -MailEnabled $false -SecurityEnabled $true -MailNickName "az140-wvd-apooled"
   $userObjectId = (Get-AzureADUser -Filter "MailNickName eq 'aaduser1'").ObjectId
   Add-AzureADGroupMember -ObjectId $az140wvdapooled.ObjectId -RefObjectId $userObjectId
   $userObjectId = (Get-AzureADUser -Filter "MailNickName eq 'aaduser2'").ObjectId
   Add-AzureADGroupMember -ObjectId $az140wvdapooled.ObjectId -RefObjectId $userObjectId
   $userObjectId = (Get-AzureADUser -Filter "MailNickName eq 'aaduser3'").ObjectId
   Add-AzureADGroupMember -ObjectId $az140wvdapooled.ObjectId -RefObjectId $userObjectId
   $userObjectId = (Get-AzureADUser -Filter "MailNickName eq 'aaduser4'").ObjectId
   Add-AzureADGroupMember -ObjectId $az140wvdapooled.ObjectId -RefObjectId $userObjectId

   $az140wvdapersonal = New-AzureADGroup -Description "az140-wvd-apersonal" -DisplayName "az140-wvd-apersonal" -MailEnabled $false -SecurityEnabled $true -MailNickName "az140-wvd-apersonal"
   $userObjectId = (Get-AzureADUser -Filter "MailNickName eq 'aaduser7'").ObjectId
   Add-AzureADGroupMember -ObjectId $az140wvdapersonal.ObjectId -RefObjectId $userObjectId
   $userObjectId = (Get-AzureADUser -Filter "MailNickName eq 'aaduser8'").ObjectId
   Add-AzureADGroupMember -ObjectId $az140wvdapersonal.ObjectId -RefObjectId $userObjectId
   $userObjectId = (Get-AzureADUser -Filter "MailNickName eq 'aaduser9'").ObjectId
   Add-AzureADGroupMember -ObjectId $az140wvdapersonal.ObjectId -RefObjectId $userObjectId
   ```

1. Feche o painel do Cloud Shell.
1. No Bastion para a VM do Azure **az140-cl-vm11a**, na janela do Microsoft Edge exibindo o portal do Azure, pesquise e selecione a folha do **Azure Active Directory**, na folha do locatário do Microsoft Entra, na barra de menus vertical à esquerda, na seção **Gerenciar**, selecione **Usuários** e, na folha **Usuários\| Todos os usuários**, verifique se novas contas de usuário foram criadas.
1. Navegue de volta até a folha do locatário do Microsoft Entra, na barra de menus vertical no lado esquerdo, na seção **Gerenciar**, selecione **Grupos** e, na folha **Grupos \|Todos os grupos **, verifique se novas contas de grupo foram criadas.
1. No Bastion para a VM do Azure **Az140-cl-vm11a**, alterne para o console **Usuários e Computadores do Active Directory**, no console **Usuários e Computadores do Active Directory**, navegue até a UO **Usuários do AADDC** e verifique se ela contém as mesmas contas de usuário e de grupo.

   >**Observação**: Talvez seja necessário atualizar o modo de exibição do console.
