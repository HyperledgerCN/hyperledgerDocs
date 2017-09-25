
| 原文 | 作者 | 审核修正 |
| --- | --- | --- |
| [原文](http://hyperledger-fabric-ca.readthedocs.io/en/latest/users-guide.html) | Yaocheng Ye | Ruijun Ou、Zhangjiong Xuan |


Fabric CA is a Certificate Authority for Hyperledger Fabric.

It provides features such as:

1. registration of identities, or connects to LDAP as the user registry;
2. issuance of Enrollment Certificates (ECerts);
3. issuance of Transaction Certificates (TCerts), providing both anonymity and unlinkability when transacting on a Hyperledger Fabric blockchain;
4. certificate renewal and revocation.

Fabric CA为Hyperledger Fabric行使证书机构的功能。

Fabric CA提供以下功能：

1. 身份注册，或者将连接到LDAP作为用户注册；
2. 颁发登录证书(ECerts)；
3. 颁发交易证书(TCerts)，保证链上交易的匿名性与不可连接性；
4. 证书续期与撤销

Fabric CA consists of both a server and a client component as described later in this document.

For developers interested in contributing to Fabric CA, see the Fabric CA repository for more information.

Fabric CA 包含一个服务端组件和一个客户端组件，稍后会进行介绍。

对贡献Fabric CA感兴趣的开发者，可以参考 [Fabric CA repository](https://github.com/hyperledger/fabric-ca)


## 概述

The diagram below illustrates how the Fabric CA server fits into the overall Hyperledger Fabric architecture.

下图说明了 Fabric CA 服务端如何在 Hyperledger Fabric 架构中发挥作用

![](img/fabric-ca.png)

There are two ways of interacting with a Fabric CA server: via the Fabric CA client or through one of the Fabric SDKs. All communication to the Fabric CA server is via REST APIs. See fabric-ca/swagger/swagger-fabric-ca.json for the swagger documentation for these REST APIs.

有两种方式与 Fabric CA 服务端交互：通过 Fabric CA 客户端，或者 Fabric SDK，所有与 Fabric CA 的交互都是通过 REST APIs 来实现的。REST APIs 的swagger说明文档见 *fabric-ca/swagger/swagger-fabric-ca.json* 

The Fabric CA client or SDK may connect to a server in a cluster of Fabric CA servers. This is illustrated in the top right section of the diagram. The client routes to an HA Proxy endpoint which load balances traffic to one of the fabric-ca-server cluster members. All Fabric CA servers in a cluster share the same database for keeping track of users and certificates. If LDAP is configured, the user information is kept in LDAP rather than the database.

Fabric CA 客户端或者 SDK 可能会连接到 Fabric CA 集群中某一个 Fabric CA 服务端，这一部分可以通过上图右上部分获得更好的理解。客户端连接的是一个 HA 代理节点，这个 HA 代理节点为 Fabric CA 集群作负载均衡。所有的 Fabric CA 服务端共享同一个数据库。数据库用来保存用户和证书信息。如果配置了 LDAP，那么用户信息将会保存在 LDAP 中，而不是数据库中。

## 入门

### 前置条件

- Go 1.7+ 或更高版本
- GOPATH 环境变量正确设置
- libtool 和 libtdhl-dev 这两个包安装好

以下命令安装 libtool

    # sudo apt install libtool libltdl-dev

了解更多有关 libtool 的信息，参考 [https://www.gnu.org/software/libtool/](https://www.gnu.org/software/libtool/)

了解更多有关 libtdhr-dev 的信息，参考 [https://www.gnu.org/software/libtool/manual/html_node/Using-libltdl.html](https://www.gnu.org/software/libtool/manual/html_node/Using-libltdl.html)

### 安装

以下命令会同时安装 fabric-ca-server 和 fabric-ca-client

    # go get -u github.com/hyperledger/fabric-ca/cmd/...

#### 原生启动服务器 

默认配置启动 fabric-ca-server

    # fabric-ca-server start -b admin:adminpw

The -b option provides the enrollment ID and secret for a bootstrap administrator. A default configuration file named fabric-ca-server-config.yaml is created in the local directory which can be customized.

*-b* 选项用来提供启动管理员的登录 ID 和密码。默认配置文件 *fabric-ca-server-config.yaml* 会自动在本地目录创建，这个配置文件可以自定义。

#### 通过 Docker 启动服务器

使用 docker-compose 来启动

    # cd $GOPATH/src/github.com/hyperledger/fabric-ca
    # make docker
    # cd docker/server
    # docker-compose up -d

hyperledger/fabric-ca docker 镜像包含 fabric-ca-server 和 the fabric-ca-client

### 体验 Fabric CA 命令行

这一部分提供 fabric-ca-server 和 fabric-ca-client 命令行的使用说明。另外的使用信息会在接下来的内容中提供。

fabric-ca-server

    Hyperledger Fabric Certificate Authority Server

    Usage:
        fabric-ca-server [command]

    Available Commands:
        init        Initialize the Fabric CA server
        start       Start the Fabric CA server

    Flags:
            --address string                         Listening address of Fabric CA server (default "0.0.0.0")
        -b, --boot string                            The user:pass for bootstrap admin which is required to build default config file
            --ca.certfile string                     PEM-encoded CA certificate file (default "ca-cert.pem")
            --ca.chainfile string                    PEM-encoded CA chain file (default "ca-chain.pem")
            --ca.keyfile string                      PEM-encoded CA key file (default "ca-key.pem")
        -n, --ca.name string                         Certificate Authority name
        -c, --config string                          Configuration file (default "fabric-ca-server-config.yaml")
            --csr.cn string                          The common name field of the certificate signing request to a parent Fabric CA server
            --csr.hosts stringSlice                  A list of space-separated host names in a certificate signing request to a parent Fabric CA server
            --csr.serialnumber string                The serial number in a certificate signing request to a parent Fabric CA server
            --db.datasource string                   Data source which is database specific (default "fabric-ca-server.db")
            --db.tls.certfiles stringSlice           PEM-encoded list of trusted certificate files
            --db.tls.client.certfile string          PEM-encoded certificate file when mutual authenticate is enabled
            --db.tls.client.keyfile string           PEM-encoded key file when mutual authentication is enabled
            --db.type string                         Type of database; one of: sqlite3, postgres, mysql (default "sqlite3")
        -d, --debug                                  Enable debug level logging
            --ldap.enabled                           Enable the LDAP client for authentication and attributes
            --ldap.groupfilter string                The LDAP group filter for a single affiliation group (default "(memberUid=%s)")
            --ldap.url string                        LDAP client URL of form ldap://adminDN:adminPassword@host[:port]/base
            --ldap.userfilter string                 The LDAP user filter to use when searching for users (default "(uid=%s)")
        -p, --port int                               Listening port of Fabric CA server (default 7054)
            --registry.maxenrollments int            Maximum number of enrollments; valid if LDAP not enabled
            --tls.certfile string                    PEM-encoded TLS certificate file for server's listening port (default "ca-cert.pem")
            --tls.clientauth.certfiles stringSlice   PEM-encoded list of trusted certificate files
            --tls.clientauth.type string             Policy the server will follow for TLS Client Authentication. (default "noclientcert")
            --tls.enabled                            Enable TLS on the listening port
            --tls.keyfile string                     PEM-encoded TLS key for server's listening port (default "ca-key.pem")
        -u, --url string                             URL of the parent Fabric CA server


    Use "fabric-ca-server [command] --help" for more information about a command.

fabric-ca-client

    # fabric-ca-client
    Hyperledger Fabric Certificate Authority Client

    Usage:
        fabric-ca-client [command]

    Available Commands:
        enroll      Enroll an identity
        getcacert   Get CA certificate chain
        reenroll    Reenroll an identity
        register    Register an identity
        revoke      Revoke an identity

    Flags:
        -c, --config string                Configuration file (default "$HOME/.fabric-ca-client/fabric-ca-client-config.yaml")
            --csr.cn string                The common name field of the certificate signing request
            --csr.hosts stringSlice        A list of space-separated host names in a certificate signing request
            --csr.serialnumber string      The serial number in a certificate signing request
        -d, --debug                        Enable debug level logging
            --enrollment.hosts string      Comma-separated host list
            --enrollment.label string      Label to use in HSM operations
            --enrollment.profile string    Name of the signing profile to use in issuing the certificate
            --id.affiliation string        The identity's affiliation
            --id.attr string               Attributes associated with this identity (e.g. hf.Revoker=true)
            --id.maxenrollments int        The maximum number of times the secret can be reused to enroll
            --id.name string               Unique name of the identity
            --id.secret string             The enrollment secret for the identity being registered
            --id.type string               Type of identity being registered (e.g. 'peer, app, user')
        -M, --mspdir string                Membership Service Provider directory (default "msp")
        -m, --myhost string                Hostname to include in the certificate signing request during enrollment (default "$HOSTNAME")
            --tls.certfiles stringSlice    PEM-encoded list of trusted certificate files
            --tls.client.certfile string   PEM-encoded certificate file when mutual authenticate is enabled
            --tls.client.keyfile string    PEM-encoded key file when mutual authentication is enabled
        -u, --url string                   URL of the Fabric CA server (default "http://localhost:7054")

    Use "fabric-ca-client [command] --help" for more information about a command.

Note that command line options that are string slices (lists) can be specified either by specifying the option with space-separated list elements or by specifying the option multiple times, each with a string value that make up the list. For example, to specify host1 and host2 for csr.hosts option, you can either pass –csr.hosts “host1 host2” or –csr.hosts host1 –csr.hosts host2

注意在命令行中需要给某个选项输入列表时，可以用空格分割，或者多次使用该选项。例如，指定`host1`和`host2`给csr.hosts选项，你可以用–csr.hosts “host1 host2”或者–csr.hosts host1 –csr.hosts host2

## 文件格式

### Fabric CA 服务端配置文件格式

A configuration file can be provided to the server using the -c or --config option. If the --config option is used and the specified file doesn’t exist, a default configuration file (like the one shown below) will be created in the specified location. However, if no config option was used, it will be created in the server’s home directory (see Fabric CA Server section more info).

配置文件可以通过 `-c` 或者 `--config` 选项来指定。如果 `--config` 选项使用了，而指定的文件不存在，默认的配置文件（如下）会在指定的位置被创建。然而，如果没有指定配置文件，默认配置文件会在 fabric-ca-server 的 home 目录下创建。（参考 [Fabric CA 服务端](#fabric-ca-服务端) 了解更多）

    # Server's listening port (default: 7054)
    port: 7054

    # Enables debug logging (default: false)
    debug: false

    #############################################################################
    #  TLS section for the server's listening port
    #############################################################################
    tls:
        # Enable TLS (default: false)
        enabled: false
        certfile: ca-cert.pem
        keyfile: ca-key.pem

    #############################################################################
    #  The CA section contains the key and certificate files used when
    #  issuing enrollment certificates (ECerts) and transaction
    #  certificates (TCerts).
    #############################################################################
    ca:
        # Certificate file (default: ca-cert.pem)
        certfile: ca-cert.pem
        # Key file (default: ca-key.pem)
        keyfile: ca-key.pem

    #############################################################################
    #  The registry section controls how the Fabric CA server does two things:
    #  1) authenticates enrollment requests which contain identity name and
    #     password (also known as enrollment ID and secret).
    #  2) once authenticated, retrieves the identity's attribute names and
    #     values which the Fabric CA server optionally puts into TCerts
    #     which it issues for transacting on the Hyperledger Fabric blockchain.
    #     These attributes are useful for making access control decisions in
    #     chaincode.
    #  There are two main configuration options:
    #  1) The Fabric CA server is the registry
    #  2) An LDAP server is the registry, in which case the Fabric CA server
    #     calls the LDAP server to perform these tasks.
    #############################################################################
    registry:
        # Maximum number of times a password/secret can be reused for enrollment
        # (default: 0, which means there is no limit)
        maxEnrollments: 0

        # Contains identity information which is used when LDAP is disabled
        identities:
            - name: <<<ADMIN>>>
            pass: <<<ADMINPW>>>
            type: client
            affiliation: ""
            attrs:
                hf.Registrar.Roles: "client,user,peer,validator,auditor,ca"
                hf.Registrar.DelegateRoles: "client,user,validator,auditor"
                hf.Revoker: true
                hf.IntermediateCA: true

    #############################################################################
    #  Database section
    #  Supported types are: "sqlite3", "postgres", and "mysql".
    #  The datasource value depends on the type.
    #  If the type is "sqlite3", the datasource value is a file name to use
    #  as the database store.  Since "sqlite3" is an embedded database, it
    #  may not be used if you want to run the Fabric CA server in a cluster.
    #  To run the Fabric CA server in a cluster, you must choose "postgres"
    #  or "mysql".
    #############################################################################
    db:
        type: sqlite3
        datasource: fabric-ca-server.db
        tls:
            enabled: false
            certfiles:
                - db-server-cert.pem
            client:
                certfile: db-client-cert.pem
                keyfile: db-client-key.pem

    #############################################################################
    #  LDAP section
    #  If LDAP is enabled, the Fabric CA server calls LDAP to:
    #  1) authenticate enrollment ID and secret (i.e. identity name and password)
    #     for enrollment requests
    #  2) To retrieve identity attributes
    #############################################################################
    ldap:
        # Enables or disables the LDAP client (default: false)
        enabled: false
        # The URL of the LDAP server
        url: ldap://<adminDN>:<adminPassword>@<host>:<port>/<base>
        tls:
            certfiles:
                - ldap-server-cert.pem
            client:
                certfile: ldap-client-cert.pem
                keyfile: ldap-client-key.pem

    #############################################################################
    #  Affiliation section
    #############################################################################
    affiliations:
        org1:
            - department1
            - department2
        org2:
            - department1

    #############################################################################
    #  Signing section
    #############################################################################
    signing:
        profiles:
            ca:
                usage:
                - cert sign
                expiry: 8000h
                caconstraint:
                isca: true
        default:
            usage:
                - cert sign
            expiry: 8000h

    ###########################################################################
    #  Certificate Signing Request section for generating the CA certificate
    ###########################################################################
    csr:
        cn: fabric-ca-server
        names:
            - C: US
                ST: North Carolina
                L:
                O: Hyperledger
                OU: Fabric
        hosts:
            - <<<MYHOST>>>
        ca:
            pathlen:
            pathlenzero:
            expiry:

    #############################################################################
    #  Crypto section configures the crypto primitives used for all
    #############################################################################
    crypto:
        software:
            hash_family: SHA2
            security_level: 256
            ephemeral: false
            key_store_dir: keys

### Fabric CA 客户端配置文件格式

A configuration file can be provided to the client using the -c or --config option. If the config option is used and the specified file doesn’t exist, a default configuration file (like the one shown below) will be created in the specified location. However, if no config option was used, it will be created in the client’s home directory (see Fabric CA Client section more info).

配置文件可以通过 `-c` 或者 `--config` 选项来指定。如果 `--config` 选项使用了，而指定的文件不存在，默认的配置文件（如下）会在指定的位置被创建。然而，如果没有指定配置文件，默认配置文件会在 fabric-ca-client 的 home 目录下创建。（参考 [Fabric CA 客户端](#fabric-ca-客户端) 了解更多）

    #############################################################################
    # Client Configuration
    #############################################################################

    # URL of the fabric-ca-server (default: http://localhost:7054)
    URL: http://localhost:7054

    # Membership Service Provider (MSP) directory
    # When the client is used to enroll a peer or an orderer, this field must be
    # set to the MSP directory of the peer/orderer
    MSPDir:

    #############################################################################
    #    TLS section for secure socket connection
    #############################################################################
    tls:
        # Enable TLS (default: false)
        enabled: false
        certfiles:   # Comma Separated (e.g. root.pem, root2.pem)
        client:
            certfile:
            keyfile:

    #############################################################################
    #  Certificate Signing Request section for generating the CSR for
    #  an enrollment certificate (ECert)
    #############################################################################
    csr:
        cn: <<<ENROLLMENT_ID>>>
        names:
            - C: US
            ST: North Carolina
            L:
            O: Hyperledger
            OU: Fabric
        hosts:
        - <<<MYHOST>>>
        ca:
            pathlen:
            pathlenzero:
            expiry:

    #############################################################################
    #  Registration section used to register a new user with fabric-ca server
    #############################################################################
    id:
        name:
        type:
        affiliation:
        attrs:
            - name:
            value:

    #############################################################################
    #  Enrollment section used to enroll a user with fabric-ca server
    #############################################################################
    enrollment:
        hosts:
        profile:
        label:

## 配置优先级说明

The Fabric CA provides 3 ways to configure settings on the Fabric CA server and client. The precedence order is:

1. CLI flags
2. Environment variables
3. Configuration file

In the remainder of this document, we refer to making changes to configuration files. However, configuration file changes can be overridden through environment variables or CLI flags.

For example, if we have the following in the client configuration file:

Fabric CA 提供3种方式来配置 fabric-ca-server 和 fabric-ca-client 。优先级如下：

1. 命令行参数
2. 环境变量
3. 配置文件

接下来，我们试着对配置文件进行修改。配置文件的修改可以被环境变量或者命令行参数覆盖。

举个例子，假设我们有以下客户端的配置文件：

    tls:
        # Enable TLS (default: false)
        enabled: false

        # TLS for the client's listenting port (default: false)
        certfiles:   # Comma Separated (e.g. root.pem, root2.pem)
        client:
            certfile: cert.pem
            keyfile:

The following environment variable may be used to override the cert.pem setting in the configuration file:

`export FABRIC_CA_CLIENT_TLS_CLIENT_CERTFILE=cert2.pem`

If we wanted to override both the environment variable and configuration file, we can use a command line flag.

`fabric-ca-client enroll --tls.client.certfile cert3.pem`

The same approach applies to fabric-ca-server, except instead of using FABIRC_CA_CLIENT as the prefix to environment variables, FABRIC_CA_SERVER is used.

可以用以下环境变量来覆盖配置文件中 `cert.pem` 的配置

`export FABRIC_CA_CLIENT_TLS_CLIENT_CERTFILE=cert2.pem`

如果我们想同时覆盖环境变量和配置文件，可以通过命令行参数

`fabric-ca-client enroll --tls.client.certfile cert3.pem`

以上方法对fabric-ca-server同样适用，区别是在环境变量的前缀，把`FABIRC_CA_CLIENT`替换为`FABRIC_CA_SERVER`。

### 关于路径的一些说明

All the properties in the Fabric CA server and client configuration file, that specify file names support both relative and absolute paths. Relative paths are relative to the config directory, where the configuration file is located. For example, if the config directory is ~/config and the tls section is as shown below, the Fabric CA server or client will look for the root.pem file in the ~/config directory, cert.pem file in the ~/config/certs directory and the key.pem file in the /abs/path directory

fabric-ca-server 和 fabirc-ca-client 的配置文件里的所有属性都支持相对路径和绝对路径。相对路径是相对于配置目录，即配置文件所在的目录。比如，如果配置目录是 `~/config` ，而 tls 部分的配置如下所示，那么Fabric CA 服务端或者客户端会在 `~/config` 目录下查找 `root.pem` ，在 `~/config／certs` 下查找 `cert.pem` ，在 `/abs/path` 目录下查找 `key.pem`

    tls:
        enabled: true
        certfiles:   root.pem
        client:
            certfile: certs/cert.pem
            keyfile: /abs/path/key.pem

## Fabric CA 服务端

This section describes the Fabric CA server.

You may initialize the Fabric CA server before starting it. This provides an opportunity for you to generate a default configuration file but to review and customize its settings before starting it.

The Fabric CA server’s home directory is determined as follows:
- if the FABRIC_CA_SERVER_HOME environment variable is set, use its value;
- otherwise, if FABRIC_CA_HOME environment variable is set, use its value;
- otherwise, if the CA_CFG_PATH environment variable is set, use its value;
- otherwise, use current working directory.

For the remainder of this server section, we assume that you have set the FABRIC_CA_HOME environment variable to $HOME/fabric-ca/server.

The instructions below assume that the server configuration file exists in the server’s home directory.

这一部分说明Fabric CA服务端。

在启动Fabric CA服务端之前，你可以先初始化Fabric CA服务端。通过初始化，程序会自动生成一份默认配置文件，方便用户在启动前自行修改一些配置项。

Fabric CA服务端的根目录通过以下方式决定：
- 如果存在环境变量 `FABRIC_CA_SERVER_HOME` ，则使用这个值
- 否则，如果存在环境变量 `FABRIC_CA_HOME` ，则使用这个值
- 否则，如果存在环境变量 `CA_CFG_PATH` ，则使用这个值
- 否则，使用当前的工作目录作为根目录

本章接下来的部分，我们假设你已经设置了环境变量 `FABRIC_CA_HOME` ，并且值设置为 `$HOME/fabric-ca/server`。

接下来的内容都默认服务端配置文件存在于服务端根目录下。

### 初始化服务端

用以下命令初始化Fabric CA服务端

    # fabric-ca-server init -b admin:adminpw

The `-b` (bootstrap identity) option is required for initialization. At least one bootstrap identity is required to start the Fabric CA server. The server configuration file contains a Certificate Signing Request (CSR) section that can be configured. The following is a sample CSR.

初始化时必须提供 `-b` (bootstrap identity: 引导身份) 选项。启动服务端时，至少需要提供一个引导身份。服务端配置文件包含一个可以配置的证书签名请求 (Certificate Signing Request, CSR) 部分。下面是一个CSR的例子。

If you are going to connect to the Fabric CA server remotely over TLS, replace “localhost” in the CSR section below with the hostname where you will be running your Fabric CA server.

如果你想通过TLS远程连接Fabric CA服务端，把下面的CSR配置内容中的"localhost"替换成你将会运行Fabric CA服务端的域名。

    cn: localhost
    key:
        algo: ecdsa
        size: 256
    names:
    - C: US
        ST: "North Carolina"
        L:
        O: Hyperledger
        OU: Fabric

All of the fields above pertain to the X.509 signing key and certificate which is generated by the `fabric-ca-server` init. This corresponds to the `ca.certfile` and `ca.keyfile` files in the server’s configuration file. The fields are as follows:

- cn is the Common Name
- key specifies the algorithm and key size as described below
- O is the organization name
- OU is the organizational unit
- L is the location or city
- ST is the state
- C is the country

上面所有的字段都符合X.509签名与证书规范，可以由`fabric-ca-server` init命令来生成。服务端配置文件中提到的`ca.certfile`和`ca.keyfile`这两个文件也符合X.509规范。字段如下：

- cn 通用名
- key 指明算法和密钥长度
- O 组织
- OU 组织单位
- L 地址或城市
- ST 州（省）
- C 国家

If custom values for the CSR are required, you may customize the configuration file, delete the files specified by the ca.certfile and ca-keyfile configuration items, and then run the fabric-ca-server init -b admin:adminpw command again.

如果需要修改CSR里面的值，你可以修改配置文件，然后把配置中由`ca.certfile`和`ca.keyfile`这两项指明的文件删除。然后再运行一次 `fabric-ca-server init -b admin:adminpw` 命令。

The `fabric-ca-server init` command generates a self-signed CA certificate unless the `-u \<parent-fabric-ca-server-URL\>` option is specified. If the `-u` is specified, the server’s CA certificate is signed by the parent Fabric CA server. The `fabric-ca-server init` command also generates a default configuration file named fabric-ca-server-config.yaml in the server’s home directory.

`fabirc-ca-server init` 命令生成一个自签名的CA证书，除非使用了 `-u <parent-fabric-ca-server-URL>` 选项。如果使用了 `-u` 选项，本服务端CA证书会由父Fabric CA服务端签名。`fabirc-ca-server init` 命令同时也会在服务端的根目录生成一个默认配置文件*fabric-ca-server-config.yaml*。

算法和密钥长度

The CSR can be customized to generate X.509 certificates and keys that support both RSA and Elliptic Curve (ECDSA). The following setting is an example of the implementation of Elliptic Curve Digital Signature Algorithm (ECDSA) with curve prime256v1 and signature algorithm ecdsa-with-SHA256:

可以通过自定义CSR来生成支持ECDSA和RSA的X.509证书和密钥。以下是一个椭圆曲线数字签名算法（ECDSA）的设置，采用的曲线是`prime256v1`，签名算法是`ecdsa-with-SHA256`。

    key:
        algo: ecdsa
        size: 256

对算法和密钥长度的选择取决于你对安全的考量。

ECDSA提供以下选项：

| size | ASN1 OID | Signature Algorithm |
|:----:|:--------:|:-------------------:|
| 256  |prime256v1| ecdsa-with-SHA256   |
| 384  |secp384r1| ecdsa-with-SHA384   |
| 521  |secp384r1| ecdsa-with-SHA521   |

RSA提供以下选项：

| size | Modulus (bits) | Signature Algorithm |
|:----:|:--------:|:-------------------:|
| 2048 |2048| sha256WithRSAEncryption   |
| 4096 |2096| sha512WithRSAEncryption   |

### 启动服务端

启动Fabric CA服务器：

    # fabric-ca-server start -b <admin>:<adminpw>

If the server has not been previously initialized, it will initialize itself as it starts for the first time. During this initialization, the server will generate the ca-cert.pem and ca-key.pem files if they don’t yet exist and will also create a default configuration file if it does not exist. See the `Initialize the Fabric CA server` section.

如果服务器之前没有初始化过，它会在第一次启动的时候进行初始化。在初始化的过程中，它会如果ca-cert.pem和ca-key.pem这两个文件不存在，它会生成这两个文件；如果配置文件不存在，它会生成默认配置文件。参考[初始化服务端](#初始化服务端)。

Unless the Fabric CA server is configured to use LDAP, it must be configured with at least one pre-registered bootstrap identity to enable you to register and enroll other identities. The `-b` option specifies the name and password for a bootstrap identity.

除非Fabric CA服务端配置了使用LDAP，否则它必须配置至少一个预注册引导身份来允许你登录其他身份。`-b`选项指明引导身份的用户名和密码。

A different configuration file may be specified with the -c option as shown below.

可以通过`-c`选项来指明其他的配置文件。

    # fabric-ca-server start -c <path-to-config-file> -b <admin>:<adminpw>

To cause the Fabric CA server to listen on https rather than http, set tls.enabled to true.

为了使fabric ca server监听`https`而不是`http`，配置`tls.enabled`为`true`。

To limit the number of times that the same secret (or password) can be used for enrollment, set the `registry.maxEnrollments` in the configuration file to the appropriate value. If you set the value to 1, the Fabric CA server allows passwords to only be used once for a particular enrollment ID. If you set the value to 0, the Fabric CA server places no limit on the number of times that a secret can be reused for enrollment. The default value is 0.

为了限制登录时相同密码的次数，可以在配置文件中设置`registry.maxEnrollments`为恰当的值。如果你设置这个值为1，Fabric CA服务端只允许一个密码被一个登录ID使用（即不会出现多个ID有相同的密码）。如果你设置这个值为0，Fabric CA服务端不会限制密码的重复使用次数。默认值为0。

The Fabric CA server should now be listening on port 7054.

Fabric CA服务端现在应该正在监听7054端口。

You may skip to the Fabric CA Client section if you do not want to configure the Fabric CA server to run in a cluster or to use LDAP.

如果你不想配置Fabric CA服务端集群，也不想使用LDAP，你可以直接跳到[Fabric CA 客户端](#farbic-ca-客户端)这一章节。

### 配置数据库

This section describes how to configure the Fabric CA server to connect to Postgres or MySQL databases. The default database is SQLite and the default database file is `fabric-ca-server.db` in the Fabric CA server’s home directory.

这一部分讲解如何配置Fabric CA服务端连接到Postgres或者MySQL。默认的数据库是SQLite，默认的数据库文件是`fabric-ca-server.db`，存放在Fabric CA服务端的根目录。

If you don’t care about running the Fabric CA server in a cluster, you may skip this section; otherwise, you must configure either Postgres or MySQL as described below.

如果你不想运行Fabric CA服务端集群，你可以跳过这一章；不然的话你可以照下面的指引配置Postgres或者MySQL。

#### Postgres

The following sample may be added to the server’s configuration file in order to connect to a Postgres database. Be sure to customize the various values appropriately.

下面的例子可以添加到服务端的配置文件中，来使服务端连接到一个Postgres数据库。别忘了正确地自定义各种参数。

    db:
        type: postgres
        datasource: host=localhost port=5432 user=Username password=Password dbname=fabric-ca-server sslmode=verify-full

Specifying sslmode configures the type of SSL authentication. Valid values for sslmode are:

指定sslmode来配置SSL认证的类型。sslmode有效的值为：

If you would like to use TLS, then the db.tls section in the Fabric CA server configuration file must be specified. If SSL client authentication is enabled on the Postgres server, then the client certificate and key file must also be specified in the db.tls.client section. The following is an example of the db.tls section:

如果你想使用TLS，那么需要在配置文件中指明`db.tls`这一部分。如果Postgres服务器开启了SSL客户端认证，那么客户端的证书和密钥文件必须在`db.tls.client`这一部分指明。下面是`db.tls`部分的一个例子：

    db:
        ...
        tls:
            enabled: true
            certfiles:
                - db-server-cert.pem
            client:
                    certfile: db-client-cert.pem
                    keyfile: db-client-key.pem

*certfiles* - A list of PEM-encoded trusted root certificate files.

*certfile* and *keyfile* - PEM-encoded certificate and key files that are used by the Fabric CA server to communicate securely with the Postgres server

*certfiles* - 可信任的根证书文件列表，采用PEM编码

*certfile* and *keyfile* - 用于与Postgres服务器安全通信的证书和密钥文件，采用PEM编码

#### MySQL

The following sample may be added to the Fabric CA server configuration file in order to connect to a MySQL database. Be sure to customize the various values appropriately.

下面的例子可以添加到Fabric CA服务端配置文件，用来连接到MySQL数据库。别忘了正确地自定义各种参数。

    db:
        type: mysql
        datasource: root:rootpw@tcp(localhost:3306)/fabric-ca?parseTime=true&tls=custom

If connecting over TLS to the MySQL server, the db.tls.client section is also required as described in the Postgres section above.

如果要使用TLS，需要配置`db.tls.client`部分，参考Postgres部分。

### 配置LDAP

The Fabric CA server can be configured to read from an LDAP server.

In particular, the Fabric CA server may connect to an LDAP server to do the following:

- authenticate an identity prior to enrollment
- retrieve an identity’s attribute values which are used for authorization.

Modify the LDAP section of the Fabric CA server’s configuration file to configure the server to connect to an LDAP server.

Fabric CA服务端可以配置为连接到一个LDAP服务器。

特别地，Fabric CA服务端可以连接到一个LDAP服务器来做下面的事情：

- 登录前验证一个身份
- 授权时获取一个身份的属性值

在配置文件中修改LDAP的配置来连接到一个LDAP服务器。

    ldap:
        # Enables or disables the LDAP client (default: false)
        enabled: false
        # The URL of the LDAP server
        url: <scheme>://<adminDN>:<adminPassword>@<host>:<port>/<base>
        userfilter: filter

where:
- scheme is one of ldap or ldaps;
- adminDN is the distinquished name of the admin user;
- pass is the password of the admin user;
- host is the hostname or IP address of the LDAP server;
- port is the optional port number, where default 389 for ldap and 636 for ldaps;
- base is the optional root of the LDAP tree to use for searches;
- filter is a filter to use when searching to convert a login user name to a distinquished name. For example, a value of (uid=%s) searches for LDAP entries with the value of a uid attribute whose value is the login user name. Similarly, (email=%s) may be used to login with an email address.

其中：
- scheme ldap或者ldaps;
- adminDN 管理员的区别名;
- pass 管理员的密码;
- host LDAP服务器的域名或者IP;
- port 可选的端口号，ldap默认为389，ldaps默认为636;
- base 可选的LDAP树的根，用于搜索时;
- filter 搜索时的过滤器，把登陆用户名转换为一个区别名。比如，（uid=%s）会搜索uid值等于用户登录名的LDAP实体。类似地，（email=%s）可以用于邮箱地址作为用户名的登陆。

The following is a sample configuration section for the default settings for the OpenLDAP server whose docker image is at `https://github.com/osixia/docker-openldap`.

下面是一个配置例子，用于OpenLDAP的默认设置，OpenLDAP的docker镜像在`https://github.com/osixia/docker-openldap`。

    ldap:
        enabled: true
        url: ldap://cn=admin,dc=example,dc=org:admin@localhost:10389/dc=example,dc=org
        userfilter: (uid=%s)

See `FABRIC_CA/scripts/run-ldap-tests` for a script which starts an OpenLDAP docker image, configures it, runs the LDAP tests in `FABRIC_CA/cli/server/ldap/ldap_test.go`, and stops the OpenLDAP server.

在`FABRIC_CA/scripts/run-ldap-tests`有一个脚本，这个脚本能启动OpenLDAP的docker镜像，配置它，然后运行`FABRIC_CA/cli/server/ldap/ldap_test.go`里面的LDAP测试，最后停止OpenLDAP服务器。

When LDAP is configured, enrollment works as follows:

- The Fabric CA client or client SDK sends an enrollment request with a basic authorization header.
- The Fabric CA server receives the enrollment request, decodes the identity name and password in the authorization header, looks up the DN (Distinquished Name) associated with the identity name using the “userfilter” from the configuration file, and then attempts an LDAP bind with the identity’s password. If the LDAP bind is successful, the enrollment processing is authorized and can proceed.

当LDAP配置好后，登录的流程如下：

- Fabric CA客户端或者客户端SDK发送一个登录请求，带上basic方式的授权头。
- Fabric CA服务端收到登录请求，解码授权头里的身份名和密码，查找与身份名相关联的区别名，（关联方式为配置文件里的"userfilter"定义的），然后用身份密码尝试一个LDAP绑定。如果LDAP绑定成功，那么登录过程被批准了，能够继续。

When LDAP is configured, attribute retrieval works as follows:

- A client SDK sends a request for a batch of tcerts with one or more attributes to the Fabric CA server.
- The Fabric CA server receives the tcert request and does as follows:
  - extracts the enrollment ID from the token in the authorization header (after validating the token);
  - does an LDAP search/query to the LDAP server, requesting all of the attribute names received in the tcert request;
  - the attribute values are placed in the tcert as normal.

当LDAP配置好了，属性返回的流程如下：

- 客户端SDK用一个或多个属性发送一个请求到服务端，请求一批tcerts。
- 服务端收到tcert请求，做如下的事:
  - 在授权头取出登录ID（在验证token后）;
  - 做一次LDAP查询，向LDAP服务器请求tcert请求中的所有的属性名;
  - 属性值放置在tcert中。

### 构建一个集群

You may use any IP sprayer to load balance to a cluster of Fabric CA servers. This section provides an example of how to set up Haproxy to route to a Fabric CA server cluster. Be sure to change hostname and port to reflect the settings of your Fabric CA servers.

你可以使用任意IP代理来为Fabric CA服务端集群做负载均衡。这一节提供了一个例子来介绍如何使用Haproxy来为集群路由。别忘了修改域名和端口。

haproxy.conf

    global
        maxconn 4096
        daemon

    defaults
        mode http
        maxconn 2000
        timeout connect 5000
        timeout client 50000
        timeout server 50000

    listen http-in
        bind *:7054
        balance roundrobin
        server server1 hostname1:port
        server server2 hostname2:port
        server server3 hostname3:port

注意：如果使用TLS，需要使用`mode tcp`

## Farbic CA 客户端

This section describes how to use the fabric-ca-client command.

The Fabric CA client’s home directory is determined as follows:

- if the `FABRIC_CA_CLIENT_HOME` environment variable is set, use its value;
- otherwise, if the `FABRIC_CA_HOME` environment variable is set, use its value;
- otherwise, if the `CA_CFG_PATH` environment variable is set, use its value;
- otherwise, use `$HOME/.fabric-ca-client`.

The instructions below assume that the client configuration file exists in the client’s home directory.

这一部分讲解如何使用fabric-ca-client的命令。

Fabric CA客户端的根目录定义规则如下：

- 如果存在环境变量 `FABRIC_CA_CLIENT_HOME` ，则使用这个值
- 否则，如果存在环境变量 `FABRIC_CA_HOME` ，则使用这个值
- 否则，如果存在环境变量 `CA_CFG_PATH` ，则使用这个值
- 否则，使用`$HOME/.fabric-ca-client`

下面的指引假设客户端的配置文件存在于客户端根目录。

### 登陆启动用户

First, if needed, customize the CSR (Certificate Signing Request) section in the client configuration file. Note that `csr.cn` field must be set to the ID of the bootstrap identity. Default CSR values are shown below:

首先，如果需要在配置文件自定义CSR（证书签名请求），注意`csr.cn`必须设置为引导身份的ID。默认CSR如下：

    csr:
        cn: <<enrollment ID>>
        key:
            algo: ecdsa
            size: 256
        names:
            - C: US
            ST: North Carolina
            L:
            O: Hyperledger Fabric
            OU: Fabric CA
        hosts:
        - <<hostname of the fabric-ca-client>>
        ca:
            pathlen:
            pathlenzero:
            expiry:

See CSR fields for description of the fields.

Then run `fabric-ca-client enroll` command to enroll the identity. For example, following command enrolls an identity whose ID is admin and password is adminpw by calling Fabric CA server that is running locally at 7054 port.

查看字段的描述，[CSR fields](#初始化服务端)

然后运行`fabric-ca-client enroll`命令来登录一个身份。比如，下面的命令向一个本地运行在7054端口的Fabric CA服务端，登录了一个ID为admin，password为adminpw的身份。

    # export FABRIC_CA_CLIENT_HOME=$HOME/fabric-ca/clients/admin
    # fabric-ca-client enroll -u http://admin:adminpw@localhost:7054

The enroll command stores an enrollment certificate (ECert), corresponding private key and CA certificate chain PEM files in the subdirectories of the Fabric CA client’s `msp` directory. You will see messages indicating where the PEM files are stored.

登录命令会存储一个登录证书（ECert），相对应的私钥，还有CA证书链PEM文件。这些存储在Fabric CA客户端的msp目录的子目录下，你会看到信息提示PEM存储在哪里。

### 注册一个新的身份

The identity performing the register request must be currently enrolled, and must also have the proper authority to register the type of the identity that is being registered.

只有已经登录了的身份才能发起注册的请求，而且必须有相应的权限来注册想要注册的身份类型。

In particular, two authorization checks are made by the Fabric CA server during registration as follows:

1. The invoker’s identity must have the “hf.Registrar.Roles” attribute with a comma-separated list of values where one of the value equals the type of identity being registered; for example, if the invoker’s identity has the “hf.Registrar.Roles” attribute with a value of “peer,app,user”, the invoker can register identities of type peer, app, and user, but not orderer.
2. The affiliation of the invoker’s identity must be equal to or a prefix of the affiliation of the identity being registered. For example, an invoker with an affiliation of “a.b” may register an identity with an affiliation of “a.b.c” but may not register an identity with an affiliation of “a.c”.

The following command uses the admin identity’s credentials to register a new identity with an enrollment id of “admin2”, a type of “user”, an affiliation of “org1.department1”, and an attribute named “hf.Revoker” with a value of “true”.

特别地，注册时Fabric CA服务端做两项权限检查：

1. 注册发起者的“hf.Registrar.Roles”属性中必须有请求注册的类型。举个例子，如果发起者的“hf.Registrar.Roles”属性的值为“peer,app,user”，那么他能注册的类型为peer，app和user，不能注册orderer。
2. 发起者的affiliation必须与他请求注册的身份的affiliation相同，或者是所请求注affiliation的前缀。举个例子，一个affiliation为“a.b”的发起者，可以注册一个affiliation为“a.b.c”的身份，但是不能注册一个affiliation为“a.c”的身份。

下面的命令使用admin身份的凭证来注册一个新的身份，登录ID是“admin2”，类型为“user”，affiliation为“org1.department1”，还有“hf.Revoker”属性为“true”。

    # export FABRIC_CA_CLIENT_HOME=$HOME/fabric-ca/clients/admin
    # fabric-ca-client register --id.name admin2 --id.type user --id.affiliation org1.department1 --id.attr hf.Revoker=true

The password, also known as the enrollment secret, is printed. This password is required to enroll the identity. This allows an administrator to register an identity and give the enrollment ID and the secret to someone else to enroll the identity.

密码会被打印出来，登录这个新注册的身份的时候，需要用到这个密码。这允许一个管理员注册身份，然后把这个身份的ID和密码给别人来登陆。

You may set default values for any of the fields used in the register command by editing the client’s configuration file. For example, suppose the configuration file contains the following:

你可以通过编辑配置文件来设置注册时一些字段的默认值。举个例子，假设配置文件包含下面的内容：

    id:
        name:
        type: user
        affiliation: org1.department1
        attributes:
            - name: hf.Revoker
            value: true
            - name: anotherAttrName
            value: anotherAttrValue

The following command would then register a new identity with an enrollment id of “admin3” which it takes from the command line, and the remainder is taken from the configuration file including the identity type: “user”, affiliation: “org1.department1”, and two attributes: “hf.Revoker” and “anotherAttrName”.

下面的命令会注册一个新的身份，id为admin3，其他的内容会从配置文件中读取出来。包括：类型“user”，affiliation “org1.department1”，还有两个属性，“hf.Revoker”和“anotherAttrName”。

    # export FABRIC_CA_CLIENT_HOME=$HOME/fabric-ca/clients/admin
    # fabric-ca-client register --id.name admin3

To register an identity with multiple attributes requires specifying all attribute names and values in the configuration file as shown above.

注册有多个属性的身份需要在配置文件中指明所有属性名和属性值，如上所示。

Next, let’s register a peer identity which will be used to enroll the peer in the following section. The following command registers the peer1 identity. Note that we choose to specify our own password (or secret) rather than letting the server generate one for us.

接下来，让我们注册一个节点身份，会在下面内容登陆节点的时候用到。下面的命令注册了一个peer1身份，在这里我们选择指明自己的密码，而不是由服务器生成。

    # export FABRIC_CA_CLIENT_HOME=$HOME/fabric-ca/clients/admin
    # fabric-ca-client register --id.name peer1 --id.type peer --id.affiliation org1.department1 --id.secret peer1pw

### 登录一个节点

Now that you have successfully registered a peer identity, you may now enroll the peer given the enrollment ID and secret (i.e. the password from the previous section). This is similar to enrolling the bootstrap identity except that we also demonstrate how to use the “-M” option to populate the Hyperledger Fabric MSP (Membership Service Provider) directory structure.

现在你成功地注册了一个节点身份，你可以用ID和密码登陆。这部分与登陆一个引导身份类似。我们还会介绍如何使用“-M”选项来更换MSP的目录。

The following command enrolls peer1. Be sure to replace the value of the “-M” option with the path to your peer’s MSP directory which is the ‘mspConfigPath’ setting in the peer’s core.yaml file. You may also set the FABRIC_CA_CLIENT_HOME to the home directory of your peer.

下面的命令登陆peer1。记得在“-M”选项下更改你自己的MSP目录，MSP目录是由节点的core.yaml里的“mspConfigPath”指定的。你也可以设置FABRIC_CA_CLIENT_HOME环境变量为peer的根目录。

    # export FABRIC_CA_CLIENT_HOME=$HOME/fabric-ca/clients/peer1
    # fabric-ca-client enroll -u http://peer1:peer1pw@localhost:7054 -M $FABRIC_CA_CLIENT_HOME/msp

Enrolling an orderer is the same, except the path to the MSP directory is the ‘LocalMSPDir’ setting in your orderer’s orderer.yaml file.

登陆一个orderer也是一样的，除了MSP目录是设置在你的orderer的orderer.yaml文件里的“LocalMSPDir”。

### 从另一个Fabric CA服务器获得CA证书链

In general, the cacerts directory of the MSP directory must contain the certificate authority chains of other certificate authorities, representing all of the roots of trust for the peer.

通常，MSP目录的ca证书目录必须包含证书链，代表这个节点所有信任的信任中心。

The fabric-ca-client getcacerts command is used to retrieve these certificate chains from other Fabric CA server instances.

`fabric-ca-client getcacerts`命令用于从其他Fabric CA服务器实例获取这些证书链。

For example, the following will start a second Fabric CA server on localhost listening on port 7055 with a name of “CA2”. This represents a completely separate root of trust and would be managed by a different member on the blockchain.

举个例子，下面的命令会在本地启动第二个Fabric CA服务器，监听7055端口，命名为“CA2“。这代表两个由不同成员管理的分开的信任中心。

    # export FABRIC_CA_SERVER_HOME=$HOME/ca2
    # fabric-ca-server start -b admin:ca2pw -p 7055 -n CA2

The following command will install CA2’s certificate chain into peer1’s MSP directory.

下面的命令会把CA2的证书链安装进peer1的MSP目录。

    # export FABRIC_CA_CLIENT_HOME=$HOME/fabric-ca/clients/peer1
    # fabric-ca-client getcacert -u http://localhost:7055 -M $FABRIC_CA_CLIENT_HOME/msp

### 重新登陆一个身份

Suppose your enrollment certificate is about to expire or has been compromised. You can issue the reenroll command to renew your enrollment certificate as follows.

假设你的登陆证书快过期了，你可以重新登陆来替换你的登陆证书（ECert）。

    # export FABRIC_CA_CLIENT_HOME=$HOME/fabric-ca/clients/peer1
    # fabric-ca-client reenroll

### 撤销一个证书或身份

An identity or a certificate can be revoked. Revoking an identity will revoke all the certificates owned by the identity and will also prevent the identity from getting any new certificates. Revoking a certificate will invalidate a single certificate.

身份和证书都能被撤销。撤销一个身份会撤销该身份拥有的所有证书，该身份也不能再获得新的证书。撤销一个证书会使该证书失效。

In order to revoke a certificate or an identity, the calling identity must have the hf.Revoker attribute. The revoking identity can only revoke a certificate or an identity that has an affiliation that is equal to or prefixed by the revoking identity’s affiliation.

为了撤销一个证书或身份，发起者必须有hf.Revoker属性。发起者只能撤销与自己的affiliation相同的证书或身份，或者发起者的affiliation是被撤销者的affiliation的前缀。

For example, a revoker with affiliation orgs.org1 can revoke an identity affiliated with orgs.org1 or orgs.org1.department1 but can’t revoke an identity affiliated with orgs.org2.

举个例子，一个“orgs.org1”的发起者只能撤销orgs.org1或者orgs.org1.department1的身份，而不能撤销orgs.org2的身份。

The following command disables an identity and revokes all of the certificates associated with the identity. All future requests received by the Fabric CA server from this identity will be rejected.

下面的命令撤销一个身份。将来所有发自该身份的请求都会被Fabric CA服务器拒收。

    fabric-ca-client revoke -e <enrollment_id> -r <reason>

The following are the supported reasons that can be specified using -r flag:

下面是`-r`选项支持的理由：

1. unspecified
2. keycompromise
3. cacompromise
4. affiliationchange
5. superseded
6. cessationofoperation
7. certificatehold
8. removefromcrl
9. privilegewithdrawn
10. aacompromise

For example, the bootstrap admin who is associated with root of the affiliation tree can revoke peer1‘s identity as follows:

举个例子，有着根affiliation的admin可以回收peer1身份：

    # export FABRIC_CA_CLIENT_HOME=$HOME/fabric-ca/clients/admin
    # fabric-ca-client revoke -e peer1

An enrollment certificate that belongs to an identity can be revoked by specifying its AKI (Authority Key Identifier) and serial number as follows:

一个身份可以撤销自己的登陆证书（ECert），需要指定ECert的AKI和序列号：

    fabric-ca-client revoke -a xxx -s yyy -r <reason>

For example, you can get the AKI and the serial number of a certificate using the openssl command and pass them to the revoke command to revoke the said certificate as follows:

举个例子，你可以通过openssl命令来获取一个证书的AKI和序列号：

    serial=$(openssl x509 -in userecert.pem -serial -noout | cut -d "=" -f 2)
    aki=$(openssl x509 -in userecert.pem -text | awk '/keyid/ {gsub(/ *keyid:|:/,"",$1);print tolower($0)}')
    fabric-ca-client revoke -s $serial -a $aki -r affiliationchange

### 启用TLS

This section describes in more detail how to configure TLS for a Fabric CA client.

这一部分介绍如何为Fabric CA客户端配置TLS。

The following sections may be configured in the `fabric-ca-client-config.yaml`.

下面的可以配置在`fabric-ca-client-config.yaml`中。

    tls:
        # Enable TLS (default: false)
        enabled: true
        certfiles:
            - root.pem
        client:
            certfile: tls_client-cert.pem
            keyfile: tls_client-key.pem
                
The certfiles option is the set of root certificates trusted by the client. This will typically just be the root Fabric CA server’s certificate found in the server’s home directory in the ca-cert.pem file.

*certfiles*是该客户端信任的根证书集合。一般这都会是Fabric CA服务端根目录下的ca-cert.pem。

The client option is required only if mutual TLS is configured on the server.

只有在服务器配置了双向TLS的情况下，*client*选项才需要。

## 附录

### Postgres SSL 配置

#### 配置Postgre服务器的基本步骤：

1. In postgresql.conf, uncomment SSL and set to “on” (SSL=on)
2. Place certificate and key files in the Postgres data directory.

Instructions for generating self-signed certificates for: https://www.postgresql.org/docs/9.5/static/ssl-tcp.html

Note: Self-signed certificates are for testing purposes and should not be used in a production environment

1. 在postgresql.conf中打开SSL（SSL=on）
2. 把证书和密钥文件放在Postgres数据目录下。

如何生成自签名的证书：https://www.postgresql.org/docs/9.5/static/ssl-tcp.html

注意：自签名的证书用于测试目的，请勿用于生产环境。

#### Postgres 服务器 - 需要客户端证书

1. Place certificates of the certificate authorities (CAs) you trust in the file root.crt in the Postgres data directory
2. In postgresql.conf, set “ssl_ca_file” to point to the root cert of the client (CA cert)
3. Set the clientcert parameter to 1 on the appropriate hostssl line(s) in pg_hba.conf.

For more details on configuring SSL on the Postgres server, please refer to the following Postgres documentation: https://www.postgresql.org/docs/9.4/static/libpq-ssl.html

1. 把你信任的CA证书放在Postgres数据目录里的root.crt里
2. 在postgresql.conf里，设置“ssl_ca_file”指向客户端的根证书
3. 在pg_hba.conf里，在正确的hostssl行把clientcert参数设为1

更多信息：https://www.postgresql.org/docs/9.4/static/libpq-ssl.html

### MySQL SSL 配置

On MySQL 5.7.X, certain modes affect whether the server permits ‘0000-00-00’ as a valid date. It might be necessary to relax the modes that MySQL server uses. We want to allow the server to be able to accept zero date values.

