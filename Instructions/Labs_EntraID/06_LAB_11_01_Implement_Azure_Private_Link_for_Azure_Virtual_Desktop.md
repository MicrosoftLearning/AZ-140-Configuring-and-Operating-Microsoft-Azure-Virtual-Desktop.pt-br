---
lab:
  title: 'Laboratório: Implementar o Link Privado do Azure para a Área de Trabalho Virtual do Azure'
  module: 'Module 1.1: Plan, implement, and manage networking for Azure Virtual Desktop'
---

# Laboratório: Implementar o Link Privado do Azure para a Área de Trabalho Virtual do Azure
# Manual de laboratório do aluno

## Dependências do laboratório

- O nome da assinatura do Azure que você usará nesse laboratório
- Uma conta de usuário do Microsoft Entra com a função Proprietário na assinatura do Azure que você usará neste laboratório e com permissões suficientes para unir dispositivos ao locatário do Entra associado a essa assinatura do Azure.
- Ter concluído o laboratório *Implantar pools de hosts e hosts de sessão usando o portal do Azure (AD DS)*

## Tempo estimado

40 minutos

## Cenário do laboratório

Você tem um ambiente de Área de Trabalho Virtual do Azure existente. Você precisa implementar a conexão com o ambiente usando o Link Privado do Azure. 

## Objetivos
  
Depois de realizar este laboratório, você será capaz de:

- Implementar o Link Privado do Azure para a Área de Trabalho Virtual do Azure

## Arquivos do laboratório

- Nenhum

## Instruções

### Exercício 1: Implementar o Link Privado do Azure para a Área de Trabalho Virtual do Azure
  
As principais tarefas desse exercício são as seguintes:

1. Registrar novamente o provedor de recursos da Área de Trabalho Virtual do Azure
1. Criar uma sub-rede em uma rede virtual do Azure
1. Implementar um ponto de extremidade privado para conexões com um pool de host
1. Implementar um ponto de extremidade privado para download de feed
1. Implementar um ponto de extremidade privado para descoberta inicial de feed
1. Validar a funcionalidade do ponto de extremidade privado
1. Permitir acesso de rede pública a um pool de host e workspace

> **Observação**: a Área de Trabalho Virtual do Azure tem três fluxos de trabalho com três tipos de recursos correspondentes para usar com pontos de extremidade privados. Estes fluxos de trabalho são:

- **Descoberta do feed inicial**: permite que clientes RDP descubram todos os workspaces atribuídos a um usuário. Para implementar esse fluxo de trabalho por meio de um Link Privado, você precisa criar um único ponto de extremidade privado para o sub-recurso global em qualquer workspace que faça parte da implantação da Área de Trabalho Virtual do Azure. No entanto, independentemente do workspace escolhido, pode haver apenas um único ponto de extremidade privado que forneça essa funcionalidade por implantação
- **Download de feed**: permite que clientes RDP baixem os detalhes de conexão para os workspaces que hospedam os grupos de aplicativos do usuário atual. Para implementar esse fluxo de trabalho por meio de um Link Privado, você precisa criar um ponto de extremidade privado para o sub-recurso de feed para cada workspace que você pretende disponibilizar por meio do ponto de extremidade privado.
- **Conexões com pools de hosts**: permite que clientes RDP e hosts de sessão se conectem a um pool de hosts. Para implementar esse fluxo de trabalho por meio de um Link Privado, você precisa criar um ponto de extremidade privado para o sub-recurso de conexão para cada pool de host que você pretende disponibilizar por meio do ponto de extremidade privado.

> **Observação**: você pode implementar esses fluxos de trabalho nas seguintes disposições:

- Todas as partes da conexão — descoberta de feed inicial, download de feed e conexões de sessão remota para clientes e hosts de sessão — usam rotas privadas.
- O download de feed e as conexões de sessão remota para clientes e hosts de sessão usam rotas privadas, mas a descoberta de feed inicial usa rotas públicas. 
- Somente conexões de sessão remota para clientes e hosts de sessão usam rotas privadas, mas a descoberta de feed inicial e o download de feed usam rotas públicas.
- As VMs de host de sessão e de clientes usam rotas públicas. O Link Privado não é usado neste cenário.

> **Observação**: neste laboratório, você implementará a primeira disposição.

