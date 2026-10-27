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

**Análise:** o sistema identificado é o Ubuntu 26.04.1 LTS, com codinome Resolute Raccoon. Essa informação ajuda a orientar as próximas verificações e a consulta à documentação específica da distribuição.

**Verificação 2 — Identificação do kernel**

Comando executado:

```bash
uname -r
```

Resultado:

```text
7.0.0-38-generic
```

**Análise:** o kernel em execução é o 7.0.0-38-generic. Essa informação será utilizada posteriormente para investigar o estado das atualizações e a manutenção de segurança do sistema.

**Limitação:** essas verificações identificam componentes do ambiente, mas não são suficientes para determinar o nível geral de segurança da máquina.

## 3. Objetivos

- Verificar usuários e privilégios administrativos.
- Analisar permissões de arquivos e diretórios importantes.
- Identificar serviços ativos e portas em escuta.
- Examinar configurações básicas do serviço SSH.
- Documentar as descobertas e propor melhorias quando necessário.

## 4. Ambiente de laboratório

- **Sistema operacional:** Ubuntu Linux
- **Virtualização:** Oracle VirtualBox
- **Tipo de ambiente:** máquina virtual de laboratório
- **Finalidade:** estudo e prática de fundamentos de segurança Linux

## 5. Metodologia

A auditoria será realizada por meio de comandos nativos do Linux e da análise de suas saídas. As verificações serão feitas no ambiente de laboratório, sem realizar testes contra sistemas de terceiros.

Os resultados serão classificados conforme o contexto, os riscos identificados e as limitações de cada verificação.

## 6. Resultados

Os resultados serão adicionados após a organização e análise das evidências coletadas durante a auditoria.

## 7. Aprendizados

Este projeto busca desenvolver conhecimentos práticos em administração Linux, permissões, gerenciamento de serviços, análise de portas de rede e configurações básicas de SSH.
