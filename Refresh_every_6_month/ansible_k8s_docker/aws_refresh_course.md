# AWS Refresh: Ежегодный/Полугодовой курс для DevOps/SysAdmin

**Цель:** Освежить в памяти ключевые концепции AWS за 3-4 часа практики и узнать 1-2 новые продвинутые техники.

**Формат:** Каждый модуль состоит из:
1. **Краткой теории (Напоминалка)**: Самое главное, что вы могли забыть
2. **Практического задания**: Реальные задачи, которые нужно решить
3. **Бонусного задания (для роста)**: Задача посложнее или с использованием новой фичи

---

## Модуль 1: IAM и Безопасность (30 минут)

### 🎯 Напоминалка

**IAM Основы:**
```bash
# AWS CLI конфигурация
aws configure
# AWS Access Key ID: YOUR_KEY
# AWS Secret Access Key: YOUR_SECRET
# Default region: us-east-1
# Default output format: json

# Проверка текущего пользователя
aws sts get-caller-identity

# Проверка прав
aws iam get-user
aws iam list-attached-user-policies --user-name username
```

**IAM Users, Groups, Roles:**
```bash
# Создание пользователя
aws iam create-user --user-name dev-user

# Создание группы
aws iam create-group --group-name developers

# Добавление пользователя в группу
aws iam add-user-to-group --user-name dev-user --group-name developers

# Присоединение политики к группе
aws iam attach-group-policy \
  --group-name developers \
  --policy-arn arn:aws:iam::aws:policy/PowerUserAccess

# Создание роли
aws iam create-role \
  --role-name EC2-S3-Role \
  --assume-role-policy-document file://trust-policy.json

# Присоединение политики к роли
aws iam attach-role-policy \
  --role-name EC2-S3-Role \
  --policy-arn arn:aws:iam::aws:policy/AmazonS3ReadOnlyAccess
```

**Политики IAM (JSON):**
```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": [
        "s3:GetObject",
        "s3:ListBucket"
      ],
      "Resource": [
        "arn:aws:s3:::my-bucket",
        "arn:aws:s3:::my-bucket/*"
      ]
    }
  ]
}
```

**Trust Policy (для роли):**
```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Principal": {
        "Service": "ec2.amazonaws.com"
      },
      "Action": "sts:AssumeRole"
    }
  ]
}
```

**MFA для пользователей:**
```bash
# Включение MFA (через консоль или CLI)
aws iam enable-mfa-device \
  --user-name username \
  --serial-number arn:aws:iam::123456789012:mfa/username \
  --authentication-code-1 123456 \
  --authentication-code-2 789012

# Проверка MFA
aws iam list-mfa-devices --user-name username
```

**Лучшие практики IAM:**
- ✅ Никогда не используй root аккаунт для повседневных задач
- ✅ Включи MFA для всех пользователей
- ✅ Следуй принципу минимальных привилегий
- ✅ Используй роли вместо долгосрочных ключей где возможно
- ✅ Ротируй ключи доступа регулярно
- ✅ Используй AWS Organizations для мультиаккаунт управления

**AWS CLI профили:**
```bash
# Конфигурация нескольких профилей
aws configure --profile prod
aws configure --profile dev

# Использование профиля
aws s3 ls --profile prod

# ~/.aws/credentials
[default]
aws_access_key_id = YOUR_KEY
aws_secret_access_key = YOUR_SECRET

[prod]
aws_access_key_id = PROD_KEY
aws_secret_access_key = PROD_SECRET

# ~/.aws/config
[default]
region = us-east-1
output = json

[profile prod]
region = us-west-2
output = json
```

### 💻 Задание

1. Создай IAM пользователя через CLI:
   ```bash
   aws iam create-user --user-name test-user
   ```

2. Создай группу "readonly-users" и присоедини политику ReadOnlyAccess

3. Добавь пользователя в группу

4. Создай кастомную политику, которая разрешает только чтение S3:
   ```bash
   aws iam create-policy \
     --policy-name S3ReadOnly \
     --policy-document file://s3-readonly-policy.json
   ```

5. Проверь какие политики присоединены к пользователю:
   ```bash
   aws iam list-attached-user-policies --user-name test-user
   aws iam list-groups-for-user --user-name test-user
   ```

6. Создай роль для EC2 с доступом к S3

7. Проверь свои текущие права:
   ```bash
   aws iam get-user
   aws sts get-caller-identity
   ```

### 🚀 Бонус (новое)

- Настрой AWS Organizations и создай Organizational Unit (OU)
- Используй Service Control Policies (SCPs) для ограничения действий на уровне OU
- Настрой IAM Access Analyzer для поиска внешних доступов
- Создай IAM роль с условиями (Condition) для доступа только из определенной IP сети
- Используй AWS CloudTrail для аудита действий IAM

---

## Модуль 2: EC2 и Сети (35 минут)

### 🎯 Напоминалка

**EC2 Basics:**
```bash
# Список инстансов
aws ec2 describe-instances

# Запуск инстанса
aws ec2 run-instances \
  --image-id ami-0c55b159cbfafe1f0 \
  --instance-type t2.micro \
  --key-name MyKeyPair \
  --security-group-ids sg-1234567890abcdef0 \
  --subnet-id subnet-1234567890abcdef0 \
  --tag-specifications 'ResourceType=instance,Tags=[{Key=Name,Value=MyInstance}]'

# Остановка инстанса
aws ec2 stop-instances --instance-ids i-1234567890abcdef0

# Запуск инстанса
aws ec2 start-instances --instance-ids i-1234567890abcdef0

# Терминация инстанса
aws ec2 terminate-instances --instance-ids i-1234567890abcdef0

# Получение информации о инстансе
aws ec2 describe-instances --instance-ids i-1234567890abcdef0
```

**Security Groups:**
```bash
# Создание Security Group
aws ec2 create-security-group \
  --group-name my-sg \
  --description "My security group" \
  --vpc-id vpc-1234567890abcdef0

# Добавление правила (SSH)
aws ec2 authorize-security-group-ingress \
  --group-id sg-1234567890abcdef0 \
  --protocol tcp \
  --port 22 \
  --cidr 0.0.0.0/0

# Добавление правила (HTTP)
aws ec2 authorize-security-group-ingress \
  --group-id sg-1234567890abcdef0 \
  --protocol tcp \
  --port 80 \
  --cidr 0.0.0.0/0

# Просмотр правил
aws ec2 describe-security-groups --group-ids sg-1234567890abcdef0

# Удаление правила
aws ec2 revoke-security-group-ingress \
  --group-id sg-1234567890abcdef0 \
  --protocol tcp \
  --port 22 \
  --cidr 0.0.0.0/0
```

**VPC и Подсети:**
```bash
# Создание VPC
aws ec2 create-vpc --cidr-block 10.0.0.0/16

# Создание подсети
aws ec2 create-subnet \
  --vpc-id vpc-1234567890abcdef0 \
  --cidr-block 10.0.1.0/24 \
  --availability-zone us-east-1a

# Создание Internet Gateway
aws ec2 create-internet-gateway

# Присоединение IGW к VPC
aws ec2 attach-internet-gateway \
  --internet-gateway-id igw-1234567890abcdef0 \
  --vpc-id vpc-1234567890abcdef0

# Создание Route Table
aws ec2 create-route-table --vpc-id vpc-1234567890abcdef0

# Добавление маршрута в интернет
aws ec2 create-route \
  --route-table-id rtb-1234567890abcdef0 \
  --destination-cidr-block 0.0.0.0/0 \
  --gateway-id igw-1234567890abcdef0

# Ассоциация подсети с route table
aws ec2 associate-route-table \
  --route-table-id rtb-1234567890abcdef0 \
  --subnet-id subnet-1234567890abcdef0

# Список VPC
aws ec2 describe-vpcs

# Список подсетей
aws ec2 describe-subnets --filters "Name=vpc-id,Values=vpc-1234567890abcdef0"
```

**Elastic IP:**
```bash
# Выделение Elastic IP
aws ec2 allocate-address --domain vpc

# Ассоциация с инстансом
aws ec2 associate-address \
  --instance-id i-1234567890abcdef0 \
  --allocation-id eipalloc-1234567890abcdef0

# Дисассоциация
aws ec2 disassociate-address --association-id eipassoc-1234567890abcdef0

# Освобождение
aws ec2 release-address --allocation-id eipalloc-1234567890abcdef0
```

**Key Pairs:**
```bash
# Создание ключевой пары
aws ec2 create-key-pair \
  --key-name MyKeyPair \
  --query 'KeyMaterial' \
  --output text > MyKeyPair.pem

chmod 400 MyKeyPair.pem

# Список ключей
aws ec2 describe-key-pairs

# Удаление ключа
aws ec2 delete-key-pair --key-name MyKeyPair
```

**User Data (автозапуск при старте):**
```bash
#!/bin/bash
yum update -y
yum install -y httpd
systemctl start httpd
systemctl enable httpd
echo "<h1>Hello from EC2</h1>" > /var/www/html/index.html
```

**Подключение к EC2:**
```bash
# SSH подключение
ssh -i MyKeyPair.pem ec2-user@ec2-54-123-45-67.compute-1.amazonaws.com

# Через Session Manager (без SSH ключей)
aws ssm start-session --target i-1234567890abcdef0
```