#### Tarefa 1: Registrar novamente o provedor de recursos da Área de Trabalho Virtual do Azure

> **Observação**: antes de usar o Link Privado com a Área de Trabalho Virtual do Azure, você deve registrar novamente o provedor de recursos **Microsoft.DesktopVirtualization**. 

1. Se necessário, no computador do laboratório, inicie um navegador da Web, navegue até o portal do Azure e entre fornecendo credenciais de uma conta de usuário com a função Proprietário na assinatura que você usará neste laboratório.

    > **Observação**: use as credenciais da conta `User1-` listada na guia Recursos no lado direito da janela da sessão de laboratório.

1. No computador do laboratório, no navegador da Web que exibe o portal do Azure, pesquise e selecione **Assinaturas**. Na página **Assinaturas**, selecione a assinatura do Azure que você está usando neste laboratório e, no menu de navegação vertical, na seção **Configurações**, selecione **Provedores de recursos**.
1. Na guia **Provedores de recursos**, na caixa de texto de pesquisa, digite **Microsoft.DesktopVirtualization**. Na lista de resultados, selecione o pequeno círculo à esquerda da entrada **Microsoft.DesktopVirtualization** e selecione **Registrar novamente**.

    > **Observação**: aguarde a conclusão do processo de novo registro. Essa etapa geralmente leva menos de um minuto.

#### Tarefa 2: Criar uma sub-rede de rede virtual do Azure

> **Observação**: você pode usar uma sub-rede existente de uma rede virtual do Azure para implementar pontos de extremidade privados no cenário de laboratório, mas é uma prática comum usar uma sub-rede dedicada para essa finalidade.

1. No computador do laboratório, no navegador da Web que exibe o portal do Azure, pesquise e selecione **Redes virtuais** e, na página **Redes virtuais**, selecione **az140-vnet11e**.
1. Na página **az140-vnet11e**, na seção **Configurações** do menu de navegação vertical, selecione **Sub-redes**.
1. Na página **az140-vnet11e \| Sub-redes**, selecione **+ Sub-rede**.
1. No painel **Adicionar uma sub-rede**, especifique as seguintes configurações e selecione **Adicionar** (deixe as outras configurações com seus valores padrão):

    |Configuração|Valor|
    |---|---|
    |Nome|**pe-Subnet**|
    |Endereço inicial|**10.20.255.0**|
    |Habilitar sub-rede privada (sem acesso de saída padrão)|Desabilitado|

#### Tarefa 3: Implementar um ponto de extremidade privado para conexões com um pool de hosts

1. No computador do laboratório, no navegador da Web que exibe o portal do Azure, pesquise e selecione **Área de Trabalho Virtual do Azure**. Na página **Área de Trabalho Virtual do Azure**, na seção **Gerenciar** do menu de navegação vertical, selecione **Pools de hosts** e, na página **Área de Trabalho Virtual do Azure \| Pools de hosts**, selecione **az140-21-hp1**. 
1. Na página **az140-21-hp1**, no menu de navegação vertical, na seção **Configurações**, selecione **Rede**.
1. Na página **az140-21-hp1 \| Rede**, selecione a guia **Conexões de ponto de extremidade privado** e, em seguida, selecione **+ Novo ponto de extremidade privado**.
1. Na guia **Básico** da página **Criar um ponto de extremidade privado**, especifique as seguintes configurações e selecione **Avançar: Recurso >**:

    |Configuração|Valor|
    |---|---|
    |Assinatura|O nome da assinatura do Azure que você está usando neste laboratório|
    |Grupo de recursos|**az140-11e-RG**|
    |Nome|**az140-11-pehp1**|
    |Nome da Interface de Rede|**az140-11-pehp1-nic**|
    |Region|O nome da região do Azure em que você implantou o ambiente da Área de Trabalho Virtual do Azure|

1. Na guia **Recurso** da página **Criar um ponto de extremidade privado**, especifique as seguintes configurações e selecione **Avançar: Rede virtual >**:

    |Configuração|Valor|
    |---|---|
    |Sub-recurso de destino|**connection**|

