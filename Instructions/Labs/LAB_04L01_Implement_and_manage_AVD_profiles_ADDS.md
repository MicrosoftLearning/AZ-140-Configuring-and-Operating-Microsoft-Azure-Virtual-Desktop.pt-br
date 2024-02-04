---
lab:
  title: 'Laboratório: Implementar e gerenciar perfis da Área de Trabalho Virtual do Azure (AD DS)'
  module: 'Module 4: Manage User Environments and Apps'
---

# Laboratório - Implementar e gerenciar perfis da Área de Trabalho Virtual do Azure (AD DS)
# Manual de laboratório do aluno

## Dependências do laboratório

- O nome da assinatura do Azure que você usará nesse laboratório
- Uma conta Microsoft ou uma conta do Microsoft Entra com a função de Proprietário ou Colaborador na assinatura do Azure que você usará neste laboratório e com a função de Administrador Global no locatário do Microsoft Entra associado a essa assinatura do Azure.
- O laboratório completo **Preparar para a implantação da Área de Trabalho Virtual do Azure (AD DS)**
- O laboratório completo **Implementar e gerenciar o armazenamento para AVD (AD DS)**

## Tempo estimado

30 minutos

## Cenário do laboratório

Você precisa implementar o gerenciamento de perfil da Área de Trabalho Virtual do Azure em um ambiente do AD DS (Active Directory Domain Services).

## Objetivos
  
Depois de realizar este laboratório, você será capaz de:

- Implementar perfis baseados em FSLogix para a Área de Trabalho Virtual do Azure

## Arquivos de laboratório

- Nenhum

## Instruções

### Exercício 1: Implementar perfis baseados em FSLogix para a Área de Trabalho Virtual do Azure

As principais tarefas deste exercício são as seguintes:

1. Configurar perfis baseados em FSLogix em VMs de host da sessão da Área de Trabalho Virtual do Azure
1. Testar perfis baseados em FSLogix com a Área de Trabalho Virtual do Azure
1. Remover recursos do Azure implantados no laboratório

#### Tarefa 1: Configurar perfis baseados em FSLogix em VMs de host da sessão da Área de Trabalho Virtual do Azure

