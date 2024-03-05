#Créer VPC:#
aws ec2 create-vpc --cidr-block 10.0.0.0/16 --query Vpc.VpcId --output text
aws ec2 create-tags --resources vpc-0fb3f11ec680d7e02 --tags Key=Name,Value=VPC_Djame_EVAL

Créer Subnet
aws ec2 create-subnet --vpc-id vpc-0fb3f11ec680d7e02 --cidr-block 10.0.1.0/24 --availability-zone us-east-2a --query Subnet.SubnetId --output text
aws ec2 create-tags --resources subnet-06e26c05a5952cfdd --tags Key=Name,Value=Public_Subnet_eval
aws ec2 create-subnet --vpc-id vpc-0fb3f11ec680d7e02 --cidr-block 10.0.2.0/24 --availability-zone us-east-2a --query Subnet.SubnetId --output text
aws ec2 create-tags --resources subnet-0260b50f6482917aa --tags Key=Name,Value=Private_Subnet

Créer et Attacher au VPC une Passerelle Internet
aws ec2 create-internet-gateway --query InternetGateway.InternetGatewayId --output text
aws ec2 create-tags --resources igw-0e23c6436d9fb355e --tags Key=Name,Value=djam-gateway-cli
aws ec2 attach-internet-gateway --vpc-id vpc-0fb3f11ec680d7e02 --internet-gateway-id igw-0e23c6436d9fb355e

Créer et Configurer une Table de Routage pour la Passerelle Internet et la Passerelle NAT
aws ec2 create-route-table --vpc-id vpc-0fb3f11ec680d7e02 --query RouteTable.RouteTableId --output text
aws ec2 create-tags --resources rtb-070164651c2ce0384 --tags Key=Name,Value=Routable_djam

aws ec2 create-route-table --vpc-id vpc-0fb3f11ec680d7e02 --query RouteTable.RouteTableId --output text
aws ec2 create-tags --resources rtb-04db62a5adc10727e --tags Key=Name,Value=Routable_djam_nat

aws ec2 create-route --route-table-id rtb-070164651c2ce0384 --destination-cidr-block 0.0.0.0/0 --gateway-id igw-0e23c6436d9fb355e
aws ec2 create-route --route-table-id rtb-04db62a5adc10727e--destination-cidr-block 0.0.0.0/0 --nat-gateway-id nat-08676ff584c3a39a6

Créer une Passerelle NAT et une Adresse IP Élastic
aws ec2 allocate-address --domain vpc --query 'AllocationId' --output text --region us-east-2a --tag-specifications 'ResourceType=elastic-ip,Tags=[{Key=Name,Value=IPElastic_Djam}]'
aws ec2 create-nat-gateway --subnet-id subnet-0260b50f6482917aa --allocation-id eipalloc-04790fbec2feb

Associate Routage table to Subnet:
aws ec2 associate-route-table --route-table-id rtb-070164651c2ce0384 --subnet-id subnet-06e26c05a5952cfdd
aws ec2 associate-route-table --route-table-id rtb-04db62a5adc10727e --subnet-id subnet-0260b50f6482917aa

Créer des Groupes de Sécurité
aws ec2 create-security-group --group-name SG-Eval --description "Groupe de securite cli" --vpc-id vpc-0fb3f11ec680d7e02

Créer des Instances EC2
Public:
aws ec2 run-instances --image-id ami-0ec3d9efceafb89e0 --count 1 --instance-type t2.micro --key-name VM_Docker_AWS_1 --security-group-ids sg-016ad7ba11da09165 --subnet-id subnet-06e26c05a5952cfdd --associate-public-ip-address --tag-specifications 'ResourceType=instance,Tags=[{Key=DebianServer,Value=Beta}]' --region us-east-2

Private:
aws ec2 run-instances --image-id ami-0ec3d9efceafb89e0 --count 1 --instance-type t2.micro --key-name VM_Docker_AWS_1 --security-group-ids sg-016ad7ba11da09165 --subnet-id subnet-0260b50f6482917aa --tag-specifications 'ResourceType=instance,Tags=[{Key=DebianServer,Value=Beta}]' --region us-east-2