**EC2 Instance Types:**
- **General Purpose**: t3, t4g, m5, m6i - сбалансированные
- **Compute Optimized**: c5, c6i - для вычислений
- **Memory Optimized**: r5, r6i, x2 - много RAM
- **Storage Optimized**: i3, d2 - локальное хранилище
- **Accelerated Computing**: p3, g4 - GPU

**Spot Instances:**
```bash
# Запрос Spot Instance
aws ec2 request-spot-instances \
  --spot-price "0.05" \
  --instance-count 1 \
  --type "one-time" \
  --launch-specification file://specification.json

# Отмена Spot Request
aws ec2 cancel-spot-instance-requests \
  --spot-instance-request-ids sir-1234567890abcdef0
```

### 💻 Задание

1. Создай VPC с CIDR 10.0.0.0/16

2. Создай две подсети в разных AZ:
   - Public subnet: 10.0.1.0/24
   - Private subnet: 10.0.2.0/24

3. Создай Internet Gateway и присоедини к VPC

4. Создай route table для public подсети с маршрутом в интернет

5. Создай Security Group разрешающий SSH (22) и HTTP (80)

6. Запусти t2.micro инстанс в public подсети с User Data скриптом, который устанавливает nginx

7. Выдели и присоедини Elastic IP к инстансу

8. Проверь доступность через браузер

9. Посмотри логи инициализации:
   ```bash
   ssh -i key.pem ec2-user@<elastic-ip>
   sudo cat /var/log/cloud-init-output.log
   ```

### 🚀 Бонус (новое)

- Создай NAT Gateway для приватной подсети
- Запусти инстанс в приватной подсети и проверь доступ в интернет
- Настрой VPC Peering между двумя VPC
- Используй EC2 Instance Connect для подключения без ключей
- Создай Launch Template для быстрого запуска инстансов
- Настрой Auto Scaling Group (будет в следующем модуле)

---

## Модуль 3: S3 и Storage (30 минут)

### 🎯 Напоминалка

**S3 Basics:**
```bash
# Создание бакета
aws s3 mb s3://my-unique-bucket-name

# Список бакетов
aws s3 ls

# Загрузка файла
aws s3 cp file.txt s3://my-bucket/
aws s3 cp folder/ s3://my-bucket/folder/ --recursive

# Скачивание файла
aws s3 cp s3://my-bucket/file.txt .
aws s3 cp s3://my-bucket/folder/ ./local-folder/ --recursive

# Синхронизация
aws s3 sync ./local-folder s3://my-bucket/folder/
aws s3 sync s3://my-bucket/folder/ ./local-folder

# Список объектов
aws s3 ls s3://my-bucket/
aws s3 ls s3://my-bucket/folder/ --recursive

# Удаление
aws s3 rm s3://my-bucket/file.txt
aws s3 rm s3://my-bucket/folder/ --recursive

# Удаление бакета
aws s3 rb s3://my-bucket --force
```

**S3 через s3api (больше контроля):**
```bash
# Создание бакета с настройками
aws s3api create-bucket \
  --bucket my-bucket \
  --region us-east-1

# Для регионов кроме us-east-1
aws s3api create-bucket \
  --bucket my-bucket \
  --region eu-west-1 \
  --create-bucket-configuration LocationConstraint=eu-west-1

# Загрузка с метаданными
aws s3api put-object \
  --bucket my-bucket \
  --key file.txt \
  --body file.txt \
  --metadata key1=value1,key2=value2 \
  --content-type text/plain

# Получение метаданных
aws s3api head-object --bucket my-bucket --key file.txt

# Копирование объекта
aws s3api copy-object \
  --copy-source my-bucket/file.txt \
  --bucket my-bucket \
  --key file-copy.txt
```

**Bucket Policies:**
```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "PublicReadGetObject",
      "Effect": "Allow",
      "Principal": "*",
      "Action": "s3:GetObject",
      "Resource": "arn:aws:s3:::my-bucket/*"
    }
  ]
}
```

```bash
# Применение политики
aws s3api put-bucket-policy \
  --bucket my-bucket \
  --policy file://bucket-policy.json

# Просмотр политики
aws s3api get-bucket-policy --bucket my-bucket

# Удаление политики
aws s3api delete-bucket-policy --bucket my-bucket
```

**Версионирование:**
```bash
# Включение версионирования
aws s3api put-bucket-versioning \
  --bucket my-bucket \
  --versioning-configuration Status=Enabled

# Проверка статуса
aws s3api get-bucket-versioning --bucket my-bucket

# Список версий
aws s3api list-object-versions --bucket my-bucket

# Восстановление версии
aws s3api copy-object \
  --copy-source my-bucket/file.txt?versionId=VERSION_ID \
  --bucket my-bucket \
  --key file.txt
```

**Lifecycle Policies:**
```json
{
  "Rules": [
    {
      "Id": "Move to Glacier",
      "Status": "Enabled",
      "Prefix": "logs/",
      "Transitions": [
        {
          "Days": 30,
          "StorageClass": "GLACIER"
        },
        {
          "Days": 90,
          "StorageClass": "DEEP_ARCHIVE"
        }
      ],
      "Expiration": {
        "Days": 365
      }
    }
  ]
}
```

```bash
# Применение lifecycle политики
aws s3api put-bucket-lifecycle-configuration \
  --bucket my-bucket \
  --lifecycle-configuration file://lifecycle.json

# Просмотр политики
aws s3api get-bucket-lifecycle-configuration --bucket my-bucket
```

**S3 Storage Classes:**
- **Standard**: Частый доступ, высокая доступность
- **Intelligent-Tiering**: Автоматическое перемещение между уровнями
- **Standard-IA**: Нечастый доступ
- **One Zone-IA**: Нечастый доступ, одна AZ
- **Glacier Instant Retrieval**: Архив с быстрым доступом
- **Glacier Flexible Retrieval**: Архив (минуты-часы)
- **Glacier Deep Archive**: Долгосрочный архив (12 часов)

**Pre-signed URLs:**
```bash
# Генерация pre-signed URL (доступен 1 час)
aws s3 presign s3://my-bucket/file.txt --expires-in 3600

# Загрузка через pre-signed URL
curl -X PUT -T file.txt "PRE_SIGNED_URL"
```

**S3 Static Website:**
```bash
# Включение static website hosting
aws s3 website s3://my-bucket/ \
  --index-document index.html \
  --error-document error.html

# Или через s3api
aws s3api put-bucket-website \
  --bucket my-bucket \
  --website-configuration file://website.json
```

**website.json:**
```json
{
  "IndexDocument": {
    "Suffix": "index.html"
  },
  "ErrorDocument": {
    "Key": "error.html"
  }
}
```

**CORS Configuration:**
```json
{
  "CORSRules": [
    {
      "AllowedOrigins": ["*"],
      "AllowedMethods": ["GET", "HEAD"],
      "AllowedHeaders": ["*"],
      "MaxAgeSeconds": 3000
    }
  ]
}
```

```bash
aws s3api put-bucket-cors \
  --bucket my-bucket \
  --cors-configuration file://cors.json
```

**Encryption:**
```bash
# Server-side encryption (SSE-S3)
aws s3 cp file.txt s3://my-bucket/ --sse AES256

# SSE-KMS
aws s3 cp file.txt s3://my-bucket/ \
  --sse aws:kms \
  --sse-kms-key-id KEY_ID

# Включение шифрования по умолчанию
aws s3api put-bucket-encryption \
  --bucket my-bucket \
  --server-side-encryption-configuration file://encryption.json
```

**EBS (Elastic Block Store):**
```bash
# Создание EBS volume
aws ec2 create-volume \
  --availability-zone us-east-1a \
  --size 10 \
  --volume-type gp3

# Присоединение к инстансу
aws ec2 attach-volume \
  --volume-id vol-1234567890abcdef0 \
  --instance-id i-1234567890abcdef0 \
  --device /dev/sdf

# Создание снапшота
aws ec2 create-snapshot \
  --volume-id vol-1234567890abcdef0 \
  --description "My snapshot"

# Восстановление из снапшота
aws ec2 create-volume \
  --snapshot-id snap-1234567890abcdef0 \
  --availability-zone us-east-1a

# Список volumes
aws ec2 describe-volumes

# Список снапшотов
aws ec2 describe-snapshots --owner-ids self
```

**EFS (Elastic File System):**
```bash
# Создание EFS
aws efs create-file-system \
  --performance-mode generalPurpose \
  --throughput-mode bursting \
  --tags Key=Name,Value=MyEFS

# Создание mount target
aws efs create-mount-target \
  --file-system-id fs-1234567890abcdef0 \
  --subnet-id subnet-1234567890abcdef0 \
  --security-groups sg-1234567890abcdef0

# Монтирование на EC2
sudo mount -t nfs4 -o nfsvers=4.1 \
  fs-1234567890abcdef0.efs.us-east-1.amazonaws.com:/ /mnt/efs
```

### 💻 Задание

1. Создай S3 бакет с уникальным именем

2. Загрузи несколько файлов (текст, изображение)

3. Включи версионирование на бакете

4. Создай и примени bucket policy для публичного чтения объектов

5. Настрой static website hosting и загрузи простую HTML страницу

6. Создай lifecycle policy для перемещения старых файлов в Glacier через 30 дней

