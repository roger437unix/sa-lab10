         ___        ______     ____ _                 _  ___  
        / \ \      / / ___|   / ___| | ___  _   _  __| |/ _ \ 
       / _ \ \ /\ / /\___ \  | |   | |/ _ \| | | |/ _` | (_) |
      / ___ \ V  V /  ___) | | |___| | (_) | |_| | (_| |\__, |
     /_/   \_\_/\_/  |____/   \____|_|\___/ \__,_|\__,_|  /_/ 
 ----------------------------------------------------------------- 


### sa-lab10


Respostas às perguntas:

C
B
A
D
A
C

------------------------------------------------------------------------------



touch S3.yaml

=> Editar o arquivo

AWSTemplateFormatVersion: "2010-09-09"
Description: "cafe S3 template"

Resources:
  S3Bucket:
    Type: AWS::S3::Bucket


=> No terminal do Bash, execute estas duas linhas de código:

aws configure get region
aws cloudformation create-stack --stack-name CreateBucket --template-body file://S3.yaml

{
    "StackId": "arn:aws:cloudformation:us-east-1:811793693373:stack/CreateBucket/53955320-b269-11ef-bd8a-0e7c9793a759"
}


=> 17.

File: S3.yaml

AWSTemplateFormatVersion: "2010-09-09"
Description: "cafe S3 template"

Resources:
  S3Bucket:
    Type: AWS::S3::Bucket
    # Anexe uma política de exclusão que reterá o bucket.
    DeletionPolicy: Retain
   
    Properties:
      # Configure o bucket para hospedar um site estático com index.html definido como o documento de índice
      WebsiteConfiguration:
        IndexDocument: index.html
        ErrorDocument: error.html
       
# Para o modelo do AWS CloudFormation, adicione uma saída que forneça o URL do site.      
Outputs:
  WebsiteURL:
    Value: !GetAtt
      - S3Bucket
      - WebsiteURL
    Description: URL for website hosted on S3


=> 20.

cd ../
aws cloudformation validate-template --template-body file://S3.yaml

{
    "Parameters": [],
    "Description": "cafe S3 template"
}


=> 21.

aws cloudformation update-stack --stack-name CreateBucket --template-body file://S3.yaml
{
    "StackId": "arn:aws:cloudformation:us-east-1:811793693373:stack/CreateBucket/53955320-b269-11ef-bd8a-0e7c9793a759"
}




-----------------------------------------------------------------------------



# cafe-app.yaml


AWSTemplateFormatVersion: 2010-09-09
Description: Cafe application

Parameters:

  LatestAmiId:
    Type: 'AWS::SSM::Parameter::Value<AWS::EC2::Image::Id>'
    Default: '/aws/service/ami-amazon-linux-latest/amzn2-ami-hvm-x86_64-gp2'

  CafeNetworkParameter:
    Type: String
    Default: update-cafe-network
    
  IntancesTypeParameters:
    Type: String
    Default: t2.small
    AllowedValues:
      - t2.micro
      - t2.small
      - t3.micro
      - t3.small

Mappings:

  RegionMap:
    us-east-1:
      "keypair": "vockey"
    us-west-2:
      "keypair": "cafe-oregon"

Resources:

  CafeInstance:
    Type: 'AWS::EC2::Instance'
    Properties:
      ImageId: !Ref LatestAmiId
      InstanceType: !Ref IntancesTypeParameters
      KeyName: !FindInMap [RegionMap, !Ref "AWS::Region", keypair]
      IamInstanceProfile: CafeRole
      NetworkInterfaces:
        - DeviceIndex: '0'
          AssociatePublicIpAddress: 'true'
          SubnetId: !ImportValue
            'Fn::Sub': '${CafeNetworkParameter}-SubnetID'
          GroupSet:
            - !Ref CafeSG
      Tags:
        - Key: Name
          Value: Cafe Web Server
      UserData:
        Fn::Base64:
          !Sub |
            #!/bin/bash
            yum -y update
            yum install -y httpd mariadb-server wget
            amazon-linux-extras install -y lamp-mariadb10.2-php7.2 php7.2
            systemctl enable httpd
            systemctl start httpd
            systemctl enable mariadb
            systemctl start mariadb
            wget https://aws-tc-largeobjects.s3.us-west-2.amazonaws.com/CUR-TF-200-ACACAD-2-91555/14-lab-mod10-challenge-CFn/s3/cafe-app.sh
            chmod +x cafe-app.sh
            ./cafe-app.sh

  CafeSG:
    Type: 'AWS::EC2::SecurityGroup'
    Properties:
      GroupDescription: Enable SSH, HTTP access
      VpcId: !ImportValue
        'Fn::Sub': '${CafeNetworkParameter}-VpcID'
      Tags:
        - Key: Name
          Value: CafeSG
      SecurityGroupIngress:
        - IpProtocol: tcp
          FromPort: '80'
          ToPort: '80'
          CidrIp: 0.0.0.0/0
        - IpProtocol: tcp
          FromPort: '22'
          ToPort: '22'
          CidrIp: 0.0.0.0/0

Outputs:
  WebServerPublicIP:
    Value: !GetAtt 'CafeInstance.PublicIp'



------------------------------------------------------------------------------


https://github.com/baselm/ACA-LAB10/tree/main/Lab-solution


------------------------------------------------------------------------------



