---
lab:
  title: 'Laboratório: Implantar e gerenciar pools de hosts e hosts usando o PowerShell'
  module: 'Module 2: Implement a WVD Infrastructure'
---

# Laboratório - implantar e gerenciar pools de hosts e hosts usando o PowerShell
# Manual de laboratório do aluno

## Dependências de laboratório

- Uma assinatura do Azure que você usará neste laboratório.
- Uma conta da Microsoft ou uma conta do Azure AD com a função de Proprietário ou Colaborador na assinatura do Azure a qual você usará neste laboratório e com a função de Administrador Global no locatário do Azure AD associado a essa assinatura do Azure.
- O laboratório completo **Preparar para a implantação da Área de Trabalho Virtual do Azure (AD DS)**

## Tempo estimado

60 minutos

## Cenário do laboratório

Você precisa automatizar a implantação de pools de hosts e hosts da Área de Trabalho Virtual do Azure usando o PowerShell em um ambiente do AD DS (Active Directory Domain Services).

## Objetivos
  
Depois de realizar este laboratório, você será capaz de:

- Implantar pools de hosts e hosts da Área de Trabalho Virtual do Azure usando o PowerShell
- Adicionar hosts ao pool de host da Área de Trabalho Virtual do Azure usando o PowerShell

## Arquivos de laboratório

-  \\\\AZ-140\\AllFiles\\Labs\\02\\az140-24_azuredeployhp3.json
-  \\\\AZ-140\\AllFiles\\Labs\\02\\az140-24_azuredeployhp3.parameters.json

## Instruções

### Exercício 1: Implementar pools de host e hosts de sessão da Área de Trabalho Virtual do Azure usando o PowerShell
  
As principais tarefas deste exercício são as seguintes:

1. Preparar-se para a implantação do pool de hosts da Área de Trabalho Virtual do Azure usando o PowerShell
1. Criar um pool de host da Área de Trabalho Virtual do Azure usando o PowerShell
1. Executar uma implantação baseada em modelo de uma VM do Azure executando o Windows 10 Enterprise usando o PowerShell
1. Adicionar uma VM do Azure executando o Windows 10 Enterprise como um host da sessão ao pool de host da Área de Trabalho Virtual do Azure usando o PowerShell
1. Verificar a implantação do host da sessão da Área de Trabalho Virtual do Azure

#### Tarefa 1: Preparar-se para a implantação do pool de hosts da Área de Trabalho Virtual do Azure usando o PowerShell

