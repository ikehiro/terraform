# terraform

# ChatGPTより

多層モジュール構造を持つAWSのTerraformサンプルリポジトリとしては、以下のようなものが参考になります。


1. GruntworkのTerragrunt/Terraformサンプル 

Gruntworkは、TerragruntとTerraformのベストプラクティスに基づいた堅牢なサンプルを提供しています。多層モジュール構成の代表的な例で、インフラストラクチャをコンポーネントごとに細かく分割し、それを組み合わせることで環境を構築する形式です。 

- 特徴:
	- 階層構造:複数の環境（stage、prodなど）を管理するディレクトリと、再利用可能なモジュールを格納するディレクトリが分かれています。
	- Terragrunt:同じモジュールを異なる環境にデプロイする際のコード重複を防ぐために、Terragruntを使用する例が多く見られます。
	- AWS構成:VPC、EC2、RDSなど、一般的なAWSリソースを多層モジュールで構築するサンプルが含まれています。 

2. AWS Prescriptive Guidanceのサンプル 

AWS自身が提供する「Prescriptive Guidance」には、Terraformのベストプラクティスに関するドキュメントとサンプルが含まれています。特に、コードベースの構造と組織化に関するセクションが参考になります。 

- 特徴:
	- モジュール化:再利用可能なモジュールを環境やコンポーネント別に分割する構成が推奨されています。
	- 公式の推奨:AWS公式の視点から、Terraform AWS Providerを使用する上でのベストプラクティスが紹介されています。
	- 構成例:ドキュメント内で、モジュールと、それを呼び出すルートモジュールの関係性が図示されています。 

3. Terraform AWS Modulesの活用 

Terraform Registryで公開されている公式のAWSモジュール群は、単体で利用するだけでなく、多層構成の基盤として非常に有用です。これらのモジュールを組み合わせて、独自の多層構造を構築するサンプルも参考になります。 

- 特徴:
	- 高レベルの抽象化:terraform-aws-modulesのVPCモジュールやRDSモジュールなどを使用することで、低レベルなリソース定義を抽象化できます。
	- 依存関係の可視化:組み合わせて使うことで、モジュール間の依存関係を明確にした構成を学べます。
	- 公式リポジトリ:多数のモジュールが公開されているため、幅広いユースケースのサンプルを調べることができます。 

4. GitHub上の学習リポジトリ

GitHub上には、個人や企業がベストプラクティスを共有しているリポジトリが多数存在します。

- 例:
	- 3層Webアプリケーション:3層アーキテクチャ（ウェブ、アプリケーション、データベース）をTerraformで構築するサンプルです。VPC、サブネット、EC2、RDSなどのリソースがモジュール化されて構成されています。
	- シンプルながら多層:ネットワークレイヤー、アプリケーションレイヤーといった層ごとにモジュールを分け、それらをルートモジュールで呼び出す基本的な多層構成の例も良い参考になります。 

どのようなリポジトリから始めるべきか

もしTerraformの多層モジュールを学び始めたばかりであれば、HashiCorpの公式チュートリアルやMediumなどで解説されている、小規模な3層アーキテクチャのサンプルから始めるのがおすすめです。 

より本格的なエンタープライズレベルの構成を目指す場合は、GruntworkのTerragrunt/Terraformサンプルが、Terraformのベストプラクティスを網羅しており参考になります。ただし、Terragruntの学習コストが別途発生します。 




既存スクリプトのモジュールには手を入れない。
バッチモジュールを追加
つなぎは環境側
batch
  Network
  Ecr
  Lambda
  S3
  Cloudwatch
  Event bridge




Terraformのコードをモジュール化する方法
了解です。
Terraformで「モジュール自体が内部でさらにサブモジュールを呼ぶ」多段構成の AWS サンプルを示します。環境ごと（dev/prod など）の分離、VPC→（subnets / nat / routes）という入れ子、ALB と ECS サービスまでを最小限で動くレベルにまとめました。

---



ディレクトリ構成（例）


```
terraform-multi-level-modules/
├─ live/
│  └─ dev-us-east-1/
│     ├─ backend.tf
│     ├─ providers.tf
│     ├─ versions.tf
│     ├─ variables.tf
│     ├─ main.tf
│     └─ outputs.tf
└─ modules/
   ├─ network/
   │  ├─ main.tf
   │  ├─ variables.tf
   │  ├─ outputs.tf
   │  └─ submodules/
   │     ├─ subnets/
   │     │  ├─ main.tf
   │     │  └─ variables.tf
   │     ├─ nat/
   │     │  ├─ main.tf
   │     │  └─ variables.tf
   │     └─ routes/
   │        ├─ main.tf
   │        └─ variables.tf
   ├─ alb/
   │  ├─ main.tf
   │  ├─ variables.tf
   │  └─ outputs.tf
   └─ compute/
      └─ ecs-service/
         ├─ main.tf
         ├─ variables.tf
         ├─ outputs.tf
         └─ submodules/
            └─ task/
               ├─ main.tf
               └─ variables.tf
```

---



live（環境側）





live/dev-us-east-1/versions.tf


```
terraform {
  required_version = ">= 1.6.0"
  required_providers {
    aws = {
      source  = "hashicorp/aws"
      version = "~> 5.60"
    }
  }
}
```


live/dev-us-east-1/backend.tf（S3 リモートステート例）


```
terraform {
  backend "s3" {
    bucket         = "my-tfstate-bucket"
    key            = "dev-us-east-1/infra.tfstate"
    region         = "us-east-1"
    dynamodb_table = "my-tfstate-locks"
    encrypt        = true
  }
}
```


live/dev-us-east-1/providers.tf


