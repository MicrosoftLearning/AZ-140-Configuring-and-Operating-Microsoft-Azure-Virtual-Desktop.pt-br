---
lab:
  title: 'Laboratório: Implementar e gerenciar o armazenamento para AVD (Microsoft Entra DS)'
  module: 'Module 2: Implement an AVD Infrastructure'
---

# Laboratório: Implementar e gerenciar o armazenamento para AVD (Microsoft Entra DS)
# Manual de laboratório do aluno

## Dependências do laboratório

- Uma assinatura do Azure
- Uma conta Microsoft ou uma conta do Microsoft Entra com a função de administrador global no locatário do Microsoft Entra associado à assinatura do Azure e com a função de proprietário ou colaborador na assinatura
- O laboratório **Preparar para implantação da Área de Trabalho Virtual do Azure (Microsoft Entra DS)** concluído

## Tempo estimado

30 minutos

## Cenário do laboratório

Você precisará implementar e gerenciar o armazenamento para uma implantação da Área de Trabalho Virtual do Azure em um ambiente do Microsoft Entra DS.

## Objetivos
  
Depois de realizar este laboratório, você será capaz de:

- Configurar os Arquivos do Azure para armazenar contêineres de perfil para a Área de Trabalho Virtual do Azure em um ambiente do Microsoft Entra DS

## Arquivos de laboratório

- Nenhum

## Instruções

### Exercício 1: Configurar os Arquivos do Azure para armazenar contêineres de perfil para a Área de Trabalho Virtual do Azure

As principais tarefas desse exercício são as seguintes:

1. Criar uma conta de Armazenamento do Azure
1. Criar um compartilhamento dos Arquivos do Azure
1. Habilitar a autenticação do Microsoft Entra DS para a conta de Armazenamento do Azure 
1. Configurar as permissões de compartilhamento dos Arquivos do Azure
1. Configurar o diretório dos Arquivos do Azure e as permissões de nível de arquivo

#### Tarefa 1: Criar uma conta de Armazenamento do Azure

