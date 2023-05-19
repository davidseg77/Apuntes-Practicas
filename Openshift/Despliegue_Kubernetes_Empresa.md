# Info del curso DO380 de Red Hat

### Consulta de recursos de OpenShift

**Extracción de información de recursos**

El siguiente comando de ejemplo muestra cómo enumerar los recursos de un lugar específico ubicado dentro de un mediante el comando:typenamespaceoc get type -n namespace -o json

```
[user@host ~]$ oc get deployment -n openshift-cluster-samples-operator -o json
```

El comando siguiente muestra cómo obtener información sobre el "esquema" de un objeto o sus propiedades:

```
[user@host ~]$ oc explain deployment.status.replicas
```

**Extracción de información de un único recurso**

Utilice el comando para recuperar un único objeto en el espacio de nombres especificado. Además, utilice plantillas JSONPath para extraer y dar formato al contenido de la información consultada.oc get type name -n namespace

```
[user@host ~]$ oc get deployment -n openshift-cluster-samples-operator \
  cluster-samples-operator -o jsonpath='{.status.availableReplicas}'
```

Iterar sobre las listas del recurso mediante un índice comodín:[*]

```
[user@host ~]$ oc get deployment -n openshift-cluster-samples-operator \
  cluster-samples-operator -o jsonpath='{.status.conditions[*].type}'
```

Obtener un elemento específico en una lista utilizando la notación de indexación:[index]

```
[user@host ~]$ oc get deployment -n openshift-cluster-samples-operator \
  cluster-samples-operator -o jsonpath='{.spec.template.spec.containers[0].name}'
```

Filtrar elementos de una lista mediante una expresión de filtro:[?(condition)]

```
[user@host ~]$ oc get deployment -n openshift-cluster-samples-operator \
  cluster-samples-operator \
  -o jsonpath='{.status.conditions[?(@.type=="Available")].status}'
```

**Extraer una sola propiedad de varios recursos**

El comando devuelve un objeto JSON que contiene un objeto, que es una lista de todos los objetos consultados.oc getitems

A continuación se muestra un ejemplo de cómo enumerar una sola propiedad de muchos objetos:

```
[user@host ~]$ oc get route -n openshift-monitoring \
  -o jsonpath='{.items[*].spec.host}'
```

Se utiliza para imprimir propiedades específicas en formato tabular:oc get -o=custom-columns

```
[user@host ~]$ oc get pod --all-namespaces \
  -o=custom-columns=NAME:.metadata.name,STATUS:.status.phase,NODE:.spec.nodeName
```

**Extracción de una sola propiedad con varios anidamientos de recursos**

El comando devuelve una lista de pods. Suministro de iteraciones sobre cada pod Cada pod contiene un objeto anidado, que es una lista de los contenedores del pod. El uso itera sobre todos los contenedores de todos los pods.oc get pods.items[*]spec.containers[*]

```
[user@host ~]$ oc get pods -A \
  -o jsonpath='{.items[*].spec.containers[*].image}'
```

**Extracción de varias propiedades en diferentes niveles de anidamiento de recursos**

Utilice la instrucción para realizar iteraciones más complejas. La instrucción se ejecuta para cada elemento en hasta alcanzar el .{range}{range x}…​template…​{end}templatexend

Utilícelo como alternativa a .{range}.items[*].property

```
[user@host ~]$ oc get pods -A \
  -o jsonpath='{range .items[*]}' \
  '{.metadata.namespace} {.metadata.creationTimestamp}{"\n"}'
```

**Uso del filtrado de etiquetas**

Utilice esta opción para ver etiquetas con .--show-labelsoc get

```
[user@host ~]$ oc get deployment -n openshift-cluster-storage-operator \
  --show-labels
```

Utilice el argumento para filtrar por etiqueta:-l label

```
[user@host ~]$ oc get deployment -n openshift-cluster-storage-operator \
  -l app=csi-snapshot-controller-operator -o name
```

### Programación de OpenShift CronJobs

CronJobs crea trabajos basados en un horario determinado. Utilice CronJobs para ejecutar automatización, como copias de seguridad o informes, que deben ejecutarse en un intervalo regular.

Cuando se utiliza CronJobs, el cronograma de ejecución se especifica en el formato cron estándar.