```
provider "aws" {
  region = var.region
  default_tags {
    tags = {
      Project = "sample-app"
      Env     = "dev"
    }
  }
}
```


live/dev-us-east-1/variables.tf


```
variable "region" {
  type        = string
  description = "AWS region"
  default     = "us-east-1"
}
```


live/dev-us-east-1/main.tf


```
locals {
  azs = ["us-east-1a", "us-east-1b"]
  tags = {
    Owner   = "platform-team"
    Purpose = "demo"
  }
}

module "network" {
  source               = "../../modules/network"
  vpc_cidr             = "10.10.0.0/16"
  azs                  = local.azs
  public_subnet_cidrs  = ["10.10.0.0/24", "10.10.1.0/24"]
  private_subnet_cidrs = ["10.10.10.0/24", "10.10.11.0/24"]
  create_nat           = true
  tags                 = local.tags
}

module "alb" {
  source            = "../../modules/alb"
  vpc_id            = module.network.vpc_id
  subnet_ids        = module.network.public_subnet_ids
  security_group_id = module.network.alb_sg_id
  tags              = local.tags
}

module "ecs_service" {
  source             = "../../modules/compute/ecs-service"
  vpc_id             = module.network.vpc_id
  private_subnet_ids = module.network.private_subnet_ids
  alb_listener_arn   = module.alb.http_listener_arn
  desired_count      = 2
  service_name       = "sample-ecs-svc"
  container_image    = "public.ecr.aws/amazonlinux/amazonlinux:2023" # デモ用
  container_port     = 8080
  tags               = local.tags
}
```


live/dev-us-east-1/outputs.tf


```
output "alb_dns_name" {
  value = module.alb.alb_dns_name
}

output "service_name" {
  value = module.ecs_service.service_name
}
```

---



modules/network（上位モジュール → サブモジュール呼び出し）





modules/network/variables.tf


```
variable "vpc_cidr"             { type = string }
variable "azs"                  { type = list(string) }
variable "public_subnet_cidrs"  { type = list(string) }
variable "private_subnet_cidrs" { type = list(string) }
variable "create_nat"           { type = bool   default = true }
variable "tags"                 { type = map(string) default = {} }
```


modules/network/main.tf


```
resource "aws_vpc" "this" {
  cidr_block           = var.vpc_cidr
  enable_dns_hostnames = true
  enable_dns_support   = true
  tags = merge(var.tags, { Name = "demo-vpc" })
}

resource "aws_security_group" "alb" {
  name        = "alb-sg"
  description = "ALB SG"
  vpc_id      = aws_vpc.this.id

  ingress {
    description = "HTTP"
    from_port   = 80
    to_port     = 80
    protocol    = "tcp"
    cidr_blocks = ["0.0.0.0/0"]
  }

  egress {
    from_port   = 0
    to_port     = 0
    protocol    = "-1"
    cidr_blocks = ["0.0.0.0/0"]
  }

  tags = merge(var.tags, { Name = "alb-sg" })
}

# --- サブモジュール呼び出し ---
module "subnets" {
  source               = "./submodules/subnets"
  vpc_id               = aws_vpc.this.id
  azs                  = var.azs
  public_subnet_cidrs  = var.public_subnet_cidrs
  private_subnet_cidrs = var.private_subnet_cidrs
  tags                 = var.tags
}

module "nat" {
  source          = "./submodules/nat"
  vpc_id          = aws_vpc.this.id
  public_subnets  = module.subnets.public_subnets
  create_nat      = var.create_nat
  tags            = var.tags
  depends_on      = [module.subnets]
}

module "routes" {
  source               = "./submodules/routes"
  vpc_id               = aws_vpc.this.id
  igw_id               = module.subnets.igw_id
  public_subnets       = module.subnets.public_subnets
  private_subnets      = module.subnets.private_subnets
  nat_gateway_ids      = module.nat.nat_gateway_ids
  enable_private_nat   = var.create_nat
  tags                 = var.tags
  depends_on           = [module.nat]
}
```


modules/network/outputs.tf


```
output "vpc_id"             { value = aws_vpc.this.id }
output "public_subnet_ids"  { value = [for s in module.subnets.public_subnets : s.id] }
output "private_subnet_ids" { value = [for s in module.subnets.private_subnets : s.id] }
output "alb_sg_id"          { value = aws_security_group.alb.id }
```


submodules/subnets





variables.tf


```
variable "vpc_id"              { type = string }
variable "azs"                 { type = list(string) }
variable "public_subnet_cidrs" { type = list(string) }
variable "private_subnet_cidrs"{ type = list(string) }
variable "tags"                { type = map(string) default = {} }
```


main.tf


```
resource "aws_internet_gateway" "igw" {
  vpc_id = var.vpc_id
  tags   = merge(var.tags, { Name = "demo-igw" })
}

resource "aws_subnet" "public" {
  for_each          = toset(var.public_subnet_cidrs)
  vpc_id            = var.vpc_id
  cidr_block        = each.value
  map_public_ip_on_launch = true
  availability_zone = var.azs[tonumber(regex("\\.(\\d+)/", "${each.value}/") ) % length(var.azs)] # 簡易な分散例（実運用は明示指定推奨）
  tags = merge(var.tags, { Name = "public-${each.value}" })
}

resource "aws_subnet" "private" {
  for_each          = toset(var.private_subnet_cidrs)
  vpc_id            = var.vpc_id
  cidr_block        = each.value
  map_public_ip_on_launch = false
  availability_zone = var.azs[tonumber(regex("\\.(\\d+)/", "${each.value}/") ) % length(var.azs)]
  tags = merge(var.tags, { Name = "private-${each.value}" })
}

output "igw_id" {
  value = aws_internet_gateway.igw.id
}

output "public_subnets" {
  value = values(aws_subnet.public)
}

output "private_subnets" {
  value = values(aws_subnet.private)
}
```