7. Создай pre-signed URL для приватного объекта

8. Настрой CORS для бакета

9. Посчитай размер бакета:
   ```bash
   aws s3 ls s3://my-bucket --recursive --summarize --human-readable
   ```

### 🚀 Бонус (новое)

- Настрой S3 Transfer Acceleration для быстрой загрузки
- Используй S3 Select для запросов к данным без скачивания
- Настрой S3 Event Notifications для Lambda триггеров
- Создай S3 Batch Operations для массовых операций
- Используй S3 Inventory для аудита бакета
- Настрой Cross-Region Replication (CRR)
- Создай EBS snapshot и восстанови volume из него

---

## Модуль 4: Load Balancing и Auto Scaling (35 минут)

### 🎯 Напоминалка

**Application Load Balancer (ALB):**
```bash
# Создание ALB
aws elbv2 create-load-balancer \
  --name my-alb \
  --subnets subnet-12345678 subnet-87654321 \
  --security-groups sg-12345678

# Создание Target Group
aws elbv2 create-target-group \
  --name my-targets \
  --protocol HTTP \
  --port 80 \
  --vpc-id vpc-12345678 \
  --health-check-path /health \
  --health-check-interval-seconds 30 \
  --health-check-timeout-seconds 5 \
  --healthy-threshold-count 2 \
  --unhealthy-threshold-count 2

# Регистрация таргетов
aws elbv2 register-targets \
  --target-group-arn TARGET_GROUP_ARN \
  --targets Id=i-1234567890abcdef0 Id=i-0987654321fedcba0

# Создание Listener
aws elbv2 create-listener \
  --load-balancer-arn LOAD_BALANCER_ARN \
  --protocol HTTP \
  --port 80 \
  --default-actions Type=forward,TargetGroupArn=TARGET_GROUP_ARN

# Добавление правила маршрутизации (path-based)
aws elbv2 create-rule \
  --listener-arn LISTENER_ARN \
  --priority 1 \
  --conditions Field=path-pattern,Values='/api/*' \
  --actions Type=forward,TargetGroupArn=API_TARGET_GROUP_ARN

# Проверка health check
aws elbv2 describe-target-health \
  --target-group-arn TARGET_GROUP_ARN

# Список ALB
aws elbv2 describe-load-balancers

# Удаление ALB
aws elbv2 delete-load-balancer --load-balancer-arn ARN
```

**Network Load Balancer (NLB):**
```bash
# Создание NLB (TCP/UDP)
aws elbv2 create-load-balancer \
  --name my-nlb \
  --type network \
  --subnets subnet-12345678 subnet-87654321

# Остальное аналогично ALB
```

**Classic Load Balancer (устаревший):**
```bash
# Создание ELB
aws elb create-load-balancer \
  --load-balancer-name my-elb \
  --listeners "Protocol=HTTP,LoadBalancerPort=80,InstanceProtocol=HTTP,InstancePort=80" \
  --subnets subnet-12345678 \
  --security-groups sg-12345678

# Регистрация инстансов
aws elb register-instances-with-load-balancer \
  --load-balancer-name my-elb \
  --instances i-1234567890abcdef0
```

**Launch Template:**
```bash
# Создание Launch Template
aws ec2 create-launch-template \
  --launch-template-name my-template \
  --version-description "Version 1" \
  --launch-template-data file://template-data.json
```

**template-data.json:**
```json
{
  "ImageId": "ami-0c55b159cbfafe1f0",
  "InstanceType": "t2.micro",
  "KeyName": "MyKeyPair",
  "SecurityGroupIds": ["sg-12345678"],
  "UserData": "IyEvYmluL2Jhc2gKZWNobyAiSGVsbG8gV29ybGQi",
  "TagSpecifications": [
    {
      "ResourceType": "instance",
      "Tags": [
        {
          "Key": "Name",
          "Value": "AutoScaled-Instance"
        }
      ]
    }
  ]
}
```

**Auto Scaling Group:**
```bash
# Создание ASG
aws autoscaling create-auto-scaling-group \
  --auto-scaling-group-name my-asg \
  --launch-template "LaunchTemplateId=lt-1234567890abcdef0,Version=1" \
  --min-size 1 \
  --max-size 5 \
  --desired-capacity 2 \
  --vpc-zone-identifier "subnet-12345678,subnet-87654321" \
  --target-group-arns TARGET_GROUP_ARN \
  --health-check-type ELB \
  --health-check-grace-period 300

# Обновление ASG
aws autoscaling update-auto-scaling-group \
  --auto-scaling-group-name my-asg \
  --min-size 2 \
  --max-size 10 \
  --desired-capacity 3

# Описание ASG
aws autoscaling describe-auto-scaling-groups \
  --auto-scaling-group-names my-asg

# Список инстансов в ASG
aws autoscaling describe-auto-scaling-instances

# Удаление ASG
aws autoscaling delete-auto-scaling-group \
  --auto-scaling-group-name my-asg \
  --force-delete
```

**Scaling Policies:**
```bash
# Target Tracking Scaling (CPU)
aws autoscaling put-scaling-policy \
  --auto-scaling-group-name my-asg \
  --policy-name cpu-target-tracking \
  --policy-type TargetTrackingScaling \
  --target-tracking-configuration file://cpu-scaling.json
```

**cpu-scaling.json:**
```json
{
  "TargetValue": 50.0,
  "PredefinedMetricSpecification": {
    "PredefinedMetricType": "ASGAverageCPUUtilization"
  }
}
```

```bash
# Step Scaling
aws autoscaling put-scaling-policy \
  --auto-scaling-group-name my-asg \
  --policy-name scale-up \
  --scaling-adjustment 1 \
  --adjustment-type ChangeInCapacity \
  --cooldown 300

# Scheduled Scaling
aws autoscaling put-scheduled-update-group-action \
  --auto-scaling-group-name my-asg \
  --scheduled-action-name scale-up-morning \
  --recurrence "0 8 * * MON-FRI" \
  --min-size 3 \
  --max-size 10 \
  --desired-capacity 5

# Удаление Scaling Policy
aws autoscaling delete-policy \
  --auto-scaling-group-name my-asg \
  --policy-name cpu-target-tracking
```

**CloudWatch Alarms для Auto Scaling:**
```bash
# Создание alarm для scale up
aws cloudwatch put-metric-alarm \
  --alarm-name high-cpu \
  --alarm-description "Scale up when CPU > 70%" \
  --metric-name CPUUtilization \
  --namespace AWS/EC2 \
  --statistic Average \
  --period 300 \
  --threshold 70 \
  --comparison-operator GreaterThanThreshold \
  --evaluation-periods 2 \
  --alarm-actions SCALING_POLICY_ARN

# Создание alarm для scale down
aws cloudwatch put-metric-alarm \
  --alarm-name low-cpu \
  --alarm-description "Scale down when CPU < 30%" \
  --metric-name CPUUtilization \
  --namespace AWS/EC2 \
  --statistic Average \
  --period 300 \
  --threshold 30 \
  --comparison-operator LessThanThreshold \
  --evaluation-periods 2 \
  --alarm-actions SCALING_POLICY_ARN
```

### 💻 Задание

1. Создай Target Group для HTTP на порту 80

2. Создай Application Load Balancer в двух подсетях разных AZ

3. Создай Listener для ALB на порту 80

4. Создай Launch Template с User Data для установки веб-сервера

5. Создай Auto Scaling Group:
   - Min: 2, Max: 6, Desired: 2
   - Используй созданный Launch Template
   - Присоедини к Target Group

6. Настрой Target Tracking Scaling Policy для CPU 50%

7. Проверь работу:
   ```bash
   # Получи DNS ALB
   aws elbv2 describe-load-balancers --names my-alb
   
   # Открой в браузере
   curl http://ALB-DNS-NAME
   ```

8. Симулируй нагрузку и наблюдай за масштабированием:
   ```bash
   # На EC2 инстансе
   sudo yum install -y stress
   stress --cpu 4 --timeout 300s
   ```

9. Проверь количество инстансов:
   ```bash
   aws autoscaling describe-auto-scaling-groups \
     --auto-scaling-group-names my-asg \
     --query 'AutoScalingGroups[0].[MinSize,MaxSize,DesiredCapacity]'
   ```

### 🚀 Бонус (новое)

- Настрой Path-based routing для ALB (разные пути на разные Target Groups)
- Используй Host-based routing (разные домены на разные Target Groups)
- Настрой SSL/TLS сертификат через ACM (AWS Certificate Manager)
- Создай Scheduled Scaling для предсказуемых пиков нагрузки
- Настрой Lifecycle Hooks для кастомных действий при запуске/остановке инстансов
- Используй Mixed Instances Policy (разные типы инстансов + Spot)
- Настрой CloudWatch Dashboard для мониторинга ASG

---

## Модуль 5: RDS и Базы данных (30 минут)

### 🎯 Напоминалка

