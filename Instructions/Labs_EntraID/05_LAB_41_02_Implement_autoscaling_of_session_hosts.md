---
lab:
  title: 'Laboratório: Implementar dimensionamento automático de hosts de sessão'
  module: 'Module 4.1: Monitor and manage Azure Virtual Desktop services'
---

# Laboratório — Implementar e monitorar o dimensionamento automático de hosts de sessão
# Manual de laboratório do aluno

## Dependências do laboratório

- O nome da assinatura do Azure que você usará nesse laboratório
- Uma conta de usuário do Microsoft Entra com a função Proprietário ou Colaborador na assinatura do Azure que você usará neste laboratório e com permissões suficientes para unir dispositivos ao locatário do Entra associado a essa assinatura do Azure.
- Ter concluído o laboratório *Implantar pools de hosts e hosts de sessão usando o portal do Azure (AD DS)*
- O laboratório *Gerenciar pools de hosts e hosts de sessão usando o portal do Azure (ID Entra)* foi concluído
- O laboratório *Conectar aos hosts da sessão (ID Entra)* foi concluído

## Tempo estimado

45 minutos

## Cenário do laboratório

Você tem um ambiente de Área de Trabalho Virtual do Azure cujo uso muda regularmente. Você quer minimizar o custo aproveitando a funcionalidade dos planos de dimensionamento de dimensionamento automático.

## Objetivos
  
Depois de realizar este laboratório, você será capaz de:

- Implementar e avaliar o dimensionamento automático da Área de Trabalho Virtual do Azure

## Arquivos do laboratório

- Nenhum

## Instruções

### Exercício 1: Implementar planos de dimensionamento automático da Área de Trabalho Virtual do Azure
  
As principais tarefas desse exercício são as seguintes:

1. Atribuir a função RBAC necessária a uma entidade de serviço da Área de Trabalho Virtual do Azure
1. Parar e desalocar todos os hosts de sessão
1. Ajustar as configurações do pool de hosts
1. Criar um plano de dimensionamento
1. Avalie a funcionalidade de dimensionamento automático
1. Desabilitar o dimensionamento automático do pool de host

#### Tarefa 1: Atribuir a função RBAC necessária a uma entidade de serviço da Área de Trabalho Virtual do Azure

> **Observação**: para que os planos de dimensionamento automático funcionem, você precisa conceder à entidade de serviço da Área de Trabalho Virtual do Azure as permissões para gerenciar o estado de energia das VMs do host da sessão. Essas permissões podem ser concedidas usando a função RBAC integrada **Desktop Virtualization Power On Off Contributor**. É importante ter em mente que a atribuição de função deve ser executada no escopo da assinatura. A atribuição dessa função em qualquer nível inferior à sua assinatura, como o grupo de recursos, o pool de host ou a VM, impedirá que o dimensionamento automático funcione de modo adequado. 

> **Observação**: esta função é diferente daquela (**Colaborador de Ativação de Virtualização da Área de Trabalho**) usada no laboratório *Gerenciar pools de hosts e hosts de sessão usando o portal do Azure (ID Entra)*, que era necessária para dar suporte à funcionalidade *Iniciar VM ao Conectar*.

1. Se necessário, no computador do laboratório, inicie um navegador da Web, navegue até o portal do Azure e entre fornecendo credenciais de uma conta de usuário com a função Proprietário na assinatura que você usará neste laboratório.

    > **Observação**: use as credenciais da conta `User1-` listada na guia Recursos no lado direito da janela da sessão de laboratório.

1. No computador do laboratório, no navegador da Web que exibe o portal do Azure, inicie uma sessão do PowerShell no Azure Cloud Shell.

    > **Observação**: se solicitado, no painel **Introdução**, na lista suspensa **Assinatura**, selecione o nome da assinatura do Azure que você está usando neste laboratório e selecione **Aplicar**.

1. Na sessão do PowerShell no painel Azure Cloud Shell, execute o seguinte comando para recuperar o valor da propriedade Id da assinatura do Azure que você está usando neste laboratório e armazená-lo em uma variável `$subId`:

    ```powershell
    $subId = (Get-AzSubscription).Id
    ```