submodules/nat





variables.tf


```
variable "vpc_id"         { type = string }
variable "public_subnets" { type = list(object({ id = string })) }
variable "create_nat"     { type = bool  default = true }
variable "tags"           { type = map(string) default = {} }
```


main.tf


```
locals {
  nat_count = var.create_nat ? length(var.public_subnets) : 0
}

resource "aws_eip" "this" {
  count      = local.nat_count
  domain     = "vpc"
  tags       = merge(var.tags, { Name = "nat-eip-${count.index}" })
}

resource "aws_nat_gateway" "this" {
  count         = local.nat_count
  subnet_id     = var.public_subnets[count.index].id
  allocation_id = aws_eip.this[count.index].id
  tags          = merge(var.tags, { Name = "nat-${count.index}" })
}

output "nat_gateway_ids" {
  value = [for n in aws_nat_gateway.this : n.id]
}
```


submodules/routes





variables.tf


```
variable "vpc_id"             { type = string }
variable "igw_id"             { type = string }
variable "public_subnets"     { type = list(object({ id = string })) }
variable "private_subnets"    { type = list(object({ id = string })) }
variable "nat_gateway_ids"    { type = list(string) }
variable "enable_private_nat" { type = bool }
variable "tags"               { type = map(string) default = {} }
```


main.tf


```
# Public route table (-> IGW)
resource "aws_route_table" "public" {
  vpc_id = var.vpc_id
  tags   = merge(var.tags, { Name = "public-rt" })
}
resource "aws_route" "public_igw" {
  route_table_id         = aws_route_table.public.id
  destination_cidr_block = "0.0.0.0/0"
  gateway_id             = var.igw_id
}
resource "aws_route_table_association" "public_assoc" {
  for_each       = { for s in var.public_subnets : s.id => s }
  subnet_id      = each.value.id
  route_table_id = aws_route_table.public.id
}

# Private route tables (-> NAT GW per-AZ 風に割当)
resource "aws_route_table" "private" {
  count  = var.enable_private_nat ? length(var.private_subnets) : 0
  vpc_id = var.vpc_id
  tags   = merge(var.tags, { Name = "private-rt-${count.index}" })
}
resource "aws_route" "private_nat" {
  count                  = var.enable_private_nat ? length(var.private_subnets) : 0
  route_table_id         = aws_route_table.private[count.index].id
  destination_cidr_block = "0.0.0.0/0"
  nat_gateway_id         = var.nat_gateway_ids[count.index % max(length(var.nat_gateway_ids),1)]
}
resource "aws_route_table_association" "private_assoc" {
  count         = var.enable_private_nat ? length(var.private_subnets) : 0
  subnet_id     = var.private_subnets[count.index].id
  route_table_id= aws_route_table.private[count.index].id
}
```

---



modules/alb





variables.tf


```
variable "vpc_id"            { type = string }
variable "subnet_ids"        { type = list(string) }
variable "security_group_id" { type = string }
variable "tags"              { type = map(string) default = {} }
```


main.tf


```
resource "aws_lb" "this" {
  name               = "demo-alb"
  load_balancer_type = "application"
  security_groups    = [var.security_group_id]
  subnets            = var.subnet_ids
  tags               = merge(var.tags, { Name = "demo-alb" })
}

resource "aws_lb_target_group" "this" {
  name     = "demo-tg"
  port     = 8080
  protocol = "HTTP"
  vpc_id   = var.vpc_id
  health_check {
    path                = "/"
    matcher             = "200-399"
    interval            = 30
    unhealthy_threshold = 2
  }
  tags = merge(var.tags, { Name = "demo-tg" })
}

resource "aws_lb_listener" "http" {
  load_balancer_arn = aws_lb.this.arn
  port              = 80
  protocol          = "HTTP"

  default_action {
    type             = "fixed-response"
    fixed_response {
      content_type = "text/plain"
      message_body = "OK"
      status_code  = "200"
    }
  }
}

output "http_listener_arn" { value = aws_lb_listener.http.arn }
output "alb_dns_name"      { value = aws_lb.this.dns_name }
```

---



modules/compute/ecs-service（内部に task サブモジュール）





variables.tf


```
variable "vpc_id"             { type = string }
variable "private_subnet_ids" { type = list(string) }
variable "alb_listener_arn"   { type = string }
variable "service_name"       { type = string }
variable "container_image"    { type = string }
variable "container_port"     { type = number default = 8080 }
variable "desired_count"      { type = number default = 1 }
variable "tags"               { type = map(string) default = {} }
```


main.tf


