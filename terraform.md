# 🏗️ Terraform - Infrastructure as Code

Guide complet de Terraform pour gérer l'infrastructure cloud de manière déclarative.

---

## 📚 Introduction

**Terraform** = Infrastructure as Code (IaC) tool par HashiCorp

**Pourquoi Terraform ?**
- Infrastructure versionnée (Git)
- Reproductible et prévisible
- Multi-cloud (AWS, Azure, GCP, etc.)
- State management
- Plan before apply

**Providers supportés :** 3000+ (AWS, Azure, GCP, Kubernetes, Docker, GitHub, etc.)

---

## 🚀 Installation

```bash
# Linux (Debian/Ubuntu)
wget -O- https://apt.releases.hashicorp.com/gpg | sudo gpg --dearmor -o /usr/share/keyrings/hashicorp-archive-keyring.gpg
echo "deb [signed-by=/usr/share/keyrings/hashicorp-archive-keyring.gpg] https://apt.releases.hashicorp.com $(lsb_release -cs) main" | sudo tee /etc/apt/sources.list.d/hashicorp.list
sudo apt update && sudo apt install terraform

# macOS
brew tap hashicorp/tap
brew install hashicorp/tap/terraform

# Vérifier installation
terraform version
```

---

## 📝 Syntaxe HCL (HashiCorp Configuration Language)

### Structure de Base

```hcl
# main.tf
terraform {
  required_version = ">= 1.0"
  
  required_providers {
    aws = {
      source  = "hashicorp/aws"
      version = "~> 5.0"
    }
  }
}

provider "aws" {
  region = "us-east-1"
}

resource "aws_instance" "web" {
  ami           = "ami-0c55b159cbfafe1f0"
  instance_type = "t2.micro"
  
  tags = {
    Name = "WebServer"
    Environment = "Production"
  }
}
```

### Variables

```hcl
# variables.tf
variable "region" {
  description = "AWS region"
  type        = string
  default     = "us-east-1"
}

variable "instance_type" {
  description = "EC2 instance type"
  type        = string
  default     = "t2.micro"
  
  validation {
    condition     = can(regex("^t2\\.", var.instance_type))
    error_message = "Instance type must be t2.* family"
  }
}

variable "tags" {
  description = "Tags to apply to resources"
  type        = map(string)
  default     = {}
}

variable "enable_monitoring" {
  description = "Enable CloudWatch monitoring"
  type        = bool
  default     = false
}

variable "availability_zones" {
  description = "List of AZs"
  type        = list(string)
  default     = ["us-east-1a", "us-east-1b"]
}

# Utilisation
resource "aws_instance" "web" {
  ami           = "ami-xxx"
  instance_type = var.instance_type
  
  tags = merge(
    var.tags,
    {
      Name = "WebServer"
    }
  )
}
```

**Définir variables :**
```bash
# terraform.tfvars
region = "eu-west-1"
instance_type = "t2.small"
tags = {
  Project = "MyApp"
  Owner   = "DevOps Team"
}

# Ligne de commande
terraform apply -var="region=eu-west-1"
terraform apply -var-file="production.tfvars"

# Variable d'environnement
export TF_VAR_region="eu-west-1"
terraform apply
```

### Outputs

```hcl
# outputs.tf
output "instance_id" {
  description = "ID of the EC2 instance"
  value       = aws_instance.web.id
}

output "instance_public_ip" {
  description = "Public IP of instance"
  value       = aws_instance.web.public_ip
}

output "instance_private_ip" {
  description = "Private IP of instance"
  value       = aws_instance.web.private_ip
  sensitive   = true  # Ne pas afficher dans logs
}

# Afficher outputs
terraform output
terraform output instance_id
terraform output -json
```

### Locals

```hcl
# Valeurs calculées/composées
locals {
  common_tags = {
    Project     = "MyApp"
    Environment = terraform.workspace
    ManagedBy   = "Terraform"
  }
  
  instance_name = "${var.project_name}-${terraform.workspace}-web"
  
  azs = slice(data.aws_availability_zones.available.names, 0, 3)
}

resource "aws_instance" "web" {
  ami           = "ami-xxx"
  instance_type = var.instance_type
  
  tags = merge(
    local.common_tags,
    {
      Name = local.instance_name
    }
  )
}
```

---

## 🔧 Commandes Terraform

### terraform init