1. No seu computador de laboratório, inicie um navegador da Web, navegue até o [portal do Azure](https://portal.azure.com) e entre fornecendo credenciais de uma conta de usuário com a função Proprietário na assinatura que você usará neste laboratório.
1. No portal do Azure, pesquise e selecione **Máquinas virtuais** e, na folha **Máquinas virtuais**, selecione **az140-dc-vm11**.
1. Na folha **az140-dc-vm11**, selecione **Conectar**, no menu suspenso, selecione **Bastion**, no guia **Bastion** da folha **Conectar \| az140-dc-vm11**, selecione **Usar o Bastion**.
1. Quando solicitado, providencie as seguintes credenciais e selecione **Conectar**:

   |Configuração|Valor|
   |---|---|
   |Nome do usuário|**Aluno**|
   |Senha|**Pa55w.rd1234**|

1. Na sessão do Bastion para **az140-dc-vm11**, inicie o **ISE do Windows PowerShell** como administrador.
1. Na sessão do Bastion para **az140-cl-vm11**, no ISE do Windows Power Shell** no console do Administrador**, execute o seguinte para identificar o nome diferenciado da unidade organizacional chamada **WVDInfra** que hospedará os objetos de computador dos hosts de sessão do pool da Área de Trabalho Virtual do Azure:

   ```powershell
   (Get-ADOrganizationalUnit -Filter "Name -eq 'WVDInfra'").distinguishedName
   ```

1. Na sessão do Bastion para **az140-cl-vm11**, no ISE do Windows Power Shell** no painel de script do Administrador**, execute o seguinte para identificar o sufixo UPN da conta do **ADATUM\\ do Aluno** que você usará para ingressar os hosts da Área de Trabalho Virtual do Azure no domínio do AD DS (**student@adatum.com**):

   ```powershell
   (Get-ADUser -Filter {sAMAccountName -eq 'student'} -Properties userPrincipalName).userPrincipalName
   ```

1. Na sessão do Bastion para **az140-cl-vm11**, no ISE do Windows Power Shell** no painel de script do Administrador**, execute o seguinte para instalar o módulo do PowerShell DesktopVirtualization (quando solicitado, clique em **Sim para Todos**):

   ```powershell
   Install-Module -Name Az.DesktopVirtualization -Force
   ```

   > **Observação**: Ignore todos os avisos relativos aos módulos existentes do PowerShell em uso.

1. Na sessão do Bastion para **az140-dc-vm11**, inicie o Microsoft Edge e navegue até o [portal do Azure](https://portal.azure.com). Se solicitado, entre usando as credenciais do Azure AD da conta de usuário com a função Proprietário na assinatura que você está usando neste laboratório.
1. Na sessão do Bastion para **az140-dc-vm11**, no portal do Azure, use a caixa de texto **Pesquisar recursos, serviços e documentos** na parte superior da página do portal do Azure para pesquisar e navegar até **Redes virtuais** e, na folha** Redes virtuais**, selecione **az140-adds-vnet11**. 
1. Na folha **az140-adds-vnet11**, selecione **Sub-redes**, na folha **Sub-redes**, selecione **+ Sub-rede**, na folha **Adicionar sub-rede**, especifique as seguintes configurações (deixe todas as outras configurações com seus valores padrão) e clique em **Salvar**:

   |Configuração|Valor|
   |---|---|
   |Nome|**hp3-Subnet**|
   |Intervalo de endereços da sub-rede|**10.0.3.0/24**|

1. Na sessão do Bastion para **az140-dc-vm11**, no portal do Azure, use a caixa de texto **Pesquisar recursos, serviços e documentos** na parte superior da página do portal do Azure para pesquisar e navegar até **Grupos de segurança de rede** e, na folha **Grupos de Segurança de Rede**, selecione o grupo de segurança no grupo de recursos **az140-11-RG**.
1. Na folha do grupo de segurança de rede, no menu vertical à esquerda, na seção **Configurações**, clique em **Propriedades**.
1. Na folha **Propriedades**, clique no ícone **Copiar para área de transferência** no lado direito da caixa de texto **ID do Recurso**. 

   > **Observação**: O valor deve ser semelhante ao formato `/subscriptions/de8279a3-0675-40e6-91e2-5c3728792cb5/resourceGroups/az140-11-RG/providers/Microsoft.Network/networkSecurityGroups/az140-cl-vm11-nsg`, embora a ID da assinatura seja diferente. Registre-o, pois você precisará dele na próxima tarefa.

#### Tarefa 2: Criar um pool de host da Área de Trabalho Virtual do Azure usando o PowerShell

1. Na sessão do Bastion para **az140-cl-vm11**, no ISE do Windows Power Shell** no painel de script do Administrador**, execute o seguinte para entrar em sua assinatura do Azure:

   ```powershell
   Connect-AzAccount
   ```

1. Quando solicitado, forneça as credenciais da conta de usuário com a função Proprietário na assinatura que você está usando neste laboratório.
1. Na sessão do Bastion para **az140-cl-vm11**, no ISE do Windows Power Shell** no painel de script do Administrador**, execute o seguinte para identificar a região do Azure que hospeda a rede virtual do Azure **az140-adds-vnet11**:

   ```powershell
   $location = (Get-AzVirtualNetwork -ResourceGroupName 'az140-11-RG' -Name 'az140-adds-vnet11').Location
   ```

1. Na sessão do Bastion para **az140-cl-vm11**, no ISE do Windows Power Shell** no painel de script do Administrador**, execute o seguinte para criar um grupo de recursos que hospedará o pool de hosts e seus recursos:

   ```powershell
   $resourceGroupName = 'az140-24-RG'
   New-AzResourceGroup -Location $location -Name $resourceGroupName
   ```

1. Na sessão do Bastion para **az140-cl-vm11**, no ISE do Windows Power Shell** no painel do Administrador**, execute o seguinte para criar um pool de hosts vazio:

   ```powershell
   $hostPoolName = 'az140-24-hp3'
   $workspaceName = 'az140-24-ws1'
   $dagAppGroupName = "$hostPoolName-DAG"
   New-AzWvdHostPool -ResourceGroupName $resourceGroupName -Name $hostPoolName -WorkspaceName $workspaceName -HostPoolType Pooled -LoadBalancerType BreadthFirst -Location $location -DesktopAppGroupName $dagAppGroupName -PreferredAppGroupType Desktop 
   ```

   > **Observação**: O cmdlet **New-AzWvdHostPool** permite que você crie um pool de hosts, um workspace e o grupo de aplicativos da área de trabalho, bem como registre o grupo de aplicativos da área de trabalho com o workspace. Você tem a opção de criar um novo workspace ou usar um existente.

1. Na sessão do Bastion para **az140-cl-vm11**, no ISE do Windows Power Shell** no console do Administrador**, execute o seguinte para recuperar o atributo objectID do grupo do Azure AD chamado **az140-wvd-pooled**:

   ```powershell
   $aadGroupObjectId = (Get-AzADGroup -DisplayName 'az140-wvd-pooled').Id
   ```

1. Na sessão do Bastion para **az140-cl-vm11**, no ISE do Windows Power Shell** no console do Administrador**, execute o seguinte para atribuir o grupo do Azure AD chamado **az140-wvd-pooled** ao grupo de aplicativos da área de trabalho padrão do pool de hosts recém-criado:

   ```powershell
   $roleDefinitionName = 'Desktop Virtualization User'
   New-AzRoleAssignment -ObjectId $aadGroupObjectId -RoleDefinitionName $roleDefinitionName -ResourceName $dagAppGroupName -ResourceGroupName $resourceGroupName -ResourceType 'Microsoft.DesktopVirtualization/applicationGroups'
   ```

#### Tarefa 3: Executar uma implantação baseada em modelo de uma VM do Azure executando o Windows 10 Enterprise usando o PowerShell

1. No computador do laboratório, navegue até a conta de armazenamento implantada. Na folha Compartilhamento de Arquivos, selecione o compartilhamento de arquivos **az140-22-profiles**.

1. Selecione **Upload** e carregue os arquivos de laboratório **\\\\AZ-140\\AllFiles\\Labs\\02\\az140-24_azuredeployhp3.jsone** e **\\\\AZ-140\\AllFiles\\Labs\\02\\az140-24_azuredeployhp3.parameters.json** para compartilhamento de arquivos.

1. Na sessão do Bastion para **az140-dc-vm11**, abra o Explorador de Arquivos e navegue até o Z: configurado anteriormente ou a letra da unidade atribuída à conexão com o Compartilhamento de Arquivos. Copie os arquivos de implantação carregados para **C:\AllFiles\Labs\02**.

1. Na sessão do Bastion para **az140-cl-vm11**, no ISE do Windows Power Shell** no console do Administrador**, execute o seguinte para implantar uma VM do Azure executando o Windows 10 Enterprise (várias sessões) que servirá como um host da sessão da Área de Trabalho Virtual do Azure no pool de hosts criado na tarefa anterior:

   ```powershell
   $resourceGroupName = 'az140-24-RG'
   $location = (Get-AzResourceGroup -ResourceGroupName $resourceGroupName).Location
   New-AzResourceGroupDeployment `
     -ResourceGroupName $resourceGroupName `
     -Location $location `
     -Name az140lab24hp3Deployment `
     -TemplateFile C:\AllFiles\Labs\02\az140-24_azuredeployhp3.json `
     -TemplateParameterFile C:\AllFiles\Labs\02\az140-24_azuredeployhp3.parameters.json
   ```

   > **Observação**: Aguarde a conclusão da implantação antes de prosseguir para a próxima tarefa. Isso pode levar cerca de cinco minutos. 

   > **Observação**: A implantação usa um modelo do Gerenciador de Recursos do Azure para provisionar uma VM do Azure e aplica uma extensão de VM que ingressa automaticamente o sistema operacional no domínio **adatum.com** do AD DS.

1. Na sessão do Bastion para **az140-cl-vm11**, no ISE do Windows Power Shell** no console do Administrador**, execute o seguinte para verificar se o host da terceira sessão foi associado com êxito ao domínio **adatum.com** do AD DS:

   ```powershell
   Get-ADComputer -Filter "sAMAccountName -eq 'az140-24-p3-0$'"
   ```

#### Tarefa 4: Adicionar uma VM do Azure executando o Windows 10 Enterprise como host ao pool de host da Área de Trabalho Virtual do Azure usando o PowerShell

1. Na sessão do Bastion para **az140-dc-vm11**, na janela do navegador que exibe o portal do Azure, pesquise e selecione **Máquinas virtuais** e, na folha **Máquinas virtuais**, na lista de máquinas virtuais, selecione **az140-24-p3-0**.
1. Na folha **az140-24-p3-0 **, selecione **Conectar**, no menu suspenso, selecione **RDP**, na guia **RDP** da folha **az140-24-p3-0\| Conectar**, na lista suspensa **endereço IP**, selecione a entrada **Endereço IP privado (10.0.3.4)** e selecione **Baixar Arquivo RDP**.
1. Quando solicitado, entre com as seguintes credenciais:

   |Configuração|Valor|
   |---|---|
   |Nome do usuário|**ADATUM\\Aluno**|
   |Senha|**Pa55w.rd1234**|

1. Na sessão da Área de Trabalho Remota para **az140-24-p3-0**, inicie o ISE do **Windows PowerShell** como administrador.
1. Na sessão da Área de Trabalho Remota para **az140-24-p3-0**, **do ISE do Windows PowerShell no painel de script do Administrador**, execute o seguinte para criar uma pasta que hospedará arquivos necessários para adicionar a VM do Azure recém-implantada como um host da sessão ao pool de hosts provisionado anteriormente neste laboratório:

   ```powershell
   $labFilesFolder = 'C:\AllFiles\Labs\02'
   New-Item -ItemType Directory -Path $labFilesFolder
   ```

   >**Observe** o uso do constructo [T] para copiar sobre os cmdlets do PowerShell. Em alguns casos, o texto copiado pode estar incorreto, como o sinal $ mostrando como um caractere de número 4. Você precisará corrigi-los antes de emitir o cmdlet. Copie para o painel de **script** do ISE do PowerShell, faça as correções lá e realce o texto corrigido e pressione** F8 **(**Executar Seleção**).

1. Na sessão da Área de Trabalho Remota para **az140-24-p3-0**, **do ISE do Windows PowerShell no painel de script do Administrador**, execute o seguinte para baixar os instaladores do Agente de Área de Trabalho Virtual do Azure e do Carregador de Inicialização, necessários para adicionar o host da sessão ao pool de hosts:

   ```powershell
   $webClient = New-Object System.Net.WebClient
   $wvdAgentInstallerURL = 'https://query.prod.cms.rt.microsoft.com/cms/api/am/binary/RWrmXv'
   $wvdAgentInstallerName = 'WVD-Agent.msi'
   $webClient.DownloadFile($wvdAgentInstallerURL,"$labFilesFolder/$wvdAgentInstallerName")
   $wvdBootLoaderInstallerURL = 'https://query.prod.cms.rt.microsoft.com/cms/api/am/binary/RWrxrH'
   $wvdBootLoaderInstallerName = 'WVD-BootLoader.msi'
   $webClient.DownloadFile($wvdBootLoaderInstallerURL,"$labFilesFolder/$wvdBootLoaderInstallerName")
   ```

1. Na sessão da Área de Trabalho Remota para **az140-24-p3-0**, **do ISE do Windows PowerShell do painel de script do Administrador**, execute o seguinte para instalar a versão mais recente do módulo PowerShellGet (selecione **Sim** quando solicitado a confirmação):

   ```powershell
   [Net.ServicePointManager]::SecurityProtocol = [Net.SecurityProtocolType]::Tls12
   Install-Module -Name PowerShellGet -Force -SkipPublisherCheck
   ```

1. No **ISE do Windows Power Shell no console do Administrador**, execute o seguinte para instalar a versão mais recente do módulo Az.DesktopVirtualization PowerShell:

   ```powershell
   Install-Module -Name Az.DesktopVirtualization -AllowClobber -Force
   Install-Module -Name Az -AllowClobber -Force
   ```

1. No **ISE do Windows Power Shell no console do Administrador**, execute o seguinte para modificar a política de execução do PowerShell e entrar em sua assinatura do Azure:

   ```powershell
   Set-ExecutionPolicy -ExecutionPolicy Bypass -Scope CurrentUser -Force
   Connect-AzAccount
   ```

1. Quando solicitado, forneça as credenciais da conta de usuário com a função Proprietário na assinatura que você está usando neste laboratório.
1. Na sessão Área de Trabalho Remota para **az140-24-p3-0**, no **ISE do Windows PowerShell no console do Administrador**, execute o seguinte para gerar o token necessário para ingressar novos hosts de sessão no pool provisionado anteriormente neste exercício:

   ```powershell
   $resourceGroupName = 'az140-24-RG'
   $hostPoolName = 'az140-24-hp3'
   $registrationInfo = New-AzWvdRegistrationInfo -ResourceGroupName $resourceGroupName -HostPoolName $hostPoolName -ExpirationTime $((get-date).ToUniversalTime().AddDays(1).ToString('yyyy-MM-ddTHH:mm:ss.fffffffZ'))
   ```
   > **Observação**: Um token de registro é necessário para autorizar um host da sessão a ingressar no pool de host. O valor da data de validade do token deve estar entre uma hora e um mês a partir da data e hora atuais.

1. Na sessão da Área de Trabalho Remota para **az140-24-p3-0**, **do ISE do Windows PowerShell do console do Administrador**, execute o seguinte para instalar o Agente de Área de Trabalho Virtual do Azure:

   ```powershell
   Set-Location -Path $labFilesFolder
   Start-Process -FilePath 'msiexec.exe' -ArgumentList "/i $WVDAgentInstallerName", "/quiet", "/qn", "/norestart", "/passive", "REGISTRATIONTOKEN=$($registrationInfo.Token)", "/l* $labFilesFolder\AgentInstall.log" | Wait-Process
   ```

1. Na sessão da Área de Trabalho Remota para **az140-24-p3-0**, **do ISE do Windows PowerShell no console do Administrador**, execute o seguinte para instalar o Carregador de Inicialização da Área de Trabalho Virtual do Azure:

   ```powershell
   Start-Process -FilePath "msiexec.exe" -ArgumentList "/i $wvdBootLoaderInstallerName", "/quiet", "/qn", "/norestart", "/passive", "/l* $labFilesFolder\BootLoaderInstall.log" | Wait-process
   ```

#### Tarefa 5: Verificar a implantação do host da Área de Trabalho Virtual do Azure

1. Alterne para o computador de laboratório, no navegador da Web que exibe o portal do Azure, pesquise e selecione a **Área de Trabalho Virtual do Azure**, na folha **Área de Trabalho Virtual do Azure**, selecione **Pools de host** e, na folha **Pools de host\| da Área de Trabalho Virtual do Azure**, selecione a entrada **az140-24-hp3** que representa o pool recém-modificado.
1. Na folha **az140-24-hp3 **, no menu vertical no lado esquerdo, na seção **Gerenciar**, clique em **Hosts de sessão**. 
1. Na folha de hosts de sessão **az140-24-hp3\| **, verifique se a implantação inclui um único host.

#### Tarefa 6: Gerenciar grupos de aplicativos usando o PowerShell

1. No computador do laboratório, alterne para a sessão do Bastion para **az140-dc-vm11**, no **ISE do Windows PowerShell no console do Administrador**, execute o seguinte para criar um grupo de Aplicativos Remotos:

   ```powershell
   $subscriptionId = (Get-AzContext).Subscription.Id
   $appGroupName = 'az140-24-hp3-Office365-RAG'
   $resourceGroupName = 'az140-24-RG'
   $hostPoolName = 'az140-24-hp3'
   $location = (Get-AzVirtualNetwork -ResourceGroupName 'az140-11-RG' -Name 'az140-adds-vnet11').Location
   New-AzWvdApplicationGroup -Name $appGroupName -ResourceGroupName $resourceGroupName -ApplicationGroupType 'RemoteApp' -HostPoolArmPath "/subscriptions/$subscriptionId/resourcegroups/$resourceGroupName/providers/Microsoft.DesktopVirtualization/hostPools/$hostPoolName"-Location $location
   ```

1. Na sessão do Bastion para **az140-cl-vm11**, no ISE do Windows Power Shell** no console do Administrador**, execute o seguinte para listar os aplicativos de menu **Iniciar** nos hosts do pool e examinar a saída:

   ```powershell
   Get-AzWvdStartMenuItem -ApplicationGroupName $appGroupName -ResourceGroupName $resourceGroupName | Format-List | more
   ```

   > **Observação**: Para qualquer aplicativo que você deseja publicar, você deve registrar as informações incluídas na saída, incluindo parâmetros como **FilePath**, **IconPath** e **IconIndex**.

1. Na sessão do Bastion para **az140-cl-vm11**, no ISE do Windows Power Shell** no console do ISE do Windows PowerShell**, execute o seguinte para publicar o Microsoft Word:

   ```powershell
   $name = 'Microsoft Word'
   $filePath = 'C:\Program Files\Microsoft Office\root\Office16\WINWORD.EXE'
   $iconPath = 'C:\Program Files\Microsoft Office\Root\VFS\Windows\Installer\{90160000-000F-0000-1000-0000000FF1CE}\wordicon.exe'
   New-AzWvdApplication -GroupName $appGroupName -Name $name -ResourceGroupName $resourceGroupName -FriendlyName $name -Filepath $filePath -IconPath $iconPath -IconIndex 0 -CommandLineSetting 'DoNotAllow' -ShowInPortal:$true
   ```

1. Na sessão do Bastion para **az140-cl-vm11**, no ISE do Windows Power Shell** no console do ISE do Windows PowerShell**, execute o seguinte para publicar o Microsoft Word:

   ```powershell
   $aadGroupObjectId = (Get-AzADGroup -DisplayName 'az140-wvd-remote-app').Id
   New-AzRoleAssignment -ObjectId $aadGroupObjectId -RoleDefinitionName 'Desktop Virtualization User' -ResourceName $appGroupName -ResourceGroupName $resourceGroupName -ResourceType 'Microsoft.DesktopVirtualization/applicationGroups'
   ```

1. Alterne para o computador de laboratório, no navegador da Web que exibe o portal do Azure, na folha de **hosts de sessão\| az140-24-hp3**, no menu vertical à esquerda, na seção **Gerenciar**, selecione **Grupos de aplicativos**.
1. Na folha de **Grupos de aplicativos \| az140-24-hp3**, na lista de grupos de aplicativos, selecione a entrada **az140-24-hp3-Office365-RAG**.
1. Na folha **az140-24-hp3-Office365-RAG**, verifique a configuração do grupo de aplicativos, incluindo os aplicativos e as atribuições.

### Exercício 2: Parar e desalocar VMs do Azure provisionadas no laboratório

As principais tarefas deste exercício são as seguintes:

1. Parar e desalocar VMs do Azure provisionadas no laboratório

>**Observação**: Neste exercício, você desalocará as VMs do Azure provisionadas neste laboratório para minimizar os encargos de computação correspondentes

#### Tarefa 1: Desalocar VMs do Azure provisionadas no laboratório

1. Alterne para o computador do laboratório e, na janela do navegador da Web que exibe o portal do Azure, abra a sessão do Shell do **PowerShell** no painel do **Cloud Shell**.
1. Na sessão do PowerShell no painel do Cloud Shell, execute o seguinte para listar todas as VMs do Azure criadas neste laboratório:

   ```powershell
   Get-AzVM -ResourceGroup 'az140-24-RG'
   ```

1. Na sessão do PowerShell no painel do Cloud Shell, execute o seguinte para parar e desalocar todas as VMs do Azure que você criou neste laboratório:

   ```powershell
   Get-AzVM -ResourceGroup 'az140-24-RG' | Stop-AzVM -NoWait -Force
   ```

   >**Observação**: O comando é executado de modo assíncrono (conforme determinado pelo parâmetro -NoWait), portanto, embora você possa executar outro comando do PowerShell imediatamente depois na mesma sessão do PowerShell, levará alguns minutos antes das VMs do Azure serem realmente interrompidas e desalocadas.