1. Na guia **Rede virtual** da página **Criar um ponto de extremidade privado**, especifique as seguintes configurações e selecione **Avançar: DNS >** (deixe as outras configurações com seus valores padrão):

    |Configuração|Valor|
    |---|---|
    |Rede virtual|**az140-vnet11e (az140-11e-RG)**|
    |Sub-rede|**pe-Subnet**|
    |Política de rede para pontos de extremidade privados|**Desabilitado**|
    |Configuração de IP privado|**Alocar dinamicamente endereço IP**|

1. Na guia **DNS** da página **Criar um ponto de extremidade privado**, especifique as seguintes configurações e selecione **Avançar: Tags >**:

    |Configuração|Valor|
    |---|---|
    |Integrar com a zona DNS privado|**Sim**|
    |Assinatura|O nome da assinatura do Azure que você está usando neste laboratório|
    |Grupo de recursos|**az140-11e-RG**|

    > **Observação**: esta etapa resultará na criação de uma zona DNS privada chamada **privatelink.wvd.microsoft.com**.

1. Na guia **Tags** da página **Criar um endpoint privado**, selecione **Avançar: Revisar + criar**.
1. Na guia **Revisar + criar** da página **Criar um ponto de extremidade privado**, selecione **Criar**.

    > **Observação**: aguarde até que a implantação seja concluída. A implantação pode levar até 3 minutos.

    > **Observação**: você precisará criar um ponto de extremidade privado para o sub-recurso de conexão para cada pool de hosts que deseja usar com o Link Privado.

#### Tarefa 4: Implementar um ponto de extremidade privado para download de feed

1. No computador do laboratório, no navegador da Web que exibe o portal do Azure, pesquise e selecione **Área de Trabalho Virtual do Azure** e, na página **Área de Trabalho Virtual do Azure**, selecione **Workspaces**.
1. Na página **Área de Trabalho Virtual do Azure \| Workspaces**, selecione **az140-21-ws1**.
1. Na página **az140-21-ws1**, no menu de navegação vertical, na seção **Configurações**, selecione **Rede**.
1. Na página **az140-21-ws1 \| Rede**, selecione a guia **Conexões de ponto de extremidade privado** e, em seguida, selecione **+ Novo ponto de extremidade privado**.
1. Na guia **Básico** da página **Criar um ponto de extremidade privado**, especifique as seguintes configurações e selecione **Avançar: Recurso >**:

    |Configuração|Valor|
    |---|---|
    |Assinatura|O nome da assinatura do Azure que você está usando neste laboratório|
    |Grupo de recursos|**az140-11e-RG**|
    |Nome|**az140-11-pefeeddwnld**|
    |Nome da Interface de Rede|**az140-11-pefeeddwnld-nic**|
    |Region|O nome da região do Azure em que você implantou o ambiente da Área de Trabalho Virtual do Azure|

1. Na guia **Recurso** da página **Criar um ponto de extremidade privado**, especifique as seguintes configurações e selecione **Avançar: Rede virtual >**:

    |Configuração|Valor|
    |---|---|
    |Sub-recurso de destino|**feed**|

1. Na guia **Rede virtual** da página **Criar um ponto de extremidade privado**, especifique as seguintes configurações e selecione **Avançar: DNS >** (deixe as outras configurações com seus valores padrão):

    |Configuração|Valor|
    |---|---|
    |Rede virtual|**az140-vnet11e (az140-11e-RG)**|
    |Sub-rede|**pe-Subnet**|
    |Política de rede para pontos de extremidade privados|**Desabilitado**|
    |Configuração de IP privado|**Alocar dinamicamente endereço IP**|

1. Na guia **DNS** da página **Criar um ponto de extremidade privado**, especifique as seguintes configurações e selecione **Avançar: Tags >**:

    |Configuração|Valor|
    |---|---|
    |Integrar com a zona DNS privado|**Sim**|
    |Assinatura|O nome da assinatura do Azure que você está usando neste laboratório|
    |Grupo de recursos|**az140-11e-RG**|

    > **Observação**: esta etapa aproveitará a zona DNS privada chamada **privatelink.wvd.microsoft.com** que você criou na tarefa anterior.

