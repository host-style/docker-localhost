# Docker Localhost - Development Environment

## Installing WSL with Ubuntu

### Prerequisites
- Windows 10 version 2004 or higher (Build 19041 or higher) or Windows 11
- Administrator permissions

### Step by Step

#### 1. Enable WSL
Open PowerShell as Administrator and run:

```powershell
wsl --install
```

This command will:
- Enable the WSL feature
- Enable the Virtual Machine Platform
- Download and install the latest Linux kernel
- Set WSL 2 as default
- Install Ubuntu as the default distribution

#### 2. Restart the Computer
After installation, restart the computer when prompted.

#### 3. Configure Ubuntu
On the first Ubuntu startup:
1. Wait for the automatic installation to complete
2. Create a UNIX username (can be different from the Windows user)
3. Set a password (it won't appear on screen while typing)
4. Confirm the password

#### 4. Verify Installation
Check the WSL version and installed distributions:

```powershell
wsl --list --verbose
```

#### 5. Set WSL 2 as Default (if needed)
```powershell
wsl --set-default-version 2
```

#### 6. Access Ubuntu
- Open the Start menu and search for "Ubuntu"
- Or run `wsl` in PowerShell/CMD to open the default distribution

## Installing Docker on Ubuntu (WSL)

After setting up WSL with Ubuntu, follow the steps below to install Docker:

### 1. Update the System
```bash
sudo apt update
sudo apt upgrade -y
```

### 2. Install Dependencies
```bash
sudo apt install ca-certificates curl
sudo install -m 0755 -d /etc/apt/keyrings
sudo curl -fsSL https://download.docker.com/linux/ubuntu/gpg -o /etc/apt/keyrings/docker.asc
sudo chmod a+r /etc/apt/keyrings/docker.asc
```

### 3. Add Docker Repository
```bash
echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.asc] https://download.docker.com/linux/ubuntu \
  $(. /etc/os-release && echo "$VERSION_CODENAME") stable" | \
  sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
sudo apt update
```

### 4. Install Docker
```bash
sudo apt-get install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin -y
```

### 5. Configure Docker for Non-Root User
```bash
sudo groupadd docker
sudo usermod -aG docker $USER
newgrp docker
```

### 6. Configure Docker to Start on Boot
```bash
sudo systemctl enable docker.service
sudo systemctl enable containerd.service
```

### 7. Install Git (if needed)
```bash
sudo apt install git -y
```

### 8. Verify Installation
```bash
docker --version
docker compose version
```

## Project Configuration

### 1. Clone the Repository
Clone the project inside Ubuntu (WSL):

```bash
cd ~
git clone https://github.com/host-style/docker-localhost.git
cd docker-localhost
```

### 2. Prepare Environment

Create the environment variables file:

```bash
cp .env.example .env
```

Edit the `.env` file as needed:

```bash
nano .env
```

Available variables:
- `MYSQL_ROOT_PASSWORD`: MySQL root user password (default: `root`)
- `PUID`: User ID for file permissions (default: `1000`)
- `PGID`: Group ID for file permissions (default: `1000`)
- `APP_ENV`: Application environment (default: `development`)

**Tip**: To check your PUID and PGID on Ubuntu, run:
```bash
id -u  # PUID
id -g  # PGID
```

Press `Ctrl+O` to save and `Ctrl+X` to exit nano.

### 3. Project Structure

Projects should be placed inside the `/web` directory. Each project must have its own directory following this convention:

**Naming rule**: The directory name must match the domain used in the browser (without `.local`).

#### Examples:

| Project Directory | Browser Domain | Public Directory |
|---------------------|---------------------|-------------------|
| `/web/mysite/` | `mysite.local` | `/web/mysite/public/` |
| `/web/api.mysite/` | `api.mysite.local` | `/web/api.mysite/public/` |
| `/web/admin.system/` | `admin.system.local` | `/web/admin.system/public/` |

#### Project Structure:

```
web/
├── mysite/
│   ├── public/          # Public directory (index.php, assets, etc)
│   │   └── index.php
│   ├── src/             # Application source code
│   ├── vendor/          # Composer dependencies
│   └── composer.json
├── api.mysite/
│   ├── public/          # API public directory
│   │   └── index.php
│   └── ...
```

**Important**:
- The `public/` directory is the document root that will be served by the web server
- All public files (index.php, CSS, JS, images) must be in `public/`
- Source code and sensitive files should be outside `public/`

#### Create a New Project:

```bash
# Example: create project that will be accessed via mysite.local
cd ~/docker-localhost/web
mkdir -p mysite/public
echo "<?php phpinfo();" > mysite/public/index.php
```

#### Configure Domain in Windows Hosts File:

To access projects by `.local` domain, you need to add entries to the Windows `hosts` file.

**Step by Step:**

1. Open Notepad as **Administrator**:
   - Press `Windows + S`
   - Type "Notepad"
   - Right-click and select "Run as administrator"

2. In Notepad, click `File` → `Open`

3. Navigate to the directory:
   ```
   C:\Windows\System32\drivers\etc
   ```

4. Change the filter from "Text Documents" to "All Files"

5. Open the `hosts` file

6. Add the following lines at the end of the file:
   ```
   127.0.0.1 mysite.local
   127.0.0.1 api.mysite.local
   127.0.0.1 admin.system.local
   ```

7. Save the file (`Ctrl+S`)

**Complete hosts file example:**
```
# Copyright (c) 1993-2009 Microsoft Corp.
#
# This is a sample HOSTS file used by Microsoft TCP/IP for Windows.
# ...

127.0.0.1 localhost
::1 localhost

# Docker Localhost Projects
127.0.0.1 mysite.local
127.0.0.1 api.mysite.local
127.0.0.1 admin.system.local
```

**Note**: Add a line `127.0.0.1 yourdomain.local` for each new project you create.

Now you can access `http://mysite.local` in your browser.

#### Direct Access via Localhost:

In addition to custom domains, you can also directly access the `/web` directory through:

```
http://localhost
```

This access will load the `/web/index.php` file, which can be used as:
- Development environment home page
- Dashboard with links to all projects
- List of available projects

**Example of a simple `index.php`:**
```php
<?php
echo "<h1>Available Projects</h1>";
echo "<ul>";
echo "<li><a href='http://mysite.local'>My Site</a></li>";
echo "<li><a href='http://api.mysite.local'>API My Site</a></li>";
echo "</ul>";
```

### 4. Start the Environment
After configuring the projects, build and start the containers:

```bash
docker compose build
docker compose up -d
```

## Configuration

The `.docker/php/php.ini` file is optimized for local development environment with:
- Extended execution time (600s)
- Increased memory (512M)
- Larger uploads (128M)
- OPcache configured for instant revalidation
- All errors visible for debugging

---

## Credits

**Created by:**
Renato Rodrigues Jr
Developer
