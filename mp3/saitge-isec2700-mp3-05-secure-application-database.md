# **ISEC2700 – Mini Project 3 (MP03-05)**

## **Phase 05: Secure Application & Database (Final Hardening)**

**Course:** ISEC2700 – Intro to Information Security Practices<br>
**Instructor:** Davis Boudreau<br>
**Type:** Mini-Project Phase<br>
**Mode:** Individual<br>
**Estimated Time:** 1–2 hours<br>
**Prerequisite:** MP03-04 completed<br>

---

# **1. Overview / Purpose**

In this phase, you will deploy and harden the final workload components in the secure network:

* a **Django demo web application container**
* a **PostgreSQL database container**

By this point, you have already secured:

* the ISP router
* the pfSense firewall
* the reverse proxy
* the core switch and VLAN segmentation

Now you must deploy the actual services that the infrastructure is protecting.

This phase is important because a secure network is not enough by itself. The services running inside the network must also be deployed correctly, exposed correctly, and hardened appropriately.

---

# **2. What You Are Building**

You will deploy:

* a **Django demo application** on **VLAN 40**
* a **PostgreSQL database** on **VLAN 50**

The Django application will act as the web workload.
The PostgreSQL database will act as the backend datastore.

The intended traffic flow is:

```text
Client
  ↓
Router
  ↓
pfSense Firewall
  ↓
Reverse Proxy
  ↓
Django App (VLAN 40)
  ↓
PostgreSQL (VLAN 50)
```

---

# **3. Learning Goals**

By the end of this phase, you should be able to:

* deploy an instructor-provided application from source or image
* import application containers into GNS3 using appliance files
* configure container networking in GNS3
* validate communication between the web app and database
* harden Django for a production-like environment
* harden PostgreSQL credentials and exposure
* understand why application and database services must not be broadly exposed

---

# **4. Background Knowledge**

## **Why this phase matters**

In real environments, security is built in layers:

| Layer                 | Example from this project |
| --------------------- | ------------------------- |
| Perimeter             | Router                    |
| Edge security         | pfSense                   |
| Internal segmentation | Core switch ACLs          |
| Host protection       | UFW                       |
| App security          | Django hardening          |
| Data protection       | PostgreSQL hardening      |

A database behind a firewall can still be insecure if:

* it uses weak credentials
* it is reachable from too many networks
* the application is misconfigured
* secrets are stored poorly

---

## **Application hardening vs network hardening**

### Network hardening answers:

* Which host can reach which host?
* Which ports are allowed?
* Which VLANs can communicate?

### Application hardening answers:

* Is debug mode disabled?
* Are secrets protected?
* Is the service exposed only where needed?
* Is the database configured with least privilege?

Both are required.

---

# **5. Phase Scenario**

Your infrastructure is ready, and your organization now needs the application stack deployed.

The instructor has provided:

* a Django demo application repository
* a deployment pattern for building the application image
* a GNS3 appliance file approach for importing the container into GNS3

You must:

* obtain the instructor-provided code
* build or pull the application container
* import and configure the workload in GNS3
* connect the Django app and PostgreSQL database to the correct networks
* validate that the reverse proxy can reach the app
* validate that the app can reach the database
* apply final hardening controls

---

# **6. Service Placement**

| Service         | VLAN    | Network         | IP Address   | Purpose         |
| --------------- | ------- | --------------- | ------------ | --------------- |
| Django Demo App | VLAN 40 | 192.168.40.0/24 | 192.168.40.2 | web application |
| PostgreSQL      | VLAN 50 | 192.168.50.0/24 | 192.168.50.2 | database        |

### Default gateways

| Service    | Gateway      |
| ---------- | ------------ |
| Django App | 192.168.40.1 |
| PostgreSQL | 192.168.50.1 |

---

# **7. Deployment Options**

Students may deploy the Django application using one of the following methods.

## **Option A – Instructor-provided prebuilt container image**

Students pull the prebuilt image from the location provided by the instructor.

This is the easiest option and keeps the focus on security and deployment.

---

## **Option B – Build the container from the instructor repository**

Students clone the instructor repository and build the image locally.

This gives students practice with source-controlled deployments.

---

## **Option C – Student pushes built image to Docker Hub**

Students build the image themselves and push it to Docker Hub, then import it into GNS3 from Docker Hub.

This is optional, but valuable for learning container publishing workflows.

---

# **8. Instructor-Provided Materials**

The instructor should provide:

1. **Django demo application repository**
2. **Dockerfile** for the Django app
3. **requirements.txt** and app source
4. **environment variable guidance**
5. **GNS3 appliance file (.gns3a)** for:

   * Django app container
   * PostgreSQL container
6. Optional:

   * Docker Hub image names
   * sample `.env` file
   * seed database instructions

---

# **9. Task 1 – Obtain the Django Demo App**

Students must retrieve the application from the instructor-provided repository.

## **Example workflow**

1. Open the instructor-provided repo URL
2. Clone the repo locally
3. Review:

   * Dockerfile
   * application source
   * requirements
   * environment variable instructions

### Example clone command

```bash
git clone <instructor-provided-repo-url>
cd <repo-folder>
```