```bash
# Initialiser projet (télécharge providers)
terraform init

# Upgrade providers
terraform init -upgrade

# Backend config
terraform init -backend-config="bucket=my-tf-state"

# Reconfigure backend
terraform init -reconfigure
```

### terraform plan

```bash
# Voir changements
terraform plan

# Sauvegarder plan
terraform plan -out=tfplan

# Cibler ressource spécifique
terraform plan -target=aws_instance.web

# Destroy plan
terraform plan -destroy

# Parallel execution
terraform plan -parallelism=20
```

### terraform apply

```bash
# Appliquer changements
terraform apply

# Auto-approve (sans confirmation)
terraform apply -auto-approve

# Depuis plan sauvegardé
terraform apply tfplan

# Cibler ressource
terraform apply -target=aws_instance.web

# Variables
terraform apply -var="region=eu-west-1"
terraform apply -var-file="prod.tfvars"
```

### terraform destroy

```bash
# Détruire toute l'infrastructure
terraform destroy

# Auto-approve
terraform destroy -auto-approve

# Cibler ressource
terraform destroy -target=aws_instance.web
```

### terraform state

```bash
# Lister ressources
terraform state list

# Voir détails ressource
terraform state show aws_instance.web

# Déplacer ressource
terraform state mv aws_instance.web aws_instance.app

# Supprimer du state (sans détruire)
terraform state rm aws_instance.web

# Pull state
terraform state pull

# Push state
terraform state push

# Replace provider (migration)
terraform state replace-provider hashicorp/aws registry.terraform.io/hashicorp/aws
```

### terraform import

```bash
# Importer ressource existante
terraform import aws_instance.web i-1234567890abcdef0

# Exemples AWS
terraform import aws_s3_bucket.bucket my-bucket-name
terraform import aws_vpc.main vpc-1234567890
terraform import aws_security_group.sg sg-1234567890
```

### Autres commandes

```bash
# Valider syntaxe
terraform validate

# Formater code
terraform fmt
terraform fmt -recursive

# Afficher outputs
terraform output
terraform output -json

# Voir graphe dépendances
terraform graph | dot -Tsvg > graph.svg

# Console interactif
terraform console

# Show current state
terraform show

# Refresh state
terraform refresh

# Workspaces
terraform workspace list
terraform workspace new dev
terraform workspace select prod
terraform workspace delete dev
```

---

## 🏢 Ressources AWS

### EC2 Instance

```hcl
resource "aws_instance" "web" {
  ami           = data.aws_ami.ubuntu.id
  instance_type = "t2.micro"
  key_name      = aws_key_pair.deployer.key_name
  
  vpc_security_group_ids = [aws_security_group.web.id]
  subnet_id              = aws_subnet.public.id
  
  user_data = file("user_data.sh")
  
  root_block_device {
    volume_size = 20
    volume_type = "gp3"
    encrypted   = true
  }
  
  tags = {
    Name = "WebServer"
  }
  
  lifecycle {
    create_before_destroy = true
    ignore_changes        = [ami]
  }
}

# AMI lookup
data "aws_ami" "ubuntu" {
  most_recent = true
  owners      = ["099720109477"]  # Canonical
  
  filter {
    name   = "name"
    values = ["ubuntu/images/hvm-ssd/ubuntu-jammy-22.04-amd64-server-*"]
  }
}
```

### VPC et Networking

