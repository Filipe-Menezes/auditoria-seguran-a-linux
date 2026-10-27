# Auditoria Básica de Segurança Linux

## 1. Sobre o projeto

Este projeto consiste em uma auditoria básica de segurança realizada em uma máquina virtual com Ubuntu Linux, como parte dos meus estudos em cibersegurança.

O objetivo é identificar e analisar configurações relacionadas à segurança do sistema, compreender possíveis riscos e registrar os resultados encontrados.

## 2. Preparação do ambiente

A auditoria está sendo realizada em uma máquina virtual dedicada, utilizando o Oracle VirtualBox e uma instalação do Ubuntu Linux.

O ambiente foi preparado para permitir verificações de segurança em um sistema controlado, sem interferir no sistema operacional principal do computador.

### 2.1. Configurações do ambiente

- **Hipervisor:** Oracle VirtualBox
- **Sistema operacional:** Ubuntu 26.04.1 LTS
- **Codinome:** Resolute Raccoon
- **Kernel inicial identificado:** 7.0.0-38-generic
- **Memória RAM:** 4 GB
- **Processadores virtuais:** 1
- **Armazenamento virtual:** 40 GB
- **Rede:** NAT

### 2.2. Reconhecimento inicial

A primeira etapa consistiu em identificar o sistema operacional e a versão do kernel em execução.

**Verificação 1 — Identificação do sistema operacional**

Comando executado:

```bash
cat /etc/os-release
```

Resultado principal:

```text
PRETTY_NAME="Ubuntu 26.04.1 LTS"
VERSION_ID="26.04"
VERSION_CODENAME=resolute
```

**Análise:** o sistema identificado é o Ubuntu 26.04.1 LTS, com codinome Resolute Raccoon. 

**Verificação 2 — Identificação do kernel**

Comando executado:

```bash
uname -r
```

Resultado:

```text
7.0.0-38-generic
```

**Análise:** o kernel em execução é o 7.0.0-38-generic.

**Limitação:** essas verificações identificam componentes do ambiente, mas não são suficientes para determinar o nível geral de segurança da máquina.

## 3. Objetivos

- Verificar usuários e privilégios administrativos.
- Analisar permissões de arquivos e diretórios importantes.
- Identificar serviços ativos e portas em escuta.
- Examinar configurações básicas do serviço SSH.
- Documentar as descobertas e propor melhorias quando necessário.

## 4. Metodologia

A auditoria será realizada por meio de comandos nativos do Linux e da análise de suas saídas. As verificações serão feitas no ambiente de laboratório, sem realizar testes contra sistemas de terceiros.

Os resultados serão classificados conforme o contexto, os riscos identificados e as limitações de cada verificação.

## 5. Resultados da auditoria

Esta seção documenta as verificações realizadas durante uma auditoria básica de segurança em um ambiente Ubuntu Linux executado em uma máquina virtual VirtualBox.

Para cada verificação, são apresentados o objetivo, o comando utilizado, as evidências coletadas, a análise técnica e, quando aplicável, as recomendações.

Os resultados representam o estado observado durante a auditoria. A ausência de uma falha evidente em determinada verificação não garante que o sistema esteja livre de vulnerabilidades.

### 5.1. Usuários, grupos e privilégios

**Objetivo:** identificar o usuário da sessão, seus grupos e os privilégios administrativos concedidos.

#### Identificação do usuário

**Comando executado:**

```bash
whoami
```

**Resultado:**

```text
ubuntu-auditoria
```

O comando identificou o usuário ativo durante a auditoria.

#### Identificação dos grupos do usuário

**Comando executado:**

```bash
id
```

**Resultado:**

```text
uid=1000(ubuntu-auditoria) gid=1000(ubuntu-auditoria) groups=1000(ubuntu-auditoria),4(adm),24(cdrom),27(sudo),30(dip),46(plugdev),100(users),111(lpadmin),114(lxd)
```

**Análise:**

O usuário possui UID `1000` e pertence, entre outros, aos grupos `sudo` e `lxd`.

O grupo `sudo` está relacionado à autorização para executar comandos administrativos. Já a associação ao grupo `lxd` merece atenção porque o acesso ao serviço LXD pode permitir operações com privilégios elevados, dependendo da configuração.

