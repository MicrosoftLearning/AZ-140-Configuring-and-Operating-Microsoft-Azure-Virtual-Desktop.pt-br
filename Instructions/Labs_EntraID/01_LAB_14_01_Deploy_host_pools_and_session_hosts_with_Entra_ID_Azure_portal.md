---
lab:
  title: 'Laboratório: Implantar pools de hosts e hosts de sessão usando o portal do Azure (ID Entra)'
  module: 'Module 1.4: Implement host pools and session hosts'
---

# Laboratório — Implantar pools de hosts e hosts de sessão usando o portal do Azure (ID Entra)
# Manual de laboratório do aluno

## Dependências do laboratório

- O nome da assinatura do Azure que você usará nesse laboratório
- Uma conta de usuário do Microsoft Entra com a função Proprietário na assinatura do Azure que você usará neste laboratório e com permissões suficientes para unir dispositivos ao locatário do Entra associado a essa assinatura do Azure.

## Tempo estimado

60 minutos

## Cenário do laboratório

Você precisa ter uma assinatura do Microsoft Azure. Você precisa implantar o ambiente de Área de Trabalho Virtual do Azure que usa hosts de sessão ingressados no Microsoft Entra.

## Objetivos
  
Depois de realizar este laboratório, você será capaz de:

- Implantar hosts de sessão da Área de Trabalho Virtual do Azure associados ao Microsoft Entra

## Arquivos do laboratório

- Nenhum

## Instruções

### Exercício 1: Implementar um ambiente de Área de Trabalho Virtual do Azure usando hosts de sessão associados ao Microsoft Entra
  
As principais tarefas desse exercício são as seguintes:

1. Preparar a assinatura do Azure para implantação de um pool de hosts da Área de Trabalho Virtual do Azure
1. Configurar uma pool de host da Área de Trabalho Virtual do Azure
1. Criar um grupo de aplicativos da Área de Trabalho Virtual do Azure
1. Criar um workspace da Área de Trabalho Virtual do Azure
1. Conceder acesso aos pools de hosts da Área de Trabalho Virtual do Azure

#### Tarefa 1: Preparar a assinatura do Azure para implantação de um pool de hosts da Área de Trabalho Virtual do Azure

