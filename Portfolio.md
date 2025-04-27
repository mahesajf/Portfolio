# Deploy Netflix Clone, LanTicket, Mario on Kubernetes with Google Cloud Platform - DevOps Engineer Project!

### **Google Cloud Platform: Setup Google Kubernetes Engine (GKE) and Provisioning**

**Step 1 : Create Cluster and Nodes pool**

Number of Nodes : 2  
Total vCPUs : 4  
Total Memory : 4 GB  
<img src="../_resources/b6f73adbed3a5f444b39c9530b67c987.png" alt="b6f73adbed3a5f444b39c9530b67c987.png" width="800" height="386">

<img src="../_resources/396a1036f99c974ab01578504514e85a.png" alt="396a1036f99c974ab01578504514e85a.png" width="800" height="405">

  

**Step 2 : Create Virtual Machine (VM) for Provisioning**  
OS : Ubuntu  
Size : 100GB  
vCPU : 4 (2 core)  
RAM : 6 GB  
<img src="../_resources/5cad5b4915f0717b1093b6304441ae17.png" alt="5cad5b4915f0717b1093b6304441ae17.png" width="800" height="307">

  

**Step 3 : Create Docker images and Test Apps**  
Before build image, setup `src/config/api.js` on the `baseURL`  
<img src="../_resources/6588fe55109efa2b69d5adeb6dcbb7a6.png" alt="6588fe55109efa2b69d5adeb6dcbb7a6.png" width="800" height="218">  
Create `Dockerfile`

```
FROM node:latest
WORKDIR /app

COPY package*.json ./

RUN npm install

COPY . .
EXPOSE 5173

CMD ["npm", "run", "host", "--", "--host", "0.0.0.0"]

```

  

build image frontend `username/frontend-apps`  
<img src="../_resources/05028fd7dad3480f10478d4b3f502d1a.png" alt="05028fd7dad3480f10478d4b3f502d1a.png" width="800" height="427">

setup backend apps `.env`

```
PORT=5000
DB_HOST=db-lan-app  # Nama layanan yang mengekspos database PostgreSQL
DB_USER=postgres
DB_PASSWORD=postgres
DB_NAME=postgres
DB_PORT=5432  # Port yang digunakan oleh aplikasi di dalam container
```

Create `Dockerfile`

```
FROM golang:latest

RUN mkdir /app
COPY . /app
WORKDIR /app

RUN go get .
RUN go build
RUN go mod download

EXPOSE 5000
CMD ["go", "run", "main.go"]
```

build image backend `username/backend-apps`  
<img src="../_resources/97ac024a6e415be36e7f14fa6f7dea31.png" alt="97ac024a6e415be36e7f14fa6f7dea31.png" width="800" height="426">

upload docker images to docker hub

**Step 4 : Setup & Deploy Netflix, Mario, Tiket apps with Kubernetes**  
Set Mario and Netflix to Node Entertainment (tag node labels)  
<img src="../_resources/486375220a2c7bd72be4b2db48bb4d92.png" alt="486375220a2c7bd72be4b2db48bb4d92.png" width="800" height="279">  
Set Tiket apps to Node Tiket (tag node labels)  
<img src="../_resources/0a44fe2d3c47cc216f4bc2b12d7742e5.png" alt="0a44fe2d3c47cc216f4bc2b12d7742e5.png" width="800" height="263">  
Setup DNS domain and nginx with SSL (certbot)  
<img src="../_resources/719558f437be91627c0b5a8533f6c258.png" alt="719558f437be91627c0b5a8533f6c258.png" width="800" height="403">

Setup Jenkins CI/CD

<img src="../_resources/Screenshot%202025-04-06%20184116.png" alt="Screenshot 2025-04-06 184116.png" width="800" height="295">

<img src="../_resources/Screenshot%202025-04-06%20175314.png" alt="Screenshot 2025-04-06 175314.png" width="800" height="199">

**FINISHED**  
**Netflix**  
<img src="../_resources/76ad6e71503e9ffe163ce5db1cb0472f.png" alt="76ad6e71503e9ffe163ce5db1cb0472f.png" width="800" height="444">  
<img src="../_resources/eb0bd5b8fa56e392f52ec3fc1e6a0931.png" alt="eb0bd5b8fa56e392f52ec3fc1e6a0931.png" width="800" height="444">

  

**Tiket**  
<img src="../_resources/8dbb482da4f65591fff5f6f934713240.png" alt="8dbb482da4f65591fff5f6f934713240.png" width="800" height="444">  
<img src="../_resources/d5f5e612fd969aa384a0da4ee6a43da7.png" alt="d5f5e612fd969aa384a0da4ee6a43da7.png" width="800" height="445">

  

**Mario**  
<img src="../_resources/38993a7a54f0a1c900ba440d372855ba.png" alt="38993a7a54f0a1c900ba440d372855ba.png" width="800" height="449">  
<img src="../_resources/5bb54b867738dd43828903293c8291fd.png" alt="5bb54b867738dd43828903293c8291fd.png" width="800" height="446">

* * *