---

# **10. Task 2 – Build the Django App Container**

If students are building locally, they should build the image from the repo.

### Example build command

```bash
docker build -t isec2700-django-demo:latest .
```

Students should verify the image exists:

```bash
docker images
```

---

# **11. Optional Task – Push the Image to Docker Hub**

Students may optionally tag and push their built image to Docker Hub.

This is useful if they want GNS3 to pull the image directly from a registry.

### Example workflow

```bash
docker tag isec2700-django-demo:latest <dockerhub-username>/isec2700-django-demo:latest
docker login
docker push <dockerhub-username>/isec2700-django-demo:latest
```

### Teaching point

This helps students understand how container registries fit into deployment workflows.

---

# **12. Task 3 – Import the GNS3 Appliance Files**

Students must import the provided `.gns3a` appliance files.

These appliance files should define:

* the Docker image to use
* the number of adapters
* startup configuration expectations
* default console behavior

## **Directions**

1. Open GNS3
2. Go to **File → Import Appliance**
3. Select the provided `.gns3a` file
4. Follow the prompts
5. Choose to run on the correct GNS3 server / compute
6. Complete the import

Repeat for:

* Django app appliance
* PostgreSQL appliance

---

# **13. Task 4 – Place the Containers in the Topology**

Students must add both appliances to the MP3 topology.

## **Placement**

* Django app container → connect to **Switch5 / VLAN 40 path**
* PostgreSQL container → connect to **Switch6 / VLAN 50 path**

Students should rename the devices clearly:

| Appliance  | Suggested Label |
| ---------- | --------------- |
| Django app | DJANGO-APP      |
| PostgreSQL | POSTGRES-DB     |

---

# **14. Task 5 – Configure Container Networking in GNS3**

Students must configure the network settings for each container.

---

## **Django App Networking**

| Setting         | Value         |
| --------------- | ------------- |
| Interface       | eth0          |
| IP Address      | 192.168.40.2  |
| Subnet Mask     | 255.255.255.0 |
| Default Gateway | 192.168.40.1  |

---

## **PostgreSQL Networking**

| Setting         | Value         |
| --------------- | ------------- |
| Interface       | eth0          |
| IP Address      | 192.168.50.2  |
| Subnet Mask     | 255.255.255.0 |
| Default Gateway | 192.168.50.1  |

---

## **Student note**

The exact method may depend on how the appliance is built:

* static network values may be passed in appliance settings
* students may configure them inside the container console
* some containers may use environment variables or startup scripts

The instructor should standardize this if possible.

---

# **15. Task 6 – Configure PostgreSQL**

Students must deploy PostgreSQL with non-default credentials.

## **Required database settings**

| Item          | Example                           |
| ------------- | --------------------------------- |
| Database name | appdb                             |
| Username      | appuser                           |
| Password      | strong password chosen by student |

Students must avoid weak defaults such as:

* postgres/postgres
* admin/admin
* blank passwords

---

# **16. Task 7 – Configure Django Environment**

The Django container must be configured to use PostgreSQL.

Students should provide environment variables such as:

```bash
DJANGO_SECRET_KEY=replace-with-strong-secret
DJANGO_DEBUG=False
DB_NAME=appdb
DB_USER=appuser
DB_PASSWORD=replace-with-strong-password
DB_HOST=192.168.50.2
DB_PORT=5432
ALLOWED_HOSTS=192.168.40.2,localhost,127.0.0.1
```

---

# **17. Task 8 – Django Hardening**

Students must verify and document the following:

## **Required Django hardening items**

* `DEBUG=False`
* `ALLOWED_HOSTS` configured
* secret key not trivially hard-coded
* PostgreSQL used instead of SQLite
* application reachable through intended path
* no unnecessary development behavior left enabled

## **Example safer Django configuration**

```python
import os

DEBUG = os.environ.get("DJANGO_DEBUG", "False") == "True"

SECRET_KEY = os.environ.get("DJANGO_SECRET_KEY", "change-me")

ALLOWED_HOSTS = os.environ.get("ALLOWED_HOSTS", "").split(",")

DATABASES = {
    "default": {
        "ENGINE": "django.db.backends.postgresql",
        "NAME": os.environ.get("DB_NAME"),
        "USER": os.environ.get("DB_USER"),
        "PASSWORD": os.environ.get("DB_PASSWORD"),
        "HOST": os.environ.get("DB_HOST", "192.168.50.2"),
        "PORT": os.environ.get("DB_PORT", "5432"),
    }
}
```

---

# **18. Task 9 – PostgreSQL Hardening**

Students must verify and document:

* non-default username/password
* database service running
* reachable only through intended path
* not directly exposed to users
* only the application-side authorized traffic path should work

### Teaching point

The database is a backend service. It is not a public-facing service.

---

# **19. Task 10 – Start the Containers and Validate Service State**

Students should start the containers and inspect their state.

## **Inside Django container**

```bash
ss -tulpn
env | sort
```

## **Inside PostgreSQL container**

```bash
ss -tulpn
ps aux | grep postgres
```

Students should identify:

