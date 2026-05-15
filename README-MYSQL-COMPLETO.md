# Comandos Kubernetes para MySQL

## Aplicar el manifiesto

```bash
kubectl apply -f mysql-completo.yaml
```

---

## Comprobar recursos creados

```bash
kubectl get all -n curso-kubernetes
```

```bash
kubectl get pv
```

```bash
kubectl get pvc -n curso-kubernetes
```

```bash
kubectl get pods -n curso-kubernetes
```

```bash
kubectl get secrets -n curso-kubernetes
```

---

## Ver logs de MySQL

```bash
kubectl logs mysql-statefulset-0 -n curso-kubernetes
```

---

## Conectarse desde un cliente MySQL en el mismo namespace

```bash
kubectl run mysql-client \
  --rm -it \
  --image=mysql:8.0 \
  -n curso-kubernetes \
  -- bash
```

Dentro del contenedor:

```bash
mysql -h mysql-service-pvc -u root -proot1234
```

Contraseña:

```text
root1234
```

---

## Conectarse usando el DNS completo del servicio

```bash
mysql -h mysql-service-pvc.curso-kubernetes.svc.cluster.local -u root -p
```

---

## Borrar todo el entorno

```bash
kubectl delete namespace curso-kubernetes
```
