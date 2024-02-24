---
lab:
  title: 'Laboratório: Configurar políticas de Acesso Condicional para AVD (AD DS)'
  module: 'Module 3: Manage Access and Security'
---

# Laboratório – Configurar políticas de Acesso Condicional para AVD (AD DS)
# Manual de laboratório do aluno

## Dependências do laboratório

- Uma assinatura do Azure
- Uma conta Microsoft ou uma conta do Microsoft Entra com a função de administrador global no locatário do Microsoft Entra associado à assinatura do Azure e com a função de proprietário ou colaborador na assinatura
- O laboratório concluído **Preparar para a implantação da Área de Trabalho Virtual do Azure (AD DS)**
- O laboratório **Implantar pools de hosts e hosts de sessão usando o portal do Azure (AD DS)** concluído

## Tempo estimado

60 minutos

## Cenário do laboratório

Você precisa controlar o acesso a uma implantação da Área de Trabalho Virtual do Azure em um ambiente do Active Directory Domain Services (AD DS) usando o acesso condicional do Microsoft Entra.

## Objetivos
  
Depois de realizar este laboratório, você será capaz de:

- Preparar-se para o Acesso Condicional baseado no Microsoft Entra para Área de Trabalho Virtual do Azure
- Implementar o acesso condicional baseado no Microsoft Entra para Área de Trabalho Virtual do Azure

## Arquivos de laboratório

- Nenhum 

## Instruções

