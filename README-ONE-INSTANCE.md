INSTALACIÓN DE UNA ÚNICA INSTANCIA

Acepto usar una imagen personalizada que el chart no reconoce como la oficial esperada.

No significa que MySQL sea automáticamente inseguro. Significa que Bitnami no puede garantizar que esa imagen personalizada sea exactamente la que el chart fue diseñado para usar.

En nuestro caso lo usamos porque:

public.ecr.aws/bitnami/mysql:latest  -> traía MySQL 9.7.0 y fallaba
docker.io/bitnami/mysql antiguo      -> no se podía descargar
public.ecr.aws/bitnami/mysql:8.4     -> sí descarga y funciona

Por eso el Values.yaml funcional queda así:

-Values.yaml

global:
  security:
    allowInsecureImages: true

image:
  registry: public.ecr.aws
  repository: bitnami/mysql
  tag: 8.4

auth:
  rootPassword: "rootpassword"
  database: "my_database"

primary:
  persistence:
    enabled: true
    size: 8Gi


Instalación limpia:

helm uninstall db2
kubectl delete pod db2-mysql-0 --force --grace-period=0 --ignore-not-found
kubectl delete pvc data-db2-mysql-0 --ignore-not-found

helm install db2 bitnami/mysql -f Values.yaml

Comprobar logs:

kubectl logs db2-mysql-0 -c mysql -f

Resultado correcto:

** MySQL setup finished! **
ready for connections

Comprobar pod y PVC:

kubectl get pods
kubectl get pvc

Conectarte:

kubectl exec -it db2-mysql-0 -- mysql -uroot -prootpassword