**RDS (Relational Database Service):**
```bash
# Создание RDS инстанса (MySQL)
aws rds create-db-instance \
  --db-instance-identifier mydb \
  --db-instance-class db.t3.micro \
  --engine mysql \
  --engine-version 8.0.35 \
  --master-username admin \
  --master-user-password MyPassword123 \
  --allocated-storage 20 \
  --storage-type gp3 \
  --vpc-security-group-ids sg-12345678 \
  --db-subnet-group-name my-db-subnet-group \
  --backup-retention-period 7 \
  --preferred-backup-window "03:00-04:00" \
  --preferred-maintenance-window "Mon:04:00-Mon:05:00" \
  --publicly-accessible \
  --tags Key=Name,Value=MyDatabase

# Создание DB Subnet Group
aws rds create-db-subnet-group \
  --db-subnet-group-name my-db-subnet-group \
  --db-subnet-group-description "My DB Subnet Group" \
  --subnet-ids subnet-12345678 subnet-87654321

# Список RDS инстансов
aws rds describe-db-instances

# Информация о конкретном инстансе
aws rds describe-db-instances \
  --db-instance-identifier mydb

# Модификация инстанса
aws rds modify-db-instance \
  --db-instance-identifier mydb \
  --db-instance-class db.t3.small \
  --apply-immediately

# Создание read replica
aws rds create-db-instance-read-replica \
  --db-instance-identifier mydb-replica \
  --source-db-instance-identifier mydb

# Остановка инстанса
aws rds stop-db-instance \
  --db-instance-identifier mydb

# Запуск инстанса
aws rds start-db-instance \
  --db-instance-identifier mydb

# Удаление инстанса
aws rds delete-db-instance \
  --db-instance-identifier mydb \
  --skip-final-snapshot
  # или с финальным снапшотом
  --final-db-snapshot-identifier mydb-final-snapshot
```

**RDS Snapshots:**
```bash
# Создание снапшота
aws rds create-db-snapshot \
  --db-instance-identifier mydb \
  --db-snapshot-identifier mydb-snapshot-2024-01

# Список снапшотов
aws rds describe-db-snapshots \
  --db-instance-identifier mydb

# Восстановление из снапшота
aws rds restore-db-instance-from-db-snapshot \
  --db-instance-identifier mydb-restored \
  --db-snapshot-identifier mydb-snapshot-2024-01

# Копирование снапшота в другой регион
aws rds copy-db-snapshot \
  --source-db-snapshot-identifier arn:aws:rds:us-east-1:123456789012:snapshot:mydb-snapshot \
  --target-db-snapshot-identifier mydb-snapshot-copy \
  --source-region us-east-1 \
  --region us-west-2

# Удаление снапшота
aws rds delete-db-snapshot \
  --db-snapshot-identifier mydb-snapshot-2024-01
```

**RDS Multi-AZ и Failover:**
```bash
# Включение Multi-AZ
aws rds modify-db-instance \
  --db-instance-identifier mydb \
  --multi-az \
  --apply-immediately

# Принудительный Failover
aws rds reboot-db-instance \
  --db-instance-identifier mydb \
  --force-failover
```

**Aurora:**
```bash
# Создание Aurora Cluster
aws rds create-db-cluster \
  --db-cluster-identifier myaurora-cluster \
  --engine aurora-mysql \
  --engine-version 8.0.mysql_aurora.3.04.0 \
  --master-username admin \
  --master-user-password MyPassword123 \
  --vpc-security-group-ids sg-12345678 \
  --db-subnet-group-name my-db-subnet-group

# Создание инстанса в кластере
aws rds create-db-instance \
  --db-instance-identifier myaurora-instance-1 \
  --db-instance-class db.t3.medium \
  --engine aurora-mysql \
  --db-cluster-identifier myaurora-cluster

# Aurora Serverless (v2)
aws rds create-db-cluster \
  --db-cluster-identifier myaurora-serverless \
  --engine aurora-mysql \
  --engine-mode serverless \
  --scaling-configuration MinCapacity=1,MaxCapacity=4,AutoPause=true
```

**DynamoDB:**
```bash
# Создание таблицы
aws dynamodb create-table \
  --table-name Users \
  --attribute-definitions \
    AttributeName=UserId,AttributeType=S \
    AttributeName=Email,AttributeType=S \
  --key-schema \
    AttributeName=UserId,KeyType=HASH \
  --provisioned-throughput \
    ReadCapacityUnits=5,WriteCapacityUnits=5 \
  --global-secondary-indexes \
    "[
      {
        \"IndexName\": \"EmailIndex\",
        \"KeySchema\": [{\"AttributeName\":\"Email\",\"KeyType\":\"HASH\"}],
        \"Projection\": {\"ProjectionType\":\"ALL\"},
        \"ProvisionedThroughput\": {\"ReadCapacityUnits\":5,\"WriteCapacityUnits\":5}
      }
    ]"

# Список таблиц
aws dynamodb list-tables

# Описание таблицы
aws dynamodb describe-table --table-name Users

# Добавление элемента
aws dynamodb put-item \
  --table-name Users \
  --item '{"UserId": {"S": "user123"}, "Email": {"S": "user@example.com"}, "Name": {"S": "John Doe"}}'

# Получение элемента
aws dynamodb get-item \
  --table-name Users \
  --key '{"UserId": {"S": "user123"}}'

# Запрос (Query)
aws dynamodb query \
  --table-name Users \
  --key-condition-expression "UserId = :userId" \
  --expression-attribute-values '{":userId": {"S": "user123"}}'

# Сканирование (Scan)
aws dynamodb scan --table-name Users

# Обновление элемента
aws dynamodb update-item \
  --table-name Users \
  --key '{"UserId": {"S": "user123"}}' \
  --update-expression "SET #name = :name" \
  --expression-attribute-names '{"#name": "Name"}' \
  --expression-attribute-values '{":name": {"S": "Jane Doe"}}'

# Удаление элемента
aws dynamodb delete-item \
  --table-name Users \
  --key '{"UserId": {"S": "user123"}}'

# Переключение на On-Demand режим
aws dynamodb update-table \
  --table-name Users \
  --billing-mode PAY_PER_REQUEST

# Backup
aws dynamodb create-backup \
  --table-name Users \
  --backup-name Users-Backup-2024-01

# Восстановление
aws dynamodb restore-table-from-backup \
  --target-table-name Users-Restored \
  --backup-arn arn:aws:dynamodb:us-east-1:123456789012:table/Users/backup/01234567890123-12345678

# Удаление таблицы
aws dynamodb delete-table --table-name Users
```

**ElastiCache (Redis):**
```bash
# Создание Redis cluster
aws elasticache create-cache-cluster \
  --cache-cluster-id my-redis \
  --cache-node-type cache.t3.micro \
  --engine redis \
  --num-cache-nodes 1 \
  --cache-subnet-group-name my-cache-subnet-group \
  --security-group-ids sg-12345678

# Создание Redis Replication Group (Multi-AZ)
aws elasticache create-replication-group \
  --replication-group-id my-redis-cluster \
  --replication-group-description "My Redis Cluster" \
  --cache-node-type cache.t3.micro \
  --engine redis \
  --automatic-failover-enabled \
  --num-cache-clusters 2 \
  --cache-subnet-group-name my-cache-subnet-group \
  --security-group-ids sg-12345678

# Получение endpoint
aws elasticache describe-cache-clusters \
  --cache-cluster-id my-redis \
  --show-cache-node-info
```

**Подключение к RDS:**
```bash
# MySQL
mysql -h mydb.c1234567890.us-east-1.rds.amazonaws.com -u admin -p

# PostgreSQL
psql -h mydb.c1234567890.us-east-1.rds.amazonaws.com -U admin -d postgres

# Через SSH tunnel (для приватных RDS)
ssh -i key.pem -L 3306:mydb.rds.amazonaws.com:3306 ec2-user@bastion-host
mysql -h 127.0.0.1 -u admin -p
```

### 💻 Задание

1. Создай DB Subnet Group в двух приватных подсетях

2. Создай Security Group для RDS (порт 3306 для MySQL)

3. Создай RDS MySQL инстанс:
   - db.t3.micro
   - 20 GB storage
   - Backup retention 7 дней
   - В приватной подсети

4. Подключись к RDS из EC2 инстанса в той же VPC

5. Создай базу данных и таблицу:
   ```sql
   CREATE DATABASE testdb;
   USE testdb;
   CREATE TABLE users (id INT PRIMARY KEY, name VARCHAR(100));
   INSERT INTO users VALUES (1, 'John Doe');
   SELECT * FROM users;
   ```

6. Создай manual snapshot

7. Создай DynamoDB таблицу:
   ```bash
   aws dynamodb create-table \
     --table-name Products \
     --attribute-definitions AttributeName=ProductId,AttributeType=S \
     --key-schema AttributeName=ProductId,KeyType=HASH \
     --billing-mode PAY_PER_REQUEST
   ```

8. Добавь несколько элементов в DynamoDB

9. Выполни Query и Scan операции

### 🚀 Бонус (новое)

- Включи Multi-AZ для RDS и протестируй failover
- Создай read replica и проверь репликацию
- Настрой автоматический backup и Point-in-Time Recovery
- Создай Aurora Serverless кластер
- Настрой DynamoDB Global Tables для multi-region репликации
- Используй DynamoDB Streams для триггеров
- Настрой ElastiCache Redis cluster
- Используй AWS Database Migration Service (DMS) для миграции данных

---

## Модуль 6: Lambda и Serverless (30 минут)

### 🎯 Напоминалка