1. No computador do laboratório, inicie um navegador da Web, navegue até o portal do Azure em [https://portal.azure.com](https://portal.azure.com) e entre fornecendo as credenciais de uma conta de usuário com a função Proprietário na assinatura que você usará neste laboratório.

    > **Observação**: use as credenciais da conta `User1-` listada na guia Recursos no lado direito da janela da sessão de laboratório.

1. No portal do Azure, inicie uma sessão do PowerShell no Azure Cloud Shell.

    > **Observação**: se solicitado, no painel **Introdução**, na lista suspensa **Assinatura**, selecione o nome da assinatura do Azure que você está usando neste laboratório e selecione **Aplicar**.

1. Na sessão do PowerShell no painel Azure Cloud Shell, execute o seguinte comando para registrar o provedor de recursos **Microsoft.DesktopVirtualization**:

    ```powershell
    Register-AzResourceProvider -ProviderNamespace Microsoft.DesktopVirtualization
    ```

    > **Observação**: não espere o registro ser concluído. Isso poderá levar alguns minutos.

1. Feche o painel do Cloud Shell.
1. No navegador da Web que exibe o portal do Azure, pesquise e selecione **Redes virtuais** e, na página **Redes virtuais**, selecione **Criar +**
1. Na guia **Noções básicas** da página **Criar rede virtual**, especifique as seguintes configurações e selecione **Avançar**:

    |Configuração|Valor|
    |---|---|
    |Assinatura|O nome da assinatura do Azure que você está usando neste laboratório|
    |Grupo de recursos|O nome de um novo grupo de recursos **az140-11e-RG**|
    |Nome da rede virtual|**az140-vnet11e**|
    |Region|O nome da região do Azure onde você deseja implantar o ambiente de área de trabalho virtual do Azure|

1. Na guia **Segurança**, aceite as configurações padrão e selecione **Avançar**.
1. Na guia **Endereços IP**, aplique as seguintes configurações (modifique o padrão, se necessário):

    |Configuração|Valor|
    |---|---|
    |Espaço de endereços IP|**10.20.0.0/16**|

1. Selecione o ícone de edição (lápis) ao lado da entrada de sub-rede **padrão**, no painel **Editar**, especifique as seguintes configurações (deixe as outras com seus valores existentes) e selecione **Salvar**:

    |Configuração|Valor|
    |---|---|
    |Nome|**hp1-Subnet**|
    |Endereço inicial|**10.20.1.0**|
    |Habilitar sub-rede privada (sem acesso de saída padrão)|Desabilitado|

1. De volta à aba **Endereços IP**, selecione **Revisar + criar** e então, na aba **Revisar + criar**, selecione **Criar**.

    > **Observação**: não espere a conclusão do processo de provisionamento. Essa etapa geralmente leva menos de um minuto.

1. No navegador da Web que exibe o portal do Azure, pesquise e selecione **Microsoft Entra ID**.
1. Na página **Visão geral** do locatário do Microsoft Entra associado à sua assinatura, na seção **Gerenciar** do menu de navegação vertical, selecione **Usuários**.
1. Na página **Usuários**, na caixa de texto **Pesquisar**, insira o nome da `User1-` conta listada na guia Recursos no lado direito da janela da sessão de laboratório.
1. Na lista de resultados da pesquisa, selecione a entrada da conta de usuário com o nome correspondente.
1. Na página que exibe as propriedades da conta de usuário, na seção **Gerenciar** do menu de navegação vertical, selecione **Grupos**.
1. Na página **Grupos**, registre o nome do grupo começando com o prefixo **AVD-DAG** (você precisará dele mais tarde neste laboratório).
1. Navegue de volta para a página **Usuários**, na caixa de texto **Pesquisar**, insira o nome da `User2-` conta listada na guia Recursos no lado direito da janela da sessão de laboratório.
1. Na lista de resultados da pesquisa, selecione a entrada da conta de usuário com o nome correspondente.
1. Na página que exibe as propriedades da conta de usuário, na seção **Gerenciar** do menu de navegação vertical, selecione **Grupos**.
1. Na página **Grupos**, registre o nome do grupo começando com o prefixo **AVD-RemoteApp** (você precisará dele mais tarde neste laboratório).

#### Tarefa 2: Configurar uma pool de host da Área de Trabalho Virtual do Azure

1. No computador do laboratório, no navegador da Web que exibe o portal do Azure, pesquise e selecione **Área de Trabalho Virtual do Azure**. Na página **Área de Trabalho Virtual do Azure**, na seção **Gerenciar** do menu de navegação vertical, selecione **Pools de hosts** e, na página **Área de Trabalho Virtual do Azure \| Pools de hosts**, selecione **+ Criar**. 
1. Na guia **Básico** da página **Criar um pool de hosts**, especifique as seguintes configurações e selecione **Avançar: Hosts de sessão >** (deixe as outras configurações com seus valores padrão):

    |Configuração|Valor|
    |---|---|
    |Assinatura|O nome da assinatura do Azure que você está usando neste laboratório|
    |Grupo de recursos|O nome de um novo grupo de recursos **az140-21e-RG**|
    |Nome do pool de host|**az140-21-hp1**|
    |Localidade|O nome da região do Azure onde você deseja implantar seu ambiente de Área de Trabalho Virtual do Azure|
    |Ambiente de validação|**Não**|
    |Tipo de grupo de aplicativos preferencial|**Desktop**|
    |Tipo de pool de host|**Em pool**|
    |Criar configuração do host de sessão|**Não**|
    |Algoritmo de balanceamento de carga|**Amplitude**|

    > **Observação**: ao usar o algoritmo de balanceamento de carga em largura, o parâmetro limite máximo de sessão é opcional.

1. Na guia **Hosts de sessão** da página **Criar um pool de hosts**, especifique as seguintes configurações e selecione **Avançar: Espaço de trabalho >** (deixe as outras configurações com seus valores padrão):

    > **Observação**: ao definir o valor do **Prefixo do nome**, alterne para a guia Recursos no lado direito da janela da sessão de laboratório e identifique a sequência de caracteres entre *Usuário1-* e o caractere *@*. Use esta cadeia de caracteres para substituir o espaço reservado *random*.

    |Configuração|Valor|
    |---|---|
    |Adicionar máquinas virtuais|**Sim**|
    |Grupo de recursos|**O padrão é o mesmo que o pool de hosts**|
    |Prefixo do nome|**sh**-*random*|
    |Tipo de máquina virtual|**Máquina Virtual do Azure**|
    |Localização da máquina virtual|O nome da região do Azure onde você deseja implantar seu ambiente de Área de Trabalho Virtual do Azure|
    |Opções de disponibilidade|**Nenhuma redundância de infraestrutura necessária**|
    |Tipo de segurança|**Máquinas virtuais de início confiável**|
    |Imagem|**Windows 11 Enterprise multi-sessão, versão 23H2 + Microsoft 365 Apps**|
    |Tamanho da máquina virtual|**Standard DC2s_v3**|
    |Número de VMs|**2**|
    |Tipo de disco de SO|**SSD Standard**|
    |Tamanho do disco de SO|**Tamanho padrão (128 GB)**|
    |Diagnóstico de Inicialização|**Habilitar com a conta de armazenamento gerenciada (recomendado)**|
    |Rede virtual|**az140-vnet11e**|
    |Sub-rede|**hp1-Subnet**|
    |Grupo de segurança de rede|**Basic**|
    |Portas de entrada públicas|**Não**|
    |Selecione o diretório ao qual deseja ingressar|**Microsoft Entra ID**|
    |Registrar a VM com o Intune|**Não**|
    |Nome de usuário|**Aluno**|
    |Senha|Qualquer sequência de caracteres suficientemente complexa que será usada como senha para a conta de administrador interna|
    |Confirmar senha|A mesma sequência de caracteres que você especificou anteriormente|

    > **Observação**: a senha deve ter pelo menos 12 caracteres e consistir em uma combinação de letras minúsculas, letras maiúsculas, dígitos e caracteres especiais. Para obter detalhes, consulte as informações sobre [os requisitos de senha ao criar uma VM do Azure](https://learn.microsoft.com/en-us/azure/virtual-machines/windows/faq#what-are-the-password-requirements-when-creating-a-vm-).

1. Na guia **Workspace** da página **Criar um pool de hosts**, confirme a seguinte configuração e selecione **Revisar + criar**:

    |Configuração|Valor|
    |---|---|
    |Registre o grupo de aplicativos da área de trabalho|**Não**|

1. Na guia **Revisar + criar** da página **Criar um pool de hosts**, selecione **Criar**.

    > **Observação**: aguarde até que a implantação seja concluída. Isso pode levar cerca de 20 minutos.

#### Tarefa 3: Criar um grupo de aplicativos da Área de Trabalho Virtual do Azure

1. No computador do laboratório, no navegador da Web que exibe o portal do Azure, pesquise e selecione **Área de Trabalho Virtual do Azure** e, na página **Área de Trabalho Virtual do Azure**, selecione **Grupos de aplicativos**.
1. Na página **Área de Trabalho Virtual do Azure \| Grupos de aplicativos**, observe o grupo de aplicativos de desktop **az140-21-hp1-DAG** gerado automaticamente e selecione-o. 
1. Na página **az140-21-hp1-DAG**, na seção **Gerenciar** do menu de navegação vertical, selecione **Atribuições**.
1. Na página **az140-21-hp1-DAG \| Atribuições**, selecione **+ Adicionar**.
1. Na página **Selecionar usuários ou grupos de usuários do Microsoft Entra**, selecione **Grupos**, na caixa de pesquisa, digite o nome completo do grupo **AVD-DAG** que você identificou na primeira tarefa deste exercício, marque a caixa de seleção ao lado do nome do grupo e clique em **Selecionar**.
1. Navegue de volta para a página **Área de Trabalho Virtual do Azure \| Grupos de aplicativos** e selecione **+ Criar**. 
1. Na guia **Básico** da página **Criar um grupo de aplicativos**, especifique as seguintes configurações e selecione **Avançar: Aplicativos>**:

    |Configuração|Valor|
    |---|---|
    |Assinatura|O nome da assinatura do Azure que você está usando neste laboratório|
    |Grupo de recursos|**az140-21e-RG**|
    |Pool de host|**az140-21-hp1**|
    |Tipo de grupo de aplicativos|**Aplicativo Remoto**|
    |Nome do grupo de aplicativos|**az140-21-hp1-Office365-RAG**|

1. Na guia **Aplicativos** da página **Criar um grupo de aplicativos**, selecione **+ Adicionar aplicativos**.
1. Na página **Adicionar aplicativo**, especifique as seguintes configurações e selecione **Revisar + adicionar**, depois selecione **Adicionar**:

    |Configuração|Valor|
    |---|---|
    |Origem do aplicativo|**Menu Iniciar**|
    |Aplicativo|**Palavra**|
    |Nome de exibição|**Microsoft Word**|
    |Descrição|**Microsoft Word**|
    |Exigir linha de comando|**Não**|

1. De volta à guia **Aplicativos** da página **Criar um grupo de aplicativos**, selecione **+ Adicionar aplicativos**.
1. Na página **Adicionar aplicativo**, especifique as seguintes configurações e selecione **Revisar + adicionar**, depois selecione **Adicionar**:

    |Configuração|Valor|
    |---|---|
    |Origem do aplicativo|**Menu Iniciar**|
    |Aplicativo|**Excel**|
    |Nome de exibição|**Microsoft Excel**|
    |Descrição|**Microsoft Excel**|
    |Exigir linha de comando|**Não**|

1. De volta à guia **Aplicativos** da página **Criar um grupo de aplicativos**, selecione **+ Adicionar aplicativos**.
1. Na página **Adicionar aplicativo**, especifique as seguintes configurações e selecione **Revisar + adicionar**, depois selecione **Adicionar**:

    |Configuração|Valor|
    |---|---|
    |Origem do aplicativo|**Menu Iniciar**|
    |Aplicativo|**PowerPoint**|
    |Nome de exibição|**Microsoft PowerPoint**|
    |Descrição|**Microsoft PowerPoint**|
    |Exigir linha de comando|**Não**|

1. De volta à guia **Aplicativos** da página **Criar um grupo de aplicativos**, selecione **Avançar: Atribuições >**.
1. Na guia **Atribuições** da página **Criar um grupo de aplicativos**, selecione **+ Adicionar usuários ou grupos de usuários do Microsoft Entra**.
1. Na página **Selecionar usuários ou grupos de usuários do Microsoft Entra**, selecione **Grupos**, digite o nome completo do grupo **AVD-RemoteApp** que você identificou na primeira tarefa deste exercício, marque a caixa de seleção ao lado do nome do grupo e clique em **Selecionar**.
1. De volta à guia **Atribuições** da página **Criar um grupo de aplicativos**, selecione **Avançar: Workspace >**.
1. Na guia **Espaço de trabalho** da página **Criar um espaço de trabalho**, especifique a seguinte configuração e selecione **Revisar + criar**:

    |Configuração|Valor|
    |---|---|
    |Registrar o grupo de aplicativos|**Não**|

1. Na guia **Revisar + criar** da página **Criar um grupo de aplicativos**, selecione **Criar**.

    > **Observação**: aguarde até que o Grupo de Aplicativos seja criado. Isso deverá levar menos de 1 minuto. 

    > **Observação**: Em seguida, você criará um grupo de aplicativos com base no caminho do arquivo como a origem do aplicativo.

1. No navegador da Web que exibe o portal do Azure, pesquise e selecione **Área de Trabalho Virtual do Azure** e, na página **Área de Trabalho Virtual do Azure**, selecione **Grupos de aplicativos**.
1. Na página **Área de Trabalho Virtual do Azure \| Grupos de aplicativos**, selecione **+ Criar**. 
1. Na guia **Básico** da página **Criar um grupo de aplicativos**, especifique as seguintes configurações e selecione **Avançar: Aplicativos>**:

    |Configuração|Valor|
    |---|---|
    |Assinatura|O nome da assinatura do Azure que você está usando neste laboratório|
    |Grupo de recursos|**az140-21e-RG**|
    |Pool de host|**az140-21-hp1**|
    |Tipo de grupo de aplicativos|**RemoteApp**|
    |Nome do grupo de aplicativos|**az140-21-hp1-Utilities-RAG**|

1. Na guia **Aplicativos** da página **Criar um grupo de aplicativos**, selecione **+ Adicionar aplicativos**.
1. Na página **Adicionar aplicativo**, na guia **Básico**, especifique as seguintes configurações e selecione **Avançar**:

    |Configuração|Valor|
    |---|---|
    |Origem do aplicativo|**Caminho do arquivo**|
    |Caminho do aplicativo|**C:\Windows\system32\cmd.exe**|
    |Identificador de aplicativo|**Prompt de comando**|
    |Nome de exibição|**Prompt de comando**|
    |Descrição|**Windows Command Prompt**|
    |Exigir linha de comando|**Não**|

1. Na guia **Ícone**, especifique as seguintes configurações e selecione **Examinar + adicionar** e, em seguida, selecione **Adicionar**:

    |Configuração|Valor|
    |---|---|
    |Caminho do ícone|**C:\Windows\system32\cmd.exe**|
    |Índice do ícone|0|

1. De volta à guia **Aplicativos** da página **Criar um grupo de aplicativos**, selecione **Avançar: Atribuições >**.
1. Na guia **Atribuições** da página **Criar um grupo de aplicativos**, selecione **+ Adicionar usuários ou grupos de usuários do Microsoft Entra**.
1. Na página **Selecionar usuários ou grupos de usuários do Microsoft Entra**, selecione **Grupos**, digite o nome completo do grupo **AVD-RemoteApp** que você identificou na primeira tarefa deste exercício, marque a caixa de seleção ao lado do nome do grupo e clique em **Selecionar**.
1. De volta à guia **Atribuições** da página **Criar um grupo de aplicativos**, selecione **Avançar: Workspace >**.
1. Na guia **Espaço de trabalho** da página **Criar um espaço de trabalho**, especifique a seguinte configuração e selecione **Revisar + criar**:

    |Configuração|Valor|
    |---|---|
    |Registrar o grupo de aplicativos|**Não**|

1. Na guia **Revisar + criar** da página **Criar um grupo de aplicativos**, selecione **Criar**.

    > **Observação**: aguarde até que o Grupo de Aplicativos seja criado. Isso deverá levar menos de 1 minuto. 

#### Tarefa 4: Criar um workspace da Área de Trabalho Virtual do Azure

1. No computador do laboratório, no navegador da Web que exibe o portal do Azure, pesquise e selecione **Área de Trabalho Virtual do Azure** e, na página **Área de Trabalho Virtual do Azure**, selecione **Workspaces**.
1. Na página **Área de Trabalho Virtual do Azure \| Workspaces**, selecione **+ Criar**. 
1. Na guia **Básico** da página **Criar um workspace**, especifique as seguintes configurações e selecione **Avançar: Grupos de aplicativos >**:

    |Configuração|Valor|
    |---|---|
    |Assinatura|O nome da assinatura do Azure que você está usando neste laboratório|
    |Grupo de recursos|**az140-21e-RG**|
    |Nome do workspace|**az140-21-ws1**|
    |Nome amigável|**az140-21-ws1**|
    |Localidade|O nome da região do Azure na qual você implantou recursos no primeiro exercício deste laboratório ou uma região próxima a ela|

1. Na guia **Grupos de aplicativos** da página **Criar um workspace**, especifique as seguintes configurações:

    |Configuração|Valor|
    |---|---|
    |Registrar grupos de aplicativos|**Sim**|

1. Na guia **Workspace** da página **Criar um workspace**, selecione **+ Registrar grupos de aplicativos**.
1. Na página **Adicionar grupos de aplicativos**, selecione o sinal de mais ao lado das entradas **az140-21-hp1-DAG**, **az140-21-hp1-Office365-RAG** e **az140-21-hp1-Utilities-RAG** e clique em **Selecionar**. 
1. De volta à guia **Grupos de aplicativos** da página **Criar um workspace**, selecione **Revisar + criar**.
1. Na guia **Revisar + criar** da página **Criar um workspace**, selecione **Criar**.

#### Tarefa 5: Conceder acesso aos pools de hosts da Área de Trabalho Virtual do Azure

> **Observação**: ao usar hosts de sessão ingressados no Microsoft Entra, você precisa atribuir aos usuários e administradores da Área de Trabalho Virtual do Azure funções apropriadas de RBAC (controle de acesso baseado em função) do Azure. Em particular, a função *Login de usuário de máquina virtual* é necessária para fazer login em hosts de sessão e a função *Login de administrador de máquina virtual* é necessária para os privilégios administrativos locais. 

1. No computador do laboratório, no navegador da Web que exibe o portal do Azure, pesquise e selecione **Grupos de recursos** e, na página **Grupos de recursos**, selecione **az140-21e-RG**.
1. Na página **az140-21e-RG**, no menu de navegação vertical, selecione **Controle de acesso (IAM)**.
1. Na página **az140-21e-RG\|Controle de acesso (IAM)**, selecione **+ Adicionar** e, no menu suspenso, selecione **Adicionar atribuição de função**.
1. Na guia **Função** da página **Adicionar atribuição de função**, certifique-se de que a guia **Funções de função de trabalho** esteja selecionada, na caixa de texto de pesquisa, insira **Login de usuário de máquina virtual**, na lista de resultados, selecione **Login de usuário de máquina virtual** e, em seguida, selecione **Avançar**.
1. Na guia **Membros** da página **Adicionar atribuição de função**, certifique-se de que a opção **Usuário, grupo ou entidade de serviço** esteja selecionada, clique em **+ Selecionar membros**, no painel **Selecionar membros**, localize o grupo **AVD-RemoteApp** que você identificou na primeira tarefa deste exercício e clique em **Selecionar**.
1. De volta à guia **Membros** da página **Adicionar atribuição de função**, selecione **Avançar**.
1. Na guia **Tipo de atribuição** da página **Adicionar atribuição de função**, defina o **Tipo de atribuição** como **Ativo** e selecione **Revisar + atribuir**.
1. Na guia **Revisar + atribuir** da página **Adicionar atribuição de função**, selecione **Revisar + atribuir**. 
1. De volta à página **az140-21e-RG\|Controle de acesso (IAM)**, selecione **+ Adicionar** e, no menu suspenso, selecione **Adicionar atribuição de função**.
1. Na guia **Função** da página **Adicionar atribuição de função**, certifique-se de que a guia **Funções de função de trabalho** esteja selecionada, na caixa de texto de pesquisa, insira **Login do administrador da máquina virtual**, na lista de resultados, selecione **Login do administrador da máquina virtual** e, em seguida, selecione **Avançar**.
1. Na guia **Membros** da página **Adicionar atribuição de função**, certifique-se de que a opção **Usuário, grupo ou entidade de serviço** esteja selecionada, clique em **+ Selecionar membros**, no painel **Selecionar membros**, localize o grupo **AVD-DAG** que você identificou na primeira tarefa deste exercício e clique em **Selecionar**.
1. De volta à guia **Membros** da página **Adicionar atribuição de função**, defina **Tipo de atribuição** como **Ativo**, selecione **Revisar + atribuir** e **Revisar + atribuir** novamente. 
