# Apuntes sobre el curso DO322 de Red Hat

### Personalización de las instalaciones sin conexión

Después de copiar el registro local de contenedores, inicie el proceso de instalación de OpenShift. Debe modificar el archivo install-config.yaml para personalizar la instalación sin conexión de OpenShift.

 **El siguiente es un archivo install-config.yaml de muestra que se usa en una instalación con conexión:**

```
[user@demo ~]$ cat ${HOME}/ocp4-cluster/install-config.yaml
apiVersion: v1
baseDomain: example.com
#proxy: 1
#  httpProxy: http://<username>:<pswd>@<ip>:<port>
#  httpsProxy: http://<username>:<pswd>@<ip>:<port>
#  noProxy: example.com
compute:
- hyperthreading: Enabled
  name: worker
  replicas: 2
controlPlane:
  hyperthreading: Enabled
  name: master
  replicas: 3
metadata:
  name: ocp4
networking:
  clusterNetwork:
  - cidr: 10.128.0.0/14
    hostPrefix: 23
  networkType: OpenShiftSDN
  serviceNetwork:
  - 172.30.0.0/16
platform:
  none: {}
fips: false
pullSecret: |
  {"auths":...output omitted...} 2
sshKey: |
  ssh-rsa AA...output omitted...
```

1

Si es necesario, puede usar un proxy de red en una instalación con conexión.

2

pull-secret contiene las credenciales para autenticarse con quay.io y registry.redhat.io.

**El siguiente es un archivo install-config.yaml de muestra que se usa en una instalación sin conexión:**

```
[user@demo ~]$ cat ${HOME}/ocp4-cluster/install-config.yaml
apiVersion: v1
baseDomain: example.com
compute:
- hyperthreading: Enabled
  name: worker
  replicas: 2
controlPlane:
  hyperthreading: Enabled
  name: master
  replicas: 3
metadata:
  name: ocp4
networking:
  clusterNetwork:
  - cidr: 10.128.0.0/14
    hostPrefix: 23
  networkType: OpenShiftSDN
  serviceNetwork:
  - 172.30.0.0/16
platform:
  none: {}
fips: false
pullSecret: |
  {"auths":...output omitted...} 1
sshKey: |
  ssh-rsa AA...output omitted...
additionalTrustBundle: | 2
  -----BEGIN CERTIFICATE-----
  ZZZZZZZZZZZZZZZZZZZZZZZZZZZZZZZZZZZZZZZZZZZZZZZZZZZZZZZZZZZZZZZZ
  -----END CERTIFICATE-----
imageContentSources: 3
- mirrors:
  - nexus-registry-int.apps.tools.dev.nextcle.com/openshift/ocp4
  source: quay.io/openshift-release-dev/ocp-release
- mirrors:
  - nexus-registry-int.apps.tools.dev.nextcle.com/openshift/ocp4
  source: quay.io/openshift-release-dev/ocp-v4.0-art-dev
```

1

pull-secret contiene las credenciales para autenticarse con el registro local de contenedores nexus-registry-int.apps.tools.dev.nextcle.com.

2

Si no se confía en el certificado usado por el registro local de contenedores, proporcione el conjunto de certificados que usó para el registro local de contenedores.

3

Agregue la sección imageContentSources de la salida del comando que se usó para copiar el repositorio. Esta personalización configurará el archivo /etc/containers/registries.conf en cada nodo del clúster para habilitar la duplicación de imágenes.

**En la siguiente tabla, se muestran algunas opciones útiles disponibles para el comando openshift-install:**

 | Opción |	Descripción |
|:--------:|:------------:|
| help | Imprime información de ayuda |
| explain -h	| Explica los campos relacionados con cada API InstallConfig soportada |
| explain installconfig	| Explica el recurso installconfig y sus campos |
| --log-level debug	| Habilita el modo de depuración |
| --dir	| Configura la ruta del directorio para almacenar los activos generados |
| create install-config	| Crea el archivo de configuración de instalación install-config.yaml |
| create manifests	| Crea manifiestos de Kubernetes |
| create ignition-configs | Crea archivos de configuración de encendido (ignition) |
| create cluster	| Implementa el clúster |
| wait-for bootstrap-complete	| Espera hasta que finalice la fase de instalación de bootstrap |
| wait-for install-complete	| Espera hasta que finaliza la instalación del clúster |
| destroy cluster	| Elimina el clúster |

### Pasos de alto nivel requeridos para instalar OpenShift:

1. Cumplir con los requisitos previos de la instalación.

2. Crear el directorio de instalación.

3. Crear el archivo de configuración del instalador install-config.yaml.

4. Generar los manifiestos de Kubernetes.

5. Generar los archivos de configuración de encendido (ignition).

6. Implementar el clúster de OpenShift.

7. Verificar el estado del clúster de OpenShift.

**Creación del directorio de instalación**

```
[user@demo ~]$ mkdir ${HOME}/ocp4-cluster
```

**Creación del archivo de configuración del instalador**

Después de crear el directorio de instalación, ejecute el instalador de OpenShift para crear el archivo de configuración del instalador install-config.yaml. El instalador le solicita la información necesaria del clúster y, luego, crea el archivo install-config.yaml correspondiente.

```
[user@demo ~]$ openshift-install create install-config \
> --dir=${HOME}/ocp4-cluster
? SSH Public Key /home/user/.ssh/ocp4-cluster.pub
? Platform aws
INFO Credentials loaded from the "default" profile in file "/home/user/.aws/credentials"
? Region us-east-2
? Base Domain mydomain.com
? Cluster Name ocp4
? Pull Secret [? for help] +++++
INFO Install-Config created in: /home/user/ocp4-cluster
```
El siguiente es un archivo de configuración install-config.yaml:

```
[user@demo ~]$ cat ${HOME}/ocp4-cluster/install-config.yaml
apiVersion: v1
baseDomain: mydomain.com
compute:
- architecture: amd64
  hyperthreading: Enabled
  name: worker
  platform: {}
  replicas: 3
controlPlane:
  architecture: amd64
  hyperthreading: Enabled
  name: master
  platform: {}
  replicas: 3
metadata:
  creationTimestamp: null
  name: ocp4
networking:
  clusterNetwork:
  - cidr: 10.128.0.0/14
    hostPrefix: 23
  machineNetwork:
  - cidr: 10.0.0.0/16
  networkType: OpenShiftSDN
  serviceNetwork:
  - 172.30.0.0/16
platform:
  aws:
    region: us-east-2
publish: External
pullSecret: |
  {"auths":...}
sshKey: |
  ssh-rsa AA...
```

**Generación de manifiestos de Kubernetes**

```
[user@demo ~]$ openshift-install create manifests \
> --dir=${HOME}/ocp4-cluster
INFO Consuming Install Config from target directory
INFO Manifests created in: /home/user/ocp4-cluster/manifests and /home/user/ocp4-cluster/openshift
```

Puede revisar los manifiestos de Kubernetes generados por el instalador de OpenShift en el directorio de instalación.

