---
lab:
  title: 'Laboratório: Empacotar aplicativos da Área de Trabalho Virtual do Azure (AD DS)'
  module: 'Module 4: Manage User Environments and Apps'
---

# Laboratório – Empacotar aplicativos da Área de Trabalho Virtual do Azure (AD DS)
# Manual de laboratório do aluno

## Dependências do laboratório

- Uma assinatura do Azure
- Uma conta Microsoft ou uma conta do Microsoft Entra com a função de administrador global no locatário do Microsoft Entra associado à assinatura do Azure e com a função de proprietário ou colaborador na assinatura
- O laboratório completo **Preparar para a implantação da Área de Trabalho Virtual do Azure (AD DS)**
- O laboratório completo **Gerenciamento de perfil da Área de Trabalho Virtual do Azure (AD DS)**
- O laboratório completo **Configurar políticas de acesso condicional para WVD (AD DS)**

## Tempo estimado

60 minutos

## Cenário do laboratório

Você precisa empacotar e implantar aplicativos da Área de Trabalho Virtual do Azure em um ambiente do AD DS (Active Directory Domain Services).

## Objetivos
  
Depois de realizar este laboratório, você será capaz de:

- Preparar e criar pacotes de aplicativo MSIX
- Implementar uma imagem de anexação de aplicativo MSIX para a Área de Trabalho Virtual do Azure em um ambiente do Microsoft Entra DS
- Implementar a anexação de aplicativo MSIX na Área de Trabalho Virtual do Azure no ambiente do AD DS

## Arquivos de laboratório

-  \\\\AZ-140\\AllFiles\\Labs\\04\\az140-42_azuredeploycl42.json
-  \\\\AZ-140\\AllFiles\\Labs\\04\\az140-42_azuredeploycl42.parameters.json

## Instruções

### Exercício 1: Preparar e criar pacotes de aplicativo MSIX

As principais tarefas deste exercício são as seguintes:

1. Preparar para a configuração de hosts de sessão da Área de Trabalho Virtual do Azure
1. Implantar uma VM do Azure executando o Windows 10 usando um modelo de Início Rápido do Azure Resource Manager
1. Preparar a VM do Azure que executa o Windows 10 para empacotamento MSIX
1. Gerar um certificado de autenticação
1. Baixar software para empacotar
1. Instalar a Ferramenta de Empacotamento MSIX
1. Criar um pacote MSIX

#### Tarefa 1: Preparar para a configuração de hosts de sessão da Área de Trabalho Virtual do Azure

