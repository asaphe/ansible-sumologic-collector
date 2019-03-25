# Sumo Logic Collector

This role installs and configures a Sumo Logic Collector using apt to install the package.  

**Important**: This document contains the options used in the role. for a more complete reference please refer the [official documentation](https://help.sumologic.com/)

[Local File source JSON specifications](https://help.sumologic.com/03Send-Data/Sources/03Use-JSON-to-Configure-Sources/JSON-Parameters-for-Installed-Sources#Local_file_source).

**Locations:**

|Description            |Location                  |
|:---------------------:|:------------------------:|
|Collector installation |/opt/SumoCollector        |
|Collector configuration|/opt/SumoCollector/config/|

>`user.properties` file is located in the Collector configuration directory

**Requires:**

- root permissions (Installation)
- Internet Connection (Package download)
- Sumo Logic API key (Configure Collector)

>There are multiple ways to collect logs & metrics using Sumo Logic. going forward we should re-examine our initial solution  
>Docker: json-file logging driver when collecting logs with Sumo or use Sumo's logging driver  
>*other workarounds* collect json-logs directly from FS

## Immutable Properties

For collector versions 19.137 and later, the user.properties file lets you pass configuration parameters during the installation of a new unregistered Collector. Once the collector is registered, to see if a parameter can be changed with a collector restart, check the "Can be changed after installation?" column of the table below.

*No*: cannot be changed (Requires a removal of the collector in Sumo Logic and recreating it)  
*API*: must use the API to change  
*Edit*: can be changed via the UI or API  

>**NOTE On "API" Changes**: When updating a collector via the API ensure you've also re-deployed `user.properties` with matching settings to avoid confusion or cases where the collector was deleted in Sumo Logic and re-registered with different values.  
>**Side note on re-registration**: collector who had their access key & id removed cannot re-register unless we re-deploy (we ensure those secrets are not saved past the initial registration)

|Name|Description|Can be changed after installation?|
|:--:|:---------:|:--------------------------------:|
|fields=[list of fields]|Comma-separated list of key=value fields. To assign an ingest budget use the field _budget with its Field Value.|No, use the Collector Management API to modify.|
|accessid=accessId|Access ID used when logging in with Access ID and Key|No|
|accesskey=accessKey|Access Key used when logging in with Access ID and Key|No|
|category=category|Source category to use when a source does not specify a category.|No, use the Collector Management API to modify.|
|clobber=true/false|When true, if there is any existing collector with the same name, that collector will be deleted (clobbered).|No, use the Collector Management API to modify.|
|description=description|Description for the collector to appear in Sumo Logic.|No, use the Collector Management API to modify.|
|ephemeral=true/false|When true, the collector will be deleted after 12 hours of inactivity.|No, use the Collector Management API to modify|
|hostName=hostname|The host name of the machine on which the collector is running. The host name can be a maximum of 128 characters.|No, use the Collector Management API to modify.|
|name=name|Sets the name of collector used on Sumo Logic.|No, use Edit the Collector or the Collector Management API to modify|
|skipAccessKeyRemoval=true/false|If true, it will skip the access key removal from the user.properties file.|No|
|targetCPU=target|You can choose to set a CPU target to limit the amount of CPU processing a collector uses. The value must be expressed as a whole number percentage.|No, use the Collector Management API to modify.|
|timeZone=timezone|The time zone to use when it cannot be extracted|No, use the Collector Management API to modify.|

**Other settings not specified above**: Yes, with Collector restart.

>**Access key**: Note that, as of Collector v19.182-17, accesskey is automatically removed from user.properties following successful installation. (This behavior can be disabled with the skipAccessKeyRemoval property, described below.)
If you configure a collector to be ephemeral, in the event that the collector is de-registered after 12 hours offline, you will need to re-add the accesskey to user.properties.

|Name|Description|Can be changed after installation?|
|:--:|:---------:|:--------------------------------:|
|sources=filepath or folder path|Specifies a single UTF-8 encoded JSON file, or a folder containing UTF-8 encoded JSON files, that defines the Sources to configure upon Collector registration. **The contents of the file or files is read upon Collector registration only**, it is not synchronized with the Collector's configuration on an on-going basis.|No|
|syncSources=filepath or folder path|Specifies either a single UTF-8 encoded JSON file, or a folder containing UTF-8 encoded JSON files, that define the Sources to configure upon Collector registration. The Source definitions will be continuously monitored and synchronized with the Collector's configuration.|Yes, with Collector restart.|

>**SynSources** is how we've chosen to handle `sources` in order to easily change them (add/remove/modify) which would not otherwise be possible

### Local File

**Blacklist** In the Blacklist field, enter the path for files to exclude from the Source collection. wildcard syntax is allowed when specifying unwanted files. For example, you are collecting /var/log/*.log but don’t want to collect unwanted*.log, then specify /var/log/unwanted*.log in the blacklist. You can also exclude sub-directories, for example, if you are collecting /var/log/**/*.log but do not want to collect anything from /var/log/unwanted directory, specify /var/log/unwanted.

>You don't need to blacklist compressed files that end with the file extensions tar, bz2, z, zip, jar, war, 7z, rar, gz, or tar.gz. Sumo Logic, automatically excludes the those compressed file extensions when collecting data.

**Wildcards in paths**

```raw
/var/log/**                     will match all files in /var/log and all files in all child directories, recursively.
/var/log/**/*.log               will match all files whose names end in .log in /var/log and all files in all child directories, recursively.
/home/*/.bashrc                 will match all .bashrc files in all user's home directories.
/home/*/.ssh/**/*.key           will match all files ending in .key in all user's .ssh directories in all user's home directories.
```