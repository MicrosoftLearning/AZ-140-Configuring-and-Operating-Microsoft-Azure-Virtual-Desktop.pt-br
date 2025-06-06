---
lab:
  title: 'Laboratório: Implantar pools de hosts e hosts usando modelos do Azure Resource Manager (AD DS)'
  module: 'Module 2: Implement an AVD Infrastructure'
---

# Laboratório: implantar pools de hosts e hosts usando modelos do Azure Resource Manager
# Manual de laboratório do aluno

## Dependências do laboratório

- O nome da assinatura do Azure que você usará nesse laboratório
- Uma conta Microsoft ou uma conta do Microsoft Entra com a função de Proprietário ou Colaborador na assinatura do Azure que você usará neste laboratório e com a função de Administrador Global no locatário do Microsoft Entra associado a essa assinatura do Azure.
- O laboratório **Preparar para a implantação da Área de Trabalho Virtual do Azure (AD DS)** concluído
- O laboratório **Implantar pools de hosts e hosts de sessão usando o portal do Azure (AD DS)** concluído

## Tempo estimado

45 minutos

## Cenário do laboratório

Você precisa automatizar a implantação de pools de host e hosts da Área de Trabalho Virtual do Azure usando modelos do Azure Resource Manager.

## Objetivos
  
Depois de realizar este laboratório, você será capaz de:

- Implantar pools e hosts de host da Área de Trabalho Virtual do Azure usando modelos do Azure Resource Manager

## Arquivos do laboratório

-  \\\\AZ-140\\AllFiles\\Labs\\02\\az140-23_azuredeployhp23.parameters.json
-  \\\\AZ-140\\AllFiles\\Labs\\02\\az140-23_azuremodifyhp23.parameters.json

## Instruções

### Exercício 1: Implantar pools e hosts de host da Área de Trabalho Virtual do Azure usando modelos do Azure Resource Manager
  
As principais tarefas desse exercício são as seguintes:

1. Preparar-se para a implantação de um pool de hosts da Área de Trabalho Virtual do Azure usando um modelo do Azure Resource Manager
1. Implantar um pool de hosts e hosts da Área de Trabalho Virtual do Azure usando um modelo do Azure Resource Manager
1. Verificar a implantação do pool de hosts e hosts da Área de Trabalho Virtual do Azure
1. Prepare-se para adicionar hosts ao pool de host da Área de Trabalho Virtual do Azure existente usando um modelo do Azure Resource Manager
1. Adicionar hosts ao pool de host da Área de Trabalho Virtual do Azure existente usando um modelo do Azure Resource Manager
1. Verificar alterações no pool de host da Área de Trabalho Virtual do Azure
1. Gerenciar atribuições de área de trabalho pessoal no pool de host da Área de Trabalho Virtual do Azure

#### Tarefa 1: Preparar-se para a implantação de um pool de hosts da Área de Trabalho Virtual do Azure usando um modelo do Azure Resource Manager

