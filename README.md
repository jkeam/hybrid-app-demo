
# Hybrid Application Deployment with OpenShift Virtualization

## Overview

This project demonstrates hosting a hybrid workload comprised of both containerized and VM-based components on OpenShift. It showcases how OpenShift Virtualization allows organizations to co-locate their container and VM workloads on the same platform. This capability is crucial for cases where workloads cannot be fully modernized, or modernization must occur in phases.

## Application Architecture

This hybrid application includes:
- **WordPress Frontend**: A containerized WordPress application.
- **Windows Server Backend**: A VM running Windows Server with MySQL.
- **Redis Cache**: A containerized Redis instance for caching.

The architecture leverages the strengths of both containerized and VM-based workloads, providing a seamless environment for running modern and legacy applications together.

## Components and Their Interactions

- **WordPress Frontend**: Communicates with the backend MySQL database on the Windows Server VM.
- **Windows Server Backend**: Hosts the MySQL database that stores WordPress data.
- **Redis Cache**: Enhances performance by caching frequently accessed data.

## Deployment Instructions

1. **Clone the Repository**
    ```sh
    git clone https://github.com/your-repo/hybrid-app-demo.git
    cd hybrid-app-demo
    ```

2. **Install the Application with Helm**
    ```sh
    helm install helm-deploy-hybrid-app . --namespace demo-hybrid --create-namespace
    ```

## Next Steps

1. **Access the WordPress Frontend**
    - Navigate to the WordPress route URL specified in your `values.yaml` file.
    - Follow the WordPress setup instructions:
        - Select your language.
        - Create the initial admin user.
        
2. **Enable Redis Plugin in WordPress**
    - Install and activate the Redis plugin from the WordPress plugin repository.

3. **Create a Sample Post in WordPress**

4. **Verify Database Contents on Windows Server**
    - Ensure you have an RDP client like FreeRDP installed.
    - RDP into the Windows Server VM using the RDP service details provided.
    - Execute a MySQL command to verify the WordPress database contains the new post:
        ```sh
        mysql -u root -p<YourPassword> -e "USE wordpress_db; SELECT * FROM wp_posts;"
        ```

## Values File

Here's a sample `values.yaml` for your reference:
```yaml
namespace:
  name: demo-hybrid

serviceAccount:
  name: hybrid-app-sa

roleBinding:
  name: hybrid-app-rolebinding

secret:
  wordpressDb:
    name: wordpress-db-secret

configMap:
  wordpressFeConfig:
    name: wordpress-config
    wp_redis_host: wordpress-cache # Name of the Redis Service
  sysprepWindows:
    name: sysprep-wordpress-be
    autounattend:
      FullName: "Administrator"
      Organization: "Organization"
      ProductKey: "<YourProductKey>"   # Placeholder for product key value
      ComputerName: "wordpress-be"
      Password: "<YourPassword>"  # Placeholder for password value
      Owner: "Owner"
      TimeZone: "Eastern Standard Time"
      LocalAccountDescription: "Administrator"
      LocalAccountDisplayName: "Administrator"
      LocalAccountGroup: "Administrators"
      LocalAccountName: "Administrator"
      Username: "Administrator"
    postInstall:
      mysqlInstallerUri: "http://example.com/mysql-8.4.0-winx64.msi"
      vcRedistributablesUri: "http://example.com/VC_redist.x64.exe"
      mysqlInstallerFile: "mysql-8.4.0-winx64.msi"
      vcRedistributablesFile: "vc_redist.x64.exe"
      mysqlBaseDataDir: 'C:\ProgramData\MySQL\MySQL Server 8.0'
      mysqlBaseInstallDir: 'C:\Program Files\MySQL\MySQL Server 8.4'
      mysqlServiceName: "MySQL80"
      firewallRuleDisplayName: "MySQL Server"
      additionalScripts: []  # Any additional powershell commands you want to run include here
    myIni:
      slowQueryLogFile: "WORDPRESS-BE-slow.log"
      logErrorFile: "WORDPRESS-BE.err"
      logBinFile: "WORDPRESS-BE-bin"

service:
  wordpressFe:
    name: wordpress-fe
  wordpressCache:
    name: wordpress-cache
  wordpressBeRdp:
    name: wordpress-be-rdp
    nodePort: 31389
  wordpressBe:
    name: wordpress-be

pvc:
  redisData:
    name: redis-data-pvc
    accessMode: ReadWriteOnce
    storage: 10Gi
    storageClassName: odf-nvme-2-replicas
    volumeMode: Filesystem
  wordpressPlugins:
    name: wordpress-plugins-pvc
    accessMode: ReadWriteOnce
    storage: 10Gi
    storageClassName: odf-nvme-2-replicas
    volumeMode: Filesystem

deployment:
  redis:
    name: wordpress-cache
    replicas: 1
    image: "redis:latest"
  wordpressFe:
    name: wordpress-fe
    replicas: 1
    dbHost: wordpress-be
    dbUser: wordpress_user
    dbName: wordpress_db
    image: "wordpress:latest"

vm:
  wordpressBe:
    name: wordpress-be
    template: windows2k22-server-medium
    flavor: medium
    os: windows2k22
    workload: server
    size: medium
    running: true
    installationCdromUrl: 'http://example.com/win2022.iso'
    installationCdromStorageClass: odf-nvme-2-replicas
    installationCdromStorage: 20Gi
    rootDiskStorageClass: vm-odf-hdd-3-replicas
    rootDiskStorage: 120Gi
    cpuCores: 4
    cpuSockets: 1
    cpuThreads: 1
    memory: 16Gi
    windowsDriversDiskImage: 'registry.redhat.io/container-native-virtualization/virtio-win-rhel9@sha256:a8d455491d6c1ff45c6d8d340aa804313ce5613a59f53c7f4a5fcb61c14cc9fc'
    machineType: pc-q35-rhel9.2.0

route:
  wordpressFe:
    name: wordpress-fe
    host: wordpress-demo.apps.example.com
```