1. Na guia **Tags** da página **Criar um endpoint privado**, selecione **Avançar: Revisar + criar**.
1. Na guia **Revisar + criar** da página **Criar um ponto de extremidade privado**, selecione **Criar**.

    > **Observação:** prossiga para a próxima etapa sem aguardar a conclusão da implantação. A implantação pode levar cerca de 1 minuto.

    > **Observação**: você precisa criar um ponto de extremidade privado para o sub-recurso de feed para cada espaço de trabalho que deseja usar com o Link Privado.

#### Tarefa 5: Implementar um ponto de extremidade privado para descoberta de feed inicial

1. No computador do laboratório, no navegador da Web que exibe o portal do Azure, pesquise e selecione **Área de Trabalho Virtual do Azure** e, na página **Área de Trabalho Virtual do Azure**, selecione **Workspaces**.
1. Na página **Área de Trabalho Virtual do Azure \| Workspaces**, selecione **az140-21-ws1**.
1. Na página **az140-21-ws1**, no menu de navegação vertical, na seção **Configurações**, selecione **Rede**.
1. Na página **az140-21-ws1 \| Rede**, selecione a guia **Conexões de ponto de extremidade privado** e, em seguida, selecione **+ Novo ponto de extremidade privado**.
1. Na guia **Básico** da página **Criar um ponto de extremidade privado**, especifique as seguintes configurações e selecione **Avançar: Recurso >**:

    |Configuração|Valor|
    |---|---|
    |Assinatura|O nome da assinatura do Azure que você está usando neste laboratório|
    |Grupo de recursos|**az140-11e-RG**|
    |Nome|**az140-11-pefeeddisc**|
    |Nome da Interface de Rede|**az140-11-pefeeddisc-nic**|
    |Region|O nome da região do Azure em que você implantou o ambiente da Área de Trabalho Virtual do Azure|

1. Na guia **Recurso** da página **Criar um ponto de extremidade privado**, especifique as seguintes configurações e selecione **Avançar: Rede virtual >**:

    |Configuração|Valor|
    |---|---|
    |Sub-recurso de destino|**global**|

1. Na guia **Rede virtual** da página **Criar um ponto de extremidade privado**, especifique as seguintes configurações e selecione **Avançar: DNS >** (deixe as outras configurações com seus valores padrão):

    |Configuração|Valor|
    |---|---|
    |Rede virtual|**az140-vnet11e (az140-11e-RG)**|
    |Sub-rede|**pe-Subnet**|
    |Política de rede para pontos de extremidade privados|**Desabilitado**|
    |Configuração de IP privado|**Alocar dinamicamente endereço IP**|

1. Na guia **DNS** da página **Criar um ponto de extremidade privado**, especifique as seguintes configurações e selecione **Avançar: Tags >**:

    |Configuração|Valor|
    |---|---|
    |Integrar com a zona DNS privado|**Sim**|
    |Assinatura|O nome da assinatura do Azure que você está usando neste laboratório|
    |Grupo de recursos|**az140-11e-RG**|

    > **Observação**: esta etapa aproveitará a zona DNS privada chamada **privatelink.wvd.microsoft.com** que você criou em uma das tarefas anteriores.

1. Na guia **Tags** da página **Criar um endpoint privado**, selecione **Avançar: Revisar + criar**.
1. Na guia **Revisar + criar** da página **Criar um ponto de extremidade privado**, selecione **Criar**.

    > **Observação:** prossiga para a próxima etapa sem aguardar a conclusão da implantação. A implantação pode levar cerca de 1 minuto.

    > **Observação**: você precisa criar um ponto de extremidade privado para o sub-recurso de feed para cada espaço de trabalho que deseja usar com o Link Privado.

    > **Observação**: para que as alterações de rede entrem em vigor, você precisa reiniciar os hosts da sessão no pool de host de destino.

1. No computador do laboratório, no navegador da Web que exibe o portal do Azure, navegue até a página **Área de Trabalho Virtual do Azure**, na seção **Gerenciar** do menu de navegação vertical, selecione **Pools de hosts** e, na página **Área de Trabalho Virtual do Azure \| Pools de hosts**, selecione **az140-21-hp1**.
1. Na página **az140-21-hp1**, na seção **Gerenciar** do menu de navegação vertical, selecione **Hosts de sessão**. 
1. Na lista de hosts de sessão, marque todas as caixas de seleção à esquerda de cada host de sessão e selecione **Reiniciar** na barra de ferramentas.

    > **Observação**: aguarde até que todos os hosts de sessão estejam no estado **Em execução**. 

