# Docker Hands-On Lab

## What is this activity?

This hands-on workshop contains step-by-step instructions on how to get started with Docker. This workshop shows you how to:

- Build and run an image as a container
- Update application code and rebuild containers
- Persist data using Docker volumes
- Use bind mounts for live development
- Deploy Docker applications using multiple containers with a database
- Run multi-container applications using Docker Compose

---

## Prerequisites

### 1. Docker Installation

#### Linux (Ubuntu)

**System Requirements:**
- x86-64 system with Ubuntu 22.04, 24.04, or the latest non-LTS version
- If you're not using GNOME, install gnome-terminal for Docker Desktop terminal access

**Installation Steps:**

1. Update the package index:
```bash
sudo apt update
```

2. Install required packages:
```bash
sudo apt install -y apt-transport-https ca-certificates curl software-properties-common
```

3. Add Docker's official GPG key:
```bash
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /usr/share/keyrings/docker-archive-keyring.gpg
```

4. Add Docker repository:
```bash
echo "deb [arch=amd64 signed-by=/usr/share/keyrings/docker-archive-keyring.gpg] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
```

5. Update package index again:
```bash
sudo apt update
```

6. Install Docker Engine:
```bash
sudo apt install -y docker-ce docker-ce-cli containerd.io
```

7. Verify Docker installation:
```bash
docker --version
```

8. (Optional) Add your user to the docker group to run Docker without sudo:
```bash
sudo usermod -aG docker $USER
```
**Note:** Log out and log back in for this to take effect.

9. If not using GNOME, install gnome-terminal:
```bash
sudo apt install gnome-terminal
```

#### Windows

**System Requirements:**
- Windows 10 64-bit: Pro, Enterprise, or Education (Build 16299 or later)
- Windows 11 64-bit: Pro, Enterprise, or Education
- WSL 2 feature enabled

**Installation Steps:**

1. Download Docker Desktop for Windows from:
   - https://www.docker.com/products/docker-desktop

2. Run the installer (Docker Desktop Installer.exe)

3. Follow the installation wizard:
   - Ensure "Use WSL 2 instead of Hyper-V" option is selected
   - Complete the installation

4. Restart your computer when prompted

5. Start Docker Desktop from the Start menu

6. Verify installation by opening PowerShell or Command Prompt:
```powershell
docker --version
```

#### macOS

**System Requirements:**
- macOS 11 or newer
- Apple Silicon (M1/M2) or Intel processor

**Installation Steps:**

1. Download Docker Desktop for Mac from:
   - https://www.docker.com/products/docker-desktop
   - Choose the appropriate version (Apple Silicon or Intel chip)

2. Open the downloaded .dmg file

3. Drag Docker.app to your Applications folder

4. Open Docker from Applications

5. Follow the installation prompts and grant necessary permissions

6. Verify installation by opening Terminal:
```bash
docker --version
```

---

### 2. Git Installation

#### Linux (Ubuntu)

```bash
sudo apt update
sudo apt install -y git
```

Verify installation:
```bash
git --version
```

#### Windows

1. Download Git for Windows from:
   - https://git-scm.com/download/win

2. Run the installer

3. Follow the installation wizard (default settings are recommended)

4. Verify installation in Command Prompt or PowerShell:
```powershell
git --version
```

#### macOS

Git is usually pre-installed on macOS. If not:

**Option 1: Using Homebrew (Recommended)**
```bash
brew install git
```

**Option 2: Using Xcode Command Line Tools**
```bash
xcode-select --install
```

Verify installation:
```bash
git --version
```

---



## Getting Started with the Lab

### Step 1: Getting the App's Source Code

We will work with a demo Todo application.

Clone the repository:
```bash
git clone https://github.com/docker/getting-started-app.git
```

Navigate to the directory:
```bash
cd getting-started-app
ls
```

You should see the project structure with `src/`, `spec/`, `package.json`, etc.

---

