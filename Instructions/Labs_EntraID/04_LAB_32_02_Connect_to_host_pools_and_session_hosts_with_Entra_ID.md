---
lab:
  title: 'Laboratório: Conectar-se aos hosts de sessão (Entra ID)'
  module: 'Module 3.2: Plan and implement user experience and client settings'
---

# Laboratório — Conectar-se aos hosts da sessão (Entra ID)
# Manual de laboratório do aluno

## Dependências do laboratório

- O nome da assinatura do Azure que você usará nesse laboratório
- Uma conta de usuário do Microsoft Entra com a função Proprietário ou Colaborador na assinatura do Azure que você usará neste laboratório e com permissões suficientes para unir dispositivos ao locatário do Entra associado a essa assinatura do Azure.
- Ter concluído o laboratório *Implantar pools de hosts e hosts de sessão usando o portal do Azure (AD DS)*
- O laboratório *Gerenciar pools de hosts e hosts de sessão usando o portal do Azure (ID Entra)* foi concluído

## Tempo estimado

20 minutos

## Cenário do laboratório

Você tem um ambiente de Área de Trabalho Virtual do Azure existente que contém hosts de sessão ingressados no Entra. Você precisa validar a funcionalidade deles conectando-se a eles a partir de um cliente Windows 11 que não esteja ingressado ou registrado no Microsoft Enterprise.

## Objetivos
  
Depois de realizar este laboratório, você será capaz de:

- Valide a funcionalidade dos hosts de sessão da Área de Trabalho Virtual do Azure associados ao Microsoft Entra conectando-se a eles a partir de um cliente Windows que não esteja associado ao Microsoft Entra ou registrado.

## Arquivos do laboratório

- Nenhum

## Instruções

### Exercício 1: Validar a funcionalidade dos hosts de sessão da Área de Trabalho Virtual do Azure associados ao Microsoft Entra conectando-se a eles de um cliente Windows 11
  
As principais tarefas desse exercício são as seguintes:

1. Ajustar as propriedades do RDP do pool de hosts da Área de Trabalho Virtual do Azure
1. Instalar o cliente Microsoft Remote Desktop em um computador Windows 11
1. Assinar um workspace da Área de Trabalho Virtual do Azure
1. Testar aplicativos da Área de Trabalho Virtual do Azure


#### Tarefa 1: Ajustar as propriedades do RDP do pool de hosts da Área de Trabalho Virtual do Azure