1. Execute o comando a seguir para criar uma variável $parameters, que armazena uma tabela de hash que contém os valores do nome da definição de função RBAC, o aplicativo Microsoft Entra que representa a entidade de serviço do **Área de Trabalho Virtual do Azure** e o escopo da assinatura:

    ```powershell
    $parameters = @{
        RoleDefinitionName = "Desktop Virtualization Power On Off Contributor"
        ApplicationId = "9cdead84-a844-4324-93f2-b2e6bb768d07"
        Scope = "/subscriptions/$subId"
    }
    ```

1. Execute o seguinte comando para criar a atribuição de função RBAC:

    ```powershell
    New-AzRoleAssignment @parameters
    ```

1. Feche o painel do Cloud Shell.

#### Tarefa 2: Parar e desalocar todos os hosts de sessão

> **Observação**: para avaliar a funcionalidade de dimensionamento automático, você interromperá e desalocará todos os hosts de sessão no ambiente de Área de Trabalho Virtual do Azure. 

1. No computador do laboratório, no navegador da Web que exibe o portal do Azure, pesquise e selecione **Área de Trabalho Virtual do Azure** e, na página **Área de Trabalho Virtual do Azure**, na seção **Gerenciar** da barra de menu vertical, selecione **Pools de hosts**.
1. Na página **Área de Trabalho Virtual do Azure \| Pools de hosts**, na lista de pools de hosts, selecione **az140-21-hp1**.
1. Na página **az140-21-hp1**, na barra de menu vertical, na seção **Gerenciar**, selecione **Hosts de sessão**.
1. Na página **az140-21-hp1 \| Hosts de sessão**, marque a caixa de seleção ao lado de cada nome de host de sessão e selecione **Parar**.
1. Quando solicitado a confirmar, na janela pop-up **Parar hosts de sessão**, selecione **Parar**.

    > **Observação**: talvez seja necessário selecionar o ícone de reticências (`...`) na barra de ferramentas para exibir o botão **Parar**.

    > **Observação**: não espere até que os hosts da sessão sejam interrompidos e desalocados, mas prossiga para a próxima tarefa. Parar e desalocar hosts de sessão pode levar cerca de 2 minutos.

#### Tarefa 3: Ajustar as configurações do pool de host

> **Observação**: ao usar o dimensionamento automático para pools de hosts agrupados, você deve ter um parâmetro MaxSessionLimit configurado para esse pool de hosts. Neste laboratório, você o definirá artificialmente baixo para facilitar a ilustração da funcionalidade de dimensionamento automático.

1. No computador do laboratório, no navegador da Web que exibe o portal do Azure, na página **az140-21-hp1**, na seção **Configurações**, selecione **Propriedades**.
1. Na página **az140-21-hp1\|Propriedades**, na caixa de texto **Limite máximo de sessão**, insira **1**.
1. Na página **az140-21-hp1\|Propriedades**, selecione **Salvar**.

#### Tarefa 4: Crie um plano de dimensionamento

1. No computador do laboratório, no navegador da Web que exibe o portal do Azure, pesquise e selecione **Área de Trabalho Virtual do Azure**. Na página **Área de Trabalho Virtual do Azure**, na seção **Gerenciar** do menu de navegação vertical, selecione **Planos de dimensionamento**.
1. Na página **Área de Trabalho Virtual do Azure \| Planos de dimensionamento**, selecione **+ Criar**.
1. Na guia **Básico** da página **Criar um plano de dimensionamento**, especifique as seguintes configurações e selecione **Avançar: Agendamentos**:

    |Configuração|Valor|
    |---|---|
    |Assinatura|O nome da assinatura do Azure que você está usando neste laboratório|
    |Grupo de recursos|O nome de um novo grupo de recursos **az140-412e-RG**|
    |Nome do plano de dimensionamento|**az140-scalingplan412e**|
    |Region|O nome da região do Azure onde você implantou o ambiente de área de trabalho virtual do Azure|
    |Nome amigável|**az140-scalingplan412e**|
    |Fuso horário|O fuso horário local da região do Azure onde você implantou o ambiente de área de trabalho virtual do Azure|
    |Tipo de pool de host|**Em pool**|
    |Método de dimensionamento|**Escalonamento automático de gerenciamento de energia**|

    > **Observação**: deixe a propriedade **Rótulo de exclusão** sem definição. Em geral, você pode usar esse recurso para excluir VMs do Azure com rótulos definidos arbitrariamente do dimensionamento automático.

