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


Guia Completo e Detalhado: Construindo a Rede Core do Zero
Este é um manual prático passo a passo desenhado para que qualquer pessoa, mesmo quem nunca abriu um terminal Linux ou configurou uma rede na vida, consiga replicar este projeto corporativo com total sucesso. Todo o processo será feito de forma manual e explicativa.

🗺️ Planejamento Espacial da Rede (Os IPs e Nomes Escolhidos)
Antes de digitar qualquer comando, precisamos desenhar o mapa da nossa rede. No mundo da TI, chamamos isso de topologia lógica.

Para este projeto, dividimos os elementos entre o que você pode customizar e o que deve ser mantido fixo para evitar falhas sistêmicas:

A Placa de Internet (WAN): Ela se chama enp0s3. Este nome é gerado automaticamente pelo Debian para identificar a placa que se conecta ao modem da sua casa. Ela recebe um IP dinâmico e automático, e nós NÃO devemos alterá-la.

A Placa da Rede Interna (LAN): Ela se chama enp0s8. É a placa de rede que vai distribuir a internet para dentro da nossa empresa de mentira.

O IP do Servidor Debian: Escolhemos o endereço fixo 192.168.10.1. Este será o coração de toda a rede. Se você quiser customizar, pode escolher outra faixa, como por exemplo 10.0.0.1 ou 172.16.0.1, mas lembre-se de trocar esse valor em todos os passos seguintes. Neste tutorial, usaremos o padrão 192.168.10.1.

O Escopo de Entrega (DHCP): Definimos que os computadores dos funcionários vão receber automaticamente IPs que vão do número 192.168.10.100 até o número 192.168.10.200. Isso nos dá margem para conectar até 101 máquinas na empresa.

O Nome da Empresa (Domínio Local): Escolhemos o nome de domínio interno empresa.local. Você pode mudar para o nome que quiser, como por exemplo meunome.lan ou corporativo.internal.

🔌 Passo 1: Preparando os Cabos de Rede Virtuais no VirtualBox
Antes de ligar o servidor Debian e a máquina de testes com Windows, precisamos garantir que os cabos de rede virtuais estão conectados no switch correto. Faça isso com as máquinas virtuais totalmente desligadas.

Abra a interface do seu VirtualBox no seu computador físico. Clique uma vez sobre a máquina virtual do seu Debian 12 e entre no menu Configurações, depois clique na aba Rede.

Na aba do Adaptador 1, certifique-se de que a caixinha "Habilitar Placa de Rede" está marcada e que o modo de conexão está selecionado como Placa em modo Bridge (ou em modo NAT). Essa placa será a nossa WAN, ou seja, o cabo de rede virtual que sai do servidor e vai direto para a internet do mundo real.

Agora, clique na aba do Adaptador 2. Marque a caixinha "Habilitar Placa de Rede". No campo "Conectado a:", mude a opção para Rede Interna. Logo abaixo, no campo Nome, apague o que estiver escrito e digite exatamente o termo: rede-corporativa. Esta placa será a nossa LAN, o switch virtual que interliga os computadores da empresa.

Por fim, vá na barra lateral do VirtualBox, clique na máquina virtual do seu Windows Client (a máquina do funcionário), entre em Configurações, vá em Rede e no Adaptador 1 mude a configuração para Rede Interna e coloque exatamente o mesmo nome: rede-corporativa. Desse modo, o Windows e o Debian estarão conectados no mesmo switch virtual.

🎯 Passo 2: Fixando o IP do Servidor Debian pelo Terminal
Ligue a sua máquina virtual do Debian 12. Faça o login na tela preta. Para fazermos alterações de sistema, precisamos virar o administrador supremo, conhecido como root. No terminal, digite o comando: su - e aperte a tecla Enter. O sistema vai pedir a sua senha de root. Digite-a (lembrando que os caracteres não aparecem na tela por segurança) e aperte Enter.

O primeiro arquivo que vamos editar gerencia os endereços de rede. Vamos usar o editor de texto integrado do Linux chamado nano. Digite o comando: nano /etc/network/interfaces e aperte Enter.

Você verá algumas linhas de texto na tela. Vá com as setas do seu teclado até o final do arquivo, pule uma linha e escreva exatamente o texto a seguir, respeitando as letras maiúsculas e minúsculas:

allow-hotplug enp0s8
iface enp0s8 inet static
address 192.168.10.1
netmask 255.255.255.0

Certifique-se de que as quatro linhas acima ficaram escritas de forma idêntica. Para salvar o arquivo usando o editor nano, pressione a combinação de teclas Ctrl e O juntas, e depois aperte a tecla Enter para confirmar. Para fechar o editor e voltar para a tela preta de comandos, pressione a combinação de teclas Ctrl e X.

