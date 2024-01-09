
This documentation will guide you through setting up Docker containers for frontend, backend, and database, as well as configuring them using a docker-compose.yml file.

# Setup Instructions

### Docker and git installation:-

To install Docker on Debian, you can follow these general steps:

1.  Update the package repository to make sure you have the latest package information:
    

```
sudo apt update
```

2.  Install necessary packages to allow apt to use repositories over HTTPS:
    

```
sudo apt install -y \
    apt-transport-https \
    ca-certificates \
    curl \
    software-properties-common
```

3.  Add Docker's official GPG key to ensure package integrity:
    

```
curl -fsSL <https://download.docker.com/linux/debian/gpg> | sudo gpg --dearmor -o /usr/share/keyrings/docker-archive-keyring.gpg
```

4.  Add the Docker repository to your system:
    

```
echo "deb [arch=amd64 signed-by=/usr/share/keyrings/docker-archive-keyring.gpg] <https://download.docker.com/linux/debian> $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
```

5.  Update the package database again to include Docker's repository:
    

```
sudo apt update
```

6.  Install Docker:
    

```
sudo apt install -y docker-ce docker-ce-cli containerd.io
```

7.  Start and enable the Docker service:
    

```
sudo systemctl start docker
sudo systemctl enable docker
```

8.  Verify that Docker is installed and running by the following command:
    

```
sudo docker --version
```

9.  Check docker status
    

```
systemctl status docker
```

10.  To install Git on Debian-based systems, you can use the apt package manager. Here are the steps:
    
11.  Update the package list to ensure you have the latest information on available packages:
    

```
sudo apt update
```

12.  Install Git by running the following command:
    

```
sudo apt install git
```

13.  Confirm the installation when prompted by typing "Y" and pressing Enter.
    
14.  Once the installation is complete, you can verify that Git is installed by checking its version:
```
git --version
```
This command should display the Git version that was installed.

15.  Install postgresql by running the following command:
```
sudo apt install postgresql postgresql-contrib
```
16. Start and Enable PostgreSQL Service
```
sudo systemctl start postgresql
```
To ensure that PostgreSQL starts on boot, you can enable the service:
```
sudo systemctl enable postgresql
```
17. check the status of postgresql
```
sudo systemctl status postgresql
```
18.Access PostgreSQL
```
psql postgresql://postgres:abcd1234@flask_dbcolor_vision_test_db:5432/color_vision
```

Clone the repository containing the code for frontend and backend applications. the repository has two folders: docker-frontend and docker-backend.

###   
  
Create a directory

```
mkdir vision_test
cd vision_test
```

# Setup Frontend

### Create a directory for frontend in “msm-docker “.

```
mkdir docker-frontend
```

```
cd docker-frontend
git clone git@github.com frontend
```

### Create docker image

In the docker-frontend folder, create Docker images using a Dockerfile.  

```
cd ../docker-frontend
```

Create a Dockerfile for frontend application Inside docker-frontend directory  

```
vim Dockerfile
```

```
FROM node:16-slim as artifact
ENV REACT_APP_BASE_URL=http://54.156.163.125:4000
WORKDIR /app
COPY ./frontend /app
RUN npm install 
RUN npm run build

FROM nginx:stable-alpine-slim
COPY --from=artifact /app/build /usr/share/nginx/html
COPY ./nginx.conf /etc/nginx/.
COPY ./waggonerhire /etc/nginx/conf.d/default.conf
EXPOSE 80
CMD ["nginx", "-g", "daemon off;"]
```

### Create nginx.conf 

```
vim nginx.conf
```

```
user nginx;
worker_processes auto;
pid /run/nginx.pid;
include /etc/nginx/modules-enabled/*.conf;

events {
	worker_connections 768;
	multi_accept on;
}

http {

	sendfile on;
	tcp_nopush on;
	tcp_nodelay on;
	keepalive_timeout 65;
	types_hash_max_size 2048;
	
	include /etc/nginx/mime.types;
	default_type application/octet-stream;

	ssl_protocols TLSv1 TLSv1.1 TLSv1.2 TLSv1.3; # Dropping SSLv3, ref: POODLE
	ssl_prefer_server_ciphers on;

	access_log /var/log/nginx/access.log;
	error_log /var/log/nginx/error.log;

	gzip on;

	include /etc/nginx/conf.d/*.conf;
	include /etc/nginx/sites-enabled/*;
        # include /etc/nginx/sites-available/*;

}
```
### Create waggonerhire
```bash
vim waggonerhire
```
insert the following data on it.
```
server {
    listen 80;
    listen [::]:80;
    root /usr/share/nginx/html;

    location / {
        index index.html;
        try_files $uri $uri/ /index.html;
    }

    location /healthcheck {
	return 200;	
    }
}
```