### Step 2: Building an Image Using Dockerfile

Create a file named `Dockerfile` inside the `getting-started-app` directory. The name should be exactly `Dockerfile` (without any extension).

Add the following content to the Dockerfile:

```dockerfile
# syntax=docker/dockerfile:1

FROM node:24-alpine
WORKDIR /app
COPY . .
RUN npm install --omit=dev
CMD ["node", "src/index.js"]
EXPOSE 3000
```

**Explanation:**
- `FROM node:24-alpine` - Uses Node.js 24 on Alpine Linux as base image
- `WORKDIR /app` - Sets working directory inside container
- `COPY . .` - Copies all files from current directory to container
- `RUN npm install --omit=dev` - Installs production dependencies
- `CMD ["node", "src/index.js"]` - Command to run when container starts
- `EXPOSE 3000` - Documents that container listens on port 3000

Now build the image:
```bash
docker build -t getting-started .
```

The `-t` flag tags the image with the name "getting-started".

View your built image:
```bash
docker images
```

You should see an image named `getting-started` in the list.

---

### Step 3: Run the Image as a Container

Run your container using the docker run command:

```bash
docker run -d -p 127.0.0.1:3000:3000 getting-started
```

**Explanation:**
- `-d` - Runs container in detached mode (in background)
- `-p 127.0.0.1:3000:3000` - Maps port 3000 of container to port 3000 on host

After a few seconds, open your web browser to http://localhost:3000. You should see your Todo app.

You can play around with the app and add items to the list. The app is running inside a container, so any changes you make will not persist if you stop the container.

In another terminal, you can see the running container:
```bash
docker ps
```

This shows all running containers with their Container ID, Image name, Status, and Ports.

---

### Step 4: Update Source Code and Rebuild Image

Let's make a change to the source code and see how to update the container.

Open VSCode in the getting-started directory:
```bash
code .
```

Or manually open the file `src/static/js/app.js` and find line 56.

Change:
```javascript
<p className="text-center">No items yet! Add one above!</p>
```

To:
```javascript
<p className="text-center">You have no todo items yet! Add one above!</p>
```

Save the file (Ctrl + S or Cmd + S).

Now go back to the browser with your already running app. You will **not** see the change because the app is running inside a container and it does not have access to your modified local files.

To see the change, you need to rebuild the image and run a new container.

**Step 4.1: Stop the running container**

First, get the container ID:
```bash
docker ps
```

Stop the container:
```bash
docker stop <container_id>
```

The container is now stopped but still exists. View all containers (running and stopped):
```bash
docker ps -a
```

Remove the stopped container:
```bash
docker rm <container_id>
```

**Tip:** You can stop and remove a container in one command:
```bash
docker rm -f <container_id>
```

**Step 4.2: Rebuild and run**

Rebuild the image:
```bash
docker build -t getting-started .
```

Run a new container:
```bash
docker run -dp 127.0.0.1:3000:3000 getting-started
```