#### Verificação dos membros do grupo sudo

**Comando executado:**

```bash
getent group sudo
```

**Resultado:**

```text
sudo:x:27:ubuntu-auditoria
```

O resultado mostra `ubuntu-auditoria` como membro listado desse grupo.

#### Verificação dos membros do grupo lxd

**Comando executado:**

```bash
getent group lxd
```

**Resultado:**

```text
lxd:x:114:ubuntu-auditoria
```

O resultado confirma a associação do usuário ao grupo `lxd`.

#### Verificação da instalação do LXD

**Comando executado:**

```bash
dpkg -l lxd
```

**Resultado:**

```text
dpkg-query: nenhum pacote encontrado correspondente a lxd
```

Também foi executado:

```bash
snap list lxd
```

**Resultado:**

```text
error: no matching snaps installed
```

As consultas não encontraram um pacote chamado `lxd` instalado por esses dois métodos.

**Limitação:** isso não exclui todas as formas possíveis de instalação ou configuração do LXD.

#### Verificação dos privilégios do sudo

**Comando executado:**

```bash
sudo -l
```

**Resultado:**

```text
User ubuntu-auditoria may run the following commands on ubuntu-auditoria-VirtualBox:
    (ALL : ALL) ALL
```

**Análise:**

A regra permite ao usuário executar qualquer comando autorizado pelo `sudo`, selecionando usuários e grupos efetivos conforme as regras aplicáveis.

Isso representa privilégios administrativos amplos, comuns em instalações pessoais do Ubuntu. Entretanto, esses privilégios devem ser concedidos apenas a usuários autorizados.

#### Consulta às contas locais

**Comando executado:**

```bash
cut -d: -f1,3,6,7 /etc/passwd
```

A consulta apresentou contas como `root`, `ubuntu-auditoria`, `www-data`, `systemd-resolve` e `vboxadd`.

Entre os resultados observados:

```text
root:0:/root:/bin/bash
www-data:33:/var/www:/usr/sbin/nologin
systemd-resolve:989:/:/usr/sbin/nologin
ubuntu-auditoria:1000:/home/ubuntu-auditoria:/bin/bash
vboxadd:997:/var/run/vboxadd:/bin/false
```

**Análise técnica:**

A consulta identificou a conta administrativa `root`, o usuário principal e diversas contas de serviço.

O uso de shells como `/usr/sbin/nologin` e `/bin/false` é comum em contas que não precisam de uma sessão interativa.

Nenhuma conta claramente suspeita foi identificada apenas com essa consulta. A interpretação definitiva exigiria informações adicionais sobre a finalidade das contas e os mecanismos de autenticação.

**Avaliação:** os privilégios administrativos do usuário foram confirmados. A associação ao grupo `lxd` merece revisão, mas não foi comprovada uma instalação ativa do LXD.

**Recomendações:**
- Manter privilégios administrativos somente para usuários autorizados.
- Revisar periodicamente os grupos privilegiados.
- Verificar a necessidade da associação ao grupo `lxd` antes de alterar a configuração.
- Investigar contas desconhecidas caso sejam encontradas em futuras verificações.

### 5.2. Permissões de arquivos e diretórios

**Objetivo:** verificar as permissões e os proprietários de arquivos importantes para a segurança do sistema.

**Comando executado:**

```bash
ls -ld /etc /etc/passwd /etc/shadow /home/ubuntu-auditoria
```

**Resultado observado:**

```text
drwxr-xr-x 142 root             root             12288 Oct 25 11:09 /etc
-rw-r--r--   1 root             root              2847 Oct 25 11:09 /etc/passwd
-rw-r-----   1 root             shadow            1344 Oct 25 11:09 /etc/shadow
drwxr-x---  15 ubuntu-auditoria ubuntu-auditoria  4096 Oct 27 20:12 /home/ubuntu-auditoria
```

**Análise técnica:**

- `/etc`: diretório de configuração do sistema, com leitura e acesso permitidos conforme as permissões apresentadas.
- `/etc/passwd`: legível por todos os usuários locais, mas com escrita restrita ao proprietário.
- `/etc/shadow`: possui permissões mais restritivas, com leitura concedida ao proprietário e ao grupo `shadow`.
- `/home/ubuntu-auditoria`: restringe o acesso de outros usuários que não sejam o proprietário ou membros do grupo com as permissões indicadas.

