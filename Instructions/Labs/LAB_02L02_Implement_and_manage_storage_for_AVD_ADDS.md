---
lab:
  title: 'Laboratório: Implementar e gerenciar o armazenamento para AVD (AD DS)'
  module: 'Module 2: Implement an AVD Infrastructure'
---

# Laboratório – Implementar e gerenciar o armazenamento para AVD (AD DS)
# Manual de laboratório do aluno

## Dependências do laboratório

- O nome da assinatura do Azure que você usará nesse laboratório
- Uma conta Microsoft ou uma conta do Microsoft Entra com a função de Proprietário ou Colaborador na assinatura do Azure que você usará neste laboratório e com a função de Administrador Global no locatário do Microsoft Entra associado a essa assinatura do Azure.
- O laboratório concluído: **Preparar para a implantação da Área de Trabalho Virtual do Azure (AD DS)**

## Tempo estimado

30 minutos

## Cenário do laboratório

Você precisa implementar e gerenciar o armazenamento para uma implantação da Área de Trabalho Virtual do Azure em um ambiente do AD DS.

## Objetivos
  
Depois de realizar este laboratório, você será capaz de:

- Configurar os Arquivos do Azure para armazenar contêineres de perfil para a Área de Trabalho Virtual do Azure

## Arquivos do laboratório

- Nenhum

## Instruções

### Exercício 1: Configurar os Arquivos do Azure para armazenar contêineres de perfil para a Área de Trabalho Virtual do Azure

As principais tarefas desse exercício são as seguintes:

1. Criar uma conta de Armazenamento do Azure
1. Criar um compartilhamento dos Arquivos do Azure
1. Habilitar a autenticação do AD DS para a conta de Armazenamento do Microsoft Azure 
1. Configurar as permissões baseadas em RBAC dos Arquivos do Azure
1. Configurar as permissões do sistema de arquivos dos Arquivos do Azure

#### Tarefa 1: Criar uma conta de Armazenamento do Azure

