# Embedding Model Deployment Plan

This document outlines the steps to deploy a Dockerized embedding model with an Nginx reverse proxy.

## Phase 1: Information Gathering & Setup

- [x] **1. Identify Docker Image:**
  - **Action:** Get the exact Docker image name from the user.
  - **Details:** The name should be in the format `username/repository:tag`.
  - **Image Name:** `cnmzsjbz199328/minilm-l6-v2:latest`

- [x] **2. Identify Container Port:**
  - **Action:** Get the internal port the model service listens on from the user.
  - **Details:** e.g., 80, 8080, 5000.
  - **Container Port:** `8000`

- [ ] **3. Create Project Directory:**
  - **Action:** Create a dedicated directory for this project.
  - **Status:** Done (`/home/tom/embedding_project`).

## Phase 2: Docker Deployment

- [ ] **1. Pull Docker Image:**
  - **Command:** `docker pull <image_name>`

- [ ] **2. Run Docker Container:**
  - **Command:** `docker run -d --name embedding_service -p <host_port>:<container_port> <image_name>`
  - **Details:** We will map the container port to a suitable host port.

## Phase 3: Nginx Configuration

- [ ] **1. Install Nginx:**
  - **Action:** Check if Nginx is installed. If not, install it.
  - **Command (Ubuntu/Debian):** `sudo apt update && sudo apt install nginx -y`

- [ ] **2. Create Nginx Config File:**
  - **Action:** Create a new Nginx site configuration file.
  - **Path:** `/etc/nginx/sites-available/embedding_proxy`
  - **Content:**
    ```nginx
    server {
        listen 80;
        server_name embedding.localhost; // Or a real domain

        location / {
            proxy_pass http://127.0.0.1:<host_port>;
            proxy_set_header Host $host;
            proxy_set_header X-Real-IP $remote_addr;
            proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
            proxy_set_header X-Forwarded-Proto $scheme;
        }
    }
    ```

- [ ] **3. Enable Nginx Site:**
  - **Command:** `sudo ln -s /etc/nginx/sites-available/embedding_proxy /etc/nginx/sites-enabled/`

- [ ] **4. Test and Reload Nginx:**
  - **Command (Test):** `sudo nginx -t`
  - **Command (Reload):** `sudo systemctl reload nginx`

## Phase 4: Verification

- [ ] **1. Test Proxy:**
  - **Action:** Send a request to the Nginx proxy to ensure it correctly forwards to the embedding service.
  - **Command:** `curl -X POST http://embedding.localhost/embed -H "Content-Type: application/json" -d '{"text": "Hello, world!"}'` (Example command, will be adjusted based on the actual API).