1. No seu computador de laboratório, inicie um navegador da Web, navegue até o [portal do Azure](https://portal.azure.com) e entre fornecendo credenciais de uma conta de usuário com a função Proprietário na assinatura que você usará neste laboratório.
1. No computador do laboratório e, na janela do navegador da Web que exibe o portal do Azure, abra a sessão shell do **PowerShell** no painel do **Cloud Shell**.
1. Na sessão do PowerShell no painel do Cloud Shell, execute o seguinte para iniciar o host da sessão das VMs do Azure da Área de Trabalho Virtual do Azure que você usará neste laboratório:

   ```powershell
   Get-AzVM -ResourceGroup 'az140-21-RG' | Start-AzVM -NoWait
   ```

   >**Observação**: O comando é executado de modo assíncrono (conforme determinado pelo parâmetro -NoWait), portanto, embora você possa executar outro comando do PowerShell imediatamente depois na mesma sessão do PowerShell, levará alguns minutos antes das VMs do Azure serem realmente iniciadas. 

   >**Observação**: Se você habilitou o PSRemoting nos hosts da sessão no grupo de recursos az140-21-RG na primeira tarefa do laboratório anterior (Implementar e gerenciar perfis AVD), poderá prosseguir diretamente para a próxima tarefa sem esperar que as VMs do Azure sejam iniciadas. Se você não habilitou anteriormente o PSRemoting nos hosts de sessão no grupo de recursos az140-21-RG, aguarde até que as VMs sejam iniciadas e execute o comando a seguir.

1. Na sessão do PowerShell do **Cloud Shell**, execute o seguinte para habilitar a Comunicação Remota do PowerShell nos hosts da sessão.

   ```powershell
   Get-AzVM -ResourceGroup 'az140-21-RG' | Enable-AzVMPSRemoting
   ```
   
#### Tarefa 2: Implantar uma VM do Azure executando o Windows 10 usando um modelo de Início Rápido do Azure Resource Manager

1. No computador do laboratório, na janela do navegador da Web que exibe o portal do Azure, na barra de ferramentas do painel do Cloud Shell, selecione o ícone **Upload/Download de arquivos**, no menu suspenso, selecione **Upload**e carregue os arquivos **\\\\AZ-140\\AllFiles\\Labs\\04\\az140-42_azuredeploycl42.json** e **\\\\AZ-140\\AllFiles\\Labs\\04\\az140-42_azuredeploycl42.parameters.json** no diretório inicial do Cloud Shell.
1. Na sessão do PowerShell no painel do Cloud Shell, execute o seguinte para implantar uma VM do Azure executando o Windows 10 que você usará para criar pacotes MSIX e ingressá-la no domínio do Microsoft Entra DS:

   ```powershell
   $vNetResourceGroupName = 'az140-11-RG'
   $location = (Get-AzResourceGroup -ResourceGroupName $vNetResourceGroupName).Location
   $resourceGroupName = 'az140-42-RG'
   New-AzResourceGroup -ResourceGroupName $resourceGroupName -Location $location
   New-AzResourceGroupDeployment `
     -ResourceGroupName $resourceGroupName `
     -Location $location `
     -Name az140lab0402vmDeployment `
     -TemplateFile $HOME/az140-42_azuredeploycl42.json `
     -TemplateParameterFile $HOME/az140-42_azuredeploycl42.parameters.json
   ```

   > **Observação**: Aguarde a conclusão da implantação antes de prosseguir para a próxima tarefa. Isso pode levar cerca de 10 minutos. 

#### Tarefa 3: Preparar a VM do Azure que executa o Windows 10 para empacotamento MSIX

1. No seu computador de laboratório, no portal do Azure, pesquise e selecione **Máquinas virtuais** e, na folha **Máquinas virtuais**, na lista de máquinas virtuais, selecione a entrada **az140-cl-vm42**. Isso abrirá a folha **az140-cl-vm42**.
1. Na folha **az140-cl-vm42**, selecione **Conectar**, no menu suspenso, selecione **Bastion**, na guia **Bastion** da folha **az140-cl-vm42\| Conectar** selecione **Usar Bastion**.
1. Quando solicitado, entre com o **wvdadmin1@adatum.com** nome de usuário e a senha definida ao criar essa conta de usuário. 
1. Na sessão do Bastion para **az140-cl-vm42**, inicie o **ISE do Windows PowerShell** como administrador, no **ISE do Windows PowerShell **do Console do Administrador, execute o seguinte para preparar o sistema operacional para o empacotamento MSIX:

   ```powershell
   Schtasks /Change /Tn "\Microsoft\Windows\WindowsUpdate\Scheduled Start" /Disable
   reg add HKLM\Software\Policies\Microsoft\WindowsStore /v AutoDownload /t REG_DWORD /d 0 /f
   reg add HKCU\Software\Microsoft\Windows\CurrentVersion\ContentDeliveryManager /v PreInstalledAppsEnabled /t REG_DWORD /d 0 /f
   reg add HKLM\SOFTWARE\Microsoft\Windows\CurrentVersion\ContentDeliveryManager\Debug /v ContentDeliveryAllowedOverride /t REG_DWORD /d 0x2 /f
   reg add HKLM\Software\Microsoft\RDInfraAgent\MSIXAppAttach /v PackageListCheckIntervalMinutes /t REG_DWORD /d 1 /f
   reg add HKLM\SOFTWARE\Microsoft\Windows\CurrentVersion\Policies\System /v EnableLUA /t REG_DWORD /d 0 /f
   ```

   > **Observação**: A última dessas alterações do Registro desabilita o Controle de Acesso do Usuário. Isso tecnicamente não é necessário, mas simplifica o processo ilustrado neste laboratório.

#### Tarefa 4: Gerar um certificado de assinatura

> **Observação**: Neste laboratório, você usará um certificado autoassinado. Em um ambiente de produção, você deve usar um certificado emitido por uma Autoridade de Certificação pública ou interna, dependendo do uso pretendido.

1. Na sessão do Bastion para **az140-cl-vm42**, no **: ISE do Windows PowerShell do Console do Administrador**, execute o seguinte para gerar um certificado autoassinado com o atributo Common Name definido como **Adatum** e armazene o certificado na pasta **Pessoal** do repositório de certificados do **Computador Local**:

   ```powershell
   New-SelfSignedCertificate -Type Custom -Subject "CN=Adatum" -KeyUsage DigitalSignature -KeyAlgorithm RSA -KeyLength 2048 -CertStoreLocation "cert:\LocalMachine\My"
   ```

1. No **ISE do Windows Power Shell: do Console do Administrador**, execute o seguinte para iniciar o console de **Certificados** direcionado ao repositório de certificados do Computador Local:

   ```powershell
   certlm.msc
   ```

1. No painel do console **Certificados**, expanda a pasta **Pessoal**, selecione a subpasta **Certificados**, clique com o botão direito do mouse no certificado **Adatum**, no menu de clique com o botão direito do mouse, selecione **Todas as Tarefas** seguidas por **Exportar**. Isso iniciará o Assistente de **Exportação de Certificados**. 
1. Na página **Bem-vindo ao Assistente de Exportação de Certificados** do **Assistente de Exportação de Certificados**, selecione **Avançar**.
1. Na página **Exportar Chave Privada** do **Assistente de Exportação de Certificados**, selecione a opção **Sim, exporte a opção de chave privada** e selecione **Avançar**.
1. Na página **Exportar Formato de Arquivo** do **Assistente de Exportação de Certificados**, marque a caixa de seleção **Exportar todas as propriedades estendidas**, desmarque a caixa de seleção **Habilitar privacidade do certificado** e selecione **Avançar**.
1. Na página **Segurança** do **Assistente de Exportação de Certificados**, marque a caixa de seleção **Senha**, nas caixas de texto abaixo, digite **Pa55w.rd1234** e selecione **Avançar**.
1. Na página **Arquivo para Exportar** do **Assistente de Exportação de Certificados**, na caixa de texto **Nome do arquivo**, selecione **Procurar**, na caixa de diálogo **Salvar como**, navegue até a pasta **C:\\Allfiles\\Labs\\04** (crie a pasta primeiro), na caixa de texto **Nome do arquivo**, digite **adatum.pfx** e selecione **Salvar**.
1. De volta à página **Arquivo para Exportação** do **Assistente de Exportação de Certificados**, verifique se a caixa de texto contém a entrada **C:\\Allfiles\\Labs\\04\\adatum.pfx** e selecione **Avançar**.
1. Na página **Assistente para Conclusão da Exportação de Certificados** do **Assistente de Exportação de Certificados**, selecione **Concluir** e selecione **OK** para reconhecer a exportação bem-sucedida. 

   > **Observação**: Como você está usando um certificado autoassinado, é necessário instalá-lo no repositório de certificados **Pessoas Confiáveis** nos hosts da sessão de destino.

1. No **ISE do Windows Power Shell: do Console do Administrador**, execute o seguinte para instalar o certificado recém-gerado no repositório de certificados **Pessoas Confiáveis** nos hosts da sessão de destino:

   ```powershell
   $wvdhosts = 'az140-21-p1-0','az140-21-p1-1','az140-21-p1-2'
   $cleartextPassword = 'Pa55w.rd1234'
   $securePassword = ConvertTo-SecureString $cleartextPassword -AsPlainText -Force
   $localPath = 'C:\Allfiles\Labs\04'
   ForEach ($wvdhost in $wvdhosts){
      $remotePath = "\\$wvdhost\C$\Allfiles\Labs\04\"
      New-Item -ItemType Directory -Path $remotePath -Force
      Copy-Item -Path "$localPath\adatum.pfx" -Destination $remotePath -Force
      Invoke-Command -ComputerName $wvdhost -ScriptBlock {
         Import-PFXCertificate -CertStoreLocation Cert:\LocalMachine\TrustedPeople -FilePath 'C:\Allfiles\Labs\04\adatum.pfx' -Password $using:securePassword
      } 
   }
   ```

#### Tarefa 5: Baixar software para empacotar

1. Na sessão do Bastion para **az140-cl-vm42**, inicie o **Microsoft Edge** e navegue até **https://github.com/microsoft/XmlNotepad**.
1. Na página **microsoft/XmlNotepad** **readme.md**, selecione o link de download do instalador para download autônomo e baixe os arquivos de instalação compactados.
1. Na sessão do Bastion para **az140-cl-vm42**, inicie o Explorador de Arquivos, navegue até a pasta **Downloads**, abra o arquivo compactado, copie o conteúdo de dentro da pasta no arquivo compactado e cole-o no diretório **C:\\AllFiles\\Labs\\04\\**. 

#### Tarefa 6: Instalar a Ferramenta de Empacotamento MSIX

1. Na sessão do Bastion para **az140-cl-vm42**, inicie o aplicativo da **Microsoft Store**.
1. No aplicativo da **Microsoft Store**, pesquise e selecione a **Ferramenta de Empacotamento MSIX**, na **página Ferramenta** de Empacotamento MSIX, selecione **Obter**.
1. Quando solicitado, ignore a entrada, aguarde a conclusão da instalação, selecione **Abrir** e, na caixa de diálogo **Enviar dados de diagnóstico**, selecione **Recusar**, 

#### Tarefa 7: Criar um pacote MSIX

1. Na sessão do Bastion para **az140-cl-vm42**, alterne para a ** janela do ISE do Windows Power Shell no Administrador** e, do **ISE do Windows PowerShell do painel de script do Administrador**, execute o seguinte para desabilitar o serviço Pesquisa do Windows:

   ```powershell
   $serviceName = 'wsearch'
   Set-Service -Name $serviceName -StartupType Disabled
   Stop-Service -Name $serviceName
   ```

1. Na sessão do Bastion para **az140-cl-vm42**, no **: ISE do Windows PowerShell do Console do Administrador**, execute o seguinte para criar a pasta que hospedará o pacote MSIX:

   ```powershell
   New-Item -ItemType Directory -Path 'C:\AllFiles\Labs\04\XmlNotepad' -Force
   ```

1. Na sessão do Bastion para **az140-cl-vm42**, no **: ISE do Windows PowerShell do Console do Administrador**, execute o seguinte para remover o fluxo de dados alternativo Zone.Identifier dos arquivos do instalador extraído, que tem um valor de "3" para indicar que eles foram baixados da Internet:

   ```powershell
   Get-ChildItem -Path 'C:\AllFiles\Labs\04' -Recurse -File | Unblock-File
   ```

1. Na sessão do Bastion para **az140-cl-vm42**, alterne para a interface da **Ferramenta de Empacotamento MSIX**, na página **Selecionar tarefa**, selecione **Pacote de aplicativos - Criar a entrada do pacote do aplicativo**. Isso iniciará o assistente **Criar novo pacote**.
1. Na página **Selecionar ambiente** do assistente **Criar novo pacote**, verifique se a opção **Criar pacote neste computador** está selecionada, selecione **Avançar** e aguarde a instalação do **Driver da Ferramenta de Empacotamento MSIX**.
1. Na página **Preparar computador** do assistente **Criar novo pacote**, examine as recomendações. Se houver uma reinicialização pendente, reinicie o sistema operacional, entre novamente usando a **wvdadmin1@adatum.com** conta e reinicie a **Ferramenta de Empacotamento MSIX** antes de continuar. 

   >**Observação**: A Ferramenta de Empacotamento MSIX desabilita temporariamente o Windows Update e o Windows Search. Nesse caso, o serviço Pesquisa do Windows já está desabilitado. 

1. Na página **Preparar computador** do assistente **Criar novo pacote**, clique em **Avançar**.
1. Na página **Selecionar instalador** do assistente **Criar novo pacote**, ao lado do assistente **Escolher o instalador que você deseja empacotar**, selecione **Procurar**, na caixa de diálogo **Abrir**, navegue até a pasta **C:\\ AllFiles\\Labs\\04**, selecione **XmlNotepadSetup.msi** e clique em **Abrir**, 
1. Na página **Selecionar instalador** do assistente **Criar novo pacote**, na lista suspensa **Preferência de assinatura**, selecione a entrada **Assinar com um certificado (.pfx)**, ao lado da caixa de texto **Procurar certificado**, selecione **Procurar**, na caixa de diálogo **Abrir**, navegue até a pasta **C:\\AllFiles\\Labs\\04**, selecione o arquivo **adatum.pfx**, clique em **Abrir**, na caixa de texto **Senha**, digite **Pa55w.rd1234** e selecione **Avançar**.
1. Na página **Informações do pacote** do assistente **Criar novo pacote**, examine as informações do pacote, valide se o nome do editor está definido como **CN=Adatum** e selecione **Avançar**. Isso disparará a instalação do software baixado.
1. Na janela **Instalação do XMLNotepad**, aceite os termos no Contrato de Licença e selecione **Instalar** e, depois que a instalação for concluída, marque a caixa de seleção **Iniciar Bloco de Notas XML** e selecione **Concluir**.
1. Quando solicitado, na janela **Análise do bloco de notas XML**, selecione **Não**, verifique se o Bloco de Notas XML está em execução, feche-o, volte para o assistente **Criar novo pacote** na janela **Ferramenta de Empacotamento MSIX** e selecione **Avançar**.

   > **Observação**: Nesse caso, a reinicialização não é necessária para concluir a instalação.

1. Na página **Primeiras tarefas de inicialização ** do assistente **Criar novo pacote**, examine as informações fornecidas e selecione **Avançar**.
1. Quando solicitado **Você terminou?**, selecione **Sim, seguir em frente**.
1. Na página **Relatório de serviços** do assistente **Criar novo pacote**, verifique se nenhum serviço está listado e selecione **Avançar**.
1. Na página **Criar pacote** do assistente **Criar novo pacote**, na caixa de texto **Salvar local**, digite **C:\\Allfiles\\Labs\\04\\XmlNotepad\XmlNotepad.msix** e clique em **Criar**.
1. Na caixa de diálogo **Pacote criado com sucesso**, observe o local do pacote salvo e selecione **Fechar**.
1. Alterne para a janela do Explorador de Arquivos, navegue até a pasta **C:\\Allfiles\\Labs\\04\\XmlNotepad** e verifique se ela contém os arquivos *.msix e *.xml.
1. Copie o arquivo **XmlNotepad.msix** para a pasta **C:\\Allfiles\\Labs\\04**.


### Exercício 2: Implementar uma imagem de anexação de aplicativo MSIX para a Área de Trabalho Virtual do Azure no ambiente do Microsoft Entra DS

As principais tarefas deste exercício são as seguintes:

1. Habilitar o Hyper-V nas VMs do Azure que executam o Windows 10 Enterprise Edition
1. Criar uma imagem de anexação de aplicativo MSIX

#### Tarefa 1: Habilitar o Hyper-V nas VMs do Azure que executam o Windows 10 Enterprise Edition

1. Na sessão do Bastion para **az140-cl-vm42**, no ** ISE do Windows PowerShell do Console do Administrador**, execute o seguinte para preparar os hosts da Área de Trabalho Virtual do Azure de destino para anexação de aplicativo MSIX: 

   ```powershell
   $wvdhosts = 'az140-21-p1-0','az140-21-p1-1','az140-21-p1-2'
   ForEach ($wvdhost in $wvdhosts){
      Invoke-Command -ComputerName $wvdhost -ScriptBlock {
         Schtasks /Change /Tn "\Microsoft\Windows\WindowsUpdate\Scheduled Start" /Disable
         reg add HKLM\Software\Policies\Microsoft\WindowsStore /v AutoDownload /t REG_DWORD /d 0 /f
         reg add HKCU\Software\Microsoft\Windows\CurrentVersion\ContentDeliveryManager /v PreInstalledAppsEnabled /t REG_DWORD /d 0 /f
         reg add HKLM\SOFTWARE\Microsoft\Windows\CurrentVersion\ContentDeliveryManager\Debug /v ContentDeliveryAllowedOverride /t REG_DWORD /d 0x2 /f
         reg add HKLM\SOFTWARE\Microsoft\Windows\CurrentVersion\Policies\System /v EnableLUA /t REG_DWORD /d 0 /f
         reg add HKLM\Software\Microsoft\RDInfraAgent\MSIXAppAttach /v PackageListCheckIntervalMinutes /t REG_DWORD /d 1 /f
         Set-Service -Name wuauserv -StartupType Disabled
      }
   }
   ```

1. Na sessão do Bastion para **az140-cl-vm42**, no ** ISE do Windows PowerShell do Console do Administrador**, execute o seguinte para instalar o Hyper-V e suas ferramentas de gerenciamento, incluindo o módulo Hyper-V PowerShell nos hosts da Área de Trabalho Virtual do Azure:

   ```powershell
   $wvdhosts = 'az140-21-p1-0','az140-21-p1-1','az140-21-p1-2'
   ForEach ($wvdhost in $wvdhosts){
      Invoke-Command -ComputerName $wvdhost -ScriptBlock {
         Enable-WindowsOptionalFeature -Online -FeatureName Microsoft-Hyper-V-All
      }
   }
   ```

1. Quando solicitado a reiniciar o sistema operacional de destino, selecione **Sim**.
1. Na sessão do Bastion para **az140-cl-vm42**, no ** ISE do Windows PowerShell do Console do Administrador**, execute o seguinte para instalar o Hyper-V e suas ferramentas de gerenciamento, incluindo o módulo Hyper-V PowerShell no computador local:

   ```powershell
   Enable-WindowsOptionalFeature -Online -FeatureName Microsoft-Hyper-V-All
   ```

1. Depois que a instalação dos componentes do Hyper-V for concluída, selecione **Sim** para reiniciar o sistema operacional. Após a reinicialização, entre novamente com o **wvdadmin1@adatum.com** nome de usuário e a senha definida ao criar essa conta de usuário.

#### Tarefa 2: Criar uma imagem de anexação de aplicativo MSIX

1. Na sessão do Bastion para **az140-cl-vm42**, inicie o **Microsoft Edge** e navegue até **https://aka.ms/msixmgr**. Isso baixará automaticamente o arquivo **msixmgr.zip** (o arquivo da ferramenta MSIX mgr) na pasta **Downloads**.
1. No Explorador de Arquivos, navegue até a pasta **Downloads**, abra o arquivo compactado e copie o conteúdo da pasta **x64** (incluindo a pasta) para a pasta **C:\\AllFiles\\Labs\\04**. 
1. Na sessão do Bastion para **az140-cl-vm42**, inicie o **ISE do Windows PowerShell** como administrador, no **ISE do Windows PowerShell no Painel de script do Administrador**, execute o seguinte para criar o arquivo VHD que servirá como a imagem de anexação do aplicativo MSIX:

   ```powershell
   New-Item -ItemType Directory -Path 'C:\Allfiles\Labs\04\MSIXVhds' -Force
   New-VHD -SizeBytes 128MB -Path 'C:\Allfiles\Labs\04\MSIXVhds\XmlNotepad.vhd' -Dynamic -Confirm:$false
   ```

1. No **ISE do Windows Power Shell: do Painel de script do Administrador**, execute o seguinte para montar o arquivo VHD recém-criado:

   ```powershell
   $vhdObject = Mount-VHD -Path 'C:\Allfiles\Labs\04\MSIXVhds\XmlNotepad.vhd' -Passthru
   ```

1. No **ISE do Windows Power Shell: do Painel de script do Administrador**, execute o seguinte para inicializar o disco, criar uma nova partição, formatar e atribuir a ele a primeira letra da unidade disponível:

   ```powershell
   $disk = Initialize-Disk -Passthru -Number $vhdObject.Number
   $partition = New-Partition -AssignDriveLetter -UseMaximumSize -DiskNumber $disk.Number
   Format-Volume -FileSystem NTFS -Confirm:$false -DriveLetter $partition.DriveLetter -Force
   ```

   > **Observação**: Se houver uma janela pop-up solicitando que você formate a unidade F: selecione **Cancelar**.

1. No **ISE do Windows Power Shell: do Painel de script do Administrador**, execute o seguinte para criar uma estrutura de pastas que hospedará os arquivos MSIX e desempacotará nele o pacote MSIX criado na tarefa anterior:

   ```powershell
   $appName = 'XmlNotepad'
   New-Item -ItemType Directory -Path "$($partition.DriveLetter):\Apps" -Force
   Set-Location -Path 'C:\AllFiles\Labs\04\x64'
   .\msixmgr.exe -Unpack -packagePath ..\$appName.msix -destination "$($partition.DriveLetter):\Apps" -applyacls
   ```

1. Na sessão do Bastion para **az140-cl-vm42**, no Explorador de Arquivos, navegue até a pasta **F:\\Apps** e examine seu conteúdo. Se solicitado a obter acesso à pasta, selecione **Continuar**.
1. Na sessão do Bastion para **az140-cl-vm42**, no ** ISE do Windows PowerShell no Console do Administrador**, execute o seguinte para desmontar o arquivo VHD que servirá como a imagem MSIX:

   ```powershell
   Dismount-VHD -Path "C:\Allfiles\Labs\04\MSIXVhds\$appName.vhd" -Confirm:$false
   ```

### Exercício 3: Implementar a anexação de aplicativo MSIX em hosts da sessão da Área de Trabalho Virtual do Azure

As principais tarefas deste exercício são as seguintes:

1. Configurar grupos do Active Directory que contêm hosts da Área de Trabalho Virtual do Azure
1. Configurar o compartilhamento de Arquivos do Azure para anexação de aplicativo MSIX
1. Montar e registrar a imagem de anexação do aplicativo MSIX em hosts de sessão da Área de Trabalho Virtual do Azure
1. Publicar os aplicativos MSIX num grupo de aplicativos
1. Validar a funcionalidade da anexação do aplicativo MSIX

#### Tarefa 1: Configurar grupos do Active Directory que contêm hosts da Área de Trabalho Virtual do Azure

1. Alterne para o computador de laboratório, no navegador da Web que exibe o portal do Azure, pesquise e selecione **Máquinas virtuais** e, na folha **Máquinas virtuais**, selecione **az140-dc-vm11**.
1. Na folha **az140-dc-vm11**, selecione **Conectar**, no menu suspenso, selecione **Bastion**, na guia **Bastion** da folha **az140-dc-vm11\| Conectar**, selecione **Usar Bastion**.
1. Quando solicitado, forneça as seguintes credenciais e selecione **Conectar**:

   |Configuração|Valor|
   |---|---|
   |Nome do usuário|**Aluno**|
   |Senha|**Pa55w.rd1234**|

1. Na sessão do Bastion para **az140-dc-vm11**, inicie o **ISE do Windows PowerShell** como administrador.
1. No **ISE do Windows Power Shell: no Painel de script do Administrador**, execute o seguinte para criar um objeto de grupo do AD DS que será sincronizado com o locatário do Microsoft Entra usado neste laboratório:

   ```powershell
   $ouPath = "OU=WVDInfra,DC=adatum,DC=com"
   New-ADGroup -Name 'az140-hosts-42-p1' -GroupScope 'Global' -GroupCategory Security -Path $ouPath
   ```

   > **Observação**: Você usará esse grupo para conceder permissões de hosts da Área de Trabalho Virtual do Azure ao compartilhamento de arquivos **az140-42-msixvhds**.

1. No **ISE do Windows Power Shell: do Console do Administrador** execute o seguinte para adicionar membros aos grupos criados na etapa anterior:

   ```powershell
   Get-ADGroup -Identity 'az140-hosts-42-p1' | Add-AdGroupMember -Members 'az140-21-p1-0$','az140-21-p1-1$','az140-21-p1-2$'
   ```

1. No **ISE do Windows Power Shell: no Painel de script do Administrador**, execute o seguinte para reiniciar os servidores que são membros do grupo 'az140-hosts-42-p1':

   ```powershell
   $hosts = (Get-ADGroup -Identity 'az140-hosts-42-p1' | Get-ADGroupMember | Select-Object Name).Name
   $hosts | ForEach-Object {Restart-Computer -ComputerName $_ -Force}
   ```

   > **Observação**: Esta etapa garante que a alteração da associação de grupo entre em vigor. 

1. Na sessão do Bastion para **az140-dc-vm11**, no menu **Iniciar**, expanda a pasta do **Microsoft Entra Connect** e selecione **Microsoft Entra Connect**.
1. Na página **Boas-vindas ao Microsoft Entra Connect** da janela do **Microsoft Entra Connect**, selecione **Configurar**.
1. Na página **Tarefas adicionais** na janela do **Microsoft Entra Connect**, selecione **Personalizar opções de sincronização** e selecione **Avançar**.
1. Na página **Conectar-se ao Microsoft Entra** na janela do **Microsoft Entra Connect**, autentique-se usando o nome principal do usuário da conta de usuário **aadsyncuser** que você identificou anteriormente nesta tarefa com a senha definida ao criar essa conta de usuário.
1. Na página **Conectar seus diretórios** na janela do **Microsoft Entra Connect**, selecione **Avançar**.
1. Na página **Filtragem de domínio e UO** na janela do **Microsoft Entra Connect**, verifique se a opção **Sincronizar domínios e UOs selecionadas** está selecionada, expanda o nó **adatum.com**, marque a caixa de seleção ao lado da UO **WVDInfra** (deixe todas as outras caixas de seleção selecionadas inalteradas) e selecione **Avançar**.
1. Na página **Recursos opcionais** na janela do **Microsoft Entra Connect**, aceite as configurações padrão e selecione **Avançar**.
1. Na página **Pronto para configurar** na janela do **Microsoft Entra Connect**, verifique se a caixa de seleção **Iniciar o processo de sincronização quando a configuração for concluída** está selecionada e selecione **Configurar**.
1. Examine as informações na página **Configuração concluída** e selecione **Sair** para fechar a janela do **Microsoft Entra Connect**.
1. Na sessão do Bastion para **az140-dc-vm11**, inicie o Microsoft Edge e navegue até o [portal do Azure](https://portal.azure.com). Quando solicitado, entre usando as credenciais do Microsoft Entra da conta de usuário com a função de Administrador Global no locatário do Microsoft Entra associado à assinatura do Azure que você está usando nesse laboratório.
1. Na sessão do Bastion para **az140-dc-vm11**, na janela do Microsoft Edge exibindo o portal do Azure, pesquise e selecione o **Azure Active Directory** para navegar até o locatário do Microsoft Entra associado à assinatura do Azure que você está usando para este laboratório.
1. Na folha do Azure Active Directory, na barra de menus vertical no lado esquerdo, na seção **Gerenciar**, clique em **Grupos**. 
1. Na folha **Grupos | Todos os grupos**, na lista de grupos, selecione a entrada **az140-hosts-42-p1**.

   > **Observação**: Talvez seja necessário atualizar a página para que o grupo seja exibido.

1. Na folha **az140-hosts-42-p1**, na barra de menus vertical no lado esquerdo, na seção **Gerenciar**, clique em **Membros**.
1. Na folha **az140-hosts-42-p1 | Membros**, verifique se a lista de **Membros diretos** inclui os três hosts do pool de Área de Trabalho Virtual do Azure que você adicionou ao grupo anteriormente nesta tarefa.

#### Tarefa 2: Configurar o compartilhamento de Arquivos do Azure para anexação de aplicativo MSIX

1. No computador do laboratório, volte para a sessão do Bastion para **az140-cl-vm42**.
1. Na sessão do Bastion para **az140-cl-vm42**, inicie o Microsoft Edge no modo InPrivate, navegue até o [portal do Azure](https://portal.azure.com) e entre fornecendo credenciais de uma conta de usuário com a função Proprietário na assinatura que você usará neste laboratório.

   > **Observação**: Certifique-se de usar o modo InPrivate do Microsoft Edge.

1. Na sessão do Bastion para **az140-cl-vm42**, na janela do Microsoft Edge exibindo o portal do Azure, pesquise e selecione **Contas de armazenamento** e, na folha **Contas de Armazenamento**, selecione a conta de armazenamento configurada para hospedar perfis de usuário.

   > **Observação**: Essa parte do laboratório depende da conclusão do laboratório **Gerenciamento de perfil da Área de Trabalho Virtual do Azure (AD DS)** ou **Gerenciamento de perfil da Área de Trabalho Virtual do Azure (Microsoft Entra DS)**

   > **Observação**: Em cenários de produção, você deve considerar o uso de uma conta de armazenamento separada. Isso exigiria configurar essa conta de armazenamento para a autenticação do Microsoft Entra DS, que você já implementou para a conta de armazenamento que hospeda perfis de usuário. Você está usando a mesma conta de armazenamento para minimizar as etapas duplicadas em laboratórios individuais.

1. Na folha da conta de armazenamento, no menu vertical no lado esquerdo, selecione **Controle de Acesso (IAM)**.
1. Na folha **Controle de acesso (IAM)** da conta de armazenamento, selecione **+ Adicionar** e, no menu suspenso, selecione **Adicionar atribuição de função**, 
1. Na folha **Adicionar atribuição de função**, especifique as seguintes configurações e selecione **Salvar**:

   |Configuração|Valor|
   |---|---|
   |Função|**Colaborador elevado de compartilhamento SMB de dados de arquivo de armazenamento**|
   |Atribuir acesso a|**Usuário, grupo ou entidade de serviço**|
   |Selecionar|**az140-wvd-admins**|

   > **Observação**: O **grupo az140-wvd-admins** contém a **conta de usuário wvdadmin1**, que você usará para configurar permissões de compartilhamento. 

1. Repita as duas etapas anteriores para configurar as seguintes atribuições de função:

   |Configuração|Valor|
   |---|---|
   |Função|**Colaborador elevado de compartilhamento SMB de dados de arquivo de armazenamento**|
   |Atribuir acesso a|**Usuário, grupo ou entidade de serviço**|
   |Selecionar|**az140-hosts-42-p1**|

   |Configuração|Valor|
   |---|---|
   |Função|**Leitor de compartilhamento SMB de dados de arquivo de armazenamento**|
   |Atribuir acesso a|**Usuário, grupo ou entidade de serviço**|
   |Selecionar|**az140-wvd-users**|

   > **Observação**: Os usuários e hosts da Área de Trabalho Virtual do Azure precisam pelo menos de acesso de leitura ao compartilhamento de arquivos.

1. Na folha da conta de armazenamento, no menu vertical à esquerda, na seção **Armazenamento de Dados**, selecione **Compartilhamentos de arquivos** e, em seguida, selecione ** + Compartilhamento de arquivos **.
1. Na folha **Novo compartilhamento de arquivos**, especifique as seguintes configurações e selecione **Criar** (deixe outras configurações com seus valores padrão):

   |Configuração|Valor|
   |---|---|
   |Nome|**az140-42-msixvhds**|

1. No Microsoft Edge que exibe o portal do Azure, na lista de compartilhamentos de arquivos, selecione o compartilhamento de arquivos recém-criado. 

1. Na sessão do Bastion para **az140-cl-vm42**, inicie o **Prompt de Comando** e, na janela **Prompt de Comando**, execute o seguinte para mapear uma unidade para o compartilhamento de **az140-42-msixvhds** (substitua o espaço reservado `<storage-account-name>` pelo nome da conta de armazenamento) e verifique se o comando foi concluído com sucesso:

   ```cmd
   net use Z: \\<storage-account-name>.file.core.windows.net\az140-42-msixvhds
   ```

1. Na sessão do Bastion para **az140-cl-vm42**, na janela **Prompt de Comando**, execute o seguinte para conceder as permissões NTFS necessárias para as contas de computador de hosts de sessão:

   ```cmd
   icacls Z:\ /grant ADATUM\az140-hosts-42-p1:(OI)(CI)(RX) /T
   icacls Z:\ /grant ADATUM\az140-wvd-users:(OI)(CI)(RX) /T
   icacls Z:\ /grant ADATUM\az140-wvd-admins:(OI)(CI)(F) /T
   ```

   >**Observação**: Você também pode definir essas permissões usando o Explorador de Arquivos enquanto estiver conectado como **wvdadmin1\@adatum.com**. 

   >**Observação**: Em seguida, você validará a funcionalidade da anexação do aplicativo MSIX

1. Na sessão do Bastion para **az140-cl-vm42**, no ISE do Windows Power Shell** na Janela do Administrador**, execute o seguinte para copiar o arquivo VHD criado no exercício anterior para o compartilhamento de Arquivos do Azure criado anteriormente neste exercício:

   ```powershell
   New-Item -ItemType Directory -Path 'Z:\packages' 
   Copy-Item -Path 'C:\Allfiles\Labs\04\MSIXVhds\XmlNotepad.vhd' -Destination 'Z:\packages' -Force
   ```

#### Tarefa 3: Montar e registrar a imagem de anexação do aplicativo MSIX em hosts de sessão da Área de Trabalho Virtual do Azure

1. Na sessão do Bastion para **az140-cl-vm42**, na janela do Microsoft Edge exibindo o portal do Azure, pesquise e selecione a **Área de Trabalho Virtual do Azure** e, na folha da **Área de Trabalho Virtual do Azure**, no menu vertical à esquerda, na seção **Gerenciar**, selecione **Pools de host**.
1. Na folha **Pools de host\| da Área de Trabalho Virtual do Azure**, na lista de pools de hosts, selecione a entrada **az140-21-hp1**.
1. Na folha **Propriedades \| az140-21-hp1**, no menu vertical no lado esquerdo, na seção **Gerenciar**, selecione **pacotes MSIX**.
1. Na folha de pacotes MSIX **az140-21-hp1\|, clique na folha pacotes MSIX** + **Adicionar**.
1. Na folha **Adicionar pacote MSIX**, na caixa de texto **Caminho da imagem MSIX**, insira o caminho para o arquivo **XmlNotepad.vhd** no formato `\\<storage-account-name>.file.core.windows.net\az140-42-msixvhds\packages\XmlNotepad.vhd` (substitua o espaço reservado `<storage-account-name>` pelo nome da conta de armazenamento que hospeda o compartilhamento de arquivo **az140-42-msixvhds**) e clique em **Adicionar**.
1. Na folha **Adicionar pacote MSIX**, especifique as seguintes configurações e clique em **Adicionar**:

   |Configuração|Valor|
   |---|---|
   |Caminho da imagem MSIX|**\\\\\<storage-account-name\>.file.core.windows.net\\az140-42-msixvhds\\XmlNotepad.vhd**, em que o espaço reservado `<storage-account-name>` designa o nome da conta de armazenamento que hospeda o compartilhamento de arquivo **az140-42-msixvhds**|
   |Pacote MSIX|o nome gerado durante a criação do pacote|
   |Nome de exibição|**Bloco de notas XML**|
   |Tipo de registro|**Sob demanda**|
   |Estado|**Com atividade**|

#### Tarefa 4: Publicar aplicativos MSIX em um grupo de aplicativos

> **Observação**: Você publicará o aplicativo MSIX no aplicativo remoto e no grupo de aplicativos da área de trabalho. 

1. Na sessão do Bastion para **az140-cl-vm42**, na janela do Microsoft Edge exibindo o portal do Azure, pesquise e selecione a **Área de Trabalho Virtual do Azure** e, na folha **Área de Trabalho Virtual do Azure**, no menu vertical à esquerda, na seção **Gerenciar**, selecione **Grupos de aplicativos**.
1. Na folha **Grupos de Aplicativos \| da Área de Trabalho Virtual do Azure**, selecione a entrada do grupo de aplicativos **az140-21-hp1-Utilities-RAG**.
1. Na folha **az140-21-hp1-Utilities-RAG**, no menu vertical no lado esquerdo, na seção **Gerenciar**, selecione **Aplicativos**. 
1. Na folha **az140-21-hp1-Utilities-RA\| Aplicativos**, clique em **+ Adicionar**.
1. Na folha **Adicionar aplicativo**, especifique as seguintes configurações e selecione **Salvar**:

   |Configuração|Valor|
   |---|---|
   |Origem do aplicativo|**Pacote MSIX**|
   |Pacote MSIX|o nome que representa o pacote incluído na imagem|
   |Aplicativo MSIX|**XMLNOTEPAD**|
   |Nome do aplicativo|**Bloco de notas XML**|
   |Nome de exibição|**Bloco de notas XML**|
   |Descrição|**Bloco de notas XML**|
   |Caminho do ícone|**C: \\Arquivos de Programas\\ WindowsApps\\XmlNotepad_2.8.0.0_x64___4vm7ty4fw38e8\\VFS\\ProgramFilesX86\\LovettSoftware\\XmlNotepad\\XmlNotepad.exe**|
   |Índice do ícone|**0**|

1. Navegue de volta para a folha **Grupos de aplicativos da \|Área de Trabalho Virtual do Azure** e selecione a entrada do grupo de aplicativos **az140-21-hp1-DAG**.
1. Na folha **az140-21-hp1-DAG**, no menu vertical no lado esquerdo, na seção **Gerenciar**, selecione **Aplicativos**. 
1. Na folha **az140-21-hp1-DAG\| Aplicativos**, clique em **+ Adicionar**.
1. Na folha **Adicionar aplicativo**, especifique as seguintes configurações e selecione **Salvar**:

   |Configuração|Valor|
   |---|---|
   |Origem do aplicativo|**Pacote MSIX**|
   |Pacote MSIX|o nome que representa o pacote incluído na imagem|
   |Nome do aplicativo|**Bloco de notas XML**|
   |Nome de exibição|**Bloco de notas XML**|
   |Descrição|**Bloco de notas XML**|

#### Tarefa 5: Validar a funcionalidade da anexação do aplicativo MSIX

1. Na sessão do Bastion para **az140-cl-vm42**, inicie o Microsoft Edge, navegue até a [página de download do cliente da Área de Trabalho do Windows](https://go.microsoft.com/fwlink/?linkid=2068602) e, depois que o download for concluído, selecione **Abrir arquivo** para iniciar sua instalação. Na página **Escopo de Instalação** do assistente de **Instalação da Área de Trabalho Remota**, selecione a opção **Instalar para todos os usuários deste computador** e clique em **Instalar**. 
1. Quando a instalação for concluída, verifique se a caixa de seleção **Iniciar Área de Trabalho Remota quando a instalação for encerrada** está marcada e clique em **Concluir** para iniciar o cliente de Área de Trabalho Remota.
1. Na janela **cliente da Área de Trabalho Remota**, selecione **Assinar** e, quando solicitado, entre com o nome de entidade de segurança do usuário **aduser1** e a senha definida ao criar essa conta de usuário. 
1. Se solicitado, na janela **Permanecer conectado a todos os seus aplicativos**, desmarque a caixa de seleção **Permitir que minha organização gerencie meu dispositivo** e clique em **Não, entre somente neste aplicativo**.
1. Na janela cliente da **Área de Trabalho Remota**, na seção **az140-21-ws1**, clique duas vezes no ícone do **Bloco de Notas XML**, quando solicitado, forneça a senha e verifique se o Bloco de Notas XML é iniciado com sucesso.


### Exercício 4: Parar e desalocar VMs do Azure provisionadas e usadas no laboratório

As principais tarefas deste exercício são as seguintes:

1. Parar e desalocar VMs do Azure provisionadas e usadas no laboratório

>**Observação**: Neste exercício, você desalocará as VMs do Azure provisionadas e usadas neste laboratório para minimizar os encargos de computação correspondentes

#### Tarefa 1: Desalocar VMs do Azure provisionadas e usadas no laboratório

1. Alterne para o computador de laboratório e, na janela do navegador da Web que exibe o portal do Azure, abra a sessão do shell do **PowerShell** no painel do **Cloud Shell**.
1. Na sessão do PowerShell no painel do Cloud Shell, execute o seguinte para listar todas as VMs do Azure criadas e usadas neste laboratório:

   ```powershell
   Get-AzVM -ResourceGroup 'az140-21-RG'
   Get-AzVM -ResourceGroup 'az140-42-RG'
   ```

1. Na sessão do PowerShell no painel do Cloud Shell, execute o seguinte para parar e desalocar todas as VMs do Azure que você criou e usou neste laboratório:

   ```powershell
   Get-AzVM -ResourceGroup 'az140-21-RG' | Stop-AzVM -NoWait -Force
   Get-AzVM -ResourceGroup 'az140-42-RG' | Stop-AzVM -NoWait -Force
   ```

   >**Observação**: O comando é executado de modo assíncrono (conforme determinado pelo parâmetro -NoWait), portanto, embora você possa executar outro comando do PowerShell imediatamente depois na mesma sessão do PowerShell, levará alguns minutos antes de o grupo de recursos ser de fato removido.