**Lambda Functions:**
```bash
# Создание Lambda функции (Python)
aws lambda create-function \
  --function-name my-function \
  --runtime python3.11 \
  --role arn:aws:iam::123456789012:role/lambda-execution-role \
  --handler lambda_function.lambda_handler \
  --zip-file fileb://function.zip \
  --timeout 30 \
  --memory-size 256 \
  --environment Variables={KEY1=value1,KEY2=value2}

# Обновление кода функции
aws lambda update-function-code \
  --function-name my-function \
  --zip-file fileb://function.zip

# Обновление конфигурации
aws lambda update-function-configuration \
  --function-name my-function \
  --timeout 60 \
  --memory-size 512

# Вызов функции
aws lambda invoke \
  --function-name my-function \
  --payload '{"key1": "value1", "key2": "value2"}' \
  response.json

# Список функций
aws lambda list-functions

# Получение информации
aws lambda get-function --function-name my-function

# Удаление функции
aws lambda delete-function --function-name my-function
```

**Пример Lambda функции (Python):**
```python
# lambda_function.py
import json

def lambda_handler(event, context):
    print(f"Event: {json.dumps(event)}")
    
    # Обработка
    name = event.get('name', 'World')
    
    return {
        'statusCode': 200,
        'body': json.dumps({
            'message': f'Hello {name}!'
        })
    }
```

**Создание ZIP архива:**
```bash
zip function.zip lambda_function.py

# С зависимостями
pip install requests -t .
zip -r function.zip .
```

**Lambda Layers:**
```bash
# Создание Layer (для общих зависимостей)
zip -r layer.zip python/

aws lambda publish-layer-version \
  --layer-name my-layer \
  --description "My Lambda Layer" \
  --zip-file fileb://layer.zip \
  --compatible-runtimes python3.11

# Присоединение Layer к функции
aws lambda update-function-configuration \
  --function-name my-function \
  --layers arn:aws:lambda:us-east-1:123456789012:layer:my-layer:1
```

**Lambda Triggers:**
```bash
# API Gateway Trigger (будет в следующей секции)

# S3 Trigger
aws lambda add-permission \
  --function-name my-function \
  --statement-id s3-trigger \
  --action lambda:InvokeFunction \
  --principal s3.amazonaws.com \
  --source-arn arn:aws:s3:::my-bucket

aws s3api put-bucket-notification-configuration \
  --bucket my-bucket \
  --notification-configuration file://s3-notification.json
```

**s3-notification.json:**
```json
{
  "LambdaFunctionConfigurations": [
    {
      "LambdaFunctionArn": "arn:aws:lambda:us-east-1:123456789012:function:my-function",
      "Events": ["s3:ObjectCreated:*"],
      "Filter": {
        "Key": {
          "FilterRules": [
            {
              "Name": "prefix",
              "Value": "uploads/"
            }
          ]
        }
      }
    }
  ]
}
```

```bash
# CloudWatch Events/EventBridge Trigger
aws events put-rule \
  --name scheduled-lambda \
  --schedule-expression "rate(5 minutes)"

aws events put-targets \
  --rule scheduled-lambda \
  --targets "Id"="1","Arn"="arn:aws:lambda:us-east-1:123456789012:function:my-function"

aws lambda add-permission \
  --function-name my-function \
  --statement-id eventbridge-invoke \
  --action lambda:InvokeFunction \
  --principal events.amazonaws.com \
  --source-arn arn:aws:events:us-east-1:123456789012:rule/scheduled-lambda

# DynamoDB Stream Trigger
aws lambda create-event-source-mapping \
  --function-name my-function \
  --event-source-arn arn:aws:dynamodb:us-east-1:123456789012:table/Users/stream/2024-01-01T00:00:00.000 \
  --starting-position LATEST
```

**API Gateway + Lambda:**
```bash
# Создание REST API
aws apigateway create-rest-api \
  --name my-api \
  --description "My API"

# Получение root resource id
aws apigateway get-resources --rest-api-id API_ID

# Создание resource
aws apigateway create-resource \
  --rest-api-id API_ID \
  --parent-id ROOT_RESOURCE_ID \
  --path-part users

# Создание метода GET
aws apigateway put-method \
  --rest-api-id API_ID \
  --resource-id RESOURCE_ID \
  --http-method GET \
  --authorization-type NONE

# Интеграция с Lambda
aws apigateway put-integration \
  --rest-api-id API_ID \
  --resource-id RESOURCE_ID \
  --http-method GET \
  --type AWS_PROXY \
  --integration-http-method POST \
  --uri arn:aws:apigateway:us-east-1:lambda:path/2015-03-31/functions/arn:aws:lambda:us-east-1:123456789012:function:my-function/invocations

# Разрешение API Gateway вызывать Lambda
aws lambda add-permission \
  --function-name my-function \
  --statement-id apigateway-invoke \
  --action lambda:InvokeFunction \
  --principal apigateway.amazonaws.com \
  --source-arn "arn:aws:execute-api:us-east-1:123456789012:API_ID/*"

# Деплой API
aws apigateway create-deployment \
  --rest-api-id API_ID \
  --stage-name prod

# URL API: https://API_ID.execute-api.us-east-1.amazonaws.com/prod/users
```

**Step Functions:**
```bash
# Создание State Machine
aws stepfunctions create-state-machine \
  --name my-workflow \
  --definition file://state-machine.json \
  --role-arn arn:aws:iam::123456789012:role/step-functions-role

# Запуск выполнения
aws stepfunctions start-execution \
  --state-machine-arn arn:aws:states:us-east-1:123456789012:stateMachine:my-workflow \
  --input '{"key": "value"}'

# Проверка статуса
aws stepfunctions describe-execution \
  --execution-arn EXECUTION_ARN
```

**EventBridge (CloudWatch Events):**
```bash
# Создание правила для событий
aws events put-rule \
  --name ec2-state-change \
  --event-pattern file://event-pattern.json

# event-pattern.json
{
  "source": ["aws.ec2"],
  "detail-type": ["EC2 Instance State-change Notification"],
  "detail": {
    "state": ["stopped"]
  }
}

# Добавление таргета (Lambda)
aws events put-targets \
  --rule ec2-state-change \
  --targets "Id"="1","Arn"="arn:aws:lambda:us-east-1:123456789012:function:my-function"
```

**SAM (Serverless Application Model):**
```yaml
# template.yaml
AWSTemplateFormatVersion: '2010-09-09'
Transform: AWS::Serverless-2016-10-31

Resources:
  MyFunction:
    Type: AWS::Serverless::Function
    Properties:
      Handler: app.lambda_handler
      Runtime: python3.11
      CodeUri: ./src
      Events:
        HelloWorld:
          Type: Api
          Properties:
            Path: /hello
            Method: get
      Environment:
        Variables:
          TABLE_NAME: !Ref MyTable
  
  MyTable:
    Type: AWS::Serverless::SimpleTable
```

```bash
# Деплой SAM приложения
sam build
sam deploy --guided
```

### 💻 Задание

1. Создай простую Lambda функцию на Python:
   ```python
   def lambda_handler(event, context):
       return {
           'statusCode': 200,
           'body': 'Hello from Lambda!'
       }
   ```

2. Загрузи функцию в AWS Lambda через CLI

3. Вызови функцию и проверь результат

4. Создай S3 бакет и настрой Lambda триггер на создание объектов

5. Загрузи файл в S3 и проверь, что Lambda была вызвана (посмотри CloudWatch Logs)

6. Создай REST API в API Gateway с одним endpoint

7. Интегрируй API Gateway с Lambda функцией

8. Протестируй API:
   ```bash
   curl https://API_ID.execute-api.us-east-1.amazonaws.com/prod/hello
   ```

9. Создай EventBridge правило для запуска Lambda каждые 5 минут

### 🚀 Бонус (новое)

- Создай Lambda Layer с общими зависимостями
- Настрой Lambda с VPC для доступа к RDS
- Используй Lambda Environment Variables и Secrets Manager
- Создай Step Functions workflow с несколькими Lambda функциями
- Настрой Lambda Destinations для асинхронных вызовов
- Используй SAM CLI для разработки и деплоя serverless приложений
- Создай Lambda@Edge для CloudFront
- Настрой Lambda Provisioned Concurrency для уменьшения cold start

---

## Модуль 7: CloudFormation и Infrastructure as Code (30 минут)

### 🎯 Напоминалка

**CloudFormation Basics:**
```bash
# Создание стека
aws cloudformation create-stack \
  --stack-name my-stack \
  --template-body file://template.yaml \
  --parameters ParameterKey=KeyName,ParameterValue=MyKeyPair

# Обновление стека
aws cloudformation update-stack \
  --stack-name my-stack \
  --template-body file://template.yaml

# Удаление стека
aws cloudformation delete-stack --stack-name my-stack

# Список стеков
aws cloudformation list-stacks

# Описание стека
aws cloudformation describe-stacks --stack-name my-stack

# События стека
aws cloudformation describe-stack-events --stack-name my-stack

# Ресурсы стека
aws cloudformation describe-stack-resources --stack-name my-stack

# Валидация template
aws cloudformation validate-template --template-body file://template.yaml
```

