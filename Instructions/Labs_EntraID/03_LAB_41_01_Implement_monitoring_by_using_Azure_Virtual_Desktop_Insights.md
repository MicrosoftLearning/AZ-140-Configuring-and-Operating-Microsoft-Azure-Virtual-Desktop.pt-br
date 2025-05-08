---
lab:
  title: 'Laboratório: Implementar monitoramento usando o Área de Trabalho Virtual do Azure Insights'
  module: 'Module 4.1: Monitor and manage Azure Virtual Desktop services'
---

# Laboratório — Implementar monitoramento usando o Área de Trabalho Virtual do Azure Insights
# Manual de laboratório do aluno

## Dependências do laboratório

- O nome da assinatura do Azure que você usará nesse laboratório
- Uma conta de usuário do Microsoft Entra com a função Proprietário ou Colaborador na assinatura do Azure que você usará neste laboratório e com permissões suficientes para unir dispositivos ao locatário do Entra associado a essa assinatura do Azure.
- Ter concluído o laboratório *Implantar pools de hosts e hosts de sessão usando o portal do Azure (AD DS)*
- O laboratório *Gerenciar pools de hosts e hosts de sessão usando o portal do Azure (ID Entra)* foi concluído

## Tempo estimado

25 minutos

## Cenário do laboratório

Você tem um ambiente de Área de Trabalho Virtual do Azure existente. Você deseja monitorar o status e as atividades do ambiente.

## Objetivos
  
Depois de realizar este laboratório, você será capaz de:

- Implementar o monitoramento de um ambiente de área de trabalho virtual do Azure

## Arquivos do laboratório

- Nenhum

## Instruções

### Exercício 1: Implementar monitoramento de um ambiente de área de trabalho virtual do Azure
  
As principais tarefas desse exercício são as seguintes:

1. Registre a assinatura do Azure com o provedor de recursos Microsoft.Insights
1. Criar um workspace do Azure Log Analytics
1. Configurar a pasta de trabalho de configuração de Insights da Área de Trabalho Virtual

#### Tarefa 1: Registre a assinatura do Azure com o provedor de recursos Microsoft.Insights

> **Observação**: os Insights da Área de Trabalho Virtual do Azure depende do provedor de recursos Microsoft.Insights, portanto, você precisa primeiro registrá-lo na assinatura do Azure que está usando para este laboratório. No laboratório *Implantar pools de hosts e hosts de sessão usando o portal do Azure (ID Entra)*, você executou esta tarefa usando o Azure PowerShell. Neste laboratório, você fará isso usando o portal do Azure (qualquer um dos métodos tem suporte e está disponível).

1. Se necessário, no computador do laboratório, inicie um navegador da Web, navegue até o portal do Azure e entre fornecendo credenciais de uma conta de usuário com a função Proprietário na assinatura que você usará neste laboratório.

    > **Observação**: use as credenciais da conta `User1-` listada na guia Recursos no lado direito da janela da sessão de laboratório.

1. No computador do laboratório, no navegador da Web que exibe o portal do Azure, pesquise e selecione **Assinaturas**. Na página **Assinaturas**, selecione a assinatura do Azure que você está usando neste laboratório e, no menu de navegação vertical, na seção **Configurações**, selecione **Provedores de recursos**.
1. Na guia **Provedores de recursos**, na caixa de texto de pesquisa, digite **Microsoft.Insights**, na lista de resultados, selecione o pequeno círculo à esquerda da entrada **Microsoft.Insights** e selecione **Registrar**.

    > **Observação**: aguarde a conclusão do processo de registro. Isso normalmente leva cerca de 1 minuto. Use o botão da barra de ferramentas **Atualizar** para exibir o valor atualizado do status do registro.

#### Tarefa 2: criar um workspace do Azure Log Analytics

> **Observação**: os Insights da Área de Trabalho Virtual do Azure é um painel criado nas pastas de trabalho do Azure Monitor que facilita o monitoramento de ambientes da Área de Trabalho Virtual do Azure. 

> **Observação**: você só pode monitorar operações de dimensionamento automático com o Insights com pools de hosts agrupados. 