1. No seu computador de laboratório, inicie um navegador da Web, navegue até o [portal do Azure](https://portal.azure.com) e entre fornecendo credenciais de uma conta de usuário com a função de Proprietário na assinatura que você usará neste laboratório.
1. No portal do Azure, pesquise e selecione **Máquinas Virtuais** e, na folha **Máquinas Virtuais**, selecione **az140-dc-vm11**.
1. Na folha **az140-dc-vm11**, selecione **Conectar**, no menu suspenso, selecione **Conectar via Bastion**.
1. Quando solicitado, forneça as seguintes credenciais e selecione **Conectar**:

   |Configuração|Valor|
   |---|---|
   |Nome do usuário|**Student@adatum.com**|
   |Senha|**Pa55w.rd1234**|

1. Na sessão do Bastion para **az140-dc-vm11**, inicie o Microsoft Edge e navegue até o [portal do Azure](https://portal.azure.com). Se solicitado, entre usando as credenciais do Microsoft Entra da conta de usuário com a função Proprietário na assinatura que você está usando nesse laboratório.
1. Na sessão do Bastion para **az140-dc-vm11**, na janela do Microsoft Edge exibindo o portal do Azure, pesquise e selecione **contas de armazenamento** e, na folha **Contas de Armazenamento**, selecione **+ Criar**.
1. Na guia **Noções básicas** da folha **Criar conta de armazenamento**, defina as seguintes configurações (deixe outras com seus valores padrão):

   |Configuração|Valor|
   |---|---|
   |Assinatura|o nome da assinatura do Azure que você está usando neste laboratório|
   |Grupo de recursos|criar um **novo** grupo de recursos chamado **az140-22-RG**|
   |Nome da conta de armazenamento|qualquer nome global exclusivo com 3 a 15 caracteres de comprimento composto por dígitos e letras em minúsculas e começando com uma letra|
   |Region|o nome de uma região do Azure que hospeda o ambiente de laboratório da Área de Trabalho Virtual do Azure|
   |Desempenho|**Standard**|
   |Redundância|**Armazenamento com redundância geográfica (GRS)**|
   |Disponibilizar o acesso de leitura de dados no caso de indisponibilidade regional|Habilitado|

   >**Observação**: Garanta que o comprimento do nome da conta de armazenamento não exceda 15 caracteres. O nome será usado para criar uma conta de computador no domínio Active Directory Domain Services (AD DS) integrado ao locatário do Microsoft Entra associado à assinatura do Azure que contém a conta de armazenamento. Isso permitirá a autenticação baseada em AD DS ao acessar compartilhamentos de arquivos hospedados nessa conta de armazenamento.

1. Na guia **Básico** da folha **Criar conta de armazenamento**, selecione **Examinar + Criar**, aguarde a conclusão do processo de validação e selecione **Criar**.

   >**Observação**: aguarde até a conta de armazenamento ser criada. Isso deverá levar cerca de dois minutos.

#### Tarefa 2: Criar um compartilhamento dos Arquivos do Azure

1. Na sessão do Bastion para **az140-dc-vm11**, na janela do Microsoft Edge que exibe o portal do Azure, navegue de volta para a folha **Contas de Armazenamento** e selecione a entrada que representa a conta de armazenamento recém-criada.
1. Na folha da conta de armazenamento, na seção **Armazenamento de dados**, selecione **Compartilhamento de arquivo** e, em seguida, selecione **+ Compartilhamento de arquivo**.
1. Na folha **Novo compartilhamento de arquivos**, especifique as seguintes configurações e selecione **Avançar: Backup >** (deixe outras configurações com seus valores padrão):

   |Configuração|Valor|
   |---|---|
   |Nome|**az140-22-profiles**|
   |Camada de acesso|**Transações otimizadas**|

1. Na folha **Backup**, desmarque a **caixa de seleção Habilitar backup**, selecione **Examinar + Criar**, aguarde a conclusão do processo de validação e selecione **Criar**.

#### Tarefa 3: Habilitar a autenticação do AD DS para a conta de Armazenamento do Microsoft Azure 

1. Na sessão do Bastion para **az140-dc-vm11**, abra outra guia na janela do Microsoft Edge, navegue até o [repositório GitHub de exemplos dos Arquivos do Azure](https://github.com/Azure-Samples/azure-files-samples/releases), baixe [a versão mais recente do módulo do PowerShell **AzFilesHybrid.zip** compactado e extraia seu conteúdo na pasta **C:\\Allfiles\\Labs\\02** (crie a pasta, se necessário).
1. Na sessão do Bastion para **az140-dc-vm11**, inicie o **ISE do Windows PowerShell** como administrador e, no **Administrador: ISE do Windows PowerShell**, execute o seguinte para remover o fluxo de dados alternativo **Zone.Identifier**, que tem um valor de **3**, indicando que ele foi baixado da Internet:

   ```powershell
   Get-ChildItem -Path C:\Allfiles\Labs\02 -File -Recurse | Unblock-File
   ```

1. Na sessão do Bastion para **az140-dc-vm11**, do **Administrador: ISE do Windows PowerShell**, execute o seguinte para definir as variáveis necessárias para executar o script subsequente:

   ```powershell
   $subscriptionId = (Get-AzContext).Subscription.Id
   $resourceGroupName = 'az140-22-RG'
   $storageAccountName = (Get-AzStorageAccount -ResourceGroupName $resourceGroupName)[0].StorageAccountName
   ```

1. Na sessão do Bastion para **az140-dc-vm11**, do **Administrador: ISE do Windows PowerShell**, execute o seguinte para criar um objeto de computador do AD DS que representa a conta de Armazenamento do Azure criada anteriormente nesta tarefa, usada para implementar sua autenticação do AD DS:

   >**Observação**: Caso receba um erro ao executar esse bloco de script, verifique se você está no mesmo diretório que o script do CopyToPSPath.ps1. Dependendo de como os arquivos foram extraídos anteriormente neste laboratório, eles podem estar em uma subpasta chamada AzFilesHybrid. No contexto do PowerShell, altere os diretórios para a pasta usando **cd AzFilesHybrid**.

   ```powershell
   Set-Location -Path 'C:\Allfiles\Labs\02'
   .\CopyToPSPath.ps1 
   Import-Module -Name AzFilesHybrid
   Join-AzStorageAccountForAuth `
      -ResourceGroupName $ResourceGroupName `
      -StorageAccountName $StorageAccountName `
      -DomainAccountType 'ComputerAccount' `
      -OrganizationalUnitDistinguishedName 'OU=WVDInfra,DC=adatum,DC=com'
   ```

1. Na sessão do Bastion para **az140-dc-vm11**, do **Administrador: ISE do Windows PowerShell**, execute o seguinte para verificar se a autenticação do AD DS está habilitada na conta de Armazenamento do Microsoft Azure:

   ```powershell
   $storageaccount = Get-AzStorageAccount -ResourceGroupName $resourceGroupName -Name $storageAccountName
   $storageAccount.AzureFilesIdentityBasedAuth.ActiveDirectoryProperties
   $storageAccount.AzureFilesIdentityBasedAuth.DirectoryServiceOptions
   ```

1. Verifique se a saída do comando `$storageAccount.AzureFilesIdentityBasedAuth.ActiveDirectoryProperties` retorna `AD`, representando o serviço de diretório da conta de armazenamento e se a saída do comando `$storageAccount.AzureFilesIdentityBasedAuth.DirectoryServiceOptions`, representando as informações de domínio do diretório, se assemelha ao seguinte formato (os valores de `DomainGuid`, `DomainSid` e `AzureStorageSid` serão diferentes):

   ```
   DomainName        : adatum.com
   NetBiosDomainName : adatum.com
   ForestName        : adatum.com
   DomainGuid        : 47c93969-9b12-4e01-ab81-1508cae3ddc8
   DomainSid         : S-1-5-21-1102940778-2483248400-1820931179
   AzureStorageSid   : S-1-5-21-1102940778-2483248400-1820931179-2109
   ```

1. Na sessão do Bastion para **az140-dc-vm11**, alterne para a janela do Microsoft Edge exibindo o portal do Azure, na folha que exibe a conta de armazenamento, selecione **Compartilhamentos de arquivos** e verifique se a **configuração de acesso baseado em identidade** está **configurada**.

   >**Observação**: Talvez seja necessário atualizar a página do navegador para que a alteração seja refletida no portal do Azure.

#### Tarefa 4: Configurar as permissões baseadas em RBAC dos Arquivos do Azure

1. Na sessão do Bastion para **az140-dc-vm11**, na janela do Microsoft Edge que exibe o portal do Azure, na folha que exibe as propriedades da conta de armazenamento que você criou anteriormente neste exercício, no menu vertical à esquerda, na seção **Armazenamento de Dados**, selecione **Compartilhamentos de arquivos**.
1. Na folha **Compartilhamento de arquivo**, na lista de compartilhamentos, selecione a entrada **az140-22-profiles**.
1. Na folha **az140-22-profiles**, no menu vertical no lado esquerdo, selecione **Controle de Acesso (IAM)**.
1. Na folha **Controle de acesso (IAM)** da conta de armazenamento, selecione **+ Adicionar** e, no menu suspenso, selecione **Adicionar atribuição de função**. 
1. Na folha **Adicionar atribuição** de função, na guia **Função**, especifique as seguintes configurações e selecione **Avançar**:

   |Configuração|Valor|
   |---|---|
   |Funções de trabalho|**Colaborador de compartilhamento SMB de dados de arquivo de armazenamento**|

1. Na folha **Adicionar atribuição** de função, na **guia Membros**, clique **em + Selecionar membros**, especifique as configurações a seguir e clique em **Selecionar**. 

   |Configuração|Valor|
   |---|---|
   |Selecionar|**az140-wvd-users**|
1. Na folha **Adicionar atribuição** de função, selecione **Examinar + atribuir**e, em seguida, selecione **Examinar + atribuir**.
1. Na folha **Controle de acesso (IAM)** da conta de armazenamento, selecione **+ Adicionar** e, no menu suspenso, selecione **Adicionar atribuição de função**, 
1. Na folha **Adicionar atribuição** de função, na guia **Função**, especifique as seguintes configurações e selecione **Avançar**:

   |Configuração|Valor|
   |---|---|
   |Funções de trabalho|**Colaborador elevado de compartilhamento SMB de dados de arquivo de armazenamento**|

1. Na folha **Adicionar atribuição** de função, na **guia Membros**, clique **em + Selecionar membros**, especifique as configurações a seguir e clique em **Selecionar**. 

   |Configuração|Valor|
   |---|---|
   |Selecionar|**az140-wvd-admins**|
1. Na folha **Adicionar atribuição** de função, selecione **Examinar + atribuir**e, em seguida, selecione **Examinar + atribuir**.

#### Tarefa 5: Configurar as permissões do sistema de arquivos dos Arquivos do Azure

1. Na sessão do Bastion para **az140-dc-vm11**, alterne para o **Administrador: Janela ISE do Windows PowerShell** e do **Administrador: ISE do Windows PowerShell**, execute o seguinte para criar uma variável que referencie o nome e a chave da conta de armazenamento que você criou anteriormente neste exercício:

   ```powershell
   $resourceGroupName = 'az140-22-RG'
   $storageAccount = (Get-AzStorageAccount -ResourceGroupName $resourceGroupName)[0]
   $storageAccountName = $storageAccount.StorageAccountName
   $storageAccountKey = (Get-AzStorageAccountKey -ResourceGroupName $resourceGroupName -Name $storageAccountName).Value[0]
   ```

1. No painel de script **Administrador: ISE do Windows PowerShell**, execute o seguinte para criar um mapeamento de unidade para o compartilhamento de arquivos criado anteriormente neste exercício:

   ```powershell
   $fileShareName = 'az140-22-profiles'
   net use Z: "\\$storageAccountName.file.core.windows.net\$fileShareName" /u:AZURE\$storageAccountName $storageAccountKey
   ```

1. No console **Administrador: ISE do Windows PowerShell**, execute o seguinte para exibir as permissões atuais do sistema de arquivos:

   ```powershell
   icacls Z:
   ```

   >**Observação**: Por padrão, **usuários autenticados\\da Autoridade NT** e **usuários do **BUILTIN\\, têm permissões que concedem aos usuários a leitura de contêineres de perfil de outros usuários. Você as removerá e adicionará permissões mínimas necessárias.

1. No painel de script **Administrador: ISE do Windows PowerShell**, execute o seguinte a fim de ajustar as permissões do sistema de arquivos para cumprir o princípio de privilégio mínimo:

   ```powershell
   $permissions = 'ADATUM\az140-wvd-admins'+':(F)'
   cmd /c icacls Z: /grant $permissions
   $permissions = 'ADATUM\az140-wvd-users'+':(M)'
   cmd /c icacls Z: /grant $permissions
   $permissions = 'Creator Owner'+':(OI)(CI)(IO)(M)'
   cmd /c icacls Z: /grant $permissions
   icacls Z: /remove 'Authenticated Users'
   icacls Z: /remove 'Builtin\Users'
   ```

   >**Observação**: Como alternativa, é possível definir permissões usando o Explorador de Arquivos.
