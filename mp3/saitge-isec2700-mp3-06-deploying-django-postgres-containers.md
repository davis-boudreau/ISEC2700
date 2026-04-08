# **ISEC2700 – GNS3 Appliance Build Guide**

## **Deploying Django and PostgreSQL Containers in GNS3**

**Course:** ISEC2700 – Intro to Information Security Practices
**Instructor:** Davis Boudreau
**Use Case:** MP03-05 – Secure Application & Database
**Audience:** Students deploying containerized services inside GNS3

---

# **1. Overview**

In this guide, you will learn how to deploy two application containers into GNS3:

* a **Django web application container**
* a **PostgreSQL database container**

These containers will be used as the final workload services in your secure network.

By the end of this guide, you should be able to:

* build a Docker image from source
* optionally publish that image to Docker Hub
* understand what a **GNS3 appliance file (`.gns3a`)** does
* import a GNS3 appliance into your project
* connect the containers to the correct GNS3 switches
* configure container networking

---

# **2. What is a GNS3 Appliance File?**

A **GNS3 appliance file** is a definition file that tells GNS3 how to deploy a device.

For Docker-based appliances, the `.gns3a` file usually describes:

* the appliance name
* the image to use
* the number of network adapters
* the console type
* categories and symbols
* how GNS3 should start the container

---

## **Why use a `.gns3a` file?**

Without an appliance file, you may still be able to add a raw Docker container in GNS3, but a `.gns3a` file gives you:

* a reusable deployment definition
* consistency across student builds
* easier import into GNS3
* standardized adapter and console settings

---

# **3. Deployment Workflow**

There are three common deployment paths.

## **Option A – Instructor provides ready-to-use Docker images**

You pull the image and import the appliance.

## **Option B – Instructor provides source code**

You build the image locally, then use it in GNS3.

## **Option C – Student builds and pushes to Docker Hub**

You build your own image and publish it, then point GNS3 to your Docker Hub image.

For this project, all three are acceptable if your instructor allows them.

---

# **4. Target Network Placement**

These are the expected locations in the MP3 project.

| Service    | VLAN    | Network         | Target IP    | Gateway      |
| ---------- | ------- | --------------- | ------------ | ------------ |
| Django App | VLAN 40 | 192.168.40.0/24 | 192.168.40.2 | 192.168.40.1 |
| PostgreSQL | VLAN 50 | 192.168.50.0/24 | 192.168.50.2 | 192.168.50.1 |

---

# **5. Part A – Build the Django Docker Image**

## **Step 1 – Clone the instructor repository**

Your instructor will provide a repository URL.

Example:

```bash
git clone <instructor-repo-url>
cd <django-demo-app-folder>
```

---

## **Step 2 – Review the project files**

Before building, inspect the project.

Look for files such as:

* `Dockerfile`
* `requirements.txt`
* `manage.py`
* project settings file
* optional `.env.example`

---

## **Step 3 – Example Dockerfile**

Your instructor may already provide one. A simple example could look like this:

```dockerfile
FROM python:3.12-slim

WORKDIR /app

ENV PYTHONDONTWRITEBYTECODE=1
ENV PYTHONUNBUFFERED=1

COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt

COPY . .

EXPOSE 8000

CMD ["python", "manage.py", "runserver", "0.0.0.0:8000"]
```

---

## **Step 4 – Build the image**

```bash
docker build -t isec2700-django-demo:latest .
```

---

## **Step 5 – Verify the image exists**

```bash
docker images
```

You should see:

```text
isec2700-django-demo   latest
```

---

# **6. Part B – Build or Pull the PostgreSQL Image**

For PostgreSQL, you will usually use an official base image.

Example:

```bash
docker pull postgres:14.22-trixie
```

If your instructor provides a custom PostgreSQL image, use that instead.

---

## **Required PostgreSQL environment values**

At minimum, PostgreSQL usually needs:

* `POSTGRES_DB`
* `POSTGRES_USER`
* `POSTGRES_PASSWORD`

Example values:

```text
POSTGRES_DB=appdb
POSTGRES_USER=appuser
POSTGRES_PASSWORD=StrongPassword123!
```

---

# **7. Part C – Optional Docker Hub Workflow**

If you want GNS3 to pull your image from Docker Hub, you can push it.

## **Step 1 – Tag the image**

```bash
docker tag isec2700-django-demo:latest yourdockerhubname/isec2700-django-demo:latest
```

---

## **Step 2 – Log in**

```bash
docker login
```

---

## **Step 3 – Push**

```bash
docker push yourdockerhubname/isec2700-django-demo:latest
```

---

