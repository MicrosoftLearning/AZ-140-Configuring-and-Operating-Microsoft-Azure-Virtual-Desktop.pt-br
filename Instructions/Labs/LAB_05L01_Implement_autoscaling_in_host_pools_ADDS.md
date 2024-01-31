---
lab:
  title: 'Laboratório: Implementar o dimensionamento automático em pools de hosts (AD DS)'
  module: 'Module 5: Monitor and Maintain an AVD Infrastructure'
---

# Laboratório: Implementar o dimensionamento automático em pools de hosts (AD DS)
# Manual de laboratório do aluno

## Dependências do laboratório

- O nome da assinatura do Azure que você usará nesse laboratório
- Uma conta Microsoft ou uma conta do Microsoft Entra com a função de Proprietário ou Colaborador na assinatura do Azure que você usará neste laboratório e com a função de Administrador Global no locatário do Microsoft Entra associado a essa assinatura do Azure.
- O laboratório **Preparar para a implantação da Área de Trabalho Virtual do Azure (AD DS)** concluído
- O laboratório **Implantar pools de hosts e hosts de sessão usando o portal do Azure (AD DS)** concluído

## Tempo estimado

60 minutos

## Cenário do laboratório

Você precisa configurar o dimensionamento automático de hosts de sessão da Área de Trabalho Virtual do Azure em um ambiente do Active Directory Domain Services (AD DS).

## Objetivos
  
Depois de realizar este laboratório, você será capaz de:

- Configurar o dimensionamento automático de hosts de sessão da Área de Trabalho Virtual do Azure
- Verificar o dimensionamento automático de hosts de sessão da Área de Trabalho Virtual do Azure

## Arquivos do laboratório

- Nenhum

## Instruções

### Exercício 1: Configurar o dimensionamento automático de hosts de sessão da Área de Trabalho Virtual do Azure

As principais tarefas desse exercício são as seguintes:

1. Preparar para dimensionar hosts de sessão da Área de Trabalho Virtual do Azure
2. Criar um plano de dimensionamento para hosts de sessão da Área de Trabalho Virtual do Azure

#### Tarefa 1: Preparar para dimensionar hosts de sessão da Área de Trabalho Virtual do Azure

