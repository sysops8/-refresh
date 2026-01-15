# Terraform Refresh: Ежедневный/Полугодовой курс для DevOps/SysAdmin

**Цель:** Освежить в памяти ключевые концепции Terraform за 2-3 часа практики и узнать 1-2 новые продвинутые техники.

**Формат:** Каждый раздел состоит из:

1. **Краткой теории (Напоминалка)**: Самое главное, что вы могли забыть
2. **Практического задания**: Реальная задача, которую нужно решить
3. **Бонусного задания (для роста)**: Задача посложнее или с использованием новой фичи

**Предварительные требования:**

- Linux/Unix базовые знания
- AWS Free Tier аккаунт (или другой cloud provider)
- AWS CLI установлен и настроен
- Git базовые знания
- Понимание Infrastructure as Code концепции

---

## Модуль 1: Базовая архитектура и установка (20 минут)

### 🎯 Напоминалка

**Terraform - что это:**

```
Terraform
├── Infrastructure as Code (IaC) инструмент
├── Декларативный подход (описываем желаемое состояние)
├── Provider-агностик (AWS, Azure, GCP, etc)
├── State management (отслеживание состояния)
├── План перед применением (plan → apply)
└── HCL (HashiCorp Configuration Language)
```

**Ключевые компоненты:**

```bash
Terraform Workflow
├── Write: Пишем .tf файлы
├── Init: terraform init (загружает providers)
├── Plan: terraform plan (предварительный просмотр)
├── Apply: terraform apply (применяет изменения)
└── Destroy: terraform destroy (удаляет ресурсы)

Core Components
├── Resources: Инфраструктурные объекты
├── Data Sources: Чтение существующих ресурсов
├── Variables: Параметризация
├── Outputs: Экспорт значений
├── Providers: Плагины для API
├── Modules: Переиспользуемые блоки
└── State: Текущее состояние
```

**Установка:**

```bash
# Linux (Ubuntu/Debian)
wget -O- https://apt.releases.hashicorp.com/gpg | sudo gpg --dearmor -o /usr/share/keyrings/hashicorp-archive-keyring.gpg
echo "deb [signed-by=/usr/share/keyrings/hashicorp-archive-keyring.gpg] https://apt.releases.hashicorp.com $(lsb_release -cs) main" | sudo tee /etc/apt/sources.list.d/hashicorp.list
sudo apt update && sudo apt install terraform

# macOS
brew tap hashicorp/tap
brew install hashicorp/tap/terraform

# Проверка
terraform version
terraform -help
```

**Базовая структура проекта:**

```
terraform-project/
├── main.tf              # Основные ресурсы
├── variables.tf         # Входные переменные
├── outputs.tf           # Выходные значения
├── terraform.tfvars     # Значения переменных
├── versions.tf          # Требования к версиям
├── provider.tf          # Конфигурация провайдеров
├── backend.tf           # Конфигурация state backend
└── modules/             # Локальные модули
```

**Базовый синтаксис HCL:**

```hcl
# Ресурс
resource "resource_type" "name" {
  argument1 = "value1"
  argument2 = "value2"
}

# Переменная
variable "instance_type" {
  description = "EC2 instance type"
  type        = string
  default     = "t3.micro"
}

# Output
output "instance_ip" {
  value = aws_instance.web.public_ip
}

# Data source
data "aws_ami" "ubuntu" {
  most_recent = true
  owners      = ["099720109477"]
  
  filter {
    name   = "name"
    values = ["ubuntu/images/hvm-ssd/ubuntu-jammy-22.04-amd64-server-*"]
  }
}
```

**Основные команды:**

```bash
# Инициализация
terraform init

# Валидация
terraform validate

# Форматирование
terraform fmt

# План
terraform plan
terraform plan -out=plan.tfplan

# Применение
terraform apply
terraform apply plan.tfplan
terraform apply -auto-approve

# Просмотр state
terraform show
terraform state list

# Удаление
terraform destroy

# Workspace
terraform workspace list
terraform workspace new dev
terraform workspace select dev

# Граф зависимостей
terraform graph | dot -Tpng > graph.png
```

### 💻 Задание

Создай первый Terraform проект с Docker provider (для локальной работы):

**1. Создай структуру:**

```bash
mkdir terraform-basics
cd terraform-basics
```

**2. provider.tf:**

```hcl
terraform {
  required_version = ">= 1.6.0"
  
  required_providers {
    docker = {
      source  = "kreuzwerker/docker"
      version = "~> 3.0"
    }
  }
}

provider "docker" {
  host = "unix:///var/run/docker.sock"
}
```

**3. variables.tf:**

```hcl
variable "container_name" {
  description = "Name of the Docker container"
  type        = string
  default     = "my_nginx"
}

variable "image_name" {
  description = "Docker image name"
  type        = string
  default     = "nginx:alpine"
}

variable "external_port" {
  description = "External port to expose"
  type        = number
  default     = 8080
}

variable "container_count" {
  description = "Number of containers"
  type        = number
  default     = 2
}
```

**4. main.tf:**

```hcl
resource "docker_image" "nginx" {
  name         = var.image_name
  keep_locally = false
}

resource "docker_container" "nginx" {
  count = var.container_count
  
  name  = "${var.container_name}_${count.index + 1}"
  image = docker_image.nginx.image_id
  
  ports {
    internal = 80
    external = var.external_port + count.index
  }
}
```

**5. outputs.tf:**

```hcl
output "container_ids" {
  description = "IDs of created containers"
  value       = docker_container.nginx[*].id
}

output "container_urls" {
  description = "URLs to access containers"
  value = [
    for idx in range(var.container_count) :
    "http://localhost:${var.external_port + idx}"
  ]
}
```

**6. Запуск:**

```bash
terraform init
terraform validate
terraform fmt
terraform plan
terraform apply

# Проверка
docker ps
curl http://localhost:8080

# Удаление
terraform destroy
```

### 🚀 Бонус (новое)

**1. Используй locals:**

```hcl
# locals.tf
locals {
  common_labels = {
    environment = terraform.workspace
    managed_by  = "Terraform"
    project     = "learning"
  }
  
  container_prefix = "${var.container_name}-${terraform.workspace}"
}

# Использование в main.tf
resource "docker_container" "nginx" {
  count = var.container_count
  name  = "${local.container_prefix}_${count.index + 1}"
  
  labels {
    label = "environment"
    value = local.common_labels.environment
  }
}
```

**2. Custom Docker network:**

```hcl
resource "docker_network" "private" {
  name = "${var.container_name}_network"
  
  driver = "bridge"
  
  ipam_config {
    subnet  = "172.20.0.0/16"
    gateway = "172.20.0.1"
  }
}

resource "docker_container" "nginx" {
  count = var.container_count
  
  networks_advanced {
    name         = docker_network.private.name
    ipv4_address = "172.20.0.${count.index + 10}"
  }
}
```

---

## Модуль 2: AWS Provider и Resources (30 минут)

### 🎯 Напоминалка

**AWS Provider:**

```hcl
terraform {
  required_providers {
    aws = {
      source  = "hashicorp/aws"
      version = "~> 5.0"
    }
  }
}

provider "aws" {
  region  = var.aws_region
  profile = var.aws_profile
  
  default_tags {
    tags = {
      Environment = var.environment
      ManagedBy   = "Terraform"
    }
  }
}
```

**Основные AWS Resources:**

```
Compute
├── aws_instance (EC2)
├── aws_launch_template
└── aws_autoscaling_group

Networking
├── aws_vpc
├── aws_subnet
├── aws_internet_gateway
├── aws_route_table
└── aws_security_group

Storage
├── aws_ebs_volume
├── aws_s3_bucket
└── aws_efs_file_system

Database
├── aws_db_instance (RDS)
└── aws_dynamodb_table

Load Balancing
├── aws_lb (ALB/NLB)
├── aws_lb_target_group
└── aws_lb_listener
```

**Data Sources:**

```hcl
data "aws_ami" "ubuntu" {
  most_recent = true
  owners      = ["099720109477"]
  
  filter {
    name   = "name"
    values = ["ubuntu/images/hvm-ssd/ubuntu-jammy-22.04-amd64-server-*"]
  }
}

data "aws_availability_zones" "available" {
  state = "available"
}

data "aws_caller_identity" "current" {}
```

### 💻 Задание

Создай AWS инфраструктуру с VPC, EC2, и S3:

**1. Подготовка:**

```bash
aws configure
aws sts get-caller-identity

mkdir terraform-aws
cd terraform-aws
```

**2. versions.tf:**

```hcl
terraform {
  required_version = ">= 1.6.0"
  
  required_providers {
    aws = {
      source  = "hashicorp/aws"
      version = "~> 5.0"
    }
  }
}
```

**3. provider.tf:**

```hcl
provider "aws" {
  region = var.aws_region
  
  default_tags {
    tags = {
      Environment = var.environment
      ManagedBy   = "Terraform"
      Project     = var.project_name
    }
  }
}
```

**4. variables.tf:**

```hcl
variable "aws_region" {
  description = "AWS region"
  type        = string
  default     = "us-east-1"
}

variable "environment" {
  description = "Environment name"
  type        = string
  default     = "dev"
}

variable "project_name" {
  description = "Project name"
  type        = string
  default     = "terraform-learning"
}

variable "vpc_cidr" {
  description = "VPC CIDR block"
  type        = string
  default     = "10.0.0.0/16"
}

variable "public_subnet_cidrs" {
  description = "Public subnet CIDRs"
  type        = list(string)
  default     = ["10.0.1.0/24", "10.0.2.0/24"]
}

variable "instance_type" {
  description = "EC2 instance type"
  type        = string
  default     = "t3.micro"
}

variable "instance_count" {
  description = "Number of instances"
  type        = number
  default     = 2
}
```

**5. data.tf:**

```hcl
data "aws_availability_zones" "available" {
  state = "available"
}

data "aws_ami" "ubuntu" {
  most_recent = true
  owners      = ["099720109477"]
  
  filter {
    name   = "name"
    values = ["ubuntu/images/hvm-ssd/ubuntu-jammy-22.04-amd64-server-*"]
  }
}

data "aws_caller_identity" "current" {}
```

**6. locals.tf:**

```hcl
locals {
  name_prefix = "${var.project_name}-${var.environment}"
  azs         = slice(data.aws_availability_zones.available.names, 0, 2)
  
  common_tags = {
    Environment = var.environment
    Project     = var.project_name
  }
}
```

**7. vpc.tf:**

```hcl
resource "aws_vpc" "main" {
  cidr_block           = var.vpc_cidr
  enable_dns_hostnames = true
  enable_dns_support   = true
  
  tags = merge(local.common_tags, {
    Name = "${local.name_prefix}-vpc"
  })
}

resource "aws_internet_gateway" "main" {
  vpc_id = aws_vpc.main.id
  
  tags = merge(local.common_tags, {
    Name = "${local.name_prefix}-igw"
  })
}

resource "aws_subnet" "public" {
  count = length(var.public_subnet_cidrs)
  
  vpc_id                  = aws_vpc.main.id
  cidr_block              = var.public_subnet_cidrs[count.index]
  availability_zone       = local.azs[count.index]
  map_public_ip_on_launch = true
  
  tags = merge(local.common_tags, {
    Name = "${local.name_prefix}-public-${local.azs[count.index]}"
  })
}

resource "aws_route_table" "public" {
  vpc_id = aws_vpc.main.id
  
  route {
    cidr_block = "0.0.0.0/0"
    gateway_id = aws_internet_gateway.main.id
  }
  
  tags = merge(local.common_tags, {
    Name = "${local.name_prefix}-public-rt"
  })
}

resource "aws_route_table_association" "public" {
  count = length(aws_subnet.public)
  
  subnet_id      = aws_subnet.public[count.index].id
  route_table_id = aws_route_table.public.id
}
```

**8. security_groups.tf:**

```hcl
resource "aws_security_group" "web" {
  name_prefix = "${local.name_prefix}-web-"
  description = "Security group for web servers"
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
  
  ingress {
    from_port   = 22
    to_port     = 22
    protocol    = "tcp"
    cidr_blocks = ["0.0.0.0/0"]  # В production - свой IP!
  }
  
  egress {
    from_port   = 0
    to_port     = 0
    protocol    = "-1"
    cidr_blocks = ["0.0.0.0/0"]
  }
  
  tags = merge(local.common_tags, {
    Name = "${local.name_prefix}-web-sg"
  })
  
  lifecycle {
    create_before_destroy = true
  }
}
```

**9. ec2.tf:**

```hcl
resource "aws_key_pair" "main" {
  key_name   = "${local.name_prefix}-key"
  public_key = file("~/.ssh/id_rsa.pub")
}

resource "aws_instance" "web" {
  count = var.instance_count
  
  ami           = data.aws_ami.ubuntu.id
  instance_type = var.instance_type
  
  subnet_id              = aws_subnet.public[count.index % length(aws_subnet.public)].id
  vpc_security_group_ids = [aws_security_group.web.id]
  key_name               = aws_key_pair.main.key_name
  
  user_data = base64encode(<<-EOF
              #!/bin/bash
              apt-get update
              apt-get install -y nginx
              echo "<h1>Server ${count.index + 1}</h1>" > /var/www/html/index.html
              systemctl start nginx
              EOF
  )
  
  root_block_device {
    volume_type = "gp3"
    volume_size = 20
    encrypted   = true
  }
  
  tags = merge(local.common_tags, {
    Name = "${local.name_prefix}-web-${count.index + 1}"
  })
}
```

**10. s3.tf:**

```hcl
resource "aws_s3_bucket" "assets" {
  bucket_prefix = "${local.name_prefix}-assets-"
  
  tags = merge(local.common_tags, {
    Name = "${local.name_prefix}-assets"
  })
}

resource "aws_s3_bucket_versioning" "assets" {
  bucket = aws_s3_bucket.assets.id
  
  versioning_configuration {
    status = "Enabled"
  }
}

resource "aws_s3_bucket_server_side_encryption_configuration" "assets" {
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
```

**11. outputs.tf:**

```hcl
output "vpc_id" {
  value = aws_vpc.main.id
}

output "instance_public_ips" {
  value = aws_instance.web[*].public_ip
}

output "instance_urls" {
  value = [
    for ip in aws_instance.web[*].public_ip :
    "http://${ip}"
  ]
}

output "s3_bucket_name" {
  value = aws_s3_bucket.assets.id
}

output "ssh_commands" {
  value = [
    for ip in aws_instance.web[*].public_ip :
    "ssh -i ~/.ssh/id_rsa ubuntu@${ip}"
  ]
}
```

**12. Запуск:**

```bash
terraform init
terraform validate
terraform fmt
terraform plan
terraform apply

# Проверка
terraform output instance_urls
curl $(terraform output -json instance_urls | jq -r '.[0]')

# Cleanup
terraform destroy
```

### 🚀 Бонус

**1. Dynamic blocks для security groups:**

```hcl
variable "ingress_rules" {
  type = list(object({
    from_port   = number
    to_port     = number
    protocol    = string
    cidr_blocks = list(string)
    description = string
  }))
  default = [
    {
      from_port   = 80
      to_port     = 80
      protocol    = "tcp"
      cidr_blocks = ["0.0.0.0/0"]
      description = "HTTP"
    },
    {
      from_port   = 443
      to_port     = 443
      protocol    = "tcp"
      cidr_blocks = ["0.0.0.0/0"]
      description = "HTTPS"
    }
  ]
}

resource "aws_security_group" "dynamic" {
  name_prefix = "${local.name_prefix}-dynamic-"
  vpc_id      = aws_vpc.main.id
  
  dynamic "ingress" {
    for_each = var.ingress_rules
    content {
      description = ingress.value.description
      from_port   = ingress.value.from_port
      to_port     = ingress.value.to_port
      protocol    = ingress.value.protocol
      cidr_blocks = ingress.value.cidr_blocks
    }
  }
}
```

**2. Application Load Balancer:**

```hcl
# alb.tf
resource "aws_lb" "main" {
  name               = "${local.name_prefix}-alb"
  internal           = false
  load_balancer_type = "application"
  security_groups    = [aws_security_group.web.id]
  subnets            = aws_subnet.public[*].id
  
  tags = local.common_tags
}

resource "aws_lb_target_group" "web" {
  name_prefix = "web-"
  port        = 80
  protocol    = "HTTP"
  vpc_id      = aws_vpc.main.id
  
  health_check {
    enabled = true
    path    = "/"
  }
  
  lifecycle {
    create_before_destroy = true
  }
}

resource "aws_lb_target_group_attachment" "web" {
  count = length(aws_instance.web)
  
  target_group_arn = aws_lb_target_group.web.arn
  target_id        = aws_instance.web[count.index].id
}

resource "aws_lb_listener" "http" {
  load_balancer_arn = aws_lb.main.arn
  port              = 80
  protocol          = "HTTP"
  
  default_action {
    type             = "forward"
    target_group_arn = aws_lb_target_group.web.arn
  }
}

output "alb_dns" {
  value = aws_lb.main.dns_name
}
```

---

## Модуль 3: Variables, Outputs и Functions (25 минут)

### 🎯 Напоминалка

**Типы переменных:**

```hcl
# Простые
variable "string_var" {
  type    = string
  default = "value"
}

variable "number_var" {
  type    = number
  default = 42
}

variable "bool_var" {
  type    = bool
  default = true
}

# Коллекции
variable "list_var" {
  type    = list(string)
  default = ["item1", "item2"]
}

variable "map_var" {
  type = map(string)
  default = {
    key1 = "value1"
  }
}

# Объекты
variable "object_var" {
  type = object({
    name = string
    age  = number
  })
}
```

**Variable Validation:**

```hcl
variable "instance_type" {
  type    = string
  default = "t3.micro"
  
  validation {
    condition     = can(regex("^t3\\.", var.instance_type))
    error_message = "Must be t3 family."
  }
}

variable "environment" {
  type = string
  
  validation {
    condition     = contains(["dev", "staging", "prod"], var.environment)
    error_message = "Must be dev, staging, or prod."
  }
}
```

**Terraform Functions:**

```
String Functions
├── format(), join(), split()
├── lower(), upper()
├── trim(), replace()
└── substr()

Collection Functions
├── length(), concat(), merge()
├── slice(), flatten()
├── keys(), values()
└── lookup(), contains()

Numeric Functions
├── max(), min()
├── ceil(), floor()
└── pow()

IP Functions
├── cidrhost()
├── cidrnetmask()
└── cidrsubnet()

Date/Time
├── timestamp()
└── formatdate()
```

### 💻 Задание

**1. Создай продвинутые переменные:**

```hcl
# variables.tf
variable "project_name" {
  description = "Project name"
  type        = string
  
  validation {
    condition     = length(var.project_name) >= 3
    error_message = "Must be at least 3 characters."
  }
}

variable "environment_config" {
  description = "Environment configurations"
  type = map(object({
    instance_count = number
    instance_type  = string
    multi_az       = bool
  }))
  
  default = {
    dev = {
      instance_count = 1
      instance_type  = "t3.micro"
      multi_az       = false
    }
    prod = {
      instance_count = 3
      instance_type  = "t3.medium"
      multi_az       = true
    }
  }
}

variable "subnets" {
  type = list(object({
    name   = string
    cidr   = string
    az     = string
    public = bool
  }))
}
```

**2. Используй locals с functions:**

```hcl
# locals.tf
locals {
  name_prefix = lower("${var.project_name}-${var.environment}")
  
  env_config = var.environment_config[var.environment]
  
  public_subnets = [
    for subnet in var.subnets :
    subnet if subnet.public
  ]
  
  private_subnets = [
    for subnet in var.subnets :
    subnet if !subnet.public
  ]
  
  subnet_map = {
    for subnet in var.subnets :
    subnet.name => subnet.cidr
  }
  
  common_tags = merge(
    var.tags,
    {
      Environment = var.environment
      ManagedBy   = "Terraform"
      CreatedAt   = formatdate("YYYY-MM-DD", timestamp())
    }
  )
  
  # IP calculations
  subnet_info = {
    for subnet in var.subnets :
    subnet.name => {
      network   = cidrhost(subnet.cidr, 0)
      broadcast = cidrhost(subnet.cidr, -1)
      netmask   = cidrnetmask(subnet.cidr)
    }
  }
}
```

**3. Продвинутые outputs:**

```hcl
# outputs.tf
output "environment_config" {
  description = "Active environment config"
  value       = local.env_config
}

output "subnet_details" {
  value = {
    for subnet in var.subnets :
    subnet.name => {
      cidr    = subnet.cidr
      public  = subnet.public
      network = local.subnet_info[subnet.name].network
    }
  }
}

output "public_subnet_cidrs" {
  value = [for s in local.public_subnets : s.cidr]
}

output "resource_count" {
  value = {
    subnets   = length(var.subnets)
    instances = local.env_config.instance_count
  }
}

# Sensitive output
output "sensitive_data" {
  value     = var.api_key
  sensitive = true
}

# Conditional output
output "production_info" {
  value = var.environment == "prod" ? {
    multi_az = local.env_config.multi_az
    ha       = true
  } : null
}
```

**4. terraform.tfvars:**

```hcl
project_name = "myapp"
environment  = "dev"

subnets = [
  {
    name   = "public-1a"
    cidr   = "10.0.1.0/24"
    az     = "us-east-1a"
    public = true
  },
  {
    name   = "private-1a"
    cidr   = "10.0.11.0/24"
    az     = "us-east-1a"
    public = false
  }
]

tags = {
  Owner = "DevOps"
}
```

**5. Тестирование:**

```bash
# Проверить переменные
terraform console
> var.project_name
> local.name_prefix
> local.public_subnets

# Валидация
terraform validate

# Просмотр outputs
terraform output
terraform output -json
```

### 🚀 Бонус

**1. Использование try() и can():**

```hcl
locals {
  # Безопасное получение значения
  custom_domain = try(
    trim(var.custom_domain),
    "${var.project_name}.example.com"
  )
  
  # Проверка валидности
  is_valid_cidr = can(cidrhost(var.vpc_cidr, 0))
  
  # Конвертация типов
  port_number = tonumber(var.port_string)
}
```

**2. Template functions:**

```hcl
locals {
  user_data = templatefile("${path.module}/user_data.sh", {
    instance_name = local.name_prefix
    environment   = var.environment
  })
}
```

---

## Модуль 4: State Management и Backends (30 минут)

### 🎯 Напоминалка

**Terraform State:**

```
State (terraform.tfstate)
├── JSON формат
├── Текущее состояние инфраструктуры
├── Маппинг конфигурации на ресурсы
├── Метаданные и зависимости
└── КРИТИЧНО: НЕ коммитить в Git!
```

**Backend Types:**

```
Remote Backends
├── S3 (AWS) - рекомендуется
├── Azure Blob Storage
├── Google Cloud Storage
├── Terraform Cloud
└── Consul

State Locking
├── DynamoDB (с S3)
├── Встроенное (Terraform Cloud)
└── Consul
```

**Backend Configuration:**

```hcl
# backend.tf
terraform {
  backend "s3" {
    bucket         = "my-terraform-state"
    key            = "project/terraform.tfstate"
    region         = "us-east-1"
    encrypt        = true
    dynamodb_table = "terraform-locks"
    
    # Partial configuration - остальное через -backend-config
  }
}
```

**State Commands:**

```bash
# Просмотр state
terraform state list
terraform state show aws_instance.web

# Перемещение ресурсов
terraform state mv aws_instance.old aws_instance.new

# Удаление из state
terraform state rm aws_instance.web

# Импорт существующих ресурсов
terraform import aws_instance.web i-1234567890

# Обновление state
terraform refresh

# Pull/Push state (осторожно!)
terraform state pull > terraform.tfstate.backup
terraform state push terraform.tfstate
```

### 💻 Задание

**1. Создай S3 backend инфраструктуру:**

```hcl
# bootstrap/main.tf - отдельный проект для backend
terraform {
  required_version = ">= 1.6.0"
  
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

# S3 bucket для state
resource "aws_s3_bucket" "terraform_state" {
  bucket = "my-terraform-state-${data.aws_caller_identity.current.account_id}"
  
  tags = {
    Name        = "Terraform State"
    Environment = "shared"
  }
}

resource "aws_s3_bucket_versioning" "terraform_state" {
  bucket = aws_s3_bucket.terraform_state.id
  
  versioning_configuration {
    status = "Enabled"
  }
}

resource "aws_s3_bucket_server_side_encryption_configuration" "terraform_state" {
  bucket = aws_s3_bucket.terraform_state.id
  
  rule {
    apply_server_side_encryption_by_default {
      sse_algorithm = "AES256"
    }
  }
}

resource "aws_s3_bucket_public_access_block" "terraform_state" {
  bucket = aws_s3_bucket.terraform_state.id
  
  block_public_acls       = true
  block_public_policy     = true
  ignore_public_acls      = true
  restrict_public_buckets = true
}

# DynamoDB для state locking
resource "aws_dynamodb_table" "terraform_locks" {
  name         = "terraform-state-locks"
  billing_mode = "PAY_PER_REQUEST"
  hash_key     = "LockID"
  
  attribute {
    name = "LockID"
    type = "S"
  }
  
  tags = {
    Name        = "Terraform State Locks"
    Environment = "shared"
  }
}

data "aws_caller_identity" "current" {}

output "s3_bucket_name" {
  value = aws_s3_bucket.terraform_state.id
}

output "dynamodb_table_name" {
  value = aws_dynamodb_table.terraform_locks.name
}
```

**2. Запуск bootstrap:**

```bash
cd bootstrap
terraform init
terraform apply

# Запомни bucket name и table name
export TF_STATE_BUCKET=$(terraform output -raw s3_bucket_name)
export TF_STATE_TABLE=$(terraform output -raw dynamodb_table_name)
```

**3. Настрой backend в основном проекте:**

```hcl
# backend.tf
terraform {
  backend "s3" {
    bucket         = "my-terraform-state-123456789012"
    key            = "myapp/dev/terraform.tfstate"
    region         = "us-east-1"
    encrypt        = true
    dynamodb_table = "terraform-state-locks"
  }
}
```

**4. Миграция на remote backend:**

```bash
# Инициализация с миграцией
terraform init -migrate-state

# Проверка
terraform state list
aws s3 ls s3://$TF_STATE_BUCKET/myapp/dev/
```

**5. Backend config через файл (рекомендуется):**

```hcl
# backend.tf - без конкретных значений
terraform {
  backend "s3" {}
}
```

```hcl
# backend-config/dev.tfbackend
bucket         = "my-terraform-state-123456789012"
key            = "myapp/dev/terraform.tfstate"
region         = "us-east-1"
encrypt        = true
dynamodb_table = "terraform-state-locks"
```

```bash
# Инициализация
terraform init -backend-config=backend-config/dev.tfbackend
```

**6. Workspaces с remote backend:**

```bash
# Создание workspace
terraform workspace new staging
terraform workspace new production

# Переключение
terraform workspace select dev

# Список
terraform workspace list

# State файлы создаются автоматически:
# s3://bucket/myapp/env:/dev/terraform.tfstate
# s3://bucket/myapp/env:/staging/terraform.tfstate
# s3://bucket/myapp/env:/production/terraform.tfstate
```

### 🚀 Бонус

**1. State locking проверка:**

```bash
# Terminal 1
terraform plan
# Держи открытым

# Terminal 2
terraform plan
# Должно показать lock error

# Проверка DynamoDB
aws dynamodb scan --table-name terraform-state-locks
```

**2. Import существующих ресурсов:**

```bash
# Найти resource ID
aws ec2 describe-instances

# Создать ресурс в .tf файле
cat >> main.tf <<EOF
resource "aws_instance" "imported" {
  ami           = "ami-xxxxx"
  instance_type = "t3.micro"
}
EOF

# Импорт
terraform import aws_instance.imported i-1234567890abcdef

# Plan покажет только изменения атрибутов
terraform plan
```

**3. State recovery:**

```bash
# Backup state
terraform state pull > terraform.tfstate.backup

# Просмотр версий в S3
aws s3api list-object-versions \
  --bucket $TF_STATE_BUCKET \
  --prefix myapp/dev/terraform.tfstate

# Восстановление предыдущей версии
aws s3api get-object \
  --bucket $TF_STATE_BUCKET \
  --key myapp/dev/terraform.tfstate \
  --version-id <version-id> \
  terraform.tfstate.recovered
```

---

## Модуль 5: Modules - переиспользование кода (30 минут)

### 🎯 Напоминалка

**Modules структура:**

```
modules/
└── vpc/
    ├── main.tf       # Ресурсы модуля
    ├── variables.tf  # Входные параметры
    ├── outputs.tf    # Выходные значения
    ├── versions.tf   # Provider requirements
    └── README.md     # Документация
```

**Использование модулей:**

```hcl
module "vpc" {
  source = "./modules/vpc"
  
  # Входные переменные
  vpc_cidr    = "10.0.0.0/16"
  environment = "dev"
  
  # Outputs из других модулей
  app_port = module.app.port
}

# Доступ к outputs
output "vpc_id" {
  value = module.vpc.vpc_id
}
```

**Module Sources:**

```hcl
# Локальный путь
source = "./modules/vpc"

# Git репозиторий
source = "git::https://github.com/user/repo.git//modules/vpc?ref=v1.0.0"

# Terraform Registry
source = "terraform-aws-modules/vpc/aws"
version = "5.0.0"

# S3
source = "s3::https://s3.amazonaws.com/bucket/vpc.zip"
```

### 💻 Задание

**1. Создай VPC модуль:**

```hcl
# modules/vpc/variables.tf
variable "vpc_name" {
  description = "Name of the VPC"
  type        = string
}

variable "vpc_cidr" {
  description = "CIDR block for VPC"
  type        = string
}

variable "azs" {
  description = "Availability zones"
  type        = list(string)
}

variable "public_subnet_cidrs" {
  description = "Public subnet CIDRs"
  type        = list(string)
}

variable "private_subnet_cidrs" {
  description = "Private subnet CIDRs"
  type        = list(string)
}

variable "enable_nat_gateway" {
  description = "Enable NAT Gateway"
  type        = bool
  default     = false
}

variable "tags" {
  description = "Tags for all resources"
  type        = map(string)
  default     = {}
}
```

```hcl
# modules/vpc/main.tf
resource "aws_vpc" "main" {
  cidr_block           = var.vpc_cidr
  enable_dns_hostnames = true
  enable_dns_support   = true
  
  tags = merge(var.tags, {
    Name = var.vpc_name
  })
}

resource "aws_internet_gateway" "main" {
  vpc_id = aws_vpc.main.id
  
  tags = merge(var.tags, {
    Name = "${var.vpc_name}-igw"
  })
}

resource "aws_subnet" "public" {
  count = length(var.public_subnet_cidrs)
  
  vpc_id                  = aws_vpc.main.id
  cidr_block              = var.public_subnet_cidrs[count.index]
  availability_zone       = var.azs[count.index]
  map_public_ip_on_launch = true
  
  tags = merge(var.tags, {
    Name = "${var.vpc_name}-public-${var.azs[count.index]}"
    Type = "Public"
  })
}

resource "aws_subnet" "private" {
  count = length(var.private_subnet_cidrs)
  
  vpc_id            = aws_vpc.main.id
  cidr_block        = var.private_subnet_cidrs[count.index]
  availability_zone = var.azs[count.index]
  
  tags = merge(var.tags, {
    Name = "${var.vpc_name}-private-${var.azs[count.index]}"
    Type = "Private"
  })
}

resource "aws_eip" "nat" {
  count  = var.enable_nat_gateway ? 1 : 0
  domain = "vpc"
  
  depends_on = [aws_internet_gateway.main]
}

resource "aws_nat_gateway" "main" {
  count = var.enable_nat_gateway ? 1 : 0
  
  allocation_id = aws_eip.nat[0].id
  subnet_id     = aws_subnet.public[0].id
  
  depends_on = [aws_internet_gateway.main]
}

resource "aws_route_table" "public" {
  vpc_id = aws_vpc.main.id
  
  route {
    cidr_block = "0.0.0.0/0"
    gateway_id = aws_internet_gateway.main.id
  }
  
  tags = merge(var.tags, {
    Name = "${var.vpc_name}-public-rt"
  })
}

resource "aws_route_table" "private" {
  vpc_id = aws_vpc.main.id
  
  dynamic "route" {
    for_each = var.enable_nat_gateway ? [1] : []
    content {
      cidr_block     = "0.0.0.0/0"
      nat_gateway_id = aws_nat_gateway.main[0].id
    }
  }
  
  tags = merge(var.tags, {
    Name = "${var.vpc_name}-private-rt"
  })
}

resource "aws_route_table_association" "public" {
  count = length(aws_subnet.public)
  
  subnet_id      = aws_subnet.public[count.index].id
  route_table_id = aws_route_table.public.id
}

resource "aws_route_table_association" "private" {
  count = length(aws_subnet.private)
  
  subnet_id      = aws_subnet.private[count.index].id
  route_table_id = aws_route_table.private.id
}
```

```hcl
# modules/vpc/outputs.tf
output "vpc_id" {
  description = "VPC ID"
  value       = aws_vpc.main.id
}

output "vpc_cidr" {
  description = "VPC CIDR block"
  value       = aws_vpc.main.cidr_block
}

output "public_subnet_ids" {
  description = "Public subnet IDs"
  value       = aws_subnet.public[*].id
}

output "private_subnet_ids" {
  description = "Private subnet IDs"
  value       = aws_subnet.private[*].id
}

output "internet_gateway_id" {
  description = "Internet Gateway ID"
  value       = aws_internet_gateway.main.id
}

output "nat_gateway_id" {
  description = "NAT Gateway ID"
  value       = var.enable_nat_gateway ? aws_nat_gateway.main[0].id : null
}
```

**2. Использование VPC модуля:**

```hcl
# main.tf
module "vpc" {
  source = "./modules/vpc"
  
  vpc_name = "${var.project_name}-${var.environment}"
  vpc_cidr = "10.0.0.0/16"
  
  azs                  = ["us-east-1a", "us-east-1b"]
  public_subnet_cidrs  = ["10.0.1.0/24", "10.0.2.0/24"]
  private_subnet_cidrs = ["10.0.11.0/24", "10.0.12.0/24"]
  
  enable_nat_gateway = var.environment == "production"
  
  tags = local.common_tags
}

# Использование outputs модуля
resource "aws_instance" "web" {
  subnet_id = module.vpc.public_subnet_ids[0]
  # ...
}

output "vpc_id" {
  value = module.vpc.vpc_id
}
```

**3. Создай EC2 модуль:**

```hcl
# modules/ec2/variables.tf
variable "name" {
  type = string
}

variable "instance_count" {
  type    = number
  default = 1
}

variable "instance_type" {
  type = string
}

variable "ami_id" {
  type = string
}

variable "subnet_ids" {
  type = list(string)
}

variable "security_group_ids" {
  type = list(string)
}

variable "user_data" {
  type    = string
  default = ""
}

variable "tags" {
  type    = map(string)
  default = {}
}
```

```hcl
# modules/ec2/main.tf
resource "aws_instance" "main" {
  count = var.instance_count
  
  ami           = var.ami_id
  instance_type = var.instance_type
  
  subnet_id              = var.subnet_ids[count.index % length(var.subnet_ids)]
  vpc_security_group_ids = var.security_group_ids
  
  user_data = var.user_data
  
  root_block_device {
    volume_type = "gp3"
    volume_size = 20
    encrypted   = true
  }
  
  tags = merge(var.tags, {
    Name = "${var.name}-${count.index + 1}"
  })
}
```

```hcl
# modules/ec2/outputs.tf
output "instance_ids" {
  value = aws_instance.main[*].id
}

output "public_ips" {
  value = aws_instance.main[*].public_ip
}

output "private_ips" {
  value = aws_instance.main[*].private_ip
}
```

**4. Композиция модулей:**

```hcl
# main.tf
module "vpc" {
  source = "./modules/vpc"
  # ... конфигурация
}

module "web_servers" {
  source = "./modules/ec2"
  
  name           = "${var.project_name}-web"
  instance_count = var.environment == "production" ? 3 : 1
  instance_type  = var.instance_type
  ami_id         = data.aws_ami.ubuntu.id
  
  subnet_ids         = module.vpc.public_subnet_ids
  security_group_ids = [aws_security_group.web.id]
  
  user_data = file("${path.module}/user_data.sh")
  
  tags = local.common_tags
  
  # Зависимость от VPC
  depends_on = [module.vpc]
}

module "app_servers" {
  source = "./modules/ec2"
  
  name           = "${var.project_name}-app"
  instance_count = 2
  instance_type  = "t3.small"
  ami_id         = data.aws_ami.ubuntu.id
  
  subnet_ids         = module.vpc.private_subnet_ids
  security_group_ids = [aws_security_group.app.id]
  
  tags = local.common_tags
}
```

**5. Terraform Registry модули:**

```hcl
# Использование готового модуля
module "vpc" {
  source  = "terraform-aws-modules/vpc/aws"
  version = "5.0.0"
  
  name = "${var.project_name}-vpc"
  cidr = "10.0.0.0/16"
  
  azs             = ["us-east-1a", "us-east-1b"]
  private_subnets = ["10.0.1.0/24", "10.0.2.0/24"]
  public_subnets  = ["10.0.101.0/24", "10.0.102.0/24"]
  
  enable_nat_gateway = true
  enable_vpn_gateway = false
  
  tags = local.common_tags
}
```

### 🚀 Бонус

**1. Module versioning в Git:**

```bash
# Создай Git repo для модулей
mkdir terraform-modules
cd terraform-modules
git init

# Добавь модуль
mkdir -p modules/vpc
# ... создай файлы

git add .
git commit -m "Add VPC module v1.0.0"
git tag v1.0.0
git push origin main --tags
```

```hcl
# Использование
module "vpc" {
  source = "git::https://github.com/youruser/terraform-modules.git//modules/vpc?ref=v1.0.0"
  # ... параметры
}
```

**2. Module с условной логикой:**

```hcl
# modules/rds/main.tf
resource "aws_db_instance" "main" {
  # Базовые параметры
  identifier     = var.identifier
  engine         = var.engine
  engine_version = var.engine_version
  instance_class = var.instance_class
  
  # Production vs Development
  multi_az               = var.environment == "production"
  backup_retention_period = var.environment == "production" ? 7 : 1
  
  # Условное создание read replica
  replicate_source_db = var.create_replica ? aws_db_instance.main[0].id : null
  
  # ...
}

# Read replica (опционально)
resource "aws_db_instance" "replica" {
  count = var.create_replica ? var.replica_count : 0
  
  identifier          = "${var.identifier}-replica-${count.index + 1}"
  replicate_source_db = aws_db_instance.main.id
  instance_class      = var.replica_instance_class
  
  # ...
}
```

**3. Module composition pattern:**

```hcl
# modules/webapp/main.tf - высокоуровневый модуль
module "vpc" {
  source = "../vpc"
  # ...
}

module "alb" {
  source = "../alb"
  
  vpc_id     = module.vpc.vpc_id
  subnet_ids = module.vpc.public_subnet_ids
  # ...
}

module "asg" {
  source = "../asg"
  
  subnet_ids         = module.vpc.private_subnet_ids
  target_group_arns  = [module.alb.target_group_arn]
  # ...
}

module "rds" {
  source = "../rds"
  
  subnet_ids         = module.vpc.private_subnet_ids
  security_group_ids = [aws_security_group.db.id]
  # ...
}
```

---


## Модуль 6: State Management, Workspaces и Backend Configuration (30 минут)

### 🎯 Напоминалка

**Terraform State - критически важный компонент:**

```
State File (terraform.tfstate)
├── JSON формат
├── Mapping конфигурации → реальные ресурсы
├── Metadata и dependencies
├── Resource attributes
└── ВАЖНО: НЕ коммитить в Git!
```

**Backend Types:**

```
Local Backend (default)
└── terraform.tfstate в текущей директории

Remote Backends
├── S3 + DynamoDB (AWS) - рекомендуется
├── Azure Blob Storage
├── Google Cloud Storage
├── Terraform Cloud/Enterprise
├── Consul
├── etcd
└── PostgreSQL
```

**Workspaces:**

```bash
# Список workspaces
terraform workspace list

# Создать новый
terraform workspace new dev
terraform workspace new staging
terraform workspace new production

# Переключиться
terraform workspace select dev

# Текущий workspace
terraform workspace show

# Удалить (только пустой)
terraform workspace delete dev
```

**Backend Configuration - S3:**

```hcl
# backend.tf
terraform {
  backend "s3" {
    bucket         = "my-terraform-state"
    key            = "project/terraform.tfstate"
    region         = "us-east-1"
    encrypt        = true
    dynamodb_table = "terraform-locks"
    
    # Для state locking
    # DynamoDB table должна иметь partition key "LockID" (String)
  }
}
```

**Создание S3 Backend Infrastructure:**

```hcl
# bootstrap/main.tf
provider "aws" {
  region = "us-east-1"
}

# S3 bucket для state
resource "aws_s3_bucket" "terraform_state" {
  bucket = "my-terraform-state-${data.aws_caller_identity.current.account_id}"
  
  lifecycle {
    prevent_destroy = true  # Защита от случайного удаления
  }
  
  tags = {
    Name        = "Terraform State"
    Environment = "shared"
  }
}

# Versioning для state файла
resource "aws_s3_bucket_versioning" "terraform_state" {
  bucket = aws_s3_bucket.terraform_state.id
  
  versioning_configuration {
    status = "Enabled"
  }
}

# Шифрование
resource "aws_s3_bucket_server_side_encryption_configuration" "terraform_state" {
  bucket = aws_s3_bucket.terraform_state.id
  
  rule {
    apply_server_side_encryption_by_default {
      sse_algorithm = "AES256"
    }
  }
}

# Блокировка публичного доступа
resource "aws_s3_bucket_public_access_block" "terraform_state" {
  bucket = aws_s3_bucket.terraform_state.id
  
  block_public_acls       = true
  block_public_policy     = true
  ignore_public_acls      = true
  restrict_public_buckets = true
}

# DynamoDB для state locking
resource "aws_dynamodb_table" "terraform_locks" {
  name         = "terraform-state-locks"
  billing_mode = "PAY_PER_REQUEST"
  hash_key     = "LockID"
  
  attribute {
    name = "LockID"
    type = "S"
  }
  
  tags = {
    Name        = "Terraform State Locks"
    Environment = "shared"
  }
}

data "aws_caller_identity" "current" {}

output "s3_bucket_name" {
  value = aws_s3_bucket.terraform_state.id
}

output "dynamodb_table_name" {
  value = aws_dynamodb_table.terraform_locks.name
}
```

**Partial Backend Configuration:**

```hcl
# backend.tf - без credentials
terraform {
  backend "s3" {
    # Конкретные значения передаются через -backend-config
  }
}
```

```bash
# backend-config/dev.tfbackend
bucket         = "my-terraform-state-123456789012"
key            = "project/dev/terraform.tfstate"
region         = "us-east-1"
encrypt        = true
dynamodb_table = "terraform-state-locks"

# Использование
terraform init -backend-config=backend-config/dev.tfbackend
```

**State Commands:**

```bash
# Просмотр state
terraform state list
terraform state show aws_instance.web

# Переместить ресурс
terraform state mv aws_instance.old aws_instance.new

# Переместить ресурс в другой state (при разделении проектов)
terraform state mv -state-out=../other-project/terraform.tfstate aws_instance.web aws_instance.web

# Удалить из state (ресурс останется в облаке)
terraform state rm aws_instance.web

# Импорт существующего ресурса
terraform import aws_instance.web i-1234567890abcdef

# Pull state (скачать)
terraform state pull > terraform.tfstate.backup

# Push state (загрузить) - ОСТОРОЖНО!
terraform state push terraform.tfstate

# Обновить state (refresh)
terraform refresh

# Replace provider (при миграции)
terraform state replace-provider hashicorp/aws registry.example.com/aws
```

**Workspaces с переменными:**

```hcl
# variables.tf
variable "instance_count" {
  type = map(number)
  default = {
    dev     = 1
    staging = 2
    prod    = 3
  }
}

# main.tf
resource "aws_instance" "web" {
  count = var.instance_count[terraform.workspace]
  
  ami           = data.aws_ami.ubuntu.id
  instance_type = terraform.workspace == "prod" ? "t3.medium" : "t3.micro"
  
  tags = {
    Name        = "web-${terraform.workspace}-${count.index + 1}"
    Environment = terraform.workspace
  }
}

# outputs.tf
output "workspace_info" {
  value = {
    workspace = terraform.workspace
    instances = aws_instance.web[*].id
    count     = length(aws_instance.web)
  }
}
```

### 💻 Задание

Настрой production-ready state management:

**1. Создай bootstrap проект:**

```bash
mkdir terraform-state-bootstrap
cd terraform-state-bootstrap
```

**2. bootstrap/provider.tf:**

```hcl
terraform {
  required_version = ">= 1.6.0"
  
  required_providers {
    aws = {
      source  = "hashicorp/aws"
      version = "~> 5.0"
    }
    random = {
      source  = "hashicorp/random"
      version = "~> 3.5"
    }
  }
}

provider "aws" {
  region = var.aws_region
  
  default_tags {
    tags = {
      ManagedBy = "Terraform"
      Purpose   = "State Management"
    }
  }
}
```

**3. bootstrap/variables.tf:**

```hcl
variable "aws_region" {
  description = "AWS region"
  type        = string
  default     = "us-east-1"
}

variable "project_name" {
  description = "Project name for state bucket"
  type        = string
  default     = "myproject"
}

variable "environment" {
  description = "Environment"
  type        = string
  default     = "shared"
}
```

**4. bootstrap/main.tf:**

```hcl
# Random suffix для уникальности
resource "random_id" "suffix" {
  byte_length = 4
}

locals {
  bucket_name = "${var.project_name}-terraform-state-${random_id.suffix.hex}"
  table_name  = "${var.project_name}-terraform-locks"
}

# S3 bucket
resource "aws_s3_bucket" "terraform_state" {
  bucket = local.bucket_name
  
  lifecycle {
    prevent_destroy = true
  }
  
  tags = {
    Name        = "Terraform State"
    Environment = var.environment
  }
}

# Versioning
resource "aws_s3_bucket_versioning" "terraform_state" {
  bucket = aws_s3_bucket.terraform_state.id
  
  versioning_configuration {
    status = "Enabled"
  }
}

# Server-side encryption
resource "aws_s3_bucket_server_side_encryption_configuration" "terraform_state" {
  bucket = aws_s3_bucket.terraform_state.id
  
  rule {
    apply_server_side_encryption_by_default {
      sse_algorithm = "AES256"
    }
  }
}

# Block public access
resource "aws_s3_bucket_public_access_block" "terraform_state" {
  bucket = aws_s3_bucket.terraform_state.id
  
  block_public_acls       = true
  block_public_policy     = true
  ignore_public_acls      = true
  restrict_public_buckets = true
}

# Lifecycle policy для старых версий
resource "aws_s3_bucket_lifecycle_configuration" "terraform_state" {
  bucket = aws_s3_bucket.terraform_state.id
  
  rule {
    id     = "expire-old-versions"
    status = "Enabled"
    
    noncurrent_version_expiration {
      noncurrent_days = 90
    }
  }
}

# DynamoDB table для locking
resource "aws_dynamodb_table" "terraform_locks" {
  name         = local.table_name
  billing_mode = "PAY_PER_REQUEST"
  hash_key     = "LockID"
  
  attribute {
    name = "LockID"
    type = "S"
  }
  
  point_in_time_recovery {
    enabled = true
  }
  
  tags = {
    Name        = "Terraform State Locks"
    Environment = var.environment
  }
}

# IAM policy для state access
resource "aws_iam_policy" "terraform_state_access" {
  name        = "${var.project_name}-terraform-state-access"
  description = "Policy for Terraform state access"
  
  policy = jsonencode({
    Version = "2012-10-17"
    Statement = [
      {
        Effect = "Allow"
        Action = [
          "s3:ListBucket",
          "s3:GetBucketVersioning"
        ]
        Resource = aws_s3_bucket.terraform_state.arn
      },
      {
        Effect = "Allow"
        Action = [
          "s3:GetObject",
          "s3:PutObject",
          "s3:DeleteObject"
        ]
        Resource = "${aws_s3_bucket.terraform_state.arn}/*"
      },
      {
        Effect = "Allow"
        Action = [
          "dynamodb:GetItem",
          "dynamodb:PutItem",
          "dynamodb:DeleteItem"
        ]
        Resource = aws_dynamodb_table.terraform_locks.arn
      }
    ]
  })
}
```

**5. bootstrap/outputs.tf:**

```hcl
output "s3_bucket_name" {
  description = "S3 bucket name for Terraform state"
  value       = aws_s3_bucket.terraform_state.id
}

output "s3_bucket_arn" {
  description = "S3 bucket ARN"
  value       = aws_s3_bucket.terraform_state.arn
}

output "dynamodb_table_name" {
  description = "DynamoDB table name for state locking"
  value       = aws_dynamodb_table.terraform_locks.name
}

output "dynamodb_table_arn" {
  description = "DynamoDB table ARN"
  value       = aws_dynamodb_table.terraform_locks.arn
}

output "iam_policy_arn" {
  description = "IAM policy ARN for state access"
  value       = aws_iam_policy.terraform_state_access.arn
}

output "backend_config" {
  description = "Backend configuration for other projects"
  value = {
    bucket         = aws_s3_bucket.terraform_state.id
    region         = var.aws_region
    dynamodb_table = aws_dynamodb_table.terraform_locks.name
    encrypt        = true
  }
}
```

**6. Запуск bootstrap:**

```bash
cd bootstrap
terraform init
terraform plan
terraform apply

# Сохрани outputs
terraform output -json > ../backend-config.json
```

**7. Создай основной проект с remote backend:**

```bash
cd ..
mkdir main-project
cd main-project
```

**8. main-project/backend.tf:**

```hcl
terraform {
  backend "s3" {
    # Значения из bootstrap outputs
    # bucket, region, dynamodb_table будут заданы через backend-config
  }
}
```

**9. main-project/backend-configs/dev.tfbackend:**

```hcl
# Используй значения из bootstrap outputs
bucket         = "myproject-terraform-state-abc12345"  # Из bootstrap output
key            = "main-project/dev/terraform.tfstate"
region         = "us-east-1"
encrypt        = true
dynamodb_table = "myproject-terraform-locks"  # Из bootstrap output
```

**10. main-project/backend-configs/staging.tfbackend:**

```hcl
bucket         = "myproject-terraform-state-abc12345"
key            = "main-project/staging/terraform.tfstate"
region         = "us-east-1"
encrypt        = true
dynamodb_table = "myproject-terraform-locks"
```

**11. main-project/backend-configs/prod.tfbackend:**

```hcl
bucket         = "myproject-terraform-state-abc12345"
key            = "main-project/prod/terraform.tfstate"
region         = "us-east-1"
encrypt        = true
dynamodb_table = "myproject-terraform-locks"
```

**12. main-project/main.tf (пример):**

```hcl
terraform {
  required_version = ">= 1.6.0"
  
  required_providers {
    aws = {
      source  = "hashicorp/aws"
      version = "~> 5.0"
    }
  }
}

provider "aws" {
  region = var.aws_region
  
  default_tags {
    tags = {
      Environment = terraform.workspace
      ManagedBy   = "Terraform"
      Project     = var.project_name
    }
  }
}

# Data source для account info
data "aws_caller_identity" "current" {}

# Пример ресурсов
resource "aws_vpc" "main" {
  cidr_block           = var.vpc_cidr
  enable_dns_hostnames = true
  enable_dns_support   = true
  
  tags = {
    Name = "${var.project_name}-${terraform.workspace}-vpc"
  }
}

resource "aws_instance" "example" {
  count = var.instance_counts[terraform.workspace]
  
  ami           = data.aws_ami.ubuntu.id
  instance_type = var.instance_types[terraform.workspace]
  
  tags = {
    Name = "${var.project_name}-${terraform.workspace}-${count.index + 1}"
  }
}

data "aws_ami" "ubuntu" {
  most_recent = true
  owners      = ["099720109477"]
  
  filter {
    name   = "name"
    values = ["ubuntu/images/hvm-ssd/ubuntu-jammy-22.04-amd64-server-*"]
  }
}
```

**13. main-project/variables.tf:**

```hcl
variable "aws_region" {
  description = "AWS region"
  type        = string
  default     = "us-east-1"
}

variable "project_name" {
  description = "Project name"
  type        = string
  default     = "myproject"
}

variable "vpc_cidr" {
  description = "VPC CIDR block"
  type        = string
  default     = "10.0.0.0/16"
}

variable "instance_counts" {
  description = "Number of instances per environment"
  type        = map(number)
  default = {
    dev     = 1
    staging = 2
    prod    = 3
  }
}

variable "instance_types" {
  description = "Instance types per environment"
  type        = map(string)
  default = {
    dev     = "t3.micro"
    staging = "t3.small"
    prod    = "t3.medium"
  }
}
```

**14. Инициализация с разными backend:**

```bash
# Dev environment
terraform init -backend-config=backend-configs/dev.tfbackend
terraform workspace new dev || terraform workspace select dev
terraform plan
terraform apply

# Staging environment
terraform init -reconfigure -backend-config=backend-configs/staging.tfbackend
terraform workspace new staging || terraform workspace select staging
terraform plan
terraform apply

# Production environment
terraform init -reconfigure -backend-config=backend-configs/prod.tfbackend
terraform workspace new prod || terraform workspace select prod
terraform plan
terraform apply
```

**15. Создай helper script:**

```bash
#!/bin/bash
# switch-env.sh

set -e

ENV=$1

if [ -z "$ENV" ]; then
  echo "Usage: $0 <dev|staging|prod>"
  exit 1
fi

if [ ! -f "backend-configs/${ENV}.tfbackend" ]; then
  echo "Backend config not found: backend-configs/${ENV}.tfbackend"
  exit 1
fi

echo "Switching to ${ENV} environment..."

terraform init -reconfigure -backend-config=backend-configs/${ENV}.tfbackend

terraform workspace select ${ENV} 2>/dev/null || terraform workspace new ${ENV}

echo "Current workspace: $(terraform workspace show)"
echo "Ready to run terraform commands for ${ENV}"
```

```bash
chmod +x switch-env.sh

# Использование
./switch-env.sh dev
terraform plan

./switch-env.sh prod
terraform plan
```

### 🚀 Бонус (новое)

**1. State Import для существующей инфраструктуры:**

```bash
# Найти существующий ресурс
aws ec2 describe-instances --filters "Name=tag:Name,Values=my-server"

# Создать ресурс в конфигурации
cat >> main.tf <<EOF
resource "aws_instance" "imported" {
  ami           = "ami-0c55b159cbfafe1f0"
  instance_type = "t3.micro"
  
  tags = {
    Name = "my-server"
  }
}
EOF

# Импортировать
terraform import aws_instance.imported i-1234567890abcdef

# План покажет только изменения атрибутов
terraform plan
```

**2. State Locking Troubleshooting:**

```bash
# Проверить lock
aws dynamodb get-item \
  --table-name terraform-state-locks \
  --key '{"LockID": {"S": "myproject-terraform-state-abc12345/main-project/dev/terraform.tfstate-md5"}}'

# Если lock висит (после crash), force-unlock
terraform force-unlock <lock-id>

# Или удалить из DynamoDB
aws dynamodb delete-item \
  --table-name terraform-state-locks \
  --key '{"LockID": {"S": "<lock-id>"}}'
```

**3. State Migration между backends:**

```bash
# Из local в S3
# 1. Текущая конфигурация без backend (local)
terraform init

# 2. Добавь backend конфигурацию
cat >> backend.tf <<EOF
terraform {
  backend "s3" {
    bucket         = "my-state-bucket"
    key            = "project/terraform.tfstate"
    region         = "us-east-1"
    dynamodb_table = "terraform-locks"
    encrypt        = true
  }
}
EOF

# 3. Мигрируй state
terraform init -migrate-state

# Terraform спросит подтверждение и перенесет state
```

**4. State Backup и Recovery:**

```bash
# Backup state локально
terraform state pull > terraform.tfstate.backup

# Список версий в S3
aws s3api list-object-versions \
  --bucket my-state-bucket \
  --prefix project/terraform.tfstate

# Восстановить конкретную версию
aws s3api get-object \
  --bucket my-state-bucket \
  --key project/terraform.tfstate \
  --version-id <version-id> \
  terraform.tfstate.recovered

# Push восстановленного state (ОСТОРОЖНО!)
terraform state push terraform.tfstate.recovered
```

**5. State Split (разделение большого проекта):**

```bash
# Создать новый проект
mkdir ../networking
cd ../networking

# Переместить ресурсы
terraform state mv \
  -state=../main-project/terraform.tfstate \
  -state-out=terraform.tfstate \
  aws_vpc.main \
  aws_vpc.main

# Теперь два независимых state файла
```

---

## Модуль 7: Data Sources и External Data (25 минут)

### 🎯 Напоминалка

**Data Sources - чтение существующей информации:**

```hcl
# Синтаксис
data "provider_type" "name" {
  # Фильтры для поиска
}

# Использование
resource "..." "..." {
  some_attribute = data.provider_type.name.attribute
}
```

**Основные AWS Data Sources:**

```hcl
# AMI
data "aws_ami" "ubuntu" {
  most_recent = true
  owners      = ["099720109477"]  # Canonical
  
  filter {
    name   = "name"
    values = ["ubuntu/images/hvm-ssd/ubuntu-jammy-22.04-amd64-server-*"]
  }
  
  filter {
    name   = "virtualization-type"
    values = ["hvm"]
  }
}

# VPC
data "aws_vpc" "default" {
  default = true
}

data "aws_vpc" "selected" {
  filter {
    name   = "tag:Name"
    values = ["my-vpc"]
  }
}

# Subnets
data "aws_subnets" "public" {
  filter {
    name   = "vpc-id"
    values = [data.aws_vpc.selected.id]
  }
  
  filter {
    name   = "tag:Type"
    values = ["public"]
  }
}

# Availability Zones
data "aws_availability_zones" "available" {
  state = "available"
  
  filter {
    name   = "opt-in-status"
    values = ["opt-in-not-required"]
  }
}

# Caller Identity (current account)
data "aws_caller_identity" "current" {}

# Region
data "aws_region" "current" {}

# Route53 Zone
data "aws_route53_zone" "selected" {
  name         = "example.com."
  private_zone = false
}

# Security Group
data "aws_security_group" "default" {
  vpc_id = data.aws_vpc.default.id
  
  filter {
    name   = "group-name"
    values = ["default"]
  }
}

# IAM Policy Document
data "aws_iam_policy_document" "assume_role" {
  statement {
    actions = ["sts:AssumeRole"]
    
    principals {
      type        = "Service"
      identifiers = ["ec2.amazonaws.com"]
    }
  }
}

# SSM Parameter
data "aws_ssm_parameter" "db_password" {
  name = "/myapp/database/password"
}

# Secrets Manager
data "aws_secretsmanager_secret_version" "db_creds" {
  secret_id = "myapp/database"
}
```

**External Data Source:**

```hcl
# Запуск внешнего script
data "external" "example" {
  program = ["python3", "${path.module}/scripts/get_data.py"]
  
  query = {
    # Input для script
    environment = var.environment
    region      = var.aws_region
  }
}

# Script должен вернуть JSON
# {"result": "value", "another_key": "another_value"}

# Использование
resource "aws_instance" "example" {
  # ...
  user_data = data.external.example.result.result
}
```

**HTTP Data Source:**

```hcl
data "http" "myip" {
  url = "https://ifconfig.me/ip"
}

data "http" "example_api" {
  url = "https://api.example.com/data"
  
  request_headers = {
    Accept        = "application/json"
    Authorization = "Bearer ${var.api_token}"
  }
}

locals {
  my_ip   = chomp(data.http.myip.response_body)
  api_data = jsondecode(data.http.example_api.response_body)
}
```

**Template Data Sources:**

```hcl
# Template file
data "template_file" "user_data" {
  template = file("${path.module}/templates/user_data.sh")
  
  vars = {
    environment = var.environment
    app_version = var.app_version
    db_host     = aws_db_instance.main.endpoint
  }
}

# Cloudinit config
data "cloudinit_config" "server" {
  gzip          = true
  base64_encode = true
  
  part {
    content_type = "text/cloud-config"
    content      = templatefile("${path.module}/cloud-config.yaml", {
      hostname = "server-${count.index}"
    })
  }
  
  part {
    content_type = "text/x-shellscript"
    content      = data.template_file.user_data.rendered
  }
}
```

### 💻 Задание

Создай проект использующий data sources для динамической конфигурации:

**1. Структура проекта:**

```bash
mkdir terraform-data-sources
cd terraform-data-sources
mkdir scripts templates
```

**2. provider.tf:**

```hcl
terraform {
  required_version = ">= 1.6.0"
  
  required_providers {
    aws = {
      source  = "hashicorp/aws"
      version = "~> 5.0"
    }
    http = {
      source  = "hashicorp/http"
      version = "~> 3.4"
    }
    external = {
      source  = "hashicorp/external"
      version = "~> 2.3"
    }
  }
}

provider "aws" {
  region = var.aws_region
}
```

**3. data-sources.tf:**

```hcl
# Current account and region
data "aws_caller_identity" "current" {}
data "aws_region" "current" {}

# Get my current IP
data "http" "myip" {
  url = "https://ifconfig.me/ip"
}

locals {
  my_ip       = chomp(data.http.myip.response_body)
  account_id  = data.aws_caller_identity.current.account_id
  region_name = data.aws_region.current.name
}

# Available AZs
data "aws_availability_zones" "available" {
  state = "available"
}

# Latest Ubuntu AMI
data "aws_ami" "ubuntu" {
  most_recent = true
  owners      = ["099720109477"]
  
  filter {
    name   = "name"
    values = ["ubuntu/images/hvm-ssd/ubuntu-jammy-22.04-amd64-server-*"]
  }
  
  filter {
    name   = "virtualization-type"
    values = ["hvm"]
  }
  
  filter {
    name   = "architecture"
    values = ["x86_64"]
  }
}

# Latest Amazon Linux 2023 AMI
data "aws_ami" "amazon_linux" {
  most_recent = true
  owners      = ["amazon"]
  
  filter {
    name   = "name"
    values = ["al2023-ami-*-x86_64"]
  }
}

# Find existing VPC (или создай новый)
data "aws_vpc" "default" {
  default = true
}

# Get subnets
data "aws_subnets" "default" {
  filter {
    name   = "vpc-id"
    values = [data.aws_vpc.default.id]
  }
}

# IAM policy document
data "aws_iam_policy_document" "ec2_assume_role" {
  statement {
    actions = ["sts:AssumeRole"]
    
    principals {
      type        = "Service"
      identifiers = ["ec2.amazonaws.com"]
    }
  }
}

data "aws_iam_policy_document" "s3_read_only" {
  statement {
    actions = [
      "s3:GetObject",
      "s3:ListBucket"
    ]
    
    resources = [
      "arn:aws:s3:::${var.bucket_name}",
      "arn:aws:s3:::${var.bucket_name}/*"
    ]
  }
}

# External data - получение данных из script
data "external" "environment_info" {
  program = ["python3", "${path.module}/scripts/get_env_info.py"]
  
  query = {
    environment = var.environment
    region      = data.aws_region.current.name
  }
}
```

**4. scripts/get_env_info.py:**

```python
#!/usr/bin/env python3
import json
import sys
import socket

def get_environment_info(environment, region):
    """
    Получить информацию об окружении
    Должен вернуть JSON в stdout
    """
    hostname = socket.gethostname()
    
    # Пример логики для разных environment
    config = {
        "dev": {
            "instance_type": "t3.micro",
            "instance_count": "1",
            "monitoring_enabled": "false"
        },
        "staging": {
            "instance_type": "t3.small",
            "instance_count": "2",
            "monitoring_enabled": "true"
        },
        "prod": {
            "instance_type": "t3.medium",
            "instance_count": "3",
            "monitoring_enabled": "true"
        }
    }
    
    env_config = config.get(environment, config["dev"])
    
    result = {
        "hostname": hostname,
        "environment": environment,
        "region": region,
        **env_config
    }
    
    return result

if __name__ == "__main__":
    # Читаем query из stdin
    input_data = json.load(sys.stdin)
    environment = input_data.get("environment", "dev")
    region = input_data.get("region", "us-east-1")
    
    # Получаем результат
    result = get_environment_info(environment, region)
	# Выводим JSON в stdout
	print(json.dumps(result))
```

````

```bash
chmod +x scripts/get_env_info.py
````

**5. templates/user_data.sh:**

```bash
#!/bin/bash
set -e

# Logging
exec > >(tee /var/log/user-data.log)
exec 2>&1

echo "Starting user data script..."
echo "Environment: ${environment}"
echo "Region: ${region}"
echo "Instance Type: ${instance_type}"

# Update system
apt-get update
apt-get upgrade -y

# Install packages
apt-get install -y \
    nginx \
    python3-pip \
    awscli \
    jq

# Create info page
cat > /var/www/html/index.html <<EOF
<!DOCTYPE html>
<html>
<head>
    <title>Server Info</title>
    <style>
        body { font-family: Arial; margin: 40px; }
        .info { background: #f0f0f0; padding: 20px; border-radius: 5px; }
    </style>
</head>
<body>
    <h1>Server Information</h1>
    <div class="info">
        <p><strong>Environment:</strong> ${environment}</p>
        <p><strong>Region:</strong> ${region}</p>
        <p><strong>Instance Type:</strong> ${instance_type}</p>
        <p><strong>AMI ID:</strong> ${ami_id}</p>
        <p><strong>Hostname:</strong> $(hostname)</p>
        <p><strong>Private IP:</strong> $(hostname -I)</p>
    </div>
</body>
</html>
EOF

# Start nginx
systemctl enable nginx
systemctl start nginx

echo "User data script completed!"
```

**6. main.tf:**

```hcl
locals {
  name_prefix = "${var.project_name}-${var.environment}"
  
  # Данные из external data source
  instance_type    = data.external.environment_info.result.instance_type
  instance_count   = tonumber(data.external.environment_info.result.instance_count)
  monitoring       = tobool(data.external.environment_info.result.monitoring_enabled)
  
  # Выбор AMI на основе переменной
  ami_id = var.use_amazon_linux ? data.aws_ami.amazon_linux.id : data.aws_ami.ubuntu.id
  
  common_tags = {
    Environment = var.environment
    Project     = var.project_name
    ManagedBy   = "Terraform"
    Region      = data.aws_region.current.name
    AccountId   = data.aws_caller_identity.current.account_id
  }
}

# User data template
data "template_file" "user_data" {
  template = file("${path.module}/templates/user_data.sh")
  
  vars = {
    environment   = var.environment
    region        = data.aws_region.current.name
    instance_type = local.instance_type
    ami_id        = local.ami_id
  }
}

# Security Group
resource "aws_security_group" "web" {
  name_prefix = "${local.name_prefix}-web-"
  description = "Security group for web servers"
  vpc_id      = data.aws_vpc.default.id
  
  ingress {
    description = "HTTP from my IP"
    from_port   = 80
    to_port     = 80
    protocol    = "tcp"
    cidr_blocks = ["${local.my_ip}/32"]
  }
  
  ingress {
    description = "HTTPS from my IP"
    from_port   = 443
    to_port     = 443
    protocol    = "tcp"
    cidr_blocks = ["${local.my_ip}/32"]
  }
  
  ingress {
    description = "SSH from my IP"
    from_port   = 22
    to_port     = 22
    protocol    = "tcp"
    cidr_blocks = ["${local.my_ip}/32"]
  }
  
  egress {
    from_port   = 0
    to_port     = 0
    protocol    = "-1"
    cidr_blocks = ["0.0.0.0/0"]
  }
  
  tags = merge(local.common_tags, {
    Name = "${local.name_prefix}-web-sg"
  })
  
  lifecycle {
    create_before_destroy = true
  }
}

# IAM Role
resource "aws_iam_role" "instance" {
  name_prefix        = "${local.name_prefix}-instance-"
  assume_role_policy = data.aws_iam_policy_document.ec2_assume_role.json
  
  tags = local.common_tags
}

resource "aws_iam_role_policy" "s3_read" {
  name_prefix = "${local.name_prefix}-s3-read-"
  role        = aws_iam_role.instance.id
  policy      = data.aws_iam_policy_document.s3_read_only.json
}

resource "aws_iam_instance_profile" "instance" {
  name_prefix = "${local.name_prefix}-instance-"
  role        = aws_iam_role.instance.name
}

# EC2 Instances
resource "aws_instance" "web" {
  count = local.instance_count
  
  ami           = local.ami_id
  instance_type = local.instance_type
  
  subnet_id                   = element(data.aws_subnets.default.ids, count.index)
  vpc_security_group_ids      = [aws_security_group.web.id]
  iam_instance_profile        = aws_iam_instance_profile.instance.name
  associate_public_ip_address = true
  
  user_data = data.template_file.user_data.rendered
  
  monitoring = local.monitoring
  
  root_block_device {
    volume_type = "gp3"
    volume_size = var.environment == "prod" ? 30 : 20
    encrypted   = true
  }
  
  tags = merge(local.common_tags, {
    Name = "${local.name_prefix}-web-${count.index + 1}"
  })
  
  lifecycle {
    create_before_destroy = true
  }
}
```

**7. variables.tf:**

```hcl
variable "aws_region" {
  description = "AWS region"
  type        = string
  default     = "us-east-1"
}

variable "project_name" {
  description = "Project name"
  type        = string
  default     = "datasource-demo"
}

variable "environment" {
  description = "Environment name"
  type        = string
  default     = "dev"
  
  validation {
    condition     = contains(["dev", "staging", "prod"], var.environment)
    error_message = "Environment must be dev, staging, or prod."
  }
}

variable "bucket_name" {
  description = "S3 bucket name for IAM policy"
  type        = string
  default     = "my-app-bucket"
}

variable "use_amazon_linux" {
  description = "Use Amazon Linux instead of Ubuntu"
  type        = bool
  default     = false
}
```

**8. outputs.tf:**

```hcl
output "account_info" {
  description = "AWS account information"
  value = {
    account_id = data.aws_caller_identity.current.account_id
    region     = data.aws_region.current.name
    arn        = data.aws_caller_identity.current.arn
  }
}

output "my_ip" {
  description = "Your current IP address"
  value       = local.my_ip
}

output "ami_info" {
  description = "Selected AMI information"
  value = {
    id           = local.ami_id
    name         = var.use_amazon_linux ? data.aws_ami.amazon_linux.name : data.aws_ami.ubuntu.name
    architecture = var.use_amazon_linux ? data.aws_ami.amazon_linux.architecture : data.aws_ami.ubuntu.architecture
  }
}

output "availability_zones" {
  description = "Available AZs"
  value       = data.aws_availability_zones.available.names
}

output "environment_config" {
  description = "Environment configuration from external data"
  value       = data.external.environment_info.result
}

output "instance_urls" {
  description = "URLs to access instances"
  value = [
    for instance in aws_instance.web :
    "http://${instance.public_ip}"
  ]
}

output "instance_ids" {
  description = "Instance IDs"
  value       = aws_instance.web[*].id
}
```

**9. Запуск:**

```bash
# Dev environment
terraform init
terraform plan -var="environment=dev"
terraform apply -var="environment=dev"

# Staging
terraform apply -var="environment=staging"

# Production
terraform apply -var="environment=prod"

# С Amazon Linux
terraform apply -var="environment=prod" -var="use_amazon_linux=true"
```

### 🚀 Бонус (новое)

**1. Data source с условной логикой:**

```hcl
# Найти VPC по тегу, или использовать default
data "aws_vpc" "selected" {
  count = var.vpc_name != "" ? 1 : 0
  
  filter {
    name   = "tag:Name"
    values = [var.vpc_name]
  }
}

data "aws_vpc" "default" {
  count   = var.vpc_name == "" ? 1 : 0
  default = true
}

locals {
  vpc_id = var.vpc_name != "" ? data.aws_vpc.selected[0].id : data.aws_vpc.default[0].id
}
```

**2. Dynamic data sources в цикле:**

```hcl
# Получить информацию о нескольких AMI
data "aws_ami" "selected" {
  for_each = toset(var.ami_names)
  
  most_recent = true
  owners      = ["self"]
  
  filter {
    name   = "name"
    values = [each.value]
  }
}

# Использование
resource "aws_instance" "example" {
  for_each = data.aws_ami.selected
  
  ami           = each.value.id
  instance_type = "t3.micro"
  
  tags = {
    Name    = each.key
    AMI_ID  = each.value.id
  }
}
```

**3. Advanced external data source:**

```python
#!/usr/bin/env python3
# scripts/get_ssl_cert_expiry.py
import json
import sys
import ssl
import socket
from datetime import datetime

def get_ssl_expiry(hostname, port=443):
    """Получить дату истечения SSL сертификата"""
    try:
        context = ssl.create_default_context()
        with socket.create_connection((hostname, port), timeout=5) as sock:
            with context.wrap_socket(sock, server_hostname=hostname) as ssock:
                cert = ssock.getpeercert()
                expiry_date = datetime.strptime(cert['notAfter'], '%b %d %H:%M:%S %Y %GMT')
                days_left = (expiry_date - datetime.now()).days
                
                return {
                    "hostname": hostname,
                    "expiry_date": cert['notAfter'],
                    "days_remaining": str(days_left),
                    "is_valid": "true" if days_left > 0 else "false"
                }
    except Exception as e:
        return {
            "hostname": hostname,
            "error": str(e),
            "is_valid": "false",
            "days_remaining": "0"
        }

if __name__ == "__main__":
    input_data = json.load(sys.stdin)
    hostname = input_data.get("hostname")
    port = int(input_data.get("port", 443))
    
    result = get_ssl_expiry(hostname, port)
    print(json.dumps(result))
```

```hcl
# Использование
data "external" "ssl_check" {
  program = ["python3", "${path.module}/scripts/get_ssl_cert_expiry.py"]
  
  query = {
    hostname = "example.com"
    port     = "443"
  }
}

resource "aws_cloudwatch_metric_alarm" "ssl_expiry" {
  count = data.external.ssl_check.result.days_remaining < 30 ? 1 : 0
  
  alarm_name          = "ssl-cert-expiring"
  comparison_operator = "LessThanThreshold"
  evaluation_periods  = "1"
  metric_name         = "SSLCertDaysRemaining"
  namespace           = "CustomMetrics"
  period              = "86400"
  statistic           = "Average"
  threshold           = "30"
  alarm_description   = "SSL cert expires in ${data.external.ssl_check.result.days_remaining} days"
}
```

---

## Модуль 8: Provisioners, null_resource и local-exec (25 минут)

### 🎯 Напоминалка

**Provisioners - выполнение действий на ресурсах:**

```
Provisioners (последний resort!)
├── file - копирование файлов
├── remote-exec - выполнение команд на удаленном хосте
├── local-exec - выполнение команд локально
└── Используй только когда нет альтернативы!

Альтернативы (предпочтительно):
├── user_data / cloud-init
├── Configuration management (Ansible, Chef, Puppet)
├── Packer для AMI
└── Container images
```

**⚠️ Важно: Provisioners - это last resort!**

Terraform рекомендует избегать provisioners когда возможно, потому что:

- Нарушают декларативную модель
- Сложнее отслеживать изменения
- Могут привести к неконсистентному state
- Усложняют rollback

**File Provisioner:**

```hcl
resource "aws_instance" "web" {
  ami           = data.aws_ami.ubuntu.id
  instance_type = "t3.micro"
  
  # Connection для provisioners
  connection {
    type        = "ssh"
    user        = "ubuntu"
    private_key = file("~/.ssh/id_rsa")
    host        = self.public_ip
  }
  
  # Копировать файл
  provisioner "file" {
    source      = "scripts/setup.sh"
    destination = "/tmp/setup.sh"
  }
  
  # Копировать директорию
  provisioner "file" {
    source      = "configs/"
    destination = "/tmp/configs"
  }
  
  # Inline content
  provisioner "file" {
    content     = templatefile("config.tpl", {
      hostname = self.private_ip
    })
    destination = "/tmp/config.txt"
  }
}
```

**Remote-exec Provisioner:**

```hcl
resource "aws_instance" "web" {
  ami           = data.aws_ami.ubuntu.id
  instance_type = "t3.micro"
  
  connection {
    type        = "ssh"
    user        = "ubuntu"
    private_key = file("~/.ssh/id_rsa")
    host        = self.public_ip
  }
  
  # Inline команды
  provisioner "remote-exec" {
    inline = [
      "sudo apt-get update",
      "sudo apt-get install -y nginx",
      "sudo systemctl start nginx",
      "sudo systemctl enable nginx"
    ]
  }
  
  # Или выполнить script
  provisioner "remote-exec" {
    script = "scripts/setup.sh"
  }
  
  # Или список scripts
  provisioner "remote-exec" {
    scripts = [
      "scripts/install.sh",
      "scripts/configure.sh",
      "scripts/start.sh"
    ]
  }
}
```

**Local-exec Provisioner:**

```hcl
resource "aws_instance" "web" {
  ami           = data.aws_ami.ubuntu.id
  instance_type = "t3.micro"
  
  # Выполнить команду локально после создания
  provisioner "local-exec" {
    command = "echo ${self.private_ip} >> private_ips.txt"
  }
  
  # С переменными окружения
  provisioner "local-exec" {
    command = "ansible-playbook -i ${self.public_ip}, playbook.yml"
    
    environment = {
      ANSIBLE_HOST_KEY_CHECKING = "False"
      INSTANCE_ID              = self.id
    }
  }
  
  # С рабочей директорией
  provisioner "local-exec" {
    working_dir = "./scripts"
    command     = "./notify_deployment.sh ${self.id}"
  }
  
  # При удалении
  provisioner "local-exec" {
    when    = destroy
    command = "echo 'Instance ${self.id} is being destroyed' >> destroy.log"
  }
}
```

**null_resource - для workflow без реальных ресурсов:**

```hcl
# null_resource с triggers
resource "null_resource" "cluster_setup" {
  # Triggers определяют когда пересоздавать
  triggers = {
    cluster_instance_ids = join(",", aws_instance.cluster[*].id)
    config_hash          = md5(file("config/cluster.yaml"))
    timestamp            = timestamp()  # Всегда пересоздавать
  }
  
  provisioner "local-exec" {
    command = "ansible-playbook -i inventory.yml setup_cluster.yml"
  }
}

# Зависимости между null_resource
resource "null_resource" "database_migration" {
  depends_on = [aws_db_instance.main]
  
  triggers = {
    db_endpoint = aws_db_instance.main.endpoint
  }
  
  provisioner "local-exec" {
    command = "python migrate.py --host ${aws_db_instance.main.endpoint}"
  }
}
```

**Connection Block - различные типы:**

```hcl
# SSH connection
connection {
  type        = "ssh"
  user        = "ubuntu"
  private_key = file("~/.ssh/id_rsa")
  host        = self.public_ip
  timeout     = "5m"
  
  # Через bastion
  bastion_host        = var.bastion_host
  bastion_user        = "ubuntu"
  bastion_private_key = file("~/.ssh/bastion_key")
}

# WinRM connection (Windows)
connection {
  type     = "winrm"
  user     = "Administrator"
  password = var.admin_password
  host     = self.public_ip
  timeout  = "10m"
  
  # HTTPS
  https    = true
  insecure = true
}
```

**Lifecycle для provisioners:**

```hcl
resource "aws_instance" "web" {
  ami           = data.aws_ami.ubuntu.id
  instance_type = "t3.micro"
  
  # Создание
  provisioner "local-exec" {
    command = "echo 'Creating instance' >> deploy.log"
  }
  
  # Удаление
  provisioner "local-exec" {
    when    = destroy
    command = "echo 'Destroying instance ${self.id}' >> deploy.log"
  }
  
  # Продолжить даже при ошибке
  provisioner "local-exec" {
    command     = "./might_fail.sh"
    on_failure  = continue  # или fail (default)
  }
}
```

### 💻 Задание

Создай инфраструктуру с provisioners для реального use case:

**1. Структура проекта:**

```bash
mkdir terraform-provisioners
cd terraform-provisioners
mkdir -p scripts configs templates
```

**2. scripts/setup.sh:**

```bash
#!/bin/bash
set -e

echo "=== System Setup Started ==="
echo "Hostname: $(hostname)"
echo "Time: $(date)"

# Update system
echo "Updating system packages..."
export DEBIAN_FRONTEND=noninteractive
apt-get update
apt-get upgrade -y

# Install required packages
echo "Installing packages..."
apt-get install -y \
    nginx \
    python3-pip \
    git \
    docker.io \
    curl \
    jq

# Configure Docker
echo "Configuring Docker..."
systemctl enable docker
systemctl start docker
usermod -aG docker ubuntu

# Install Docker Compose
echo "Installing Docker Compose..."
curl -L "https://github.com/docker/compose/releases/download/v2.20.0/docker-compose-$(uname -s)-$(uname -m)" \
    -o /usr/local/bin/docker-compose
chmod +x /usr/local/bin/docker-compose

echo "=== Setup completed successfully ==="
```

**3. scripts/deploy_app.sh:**

```bash
#!/bin/bash
set -e

APP_DIR="/opt/myapp"
CONFIG_FILE="$APP_DIR/config.json"

echo "=== Application Deployment Started ==="

# Create app directory
mkdir -p $APP_DIR
cd $APP_DIR

# Create sample application
cat > index.html <<EOF
<!DOCTYPE html>
<html>
<head>
    <title>Deployed by Terraform</title>
    <style>
        body { font-family: Arial; margin: 40px; background: #f0f0f0; }
        .container { background: white; padding: 30px; border-radius: 10px; }
        .success { color: #28a745; }
    </style>
</head>
<body>
    <div class="container">
        <h1 class="success">✓ Deployment Successful!</h1>
        <p>Server: $(hostname)</p>
        <p>IP: $(hostname -I)</p>
        <p>Time: $(date)</p>
        <p>Config: $(cat $CONFIG_FILE)</p>
    </div>
</body>
</html>
EOF

# Configure Nginx
cat > /etc/nginx/sites-available/default <<EOF
server {
    listen 80 default_server;
    root $APP_DIR;
    index index.html;
    
    location / {
        try_files \$uri \$uri/ =404;
    }
}
EOF

# Start Nginx
systemctl restart nginx
systemctl enable nginx

echo "=== Application deployed successfully ==="
```

**4. scripts/health_check.sh:**

```bash
#!/bin/bash
set -e

HOST=$1
MAX_RETRIES=30
RETRY_DELAY=10

echo "=== Health Check Started ==="
echo "Host: $HOST"

for i in $(seq 1 $MAX_RETRIES); do
    echo "Attempt $i of $MAX_RETRIES..."
    
    if curl -sf http://$HOST > /dev/null; then
        echo "✓ Health check passed!"
        exit 0
    fi
    
    echo "Service not ready yet, waiting ${RETRY_DELAY}s..."
    sleep $RETRY_DELAY
done

echo "✗ Health check failed after $MAX_RETRIES attempts"
exit 1
```

**5. scripts/notify_slack.sh:**

```bash
#!/bin/bash

WEBHOOK_URL=$1
MESSAGE=$2
INSTANCE_ID=$3
INSTANCE_IP=$4

if [ -z "$WEBHOOK_URL" ]; then
    echo "No webhook URL provided, skipping notification"
    exit 0
fi

curl -X POST "$WEBHOOK_URL" \
    -H 'Content-Type: application/json' \
    -d "{
        \"text\": \"$MESSAGE\",
        \"attachments\": [{
            \"color\": \"good\",
            \"fields\": [
                {\"title\": \"Instance ID\", \"value\": \"$INSTANCE_ID\", \"short\": true},
                {\"title\": \"IP Address\", \"value\": \"$INSTANCE_IP\", \"short\": true}
            ]
        }]
    }"
```

**6. templates/app_config.json.tpl:**

```json
{
  "environment": "${environment}",
  "hostname": "${hostname}",
  "ip_address": "${ip_address}",
  "deployment_time": "${deployment_time}",
  "version": "${app_version}",
  "database": {
    "host": "${db_host}",
    "port": ${db_port}
  }
}
```

**7. main.tf:**

```hcl
terraform {
  required_version = ">= 1.6.0"
  
  required_providers {
    aws = {
      source  = "hashicorp/aws"
      version = "~> 5.0"
    }
    null = {
      source  = "hashicorp/null"
      version = "~> 3.2"
    }
  }
}

provider "aws" {
  region = var.aws_region
}

# Get current timestamp
locals {
  deployment_time = timestamp()
  name_prefix     = "${var.project_name}-${var.environment}"
}

# Data sources
data "aws_ami" "ubuntu" {
  most_recent = true
  owners      = ["099720109477"]
  
  filter {
    name   = "name"
    values = ["ubuntu/images/hvm-ssd/ubuntu-jammy-22.04-amd64-server-*"]
  }
}

data "aws_vpc" "default" {
  default = true
}

data "aws_subnets" "default" {
  filter {
    name   = "vpc-id"
    values = [data.aws_vpc.default.id]
  }
}

# Security Group
resource "aws_security_group" "web" {
  name_prefix = "${local.name_prefix}-web-"
  description = "Security group for web servers"
  vpc_id      = data.aws_vpc.default.id
  
  ingress {
    from_port   = 80
    to_port     = 80
    protocol    = "tcp"
    cidr_blocks = ["0.0.0.0/0"]
  }
  
  ingress {
    from_port   = 22
    to_port     = 22
    protocol    = "tcp"
    cidr_blocks = ["0.0.0.0/0"]  # В production используй свой IP!
  }
  
  egress {
    from_port   = 0
    to_port     = 0
    protocol    = "-1"
    cidr_blocks = ["0.0.0.0/0"]
  }
  
  tags = {
    Name = "${local.name_prefix}-web-sg"
  }
  
  lifecycle {
    create_before_destroy = true
  }
}

# SSH Key
resource "aws_key_pair" "deployer" {
  key_name   = "${local.name_prefix}-deployer"
  public_key = file(var.public_key_path)
}

# EC2 Instance с provisioners
resource "aws_instance" "web" {
  count = var.instance_count
  
  ami           = data.aws_ami.ubuntu.id
  instance_type = var.instance_type
  
  subnet_id                   = element(data.aws_subnets.default.ids, count.index)
  vpc_security_group_ids      = [aws_security_group.web.id]
  key_name                    = aws_key_pair.deployer.key_name
  associate_public_ip_address = true
  
  tags = {
    Name        = "${local.name_prefix}-web-${count.index + 1}"
    Environment = var.environment
  }
  
  # Connection для всех provisioners
  connection {
    type        = "ssh"
    user        = "ubuntu"
    private_key = file(var.private_key_path)
    host        = self.public_ip
    timeout     = "5m"
  }
  
  # 1. Copy setup script
  provisioner "file" {
    source      = "scripts/setup.sh"
    destination = "/tmp/setup.sh"
  }
  
  # 2. Copy deploy script
  provisioner "file" {
    source      = "scripts/deploy_app.sh"
    destination = "/tmp/deploy_app.sh"
  }
  
  # 3. Copy config file
  provisioner "file" {
    content = templatefile("${path.module}/templates/app_config.json.tpl", {
      environment     = var.environment
      hostname        = "web-${count.index + 1}"
      ip_address      = self.private_ip
      deployment_time = local.deployment_time
      app_version     = var.app_version
      db_host         = var.db_host
      db_port         = var.db_port
    })
    destination = "/tmp/config.json"
  }
  
  # 4. Run setup
  provisioner "remote-exec" {
    inline = [
      "chmod +x /tmp/setup.sh",
      "sudo /tmp/setup.sh"
    ]
  }
  
  # 5. Deploy application
  provisioner "remote-exec" {
    inline = [
      "sudo mkdir -p /opt/myapp",
      "sudo mv /tmp/config.json /opt/myapp/config.json",
      "chmod +x /tmp/deploy_app.sh",
      "sudo /tmp/deploy_app.sh"
    ]
  }
  
  # 6. Local notification (создание)
  provisioner "local-exec" {
    command = "echo '${local.deployment_time}: Created instance ${self.id} at ${self.public_ip}' >> deployment.log"
  }
  
  # 7. Local notification (удаление)
  provisioner "local-exec" {
    when    = destroy
    command = "echo '${timestamp()}: Destroying instance ${self.id}' >> deployment.log"
  }
  
  # Продолжить даже если notification fails
  provisioner "local-exec" {
    command = "bash scripts/notify_slack.sh '${var.slack_webhook}' 'Instance ${self.id} deployed' '${self.id}' '${self.public_ip}'"
    
    on_failure = continue
  }
}

# null_resource для health check
resource "null_resource" "health_check" {
  count = var.instance_count
  
  depends_on = [aws_instance.web]
  
  triggers = {
    instance_id = aws_instance.web[count.index].id
    instance_ip = aws_instance.web[count.index].public_ip
  }
  
  provisioner "local-exec" {
    command = "bash scripts/health_check.sh ${aws_instance.web[count.index].public_ip}"
  }
}

# null_resource для inventory file (для Ansible)
resource "null_resource" "ansible_inventory" {
  depends_on = [aws_instance.web]
  
  triggers = {
    instance_ids = join(",", aws_instance.web[*].id)
  }
  
  provisioner "local-exec" {
    command = <<-EOT
      cat > inventory.ini <<EOF
      [webservers]
      ${join("\n", [for i in aws_instance.web : "${i.public_ip} ansible_user=ubuntu"])}
      
      [all:vars]
      ansible_ssh_private_key_file=${var.private_key_path}
      EOF
    EOT
  }
  
  provisioner "local-exec" {
    when    = destroy
    command = "rm -f inventory.ini"
  }
}

# null_resource для cleanup при изменении конфигурации
resource "null_resource" "config_cleanup" {
  triggers = {
    config_hash = md5(file("${path.module}/templates/app_config.json.tpl"))
  }
  
  provisioner "local-exec" {
    command = "echo 'Configuration changed, triggering redeployment'"
  }
}
```

**8. variables.tf:**

```hcl
variable "aws_region" {
  description = "AWS region"
  type        = string
  default     = "us-east-1"
}

variable "project_name" {
  description = "Project name"
  type        = string
  default     = "provisioner-demo"
}

variable "environment" {
  description = "Environment name"
  type        = string
  default     = "dev"
}

variable "instance_count" {
  description = "Number of instances"
  type        = number
  default     = 2
}

variable "instance_type" {
  description = "EC2 instance type"
  type        = string
  default     = "t3.micro"
}

variable "public_key_path" {
  description = "Path to public SSH key"
  type        = string
  default     = "~/.ssh/id_rsa.pub"
}

variable "private_key_path" {
  description = "Path to private SSH key"
  type        = string
  default     = "~/.ssh/id_rsa"
}

variable "app_version" {
  description = "Application version"
  type        = string
  default     = "1.0.0"
}

variable "db_host" {
  description = "Database host"
  type        = string
  default     = "db.example.com"
}

variable "db_port" {
  description = "Database port"
  type        = number
  default     = 5432
}

variable "slack_webhook" {
  description = "Slack webhook URL for notifications"
  type        = string
  default     = ""
  sensitive   = true
}
```

**9. outputs.tf:**

```hcl
output "instance_public_ips" {
  description = "Public IP addresses"
  value       = aws_instance.web[*].public_ip
}

output "instance_urls" {
  description = "URLs to access instances"
  value = [
    for instance in aws_instance.web :
    "http://${instance.public_ip}"
  ]
}

output "ssh_commands" {
  description = "SSH commands to connect"
  value = [
    for instance in aws_instance.web :
    "ssh -i ${var.private_key_path} ubuntu@${instance.public_ip}"
  ]
}

output "deployment_time" {
  description = "Deployment timestamp"
  value       = local.deployment_time
}

output "ansible_inventory" {
  description = "Path to Ansible inventory"
  value       = "inventory.ini"
}
```

**10. Подготовка и запуск:**

```bash
# Убедись что есть SSH ключи
ls -la ~/.ssh/id_rsa*

# Сделай scripts исполняемыми
chmod +x scripts/*.sh

# Инициализация
terraform init

# План
terraform plan

# Применить
terraform apply

# Проверить deployment log
cat deployment.log

# Проверить сгенерированный inventory
cat inventory.ini

# Протестировать endpoints
for url in $(terraform output -json instance_urls | jq -r '.[]'); do
    echo "Testing $url"
    curl -s $url | grep "Deployment Successful"
done
```

### 🚀 Бонус (продвинутое)

**1. Provisioner с retry logic:**

```hcl
resource "null_resource" "deployment_with_retry" {
  triggers = {
    deployment_id = timestamp()
  }
  
  provisioner "local-exec" {
    command = <<-EOT
      for i in {1..5}; do
        echo "Attempt $i of 5..."
        if ./deploy.sh; then
          echo "Deployment successful!"
          exit 0
        fi
        echo "Deployment failed, retrying in 30s..."
        sleep 30
      done
      echo "Deployment failed after 5 attempts"
      exit 1
    EOT
    
    interpreter = ["/bin/bash", "-c"]
  }
}
```

**2. Conditional provisioners:**

```hcl
resource "aws_instance" "web" {
  ami           = data.aws_ami.ubuntu.id
  instance_type = "t3.micro"
  
  # Выполнить только для production
  dynamic "provisioner" {
    for_each = var.environment == "prod" ? [1] : []
    
    content {
      local-exec {
        command = "python notify_production_deploy.py ${self.id}"
      }
    }
  }
}
```

**3. Multi-step deployment с зависимостями:**

```hcl
# Step 1: Database setup
resource "null_resource" "database_setup" {
  triggers = {
    db_instance_id = aws_db_instance.main.id
  }
  
  provisioner "local-exec" {
    command = "python db_migrate.py --host ${aws_db_instance.main.endpoint}"
  }
}

# Step 2: Application deployment (зависит от DB)
resource "null_resource" "app_deployment" {
  depends_on = [null_resource.database_setup]
  
  triggers = {
    app_version = var.app_version
  }
  
  provisioner "local-exec" {
    command = "ansible-playbook -i inventory.ini deploy_app.yml"
  }
}

# Step 3: Smoke tests (зависит от app)
resource "null_resource" "smoke_tests" {
  depends_on = [null_resource.app_deployment]
  
  provisioner "local-exec" {
    command = "pytest tests/smoke_tests.py"
  }
}

# Step 4: Add to monitoring (после успешного deployment)
resource "null_resource" "add_monitoring" {
  depends_on = [null_resource.smoke_tests]
  
  provisioner "local-exec" {
    command = "python add_to_monitoring.py --instances ${join(",", aws_instance.web[*].id)}"
  }
}
```

**4. Remote-exec с error handling:**

```hcl
resource "aws_instance" "web" {
  # ...
  
  connection {
    type        = "ssh"
    user        = "ubuntu"
    private_key = file(var.private_key_path)
    host        = self.public_ip
  }
  
  provisioner "remote-exec" {
    inline = [
      "set -e",  # Exit on error
      "exec 2>&1 | tee /tmp/provisioner.log",  # Log everything
      "",
      "echo '=== Starting provisioner at $(date) ==='",
      "",
      "# Function for error handling",
      "handle_error() {",
      "  echo 'ERROR: Provisioner failed at $(date)' >&2",
      "  cat /tmp/provisioner.log >&2",
      "  exit 1",
      "}",
      "trap handle_error ERR",
      "",
      "# Actual provisioning steps",
      "sudo apt-get update || handle_error",
      "sudo apt-get install -y nginx || handle_error",
      "sudo systemctl start nginx || handle_error",
      "",
      "echo '=== Provisioner completed successfully at $(date) ==='"
    ]
  }
}
```

---

Продолжить с **Модулем 9: Advanced Features - Dynamic Blocks, For Expressions**?

---

## Модуль 9: Advanced Features - Dynamic Blocks, For Expressions (30 минут)

### 🎯 Напоминалка

**Dynamic Blocks - генерация вложенных блоков:**

```hcl
# Без dynamic - повторяющийся код
resource "aws_security_group" "example" {
  name = "example"
  
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
  
  ingress {
    from_port   = 22
    to_port     = 22
    protocol    = "tcp"
    cidr_blocks = ["10.0.0.0/8"]
  }
}

# С dynamic - чисто и гибко
resource "aws_security_group" "example" {
  name = "example"
  
  dynamic "ingress" {
    for_each = var.ingress_rules
    content {
      from_port   = ingress.value.from_port
      to_port     = ingress.value.to_port
      protocol    = ingress.value.protocol
      cidr_blocks = ingress.value.cidr_blocks
    }
  }
}
```

**For Expressions:**

```hcl
# For list → list
[for item in var.list : upper(item)]

# For list → map
{for item in var.list : item => upper(item)}

# For map → list
[for key, value in var.map : "${key}=${value}"]

# For map → map
{for key, value in var.map : key => upper(value)}

# С условием (if)
[for item in var.list : item if item != ""]

# С индексом
[for idx, item in var.list : "${idx}: ${item}"]

# Nested loops
[for outer in var.outer : [
  for inner in outer.inner : inner
]]

# Flatten nested results
flatten([for outer in var.outer : [
  for inner in outer.inner : inner
]])
```

**Conditional Expressions:**

```hcl
# Тернарный оператор
condition ? true_val : false_val

# Примеры
instance_type = var.environment == "prod" ? "t3.large" : "t3.micro"

enabled = var.create_resource ? 1 : 0

tags = var.environment == "prod" ? {
  Backup = "true"
  HighAvailability = "true"
} : {}
```

**Type Constraints (продвинутые):**

```hcl
variable "complex_object" {
  type = object({
    name     = string
    age      = number
    enabled  = bool
    tags     = map(string)
    subnets  = list(string)
    metadata = optional(object({
      department = string
      cost_center = optional(string, "default")
    }))
  })
  
  validation {
    condition     = length(var.complex_object.name) > 3
    error_message = "Name must be longer than 3 characters."
  }
  
  validation {
    condition     = var.complex_object.age >= 0 && var.complex_object.age <= 150
    error_message = "Age must be between 0 and 150."
  }
}

variable "list_of_objects" {
  type = list(object({
    name = string
    size = number
  }))
}

variable "map_of_objects" {
  type = map(object({
    ami           = string
    instance_type = string
  }))
}
```

**Sensitive Data:**

```hcl
variable "database_password" {
  type      = string
  sensitive = true
}

output "db_password" {
  value     = var.database_password
  sensitive = true
}

resource "aws_db_instance" "example" {
  password = var.database_password  # Не будет показан в logs
  
  # ...
}

# В locals тоже можно
locals {
  sensitive_data = sensitive(var.some_value)
}
```

**Advanced Locals:**

```hcl
locals {
  # Условия
  instance_type = var.environment == "prod" ? "t3.large" : "t3.micro"
  
  # For expressions
  subnet_ids = [for s in data.aws_subnets.default.ids : s if length(s) > 0]
  
  # Вычисления
  total_instances = sum([for k, v in var.instance_counts : v])
  
  # Merge maps
  all_tags = merge(
    var.common_tags,
    var.environment_tags,
    {
      ManagedBy = "Terraform"
      Timestamp = timestamp()
    }
  )
  
  # Nested
  vpc_config = {
    for az in data.aws_availability_zones.available.names :
    az => {
      public_subnet  = cidrsubnet(var.vpc_cidr, 8, index(data.aws_availability_zones.available.names, az))
      private_subnet = cidrsubnet(var.vpc_cidr, 8, index(data.aws_availability_zones.available.names, az) + 10)
    }
  }
}
```

**Validation Rules:**

```hcl
variable "environment" {
  type = string
  
  validation {
    condition     = contains(["dev", "staging", "prod"], var.environment)
    error_message = "Environment must be dev, staging, or prod."
  }
}

variable "cidr_block" {
  type = string
  
  validation {
    condition     = can(cidrhost(var.cidr_block, 0))
    error_message = "Must be a valid CIDR block."
  }
}

variable "instance_count" {
  type = number
  
  validation {
    condition     = var.instance_count >= 1 && var.instance_count <= 10
    error_message = "Instance count must be between 1 and 10."
  }
}

variable "email" {
  type = string
  
  validation {
    condition     = can(regex("^[a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\\.[a-zA-Z]{2,}$", var.email))
    error_message = "Must be a valid email address."
  }
}
```

### 💻 Задание

Создай продвинутую конфигурацию с использованием всех advanced features:

**1. Структура проекта:**

```bash
mkdir terraform-advanced
cd terraform-advanced
mkdir -p modules/vpc modules/security-group modules/compute
```

**2. variables.tf:**

```hcl
# Complex variable types
variable "vpc_config" {
  description = "VPC configuration"
  type = object({
    name                 = string
    cidr_block          = string
    enable_dns_hostnames = bool
    enable_nat_gateway  = bool
    availability_zones  = list(string)
    
    subnet_config = object({
      public_subnet_bits  = number
      private_subnet_bits = number
      database_subnet_bits = number
    })
    
    tags = optional(map(string), {})
  })
  
  validation {
    condition     = can(cidrhost(var.vpc_config.cidr_block, 0))
    error_message = "VPC CIDR block must be valid."
  }
  
  validation {
    condition     = length(var.vpc_config.availability_zones) >= 2
    error_message = "At least 2 availability zones required."
  }
}

variable "security_groups" {
  description = "Security group configurations"
  type = map(object({
    description = string
    
    ingress_rules = list(object({
      from_port   = number
      to_port     = number
      protocol    = string
      cidr_blocks = optional(list(string), [])
      description = string
    }))
    
    egress_rules = list(object({
      from_port   = number
      to_port     = number
      protocol    = string
      cidr_blocks = list(string)
      description = string
    }))
    
    tags = optional(map(string), {})
  }))
  
  default = {}
}

variable "instances" {
  description = "EC2 instance configurations"
  type = map(object({
    instance_type = string
    ami_filters = object({
      owners = list(string)
      name   = string
    })
    count           = number
    subnet_type     = string  # "public" or "private"
    security_groups = list(string)
    user_data       = optional(string, "")
    monitoring      = optional(bool, false)
    
    root_block_device = optional(object({
      volume_type = string
      volume_size = number
      encrypted   = bool
    }), {
      volume_type = "gp3"
      volume_size = 20
      encrypted   = true
    })
    
    tags = optional(map(string), {})
  }))
  
  validation {
    condition     = alltrue([for k, v in var.instances : contains(["public", "private"], v.subnet_type)])
    error_message = "subnet_type must be either 'public' or 'private'."
  }
}

variable "load_balancers" {
  description = "Load balancer configurations"
  type = map(object({
    type            = string  # "application" or "network"
    internal        = bool
    subnets_type    = string  # "public" or "private"
    security_groups = optional(list(string), [])
    
    target_groups = list(object({
      name     = string
      port     = number
      protocol = string
      
      health_check = object({
        enabled             = bool
        path                = optional(string, "/")
        healthy_threshold   = optional(number, 3)
        unhealthy_threshold = optional(number, 3)
      })
      
      targets = list(string)  # instance keys from var.instances
    }))
    
    listeners = list(object({
      port            = number
      protocol        = string
      target_group    = string
      certificate_arn = optional(string, "")
    }))
  }))
  
  default = {}
}

variable "environment" {
  description = "Environment name"
  type        = string
  
  validation {
    condition     = contains(["dev", "staging", "prod"], var.environment)
    error_message = "Environment must be dev, staging, or prod."
  }
}

variable "project_name" {
  description = "Project name"
  type        = string
  
  validation {
    condition     = can(regex("^[a-z][a-z0-9-]*$", var.project_name))
    error_message = "Project name must start with a letter and contain only lowercase letters, numbers, and hyphens."
  }
}

variable "common_tags" {
  description = "Common tags for all resources"
  type        = map(string)
  default     = {}
}

variable "enable_monitoring" {
  description = "Enable CloudWatch monitoring"
  type        = bool
  default     = false
}

variable "backup_retention_days" {
  description = "Backup retention in days"
  type        = number
  default     = 7
  
  validation {
    condition     = var.backup_retention_days >= 1 && var.backup_retention_days <= 35
    error_message = "Backup retention must be between 1 and 35 days."
  }
}

variable "slack_webhook" {
  description = "Slack webhook for notifications"
  type        = string
  default     = ""
  sensitive   = true
}
```

**3. locals.tf:**

```hcl
locals {
  # Name prefix
  name_prefix = "${var.project_name}-${var.environment}"
  
  # Calculate subnets using for expressions
  public_subnets = {
    for idx, az in var.vpc_config.availability_zones :
    az => cidrsubnet(
      var.vpc_config.cidr_block,
      var.vpc_config.subnet_config.public_subnet_bits,
      idx
    )
  }
  
  private_subnets = {
    for idx, az in var.vpc_config.availability_zones :
    az => cidrsubnet(
      var.vpc_config.cidr_block,
      var.vpc_config.subnet_config.private_subnet_bits,
      idx + 10
    )
  }
  
  database_subnets = {
    for idx, az in var.vpc_config.availability_zones :
    az => cidrsubnet(
      var.vpc_config.cidr_block,
      var.vpc_config.subnet_config.database_subnet_bits,
      idx + 20
    )
  }
  
  # Flatten all subnets for easy access
  all_subnets = merge(
    { for k, v in local.public_subnets : "public-${k}" => v },
    { for k, v in local.private_subnets : "private-${k}" => v },
    { for k, v in local.database_subnets : "database-${k}" => v }
  )
  
  # Group instances by subnet type
  public_instances = {
    for k, v in var.instances :
    k => v if v.subnet_type == "public"
  }
  
  private_instances = {
    for k, v in var.instances :
    k => v if v.subnet_type == "private"
  }
  
  # Calculate total instance count
  total_instance_count = sum([for k, v in var.instances : v.count])
  
  # Environment-specific settings
  environment_config = {
    dev = {
      backup_enabled        = false
      multi_az             = false
      deletion_protection  = false
    }
    staging = {
      backup_enabled        = true
      multi_az             = false
      deletion_protection  = false
    }
    prod = {
      backup_enabled        = true
      multi_az             = true
      deletion_protection  = true
    }
  }
  
  current_env_config = local.environment_config[var.environment]
  
  # Merge all tags
  common_tags = merge(
    var.common_tags,
    {
      Environment        = var.environment
      Project           = var.project_name
      ManagedBy         = "Terraform"
      TerraformWorkspace = terraform.workspace
      BackupEnabled     = tostring(local.current_env_config.backup_enabled)
    }
  )
  
  # Security group rules flattened for easier processing
  all_sg_ingress_rules = flatten([
    for sg_key, sg in var.security_groups : [
      for rule in sg.ingress_rules : {
        sg_key      = sg_key
        from_port   = rule.from_port
        to_port     = rule.to_port
        protocol    = rule.protocol
        cidr_blocks = rule.cidr_blocks
        description = rule.description
      }
    ]
  ])
  
  # Load balancer target groups with instance mappings
  lb_target_mappings = flatten([
    for lb_key, lb in var.load_balancers : [
      for tg in lb.target_groups : [
        for target in tg.targets : {
          lb_key     = lb_key
          tg_name    = tg.name
          instance_key = target
        }
      ]
    ]
  ])
}
```

**4. main.tf:**

```hcl
terraform {
  required_version = ">= 1.6.0"
  
  required_providers {
    aws = {
      source  = "hashicorp/aws"
      version = "~> 5.0"
    }
  }
}

provider "aws" {
  region = "us-east-1"
  
  default_tags {
    tags = local.common_tags
  }
}

# Data sources
data "aws_availability_zones" "available" {
  state = "available"
}

# VPC
resource "aws_vpc" "main" {
  cidr_block           = var.vpc_config.cidr_block
  enable_dns_hostnames = var.vpc_config.enable_dns_hostnames
  enable_dns_support   = true
  
  tags = merge(local.common_tags, var.vpc_config.tags, {
    Name = "${local.name_prefix}-vpc"
  })
}

# Internet Gateway
resource "aws_internet_gateway" "main" {
  vpc_id = aws_vpc.main.id
  
  tags = {
    Name = "${local.name_prefix}-igw"
  }
}

# Public Subnets with dynamic block
resource "aws_subnet" "public" {
  for_each = local.public_subnets
  
  vpc_id                  = aws_vpc.main.id
  cidr_block              = each.value
  availability_zone       = each.key
  map_public_ip_on_launch = true
  
  tags = {
    Name = "${local.name_prefix}-public-${each.key}"
    Type = "public"
  }
}

# Private Subnets
resource "aws_subnet" "private" {
  for_each = local.private_subnets
  
  vpc_id            = aws_vpc.main.id
  cidr_block        = each.value
  availability_zone = each.key
  
  tags = {
    Name = "${local.name_prefix}-private-${each.key}"
    Type = "private"
  }
}

# Database Subnets
resource "aws_subnet" "database" {
  for_each = local.database_subnets
  
  vpc_id            = aws_vpc.main.id
  cidr_block        = each.value
  availability_zone = each.key
  
  tags = {
    Name = "${local.name_prefix}-database-${each.key}"
    Type = "database"
  }
}

# NAT Gateway (conditional)
resource "aws_eip" "nat" {
  count = var.vpc_config.enable_nat_gateway ? length(local.public_subnets) : 0
  
  domain = "vpc"
  
  tags = {
    Name = "${local.name_prefix}-nat-eip-${count.index + 1}"
  }
  
  depends_on = [aws_internet_gateway.main]
}

resource "aws_nat_gateway" "main" {
  count = var.vpc_config.enable_nat_gateway ? length(local.public_subnets) : 0
  
  allocation_id = aws_eip.nat[count.index].id
  subnet_id     = values(aws_subnet.public)[count.index].id
  
  tags = {
    Name = "${local.name_prefix}-nat-${count.index + 1}"
  }
}

# Route Tables with dynamic routes
resource "aws_route_table" "public" {
  vpc_id = aws_vpc.main.id
  
  # Dynamic route to IGW
  dynamic "route" {
    for_each = [1]
    content {
      cidr_block = "0.0.0.0/0"
      gateway_id = aws_internet_gateway.main.id
    }
  }
  
  tags = {
    Name = "${local.name_prefix}-public-rt"
  }
}

resource "aws_route_table" "private" {
  for_each = var.vpc_config.enable_nat_gateway ? local.private_subnets : {}
  
  vpc_id = aws_vpc.main.id
  
  # Dynamic route to NAT (if enabled)
  dynamic "route" {
    for_each = var.vpc_config.enable_nat_gateway ? [1] : []
    content {
      cidr_block     = "0.0.0.0/0"
      nat_gateway_id = aws_nat_gateway.main[index(keys(local.private_subnets), each.key)].id
    }
  }
  
  tags = {
    Name = "${local.name_prefix}-private-rt-${each.key}"
  }
}

# Route Table Associations
resource "aws_route_table_association" "public" {
  for_each = aws_subnet.public
  
  subnet_id      = each.value.id
  route_table_id = aws_route_table.public.id
}

resource "aws_route_table_association" "private" {
  for_each = var.vpc_config.enable_nat_gateway ? aws_subnet.private : {}
  
  subnet_id      = each.value.id
  route_table_id = aws_route_table.private[each.key].id
}

# Security Groups with dynamic ingress/egress
resource "aws_security_group" "custom" {
  for_each = var.security_groups
  
  name_prefix = "${local.name_prefix}-${each.key}-"
  description = each.value.description
  vpc_id      = aws_vpc.main.id
  
  # Dynamic ingress rules
  dynamic "ingress" {
    for_each = each.value.ingress_rules
    content {
      from_port   = ingress.value.from_port
      to_port     = ingress.value.to_port
      protocol    = ingress.value.protocol
      cidr_blocks = length(ingress.value.cidr_blocks) > 0 ? ingress.value.cidr_blocks : ["0.0.0.0/0"]
      description = ingress.value.description
    }
  }
  
  # Dynamic egress rules
  dynamic "egress" {
    for_each = each.value.egress_rules
    content {
      from_port   = egress.value.from_port
      to_port     = egress.value.to_port
      protocol    = egress.value.protocol
      cidr_blocks = egress.value.cidr_blocks
      description = egress.value.description
    }
  }
  
  tags = merge(local.common_tags, each.value.tags, {
    Name = "${local.name_prefix}-${each.key}-sg"
  })
  
  lifecycle {
    create_before_destroy = true
  }
}

# AMI lookup for each instance type
data "aws_ami" "instance_ami" {
  for_each = var.instances
  
  most_recent = true
  owners      = each.value.ami_filters.owners
  
  filter {
    name   = "name"
    values = [each.value.ami_filters.name]
  }
}

# EC2 Instances
resource "aws_instance" "main" {
  for_each = {
    for combo in flatten([
      for key, config in var.instances : [
        for i in range(config.count) : {
          key           = "${key}-${i}"
          config_key    = key
          config        = config
          instance_num  = i
        }
      ]
    ]) : combo.key => combo
  }
  
  ami           = data.aws_ami.instance_ami[each.value.config_key].id
  instance_type = each.value.config.instance_type
  
  # Select subnet based on type
  subnet_id = each.value.config.subnet_type == "public" ? (
    values(aws_subnet.public)[each.value.instance_num % length(aws_subnet.public)].id
  ) : (
    values(aws_subnet.private)[each.value.instance_num % length(aws_subnet.private)].id
  )
  
  # Map security group names to IDs
  vpc_security_group_ids = [
    for sg_name in each.value.config.security_groups :
    aws_security_group.custom[sg_name].id
  ]
  
  associate_public_ip_address = each.value.config.subnet_type == "public"
  monitoring                  = each.value.config.monitoring
  
  user_data = each.value.config.user_data != "" ? each.value.config.user_data : null
  
  # Dynamic root block device
  dynamic "root_block_device" {
    for_each = [each.value.config.root_block_device]
    content {
      volume_type = root_block_device.value.volume_type
      volume_size = root_block_device.value.volume_size
      encrypted   = root_block_device.value.encrypted
    }
  }
  
  tags = merge(local.common_tags, each.value.config.tags, {
    Name         = "${local.name_prefix}-${each.value.config_key}-${each.value.instance_num + 1}"
    InstanceKey  = each.value.config_key
    InstanceNum  = tostring(each.value.instance_num + 1)
    SubnetType   = each.value.config.subnet_type
  })
  
  lifecycle {
    create_before_destroy = true
  }
}
```

**5. outputs.tf:**

```hcl
output "vpc_info" {
  description = "VPC information"
  value = {
    id         = aws_vpc.main.id
    cidr_block = aws_vpc.main.cidr_block
    
    public_subnets = {
      for k, v in aws_subnet.public :
      k => {
        id         = v.id
        cidr_block = v.cidr_block
        az         = v.availability_zone
      }
    }
    
    private_subnets = {
      for k, v in aws_subnet.private :
      k => {
        id         = v.id
        cidr_block = v.cidr_block
        az         = v.availability_zone
      }
    }
  }
}

output "security_groups" {
  description = "Security group IDs"
  value = {
    for k, v in aws_security_group.custom :
    k => {
      id          = v.id
      name        = v.name
      description = v.description
    }
  }
}

output "instances" {
  description = "Instance information"
  value = {
    for k, v in aws_instance.main :
    k => {
      id         = v.id
      public_ip  = v.public_ip
      private_ip = v.private_ip
      subnet_id  = v.subnet_id
      az         = v.availability_zone
    }
  }
}

output "instance_summary" {
  description = "Summary of instances by type"
  value = {
    total_count = local.total_instance_count
    
    by_type = {
      for config_key in keys(var.instances) :
      config_key => {
        count      = var.instances[config_key].count
        type       = var.instances[config_key].instance_type
        subnet     = var.instances[config_key].subnet_type
        instances  = [
          for k, v in aws_instance.main :
          {
            id         = v.id
            public_ip  = v.public_ip
            private_ip = v.private_ip
          }
          if startswith(k, config_key)
        ]
      }
    }
  }
}

output "subnet_summary" {
  description = "Subnet CIDR allocations"
  value = {
    vpc_cidr = var.vpc_config.cidr_block
    
    allocations = {
      public   = local.public_subnets
      private  = local.private_subnets
      database = local.database_subnets
    }
    
    utilization = {
      total_addresses = pow(2, 32 - tonumber(split("/", var.vpc_config.cidr_block)[1]))
      
      allocated_addresses = sum([
        for subnet in values(local.all_subnets) :
        pow(2, 32 - tonumber(split("/", subnet)[1]))
      ])
    }
  }
}

output "environment_config" {
  description = "Applied environment configuration"
  value       = local.current_env_config
}
```

**6. terraform.tfvars:**

```hcl
project_name = "advanced-demo"
environment  = "dev"

vpc_config = {
  name                 = "main"
  cidr_block          = "10.0.0.0/16"
  enable_dns_hostnames = true
  enable_nat_gateway  = false
  availability_zones  = ["us-east-1a", "us-east-1b"]
  
  subnet_config = {
    public_subnet_bits   = 8  # /24 subnets
    private_subnet_bits  = 8  # /24 subnets
    database_subnet_bits = 8  # /24 subnets
  }
  
  tags = {
    CostCenter = "Engineering"
  }
}

security_groups = {
  web = {
    description = "Security group for web servers"
    
    ingress_rules = [
      {
        from_port   = 80
        to_port     = 80
        protocol    = "tcp"
        cidr_blocks = ["0.0.0.0/0"]
        description = "HTTP from anywhere"
      },
      {
        from_port   = 443
        to_port     = 443
        protocol    = "tcp"
        cidr_blocks = ["0.0.0.0/0"]
        description = "HTTPS from anywhere"
      },
      {
        from_port   = 22
        to_port     = 22
        protocol    = "tcp"
        cidr_blocks = ["10.0.0.0/8"]
        description = "SSH from private networks"
      }
    ]
    
    egress_rules = [
      {
        from_port   = 0
        to_port     = 0
        protocol    = "-1"
        cidr_blocks = ["0.0.0.0/0"]
        description = "All outbound traffic"
      }
    ]
  }
  
  app = {
    description = "Security group for app servers"
    
    ingress_rules = [
      {
        from_port   = 8080
        to_port     = 8080
        protocol    = "tcp"
        cidr_blocks = ["10.0.0.0/16"]
        description = "App port from VPC"
      }
    ]
    
    egress_rules = [
      {
        from_port   = 0
        to_port     = 0
        protocol    = "-1"
        cidr_blocks = ["0.0.0.0/0"]
        description = "All outbound traffic"
      }
    ]
  }
}

instances = {
  web = {
    instance_type = "t3.micro"
    ami_filters = {
      owners = ["099720109477"]
      name   = "ubuntu/images/hvm-ssd/ubuntu-jammy-22.04-amd64-server-*"
    }
    count           = 2
    subnet_type     = "public"
    security_groups = ["web"]
    monitoring      = false
    
    tags = {
      Role = "WebServer"
    }
  }
  
  app = {
    instance_type = "t3.small"
    ami_filters = {
      owners = ["099720109477"]
      name   = "ubuntu/images/hvm-ssd/ubuntu-jammy-22.04-amd64-server-*"
    }
    count           = 2
    subnet_type     = "private"
    security_groups = ["app"]
    monitoring      = true
    
    root_block_device = {
      volume_type = "gp3"
      volume_size = 30
      encrypted   = true
    }
    
    tags = {
      Role = "AppServer"
    }
  }
}

common_tags = {
  Owner       = "DevOps Team"
  Project     = "Advanced Terraform Demo"
  Environment = "Development"
}

enable_monitoring = false
backup_retention_days = 7
```


### 💻 Задание 

**7. Запуск и тестирование:**

```bash
# Инициализация
terraform init

# Валидация конфигурации
terraform validate

# Форматирование
terraform fmt -recursive

# План с детальным выводом
terraform plan -out=tfplan

# Проверка плана
terraform show tfplan

# Применение
terraform apply tfplan

# Просмотр outputs
terraform output

# Детальный output одного значения
terraform output -json vpc_info | jq

# Проверка созданных ресурсов
terraform state list

# Детали конкретного ресурса
terraform state show 'aws_instance.main["web-0"]'
```

**8. Тестирование динамических блоков:**

```bash
# Проверить все security group rules
terraform state list | grep aws_security_group

# Проверить subnet allocations
terraform output -json subnet_summary | jq '.allocations'

# Проверить instance distribution
terraform output -json instance_summary | jq '.by_type'
```

### 🚀 Бонус (продвинутое)

**1. Advanced For Expressions - Nested Transformations:**

```hcl
# advanced-transformations.tf

locals {
  # Группировка instances по AZ
  instances_by_az = {
    for az in var.vpc_config.availability_zones :
    az => [
      for k, v in aws_instance.main :
      v if v.availability_zone == az
    ]
  }
  
  # Создание DNS records из instances
  dns_records = {
    for k, v in aws_instance.main :
    "${k}.internal" => {
      type    = "A"
      ttl     = 300
      records = [v.private_ip]
    }
  }
  
  # Вычисление стоимости (примерная)
  instance_costs = {
    for k, v in var.instances :
    k => {
      hourly_cost = (
        v.instance_type == "t3.micro" ? 0.0104 :
        v.instance_type == "t3.small" ? 0.0208 :
        v.instance_type == "t3.medium" ? 0.0416 :
        v.instance_type == "t3.large" ? 0.0832 :
        0.10
      )
      daily_cost   = v.count * (v.instance_type == "t3.micro" ? 0.0104 : 0.0208) * 24
      monthly_cost = v.count * (v.instance_type == "t3.micro" ? 0.0104 : 0.0208) * 24 * 30
    }
  }
  
  total_monthly_cost = sum([
    for k, v in local.instance_costs :
    v.monthly_cost
  ])
  
  # Multi-level nested transformation
  subnet_instance_map = {
    for subnet_type in ["public", "private"] :
    subnet_type => {
      subnets = {
        for az, subnet_id in (subnet_type == "public" ? 
          { for k, v in aws_subnet.public : k => v.id } :
          { for k, v in aws_subnet.private : k => v.id }
        ) :
        az => {
          subnet_id = subnet_id
          instances = [
            for k, v in aws_instance.main :
            {
              id         = v.id
              private_ip = v.private_ip
              public_ip  = v.public_ip
            }
            if v.subnet_id == subnet_id
          ]
        }
      }
    }
  }
}

output "cost_analysis" {
  description = "Infrastructure cost analysis"
  value = {
    by_instance_type = local.instance_costs
    total_monthly    = local.total_monthly_cost
    
    breakdown = {
      for k, v in local.instance_costs :
      k => "$$${format("%.2f", v.monthly_cost)}/month"
    }
  }
}

output "subnet_instance_distribution" {
  description = "Instances distributed across subnets"
  value       = local.subnet_instance_map
}
```

**2. Advanced Dynamic Blocks - Nested Dynamics:**

```hcl
# advanced-dynamic-blocks.tf

variable "alb_listeners" {
  type = map(object({
    port     = number
    protocol = string
    
    default_action = object({
      type             = string
      target_group_arn = optional(string)
    })
    
    rules = optional(list(object({
      priority = number
      
      conditions = list(object({
        field  = string
        values = list(string)
      }))
      
      actions = list(object({
        type             = string
        target_group_arn = optional(string)
      }))
    })), [])
  }))
  
  default = {}
}

resource "aws_lb_listener" "main" {
  for_each = var.alb_listeners
  
  load_balancer_arn = aws_lb.main.arn
  port              = each.value.port
  protocol          = each.value.protocol
  
  default_action {
    type             = each.value.default_action.type
    target_group_arn = each.value.default_action.target_group_arn
  }
  
  # Nested dynamic blocks for listener rules
  dynamic "rule" {
    for_each = each.value.rules
    content {
      priority = rule.value.priority
      
      # Nested conditions
      dynamic "condition" {
        for_each = rule.value.conditions
        content {
          dynamic "path_pattern" {
            for_each = condition.value.field == "path-pattern" ? [1] : []
            content {
              values = condition.value.values
            }
          }
          
          dynamic "host_header" {
            for_each = condition.value.field == "host-header" ? [1] : []
            content {
              values = condition.value.values
            }
          }
          
          dynamic "http_header" {
            for_each = condition.value.field == "http-header" ? [1] : []
            content {
              http_header_name = split(":", condition.value.field)[1]
              values           = condition.value.values
            }
          }
        }
      }
      
      # Nested actions
      dynamic "action" {
        for_each = rule.value.actions
        content {
          type             = action.value.type
          target_group_arn = action.value.target_group_arn
        }
      }
    }
  }
}
```

**3. Conditional Resource Creation with For:**

```hcl
# conditional-resources.tf

variable "feature_flags" {
  type = object({
    enable_monitoring    = bool
    enable_backup        = bool
    enable_cdn          = bool
    enable_waf          = bool
    enable_auto_scaling = bool
  })
  
  default = {
    enable_monitoring    = false
    enable_backup        = false
    enable_cdn          = false
    enable_waf          = false
    enable_auto_scaling = false
  }
}

# CloudWatch Alarms (conditional)
resource "aws_cloudwatch_metric_alarm" "cpu" {
  for_each = var.feature_flags.enable_monitoring ? var.instances : {}
  
  alarm_name          = "${local.name_prefix}-${each.key}-cpu"
  comparison_operator = "GreaterThanThreshold"
  evaluation_periods  = "2"
  metric_name         = "CPUUtilization"
  namespace           = "AWS/EC2"
  period              = "300"
  statistic           = "Average"
  threshold           = "80"
  alarm_description   = "CPU utilization for ${each.key}"
  
  dimensions = {
    InstanceId = aws_instance.main["${each.key}-0"].id
  }
}

# Backup Plans (conditional)
resource "aws_backup_plan" "main" {
  count = var.feature_flags.enable_backup ? 1 : 0
  
  name = "${local.name_prefix}-backup-plan"
  
  rule {
    rule_name         = "daily_backup"
    target_vault_name = aws_backup_vault.main[0].name
    schedule          = "cron(0 2 * * ? *)"
    
    lifecycle {
      delete_after = var.backup_retention_days
    }
  }
}

resource "aws_backup_vault" "main" {
  count = var.feature_flags.enable_backup ? 1 : 0
  
  name = "${local.name_prefix}-backup-vault"
}

# Auto Scaling Groups (conditional)
resource "aws_autoscaling_group" "main" {
  for_each = var.feature_flags.enable_auto_scaling ? var.instances : {}
  
  name                = "${local.name_prefix}-${each.key}-asg"
  vpc_zone_identifier = [for s in aws_subnet.private : s.id]
  
  min_size         = each.value.count
  max_size         = each.value.count * 2
  desired_capacity = each.value.count
  
  launch_template {
    id      = aws_launch_template.main[each.key].id
    version = "$Latest"
  }
  
  dynamic "tag" {
    for_each = merge(local.common_tags, {
      Name = "${local.name_prefix}-${each.key}"
    })
    
    content {
      key                 = tag.key
      value               = tag.value
      propagate_at_launch = true
    }
  }
}

resource "aws_launch_template" "main" {
  for_each = var.feature_flags.enable_auto_scaling ? var.instances : {}
  
  name_prefix   = "${local.name_prefix}-${each.key}-"
  image_id      = data.aws_ami.instance_ami[each.key].id
  instance_type = each.value.instance_type
  
  vpc_security_group_ids = [
    for sg_name in each.value.security_groups :
    aws_security_group.custom[sg_name].id
  ]
  
  monitoring {
    enabled = each.value.monitoring
  }
}
```

**4. Complex Validation with For Expressions:**

```hcl
# complex-validations.tf

variable "network_config" {
  type = object({
    vpc_cidr = string
    subnets = list(object({
      name = string
      cidr = string
      type = string
    }))
  })
  
  # Validate all subnets are within VPC CIDR
  validation {
    condition = alltrue([
      for subnet in var.network_config.subnets :
      can(cidrsubnet(var.network_config.vpc_cidr, 0, 0)) &&
      can(cidrsubnet(subnet.cidr, 0, 0))
    ])
    error_message = "All subnet CIDRs must be valid."
  }
  
  # Validate no overlapping subnets
  validation {
    condition = length(var.network_config.subnets) == length(
      distinct([for s in var.network_config.subnets : s.cidr])
    )
    error_message = "Subnet CIDRs must not overlap."
  }
  
  # Validate subnet types
  validation {
    condition = alltrue([
      for subnet in var.network_config.subnets :
      contains(["public", "private", "database"], subnet.type)
    ])
    error_message = "Subnet type must be public, private, or database."
  }
}

variable "instance_config" {
  type = map(object({
    count         = number
    instance_type = string
    subnet_type   = string
  }))
  
  # Validate instance counts
  validation {
    condition = alltrue([
      for k, v in var.instance_config :
      v.count >= 1 && v.count <= 10
    ])
    error_message = "Instance count must be between 1 and 10."
  }
  
  # Validate instance types
  validation {
    condition = alltrue([
      for k, v in var.instance_config :
      can(regex("^t3\\.(micro|small|medium|large|xlarge)$", v.instance_type))
    ])
    error_message = "Instance type must be t3 family."
  }
  
  # Validate total instances
  validation {
    condition = sum([
      for k, v in var.instance_config : v.count
    ]) <= 20
    error_message = "Total instance count cannot exceed 20."
  }
}
```

**5. Advanced Outputs with Transformations:**

```hcl
# advanced-outputs.tf

output "network_topology" {
  description = "Complete network topology visualization"
  value = {
    vpc = {
      id         = aws_vpc.main.id
      cidr       = aws_vpc.main.cidr_block
      dns_enabled = aws_vpc.main.enable_dns_hostnames
    }
    
    # Subnet hierarchy
    subnets = {
      for type in ["public", "private", "database"] :
      type => {
        count = length([
          for k, v in local.all_subnets :
          v if startswith(k, type)
        ])
        
        details = {
          for k, v in local.all_subnets :
          k => {
            cidr = v
            az   = split("-", k)[1]
            
            # Calculate usable IPs
            total_ips   = pow(2, 32 - tonumber(split("/", v)[1]))
            usable_ips  = pow(2, 32 - tonumber(split("/", v)[1])) - 5
            
            # Instance count in this subnet
            instances = length([
              for inst_k, inst in aws_instance.main :
              inst if inst.subnet_id == (
                type == "public" ? 
                  aws_subnet.public[split("-", k)[1]].id :
                  aws_subnet.private[split("-", k)[1]].id
              )
            ])
          }
          if startswith(k, type)
        }
      }
    }
    
    # Network summary
    summary = {
      total_subnets = length(local.all_subnets)
      total_instances = local.total_instance_count
      
      instances_per_subnet_type = {
        for type in ["public", "private"] :
        type => length([
          for k, v in aws_instance.main :
          v if v.subnet_id in (
            type == "public" ?
              [for s in aws_subnet.public : s.id] :
              [for s in aws_subnet.private : s.id]
          )
        ])
      }
    }
  }
}

output "security_matrix" {
  description = "Security group rules matrix"
  value = {
    for sg_name, sg in var.security_groups :
    sg_name => {
      id = aws_security_group.custom[sg_name].id
      
      ingress_summary = {
        total_rules = length(sg.ingress_rules)
        
        by_port = {
          for rule in sg.ingress_rules :
          "${rule.from_port}-${rule.to_port}" => {
            protocol    = rule.protocol
            sources     = length(rule.cidr_blocks)
            description = rule.description
          }
        }
        
        open_to_internet = length([
          for rule in sg.ingress_rules :
          rule if contains(rule.cidr_blocks, "0.0.0.0/0")
        ])
      }
      
      attached_to = [
        for k, v in aws_instance.main :
        {
          instance_id = v.id
          instance_name = k
        }
        if contains(v.vpc_security_group_ids, aws_security_group.custom[sg_name].id)
      ]
    }
  }
}

output "resource_inventory" {
  description = "Complete resource inventory"
  value = {
    timestamp = timestamp()
    environment = var.environment
    
    counts = {
      vpcs              = 1
      subnets           = length(local.all_subnets)
      security_groups   = length(var.security_groups)
      instances         = local.total_instance_count
      nat_gateways      = var.vpc_config.enable_nat_gateway ? length(local.public_subnets) : 0
    }
    
    # Resources by tag
    by_tags = {
      for tag_key in keys(local.common_tags) :
      tag_key => {
        value = local.common_tags[tag_key]
        
        resources = concat(
          [for k, v in aws_instance.main : "instance:${k}"],
          [for k, v in aws_subnet.public : "subnet:${k}"],
          [for k, v in aws_subnet.private : "subnet:${k}"]
        )
      }
    }
    
    # Cost estimation
    estimated_monthly_cost = local.total_monthly_cost
  }
}
```

---

## Заключение и Best Practices

### 📚 Ключевые takeaways

**1. Dynamic Blocks - когда использовать:**

- ✅ Повторяющиеся вложенные блоки (ingress/egress rules)
- ✅ Условные вложенные блоки
- ❌ НЕ использовать для top-level ресурсов (используй `for_each`)

**2. For Expressions - patterns:**

```hcl
# List comprehension
[for item in list : transform(item)]

# Map comprehension
{for key, val in map : key => transform(val)}

# Filtering
[for item in list : item if condition]

# Flattening
flatten([for outer in list : outer.inner])
```

**3. Best Practices:**

- Используй `for_each` вместо `count` для управляемых ресурсов
- Validation rules для всех критичных переменных
- Sensitive data для паролей и ключей
- Locals для сложных вычислений
- Dynamic blocks только для nested blocks

**4. Performance tips:**

- Избегай слишком сложных for expressions (усложняет debugging)
- Используй locals для кеширования тяжелых вычислений
- Flatten nested structures заранее

Ты освоил:

- ✅ Dynamic blocks для гибкой конфигурации
- ✅ For expressions для трансформаций
- ✅ Advanced type constraints
- ✅ Validation rules
- ✅ Complex data transformations

**Следующие шаги:**

1. Практика на реальных проектах
2. Изучение Terraform Cloud / Enterprise
3. CI/CD интеграция
4. Testing (terratest, kitchen-terraform)
5. Policy as Code (Sentinel, OPA)

---

## Модуль 10: Lifecycle, Preconditions & Postconditions (30 минут)

### 🎯 Напоминалка

**Lifecycle Meta-Arguments - контроль поведения ресурсов:**

```hcl
resource "..." "..." {
  # ...
  
  lifecycle {
    create_before_destroy = bool
    prevent_destroy       = bool
    ignore_changes        = [attribute1, attribute2, ...]
    replace_triggered_by  = [resource.id, ...]
    
    precondition {
      condition     = bool
      error_message = string
    }
    
    postcondition {
      condition     = bool
      error_message = string
    }
  }
}
```

**create_before_destroy:**

```hcl
# Полезно для ресурсов, которые не могут быть изменены in-place
resource "aws_launch_configuration" "web" {
  name_prefix   = "web-"
  image_id      = data.aws_ami.ubuntu.id
  instance_type = "t3.micro"
  
  lifecycle {
    create_before_destroy = true
  }
}

# Также полезно для security groups, чтобы избежать
# блокировки трафика во время обновления
resource "aws_security_group" "web" {
  name_prefix = "web-"
  vpc_id      = aws_vpc.main.id
  
  # ...
  
  lifecycle {
    create_before_destroy = true
  }
}
```

**prevent_destroy:**

```hcl
# Защита критичных ресурсов от случайного удаления
resource "aws_s3_bucket" "important_data" {
  bucket = "super-important-data"
  
  lifecycle {
    prevent_destroy = true
  }
}

# Terraform выдаст ошибку при попытке destroy:
# Error: Instance cannot be destroyed
```

**ignore_changes:**

```hcl
# Игнорировать изменения, сделанные вне Terraform
resource "aws_instance" "web" {
  ami           = data.aws_ami.ubuntu.id
  instance_type = "t3.micro"
  
  # Игнорировать изменения конкретных атрибутов
  lifecycle {
    ignore_changes = [
      tags,           # Игнорировать все изменения tags
      user_data,      # Игнорировать user_data
    ]
  }
}

# Игнорировать ВСЕ изменения
resource "aws_instance" "managed_externally" {
  ami           = data.aws_ami.ubuntu.id
  instance_type = "t3.micro"
  
  lifecycle {
    ignore_changes = all
  }
}

# Игнорировать изменения вложенных атрибутов
resource "aws_security_group" "web" {
  name = "web-sg"
  
  ingress {
    from_port   = 80
    to_port     = 80
    protocol    = "tcp"
    cidr_blocks = ["0.0.0.0/0"]
  }
  
  lifecycle {
    # Игнорировать изменения в egress rules
    ignore_changes = [
      egress,
    ]
  }
}
```

**replace_triggered_by (Terraform 1.2+):**

```hcl
# Пересоздать ресурс при изменении другого ресурса
resource "aws_ami_copy" "example" {
  name              = "my-ami-copy"
  source_ami_id     = data.aws_ami.latest.id
  source_ami_region = "us-east-1"
}

resource "aws_instance" "web" {
  ami           = aws_ami_copy.example.id
  instance_type = "t3.micro"
  
  lifecycle {
    # Пересоздать instance при изменении AMI
    replace_triggered_by = [
      aws_ami_copy.example.id
    ]
  }
}

# Можно использовать с null_resource для триггеров
resource "null_resource" "config_version" {
  triggers = {
    config_hash = filemd5("${path.module}/config.yaml")
  }
}

resource "aws_instance" "app" {
  ami           = data.aws_ami.ubuntu.id
  instance_type = "t3.micro"
  
  lifecycle {
    replace_triggered_by = [
      null_resource.config_version
    ]
  }
}
```

**Preconditions - проверки перед созданием:**

```hcl
# Проверка условий перед созданием ресурса
resource "aws_instance" "web" {
  ami           = var.ami_id
  instance_type = var.instance_type
  
  lifecycle {
    # Проверить что AMI существует
    precondition {
      condition     = data.aws_ami.selected.id != ""
      error_message = "AMI ${var.ami_id} not found."
    }
    
    # Проверить что instance type доступен в регионе
    precondition {
      condition = contains(
        data.aws_ec2_instance_types.available.instance_types,
        var.instance_type
      )
      error_message = "Instance type ${var.instance_type} not available in ${data.aws_region.current.name}."
    }
    
    # Проверить что есть доступные subnets
    precondition {
      condition     = length(data.aws_subnets.available.ids) > 0
      error_message = "No subnets available in VPC."
    }
  }
}

# Preconditions в data sources
data "aws_ami" "ubuntu" {
  most_recent = true
  owners      = ["099720109477"]
  
  filter {
    name   = "name"
    values = ["ubuntu/images/hvm-ssd/ubuntu-jammy-22.04-amd64-server-*"]
  }
  
  lifecycle {
    precondition {
      condition     = self.state == "available"
      error_message = "AMI is not in available state."
    }
  }
}
```

**Postconditions - проверки после создания:**

```hcl
# Проверка результата после создания
resource "aws_instance" "web" {
  ami           = data.aws_ami.ubuntu.id
  instance_type = "t3.micro"
  subnet_id     = aws_subnet.public.id
  
  lifecycle {
    # Проверить что instance получил public IP
    postcondition {
      condition     = self.public_ip != ""
      error_message = "Instance did not receive a public IP address."
    }
    
    # Проверить что instance в правильном subnet
    postcondition {
      condition     = self.subnet_id == aws_subnet.public.id
      error_message = "Instance created in wrong subnet."
    }
  }
}

# Postconditions для валидации outputs
resource "aws_db_instance" "main" {
  identifier     = "mydb"
  engine         = "postgres"
  instance_class = "db.t3.micro"
  
  lifecycle {
    postcondition {
      condition     = self.endpoint != ""
      error_message = "Database endpoint is empty."
    }
    
    postcondition {
      condition     = self.status == "available"
      error_message = "Database is not available. Status: ${self.status}"
    }
  }
}

# В outputs
output "db_endpoint" {
  value = aws_db_instance.main.endpoint
  
  precondition {
    condition     = aws_db_instance.main.endpoint != ""
    error_message = "Cannot output empty database endpoint."
  }
}
```

**Комбинирование lifecycle правил:**

```hcl
resource "aws_security_group" "web" {
  name_prefix = "web-"
  vpc_id      = aws_vpc.main.id
  
  ingress {
    from_port   = 80
    to_port     = 80
    protocol    = "tcp"
    cidr_blocks = ["0.0.0.0/0"]
  }
  
  lifecycle {
    # Создать новый перед удалением старого
    create_before_destroy = true
    
    # Игнорировать изменения description (может меняться внешне)
    ignore_changes = [
      description,
      tags["LastModified"]
    ]
    
    # Проверка перед созданием
    precondition {
      condition     = aws_vpc.main.id != ""
      error_message = "VPC must exist before creating security group."
    }
    
    # Проверка после создания
    postcondition {
      condition     = self.id != ""
      error_message = "Security group was not created properly."
    }
  }
}
```

**Use Cases для каждого lifecycle:**

```
create_before_destroy
├── Launch configurations/templates
├── Security groups (чтобы не блокировать трафик)
├── ASG launch templates
└── Любые ресурсы, где downtime недопустим

prevent_destroy
├── Production databases
├── S3 buckets с важными данными
├── State backends
├── KMS keys
└── Route53 hosted zones

ignore_changes
├── Tags, управляемые внешними системами
├── Auto-scaling group size (управляется auto-scaling)
├── User data (изменяется через configuration management)
└── Атрибуты, изменяемые AWS автоматически

replace_triggered_by
├── Instance при изменении AMI
├── Containers при изменении image
├── Config-dependent ресурсы
└── Resources с external dependencies
```

### 💻 Задание

Создай production-ready проект с использованием всех lifecycle features:

**1. Структура проекта:**

```bash
mkdir terraform-lifecycle
cd terraform-lifecycle
mkdir -p configs scripts
```

**2. provider.tf:**

```hcl
terraform {
  required_version = ">= 1.6.0"
  
  required_providers {
    aws = {
      source  = "hashicorp/aws"
      version = "~> 5.0"
    }
    null = {
      source  = "hashicorp/null"
      version = "~> 3.2"
    }
  }
}

provider "aws" {
  region = var.aws_region
  
  default_tags {
    tags = local.common_tags
  }
}
```

**3. variables.tf:**

```hcl
variable "aws_region" {
  description = "AWS region"
  type        = string
  default     = "us-east-1"
}

variable "project_name" {
  description = "Project name"
  type        = string
  
  validation {
    condition     = can(regex("^[a-z][a-z0-9-]*$", var.project_name))
    error_message = "Project name must start with lowercase letter."
  }
}

variable "environment" {
  description = "Environment name"
  type        = string
  
  validation {
    condition     = contains(["dev", "staging", "prod"], var.environment)
    error_message = "Must be dev, staging, or prod."
  }
}

variable "instance_type" {
  description = "EC2 instance type"
  type        = string
  default     = "t3.micro"
}

variable "min_instance_count" {
  description = "Minimum number of instances"
  type        = number
  default     = 1
  
  validation {
    condition     = var.min_instance_count >= 1 && var.min_instance_count <= 10
    error_message = "Min instance count must be between 1 and 10."
  }
}

variable "max_instance_count" {
  description = "Maximum number of instances"
  type        = number
  default     = 3
  
  validation {
    condition     = var.max_instance_count >= var.min_instance_count
    error_message = "Max must be >= min instance count."
  }
}

variable "enable_protection" {
  description = "Enable deletion protection for critical resources"
  type        = bool
  default     = false
}

variable "app_version" {
  description = "Application version (triggers redeployment)"
  type        = string
  default     = "1.0.0"
}
```

**4. data-sources.tf:**

```hcl
data "aws_region" "current" {}
data "aws_caller_identity" "current" {}

# Latest Ubuntu AMI with precondition
data "aws_ami" "ubuntu" {
  most_recent = true
  owners      = ["099720109477"]
  
  filter {
    name   = "name"
    values = ["ubuntu/images/hvm-ssd/ubuntu-jammy-22.04-amd64-server-*"]
  }
  
  filter {
    name   = "virtualization-type"
    values = ["hvm"]
  }
  
  filter {
    name   = "architecture"
    values = ["x86_64"]
  }
  
  lifecycle {
    # Проверка что AMI доступен
    postcondition {
      condition     = self.state == "available"
      error_message = "Selected AMI (${self.id}) is not in available state."
    }
    
    # Проверка что AMI не слишком старый (< 90 дней)
    postcondition {
      condition = timecmp(
        timestamp(),
        timeadd(self.creation_date, "2160h")  # 90 days
      ) < 0
      error_message = "AMI is older than 90 days. Consider using a newer image."
    }
  }
}

# VPC с проверками
data "aws_vpc" "default" {
  default = true
  
  lifecycle {
    postcondition {
      condition     = self.enable_dns_hostnames == true
      error_message = "VPC must have DNS hostnames enabled."
    }
  }
}

# Subnets с валидацией
data "aws_subnets" "public" {
  filter {
    name   = "vpc-id"
    values = [data.aws_vpc.default.id]
  }
  
  filter {
    name   = "default-for-az"
    values = ["true"]
  }
  
  lifecycle {
    postcondition {
      condition     = length(self.ids) >= 2
      error_message = "At least 2 subnets required for high availability."
    }
  }
}

# Availability Zones
data "aws_availability_zones" "available" {
  state = "available"
  
  lifecycle {
    postcondition {
      condition     = length(self.names) >= 2
      error_message = "Region must have at least 2 availability zones."
    }
  }
}
```

**5. locals.tf:**

```hcl
locals {
  name_prefix = "${var.project_name}-${var.environment}"
  
  # Environment-specific configuration
  env_config = {
    dev = {
      protection_enabled = false
      backup_enabled     = false
      monitoring_enabled = false
    }
    staging = {
      protection_enabled = false
      backup_enabled     = true
      monitoring_enabled = true
    }
    prod = {
      protection_enabled = true
      backup_enabled     = true
      monitoring_enabled = true
    }
  }
  
  current_config = local.env_config[var.environment]
  
  common_tags = {
    Project     = var.project_name
    Environment = var.environment
    ManagedBy   = "Terraform"
    Region      = data.aws_region.current.name
    AppVersion  = var.app_version
  }
  
  # User data hash для triggering updates
  user_data_hash = md5(templatefile("${path.module}/configs/user_data.sh", {
    environment = var.environment
    app_version = var.app_version
    region      = data.aws_region.current.name
  }))
}
```

**6. configs/user_data.sh:**

```bash
#!/bin/bash
set -e

echo "=== User Data Script Started ==="
echo "Environment: ${environment}"
echo "App Version: ${app_version}"
echo "Region: ${region}"

# Update system
export DEBIAN_FRONTEND=noninteractive
apt-get update
apt-get upgrade -y

# Install packages
apt-get install -y nginx curl jq

# Create app info page
cat > /var/www/html/index.html <<EOF
<!DOCTYPE html>
<html>
<head>
    <title>Lifecycle Demo</title>
    <style>
        body { font-family: Arial; margin: 40px; background: #f5f5f5; }
        .container { background: white; padding: 30px; border-radius: 10px; box-shadow: 0 2px 4px rgba(0,0,0,0.1); }
        .success { color: #28a745; font-size: 24px; }
        .info { margin: 20px 0; }
        .info strong { display: inline-block; width: 150px; }
    </style>
</head>
<body>
    <div class="container">
        <h1 class="success">✓ Instance Running</h1>
        <div class="info">
            <p><strong>Environment:</strong> ${environment}</p>
            <p><strong>App Version:</strong> ${app_version}</p>
            <p><strong>Region:</strong> ${region}</p>
            <p><strong>Instance ID:</strong> $(ec2-metadata --instance-id | cut -d ' ' -f 2)</p>
            <p><strong>Private IP:</strong> $(hostname -I)</p>
            <p><strong>Deployed:</strong> $(date)</p>
        </div>
    </div>
</body>
</html>
EOF

# Start nginx
systemctl enable nginx
systemctl start nginx

echo "=== User Data Script Completed ==="
```

**7. security-groups.tf:**

```hcl
# Security Group с create_before_destroy
resource "aws_security_group" "web" {
  name_prefix = "${local.name_prefix}-web-"
  description = "Security group for web servers"
  vpc_id      = data.aws_vpc.default.id
  
  lifecycle {
    # Создать новый SG перед удалением старого
    # чтобы избежать блокировки трафика
    create_before_destroy = true
    
    # Игнорировать изменения в description
    # (может обновляться внешними системами)
    ignore_changes = [
      description
    ]
    
    # Проверка перед созданием
    precondition {
      condition     = data.aws_vpc.default.id != ""
      error_message = "VPC must exist before creating security group."
    }
    
    # Проверка после создания
    postcondition {
      condition     = length(self.ingress) > 0
      error_message = "Security group must have at least one ingress rule."
    }
  }
  
  tags = {
    Name = "${local.name_prefix}-web-sg"
  }
}

resource "aws_vpc_security_group_ingress_rule" "http" {
  security_group_id = aws_security_group.web.id
  
  from_port   = 80
  to_port     = 80
  ip_protocol = "tcp"
  cidr_ipv4   = "0.0.0.0/0"
  description = "HTTP from anywhere"
}

resource "aws_vpc_security_group_ingress_rule" "https" {
  security_group_id = aws_security_group.web.id
  
  from_port   = 443
  to_port     = 443
  ip_protocol = "tcp"
  cidr_ipv4   = "0.0.0.0/0"
  description = "HTTPS from anywhere"
}

resource "aws_vpc_security_group_ingress_rule" "ssh" {
  security_group_id = aws_security_group.web.id
  
  from_port   = 22
  to_port     = 22
  ip_protocol = "tcp"
  cidr_ipv4   = "0.0.0.0/0"
  description = "SSH from anywhere"
}

resource "aws_vpc_security_group_egress_rule" "all" {
  security_group_id = aws_security_group.web.id
  
  ip_protocol = "-1"
  cidr_ipv4   = "0.0.0.0/0"
  description = "All outbound traffic"
}
```

**8. launch-template.tf:**

```hcl
# Trigger для пересоздания instances при изменении конфигурации
resource "null_resource" "config_version" {
  triggers = {
    user_data_hash = local.user_data_hash
    app_version    = var.app_version
    ami_id         = data.aws_ami.ubuntu.id
  }
}

# Launch Template с create_before_destroy
resource "aws_launch_template" "web" {
  name_prefix   = "${local.name_prefix}-web-"
  image_id      = data.aws_ami.ubuntu.id
  instance_type = var.instance_type
  
  vpc_security_group_ids = [aws_security_group.web.id]
  
  user_data = base64encode(templatefile("${path.module}/configs/user_data.sh", {
    environment = var.environment
    app_version = var.app_version
    region      = data.aws_region.current.name
  }))
  
  monitoring {
    enabled = local.current_config.monitoring_enabled
  }
  
  metadata_options {
    http_endpoint               = "enabled"
    http_tokens                 = "required"  # IMDSv2
    http_put_response_hop_limit = 1
  }
  
  tag_specifications {
    resource_type = "instance"
    tags = merge(local.common_tags, {
      Name = "${local.name_prefix}-web"
    })
  }
  
  lifecycle {
    create_before_destroy = true
    
    # Игнорировать изменения в tag_specifications
    # (может обновляться AWS автоматически)
    ignore_changes = [
      tag_specifications[0].tags["LastModified"]
    ]
    
    precondition {
      condition     = var.instance_type != ""
      error_message = "Instance type must be specified."
    }
    
    postcondition {
      condition     = self.id != ""
      error_message = "Launch template was not created."
    }
  }
  
  tags = {
    Name = "${local.name_prefix}-web-lt"
  }
}
```

**9. autoscaling.tf:**

```hcl
# Auto Scaling Group
resource "aws_autoscaling_group" "web" {
  name_prefix         = "${local.name_prefix}-web-"
  vpc_zone_identifier = data.aws_subnets.public.ids
  
  min_size         = var.min_instance_count
  max_size         = var.max_instance_count
  desired_capacity = var.min_instance_count
  
  health_check_type         = "ELB"
  health_check_grace_period = 300
  
  launch_template {
    id      = aws_launch_template.web.id
    version = "$Latest"
  }
  
  # Target group attachments (если есть ALB)
  target_group_arns = try([aws_lb_target_group.web[0].arn], [])
  
  # Lifecycle управление
  lifecycle {
    # Игнорировать изменения в desired_capacity и target_group_arns
    # (управляется auto-scaling policies)
    ignore_changes = [
      desired_capacity,
      target_group_arns
    ]
    
    # Проверка перед созданием
    precondition {
      condition     = var.min_instance_count <= var.max_instance_count
      error_message = "Min size must be <= max size."
    }
    
    precondition {
      condition     = length(data.aws_subnets.public.ids) >= 2
      error_message = "At least 2 subnets required for ASG."
    }
    
    # Проверка после создания
    postcondition {
      condition     = self.min_size >= 1
      error_message = "ASG must have min_size >= 1."
    }
  }
  
  tag {
    key                 = "Name"
    value               = "${local.name_prefix}-web-asg"
    propagate_at_launch = true
  }
  
  dynamic "tag" {
    for_each = local.common_tags
    content {
      key                 = tag.key
      value               = tag.value
      propagate_at_launch = true
    }
  }
}

# Auto Scaling Policy
resource "aws_autoscaling_policy" "scale_up" {
  name                   = "${local.name_prefix}-scale-up"
  autoscaling_group_name = aws_autoscaling_group.web.name
  
  policy_type            = "TargetTrackingScaling"
  estimated_instance_warmup = 300
  
  target_tracking_configuration {
    predefined_metric_specification {
      predefined_metric_type = "ASGAverageCPUUtilization"
    }
    target_value = 70.0
  }
}
```

**10. load-balancer.tf:**

```hcl
# Application Load Balancer (опционально)
resource "aws_lb" "web" {
  count = var.environment == "prod" ? 1 : 0
  
  name_prefix        = substr("${local.name_prefix}-", 0, 6)
  internal           = false
  load_balancer_type = "application"
  security_groups    = [aws_security_group.web.id]
  subnets            = data.aws_subnets.public.ids
  
  enable_deletion_protection = local.current_config.protection_enabled
  
  lifecycle {
    # Защита от удаления в production
    prevent_destroy = false  # Установить в true для production!
    
    precondition {
      condition     = length(data.aws_subnets.public.ids) >= 2
      error_message = "ALB requires at least 2 subnets in different AZs."
    }
    
    postcondition {
      condition     = self.arn != ""
      error_message = "ALB was not created properly."
    }
  }
  
  tags = {
    Name = "${local.name_prefix}-web-alb"
  }
}

resource "aws_lb_target_group" "web" {
  count = var.environment == "prod" ? 1 : 0
  
  name_prefix = substr("${local.name_prefix}-", 0, 6)
  port        = 80
  protocol    = "HTTP"
  vpc_id      = data.aws_vpc.default.id
  
  health_check {
    enabled             = true
    healthy_threshold   = 2
    unhealthy_threshold = 2
    timeout             = 5
    interval            = 30
    path                = "/"
    matcher             = "200"
  }
  
  lifecycle {
    create_before_destroy = true
    
    postcondition {
      condition     = self.health_check[0].enabled == true
      error_message = "Health check must be enabled."
    }
  }
  
  tags = {
    Name = "${local.name_prefix}-web-tg"
  }
}

resource "aws_lb_listener" "http" {
  count = var.environment == "prod" ? 1 : 0
  
  load_balancer_arn = aws_lb.web[0].arn
  port              = 80
  protocol          = "HTTP"
  
  default_action {
    type             = "forward"
    target_group_arn = aws_lb_target_group.web[0].arn
  }
  
  lifecycle {
    postcondition {
      condition     = self.port == 80
      error_message = "HTTP listener must be on port 80."
    }
  }
}
```

**11. s3-state-backend.tf:**

```hcl
# S3 bucket для state (пример защищенного ресурса)
resource "aws_s3_bucket" "terraform_state" {
  bucket_prefix = "${local.name_prefix}-tfstate-"
  
  lifecycle {
    # КРИТИЧНО: Защита от удаления
    prevent_destroy = false  # Установить в true для production!
    
    # Игнорировать изменения в lifecycle правилах
    # (могут управляться отдельно)
    ignore_changes = [
      lifecycle_rule
    ]
    
    precondition {
      condition     = var.project_name != ""
      error_message = "Project name required for state bucket."
    }
    
    postcondition {
      condition     = self.bucket != ""
      error_message = "Bucket was not created."
    }
  }
  
  tags = {
    Name    = "${local.name_prefix}-tfstate"
    Purpose = "TerraformState"
  }
}

resource "aws_s3_bucket_versioning" "terraform_state" {
  bucket = aws_s3_bucket.terraform_state.id
  
  versioning_configuration {
    status = "Enabled"
  }
  
  lifecycle {
    precondition {
      condition     = aws_s3_bucket.terraform_state.id != ""
      error_message = "S3 bucket must exist before enabling versioning."
    }
  }
}

resource "aws_s3_bucket_server_side_encryption_configuration" "terraform_state" {
  bucket = aws_s3_bucket.terraform_state.id
  
  rule {
    apply_server_side_encryption_by_default {
      sse_algorithm = "AES256"
    }
  }
  
  lifecycle {
    postcondition {
      condition     = self.rule[0].apply_server_side_encryption_by_default[0].sse_algorithm == "AES256"
      error_message = "Encryption must be enabled with AES256."
    }
  }
}

resource "aws_s3_bucket_public_access_block" "terraform_state" {
  bucket = aws_s3_bucket.terraform_state.id
  
  block_public_acls       = true
  block_public_policy     = true
  ignore_public_acls      = true
  restrict_public_buckets = true
  
  lifecycle {
    postcondition {
      condition     = self.block_public_acls == true
      error_message = "Public access must be blocked."
    }
  }
}
```

**12. outputs.tf:**

```hcl
output "asg_info" {
  description = "Auto Scaling Group information"
  value = {
    id           = aws_autoscaling_group.web.id
    name         = aws_autoscaling_group.web.name
    min_size     = aws_autoscaling_group.web.min_size
    max_size     = aws_autoscaling_group.web.max_size
    desired_size = aws_autoscaling_group.web.desired_capacity
  }
  
  precondition {
    condition     = aws_autoscaling_group.web.id != ""
    error_message = "ASG must be created before outputting info." } }

 output "launch_template_info" {
	description = "Launch template information"
	value = {
	id           = aws_launch_template.web.id
	latest_version = aws_launch_template.web.latest_version
	ami_id       = data.aws_ami.ubuntu.id
	ami_name     = data.aws_ami.ubuntu.name
}
}
 output "alb_dns" {
	description = "ALB DNS name (if created)"
	value       = try(aws_lb.web[0].dns_name, "Not created (non-prod environment)")
}
  output "protection_status" {
	description = "Protection status of resources"
	value = {
	environment          = var.environment
	deletion_protection  = local.current_config.protection_enabled
	backup_enabled      = local.current_config.backup_enabled
	monitoring_enabled  = local.current_config.monitoring_enabled
}
}
  output "deployment_info" {
	description = "Deployment information"
	value = {
	app_version     = var.app_version
	user_data_hash  = local.user_data_hash
	deployed_at     = timestamp()
	ami_id          = data.aws_ami.ubuntu.id
	ami_created     = data.aws_ami.ubuntu.creation_date
}
}
```

**13. terraform.tfvars:**

```hcl
project_name = "lifecycle-demo"
environment  = "dev"

instance_type       = "t3.micro"
min_instance_count = 2
max_instance_count = 4

enable_protection = false

app_version = "1.0.0"
````

**14. Makefile для удобства:**

```makefile
# Makefile

.PHONY: help init plan apply destroy fmt validate clean

help:
	@echo "Available targets:"
	@echo "  init       - Initialize Terraform"
	@echo "  plan       - Run terraform plan"
	@echo "  apply      - Apply changes"
	@echo "  destroy    - Destroy infrastructure"
	@echo "  fmt        - Format code"
	@echo "  validate   - Validate configuration"
	@echo "  clean      - Clean terraform files"

init:
	terraform init

plan:
	terraform plan -out=tfplan

apply:
	terraform apply tfplan

destroy:
	terraform destroy

fmt:
	terraform fmt -recursive

validate:
	terraform validate

clean:
	rm -rf .terraform terraform.tfstate* tfplan .terraform.lock.hcl

# Environment-specific targets
plan-prod:
	terraform plan -var-file=prod.tfvars -out=tfplan

apply-prod:
	terraform apply -var-file=prod.tfvars

# Update app version and trigger redeployment
update-version:
	@read -p "Enter new version: " version; \
	sed -i '' "s/app_version = .*/app_version = \"$$version\"/" terraform.tfvars; \
	terraform plan -out=tfplan
```

**15. Запуск и тестирование:**

```bash
# Инициализация
terraform init

# Валидация
terraform validate

# Форматирование
terraform fmt -recursive

# План
terraform plan -out=tfplan

# Применить
terraform apply tfplan

# Проверить созданные ресурсы
terraform state list

# Проверить ASG
aws autoscaling describe-auto-scaling-groups \
  --auto-scaling-group-names $(terraform output -raw asg_info | jq -r '.name')

# Проверить instances
aws ec2 describe-instances \
  --filters "Name=tag:Project,Values=lifecycle-demo"

# Проверить launch template versions
aws ec2 describe-launch-template-versions \
  --launch-template-id $(terraform output -json launch_template_info | jq -r '.id')
```

**16. Тестирование lifecycle features:**

```bash
# Test 1: Update app version (должен пересоздать instances)
echo 'app_version = "1.1.0"' >> terraform.tfvars
terraform plan
# Должно показать replacement instances из-за replace_triggered_by

# Test 2: Попытка удалить защищенный ресурс (если enable_protection = true)
# Раскомментируй prevent_destroy = true в s3-state-backend.tf
terraform destroy
# Должно выдать ошибку: "Instance cannot be destroyed"

# Test 3: Изменить desired_capacity вручную
aws autoscaling set-desired-capacity \
  --auto-scaling-group-name $(terraform output -raw asg_info | jq -r '.name') \
  --desired-capacity 3

# Plan не должен показывать изменений (ignore_changes работает)
terraform plan

# Test 4: Проверка preconditions
# Попробуй установить min > max
echo 'min_instance_count = 5' >> terraform.tfvars
echo 'max_instance_count = 3' >> terraform.tfvars
terraform plan
# Должно выдать ошибку из precondition

# Test 5: Изменить security group description вручную
aws ec2 modify-security-group-rules \
  --group-id $(terraform state show aws_security_group.web | grep "id " | awk '{print $3}' | tr -d '"')

# Plan не должен показывать изменений (ignore_changes)
terraform plan
```

### 🚀 Бонус (продвинутое)

**1. Custom Validation Functions:**

```hcl
# custom-validations.tf

variable "cidr_blocks" {
  type = list(string)
  
  validation {
    # Проверка что все CIDR блоки валидны
    condition = alltrue([
      for cidr in var.cidr_blocks :
      can(cidrhost(cidr, 0))
    ])
    error_message = "All CIDR blocks must be valid."
  }
  
  validation {
    # Проверка что нет пересечений
    condition = length(var.cidr_blocks) == length(distinct(var.cidr_blocks))
    error_message = "CIDR blocks must be unique."
  }
}

variable "instance_tags" {
  type = map(string)
  
  validation {
    # Проверка обязательных тегов
    condition = alltrue([
      contains(keys(var.instance_tags), "Environment"),
      contains(keys(var.instance_tags), "Owner")
    ])
    error_message = "Tags must include Environment and Owner."
  }
  
  validation {
    # Проверка формата тегов
    condition = alltrue([
      for k, v in var.instance_tags :
      can(regex("^[A-Z][a-zA-Z0-9]*$", k))
    ])
    error_message = "Tag keys must start with capital letter."
  }
}
```

**2. Complex Preconditions with Data Sources:**

```hcl
# complex-preconditions.tf

data "aws_ec2_instance_type" "selected" {
  instance_type = var.instance_type
  
  lifecycle {
    postcondition {
      condition     = self.supported_architectures[0] == "x86_64"
      error_message = "Instance type must support x86_64 architecture."
    }
    
    postcondition {
      condition     = self.ebs_optimized_support == "default"
      error_message = "Instance type must support EBS optimization."
    }
    
    postcondition {
      condition     = contains(self.supported_virtualization_types, "hvm")
      error_message = "Instance type must support HVM virtualization."
    }
  }
}

resource "aws_instance" "validated" {
  ami           = data.aws_ami.ubuntu.id
  instance_type = var.instance_type
  
  lifecycle {
    precondition {
      condition = data.aws_ec2_instance_type.selected.memory_size >= 1024
      error_message = "Instance type must have at least 1GB RAM. Current: ${data.aws_ec2_instance_type.selected.memory_size}MB"
    }
    
    precondition {
      condition     = data.aws_ec2_instance_type.selected.default_vcpus >= 1
      error_message = "Instance must have at least 1 vCPU."
    }
    
    # Проверка стоимости (примерная)
    precondition {
      condition = (
        var.environment == "prod" ||
        data.aws_ec2_instance_type.selected.default_vcpus <= 2
      )
      error_message = "Non-prod environments limited to 2 vCPUs max."
    }
  }
}
```

**3. Time-based Conditions:**

```hcl
# time-based-conditions.tf

locals {
  current_hour = formatdate("hh", timestamp())
  current_day  = formatdate("EEEE", timestamp())
  
  # Разрешить деструктивные операции только в рабочие часы
  is_business_hours = (
    !contains(["Saturday", "Sunday"], local.current_day) &&
    tonumber(local.current_hour) >= 9 &&
    tonumber(local.current_hour) <= 17
  )
}

resource "aws_instance" "time_protected" {
  ami           = data.aws_ami.ubuntu.id
  instance_type = "t3.micro"
  
  lifecycle {
    precondition {
      condition = (
        var.environment != "prod" ||
        local.is_business_hours
      )
      error_message = "Production changes only allowed during business hours (Mon-Fri, 9AM-5PM)."
    }
  }
}
```

**4. Cross-Resource Validations:**

```hcl
# cross-resource-validations.tf

resource "aws_db_instance" "main" {
  identifier     = "${local.name_prefix}-db"
  engine         = "postgres"
  instance_class = "db.t3.micro"
  
  vpc_security_group_ids = [aws_security_group.db.id]
  db_subnet_group_name   = aws_db_subnet_group.main.name
  
  lifecycle {
    precondition {
      condition     = aws_security_group.db.id != ""
      error_message = "Database security group must exist."
    }
    
    precondition {
      condition     = length(aws_db_subnet_group.main.subnet_ids) >= 2
      error_message = "Database requires subnets in at least 2 AZs."
    }
    
    postcondition {
      condition     = self.endpoint != ""
      error_message = "Database endpoint not available."
    }
    
    postcondition {
      condition     = self.status == "available"
      error_message = "Database not in available state: ${self.status}"
    }
  }
}

resource "aws_instance" "app" {
  ami           = data.aws_ami.ubuntu.id
  instance_type = "t3.micro"
  
  lifecycle {
    precondition {
      condition     = aws_db_instance.main.endpoint != ""
      error_message = "Database must be ready before creating app server."
    }
    
    replace_triggered_by = [
      aws_db_instance.main.endpoint
    ]
  }
  
  user_data = templatefile("app_config.sh", {
    db_endpoint = aws_db_instance.main.endpoint
  })
}
```

**5. Environment-Specific Lifecycle Rules:**

```hcl
# environment-lifecycle.tf

locals {
  lifecycle_config = {
    dev = {
      prevent_destroy       = false
      create_before_destroy = false
      enable_protection     = false
    }
    staging = {
      prevent_destroy       = false
      create_before_destroy = true
      enable_protection     = false
    }
    prod = {
      prevent_destroy       = true
      create_before_destroy = true
      enable_protection     = true
    }
  }
  
  current_lifecycle = local.lifecycle_config[var.environment]
}

resource "aws_db_instance" "adaptive" {
  identifier     = "${local.name_prefix}-db"
  engine         = "postgres"
  instance_class = "db.t3.micro"
  
  # Адаптивная защита от удаления
  deletion_protection = local.current_lifecycle.enable_protection
  
  lifecycle {
    # Динамическая защита (к сожалению, не поддерживается напрямую)
    # Workaround через depends_on и условные ресурсы
    
    precondition {
      condition = (
        var.environment != "prod" ||
        var.enable_protection == true
      )
      error_message = "Protection must be enabled in production."
    }
    
    postcondition {
      condition = (
        var.environment != "prod" ||
        self.deletion_protection == true
      )
      error_message = "Deletion protection must be enabled in production."
    }
  }
}

# Conditional protection resource
resource "null_resource" "prod_protection_check" {
  count = var.environment == "prod" ? 1 : 0
  
  lifecycle {
    precondition {
      condition     = var.enable_protection == true
      error_message = "Production environment requires enable_protection = true."
    }
  }
}
```

**6. Comprehensive Health Checks:**

```hcl
# health-checks.tf

resource "aws_instance" "health_checked" {
  ami           = data.aws_ami.ubuntu.id
  instance_type = var.instance_type
  
  lifecycle {
    # Pre-launch checks
    precondition {
      condition     = data.aws_ami.ubuntu.state == "available"
      error_message = "AMI must be available."
    }
    
    precondition {
      condition = timecmp(
        timestamp(),
        timeadd(data.aws_ami.ubuntu.creation_date, "2160h")  # 90 days
      ) < 0
      error_message = "AMI is older than 90 days."
    }
    
    # Post-launch checks
    postcondition {
      condition     = self.instance_state == "running"
      error_message = "Instance failed to reach running state."
    }
    
    postcondition {
      condition     = self.public_ip != "" || !self.associate_public_ip_address
      error_message = "Instance should have public IP when requested."
    }
    
    postcondition {
      condition     = length(self.security_groups) > 0
      error_message = "Instance must have at least one security group."
    }
  }
}

# Health check using external script
data "external" "instance_health" {
  depends_on = [aws_instance.health_checked]
  
  program = ["bash", "-c", <<-EOT
    instance_id="${aws_instance.health_checked.id}"
    
    # Wait for instance to be ready
    aws ec2 wait instance-running --instance-ids $instance_id
    
    # Check system status
    status=$(aws ec2 describe-instance-status \
      --instance-ids $instance_id \
      --query 'InstanceStatuses[0].SystemStatus.Status' \
      --output text)
    
    if [ "$status" == "ok" ]; then
      echo '{"status": "healthy", "instance_id": "'$instance_id'"}'
    else
      echo '{"status": "unhealthy", "instance_id": "'$instance_id'"}' >&2
      exit 1
    fi
  EOT
  ]
}

output "instance_health" {
  value = data.external.instance_health.result
  
  precondition {
    condition     = data.external.instance_health.result.status == "healthy"
    error_message = "Instance health check failed."
  }
}
```

---

## Заключение Модуля 10

### 📚 Ключевые takeaways

**1. Lifecycle Meta-Arguments:**

- ✅ `create_before_destroy` - для zero-downtime updates
- ✅ `prevent_destroy` - защита критичных ресурсов
- ✅ `ignore_changes` - для атрибутов, управляемых извне
- ✅ `replace_triggered_by` - явные триггеры пересоздания

**2. Preconditions/Postconditions:**

- ✅ Ранняя валидация входных данных
- ✅ Проверка состояния после создания
- ✅ Защита от некорректных конфигураций
- ✅ Понятные сообщения об ошибках

**3. Best Practices:**

```hcl
# ✅ DO: Защищай критичные ресурсы
resource "aws_s3_bucket" "important" {
  lifecycle {
    prevent_destroy = true
  }
}

# ✅ DO: Используй create_before_destroy для zero-downtime
resource "aws_launch_template" "web" {
  lifecycle {
    create_before_destroy = true
  }
}

# ✅ DO: Игнорируй auto-managed атрибуты
resource "aws_autoscaling_group" "web" {
  lifecycle {
    ignore_changes = [desired_capacity]
  }
}

# ✅ DO: Добавляй понятные error messages
lifecycle {
  precondition {
    condition     = var.count >= 1
    error_message = "Count must be at least 1, got ${var.count}."
  }
}

# ❌ DON'T: Не злоупотребляй ignore_changes
lifecycle {
  ignore_changes = all  # Плохая практика!
}
```

**4. Когда использовать каждый lifecycle:**

|Feature|Use Case|Example|
|---|---|---|
|`create_before_destroy`|Zero-downtime updates|Security groups, Launch templates|
|`prevent_destroy`|Critical data protection|Databases, S3 buckets, KMS keys|
|`ignore_changes`|External management|ASG capacity, tags from other systems|
|`replace_triggered_by`|Forced replacement|Instance when AMI changes|
|`precondition`|Input validation|Check requirements before creation|
|`postcondition`|State validation|Verify resource created correctly|

Ты освоил:

- ✅ Lifecycle meta-arguments для управления ресурсами
- ✅ Preconditions для ранней валидации
- ✅ Postconditions для проверки результатов
- ✅ Best practices для production использования
- ✅ Advanced patterns и примеры


## Модуль 11: Testing и Validation (30 минут)

### 🎯 Напоминалка

**Зачем тестировать Terraform код:**

```
Testing Strategy
├── Syntax Validation (terraform validate)
├── Linting (TFLint, tfsec)
├── Security Scanning (Checkov, Terrascan)
├── Unit Testing (Terratest, kitchen-terraform)
├── Integration Testing (Terratest)
├── Policy as Code (OPA, Sentinel)
└── Pre-commit Hooks (автоматизация)
```

**Уровни тестирования:**

```
Level 1: Static Analysis
├── terraform fmt (форматирование)
├── terraform validate (синтаксис)
├── TFLint (best practices)
└── tfsec (security issues)

Level 2: Policy Checks
├── Checkov (security & compliance)
├── Terrascan (policy violations)
└── OPA (custom policies)

Level 3: Integration Tests
├── Terratest (Go-based testing)
├── kitchen-terraform (Ruby-based)
└── pytest-terraform (Python-based)

Level 4: Automated Checks
├── Pre-commit hooks
├── CI/CD pipelines
└── Pull request checks
```

**TFLint - Terraform Linter:**

```bash
# Установка
brew install tflint  # macOS
curl -s https://raw.githubusercontent.com/terraform-linters/tflint/master/install_linux.sh | bash  # Linux

# Базовое использование
tflint
tflint --init  # Загрузить plugins
tflint --recursive  # Рекурсивно по директориям

# С конкигом
tflint --config=.tflint.hcl
```

**TFLint Configuration:**

```hcl
# .tflint.hcl
config {
  module = true
  force = false
  disabled_by_default = false
}

plugin "aws" {
  enabled = true
  version = "0.27.0"
  source  = "github.com/terraform-linters/tflint-ruleset-aws"
}

# Custom rules
rule "terraform_naming_convention" {
  enabled = true
}

rule "terraform_required_version" {
  enabled = true
}

rule "terraform_required_providers" {
  enabled = true
}

# AWS-specific rules
rule "aws_instance_invalid_type" {
  enabled = true
}

rule "aws_instance_previous_type" {
  enabled = true
}

rule "aws_resource_missing_tags" {
  enabled = true
  tags = ["Environment", "Owner", "Project"]
}
```

**Checkov - Security & Compliance Scanner:**

```bash
# Установка
pip install checkov

# Использование
checkov -d .                    # Scan current directory
checkov -f main.tf              # Scan specific file
checkov --framework terraform   # Only Terraform
checkov --check CKV_AWS_79      # Specific check

# Output formats
checkov -d . -o json            # JSON output
checkov -d . -o junitxml        # JUnit XML
checkov -d . -o sarif           # SARIF format

# Skip checks
checkov -d . --skip-check CKV_AWS_79
checkov -d . --skip-framework dockerfile

# Baseline
checkov -d . --baseline checkov_baseline.json
```

**Checkov Inline Suppressions:**

```hcl
# Suppress specific check for resource
resource "aws_s3_bucket" "example" {
  bucket = "my-bucket"
  
  #checkov:skip=CKV_AWS_18:Ensure the S3 bucket has access logging enabled
  #checkov:skip=CKV_AWS_144:Ensure S3 bucket has cross-region replication enabled
}

# Suppress for module
#checkov:skip=CKV_AWS_18:Logging handled externally
module "vpc" {
  source = "./modules/vpc"
  # ...
}
```

**tfsec - Security Scanner:**

```bash
# Установка
brew install tfsec  # macOS
curl -s https://raw.githubusercontent.com/aquasecurity/tfsec/master/scripts/install_linux.sh | bash

# Использование
tfsec .
tfsec --soft-fail  # Don't exit with error
tfsec --format json
tfsec --format junit
tfsec --format sarif

# Exclude checks
tfsec --exclude aws-s3-enable-versioning
tfsec --exclude-downloaded-modules

# Config file
tfsec --config-file tfsec.yml
```

**tfsec Configuration:**

```yaml
# tfsec.yml
exclude:
  - aws-s3-enable-versioning
  - aws-ec2-enable-at-rest-encryption

severity_overrides:
  aws-s3-enable-bucket-logging: WARNING

minimum_severity: MEDIUM

exclude_paths:
  - modules/legacy/**
  - examples/**
```

**Terratest - Go-based Testing Framework:**

```go
// test/terraform_aws_example_test.go
package test

import (
    "testing"
    "github.com/gruntwork-io/terratest/modules/terraform"
    "github.com/stretchr/testify/assert"
)

func TestTerraformAwsExample(t *testing.T) {
    t.Parallel()
    
    terraformOptions := terraform.WithDefaultRetryableErrors(t, &terraform.Options{
        TerraformDir: "../examples/basic",
        Vars: map[string]interface{}{
            "instance_type": "t3.micro",
            "environment":   "test",
        },
    })
    
    defer terraform.Destroy(t, terraformOptions)
    
    terraform.InitAndApply(t, terraformOptions)
    
    // Validate outputs
    instanceID := terraform.Output(t, terraformOptions, "instance_id")
    assert.NotEmpty(t, instanceID)
    
    vpcID := terraform.Output(t, terraformOptions, "vpc_id")
    assert.Regexp(t, "^vpc-", vpcID)
}
```

**Pre-commit Hooks:**

```yaml
# .pre-commit-config.yaml
repos:
  - repo: https://github.com/antonbabenko/pre-commit-terraform
    rev: v1.83.5
    hooks:
      - id: terraform_fmt
      - id: terraform_validate
      - id: terraform_docs
      - id: terraform_tflint
        args:
          - --args=--config=__GIT_WORKING_DIR__/.tflint.hcl
      - id: terraform_tfsec
      - id: terraform_checkov
        args:
          - --args=--quiet
          - --args=--skip-check CKV_AWS_18
      - id: terrascan
        args:
          - --args=--non-recursive
          - --args=--policy-type aws
```

### 💻 Задание

Создай полноценный testing pipeline для Terraform проекта:

**1. Структура проекта:**

```bash
mkdir terraform-testing
cd terraform-testing
mkdir -p {modules,test,examples,.github/workflows}
```

**2. Установка инструментов:**

```bash
# TFLint
brew install tflint  # или curl script для Linux
tflint --init

# Checkov
pip3 install checkov

# tfsec
brew install tfsec

# Terratest (требует Go)
go version  # Проверить наличие Go

# Pre-commit
pip3 install pre-commit
```

**3. .tflint.hcl:**

```hcl
config {
  module = true
  force = false
  disabled_by_default = false
  
  # Игнорировать определенные директории
  ignore_module = {
    "terraform-aws-modules/vpc/aws"            = true
    "terraform-aws-modules/security-group/aws" = true
  }
}

# AWS Plugin
plugin "aws" {
  enabled = true
  version = "0.27.0"
  source  = "github.com/terraform-linters/tflint-ruleset-aws"
}

# Terraform Plugin
plugin "terraform" {
  enabled = true
  preset  = "recommended"
}

# Custom rules
rule "terraform_naming_convention" {
  enabled = true
  format  = "snake_case"
}

rule "terraform_required_version" {
  enabled = true
}

rule "terraform_required_providers" {
  enabled = true
}

rule "terraform_typed_variables" {
  enabled = true
}

rule "terraform_unused_declarations" {
  enabled = true
}

rule "terraform_comment_syntax" {
  enabled = true
}

rule "terraform_deprecated_index" {
  enabled = true
}

rule "terraform_documented_outputs" {
  enabled = true
}

rule "terraform_documented_variables" {
  enabled = true
}

rule "terraform_module_pinned_source" {
  enabled = true
  style   = "semver"
}

# AWS Rules
rule "aws_instance_invalid_type" {
  enabled = true
}

rule "aws_instance_previous_type" {
  enabled = true
}

rule "aws_resource_missing_tags" {
  enabled = true
  tags    = ["Environment", "Owner", "Project", "ManagedBy"]
}

rule "aws_db_instance_invalid_type" {
  enabled = true
}

rule "aws_s3_bucket_name" {
  enabled = true
  regex   = "^[a-z0-9][a-z0-9-]*[a-z0-9]$"
}
```

**4. .checkov.yml:**

```yaml
# Checkov configuration
framework:
  - terraform

skip-check:
  # S3 logging can be handled externally
  - CKV_AWS_18
  # VPC flow logs in centralized account
  - CKV_AWS_356
  # CloudWatch logs retention is managed separately
  - CKV_AWS_158

quiet: false
compact: false
output: cli

soft-fail: false

# External checks directory
external-checks-dir:
  - ./custom-checks

# Baseline file for CI/CD
baseline: .checkov.baseline

# Custom policies
policy-metadata-filter: "CKV_CUSTOM_*"

# Output formats
output-format:
  - cli
  - json
  - junitxml
  - github_failed_only

# Severity threshold
check-level: warning
```

**5. .tfsec.json:**

```json
{
  "severity_overrides": {
    "aws-s3-enable-bucket-logging": "WARNING",
    "aws-vpc-no-excessive-port-access": "ERROR"
  },
  "minimum_severity": "MEDIUM",
  "exclude": [
    "aws-s3-enable-versioning"
  ],
  "exclude_paths": [
    "modules/legacy/**",
    "examples/**",
    "test/**"
  ],
  "tfvars_file": [
    "terraform.tfvars",
    "prod.tfvars"
  ]
}
```

**6. Пример модуля для тестирования:**

```hcl
# modules/secure-s3/variables.tf
variable "bucket_name" {
  description = "Name of the S3 bucket"
  type        = string
  
  validation {
    condition     = can(regex("^[a-z0-9][a-z0-9-]*[a-z0-9]$", var.bucket_name))
    error_message = "Bucket name must be lowercase alphanumeric with hyphens."
  }
}

variable "environment" {
  description = "Environment name"
  type        = string
  
  validation {
    condition     = contains(["dev", "staging", "prod"], var.environment)
    error_message = "Must be dev, staging, or prod."
  }
}

variable "enable_versioning" {
  description = "Enable bucket versioning"
  type        = bool
  default     = true
}

variable "enable_encryption" {
  description = "Enable server-side encryption"
  type        = bool
  default     = true
}

variable "enable_logging" {
  description = "Enable access logging"
  type        = bool
  default     = true
}

variable "tags" {
  description = "Additional tags"
  type        = map(string)
  default     = {}
}
```

```hcl
# modules/secure-s3/main.tf
terraform {
  required_version = ">= 1.6.0"
  
  required_providers {
    aws = {
      source  = "hashicorp/aws"
      version = "~> 5.0"
    }
  }
}

locals {
  required_tags = {
    Environment = var.environment
    ManagedBy   = "Terraform"
  }
  
  all_tags = merge(local.required_tags, var.tags)
}

# S3 Bucket
resource "aws_s3_bucket" "main" {
  bucket = var.bucket_name
  
  tags = merge(local.all_tags, {
    Name = var.bucket_name
  })
  
  lifecycle {
    prevent_destroy = false  # Set to true in production
    
    precondition {
      condition     = length(var.bucket_name) >= 3 && length(var.bucket_name) <= 63
      error_message = "Bucket name must be between 3 and 63 characters."
    }
  }
}

# Versioning
resource "aws_s3_bucket_versioning" "main" {
  count = var.enable_versioning ? 1 : 0
  
  bucket = aws_s3_bucket.main.id
  
  versioning_configuration {
    status = "Enabled"
  }
}

# Server-side encryption
resource "aws_s3_bucket_server_side_encryption_configuration" "main" {
  count = var.enable_encryption ? 1 : 0
  
  bucket = aws_s3_bucket.main.id
  
  rule {
    apply_server_side_encryption_by_default {
      sse_algorithm = "AES256"
    }
    bucket_key_enabled = true
  }
}

# Block public access
resource "aws_s3_bucket_public_access_block" "main" {
  bucket = aws_s3_bucket.main.id
  
  block_public_acls       = true
  block_public_policy     = true
  ignore_public_acls      = true
  restrict_public_buckets = true
}

# Logging bucket
resource "aws_s3_bucket" "logs" {
  count = var.enable_logging ? 1 : 0
  
  bucket = "${var.bucket_name}-logs"
  
  tags = merge(local.all_tags, {
    Name    = "${var.bucket_name}-logs"
    Purpose = "AccessLogs"
  })
}

resource "aws_s3_bucket_acl" "logs" {
  count = var.enable_logging ? 1 : 0
  
  bucket = aws_s3_bucket.logs[0].id
  acl    = "log-delivery-write"
  
  depends_on = [aws_s3_bucket_ownership_controls.logs]
}

resource "aws_s3_bucket_ownership_controls" "logs" {
  count = var.enable_logging ? 1 : 0
  
  bucket = aws_s3_bucket.logs[0].id
  
  rule {
    object_ownership = "BucketOwnerPreferred"
  }
}

# Enable logging
resource "aws_s3_bucket_logging" "main" {
  count = var.enable_logging ? 1 : 0
  
  bucket = aws_s3_bucket.main.id
  
  target_bucket = aws_s3_bucket.logs[0].id
  target_prefix = "access-logs/"
}

# Lifecycle policy
resource "aws_s3_bucket_lifecycle_configuration" "main" {
  bucket = aws_s3_bucket.main.id
  
  rule {
    id     = "expire-old-versions"
    status = "Enabled"
    
    noncurrent_version_expiration {
      noncurrent_days = 90
    }
  }
  
  rule {
    id     = "delete-incomplete-uploads"
    status = "Enabled"
    
    abort_incomplete_multipart_upload {
      days_after_initiation = 7
    }
  }
}
```

```hcl
# modules/secure-s3/outputs.tf
output "bucket_id" {
  description = "The name of the bucket"
  value       = aws_s3_bucket.main.id
}

output "bucket_arn" {
  description = "The ARN of the bucket"
  value       = aws_s3_bucket.main.arn
}

output "bucket_domain_name" {
  description = "The bucket domain name"
  value       = aws_s3_bucket.main.bucket_domain_name
}

output "bucket_regional_domain_name" {
  description = "The bucket region-specific domain name"
  value       = aws_s3_bucket.main.bucket_regional_domain_name
}

output "logging_bucket_id" {
  description = "The name of the logging bucket"
  value       = var.enable_logging ? aws_s3_bucket.logs[0].id : null
}

output "versioning_enabled" {
  description = "Whether versioning is enabled"
  value       = var.enable_versioning
}

output "encryption_enabled" {
  description = "Whether encryption is enabled"
  value       = var.enable_encryption
}
```

**7. Example для тестирования:**

```hcl
# examples/basic/main.tf
terraform {
  required_version = ">= 1.6.0"
  
  required_providers {
    aws = {
      source  = "hashicorp/aws"
      version = "~> 5.0"
    }
    random = {
      source  = "hashicorp/random"
      version = "~> 3.5"
    }
  }
}

provider "aws" {
  region = var.aws_region
}

# Random suffix для уникальности
resource "random_id" "suffix" {
  byte_length = 4
}

# Использование модуля
module "secure_s3" {
  source = "../../modules/secure-s3"
  
  bucket_name        = "test-secure-bucket-${random_id.suffix.hex}"
  environment        = var.environment
  enable_versioning  = true
  enable_encryption  = true
  enable_logging     = true
  
  tags = {
    Owner   = "DevOps Team"
    Project = "Testing"
  }
}
```

```hcl
# examples/basic/variables.tf
variable "aws_region" {
  description = "AWS region"
  type        = string
  default     = "us-east-1"
}

variable "environment" {
  description = "Environment name"
  type        = string
  default     = "dev"
}
```

```hcl
# examples/basic/outputs.tf
output "bucket_id" {
  description = "Created bucket ID"
  value       = module.secure_s3.bucket_id
}

output "bucket_arn" {
  description = "Created bucket ARN"
  value       = module.secure_s3.bucket_arn
}

output "logging_bucket_id" {
  description = "Logging bucket ID"
  value       = module.secure_s3.logging_bucket_id
}
```

**8. Terratest код:**

```go
// test/secure_s3_test.go
package test

import (
    "fmt"
    "strings"
    "testing"
    "time"

    "github.com/gruntwork-io/terratest/modules/aws"
    "github.com/gruntwork-io/terratest/modules/random"
    "github.com/gruntwork-io/terratest/modules/terraform"
    "github.com/stretchr/testify/assert"
    "github.com/stretchr/testify/require"
)

func TestSecureS3Bucket(t *testing.T) {
    t.Parallel()

    // Generate random suffix
    uniqueID := random.UniqueId()
    bucketName := fmt.Sprintf("test-secure-bucket-%s", strings.ToLower(uniqueID))
    awsRegion := "us-east-1"

    terraformOptions := terraform.WithDefaultRetryableErrors(t, &terraform.Options{
        TerraformDir: "../examples/basic",
        Vars: map[string]interface{}{
            "aws_region":  awsRegion,
            "environment": "dev",
        },
        EnvVars: map[string]string{
            "AWS_DEFAULT_REGION": awsRegion,
        },
    })

    defer terraform.Destroy(t, terraformOptions)

    // Run terraform init and apply
    terraform.InitAndApply(t, terraformOptions)

    // Validate outputs
    bucketID := terraform.Output(t, terraformOptions, "bucket_id")
    bucketARN := terraform.Output(t, terraformOptions, "bucket_arn")
    loggingBucketID := terraform.Output(t, terraformOptions, "logging_bucket_id")

    // Assertions
    assert.Contains(t, bucketID, "test-secure-bucket")
    assert.Contains(t, bucketARN, "arn:aws:s3:::")
    assert.Contains(t, loggingBucketID, "-logs")

    // Verify bucket exists
    aws.AssertS3BucketExists(t, awsRegion, bucketID)

    // Verify versioning
    actualStatus := aws.GetS3BucketVersioning(t, awsRegion, bucketID)
    assert.Equal(t, "Enabled", actualStatus)

    // Verify encryption
    bucketEncryption := aws.GetS3BucketEncryption(t, awsRegion, bucketID)
    require.NotNil(t, bucketEncryption)
    assert.NotEmpty(t, bucketEncryption.ServerSideEncryptionConfiguration.Rules)

    // Verify public access block
    publicAccessBlock := aws.GetS3BucketPublicAccessBlock(t, awsRegion, bucketID)
    require.NotNil(t, publicAccessBlock)
    assert.True(t, *publicAccessBlock.BlockPublicAcls)
    assert.True(t, *publicAccessBlock.BlockPublicPolicy)
    assert.True(t, *publicAccessBlock.IgnorePublicAcls)
    assert.True(t, *publicAccessBlock.RestrictPublicBuckets)

    // Verify tags
    tags := aws.GetS3BucketTags(t, awsRegion, bucketID)
    assert.Equal(t, "dev", tags["Environment"])
    assert.Equal(t, "Terraform", tags["ManagedBy"])
    assert.Equal(t, "DevOps Team", tags["Owner"])
}

func TestSecureS3BucketWithoutLogging(t *testing.T) {
    t.Parallel()

    awsRegion := "us-east-1"

    terraformOptions := terraform.WithDefaultRetryableErrors(t, &terraform.Options{
        TerraformDir: "../examples/basic",
        Vars: map[string]interface{}{
            "aws_region":      awsRegion,
            "environment":     "dev",
            "enable_logging":  false,
        },
        EnvVars: map[string]string{
            "AWS_DEFAULT_REGION": awsRegion,
        },
    })

    defer terraform.Destroy(t, terraformOptions)

    terraform.InitAndApply(t, terraformOptions)

    loggingBucketID := terraform.Output(t, terraformOptions, "logging_bucket_id")
    assert.Empty(t, loggingBucketID)
}

func TestSecureS3BucketNaming(t *testing.T) {
    t.Parallel()

    testCases := []struct {
        name        string
        bucketName  string
        shouldFail  bool
    }{
        {"Valid lowercase", "my-test-bucket-123", false},
        {"Invalid uppercase", "My-Test-Bucket", true},
        {"Invalid start with hyphen", "-my-bucket", true},
        {"Invalid end with hyphen", "my-bucket-", true},
        {"Valid with numbers", "bucket123", false},
    }

    for _, tc := range testCases {
        tc := tc
        t.Run(tc.name, func(t *testing.T) {
            t.Parallel()

            terraformOptions := terraform.WithDefaultRetryableErrors(t, &terraform.Options{
                TerraformDir: "../examples/basic",
                Vars: map[string]interface{}{
                    "bucket_name": tc.bucketName,
                    "environment": "dev",
                },
            })

            if tc.shouldFail {
                _, err := terraform.InitAndPlanE(t, terraformOptions)
                assert.Error(t, err)
            } else {
                defer terraform.Destroy(t, terraformOptions)
                terraform.InitAndApply(t, terraformOptions)
            }
        })
    }
}
```

```go
// test/go.mod
module github.com/yourorg/terraform-testing/test

go 1.21

require (
    github.com/gruntwork-io/terratest v0.46.7
    github.com/stretchr/testify v1.8.4
)
```

**9. Makefile для тестирования:**

```makefile
# Makefile

.PHONY: help fmt validate lint security test test-module clean

help:
	@echo "Available targets:"
	@echo "  fmt        - Format Terraform code"
	@echo "  validate   - Validate Terraform configuration"
	@echo "  lint       - Run TFLint"
	@echo "  security   - Run security scanners (tfsec, checkov)"
	@echo "  test       - Run all tests"
	@echo "  test-unit  - Run Go unit tests"
	@echo "  clean      - Clean test artifacts"

fmt:
	@echo "==> Formatting Terraform code..."
	terraform fmt -recursive

validate:
	@echo "==> Validating Terraform configuration..."
	cd examples/basic && terraform init -backend=false
	cd examples/basic && terraform validate

lint:
	@echo "==> Running TFLint..."
	tflint --init
	tflint --recursive
	@echo "==> Running terraform-docs..."
	terraform-docs markdown table --output-file README.md modules/secure-s3

security:
	@echo "==> Running tfsec..."
	tfsec . --format default
	@echo ""
	@echo "==> Running Checkov..."
	checkov -d . --quiet --compact
	@echo ""
	@echo "==> Running Terrascan..."
	terrascan scan -t aws

test: fmt validate lint security test-unit
	@echo "==> All tests passed!"

test-unit:
	@echo "==> Running Terratest..."
	cd test && go mod download
	cd test && go test -v -timeout 30m

test-module:
	@echo "==> Testing module in isolation..."
	cd modules/secure-s3 && terraform init -backend=false
	cd modules/secure-s3 && terraform validate

clean:
	@echo "==> Cleaning test artifacts..."
	find . -type d -name ".terraform" -exec rm -rf {} +
	find . -type f -name "terraform.tfstate*" -delete
	find . -type f -name ".terraform.lock.hcl" -delete
	find . -type f -name "tfplan" -delete
	cd test && go clean -testcache

ci: fmt validate lint security
	@echo "==> CI checks completed!"
```

**10. Pre-commit hooks:**

```yaml
# .pre-commit-config.yaml
repos:
  - repo: https://github.com/antonbabenko/pre-commit-terraform
    rev: v1.83.5
    hooks:
      - id: terraform_fmt
        args:
          - --args=-recursive

      - id: terraform_validate
        args:
          - --hook-config=--retry-once-with-cleanup=true
          - --tf-init-args=-backend=false

      - id: terraform_docs
        args:
          - --hook-config=--path-to-file=README.md
          - --hook-config=--add-to-existing-file=true
          - --hook-config=--create-file-if-not-exist=true

      - id: terraform_tflint
        args:
          - --args=--config=__GIT_WORKING_DIR__/.tflint.hcl
          - --args=--module

      - id: terraform_tfsec
        args:
          - --args=--config-file=__GIT_WORKING_DIR__/.tfsec.json

      - id: terraform_checkov
        args:
          - --args=--config-file __GIT_WORKING_DIR__/.checkov.yml
          - --args=--quiet
          - --args=--compact

      - id: terraform_trivy
        args:
          - --args=--severity HIGH,CRITICAL
          - --args=--exit-code 1

  - repo: https://github.com/pre-commit/pre-commit-hooks
    rev: v4.5.0
    hooks:
      - id: trailing-whitespace
      - id: end-of-file-fixer
      - id: check-yaml
      - id: check-added-large-files
        args: ['--maxkb=1000']
      - id: check-merge-conflict
      - id: detect-private-key

  - repo: local
    hooks:
      - id: terraform-custom-validation
        name: Custom Terraform Validation
        entry: scripts/validate-terraform.sh
        language: script
        files: \.tf$
```

**11. Custom validation script:**

```bash
#!/bin/bash
# scripts/validate-terraform.sh

set -e

echo "==> Running custom Terraform validations..."

# Check for hardcoded values
echo "Checking for hardcoded credentials..."
if grep -r -E "(aws_access_key|aws_secret_key|password|secret)" --include="*.tf" .; then
    echo "ERROR: Found hardcoded credentials!"
    exit 1
fi

# Check for required tags
echo "Checking for required tags..."
required_tags=("Environment" "Owner" "Project" "ManagedBy")
for tag in "${required_tags[@]}"; do
    if ! grep -r "tags.*$tag" --include="*.tf" . > /dev/null; then
        echo "WARNING: Tag '$tag' not found in some resources"
    fi
done

# Check naming conventions
echo "Checking resource naming conventions..."
if grep -r "resource.*\"[A-Z]" --include="*.tf" .; then
    echo "ERROR: Resource names should use snake_case!"
    exit 1
fi

# Check for prevent_destroy in production resources
echo "Checking prevent_destroy for critical resources..."
critical_resources=("aws_db_instance" "aws_s3_bucket")
for resource in "${critical_resources[@]}"; do
    if grep -A 10 "resource \"$resource\"" --include="*.tf" -r . | grep -q "prevent_destroy = true"; then
        echo "✓ Found prevent_destroy for $resource"
    else
        echo "WARNING: prevent_destroy not found for $resource"
    fi
done

echo "==> Custom validations completed!"
```

```bash
chmod +x scripts/validate-terraform.sh
```

**12. GitHub Actions Workflow:**

```yaml
# .github/workflows/terraform-test.yml
name: Terraform Testing

on:
  push:
    branches: [ main, develop ]
  pull_request:
    branches: [ main ]

env:
  AWS_REGION: us-east-1
  TF_VERSION: 1.6.0

jobs: 
	static-analysis: 
	 name: Static Analysis 
	 runs-on: ubuntu-latest

steps:
  - name: Checkout code
    uses: actions/checkout@v4

  - name: Setup Terraform
    uses: hashicorp/setup-terraform@v3
    with:
      terraform_version: ${{ env.TF_VERSION }}

  - name: Terraform Format Check
    run: terraform fmt -check -recursive

  - name: Setup TFLint
    uses: terraform-linters/setup-tflint@v4
    with:
      tflint_version: latest

  - name: Initialize TFLint
    run: tflint --init

  - name: Run TFLint
    run: tflint --recursive --format compact

  - name: Run tfsec
    uses: aquasecurity/tfsec-action@v1.0.3
    with:
      working_directory: .
      format: sarif
      soft_fail: false

  - name: Upload tfsec SARIF
    uses: github/codeql-action/upload-sarif@v2
    if: always()
    with:
      sarif_file: tfsec.sarif
```

security-scan: name: Security Scanning runs-on: ubuntu-latest

```yaml
steps:
  - name: Checkout code
    uses: actions/checkout@v4

  - name: Run Checkov
    uses: bridgecrewio/checkov-action@master
    with:
      directory: .
      framework: terraform
      output_format: sarif
      output_file_path: checkov.sarif
      soft_fail: false
      skip_check: CKV_AWS_18,CKV_AWS_356

  - name: Upload Checkov SARIF
    uses: github/codeql-action/upload-sarif@v2
    if: always()
    with:
      sarif_file: checkov.sarif

  - name: Run Terrascan
    uses: tenable/terrascan-action@main
    with:
      iac_type: 'terraform'
      iac_version: 'v14'
      policy_type: 'aws'
      only_warn: false
      sarif_upload: true

 unit-tests: 
	name: Unit Tests 
	runs-on: ubuntu-latest 
	needs: [static-analysis, security-scan]


steps:
  - name: Checkout code
    uses: actions/checkout@v4

  - name: Setup Terraform
    uses: hashicorp/setup-terraform@v3
    with:
      terraform_version: ${{ env.TF_VERSION }}
      terraform_wrapper: false

  - name: Setup Go
    uses: actions/setup-go@v4
    with:
      go-version: '1.21'

  - name: Configure AWS Credentials
    uses: aws-actions/configure-aws-credentials@v4
    with:
      aws-access-key-id: ${{ secrets.AWS_ACCESS_KEY_ID }}
      aws-secret-access-key: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
      aws-region: ${{ env.AWS_REGION }}

  - name: Download Go dependencies
    working-directory: test
    run: go mod download

  - name: Run Terratest
    working-directory: test
    run: |
      go test -v -timeout 30m -parallel 2 | tee test-output.log

  - name: Upload test results
    uses: actions/upload-artifact@v3
    if: always()
    with:
      name: test-results
      path: test/test-output.log

 validate-examples:
	name: Validate Examples
	runs-on: ubuntu-latest
	strategy:
	matrix:
	example: [basic]


steps:
  - name: Checkout code
    uses: actions/checkout@v4

  - name: Setup Terraform
    uses: hashicorp/setup-terraform@v3
    with:
      terraform_version: ${{ env.TF_VERSION }}

  - name: Terraform Init
    working-directory: examples/${{ matrix.example }}
    run: terraform init -backend=false

  - name: Terraform Validate
    working-directory: examples/${{ matrix.example }}
    run: terraform validate

  - name: Terraform Plan
    working-directory: examples/${{ matrix.example }}
    run: terraform plan -out=tfplan
    env:
      AWS_ACCESS_KEY_ID: ${{ secrets.AWS_ACCESS_KEY_ID }}
      AWS_SECRET_ACCESS_KEY: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
```


**13. Установка и запуск pre-commit:**

```bash
# Установить pre-commit
pre-commit install

# Запустить на всех файлах
pre-commit run --all-files

# Обновить hooks
pre-commit autoupdate
```

**14. Запуск всех тестов:**

```bash
# Запустить все проверки
make test

# Только форматирование
make fmt

# Только валидация
make validate

# Только линтинг
make lint

# Только security сканы
make security

# Только unit тесты
make test-unit

# CI проверки (без unit тестов)
make ci

# Очистка
make clean
```

**15. Custom Checkov check:**

```python
# custom-checks/check_s3_encryption.py
from checkov.common.models.enums import CheckCategories, CheckResult
from checkov.terraform.checks.resource.base_resource_check import BaseResourceCheck


class S3BucketEncryption(BaseResourceCheck):
    def __init__(self):
        name = "Ensure S3 bucket has encryption enabled"
        id = "CKV_CUSTOM_S3_1"
        supported_resources = ['aws_s3_bucket']
        categories = [CheckCategories.ENCRYPTION]
        super().__init__(name=name, id=id, categories=categories, supported_resources=supported_resources)

    def scan_resource_conf(self, conf):
        """
        Looks for encryption configuration in S3 bucket
        """
        # Check for server_side_encryption_configuration
        if 'server_side_encryption_configuration' in conf:
            encryption_conf = conf['server_side_encryption_configuration'][0]
            if 'rule' in encryption_conf:
                rules = encryption_conf['rule']
                if isinstance(rules, list) and len(rules) > 0:
                    rule = rules[0]
                    if 'apply_server_side_encryption_by_default' in rule:
                        return CheckResult.PASSED
        
        return CheckResult.FAILED


check = S3BucketEncryption()
```

### 🚀 Бонус (продвинутое)

**1. Custom TFLint Rule:**

```hcl
# .tflint/custom-rules.hcl
rule "custom_naming_convention" {
  enabled = true
  
  resource_types = ["aws_instance", "aws_security_group"]
  
  pattern = "^[a-z][a-z0-9-]*$"
  
  message = "Resource names must use lowercase letters, numbers, and hyphens only"
}

rule "custom_tagging_standard" {
  enabled = true
  
  required_tags = {
    "Environment" = ".*"
    "Owner"       = "^[A-Za-z\\s]+$"
    "Project"     = ".*"
    "CostCenter"  = "^CC-[0-9]{4}$"
  }
  
  message = "All resources must have required tags with correct format"
}
```

**2. OPA (Open Policy Agent) Policies:**

```rego
# policies/terraform.rego
package terraform.analysis

import future.keywords.contains
import future.keywords.if

# Deny S3 buckets without encryption
deny[msg] {
    resource := input.resource_changes[_]
    resource.type == "aws_s3_bucket"
    not has_encryption(resource)
    
    msg := sprintf(
        "S3 bucket '%s' must have encryption enabled",
        [resource.address]
    )
}

has_encryption(resource) {
    resource.change.after.server_side_encryption_configuration[_]
}

# Deny instances without monitoring in production
deny[msg] {
    resource := input.resource_changes[_]
    resource.type == "aws_instance"
    is_production(resource)
    not has_monitoring(resource)
    
    msg := sprintf(
        "Production instance '%s' must have detailed monitoring enabled",
        [resource.address]
    )
}

is_production(resource) {
    resource.change.after.tags.Environment == "prod"
}

has_monitoring(resource) {
    resource.change.after.monitoring == true
}

# Warn about expensive instance types in dev
warn[msg] {
    resource := input.resource_changes[_]
    resource.type == "aws_instance"
    is_development(resource)
    is_expensive_type(resource.change.after.instance_type)
    
    msg := sprintf(
        "Instance '%s' uses expensive type '%s' in dev environment",
        [resource.address, resource.change.after.instance_type]
    )
}

is_development(resource) {
    resource.change.after.tags.Environment == "dev"
}

is_expensive_type(instance_type) {
    expensive_types := ["t3.large", "t3.xlarge", "m5.large", "m5.xlarge"]
    instance_type == expensive_types[_]
}

# Require specific tags
deny[msg] {
    resource := input.resource_changes[_]
    not is_ignored_resource_type(resource.type)
    missing_tags := get_missing_required_tags(resource)
    count(missing_tags) > 0
    
    msg := sprintf(
        "Resource '%s' is missing required tags: %v",
        [resource.address, missing_tags]
    )
}

required_tags := ["Environment", "Owner", "Project", "ManagedBy"]

get_missing_required_tags(resource) = missing {
    existing_tags := {tag | resource.change.after.tags[tag]}
    required := {tag | required_tags[_] = tag}
    missing := required - existing_tags
}

is_ignored_resource_type(type) {
    ignored_types := ["random_id", "random_string", "null_resource"]
    type == ignored_types[_]
}
```

```bash
# Использование OPA
terraform plan -out=tfplan.binary
terraform show -json tfplan.binary > tfplan.json

# Run OPA
opa eval -i tfplan.json -d policies/terraform.rego "data.terraform.analysis.deny"
```

**3. Pytest-Terraform Integration:**

```python
# test/test_s3_module.py
import pytest
import tftest
import json


@pytest.fixture
def tf():
    """Setup terraform test fixture"""
    tf = tftest.TerraformTest('examples/basic')
    tf.setup(extra_files=['terraform.tfvars'])
    return tf


def test_s3_outputs(tf):
    """Test S3 module outputs"""
    tf.apply()
    
    # Get outputs
    outputs = tf.output()
    
    # Assertions
    assert 'bucket_id' in outputs
    assert 'bucket_arn' in outputs
    assert outputs['bucket_id'].startswith('test-secure-bucket')
    assert outputs['bucket_arn'].startswith('arn:aws:s3:::')
    
    # Cleanup
    tf.destroy()


def test_s3_versioning(tf):
    """Test S3 versioning configuration"""
    tf.apply()
    
    # Get state
    state = json.loads(tf.show(json_format=True))
    
    # Find versioning resource
    versioning_resources = [
        r for r in state.get('values', {}).get('root_module', {}).get('resources', [])
        if r['type'] == 'aws_s3_bucket_versioning'
    ]
    
    assert len(versioning_resources) == 1
    assert versioning_resources[0]['values']['versioning_configuration'][0]['status'] == 'Enabled'
    
    tf.destroy()


def test_s3_encryption(tf):
    """Test S3 encryption configuration"""
    tf.apply()
    
    state = json.loads(tf.show(json_format=True))
    
    # Find encryption resource
    encryption_resources = [
        r for r in state.get('values', {}).get('root_module', {}).get('resources', [])
        if r['type'] == 'aws_s3_bucket_server_side_encryption_configuration'
    ]
    
    assert len(encryption_resources) == 1
    rule = encryption_resources[0]['values']['rule'][0]
    assert rule['apply_server_side_encryption_by_default'][0]['sse_algorithm'] == 'AES256'
    
    tf.destroy()


def test_s3_public_access_block(tf):
    """Test S3 public access block"""
    tf.apply()
    
    state = json.loads(tf.show(json_format=True))
    
    # Find public access block resource
    pab_resources = [
        r for r in state.get('values', {}).get('root_module', {}).get('resources', [])
        if r['type'] == 'aws_s3_bucket_public_access_block'
    ]
    
    assert len(pab_resources) == 1
    pab = pab_resources[0]['values']
    assert pab['block_public_acls'] is True
    assert pab['block_public_policy'] is True
    assert pab['ignore_public_acls'] is True
    assert pab['restrict_public_buckets'] is True
    
    tf.destroy()


@pytest.mark.parametrize("enable_logging", [True, False])
def test_s3_conditional_logging(tf, enable_logging):
    """Test S3 logging configuration"""
    tf.apply(tf_vars={'enable_logging': enable_logging})
    
    outputs = tf.output()
    
    if enable_logging:
        assert outputs.get('logging_bucket_id') is not None
        assert outputs['logging_bucket_id'].endswith('-logs')
    else:
        assert outputs.get('logging_bucket_id') is None
    
    tf.destroy()
```

```python
# test/conftest.py
import pytest
import os


def pytest_configure(config):
    """Configure pytest"""
    # Set AWS credentials from environment
    os.environ.setdefault('AWS_DEFAULT_REGION', 'us-east-1')


@pytest.fixture(scope='session')
def aws_region():
    """AWS region fixture"""
    return 'us-east-1'


@pytest.fixture(scope='session')
def environment():
    """Environment fixture"""
    return 'test'
```

**4. Cost Estimation Integration:**

```yaml
# .github/workflows/cost-estimate.yml
name: Cost Estimation

on:
  pull_request:
    branches: [ main ]

jobs:
  infracost:
    name: Infracost
    runs-on: ubuntu-latest
    
    steps:
      - name: Checkout code
        uses: actions/checkout@v4

      - name: Setup Terraform
        uses: hashicorp/setup-terraform@v3
        with:
          terraform_version: 1.6.0

      - name: Setup Infracost
        uses: infracost/actions/setup@v2
        with:
          api-key: ${{ secrets.INFRACOST_API_KEY }}

      - name: Generate Infracost JSON
        run: |
          cd examples/basic
          infracost breakdown --path . \
            --format json \
            --out-file /tmp/infracost.json

      - name: Post Infracost comment
        uses: infracost/actions/comment@v1
        with:
          path: /tmp/infracost.json
          behavior: update
```

**5. Performance Testing:**

```bash
#!/bin/bash
# scripts/performance-test.sh

set -e

echo "==> Running Terraform Performance Tests..."

TEST_DIR="examples/basic"
ITERATIONS=5

# Cleanup function
cleanup() {
    cd $TEST_DIR
    terraform destroy -auto-approve || true
}

trap cleanup EXIT

cd $TEST_DIR

# Init timing
echo "Testing terraform init..."
for i in $(seq 1 $ITERATIONS); do
    rm -rf .terraform .terraform.lock.hcl
    /usr/bin/time -f "Init iteration $i: %E elapsed" terraform init
done

# Plan timing
echo ""
echo "Testing terraform plan..."
terraform init -upgrade
for i in $(seq 1 $ITERATIONS); do
    /usr/bin/time -f "Plan iteration $i: %E elapsed" terraform plan -out=tfplan
    rm tfplan
done

# Apply timing
echo ""
echo "Testing terraform apply..."
for i in $(seq 1 $ITERATIONS); do
    terraform plan -out=tfplan
    /usr/bin/time -f "Apply iteration $i: %E elapsed" terraform apply tfplan
    terraform destroy -auto-approve
done

echo "==> Performance tests completed!"
```

---

## Заключение Модуля 11

### 📚 Ключевые takeaways

**1. Уровни тестирования:**

```
Static Analysis (быстро, дешево)
├── terraform fmt
├── terraform validate
├── TFLint
└── terraform-docs

Security Scanning (критично)
├── tfsec
├── Checkov
├── Terrascan
└── OPA policies

Integration Testing (дорого, важно)
├── Terratest
├── pytest-terraform
└── kitchen-terraform

Automation (обязательно)
├── Pre-commit hooks
├── CI/CD pipelines
└── Automated PR checks
```

**2. Best Practices:**

```hcl
# ✅ DO: Automate все проверки
pre-commit install

# ✅ DO: Используй несколько инструментов
make lint security test

# ✅ DO: Пиши понятные error messages
validation {
  condition     = var.count >= 1
  error_message = "Count must be at least 1, got ${var.count}"
}

# ✅ DO: Тестируй edge cases
@pytest.mark.parametrize("enable_feature", [True, False])

# ❌ DON'T: Пропускай security сканы
# Всегда запускай tfsec и Checkov

# ❌ DON'T: Игнорируй warnings
# Обрабатывай или явно suppress с объяснением
```

**3. CI/CD Integration:**

```yaml
# Минимальный pipeline
steps:
  - terraform fmt -check
  - terraform validate
  - tflint
  - tfsec
  - checkov
  - terraform plan

# Production pipeline
steps:
  - Static analysis
  - Security scanning
  - Unit tests (Terratest)
  - Cost estimation (Infracost)
  - Manual approval
  - Apply
```

**4. Когда использовать каждый инструмент:**

|Tool|Purpose|When to Use|
|---|---|---|
|TFLint|Best practices|Always, in CI/CD|
|tfsec|Security issues|Always, in CI/CD|
|Checkov|Compliance|Always, especially for regulated industries|
|Terratest|Integration tests|Before major releases|
|OPA|Custom policies|For specific organizational rules|
|Pre-commit|Prevention|For all developers|

Ты освоил:

- ✅ Static analysis с TFLint
- ✅ Security scanning с Checkov и tfsec
- ✅ Integration testing с Terratest
- ✅ Automation с pre-commit hooks
- ✅ CI/CD integration
- ✅ Custom validation patterns

**Практические рекомендации:**

1. **Начни с простого**: fmt → validate → lint
2. **Добавь security**: tfsec → Checkov
3. **Автоматизируй**: pre-commit hooks
4. **Интегрируй в CI/CD**: GitHub Actions / GitLab CI
5. **Пиши тесты**: Terratest для критичных модулей

---

Теперь ты знаешь:

- ✅ Базовую архитектуру и best practices
- ✅ State management и backends
- ✅ Modules и code reusability
- ✅ Advanced features (dynamic blocks, for expressions)
- ✅ Lifecycle management
- ✅ Testing и validation

**Следующие шаги:**

1. Практикуй на реальных проектах
2. Изучи Terraform Cloud/Enterprise
3. Освой CI/CD integration
4. Dive deeper в Policy as Code
5. Contribute to community modules

# Модуль 12: CI/CD Pipeline (30 минут)

## 🎯 Напоминалка

**CI/CD для Terraform - основные компоненты:**

```
CI/CD Pipeline для Terraform
├── Version Control (Git)
├── Pull Request / Merge Request Workflow
├── Automated Testing
│   ├── terraform fmt -check
│   ├── terraform validate
│   ├── tflint / checkov
│   └── terratest / kitchen-terraform
├── Plan on PR (Preview changes)
├── Apply on Merge (Deploy changes)
├── State Management (Remote backend)
└── Security & Compliance
    ├── Secret scanning
    ├── Policy as Code (OPA/Sentinel)
    └── Cost estimation
```

**Основные инструменты:**

```
CI/CD Tools
├── GitHub Actions - встроенный в GitHub
├── GitLab CI/CD - встроенный в GitLab
├── Jenkins - самохостинг
├── CircleCI - SaaS
└── Azure DevOps - Microsoft

Terraform-Specific Tools
├── Atlantis - Pull Request automation
├── Terraform Cloud - HashiCorp SaaS
├── Spacelift - Advanced automation
└── env0 - Self-service infrastructure
```

**Best Practices для Terraform CI/CD:**

1. **Всегда** делай `terraform plan` на Pull Request
2. **Никогда** не коммить credentials в Git
3. **Всегда** используй remote state
4. **Защити** production branch (требуй approvals)
5. **Используй** workspaces или отдельные directories для environments
6. **Автоматизируй** все проверки (lint, validate, security)
7. **Логируй** все изменения инфраструктуры
8. **Используй** CODEOWNERS для review процесса

---

## Часть 1: GitHub Actions (Полный Workflow)

### 💻 Задание: Настройка GitHub Actions для Terraform

**1. Структура проекта:**

```bash
terraform-github-actions/
├── .github/
│   └── workflows/
│       ├── terraform-plan.yml      # Plan on PR
│       ├── terraform-apply.yml     # Apply on merge
│       ├── terraform-destroy.yml   # Manual destroy
│       └── security-scan.yml       # Security checks
├── environments/
│   ├── dev/
│   │   ├── main.tf
│   │   ├── variables.tf
│   │   ├── terraform.tfvars
│   │   └── backend.tf
│   ├── staging/
│   │   └── ...
│   └── prod/
│       └── ...
├── modules/
│   └── vpc/
│       ├── main.tf
│       ├── variables.tf
│       └── outputs.tf
├── tests/
│   └── vpc_test.go
├── .tflint.hcl
├── .checkov.yml
├── CODEOWNERS
└── README.md
```

**2. .github/workflows/terraform-plan.yml:**

```yaml
name: 'Terraform Plan'

on:
  pull_request:
    branches:
      - main
      - develop
    paths:
      - 'environments/**'
      - 'modules/**'
      - '.github/workflows/terraform-plan.yml'

# Permissions для комментирования PR
permissions:
  contents: read
  pull-requests: write
  id-token: write  # Для OIDC authentication

env:
  TF_VERSION: '1.6.0'
  TF_LOG: INFO
  AWS_REGION: 'us-east-1'

jobs:
  # Job 1: Validate и Lint
  validate:
    name: 'Validate & Lint'
    runs-on: ubuntu-latest
    
    steps:
      - name: Checkout code
        uses: actions/checkout@v4
      
      - name: Setup Terraform
        uses: hashicorp/setup-terraform@v3
        with:
          terraform_version: ${{ env.TF_VERSION }}
      
      - name: Terraform Format Check
        id: fmt
        run: terraform fmt -check -recursive
        continue-on-error: true
      
      - name: Terraform Init (for validation)
        run: |
          cd environments/dev
          terraform init -backend=false
      
      - name: Terraform Validate
        id: validate
        run: |
          cd environments/dev
          terraform validate -no-color
      
      - name: Setup TFLint
        uses: terraform-linters/setup-tflint@v4
        with:
          tflint_version: latest
      
      - name: Init TFLint
        run: tflint --init
      
      - name: Run TFLint
        id: tflint
        run: tflint --recursive --format compact
        continue-on-error: true
      
      - name: Comment PR with results
        if: always()
        uses: actions/github-script@v7
        with:
          script: |
            const output = `#### Terraform Format and Style 🖌 \`${{ steps.fmt.outcome }}\`
            #### Terraform Validation 🤖 \`${{ steps.validate.outcome }}\`
            #### TFLint 📝 \`${{ steps.tflint.outcome }}\`
            
            *Pusher: @${{ github.actor }}, Action: \`${{ github.event_name }}\`*`;
            
            github.rest.issues.createComment({
              issue_number: context.issue.number,
              owner: context.repo.owner,
              repo: context.repo.repo,
              body: output
            });

  # Job 2: Security Scan
  security:
    name: 'Security Scan'
    runs-on: ubuntu-latest
    needs: validate
    
    steps:
      - name: Checkout code
        uses: actions/checkout@v4
      
      - name: Run Checkov
        id: checkov
        uses: bridgecrewio/checkov-action@master
        with:
          directory: .
          framework: terraform
          quiet: false
          soft_fail: true
          output_format: cli
          download_external_modules: true
      
      - name: Run tfsec
        id: tfsec
        uses: aquasecurity/tfsec-action@v1.0.3
        with:
          soft_fail: true
          format: default
          additional_args: --concise-output
      
      - name: Comment Security Results
        if: always()
        uses: actions/github-script@v7
        with:
          script: |
            const output = `#### Security Scan Results 🔒
            
            **Checkov**: \`${{ steps.checkov.outcome }}\`
            **tfsec**: \`${{ steps.tfsec.outcome }}\`
            
            [View detailed results in the Actions tab](${{ github.server_url }}/${{ github.repository }}/actions/runs/${{ github.run_id }})`;
            
            github.rest.issues.createComment({
              issue_number: context.issue.number,
              owner: context.repo.owner,
              repo: context.repo.repo,
              body: output
            });

  # Job 3: Plan for each environment
  plan:
    name: 'Terraform Plan'
    runs-on: ubuntu-latest
    needs: [validate, security]
    
    strategy:
      fail-fast: false
      matrix:
        environment: [dev, staging]
    
    steps:
      - name: Checkout code
        uses: actions/checkout@v4
      
      - name: Configure AWS Credentials (OIDC)
        uses: aws-actions/configure-aws-credentials@v4
        with:
          role-to-assume: ${{ secrets[format('AWS_ROLE_{0}', matrix.environment)] }}
          aws-region: ${{ env.AWS_REGION }}
      
      - name: Setup Terraform
        uses: hashicorp/setup-terraform@v3
        with:
          terraform_version: ${{ env.TF_VERSION }}
          terraform_wrapper: false  # Для чистого output
      
      - name: Terraform Init
        id: init
        run: |
          cd environments/${{ matrix.environment }}
          terraform init
      
      - name: Terraform Plan
        id: plan
        run: |
          cd environments/${{ matrix.environment }}
          terraform plan -no-color -out=tfplan
        continue-on-error: true
      
      - name: Generate Plan Summary
        id: summary
        run: |
          cd environments/${{ matrix.environment }}
          terraform show -no-color tfplan > plan_output.txt
          
          # Extract summary
          echo "## Plan Summary" >> $GITHUB_STEP_SUMMARY
          echo "" >> $GITHUB_STEP_SUMMARY
          echo "**Environment:** ${{ matrix.environment }}" >> $GITHUB_STEP_SUMMARY
          echo "" >> $GITHUB_STEP_SUMMARY
          grep -E "Plan:|No changes" plan_output.txt >> $GITHUB_STEP_SUMMARY || echo "Changes detected" >> $GITHUB_STEP_SUMMARY
      
      - name: Upload Plan Artifact
        uses: actions/upload-artifact@v4
        with:
          name: tfplan-${{ matrix.environment }}
          path: environments/${{ matrix.environment }}/tfplan
          retention-days: 5
      
      - name: Comment Plan on PR
        uses: actions/github-script@v7
        with:
          script: |
            const fs = require('fs');
            const planOutput = fs.readFileSync('environments/${{ matrix.environment }}/plan_output.txt', 'utf8');
            
            // Truncate if too long (GitHub has 65536 char limit)
            const maxLength = 65000;
            const truncatedOutput = planOutput.length > maxLength 
              ? planOutput.substring(0, maxLength) + '\n\n... (truncated)'
              : planOutput;
            
            const output = `### Terraform Plan - ${{ matrix.environment }} 📊
            
            <details><summary>Show Plan</summary>
            
            \`\`\`terraform
            ${truncatedOutput}
            \`\`\`
            
            </details>
            
            *Pusher: @${{ github.actor }}, Workflow: \`${{ github.workflow }}\`*`;
            
            github.rest.issues.createComment({
              issue_number: context.issue.number,
              owner: context.repo.owner,
              repo: context.repo.repo,
              body: output
            });
      
      - name: Fail if Plan Failed
        if: steps.plan.outcome == 'failure'
        run: exit 1

  # Job 4: Cost Estimation (optional)
  cost:
    name: 'Cost Estimation'
    runs-on: ubuntu-latest
    needs: plan
    if: github.event_name == 'pull_request'
    
    steps:
      - name: Checkout code
        uses: actions/checkout@v4
      
      - name: Setup Infracost
        uses: infracost/actions/setup@v2
        with:
          api-key: ${{ secrets.INFRACOST_API_KEY }}
      
      - name: Download Plan Artifacts
        uses: actions/download-artifact@v4
        with:
          pattern: tfplan-*
          path: plans/
      
      - name: Generate Cost Estimate
        run: |
          for env in dev staging; do
            infracost breakdown \
              --path plans/tfplan-${env}/tfplan \
              --format json \
              --out-file /tmp/infracost-${env}.json
          done
          
          infracost output \
            --path "/tmp/infracost-*.json" \
            --format github-comment \
            --out-file /tmp/infracost_comment.md
      
      - name: Post Cost Comment
        uses: actions/github-script@v7
        with:
          script: |
            const fs = require('fs');
            const comment = fs.readFileSync('/tmp/infracost_comment.md', 'utf8');
            
            github.rest.issues.createComment({
              issue_number: context.issue.number,
              owner: context.repo.owner,
              repo: context.repo.repo,
              body: comment
            });
```

**3. .github/workflows/terraform-apply.yml:**

```yaml
name: 'Terraform Apply'

on:
  push:
    branches:
      - main
    paths:
      - 'environments/**'
      - 'modules/**'
  
  workflow_dispatch:  # Manual trigger
    inputs:
      environment:
        description: 'Environment to deploy'
        required: true
        type: choice
        options:
          - dev
          - staging
          - prod

permissions:
  contents: read
  id-token: write

env:
  TF_VERSION: '1.6.0'
  AWS_REGION: 'us-east-1'

jobs:
  # Определяем environments для деплоя
  determine-environments:
    name: 'Determine Environments'
    runs-on: ubuntu-latest
    outputs:
      environments: ${{ steps.set-envs.outputs.environments }}
    
    steps:
      - name: Set Environments
        id: set-envs
        run: |
          if [ "${{ github.event_name }}" == "workflow_dispatch" ]; then
            # Manual trigger - только выбранный environment
            echo "environments=[\"${{ inputs.environment }}\"]" >> $GITHUB_OUTPUT
          else
            # Auto trigger на main - только dev и staging
            # Production требует manual approval
            echo "environments=[\"dev\",\"staging\"]" >> $GITHUB_OUTPUT
          fi

  # Apply для каждого environment
  apply:
    name: 'Apply - ${{ matrix.environment }}'
    runs-on: ubuntu-latest
    needs: determine-environments
    
    strategy:
      fail-fast: false
      max-parallel: 1  # Deploy по одному environment
      matrix:
        environment: ${{ fromJSON(needs.determine-environments.outputs.environments) }}
    
    environment:
      name: ${{ matrix.environment }}
      url: https://${{ steps.output.outputs.url }}
    
    steps:
      - name: Checkout code
        uses: actions/checkout@v4
      
      - name: Configure AWS Credentials
        uses: aws-actions/configure-aws-credentials@v4
        with:
          role-to-assume: ${{ secrets[format('AWS_ROLE_{0}', matrix.environment)] }}
          aws-region: ${{ env.AWS_REGION }}
      
      - name: Setup Terraform
        uses: hashicorp/setup-terraform@v3
        with:
          terraform_version: ${{ env.TF_VERSION }}
      
      - name: Terraform Init
        run: |
          cd environments/${{ matrix.environment }}
          terraform init
      
      - name: Terraform Plan
        id: plan
        run: |
          cd environments/${{ matrix.environment }}
          terraform plan -out=tfplan
      
      - name: Terraform Apply
        id: apply
        run: |
          cd environments/${{ matrix.environment }}
          terraform apply -auto-approve tfplan
      
      - name: Get Outputs
        id: output
        run: |
          cd environments/${{ matrix.environment }}
          terraform output -json > outputs.json
          
          # Extract URL if exists
          url=$(jq -r '.alb_dns.value // empty' outputs.json)
          echo "url=$url" >> $GITHUB_OUTPUT
      
      - name: Upload State Backup
        uses: actions/upload-artifact@v4
        with:
          name: tfstate-${{ matrix.environment }}-${{ github.sha }}
          path: environments/${{ matrix.environment }}/.terraform/
          retention-days: 30
      
      - name: Notify Success
        if: success()
        uses: actions/github-script@v7
        with:
          script: |
            const output = `### ✅ Deployment Successful - ${{ matrix.environment }}
            
            **Commit:** ${{ github.sha }}
            **Actor:** @${{ github.actor }}
            **Timestamp:** ${new Date().toISOString()}
            
            ${steps.output.outputs.url ? `**URL:** https://${steps.output.outputs.url}` : ''}
            
            [View deployment details](${{ github.server_url }}/${{ github.repository }}/actions/runs/${{ github.run_id }})`;
            
            github.rest.repos.createCommitComment({
              owner: context.repo.owner,
              repo: context.repo.repo,
              commit_sha: context.sha,
              body: output
            });
      
      - name: Notify Failure
        if: failure()
        uses: actions/github-script@v7
        with:
          script: |
            const output = `### ❌ Deployment Failed - ${{ matrix.environment }}
            
            **Commit:** ${{ github.sha }}
            **Actor:** @${{ github.actor }}
            **Error:** Check the [workflow run](${{ github.server_url }}/${{ github.repository }}/actions/runs/${{ github.run_id }}) for details.`;
            
            github.rest.repos.createCommitComment({
              owner: context.repo.owner,
              repo: context.repo.repo,
              commit_sha: context.sha,
              body: output
            });

  # Post-deployment tests
  verify:
    name: 'Verify Deployment'
    runs-on: ubuntu-latest
    needs: apply
    
    strategy:
      matrix:
        environment: ${{ fromJSON(needs.determine-environments.outputs.environments) }}
    
    steps:
      - name: Checkout code
        uses: actions/checkout@v4
      
      - name: Configure AWS Credentials
        uses: aws-actions/configure-aws-credentials@v4
        with:
          role-to-assume: ${{ secrets[format('AWS_ROLE_{0}', matrix.environment)] }}
          aws-region: ${{ env.AWS_REGION }}
      
      - name: Health Check
        run: |
          cd environments/${{ matrix.environment }}
          
          # Get ALB URL from Terraform outputs
          terraform init
          url=$(terraform output -raw alb_dns 2>/dev/null || echo "")
          
          if [ -n "$url" ]; then
            echo "Testing endpoint: http://$url"
            
            # Retry logic
            for i in {1..10}; do
              if curl -sf "http://$url" > /dev/null; then
                echo "✅ Health check passed"
                exit 0
              fi
              echo "Attempt $i failed, retrying in 30s..."
              sleep 30
            done
            
            echo "❌ Health check failed"
            exit 1
          else
            echo "⚠️ No URL to test, skipping health check"
          fi
```

**4. .github/workflows/terraform-destroy.yml:**

```yaml
name: 'Terraform Destroy'

on:
  workflow_dispatch:
    inputs:
      environment:
        description: 'Environment to destroy'
        required: true
        type: choice
        options:
          - dev
          - staging
      confirm:
        description: 'Type "destroy" to confirm'
        required: true

permissions:
  contents: read
  id-token: write

env:
  TF_VERSION: '1.6.0'
  AWS_REGION: 'us-east-1'

jobs:
  destroy:
    name: 'Destroy Infrastructure'
    runs-on: ubuntu-latest
    
    # Дополнительная защита
    environment:
      name: ${{ inputs.environment }}-destroy
    
    steps:
      - name: Validate Confirmation
        run: |
          if [ "${{ inputs.confirm }}" != "destroy" ]; then
            echo "❌ Confirmation failed. You must type 'destroy' to proceed."
            exit 1
          fi
      
      - name: Block Production
        if: inputs.environment == 'prod'
        run: |
          echo "❌ Production destruction is not allowed via workflow"
          exit 1
      
      - name: Checkout code
        uses: actions/checkout@v4
      
      - name: Configure AWS Credentials
        uses: aws-actions/configure-aws-credentials@v4
        with:
          role-to-assume: ${{ secrets[format('AWS_ROLE_{0}', inputs.environment)] }}
          aws-region: ${{ env.AWS_REGION }}
      
      - name: Setup Terraform
        uses: hashicorp/setup-terraform@v3
        with:
          terraform_version: ${{ env.TF_VERSION }}
      
      - name: Terraform Init
        run: |
          cd environments/${{ inputs.environment }}
          terraform init
      
      - name: Terraform Destroy Plan
        id: plan
        run: |
          cd environments/${{ inputs.environment }}
          terraform plan -destroy -out=destroy.tfplan
      
      - name: Show Destroy Plan
        run: |
          cd environments/${{ inputs.environment }}
          terraform show destroy.tfplan
      
      - name: Wait for Final Confirmation
        uses: trstringer/manual-approval@v1
        with:
          secret: ${{ github.TOKEN }}
          approvers: ${{ github.actor }}
          minimum-approvals: 1
          issue-title: "Approve destroy for ${{ inputs.environment }}"
          issue-body: |
            Please review the destroy plan and approve if you want to proceed.
            
            **Environment:** ${{ inputs.environment }}
            **Triggered by:** @${{ github.actor }}
      
      - name: Terraform Destroy
        run: |
          cd environments/${{ inputs.environment }}
          terraform apply -auto-approve destroy.tfplan
      
      - name: Notify Destruction
        if: always()
        uses: actions/github-script@v7
        with:
          script: |
            const output = `### 🗑️ Infrastructure Destroyed
            
            **Environment:** ${{ inputs.environment }}
            **Actor:** @${{ github.actor }}
            **Status:** ${{ job.status }}
            **Timestamp:** ${new Date().toISOString()}`;
            
            github.rest.repos.createCommitComment({
              owner: context.repo.owner,
              repo: context.repo.repo,
              commit_sha: context.sha,
              body: output
            });
```

**5. .github/workflows/security-scan.yml:**

```yaml
name: 'Security Scan'

on:
  schedule:
    # Run daily at 2 AM UTC
    - cron: '0 2 * * *'
  
  workflow_dispatch:

permissions:
  contents: read
  security-events: write

jobs:
  scan:
    name: 'Security Scan'
    runs-on: ubuntu-latest
    
    steps:
      - name: Checkout code
        uses: actions/checkout@v4
      
      - name: Run Trivy vulnerability scanner
        uses: aquasecurity/trivy-action@master
        with:
          scan-type: 'config'
          scan-ref: '.'
          format: 'sarif'
          output: 'trivy-results.sarif'
      
      - name: Upload Trivy results to GitHub Security
        uses: github/codeql-action/upload-sarif@v3
        with:
          sarif_file: 'trivy-results.sarif'
      
      - name: Run Checkov
        uses: bridgecrewio/checkov-action@master
        with:
          directory: .
          framework: terraform
          output_format: sarif
          output_file_path: checkov-results.sarif
      
      - name: Upload Checkov results
        uses: github/codeql-action/upload-sarif@v3
        with:
          sarif_file: checkov-results.sarif
      
      - name: Secret Scanning
        uses: trufflesecurity/trufflehog@main
        with:
          path: ./
          base: main
          head: HEAD
```

**6. CODEOWNERS:**

```
# DevOps team owns all Terraform code
* @devops-team

# Production requires additional approval
/environments/prod/ @devops-team @platform-team

# Modules require review from architects
/modules/ @devops-team @architecture-team

# CI/CD changes require security review
/.github/workflows/ @devops-team @security-team
```

**7. .tflint.hcl:**

```hcl
plugin "aws" {
  enabled = true
  version = "0.29.0"
  source  = "github.com/terraform-linters/tflint-ruleset-aws"
}

rule "terraform_naming_convention" {
  enabled = true
  format  = "snake_case"
}

rule "terraform_required_version" {
  enabled = true
}

rule "terraform_required_providers" {
  enabled = true
}

rule "terraform_unused_declarations" {
  enabled = true
}

rule "aws_instance_invalid_type" {
  enabled = true
}

rule "aws_s3_bucket_versioning_enabled" {
  enabled = true
}
```

**8. .checkov.yml:**

```yaml
# Checkov configuration
framework:
  - terraform

soft-fail: false

# Skip specific checks
skip-check:
  - CKV_AWS_18  # ELB logging
  - CKV_AWS_19  # S3 encryption

# Custom checks directory
external-checks-dir:
  - ./custom-checks

# Output format
output: cli

# Compact output
compact: true

# Download external modules
download-external-modules: true
```

---

## Часть 2: GitLab CI/CD

### 💻 Задание: Настройка GitLab CI

**1. .gitlab-ci.yml:**

```yaml
image:
  name: hashicorp/terraform:1.6
  entrypoint: [""]

variables:
  TF_ROOT: ${CI_PROJECT_DIR}/environments
  TF_VERSION: "1.6.0"
  AWS_REGION: "us-east-1"
  TF_STATE_NAME: ${CI_ENVIRONMENT_NAME}

# Кэширование для ускорения
cache:
  key: ${CI_COMMIT_REF_SLUG}
  paths:
    - ${TF_ROOT}/**/.terraform
    - ${TF_ROOT}/**/.terraform.lock.hcl

# Stages
stages:
  - validate
  - plan
  - apply
  - destroy

# Before script для всех jobs
before_script:
  - cd ${TF_ROOT}/${CI_ENVIRONMENT_NAME}
  - terraform --version
  - terraform init

# ============================================
# STAGE: VALIDATE
# ============================================

format:
  stage: validate
  script:
    - terraform fmt -check -recursive -diff
  allow_failure: true
  only:
    - merge_requests
    - main

validate:
  stage: validate
  script:
    - terraform validate
  only:
    - merge_requests
    - main

lint:
  stage: validate
  image: ghcr.io/terraform-linters/tflint:latest
  before_script:
    - cd ${TF_ROOT}
    - tflint --init
  script:
    - tflint --recursive --format compact
  allow_failure: true
  only:
    - merge_requests
    - main

security:
  stage: validate
  image: bridgecrew/checkov:latest
  before_script:
    - echo "Running security scan..."
  script:
    - checkov -d ${TF_ROOT} --framework terraform --quiet --compact
  allow_failure: true
  only:
    - merge_requests
    - main
  artifacts:
    reports:
      # GitLab Security Reports
      sast: gl-sast-report.json

# ============================================
# STAGE: PLAN
# ============================================

.plan_template: &plan_template
  stage: plan
  script:
    - |
      terraform plan \
        -out=tfplan \
        -var-file=terraform.tfvars
    
    # Convert plan to JSON for easier parsing
    - terraform show -json tfplan > tfplan.json
    
    # Show summary
    - terraform show tfplan
  artifacts:
    name: plan-${CI_ENVIRONMENT_NAME}
    paths:
      - ${TF_ROOT}/${CI_ENVIRONMENT_NAME}/tfplan
      - ${TF_ROOT}/${CI_ENVIRONMENT_NAME}/tfplan.json
    reports:
      terraform: ${TF_ROOT}/${CI_ENVIRONMENT_NAME}/tfplan.json
    expire_in: 7 days
  only:
    - merge_requests
    - main

plan:dev:
  <<: *plan_template
  environment:
    name: dev
  only:
    - merge_requests
    - main

plan:staging:
  <<: *plan_template
  environment:
    name: staging
  only:
    - merge_requests
    - main

plan:prod:
  <<: *plan_template
  environment:
    name: prod
  only:
    - main
  when: manual

# ============================================
# STAGE: APPLY
# ============================================

.apply_template: &apply_template
  stage: apply
  script:
    - terraform apply -auto-approve tfplan
    
    # Save outputs
    - terraform output -json > outputs.json
    
    # Display important outputs
    - |
      echo "=== Deployment Outputs ==="
      terraform output
  artifacts:
    name: outputs-${CI_ENVIRONMENT_NAME}
    paths:
      - ${TF_ROOT}/${CI_ENVIRONMENT_NAME}/outputs.json
    expire_in: 30 days
  dependencies:
    - plan:${CI_ENVIRONMENT_NAME}

apply:dev:
  <<: *apply_template
  environment:
    name: dev
    on_stop: destroy:dev
  dependencies:
    - plan:dev
  only:
    - main

apply:staging:
  <<: *apply_template
  environment:
    name: staging
    on_stop: destroy:staging
  dependencies:
    - plan:staging
  only:
    - main
  when: manual  # Require manual approval

apply:prod:
  <<: *apply_template
  environment:
    name: prod
    on_stop: destroy:prod
  dependencies:
    - plan:prod
  only:
    - main
  when: manual  # Require manual approval
  # Require protected environment approval

# ============================================
# STAGE: DESTROY
# ============================================

.destroy_template: &destroy_template
  stage: destroy
  script:
    - terraform destroy -auto-approve -var-file=terraform.tfvars
  when: manual
  only:
    - main

destroy:dev:
  <<: *destroy_template
  environment:
    name: dev
    action: stop

destroy:staging:
  <<: *destroy_template
  environment:
    name: staging
    action: stop

destroy:prod:
  <<: *destroy_template
  environment:
    name: prod
    action: stop
  # Additional confirmation required for prod
  needs:
    - apply:prod
```

**2. GitLab CI Variables (Settings → CI/CD → Variables):**

```bash
# AWS Credentials (per environment)
AWS_ACCESS_KEY_ID_DEV         # Protected, Masked
AWS_SECRET_ACCESS_KEY_DEV     # Protected, Masked
AWS_ACCESS_KEY_ID_STAGING     # Protected, Masked
AWS_SECRET_ACCESS_KEY_STAGING # Protected, Masked
AWS_ACCESS_KEY_ID_PROD        # Protected, Masked
AWS_SECRET_ACCESS_KEY_PROD    # Protected, Masked

# Terraform Backend
TF_STATE_BUCKET               # terraform-state-bucket
TF_STATE_DYNAMODB_TABLE       # terraform-locks

# API Keys
INFRACOST_API_KEY             # For cost estimation
CHECKOV_API_KEY               # For security scanning
```

**3. Дополнительный .gitlab-ci.yml с расширенными возможностями:**

```yaml
# Advanced GitLab CI with cost estimation and notifications

include:
  - template: Security/SAST.gitlab-ci.yml
  - template: Security/Secret-Detection.gitlab-ci.yml

# Add cost estimation job

cost:estimate: 
	stage: plan 
	image: infracost/infracost:latest 
	before_script: - | # Setup Infracost infracost configure set api_key ${INFRACOST_API_KEY} script: - | # Generate cost estimates for all environments for env in dev staging prod; do cd ${TF_ROOT}/${env}


    infracost breakdown \
      --path . \
      --format json \
      --out-file /tmp/infracost-${env}.json
  done
  
  # Create comparison
  infracost output \
    --path "/tmp/infracost-*.json" \
    --format gitlab-comment \
    --out-file /tmp/infracost_comment.md
  
  # Post as MR comment
  cat /tmp/infracost_comment.md


artifacts: 
	paths: - /tmp/infracost_comment.md 
	reports: dotenv: /tmp/infracost.env 
	only: - merge_requests

# Slack notifications

.notify_template: &notify_template 
image: curlimages/curl:latest 
script: - | curl -X POST ${SLACK_WEBHOOK_URL}  
-H 'Content-Type: application/json'  
-d "{ "text": "${NOTIFICATION_TEXT}", "attachments": [{ "color": "${COLOR}", "fields": [ {"title": "Project", "value": "${CI_PROJECT_NAME}", "short": true}, {"title": "Environment", "value": "${CI_ENVIRONMENT_NAME}", "short": true}, {"title": "Branch", "value": "${CI_COMMIT_REF_NAME}", "short": true}, {"title": "Status", "value": "${CI_JOB_STATUS}", "short": true}, {"title": "Pipeline", "value": "${CI_PIPELINE_URL}", "short": false} ] }] }"

notify:success: <<: *notify_template stage: .post variables: NOTIFICATION_TEXT: "✅ Deployment Successful" COLOR: "good" when: on_success only: - main

notify:failure: <<: *notify_template stage: .post variables: NOTIFICATION_TEXT: "❌ Deployment Failed" COLOR: "danger" when: on_failure only: - main

````

---

## Часть 3: Atlantis Setup

### 💻 Задание: Настройка Atlantis для Pull Request Automation

**Atlantis** - это инструмент для автоматизации Terraform в Pull Requests.

**1. atlantis.yaml (в корне репозитория):**

```yaml
version: 3

automerge: false
delete_source_branch_on_merge: false

# Parallel execution
parallel_plan: true
parallel_apply: false

projects:
  # Dev environment
  - name: dev
    dir: environments/dev
    workspace: dev
    terraform_version: v1.6.0
    autoplan:
      when_modified:
        - "**/*.tf"
        - "../../modules/**/*.tf"
      enabled: true
    apply_requirements:
      - approved
    workflow: default

  # Staging environment
  - name: staging
    dir: environments/staging
    workspace: staging
    terraform_version: v1.6.0
    autoplan:
      when_modified:
        - "**/*.tf"
        - "../../modules/**/*.tf"
      enabled: true
    apply_requirements:
      - approved
      - mergeable
    workflow: default

  # Production environment
  - name: prod
    dir: environments/prod
    workspace: prod
    terraform_version: v1.6.0
    autoplan:
      when_modified:
        - "**/*.tf"
        - "../../modules/**/*.tf"
      enabled: true
    apply_requirements:
      - approved
      - mergeable
    workflow: production

# Custom workflows
workflows:
  default:
    plan:
      steps:
        - init
        - plan:
            extra_args: ["-var-file=terraform.tfvars"]
    
    apply:
      steps:
        - apply
  
  production:
    plan:
      steps:
        - run: echo "Planning production changes..."
        - init
        - plan:
            extra_args: ["-var-file=terraform.tfvars"]
        - run: |
            echo "=== PRODUCTION DEPLOYMENT ==="
            echo "This will affect production infrastructure!"
            echo "Please review carefully before applying."
    
    apply:
      steps:
        - run: echo "Applying to production..."
        - apply
        - run: |
            echo "=== Deployment Complete ==="
            echo "Production infrastructure updated successfully."

# Allowed overrides
allowed_overrides:
  - workflow
  - apply_requirements

# Custom commands
allowed_workflow_names:
  - default
  - production
````

**2. server-atlantis.yaml (конфигурация Atlantis сервера):**

```yaml
# Atlantis server configuration
---
# Repository configuration
repos:
  - id: github.com/yourorg/terraform-infrastructure
    allowed_overrides: [workflow, apply_requirements]
    allow_custom_workflows: true
    
    # Branch protection
    branch: main
    
    # Require approval from CODEOWNERS
    apply_requirements: [approved, mergeable]

# GitHub/GitLab integration
gh-user: atlantis-bot
gh-token: ${GITHUB_TOKEN}
gh-webhook-secret: ${GITHUB_WEBHOOK_SECRET}

# GitLab (alternative)
# gitlab-user: atlantis-bot
# gitlab-token: ${GITLAB_TOKEN}
# gitlab-webhook-secret: ${GITLAB_WEBHOOK_SECRET}

# Web server
atlantis-url: https://atlantis.yourcompany.com
port: 4141

# Logging
log-level: info

# Autoplanning
automerge: false

# Checkout strategy
checkout-strategy: merge

# Repo allowlist
repo-allowlist: github.com/yourorg/*

# Slack notifications
slack-token: ${SLACK_TOKEN}

# Data directory
data-dir: /atlantis-data

# TLS
# tls-cert-file: /etc/atlantis/tls.crt
# tls-key-file: /etc/atlantis/tls.key
```

**3. Docker Compose для локального запуска Atlantis:**

```yaml
# docker-compose.yml
version: '3.8'

services:
  atlantis:
    image: ghcr.io/runatlantis/atlantis:latest
    container_name: atlantis
    
    ports:
      - "4141:4141"
    
    environment:
      # GitHub
      ATLANTIS_GH_USER: ${ATLANTIS_GH_USER}
      ATLANTIS_GH_TOKEN: ${ATLANTIS_GH_TOKEN}
      ATLANTIS_GH_WEBHOOK_SECRET: ${ATLANTIS_GH_WEBHOOK_SECRET}
      
      # Server
      ATLANTIS_ATLANTIS_URL: ${ATLANTIS_URL}
      ATLANTIS_REPO_ALLOWLIST: ${ATLANTIS_REPO_ALLOWLIST}
      
      # AWS credentials
      AWS_ACCESS_KEY_ID: ${AWS_ACCESS_KEY_ID}
      AWS_SECRET_ACCESS_KEY: ${AWS_SECRET_ACCESS_KEY}
      AWS_REGION: ${AWS_REGION}
      
      # Terraform
      ATLANTIS_DEFAULT_TF_VERSION: v1.6.0
      
      # Logging
      ATLANTIS_LOG_LEVEL: info
    
    volumes:
      - ./atlantis-data:/atlantis-data
      - ./atlantis.yaml:/etc/atlantis/repos.yaml:ro
    
    command: server
    
    restart: unless-stopped
    
    healthcheck:
      test: ["CMD", "curl", "-f", "http://localhost:4141/healthz"]
      interval: 30s
      timeout: 10s
      retries: 3

  # Optional: Ngrok для локального тестирования
  ngrok:
    image: ngrok/ngrok:latest
    container_name: atlantis-ngrok
    
    environment:
      NGROK_AUTHTOKEN: ${NGROK_AUTHTOKEN}
    
    command: http atlantis:4141
    
    depends_on:
      - atlantis
```

**4. Kubernetes Deployment для Atlantis:**

```yaml
# atlantis-deployment.yaml
---
apiVersion: v1
kind: Namespace
metadata:
  name: atlantis

---
apiVersion: v1
kind: Secret
metadata:
  name: atlantis-secrets
  namespace: atlantis
type: Opaque
stringData:
  github-token: ${GITHUB_TOKEN}
  github-webhook-secret: ${GITHUB_WEBHOOK_SECRET}
  aws-access-key-id: ${AWS_ACCESS_KEY_ID}
  aws-secret-access-key: ${AWS_SECRET_ACCESS_KEY}

---
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: atlantis-data
  namespace: atlantis
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 20Gi

---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: atlantis
  namespace: atlantis
spec:
  replicas: 1
  selector:
    matchLabels:
      app: atlantis
  template:
    metadata:
      labels:
        app: atlantis
    spec:
      securityContext:
        fsGroup: 1000
        runAsUser: 100
      
      containers:
        - name: atlantis
          image: ghcr.io/runatlantis/atlantis:latest
          
          ports:
            - name: http
              containerPort: 4141
          
          env:
            - name: ATLANTIS_GH_USER
              value: "atlantis-bot"
            
            - name: ATLANTIS_GH_TOKEN
              valueFrom:
                secretKeyRef:
                  name: atlantis-secrets
                  key: github-token
            
            - name: ATLANTIS_GH_WEBHOOK_SECRET
              valueFrom:
                secretKeyRef:
                  name: atlantis-secrets
                  key: github-webhook-secret
            
            - name: ATLANTIS_REPO_ALLOWLIST
              value: "github.com/yourorg/*"
            
            - name: ATLANTIS_ATLANTIS_URL
              value: "https://atlantis.yourcompany.com"
            
            - name: AWS_ACCESS_KEY_ID
              valueFrom:
                secretKeyRef:
                  name: atlantis-secrets
                  key: aws-access-key-id
            
            - name: AWS_SECRET_ACCESS_KEY
              valueFrom:
                secretKeyRef:
                  name: atlantis-secrets
                  key: aws-secret-access-key
            
            - name: AWS_REGION
              value: "us-east-1"
            
            - name: ATLANTIS_DATA_DIR
              value: "/atlantis-data"
            
            - name: ATLANTIS_LOG_LEVEL
              value: "info"
          
          volumeMounts:
            - name: atlantis-data
              mountPath: /atlantis-data
            
            - name: config
              mountPath: /etc/atlantis
              readOnly: true
          
          livenessProbe:
            httpGet:
              path: /healthz
              port: http
            initialDelaySeconds: 30
            periodSeconds: 10
          
          readinessProbe:
            httpGet:
              path: /healthz
              port: http
            initialDelaySeconds: 10
            periodSeconds: 5
          
          resources:
            requests:
              memory: "512Mi"
              cpu: "250m"
            limits:
              memory: "2Gi"
              cpu: "1000m"
      
      volumes:
        - name: atlantis-data
          persistentVolumeClaim:
            claimName: atlantis-data
        
        - name: config
          configMap:
            name: atlantis-config

---
apiVersion: v1
kind: Service
metadata:
  name: atlantis
  namespace: atlantis
spec:
  type: ClusterIP
  ports:
    - port: 80
      targetPort: http
      protocol: TCP
      name: http
  selector:
    app: atlantis

---
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: atlantis
  namespace: atlantis
  annotations:
    cert-manager.io/cluster-issuer: letsencrypt-prod
    nginx.ingress.kubernetes.io/ssl-redirect: "true"
spec:
  ingressClassName: nginx
  tls:
    - hosts:
        - atlantis.yourcompany.com
      secretName: atlantis-tls
  rules:
    - host: atlantis.yourcompany.com
      http:
        paths:
          - path: /
            pathType: Prefix
            backend:
              service:
                name: atlantis
                port:
                  name: http

---
apiVersion: v1
kind: ConfigMap
metadata:
  name: atlantis-config
  namespace: atlantis
data:
  repos.yaml: |
    repos:
      - id: github.com/yourorg/terraform-infrastructure
        allowed_overrides: [workflow, apply_requirements]
        allow_custom_workflows: true
        apply_requirements: [approved, mergeable]
```

**5. Atlantis Commands (в Pull Request комментариях):**

```bash
# Plan для всех проектов
atlantis plan

# Plan для конкретного проекта
atlantis plan -p dev

# Apply для всех проектов
atlantis apply

# Apply для конкретного проекта
atlantis apply -p dev

# Plan с дополнительными аргументами
atlantis plan -p prod -- -target=aws_instance.web

# Unlock (если plan/apply застрял)
atlantis unlock

# Version
atlantis version

# Help
atlantis help
```

**6. Webhook Setup (GitHub):**

```bash
# GitHub Repository → Settings → Webhooks → Add webhook

Payload URL: https://atlantis.yourcompany.com/events
Content type: application/json
Secret: ${GITHUB_WEBHOOK_SECRET}

Events to trigger:
☑ Pull requests
☑ Issue comments
☑ Push

Active: ☑
```

---

## Часть 4: Automated Testing in Pipeline

### 💻 Задание: Добавление автоматизированного тестирования

**1. tests/vpc_test.go (Terratest):**

```go
package test

import (
	"testing"

	"github.com/gruntwork-io/terratest/modules/aws"
	"github.com/gruntwork-io/terratest/modules/terraform"
	"github.com/stretchr/testify/assert"
)

func TestVPCModule(t *testing.T) {
	t.Parallel()

	// AWS Region
	awsRegion := "us-east-1"

	// Terraform options
	terraformOptions := terraform.WithDefaultRetryableErrors(t, &terraform.Options{
		TerraformDir: "../modules/vpc",
		
		Vars: map[string]interface{}{
			"vpc_name":             "test-vpc",
			"vpc_cidr":             "10.0.0.0/16",
			"availability_zones":   []string{"us-east-1a", "us-east-1b"},
			"public_subnet_cidrs":  []string{"10.0.1.0/24", "10.0.2.0/24"},
			"private_subnet_cidrs": []string{"10.0.11.0/24", "10.0.12.0/24"},
			"enable_nat_gateway":   false,
		},
		
		EnvVars: map[string]string{
			"AWS_DEFAULT_REGION": awsRegion,
		},
	})

	// Cleanup
	defer terraform.Destroy(t, terraformOptions)

	// Deploy
	terraform.InitAndApply(t, terraformOptions)

	// Test outputs
	vpcID := terraform.Output(t, terraformOptions, "vpc_id")
	assert.NotEmpty(t, vpcID)

	publicSubnetIDs := terraform.OutputList(t, terraformOptions, "public_subnet_ids")
	assert.Len(t, publicSubnetIDs, 2)

	privateSubnetIDs := terraform.OutputList(t, terraformOptions, "private_subnet_ids")
	assert.Len(t, privateSubnetIDs, 2)

	// Verify VPC exists in AWS
	vpc := aws.GetVpcById(t, vpcID, awsRegion)
	assert.Equal(t, "10.0.0.0/16", vpc.CidrBlock)

	// Verify subnets
	for _, subnetID := range publicSubnetIDs {
		subnet := aws.GetSubnetById(t, subnetID, awsRegion)
		assert.True(t, subnet.MapPublicIpOnLaunch)
	}

	for _, subnetID := range privateSubnetIDs {
		subnet := aws.GetSubnetById(t, subnetID, awsRegion)
		assert.False(t, subnet.MapPublicIpOnLaunch)
	}
}

func TestVPCWithNATGateway(t *testing.T) {
	t.Parallel()

	awsRegion := "us-east-1"

	terraformOptions := &terraform.Options{
		TerraformDir: "../modules/vpc",
		
		Vars: map[string]interface{}{
			"vpc_name":             "test-vpc-nat",
			"vpc_cidr":             "10.1.0.0/16",
			"availability_zones":   []string{"us-east-1a", "us-east-1b"},
			"public_subnet_cidrs":  []string{"10.1.1.0/24", "10.1.2.0/24"},
			"private_subnet_cidrs": []string{"10.1.11.0/24", "10.1.12.0/24"},
			"enable_nat_gateway":   true,
		},
		
		EnvVars: map[string]string{
			"AWS_DEFAULT_REGION": awsRegion,
		},
	}

	defer terraform.Destroy(t, terraformOptions)
	terraform.InitAndApply(t, terraformOptions)

	// Verify NAT Gateway was created
	natGatewayID := terraform.Output(t, terraformOptions, "nat_gateway_id")
	assert.NotEmpty(t, natGatewayID)
}
```

**2. tests/go.mod:**

```go
module github.com/yourorg/terraform-infrastructure/tests

go 1.21

require (
	github.com/gruntwork-io/terratest v0.46.0
	github.com/stretchr/testify v1.8.4
)
```

**3. .github/workflows/test.yml:**

```yaml
name: 'Terratest'

on:
  pull_request:
    branches:
      - main
    paths:
      - 'modules/**'
      - 'tests/**'
  
  workflow_dispatch:

permissions:
  contents: read
  id-token: write

env:
  GO_VERSION: '1.21'
  AWS_REGION: 'us-east-1'

jobs:
  test:
    name: 'Run Terratest'
    runs-on: ubuntu-latest
    
    steps:
      - name: Checkout code
        uses: actions/checkout@v4
      
      - name: Setup Go
        uses: actions/setup-go@v5
        with:
          go-version: ${{ env.GO_VERSION }}
      
      - name: Setup Terraform
        uses: hashicorp/setup-terraform@v3
        with:
          terraform_version: 1.6.0
          terraform_wrapper: false
      
      - name: Configure AWS Credentials
        uses: aws-actions/configure-aws-credentials@v4
        with:
          role-to-assume: ${{ secrets.AWS_ROLE_TEST }}
          aws-region: ${{ env.AWS_REGION }}
      
      - name: Go Cache
        uses: actions/cache@v4
        with:
          path: |
            ~/.cache/go-build
            ~/go/pkg/mod
          key: ${{ runner.os }}-go-${{ hashFiles('**/go.sum') }}
          restore-keys: |
            ${{ runner.os }}-go-
      
      - name: Download Go Modules
        working-directory: tests
        run: go mod download
      
      - name: Run Tests
        working-directory: tests
        run: |
          go test -v -timeout 30m -parallel 4 ./...
        env:
          AWS_DEFAULT_REGION: ${{ env.AWS_REGION }}
      
      - name: Test Summary
        if: always()
        run: |
          echo "## Test Results" >> $GITHUB_STEP_SUMMARY
          echo "" >> $GITHUB_STEP_SUMMARY
          
          if [ $? -eq 0 ]; then
            echo "✅ All tests passed" >> $GITHUB_STEP_SUMMARY
          else
            echo "❌ Some tests failed" >> $GITHUB_STEP_SUMMARY
          fi
```

**4. Makefile для локального тестирования:**

```makefile
# Makefile для тестирования

.PHONY: test test-fast test-module clean

# Run all tests
test:
	cd tests && go test -v -timeout 30m -parallel 4 ./...

# Run tests without cleanup (faster for development)
test-fast:
	cd tests && TF_ACC=1 go test -v -timeout 15m ./...

# Test specific module
test-module:
	cd tests && go test -v -timeout 30m -run $(MODULE)

# Clean test cache
clean:
	cd tests && go clean -testcache

# Format test code
fmt:
	cd tests && go fmt ./...

# Lint test code
lint:
	cd tests && golangci-lint run
```

---

## Заключение и Best Practices

### 📚 Ключевые takeaways

**1. CI/CD Pipeline Components:**

```
Essential Pipeline Stages:
1. ✅ Validate (fmt, validate, lint)
2. ✅ Security Scan (checkov, tfsec, trivy)
3. ✅ Plan (preview changes)
4. ✅ Test (terratest)
5. ✅ Cost Estimation (infracost)
6. ✅ Apply (with approval)
7. ✅ Verify (health checks)
```

**2. Security Best Practices:**

```hcl
✅ DO:
- Store credentials in CI secrets
- Use OIDC for AWS authentication
- Scan for secrets in code
- Require PR approvals
- Use protected branches
- Enable branch protection rules
- Audit all changes

❌ DON'T:
- Commit credentials to Git
- Allow direct pushes to main
- Skip security scans
- Auto-approve production changes
```

**3. Workflow Best Practices:**

```yaml
✅ DO:
- Plan on every PR
- Apply only on merge to main
- Use environment-specific workflows
- Require manual approval for production
- Comment plan results on PR
- Cache Terraform providers
- Use remote state
- Enable state locking

❌ DON'T:
- Apply without planning
- Skip validation steps
- Use local state in CI
- Deploy to prod without review
```

**4. Testing Strategy:**

```
Testing Levels:
1. Static Analysis
   - terraform fmt
   - terraform validate
   - tflint
   
2. Security Testing
   - checkov
   - tfsec
   - trivy
   
3. Integration Testing
   - terratest
   - kitchen-terraform
   
4. Acceptance Testing
   - Deploy to dev/staging
   - Run smoke tests
   - Health checks
```

**5. Atlantis vs GitHub Actions:**

|Feature|Atlantis|GitHub Actions|
|---|---|---|
|Setup|Requires server|Built-in|
|PR Comments|Native|Requires scripts|
|Plan/Apply UI|Excellent|Good|
|Custom Workflows|Limited|Unlimited|
|Multi-repo|Easy|Complex|
|Cost|Self-hosted|Free tier available|
|Best For|Large teams, many repos|Single repo, full control|

**6. Production Checklist:**

```
Before deploying to production:
□ All tests passing
□ Security scans clear
□ Cost estimated and approved
□ PR approved by CODEOWNERS
□ Plan reviewed and understood
□ Rollback plan prepared
□ Monitoring ready
□ Documentation updated
□ Stakeholders notified
```

Ты освоил:

- ✅ GitHub Actions полный workflow
- ✅ GitLab CI/CD конфигурация
- ✅ Atlantis setup и использование
- ✅ Automated testing с Terratest
- ✅ Security scanning в pipeline
- ✅ Cost estimation интеграция
- ✅ Best practices для production

**Следующие шаги:**

1. Практика на реальных проектах
2. Настройка Terraform Cloud/Enterprise
3. Изучение Policy as Code (Sentinel/OPA)
4. Advanced testing strategies
5. Multi-region deployments
6. Disaster recovery automation

---
# Модуль 13: Security & Secrets Management (30 минут)

### 🎯 Напоминалка

**Security в Terraform - ключевые принципы:**

```
Security Layers
├── Secrets Management (никогда не храни secrets в коде!)
├── State Security (шифрование state файлов)
├── Access Control (IAM, RBAC)
├── Encryption (at-rest и in-transit)
└── Audit & Compliance (логирование изменений)
```

**⚠️ КРИТИЧНО: Что НИКОГДА не делать:**

```hcl
# ❌ НИКОГДА так не делай!
variable "database_password" {
  default = "SuperSecret123!"  # Пароль в коде!
}

resource "aws_db_instance" "bad" {
  password = "hardcoded_password"  # Хардкод пароля!
}

# ❌ Не коммить в Git
terraform.tfvars  # Содержит секреты!
*.tfstate         # Может содержать секреты в plain text!
```

**✅ Правильный подход:**

```hcl
# ✅ Используй sensitive переменные
variable "database_password" {
  type      = string
  sensitive = true  # Скрывает из логов
}

# ✅ Получай секреты из внешних источников
data "aws_secretsmanager_secret_version" "db_password" {
  secret_id = "myapp/database/password"
}

resource "aws_db_instance" "good" {
  password = data.aws_secretsmanager_secret_version.db_password.secret_string
}
```

**AWS Secrets Manager:**

```hcl
# Создание секрета
resource "aws_secretsmanager_secret" "db_password" {
  name_prefix             = "myapp-db-password-"
  recovery_window_in_days = 30
  
  tags = {
    Environment = var.environment
  }
}

resource "aws_secretsmanager_secret_version" "db_password" {
  secret_id     = aws_secretsmanager_secret.db_password.id
  secret_string = random_password.db_password.result
}

# Генерация случайного пароля
resource "random_password" "db_password" {
  length  = 32
  special = true
  
  lifecycle {
    ignore_changes = [
      special,  # Не пересоздавать при каждом apply
    ]
  }
}

# Чтение секрета
data "aws_secretsmanager_secret" "db_password" {
  name = "myapp/database/password"
}

data "aws_secretsmanager_secret_version" "db_password" {
  secret_id = data.aws_secretsmanager_secret.db_password.id
}

# Использование
resource "aws_db_instance" "main" {
  password = data.aws_secretsmanager_secret_version.db_password.secret_string
}
```

**HashiCorp Vault Provider:**

```hcl
# Настройка Vault provider
terraform {
  required_providers {
    vault = {
      source  = "hashicorp/vault"
      version = "~> 3.20"
    }
  }
}

provider "vault" {
  address = "https://vault.example.com:8200"
  
  # Аутентификация через токен (из переменной окружения VAULT_TOKEN)
  # или через другие методы
}

# Чтение секрета из Vault
data "vault_generic_secret" "db_credentials" {
  path = "secret/database/credentials"
}

# Использование
resource "aws_db_instance" "main" {
  username = data.vault_generic_secret.db_credentials.data["username"]
  password = data.vault_generic_secret.db_credentials.data["password"]
}

# Создание секрета в Vault
resource "vault_generic_secret" "api_key" {
  path = "secret/myapp/api_key"
  
  data_json = jsonencode({
    api_key    = random_password.api_key.result
    created_at = timestamp()
  })
}
```

**Sensitive Data Handling:**

```hcl
# Sensitive variables
variable "api_key" {
  type      = string
  sensitive = true
  
  validation {
    condition     = length(var.api_key) >= 32
    error_message = "API key must be at least 32 characters."
  }
}

# Sensitive locals
locals {
  db_connection_string = sensitive(
    "postgresql://${var.db_user}:${var.db_password}@${aws_db_instance.main.endpoint}"
  )
}

# Sensitive outputs
output "db_password" {
  value     = random_password.db_password.result
  sensitive = true
}

output "connection_string" {
  value     = local.db_connection_string
  sensitive = true
}
```

**Encryption Best Practices:**

```hcl
# KMS для шифрования
resource "aws_kms_key" "main" {
  description             = "KMS key for ${var.project_name}"
  deletion_window_in_days = 30
  enable_key_rotation     = true
  
  tags = {
    Name = "${var.project_name}-kms-key"
  }
}

resource "aws_kms_alias" "main" {
  name          = "alias/${var.project_name}"
  target_key_id = aws_kms_key.main.key_id
}

# S3 bucket с шифрованием
resource "aws_s3_bucket" "encrypted" {
  bucket = "${var.project_name}-encrypted"
}

resource "aws_s3_bucket_server_side_encryption_configuration" "encrypted" {
  bucket = aws_s3_bucket.encrypted.id
  
  rule {
    apply_server_side_encryption_by_default {
      sse_algorithm     = "aws:kms"
      kms_master_key_id = aws_kms_key.main.arn
    }
    bucket_key_enabled = true
  }
}

# RDS с шифрованием
resource "aws_db_instance" "encrypted" {
  identifier = "${var.project_name}-db"
  
  storage_encrypted = true
  kms_key_id       = aws_kms_key.main.arn
  
  # ... остальные параметры
}

# EBS volume с шифрованием
resource "aws_ebs_volume" "encrypted" {
  availability_zone = var.availability_zone
  size             = 20
  
  encrypted  = true
  kms_key_id = aws_kms_key.main.arn
}
```

**State File Security:**

```hcl
# backend.tf - безопасный S3 backend
terraform {
  backend "s3" {
    bucket         = "my-terraform-state"
    key            = "project/terraform.tfstate"
    region         = "us-east-1"
    encrypt        = true  # Шифрование state
    kms_key_id     = "arn:aws:kms:us-east-1:ACCOUNT:key/KEY_ID"
    dynamodb_table = "terraform-locks"
    
    # Дополнительная безопасность
    acl                  = "private"
    skip_region_validation      = false
    skip_credentials_validation = false
  }
}
```

**IAM Policies для Security:**

```hcl
# IAM policy для Secrets Manager
data "aws_iam_policy_document" "secrets_access" {
  statement {
    effect = "Allow"
    
    actions = [
      "secretsmanager:GetSecretValue",
      "secretsmanager:DescribeSecret"
    ]
    
    resources = [
      "arn:aws:secretsmanager:${data.aws_region.current.name}:${data.aws_caller_identity.current.account_id}:secret:${var.project_name}/*"
    ]
    
    condition {
      test     = "StringEquals"
      variable = "aws:RequestedRegion"
      values   = [data.aws_region.current.name]
    }
  }
  
  statement {
    effect = "Allow"
    
    actions = [
      "kms:Decrypt",
      "kms:DescribeKey"
    ]
    
    resources = [aws_kms_key.main.arn]
  }
}

resource "aws_iam_policy" "secrets_access" {
  name_prefix = "${var.project_name}-secrets-access-"
  description = "Allow access to application secrets"
  policy      = data.aws_iam_policy_document.secrets_access.json
}
```

**Password Rotation:**

```hcl
# Автоматическая ротация паролей
resource "aws_secretsmanager_secret_rotation" "db_password" {
  secret_id           = aws_secretsmanager_secret.db_password.id
  rotation_lambda_arn = aws_lambda_function.rotate_secret.arn
  
  rotation_rules {
    automatically_after_days = 30
  }
}

# Lambda для ротации
resource "aws_lambda_function" "rotate_secret" {
  filename      = "rotation_lambda.zip"
  function_name = "${var.project_name}-secret-rotation"
  role          = aws_iam_role.lambda_rotation.arn
  handler       = "index.handler"
  runtime       = "python3.11"
  
  environment {
    variables = {
      SECRETS_MANAGER_ENDPOINT = "https://secretsmanager.${data.aws_region.current.name}.amazonaws.com"
    }
  }
}
```

### 💻 Задание

Создай production-ready security setup с полным управлением секретами:

**1. Структура проекта:**

```bash
mkdir terraform-security
cd terraform-security
mkdir -p modules/{kms,secrets,iam} scripts lambda
```

**2. provider.tf:**

```hcl
terraform {
  required_version = ">= 1.6.0"
  
  required_providers {
    aws = {
      source  = "hashicorp/aws"
      version = "~> 5.0"
    }
    random = {
      source  = "hashicorp/random"
      version = "~> 3.5"
    }
    archive = {
      source  = "hashicorp/archive"
      version = "~> 2.4"
    }
  }
  
  # Безопасный backend
  backend "s3" {
    # Конфигурация через backend-config файл
  }
}

provider "aws" {
  region = var.aws_region
  
  default_tags {
    tags = local.common_tags
  }
}
```

**3. variables.tf:**

```hcl
variable "aws_region" {
  description = "AWS region"
  type        = string
  default     = "us-east-1"
}

variable "project_name" {
  description = "Project name"
  type        = string
  
  validation {
    condition     = can(regex("^[a-z][a-z0-9-]*$", var.project_name))
    error_message = "Project name must start with lowercase letter."
  }
}

variable "environment" {
  description = "Environment name"
  type        = string
  
  validation {
    condition     = contains(["dev", "staging", "prod"], var.environment)
    error_message = "Must be dev, staging, or prod."
  }
}

# Sensitive variables
variable "master_admin_email" {
  description = "Master admin email for notifications"
  type        = string
  sensitive   = true
  
  validation {
    condition     = can(regex("^[a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\\.[a-zA-Z]{2,}$", var.master_admin_email))
    error_message = "Must be a valid email address."
  }
}

variable "allowed_cidr_blocks" {
  description = "Allowed CIDR blocks for access"
  type        = list(string)
  default     = []
  
  validation {
    condition = alltrue([
      for cidr in var.allowed_cidr_blocks :
      can(cidrhost(cidr, 0))
    ])
    error_message = "All CIDR blocks must be valid."
  }
}

variable "enable_encryption" {
  description = "Enable encryption for all resources"
  type        = bool
  default     = true
}

variable "enable_password_rotation" {
  description = "Enable automatic password rotation"
  type        = bool
  default     = false
}

variable "rotation_days" {
  description = "Password rotation interval in days"
  type        = number
  default     = 30
  
  validation {
    condition     = var.rotation_days >= 7 && var.rotation_days <= 365
    error_message = "Rotation days must be between 7 and 365."
  }
}

variable "secret_recovery_window_days" {
  description = "Recovery window for deleted secrets"
  type        = number
  default     = 30
  
  validation {
    condition     = var.secret_recovery_window_days >= 7 && var.secret_recovery_window_days <= 30
    error_message = "Recovery window must be between 7 and 30 days."
  }
}
```

**4. data-sources.tf:**

```hcl
data "aws_region" "current" {}
data "aws_caller_identity" "current" {}
data "aws_partition" "current" {}

# Получить существующий VPC
data "aws_vpc" "default" {
  default = true
}

data "aws_subnets" "default" {
  filter {
    name   = "vpc-id"
    values = [data.aws_vpc.default.id]
  }
}

# IAM policy для текущего пользователя
data "aws_iam_policy_document" "current_user" {
  statement {
    effect = "Allow"
    
    principals {
      type        = "AWS"
      identifiers = [data.aws_caller_identity.current.arn]
    }
    
    actions = ["*"]
    resources = ["*"]
  }
}
```

**5. locals.tf:**

```hcl
locals {
  name_prefix = "${var.project_name}-${var.environment}"
  
  # Security settings по окружениям
  security_config = {
    dev = {
      kms_deletion_window     = 7
      enable_key_rotation     = false
      require_mfa            = false
      password_length        = 16
      backup_retention_days  = 7
    }
    staging = {
      kms_deletion_window     = 14
      enable_key_rotation     = true
      require_mfa            = false
      password_length        = 24
      backup_retention_days  = 14
    }
    prod = {
      kms_deletion_window     = 30
      enable_key_rotation     = true
      require_mfa            = true
      password_length        = 32
      backup_retention_days  = 30
    }
  }
  
  current_security = local.security_config[var.environment]
  
  common_tags = {
    Project       = var.project_name
    Environment   = var.environment
    ManagedBy     = "Terraform"
    SecurityLevel = var.environment == "prod" ? "High" : "Medium"
    Compliance    = var.environment == "prod" ? "PCI-DSS,SOC2" : "Internal"
  }
  
  # Список секретов для управления
  secrets_config = {
    database = {
      description = "Database master credentials"
      type        = "credentials"
      rotation    = var.enable_password_rotation
    }
    api_keys = {
      description = "External API keys"
      type        = "api_key"
      rotation    = false
    }
    ssl_certs = {
      description = "SSL/TLS certificates"
      type        = "certificate"
      rotation    = false
    }
    encryption_keys = {
      description = "Application encryption keys"
      type        = "encryption_key"
      rotation    = var.enable_password_rotation
    }
  }
}
```

**6. kms.tf:**

```hcl
# Основной KMS ключ
resource "aws_kms_key" "main" {
  description             = "Main KMS key for ${local.name_prefix}"
  deletion_window_in_days = local.current_security.kms_deletion_window
  enable_key_rotation     = local.current_security.enable_key_rotation
  
  # Policy для ключа
  policy = jsonencode({
    Version = "2012-10-17"
    Statement = [
      {
        Sid    = "Enable IAM User Permissions"
        Effect = "Allow"
        Principal = {
          AWS = "arn:${data.aws_partition.current.partition}:iam::${data.aws_caller_identity.current.account_id}:root"
        }
        Action   = "kms:*"
        Resource = "*"
      },
      {
        Sid    = "Allow CloudWatch Logs"
        Effect = "Allow"
        Principal = {
          Service = "logs.${data.aws_region.current.name}.amazonaws.com"
        }
        Action = [
          "kms:Encrypt",
          "kms:Decrypt",
          "kms:ReEncrypt*",
          "kms:GenerateDataKey*",
          "kms:CreateGrant",
          "kms:DescribeKey"
        ]
        Resource = "*"
      },
      {
        Sid    = "Allow Secrets Manager"
        Effect = "Allow"
        Principal = {
          Service = "secretsmanager.amazonaws.com"
        }
        Action = [
          "kms:Decrypt",
          "kms:GenerateDataKey"
        ]
        Resource = "*"
      }
    ]
  })
  
  tags = merge(local.common_tags, {
    Name = "${local.name_prefix}-main-kms"
  })
}

resource "aws_kms_alias" "main" {
  name          = "alias/${local.name_prefix}-main"
  target_key_id = aws_kms_key.main.key_id
}

# Дополнительные KMS ключи для разных целей
resource "aws_kms_key" "secrets" {
  description             = "KMS key for secrets encryption"
  deletion_window_in_days = local.current_security.kms_deletion_window
  enable_key_rotation     = local.current_security.enable_key_rotation
  
  tags = merge(local.common_tags, {
    Name    = "${local.name_prefix}-secrets-kms"
    Purpose = "SecretsEncryption"
  })
}

resource "aws_kms_alias" "secrets" {
  name          = "alias/${local.name_prefix}-secrets"
  target_key_id = aws_kms_key.secrets.key_id
}

resource "aws_kms_key" "logs" {
  description             = "KMS key for CloudWatch Logs encryption"
  deletion_window_in_days = local.current_security.kms_deletion_window
  enable_key_rotation     = local.current_security.enable_key_rotation
  
  tags = merge(local.common_tags, {
    Name    = "${local.name_prefix}-logs-kms"
    Purpose = "LogsEncryption"
  })
}

resource "aws_kms_alias" "logs" {
  name          = "alias/${local.name_prefix}-logs"
  target_key_id = aws_kms_key.logs.key_id
}

# Grant для EC2 instances
resource "aws_kms_grant" "ec2_decrypt" {
  name              = "${local.name_prefix}-ec2-decrypt"
  key_id            = aws_kms_key.secrets.key_id
  grantee_principal = aws_iam_role.ec2_instance.arn
  
  operations = [
    "Decrypt",
    "DescribeKey"
  ]
}
```

**7. secrets-manager.tf:**

```hcl
# Database credentials
resource "random_password" "db_master" {
  length  = local.current_security.password_length
  special = true
  
  # Избегаем проблемных символов
  override_special = "!#$%&*()-_=+[]{}<>:?"
  
  lifecycle {
    ignore_changes = [
      length,
      special,
      override_special
    ]
  }
}

resource "aws_secretsmanager_secret" "db_master" {
  name_prefix             = "${local.name_prefix}-db-master-"
  description             = "Master database credentials"
  recovery_window_in_days = var.secret_recovery_window_days
  kms_key_id             = aws_kms_key.secrets.id
  
  tags = merge(local.common_tags, {
    Name        = "${local.name_prefix}-db-master"
    SecretType  = "database"
    Rotation    = tostring(var.enable_password_rotation)
  })
}

resource "aws_secretsmanager_secret_version" "db_master" {
  secret_id = aws_secretsmanager_secret.db_master.id
  
  secret_string = jsonencode({
    username = "admin"
    password = random_password.db_master.result
    engine   = "postgres"
    host     = "localhost"  # Будет обновлено после создания RDS
    port     = 5432
    dbname   = var.project_name
  })
  
  lifecycle {
    ignore_changes = [
      secret_string  # Будет обновлено ротацией
    ]
  }
}

# API Keys
resource "random_password" "api_key" {
  length  = 64
  special = false
  upper   = true
  lower   = true
  numeric = true
}

resource "aws_secretsmanager_secret" "api_key" {
  name_prefix             = "${local.name_prefix}-api-key-"
  description             = "External API key"
  recovery_window_in_days = var.secret_recovery_window_days
  kms_key_id             = aws_kms_key.secrets.id
  
  tags = merge(local.common_tags, {
    Name       = "${local.name_prefix}-api-key"
    SecretType = "api_key"
  })
}

resource "aws_secretsmanager_secret_version" "api_key" {
  secret_id = aws_secretsmanager_secret.api_key.id
  
  secret_string = jsonencode({
    api_key    = random_password.api_key.result
    created_at = timestamp()
    expires_at = timeadd(timestamp(), "8760h")  # 1 year
  })
}

# Application encryption key
resource "random_password" "app_encryption_key" {
  length  = 32
  special = false
}

resource "aws_secretsmanager_secret" "app_encryption_key" {
  name_prefix             = "${local.name_prefix}-app-key-"
  description             = "Application encryption key"
  recovery_window_in_days = var.secret_recovery_window_days
  kms_key_id             = aws_kms_key.secrets.id
  
  tags = merge(local.common_tags, {
    Name       = "${local.name_prefix}-app-key"
    SecretType = "encryption_key"
  })
}

resource "aws_secretsmanager_secret_version" "app_encryption_key" {
  secret_id = aws_secretsmanager_secret.app_encryption_key.id
  
  secret_string = jsonencode({
    key        = random_password.app_encryption_key.result
    algorithm  = "AES-256-GCM"
    created_at = timestamp()
  })
}

# TLS Certificate (placeholder для настоящего сертификата)
resource "aws_secretsmanager_secret" "tls_cert" {
  name_prefix             = "${local.name_prefix}-tls-cert-"
  description             = "TLS certificate and private key"
  recovery_window_in_days = var.secret_recovery_window_days
  kms_key_id             = aws_kms_key.secrets.id
  
  tags = merge(local.common_tags, {
    Name       = "${local.name_prefix}-tls-cert"
    SecretType = "certificate"
  })
}

resource "aws_secretsmanager_secret_version" "tls_cert" {
  secret_id = aws_secretsmanager_secret.tls_cert.id
  
  secret_string = jsonencode({
    certificate = "-----BEGIN CERTIFICATE-----\nPlaceholder\n-----END CERTIFICATE-----"
    private_key = "-----BEGIN PRIVATE KEY-----\nPlaceholder\n-----END PRIVATE KEY-----"
    created_at  = timestamp()
    expires_at  = timeadd(timestamp(), "8760h")
  })
  
  lifecycle {
    ignore_changes = [secret_string]
  }
}

# Secret для admin notifications
resource "aws_secretsmanager_secret" "admin_config" {
  name_prefix             = "${local.name_prefix}-admin-"
  description             = "Admin configuration"
  recovery_window_in_days = var.secret_recovery_window_days
  kms_key_id             = aws_kms_key.secrets.id
  
  tags = merge(local.common_tags, {
    Name       = "${local.name_prefix}-admin-config"
    SecretType = "config"
  })
}

resource "aws_secretsmanager_secret_version" "admin_config" {
  secret_id = aws_secretsmanager_secret.admin_config.id
  
  secret_string = jsonencode({
    admin_email = var.master_admin_email
    alert_phone = "+1234567890"  # Placeholder
    pagerduty_key = "placeholder"
  })
  
  lifecycle {
    ignore_changes = [secret_string]
  }
}
```

**8. rotation-lambda.tf:**

```hcl
# Lambda для ротации секретов (только если включено)
data "archive_file" "rotation_lambda" {
  count = var.enable_password_rotation ? 1 : 0
  
  type        = "zip"
  source_dir  = "${path.module}/lambda/rotation"
  output_path = "${path.module}/lambda/rotation.zip"
}

resource "aws_lambda_function" "rotate_secret" {
  count = var.enable_password_rotation ? 1 : 0
  
  filename      = data.archive_file.rotation_lambda[0].output_path
  function_name = "${local.name_prefix}-secret-rotation"
  role          = aws_iam_role.lambda_rotation[0].arn
  handler       = "index.handler"
  runtime       = "python3.11"
  timeout       = 30
  
  source_code_hash = data.archive_file.rotation_lambda[0].output_base64sha256
  
  environment {
    variables = {
      SECRETS_MANAGER_ENDPOINT = "https://secretsmanager.${data.aws_region.current.name}.amazonaws.com"
      EXCLUDE_CHARACTERS       = "/@\"'\\"
    }
  }
  
  vpc_config {
    subnet_ids         = data.aws_subnets.default.ids
    security_group_ids = [aws_security_group.lambda_rotation[0].id]
  }
  
  tags = merge(local.common_tags, {
    Name = "${local.name_prefix}-rotation-lambda"
  })
}

# Security Group для Lambda
resource "aws_security_group" "lambda_rotation" {
  count = var.enable_password_rotation ? 1 : 0
  
  name_prefix = "${local.name_prefix}-lambda-rotation-"
  description = "Security group for secret rotation Lambda"
  vpc_id      = data.aws_vpc.default.id
  
  egress {
    from_port   = 443
    to_port     = 443
    protocol    = "tcp"
    cidr_blocks = ["0.0.0.0/0"]
    description = "HTTPS to AWS services"
  }
  
  tags = merge(local.common_tags, {
    Name = "${local.name_prefix}-lambda-rotation-sg"
  })
  
  lifecycle {
    create_before_destroy = true
  }
}

# IAM роль для Lambda
resource "aws_iam_role" "lambda_rotation" {
  count = var.enable_password_rotation ? 1 : 0
  
  name_prefix = "${local.name_prefix}-lambda-rotation-"
  
  assume_role_policy = jsonencode({
    Version = "2012-10-17"
    Statement = [{
      Effect = "Allow"
      Principal = {
        Service = "lambda.amazonaws.com"
      }
      Action = "sts:AssumeRole"
    }]
  })
  
  tags = local.common_tags
}

# IAM policy для Lambda
resource "aws_iam_role_policy" "lambda_rotation" {
  count = var.enable_password_rotation ? 1 : 0
  
  name_prefix = "${local.name_prefix}-lambda-rotation-"
  role        = aws_iam_role.lambda_rotation[0].id
  
  policy = jsonencode({
    Version = "2012-10-17"
    Statement = [
      {
        Effect = "Allow"
        Action = [
          "secretsmanager:DescribeSecret",
          "secretsmanager:GetSecretValue",
          "secretsmanager:PutSecretValue",
          "secretsmanager:UpdateSecretVersionStage"
        ]
        Resource = [
          aws_secretsmanager_secret.db_master.arn,
          aws_secretsmanager_secret.app_encryption_key.arn
        ]
      },
      {
        Effect = "Allow"
        Action = [
          "secretsmanager:GetRandomPassword"
        ]
        Resource = "*"
      },
      {
        Effect = "Allow"
        Action = [
          "kms:Decrypt",
          "kms:DescribeKey",
          "kms:GenerateDataKey"
        ]
        Resource = aws_kms_key.secrets.arn
      },
      {
        Effect = "Allow"
        Action = [
          "ec2:CreateNetworkInterface",
          "ec2:DescribeNetworkInterfaces",
          "ec2:DeleteNetworkInterface",
          "ec2:AssignPrivateIpAddresses",
          "ec2:UnassignPrivateIpAddresses"
        ]
        Resource = "*"
      },
      {
        Effect = "Allow"
        Action = [
          "logs:CreateLogGroup",
          "logs:CreateLogStream",
          "logs:PutLogEvents"
        ]
        Resource = "arn:${data.aws_partition.current.partition}:logs:*:*:*"
      }
    ]
  })
}

# Настройка ротации для секретов
resource "aws_secretsmanager_secret_rotation" "db_master" {
  count = var.enable_password_rotation ? 1 : 0

  secret_id           = aws_secretsmanager_secret.db_master.id
  rotation_lambda_arn = aws_lambda_function.rotate_secret[0].arn

  rotation_rules {
    automatically_after_days = var.rotation_days
  }

  depends_on = [aws_lambda_function.rotate_secret, aws_iam_role_policy.lambda_rotation]
}

# Permission для Lambda вызова из Secrets Manager
resource "aws_lambda_permission" "allow_secrets_manager" {
  count = var.enable_password_rotation ? 1 : 0

  statement_id  = "AllowExecutionFromSecretsManager"
  action        = "lambda:InvokeFunction"
  function_name = aws_lambda_function.rotate_secret[0].function_name
  principal     = "secretsmanager.amazonaws.com"
}

````

**9. lambda/rotation/index.py:**

```python
import json
import boto3
import os
from typing import Dict, Any

secrets_client = boto3.client('secretsmanager')

def handler(event: Dict[str, Any], context: Any) -> None:
    """
    Lambda handler для ротации секретов AWS Secrets Manager
    """
    # Получаем параметры из event
    arn = event['SecretId']
    token = event['ClientRequestToken']
    step = event['Step']
    
    # Metadata секрета
    metadata = secrets_client.describe_secret(SecretId=arn)
    
    # Проверяем что версия для ротации существует
    if not metadata.get('RotationEnabled'):
        raise ValueError(f"Secret {arn} is not enabled for rotation")
    
    versions = metadata['VersionIdsToStages']
    if token not in versions:
        raise ValueError(f"Secret version {token} has no stage for rotation")
    
    if "AWSCURRENT" in versions[token]:
        print(f"Secret version {token} already set as AWSCURRENT")
        return
    elif "AWSPENDING" not in versions[token]:
        raise ValueError(f"Secret version {token} not set as AWSPENDING for rotation")
    
    # Выполняем шаги ротации
    if step == "createSecret":
        create_secret(secrets_client, arn, token)
    elif step == "setSecret":
        set_secret(secrets_client, arn, token)
    elif step == "testSecret":
        test_secret(secrets_client, arn, token)
    elif step == "finishSecret":
        finish_secret(secrets_client, arn, token)
    else:
        raise ValueError(f"Invalid step parameter: {step}")


def create_secret(client, arn: str, token: str) -> None:
    """Создать новую версию секрета"""
    print(f"Creating new secret version for {arn}")
    
    # Получаем текущий секрет
    current_secret = client.get_secret_value(
        SecretId=arn,
        VersionStage="AWSCURRENT"
    )
    
    secret_dict = json.loads(current_secret['SecretString'])
    
    # Генерируем новый пароль
    new_password = client.get_random_password(
        PasswordLength=32,
        ExcludeCharacters=os.environ.get('EXCLUDE_CHARACTERS', '/@"\'\\')
    )
    
    # Обновляем пароль
    secret_dict['password'] = new_password['RandomPassword']
    
    # Создаем новую версию
    client.put_secret_value(
        SecretId=arn,
        ClientRequestToken=token,
        SecretString=json.dumps(secret_dict),
        VersionStages=['AWSPENDING']
    )
    
    print(f"Successfully created new secret version {token}")


def set_secret(client, arn: str, token: str) -> None:
    """Установить новый секрет в целевой системе"""
    print(f"Setting secret in target system for {arn}")
    
    # Получаем новый секрет
    pending_secret = client.get_secret_value(
        SecretId=arn,
        VersionId=token,
        VersionStage="AWSPENDING"
    )
    
    secret_dict = json.loads(pending_secret['SecretString'])
    
    # Здесь должна быть логика обновления пароля в целевой системе
    # Например, обновление пароля в RDS
    # rds_client.modify_db_instance(...)
    
    print(f"Successfully set new secret in target system")


def test_secret(client, arn: str, token: str) -> None:
    """Протестировать новый секрет"""
    print(f"Testing new secret for {arn}")
    
    # Получаем новый секрет
    pending_secret = client.get_secret_value(
        SecretId=arn,
        VersionId=token,
        VersionStage="AWSPENDING"
    )
    
    secret_dict = json.loads(pending_secret['SecretString'])
    
    # Здесь должна быть логика тестирования подключения
    # Например, тест подключения к базе данных
    # test_db_connection(secret_dict)
    
    print(f"Successfully tested new secret")


def finish_secret(client, arn: str, token: str) -> None:
    """Завершить ротацию - переключить AWSCURRENT на новую версию"""
    print(f"Finishing secret rotation for {arn}")
    
    # Получаем metadata
    metadata = client.describe_secret(SecretId=arn)
    
    # Находим current version
    current_version = None
    for version, stages in metadata['VersionIdsToStages'].items():
        if "AWSCURRENT" in stages:
            if version == token:
                # Уже current
                print(f"Version {token} already marked as AWSCURRENT")
                return
            current_version = version
            break
    
    # Переключаем stages
    client.update_secret_version_stage(
        SecretId=arn,
        VersionStage="AWSCURRENT",
        MoveToVersionId=token,
        RemoveFromVersionId=current_version
    )
    
    print(f"Successfully finished secret rotation for {arn}")
````

**10. iam-roles.tf:**

```hcl
# IAM роль для EC2 instances
resource "aws_iam_role" "ec2_instance" {
  name_prefix = "${local.name_prefix}-ec2-"
  
  assume_role_policy = jsonencode({
    Version = "2012-10-17"
    Statement = [{
      Effect = "Allow"
      Principal = {
        Service = "ec2.amazonaws.com"
      }
      Action = "sts:AssumeRole"
    }]
  })
  
  tags = local.common_tags
}

# Policy для доступа к секретам
resource "aws_iam_role_policy" "ec2_secrets_access" {
  name_prefix = "${local.name_prefix}-ec2-secrets-"
  role        = aws_iam_role.ec2_instance.id
  
  policy = jsonencode({
    Version = "2012-10-17"
    Statement = [
      {
        Effect = "Allow"
        Action = [
          "secretsmanager:GetSecretValue",
          "secretsmanager:DescribeSecret"
        ]
        Resource = [
          aws_secretsmanager_secret.db_master.arn,
          aws_secretsmanager_secret.api_key.arn,
          aws_secretsmanager_secret.app_encryption_key.arn
        ]
      },
      {
        Effect = "Allow"
        Action = [
          "kms:Decrypt",
          "kms:DescribeKey"
        ]
        Resource = aws_kms_key.secrets.arn
      },
      {
        Effect = "Allow"
        Action = [
          "logs:CreateLogGroup",
          "logs:CreateLogStream",
          "logs:PutLogEvents"
        ]
        Resource = "*"
      }
    ]
  })
}

# Instance profile
resource "aws_iam_instance_profile" "ec2" {
  name_prefix = "${local.name_prefix}-ec2-"
  role        = aws_iam_role.ec2_instance.name
  
  tags = local.common_tags
}

# Policy для ограниченного доступа (read-only)
data "aws_iam_policy_document" "readonly_secrets" {
  statement {
    effect = "Allow"
    
    actions = [
      "secretsmanager:GetSecretValue",
      "secretsmanager:DescribeSecret",
      "secretsmanager:ListSecrets"
    ]
    
    resources = [
      "arn:${data.aws_partition.current.partition}:secretsmanager:${data.aws_region.current.name}:${data.aws_caller_identity.current.account_id}:secret:${var.project_name}/*"
    ]
    
    condition {
      test     = "StringEquals"
      variable = "secretsmanager:ResourceTag/Environment"
      values   = [var.environment]
    }
  }
  
  statement {
    effect = "Allow"
    
    actions = [
      "kms:Decrypt",
      "kms:DescribeKey"
    ]
    
    resources = [
      aws_kms_key.secrets.arn
    ]
  }
}

resource "aws_iam_policy" "readonly_secrets" {
  name_prefix = "${local.name_prefix}-readonly-secrets-"
  description = "Read-only access to application secrets"
  policy      = data.aws_iam_policy_document.readonly_secrets.json
  
  tags = local.common_tags
}
```

**11. database.tf:**

```hcl
# RDS Instance с зашифрованными credentials
resource "aws_db_subnet_group" "main" {
  name_prefix = "${local.name_prefix}-db-"
  subnet_ids  = data.aws_subnets.default.ids
  
  tags = merge(local.common_tags, {
    Name = "${local.name_prefix}-db-subnet-group"
  })
}

resource "aws_security_group" "database" {
  name_prefix = "${local.name_prefix}-db-"
  description = "Security group for RDS database"
  vpc_id      = data.aws_vpc.default.id
  
  ingress {
    from_port       = 5432
    to_port         = 5432
    protocol        = "tcp"
    security_groups = [aws_security_group.app.id]
    description     = "PostgreSQL from app servers"
  }
  
  egress {
    from_port   = 0
    to_port     = 0
    protocol    = "-1"
    cidr_blocks = ["0.0.0.0/0"]
    description = "All outbound"
  }
  
  tags = merge(local.common_tags, {
    Name = "${local.name_prefix}-db-sg"
  })
  
  lifecycle {
    create_before_destroy = true
  }
}

# Получаем credentials из Secrets Manager
data "aws_secretsmanager_secret_version" "db_master" {
  secret_id  = aws_secretsmanager_secret.db_master.id
  depends_on = [aws_secretsmanager_secret_version.db_master]
}

locals {
  db_creds = jsondecode(data.aws_secretsmanager_secret_version.db_master.secret_string)
}

resource "aws_db_instance" "main" {
  identifier     = "${local.name_prefix}-db"
  engine         = "postgres"
  engine_version = "15.4"
  instance_class = var.environment == "prod" ? "db.t3.medium" : "db.t3.micro"
  
  allocated_storage     = 20
  max_allocated_storage = 100
  storage_type          = "gp3"
  storage_encrypted     = true
  kms_key_id           = aws_kms_key.main.arn
  
  db_name  = var.project_name
  username = local.db_creds.username
  password = local.db_creds.password
  
  db_subnet_group_name   = aws_db_subnet_group.main.name
  vpc_security_group_ids = [aws_security_group.database.id]
  
  backup_retention_period = local.current_security.backup_retention_days
  backup_window          = "03:00-04:00"
  maintenance_window     = "mon:04:00-mon:05:00"
  
  enabled_cloudwatch_logs_exports = ["postgresql", "upgrade"]
  
  deletion_protection = var.environment == "prod"
  skip_final_snapshot = var.environment != "prod"
  final_snapshot_identifier = var.environment == "prod" ? "${local.name_prefix}-final-snapshot-${formatdate("YYYY-MM-DD-hhmm", timestamp())}" : null
  
  # Автоматическое обновление minor versions
  auto_minor_version_upgrade = true
  
  tags = merge(local.common_tags, {
    Name = "${local.name_prefix}-db"
  })
  
  lifecycle {
    ignore_changes = [
      password  # Управляется через Secrets Manager rotation
    ]
  }
}

# Обновляем secret с актуальным endpoint
resource "null_resource" "update_db_secret" {
  triggers = {
    db_endpoint = aws_db_instance.main.endpoint
  }
  
  provisioner "local-exec" {
    command = <<-EOT
      aws secretsmanager put-secret-value \
        --secret-id ${aws_secretsmanager_secret.db_master.id} \
        --secret-string '{
          "username": "${local.db_creds.username}",
          "password": "${local.db_creds.password}",
          "engine": "postgres",
          "host": "${element(split(":", aws_db_instance.main.endpoint), 0)}",
          "port": ${aws_db_instance.main.port},
          "dbname": "${aws_db_instance.main.db_name}"
        }' \
        --region ${data.aws_region.current.name}
    EOT
  }
  
  depends_on = [aws_db_instance.main]
}
```

**12. application-resources.tf:**

```hcl
# Security Group для приложения
resource "aws_security_group" "app" {
  name_prefix = "${local.name_prefix}-app-"
  description = "Security group for application servers"
  vpc_id      = data.aws_vpc.default.id
  
  ingress {
    from_port   = 443
    to_port     = 443
    protocol    = "tcp"
    cidr_blocks = var.allowed_cidr_blocks
    description = "HTTPS from allowed networks"
  }
  
  ingress {
    from_port   = 22
    to_port     = 22
    protocol    = "tcp"
    cidr_blocks = var.allowed_cidr_blocks
    description = "SSH from allowed networks"
  }
  
  egress {
    from_port   = 0
    to_port     = 0
    protocol    = "-1"
    cidr_blocks = ["0.0.0.0/0"]
    description = "All outbound"
  }
  
  tags = merge(local.common_tags, {
    Name = "${local.name_prefix}-app-sg"
  })
  
  lifecycle {
    create_before_destroy = true
  }
}

# User data script для безопасной загрузки секретов
data "template_file" "user_data" {
  template = file("${path.module}/scripts/user_data.sh")
  
  vars = {
    region              = data.aws_region.current.name
    db_secret_arn       = aws_secretsmanager_secret.db_master.arn
    api_key_secret_arn  = aws_secretsmanager_secret.api_key.arn
    app_key_secret_arn  = aws_secretsmanager_secret.app_encryption_key.arn
    project_name        = var.project_name
    environment         = var.environment
  }
}

# Launch Template
resource "aws_launch_template" "app" {
  name_prefix   = "${local.name_prefix}-app-"
  image_id      = data.aws_ami.amazon_linux.id
  instance_type = var.environment == "prod" ? "t3.medium" : "t3.micro"
  
  iam_instance_profile {
    arn = aws_iam_instance_profile.ec2.arn
  }
  
  vpc_security_group_ids = [aws_security_group.app.id]
  
  user_data = base64encode(data.template_file.user_data.rendered)
  
  metadata_options {
    http_endpoint               = "enabled"
    http_tokens                 = "required"  # IMDSv2 обязательно
    http_put_response_hop_limit = 1
    instance_metadata_tags      = "enabled"
  }
  
  monitoring {
    enabled = local.current_security.monitoring_enabled
  }
  
  block_device_mappings {
    device_name = "/dev/xvda"
    
    ebs {
      volume_size           = 20
      volume_type           = "gp3"
      encrypted             = true
      kms_key_id           = aws_kms_key.main.arn
      delete_on_termination = true
    }
  }
  
  tag_specifications {
    resource_type = "instance"
    tags = merge(local.common_tags, {
      Name = "${local.name_prefix}-app"
    })
  }
  
  tag_specifications {
    resource_type = "volume"
    tags = merge(local.common_tags, {
      Name = "${local.name_prefix}-app-volume"
    })
  }
  
  lifecycle {
    create_before_destroy = true
  }
  
  tags = merge(local.common_tags, {
    Name = "${local.name_prefix}-app-lt"
  })
}

# AMI data source
data "aws_ami" "amazon_linux" {
  most_recent = true
  owners      = ["amazon"]
  
  filter {
    name   = "name"
    values = ["al2023-ami-*-x86_64"]
  }
  
  filter {
    name   = "virtualization-type"
    values = ["hvm"]
  }
}
```

**13. scripts/user_data.sh:**

```bash
#!/bin/bash
set -e

# Логирование
exec > >(tee /var/log/user-data.log)
exec 2>&1

echo "=== Security-Enhanced User Data Script ==="
echo "Starting at: $(date)"

# Обновление системы
yum update -y

# Установка AWS CLI и jq
yum install -y aws-cli jq

# Функция для безопасного получения секретов
get_secret() {
    local secret_arn=$1
    local output_file=$2
    local max_retries=5
    local retry_count=0
    
    echo "Retrieving secret: $secret_arn"
    
    while [ $retry_count -lt $max_retries ]; do
        if aws secretsmanager get-secret-value \
            --secret-id "$secret_arn" \
            --region "${region}" \
            --query SecretString \
            --output text > "$output_file" 2>/dev/null; then
            
            # Установить строгие permissions
            chmod 600 "$output_file"
            echo "Successfully retrieved secret"
            return 0
        fi
        
        retry_count=$((retry_count + 1))
        echo "Retry $retry_count/$max_retries..."
        sleep 5
    done
    
    echo "Failed to retrieve secret after $max_retries attempts"
    return 1
}

# Создать безопасную директорию для секретов
SECRETS_DIR="/opt/app/secrets"
mkdir -p "$SECRETS_DIR"
chmod 700 "$SECRETS_DIR"

# Получить секреты
get_secret "${db_secret_arn}" "$SECRETS_DIR/db_credentials.json"
get_secret "${api_key_secret_arn}" "$SECRETS_DIR/api_key.json"
get_secret "${app_key_secret_arn}" "$SECRETS_DIR/app_key.json"

# Установить приложение
mkdir -p /opt/app
cd /opt/app

# Создать конфигурацию приложения
cat > /opt/app/config.json <<EOF
{
  "project": "${project_name}",
  "environment": "${environment}",
  "region": "${region}",
  "secrets_dir": "$SECRETS_DIR",
  "security": {
    "encryption_at_rest": true,
    "tls_enabled": true,
    "min_tls_version": "1.2"
  }
}
EOF

chmod 600 /opt/app/config.json

# Создать systemd service для приложения
cat > /etc/systemd/system/myapp.service <<'EOF'
[Unit]
Description=My Secure Application
After=network.target

[Service]
Type=simple
User=app
Group=app
WorkingDirectory=/opt/app
Environment="SECRETS_DIR=/opt/app/secrets"
ExecStart=/opt/app/start.sh
Restart=always
RestartSec=10

# Security hardening
NoNewPrivileges=true
PrivateTmp=true
ProtectSystem=strict
ProtectHome=true
ReadWritePaths=/opt/app

[Install]
WantedBy=multi-user.target
EOF

# Создать пользователя для приложения
useradd -r -s /bin/false app
chown -R app:app /opt/app

# Настроить логирование в CloudWatch (опционально)
cat > /opt/app/start.sh <<'STARTSCRIPT'
#!/bin/bash

# Загрузить секреты в переменные окружения
export DB_CREDS=$(cat $SECRETS_DIR/db_credentials.json)
export API_KEY=$(cat $SECRETS_DIR/api_key.json | jq -r '.api_key')

# Запустить приложение
echo "Application starting with secure configuration..."

# Placeholder для реального приложения
while true; do
    echo "App running securely at $(date)"
    sleep 60
done
STARTSCRIPT

chmod +x /opt/app/start.sh

# Включить и запустить service
systemctl daemon-reload
systemctl enable myapp
systemctl start myapp

# Настроить автоматическое обновление секретов
cat > /usr/local/bin/refresh-secrets.sh <<'EOF'
#!/bin/bash
set -e

SECRETS_DIR="/opt/app/secrets"

# Функция получения секрета
get_secret() {
    aws secretsmanager get-secret-value \
        --secret-id "$1" \
        --region "${region}" \
        --query SecretString \
        --output text > "$2"
    chmod 600 "$2"
}

# Обновить все секреты
get_secret "${db_secret_arn}" "$SECRETS_DIR/db_credentials.json"
get_secret "${api_key_secret_arn}" "$SECRETS_DIR/api_key.json"
get_secret "${app_key_secret_arn}" "$SECRETS_DIR/app_key.json"

# Перезапустить приложение для применения новых секретов
systemctl restart myapp

echo "Secrets refreshed at $(date)"
EOF

chmod +x /usr/local/bin/refresh-secrets.sh

# Настроить cron для периодического обновления секретов
cat > /etc/cron.d/refresh-secrets <<EOF
# Обновлять секреты каждый час
0 * * * * root /usr/local/bin/refresh-secrets.sh >> /var/log/refresh-secrets.log 2>&1
EOF

# Настроить fail2ban для дополнительной безопасности
yum install -y fail2ban
systemctl enable fail2ban
systemctl start fail2ban

# Настроить firewall
yum install -y firewalld
systemctl enable firewalld
systemctl start firewalld

# Разрешить только необходимые порты
firewall-cmd --permanent --add-service=https
firewall-cmd --permanent --add-service=ssh
firewall-cmd --reload

echo "=== User Data Script Completed ==="
echo "Finished at: $(date)"
```

**14. cloudwatch-logs.tf:**

```hcl
# CloudWatch Log Group для audit logs
resource "aws_cloudwatch_log_group" "security_audit" {
  name              = "/aws/${var.project_name}/${var.environment}/security/audit"
  retention_in_days = var.environment == "prod" ? 90 : 30
  kms_key_id        = aws_kms_key.logs.arn
  
  tags = merge(local.common_tags, {
    Name    = "${local.name_prefix}-security-audit"
    Purpose = "SecurityAudit"
  })
}

# CloudWatch Log Group для application logs
resource "aws_cloudwatch_log_group" "application" {
  name              = "/aws/${var.project_name}/${var.environment}/application"
  retention_in_days = var.environment == "prod" ? 30 : 7
  kms_key_id        = aws_kms_key.logs.arn
  
  tags = merge(local.common_tags, {
    Name = "${local.name_prefix}-application-logs"
  })
}

# CloudWatch Log Group для secrets access
resource "aws_cloudwatch_log_group" "secrets_access" {
  name              = "/aws/${var.project_name}/${var.environment}/secrets/access"
  retention_in_days = var.environment == "prod" ? 365 : 90
  kms_key_id        = aws_kms_key.logs.arn
  
  tags = merge(local.common_tags, {
    Name    = "${local.name_prefix}-secrets-access"
    Purpose = "AuditTrail"
  })
}

# Metric Filter для мониторинга доступа к секретам
resource "aws_cloudwatch_log_metric_filter" "unauthorized_secret_access" {
  name           = "${local.name_prefix}-unauthorized-secret-access"
  log_group_name = aws_cloudwatch_log_group.secrets_access.name
  
  pattern = "[time, request_id, event_type = AccessDenied*, ...]"
  
  metric_transformation {
    name      = "UnauthorizedSecretAccess"
    namespace = "${var.project_name}/Security"
    value     = "1"
    
    dimensions = {
      Environment = var.environment
    }
  }
}

# Alarm для unauthorized access
resource "aws_cloudwatch_metric_alarm" "unauthorized_access" {
  alarm_name          = "${local.name_prefix}-unauthorized-secret-access"
  comparison_operator = "GreaterThanThreshold"
  evaluation_periods  = "1"
  metric_name         = "UnauthorizedSecretAccess"
  namespace           = "${var.project_name}/Security"
  period              = "300"
  statistic           = "Sum"
  threshold           = "5"
  alarm_description   = "Alert on multiple unauthorized secret access attempts"
  treat_missing_data  = "notBreaching"
  
  dimensions = {
    Environment = var.environment
  }
  
  alarm_actions = [aws_sns_topic.security_alerts.arn]
  
  tags = local.common_tags
}

# SNS Topic для security alerts
resource "aws_sns_topic" "security_alerts" {
  name_prefix       = "${local.name_prefix}-security-alerts-"
  kms_master_key_id = aws_kms_key.main.id
  
  tags = merge(local.common_tags, {
    Name    = "${local.name_prefix}-security-alerts"
    Purpose = "SecurityNotifications"
  })
}

resource "aws_sns_topic_subscription" "security_alerts_email" {
  topic_arn = aws_sns_topic.security_alerts.arn
  protocol  = "email"
  endpoint  = var.master_admin_email
}
```

**15. s3-secure-storage.tf:**

```hcl
# S3 bucket для secure storage
resource "aws_s3_bucket" "secure_storage" {
  bucket_prefix = "${local.name_prefix}-secure-"
  
  tags = merge(local.common_tags, {
    Name    = "${local.name_prefix}-secure-storage"
    Purpose = "SecureDataStorage"
  })
}

# Versioning
resource "aws_s3_bucket_versioning" "secure_storage" {
  bucket = aws_s3_bucket.secure_storage.id
  
  versioning_configuration {
    status = "Enabled"
  }
}

# Server-side encryption с KMS
resource "aws_s3_bucket_server_side_encryption_configuration" "secure_storage" {
  bucket = aws_s3_bucket.secure_storage.id
  
  rule {
    apply_server_side_encryption_by_default {
      sse_algorithm     = "aws:kms"
      kms_master_key_id = aws_kms_key.main.arn
    }
    bucket_key_enabled = true
  }
}

# Блокировка публичного доступа
resource "aws_s3_bucket_public_access_block" "secure_storage" {
  bucket = aws_s3_bucket.secure_storage.id
  
  block_public_acls       = true
  block_public_policy     = true
  ignore_public_acls      = true
  restrict_public_buckets = true
}

# Lifecycle policy
resource "aws_s3_bucket_lifecycle_configuration" "secure_storage" {
  bucket = aws_s3_bucket.secure_storage.id
  
  rule {
    id     = "archive-old-versions"
    status = "Enabled"
    
    noncurrent_version_transition {
      noncurrent_days = 30
      storage_class   = "GLACIER"
    }
    
    noncurrent_version_expiration {
      noncurrent_days = 90
    }
  }
  
  rule {
    id     = "delete-incomplete-uploads"
    status = "Enabled"
    
    abort_incomplete_multipart_upload {
      days_after_initiation = 7
    }
  }
}

# Bucket policy для ограничения доступа
resource "aws_s3_bucket_policy" "secure_storage" {
  bucket = aws_s3_bucket.secure_storage.id
  
  policy = jsonencode({
    Version = "2012-10-17"
    Statement = [
      {
        Sid    = "DenyUnencryptedObjectUploads"
        Effect = "Deny"
        Principal = "*"
        Action = "s3:PutObject"
        Resource = "${aws_s3_bucket.secure_storage.arn}/*"
        Condition = {
          StringNotEquals = {
            "s3:x-amz-server-side-encryption" = "aws:kms"
          }
        }
      },
      {
        Sid    = "DenyInsecureTransport"
        Effect = "Deny"
        Principal = "*"
        Action = "s3:*"
        Resource = [
          aws_s3_bucket.secure_storage.arn,
          "${aws_s3_bucket.secure_storage.arn}/*"
        ]
        Condition = {
          Bool = {
            "aws:SecureTransport" = "false"
          }
        }
      },
      {
        Sid    = "AllowEC2InstanceAccess"
        Effect = "Allow"
        Principal = {
          AWS = aws_iam_role.ec2_instance.arn
        }
        Action = [
          "s3:GetObject",
          "s3:PutObject"
        ]
        Resource = "${aws_s3_bucket.secure_storage.arn}/*"
      }
    ]
  })
}

# Logging bucket
resource "aws_s3_bucket" "access_logs" {
  bucket_prefix = "${local.name_prefix}-logs-"
  
  tags = merge(local.common_tags, {
    Name    = "${local.name_prefix}-access-logs"
    Purpose = "AuditLogs"
  })
}

resource "aws_s3_bucket_server_side_encryption_configuration" "access_logs" {
  bucket = aws_s3_bucket.access_logs.id
  
  rule {
    apply_server_side_encryption_by_default {
      sse_algorithm = "AES256"
    }
  }
}

resource "aws_s3_bucket_public_access_block" "access_logs" {
  bucket = aws_s3_bucket.access_logs.id
  
  block_public_acls       = true
  block_public_policy     = true
  ignore_public_acls      = true
  restrict_public_buckets = true
}

# Enable logging
resource "aws_s3_bucket_logging" "secure_storage" {
  bucket = aws_s3_bucket.secure_storage.id
  
  target_bucket = aws_s3_bucket.access_logs.id
  target_prefix = "s3-access-logs/"
}
```

**16. outputs.tf:**

```hcl
# KMS Outputs
output "kms_keys" {
  description = "KMS key information"
  value = {
    main = {
      id     = aws_kms_key.main.id
      arn    = aws_kms_key.main.arn
      alias  = aws_kms_alias.main.name
    }
    secrets = {
      id     = aws_kms_key.secrets.id
      arn    = aws_kms_key.secrets.arn
      alias  = aws_kms_alias.secrets.name
    }
    logs = {
      id     = aws_kms_key.logs.id
      arn    = aws_kms_key.logs.arn
      alias  = aws_kms_alias.logs.name
    }
  }
}

# Secrets Manager Outputs (не sensitive данные!)
output "secrets_arns" {
  description = "Secrets Manager ARNs"
  value = {
    database           = aws_secretsmanager_secret.db_master.arn
    api_key           = aws_secretsmanager_secret.api_key.arn
    app_encryption_key = aws_secretsmanager_secret.app_encryption_key.arn
    tls_cert          = aws_secretsmanager_secret.tls_cert.arn
    admin_config      = aws_secretsmanager_secret.admin_config.arn
  }
}

# Database Outputs
output "database_info" {
  description = "Database connection information"
  value = {
    endpoint = aws_db_instance.main.endpoint
    port     = aws_db_instance.main.port
    name     = aws_db_instance.main.db_name
    arn      = aws_db_instance.main.arn
  }
}

# Security Groups
output "security_groups" {
  description = "Security group IDs"
  value = {
    app      = aws_security_group.app.id
    database = aws_security_group.database.id
  }
}

# IAM Roles
output "iam_roles" {
  description = "IAM role ARNs"
  value = {
    ec2_instance      = aws_iam_role.ec2_instance.arn
    instance_profile  = aws_iam_instance_profile.ec2.arn
  }
}

# S3 Buckets
output "s3_buckets" {
  description = "S3 bucket information"
  value = {
    secure_storage = {
      id  = aws_s3_bucket.secure_storage.id
      arn = aws_s3_bucket.secure_storage.arn
    }
    access_logs = {
      id  = aws_s3_bucket.access_logs.id
      arn = aws_s3_bucket.access_logs.arn
    }
  }
}

# CloudWatch Log Groups
output "log_groups" {
  description = "CloudWatch Log Group names"
  value = {
    security_audit  = aws_cloudwatch_log_group.security_audit.name
    application     = aws_cloudwatch_log_group.application.name
    secrets_access  = aws_cloudwatch_log_group.secrets_access.name
  }
}

# Security Configuration Summary
output "security_summary" {
  description = "Security configuration summary"
  value = {
    environment                = var.environment
    encryption_enabled         = var.enable_encryption
    password_rotation_enabled  = var.enable_password_rotation
    rotation_interval_days     = var.rotation_days
    kms_key_rotation          = local.current_security.enable_key_rotation
    password_length           = local.current_security.password_length
    backup_retention_days     = local.current_security.backup_retention_days
    deletion_protection       = var.environment == "prod"
  }
}

# Sensitive outputs (не отображаются в console)
output "db_secret_arn" {
  description = "Database secret ARN for applications"
  value       = aws_secretsmanager_secret.db_master.arn
  sensitive   = true
}

output "api_key_secret_arn" {
  description = "API key secret ARN"
  value       = aws_secretsmanager_secret.api_key.arn
  sensitive   = true
}

# Commands для получения секретов
output "useful_commands" {
  description = "Useful AWS CLI commands"
  value = {
    get_db_credentials = "aws secretsmanager get-secret-value --secret-id ${aws_secretsmanager_secret.db_master.arn} --query SecretString --output text | jq ."
    get_api_key       = "aws secretsmanager get-secret-value --secret-id ${aws_secretsmanager_secret.api_key.arn} --query SecretString --output text | jq -r '.api_key'"
    decrypt_with_kms  = "aws kms decrypt --ciphertext-blob fileb://encrypted.txt --key-id ${aws_kms_key.main.id} --output text --query Plaintext | base64 --decode"
    list_secrets      = "aws secretsmanager list-secrets --filters Key=name,Values=${var.project_name}"
  }
}
```

**17. terraform.tfvars:**

```hcl
project_name = "secure-app"
environment  = "dev"
aws_region   = "us-east-1"

# Security settings
enable_encryption         = true
enable_password_rotation  = false  # Включить для prod
rotation_days            = 30
secret_recovery_window_days = 30

# Network security
allowed_cidr_blocks = [
  "10.0.0.0/8",    # Internal network
  "203.0.113.0/24" # Office IP range (example)
]

# Admin email для notifications
master_admin_email = "admin@example.com"
```

**18. Makefile:**

```makefile
.PHONY: help init plan apply destroy secrets clean

ENVIRONMENT ?= dev

help:
	@echo "Available targets:"
	@echo "  init       - Initialize Terraform"
	@echo "  plan       - Run terraform plan"
	@echo "  apply      - Apply infrastructure changes"
	@echo "  destroy    - Destroy infrastructure"
	@echo "  secrets    - Show secrets information"
	@echo "  clean      - Clean terraform files"
	@echo ""
	@echo "Environment: $(ENVIRONMENT)"

init:
	terraform init

plan:
	terraform plan -var-file=$(ENVIRONMENT).tfvars -out=tfplan

apply:
	terraform apply tfplan

destroy:
	terraform destroy -var-file=$(ENVIRONMENT).tfvars

# Security commands
secrets:
	@echo "=== Secrets Information ==="
	@terraform output -json secrets_arns | jq .
	@echo ""
	@echo "=== Get Database Credentials ==="
	@echo "$$(terraform output -raw useful_commands | jq -r '.get_db_credentials')"
	@echo ""
	@echo "=== Get API Key ==="
	@echo "$$(terraform output -raw useful_commands | jq -r '.get_api_key')"

test-db-connection:
	@echo "Testing database connection..."
	@DB_SECRET=$$(aws secretsmanager get-secret-value \
		--secret-id $$(terraform output -raw db_secret_arn) \
		--query SecretString --output text); \
	DB_HOST=$$(echo $$DB_SECRET | jq -r '.host'); \
	DB_PORT=$$(echo $$DB_SECRET | jq -r '.port'); \
	DB_USER=$$(echo $$DB_SECRET | jq -r '.username'); \
	DB_PASS=$$(echo $$DB_SECRET | jq -r '.password'); \
	DB_NAME=$$(echo $$DB_SECRET | jq -r '.dbname'); \
	PGPASSWORD=$$DB_PASS psql -h $$DB_HOST -p $$DB_PORT -U $$DB_USER -d $$DB_NAME -c "SELECT version();"

rotate-secrets:
	@echo "Triggering secret rotation..."
	@aws secretsmanager rotate-secret \
		--secret-id $$(terraform output -raw db_secret_arn)

audit-secrets:
	@echo "=== Secret Access Audit ==="
	@aws cloudtrail lookup-events \
		--lookup-attributes AttributeKey=ResourceType,AttributeValue=AWS::SecretsManager::Secret \
		--max-results 50 | jq '.Events[] | {EventTime, EventName, Username}'

check-encryption:
	@echo "=== Checking Encryption Status ==="
	@echo "RDS Encryption:"
	@aws rds describe-db-instances --query 'DBInstances[_].[DBInstanceIdentifier,StorageEncrypted,KmsKeyId]' --output table
	@echo ""
	@echo "S3 Encryption:"
	@aws s3api get-bucket-encryption --bucket $$(terraform output -json s3_buckets | jq -r '.secure_storage.id')
	@echo ""
	@echo "EBS Encryption:"
	@aws ec2 describe-volumes --query 'Volumes[_].[VolumeId,Encrypted,KmsKeyId]' --output table

clean:
	rm -rf .terraform terraform.tfstate* tfplan .terraform.lock.hcl

.DEFAULT_GOAL := help
```

**19. Документация: README.md:**

```markdown
# Secure Infrastructure with Terraform

Production-ready infrastructure с полным управлением секретами и шифрованием.

## 🔐 Security Features

- **KMS Encryption**: Все данные зашифрованы с помощью AWS KMS
- **Secrets Manager**: Централизованное управление секретами
- **Password Rotation**: Автоматическая ротация паролей
- **IAM Least Privilege**: Минимальные необходимые права
- **Encrypted Storage**: Шифрование S3, RDS, EBS
- **Audit Logging**: Полное логирование доступа к секретам
- **Network Security**: Security Groups и Network ACLs

## 📋 Prerequisites

```bash
# AWS CLI
aws --version

# Terraform >= 1.6.0
terraform --version

# jq для обработки JSON
jq --version

# PostgreSQL client (для тестирования)
psql --version
````

## 🚀 Quick Start

### 1. Инициализация

```bash
# Настроить AWS credentials
export AWS_PROFILE=your-profile
export AWS_REGION=us-east-1

# Инициализировать Terraform
make init
```

### 2. Планирование

```bash
# Создать план для dev
make plan ENVIRONMENT=dev

# Для production
make plan ENVIRONMENT=prod
```

### 3. Применение

```bash
make apply
```

### 4. Получение секретов

```bash
# Показать ARNs секретов
make secrets

# Получить database credentials
aws secretsmanager get-secret-value \
  --secret-id $(terraform output -raw db_secret_arn) \
  --query SecretString --output text | jq .

# Получить API key
aws secretsmanager get-secret-value \
  --secret-id $(terraform output -raw api_key_secret_arn) \
  --query SecretString --output text | jq -r '.api_key'
```

## 🔄 Password Rotation

### Включение автоматической ротации

```hcl
# В terraform.tfvars
enable_password_rotation = true
rotation_days = 30
```

### Ручная ротация

```bash
make rotate-secrets
```

### Проверка статуса ротации

```bash
aws secretsmanager describe-secret \
  --secret-id $(terraform output -raw db_secret_arn) \
  --query RotationEnabled
```

## 🔍 Security Audit

### Просмотр логов доступа к секретам

```bash
make audit-secrets
```

### Проверка статуса шифрования

```bash
make check-encryption
```

### CloudWatch Logs

```bash
# Security audit logs
aws logs tail /aws/secure-app/dev/security/audit --follow

# Application logs
aws logs tail /aws/secure-app/dev/application --follow
```

## 🏗️ Architecture

```
┌─────────────────────────────────────────────┐
│           KMS Master Keys                    │
│  ┌────────┐  ┌────────┐  ┌────────┐        │
│  │  Main  │  │Secrets │  │  Logs  │        │
│  └────────┘  └────────┘  └────────┘        │
└─────────────────────────────────────────────┘
                    │
                    ▼
┌─────────────────────────────────────────────┐
│         AWS Secrets Manager                  │
│  ┌──────────────┐  ┌──────────────┐        │
│  │ DB Creds     │  │  API Keys    │        │
│  │ (Rotated)    │  │              │        │
│  └──────────────┘  └──────────────┘        │
└─────────────────────────────────────────────┘
                    │
                    ▼
┌─────────────────────────────────────────────┐
│           Application Layer                  │
│  ┌──────────────┐  ┌──────────────┐        │
│  │  EC2 + IAM   │  │   RDS (enc)  │        │
│  │  (IMDSv2)    │  │              │        │
│  └──────────────┘  └──────────────┘        │
└─────────────────────────────────────────────┘
                    │
                    ▼
┌─────────────────────────────────────────────┐
│           Storage Layer                      │
│  ┌──────────────┐  ┌──────────────┐        │
│  │S3 (KMS enc)  │  │ EBS (enc)    │        │
│  └──────────────┘  └──────────────┘        │
└─────────────────────────────────────────────┘
                    │
                    ▼
┌─────────────────────────────────────────────┐
│         Monitoring & Audit                   │
│  CloudWatch Logs + Metrics + Alarms         │
└─────────────────────────────────────────────┘
```

## 🔒 Security Best Practices

### ✅ DO

- Всегда используй Secrets Manager для паролей
- Включай шифрование для всех хранилищ
- Используй IMDSv2 для EC2 metadata
- Настрой password rotation для prod
- Логируй все доступы к секретам
- Используй least privilege IAM policies

### ❌ DON'T

- Не храни секреты в коде
- Не коммить .tfstate в Git
- Не используй хардкод паролей
- Не отключай шифрование
- Не игнорируй security alerts

## 📊 Outputs

```bash
# Показать все outputs
terraform output

# Конкретный output
terraform output kms_keys
terraform output secrets_arns
terraform output security_summary
```

## 🧹 Cleanup

```bash
# Удалить инфраструктуру
make destroy

# Очистить локальные файлы
make clean
```

## 📝 Environment Variables

```bash
# AWS
export AWS_PROFILE=default
export AWS_REGION=us-east-1

# Terraform
export TF_LOG=DEBUG  # Для отладки
export TF_VAR_master_admin_email="admin@example.com"
```

## 🚨 Troubleshooting

### Проблема: Secrets не читаются

```bash
# Проверить IAM permissions
aws iam get-role-policy \
  --role-name $(terraform output -raw iam_roles | jq -r '.ec2_instance') \
  --policy-name secrets-access

# Проверить KMS permissions
aws kms describe-key --key-id $(terraform output -json kms_keys | jq -r '.secrets.id')
```

### Проблема: Ротация не работает

```bash
# Проверить Lambda logs
aws logs tail /aws/lambda/secure-app-dev-secret-rotation --follow

# Проверить Lambda permissions
aws lambda get-policy \
  --function-name secure-app-dev-secret-rotation
```

## 📚 Additional Resources

- [AWS Secrets Manager Best Practices](https://docs.aws.amazon.com/secretsmanager/latest/userguide/best-practices.html)
- [KMS Key Policies](https://docs.aws.amazon.com/kms/latest/developerguide/key-policies.html)
- [Terraform Sensitive Data](https://www.terraform.io/docs/language/values/variables.html#suppressing-values-in-cli-output)

````

**20. Запуск и тестирование:**

```bash
# 1. Инициализация
cd terraform-security
terraform init

# 2. Валидация
terraform validate
terraform fmt -recursive

# 3. План
terraform plan -var-file=terraform.tfvars -out=tfplan

# 4. Применение
terraform apply tfplan

# 5. Проверка секретов
aws secretsmanager list-secrets --filters Key=name,Values=secure-app

# 6. Получение database credentials
DB_SECRET=$(aws secretsmanager get-secret-value \
  --secret-id $(terraform output -raw db_secret_arn) \
  --query SecretString --output text)

echo $DB_SECRET | jq .

# 7. Проверка шифрования RDS
aws rds describe-db-instances \
  --db-instance-identifier $(terraform output -json database_info | jq -r '.arn' | cut -d: -f7) \
  --query 'DBInstances[0].[StorageEncrypted,KmsKeyId]'

# 8. Проверка S3 encryption
aws s3api get-bucket-encryption \
  --bucket $(terraform output -json s3_buckets | jq -r '.secure_storage.id')

# 9. Audit trail
aws cloudtrail lookup-events \
  --lookup-attributes AttributeKey=ResourceType,AttributeValue=AWS::SecretsManager::Secret \
  --max-results 10

# 10. Тестирование rotation (если включено)
if [ "$(terraform output -json security_summary | jq -r '.password_rotation_enabled')" = "true" ]; then
  aws secretsmanager rotate-secret \
    --secret-id $(terraform output -raw db_secret_arn)
fi
````

---

## 🎯 Заключение Модуля 13

### 📚 Ключевые takeaways

**1. Secrets Management Hierarchy:**

```
Security Levels
├── KMS (Infrastructure layer)
│   ├── Master keys
│   ├── Key rotation
│   └── Key policies
├── Secrets Manager (Application layer)
│   ├── Secret storage
│   ├── Automatic rotation
│   └── Version management
├── IAM (Access control)
│   ├── Least privilege
│   ├── Role-based access
│   └── MFA requirements
└── Audit (Compliance)
    ├── CloudWatch Logs
    ├── CloudTrail
    └── Metric Alarms
```

**2. Best Practices Checklist:**

```
✅ Encryption
   ├── KMS for all data at rest
   ├── TLS for data in transit
   └── IMDSv2 for instance metadata

✅ Secrets Management
   ├── Never hardcode secrets
   ├── Use Secrets Manager/Vault
   ├── Enable automatic rotation
   └── Separate secrets per environment

✅ Access Control
   ├── Least privilege IAM
   ├── MFA for sensitive operations
   ├── Service-specific roles
   └── Regular access reviews

✅ Monitoring
   ├── CloudWatch Logs
   ├── Security alerts
   ├── Access audit trails
   └── Automated compliance checks
```

**3. Production Checklist:**

- [ ] KMS keys с rotation enabled
- [ ] Secrets в Secrets Manager (не в коде!)
- [ ] Password rotation configured
- [ ] Encryption enabled везде (RDS, S3, EBS)
- [ ] IAM policies следуют least privilege
- [ ] CloudWatch alarms настроены
- [ ] Audit logging включено
- [ ] Backup и recovery протестированы
- [ ] Security groups minimal
- [ ] Network isolation (VPC, subnets)

**4. Common Mistakes:**

```hcl
# ❌ ПЛОХО
variable "db_password" {
  default = "MyPassword123"  # Hardcoded!
}

# ❌ ПЛОХО
resource "aws_instance" "web" {
  user_data = "PASSWORD=secret123"  # В plain text!
}

# ✅ ХОРОШО
data "aws_secretsmanager_secret_version" "db" {
  secret_id = aws_secretsmanager_secret.db.id
}

resource "aws_db_instance" "main" {
  password = jsondecode(data.aws_secretsmanager_secret_version.db.secret_string)["password"]
}
```

Ты освоил:

- ✅ AWS Secrets Manager integration
- ✅ KMS encryption at rest
- ✅ Password rotation automation
- ✅ IAM least privilege patterns
- ✅ Security audit и monitoring
- ✅ Production-ready security setup

**Next Steps:**

1. Интеграция с HashiCorp Vault (для multi-cloud)
2. Advanced compliance (PCI-DSS, SOC2)
3. Security scanning (tfsec, checkov)
4. Disaster recovery planning

# Модуль 14: Debugging & Troubleshooting (25 минут)

### 🎯 Напоминалка

**Terraform Logging Levels:**

```bash
# Уровни логирования (от минимального до максимального)
TF_LOG Levels
├── TRACE - самый детальный (все операции)
├── DEBUG - детальная отладочная информация
├── INFO  - общая информация о прогрессе
├── WARN  - предупреждения
└── ERROR - только ошибки
```

**Environment Variables для Debugging:**

```bash
# Основные переменные окружения
export TF_LOG=DEBUG                    # Уровень логирования
export TF_LOG_PATH=./terraform.log     # Файл для логов
export TF_LOG_CORE=TRACE              # Логи Terraform Core
export TF_LOG_PROVIDER=DEBUG          # Логи провайдера

# Дополнительные полезные переменные
export TF_INPUT=false                 # Отключить интерактивные промпты
export TF_CLI_ARGS_plan="-no-color"   # Аргументы для plan
export TF_DATA_DIR=./.terraform       # Директория для данных
export TF_PLUGIN_CACHE_DIR=$HOME/.terraform.d/plugin-cache
```

**Crash Logs:**

```bash
# Terraform создает crash logs автоматически
crash.log                              # В текущей директории
crash.*.log                            # С timestamp

# Содержимое crash.log:
- Stack trace
- Panic message
- Environment information
- Provider versions
```

**Common Error Patterns:**

```
Error Types
├── Syntax Errors - ошибки в HCL коде
├── Validation Errors - нарушение constraints
├── Provider Errors - ошибки API провайдера
├── State Errors - проблемы со state файлом
├── Dependency Errors - циклические зависимости
├── Resource Conflicts - конфликты ресурсов
└── Permission Errors - недостаточно прав
```

**Debugging Commands:**

```bash
# Валидация конфигурации
terraform validate

# Проверка форматирования
terraform fmt -check -recursive

# Граф зависимостей
terraform graph | dot -Tpng > graph.png

# Просмотр плана в деталях
terraform plan -out=tfplan
terraform show -json tfplan | jq

# Просмотр state
terraform state list
terraform state show <resource>

# Проверка провайдеров
terraform providers
terraform version

# Обновление lock файла
terraform providers lock

# Refresh state
terraform refresh  # Deprecated, используй apply -refresh-only
terraform apply -refresh-only
```

**State Recovery Methods:**

```bash
# 1. Backup state
terraform state pull > backup.tfstate

# 2. Restore state
terraform state push backup.tfstate

# 3. Remove corrupted resource
terraform state rm <resource>

# 4. Import existing resource
terraform import <resource> <id>

# 5. Force unlock (если state залочен)
terraform force-unlock <lock-id>

# 6. Migrate state
terraform init -migrate-state
```

### 💻 Задание

Создай debug-friendly проект с примерами распространенных ошибок и их решений:

**1. Структура проекта:**

```bash
mkdir terraform-debugging
cd terraform-debugging
mkdir -p scripts logs examples
```

**2. debug-helper.sh:**

```bash
#!/bin/bash
# scripts/debug-helper.sh

set -e

# Colors
RED='\033[0;31m'
GREEN='\033[0;32m'
YELLOW='\033[1;33m'
NC='\033[0m' # No Color

log_info() {
    echo -e "${GREEN}[INFO]${NC} $1"
}

log_warn() {
    echo -e "${YELLOW}[WARN]${NC} $1"
}

log_error() {
    echo -e "${RED}[ERROR]${NC} $1"
}

# Function: Enable debug logging
enable_debug() {
    log_info "Enabling debug logging..."
    export TF_LOG=DEBUG
    export TF_LOG_PATH="./logs/terraform-$(date +%Y%m%d-%H%M%S).log"
    log_info "Logs will be written to: $TF_LOG_PATH"
}

# Function: Disable debug logging
disable_debug() {
    log_info "Disabling debug logging..."
    unset TF_LOG
    unset TF_LOG_PATH
}

# Function: Validate configuration
validate_config() {
    log_info "Validating Terraform configuration..."
    
    if terraform validate; then
        log_info "✓ Configuration is valid"
        return 0
    else
        log_error "✗ Configuration has errors"
        return 1
    fi
}

# Function: Check formatting
check_format() {
    log_info "Checking code formatting..."
    
    if terraform fmt -check -recursive; then
        log_info "✓ Code is properly formatted"
        return 0
    else
        log_warn "✗ Code needs formatting. Run: terraform fmt -recursive"
        return 1
    fi
}

# Function: Generate dependency graph
generate_graph() {
    log_info "Generating dependency graph..."
    
    if ! command -v dot &> /dev/null; then
        log_error "Graphviz not installed. Install with: brew install graphviz"
        return 1
    fi
    
    terraform graph > logs/graph.dot
    dot -Tpng logs/graph.dot -o logs/graph.png
    log_info "✓ Graph saved to logs/graph.png"
}

# Function: Backup state
backup_state() {
    log_info "Backing up Terraform state..."
    
    BACKUP_DIR="./backups/$(date +%Y%m%d-%H%M%S)"
    mkdir -p "$BACKUP_DIR"
    
    if terraform state pull > "$BACKUP_DIR/terraform.tfstate"; then
        log_info "✓ State backed up to: $BACKUP_DIR"
        return 0
    else
        log_error "✗ Failed to backup state"
        return 1
    fi
}

# Function: Check state health
check_state_health() {
    log_info "Checking state health..."
    
    # Check if state exists
    if ! terraform state list &> /dev/null; then
        log_error "State is corrupted or doesn't exist"
        return 1
    fi
    
    # Count resources
    RESOURCE_COUNT=$(terraform state list | wc -l)
    log_info "State contains $RESOURCE_COUNT resources"
    
    # Check for drift
    log_info "Checking for drift..."
    if terraform plan -detailed-exitcode > /dev/null 2>&1; then
        log_info "✓ No drift detected"
    else
        EXIT_CODE=$?
        if [ $EXIT_CODE -eq 2 ]; then
            log_warn "⚠ Drift detected! Run 'terraform plan' to see changes"
        else
            log_error "✗ Error checking for drift"
        fi
    fi
}

# Function: Analyze crash log
analyze_crash() {
    log_info "Analyzing crash logs..."
    
    CRASH_LOG=$(ls -t crash.*.log 2>/dev/null | head -1)
    
    if [ -z "$CRASH_LOG" ]; then
        log_info "No crash logs found"
        return 0
    fi
    
    log_warn "Found crash log: $CRASH_LOG"
    
    # Extract key information
    echo ""
    echo "=== Panic Message ==="
    grep -A 5 "panic:" "$CRASH_LOG" || echo "No panic message found"
    
    echo ""
    echo "=== Provider Versions ==="
    grep -A 10 "provider.terraform-provider" "$CRASH_LOG" || echo "No provider info found"
}

# Function: Clean up locks
cleanup_locks() {
    log_warn "Cleaning up state locks..."
    
    read -p "This will force-unlock the state. Continue? (y/N) " -n 1 -r
    echo
    
    if [[ ! $REPLY =~ ^[Yy]$ ]]; then
        log_info "Cancelled"
        return 0
    fi
    
    # Get lock ID from error message or state
    log_error "Please provide lock ID manually"
    read -p "Lock ID: " LOCK_ID
    
    if [ -n "$LOCK_ID" ]; then
        terraform force-unlock "$LOCK_ID"
    fi
}

# Function: Performance analysis
analyze_performance() {
    log_info "Analyzing performance..."
    
    # Run plan with timing
    time terraform plan > /dev/null
    
    # Analyze state size
    STATE_SIZE=$(terraform state pull | wc -c)
    log_info "State size: $((STATE_SIZE / 1024))KB"
    
    # Count resources by type
    log_info "Resources by type:"
    terraform state list | cut -d. -f1 | sort | uniq -c | sort -rn
}

# Main menu
show_menu() {
    echo ""
    echo "=== Terraform Debug Helper ==="
    echo "1. Enable debug logging"
    echo "2. Disable debug logging"
    echo "3. Validate configuration"
    echo "4. Check formatting"
    echo "5. Generate dependency graph"
    echo "6. Backup state"
    echo "7. Check state health"
    echo "8. Analyze crash logs"
    echo "9. Clean up locks"
    echo "10. Analyze performance"
    echo "0. Exit"
    echo ""
}

# Main loop
if [ $# -eq 0 ]; then
    while true; do
        show_menu
        read -p "Select option: " choice
        
        case $choice in
            1) enable_debug ;;
            2) disable_debug ;;
            3) validate_config ;;
            4) check_format ;;
            5) generate_graph ;;
            6) backup_state ;;
            7) check_state_health ;;
            8) analyze_crash ;;
            9) cleanup_locks ;;
            10) analyze_performance ;;
            11) exit 0 ;;
            *) log_error "Invalid option" ;;
        esac
        
        echo ""
        read -p "Press Enter to continue..."
    done
else
    # Command line mode
    case $1 in
        debug-on) enable_debug ;;
        debug-off) disable_debug ;;
        validate) validate_config ;;
        format) check_format ;;
        graph) generate_graph ;;
        backup) backup_state ;;
        health) check_state_health ;;
        crash) analyze_crash ;;
        unlock) cleanup_locks ;;
        perf) analyze_performance ;;
        *) log_error "Unknown command: $1" ;;
    esac
fi
```

```bash
chmod +x scripts/debug-helper.sh
```

**3. examples/common-errors.tf:**

```hcl
# examples/common-errors.tf
# Примеры распространенных ошибок и их решений

# ==========================================
# ERROR 1: Syntax Error
# ==========================================

# ❌ WRONG: Missing equals sign
variable "example" {
  type string
  default = "value"
}

# ✅ CORRECT:
variable "example" {
  type    = string
  default = "value"
}

# ❌ WRONG: Invalid interpolation
resource "aws_instance" "bad" {
  ami = "${data.aws_ami.ubuntu.id}"  # Не нужны ${}
}

# ✅ CORRECT:
resource "aws_instance" "good" {
  ami = data.aws_ami.ubuntu.id
}

# ==========================================
# ERROR 2: Circular Dependency
# ==========================================

# ❌ WRONG: Circular reference
resource "aws_security_group" "a" {
  name = "sg-a"
  
  ingress {
    from_port       = 80
    to_port         = 80
    protocol        = "tcp"
    security_groups = [aws_security_group.b.id]
  }
}

resource "aws_security_group" "b" {
  name = "sg-b"
  
  ingress {
    from_port       = 443
    to_port         = 443
    protocol        = "tcp"
    security_groups = [aws_security_group.a.id]
  }
}

# ✅ CORRECT: Use separate rules
resource "aws_security_group" "a" {
  name = "sg-a"
}

resource "aws_security_group" "b" {
  name = "sg-b"
}

resource "aws_security_group_rule" "a_to_b" {
  type                     = "ingress"
  from_port                = 80
  to_port                  = 80
  protocol                 = "tcp"
  security_group_id        = aws_security_group.b.id
  source_security_group_id = aws_security_group.a.id
}

# ==========================================
# ERROR 3: Type Mismatch
# ==========================================

# ❌ WRONG: String instead of number
variable "count" {
  type    = number
  default = "5"  # String!
}

# ✅ CORRECT:
variable "count" {
  type    = number
  default = 5
}

# ❌ WRONG: List instead of string
variable "cidr" {
  type    = string
  default = ["10.0.0.0/16"]  # List!
}

# ✅ CORRECT:
variable "cidr" {
  type    = string
  default = "10.0.0.0/16"
}

# ==========================================
# ERROR 4: Missing Required Argument
# ==========================================

# ❌ WRONG: Missing required ami
resource "aws_instance" "bad" {
  instance_type = "t3.micro"
  # ami is required!
}

# ✅ CORRECT:
resource "aws_instance" "good" {
  ami           = data.aws_ami.ubuntu.id
  instance_type = "t3.micro"
}

# ==========================================
# ERROR 5: Invalid For_Each Value
# ==========================================

# ❌ WRONG: for_each with list
resource "aws_subnet" "bad" {
  for_each = ["10.0.1.0/24", "10.0.2.0/24"]  # List!
  
  vpc_id     = aws_vpc.main.id
  cidr_block = each.value
}

# ✅ CORRECT: Use toset()
resource "aws_subnet" "good" {
  for_each = toset(["10.0.1.0/24", "10.0.2.0/24"])
  
  vpc_id     = aws_vpc.main.id
  cidr_block = each.value
}

# ==========================================
# ERROR 6: Resource Already Exists
# ==========================================

# ❌ WRONG: Trying to create existing resource
# Error: resource already exists
resource "aws_s3_bucket" "existing" {
  bucket = "bucket-that-exists"
}

# ✅ CORRECT: Import first
# terraform import aws_s3_bucket.existing bucket-that-exists

# ==========================================
# ERROR 7: Invalid Count/For_Each
# ==========================================

# ❌ WRONG: Both count and for_each
resource "aws_instance" "bad" {
  count    = 3
  for_each = toset(["a", "b"])  # Can't use both!
  
  ami           = data.aws_ami.ubuntu.id
  instance_type = "t3.micro"
}

# ✅ CORRECT: Use only one
resource "aws_instance" "good" {
  count = 3
  
  ami           = data.aws_ami.ubuntu.id
  instance_type = "t3.micro"
}

# ==========================================
# ERROR 8: Null/Empty Value
# ==========================================

# ❌ WRONG: Null in required field
locals {
  subnet_id = null
}

resource "aws_instance" "bad" {
  ami           = data.aws_ami.ubuntu.id
  instance_type = "t3.micro"
  subnet_id     = local.subnet_id  # Null!
}

# ✅ CORRECT: Use coalesce or validation
locals {
  subnet_id = coalesce(var.subnet_id, data.aws_subnet.default.id)
}

resource "aws_instance" "good" {
  ami           = data.aws_ami.ubuntu.id
  instance_type = "t3.micro"
  subnet_id     = local.subnet_id
}
```

**4. debugging.tf:**

```hcl
# debugging.tf
# Production-ready configuration с debugging features

terraform {
  required_version = ">= 1.6.0"
  
  required_providers {
    aws = {
      source  = "hashicorp/aws"
      version = "~> 5.0"
    }
  }
}

provider "aws" {
  region = var.aws_region
  
  # Default tags помогают в debugging
  default_tags {
    tags = {
      Environment    = var.environment
      ManagedBy      = "Terraform"
      TerraformPath  = path.module
      LastApplied    = timestamp()
      GitCommit      = try(trimspace(file("${path.module}/.git/refs/heads/main")), "unknown")
    }
  }
}

# ==========================================
# Data Sources с Error Handling
# ==========================================

data "aws_caller_identity" "current" {}
data "aws_region" "current" {}

# AMI с детальными проверками
data "aws_ami" "ubuntu" {
  most_recent = true
  owners      = ["099720109477"]
  
  filter {
    name   = "name"
    values = ["ubuntu/images/hvm-ssd/ubuntu-jammy-22.04-amd64-server-*"]
  }
  
  lifecycle {
    postcondition {
      condition     = self.state == "available"
      error_message = "AMI ${self.id} is not available. State: ${self.state}"
    }
    
    postcondition {
      condition     = self.image_id != ""
      error_message = "No AMI found matching the criteria"
    }
  }
}

# VPC с проверкой существования
data "aws_vpc" "selected" {
  count = var.vpc_id != "" ? 1 : 0
  id    = var.vpc_id
  
  lifecycle {
    postcondition {
      condition     = self.id != ""
      error_message = "VPC ${var.vpc_id} not found"
    }
  }
}

data "aws_vpc" "default" {
  count   = var.vpc_id == "" ? 1 : 0
  default = true
}

locals {
  vpc_id = var.vpc_id != "" ? data.aws_vpc.selected[0].id : data.aws_vpc.default[0].id
}

# ==========================================
# Debugging Outputs
# ==========================================

locals {
  debug_info = {
    terraform_version = terraform.version
    provider_version  = provider::aws::version
    workspace         = terraform.workspace
    working_directory = path.module
    root_directory    = path.root
    cwd              = path.cwd
    
    aws_account_id = data.aws_caller_identity.current.account_id
    aws_region     = data.aws_region.current.name
    aws_user_arn   = data.aws_caller_identity.current.arn
    
    vpc_id         = local.vpc_id
    ami_id         = data.aws_ami.ubuntu.id
    ami_name       = data.aws_ami.ubuntu.name
    
    timestamp      = timestamp()
  }
}

# ==========================================
# Example Resource с Debug Features
# ==========================================

resource "aws_instance" "debug_example" {
  count = var.create_instance ? 1 : 0
  
  ami           = data.aws_ami.ubuntu.id
  instance_type = var.instance_type
  
  # User data для debugging
  user_data = base64encode(templatefile("${path.module}/scripts/debug-userdata.sh", {
    environment   = var.environment
    instance_name = "${var.project_name}-debug"
    debug_level   = var.debug_level
  }))
  
  # Tags для debugging
  tags = {
    Name          = "${var.project_name}-debug"
    CreatedBy     = data.aws_caller_identity.current.arn
    CreatedAt     = timestamp()
    DebugEnabled  = var.debug_level > 0 ? "true" : "false"
  }
  
  # Lifecycle с проверками
  lifecycle {
    precondition {
      condition     = contains(["t3.micro", "t3.small", "t3.medium"], var.instance_type)
      error_message = "Instance type ${var.instance_type} not allowed. Use t3.micro, t3.small, or t3.medium"
    }
    
    postcondition {
      condition     = self.instance_state == "running"
      error_message = "Instance failed to start. State: ${self.instance_state}"
    }
    
    postcondition {
      condition     = self.ami == data.aws_ami.ubuntu.id
      error_message = "Instance created with wrong AMI. Expected: ${data.aws_ami.ubuntu.id}, Got: ${self.ami}"
    }
  }
}

# ==========================================
# Debugging Helpers
# ==========================================

# Null resource для логирования
resource "null_resource" "debug_logger" {
  count = var.debug_level > 0 ? 1 : 0
  
  triggers = {
    always_run = timestamp()
  }
  
  provisioner "local-exec" {
    command = <<-EOT
      echo "=== Debug Information ===" >> ${path.module}/logs/debug.log
      echo "Timestamp: $(date)" >> ${path.module}/logs/debug.log
      echo "Workspace: ${terraform.workspace}" >> ${path.module}/logs/debug.log
      echo "Account: ${data.aws_caller_identity.current.account_id}" >> ${path.module}/logs/debug.log
      echo "Region: ${data.aws_region.current.name}" >> ${path.module}/logs/debug.log
      echo "========================" >> ${path.module}/logs/debug.log
    EOT
  }
}

# ==========================================
# Outputs для Debugging
# ==========================================

output "debug_info" {
  description = "Debug information"
  value       = var.debug_level > 0 ? local.debug_info : null
}

output "resource_ids" {
  description = "Created resource IDs"
  value = {
    instance_id = try(aws_instance.debug_example[0].id, null)
    vpc_id      = local.vpc_id
  }
}

output "validation_status" {
  description = "Validation status"
  value = {
    ami_valid  = data.aws_ami.ubuntu.state == "available"
    vpc_valid  = local.vpc_id != ""
    region     = data.aws_region.current.name
  }
}
```

**5. variables.tf:**

```hcl
variable "aws_region" {
  description = "AWS region"
  type        = string
  default     = "us-east-1"
}

variable "project_name" {
  description = "Project name"
  type        = string
  default     = "debug-demo"
}

variable "environment" {
  description = "Environment name"
  type        = string
  default     = "dev"
}

variable "vpc_id" {
  description = "VPC ID (leave empty for default VPC)"
  type        = string
  default     = ""
}

variable "instance_type" {
  description = "EC2 instance type"
  type        = string
  default     = "t3.micro"
}

variable "create_instance" {
  description = "Whether to create instance"
  type        = bool
  default     = false
}

variable "debug_level" {
  description = "Debug level (0-3)"
  type        = number
  default     = 0
  
  validation {
    condition     = var.debug_level >= 0 && var.debug_level <= 3
    error_message = "Debug level must be between 0 and 3"
  }
}
```

**6. scripts/debug-userdata.sh:**

```bash
#!/bin/bash
# Debug-enabled user data script

set -e

# Debug level: ${debug_level}
if [ "${debug_level}" -gt "0" ]; then
    set -x  # Enable debug output
fi

# Logging
LOGFILE="/var/log/userdata-debug.log"
exec > >(tee -a $LOGFILE)
exec 2>&1

echo "=== User Data Script Started ==="
echo "Environment: ${environment}"
echo "Instance: ${instance_name}"
echo "Debug Level: ${debug_level}"
echo "Time: $(date)"

# Function: Log with level
log() {
    local level=$1
    shift
    echo "[$(date '+%Y-%m-%d %H:%M:%S')] [$level] $*" | tee -a $LOGFILE
}

log "INFO" "Installing packages..."

export DEBIAN_FRONTEND=noninteractive
apt-get update
apt-get install -y \
    nginx \
    curl \
    jq \
    htop \
    net-tools

log "INFO" "Creating debug page..."

cat > /var/www/html/index.html <<EOF
<!DOCTYPE html>
<html>
<head>
    <title>Debug Instance</title>
    <style>
        body { font-family: monospace; margin: 20px; background: #1e1e1e; color: #d4d4d4; }
        .container { background: #252526; padding: 20px; border-radius: 5px; }
        h1 { color: #4ec9b0; }
        .info { margin: 10px 0; }
        .info strong { color: #569cd6; }
    </style>
</head>
<body>
    <div class="container">
        <h1>🔍 Debug Instance</h1>
        <div class="info"><strong>Environment:</strong> ${environment}</div>
        <div class="info"><strong>Instance:</strong> ${instance_name}</div>
        <div class="info"><strong>Debug Level:</strong> ${debug_level}</div>
        <div class="info"><strong>Instance ID:</strong> $(ec2-metadata --instance-id | cut -d ' ' -f 2)</div>
        <div class="info"><strong>Private IP:</strong> $(hostname -I)</div>
        <div class="info"><strong>Started:</strong> $(date)</div>
    </div>
</body>
</html>
EOF

log "INFO" "Starting nginx..."
systemctl enable nginx
systemctl start nginx

log "INFO" "User data script completed successfully"
```

**7. Makefile для debugging:**

```makefile
# Makefile

.PHONY: help debug-on debug-off validate graph backup test-errors clean

help:
	@echo "Debugging Commands:"
	@echo "  debug-on     - Enable debug logging"
	@echo "  debug-off    - Disable debug logging"
	@echo "  validate     - Validate configuration"
	@echo "  graph        - Generate dependency graph"
	@echo "  backup       - Backup state"
	@echo "  test-errors  - Test error handling"
	@echo "  clean        - Clean logs and backups"

debug-on:
	@echo "Enabling debug logging..."
	@export TF_LOG=DEBUG TF_LOG_PATH=./logs/terraform-$$(date +%Y%m%d-%H%M%S).log
	@echo "Debug logging enabled. Logs: $$TF_LOG_PATH"

debug-off:
	@echo "Disabling debug logging..."
	@unset TF_LOG TF_LOG_PATH
	@echo "Debug logging disabled"

validate:
	@./scripts/debug-helper.sh validate

graph:
	@./scripts/debug-helper.sh graph

backup:
	@./scripts/debug-helper.sh backup

test-errors:
	@echo "Testing common errors..."
	@terraform validate examples/common-errors.tf || true

clean:
	@echo "Cleaning logs and backups..."
	@rm -rf logs/* backups/* crash.*.log
	@echo "Clean complete"

# Debugging specific issues
debug-state:
	@echo "=== State Information ==="
	@terraform state list
	@echo ""
	@echo "=== State Size ==="
	@terraform state pull | wc -c | awk '{print $$1/1024 "KB"}'

debug-providers:
	@echo "=== Provider Versions ==="
	@terraform providers
	@echo ""
	@echo "=== Provider Schema ==="
	@terraform providers schema -json | jq '.provider_schemas'

debug-plan:
	@echo "=== Running plan with debug ==="
	@TF_LOG=DEBUG terraform plan 2>&1 | tee logs/plan-debug.log

debug-performance:
	@echo "=== Performance Analysis ==="
	@time terraform plan > /dev/null
	@echo ""
	@./scripts/debug-helper.sh perf
```

**8. .gitignore:**

```
# Terraform
.terraform/
*.tfstate
*.tfstate.*
*.tfplan
crash.log
crash.*.log
override.tf
override.tf.json
*_override.tf
*_override.tf.json

# Logs
logs/*.log
*.log

# Backups
backups/

# OS
.DS_Store
Thumbs.db

# IDE
.idea/
.vscode/
*.swp
*.swo
```

**9. README.md:**

````markdown
# Terraform Debugging Guide

## Quick Start

```bash
# Enable debug logging
make debug-on

# Run terraform commands
terraform plan

# Disable debug logging
make debug-off
````

## Debug Helper Script

```bash
# Interactive mode
./scripts/debug-helper.sh

# Command line mode
./scripts/debug-helper.sh validate
./scripts/debug-helper.sh health
./scripts/debug-helper.sh graph
```

## Common Issues

### Issue 1: State Lock

```bash
# Check lock status
terraform state list

# Force unlock (use with caution!)
terraform force-unlock <LOCK_ID>
```

### Issue 2: Resource Drift

```bash
# Check for drift
terraform plan -detailed-exitcode

# Refresh state
terraform apply -refresh-only
```

### Issue 3: Provider Errors

```bash
# Re-initialize providers
rm -rf .terraform
terraform init -upgrade
```

## Debug Levels

```bash
export TF_LOG=TRACE  # Most verbose
export TF_LOG=DEBUG  # Detailed debugging
export TF_LOG=INFO   # General information
export TF_LOG=WARN   # Warnings only
export TF_LOG=ERROR  # Errors only
```

## Troubleshooting

1. **Validation Errors**: Run `terraform validate`
2. **Syntax Errors**: Run `terraform fmt -check`
3. **State Issues**: Run `./scripts/debug-helper.sh health`
4. **Performance**: Run `./scripts/debug-helper.sh perf`



**10. Запуск и тестирование:**

```bash
# Создать необходимые директории
mkdir -p logs backups

# Инициализация
terraform init

# Тест 1: Валидация
make validate

# Тест 2: Проверка форматирования
terraform fmt -check -recursive

# Тест 3: Генерация графа зависимостей
make graph open logs/graph.png

# Тест 4: Debug режим
export TF_LOG=DEBUG export TF_LOG_PATH=./logs/terraform-debug.log terraform plan

# Проверить логи
cat logs/terraform-debug.log | grep -i error

# Тест 5: Тестирование распространенных ошибок
terraform validate examples/common-errors.tf

# Тест 6: Backup state
make backup

# Тест 7: Health check
./scripts/debug-helper.sh health

# Тест 8: Performance analysis
./scripts/debug-helper.sh perf

```

### 🚀 Бонус (продвинутое)

**1. Advanced Log Analysis Script:**

```bash
#!/bin/bash
# scripts/analyze-logs.sh

LOG_FILE=${1:-"logs/terraform-debug.log"}

if [ ! -f "$LOG_FILE" ]; then
    echo "Log file not found: $LOG_FILE"
    exit 1
fi

echo "=== Log Analysis for $LOG_FILE ==="
echo ""

# Statistics
echo "Total lines: $(wc -l < "$LOG_FILE")"
echo "Errors: $(grep -c "ERROR" "$LOG_FILE" || echo 0)"
echo "Warnings: $(grep -c "WARN" "$LOG_FILE" || echo 0)"
echo ""

# Top errors
echo "=== Top Errors ==="
grep "ERROR" "$LOG_FILE" | cut -d':' -f3- | sort | uniq -c | sort -rn | head -10
echo ""

# Provider calls
echo "=== Provider API Calls ==="
grep "provider.terraform-provider" "$LOG_FILE" | \
    grep -oP '(?<=provider.terraform-provider-)[^:]+' | \
    sort | uniq -c | sort -rn
echo ""

# Slowest operations
echo "=== Slowest Operations ==="
grep "aws_.*: Creation complete" "$LOG_FILE" | \
    grep -oP '\[\d+[ms]+\]' | \
    sort -rn | head -10
echo ""

# Resource changes
echo "=== Resource Changes ==="
echo "Created: $(grep -c "Creation complete" "$LOG_FILE" || echo 0)"
echo "Updated: $(grep -c "Modifications complete" "$LOG_FILE" || echo 0)"
echo "Destroyed: $(grep -c "Destruction complete" "$LOG_FILE" || echo 0)"
````

**2. State Recovery Script:**

```bash
#!/bin/bash
# scripts/recover-state.sh

set -e

BACKUP_DIR="./backups"

echo "=== State Recovery Utility ==="

# List available backups
echo "Available backups:"
ls -lht "$BACKUP_DIR" | head -10

echo ""
read -p "Enter backup timestamp (YYYYMMDD-HHMMSS) or 'latest': " TIMESTAMP

if [ "$TIMESTAMP" = "latest" ]; then
    BACKUP_PATH=$(ls -t "$BACKUP_DIR"/*/terraform.tfstate | head -1)
else
    BACKUP_PATH="$BACKUP_DIR/$TIMESTAMP/terraform.tfstate"
fi

if [ ! -f "$BACKUP_PATH" ]; then
    echo "Backup not found: $BACKUP_PATH"
    exit 1
fi

echo "Selected backup: $BACKUP_PATH"

# Verify backup
echo "Verifying backup..."
if ! cat "$BACKUP_PATH" | jq empty 2>/dev/null; then
    echo "❌ Backup is corrupted (invalid JSON)"
    exit 1
fi

echo "✓ Backup is valid"

# Show backup info
echo ""
echo "=== Backup Information ==="
echo "Terraform version: $(cat "$BACKUP_PATH" | jq -r '.terraform_version')"
echo "Serial: $(cat "$BACKUP_PATH" | jq -r '.serial')"
echo "Resources: $(cat "$BACKUP_PATH" | jq '.resources | length')"

echo ""
read -p "Proceed with recovery? (y/N) " -n 1 -r
echo

if [[ ! $REPLY =~ ^[Yy]$ ]]; then
    echo "Cancelled"
    exit 0
fi

# Create current state backup
echo "Creating backup of current state..."
CURRENT_BACKUP="$BACKUP_DIR/pre-recovery-$(date +%Y%m%d-%H%M%S)"
mkdir -p "$CURRENT_BACKUP"
terraform state pull > "$CURRENT_BACKUP/terraform.tfstate"

# Push recovered state
echo "Recovering state..."
terraform state push "$BACKUP_PATH"

echo "✓ State recovered successfully"
echo "Current state backed up to: $CURRENT_BACKUP"
```

**3. Performance Profiler:**

```bash
#!/bin/bash
# scripts/profile-terraform.sh

set -e

echo "=== Terraform Performance Profiler ==="

# Profile initialization
echo "Profiling terraform init..."
time terraform init -upgrade 2>&1 | tee logs/profile-init.log

# Profile plan
echo "Profiling terraform plan..."
TF_LOG=TRACE TF_LOG_PATH=logs/profile-plan-trace.log \
    time terraform plan -out=tfplan 2>&1 | tee logs/profile-plan.log

# Analyze plan timing
echo ""
echo "=== Plan Analysis ==="

# Resource timings
grep -oP 'aws_\S+.*\[\d+[ms]+\]' logs/profile-plan-trace.log | \
    awk '{print $NF, $0}' | \
    sort -rn | \
    head -20

# State size
echo ""
echo "=== State Size ==="
STATE_SIZE=$(terraform state pull | wc -c)
echo "Current: $((STATE_SIZE / 1024))KB"

# Resource count by type
echo ""
echo "=== Resources by Type ==="
terraform state list | \
    cut -d. -f1 | \
    sort | \
    uniq -c | \
    sort -rn

# Provider version info
echo ""
echo "=== Provider Versions ==="
terraform providers | grep provider

# Lock file analysis
echo ""
echo "=== Lock File ==="
if [ -f .terraform.lock.hcl ]; then
    echo "Size: $(wc -c < .terraform.lock.hcl) bytes"
    echo "Providers:"
    grep "provider" .terraform.lock.hcl | wc -l
fi
```

**4. Automated Troubleshooting:**

```bash
#!/bin/bash
# scripts/auto-troubleshoot.sh

set -e

RED='\033[0;31m'
GREEN='\033[0;32m'
YELLOW='\033[1;33m'
NC='\033[0m'

run_check() {
    local name=$1
    local command=$2
    
    echo -n "Checking $name... "
    
    if eval "$command" > /dev/null 2>&1; then
        echo -e "${GREEN}✓${NC}"
        return 0
    else
        echo -e "${RED}✗${NC}"
        return 1
    fi
}

echo "=== Automated Troubleshooting ==="
echo ""

# Check 1: Terraform version
run_check "Terraform version" "terraform version"

# Check 2: Provider plugins
run_check "Provider plugins" "test -d .terraform/providers"

# Check 3: Lock file
run_check "Lock file" "test -f .terraform.lock.hcl"

# Check 4: State file
run_check "State file" "terraform state list"

# Check 5: AWS credentials
run_check "AWS credentials" "aws sts get-caller-identity"

# Check 6: Syntax validation
run_check "Syntax validation" "terraform validate"

# Check 7: Formatting
run_check "Code formatting" "terraform fmt -check -recursive"

# Check 8: State health
echo -n "Checking state health... "
DRIFT_EXIT_CODE=0
terraform plan -detailed-exitcode > /dev/null 2>&1 || DRIFT_EXIT_CODE=$?

if [ $DRIFT_EXIT_CODE -eq 0 ]; then
    echo -e "${GREEN}✓ No drift${NC}"
elif [ $DRIFT_EXIT_CODE -eq 2 ]; then
    echo -e "${YELLOW}⚠ Drift detected${NC}"
else
    echo -e "${RED}✗ Error${NC}"
fi

# Check 9: Lock status
echo -n "Checking for locks... "
if terraform state list > /dev/null 2>&1; then
    echo -e "${GREEN}✓ No locks${NC}"
else
    echo -e "${RED}✗ State locked${NC}"
fi

# Summary
echo ""
echo "=== Troubleshooting Complete ==="

# Suggestions
echo ""
echo "Suggestions:"

if [ ! -d .terraform/providers ]; then
    echo "  - Run: terraform init"
fi

if ! terraform fmt -check -recursive > /dev/null 2>&1; then
    echo "  - Run: terraform fmt -recursive"
fi

if [ $DRIFT_EXIT_CODE -eq 2 ]; then
    echo "  - Run: terraform plan (to see drift)"
fi
```

**5. CI/CD Debug Configuration:**

```yaml
# .github/workflows/terraform-debug.yml
name: Terraform Debug

on:
  pull_request:
    branches: [main]
  workflow_dispatch:

env:
  TF_LOG: DEBUG
  TF_LOG_PATH: ./logs/terraform-ci.log

jobs:
  debug:
    runs-on: ubuntu-latest
    
    steps:
      - uses: actions/checkout@v4
      
      - name: Setup Terraform
        uses: hashicorp/setup-terraform@v3
        with:
          terraform_version: 1.6.0
      
      - name: Create logs directory
        run: mkdir -p logs
      
      - name: Terraform Init
        run: |
          terraform init
          echo "::group::Init Logs"
          cat logs/terraform-ci.log || true
          echo "::endgroup::"
      
      - name: Terraform Validate
        run: terraform validate
      
      - name: Terraform Format Check
        run: terraform fmt -check -recursive
      
      - name: Terraform Plan
        run: |
          terraform plan -no-color | tee logs/plan.log
      
      - name: Analyze Logs
        if: always()
        run: |
          echo "=== Log Analysis ==="
          echo "Total errors: $(grep -c ERROR logs/terraform-ci.log || echo 0)"
          echo "Total warnings: $(grep -c WARN logs/terraform-ci.log || echo 0)"
          
          if grep -q ERROR logs/terraform-ci.log; then
            echo "::error::Errors found in logs"
            echo "::group::Error Details"
            grep ERROR logs/terraform-ci.log
            echo "::endgroup::"
          fi
      
      - name: Upload Logs
        if: always()
        uses: actions/upload-artifact@v4
        with:
          name: terraform-logs
          path: logs/
          retention-days: 7
```

---

## Заключение Модуля 14

### 📚 Ключевые takeaways

**1. Debug Logging:**

```bash
# Базовое использование
export TF_LOG=DEBUG
export TF_LOG_PATH=./terraform.log
terraform plan

# Раздельное логирование
export TF_LOG_CORE=TRACE
export TF_LOG_PROVIDER=DEBUG
```

**2. Распространенные ошибки и решения:**

|Ошибка|Причина|Решение|
|---|---|---|
|Syntax Error|Неправильный HCL|`terraform validate`|
|Circular Dependency|A → B → A|Разделить на rules|
|Type Mismatch|Неверный тип данных|Проверить типы переменных|
|State Lock|Незавершенная операция|`terraform force-unlock`|
|Resource Exists|Дубликат ресурса|`terraform import`|

**3. State Recovery Workflow:**

```bash
# 1. Backup current state
terraform state pull > backup.tfstate

# 2. Fix issues

# 3. Restore if needed
terraform state push backup.tfstate
```

**4. Performance Optimization:**

- ✅ Используй `-parallelism` для контроля параллельности
- ✅ Минимизируй data sources
- ✅ Используй provider cache
- ✅ Разделяй большие проекты на модули
- ✅ Используй partial backend configuration

**5. Best Practices:**

```bash
# ✅ DO: Enable logging для debugging
export TF_LOG=DEBUG terraform plan

# ✅ DO: Регулярные backups state
./scripts/debug-helper.sh backup

# ✅ DO: Validate перед apply
terraform validate && terraform plan

# ✅ DO: Use version constraints
terraform {
  required_version = ">= 1.6.0"
}

# ❌ DON'T: Commit logs в Git
# ❌ DON'T: Игнорировать warnings
# ❌ DON'T: Force-unlock без понимания причины
```

Ты освоил:

- ✅ Debugging с TF_LOG
- ✅ Анализ и решение распространенных ошибок
- ✅ State recovery процедуры
- ✅ Performance профилирование и оптимизация
- ✅ Автоматизированный troubleshooting
- ✅ CI/CD интеграция для debugging

**Финальные рекомендации:**

1. **Всегда используй version control** для конфигурации
2. **Регулярно делай backup state** перед критичными изменениями
3. **Включай debug logging** при проблемах
4. **Автоматизируй проверки** через CI/CD
5. **Документируй решения** распространенных проблем

---

**Следующие шаги:**

1. Практикуйся на реальных проектах
2. Изучи Terraform Cloud/Enterprise
3. Внедри CI/CD для Terraform
4. Изучи Testing (Terratest, kitchen-terraform)
5. Policy as Code (Sentinel, OPA)

---
# Модуль 15: Advanced AWS - Serverless (30 минут)

## 🎯 Напоминалка

**AWS Serverless Stack:**

```
Serverless Architecture
├── Compute
│   ├── Lambda - функции без серверов
│   └── Fargate - контейнеры без серверов
├── API
│   ├── API Gateway - HTTP/REST/WebSocket APIs
│   └── AppSync - GraphQL APIs
├── Orchestration
│   ├── Step Functions - workflow orchestration
│   └── EventBridge - event bus
├── Storage
│   ├── S3 - объектное хранилище
│   └── DynamoDB - NoSQL база данных
└── Integration
    ├── SQS - очереди сообщений
    ├── SNS - pub/sub notifications
    └── Kinesis - потоковая обработка
```

**Lambda Best Practices:**

```hcl
# Lambda Function структура
resource "aws_lambda_function" "example" {
  function_name = "my-function"
  role          = aws_iam_role.lambda.arn
  
  # Runtime
  runtime = "python3.11"
  handler = "index.handler"
  
  # Code
  filename         = "lambda.zip"
  source_code_hash = filebase64sha256("lambda.zip")
  
  # Configuration
  memory_size = 256
  timeout     = 30
  
  # Environment
  environment {
    variables = {
      TABLE_NAME = aws_dynamodb_table.main.name
    }
  }
  
  # VPC (опционально)
  vpc_config {
    subnet_ids         = var.subnet_ids
    security_group_ids = [aws_security_group.lambda.id]
  }
  
  # Tracing
  tracing_config {
    mode = "Active"  # X-Ray
  }
  
  # Logging
  logging_config {
    log_format = "JSON"
    log_group  = aws_cloudwatch_log_group.lambda.name
  }
}
```

**API Gateway Patterns:**

```
API Gateway Types
├── REST API - RESTful APIs
│   ├── Edge-optimized
│   ├── Regional
│   └── Private
├── HTTP API - простые HTTP APIs (дешевле)
│   └── Меньше функций, но быстрее
└── WebSocket API - real-time двусторонняя связь
```

**Event-Driven Architecture:**

```
EventBridge Pattern
├── Event Sources
│   ├── AWS Services (S3, DynamoDB, etc)
│   ├── Custom Applications
│   └── SaaS Partners
├── Event Bus
│   ├── Default bus
│   ├── Custom buses
│   └── Partner buses
└── Targets
    ├── Lambda functions
    ├── Step Functions
    ├── SNS/SQS
    └── Third-party services
```

---

## Часть 1: Lambda + API Gateway - CRUD API

### 💻 Задание: Создание Serverless REST API

**1. Структура проекта:**

```bash
mkdir terraform-serverless
cd terraform-serverless
mkdir -p lambda/functions/{create,read,update,delete,list} scripts
```

**2. provider.tf:**

```hcl
terraform {
  required_version = ">= 1.6.0"
  
  required_providers {
    aws = {
      source  = "hashicorp/aws"
      version = "~> 5.0"
    }
    archive = {
      source  = "hashicorp/archive"
      version = "~> 2.4"
    }
  }
}

provider "aws" {
  region = var.aws_region
  
  default_tags {
    tags = local.common_tags
  }
}
```

**3. variables.tf:**

```hcl
variable "aws_region" {
  description = "AWS region"
  type        = string
  default     = "us-east-1"
}

variable "project_name" {
  description = "Project name"
  type        = string
  default     = "serverless-api"
  
  validation {
    condition     = can(regex("^[a-z][a-z0-9-]*$", var.project_name))
    error_message = "Project name must be lowercase alphanumeric with hyphens."
  }
}

variable "environment" {
  description = "Environment name"
  type        = string
  default     = "dev"
  
  validation {
    condition     = contains(["dev", "staging", "prod"], var.environment)
    error_message = "Must be dev, staging, or prod."
  }
}

variable "lambda_runtime" {
  description = "Lambda runtime"
  type        = string
  default     = "python3.11"
}

variable "lambda_memory_size" {
  description = "Lambda memory size in MB"
  type        = number
  default     = 256
  
  validation {
    condition     = var.lambda_memory_size >= 128 && var.lambda_memory_size <= 10240
    error_message = "Memory size must be between 128 and 10240 MB."
  }
}

variable "lambda_timeout" {
  description = "Lambda timeout in seconds"
  type        = number
  default     = 30
  
  validation {
    condition     = var.lambda_timeout >= 1 && var.lambda_timeout <= 900
    error_message = "Timeout must be between 1 and 900 seconds."
  }
}

variable "enable_xray" {
  description = "Enable AWS X-Ray tracing"
  type        = bool
  default     = true
}

variable "log_retention_days" {
  description = "CloudWatch Logs retention in days"
  type        = number
  default     = 7
}

variable "api_throttle_burst_limit" {
  description = "API Gateway burst limit"
  type        = number
  default     = 5000
}

variable "api_throttle_rate_limit" {
  description = "API Gateway rate limit"
  type        = number
  default     = 10000
}

variable "cors_allowed_origins" {
  description = "CORS allowed origins"
  type        = list(string)
  default     = ["*"]
}
```

**4. locals.tf:**

```hcl
locals {
  name_prefix = "${var.project_name}-${var.environment}"
  
  common_tags = {
    Project     = var.project_name
    Environment = var.environment
    ManagedBy   = "Terraform"
    Service     = "Serverless"
  }
  
  # Lambda functions configuration
  lambda_functions = {
    create = {
      handler     = "create.handler"
      description = "Create item"
      memory      = var.lambda_memory_size
      timeout     = var.lambda_timeout
    }
    read = {
      handler     = "read.handler"
      description = "Read item"
      memory      = 128
      timeout     = 10
    }
    update = {
      handler     = "update.handler"
      description = "Update item"
      memory      = var.lambda_memory_size
      timeout     = var.lambda_timeout
    }
    delete = {
      handler     = "delete.handler"
      description = "Delete item"
      memory      = 128
      timeout     = 10
    }
    list = {
      handler     = "list.handler"
      description = "List items"
      memory      = 128
      timeout     = 10
    }
  }
  
  # Environment variables для Lambda
  lambda_environment = {
    TABLE_NAME     = aws_dynamodb_table.items.name
    ENVIRONMENT    = var.environment
    LOG_LEVEL      = var.environment == "prod" ? "INFO" : "DEBUG"
    POWERTOOLS_SERVICE_NAME = var.project_name
  }
}
```

**5. dynamodb.tf:**

```hcl
# DynamoDB table для хранения данных
resource "aws_dynamodb_table" "items" {
  name           = "${local.name_prefix}-items"
  billing_mode   = "PAY_PER_REQUEST"  # On-demand для serverless
  hash_key       = "id"
  
  attribute {
    name = "id"
    type = "S"
  }
  
  # Global Secondary Index для query по дате
  attribute {
    name = "created_at"
    type = "S"
  }
  
  global_secondary_index {
    name            = "CreatedAtIndex"
    hash_key        = "created_at"
    projection_type = "ALL"
  }
  
  # Time To Live
  ttl {
    attribute_name = "ttl"
    enabled        = true
  }
  
  # Point-in-time recovery для production
  point_in_time_recovery {
    enabled = var.environment == "prod"
  }
  
  # Server-side encryption
  server_side_encryption {
    enabled = true
  }
  
  tags = merge(local.common_tags, {
    Name = "${local.name_prefix}-items-table"
  })
}

# DynamoDB Auto Scaling (опционально)
resource "aws_appautoscaling_target" "dynamodb_read" {
  count = var.environment == "prod" ? 1 : 0
  
  max_capacity       = 100
  min_capacity       = 5
  resource_id        = "table/${aws_dynamodb_table.items.name}"
  scalable_dimension = "dynamodb:table:ReadCapacityUnits"
  service_namespace  = "dynamodb"
}

resource "aws_appautoscaling_policy" "dynamodb_read" {
  count = var.environment == "prod" ? 1 : 0
  
  name               = "${local.name_prefix}-dynamodb-read-policy"
  policy_type        = "TargetTrackingScaling"
  resource_id        = aws_appautoscaling_target.dynamodb_read[0].resource_id
  scalable_dimension = aws_appautoscaling_target.dynamodb_read[0].scalable_dimension
  service_namespace  = aws_appautoscaling_target.dynamodb_read[0].service_namespace
  
  target_tracking_scaling_policy_configuration {
    predefined_metric_specification {
      predefined_metric_type = "DynamoDBReadCapacityUtilization"
    }
    target_value = 70.0
  }
}
```

**6. lambda/functions/create/create.py:**

```python
import json
import boto3
import uuid
import os
from datetime import datetime
from decimal import Decimal

# Initialize DynamoDB
dynamodb = boto3.resource('dynamodb')
table_name = os.environ['TABLE_NAME']
table = dynamodb.Table(table_name)

def handler(event, context):
    """
    Lambda handler для создания item
    """
    print(f"Event: {json.dumps(event)}")
    
    try:
        # Parse request body
        body = json.loads(event.get('body', '{}'))
        
        # Validate required fields
        if 'name' not in body:
            return {
                'statusCode': 400,
                'headers': get_cors_headers(),
                'body': json.dumps({
                    'error': 'Missing required field: name'
                })
            }
        
        # Create item
        item_id = str(uuid.uuid4())
        timestamp = datetime.utcnow().isoformat()
        
        item = {
            'id': item_id,
            'name': body['name'],
            'description': body.get('description', ''),
            'created_at': timestamp,
            'updated_at': timestamp,
            'status': 'active'
        }
        
        # Add optional fields
        if 'metadata' in body:
            item['metadata'] = body['metadata']
        
        # Save to DynamoDB
        table.put_item(Item=item)
        
        return {
            'statusCode': 201,
            'headers': get_cors_headers(),
            'body': json.dumps({
                'message': 'Item created successfully',
                'item': item
            }, default=str)
        }
        
    except Exception as e:
        print(f"Error: {str(e)}")
        return {
            'statusCode': 500,
            'headers': get_cors_headers(),
            'body': json.dumps({
                'error': 'Internal server error',
                'message': str(e)
            })
        }

def get_cors_headers():
    """CORS headers"""
    return {
        'Content-Type': 'application/json',
        'Access-Control-Allow-Origin': '*',
        'Access-Control-Allow-Headers': 'Content-Type,X-Amz-Date,Authorization,X-Api-Key',
        'Access-Control-Allow-Methods': 'GET,POST,PUT,DELETE,OPTIONS'
    }
```

**7. lambda/functions/read/read.py:**

```python
import json
import boto3
import os
from decimal import Decimal

dynamodb = boto3.resource('dynamodb')
table_name = os.environ['TABLE_NAME']
table = dynamodb.Table(table_name)

def handler(event, context):
    """
    Lambda handler для чтения item
    """
    print(f"Event: {json.dumps(event)}")
    
    try:
        # Get item ID from path parameters
        item_id = event.get('pathParameters', {}).get('id')
        
        if not item_id:
            return {
                'statusCode': 400,
                'headers': get_cors_headers(),
                'body': json.dumps({
                    'error': 'Missing item ID'
                })
            }
        
        # Get item from DynamoDB
        response = table.get_item(Key={'id': item_id})
        
        if 'Item' not in response:
            return {
                'statusCode': 404,
                'headers': get_cors_headers(),
                'body': json.dumps({
                    'error': 'Item not found'
                })
            }
        
        item = response['Item']
        
        return {
            'statusCode': 200,
            'headers': get_cors_headers(),
            'body': json.dumps({
                'item': item
            }, default=decimal_default)
        }
        
    except Exception as e:
        print(f"Error: {str(e)}")
        return {
            'statusCode': 500,
            'headers': get_cors_headers(),
            'body': json.dumps({
                'error': 'Internal server error',
                'message': str(e)
            })
        }

def decimal_default(obj):
    """JSON serializer для Decimal"""
    if isinstance(obj, Decimal):
        return float(obj)
    raise TypeError

def get_cors_headers():
    return {
        'Content-Type': 'application/json',
        'Access-Control-Allow-Origin': '*',
        'Access-Control-Allow-Headers': 'Content-Type,X-Amz-Date,Authorization,X-Api-Key',
        'Access-Control-Allow-Methods': 'GET,POST,PUT,DELETE,OPTIONS'
    }
```

**8. lambda/functions/list/list.py:**

```python
import json
import boto3
import os
from decimal import Decimal

dynamodb = boto3.resource('dynamodb')
table_name = os.environ['TABLE_NAME']
table = dynamodb.Table(table_name)

def handler(event, context):
    """
    Lambda handler для получения списка items
    """
    print(f"Event: {json.dumps(event)}")
    
    try:
        # Get query parameters
        query_params = event.get('queryStringParameters') or {}
        limit = int(query_params.get('limit', 100))
        
        # Scan table (для production используй Query с GSI)
        scan_kwargs = {
            'Limit': min(limit, 100)
        }
        
        # Add pagination
        if 'lastKey' in query_params:
            scan_kwargs['ExclusiveStartKey'] = json.loads(query_params['lastKey'])
        
        response = table.scan(**scan_kwargs)
        
        items = response.get('Items', [])
        last_key = response.get('LastEvaluatedKey')
        
        return {
            'statusCode': 200,
            'headers': get_cors_headers(),
            'body': json.dumps({
                'items': items,
                'count': len(items),
                'lastKey': last_key
            }, default=decimal_default)
        }
        
    except Exception as e:
        print(f"Error: {str(e)}")
        return {
            'statusCode': 500,
            'headers': get_cors_headers(),
            'body': json.dumps({
                'error': 'Internal server error',
                'message': str(e)
            })
        }

def decimal_default(obj):
    if isinstance(obj, Decimal):
        return float(obj)
    raise TypeError

def get_cors_headers():
    return {
        'Content-Type': 'application/json',
        'Access-Control-Allow-Origin': '*',
        'Access-Control-Allow-Headers': 'Content-Type,X-Amz-Date,Authorization,X-Api-Key',
        'Access-Control-Allow-Methods': 'GET,POST,PUT,DELETE,OPTIONS'
    }
```

**9. lambda/functions/update/update.py:**

```python
import json
import boto3
import os
from datetime import datetime
from decimal import Decimal

dynamodb = boto3.resource('dynamodb')
table_name = os.environ['TABLE_NAME']
table = dynamodb.Table(table_name)

def handler(event, context):
    """
    Lambda handler для обновления item
    """
    print(f"Event: {json.dumps(event)}")
    
    try:
        # Get item ID
        item_id = event.get('pathParameters', {}).get('id')
        if not item_id:
            return {
                'statusCode': 400,
                'headers': get_cors_headers(),
                'body': json.dumps({'error': 'Missing item ID'})
            }
        
        # Parse request body
        body = json.loads(event.get('body', '{}'))
        
        # Build update expression
        update_expr = "SET updated_at = :updated_at"
        expr_values = {':updated_at': datetime.utcnow().isoformat()}
        expr_names = {}
        
        # Add fields to update
        if 'name' in body:
            update_expr += ", #n = :name"
            expr_values[':name'] = body['name']
            expr_names['#n'] = 'name'
        
        if 'description' in body:
            update_expr += ", description = :description"
            expr_values[':description'] = body['description']
        
        if 'status' in body:
            update_expr += ", #s = :status"
            expr_values[':status'] = body['status']
            expr_names['#s'] = 'status'
        
        # Update item
        response = table.update_item(
            Key={'id': item_id},
            UpdateExpression=update_expr,
            ExpressionAttributeValues=expr_values,
            ExpressionAttributeNames=expr_names if expr_names else None,
            ReturnValues='ALL_NEW'
        )
        
        return {
            'statusCode': 200,
            'headers': get_cors_headers(),
            'body': json.dumps({
                'message': 'Item updated successfully',
                'item': response['Attributes']
            }, default=decimal_default)
        }
        
    except Exception as e:
        print(f"Error: {str(e)}")
        return {
            'statusCode': 500,
            'headers': get_cors_headers(),
            'body': json.dumps({
                'error': 'Internal server error',
                'message': str(e)
            })
        }

def decimal_default(obj):
    if isinstance(obj, Decimal):
        return float(obj)
    raise TypeError

def get_cors_headers():
    return {
        'Content-Type': 'application/json',
        'Access-Control-Allow-Origin': '*',
        'Access-Control-Allow-Headers': 'Content-Type,X-Amz-Date,Authorization,X-Api-Key',
        'Access-Control-Allow-Methods': 'GET,POST,PUT,DELETE,OPTIONS'
    }
```

**10. lambda/functions/delete/delete.py:**

```python
import json
import boto3
import os

dynamodb = boto3.resource('dynamodb')
table_name = os.environ['TABLE_NAME']
table = dynamodb.Table(table_name)

def handler(event, context):
    """
    Lambda handler для удаления item
    """
    print(f"Event: {json.dumps(event)}")
    
    try:
        # Get item ID
        item_id = event.get('pathParameters', {}).get('id')
        if not item_id:
            return {
                'statusCode': 400,
                'headers': get_cors_headers(),
                'body': json.dumps({'error': 'Missing item ID'})
            }
        
        # Delete item
        table.delete_item(Key={'id': item_id})
        
        return {
            'statusCode': 200,
            'headers': get_cors_headers(),
            'body': json.dumps({
                'message': 'Item deleted successfully'
            })
        }
        
    except Exception as e:
        print(f"Error: {str(e)}")
        return {
            'statusCode': 500,
            'headers': get_cors_headers(),
            'body': json.dumps({
                'error': 'Internal server error',
                'message': str(e)
            })
        }

def get_cors_headers():
    return {
        'Content-Type': 'application/json',
        'Access-Control-Allow-Origin': '*',
        'Access-Control-Allow-Headers': 'Content-Type,X-Amz-Date,Authorization,X-Api-Key',
        'Access-Control-Allow-Methods': 'GET,POST,PUT,DELETE,OPTIONS'
    }
```

**11. lambda-iam.tf:**

```hcl
# IAM роль для Lambda
resource "aws_iam_role" "lambda" {
  name_prefix = "${local.name_prefix}-lambda-"
  
  assume_role_policy = jsonencode({
    Version = "2012-10-17"
    Statement = [{
      Effect = "Allow"
      Principal = {
        Service = "lambda.amazonaws.com"
      }
      Action = "sts:AssumeRole"
    }]
  })
  
  tags = local.common_tags
}

# Policy для CloudWatch Logs
resource "aws_iam_role_policy_attachment" "lambda_logs" {
  role       = aws_iam_role.lambda.name
  policy_arn = "arn:aws:iam::aws:policy/service-role/AWSLambdaBasicExecutionRole"
}

# Policy для X-Ray (если включено)
resource "aws_iam_role_policy_attachment" "lambda_xray" {
  count = var.enable_xray ? 1 : 0
  
  role       = aws_iam_role.lambda.name
  policy_arn = "arn:aws:iam::aws:policy/AWSXRayDaemonWriteAccess"
}

# Policy для DynamoDB
resource "aws_iam_role_policy" "lambda_dynamodb" {
  name_prefix = "${local.name_prefix}-lambda-dynamodb-"
  role        = aws_iam_role.lambda.id
  
  policy = jsonencode({
    Version = "2012-10-17"
    Statement = [
      {
        Effect = "Allow"
        Action = [
          "dynamodb:GetItem",
          "dynamodb:PutItem",
          "dynamodb:UpdateItem",
          "dynamodb:DeleteItem",
          "dynamodb:Query",
          "dynamodb:Scan"
        ]
        Resource = [
          aws_dynamodb_table.items.arn,
          "${aws_dynamodb_table.items.arn}/index/*"
        ]
      }
    ]
  })
}
```

**12. lambda-functions.tf:**

```hcl
# CloudWatch Log Groups для Lambda
resource "aws_cloudwatch_log_group" "lambda" {
  for_each = local.lambda_functions
  
  name              = "/aws/lambda/${local.name_prefix}-${each.key}"
  retention_in_days = var.log_retention_days
  
  tags = merge(local.common_tags, {
    Name     = "${local.name_prefix}-${each.key}-logs"
    Function = each.key
  })
}

# Package Lambda functions
data "archive_file" "lambda" {
  for_each = local.lambda_functions
  
  type        = "zip"
  source_file = "${path.module}/lambda/functions/${each.key}/${each.key}.py"
  output_path = "${path.module}/lambda/packages/${each.key}.zip"
}

# Lambda Functions
resource "aws_lambda_function" "api" {
  for_each = local.lambda_functions
  
  function_name = "${local.name_prefix}-${each.key}"
  description   = each.value.description
  role          = aws_iam_role.lambda.arn
  
  # Code
  filename         = data.archive_file.lambda[each.key].output_path
  source_code_hash = data.archive_file.lambda[each.key].output_base64sha256
  
  # Runtime
  runtime = var.lambda_runtime
  handler = each.value.handler
  
  # Resources
  memory_size = each.value.memory
  timeout     = each.value.timeout
  
  # Environment
  environment {
    variables = local.lambda_environment
  }
  
  # Tracing
  tracing_config {
    mode = var.enable_xray ? "Active" : "PassThrough"
  }
  
  # Logging
  logging_config {
    log_format = "JSON"
    log_group  = aws_cloudwatch_log_group.lambda[each.key].name
  }
  
  # Reserved concurrent executions для production
  reserved_concurrent_executions = var.environment == "prod" ? 100 : -1
  
  tags = merge(local.common_tags, {
    Name     = "${local.name_prefix}-${each.key}"
    Function = each.key
  })
  
  depends_on = [aws_cloudwatch_log_group.lambda]
}

# Lambda Permissions для API Gateway
resource "aws_lambda_permission" "api_gateway" {
  for_each = local.lambda_functions
  
  statement_id  = "AllowAPIGatewayInvoke"
  action        = "lambda:InvokeFunction"
  function_name = aws_lambda_function.api[each.key].function_name
  principal     = "apigateway.amazonaws.com"
  
  source_arn = "${aws_apigatewayv2_api.main.execution_arn}/*/*"
}
```

**13. api-gateway.tf:**

```hcl
# HTTP API (дешевле и проще чем REST API)
resource "aws_apigatewayv2_api" "main" {
  name          = "${local.name_prefix}-api"
  protocol_type = "HTTP"
  description   = "Serverless CRUD API"
  
  # CORS configuration
  cors_configuration {
    allow_origins = var.cors_allowed_origins
    allow_methods = ["GET", "POST", "PUT", "DELETE", "OPTIONS"]
    allow_headers = ["Content-Type", "X-Amz-Date", "Authorization", "X-Api-Key"]
    max_age       = 300
  }
  
  tags = merge(local.common_tags, {
    Name = "${local.name_prefix}-api"
  })
}

# API Gateway Stage
resource "aws_apigatewayv2_stage" "default" {
  api_id      = aws_apigatewayv2_api.main.id
  name        = "$default"
  auto_deploy = true
  
  # Access logs
  access_log_settings {
    destination_arn = aws_cloudwatch_log_group.api_gateway.arn
    
    format = jsonencode({
      requestId      = "$context.requestId"
      ip             = "$context.identity.sourceIp"
      requestTime    = "$context.requestTime"
      httpMethod     = "$context.httpMethod"
      routeKey       = "$context.routeKey"
      status         = "$context.status"
      protocol       = "$context.protocol"
      responseLength = "$context.responseLength"
      integrationError = "$context.integrationErrorMessage"
    })
  }
  
  # Throttling
  default_route_settings {
    throttling_burst_limit = var.api_throttle_burst_limit
    throttling_rate_limit  = var.api_throttle_rate_limit
  }
  
  tags = merge(local.common_tags, {
    Name = "${local.name_prefix}-api-stage"
  })
}

# CloudWatch Log Group для API Gateway
resource "aws_cloudwatch_log_group" "api_gateway" {
  name              = "/aws/apigateway/${local.name_prefix}"
  retention_in_days = var.log_retention_days
  
  tags = merge(local.common_tags, {
    Name = "${local.name_prefix}-api-logs"
  })
}

# Lambda Integrations
resource "aws_apigatewayv2_integration" "lambda" {
  for_each = local.lambda_functions
  
  api_id = aws_apigatewayv2_api.main.id
  
  integration_type = "AWS_PROXY"
  integration_uri  = aws_lambda_function.api[each.key].invoke_arn
  
  payload_format_version = "2.0"
  timeout_milliseconds   = (each.value.timeout + 1) * 1000
}

# API Routes
resource "aws_apigatewayv2_route" "create" {
  api_id    = aws_apigatewayv2_api.main.id
  route_key = "POST /items"
  target    = "integrations/${aws_apigatewayv2_integration.lambda["create"].id}"
}

resource "aws_apigatewayv2_route" "list" {
  api_id    = aws_apigatewayv2_api.main.id
  route_key = "GET /items"
  target    = "integrations/${aws_apigatewayv2_integration.lambda["list"].id}"
}

resource "aws_apigatewayv2_route" "read" {
  api_id    = aws_apigatewayv2_api.main.id
  route_key = "GET /items/{id}"
  target    = "integrations/${aws_apigatewayv2_integration.lambda["read"].id}"
}

resource "aws_apigatewayv2_route" "update" {
  api_id    = aws_apigatewayv2_api.main.id
  route_key = "PUT /items/{id}"
  target    = "integrations/${aws_apigatewayv2_integration.lambda["update"].id}"
}

resource "aws_apigatewayv2_route" "delete" {
  api_id    = aws_apigatewayv2_api.main.id
  route_key = "DELETE /items/{id}"
  target    = "integrations/${aws_apigatewayv2_integration.lambda["delete"].id}"
}
```

Продолжу с частью 2: EventBridge и Step Functions для создания полноценного serverless workflow.

---

## Часть 2: EventBridge + Step Functions

### 💻 Задание: Event-Driven Workflow

**14. event-bridge.tf:**

```hcl
# EventBridge Event Bus
resource "aws_cloudwatch_event_bus" "main" {
  name = "${local.name_prefix}-events"
  
  tags = merge(local.common_tags, {
    Name = "${local.name_prefix}-event-bus"
  })
}

# EventBridge Rule для DynamoDB Streams
resource "aws_cloudwatch_event_rule" "item_created" {
  name        = "${local.name_prefix}-item-created"
  description = "Trigger when new item is created"
  event_bus_name = aws_cloudwatch_event_bus.main.name
  
  event_pattern = jsonencode({
    source      = ["custom.${var.project_name}"]
    detail-type = ["Item Created"]
  })
  
  tags = merge(local.common_tags, {
    Name = "${local.name_prefix}-item-created-rule"
  })
}

# EventBridge Rule для scheduled tasks
resource "aws_cloudwatch_event_rule" "scheduled_cleanup" {
  name                = "${local.name_prefix}-cleanup"
  description         = "Run cleanup task daily"
  schedule_expression = "cron(0 2 * * ? *)"  # 2 AM UTC daily
  
  tags = merge(local.common_tags, {
    Name = "${local.name_prefix}-cleanup-rule"
  })
}

# Lambda function для обработки событий
resource "aws_lambda_function" "event_processor" {
  function_name = "${local.name_prefix}-event-processor"
  description   = "Process EventBridge events"
  role          = aws_iam_role.lambda.arn
  
  filename         = data.archive_file.event_processor.output_path
  source_code_hash = data.archive_file.event_processor.output_base64sha256
  
  runtime = var.lambda_runtime
  handler = "event_processor.handler"
  
  memory_size = 256
  timeout     = 60
  
  environment {
    variables = merge(local.lambda_environment, {
      SNS_TOPIC_ARN = aws_sns_topic.notifications.arn
    })
  }
  
  tracing_config {
    mode = var.enable_xray ? "Active" : "PassThrough"
  }
  
  logging_config {
    log_format = "JSON"
    log_group  = aws_cloudwatch_log_group.event_processor.name
  }
  
  tags = merge(local.common_tags, {
    Name = "${local.name_prefix}-event-processor"
  })
}

# CloudWatch Log Group
resource "aws_cloudwatch_log_group" "event_processor" {
  name              = "/aws/lambda/${local.name_prefix}-event-processor"
  retention_in_days = var.log_retention_days
  
  tags = local.common_tags
}

# Package event processor
data "archive_file" "event_processor" {
  type        = "zip"
  source_file = "${path.module}/lambda/functions/event_processor/event_processor.py"
  output_path = "${path.module}/lambda/packages/event_processor.zip"
}

# EventBridge Target - Lambda
resource "aws_cloudwatch_event_target" "lambda" {
  rule      = aws_cloudwatch_event_rule.item_created.name
  event_bus_name = aws_cloudwatch_event_bus.main.name
  target_id = "lambda"
  arn       = aws_lambda_function.event_processor.arn
}

# Lambda Permission для EventBridge
resource "aws_lambda_permission" "eventbridge" {
  statement_id  = "AllowExecutionFromEventBridge"
  action        = "lambda:InvokeFunction"
  function_name = aws_lambda_function.event_processor.function_name
  principal     = "events.amazonaws.com"
  source_arn    = aws_cloudwatch_event_rule.item_created.arn
}

# SNS Topic для notifications
resource "aws_sns_topic" "notifications" {
  name = "${local.name_prefix}-notifications"
  
  tags = merge(local.common_tags, {
    Name = "${local.name_prefix}-notifications"
  })
}

# SNS Topic Subscription (email для примера)
resource "aws_sns_topic_subscription" "email" {
  count = var.environment == "prod" ? 1 : 0
  
  topic_arn = aws_sns_topic.notifications.arn
  protocol  = "email"
  endpoint  = "admin@example.com"  # Замени на реальный email
}

# IAM Policy для Lambda → SNS
resource "aws_iam_role_policy" "lambda_sns" {
  name_prefix = "${local.name_prefix}-lambda-sns-"
  role        = aws_iam_role.lambda.id
  
  policy = jsonencode({
    Version = "2012-10-17"
    Statement = [{
      Effect = "Allow"
      Action = [
        "sns:Publish"
      ]
      Resource = aws_sns_topic.notifications.arn
    }]
  })
}
```

**15. lambda/functions/event_processor/event_processor.py:**

```python
import json
import boto3
import os

sns = boto3.client('sns')
sns_topic_arn = os.environ.get('SNS_TOPIC_ARN')

def handler(event, context):
    """
    Process EventBridge events
    """
    print(f"Event: {json.dumps(event)}")
    
    try:
        detail = event.get('detail', {})
        detail_type = event.get('detail-type')
        
        # Process based on event type
        if detail_type == 'Item Created':
            return process_item_created(detail)
        else:
            print(f"Unknown event type: {detail_type}")
            return {'statusCode': 200}
            
    except Exception as e:
        print(f"Error processing event: {str(e)}")
        raise

def process_item_created(detail):
    """
    Process item created event
    """
    item_id = detail.get('id')
    item_name = detail.get('name')
    
    print(f"Processing item created: {item_id} - {item_name}")
    
    # Send notification
    if sns_topic_arn:
        message = f"New item created:\nID: {item_id}\nName: {item_name}"
        
        sns.publish(
            TopicArn=sns_topic_arn,
            Subject='New Item Created',
            Message=message
        )
        
        print(f"Notification sent for item {item_id}")
    
    # Additional processing could go here
    # - Send to analytics
    # - Trigger other workflows
    # - Update search index
    # etc.
    
    return {
        'statusCode': 200,
        'message': f'Processed item {item_id}'
    }
```

**16. step-functions.tf:**

```hcl
# Step Functions State Machine для сложного workflow
resource "aws_sfn_state_machine" "item_workflow" {
  name     = "${local.name_prefix}-item-workflow"
  role_arn = aws_iam_role.step_functions.arn
  
  definition = jsonencode({
    Comment = "Item processing workflow"
    StartAt = "ValidateItem"
    States = {
      ValidateItem = {
        Type     = "Task"
        Resource = aws_lambda_function.validate_item.arn
        Next     = "CheckValidation"
        Catch = [{
          ErrorEquals = ["States.ALL"]
          Next        = "ValidationFailed"
        }]
        Retry = [{
          ErrorEquals     = ["States.TaskFailed"]
          IntervalSeconds = 2
          MaxAttempts     = 3
          BackoffRate     = 2.0
        }]
      }
      
      CheckValidation = {
        Type = "Choice"
        Choices = [{
          Variable      = "$.valid"
          BooleanEquals = true
          Next          = "ProcessItem"
        }]
        Default = "ValidationFailed"
      }
      
      ProcessItem = {
        Type     = "Task"
        Resource = aws_lambda_function.process_item.arn
        Next     = "EnrichItem"
      }
      
      EnrichItem = {
        Type     = "Task"
        Resource = aws_lambda_function.enrich_item.arn
        Next     = "SaveItem"
      }
      
      SaveItem = {
        Type     = "Task"
        Resource = aws_lambda_function.save_item.arn
        Next     = "NotifySuccess"
      }
      
      NotifySuccess = {
        Type     = "Task"
        Resource = "arn:aws:states:::sns:publish"
        Parameters = {
          TopicArn = aws_sns_topic.notifications.arn
          Message = {
            "status"  = "success"
            "item_id" = "$.id"
            "message" = "Item processed successfully"
          }
        }
        End = true
      }
      
      ValidationFailed = {
        Type     = "Task"
        Resource = "arn:aws:states:::sns:publish"
        Parameters = {
          TopicArn = aws_sns_topic.notifications.arn
          Message = {
            "status"  = "failed"
            "message" = "Item validation failed"
          }
        }
        End = true
      }
    }
  })
  
  logging_configuration {
    log_destination        = "${aws_cloudwatch_log_group.step_functions.arn}:*"
    include_execution_data = true
    level                  = "ALL"
  }
  
  tracing_configuration {
    enabled = var.enable_xray
  }
  
  tags = merge(local.common_tags, {
    Name = "${local.name_prefix}-item-workflow"
  })
}

# CloudWatch Log Group для Step Functions
resource "aws_cloudwatch_log_group" "step_functions" {
  name              = "/aws/stepfunctions/${local.name_prefix}"
  retention_in_days = var.log_retention_days
  
  tags = local.common_tags
}

# IAM роль для Step Functions
resource "aws_iam_role" "step_functions" {
  name_prefix = "${local.name_prefix}-sfn-"
  
  assume_role_policy = jsonencode({
    Version = "2012-10-17"
    Statement = [{
      Effect = "Allow"
      Principal = {
        Service = "states.amazonaws.com"
      }
      Action = "sts:AssumeRole"
    }]
  })
  
  tags = local.common_tags
}

# IAM Policy для Step Functions
resource "aws_iam_role_policy" "step_functions" {
  name_prefix = "${local.name_prefix}-sfn-"
  role        = aws_iam_role.step_functions.id
  
  policy = jsonencode({
    Version = "2012-10-17"
    Statement = [
      {
        Effect = "Allow"
        Action = [
          "lambda:InvokeFunction"
        ]
        Resource = [
          aws_lambda_function.validate_item.arn,
          aws_lambda_function.process_item.arn,
          aws_lambda_function.enrich_item.arn,
          aws_lambda_function.save_item.arn
        ]
      },
      {
        Effect = "Allow"
        Action = [
          "sns:Publish"
        ]
        Resource = aws_sns_topic.notifications.arn
      },
      {
        Effect = "Allow"
        Action = [
          "logs:CreateLogDelivery",
          "logs:GetLogDelivery",
          "logs:UpdateLogDelivery",
          "logs:DeleteLogDelivery",
          "logs:ListLogDeliveries",
          "logs:PutResourcePolicy",
          "logs:DescribeResourcePolicies",
          "logs:DescribeLogGroups"
        ]
        Resource = "*"
      },
      {
        Effect = "Allow"
        Action = [
          "xray:PutTraceSegments",
          "xray:PutTelemetryRecords"
        ]
        Resource = "*"
      }
    ]
  })
}

# Lambda functions для Step Functions workflow
resource "aws_lambda_function" "validate_item" {
  function_name = "${local.name_prefix}-validate-item"
  description   = "Validate item data"
  role          = aws_iam_role.lambda.arn
  
  filename         = data.archive_file.validate_item.output_path
  source_code_hash = data.archive_file.validate_item.output_base64sha256
  
  runtime = var.lambda_runtime
  handler = "validate_item.handler"
  
  memory_size = 128
  timeout     = 10
  
  environment {
    variables = local.lambda_environment
  }
  
  tracing_config {
    mode = var.enable_xray ? "Active" : "PassThrough"
  }
  
  tags = merge(local.common_tags, {
    Name = "${local.name_prefix}-validate-item"
  })
}

# Аналогично создай остальные Lambda functions
resource "aws_lambda_function" "process_item" {
  function_name = "${local.name_prefix}-process-item"
  description   = "Process item"
  role          = aws_iam_role.lambda.arn
  
  filename         = data.archive_file.process_item.output_path
  source_code_hash = data.archive_file.process_item.output_base64sha256
  
  runtime = var.lambda_runtime
  handler = "process_item.handler"
  
  memory_size = 256
  timeout     = 30
  
  environment {
    variables = local.lambda_environment
  }
  
  tracing_config {
    mode = var.enable_xray ? "Active" : "PassThrough"
  }
  
  tags = merge(local.common_tags, {
    Name = "${local.name_prefix}-process-item"
  })
}

resource "aws_lambda_function" "enrich_item" {
  function_name = "${local.name_prefix}-enrich-item"
  description   = "Enrich item with additional data"
  role          = aws_iam_role.lambda.arn
  
  filename         = data.archive_file.enrich_item.output_path
  source_code_hash = data.archive_file.enrich_item.output_base64sha256
  
  runtime = var.lambda_runtime
  handler = "enrich_item.handler"
  
  memory_size = 256
  timeout     = 30
  
  environment {
    variables = local.lambda_environment
  }
  
  tracing_config {
    mode = var.enable_xray ? "Active" : "PassThrough"
  }
  
  tags = merge(local.common_tags, {
    Name = "${local.name_prefix}-enrich-item"
  })
}

resource "aws_lambda_function" "save_item" {
  function_name = "${local.name_prefix}-save-item"
  description   = "Save processed item"
  role          = aws_iam_role.lambda.arn
  
  filename         = data.archive_file.save_item.output_path
  source_code_hash = data.archive_file.save_item.output_base64sha256
  
  runtime = var.lambda_runtime
  handler = "save_item.handler"
  
  memory_size = 256
  timeout     = 30
  
  environment {
    variables = local.lambda_environment
  }
  
  tracing_config {
    mode = var.enable_xray ? "Active" : "PassThrough"
  }
  
  tags = merge(local.common_tags, {
    Name = "${local.name_prefix}-save-item"
  })
}

# Archive files для Step Functions Lambda
data "archive_file" "validate_item" {
  type        = "zip"
  source_file = "${path.module}/lambda/functions/workflow/validate_item.py"
  output_path = "${path.module}/lambda/packages/validate_item.zip"
}

data "archive_file" "process_item" {
  type        = "zip"
  source_file = "${path.module}/lambda/functions/workflow/process_item.py"
  output_path = "${path.module}/lambda/packages/process_item.zip"
}

data "archive_file" "enrich_item" {
  type        = "zip"
  source_file = "${path.module}/lambda/functions/workflow/enrich_item.py"
  output_path = "${path.module}/lambda/packages/enrich_item.zip"
}

data "archive_file" "save_item" {
  type        = "zip"
  source_file = "${path.module}/lambda/functions/workflow/save_item.py"
  output_path = "${path.module}/lambda/packages/save_item.zip"
}

# EventBridge Rule для запуска Step Functions
resource "aws_cloudwatch_event_rule" "start_workflow" {
  name        = "${local.name_prefix}-start-workflow"
  description = "Start Step Functions workflow"
  
  event_pattern = jsonencode({
    source      = ["custom.${var.project_name}"]
    detail-type = ["Start Workflow"]
  })
  
  tags = merge(local.common_tags, {
    Name = "${local.name_prefix}-start-workflow-rule"
  })
}

# EventBridge Target - Step Functions
resource "aws_cloudwatch_event_target" "step_functions" {
  rule     = aws_cloudwatch_event_rule.start_workflow.name
  arn      = aws_sfn_state_machine.item_workflow.arn
  role_arn = aws_iam_role.eventbridge_sfn.arn
}

# IAM роль для EventBridge → Step Functions
resource "aws_iam_role" "eventbridge_sfn" {
  name_prefix = "${local.name_prefix}-eventbridge-sfn-"
  
  assume_role_policy = jsonencode({
    Version = "2012-10-17"
    Statement = [{
      Effect = "Allow"
      Principal = {
        Service = "events.amazonaws.com"
      }
      Action = "sts:AssumeRole"
    }]
  })
  
  tags = local.common_tags
}

resource "aws_iam_role_policy" "eventbridge_sfn" {
  name_prefix = "${local.name_prefix}-eventbridge-sfn-"
  role        = aws_iam_role.eventbridge_sfn.id
  
  policy = jsonencode({
    Version = "2012-10-17"
    Statement = [{
      Effect = "Allow"
      Action = [
        "states:StartExecution"
      ]
      Resource = aws_sfn_state_machine.item_workflow.arn
    }]
  })
}
```

**17. Lambda functions для Step Functions workflow:**

```python
# lambda/functions/workflow/validate_item.py
import json

def handler(event, context):
    """
    Validate item data
    """
    print(f"Validating item: {json.dumps(event)}")
    
    # Простая валидация
    required_fields = ['id', 'name']
    
    valid = all(field in event for field in required_fields)
    
    # Дополнительные проверки
    if valid and len(event.get('name', '')) < 3:
        valid = False
        error = "Name must be at least 3 characters"
    else:
        error = None
    
    return {
        **event,
        'valid': valid,
        'validation_error': error
    }
```

```python
# lambda/functions/workflow/process_item.py
import json
from datetime import datetime

def handler(event, context):
    """
    Process item
    """
    print(f"Processing item: {json.dumps(event)}")
    
    # Обработка item
    processed_item = {
        **event,
        'processed': True,
        'processed_at': datetime.utcnow().isoformat(),
        'processing_duration_ms': 100  # Example
    }
    
    return processed_item
```

```python
# lambda/functions/workflow/enrich_item.py
import json
import random

def handler(event, context):
    """
    Enrich item with additional data
    """
    print(f"Enriching item: {json.dumps(event)}")
    
    # Добавление дополнительных данных
    # В реальном приложении: API calls, database lookups, etc.
    enriched_item = {
        **event,
        'enrichment': {
            'category': random.choice(['electronics', 'books', 'clothing']),
            'popularity_score': random.randint(1, 100),
            'tags': ['featured', 'new']
        }
    }
    
    return enriched_item
```

```python
# lambda/functions/workflow/save_item.py
import json
import boto3
import os

dynamodb = boto3.resource('dynamodb')
table_name = os.environ['TABLE_NAME']
table = dynamodb.Table(table_name)

def handler(event, context):
    """
    Save processed item to DynamoDB
    """
    print(f"Saving item: {json.dumps(event)}")
    
    try:
        # Save to DynamoDB
        table.put_item(Item=event)
        
        return {
            **event,
            'saved': True,
            'message': 'Item saved successfully'
        }
        
    except Exception as e:
        print(f"Error saving item: {str(e)}")
        raise
```

**18. monitoring.tf:**

```hcl
# CloudWatch Dashboard для мониторинга
resource "aws_cloudwatch_dashboard" "main" {
  dashboard_name = "${local.name_prefix}-dashboard"
  
  dashboard_body = jsonencode({
    widgets = [
      {
        type = "metric"
        properties = {
          metrics = [
            for func_name, func_config in local.lambda_functions : [
              "AWS/Lambda",
              "Invocations",
              {
                stat   = "Sum"
                label  = func_name
                id     = "m${index(keys(local.lambda_functions), func_name)}"
                region = var.aws_region
              },
              {
                expression = "m${index(keys(local.lambda_functions), func_name)}/60"
                label      = "${func_name} per minute"
                id         = "e${index(keys(local.lambda_functions), func_name)}"
              }
            ]
          ]
          period = 300
          stat   = "Sum"
          region = var.aws_region
          title  = "Lambda Invocations"
        }
      },
      {
        type = "metric"
        properties = {
          metrics = [
            for func_name in keys(local.lambda_functions) : [
              "AWS/Lambda",
              "Duration",
              {
                stat  = "Average"
                label = func_name
              }
            ]
          ]
          period = 300
          stat   = "Average"
          region = var.aws_region
          title  = "Lambda Duration (ms)"
        }
      },
      {
        type = "metric"
        properties = {
          metrics = [
            [
              "AWS/Lambda",
              "Errors",
              {
                stat = "Sum"
              }
            ]
          ]
          period = 300
          stat   = "Sum"
          region = var.aws_region
          title  = "Lambda Errors"
        }
      },
      {
        type = "metric"
        properties = {
          metrics = [
            [
              "AWS/ApiGateway",
              "Count",
              {
                stat = "Sum"
              }
            ],
            [
              ".",
              "4XXError",
              {
                stat = "Sum"
              }
            ],
            [
              ".",
              "5XXError",
              {
                stat = "Sum"
              }
            ]
          ]
          period = 300
          stat   = "Sum"
          region = var.aws_region
          title  = "API Gateway Metrics"
        }
      },
      {
        type = "metric"
        properties = {
          metrics = [
            [
              "AWS/DynamoDB",
              "ConsumedReadCapacityUnits",
              {
                stat = "Sum"
              }
            ],
            [
              ".",
              "ConsumedWriteCapacityUnits",
              {
                stat = "Sum"
              }
            ]
          ]
          period = 300
          stat   = "Sum"
          region = var.aws_region
          title  = "DynamoDB Capacity"
        }
      },
      {
        type = "metric"
        properties = {
          metrics = [
            [
              "AWS/States",
              "ExecutionsFailed",
              {
                stat = "Sum"
              }
            ],
            [
              ".",
              "ExecutionsSucceeded",
              {
                stat = "Sum"
              }
            ]
          ]
          period = 300
          stat   = "Sum"
          region = var.aws_region
          title  = "Step Functions Executions"
        }
      }
    ]
  })
}

# CloudWatch Alarms
resource "aws_cloudwatch_metric_alarm" "lambda_errors" {
  alarm_name          = "${local.name_prefix}-lambda-errors"
  comparison_operator = "GreaterThanThreshold"
  evaluation_periods  = "2"
  metric_name         = "Errors"
  namespace           = "AWS/Lambda"
  period              = "300"
  statistic           = "Sum"
  threshold           = "10"
  alarm_description   = "Alert on Lambda errors"
  treat_missing_data  = "notBreaching"
  
  alarm_actions = [aws_sns_topic.notifications.arn]
  
  tags = local.common_tags
}

resource "aws_cloudwatch_metric_alarm" "api_5xx_errors" {
  alarm_name          = "${local.name_prefix}-api-5xx-errors"
  comparison_operator = "GreaterThanThreshold"
  evaluation_periods  = "2"
  metric_name         = "5XXError"
  namespace           = "AWS/ApiGateway"
  period              = "300"
  statistic           = "Sum"
  threshold           = "5"
  alarm_description   = "Alert on API Gateway 5XX errors"
  treat_missing_data  = "notBreaching"
  
  alarm_actions = [aws_sns_topic.notifications.arn]
  
  tags = local.common_tags
}

resource "aws_cloudwatch_metric_alarm" "step_functions_failed" {
  alarm_name          = "${local.name_prefix}-sfn-failures"
  comparison_operator = "GreaterThanThreshold"
  evaluation_periods  = "1"
  metric_name         = "ExecutionsFailed"
  namespace           = "AWS/States"
  period              = "300"
  statistic           = "Sum"
  threshold           = "1"
  alarm_description   = "Alert on Step Functions failures"
  treat_missing_data  = "notBreaching"
  
  dimensions = {
    StateMachineArn = aws_sfn_state_machine.item_workflow.arn
  }
  
  alarm_actions = [aws_sns_topic.notifications.arn]
  
  tags = local.common_tags
}
```

Продолжу с outputs и финальной частью.

**19. outputs.tf:**

```hcl
# API Gateway Outputs
output "api_endpoint" {
  description = "API Gateway endpoint URL"
  value       = aws_apigatewayv2_api.main.api_endpoint
}

output "api_id" {
  description = "API Gateway ID"
  value       = aws_apigatewayv2_api.main.id
}

# Lambda Functions
output "lambda_functions" {
  description = "Lambda function information"
  value = {
    for name, func in aws_lambda_function.api :
    name => {
      arn          = func.arn
      function_name = func.function_name
      version      = func.version
    }
  }
}

# DynamoDB Table
output "dynamodb_table" {
  description = "DynamoDB table information"
  value = {
    name = aws_dynamodb_table.items.name
    arn  = aws_dynamodb_table.items.arn
  }
}

# Step Functions
output "step_functions_arn" {
  description = "Step Functions state machine ARN"
  value       = aws_sfn_state_machine.item_workflow.arn
}

# EventBridge
output "event_bus_name" {
  description = "EventBridge event bus name"
  value       = aws_cloudwatch_event_bus.main.name
}

# SNS Topic
output "sns_topic_arn" {
  description = "SNS topic ARN"
  value       = aws_sns_topic.notifications.arn
}

# CloudWatch Dashboard
output "dashboard_url" {
  description = "CloudWatch Dashboard URL"
  value       = "https://console.aws.amazon.com/cloudwatch/home?region=${var.aws_region}#dashboards:name=${aws_cloudwatch_dashboard.main.dashboard_name}"
}

# Useful Commands
output "useful_commands" {
  description = "Useful AWS CLI commands"
  value = {
    test_api_create = "curl -X POST ${aws_apigatewayv2_api.main.api_endpoint}/items -H 'Content-Type: application/json' -d '{\"name\":\"Test Item\"}'"
    
    test_api_list = "curl ${aws_apigatewayv2_api.main.api_endpoint}/items"
    
    invoke_step_function = "aws stepfunctions start-execution --state-machine-arn ${aws_sfn_state_machine.item_workflow.arn} --input '{\"id\":\"test-123\",\"name\":\"Test\"}'"
    
    send_event = "aws events put-events --entries '[{\"Source\":\"custom.${var.project_name}\",\"DetailType\":\"Item Created\",\"Detail\":\"{\\\"id\\\":\\\"test-123\\\",\\\"name\\\":\\\"Test\\\"}\"}]'"
    
    view_logs = "aws logs tail /aws/lambda/${local.name_prefix}-create --follow"
    
    scan_table = "aws dynamodb scan --table-name ${aws_dynamodb_table.items.name}"
  }
}

# Architecture Diagram (текстовый)
output "architecture_overview" {
  description = "Serverless architecture overview"
  value = <<-EOT
    
    ┌─────────────────────────────────────────────┐
    │           API Gateway                        │
    │  ${aws_apigatewayv2_api.main.api_endpoint}
    └─────────────────┬───────────────────────────┘
                      │
                      ▼
    ┌─────────────────────────────────────────────┐
    │           Lambda Functions                   │
    │  ┌─────────┐  ┌─────────┐  ┌─────────┐    │
    │  │ Create  │  │  Read   │  │  List   │    │
    │  │ Update  │  │ Delete  │  │  ...    │    │
    │  └─────────┘  └─────────┘  └─────────┘    │
    └─────────────────┬───────────────────────────┘
                      │
                      ▼
    ┌─────────────────────────────────────────────┐
    │           DynamoDB                           │
    │  Table: ${aws_dynamodb_table.items.name}
    └─────────────────────────────────────────────┘
                  │
                  ▼
┌─────────────────────────────────────────────┐
│           EventBridge                        │
│  Bus: ${aws_cloudwatch_event_bus.main.name}
└─────────────────┬───────────────────────────┘
                  │
      ┌───────────┴───────────┐
      ▼                       ▼
┌──────────┐         ┌────────────────┐
│  Lambda  │         │ Step Functions │
│ Processor│         │   Workflow     │
└──────────┘         └────────────────┘
      │                       │
      ▼                       ▼
┌──────────────────────────────────┐
│         SNS Notifications         │
└──────────────────────────────────┘

EOT }

````

**20. scripts/test-api.sh:**

```bash
#!/bin/bash
# scripts/test-api.sh

set -e

# Colors
GREEN='\033[0;32m'
YELLOW='\033[1;33m'
RED='\033[0;31m'
NC='\033[0m'

# Get API endpoint from Terraform
API_ENDPOINT=$(terraform output -raw api_endpoint 2>/dev/null)

if [ -z "$API_ENDPOINT" ]; then
    echo -e "${RED}Error: Could not get API endpoint${NC}"
    echo "Run: terraform apply first"
    exit 1
fi

echo -e "${GREEN}Testing API: $API_ENDPOINT${NC}"
echo ""

# Test 1: Create item
echo -e "${YELLOW}Test 1: Create Item${NC}"
CREATE_RESPONSE=$(curl -s -X POST "$API_ENDPOINT/items" \
    -H "Content-Type: application/json" \
    -d '{"name":"Test Item","description":"Created by test script"}')

echo "$CREATE_RESPONSE" | jq .
ITEM_ID=$(echo "$CREATE_RESPONSE" | jq -r '.item.id')
echo -e "${GREEN}✓ Created item: $ITEM_ID${NC}"
echo ""

# Test 2: List items
echo -e "${YELLOW}Test 2: List Items${NC}"
curl -s "$API_ENDPOINT/items" | jq .
echo -e "${GREEN}✓ Listed items${NC}"
echo ""

# Test 3: Read item
echo -e "${YELLOW}Test 3: Read Item${NC}"
curl -s "$API_ENDPOINT/items/$ITEM_ID" | jq .
echo -e "${GREEN}✓ Read item: $ITEM_ID${NC}"
echo ""

# Test 4: Update item
echo -e "${YELLOW}Test 4: Update Item${NC}"
curl -s -X PUT "$API_ENDPOINT/items/$ITEM_ID" \
    -H "Content-Type: application/json" \
    -d '{"name":"Updated Item","description":"Updated by test script","status":"active"}' | jq .
echo -e "${GREEN}✓ Updated item: $ITEM_ID${NC}"
echo ""

# Test 5: Delete item
echo -e "${YELLOW}Test 5: Delete Item${NC}"
curl -s -X DELETE "$API_ENDPOINT/items/$ITEM_ID" | jq .
echo -e "${GREEN}✓ Deleted item: $ITEM_ID${NC}"
echo ""

echo -e "${GREEN}All tests passed!${NC}"
````

```bash
chmod +x scripts/test-api.sh
```

**21. scripts/trigger-workflow.sh:**

```bash
#!/bin/bash
# scripts/trigger-workflow.sh

set -e

# Get Step Functions ARN
SFN_ARN=$(terraform output -raw step_functions_arn 2>/dev/null)

if [ -z "$SFN_ARN" ]; then
    echo "Error: Could not get Step Functions ARN"
    exit 1
fi

echo "Triggering Step Functions workflow..."

# Start execution
EXECUTION_ARN=$(aws stepfunctions start-execution \
    --state-machine-arn "$SFN_ARN" \
    --input '{"id":"workflow-test-123","name":"Test Workflow Item"}' \
    --query 'executionArn' \
    --output text)

echo "Execution started: $EXECUTION_ARN"
echo ""

# Wait and check status
echo "Waiting for execution to complete..."
sleep 3

STATUS=$(aws stepfunctions describe-execution \
    --execution-arn "$EXECUTION_ARN" \
    --query 'status' \
    --output text)

echo "Status: $STATUS"

if [ "$STATUS" == "SUCCEEDED" ]; then
    echo "✓ Workflow completed successfully"
    
    # Get output
    echo ""
    echo "Output:"
    aws stepfunctions describe-execution \
        --execution-arn "$EXECUTION_ARN" \
        --query 'output' \
        --output text | jq .
else
    echo "⚠ Workflow status: $STATUS"
fi
```

```bash
chmod +x scripts/trigger-workflow.sh
```

**22. Makefile:**

```makefile
.PHONY: help init plan apply destroy test-api test-workflow logs clean

ENVIRONMENT ?= dev

help:
	@echo "Serverless Terraform Commands:"
	@echo "  init           - Initialize Terraform"
	@echo "  plan           - Plan infrastructure"
	@echo "  apply          - Apply infrastructure"
	@echo "  destroy        - Destroy infrastructure"
	@echo "  test-api       - Test API endpoints"
	@echo "  test-workflow  - Test Step Functions workflow"
	@echo "  logs           - Tail Lambda logs"
	@echo "  clean          - Clean build artifacts"

init:
	terraform init

plan:
	terraform plan -var="environment=$(ENVIRONMENT)" -out=tfplan

apply:
	terraform apply tfplan

destroy:
	terraform destroy -var="environment=$(ENVIRONMENT)"

test-api:
	@./scripts/test-api.sh

test-workflow:
	@./scripts/trigger-workflow.sh

logs:
	@API_ENDPOINT=$$(terraform output -raw api_endpoint); \
	FUNCTION_NAME=$$(echo $$API_ENDPOINT | grep -oP '(?<=https://).*(?=.execute-api)'); \
	aws logs tail /aws/lambda/serverless-api-$(ENVIRONMENT)-create --follow

# Package all Lambda functions
package:
	@echo "Packaging Lambda functions..."
	@for func in lambda/functions/*/*.py; do \
		dir=$$(dirname $$func); \
		name=$$(basename $$dir); \
		cd $$dir && zip -r ../../../lambda/packages/$$name.zip *.py && cd -; \
	done

# View CloudWatch Dashboard
dashboard:
	@DASHBOARD_URL=$$(terraform output -raw dashboard_url); \
	open $$DASHBOARD_URL || xdg-open $$DASHBOARD_URL

# Clean artifacts
clean:
	@rm -rf lambda/packages/*.zip
	@rm -rf .terraform
	@rm -f tfplan
	@rm -f terraform.tfstate*

.DEFAULT_GOAL := help
```

**23. README.md:**

```markdown
# Serverless Application with Terraform

Complete serverless application на AWS с Lambda, API Gateway, DynamoDB, EventBridge и Step Functions.

## 🏗️ Architecture

```

API Gateway (HTTP API) ↓ Lambda Functions (CRUD) ↓ DynamoDB (NoSQL) ↓ EventBridge (Events) ↓ Step Functions (Workflows) ↓ SNS (Notifications)

````

## 🚀 Quick Start

```bash
# 1. Initialize
terraform init

# 2. Deploy
terraform plan -out=tfplan
terraform apply tfplan

# 3. Get API endpoint
terraform output api_endpoint
````

## 📝 API Endpoints

```bash
POST   /items      - Create item
GET    /items      - List items
GET    /items/{id} - Get item
PUT    /items/{id} - Update item
DELETE /items/{id} - Delete item
```

## 🧪 Testing

```bash
# Test API
make test-api

# Test workflow
make test-workflow

# View logs
make logs
```

## 📊 Monitoring

```bash
# View CloudWatch Dashboard
make dashboard

# Check alarms
aws cloudwatch describe-alarms --alarm-name-prefix serverless-api
```

## 💰 Cost Optimization

- Lambda: Pay per invocation
- API Gateway HTTP API: Дешевле REST API
- DynamoDB: On-demand billing
- EventBridge: Pay per event
- Step Functions: Pay per state transition

## 🔒 Security

- ✅ IAM least privilege
- ✅ API Gateway throttling
- ✅ Lambda environment variables
- ✅ DynamoDB encryption
- ✅ CloudWatch logging

## 📈 Scalability

- Lambda: Автомасштабирование
- API Gateway: 10,000 RPS default
- DynamoDB: On-demand capacity
- Step Functions: Automatic scaling


**24. Запуск и тестирование:**

```bash
# Создать необходимые директории
mkdir -p lambda/packages lambda/functions/workflow

# Создать Lambda functions
# (создай файлы validate_item.py, process_item.py, enrich_item.py, save_item.py)

# Инициализация
terraform init

# План
terraform plan -var="environment=dev" -out=tfplan

# Применение
terraform apply tfplan

# Получить API endpoint
API_ENDPOINT=$(terraform output -raw api_endpoint)
echo "API Endpoint: $API_ENDPOINT"

# Тест 1: Create item
curl -X POST "$API_ENDPOINT/items" \
  -H "Content-Type: application/json" \
  -d '{"name":"Test Item","description":"Test"}'

# Тест 2: List items
curl "$API_ENDPOINT/items"

# Тест 3: Запустить workflow
./scripts/trigger-workflow.sh

# Тест 4: Send event
aws events put-events --entries \
  '[{"Source":"custom.serverless-api","DetailType":"Item Created","Detail":"{\"id\":\"test-123\",\"name\":\"Test\"}"}]'

# Мониторинг
aws logs tail /aws/lambda/serverless-api-dev-create --follow

# Проверка DynamoDB
aws dynamodb scan --table-name serverless-api-dev-items --max-items 10

# Cleanup
terraform destroy -var="environment=dev"
````

---

## 🎯 Заключение Модуля 15

### 📚 Ключевые takeaways

**1. Serverless Components:**

```
Lambda
├── Stateless functions
├── Automatic scaling
├── Pay per invocation
└── No server management

API Gateway
├── HTTP/REST/WebSocket APIs
├── Built-in throttling
├── CORS support
└── Caching

DynamoDB
├── NoSQL database
├── On-demand capacity
├── Single-digit ms latency
└── Global tables

EventBridge
├── Event bus
├── Event routing
├── Schema registry
└── 90+ integrations

Step Functions
├── Visual workflows
├── Error handling
├── Parallel execution
└── Human approval steps
```

**2. Best Practices:**

```hcl
✅ DO:
- Use HTTP API (дешевле REST API)
- Enable X-Ray tracing
- Use environment variables
- Implement retry logic
- Set proper timeouts
- Use layers для shared code
- Monitor with CloudWatch

❌ DON'T:
- Store state in Lambda
- Use large deployment packages
- Ignore cold starts
- Skip error handling
- Forget about limits
```

**3. Cost Optimization:**

|Service|Free Tier|Pricing|
|---|---|---|
|Lambda|1M requests/mo|$0.20 per 1M requests|
|API Gateway HTTP|N/A|$1.00 per million|
|DynamoDB|25GB storage|On-demand or provisioned|
|EventBridge|Free for AWS events|$1.00 per million custom|
|Step Functions|4,000 transitions|$0.025 per 1,000|

**4. Performance Tips:**

- ✅ Use Lambda Layers для dependencies
- ✅ Optimize package size
- ✅ Use provisioned concurrency для low latency
- ✅ Implement caching
- ✅ Use DynamoDB global tables для multi-region

Ты освоил:

- ✅ Lambda functions с Terraform
- ✅ API Gateway HTTP API
- ✅ DynamoDB NoSQL database
- ✅ EventBridge event-driven architecture
- ✅ Step Functions workflows
- ✅ Complete serverless application

**Next Steps:**

1. Добавить Cognito для authentication
2. Implement API Gateway custom authorizers
3. Add Lambda layers
4. Multi-region deployment
5. Advanced monitoring с X-Ray

---

# Модуль 16: ECS/EKS Deployment (30 минут)

## 🎯 Напоминалка

**Container Orchestration на AWS:**

```
AWS Container Services
├── ECS (Elastic Container Service)
│   ├── EC2 Launch Type - управление EC2 instances
│   └── Fargate Launch Type - serverless containers
└── EKS (Elastic Kubernetes Service)
    ├── Managed Kubernetes control plane
    └── Self-managed or Managed node groups
```

**ECS Architecture:**

```
ECS Components
├── Cluster - логическая группа ресурсов
├── Task Definition - blueprint для containers
│   ├── Container definitions
│   ├── CPU/Memory requirements
│   ├── Networking mode
│   └── IAM roles
├── Service - управление tasks
│   ├── Desired count
│   ├── Load balancing
│   └── Auto scaling
└── Tasks - running containers
```

**EKS Architecture:**

```
EKS Components
├── Control Plane (AWS managed)
│   ├── API Server
│   ├── etcd
│   └── Controller Manager
├── Data Plane (Customer managed)
│   ├── Worker Nodes (EC2/Fargate)
│   ├── Pods
│   └── Services
└── Add-ons
    ├── VPC CNI
    ├── CoreDNS
    └── kube-proxy
```

**Основные концепции:**

```hcl
# ECS Task Definition
resource "aws_ecs_task_definition" "app" {
  family                   = "my-app"
  network_mode             = "awsvpc"
  requires_compatibilities = ["FARGATE"]
  cpu                      = "256"
  memory                   = "512"
  
  container_definitions = jsonencode([{
    name  = "app"
    image = "nginx:latest"
    portMappings = [{
      containerPort = 80
      protocol      = "tcp"
    }]
  }])
}

# ECS Service
resource "aws_ecs_service" "app" {
  name            = "my-app-service"
  cluster         = aws_ecs_cluster.main.id
  task_definition = aws_ecs_task_definition.app.arn
  desired_count   = 2
  launch_type     = "FARGATE"
  
  network_configuration {
    subnets         = var.private_subnet_ids
    security_groups = [aws_security_group.app.id]
  }
  
  load_balancer {
    target_group_arn = aws_lb_target_group.app.arn
    container_name   = "app"
    container_port   = 80
  }
}
```

---

## Часть 1: ECS Fargate Deployment

### 💻 Задание: Production-Ready ECS Fargate Setup

**1. Структура проекта:**

```bash
mkdir terraform-ecs
cd terraform-ecs
mkdir -p modules/{networking,ecs,alb,security} docker
```

**2. provider.tf:**

```hcl
terraform {
  required_version = ">= 1.6.0"
  
  required_providers {
    aws = {
      source  = "hashicorp/aws"
      version = "~> 5.0"
    }
  }
}

provider "aws" {
  region = var.aws_region
  
  default_tags {
    tags = local.common_tags
  }
}
```

**3. variables.tf:**

```hcl
variable "aws_region" {
  description = "AWS region"
  type        = string
  default     = "us-east-1"
}

variable "project_name" {
  description = "Project name"
  type        = string
  default     = "ecs-demo"
}

variable "environment" {
  description = "Environment name"
  type        = string
  
  validation {
    condition     = contains(["dev", "staging", "prod"], var.environment)
    error_message = "Must be dev, staging, or prod."
  }
}

variable "vpc_cidr" {
  description = "VPC CIDR block"
  type        = string
  default     = "10.0.0.0/16"
}

variable "availability_zones" {
  description = "Availability zones"
  type        = list(string)
  default     = ["us-east-1a", "us-east-1b", "us-east-1c"]
}

variable "container_image" {
  description = "Docker image for the application"
  type        = string
  default     = "nginx:latest"
}

variable "container_port" {
  description = "Container port"
  type        = number
  default     = 80
}

variable "cpu" {
  description = "Task CPU units"
  type        = string
  default     = "256"
  
  validation {
    condition     = contains(["256", "512", "1024", "2048", "4096"], var.cpu)
    error_message = "CPU must be a valid Fargate value."
  }
}

variable "memory" {
  description = "Task memory in MB"
  type        = string
  default     = "512"
  
  validation {
    condition     = contains(["512", "1024", "2048", "4096", "8192"], var.memory)
    error_message = "Memory must be a valid Fargate value."
  }
}

variable "desired_count" {
  description = "Desired number of tasks"
  type        = number
  default     = 2
  
  validation {
    condition     = var.desired_count >= 1 && var.desired_count <= 10
    error_message = "Desired count must be between 1 and 10."
  }
}

variable "enable_autoscaling" {
  description = "Enable auto scaling"
  type        = bool
  default     = true
}

variable "min_capacity" {
  description = "Minimum number of tasks"
  type        = number
  default     = 2
}

variable "max_capacity" {
  description = "Maximum number of tasks"
  type        = number
  default     = 10
}

variable "enable_execute_command" {
  description = "Enable ECS Exec for debugging"
  type        = bool
  default     = false
}

variable "health_check_path" {
  description = "Health check path"
  type        = string
  default     = "/"
}

variable "health_check_grace_period" {
  description = "Health check grace period in seconds"
  type        = number
  default     = 60
}
```

**4. locals.tf:**

```hcl
locals {
  name_prefix = "${var.project_name}-${var.environment}"
  
  common_tags = {
    Project     = var.project_name
    Environment = var.environment
    ManagedBy   = "Terraform"
    Service     = "ECS"
  }
  
  # Environment-specific settings
  env_config = {
    dev = {
      enable_deletion_protection = false
      enable_container_insights  = false
      log_retention_days        = 7
    }
    staging = {
      enable_deletion_protection = false
      enable_container_insights  = true
      log_retention_days        = 14
    }
    prod = {
      enable_deletion_protection = true
      enable_container_insights  = true
      log_retention_days        = 30
    }
  }
  
  current_config = local.env_config[var.environment]
}
```

**5. data.tf:**

```hcl
data "aws_caller_identity" "current" {}
data "aws_region" "current" {}

# Latest ECS-optimized AMI (if needed for EC2 launch type)
data "aws_ami" "ecs_optimized" {
  most_recent = true
  owners      = ["amazon"]
  
  filter {
    name   = "name"
    values = ["amzn2-ami-ecs-hvm-*-x86_64-ebs"]
  }
}
```

**6. networking.tf:**

```hcl
# VPC
resource "aws_vpc" "main" {
  cidr_block           = var.vpc_cidr
  enable_dns_hostnames = true
  enable_dns_support   = true
  
  tags = merge(local.common_tags, {
    Name = "${local.name_prefix}-vpc"
  })
}

# Internet Gateway
resource "aws_internet_gateway" "main" {
  vpc_id = aws_vpc.main.id
  
  tags = merge(local.common_tags, {
    Name = "${local.name_prefix}-igw"
  })
}

# Public Subnets
resource "aws_subnet" "public" {
  count = length(var.availability_zones)
  
  vpc_id                  = aws_vpc.main.id
  cidr_block              = cidrsubnet(var.vpc_cidr, 8, count.index)
  availability_zone       = var.availability_zones[count.index]
  map_public_ip_on_launch = true
  
  tags = merge(local.common_tags, {
    Name = "${local.name_prefix}-public-${var.availability_zones[count.index]}"
    Type = "Public"
  })
}

# Private Subnets
resource "aws_subnet" "private" {
  count = length(var.availability_zones)
  
  vpc_id            = aws_vpc.main.id
  cidr_block        = cidrsubnet(var.vpc_cidr, 8, count.index + 10)
  availability_zone = var.availability_zones[count.index]
  
  tags = merge(local.common_tags, {
    Name = "${local.name_prefix}-private-${var.availability_zones[count.index]}"
    Type = "Private"
  })
}

# Elastic IPs for NAT Gateways
resource "aws_eip" "nat" {
  count = length(var.availability_zones)
  
  domain = "vpc"
  
  tags = merge(local.common_tags, {
    Name = "${local.name_prefix}-nat-eip-${count.index + 1}"
  })
  
  depends_on = [aws_internet_gateway.main]
}

# NAT Gateways
resource "aws_nat_gateway" "main" {
  count = length(var.availability_zones)
  
  allocation_id = aws_eip.nat[count.index].id
  subnet_id     = aws_subnet.public[count.index].id
  
  tags = merge(local.common_tags, {
    Name = "${local.name_prefix}-nat-${count.index + 1}"
  })
  
  depends_on = [aws_internet_gateway.main]
}

# Public Route Table
resource "aws_route_table" "public" {
  vpc_id = aws_vpc.main.id
  
  route {
    cidr_block = "0.0.0.0/0"
    gateway_id = aws_internet_gateway.main.id
  }
  
  tags = merge(local.common_tags, {
    Name = "${local.name_prefix}-public-rt"
  })
}

# Private Route Tables
resource "aws_route_table" "private" {
  count = length(var.availability_zones)
  
  vpc_id = aws_vpc.main.id
  
  route {
    cidr_block     = "0.0.0.0/0"
    nat_gateway_id = aws_nat_gateway.main[count.index].id
  }
  
  tags = merge(local.common_tags, {
    Name = "${local.name_prefix}-private-rt-${count.index + 1}"
  })
}

# Route Table Associations
resource "aws_route_table_association" "public" {
  count = length(aws_subnet.public)
  
  subnet_id      = aws_subnet.public[count.index].id
  route_table_id = aws_route_table.public.id
}

resource "aws_route_table_association" "private" {
  count = length(aws_subnet.private)
  
  subnet_id      = aws_subnet.private[count.index].id
  route_table_id = aws_route_table.private[count.index].id
}
```

**7. security-groups.tf:**

```hcl
# ALB Security Group
resource "aws_security_group" "alb" {
  name_prefix = "${local.name_prefix}-alb-"
  description = "Security group for Application Load Balancer"
  vpc_id      = aws_vpc.main.id
  
  tags = merge(local.common_tags, {
    Name = "${local.name_prefix}-alb-sg"
  })
  
  lifecycle {
    create_before_destroy = true
  }
}

resource "aws_vpc_security_group_ingress_rule" "alb_http" {
  security_group_id = aws_security_group.alb.id
  
  from_port   = 80
  to_port     = 80
  ip_protocol = "tcp"
  cidr_ipv4   = "0.0.0.0/0"
  description = "HTTP from anywhere"
}

resource "aws_vpc_security_group_ingress_rule" "alb_https" {
  security_group_id = aws_security_group.alb.id
  
  from_port   = 443
  to_port     = 443
  ip_protocol = "tcp"
  cidr_ipv4   = "0.0.0.0/0"
  description = "HTTPS from anywhere"
}

resource "aws_vpc_security_group_egress_rule" "alb_all" {
  security_group_id = aws_security_group.alb.id
  
  ip_protocol = "-1"
  cidr_ipv4   = "0.0.0.0/0"
  description = "All outbound traffic"
}

# ECS Tasks Security Group
resource "aws_security_group" "ecs_tasks" {
  name_prefix = "${local.name_prefix}-ecs-tasks-"
  description = "Security group for ECS tasks"
  vpc_id      = aws_vpc.main.id
  
  tags = merge(local.common_tags, {
    Name = "${local.name_prefix}-ecs-tasks-sg"
  })
  
  lifecycle {
    create_before_destroy = true
  }
}

resource "aws_vpc_security_group_ingress_rule" "ecs_from_alb" {
  security_group_id = aws_security_group.ecs_tasks.id
  
  from_port                    = var.container_port
  to_port                      = var.container_port
  ip_protocol                  = "tcp"
  referenced_security_group_id = aws_security_group.alb.id
  description                  = "Traffic from ALB"
}

resource "aws_vpc_security_group_egress_rule" "ecs_all" {
  security_group_id = aws_security_group.ecs_tasks.id
  
  ip_protocol = "-1"
  cidr_ipv4   = "0.0.0.0/0"
  description = "All outbound traffic"
}
```

**8. iam.tf:**

```hcl
# ECS Task Execution Role
resource "aws_iam_role" "ecs_task_execution" {
  name_prefix = "${local.name_prefix}-ecs-exec-"
  
  assume_role_policy = jsonencode({
    Version = "2012-10-17"
    Statement = [{
      Effect = "Allow"
      Principal = {
        Service = "ecs-tasks.amazonaws.com"
      }
      Action = "sts:AssumeRole"
    }]
  })
  
  tags = local.common_tags
}

resource "aws_iam_role_policy_attachment" "ecs_task_execution" {
  role       = aws_iam_role.ecs_task_execution.name
  policy_arn = "arn:aws:iam::aws:policy/service-role/AmazonECSTaskExecutionRolePolicy"
}

# Additional permissions for ECR and Secrets Manager
resource "aws_iam_role_policy" "ecs_task_execution_additional" {
  name_prefix = "${local.name_prefix}-ecs-exec-additional-"
  role        = aws_iam_role.ecs_task_execution.id
  
  policy = jsonencode({
    Version = "2012-10-17"
    Statement = [
      {
        Effect = "Allow"
        Action = [
          "ecr:GetAuthorizationToken",
          "ecr:BatchCheckLayerAvailability",
          "ecr:GetDownloadUrlForLayer",
          "ecr:BatchGetImage"
        ]
        Resource = "*"
      },
      {
        Effect = "Allow"
        Action = [
          "secretsmanager:GetSecretValue",
          "kms:Decrypt"
        ]
        Resource = "*"
      },
      {
        Effect = "Allow"
        Action = [
          "logs:CreateLogGroup",
          "logs:CreateLogStream",
          "logs:PutLogEvents"
        ]
        Resource = "*"
      }
    ]
  })
}

# ECS Task Role (for application permissions)
resource "aws_iam_role" "ecs_task" {
  name_prefix = "${local.name_prefix}-ecs-task-"
  
  assume_role_policy = jsonencode({
    Version = "2012-10-17"
    Statement = [{
      Effect = "Allow"
      Principal = {
        Service = "ecs-tasks.amazonaws.com"
      }
      Action = "sts:AssumeRole"
    }]
  })
  
  tags = local.common_tags
}

# Task role policy (customize based on application needs)
resource "aws_iam_role_policy" "ecs_task" {
  name_prefix = "${local.name_prefix}-ecs-task-policy-"
  role        = aws_iam_role.ecs_task.id
  
  policy = jsonencode({
    Version = "2012-10-17"
    Statement = [
      {
        Effect = "Allow"
        Action = [
          "s3:GetObject",
          "s3:PutObject"
        ]
        Resource = "arn:aws:s3:::${local.name_prefix}-*/*"
      },
      {
        Effect = "Allow"
        Action = [
          "dynamodb:GetItem",
          "dynamodb:PutItem",
          "dynamodb:Query",
          "dynamodb:Scan"
        ]
        Resource = "arn:aws:dynamodb:${data.aws_region.current.name}:${data.aws_caller_identity.current.account_id}:table/${local.name_prefix}-*"
      }
    ]
  })
}

# ECS Exec Role (for debugging with ECS Exec)
resource "aws_iam_role_policy" "ecs_exec" {
  count = var.enable_execute_command ? 1 : 0
  
  name_prefix = "${local.name_prefix}-ecs-exec-"
  role        = aws_iam_role.ecs_task.id
  
  policy = jsonencode({
    Version = "2012-10-17"
    Statement = [{
      Effect = "Allow"
      Action = [
        "ssmmessages:CreateControlChannel",
        "ssmmessages:CreateDataChannel",
        "ssmmessages:OpenControlChannel",
        "ssmmessages:OpenDataChannel"
      ]
      Resource = "*"
    }]
  })
}

# Auto Scaling Role
resource "aws_iam_role" "ecs_autoscale" {
  name_prefix = "${local.name_prefix}-ecs-autoscale-"
  
  assume_role_policy = jsonencode({
    Version = "2012-10-17"
    Statement = [{
      Effect = "Allow"
      Principal = {
        Service = "application-autoscaling.amazonaws.com"
      }
      Action = "sts:AssumeRole"
    }]
  })
  
  tags = local.common_tags
}

resource "aws_iam_role_policy" "ecs_autoscale" {
  name_prefix = "${local.name_prefix}-ecs-autoscale-policy-"
  role        = aws_iam_role.ecs_autoscale.id
  
  policy = jsonencode({
    Version = "2012-10-17"
    Statement = [
      {
        Effect = "Allow"
        Action = [
          "ecs:DescribeServices",
          "ecs:UpdateService"
        ]
        Resource = "*"
      },
      {
        Effect = "Allow"
        Action = [
          "cloudwatch:DescribeAlarms",
          "cloudwatch:PutMetricAlarm"
        ]
        Resource = "*"
      }
    ]
  })
}
```

**9. cloudwatch.tf:**

```hcl
# CloudWatch Log Group
resource "aws_cloudwatch_log_group" "ecs" {
  name              = "/ecs/${local.name_prefix}"
  retention_in_days = local.current_config.log_retention_days
  
  tags = merge(local.common_tags, {
    Name = "${local.name_prefix}-ecs-logs"
  })
}

# CloudWatch Log Stream
resource "aws_cloudwatch_log_stream" "ecs" {
  name           = "ecs-tasks"
  log_group_name = aws_cloudwatch_log_group.ecs.name
}

# CloudWatch Metric Alarms

# High CPU Alarm
resource "aws_cloudwatch_metric_alarm" "ecs_cpu_high" {
  alarm_name          = "${local.name_prefix}-ecs-cpu-high"
  comparison_operator = "GreaterThanThreshold"
  evaluation_periods  = "2"
  metric_name         = "CPUUtilization"
  namespace           = "AWS/ECS"
  period              = "60"
  statistic           = "Average"
  threshold           = "80"
  alarm_description   = "ECS CPU utilization is too high"
  
  dimensions = {
    ClusterName = aws_ecs_cluster.main.name
    ServiceName = aws_ecs_service.app.name
  }
  
  alarm_actions = var.enable_autoscaling ? [aws_appautoscaling_policy.ecs_scale_up[0].arn] : []
  
  tags = local.common_tags
}

# Low CPU Alarm
resource "aws_cloudwatch_metric_alarm" "ecs_cpu_low" {
  alarm_name          = "${local.name_prefix}-ecs-cpu-low"
  comparison_operator = "LessThanThreshold"
  evaluation_periods  = "2"
  metric_name         = "CPUUtilization"
  namespace           = "AWS/ECS"
  period              = "60"
  statistic           = "Average"
  threshold           = "20"
  alarm_description   = "ECS CPU utilization is too low"
  
  dimensions = {
    ClusterName = aws_ecs_cluster.main.name
    ServiceName = aws_ecs_service.app.name
  }
  
  alarm_actions = var.enable_autoscaling ? [aws_appautoscaling_policy.ecs_scale_down[0].arn] : []
  
  tags = local.common_tags
}

# High Memory Alarm
resource "aws_cloudwatch_metric_alarm" "ecs_memory_high" {
  alarm_name          = "${local.name_prefix}-ecs-memory-high"
  comparison_operator = "GreaterThanThreshold"
  evaluation_periods  = "2"
  metric_name         = "MemoryUtilization"
  namespace           = "AWS/ECS"
  period              = "60"
  statistic           = "Average"
  threshold           = "80"
  alarm_description   = "ECS Memory utilization is too high"
  
  dimensions = {
    ClusterName = aws_ecs_cluster.main.name
    ServiceName = aws_ecs_service.app.name
  }
  
  tags = local.common_tags
}

# Target tracking alarms will be created automatically by auto scaling
```

**10. alb.tf:**

```hcl
# Application Load Balancer
resource "aws_lb" "main" {
  name_prefix        = substr(local.name_prefix, 0, 6)
  internal           = false
  load_balancer_type = "application"
  security_groups    = [aws_security_group.alb.id]
  subnets            = aws_subnet.public[*].id
  
  enable_deletion_protection = local.current_config.enable_deletion_protection
  enable_http2              = true
  enable_cross_zone_load_balancing = true
  
  tags = merge(local.common_tags, {
    Name = "${local.name_prefix}-alb"
  })
}

# Target Group
resource "aws_lb_target_group" "app" {
  name_prefix = substr(local.name_prefix, 0, 6)
  port        = var.container_port
  protocol    = "HTTP"
  vpc_id      = aws_vpc.main.id
  target_type = "ip"
  
  deregistration_delay = 30
  
  health_check {
    enabled             = true
    healthy_threshold   = 2
    unhealthy_threshold = 3
    timeout             = 5
    interval            = 30
    path                = var.health_check_path
    protocol            = "HTTP"
    matcher             = "200-299"
  }
  
  tags = merge(local.common_tags, {
    Name = "${local.name_prefix}-tg"
  })
  
  lifecycle {
    create_before_destroy = true
  }
}

# HTTP Listener
resource "aws_lb_listener" "http" {
  load_balancer_arn = aws_lb.main.arn
  port              = 80
  protocol          = "HTTP"
  
  default_action {
    type             = "forward"
    target_group_arn = aws_lb_target_group.app.arn
  }
}

# HTTPS Listener (если есть сертификат)
# resource "aws_lb_listener" "https" {
#   load_balancer_arn = aws_lb.main.arn
#   port              = 443
#   protocol          = "HTTPS"
#   ssl_policy        = "ELBSecurityPolicy-TLS-1-2-2017-01"
#   certificate_arn   = var.certificate_arn
#   
#   default_action {
#     type             = "forward"
#     target_group_arn = aws_lb_target_group.app.arn
#   }
# }
```

**11. ecs-cluster.tf:**

```hcl
# ECS Cluster
resource "aws_ecs_cluster" "main" {
  name = local.name_prefix
  
  setting {
    name  = "containerInsights"
    value = local.current_config.enable_container_insights ? "enabled" : "disabled"
  }
  
  tags = merge(local.common_tags, {
    Name = "${local.name_prefix}-cluster"
  })
}

# Cluster Capacity Providers
resource "aws_ecs_cluster_capacity_providers" "main" {
  cluster_name = aws_ecs_cluster.main.name
  
  capacity_providers = ["FARGATE", "FARGATE_SPOT"]
  
  default_capacity_provider_strategy {
    capacity_provider = "FARGATE"
    weight           = 1
    base             = 1
  }
}
```

**12. ecs-task-definition.tf:**

```hcl
# Task Definition
resource "aws_ecs_task_definition" "app" {
  family                   = local.name_prefix
  network_mode             = "awsvpc"
  requires_compatibilities = ["FARGATE"]
  cpu                      = var.cpu
  memory                   = var.memory
  
  execution_role_arn = aws_iam_role.ecs_task_execution.arn
  task_role_arn      = aws_iam_role.ecs_task.arn
  
  container_definitions = jsonencode([
    {
      name      = "app"
      image     = var.container_image
      essential = true
      
      portMappings = [{
        containerPort = var.container_port
        protocol      = "tcp"
      }]
      
      environment = [
        {
          name  = "ENVIRONMENT"
          value = var.environment
        },
        {
          name  = "PROJECT_NAME"
          value = var.project_name
        },
        {
          name  = "AWS_REGION"
          value = data.aws_region.current.name
        }
      ]
      
      # Secrets from Secrets Manager (optional)
      # secrets = [{
      #   name      = "DB_PASSWORD"
      #   valueFrom = aws_secretsmanager_secret.db_password.arn
      # }]
      
      logConfiguration = {
        logDriver = "awslogs"
        options = {
          "awslogs-group"         = aws_cloudwatch_log_group.ecs.name
          "awslogs-region"        = data.aws_region.current.name
          "awslogs-stream-prefix" = "app"
        }
      }
      
      healthCheck = {
        command     = ["CMD-SHELL", "curl -f http://localhost:${var.container_port}${var.health_check_path} || exit 1"]
        interval    = 30
        timeout     = 5
        retries     = 3
        startPeriod = 60
      }
      
      # Resource limits
      ulimits = [{
        name      = "nofile"
        softLimit = 65536
        hardLimit = 65536
      }]
      
      # Container insights
      linuxParameters = {
        initProcessEnabled = true
      }
    }
  ])
  
  tags = merge(local.common_tags, {
    Name = "${local.name_prefix}-task-def"
  })
}
```

**13. ecs-service.tf:**

```hcl
# ECS Service
resource "aws_ecs_service" "app" {
  name            = "${local.name_prefix}-service"
  cluster         = aws_ecs_cluster.main.id
  task_definition = aws_ecs_task_definition.app.arn
  desired_count   = var.desired_count
  launch_type     = "FARGATE"
  
  platform_version = "LATEST"
  
  enable_execute_command = var.enable_execute_command
  
  health_check_grace_period_seconds = var.health_check_grace_period
  
  network_configuration {
    subnets          = aws_subnet.private[*].id
    security_groups  = [aws_security_group.ecs_tasks.id]
    assign_public_ip = false
  }
  
  load_balancer {
    target_group_arn = aws_lb_target_group.app.arn
    container_name   = "app"
    container_port   = var.container_port
  }
  
  deployment_configuration {
    maximum_percent         = 200
    minimum_healthy_percent = 100
    
    deployment_circuit_breaker {
      enable   = true
      rollback = true
    }
  }
  
  # Deployment controller
  deployment_controller {
    type = "ECS"  # or "CODE_DEPLOY" for blue/green
  }
  
  # Service discovery (optional)
  # service_registries {
  #   registry_arn = aws_service_discovery_service.app.arn
  # }
  
  tags = merge(local.common_tags, {
    Name = "${local.name_prefix}-service"
  })
  
  depends_on = [aws_lb_listener.http, aws_iam_role_policy.ecs_task_execution_additional]
  
  lifecycle {
    ignore_changes = [
      desired_count # Managed by auto scaling
    ]
  }
}
```

**14. autoscaling.tf:**

```hcl
# Auto Scaling Target
resource "aws_appautoscaling_target" "ecs" {
  count = var.enable_autoscaling ? 1 : 0
  
  max_capacity       = var.max_capacity
  min_capacity       = var.min_capacity
  resource_id        = "service/${aws_ecs_cluster.main.name}/${aws_ecs_service.app.name}"
  scalable_dimension = "ecs:service:DesiredCount"
  service_namespace  = "ecs"
  
  role_arn = aws_iam_role.ecs_autoscale.arn
}

# Scale Up Policy
resource "aws_appautoscaling_policy" "ecs_scale_up" {
  count = var.enable_autoscaling ? 1 : 0
  
  name               = "${local.name_prefix}-scale-up"
  policy_type        = "StepScaling"
  resource_id        = aws_appautoscaling_target.ecs[0].resource_id
  scalable_dimension = aws_appautoscaling_target.ecs[0].scalable_dimension
  service_namespace  = aws_appautoscaling_target.ecs[0].service_namespace
  
  step_scaling_policy_configuration {
    adjustment_type         = "ChangeInCapacity"
    cooldown                = 60
    metric_aggregation_type = "Average"
    
    step_adjustment {
      metric_interval_lower_bound = 0
      metric_interval_upper_bound = 10
      scaling_adjustment          = 1
    }
    
    step_adjustment {
      metric_interval_lower_bound = 10
      scaling_adjustment          = 2
    }
  }
}

# Scale Down Policy
resource "aws_appautoscaling_policy" "ecs_scale_down" {
  count = var.enable_autoscaling ? 1 : 0
  
  name               = "${local.name_prefix}-scale-down"
  policy_type        = "StepScaling"
  resource_id        = aws_appautoscaling_target.ecs[0].resource_id
  scalable_dimension = aws_appautoscaling_target.ecs[0].scalable_dimension
  service_namespace  = aws_appautoscaling_target.ecs[0].service_namespace
  
  step_scaling_policy_configuration {
    adjustment_type         = "ChangeInCapacity"
    cooldown                = 300
    metric_aggregation_type = "Average"
    
    step_adjustment {
      metric_interval_upper_bound = 0
      scaling_adjustment          = -1
    }
  }
}

# Target Tracking Policy (alternative to step scaling)
resource "aws_appautoscaling_policy" "ecs_target_tracking" {
  count = var.enable_autoscaling ? 1 : 0
  
  name               = "${local.name_prefix}-target-tracking"
  policy_type        = "TargetTrackingScaling"
  resource_id        = aws_appautoscaling_target.ecs[0].resource_id
  scalable_dimension = aws_appautoscaling_target.ecs[0].scalable_dimension
  service_namespace  = aws_appautoscaling_target.ecs[0].service_namespace
  
  target_tracking_scaling_policy_configuration {
    target_value       = 70.0
    scale_in_cooldown  = 300
    scale_out_cooldown = 60
    
    predefined_metric_specification {
      predefined_metric_type = "ECSServiceAverageCPUUtilization"
    }
  }
}
````

**15. outputs.tf:**

```hcl
output "vpc_id" {
  description = "VPC ID"
  value       = aws_vpc.main.id
}

output "public_subnet_ids" {
  description = "Public subnet IDs"
  value       = aws_subnet.public[*].id
}

output "private_subnet_ids" {
  description = "Private subnet IDs"
  value       = aws_subnet.private[*].id
}

output "ecs_cluster_id" {
  description = "ECS cluster ID"
  value       = aws_ecs_cluster.main.id
}

output "ecs_cluster_name" {
  description = "ECS cluster name"
  value       = aws_ecs_cluster.main.name
}

output "ecs_service_name" {
  description = "ECS service name"
  value       = aws_ecs_service.app.name
}

output "ecs_task_definition_arn" {
  description = "Task definition ARN"
  value       = aws_ecs_task_definition.app.arn
}

output "alb_dns_name" {
  description = "ALB DNS name"
  value       = aws_lb.main.dns_name
}

output "alb_zone_id" {
  description = "ALB zone ID"
  value       = aws_lb.main.zone_id
}

output "application_url" {
  description = "Application URL"
  value       = "http://${aws_lb.main.dns_name}"
}

output "cloudwatch_log_group" {
  description = "CloudWatch log group name"
  value       = aws_cloudwatch_log_group.ecs.name
}

output "task_execution_role_arn" {
  description = "Task execution role ARN"
  value       = aws_iam_role.ecs_task_execution.arn
}

output "task_role_arn" {
  description = "Task role ARN"
  value       = aws_iam_role.ecs_task.arn
}

output "ecs_exec_command" {
  description = "Command to exec into running task"
  value       = var.enable_execute_command ? "aws ecs execute-command --cluster ${aws_ecs_cluster.main.name} --task TASK_ID --container app --interactive --command /bin/sh" : "ECS Exec is disabled"
}
```

**16. terraform.tfvars:**

```hcl
project_name = "myapp"
environment  = "dev"
aws_region   = "us-east-1"

vpc_cidr           = "10.0.0.0/16"
availability_zones = ["us-east-1a", "us-east-1b", "us-east-1c"]

# Container configuration
container_image = "nginx:latest"
container_port  = 80
cpu            = "256"
memory         = "512"

# Service configuration
desired_count   = 2
min_capacity    = 2
max_capacity    = 10

enable_autoscaling     = true
enable_execute_command = true

health_check_path           = "/"
health_check_grace_period   = 60
```

**17. Makefile:**

```makefile
.PHONY: help init plan apply destroy logs exec clean

ENVIRONMENT ?= dev

help:
	@echo "ECS Deployment Commands:"
	@echo "  init       - Initialize Terraform"
	@echo "  plan       - Plan infrastructure"
	@echo "  apply      - Apply infrastructure"
	@echo "  destroy    - Destroy infrastructure"
	@echo "  logs       - View ECS task logs"
	@echo "  exec       - Execute command in container"
	@echo "  scale      - Scale ECS service"
	@echo "  clean      - Clean Terraform files"

init:
	terraform init

plan:
	terraform plan -var-file=$(ENVIRONMENT).tfvars -out=tfplan

apply:
	terraform apply tfplan

destroy:
	terraform destroy -var-file=$(ENVIRONMENT).tfvars

# View logs
logs:
	@CLUSTER=$$(terraform output -raw ecs_cluster_name); \
	SERVICE=$$(terraform output -raw ecs_service_name); \
	LOG_GROUP=$$(terraform output -raw cloudwatch_log_group); \
	aws logs tail $$LOG_GROUP --follow

# Execute command in running task
exec:
	@CLUSTER=$$(terraform output -raw ecs_cluster_name); \
	SERVICE=$$(terraform output -raw ecs_service_name); \
	TASK_ID=$$(aws ecs list-tasks --cluster $$CLUSTER --service-name $$SERVICE --query 'taskArns[0]' --output text | cut -d'/' -f3); \
	echo "Connecting to task: $$TASK_ID"; \
	aws ecs execute-command --cluster $$CLUSTER --task $$TASK_ID --container app --interactive --command /bin/sh

# Scale service
scale:
	@read -p "Enter desired count: " COUNT; \
	CLUSTER=$$(terraform output -raw ecs_cluster_name); \
	SERVICE=$$(terraform output -raw ecs_service_name); \
	aws ecs update-service --cluster $$CLUSTER --service $$SERVICE --desired-count $$COUNT

# Get service status
status:
	@CLUSTER=$$(terraform output -raw ecs_cluster_name); \
	SERVICE=$$(terraform output -raw ecs_service_name); \
	aws ecs describe-services --cluster $$CLUSTER --services $$SERVICE --query 'services[0].{Name:serviceName,Status:status,Running:runningCount,Desired:desiredCount,Pending:pendingCount}' --output table

# List tasks
tasks:
	@CLUSTER=$$(terraform output -raw ecs_cluster_name); \
	SERVICE=$$(terraform output -raw ecs_service_name); \
	aws ecs list-tasks --cluster $$CLUSTER --service-name $$SERVICE --query 'taskArns[]' --output table

clean:
	rm -rf .terraform terraform.tfstate* tfplan .terraform.lock.hcl
```

**18. Запуск и тестирование:**

```bash
# 1. Инициализация
terraform init

# 2. Валидация
terraform validate

# 3. План
terraform plan -var-file=dev.tfvars -out=tfplan

# 4. Применение
terraform apply tfplan

# 5. Получение URL приложения
APP_URL=$(terraform output -raw application_url)
echo "Application URL: $APP_URL"

# 6. Тестирование endpoint
curl $APP_URL

# 7. Просмотр логов
make logs

# 8. Проверка статуса service
make status

# 9. Список tasks
make tasks

# 10. Exec в container (если enable_execute_command=true)
make exec

# 11. Масштабирование
make scale

# 12. Cleanup
make destroy
```

---

## Часть 2: EKS (Elastic Kubernetes Service) Deployment

### 💻 Задание: Production-Ready EKS Setup

**1. Структура EKS проекта:**

```bash
mkdir terraform-eks
cd terraform-eks
mkdir -p modules/{eks,networking,addons} kubernetes manifests
```

**2. eks/provider.tf:**

```hcl
terraform {
  required_version = ">= 1.6.0"
  
  required_providers {
    aws = {
      source  = "hashicorp/aws"
      version = "~> 5.0"
    }
    kubernetes = {
      source  = "hashicorp/kubernetes"
      version = "~> 2.23"
    }
    helm = {
      source  = "hashicorp/helm"
      version = "~> 2.11"
    }
    kubectl = {
      source  = "gavinbunney/kubectl"
      version = "~> 1.14"
    }
  }
}

provider "aws" {
  region = var.aws_region
  
  default_tags {
    tags = local.common_tags
  }
}

# Kubernetes provider configuration
provider "kubernetes" {
  host                   = module.eks.cluster_endpoint
  cluster_ca_certificate = base64decode(module.eks.cluster_certificate_authority_data)
  
  exec {
    api_version = "client.authentication.k8s.io/v1beta1"
    command     = "aws"
    args = [
      "eks",
      "get-token",
      "--cluster-name",
      module.eks.cluster_name
    ]
  }
}

# Helm provider configuration
provider "helm" {
  kubernetes {
    host                   = module.eks.cluster_endpoint
    cluster_ca_certificate = base64decode(module.eks.cluster_certificate_authority_data)
    
    exec {
      api_version = "client.authentication.k8s.io/v1beta1"
      command     = "aws"
      args = [
        "eks",
        "get-token",
        "--cluster-name",
        module.eks.cluster_name
      ]
    }
  }
}

# Kubectl provider
provider "kubectl" {
  host                   = module.eks.cluster_endpoint
  cluster_ca_certificate = base64decode(module.eks.cluster_certificate_authority_data)
  load_config_file       = false
  
  exec {
    api_version = "client.authentication.k8s.io/v1beta1"
    command     = "aws"
    args = [
      "eks",
      "get-token",
      "--cluster-name",
      module.eks.cluster_name
    ]
  }
}
```

**3. eks/variables.tf:**

```hcl
variable "aws_region" {
  description = "AWS region"
  type        = string
  default     = "us-east-1"
}

variable "project_name" {
  description = "Project name"
  type        = string
  default     = "eks-demo"
}

variable "environment" {
  description = "Environment name"
  type        = string
  
  validation {
    condition     = contains(["dev", "staging", "prod"], var.environment)
    error_message = "Must be dev, staging, or prod."
  }
}

variable "vpc_cidr" {
  description = "VPC CIDR block"
  type        = string
  default     = "10.0.0.0/16"
}

variable "availability_zones" {
  description = "Availability zones"
  type        = list(string)
  default     = ["us-east-1a", "us-east-1b", "us-east-1c"]
}

variable "cluster_version" {
  description = "Kubernetes version"
  type        = string
  default     = "1.28"
}

variable "cluster_endpoint_public_access" {
  description = "Enable public API server endpoint"
  type        = bool
  default     = true
}

variable "cluster_endpoint_private_access" {
  description = "Enable private API server endpoint"
  type        = bool
  default     = true
}

variable "enable_cluster_autoscaler" {
  description = "Enable cluster autoscaler"
  type        = bool
  default     = true
}

variable "enable_metrics_server" {
  description = "Enable metrics server"
  type        = bool
  default     = true
}

variable "enable_aws_load_balancer_controller" {
  description = "Enable AWS Load Balancer Controller"
  type        = bool
  default     = true
}

variable "node_groups" {
  description = "EKS managed node group configurations"
  type = map(object({
    desired_size   = number
    min_size       = number
    max_size       = number
    instance_types = list(string)
    capacity_type  = string
    disk_size      = number
    labels         = map(string)
    taints = list(object({
      key    = string
      value  = string
      effect = string
    }))
  }))
  
  default = {
    general = {
      desired_size   = 2
      min_size       = 2
      max_size       = 10
      instance_types = ["t3.medium"]
      capacity_type  = "ON_DEMAND"
      disk_size      = 50
      labels = {
        role = "general"
      }
      taints = []
    }
  }
}

variable "enable_irsa" {
  description = "Enable IAM Roles for Service Accounts"
  type        = bool
  default     = true
}
```

**4. eks/locals.tf:**

```hcl
locals {
  name_prefix = "${var.project_name}-${var.environment}"
  
  common_tags = {
    Project     = var.project_name
    Environment = var.environment
    ManagedBy   = "Terraform"
    Service     = "EKS"
  }
  
  # Cluster tags for AWS Load Balancer Controller
  cluster_tags = merge(
    local.common_tags,
    {
      "kubernetes.io/cluster/${local.name_prefix}" = "owned"
    }
  )
}
```

**5. eks/vpc.tf:**

```hcl
# VPC для EKS
resource "aws_vpc" "main" {
  cidr_block           = var.vpc_cidr
  enable_dns_hostnames = true
  enable_dns_support   = true
  
  tags = merge(local.cluster_tags, {
    Name = "${local.name_prefix}-vpc"
  })
}

# Public Subnets (для Load Balancers)
resource "aws_subnet" "public" {
  count = length(var.availability_zones)
  
  vpc_id                  = aws_vpc.main.id
  cidr_block              = cidrsubnet(var.vpc_cidr, 8, count.index)
  availability_zone       = var.availability_zones[count.index]
  map_public_ip_on_launch = true
  
  tags = merge(local.cluster_tags, {
    Name                                        = "${local.name_prefix}-public-${var.availability_zones[count.index]}"
    "kubernetes.io/role/elb"                    = "1"
    "kubernetes.io/cluster/${local.name_prefix}" = "shared"
  })
}

# Private Subnets (для Nodes)
resource "aws_subnet" "private" {
  count = length(var.availability_zones)
  
  vpc_id            = aws_vpc.main.id
  cidr_block        = cidrsubnet(var.vpc_cidr, 8, count.index + 10)
  availability_zone = var.availability_zones[count.index]
  
  tags = merge(local.cluster_tags, {
    Name                                        = "${local.name_prefix}-private-${var.availability_zones[count.index]}"
    "kubernetes.io/role/internal-elb"           = "1"
    "kubernetes.io/cluster/${local.name_prefix}" = "shared"
  })
}

# Internet Gateway
resource "aws_internet_gateway" "main" {
  vpc_id = aws_vpc.main.id
  
  tags = merge(local.common_tags, {
    Name = "${local.name_prefix}-igw"
  })
}

# Elastic IPs для NAT Gateways
resource "aws_eip" "nat" {
  count = length(var.availability_zones)
  
  domain = "vpc"
  
  tags = merge(local.common_tags, {
    Name = "${local.name_prefix}-nat-eip-${count.index + 1}"
  })
  
  depends_on = [aws_internet_gateway.main]
}

# NAT Gateways
resource "aws_nat_gateway" "main" {
  count = length(var.availability_zones)
  
  allocation_id = aws_eip.nat[count.index].id
  subnet_id     = aws_subnet.public[count.index].id
  
  tags = merge(local.common_tags, {
    Name = "${local.name_prefix}-nat-${count.index + 1}"
  })
  
  depends_on = [aws_internet_gateway.main]
}

# Public Route Table
resource "aws_route_table" "public" {
  vpc_id = aws_vpc.main.id
  
  route {
    cidr_block = "0.0.0.0/0"
    gateway_id = aws_internet_gateway.main.id
  }
  
  tags = merge(local.common_tags, {
    Name = "${local.name_prefix}-public-rt"
  })
}

# Private Route Tables
resource "aws_route_table" "private" {
  count = length(var.availability_zones)
  
  vpc_id = aws_vpc.main.id
  
  route {
    cidr_block     = "0.0.0.0/0"
    nat_gateway_id = aws_nat_gateway.main[count.index].id
  }
  
  tags = merge(local.common_tags, {
    Name = "${local.name_prefix}-private-rt-${count.index + 1}"
  })
}

# Route Table Associations
resource "aws_route_table_association" "public" {
  count = length(aws_subnet.public)
  
  subnet_id      = aws_subnet.public[count.index].id
  route_table_id = aws_route_table.public.id
}

resource "aws_route_table_association" "private" {
  count = length(aws_subnet.private)
  
  subnet_id      = aws_subnet.private[count.index].id
  route_table_id = aws_route_table.private[count.index].id
}
```

**6. eks/eks-cluster.tf:**

```hcl
# EKS Cluster
resource "aws_eks_cluster" "main" {
  name     = local.name_prefix
  role_arn = aws_iam_role.cluster.arn
  version  = var.cluster_version
  
  vpc_config {
    subnet_ids              = concat(aws_subnet.private[*].id, aws_subnet.public[*].id)
    endpoint_private_access = var.cluster_endpoint_private_access
    endpoint_public_access  = var.cluster_endpoint_public_access
    
    # Public access CIDRs (restrict in production)
    # public_access_cidrs = ["0.0.0.0/0"]
  }
  
  # Enable control plane logging
  enabled_cluster_log_types = ["api", "audit", "authenticator", "controllerManager", "scheduler"]
  
  # Encryption config
  encryption_config {
    provider {
      key_arn = aws_kms_key.eks.arn
    }
    resources = ["secrets"]
  }
  
  tags = merge(local.common_tags, {
    Name = "${local.name_prefix}-cluster"
  })
  
  depends_on = [
    aws_iam_role_policy_attachment.cluster_policy,
    aws_iam_role_policy_attachment.vpc_resource_controller,
    aws_cloudwatch_log_group.cluster
  ]
}

# CloudWatch Log Group for EKS
resource "aws_cloudwatch_log_group" "cluster" {
  name              = "/aws/eks/${local.name_prefix}/cluster"
  retention_in_days = 7
  
  tags = local.common_tags
}

# KMS Key for EKS encryption
resource "aws_kms_key" "eks" {
  description             = "EKS Secret Encryption Key"
  deletion_window_in_days = 7
  enable_key_rotation     = true
  
  tags = merge(local.common_tags, {
    Name = "${local.name_prefix}-eks-key"
  })
}

resource "aws_kms_alias" "eks" {
  name          = "alias/${local.name_prefix}-eks"
  target_key_id = aws_kms_key.eks.key_id
}
```

**7. eks/eks-node-groups.tf:**

```hcl
# EKS Managed Node Groups
resource "aws_eks_node_group" "main" {
  for_each = var.node_groups
  
  cluster_name    = aws_eks_cluster.main.name
  node_group_name = "${local.name_prefix}-${each.key}"
  node_role_arn   = aws_iam_role.node_group.arn
  subnet_ids      = aws_subnet.private[*].id
  
  instance_types = each.value.instance_types
  capacity_type  = each.value.capacity_type
  disk_size      = each.value.disk_size
  
  scaling_config {
    desired_size = each.value.desired_size
    min_size     = each.value.min_size
    max_size     = each.value.max_size
  }
  
  update_config {
    max_unavailable_percentage = 33
  }
  
  labels = merge(
    each.value.labels,
    {
      Environment = var.environment
      NodeGroup   = each.key
    }
  )
  
  # Taints
  dynamic "taint" {
    for_each = each.value.taints
    content {
      key    = taint.value.key
      value  = taint.value.value
      effect = taint.value.effect
    }
  }
  
  # Launch template for advanced configuration
  launch_template {
    id      = aws_launch_template.node_group[each.key].id
    version = aws_launch_template.node_group[each.key].latest_version
  }
  
  tags = merge(local.common_tags, {
    Name      = "${local.name_prefix}-${each.key}-node-group"
    NodeGroup = each.key
  })
  
  depends_on = [
    aws_iam_role_policy_attachment.node_group_policy,
    aws_iam_role_policy_attachment.cni_policy,
    aws_iam_role_policy_attachment.container_registry_policy
  ]
  
  lifecycle {
    create_before_destroy = true
    ignore_changes        = [scaling_config[0].desired_size]
  }
}

# Launch Template for Node Groups
resource "aws_launch_template" "node_group" {
  for_each = var.node_groups
  
  name_prefix = "${local.name_prefix}-${each.key}-"
  
  block_device_mappings {
    device_name = "/dev/xvda"
    
    ebs {
      volume_size           = each.value.disk_size
      volume_type           = "gp3"
      encrypted             = true
      kms_key_id            = aws_kms_key.eks.arn
      delete_on_termination = true
    }
  }
  
  metadata_options {
    http_endpoint               = "enabled"
    http_tokens                 = "required"
    http_put_response_hop_limit = 1
    instance_metadata_tags      = "enabled"
  }
  
  monitoring {
    enabled = true
  }
  
  tag_specifications {
    resource_type = "instance"
    tags = merge(local.common_tags, {
      Name      = "${local.name_prefix}-${each.key}-node"
      NodeGroup = each.key
    })
  }
  
  tag_specifications {
    resource_type = "volume"
    tags = merge(local.common_tags, {
      Name      = "${local.name_prefix}-${each.key}-volume"
      NodeGroup = each.key
    })
  }
  
  user_data = base64encode(templatefile("${path.module}/userdata.sh", {
    cluster_name        = aws_eks_cluster.main.name
    cluster_endpoint    = aws_eks_cluster.main.endpoint
    cluster_ca          = aws_eks_cluster.main.certificate_authority[0].data
    node_group_name     = each.key
  }))
  
  tags = merge(local.common_tags, {
    Name = "${local.name_prefix}-${each.key}-lt"
  })
}
```

**8. eks/userdata.sh:**

```bash
#!/bin/bash
set -ex

# Bootstrap EKS node
/etc/eks/bootstrap.sh ${cluster_name} \
  --b64-cluster-ca ${cluster_ca} \
  --apiserver-endpoint ${cluster_endpoint} \
  --kubelet-extra-args '--node-labels=node-group=${node_group_name}'

# Additional customizations
echo "Node bootstrapped successfully"
```

**9. eks/iam.tf:**

```hcl
# ================================================
# EKS Cluster IAM Role
# ================================================

resource "aws_iam_role" "cluster" {
  name_prefix = "${local.name_prefix}-cluster-"
  
  assume_role_policy = jsonencode({
    Version = "2012-10-17"
    Statement = [{
      Effect = "Allow"
      Principal = {
        Service = "eks.amazonaws.com"
      }
      Action = "sts:AssumeRole"
    }]
  })
  
  tags = local.common_tags
}

resource "aws_iam_role_policy_attachment" "cluster_policy" {
  role       = aws_iam_role.cluster.name
  policy_arn = "arn:aws:iam::aws:policy/AmazonEKSClusterPolicy"
}

resource "aws_iam_role_policy_attachment" "vpc_resource_controller" {
  role       = aws_iam_role.cluster.name
  policy_arn = "arn:aws:iam::aws:policy/AmazonEKSVPCResourceController"
}

# ================================================
# EKS Node Group IAM Role
# ================================================

resource "aws_iam_role" "node_group" {
  name_prefix = "${local.name_prefix}-node-group-"
  
  assume_role_policy = jsonencode({
    Version = "2012-10-17"
    Statement = [{
      Effect = "Allow"
      Principal = {
        Service = "ec2.amazonaws.com"
      }
      Action = "sts:AssumeRole"
    }]
  })
  
  tags = local.common_tags
}

resource "aws_iam_role_policy_attachment" "node_group_policy" {
  role       = aws_iam_role.node_group.name
  policy_arn = "arn:aws:iam::aws:policy/AmazonEKSWorkerNodePolicy"
}

resource "aws_iam_role_policy_attachment" "cni_policy" {
  role       = aws_iam_role.node_group.name
  policy_arn = "arn:aws:iam::aws:policy/AmazonEKS_CNI_Policy"
}

resource "aws_iam_role_policy_attachment" "container_registry_policy" {
  role       = aws_iam_role.node_group.name
  policy_arn = "arn:aws:iam::aws:policy/AmazonEC2ContainerRegistryReadOnly"
}

# Additional permissions for SSM (debugging)
resource "aws_iam_role_policy_attachment" "ssm_managed_instance" {
  role       = aws_iam_role.node_group.name
  policy_arn = "arn:aws:iam::aws:policy/AmazonSSMManagedInstanceCore"
}

# ================================================
# IRSA (IAM Roles for Service Accounts)
# ================================================

# OIDC Provider for IRSA
data "tls_certificate" "cluster" {
  url = aws_eks_cluster.main.identity[0].oidc[0].issuer
}

resource "aws_iam_openid_connect_provider" "cluster" {
  count = var.enable_irsa ? 1 : 0
  
  client_id_list  = ["sts.amazonaws.com"]
  thumbprint_list = [data.tls_certificate.cluster.certificates[0].sha1_fingerprint]
  url             = aws_eks_cluster.main.identity[0].oidc[0].issuer
  
  tags = local.common_tags
}

# ================================================
# AWS Load Balancer Controller IAM Role
# ================================================

module "lb_controller_irsa" {
  source  = "terraform-aws-modules/iam/aws//modules/iam-role-for-service-accounts-eks"
  version = "~> 5.0"
  
  count = var.enable_aws_load_balancer_controller ? 1 : 0
  
  role_name = "${local.name_prefix}-lb-controller"
  
  attach_load_balancer_controller_policy = true
  
  oidc_providers = {
    main = {
      provider_arn               = aws_iam_openid_connect_provider.cluster[0].arn
      namespace_service_accounts = ["kube-system:aws-load-balancer-controller"]
    }
  }
  
  tags = local.common_tags
}

# ================================================
# Cluster Autoscaler IAM Role
# ================================================

module "cluster_autoscaler_irsa" {
  source  = "terraform-aws-modules/iam/aws//modules/iam-role-for-service-accounts-eks"
  version = "~> 5.0"
  
  count = var.enable_cluster_autoscaler ? 1 : 0
  
  role_name = "${local.name_prefix}-cluster-autoscaler"
  
  attach_cluster_autoscaler_policy = true
  cluster_autoscaler_cluster_names = [aws_eks_cluster.main.name]
  
  oidc_providers = {
    main = {
      provider_arn               = aws_iam_openid_connect_provider.cluster[0].arn
      namespace_service_accounts = ["kube-system:cluster-autoscaler"]
    }
  }
  
  tags = local.common_tags
}
```

**10. eks/addons.tf:**

```hcl
# EKS Addons

# VPC CNI
resource "aws_eks_addon" "vpc_cni" {
  cluster_name             = aws_eks_cluster.main.name
  addon_name               = "vpc-cni"
  addon_version            = data.aws_eks_addon_version.vpc_cni.version
  resolve_conflicts_on_update = "PRESERVE"
  
  tags = local.common_tags
}

# CoreDNS
resource "aws_eks_addon" "coredns" {
  cluster_name             = aws_eks_cluster.main.name
  addon_name               = "coredns"
  addon_version            = data.aws_eks_addon_version.coredns.version
  resolve_conflicts_on_update = "PRESERVE"
  
  tags = local.common_tags
  
  depends_on = [aws_eks_node_group.main]
}

# kube-proxy
resource "aws_eks_addon" "kube_proxy" {
  cluster_name             = aws_eks_cluster.main.name
  addon_name               = "kube-proxy"
  addon_version            = data.aws_eks_addon_version.kube_proxy.version
  resolve_conflicts_on_update = "PRESERVE"
  
  tags = local.common_tags
}

# EBS CSI Driver
resource "aws_eks_addon" "ebs_csi_driver" {
  cluster_name             = aws_eks_cluster.main.name
  addon_name               = "aws-ebs-csi-driver"
  addon_version            = data.aws_eks_addon_version.ebs_csi.version
  service_account_role_arn = module.ebs_csi_irsa[0].iam_role_arn
  resolve_conflicts_on_update = "PRESERVE"
  
  tags = local.common_tags
}

# Get latest addon versions
data "aws_eks_addon_version" "vpc_cni" {
  addon_name         = "vpc-cni"
  kubernetes_version = aws_eks_cluster.main.version
  most_recent        = true
}

data "aws_eks_addon_version" "coredns" {
  addon_name         = "coredns"
  kubernetes_version = aws_eks_cluster.main.version
  most_recent        = true
}

data "aws_eks_addon_version" "kube_proxy" {
  addon_name         = "kube-proxy"
  kubernetes_version = aws_eks_cluster.main.version
  most_recent        = true
}

data "aws_eks_addon_version" "ebs_csi" {
  addon_name         = "aws-ebs-csi-driver"
  kubernetes_version = aws_eks_cluster.main.version
  most_recent        = true
}

# EBS CSI Driver IRSA
module "ebs_csi_irsa" {
  source  = "terraform-aws-modules/iam/aws//modules/iam-role-for-service-accounts-eks"
  version = "~> 5.0"
  
  count = 1
  
  role_name = "${local.name_prefix}-ebs-csi-driver"
  
  attach_ebs_csi_policy = true
  
  oidc_providers = {
    main = {
      provider_arn               = aws_iam_openid_connect_provider.cluster[0].arn
      namespace_service_accounts = ["kube-system:ebs-csi-controller-sa"]
    }
  }
  
  tags = local.common_tags
}
```

**11. eks/helm-releases.tf:**

```hcl
# Metrics Server
resource "helm_release" "metrics_server" {
  count = var.enable_metrics_server ? 1 : 0
  
  name       = "metrics-server"
  repository = "https://kubernetes-sigs.github.io/metrics-server/"
  chart      = "metrics-server"
  namespace  = "kube-system"
  version    = "3.11.0"
  
  set {
    name  = "args[0]"
    value = "--kubelet-insecure-tls"
  }
  
  depends_on = [aws_eks_node_group.main]
}

# AWS Load Balancer Controller
resource "helm_release" "aws_load_balancer_controller" {
  count = var.enable_aws_load_balancer_controller ? 1 : 0
  
  name       = "aws-load-balancer-controller"
  repository = "https://aws.github.io/eks-charts"
  chart      = "aws-load-balancer-controller"
  namespace  = "kube-system"
  version    = "1.6.2"
  
  set {
    name  = "clusterName"
    value = aws_eks_cluster.main.name
  }
  
  set {
    name  = "serviceAccount.create"
    value = "true"
  }
  
  set {
    name  = "serviceAccount.name"
    value = "aws-load-balancer-controller"
  }
  
  set {
    name  = "serviceAccount.annotations.eks\\.amazonaws\\.com/role-arn"
    value = module.lb_controller_irsa[0].iam_role_arn
  }
  
  depends_on = [aws_eks_node_group.main]
}

# Cluster Autoscaler
resource "helm_release" "cluster_autoscaler" {
  count = var.enable_cluster_autoscaler ? 1 : 0
  
  name       = "cluster-autoscaler"
  repository = "https://kubernetes.github.io/autoscaler"
  chart      = "cluster-autoscaler"
  namespace  = "kube-system"
  version    = "9.29.3"
  
  set {
    name  = "autoDiscovery.clusterName"
    value = aws_eks_cluster.main.name
  }
  
  set {
    name  = "awsRegion"
    value = var.aws_region
  }
  
  set {
    name  = "rbac.serviceAccount.create"
    value = "true"
  }
  
  set {
    name  = "rbac.serviceAccount.name"
    value = "cluster-autoscaler"
  }
  
  set {
    name  = "rbac.serviceAccount.annotations.eks\\.amazonaws\\.com/role-arn"
    value = module.cluster_autoscaler_irsa[0].iam_role_arn
  }
  
  depends_on = [aws_eks_node_group.main]
}
```

**12. eks/outputs.tf:**

```hcl
output "cluster_id" {
  description = "EKS cluster ID"
  value       = aws_eks_cluster.main.id
}

output "cluster_name" {
  description = "EKS cluster name"
  value       = aws_eks_cluster.main.name
}

output "cluster_endpoint" {
  description = "EKS cluster endpoint"
  value       = aws_eks_cluster.main.endpoint
}

output "cluster_version" {
  description = "EKS cluster version"
  value       = aws_eks_cluster.main.version
}

output "cluster_certificate_authority_data" {
  description = "EKS cluster certificate authority data"
  value       = aws_eks_cluster.main.certificate_authority[0].data
  sensitive   = true
}

output "cluster_arn" {
  description = "EKS cluster ARN"
  value       = aws_eks_cluster.main.arn
}

output "cluster_oidc_issuer_url" {
  description = "OIDC issuer URL"
  value       = try(aws_iam_openid_connect_provider.cluster[0].url, "")
}

output "node_groups" {
  description = "EKS node groups"
  value = {
    for k, v in aws_eks_node_group.main :
    k => {
      id     = v.id
      status = v.status
      arn    = v.arn
    }
  }
}

output "vpc_id" {
  description = "VPC ID"
  value       = aws_vpc.main.id
}

output "private_subnet_ids" {
  description = "Private subnet IDs"
  value       = aws_subnet.private[*].id
}

output "public_subnet_ids" {
  description = "Public subnet IDs"
  value       = aws_subnet.public[*].id
}

output "configure_kubectl" {
  description = "Command to configure kubectl"
  value       = "aws eks update-kubeconfig --region ${var.aws_region} --name ${aws_eks_cluster.main.name}"
}
```

**13. manifests/sample-app.yaml:**

```yaml
---
apiVersion: v1
kind: Namespace
metadata:
  name: demo
  labels:
    name: demo
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx
  namespace: demo
spec:
  replicas: 3
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
        - name: nginx
          image: nginx:latest
          ports:
            - containerPort: 80
          resources:
            requests:
              memory: "64Mi"
              cpu: "250m"
            limits:
              memory: "128Mi"
              cpu: "500m"
          livenessProbe:
            httpGet:
              path: /
              port: 80
            initialDelaySeconds: 30
            periodSeconds: 10
          readinessProbe:
            httpGet:
              path: /
              port: 80
            initialDelaySeconds: 5
            periodSeconds: 5
---
apiVersion: v1
kind: Service
metadata:
  name: nginx
  namespace: demo
spec:
  type: LoadBalancer
  selector:
    app: nginx
  ports:
    - port: 80
      targetPort: 80
      protocol: TCP
---
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: nginx
  namespace: demo
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: nginx
  minReplicas: 2
  maxReplicas: 10
  metrics:
    - type: Resource
      resource:
        name: cpu
        target:
          type: Utilization
          averageUtilization: 70
    - type: Resource
      resource:
        name: memory
        target:
          type: Utilization
          averageUtilization: 80

````

**14. Makefile:**

```makefile
.PHONY: help init plan apply destroy kubeconfig deploy clean

ENVIRONMENT ?= dev

help:
	@echo "EKS Deployment Commands:"
	@echo "  init       - Initialize Terraform"
	@echo "  plan       - Plan infrastructure"
	@echo "  apply      - Apply infrastructure"
	@echo "  destroy    - Destroy infrastructure"
	@echo "  kubeconfig - Configure kubectl"
	@echo "  deploy     - Deploy sample application"
	@echo "  clean      - Clean Terraform files"

init:
	terraform init

plan:
	terraform plan -var-file=$(ENVIRONMENT).tfvars -out=tfplan

apply:
	terraform apply tfplan

destroy:
	terraform destroy -var-file=$(ENVIRONMENT).tfvars

kubeconfig:
	@aws eks update-kubeconfig \
		--region $$(terraform output -raw aws_region) \
		--name $$(terraform output -raw cluster_name)
	@echo "kubectl configured for cluster: $$(terraform output -raw cluster_name)"

deploy:
	kubectl apply -f manifests/sample-app.yaml
	@echo "Sample app deployed. Waiting for LoadBalancer..."
	@sleep 10
	@kubectl get svc -n demo nginx

nodes:
	kubectl get nodes -o wide

pods:
	kubectl get pods -A

clean:
	rm -rf .terraform terraform.tfstate* tfplan .terraform.lock.hcl
````

**15. Запуск и тестирование:**

```bash
# 1. Инициализация
terraform init

# 2. План
terraform plan -var-file=dev.tfvars -out=tfplan

# 3. Применение
terraform apply tfplan

# 4. Настройка kubectl
aws eks update-kubeconfig --region us-east-1 --name eks-demo-dev

# 5. Проверка кластера
kubectl get nodes
kubectl get pods -A

# 6. Deploy sample app
kubectl apply -f manifests/sample-app.yaml

# 7. Проверка приложения
kubectl get pods -n demo
kubectl get svc -n demo

# 8. Получить LoadBalancer URL
LB_URL=$(kubectl get svc nginx -n demo -o jsonpath='{.status.loadBalancer.ingress[0].hostname}')
echo "Application URL: http://$LB_URL"

# 9. Тест приложения
curl http://$LB_URL

# 10. Cleanup
terraform destroy -var-file=dev.tfvars
```

---

## 🎯 Сравнение ECS vs EKS

|Feature|ECS Fargate|EKS|
|---|---|---|
|**Управление**|AWS-managed|Kubernetes standard|
|**Сложность**|Простая|Средняя/Высокая|
|**Стоимость**|Lower overhead|Higher (control plane cost)|
|**Портабельность**|AWS-only|Multi-cloud ready|
|**Экосистема**|AWS-specific|Kubernetes ecosystem|
|**Масштабирование**|Auto-scaling встроен|Требует настройки|
|**Networking**|AWS VPC|CNI plugins|
|**Мониторинг**|CloudWatch|Prometheus/Grafana|
|**Best For**|AWS-native apps|Kubernetes workloads|

---

## 🚀 Бонус: Blue/Green Deployment

**1. Blue/Green для ECS:**

```hcl
# blue-green-ecs.tf

resource "aws_codedeploy_app" "ecs" {
  name             = "${local.name_prefix}-ecs"
  compute_platform = "ECS"
}

resource "aws_codedeploy_deployment_group" "ecs" {
  app_name               = aws_codedeploy_app.ecs.name
  deployment_group_name  = "${local.name_prefix}-dg"
  service_role_arn       = aws_iam_role.codedeploy.arn
  deployment_config_name = "CodeDeployDefault.ECSAllAtOnce"
  
  auto_rollback_configuration {
    enabled = true
    events  = ["DEPLOYMENT_FAILURE"]
  }
  
  blue_green_deployment_config {
    terminate_blue_instances_on_deployment_success {
      action                           = "TERMINATE"
      termination_wait_time_in_minutes = 5
    }
    
    deployment_ready_option {
      action_on_timeout = "CONTINUE_DEPLOYMENT"
    }
  }
  
  deployment_style {
    deployment_option = "WITH_TRAFFIC_CONTROL"
    deployment_type   = "BLUE_GREEN"
  }
  
  ecs_service {
    cluster_name = aws_ecs_cluster.main.name
    service_name = aws_ecs_service.app.name
  }
  
  load_balancer_info {
    target_group_pair_info {
      prod_traffic_route {
        listener_arns = [aws_lb_listener.http.arn]
      }
      
      target_group {
        name = aws_lb_target_group.blue.name
      }
      
      target_group {
        name = aws_lb_target_group.green.name
      }
    }
  }
}
```

**2. Blue/Green для EKS с Argo Rollouts:**

```yaml
# manifests/rollout.yaml
apiVersion: argoproj.io/v1alpha1
kind: Rollout
metadata:
  name: nginx-rollout
  namespace: demo
spec:
  replicas: 5
  strategy:
    blueGreen:
      activeService: nginx-active
      previewService: nginx-preview
      autoPromotionEnabled: false
      scaleDownDelaySeconds: 30
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx:latest
        ports:
        - containerPort: 80
```

---

## 📊 Заключение Модуля 16

### 📚 Ключевые takeaways

**1. ECS Fargate:**

```
Преимущества:
✅ Serverless - не нужно управлять EC2
✅ Простая интеграция с AWS
✅ Быстрый старт
✅ Меньше overhead
✅ Pay-per-use pricing

Недостатки:
❌ AWS-only (vendor lock-in)
❌ Меньше гибкости
❌ Ограниченная экосистема
```

**2. EKS:**

```
Преимущества:
✅ Kubernetes standard
✅ Multi-cloud portability
✅ Богатая экосистема
✅ Advanced features (service mesh, etc)
✅ Community support

Недостатки:
❌ Более сложная настройка
❌ Выше стоимость ($0.10/час за control plane)
❌ Требует Kubernetes expertise
```

**3. Когда использовать что:**

```
Используй ECS Fargate если:
- AWS-native приложение
- Нужна простота
- Малая/средняя команда
- Быстрый time-to-market

Используй EKS если:
- Multi-cloud strategy
- Kubernetes expertise в команде
- Сложные workloads
- Нужна портабельность
- Используете Kubernetes ecosystem
```

**4. Production Checklist:**

```
ECS:
□ VPC с private subnets
□ NAT Gateways для outbound
□ ALB health checks
□ Auto scaling настроен
□ CloudWatch logs
□ IAM roles с least privilege
□ Secrets в Secrets Manager
□ Blue/Green deployment (опционально)

EKS:
□ Private node groups
□ IRSA enabled
□ Cluster autoscaler
□ Metrics server
□ AWS Load Balancer Controller
□ EBS CSI driver
□ Pod Security Standards
□ Network policies
□ Monitoring (Prometheus/Grafana)
□ Logging (FluentBit/CloudWatch)
```

**5. Best Practices:**

```hcl
# ✅ DO: Use Fargate for stateless workloads
resource "aws_ecs_service" "app" {
  launch_type = "FARGATE"
}

# ✅ DO: Enable container insights
setting {
  name  = "containerInsights"
  value = "enabled"
}

# ✅ DO: Use private subnets for tasks
network_configuration {
  subnets         = var.private_subnet_ids
  assign_public_ip = false
}

# ✅ DO: Enable ECS Exec for debugging
enable_execute_command = true

# ❌ DON'T: Hardcode sensitive data
# ❌ DON'T: Use public subnets for workloads
# ❌ DON'T: Skip health checks
# ❌ DON'T: Ignore auto scaling
```

Ты освоил:

- ✅ ECS Fargate полный setup
- ✅ EKS cluster с managed node groups
- ✅ Auto scaling для обеих платформ
- ✅ Load balancing конфигурация
- ✅ IAM roles и IRSA
- ✅ CloudWatch logging
- ✅ Blue/Green deployment patterns
- ✅ Production-ready configurations

**Следующие шаги:**

1. Service Mesh (Istio/App Mesh)
2. GitOps (ArgoCD/Flux)
3. Advanced monitoring (Prometheus/Grafana)
4. Cost optimization
5. Multi-region deployment

---

🎉 **Модуль 16 завершен!** Ты теперь знаешь как deploy containerized applications на AWS используя как ECS Fargate, так и EKS!


# Модуль 17: Final Production Project - Three-Tier Application (60 минут)

## 🎯 Цель модуля

Создать **production-ready** three-tier приложение с полным CI/CD pipeline, мониторингом и документацией.

**Что будем строить:**

```
Production Three-Tier Application
├── Presentation Tier (Web)
│   ├── CloudFront + S3 (Static Frontend)
│   └── ALB + ECS Fargate (API Gateway)
├── Application Tier (App)
│   ├── ECS Fargate (Backend Services)
│   ├── Lambda (Serverless Functions)
│   └── API Gateway (REST API)
├── Data Tier (Database)
│   ├── RDS Aurora (Primary Database)
│   ├── DynamoDB (NoSQL Cache)
│   ├── ElastiCache Redis (Session Store)
│   └── S3 (Object Storage)
└── Supporting Services
    ├── Route53 (DNS)
    ├── ACM (SSL Certificates)
    ├── WAF (Security)
    ├── CloudWatch (Monitoring)
    ├── SNS/SES (Notifications)
    └── Secrets Manager (Credentials)
```

---

## Часть 1: Структура проекта

### 💻 Задание: Создание структуры

```bash
mkdir production-three-tier-app
cd production-three-tier-app

# Создать структуру
mkdir -p {terraform,docker,kubernetes,scripts,docs}

# Terraform structure
mkdir -p terraform/{modules,environments}
mkdir -p terraform/modules/{networking,compute,database,storage,security,monitoring}
mkdir -p terraform/environments/{dev,staging,prod}

# Docker structure
mkdir -p docker/{frontend,backend,nginx}

# Scripts
mkdir -p scripts/{deploy,backup,monitoring}

# Documentation
mkdir -p docs/{architecture,runbooks,api}
```

**Итоговая структура:**

```
production-three-tier-app/
├── terraform/
│   ├── modules/
│   │   ├── networking/      # VPC, Subnets, NAT, etc
│   │   ├── compute/         # ECS, Lambda, Auto Scaling
│   │   ├── database/        # RDS, DynamoDB, Redis
│   │   ├── storage/         # S3, CloudFront
│   │   ├── security/        # WAF, Security Groups, IAM
│   │   └── monitoring/      # CloudWatch, Alarms, Dashboards
│   └── environments/
│       ├── dev/
│       ├── staging/
│       └── prod/
├── docker/
│   ├── frontend/
│   ├── backend/
│   └── nginx/
├── kubernetes/
│   └── manifests/
├── scripts/
│   ├── deploy/
│   ├── backup/
│   └── monitoring/
├── docs/
│   ├── architecture/
│   ├── runbooks/
│   └── api/
├── .github/
│   └── workflows/
└── README.md
```

---

## Часть 2: Terraform Modules

### Module 1: Networking

**terraform/modules/networking/main.tf:**

```hcl
# VPC
resource "aws_vpc" "main" {
  cidr_block           = var.vpc_cidr
  enable_dns_hostnames = true
  enable_dns_support   = true
  
  tags = merge(var.tags, {
    Name = "${var.name_prefix}-vpc"
  })
}

# Internet Gateway
resource "aws_internet_gateway" "main" {
  vpc_id = aws_vpc.main.id
  
  tags = merge(var.tags, {
    Name = "${var.name_prefix}-igw"
  })
}

# Public Subnets
resource "aws_subnet" "public" {
  count = length(var.public_subnet_cidrs)
  
  vpc_id                  = aws_vpc.main.id
  cidr_block              = var.public_subnet_cidrs[count.index]
  availability_zone       = var.availability_zones[count.index]
  map_public_ip_on_launch = true
  
  tags = merge(var.tags, {
    Name = "${var.name_prefix}-public-${var.availability_zones[count.index]}"
    Tier = "Public"
  })
}

# Private Subnets (Application)
resource "aws_subnet" "private_app" {
  count = length(var.private_app_subnet_cidrs)
  
  vpc_id            = aws_vpc.main.id
  cidr_block        = var.private_app_subnet_cidrs[count.index]
  availability_zone = var.availability_zones[count.index]
  
  tags = merge(var.tags, {
    Name = "${var.name_prefix}-private-app-${var.availability_zones[count.index]}"
    Tier = "Application"
  })
}

# Private Subnets (Database)
resource "aws_subnet" "private_db" {
  count = length(var.private_db_subnet_cidrs)
  
  vpc_id            = aws_vpc.main.id
  cidr_block        = var.private_db_subnet_cidrs[count.index]
  availability_zone = var.availability_zones[count.index]
  
  tags = merge(var.tags, {
    Name = "${var.name_prefix}-private-db-${var.availability_zones[count.index]}"
    Tier = "Database"
  })
}

# NAT Gateways
resource "aws_eip" "nat" {
  count = var.enable_nat_gateway ? length(var.availability_zones) : 0
  
  domain = "vpc"
  
  tags = merge(var.tags, {
    Name = "${var.name_prefix}-nat-eip-${count.index + 1}"
  })
  
  depends_on = [aws_internet_gateway.main]
}

resource "aws_nat_gateway" "main" {
  count = var.enable_nat_gateway ? length(var.availability_zones) : 0
  
  allocation_id = aws_eip.nat[count.index].id
  subnet_id     = aws_subnet.public[count.index].id
  
  tags = merge(var.tags, {
    Name = "${var.name_prefix}-nat-${count.index + 1}"
  })
}

# Route Tables
resource "aws_route_table" "public" {
  vpc_id = aws_vpc.main.id
  
  route {
    cidr_block = "0.0.0.0/0"
    gateway_id = aws_internet_gateway.main.id
  }
  
  tags = merge(var.tags, {
    Name = "${var.name_prefix}-public-rt"
  })
}

resource "aws_route_table" "private_app" {
  count = var.enable_nat_gateway ? length(var.availability_zones) : 0
  
  vpc_id = aws_vpc.main.id
  
  route {
    cidr_block     = "0.0.0.0/0"
    nat_gateway_id = aws_nat_gateway.main[count.index].id
  }
  
  tags = merge(var.tags, {
    Name = "${var.name_prefix}-private-app-rt-${count.index + 1}"
  })
}

resource "aws_route_table" "private_db" {
  vpc_id = aws_vpc.main.id
  
  tags = merge(var.tags, {
    Name = "${var.name_prefix}-private-db-rt"
  })
}

# Route Table Associations
resource "aws_route_table_association" "public" {
  count = length(aws_subnet.public)
  
  subnet_id      = aws_subnet.public[count.index].id
  route_table_id = aws_route_table.public.id
}

resource "aws_route_table_association" "private_app" {
  count = length(aws_subnet.private_app)
  
  subnet_id      = aws_subnet.private_app[count.index].id
  route_table_id = aws_route_table.private_app[count.index].id
}

resource "aws_route_table_association" "private_db" {
  count = length(aws_subnet.private_db)
  
  subnet_id      = aws_subnet.private_db[count.index].id
  route_table_id = aws_route_table.private_db.id
}

# VPC Flow Logs
resource "aws_flow_log" "main" {
  count = var.enable_flow_logs ? 1 : 0
  
  iam_role_arn    = aws_iam_role.flow_logs[0].arn
  log_destination = aws_cloudwatch_log_group.flow_logs[0].arn
  traffic_type    = "ALL"
  vpc_id          = aws_vpc.main.id
  
  tags = var.tags
}

resource "aws_cloudwatch_log_group" "flow_logs" {
  count = var.enable_flow_logs ? 1 : 0
  
  name              = "/aws/vpc/${var.name_prefix}"
  retention_in_days = 7
  
  tags = var.tags
}

resource "aws_iam_role" "flow_logs" {
  count = var.enable_flow_logs ? 1 : 0
  
  name_prefix = "${var.name_prefix}-flow-logs-"
  
  assume_role_policy = jsonencode({
    Version = "2012-10-17"
    Statement = [{
      Effect = "Allow"
      Principal = {
        Service = "vpc-flow-logs.amazonaws.com"
      }
      Action = "sts:AssumeRole"
    }]
  })
}

resource "aws_iam_role_policy" "flow_logs" {
  count = var.enable_flow_logs ? 1 : 0
  
  name_prefix = "${var.name_prefix}-flow-logs-"
  role        = aws_iam_role.flow_logs[0].id
  
  policy = jsonencode({
    Version = "2012-10-17"
    Statement = [
      {
        Effect = "Allow"
        Action = [
          "logs:CreateLogGroup",
          "logs:CreateLogStream",
          "logs:PutLogEvents",
          "logs:DescribeLogGroups",
          "logs:DescribeLogStreams"
        ]
        Resource = "*"
      }
    ]
  })
}
```

**terraform/modules/networking/variables.tf:**

```hcl
variable "name_prefix" {
  description = "Prefix for resource names"
  type        = string
}

variable "vpc_cidr" {
  description = "VPC CIDR block"
  type        = string
}

variable "availability_zones" {
  description = "List of availability zones"
  type        = list(string)
}

variable "public_subnet_cidrs" {
  description = "Public subnet CIDRs"
  type        = list(string)
}

variable "private_app_subnet_cidrs" {
  description = "Private application subnet CIDRs"
  type        = list(string)
}

variable "private_db_subnet_cidrs" {
  description = "Private database subnet CIDRs"
  type        = list(string)
}

variable "enable_nat_gateway" {
  description = "Enable NAT Gateway"
  type        = bool
  default     = true
}

variable "enable_flow_logs" {
  description = "Enable VPC Flow Logs"
  type        = bool
  default     = true
}

variable "tags" {
  description = "Resource tags"
  type        = map(string)
  default     = {}
}
```

**terraform/modules/networking/outputs.tf:**

```hcl
output "vpc_id" {
  description = "VPC ID"
  value       = aws_vpc.main.id
}

output "vpc_cidr" {
  description = "VPC CIDR block"
  value       = aws_vpc.main.cidr_block
}

output "public_subnet_ids" {
  description = "Public subnet IDs"
  value       = aws_subnet.public[*].id
}

output "private_app_subnet_ids" {
  description = "Private application subnet IDs"
  value       = aws_subnet.private_app[*].id
}

output "private_db_subnet_ids" {
  description = "Private database subnet IDs"
  value       = aws_subnet.private_db[*].id
}

output "nat_gateway_ids" {
  description = "NAT Gateway IDs"
  value       = aws_nat_gateway.main[*].id
}
```

---

### Module 2: Database

**terraform/modules/database/main.tf:**

```hcl
# RDS Aurora Cluster
resource "aws_rds_cluster" "main" {
  cluster_identifier      = "${var.name_prefix}-aurora"
  engine                  = var.engine
  engine_version          = var.engine_version
  engine_mode            = "provisioned"
  database_name          = var.database_name
  master_username        = var.master_username
  master_password        = random_password.db_password.result
  
  db_subnet_group_name   = aws_db_subnet_group.main.name
  vpc_security_group_ids = [aws_security_group.rds.id]
  
  backup_retention_period      = var.backup_retention_period
  preferred_backup_window      = var.backup_window
  preferred_maintenance_window = var.maintenance_window
  
  enabled_cloudwatch_logs_exports = ["postgresql"]
  
  storage_encrypted = true
  kms_key_id       = aws_kms_key.rds.arn
  
  skip_final_snapshot       = var.environment != "prod"
  final_snapshot_identifier = var.environment == "prod" ? "${var.name_prefix}-final-${formatdate("YYYY-MM-DD-hhmm", timestamp())}" : null
  
  deletion_protection = var.environment == "prod"
  
  serverlessv2_scaling_configuration {
    max_capacity = var.max_capacity
    min_capacity = var.min_capacity
  }
  
  tags = var.tags
}

# Aurora Instances
resource "aws_rds_cluster_instance" "main" {
  count = var.instance_count
  
  identifier         = "${var.name_prefix}-aurora-${count.index + 1}"
  cluster_identifier = aws_rds_cluster.main.id
  instance_class     = var.instance_class
  engine             = aws_rds_cluster.main.engine
  engine_version     = aws_rds_cluster.main.engine_version
  
  publicly_accessible = false
  
  performance_insights_enabled = var.enable_performance_insights
  
  tags = merge(var.tags, {
    Name = "${var.name_prefix}-aurora-instance-${count.index + 1}"
  })
}

# DB Subnet Group
resource "aws_db_subnet_group" "main" {
  name_prefix = "${var.name_prefix}-"
  subnet_ids  = var.subnet_ids
  
  tags = merge(var.tags, {
    Name = "${var.name_prefix}-db-subnet-group"
  })
}

# Security Group
resource "aws_security_group" "rds" {
  name_prefix = "${var.name_prefix}-rds-"
  description = "Security group for RDS Aurora"
  vpc_id      = var.vpc_id
  
  tags = merge(var.tags, {
    Name = "${var.name_prefix}-rds-sg"
  })
  
  lifecycle {
    create_before_destroy = true
  }
}

resource "aws_vpc_security_group_ingress_rule" "rds" {
  security_group_id = aws_security_group.rds.id
  
  from_port                    = var.port
  to_port                      = var.port
  ip_protocol                  = "tcp"
  referenced_security_group_id = var.app_security_group_id
  description                  = "PostgreSQL from application"
}

resource "aws_vpc_security_group_egress_rule" "rds" {
  security_group_id = aws_security_group.rds.id
  
  ip_protocol = "-1"
  cidr_ipv4   = "0.0.0.0/0"
  description = "All outbound traffic"
}

# KMS Key
resource "aws_kms_key" "rds" {
  description             = "KMS key for RDS encryption"
  deletion_window_in_days = 7
  enable_key_rotation     = true
  
  tags = merge(var.tags, {
    Name = "${var.name_prefix}-rds-kms"
  })
}

resource "aws_kms_alias" "rds" {
  name          = "alias/${var.name_prefix}-rds"
  target_key_id = aws_kms_key.rds.key_id
}

# Random Password
resource "random_password" "db_password" {
  length  = 32
  special = true
  
  lifecycle {
    ignore_changes = [length, special]
  }
}

# Store password in Secrets Manager
resource "aws_secretsmanager_secret" "db_password" {
  name_prefix             = "${var.name_prefix}-db-password-"
  recovery_window_in_days = 7
  kms_key_id             = aws_kms_key.rds.id
  
  tags = var.tags
}

resource "aws_secretsmanager_secret_version" "db_password" {
  secret_id = aws_secretsmanager_secret.db_password.id
  
  secret_string = jsonencode({
    username = var.master_username
    password = random_password.db_password.result
    engine   = var.engine
    host     = aws_rds_cluster.main.endpoint
    port     = var.port
    dbname   = var.database_name
  })
}

# DynamoDB Table для caching
resource "aws_dynamodb_table" "cache" {
  name           = "${var.name_prefix}-cache"
  billing_mode   = "PAY_PER_REQUEST"
  hash_key       = "id"
  
  attribute {
    name = "id"
    type = "S"
  }
  
  ttl {
    attribute_name = "ttl"
    enabled        = true
  }
  
  point_in_time_recovery {
    enabled = var.environment == "prod"
  }
  
  server_side_encryption {
    enabled = true
  }
  
  tags = merge(var.tags, {
    Name = "${var.name_prefix}-cache-table"
  })
}

# ElastiCache Redis для session store
resource "aws_elasticache_subnet_group" "main" {
  name       = "${var.name_prefix}-redis"
  subnet_ids = var.subnet_ids
  
  tags = var.tags
}

resource "aws_security_group" "redis" {
  name_prefix = "${var.name_prefix}-redis-"
  description = "Security group for Redis"
  vpc_id      = var.vpc_id
  
  tags = merge(var.tags, {
    Name = "${var.name_prefix}-redis-sg"
  })
  
  lifecycle {
    create_before_destroy = true
  }
}

resource "aws_vpc_security_group_ingress_rule" "redis" {
  security_group_id = aws_security_group.redis.id
  
  from_port                    = 6379
  to_port                      = 6379
  ip_protocol                  = "tcp"
  referenced_security_group_id = var.app_security_group_id
  description                  = "Redis from application"
}

resource "aws_vpc_security_group_egress_rule" "redis" {
  security_group_id = aws_security_group.redis.id
  
  ip_protocol = "-1"
  cidr_ipv4   = "0.0.0.0/0"
  description = "All outbound traffic"
}

resource "aws_elasticache_replication_group" "redis" {
  replication_group_id       = "${var.name_prefix}-redis"
  replication_group_description = "Redis cluster for session store"
  
  engine               = "redis"
  engine_version       = "7.0"
  node_type            = var.redis_node_type
  num_cache_clusters   = var.environment == "prod" ? 2 : 1
  port                 = 6379
  
  subnet_group_name    = aws_elasticache_subnet_group.main.name
  security_group_ids   = [aws_security_group.redis.id]
  
  automatic_failover_enabled = var.environment == "prod"
  multi_az_enabled          = var.environment == "prod"
  
  at_rest_encryption_enabled = true
  transit_encryption_enabled = true
  
  snapshot_retention_limit = var.environment == "prod" ? 5 : 1
  snapshot_window         = "03:00-05:00"
  
  tags = var.tags
}

# CloudWatch Alarms
resource "aws_cloudwatch_metric_alarm" "rds_cpu" {
  alarm_name          = "${var.name_prefix}-rds-cpu-high"
  comparison_operator = "GreaterThanThreshold"
  evaluation_periods  = "2"
  metric_name         = "CPUUtilization"
  namespace           = "AWS/RDS"
  period              = "300"
  statistic           = "Average"
  threshold           = "80"
  alarm_description   = "RDS CPU utilization is too high"
  
  dimensions = {
    DBClusterIdentifier = aws_rds_cluster.main.id
  }
  
  tags = var.tags
}

resource "aws_cloudwatch_metric_alarm" "redis_cpu" {
  alarm_name          = "${var.name_prefix}-redis-cpu-high"
  comparison_operator = "GreaterThanThreshold"
  evaluation_periods  = "2"
  metric_name         = "CPUUtilization"
  namespace           = "AWS/ElastiCache"
  period              = "300"
  statistic           = "Average"
  threshold           = "75"
  alarm_description   = "Redis CPU utilization is too high"
  
  dimensions = {
    CacheClusterId = aws_elasticache_replication_group.redis.id
  }
  
  tags = var.tags
}
```

**terraform/modules/database/variables.tf:**

```hcl
variable "name_prefix" {
  description = "Prefix for resource names"
  type        = string
}

variable "environment" {
  description = "Environment name"
  type        = string
}

variable "vpc_id" {
  description = "VPC ID"
  type        = string
}

variable "subnet_ids" {
  description = "Subnet IDs for database"
  type        = list(string)
}

variable "app_security_group_id" {
  description = "Application security group ID"
  type        = string
}

# RDS Configuration
variable "engine" {
  description = "Database engine"
  type        = string
  default     = "aurora-postgresql"
}

variable "engine_version" {
  description = "Database engine version"
  type        = string
  default     = "15.4"
}

variable "instance_class" {
  description = "Instance class"
  type        = string
  default     = "db.serverless"
}

variable "instance_count" {
  description = "Number of instances"
  type        = number
  default     = 1
}

variable "database_name" {
  description = "Database name"
  type        = string
}

variable "master_username" {
  description = "Master username"
  type        = string
  default     = "postgres"
}

variable "port" {
  description = "Database port"
  type        = number
  default     = 5432
}

variable "backup_retention_period" {
  description = "Backup retention period in days"
  type        = number
  default     = 7
}

variable "backup_window" {
  description = "Backup window"
  type        = string
  default     = "03:00-04:00"
}

variable "maintenance_window" {
  description = "Maintenance window"
  type        = string
  default     = "mon:04:00-mon:05:00"
}

variable "min_capacity" {
  description = "Minimum ACU capacity"
  type        = number
  default     = 0.5
}

variable "max_capacity" {
  description = "Maximum ACU capacity"
  type        = number
  default     = 2
}

variable "enable_performance_insights" {
  description = "Enable Performance Insights"
  type        = bool
  default     = true
}

# Redis Configuration
variable "redis_node_type" {
  description = "Redis node type"
  type        = string
  default     = "cache.t4g.micro"
}

variable "tags" {
  description = "Resource tags"
  type        = map(string)
  default     = {}
}
```

**terraform/modules/database/outputs.tf:**

```hcl
output "rds_cluster_id" {
  description = "RDS cluster ID"
  value       = aws_rds_cluster.main.id
}

output "rds_cluster_endpoint" {
  description = "RDS cluster endpoint"
  value       = aws_rds_cluster.main.endpoint
}

output "rds_cluster_reader_endpoint" {
  description = "RDS cluster reader endpoint"
  value       = aws_rds_cluster.main.reader_endpoint
}

output "rds_security_group_id" {
  description = "RDS security group ID"
  value       = aws_security_group.rds.id
}

output "db_secret_arn" {
  description = "Database credentials secret ARN"
  value       = aws_secretsmanager_secret.db_password.arn
}

output "dynamodb_table_name" {
  description = "DynamoDB table name"
  value       = aws_dynamodb_table.cache.name
}

output "redis_endpoint" {
  description = "Redis primary endpoint"
  value       = aws_elasticache_replication_group.redis.primary_endpoint_address
}

output "redis_security_group_id" {
  description = "Redis security group ID"
  value       = aws_security_group.redis.id
}
```

Продолжу с остальными модулями и финальной частью проекта.

---

### Module 3: Compute (ECS Fargate)

**terraform/modules/compute/main.tf:**

```hcl
# ECS Cluster
resource "aws_ecs_cluster" "main" {
  name = var.name_prefix
  
  setting {
    name  = "containerInsights"
    value = var.enable_container_insights ? "enabled" : "disabled"
  }
  
  tags = var.tags
}

# Cluster Capacity Providers
resource "aws_ecs_cluster_capacity_providers" "main" {
  cluster_name = aws_ecs_cluster.main.name
  
  capacity_providers = ["FARGATE", "FARGATE_SPOT"]
  
  default_capacity_provider_strategy {
    capacity_provider = "FARGATE"
    weight           = var.fargate_weight
    base             = 1
  }
  
  default_capacity_provider_strategy {
    capacity_provider = "FARGATE_SPOT"
    weight           = var.fargate_spot_weight
  }
}

# CloudWatch Log Group
resource "aws_cloudwatch_log_group" "ecs" {
  name              = "/ecs/${var.name_prefix}"
  retention_in_days = var.log_retention_days
  
  tags = var.tags
}

# Task Execution Role
resource "aws_iam_role" "task_execution" {
  name_prefix = "${var.name_prefix}-task-exec-"
  
  assume_role_policy = jsonencode({
    Version = "2012-10-17"
    Statement = [{
      Effect = "Allow"
      Principal = {
        Service = "ecs-tasks.amazonaws.com"
      }
      Action = "sts:AssumeRole"
    }]
  })
  
  tags = var.tags
}

resource "aws_iam_role_policy_attachment" "task_execution" {
  role       = aws_iam_role.task_execution.name
  policy_arn = "arn:aws:iam::aws:policy/service-role/AmazonECSTaskExecutionRolePolicy"
}

resource "aws_iam_role_policy" "task_execution_secrets" {
  name_prefix = "${var.name_prefix}-secrets-"
  role        = aws_iam_role.task_execution.id
  
  policy = jsonencode({
    Version = "2012-10-17"
    Statement = [
      {
        Effect = "Allow"
        Action = [
          "secretsmanager:GetSecretValue",
          "kms:Decrypt"
        ]
        Resource = "*"
      }
    ]
  })
}

# Task Role (application permissions)
resource "aws_iam_role" "task" {
  name_prefix = "${var.name_prefix}-task-"
  
  assume_role_policy = jsonencode({
    Version = "2012-10-17"
    Statement = [{
      Effect = "Allow"
      Principal = {
        Service = "ecs-tasks.amazonaws.com"
      }
      Action = "sts:AssumeRole"
    }]
  })
  
  tags = var.tags
}

resource "aws_iam_role_policy" "task_permissions" {
  name_prefix = "${var.name_prefix}-task-policy-"
  role        = aws_iam_role.task.id
  
  policy = jsonencode({
    Version = "2012-10-17"
    Statement = [
      {
        Effect = "Allow"
        Action = [
          "s3:GetObject",
          "s3:PutObject"
        ]
        Resource = "${var.s3_bucket_arn}/*"
      },
      {
        Effect = "Allow"
        Action = [
          "dynamodb:GetItem",
          "dynamodb:PutItem",
          "dynamodb:Query"
        ]
        Resource = var.dynamodb_table_arn
      }
    ]
  })
}

# For each service
resource "aws_ecs_task_definition" "services" {
  for_each = var.services
  
  family                   = "${var.name_prefix}-${each.key}"
  network_mode             = "awsvpc"
  requires_compatibilities = ["FARGATE"]
  cpu                      = each.value.cpu
  memory                   = each.value.memory
  
  execution_role_arn = aws_iam_role.task_execution.arn
  task_role_arn      = aws_iam_role.task.arn
  
  container_definitions = jsonencode([
    {
      name      = each.key
      image     = each.value.image
      essential = true
      
      portMappings = [{
        containerPort = each.value.port
        protocol      = "tcp"
      }]
      
      environment = concat(
        [
          {
            name  = "ENVIRONMENT"
            value = var.environment
          },
          {
            name  = "SERVICE_NAME"
            value = each.key
          }
        ],
        each.value.environment_variables
      )
      
      secrets = [
        {
          name      = "DB_CREDENTIALS"
          valueFrom = var.db_secret_arn
        }
      ]
      
      logConfiguration = {
        logDriver = "awslogs"
        options = {
          "awslogs-group"         = aws_cloudwatch_log_group.ecs.name
          "awslogs-region"        = data.aws_region.current.name
          "awslogs-stream-prefix" = each.key
        }
      }
      
      healthCheck = {
        command     = ["CMD-SHELL", each.value.health_check_command]
        interval    = 30
        timeout     = 5
        retries     = 3
        startPeriod = 60
      }
    }
  ])
  
  tags = merge(var.tags, {
    Service = each.key
  })
}

# ECS Services
resource "aws_ecs_service" "services" {
  for_each = var.services
  
  name            = each.key
  cluster         = aws_ecs_cluster.main.id
  task_definition = aws_ecs_task_definition.services[each.key].arn
  desired_count   = each.value.desired_count
  
  capacity_provider_strategy {
    capacity_provider = "FARGATE"
    weight           = var.fargate_weight
    base             = 1
  }
  
  capacity_provider_strategy {
    capacity_provider = "FARGATE_SPOT"
    weight           = var.fargate_spot_weight
  }
  
  network_configuration {
    subnets          = var.subnet_ids
    security_groups  = [aws_security_group.ecs_tasks[each.key].id]
    assign_public_ip = false
  }
  
  load_balancer {
    target_group_arn = var.target_group_arns[each.key]
    container_name   = each.key
    container_port   = each.value.port
  }
  
  deployment_configuration {
    maximum_percent         = 200
    minimum_healthy_percent = 100
    
    deployment_circuit_breaker {
      enable   = true
      rollback = true
    }
  }
  
  enable_execute_command = var.enable_exec_command
  
  tags = merge(var.tags, {
    Service = each.key
  })
  
  depends_on = [var.alb_listener]
}

# Auto Scaling
resource "aws_appautoscaling_target" "ecs" {
  for_each = var.services
  
  max_capacity       = each.value.max_count
  min_capacity       = each.value.min_count
  resource_id        = "service/${aws_ecs_cluster.main.name}/${aws_ecs_service.services[each.key].name}"
  scalable_dimension = "ecs:service:DesiredCount"
  service_namespace  = "ecs"
}

resource "aws_appautoscaling_policy" "ecs_cpu" {
  for_each = var.services
  
  name               = "${var.name_prefix}-${each.key}-cpu-scaling"
  policy_type        = "TargetTrackingScaling"
  resource_id        = aws_appautoscaling_target.ecs[each.key].resource_id
  scalable_dimension = aws_appautoscaling_target.ecs[each.key].scalable_dimension
  service_namespace  = aws_appautoscaling_target.ecs[each.key].service_namespace
  
  target_tracking_scaling_policy_configuration {
    target_value = 70.0
    
    predefined_metric_specification {
      predefined_metric_type = "ECSServiceAverageCPUUtilization"
    }
    
    scale_in_cooldown  = 300
    scale_out_cooldown = 60
  }
}

resource "aws_appautoscaling_policy" "ecs_memory" {
  for_each = var.services
  
  name               = "${var.name_prefix}-${each.key}-memory-scaling"
  policy_type        = "TargetTrackingScaling"
  resource_id        = aws_appautoscaling_target.ecs[each.key].resource_id
  scalable_dimension = aws_appautoscaling_target.ecs[each.key].scalable_dimension
  service_namespace  = aws_appautoscaling_target.ecs[each.key].service_namespace
  
  target_tracking_scaling_policy_configuration {
    target_value = 80.0
    
    predefined_metric_specification {
      predefined_metric_type = "ECSServiceAverageMemoryUtilization"
    }
    
    scale_in_cooldown  = 300
    scale_out_cooldown = 60
  }
}

# Security Groups
resource "aws_security_group" "ecs_tasks" {
  for_each = var.services
  
  name_prefix = "${var.name_prefix}-${each.key}-"
  description = "Security group for ${each.key} ECS tasks"
  vpc_id      = var.vpc_id
  
  tags = merge(var.tags, {
    Name    = "${var.name_prefix}-${each.key}-sg"
    Service = each.key
  })
  
  lifecycle {
    create_before_destroy = true
  }
}

resource "aws_vpc_security_group_ingress_rule" "ecs_tasks" {
  for_each = var.services
  
  security_group_id = aws_security_group.ecs_tasks[each.key].id
  
  from_port                    = each.value.port
  to_port                      = each.value.port
  ip_protocol                  = "tcp"
  referenced_security_group_id = var.alb_security_group_id
  description                  = "Allow traffic from ALB"
}

resource "aws_vpc_security_group_egress_rule" "ecs_tasks" {
  for_each = var.services
  
  security_group_id = aws_security_group.ecs_tasks[each.key].id
  
  ip_protocol = "-1"
  cidr_ipv4   = "0.0.0.0/0"
  description = "Allow all outbound traffic"
}

# CloudWatch Alarms
resource "aws_cloudwatch_metric_alarm" "service_cpu_high" {
  for_each = var.services
  
  alarm_name          = "${var.name_prefix}-${each.key}-cpu-high"
  comparison_operator = "GreaterThanThreshold"
  evaluation_periods  = 2
  metric_name         = "CPUUtilization"
  namespace           = "AWS/ECS"
  period              = 300
  statistic           = "Average"
  threshold           = 80
  alarm_description   = "CPU utilization is too high for ${each.key}"
  
  dimensions = {
    ClusterName = aws_ecs_cluster.main.name
    ServiceName = aws_ecs_service.services[each.key].name
  }
  
  tags = var.tags
}

resource "aws_cloudwatch_metric_alarm" "service_memory_high" {
  for_each = var.services
  
  alarm_name          = "${var.name_prefix}-${each.key}-memory-high"
  comparison_operator = "GreaterThanThreshold"
  evaluation_periods  = 2
  metric_name         = "MemoryUtilization"
  namespace           = "AWS/ECS"
  period              = 300
  statistic           = "Average"
  threshold           = 85
  alarm_description   = "Memory utilization is too high for ${each.key}"
  
  dimensions = {
    ClusterName = aws_ecs_cluster.main.name
    ServiceName = aws_ecs_service.services[each.key].name
  }
  
  tags = var.tags
}

data "aws_region" "current" {}


```
##### terraform/modules/compute/variables.tf

```
# terraform/modules/compute/variables.tf

variable "name_prefix" {
  description = "Prefix for resource names"
  type        = string
}

variable "environment" {
  description = "Environment name"
  type        = string
}

variable "vpc_id" {
  description = "VPC ID"
  type        = string
}

variable "subnet_ids" {
  description = "Subnet IDs for ECS tasks"
  type        = list(string)
}

variable "alb_security_group_id" {
  description = "ALB security group ID"
  type        = string
}

variable "alb_listener" {
  description = "ALB listener (for dependency)"
  type        = any
}

variable "target_group_arns" {
  description = "Map of service names to target group ARNs"
  type        = map(string)
}

variable "s3_bucket_arn" {
  description = "S3 bucket ARN for task permissions"
  type        = string
}

variable "dynamodb_table_arn" {
  description = "DynamoDB table ARN for task permissions"
  type        = string
}

variable "db_secret_arn" {
  description = "Database credentials secret ARN"
  type        = string
}

variable "services" {
  description = "Map of ECS services configuration"
  type = map(object({
    image                  = string
    cpu                    = number
    memory                 = number
    port                   = number
    desired_count          = number
    min_count              = number
    max_count              = number
    health_check_command   = string
    environment_variables  = list(object({
      name  = string
      value = string
    }))
  }))
}

variable "enable_container_insights" {
  description = "Enable Container Insights"
  type        = bool
  default     = true
}

variable "enable_exec_command" {
  description = "Enable ECS Exec for debugging"
  type        = bool
  default     = true
}

variable "fargate_weight" {
  description = "Weight for FARGATE capacity provider"
  type        = number
  default     = 70
}

variable "fargate_spot_weight" {
  description = "Weight for FARGATE_SPOT capacity provider"
  type        = number
  default     = 30
}

variable "log_retention_days" {
  description = "CloudWatch log retention in days"
  type        = number
  default     = 7
}

variable "tags" {
  description = "Resource tags"
  type        = map(string)
  default     = {}
}

# terraform/modules/compute/outputs.tf

output "cluster_id" {
  description = "ECS cluster ID"
  value       = aws_ecs_cluster.main.id
}

output "cluster_name" {
  description = "ECS cluster name"
  value       = aws_ecs_cluster.main.name
}

output "cluster_arn" {
  description = "ECS cluster ARN"
  value       = aws_ecs_cluster.main.arn
}

output "service_names" {
  description = "Map of service names"
  value = {
    for k, v in aws_ecs_service.services : k => v.name
  }
}

output "service_arns" {
  description = "Map of service ARNs"
  value = {
    for k, v in aws_ecs_service.services : k => v.id
  }
}

output "security_group_ids" {
  description = "Map of security group IDs"
  value = {
    for k, v in aws_security_group.ecs_tasks : k => v.id
  }
}

output "task_execution_role_arn" {
  description = "Task execution role ARN"
  value       = aws_iam_role.task_execution.arn
}

output "task_role_arn" {
  description = "Task role ARN"
  value       = aws_iam_role.task.arn
}

output "log_group_name" {
  description = "CloudWatch log group name"
  value       = aws_cloudwatch_log_group.ecs.name
}
```

###### terraform/modules/storage/main.tf

```
# terraform/modules/storage/main.tf

# S3 Bucket для frontend assets
resource "aws_s3_bucket" "frontend" {
  bucket_prefix = "${var.name_prefix}-frontend-"
  
  tags = merge(var.tags, {
    Name = "${var.name_prefix}-frontend"
    Tier = "Presentation"
  })
}

resource "aws_s3_bucket_versioning" "frontend" {
  bucket = aws_s3_bucket.frontend.id
  
  versioning_configuration {
    status = "Enabled"
  }
}

resource "aws_s3_bucket_server_side_encryption_configuration" "frontend" {
  bucket = aws_s3_bucket.frontend.id
  
  rule {
    apply_server_side_encryption_by_default {
      sse_algorithm = "AES256"
    }
  }
}

resource "aws_s3_bucket_public_access_block" "frontend" {
  bucket = aws_s3_bucket.frontend.id
  
  block_public_acls       = true
  block_public_policy     = true
  ignore_public_acls      = true
  restrict_public_buckets = true
}

resource "aws_s3_bucket_cors_configuration" "frontend" {
  bucket = aws_s3_bucket.frontend.id
  
  cors_rule {
    allowed_headers = ["*"]
    allowed_methods = ["GET", "HEAD"]
    allowed_origins = ["*"]
    expose_headers  = ["ETag"]
    max_age_seconds = 3000
  }
}

# S3 Bucket для application data
resource "aws_s3_bucket" "data" {
  bucket_prefix = "${var.name_prefix}-data-"
  
  tags = merge(var.tags, {
    Name = "${var.name_prefix}-data"
    Tier = "Data"
  })
}

resource "aws_s3_bucket_versioning" "data" {
  bucket = aws_s3_bucket.data.id
  
  versioning_configuration {
    status = "Enabled"
  }
}

resource "aws_s3_bucket_server_side_encryption_configuration" "data" {
  bucket = aws_s3_bucket.data.id
  
  rule {
    apply_server_side_encryption_by_default {
      sse_algorithm     = "aws:kms"
      kms_master_key_id = aws_kms_key.s3.id
    }
  }
}

resource "aws_s3_bucket_public_access_block" "data" {
  bucket = aws_s3_bucket.data.id
  
  block_public_acls       = true
  block_public_policy     = true
  ignore_public_acls      = true
  restrict_public_buckets = true
}

resource "aws_s3_bucket_lifecycle_configuration" "data" {
  bucket = aws_s3_bucket.data.id
  
  rule {
    id     = "transition-to-ia"
    status = "Enabled"
    
    transition {
      days          = 30
      storage_class = "STANDARD_IA"
    }
    
    transition {
      days          = 90
      storage_class = "GLACIER"
    }
    
    expiration {
      days = 365
    }
    
    noncurrent_version_expiration {
      noncurrent_days = 30
    }
  }
}

# S3 Bucket для ALB logs
resource "aws_s3_bucket" "alb_logs" {
  bucket_prefix = "${var.name_prefix}-alb-logs-"
  
  tags = merge(var.tags, {
    Name = "${var.name_prefix}-alb-logs"
  })
}

resource "aws_s3_bucket_lifecycle_configuration" "alb_logs" {
  bucket = aws_s3_bucket.alb_logs.id
  
  rule {
    id     = "expire-old-logs"
    status = "Enabled"
    
    expiration {
      days = 90
    }
  }
}

resource "aws_s3_bucket_server_side_encryption_configuration" "alb_logs" {
  bucket = aws_s3_bucket.alb_logs.id
  
  rule {
    apply_server_side_encryption_by_default {
      sse_algorithm = "AES256"
    }
  }
}

resource "aws_s3_bucket_public_access_block" "alb_logs" {
  bucket = aws_s3_bucket.alb_logs.id
  
  block_public_acls       = true
  block_public_policy     = true
  ignore_public_acls      = true
  restrict_public_buckets = true
}

# ALB logs bucket policy
data "aws_elb_service_account" "main" {}

resource "aws_s3_bucket_policy" "alb_logs" {
  bucket = aws_s3_bucket.alb_logs.id
  
  policy = jsonencode({
    Version = "2012-10-17"
    Statement = [
      {
        Effect = "Allow"
        Principal = {
          AWS = data.aws_elb_service_account.main.arn
        }
        Action   = "s3:PutObject"
        Resource = "${aws_s3_bucket.alb_logs.arn}/*"
      }
    ]
  })
}

# KMS Key для S3
resource "aws_kms_key" "s3" {
  description             = "KMS key for S3 encryption"
  deletion_window_in_days = 7
  enable_key_rotation     = true
  
  tags = merge(var.tags, {
    Name = "${var.name_prefix}-s3-kms"
  })
}

resource "aws_kms_alias" "s3" {
  name          = "alias/${var.name_prefix}-s3"
  target_key_id = aws_kms_key.s3.key_id
}

# CloudFront Origin Access Identity
resource "aws_cloudfront_origin_access_identity" "main" {
  comment = "${var.name_prefix} OAI"
}

# CloudFront Distribution
resource "aws_cloudfront_distribution" "main" {
  enabled             = true
  is_ipv6_enabled     = true
  comment             = "${var.name_prefix} distribution"
  default_root_object = "index.html"
  price_class         = var.cloudfront_price_class
  aliases             = var.domain_aliases
  
  origin {
    domain_name = aws_s3_bucket.frontend.bucket_regional_domain_name
    origin_id   = "S3-${aws_s3_bucket.frontend.id}"
    
    s3_origin_config {
      origin_access_identity = aws_cloudfront_origin_access_identity.main.cloudfront_access_identity_path
    }
  }
  
  origin {
    domain_name = var.alb_dns_name
    origin_id   = "ALB-Backend"
    
    custom_origin_config {
      http_port              = 80
      https_port             = 443
      origin_protocol_policy = "https-only"
      origin_ssl_protocols   = ["TLSv1.2"]
    }
    
    custom_header {
      name  = "X-Custom-Header"
      value = var.custom_header_value
    }
  }
  
  default_cache_behavior {
    allowed_methods  = ["GET", "HEAD", "OPTIONS"]
    cached_methods   = ["GET", "HEAD"]
    target_origin_id = "S3-${aws_s3_bucket.frontend.id}"
    
    forwarded_values {
      query_string = false
      
      cookies {
        forward = "none"
      }
    }
    
    viewer_protocol_policy = "redirect-to-https"
    min_ttl                = 0
    default_ttl            = 3600
    max_ttl                = 86400
    compress               = true
  }
  
  ordered_cache_behavior {
    path_pattern     = "/api/*"
    allowed_methods  = ["DELETE", "GET", "HEAD", "OPTIONS", "PATCH", "POST", "PUT"]
    cached_methods   = ["GET", "HEAD"]
    target_origin_id = "ALB-Backend"
    
    forwarded_values {
      query_string = true
      headers      = ["Authorization", "CloudFront-Forwarded-Proto"]
      
      cookies {
        forward = "all"
      }
    }
    
    viewer_protocol_policy = "https-only"
    min_ttl                = 0
    default_ttl            = 0
    max_ttl                = 0
    compress               = true
  }
  
  restrictions {
    geo_restriction {
      restriction_type = var.geo_restriction_type
      locations        = var.geo_restriction_locations
    }
  }
  
  viewer_certificate {
    acm_certificate_arn      = var.acm_certificate_arn
    ssl_support_method       = "sni-only"
    minimum_protocol_version = "TLSv1.2_2021"
  }
  
  custom_error_response {
    error_code         = 403
    response_code      = 200
    response_page_path = "/index.html"
  }
  
  custom_error_response {
    error_code         = 404
    response_code      = 200
    response_page_path = "/index.html"
  }
  
  logging_config {
    include_cookies = false
    bucket          = aws_s3_bucket.cloudfront_logs.bucket_domain_name
    prefix          = "cloudfront/"
  }
  
  tags = var.tags
}

# S3 Bucket для CloudFront logs
resource "aws_s3_bucket" "cloudfront_logs" {
  bucket_prefix = "${var.name_prefix}-cf-logs-"
  
  tags = merge(var.tags, {
    Name = "${var.name_prefix}-cloudfront-logs"
  })
}

resource "aws_s3_bucket_ownership_controls" "cloudfront_logs" {
  bucket = aws_s3_bucket.cloudfront_logs.id
  
  rule {
    object_ownership = "BucketOwnerPreferred"
  }
}

resource "aws_s3_bucket_acl" "cloudfront_logs" {
  bucket = aws_s3_bucket.cloudfront_logs.id
  acl    = "private"
  
  depends_on = [aws_s3_bucket_ownership_controls.cloudfront_logs]
}

resource "aws_s3_bucket_lifecycle_configuration" "cloudfront_logs" {
  bucket = aws_s3_bucket.cloudfront_logs.id
  
  rule {
    id     = "expire-old-logs"
    status = "Enabled"
    
    expiration {
      days = 30
    }
  }
}

# S3 Bucket Policy для CloudFront OAI
resource "aws_s3_bucket_policy" "frontend" {
  bucket = aws_s3_bucket.frontend.id
  
  policy = jsonencode({
    Version = "2012-10-17"
    Statement = [
      {
        Effect = "Allow"
        Principal = {
          AWS = aws_cloudfront_origin_access_identity.main.iam_arn
        }
        Action   = "s3:GetObject"
        Resource = "${aws_s3_bucket.frontend.arn}/*"
      }
    ]
  })
}

# CloudWatch Alarms для CloudFront
resource "aws_cloudwatch_metric_alarm" "cloudfront_error_rate" {
  alarm_name          = "${var.name_prefix}-cloudfront-error-rate"
  comparison_operator = "GreaterThanThreshold"
  evaluation_periods  = "2"
  metric_name         = "5xxErrorRate"
  namespace           = "AWS/CloudFront"
  period              = "300"
  statistic           = "Average"
  threshold           = "5"
  alarm_description   = "CloudFront 5xx error rate is too high"
  
  dimensions = {
    DistributionId = aws_cloudfront_distribution.main.id
  }
  
  tags = var.tags
}
```

terraform/modules/storage/variables.tf

```
# terraform/modules/storage/variables.tf

variable "name_prefix" {
  description = "Prefix for resource names"
  type        = string
}

variable "alb_dns_name" {
  description = "ALB DNS name for CloudFront origin"
  type        = string
}

variable "acm_certificate_arn" {
  description = "ACM certificate ARN for CloudFront"
  type        = string
}

variable "domain_aliases" {
  description = "Domain aliases for CloudFront"
  type        = list(string)
  default     = []
}

variable "custom_header_value" {
  description = "Custom header value for ALB origin"
  type        = string
  sensitive   = true
}

variable "cloudfront_price_class" {
  description = "CloudFront price class"
  type        = string
  default     = "PriceClass_100"
}

variable "geo_restriction_type" {
  description = "Geo restriction type (whitelist, blacklist, none)"
  type        = string
  default     = "none"
}

variable "geo_restriction_locations" {
  description = "List of country codes for geo restriction"
  type        = list(string)
  default     = []
}

variable "tags" {
  description = "Resource tags"
  type        = map(string)
  default     = {}
}

# terraform/modules/storage/outputs.tf

output "frontend_bucket_id" {
  description = "Frontend S3 bucket ID"
  value       = aws_s3_bucket.frontend.id
}

output "frontend_bucket_arn" {
  description = "Frontend S3 bucket ARN"
  value       = aws_s3_bucket.frontend.arn
}

output "data_bucket_id" {
  description = "Data S3 bucket ID"
  value       = aws_s3_bucket.data.id
}

output "data_bucket_arn" {
  description = "Data S3 bucket ARN"
  value       = aws_s3_bucket.data.arn
}

output "alb_logs_bucket_id" {
  description = "ALB logs S3 bucket ID"
  value       = aws_s3_bucket.alb_logs.id
}

output "cloudfront_distribution_id" {
  description = "CloudFront distribution ID"
  value       = aws_cloudfront_distribution.main.id
}

output "cloudfront_distribution_arn" {
  description = "CloudFront distribution ARN"
  value       = aws_cloudfront_distribution.main.arn
}

output "cloudfront_domain_name" {
  description = "CloudFront distribution domain name"
  value       = aws_cloudfront_distribution.main.domain_name
}

output "cloudfront_hosted_zone_id" {
  description = "CloudFront hosted zone ID"
  value       = aws_cloudfront_distribution.main.hosted_zone_id
}

output "kms_key_id" {
  description = "KMS key ID for S3 encryption"
  value       = aws_kms_key.s3.id
}

output "kms_key_arn" {
  description = "KMS key ARN for S3 encryption"
  value       = aws_kms_key.s3.arn
}
```

Создадим производственное окружение и CI/CD pipeline:
##### terraform/environments/prod/main.tf

```
# terraform/environments/prod/main.tf

terraform {
  required_version = ">= 1.5.0"
  
  required_providers {
    aws = {
      source  = "hashicorp/aws"
      version = "~> 5.0"
    }
    random = {
      source  = "hashicorp/random"
      version = "~> 3.5"
    }
  }
  
  backend "s3" {
    bucket         = "your-terraform-state-bucket"
    key            = "prod/terraform.tfstate"
    region         = "us-east-1"
    encrypt        = true
    dynamodb_table = "terraform-state-lock"
  }
}

provider "aws" {
  region = var.aws_region
  
  default_tags {
    tags = {
      Environment = "production"
      Project     = var.project_name
      ManagedBy   = "Terraform"
    }
  }
}

# Local variables
locals {
  name_prefix = "${var.project_name}-prod"
  
  common_tags = {
    Environment = "production"
    Project     = var.project_name
    Terraform   = "true"
  }
  
  availability_zones = ["${var.aws_region}a", "${var.aws_region}b", "${var.aws_region}c"]
}

# Networking Module
module "networking" {
  source = "../../modules/networking"
  
  name_prefix         = local.name_prefix
  vpc_cidr           = "10.0.0.0/16"
  availability_zones = local.availability_zones
  
  public_subnet_cidrs = [
    "10.0.1.0/24",
    "10.0.2.0/24",
    "10.0.3.0/24"
  ]
  
  private_app_subnet_cidrs = [
    "10.0.11.0/24",
    "10.0.12.0/24",
    "10.0.13.0/24"
  ]
  
  private_db_subnet_cidrs = [
    "10.0.21.0/24",
    "10.0.22.0/24",
    "10.0.23.0/24"
  ]
  
  enable_nat_gateway = true
  enable_flow_logs   = true
  
  tags = local.common_tags
}

# Database Module
module "database" {
  source = "../../modules/database"
  
  name_prefix            = local.name_prefix
  environment            = "prod"
  vpc_id                 = module.networking.vpc_id
  subnet_ids             = module.networking.private_db_subnet_ids
  app_security_group_id  = module.compute.service_security_group_ids["api"]
  
  # RDS Configuration
  engine                     = "aurora-postgresql"
  engine_version            = "15.4"
  instance_class            = "db.serverless"
  instance_count            = 2
  database_name             = var.database_name
  master_username           = "postgres"
  
  backup_retention_period    = 30
  backup_window             = "03:00-04:00"
  maintenance_window        = "mon:04:00-mon:05:00"
  
  min_capacity              = 0.5
  max_capacity              = 8
  enable_performance_insights = true
  
  # Redis Configuration
  redis_node_type = "cache.r7g.large"
  
  tags = local.common_tags
}

# Storage Module
module "storage" {
  source = "../../modules/storage"
  
  name_prefix         = local.name_prefix
  alb_dns_name        = module.compute.alb_dns_name
  acm_certificate_arn = var.acm_certificate_arn
  domain_aliases      = var.cloudfront_aliases
  
  custom_header_value    = var.custom_header_value
  cloudfront_price_class = "PriceClass_All"
  
  geo_restriction_type = "none"
  
  tags = local.common_tags
}

# Compute Module
module "compute" {
  source = "../../modules/compute"
  
  name_prefix         = local.name_prefix
  environment         = "prod"
  aws_region          = var.aws_region
  vpc_id              = module.networking.vpc_id
  public_subnet_ids   = module.networking.public_subnet_ids
  private_subnet_ids  = module.networking.private_app_subnet_ids
  
  acm_certificate_arn = var.acm_certificate_arn
  alb_logs_bucket     = module.storage.alb_logs_bucket_id
  s3_bucket_arn       = module.storage.data_bucket_arn
  dynamodb_table_arn  = module.database.dynamodb_table_name
  db_secret_arn       = module.database.db_secret_arn
  
  enable_container_insights = true
  log_retention_days        = 30
  
  fargate_weight      = 2
  fargate_spot_weight = 3
  
  services = {
    api = {
      image                = "${var.ecr_repository_url}/api:latest"
      cpu                  = 512
      memory               = 1024
      port                 = 8080
      desired_count        = 3
      min_capacity         = 2
      max_capacity         = 10
      health_check_path    = "/health"
      health_check_command = "curl -f http://localhost:8080/health || exit 1"
      path_patterns        = ["/api/*"]
      priority             = 100
      
      environment_variables = [
        {
          name  = "LOG_LEVEL"
          value = "info"
        },
        {
          name  = "REDIS_ENDPOINT"
          value = module.database.redis_endpoint
        }
      ]
    }
    
    worker = {
      image                = "${var.ecr_repository_url}/worker:latest"
      cpu                  = 256
      memory               = 512
      port                 = 8081
      desired_count        = 2
      min_capacity         = 1
      max_capacity         = 5
      health_check_path    = "/health"
      health_check_command = "curl -f http://localhost:8081/health || exit 1"
      path_patterns        = ["/worker/*"]
      priority             = 200
      
      environment_variables = [
        {
          name  = "WORKER_QUEUE"
          value = "production"
        }
      ]
    }
  }
  
  tags = local.common_tags
}

# Route53 Records
resource "aws_route53_record" "main" {
  zone_id = var.route53_zone_id
  name    = var.domain_name
  type    = "A"
  
  alias {
    name                   = module.storage.cloudfront_domain_name
    zone_id                = module.storage.cloudfront_hosted_zone_id
    evaluate_target_health = false
  }
}

resource "aws_route53_record" "api" {
  zone_id = var.route53_zone_id
  name    = "api.${var.domain_name}"
  type    = "A"
  
  alias {
    name                   = module.compute.alb_dns_name
    zone_id                = module.compute.alb_zone_id
    evaluate_target_health = true
  }
}

# SNS Topic для alerts
resource "aws_sns_topic" "alerts" {
  name = "${local.name_prefix}-alerts"
  
  tags = local.common_tags
}

resource "aws_sns_topic_subscription" "alerts_email" {
  topic_arn = aws_sns_topic.alerts.arn
  protocol  = "email"
  endpoint  = var.alert_email
}

# CloudWatch Dashboard
resource "aws_cloudwatch_dashboard" "main" {
  dashboard_name = local.name_prefix
  
  dashboard_body = jsonencode({
    widgets = [
      {
        type = "metric"
        properties = {
          metrics = [
            ["AWS/ECS", "CPUUtilization", { stat = "Average" }],
            [".", "MemoryUtilization", { stat = "Average" }]
          ]
          period = 300
          stat   = "Average"
          region = var.aws_region
          title  = "ECS Cluster Metrics"
        }
      },
      {
        type = "metric"
        properties = {
          metrics = [
            ["AWS/RDS", "CPUUtilization", { stat = "Average" }],
            [".", "DatabaseConnections", { stat = "Sum" }]
          ]
          period = 300
          stat   = "Average"
          region = var.aws_region
          title  = "RDS Metrics"
        }
      },
      {
        type = "metric"
        properties = {
          metrics = [
            ["AWS/ApplicationELB", "TargetResponseTime", { stat = "Average" }],
            [".", "RequestCount", { stat = "Sum" }]
          ]
          period = 300
          region = var.aws_region
          title  = "ALB Metrics"
        }
      },
      {
        type = "metric"
        properties = {
          metrics = [
            ["AWS/CloudFront", "Requests", { stat = "Sum" }],
            [".", "BytesDownloaded", { stat = "Sum" }]
          ]
          period = 300
          region = "us-east-1"
          title  = "CloudFront Metrics"
        }
      }
    ]
  })
}
```

###### terraform/environments/prod/variables.tf

```
# terraform/environments/prod/variables.tf

variable "aws_region" {
  description = "AWS region"
  type        = string
  default     = "us-east-1"
}

variable "project_name" {
  description = "Project name"
  type        = string
}

variable "database_name" {
  description = "Database name"
  type        = string
  default     = "production_db"
}

variable "domain_name" {
  description = "Primary domain name"
  type        = string
}

variable "cloudfront_aliases" {
  description = "CloudFront domain aliases"
  type        = list(string)
}

variable "acm_certificate_arn" {
  description = "ACM certificate ARN"
  type        = string
}

variable "route53_zone_id" {
  description = "Route53 hosted zone ID"
  type        = string
}

variable "ecr_repository_url" {
  description = "ECR repository URL"
  type        = string
}

variable "custom_header_value" {
  description = "Custom header value for ALB origin verification"
  type        = string
  sensitive   = true
}

variable "alert_email" {
  description = "Email address for alerts"
  type        = string
}

# terraform/environments/prod/outputs.tf

output "vpc_id" {
  description = "VPC ID"
  value       = module.networking.vpc_id
}

output "alb_dns_name" {
  description = "ALB DNS name"
  value       = module.compute.alb_dns_name
}

output "cloudfront_domain_name" {
  description = "CloudFront distribution domain name"
  value       = module.storage.cloudfront_domain_name
}

output "website_url" {
  description = "Website URL"
  value       = "https://${var.domain_name}"
}

output "api_url" {
  description = "API URL"
  value       = "https://api.${var.domain_name}"
}

output "rds_endpoint" {
  description = "RDS cluster endpoint"
  value       = module.database.rds_cluster_endpoint
  sensitive   = true
}

output "redis_endpoint" {
  description = "Redis endpoint"
  value       = module.database.redis_endpoint
  sensitive   = true
}

output "frontend_bucket" {
  description = "Frontend S3 bucket name"
  value       = module.storage.frontend_bucket_id
}

output "ecs_cluster_name" {
  description = "ECS cluster name"
  value       = module.compute.ecs_cluster_name
}

output "db_secret_arn" {
  description = "Database credentials secret ARN"
  value       = module.database.db_secret_arn
  sensitive   = true
}

# terraform/environments/prod/terraform.tfvars.example

aws_region   = "us-east-1"
project_name = "myapp"

database_name = "production_db"

domain_name = "example.com"
cloudfront_aliases = [
  "example.com",
  "www.example.com"
]

acm_certificate_arn = "arn:aws:acm:us-east-1:123456789012:certificate/..."
route53_zone_id     = "Z1234567890ABC"
ecr_repository_url  = "123456789012.dkr.ecr.us-east-1.amazonaws.com/myapp"

custom_header_value = "random-secure-string-here"
alert_email        = "alerts@example.com"
```

##### .github/workflows/deploy-production.yml
```
# .github/workflows/deploy-production.yml

name: Deploy to Production

on:
  push:
    branches:
      - main
  workflow_dispatch:

env:
  AWS_REGION: us-east-1
  ECR_REPOSITORY: myapp
  ECS_CLUSTER: myapp-prod
  TERRAFORM_VERSION: 1.5.0

jobs:
  test:
    name: Run Tests
    runs-on: ubuntu-latest
    
    steps:
      - name: Checkout code
        uses: actions/checkout@v4
      
      - name: Set up Node.js
        uses: actions/setup-node@v4
        with:
          node-version: '20'
          cache: 'npm'
      
      - name: Install dependencies
        run: npm ci
      
      - name: Run linter
        run: npm run lint
      
      - name: Run unit tests
        run: npm test
      
      - name: Run integration tests
        run: npm run test:integration
  
  security-scan:
    name: Security Scanning
    runs-on: ubuntu-latest
    
    steps:
      - name: Checkout code
        uses: actions/checkout@v4
      
      - name: Run Trivy vulnerability scanner
        uses: aquasecurity/trivy-action@master
        with:
          scan-type: 'fs'
          scan-ref: '.'
          format: 'sarif'
          output: 'trivy-results.sarif'
      
      - name: Upload Trivy results to GitHub Security
        uses: github/codeql-action/upload-sarif@v2
        with:
          sarif_file: 'trivy-results.sarif'
      
      - name: OWASP Dependency Check
        uses: dependency-check/Dependency-Check_Action@main
        with:
          project: 'myapp'
          path: '.'
          format: 'HTML'
  
  build-and-push:
    name: Build and Push Docker Images
    runs-on: ubuntu-latest
    needs: [test, security-scan]
    
    outputs:
      api-image: ${{ steps.build-api.outputs.image }}
      worker-image: ${{ steps.build-worker.outputs.image }}
    
    steps:
      - name: Checkout code
        uses: actions/checkout@v4
      
      - name: Configure AWS credentials
        uses: aws-actions/configure-aws-credentials@v4
        with:
          aws-access-key-id: ${{ secrets.AWS_ACCESS_KEY_ID }}
          aws-secret-access-key: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
          aws-region: ${{ env.AWS_REGION }}
      
      - name: Login to Amazon ECR
        id: login-ecr
        uses: aws-actions/amazon-ecr-login@v2
      
      - name: Set up Docker Buildx
        uses: docker/setup-buildx-action@v3
      
      - name: Build and push API image
        id: build-api
        env:
          ECR_REGISTRY: ${{ steps.login-ecr.outputs.registry }}
          IMAGE_TAG: ${{ github.sha }}
        run: |
          docker buildx build \
            --platform linux/amd64 \
            --cache-from type=registry,ref=$ECR_REGISTRY/$ECR_REPOSITORY/api:cache \
            --cache-to type=registry,ref=$ECR_REGISTRY/$ECR_REPOSITORY/api:cache,mode=max \
            --tag $ECR_REGISTRY/$ECR_REPOSITORY/api:$IMAGE_TAG \
            --tag $ECR_REGISTRY/$ECR_REPOSITORY/api:latest \
            --file docker/backend/Dockerfile \
            --push \
            .
          echo "image=$ECR_REGISTRY/$ECR_REPOSITORY/api:$IMAGE_TAG" >> $GITHUB_OUTPUT
      
      - name: Build and push Worker image
        id: build-worker
        env:
          ECR_REGISTRY: ${{ steps.login-ecr.outputs.registry }}
          IMAGE_TAG: ${{ github.sha }}
        run: |
          docker buildx build \
            --platform linux/amd64 \
            --cache-from type=registry,ref=$ECR_REGISTRY/$ECR_REPOSITORY/worker:cache \
            --cache-to type=registry,ref=$ECR_REGISTRY/$ECR_REPOSITORY/worker:cache,mode=max \
            --tag $ECR_REGISTRY/$ECR_REPOSITORY/worker:$IMAGE_TAG \
            --tag $ECR_REGISTRY/$ECR_REPOSITORY/worker:latest \
            --file docker/worker/Dockerfile \
            --push \
            .
          echo "image=$ECR_REGISTRY/$ECR_REPOSITORY/worker:$IMAGE_TAG" >> $GITHUB_OUTPUT
      
      - name: Scan API image with Trivy
        uses: aquasecurity/trivy-action@master
        with:
          image-ref: ${{ steps.build-api.outputs.image }}
          format: 'sarif'
          output: 'trivy-api-results.sarif'
      
      - name: Scan Worker image with Trivy
        uses: aquasecurity/trivy-action@master
        with:
          image-ref: ${{ steps.build-worker.outputs.image }}
          format: 'sarif'
          output: 'trivy-worker-results.sarif'
  
  terraform-plan:
    name: Terraform Plan
    runs-on: ubuntu-latest
    needs: [build-and-push]
    
    steps:
      - name: Checkout code
        uses: actions/checkout@v4
      
      - name: Configure AWS credentials
        uses: aws-actions/configure-aws-credentials@v4
        with:
          aws-access-key-id: ${{ secrets.AWS_ACCESS_KEY_ID }}
          aws-secret-access-key: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
          aws-region: ${{ env.AWS_REGION }}
      
      - name: Setup Terraform
        uses: hashicorp/setup-terraform@v3
        with:
          terraform_version: ${{ env.TERRAFORM_VERSION }}
      
      - name: Terraform Init
        working-directory: terraform/environments/prod
        run: terraform init
      
      - name: Terraform Validate
        working-directory: terraform/environments/prod
        run: terraform validate
      
      - name: Terraform Plan
        working-directory: terraform/environments/prod
        run: |
          terraform plan \
            -var="ecr_repository_url=${{ secrets.ECR_REGISTRY }}" \
            -out=tfplan
      
      - name: Upload Terraform Plan
        uses: actions/upload-artifact@v3
        with:
          name: tfplan
          path: terraform/environments/prod/tfplan
  
  terraform-apply:
    name: Terraform Apply
    runs-on: ubuntu-latest
    needs: [terraform-plan]
    environment: production
    
    steps:
      - name: Checkout code
        uses: actions/checkout@v4
      
      - name: Configure AWS credentials
        uses: aws-actions/configure-aws-credentials@v4
        with:
          aws-access-key-id: ${{ secrets.AWS_ACCESS_KEY_ID }}
          aws-secret-access-key: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
          aws-region: ${{ env.AWS_REGION }}
      
      - name: Setup Terraform
        uses: hashicorp/setup-terraform@v3
        with:
          terraform_version: ${{ env.TERRAFORM_VERSION }}
      
      - name: Terraform Init
        working-directory: terraform/environments/prod
        run: terraform init
      
      - name: Download Terraform Plan
        uses: actions/download-artifact@v3
        with:
          name: tfplan
          path: terraform/environments/prod/
      
      - name: Terraform Apply
        working-directory: terraform/environments/prod
        run: terraform apply -auto-approve tfplan
  
  deploy-ecs:
    name: Deploy to ECS
    runs-on: ubuntu-latest
    needs: [build-and-push, terraform-apply]
    environment: production
    
    strategy:
      matrix:
        service: [api, worker]
    
    steps:
      - name: Checkout code
        uses: actions/checkout@v4
      
      - name: Configure AWS credentials
        uses: aws-actions/configure-aws-credentials@v4
        with:
          aws-access-key-id: ${{ secrets.AWS_ACCESS_KEY_ID }}
          aws-secret-access-key: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
          aws-region: ${{ env.AWS_REGION }}
      
      - name: Download task definition
        run: |
          aws ecs describe-task-definition \
            --task-definition myapp-prod-${{ matrix.service }} \
            --query taskDefinition > task-definition.json
      
      - name: Fill in the new image ID in the task definition
        id: task-def
        uses: aws-actions/amazon-ecs-render-task-definition@v1
        with:
          task-definition: task-definition.json
          container-name: ${{ matrix.service }}
          image: ${{ needs.build-and-push.outputs[format('{0}-image', matrix.service)] }}
      
      - name: Deploy to ECS
        uses: aws-actions/amazon-ecs-deploy-task-definition@v1
        with:
          task-definition: ${{ steps.task-def.outputs.task-definition }}
          service: ${{ matrix.service }}
          cluster: ${{ env.ECS_CLUSTER }}
          wait-for-service-stability: true
  
  deploy-frontend:
    name: Deploy Frontend to S3
    runs-on: ubuntu-latest
    needs: [test, terraform-apply]
    environment: production
    
    steps:
      - name: Checkout code
        uses: actions/checkout@v4
      
      - name: Set up Node.js
        uses: actions/setup-node@v4
        with:
          node-version: '20'
          cache: 'npm'
      
      - name: Install dependencies
        working-directory: frontend
        run: npm ci
      
      - name: Build frontend
        working-directory: frontend
        env:
          REACT_APP_API_URL: https://api.example.com
          REACT_APP_ENV: production
        run: npm run build
      
      - name: Configure AWS credentials
        uses: aws-actions/configure-aws-credentials@v4
        with:
          aws-access-key-id: ${{ secrets.AWS_ACCESS_KEY_ID }}
          aws-secret-access-key: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
          aws-region: ${{ env.AWS_REGION }}
      
      - name: Deploy to S3
        run: |
          aws s3 sync frontend/build/ s3://${{ secrets.FRONTEND_BUCKET }} \
            --delete \
            --cache-control "public,max-age=31536000,immutable" \
            --exclude "index.html" \
            --exclude "service-worker.js"
          
          aws s3 cp frontend/build/index.html s3://${{ secrets.FRONTEND_BUCKET }}/index.html \
            --cache-control "public,max-age=0,must-revalidate"
          
          aws s3 cp frontend/build/service-worker.js s3://${{ secrets.FRONTEND_BUCKET }}/service-worker.js \
            --cache-control "public,max-age=0,must-revalidate"
      
      - name: Invalidate CloudFront cache
        run: |
          aws cloudfront create-invalidation \
            --distribution-id ${{ secrets.CLOUDFRONT_DISTRIBUTION_ID }} \
            --paths "/*"
  
  smoke-tests:
    name: Run Smoke Tests
    runs-on: ubuntu-latest
    needs: [deploy-ecs, deploy-frontend]
    
    steps:
      - name: Checkout code
        uses: actions/checkout@v4
      
      - name: Set up Node.js
        uses: actions/setup-node@v4
        with:
          node-version: '20'
      
      - name: Install dependencies
        run: npm ci
      
      - name: Run smoke tests
        env:
          API_URL: https://api.example.com
          FRONTEND_URL: https://example.com
        run: npm run test:smoke
      
      - name: Health check
        run: |
          curl -f https://api.example.com/health || exit 1
          curl -f https://example.com || exit 1
  
  notify:
    name: Send Notifications
    runs-on: ubuntu-latest
    needs: [smoke-tests]
    if: always()
    
    steps:
      - name: Send Slack notification
        uses: slackapi/slack-github-action@v1
        with:
          payload: |
            {
              "text": "Production Deployment ${{ needs.smoke-tests.result }}",
              "blocks": [
                {
                  "type": "section",
                  "text": {
                    "type": "mrkdwn",
                    "text": "Production deployment *${{ needs.smoke-tests.result }}*\n*Commit:* ${{ github.sha }}\n*Author:* ${{ github.actor }}"
                  }
                }
              ]
            }
        env:
          SLACK_WEBHOOK_URL: ${{ secrets.SLACK_WEBHOOK_URL }}
```

Создадим примеры Dockerfiles и скрипты деплоя:

###### docker/backend/Dockerfile

```
# docker/backend/Dockerfile

FROM node:20-alpine AS builder

WORKDIR /app

# Copy package files
COPY package*.json ./
RUN npm ci --only=production

# Copy application files
COPY . .

# Build TypeScript
RUN npm run build

# Production image
FROM node:20-alpine

# Install dumb-init for proper signal handling
RUN apk add --no-cache dumb-init

# Create app user
RUN addgroup -g 1001 -S nodejs && \
    adduser -S nodejs -u 1001

WORKDIR /app

# Copy built application
COPY --from=builder --chown=nodejs:nodejs /app/dist ./dist
COPY --from=builder --chown=nodejs:nodejs /app/node_modules ./node_modules
COPY --from=builder --chown=nodejs:nodejs /app/package.json ./

# Health check
HEALTHCHECK --interval=30s --timeout=3s --start-period=60s --retries=3 \
  CMD node -e "require('http').get('http://localhost:8080/health', (r) => process.exit(r.statusCode === 200 ? 0 : 1))"

USER nodejs

EXPOSE 8080

ENTRYPOINT ["dumb-init", "--"]
CMD ["node", "dist/index.js"]
```

##### docker/worker/Dockerfile

```
# docker/worker/Dockerfile

FROM node:20-alpine AS builder

WORKDIR /app

COPY package*.json ./
RUN npm ci --only=production

COPY . .
RUN npm run build

FROM node:20-alpine

RUN apk add --no-cache dumb-init

RUN addgroup -g 1001 -S nodejs && \
    adduser -S nodejs -u 1001

WORKDIR /app

COPY --from=builder --chown=nodejs:nodejs /app/dist ./dist
COPY --from=builder --chown=nodejs:nodejs /app/node_modules ./node_modules
COPY --from=builder --chown=nodejs:nodejs /app/package.json ./

HEALTHCHECK --interval=30s --timeout=3s --start-period=60s --retries=3 \
  CMD node -e "require('http').get('http://localhost:8081/health', (r) => process.exit(r.statusCode === 200 ? 0 : 1))"

USER nodejs

EXPOSE 8081

ENTRYPOINT ["dumb-init", "--"]
CMD ["node", "dist/worker.js"]
```
##### docker/frontend/Dockerfile

```

# docker/frontend/Dockerfile

FROM node:20-alpine AS builder

WORKDIR /app

# Copy package files
COPY package*.json ./
RUN npm ci

# Copy source code
COPY . .

# Build application
ARG REACT_APP_API_URL
ARG REACT_APP_ENV=production

RUN npm run build

# Production image with nginx
FROM nginx:alpine

# Copy custom nginx config
COPY docker/nginx/nginx.conf /etc/nginx/nginx.conf
COPY docker/nginx/default.conf /etc/nginx/conf.d/default.conf

# Copy built application
COPY --from=builder /app/build /usr/share/nginx/html

# Add health check
HEALTHCHECK --interval=30s --timeout=3s --start-period=10s --retries=3 \
  CMD wget --quiet --tries=1 --spider http://localhost:80/health || exit 1

EXPOSE 80

CMD ["nginx", "-g", "daemon off;"]
```

Nginx Configuration
##### docker/nginx/nginx.conf

```
# docker/nginx/nginx.conf

user nginx;
worker_processes auto;
error_log /var/log/nginx/error.log warn;
pid /var/run/nginx.pid;

events {
    worker_connections 1024;
    use epoll;
}

http {
    include /etc/nginx/mime.types;
    default_type application/octet-stream;

    log_format main '$remote_addr - $remote_user [$time_local] "$request" '
                    '$status $body_bytes_sent "$http_referer" '
                    '"$http_user_agent" "$http_x_forwarded_for"';

    access_log /var/log/nginx/access.log main;

    sendfile on;
    tcp_nopush on;
    tcp_nodelay on;
    keepalive_timeout 65;
    types_hash_max_size 2048;
    server_tokens off;

    # Gzip compression
    gzip on;
    gzip_vary on;
    gzip_proxied any;
    gzip_comp_level 6;
    gzip_types text/plain text/css text/xml text/javascript 
               application/json application/javascript application/xml+rss 
               application/rss+xml font/truetype font/opentype 
               application/vnd.ms-fontobject image/svg+xml;

    # Security headers
    add_header X-Frame-Options "SAMEORIGIN" always;
    add_header X-Content-Type-Options "nosniff" always;
    add_header X-XSS-Protection "1; mode=block" always;
    add_header Referrer-Policy "no-referrer-when-downgrade" always;
    add_header Content-Security-Policy "default-src 'self' http: https: data: blob: 'unsafe-inline'" always;

    include /etc/nginx/conf.d/*.conf;
}

# docker/nginx/default.conf

server {
    listen 80;
    server_name _;
    root /usr/share/nginx/html;
    index index.html;

    # Security headers
    add_header Strict-Transport-Security "max-age=31536000; includeSubDomains" always;
    add_header X-Frame-Options "SAMEORIGIN" always;
    add_header X-Content-Type-Options "nosniff" always;
    add_header X-XSS-Protection "1; mode=block" always;

    # Health check endpoint
    location /health {
        access_log off;
        return 200 "healthy\n";
        add_header Content-Type text/plain;
    }

    # Static files with cache
    location ~* \.(jpg|jpeg|png|gif|ico|css|js|svg|woff|woff2|ttf|eot)$ {
        expires 1y;
        add_header Cache-Control "public, immutable";
    }

    # Main application
    location / {
        try_files $uri $uri/ /index.html;
        add_header Cache-Control "no-cache, no-store, must-revalidate";
        add_header Pragma "no-cache";
        add_header Expires "0";
    }

    # API proxy (if needed)
    location /api/ {
        proxy_pass http://api-backend:8080/;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection 'upgrade';
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
        proxy_cache_bypass $http_upgrade;
        proxy_read_timeout 90;
    }

    # Error pages
    error_page 404 /404.html;
    error_page 500 502 503 504 /50x.html;
    
    location = /50x.html {
        root /usr/share/nginx/html;
    }
}
```

Deployment scripts

```bash
#!/bin/bash
# scripts/deploy/deploy.sh

set -e

# Colors for output
RED='\033[0;31m'
GREEN='\033[0;32m'
YELLOW='\033[1;33m'
NC='\033[0m' # No Color

# Configuration
ENVIRONMENT=${1:-dev}
AWS_REGION=${AWS_REGION:-us-east-1}
PROJECT_NAME="myapp"

echo -e "${GREEN}Starting deployment to ${ENVIRONMENT}...${NC}"

# Function to check if command exists
command_exists() {
    command -v "$1" >/dev/null 2>&1
}

# Check required tools
echo -e "${YELLOW}Checking required tools...${NC}"
for tool in aws terraform docker; do
    if ! command_exists $tool; then
        echo -e "${RED}Error: $tool is not installed${NC}"
        exit 1
    fi
done

# Build and push Docker images
echo -e "${YELLOW}Building Docker images...${NC}"

# Get ECR login
aws ecr get-login-password --region $AWS_REGION | \
    docker login --username AWS --password-stdin \
    $(aws sts get-caller-identity --query Account --output text).dkr.ecr.$AWS_REGION.amazonaws.com

# Build and push API
echo -e "${YELLOW}Building API image...${NC}"
docker build -t ${PROJECT_NAME}/api:latest -f docker/backend/Dockerfile .
docker tag ${PROJECT_NAME}/api:latest \
    $(aws sts get-caller-identity --query Account --output text).dkr.ecr.$AWS_REGION.amazonaws.com/${PROJECT_NAME}/api:latest
docker push $(aws sts get-caller-identity --query Account --output text).dkr.ecr.$AWS_REGION.amazonaws.com/${PROJECT_NAME}/api:latest

# Build and push Worker
echo -e "${YELLOW}Building Worker image...${NC}"
docker build -t ${PROJECT_NAME}/worker:latest -f docker/worker/Dockerfile .
docker tag ${PROJECT_NAME}/worker:latest \
    $(aws sts get-caller-identity --query Account --output text).dkr.ecr.$AWS_REGION.amazonaws.com/${PROJECT_NAME}/worker:latest
docker push $(aws sts get-caller-identity --query Account --output text).dkr.ecr.$AWS_REGION.amazonaws.com/${PROJECT_NAME}/worker:latest

# Deploy infrastructure
echo -e "${YELLOW}Deploying infrastructure with Terraform...${NC}"
cd terraform/environments/${ENVIRONMENT}

terraform init
terraform plan -out=tfplan
echo -e "${YELLOW}Review the plan above. Press Enter to continue or Ctrl+C to cancel${NC}"
read

terraform apply tfplan

# Get outputs
CLUSTER_NAME=$(terraform output -raw ecs_cluster_name)
FRONTEND_BUCKET=$(terraform output -raw frontend_bucket)

# Deploy frontend
echo -e "${YELLOW}Deploying frontend...${NC}"
cd ../../../frontend
npm ci
npm run build

aws s3 sync build/ s3://${FRONTEND_BUCKET} --delete

# Invalidate CloudFront cache
DISTRIBUTION_ID=$(aws cloudfront list-distributions \
    --query "DistributionList.Items[?contains(Aliases.Items, '${PROJECT_NAME}')].Id" \
    --output text)

if [ ! -z "$DISTRIBUTION_ID" ]; then
    echo -e "${YELLOW}Invalidating CloudFront cache...${NC}"
    aws cloudfront create-invalidation \
        --distribution-id $DISTRIBUTION_ID \
        --paths "/*"
fi

# Update ECS services
echo -e "${YELLOW}Updating ECS services...${NC}"
for service in api worker; do
    echo -e "${YELLOW}Updating ${service}...${NC}"
    aws ecs update-service \
        --cluster ${CLUSTER_NAME} \
        --service ${service} \
        --force-new-deployment \
        --region ${AWS_REGION}
done

echo -e "${GREEN}Deployment completed successfully!${NC}"
```

#### scripts/deploy/rollback.sh

```bash
# scripts/deploy/rollback.sh

#!/bin/bash

set -e

RED='\033[0;31m'
GREEN='\033[0;32m'
YELLOW='\033[1;33m'
NC='\033[0m'

ENVIRONMENT=${1:-dev}
AWS_REGION=${AWS_REGION:-us-east-1}
CLUSTER_NAME="${2}"
SERVICE_NAME="${3}"

if [ -z "$CLUSTER_NAME" ] || [ -z "$SERVICE_NAME" ]; then
    echo -e "${RED}Usage: $0 <environment> <cluster-name> <service-name>${NC}"
    exit 1
fi

echo -e "${YELLOW}Rolling back ${SERVICE_NAME} in ${CLUSTER_NAME}...${NC}"

# Get current task definition
TASK_DEFINITION=$(aws ecs describe-services \
    --cluster ${CLUSTER_NAME} \
    --services ${SERVICE_NAME} \
    --query 'services[0].taskDefinition' \
    --output text \
    --region ${AWS_REGION})

# Get task definition family
FAMILY=$(echo $TASK_DEFINITION | cut -d'/' -f2 | cut -d':' -f1)

# Get previous revision
CURRENT_REVISION=$(echo $TASK_DEFINITION | cut -d':' -f2)
PREVIOUS_REVISION=$((CURRENT_REVISION - 1))

if [ $PREVIOUS_REVISION -lt 1 ]; then
    echo -e "${RED}No previous revision found${NC}"
    exit 1
fi

PREVIOUS_TASK_DEF="${FAMILY}:${PREVIOUS_REVISION}"

echo -e "${YELLOW}Rolling back from revision ${CURRENT_REVISION} to ${PREVIOUS_REVISION}${NC}"
echo -e "${YELLOW}Task definition: ${PREVIOUS_TASK_DEF}${NC}"

# Update service to use previous task definition
aws ecs update-service \
    --cluster ${CLUSTER_NAME} \
    --service ${SERVICE_NAME} \
    --task-definition ${PREVIOUS_TASK_DEF} \
    --region ${AWS_REGION}

echo -e "${GREEN}Rollback initiated. Waiting for service to stabilize...${NC}"

# Wait for service to stabilize
aws ecs wait services-stable \
    --cluster ${CLUSTER_NAME} \
    --services ${SERVICE_NAME} \
    --region ${AWS_REGION}

echo -e "${GREEN}Rollback completed successfully!${NC}"
```
#### scripts/deploy/db-migration.sh
```bash
# scripts/deploy/db-migration.sh

#!/bin/bash

set -e

RED='\033[0;31m'
GREEN='\033[0;32m'
YELLOW='\033[1;33m'
NC='\033[0m'

ENVIRONMENT=${1:-dev}
AWS_REGION=${AWS_REGION:-us-east-1}

echo -e "${YELLOW}Running database migrations for ${ENVIRONMENT}...${NC}"

# Get database credentials from Secrets Manager
SECRET_ARN=$(aws secretsmanager list-secrets \
    --filters Key=name,Values=myapp-${ENVIRONMENT}-db-password \
    --query 'SecretList[0].ARN' \
    --output text \
    --region ${AWS_REGION})

if [ "$SECRET_ARN" == "None" ]; then
    echo -e "${RED}Database secret not found${NC}"
    exit 1
fi

# Get secret value
SECRET_JSON=$(aws secretsmanager get-secret-value \
    --secret-id ${SECRET_ARN} \
    --query SecretString \
    --output text \
    --region ${AWS_REGION})

# Extract connection details
DB_HOST=$(echo $SECRET_JSON | jq -r '.host')
DB_PORT=$(echo $SECRET_JSON | jq -r '.port')
DB_NAME=$(echo $SECRET_JSON | jq -r '.dbname')
DB_USER=$(echo $SECRET_JSON | jq -r '.username')
DB_PASS=$(echo $SECRET_JSON | jq -r '.password')

# Export for migration tool
export DATABASE_URL="postgresql://${DB_USER}:${DB_PASS}@${DB_HOST}:${DB_PORT}/${DB_NAME}"

# Run migrations
echo -e "${YELLOW}Applying migrations...${NC}"
npm run migrate

echo -e "${GREEN}Migrations completed successfully!${NC}"
```

#### scripts/backup/backup-rds.sh
```bash
# scripts/backup/backup-rds.sh

#!/bin/bash

set -e

RED='\033[0;31m'
GREEN='\033[0;32m'
YELLOW='\033[1;33m'
NC='\033[0m'

CLUSTER_ID=${1}
AWS_REGION=${AWS_REGION:-us-east-1}
TIMESTAMP=$(date +%Y%m%d-%H%M%S)

if [ -z "$CLUSTER_ID" ]; then
    echo -e "${RED}Usage: $0 <cluster-id>${NC}"
    exit 1
fi

echo -e "${YELLOW}Creating snapshot of ${CLUSTER_ID}...${NC}"

SNAPSHOT_ID="${CLUSTER_ID}-manual-${TIMESTAMP}"

aws rds create-db-cluster-snapshot \
    --db-cluster-identifier ${CLUSTER_ID} \
    --db-cluster-snapshot-identifier ${SNAPSHOT_ID} \
    --region ${AWS_REGION}

echo -e "${YELLOW}Waiting for snapshot to complete...${NC}"

aws rds wait db-cluster-snapshot-available \
    --db-cluster-snapshot-identifier ${SNAPSHOT_ID} \
    --region ${AWS_REGION}

echo -e "${GREEN}Snapshot ${SNAPSHOT_ID} created successfully!${NC}"

# Clean up old snapshots (keep last 7)
echo -e "${YELLOW}Cleaning up old snapshots...${NC}"

OLD_SNAPSHOTS=$(aws rds describe-db-cluster-snapshots \
    --db-cluster-identifier ${CLUSTER_ID} \
    --snapshot-type manual \
    --query "DBClusterSnapshots | sort_by(@, &SnapshotCreateTime) | [0:-7].DBClusterSnapshotIdentifier" \
    --output text \
    --region ${AWS_REGION})

for snapshot in $OLD_SNAPSHOTS; do
    echo -e "${YELLOW}Deleting old snapshot: ${snapshot}${NC}"
    aws rds delete-db-cluster-snapshot \
        --db-cluster-snapshot-identifier ${snapshot} \
        --region ${AWS_REGION}
done

echo -e "${GREEN}Backup and cleanup completed!${NC}"
```


Comprehensive README и документацию:
```markdown
# Production Three-Tier Application

Production-ready three-tier application на AWS с полным CI/CD pipeline, мониторингом и best practices.


## 🏗 Архитектура


┌─────────────────────────────────────────────────────────────┐
│                     Presentation Tier                        │
│                                                              │
│  ┌──────────────┐      ┌─────────────────────────────────┐ │
│  │  CloudFront  │──────│   S3 Static Website Hosting     │ │
│  └──────────────┘      └─────────────────────────────────┘ │
│         │                                                    │
│         │              ┌─────────────────────────────────┐  │
│         └──────────────│  Application Load Balancer      │  │
│                        └─────────────────────────────────┘  │
└────────────────────────────────┬────────────────────────────┘
                                 │
┌────────────────────────────────┼────────────────────────────┐
│                     Application Tier                         │
│                                │                              │
│  ┌─────────────────────────────┴─────────────────────────┐  │
│  │           ECS Fargate Cluster                          │  │
│  │                                                         │  │
│  │  ┌──────────────┐  ┌──────────────┐  ┌─────────────┐ │  │
│  │  │  API Service │  │Worker Service│  │Lambda Funcs │ │  │
│  │  │  (3 tasks)   │  │  (2 tasks)   │  │             │ │  │
│  │  └──────────────┘  └──────────────┘  └─────────────┘ │  │
│  └─────────────────────────────────────────────────────┘  │
└────────────────────────────────┬────────────────────────────┘
                                 │
┌────────────────────────────────┼────────────────────────────┐
│                       Data Tier                              │
│                                │                              │
│  ┌─────────────────┐  ┌────────┴──────────┐  ┌───────────┐ │
│  │ Aurora          │  │  DynamoDB Cache   │  │ElastiCache│ │
│  │ PostgreSQL      │  │                   │  │   Redis   │ │
│  │ (Multi-AZ)      │  │                   │  │           │ │
│  └─────────────────┘  └───────────────────┘  └───────────┘ │
│                                                              │
│  ┌───────────────────────────────────────────────────────┐  │
│  │               S3 Object Storage                        │  │
│  └───────────────────────────────────────────────────────┘  │
└─────────────────────────────────────────────────────────────┘
```

### Компоненты

**Presentation Tier:**

- CloudFront CDN для глобальной доставки контента
- S3 для хостинга статических файлов React
- WAF для защиты от веб-атак
- SSL/TLS через ACM

**Application Tier:**

- ECS Fargate для containerized workloads
- Application Load Balancer с path-based routing
- Auto Scaling на основе CPU/Memory
- Service Discovery через AWS Cloud Map

**Data Tier:**

- Aurora PostgreSQL Serverless v2 (Multi-AZ)
- DynamoDB для кэширования
- ElastiCache Redis для сессий
- S3 для хранения объектов

**Supporting Services:**

- Route53 для DNS
- Secrets Manager для credentials
- CloudWatch для логов и метрик
- SNS для уведомлений
- KMS для шифрования

## 📦 Требования

### Локальная разработка

- **Docker** >= 20.10
- **Terraform** >= 1.5.0
- **AWS CLI** >= 2.0
- **Node.js** >= 20.x
- **npm** >= 10.x

### AWS Account

- Активный AWS аккаунт
- IAM пользователь с необходимыми правами
- Route53 hosted zone (опционально)
- ACM сертификат для домена

### Необходимые AWS сервисы

```bash
# Проверка доступа к AWS
aws sts get-caller-identity

# Создание ECR репозиториев
aws ecr create-repository --repository-name myapp/api
aws ecr create-repository --repository-name myapp/worker

# Создание S3 bucket для Terraform state
aws s3 mb s3://myapp-terraform-state --region us-east-1
aws s3api put-bucket-versioning \
    --bucket myapp-terraform-state \
    --versioning-configuration Status=Enabled

# Создание DynamoDB для state locking
aws dynamodb create-table \
    --table-name terraform-state-lock \
    --attribute-definitions AttributeName=LockID,AttributeType=S \
    --key-schema AttributeName=LockID,KeyType=HASH \
    --billing-mode PAY_PER_REQUEST
```

## 🚀 Быстрый старт

### 1. Clone репозитория

```bash
git clone https://github.com/yourorg/production-three-tier-app.git
cd production-three-tier-app
```

### 2. Конфигурация

```bash
# Создать terraform.tfvars из примера
cd terraform/environments/prod
cp terraform.tfvars.example terraform.tfvars

# Отредактировать переменные
vim terraform.tfvars
```

**Минимально необходимые переменные:**

```hcl
aws_region         = "us-east-1"
project_name       = "myapp"
domain_name        = "example.com"
acm_certificate_arn = "arn:aws:acm:us-east-1:123456789012:certificate/..."
route53_zone_id    = "Z1234567890ABC"
alert_email        = "alerts@example.com"
```

### 3. Инициализация Terraform

```bash
terraform init
terraform validate
terraform plan
```

### 4. Деплой инфраструктуры

```bash
# Production деплой требует approval
terraform apply

# Получение outputs
terraform output
```

### 5. Build и Push Docker images

```bash
# Login to ECR
aws ecr get-login-password --region us-east-1 | \
    docker login --username AWS --password-stdin \
    $(aws sts get-caller-identity --query Account --output text).dkr.ecr.us-east-1.amazonaws.com

# Build and push
./scripts/deploy/deploy.sh prod
```

### 6. Деплой приложения

```bash
# Через GitHub Actions (рекомендуется)
git push origin main

# Или вручную
./scripts/deploy/deploy.sh prod
```

## 📁 Структура проекта

```
production-three-tier-app/
├── terraform/
│   ├── modules/
│   │   ├── networking/       # VPC, Subnets, NAT Gateway
│   │   ├── compute/          # ECS, ALB, Auto Scaling
│   │   ├── database/         # RDS, DynamoDB, Redis
│   │   ├── storage/          # S3, CloudFront
│   │   ├── security/         # WAF, Security Groups
│   │   └── monitoring/       # CloudWatch, Alarms
│   └── environments/
│       ├── dev/              # Development environment
│       ├── staging/          # Staging environment
│       └── prod/             # Production environment
├── docker/
│   ├── backend/              # Backend API Dockerfile
│   ├── worker/               # Worker Dockerfile
│   └── nginx/                # NGINX configuration
├── frontend/                 # React application
├── backend/                  # Node.js API
├── worker/                   # Background worker
├── scripts/
│   ├── deploy/              # Deployment scripts
│   ├── backup/              # Backup scripts
│   └── monitoring/          # Monitoring scripts
├── .github/
│   └── workflows/           # CI/CD pipelines
└── docs/
    ├── architecture/        # Architecture diagrams
    ├── runbooks/           # Operational runbooks
    └── api/                # API documentation
```

## 🔄 Deployment

### CI/CD Pipeline

Pipeline автоматически выполняет:

1. **Test Stage**
    
    - Unit tests
    - Integration tests
    - Linting
2. **Security Stage**
    
    - Trivy vulnerability scanning
    - OWASP dependency check
    - Secret scanning
3. **Build Stage**
    
    - Docker image build
    - Multi-arch support (amd64)
    - Image caching для быстрых builds
4. **Infrastructure Stage**
    
    - Terraform plan
    - Manual approval для production
    - Terraform apply
5. **Deploy Stage**
    
    - ECS service updates
    - Frontend deploy to S3
    - CloudFront invalidation
6. **Smoke Tests**
    
    - Health checks
    - API endpoint tests
    - Frontend availability

### Manual Deployment

```bash
# Деплой всего стека
./scripts/deploy/deploy.sh prod

# Деплой только frontend
aws s3 sync frontend/build/ s3://myapp-prod-frontend
aws cloudfront create-invalidation --distribution-id XXX --paths "/*"

# Деплой только backend
aws ecs update-service \
    --cluster myapp-prod \
    --service api \
    --force-new-deployment

# Database migration
./scripts/deploy/db-migration.sh prod
```

### Rollback

```bash
# Откат ECS service к предыдущей версии
./scripts/deploy/rollback.sh prod myapp-prod api

# Откат через Terraform
cd terraform/environments/prod
terraform plan -target=module.compute
terraform apply -target=module.compute
```

## 📊 Мониторинг

### CloudWatch Dashboard

Доступ к dashboard:

```bash
DASHBOARD_URL=$(terraform output -raw cloudwatch_dashboard_url)
echo $DASHBOARD_URL
```

### Метрики

**ECS Metrics:**

- CPU Utilization
- Memory Utilization
- Task Count
- Service Deployment Status

**ALB Metrics:**

- Request Count
- Target Response Time
- HTTP 4xx/5xx Errors
- Healthy/Unhealthy Target Count

**Database Metrics:**

- CPU Utilization
- Database Connections
- Read/Write Latency
- Storage Usage

**CloudFront Metrics:**

- Requests
- Data Transfer
- Error Rate
- Cache Hit Rate

### Alarms

Настроенные CloudWatch alarms:

```bash
# Список всех alarms
aws cloudwatch describe-alarms \
    --query "MetricAlarms[?starts_with(AlarmName, 'myapp-prod')]"

# Test alarm
aws cloudwatch set-alarm-state \
    --alarm-name myapp-prod-api-cpu-high \
    --state-value ALARM \
    --state-reason "Testing alarm"
```

### Логи

```bash
# Просмотр логов ECS
aws logs tail /ecs/myapp-prod --follow

# Поиск ошибок
aws logs filter-log-events \
    --log-group-name /ecs/myapp-prod \
    --filter-pattern "ERROR"

# Export логов
aws logs create-export-task \
    --log-group-name /ecs/myapp-prod \
    --from $(date -d '1 day ago' +%s)000 \
    --to $(date +%s)000 \
    --destination myapp-logs-archive
```

## 🔐 Security

### Secrets Management

```bash
# Создание secret
aws secretsmanager create-secret \
    --name myapp/prod/api-key \
    --secret-string "your-secret-value"

# Rotation
aws secretsmanager rotate-secret \
    --secret-id myapp/prod/db-password \
    --rotation-lambda-arn arn:aws:lambda:...
```

### IAM Roles

- **Task Execution Role**: Права для pull образов и логов
- **Task Role**: Права приложения (S3, DynamoDB, Secrets)
- **Auto Scaling Role**: Права для scaling операций

### Network Security

- Private subnets для application и database tiers
- Security Groups с минимальными правами
- NACLs для дополнительной защиты
- VPC Flow Logs включены

### Encryption

- **At Rest**: KMS encryption для RDS, S3, EBS
- **In Transit**: TLS 1.2+ для всех соединений
- **Secrets**: Encrypted в Secrets Manager

## 🛠 Troubleshooting

### Частые проблемы

**ECS Tasks не запускаются:**

```bash
# Проверка event logs
aws ecs describe-services \
    --cluster myapp-prod \
    --services api \
    --query 'services[0].events'

# Проверка task logs
aws logs get-log-events \
    --log-group-name /ecs/myapp-prod \
    --log-stream-name ecs/api/TASK_ID
```

**Database connection issues:**

```bash
# Проверка security groups
aws ec2 describe-security-groups \
    --group-ids sg-xxx

# Test connection из ECS
aws ecs execute-command \
    --cluster myapp-prod \
    --task TASK_ID \
    --container api \
    --command "nc -zv DB_HOST 5432" \
    --interactive
```

**High latency:**

```bash
# Проверка CloudWatch metrics
aws cloudwatch get-metric-statistics \
    --namespace AWS/ApplicationELB \
    --metric-name TargetResponseTime \
    --dimensions Name=LoadBalancer,Value=app/myapp-prod/xxx \
    --statistics Average \
    --start-time $(date -d '1 hour ago' -u +%Y-%m-%dT%H:%M:%S) \
    --end-time $(date -u +%Y-%m-%dT%H:%M:%S) \
    --period 300

# Enable X-Ray tracing для debugging
aws xray get-trace-summaries \
    --start-time $(date -d '1 hour ago' +%s) \
    --end-time $(date +%s)
```

### Debugging Tools

```bash
# SSH в running task
aws ecs execute-command \
    --cluster myapp-prod \
    --task TASK_ID \
    --container api \
    --command "/bin/bash" \
    --interactive

# Performance Insights
aws rds describe-db-cluster-performance-insights \
    --db-cluster-identifier myapp-prod-aurora

# CloudWatch Insights queries
aws logs start-query \
    --log-group-name /ecs/myapp-prod \
    --start-time $(date -d '1 hour ago' +%s) \
    --end-time $(date +%s) \
    --query-string 'fields @timestamp, @message | filter @message like /ERROR/'
```

## 📈 Scaling

### Auto Scaling Configuration

**ECS Service Auto Scaling:**

- Target CPU: 70%
- Target Memory: 80%
- Scale out cooldown: 60s
- Scale in cooldown: 300s

**Aurora Auto Scaling:**

- Min ACU: 0.5
- Max ACU: 8
- Auto pause: После 5 минут inactivity

### Manual Scaling

```bash
# Scale ECS service
aws ecs update-service \
    --cluster myapp-prod \
    --service api \
    --desired-count 5

# Scale Aurora
aws rds modify-db-cluster \
    --db-cluster-identifier myapp-prod-aurora \
    --serverless-v2-scaling-configuration MinCapacity=1,MaxCapacity=16
```

## 🔄 Backup & Recovery

### Automated Backups

- **RDS**: Daily snapshots, retention 30 days
- **DynamoDB**: Point-in-time recovery enabled
- **S3**: Versioning enabled

### Manual Backup

```bash
# RDS snapshot
./scripts/backup/backup-rds.sh myapp-prod-aurora

# Export DynamoDB table
aws dynamodb export-table-to-point-in-time \
    --table-arn arn:aws:dynamodb:... \
    --s3-bucket myapp-backups \
    --s3-prefix dynamodb-backup/
```

### Recovery

```bash
# Restore RDS from snapshot
aws rds restore-db-cluster-from-snapshot \
    --db-cluster-identifier myapp-prod-aurora-restored \
    --snapshot-identifier snapshot-id \
    --engine aurora-postgresql

# Restore DynamoDB from backup
aws dynamodb restore-table-from-backup \
    --target-table-name myapp-cache-restored \
    --backup-arn arn:aws:dynamodb:...
```

## 📚 Documentation

- [Architecture Decision Records](https://claude.ai/chat/docs/architecture/ADR.md)
- [API Documentation](https://claude.ai/chat/docs/api/README.md)
- [Runbooks](https://claude.ai/chat/docs/runbooks/README.md)
- [Database Schema](https://claude.ai/chat/docs/database/SCHEMA.md)

## 🤝 Contributing

См. [CONTRIBUTING.md](https://claude.ai/chat/CONTRIBUTING.md) для деталей.

## 📝 License

MIT License - см. [LICENSE](https://claude.ai/chat/LICENSE)

## 📧 Support

- Email: support@example.com
- Slack: #myapp-support
- On-call: PagerDuty integration

# Operational Runbook

## 🚨 Incident Response

### High CPU Utilization

**Symptoms:**

- CloudWatch alarm: `myapp-prod-api-cpu-high`
- Slow response times
- Increased error rates

**Investigation:**

```bash
# 1. Check current metrics
aws cloudwatch get-metric-statistics \
    --namespace AWS/ECS \
    --metric-name CPUUtilization \
    --dimensions Name=ClusterName,Value=myapp-prod Name=ServiceName,Value=api \
    --statistics Average Maximum \
    --start-time $(date -d '1 hour ago' -u +%Y-%m-%dT%H:%M:%S) \
    --end-time $(date -u +%Y-%m-%dT%H:%M:%S) \
    --period 300

# 2. Check running tasks
aws ecs list-tasks --cluster myapp-prod --service-name api

# 3. Get task details
aws ecs describe-tasks --cluster myapp-prod --tasks TASK_ARN

# 4. Check logs for errors
aws logs filter-log-events \
    --log-group-name /ecs/myapp-prod \
    --filter-pattern "ERROR" \
    --start-time $(date -d '15 minutes ago' +%s)000
```

**Resolution:**

```bash
# Option 1: Scale up service
aws ecs update-service \
    --cluster myapp-prod \
    --service api \
    --desired-count 5

# Option 2: Increase task resources
# Update task definition with higher CPU/memory
aws ecs register-task-definition --cli-input-json file://task-def-updated.json

# Option 3: Restart service
aws ecs update-service \
    --cluster myapp-prod \
    --service api \
    --force-new-deployment
```

### Database Connection Issues

**Symptoms:**

- Connection timeout errors
- `too many connections` errors
- Failed health checks

**Investigation:**

```bash
# 1. Check database status
aws rds describe-db-clusters \
    --db-cluster-identifier myapp-prod-aurora \
    --query 'DBClusters[0].Status'

# 2. Check active connections
aws cloudwatch get-metric-statistics \
    --namespace AWS/RDS \
    --metric-name DatabaseConnections \
    --dimensions Name=DBClusterIdentifier,Value=myapp-prod-aurora \
    --statistics Sum Maximum \
    --start-time $(date -d '1 hour ago' -u +%Y-%m-%dT%H:%M:%S) \
    --end-time $(date -u +%Y-%m-%dT%H:%M:%S) \
    --period 300

# 3. Check security groups
aws ec2 describe-security-group-rules \
    --filters "Name=group-id,Values=sg-xxx"
```

**Resolution:**

```bash
# 1. Verify network connectivity
# Connect to ECS task
aws ecs execute-command \
    --cluster myapp-prod \
    --task TASK_ID \
    --container api \
    --command "nc -zv DB_ENDPOINT 5432" \
    --interactive

# 2. Check connection pool settings
# Update application environment variables
aws ecs update-service \
    --cluster myapp-prod \
    --service api \
    --force-new-deployment

# 3. Scale up database if needed
aws rds modify-db-cluster \
    --db-cluster-identifier myapp-prod-aurora \
    --serverless-v2-scaling-configuration MaxCapacity=16
```

### Application Not Responding

**Symptoms:**

- All health checks failing
- 503 errors from ALB
- No logs from application

**Investigation:**

```bash
# 1. Check ALB target health
aws elbv2 describe-target-health \
    --target-group-arn TARGET_GROUP_ARN

# 2. Check ECS service events
aws ecs describe-services \
    --cluster myapp-prod \
    --services api \
    --query 'services[0].events[0:10]'

# 3. Check recent deployments
aws ecs describe-services \
    --cluster myapp-prod \
    --services api \
    --query 'services[0].deployments'
```

**Resolution:**

```bash
# 1. Rollback to previous version
./scripts/deploy/rollback.sh prod myapp-prod api

# 2. Check application logs
aws logs tail /ecs/myapp-prod --follow --filter-pattern "ERROR"

# 3. Force new deployment
aws ecs update-service \
    --cluster myapp-prod \
    --service api \
    --force-new-deployment
```

## 🔄 Common Operations

### Deploy New Version

```bash
# 1. Build and tag image
IMAGE_TAG=$(git rev-parse --short HEAD)
docker build -t myapp/api:$IMAGE_TAG -f docker/backend/Dockerfile .

# 2. Push to ECR
AWS_ACCOUNT_ID=$(aws sts get-caller-identity --query Account --output text)
ECR_REGISTRY="${AWS_ACCOUNT_ID}.dkr.ecr.us-east-1.amazonaws.com"

aws ecr get-login-password --region us-east-1 | \
    docker login --username AWS --password-stdin $ECR_REGISTRY

docker tag myapp/api:$IMAGE_TAG $ECR_REGISTRY/myapp/api:$IMAGE_TAG
docker push $ECR_REGISTRY/myapp/api:$IMAGE_TAG

# 3. Update ECS service
aws ecs update-service \
    --cluster myapp-prod \
    --service api \
    --force-new-deployment
```

### Scale Application

```bash
# Scale up
aws ecs update-service \
    --cluster myapp-prod \
    --service api \
    --desired-count 10

# Scale down
aws ecs update-service \
    --cluster myapp-prod \
    --service api \
    --desired-count 2

# Check scaling status
aws ecs describe-services \
    --cluster myapp-prod \
    --services api \
    --query 'services[0].[desiredCount,runningCount,pendingCount]'
```

### Database Maintenance

```bash
# Create manual snapshot
aws rds create-db-cluster-snapshot \
    --db-cluster-identifier myapp-prod-aurora \
    --db-cluster-snapshot-identifier manual-$(date +%Y%m%d-%H%M%S)

# Apply pending maintenance
aws rds modify-db-cluster \
    --db-cluster-identifier myapp-prod-aurora \
    --apply-immediately

# Upgrade minor version
aws rds modify-db-cluster \
    --db-cluster-identifier myapp-prod-aurora \
    --engine-version 15.5 \
    --apply-immediately
```

### Update Configuration

```bash
# Update environment variables
aws ecs register-task-definition \
    --cli-input-json file://updated-task-def.json

# Update secrets
aws secretsmanager update-secret \
    --secret-id myapp/prod/api-key \
    --secret-string "new-value"

# Force service update to pick up changes
aws ecs update-service \
    --cluster myapp-prod \
    --service api \
    --force-new-deployment
```

## 📊 Monitoring Tasks

### Daily Health Check

```bash
#!/bin/bash
# scripts/monitoring/daily-health-check.sh

echo "=== Daily Health Check ==="
echo

# Check ECS services
echo "ECS Services Status:"
aws ecs describe-services \
    --cluster myapp-prod \
    --services api worker \
    --query 'services[].[serviceName,status,runningCount,desiredCount]' \
    --output table

# Check database
echo -e "\nRDS Status:"
aws rds describe-db-clusters \
    --db-cluster-identifier myapp-prod-aurora \
    --query 'DBClusters[0].[DBClusterIdentifier,Status,Engine,EngineVersion]' \
    --output table

# Check alarms
echo -e "\nActive Alarms:"
aws cloudwatch describe-alarms \
    --state-value ALARM \
    --query 'MetricAlarms[].[AlarmName,StateValue,StateReason]' \
    --output table

# Check costs
echo -e "\nMonthly Costs (MTD):"
aws ce get-cost-and-usage \
    --time-period Start=$(date -d "$(date +%Y-%m-01)" +%Y-%m-%d),End=$(date +%Y-%m-%d) \
    --granularity MONTHLY \
    --metrics BlendedCost \
    --group-by Type=SERVICE \
    --query 'ResultsByTime[0].Groups[?Metrics.BlendedCost.Amount>`10`].[Keys[0],Metrics.BlendedCost.Amount]' \
    --output table
```

### Weekly Maintenance

```bash
# Update all tasks to latest version
for service in api worker; do
    echo "Updating $service..."
    aws ecs update-service \
        --cluster myapp-prod \
        --service $service \
        --force-new-deployment
done

# Clean up old ECR images
for repo in api worker; do
    echo "Cleaning up $repo..."
    aws ecr list-images \
        --repository-name myapp/$repo \
        --filter tagStatus=UNTAGGED \
        --query 'imageIds[*]' \
        --output json | \
    jq -r '.[] | .imageDigest' | \
    while read digest; do
        aws ecr batch-delete-image \
            --repository-name myapp/$repo \
            --image-ids imageDigest=$digest
    done
done

# Vacuum database
./scripts/maintenance/vacuum-db.sh prod
```

## 🔐 Security Tasks

### Rotate Credentials

```bash
# Rotate database password
aws secretsmanager rotate-secret \
    --secret-id myapp/prod/db-password

# Rotate API keys
aws secretsmanager update-secret \
    --secret-id myapp/prod/api-key \
    --secret-string "$(openssl rand -base64 32)"

# Update service to use new credentials
aws ecs update-service \
    --cluster myapp-prod \
    --service api \
    --force-new-deployment
```

### Security Audit

```bash
# Check security groups
aws ec2 describe-security-groups \
    --filters "Name=tag:Project,Values=myapp" \
    --query 'SecurityGroups[].[GroupId,GroupName,IpPermissions]'

# Check IAM roles
aws iam list-roles \
    --query 'Roles[?contains(RoleName, `myapp`)].[RoleName,CreateDate]' \
    --output table

# Check S3 bucket policies
for bucket in $(aws s3 ls | grep myapp | awk '{print $3}'); do
    echo "Bucket: $bucket"
    aws s3api get-bucket-policy --bucket $bucket 2>/dev/null || echo "No policy"
done

# Check encryption
aws rds describe-db-clusters \
    --db-cluster-identifier myapp-prod-aurora \
    --query 'DBClusters[0].StorageEncrypted'
```

## 📈 Performance Optimization

### Identify Slow Queries

```bash
# Enable Performance Insights if not enabled
aws rds modify-db-cluster \
    --db-cluster-identifier myapp-prod-aurora \
    --enable-performance-insights

# Get slow queries from Performance Insights
aws pi get-resource-metrics \
    --service-type RDS \
    --identifier db-XXXXX \
    --start-time $(date -d '1 hour ago' -u +%Y-%m-%dT%H:%M:%S) \
    --end-time $(date -u +%Y-%m-%dT%H:%M:%S) \
    --period-in-seconds 3600 \
    --metric-queries '[{"Metric":"db.load.avg"}]'
```

### Optimize CloudFront

```bash
# Check cache hit ratio
aws cloudfront get-distribution-metrics \
    --distribution-id DISTRIBUTION_ID \
    --start-time $(date -d '1 day ago' -u +%Y-%m-%dT%H:%M:%S) \
    --end-time $(date -u +%Y-%m-%dT%H:%M:%S) \
    --granularity ONE_HOUR \
    --metric CacheHitRate

# Update cache behaviors for better performance
aws cloudfront update-distribution \
    --id DISTRIBUTION_ID \
    --if-match ETAG \
    --distribution-config file://updated-config.json
```

## 🔄 Disaster Recovery

### Full System Recovery

```bash
# 1. Restore database from snapshot
SNAPSHOT_ID=$(aws rds describe-db-cluster-snapshots \
    --db-cluster-identifier myapp-prod-aurora \
    --query 'DBClusterSnapshots | sort_by(@, &SnapshotCreateTime) | [-1].DBClusterSnapshotIdentifier' \
    --output text)

aws rds restore-db-cluster-from-snapshot \
    --db-cluster-identifier myapp-prod-aurora-dr \
    --snapshot-identifier $SNAPSHOT_ID \
    --engine aurora-postgresql

# 2. Restore DynamoDB
aws dynamodb restore-table-to-point-in-time \
    --source-table-name myapp-prod-cache \
    --target-table-name myapp-prod-cache-dr \
    --restore-date-time $(date -d '1 hour ago' -u +%Y-%m-%dT%H:%M:%S)

# 3. Redeploy infrastructure
cd terraform/environments/prod
terraform apply -auto-approve

# 4. Deploy applications
./scripts/deploy/deploy.sh prod

# 5. Update DNS
aws route53 change-resource-record-sets \
    --hosted-zone-id ZONE_ID \
    --change-batch file://dns-update.json
```

## 📞 Escalation

### Contact Information

- **Level 1**: On-call engineer (PagerDuty)
- **Level 2**: Senior DevOps (Slack: @devops-team)
- **Level 3**: Platform Lead (Phone: +1-XXX-XXX-XXXX)
- **Level 4**: CTO (Email: cto@example.com)

### Escalation Criteria

- **Immediate**: Complete service outage, data loss
- **High**: Degraded performance affecting >50% users
- **Medium**: Single service degradation
- **Low**: Non-critical alerts

# 🎓 Модуль 17: Final Production Project - Summary

## ✅ Что мы построили

### 🏗 Infrastructure as Code (Terraform)

**6 модулей инфраструктуры:**

1. **Networking Module**
    
    - ✅ VPC с 3 AZ
    - ✅ Public, Private App, Private DB subnets
    - ✅ NAT Gateways для Multi-AZ
    - ✅ VPC Flow Logs
    - ✅ Route Tables и Associations
2. **Database Module**
    
    - ✅ Aurora PostgreSQL Serverless v2
    - ✅ Multi-AZ deployment
    - ✅ DynamoDB для кэширования
    - ✅ ElastiCache Redis для сессий
    - ✅ Automated backups (30 days)
    - ✅ KMS encryption
    - ✅ Secrets Manager integration
3. **Compute Module**
    
    - ✅ ECS Fargate Cluster
    - ✅ Application Load Balancer
    - ✅ Auto Scaling (CPU/Memory based)
    - ✅ Service Discovery (Cloud Map)
    - ✅ Health checks и Circuit breakers
    - ✅ Container Insights enabled
4. **Storage Module**
    
    - ✅ S3 для static hosting
    - ✅ CloudFront distribution
    - ✅ SSL/TLS через ACM
    - ✅ Object versioning
    - ✅ Lifecycle policies
    - ✅ Access logs
5. **Security Module**
    
    - ✅ Security Groups (least privilege)
    - ✅ WAF rules
    - ✅ KMS для encryption at rest
    - ✅ IAM roles и policies
    - ✅ Secrets rotation
6. **Monitoring Module**
    
    - ✅ CloudWatch Dashboards
    - ✅ Alarms для critical metrics
    - ✅ SNS notifications
    - ✅ Log aggregation
    - ✅ X-Ray tracing ready

### 🐳 Containerization

**Docker Images:**

- ✅ Multi-stage builds для optimization
- ✅ Security scanning (Trivy)
- ✅ Health checks встроены
- ✅ Non-root users
- ✅ Proper signal handling (dumb-init)
- ✅ Layer caching для быстрых builds

### 🔄 CI/CD Pipeline (GitHub Actions)

**7-stage pipeline:**

1. **Test** - Unit, Integration, Linting
2. **Security Scan** - Trivy, OWASP Dependency Check
3. **Build & Push** - Multi-arch Docker images
4. **Terraform Plan** - Infrastructure preview
5. **Terraform Apply** - Infrastructure deployment
6. **Deploy** - ECS и Frontend deployment
7. **Smoke Tests** - Post-deployment validation

### 📊 Observability

**Мониторинг:**

- ✅ Centralized logging (CloudWatch)
- ✅ Metrics collection (CloudWatch Metrics)
- ✅ Distributed tracing ready (X-Ray)
- ✅ Custom dashboards
- ✅ Alerting (SNS/Email/Slack)

**Metrics tracked:**

- Application: Requests, Errors, Latency
- Infrastructure: CPU, Memory, Network
- Database: Connections, Query performance
- Cost: Daily spending by service

## 🎯 Production-Ready Features

### High Availability

- ✅ Multi-AZ deployment
- ✅ Auto Scaling groups
- ✅ Health checks на всех уровнях
- ✅ Circuit breakers для graceful degradation
- ✅ Database failover (Aurora Multi-AZ)

### Security

- ✅ Encryption at rest (KMS)
- ✅ Encryption in transit (TLS 1.2+)
- ✅ Secrets management (Secrets Manager)
- ✅ IAM least privilege
- ✅ Security groups isolation
- ✅ WAF protection
- ✅ VPC Flow Logs

### Performance

- ✅ CDN (CloudFront)
- ✅ Caching layers (DynamoDB, Redis)
- ✅ Auto Scaling based on metrics
- ✅ Connection pooling
- ✅ Database read replicas ready

### Reliability

- ✅ Automated backups (RDS, DynamoDB)
- ✅ Point-in-time recovery
- ✅ Blue-green deployments ready
- ✅ Rollback procedures
- ✅ Disaster recovery plan

### Cost Optimization

- ✅ Fargate Spot для non-critical workloads
- ✅ Aurora Serverless auto-scaling
- ✅ S3 lifecycle policies
- ✅ CloudFront caching
- ✅ Right-sized instances

## 📝 Checklist для Production Launch

### Pre-Launch

**Infrastructure:**

- [ ] All Terraform modules tested
- [ ] Proper resource tags applied
- [ ] Backup policies configured
- [ ] Monitoring dashboards created
- [ ] Alarms configured и tested

**Security:**

- [ ] Security group rules reviewed
- [ ] WAF rules активированы
- [ ] SSL certificates validated
- [ ] Secrets rotation configured
- [ ] IAM policies reviewed
- [ ] Security scanning passed

**Application:**

- [ ] All tests passing (Unit, Integration, E2E)
- [ ] Performance testing completed
- [ ] Load testing passed
- [ ] Database migrations tested
- [ ] Health checks validated

**Documentation:**

- [ ] Architecture diagrams updated
- [ ] Runbooks created
- [ ] API documentation complete
- [ ] Disaster recovery plan documented
- [ ] On-call rotation configured

### Launch Day

**Deployment:**

- [ ] Final code review completed
- [ ] Database backed up
- [ ] Infrastructure deployed via Terraform
- [ ] Applications deployed via CI/CD
- [ ] Smoke tests passing
- [ ] DNS updated
- [ ] CloudFront cache warmed

**Validation:**

- [ ] All services healthy
- [ ] Monitoring dashboards showing data
- [ ] Logs flowing correctly
- [ ] No critical alarms
- [ ] Performance metrics acceptable
- [ ] User acceptance testing passed

### Post-Launch

**Day 1:**

- [ ] Monitor metrics continuously
- [ ] Check error rates
- [ ] Validate backups running
- [ ] Review logs for issues
- [ ] Gather user feedback

**Week 1:**

- [ ] Daily health checks
- [ ] Performance optimization
- [ ] Cost review
- [ ] Security audit
- [ ] Update documentation

**Month 1:**

- [ ] Monthly security review
- [ ] Cost optimization review
- [ ] Capacity planning
- [ ] Disaster recovery drill
- [ ] Team retrospective

## 💡 Key Learnings

### Infrastructure as Code

```hcl
# Модульная структура позволяет:
# - Переиспользование компонентов
# - Легкое тестирование
# - Независимое развертывание
# - Версионирование инфраструктуры
```

### Best Practices Применены

1. **Immutable Infrastructure**
    
    - Container images неизменяемы
    - Infrastructure через code
    - Никаких manual changes
2. **Security by Design**
    
    - Принцип наименьших привилегий
    - Encryption everywhere
    - Network isolation
    - Secret management
3. **Observability First**
    
    - Structured logging
    - Metrics для всего
    - Distributed tracing ready
    - Proper alerting
4. **Automation**
    
    - CI/CD для всего
    - Automated testing
    - Automated deployments
    - Automated rollbacks
5. **Cost Awareness**
    
    - Tagging strategy
    - Auto-scaling
    - Spot instances
    - Lifecycle policies

## 🚀 Next Steps

### Immediate

1. Review и customize для вашего use case
2. Установить все необходимые credentials
3. Deploy в dev environment
4. Run полный test suite
5. Validate monitoring и alerting

### Short Term (1-2 недели)

1. Настроить production domain
2. Configure DNS в Route53
3. Set up SSL certificates
4. Deploy в staging
5. Run load testing
6. Optimize performance

### Medium Term (1 месяц)

1. Production deployment
2. User acceptance testing
3. Disaster recovery drill
4. Security penetration testing
5. Cost optimization review

### Long Term (3+ месяца)

1. Implement advanced features:
    - Blue-green deployments
    - Canary releases
    - Feature flags
    - A/B testing
2. Add additional monitoring:
    - APM (Application Performance Monitoring)
    - RUM (Real User Monitoring)
    - Synthetic monitoring
3. Compliance:
    - SOC 2 если required
    - HIPAA если healthcare
    - PCI-DSS если payments

## 📊 Estimated Costs (Monthly)

### Development Environment

- ECS Fargate (2 tasks): ~$50
- RDS Aurora (Serverless): ~$30
- ElastiCache (t4g.micro): ~$15
- Other services: ~$20
- **Total: ~$115/month**

### Production Environment

- ECS Fargate (6 tasks): ~$150
- RDS Aurora (Serverless): ~$100
- ElastiCache (r7g.large): ~$80
- CloudFront: ~$50
- ALB: ~$20
- Other services: ~$50
- **Total: ~$450/month**

_Costs vary based on usage and data transfer_

## 🎓 Skills Mastered

После завершения этого модуля вы:

✅ Создали production-ready three-tier application ✅ Освоили Terraform modules и best practices ✅ Настроили complete CI/CD pipeline ✅ Реализовали comprehensive monitoring ✅ Применили security best practices ✅ Научились работать с AWS managed services ✅ Создали disaster recovery procedures ✅ Документировали всю инфраструктуру

## 🏆 Achievement Unlocked!

Вы создали **enterprise-grade cloud infrastructure**, которая:

- ⚡ Highly available (99.95% uptime)
- 🔒 Secure by design
- 📈 Auto-scales based on demand
- 💰 Cost-optimized
- 📊 Fully observable
- 🔄 Continuously deployed
- 📚 Well documented

**Congratulations! Вы готовы к production!** 🎉

---

## 📚 Additional Resources

- [AWS Well-Architected Framework](https://aws.amazon.com/architecture/well-architected/)
- [Terraform Best Practices](https://www.terraform-best-practices.com/)
- [12-Factor App](https://12factor.net/)
- [Docker Best Practices](https://docs.docker.com/develop/dev-best-practices/)
- [GitHub Actions Documentation](https://docs.github.com/en/actions)

## 🤝 Contributing

Нашли bug или хотите улучшить? Pull requests welcome!

---

**Happy Deploying! 🚀**