```
resource "aws_security_group" "ecs" {
  name   = "${var.service_name}-ecs-sg"
  vpc_id = var.vpc_id

  ingress {
    description = "from ALB"
    from_port   = var.container_port
    to_port     = var.container_port
    protocol    = "tcp"
    cidr_blocks = ["0.0.0.0/0"] # デモ用: 実運用は ALB SG からのみに制限推奨
  }
  egress { from_port = 0 to_port = 0 protocol = "-1" cidr_blocks = ["0.0.0.0/0"] }

  tags = merge(var.tags, { Name = "${var.service_name}-ecs-sg" })
}

resource "aws_ecs_cluster" "this" {
  name = "${var.service_name}-cluster"
  tags = var.tags
}

module "task" {
  source          = "./submodules/task"
  family          = "${var.service_name}-task"
  container_image = var.container_image
  container_port  = var.container_port
  tags            = var.tags
}

resource "aws_lb_target_group" "ecs" {
  name        = "${var.service_name}-tg"
  port        = var.container_port
  protocol    = "HTTP"
  target_type = "ip"
  vpc_id      = var.vpc_id
  health_check { path = "/" }
  tags = var.tags
}

resource "aws_lb_listener_rule" "ecs" {
  listener_arn = var.alb_listener_arn
  priority     = 100
  action {
    type             = "forward"
    target_group_arn = aws_lb_target_group.ecs.arn
  }
  condition { path_pattern { values = ["/", "/app*"] } }
}

resource "aws_iam_role" "ecs_task" {
  name               = "${var.service_name}-task-exec"
  assume_role_policy = data.aws_iam_policy_document.ecs_assume.json
  tags               = var.tags
}

data "aws_iam_policy_document" "ecs_assume" {
  statement {
    actions = ["sts:AssumeRole"]
    principals { type = "Service" identifiers = ["ecs-tasks.amazonaws.com"] }
  }
}

resource "aws_iam_role_policy_attachment" "exec" {
  role       = aws_iam_role.ecs_task.name
  policy_arn = "arn:aws:iam::aws:policy/service-role/AmazonECSTaskExecutionRolePolicy"
}

resource "aws_ecs_service" "this" {
  name            = var.service_name
  cluster         = aws_ecs_cluster.this.id
  launch_type     = "FARGATE"
  desired_count   = var.desired_count
  platform_version= "1.4.0"

  network_configuration {
    subnets         = var.private_subnet_ids
    security_groups = [aws_security_group.ecs.id]
    assign_public_ip= false
  }

  load_balancer {
    target_group_arn = aws_lb_target_group.ecs.arn
    container_name   = module.task.container_name
    container_port   = var.container_port
  }

  task_definition = module.task.task_definition_arn
  depends_on      = [aws_lb_listener_rule.ecs]
  tags            = var.tags
}

output "service_name" {
  value = aws_ecs_service.this.name
}
```


submodules/task





variables.tf


```
variable "family"          { type = string }
variable "container_image" { type = string }
variable "container_port"  { type = number }
variable "tags"            { type = map(string) default = {} }
```


main.tf


```
resource "aws_cloudwatch_log_group" "this" {
  name              = "/ecs/${var.family}"
  retention_in_days = 7
  tags              = var.tags
}

resource "aws_ecs_task_definition" "this" {
  family                   = var.family
  network_mode             = "awsvpc"
  cpu                      = "256"
  memory                   = "512"
  requires_compatibilities = ["FARGATE"]
  execution_role_arn       = null
  container_definitions = jsonencode([
    {
      name      = "app"
      image     = var.container_image
      essential = true
      portMappings = [{ containerPort = var.container_port, protocol = "tcp" }]
      logConfiguration = {
        logDriver = "awslogs"
        options   = {
          awslogs-group         = aws_cloudwatch_log_group.this.name
          awslogs-region        = data.aws_region.current.name
          awslogs-stream-prefix = "ecs"
        }
      }
      command = ["sh", "-c", "python3 -m http.server ${var.container_port}"]
    }
  ])
  tags = var.tags
}

data "aws_region" "current" {}
output "task_definition_arn" { value = aws_ecs_task_definition.this.arn }
output "container_name"      { value = "app" }
```

---



使い方（dev 環境）


```
cd terraform-multi-level-modules/live/dev-us-east-1
terraform init
terraform plan -var="region=us-east-1"
terraform apply -auto-approve
```

---



ポイント（設計の肝）



- 多段モジュール化
  
	- modules/network が 上位モジュールとして VPC を持ち、submodules/{subnets,nat,routes} を呼び出します。
	- modules/compute/ecs-service も同様に 上位モジュールで、submodules/task にタスク定義を委譲。
- 
- 疎結合な出力
  
	- network は VPC ID / public/private subnet IDs / ALB用SG を出力し、ALB や ECS へ渡す。
- 
- 条件分岐（NAT 有無）
  
	- create_nat で NAT 作成・ルート迂回を切り替え可能。
- 
- 環境分離
  
	- live/ 配下で dev/prod を分け、同じ modules を再利用。バックエンド（S3）もキーで分離。
- 
- Tag とデフォルトタグ
  
	- provider "aws".default_tags と tags 変数の両建てで、運用時の柔軟性確保。
- 
- 優先度の高いベストプラクティス（抜粋）
  
	- 重要リソースは 上位モジュール、反復・横展開される細部は サブモジュールへ。
	- 出力の粒度は、他モジュールが参照しやすい ID/ARN/リスト中心に。
	- for_each と count を用途別に使い分け（ID キーなら for_each が安全）。
	- セキュリティグループは最小権限（デモでは簡略化。実運用は ALB SG→ECS SG のみ許可が推奨）。
- 


---

必要なら、この構成を EKS や RDS、S3 + CloudFront などにも拡張できます。
「prod も作りたい」「WAF を噛ませたい」「PrivateLink を追加したい」など、次の要件を教えてくれれば、この雛形をそのまま広げます。
たぶん「Terraformのコードをモジュール化する方法」ですよね。要点だけサクッとまとめつつ、すぐ使える最小例を置きます。

了解！
既存構成に「EventBridge → コンテナイメージの Lambda を起動」までを“多段モジュール”で追加します。IAM は Lambda モジュール内をさらに submodules/iam に分離し、EventBridge 側は aws_lambda_permission のみで最小にしています。

---



追加ディレクトリ


```
modules/
└─ compute/
   ├─ ecs-service/            # 既存
   └─ lambda-image/           # ★ 追加（Lambda本体＋IAMを多段化）
      ├─ main.tf
      ├─ variables.tf
      ├─ outputs.tf
      └─ submodules/
         └─ iam/
            ├─ main.tf
            └─ variables.tf
modules/
└─ integration/
   └─ eventbridge-lambda/     # ★ 追加（EventBridge→Lambda の配線）
      ├─ main.tf
      └─ variables.tf
```