**Простой CloudFormation template:**
```yaml
AWSTemplateFormatVersion: '2010-09-09'
Description: Simple EC2 instance

Parameters:
  KeyName:
    Description: EC2 Key Pair
    Type: AWS::EC2::KeyPair::KeyName
  
  InstanceType:
    Description: EC2 instance type
    Type: String
    Default: t2.micro
    AllowedValues:
      - t2.micro
      - t2.small
      - t2.medium

Resources:
  MySecurityGroup:
    Type: AWS::EC2::SecurityGroup
    Properties:
      GroupDescription: Allow SSH and HTTP
      SecurityGroupIngress:
        - IpProtocol: tcp
          FromPort: 22
          ToPort: 22
          CidrIp: 0.0.0.0/0
        - IpProtocol: tcp
          FromPort: 80
          ToPort: 80
          CidrIp: 0.0.0.0/0
  
  MyInstance:
    Type: AWS::EC2::Instance
    Properties:
      ImageId: !Ref LatestAmiId
      InstanceType: !Ref InstanceType
      KeyName: !Ref KeyName
      SecurityGroups:
        - !Ref MySecurityGroup
      UserData:
        Fn::Base64: !Sub |
          #!/bin/bash
          yum update -y
          yum install -y httpd
          systemctl start httpd
          systemctl enable httpd
          echo "<h1>Hello from CloudFormation!</h1>" > /var/www/html/index.html
      Tags:
        - Key: Name
          Value: !Sub ${AWS::StackName}-instance

Parameters:
  LatestAmiId:
    Type: AWS::SSM::Parameter::Value<AWS::EC2::Image::Id>
    Default: /aws/service/ami-amazon-linux-latest/amzn2-ami-hvm-x86_64-gp2

Outputs:
  InstanceId:
    Description: Instance ID
    Value: !Ref MyInstance
  
  PublicIP:
    Description: Public IP address
    Value: !GetAtt MyInstance.PublicIp
  
  Website:
    Description: Website URL
    Value: !Sub 'http://${MyInstance.PublicDnsName}'
```

**Более сложный template (VPC + EC2 + RDS):**
```yaml
AWSTemplateFormatVersion: '2010-09-09'
Description: VPC with EC2 and RDS

Parameters:
  DBPassword:
    Description: Database password
    Type: String
    NoEcho: true
    MinLength: 8

Resources:
  VPC:
    Type: AWS::EC2::VPC
    Properties:
      CidrBlock: 10.0.0.0/16
      EnableDnsHostnames: true
      EnableDnsSupport: true
      Tags:
        - Key: Name
          Value: !Sub ${AWS::StackName}-VPC
  
  PublicSubnet1:
    Type: AWS::EC2::Subnet
    Properties:
      VpcId: !Ref VPC
      CidrBlock: 10.0.1.0/24
      AvailabilityZone: !Select [0, !GetAZs '']
      MapPublicIpOnLaunch: true
  
  PublicSubnet2:
    Type: AWS::EC2::Subnet
    Properties:
      VpcId: !Ref VPC
      CidrBlock: 10.0.2.0/24
      AvailabilityZone: !Select [1, !GetAZs '']
      MapPublicIpOnLaunch: true
  
  PrivateSubnet1:
    Type: AWS::EC2::Subnet
    Properties:
      VpcId: !Ref VPC
      CidrBlock: 10.0.10.0/24
      AvailabilityZone: !Select [0, !GetAZs '']
  
  PrivateSubnet2:
    Type: AWS::EC2::Subnet
    Properties:
      VpcId: !Ref VPC
      CidrBlock: 10.0.11.0/24
      AvailabilityZone: !Select [1, !GetAZs '']
  
  InternetGateway:
    Type: AWS::EC2::InternetGateway
  
  AttachGateway:
    Type: AWS::EC2::VPCGatewayAttachment
    Properties:
      VpcId: !Ref VPC
      InternetGatewayId: !Ref InternetGateway
  
  PublicRouteTable:
    Type: AWS::EC2::RouteTable
    Properties:
      VpcId: !Ref VPC
  
  PublicRoute:
    Type: AWS::EC2::Route
    DependsOn: AttachGateway
    Properties:
      RouteTableId: !Ref PublicRouteTable
      DestinationCidrBlock: 0.0.0.0/0
      GatewayId: !Ref InternetGateway
  
  SubnetRouteTableAssociation1:
    Type: AWS::EC2::SubnetRouteTableAssociation
    Properties:
      SubnetId: !Ref PublicSubnet1
      RouteTableId: !Ref PublicRouteTable
  
  SubnetRouteTableAssociation2:
    Type: AWS::EC2::SubnetRouteTableAssociation
    Properties:
      SubnetId: !Ref PublicSubnet2
      RouteTableId: !Ref PublicRouteTable
  
  WebServerSecurityGroup:
    Type: AWS::EC2::SecurityGroup
    Properties:
      GroupDescription: Security group for web servers
      VpcId: !Ref VPC
      SecurityGroupIngress:
        - IpProtocol: tcp
          FromPort: 80
          ToPort: 80
          CidrIp: 0.0.0.0/0
        - IpProtocol: tcp
          FromPort: 22
          ToPort: 22
          CidrIp: 0.0.0.0/0
  
  DBSecurityGroup:
    Type: AWS::EC2::SecurityGroup
    Properties:
      GroupDescription: Security group for RDS
      VpcId: !Ref VPC
      SecurityGroupIngress:
        - IpProtocol: tcp
          FromPort: 3306
          ToPort: 3306
          SourceSecurityGroupId: !Ref WebServerSecurityGroup
  
  DBSubnetGroup:
    Type: AWS::RDS::DBSubnetGroup
    Properties:
      DBSubnetGroupDescription: Subnet group for RDS
      SubnetIds:
        - !Ref PrivateSubnet1
        - !Ref PrivateSubnet2
  
  Database:
    Type: AWS::RDS::DBInstance
    Properties:
      DBInstanceIdentifier: !Sub ${AWS::StackName}-db
      DBInstanceClass: db.t3.micro
      Engine: mysql
      MasterUsername: admin
      MasterUserPassword: !Ref DBPassword
      AllocatedStorage: 20
      VPCSecurityGroups:
        - !Ref DBSecurityGroup
      DBSubnetGroupName: !Ref DBSubnetGroup

Outputs:
  VPCId:
    Description: VPC ID
    Value: !Ref VPC
    Export:
      Name: !Sub ${AWS::StackName}-VPCID
  
  DBEndpoint:
    Description: Database endpoint
    Value: !GetAtt Database.Endpoint.Address
```

**CloudFormation Intrinsic Functions:**
```yaml
# !Ref - ссылка на ресурс или параметр
SecurityGroups:
  - !Ref MySecurityGroup

# !GetAtt - получение атрибута ресурса
PublicIP: !GetAtt MyInstance.PublicIp

# !Sub - подстановка переменных
Value: !Sub 'http://${MyInstance.PublicDnsName}'

# !Join - объединение строк
!Join [':', [!Ref 'AWS::StackName', !Ref 'AWS::Region']]

# !Select - выбор элемента из списка
AvailabilityZone: !Select [0, !GetAZs '']

# !GetAZs - получение AZ
!GetAZs 'us-east-1'

# !If - условие
!If [CreateProdResources, m5.large, t3.micro]

# !Equals - сравнение
Conditions:
  IsProduction: !Equals [!Ref Environment, 'prod']
```

**Change Sets (для безопасного обновления):**
```bash
# Создание change set
aws cloudformation create-change-set \
  --stack-name my-stack \
  --change-set-name my-changes \
  --template-body file://template.yaml

# Просмотр изменений
aws cloudformation describe-change-set \
  --stack-name my-stack \
  --change-set-name my-changes

# Применение изменений
aws cloudformation execute-change-set \
  --stack-name my-stack \
  --change-set-name my-changes

# Удаление change set
aws cloudformation delete-change-set \
  --stack-name my-stack \
  --change-set-name my-changes
```

### 💻 Задание

1. Создай CloudFormation template для VPC с двумя публичными подсетями

2. Добавь в template Security Group и EC2 инстанс

3. Используй Parameters для KeyName и InstanceType

4. Добавь Outputs для VPC ID и Instance Public IP

5. Создай стек через CLI:
   ```bash
   aws cloudformation create-stack \
     --stack-name my-infrastructure \
     --template-body file://template.yaml \
     --parameters ParameterKey=KeyName,ParameterValue=MyKey
   ```

6. Проверь статус создания:
   ```bash
   aws cloudformation describe-stacks --stack-name my-infrastructure
   aws cloudformation describe-stack-events --stack-name my-infrastructure
   ```

7. Измени template (например, добавь тег) и создай change set

8. Просмотри изменения и примени их

9. Удали стек:
   ```bash
   aws cloudformation delete-stack --stack-name my-infrastructure
   ```

### 🚀 Бонус (новое)

- Создай nested stacks (вложенные стеки)
- Используй CloudFormation Macros для трансформации template
- Настрой Stack Sets для multi-account/multi-region деплоя
- Используй CloudFormation Drift Detection для обнаружения изменений
- Создай Custom Resources с Lambda для расширенной функциональности
- Используй CloudFormation Registry для custom resource types
- Интегрируй с Git для CI/CD пайплайна

---

## Модуль 8: Мониторинг и Логирование (25 минут)

### 🎯 Напоминалка

