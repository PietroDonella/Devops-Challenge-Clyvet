# 🐾 Clyvo Vet - Infraestrutura e DevOps (Challenge 2026)

## 📌 Descrição do Projeto
A **Clyvo Vet API** é o ecossistema backend desenvolvido em **.NET 10** (Clean Architecture) planejado para atuar como o sistema operacional definitivo para o relacionamento contínuo entre clínicas veterinárias, médicos veterinários e tutores de pets. A solução resolve o problema da descontinuidade clínica através da centralização avançada do histórico de saúde biológica e controle de atendimentos.

Este repositório documenta a camada de **Cloud Computing & DevOps**, demonstrando o provisionamento ágil e automatizado de recursos de rede e servidores na nuvem **Microsoft Azure** (Azure CLI), bem como a arquitetura de microsserviços isolados com containers autônomos para a aplicação .NET e o banco de dados Oracle Express Edition.

---

## 💼 Benefícios para o Negócio
* **Plataforma Preditiva e Inteligente:** Centraliza os dados biológicos do pet (raça, idade, peso), viabilizando futuros cruzamentos de algoritmos para predição de riscos de saúde e alertas de cuidado preventivo.
* **Redução da Descontinuidade Clínica:** Estabelece uma ponte direta de dados de atendimento, otimizando o fluxo de retornos e garantindo o acompanhamento vacinal integral do animal.
* **Eficiência e Handoff Médico:** O prontuário e histórico unificado permitem que o médico veterinário avalie o quadro do paciente com agilidade antes mesmo do início da consulta física.
* **CRM Clínico e Visão Gerencial:** Entrega indicadores estratégicos para as clínicas parceiras gerenciarem taxas de retenção, engajamento e adesão a tratamentos.

---

## 🛣️ Rotas da API (Endpoints REST)
A API expõe os seguintes barramentos para o consumo das operações de CRUD:
* **`/api/tutores`** (GET, POST, PUT, DELETE): Gestão de responsáveis pelos animais de estimação.
* **`/api/animais`** (GET, POST, PUT, DELETE): Gerenciamento do perfil biológico e dados dos pets.
* **`/api/clinicas`** (GET, POST, PUT, DELETE): Unidades de atendimento médico parceiras.
* **`/api/veterinarios`** (GET, POST, PUT, DELETE): Profissionais veterinários cadastrados no ecossistema.
* **`/api/prontuarios`** (GET, POST, PUT, DELETE): Histórico clínico de diagnósticos e ocorrências.
* **`/api/consultas`** (GET, POST, PUT, DELETE): Agendamentos e controle relacional de atendimentos.

---

## 🚀 Instalação da Solução (How to)

Siga rigorosamente a ordem sequencial de ações abaixo para executar e validar a solução na nuvem:

### 1. Bash no Azure
No terminal do **Azure Cloud Shell (Bash)**, configure as permissões e execute o script `criacao.sh` disponível na raiz do repositório para erguer toda a infraestrutura e o banco em nuvem de forma automatizada:
```bash
chmod +x criacao.sh
sed -i 's/\r$//' criacao.sh
./criacao.sh
```

### 2. Conectando na máquina
Assim que o script CLI finalizar a execução, copie o endereço IP Público gerado na Azure e conecte-se à Máquina Virtual via SSH pelo seu terminal local:
```bash
ssh admin_fiap@<SEU_IP_PUBLICO>
```
*(Utilize a senha de credenciais configurada: `Fiap2026Clyvet`)*

### 3. Acessando o Banco por linha de comando
Dentro do terminal Linux da VM, execute o comando abaixo para entrar na interface do SQL*Plus e injetar seus scripts de criação de tabelas (DDL) e inserts de teste no container Oracle Express:
```bash
docker exec -it oracle-db sqlplus system/\"Fiap2026Clyvet\"@//localhost:1521/XEPDB1
```

