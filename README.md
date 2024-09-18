# terraform-cloud-init-enhanced
Mejora de cloud-init con usuario dinámico y contraseña cifrada

# Configuración Cloud-Init Mejorada

Este repositorio se basa en [terraform-ubuntu-cloudimage-vsphere-deploy](https://github.com/vrd83/terraform-ubuntu-cloudimage-vsphere-deploy), originalmente creado por Vaughan Delport.

## Cambios realizados:
- Puedes ingresar el nombre del usuario directamente o usar una variable para definirlo de forma dinámica en distintos entornos.
- **Se añadió una contraseña cifrada utilizando el método SHA-512**, en conjunto con `lock_passwd: false`, lo cual es **obligatorio** para que la configuración funcione correctamente. Si la contraseña no está cifrada, el proceso fallará.
- Se mantuvo la configuración original de la red y la desactivación de IPv6.
- Si se utiliza el archivo tal como está, es probable que las personas que intenten conectarse por SSH se encuentren con un error similar a este:

```bash
Permission denied (publickey).
```
Este error ocurre porque:

No se ha configurado el archivo sshd_config correctamente: No se asegura que la autenticación por clave pública esté habilitada.
Permisos incorrectos en el directorio .ssh o el archivo authorized_keys: Si los permisos del directorio .ssh o del archivo authorized_keys no son los correctos, SSH rechazará la clave.

- Se agrega un nuevo archivo **cloud-config_new** donde se solucina el problema del acceso por ssh

## Compatibilidad con versiones de Terraform vsphere provider

La configuración ha sido probada con la siguiente versión del proveedor **vsphere** de Terraform:

```hcl
source  = "hashicorp/vsphere"
version = "2.9.0"

```

Sin embargo, también es compatible con la versión mencionada en el repositorio original:

```hcl
source  = "terraform-providers/vsphere"
version = "~> 1.11.0"

```

## Importante: Uso de contraseña cifrada

Para que el archivo cloud-init funcione correctamente, la contraseña debe estar cifrada utilizando el método SHA-512. Además, el campo lock_passwd debe estar configurado como false. Si se omite este paso, la configuración no permitirá el acceso con la contraseña especificada.

Cómo generar la contraseña cifrada:
1- Instala la herramienta whois (que incluye el comando mkpasswd):

```bash
 sudo apt install whois
```

2- Luego, genera el hash de tu contraseña:

```bash
echo "tu_clave" | mkpasswd -m sha-512 -s
```
3- Copia el resultado que te devuelve el comando y úsalo en el archivo cloud-init como el valor de la variable ${password} o directamente en el campo passwd.

Ejemplo:
Si tu contraseña es mypassword, el comando te devolverá algo como esto:

```bash
$6$abc123$ABCDEFGHIJKLmnopqrstuvwxyzABCDEFGHIJKLMNOPQRSTUVWXYZ0123456789abcdefghijklmnopqrstuv
```
Copia este hash y pégalo en el archivo cloud-config:

```hcl
users:
  - name: ${username}  # O si prefieres ingresar el nombre directamente
    lock_passwd: false
    passwd: $6$abc123$ABCDEFGHIJKLmnopqrstuvwxyzABCDEFGHIJKLMNOPQRSTUVWXYZ0123456789abcdefghijklmnopqrstuv
```
De esta manera, el usuario podrá iniciar sesión usando la contraseña cifrada, y puedes personalizar el nombre del usuario con una variable o ingresarlo directamente.


### Explicación:
- **Resaltando la obligatoriedad del cifrado**: La contraseña debe estar cifrada con SHA-512 y `lock_passwd` debe ser `false` para que el archivo funcione correctamente.


## Licencia

Este proyecto está licenciado bajo la [Licencia MIT](LICENSE). 

El archivo de licencia del repositorio original se encuentra en [este enlace](https://github.com/vrd83/terraform-ubuntu-cloudimage-vsphere-deploy/blob/main/LICENSE). Asegúrate de consultar el archivo de licencia original para obtener más detalles sobre los términos aplicables al código original.

### Nota

Las nuevas mejoras y modificaciones realizadas en este repositorio están sujetas a la Licencia MIT, mientras que el código original sigue bajo la licencia especificada en el repositorio original.