```
apiVersion: batch/v1beta1
kind: CronJob
metadata:
  name: backup-cron
  namespace: database
spec:
  schedule: "1 0 * * *" 1
  jobTemplate: 2
    spec:
      activeDeadlineSeconds: 600
      parallelism: 2
      template:
        metadata:
          name: backup
        spec:
          serviceAccountName: backup_sa
          containers:
          - name: backup
            image: example/backup_maker:v1.2.3
          restartPolicy: OnFailure
```

1

La programación de ejemplo ejecuta el trabajo un minuto después de la medianoche.

2

Especifique un trabajo normal en el campo CronJob.jobTemplate

### Ejercicio guiado: Navegar por la API REST de OpenShift

**Intente una solicitud GET al servidor API de OpenShift sin autenticación.**

```
[student@workstation ~]$ curl -k https://api.ocp4.example.com:6443/api
```

**Autentíquese con el servidor OAuth de OpenShift para recuperar un token de acceso.**

Inicie sesión en OpenShift como usuario.admin

```
[student@workstation ~]$ oc login -u admin -p redhat \
  https://api.ocp4.example.com:6443
```

Busque la ruta al servidor OAuth de OpenShift mediante el comando y, a continuación, asígnela a una variable.oc

```
[student@workstation ~]$ oc get route -n openshift-authentication
NAME              HOST/PORT                               PATH   SERVICES          PORT   TERMINATION            WILDCARD
oauth-openshift   oauth-openshift.apps.ocp4.example.com          oauth-openshift   6443   passthrough/Redirect   None

[student@workstation ~]$ OAUTH_HOST=$(oc get route oauth-openshift \
  -n openshift-authentication -o jsonpath='{.spec.host}')

[student@workstation ~]$ echo $OAUTH_HOST
oauth-openshift.apps.ocp4.example.com
```

Autentíquese con el servidor OAuth mediante el comando. OpenShift responde con un código de respuesta HTTP 302 Redirect, pero no hay necesidad de seguirlo. El token se puede encontrar como parámetro en la URL.curlaccess_tokenLocation

```
[student@workstation ~]$ curl -u admin -kv "https://$OAUTH_HOST/oauth/\
  authorize?client_id=openshift-challenging-client&response_type=token"
```

Guarde el token en una variable.

```
[student@workstation ~]$ TOKEN=sha256~xvZ8SsTiA3jRIiEX9QMUOLdaZRUPqubLy2AiQbQGDb0
```

**Intente una solicitud GET al servidor API de OpenShift utilizando el token de acceso.**

```
[student@workstation ~]$ HEADER="Authorization: Bearer $TOKEN"
[student@workstation ~]$ curl -k --header "$HEADER" \
  -X GET https://api.ocp4.example.com:6443/
```

**Recorra el grupo de API mediante solicitudes.v1curl**

Obtenga la lista de recursos de API disponibles en el grupo de API.v1

```
[student@workstation ~]$ curl -k --header "$HEADER" \
  -X GET https://api.ocp4.example.com:6443/api/
```

A continuación, obtenga una lista de uno de los recursos, como .pods

```
[student@workstation ~]$ curl -k --header "$HEADER" \
  -X GET https://api.ocp4.example.com:6443/api/v1/pods
```

Filtre los resultados para enumerar sólo los nombres que incluyen .jq

```
[student@workstation ~]$ curl -sk --header "$HEADER" \
  -X GET https://api.ocp4.example.com:6443/api/v1/pods/ \
  | jq ".items[].metadata.name"
```

**Busque el punto de enlace de la API para los recursos mediante el comando y, a continuación, obtenga una lista de todos los hosts que usan el comando.Routeoccurl**

Utilice el comando para detectar el recurso y .oc explainKindAPI Version

```
[student@workstation ~]$ oc explain routes
```

Enumere las API disponibles para el grupo de versiones de API.route.openshift.io/v1

```
[student@workstation ~]$ curl -k --header "$HEADER" \
  -X GET https://api.ocp4.example.com:6443/apis/route.openshift.io/v1
```

Enumere los hosts para todas las rutas utilizando los comandos and.curljq

