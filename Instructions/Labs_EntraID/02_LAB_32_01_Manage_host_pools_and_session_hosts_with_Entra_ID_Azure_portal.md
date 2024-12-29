---
lab:
  title: 'Laboratório: Gerenciar pools de hosts e hosts de sessão usando o portal do Azure (ID Entra)'
  module: 'Module 3.2: Plan and implement user experience and client settings'
---

# Laboratório — Gerenciar pools de hosts e hosts de sessão usando o portal do Azure (ID Entra)
# Manual de laboratório do aluno

## Dependências do laboratório

- O nome da assinatura do Azure que você usará nesse laboratório
- Uma conta de usuário do Microsoft Entra com a função Proprietário ou Colaborador na assinatura do Azure que você usará neste laboratório e com permissões suficientes para unir dispositivos ao locatário do Entra associado a essa assinatura do Azure.
- Ter concluído o laboratório *Implantar pools de hosts e hosts de sessão usando o portal do Azure (AD DS)*

## Tempo estimado

30 minutos

## Cenário do laboratório

Você tem um ambiente de Área de Trabalho Virtual do Azure existente. Você precisa configurar o pool de hosts com hosts de sessão ingressados no Microsoft Entra para dar suporte a uma variedade de requisitos funcionais e de negócios. Estes requisitos incluem:

- Implante hosts de sessão adicionais para acomodar um número maior de usuários remotos
- Minimize o custo do ambiente de área de trabalho virtual do Azure otimizando a configuração de balanceamento de carga do pool de hosts e aproveitando a funcionalidade *Iniciar VM ao conectar*
- Maximize a disponibilidade dos hosts de sessão durante o horário comercial implementando janelas de manutenção
- Habilitar logon único para hosts de sessão ingressados no Microsoft Entra
- Maximizar a usabilidade e a experiência do usuário (como reconexão automática de sessões desconectadas)

## Objetivos
  
Depois de realizar este laboratório, você será capaz de:

- Configurar hosts de sessão da Área de Trabalho Virtual do Azure associados ao Microsoft Entra para oferecer suporte a uma variedade de requisitos funcionais e comerciais

## Arquivos do laboratório

- Nenhum

## Instruções

### Exercício 1: Gerenciar um ambiente de Área de Trabalho Virtual do Azure contendo hosts de sessão ingressados no Microsoft Entra
  
As principais tarefas desse exercício são as seguintes:

1. Implantar hosts de sessão de pool de hosts adicionais da Área de Trabalho Virtual do Azure
1. Revisar e configurar as propriedades do pool de hosts
1. Atribuir a função RBAC necessária a uma entidade de serviço da Área de Trabalho Virtual do Azure
1. Configurar atualizações agendadas do agente
1. Configurar propriedades RDP do pool de hosts

#### Tarefa 1: Implantar hosts de sessão de pool de hosts adicionais da Área de Trabalho Virtual do Azure

1. Se necessário, no computador do laboratório, inicie um navegador da Web, navegue até o portal do Azure e entre fornecendo credenciais de uma conta de usuário com a função Proprietário na assinatura que você usará neste laboratório.

    > **Observação**: use as credenciais da conta `User1-` listada na guia Recursos no lado direito da janela da sessão de laboratório.

