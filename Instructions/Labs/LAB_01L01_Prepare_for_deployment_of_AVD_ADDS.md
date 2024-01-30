---
lab:
  title: 'Laboratório: Laboratório: preparar para a implantação da Área de Trabalho Virtual do Azure (AD DS)'
  module: 'Module 1: Plan a AVD Architecture'
---

# Laboratório: preparar para a implantação da Área de Trabalho Virtual do Azure (AD DS)
# Manual de laboratório do aluno

## Dependências de laboratório

- O nome da assinatura do Azure que você usará neste laboratório.
- Uma conta Microsoft ou uma conta do Microsoft Entra com a função de Proprietário ou Colaborador na assinatura do Azure que você usará neste laboratório e com a função de Administrador Global no locatário do Microsoft Entra associado a essa assinatura do Azure.

## Tempo estimado

60 minutos

## Cenário do laboratório

Você precisa se preparar para a implantação de um ambiente do Active Directory Domain Services (AD DS)

## Objetivos
  
Depois de realizar este laboratório, você será capaz de:

- Implantar uma floresta de domínio único do Active Directory Domain Services (AD DS) usando VMs do Azure
- Integrar uma floresta do AD DS a um locatário do Microsoft Entra

## Arquivos de laboratório

-  \\\\AZ-140\\AllFiles\\Labs\\01\\az140-11_azuredeploydc11.parameters.json
-  \\\\AZ-140\\AllFiles\\Labs\\01\\az140-11_azuredeploycl11.json
-  \\\\AZ-140\\AllFiles\\Labs\\01\\az140-11_azuredeploycl11.parameters.json

## Instruções

### Exercício 0: Aumentar o número de cotas de vCPU

As principais tarefas deste exercício são as seguintes:

1. Identificar o uso atual da vCPU
1. Solicitar aumento de cota de vCPU

#### Tarefa 1: Identificar o uso atual da vCPU

