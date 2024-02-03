---
lab:
  title: 'Laboratório: Implantar pools de hosts e hosts de sessão usando o portal do Azure (AD DS)'
  module: 'Module 2: Implement an AVD Infrastructure'
---

# Laboratório: Implantar pools de hosts e hosts de sessão usando o portal do Azure (AD DS)
# Manual de laboratório do aluno

## Dependências do laboratório

- O nome da assinatura do Azure que você usará nesse laboratório
- Uma conta Microsoft ou uma conta do Microsoft Entra com a função de Proprietário ou Colaborador na assinatura do Azure que você usará neste laboratório e com a função de Administrador Global no locatário do Microsoft Entra associado a essa assinatura do Azure.
- O laboratório concluído: **Preparar para a implantação da Área de Trabalho Virtual do Azure (AD DS)**

## Tempo estimado

60 minutos

## Cenário do laboratório

Você precisa criar e configurar pools de host e hosts de sessão em um ambiente do Active Directory Domain Services (AD DS).

## Objetivos
  
Depois de realizar este laboratório, você será capaz de:

- Implementar um ambiente da Área de Trabalho Virtual do Azure em um domínio do AD DS
- Validar um ambiente da Área de Trabalho Virtual do Azure em um domínio do AD DS

## Arquivos do laboratório

- Nenhum

## Instruções

### Exercício 1: Implementar um ambiente da Área de Trabalho Virtual do Azure em um domínio do AD DS
  
As principais tarefas desse exercício são as seguintes:

1. Preparar o domínio do AD DS e a assinatura do Azure para implantação de um pool de hosts da Área de Trabalho Virtual do Azure
1. Configurar uma pool de host da Área de Trabalho Virtual do Azure
1. Gerenciar os hosts de sessão do pool de host da Área de Trabalho Virtual do Azure
1. Configurar grupos de aplicativos da Área de Trabalho Virtual do Azure
1. Configurar workspaces da Área de Trabalho Virtual do Azure

#### Tarefa 1: Preparar o domínio do AD DS e a assinatura do Azure para implantação de um pool de hosts da Área de Trabalho Virtual do Azure

1. No seu computador de laboratório, inicie um navegador da Web, navegue até o [portal do Azure]( ) e entre fornecendo credenciais de uma conta de usuário com a função Proprietário na assinatura que você usará nesse laboratório.
1. No portal do Azure, pesquise e selecione **Máquinas Virtuais** e, na folha **Máquinas Virtuais**, selecione **az140-dc-vm11**.
1. Na folha **az140-dc-vm11**, selecione **Conectar**, no menu suspenso, selecione **Bastion**, na guia **Bastion** da folha **Conectar \|az140-dc-vm11**, selecione **Usar o Bastion**.
1. Quando solicitado, forneça as seguintes credenciais e selecione **Conectar**:

   |Configuração|Valor|
   |---|---|
   |Nome do usuário|**Aluno**|
   |Senha|**Pa55w.rd1234**|

1. Na sessão da Área de Trabalho Remota para **az140-dc-vm11**, inicie o **ISE do Windows PowerShell** como administrador.
1. Na sessão da Área de Trabalho Remota para **az140-dc-vm11**, a partir do **Administrador: Console ISE do Windows PowerShell**: execute o seguinte para criar uma unidade organizacional que hospedará os objetos de computador dos hosts da Área de Trabalho Virtual do Azure:

   ```powershell
   New-ADOrganizationalUnit 'WVDInfra' –path 'DC=adatum,DC=com' -ProtectedFromAccidentalDeletion $false
   ```

1. Do **Administrador: Console do ISE do Windows PowerShell**: execute o seguinte para entrar em sua assinatura do Azure:

   ```powershell
   Connect-AzAccount
   ```

1. Quando solicitado, forneça as credenciais da conta de usuário com a função Proprietário na assinatura que você está usando neste laboratório.
1. Do **Administrador: Console ISE do Windows PowerShell**: execute o seguinte para identificar o nome principal do usuário da conta **aduser1**

   ```powershell
   (Get-AzADUser -DisplayName 'aduser1').UserPrincipalName
   ```

   > **Observação**: Registre o nome da entidade de segurança do usuário que você identificou nessa etapa. Você precisará disso em uma etapa posterior deste laboratório.