```
[user@demo ~]$ find ${HOME}/ocp4-cluster/manifests
/home/user/ocp4-cluster/manifests
/home/user/ocp4-cluster/manifests/04-openshift-machine-config-operator.yaml
...output omitted...
```

**Generación de archivos de configuración de encendido (ignition)**

Después de generar los manifiestos de Kubernetes, ejecute el instalador de OpenShift para crear los archivos de configuración de encendido (ignition) a partir del contenido de los manifiestos. El instalador de OpenShift crea los archivos de configuración de encendido (ignition) para los nodos bootstrap, de plano de control y de cómputo. Al implementar el clúster de OpenShift, el instalador de OpenShift usa estos archivos de configuración de encendido (ignition) para instalar y configurar RHCOS en los nodos bootstrap, de plano de control y de cómputo.

```
[user@demo ~]$ openshift-install create ignition-configs \
> --dir=${HOME}/ocp4-cluster
INFO Consuming Master Machines from target directory
INFO Consuming OpenShift Install (Manifests) from target directory
INFO Consuming Openshift Manifests from target directory
INFO Consuming Worker Machines from target directory
INFO Consuming Common Manifests from target directory
INFO Ignition-Configs created in: /home/user/ocp4-cluster and /home/user/ocp4-cluster/auth
```

Puede revisar los archivos de configuración de encendido (ignition) generados por el instalador de OpenShift en el directorio de instalación.

```
[user@demo ~]$ find ${HOME}/ocp4-cluster -name '*.ign' | xargs ls -lrt
-rw-r-----. 1 user user   1732 Dec 27 19:35 /home/user/ocp4-cluster/master.ign
-rw-r-----. 1 user user   1732 Dec 27 19:35 /home/user/ocp4-cluster/worker.ign
-rw-r-----. 1 user user 307594 Dec 27 19:35 /home/user/ocp4-cluster/bootstrap.ign
```

**Implementación del clúster de OpenShift**

Después de generar los archivos de configuración de encendido (ignition), ejecute el instalador de OpenShift para implementar el clúster de OpenShift.

```
[user@demo ~]$ openshift-install create cluster \
> --dir=${HOME}/ocp4-cluster --log-level=debug
DEBUG OpenShift Installer 4.6.4
DEBUG Built from commit 6e02d049701437fa81521fe981405745a62c86c5
...output omitted...
INFO Consuming Bootstrap Ignition Config from target directory
INFO Consuming Worker Ignition Config from target directory
INFO Consuming Master Ignition Config from target directory
...output omitted...
DEBUG Apply complete! Resources: 122 added, 0 changed, 0 destroyed.
DEBUG OpenShift Installer 4.6.4
DEBUG Built from commit 6e02d049701437fa81521fe981405745a62c86c5
INFO Waiting up to 20m0s for the Kubernetes API at https://api.ocp4.example.com:6443...
INFO API v1.19.0+9f84db3 up
INFO Waiting up to 30m0s for bootstrapping to complete...
...output omitted...
DEBUG Bootstrap status: complete
INFO Destroying the bootstrap resources...
...output omitted...
INFO Waiting up to 40m0s for the cluster at https://api.ocp4.example.com:6443 to initialize...
DEBUG Still waiting for the cluster to initialize: Working towards 4.6.4: 82% complete
...output omitted...
INFO Install complete!
INFO To access the cluster as the system:admin user when using 'oc', run 'export KUBECONFIG=/home/user/ocp4-aws-cluster/auth/kubeconfig'
INFO Access the OpenShift web-console here: https://console-openshift-console.apps.ocp4.example.com
INFO Login to the console with user: "kubeadmin", and password: "xxxx"
...output omitted...
INFO Time elapsed: 34m10s
```

En una instalación de infraestructura preexistente, usted ya ha instalado los nodos del clúster y, por tal motivo, no puede usar el comando openshift-install create cluster para implementar el clúster. La instalación del nodo bootstrap desencadena la instalación del clúster; por lo tanto, ejecute la siguiente secuencia de comandos para monitorear la instalación del clúster:

```
[user@demo ~]$ openshift-install wait-for bootstrap-complete \
> --dir=${HOME}/ocp4-cluster --log-level=debug
```

```
[user@demo ~]$ openshift-install wait-for install-complete \
> --dir=${HOME}/ocp4-cluster --log-level=debug
```

**Monitoreo de instalaciones de OpenShift**

Además de revisar los registros de instalación, puede realizar un monitoreo del proceso de instalación activo con el comando oc. Puede comenzar a usar el comando oc una vez que el nodo bootstrap tenga la API de Kubernetes ejecutándose en el plano de control temporal.

Para comenzar a usar el comando oc, autentíquense en la API de Kubernetes con privilegios de cluster-admin mediante uno de los siguientes métodos:

A través de la configuración de la variable de entorno KUBECONFIG para usar el archivo kubeconfig.

```
[user@demo ~]$ export KUBECONFIG=${HOME}/ocp4-cluster/auth/kubeconfig
```

Después de iniciar sesión en la API de Kubernetes como cluster-admin, use la siguiente secuencia de comandos oc para monitorear el proceso de instalación:

```
[user@demo ~]$ watch 'oc get clusterversion; oc get clusteroperators; \
> oc get pods --all-namespaces | grep -v -E "Running|Completed"; oc get nodes'
...output omitted...
```

### Proceso de implementación del clúster de OpenShift

Tres etapas:

1. Etapa de bootstrap (Bootkube)

El proceso de instalación de OpenShift implementa el nodo bootstrap. A continuación, el servicio systemd release-image que se ejecuta en el nodo bootstrap descarga las imágenes de contenedores requeridas para iniciar el plano de control temporal. Por último, el servicio systemd bootkube que se ejecuta en el nodo bootstrap inicia el plano de control temporal.

2. Etapa de bootstrap (plano de control temporal)

La API de Kubernetes y el plano de control temporal se ejecutan en el nodo bootstrap. A continuación, el nodo bootstrap espera hasta que los nodos de plano de control arranquen y formen un clúster de etcd. Finalmente, el nodo bootstrap programa el plano de control de producción según los nodos de plano de control.

3. Etapa de plano de control de producción

Después de que el plano de control de producción se ejecuta en los nodos de plano de control, el operador de versión del clúster (CVO) finaliza la implementación del clúster de OpenShift.

**Solución de problemas en la etapa de bootstrap (Bootkube)**

En esta etapa, la API de Kubernetes todavía no está disponible en el nodo bootstrap. Para solucionar problemas, debe obtener los registros del nodo bootstrap usando SSH. Puede iniciar sesión en el nodo bootstrap usando SSH desde el host bastión e inspeccionar los registros del diario (journal) y del contenedor. Uno de los requisitos previos para instalar OpenShift es generar una clave SSH (clave SSH del clúster) para permitir el acceso SSH desde el host bastión a los nodos del clúster.

No ejecute el comando ssh-keygen en esta etapa, ya que sobrescribirá la clave SSH del clúster. No tendrá acceso a los nodos del clúster con la nueva clave SSH del clúster. Debe ejecutar el comando ssh-keygen antes de ejecutar la instalación de OpenShift.