1. No seu computador de laboratório, inicie um navegador da Web, navegue até o [portal do Azure](https://portal.azure.com) e entre fornecendo credenciais de uma conta de usuário com a função Proprietário na assinatura que você usará nesse laboratório.
1. No seu computador de laboratório, no portal do Azure, pesquise e selecione **Máquinas virtuais** e, na folha **Máquinas virtuais**, selecione a entrada **az140-cl-vm11a**. Isso irá abrir a folha **az140-cl-vm11a**.
1. Na folha **az140-cl-vm11a**, selecione **Conectar**, no menu suspenso, selecione **Bastion**, na guia **Bastion** da folha **az140-cl-vm11a \| Conectar**, selecione **Usar Bastion**.
1. Quando solicitado, forneça as seguintes credenciais e selecione **Conectar**:

   |Configuração|Valor|
   |---|---|
   |Nome do usuário|**aadadmin1@adatum.com**|
   |Senha|Senha definida anteriormente|

1. Na sessão do Bastion para a VM do Azure **az140-cl-vm11a**, inicie o Microsoft Edge, navegue até o [portal do Azure](https://portal.azure.com) e entre fornecendo o nome UPN da conta de usuário a**aadadmin1** e a senha definida ao criar essa conta.

   >**Observação**: Você pode identificar o atributo nome de entidade de usuário (UPN) da conta do **aadadmin1**examinando sua caixa de diálogo de propriedades do console de Usuários e Computadores do Active Directory ou alternando de volta para o seu computador de laboratório e examinando suas propriedades da folha de locatário do Microsoft Entra no portal do Azure.

1. Na sessão do Bastion para **az140-cl-vm11a**, na janela do Microsoft Edge que exibe o portal do Azure, pesquise e selecione **Contas de armazenamento** e, na folha **Contas de Armazenamento**, selecione **+ Criar**.
1. Na guia **Noções básicas** da folha **Criar conta de armazenamento**, defina as seguintes configurações (deixe outras com seus valores padrão):

   |Configuração|Valor|
   |---|---|
   |Assinatura|o nome da assinatura do Azure que você está usando neste laboratório|
   |Grupo de recursos|o nome de um novo grupo de recursos **az140-22a-RG**|
   |Nome da conta de armazenamento|Qualquer nome global exclusivo contendo entre 3 e 15 caracteres composto por dígitos e letras em minúsculas e começando com uma letra|
   |Localidade|o nome de uma região do Azure que hospeda o ambiente de laboratório da Área de Trabalho Virtual do Azure|
   |Desempenho|**Standard**|
   |Replicação|**LRS (armazenamento com redundância local)**|

   >**Observação**: verifique se o comprimento do nome da conta de armazenamento não excede 15 caracteres. O nome será usado para criar uma conta de computador no domínio Active Directory Domain Services (AD DS) integrado ao locatário do Microsoft Entra associado à assinatura do Azure que contém a conta de armazenamento. Isso permitirá a autenticação baseada em AD DS ao acessar compartilhamentos de arquivos hospedados nessa conta de armazenamento.

1. Na guia **Básico** da folha **Criar conta de armazenamento**, selecione **Examinar + Criar**, aguarde a conclusão do processo de validação e selecione **Criar**.

   >**Observação**: aguarde até a conta de armazenamento ser criada. Deve levar cerca de dois minutos.

#### Tarefa 2: Criar um compartilhamento dos Arquivos do Azure

1. Na sessão do Bastion para **az140-cl-vm11a**, na janela do Microsoft Edge que exibe o portal do Azure, navegue de volta para a folha **Contas de Armazenamento** e selecione a entrada que representa a conta de armazenamento recém-criada.
1. Na folha da conta de armazenamento, no menu vertical à esquerda, na seção **Armazenamento de Dados**, selecione **Compartilhamentos de arquivos** e, em seguida, selecione ** + Compartilhamento de arquivos **.
1. Na folha **Novo compartilhamento de arquivos**, especifique as seguintes configurações e selecione **Criar** (deixe outras configurações com seus valores padrão):

   |Configuração|Valor|
   |---|---|
   |Nome|**az140-22a-profiles**|

#### Tarefa 3: Habilitar a autenticação do Microsoft Entra DS para a conta de Armazenamento do Azure

1. Na sessão do Bastion para **az140-cl-vm11a**, na janela do Microsoft Edge, no portal do Azure, na folha que exibe as propriedades da conta de armazenamento que você criou na tarefa anterior, no menu vertical à esquerda, na seção **Armazenamento de dados**, selecione **Compartilhamentos de arquivos**. 
1. Na seção **Configurações de compartilhamento de arquivo**, ao lado do rótulo do **Active Directory**, selecione o link **Não configurado**.
1. Na seção **Habilitar uma origem do Active Directory**, no retângulo rotulado **Azure Active Directory Domain Services**, selecione **Configurar**.
1.  Na folha **Acesso baseado em identidade**, selecione a opção **Habilitado** e selecione **Salvar**.

#### Tarefa 4: Configurar as permissões baseadas em RBAC dos Arquivos do Azure

1. Na sessão do Bastion para **az140-cl-vm11a**, na janela do Microsoft Edge que exibe o portal do Azure, na folha que exibe as propriedades da conta de armazenamento que você criou anteriormente neste exercício, no menu vertical à esquerda, na seção **Armazenamento de Dados**, selecione **Compartilhamentos** de arquivos e, na lista de compartilhamentos, selecione a entrada **az140-22a-profiles**.
1. Na folha **az140-22a-profiles**, no menu vertical no lado esquerdo, selecione **Controle de Acesso (IAM)**.
1. Na folha **az140-22a-profiles\| Controle de Acesso (IAM)** selecione **+ Adicionar** e, no menu suspenso, selecione **Adicionar atribuição de função**.
1. Na folha **Adicionar atribuição de função**, selecione **Colaborador de Compartilhamento SMB de Dados de Arquivo de Armazenamento** e selecione **Avançar**:
1. Na folha **Membros**, selecione **Atribuir acesso** e clique em ** + Selecionar membros **.
1. Na folha **Selecionar Membros**, na caixa de texto **Selecionar texto**, digite **az140-wvd-ausers** e clique em **Selecionar**.
1. Na folha **Membros**, selecione **Examinar + atribuir** duas vezes.
1. Repita as etapas 3 a 8 acima e especifique as seguintes configurações:

   |Configuração|Valor|
   |---|---|
   |Função|**Colaborador elevado de compartilhamento SMB de dados de arquivo de armazenamento**|
   |Selecionar|**az140-wvd-aadmins**|

   > **Observação**: Você usará a conta de usuário **aadadmin1**, que é membro do grupo **az140-wvd-aadmins** para configurar permissões de compartilhamento de arquivos. 

#### Tarefa 5: Configurar o diretório dos Arquivos do Azure e as permissões de nível de arquivo

1. Na sessão do Bastion para **az140-cl-vm11a**, inicie o **Prompt de Comando** e, na janela **Prompt de Comando**, execute o seguinte para mapear uma unidade para o compartilhamento de destino (substitua o espaço reservado `<storage-account-name>` pelo nome da conta de armazenamento):

   ```cmd
   net use Z: \\<storage-account-name>.file.core.windows.net\az140-22a-profiles
   ```

1. Dentro da sessão Bastion para **az140-cl-vm11a**, abra o Explorador de Arquivos, navegue até a unidade Z: recém-mapeada, exiba sua caixa de diálogo**Propriedades**, selecione a guia **Segurança**, selecione **Editar**, selecione **Adicionar**. Na caixa de diálogo **Selecionar Usuários, Computadores, Contas de Serviço e Grupos**, verifique se a caixa de texto **Desse local** contém a entrada **adatum.com**. Na caixa de texto **Digite o nome do objeto a ser selecionado**, digite **az140-wvd-ausers** e clique em **OK**.
1. De volta à guia **Segurança** da caixa de diálogo que exibe as permissões da unidade mapeada, verifique se a entrada **az140-wvd-ausers** está selecionada, marque a caixa de seleção **Modificar** na coluna **Permitir**, clique em **OK **, examine a mensagem exibida na caixa de texto **Segurança do Windows** e clique em **Sim**. 
1. De volta à guia **Segurança** da caixa de diálogo que exibe permissões da unidade mapeada, selecione **Editar**, selecione **Adicionar**, na caixa de diálogo **Selecionar Usuários, Computadores, Contas de Serviço e Grupos**, verifique se a caixa de texto** A partir desse local** contém a entrada **adatum.com**. Na caixa de texto **Digite o nome do objeto para selecionar**, digite **az140-wvd-aadmins** e clique em **OK**.
1. De volta à guia **Segurança** da caixa de diálogo que exibe as permissões da unidade mapeada, verifique se a entrada **az140-wvd-aadmins** está selecionada, marque a caixa de seleção **Controle total** na coluna **Permitir ** e clique em **OK**. 
1. Na guia **Segurança** da caixa de diálogo que exibe as permissões da unidade mapeada, selecione **Editar**, na lista de grupos e nomes de usuário, selecione a entrada **Usuários autenticados** e selecione **Remover**.
1. Enquanto ainda estiver na tela Editar, na lista de grupos e nomes de usuário, selecione a entrada **Usuários**, selecione **Remover**, clique em **OK**e clique em **OK** duas vezes para concluir o processo. 

   >**Observação**: como alternativa, você pode definir permissões usando o utilitário de linha de comando **icacls**. 