Com o arquivo salvo, precisamos mandar o Linux aplicar essa configuração agora mesmo. Digite o comando: systemctl restart networking e aperte Enter. Para ter certeza absoluta de que a placa enp0s8 assumiu o IP correto, digite o comando: ip a e aperte Enter. Procure pelo bloco da placa enp0s8 e verifique se o IP 192.168.10.1 está aparecendo lá.

🛡️ Passo 3: Ativando o Roteamento de Internet e o Firewall NAT
Por padrão, o Linux vem com uma trava de segurança que o impede de passar internet de uma placa de rede para a outra. Vamos quebrar essa trava transformando o Debian em um roteador de borda.

Ainda no terminal como root, precisamos editar o arquivo de parâmetros do Kernel do sistema. Digite o comando: nano /etc/sysctl.conf e aperte Enter.

Use as setas do teclado para descer o arquivo até encontrar uma linha que está escrita exatamente assim: #net.ipv4.ip_forward=1. Repare que existe um símbolo de hashtag no começo dela. Esse símbolo faz a linha virar um mero comentário ignorado pelo sistema. Use a tecla Backspace para apagar apenas o símbolo de hashtag, deixando a linha começando direto com o termo: net.ipv4.ip_forward=1. Salve o arquivo apertando Ctrl e O, dê Enter, e saia apertando Ctrl e X.

Para ativar essa função imediatamente sem precisar reiniciar o computador, digite o comando: sysctl -p e aperte Enter.

Agora que o Debian sabe rotear, precisamos criar o Firewall que traduz os IPs da rede interna para que eles possam navegar na internet externa. Essa tecnologia se chama NAT (Mascaramento de Rede). No Debian 12, usamos o sistema nftables.

Primeiro, instale o sistema digitando o comando: apt install nftables -y e aperte Enter. Aguarde a barra de carregamento chegar a 100%.

Agora, vamos aplicar as três regras fundamentais do nosso Firewall sequencialmente. Digite o primeiro comando: nft add table ip nat e aperte Enter.

Depois, digite o segundo comando exatamente assim, prestando atenção nos espaços e nas chaves: nft add chain ip nat postrouting { type nat hook postrouting priority 100 ; } e aperte Enter.

Por fim, digite o terceiro comando que faz a mágica do mascaramento apontando para a nossa placa de internet externa: nft add rule ip nat postrouting oifname "enp0s3" masquerade e aperte Enter.

Para garantir que o firewall vai ligar sozinho caso o servidor seja reiniciado por falta de energia, digite o comando: systemctl enable nftables e aperte Enter.

🛜 Passo 4: Configurando o Distribuidor Automático de IPs (Servidor DHCP)
Com o roteador pronto, precisamos fazer o Debian entregar as configurações de rede para o Windows automaticamente, evitando que o usuário precise configurar isso na mão.

No terminal, instale o pacote do servidor DHCP digitando o comando: apt install isc-dhcp-server -y e aperte Enter. Assim que a instalação terminar, o sistema vai tentar iniciar o serviço e vai dar uma mensagem de erro vermelha na tela. Não se assuste! Isso acontece porque o arquivo de configuração ainda está em branco.

Primeiro, precisamos dizer ao DHCP em qual placa de rede ele deve trabalhar. Digite o comando: nano /etc/default/isc-dhcp-server e aperte Enter.

Logo nas primeiras linhas, procure pelo parâmetro escrito INTERFACESv4="". Coloque o nome da nossa placa interna dentro das aspas duplas, fazendo a linha ficar exatamente assim: INTERFACESv4="enp0s8". Salve o arquivo com Ctrl e O, dê Enter, e saia com Ctrl e X.

Agora vamos configurar o escopo de rede da nossa empresa. Digite o comando para abrir o arquivo de configuração principal: nano /etc/dhcp/dhcpd.conf e aperte Enter. Use a seta para baixo do teclado para ir até o final do arquivo. Apague as últimas linhas de exemplo e digite a estrutura de texto corrida a seguir, respeitando rigorosamente cada espaço, abertura de chaves e os pontos e vírgulas no final de cada linha:

subnet 192.168.10.0 netmask 255.255.255.0 {
range 192.168.10.100 192.168.10.200;
option routers 192.168.10.1;
option domain-name-servers 192.168.10.1;
option domain-name "empresa.local";
default-lease-time 600;
max-lease-time 7200;
}

