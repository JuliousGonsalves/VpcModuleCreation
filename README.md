### VpcModuleCreation

[![Builds](https://travis-ci.org/joemccann/dillinger.svg?branch=master)](https://travis-ci.org/joemccann/dillinger)

## Description.

Here we are going to create a AWS VPC module which consist of Public/Private Subnet and Network Gateway's for the VPC.

Ie, 3 Public subnet, 3 Private subnet , 1 NAT Gateways, 1 Internet Gateway, and 2 Route Tables.

### prerequisite for this project

-Iam role with with attached policies for the creation of VPC. (By using role we can avoid adding access keys and secretkey in terraform files)

-Basic knowledge in AWS services such as VPC and IP subnetting


### Used Languages

Terraform - IAC Tool

#-----------------------------------
#### Variable declartion -  variables.tf
#-----------------------------------
```sh
variable "vpc_cidr" {

  default = "172.16.0.0/16"

}

variable "project" {

  default = "example"

}


variable "env" {

  default = "test"

}
```
#----------------------------------------------
### Datasource Declaration (AZ) - datasource.tf
#-----------------------------------------------
```sh
data "aws_availability_zones" "az" {
  state = "available"
}
```

#### Creating VPC file- main.tf  

# -------------------------------------------------------------------
### Vpc Creation  
# -------------------------------------------------------------------
```sh
resource "aws_vpc" "vpc" {

  cidr_block       = var.vpc_cidr
  instance_tenancy = "default"
  enable_dns_support = true
  enable_dns_hostnames = true
  tags = {
    Name = "${var.project}-vpc-${var.env}"
    project = var.project
    environment = var.env
  }

}
```

# -------------------------------------------------------------------
### InterNet GateWay Creation
# -------------------------------------------------------------------
```sh
resource "aws_internet_gateway" "igw" {
  vpc_id = aws_vpc.vpc.id
   tags = {
    Name = "${var.project}-igw-${var.env}"
    project = var.project
     environment = var.env
  }

}


# -------------------------------------------------------------------
### Public Subnet 1
# -------------------------------------------------------------------
```sh

resource "aws_subnet" "public1" {

  vpc_id     = aws_vpc.vpc.id
  cidr_block = cidrsubnet(var.vpc_cidr, "3", 0)
  map_public_ip_on_launch = true
  availability_zone = data.aws_availability_zones.az.names[0]
  tags = {
    Name = "${var.project}-public1-${var.env}"
    project = var.project
     environment = var.env
  }
}

# -------------------------------------------------------------------
### Public Subnet 2
# -------------------------------------------------------------------

resource "aws_subnet" "public2" {

  vpc_id     = aws_vpc.vpc.id
  cidr_block = cidrsubnet(var.vpc_cidr, "3", 1)
  map_public_ip_on_launch = true
  availability_zone = data.aws_availability_zones.az.names[1]
  tags = {
    Name = "${var.project}-public2-${var.env}"
    project = var.project
     environment = var.env
  }
}

# -------------------------------------------------------------------
### Public Subnet 3
# -------------------------------------------------------------------
resource "aws_subnet" "public3" {

  vpc_id     = aws_vpc.vpc.id
  cidr_block = cidrsubnet(var.vpc_cidr, "3", 2)
  map_public_ip_on_launch = true
  availability_zone = data.aws_availability_zones.az.names[2]
  tags = {
    Name = "${var.project}-public3-${var.env}"
    project = var.project
     environment = var.env
  }
}

# -------------------------------------------------------------------
### Private Subnet 1
# -------------------------------------------------------------------
resource "aws_subnet" "private1" {

  vpc_id     = aws_vpc.vpc.id
  cidr_block = cidrsubnet(var.vpc_cidr, "3", 3)
  map_public_ip_on_launch = false
  availability_zone = data.aws_availability_zones.az.names[0]
  tags = {
    Name = "${var.project}-private1-${var.env}"
    project = var.project
     environment = var.env
  }
}

# -------------------------------------------------------------------
### Private Subnet 2
# -------------------------------------------------------------------
resource "aws_subnet" "private2" {

  vpc_id     = aws_vpc.vpc.id
  cidr_block = cidrsubnet(var.vpc_cidr, "3", 4)
  map_public_ip_on_launch = false
  availability_zone = data.aws_availability_zones.az.names[1]
  tags = {
    Name = "${var.project}-private2-${var.env}"
    project = var.project
     environment = var.env
  }
}

# -------------------------------------------------------------------
### Private Subnet 3
# -------------------------------------------------------------------
resource "aws_subnet" "private3" {

  vpc_id     = aws_vpc.vpc.id
  cidr_block = cidrsubnet(var.vpc_cidr, "3", 5)
  map_public_ip_on_launch = false
  availability_zone = data.aws_availability_zones.az.names[2]
  tags = {
    Name = "${var.project}-private3-${var.env}"
    project = var.project
     environment = var.env
  }
}
```

# -------------------------------------------------------------------
### ElasticIp for NatGateway
# -------------------------------------------------------------------
```sh

resource "aws_eip" "nat" {
  vpc      = true
  tags = {
    Name = "${var.project}-nat-${var.env}"
    project = var.project
     environment = var.env
  }
}
```
# -------------------------------------------------------------------
###  NatGateway  Creation
# -------------------------------------------------------------------
```sh
resource "aws_nat_gateway" "nat" {

  allocation_id = aws_eip.nat.id
  subnet_id     = aws_subnet.public1.id
  tags = {
    Name = "${var.project}-nat-${var.env}"
    project = var.project
     environment = var.env
  }
  depends_on = [aws_internet_gateway.igw]
}
```

# -------------------------------------------------------------------
###  Public RouteTable
# -------------------------------------------------------------------
```sh
resource "aws_route_table" "public" {

  vpc_id = aws_vpc.vpc.id

  route {
    cidr_block = "0.0.0.0/0"
    gateway_id = aws_internet_gateway.igw.id
  }

  tags = {
    Name = "${var.project}-public-${var.env}"
    project = var.project
     environment = var.env
  }
}
```
# -------------------------------------------------------------------
###  Private RouteTable
# -------------------------------------------------------------------
```sh

resource "aws_route_table" "private" {

  vpc_id = aws_vpc.vpc.id

  route {
    cidr_block = "0.0.0.0/0"
    nat_gateway_id = aws_nat_gateway.nat.id
  }

  tags = {
    Name = "${var.project}-private-${var.env}"
    project = var.project
    environment = var.env
  }
}
```
# -------------------------------------------------------------------
###  Public RouteTable association
# -------------------------------------------------------------------
```sh

resource "aws_route_table_association" "public1" {
  subnet_id      = aws_subnet.public1.id
  route_table_id = aws_route_table.public.id
}

resource "aws_route_table_association" "public2" {
  subnet_id      = aws_subnet.public2.id
  route_table_id = aws_route_table.public.id
}

resource "aws_route_table_association" "public3" {
  subnet_id      = aws_subnet.public3.id
  route_table_id = aws_route_table.public.id
}
```

# -------------------------------------------------------------------
###  Private RouteTable association
# -------------------------------------------------------------------
```sh

resource "aws_route_table_association" "private1" {
  subnet_id      = aws_subnet.private1.id
  route_table_id = aws_route_table.private.id
}

resource "aws_route_table_association" "private2" {
  subnet_id      = aws_subnet.private2.id
  route_table_id = aws_route_table.private.id
}

resource "aws_route_table_association" "private3" {
  subnet_id      = aws_subnet.private3.id
  route_table_id = aws_route_table.private.id
}
```
#--------------------------------------
### output.tf - to get terraform results
#--------------------------------------
```sh
output "vpc_id" {
  value = aws_vpc.vpc.id
}

output "subnet_public1_id" {
  value = aws_subnet.public1.id
}

output "subnet_public2_id" {
  value = aws_subnet.public2.id
}

output "subnet_public3_id" {
  value = aws_subnet.public3.id
}


output "subnet_private1_id" {
  value = aws_subnet.private1.id
}

output "subnet_private2_id" {
  value = aws_subnet.private2.id
}

output "subnet_private3_id" {
  value = aws_subnet.private3.id
}
```



 


