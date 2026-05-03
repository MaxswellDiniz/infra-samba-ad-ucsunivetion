# 🚀 Infraestrutura de Diretório e Serviços com Samba AD (UCS)

![Status](https://img.shields.io/badge/Status-Conclu%C3%ADdo-success)
![Versão](https://img.shields.io/badge/Vers%C3%A3o-1.0-blue)
![Licença](https://img.shields.io/badge/Licen%C3%A7a-MIT-green)

## 📋 Índice
1. [Descrição do Projeto](#-descrição-do-projeto)
2. [Tecnologias Utilizadas](#-tecnologias-utilizadas)
3. [Funcionalidades Implementadas](#-funcionalidades-implementadas)
4. [Estrutura do Repositório](#-estrutura-do-repositório)
5. [Guia de Configuração (Passo a Passo)](#-guia-de-configuração-passo-a-passo)
6. [Como Contribuir](#-como-contribuir)

---

## 📖 Descrição do Projeto
Este projeto documenta a implantação, configuração e hardening de um servidor **Univention Corporate Server (UCS)** atuando como **Controlador de Domínio Active Directory (Samba 4)**. O laboratório integra nativamente estações Windows 10, fornecendo autenticação centralizada, servidor de impressão (CUPS) com distribuição automatizada de drivers (Point 'n' Print) e servidor de arquivos com mapeamento via GPO e Scripts de Logon.

---

## 💻 Tecnologias Utilizadas
*   **Servidor Base:** Univention Corporate Server (UCS) 5.x (Baseado em Debian GNU/Linux)
*   **Serviço de Diretório:** Samba 4 (Active Directory Domain Controller)
*   **Sistema Cliente:** Windows 10 Pro / Enterprise
*   **Serviço de Impressão:** CUPS integrado ao Samba
*   **Gerenciamento Remoto:** RSAT (Remote Server Administration Tools) Microsoft
*   **Virtualização:** Hyper-V (Rede via Switch Interno/Privado)

---

## 🔨 Funcionalidades Implementadas
- [x] Provisionamento de Domínio Active Directory (LDAP/Kerberos/DNS).
- [x] Ingresso seguro de estações Windows 10 com sincronização de NTP (`w32tm`).
- [x] Gestão remota de infraestrutura pelo Windows 10 utilizando o RSAT (`dsa.msc`, `gpmc.msc`, `printmanagement.msc`).
- [x] Servidor de impressão Linux (CUPS) hospedando drivers do Windows de forma nativa.
- [x] Criação de pastas de rede departamentais públicas e pastas exclusivas de usuários (`homes`).
- [x] Mapeamento automático de unidades de rede via Scripts (`NETLOGON`) e Políticas de Grupo (GPO).

---

## 📂 Estrutura do Repositório
*   `/scripts/linux/setup_cups.sh` - Automação da instalação do CUPS e criação da impressora no UCS.
*   `/scripts/windows/mapear.bat` - Script de mapeamento de disco enviado via diretório `NETLOGON`.
*   `/docs/` - Documentações de apoio e referências técnicas.

---

## 🚀 Guia de Configuração (Passo a Passo)

### Fase 1: Preparação do Controlador de Domínio (UCS)
1. Instale o UCS e realize o acesso web inicial no Univention Management Console (UMC) via usuário `Administrator`.
2. No **App Center**, instale a aplicação **Active Directory-compatible Domain Controller**. Isso configurará a base do Samba 4 e os apontamentos de DNS (SRV Records).
3. No painel, vá em **Usuários**, crie uma conta administrativa com padrão `nome.sobrenome` (ex: `maxswell.diniz`) e adicione-a ao grupo **Domain Admins**.

### Fase 2: Configuração da Rede e Ingresso do Windows 10
1. Certifique-se de que a VM do servidor e o Windows 10 estão na mesma rede privada.
2. No Windows 10, configure o IP fixo e aponte o **Servidor DNS Preferencial obrigatoriamente para o IP do UCS**. Desative o IPv6 na placa de rede.
3. Sincronize o relógio executando o comando no CMD como Administrador: `w32tm /resync`.
4. Ingresse a máquina no Domínio através das Configurações Avançadas do Sistema no Windows.

### Fase 3: Administração Remota (RSAT)
Em vez de depender apenas da interface web, transforme a estação Windows 10 no seu painel de controle mestre:
1. Logado no Windows com a conta Domain Admin, abra o **PowerShell como Administrador**.
2. Force a instalação do RSAT:
   ```powershell
   Add-WindowsCapability -Online -Name RsaT.ActiveDirectory.DS-LDS.Tools~~~~0.0.1.0
   Add-WindowsCapability -Online -Name RsaT.GroupPolicy.Management.Tools~~~~0.0.1.0
