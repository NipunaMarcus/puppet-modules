# WSO2 Identity Server Key Manager Puppet Module

This repository contains the Puppet Module for installing and configuring WSO2 Identity Server Key Manager on various environments. Configuration data is managed using [Hiera](http://docs.puppetlabs.com/hiera/1/). Hiera provides a mechanism for separating configuration data from Puppet scripts and managing them in a separate set of YAML files in a hierarchical manner.

## Supported Operating Systems

- Debian 6 or higher
- Ubuntu 12.04 or higher

## Supported Puppet Versions

- Puppet 2.7, 3 or newer

## How to Contribute
Follow the steps mentioned in the [wiki](https://github.com/wso2/puppet-modules/wiki) to setup a development environment and update/create new puppet modules.

## Packs to be Copied

Copy the following files to their corresponding locations.

1. Get the WSO2 Identity Server distribution pre-packaged with Key Manager features, unzip it, rename the unzipped folder as wso2is-km-<version>, compress the wso2is-km-<version> as zip file and copy to `<PUPPET_HOME>/modules/wso2is-km/files`
2. JDK 1.7_80 distribution to `<PUPPET_HOME>/modules/wso2base/files`

## Running  WSO2 Identity Server Key Manager in the `default` profile
By default `WSO2AM_DB` is configured to point to WSO2 API Manager's APIM_DB database. Change the `WSO2AM_DB` datasource url to `WSO2_CONFIG_DB` datasource url to run the `default` profile.  Copy the above mentioned files to their corresponding locations and apply the Puppet Modules.

## Running  WSO2 Identity Server Key Manager with clustering in specific profiles
No changes to Hiera data are required to run the distributed deployment of WSO2 Identity Server Key Manager, other than pointing to the correct resources such as the deployment synchronization and remote DB instances. WSO2 API Manager store, publisher and gateway nodes have to configured with WSO2 Identity Server Key Manager as the key manager node. Before starting WSO2 Identity Server Key Manager node, other WSO2 API Manager components have to be started first in distributed deployment. For more details refer the [ WSO2 Identity Server as Key Manager ](https://docs.wso2.com/display/CLUSTER44x/Configuring+the+Pre-Packaged+Identity+Server+5.1.0+with+API+Manager+1.10.0) clustering guides.

1. If the Clustering Membership Scheme is `WKA`, add the Well Known Address list.

   Ex:
    ```yaml
    wso2::clustering :
        enabled: true
        local_member_host: 192.168.100.13
        local_member_port: 4000
        membership_scheme: wka
        sub_domain: mgt
        wka:
           members:
             -
               hostname: 192.168.100.113
               port: 4000
             -
               hostname: 192.168.100.114
               port: 4000
    ```

2. Add external databases to master datasources

   Ex:
    ```yaml
    wso2::master_datasources:
      wso2_config_db:
        name: WSO2_CONFIG_DB
        description: The datasource used for config registry
        driver_class_name: "%{hiera('wso2::datasources::mysql::driver_class_name')}"
        url: jdbc:mysql://mysql-is-db:3306/IS_DB?autoReconnect=true
        username: "%{hiera('wso2::datasources::common::username')}"
        password: "%{hiera('wso2::datasources::common::password')}"
        jndi_config: jdbc/WSO2_CONFIG_DB
        max_active: "%{hiera('wso2::datasources::common::max_active')}"
        max_wait: "%{hiera('wso2::datasources::common::max_wait')}"
        test_on_borrow: "%{hiera('wso2::datasources::common::test_on_borrow')}"
        default_auto_commit: "%{hiera('wso2::datasources::common::default_auto_commit')}"
        validation_query: "%{hiera('wso2::datasources::mysql::validation_query')}"
        validation_interval: "%{hiera('wso2::datasources::common::validation_interval')}"

    ```

3. Configure registry mounting

   Ex:
    ```yaml
    wso2_config_db:
      path: /_system/config
      target_path: /_system/config/is-km
      read_only: false
      registry_root: /
      enable_cache: true

    wso2_gov_db:
      path: /_system/governance
      target_path: /_system/governance
      read_only: false
      registry_root: /
      enable_cache: true
    ```

4. Configure deployment synchronization

    Ex:
    ```yaml
    wso2::dep_sync:
        enabled: true
        auto_checkout: true
        auto_commit: true
        repository_type: svn
        svn:
           url: http://svnrepo.example.com/repos/
           user: username
           password: password
           append_tenant_id: true
    ```

## Running  WSO2 Identity Server Key Manager with Secure Vault
WSO2 Carbon products may contain sensitive information such as passwords in configuration files. [WSO2 Secure Vault](https://docs.wso2.com/display/Carbon444/Securing+Passwords+in+Configuration+Files) provides a solution for securing such information.

Uncomment and modify the below changes in Hiera file to apply Secure Vault.

1. Enable Secure Vault

    ```yaml
    wso2::enable_secure_vault: true
    ```

2. Add Secure Vault configurations as below

    ```yaml
    wso2::secure_vault_configs:
      <secure_vault_config_name>:
        secret_alias: <secret_alias>
        secret_alias_value: <secret_alias_value>
        password: <password>
    ```

    For Identity Server `5.1.0` which is based on WSO2 Carbon Kernel 4.4.x

    Ex:
    ```yaml
    wso2::secure_vault_configs:
      key_store_password:
        secret_alias: Carbon.Security.KeyStore.Password
        secret_alias_value: repository/conf/carbon.xml//Server/Security/KeyStore/Password,false
        password: wso2carbon
    ```

3. Add Cipher Tool configuration file templates to `template_list`

    ```yaml
    wso2::template_list:
      - repository/conf/security/cipher-text.properties
      - repository/conf/security/cipher-tool.properties
      - bin/ciphertool.sh
      - password-tmp
    ```

## Running  WSO2 Identity Server Key Manager on Kubernetes
WSO2 Puppet Module ships Hiera data required to deploy  WSO2 Identity Server Key Manager on Kubernetes. For more information refer to the documentation on [deploying WSO2 products on Kubernetes using WSO2 Puppet Modules](https://docs.wso2.com/display/PM200/Deploying+WSO2+Products+on+Kubernetes+Using+WSO2+Puppet+Modules).