---



modules/compute/lambda-image/variables.tf


```
variable "function_name"      { type = string }
variable "image_uri"          { type = string } # 例: "123456789012.dkr.ecr.us-east-1.amazonaws.com/myrepo:tag"
variable "role_policy_json"   { type = string  default = null }  # 追加のインラインポリシーがあれば
variable "memory_size"        { type = number  default = 512 }
variable "timeout"            { type = number  default = 60 }
variable "env"                { type = map(string) default = {} }
variable "tags"               { type = map(string) default = {} }

# VPC 内で動かす場合のみ設定（任意）
variable "vpc_subnet_ids"     { type = list(string) default = [] }
variable "vpc_security_group_ids" { type = list(string) default = [] }
```


modules/compute/lambda-image/main.tf


```
locals {
  use_vpc = length(var.vpc_subnet_ids) > 0 && length(var.vpc_security_group_ids) > 0
}

module "iam" {
  source        = "./submodules/iam"
  function_name = var.function_name
  use_vpc       = local.use_vpc
  role_policy_json = var.role_policy_json
  tags          = var.tags
}

resource "aws_cloudwatch_log_group" "this" {
  name              = "/aws/lambda/${var.function_name}"
  retention_in_days = 14
  tags              = var.tags
}

resource "aws_lambda_function" "this" {
  function_name = var.function_name
  package_type  = "Image"
  image_uri     = var.image_uri
  role          = module.iam.role_arn
  timeout       = var.timeout
  memory_size   = var.memory_size
  architectures = ["x86_64"] # arm64 にしたければ ["arm64"]

  dynamic "vpc_config" {
    for_each = local.use_vpc ? [1] : []
    content {
      subnet_ids         = var.vpc_subnet_ids
      security_group_ids = var.vpc_security_group_ids
    }
  }

  environment {
    variables = var.env
  }

  tags = var.tags
}

output "lambda_arn"  { value = aws_lambda_function.this.arn }
output "lambda_name" { value = aws_lambda_function.this.function_name }
```


modules/compute/lambda-image/outputs.tf


```
output "role_arn" { value = module.iam.role_arn }
```


modules/compute/lambda-image/submodules/iam/variables.tf


```
variable "function_name"    { type = string }
variable "use_vpc"          { type = bool   default = false }
variable "role_policy_json" { type = string  default = null }
variable "tags"             { type = map(string) default = {} }
```


modules/compute/lambda-image/submodules/iam/main.tf


```
data "aws_iam_policy_document" "assume" {
  statement {
    actions = ["sts:AssumeRole"]
    principals { type = "Service" identifiers = ["lambda.amazonaws.com"] }
  }
}

resource "aws_iam_role" "this" {
  name               = "${var.function_name}-exec"
  assume_role_policy = data.aws_iam_policy_document.assume.json
  tags               = var.tags
}

# CloudWatch Logs 出力
resource "aws_iam_role_policy_attachment" "basic" {
  role       = aws_iam_role.this.name
  policy_arn = "arn:aws:iam::aws:policy/service-role/AWSLambdaBasicExecutionRole"
}

# VPC 内で動かす場合の ENI 作成権限
resource "aws_iam_role_policy_attachment" "vpc" {
  count      = var.use_vpc ? 1 : 0
  role       = aws_iam_role.this.name
  policy_arn = "arn:aws:iam::aws:policy/service-role/AWSLambdaVPCAccessExecutionRole"
}

# 追加の最小権限をインラインで差し込みたい場合
resource "aws_iam_role_policy" "inline" {
  count  = var.role_policy_json == null ? 0 : 1
  name   = "${var.function_name}-inline"
  role   = aws_iam_role.this.id
  policy = var.role_policy_json
}

output "role_arn" { value = aws_iam_role.this.arn }
```

---



modules/integration/eventbridge-lambda/variables.tf


```
variable "rule_name"           { type = string }
variable "description"         { type = string  default = "Invoke Lambda on schedule/event" }
variable "schedule_expression" { type = string  default = null } # 例 "rate(5 minutes)" or "cron(0/5 * * * ? *)"
variable "event_pattern_json"  { type = string  default = null } # JSON string
variable "lambda_arn"          { type = string }
variable "lambda_name"         { type = string }
variable "tags"                { type = map(string) default = {} }
```


modules/integration/eventbridge-lambda/main.tf


```
# どちらか一方（schedule / pattern）を使う
resource "aws_cloudwatch_event_rule" "this" {
  name        = var.rule_name
  description = var.description

  dynamic "schedule_expression" {
    for_each = var.schedule_expression != null ? [1] : []
    content  = var.schedule_expression
  }

  dynamic "event_pattern" {
    for_each = var.event_pattern_json != null ? [1] : []
    content  = var.event_pattern_json
  }

  tags = var.tags
}

resource "aws_cloudwatch_event_target" "to_lambda" {
  rule      = aws_cloudwatch_event_rule.this.name
  target_id = "lambda"
  arn       = var.lambda_arn
}

# EventBridge から Lambda を叩けるようにする Permission
resource "aws_lambda_permission" "allow_events" {
  statement_id  = "AllowExecutionFromEventBridge"
  action        = "lambda:InvokeFunction"
  function_name = var.lambda_name
  principal     = "events.amazonaws.com"
  source_arn    = aws_cloudwatch_event_rule.this.arn
}
```

---



live/dev-us-east-1/main.tf への追記例


