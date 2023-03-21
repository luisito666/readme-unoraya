# IaC Fundaciíon Bolivar Wordpress

Este repositorio crea la infraestructura necesaria para el despliegue de la aplicación mediante terraform.

# Preparación de ambiente para despliegue - Amazon Provider

1. Crear un bucket con versionamiento y encriptación en la infraestructura de AWS que permita soportar los ficheros ".tfsate" de terraform.
2. Añadir nombre de bucket en fichero remote_state.tf. (bucket  = "nombre_bucket")
3. Crear usuario de IAM y credenciales de AWS. "Access key - Programmatic access - Enabled".
4. Setear AccessKey y SecretAccessKey en ficheros "config", "credentials" del directorio ~/.aws/ desde donde se realizara el despligue.
5. Validar datos para despliegue centralizado en fichero terraform.tfvars (Nombre Proyecto, Account, Region)....

# Deployment de IaC

1. Command: terraform init
2. Command: terraform apply --auto-approve --target module.network
3. Añadir CNAME Records al proveedor DNS (CPANEL/GoDaddy...). El CNAME se extrae del servicio.
   AWS Certificate Manager/Certificates/*.sudominio/Domains (1). Formato Name/Value -> Cambia dependiendo del deploy
4. Command: terraform apply --auto-approve --target module.ec2
5. Command: terraform apply --auto-approve --target module.database
6. Command: terraform apply --auto-approve --target module.backend
7. Command: terraform apply --auto-approve --target module.frontend
8. Crear CNAME para apuntar registro DNS a CNAME del ALB -> Cambia dependiendo del deploy
9. Command: terraform output:
```
Cloudfront_S3 = "d11jcxqxxxxx.cloudfront.net"
DB_Endpoint = "cluster-aurora-mysql.cluster-cpg28xxxxx.us-east-1.rds.amazonaws.com"
DNS_Alb = "alb-production-backend-iac-285xxxx.us-east-1.elb.amazonaws.com"
repository_url = "761265xxxx.dkr.ecr.us-east-1.amazonaws.com/fundacionbolivar"
```

# Conexión EC2 - Bastión

1. aws ec2 describe-instances --filters "Name=tag:Name,Values=ec2-bastion" \
   --query 'Reservations[*].Instances[*].{Instance:InstanceId,Name:Tags[?Key==`Name`]|[0].Value}' \
   --output text
2. aws ssm start-session --target i-0078a11e6d98b64ee -> Cambia dependiendo del deploy


