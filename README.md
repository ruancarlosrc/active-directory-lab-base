# active-directory-lab-base
# 🖥️ Active Directory Home Lab — Josh Madakor

Laboratório prático de Active Directory baseado no tutorial do [Josh Madakor](https://www.youtube.com/watch?v=MHsI8hJmggI), com foco na construção de um ambiente corporativo simulado do zero usando VirtualBox.

---

## 🖥️ Ambiente

| Componente | Detalhe |
|---|---|
| Hypervisor | Oracle VirtualBox |
| Domain Controller | Windows Server 2022 |
| Cliente | Windows 10 (Client1) |
| Domínio | mydomain.com (FQDN: DC.mydomain.com) |
| Usuários criados | ~1000 (via PowerShell script) |

---

## 🌐 Topologia de Rede

```
INTERNET
    │
    │ (DHCP via roteador doméstico)
    │
[NIC _INTERNET_]
    │
   DC (Windows Server 2022)
    │
[NIC X_INTERNAL_X]  IP: 172.16.0.1 | Mask: 255.255.255.0 | DNS: 127.0.0.1
    │
    │ (DHCP via DC — Range: 172.16.0.100-200)
    │
[NIC Internal]
    │
 Client1 (Windows 10)
```

O DC atua como roteador entre a rede interna e a internet via **RAS/NAT**, e como servidor **DHCP** para os clientes da rede interna.

---

## ✅ O que foi configurado

### 1. Domain Controller
- Instalação do Windows Server 2022 no VirtualBox
- Identificação e renomeação dos adaptadores de rede (`_INTERNET_` e `X_INTERNAL_X`)
- Configuração de IP estático na interface interna (`172.16.0.1`)
- Instalação e configuração do **AD DS** (Active Directory Domain Services)
- Promoção do servidor a Domain Controller com domínio `mydomain.com`

### 2. RAS / NAT
- Instalação do papel **Remote Access** com roteamento
- Configuração do **NAT** para permitir que os clientes da rede interna acessem a internet através do DC

### 3. DHCP
- Instalação do papel **DHCP Server**
- Criação de escopo IPv4:
  - Range: `172.16.0.100 – 172.16.0.200`
  - Máscara: `255.255.255.0`
  - Gateway: `172.16.0.1`
  - DNS: `172.16.0.1`

### 4. Criação de ~1000 usuários via PowerShell
- Script PowerShell que lê uma lista de nomes (`names.txt`) e cria automaticamente contas de usuário na OU `_USERS`
- Padrão de username: primeira letra do nome + sobrenome (ex: `jsmith`)
- Todos os usuários criados com senha padrão `Password1`

```powershell
# Trecho principal do script
$PASSWORD_FOR_USERS = "Password1"
$USER_FIRST_LAST_LIST = Get-Content .\names.txt

$password = ConvertTo-SecureString $PASSWORD_FOR_USERS -AsPlainText -Force
New-ADOrganizationalUnit -Name _USERS -ProtectedFromAccidentalDeletion $false

foreach ($n in $USER_FIRST_LAST_LIST) {
    $first = $n.Split(" ")[0].ToLower()
    $last = $n.Split(" ")[1].ToLower()
    $username = "$($first.Substring(0,1))$($last)".ToLower()

    New-ADUser -AccountPassword $password `
               -GivenName $first `
               -Surname $last `
               -DisplayName $username `
               -Name $username `
               -EmployeeID $username `
               -PasswordNeverExpires $true `
               -Path "ou=_USERS,$(([ADSI]`"").distinguishedName)" `
               -Enabled $true
}
```

### 5. Cliente Windows 10
- Instalação do Windows 10 no VirtualBox com adaptador em **Internal Network**
- IP obtido automaticamente via DHCP do DC
- Ingresso no domínio `mydomain.com`
- Login com usuário criado pelo script PowerShell

---

## 📸 Evidências

| Etapa | Print |
|---|---|
| Diagrama do esquema de rede | ![Diagrama](evidencias/DESENHO_DO_ESQUEMA_DE_REDE_E_CONFIGURAÇÃO.png) |
| Adaptadores de rede identificados | ![Adaptadores](evidencias/ADAPTADORES_DE_REDE_IDENTIFICADOS.png) |
| DC configurado no Server Manager | ![DC](evidencias/DC_.png) |
| Routing and Remote Access (NAT) | ![RAS](evidencias/ROUTING_E_REMOTE_ACCESS.png) |
| DHCP configurado com escopo | ![DHCP](evidencias/DHCP_CONFIGURADO.png) |
| ~1000 usuários criados via PowerShell | ![Usuários](evidencias/MIL_USUARIOS_CRIADOS_COM_SCRIP_POWERSHELL.png) |
| Cliente Windows 10 ingressado no domínio | ![Cliente](evidencias/CLIENTE_WINDOWS10_ADICIONADO.png) |

---

## 📚 Referências

- [Josh Madakor — Active Directory Lab (YouTube)](https://www.youtube.com/watch?v=MHsI8hJmggI)
- [Microsoft Docs — Active Directory Domain Services](https://learn.microsoft.com/en-us/windows-server/identity/ad-ds/get-started/virtual-dc/active-directory-domain-services-overview)
- [Microsoft Docs — DHCP Server](https://learn.microsoft.com/en-us/windows-server/networking/technologies/dhcp/dhcp-top)

---

> Este laboratório é a base para o projeto [Active Directory — Blue Team Edition](https://github.com/ruancarlosrc/active-directory-lab), onde o ambiente foi estendido com GPOs defensivos, auditoria de eventos e simulação de Kerberoasting.
