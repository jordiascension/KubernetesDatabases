# Crear y comprobar un Pod de MySQL en Kubernetes

## 1. Crear el archivo `mysql-pod.yaml`

Crea un archivo llamado `mysql-pod.yaml` con el siguiente contenido:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: mysql-pod
  labels:
    app: mysql
spec:
  containers:
    - name: mysql
      image: mysql:8.0
      ports:
        - containerPort: 3306
      env:
        - name: MYSQL_ROOT_PASSWORD
          value: root1234
```

## 2. Aplicar el manifiesto

```bash
kubectl apply -f mysql-pod.yaml
```

## 3. Comprobar que el Pod se ha creado

```bash
kubectl get pods
```

Resultado esperado:

```text
NAME        READY   STATUS    RESTARTS   AGE
mysql-pod   1/1     Running   0          1m
```

Puede tardar unos segundos o minutos en pasar a `Running`.

## 4. Ver los logs de MySQL

```bash
kubectl logs mysql-pod
```

Debes ver mensajes de inicialización de MySQL.

Para ver los logs en tiempo real:

```bash
kubectl logs -f mysql-pod
```

## 5. Conectarse al contenedor

```bash
kubectl exec -it mysql-pod -- bash
```

Una vez dentro del contenedor:

```bash
mysql -u root -p
```

Cuando pida la contraseña, introduce:

```text
root1234
```

## 6. Comprobar las bases de datos

Dentro de MySQL, ejecuta:

```sql
SHOW DATABASES;
```