```hcl
# VPC
resource "aws_vpc" "main" {
  cidr_block           = "10.0.0.0/16"
  enable_dns_hostnames = true
  enable_dns_support   = true
  
  tags = {
    Name = "main-vpc"
  }
}

# Subnets
resource "aws_subnet" "public" {
  count             = 2
  vpc_id            = aws_vpc.main.id
  cidr_block        = "10.0.${count.index}.0/24"
  availability_zone = data.aws_availability_zones.available.names[count.index]
  
  map_public_ip_on_launch = true
  
  tags = {
    Name = "public-subnet-${count.index + 1}"
  }
}

resource "aws_subnet" "private" {
  count             = 2
  vpc_id            = aws_vpc.main.id
  cidr_block        = "10.0.${count.index + 10}.0/24"
  availability_zone = data.aws_availability_zones.available.names[count.index]
  
  tags = {
    Name = "private-subnet-${count.index + 1}"
  }
}

# Internet Gateway
resource "aws_internet_gateway" "main" {
  vpc_id = aws_vpc.main.id
  
  tags = {
    Name = "main-igw"
  }
}

# Route Table
resource "aws_route_table" "public" {
  vpc_id = aws_vpc.main.id
  
  route {
    cidr_block = "0.0.0.0/0"
    gateway_id = aws_internet_gateway.main.id
  }
  
  tags = {
    Name = "public-rt"
  }
}

# Route Table Association
resource "aws_route_table_association" "public" {
  count          = length(aws_subnet.public)
  subnet_id      = aws_subnet.public[count.index].id
  route_table_id = aws_route_table.public.id
}

# NAT Gateway (pour private subnets)
resource "aws_eip" "nat" {
  domain = "vpc"
}

resource "aws_nat_gateway" "main" {
  allocation_id = aws_eip.nat.id
  subnet_id     = aws_subnet.public[0].id
  
  tags = {
    Name = "main-nat"
  }
}

# Security Group
resource "aws_security_group" "web" {
  name        = "web-sg"
  description = "Allow HTTP/HTTPS inbound"
  vpc_id      = aws_vpc.main.id
  
  ingress {
    from_port   = 80
    to_port     = 80
    protocol    = "tcp"
    cidr_blocks = ["0.0.0.0/0"]
  }
  
  ingress {
    from_port   = 443
    to_port     = 443
    protocol    = "tcp"
    cidr_blocks = ["0.0.0.0/0"]
  }
  
  egress {
    from_port   = 0
    to_port     = 0
    protocol    = "-1"
    cidr_blocks = ["0.0.0.0/0"]
  }
  
  tags = {
    Name = "web-sg"
  }
}
```

### S3 Bucket

```hcl
resource "aws_s3_bucket" "assets" {
  bucket = "my-app-assets-${random_id.bucket.hex}"
  
  tags = {
    Name = "Assets Bucket"
  }
}

resource "aws_s3_bucket_versioning" "assets" {
  bucket = aws_s3_bucket.assets.id
  
  versioning_configuration {
    status = "Enabled"
  }
}

resource "aws_s3_bucket_encryption" "assets" {
  bucket = aws_s3_bucket.assets.id
  
  rule {
    apply_server_side_encryption_by_default {
      sse_algorithm = "AES256"
    }
  }
}

resource "aws_s3_bucket_public_access_block" "assets" {
  bucket = aws_s3_bucket.assets.id
  
  block_public_acls       = true
  block_public_policy     = true
  ignore_public_acls      = true
  restrict_public_buckets = true
}

resource "random_id" "bucket" {
  byte_length = 4
}
```

### RDS Database

```hcl
resource "aws_db_instance" "mysql" {
  identifier           = "myapp-db"
  engine               = "mysql"
  engine_version       = "8.0"
  instance_class       = "db.t3.micro"
  allocated_storage    = 20
  storage_type         = "gp3"
  storage_encrypted    = true
  
  db_name  = "myappdb"
  username = "admin"
  password = random_password.db_password.result
  
  vpc_security_group_ids = [aws_security_group.db.id]
  db_subnet_group_name   = aws_db_subnet_group.main.name
  
  backup_retention_period = 7
  backup_window          = "03:00-04:00"
  maintenance_window     = "mon:04:00-mon:05:00"
  
  skip_final_snapshot = false
  final_snapshot_identifier = "myapp-db-final-snapshot"
  
  tags = {
    Name = "MyApp Database"
  }
}

resource "random_password" "db_password" {
  length  = 16
  special = true
}

resource "aws_db_subnet_group" "main" {
  name       = "main-db-subnet-group"
  subnet_ids = aws_subnet.private[*].id
  
  tags = {
    Name = "Main DB Subnet Group"
  }
}
```

### Load Balancer

