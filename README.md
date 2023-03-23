# IaC Fundaciíon Bolivar Wordpress

Este repositorio crea la infraestructura necesaria para el despliegue de la aplicación mediante terraform.

# Preparacion del Docker image.

1. Se debe dejar el docker image listo y construido, para que despues que se cree el registro en la infra, se pueda hacer push de la imagen de docker.
2. La imagen de docker se crea a partir del repositorio de codigo

´´´
docker build -t nombre/imagen .
´´´

en el comando anterior el punto indica el contexto de ejecucion de la imagen de docker.

3. despues de eso se crea el tag, pero como el repo aun no se ha creado, esta tarea se hace despues de que este lista la infra estructura.

´´´
docker tag nombre/image repositorio_remoto_aws
´´´

# Preparación de ambiente para despliegue - Amazon Provider

1. Crear un bucket con versionamiento y encriptación en la infraestructura de AWS que permita soportar los ficheros ".tfsate" de terraform.
2. Añadir nombre de bucket en fichero remote_state.tf. (bucket  = "nombre_bucket")
3. Crear usuario de IAM y credenciales de AWS. "Access key - Programmatic access - Enabled".
4. Setear AccessKey y SecretAccessKey en ficheros "config", "credentials" del directorio ~/.aws/ desde donde se realizara el despligue.
5. Validar datos para despliegue centralizado en fichero terraform.tfvars (Nombre Proyecto, Account, Region)....

# Deployment de IaC

1. Inicializamos el terraform
```
terraform init
```
2. El primer modulo que se despliega es el de red ya que de este depende toda la infra
```
terraform apply --auto-approve --target module.network
```
3. Añadir CNAME Records al proveedor DNS (CPANEL/GoDaddy...). El CNAME se extrae del servicio.
   AWS Certificate Manager/Certificates/*.sudominio/Domains (1). Formato Name/Value -> Cambia dependiendo del deploy

4. Se ejecuta el modulo de ec2, este se encarga de crear el bastion host
```
terraform apply --auto-approve --target module.ec2
```
5. Ahora desplegamos la base de datos
```
terraform apply --auto-approve --target module.database
```
6. Ahora bamos a desplegar el ecs y los micro servicios
```
terraform apply --auto-approve --target module.backend
```
7. Ahora vamos a desplegar el fronted, este se encargara de crear un cdn
```
terraform apply --auto-approve --target module.frontend
```
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

# Sincronización de la base de datos.

Ingresar al bastion host. Inicia una sesión en PostgreSQL con el usuario y la base de datos donde deseas importar el archivo:

```
psql -U myuser -d mydatabase
```

Una vez que estés en la sesión de PostgreSQL, ejecuta el comando \i seguido del nombre del archivo de copia de seguridad que deseas importar. Por ejemplo:

Asegúrate de estar en el directorio donde está el archivo .sql

```
\i backup.sql
```

Esto ejecutará el archivo de copia de seguridad y restaurará los datos en la base de datos especificada.

# Sincronizar buckets de S3

1. una vez tengamos el backup de las imagenes listas, lo descomprimimos e ingresamos al directorio
```
tar -xvf bk_images.tar.gz
cd bk_images
```
2. ahora ejecutamos el comando para que se realice la sincronizacion 

```
aws s3 sync . s3://nombre_del_bucket
```

En resumen, este texto explica los pasos necesarios para desplegar una aplicación WordPress en AWS mediante el uso de Terraform. También se incluyen instrucciones para la sincronización de los buckets de S3 y la importación de un backup de PostgreSQL.

