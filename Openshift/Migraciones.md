# Migraciones en Openshift


### Estado de los certificados de cluster

Verifique y apruebe cualquier CSR pendiente.

```
[lab@installer ~]$ oc get csr | grep Pending
No resources found.
```

Si el comando anterior muestra alguna CSR pendiente, fírmela con el siguiente comando:

```
[user@demo ~]$ oc get csr -o go-template='{{range .items}}{{if not .status}}{{.metadata.name}}{{"\n"}}{{end}}{{end}}' | xargs oc adm certificate approve
```

### Implementación del MTC en el clúster de origen

MTC (Herramientas para la migración en Contenedores)

Para instalar el operador MTC en un clúster de OpenShift 3, inicie sesión y descargue los archivos YAML necesarios con los siguientes comandos:registry.redhat.io

```
[user@demo ~]$ podman login registry.redhat.io
[user@demo ~]$ mtc_img=rhmtc/openshift-migration-legacy-rhel8-operator:v1.5
[user@demo ~]$ sudo podman cp $(sudo podman create \
> registry.redhat.io/${mtc_img}):/operator.yml ./
[user@demo ~]$ sudo podman cp $(sudo podman create \
> registry.redhat.io/${mtc_img}):/controller.yml ./
```

A continuación, cree el operador y sus recursos utilizando los archivos YAML descargados:

```
[user@demo ~]$ oc create -f operator.yml
...output omitted...
[user@demo ~]$ oc create -f controller.yml
...output omitted...
```

**Configuración del MTC**

Para acceder a la consola web de MTC después de la instalación, obtenga la URL del clúster de OpenShift 4 de destino:

```
[user@demo ~]$ oc get -n openshift-migration route/migration
NAME        HOST/PORT                                             ...
migration   migration-openshift-migration.apps.ocp4.example.com   ..
```

**Eliminación del controlador de migración y sus recursos**

Para eliminar el controlador de migración en ambos clústeres, ejecute lo siguiente:

```
[user@demo ~]$ oc get migrationcontroller -n openshift-migration
NAME                   AGE
migration-controller   7d
[user@demo ~]$ oc delete migrationcontroller migration-controller \
> -n openshift-migration
```

**Desinstalación del operador MTC**

En OpenShift 4, desinstale el operador de la consola web y, a continuación, elimine el espacio de nombres ejecutando:

```
[user@demo ~]$ oc delete ns openshift-migration
```

En OpenShift 3, desinstale el operador utilizando el mismo archivo utilizado para instalarlo:operator.yml

```
[user@demo ~]$ oc delete -f operator.yml
```

**Eliminar el resto de los recursos con ámbito de clúster**

Hay varios recursos de ámbito de clúster creados durante una migración. Elimínelos todos ejecutando los siguientes comandos en ambos clústeres:

```
[user@demo ~]$ oc delete $(oc get crds -o name | grep 'migration.openshift.io')
[user@demo ~]$ oc delete $(oc get crds -o name | grep 'velero')
[user@demo ~]$ oc delete $(oc get clusterroles -o name | \
> grep 'migration.openshift.io')
[user@demo ~]$ oc delete clusterrole migration-operator
[user@demo ~]$ oc delete $(oc get clusterroles -o name | grep 'velero')
[user@demo ~]$ oc delete $(oc get clusterrolebindings -o name | \
> grep 'migration.openshift.io')
[user@demo ~]$ oc delete clusterrolebindings migration-operator
[user@demo ~]$ oc delete $(oc get clusterrolebindings -o name | grep 'velero')
```

**Recuperar token de cuenta de servicio del controlador de migración**

Desde un terminal activado en , recupere el token de cuenta de servicio del controlador de migración para el clúster de origen.workstation

```
[lab@utility ~]$ exit
...output omitted...
[student@workstation ~]$ ssh lab@installer oc sa get-token \
> migration-controller -n openshift-migration
...output omitted...
```

### Adaptación de límites para migraciones grandes

Si el número de objetos implicados en la migración es alto, es posible que los administradores deban aumentar los límites predeterminados. Esto se hace editando el objeto.migrationcontroller

```
[user@demo ~]$ oc edit migrationcontroller -n openshift-migration
...output omitted...
mig_controller_limits_cpu: "1" 1
mig_controller_limits_memory: "10Gi" 2
...output omitted...
mig_controller_requests_cpu: "100m" 3
mig_controller_requests_memory: "350Mi" 4
...output omitted...
mig_pv_limit: 100 5
mig_pod_limit: 100 6
mig_namespace_limit: 10 7
...output omitted...
```

1

Número de CPU disponibles para el recurso personalizado (CR)MigrationController

2

Cantidad de memoria disponible para el CRMigrationController

3

Número de unidades de CPU disponibles para el CRMigrationController

4

Número de solicitudes de memoria disponibles para el CRMigrationController

5

Número de PV que se pueden migrar por plan de migración

6

Número de pods que se pueden migrar por plan de migración

7

Número de espacios de nombres que se pueden migrar por plan de migración


