# Oracle Autonomous Database Extproc Container Image Documentatio

adb-extproc container image spawns an Oracle extproc process which allows you to invoke
external scripts, functions and procedures written in other languages like (Python, C) from within your PL/SQL code.

Following key features are supported:
- SSL - An Oracle wallet containing self-signed certificate is generated on container startup to encrypt the communication between your ADB-S instance and extproc agent
- Invoke functions defined in custom libraries - Configure the container to load custom libraries from volume mounted in the container
- Execute custom scripts - For e.g. Python or bash scripts to perform ETL operations in the database
- Allow list - Configure `TCP_INVITED_NODES` to specify clients allowed to access the extproc listener 

adb-extproc container should be used with an Autonomous Database Serverless (ADB-S) instance in Cloud

## Using this image

To start an extproc container, run the following `podman` command

```text
/usr/bin/podman run -d \
-p 16000:16000 \
-e EXTPROC_WALLET_PASSWORD=${wallet_password} \
-e EXTPROC_DLLS='${dlls}' \
-e TCP_INVITED_NODES='${acls}' \
-e SCRIPTS_FOLDER_LOC_ENV='/tmp/' \
-e TRACE_FILE_LOC_ENV='/tmp/' \
-v /u01/app/oracle/extproc_libs:/u01/app/oracle/extproc_libs:Z \
-v /u01/app/oracle/extproc_logs:/u01/app/oracle/extproc_logs:Z \
--network=host \
--name adb-extproc \
ghcr.io/oracle/adb-extproc:latest
```
> Note the paths `/u01/app/oracle/extproc_logs` 
> and `/u01/app/oracle/extproc_libs` on the host VM are mounted in the container.

## Port Mapping

Note the following ports which are forwarded to the container process

| Port  | Description                          |
|-------|--------------------------------------|
| 16000 | TLS                                  |

## Configuration

Following table explains the environment variables passed to the container

| Environment variable | Description                                                                                                                                                                                         |
|----------------------|-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| EXTPROC_WALLET_PASSWORD       | Wallet is generated using the passed wallet password. Wallet password must have a minimum length of eight characters and contain alphabetic characters combined with numbers or special characters. |
| EXTPROC_DLLS       | Comma separated list of shared objects extproc is allowed to invoke. All custom libs should be accessible from within the container at location `/u01/app/oracle/extproc_libs`                      |
| TCP_INVITED_NODES       | ADB-S Private Endpoint (PE) IP address                                                                                                                                                              |
| SCRIPTS_FOLDER_LOC_ENV      | Location of custom scripts in the container. Default value is `/tmp` folder. You can copy scripts using `podman cp <myscript.sh> <containerid>:/tmp/`                                               |
| TRACE_FILE_LOC_ENV   | Every extproc invocation generates a trace file at this location. Default value is `/tmp` folder                                                                                                    |

## Wallet setup

In the container, TLS wallet is generated at location `/u01/app/oracle/wallets/extproc_wallet`

Copy wallet to your host.

```bash
rm -rf /scratch/extproc_wallet
podman cp adb-extproc:/u01/app/oracle/wallets/extproc_wallet /scratch/extproc_wallet
```
In this example, wallet is copied to `/scratch/extproc_wallet` folder on your machine

You can upload this wallet to Object store and download it using `DBMS_CLOUD.GET_OBJECT` to make it accessible to your Autonomous Database instance

```sql

   BEGIN
     DBMS_CLOUD.GET_OBJECT(
       credential_name => 'my_object_storage_creds',
       object_uri => '<object_storage_wallet_uri>',
       directory_name => 'MY_WALLET_DIR',
       file_name => 'cwallet.sso'
     );
   END;
   /

```

## `DBMS_CLOUD_FUNCTION` documentation

Once the container is up and running, refer to `DBMS_CLOUD_FUNCTION` [package documentation](https://docs.oracle.com/en-us/iaas/autonomous-database-serverless/doc/dbms-cloud-function.html#GUID-B9154A3C-A696-4C67-A7FE-F5A9FFECE87C)
to understand how to create a catalog and invoke functions remotely


## Contributing

This project welcomes contributions from the community. Before submitting a pull request, please [review our contribution guide](./CONTRIBUTING.md)

## Security

Please consult the [security guide](./SECURITY.md) for our responsible security vulnerability disclosure process

## License

Copyright (c) 2025 Oracle and/or its affiliates.

Released under the Universal Permissive License v1.0 as shown at
<https://oss.oracle.com/licenses/upl/>.