1. No seu computador de laboratório, inicie um navegador da Web, navegue até o [portal do Azure](https://portal.azure.com) e entre fornecendo credenciais de uma conta de usuário com a função Proprietário na assinatura que você usará nesse laboratório.
1. No computador do laboratório, na janela do navegador da Web que exibe o portal do Azure, abra a sessão do shell do **PowerShell** no painel do **Cloud Shell**.

   >**Observação**: os pools de host que você planeja usar com o dimensionamento automático devem ser configurados com um valor não padrão do parâmetro **MaxSessionLimit**. Você pode definir esse valor nas configurações do pool de host no portal do Azure ou executando os cmdlets **Update-AzWvdHostPool** do Azure PowerShell, como neste exemplo. Você também pode defini-lo explicitamente ao criar um pool no portal do Azure ou ao executar o cmdlet **New-AzWvdHostPool** do Azure PowerShell.

1. Na sessão do PowerShell no painel do Cloud Shell, execute o seguinte comando para definir o valor do parâmetro **MaxSessionLimit** do pool de hosts **az140-21-hp1** como **2**: 

   ```powershell
   Update-AzWvdHostPool -ResourceGroupName 'az140-21-RG' `
   -Name az140-21-hp1 `
   -MaxSessionLimit 2
   ```

   >**Observação**: Neste laboratório, o valor do parâmetro **MaxSessionLimit** é definido artificialmente baixo para facilitar o acionamento do comportamento de dimensionamento automático.

   >**Observação**: Antes de criar seu primeiro plano de dimensionamento, você precisará atribuir a função **RBAC do Colaborador do Power On Off da Virtualização de Área de Trabalho à Área de Trabalho Virtual do Azure** com sua assinatura do Azure como o escopo de destino. 

1. Na janela do navegador que exibe o portal do Azure, feche o painel do Cloud Shell.
1. No portal do Azure, pesquise e selecione **Assinaturas** e, na lista de assinaturas, selecione a que contém os recursos da Área de Trabalho Virtual do Azure. 
1. Na página de assinatura, selecione **Controle de acesso (IAM).**
1. Na página **Controle de acesso (IAM)**, na barra de ferramentas, selecione o botão **+ Adicionar** e, em seguida, selecione **Adicionar atribuição de função** no menu suspenso.
1. Na guia **Função** do assistente **Adicionar atribuição de função**, pesquise e selecione a função **Colaborador do Power On Off de Virtualização da Área de Trabalho** e clique em **Avançar**.
1. Na guia **Membros** do assistente **Adicionar atribuição de função**, selecione **+ Selecionar membros**, pesquise e selecione**Área de Trabalho Virtual do Azure** ou **Área de Trabalho Virtual do Windows**, clique em ** Selecionar** e clique em **Avançar**.

   >**Observação**: o valor depende de quando o provedor de recursos **Microsoft.DesktopVirtualization** foi registrado pela primeira vez em seu locatário do Azure.

1. Na guia **Revisão + atribuição**, selecione **Examinar + atribuir**.

#### Tarefa 2: Criar um plano de dimensionamento para hosts de sessão da Área de Trabalho Virtual do Azure

1. No computador do laboratório, no navegador que exibe o portal do Azure, pesquise e selecione a **Área de Trabalho Virtual do Azure**. 
1. Na página **Área de Trabalho Virtual do Azure**, selecione **Dimensionamento de Planos** e, em seguida, selecione **+ Criar**.
1. Na guia **Noções básicas** do assistente **Criar um plano de dimensionamento**, especifique as informações a seguir e selecione **Próximos Agendamentos >** (deixe as outras com seus valores padrão):

   |Configuração|Valor|
   |---|---|
   |Grupo de recursos|o nome **az140-51-RG** de um novo grupo de recursos|
   |Nome|**az140-51-scaling-plan**|
   |Localidade|a mesma região do Azure na qual você implantou os hosts de sessão nos laboratórios anteriores|
   |Nome amigável|**plano de dimensionamento az140-51**|
   |Fuso horário|seu fuso horário local|

   >**Observação**: As marcas de exclusão permitem designar um nome de marca para hosts de sessão que você deseja excluir das operações de dimensionamento. Por exemplo, você pode marcar as VMs definidas com o modo esvaziar para que o dimensionamento automático não substitua o modo esvaziar durante a manutenção usando a marca de exclusão "excludeFromScaling". 

1. Na guia **Agendamentos**do assistente **Criar um plano de dimensionamento**, selecione **+ Adicionar agendamento**.
1. Na guia **Geral** do assistente **Adicionar agendamento**, especifique as informações a seguir e clique em **Avançar**.

   |Configuração|Valor|
   |---|---|
   |Nome da agenda|**az140-51-schedule**|
   |Repetir Em|**Sete selecionados** (selecione todos os dias da semana)|

1. Na guia **Acelerar** do assistente **Adicionar agendamento**, especifique as informações a seguir e clique em **Avançar**.

   |Configuração|Valor|
   |---|---|
   |Hora de início (sistema de 24 horas)|sua hora atual menos 9 horas|
   |Algoritmo de balanceamento de carga|**Amplitude primeiro**|
   |Percentual mínimo de hosts (%)|**20**|
   |Limite de capacidade (%)|**60**|

   >**Observação**: A preferência de balanceamento de carga selecionada aqui substituirá a que você selecionou para as configurações originais do pool de hosts.

   >**Observação**: O percentual mínimo de hosts designa a porcentagem de hosts de sessão em que você deseja que sempre permaneçam ativas. Se a porcentagem inserida não for um número inteiro, ela será arredondada para o número inteiro mais próximo. 

   >**Observação**: O limite de capacidade representa a porcentagem da capacidade do pool de hosts disponível que irá disparar uma ação de dimensionamento a ser realizada. Por exemplo, se dois hosts de sessão no pool de host com um limite máximo de sessão de 20 forem ativados, a capacidade do pool de host disponível será de 40. Se você definir o limite de capacidade como 75% e os hosts de sessão tiverem mais de 30 sessões de usuário, o dimensionamento automático ativará um terceiro host de sessão. Isso irá alterar a capacidade do pool de hosts disponível de 40 para 60.

1. Na guia **Horário de Pico** do assistente **Adicionar agendamento**, especifique as informações a seguir e clique em **Avançar**.

   |Configuração|Valor|
   |---|---|
   |Hora de início (sistema de 24 horas)|sua hora atual menos 8 horas|
   |Algoritmo de balanceamento de carga|**Profundidade**|

   >**Observação**: a hora de início designa a hora de término para a fase de aceleração.

   >**Observação**: O valor do limite de capacidade nesta fase é determinado pelo valor do limite de capacidade de aumento.

1. Na guia **Desacelerar** do assistente **Adicionar agendamento**, especifique as informações a seguir e clique em **Avançar**.

   |Configuração|Valor|
   |---|---|
   |Hora de início (sistema de 24 horas)|sua hora atual menos 2 horas|
   |Algoritmo de balanceamento de carga|**Profundidade**|
   |Percentual mínimo de hosts (%)|**10**|
   |Limite de capacidade (%)|**90**|
   |Forçar o logoff de usuários|**Sim**|
   |Tempo de atraso antes de registrar em log usuários e desligar VMs (min)|**30**|

   >**Observação**: se **Forçar saída de usuários** estiver habilitada, o dimensionamento automático colocará o host de sessão com o menor número de sessões de usuário no modo de esvaziamento, enviará a todas as sessões de usuário ativas uma notificação sobre o desligamento iminente e os desconectará à força após o tempo de atraso especificado. O dimensionamento automático desconecta todas as sessões de usuário, ele Desaloca a VM. 

   >**Observação**: se você não tiver habilitado a saída forçada durante a desaceleração, os hosts de sessão sem sessões ativas ou desconectadas serão desalocados.

1. Na guia **Fora do horário de pico** do assistente **Adicionar agendamento**, especifique as informações a seguir e clique em **Adicionar**.

   |Configuração|Valor|
   |---|---|
   |Hora de início (sistema de 24 horas)|sua hora atual menos 1 hora|
   |Algoritmo de balanceamento de carga|**Profundidade**|

   >**Observação**: O valor do limite de capacidade nesta fase é determinado pelo valor do limite de capacidade de desaceleração.

1. De volta à guia **Agendamentos** do assistente **Criar um plano de dimensionamento**, selecione **Avançar: Atribuições do pool de host >**:
1. Na página **Atribuições do pool de host**, na lista suspensa **Selecionar pool de hosts**, selecione **az140-21-hp1**. Verifique se a caixa de seleção **Habilitar dimensionamento automático** está habilitada, selecione **Examinar + criar** e, em seguida, selecione **Criar**.


### Exercício 2: Verificar o dimensionamento automático de hosts de sessão da Área de Trabalho Virtual do Azure

As principais tarefas desse exercício são as seguintes:

1. Configurar o diagnóstico para acompanhar o dimensionamento automático da Área de Trabalho Virtual do Azure
1. Verificar o dimensionamento automático de hosts de sessão da Área de Trabalho Virtual do Azure

#### Tarefa 1: Configurar o diagnóstico para acompanhar o dimensionamento automático da Área de Trabalho Virtual do Azure

1. No computador do laboratório, na janela do navegador da Web que exibe o portal do Azure, abra a sessão do shell do **PowerShell** no painel do **Cloud Shell**.

   >**Observação**: Você usará uma conta de Armazenamento do Azure para armazenar eventos de dimensionamento automático. Você pode criá-lo diretamente no portal do Azure ou usar o Azure PowerShell, conforme ilustrado nesta tarefa.

1. Na sessão do PowerShell no painel do Cloud Shell, execute os seguintes comandos para criar uma conta de Armazenamento do Azure:

   ```powershell
   $resourceGroupName = 'az140-51-RG'
   $location = (Get-AzResourceGroup -ResourceGroupName $resourceGroupName).Location
   $suffix = Get-Random
   $storageAccountName = "az140st51$suffix"
   New-AzStorageAccount -Location $location -Name $storageAccountName -ResourceGroupName $resourceGroupName -SkuName Standard_LRS
   ```

   >**Observação**: Aguarde até que a conta de armazenamento seja provisionada.

1. Na janela do navegador que exibe o portal do Azure, feche o painel do Cloud Shell.
1. No computador do laboratório, no navegador que exibe o portal do Azure, navegue até a página do plano de dimensionamento criado no exercício anterior.
1. Na página **az140-51-scaling-plan**, selecione **Configurações de diagnóstico** e, em seguida, selecione **+ Adicionar configuração de diagnóstico**.
1. Na página **Configuração de diagnóstico**, na caixa de texto **Nome da configuração de diagnóstico**, digite **az140-51-scaling-plan-diagnostics** e, na seção **Grupos de categorias,** selecione**allLogs **. 
1. Na mesma página, na seção **Detalhes de destino**, selecione**Arquivar em uma conta de armazenamento** e, na lista suspensa **Conta de armazenamento**, selecione o nome da conta de armazenamento que começa com o prefixo **az140st51**. 
1. Selecione **Salvar**.

#### Tarefa 2: Verificar o dimensionamento automático de hosts de sessão da Área de Trabalho Virtual do Azure

1. No computador do laboratório, na janela do navegador da Web que exibe o portal do Azure, abra a sessão do shell do **PowerShell** no painel do **Cloud Shell**.
1. Na sessão do PowerShell no painel do Cloud Shell, execute o seguinte comando para iniciar as VMs do Azure host da sessão da Área de Trabalho Virtual do Azure que você usará nesse laboratório:

   ```powershell
   Get-AzVM -ResourceGroup 'az140-21-RG' | Start-AzVM
   ```

   >**Observação**: Aguarde até que as VMs do Azure do host da sessão estejam em execução.

1. No computador do laboratório, na janela do navegador da Web que exibe o portal do Azure, navegue até a página do pool de hosts **az140-21-hp1**.
1. Na página **az140-21-hp1**, selecione **Hosts de sessão**.
1. Aguarde até que pelo menos um host de sessão esteja listado com o status **Desligar**.

   >**Observação**: Talvez seja necessário atualizar a página para atualizar o status dos hosts de sessão.

   >**Observação**: Se todos os hosts de sessão permanecerem disponíveis, navegue de volta para a página **az140-51-scaling-plan** e reduza o valor da configuração **porcentagem mínima de hosts (%) ** ** Desacelerar**.

   >**Observação**: Depois que o status de um ou mais hosts de sessão for alterado, os logs de dimensionamento automático deverão estar disponíveis na conta de Armazenamento do Azure. 

1. No portal do Azure, pesquise e selecione **Contas de armazenamento** e, na página **Contas de armazenamento**, selecione a entrada que representa a conta de armazenamento criada anteriormente neste exercício, cujo nome começa com o prefixo **az140st51**.
1. Na página Conta de armazenamento, selecione **Contêineres**.
1. Na lista de contêineres, selecione **insights-logs-autoscale**.
1. Na página **insights-logs-autoscale**, navegue pela hierarquia de pastas até chegar à entrada que representa um blob formatado em JSON armazenado no contêiner.
1. Selecione a entrada de blob, selecione o ícone de reticências na extrema direita da página e, no menu suspenso, selecione **Baixar**.
1. Em seu computador de laboratório, abra o blob baixado em um editor de texto de sua escolha e examine seu conteúdo. Você deve ser capaz de encontrar referências a eventos de dimensionamento automático. 

   >**Observação**: aqui está um conteúdo de blob de exemplo que inclui referências a eventos de dimensionamento automático:

   ```json
   host_Ring    "R0"
   Level    4
   ActivityId   "00000000-0000-0000-0000-000000000000"
   time "2023-03-26T19:35:46.0074598Z"
   resourceId   "/SUBSCRIPTIONS/AAAAAAAE-0000-1111-2222-333333333333/RESOURCEGROUPS/AZ140-51-RG/PROVIDERS/MICROSOFT.DESKTOPVIRTUALIZATION/SCALINGPLANS/AZ140-51-SCALING-PLAN"
   operationName    "ScalingEvaluationSummary"
   category "Autoscale"
   resultType   "Succeeded"
   level    "Informational"
   correlationId    "ddd3333d-90c2-478c-ac98-b026d29e24d5"
   properties   
   Message  "Active session hosts are at 0.00% capacity (0 sessions across 3 active session hosts). This is below the minimum capacity threshold of 90%. 2 session hosts can be drained and deallocated."
   HostPoolArmPath  "/subscriptions/aaaaaaaa-0000-1111-2222-333333333333/resourcegroups/az140-21-rg/providers/microsoft.desktopvirtualization/hostpools/az140-21-hp1"
   ScalingEvaluationStartTime   "2023-03-26T19:35:43.3593413Z"
   TotalSessionHostCount    "3"
   UnhealthySessionHostCount    "0"
   ExcludedSessionHostCount "0"
   ActiveSessionHostCount   "3"
   SessionCount "0"
   CurrentSessionOccupancyPercent   "0"
   CurrentActiveSessionHostsPercent "100"
   Config.ScheduleName  "az140-51-schedule"
   Config.SchedulePhase "OffPeak"
   Config.MaxSessionLimitPerSessionHost "2"
   Config.CapacityThresholdPercent  "90"
   Config.MinActiveSessionHostsPercent  "5"
   DesiredToScaleSessionHostCount   "-2"
   EligibleToScaleSessionHostCount  "1"
   ScalingReasonType    "DeallocateVMs_BelowMinSessionThreshold"
   BeganForceLogoffOnSessionHostCount   "0"
   BeganDeallocateVmCount   "1"
   BeganStartVmCount    "0"
   TurnedOffDrainModeCount  "0"
   TurnedOnDrainModeCount   "1"
   ```


### Exercício 3: Parar e desalocar VMs do Azure provisionadas no laboratório

As principais tarefas desse exercício são as seguintes:

1. Parar e desalocar VMs do Azure provisionadas no laboratório

>**Observação**: Neste exercício, você desalocará as VMs do Azure usadas nesse laboratório para minimizar os encargos de computação correspondentes.

#### Tarefa 1: Desalocar VMs do Azure provisionadas no laboratório

1. Alterne para o computador de laboratório e, na janela do navegador da Web que exibe o portal do Azure, abra uma sessão do **PowerShell** no painel do **Cloud Shell**.
1. Na sessão do PowerShell no painel do Cloud Shell, execute o seguinte comando para listar todas as VMs do Azure usadas neste laboratório:

   ```powershell
   Get-AzVM -ResourceGroup 'az140-21-RG'
   ```

1. Na sessão do PowerShell no painel do Cloud Shell, execute o seguinte comando para parar e desalocar todas as VMs do Azure usadas neste laboratório:

   ```powershell
   Get-AzVM -ResourceGroup 'az140-21-RG' | Stop-AzVM -NoWait -Force
   ```

   >**Observação**: O comando é executado de modo assíncrono (conforme determinado pelo parâmetro -NoWait), portanto, embora você possa executar outro comando do PowerShell imediatamente depois na mesma sessão do PowerShell, levará alguns minutos antes de o grupo de recursos ser de fato removido.