1. No computador do laboratório, no navegador da Web que exibe o portal do Azure, pesquise e selecione **Workspaces do Log Analytics**e, na página **Workspaces do Log Analytics**, selecione **+ Criar**.
1. Na guia **Básico** da página **Criar workspace do Log Analytics**, especifique as seguintes configurações e selecione **Revisar + criar**:

    |Configuração|Valor|
    |---|---|
    |Assinatura|O nome da assinatura do Azure que você está usando neste laboratório|
    |Grupo de recursos|O nome de um novo grupo de recursos **az140-411e-RG**|
    |Nome|**az140-laworkspace41e**|
    |Region|O nome da região do Azure onde você implantou o ambiente de área de trabalho virtual do Azure|

1. Na página **Examinar + Criar**, selecione **Criar**.

    > **Observação**: Aguarde o processo de provisionamento ser concluído. Isso normalmente leva cerca de 1 minuto.

    > **Observação**: em seguida, você precisa habilitar a coleta de dados no workspace do Log Analytics recém-provisionado de diagnósticos do ambiente de Área de Trabalho Virtual do Azure, contadores de desempenho dos hosts de sessão e Logs de Eventos do Windows dos hosts de sessão da Área de Trabalho Virtual do Azure.

#### Tarefa 3: Configurar a pasta de trabalho de configuração dos Insights da Área de Trabalho Virtual

> **Observação**: ao abrir os Insights da Área de Trabalho Virtual do Azure pela primeira vez, você precisa configurar os Insights da Área de Trabalho Virtual do Azure para direcionar seu ambiente da Área de Trabalho Virtual do Azure.

1. No computador do laboratório, no navegador da Web que exibe o portal do Azure, pesquise e selecione **Área de Trabalho Virtual do Azure** e, na página **Área de Trabalho Virtual do Azure**, no menu de navegação vertical, na seção **Monitoramento**, selecione **Pastas de trabalho**.
1. Na lista de pastas de trabalho da **Área de Trabalho Virtual do Windows**, na seção **Área de Trabalho Virtual do Windows**, selecione a pasta de trabalho **Insights**.
1. Na página **Área de Trabalho Virtual do Azure \| Pastas de Trabalho \| Insights**, revise as mensagens de aviso indicando que o workspace e os hosts de sessão não estão enviando dados para o workspace e selecione o link **Configuração de pasta de trabalho** para reparar o problema.
1. Na página **CheckAMAConfiguration**, na guia **Configurações de diagnóstico de recursos**, na lista suspensa **Workspace do Log Analytics**, selecione **az140-laworkspace41e**.
1. Na página **CheckAMAConfiguration**, na guia **Configurações de diagnóstico de recursos**, na seção **Pool de host az140-21-hp1**, observe a mensagem de aviso indicando que nenhuma configuração de diagnóstico existente foi encontrada para o pool de host selecionado e escolha **Configurar pool de host**.
1. No painel **Implantar modelo**, selecione **Implantar**.

    > **Observação**: isso habilita efetivamente as seguintes tabelas de diagnóstico no workspace do Log Analytics de destino:
    - Atividades de gerenciamento
    - Feed
    - conexões
    - Errors
    - Pontos de verificação
    - HostRegistration
    - AgentHealthStatus

    > **Observação**: aguarde até que a implantação seja concluída. Essa etapa geralmente leva menos de um minuto.

1. Na página **CheckAMAConfiguration**, na guia **Configurações de diagnóstico de recursos**, selecione o ícone **Atualizar** (uma seta circular) na barra de ferramentas.
1. Revise a seção **Host pool az140-21-hp1** e verifique se as configurações de diagnóstico estão habilitadas para **allLogs**.
1. Na guia **Configurações de diagnóstico de recursos**, role para baixo até a seção **Workspace az140-21-ws1** e selecione **Configurar workspace**.
1. No painel **Implantar modelo**, selecione **Implantar**.

    > **Observação**: isso configura efetivamente o workspace para **allLogs**.

    > **Observação**: aguarde até que a implantação seja concluída. Essa etapa geralmente leva menos de um minuto.

