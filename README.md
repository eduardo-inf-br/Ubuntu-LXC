# Implantação de Contêineres LXC no Ubuntu.

## Introdução ao LXC

Linux Containers (LXC) é uma tecnologia de virtualização em nível de sistema operacional que permite executar múltiplos sistemas Linux isolados (contêineres) em um único host Linux. Diferente das máquinas virtuais tradicionais, que emulam hardware completo, os contêineres LXC compartilham o kernel do sistema operacional host, resultando em menor sobrecarga e maior eficiência. Isso os torna ideais para ambientes de desenvolvimento, testes e até mesmo produção, onde a agilidade e o uso eficiente de recursos são cruciais.

Este guia detalhado abordará a instalação, configuração, criação e gerenciamento de contêineres LXC em sistemas Ubuntu, fornecendo as informações necessárias para que você possa começar a utilizar essa poderosa ferramenta de virtualização.

## 1. Instalação do LXC no Ubuntu

A instalação do LXC no Ubuntu é um processo direto, geralmente envolvendo o uso do gerenciador de pacotes `apt`. É recomendável atualizar o sistema antes de iniciar a instalação para garantir que você tenha os pacotes mais recentes disponíveis [2, 4].

### 1.1. Atualização do Sistema

Antes de instalar qualquer novo software, é uma boa prática atualizar a lista de pacotes e os pacotes existentes no seu sistema Ubuntu:

```bash
sudo apt update
sudo apt upgrade -y
```

### 1.2. Instalação dos Pacotes LXC

Após a atualização do sistema, você pode instalar o LXC e suas dependências. O pacote principal é `lxc`, mas também é útil instalar `lxc-utils` para ferramentas adicionais de gerenciamento e `lxd` para uma experiência mais moderna e gerenciada de contêineres, embora este guia se concentre principalmente no LXC puro [4, 6].

```bash
sudo apt install lxc lxc-utils -y
```

### 1.3. Verificação da Instalação

Para confirmar que o LXC foi instalado corretamente, você pode verificar a versão do LXC ou listar os contêineres (inicialmente, não haverá nenhum):

```bash
lxc-checkconfig
lxc-ls --fancy
```

O comando `lxc-checkconfig` verificará se o kernel do seu sistema está configurado para suportar LXC, exibindo `enabled` para os recursos necessários. Se algum recurso estiver desabilitado, pode ser necessário atualizar o kernel ou habilitar módulos específicos, embora em instalações modernas do Ubuntu isso raramente seja um problema.

## 2. Configuração Inicial do LXC

Após a instalação, algumas configurações iniciais podem ser necessárias, especialmente se você planeja usar contêineres não privilegiados ou configurar redes mais complexas. Por padrão, o LXC pode ser usado com contêineres privilegiados, mas para maior segurança, contêineres não privilegiados são preferíveis [8].

### 2.1. Configuração de Rede (Opcional)

O LXC pode configurar uma ponte de rede padrão para seus contêineres, permitindo que eles obtenham endereços IP e acessem a rede externa. Se você instalou o `lxd`, ele geralmente lida com a configuração de rede automaticamente. Para LXC puro, você pode precisar configurar `lxc-net` ou uma ponte manual.

Para verificar a configuração de rede padrão:

```bash
cat /etc/default/lxc-net
```

Se você deseja que usuários não-root criem e gerenciem contêineres, é necessário configurar `subuid` e `subgid` para mapeamento de IDs de usuário e grupo. Isso é crucial para a segurança de contêineres não privilegiados.

### 2.2. Configuração de Usuários Não Privilegiados

Para permitir que um usuário comum (não-root) crie e gerencie contêineres LXC não privilegiados, você precisa configurar o mapeamento de IDs de usuário e grupo. Adicione as seguintes linhas aos arquivos `/etc/subuid` e `/etc/subgid` para o seu usuário (substitua `seu_usuario` pelo seu nome de usuário real):

```bash
echo "seu_usuario:100000:65536" | sudo tee -a /etc/subuid
echo "seu_usuario:100000:65536" | sudo tee -a /etc/subgid
```

Isso mapeia um intervalo de 65536 UIDs e GIDs começando em 100000 para o seu usuário dentro do contêiner. Após essas alterações, você precisará reiniciar o sistema ou fazer logout e login novamente para que as alterações entrem em vigor.

## 3. Criação de Contêineres LXC

Com o LXC instalado e configurado, você pode começar a criar seus contêineres. O LXC utiliza modelos (templates) para criar contêineres baseados em diferentes distribuições Linux [5].

### 3.1. Listar Modelos Disponíveis

Você pode listar os modelos de contêineres disponíveis com o seguinte comando:

```bash
ls /usr/share/lxc/templates
```

Modelos comuns incluem `lxc-ubuntu`, `lxc-debian`, `lxc-fedora`, etc.

### 3.2. Criar um Novo Contêiner

Para criar um novo contêiner Ubuntu chamado `meu-primeiro-container`, use o comando `lxc-create`:

```bash
sudo lxc-create -t ubuntu -n meu-primeiro-container
```

Este comando baixará a imagem base do Ubuntu e configurará o contêiner. O processo pode levar alguns minutos, dependendo da sua conexão com a internet. Você pode especificar a versão do Ubuntu usando a opção `-r` (release), por exemplo, `-r focal` para Ubuntu 20.04 LTS.

Para criar um contêiner não privilegiado, você precisará usar o comando como seu usuário e especificar o diretório de armazenamento:

```bash
lxc-create -t ubuntu -n meu-container-nao-privilegiado --dir ~/.local/share/lxc
```

### 3.3. Configuração do Contêiner

