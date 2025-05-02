# 🛠️ Configuración de Clusters de BD en Mongo DB con docker compose en React

Este proyecto detalla cómo configurar un **Replica Set** de MongoDB en dos computadoras utilizando Docker y autenticación con un **keyfile**. Los pasos están diseñados para ser fáciles de seguir y asegurar que ambos nodos estén sincronizados de manera eficiente.

## 📋 Requisitos

Antes de comenzar, asegúrate de tener lo siguiente:

- Docker y Docker Compose instalados en ambas PCs.
- MongoDB versión 6 o superior.
- Acceso de red entre **PC1** y **PC2** (verifica con `ping` o `telnet`).
  
## 🚀 Pasos para la configuración

### 1. Generar y transferir el **keyFile**

Primero, generamos un archivo de clave para asegurar la autenticación entre los nodos.

#### En **PC1**:

```bash
openssl rand -base64 756 > mongodb-keyfile
chmod 400 mongodb-keyfile

```
> 📝 **Nota:** El archivo generado en **PC1** debe ser enviado a **PC2**.  
> Una vez copiado, en **PC2** ejecuta el siguiente comando para otorgar los permisos adecuados:

```bash
chmod 400 mongodb-keyfile
```
### 🧩 2. Configurar el `docker-compose.yml` en PC1 y PC2

En ambas computadoras, debes agregar la configuración del **Replica Set** y el archivo **keyFile** en el archivo `docker-compose.yml`.

#### 📦 En **PC1**:

```yaml
services:
  mongo:
    image: mongo:6
    container_name: mongo1
    ports:
      - "27019:27017"
    volumes:
      - ./mongodb-keyfile:/data/configdb/mongodb-keyfile
    command: ["mongod", "--replSet", "rs0", "--keyFile", "/data/configdb/mongodb-keyfile"]
```
#### 📦 En **PC2**:

```yaml
services:
  mongo:
    image: mongo:6
    container_name: mongo2
    ports:
      - "27019:27017"
    volumes:
      - ./mongodb-keyfile:/data/configdb/mongodb-keyfile
    command: ["mongod", "--replSet", "rs0", "--keyFile", "/data/configdb/mongodb-keyfile"]
```
### 3️⃣ Levantar los contenedores de MongoDB

Una vez que hayas configurado el archivo `docker-compose.yml` en ambas PCs, ejecuta el siguiente comando en **ambas máquinas** para levantar los contenedores:

```bash
docker-compose up -d
```
### 4️⃣ Iniciar el Replica Set desde PC1

En **PC1**, ingresa al contenedor de MongoDB ejecutando el siguiente comando:

```bash
docker exec -it chatdb2 mongosh --username admin --password password --authenticationDatabase admin
```
### 5️⃣ Inicializar el Replica Set

Una vez dentro del shell de MongoDB en **PC1**, ejecuta el siguiente comando para iniciar el Replica Set y agregar ambos nodos:

```js
rs.initiate({
  _id: "rs0",
  members: [
    { _id: 0, host: "192.168.138.38:27019" },  // IP de PC1
    { _id: 1, host: "192.168.138.207:27019" }  // IP de PC2
  ]
});
```
### 6️⃣ Verificar conectividad entre nodos

Desde **PC1**, ejecuta el siguiente comando en la terminal para verificar que puedes conectarte al puerto del contenedor de MongoDB en **PC2**:

```bash
telnet 192.168.138.207 27019
```