```hcl
# Application Load Balancer
resource "aws_lb" "main" {
  name               = "main-alb"
  internal           = false
  load_balancer_type = "application"
  security_groups    = [aws_security_group.alb.id]
  subnets            = aws_subnet.public[*].id
  
  enable_deletion_protection = false
  
  tags = {
    Name = "Main ALB"
  }
}

# Target Group
resource "aws_lb_target_group" "web" {
  name     = "web-tg"
  port     = 80
  protocol = "HTTP"
  vpc_id   = aws_vpc.main.id
  
  health_check {
    enabled             = true
    healthy_threshold   = 2
    interval            = 30
    matcher             = "200"
    path                = "/health"
    port                = "traffic-port"
    protocol            = "HTTP"
    timeout             = 5
    unhealthy_threshold = 2
  }
}

# Listener
resource "aws_lb_listener" "http" {
  load_balancer_arn = aws_lb.main.arn
  port              = "80"
  protocol          = "HTTP"
  
  default_action {
    type             = "forward"
    target_group_arn = aws_lb_target_group.web.arn
  }
}

# HTTPS Listener
resource "aws_lb_listener" "https" {
  load_balancer_arn = aws_lb.main.arn
  port              = "443"
  protocol          = "HTTPS"
  ssl_policy        = "ELBSecurityPolicy-2016-08"
  certificate_arn   = aws_acm_certificate.main.arn
  
  default_action {
    type             = "forward"
    target_group_arn = aws_lb_target_group.web.arn
  }
}
```

---

## 🔄 Modules

### Créer un Module

```hcl
# modules/vpc/main.tf
variable "vpc_cidr" {
  type = string
}

variable "name" {
  type = string
}

resource "aws_vpc" "this" {
  cidr_block = var.vpc_cidr
  
  tags = {
    Name = var.name
  }
}

output "vpc_id" {
  value = aws_vpc.this.id
}

output "vpc_cidr" {
  value = aws_vpc.this.cidr_block
}
```

### Utiliser un Module

```hcl
# main.tf
module "vpc" {
  source = "./modules/vpc"
  
  vpc_cidr = "10.0.0.0/16"
  name     = "production-vpc"
}

# Référencer outputs du module
resource "aws_subnet" "public" {
  vpc_id     = module.vpc.vpc_id
  cidr_block = "10.0.1.0/24"
}

# Module depuis Terraform Registry
module "vpc" {
  source  = "terraform-aws-modules/vpc/aws"
  version = "5.0.0"
  
  name = "my-vpc"
  cidr = "10.0.0.0/16"
  
  azs             = ["us-east-1a", "us-east-1b"]
  private_subnets = ["10.0.1.0/24", "10.0.2.0/24"]
  public_subnets  = ["10.0.101.0/24", "10.0.102.0/24"]
  
  enable_nat_gateway = true
  enable_vpn_gateway = false
}
```

---

## 🗄️ Backend et State

### Local Backend (défaut)

```hcl
# terraform.tf
terraform {
  backend "local" {
    path = "terraform.tfstate"
  }
}
```

### S3 Backend (recommandé production)

```hcl
terraform {
  backend "s3" {
    bucket         = "my-terraform-state"
    key            = "prod/terraform.tfstate"
    region         = "us-east-1"
    encrypt        = true
    dynamodb_table = "terraform-lock"
  }
}

# DynamoDB pour locking
resource "aws_dynamodb_table" "terraform_lock" {
  name           = "terraform-lock"
  billing_mode   = "PAY_PER_REQUEST"
  hash_key       = "LockID"
  
  attribute {
    name = "LockID"
    type = "S"
  }
}
```

### Remote Backend (Terraform Cloud)

```hcl
terraform {
  cloud {
    organization = "my-org"
    
    workspaces {
      name = "production"
    }
  }
}
```

---

## 🔁 Loops et Conditions

### count

```hcl
# Créer multiple ressources
resource "aws_instance" "web" {
  count = 3
  
  ami           = "ami-xxx"
  instance_type = "t2.micro"
  
  tags = {
    Name = "web-${count.index + 1}"
  }
}

# Référencer
output "instance_ids" {
  value = aws_instance.web[*].id
}

# Condition avec count
resource "aws_instance" "web" {
  count = var.create_instance ? 1 : 0
  
  ami           = "ami-xxx"
  instance_type = "t2.micro"
}
```

### for_each

```hcl
# Map
variable "instances" {
  type = map(string)
  default = {
    web = "t2.micro"
    app = "t2.small"
    db  = "t2.medium"
  }
}

resource "aws_instance" "server" {
  for_each = var.instances
  
  ami           = "ami-xxx"
  instance_type = each.value
  
  tags = {
    Name = each.key
  }
}

# Set
variable "subnet_cidrs" {
  type = set(string)
  default = ["10.0.1.0/24", "10.0.2.0/24"]
}

resource "aws_subnet" "example" {
  for_each = var.subnet_cidrs
  
  vpc_id     = aws_vpc.main.id
  cidr_block = each.value
}
```

### for expression

