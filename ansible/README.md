# Conjuto de roles y playbooks de Ansible
Las configuraciones mediante roles son aplicadas con archivos .yaml ubicados en la carpeta **servers** con el nombre de cada servidor. Si lo que deseamos es aplicar un playbook, podemos encontrarlos en **playbooks**

Para correr un playbook instalamos ansible modificamos el archivo inventory agregando/quitando equipos y ademas agregamos parametros de conexion en .ssh/config
Lo ideal, es trabajar con una clave privada ssh que el host destino ya debe tener habilitada en el usuario que vayamos a utilizar para conectarnos y que éste sea miembro de sudo ya que en muchos casos vamos a necesitar permisos para realizar muchas de las tareas que existen en los playbooks.

## Cómo corremos un playbook
Para correr un playbook de un equipo agregado en inventario con usuario matias, solicite clave de sudo y conexion ssh 

```
ansible-playbook playbooks/base_configs.yaml --limit server1 -Kkv -u matias
```

Tambien es posible correr el playbook sin agregar el equipo en el archivo inventory

```
ansible-playbook playbooks/base_configs.yaml -i 192.168.0.10, -Kkv -u matias
```

## Cómo corremos un conjunto de roles ya definidos para un servidor
Cada servidor posee un archivo yaml en la carpeta *servers* con el nombre del mismo, en el cual se definen que roles se le aplican a ese equipo. En el caso de necesitar actualizar el estado de esos roles o modificarlos. En este caso se corren todos los roles asignados a *servidor1*, definiendo variables de entorno para la contraseña de sudo y otra para especificar el entorno para seleccionar configuraciones que se le aplican.


```
ansible-playbook playbooks/server1.yml -e "@vault/logistica.yaml" -e "logistica_environment=prod" -e "ansible_sudo_pass=$MY_PASS"
```
El parametro @vault/logistica.yaml permite cargar los secretos definidios en ese archivo y aplicarlos a la ejecucion del playbook y el parametro ansible_sudo_pass=$MY_PASS asigna el valor de password de sudo desde la variable de entorno $MY_PASS la cual fue definida desde bash antes de ejecutar el playbook.

## Definiendo secretos
Vault creará un archivo en el path especificado, lo cifrará y nos va a permitir ingresar variables, texto e incluso cifrar archivos existentes como certificados o archivos comprimidos


**Crear secreto**

Creamos el secreto y automaticamente nos abrira un editor de texto dónde podemos especificar un texto o incluso definir una variable. Como buena práctica, es recomendable que todas la variables que se definan dentro de un vault comiencen con el prefijo vault_ de esta manera sabremos que esa variable es un secreto.

```
ansible-vault create secrets/logistica.yaml
```

**Visualizar/editar secreto existente**

```
ansible-vault edit secrets/logistica.yaml
ansible-vault view secrets/logistica.yaml
```

**Cifrar archivo existente**

```
ansible-vault encrypt miArchivo.rar
```

**Utilizar secretos**

```
ansible-playbook -i inventory -e "@secrets/logistica.yaml" -e "logistica_environment=[prod|stage|both]" server1.yml -e "ansible_sudo_pass=$MY_PASS" --vault-password-file=vaultpass
```
## Buscar info de modulos y obtener ayuda

```
ansible-doc -l |grep -i user
ansible-doc user
```

## Comandos útiles

**Testea conexión a miHost**

```
ansible miHost -m ping
```

**Corre el comando en miHost**

```
ansible miHost -a 'uname -r'
```

**Copiar archivo a miHost o insertar contenido en el mismo**

```
ansible miHost -m copy -a 'src=/etc/mi-motd dest=/etc/motd'
ansible miHost -m copy -a 'content="Managed by Ansible\n" dest=/etc/motd'
```

**Instala paquete en miHost**

```
ansible miHost -m apt -a 'name=squid state=latest' -b
```

**Reinicia apache en miHost**

```
ansible miHost -m service -a 'name=apache2 state=restarted' -b
```

**Verifica síntaxis de playbook**
```
ansible-playbook --syntax-check apache.yml
```

## Crear estructura de carpetas para un nuevo rol

```
ansible-galaxy role init test_ansible
```

## Instalar modulos en el proyecto

```
ansible-galaxy collection install community.docker -p collections
```
## Redireccion de stdout
Cuando la salida de los cambios es muy larga y neceitemos volcar eso a una archivo para poder verlo completo


```
export PYTHONUNBUFFERED=1

ansible-playbook playbooks/server.yml -e '@vault/logistica.yaml' -K 2>&1 | tee playbook_run.txt
less playbook_run.txt
```

## Para habilitar logs de stados en disco
Editar /etc/ansible/ansible.cfg y agregar
callback_whitelist = log_plays

Por default los logs se vuelcan en _/var/log/ansible/hosts_

El path de logs puede cambiarse seteando:

_export ANSIBLE_LOG_FOLDER=_

