# infraestrutura-corporativa-linux
Projeto prático de construção de uma infraestrutura corporativa do zero utilizando Debian 12 (Roteamento, Firewall, DHCP, DNS e AD)

# 🏢 Infraestrutura Corporativa Zero-Cost com Debian 12

![Debian](https://img.shields.io/badge/Debian-A81D33?style=for-the-badge&logo=debian&logoColor=white)
![Linux](https://img.shields.io/badge/Linux-FCC624?style=for-the-badge&logo=linux&logoColor=black)
![Redes](https://img.shields.io/badge/Redes-00599C?style=for-the-badge&logo=cisco&logoColor=white)

## 📌 Sobre o Projeto
Projeto prático focado na construção de uma infraestrutura de rede corporativa completa utilizando ferramentas Open Source. O objetivo é simular o ambiente "Core" de uma pequena/média empresa, provendo roteamento, segurança, distribuição de IPs e resolução de nomes totalmente baseados em Linux.

## 🚀 Fase 1: Core Network (Concluída)
Nesta primeira fase, o servidor Debian foi configurado para atuar como o coração da rede, assumindo as funções de Roteador de Borda, Firewall e Servidor de Serviços Essenciais para os clientes (Windows/Linux) da rede interna.

### Tecnologias e Serviços:
* **Roteamento e NAT:** `nftables` (Mascaramento de rede)
* **Servidor DHCP:** `isc-dhcp-server` (Distribuição automatizada de escopo IP)
* **Servidor DNS:** `bind9` / `named` (Resolução de nomes e Forwarder)

### 📸 Evidências e Validação

**1. Cliente recebendo IP corporativo (DHCP)**
![IP Dinâmico DHCP](./assets/DHCP.png)

**2. Navegação via Firewall/NAT (DNS e Roteamento)**
![Ping Externo](./assets/ping.png)

---
*Status: Aguardando início da Fase 2 (Servidor de Arquivos e Domínio com Samba).*


# 📖 Guia Passo a Passo: Construindo a Rede Core do Zero

Este é um tutorial detalhado, projetado para que **qualquer pessoa (mesmo sem experiência prévia em Linux ou Redes)** consiga replicar este laboratório corporativo do absoluto zero. 

---

## 🛠️ Cenário e Tabela de Configurações

Antes de começar, entenda as regras do nosso ambiente. Você pode alterar os valores da coluna **"Seu Valor (Customizável)"**, mas lembre-se de substituir a sua escolha em todas as etapas do tutorial!

| Parâmetro de Rede | Valor Padrão do Projeto | Tipo | O que significa? |
| :--- | :--- | :--- | :--- |
| **Interface WAN** | `enp0s3` | 🚨 Fixo (Verifique o seu) | A placa que recebe internet da sua casa. |
| **Interface LAN** | `enp0s8` | 🚨 Fixo (Verifique o seu) | A placa que vai distribuir internet para a rede interna. |
| **IP do Servidor** | `192.168.10.1` | 🟢 Customizável | O endereço fixo do seu servidor Debian na rede interna. |
| **Máscara de Rede** | `/24` ou `255.255.255.0` | 🟢 Customizável | Define que a sua rede pode ter até 254 computadores. |
| **Nome da Rede Virtual** | `rede-corporativa` | 🟢 Customizável | O nome do "cabo virtual" dentro do VirtualBox. |
| **Escopo DHCP** | `192.168.10.100` a `192.168.10.200` | 🟢 Customizável | Faixa de IPs que os clientes vão receber automaticamente. |
| **Domínio Local** | `empresa.local` | 🟢 Customizável | O sufixo de nome da sua rede interna. |

---

## 📖 Guia Passo a Passo: Construindo a Rede Core do Zero

Este é um tutorial detalhado, projetado para que **qualquer pessoa (mesmo sem experiência prévia em Linux ou Redes)** consiga replicar este laboratório corporativo do absoluto zero.

---

## 🛠️ Cenário e Tabela de Configurações

Antes de começar, entenda as regras do nosso ambiente. Você pode alterar os valores da coluna **"Seu Valor (Customizável)"**, mas lembre-se de substituir a sua escolha em todas as etapas do tutorial!

| Parâmetro de Rede | Valor Padrão do Projeto | Tipo | O que significa? |
| --- | --- | --- | --- |
| **Interface WAN** | `enp0s3` | 🚨 Fixo (Verifique o seu) | A placa que recebe internet da sua casa. |
| **Interface LAN** | `enp0s8` | 🚨 Fixo (Verifique o seu) | A placa que vai distribuir internet para a rede interna. |
| **IP do Servidor** | `192.168.10.1` | 🟢 Customizável | O endereço fixo do seu servidor Debian na rede interna. |
| **Máscara de Rede** | `/24` ou `255.255.255.0` | 🟢 Customizável | Define que a sua rede pode ter até 254 computadores. |
| **Nome da Rede Virtual** | `rede-corporativa` | 🟢 Customizável | O nome do "cabo virtual" dentro do VirtualBox. |
| **Escopo DHCP** | `192.168.10.100` a `192.168.10.200` | 🟢 Customizável | Faixa de IPs que os clientes vão receber automaticamente. |
| **Domínio Local** | `empresa.local` | 🟢 Customizável | O sufixo de nome da sua rede interna. |

---

## 🏗️ Passo 1: Configuração das Placas no VirtualBox (Antes de ligar as máquinas)

Para que o servidor converse com a internet e com a rede interna ao mesmo tempo, precisamos de duas placas de rede virtuais.

1. Clique na sua máquina **Debian** no VirtualBox e vá em **Configurações > Rede**.
2. **Adaptador 1:** Deixe habilitado em modo **Placa em modo Bridge** (ou NAT). *Isso garante a internet do servidor (WAN).*
3. **Adaptador 2:** Marque "Habilitar Placa de Rede", mude para **Rede Interna** e no campo *Nome* digite:
    * 🟢 **[VOCÊ ESCOLHE]:** `rede-corporativa`
4. Vá na sua máquina **Windows Client**, entre em **Configurações > Rede > Adaptador 1** e configure **exatamente igual** ao Adaptador 2 do Debian: **Rede Interna** com o nome `rede-corporativa`.

---

## 🌐 Passo 2: Fixando o IP do Servidor Debian

Por padrão, servidores não podem ter IPs dinâmicos que mudam sozinhos. Vamos fixar o IP da nossa rede interna.

No terminal do Debian, faça login como administrador (`root`):

```bash
su -
```

Abra o arquivo de configuração de rede:

```bash
nano /etc/network/interfaces
```

Deixe o arquivo exatamente assim (adicione as linhas da interface local no final):

```plaintext
# Placa de Internet (WAN) - Recebe IP automático
allow-hotplug enp0s3
iface enp0s3 inet dhcp

# Placa da Rede Interna (LAN) - IP Fixo do Servidor
allow-hotplug enp0s8
iface enp0s8 inet static
address 192.168.10.1
netmask 255.255.255.0
```

> ⌨️ **[DICA DE TERMINAL]:** No nano, para salvar aperte `Ctrl + O`, dê `Enter` e para sair aperte `Ctrl + X`.

Reinicie o serviço de rede para aplicar o IP novo:

```bash
systemctl restart networking
```

---

## 🛡️ Passo 3: Ativando o Roteamento e o Firewall NAT (nftables)

Agora vamos transformar o Debian em um roteador, fazendo com que ele compartilhe a internet da placa externa (`enp0s3`) com a placa interna (`enp0s8`).

Primeiro, diga ao sistema operacional que ele tem permissão para encaminhar pacotes. Edite o arquivo de parâmetros do Kernel:

```bash
nano /etc/sysctl.conf
```

Procure a linha `#net.ipv4.ip_forward=1` e apague o símbolo `#` para descomentá-la, deixando assim:

```plaintext
net.ipv4.ip_forward=1
```

Ative a configuração imediatamente:

```bash
sysctl -p
```

Instale o sistema de Firewall:

```bash
apt install nftables -y
```

Crie as regras de mascaramento (NAT) para traduzir os IPs da rede interna:

```bash
nft add table ip nat
nft add chain ip nat postrouting { type nat hook postrouting priority 100 \; }
nft add rule ip nat postrouting oifname "enp0s3" masquerade
```

Garanta que o firewall inicie com o sistema:

```bash
systemctl enable nftables
```

---

## 🛜 Passo 4: Configurando o Servidor DHCP (isc-dhcp-server)

O DHCP vai entregar as configurações de rede para qualquer computador que plugar na nossa rede interna de forma automática.

Instale o serviço:

```bash
apt install isc-dhcp-server -y
```

Diga ao DHCP em qual placa de rede ele deve escutar. Abra o arquivo de configuração:

```bash
nano /etc/default/isc-dhcp-server
```

Procure pela linha `INTERFACESv4=""` e coloque a sua placa interna dentro das aspas:

```plaintext
INTERFACESv4="enp0s8"
```

Agora, vamos configurar o escopo (a faixa de IPs que ele vai distribuir). Abra o arquivo principal:

```bash
nano /etc/dhcp/dhcpd.conf
```

Vá até o final do arquivo, apague tudo o que estiver nas últimas linhas desnecessárias e cole esta estrutura limpa:

```plaintext
subnet 192.168.10.0 netmask 255.255.255.0 {
    range 192.168.10.100 192.168.10.200;
    option routers 192.168.10.1;
    option domain-name-servers 192.168.10.1;
    option domain-name "empresa.local";
    default-lease-time 600;
    max-lease-time 7200;
}
```

Salve, saia do arquivo e ligue o serviço:

```bash
systemctl restart isc-dhcp-server
systemctl enable isc-dhcp-server
```

---

## 🔍 Passo 5: Configurando o Servidor DNS (Bind9 / named)

O DNS vai traduzir nomes de sites em IPs. Ele vai guardar um cache local para deixar a navegação interna muito mais rápida.

Instale o Bind9:

```bash
apt install bind9 -y
```

Abra o arquivo de opções do servidor DNS:

```bash
nano /etc/bind/named.conf.options
```

Modifique o conteúdo para que ele aceite requisições da sua rede e encaminhe o que não souber para os DNS públicos da Google (`8.8.8.8`) e Cloudflare (`1.1.1.1`):

```plaintext
options {
    directory "/var/cache/bind";

    forwarders {
            8.8.8.8;
            1.1.1.1;
    };

    allow-query { any; };
    dnssec-validation auto;
    listen-on-v6 { any; };
};
```

> 🚨 **[SUPER TRUQUE DE DEBUG]:** Antes de reiniciar, verifique se você não esqueceu nenhum ponto e vírgula `;` rodando o comando de validação:

```bash
named-checkconf
```
*(Se o comando não retornar absolutamente nada, parabéns! Sua sintaxe está perfeita).*

Ative o serviço usando o nome real do processo do sistema (`named`):

```bash
systemctl restart named
systemctl enable named
```

---

## 🖥️ Passo 6: A Prova de Fogo (Validando no Cliente Windows)

Agora vamos testar se o seu trabalho deu certo. Ligue a sua máquina virtual Windows Client.

No Windows, abra o menu iniciar, digite `cmd` e abra o Prompt de Comando.

Force a placa de rede a liberar qualquer IP antigo:

```dos
ipconfig /release
```

Peça um IP novo para o nosso servidor Debian:

```dos
ipconfig /renew
```

Veja o resultado digitando:

```dos
ipconfig
```

🎯 **Resultado esperado:** O IP do Windows deve ser algo entre `192.168.10.100` e `192.168.10.200`. O Gateway Padrão e o Servidor DNS devem apontar para o IP do seu Debian: `192.168.10.1`.

Faça o teste final de internet passando pelo seu roteador Linux:

```dos
ping google.com
```

Se os pacotes responderem, a sua infraestrutura de redes corporativa está completa, functional e pronta para produção! 🚀
