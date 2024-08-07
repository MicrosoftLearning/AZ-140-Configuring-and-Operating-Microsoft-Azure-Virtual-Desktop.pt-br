---
lab:
  title: 'Laboratório: Criar e configurar pools de hosts e hosts de sessão (Microsoft Entra DS)'
  module: 'Module 2: Implement an AVD Infrastructure'
---

# Laboratório: Criar e configurar pools de hosts e hosts de sessão (Microsoft Entra DS)
# Manual de laboratório do aluno

## Dependências do laboratório

- Uma assinatura do Azure
- Uma conta Microsoft ou uma conta do Microsoft Entra com a função de administrador global no locatário do Microsoft Entra associado à assinatura do Azure e com a função de proprietário ou colaborador na assinatura
- O laboratório**Preparar para implantação da Área de Trabalho Virtual do Azure (Microsoft Entra DS)** concluído

## Tempo estimado

60 minutos

## Cenário do laboratório

Você precisa criar e configurar pools de host e hosts de sessão em um ambiente do Azure Active Directory Domain Services (Microsoft Entra DS).

## Objetivos
  
Depois de realizar este laboratório, você será capaz de:

- Configure um ambiente da Área de Trabalho Virtual do Azure em um domínio do Microsoft Entra DS. 
- Valide o ambiente da Área de Trabalho Virtual do Azure em um domínio do Microsoft Entra DS. 

## Arquivos do laboratório

- Nenhum 

## Instruções

### Exercício 1: Configurar um ambiente da Área de Trabalho Virtual do Azure
  
As principais tarefas desse exercício são as seguintes:

1. Preparar o domínio do AD DS e a assinatura do Azure para implantação de um pool de hosts da Área de Trabalho Virtual do Azure
1. Configurar uma pool de host da Área de Trabalho Virtual do Azure
1. Configurar grupos de aplicativos da Área de Trabalho Virtual do Azure
1. Configurar workspaces da Área de Trabalho Virtual do Azure

#### Tarefa 1: Preparar o domínio do AD DS e a assinatura do Azure para implantação de um pool de hosts da Área de Trabalho Virtual do Azure