**CloudWatch Metrics:**
```bash
# Просмотр метрик
aws cloudwatch list-metrics --namespace AWS/EC2

# Получение статистики метрики
aws cloudwatch get-metric-statistics \
  --namespace AWS/EC2 \
  --metric-name CPUUtilization \
  --dimensions Name=InstanceId,Value=i-1234567890abcdef0 \
  --start-time 2024-01-01T00:00:00Z \
  --end-time 2024-01-01T23:59:59Z \
  --period 3600 \
  --statistics Average,Maximum

# Публикация custom метрики
aws cloudwatch put-metric-data \
  --namespace MyApp \
  --metric-name PageViews \
  --value 100 \
  --timestamp 2024-01-01T12:00:00Z \
  --dimensions Site=example.com
```

**CloudWatch Alarms:**
```bash
# Создание alarm (CPU > 80%)
aws cloudwatch put-metric-alarm \
  --alarm-name high-cpu-alarm \
  --alarm-description "Alert when CPU exceeds 80%" \
  --metric-name CPUUtilization \
  --namespace AWS/EC2 \
  --statistic Average \
  --period 300 \
  --evaluation-periods 2 \
  --threshold 80 \
  --comparison-operator GreaterThanThreshold \
  --dimensions Name=InstanceId,Value=i-1234567890abcdef0 \
  --alarm-actions arn:aws:sns:us-east-1:123456789012:my-topic

# Список alarms
aws cloudwatch describe-alarms

# Удаление alarm
aws cloudwatch delete-alarms --alarm-names high-cpu-alarm

# Проверка истории alarm
aws cloudwatch describe-alarm-history --alarm-name high-cpu-alarm
```

**CloudWatch Logs:**
```bash
# Список log groups
aws logs describe-log-groups

# Создание log group
aws logs create-log-group --log-group-name /aws/my-app

# Создание log stream
aws logs create-log-stream \
  --log-group-name /aws/my-app \
  --log-stream-name my-stream

# Запись логов
aws logs put-log-events \
  --log-group-name /aws/my-app \
  --log-stream-name my-stream \
  --log-events timestamp=1234567890000,message="Log message"

# Чтение логов
aws logs tail /aws/my-app --follow

# Фильтрация логов
aws logs filter-log-events \
  --log-group-name /aws/lambda/my-function \
  --filter-pattern "ERROR"

# Экспорт логов в S3
aws logs create-export-task \
  --log-group-name /aws/my-app \
  --from 1234567890000 \
  --to 1234567900000 \
  --destination my-log-bucket \
  --destination-prefix logs/

# Retention policy
aws logs put-retention-policy \
  --log-group-name /aws/my-app \
  --retention-in-days 7
```

**CloudWatch Insights:**
```bash
# Запуск запроса
aws logs start-query \
  --log-group-name /aws/lambda/my-function \
  --start-time 1234567890 \
  --end-time 1234567900 \
  --query-string 'fields @timestamp, @message | filter @message like /ERROR/ | sort @timestamp desc | limit 20'

# Проверка результатов
aws logs get-query-results --query-id QUERY_ID
```

**CloudWatch Dashboards:**
```bash
# Создание dashboard
aws cloudwatch put-dashboard \
  --dashboard-name my-dashboard \
  --dashboard-body file://dashboard.json
```

**dashboard.json:**
```json
{
  "widgets": [
    {
      "type": "metric",
      "properties": {
        "metrics": [
          ["AWS/EC2", "CPUUtilization", {"stat": "Average"}]
        ],
        "period": 300,
        "stat": "Average",
        "region": "us-east-1",
        "title": "EC2 CPU Utilization"
      }
    }
  ]
}
```

**SNS для уведомлений:**
```bash
# Создание SNS topic
aws sns create-topic --name my-alerts

# Подписка email
aws sns subscribe \
  --topic-arn arn:aws:sns:us-east-1:123456789012:my-alerts \
  --protocol email \
  --notification-endpoint admin@example.com

# Подтверждение подписки (через email)

# Публикация сообщения
aws sns publish \
  --topic-arn arn:aws:sns:us-east-1:123456789012:my-alerts \
  --message "Alert: High CPU usage detected"

# SMS подписка
aws sns subscribe \
  --topic-arn arn:aws:sns:us-east-1:123456789012:my-alerts \
  --protocol sms \
  --notification-endpoint +1234567890
```

**CloudTrail (аудит API calls):**
```bash
# Создание trail
aws cloudtrail create-trail \
  --name my-trail \
  --s3-bucket-name my-cloudtrail-bucket

# Начать логирование
aws cloudtrail start-logging --name my-trail

# Просмотр событий
aws cloudtrail lookup-events \
  --lookup-attributes AttributeKey=EventName,AttributeValue=RunInstances

# Список trails
aws cloudtrail describe-trails
```

**X-Ray (distributed tracing):**
```bash
# Установка X-Ray daemon на EC2
curl https://s3.us-east-2.amazonaws.com/aws-xray-assets.us-east-2/xray-daemon/aws-xray-daemon-3.x.rpm -o xray.rpm
sudo yum install -y xray.rpm
sudo systemctl start xray

# В коде Lambda (Python)
from aws_xray_sdk.core import xray_recorder
from aws_xray_sdk.core import patch_all

patch_all()

@xray_recorder.capture('my_function')
def lambda_handler(event, context):
    # Ваш код
    pass
```

**AWS Config (соответствие правилам):**
```bash
# Включение AWS Config
aws configservice put-configuration-recorder \
  --configuration-recorder name=default,roleARN=arn:aws:iam::123456789012:role/config-role

aws configservice put-delivery-channel \
  --delivery-channel name=default,s3BucketName=my-config-bucket

# Запуск recorder
aws configservice start-configuration-recorder \
  --configuration-recorder-name default

# Создание Config Rule (проверка шифрования S3)
aws configservice put-config-rule \
  --config-rule file://s3-encryption-rule.json
```

### 💻 Задание

1. Создай CloudWatch Alarm для мониторинга CPU EC2 инстанса:
   ```bash
   aws cloudwatch put-metric-alarm \
     --alarm-name my-cpu-alarm \
     --metric-name CPUUtilization \
     --namespace AWS/EC2 \
     --statistic Average \
     --period 300 \
     --evaluation-periods 1 \
     --threshold 70 \
     --comparison-operator GreaterThanThreshold \
     --dimensions Name=InstanceId,Value=YOUR_INSTANCE_ID
   ```

2. Создай SNS topic и подпишись на email уведомления

3. Обнови alarm для отправки уведомлений в SNS

4. Создай Log Group для своего приложения

5. Настрой retention policy на 7 дней

6. Напиши скрипт для публикации custom метрики:
   ```bash
   aws cloudwatch put-metric-data \
     --namespace MyApp \
     --metric-name RequestCount \
     --value 42 \
     --unit Count
   ```

7. Создай простой CloudWatch Dashboard с метриками EC2

8. Включи CloudTrail для логирования всех API вызовов

9. Посмотри последние события в CloudTrail:
   ```bash
   aws cloudtrail lookup-events --max-results 10
   ```

### 🚀 Бонус (новое)

- Настрой CloudWatch Log Insights запросы для анализа логов
- Создай composite alarm (комбинация нескольких алармов)
- Используй CloudWatch Contributor Insights
- Настрой CloudWatch Anomaly Detection
- Интегрируй X-Ray для трассировки распределенных приложений
- Используй AWS Config Rules для проверки соответствия
- Настрой CloudWatch Events для автоматизации реакции на события
- Создай Custom CloudWatch Dashboard с несколькими виджетами

---

## Финальный проект (60 минут)

### Задача: Развернуть полноценное веб-приложение

Создай комплексную инфраструктуру используя все изученные сервисы:

### Требования:

**1. Сетевая инфраструктура:**
- VPC с публичными и приватными подсетями в 2 AZ
- Internet Gateway и NAT Gateway
- Правильные route tables

**2. Compute:**
- Application Load Balancer в публичных подсетях
- Auto Scaling Group с Launch Template
- Min: 2, Max: 6, Target CPU: 50%
- EC2 инстансы с веб-сервером (nginx/apache)

**3. База данных:**
- RDS MySQL в приватных подсетях
- Multi-AZ enabled
- Автоматические бэкапы

**4. Storage:**
- S3 бакет для статических файлов
- CloudFront для CDN (опционально)
- Lifecycle policy для архивации старых файлов

**5. Serverless:**
- Lambda функция для обработки S3 событий
- API Gateway endpoint
- EventBridge правило для scheduled tasks

**6. Мониторинг:**
- CloudWatch Alarms для CPU, Memory, Disk
- SNS уведомления на email
- CloudWatch Dashboard
- CloudTrail для аудита

**7. Infrastructure as Code:**
- Весь проект описан в CloudFormation template
- Использование параметров и outputs
- Вложенные стеки (опционально)

**8. Безопасность:**
- Минимальные привилегии для всех IAM ролей
- Security Groups с минимальным доступом
- Encrypted RDS и S3
- MFA включен для пользователей

### Пошаговый план:

1. **Создай CloudFormation template** со всей инфраструктурой

2. **Разверни через CLI:**
   ```bash
   aws cloudformation create-stack \
     --stack-name production-infrastructure \
     --template-body file://main-template.yaml \
     --parameters file://parameters.json \
     --capabilities CAPABILITY_IAM
   ```