```hcl
# Transform list
locals {
  uppercase_names = [for name in var.names : upper(name)]
  
  # Avec condition
  small_instances = [for inst in var.instances : inst if inst.size == "small"]
  
  # Map transformation
  instance_ips = {
    for name, instance in aws_instance.server :
    name => instance.private_ip
  }
}
```

### dynamic blocks

```hcl
variable "ingress_rules" {
  type = list(object({
    port        = number
    cidr_blocks = list(string)
  }))
}

resource "aws_security_group" "example" {
  name = "example"
  
  dynamic "ingress" {
    for_each = var.ingress_rules
    
    content {
      from_port   = ingress.value.port
      to_port     = ingress.value.port
      protocol    = "tcp"
      cidr_blocks = ingress.value.cidr_blocks
    }
  }
}
```

---

## 🔒 Bonnes Pratiques

### 1. Structure de Projet

```
terraform-project/
├── environments/
│   ├── dev/
│   │   ├── main.tf
│   │   ├── variables.tf
│   │   ├── outputs.tf
│   │   └── terraform.tfvars
│   ├── staging/
│   └── prod/
├── modules/
│   ├── vpc/
│   ├── ec2/
│   └── rds/
├── README.md
└── .gitignore
```

### 2. .gitignore

```
# .gitignore
**/.terraform/*
*.tfstate
*.tfstate.*
crash.log
*.tfvars
override.tf
override.tf.json
.terraform.lock.hcl
```

### 3. Remote State

```hcl
# ✅ Bon : S3 + DynamoDB locking
terraform {
  backend "s3" {
    bucket         = "terraform-state"
    key            = "prod/terraform.tfstate"
    region         = "us-east-1"
    encrypt        = true
    dynamodb_table = "terraform-lock"
  }
}
```

### 4. Versioning

```hcl
# ✅ Bon : Spécifier versions
terraform {
  required_version = "~> 1.6"
  
  required_providers {
    aws = {
      source  = "hashicorp/aws"
      version = "~> 5.0"
    }
  }
}
```

### 5. Secrets Management

```hcl
# ✅ Bon : Utiliser AWS Secrets Manager
data "aws_secretsmanager_secret_version" "db_password" {
  secret_id = "prod/db/password"
}

resource "aws_db_instance" "main" {
  password = data.aws_secretsmanager_secret_version.db_password.secret_string
}

# ❌ Mauvais : Hardcoder secrets
resource "aws_db_instance" "main" {
  password = "mysecretpassword"  # NE JAMAIS FAIRE
}
```

### 6. Tagging Strategy

```hcl
locals {
  common_tags = {
    Project     = var.project_name
    Environment = var.environment
    ManagedBy   = "Terraform"
    Owner       = "DevOps Team"
    CostCenter  = "Engineering"
  }
}

resource "aws_instance" "web" {
  tags = merge(
    local.common_tags,
    {
      Name = "web-server"
      Role = "frontend"
    }
  )
}
```

---

## 📊 Exemple Complet - Infrastructure 3-Tier