1. Do **Administrador: Console ISE do Windows PowerShell**: execute o seguinte para registrar o provedor de recursos **Microsoft.DesktopVirtualization**:

   ```powershell
   Register-AzResourceProvider -ProviderNamespace Microsoft.DesktopVirtualization
   ```

1. Na sessão da Área de Trabalho Remota para **az140-dc-vm11**, inicie o Microsoft Edge e navegue até o [portal do Azure](https://portal.azure.com). Se solicitado, entre usando as credenciais do Microsoft Entra da conta de usuário com a função Proprietário na assinatura que você está usando nesse laboratório.
1. Na sessão do Bastion para **az140-dc-vm11**, no portal do Azure, use a caixa de texto **Pesquisar recursos, serviços e documentos** na parte superior da página do portal do Azure para pesquisar e navegar até **Redes virtuais** e, na folha **Redes virtuais**, selecione **az140-adds-vnet11**. 
1. Na folha **az140-adds-vnet11**, selecione **Sub-redes**, na folha **Sub-redes**, selecione **+ Sub-rede**, na folha **Adicionar sub-rede**, especifique as seguintes configurações (deixe todas as outras configurações com seus valores padrão) e clique em **Salvar**:

   |Configuração|Valor|
   |---|---|
   |Nome|**hp1-Subnet**|
   |Intervalo de endereços da sub-rede|**10.0.1.0/24**|

#### Tarefa 2: Configurar uma pool de host da Área de Trabalho Virtual do Azure

1. Na sessão da Área de Trabalho Remota para **az140-dc-vm11**, na janela do Microsoft Edge que exibe o portal do Azure, pesquise e selecione **Área de Trabalho Virtual do Azure**. Na folha **Área de Trabalho Virtual do Azure**, selecione **Pools de Host** e, na folha **Pools de Host\| da Área de Trabalho Virtual do Azure**, selecione **+ Criar**. 
1. Na guia **Noções Básicas** da folha **Criar um pool de host**, especifique as seguintes configurações e selecione **Avançar: Máquinas Virtuais >** (deixe as outras configurações com seus valores padrão):

   |Configuração|Valor|
   |---|---|
   |Assinatura|o nome da assinatura do Azure que você está usando neste laboratório|
   |Grupo de recursos|O nome de um novo grupo de recursos **az140-21-RG**|
   |Nome do pool de host|**az140-21-hp1**|
   |Localidade|o nome da região do Azure na qual você implantou recursos no primeiro exercício desse laboratório ou em uma região próxima a ele |
   |Ambiente de validação|**Não**|
   |Tipo de grupo de aplicativos preferencial|**Desktop**|
   |Tipo de pool de host|**Em pool**|
   |Algoritmo de balanceamento de carga|**Amplitude**|
   |Limite máximo da sessão|**12**|

1. Na guia **Máquinas Virtuais** da folha **Criar um pool de hosts**, especifique as seguintes configurações e selecione **Avançar: Workspace >** (deixe as outras configurações com seus valores padrão):

   |Configuração|Valor|
   |---|---|
   |Adicionar máquinas virtuais do Azure|**Sim**|
   |Grupo de recursos|**O padrão é o mesmo que o pool de hosts**|
   |Prefixo do nome|**az140-21-p1**|
   |Localização da máquina virtual|o nome da região do Azure na qual você implantou recursos no primeiro exercício deste laboratório|
   |Opções de disponibilidade|**Nenhuma redundância de infraestrutura necessária**|
   |Tipo de segurança|**Standard**|
   |Imagem|**Windows 11 Enterprise multissessão + Microsoft 365 Apps, versão 22H2**|
   |Tamanho da máquina virtual|**Standard D2s v3**|
   |Número de VMs|**2**|
   |Tipo de disco de SO|**SSD Standard**|
   |Diagnóstico de Inicialização|**Habilitar com a conta de armazenamento gerenciada (recomendado)**|
   |Rede virtual|**az140-adds-vnet11**|
   |Sub-rede|**hp1-Subnet (10.0.1.0/24)**|
   |Grupo de segurança de rede|**Basic**|
   |Portas de entrada públicas|**Não**|
   |Selecione o diretório ao qual deseja ingressar|**Active Directory**|
   |UPN de ingresso no domínio do AD|**student@adatum.com**|
   |Senha|**Pa55w.rd1234**|
   |Especificar o domínio ou a unidade|**Sim**|
   |Domínio a ingressar|**adatum.com**|
   |Caminho da Unidade Organizacional|**OU=WVDInfra,DC=adatum,DC=com**|
   |Nome de usuário|**Aluno**|
   |Senha|**Pa55w.rd1234**|
   |Confirmar senha|**Pa55w.rd1234**|

1. Na guia **Workspace** da folha **Criar um pool de hosts**, especifique as seguintes configurações e selecione **Examinar + criar**:

   |Configuração|Valor|
   |---|---|
   |Registre o grupo de aplicativos da área de trabalho|**Não**|

1. Na guia **Revisar + criar** da folha **Criar um pool de host**, selecione **Criar**.

   > **Observação**: aguarde até que a implantação seja concluída. Isso pode levar cerca de 10 minutos.

#### Tarefa 3: Gerenciar os hosts de sessão do pool de host da Área de Trabalho Virtual do Azure

1. Na sessão da Área de Trabalho Remota para **az140-dc-vm11**, na janela do navegador da Web que exibe o portal do Azure, pesquise e selecione **Área de Trabalho Virtual do Azure** e, na folha **Área de Trabalho Virtual do Azure**, na barra de menus vertical, na **seção Gerenciar**, selecione **Pools de host**.
1. Na folha **Pools de host\| da Área de Trabalho Virtual do Azure**, na lista de pools de hosts, selecione **az140-21-hp1**.
1. Na folha **az140-21-hp1**, na barra de menus vertical, na **seção Gerenciar**, selecione **Hosts de sessão** e verifique se o pool consiste em dois hosts. 
1. Na folha **Hosts de sessão\|az140-21-hp1**, selecione **+ Adicionar**.
1. Na guia **Noções Básicas** da folha **Adicionar máquinas virtuais a um pool de hosts**, examine as configurações pré-configuradas e selecione **Avançar: Máquinas Virtuais**.
1. Na guia **Máquinas Virtuais** da folha **Adicionar máquinas virtuais a um pool** de hosts, especifique as seguintes configurações e selecione **Revisar + criar** (deixe as outras com suas configurações padrão):

   |Configuração|Valor|
   |---|---|
   |Grupo de recursos|**az140-21-RG**|
   |Prefixo do nome|**az140-21-p1**|
   |Localização da máquina virtual|o nome da região do Azure na qual você implantou as duas primeiras VMs de host de sessão|
   |Opções de disponibilidade|**Nenhuma redundância de infraestrutura necessária**|
   |Tipo de segurança|**Standard**|
   |Imagem|**Windows 11 Enterprise multissessão + Microsoft 365 Apps, versão 22H2**|
   |Número de VMs|**1**|
   |Tipo de disco de SO|**SSD Standard**|
   |Diagnóstico de Inicialização|**Habilitar com a conta de armazenamento gerenciada (recomendado)**|
   |Rede virtual|**az140-adds-vnet11**|
   |Sub-rede|**hp1-Subnet (10.0.1.0/24)**|
   |Grupo de segurança de rede|**Basic**|
   |Portas de entrada públicas|**Não**|
   |UPN de ingresso no domínio do AD|**student@adatum.com**|
   |Senha|**Pa55w.rd1234**|
   |Especificar o domínio ou a unidade|**Sim**|
   |Domínio a ingressar|**adatum.com**|
   |Caminho da Unidade Organizacional|**OU=WVDInfra,DC=adatum,DC=com**|   
   |Nome de usuário da Conta de administrador da Máquina Virtual|**Aluno**|
   |senha da Conta de administrador da Máquina Virtual|**Pa55w.rd1234**|

   > **Observação**: Como você provavelmente notou, é possível alterar a imagem e o prefixo das VMs à medida que você adiciona hosts de sessão ao pool existente. Em geral, isso não é recomendado, a menos que você planeje substituir todas as VMs no pool. 

1. Na guia **Revisar + criar** da folha **Adicionar máquinas virtuais a um pool** de hosts, selecione **Criar**

   > **Observação**: Aguarde a conclusão da implantação antes de prosseguir para a próxima tarefa. A implantação pode levar cerca de cinco minutos. 

#### Tarefa 4: Configurar grupos de aplicativos da Área de Trabalho Virtual do Azure

1. Na sessão da Área de Trabalho Remota para **az140-dc-vm11**, na janela do navegador da Web que exibe o portal do Azure, pesquise e selecione **Área de Trabalho Virtual do Azure** e, na folha **Área de Trabalho Virtual do Azure**, selecione **Grupos de aplicativos**.
1. Na folha **Grupos de Aplicativos \|da Área de Trabalho Virtual do Azure**, observe o grupo de aplicativos de área de trabalho existente e gerado automaticamente **az140-21-hp1-DAG** e selecione-o. 
1. Na folha **az140-21-hp1-DAG**, selecione **Atribuições**.
1. Na folha **az140-21-hp1-DAG \| Atribuições**, selecione** + Adicionar **.
1. Na folha **Selecionar usuários ou grupos de usuários do Microsoft Entra**, selecione **z140-wvd-pooled** e clique em **Selecionar**.
1. Navegue de volta para a folha **Grupos de Aplicativos\| da Área de Trabalho Virtual do Azure**, selecione **+ Criar**. 
1. Na guia **Noções básicas** da folha **Criar um grupo de aplicativos**, especifique as seguintes configurações e selecione **Avançar: Aplicativos > **:

   |Configuração|Valor|
   |---|---|
   |Assinatura|o nome da assinatura do Azure que você está usando neste laboratório|
   |Grupo de recursos|**az140-21-RG**|
   |Pool de host|**az140-21-hp1**|
   |Tipo de grupo de aplicativos|**Aplicativo Remoto (RAIL)**|
   |Nome do grupo de aplicativos|**az140-21-hp1-Office365-RAG**|

1. Na guia **Aplicativos** da folha **Criar um grupo de aplicativos**, selecione **+ Adicionar aplicativos**.
1. Na folha **Adicionar aplicativo**, especifique as seguintes configurações e selecione **Salvar**:

   |Configuração|Valor|
   |---|---|
   |Origem do aplicativo|**Menu Iniciar**|
   |Aplicativo|**Word**|
   |Descrição|**Microsoft Word**|
   |Exigir linha de comando|**Não**|

1. De volta à guia **Aplicativos** da folha **Criar um grupo de aplicativos,** selecione **+ Adicionar aplicativos**.
1. Na folha **Adicionar aplicativo**, especifique as seguintes configurações e selecione **Salvar**:

   |Configuração|Valor|
   |---|---|
   |Origem do aplicativo|**Menu Iniciar**|
   |Aplicativo|**Excel**|
   |Descrição|**Microsoft Excel**|
   |Exigir linha de comando|**Não**|

1. De volta à guia **Aplicativos** da folha **Criar um grupo de aplicativos,** selecione **+ Adicionar aplicativos**.
1. Na folha **Adicionar aplicativo**, especifique as seguintes configurações e selecione **Salvar**:

   |Configuração|Valor|
   |---|---|
   |Origem do aplicativo|**Menu Iniciar**|
   |Aplicativo|**PowerPoint**|
   |Descrição|**Microsoft PowerPoint**|
   |Exigir linha de comando|**Não**|

1. De volta à guia **Aplicativos** da folha **Criar um grupo de aplicativos**, selecione **Avançar: Atribuições >**.
1. Na guia **Atribuições** da folha **Criar um grupo de aplicativos,** selecione **+ Adicionar usuários ou grupos de usuários do Microsoft Entra**.
1. Na folha **Selecionar usuários ou grupos de usuários do Microsoft Entra**, selecione **az140-wvd-remote-app** e clique em **Selecionar**.
1. De volta à guia **Atribuições** da folha **Criar um grupo** de aplicativos, selecione **Avançar: Workspace >**.
1. Na guia **Workspace** da folha **Criar um workspace**, especifique a seguinte configuração e selecione **Examinar + criar**:

   |Configuração|Valor|
   |---|---|
   |Registrar o grupo de aplicativos|**Não**|

1. Na guia **Revisar + criar** da folha **Criar um grupo** de aplicativos, selecione **Criar**.

   > **Observação**: aguarde até que o Grupo de Aplicativos seja criado. Isso deverá levar menos de 1 minuto. 

   > **Observação**: Em seguida, você criará um grupo de aplicativos com base no caminho do arquivo como a origem do aplicativo.

1. Na sessão da Área de Trabalho Remota para **az140-dc-vm11**, pesquise e selecione **Área de Trabalho Virtual do Azure** e, na folha **Área de Trabalho Virtual do Azure**, selecione **Grupos de aplicativos**.
1. Na folha **Grupos de Aplicativos \| da Área de Trabalho Virtual do Azure**, selecione **+ Criar**. 
1. Na guia **Noções básicas** da folha **Criar um grupo de aplicativos**, especifique as seguintes configurações e selecione **Avançar: Aplicativos >**:

   |Configuração|Valor|
   |---|---|
   |Assinatura|o nome da assinatura do Azure que você está usando neste laboratório|
   |Grupo de recursos|**az140-21-RG**|
   |Pool de host|**az140-21-hp1**|
   |Tipo de grupo de aplicativos|**RemoteApp (RAIL)**|
   |Nome do grupo de aplicativos|**az140-21-hp1-Utilities-RAG**|

1. Na guia **Aplicativos** da folha **Criar um grupo de aplicativos**, selecione **+ Adicionar aplicativos**.
1. Na folha **Adicionar aplicativo**, especifique as seguintes configurações e selecione **Salvar**:

   |Configuração|Valor|
   |---|---|
   |Origem do aplicativo|**Caminho do arquivo**|
   |Caminho do aplicativo|**C:\Windows\system32\cmd.exe**|
   |Nome do aplicativo|**Prompt de comando**|
   |Nome de exibição|**Prompt de comando**|
   |Caminho do ícone|**C:\Windows\system32\cmd.exe**|
   |Índice do ícone|**0**|
   |Descrição|**Windows Command Prompt**|
   |Exigir linha de comando|**Não**|

1. De volta à guia **Aplicativos** da folha **Criar um grupo de aplicativos**, selecione **Avançar: Atribuições >**.
1. Na guia **Atribuições** da folha **Criar um grupo de aplicativos,** selecione **+ Adicionar usuários ou grupos de usuários do Microsoft Entra**.
1. Na folha **Selecionar usuários ou grupos de usuários do Microsoft Entra**, selecione **az140-wvd-remote-app** e **az140-wvd-admins** e clique em**Selecionar **.
1. De volta à guia **Atribuições** da folha **Criar um grupo** de aplicativos, selecione **Avançar: Workspace >**.
1. Na guia **Workspace** da folha **Criar um workspace**, especifique a seguinte configuração e selecione **Examinar + criar**:

   |Configuração|Valor|
   |---|---|
   |Registrar o grupo de aplicativos|**Não**|

1. Na guia **Revisar + criar** da folha **Criar um grupo** de aplicativos, selecione **Criar**.

#### Tarefa 5: Configurar workspaces da Área de Trabalho Virtual do Azure

1. Na sessão da Área de Trabalho Remota para **az140-dc-vm11**, na janela do Microsoft Edge que exibe o portal do Azure, pesquise e selecione **Área de Trabalho Virtual do Azure** e, na folha **Área de Trabalho Virtual do Azure**, selecione **Workspaces**.
1. Na folha **Workspaces\| da Área de Trabalho Virtual do Azure**, selecione **+ Criar**. 
1. Na guia **Noções básicas** da folha **Criar um workspace**, especifique as seguintes configurações e selecione **Avançar: Grupos de aplicativos >**:

   |Configuração|Valor|
   |---|---|
   |Assinatura|o nome da assinatura do Azure que você está usando neste laboratório|
   |Grupo de recursos|**az140-21-RG**|
   |Nome do workspace|**az140-21-ws1**|
   |Nome amigável|**az140-21-ws1**|
   |Localidade|o nome da região do Azure na qual você implantou recursos no primeiro exercício desse laboratório ou em uma região próxima a ele|

1. Na guia **Grupos de aplicativos** da folha **Criar um workspace**, especifique as seguintes configurações:

   |Configuração|Valor|
   |---|---|
   |Registrar grupos de aplicativos|**Sim**|

1. Na guia **Workspace** da folha **Criar um workspace**, selecione** + Registrar grupos de aplicativos **.
1. Na folha **Adicionar grupos de aplicativos**, selecione o sinal de adição ao lado das entradas a**az140-21-hp1-DAG**, **az140-21-hp1-Office365-RAG** e **az140-21-hp1-Utilities-RAG** e clique em **Selecionar**. 
1. De volta à guia **Grupos de aplicativos** da folha **Criar um workspace**, selecione **Examinar + criar**.
1. Na guia **Revisar + criar** da folha **Criar um workspace**, selecione **Criar**.

### Exercício 2: Validar o ambiente da Área de Trabalho Virtual do Azure
  
As principais tarefas desse exercício são as seguintes:

1. Instalar o cliente de Área de Trabalho Remota (MSRDC) da Microsoft em um computador Windows 10
1. Assinar um workspace da Área de Trabalho Virtual do Azure
1. Testar aplicativos da Área de Trabalho Virtual do Azure

#### Tarefa 1: Instalar o cliente da Área de Trabalho Remota (MSRDC) da Microsoft em um computador Windows 10

1. Na sessão da Área de Trabalho Remota para **az140-dc-vm11**, na janela do navegador que exibe o portal do Azure, pesquise e selecione **Máquinas virtuais** e, na folha **Máquinas virtuais**, selecione a entrada **az140-cl-vm11**.
1. Na folha **az140-cl-vm11**, role para baixo até a seção **Operações** e selecione **Executar Comando**. 
1. Na folha **az140-cl-vm11 \| Executar comando**, selecione ** EnableRemotePS** e selecione **Executar **. 

   > **Observação**: Aguarde até que o comando seja concluído antes de prosseguir para a próxima etapa. Isso pode levar cerca de um minuto. Você pode receber erros de texto em vermelho que tratam do Perfil público que está sendo usado e não do perfil de domínio, se assim for, você pode ignorar e ir para a próxima etapa.

1. Na sessão da Área de Trabalho Remota para **az140-dc-vm11**, a partir do **Administrador: Painel de script ISE do Windows PowerShell**: execute o seguinte para adicionar todos os membros do **ADATUM\\az140-wvd-users** ao grupo de **Usuários da Área de Trabalho Remota** local na VM do Azure **az140-cl-vm11** que está executando o Windows 10 que você implantou no laboratório **Preparar para implantação da Área de Trabalho Virtual do Azure (AD DS)**.

   ```powershell
   $computerName = 'az140-cl-vm11'
   Invoke-Command -ComputerName $computerName -ScriptBlock {Add-LocalGroupMember -Group 'Remote Desktop Users' -Member 'ADATUM\az140-wvd-users'}
   ```

1. Alterne para o seu computador de laboratório, no computador do laboratório, na janela do navegador que exibe o portal do Azure, pesquise e selecione **Máquinas virtuais** e, na folha **Máquinas virtuais**, selecione a entrada **az140-cl-vm11**.
1. Na folha **az140-cl-vm11**, selecione **Conectar**, no menu suspenso, selecione **Bastion**, na guia **Bastion** da folha **az140-cl-vm11 \| Conectar**, selecione **Usar Bastion**.
1. Quando solicitado, forneça as seguintes credenciais e selecione **Conectar**:

   |Configuração|Valor|
   |---|---|
   |Nome do usuário|**Student@adatum.com**|
   |Senha|**Pa55w.rd1234**|

1. Na sessão da Área de Trabalho Remota para **az140-cl-vm11**, inicie o Microsoft Edge e navegue até a [página de download do cliente da Área de Trabalho do Windows](https://go.microsoft.com/fwlink/?linkid=2068602) e, quando solicitado, selecione **Executar** para iniciar sua instalação. Na página **Escopo de Instalação** do assistente de **Instalação da Área de Trabalho Remota**, selecione a opção **Instalar para todos os usuários desse computador** e clique em **Instalar**. Se solicitado pelo Controle de Conta de Usuário para credenciais administrativas, autentique usando o nome de usuário do **Aluno\\do ADATUM** com a senha** Pa55w.rd1234**.
1. Assim que a instalação for concluída, certifique-se de que a caixa de seleção **Iniciar Área de Trabalho Remota quando a instalação for encerrada** esteja marcada e clique em **Concluir** para iniciar o cliente de Área de Trabalho Remota.

#### Tarefa 2: Assinar um workspace da Área de Trabalho Virtual do Azure

1. Na janela cliente da **Área de Trabalho Remota**, selecione **Assinar** e, quando solicitado, entre com as credenciais **aduser1** fornecendo seu userPrincipalName e a senha definida ao criar essa conta de usuário.

   > **Observação**: Como alternativa, na janela cliente da **Área de Trabalho Remota**, selecione **Assinar com a URL**, no painel **Assinar um Workspace**. Na **URL de Email ou Workspace**, digite **https://rdweb.wvd.microsoft.com/api/arm/feeddiscovery**, selecione **Avançar** e, uma vez solicitado, entre com as credenciais do **aduser1** (usando seu atributo userPrincipalName como o nome de usuário e a senha definida ao criar essa conta). 

1. Se solicitado, na janela **Permanecer conectado a todos os seus aplicativos**, desmarque a caixa de seleção **Permitir que minha organização gerencie meu dispositivo** e selecione **Não, entre somente neste aplicativo**. 
1. Verifique se a página **Área de trabalho remota** exibe a listagem de aplicativos incluídos nos grupos de aplicativos publicados no workspace e associados à conta de usuário **aduser1** por meio de sua associação de grupo. 

#### Tarefa 3: Testar aplicativos da Área de Trabalho Virtual do Azure

1. Na sessão da Área de Trabalho Remota para **az140-cl-vm11**, na janela cliente da **Área de Trabalho Remota**, na lista de aplicativos, clique duas vezes no **Prompt de Comando** e verifique se ele inicia uma janela do **Prompt de Comando**. Quando solicitado a autenticar, digite a senha definida ao criar a conta de usuário aduser1 **, marque a caixa **de seleção **Lembrar de mim** e selecione **OK.**

   > **Observação**: Inicialmente, pode levar alguns minutos para o aplicativo ser iniciado, mas, posteriormente, a inicialização do aplicativo deve ser muito mais rápida.

   > **Observação**: Se você receber o prompt de entrada **Boas-vindas ao Microsoft Teams**, feche-o.

1. No Prompt de Comando, digite **hostname** e pressione a tecla **Enter** para exibir o nome do computador no qual o Prompt de Comando está em execução.

   > **Observação**: Verifique se o nome exibido é **az140-21-p1-0**, **az140-21-p1-1** or **az140-21-p1-2** em vez de **az140-cl-vm11**.

1. No Prompt de Comando, digite **logoff** e pressione a tecla **Enter** para sair da sessão de Aplicativo Remoto atual.
1. Na sessão do Bastion para **az140-cl-vm11**, na janela do cliente da **Área de Trabalho Remota**, na lista de aplicativos, clique duas vezes em **SessionDesktop** e verifique se uma sessão da Área de Trabalho Remota é iniciada. 
1. Na sessão **Área de Trabalho Padrão**, clique com o botão direito do mouse em **Iniciar**, selecione**Executar **, na caixa de texto **Abrir** da caixa de diálogo **Executar**, digite **cmd** e selecione**OK**. 
1. Na sessão da **Área de Trabalho Padrão**, no Prompt de Comando, digite **hostname** e pressione a tecla **Enter** para exibir o nome do computador no qual a sessão da Área de Trabalho Remota está em execução.
1. Verifique se o nome exibido é **az140-21-p1-0**, **az140-21-p1-1** ou **az140-21-p1-2**.

### Exercício 3: Parar e desalocar VMs do Azure provisionadas no laboratório

As principais tarefas desse exercício são as seguintes:

1. Parar e desalocar VMs do Azure provisionadas no laboratório

>**Observação**: Nesse exercício, você desalocará as VMs do Azure provisionadas neste laboratório para minimizar os encargos de computação correspondentes

#### Tarefa 1: Desalocar VMs do Azure provisionadas no laboratório

1. Alterne para o computador de laboratório e, na janela do navegador da Web que exibe o portal do Azure, abra a sessão do Shell do **PowerShell** no painel do **Cloud Shell**.
1. Na sessão do PowerShell no painel do Cloud Shell, execute o seguinte para listar todas as VMs do Azure criadas neste laboratório:

   ```powershell
   Get-AzVM -ResourceGroup 'az140-21-RG'
   ```

1. Na sessão do PowerShell no painel do Cloud Shell, execute o seguinte para parar e desalocar todas as VMs do Azure criadas neste laboratório:

   ```powershell
   Get-AzVM -ResourceGroup 'az140-21-RG' | Stop-AzVM -NoWait -Force
   ```

   >**Observação**: O comando é executado de modo assíncrono (conforme determinado pelo parâmetro -NoWait), portanto, embora você possa executar outro comando do PowerShell imediatamente depois na mesma sessão do PowerShell, levará alguns minutos antes de o grupo de recursos ser de fato removido.
