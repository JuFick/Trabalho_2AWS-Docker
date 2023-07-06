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
