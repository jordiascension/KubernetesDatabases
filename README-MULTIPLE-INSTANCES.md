INSTALACIÓN DE MÚLTIPLES INSTANCIAS

Importante

Esto no es un cluster multi-primary. Es una arquitectura:

1 primary
2 secondary

Por tanto:

Escrituras -> primary
Lecturas   -> primary o secondary

Si el primary cae, dependiendo de la versión/configuración del chart, puede que necesites intervención manual o configuración adicional para failover automático. Para alta disponibilidad real con failover automático, normalmente se usa MySQL InnoDB Cluster, MySQL Operator, Percona XtraDB Cluster o MariaDB Galera.

La configuración 1 primary + N réplicas es muy común cuando una aplicación tiene muchas más lecturas que escrituras. Por ejemplo:

Escrituras  -> MySQL primary
Lecturas    -> MySQL replicas/secondary

Se usa mucho en:

APIs con mucho tráfico de lectura
dashboards
catálogos
CMS
reporting
backoffice
consultas pesadas de solo lectura

Sí. Con el chart de Bitnami puedes pasar de una sola instancia a una arquitectura con 1 primary + 2 secondary, es decir, 3 pods de MySQL.

La idea sería:

db2-mysql-primary-0      -> escritura y lectura
db2-mysql-secondary-0    -> réplica de solo lectura
db2-mysql-secondary-1    -> réplica de solo lectura
Values.yaml para 3 instancias

Usa este fichero:

-ValuesReplicator.yaml

global:
  security:
    allowInsecureImages: true

image:
  registry: public.ecr.aws
  repository: bitnami/mysql
  tag: 8.4

architecture: replication

auth:
  rootPassword: "rootpassword"
  database: "my_database"
  replicationUser: "replicator"
  replicationPassword: "replicatorpassword"

primary:
  persistence:
    enabled: true
    size: 8Gi

secondary:
  replicaCount: 2
  persistence:
    enabled: true
    size: 8Gi


Instalación limpia

Como vas a cambiar de una instancia simple a replicación, te recomiendo borrar el release y los PVC antiguos si no necesitas conservar datos:

helm uninstall db2
kubectl delete pvc data-db2-mysql-0 --ignore-not-found
kubectl delete pvc data-db2-mysql-primary-0 --ignore-not-found
kubectl delete pvc data-db2-mysql-secondary-0 --ignore-not-found
kubectl delete pvc data-db2-mysql-secondary-1 --ignore-not-found

Instala:

helm install db2 bitnami/mysql -f ValuesReplicator.yaml
Comprobar pods
kubectl get pods

Deberías ver algo parecido a:
db2-mysql-primary-0     1/1   Running
db2-mysql-secondary-0   1/1   Running
db2-mysql-secondary-1   1/1   Running
Comprobar PVCs
kubectl get pvc

Deberías ver un volumen por instancia:

data-db2-mysql-primary-0
data-db2-mysql-secondary-0
data-db2-mysql-secondary-1

Conexión al primary

Para escritura, usa el primary:
kubectl exec -it db2-mysql-primary-0 -- mysql -uroot -prootpassword

Prueba:

CREATE DATABASE prueba_cluster;
SHOW DATABASES;
Comprobar replicación en las réplicas

Entra en una secundaria:

kubectl exec -it db2-mysql-secondary-0 -- mysql -uroot -prootpassword

Y comprueba:

SHOW DATABASES;

Deberías ver también:

prueba_cluster
Servicios que tendrás

Mira los servicios:

kubectl get svc

Normalmente tendrás algo similar a:

db2-mysql-primary
db2-mysql-secondary
db2-mysql-headless

Usa:
db2-mysql-primary

para escritura.

Usa:

db2-mysql-secondary

para lecturas distribuidas entre las réplicas.

Importante

Esto no es un cluster multi-primary. Es una arquitectura:

1 primary
2 secondary

Por tanto:

Escrituras -> primary
Lecturas   -> primary o secondary

Si el primary cae, dependiendo de la versión/configuración del chart, puede que necesites intervención manual o configuración adicional para failover automático. Para alta disponibilidad real con failover automático, normalmente se usa MySQL InnoDB Cluster, MySQL Operator, Percona XtraDB Cluster o MariaDB Galera

Sí, se utiliza bastante, pero con matices.

La configuración 1 primary + N réplicas es muy común cuando una aplicación tiene muchas más lecturas que escrituras. Por ejemplo:

Escrituras  -> MySQL primary
Lecturas    -> MySQL replicas/secondary

Se usa mucho en:

APIs con mucho tráfico de lectura
dashboards
catálogos
CMS
reporting
backoffice
consultas pesadas de solo lectura

La ventaja principal es que descargas trabajo del nodo principal. El primary se encarga de escribir, y las réplicas atienden lecturas.

Pero hay una cosa importante: las réplicas pueden ir un poco por detrás. Es decir, puedes insertar un dato en el primary y que tarde unos milisegundos o segundos en aparecer en una secondary. A eso se le llama replication lag.