**In the docker-frontend folder, build the Dockerimage using a Dockerfile.**

```bash
# Inside docker-frontend directory
docker build --no-cache -t vision-backend .
```

# Setup Backend  

### Create a directory for Backend in “vision_test“.

```
mkdir docker-backend
```

```
cd docker-backend
git clone git@github.com backend
```

### Create docker image

In the docker-backend folder, create a Dockerfile.

```
cd ../docker-backend
```

```
vim Dockerfile
```

```
FROM python:3.9
WORKDIR /app
COPY ./backend/* /app
RUN pip install -r requirements.txt
EXPOSE 4000
CMD [ "flask", "run", "--host=0.0.0.0", "--port=4000"]
```

### **In the docker-backend folder, build a Dockerimage using a Dockerfile.**

```bash
# Inside docker-backend directory
docker build --no-cache -t vision-backend .
```

# **Check docker images**

```bash
docker images
```

### Set up docker compose

Create a docker-compose.yml file in “**vision_test**” folder with the following content:

```bash
version: '3.8'

services:
  frontend:
    container_name: vision-frontend
    image: vision-frontend
    restart: unless-stopped
    ports:
      - "80:80"
    environment:
      - REACT_APP_BASE_URL="http://vision-backend:4000"

  backend:
    container_name: vision-backend
    image: vision-backend
    restart: unless-stopped
    ports:
      - "4000:4000"
    environment:
      - DB_URL=postgresql://postgres:abcd1234@flask_dbcolor_vision_test_db:5432/color_vision
    depends_on:
      - flask_dbcolor_vision_test_db
        
  flask_dbcolor_vision_test_db:
    container_name: flask_dbcolor_vision_test_db
    image: postgres:12
    ports:
      - "5432:5432"
    environment:
      - POSTGRES_PASSWORD=abcd1234
      - POSTGRES_USER=postgres
      - POSTGRES_DB=color_vision
    volumes:
      - pgdata:/var/lib/postgresql/data

volumes:
  pgdata: {}
```

### Run the Docker Containers

In the project directory, run the following command to start the Docker containers:

```
docker compose -f docker-compose.yaml up -d
```

The -d flag runs the containers in the background.

### **Check running docker containers**  

```
docker ps -a
```
![image](https://github.com/akii2137/akii2137/assets/130637600/bdfeb447-85d3-46d9-a926-d8ffce9bcafd)


**On the server where Docker is running, navigate to the respective directory and pull the latest changes:**

```bash
# For frontend
cd /home/ubuntu/vision_test/docker-frontend/frontend
git fetch origin main
git pull origin main

# For backend
cd /home/ubuntu/vision_test/docker-backend/backend
git fetch origin main
git pull origin main
```

### Stop and Remove Containers

To stop and remove the Docker containers,run this following commands.

```
docker compose -f docker-compose.yaml down
docker compose -f docker-compose.yaml up -d
```

That's it! You now have a Dockerized setup for your frontend, backend, and database, along with instructions for updating your code and managing the containers.

----------
# The hierarchy of a directory tree
![image](https://github.com/akii2137/akii2137/assets/130637600/b14ca483-ba99-4431-b32a-5afcac23f32d)
----------

### Access the web page using ip address
```http://54.156.163.125/login```

#### Output:
![image](https://github.com/akii2137/akii2137/assets/130637600/9e635d1e-0679-4141-91f4-8b83fe44ca6a)

#### Login with the valid credentials.
![image](https://github.com/akii2137/akii2137/assets/130637600/10dd28a3-1d37-4da3-b35f-1710aaf042e9)

#### after login it will redirect to dashboard page.
![image](https://github.com/akii2137/akii2137/assets/130637600/7bf1e923-a856-4054-89e0-3d35a532c0c8)