1. Na aba **Programação**, selecione **+ Adicionar programação**.

    > **Observação**: os cronogramas permitem que você defina horários de aceleração, horário de pico, horários de diminuição e horários de baixa demanda para dias úteis e especifique gatilhos de dimensionamento automático. O plano de escalonamento deve incluir uma programação associada para pelo menos um dia da semana. 

1. Na guia **Geral** do painel **Adicionar uma programação**, ajuste a configuração padrão para corresponder às seguintes configurações e selecione **Avançar**:

    |Configuração|Valor|
    |---|---|
    |Fuso horário|O fuso horário local do seu ambiente de Área de Trabalho Virtual do Azure (com base na região selecionada anteriormente nesta tarefa)|
    |Nome da agenda|**week_schedule**|
    |Repetir Em|**Sete selecionados** (selecione todos os dias da semana)|

    > **Observação**: o cronograma cobre efetivamente todos os dias da semana, o que facilitará a avaliação do resultado do dimensionamento automático.

1. Na guia **Acelerar** do painel **Adicionar uma agenda**, ajuste a configuração padrão para corresponder às seguintes configurações e selecione **Avançar**:

    |Configuração|Valor|
    |---|---|
    |Hora de início (sistema de 12 horas)|Sua hora atual menos 1 hora|
    |Algoritmo de balanceamento de carga|**Amplitude**|
    |Percentual mínimo de hosts (%)|**30**|
    |Limite de capacidade (%)|**60**|

    > **Observação**: para pools de hosts agrupados, o dimensionamento automático ignora os algoritmos de balanceamento de carga existentes nas configurações do pool de hosts e, em vez disso, aplica o balanceamento de carga com base na configuração da sua programação.

    > **Observação**: a configuração **Porcentagem mínima de hosts** designa a porcentagem mínima de máquinas virtuais de host de sessão a serem iniciadas para aceleração e horário de pico. Por exemplo, se a **Porcentagem mínima de hosts** for especificada como 30% e o número total de hosts de sessão no seu pool de hosts for 3, o dimensionamento automático garantirá que no mínimo 1 host de sessão esteja disponível para aceitar conexões de usuários.

    > **Observação**: o dimensionamento automático arredonda para o número inteiro mais próximo.

    > **Observação**: a configuração do **Limite de capacidade** é a porcentagem da capacidade do pool de hosts usada que será considerada para avaliar se as máquinas virtuais devem ser ligadas/desligadas durante os horários de pico e de aceleração. Por exemplo, se o limite de capacidade for especificado como 60% e a capacidade do seu pool de hosts for de 1 sessão (com um host em execução), o dimensionamento automático ativará hosts de sessão adicionais quando a carga do pool de hosts exceder 60% (neste caso, 100%).

1. Na guia **Horário de pico** do painel **Adicionar uma programação**, ajuste a configuração padrão para corresponder às seguintes configurações e selecione **Avançar**:

    |Configuração|Valor|
    |---|---|
    |Hora de início (sistema de 12 horas)|Sua hora atual mais 1 hora|
    |Algoritmo de balanceamento de carga|**Profundidade**|
    |Limite de capacidade (%)|**60**|

    > **Observação**: a configuração **Limite de capacidade (%)** é compartilhada entre as configurações **Aceleração** e **Horário de pico** .

1. Na guia **Desaceleração** do painel **Adicionar uma programação**, ajuste a configuração padrão para corresponder às seguintes configurações e selecione **Avançar**:

    |Configuração|Valor|
    |---|---|
    |Hora de início (sistema de 12 horas)|Sua hora atual mais 2 horas|
    |Algoritmo de balanceamento de carga|**Profundidade**|
    |Percentual mínimo de hosts ativos (%)|**10**|
    |Limite de capacidade (%)|**80**|
    |Forçar usuários a sair|**Não**|
    |Parar as VMs quando|**As VMs não têm sessões ativas ou desconectadas**|

    > **Observação**: a configuração **Porcentagem mínima de hosts ativos (%)** designa a porcentagem mínima de máquinas virtuais de host de sessão que você gostaria de acessar para deceleração e horários de menor pico. Por exemplo, se a **Porcentagem mínima de hosts ativos (%)** for definida como 10% e o número total de hosts de sessão no seu pool de hosts for 3, o dimensionamento automático garantirá que no mínimo 1 host de sessão esteja disponível para receber conexões de usuários.

    > **Observação**: a configuração **Limite de capacidade (%)** designa a porcentagem da capacidade do pool de hosts usada que será considerada para avaliar se as máquinas virtuais devem ser desligadas durante os horários de desaceleração e de menor movimento. Por exemplo, com 1 conexão de usuário e 3 hosts em execução, se o limite de capacidade for especificado como 80%, o dimensionamento automático desligará 1 host (resultando em 50% da capacidade do pool de hosts usada).

    > **Observação**: em geral, o dimensionamento automático interromperá e desalocará hosts de sessão de acordo com as seguintes regras:

    - A capacidade usada do pool de host está abaixo do limite de capacidade.
    - Desativar hosts de sessão não resultará em exceder o limite de capacidade.
    - O dimensionamento automático só desativa hosts de sessão sem sessões de usuário, a menos que o plano de dimensionamento esteja em fase de desaceleração e você tenha habilitado a configuração para forçar o logoff do usuário. 
    - O dimensionamento automático agrupado não desativará os hosts de sessão na fase de aceleração para garantir que a experiência do usuário não seja afetada.

