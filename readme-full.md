How to create cluster EKS for dummies

O gerenciamento do cluster é feito por um usuário no IAM.(Esse user se torna OWNER do EKS+kubernets)
Dessa forma iremos criar um usuário no IAM e dar permissao (Administrator Acess)

1 - Criar user no IAM (Utilizando CONSOLE)
Permissao Administrator Acess

########################################################################################################
Nesse passo iremos criar um KMS
(Utilizando CLI lembre-se de estar logado com o user que foi criado no passo 1)

2 - Create an AWS KMS Custom Managed Key (CMK) 

#aws kms create-alias --alias-name alias/eksworkshop --target-key-id $(aws kms create-key --query KeyMetadata.Arn --output text)

#export cmk
#export MASTER_ARN=$(aws kms describe-key --key-id alias/eksworkshop --query KeyMetadata.Arn --output text)

#echo "export MASTER_ARN=${MASTER_ARN}" | tee -a ~/.bash_profile

###########################################################################################################################################

Instalação do eksctl na sua maquina.Verifique a versão do seu sistema operacional na documentação oficial AWS
https://docs.aws.amazon.com/eks/latest/userguide/eksctl.html

3 - Fazer os pre-reqs eksctl
#curl --silent --location "https://github.com/weaveworks/eksctl/releases/latest/download/eksctl_$(uname -s)_amd64.tar.gz" | tar xz -C /tmp

#sudo mv -v /tmp/eksctl /usr/local/bin

#eksctl version

#eksctl completion bash >> ~/.bash_completion
. /etc/profile.d/bash_completion.sh
. ~/.bash_completion



curl -o kubectl.sha256 https://s3.us-west-2.amazonaws.com/amazon-eks/1.21.2/2021-07-05/bin/linux/amd64/kubectl.sha256

###########################################################################################################################################

Retorna detalhes sobre o usuário ou função do IAM cujas credenciais são usadas para chamar a operação.
4 - Checa user se o user IAM tem comunicação AWS

#aws sts get-caller-identity

###########################################################################################################################################

Crie um arquivo de implantação eksctl (eksworkshop.yaml) para usar na criação do cluster usando a seguinte sintaxe:

5 - Create an EKS cluster

cat << EOF > eksworkshop.yaml
---
apiVersion: eksctl.io/v1alpha5
kind: ClusterConfig

metadata:
  name: eksworkshop-eksctl
  region: ${AWS_REGION}
  version: "1.19"

#Identifica AZS baseada na region declarada(Cria automaticamente as subnets)
availabilityZones: ["${AZS[0]}", "${AZS[1]}", "${AZS[2]}"]

#Gerenciador de Instancias lancadas
managedNodeGroups:
- name: nodegroup
  desiredCapacity: 3
  instanceType: t3.small
  ssh:
    enableSsm: true

# To enable all of the control plane logs, uncomment below:
# cloudWatch:
#  clusterLogging:
#    enableTypes: ["*"]

#Pega o KMS criado no Passo 2
secretsEncryption:
  keyARN: ${MASTER_ARN}
EOF

###########################################################################################################################################

6 - Criar o cluster utilizando o arquivo <eksworkshop.yaml>

Lembrando que para este exercicio os campos que foram alterados no passo 5 foram.

metadata:
  region: us-east-1
  
availabilityZones: ["us-east-1a", "us-east-1b", "us-east-1c"]

Após isso execute o comando abaixo com o nome do arquivo criado

#eksctl create cluster -f eksworkshop-v2.yaml

Launching EKS and all the dependencies will take approximately 15 minutes

###########################################################################################################################################

Lembre-se esse teste deve ser efetuado com o usuario que voce criou no passo 1
7 - Test the cluster.

Confirm your nodes:
kubectl get nodes # if we see our 3 nodes, we know we have authenticated correctly

###########################################################################################################################################

8 - Now you can verify your entry in the AWS auth map within the console.
kubectl describe configmap -n kube-system aws-auth

###########################################################################################################################################

9 - Para limpeza dos recursos lançados nesse tutorial voce executa o comando abaixo:

to cleanup resources, run 'eksctl delete cluster --region=us-east-1 --name=eksworkshop-eksctl'