Revise linha por linha. Note que as opções range, routers, domain-name-servers, domain-name, default-lease-time e max-lease-time possuem um caractere de ponto e vírgula no final. Se esquecer um único ponto e vírgula, o servidor não liga. Salve o arquivo pressionando Ctrl e O, dê Enter, e saia com Ctrl e X.

Com tudo pronto, ligue os motores do serviço DHCP digitando o comando: systemctl restart isc-dhcp-server e aperte Enter. Para garantir a inicialização automática pós-boot, digite: systemctl enable isc-dhcp-server e aperte Enter.

🔍 Passo 5: Configurando o Servidor de Resolução de Nomes (Servidor DNS BIND9)
O último pilar da nossa rede Core é o DNS. Ele vai receber os pedidos de sites dos clientes (como google.com), vai consultar a internet e vai guardar uma cópia em cache no servidor, acelerando a velocidade de navegação de toda a empresa.

Instale o pacote do DNS digitando o comando: apt install bind9 -y e aperte Enter.

O arquivo principal de configuração do DNS se chama named.conf.options. Vamos abri-lo digitando o comando: nano /etc/bind/named.conf.options e aperte Enter.

Apague completamente todo o conteúdo que veio de fábrica dentro desse arquivo e escreva a estrutura limpa descrita a seguir, prestando uma atenção redobrada aos pontos e vírgulas:

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

Repare detalhadamente na estrutura: a palavra options abre uma chave. Dentro dela, temos o diretório e o bloco forwarders (encaminhadores), que aponta para os IPs do Google e da Cloudflare. Note que há um ponto e vírgula depois do IP 8.8.8.8, um depois do 1.1.1.1, um depois do fechamento da chave do forwarders, um depois do termo any, um depois do termo auto, um depois do segundo any, e por fim, um ponto e vírgula após o fechamento da chave mestre do options.

Salve o arquivo pressionando Ctrl e O, confirme com Enter, e saia com Ctrl e X.

Antes de tentarmos iniciar o serviço, vamos usar o validador oficial do Bind9 para ter certeza de que não erramos nenhum caractere. Digite o comando: named-checkconf e aperte Enter. Se o terminal simplesmente pular para a linha de baixo sem mostrar nenhuma mensagem, significa que o seu arquivo está perfeito! Se mostrar algum erro, reabra o arquivo e procure o ponto e vírgula que ficou faltando.

Com a sintaxe validada, inicie o serviço usando o nome real do daemon de segundo plano do sistema. Digite o comando: systemctl restart named e aperte Enter. Para finalizar a persistência do sistema, digite: systemctl enable named e aperte Enter.

🖥️ Passo 6: A Prova de Fogo (Validando tudo no Cliente Windows)
O seu servidor Debian 12 está completamente configurado e operando como um roteador de borda corporativo de nível profissional. Chegou a hora de validar se tudo funciona do ponto de vista do usuário final.

Ligue a sua máquina virtual com o Windows Client. Faça o login no Windows. No teclado, pressione a combinação de teclas Windows e R juntas para abrir a caixa do Executar. Digite as letras cmd e aperte Enter. A tela preta do Prompt de Comando do Windows vai se abrir.

Primeiro, vamos limpar qualquer configuração de rede antiga que o Windows tenha guardado na memória. Digite o comando: ipconfig /release e aperte Enter. Sua placa de rede vai ficar temporariamente desconectada.

Agora, vamos mandar o Windows gritar na rede interna procurando pelo nosso servidor Debian para solicitar os novos dados de conexão. Digite o comando: ipconfig /renew e aperte Enter. Aguarde cerca de 5 segundos.

Para ver se a mágica aconteceu, digite o comando de checagem: ipconfig e aperte Enter.

Olhe atentamente para os dados que apareceram na tela do seu prompt. O Endereço IPv4 do seu Windows deve estar mostrando um número entre 192.168.10.100 e 192.168.10.200. A Máscara de Sub-rede deve ser 255.255.255.0. E o mais importante: o Gateway Padrão deve estar carimbado com o IP do nosso servidor Debian, o número 192.168.10.1.

Para coroar o sucesso do projeto e testar o Firewall NAT e o DNS trabalhando juntos, vamos disparar um teste de comunicação externa. Digite o comando: ping google.com e aperte Enter.

Se o prompt do Windows começar a mostrar linhas escritas "Resposta de..." acompanhadas do tempo em milissegundos, parabéns! O Windows acabou de solicitar a tradução do nome do site para o servidor DNS do Debian, recebeu a resposta, passou pelo filtro do Firewall e navegou com sucesso absoluto pela internet através do seu roteador Linux feito à mão.