1. No seu computador de laboratório, inicie um navegador da Web, navegue até o [portal do Azure](https://portal.azure.com) e entre fornecendo credenciais de uma conta de usuário com a função Proprietário na assinatura que você usará neste laboratório.
1. No portal do Azure, pesquise e selecione **Máquinas virtuais** e, na folha **Máquinas virtuais**, selecione **az140-21-p1-0**.
1. Na folha **az140-21-p1-0**, selecione **Iniciar** e aguarde até que o status da máquina virtual mude para **Em execução**.
1. Na folha **az140-21-p1-0**, selecione **Conectar**, no menu suspenso, selecione **Bastion**, na guia **Bastion** da folha **az140-21-p1-0 \| Conectar**, selecione **Usar Bastion**.
1. Quando solicitado, entre com as seguintes credenciais:

   |Configuração|Valor|
   |---|---|
   |Nome do usuário|**student@adatum.com**|
   |Senha|**Pa55w.rd1234**|

1. Na sessão do Bastion para **az140-21-p1-0**, inicie o Microsoft Edge e navegue até o [portal do Azure](https://portal.azure.com). Se solicitado, entre usando as credenciais do Microsoft Entra da conta de usuário com a função de Proprietário na assinatura que está usando neste laboratório.
1. Na sessão Bastion para **az140-21-p1-0**, na janela do Microsoft Edge que exibe o portal do Azure, abra uma sessão do PowerShell no painel do Cloud Shell. 
1. Na sessão do PowerShell no painel **Cloud Shell**, execute o seguinte para iniciar as VMs do Azure de host da sessão da Área de Trabalho Virtual do Azure que você usará neste laboratório:

   ```powershell
   Get-AzVM -ResourceGroup 'az140-21-RG' | Start-AzVM
   ```

   >**Observação**: Aguarde até que as VMs do Azure estejam em execução antes de prosseguir para a próxima etapa.

1. Na sessão do PowerShell no painel **Cloud Shell**, execute o seguinte para habilitar a Comunicação Remota do PowerShell nos hosts da sessão.

   ```powershell
   Get-AzVM -ResourceGroup 'az140-21-RG' | Enable-AzVMPSRemoting
   ```
   
1. Fechar o Cloud Shell
1. Dentro da sessão Bastion para **az140-21-p1-0**, inicie o Microsoft Edge, navegue até a [página de download do FSLogix](https://aka.ms/fslogix_download), baixe os binários de instalação compactados do FSLogix, extraia-os para a pasta **C:\\Allfiles\\Labs\\04** (crie a pasta, se necessário), navegue até a subpasta **x64\\Versão**, clique duas vezes no arquivo **FSLogixAppsSetup.exe** para iniciar o assistente do **Microsoft FSLogix Apps Setup** e instale o Microsoft FSLogix Apps com as configurações padrão.

   > **Observação**: A instalação do FXLogic não será necessária se a imagem já o incluir.

1. Na sessão do Bastion para **az140-21-p1-0**, inicie o **ISE do PowerShell do Windows** como administrador e, no painel de script **Administrador: Painel de script ISE** do Windows PowerShell, execute o seguinte para instalar a versão mais recente do módulo PowerShellGet (selecione **Sim** quando a confirmação for solicitada):

   ```powershell
   [Net.ServicePointManager]::SecurityProtocol = [Net.SecurityProtocolType]::Tls12
   Install-Module -Name PowerShellGet -Force -SkipPublisherCheck
   ```

1. No **ISE do Windows Power Shell: Console ISE** do Windows PowerShell, execute o seguinte para instalar a versão mais recente do módulo Az PowerShell (selecione **Sim para Todos** quando a confirmação for solicitada):

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
1. Na sessão do Bastion para **az140-21-p1-0**, no ISE do Windows Power Shell **: Painel de script do ISE do Windows PowerShell**, execute o seguinte para recuperar o nome da conta de Armazenamento do Microsoft Azure configurada anteriormente neste laboratório:

   ```powershell
   $resourceGroupName = 'az140-22-RG'
   $storageAccountName = (Get-AzStorageAccount -ResourceGroupName $resourceGroupName)[0].StorageAccountName   
   ```

1. Na sessão do Bastion para **az140-21-p1-0**, no ISE do Windows Power Shell **: No painel de script do ISE do Windows PowerShell**, execute o seguinte para definir as configurações do registro de perfil:

   ```powershell
   $profilesParentKey = 'HKLM:\SOFTWARE\FSLogix'
   $profilesChildKey = 'Profiles'
   $fileShareName = 'az140-22-profiles'
   New-Item -Path $profilesParentKey -Name $profilesChildKey –Force
   New-ItemProperty -Path $profilesParentKey\$profilesChildKey -Name 'Enabled' -PropertyType DWord -Value 1
   New-ItemProperty -Path $profilesParentKey\$profilesChildKey -Name 'VHDLocations' -PropertyType MultiString -Value "\\$storageAccountName.file.core.windows.net\$fileShareName"
   ```

1. Na sessão do Bastion para **az140-21-p1-0**, clique com o botão direito do mouse em **Iniciar**, no menu do botão direito do mouse, selecione **Executar**, na caixa de diálogo **Executar**, na caixa de texto **Abrir**, digite o seguinte e selecione **OK** para iniciar o console **Usuários e Grupos Locais**:

   ```cmd
   lusrmgr.msc
   ```

1. No console **Usuários e Grupos Locais**, observe os quatro grupos cujos nomes começam com a cadeia de caracteres **FSLogix**:

   - Lista de Exclusão de ODFC do FSLogix
   - Lista de Inclusão de ODFC do FSLogix
   - Lista de Exclusão de Perfil do FSLogix
   - Lista de Inclusão de Perfil do FSLogix

1. No console **Usuários e Grupos Locais**, na lista de grupos, clique duas vezes no grupo **Lista de Inclusão do Perfil do FSLogix**, observe que ele inclui o grupo **\\Todos** e selecione **OK** para fechar a janela **Propriedades** do grupo. 
1. No console **Usuários e Grupos Locais**, na lista de grupos, clique duas vezes no grupo **Lista de Exclusão do Perfil do FSLogix**, observe que ele não inclui membros de grupo por padrão e selecione **OK** para fechar a janela **Propriedades** do grupo. 

   > **Observação**: Para fornecer uma experiência de usuário consistente, você precisa instalar e configurar componentes FSLogix em todos os hosts de sessão da Área de Trabalho Virtual do Azure. Você executará essa tarefa de maneira autônoma nos outros hosts da sessão em nosso ambiente de laboratório. 

1. Na sessão do Bastion para **az140-21-p1-0**, no ISE do Windows Power Shell **: No painel de script do ISE do Windows PowerShell**, execute o seguinte para instalar os componentes do FSLogix nas sessões de host **az140-21-p1-1** e **az140-21-p1-2**:

   ```powershell
   $servers = 'az140-21-p1-1', 'az140-21-p1-2'
   foreach ($server in $servers) {
      $localPath = 'C:\Allfiles\Labs\04\x64'
      $remotePath = "\\$server\C$\Allfiles\Labs\04\x64\Release"
      Copy-Item -Path $localPath\Release -Destination $remotePath -Filter '*.exe' -Force -Recurse
      Invoke-Command -ComputerName $server -ScriptBlock {
         Start-Process -FilePath $using:localPath\Release\FSLogixAppsSetup.exe -ArgumentList '/quiet' -Wait
      } 
   }
   ```

   > **Observação**: Aguarde a conclusão da execução do script. Isso pode levar cerca de dois minutos.

1. Na sessão do Bastion para **az140-21-p1-0**, no ISE do Windows Power Shell **: No painel de script do ISE do Windows PowerShell**, execute o seguinte para definir as configurações do registro de perfil nas sessões de host **az140-21-p1-1** e **az140-21-p1-1**:

   ```powershell
   $profilesParentKey = 'HKLM:\SOFTWARE\FSLogix'
   $profilesChildKey = 'Profiles'
   $fileShareName = 'az140-22-profiles'
   foreach ($server in $servers) {
      Invoke-Command -ComputerName $server -ScriptBlock {
         New-Item -Path $using:profilesParentKey -Name $using:profilesChildKey –Force
         New-ItemProperty -Path $using:profilesParentKey\$using:profilesChildKey -Name 'Enabled' -PropertyType DWord -Value 1
         New-ItemProperty -Path $using:profilesParentKey\$using:profilesChildKey -Name 'VHDLocations' -PropertyType MultiString -Value "\\$using:storageAccountName.file.core.windows.net\$using:fileShareName"
      }
   }
   ```

   > **Observação**: Antes de testar a funcionalidade de perfil baseada em FSLogix, você precisa remover o perfil armazenado em cache localmente da conta **ADATUM\\aduser1** que você usará para teste dos hosts de sessão da Área de Trabalho Virtual do Azure usados no laboratório anterior.

1. Na sessão do Bastion para **az140-21-p1-0**, no ISE do Windows Power Shell **: Painel de script** ISE do Windows PowerShell, execute o seguinte para remover o perfil armazenado em cache localmente da conta do **ADATUM\\aduser1** em todas as VMs do Azure que atuam como hosts de sessão:

   ```powershell
   $userName = 'aduser1'
   $servers = 'az140-21-p1-0','az140-21-p1-1', 'az140-21-p1-2'
   Get-CimInstance -ComputerName $servers -Class Win32_UserProfile | Where-Object { $_.LocalPath.split('\')[-1] -eq $userName } | Remove-CimInstance
   ```

#### Tarefa 2: Testar perfis baseados em FSLogix com a Área de Trabalho Virtual do Azure

1. Alterne para o computador do laboratório, no computador do laboratório, na janela do navegador que exibe o portal do Azure, pesquise e selecione **Máquinas virtuais** e, na folha **Máquinas virtuais**, selecione a entrada**az140-cl-vm11**.
1. Na folha **az140-cl-vm11**, selecione **Conectar**, no menu suspenso, selecione **Bastion**, na guia **Bastion** da folha **az140-cl-vm11 \| Conectar**, selecione **Usar Bastion**.
1. Quando solicitado, forneça as seguintes credenciais e selecione **Conectar**:

   |Configuração|Valor|
   |---|---|
   |Nome do usuário|**Student@adatum.com**|
   |Senha|**Pa55w.rd1234**|

1. Na sessão do Bastion para **az140-cl-vm11**, clique em **Iniciar** e, no menu **Iniciar**, clique em **Área de Trabalho Remota** para iniciar o cliente da Área de Trabalho Remota.
1. Na sessão Bastion para **az140-cl-vm11**, na janela do cliente da **Área de Trabalho Remota**, selecione **Assinar** e, quando solicitado, entre com as credenciais **aduser1**.

   >**Observação** Se você não for solicitado a se inscrever, talvez seja necessário cancelar uma assinatura anterior.
3. na lista de aplicativos, clique duas vezes em **Prompt de Comando**, quando solicitado, forneça a senha da conta **aduser1** e verifique se uma janela do **Prompt de Comando** é aberta com êxito.
4. No canto superior esquerdo da janela do **Prompt de Comando**, clique com o botão direito do mouse no ícone do **Prompt de Comando** e, no menu suspenso, selecione **Propriedades**.
5. Na caixa de diálogo **Propriedades do Prompt de Comando**, selecione a guia **Fonte**, modifique as configurações de tamanho e fonte e selecione **OK**.
6. Na janela **Prompt de Comando**, digite **logoff** e pressione a tecla **Enter** para sair da sessão da Área de Trabalho Remota.
7. Na sessão do Bastion para **az140-cl-vm11**, na janela do cliente da **Área de Trabalho Remota**, na lista de aplicativos, clique duas vezes em **SessionDesktop** em az140-21-ws1 e verifique se uma sessão da Área de Trabalho Remota é iniciada. 
8. Na sessão **SessionDesktop**, clique com o botão direito do mouse em **Iniciar**, no menu do botão direito do mouse, selecione **Executar**, na caixa de diálogo **Executar**, na caixa de texto **Abrir**, digite **cmd** e selecione **OK** para iniciar uma janela do **Prompt de Comando**:
9. Verifique se as configurações da janela do **Prompt de Comando** correspondem àquelas que você configurou anteriormente nesta tarefa.
10. Na sessão **SessionDesktop**, minimize todas as janelas, clique com o botão direito do mouse na área de trabalho, no menu do botão direito do mouse, selecione **Novo** e, no menu em cascata, selecione **Atalho**. 
11. Na página **Para qual item você gostaria de criar um atalho?**, do assistente **Criar Atalho**, na caixa de texto **Digite o local do item**, digite **Bloco de Notas** e selecione **Avançar**.
12. Na página **Como você gostaria de nomear o atalho** do assistente **Criar Atalho**, na caixa de texto **Digite um nome para este atalho**, digite **Bloco de Notas** e selecione **Concluir**.
13. Na sessão **SessionDesktop**, clique com o botão direito do mouse em **Iniciar**, no menu do botão direito do mouse, selecione **Desligar ou sair** e, em seguida, no menu em cascata, selecione **Sair**.
14. Volte à sessão do Bastion para **az140-cl-vm11**, na janela do cliente da **Área de Trabalho Remota**, na lista de aplicativos e clique duas vezes em **SessionDesktop** para iniciar uma nova sessão da Área de Trabalho Remota. 
15. Na sessão **SessionDesktop**, verifique se o atalho do **Bloco de Notas** aparece na área de trabalho.
16. Na sessão **SessionDesktop**, clique com o botão direito do mouse em **Iniciar**, no menu do botão direito do mouse, selecione **Desligar ou sair** e, em seguida, no menu em cascata, selecione **Sair**.
17. Alterne para o computador do laboratório e, na janela do Microsoft Edge que exibe o portal do Azure, navegue até a folha **Contas de armazenamento** e selecione a entrada que representa a conta de armazenamento criada no exercício anterior.
18. Na folha da conta de armazenamento, na seção **Serviços de arquivo**, selecione **Compartilhamentos de arquivos** e, na lista de compartilhamentos de arquivos, selecione **az140-22-profiles**. 
19. Na folha **az140-22-profiles**, selecione **Procurar** e verifique se seu conteúdo inclui uma pasta cujo nome consiste em uma combinação do identificador de segurança (SID) da conta **ADATUM\\aduser1** seguido pelo sufixo **_aduser1**.
20. Selecione a pasta que você identificou na etapa anterior e observe que ela contém um único arquivo chamado **Profile_aduser1.vhd**.

### Exercício 2: Parar e desalocar VMs do Azure provisionadas e usadas no laboratório

As principais tarefas deste exercício são as seguintes:

1. Parar e desalocar VMs do Azure provisionadas e usadas no laboratório

>**Observação**: Neste exercício, você desalocará as VMs do Azure provisionadas e usadas neste laboratório para minimizar os encargos de computação correspondentes

#### Tarefa 1: Desalocar VMs do Azure provisionadas e usadas no laboratório

1. Alterne para o computador de laboratório e, na janela do navegador da Web que exibe o portal do Azure, abra a sessão do shell do **PowerShell** no painel do **Cloud Shell**.
1. Na sessão do PowerShell no painel do Cloud Shell, execute o seguinte para listar todas as VMs do Azure criadas e usadas neste laboratório:

   ```powershell
   Get-AzVM -ResourceGroup 'az140-21-RG'
   ```

1. Na sessão do PowerShell no painel do Cloud Shell, execute o seguinte para parar e desalocar todas as VMs do Azure que você criou e usou neste laboratório:

   ```powershell
   Get-AzVM -ResourceGroup 'az140-21-RG' | Stop-AzVM -NoWait -Force
   ```

   >**Observação**: O comando é executado de modo assíncrono (conforme determinado pelo parâmetro -NoWait), portanto, embora você possa executar outro comando do PowerShell imediatamente depois na mesma sessão do PowerShell, levará alguns minutos antes de o grupo de recursos ser de fato removido.
