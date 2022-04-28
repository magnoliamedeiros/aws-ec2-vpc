# Exercício prático - Semana 1
Repositório destino a descrição do exercício prático sobre EC2 e VPC.
## Criar e acessar uma Amazon EC2 (Elastic Compute Cloud)
### Passos:
1. Escolher uma imagem de máquina da Amazon (AMI - Amazon Machine Image)   
    1. _Ubuntu Server 20.04 LTS_
2. Escolher o tipo de instância   
    1. _t2.micro (1 vCPUs, 1 GiB memory, EBS only - Free tier eligible)_
3. Configurar detalhes da instância   
    1. Nesse momento podemos deixar tudo padrão porque não temos nenhuma **VPC** e/ou **Subnet** criadas
4. Adicionar armazenamento   
    1. Nesse momento podemos deixar o padrão
5. Adicionar tags   
    1. Uma tag consiste em um par chave-valor que diferencia maiúsculas de minúsculas. Por exemplo, podemos definir uma tag com Key = Name e Value = Webserver   
    2. Para o exercício definir a seguinte tag: magnolia-ec2
6. Configurar o grupo de segurança   
    1. Um grupo de segurança é um conjunto de regras de firewall que controlam o tráfego da sua instância   
    2. Um ponto importante explicado pelo professor é que existe duas portas de entrada na AWS, a primeira é a ACL e a segunda porta é o Security Group   
    3. Estamos configurando a porta de entrada da nossa máquina   
    4. As regras com origem 0.0.0.0/0 permitem que todos os endereços IP acessem sua instância   
7. Revisar o lançamento da instância   
    1. Nesse momento ao clicar em iniciar (launch) precisaremos ter um par de chaves para atribuir a instância e assim concluir o processo   
    2. Para criar a chave SSH faça o seguinte comando no terminal:
    ```
    ssh-keygen [enter]
    Generating public/private rsa key pair.
    Enter file in which to save the key (/Users/seu.usuario/.ssh/id_rsa): [enter ou informe um caminho]
    Enter passphrase (empty for no passphrase): [pode deixar em branco]
    Enter same passphrase again: [pode deixar em branco]
    ```    
    3. Um par de chaves (key pair), consiste em uma chave privada e uma chave pública, é um conjunto de credenciais de segurança que você usa para provar sua identidade ao se conectar a uma instância   
    4. Para subir a chave pública para a sua conta da AWS vamos no serviço EC2 > Network & Security > Key Pairs, clicamos em Actions > Import key pair, adicionamos um Name para identificar a key, em key pair file clicamos em Browse para procurar a chave pública e ao carregar essa chave podemos então clicar no botão de Import key pair no final da página   
    5. Com a chave importada agora podemos escolher a mesma na página de Review Instance Launch e selecionar, marcar o check e finalizar clicando em Launch Instance   
8. Agora é possível conectar na instância via SSH:
    ```
    ssh -i id_rsa ubuntu@[ip-ou-dns-publico-do-ec2]   
    ```
## Criando uma VPC (Virtual Private Cloud)
O que é Amazon VPC? A Amazon Virtual Private Cloud (Amazon VPC) permite iniciar recursos da AWS em uma rede virtual definida por nós. Essa rede virtual se assemelha a uma rede tradicional, com os benefícios de usar a infraestrutura escalável da AWS.
### Passos:
1. Calcular o número de subredes desejada e para isso podemos utilizar o site: <https://www.site24x7.com/pt/tools/ipv4-sub-rede-calculadora.html>
2. Acessar o console da Amazon VPC em: <https://console.aws.amazon.com/vpc/>   
3. No painel de navegação, escolher **Your VPCs** > **Create VPC**
4. Em  **Resources to create**, escolher **VPC Only** por enquanto, dessa forma criará apenas a **VPC** sem subnets. A seguir definir os detalhres da **VPC**:
- **Name tag (opcional)**: my-vpc-grupo-8
- **IPv4 CIDR block**: _IPV4_
- **IPv4 CIDR**: _10.8.0.0/24 (aqui é a rede definida por você)_
- **IPv6 CIDR block**: _No IPv6 CIDR block_
- **Tenancy**: _Default_   