Now refresh the browser (http://localhost:3000), and you should see the updated text!

**Note:** A new container won't start if the old one is still running because of port conflict. You can have multiple containers running at the same time, but they cannot use the same port on the host machine. Always stop and remove the old container before starting a new one with the same port mapping.

**Important Observation:** If you add items to the todo list, then stop the container and start a new one, you will lose all the data because the data is stored inside the container, which is temporary. Next, we'll see how to persist data using volumes.

---

### Step 5: Persisting Data Using Volumes

Volumes provide the ability to connect specific filesystem paths of the container back to the host machine. If you mount a directory in the container, changes in that directory are also seen on the host machine. If you mount that same directory across container restarts, you'd see the same files.

There are two main types of volumes. You'll eventually use both, but you'll start with **volume mounts**.

#### Understanding the Problem

The demo todo app stores its data in `/etc/todos/todo.db` (SQLite database). With the database being a single file, if you can persist that file on the host and make it available to the next container, it should be able to pick up where the last one left off. By creating a volume and attaching (often called "mounting") it to the directory where you stored the data, you can persist the data. As your container writes to the `todo.db` file, it will persist the data to the host in the volume.

#### Using Volume Mounts

Think of a volume mount as an opaque bucket of data. Docker fully manages the volume, including the storage location on disk. You only need to remember the name of the volume.

**Step 5.1: Create a volume**

```bash
docker volume create todo-db
```

**Step 5.2: Stop and remove the current container**

Find the container ID:
```bash
docker ps
```

Stop and remove it:
```bash
docker rm -f <container_id>
```

**Step 5.3: Start container with volume attached**

```bash
docker run -dp 127.0.0.1:3000:3000 --mount type=volume,src=todo-db,target=/etc/todos getting-started
```

**Explanation:**
- `--mount type=volume,src=todo-db,target=/etc/todos` - Mounts the volume named `todo-db` to `/etc/todos` in the container
- `src` is the name of the volume
- `target` is the path inside the container where the volume is mounted

**Step 5.4: Test persistence**

1. Open the app (http://localhost:3000) and add a few items to the todo list

2. Stop and remove the container:
```bash
docker ps
docker rm -f <container_id>
```

3. Start a new container with the same volume:
```bash
docker run -dp 127.0.0.1:3000:3000 --mount type=volume,src=todo-db,target=/etc/todos getting-started
```

**Optional:** You can also add a name to your container using the `--name` flag:
```bash
docker run -dp 127.0.0.1:3000:3000 --mount type=volume,src=todo-db,target=/etc/todos --name todo-app getting-started
```

4. Refresh the browser. You should see the previous items you added!

The data is persisted in the volume, and the new container is using the same volume. You can stop and start containers as many times as you want, and your data will still be there as long as you are using the same volume.

**Key Takeaway:** Volumes help save database data, but what if you want to see your source code changes without rebuilding the image every time? For that, we can use **bind mounts** (covered in the next step).

---

### Step 6: Using Bind Mounts for Development

You used a volume mount to persist the data in your database. A volume mount is a great choice when you need somewhere persistent to store your application data.

A **bind mount** is another type of mount, which lets you share a directory from the host's filesystem into the container. When working on an application, you can use a bind mount to mount source code into the container. The container sees the changes you make to the code immediately, as soon as you save a file. This means that you can run processes in the container that watch for filesystem changes and respond to them.

#### Comparison: Volume vs Bind Mount

**Named volume:**
```
type=volume,src=my-volume,target=/usr/local/data
```

**Bind mount:**
```
type=bind,src=/path/to/data,target=/usr/local/data
```

#### Step 6.1: Experiment with Bind Mounts

First, stop and remove the current container:
```bash
docker ps
docker rm -f <container_id>
```

Make sure you're in the getting-started-app directory:
```bash
cd /path/to/getting-started-app
```

**Note:** Replace `/path/to/getting-started-app` with your actual path. If you're already in the directory, you can skip this step.

Start an Ubuntu container with a bind mount:
```bash
docker run -it --mount type=bind,src="$(pwd)",target=/src ubuntu bash
```

**Explanation:**
- `-it` - Interactive mode with terminal
- `type=bind` - Uses bind mount
- `src="$(pwd)"` - Source is current directory (pwd = print working directory)
- `target=/src` - Mounts to `/src` inside container
- `ubuntu` - Uses Ubuntu image
- `bash` - Starts bash shell

**Optional:** If you don't have the Ubuntu image, Docker will automatically pull it. You can also pull it manually first:
```bash
docker pull ubuntu
```

You should now be inside the Ubuntu container with a bash terminal.

Explore the container:
```bash
ls
```

You should see the standard Linux directories like `bin`, `etc`, `home`, `src`, etc.

Navigate to the mounted directory:
```bash
cd /src
ls
```

You should see the same files as in your `getting-started-app` directory on the host machine!

Create a test file:
```bash
touch nhai.txt
ls
```

Exit the container:
```bash
exit
```

Now check your host machine's `getting-started-app` directory. You should see the file `nhai.txt` there! This demonstrates that bind mounts sync changes bidirectionally between the host and container.

#### Step 6.2: Development Container with Live Reload

Stop and remove the Ubuntu container (if it's still running, though it should have stopped when you exited):
```bash
docker ps -a
docker rm <container_id>
```

Now we'll run a development container with live code reloading. This container will:
- Mount your source code into the container
- Install all dependencies
- Start nodemon to watch for filesystem changes

**Important:** Make sure you're in the `getting-started-app` directory.

Run the development container:
```bash
docker run -dp 127.0.0.1:3000:3000 \
    -w /app --mount type=bind,src="$(pwd)",target=/app \
    node:24-alpine \
    sh -c "npm install && npm run dev"
```

**For Windows (PowerShell):**
```powershell
docker run -dp 127.0.0.1:3000:3000 `
    -w /app --mount type=bind,src="${PWD}",target=/app `
    node:24-alpine `
    sh -c "npm install && npm run dev"
```

**For Windows (Command Prompt):**
```cmd
docker run -dp 127.0.0.1:3000:3000 -w /app --mount type=bind,src="%cd%",target=/app node:24-alpine sh -c "npm install && npm run dev"
```

**Explanation:**
- `-w /app` - Sets working directory to `/app` inside container
- `--mount type=bind,src="$(pwd)",target=/app` - Bind mounts current directory to `/app`
- `node:24-alpine` - Uses Node.js 24 Alpine image
- `sh -c "npm install && npm run dev"` - Installs dependencies and starts dev server with nodemon

**Optional Task:** You can also mount the database volume along with the bind mount to persist data:
```bash
docker run -dp 127.0.0.1:3000:3000 \
    -w /app --mount type=bind,src="$(pwd)",target=/app \
    --mount type=volume,src=todo-db,target=/etc/todos \
    node:24-alpine \
    sh -c "npm install && npm run dev"
```

#### Step 6.3: Test Live Reload

Wait a few seconds for the container to start and dependencies to install. Open the app at http://localhost:3000.

Now open VSCode in your `getting-started-app` directory:
```bash
code .
```

Open `src/static/js/app.js` and find line 109.

Change:
```javascript
{submitting ? 'Adding...' : 'Add Item'}
```

To:
```javascript
{submitting ? 'Adding...' : 'Add'}
```

Save the file (Ctrl + S or Cmd + S).

Refresh your browser (http://localhost:3000). You should see the button text changed from "Add Item" to "Add"!

**Magic!** You can make any changes to the source code and see them immediately in the browser without rebuilding the image or restarting the container. This is because:
1. Bind mount syncs your local files with the container
2. Nodemon watches for file changes and automatically restarts the application

#### Step 6.4: Build Final Image with Changes

After you've made and tested your changes, you can build a production image with the updated code.

Stop and remove the development container:
```bash
docker ps
docker rm -f <container_id>
```

Build the image:
```bash
docker build -t getting-started .
```

Run the production container:
```bash
docker run -dp 127.0.0.1:3000:3000 getting-started
```

Your changes are now part of the image!

---

### Step 7: Use Docker Compose

Docker Compose is a tool that helps you define and share multi-container applications. With Compose, you can create a YAML file to define the services and with a single command, you can spin everything up or tear it all down.

#### Why Use Docker Compose?

The big advantage of using Compose is you can define your application stack in a file, keep it at the root of your project repository (it's now version controlled), and easily enable someone else to contribute to your project. Someone would only need to clone your repository and start the app using Compose. In fact, you might see quite a few projects on GitHub/GitLab doing exactly this now.

#### Step 7.1: Create the Compose File

In the `getting-started-app` directory, create a file named `compose.yaml`.

```bash
touch compose.yaml
```

Or create it manually using VSCode:
```bash
code compose.yaml
```

#### Step 7.2: Define the App Service

In Step 6, you used the following command to start the application service:

```bash
docker run -dp 127.0.0.1:3000:3000 \
    -w /app --mount type=bind,src="$(pwd)",target=/app \
    --mount type=volume,src=todo-db,target=/etc/todos \
    node:24-alpine \
    sh -c "npm install && npm run dev"
```

Let's define this service in the `compose.yaml` file. Add the following content:

```yaml
services:
  app:
    image: node:24-alpine
    command: sh -c "npm install && npm run dev"
    ports:
      - 127.0.0.1:3000:3000
    working_dir: /app
    volumes:
      - ./:/app
    environment:
      MYSQL_HOST: mysql
      MYSQL_USER: root
      MYSQL_PASSWORD: secret
      MYSQL_DB: todos
```

**Explanation:**
- `services:` - Defines all the services (containers) in your application
- `app:` - The name of the first service
- `image:` - The Docker image to use
- `command:` - The command to run (overrides default CMD)
- `ports:` - Port mapping (host:container)
- `working_dir:` - Sets the working directory
- `volumes:` - Bind mount current directory to `/app`
- `environment:` - Environment variables for database connection

#### Step 7.3: Define the MySQL Service

Now, let's add the MySQL database service. Traditionally, you would run it with a command like:

```bash
docker run -d \
  --network todo-app --network-alias mysql \
  -v todo-mysql-data:/var/lib/mysql \
  -e MYSQL_ROOT_PASSWORD=secret \
  -e MYSQL_DATABASE=todos \
  mysql:8.0
```

Add the MySQL service to your `compose.yaml` file. Your complete file should now look like this:

```yaml
services:
  app:
    image: node:24-alpine
    command: sh -c "npm install && npm run dev"
    ports:
      - 127.0.0.1:3000:3000
    working_dir: /app
    volumes:
      - ./:/app
    environment:
      MYSQL_HOST: mysql
      MYSQL_USER: root
      MYSQL_PASSWORD: secret
      MYSQL_DB: todos

  mysql:
    image: mysql:8.0
    volumes:
      - todo-mysql-data:/var/lib/mysql
    environment:
      MYSQL_ROOT_PASSWORD: secret
      MYSQL_DATABASE: todos

volumes:
  todo-mysql-data:
```

**Explanation:**
- `mysql:` - The name of the database service (automatically becomes the network alias)
- `image: mysql:8.0` - Uses MySQL 8.0 image
- `volumes:` - Named volume for database persistence
- `environment:` - MySQL configuration variables
- `volumes:` (top-level) - Defines named volumes used by services

**Note:** When you ran the container with `docker run`, Docker created the named volume automatically. However, that doesn't happen when running with Compose. You need to define the volume in the top-level `volumes:` section and then specify the mountpoint in the service config.

#### Step 7.4: Start the Application Stack

First, make sure no other copies of the containers are running. Check running containers:

```bash
docker ps
```

If you have containers from previous steps, remove them:

```bash
docker rm -f <container_id>
```

Or remove all running containers:
```bash
docker rm -f $(docker ps -aq)
```

Now start the application stack using Docker Compose:

```bash
docker compose up -d
```

**Explanation:**
- `docker compose up` - Starts all services defined in compose.yaml
- `-d` - Runs in detached mode (background)

You should see output like this:

```
[+] Running 4/4
 ✔ Network getting-started-app_default           Created
 ✔ Volume "getting-started-app_todo-mysql-data"  Created
 ✔ Container getting-started-app-mysql-1         Started
 ✔ Container getting-started-app-app-1           Started
```

**Notice:** Docker Compose created the volume as well as a network. By default, Docker Compose automatically creates a network specifically for the application stack (which is why you didn't define one in the Compose file).

#### Step 7.5: View Logs

View the logs using the `docker compose logs` command:

```bash
docker compose logs -f
```

**Explanation:**
- `logs` - Shows logs from all services
- `-f` - Follows the log (live output)

You'll see the logs from each of the services interleaved into a single stream. This is incredibly useful when you want to watch for timing-related issues.

Example output:
```
getting-started-app-mysql-1  | 2024-01-15 10:30:15+00:00 [Note] [Entrypoint]: Entrypoint script for MySQL Server 8.0.35-1.el8 started.
getting-started-app-mysql-1  | 2024-01-15 10:30:16+00:00 [Note] [Entrypoint]: Switching to dedicated user 'mysql'
getting-started-app-app-1    | npm info using npm@10.2.4
getting-started-app-app-1    | npm info using node@v24.0.0
getting-started-app-mysql-1  | 2024-01-15 10:30:18+00:00 [Note] [Entrypoint]: Database initialized
```

The service name is displayed at the beginning of the line (often colored) to help distinguish messages.

**View logs for a specific service:**
```bash
docker compose logs -f app
```

Or for MySQL:
```bash
docker compose logs -f mysql
```

Press `Ctrl + C` to stop following the logs.

#### Step 7.6: Access the Application

Open your browser and navigate to http://localhost:3000. You should see your Todo app running with MySQL database backend!

Try adding some todo items to test the database connection.

#### Step 7.7: View in Docker Desktop

If you're using Docker Desktop, look at the Dashboard. You'll see a group named `getting-started-app`. This is the project name from Docker Compose and is used to group the containers together. By default, the project name is simply the name of the directory that the `compose.yaml` was located in.

If you expand the stack, you'll see the two containers you defined in the Compose file:
- `getting-started-app-app-1` - Your Node.js application
- `getting-started-app-mysql-1` - Your MySQL database

The names follow the pattern of `<project-name>-<service-name>-<replica-number>`, making it easy to identify what each container does.

#### Step 7.8: Tear Down the Application

When you're ready to stop the application, run:

```bash
docker compose down
```

This will:
- Stop all running containers
- Remove the containers
- Remove the network

Alternatively, you can click the trash can icon on the Docker Desktop Dashboard for the entire app.

**Important Warning:** By default, named volumes in your compose file are **not removed** when you run `docker compose down`. If you want to remove the volumes as well, add the `--volumes` flag:

```bash
docker compose down --volumes
```

**Note:** The Docker Desktop Dashboard does not remove volumes when you delete the app stack either.

#### Step 7.9: Restart the Application

To start everything again, simply run:

```bash
docker compose up -d
```

All your services will start with the same configuration!

---

## Summary

Congratulations! You've completed the Docker hands-on lab. You've learned:

1. **Building images** - Using Dockerfile to create container images
2. **Running containers** - Starting containers from images
3. **Managing containers** - Stopping, removing, and listing containers
4. **Updating applications** - Modifying code and rebuilding images
5. **Volume mounts** - Persisting database data across container restarts
6. **Bind mounts** - Live code reloading during development
7. **Docker Compose** - Defining and running multi-container applications with a single command

### Key Commands Reference

| Command | Description |
|---------|-------------|
| `docker build -t <name> .` | Build an image from Dockerfile |
| `docker images` | List all images |
| `docker run -dp <port>:<port> <image>` | Run container in detached mode |
| `docker ps` | List running containers |
| `docker ps -a` | List all containers (including stopped) |
| `docker stop <container_id>` | Stop a running container |
| `docker rm <container_id>` | Remove a stopped container |
| `docker rm -f <container_id>` | Force stop and remove a container |
| `docker volume create <name>` | Create a named volume |
| `docker compose up -d` | Start all services defined in compose.yaml |
| `docker compose down` | Stop and remove all services |
| `docker compose logs -f` | View logs from all services |
| `docker --version` | Check Docker version |

### Next Steps

- Explore Docker Compose for multi-container applications
- Learn about Docker networking
- Use Docker Hub to share your images
- Experiment with Docker playground for interactive learning

---

**Happy Dockerizing! 🐳**