**Verificação do arquivo sudoers:**

**Comando executado:**

```bash
ls -l /etc/sudoers
```

O comando original também consultou `/etc/ssh/sshd_config`.

**Resultado observado para sudoers:**

```text
-r--r----- 1 root root 1800 Jan 21 2026 /etc/sudoers
```

O arquivo pertence a `root:root`. As permissões observadas não concedem escrita direta pelas permissões tradicionais.

**Verificação do arquivo de configuração do servidor SSH:**

```bash
ls -l /etc/sudoers /etc/ssh/sshd_config
```

**Resultado para o arquivo SSH:**

```text
ls: não é possível acessar '/etc/ssh/sshd_config': Arquivo ou diretório inexistente
```

Para investigar esse resultado, foi executado:

```bash
dpkg -l openssh-server
```

O estado do pacote foi apresentado como:

```text
un  openssh-server <nenhum> <nenhum> (nenhuma descrição disponível)
```

Isso indica que o pacote `openssh-server` não está instalado conforme o estado registrado pelo `dpkg`.

**Avaliação:** as permissões dos arquivos examinados são compatíveis com configurações comuns do Ubuntu. Não foi identificada uma falha evidente nesses caminhos.

A ausência do arquivo de configuração SSH é consistente com a ausência do pacote do servidor SSH pelo `dpkg`. Isso não constitui, por si só, uma falha de segurança.

**Recomendações:**
- Preservar permissões restritivas nos arquivos de configuração e credenciais.
- Evitar permissões de escrita desnecessárias em arquivos administrativos.
- Instalar um servidor SSH somente se houver necessidade de acesso remoto à máquina.

### 5.3. Serviços e portas de rede

**Objetivo:** identificar serviços em execução e portas em estado de escuta.

#### Serviços ativos

**Comando executado:**

```bash
systemctl --type=service --state=running --no-pager
```

**Resultado:** 29 unidades de serviço foram listadas como ativas e em execução.

Entre os serviços observados estavam:

```text
avahi-daemon.service
chrony.service
cron.service
cups-browsed.service
cups.service
NetworkManager.service
rsyslog.service
snapd.service
systemd-resolved.service
unattended-upgrades.service
vboxadd-service.service
```

A lista completa foi obtida durante a auditoria. O recorte acima destaca serviços relevantes para a análise de rede, impressão, registros e manutenção.

**Análise técnica:**

Serviços em execução são componentes normais de um sistema operacional. A presença de um serviço ativo não significa, por si só, que exista uma vulnerabilidade ou que ele esteja acessível pela rede.

A necessidade de cada serviço deve ser avaliada conforme a finalidade da máquina.

#### Portas em escuta

**Comando executado:**

```bash
sudo ss -lntup
```

**Resultado observado:**

```text
udp UNCONN 0 0 127.0.0.54:53       0.0.0.0:* users:(("systemd-resolve",pid=371,fd=18))
udp UNCONN 0 0 127.0.0.53%lo:53   0.0.0.0:* users:(("systemd-resolve",pid=371,fd=16))
udp UNCONN 0 0 127.0.0.1:323      0.0.0.0:* users:(("chronyd",pid=922,fd=4))
udp UNCONN 0 0 0.0.0.0:5353       0.0.0.0:* users:(("avahi-daemon",pid=730,fd=12))
udp UNCONN 0 0 [::1]:323           [::]:*    users:(("chronyd",pid=922,fd=5))
udp UNCONN 0 0 [::]:5353          [::]:*    users:(("avahi-daemon",pid=730,fd=13))
tcp LISTEN 0 4096 127.0.0.53%lo:53 0.0.0.0:* users:(("systemd-resolve",pid=371,fd=17))
tcp LISTEN 0 4096 127.0.0.54:53   0.0.0.0:* users:(("systemd-resolve",pid=371,fd=19))
tcp LISTEN 0 4096 127.0.0.1:631   0.0.0.0:* users:(("cupsd",pid=1073,fd=7))
tcp LISTEN 0 4096 [::1]:631       [::]:*    users:(("cupsd",pid=1073,fd=6))
```

