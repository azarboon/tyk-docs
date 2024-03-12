---
title: "Upgrading Tyk On Hybrid"
date: 2024-02-6
tags: ["Upgrade Go Plugins", "Tyk plugins", "Hybrid", "Self Managed"]
description: "Explains how to upgrade Go Plugins on Self Managed (Hybrid)"
---

In a Hybrid deployment, the client hosts both the Control Plane and the Data Plane.

The Control Plane includes the following components:
- Tyk Dashboard
- MongoDB or PostgreSQL
- Redis (Master Instance)
- Management Gateway
- MDCB

The Data Plane consists of at least one Hybrid Gateway, a Redis instance and Tyk Pump (optional).

## Strategy

#### Control Plane

Upgrade the Control Plane components in the following order:
1. MDCB
2. Tyk Pump (if deployed)
3. Tyk Dashboard
4. Management Gateway.

#### Data Plane

Upgrade the Data Plane components in the following order:
1. Go Plugins (if applicable)
2. Hybrid Gateway(s)

---
## 1. Upgrade your Control Plane

### MDCB
Follow the instructions for your deployment type:
- **Docker**
    1. Backup your MDCB config file `tyk_sink.conf`
    2. Update the image version in the docker command or script to the target version
    3. Restart MDCB
- **Helm**
    1. Backup your MDCB config file `tyk_sink.conf`.  Note this step may not be relevant if you’re exclusively using the environment variables from the `values.yaml` to define your configuration.
    2. Update the image version in your `values.yaml` to the target version
    3. Run helm upgrade with the updated `values.yaml` file