```
[user@demo ~]$ ssh-keygen -t rsa -b 4096 -N '' -f ${HOME}/.ssh/ocp4-cluster
```

Antes de ejecutar la implementación del clúster, debe incluir la clave SSH del clúster ocp4-cluster.pub en el archivo install-config.yaml. Después de instalar los nodos del clúster, la clave SSH del clúster le dará acceso SSH desde el host bastión. Para solucionar problemas en el nodo bootstrap, ejecute el comando ssh desde el host bastión para conectarse al nodo bootstrap. En el siguiente ejemplo, se asume que 192.168.50.9 es la dirección IP del nodo bootstrap. Use la clave SSH privada del clúster para conectarse al nodo bootstrap desde el host bastión:

```
[user@demo ~]$ ssh -i ${HOME}/.ssh/ocp4-cluster core@192.168.50.9
```

El nodo bootstrap ejecuta dos servicios systemd, el servicio release-image y el servicio bootkube.

El servicio release-image descarga las imágenes de contenedores de la versión de OpenShift que se usan durante la instalación.

El servicio bootkube coordina los pasos de la etapa de bootstrap.

Después de iniciar sesión en el nodo bootstrap, ejecute el siguiente comando para depurar ambos servicios:

```
sh-4.2# journalctl -b -f -u release-image.service -u bootkube.service
```

Además, ejecute el comando crictl en el nodo bootstrap para buscar contenedores fallidos e imprimir sus registros:

```
sh-4.2# sudo bash
sh-4.2# crictl ps -a
sh-4.2# crictl logs <container_id>
```

Los errores típicos en esta etapa incluyen los siguientes:

- El nodo bootstrap no puede descargar imágenes de contenedores desde quay.io o desde el registro local debido a problemas de autenticación o de red.

- Los nodos del clúster no pueden obtener la dirección IP de la API de Kubernetes debido a problemas de DNS.

**Solución de problemas en la etapa de bootstrap (plano de control temporal)**

Para solucionar problemas, debe obtener los registros del nodo bootstrap. Puede usar la clave SSH para obtener los registros del nodo bootstrap, como se explica en otra parte. Dado que la API de Kubernetes está disponible en el nodo bootstrap, también puede usar los comandos openshift-install y oc para obtener los registros de los nodos bootstrap y de plano de control.

```
[user@demo ~]$ export KUBECONFIG=${HOME}/ocp4-cluster/auth/kubeconfig
```

```
[user@demo ~]$ openshift-install gather bootstrap \
> --dir=${HOME}/ocp4-cluster
INFO Pulling debug logs from the bootstrap machine
INFO Bootstrap gather logs captured here "/home/user/ocp4-cluster/log-bundle-20210107135825.tar.gz"
```

```
[user@demo ~]$ cd ocp4-cluster
```

```
[user@demo ~]$ tar -xvzf log-bundle-20210107135825.tar.gz
```

```
[user@demo ~]$ cd log-bundle-20210107135825
```

```
[user@demo ~]$ ls
bootstrap/  control-plane/  failed-units.txt  rendered-assets/	resources/  unit-status/
```

Para solucionar problemas en los servicios systemd bootstrap, use los archivos de registro almacenados en el directorio bootstrap/journals/:

```
[user@demo ~]$ ls bootstrap/journals/
approve-csr.log  bootkube.log  crio-configure.log  crio.log  ironic.log  kubelet.log  master-update-bmh.log  release-image.log
```

Para solucionar problemas en los contenedores de bootstrap, use los archivos almacenados en el directorio bootstrap/containers/:

```
[user@demo ~]$ ls bootstrap/containers/
...output omitted...
cluster-version-operator-652...f5.inspect
cluster-version-operator-652...f5.log
...output omitted...
```

Para solucionar problemas en los nodos de plano de control, use los archivos de registro almacenados en el directorio control-plane/:

```
[user@demo ~]$ find  control-plane/
control-plane/
control-plane/10.0.203.117
control-plane/10.0.203.117/unit-status
control-plane/10.0.203.117/journals
control-plane/10.0.203.117/journals/kubelet.log
control-plane/10.0.203.117/journals/crio.log
...output omitted...
```

NOTA
Al usar el método de instalación de infraestructura preexistente, debe especificar las direcciones IP de los nodos bootstrap y de plano de control cuando ejecute el comando openshift-install gather bootstrap.

```
[user@demo ~]$ openshift-install gather bootstrap \
> --dir=${HOME}/ocp4-cluster \
> --bootstrap <bootstrap_address> \
> --master <master_1_address> \
> --master <master_2_address> \
> --master <master_3_address>"
```

También puede acceder a la API de Kubernetes que se ejecuta en el nodo bootstrap y monitorear el progreso de la instalación del clúster. Use los comandos oc estándares del host bastión para resolver problemas.

```
[user@demo ~]$ export KUBECONFIG=${HOME}/ocp4-cluster/auth/kubeconfig
```

```
[user@demo ~]$ oc get nodes
NAME       STATUS   ROLES    AGE     VERSION
master01   Ready    master   2m35s   v1.19.0+9f84db3
master02   Ready    master   2m34s   v1.19.0+9f84db3
master03   Ready    master   2m35s   v1.19.0+9f84db3
```

```
[user@demo ~]$ oc get clusterversion
NAME    VERSION AVAILABLE PROGRESSING SINCE STATUS
version         False     True        8m32s Unable to apply 4.6.4: an unknown error has occurred: MultipleErrors
```

```
[user@demo ~]$ oc get clusteroperators
NAME                    VERSION AVAILABLE PROGRESSING DEGRADED SINCE
authentication
cloud-credential                True      False       False    10m
console
csi-snapshot-controller
dns
etcd                     4.6.4  Unknown   Unknown     False    1s
...output omitted...
```

Puede seguir el proceso de instalación en detalle mediante el monitoreo de los eventos del clúster.

```
[user@demo ~]$ oc get events -A -w
...output omitted...
openshift-infra       0s     Warning  NetworkNotReady
```

En esta etapa, también puede usar el comando oc adm node-logs para recopilar los registros del nodo del clúster.

```
[user@demo ~]$ oc adm node-logs -u crio master01
...output omitted...
Jan 27 18:25:57.423858 master01 crio[1627]: time="2021-01-27 18:25:57.423599752Z" level=info msg="Checking image status: quay.io/openshift-release-dev/ocp-v4.0-art-dev@sha256:009...fd5" id=39...fe name=/runtime.v1alpha2.ImageService/ImageStatus
```

```
[user@demo ~]$ oc adm node-logs -u kubelet master01
...output omitted...
Jan 27 18:31:13.904091 master01 hyperkube[1661]: I0127 18:31:13.903991    1661 prober.go:126] Readiness probe for "etcd-master01_openshift-etcd(63...40):etcd" succeeded
```

```
[user@demo ~]$ oc adm node-logs master01
...output omitted...
Jan 27 18:32:47.487425 master01 hyperkube[1661]: I0127 18:32:47.487445    1661 kubelet.go:1914] SyncLoop (housekeeping)
Jan 27 18:32:47.536556 master01 hyperkube[1661]: I0127 18:32:47.536513    1661 exec.go:60] Exec probe response: ""
Jan 27 18:32:47.536556 master01 hyperkube[1661]: I0127 18:32:47.536546    1661 prober.go:126] Readiness probe for "ovs-z57pn_openshift-sdn(9e...2c):openvswitch" succeeded
```