1. No seu computador de laboratório, inicie um navegador da Web, navegue até o [portal do Azure](https://portal.azure.com) e entre fornecendo credenciais de uma conta de usuário com a função Proprietário na assinatura que você usará neste laboratório.
1. No portal do Azure, abra o painel do **Cloud Shell** selecionando o ícone da barra de ferramentas diretamente à direita da caixa de texto de pesquisa.
1. Se for solicitado que você selecione **Bash** ou **PowerShell**, selecione **PowerShell**. 

   >**Observação**: se esta é a primeira vez que você inicia o **Cloud Shell** e recebe a mensagem **Você não tem nenhum armazenamento montado**, selecione a assinatura que está usando neste laboratório e selecione **Criar armazenamento**. 

1. No portal do Azure, na sessão do PowerShell do **Cloud Shell**, execute o seguinte para registrar os provedores de recursos **Microsoft.Compute** e **Microsoft.Network**, caso eles não estejam registrados:

   ```powershell
   Register-AzResourceProvider -ProviderNamespace 'Microsoft.Compute'
   Register-AzResourceProvider -ProviderNamespace 'Microsoft.Network'
   ```

1. No portal do Azure, na sessão do PowerShell do **Cloud Shell**, execute o seguinte para verificar o status de registro do provedor de recursos **Microsoft.Compute**:

   ```powershell
   Get-AzResourceProvider -ListAvailable | Where-Object {$_.ProviderNamespace -eq 'Microsoft.Compute'}
   ```

   >**Observação**: Verifique se o status está listado como **Registrado**. Se não estiver, aguarde alguns minutos e repita essa etapa.

1. No portal do Azure, na sessão do PowerShell do **Cloud Shell**, execute o seguinte para definir o local para os próximos comandos (substitua o espaço reservado `<Azure_region>` pelo nome da região do Azure que você pretende usar para este laboratório, como, por exemplo, `eastus`):

   ```powershell
   $location = '<Azure_region>'
   ```

1. No portal do Azure, na sessão do PowerShell do **Cloud Shell**, execute o seguinte para identificar o uso atual de vCPUs e os limites correspondentes para as VMs **StandardDSv3Family** e **StandardBSFamily** do Azure: 

   ```powershell
   Get-AzVMUsage -Location $location | Where-Object {$_.Name.Value -eq 'StandardDSv3Family'}
   ```

   > **Observação**: Para identificar os nomes das regiões do Azure, no **Cloud Shell** no prompt do PowerShell, execute `(Get-AzLocation).Location`.
   
1. Examine a saída do comando executado na etapa anterior e verifique se você tem pelo menos **30** vCPUs disponíveis nas **vCPUs da Família DSv3 Standard** das VMs do Azure na região do Azure de destino. Se esse já for o caso, prossiga diretamente para o próximo exercício. Caso contrário, prossiga para a próxima tarefa deste exercício. 

#### Tarefa 2: Solicitar aumento de cota de vCPU

1. No portal do Azure, pesquise e selecione **Assinaturas** e, na folha **Assinaturas**, selecione a entrada que representa a assinatura do Azure que você pretende usar para este laboratório.
1. Na folha que exibe a assinatura do Azure, no menu vertical à esquerda, na seção **Configurações**, selecione **Uso + cotas**. 

   **Observação:** Talvez você não precise abrir um tíquete de suporte para aumentar as cotas.

   **Observação:** A solicitação de aumento de cota requer a entrada com a autenticação multifator (MFA). Se você precisar configurar sua conta com a MFA, consulte [Planejar uma implantação de Autenticação Multifator do Azure Active Directory](https://learn.microsoft.com/en-us/azure/active-directory/authentication/howto-mfa-getstarted). 
   
1. Na folha **Azure Pass – Patrocínio | Folha Uso + cotas**, selecione **Região**, na lista suspensa, marque a caixa de seleção ao lado do nome da região do Azure que você pretende usar para este laboratório, selecione **Aplicar**, verifique se a entrada **Computação** aparece na lista suspensa à esquerda da entrada **Região** e, na caixa de pesquisa, digite **Standard DSv3**. 
1. Na lista de resultados, marque a caixa de seleção ao lado do item **VCPUs da Família DSv3 Standard**, selecione a entrada **Solicitar aumento de cota** na barra de ferramentas e, na lista suspensa, selecione **Inserir um novo limite**.
1. No painel **Solicitar aumento de cota**, na caixa de texto de coluna**Novo limite**, digite **30**e selecione **Enviar**.
1. Se solicitado, no painel **Solicitar aumento de cota**, selecione **Autenticar com a Autenticação multifator** e siga os prompts para autenticação.
1. Permitir que a solicitação de cota seja concluída.  Após alguns instantes, a folha **Detalhes da Cota** especificará que a solicitação foi aprovada e Cota aumentada. Feche a folha **Detalhes da Cota**.

   >**Observação**: Dependendo da escolha da região do Azure e da demanda atual, talvez seja necessário abrir uma solicitação de suporte. Para obter instruções sobre o processo de criação de solicitação de suporte, consulte [Criar uma solicitação de suporte do Azure](https://docs.microsoft.com/en-us/azure/azure-portal/supportability/how-to-create-azure-support-request).

### Exercício 1: Implantar um domínio do Active Directory Domain Services (AD DS)

As principais tarefas deste exercício são as seguintes:

1. Preparar para uma implantação de VM do Azure
1. Implantar uma VM do Azure executando um controlador de domínio do AD DS usando um modelo de Início Rápido do Azure Resource Manager
1. Implantar uma VM do Azure executando o Windows 10 usando um modelo de Início Rápido do Azure Resource Manager
1. Implantar o Azure Bastion

#### Tarefa 1: Preparar para uma implantação de VM do Azure

1. No seu computador de laboratório, inicie um navegador da Web, navegue até o [portal do Azure](https://portal.azure.com) e entre fornecendo credenciais de uma conta de usuário com a função Proprietário na assinatura que você usará neste laboratório.
1. No navegador da Web que exibe o portal do Azure, navegue até a folha **Visão geral** do locatário do Microsoft Entra e, no menu vertical à esquerda, na seção **Gerenciar**, clique em **Propriedades**.
1. Na folha **Propriedades** do locatário do Microsoft Entra, na parte inferior da folha, selecione o link **Gerenciar Padrões de Segurança**.
1. Na folha **Habilitar Segurança padrão**, se necessário, selecione **Não**, selecione a caixa de seleção **Minha organização está usando o Acesso Condicional** e selecione **Salvar**.
1. No portal do Azure, abra o painel do **Cloud Shell** selecionando no ícone da barra de ferramentas diretamente à direita da caixa de texto de pesquisa.
1. Se for solicitado que você selecione **Bash** ou **PowerShell**, selecione **PowerShell**. 

   >**Observação**: se esta é a primeira vez que você inicia o **Cloud Shell** e recebe a mensagem **Você não tem nenhum armazenamento montado**, selecione a assinatura que está usando neste laboratório e selecione **Criar armazenamento**. 


#### Tarefa 2: Implantar uma VM do Azure executando um controlador de domínio do AD DS usando um modelo de Início Rápido do Azure Resource Manager

1. No computador do laboratório, no navegador da Web que exibe o portal do Azure, na sessão do PowerShell no painel do Cloud Shell, execute o seguinte para criar um grupo de recursos (substitua o espaço reservado `<Azure_region>` pelo nome da região do Azure que você pretende usar para este laboratório, como, por exemplo, `eastus`):

   ```powershell
   $location = '<Azure_region>'
   $resourceGroupName = 'az140-11-RG'
   New-AzResourceGroup -Location $location -Name $resourceGroupName
   ```

1. No portal do Azure, feche o painel do **Cloud Shell**.
1. No computador do laboratório, na mesma janela do navegador da Web, abra outra guia do navegador da Web e navegue por uma versão personalizada do modelo de Início Rápido chamado [Criar uma nova VM do Windows e crie uma nova Floresta, Domínio e DC do AD](https://github.com/az140mp/azure-quickstart-templates/tree/master/application-workloads/active-directory/active-directory-new-domain). 
1. Na página **Criar uma nova VM do Windows e criar uma nova Floresta, Domínio e DC do AD**, selecione **Implantar no Azure**. Isso redirecionará automaticamente o navegador para o painel **Criar uma VM do Azure com uma nova Floresta do AD** no portal do Azure.
1. Na folha **Criar uma VM do Azure com uma nova folha Floresta do AD**, selecione **Editar parâmetros**.
1. Na folha **Editar parâmetros**, selecione **Carregar arquivo**, na caixa de diálogo **Abrir**, selecione **\\\\AZ-140\\AllFiles\\Labs\\01\\az140-11_azuredeploydc11.parameters.json**, selecione **Abrir** e, em seguida, selecione **Salvar**. 
1. No painel **Criar uma VM do Azure com uma nova Floresta do AD**, especifique as seguintes configurações (deixe as outras com seus valores existentes):

   |Configuração|Valor|
   |---|---|
   |Assinatura|o nome da assinatura do Azure que você está usando neste laboratório|
   |Grupo de recursos|**az140-11-RG**|
   |Nome de domínio|**adatum.com**|

1. Na folha **Criar uma VM do Azure com uma nova Floresta do AD**, selecione **Examinar + criar** e selecionar **Criar**.

   > **Observação**: Aguarde a conclusão da implantação antes de prosseguir para o próximo exercício. Isso pode levar cerca de 15 minutos. 

#### Tarefa 3: Implantar uma VM do Azure executando o Windows 10 usando um modelo de Início Rápido do Azure Resource Manager

1. No computador do laboratório, no navegador da Web que exibe o portal do Azure, abra uma sessão do PowerShell no painel do Cloud Shell e execute o seguinte para adicionar uma sub-rede chamada **cl-Subnet** à rede virtual chamada **az140-adds-vnet11** criada na tarefa anterior:

   ```powershell
   $resourceGroupName = 'az140-11-RG'
   $vnet = Get-AzVirtualNetwork -ResourceGroupName $resourceGroupName -Name 'az140-adds-vnet11'
   $subnetConfig = Add-AzVirtualNetworkSubnetConfig `
     -Name 'cl-Subnet' `
     -AddressPrefix 10.0.255.0/24 `
     -VirtualNetwork $vnet
   $vnet | Set-AzVirtualNetwork
   ```

1. No portal do Azure, na barra de ferramentas do painel do Cloud Shell, selecione o ícone **Carregar/Baixar arquivos**, no menu suspenso, selecione **Carregar**e carregue os arquivos **\\\\AZ-140\\AllFiles\\Labs\\01\\az140-11_azuredeploycl11.json** e **\\\\AZ-140\\AllFiles\\Labs\\01\\az140-11_azuredeploycl11.parameters.json** no diretório inicial do Cloud Shell.
1. Na sessão do PowerShell no painel do Cloud Shell, execute o seguinte para implantar uma VM do Azure executando o Windows 10 que servirá como um cliente na sub-rede recém-criada:

   ```powershell
   $location = (Get-AzResourceGroup -ResourceGroupName $resourceGroupName).Location
   New-AzResourceGroupDeployment `
     -ResourceGroupName $resourceGroupName `
     -Location $location `
     -Name az140lab0101vmDeployment `
     -TemplateFile $HOME/az140-11_azuredeploycl11.json `
     -TemplateParameterFile $HOME/az140-11_azuredeploycl11.parameters.json
   ```

   > **Observação:** prossiga para a próxima etapa sem aguardar a conclusão da implantação. A implantação pode levar cerca de 10 minutos.

#### Tarefa 4: Implantar o Azure Bastion 

> **Observação**: O Azure Bastion permite a conexão com as VMs do Azure sem pontos de extremidade públicos implantados na tarefa anterior deste exercício, ao mesmo tempo em que fornece proteção contra explorações de força bruta direcionadas às credenciais de nível do sistema operacional.

> **Observação**: Verifique se o navegador tem a funcionalidade pop-up habilitada.

1. Na janela do navegador da Web que está exibindo o portal do Azure, abra outra guia e navegue até o [portal do Azure](https://portal.azure.com).
1. No portal do Azure, abra o painel do **Cloud Shell** selecionando no ícone da barra de ferramentas diretamente à direita da caixa de texto de pesquisa.
1. Na sessão do PowerShell no painel do Cloud Shell, execute o seguinte para adicionar uma sub-rede chamada **AzureBastionSubnet** à rede virtual chamada **az140-adds-vnet11** criada anteriormente nesse exercício:

   ```powershell
   $resourceGroupName = 'az140-11-RG'
   $vnet = Get-AzVirtualNetwork -ResourceGroupName $resourceGroupName -Name 'az140-adds-vnet11'
   $subnetConfig = Add-AzVirtualNetworkSubnetConfig `
     -Name 'AzureBastionSubnet' `
     -AddressPrefix 10.0.254.0/24 `
     -VirtualNetwork $vnet
   $vnet | Set-AzVirtualNetwork
   ```

1. Feche o painel do Cloud Shell.
1. No portal do Azure, pesquise e selecione **Bastions** e, na folha **Bastions**, selecione **+ Criar**.
1. Na guia **Noções Básica** da folha **Criar um Bastion**, especifique as seguintes configurações e selecione **Examinar + criar**:

   |Configuração|Valor|
   |---|---|
   |Assinatura|o nome da assinatura do Azure que você está usando neste laboratório|
   |Grupo de recursos|**az140-11-RG**|
   |Nome|**az140-11-bastion**|
   |Region|a mesma região do Azure na qual você implantou os recursos nas tarefas anteriores desse exercício|
   |Camada|**Basic**|
   |Rede virtual|**az140-adds-vnet11**|
   |Sub-rede|**AzureBastionSubnet (10.0.254.0/24)**|
   |Endereço IP público|**Criar novo**|
   |Nome do IP público|**az140-adds-vnet11-ip**|

1. Na guia **Revisar + criar** da folha **Criar um Bastion**, selecione **Criar**:

   > **Observação**: Aguarde a conclusão da implantação antes de prosseguir para o próximo exercício. A implantação pode levar cerca de cinco minutos.

### Exercício 2: Integrar uma floresta do AD DS a um locatário do Microsoft Entra
  
As principais tarefas deste exercício são as seguintes:

1. Criar usuários e grupos do AD DS que serão sincronizados com o Microsoft Entra
1. Configurar o sufixo UPN do AD DS
1. Criar um usuário do Microsoft Entra que será usado para configurar a sincronização com o Microsoft Entra
1. Instalar o Microsoft Entra Connect
1. Configurar a o ingresso híbrido do Microsoft Entra

#### Tarefa 1: Criar usuários e grupos do AD DS que serão sincronizados com o Microsoft Entra

1. No computador do laboratório, no navegador da Web que exibe o portal do Azure, pesquise e selecione **Máquinas Virtuais** e, na folha **Máquinas Virtuais**, selecione**az140-dc-vm11**.
1. Na folha **az140-dc-vm11**, selecione **Conectar**, no menu suspenso, selecione **Bastion**, na guia **Bastion** da folha **az140-dc-vm11\| Conectar**, selecione **Usar Bastion**.
1. Quando solicitado, forneça as seguintes credenciais e selecione **Conectar**:

   |Configuração|Valor|
   |---|---|
   |Nome do usuário|**Aluno**|
   |Senha|**Pa55w.rd1234**|

1. Na sessão da Área de Trabalho Remota para **az140-dc-vm11**, inicie o **ISE do Windows PowerShell** como administrador.
1. A partir do **Administrador: Painel de script do ISE do Windows PowerShell**: execute o seguinte para desabilitar a Segurança Aprimorada do Internet Explorer para Administradores:

   ```powershell
   $adminRegEntry = 'HKLM:\SOFTWARE\Microsoft\Active Setup\Installed Components\{A509B1A7-37EF-4b3f-8CFC-4F3A74704073}'
   Set-ItemProperty -Path $AdminRegEntry -Name 'IsInstalled' -Value 0
   Stop-Process -Name Explorer
   ```

1. A partir do **Administrador: Console do ISE do Windows PowerShell**: execute o seguinte para criar uma unidade organizacional do AD DS que conterá objetos incluídos no escopo da sincronização com o locatário do Microsoft Entra usado nesse laboratório:

   ```powershell
   New-ADOrganizationalUnit 'ToSync' -path 'DC=adatum,DC=com' -ProtectedFromAccidentalDeletion $false
   ```

1. A partir do **Administrador: Console ISE do Windows PowerShell**: execute o seguinte para criar uma unidade organizacional do AD DS que conterá objetos de computador de computadores cliente ingressados no domínio do Windows 10:

   ```powershell
   New-ADOrganizationalUnit 'WVDClients' -path 'DC=adatum,DC=com' -ProtectedFromAccidentalDeletion $false
   ```

1. A partir do **Administrador: Painel de script do ISE do Windows PowerShell**: execute o seguinte para criar contas de usuário do AD DS que serão sincronizadas com o locatário do Microsoft Entra usado nesse laboratório (substitua ambos os espaços reservados `<password>` por senhas aleatórias e complexas):

   > **Observação**: Certifique-se de registrar as senhas usadas. Você precisará delas mais tarde neste e nos laboratórios subsequentes.

   ```powershell
   $ouName = 'ToSync'
   $ouPath = "OU=$ouName,DC=adatum,DC=com"
   $adUserNamePrefix = 'aduser'
   $adUPNSuffix = 'adatum.com'
   $userCount = 1..9
   foreach ($counter in $userCount) {
     New-AdUser -Name $adUserNamePrefix$counter -Path $ouPath -Enabled $True `
       -ChangePasswordAtLogon $false -userPrincipalName $adUserNamePrefix$counter@$adUPNSuffix `
       -AccountPassword (ConvertTo-SecureString '<password>' -AsPlainText -Force) -passThru
   } 

   $adUserNamePrefix = 'wvdadmin1'
   $adUPNSuffix = 'adatum.com'
   New-AdUser -Name $adUserNamePrefix -Path $ouPath -Enabled $True `
       -ChangePasswordAtLogon $false -userPrincipalName $adUserNamePrefix@$adUPNSuffix `
       -AccountPassword (ConvertTo-SecureString '<password>' -AsPlainText -Force) -passThru

   Get-ADGroup -Identity 'Domain Admins' | Add-AdGroupMember -Members 'wvdadmin1'
   ```

   > **Observação**: O script cria nove contas de usuário sem privilégios chamadas **aduser1** - **aduser9** e uma conta com privilégios que é membro do grupo **Administradores de Domínio do \\ADATUM** chamado **wvdadmin1**.

1. A partir do **Administrador: Painel de script ISE do Windows PowerShell**: execute o seguinte para criar objetos de grupo do AD DS que serão sincronizados com o locatário do Microsoft Entra usado neste laboratório:

   ```powershell
   New-ADGroup -Name 'az140-wvd-pooled' -GroupScope 'Global' -GroupCategory Security -Path $ouPath
   New-ADGroup -Name 'az140-wvd-remote-app' -GroupScope 'Global' -GroupCategory Security -Path $ouPath
   New-ADGroup -Name 'az140-wvd-personal' -GroupScope 'Global' -GroupCategory Security -Path $ouPath
   New-ADGroup -Name 'az140-wvd-users' -GroupScope 'Global' -GroupCategory Security -Path $ouPath
   New-ADGroup -Name 'az140-wvd-admins' -GroupScope 'Global' -GroupCategory Security -Path $ouPath
   ```

1. A partir do **Administrador: Console do ISE do Windows PowerShell**: execute o seguinte para adicionar membros aos grupos criados na etapa anterior:

   ```powershell
   Get-ADGroup -Identity 'az140-wvd-pooled' | Add-AdGroupMember -Members 'aduser1','aduser2','aduser3','aduser4'
   Get-ADGroup -Identity 'az140-wvd-remote-app' | Add-AdGroupMember -Members 'aduser1','aduser5','aduser6'
   Get-ADGroup -Identity 'az140-wvd-personal' | Add-AdGroupMember -Members 'aduser7','aduser8','aduser9'
   Get-ADGroup -Identity 'az140-wvd-users' | Add-AdGroupMember -Members 'aduser1','aduser2','aduser3','aduser4','aduser5','aduser6','aduser7','aduser8','aduser9'
   Get-ADGroup -Identity 'az140-wvd-admins' | Add-AdGroupMember -Members 'wvdadmin1'
   ```

#### Tarefa 2: Configurar o sufixo UPN do AD DS

1. Na sessão da Área de Trabalho Remota para **az140-dc-vm11**, do **Administrador: Painel de script do ISE do Windows PowerShell**: execute o seguinte para instalar a versão mais recente do módulo PowerShellGet (selecione **Sim** quando solicitado a confirmar):

   ```powershell
   [Net.ServicePointManager]::SecurityProtocol = [Net.SecurityProtocolType]::Tls12
   Install-Module -Name PowerShellGet -Force -SkipPublisherCheck
   ```

1. A partir do **Administrador: Console ISE do Windows PowerShell**: execute o seguinte para instalar a versão mais recente do módulo do Az PowerShell (selecione **Sim para Todos** quando solicitado a confirmação):

   ```powershell
   Install-Module -Name Az -AllowClobber -SkipPublisherCheck
   ```

1. A partir do **Administrador: Console do ISE do Windows PowerShell**: execute o seguinte para entrar em sua assinatura do Azure:

   ```powershell
   Connect-AzAccount
   ```

1. Quando solicitado, forneça as credenciais da conta de usuário com a função Proprietário na assinatura que você está usando neste laboratório.
1. A partir do **Administrador: Console do ISE do Windows PowerShell**: execute o seguinte para recuperar a propriedade de ID do locatário do Microsoft Entra associado à sua assinatura do Azure:

   ```powershell
   $tenantId = (Get-AzContext).Tenant.Id
   ```

1. A partir do **Administrador: Console do ISE do Windows PowerShell**: execute o seguinte para instalar e importar a versão mais recente do módulo do PowerShell do Azure AD:

   ```powershell
   Install-Module -Name AzureAD -Force
   Import-Module -Name AzureAD
   ```

1. A partir do **Administrador: Console do ISE do Windows PowerShell**: execute o seguinte para autenticar em seu locatário do Microsoft Entra:

   ```powershell
   Connect-AzureAD -TenantId $tenantId
   ```

1. Quando solicitado, entre com as mesmas credenciais usadas anteriormente nesta tarefa (a conta de usuário com a função Proprietário na assinatura que você está usando neste laboratório). 
1. A partir do **Administrador: Console do ISE do Windows PowerShell**: execute o seguinte para recuperar o nome de domínio DNS primário do locatário do Microsoft Entra associado à sua assinatura do Azure:

   ```powershell
   $aadDomainName = ((Get-AzureAdTenantDetail).VerifiedDomains)[0].Name
   ```

1. A partir do **Administrador: Console do ISE do Windows PowerShell**: execute o seguinte para adicionar o nome de domínio DNS primário do locatário do Microsoft Entra associado à sua assinatura do Azure à lista de sufixos UPN da floresta do AD DS:

   ```powershell
   Get-ADForest|Set-ADForest -UPNSuffixes @{add="$aadDomainName"}
   ```

1. A partir do **Administrador: Painel de script do ISE do Windows PowerShell**: execute o seguinte para atribuir o nome de domínio DNS primário do locatário do Microsoft Entra associado à sua assinatura do Azure como o sufixo UPN de todos os usuários no domínio do AD DS:

   ```powershell
   $domainUsers = Get-ADUser -Filter {UserPrincipalName -like '*adatum.com'} -Properties userPrincipalName -ResultSetSize $null
   $domainUsers | foreach {$newUpn = $_.UserPrincipalName.Replace('adatum.com',$aadDomainName); $_ | Set-ADUser -UserPrincipalName $newUpn}
   ```

1. A partir do **Administrador: Console do ISE do Windows PowerShell**: execute o seguinte para atribuir o sufixo UPN **adatum.com** ao usuário do domínio **Estudante**:

   ```powershell
   $domainAdminUser = Get-ADUser -Filter {sAMAccountName -eq 'Student'} -Properties userPrincipalName
   $domainAdminUser | Set-ADUser -UserPrincipalName 'student@adatum.com'
   ```

#### Tarefa 3: Criar um usuário do Microsoft Entra que será usado para configurar a sincronização de diretórios

1. Na sessão da Área de Trabalho Remota para **az140-dc-vm11**, a partir do **Administrador: Painel de script do ISE do Windows PowerShell**: execute o seguinte para criar um novo usuário do Microsoft Entra (substitua o espaço reservado `<password>` por uma senha aleatória e complexa):

   > **Observação**: Certifique-se de registrar a senha usada. Você precisará dela mais tarde neste e nos laboratórios subsequentes.

   ```powershell
   $userName = 'aadsyncuser'
   $passwordProfile = New-Object -TypeName Microsoft.Open.AzureAD.Model.PasswordProfile
   $passwordProfile.Password = '<password>'
   $passwordProfile.ForceChangePasswordNextLogin = $false
   New-AzureADUser -AccountEnabled $true -DisplayName $userName -PasswordProfile $passwordProfile -MailNickName $userName -UserPrincipalName "$userName@$aadDomainName"
   ```

1. A partir do **Administrador: Painel de script do ISE do Windows PowerShell**: execute o seguinte para atribuir a função de Administrador Global ao usuário recém-criado do Microsoft Entra: 

   ```powershell
   $aadUser = Get-AzureADUser -ObjectId "$userName@$aadDomainName"
   $aadRole = Get-AzureADDirectoryRole | Where-Object {$_.displayName -eq 'Global administrator'} 
   Add-AzureADDirectoryRoleMember -ObjectId $aadRole.ObjectId -RefObjectId $aadUser.ObjectId
   ```

1. A partir do **Administrador: Painel de script do ISE do Windows PowerShell**: execute o seguinte para identificar o nome principal do usuário recém-criado do Microsoft Entra:

   ```powershell
   (Get-AzureADUser -Filter "MailNickName eq '$userName'").UserPrincipalName
   ```

   > **Observação**: Registre o nome da entidade de segurança do usuário. Você precisará dele posteriormente nesse exercício. 


#### Tarefa 4: O que é Microsoft Entra Connect?

1. Na sessão da Área de Trabalho Remota para **az140-dc-vm11**, a partir do **Administrador: Painel de script do ISE do Windows PowerShell**: execute o seguinte para habilitar o TLS 1.2:

   ```powershell
   New-Item 'HKLM:\SOFTWARE\WOW6432Node\Microsoft\.NETFramework\v4.0.30319' -Force | Out-Null
   New-ItemProperty -path 'HKLM:\SOFTWARE\WOW6432Node\Microsoft\.NETFramework\v4.0.30319' -name 'SystemDefaultTlsVersions' -value '1' -PropertyType 'DWord' -Force | Out-Null
   New-ItemProperty -path 'HKLM:\SOFTWARE\WOW6432Node\Microsoft\.NETFramework\v4.0.30319' -name 'SchUseStrongCrypto' -value '1' -PropertyType 'DWord' -Force | Out-Null
   New-Item 'HKLM:\SOFTWARE\Microsoft\.NETFramework\v4.0.30319' -Force | Out-Null
   New-ItemProperty -path 'HKLM:\SOFTWARE\Microsoft\.NETFramework\v4.0.30319' -name 'SystemDefaultTlsVersions' -value '1' -PropertyType 'DWord' -Force | Out-Null
   New-ItemProperty -path 'HKLM:\SOFTWARE\Microsoft\.NETFramework\v4.0.30319' -name 'SchUseStrongCrypto' -value '1' -PropertyType 'DWord' -Force | Out-Null
   New-Item 'HKLM:\SYSTEM\CurrentControlSet\Control\SecurityProviders\SCHANNEL\Protocols\TLS 1.2\Server' -Force | Out-Null
   New-ItemProperty -path 'HKLM:\SYSTEM\CurrentControlSet\Control\SecurityProviders\SCHANNEL\Protocols\TLS 1.2\Server' -name 'Enabled' -value '1' -PropertyType 'DWord' -Force | Out-Null
   New-ItemProperty -path 'HKLM:\SYSTEM\CurrentControlSet\Control\SecurityProviders\SCHANNEL\Protocols\TLS 1.2\Server' -name 'DisabledByDefault' -value 0 -PropertyType 'DWord' -Force | Out-Null
   New-Item 'HKLM:\SYSTEM\CurrentControlSet\Control\SecurityProviders\SCHANNEL\Protocols\TLS 1.2\Client' -Force | Out-Null
   New-ItemProperty -path 'HKLM:\SYSTEM\CurrentControlSet\Control\SecurityProviders\SCHANNEL\Protocols\TLS 1.2\Client' -name 'Enabled' -value '1' -PropertyType 'DWord' -Force | Out-Null
   New-ItemProperty -path 'HKLM:\SYSTEM\CurrentControlSet\Control\SecurityProviders\SCHANNEL\Protocols\TLS 1.2\Client' -name 'DisabledByDefault' -value 0 -PropertyType 'DWord' -Force | Out-Null
   Write-Host 'TLS 1.2 has been enabled.'
   ```
   
1. Na sessão da Área de Trabalho Remota para **az140-dc-vm11**, inicie o Internet Explorer e navegue até a [página de download do Microsoft Edge for Business](https://www.microsoft.com/en-us/edge/business/download).
1. Na [página de download do Microsoft Edge for Business](https://www.microsoft.com/en-us/edge/business/download), baixe a versão estável mais recente do Microsoft Edge, instale-a, inicie-a e configure-a com as configurações padrão.
1. Na sessão da Área de Trabalho Remota para **az140-dc-vm11**, use o Microsoft Edge para navegar até o [portal do Azure](https://portal.azure.com). Se solicitado, entre usando as credenciais do Microsoft Entra da conta de usuário com a função Proprietário na assinatura que você está usando neste laboratório.
1. No portal do Azure, use a caixa de texto **Pesquisar recursos, serviços e documentos** na parte superior da página do portal do Azure para pesquisar e navegar até a folha do **Azure Active Directory** e, na folha do locatário do Microsoft Entra, na seção **Gerenciar** do menu do hub, selecione **Microsoft Entra Connect**.
1. Na folha do **Microsoft Entra Connect**, selecione primeiro o link **Conectar Sincronização** à esquerda e, em seguida, selecione o link **Baixar o Microsoft Entra Connect**. Isso abrirá automaticamente uma nova guia do navegador exibindo a página de download do **Microsoft Azure Active Directory Connect**.
1. Na página de download do **Microsoft Azure Active Directory Connect**, selecione **Baixar**.
1. Se solicitado a executar ou salvar o instalador **AzureADConnect.msi**, selecione **Executar**. Caso contrário, abra o arquivo após o download para iniciar o assistente do **Microsoft Azure Active Directory Connect**.
1. Na página **Boas-vindas ao Microsoft Entra Connect** do assistente do **Microsoft Azure Active Directory Connect**, marque a caixa de seleção ** Concordo com os termos de licença e com o aviso de privacidade** e selecione **Continuar**.
1. Na página **Configurações Expressas** do assistente do **Microsoft Azure Active Directory Connect**, selecione a opção **Personalizar**.
1. Na página **Instalar componentes necessários**, deixe todas as opções de configuração opcionais desmarcadas e selecione **Instalar**.
1. Na página de **Entrada do usuário**, verifique se somente a ** Sincronização de Hash de Senha** está habilitada e selecione**Avançar**.
1. Na página **Conectar ao Microsoft Entra**, autentique-se usando as credenciais da conta de usuário do **aadsyncuser** que você criou no exercício anterior e selecione **Avançar**. 

   > **Observação**: Forneça o atributo userPrincipalName da conta **aadsyncuser** que você registrou anteriormente nesse exercício e especifique a senha definida anteriormente neste laboratório como sua senha.

1. Na página **Conectar seus diretórios**, selecione o botão **Adicionar Diretório** à direita da entrada da floresta **adatum.com**.
1. Na janela da **conta da floresta do AD**, verifique se a opção de **Criar nova conta do AD** está selecionada, especifique as seguintes credenciais e selecione**OK**:

   |Configuração|Valor|
   |---|---|
   |Nome do usuário|**ADATUM\Student**|
   |Senha|**Pa55w.rd1234**|

1. De volta à página **Conectar seus diretórios**, verifique se a entrada **adatum.com** aparece como um diretório configurado e selecione **Avançar**
1. Na página de **configuração de entrada do Microsoft Entra** observe o aviso informando que **Os usuários não poderão entrar no Microsoft Entra com credenciais locais se o sufixo UPN não corresponder a um nome de domínio verificado**, habilite a caixa de seleção**Continuar sem corresponder a todos os sufixos UPN para o domínio verificado** e selecione **Avançar**.

   > **Observação**: Isso é esperado, já que o locatário do Microsoft Entra não tem um domínio DNS personalizado verificado que corresponda a um dos sufixos UPN do AD DS do **adatum.com**.

1. Na página **Filtragem de Domínio e UO**, selecione a opção **Sincronizar domínios e OUs selecionados**, expanda o nó adatum.com, desmarque todas as caixas de seleção, marque apenas a caixa de seleção ao lado da UO **ToSync** e selecione **Avançar**.
1. Na página **Identificar exclusivamente os usuários**, aceite as configurações padrão e selecione **Avançar**.
1. Na página **Filtrar usuários e dispositivos**, aceite as configurações padrão e selecione **Avançar**.
1. Na página **Recursos opcionais**, aceite as configurações padrão e selecione **Avançar**.
1. Na página **Pronto para configurar**, verifique se a caixa de seleção** Iniciar o processo de sincronização quando a configuração for concluída** está selecionada e selecione **Instalar**.

   > **Observação**: a instalação levará cerca de dois minutos.

1. Examine as informações na página **Configuração concluída** e selecione **Sair** para fechar a janela do **Microsoft Azure Active Directory Connect**.
1. Na sessão da Área de Trabalho Remota para **az140-dc-vm11**, na janela do Microsoft Edge que exibe o portal do Azure, navegue até a folha **Usuários – Todos os usuários** do locatário do Microsoft Entra do Adatum Lab.
1. Na folha **Usuários \| Todos os usuários**, observe que a lista de objetos de usuário inclui a listagem de contas de usuário do AD DS criadas anteriormente nesse laboratório, com a entrada **Sim** aparecendo na coluna **Sincronização local habilitada**.

   > **Observação**: Talvez seja necessário aguardar alguns minutos e atualizar a página do navegador para que as contas de usuário do AD DS apareçam.