## **Why do this?**

This is useful if:

* you build on one machine
* GNS3 runs on another machine/server
* you want the GNS3 server to pull the image directly

---

# **8. Part D – Sample `.gns3a` Appliance Structure**

A `.gns3a` file is JSON.

Below is a **sample Django appliance file**.

## **Sample: `django-demo.gns3a`**

```json
{
  "name": "Django Demo App",
  "category": "guest",
  "description": "Simple Django demo application for ISEC2700 MP03-05.",
  "vendor_name": "NSCC / Instructor Provided",
  "vendor_url": "https://example.com",
  "product_name": "Django Demo Container",
  "registry_version": 4,
  "status": "stable",
  "symbol": ":/symbols/docker_guest.svg",
  "usage": "Deploy this appliance on VLAN 40. Configure eth0 with 192.168.40.2/24 and gateway 192.168.40.1.",
  "maintainer": "Instructor",
  "maintainer_email": "instructor@example.com",
  "first_port_name": "eth0",
  "port_name_format": "eth{0}",
  "qemu": false,
  "docker": {
    "adapters": 1,
    "image": "yourdockerhubname/isec2700-django-demo:latest",
    "console_type": "telnet",
    "start_command": "",
    "environment": [
      "DJANGO_DEBUG=False",
      "DJANGO_SECRET_KEY=ChangeThisSecret",
      "DB_NAME=appdb",
      "DB_USER=appuser",
      "DB_PASSWORD=StrongPassword123!",
      "DB_HOST=192.168.50.2",
      "DB_PORT=5432",
      "ALLOWED_HOSTS=192.168.40.2,localhost,127.0.0.1"
    ]
  },
  "images": [],
  "versions": [
    {
      "name": "latest",
      "images": [],
      "docker_image": "yourdockerhubname/isec2700-django-demo:latest"
    }
  ]
}
```

---

## **Sample: `postgres-db.gns3a`**

```json
{
  "name": "PostgreSQL DB",
  "category": "guest",
  "description": "PostgreSQL database container for ISEC2700 MP03-05.",
  "vendor_name": "Docker Official Image / Instructor Guided",
  "vendor_url": "https://example.com",
  "product_name": "PostgreSQL Container",
  "registry_version": 4,
  "status": "stable",
  "symbol": ":/symbols/docker_guest.svg",
  "usage": "Deploy this appliance on VLAN 50. Configure eth0 with 192.168.50.2/24 and gateway 192.168.50.1.",
  "maintainer": "Instructor",
  "maintainer_email": "instructor@example.com",
  "first_port_name": "eth0",
  "port_name_format": "eth{0}",
  "qemu": false,
  "docker": {
    "adapters": 1,
    "image": "postgres:14.22-trixie",
    "console_type": "telnet",
    "start_command": "",
    "environment": [
      "POSTGRES_DB=appdb",
      "POSTGRES_USER=appuser",
      "POSTGRES_PASSWORD=StrongPassword123!"
    ]
  },
  "images": [],
  "versions": [
    {
      "name": "14.22-trixie",
      "images": [],
      "docker_image": "postgres:14.22-trixie"
    }
  ]
}
```

---

# **9. Important Notes About the Sample Appliance Files**

These sample appliance files are meant to teach structure. In a real GNS3 deployment, your instructor may customize:

* the image name
* environment variables
* startup command
* console behavior
* volumes
* additional adapters

---

## **Common fields explained**

| Field             | Meaning                                   |
| ----------------- | ----------------------------------------- |
| `name`            | appliance display name in GNS3            |
| `category`        | type of appliance                         |
| `description`     | what the appliance is for                 |
| `symbol`          | icon in GNS3                              |
| `docker.adapters` | number of NICs                            |
| `docker.image`    | Docker image name                         |
| `console_type`    | terminal access type                      |
| `environment`     | environment variables passed to container |
| `versions`        | version records used by GNS3              |

---

# **10. Part E – Import the Appliance into GNS3**

## **Step 1 – Open GNS3**

Launch GNS3 and open your MP3 project.

---

## **Step 2 – Import appliance**

Go to:

```text
File → Import Appliance
```

---

## **Step 3 – Select the `.gns3a` file**

Choose either:

* `django-demo.gns3a`
* `postgres-db.gns3a`

---

## **Step 4 – Follow the prompts**

When GNS3 asks how to run the appliance:

* choose the correct GNS3 server / compute
* allow GNS3 to pull the Docker image if required

---

## **Step 5 – Finish import**

After import, the appliance should appear in your device list.

Repeat for both containers.

---

# **11. Part F – Add the Containers to the Topology**