Además, use el comando oc debug para crear una sesión de depuración en los nodos de plano de control para resolver problemas.

```
[user@demo ~]$ export KUBECONFIG=${HOME}/ocp4-cluster/auth/kubeconfig
```

```
[user@demo ~]$ oc get nodes
NAME       STATUS   ROLES    AGE     VERSION
master01   Ready    master   2m35s   v1.19.0+9f84db3
master02   Ready    master   2m34s   v1.19.0+9f84db3
master03   Ready    master   2m35s   v1.19.0+9f84db3
```

```
[user@demo ~]$ oc debug node/master01
...output omitted...
sh-4.2#
Una vez que inicie sesión, ejecute el siguiente comando para depurar los registros de nodos:

sh-4.2# chroot /host
sh-4.2# journalctl -f
```

Además, use el comando crictl en el nodo de plano de control para buscar contenedores fallidos y obtener sus registros:

```
sh-4.2# sudo bash
sh-4.2# crictl ps -a
sh-4.2# crictl logs <container_id>
```

Los errores típicos en esta etapa incluyen los siguientes:

Problemas de instalación de los nodos de plano de control

Problemas de resolución DNS

**Solución de problemas en la etapa de plano de control de producción**

En esta etapa, la API de Kubernetes y el plano de control de producción se están ejecutando en los nodos de plano de control. El operador de versión del clúster (CVO) que se ejecuta en el plano de control de producción instala todos los operadores que compilan el clúster de OpenShift y finalizan la instalación. Puede aplicar las mismas técnicas de solución de problemas usadas en la sección anterior. Los errores típicos en esta etapa incluyen los siguientes:

Problemas de instalación de operadores de OpenShift

### Verificación de instalaciones de OpenShift

Una vez que la instalación de OpenShift finalice correctamente, los administradores deben asegurarse de que el clúster instalado esté en buen estado y listo para las tareas del Día 2 a fin de incorporar usuarios y aplicaciones.

**Estado del clúster de OpenShift**

Desde el host bastión, puede realizar una comprobación de estado básica con el comando oc.

Configure la variable de entorno KUBECONFIG para autenticarse contra la API de Kubernetes con permisos de cluster-admin.

```
[user@demo ~]$ export KUBECONFIG=${HOME}/ocp4-cluster/auth/kubeconfig
```

Verifique que todos los nodos del clúster tengan el reloj del sistema sincronizado con un servidor de Protocolo de tiempo en red (NTP).

```
[user@demo ~]$ oc debug node/master01
...output omitted...
sh-4.4# chroot /host
sh-4.4# cat /etc/chrony.conf
# Use public servers from the pool.ntp.org project.
# Please consider joining the pool (http://www.pool.ntp.org/join.html).
pool 2.rhel.pool.ntp.org iburst
...output omitted...

sh-4.4# sudo chronyc tracking
Reference ID    : 8AEC8070 (time.gac.edu)
Stratum         : 3
Ref time (UTC)  : Thu Feb 11 13:06:57 2021
System time     : 0.000034756 seconds fast of NTP time
Last offset     : -0.000001187 seconds
RMS offset      : 0.004707427 seconds
Frequency       : 28.194 ppm fast
Residual freq   : -0.000 ppm
Skew            : 0.136 ppm
Root delay      : 0.052070152 seconds
Root dispersion : 0.018801220 seconds
Update interval : 64.9 seconds
Leap status     : Normal
```

Verifique que todos los nodos del clúster tengan el estado Ready.

Si un nodo del clúster no tiene el estado Ready (Listo), no puede comunicarse con el plano de control de OpenShift y no está disponible para el clúster.

```
[user@demo ~]$ oc get nodes
```

Verifique que todos los nodos del clúster informen métricas de uso.

```
[user@demo ~]$ oc adm top node
```

Asegúrese de que no haya solicitudes de firma de certificado (CSR) pendientes de aprobación.

```
[user@demo ~]$ oc get csr | grep Pending
```

Verifique que el informe del operador de versión del clúster muestre que el clúster de OpenShift está disponible y listo.

```
[user@demo ~]$ oc get clusterversion
```

Verifique que todos los operadores del clúster estén disponibles y listos.

Si el clúster está en buen estado, todos los operadores del clúster deben estar disponibles y no en progreso, a menos que uno o más operadores aún estén aplicando la configuración.

```
[user@demo ~]$ oc get clusteroperators
```

Verifique que no haya pods con problemas de programación o ejecución en el clúster.

```
[user@demo ~]$ oc get pods --all-namespaces | grep -v -E 'Running|Completed'
```

**Estado de Etcd de OpenShift**

Asegúrese de que todos los miembros del clúster de etcd estén en buen estado.

```
[user@demo ~]$ oc get pods -n openshift-etcd | grep etcd-master
etcd-master01  3/3 Running  0  22h
etcd-master02  3/3 Running  0  22h
etcd-master03  3/3 Running  0  22h
```

```
[user@demo ~]$ oc rsh -n openshift-etcd etcd-master01
sh-4.4# etcdctl endpoint health --cluster
https://192.168.50.10:2379 is healthy: successfully committed proposal: took=10.8ms
https://192.168.50.12:2379 is healthy: successfully committed proposal: took=11.8ms
https://192.168.50.11:2379 is healthy: successfully committed proposal: took=12.1ms
```

**Estado de la consola y API de OpenShift**

Verifique que el registro de DNS de la API de OpenShift api.ocp4.example.com esté configurado para usar la dirección IP del balanceador de carga externo 192.168.50.254.

```
[user@demo ~]$ dig api.ocp4.example.com
...output omitted...
;; QUESTION SECTION:
;api.ocp4.example.com.	IN	A

;; ANSWER SECTION:
api.ocp4.example.com.	85333	IN	A	192.168.50.254

;; Query time: 0 msec
;; SERVER: 172.25.250.254#53(172.25.250.254)
;; WHEN: Thu Jan 28 05:11:46 EST 2021
;; MSG SIZE  rcvd: 71
```

Solicite la versión de Kubernetes para verificar que la API de OpenShift esté disponible.

```
[user@demo ~]$ curl -k https://api.ocp4.example.com:6443/version
...output omitted...
  "gitVersion": "v1.19.0+9f84db3",
...output omitted...
```

Verifique que puede conectarse a la consola de OpenShift.

```
[user@demo ~]$ curl -kIs \
> https://console-openshift-console.apps.ocp4.example.com
...output omitted...
HTTP/1.1 200 OK
...output omitted...
```

```
[user@demo ~]$ firefox https://console-openshift-console.apps.ocp4.example.com
```

**Estado del registro de OpenShift**

Asegúrese de que la cantidad de pods de registro interno que se ejecutan en el clúster de OpenShift coincida con su configuración de implementación.

```
[user@demo ~]$ oc -n openshift-image-registry get deployment.apps/image-registry
```