### 4. Criando o Dockerfile
Na pasta raiz onde os arquivos da sua aplicação estão localizados no servidor, utilize o bloco abaixo para estruturar as etapas de build e publicação multi-estágio da aplicação .NET 10:
```bash
cat <<EOF > Dockerfile
# Estágio de Build
FROM [mcr.microsoft.com/dotnet/sdk:10.0](https://mcr.microsoft.com/dotnet/sdk:10.0) AS build
WORKDIR /src
COPY ["ClyvoVet.Api/ClyvoVet.Api.csproj", "ClyvoVet.Api/"]
RUN dotnet restore "ClyvoVet.Api/ClyvoVet.Api.csproj"
COPY . .
WORKDIR "/src/ClyvoVet.Api"
RUN dotnet build "ClyvoVet.Api.csproj" -c Release -o /app/build
RUN dotnet publish "ClyvoVet.Api.csproj" -c Release -o /app/publish

# Estágio de Execução
FROM [mcr.microsoft.com/dotnet/aspnet:10.0](https://mcr.microsoft.com/dotnet/aspnet:10.0)
WORKDIR /app
COPY --from=build /app/publish .
EXPOSE 8080
ENV ASPNETCORE_URLS=http://+:8080
CMD ["dotnet", "ClyvoVet.Api.dll"]
EOF
```

### 5. Build da Imagem
Compile a imagem isolada da API .NET acionando as diretivas do arquivo criado:
```bash
docker build -t clyvovet-api-image .
```

### 6. Execução do Container
Suba o container do microsserviço da API rodando em background e integrado nativamente à rede interna mapeada para conversar com o Oracle:
```bash
docker run -d --name clyvovet-api -p 8080:8080 --network clyvovet-network clyvovet-api-image
```

---

## 🐳 Dockerfile
O arquivo `Dockerfile` corporativo estabelece a segregação de ambientes (Build vs Runtime) mitigando vulnerabilidades e garantindo pacotes leves para entrega contínua:

```dockerfile
# Estágio de Build
FROM [mcr.microsoft.com/dotnet/sdk:10.0](https://mcr.microsoft.com/dotnet/sdk:10.0) AS build
WORKDIR /src
COPY ["ClyvoVet.Api/ClyvoVet.Api.csproj", "ClyvoVet.Api/"]
RUN dotnet restore "ClyvoVet.Api/ClyvoVet.Api.csproj"
COPY . .
WORKDIR "/src/ClyvoVet.Api"
RUN dotnet build "ClyvoVet.Api.csproj" -c Release -o /app/build
RUN dotnet publish "ClyvoVet.Api.csproj" -c Release -o /app/publish

# Estágio de Execução
FROM [mcr.microsoft.com/dotnet/aspnet:10.0](https://mcr.microsoft.com/dotnet/aspnet:10.0)
WORKDIR /app
COPY --from=build /app/publish .
EXPOSE 8080
ENV ASPNETCORE_URLS=http://+:8080
CMD ["dotnet", "ClyvoVet.Api.dll"]
```

---

## ☁️ Script do Azure CLI (`criacao.sh`)
O script estrutural automatiza o ciclo completo de provisionamento de ativos de rede, regras de tráfego de entrada em portas específicas (22, 8080, 1521) e orquestração do provisionamento de dependências Linux e instâncias de banco de dados locais:

