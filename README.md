# Blobfuse2 - A Microsoft supported Azure Storage FUSE driver
## About
Blobfuse2 is an open source project developed to provide a virtual filesystem backed by the Azure Storage. It uses the libfuse open source library (fuse3) to communicate with the Linux FUSE kernel module, and implements the filesystem operations using the Azure Storage REST APIs.
This is the next generation [blobfuse](https://github.com/Azure/azure-storage-fuse)

Blobfuse2 is stable, and is ***supported by Microsoft*** provided that it is used within its limits documented here. Blobfuse2 supports both reads and writes however, it does not guarantee continuous sync of data written to storage using other APIs or other mounts of Blobfuse2. For data integrity it is recommended that multiple sources do not modify the same blob/file. Please submit an issue [here](https://github.com/azure/azure-storage-fuse/issues) for any issues/feature requests/questions.

## Features
- Mount an Azure storage blob container or datalake file system on Linux.
- Basic file system operations such as mkdir, opendir, readdir, rmdir, open, 
   read, create, write, close, unlink, truncate, stat, rename
- Local caching to improve subsequent access times
- Streaming to support reading AND writing large files 
- Parallel downloads and uploads to improve access time for large files
- Multiple mounts to the same container for read-only workloads

## _New BlobFuse2 Health Monitor_
One of the biggest BlobFuse2 features is our brand new health monitor. It allows customers gain more insight into how their BlobFuse2 instance is behaving with the rest of their machine. Visit [here](https://github.com/Azure/azure-storage-fuse/blob/main/tools/health-monitor/README.md) to set it up.

## Distinctive features compared to blobfuse (v1.x)
- Blobfuse2 is fuse3 compatible (other than Ubuntu-18 and Debian-9, where it still runs with fuse2)
- Support for higher service version offering latest and greatest of azure storage features (supported by azure go-sdk)
- Set blob tier while uploading the data to storage
- Attribute cache invalidation based on timeout
- For flat namesepce accounts, user can configure default permissions for files and folders
- Improved cache eviction algorithm for file cache to control disk footprint of blobfuse2
- Improved cache eviction algorithm for streamed buffers to control memory footprint of blobfuse2
- Utility to convert blobfuse CLI and config parameters to a blobfuse2 compatible config for easy migration
- CLI to mount Blobfuse2 with legacy Blobfuse config and CLI parameters (Refer to Migration guide for this)
- Version check and upgrade prompting 
- Option to mount a sub-directory from a container 
- CLI to mount all containers (with a allowlist and denylist) in a given storage account
- CLI to list all blobfuse2 mount points
- CLI to unmount one, multiple or all blobfuse2 mountpoints
- Option to dump logs to syslog or a file on disk
- Support for config file encryption and mounting with an encrypted config file via a passphrase (CLI or environment variable) to decrypt the config file
- CLI to check or update a parameter in the encrypted config
- Set MD5 sum of a blob while uploading
- Validate MD5 sum on download and fail file open on mismatch
- Large file writing through write streaming

 ## Blobfuse2 performance compared to blobfuse(v1.x.x)
- 'git clone' operation is 25% faster (tested with vscode repo cloning)
- ResNet50 image classification job is 7-8% faster (tested with 1.3 million images)
- Regular file uploads are 10% faster
- Verified listing of 1-Billion files in a directory (which v1.x does not support)


## Download Blobfuse2
You can install Blobfuse2 by cloning this repository. In the workspace root execute `go build` to build the binary. 

<!-- ## Find Help
For complete guidance, visit any of these articles
* Blobfuse2 Wiki -->

## Supported Operations
The general format of the Blobfuse2 commands is `blobfuse2 [command] [arguments] --[flag-name]=[flag-value]`
* `help` - Help about any command
* `mount` - Mounts an Azure container as a filesystem. The supported containers include
  - Azure Blob Container
  - Azure Datalake Gen2 Container
* `mount all` - Mounts all the containers in an Azure account as a filesystem. The supported storage services include
  - [Blob Storage](https://docs.microsoft.com/en-us/azure/storage/blobs/storage-blobs-introduction)
  - [Datalake Storage Gen2](https://docs.microsoft.com/en-us/azure/storage/blobs/data-lake-storage-introduction)
* `mount list` - Lists all Blobfuse2 filesystems.
* `secure decrypt` - Decrypts a config file.
* `secure encrypt` - Encrypts a config file.
* `secure get` - Gets value of a config parameter from an encrypted config file.
* `secure set` - Updates value of a config parameter.
* `unmount` - Unmounts the Blobfuse2 filesystem.
* `unmount all` - Unmounts all Blobfuse2 filesystems.

## Find help from your command prompt
To see a list of commands, type `blobfuse2 -h` and then press the ENTER key.
To learn about a specific command, just include the name of the command (For example: `blobfuse2 mount -h`).

## Usage
- Mount with blobfuse2
    * blobfuse2 mount <mount path> --config-file=<config file>
- Mount blobfuse2 using legacy blobfuse config and cli parameters
    * blobfuse2 mountv1 <blobfuse mount cli with options>
- Mount all containers in your storage account
    * blobfuse2 mount all <mount path> --config-file=<config file>
- List all mount instances of blobfuse2
    * blobfuse2 mount list
- Unmount blobfuse2
    * sudo fusermount3 -u <mount path>
- Unmount all blobfuse2 instances
    * blobfuse2 unmount all 

<!---TODO Add Usage for mount, unmount, etc--->
## CLI parameters
- Note: Blobfuse2 accepts all CLI parameters that Blobfuse does, but may ignore parameters that are no longer applicable. 
- General options
    * `--config-file=<PATH>`: The path to the config file.
    * `--log-level=<LOG_*>`: The level of logs to capture.
    * `--log-file-path=<PATH>`: The path for the log file.
    * `--foreground=true`: Mounts the system in foreground mode.
    * `--read-only=true`: Mount container in read-only mode.
    * `--default-working-dir`: The default working directory to store log files and other blobfuse2 related information.
    * `--disable-version-check=true`: Disable the blobfuse2 version check.
    * `----secure-config=true` : Config file is encrypted suing 'blobfuse2 secure` command.
    * `----passphrase=<STRING>` : Passphrase used to encrypt/decrypt config file.
- Attribute cache options
    * `--attr-cache-timeout=<TIMEOUT IN SECONDS>`: The timeout for the attribute cache entries.
    * `--no-symlinks=true`: To improve performance disable symlink support.
- Storage options
    * `--container-name=<CONTAINER NAME>`: The container to mount.
    * `--cancel-list-on-mount-seconds=<TIMEOUT IN SECONDS>`: Time for which list calls will be blocked after mount. ( prevent billing charges on mounting)
    * `--virtual-directory=true` : Support virtual directories without existence of a special marker blob for block blob account.
- File cache options
    * `--file-cache-timeout=<TIMEOUT IN SECONDS>`: Timeout for which file is cached on local system.
    * `--tmp-path=<PATH>`: The path to the file cache.
    * `--cache-size-mb=<SIZE IN MB>`: Amount of disk cache that can be used by blobfuse.
    * `--high-disk-threshold=<PERCENTAGE>`: If local cache usage exceeds this, start early eviction of files from cache.
    * `--low-disk-threshold=<PERCENTAGE>`: If local cache usage comes below this threshold then stop early eviction.
- Stream options
    * `--block-size-mb=<SIZE IN MB>`: Size of a block to be downloaded during streaming.
- Fuse options
    * `--attr-timeout=<TIMEOUT IN SECONDS>`: Time the kernel can cache inode attributes.
    * `--entry-timeout=<TIMEOUT IN SECONDS>`: Time the kernel can cache directory listing.
    * `--negative-timeout=<TIMEOUT IN SECONDS>`: Time the kernel can cache non-existance of file or directory.
    * `--allow-other`: Allow other users to have access this mount point.
    * `--disable-writeback-cache=true`: Disallow libfuse to buffer write requests if you must strictly open files in O_WRONLY or O_APPEND mode.
    * `--ignore-open-flags=true`: Ignore the append and write only flag since O_APPEND and O_WRONLY is not supported with writeback caching.


## Environment variables
- General options
    * `AZURE_STORAGE_ACCOUNT`: Specifies the storage account to be connected.
    * `AZURE_STORAGE_ACCOUNT_TYPE`: Specifies the account type 'block' or 'adls'
    * `AZURE_STORAGE_ACCOUNT_CONTAINER`: Specifies the name of the container to be mounted
    * `AZURE_STORAGE_BLOB_ENDPOINT`: Specifies the blob endpoint to use. Defaults to *.blob.core.windows.net, but is useful for targeting storage emulators.
    * `AZURE_STORAGE_AUTH_TYPE`: Overrides the currently specified auth type. Case insensitive. Options: Key, SAS, MSI, SPN
- Account key auth:
    * `AZURE_STORAGE_ACCESS_KEY`: Specifies the storage account key to use for authentication.
- SAS token auth:
    * `AZURE_STORAGE_SAS_TOKEN`: Specifies the SAS token to use for authentication.
- Managed Identity auth:
    * `AZURE_STORAGE_IDENTITY_CLIENT_ID`: Only one of these three parameters are needed if multiple identities are present on the system.
    * `AZURE_STORAGE_IDENTITY_OBJECT_ID`: Only one of these three parameters are needed if multiple identities are present on the system.
    * `AZURE_STORAGE_IDENTITY_RESOURCE_ID`: Only one of these three parameters are needed if multiple identities are present on the system.
    * `MSI_ENDPOINT`: Specifies a custom managed identity endpoint, as IMDS may not be available under some scenarios. Uses the `MSI_SECRET` parameter as the `Secret` header.
    * `MSI_SECRET`: Specifies a custom secret for an alternate managed identity endpoint.
- Service Principal Name auth:
    * `AZURE_STORAGE_SPN_CLIENT_ID`: Specifies the client ID for your application registration
    * `AZURE_STORAGE_SPN_TENANT_ID`: Specifies the tenant ID for your application registration
    * `AZURE_STORAGE_AAD_ENDPOINT`: Specifies a custom AAD endpoint to authenticate against
    * `AZURE_STORAGE_SPN_CLIENT_SECRET`: Specifies the client secret for your application registration.
- Proxy Server:
    * `http_proxy`: The proxy server address. Example: `10.1.22.4:8080`.    
    * `https_proxy`: The proxy server address when https is turned off forcing http. Example: `10.1.22.4:8080`.

## Config file
- See [this](./sampleFileCacheConfig.yaml) sample config file.
- See [this](./setup/baseConfig.yaml) config file for a list and description of all possible configurable options in blobfuse2. 

***Please note: do not use quotations `""` for any of the config parameters***

## Frequently Asked Questions
- How do I generate a SAS with permissions for rename?
az cli has a command to generate a sas token. Open a command prompt and make sure you are logged in to az cli. Run the following command and the sas token will be displayed in the command prompt.
az storage container generate-sas --account-name <account name ex:myadlsaccount> --account-key <accountKey> -n <container name> --permissions dlrwac --start <today's date ex: 2021-03-26> --expiry <date greater than the current time ex:2021-03-28>
- Why do I get EINVAL on opening a file with WRONLY or APPEND flags?
To improve performance, Blobfuse2 by default enables writeback caching, which can produce unexpected behavior for files opened with WRONLY or APPEND flags, so Blobfuse2 returns EINVAL on open of a file with those flags. Either use disable-writeback-caching to turn off writeback caching (can potentially result in degraded performance) or ignore-open-flags (replace WRONLY with RDWR and ignore APPEND) based on your workload. 
- How to mount blobfuse2 inside a container?
Refer to 'docker' folder in this repo. It contains a sample 'Dockerfile'. If you wish to create your own container image, try 'buildandruncontainer.sh' script, it will create a container image and launch the container using current environment variables holding your storage account credentials.
 
## Un-Supported File system operations
- mkfifo : fifo creation is not supported by blobfuse2 and this will result in "function not implemented" error
- chown  : Change of ownership is not supported by Azure Storage hence Blobfuse2 does not support this.
- Creation of device files or pipes is not supported by Blobfuse2.
- Blobfuse2 does not support extended-attributes (x-attrs) operations

## Un-Supported Scenarios
- Blobfuse2 does not support overlapping mount paths. While running multiple instances of Blobfuse2 make sure each instance has a unique and non-overlapping mount point.
- Blobfuse2 does not support co-existance with NFS on same mount path. Behaviour in this case is undefined.
- For block blob accounts, where data is uploaded through other means, Blobfuse2 expects special directory marker files to exist in container. In absence of this
  few file operations might not work. For e.g. if you have a blob 'A/B/c.txt' then special marker files shall exists for 'A' and 'A/B', otherwise opening of 'A/B/c.txt' will fail.
  Once a 'ls' operation is done on these directories 'A' and 'A/B' you will be able to open 'A/B/c.txt' as well. Possible workaround to resolve this from your container is to either

  create the directory marker files manually through portal or run 'mkdir' command for 'A' and 'A/B' from blobfuse. Refer [me](https://github.com/Azure/azure-storage-fuse/issues/866) 
  for details on this.

## Limitations
- In case of BlockBlob accounts, ACLs are not supported by Azure Storage so Blobfuse2 will by default return success for 'chmod' operation. However it will work fine for Gen2 (DataLake) accounts.


### Syslog security warning
By default, Blobfuse2 will log to syslog. The default settings will, in some cases, log relevant file paths to syslog. 
If this is sensitive information, turn off logging or set log-level to LOG_ERR.  


## License
This project is licensed under MIT.
 
## Contributing
This project welcomes contributions and suggestions.  Most contributions 
require you to agree to a Contributor License Agreement (CLA) declaring 
that you have the right to, and actually do, grant us the rights to use 
your contribution. For details, visit https://cla.microsoft.com.

When you submit a pull request, a CLA-bot will automatically determine 
whether you need to provide a CLA and decorate the PR appropriately 
(e.g., label, comment). Simply follow the instructions provided by the 
bot. You will only need to do this once across all repos using our CLA.

This project has adopted the [Microsoft Open Source Code of Conduct](https://opensource.microsoft.com/codeofconduct/).
For more information see the [Code of Conduct FAQ](https://opensource.microsoft.com/codeofconduct/faq/) or
contact [opencode@microsoft.com](mailto:opencode@microsoft.com) with any additional questions or comments.