> **Observação**: as configurações de RDP implementadas no laboratório anterior fornecem a experiência ideal do usuário (por meio do suporte para logon único); no entanto, isso requer alterações adicionais descritas em [Configurar logon único para o Área de Trabalho Virtual do Azure usando a autenticação do Microsoft Entra ID](https://learn.microsoft.com/en-us/azure/virtual-desktop/configure-single-sign-on). Sem essas alterações, por padrão, a autenticação é suportada, desde que o computador cliente atenda a um dos seguintes critérios:

- É o Microsoft Entra associado ao mesmo locatário do Microsoft Entra que o host da sessão
- É um híbrido do Microsoft Entra associado ao mesmo locatário do Microsoft Entra que o host da sessão
- É o Microsoft Entra registrado no mesmo locatário do Microsoft Entra que o host da sessão

Como nenhum desses critérios se aplica ao computador de laboratório, é necessário adicionar `targetisaadjoined:i:1` como uma propriedade do RDP personalizada ao pool de hosts.

1. Se necessário, no computador do laboratório, inicie um navegador da Web, navegue até o portal do Azure e entre fornecendo credenciais de uma conta de usuário com a função Proprietário na assinatura que você usará neste laboratório.

    > **Observação**: use as credenciais da conta `User1-` listada na guia Recursos no lado direito da janela da sessão de laboratório.

1. No navegador da Web que exibe o portal do Azure, na página do pool de host da Área de Trabalho Virtual do Azure **az140-21-hp1**, na barra de menu vertical, na seção **Configurações**, selecione a entrada **Propriedades do RDP**.
1. Na página **az140-21-hp1 \| Propriedades do RDP**, selecione a guia **Avançado**. 
1. Na guia **Avançado** da página **az140-21-hp1 \| Propriedades do RDP**, na caixa de texto **Propriedades do RDP**, anexe a seguinte cadeia de caracteres ao conteúdo existente (certifique-se de adicionar um ponto e vírgula inicial (`;`) se necessário para separar esta cadeia de caracteres daquela que a precede):

    ```txt
    targetisaadjoined:i:1
    ```

1. Na caixa de texto **Propriedades do RDP**, remova a seguinte sequência de caracteres (se presente) do conteúdo existente (com seu caractere de ponto e vírgula final):

    ```txt
    enablerdsaadauth:i:value
    ```

1. Na página **az140-21-hp1 \| Propriedades do RDP**, selecione **Salvar**.

#### Tarefa 2: instalar o cliente Microsoft Remote Desktop em um computador Windows 11

1. No computador do laboratório, inicie um navegador da Web, navegue até a página [Conectar-se à Área de Trabalho Virtual do Azure com o cliente de Área de Trabalho Remota para Windows](https://learn.microsoft.com/en-us/azure/virtual-desktop/users/connect-windows), role para baixo até a seção **Baixar e instalar o cliente de Área de Trabalho Remota (MSI)** e selecione o link [Windows de 64 bits](https://go.microsoft.com/fwlink/?linkid=2139369). 
1. Abra o Explorador de Arquivos, navegue até a pasta **Downloads** e inicie a instalação do arquivo MSI recém-baixado. 
1. Na janela **Configuração da Área de Trabalho Remota**, quando solicitado, aceite os termos do contrato de licenciamento e escolha a opção **Instalar para todos os usuários desta máquina**. Se solicitado, aceite o prompt do Controle de Conta de Usuário para prosseguir com a instalação.
1. Após a conclusão da instalação, certifique-se de que a caixa de seleção **Iniciar a Área de Trabalho Remota quando a instalação sair** esteja marcada e selecione **Concluir** para iniciar o cliente da Área de Trabalho Remota da Microsoft.

   > **Observação**: o [aplicativo da Loja da Área de Trabalho Remota](https://learn.microsoft.com/en-us/azure/virtual-desktop/users/connect-windows?pivots=rd-store) para Windows não oferece suporte à conexão com hosts de sessão ingressados no Microsoft Entra.

#### Tarefa 3: Assinar um workspace da Área de Trabalho Virtual do Azure

1. No computador do laboratório, alterne para a janela do cliente **Área de Trabalho Remota**, selecione **Assinar** e, quando solicitado, entre com as credenciais da conta de usuário do Entra ID `User1`, que você pode localizar na guia **Recursos** no painel direito da janela da interface do laboratório.

   > **Observação**: selecione a conta de usuário que é membro do grupo Entra com o prefixo **AVD-DAG**.

   > **Observação**: como alternativa, na janela do cliente **Área de Trabalho Remota**, selecione **Assinar com URL**, no painel **Assinar um Workspace**, no **URL de email ou Workspace**, digite **https://client.wvd.microsoft.com/api/arm/feeddiscovery**, selecione **Avançar** e, quando solicitado, entre com as credenciais do Microsoft Entra.

1. Certifique-se de que a página **Área de Trabalho Remota** exiba apenas o ícone **SessionDesktop**.

   > **Observação**: isso é esperado, porque a conta de usuário do Microsoft Entra selecionada foi atribuída no primeiro laboratório *Implantar pools de hosts e hosts de sessão usando o portal do Azure (ID Entra)* ao grupo de aplicativos de desktop **az140-21-hp1-DAG** gerado automaticamente.

1. Na página **Área de Trabalho Remota**, clique com o botão direito do mouse no ícone **SessionDesktop** e, no menu pop-up, selecione **Configurações**.
1. No painel **SessionDesktop**, desative a opção **Usar configurações padrão**.
1. Na seção **Configurações de exibição**, no menu suspenso, selecione a entrada **Selecionar exibições** e escolha as exibições que deseja usar para a sessão.
1. No painel **SessionDesktop**, revise as opções restantes, incluindo **Maximizar para exibições atuais**, **Exibição única no modo de janela** e **Ajustar sessão à janela**, sem fazer nenhuma alteração. 
1. Feche o painel **SessionDesktop**. 
1. Na página **Área de Trabalho Remota**, clique duas vezes no ícone **SessionDesktop**.
1. Quando solicitado a entrar, na caixa de diálogo **Segurança do Windows**, insira a senha da primeira conta de usuário do Microsoft Entra, que você usou nesta tarefa para se conectar ao ambiente de destino da Área de Trabalho Virtual do Azure.

   > **Observação**: a Área de Trabalho Virtual do Azure não oferece suporte para entrar no Microsoft Entra ID com uma conta de usuário e depois entrar no Windows com uma conta de usuário separada. Entrar com duas contas diferentes ao mesmo tempo pode levar os usuários a se reconectarem ao host de sessão incorreto, informações incorretas ou ausentes no portal do Azure e mensagens de erro que aparecem ao usar a anexação de aplicativo ou anexação de aplicativo MSIX.

   > **Observação**: a janela **SessionDesktop** será exibida automaticamente.

1. Na janela de sessão da Área de Trabalho Remota, verifique se você tem acesso administrativo total na sessão (por exemplo, selecione o ícone do logotipo do **Windows** na barra de tarefas e, em seguida, selecione o item **Windows PowerShell(Admin)** no menu pop-up.
1. Na janela da sessão da Área de Trabalho Remota, selecione o ícone do logotipo do Windows na barra de tarefas, selecione o ícone de avatar que representa a conta de usuário do Microsoft Entra que você usou para entrar e, no menu pop-up, selecione **Sair**.

   > **Observação**: isso encerrará automaticamente a sessão da Área de Trabalho Remota. 

1. De volta à janela **Área de Trabalho Remota**, selecione o ícone de reticências (`...`) à direita da entrada do workspace **az140-21-ws1**, selecione **Cancelar inscrição** e, quando solicitado a confirmar, selecione **Continuar**.
1. Na janela do cliente **Área de Trabalho Remota**, selecione **Assinar** e, quando solicitado, entre com as credenciais da segunda conta de usuário do Entra ID, que você pode localizar na guia **Recursos** no painel direito da janela da interface do laboratório.

   > **Observação**: Selecione a conta de usuário que é membro do grupo Entra com o prefixo **AVD-RemoteApp**.

1. Certifique-se de que a página **Área de Trabalho Remota** exiba quatro ícones, incluindo Prompt de Comando, Microsoft Word, Microsoft Excel, Microsoft PowerPoint. 

   > **Observação**: isso é esperado, porque a conta de usuário do Microsoft Entra selecionada foi atribuída no primeiro laboratório *Implantar pools de hosts e hosts de sessão usando o portal do Azure (ID Entra)* aos grupos de aplicativos **az140-21-hp1-Office365-RAG** e **az140-21-hp1-Utilities-RAG** .

1. Clique duas vezes no ícone do prompt de comando. 
1. Quando solicitado a entrar, na caixa de diálogo **Segurança do Windows**, insira a senha da segunda conta de usuário do Microsoft Entra que você usou para se conectar ao ambiente de destino da Área de Trabalho Virtual do Azure.
1. Verifique se uma janela de **Prompt de Comando** aparece logo depois. 
1. Na janela do Prompt de Comando, digite **hostname** e pressione a tecla **Enter** para exibir o nome do computador no qual o Prompt de Comando está sendo executado.

   > **Observação**: verifique se o nome exibido começa com o prefixo **sh-**.

1. No Prompt de Comando, digite **logoff** e pressione a tecla **Enter** para sair da sessão de Aplicativo Remoto atual.
1. Clique duas vezes nos ícones restantes na página **Área de Trabalho Remota** para iniciar o Microsoft Word, o Microsoft Excel e o Microsoft PowerPoint.
1. Feche cada janela de sessão.