## Criando Subnets (Sub-redes) para a VPC
O que é uma sub-rede? é uma subdivisão lógica de uma rede.
### Passos:
Porque criamos subredes (subnets)? para que possamos ter datacenters diferentes em cada subrede e assim manter a alta disponibilidade.
1. No painel de navegação, escolher **Subnets** > **Create subnet**
2. Em **VPC ID** selecione a **VPC** criada anteriormente
3. Em **Subnet settings** > inserir um **Subnet name**: _my-subnet-grupo-8-1a (1a é para dizer que é a zona de disponibilidade 1a)_
4. Em **Availability Zone** > escolher a zona na qual a **subnet** residirá ou deixe a que a Amazon escolher: US East (N. Virginia) / us-east-1a
- IPv4 CIDR block: 10.8.0.64/26 (Na tabela em Notações de CIDR mostra que é /26)
- Clicar em Create subnet para finalizar.
- Repete a ação para criar todas as subnets necessárias, lembrando sempre de trocar a zona de disponibilidade.   

#### Em resumo temos o seguinte:
```
# VPC
my-vpc-grupo-8 (10.8.0.0/24)
# Subnets
my-subnet-grupo-8-1a (10.8.0.64/26)
my-subnet-grupo-8-1b (10.8.0.128/26)
my-subnet-grupo-8-1c (10.8.0.192/26)
```

### Route Tables e Internet Gateways
Route tables: São tabelas de rotas que podemos adicionar a um grupo de subnets algum tipo de recurso específico como exemplo: internet. Uma rede que não tem na sua Route Table atrelado uma Internet Gateway é uma rede privada.   

ACL (Access Control List): Uma ACL é uma lista sequencial de instruções de permissão ou negação que se aplicam a rede.   

Bastion Host: conecta o mundo externo a uma instância dentro de uma determinada VPC. Bastion é uma máquina pública que dará acesso a uma máquina privada.
#### Passos realizados:
1. Criar um Internet Gateways (para cada VPC) para
- Em Internet Gateways (dentro de VPC);
- Clica em Create Internet gateway;
- Internet gateway settings: em Name tag adicione um nome: my-internet-gateway-grupo-8;
- E clique no botão logo abaixo;
- Com IG criado precisamos "atachar" o mesmo a minha VPC;
- Para "attach" clica em Action > Attach to VPC e logo em seguida seleciona a VPC;
- Para finalizar clica no botão "Attach Internet gateway";
- Nesse momento a internet NÃO foi liberada apenas foi vinculada a sua VPC;
- Agora vamos para a Route Tables e procuramos a "route table" referente a minha VPC;
- Em Route tables procura a rtb que está vinculada a sua VPC e clica nela (rtb);
- Logo abaixo em Routes temos o botão de "Edit routes" e em "Add route";
- Quando adicione o 0.0.0.0/0 no Destination quer dizer que todos os meus ips (subnets) terão acesso a internet;
- Target: Internet Gateway > deverá aparecer o igw;
- Clica em Save changes;
- Agora temos o o internet gateway na route table;
- Próximo passo é clicar na aba "Subnet associations", perceba que de forma explicita (Explicit subnet associations) não temos nenhuma rede vincula a esse route table. Mas sem associações explicitas (Subnets without explicit associations) já temos a nossa subnet associada, automaticamente essas subnets já tem acesso a internet, mas isso é inseguro, o correto é colocar de forma explicita;
- Então em "Explicit subnet associations" clica no botão "Edit subnet associations" seleciona todas as suas subnets e clica em Save associations;
- Agora temos uma rede pública onde todos mundo pode se conectar via SSH.
