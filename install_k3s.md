**Guía Completa para la Configuración de K3s en Flatcar Container Linux**

Esta guía cubre la instalación y configuración de K3s en Flatcar Container Linux, usando etcd como backend de datos y adaptando el uso de kubectl en un sistema de solo lectura. Además, se incluye la configuración de nodos adicionales y la estructura completa del clúster.

### Paso 1: Instalación de K3s en el Nodo Maestro Principal

#### 1.1 Descargar y Preparar el Binario de K3s
Ejecuta los siguientes comandos en el nodo maestro principal para descargar y preparar K3s:

```bash
curl -Lo k3s https://github.com/k3s-io/k3s/releases/latest/download/k3s
chmod +x k3s
sudo mkdir -p /opt/bin
sudo mv k3s /opt/bin/
```

#### 1.2 Crear el Archivo de Servicio k3s.service
Para configurar K3s como servidor y usar etcd, crea el archivo de servicio con el siguiente contenido:

```bash
sudo tee /etc/systemd/system/k3s.service > /dev/null <<EOF
[Unit]
Description=Lightweight Kubernetes
Documentation=https://k3s.io
Wants=network-online.target
After=network-online.target

[Service]
Type=exec
ExecStart=/opt/bin/k3s server --cluster-init --datastore-endpoint='etcd://<etcd-endpoint>:2379'
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

Nota: Reemplaza `<etcd-endpoint>` con la IP del servidor etcd.

#### 1.3 Iniciar y Verificar el Estado del Servicio K3s
Recarga el demonio de systemd, habilita y arranca el servicio de K3s:

```bash
sudo systemctl daemon-reload
sudo systemctl enable k3s
sudo systemctl start k3s
sudo systemctl status k3s
```

#### 1.4 Obtener el Token de Autenticación
Ejecuta el siguiente comando para obtener el token necesario para unir otros nodos al clúster:

```bash
sudo cat /var/lib/rancher/k3s/server/node-token
```

Nota: Copia este token para usarlo en la configuración de los nodos adicionales.

### Paso 2: Configuración de Nodos Maestros Secundarios

#### 2.1 Instalar K3s y Configurar el Archivo de Servicio
En cada nodo maestro adicional, realiza los siguientes pasos para instalar y configurar K3s:

```bash
curl -Lo k3s https://github.com/k3s-io/k3s/releases/latest/download/k3s
chmod +x k3s
sudo mkdir -p /opt/bin
sudo mv k3s /opt/bin/
```

Crea el archivo de servicio en `/etc/systemd/system/k3s.service`:

```bash
sudo tee /etc/systemd/system/k3s.service > /dev/null <<EOF
[Unit]
Description=Lightweight Kubernetes
Documentation=https://k3s.io
Wants=network-online.target
After=network-online.target

[Service]
Type=exec
ExecStart=/opt/bin/k3s server --server https://10.17.4.21:6443 --token <TOKEN>
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

Nota: Reemplaza `<TOKEN>` con el token obtenido en el Paso 1.4.

#### 2.2 Iniciar y Verificar el Servicio en Nodos Maestros Secundarios
Recarga el demonio de systemd, habilita y arranca el servicio de K3s en cada nodo maestro:

```bash
sudo systemctl daemon-reload
sudo systemctl enable k3s
sudo systemctl start k3s
sudo systemctl status k3s
```

### Paso 3: Configuración de Nodos Trabajadores

#### 3.1 Descargar y Preparar el Binario de K3s
Ejecuta los siguientes comandos en cada nodo trabajador para descargar y configurar K3s:

```bash
curl -Lo k3s https://github.com/k3s-io/k3s/releases/latest/download/k3s
chmod +x k3s
sudo mkdir -p /opt/bin
sudo mv k3s /opt/bin/
```

#### 3.2 Crear el Archivo de Servicio k3s-agent.service
Crea el archivo `/etc/systemd/system/k3s-agent.service` con el siguiente contenido:

