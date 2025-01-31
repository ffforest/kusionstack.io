---
id: databse
---

import Tabs from '@theme/Tabs';
import TabItem from '@theme/TabItem';

# Managed Databases

You could also specify a database needed for the application. That can be achieved via a `mysql` or a `postgres` module (or bring-your-own-module) in the `accessories` field in `AppConfiguration` to achieve that.

You can currently have several databases with **different database names** for an application at the same time.

## Import

In the examples below, we are using schemas defined in the `kam` package and the `mysql` Kusion Module. For more details on KCL package and module import, please refer to the [Configuration File Overview](./1-overview.md#configuration-file-overview).

The `import` statements needed for the following walkthrough:
```
import kam.v1.app_configuration as ac
import service
import service.container as c
import mysql
import postgres
```

The `kcl.mod` must contain reference to the `mysql` module or `postgres` module:
```
#...

[dependencies]
mysql = { oci = "oci://ghcr.io/kusionstack/mysql", tag = "0.2.0" }
postgres = { oci = "oci://ghcr.io/kusionstack/postgres", tag = "0.2.0" }
#...
```

## Types of Database offerings

As of version 0.11.0, Kusion supports the following database offerings on the cloud:
- MySQL and PostgreSQL Relational Database Service (RDS) on [AWS](https://aws.amazon.com/rds/)
- MySQL and PostgreSQL Relational Database Service (RDS) on [AliCloud](https://www.alibabacloud.com/product/databases)

More database types on more cloud vendors will be added in the future.

Alternatively, Kusion also supports creating a database at `localhost` for local testing needs. A local database is quicker to stand up and easier to manage. It also eliminates the need for an account and any relevant costs with the cloud providers in the case that a local testing environment is sufficient.

:::info
You do need a local Kubernetes cluster to run the local database workloads. You can refer to [Minikube](https://minikube.sigs.k8s.io/docs/start/) or [Kind](https://kind.sigs.k8s.io/docs/user/quick-start/) to get started.
To see an end-to-end use case for standing up a local testing environment including a local database, please refer to the [Kusion Quickstart](../2-getting-started/2-getting-started-with-kusion-cli/1-deliver-quickstart.md).
:::

## Cloud Credentials and Permissions

Kusion provisions databases on the cloud via [terraform](https://www.terraform.io/) providers. For it to create _any_ cloud resources, it requires a set of credentials that belongs to an account that has the appropriate write access so the terraform provider can be initialized properly.

For AWS, the environment variables needed:
```
export AWS_REGION=us-east-1            # replace it with your region
export AWS_ACCESS_KEY_ID="xxxxxxxxxxx" # replace it with your AccessKey
export AWS_SECRET_ACCESS_KEY="xxxxxxx" # replace it with your SecretKey
```

For AliCloud, the environment variables needed:
```
export ALICLOUD_REGION=cn-shanghai     # replace it with your region
export ALICLOUD_ACCESS_KEY="xxxxxxxxx" # replace it with your AccessKey
export ALICLOUD_SECRET_KEY="xxxxxxxxx" # replace it with your SecretKey
```

The user account that owns these credentials would need to have the proper permission policies attached to create databases and security groups. If you are using the cloud-managed policies, the policies needed to provision a database and configure firewall rules are listed below.

For AWS:
- `AmazonVPCFullAccess` for creating and managing database firewall rules via security group
- `AmazonRDSFullAccess` for creating and managing RDS instances

For AliCloud:
- `AliyunVPCFullAccess` for creating and managing database firewall rules via security group
- `AliyunRDSFullAccess` for creating and managing RDS instances

Alternatively, you can use customer managed policies if the cloud provider built-in policies don't meet your needs. The list of permissions needed are in the [AmazonRDSFullAccess Policy Document](https://docs.aws.amazon.com/aws-managed-policy/latest/reference/AmazonRDSFullAccess.html#AmazonRDSFullAccess-json) and [AmazonVPCFullAccess Policy Document](https://docs.aws.amazon.com/aws-managed-policy/latest/reference/AmazonVPCFullAccess.html). It will most likely be a subset of the permissions in the policy documents.

## Configure Database

### Provision a Cloud Database

Assuming the steps in the [Cloud Credentials and Permissions](#cloud-credentials-and-permissions) section is setup properly, you can now provision cloud databases via Kusion.

#### AWS RDS Instance
To provision an AWS RDS instance with MySQL v8.0 or PostgreSQL v14.0, you can append the following YAML file to your own workspace configurations and update the corresponding workspace with command `kusion workspace update`. 

<Tabs>
<TabItem value="MySQL" >

```yaml
runtimes: 
  terraform: 
    random: 
      version: 3.5.1
      source: hashicorp/random
    aws: 
      version: 5.0.1
      source: hashicorp/aws
      region: us-east-1 # Please replace with your own aws provider region

# MySQL configurations for AWS RDS
modules: 
  kusionstack/mysql@0.1.0:
    default: 
      cloud: aws
      size: 20
      instanceType: db.t3.micro
      securityIPs: 
        - 0.0.0.0/0
      suffix: "-mysql"
```

```mdx-code-block
</TabItem>
<TabItem value="PostgreSQL">
```
```yaml
runtimes: 
  terraform: 
    random: 
      version: 3.5.1
      source: hashicorp/random
    aws: 
      version: 5.0.1
      source: hashicorp/aws
      region: us-east-1 # Please replace with your own aws provider region

# PostgreSQL configurations for AWS RDS
modules: 
  kusionstack/postgres@0.1.0:
    default: 
      cloud: aws
      size: 20
      instanceType: db.t3.micro
      securityIPs: 
        - 0.0.0.0/0
      suffix: "-postgres"
```

```mdx-code-block
</TabItem>
</Tabs>
```

For KCL configuration file declarations: 

<Tabs>
<TabItem value="MySQL" >

```python
wordpress: ac.AppConfiguration {
    # ...
    accessories: {
        "mysql": mysql.MySQL {
            type:   "cloud"
            version: "8.0"
        }
    }
}
```

```mdx-code-block
</TabItem>
<TabItem value="PostgreSQL">
```

```python
pgadmin: ac.AppConfiguration {
    # ...
    accessories: {
        "postgres": postgres.PostgreSQL {
            type:   "cloud"
            version: "14.0"
        }
    }
}
```

```mdx-code-block
</TabItem>
</Tabs>
```

It's highly recommended to replace `0.0.0.0/0` and closely manage the whitelist of IPs that can access the database for security purposes. The `0.0.0.0/0` in the example above or if `securityIPs` is omitted altogether will allow connections from anywhere which would typically be a security bad practice.

The `instanceType` field determines the computation and memory capacity of the RDS instance. The `db.t3.micro` instance type in the example above represents the `db.t3` instance class with a size of `micro`. In the same `db.t3` instance family there are also `db.t3.small`, `db.t3.medium`, `db.t3.2xlarge`, etc.

The full list of supported `instanceType` values can be found [here](https://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/Concepts.DBInstanceClass.html#Concepts.DBInstanceClass.Support).

You can also adjust the storage capacity for the database instance by changing the `size` field which is storage size measured in gigabytes. The minimum is 20. More details can be found [here](https://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/CHAP_Storage.html#Concepts.Storage.GeneralSSD).

#### AliCloud RDS Instance

To provision an Alicloud RDS instance with MySQL or PostgreSQL, you can append the following YAML file to your own workspace configurations and update the corresponding workspace with command `kusion workspace update`. Note that AliCloud RDS has several additional fields such as `category`, `subnetID` and `privateRouting`:

<Tabs>
<TabItem value="MySQL" >

```yaml
runtimes: 
  terraform: 
    random: 
      version: 3.5.1
      source: hashicorp/random
    alicloud:
      version: 1.209.1
      source: aliyun/alicloud
      region: cn-beijing # Please replace with your own alicloud provider region

# MySQL configurations for Alicloud RDS
modules: 
  kusionstack/mysql@0.1.0: 
    default: 
      cloud: alicloud
      size: 20
      instanceType: mysql.n2.serverless.1c
      category: serverless_basic
      privateRouting: false
      subnetID: [your-subnet-id]
      securityIPs: 
        - 0.0.0.0/0
      suffix: "-mysql"
```

```mdx-code-block
</TabItem>
<TabItem value="PostgreSQL">
```
```yaml
runtimes: 
  terraform: 
    random: 
      version: 3.5.1
      source: hashicorp/random
    alicloud:
      version: 1.209.1
      source: aliyun/alicloud
      region: cn-beijing # Please replace with your own alicloud provider region

# PostgreSQL configurations for Alicloud RDS
modules: 
  kusionstack/postgres@0.1.0:
    default:
      cloud: alicloud
      size: 20
      instanceType: pg.n2.serverless.1c
      category: serverless_basic
      privateRouting: false
      subnetID: [your-subnet-id]
      securityIPs: 
        - 0.0.0.0/0
      suffix: "-postgres"
```

```mdx-code-block
</TabItem>
</Tabs>
```

For KCL configuration file declarations: 

<Tabs>
<TabItem value="MySQL" >

```python
wordpress: ac.AppConfiguration {
    # ...
    accessories: {
        "mysql": mysql.MySQL {
            type:   "cloud"
            version: "8.0"
        }
    }
}
```

```mdx-code-block
</TabItem>
<TabItem value="PostgreSQL">
```

```python
pgadmin: ac.AppConfiguration {
    # ...
    accessories: {
        "postgres": postgres.PostgreSQL {
            type:   "cloud"
            version: "14.0"
        }
    }
}
```

```mdx-code-block
</TabItem>
</Tabs>
```

We will walkthrough `subnetID` and `privateRouting` in the [Configure Network Access](#configure-network-access) section.

The full list of supported `instanceType` values can be found in:
- [MySQL instance types(x86)](https://www.alibabacloud.com/help/en/rds/apsaradb-rds-for-mysql/primary-apsaradb-rds-for-mysql-instance-types#concept-2096487)
- [PostgreSQL instance types](https://www.alibabacloud.com/help/en/rds/apsaradb-rds-for-postgresql/primary-apsaradb-rds-for-postgresql-instance-types#concept-2096578)

### Local Database

To deploy a local database with MySQL v8.0 or PostgreSQL v14.0:

<Tabs>
<TabItem value="MySQL" >

```python
wordpress: ac.AppConfiguration {
    # ...
    accessories: {
        "mysql": mysql.MySQL {
            type: "local"
            version: "8.0"
        }
    }
}
```

```mdx-code-block
</TabItem>
<TabItem value="PostgreSQL">
```

```python
pgadmin: ac.AppConfiguration {
    # ...
    accessories: {
        "postgres": postgres.PostgreSQL {
            type:   "local"
            version: "14.0"
        }
    }
}
```

```mdx-code-block
</TabItem>
</Tabs>
```

## Database Credentials

There is no need to manage the database credentials manually. Kusion will automatically generate a random password, set it as the credential when creating the database, and then inject the hostname, username and password into the application runtime.

You have the option to BYO (Bring Your Own) username for the database credential by specifying the `username` attribute in the `workspace.yaml`:
```yaml
modules:
  kusionstack/mysql@0.1.0:
    default:
      # ...
      username: "my_username"
```

You **cannot** bring your own password. The password will always be managed by Kusion automatically.

The database credentials are injected into the environment variables of the application container. You can access them via the following env vars:
```
# env | grep KUSION_DB
KUSION_DB_HOST_WORDPRESS_MYSQL=wordpress.xxxxxxxx.us-east-1.rds.amazonaws.com
KUSION_DB_USERNAME_WORDPRESS_MYSQL=xxxxxxxxx
KUSION_DB_PASSWORD_WORDPRESS_MYSQL=xxxxxxxxx
```

:::info
More details about the environment of database credentials injected by Kusion can be found at [mysql credentials and connectivity](../6-reference/2-modules/1-developer-schemas/database/mysql.md#credentials-and-connectivity) and [postgres credentials and connectivity](../6-reference/2-modules/1-developer-schemas/database/postgres.md#credentials-and-connectivity)
:::

You can use these environment variables out of the box. Or most likely, your application might retrieve the connection details from a different set of environment variables. In that case, you can map the kusion environment variables to the ones expected by your application using the `$()` expression. 

This example below will assign the value of `KUSION_DB_HOST_WORDPRESS_MYSQL` into `WORDPRESS_DB_HOST`, `KUSION_DB_USERNAME_WORDPRESS_MYSQL` into `WORDPRESS_DB_USER`, likewise for `KUSION_DB_PASSWORD_WORDPRESS_MYSQL` and `WORDPRESS_DB_PASSWORD`:
```
wordpress: ac.AppConfiguration {
    workload: service.Service {
        containers: {
            wordpress: c.Container {
                image = "wordpress:6.3-apache"
                env: {
                    "WORDPRESS_DB_HOST": "$(KUSION_DB_HOST_WORDPRESS_MYSQL)"
                    "WORDPRESS_DB_USER": "$(KUSION_DB_USERNAME_WORDPRESS_MYSQL)"
                    "WORDPRESS_DB_PASSWORD": "$(KUSION_DB_PASSWORD_WORDPRESS_MYSQL)"
                }
                # ...
            }
        }
        # ...
    }
    accessories: {
        # ...
    }
}
```

## Configure Network Access

You can also optionally configure the network access to the database as part of the `AppConfiguration`. This is highly recommended because it dramatically increases the security posture of your cloud environment in the means of least privilege principle.

The `securityIPs` field in the `Database` schema declares the list of network addresses that are allowed to access the database. The network addresses are in the [CIDR notation](https://aws.amazon.com/what-is/cidr/) and can be either a private IP range ([RFC-1918](https://datatracker.ietf.org/doc/html/rfc1918) and [RFC-6598](https://datatracker.ietf.org/doc/html/rfc6598) address) or a public one.

If the database need to be accessed from a public location (which should most likely not be the case in a production environment), `securityIPs` need to include the public IP address of the traffic source (For instance, if the RDS database needs to be accessed from your computer).

To configure AWS RDS to restrict network access from a VPC with a CIDR of `10.0.1.0/24` and a public IP of `103.192.227.125`:

```yaml
modules: 
  kusionstack/mysql@0.1.0: 
    default: 
      cloud: aws
      # ...
      securityIPs: 
        - "10.0.1.0/24"
        - "103.192.227.125/32"
```

Depending on the cloud provider, the default behavior of the database firewall settings may differ if omitted.

### Subnet ID

On AWS, you have the option to launch the RDS instance inside a specific VPC if a `subnetID` is present in the application configuration. By default, if `subnetID` is not provided, the RDS will be created in the default VPC for that account. However, the recommendation is to self-manage your VPCs to provider better isolation from a network security perspective.

On AliCloud, the `subnetID` is required. The concept of subnet maps to VSwitch in AliCloud.

To place the RDS instance into a specific VPC on Alicloud:

```yaml
modules: 
  kusionstack/mysql@0.1.0: 
    default: 
      cloud: alicloud
      # ...
      subnetID: "subnet-xxxxxxxxxxxxxxxx"
```

### Private Routing

There is an option to enforce private routing on certain cloud providers if both the workload and the database are running on the cloud.

On AliCloud, you can set the `privateRouting` flag to `True`. The database host generated will be a private FQDN that is only resolvable and accessible from within the AliCloud VPCs. Setting `privateRouting` flag to `True` when `type` is `aws` is a no-op.

To enforce private routing on AliCloud:

```yaml
modules: 
  kusionstack/mysql@0.1.0: 
    default: 
      cloud: alicloud
      # ...
      privateRouting: true
```

Kusion will then generate a private FQDN and inject it into the application runtime as the environment variable `KUSION_DB_HOST_<DATABASE_NAME>` for the application to use. A complete list of Kusion-managed environment variables for mysql database can be found [here](../6-reference/2-modules/1-developer-schemas/database/mysql.md#credentials-and-connectivity).

Otherwise when using the public FQDN to connect to a database from the workload, the route will depend on cloud provider's routing preference. The options are generally either:
- Travel as far as possible on the cloud provider's global backbone network, or also referred to as cold potato routing, or
- Egress as early as possible to the public Internet and re-enter the cloud provider's datacenter later, or also referred to as hot potato routing

The prior generally has better performance but is also more expensive.

You can find a good read on the [AWS Blog](https://aws.amazon.com/blogs/architecture/internet-routing-and-traffic-engineering/) or the [Microsoft Learn](https://learn.microsoft.com/en-us/azure/virtual-network/ip-services/routing-preference-overview).