1. Na guia **Horário de menor movimento** do painel **Adicionar uma programação**, ajuste a configuração padrão para corresponder às seguintes configurações e selecione **Adicionar**:

    |Configuração|Valor|
    |---|---|
    |Hora de início (sistema de 12 horas)|Sua hora atual mais 3 horas|
    |Algoritmo de balanceamento de carga|**Profundidade**|
    |Limite de capacidade (%)|**80**|

    > **Observação**: a configuração de **Limite de capacidade** é compartilhada entre as configurações de **Desaceleração** e **Horário de baixa demanda**.

1. De volta à guia **Agendamento** da página **Criar um plano de dimensionamento**, selecione **Avançar: Atribuições de pool de hosts**.
1. Na guia **Atribuições de pool de hosts**, na lista suspensa **Selecionar pool de hosts**, selecione **az140-21-hp1**, certifique-se de que a caixa de seleção **Ativar dimensionamento automático** esteja marcada e, em seguida, selecione **Revisar + criar**.
1. Na página **Examinar + Criar**, selecione **Criar**.

    > **Observação**: aguarde a conclusão da configuração de dimensionamento automático. Isso normalmente leva apenas alguns segundos.

#### Tarefa 5: Avaliar a funcionalidade de dimensionamento automático

> **Observação**: você começará avaliando as configurações de **Aceleração**.

1. No computador do laboratório, no navegador da Web que exibe o portal do Azure, pesquise e selecione **Área de Trabalho Virtual do Azure** e, na página **Área de Trabalho Virtual do Azure**, na seção **Gerenciar** da barra de menu vertical, selecione **Pools de hosts**.
1. Na página **Área de Trabalho Virtual do Azure \| Pools de hosts**, na lista de pools de hosts, selecione **az140-21-hp1**.
1. Na página **az140-21-hp1**, na barra de menu vertical, na seção **Gerenciar**, selecione **Hosts de sessão**.
1. Na página **az140-21-hp1 \| Hosts de sessão**, revise os valores da configuração **Estado de energia** dos hosts de sessão e verifique se um deles está listado como **Em execução**.

    > **Observação**: pode ser necessário aguardar alguns minutos antes que o primeiro host da sessão atinja o estado **Em execução**.

    > **Observação**: isso é esperado, pois, de acordo com as configurações de **Aceleração** do plano de dimensionamento recém-criado, pelo menos um host de sessão deve estar sempre online. Neste ponto, a capacidade do pool de hosts é 1 (já que há apenas 1 host em execução), mas a capacidade do pool de hosts usados é 0%, já que não há conexões de usuários.

    > **Observação**: em seguida, você avaliará a configuração de limite de capacidade de **Aceleração** e **Horas de pico** iniciando uma única sessão de usuário. Podemos avaliar isso mesmo fora da janela de **Horário de pico**, já que os dois estágios compartilham o mesmo limite de capacidade.