```bash
sudo tee /etc/systemd/system/k3s-agent.service > /dev/null <<EOF
[Unit]
Description=Lightweight Kubernetes Node
Documentation=https://k3s.io
After=network-online.target

[Service]
Type=simple
ExecStart=/opt/bin/k3s agent --server https://10.17.4.21:6443 --token <TOKEN>
Restart=always
LimitNOFILE=1048576
LimitNPROC=1048576
LimitCORE=infinity

[Install]
WantedBy=multi-user.target
EOF
```

Nota: Reemplaza `<TOKEN>` con el token obtenido en el Paso 1.4.


```bash
sudo tee /etc/systemd/system/k3s-agent.service > /dev/null <<EOF
[Unit]
Description=Lightweight Kubernetes Node
Documentation=https://k3s.io
After=network-online.target

[Service]
Type=simple
ExecStart=/opt/bin/k3s agent --server https://10.17.4.21:6443 --token K10d65b4c3c09dd1384773f15e3f8ae91d3bbeac43aeb24b5038c1fbe23431a1d0d::server:db1ca19be34084bf1536fa068453ad63
Restart=always
LimitNOFILE=1048576
LimitNPROC=1048576
LimitCORE=infinity

[Install]
WantedBy=multi-user.target
EOF
```


#### 3.3 Iniciar y Verificar el Servicio en Nodos Trabajadores
Recarga el demonio de systemd, habilita y arranca el servicio de K3s agent en cada nodo trabajador:

```bash
sudo systemctl daemon-reload
sudo systemctl enable k3s-agent
sudo systemctl start k3s-agent
sudo systemctl status k3s-agent
```

### Paso 4: Configuración de kubectl en Flatcar Container Linux
Flatcar Container Linux tiene un sistema de archivos de solo lectura, por lo que necesitamos crear un alias para usar kubectl.

#### 4.1 Copiar el Archivo de Configuración de K3s
Copia el archivo de configuración de K3s a tu directorio de usuario para usarlo con kubectl:

la copia de `/etc/rancher/k3s/k3s.yaml` del nodo master1 tiene que hacer para los otros nodos funcionen


```bash
mkdir -p ~/.kube
sudo cp /etc/rancher/k3s/k3s.yaml ~/.kube/config
sudo chown $(whoami):$(whoami) ~/.kube/config
```

#### 4.2 Crear un Alias para kubectl
Edita el archivo de configuración de tu shell (por ejemplo, `~/.bashrc`) y añade el siguiente alias:

```bash
alias kubectl='sudo k3s kubectl'
```

Recarga el archivo de configuración del shell para activar el alias:

```bash
source ~/.bashrc
```

#### 4.3 Verificar el Alias de kubectl
Ahora deberías poder utilizar kubectl directamente:

```bash
kubectl get nodes
```

### Paso 5: Verificar el Estado del Clúster
Desde el nodo maestro principal, verifica el estado de todos los nodos en el clúster:

```bash
kubectl get nodes
```

Si todo está correctamente configurado, deberías ver todos los nodos maestros y trabajadores en estado **Ready**.

### Descripción de la Configuración del Clúster

**Nodos Maestros**
Los nodos maestros administran el clúster y mantienen el estado a través de etcd:
- **Master Node 1 (IP: 10.17.4.21)** - Inicializa el clúster y configura etcd.
- **Master Node 2 (IP: 10.17.4.22)** - Nodo maestro adicional que también participa en etcd.
- **Master Node 3 (IP: 10.17.4.23)** - Otro nodo maestro que garantiza alta disponibilidad.

**Nodos Trabajadores**
Los nodos trabajadores ejecutan las aplicaciones y contenedores desplegados en el clúster:
- **Worker Node 1 (IP: 10.17.4.24)**
- **Worker Node 2 (IP: 10.17.4.25)**
- **Worker Node 3 (IP: 10.17.4.26)**

Este diseño de clúster con K3s asegura alta disponibilidad y redundancia al utilizar etcd en cada nodo maestro.

---

Siguiendo esta guía, habrás configurado un clúster de K3s en Flatcar Container Linux, usando etcd como backend de datos y con kubectl adaptado para un sistema de solo lectura.