Add the appliances to your project and connect them correctly.

## **Django app**

* place on the topology
* rename to `DJANGO-APP`
* connect `eth0` to **Switch5 / VLAN 40**

## **PostgreSQL**

* place on the topology
* rename to `POSTGRES-DB`
* connect `eth0` to **Switch6 / VLAN 50**

---

# **12. Part G – Configure Container Networking**

The exact method depends on how your appliance is built.

Some appliances use:

* manual console configuration
* startup scripts
* environment-based initialization

For this project, you may configure the addresses manually inside the container.

---

## **Django networking**

```bash
ip addr flush dev eth0
ip addr add 192.168.40.2/24 dev eth0
ip link set eth0 up
ip route add default via 192.168.40.1
```

---

## **PostgreSQL networking**

```bash
ip addr flush dev eth0
ip addr add 192.168.50.2/24 dev eth0
ip link set eth0 up
ip route add default via 192.168.50.1
```

---

## **Verify**

```bash
ip addr
ip route
```

---

# **13. Part H – Start and Validate the Containers**

## **Django container**

Inside the Django container, check:

```bash
ss -tulpn
env | sort
```

You want to confirm:

* IP is correct
* default route is correct
* app-related environment variables are present
* Django is listening on the expected port

---

## **PostgreSQL container**

Inside the PostgreSQL container, check:

```bash
ss -tulpn
ps aux | grep postgres
env | sort
```

You want to confirm:

* IP is correct
* PostgreSQL is running
* port 5432 is listening
* environment variables are set

---

# **14. Part I – Test the Services**

## **Test Django from the reverse proxy path**

Use the Chromium appliance and browse to the reverse proxy’s public entry point.

Expected:

* the Django app loads

---

## **Test PostgreSQL from authorized source**

From VLAN 40 or pgAdmin on the approved network:

```bash
nc -zv 192.168.50.2 5432
```

Expected:

* success

---

## **Test PostgreSQL from unauthorized source**

From VLAN 30 or another unauthorized network:

```bash
nc -zv 192.168.50.2 5432
```

Expected:

* fail

---

# **15. Suggested Student Workflow Summary**

Use this checklist.

## **Django**

1. Clone repo
2. Review Dockerfile
3. Build image
4. Optionally push to Docker Hub
5. Import `.gns3a`
6. Add to GNS3 project
7. Configure `eth0`
8. Validate service

## **PostgreSQL**

1. Pull official image or instructor image
2. Import `.gns3a`
3. Add to GNS3 project
4. Configure `eth0`
5. Set DB environment variables
6. Validate service

---

# **16. Troubleshooting**

## **Problem: appliance imports but container does not start**

Check:

* image name is correct
* Docker image exists on the GNS3 server
* `.gns3a` references the correct Docker image

---

## **Problem: GNS3 cannot pull the image**

Check:

* image name spelling
* Docker Hub access
* whether the image is private
* whether you are logged in on the correct host if needed

---

## **Problem: Django starts but app not reachable**

Check:

* container IP
* default gateway
* reverse proxy config
* whether Django is listening on `0.0.0.0`

---

## **Problem: PostgreSQL unreachable**

Check:

* VLAN placement
* ACLs on the core switch
* PostgreSQL startup state
* port 5432 listener

---

## **Problem: student-built image works locally but not in GNS3**

Check:

* whether the GNS3 server can access the image
* whether image was pushed to Docker Hub
* whether appliance points to the pushed tag

---

# **17. Deliverables**

Students should submit:

1. screenshot of the Django image build or pull result
2. screenshot of the PostgreSQL image pull result
3. screenshot of imported GNS3 appliances
4. screenshot of both containers placed in the topology
5. screenshot of Django container network configuration
6. screenshot of PostgreSQL container network configuration
7. screenshot of successful application access
8. screenshot of successful authorized PostgreSQL connectivity
9. screenshot of failed unauthorized PostgreSQL connectivity

---

# **18. Reflection Questions**

1. What is the purpose of a `.gns3a` appliance file?
2. Why might a student push an image to Docker Hub before importing it into GNS3?
3. Why is container networking configuration important inside this project?
4. Why should the Django container and PostgreSQL container live on different VLANs?
5. Why is a container not automatically secure just because it runs in Docker?

---

# **19. Instructor Notes**

Best practice is for the instructor to provide:

* one working Django demo repository
* one working Dockerfile
* one sample `.gns3a` for Django
* one sample `.gns3a` for PostgreSQL
* clear environment variable requirements

This keeps student effort focused on:

* deployment
* networking
* validation
* hardening

rather than losing time on packaging issues.

---