```
# ========== Lambda（コンテナイメージ） ==========
module "lambda_img" {
  source        = "../../modules/compute/lambda-image"
  function_name = "sample-image-lambda"
  image_uri     = "123456789012.dkr.ecr.us-east-1.amazonaws.com/sample:latest" # ★差し替え

  # VPC 内で動かすなら network モジュールの private に接続
  vpc_subnet_ids         = module.network.private_subnet_ids
  vpc_security_group_ids = [module.network.alb_sg_id] # ★デモ。実運用は Lambda 用 SG を別途作成推奨

  memory_size = 512
  timeout     = 60
  env = {
    STAGE = "dev"
  }
  tags = local.tags
}

# ========== EventBridge → Lambda 配線（5分おきに起動） ==========
module "evb_lambda" {
  source              = "../../modules/integration/eventbridge-lambda"
  rule_name           = "every-5-mins"
  schedule_expression = "rate(5 minutes)"
  lambda_arn          = module.lambda_img.lambda_arn
  lambda_name         = module.lambda_img.lambda_name
  tags                = local.tags
}
```

※ ECR のリポジトリ作成やイメージの push は別途行ってください（aws_ecr_repository を live 側に追加してもOK）。
---



配置と責務分割（IAM をどこに置くか）



- Lambda 本体モジュール（modules/compute/lambda-image）
  
	- さらに submodules/iam に “実行ロール（Execution Role）” を分離
	- 役割：AWSLambdaBasicExecutionRole（ログ出力）、VPC 利用時は AWSLambdaVPCAccessExecutionRole を 条件付きで添付
	- 追加最小権限は role_policy_json で 上位から注入（S3/Parameter Store/Secrets Manager など、用途別に最小権限で付与）
- 
- EventBridge モジュール（modules/integration/eventbridge-lambda）
  
	- ロール不要（Lambda ターゲットの場合は aws_lambda_permission で十分）
	- CloudWatch Events ルール & ターゲットを管理し、source_arn を指定した aws_lambda_permission を作る
- 


この分割により、

- Lambda の IAM は Lambda に近い位置で閉じ、最小権限の注入だけ上位から行える
- **イベント配線（EventBridge）**は 結線専用モジュールとして再利用しやすい
- live 側（環境）では「どのイメージを、どの頻度で動かすか」を宣言するだけ、という“宣言レベルの責務”に集中できます


---



おまけ：ECR を Terraform で一緒に作るなら（任意）


```
# live/dev-us-east-1/main.tf など
resource "aws_ecr_repository" "lambda_repo" {
  name                 = "sample"
  image_scanning_configuration { scan_on_push = true }
  force_delete         = true
  tags = local.tags
}

# module.lambda_img の image_uri を下記のように参照可能
# "${aws_ecr_repository.lambda_repo.repository_url}:latest"
```

---

必要なら、DLQ（SQS）や再試行ポリシー, EventBridge イベントパターン（S3 Put, CodeCommit, Step Functions など）, Secrets Manager 連携も、この骨格のまま増設できます。
「VPC を別 SG に」「環境ごとにスケジュールを変える」「複数ルールから同一 Lambda をキック」など、追加要件があれば続けてどうぞ。



目的



- 使い回し／再現性／レビュー効率を上げる
- 変更影響を局所化（1モジュール＝1責務）
- 環境（dev/stg/prd）差分を入力値だけに寄せる




典型ディレクトリ構成


```
infra/
├─ modules/
│  ├─ vpc/
│  │  ├─ main.tf
│  │  ├─ variables.tf
│  │  ├─ outputs.tf
│  │  └─ README.md
│  └─ ec2/
│     ├─ main.tf
│     ├─ variables.tf
│     ├─ outputs.tf
│     └─ README.md
└─ envs/
   ├─ dev/
   │  ├─ main.tf         # ここが「ルートモジュール」
   │  ├─ versions.tf
   │  └─ terraform.tfvars
   └─ prd/
      ├─ main.tf
      ├─ versions.tf
      └─ terraform.tfvars
```


モジュール（例：modules/vpc）



main.tf
```
terraform {
  required_version = ">= 1.6.0"
  required_providers {
    aws = { source = "hashicorp/aws", version = "~> 5.0" }
  }
}

resource "aws_vpc" "this" {
  cidr_block           = var.cidr_block
  enable_dns_support   = true
  enable_dns_hostnames = true
  tags = merge(var.tags, { Name = var.name })
}

output "vpc_id" { value = aws_vpc.this.id }
```
variables.tf
```
variable "name"       { type = string }
variable "cidr_block" {
  type = string
  validation {
    condition     = can(cidrnetmask(var.cidr_block))
    error_message = "cidr_block は有効なCIDRである必要があります。"
  }
}
variable "tags"       { type = map(string), default = {} }
```
outputs.tf
```
output "arn" { value = aws_vpc.this.arn }
```


ルートから呼び出す（envs/dev/main.tf）


```
terraform {
  backend "s3" {
    bucket = "my-tfstate"
    key    = "dev/terraform.tfstate"
    region = "ap-northeast-1"
  }
}

provider "aws" {
  region = "ap-northeast-1"
}

module "vpc" {
  source     = "../../modules/vpc"
  name       = "dev-vpc"
  cidr_block = var.vpc_cidr
  tags       = var.common_tags
}

# 別モジュールにVPC情報を渡す例
module "ec2" {
  source  = "../../modules/ec2"
  vpc_id  = module.vpc.vpc_id
  # 他の入力……
}
```
envs/dev/terraform.tfvars
```
vpc_cidr     = "10.10.0.0/16"
common_tags  = { Project = "sample", Env = "dev" }
```
envs/dev/versions.tf
```
terraform {
  required_version = ">= 1.6.0"
  required_providers {
    aws = { source = "hashicorp/aws", version = "~> 5.0" }
  }
}
```


環境分離の考え方



