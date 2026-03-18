Realtime Chat App with Docker Volumes

This is a Node.js real-time chat app running in Docker with all 5 volume types demonstrated: anonymous, named, bind mount, tmpfs, and DB persistent volume.

Project Structure:

realtime-chat-app/
├── backend/
│   ├── Dockerfile
│   ├── app.js
│   ├── package.json
│   └── index.html
├── data/
│   └── persistent/      # Host folder for bind mount
├── tmp/
├── docker-compose.yml
└── README.md


Prerequisites

Ubuntu EC2 instance

Docker & Docker Compose installed

Node.js & npm installed (for backend dependencies)

Node.js + WebSocket chat app with PostgreSQL. Here’s how each volume type fits:


Node.js + WebSocket container > Handles real-time connections.

Uses tmpfs for< > active message buffers (fast memory access).

Writes logs to bind-mounted folder> for host monitoring.

Stores non-critical temporary files in> anonymous volume.

Stores persistent attachments in> named volume.


Setup by Step  Instructions which I followed to complete this task:

Step 1: Updateed Ubuntu and Installed Docker

Update packages:

sudo apt update -y && sudo apt upgrade -y

Step2: Installed Docker & Docker-Compose:

sudo apt install -y docker.io
sudo apt install -y docker-compose

![docker-docker-compose version](day3_images/docker-docker-compose-version-1.1.png)

Step 3: Created Project Folders
mkdir -p ~/realtime-chat-app/backend ~/realtime-chat-app/data ~/realtime-chat-app/tmp
cd ~/realtime-chat-app
![folderstructure](day3_images/Folder_structure_1.2.png)


>backend → Node.js code

>data → bind mount folder (host folder)

>tmp → optional temp storage


Step 4: Initialize Node.js App

Go to backend folder:
cd backend


Initialize Node.js project:
npm init -y

Installed dependencies:npm install express socket.io

Note: express is the web server, socket.io is for real-time chat.

Step 5: Created Backend Files
vim app.js

const express = require('express');
const http = require('http');
const { Server } = require('socket.io');
const fs = require('fs');

const app = express();
const server = http.createServer(app);
const io = new Server(server);
const port = process.env.APP_PORT || 3000;

const anonDir = '/data/anon';
const tmpfsDir = '/data/tmp';
const bindDir = '/data/bind';
const namedDir = '/data/named';

[anonDir, tmpfsDir, bindDir, namedDir].forEach(dir => {
    if (!fs.existsSync(dir)) fs.mkdirSync(dir, { recursive: true });
});

fs.writeFileSync(`${anonDir}/anon.log`, 'Anonymous volume log\n', { flag: 'a' });
fs.writeFileSync(`${tmpfsDir}/tmp.log`, 'Tmpfs volume log\n', { flag: 'a' });
fs.writeFileSync(`${bindDir}/bind.log`, 'Bind mount log\n', { flag: 'a' });
fs.writeFileSync(`${namedDir}/named.log`, 'Named volume log\n', { flag: 'a' });

app.get('/', (req, res) => res.sendFile(__dirname + '/index.html'));

io.on('connection', (socket) => {
    console.log('User connected');
    socket.on('chat message', (msg) => {
        fs.appendFileSync(`${tmpfsDir}/messages.log`, msg + '\n');
        io.emit('chat message', msg);
    });
    socket.on('disconnect', () => console.log('User disconnected'));
});

server.listen(port, () => console.log(`Chat app running on port ${port}`));



index.html:
![index.html](day3_images/index.html_1.5.png)


Step 6: Created Multi-Stage Dockerfile
![Dockerfile](day3_images/Dockerfile_1.6.png)


Step 7: Created Docker Compose File

![ocker-compose-file](day3_images/docker-composefile_1.7.png)

Now go Go back to root:

cd ~/realtime-chat-app
vim docker-compose.yml


Step 8: Build and Run Containers

sudo docker-compose up -d --build
Check containers:
docker ps
Here we will see chat-app and db running.

![running -apps](day3_images/Chat_app%20_running_1.8.png)

Step 9: Test Chat App
   
Open browser: http://<EC2-Public-IP>:3000
http://43.204.25.214:3000


Open multiple tabs.

Send messages → should appear in all tabs.

![App is running](day3_images/Final_output_1.9.png)

Step 10: Verify Docker Volumes
# List volumes
docker volume ls

# Check named volume content
docker run --rm -v named-volume:/data busybox ls /data

# Check tmpfs content
docker exec -it <chat-app-container-id> ls /data/tmp

# Check bind mount logs
ls -l ./data/persistent
docker volume ls
docker run --rm -v named-volume:/data busybox ls /data
docker exec -it <chat-app-container-id> ls /data/tmp
ls -l ./data/persistent
![list of Volumes](day3_images/list_of_volumes_1.11.png)

Verified all 5 types of Docker volumes:

Anonymous: /data/anon

Named: /data/named

Bind: ./data/persistent

Tmpfs: /data/tmp

DB named volume: /var/lib/postgresql/data
![named-value](day3_images/named_volume_1.12.png)

Step 11: Stop Containers
docker-compose down
docker volume ls
Note: Here docker volume ls showing named volumes persist and anonymous removed.
Note: Containers stopped. Named volumes and DB volume persisted; anonymous volumes removed.
![volume_list](day3_images/volume_lists_1.13.png)

Step 12: Notes some important points:
Anonymous Volume: Temporary, auto-created.

Named Volume: Persistent, survives container rebuilds.

Bind Mount: Maps host folder to container.

Tmpfs: In-memory ephemeral storage.
DB Volume: Persistent database storage.

![volume-inspect](day3_images/volume_inspect_1.14.png)


All volumes:
![tested-volumes](day3_images/tested_all_volumes_1.15.png)


Check chat-app logs
sudo docker-compose logs -f chat-app
here we see
Server running on port 3000

Application is now fully running on EC2 + Docker + Browser