1. No navegador da Web que exibe o portal do Azure, pesquise e selecione **Área de Trabalho Virtual do Azure** e, na página **Área de Trabalho Virtual do Azure**, na seção **Gerenciar** da barra de menu vertical, selecione **Pools de hosts**.
1. Na página **Área de Trabalho Virtual do Azure \| Pools de hosts**, na lista de pools de hosts, selecione **az140-21-hp1**.
1. Na página **az140-21-hp1**, na barra de menu vertical, na seção **Gerenciar**, selecione **Hosts de sessão** e verifique se o pool consiste em dois hosts. 
1. Na página **az140-21-hp1 \| Hosts de sessão**, selecione **+ Adicionar**.
1. Na guia **Básico** da página **Adicionar máquinas virtuais a um pool de hosts**, revise as configurações pré-configuradas e selecione **Avançar: Máquinas virtuais**.
1. Na guia **Máquinas virtuais** da página **Adicionar máquinas virtuais a um pool de hosts**, especifique as seguintes configurações e selecione **Revisar + criar** (deixe as outras com suas configurações padrão):

    > **Observação**: ao definir o valor do **Prefixo do nome**, alterne para a guia Recursos no lado direito da janela da sessão de laboratório e identifique a sequência de caracteres entre *Usuário1-* e o caractere *@*. Use esta cadeia de caracteres para substituir o espaço reservado *random*.

    |Configuração|Valor|
    |---|---|
    |Grupo de recursos|**az140-21e-RG**|
    |Prefixo do nome|**sh**-*random*|
    |Localização da máquina virtual|O nome da região do Azure na qual você implantou as duas primeiras VMs do host de sessão|
    |Opções de disponibilidade|**Nenhuma redundância de infraestrutura necessária**|
    |Tipo de segurança|**Máquinas virtuais de início confiável**|
    |Imagem|**Windows 11 Enterprise multi-sessão, versão 23H2 + Microsoft 365 Apps**|
    |Tamanho da máquina virtual|**Standard DC2s_v3**|
    |Número de VMs|**1**|
    |Tipo de disco de SO|**SSD Standard**|
    |Tamanho do disco de SO|**Tamanho padrão (128GB)**|
    |Diagnóstico de Inicialização|**Habilitar com a conta de armazenamento gerenciada (recomendado)**|
    |Rede virtual|**az140-vnet11e**|
    |Sub-rede|**hp1-Subnet**|
    |Grupo de segurança de rede|**Basic**|
    |Portas de entrada públicas|**Não**|
    |Selecione o diretório ao qual deseja ingressar|**Microsoft Entra ID**|
    |Registrar a VM com o Intune|**Não**|
    |Nome de usuário|**Aluno**|
    |Senha|A mesma senha que você usou ao implantar os hosts de sessão no laboratório *Implantar pools de hosts e hosts de sessão usando o portal do Azure (ID Entra)* 
    |Confirmar senha|A mesma senha que você especificou anteriormente|

    > **Observação**: a senha deve ter pelo menos 12 caracteres e consistir em uma combinação de letras minúsculas, letras maiúsculas, dígitos e caracteres especiais. Para obter detalhes, consulte as informações sobre [os requisitos de senha ao criar uma VM do Azure](https://learn.microsoft.com/en-us/azure/virtual-machines/windows/faq#what-are-the-password-requirements-when-creating-a-vm-).

    > **Observação**: Como você provavelmente notou, é possível alterar a imagem e o prefixo das VMs à medida que você adiciona hosts de sessão ao pool existente. Em geral, isso não é recomendado, a menos que você planeje substituir todas as VMs no pool. 

1. Na guia **Revisar + criar** da página **Adicionar máquinas virtuais a um pool de hosts**, selecione **Criar**

    > **Observação**: Não espere que o processo de provisionamento seja concluído, mas prossiga para a próxima tarefa. O processo de provisionamento pode levar cerca de 20 minutos. 

#### Tarefa 2: Revisar e configurar as propriedades do pool de hosts

1. No computador do laboratório, no navegador da Web que exibe o portal do Azure, pesquise e selecione **Área de Trabalho Virtual do Azure**. Na página **Área de Trabalho Virtual do Azure**, na seção **Gerenciar** do menu de navegação vertical, selecione **Pools de hosts** e, na página **Área de Trabalho Virtual do Azure \| Pools de hosts**, selecione **az140-21-hp1**. 
1. Na página **az140-21-hp1**, na seção **Configurações**, selecione **Propriedades**.
1. Na página **az140-21-hp1\|Propriedades**, revise as opções de configuração disponíveis, incluindo:

    - **Tipo de grupo de aplicativos preferencial**: esta opção define o tipo de grupo de aplicativos preferencial para o pool de hosts como **Desktop** ou **RemoteApp**. Se os usuários finais tiverem aplicativos RemoteApp e Desktop publicados no pool de hosts, eles verão apenas o tipo de aplicativo selecionado em seu feed.
    - **Iniciar VM na conexão**: habilitar esta opção permite que os usuários iniciem máquinas virtuais individuais no pool de hosts a partir do estado desalocado.
    - **Ambiente de validação**: o pool de hosts de validação é destinado a testar alterações de serviço antes que elas sejam implantadas na produção.
    - **Algoritmo de balanceamento de carga**: esta opção oferece a escolha entre balanceamento de carga em largura e em profundidade. O balanceamento de carga em largura distribui novas sessões de usuário entre todos os hosts de sessão disponíveis no pool de hosts. O balanceamento de carga em profundidade distribui novas sessões de usuário para um host de sessão disponível com o maior número de conexões que não atingiu o limite máximo de sessões.

1. Na página **az140-21-hp1\|Propriedades**, na lista suspensa **Algoritmo de balanceamento de carga**, selecione **Balanceamento em profundidade**.
1. Na caixa de texto **Limite máximo de sessão**, digite **8**.
1. Na página **az140-21-hp1\|Propriedades**, defina **Iniciar VM na conexão** como **Sim**.

    > **Observação**: *iniciar VM ao conectar* permite reduzir custos ao permitir que os usuários finais liguem as VMs (máquinas virtuais) usadas como hosts de sessão somente quando necessário. Para pools de hosts pessoais, *Iniciar VM ao conectar* somente liga uma VM de host de sessão existente que já esteja atribuída ou possa ser atribuída a um usuário. Para pools de hosts agrupados, *Iniciar VM ao conectar* só liga uma VM de host de sessão quando nenhuma está ligada e mais VMs só podem ser ligadas quando a primeira VM atinge o limite de sessão.

1. Na página **az140-21-hp1\|Propriedades**, selecione **Salvar**.

    > **Observação**: usar *Iniciar VM ao Conectar* requer a atribuição da função RBAC (controle de acesso baseado em função) *Colaborador de Ativação de Virtualização da Área de Trabalho* à entidade de serviço *Área de Trabalho Virtual do Azure* no escopo da assinatura do Azure. 

#### Tarefa 3: Atribuir a função RBAC necessária a uma entidade de serviço da Área de Trabalho Virtual do Azure

1. No computador do laboratório, no navegador da Web que exibe o portal do Azure, inicie uma sessão do PowerShell no Azure Cloud Shell.

    > **Observação**: se solicitado, no painel **Introdução**, na lista suspensa **Assinatura**, selecione o nome da assinatura do Azure que você está usando neste laboratório e selecione **Aplicar**.

1. Na sessão do PowerShell no painel Azure Cloud Shell, execute o seguinte comando para recuperar o valor da propriedade Id da assinatura do Azure que você está usando neste laboratório e armazená-lo em uma variável `$subId`:

    ```powershell
    $subId = (Get-AzSubscription).Id
    ```

1. Execute o comando a seguir para criar uma variável $parameters, que armazena uma tabela de hash que contém os valores do nome da definição de função RBAC, o aplicativo Microsoft Entra que representa a entidade de serviço do **Área de Trabalho Virtual do Azure** e o escopo da assinatura:

    ```powershell
    $parameters = @{
        RoleDefinitionName = "Desktop Virtualization Power On Contributor"
        ApplicationId = "9cdead84-a844-4324-93f2-b2e6bb768d07"
        Scope = "/subscriptions/$subId"
    }
    ```

1. Execute o seguinte comando para criar a atribuição de função RBAC:

    ```powershell
    New-AzRoleAssignment @parameters
    ```

1. Feche o painel do Cloud Shell.

#### Tarefa 4: Configurar atualizações agendadas do agente

> **Observação**: o recurso Atualizações do Agente Agendadas permite que você crie até duas janelas de manutenção para as atualizações do agente da Área de Trabalho Virtual do Azure, da pilha lado a lado e do agente do Geneva Monitoring, para que essas atualizações ocorram fora do horário comercial. 

1. No navegador da Web que exibe o portal do Azure, volte para a página do pool de hosts **az140-21-hp1**.
1. Na página **az140-21-hp1**, na barra de menu vertical, na seção **Configurações**, selecione a entrada **Atualizações programadas do agente** e, na página **az140-21-hp1\|Atualizações programadas do agente**, marque a caixa de seleção **Atualizações programadas do agente** .
1. Na seção **Agendar**, marque a caixa de seleção **Usar fuso horário do host de sessão local**.
1. Na seção **Janela de manutenção**, na lista suspensa **Dia**, selecione **Sábado** e, na lista suspensa **Hora**, selecione **11:00 PM**.
1. Escolha **Aplicar**.

#### Tarefa 5: Configurar propriedades RDP do pool de hosts

1. No navegador da Web que exibe o portal do Azure, na página **az140-21-hp1**, na barra de menu vertical, na seção **Configurações**, selecione a entrada **Propriedades do RDP**.
1. Na guia **Informações de conexão** da página **az140-21-hp1\|Propriedades RDP**, revise as opções de configuração disponíveis, incluindo:

    - **Logon único do Microsoft Entra**: esta opção determina se as conexões tentarão aproveitar a autenticação do Microsoft Entra para fazer login em hosts de sessão ingressados no Microsoft Entra e, efetivamente, fornecer uma experiência de logon único. Observe que não é necessário que o computador cliente esteja conectado ao Microsoft Enterprise. 
    - **Provedor de suporte de segurança de credenciais**: esta opção controla o uso do CredSSP para autenticação. O CredSSP oferece a capacidade de encaminhar com segurança as credenciais do usuário do dispositivo cliente para o host da sessão da área de trabalho remota. No entanto, seus recursos não incluem suporte para autenticação Entra ID.
    - **Shell alternativo**: esta opção permite que você especifique um executável para iniciar sempre que uma nova conexão com um host de sessão for estabelecida. Esta configuração se aplica somente a hosts de sessão que executam o Windows Server.
    - **Nome do proxy KDC**: esta opção fornece a capacidade de enviar proxy do tráfego de autenticação Kerberos para controladores de domínio do Active Directory.

    > **Observação**: considerando que três dessas opções não são aplicáveis em nosso cenário (que envolve hosts de sessão ingressados no Microsoft Entra sem nenhuma presença dos Active Directory Domain Services), você configurará apenas a primeira. Esta opção corresponde à propriedade `enablerdsaadauth:i:value` do RDP.

1. Na lista suspensa **Logon único do Microsoft Entra**, selecione a opção **As conexões usarão a autenticação do Microsoft Entra para fornecer logon único** e então selecione **Salvar**.

    > **Importante**: é essencial ter em mente que habilitar essa propriedade RDP específica é apenas uma das várias etapas necessárias para implementar a funcionalidade de logon único. Outras ações aplicáveis a este cenário incluem habilitar a autenticação do Microsoft Entra para RDP no locatário do Entra e configurar grupos de dispositivos, que não são suportados na versão atual do ambiente de laboratório e, portanto, não estão incluídos nas instruções. Para obter a lista completa de ações necessárias para implementar o logon único para o Microsoft Entra ID, consulte [Configurar o logon único para o Área de Trabalho Virtual do Azure usando a autenticação do Microsoft Entra ID](https://learn.microsoft.com/en-us/azure/virtual-desktop/configure-single-sign-on).

1. Na página **az140-21-hp1\|Propriedades do RDP**, selecione a guia **Comportamento da sessão** e revise as opções de configuração disponíveis, incluindo:

    - **Reconexão**: esta opção determina se o computador cliente tentará se reconectar automaticamente ao computador remoto se a conexão for interrompida.
    - **Detecção automática de largura de banda**: esta opção determina se a detecção automática de largura de banda da rede deve ser usada ou não.
    - **Detecção automática de rede**: esta opção permite que você habilite a detecção automática do tipo de rede. Ele é usado em conjunto com a **Detecção automática de largura de banda**. 
    - **Compressão**: esta opção determina se a conexão deve usar compressão em massa.
    - **Reprodução de vídeo**: esta opção permite o uso de streaming multimídia eficiente de RDP para reprodução de vídeo.

1. Na guia **Comportamento da sessão**, na lista suspensa **Reconexão**, selecione **O cliente tenta se reconectar automaticamente** e então selecione **Salvar**.
1. Na página **az140-21-hp1\|Propriedades do RDP**, selecione a guia **Redirecionamento de dispositivo** e revise as opções de configuração disponíveis, incluindo duas categorias principais:

    - **Áudio e vídeo**
    - **Dispositivos e recursos locais**

    > **Observação**: por padrão, o redirecionamento se aplica a todas as unidades de disco, incluindo aquelas que são montadas após a conexão inicial ser estabelecida.

1. Na página **az140-21-hp1\|Propriedades de RDP**, selecione a guia **Configurações de exibição** e revise as opções de configuração disponíveis, incluindo suporte para vários monitores, dimensionamento inteligente e tamanhos específicos de área de trabalho (em pixels). 
1. Na página **az140-21-hp1\|Propriedades do RDP**, selecione a guia **Avançado** e revise as configurações existentes. Observe que essas configurações refletem as alterações feitas anteriormente nesta tarefa.