```
[student@workstation ~]$ curl -sk --header "$HEADER" \
  -X GET https://api.ocp4.example.com:6443/apis/route.openshift.io/v1/routes \
  | jq '.items[].spec.host'
```

### Operadores de instalación

**Instalación de un operador desde OLM mediante la CLI**

Consulte la lista de operadores disponibles.

```
[user@host ~]$ oc get packagemanifests
```

Inspeccione a un operador. Cada operador agrega nuevos tipos de recursos mediante definiciones de recursos personalizadas (CRD).

```
[user@host ~]$ oc describe packagemanifests file-integrity-operator
```

En la CLI, no hay una forma directa de obtener los canales del operador, por lo que necesita una consulta JSONPath.

```
[user@host ~]$ oc get packagemanifests file-integrity-operator \
  --output='jsonpath={range .status.channels[*]}{.name}{"\n"}{end}'
```

**Verificación del estado del operador**

Hay varias maneras de comprobar el estado del operador. El resto de esta sección proporciona información adicional acerca de la inspección de registros de eventos, objetos de suscripción e información de espacio de nombres para comprobar el estado de un operador.

Compruebe los registros del pod OLM para verificar la instalación del operador.

```
[user@host ~]$ oc logs pod/olm-operator-c5599dfd7-nknfx \
  -n openshift-operator-lifecycle-manager
```

Inspeccione el objeto de suscripción para comprobar el estado del operador.

```
[user@host ~]$ oc describe sub <subscription-name> -n <namespace>
...output omitted...
```

**Listar todos los operadores**

Enumere los operadores instalados comprobando las versiones del servicio de clúster. Para enumerar todos los operadores instalados, obtenga las versiones del servicio de clúster en todos los espacios de nombres.

```
[user@host ~]$ oc get csv -A
```

Los operadores se pueden suscribir a un espacio de nombres o a todos los espacios de nombres. Para enumerar los operadores administrados por OLM, enumere las suscripciones activas.

```
[user@host ~]$ oc get subs -A
```

Para ver el estado y los eventos de los recursos personalizados relacionados con un operador determinado, describa la implementación del operador.

```
[user@host ~]$ oc describe deployment.apps/file-integrity-operator | \
  grep -i kind
```

Compruebe los elementos del operador obteniendo todos los objetos del espacio de nombres del operador. Si el operador está instalado en todos los espacios de nombres, realice la consulta en el espacio de nombres y busque el nombre del operador para descubrir los elementos.openshift-operators

```
[user@host ~]$ oc get all -n openshift-file-integrity
```

Utilice el comando para ver eventos y registros de un operador.oc logs

```
[user@host ~]$ oc logs deployment.apps/file-integrity-operator
```

**Actualización de un operador desde OLM mediante la CLI**

Modifique el archivo YAML del operador y aplique los cambios deseados al objeto de suscripción.

```
[user@host ~]$ oc apply -f file-integrity-operator-subscription.yaml
```

**Eliminación de operadores**

Para quitar un operador del clúster, elimine los objetos de suscripción y versión del servicio de clúster.

Compruebe la versión actual del operador suscrito en el campo.currentCSV

```
[user@host ~]$ oc get sub <subscription-name> -o yaml | grep currentCSV
currentCSV: ...output omitted...
```

Elimine el objeto de suscripción. Utilice el valor obtenido del comando anterior para eliminar el objeto de versión del servicio de Cluster Server.

```
[user@host ~]$ oc delete sub <subscription-name>
[user@host ~]$ oc delete csv <currentCSV>
```

### Implementación de Jenkins en OpenShift mediante plantillas estándar

Las plantillas estándar definen parámetros y valores predeterminados para el tamaño de volumen persistente y los requisitos de memoria, entre otros. Las instancias pequeñas de Jenkins no requieren cambios en estos valores de parámetros predeterminados y se pueden implementar creando un proyecto y, a continuación, ejecutando:

```
[user@host ~]$ oc new-app --template jenkins-persistent
```

**Generación de tokens de API de Jenkins**

Para usar la API de Jenkins o la CLI de Jenkins, necesita un token de API. Ese token reemplaza la contraseña en un encabezado de autenticación HTTP BASIC, por ejemplo:

```
[user@host ~] curl --user 'jenkins-user:token' \
  https://jenkins-host/resource-path
```