* listening ports
* configured environment variables
* whether the services are actually running

---

# **20. Task 11 – Validate Django Through the Reverse Proxy**

Use the Chromium browser appliance and browse to the exposed entry point through the reverse proxy.

Expected result:

* the Django demo app loads successfully through the intended network path

Students should document:

* the URL/IP used
* whether the page loaded
* whether the path used the reverse proxy rather than direct DB access

---

# **21. Task 12 – Validate Authorized Database Access**

Use the application path or an authorized test tool from VLAN 40.

Examples:

```bash
nc -zv 192.168.50.2 5432
```

Expected:

* success from the authorized web/application-side path

Students may also use pgAdmin if available on the correct VLAN for testing.

---

# **22. Task 13 – Validate Unauthorized Database Access Fails**

Use a test container from VLAN 30 or another unauthorized network and attempt:

```bash
nc -zv 192.168.50.2 5432
```

Expected:

* fail

This confirms the earlier switch ACL and segmentation controls are still protecting the database.

---

# **23. Task 14 – Inspect Open Ports**

Students should use the Debian IP tools container and other local inspection commands.

## **Examples**

```bash
curl http://192.168.40.2
nc -zv 192.168.50.2 5432
nmap 192.168.40.2
nmap 192.168.50.2
```

Students should interpret:

* which ports are open
* which paths are allowed
* which paths are blocked

---

# **24. Suggested GNS3 Appliance Notes**

The appliance design should keep things simple for students.

## **Django appliance**

Recommended:

* 1 adapter
* Docker image name set by instructor or student
* console enabled
* environment variable support if possible

## **PostgreSQL appliance**

Recommended:

* 1 adapter
* volume mount if persistence is desired
* environment variables for DB init
* console enabled

## **Optional instructor improvement**

Provide two variants:

* **prebuilt appliance** using instructor image
* **student-customizable appliance** using their own Docker Hub image

This gives flexibility without blocking the lab.

---

# **25. Troubleshooting Guide**

## **Problem: Django app does not load**

Check:

* container started
* correct IP on VLAN 40
* reverse proxy config
* Django listening on expected port
* environment variables set correctly

---

## **Problem: Django cannot connect to PostgreSQL**

Check:

* DB host is `192.168.50.2`
* DB port is `5432`
* credentials are correct
* PostgreSQL service is running
* VLAN 40 → VLAN 50 ACL permits 5432

---

## **Problem: PostgreSQL reachable from unauthorized network**

Check:

* core switch ACLs
* which switch/VLAN the test container is attached to
* whether testing is really being performed from VLAN 30

---

## **Problem: Django still running insecurely**

Check:

* `DEBUG` value
* secret key handling
* `ALLOWED_HOSTS`
* whether the container was rebuilt or restarted after changes

---

# **26. Deliverables**

Students should submit:

1. screenshot of the Django app container deployed in GNS3
2. screenshot of the PostgreSQL container deployed in GNS3
3. screenshot of the Django environment or settings showing:

   * `DEBUG=False`
   * PostgreSQL configuration
   * host configuration
4. screenshot showing the Django app successfully loaded through the reverse proxy
5. screenshot of successful authorized PostgreSQL connectivity
6. screenshot of failed unauthorized PostgreSQL connectivity
7. short written explanation covering:

   * why `DEBUG=False` matters
   * why secrets should not be hard-coded
   * why PostgreSQL should not be directly exposed
   * why container deployment does not automatically make an app secure

---

# **27. Reflection Questions**

1. Why is application hardening still necessary after securing the network?
2. Why should Django use PostgreSQL instead of SQLite in this phase?
3. Why is `DEBUG=False` important?
4. Why should PostgreSQL not be directly exposed to users?
5. What is the benefit of importing container appliances into GNS3 rather than only describing the services conceptually?
6. What is the value of optionally pushing a built image to Docker Hub?

---

# **28. Assessment Criteria (Suggested)**

| Criteria      | Description                                                            | Marks |
| ------------- | ---------------------------------------------------------------------- | ----: |
| Deployment    | Django and PostgreSQL containers correctly imported and placed in GNS3 |    10 |
| Configuration | Correct networking, environment variables, and service integration     |    10 |
| Hardening     | Safer Django and PostgreSQL settings                                   |     5 |
| Validation    | Allowed and denied paths demonstrated                                  |     5 |

**Total: 30 Marks**

---

# **29. Instructor Notes**

This phase is strongest when the instructor provides a clean deployment path.

Best practice is to provide:

* a ready repo
* a working Dockerfile
* clear required environment variables
* a `.gns3a` appliance for each container

This avoids wasting class time on container packaging errors and keeps the focus on:

* deployment
* networking
* security
* validation

A useful teaching point here is:

> A container is only a packaging format. It is not a security control by itself.

---

# **30. Suggested Closing Summary for Students**

By the end of MP03-05, you should have a complete layered environment in which:

* the reverse proxy is the front door
* the Django app is the application workload
* PostgreSQL is the protected backend datastore
* network segmentation limits access
* application settings reduce exposure
* database access is controlled and justified

At this point, the system is ready for a final validation and attack simulation phase.