#### Tarefa 6: Validar a funcionalidade do ponto de extremidade privado

> **Observação**: por padrão, a conectividade com workspaces da Área de Trabalho Virtual do Azure e pools de host é permitida de redes públicas. Você começará alterando as configurações padrão e aplicando o acesso privado.

1. No computador do laboratório, no navegador da Web que exibe o portal do Azure, pesquise e selecione **Área de Trabalho Virtual do Azure** e, na página **Área de Trabalho Virtual do Azure**, selecione **Workspaces**.
1. Na página **Área de Trabalho Virtual do Azure \| Workspaces**, selecione **az140-21-ws1**.
1. Na página **az140-21-ws1**, no menu de navegação vertical, na seção **Configurações**, selecione **Rede**.
1. Na página **az140-21-ws1 \| Rede**, na guia **Acesso público**, selecione a opção **Desativar acesso público e usar acesso privado** e, em seguida, selecione **Salvar**.
1. No computador do laboratório, no navegador da Web que exibe o portal do Azure, pesquise e selecione **Área de Trabalho Virtual do Azure**. Na página **Área de Trabalho Virtual do Azure**, na seção **Gerenciar** do menu de navegação vertical, selecione **Pools de hosts** e, na página **Área de Trabalho Virtual do Azure \| Pools de hosts**, selecione **az140-21-hp1**. 
1. Na página **az140-21-hp1**, no menu de navegação vertical, na seção **Configurações**, selecione **Rede**.
1. Na página **az140-21-hp1 \| Rede**, na guia **Acesso público**, selecione a opção **Desativar acesso público e usar acesso privado** e, em seguida, selecione **Salvar**.

    > **Observação**: para validar a funcionalidade do ponto de extremidade privado, um cliente RDP precisa estar conectado a uma rede que tenha conectividade privada com a rede virtual do Azure que contém a sub-rede que hospeda os pontos de extremidade privados que você criou anteriormente neste laboratório. Para simular esse cenário, você criará outra sub-rede na mesma rede virtual usada para criar pontos de extremidade privados e implantará uma VM do Azure executando o Windows 11 nessa sub-rede.

1. No computador do laboratório, no navegador da Web que exibe o portal do Azure, pesquise e selecione **Redes virtuais** e, na página **Redes virtuais**, selecione **az140-vnet11e**.
1. Na página **az140-vnet11e**, na seção **Configurações** do menu de navegação vertical, selecione **Sub-redes**.
1. Na página **az140-vnet11e \| Sub-redes**, selecione **+ Sub-rede**.
1. No painel **Adicionar uma sub-rede**, especifique as seguintes configurações e selecione **Adicionar** (deixe as outras configurações com seus valores padrão):

    |Configuração|Valor|
    |---|---|
    |Nome|**client-Subnet**|
    |Endereço inicial|**10.20.2.0**|
    |Habilitar sub-rede privada (sem acesso de saída padrão)|Desabilitado|

