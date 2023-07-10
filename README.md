# Trabalho_2AWS-Docker
Repositório para a atividade do programa de bolsas da Compass UOL - AWS + Docker

## Requisitos 
1. instalação e configuração do DOCKER ou CONTAINERD no
host EC2;
Ponto adicional para o trabalho utilizar a instalação via script de
Start Instance (user_data.sh)
2. Efetuar Deploy de uma aplicação Wordpress com:
container de aplicação
RDS database Mysql
3. configuração da utilização do serviço EFS AWS para estáticos
do container de aplicação Wordpress
4. configuração do serviço de Load Balancer AWS para a aplicação
Wordpress
#### Pontos de atenção:
- não utilizar ip público para saída do serviços WP (Evitem
publicar o serviço WP via IP Público)
- sugestão para o tráfego de internet sair pelo LB (Load
Balancer)
- pastas públicas e estáticos do wordpress sugestão de utilizar
o EFS (Elastic File System)
- Fica a critério de cada integrante (ou dupla) usar Dockerfile
ou Dockercompose;
- Necessário demonstrar a aplicação wordpress funcionando
(tela de login)
- Aplicação Wordpress precisa estar rodando na porta 80 ou
8080;
- Utilizar repositório git para versionamento;
- Criar documentação.
#### Arquitetura:
![image](https://github.com/JuFick/Trabalho_2AWS-Docker/assets/132408071/09b4a3a4-0b85-4817-a126-e8710c41b16b)

---
## Passos do Desenvolvimento
### Network
- Vá até o serviço de VPC na sua console AWS;
- No menu esquerdo, clique em **Suas VPCs** e, em seguida, clique em **Criar VPC**;
- Crie sua VPC.
- Logo após, acesse o **Gateways da Internet**, e crie seu Internet Gateway. Não se esqueça de associá-lo à VPC criada anteriormente;
- Depois, acesse as **Sub-redes**. Crie duas sub-redes, uma pública e uma privada, elas precisam estar na mesma zona de disponibilidade. Repita o processo para a segunda zona de diponibilidade;
- Agora, crie duas **Tabelas de Rotas**, uma para as subnetes públicas e uma para as privadas;
- Para as subnetes públicas, edite a tabela de rotas adicionando uma regra de destino para *0.0.0.0/0* e alvo/target para o *Gateway de internet criado*;
- No menu lateral esquerdo, clique em **NAT Gateway**;
- Crie um Nat Gateway e um **Elastic IP** para a subnet pública;
- Vá para tabelas de rotas novamente e adicione uma nova regra às subnetes privadas. Nela, configure o destino como *0.0.0.0/0* e alvo/target para o *NAT Gateway* criado.
### Configuração dos Security Groups
- Entre no serviço de EC2 no console da AWS e no menu lateral esquerdo clique em **Grupos de Segurança**;
- Crie um grupo de segurança na nova VPC para as instâncias a serem criadas com as permissões a seguir: (com origem da port 22 para security group do bastion e port 80 para do load balancer)
  ![Sg-instancias](https://github.com/JuFick/Trabalho_2AWS-Docker/assets/132408071/bcaa740a-23d3-4a3f-953a-53309ea9542a)

- Faça o mesmo processo para o security group do EFS, no entanto com as seguintes permissões:  (origem para o grupo de segurança das instâncias e do bastion host)
  ![Sg-efs](https://github.com/JuFick/Trabalho_2AWS-Docker/assets/132408071/413b9bca-ce50-43ed-809d-8a069734207e)

- Crie um para o Load Balancer:
  ![sg-loadbalancer](https://github.com/JuFick/Trabalho_2AWS-Docker/assets/132408071/23152065-ba15-4320-b5d8-b3e1fbfd3152)

- Crie outo para o Banco de dados:(com origens no security group da instancia e do bastion host)
 ![sg-bancodados](https://github.com/JuFick/Trabalho_2AWS-Docker/assets/132408071/c185bf21-eebd-42fd-aae5-53fce95e3e14)

- Crie mais um para o Bastion Host: (com origem 0.0.0.0/0)
  ![sg-bastion](https://github.com/JuFick/Trabalho_2AWS-Docker/assets/132408071/430f89c2-2edc-42da-b475-027a24b42672)
### EFS (Elastic File Sistem)
- Vá até o serviço de EFS na aws e clique em **Criar Sistema de Arquivos**;
- Selecione a VPC criada anteriormente e clique em *Personalizar e Próximo*;
- Defina as duas zonas de disponibilidade e mude o security group para o previamente criado;
- Selecone, novamente, *Próximo* para as duas etapas seguintes e crie seu Elastic File Sistem.
### RDS MySQL (Relational Database Service)
- Abra o console AWS e selecione o serviço de RDS e clique em **Criar Banco de Dados;
- Selecione as opções de *Criação Padrão > MySQL > Nível Gratuito*.
- Abaixo, faça as configurações que achar necessário. Neste caso, manteremos como padrão.
- Altere a VPC para a criada anteriormente, bem como o Grupo de segurança e a preferência pela zona de disponibilidade.
- Finalize a criação do DB.
### EC2 (Elastic File Sistem)
- Vá até o serviço de EC2 no console da AWS e selecione a opção de Instâncias no menu lateral esquerdo;
- Em seguida, clique na opção de esxecutar instância;
- Adicione as seguintes Tags > *Para as chaves: Name, Project e CostCenter, selecione "intâncias", "volumes" e "interfaces de rede" como tipos de recurso e os valores respectivos;
- Determine sua imagem (AMI), neste caso usaremos a *Amazon Linux 2 AMI (HVM) - Kernek 5.10, SSD Volume Type* do tipo *t3.small*;
- Selecione seu par de chaver já existente ou crie um novo no formato *.pem*; (não esqueça de baixar seu arquivo para aceesar a instância)
- Clique em **editar** configuração de rede, selecione a VPC anteriormente já criada;
- Selecione a **Subnet pública 1a e habilite o endereçamento de ip público**;
- Após, selecione o Security Group do Bastion Host;
- Clique em **Detalhes Avançados** e, no fim da página, preencha o espaço em branco com seu userdata-instance.sh;
 -Segue exemplo abaixo:
```
#!/bin/bash
# Instalar Docker-CE ( Container Engine): 
sudo yum update
sudo yum install docker -y
sudo systemctl start docker
sudo systemctl enable docker
sudo usermod -aG docker ec2-user
# Instalar Docker Compose:
sudo curl -SL https://github.com/docker/compose/releases/download/v2.19.1/docker-compose-linux-x86_64 -o /usr/local/bin/docker-compose
sudo chmod +x /usr/local/bin/docker-compose
# EFS
sudo yum install amazon-efs-utils -y
sudo mkdir -m 666 /home/ec2-user/efs
sudo echo "${ID_EFS}   /home/ec2-user/efs    efs     defaults,_netdev    0   0" | sudo tee -a /etc/fstab
sudo mount -a
# Cria o docker-compose.yaml
sudo mkdir /home/ec2-user/docker-compose
sudo echo -e "version: '3.1'\n 
services:\n
  wordpress:\n
    image: wordpress:latest\n        
    restart: always\n
    ports:\n
      - 80:80\n
    environment:\n
      WORDPRESS_DB_HOST: ${ENDPOINT_RDS}\n
      WORDPRESS_DB_USER: admin\n
      WORDPRESS_DB_PASSWORD: 12345678\n
      WORDPRESS_DB_NAME: wordpress\n
    volumes:\n
      - /home/ec2-user/efs:/var/www/html\n
" | sudo tee /home/ec2-user/docker-compose/docker-compose.yml
sudo sed -i '/^$/d' /home/ec2-user/docker-compose/docker-compose.yml
```
- Finalize sua instância.
### Criação da AMI a partir da EC2
- Vá até o serviço de EC2 no console AWS e acesse as instãncias;
- Selecione a instância previamente criada, clique em ações > "imagem e modelos" > "criar imagem";
- Nomeie e finalize a criação.
### Criação do Auto Scaling Group
- Acesse o serviço de auto scaling grou pelo manu lateral e clique em **Criar grupo do Auto Scaling**;
- Em seguida, clique em **Criar modelo de execução** 
- Nomeie seu grupo, selecione a AMI criada, seu par de chaves, o grupo de segurança das Instancias e adicione as Tags de Recurso;
- Clique em detalhes avançados e cole o seguinte script:
```
#!/bin/bash
sudo yum update -y
sudo systemctl start docker
cd /home/ec2-user/docker-compose/
docker-compose up -d
```
- Clique em Criar Modelo de Execução;
- Volte para o passo anterior, selecione o modelo criado e clique em **Próximo**;
- Selecione a VPC criada anteriormente e as subnetes privadas de cada zona de disponibilidade > *Próximo*;
- Depois, selecione *Anexar a um novo balanceador de carga*;
- Nomeie e selecione a opção *Internet facing*;
- Verifique se a VPC é a certa e se as subnetes são as públicas das duas zonas de disponibilidade;
- Abaixo, na parte de Listeners clique na opçaõ para criar um novo, ele nomeia automático;
- Mantenha o restante padrão e clique em próximo;
- Define as capacidades que deseja. Neste caso, faremos da seguinte maneira (Capacidade desejada: 2, Capacidade mínima: 2, Capacidade máxima: 4);
- Clique em Próximo para esta e para a etapa seguinte;
- Repita as Tags de Recurso.
- Clique em Próximo > **Criar grupo auto Scaling**.