1. Na página **CheckAMAConfiguration**, na guia **Configurações de diagnóstico de recursos**, selecione o ícone **Atualizar** (uma seta circular) na barra de ferramentas.
1. Revise a seção **Workspace az140-21-ws1** e verifique se as configurações de diagnóstico estão habilitadas para **allLogs** e se não há mensagens de aviso restantes.
1. Navegue até o topo da página **CheckAMAConfiguration** e mude para a aba **Selecionar configurações de dados do host**.
1. Na guia **Selecionar configurações de dados do host**, na seção **Criar DCR**, na lista suspensa **Destino do workspace**, selecione **az140-laworkspace41e** e, em seguida, selecione **Criar regra de coleta de dados**.
1. No painel **Implantar modelo**, selecione **Implantar**.

    > **Observação**: aguarde até que a implantação seja concluída. Essa etapa geralmente leva menos de um minuto.

1. Na página **CheckAMAConfiguration**, na guia **Configurações dos dados do host da sessão**, clique no ícone **Atualizar** (uma seta circular) na barra de ferramentas.

    > **Observação**: antes de prosseguir, certifique-se de que o DCR recém-criado esteja listado na subseção **DCRs disponíveis** da seção **Criar DCR**. Se esse não for o caso, aguarde mais um minuto e atualize a página novamente.

1. Na guia **Configurações dos dados do host da sessão**, na lista suspensa **DCR selecionado**, selecione a entrada que começa com o prefixo **microsoft-avdi-**.
1. Se necessário, na guia **Configurações dos dados do host de sessão**, na seção **Associações DCR**, selecione **Implantar associação** e, no painel **Implantar modelo**, selecione **Implantar**.

    > **Observação**: isso associa efetivamente o DCR recém-criado aos hosts de sessão no pool de hosts **az140-21-hp1**.

    > **Observação**: aguarde até que a implantação seja concluída. Essa etapa geralmente leva menos de um minuto.

1. Na página **CheckAMAConfiguration**, na guia **Configurações dos dados do host da sessão**, clique no ícone **Atualizar** (uma seta circular) na barra de ferramentas.
1. Na guia **Configurações dos dados do host da sessão**, na seção **Hosts de sessão sem extensão do Azure Monitor**, selecione **Adicionar extensão**.
1. No painel **Implantar modelo**, selecione **Implantar**.

    > **Observação**: isso efetivamente instala a extensão do Azure Monitor nos hosts de sessão no pool de hosts **az140-21-hp1**.

    > **Observação**: aguarde até que a implantação seja concluída. Isso pode levar cerca de um minuto.

1. Na página **CheckAMAConfiguration**, na guia **Configurações dos dados do host da sessão**, clique no ícone **Atualizar** (uma seta circular) na barra de ferramentas.
1. Verifique se não há mensagens de erro ou aviso exibidas. 
1. Navegue até o topo da página **CheckAMAConfiguration**, selecione a guia **Dados gerados** e, em seguida, selecione o ícone **Atualizar** (uma seta circular) na barra de ferramentas.
1. Revise as seções que exibem gráficos que representam dados coletados, incluindo **Dados faturados nas últimas 24 horas**, **Contadores de desempenho** e **Eventos**.

    > **Observação**: use a seção **Dados cobrados nas últimas 24 horas** para monitorar a ingestão de dados. Você é responsável pelas cobranças do Log Analytics para armazenamento e ingestão de dados.

1. No navegador da Web que exibe o portal do Azure, navegue de volta para a página **Área de Trabalho Virtual do Azure** e, na seção **Monitoramento** do menu de navegação vertical, selecione **Insights**.
1. Na página **Área de Trabalho Virtual do Azure \| Insights**, revise o conteúdo da guia **Visão geral**, incluindo a seção **Capacidade**, **Diagnóstico de conexão: % de usuários capazes de se conectar**, **Desempenho da conexão: Tempo para conectar (novas sessões)** e telemetria **Utilização**. 
1. Em seguida, revise todas as guias restantes na página  **Área de Trabalho Virtual do Azure \| Insights**, incluindo **Confiabilidade da conexão**, **Diagnóstico da conexão**, **Desempenho da conexão**, **Usuários**, **Utilização**, **Clientes** e **Alertas**.

    > **Observação**: considere revisitar essas guias da página Insights depois de concluir os laboratórios subsequentes para revisar os gráficos que representam a telemetria coletada.