```bash
#!/bin/bash
GRUPO="clyvovet"
LOCATION="brazilsouth"
USER="admin_fiap"
PASSWORD='Fiap2026Clyvet'
RG="rg-$GRUPO"
VNET="vnet-$GRUPO"
SUBNET="subnet-$GRUPO"
NSG="nsg-$GRUPO"
VM="vm-$GRUPO"

echo "1. Criando Resource Group e Redes..."
az group create --name $RG --location $LOCATION --tags owner=$GRUPO environment=dev cost-center=fiap
az network vnet create --resource-group $RG --name $VNET --address-prefix 10.10.0.0/16 --subnet-name $SUBNET --subnet-prefix 10.10.1.0/24

echo "2. Criando Regras de Firewall (NSG)..."
az network nsg create --resource-group $RG --name $NSG
az network nsg rule create --resource-group $RG --nsg-name $NSG --name allow-ssh --protocol Tcp --priority 1000 --destination-port-range 22 --access Allow
az network nsg rule create --resource-group $RG --nsg-name $NSG --name allow-8080 --protocol Tcp --priority 1001 --destination-port-range 8080 --access Allow
az network nsg rule create --resource-group $RG --nsg-name $NSG --name allow-1521 --protocol Tcp --priority 1002 --destination-port-range 1521 --access Allow
az network vnet subnet update --resource-group $RG --vnet-name $VNET --name $SUBNET --network-security-group $NSG

echo "3. Criando a Máquina Virtual Linux..."
az vm create --resource-group $RG --name $VM --image Ubuntu2204 --admin-username $USER --admin-password $PASSWORD --authentication-type password --size Standard_E2s_v3 --vnet-name $VNET --subnet $SUBNET --nsg $NSG

echo "4. Instalando Docker e Subindo o Banco Oracle..."
az vm run-command invoke --resource-group $RG --name $VM --command-id RunShellScript --scripts '
export DEBIAN_FRONTEND=noninteractive
sudo apt-get update -y
sudo apt-get install -y ca-certificates curl git nano

sudo install -m 0755 -d /etc/apt/keyrings
sudo curl -fsSL [https://download.docker.com/linux/ubuntu/gpg](https://download.docker.com/linux/ubuntu/gpg) -o /etc/apt/keyrings/docker.asc
sudo chmod a+r /etc/apt/keyrings/docker.asc
echo "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.asc] [https://download.docker.com/linux/ubuntu](https://download.docker.com/linux/ubuntu) \$(. /etc/os-release && echo "\$VERSION_CODENAME") stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
sudo apt-get update -y
sudo apt-get install -y docker-ce docker-ce-cli containerd.io

sudo usermod -aG docker admin_fiap

sudo docker network create clyvovet-network
sudo docker volume create oracle-clyvovet-data

sudo docker run -d --name oracle-db \
  -p 1521:1521 \
  -v oracle-clyvovet-data:/opt/oracle/oradata \
  --network clyvovet-network \
  -e ORACLE_PASSWORD="Fiap2026Clyvet" \
  gvenzl/oracle-xe
'
echo "Provisionamento finalizado com sucesso!"
```
---

## 📐 Desenho Macro da Arquitetura

![Arquitetura da Solução](diagrama%20arquitetura.JPG)

### 📋 Descrição Detalhada da Arquitetura Cloud e Containers
* **Fluxo de Requisições Externas:** O cliente (administradores, veterinários, tutores ou testes via Postman) dispara requisições HTTP direcionadas à API utilizando o endereço IP Público alocado na infraestrutura da Azure.
* **Camada de Segurança perimetral (Network Security Group):** O tráfego de entrada bate diretamente no firewall de borda (NSG), que audita e bloqueia acessos indesejados, mantendo abertas e seguras apenas as portas configuradas no script: `22` para gerência operacional SSH, `8080` para comunicação com a API .NET e `1521` para conexões locais de monitoria ao banco de dados.
* **Segregação de Rede Comercial (VNet & Subnet):** Toda a arquitetura foi provisionada na região `brazilsouth` (Sul do Brasil) encapsulada dentro de uma Rede Virtual própria (`10.10.0.0/16`) com uma sub-rede lógica exclusiva (`10.10.1.0/24`) para hospedagem da infraestrutura de servidores.
* **Hospedagem Dedicada (Máquina Virtual Linux):** O nó computacional consiste em uma instância de VM escalável (`Standard_E2s_v3`) executando o sistema operacional corporativo Ubuntu Server 22.04 LTS, onde reside o ecossistema Docker Engine.
* **Isolamento de Redes em Containers (`clyvovet-network`):** Dentro da VM, criamos uma rede interna isolada no Docker. Os microsserviços conversam de forma privativa utilizando resolução de nomes DNS interna (`oracle-db`), impossibilitando acessos diretos externos à porta do banco e mitigando riscos de vazamento de dados.
* **Persistência de Dados via Volume Nomeado (`oracle-clyvovet-data`):** Para assegurar a con