Si hay varios nodos de cómputo, verifique que cada pod de registro se esté ejecutando en un nodo de cómputo diferente.

```
[user@demo ~]$ oc -n openshift-image-registry  get pods -o wide
```

Desde cualquier nodo de clúster, verifique el estado del registro interno.

```
[user@demo ~]$ oc debug node/master01
...output omitted...
sh-4.4# chroot /host
sh-4.4# curl -kIs \
> https://image-registry.openshift-image-registry.svc:5000/healthz
...output omitted...
HTTP/2 200
...output omitted...
```

Verifique que la implementación del registro interno esté usando almacenamiento persistente. Además, asegúrese de que el operador de registro de imágenes tenga el estado de administración Managed (Administrado).

```
[user@demo ~]$ oc get configs.imageregistry.operator.openshift.io cluster -o yaml
...output omitted...
spec:
  managementState: Managed
...output omitted...
  storage:
    pvc:
      claim: registry-claim
...output omitted...
```

**Estado de Ingress de OpenShift**

Verifique que el registro de DNS comodín para las aplicaciones, *.apps.ocp4.example.com, esté configurado para usar la dirección IP del balanceador de carga externo 192.168.50.254.

```
[user@demo ~]$ dig test.apps.ocp4.example.com
```

Verifique que puede acceder a una aplicación expuesta por una ruta de Ingress de OpenShift.

```
[user@demo ~]$ oc get routes -A | grep downloads
openshift-console downloads downloads-openshift-console.apps.ocp4.example.com
```

```
[user@demo ~]$ curl -kIs \
> https://downloads-openshift-console.apps.ocp4.example.com
...output omitted...
HTTP/1.0 200 OK
...output omitted...
```

Asegúrese de que la cantidad de pods del enrutador que se ejecutan en el clúster de OpenShift coincida con su configuración de implementación.

```
[user@demo ~]$ oc -n openshift-ingress get deployment.apps/router-default
```

Si hay varios nodos de cómputo, verifique que cada pod del enrutador se esté ejecutando en un nodo de cómputo diferente.

```
[user@demo ~]$ oc -n openshift-ingress  get pods -o wide | grep router
```

**Estado del proveedor de almacenamiento dinámico de OpenShift**

Al instalar OpenShift en un proveedor de nube soportado, el instalador configura un proveedor de almacenamiento dinámico. En este caso, debe verificar el estado del proveedor de almacenamiento dinámico.

Durante la instalación de OpenShift en AWS mediante el método de automatización full-stack, el instalador configura un proveedor de almacenamiento dinámico de EBS en AWS. Este proveedor de almacenamiento dinámico usa el aprovisionador de almacenamiento aws-ebs.

El proceso de instalación de OpenShift crea una clase de almacenamiento denominada gp2 que usa el proveedor de almacenamiento dinámico de EBS en AWS como back-end. La clase de almacenamiento gp2 aprovisiona dinámicamente almacenamiento persistente para las aplicaciones contenerizadas que se ejecutan en el clúster de OpenShift.

Verifique el estado de la clase de almacenamiento gp2 de EBS en AWS.

```
[user@demo ~]$ oc get sc
```

Verifique que la clase de almacenamiento gp2 funcione de la forma esperada.

Cree una aplicación httpd simple que use almacenamiento persistente para su directorio DocumentRoot en /var/www/html.

```
[user@demo ~]$ oc new-project httpd-persistent
...output omitted...
[user@demo ~]$ cat /tmp/httpd-persistent.yaml
[user@demo ~]$ oc create -f /tmp/httpd-persistent.yaml
```

Verifique que la creación de PVC active automáticamente el aprovisionamiento y la vinculación del PV a través de la clase de almacenamiento predeterminada gp2.

```
[user@demo ~]$ oc get pvc
NAME          STATUS   VOLUME       CAPACITY   ACCESS MODES   STORAGECLASS   AGE
httpd-claim   Bound    pvc-d965cb1f  3Gi        RWO            gp2            29s
[user@demo ~]$ oc rsh httpd
sh-4.4$ df -h
```

**Prueba de implementación y compilación a través de una aplicación de OpenShift**

Compile e implemente una aplicación de prueba para verificar el ciclo de compilación de la aplicación de OpenShift.

```
[user@demo ~]$ oc new-project validate
...output omitted...
[user@demo ~]$ oc new-app django-psql-example
...output omitted...
[user@demo ~]$ oc get pods -n validate
NAME                           READY   STATUS      RESTARTS   AGE
django-psql-example-1-build    0/1     Completed   0          11m
django-psql-example-1-deploy   0/1     Completed   0          10m
django-psql-example-1-vfb5l    1/1     Running     0          10m
postgresql-1-bdgkk             1/1     Running     0          11m
postgresql-1-deploy            0/1     Completed   0          11m
[user@demo ~]$ oc logs -f django-psql-example-1-build
...output omitted...
Successfully pushed image-registry.openshift-image-registry.svc:5000/validate/django-psql-example@sha256:b97b...ff82
Push successful
[user@demo ~]$ oc logs -f django-psql-example-1-deploy
...output omitted...
--> Scaling django-psql-example-1 to 1
--> Success
[user@demo ~]$ oc get routes -n validate
NAME                HOST/PORT                                                PATH SERVICES            PORT  TERMINATION WILDCARD
django-psql-example django-psql-example-validate.apps.ocp4.example.com      django-psql-example <all>             None
[user@demo ~]$ curl -Is \
> django-psql-example-validate.apps.ocp4.example.com
...output omitted...
HTTP/1.1 200 OK
...output omitted...
[user@demo ~]$ firefox http://django-psql-example-validate.apps.ocp4-aws.example.com
```

**Red de clústeres de OpenShift**

Verifique las configuraciones de la red del clúster de OpenShift.

```
[user@demo ~]$ oc get network.config/cluster -o yaml
```

**Rendimiento del almacenamiento de Etcd de OpenShift**

Verifique el rendimiento del almacenamiento de etcd.

```
[user@demo ~]$ oc get pods -n openshift-etcd | grep etcd-master
```

```
[user@demo ~]$ oc rsh -n openshift-etcd etcd-master01
...output omitted...
sh-4.4# etcdctl check perf --load="s"
 60 / 60 Boooooooooooooooooooooooooooooooooooooooooooooooooo! 100.00% 1m0s
PASS: Throughput is 150 writes/s
PASS: Slowest request took 0.220329s
PASS: Stddev is 0.018010s
PASS

sh-4.4# etcdctl check perf --load="m"
 60 / 60 Boooooooooooooooooooooooooooooooooooooooooooooooooo! 100.00% 1m0s
PASS: Throughput is 964 writes/s
PASS: Slowest request took 0.379547s
PASS: Stddev is 0.022218s
PASS

sh-4.4# etcdctl check perf --load="l"
 60 / 60 Boooooooooooooooooooooooooooooooooooooooooooooooooo! 100.00% 1m0s
FAIL: Throughput too low: 4586 writes/s
PASS: Slowest request took 0.258474s
PASS: Stddev is 0.032695s
FAIL
```