3. **Проверь создание ресурсов:**
   ```bash
   aws cloudformation describe-stack-events \
     --stack-name production-infrastructure
   ```

4. **Протестируй доступность:**
   ```bash
   # Получи DNS ALB
   ALB_DNS=$(aws elbv2 describe-load-balancers \
     --query 'LoadBalancers[0].DNSName' --output text)
   
   # Проверь доступность
   curl http://$ALB_DNS
   ```

5. **Проверь Auto Scaling:**
   ```bash
   # Симулируй нагрузку на EC2
   # Наблюдай за масштабированием
   aws autoscaling describe-auto-scaling-groups \
     --auto-scaling-group-names my-asg
   ```

6. **Протестируй Lambda:**
   ```bash
   # Загрузи файл в S3
   aws s3 cp test.txt s3://my-bucket/
   
   # Проверь CloudWatch Logs Lambda
   aws logs tail /aws/lambda/my-function --follow
   ```

7. **Проверь мониторинг:**
   - Открой CloudWatch Dashboard
   - Проверь алармы
   - Посмотри CloudTrail события

8. **Документируй архитектуру:**
   - Создай diagram (можно в draw.io или lucidchart)
   - Опиши каждый компонент
   - Задокументируй Security Groups правила

### Критерии успеха:

- ✅ Вся инфраструктура создана через CloudFormation
- ✅ Веб-приложение доступно через ALB DNS
- ✅ Auto Scaling работает (scale up/down)
- ✅ RDS доступна из EC2 в приватной подсети
- ✅ Lambda триггерится от S3 событий
- ✅ CloudWatch алармы работают
- ✅ SNS уведомления приходят
- ✅ Все Security Groups настроены правильно
- ✅ CloudTrail логирует все действия

---

## Чеклист навыков AWS DevOps/SysAdmin

После прохождения курса ты должен уметь:

### Базовые навыки:
- [ ] Настраивать AWS CLI и работать с профилями
- [ ] Создавать и управлять IAM пользователями, группами, ролями
- [ ] Запускать и управлять EC2 инстансами
- [ ] Создавать VPC, подсети, route tables
- [ ] Настраивать Security Groups
- [ ] Работать с S3 (загрузка, скачивание, политики)

### Продвинутые навыки:
- [ ] Создавать и управлять Load Balancers
- [ ] Настраивать Auto Scaling Groups
- [ ] Работать с RDS (создание, backup, restore)
- [ ] Создавать Lambda функции
- [ ] Интегрировать API Gateway с Lambda
- [ ] Писать CloudFormation templates
- [ ] Настраивать CloudWatch алармы и дашборды

### Безопасность:
- [ ] Применять принцип минимальных привилегий
- [ ] Создавать кастомные IAM политики
- [ ] Настраивать MFA
- [ ] Использовать IAM роли вместо ключей
- [ ] Шифровать данные в S3 и RDS
- [ ] Настраивать CloudTrail для аудита

### Мониторинг:
- [ ] Создавать CloudWatch алармы
- [ ] Работать с CloudWatch Logs
- [ ] Анализировать метрики
- [ ] Настраивать SNS уведомления
- [ ] Использовать CloudWatch Insights

### Infrastructure as Code:
- [ ] Писать CloudFormation templates
- [ ] Использовать параметры и outputs
- [ ] Создавать change sets
- [ ] Работать с nested stacks

---

## Полезные ресурсы

**Официальная документация:**
- AWS Documentation: https://docs.aws.amazon.com/
- AWS CLI Reference: https://docs.aws.amazon.com/cli/
- CloudFormation Templates: https://github.com/awslabs/aws-cloudformation-templates

**Обучение:**
- AWS Free Tier: https://aws.amazon.com/free/
- AWS Training: https://aws.amazon.com/training/
- AWS Well-Architected: https://aws.amazon.com/architecture/well-architected/

**Инструменты:**
- AWS CLI: https://aws.amazon.com/cli/
- AWS SAM: https://aws.amazon.com/serverless/sam/
- AWS CDK: https://aws.amazon.com/cdk/
- Terraform: https://www.terraform.io/

**Community:**
- AWS Reddit: https://reddit.com/r/aws
- AWS Forums: https://forums.aws.amazon.com/
- Stack Overflow: https://stackoverflow.com/questions/tagged/aws

**Сертификации:**
- AWS Certified Solutions Architect - Associate
- AWS Certified Developer - Associate
- AWS Certified SysOps Administrator - Associate
- AWS Certified DevOps Engineer - Professional

---

## Советы по прохождению курса

1. **Используй Free Tier.** Большинство заданий можно выполнить бесплатно в рамках AWS Free Tier.

2. **Всегда удаляй ресурсы.** После каждого модуля удаляй созданные ресурсы чтобы не платить.

3. **Используй CloudFormation.** Это упрощает создание и удаление инфраструктуры.

4. **Веди заметки.** Создай свою базу знаний с часто используемыми командами.

5. **Практикуй на реальных кейсах.** Применяй знания на своих проектах.

6. **Следи за биллингом.** Настрой Billing Alerts чтобы контролировать расходы:
   ```bash
   aws cloudwatch put-metric-alarm \
     --alarm-name billing-alarm \
     --alarm-description "Alert when charges exceed $10" \
     --metric-name EstimatedCharges \
     --namespace AWS/Billing \
     --statistic Maximum \
     --period 21600 \
     --evaluation-periods 1 \
     --threshold 10 \
     --comparison-operator GreaterThanThreshold
   ```

7. **Автоматизируй.** Любую задачу, которую делаешь дважды, стоит автоматизировать скриптом.

8. **Используй теги.** Всегда тегируй ресурсы для легкого управления и контроля затрат.

9. **Читай документацию.** AWS документация очень подробная - используй её.

10. **Учись у других.** Изучай open-source проекты и AWS Well-Architected Framework.

---

## Типичные ошибки и как их избежать

### ❌ Ошибка 1: Забыл удалить ресурсы
```bash
# Всегда проверяй активные ресурсы
aws ec2 describe-instances --query 'Reservations[].Instances[].[InstanceId,State.Name]'
aws rds describe-db-instances --query 'DBInstances[].[DBInstanceIdentifier,DBInstanceStatus]'
aws elb describe-load-balancers --query 'LoadBalancerDescriptions[].[LoadBalancerName]'
```

### ❌ Ошибка 2: Открытые Security Groups
```bash
# Плохо - доступ отовсюду
--cidr 0.0.0.0/0

# Хорошо - только с определенного IP
--cidr 203.0.113.0/24
```

### ❌ Ошибка 3: Хранение ключей в коде
```bash
# Плохо
aws s3 cp file.txt s3://bucket/ --profile prod

# Хорошо - используй IAM роли для EC2
# Не нужны ключи, инстанс использует роль
```

### ❌ Ошибка 4: Использование root аккаунта
```bash
# Всегда создавай IAM пользователей
# Включай MFA для root
# Используй root только для billing
```

### ❌ Ошибка 5: Нет бэкапов
```bash
# Всегда настраивай автоматические бэкапы
aws rds modify-db-instance \
  --db-instance-identifier mydb \
  --backup-retention-period 7
```

---

## План повторения

### При первом прохождении (3-4 часа):
- Модули 1-4 обязательно (основы)
- Модули 5-6 базовые задания
- Модуль 7-8 прочитать и попрактиковать
- Начать финальный проект (минимум VPC + EC2)

### При повторном прохождении (через 6 месяцев):
- Фокус на бонусные задания
- Модули 5-8 полностью
- Сделать финальный проект полностью
- Добавить свои кейсы из реальной работы

### Для закрепления:
- Автоматизируй деплой своих проектов в AWS
- Создай полноценное production окружение
- Настрой CI/CD пайплайн (CodePipeline, GitHub Actions)
- Изучи новые сервисы (EKS, Fargate, App Runner)
- Получи сертификацию AWS

### Что отслеживать при повторных прохождениях:
- ✅ Помню ли основные AWS CLI команды без подглядывания?
- ✅ Могу ли быстро создать VPC с правильной настройкой сети?
- ✅ Уверенно ли работаю с IAM ролями и политиками?
- ✅ Могу ли написать CloudFormation template для типовой инфраструктуры?
- ✅ Знаю ли как диагностировать проблемы с помощью CloudWatch?
- ✅ Понимаю ли принципы Well-Architected Framework?

---

## Заключение

**Этот курс - не одноразовое действие, а регулярная практика!**

🎯 **Проходи курс каждые 6-12 месяцев:**
- Освежишь забытые команды
- Узнаешь новые сервисы
- Улучшишь понимание AWS
- Повысишь скорость работы

📊 **Метрики успеха:**
- Можешь создать VPC + EC2 + RDS за 15 минут
- Находишь проблемы через CloudWatch за 5 минут
- Пишешь CloudFormation template без постоянного гугления
- Понимаешь 90% AWS сервисов и их назначение
- Решаешь типичные проблемы без паники

🚀 **Следующие шаги:**
1. Пройди весь курс хотя бы один раз
2. Примени знания на своих проектах
3. Автоматизируй рутинные задачи
4. Верни курсу через 6 месяцев

💪 **Remember:**
> "Infrastructure is code. Treat it like code." 

**Удачи в освоении AWS! Пусть твои инстансы всегда будут healthy, а биллинг - минимальным!** 🎉☁️
      