1. No computador do laboratório, no navegador da Web que exibe o portal do Azure, pesquise e selecione **Máquinas virtuais**. Na página **Máquinas virtuais**, selecione **+ Criar** e, na lista suspensa, selecione **Máquina virtual do Azure**.
1. Na guia **Básico** da página **Criar uma máquina virtual**, especifique as seguintes configurações (deixe as outras configurações com seus valores padrão) e selecione **Avançar: Discos >**:

    |Configuração|Valor|
    |---|---|
    |Assinatura|O nome da assinatura do Azure que você está usando neste laboratório|
    |Grupo de recursos|O nome de um novo grupo de recursos **az140-111e-RG**|
    |Nome da máquina virtual|**az140-111e-vm0**|
    |Region|O nome da região do Azure em que você implantou o ambiente da Área de Trabalho Virtual do Azure|
    |Opções de disponibilidade|**Nenhuma redundância de infraestrutura necessária**|
    |Tipo de segurança|**Standard**|
    |Imagem|**Windows 11 Pro, versão 23H2 - x64 Gen2**|
    |Tamanho|**Standard DC2s_v3**|
    |Nome de Usuário|Qualquer nome de usuário válido de sua escolha|
    |Senha|Qualquer senha válida de sua escolha|
    |Porta de entrada públicas|**Nenhuma**|
    |Licenciamento|Marque a caixa de seleção|

    > **Observação**: a senha deve ter pelo menos 12 caracteres e consistir em uma combinação de letras minúsculas, letras maiúsculas, dígitos e caracteres especiais. Para obter detalhes, consulte as informações sobre [os requisitos de senha ao criar uma VM do Azure](https://learn.microsoft.com/en-us/azure/virtual-machines/windows/faq#what-are-the-password-requirements-when-creating-a-vm-).

1. Na guia **Discos** da página **Criar uma máquina virtual**, defina o **Tipo de disco do SO** como **HDD padrão (armazenamento localmente redundante)** e selecione **Avançar: Rede >**.
1. Na guia **Rede** da página **Criar uma máquina virtual**, especifique as seguintes configurações (deixe as outras configurações com seus valores padrão):

    |Configuração|Valor|
    |---|---|
    |Rede virtual|**az140-vnet11e**|
    |Sub-rede|**client-Subnet**|
    |IP público|**(novo) az140-111e-vm0-ip**|
    |Grupo de segurança de rede da NIC|**Avançado**|

1. Na guia **Rede** da página **Criar uma máquina virtual**, ao lado da lista suspensa **Configurar grupo de segurança de rede**, selecione **Criar novo**.
1. Na página **Criar grupo de segurança de rede**, exclua a regra de entrada pré-criada **1000: default-allow-rdp** e selecione **+ Adicionar uma regra de entrada**.
1. No painel **Adicionar regra de segurança de entrada**, na lista suspensa **Origem**, selecione **Meu endereço IP** para identificar o endereço IP público que representa sua conexão com a Internet.
1. No painel **Adicionar regra de segurança de entrada**, especifique as seguintes configurações (deixe as outras configurações com seus valores padrão) e selecione **Adicionar**:

    |Configuração|Valor|
    |---|---|
    |Fonte|**Endereços IP**|
    |Endereços IP de origem/Intervalos de CIDR|Deixe inalterado (ainda deve conter seu endereço IP público)|
    |Intervalos de portas de origem|*|
    |Destino|**Qualquer**|
    |Serviço|**RDP**|
    |Ação|**Permitir**|
    |Prioridade|**300**|
    |Nome|**AllowCidrBlockRDPInbound**|

1. De volta à página **Criar grupo de segurança de rede**, selecione **OK**.
1. De volta à guia  **Rede** da página **Criar uma máquina virtual**, selecione **Avançar: Gerenciamento >**:
1. Na guia **Gerenciamento** da página **Criar uma máquina virtual**, especifique as seguintes configurações (deixe as outras configurações com seus valores padrão) e selecione **Avançar: Monitoramento >**:

    |Configuração|Valor|
    |---|---|
    |Habilitar plano básico gratuitamente|desabilitado|
    |Opções de orquestração de patch|**Atualizações manuais**|

1. Na guia **Monitoramento** da página **Criar uma máquina virtual**, especifique as seguintes configurações (deixe as outras configurações com seus valores padrão) e selecione **Revisar + criar**:

    |Configuração|Valor|
    |---|---|
    |Diagnóstico de inicialização|**Desabilitar**|

1. Na guia **Revisar + criar** da página **Criar uma máquina virtual**, selecione **Criar**.

    > **Observação**: aguarde até que a implantação seja concluída. A implantação pode levar cerca de cinco minutos.

1. No computador do laboratório, no navegador da Web que exibe o portal do Azure, pesquise e selecione **Máquinas virtuais**, na página **Máquinas virtuais**, selecione **az140-111e-vm0**.
1. Na página **az140-111e-vm0**, selecione **Conectar** e, no menu suspenso, selecione **Conectar**.
1. Na página **az140-111e-vm0 \| Conectar**, na seção **Mais comum**, selecione **Baixar arquivo RDP**.
1. Na janela pop-up **Download**, selecione **Manter** e depois selecione **Abrir arquivo**.
1. Quando solicitado, selecione **Conectar** e, na caixa de diálogo **Segurança do Windows**, insira o nome de usuário e a senha especificados ao implantar a VM do Azure.
1. Quando for solicitada a confirmação, selecione **Conectar** novamente.
1. Na sessão da Área de Trabalho Remota para **az140-111e-vm0**, escolha e aceite suas configurações de privacidade preferidas.
1. Na sessão da Área de Trabalho Remota para **az140-111e-vm0**, inicie o Microsoft Edge, navegue até a página [Conectar-se à Área de Trabalho Virtual do Azure com o cliente da Área de Trabalho Remota para Windows](https://learn.microsoft.com/en-us/azure/virtual-desktop/users/connect-windows), role para baixo até a seção **Baixar e instalar o cliente da Área de Trabalho Remota (MSI)** e selecione o link [Windows 64 bits](https://go.microsoft.com/fwlink/?linkid=2139369). 
1. Abra o Explorador de Arquivos, navegue até a pasta **Downloads** e inicie a instalação do arquivo MSI recém-baixado. 
1. Quando solicitado, aceite os termos do contrato de licenciamento e escolha a opção **Instalar para todos os usuários desta máquina**. Se solicitado, aceite o prompt do Controle de Conta de Usuário para prosseguir com a instalação. 
1. Após a conclusão da instalação, certifique-se de que a caixa de seleção **Iniciar a Área de Trabalho Remota quando a instalação sair** esteja marcada e selecione **Concluir** para iniciar o cliente da Área de Trabalho Remota da Microsoft.
1. Na sessão da Área de Trabalho Remota para **az140-111e-vm0**, na janela do cliente **Área de Trabalho Remota**, selecione **Inscrever-se** e, quando solicitado, entre com as credenciais da `User2` conta de usuário Entra ID, que você pode localizar na guia **Recursos** no painel direito da janela da interface do laboratório.

   > **Observação**: Selecione a conta de usuário que é membro do grupo Entra com o prefixo **AVD-RemoteApp**.

1. Certifique-se de que a página **Área de Trabalho Remota** exiba quatro ícones, incluindo Prompt de Comando, Microsoft Word, Microsoft Excel, Microsoft PowerPoint. 
1. Clique duas vezes no ícone do prompt de comando. 
1. Quando solicitado a entrar, na caixa de diálogo **Segurança do Windows**, insira a senha da mesma conta de usuário do Microsoft Entra que você usou para se conectar ao ambiente de destino da Área de Trabalho Virtual do Azure.
1. Verifique se uma janela de **Prompt de Comando** aparece logo depois. 
1. No Prompt de Comando, digite **logoff** e pressione a tecla **Enter** para sair da sessão de Aplicativo Remoto atual.

   > **Observação**: opcionalmente, você pode tentar assinar o feed e se conectar ao workspace da Área de Trabalho Virtual do Azure a partir do computador do laboratório para validar se essa conexão falhará. 

    > **Observação**: para minimizar os custos associados à execução do ambiente de laboratório, você interromperá e desalocará a VM do Azure recém-provisionada.

#### Tarefa 7: Permitir acesso de rede pública a um pool de hosts e workspace

1. No computador do laboratório, no navegador da Web que exibe o portal do Azure, pesquise e selecione **Área de Trabalho Virtual do Azure** e, na página **Área de Trabalho Virtual do Azure**, selecione **Workspaces**.
1. Na página **Área de Trabalho Virtual do Azure \| Workspaces**, selecione **az140-21-ws1**.
1. Na página **az140-21-ws1**, no menu de navegação vertical, na seção **Configurações**, selecione **Rede**.
1. Na página **az140-21-ws1 \| Rede**, na guia **Acesso público**, selecione a opção **Habilitar acesso público de todas as redes** e, em seguida, selecione **Salvar**.
1. No computador do laboratório, no navegador da Web que exibe o portal do Azure, pesquise e selecione **Área de Trabalho Virtual do Azure**. Na página **Área de Trabalho Virtual do Azure**, na seção **Gerenciar** do menu de navegação vertical, selecione **Pools de hosts** e, na página **Área de Trabalho Virtual do Azure \| Pools de hosts**, selecione **az140-21-hp1**. 
1. Na página **az140-21-hp1**, no menu de navegação vertical, na seção **Configurações**, selecione **Rede**.
1. Na página **az140-21-hp1 \| Rede**, na guia **Acesso público**, selecione a opção **Habilitar acesso público de todas as redes** e, em seguida, selecione **Salvar**.
