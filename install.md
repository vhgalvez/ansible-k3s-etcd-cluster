# Guía Completa para la Instalación de K3s en el Nodo Maestro Principal y la Adición de Nodos Secundarios y Trabajadores

Este documento proporciona una guía detallada para configurar el nodo maestro principal, seguido de la instalación de nodos maestros adicionales y trabajadores en un clúster de K3s.

## Paso 1: Instalar K3s en el Nodo Maestro Principal (Master Node 1)

Descargar y preparar el binario de K3s en el nodo maestro principal:

```bash
curl -Lo k3s https://github.com/k3s-io/k3s/releases/latest/download/k3s
chmod +x k3s
sudo mkdir -p /opt/bin
sudo mv k3s /opt/bin/
```

Configurar el servicio systemd para iniciar K3s en el nodo maestro principal: Crea el archivo de servicio `/etc/systemd/system/k3s.service`:

```bash
sudo tee /etc/systemd/system/k3s.service > /dev/null <<EOF
[Unit]
Description=Lightweight Kubernetes
Documentation=https://k3s.io
Wants=network-online.target
After=network-online.target

[Service]
Type=exec
ExecStart=/opt/bin/k3s server --cluster-init --datastore-endpoint=<ETCD_ENDPOINT>
Restart=on-failure
KillMode=process
Delegate=yes
LimitNOFILE=1048576
LimitNPROC=1048576
LimitCORE=infinity

[Install]
WantedBy=multi-user.target
EOF
```

Reemplaza `<ETCD_ENDPOINT>` con la URL de tu endpoint de etcd.

Habilitar e iniciar K3s en el nodo maestro principal:

```bash
sudo systemctl daemon-reload
sudo systemctl enable k3s
sudo systemctl start k3s
```

Verificar el estado de K3s:

```bash
sudo systemctl status k3s
```

## Paso 2: Obtener el Token de Autenticación y la IP del Nodo Maestro Principal

Obtener el token de K3s en el primer nodo maestro para agregar otros nodos al clúster:

```bash
sudo cat /var/lib/rancher/k3s/server/node-token
```

Copia el token para usarlo en la configuración de los nodos secundarios y trabajadores.

Verificar la IP del nodo maestro principal: La IP de Master Node 1 (IP de entrada del clúster) es `10.17.4.21`.

## Paso 3: Instalar K3s en los Nodos Maestros Secundarios (Master Node 2 y Master Node 3)

Para cada nodo maestro adicional, repite estos pasos:

Descargar y preparar el binario de K3s:

```bash
curl -Lo k3s https://github.com/k3s-io/k3s/releases/latest/download/k3s
chmod +x k3s
sudo mkdir -p /opt/bin
sudo mv k3s /opt/bin/
```

Crear el servicio systemd para unir el nodo maestro secundario al clúster: Configura `/etc/systemd/system/k3s.service`:

```bash
sudo tee /etc/systemd/system/k3s.service > /dev/null <<EOF
[Unit]
Description=Lightweight Kubernetes
Documentation=https://k3s.io
Wants=network-online.target
After=network-online.target

[Service]
Type=exec
ExecStart=/opt/bin/k3s server --server https://10.17.4.21:6443 --token <TOKEN> --datastore-endpoint=<ETCD_ENDPOINT>
Restart=on-failure
KillMode=process
Delegate=yes
LimitNOFILE=1048576
LimitNPROC=1048576
LimitCORE=infinity

[Install]
WantedBy=multi-user.target
EOF
```

Reemplaza `<TOKEN>` con el token copiado en el Paso 2 y `<ETCD_ENDPOINT>` con la URL de tu endpoint de etcd.

Habilitar e iniciar K3s en los nodos maestros secundarios:

```bash
sudo systemctl daemon-reload
sudo systemctl enable k3s
sudo systemctl start k3s
```

## Paso 4: Instalar K3s en los Nodos Trabajadores

En cada nodo trabajador (Worker 1, Worker 2, y Worker 3):

Descargar y preparar el binario de K3s:

```bash
curl -Lo k3s https://github.com/k3s-io/k3s/releases/latest/download/k3s
chmod +x k3s
sudo mkdir -p /opt/bin
sudo mv k3s /opt/bin/
```

Crear un archivo de servicio para el modo agent: Configura `/etc/systemd/system/k3s-agent.service`:

```bash
sudo tee /etc/systemd/system/k3s-agent.service > /dev/null <<EOF
[Unit]
Description=Lightweight Kubernetes Node
Documentation=https://k3s.io
After=network-online.target

[Service]
Type=exec
ExecStart=/opt/bin/k3s agent --server https://10.17.4.21:6443 --token <TOKEN>
Restart=always
LimitNOFILE=1048576
LimitNPROC=1048576
LimitCORE=infinity

[Install]
WantedBy=multi-user.target
EOF
```

Reemplaza `<TOKEN>` con el token obtenido en el Paso 2.

Habilitar e iniciar K3s-agent en cada nodo trabajador:

```bash
sudo systemctl daemon-reload
sudo systemctl enable k3s-agent
sudo systemctl start k3s-agent
```

## Paso 5: Configuración de kubectl para Uso sin sudo en los Nodos Maestros

Para permitir el uso de `kubectl` sin permisos elevados:

Modificar permisos del archivo de configuración `k3s.yaml`:

```bash
sudo chmod 644 /etc/rancher/k3s/k3s.yaml
```

Copiar el archivo de configuración a tu directorio de usuario:

```bash
mkdir -p ~/.kube
sudo cp /etc/rancher/k3s/k3s.yaml ~/.kube/config
sudo chown $(whoami):$(whoami) ~/.kube/config
```

Verificar que `kubectl` funcione sin sudo:

```bash
kubectl get nodes
```

Asegúrate de que el archivo `kubeconfig` esté correctamente configurado y apunte al servidor K3s correcto.

## Paso 6: Verificar el Estado del Clúster desde el Nodo Maestro Principal

Ejecuta el siguiente comando en el nodo maestro principal para ver el estado de todos los nodos:

```bash
kubectl get nodes
```

Si todo está bien, deberías ver todos los nodos maestros y trabajadores en estado `Ready`.

## Conclusión

Siguiendo estos pasos, habrás configurado correctamente el nodo maestro principal y añadido nodos maestros y trabajadores adicionales a tu clúster de K3s. Este procedimiento garantiza una configuración escalable y estable para tu clúster Kubernetes.