1. No computador do laboratório, inicie o cliente da Área de Trabalho Remota da Microsoft.
1. No computador do laboratório, na janela do cliente **Área de Trabalho Remota**, selecione **Inscrever-se** e, quando solicitado, entre com as credenciais da conta de usuário Entra ID `User2`, que você pode localizar na guia **Recursos** no painel direito da janela da interface do laboratório.
1. Certifique-se de que a página **Área de Trabalho Remota** exiba quatro ícones, incluindo Microsoft Word, Microsoft Excel, Microsoft PowerPoint e Prompt de Comando. 
1. Clique duas vezes no ícone do prompt de comando. 
1. Quando solicitado a entrar, na caixa de diálogo **Segurança do Windows**, insira a senha da conta de usuário do Microsoft Entra que você usou para se conectar ao ambiente de área de trabalho virtual do Azure de destino.
1. Verifique se uma janela de **Prompt de Comando** aparece logo depois. 

    > **Observação**: neste ponto, a capacidade do pool de hosts usado é 100%, o que é maior que o limite de capacidade (60%). Isso deve fazer com que o dimensionamento automático seja ativado em outro host, o que levará a capacidade do pool de hosts usados para 50%. Como isso está abaixo do limite de capacidade, o terceiro host permanecerá parado/desalocado. Você verificará isso a seguir.

1. No computador do laboratório, alterne para o navegador da Web que exibe o portal do Azure. 
1. Na página **az140-21-hp1 \| Hosts de sessão**, selecione **Atualizar**, revise os valores da configuração **Estado de energia** dos hosts de sessão e verifique se agora dois deles estão listados como **Em execução**.

    > **Observação**: em seguida, você avaliará a configuração do limite de capacidade de **Desaceleração** ajustando sua janela de tempo. 

1. Na página **az140-21-hp1 \| Hosts de sessão**, no menu de navegação vertical, na seção **Gerenciar**, selecione **Planos de dimensionamento** e, em seguida, na página **Planos de dimensionamento**, selecione **az140-scalingplan412e**.
1. Na página **az140-scalingplan412e**, no menu de navegação vertical, na seção **Configurações**, selecione **Programações** e depois selecione **week_schedule**.
1. No painel **week_schedule**, navegue até a guia **Desaceleração** e ajuste o valor da configuração **Hora de início (sistema de 12 horas)** para qualquer hora entre **Hora de início (sistema de 12 horas)** da fase **Horário de pico** e sua hora atual.

    > **Observação**: talvez seja necessário ajustar o valor de **Hora de início (sistema de 12 horas)** da fase **Horas de pico**.

1. No painel **week_schedule**, navegue até a guia **Horário de menor movimento** e selecione **Salvar**.
1. Alterne para a janela **Prompt de Comando** que representa a única sessão RDP para o pool de hosts e, no Prompt de Comando, digite o seguinte e pressione a tecla **Enter**:

    ```cmd
    logoff
    ```

1. No computador do laboratório, no navegador da Web que exibe o portal do Azure, pesquise e selecione **Área de Trabalho Virtual do Azure** e, na página **Área de Trabalho Virtual do Azure**, na seção **Gerenciar** da barra de menu vertical, selecione **Pools de hosts**.
1. Na página **Área de Trabalho Virtual do Azure \| Pools de hosts**, na lista de pools de hosts, selecione **az140-21-hp1**.
1. Na página **az140-21-hp1**, na barra de menu vertical, na seção **Gerenciar**, selecione **Hosts de sessão**.
1. Na página **az140-21-hp1 \| Hosts de sessão**, revise os valores da configuração **Estado de energia** dos hosts de sessão e verifique se agora apenas um deles está listado como **Em execução**.

    > **Observação**: pode levar de 1 a 2 minutos para que o host da sessão seja desligado.

#### Tarefa 6: Desabilitar o dimensionamento automático do pool de hosts

> **Observação**: para garantir que a configuração de dimensionamento automático não afete outros laboratórios, você removerá a atribuição de pool de hosts do plano de dimensionamento implementado neste laboratório.

1. No computador do laboratório, no navegador da Web que exibe o portal do Azure, pesquise e selecione **Área de Trabalho Virtual do Azure**. Na página **Área de Trabalho Virtual do Azure**, na seção **Gerenciar** do menu de navegação vertical, selecione **Planos de dimensionamento** e, em seguida, na página **Planos de dimensionamento**, selecione **az140-scalingplan412e**.
1. Na página **az140-scalingplan412e**, na seção **Gerenciar**, selecione **Atribuições de pool de hosts**.
1. Na página **az140-scalingplan412e \| Atribuições de pool de hosts**, selecione **az140-21-hp1**, depois selecione **Cancelar atribuição** e, quando solicitado a confirmar, na caixa de diálogo **Cancelar atribuição de pool de hosts**, selecione **Cancelar atribuição**.