1. No seu computador de laboratório, inicie um navegador da Web, navegue até o [portal do Azure](https://portal.azure.com) e entre fornecendo credenciais de uma conta de usuário com a função Proprietário na assinatura que você usará nesse laboratório.
1. No seu computador de laboratório, no portal do Azure, pesquise e selecione **Máquinas virtuais** e, na folha **Máquinas virtuais**, selecione a entrada **az140-cl-vm11a**. Isso abrirá a folha **az140-cl-vm11a**.
1. Na folha **az140-cl-vm11a**, selecione **Conectar**, no menu suspenso, selecione **Bastion**, na guia **Bastion** da folha **az140-cl-vm11a \| Conectar**, selecione **Usar Bastion**.
1. Quando solicitado, forneça as seguintes credenciais e selecione **Conectar**:

   |Configuração|Valor|
   |---|---|
   |Nome do usuário|**aadadmin1@adatum.com**|
   |Senha|Senha definida no laboratório anterior|

1. No Bastion para a VM do Azure **az140-cl-vm11a**, inicie o Microsoft Edge, navegue até o [portal do Azure](https://portal.azure.com) e entre fornecendo o nome da entidade de usuário da conta de usuário **aadadmin1** com a senha definida anteriormente nesse laboratório como sua senha.

   >**Observação**: Você pode identificar o atributo nome de entidade de usuário (UPN) da conta do **aadadmin1** examinando sua caixa de diálogo de propriedades do console de Usuários e Computadores do Active Directory ou alternando de volta para o seu computador de laboratório e examinando suas propriedades da folha de locatário do Microsoft Entra no portal do Azure.

1. Na sessão do Bastion para **az140-cl-vm11a**, no Microsoft Edge que exibe o portal do Azure, abra uma sessão do PowerShell no **Cloud Shell** e execute o seguinte registro do provedor de recursos **Microsoft.DesktopVirtualization**:

   ```powershell
   Register-AzResourceProvider -ProviderNamespace Microsoft.DesktopVirtualization
   ```

1. Na sessão do Bastion para **az140-cl-vm11a**, no Microsoft Edge que exibe o portal do Azure, pesquise e selecione **Redes virtuais** e, na folha **Redes virtuais**, selecione a entrada **az140-aadds-vnet11a**. 
1. Na folha **az140-aadds-vnet11a**, selecione **Sub-redes**. Na folha **Sub-redes**, selecione **+ Sub-rede**, na folha **Adicionar sub-rede**. Na caixa de texto **Nome**, digite **hp1-Subnet**, deixe todas as outras configurações com seus valores padrão e selecione **Salvar**. 

#### Tarefa 2: Configurar uma pool de host da Área de Trabalho Virtual do Azure

1. Na sessão do Bastion para **az140-cl-vm11a**, na janela do Microsoft Edge que exibe o portal do Azure, pesquise e selecione **Área de Trabalho Virtual do Azur** e, na folha **Área de Trabalho Virtual do Azure**, no menu vertical à esquerda, na seção **Gerenciar**, selecione **Pools de Host** e, na folha **Pools de Host\|da Área de Trabalho Virtual do Azure**, selecione **+ Criar**. 
1. Na guia **Noções básicas** da folha **Criar um pool de hosts**, especifique as seguintes configurações e selecione **Avançar: Máquinas Virtuais >**:

   |Configuração|Valor|
   |---|---|
   |Assinatura|o nome da assinatura do Azure que você está usando neste laboratório|
   |Grupo de recursos|o nome de um novo grupo de recursos **az140-21a-RG**|
   |Nome do pool de host|**az140-21a-hp1**|
   |Localidade|o nome da região do Azure na qual você implantou a instância do Microsoft Entra DS anteriormente neste laboratório|
   |Ambiente de validação|**Não**|
   |Tipo de grupo de aplicativos preferencial|**Desktop**|
   |Tipo de pool de host|**Em pool**|
   |Limite máximo da sessão|**12**|
   |Algoritmo de balanceamento de carga|**Amplitude**|

   > **Observação**: se o usuário tem os aplicativos RemoteApp e Desktop publicados, o tipo de grupo de aplicativos preferencial determinará qual deles aparecerá em seu feed.

1. Na guia **Máquinas Virtuais** da folha **Criar um pool de hosts**, especifique as seguintes configurações (deixe as outras com seus padrões) e selecione **Avançar: Workspace >** (substitua o espaço reservado *<Azure_AD_domain_name>* pelo nome do locatário do Microsoft Entra associado à assinatura na qual você implantou a instância do Microsoft Entra DS e substitua o espaço reservado `<password>` pela senha definida ao criar a conta aadadmin1):

   > **Observação**: lembre-se da senha usada. Você precisará dela mais tarde neste e nos laboratórios subsequentes.:

   |Configuração|Valor|
   |---|---|
   |Adicionar máquinas virtuais|**Sim**|
   |Grupo de recursos|**O padrão é o mesmo que o pool de hosts**|
   |Prefixo do nome|**az140-21-p1**|
   |Localização da máquina virtual|o nome da região do Azure na qual você implantou recursos no primeiro exercício deste laboratório|
   |Opções de disponibilidade|**Nenhuma redundância de infraestrutura necessária**|
   |Tipo de segurança|**Máquinas virtuais de início confiável**|
   |Imagem|**Windows 11 Enterprise multissessão + Microsoft 365 Apps, versão 22H2**|
   |Tamanho da máquina virtual|**Standard DC2s_v3**|
   |Número de VMs|**2**|
   |Tipo de disco de SO|**SSD Standard**|
   |Rede virtual|**az140-aadds-vnet11a**|
   |Sub-rede|**hp1-Subnet (10.10.1.0/24)**|
   |Grupo de segurança de rede|**Basic**|
   |Portas de entrada públicas|**Não**|
   |Selecione o diretório ao qual deseja ingressar|**Active Directory**|
   |UPN de ingresso no domínio do AD|**aadadmin1@adatum.com**|
   |Senha|Use a senha para aadadmin1|
   |Especificar o domínio ou a unidade|**Sim**|
   |Domínio a ingressar|**adatum.com**|
   |Caminho da Unidade Organizacional|**OU=AADDC Computers,DC=adatum,DC=com**|
   |Nome de usuário da conta de Administrador da Máquina Virtual|**student**|
   |Senha da conta de Administrador da Máquina Virtual|**Pa55w.rd1234**|

1. Na guia **Workspace** da folha **Criar um pool de hosts**, especifique as seguintes configurações e selecione **Examinar + criar**:

   |Configuração|Valor|
   |---|---|
   |Registre o grupo de aplicativos da área de trabalho|**Não**|

1. Na guia **Revisar + criar** da folha **Criar um pool de host**, selecione **Criar**.

   > **Observação**: aguarde até que a implantação seja concluída. Isso deverá levar cerca de 15 minutos.

#### Tarefa 3: Configurar grupos de aplicativos da Área de Trabalho Virtual do Azure

1. Na sessão do Bastion para **az140-cl-vm11a**, no portal do Azure, pesquise e selecione **Área de Trabalho Virtual do Azure** e, na folha **Área de Trabalho Virtual do Azure**, selecione **Grupos de aplicativos**.
1. Na folha **Grupos de Aplicativos \|da Área de Trabalho Virtual do Azure**, selecione o grupo de aplicativos da área de trabalho **az140-21a-hp1-DAG**gerado automaticamente.
1. Na folha **az140-21a-hp1-DAG**, no menu vertical ao lado esquerdo, na seção **Gerenciar**, selecione **Atribuições**.
1. Na folha **az140-21a-hp1-DAG \| Atribuições**, selecione **+ Adicionar**.
1. Na folha **Selecionar usuários ou grupos de usuários do Microsoft Entra**, selecione**az140-wvd-apooled ** e clique em **Selecionar**.
1. Navegue de volta para a folha **Grupos de Aplicativos \| da Área de Trabalho Virtual do Azure** e selecione **+ Criar**.
1. Na guia **Noções básicas** da folha **Criar um grupo de aplicativos**, especifique as seguintes configurações e selecione **Avançar: Aplicativos >**:

   |Configuração|Valor|
   |---|---|
   |Assinatura|o nome da assinatura do Azure que você está usando neste laboratório|
   |Grupo de recursos|**az140-21a-RG**|
   |Pool de host|**az140-21a-hp1**|
   |Tipo de grupo de aplicativos|**RemoteApp**|
   |Nome do grupo de aplicativos|**az140-21a-hp1-Office365-RAG**|

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
1. De volta à guia **Atribuições** da folha **Criar um grupo de aplicativos**, selecione **Avançar: Workspace >**.
1. Na guia **Workspace** da folha **Criar um workspace**, especifique a seguinte configuração e selecione **Examinar + criar**:

   |Configuração|Valor|
   |---|---|
   |Registrar o grupo de aplicativos|**Não**|

1. Na guia **Revisar + criar** da folha **Criar um grupo** de aplicativos, selecione **Criar**.

   > **Observação**: Agora você criará um grupo de aplicativos com base no caminho do arquivo como a origem do aplicativo

1. Na sessão do Bastion para **az140-cl-vm11a**, na janela do navegador da Web que exibe o portal do Azure, pesquise e selecione **Área de Trabalho Virtual do Azure** e, na folha **Área de Trabalho Virtual do Azure**, selecione **Grupos de aplicativos**.
1. Na folha **Grupos de Aplicativos \| da Área de Trabalho Virtual do Azure**, selecione **+ Criar**. 
1. Na guia **Noções básicas** da folha **Criar um grupo de aplicativos**, especifique as seguintes configurações e selecione **Avançar: Aplicativos >**:

   |Configuração|Valor|
   |---|---|
   |Assinatura|o nome da assinatura do Azure que você está usando neste laboratório|
   |Grupo de recursos|**az140-21a-RG**|
   |Pool de host|**az140-21a-hp1**|
   |Tipo de grupo de aplicativos|**RemoteApp**|
   |Nome do grupo de aplicativos|**az140-21a-hp1-Utilities-RAG**|

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
1. Na folha **Selecionar usuários ou grupos de usuários do Microsoft Entra**, selecione **az140-wvd-aremote-app** e **az140-wvd-aadmins** e clique em**Selecionar**.
1. De volta à guia **Atribuições** da folha **Criar um grupo de aplicativos**, selecione **Avançar: Workspace >**.
1. Na guia **Workspace** da folha **Criar um workspace**, especifique a seguinte configuração e selecione **Examinar + criar**:

   |Configuração|Valor|
   |---|---|
   |Registrar o grupo de aplicativos|**Não**|

1. Na guia **Revisar + criar** da folha **Criar um grupo** de aplicativos, selecione **Criar**.

#### Tarefa 4: Configurar workspaces da Área de Trabalho Virtual do Azure

1. Na sessão do Bastion para **az140-cl-vm11a**, na janela do Microsoft Edge que exibe o portal do Azure, pesquise e selecione **Área de Trabalho Virtual do Azure** e, na folha ** Área de Trabalho Virtual do Azure**, selecione **Workspaces**.
1. Na folha **Workspaces\| da Área de Trabalho Virtual do Azure**, selecione **+ Criar**. 
1. Na guia **Noções básicas** da folha **Criar um workspace**, especifique as seguintes configurações e selecione **Avançar: Grupos de aplicativos >**:

   |Configuração|Valor|
   |---|---|
   |Assinatura|o nome da assinatura do Azure que você está usando neste laboratório|
   |Grupo de recursos|**az140-21a-RG**|
   |Nome do workspace|**az140-21a-ws1**|
   |Nome amigável|**az140-21a-ws1**|
   |Localidade|o nome da região do Azure na qual você implantou recursos neste laboratório||

1. Na guia **Grupos de aplicativos** da folha **Criar um workspace**, especifique as seguintes configurações:

   |Configuração|Valor|
   |---|---|
   |Registrar grupos de aplicativos|**Sim**|

1. Na guia **Workspace** da folha **Criar um workspace**, selecione** + Registrar grupos de aplicativos **.
1. Na folha **Adicionar grupos de aplicativos**, selecione o sinal de adição ao lado das entradas **az140-21a-hp1-DAG**, **az140-21a-hp1-Office365-RAG**, e **az140-21a-hp1-Utilities-RAG** e clique em **Selecionar**. 
1. De volta à guia **Grupos de aplicativos** da folha **Criar um workspace**, selecione **Examinar + criar**.
1. Na guia **Revisar + criar** da folha **Criar um workspace**, selecione **Criar**.

### Exercício 2: Validar o ambiente da Área de Trabalho Virtual do Azure
  
As principais tarefas desse exercício são as seguintes:

1. Instalar o cliente de Área de Trabalho Remota (MSRDC) da Microsoft em um computador Windows 10
1. Assinar um workspace da Área de Trabalho Virtual do Azure
1. Testar aplicativos da Área de Trabalho Virtual do Azure

#### Tarefa 1: Instalar o cliente de Área de Trabalho Remota (MSRDC) da Microsoft em um computador Windows 10

1. Na sessão do Bastion para **az140-cl-vm11a**, inicie o Microsoft Edge e navegue até a [página de download do cliente da Área de Trabalho do Windows](https://go.microsoft.com/fwlink/?linkid=2068602) e, quando solicitado, execute sua instalação seguindo os prompts. Selecione a opção **Instalar para todos os usuários neste computador**. 
1. Depois que a instalação for concluída, inicie o cliente da Área de Trabalho Remota.

#### Tarefa 2: Assinar um workspace da Área de Trabalho Virtual do Azure

1. Na janela cliente da **Área de Trabalho Remota**, selecione **Assinar** e, quando solicitado, entre com as credenciais de **aaduser1** (usando seu atributo userPrincipalName como o nome de usuário e a senha definida ao criar essa conta). 

   > **Observação**: como alternativa, na janela cliente da **Área de Trabalho Remota**, selecione **Assinar com a URL**, no painel **Assinar um Workspace**. Na **URL de Email ou Workspace**, digite **https://rdweb.wvd.microsoft.com/api/arm/feeddiscovery**, selecione **Avançar** e, uma vez solicitado, entre com as credenciais de **aaduser1** (usando seu atributo userPrincipalName como o nome de usuário e **Pa55w.rd1234** como a senha). 

   > **Observação**: o nome UPN do **aaduser1** deve estar no formato **aaduser1@***<Azure_AD_domain_name>*, em que o espaço reservado * <Azure_AD_domain_name>* corresponde ao nome do locatário do Microsoft Entra associado à assinatura na qual você implantou a instância do Microsoft Entra DS.

1. Na janela **Permanecer conectado a todos os seus aplicativos** desmarque a caixa de seleção **Permitir que minha organização gerencie meu dispositivo** e selecione **Não, entrar somente nesse aplicativo**. 
1. Verifique se a página **Área de trabalho remota** exibe  SessionDesktop incluído no grupo de aplicativos da área de trabalho az140-21-hp1-DAG gerado automaticamente, publicado no workspace e associado à conta de usuário **aduser1** por meio de sua associação de grupo. 

   > **Observação**: isso é esperado, pois o **Tipo de grupo de aplicativos preferencial** do pool de host está definido como **Área de trabalho**.

#### Tarefa 3: Testar aplicativos da Área de Trabalho Virtual do Azure

1. Na sessão do Bastion para **az140-cl-vm11a**, na janela do cliente da **Área de Trabalho Remota**, na lista de aplicativos, clique duas vezes em **SessionDesktop** e verifique se uma sessão da Área de Trabalho Remota é iniciada. 

   > **Observação**: Inicialmente, pode levar alguns minutos para o aplicativo ser iniciado, mas, posteriormente, a inicialização do aplicativo deve ser muito mais rápida.

   > **Observação**: Se você receber o prompt de entrada **Boas-vindas ao Microsoft Teams**, feche-o. 

1. Na sessão da **Área de Trabalho de Sessão**, clique com o botão direito do mouse em **Iniciar**, selecione **Executar**, na caixa de texto **Abrir**da caixa de diálogo **Executar**, digite **cmd** e selecione **OK**. 
1. Na sessão da **Área de Trabalho de Sessão**, no Prompt de Comando, digite o **nome do host** e pressione a tecla **Enter** para exibir o nome do computador no qual a sessão da Área de Trabalho Remota está em execução.
1. Verifique se o nome exibido é **az140-21-p1-0**, **az140-21-p1-1** ou **az140-21-p1-2**.
1. No Prompt de Comando, digite **logoff** e pressione a tecla **Enter** para fazer logoff da Área de Trabalho de Sessão.

   > **Observação**: em seguida, você modificará o **tipo de grupo de aplicativos preferencial**, definindo-o como **RemoteApp**.

1. Na sessão do Bastion para **az140-cl-vm11a**, na janela do navegador da web que exibe o portal do Azure, pesquise e selecione **Área de Trabalho Virtual do Azure** e, na folha da **Área de Trabalho Virtual do Azure**, na barra de menu vertical, acesse a **seção Gerenciar** e selecione **Pools de hosts**.
1. Na folha **Pools de host\| da Área de Trabalho Virtual do Azure**, na lista de pools de hosts, selecione **az140-21-hp1**.
1. Na folha **az140-21-hp1**, vá para a barra de menus vertical e, na seção **Configurações**, selecione **Propriedades**. Em **Tipo de grupo de aplicativos preferencial**, selecione **Aplicativo Remoto** e, em seguida, escolha **Salvar**. 
1. Na sessão do Bastion para **az140-cl-vm11**, na janela do cliente da **Área de Trabalho Remota**, selecione o símbolo de reticências no canto superior direito e, no menu suspenso, selecione **Atualizar**.
1. Verifique se a página **Área de Trabalho Remota** exibe os aplicativos individuais incluídos nos dois grupos de aplicativos que você criou e publicou no workspace e que também estão associados à conta de usuário **aduser1** por meio de sua associação de grupo. 

   > **Observação**: isso é o esperado, pois o **Tipo de grupo de aplicativos preferencial** do pool de host agora está definido como **RemoteApp**.

1. Na sessão do Bastion para **az140-cl-vm11a**, na janela cliente da **Área de Trabalho Remota**, na lista de aplicativos, clique duas vezes no **Prompt de Comando** e verifique se ele inicia uma janela do **Prompt de Comando**. Quando solicitado a autenticar, digite a senha definida ao criar a conta de usuário aduser1 **, marque a caixa **de seleção **Lembrar de mim** e selecione **OK.**
1. No Prompt de Comando, digite **logoff** e pressione a tecla **Enter** para sair da sessão de Aplicativo Remoto atual.

### Exercício 3: Parar e desalocar VMs do Azure provisionadas e usadas no laboratório

As principais tarefas desse exercício são as seguintes:

1. Parar e desalocar VMs do Azure provisionadas e usadas no laboratório

>**Observação**: Neste exercício, você desalocará as VMs do Azure provisionadas e usadas neste laboratório para minimizar os encargos de computação correspondentes

#### Tarefa 1: Desalocar VMs do Azure provisionadas e usadas no laboratório

1. Alterne para o computador de laboratório e, na janela do navegador da Web que exibe o portal do Azure, abra a sessão do shell do **PowerShell** no painel do **Cloud Shell**.
1. Na sessão do PowerShell no painel do Cloud Shell, execute o seguinte para listar todas as VMs do Azure criadas e usadas neste laboratório:

   ```powershell
   Get-AzVM -ResourceGroup 'az140-21a-RG'
   ```

1. Na sessão do PowerShell no painel do Cloud Shell, execute o seguinte para parar e desalocar todas as VMs do Azure que você criou e usou neste laboratório:

   ```powershell
   Get-AzVM -ResourceGroup 'az140-21a-RG' | Stop-AzVM -NoWait -Force
   ```

   >**Observação**: O comando é executado de modo assíncrono (conforme determinado pelo parâmetro -NoWait), portanto, embora você possa executar outro comando do PowerShell imediatamente depois na mesma sessão do PowerShell, levará alguns minutos antes de o grupo de recursos ser de fato removido.