**Observação:** o resultado acima é um recorte representativo da saída original, com as portas e serviços relevantes para a análise.

**Análise técnica:**

- **Porta 53:** utilizada pelo resolvedor DNS local, vinculado a endereços de loopback.
- **Porta UDP 323:** associada ao `chronyd`, escutando no endereço local IPv4 e no loopback IPv6.
- **Porta UDP 5353:** utilizada pelo Avahi para mDNS. Os endereços `0.0.0.0` e `[::]` indicam escuta em todas as interfaces IPv4 e IPv6 disponíveis, respectivamente.
- **Porta TCP 631:** associada ao CUPS, vinculada aos endereços de loopback IPv4 e IPv6.

A escuta em todas as interfaces merece atenção porque pode permitir que o serviço receba tráfego pelas interfaces de rede. A acessibilidade efetiva depende também das regras de filtragem, da configuração da rede e de outros controles.

#### Ambiente de rede

A máquina virtual utiliza rede NAT no VirtualBox.

Nesse modo, a possibilidade de acesso a serviços a partir de outros sistemas depende da configuração de rede, de eventuais regras de redirecionamento de portas e de outros controles aplicáveis.

**Avaliação:** os serviços DNS e de impressão observados estão vinculados a endereços locais. O Avahi merece avaliação por escutar em todas as interfaces disponíveis.

Não foi comprovada acessibilidade pela Internet nem uma vulnerabilidade específica a partir da consulta às portas.

**Recomendações:**
- Avaliar se o Avahi é necessário para a finalidade do ambiente.
- Restringir os serviços às interfaces necessárias quando apropriado.
- Evitar a exposição desnecessária de serviços.
- Repetir a verificação de portas após mudanças relevantes na configuração.

### 5.4. Firewall e recomendações de segurança

**Objetivo:** verificar o estado do UFW, ferramenta de gerenciamento de firewall disponível no Ubuntu.

**Comando executado:**

```bash
sudo ufw status verbose
```

**Resultado:**

```text
Estado: inativo
```

**Análise técnica:**

O resultado indica que o UFW estava inativo no momento da consulta. Portanto, suas regras não estavam sendo aplicadas por meio dessa ferramenta.

Esse resultado, isoladamente, não comprova que o sistema esteja sem qualquer filtragem de rede. Outras ferramentas ou camadas de rede podem aplicar regras adicionais.

A máquina virtual utiliza NAT no VirtualBox, o que também influencia sua acessibilidade a partir de outros sistemas.

**Avaliação:** o estado inativo do UFW é um ponto de atenção que deve ser considerado conforme as necessidades de conectividade e a configuração geral do ambiente.

**Recomendações:**
- Avaliar a ativação de um firewall compatível com as necessidades do laboratório.
- Definir regras de entrada conforme os serviços realmente necessários.
- Antes de aplicar mudanças, verificar se elas podem interromper a conectividade necessária.
- Repetir as verificações de portas e regras após eventuais alterações.

## 6. Conclusão da auditoria

A auditoria permitiu identificar a versão do sistema operacional e do kernel, examinar contas e privilégios, analisar permissões de arquivos importantes e observar serviços e portas de rede.

Os resultados demonstraram configurações compatíveis com o funcionamento esperado do Ubuntu, incluindo permissões restritivas nos arquivos examinados e serviços locais de resolução DNS e impressão.

Também foram identificados pontos que merecem atenção:

- Privilégios administrativos amplos para o usuário principal.
- Associação ao grupo `lxd` sem que o pacote tenha sido encontrado pelas consultas realizadas.
- Serviço Avahi escutando em todas as interfaces disponíveis.
- UFW inativo no momento da verificação.

Esses pontos não representam, automaticamente, vulnerabilidades confirmadas. Cada um precisa ser interpretado de acordo com a finalidade do sistema, a configuração e as evidências disponíveis.

O exercício demonstrou a importância de coletar evidências técnicas, interpretar resultados e propor recomendações proporcionais ao que foi observado.

**Limitação:** esta foi uma auditoria básica de configuração. Não foram realizadas todas as verificações possíveis, testes de exploração ou uma avaliação completa de vulnerabilidades. Portanto, os resultados não garantem que o sistema esteja livre de falhas de segurança.