### Configuración de hosts y servicios de red para la instalación de OpenShift sin un proveedor de infraestructura

El archivo de configuración predeterminado para el servidor httpd es /etc/httpd/conf/httpd.conf. Dado que lo usará como servidor de archivos, debe encontrar el directorio donde colocará los archivos. Para encontrar este directorio, ejecute el siguiente comando.

```
[root@utility ~]# grep DocumentRoot /etc/httpd/conf/httpd.conf
# DocumentRoot: The directory out of which you will serve your
DocumentRoot "/var/www/html"
...output omitted...
```

Cree la carpeta /var/www/html/openshift4/images.

```
[root@utility ~]# mkdir -p /var/www/html/openshift4/images
```

Cree la carpeta /var/www/html/openshift4/4.6.4/ignitions.

```
[root@utility ~]# mkdir -p /var/www/html/openshift4/4.6.4/ignitions
```

Restaure el contexto de seguridad de SELinux de la carpeta /var/www/html/openshift4/images.

```
[root@utility ~]# restorecon -Rv /var/www/html/openshift4
```

Revise la estructura de la carpeta /var/www/html:

```
[root@utility ~]# tree /var/www/html/
 /var/www/html/
 └── openshift4
    ├── 4.6.4
    │   └── ignitions
    └── images
```

El servidor httpd está preparado para colocar los archivos de encendido (ignition) en /var/www/html/openshift4/4.6.4/ignitions/ y las imágenes RHCOS en /var/www/html/openshift4/images/.

Descargue las imágenes RHCOS en la carpeta /var/www/html/openshift4/images/:

```
[root@utility ~]# ocp_maj=4.6;rhcos_ver=4.6.1
[root@utility ~]# mirror=https://mirror.openshift.com/pub/openshift-v4
[root@utility ~]# baseurl=${mirror}/dependencies/rhcos/${ocp_maj}/${rhcos_ver}
[root@utility ~]# cd /var/www/html/openshift4/images/
[root@utility images]# wget \
> ${baseurl}/rhcos-${rhcos_ver}-x86_64-live-rootfs.x86_64.img
...output omitted...
2020-11-23 10:14:02 (57,0 MB/s) - “rhcos-4.6.1-x86_64-live-rootfs.x86_64.img” saved [819969024/819969024]
[root@utility images]# wget \
> ${baseurl}/rhcos-${rhcos_ver}-x86_64-live-kernel-x86_64
...output omitted...

2020-11-23 10:15:50 (6,26 MB/s) - “rhcos-4.6.1-x86_64-live-kernel-x86_64” saved [8924528/8924528]
[root@utility images]# wget \
> ${baseurl}/rhcos-${rhcos_ver}-x86_64-live-initramfs.x86_64.img
...output omitted...

2020-11-23 10:16:33 (29,2 MB/s) - “rhcos-4.6.1-x86_64-live-initramfs.x86_64.img” saved [80239428/80239428]
```

Regrese a la carpeta de inicio root:

```
[root@utility images]# cd ~
[root@utility ~]#
```

Verifique y configure (si es necesario) el servicio del balanceador de carga HAProxy provisto por el daemon haproxy. El archivo de configuración es /etc/haproxy/haproxy.cfg, como se muestra en la salida del estado de la unidad systemd:

```
[root@utility ~]$ systemctl status haproxy
```

Edite el archivo /etc/haproxy/haproxy.cfg para agregar los servidores de back end para APIServer, el servidor Machine Config y las aplicaciones (seguras y no seguras).

```
[root@utility ~]# vi /etc/haproxy/haproxy.cfg

...output omitted...
#---------------------------------------------------------------------
# round robin balancing for RHOCP Kubernetes API Server
#---------------------------------------------------------------------
 frontend k8s_api
   bind *:6443
   mode tcp
   default_backend k8s_api_backend
 backend k8s_api_backend
   balance roundrobin
   mode tcp
   server bootstrap 192.168.50.9:6443 check
   server master01 192.168.50.10:6443 check
   server master02 192.168.50.11:6443 check
   server master03 192.168.50.12:6443 check

# ---------------------------------------------------------------------
# round robin balancing for RHOCP Machine Config Server
# ---------------------------------------------------------------------
 frontend machine_config
   bind *:22623
   mode tcp
   default_backend machine_config_backend
 backend machine_config_backend
   balance roundrobin
   mode tcp
   server bootstrap 192.168.50.9:22623 check
   server master01 192.168.50.10:22623 check
   server master02 192.168.50.11:22623 check
   server master03 192.168.50.12:22623 check

# ---------------------------------------------------------------------
# round robin balancing for RHOCP Ingress Insecure Port
# ---------------------------------------------------------------------
 frontend ingress_insecure
   bind *:80
   mode tcp
   default_backend ingress_insecure_backend
 backend ingress_insecure_backend
   balance roundrobin
   mode tcp
   server worker01 192.168.50.13:80 check
   server worker02 192.168.50.14:80 check

# ---------------------------------------------------------------------
# round robin balancing for RHOCP Ingress Secure Port
# ---------------------------------------------------------------------
 frontend ingress_secure
   bind *:443
   mode tcp
   default_backend ingress_secure_backend
 backend ingress_secure_backend
   balance roundrobin
   mode tcp
   server worker01 192.168.50.13:443 check
   server worker02 192.168.50.14:443 check

# ---------------------------------------------------------------------
# Exposing HAProxy Statistic Page
# ---------------------------------------------------------------------
 listen stats
     bind :32700
     stats enable
     stats uri /
     stats hide-version
     stats auth admin:RedH@t322
```
Guarde los cambios en el archivo /etc/haproxy/haproxy.cfg.

Verifique que el archivo haproxy.cfg no contenga errores. Recargue el servicio HAProxy para aplicar los cambios. Verifique el estado del servicio.

```
[root@utility ~]# haproxy -c -f /etc/haproxy/haproxy.cfg
```
```
[root@utility ~]# systemctl reload haproxy
[root@utility ~]$ systemctl status haproxy
```

Verifique y configure el servicio TFTP. Después de este paso, la estructura de la carpeta /var/lib/tftpboot/pxelinux.cfg/ debe verse de la siguiente forma:

```
[root@utility ~]# tree /var/lib/tftpboot/pxelinux.cfg/
```

Verifique y configure (si es necesario) el servicio TFTP provisto por el daemon tftp.

```
[root@utility ~]# systemctl status tftp
```

Con las direcciones MAC del archivo /etc/dhcp/dhcpd.conf al que se hace referencia en el paso anterior, cree seis archivos en /var/lib/tftpboot/pxelinux.cfg/ de la siguiente manera.

Anteponga 01- a la dirección MAC en minúscula, separando cada par de dígitos hexadecimales con un guion.

master01 - /var/lib/tftpboot/pxelinux.cfg/01-52-54-00-00-32-0a

master02 - /var/lib/tftpboot/pxelinux.cfg/01-52-54-00-00-32-0b

master03 - /var/lib/tftpboot/pxelinux.cfg/01-52-54-00-00-32-0c

Contenido de los tres archivos:

