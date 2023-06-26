# Collections de roles Ansible

## Dependencias
Para utilizar estos playbooks en necesario instalar Ansible y una serie de dependencias

**Instalacion de Ansible**

https://docs.ansible.com/ansible/latest/installation_guide/intro_installation.html

**Instalacion de dependencias de roles**
Es necesaria la instalación de algunas collections para poder ejecutar las tareas, para ello ejecutar el comando siguiente desde el el path del proyecto ansible.
```
ansible-galaxy install -r requeriments.yaml
```

## Cómo corremos un playbook
Para correr el playbook del servidor osadmin en modo test o dry-run

```
ansible-playbook playbooks/os-admin.yaml -C
```

## Definiendo secretos
Vault creará un archivo en el path especificado, lo cifrará y nos va a permitir ingresar variables, texto e incluso cifrar archivos existentes como certificados o archivos comprimidos. Antes de poder realizar el cifrado, es necesario crear un archivo llamado _vaultpass_ en el raiz del proyecto y éste debe contener la clave para cifrar todos los secretos.


**Crear secreto**

Creamos el secreto y automaticamente nos abrira un editor de texto dónde podemos especificar un texto o incluso definir una variable. Como buena práctica, es recomendable que todas la variables que se definan dentro de un vault comiencen con el prefijo vault_ de esta manera sabremos que esa variable es un secreto.

```
ansible-vault create secrets/os-admin.yaml
```

**Visualizar/editar secreto existente**

```
ansible-vault edit secrets/os-admin.yaml
ansible-vault view secrets/os-admin.yaml
```

**Cifrar archivo existente**

```
ansible-vault encrypt miArchivo.rar
```

**Utilizar secretos**
Para utilizar los secretos creados anteriormente, es necesario agregar en la configuración del playbook lo siguiente:


```
  vars_files:
    - ../vault/os-admin.yaml
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

