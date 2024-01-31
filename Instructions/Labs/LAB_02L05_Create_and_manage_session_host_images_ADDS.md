---
lab:
  title: 'Laboratório: Criar e gerenciar imagens de host de sessão (AD DS)'
  module: 'Module 2: Implement an AVD Infrastructure'
---

# Laboratório: Criar e gerenciar imagens de host de sessão (AD DS)
# Manual de laboratório do aluno

## Dependências do laboratório

- O nome da assinatura do Azure que você usará nesse laboratório.
- Uma conta Microsoft ou uma conta do Microsoft Entra com a função de Proprietário ou Colaborador na assinatura do Azure que você usará neste laboratório e com a função de Administrador Global no locatário do Microsoft Entra associado a essa assinatura do Azure.
- O laboratório concluído: **Preparar para a implantação da Área de Trabalho Virtual do Azure (AD DS)**

## Tempo estimado

60 minutos

## Cenário do laboratório

Você precisa criar e gerenciar imagens de host da Área de Trabalho Virtual do Azure em um ambiente do Microsoft Entra DS.

## Objetivos
  
Depois de realizar este laboratório, você será capaz de:

- Criar e gerenciar imagens de host de sessão

## Arquivos do laboratório

-  \\\\AZ-140\\AllFiles\\Labs\\02\\az140-25_azuredeployvm25.json
-  \\\\AZ-140\\AllFiles\\Labs\\02\\az140-25_azuredeployvm25.parameters.json

## Instruções

### Exercício 1: Criar e gerenciar imagens de host de sessão
  
As principais tarefas desse exercício são as seguintes:

1. Preparar para a configuração de uma imagem de host da Área de Trabalho Virtual do Azure
1. Implantar o Azure Bastion
1. Configurar uma imagem de host da Área de Trabalho Virtual do Azure
1. Criar uma imagem de host da Área de Trabalho Virtual do Azure
1. Provisionar um pool de hosts da Área de Trabalho Virtual do Azure usando a imagem personalizada

#### Tarefa 1: Preparar para a configuração de uma imagem de host da Área de Trabalho Virtual do Azure

