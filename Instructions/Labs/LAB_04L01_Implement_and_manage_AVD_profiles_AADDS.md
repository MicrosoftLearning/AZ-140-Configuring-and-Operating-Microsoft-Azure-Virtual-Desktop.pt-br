---
lab:
  title: 'Laboratório: Implementar e gerenciar perfis da Área de Trabalho Virtual do Azure (Microsoft Entra DS)'
  module: 'Module 4: Manage User Environments and Apps'
---

# Laboratório - implementar e gerenciar perfis da Área de Trabalho Virtual do Azure (Microsoft Entra DS)
# Manual de laboratório do aluno

## Dependências do laboratório

- Uma assinatura do Azure
- Uma conta Microsoft ou uma conta do Microsoft Entra com a função de administrador global no locatário do Microsoft Entra associado à assinatura do Azure e com a função de proprietário ou colaborador na assinatura
- Um ambiente de Área de Trabalho Virtual do Azure provisionado no laboratório **Introdução à Área de Trabalho Virtual do Azure (Microsoft Entra DS)**

## Tempo estimado

30 minutos

## Cenário do laboratório

Você precisa implementar o gerenciamento de perfil da Área de Trabalho Virtual do Azure em um ambiente do Microsoft Entra DS.

## Objetivos
  
Depois de realizar este laboratório, você será capaz de:

- Configurar os Arquivos do Azure para armazenar contêineres de perfil para a Área de Trabalho Virtual do Azure em um ambiente do Microsoft Entra DS
- Implementar perfis baseados em FSLogix para a Área de Trabalho Virtual do Azure no ambiente do Microsoft Entra DS

## Arquivos de laboratório

- Nenhum

## Instruções

### Exercício 1: Implementar perfis baseados em FSLogix para a Área de Trabalho Virtual do Azure

As principais tarefas deste exercício são as seguintes:

1. Configurar o grupo Administradores local nas VMs de host da sessão da Área de Trabalho Virtual do Azure
1. Configurar perfis baseados em FSLogix em VMs de host da sessão da Área de Trabalho Virtual do Azure
1. Testar perfis baseados em FSLogix com a Área de Trabalho Virtual do Azure
1. Excluir recursos do laboratório do Azure

#### Tarefa 1: Configurar o grupo Administradores local nas VMs de host da sessão da Área de Trabalho Virtual do Azure