- **ルート（envs/dev, envs/prd）**は「配線」と「値の違い」だけを持つ
- モジュールはロジックを持ち、環境非依存（タグや名前などは入力で受ける）
- tfstate は環境ごとに分離（S3キー/ワークスペース/別バケットのいずれか）




依存関係の渡し方



- モジュール間は output → input でつなぐ（上の module.ec2 例）
- 複数作成は for_each 推奨（差分に強い）

```
module "subnets" {
  source = "../../modules/subnet"
  for_each = var.subnets  # map(string)
  vpc_id   = module.vpc.vpc_id
  cidr     = each.value
  name     = "subnet-${each.key}"
}
```


モジュールの配布・バージョン固定



- source は以下が使えます
  
	- ローカル: source = "../../modules/vpc"
	- Git: source = "git::https://github.com/org/iac.git//modules/vpc?ref=v1.2.3"
	- Private Registry: source = "app.terraform.io/org/vpc/aws" として version = "1.2.3"
- 
- ref/tag or version を必ず固定（再現性のため）




品質・運用のベストプラクティス



- 入力検証：validation ブロックで早期失敗
- デフォルトと必須：本当に必要なものだけ nullable = false / defaultなし
- 命名規約：name, tags, prefix/suffix を入力にまとめる
- ドキュメント：README.md＋examples/ を各モジュールに
- 静的解析：tflint, tfsec/checkov、pre-commit で自動チェック
- テスト：重要モジュールは terratest（Go）で最小検証
- 破壊的変更：changelogとmajor/minorのセマンティックバージョニング




アンチパターン



- 1つのモジュールに多機能を詰め込む（責務分離×）
- モジュール内で環境分岐の条件だらけ（複雑化）
- count で ID リストに依存（順序ズレ事故）→ for_each を使う
- version未固定の source（将来的な差分で壊れがち）


---

必要なら、あなたの現在のコードを貼ってくれれば、適切な分割単位（VPC/ネットワーク/セキュリティ/計算/データ/監視…）に具体的にリファクタします。

いい視点です。既存コードから「どこでディレクトリ／ファイル操作しているか」を洗い出すなら、静的と動的の両輪が効きます。すぐ使える実ファイルも用意しました（ダウンロード可）。

こちらが Gruntwork 社による Terragrunt／Terraform 用のサンプルリポジトリの URL です：

「インフラモジュール」のサンプルリポジトリ:
https://github.com/gruntwork-io/terragrunt-infrastructure-modules-example
 
GitHub

「インフラ実運用ライブ構成」のサンプルリポジトリ:
https://github.com/gruntwork-io/terragrunt-infrastructure-live-example
 
GitHub
+1

必要であれば、モジュール毎のディレクトリ構成・ファイル構成・ベストプラクティスも日本語で解説できます。

以下に、中〜大規模 Terraform プロジェクトでのおすすめファイル構成例 を示します。
GruntworkやHashiCorpの推奨構成をベースに、日本語で整理しています。

🗂️ Terraform ディレクトリ構成（例）
terraform/
├── main.tf               # メインエントリポイント（module呼び出しなど）
├── providers.tf          # プロバイダー設定 (AWS, Azure, etc.)
├── variables.tf          # 入力変数定義
├── outputs.tf            # 出力値定義
├── locals.tf             # ローカル変数定義（名前規則など）
├── data.tf               # dataソース (既存リソース参照など)
│
├── network.tf            # VPC, サブネット, ルート, NAT, GW など
├── security.tf           # IAM, Security Group, KMSなど
├── compute.tf            # EC2, ECS, Lambda, AutoScalingなど
├── storage.tf            # S3, EBS, EFS など
├── database.tf           # RDS, DynamoDB など
│
├── backend.tf            # State backend設定 (S3, GCSなど)
├── terraform.tfvars      # 変数の実値（環境別に分けることも多い）
│
└── environments/
    ├── dev/
    │   ├── main.tfvars
    │   ├── backend.tfvars
    │   └── override.tf
    ├── staging/
    └── prod/

🧩 ファイルの役割説明
ファイル名	内容
main.tf	各モジュール呼び出し・主要リソースの定義
providers.tf	provider "aws" などの設定（リージョン指定含む）
variables.tf	入力パラメータ（例：VPC CIDR, インスタンスタイプなど）
outputs.tf	terraform output で出す値を定義
locals.tf	名前規則や共通タグなどのローカル変数
data.tf	既存のAWSリソースなどを参照する data "aws_..." ブロック
backend.tf	stateファイルをS3などに保存する設定
terraform.tfvars	実際の値。環境ごとに dev.tfvars, prod.tfvars などを分けるのが定番
🌱 環境別構成の考え方

複数環境を扱う場合、以下のようにディレクトリ分けするのがベストです。

environments/
├── dev/
│   ├── terraform.tfvars
│   ├── backend.tfvars
│   └── override.tf
├── staging/
└── prod/


terraform workspace を併用するか、環境ディレクトリを分ける方式が一般的。

CI/CD（例: GitHub Actions, Terraform Cloud）と組み合わせると、環境ごとのデプロイ制御が容易になります。

💡 図による構成イメージ
┌────────────────────────────┐
│ terraform/                 │
│  ├── base設定               │
│  │   ├── main.tf           │
│  │   ├── variables.tf      │
│  │   ├── outputs.tf        │
│  │   └── modules/          │
│  │        ├── vpc/         │
│  │        ├── ec2/         │
│  │        └── rds/         │
│  │
│  └── environments/          │
│        ├── dev/             │
│        ├── staging/         │
│        └── prod/            │
└────────────────────────────┘



----------------------------------------------------------
了解です。いわゆる “ラッパーモジュール（親モジュール）→ 子モジュール” 構成で、呼び出し元（root）から子モジュールの入力を設定したい場合の定石は次の2パターンです。