1. No seu computador de laboratório, inicie um navegador da Web, navegue até o [portal do Azure](https://portal.azure.com) e entre fornecendo credenciais de uma conta de usuário com a função Proprietário na assinatura que você usará nesse laboratório.
1. No portal do Azure, abra o painel do **Cloud Shell** selecionando no ícone da barra de ferramentas diretamente à direita da caixa de texto de pesquisa.
1. Se for solicitado que você selecione **Bash** ou **PowerShell**, selecione **PowerShell**. 
1. No computador de laboratório, no navegador da Web que exibe o portal do Azure, na sessão do PowerShell no painel do Cloud Shell, execute o seguinte para criar um grupo de recursos que conterá a imagem do host da Área de Trabalho Virtual do Azure:

   ```powershell
   $vnetResourceGroupName = 'az140-11-RG'
   $location = (Get-AzResourceGroup -ResourceGroupName $vnetResourceGroupName).Location
   $imageResourceGroupName = 'az140-25-RG'
   New-AzResourceGroup -Location $location -Name $imageResourceGroupName
   ```

1. No portal do Azure, na barra de ferramentas do painel do Cloud Shell, selecione o ícone **Carregar/Baixar arquivos**, no menu suspenso, selecione **Carregar**e carregue os arquivos **\\\\AZ-140\\AllFiles\\Labs\\02\\az140-25_azuredeployvm25.json** e **\\\\AZ-140\\AllFiles\\Labs\\02\\az140-25_azuredeployvm25.parameters.json** no diretório inicial do Cloud Shell.
1. Na sessão do PowerShell no painel do Cloud Shell, execute o seguinte para implantar uma VM do Azure executando o Windows 10 que servirá como um cliente da Área de Trabalho Virtual do Azure na sub-rede recém-criada:

   ```powershell
   New-AzResourceGroupDeployment `
     -ResourceGroupName $imageResourceGroupName `
     -Name az140lab0205vmDeployment `
     -TemplateFile $HOME/az140-25_azuredeployvm25.json `
     -TemplateParameterFile $HOME/az140-25_azuredeployvm25.parameters.json
   ```

   > **Observação**: Aguarde a conclusão da implantação antes de prosseguir para o próximo exercício. A implantação pode levar cerca de 10 minutos.

#### Tarefa 2: Implantar o Azure Bastion 

> **Observação**: O Azure Bastion permite a conexão com as VMs do Azure sem pontos de extremidade públicos implantados na tarefa anterior deste exercício, ao mesmo tempo em que fornece proteção contra explorações de força bruta direcionadas às credenciais de nível do sistema operacional.

> **Observação**: Verifique se o navegador tem a funcionalidade pop-up habilitada.

1. Na janela do navegador da Web que exibe o portal do Azure, abra outra guia e navegue até o portal do Azure.
1. No portal do Azure, abra o painel do **Cloud Shell** selecionando no ícone da barra de ferramentas diretamente à direita da caixa de texto de pesquisa.
1. Na sessão do PowerShell no painel do Cloud Shell, execute o seguinte para adicionar uma sub-rede chamada **AzureBastionSubnet** à rede virtual chamada **az140-25-vnet** criada anteriormente nesse exercício:

   ```powershell
   $resourceGroupName = 'az140-25-RG'
   $vnet = Get-AzVirtualNetwork -ResourceGroupName $resourceGroupName -Name 'az140-25-vnet'
   $subnetConfig = Add-AzVirtualNetworkSubnetConfig `
     -Name 'AzureBastionSubnet' `
     -AddressPrefix 10.25.254.0/24 `
     -VirtualNetwork $vnet
   $vnet | Set-AzVirtualNetwork
   ```

1. Feche o painel do Cloud Shell.
1. No portal do Azure, pesquise e selecione **Bastions** e, na folha **Bastions**, selecione **+ Criar**.
1. Na guia **Noções Básicas** da folha **Criar um Bastion**, especifique as seguintes configurações e selecione **Examinar + criar**:

   |Configuração|Valor|
   |---|---|
   |Assinatura|o nome da assinatura do Azure que você está usando neste laboratório|
   |Grupo de recursos|**az140-25-RG**|
   |Nome|**az140-25-bastion**|
   |Region|a mesma região do Azure na qual você implantou os recursos nas tarefas anteriores deste exercício|
   |Camada|**Basic**|
   |Rede virtual|**az140-25-vnet**|
   |Sub-rede|**AzureBastionSubnet (10.25.254.0/24)**|
   |Endereço IP público|**Criar novo**|
   |Nome do IP público|**az140-25-vnet-ip**|

1. Na guia **Revisar + criar** da folha **Criar um Bastion**, selecione **Criar**:

   > **Observação**: Aguarde a conclusão da implantação antes de prosseguir para o próximo exercício. A implantação pode levar cerca de cinco minutos.

#### Tarefa 3: Configurar uma imagem de host da Área de Trabalho Virtual do Azure

1. No portal do Azure, pesquise e selecione **Máquinas virtuais** e, na folha **Máquinas virtuais**, selecione **az140-25-vm0-0**.
1. Na folha **az140-25-vm0**, selecione **Conectar**, no menu suspenso, selecione **Bastion**, na guia **Bastion** da folha **az140-25-vm0 \| Conectar**, selecione **Usar Bastion**.
1. Quando solicitado, forneça as seguintes credenciais e selecione **Conectar**:

   |Configuração|Valor|
   |---|---|
   |Nome do usuário|**Aluno**|
   |Senha|**Pa55w.rd1234**|

   > **Observação**: Você começará instalando binários do FSLogix.

1. Na sessão do Bastion para **az140-25-vm0**, inicie o **ISE do Windows PowerShell** como administrador.
1. Na sessão do Bastion para **az140-25-vm0**, do **Administrador: Console ISE do Windows PowerShell**: execute o seguinte para criar uma pasta que você usará como um local temporário para a configuração da imagem:

   ```powershell
   New-Item -Type Directory -Path 'C:\Allfiles\Labs\02' -Force
   ```

1. Na sessão do Bastion para **az140-25-vm0**, inicie o Microsoft Edge, navegue até a [página de download do FSLogix](https://aka.ms/fslogix_download), baixe binários de instalação compactada FSLogix na pasta **C:\\Allfiles\\Labs\\02** e, no Explorador de Arquivos, extraia a subpasta **x64** na mesma pasta.
1. Na sessão do Bastion para **az140-25-vm0**, alterne para o **Administrador: Janela ISE do Windows PowerShell** e do **Administrador: Console do ISE do Windows PowerShell**: execute o seguinte para executar a instalação por computador do OneDrive:

   ```powershell
   Start-Process -FilePath 'C:\Allfiles\Labs\02\x64\Release\FSLogixAppsSetup.exe' -ArgumentList '/quiet' -Wait
   ```

   > **Observação**: aguarde a conclusão da instalação. Isso pode levar cerca de um minuto. Se a instalação disparar uma reinicialização, conecte-se novamente a **az140-25-vm0**.

   > **Observação**: Em seguida, você irá percorrer a instalação e a configuração do Microsoft Teams (para fins de aprendizado, já que o Teams já está presente na imagem usada para esse laboratório).

1. Na sessão do Bastion para **az140-25-vm0**, clique com o botão direito do mouse em **Iniciar**, no menu com o botão direito do mouse, selecione **Executar**. Na caixa de diálogo **Executar**, na caixa de texto **Abrir**, digite **cmd** e pressione a tecla **Enter** para iniciar o **Prompt de Comando**.
1. **No Administrador: Janela C:\windows\system32\cmd.exe**: no prompt de comando, execute o seguinte para se preparar para a instalação por computador do Microsoft Teams:

   ```cmd
   reg add "HKLM\Software\Microsoft\Teams" /v IsWVDEnvironment /t REG_DWORD /d 1 /f
   ```

1. Na sessão Bastion para **az140-25-vm0**, no Microsoft Edge, navegue até [a página de download do Pacote Redistribuível do Visual C++](https://aka.ms/vs/16/release/vc_redist.x64.exe), salve **VC_redist.x64** na **pasta C:\\Allfiles\\Labs\\02**.
1. Na sessão do Bastion para **az140-25-vm0**, alterne para o **Administrador: Janela C:\windows\system32\cmd.exe** e, no prompt de comando, execute o seguinte para executar a instalação do Pacote Redistribuível do Visual C++:

   ```cmd
   C:\Allfiles\Labs\02\vc_redist.x64.exe /install /passive /norestart /log C:\Allfiles\Labs\02\vc_redist.log
   ```

1. Na sessão Bastion para **az140-25-vm0**, no Microsoft Edge, navegue até a página de documentação intitulada [ Implantar o aplicativo de área de trabalho do Teams na VM,](https://docs.microsoft.com/en-us/microsoftteams/teams-for-vdi#deploy-the-teams-desktop-app-to-the-vm) clique no link da **versão de 64 bits ** e, quando solicitado, salve o arquivo **Teams_windows_x64.msi** na pasta **C:\\Allfiles\\Labs\\02**.
1. Na sessão do Bastion para **az140-25-vm0**, alterne para o **Administrador: Janela C:\windows\system32\cmd.exe** e, no prompt de comando, execute o seguinte para executar a instalação por computador do Microsoft Teams:

   ```cmd
   msiexec /i C:\Allfiles\Labs\02\Teams_windows_x64.msi /l*v C:\Allfiles\Labs\02\Teams.log ALLUSER=1
   ```

   > **Observação**: O instalador dá suporte aos parâmetros ALLUSER=1 e ALLUSERS=1. O parâmetro ALLUSER=1 destina-se à instalação por computador em ambientes VDI. O parâmetroAllUsers = 1pode ser usado em ambientes não VDI e VDI. 
   > **Observe** que se você encontrar um erro informando que **Outra versão do produto já está instalada**, conclua as seguintes etapas: Acesse **Painel de Controle > Programas > Programas e Recursos**, clique com o botão direito do mouse no programa **Instalador do Teams em Todo o Computador** e selecione **Desinstalar**. Prossiga com a remoção do programa e execute novamente a etapa 13 acima. 

1. Na sessão do Bastion para **az140-25-vm0-0**, inicie o **ISE do PowerShell do Windows** como Administrador e, do**Administrador: Console ISE do Windows PowerShell**: execute o seguinte para instalar o Microsoft Edge Chromium (para fins de aprendizado, já que o Edge já está presente na imagem usada para este laboratório).:

   ```powershell
   Start-BitsTransfer -Source "https://aka.ms/edge-msi" -Destination 'C:\Allfiles\Labs\02\MicrosoftEdgeEnterpriseX64.msi'
   Start-Process -Wait -Filepath msiexec.exe -Argumentlist "/i C:\Allfiles\Labs\02\MicrosoftEdgeEnterpriseX64.msi /q"
   ```

   > **Observação**: aguarde a conclusão da instalação. Isso pode levar cerca de dois minutos.

   > **Observação**: ao operar em um ambiente de vários idiomas, talvez seja necessário instalar pacotes de idiomas. Para obter detalhes sobre este procedimento, consulte o artigo do Microsoft Docs [Adicionar pacotes de idiomas a uma imagem de várias sessões do Windows 10](https://docs.microsoft.com/en-us/azure/virtual-desktop/language-packs).

   > **Observação**: em seguida, você desabilitará as Atualizações Automáticas do Windows, desabilitará o Sensor de Armazenamento, configurará o redirecionamento de fuso horário e configurará a coleção de telemetria. Em geral, primeiro você deve aplicar todas as atualizações atuais primeiro. Nesse laboratório, você ignorará esta etapa para minimizar a duração do laboratório.

1. Na sessão do Bastion para **az140-25-vm0**, alterne para o **Administrador: Janela C:\windows\system32\cmd.exe**e, no prompt de comando, execute o seguinte para desabilitar Atualizações Automáticas:

   ```cmd
   reg add "HKLM\SOFTWARE\Policies\Microsoft\Windows\WindowsUpdate\AU" /v NoAutoUpdate /t REG_DWORD /d 1 /f
   ```

1. **No Administrador: Janela C:\windows\system32\cmd.exe**, no prompt de comando, execute o seguinte para desabilitar o Sentido de Armazenamento:

   ```cmd
   reg add "HKCU\Software\Microsoft\Windows\CurrentVersion\StorageSense\Parameters\StoragePolicy" /v 01 /t REG_DWORD /d 0 /f
   ```

1. **No Administrador: Janela C:\windows\system32\cmd.exe**, no prompt de comando, execute o seguinte para configurar o redirecionamento de fuso horário:

   ```cmd
   reg add "HKLM\SOFTWARE\Policies\Microsoft\Windows NT\Terminal Services" /v fEnableTimeZoneRedirection /t REG_DWORD /d 1 /f
   ```

1. **No Administrador: Janela C:\windows\system32\cmd.exe**, no prompt de comando, execute o seguinte para desabilitar a coleta de dados de telemetria do hub de comentários:

   ```cmd
   reg add "HKLM\SOFTWARE\Policies\Microsoft\Windows\DataCollection" /v AllowTelemetry /t REG_DWORD /d 0 /f
   ```

1. **No Administrador: Janela C:\windows\system32\cmd.exe**, no prompt de comando, execute o seguinte para excluir a pasta temporária que você criou anteriormente nesta tarefa:

   ```cmd
   rmdir C:\Allfiles /s /q
   ```

1. **No Administrador: Janela C:\windows\system32\cmd.exe**, no prompt de comando, execute o utilitário de Limpeza de Disco e clique em **OK** após a conclusão:

   ```cmd
   cleanmgr /d C: /verylowdisk
   ```

#### Tarefa 4: Criar uma imagem de host da Área de Trabalho Virtual do Azure

1. Na sessão do Bastion para **az140-25-vm0**, no **Administrador: Janela C:\windows\system32\cmd.exe**, no prompt de comando, execute o utilitário sysprep para preparar o sistema operacional para gerar uma imagem e desligá-la automaticamente:

   ```cmd
   C:\Windows\System32\Sysprep\sysprep.exe /oobe /generalize /shutdown /mode:vm
   ```

   > **Observação**: Aguarde a conclusão do processo sysprep. Isso pode levar cerca de dois minutos. Isso irá desligar automaticamente o sistema operacional. 

1. No computador do laboratório, no navegador da Web que exibe o portal do Azure, pesquise e selecione **Máquinas Virtuais** e, na folha **Máquinas Virtuais**, selecione **az140-25-vm0**.
1. Na folha **az140-25-vm0**, na barra de ferramentas acima da seção **Noções Básicas**, clique em **Atualizar**. Verifique se o **Status**da VM do Azure foi alterado para **Parada**. Clique **em Parar** e, quando solicitado a confirmar, clique em **OK** para fazer a transição da VM do Azure para o estado **Parada (desalocada)**.
1. Na folha **az140-25-vm0** verifique se o **Status** da VM do Azure foi alterado para o estado **Parada (desalocada)** e, na barra de ferramentas, clique em **Capturar**. Isso exibirá automaticamente a folha **Criar uma imagem** .
1. Na guia **Noções básicas** da folha **Criar uma imagem **, especifique as seguintes configurações:

   |Configuração|Valor|
   |---|---|
   |Compartilhar a imagem com galeria de computação do Azure|**Sim, fazer o compartilhamento com uma galeria como uma versão de imagem**|
   |Excluir automaticamente esta máquina virtual após a criação da imagem|caixa de seleção desmarcada|
   |Galeria de computação do Azure de destino|o nome de uma nova galeria **az14025imagegallery**|
   |Estado do sistema operacional|**Generalizado**|

1. Na guia **Noções básicas** da folha **Criar uma imagem**, abaixo da caixa de texto **Definição de imagem da VM de destino**, clique em **Criar novo**.
1. NA**Definição criar uma imagem de VM**, especifique as seguintes configurações e clique em **OK**:

   |Configuração|Valor|
   |---|---|
   |Nome da definição de imagem da VM|**az140-25-host-image**|
   |Publisher|**MicrosoftWindowsDesktop**|
   |Oferta|**office-365**|
   |SKU|**win11-22h2-avd-m365**|

1. De volta à guia **Noções básicas** da folha **Criar uma imagem**, especifique as seguintes configurações e clique em **Examinar + criar**:

   |Configuração|Valor|
   |---|---|
   |Número da versão|**1.0.0**|
   |Excluir do mais recente|caixa de seleção desmarcada|
   |Data de fim da vida útil|um ano antes da data atual|
   |Contagem de réplicas padrão|**1**|
   |Contagem de réplicas de região de destino|**1**|
   |Tipo de conta de armazenamento|**LRS do SSD Premium**|

1. Na guia **Revisar + criar** da folha **Criar uma imagem** , clique em **Criar**.

   > **Observação**: aguarde até que a implantação seja concluída. Isso pode levar cerca de 20 minutos.

1. No seu computador de laboratório, no navegador da Web que exibe o portal do Azure, pesquise e selecione **Galerias de computação do Azure** e, na folha **Galerias de computação do Azure**, selecione a entrada **az14025imagegallery** e, na folha ****az14025imagegallery****, verifique a presença da entrada **az140-25-host-image**que representa a imagem recém-criada.

#### Tarefa 5: Provisionar um pool de hosts da Área de Trabalho Virtual do Azure usando uma imagem personalizada

1. No computador do laboratório, no portal do Azure, use a caixa de texto **Pesquisar recursos, serviços e documentos** na parte superior da página do portal do Azure para pesquisar e navegar até **Redes virtuais** e, na folha **Redes virtuais**, selecione **az140-adds-vnet11**. 
1. Na folha **az140-adds-vnet11**, selecione **Sub-redes**, na folha **Sub-redes**, selecione **+ Sub-rede**, na folha **Adicionar sub-rede**, especifique as seguintes configurações (deixe todas as outras configurações com seus valores padrão) e clique em **Salvar**:

   |Configuração|Valor|
   |---|---|
   |Nome|**hp4-Subnet**|
   |Intervalo de endereços da sub-rede|**10.0.4.0/24**|

1. No computador do laboratório, no portal do Azure, na janela do navegador da Web que exibe o portal do Azure, pesquise e selecione **Área de Trabalho Virtual do Azure**. Na folha **Área de Trabalho Virtual do Azure**, selecione **Pools de Host** e, na folha **Pools de Host \|da Área de Trabalho Virtual do Azure**, selecione **+ Criar**. 
1. Na guia **Noções básicas** da folha **Criar um pool de hosts**, especifique as seguintes configurações e selecione **Avançar: Máquinas Virtuais >**:

   |Configuração|Valor|
   |---|---|
   |Assinatura|o nome da assinatura do Azure que você está usando neste laboratório|
   |Grupo de recursos|**az140-25-RG**|
   |Nome do pool de host|**az140-25-hp4**|
   |Localidade|o nome da região do Azure na qual você implantou recursos no primeiro exercício desse laboratório|
   |Ambiente de validação|**Não**|
   |Tipo de pool de host|**Em pool**|
   |Limite máximo da sessão|**50**|
   |Algoritmo de balanceamento de carga|**Amplitude**|

1. Na guia **Máquinas Virtuais** da folha **Criar um pool de hosts**, especifique as seguintes configurações:

   |Configuração|Valor|
   |---|---|
   |Adicionar máquinas virtuais do Azure|**Sim**|
   |Grupo de recursos|**O padrão é o mesmo que o do pool de hosts**|
   |Prefixo do nome|**az140-25-p4**|
   |Localização da máquina virtual|o nome da região do Azure na qual você implantou recursos no primeiro exercício desse laboratório|
   |Opções de disponibilidade|**Nenhuma redundância de infraestrutura necessária**|
   
1. Na guia **Máquinas virtuais** da folha **Criar um pool de host**, diretamente abaixo da lista suspensa **Imagem**, clique no link **Ver todas as imagens**.
1. Na folha **Selecionar uma imagem**, em **Outros Itens**, clique em **Imagens Compartilhadas** e, na lista de imagens compartilhadas, selecione **az140-25-host-image**. 
1. Na guia **Máquinas Virtuais** da folha **Criar um pool de hosts**, especifique as seguintes configurações e selecione **Avançar: Workspace >**

   |Configuração|Valor|
   |---|---|
   |Tamanho da máquina virtual|**Standard D2s v3**|
   |Número de VMs|**1**|
   |Tipo de disco de SO|**SSD Standard**|
   |Rede virtual|**az140-adds-vnet11**|
   |Sub-rede|**hp4-Subnet (10.0.4.0/24)**|
   |Grupo de segurança de rede|**Basic**|
   |Portas de entrada públicas|**Sim**|
   |Portas de entrada a serem permitidas|**RDP**|
   |UPN de ingresso no domínio do AD|**student@adatum.com**|
   |Senha|**Pa55w.rd1234**|
   |Especificar o domínio ou a unidade|**Sim**|
   |Domínio a ingressar|**adatum.com**|
   |Caminho da Unidade Organizacional|**OU=WVDInfra,DC=adatum,DC=com**|
   |Nome de usuário|Aluno|
   |Senha|Pa55w.rd1234|
   |Confirmar senha|Pa55w.rd1234|

1. Na guia **Workspace** da folha **Criar um pool** de hosts, especifique as seguintes configurações e selecione **Examinar + criar**:

   |Configuração|Valor|
   |---|---|
   |Registre o grupo de aplicativos da área de trabalho|**Não**|

1. Na guia **Revisar + criar** da folha **Criar um pool de host**, selecione **Criar**.

   > **Observação**: aguarde até que a implantação seja concluída. Isso pode levar cerca de 10 minutos.
   > 
   > **Observação** se a implantação falhar devido ao limite de cota atingido, execute as etapas descritas no primeiro laboratório para solicitar automaticamente o aumento da cota do limite D2sv3 Padrão para 30.

   > **Observação**: Após a implantação de hosts com base em imagens personalizadas, você deve considerar a execução da Ferramenta de Otimização da Área de Trabalho Virtual, disponível em [seu repositório GitHub](https://github.com/The-Virtual-Desktop-Team/).


### Exercício 2: Parar e desalocar VMs do Azure provisionadas no laboratório

As principais tarefas desse exercício são as seguintes:

1. Parar e desalocar VMs do Azure provisionadas no laboratório

>**Observação**: Nesse exercício, você desalocará as VMs do Azure provisionadas neste laboratório para minimizar os encargos de computação correspondentes

#### Tarefa 1: Desalocar VMs do Azure provisionadas no laboratório

1. Alterne para o computador de laboratório e, na janela do navegador da Web que exibe o portal do Azure, abra a sessão do Shell do **PowerShell** no painel do **Cloud Shell**.
1. Na sessão do PowerShell no painel do Cloud Shell, execute o seguinte para listar todas as VMs do Azure criadas neste laboratório:

   ```powershell
   Get-AzVM -ResourceGroup 'az140-25-RG'
   ```

1. Na sessão do PowerShell no painel do Cloud Shell, execute o seguinte para parar e desalocar todas as VMs do Azure criadas neste laboratório:

   ```powershell
   Get-AzVM -ResourceGroup 'az140-25-RG' | Stop-AzVM -NoWait -Force
   ```

   >**Observação**: O comando é executado de modo assíncrono (conforme determinado pelo parâmetro -NoWait), portanto, embora você possa executar outro comando do PowerShell imediatamente depois na mesma sessão do PowerShell, levará alguns minutos antes de o grupo de recursos ser de fato removido.