1. No seu computador de laboratório, inicie um navegador da Web, navegue até o [portal do Azure](https://portal.azure.com) e entre fornecendo credenciais de uma conta de usuário com a função Proprietário na assinatura que você usará nesse laboratório.
1. No portal do Azure, pesquise e selecione **Máquinas Virtuais** e, na folha **Máquinas Virtuais**, selecione **az140-dc-vm11**.
1. Na folha **az140-dc-vm11**, selecione **Conectar**, no menu suspenso, selecione **Conectar via Bastion**.
1. Quando solicitado, forneça as seguintes credenciais e selecione **Conectar**:

   |Configuração|Valor|
   |---|---|
   |Nome do usuário|**Aluno**|
   |Senha|**Pa55w.rd1234**|

1. Na sessão do Bastion para **az140-dc-vm11**, inicie o **ISE do Windows PowerShell** como administrador.
1. Na sessão do Bastion para **az140-cl-vm11**, no ISE do Windows Power Shell** Console ISE do Windows PowerShell**: execute o seguinte para identificar o nome diferenciado da unidade organizacional chamada **WVDInfra** que hospedará os objetos de computador dos hosts do pool da Área de Trabalho Virtual do Azure:

   ```powershell
   (Get-ADOrganizationalUnit -Filter "Name -eq 'WVDInfra'").distinguishedName
   ```

1. Na sessão do Bastion para **az140-cl-vm11**, no ISE do Windows Power Shell** Painel de script ISE do Windows PowerShell**: execute o seguinte para identificar o atributo de nome de entidade de usuário da conta de **Aluno\\do ADATUM** que você usará para ingressar os hosts da Área de Trabalho Virtual do Azure no domínio do AD DS (**student@adatum.com**):

   ```powershell
   (Get-ADUser -Filter "sAMAccountName -eq 'student'").userPrincipalName
   ```

1. Na sessão do Bastion para **az140-cl-vm11**, no ISE do Windows Power Shell** Painel de script do ISE do Windows PowerShell**: execute o seguinte para identificar o nome principal do usuário das **ADATUM\\aduser7** e **ADATUM\\aduser8** que você irá usar para testar as atribuições pessoais da área de trabalho posteriormente neste laboratório:

   ```powershell
   (Get-ADUser -Filter "sAMAccountName -eq 'aduser7'").userPrincipalName
   (Get-ADUser -Filter "sAMAccountName -eq 'aduser8'").userPrincipalName
   ```

   > **Observação**: Registre todos os valores de nome de entidade de usuário que você identificou **e** o nome diferenciado para a UO WVDInfra. Você precisará deles adiante neste laboratório.

1. Na sessão do Bastion para **az140-cl-vm11**, no ISE do Windows Power Shell** Painel de script do ISE do Windows PowerShell**: execute o seguinte para calcular o tempo de expiração do token necessário para executar uma implantação baseada em modelo:

   ```powershell
   $((get-date).ToUniversalTime().AddDays(1).ToString('yyyy-MM-ddTHH:mm:ss.fffffffZ'))
   ```

   > **Observação**: o valor deve ser semelhante ao formato `2022-03-27T00:51:28.3008055Z`. Registre-o, pois você precisará dele na próxima tarefa.

   > **Observação**: Um token de registro é necessário para autorizar um host a ingressar no pool. O valor da data de validade do token deve estar entre uma hora e um mês a partir da data e hora atuais.

1. Na sessão do Bastion para **az140-dc-vm11**, inicie o Microsoft Edge e navegue até o [portal do Azure](https://portal.azure.com). Se solicitado, entre usando as credenciais do Microsoft Entra da conta de usuário com a função Proprietário na assinatura que você está usando nesse laboratório.
1. Na sessão do Bastion para **az140-dc-vm11**, no portal do Azure, use a caixa de texto **Pesquisar recursos, serviços e documentos** na parte superior da página do portal do Azure para pesquisar e navegar até **Redes virtuais** e, na folha **Redes virtuais**, selecione **az140-adds-vnet11**. 
1. Na folha **az140-adds-vnet11**, selecione **Sub-redes**, na folha **Sub-redes**, selecione **+ Sub-rede**, na folha **Adicionar sub-rede**, especifique as seguintes configurações (deixe todas as outras configurações com seus valores padrão) e clique em **Salvar**:

   |Configuração|Valor|
   |---|---|
   |Nome|**hp2-Subnet**|
   |Intervalo de endereços da sub-rede|**10.0.2.0/24**|

1. Na sessão do Bastion para **az140-dc-vm11**, no portal do Azure, use a caixa de texto **Pesquisar recursos, serviços e documentos** na parte superior da página do portal do Azure para pesquisar e navegar até **Grupos de Segurança de Rede** e, na folha **Grupos de Segurança de Rede**, selecione o único grupo de segurança de rede.
1. Na folha do grupo de segurança de rede, no menu vertical à esquerda, na seção **Configurações**, clique em **Propriedades**.
1. Na folha **Propriedades**, clique no ícone **Copiar para área de transferência** no lado direito da caixa de texto **ID do Recurso**. 

   > **Observação**: O valor deve ser semelhante ao formato `/subscriptions/de8279a3-0675-40e6-91e2-5c3728792cb5/resourceGroups/az140-11-RG/providers/Microsoft.Network/networkSecurityGroups/az140-cl-vm11-nsg`, embora a ID da assinatura seja diferente. Registre-o, pois você precisará dele na próxima tarefa.
1. Agora você deve ter **seis** valores registrados. Um nome diferenciado, três nomes de entidade de usuário, um valor DateTime e a ID do recurso. Se você não tiver seis valores registrados, leia esta tarefa novamente **antes** de continuar. 

#### Tarefa 2: Implantar um pool de hosts e hosts da Área de Trabalho Virtual do Azure usando um modelo do Azure Resource Manager

1. No seu computador de laboratório, inicie um navegador da Web, navegue até o [portal do Azure](https://portal.azure.com) e entre fornecendo credenciais de uma conta de usuário com a função Proprietário na assinatura que você usará nesse laboratório.
1. No computador do laboratório, na mesma janela do navegador da Web, abra outra guia do navegador da Web e navegue até a página de repositório de modelos do GitHub Azure RDS [Modelo ARM para criar e provisionar novo hostpool da Área de Trabalho Virtual do Azure](https://github.com/Azure/RDS-Templates/tree/master/ARM-wvd-templates/CreateAndProvisionHostPool). 
1. Na página **Modelo ARM para criar e provisionar novo hostpool da Área de Trabalho Virtual do Azure**, selecione **Implantar no Azure**. Isso redirecionará automaticamente o navegador para a folha de **Implantação personalizada** no portal do Azure.
1. Na folha **Implantação personalizada**, selecione **Editar parâmetros**.
1. Na folha **Editar parâmetros**, selecione **Carregar arquivo**, na caixa de diálogo **Abrir**, selecione **\\\\AZ-140\\AllFiles\\Labs\\02\\az140-23_azuredeployhp23.parameters.json**, selecione **Abrir** e, em seguida, selecione **Salvar**. 
1. De volta à folha de **Implantação personalizada**, especifique as seguintes configurações (deixe outras com seus valores existentes):

   |Configuração|Valor|
   |---|---|
   |Assinatura|o nome da assinatura do Azure que você está usando neste laboratório|
   |Grupo de recursos|criar um **novo** grupo de recursos chamado **az140-23-RG**|
   |Region|o nome da região do Azure na qual você implantou VMs do Azure que hospedam controladores de domínio do AD DS no laboratório **Preparar para a implantação da Área de Trabalho Virtual do Azure (AD DS)**|
   |Localidade|o nome da mesma região do Azure que o conjunto como o valor dos parâmetros de **Região**|
   |Localização do workspace|o nome da mesma região do Azure que o conjunto como o valor dos parâmetros de **Região**|
   |Grupo de Recursos do Workspace|nenhum, uma vez que, se nulo, seu valor será definido automaticamente para corresponder ao grupo de recursos de destino de implantação|
   |Todas as referências do grupo de aplicativos|nenhum, já que não há grupos de aplicativos existentes no workspace de destino (não há workspace)|
   |Local da VM|o nome da mesma região do Azure que aquele definido como o valor dos parâmetros de **Localização** |
   |Criar um grupo de segurança de rede|**false**|
   |ID do Grupo de Segurança de Rede|o valor do parâmetro resourceID do grupo de segurança de rede existente que você identificou na tarefa anterior|
   |Tempo de Expiração do Token| o valor do tempo de expiração do token calculado na tarefa anterior|

   > **Observação**: A implantação provisiona um pool com tipo de atribuição de área de trabalho pessoal.

1. Na folha **Implantação personalizada**, selecione **Examinar + criar** e selecionar **Criar**.

   > **Observação**: Aguarde a conclusão da implantação antes de prosseguir para a próxima tarefa. Isso pode levar cerca de 15 minutos. 

#### Tarefa 3: Verificar a implantação do pool de hosts e hosts da Área de Trabalho Virtual do Azure

1. Do computador de laboratório, no navegador da Web que exibe o portal do Azure, pesquise e selecione a **Área de Trabalho Virtual do Azure**, na folha **Área de Trabalho Virtual do Azure**, selecione **Pools de host** e, na folha **Pools de host\| da Área de Trabalho Virtual do Azure**, selecione a entrada **az140-23-hp2** que representa o pool recém-modificado.
1. Na folha **az140-23-hp2 **, no menu vertical no lado esquerdo, na seção **Gerenciar**, clique em **Hosts de sessão**. 
1. Na folha **Hosts de sessão\|az140-23-hp2**, verifique se a implantação consiste em dois hosts.
1. Na folha **Hosts de sessão\|az140-23-hp2**, no menu vertical no lado esquerdo, na seção **Gerenciar**, clique em **Grupos de aplicativos**.
1. Na folha **Grupos de aplicativos\|az140-23-hp2**, verifique se a implantação inclui o grupo de aplicativos da **Área de Trabalho Padrão** chamado **az140-23-hp2-DAG**.

#### Tarefa 4: Prepare-se para adicionar hosts ao pool de host da Área de Trabalho Virtual do Azure existente usando um modelo do Azure Resource Manager

1. No computador do laboratório, alterne para a sessão Bastion para **az140-dc-vm11**. 
1. Na sessão do Bastion para **az140-cl-vm11**, no ISE do Windows Power Shell** Console ISE do Windows PowerShell**: execute o seguinte para gerar o token necessário para ingressar novos hosts no pool provisionado anteriormente neste exercício:

   ```powershell
   $registrationInfo = New-AzWvdRegistrationInfo -ResourceGroupName 'az140-23-RG' -HostPoolName 'az140-23-hp2' -ExpirationTime $((get-date).ToUniversalTime().AddDays(1).ToString('yyyy-MM-ddTHH:mm:ss.fffffffZ'))
   ```

1. Na sessão do Bastion para **az140-cl-vm11**, no ISE do Windows Power Shell** Console do ISE do Windows PowerShell**: execute o seguinte para recuperar o valor do token e cole-o na Área de Transferência:

   ```powershell
   $registrationInfo.Token | clip
   ```

   > **Observação**: registre o valor copiado na Área de Transferência (por exemplo, iniciando o Bloco de Notas e pressionando a combinação de teclas Ctrl+V para colar o conteúdo da Área de Transferência no Bloco de Notas) uma vez você irá precisar dele na próxima tarefa. Verifique se o valor que você está usando inclui uma única linha de texto, sem quebras de linha. 

   > **Observação**: Um token de registro é necessário para autorizar um host a ingressar no pool. O valor da data de validade do token deve estar entre uma hora e um mês a partir da data e hora atuais.

#### Tarefa 5: Adicionar hosts ao pool de host da Área de Trabalho Virtual do Azure existente usando um modelo do Azure Resource Manager

1. No computador do laboratório, na mesma janela do navegador da Web, abra outra guia do navegador da Web e navegue até a página do repositório de modelos GitHub Azure RDS [Modelo do ARM para adicionar sessionhosts a um hostpool da Área de Trabalho Virtual do Azure existente](https://github.com/Azure/RDS-Templates/tree/master/ARM-wvd-templates/AddVirtualMachinesToHostPool). 
1. Na página **Modelo do ARM para adicionar sessionhosts a um hostpool d Área de Trabalho Virtual do Azure existente**, selecione **Implantar no Azure**. Isso redirecionará automaticamente o navegador para a folha de **Implantação personalizada** no portal do Azure.
1. Na folha **Implantação personalizada**, selecione **Editar parâmetros**.
1. Na folha **Editar parâmetros**, selecione **Carregar arquivo**, na caixa de diálogo **Abrir**, selecione **\\\\AZ-140\\AllFiles\\Labs\\02\\az140-23_azuremodifyhp23.parameters.json**, selecione **Abrir** e, em seguida, selecione **Salvar**. 
1. De volta à folha de **Implantação personalizada**, especifique as seguintes configurações (deixe outras com seus valores existentes):

   |Configuração|Valor|
   |---|---|
   |Assinatura|o nome da assinatura do Azure que você está usando neste laboratório|
   |Grupo de recursos|**az140-23-RG**|
   |Token do hostpool|o valor do token gerado na tarefa anterior|
   |Local do hostpool|o nome da região do Azure na qual você implantou o hostpool anteriormente neste laboratório|
   |Local da VM|o nome da mesma região do Azure que a que foi definida como o valor dos parâmetros de **Local** do Hostpool|
   |Criar um grupo de segurança de rede|**false**|
   |ID do Grupo de Segurança de Rede|o valor do parâmetro resourceID do grupo de segurança de rede existente que você identificou na tarefa anterior|

1. Na folha **Implantação personalizada**, selecione **Examinar + criar** e selecionar **Criar**.

   > **Observação**: Aguarde a conclusão da implantação antes de prosseguir para a próxima tarefa. Isso pode levar cerca de 10 minutos.

#### Tarefa 6: Verificar alterações no pool de host da Área de Trabalho Virtual do Azure

1. No seu computador de laboratório, no navegador da Web que exibe o portal do Azure, pesquise e selecione **Máquinas Virtuais** e, na folha **Máquinas Virtuais**, observe que a lista inclui uma máquina virtual adicional chamada **az140-23-p2-2**.
1. No computador do laboratório, alterne para a sessão Bastion para **az140-dc-vm11**. 
1. Na sessão do Bastion para **az140-cl-vm11**, no ISE do Windows Power Shell** Console do ISE do Windows PowerShell**: execute o seguinte para verificar se o terceiro host foi associado com êxito ao **domínio adatum.com** AD DS:

   ```powershell
   Get-ADComputer -Filter "sAMAccountName -eq 'az140-23-p2-2$'"
   ```
1. Alterne para o computador de laboratório, no navegador da Web que exibe o portal do Azure, pesquise e selecione a **Área de Trabalho Virtual do Azure**, na folha **Área de Trabalho Virtual do Azure**, selecione **Pools de host** e, na folha **Pools de host\| da Área de Trabalho Virtual do Azure**, selecione a entrada **az140-23-hp2** que representa o pool recém-modificado.
1. Na folha **az140-23-hp2**, examine a seção **Noções Básicas** e verifique se o **Tipo de pool de host** está definido como **Pessoal** com o **Tipo atribuição** definido como **Automático**.
1. Na folha **az140-23-hp2 **, no menu vertical no lado esquerdo, na seção **Gerenciar**, clique em **Hosts de sessão**. 
1. Na folha **Hosts de sessão\|az140-23-hp2**, verifique se a implantação consiste em três hosts. 

#### Tarefa 7: Gerenciar atribuições de área de trabalho pessoal no pool de host da Área de Trabalho Virtual do Azure

1. No seu computador de laboratório, no navegador da Web que exibe o portal do Azure, na folha **hosts de sessão\|az140-23-hp2**, no menu vertical à esquerda, na seção **Gerenciar**, selecione **Grupos de aplicativos**. 
1. Na folha **Grupos de aplicativos\| az140-23-hp2**, na lista de grupos de aplicativos, selecione **az140-23-hp2-DAG**.
1. Na folha **az140-23-hp2-DAG**, no menu vertical à esquerda, selecione **Atribuições**. 
1. Na folha **az140-23-hp2-DAG \| Atribuições**, selecione** + Adicionar **.
1. Na folha **Selecionar usuários ou grupos de usuários do Microsoft Entra**, selecione**Grupos** e selecione **az140-wvd-personal** e clique em **Selecionar**.

   > **Observação**: Agora vamos examinar a experiência de um usuário que se conecta ao pool de host da Área de Trabalho Virtual do Azure.

1. No seu computador de laboratório, na janela do navegador que exibe o portal do Azure, pesquise e selecione **Máquinas Virtuais** e, na folha **Máquinas Virtuais**, selecione a entrada **az140-cl-vm11**.
1. Na folha **az140-cl-vm11**, selecione **Conectar**, no menu suspenso, selecione **Conectar via Bastion**.
1. Quando solicitado, forneça as seguintes credenciais e selecione **Conectar**:

   |Configuração|Valor|
   |---|---|
   |Nome do usuário|**Student@adatum.com**|
   |Senha|**Pa55w.rd1234**|

1. Na sessão do Bastion para **az140-cl-vm11**, clique em **Iniciar** e, no menu **Iniciar**, clique em **Área de Trabalho Remota** para iniciar o cliente da Área de Trabalho Remota.
2. Na janela Área de Trabalho Remota, clique no ícone de reticências no canto superior direito, no menu suspenso, clique em **Cancelar assinatura** e, quando solicitado, clique em **Continuar**.
3. Na sessão Bastion para **az140-cl-vm11**, na janela Área de Trabalho Remota, na **página Vamos começar**, clique em **Assinar**.
4. Na janela cliente da **Área de Trabalho Remota**, selecione **Assinar** e, quando solicitado, entre com as credenciais **aduser7** fornecendo seu userPrincipalName e a senha definida ao criar essa conta de usuário.

   > **Observação**: como alternativa, na janela cliente da **Área de Trabalho Remota**, selecione **Assinar com a URL**, no painel **Assinar um Workspace**, na **URL de Email ou Workspace**, digite **https://client.wvd.microsoft.com/api/arm/feeddiscovery**, selecione **Avançar**e, uma vez solicitado, entre com as credenciais **aduser7** (usando seu atributo userPrincipalName como o nome de usuário e a senha definida ao criar essa conta). 

1. **Na página Área de Trabalho Remota**, clique duas vezes no ícone **SessionDesktop**, quando solicitado a inserir as credenciais, digite a mesma senha novamente, selecione a caixa de seleção **Lembrar-me** e clique em **OK**.
1. Verifique se o **aduser7** entrou com êxito por meio da Área de Trabalho Remota em um host.
1. Na sessão da Área de Trabalho Remota para um dos hosts como **aduser7**, clique com o botão direito do mouse em **Iniciar**, no menu com o botão direito do mouse, selecione **Desligar ou sair** e, no menu em cascata, clique em **Sair**.

   > **Observação**: Agora, vamos alternar a atribuição de área de trabalho pessoal do modo direto para o automático. 

1. Alterne para o computador de laboratório, para o navegador da Web que exibe o portal do Azure, na folha **Atribuições\|az140-23-hp2-DAG **, na barra informativa diretamente acima da lista de atribuições, clique no link **Atribuir VM**. Isso redirecionará você para a folha **Hosts de sessão\|az140-23-hp2**. 
1. Na folha **Hosts de sessão\|az140-23-hp2 **, verifique se um dos hosts tem **aduser7** listado na coluna **Usuário Atribuído**.

   > **Observação**: isso é esperado, pois o pool de hosts está configurado para atribuição automática.

1. No computador do laboratório, na janela do navegador da Web que exibe o portal do Azure, abra a sessão do shell do **PowerShell** no painel do **Cloud Shell**.
1. Na sessão do PowerShell no painel do Cloud Shell, execute o seguinte para alternar para o modo de atribuição direta:

    ```powershell
    Update-AzWvdHostPool -ResourceGroupName 'az140-23-RG' -Name 'az140-23-hp2' -PersonalDesktopAssignmentType Direct
    ```

1. No computador do laboratório, na janela do navegador da Web que exibe o portal do Azure, navegue até a folha pool de hosts **az140-23-hp2**, examine a seção **Noções Básicas** e verifique se o **Tipo de pool de host** está definido como **Pessoal** com o **Tipo de atribuição** definido como **Direta**.
1. Volte para a sessão do Bastion para **az140-cl-vm11**, na janela **Área de Trabalho Remota**, clique no ícone de reticências no canto superior direito, no menu suspenso, clique em **Cancelar assinatura** e, quando solicitado, clique em **Continuar**.
1. Na sessão Bastion para **az140-cl-vm11**, na janela **Área de Trabalho Remota**, na página **Vamos começar**, clique em **Assinar**.
1. Quando solicitado a entrar, no painel **Escolher uma conta**, clique em **Usar outra conta** e, quando solicitado, entre usando o nome principal do usuário da conta de usuário **aduser8** com a senha definida ao criar essa conta.
1. Na página **Área de Trabalho Remota**, clique duas vezes no ícone **SessionDesktop**, verifique se você recebeu uma mensagem de erro informando que **Não foi possível conectar porque não há recursos disponíveis no momento. Tente novamente mais tarde ou entre em contato com o suporte técnico para obter ajuda se isso continuar acontecendo** e clique em **OK**.

   > **Observação**: Isso é esperado, pois o pool de hosts está configurado para atribuição direta e não foi atribuído um host a **aduser8**.

1. Alterne para o seu computador de laboratório, para o navegador da Web que exibe o portal do Azure e, na folha **Hosts de sessão\|az140-23-hp2**, selecione o link **(Atribuir)** na coluna **Usuário Atribuído** ao lado de um dos dois hosts não atribuídos restantes.
1. Na opção **Atribuir um usuário**, selecione **aduser8**, clique em **Selecionar** e, quando solicitado a confirmar, clique em **OK**.
1. Volte para a sessão do Bastion para **az140-cl-vm11**, na janela **Área de Trabalho Remota**, clique duas vezes no ícone **SessionDesktop**, quando for solicitada a senha, digite a senha definida ao criar essa conta de usuário, clique em **OK** e verifique se você consegue entrar com êxito no host atribuído.
1. Na Área de Trabalho de Sessão para o host atribuído para **aduser8**, clique com o botão direito do mouse em **Iniciar**, no menu com o botão direito do mouse, selecione **Desligar ou sair** e, no menu em cascata, clique em **Sair**.
1. Na sessão do Bastion para **az140-cl-vm11**, clique com o botão direito do mouse em **Iniciar**, no menu com o botão direito do mouse, selecione **Desligar ou sair** e, no menu em cascata, clique em **Sair**e clique em **Fechar**.

### Exercício 2: Parar e desalocar VMs do Azure provisionadas no laboratório

As principais tarefas desse exercício são as seguintes:

1. Parar e desalocar VMs do Azure provisionadas no laboratório

>**Observação**: Nesse exercício, você desalocará as VMs do Azure provisionadas neste laboratório para minimizar os encargos de computação correspondentes

#### Tarefa 1: Desalocar VMs do Azure provisionadas no laboratório

1. Alterne para o computador de laboratório e, na janela do navegador da Web que exibe o portal do Azure, abra a sessão do Shell do **PowerShell** no painel do **Cloud Shell**.
1. Na sessão do PowerShell no painel do Cloud Shell, execute o seguinte para listar todas as VMs do Azure criadas neste laboratório:

   ```powershell
   Get-AzVM -ResourceGroup 'az140-23-RG'
   ```

1. Na sessão do PowerShell no painel do Cloud Shell, execute o seguinte para parar e desalocar todas as VMs do Azure criadas neste laboratório:

   ```powershell
   Get-AzVM -ResourceGroup 'az140-23-RG' | Stop-AzVM -NoWait -Force
   ```

   >**Observação**: O comando é executado de modo assíncrono (conforme determinado pelo parâmetro -NoWait), portanto, embora você possa executar outro comando do PowerShell imediatamente depois na mesma sessão do PowerShell, levará alguns minutos antes de o grupo de recursos ser de fato removido.