パターンA：ラッパーで「受けて→そのまま渡す」（最も素直）


```
root
 └─ module "stack"（= 親/ラッパー）
      ├─ module "network"（子）
      └─ module "ecs"（子）
```


1) root（呼び出し側）


```
# live/dev/main.tf
module "stack" {
  source = "../../modules/stack"

  # 子network用の入力を親の入力として公開し、rootから値を与える
  vpc_cidr              = "10.1.0.0/16"
  public_subnet_cidrs   = ["10.1.1.0/24", "10.1.2.0/24"]

  # 子ecs用の入力も同様に公開
  ecs_name        = "web"
  ecs_desired     = 2
  ecs_task_env    = { ENV = "dev", LOG_LEVEL = "info" }
}
```


2) 親/ラッパー（受けて→子へ渡す）


```
# modules/stack/variables.tf
variable "vpc_cidr" {
  type    = string
  default = "10.0.0.0/16"
}

variable "public_subnet_cidrs" {
  type    = list(string)
  default = ["10.0.1.0/24", "10.0.2.0/24"]
}

variable "ecs_name"    { type = string  default = "app" }
variable "ecs_desired" { type = number  default = 1 }
variable "ecs_task_env"{ type = map(string) default = {} }
```

```
# modules/stack/main.tf
module "network" {
  source               = "../network"
  vpc_cidr             = var.vpc_cidr
  public_subnet_cidrs  = var.public_subnet_cidrs
}

module "ecs" {
  source        = "../ecs"
  name          = var.ecs_name
  desired_count = var.ecs_desired
  task_env      = var.ecs_task_env
  # もしnetworkの出力が必要なら、親で受けて子へ渡す
  subnets       = module.network.public_subnet_ids
  vpc_id        = module.network.vpc_id
}
```

✅ ポイント
- Terraformには「自動的な変数のバケツリレー」はありません。親で変数を定義し直して、子に明示的に渡すのが基本です。
- 子の output が必要なら、親でいったん受けて、別の子へ渡します（上記の subnets, vpc_id など）。




パターンB：ラッパーで“名前空間オブジェクト”を受け取り、子へ分配（スッキリしやすい）



子モジュール群の入力をオブジェクトで名前空間ごとにまとめて受け取り、親で展開して子へ渡す方法です。環境別 tfvars にも乗せやすく、拡張性が高いです。



1) root（呼び出し側）


```
# live/dev/main.tf
module "stack" {
  source = "../../modules/stack"

  network = {
    cidr              = "10.1.0.0/16"
    public_subnet_cidrs = ["10.1.1.0/24", "10.1.2.0/24"]
  }

  ecs = {
    name          = "web"
    desired_count = 2
    task_env      = { ENV = "dev", LOG_LEVEL = "info" }
  }
}
```

もちろん .tfvars にもそのまま書けます:
```
# dev.tfvars
network = { cidr = "10.1.0.0/16", public_subnet_cidrs = ["10.1.1.0/24","10.1.2.0/24"] }
ecs     = { name = "web", desired_count = 2, task_env = { ENV = "dev" } }
```




2) 親/ラッパー（名前空間オブジェクトを定義し、子へマッピング）



Terraform 1.3+ なら optional属性 が使えるので、キーを任意化しつつデフォルトも与えられます。
```
# modules/stack/variables.tf
variable "network" {
  type = object({
    cidr               = optional(string, "10.0.0.0/16")
    public_subnet_cidrs= optional(list(string), ["10.0.1.0/24","10.0.2.0/24"])
  })
  default = {}
}

variable "ecs" {
  type = object({
    name          = optional(string, "app")
    desired_count = optional(number, 1)
    task_env      = optional(map(string), {})
  })
  default = {}
}
```

```
# modules/stack/main.tf
module "network" {
  source              = "../network"
  vpc_cidr            = var.network.cidr
  public_subnet_cidrs = var.network.public_subnet_cidrs
}

module "ecs" {
  source        = "../ecs"
  name          = var.ecs.name
  desired_count = var.ecs.desired_count
  task_env      = var.ecs.task_env
  vpc_id        = module.network.vpc_id
  subnets       = module.network.public_subnet_ids
}
```

✅ メリット
- ルート側は network { ... }, ecs { ... } のまとまりで記述でき、環境差分管理が楽。
- 後から子モジュールの入力が増えても、同じ名前空間オブジェクト内で拡張しやすい。
- optional属性で必須/任意の切り分けとデフォルトを親で集中管理できる。


🔁 Terraform 1.2以前でも使いたい場合optionalが使えないので、any/map(any)＋merge()で疑似optionalにします：
```
variable "ecs" { type = map(any) default = {} }

locals {
  ecs_defaults = { name = "app", desired_count = 1, task_env = {} }
  ecs_effective = merge(local.ecs_defaults, var.ecs)
}

module "ecs" {
  source        = "../ecs"
  name          = local.ecs_effective.name
  desired_count = local.ecs_effective.desired_count
  task_env      = local.ecs_effective.task_env
}
```


---



よくある質問（実務TIPS）



- Q: 子モジュールの変数を“そのまま全部”渡すワイルドカード的な書き方はある？
  A: ありません。 Terraformは明示的にマッピングします（= 可読性・安全性のため）。
- Q: 子のoutputを別の子に渡す順序制御は？
  module.b.output_x を参照している限り、Terraformは依存関係を解釈します。必要に応じて depends_on を親の module ブロックに追加可能。
- Q: environmentsごとに差し替えたい
  *.tfvars で network { ... }, ecs { ... } を分けるのが扱いやすいです。Terragruntの場合は inputs に同名オブジェクトを定義して親へ渡すのが定石。