```hcl
# main.tf
terraform {
  required_version = ">= 1.6"
  
  required_providers {
    aws = {
      source  = "hashicorp/aws"
      version = "~> 5.0"
    }
  }
  
  backend "s3" {
    bucket         = "my-terraform-state"
    key            = "prod/infrastructure.tfstate"
    region         = "us-east-1"
    encrypt        = true
    dynamodb_table = "terraform-lock"
  }
}

provider "aws" {
  region = var.aws_region
  
  default_tags {
    tags = local.common_tags
  }
}

# VPC Module
module "vpc" {
  source  = "terraform-aws-modules/vpc/aws"
  version = "5.0.0"
  
  name = "${var.project_name}-vpc"
  cidr = var.vpc_cidr
  
  azs             = data.aws_availability_zones.available.names
  private_subnets = var.private_subnet_cidrs
  public_subnets  = var.public_subnet_cidrs
  
  enable_nat_gateway = true
  enable_vpn_gateway = false
  
  tags = {
    Tier = "Network"
  }
}

# Web Tier (ALB + EC2)
resource "aws_lb" "web" {
  name               = "${var.project_name}-alb"
  internal           = false
  load_balancer_type = "application"
  security_groups    = [module.web_sg.security_group_id]
  subnets            = module.vpc.public_subnets
}

resource "aws_launch_template" "web" {
  name_prefix   = "${var.project_name}-web-"
  image_id      = data.aws_ami.amazon_linux_2.id
  instance_type = var.web_instance_type
  
  vpc_security_group_ids = [module.web_sg.security_group_id]
  
  user_data = base64encode(templatefile("${path.module}/user_data.sh", {
    app_port = 8080
  }))
  
  lifecycle {
    create_before_destroy = true
  }
}

resource "aws_autoscaling_group" "web" {
  name                = "${var.project_name}-web-asg"
  vpc_zone_identifier = module.vpc.private_subnets
  target_group_arns   = [aws_lb_target_group.web.arn]
  health_check_type   = "ELB"
  min_size            = var.web_asg_min
  max_size            = var.web_asg_max
  desired_capacity    = var.web_asg_desired
  
  launch_template {
    id      = aws_launch_template.web.id
    version = "$Latest"
  }
  
  tag {
    key                 = "Name"
    value               = "${var.project_name}-web"
    propagate_at_launch = true
  }
}

# App Tier (EC2)
resource "aws_instance" "app" {
  count         = var.app_instance_count
  ami           = data.aws_ami.amazon_linux_2.id
  instance_type = var.app_instance_type
  subnet_id     = element(module.vpc.private_subnets, count.index)
  
  vpc_security_group_ids = [module.app_sg.security_group_id]
  
  tags = {
    Name = "${var.project_name}-app-${count.index + 1}"
    Tier = "Application"
  }
}

# Database Tier (RDS)
module "rds" {
  source  = "terraform-aws-modules/rds/aws"
  version = "6.0.0"
  
  identifier = "${var.project_name}-db"
  
  engine               = "mysql"
  engine_version       = "8.0"
  family               = "mysql8.0"
  major_engine_version = "8.0"
  instance_class       = var.db_instance_class
  
  allocated_storage     = 20
  max_allocated_storage = 100
  storage_encrypted     = true
  
  db_name  = var.db_name
  username = var.db_username
  password = random_password.db_password.result
  port     = 3306
  
  multi_az               = true
  db_subnet_group_name   = module.vpc.database_subnet_group_name
  vpc_security_group_ids = [module.db_sg.security_group_id]
  
  backup_retention_period = 7
  skip_final_snapshot     = false
  deletion_protection     = true
  
  enabled_cloudwatch_logs_exports = ["error", "general", "slowquery"]
  
  tags = {
    Tier = "Database"
  }
}

# Security Groups Modules
module "web_sg" {
  source = "terraform-aws-modules/security-group/aws//modules/http-80"
  
  name   = "${var.project_name}-web-sg"
  vpc_id = module.vpc.vpc_id
  
  ingress_cidr_blocks = ["0.0.0.0/0"]
}

module "app_sg" {
  source = "terraform-aws-modules/security-group/aws"
  
  name   = "${var.project_name}-app-sg"
  vpc_id = module.vpc.vpc_id
  
  ingress_with_source_security_group_id = [
    {
      from_port                = 8080
      to_port                  = 8080
      protocol                 = "tcp"
      source_security_group_id = module.web_sg.security_group_id
    }
  ]
}

module "db_sg" {
  source = "terraform-aws-modules/security-group/aws"
  
  name   = "${var.project_name}-db-sg"
  vpc_id = module.vpc.vpc_id
  
  ingress_with_source_security_group_id = [
    {
      from_port                = 3306
      to_port                  = 3306
      protocol                 = "tcp"
      source_security_group_id = module.app_sg.security_group_id
    }
  ]
}

# Data Sources
data "aws_availability_zones" "available" {
  state = "available"
}

data "aws_ami" "amazon_linux_2" {
  most_recent = true
  owners      = ["amazon"]
  
  filter {
    name   = "name"
    values = ["amzn2-ami-hvm-*-x86_64-gp2"]
  }
}

# Random password
resource "random_password" "db_password" {
  length  = 16
  special = true
}

# Store password in Secrets Manager
resource "aws_secretsmanager_secret" "db_password" {
  name = "${var.project_name}/db/password"
}

resource "aws_secretsmanager_secret_version" "db_password" {
  secret_id     = aws_secretsmanager_secret.db_password.id
  secret_string = random_password.db_password.result
}

# Locals
locals {
  common_tags = {
    Project     = var.project_name
    Environment = var.environment
    ManagedBy   = "Terraform"
  }
}
```

---

**🎓 Prochaine étape : [Ansible Configuration Management](./ansible.md) →**