```
default menu.c32
prompt 0
timeout 0
menu title **** OpenShift 4 MASTER PXE Boot Menu ****

label Install RHCOS 4.6.1 Master Node
 kernel http://192.168.50.254:8080/openshift4/images/rhcos-4.6.1-x86_64-live-kernel-x86_64
 append ip=dhcp rd.neednet=1 coreos.inst.install_dev=vda console=tty0 console=ttyS0 coreos.inst=yes coreos.live.rootfs_url=http://192.168.50.254:8080/openshift4/images/rhcos-4.6.1-x86_64-live-rootfs.x86_64.img coreos.inst.ignition_url=http://192.168.50.254:8080/openshift4/4.6.4/ignitions/master.ign initrd=http://192.168.50.254:8080/openshift4/images/rhcos-4.6.1-x86_64-live-initramfs.x86_64.img

 ```

 bootstrap - /var/lib/tftpboot/pxelinux.cfg/01-52-54-00-00-32-09

Contenido del archivo:

```
default menu.c32
prompt 0
timeout 0
menu title **** OpenShift 4 BOOTSTRAP PXE Boot Menu ****

label Install RHCOS 4.6.1 Bootstrap Node
 kernel http://192.168.50.254:8080/openshift4/images/rhcos-4.6.1-x86_64-live-kernel-x86_64
 append ip=dhcp rd.neednet=1 coreos.inst.install_dev=vda console=tty0 console=ttyS0 coreos.inst=yes coreos.live.rootfs_url=http://192.168.50.254:8080/openshift4/images/rhcos-4.6.1-x86_64-live-rootfs.x86_64.img coreos.inst.ignition_url=http://192.168.50.254:8080/openshift4/4.6.4/ignitions/bootstrap.ign initrd=http://192.168.50.254:8080/openshift4/images/rhcos-4.6.1-x86_64-live-initramfs.x86_64.img

 ```

worker01 - /var/lib/tftpboot/pxelinux.cfg/01-52-54-00-00-32-0d

worker02 - /var/lib/tftpboot/pxelinux.cfg/01-52-54-00-00-32-0e

Contenido de los dos archivos:

```
default menu.c32
prompt 0
timeout 0
menu title **** OpenShift 4 WORKER PXE Boot Menu ****

label Install RHCOS 4.6.1 Worker Node
 kernel http://192.168.50.254:8080/openshift4/images/rhcos-4.6.1-x86_64-live-kernel-x86_64
 append ip=dhcp rd.neednet=1 coreos.inst.install_dev=vda console=tty0 console=ttyS0 coreos.inst=yes coreos.live.rootfs_url=http://192.168.50.254:8080/openshift4/images/rhcos-4.6.1-x86_64-live-rootfs.x86_64.img coreos.inst.ignition_url=http://192.168.50.254:8080/openshift4/4.6.4/ignitions/worker.ign initrd=http://192.168.50.254:8080/openshift4/images/rhcos-4.6.1-x86_64-live-initramfs.x86_64.img

 ```

 Con el usuario lab, cree los archivos de encendido (ignition) y colóquelos en el servidor de archivos.

 Cree una carpeta denominada ocp4upi y cambie a esa carpeta:

```
[root@utility ~]# exit
logout
[lab@utility ~]$ mkdir ocp4upi && cd ocp4upi
[lab@utility ocp4upi]$
```

Anteriormente en este curso, generó el archivo pull-secret-oneline.json con el registro local de su región y lo copió en el servidor utility.

```
[lab@utility ocp4upi]$ cat ~/pull-secret-oneline.json
```
Tras esto, solo quedará crear el install-config.yaml y crear los manifiestos.


### Realización de las operaciones del Día 1 y del Día 2

Con el usuario student, copie el archivo kubeconfig de /home/lab/ocp4upi/auth/kubeconfig a la máquina de la estación de trabajo.

```
[student@workstation ~]$ ssh lab@utility
```

```
[lab@utility ~]$ scp /home/lab/ocp4upi/auth/kubeconfig student@workstation:
```

Configure el registro de imágenes con almacenamiento persistente.

Use el archivo kubeconfig para realizar la autenticación en OpenShift.

```
[lab@utility ~]$ export KUBECONFIG=~/ocp4upi/auth/kubeconfig
```



Investigue las exportaciones NFS en el servidor utility.

```
[lab@utility ~]$ cat /etc/exports
/exports *(rw,sync,no_wdelay,no_root_squash,insecure,fsid=0)

[lab@utility ~]$ lsblk
NAME   MAJ:MIN RM  SIZE RO TYPE MOUNTPOINT
vda    252:0    0   10G  0 disk
├─vda1 252:1    0    1M  0 part
├─vda2 252:2    0  100M  0 part /boot/efi
└─vda3 252:3    0  9,9G  0 part /
vdb    252:16   0   40G  0 disk
└─vdb1 252:17   0   40G  0 part /exports

[lab@utility ~]$ df -h
Filesystem       Size   Used Avail Use% Mounted on
...output omitted...
/dev/vdb1         40G   318M   40G   1% /exports
/dev/vda2        100M   6,8M   94M   7% /boot/efi
tmpfs            183M      0  183M   0% /run/user/1000
```

La exportación NFS /exports tiene aproximadamente 40 GB de espacio en disco disponible.

Cree el archivo de volumen persistente (PV) denominado pv.yaml.

```
[lab@utility ~]$ vi pv.yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: registry-pv
spec:
  capacity:
    storage: 5Gi
  accessModes:
  - ReadWriteMany
  nfs:
    path: /exports/registry
    server: 192.168.50.254
  persistentVolumeReclaimPolicy: Recycle
  ```
Cree la carpeta /exports/registry con los permisos adecuados y el volumen persistente (PV) desde el archivo pv.yaml:

```
[lab@utility ~]$ mkdir /exports/registry
[lab@utility ~]$ sudo chmod 777 /exports/registry/
[lab@utility ~]$ oc create -f pv.yaml
persistentvolume/registry-pv created
```

Cree un archivo llamado pvc.yaml con el siguiente contenido.

```
[lab@utility ~]$ vi pvc.yaml
kind: PersistentVolumeClaim
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: registry-claim
  namespace: openshift-image-registry
spec:
  accessModes:
    - ReadWriteMany
  resources:
    requests:
      storage: 5Gi
```

Cree la reclamación de volumen persistente (PVC) a partir del archivo pvc.yaml:

```
[lab@utility ~]$ oc create -f pvc.yaml
```
En la sección spec, establezca el operador de registro de imágenes en estado "Managed" (Administrado). Edite la configuración del registro de imágenes de clúster para agregar la PVC. Además, configure el registro de imágenes para que tenga dos réplicas de pod.

```
[lab@utility ~]$ oc edit configs.imageregistry/cluster
...output omitted...
spec:
...output omitted...
  managementState: Managed
...output omitted...
  proxy: {}
  replicas: 2
  requests:
...output omitted...
  rolloutStrategy: RollingUpdate
  storage:
    pvc:
      claim: registry-claim
...output omitted...
```
Verifique el estado del operador del registro de imágenes, PV y PVC.