Após a criação, o arquivo de configuração do contêiner estará localizado em `/var/lib/lxc/meu-primeiro-container/config` (para contêineres privilegiados) ou `~/.local/share/lxc/meu-container-nao-privilegiado/config` (para contêineres não privilegiados). Você pode editar este arquivo para ajustar parâmetros como rede, recursos de CPU/memória, etc.

Por exemplo, para definir um hostname e configurar a rede para DHCP:

```ini
lxc.uts.name = meu-primeiro-container
lxc.net.0.type = veth
lxc.net.0.link = lxcbr0
lxc.net.0.flags = up
lxc.net.0.hwaddr = 00:16:3e:xx:xx:xx
```

## 4. Gerenciamento de Contêineres LXC

O gerenciamento de contêineres LXC é feito através de uma série de comandos `lxc-*`.

### 4.1. Iniciar um Contêiner

Para iniciar um contêiner:

```bash
sudo lxc-start -n meu-primeiro-container -d
```

A opção `-d` inicia o contêiner em segundo plano (daemon).

### 4.2. Listar Contêineres

Para ver o status de todos os contêineres:

```bash
sudo lxc-ls --fancy
```

Isso mostrará o nome, estado, endereço IP e outros detalhes dos seus contêineres.

### 4.3. Acessar um Contêiner

Você pode acessar o console de um contêiner em execução usando `lxc-attach` ou `lxc-console`:

```bash
sudo lxc-attach -n meu-primeiro-container
```

Ou, para um console mais tradicional (útil para depuração de boot):

```bash
sudo lxc-console -n meu-primeiro-container
```

Para sair do `lxc-console`, pressione `Ctrl+a` seguido de `q`.

### 4.4. Parar um Contêiner

Para parar um contêiner em execução:

```bash
sudo lxc-stop -n meu-primeiro-container
```

### 4.5. Reiniciar um Contêiner

Para reiniciar um contêiner:

```bash
sudo lxc-stop -n meu-primeiro-container && sudo lxc-start -n meu-primeiro-container -d
```

### 4.6. Deletar um Contêiner

Antes de deletar um contêiner, certifique-se de que ele esteja parado:

```bash
sudo lxc-stop -n meu-primeiro-container
sudo lxc-destroy -n meu-primeiro-container
```

## 5. Boas Práticas e Considerações de Segurança

- **Contêineres Não Privilegiados**: Sempre que possível, utilize contêineres não privilegiados para aumentar a segurança. Eles são mais isolados do sistema host e limitam o impacto de uma possível vulnerabilidade dentro do contêiner [8].
- **Atualizações**: Mantenha o sistema host e os contêineres atualizados para garantir que você tenha as últimas correções de segurança.
- **Recursos**: Monitore o uso de recursos (CPU, memória, disco) dos seus contêineres para evitar que um contêiner consuma todos os recursos do host.
- **Backup**: Implemente uma estratégia de backup para seus contêineres, especialmente se eles contiverem dados importantes.
- **Rede**: Configure as regras de firewall no host para controlar o tráfego de rede para e dos seus contêineres.

## Conclusão

Os contêineres LXC oferecem uma solução leve e eficiente para virtualização no Linux. Este guia cobriu os passos essenciais para instalar, configurar, criar e gerenciar contêineres LXC no Ubuntu. Ao seguir estas instruções e as boas práticas recomendadas, você estará bem equipado para aproveitar os benefícios da conteinerização em seus projetos e ambientes.

## Homepage
[https://informatizar.netlify.app/project/labs/backoffice/containers/ubuntu-lxc](https://informatizar.netlify.app/project/labs/backoffice/containers/ubuntu-lxc)

<br>
## Eduardo Schmidt

**`Informática para Internet`**

Atuando desde 1999 em Tecnologia da Informação, Eduardo Schmidt é um profissional experiente em suporte a soluções de automação comercial, terminais de transferência eletrônica, impressoras de cupons fiscais, sistemas operacionais e servidores da Microsoft e Linux. Plataformas de Hospedagens em Servidores Web. "[Eduardo.Inf.Br](https://informatizar.netlify.app/)".

---

<img 
    align="left" 
    alt="HTML"
    title="HTML" 
    width="30px" 
    style="padding-right: 10px;" 
    src="https://cdn.jsdelivr.net/gh/devicons/devicon@latest/icons/html5/html5-original.svg" 
/>
<img 
    align="left" 
    alt="CSS" 
    title="CSS"
    width="30px" 
    style="padding-right: 10px;" 
    src="https://cdn.jsdelivr.net/gh/devicons/devicon@latest/icons/css3/css3-original.svg" 
/>
<img 
    align="left" 
    alt="JavaScript" 
    title="JavaScript"
    width="30px" 
    style="padding-right: 10px;" 
    src="https://cdn.jsdelivr.net/gh/devicons/devicon@latest/icons/javascript/javascript-original.svg" 
/>
<img 
    align="left" 
    alt="Bootstrap"
    title="Bootstrap" 
    width="30px" 
    style="padding-right: 10px;" 
    src="https://cdn.jsdelivr.net/gh/devicons/devicon@latest/icons/bootstrap/bootstrap-original.svg" 
/>
<img 
    align="left" 
    alt="Git" 
    title="Git"
    width="30px" 
    style="padding-right: 10px;" 
    src="https://cdn.jsdelivr.net/gh/devicons/devicon@latest/icons/git/git-original.svg" 
/>

<br/>

#### Participações | Conhecimentos

<p align="left">
  <img src="https://informatizar.netlify.app/id/img/logo-uolhost.png" alt="UOL Host" width="110"/>
  <img src="https://informatizar.netlify.app/id/img/logo-NCR.png" alt="NCR" width="110"/>
</p>