>**Importante**: A Microsoft renomeou o **Azure Active Directory** (**Azure AD**) para **Microsoft Entra ID**. Para obter detalhes sobre essa alteração, confira [Novo nome do Azure Active Directory](https://learn.microsoft.com/en-us/entra/fundamentals/new-name). Este é um esforço contínuo, portanto, você ainda pode encontrar instâncias com uma incompatibilidade entre a instrução do laboratório e os elementos da interface à medida que você avança em exercícios individuais. Leve isso em consideração (especialmente neste laboratório, o **Microsoft Entra Connect** designa o novo nome do **Azure Active Directory Connect**, e o termo **Azure Active Directory** ainda é usado ao configurar o ponto de conexão de serviço na tarefa 4 do exercício 1).

>**Importante**: Ativar uma avaliação gratuita do Microsoft Entra ID P2 requer o fornecimento de informações de cartão de crédito. Por esse motivo, este exercício é totalmente opcional. Em vez disso, os instrutores de curso podem optar por demonstrar essa funcionalidade aos alunos.

### Exercício 1: Preparar-se para o Acesso Condicional baseado no Microsoft Entra para Área de Trabalho Virtual do Azure

As principais tarefas desse exercício são as seguintes:

1. Configurar o licenciamento do Microsoft Entra Premium P2
1. Configurar a Autenticação Multifator (MFA) do Microsoft Entra
1. Registrar um usuário para a MFA do Microsoft Entra
1. Configurar o ingresso híbrido no Microsoft Entra
1. Disparar sincronização delta do Microsoft Entra Connect

#### Tarefa 1: Configurar o licenciamento do Microsoft Entra Premium P2

>**Observação**: o licenciamento P1 ou P2 Premium do Microsoft Entra é necessário para implementar o Acesso Condicional do Microsoft Entra. Você usará uma avaliação de 30 dias para esse laboratório.

1. No seu computador de laboratório, inicie um navegador da Web, navegue até o [portal do Azure](https://portal.azure.com) e entre fornecendo as credenciais do Microsoft Entra de uma conta de usuário com a função Proprietário na assinatura que você usará neste laboratório e a função Administrador Global no locatário do Microsoft Entra associado à assinatura.

    >**Importante**: Certifique-se de que você esteja usando uma conta corporativa ou de estudante, **não** uma conta Microsoft.

1. No portal do Azure, pesquise e selecione **Azure Active Directory** para navegar até o locatário do Microsoft Entra associado à assinatura do Azure que você está usando para este laboratório.
1. Na folha do Azure Active Directory, na barra de menus vertical no lado esquerdo, na seção **Gerenciar**, clique em **Usuários**. 
1. Na folha **Usuários | Todos os usuários (versão prévia)**, selecione **aduser5**.
1. Na folha **aduser5 | Perfil**, na barra de ferramentas, clique em **Editar**, na seção **Configurações**, na lista suspensa **Local de uso**, selecione o país onde o ambiente de laboratório está localizado e, na barra de ferramentas, clique em **Salvar**.
1. Na folha **aduser5 | Perfil**, na seção **Identidade**, identifique o nome principal do usuário da conta do **aduser5**.

    >**Observação**: guarde esse valor. Você precisará disso em uma etapa posterior deste laboratório.

1. Na folha **Usuários | Todos os usuários (versão prévia)** selecione a conta de usuário que você usou para assinar no início desta tarefa e repita a etapa anterior caso sua conta não tenha o **Local de uso** atribuído. 

    >**Observação**: A propriedade **Local de uso** deve ser definida para atribuir licenças do Microsoft Entra Premium P2 a contas de usuário.

1. Na folha **Usuários | Todos os usuários (versão prévia)**, selecione a conta de usuário **aadsyncuser** e identifique seu nome de entidade de usuário.

    >**Observação**: guarde esse valor. Você precisará disso em uma etapa posterior deste laboratório.

1. No portal do Azure, navegue de volta até a folha **Visão geral** do locatário do Microsoft Entra e, na barra de menus vertical no lado esquerdo, na seção **Gerenciar**, clique em **Licenças**.
1. Na folha **Licenças\| Visão Geral**, na barra de menus vertical no lado esquerdo, na seção **Gerenciar**, clique em **Todos os produtos **.
1. Na folha **Licenças \| Todos os produtos**, na barra de ferramentas, clique em **+ Tentar/Comprar**.
1. Na folha **Ativar**, clique em **Avaliação gratuita** na seção **MICROSOFT ENTRA ID P2**, clique em **Ativar** e siga os prompts para concluir o processo de ativação.
1. Na folha **Licenças – Todos os produtos**, selecione a entrada **Enterprise Mobility + Security E5**. 
1. Na folha **Enterprise Mobility + Security E5**, na barra de ferramentas, clique em **+ Atribuir**.
1. Na folha **Atribuir licença**, clique em **Adicionar usuários e grupos**, na folha **Adicionar usuários e grupos**, selecione **aduser5** e sua conta de usuário e clique em **Selecionar**.
1. De volta à folha **Atribuir licença**, clique em **Opções de Atribuição**, na folha **Opções de Atribuição**, verifique se todas as opções estão habilitadas, clique em **Examinar + atribuir**, clique em **Atribuir**.

#### Tarefa 2: Configurar a Autenticação Multifator (MFA) do Microsoft Entra

1. No seu computador de laboratório, no navegador da Web que exibe o portal do Azure, navegue de volta para a folha **Visão Geral** do locatário do Microsoft Entra e, no menu vertical à esquerda, na seção **Gerenciar**, clique em **Segurança**.
1. Na folha **Segurança | Introdução**, no menu vertical à esquerda, na seção **Proteger**, clique em **Proteção de Identidade**.
1. Na folha **Proteção de Identidade | Visão geral**, no menu vertical à esquerda, na seção **Proteger**, clique em **Política e registro de autenticação multifator** (se necessário, atualize a página do navegador da Web).
1. Na folha **Proteção de Identidade | Política de registro de autenticação multifator**, na seção **Atribuições** da **Política de registro de autenticação multifator**, clique em **Todos os usuários**. Na guia **Incluir**, clique na opção **Selecionar indivíduos e grupos**. Na opção **Selecionar usuários**, clique em **Aduser5**, clique em **Selecionar**, na parte inferior da folha, defina a opção **Impor política** como **Ativar**e clique em **Salvar**.

#### Tarefa 3: Registrar um usuário para a MFA do Microsoft Entra

1. Em seu computador de laboratório, abra uma sessão do navegador da Web **InPrivate**, navegue até o [portal do Azure](https://portal.azure.com) e entre fornecendo o nome da entidade de segurança do usuário **aduser5** que você identificou anteriormente neste exercício e a senha que você definiu ao criar essa conta de usuário.
1. Quando for aparecer a mensagem **Mais informações necessárias**, clique em **Avançar**. Isso redirecionará automaticamente seu navegador para a página do **Microsoft Authenticator**.
1. Na página de **Verificação de segurança adicional**, na **Etapa 1: Como devemos entrar em contato com você?** seção, selecione seu método de autenticação preferencial e siga as instruções para concluir o processo de registro. 
1. Na página do portal do Azure, no canto superior direito, clique no ícone que representa o avatar do usuário, clique em **Sair**e feche a janela **InPrivate** do navegador. 

#### Tarefa 4: Configurar o ingresso híbrido no Microsoft Entra

> **Observação**: essa funcionalidade pode ser aproveitada para implementar segurança adicional ao configurar o Acesso Condicional para dispositivos com base no status de ingresso do Microsoft Entra.

1. No computador do laboratório, no navegador da Web que exibe o portal do Azure, pesquise e selecione **Máquinas Virtuais** e, na folha **Máquinas Virtuais**, selecione **az140-dc-vm11**.
1. Na folha **az140-dc-vm11**, selecione **Conectar**, no menu suspenso, selecione **Bastion**, na guia **Bastion** da folha **Conectar \|az140-dc-vm11**, selecione **Usar o Bastion**.
1. Quando solicitado, forneça as seguintes credenciais e selecione **Conectar**:

   |Configuração|Valor|
   |---|---|
   |Nome do usuário|**Aluno**|
   |Senha|**Pa55w.rd1234**|

1. Na sessão do Bastion para **az140-dc-vm11**, no menu **Iniciar**, expanda a pasta do **Microsoft Entra Connect** e selecione **Microsoft Entra Connect**.
   > **Observe** que se você receber uma janela de erro de falha que o Serviço de Sincronização não está em execução, vá para a janela de comando do PowerShell e digite **Start-Service "ADSync"** e tente a etapa 4 novamente.
1. Na página **Boas-vindas ao Microsoft Entra Connect** da janela do **Microsoft Entra Connect**, selecione **Configurar**.
1. Na página **Tarefas adicionais** na janela do **Microsoft Entra Connect**, selecione **Personalizar opções de sincronização** e selecione **Avançar**.
1. Na página **Visão Geral** na janela do **Microsoft Entra Connect**, revise as informações sobre **ingresso híbrido do Microsoft Entra** e **Write-back de dispositivo** e selecione **Avançar**.
1. Na página **Conectar ao Microsoft Entra**, na janela do **Microsoft Entra Connect**, autentique-se usando as credenciais da conta de usuário d**aadsyncuser** que você criou no exercício anterior e selecione **Avançar**.  

   > **Observação**: Forneça o atributo userPrincipalName da conta **aadsyncuser** que você registrou anteriormente neste laboratório e especifique a senha definida ao criar essa conta de usuário. 

1. Na página **Opções de dispositivo** na janela do **Microsoft Entra Connect**, verifique se a opção **Configurar Ingresso híbrido do Microsoft Entra** está selecionada e selecione **Avançar**. 
1. Na página **Sistemas operacionais do dispositivo** na janela do **Microsoft Entra Connect**, selecione a caixa de seleção **Dispositivos ingressados no domínio do Windows 10 ou posterior** e selecione **Avançar**. 
1. Na página **Configuração do SCP** na janela do **Microsoft Entra Connect**, marque a caixa de seleção ao lado da entrada **adatum.com**, na lista suspensa **Serviço de Autenticação**, selecione a entrada **Microsoft Entra** e selecione **Adicionar**. 
1. Quando solicitado, na caixa de diálogo **Credenciais de Administrador Corporativo**, especifique as seguintes credenciais e selecione **OK**:

   |Configuração|Valor|
   |---|---|
   |Nome do usuário|**ADATUM\Student**|
   |Senha|**Pa55w.rd1234**|

1. De volta à página **Configuração do SCP** na janela do **Microsoft Entra Connect**, selecione **Avançar**.
1. Na página **Pronto para configurar**, na janela do **Microsoft Entra Connect**, selecione **Configurar** e, depois que a configuração for concluída, selecione **Sair**.
1. Na sessão do Bastion para **az140-dc-vm11**, inicie o **ISE do Windows PowerShell** como administrador.
1. Na sessão do Bastion para **az140-dc-vm11**, do **Administrador: Console do ISE do Windows PowerShell**: execute o seguinte para mover a conta de computador **az140-cl-vm11** para a unidade organizacional (UO) **WVDClients**:

   ```powershell
   Move-ADObject -Identity "CN=az140-cl-vm11,CN=Computers,DC=adatum,DC=com" -TargetPath "OU=WVDClients,DC=adatum,DC=com"
   ```

1. Na sessão do Bastion para **az140-dc-vm11**, no menu **Iniciar**, expanda a pasta do **Microsoft Entra Connect** e selecione **Microsoft Entra Connect**.
1. Na página **Boas-vindas ao Microsoft Entra Connect** da janela do **Microsoft Entra Connect**, selecione **Configurar**.
1. Na página **Tarefas adicionais** na janela do **Microsoft Entra Connect**, selecione **Personalizar opções de sincronização** e selecione **Avançar**.
1. Na página **Conectar ao Microsoft Entra**, na janela do **Microsoft Entra Connect**, autentique-se usando as credenciais da conta de usuário d**aadsyncuser** que você criou no exercício anterior e selecione **Avançar**. 

   > **Observação**: Forneça o atributo userPrincipalName da conta **aadsyncuser** que você registrou anteriormente neste laboratório e especifique a senha definida ao criar essa conta de usuário. 

1. Na página **Conectar seus diretórios** na janela do **Microsoft Entra Connect**, selecione **Avançar**.
1. Na página **Filtragem de domínio e UO** na janela do **Microsoft Entra Connect**, verifique se a opção **Sincronizar domínios e UOs selecionadas** está selecionada, expanda o nó **adatum.com**, marque a caixa de seleção ao lado da UO **ToSync** (deixe todas as outras caixas de seleção selecionadas inalteradas) e selecione **Avançar**.
1. Na página **Recursos opcionais** na janela do **Microsoft Entra Connect**, aceite as configurações padrão e selecione **Avançar**.
1. Na página **Pronto para configurar** na janela do **Microsoft Entra Connect**, verifique se a caixa de seleção **Iniciar o processo de sincronização quando a configuração for concluída** está selecionada e selecione **Configurar**.
1. Examine as informações na página **Configuração concluída** e selecione **Sair** para fechar a janela do **Microsoft Entra Connect**.

#### Tarefa 5: Disparar sincronização delta do Microsoft Entra Connect

1. Na sessão do Bastion para **az140-dc-vm11**, alterne para o **Administrador: Janela do ISE do Windows PowerShell**.
1. Na sessão do Bastion para **az140-dc-vm11**, do **Administrador: Painel de console do ISE do Windows PowerShell**: execute o seguinte para disparar a sincronização delta do Microsoft Entra Connect:

   ```powershell
   Import-Module -Name "C:\Program Files\Microsoft Azure AD Sync\Bin\ADSync"
   Start-ADSyncSyncCycle -PolicyType Initial
   ```

1. Na sessão do Bastion para **az140-dc-vm11**, inicie o Microsoft Edge e navegue até o [portal do Azure](https://portal.azure.com). Quando solicitado, entre usando as credenciais do Microsoft Entra da conta de usuário com a função de Administrador Global no locatário do Microsoft Entra associado à assinatura do Azure que você está usando nesse laboratório.
1. Na sessão do Bastion para **az140-dc-vm11**, na janela do Microsoft Edge exibindo o portal do Azure, pesquise e selecione **Azure Active Directory** para navegar até o locatário do Microsoft Entra associado à assinatura do Azure que você está usando para esse laboratório.
1. Na folha do Azure Active Directory, na barra de menus vertical no lado esquerdo, na seção **Gerenciar**, clique em **Dispositivos**. 
1. Na folha **Dispositivos | Todas os dispositivos**, examine a lista de dispositivos e verifique se o dispositivo **az140-cl-vm11** está listado com a entrada **ingressada de forma híbrida no Microsoft Entra** na coluna **Tipo de Ingresso**.

   > **Observação**: Talvez seja necessário aguardar alguns minutos para que a sincronização seja realizada antes que o dispositivo apareça no portal do Azure.

### Exercício 2: Implementar o acesso condicional baseado no Microsoft Entra para Área de Trabalho Virtual do Azure

As principais tarefas desse exercício são as seguintes:

1. Criar uma política de Acesso Condicional baseada no Microsoft Entra para todas as conexões da Área de Trabalho Virtual do Azure
1. Testar a política de Acesso Condicional baseada no Microsoft Entra para todas as conexões da Área de Trabalho Virtual do Azure
1. Modificar a política de Acesso Condicional baseado no Microsoft Entra para excluir computadores ingressados no Microsoft Entra híbridos do requisito de MFA
1. Testar a política de Acesso Condicional baseada no Microsoft Entra modificada

#### Tarefa 1: Criar uma política de Acesso Condicional baseada no Microsoft Entra para todas as conexões da Área de Trabalho Virtual do Azure

>**Observação**: nesta tarefa, você configurará uma política de Acesso Condicional baseada no Microsoft Entra que exige que a MFA entre em uma sessão da Área de Trabalho Virtual do Azure. A política também imporá a reautenticação após as primeiras 4 horas após uma autenticação bem-sucedida.

1. No seu computador de laboratório, no navegador da Web que exibe o portal do Azure, navegue de volta para a folha **Visão Geral** do locatário do Microsoft Entra e, no menu vertical à esquerda, na seção **Gerenciar**, clique em **Segurança**.
1. Na folha **Segurança \| Introdução**, no menu vertical à esquerda, na seção **Proteger** clique em ** Acesso Condicional**.
1. Na folha **Acesso Condicional\| Políticas** na barra de ferramentas, clique em **+ Nova política**.
1. No painel **Novo**, defina as seguintes configurações:

   - Na caixa de texto **Nome**, digite **az140-31-wvdpolicy1**
   - Na seção **Atribuições**, selecione a opção **Usuários ou identidades de carga de trabalho**, na lista suspensa **A quê essa política se aplica?**. Verifique se a opção **Usuários e grupos** está selecionada. Na seção **Selecionar Usuários e grupos**, selecione a caixa de seleção **Usuários e grupos**. Na folha **Selecionar**, clique em **aduser5** e clique em **Selecionar**.
   - Na seção **Atribuições**, clique em **Aplicativos ou ações de nuvem** e certifique-se de que, na caixa de texto **Selecionar a que esta política se aplica**, a opção **Aplicativos de nuvem** esteja selecionada; clique na opção **Selecionar aplicativos** e, na folha **Selecionar**, na caixa de texto **Pesquisar**, insira **Área de Trabalho Virtual do Azure**; na listagem de resultados, selecione a caixa de seleção ao lado da entrada da **Área de Trabalho Virtual do Azure** e, na caixa de texto **Pesquisar**, insira **Área de Trabalho Remota da Microsoft**, marque a caixa de seleção ao lado da entrada da **Área de Trabalho Remota da Microsoft** e clique em **Selecionar**. 

   > **Observação**: a Área de Trabalho Virtual do Azure (ID do aplicativo 9cdead84-a844-4324-93f2-b2e6bb768d07) é usada quando o usuário assina um feed e se autentica no Gateway de Área de Trabalho Virtual do Azure durante uma conexão. A Área de Trabalho Remota da Microsoft (ID do aplicativo a4a365df-50f1-4397-bc59-1a1564b8bb9c) é usada quando o usuário se autentica no host da sessão quando o logon único está habilitado.

   - Na seção **Atribuições**, clique em **Condições**, clique em **Aplicativos**cliente, na folha **Aplicativos cliente**, defina a opção **Configurar** como **Sim**, verifique se as caixas de seleção **Navegador** e ** Aplicativos móveis e clientes da área de trabalho** estão selecionadas e clique em **Concluído**.
   - Na seção **Controles de Acesso**, clique em **Conceder**. Na folha **Conceder**, verifique se a opção **Conceder acesso** está selecionada. Selecione a caixa de seleção **Exigir autenticação multifator** e clique em **Selecionar**.
   - Na seção **Controles de Acesso**, clique em **Sessão**. Na folha **Sessão**, selecione a caixa de seleção de **Frequência de entrada**. Na primeira caixa de texto, digite **4**. Na lista suspensa **Selecionar unidades**, selecione**Horas**, deixe a caixa de seleção **Sessão do navegador persistente** desmarcada e clique em **Selecionar**.
   - Defina a opção **Habilitar política** como **Ativar**.

1. Na folha **Novo**, clique em **Criar**. 

#### Tarefa 2: Testar a política de Acesso Condicional baseada no Microsoft Entra para todas as conexões da Área de Trabalho Virtual do Azure

1. No computador do laboratório e, na janela do navegador da Web que exibe o portal do Azure, abra a sessão do shell do **PowerShell** no painel do **Cloud Shell**.
1. Na sessão do PowerShell no painel do Cloud Shell, execute o seguinte para iniciar as VMs do Azure host da sessão da Área de Trabalho Virtual do Azure que você irá usar nesse laboratório:

   ```powershell
   Get-AzVM -ResourceGroup 'az140-21-RG' | Start-AzVM
   ```

   >**Observação**: aguarde até que o comando seja concluído e todas as VMs do Azure no grupo de recursos **az140-21-RG** estejam em execução. 

1. Em seu computador de laboratório, abra uma sessão do navegador da Web **InPrivate**, navegue até o [portal do Azure](https://portal.azure.com) e entre fornecendo o nome da entidade de segurança do usuário **aduser5** que você identificou anteriormente neste exercício e a senha que você definiu ao criar essa conta de usuário.

   > **Observação**: verifique se você não foi solicitado a autenticar via MFA.

1. Na sessão do navegador da Web **InPrivate**, navegue até a página do cliente Web HTML5 da Área de Trabalho Virtual do Azure em [https://rdweb.wvd.microsoft.com/arm/webclient](https://rdweb.wvd.microsoft.com/arm/webclient).

   > **Observação**: verifique se isso irá dispara automaticamente a autenticação por meio da MFA.

1. No painel **Inserir código**, digite o código da mensagem de texto ou do aplicativo autenticador que você registrou e selecione **Verificar**.
1. Na página **Todos os Recursos**, clique em **Prompt de Comando**. No painel **Acessar recursos locais**, desmarque a caixa de seleção **Impressora** e clique em **Permitir**.
1. Quando solicitado, na caixa de texto **Inserir suas credenciais**, na caixa de texto **Nome de usuário** digite o nome de entidade de usuário **aduser5** e, na caixa de texto **Senha**, digite a senha definida ao criar essa conta de usuário e clique em **Enviar**.
1. Verifique se o Aplicativo Remoto do **Prompt de Comando** foi iniciado com êxito.
1. Na janela Aplicativo Remoto do **Prompt de Comando**, no prompt de comando, digite **logoff** e pressione a tecla **Enter**.
1. De volta à página **Todos os Recursos**, no canto superior direito, clique em **aduser5**. No menu suspenso, clique em **Sair** e feche a janela do navegador da Web **InPrivate**.

#### Tarefa 3: Modificar a política de Acesso Condicional baseado no Microsoft Entra para excluir computadores ingressados no Microsoft Entra híbridos do requisito de MFA

>**Observação**: Nessa tarefa, você modificará a política de Acesso Condicional baseada no Microsoft Entra que exige que a MFA entre em uma sessão da Área de Trabalho Virtual do Azure para que as conexões provenientes de computadores ingressados no Microsoft Entra não exijam MFA.

1. No computador do laboratório, na janela do navegador que exibe o portal do Azure, na folha **Acesso Condicional | Políticas**, clique na entrada que representa a política **az140-31-wvdpolicy1**.
1. Na folha **az140-31-wvdpolicy1**, na seção **Controles de Acesso**, clique em **Conceder**. Na folha **Conceder**, marque a caixa de seleção **Exigir autenticação multifator** e **Exigir Dispositivo Ingressado de Forma Híbrida no Microsoft Entra**. Verifique se a opção **Exigir um dos controles selecionados** está habilitada e clique em **Selecionar**.
1. Na folha **az140-31-wvdpolicy1**, clique em **Salvar**.

>**Observação**: Pode levar alguns minutos para que a política entre em vigor.

#### Tarefa 4: Testar a política de Acesso Condicional baseada no Microsoft Entra modificada

1. No seu computador de laboratório, na janela do navegador que exibe o portal do Azure, pesquise e selecione **Máquinas Virtuais** e, na folha **Máquinas Virtuais**, selecione a entrada **az140-cl-vm11**.
1. Na folha **az140-cl-vm11**, selecione **Conectar**, no menu suspenso, selecione **Bastion**, na guia **Bastion** da folha **az140-cl-vm11 \| Conectar**, selecione **Usar Bastion**.
1. Quando solicitado, forneça as seguintes credenciais e selecione **Conectar**:

   |Configuração|Valor|
   |---|---|
   |Nome do usuário|**Student@adatum.com**|
   |Senha|**Pa55w.rd1234**|

1. Na sessão do Bastion para **az140-cl-vm11**, inicie o Microsoft Edge e navegue até a página do cliente Web HTML5 da Área de Trabalho Virtual do Azure em [https://rdweb.wvd.microsoft.com/arm/webclient](https://rdweb.wvd.microsoft.com/arm/webclient).

   > **Observação**: verifique se, dessa vez, você não será solicitado a autenticar via MFA. Isso ocorre porque **az140-cl-vm11** está ingressado de forma híbrida no Microsoft Entra.

1. Na página **Todos os Recursos**, clique em **Prompt de Comando**. No painel **Acessar recursos locais**, desmarque a caixa de seleção **Impressora** e clique em **Permitir**.
1. Quando solicitado, na caixa de texto **Inserir suas credenciais**, na caixa de texto **Nome de usuário** digite o nome de entidade de usuário **aduser5** e, na caixa de texto **Senha**, digite a senha definida ao criar essa conta de usuário e clique em **Enviar**.
1. Verifique se o Aplicativo Remoto do **Prompt de Comando** foi iniciado com êxito.
1. Na janela Aplicativo Remoto do **Prompt de Comando**, no prompt de comando, digite **logoff** e pressione a tecla **Enter**.
1. De volta à página **Todos os Recursos**, no canto superior direito, clique em **aduser5**. No menu suspenso, clique em **Sair**.
1. Na sessão do Bastion para **az140-cl-vm11**, clique em **Iniciar**. Na barra vertical diretamente acima do botão **Iniciar**, clique no ícone que representa a conta de usuário conectado e, no menu pop-up, clique em **Sair**.

### Exercício 3: Parar e desalocar VMs do Azure provisionadas e usadas no laboratório

As principais tarefas desse exercício são as seguintes:

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