- **Other (Linux)**
    1. Find the target version you want to upgrade in the [packagecloud.io/tyk/tyk-mdcb-stable](https://packagecloud.io/tyk/tyk-mdcb-stable) repository.
    2. Follow the upgrade instructions for your distro
        - RHEL/CentOS Upgrade
        
        ```bash
        sudo yum upgrade tyk-mdcb-stable-5.0.0
        ```
        
        - Debian/Ubuntu
        ```bash
        sudo apt-get install tyk-mdcb-stable-5.0.0
        ``` 

### Tyk Pump
Follow the instructions for component deployment type:
- **Docker**
    1. Back up your Pump config file `pump.conf`
    2. Update the image version in the docker command or script to the target version
    3. Restart the Tyk Pump
- **Helm**
    1. Backup your Pump config file `pump.conf`. Note this step may not be relevant if you’re exclusively using the environment variables from the `values.yaml` to define your configuration.
    2. Update the image version in your `values.yaml` to the target version
    3. Run helm upgrade with the updated `values.yaml` file
- **Other (Linux)**
    1. Find the target version you want to upgrade in the [packagecloud.io/tyk/tyk-pump](https://packagecloud.io/tyk/tyk-pump) repository.
    2. Follow the upgrade instructions for your distro
        - RHEL/CentOS Upgrade

        ```bash
        sudo yum upgrade tyk-pump-1.8.1
        ```

        - Debian/Ubuntu
        ```bash
        sudo apt-get install tyk-pump-1.8.1
        ```

### Tyk Dashboard
Follow the instructions for component deployment type:

- **Docker**
    1. Backup your Dashboard config file `tyk_analytics.conf`
    2. Update the image version in the docker command or script to the target version
    3. Restart the Tyk Dashboard
- **Helm**
    1. Backup your Dashboard config file `tyk_analytics.conf`. Note this step may not be relevant if you’re exclusively using the environment variables from the `values.yaml` to define your configuration.
    2. Update the image version in your `values.yaml` to the target version
    3. Run helm upgrade with the updated `values.yaml` file
- **Other (Linux)**
    1. Find the target version you want to upgrade in the [packagecloud.io/tyk/tyk-dashboard](https://packagecloud.io/tyk/tyk-dashboard) repository.
    2. Follow the upgrade instructions for your distro
        - RHEL/CentOS Upgrade
        
        ```bash
        sudo yum upgrade tyk-pump-5.2.5
        ```

        - Debian/Ubuntu
        
        ```bash
        sudo apt-get install tyk-pump-5.2.5 
        ```
### Management Gateway
Follow the instructions for component deployment type:
- **Docker**
    1. Backup your Gateway config file `tyk.conf`
    2. Update the image version in the docker command or script to the target version
    3. Restart the Gateway
- **Helm**
    1. Backup your Gateway config file `tyk.conf`. Note this step may not be relevant if you’re exclusively using the environment variables from the `values.yaml` to define your configuration.
    2. Update the image version in your `values.yaml` to the target version
    3. Run helm upgrade with the updated `values.yaml` file
- **Other (Linux)**
    1. Find the target version you want to upgrade in the [packagecloud.io/tyk/tyk-gateway](https://packagecloud.io/tyk/tyk-gateway)
    2. Follow the upgrade instructions for your distro
        - RHEL/CentOS Upgrade
        
        ```bash
        sudo yum upgrade tyk-gateway-5.2.5
        ```

        - Debian/Ubuntu

        ```bash
        sudo apt-get install tyk-gateway-5.2.5 
        ```
---
## 2. Upgrade your Go Plugins

 | Upgrade Path | Current Version | Target Version |
 | ---- | --------------- | -------------- |
 | [1](#path-1)    | < 4.1.0         | < 4.1.0        |
 | [2](#path-2)    | < 4.1.0         | \>= 4.1.0      |
 | [3](#path-3)    | \>= 4.1.0       | \>=5.1.0       |

### Path 1 - Upgrading Go Plugins (Before Upgrading Tyk Gateway) {#path-1}
 1. Open a terminal/command prompt in the directory of your plugin source file(s)  
 2. Run the following commands to initialize your plugin:
 ```bash
 go get
 github.com/TykTechnologies/tyk@6c76e802a29838d058588ff924358706a078d0c5

 # Tyk Gateway versions < 4.2 have a dependency on graphql-go-tools
 go mod edit -replace github.com/jensneuse/graphql-go-tools=github.com/TykTechnologies/graphql-go-tools@v1.6.2-0.20220426094453-0cc35471c1ca

 go mod tidy
 go mod vendor
 ```
3. Download the plugin compiler for the target version you’re upgrading to (e.g. 4.0.9).  See the Tyk Docker Hub [repo](https://hub.docker.com/r/tykio/tyk-plugin-compiler) for available versions. 
4. [Compile]({{< ref "plugins/supported-languages/golang#building-the-plugin">}}) your plugin using this compiler
5. [Create a plugin bundle]({{< ref "plugins/how-to-serve-plugins/plugin-bundles" >}}) that includes the newly compiled version

    {{< img src="img/developer-support/bundle_files_example.png" alt="Bundle ZIP example" width="800">}}

    Your manifest.json will look something like this:

    ```json
    {
      "file_list": [
	      "CustomGoPlugin.so"
      ],
      "custom_middleware": {
      "pre": [
      {
        "name": "AddHeader",
        "path": "CustomGoPlugin.so",
        "require_session": false,
        "raw_body_only": false
      }],
      "driver": "goplugin",
      "id_extractor": {
        "extract_from": "",
        "extract_with": "", 
        "extractor_config": {}}
      },
      "checksum": "",
      "signature": ""
    }
    ```

6. [Upload this bundle]({{< ref "tyk-cloud/configuration-options/using-plugins/uploading-bundle" >}}) to your configured bundled server.
> Before executing the next step, upgrade your Hybrid Gateways to the target version.  Otherwise, your Gateway(s) will pull down this bundle that was built with the target version and your plugin(s) will not load due to a version mismatch.
7. Update the [custom_middleware_bundle]({{< ref "plugins/how-to-serve-plugins/plugin-bundles#per-api--local-parameters" >}}) field in the API Definitions of all APIs that use your plugin. The field should be updated to use the new bundle file you created in step 5.
8. Validate that your plugin is working per your expectations. 

### Path 2 - Upgrading Go Plugins (Before Upgrading Tyk Gateway) {#path-2} 
1. Open a terminal/command prompt in the directory of your plugin source file(s)  
2. Based on your Target Version run the appropriate commands to initialize your plugin:
    - **Target Version <= v4.2.0**  
    ```bash
    go get github.com/TykTechnologies/tyk@6c76e802a29838d058588ff924358706a078d0c5
    # Tyk Gateway versions < 4.2 have a dependency on graphql-go-tools
    go mod edit -replace github.com/jensneuse/graphql-go-tools=github.com/TykTechnologies/graphql-go-tools@v1.6.2-0.20220426094453-0cc35471c1ca
    go mod tidy
    go mod vendor
    ```
    - **Target Version > v4.20 and < v5.1**
    ```bash
    go get github.com/TykTechnologies/tyk@54e1072a6a9918e29606edf6b60def437b273d0a
    # For Gateway versions earlier than 5.1 using the go mod vendor tool is required
    go mod tidy
    go mod vendor
    ```
    - **Target Version >= v5.1.0**
    ```bash
    go get github.com/TykTechnologies/tyk@ffa83a27d3bf793aa27e5f6e4c7106106286699d
    # In Gateway version 5.1, the Gateway and plugins transitioned to using # Go modules builds and don't use Go mod vendor anymore
    go mod tidy
    ```
3. Download the plugin compiler for the target version you’re upgrading to (e.g. 5.1.0).  See the Tyk Docker Hub [repo](https://hub.docker.com/r/tykio/tyk-plugin-compiler) for available versions. 
4. [Compile]({{< ref "plugins/supported-languages/golang#building-the-plugin">}}) your plugin using this compiler
5. [Create a plugin bundle]({{< ref "plugins/how-to-serve-plugins/plugin-bundles" >}}) that includes both your current version’s plugin along with the newly compiled version

    {{< img src="img/developer-support/bundle_files_example.png" alt="Bundle ZIP example" width="800">}}

    Your manifest.json will look something like this:

    ```json
    {
      "file_list": [
	      "CustomGoPlugin.so",
	      "CustomGoPlugin_v4.3.3_linux_amd64.so"
      ],
      "custom_middleware": {
        "pre": [
        {
          "name": "AddHeader",
          "path": "CustomGoPlugin.so",
          "require_session": false,
          "raw_body_only": false
        }],
        "driver": "goplugin",
        "id_extractor": {
          "extract_from": "",
          "extract_with": "", 
          "extractor_config": {}
        }
      },
      "checksum": "",
      "signature": ""
    }
    ```

    In this example,  the CustomGoPlugin.so in the file list would be the filename of your current version’s plugin.  You will already have this on hand as this is what has been running in your environment.  The *CustomGoPlugin_v4.3.3_linux_amd64.so* is the plugin compiled for the target version.  The “_v4.3.3_linux_amd64” is generated automatically by the compiler.  If your target version was 5.2.0, then “_v5.2.0_linux_amd64” would be appended to the shared object file output by the compiler.

    Your bundle zip file should include both the current version and target versions of the plugin.

6. [Upload this bundle]({{< ref "tyk-cloud/configuration-options/using-plugins/uploading-bundle" >}}) to your configured bundle server.  
7. Update the [custom_middleware_bundle]({{< ref "plugins/how-to-serve-plugins/plugin-bundles#per-api--local-parameters" >}}) field in the API Definitions of all APIs that use your plugin. The field should be updated to use the new bundle file you created in step 5.
8. Validate that your plugin is working per your expectations.  
9. Proceed with upgrading your [Tyk Data Plane (Hybrid Gateway(s))](#upgrading-data-plane-hybrid-gateways). Given that you loaded your target version plugin ahead of time, this version will be loaded automatically once you upgrade.

### Path 3 - Upgrading Go Plugins (Before Upgrading Tyk Gateway) {#path-3}
1. Open a terminal/command prompt in the directory of your plugin source file(s)  
2. Based on your Target Version run the appropriate commands to initialize your plugin:
    - **Target Version > v4.20 and < v5.1.0**
    ```bash
    go get github.com/TykTechnologies/tyk@54e1072a6a9918e29606edf6b60def437b273d0a
    # For Gateway versions earlier than 5.1 using the go mod vendor tool is required
    go mod tidy
    go mod vendor
    ```
    - **Target Version >= v5.1.0**
    ```bash
    go get github.com/TykTechnologies/tyk@ffa83a27d3bf793aa27e5f6e4c7106106286699d
    # In Gateway version 5.1, the Gateway and plugins transitioned to using # Go modules builds and don't use Go mod vendor anymore
    go mod tidy
    ```
3. Download the plugin compiler for the target version you’re upgrading to (e.g. 4.3.3).  See the Tyk Docker Hub [repo](https://hub.docker.com/r/tykio/tyk-plugin-compiler/tags) for available versions. 
4. [Compile]({{< ref "plugins/supported-languages/golang#building-the-plugin">}}) your plugin using this compiler
5. [Create a plugin bundle]({{< ref "plugins/how-to-serve-plugins/plugin-bundles" >}}) with the newly compiled version

    Your manifest.json will look something like this:

    ```json
    {
      "file_list": [
	      "CustomGoPlugin_v4.3.3_linux_amd64.so"
      ],
      "custom_middleware": {
      "pre": [
      {
        "name": "AddHeader",
        "path": "CustomGoPlugin.so",
        "require_session": false,
        "raw_body_only": false
      }],
      "driver": "goplugin",
      "id_extractor": {
        "extract_from": "",
        "extract_with": "", 
        "extractor_config": {}}
      },
      "checksum": "",
      "signature": ""
    }
    ```

    In this example, the CustomGoPlugin_v4.3.3_linux_amd64.so is the plugin compiled for the target version.  The “_v4.3.3_linux_amd64” is generated automatically by the compiler.  If your target version was 5.2.0, then “_v5.2.0_linux_amd64” would be appended to the shared object file output by the compiler. 

6. [Upload this bundle]({{< ref "tyk-cloud/configuration-options/using-plugins/uploading-bundle" >}}) to your configured bundle server.  
7. Proceed with upgrading your [Tyk Data Plane (Gateway)](#upgrading-data-plane-hybrid-gateways). 
8. Update the [custom_middleware_bundle]({{< ref "plugins/how-to-serve-plugins/plugin-bundles#per-api--local-parameters" >}}) field in the API Definitions of all APIs that use your plugin. The field should be updated to use the new bundle file you created in step 5.
9. Validate that your plugin is working per your expectations.  
10. Proceed with upgrading your [Tyk Data Plane (Hybrid Gateway(s))](#upgrading-data-plane-hybrid-gateways).  Given that you loaded your target version plugin ahead of time, this version will be loaded automatically once you upgrade.

---
## 3. Upgrade your Data Plane Hybrid Gateway(s){#upgrading-data-plane-hybrid-gateways}
Follow the instructions for component deployment type:
- **Docker**
    1. Backup your Gateway config file `tyk.conf`
    2. Update the image version in the docker command or script to the target version
    3. Restart the Gateway
- **Helm**
    1. Backup your Gateway config file `tyk.conf`. Note this step may not be relevant if you’re exclusively using the environment variables from the `values.yaml` to define your configuration.
    2. Update the image version in your `values.yaml` to the target version
    3. Run helm upgrade with the updated `values.yaml` file
- **Other (Linux)**
    1. Find the target version you want to upgrade in the Packagecloud repository: https://packagecloud.io/tyk/tyk-gateway
    2. Follow the upgrade instructions for your distro
        - RHEL/CentOS Upgrade
        ```bash
        sudo yum upgrade tyk-gateway-5.2.5
        ```
        - Debian/Ubuntu
        ```bash
        sudo apt-get install tyk-gateway-5.2.5 
        ```