```
[lab@utility ~]$ oc get pv -A
NAME          CAPACITY   ACCESS MODES   RECLAIM POLICY   STATUS   CLAIM                                     STORAGECLASS   REASON   AGE
registry-pv   5Gi        RWX            Recycle          Bound    openshift-image-registry/registry-claim                           111m

[lab@utility ~]$ oc get pvc -A
NAMESPACE                  NAME             STATUS   VOLUME        CAPACITY   ACCESS MODES   STORAGECLASS   AGE
openshift-image-registry   registry-claim   Bound    registry-pv   5Gi        RWX                           105m

[lab@utility ~]$ oc get clusteroperator
```

Verifique la cantidad de pods de registro en el espacio de nombres openshift-image-registry.

```
[lab@utility ~]$ oc get pods -n openshift-image-registry -o wide
```

Configure un aprovisionador de almacenamiento dinámico.

Clone el repositorio de GitHub kubernetes-sigs/nfs-subdir-external-provisioner:

```
[lab@utility ~]$ git clone \
> https://github.com/kubernetes-sigs/nfs-subdir-external-provisioner/
```

Edite los archivos de la carpeta deploy con información del servidor NFS en utility. La salida debería verse de la siguiente manera:

```
[lab@utility ~]$ vi nfs-subdir-external-provisioner/deploy/class.yaml
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: nfs-client
  annotations:
    storageclass.kubernetes.io/is-default-class: "true"
provisioner: nfs-dynamic-provisioner
reclaimPolicy: Retain
...output omitted...
```

```
[lab@utility ~]$ vi nfs-subdir-external-provisioner/deploy/rbac.yaml
apiVersion: v1
kind: ServiceAccount
metadata:
  name: nfs-client-provisioner
  namespace: nfs-dynamic-namespace
---
kind: ClusterRole
apiVersion: rbac.authorization.k8s.io/v1
metadata:
  name: nfs-client-provisioner-runner
rules:
  - apiGroups: [""]
    resources: ["persistentvolumes"]
    verbs: ["get", "list", "watch", "create", "delete"]
  - apiGroups: [""]
    resources: ["persistentvolumeclaims"]
    verbs: ["get", "list", "watch", "update"]
  - apiGroups: ["storage.k8s.io"]
    resources: ["storageclasses"]
    verbs: ["get", "list", "watch"]
  - apiGroups: [""]
    resources: ["events"]
    verbs: ["create", "update", "patch"]
---
kind: ClusterRoleBinding
apiVersion: rbac.authorization.k8s.io/v1
metadata:
  name: run-nfs-client-provisioner
subjects:
  - kind: ServiceAccount
    name: nfs-client-provisioner
    namespace: nfs-dynamic-namespace
roleRef:
  kind: ClusterRole
  name: nfs-client-provisioner-runner
  apiGroup: rbac.authorization.k8s.io
---
kind: Role
apiVersion: rbac.authorization.k8s.io/v1
metadata:
  name: leader-locking-nfs-client-provisioner
  namespace: nfs-dynamic-namespace
rules:
  - apiGroups: [""]
    resources: ["endpoints"]
    verbs: ["get", "list", "watch", "create", "update", "patch"]
---
kind: RoleBinding
apiVersion: rbac.authorization.k8s.io/v1
metadata:
  name: leader-locking-nfs-client-provisioner
  namespace: nfs-dynamic-namespace
subjects:
  - kind: ServiceAccount
    name: nfs-client-provisioner
    namespace: nfs-dynamic-namespace
roleRef:
  kind: Role
  name: leader-locking-nfs-client-provisioner
  apiGroup: rbac.authorization.k8s.io
```

```
[lab@utility ~]$ vi nfs-subdir-external-provisioner/deploy/deployment.yaml

apiVersion: apps/v1
kind: Deployment
metadata:
  name: nfs-client-provisioner
  labels:
    app: nfs-client-provisioner
  namespace: nfs-dynamic-namespace
spec:
  replicas: 1
  strategy:
    type: Recreate
  selector:
    matchLabels:
      app: nfs-client-provisioner
  template:
    metadata:
      labels:
        app: nfs-client-provisioner
    spec:
      serviceAccountName: nfs-client-provisioner
      containers:
        - name: nfs-client-provisioner
          image: registry.k8s.io/sig-storage/nfs-subdir-external-provisioner:v4.0.2
          volumeMounts:
            - name: nfs-client-root
              mountPath: /persistentvolumes
          env:
            - name: PROVISIONER_NAME
              value: nfs-dynamic-provisioner
            - name: NFS_SERVER
              value: 192.168.50.254
            - name: NFS_PATH
              value: /exports
      volumes:
        - name: nfs-client-root
          nfs:
            server: 192.168.50.254
            path: /exports
```

Cree los objetos en el espacio de nombres nfs-dynamic-namespace.

```
[lab@utility ~]$ oc create namespace nfs-dynamic-namespace
namespace/nfs-dynamic-namespace created
[lab@utility ~]$ oc create -f nfs-subdir-external-provisioner/deploy/rbac.yaml
```

Cree el rol use-scc-hostmount-anyuid en el espacio de nombres nfs-dynamic-namespace. Agregue el rol use-scc-hostmount-anyuid a la cuenta de servicio nfs-client-provisioner.

```
[lab@utility ~]$ oc create role use-scc-hostmount-anyuid --verb=use \
> --resource=scc --resource-name=hostmount-anyuid -n nfs-dynamic-namespace
role.rbac.authorization.k8s.io/use-scc-hostmount-anyuid created
```

```
[lab@utility ~]$ oc project nfs-dynamic-namespace
Now using project "nfs-dynamic-namespace" on server "https://api.ocp4.example.com:6443".
```

```
[lab@utility ~]$ oc adm policy add-role-to-user use-scc-hostmount-anyuid \
> -z nfs-client-provisioner --role-namespace='nfs-dynamic-namespace'
role.rbac.authorization.k8s.io/use-scc-hostmount-anyuid added: "nfs-client-provisioner"
```

Cree la implementación.

```
[lab@utility ~]$ oc create -f \
> nfs-subdir-external-provisioner/deploy/deployment.yaml
```

Verifique todos los recursos creados en el espacio de nombres nfs-dynamic-namespace.

```
[lab@utility ~]$ oc get all
```

Cree la NFS StorageClass.

```
[lab@utility ~]$ oc create -f \
> nfs-subdir-external-provisioner/deploy/class.yaml
```

```
[lab@utility ~]$ oc get storageclass
```

Pruebe el aprovisionamiento de volúmenes dinámicos NFS.

```
[lab@utility ~]$ oc create -f \
> nfs-subdir-external-provisioner/deploy/test-claim.yaml
```

```
[lab@utility ~]$ oc get pvc
[lab@utility ~]$ oc create -f \
> nfs-subdir-external-provisioner/deploy/test-pod.yaml
[lab@utility ~]$ oc get all
```

Si el aprovisionador dinámico NFS funciona correctamente, habrá un archivo SUCCESS en la carpeta de exportación NFS.

```
[lab@utility ~]$ tree /exports/
```