1. No seu computador de laboratório, inicie um navegador da Web, navegue até o [portal do Azure](https://portal.azure.com) e entre fornecendo credenciais de uma conta de usuário com a função Proprietário na assinatura que você usará neste laboratório.
1. No portal do Azure, abra o painel **Cloud Shell** selecionando o ícone da barra de ferramentas à direita da caixa de texto de pesquisa.
1. Se for solicitado que você selecione **Bash** ou **PowerShell**, selecione **PowerShell**. 

   >**Observação**: se esta é a primeira vez que você inicia o **Cloud Shell** e recebe a mensagem **Você não tem nenhum armazenamento montado**, selecione a assinatura que está usando neste laboratório e selecione **Criar armazenamento**. 

1. Na sessão do PowerShell no painel **Cloud Shell**, execute o seguinte para iniciar as VMs do Azure de host da sessão da Área de Trabalho Virtual do Azure que você usará neste laboratório:

   ```powershell
   Get-AzVM -ResourceGroup 'az140-21a-RG' | Start-AzVM
   ```

   >**Observação**: Aguarde até que as VMs do Azure estejam em execução antes de prosseguir para a próxima etapa.
   
      
1. Na sessão do PowerShell no painel **Cloud Shell**, execute o seguinte para habilitar a Comunicação Remota do PowerShell nos hosts da sessão.

   ```powershell
   Get-AzVM -ResourceGroup 'az140-21a-RG' | Enable-AzVMPSRemoting
   ```
   
1. Fechar o Cloud Shell
1. No seu computador de laboratório, no portal do Azure, pesquise e selecione **Máquinas virtuais** e, na folha **Máquinas virtuais**, selecione a entrada **az140-cl-vm11a**. Isso abrirá a folha **az140-cl-vm11a**.
1. Na folha **az140-cl-vm11a**, selecione **Conectar**, no menu suspenso, selecione **Bastion**, na guia **Bastion** da folha **az140-cl-vm11a \| Conectar**, selecione **Usar Bastion**.
1. Quando solicitado, forneça as seguintes credenciais e selecione **Conectar**:

   |Configuração|Valor|
   |---|---|
   |Nome do usuário|**aadadmin1@adatum.com**|
   |Senha|Senha configurada anteriormente|

1. Na sessão Bastion para **az140-cl-vm11a**, no menu Iniciar, navegue até a pasta **Ferramentas de Administração do Windows**, expanda-a e selecione **Usuários e Computadores do Active Directory**.
1. No console **Usuários e Computadores do Active Directory**, clique com o botão direito do mouse no nó do domínio, selecione **Novo**, seguido de **Unidade Organizacional**, na caixa de diálogo **Novo Objeto - Unidade Organizacional**, na caixa de texto **Nome**, digite **Usuários ADDC** e selecione **OK**.
1. No console **Usuários e Computadores do Active Directory**, clique com o botão direito do mouse em **Usuários do ADDC**, selecione **Novo**, seguido de **Grupo**, na caixa de diálogo **Novo Objeto - Grupo**, especifique as seguintes configurações e selecione **OK**:

   |Configuração|Valor|
   |---|---|
   |Nome do grupo|**Administradores Locais**|
   |Nome do grupo (anterior ao Windows 2000)|**Administradores Locais**|
   |Group scope|**Global**|
   |Tipo de grupo|**Segurança**|

1. No console **Usuários e Computadores do Active Directory**, exiba as propriedades do grupo **Administradores Locais**, alterne para a guia **Membros**, selecione **Adicionar**, na caixa de diálogo **Selecionar Usuários, Contatos, Computadores, Contas de Serviço ou Grupos**, em **Insira os nomes de objeto para selecionar**, digite **aadadmin1; wvdaadmin1** e selecione **OK**.
1. Na sessão Bastion para **az140-cl-vm11a**, no menu Iniciar, navegue até a pasta **Ferramentas de Administração do Windows**, expanda-a e selecione **Gerenciamento de Política de Grupo**.
1. No console de **Gerenciamento de Política de Grupo**, navegue até a UO **Computadores AADDC**, clique com o botão direito do mouse no ícone **GPO dos Computadores AADDC** e selecione **Editar**.
1. No console do **Editor de Gerenciamento de Política de Grupo**, expanda **Configuração de Computador**, **Políticas**, **Configurações do Windows**, **Configurações de Segurança**, clique com o botão direito do mouse em **Grupos Restritos**e selecione **Adicionar Grupo**.
1. Na caixa de diálogo **Adicionar Grupo**, na caixa de texto **Grupo**, selecione **Procurar**, na caixa de diálogo **Selecionar Grupos**, em **Inserir os nomes dos objetos a serem selecionados**, digite **Administradores Locais** e selecione **OK**.
1. De volta à caixa de diálogo **Adicionar Grupo**, selecione **OK**.
1. Na caixa de diálogo **ADATUM\Propriedades de Administradores Locais**, na seção rotulada **Esse grupo é membro do**, selecione **Adicionar**, na caixa de diálogo **Associação de Grupo** digite **Administradores**, selecione **OK** e selecione **OK** novamente para finalizar a alteração.

   >**Observação**: Use a seção rotulada **Esse grupo é membro do**

1. Na sessão do Bastion para a VM do Azure az140-cl-vm11a, inicie o ISE do PowerShell como Administrador e execute o seguinte para reiniciar os dois hosts da Área de Trabalho Virtual do Azure para disparar o processamento da Política de Grupo:

   ```powershell
   $servers = 'az140-21-p1-0','az140-21-p1-1'
   Restart-Computer -ComputerName $servers -Force -Wait
   ```

1. Aguarde a conclusão do script. Isso deverá levar cerca de 3 minutos.

#### Tarefa 2: Configurar perfis baseados em FSLogix em VMs de host da sessão da Área de Trabalho Virtual do Azure

1. Na sessão do Bastion para **az140-cl-vm11a**, inicie uma sessão de Área de Trabalho Remota para **az140-21-p1-0** e, quando solicitado, entre com o nome de usuário **ADATUM\wvdaadmin1** e a senha definida ao criar essa conta de usuário. 

   > **Observação**: Se a conexão RDP não puder se conectar, use o Portal do Azure para se conectar à VM usando o Bastion.

1. Na sessão da Área de Trabalho Remota para **az140-21-p1-0**, inicie o Microsoft Edge, navegue até a [página de download do FSLogix](https://aka.ms/fslogix_download), baixe os binários de instalação compactados do FSLogix, extraia-os na pasta **C:\\Origem**, navegue até a subpasta **x64\\Versão** e use **FSLogixAppsSetup.exe** para instalar o Microsoft FSLogix Apps com as configurações padrão.

   > **Observação**: A instalação do FXLogic pode não ser necessária, dependendo se a imagem já o inclui. Uma instalação do FX Logix requer uma reinicialização.

1. Na sessão da Área de Trabalho Remota para **az140-21-p1-0**, inicie o **ISE do PowerShell do Windows** como administrador e, no painel de script **Administrador: No painel de script do ISE do Windows PowerShell**: execute o seguinte para instalar a versão mais recente do módulo PowerShellGet (selecione **Sim** quando solicitado a confirmar):

   ```powershell
   [Net.ServicePointManager]::SecurityProtocol = [Net.SecurityProtocolType]::Tls12
   Install-Module -Name PowerShellGet -Force -SkipPublisherCheck
   ```

1. A partir do **Administrador: Console ISE do Windows PowerShell**: execute o seguinte para instalar a versão mais recente do módulo do Az PowerShell (selecione **Sim para Todos** quando solicitado a confirmação):

   ```powershell
   Install-Module -Name Az -AllowClobber -SkipPublisherCheck
   ```

1. A partir do **Administrador: Console ISE** do Windows PowerShell, execute o seguinte para modificar a política de execução:

   ```powershell
   Set-ExecutionPolicy RemoteSigned -Force
   ```

1. A partir do **Administrador: ISE do Windows PowerShell**, execute o seguinte para entrar em sua assinatura do Azure:

   ```powershell
   Connect-AzAccount
   ```

1. Quando solicitado, entre com as credenciais do Microsoft Entra da conta de usuário com a função de Proprietário na assinatura que você está usando neste laboratório.
1. A partir do **Administrador: No painel de script do ISE do Windows PowerShell**, execute o seguinte para recuperar o nome da conta de Armazenamento do Microsoft Azure configurada anteriormente neste laboratório:

   ```powershell
   $resourceGroupName = 'az140-22a-RG'
   $storageAccountName = (Get-AzStorageAccount -ResourceGroupName $resourceGroupName)[0].StorageAccountName   
   ```

1. Na sessão da Área de Trabalho Remota para **az140-21-p1-0**, no painel de script do **Administrador: No painel de script do ISE do Windows PowerShell**, execute o seguinte para definir as configurações do registro de perfil:

   ```powershell
   $profilesParentKey = 'HKLM:\SOFTWARE\FSLogix'
   $profilesChildKey = 'Profiles'
   $fileShareName = 'az140-22a-profiles'
   New-Item -Path $profilesParentKey -Name $profilesChildKey -Force
   New-ItemProperty -Path $profilesParentKey\$profilesChildKey -Name 'Enabled' -PropertyType DWord -Value 1
   New-ItemProperty -Path $profilesParentKey\$profilesChildKey -Name 'VHDLocations' -PropertyType MultiString -Value "\\$storageAccountName.file.core.windows.net\$fileShareName"
   ```
   >**Observação** Se o comando gerar um erro, vá para a próxima etapa.
   
1. Na sessão da Área de Trabalho Remota para **az140-21-p1-0**, clique com o botão direito do mouse em **Iniciar**, no menu do botão direito do mouse, selecione **Executar**, na caixa de diálogo **Executar**, na caixa de texto **Abrir**, digite o seguinte e selecione **OK** para abrir a janela **Usuários e Grupos Locais**:

   ```cmd
   lusrmgr.msc
   ```

1. No console **Usuários e Grupos Locais**, observe os quatro grupos cujos nomes começam com a cadeia de caracteres **FSLogix**:

   - Lista de Exclusão de ODFC do FSLogix
   - Lista de Inclusão de ODFC do FSLogix
   - Lista de Exclusão de Perfil do FSLogix
   - Lista de Inclusão de Perfil do FSLogix

1. No console **Usuários e Grupos Locais**, clique duas vezes na entidade de grupo **Lista de Inclusão do Perfil do FSLogix**, observe que ele inclui o grupo **\\Todos** e selecione **OK** para fechar a janela **Propriedades** do grupo. 
1. No console **Usuários e Grupos Locais**, clique duas vezes na entidade de grupo **Lista de Exclusão do Perfil do FSLogix**, observe que ele não inclui membros de grupo por padrão e selecione **OK** para fechar a janela **Propriedades** do grupo. 

   > **Observação**: Para fornecer uma experiência de usuário consistente, você precisa instalar e configurar componentes FSLogix em todos os hosts de sessão da Área de Trabalho Virtual do Azure. Você executará essa tarefa de maneira autônoma no outro host da sessão em nosso ambiente de laboratório. 

1. Na sessão da Área de Trabalho Remota para **az140-21-p1-0**, no painel de script do **Administrador: No painel de script do ISE do Windows PowerShell**, execute o seguinte para instalar os componentes do FSLogix no host da sessão **az140-21-p1-1**:

   ```powershell
   $server = 'az140-21-p1-1' 
   $localPath = 'C:\Source\x64'
   $remotePath = "\\$server\C$\Source\x64\Release"
   Copy-Item -Path $localPath\Release -Destination $remotePath -Filter '*.exe' -Force -Recurse
   Invoke-Command -ComputerName $server -ScriptBlock {
      Start-Process -FilePath $using:localPath\Release\FSLogixAppsSetup.exe -ArgumentList '/quiet' -Wait
   } 
   ```

1. Na sessão da Área de Trabalho Remota para **az140-21-p1-0**, inicie o **ISE do PowerShell do Windows** como administrador e, no painel de script **Administrador: No painel de script do ISE do Windows PowerShell**, execute o seguinte para definir as configurações do registro de perfil no host da sessão **az140-21-p1-1**:

   ```powershell
   $profilesParentKey = 'HKLM:\SOFTWARE\FSLogix'
   $profilesChildKey = 'Profiles'
   $fileShareName = 'az140-22a-profiles'
   Invoke-Command -ComputerName $server -ScriptBlock {
      New-Item -Path $using:profilesParentKey -Name $using:profilesChildKey -Force
      New-ItemProperty -Path $using:profilesParentKey\$using:profilesChildKey -Name 'Enabled' -PropertyType DWord -Value 1
      New-ItemProperty -Path $using:profilesParentKey\$using:profilesChildKey -Name 'VHDLocations' -PropertyType MultiString -Value "\\$storageAccountName.file.core.windows.net\$using:fileShareName"
   }
   ```

   > **Observação**: Antes de testar a funcionalidade de perfil baseada em FSLogix, você precisa remover o perfil armazenado em cache localmente da conta ADATUM\wvdaadmin1 que você usará para teste dos hosts de sessão da Área de Trabalho Virtual do Azure usados no laboratório anterior.

1. Alterne para a sessão do Bastion para **az140-cl-vm11a**, dentro da sessão do Bastion para **az140-cl-vm11a**, alterne para o Administrador **: Janela do ISE do Windows PowerShell** e no **Administrador: No painel de script do ISE do Windows PowerShell**, execute o seguinte para remover o perfil armazenado em cache localmente da conta so ADATUM\aaduser1:

   ```powershell
   $userName = 'aaduser1'
   $servers = 'az140-21-p1-0','az140-21-p1-1'
   Get-CimInstance -ComputerName $servers -Class Win32_UserProfile | Where-Object { $_.LocalPath.split('\')[-1] -eq $userName } | Remove-CimInstance
   ```

#### Tarefa 3: Testar perfis baseados em FSLogix com a Área de Trabalho Virtual do Azure

1. Na sessão Bastion para **az140-cl-vm11a**, alterne para o cliente da Área de Trabalho Remota.
1. Na sessão do Bastion para **az140-cl-vm11a**, na janela do cliente da **Área de Trabalho Remota**, na lista de aplicativos, clique duas vezes em **Prompt de Comando**, quando solicitado, forneça a senha e verifique se uma janela do **Prompt de Comando** é aberta. 

   > **Observação**: Pode levar alguns minutos para o aplicativo ser iniciado na primeira vez, mas, posteriormente, a inicialização do aplicativo deve ser muito mais rápida.

1. No canto superior esquerdo da janela do **Prompt de Comando**, clique com o botão direito do mouse no ícone do **Prompt de Comando** e, no menu suspenso, selecione **Propriedades**.
1. Na caixa de diálogo **Propriedades do Prompt de Comando**, selecione a guia **Fonte**, modifique as configurações de tamanho e fonte e selecione **OK**.
1. Na janela **Prompt de Comando**, digite **fazer logoff** e pressione a tecla **Enter** para sair da sessão da Área de Trabalho Remota.
1. Na sessão do Bastion para **az140-cl-vm11a**, na janela do cliente da **Área de Trabalho Remota**, na lista de aplicativos, clique duas vezes em **SessionDesktop** e verifique se uma sessão da Área de Trabalho Remota é iniciada. 
1. Na sessão **SessionDesktop**, clique com o botão direito do mouse em **Iniciar**, no menu do botão direito do mouse, selecione **Executar**, na caixa de diálogo **Executar**, na caixa de texto **Abrir**, digite **cmd** e selecione **OK** para iniciar uma janela do **Prompt de Comando**:
1. Verifique se as propriedades da janela do **Prompt de Comando** correspondem àquelas que você definiu anteriormente nesta tarefa.
1. Na sessão **SessionDesktop**, minimize todas as janelas, clique com o botão direito do mouse na área de trabalho, no menu do botão direito do mouse, selecione **Novo** e, no menu em cascata, selecione **Atalho**. 
1. Na página **Para qual item você gostaria de criar um atalho?**, do assistente **Criar Atalho**, na caixa de texto **Digite o local do item**, digite **Bloco de Notas** e selecione **Avançar**.
1. Na página **Como você gostaria de nomear o atalho** do assistente **Criar Atalho**, na caixa de texto **Digite um nome para este atalho**, digite **Bloco de Notas** e selecione **Concluir**.
1. Na sessão **SessionDesktop**, clique com o botão direito do mouse em **Iniciar**, no menu do botão direito do mouse, selecione **Desligar ou sair** e, em seguida, no menu em cascata, selecione **Sair**.
1. Volte à sessão do Bastion para **az140-cl-vm11a**, na janela do cliente da **Área de Trabalho Remota**, na lista de aplicativos e clique duas vezes em **SessionDesktop** para iniciar uma nova sessão da Área de Trabalho Remota. 
1. Na sessão **SessionDesktop**, verifique se o atalho do **Bloco de Notas** aparece na área de trabalho.
1. Na sessão **SessionDesktop**, clique com o botão direito do mouse em **Iniciar**, no menu do botão direito do mouse, selecione **Desligar ou sair** e, em seguida, no menu em cascata, selecione **Sair**.
1. Alterne para a sessão Bastion para **az140-cl-vm11a**, alterne para a janela do Microsoft Edge que exibe o portal do Azure.
1. Na janela do Microsoft Edge que exibe o portal do Azure, retorne para a folha **Contas de armazenamento** e selecione a entrada que representa a conta de armazenamento criada no exercício anterior.
1. Na folha da conta de armazenamento, na seção **Serviços de arquivo**, selecione **Compartilhamentos de arquivos** e, na lista de compartilhamentos de arquivos, selecione **az140-22a-profiles**. 
1. Na folha **az140-22a-profiles**, selecione **Procurar** e verifique se seu conteúdo inclui uma pasta cujo nome consiste em uma combinação do identificador de segurança (SID) da conta **ADATUM\\ADATUMaaduser1** seguido pelo sufixo **_aaduser1**.
1. Selecione a pasta que você identificou na etapa anterior e observe que ela contém um único arquivo chamado **Profile_aaduser1.vhd**.

### Exercício 2: Excluir recursos do laboratório do Azure (opcional)

1. Remova a implantação do Microsoft Entra DS seguindo as instruções descritas em [Excluir um domínio gerenciado do Azure Active Directory Domain Services usando o portal do Azure]( https://docs.microsoft.com/en-us/azure/active-directory-domain-services/delete-aadds).
1. Remova todos os grupos de recursos do Azure provisionados nos laboratórios do Microsoft Entra DS deste curso seguindo as instruções descritas em [Grupo de recursos e exclusão de recursos do Azure Resource Manager](https://docs.microsoft.com/en-us/azure/azure-resource-manager/management/delete-resource-group?tabs=azure-portal).
