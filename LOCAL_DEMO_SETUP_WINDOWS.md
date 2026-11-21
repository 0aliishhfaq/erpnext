# Local ERPNext Demo — Windows 10/11

This guide starts the official disposable ERPNext demo locally. It is for trying the standard ERPNext interface only.

**Important scope:** This quick demo is based on the official `frappe_docker/pwd.yml` setup. It does not load custom Travel Agency modules, Frappe CRM customizations, the PDF/Excel importer, or the design/print formats described elsewhere in this repository. Those require a separate development setup and a real custom Frappe app.

**Do not enter real passenger, passport, payment, or customer data.** The official `pwd.yml` setup is disposable and is not production-ready.

## 1. Windows prerequisites

1. Use a supported Windows 10/11 desktop with hardware virtualization enabled.
2. Install/update WSL 2. Open PowerShell as Administrator and run:

   ```powershell
   wsl --install
   wsl --update
   wsl --version
   ```

   Restart Windows if prompted. Docker's current Windows requirements specify WSL 2.1.5 or later for the WSL backend.

3. Install Docker Desktop for Windows from https://docs.docker.com/desktop/setup/install/windows-install/
4. In Docker Desktop settings, enable the WSL 2 backend and use Linux containers. Start Docker Desktop and wait until it says Docker is running.
5. Install Git for Windows from https://git-scm.com/download/win if Git is not already installed.
6. Open a new PowerShell window and verify:

   ```powershell
   docker --version
   docker compose version
   git --version
   ```

   Docker Desktop's documented Windows hardware baseline includes 8 GB system RAM and hardware virtualization. More RAM is helpful for running ERPNext alongside normal desktop applications.

## 2. Start the official ERPNext demo

Open PowerShell and run:

```powershell
mkdir $HOME\frappe-demo -Force
cd $HOME\frappe-demo
git clone https://github.com/frappe/frappe_docker.git
cd .\frappe_docker
docker compose -f pwd.yml up -d
```

The first start downloads container images and creates a demo site. Wait for the setup to finish. Follow the site creation logs:

```powershell
docker compose -f pwd.yml logs -f create-site
```

When the `create-site` container exits successfully, press Ctrl+C to stop following logs (this does not stop the containers).

Open:

- URL: http://localhost:8080
- Username: `Administrator`
- Password: `admin`

These are the credentials documented for the official disposable demo. Change them only if the demo setup instructions/version you use specify a different value. Do not expose this demo to the public internet.

## 3. Common commands

Run from the cloned `frappe_docker` folder:

```powershell
# See container status
docker compose -f pwd.yml ps

# Follow logs
docker compose -f pwd.yml logs -f

# Stop containers but retain named volumes for this local demo
docker compose -f pwd.yml down

# Start the existing demo again
docker compose -f pwd.yml up -d

# Delete the demo and its volumes (DATA LOSS — only when you want a clean reset)
docker compose -f pwd.yml down -v
```

Do not use `down -v` unless you intentionally want to delete the local demo data.

## 4. What this demo does and does not show

| Capability | Available in quick demo? |
| --- | --- |
| Standard ERPNext screens | Yes |
| Accounting / stock / sales sample workflows | Yes, subject to included ERPNext configuration |
| Our Travel Agency dashboard / DocTypes | No |
| Visa, Umrah, tour custom workflows | No |
| Consolidator PDF/Excel importer | No |
| Our professional travel print formats | No |
| Install custom apps into the quick PWD demo | No |

The official Frappe Docker documentation explicitly describes `pwd.yml` as a disposable quick exploration setup and directs custom app development to the VS Code Dev Containers development environment.

## 5. Next step for the actual Travel Agency demo

Once standard ERPNext is running, the next development setup must use the official Frappe Docker Dev Container workflow. It will need:
- A supported and pinned Frappe/ERPNext version pair (do not use a moving development branch as production).
- A separate `travel_agency` Frappe app repository with its DocTypes, workflows, permissions, print formats, importer and tests.
- Frappe CRM installation and integration.
- Synthetic-only demo data and private-file tests.

This repository is currently the ERPNext upstream fork and contains design/implementation documents; the custom Travel Agency app has not yet been implemented. Do not treat the stock ERPNext PWD demo as the completed Travel Agency product.

## Official references

- Frappe Docker repository and demo setup: https://github.com/frappe/frappe_docker
- Choosing deployment vs development method: https://frappe.github.io/frappe_docker/01-getting-started/01-choosing-a-deployment-method.html
- Frappe Docker development getting started: https://frappe.github.io/frappe_docker/getting-started.html
- Docker Desktop for Windows: https://docs.docker.com/desktop/setup/install/windows-install/
- Docker Desktop with WSL 2: https://docs.docker.com/desktop/features/wsl/
