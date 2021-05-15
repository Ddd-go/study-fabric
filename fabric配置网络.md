# fabric配置网络

## 五个核心模块

1. `peer`：主要是维护链码

1. `oderer`：排序服务

1. `cryptogen`：给组织和节点生成证书（先编写.yaml配置文件，再用`cryptogen`初始化）

1. `configtxgen`：区块和交易生成（生成创世区块和通道）

1. `configtxlator`：两个用途：a. 将.block文件转换为json格式；b. 之后进行动态添加组织

   > 本次配置的需求是：三个组织（ordeeerOrg、GoOrg、CppOrg），ordeeerOrg只有一个节点，GoOrg有两个节点、三个用户，CppOrg有两个节点、三个用户

# 1. 生成证书 -- 用cryptogen

- 模板和配置文件注释

  用命令`cryptogen showtemplate > a.yaml`获取模板并重定向到a.yaml中，对a.yaml配置文件进行一些解读如下，模板如下（文件放在了`$GOPATH/src/模板`）：

```yaml

# ---------------------------------------------------------------------------
# "OrdererOrgs" - Definition of organizations managing orderer nodes
# ---------------------------------------------------------------------------
OrdererOrgs: #排序节点组织
  # ---------------------------------------------------------------------------
  # Orderer
  # ---------------------------------------------------------------------------
  - Name: Orderer # 排序节点的名字可改
    Domain: example.com # 根域名，排序节点组织的根域名
    EnableNodeOUs: false

    # ---------------------------------------------------------------------------
    # "Specs" - See PeerOrgs below for complete description
    # ---------------------------------------------------------------------------
    Specs: # 列举出全部的orderer节点
      - Hostname: orderer # 故该排序节点的地址为：orderer.example.com

# ---------------------------------------------------------------------------
# "PeerOrgs" - Definition of organizations managing peer nodes
# ---------------------------------------------------------------------------
PeerOrgs: 
  # ---------------------------------------------------------------------------
  # Org1
  # ---------------------------------------------------------------------------
  - Name: Org1 # 第一个组织的名字
    Domain: org1.example.com # 访问第一个组织的根域名
    EnableNodeOUs: false # 链码编写是否支持node.js

    # ---------------------------------------------------------------------------
    # "CA"
    # ---------------------------------------------------------------------------
    # Uncomment this section to enable the explicit definition of the CA for this
    # organization.  This entry is a Spec.  See "Specs" section below for details.
    # ---------------------------------------------------------------------------
    # CA:
    #    Hostname: ca # implicitly ca.org1.example.com
    #    Country: US
    #    Province: California
    #    Locality: San Francisco
    #    OrganizationalUnit: Hyperledger Fabric
    #    StreetAddress: address for org # default nil
    #    PostalCode: postalCode for org # default nil

    # ---------------------------------------------------------------------------
    # "Specs"
    # ---------------------------------------------------------------------------
    # Uncomment this section to enable the explicit definition of hosts in your
    # configuration.  Most users will want to use Template, below
    #
    # Specs is an array of Spec entries.  Each Spec entry consists of two fields:
    #   - Hostname:   (Required) The desired hostname, sans the domain.
    #   - CommonName: (Optional) Specifies the template or explicit override for
    #                 the CN.  By default, this is the template:
    #
    #                              "{{.Hostname}}.{{.Domain}}"
    #
    #                 which obtains its values from the Spec.Hostname and
    #                 Org.Domain, respectively.
    #   - SANS:       (Optional) Specifies one or more Subject Alternative Names
    #                 to be set in the resulting x509. Accepts template
    #                 variables {{.Hostname}}, {{.Domain}}, {{.CommonName}}. IP
    #                 addresses provided here will be properly recognized. Other
    #                 values will be taken as DNS names.
    #                 NOTE: Two implicit entries are created for you:
    #                     - {{ .CommonName }}
    #                     - {{ .Hostname }}
    # ---------------------------------------------------------------------------
    # Specs:
    #   - Hostname: foo # implicitly "foo.org1.example.com"
    #     CommonName: foo27.org5.example.com # overrides Hostname-based FQDN set above
    #     SANS:
    #       - "bar.{{.Domain}}"
    #       - "altfoo.{{.Domain}}"
    #       - "{{.Hostname}}.org6.net"
    #       - 172.16.10.31
    #   - Hostname: bar
    #   - Hostname: baz

    # ---------------------------------------------------------------------------
    # "Template"
    # ---------------------------------------------------------------------------
    # Allows for the definition of 1 or more hosts that are created sequentially
    # from a template. By default, this looks like "peer%d" from 0 to Count-1.
    # You may override the number of nodes (Count), the starting index (Start)
    # or the template used to construct the name (Hostname).
    #
    # Note: Template and Specs are not mutually exclusive.  You may define both
    # sections and the aggregate nodes will be created for you.  Take care with
    # name collisions
    # ---------------------------------------------------------------------------
    Template: # 模板，根据默认的规则生成Count个peer节点
      Count: 2 # 默认生成的peer节点的域名为peer0.org1.example.com和peer1.org1.example.com
      # Start: 5
      # Hostname: {{.Prefix}}{{.Index}} # default
      # SANS:
      #   - "{{.Hostname}}.alt.{{.Domain}}"

    # ---------------------------------------------------------------------------
    # "Users"
    # ---------------------------------------------------------------------------
    # Count: The number of user accounts _in addition_ to Admin
    # ---------------------------------------------------------------------------
    Users: # 创建普通用户的个数
      Count: 3

  # ---------------------------------------------------------------------------
  # Org2: See "Org1" for full specification
  # ---------------------------------------------------------------------------
  - Name: Org2 # 与组织1同理
    Domain: org2.example.com
    EnableNodeOUs: false
    Template:
      Count: 2 
    Users:
      Count: 3

```

- 按照需求进行修改配置文

我们将这个文件复制到自己想要的fabric网络文件夹中，我是放在`~/go/src/myfabric`，创建一个yaml文件，将a.yaml复制到其中，并进行修改，名字改为`crypto-config.yaml`（尽量用这个名字），对其域名和名字进行修改，`crypto-config.yaml`如下：

```yaml

# ---------------------------------------------------------------------------
# "OrdererOrgs" - Definition of organizations managing orderer nodes
# ---------------------------------------------------------------------------
OrdererOrgs: #排序节点组织
  # ---------------------------------------------------------------------------
  # Orderer
  # ---------------------------------------------------------------------------
  - Name: Orderer # 排序节点的名字可改
    Domain: itcast.com # 根域名，排序节点组织的根域名
    EnableNodeOUs: false

    # ---------------------------------------------------------------------------
    # "Specs" - See PeerOrgs below for complete description
    # ---------------------------------------------------------------------------
    Specs: # 列举出全部的orderer节点
      - Hostname: orderer # 故该排序节点的地址为：orderer.example.com

# ---------------------------------------------------------------------------
# "PeerOrgs" - Definition of organizations managing peer nodes
# ---------------------------------------------------------------------------
PeerOrgs: 
  # ---------------------------------------------------------------------------
  # Org1
  # ---------------------------------------------------------------------------
  - Name: OrgGo # 第一个组织的名字
    Domain: orgGo.itcast.com # 访问第一个组织的根域名
    EnableNodeOUs: false # 链码编写是否支持node.js

    # ---------------------------------------------------------------------------
    # "CA"
    # ---------------------------------------------------------------------------
    # Uncomment this section to enable the explicit definition of the CA for this
    # organization.  This entry is a Spec.  See "Specs" section below for details.
    # ---------------------------------------------------------------------------
    # CA:
    #    Hostname: ca # implicitly ca.org1.example.com
    #    Country: US
    #    Province: California
    #    Locality: San Francisco
    #    OrganizationalUnit: Hyperledger Fabric
    #    StreetAddress: address for org # default nil
    #    PostalCode: postalCode for org # default nil

    # ---------------------------------------------------------------------------
    # "Specs"
    # ---------------------------------------------------------------------------
    # Uncomment this section to enable the explicit definition of hosts in your
    # configuration.  Most users will want to use Template, below
    #
    # Specs is an array of Spec entries.  Each Spec entry consists of two fields:
    #   - Hostname:   (Required) The desired hostname, sans the domain.
    #   - CommonName: (Optional) Specifies the template or explicit override for
    #                 the CN.  By default, this is the template:
    #
    #                              "{{.Hostname}}.{{.Domain}}"
    #
    #                 which obtains its values from the Spec.Hostname and
    #                 Org.Domain, respectively.
    #   - SANS:       (Optional) Specifies one or more Subject Alternative Names
    #                 to be set in the resulting x509. Accepts template
    #                 variables {{.Hostname}}, {{.Domain}}, {{.CommonName}}. IP
    #                 addresses provided here will be properly recognized. Other
    #                 values will be taken as DNS names.
    #                 NOTE: Two implicit entries are created for you:
    #                     - {{ .CommonName }}
    #                     - {{ .Hostname }}
    # ---------------------------------------------------------------------------
    # Specs:
    #   - Hostname: foo # implicitly "foo.org1.example.com"
    #     CommonName: foo27.org5.example.com # overrides Hostname-based FQDN set above
    #     SANS:
    #       - "bar.{{.Domain}}"
    #       - "altfoo.{{.Domain}}"
    #       - "{{.Hostname}}.org6.net"
    #       - 172.16.10.31
    #   - Hostname: bar
    #   - Hostname: baz

    # ---------------------------------------------------------------------------
    # "Template"
    # ---------------------------------------------------------------------------
    # Allows for the definition of 1 or more hosts that are created sequentially
    # from a template. By default, this looks like "peer%d" from 0 to Count-1.
    # You may override the number of nodes (Count), the starting index (Start)
    # or the template used to construct the name (Hostname).
    #
    # Note: Template and Specs are not mutually exclusive.  You may define both
    # sections and the aggregate nodes will be created for you.  Take care with
    # name collisions
    # ---------------------------------------------------------------------------
    Template: # 模板，根据默认的规则生成Count个peer节点
      Count: 2 # 默认生成的peer节点的域名为peer0.org1.example.com和peer1.org1.example.com
      # Start: 5
      # Hostname: {{.Prefix}}{{.Index}} # default
      # SANS:
      #   - "{{.Hostname}}.alt.{{.Domain}}"

    # ---------------------------------------------------------------------------
    # "Users"
    # ---------------------------------------------------------------------------
    # Count: The number of user accounts _in addition_ to Admin
    # ---------------------------------------------------------------------------
    Users: # 创建普通用户的个数 他会默认添加一个管理员用户
      Count: 3

  # ---------------------------------------------------------------------------
  # Org2: See "Org1" for full specification
  # ---------------------------------------------------------------------------
  - Name: OrgCpp # 与组织1同理
    Domain: orgCpp.itcast.com
    EnableNodeOUs: false
    Template:
      Count: 2
    Users:
      Count: 3
```



再回到命令窗口在该目录下使用命令

```shell
aaa@aaa-G3-3579:~/go/src/myfabric$ cryptogen generate --config=crypto-config.yaml
orgGo.itcast.com
orgCpp.itcast.com
```

可以看到这两个组织已经生成。同时在该文件夹下会生成一个`crypto-config`子目录

> 注意：若 --config=crypto-config.yaml未给出这个参数，则会根据模板的yaml文件初始化

然后使用命令`tree crypto-config  `查看生成的子目录的结构，可以看到他会默认在每个组织生成一个管理员

```shell


aaa@aaa-G3-3579:~/go/src/myfabric$ tree crypto-config/
locales-launch: Data of zh_CN locale not found, generating, please wait...
crypto-config/
├── ordererOrganizations
│   └── itcast.com
│       ├── ca
│       │   ├── ca.itcast.com-cert.pem
│       │   └── priv_sk
│       ├── msp
│       │   ├── admincerts
│       │   │   └── Admin@itcast.com-cert.pem
│       │   ├── cacerts
│       │   │   └── ca.itcast.com-cert.pem
│       │   └── tlscacerts
│       │       └── tlsca.itcast.com-cert.pem
│       ├── orderers
│       │   └── orderer.itcast.com
│       │       ├── msp
│       │       │   ├── admincerts
│       │       │   │   └── Admin@itcast.com-cert.pem
│       │       │   ├── cacerts
│       │       │   │   └── ca.itcast.com-cert.pem
│       │       │   ├── keystore
│       │       │   │   └── priv_sk
│       │       │   ├── signcerts
│       │       │   │   └── orderer.itcast.com-cert.pem
│       │       │   └── tlscacerts
│       │       │       └── tlsca.itcast.com-cert.pem
│       │       └── tls
│       │           ├── ca.crt
│       │           ├── server.crt
│       │           └── server.key
│       ├── tlsca
│       │   ├── priv_sk
│       │   └── tlsca.itcast.com-cert.pem
│       └── users
│           └── Admin@itcast.com
│               ├── msp
│               │   ├── admincerts
│               │   │   └── Admin@itcast.com-cert.pem
│               │   ├── cacerts
│               │   │   └── ca.itcast.com-cert.pem
│               │   ├── keystore
│               │   │   └── priv_sk
│               │   ├── signcerts
│               │   │   └── Admin@itcast.com-cert.pem
│               │   └── tlscacerts
│               │       └── tlsca.itcast.com-cert.pem
│               └── tls
│                   ├── ca.crt
│                   ├── client.crt
│                   └── client.key
└── peerOrganizations
    ├── orgCpp.itcast.com
    │   ├── ca
    │   │   ├── ca.orgCpp.itcast.com-cert.pem
    │   │   └── priv_sk
    │   ├── msp
    │   │   ├── admincerts
    │   │   │   └── Admin@orgCpp.itcast.com-cert.pem
    │   │   ├── cacerts
    │   │   │   └── ca.orgCpp.itcast.com-cert.pem
    │   │   └── tlscacerts
    │   │       └── tlsca.orgCpp.itcast.com-cert.pem
    │   ├── peers
    │   │   ├── peer0.orgCpp.itcast.com
    │   │   │   ├── msp
    │   │   │   │   ├── admincerts
    │   │   │   │   │   └── Admin@orgCpp.itcast.com-cert.pem
    │   │   │   │   ├── cacerts
    │   │   │   │   │   └── ca.orgCpp.itcast.com-cert.pem
    │   │   │   │   ├── keystore
    │   │   │   │   │   └── priv_sk
    │   │   │   │   ├── signcerts
    │   │   │   │   │   └── peer0.orgCpp.itcast.com-cert.pem
    │   │   │   │   └── tlscacerts
    │   │   │   │       └── tlsca.orgCpp.itcast.com-cert.pem
    │   │   │   └── tls
    │   │   │       ├── ca.crt
    │   │   │       ├── server.crt
    │   │   │       └── server.key
    │   │   └── peer1.orgCpp.itcast.com
    │   │       ├── msp
    │   │       │   ├── admincerts
    │   │       │   │   └── Admin@orgCpp.itcast.com-cert.pem
    │   │       │   ├── cacerts
    │   │       │   │   └── ca.orgCpp.itcast.com-cert.pem
    │   │       │   ├── keystore
    │   │       │   │   └── priv_sk
    │   │       │   ├── signcerts
    │   │       │   │   └── peer1.orgCpp.itcast.com-cert.pem
    │   │       │   └── tlscacerts
    │   │       │       └── tlsca.orgCpp.itcast.com-cert.pem
    │   │       └── tls
    │   │           ├── ca.crt
    │   │           ├── server.crt
    │   │           └── server.key
    │   ├── tlsca
    │   │   ├── priv_sk
    │   │   └── tlsca.orgCpp.itcast.com-cert.pem
    │   └── users
    │       ├── Admin@orgCpp.itcast.com
    │       │   ├── msp
    │       │   │   ├── admincerts
    │       │   │   │   └── Admin@orgCpp.itcast.com-cert.pem
    │       │   │   ├── cacerts
    │       │   │   │   └── ca.orgCpp.itcast.com-cert.pem
    │       │   │   ├── keystore
    │       │   │   │   └── priv_sk
    │       │   │   ├── signcerts
    │       │   │   │   └── Admin@orgCpp.itcast.com-cert.pem
    │       │   │   └── tlscacerts
    │       │   │       └── tlsca.orgCpp.itcast.com-cert.pem
    │       │   └── tls
    │       │       ├── ca.crt
    │       │       ├── client.crt
    │       │       └── client.key
    │       ├── User1@orgCpp.itcast.com
    │       │   ├── msp
    │       │   │   ├── admincerts
    │       │   │   │   └── User1@orgCpp.itcast.com-cert.pem
    │       │   │   ├── cacerts
    │       │   │   │   └── ca.orgCpp.itcast.com-cert.pem
    │       │   │   ├── keystore
    │       │   │   │   └── priv_sk
    │       │   │   ├── signcerts
    │       │   │   │   └── User1@orgCpp.itcast.com-cert.pem
    │       │   │   └── tlscacerts
    │       │   │       └── tlsca.orgCpp.itcast.com-cert.pem
    │       │   └── tls
    │       │       ├── ca.crt
    │       │       ├── client.crt
    │       │       └── client.key
    │       ├── User2@orgCpp.itcast.com
    │       │   ├── msp
    │       │   │   ├── admincerts
    │       │   │   │   └── User2@orgCpp.itcast.com-cert.pem
    │       │   │   ├── cacerts
    │       │   │   │   └── ca.orgCpp.itcast.com-cert.pem
    │       │   │   ├── keystore
    │       │   │   │   └── priv_sk
    │       │   │   ├── signcerts
    │       │   │   │   └── User2@orgCpp.itcast.com-cert.pem
    │       │   │   └── tlscacerts
    │       │   │       └── tlsca.orgCpp.itcast.com-cert.pem
    │       │   └── tls
    │       │       ├── ca.crt
    │       │       ├── client.crt
    │       │       └── client.key
    │       └── User3@orgCpp.itcast.com
    │           ├── msp
    │           │   ├── admincerts
    │           │   │   └── User3@orgCpp.itcast.com-cert.pem
    │           │   ├── cacerts
    │           │   │   └── ca.orgCpp.itcast.com-cert.pem
    │           │   ├── keystore
    │           │   │   └── priv_sk
    │           │   ├── signcerts
    │           │   │   └── User3@orgCpp.itcast.com-cert.pem
    │           │   └── tlscacerts
    │           │       └── tlsca.orgCpp.itcast.com-cert.pem
    │           └── tls
    │               ├── ca.crt
    │               ├── client.crt
    │               └── client.key
    └── orgGo.itcast.com
        ├── ca
        │   ├── ca.orgGo.itcast.com-cert.pem
        │   └── priv_sk
        ├── msp
        │   ├── admincerts
        │   │   └── Admin@orgGo.itcast.com-cert.pem
        │   ├── cacerts
        │   │   └── ca.orgGo.itcast.com-cert.pem
        │   └── tlscacerts
        │       └── tlsca.orgGo.itcast.com-cert.pem
        ├── peers
        │   ├── peer0.orgGo.itcast.com
        │   │   ├── msp
        │   │   │   ├── admincerts
        │   │   │   │   └── Admin@orgGo.itcast.com-cert.pem
        │   │   │   ├── cacerts
        │   │   │   │   └── ca.orgGo.itcast.com-cert.pem
        │   │   │   ├── keystore
        │   │   │   │   └── priv_sk
        │   │   │   ├── signcerts
        │   │   │   │   └── peer0.orgGo.itcast.com-cert.pem
        │   │   │   └── tlscacerts
        │   │   │       └── tlsca.orgGo.itcast.com-cert.pem
        │   │   └── tls
        │   │       ├── ca.crt
        │   │       ├── server.crt
        │   │       └── server.key
        │   └── peer1.orgGo.itcast.com
        │       ├── msp
        │       │   ├── admincerts
        │       │   │   └── Admin@orgGo.itcast.com-cert.pem
        │       │   ├── cacerts
        │       │   │   └── ca.orgGo.itcast.com-cert.pem
        │       │   ├── keystore
        │       │   │   └── priv_sk
        │       │   ├── signcerts
        │       │   │   └── peer1.orgGo.itcast.com-cert.pem
        │       │   └── tlscacerts
        │       │       └── tlsca.orgGo.itcast.com-cert.pem
        │       └── tls
        │           ├── ca.crt
        │           ├── server.crt
        │           └── server.key
        ├── tlsca
        │   ├── priv_sk
        │   └── tlsca.orgGo.itcast.com-cert.pem
        └── users
            ├── Admin@orgGo.itcast.com
            │   ├── msp
            │   │   ├── admincerts
            │   │   │   └── Admin@orgGo.itcast.com-cert.pem
            │   │   ├── cacerts
            │   │   │   └── ca.orgGo.itcast.com-cert.pem
            │   │   ├── keystore
            │   │   │   └── priv_sk
            │   │   ├── signcerts
            │   │   │   └── Admin@orgGo.itcast.com-cert.pem
            │   │   └── tlscacerts
            │   │       └── tlsca.orgGo.itcast.com-cert.pem
            │   └── tls
            │       ├── ca.crt
            │       ├── client.crt
            │       └── client.key
            ├── User1@orgGo.itcast.com
            │   ├── msp
            │   │   ├── admincerts
            │   │   │   └── User1@orgGo.itcast.com-cert.pem
            │   │   ├── cacerts
            │   │   │   └── ca.orgGo.itcast.com-cert.pem
            │   │   ├── keystore
            │   │   │   └── priv_sk
            │   │   ├── signcerts
            │   │   │   └── User1@orgGo.itcast.com-cert.pem
            │   │   └── tlscacerts
            │   │       └── tlsca.orgGo.itcast.com-cert.pem
            │   └── tls
            │       ├── ca.crt
            │       ├── client.crt
            │       └── client.key
            ├── User2@orgGo.itcast.com
            │   ├── msp
            │   │   ├── admincerts
            │   │   │   └── User2@orgGo.itcast.com-cert.pem
            │   │   ├── cacerts
            │   │   │   └── ca.orgGo.itcast.com-cert.pem
            │   │   ├── keystore
            │   │   │   └── priv_sk
            │   │   ├── signcerts
            │   │   │   └── User2@orgGo.itcast.com-cert.pem
            │   │   └── tlscacerts
            │   │       └── tlsca.orgGo.itcast.com-cert.pem
            │   └── tls
            │       ├── ca.crt
            │       ├── client.crt
            │       └── client.key
            └── User3@orgGo.itcast.com
                ├── msp
                │   ├── admincerts
                │   │   └── User3@orgGo.itcast.com-cert.pem
                │   ├── cacerts
                │   │   └── ca.orgGo.itcast.com-cert.pem
                │   ├── keystore
                │   │   └── priv_sk
                │   ├── signcerts
                │   │   └── User3@orgGo.itcast.com-cert.pem
                │   └── tlscacerts
                │       └── tlsca.orgGo.itcast.com-cert.pem
                └── tls
                    ├── ca.crt
                    ├── client.crt
                    └── client.key
```





第一步完事儿！

# 2. 创始区块和通道文件的生成--configtxgen模块

同样是要先编写一个configtx.yaml配置文件

> 注意：那个文件名字要固定，不能修改！！！

- 模板和配置文件注解

  且此时的配置文件不能像前面的一样通过子命令获取模板，但是我们可以从官方的例子中获取找到这个文件，然后复制，模板如下（）：

`````yaml
# Copyright IBM Corp. All Rights Reserved.
#
# SPDX-License-Identifier: Apache-2.0
#

---
################################################################################
#
#   Section: Organizations
#
#   - This section defines the different organizational identities which will
#   be referenced later in the configuration.
#
################################################################################
Organizations: # 组织，这里面的name不用和1中的名字进行对应

    
    - &OrdererOrg # &OrdererOrg的作用是，相当于把后面的Name、ID、MSPDir、Policies封装成一个结构体，后面要用时直接*OrdererOrg即可
        # DefaultOrg defines the organization which is used in the sampleconfig
        # of the fabric.git development environment
        Name: OrdererOrg # 组织名

        # ID to load the MSP definition as
        ID: OrdererMSP # 

        # MSPDir is the filesystem path which contains the MSP configuration
        # 找到orderer节点的msp文件路径复制过来，注意是相对路径
        MSPDir: crypto-config/ordererOrganizations/example.com/msp

        # Policies defines the set of policies at this level of the config tree
        # For organization policies, their canonical path is usually
        #   /Channel/<Application|Orderer>/<OrgName>/<PolicyName>
        Policies:
            Readers:
                Type: Signature
                Rule: "OR('OrdererMSP.member')"
            Writers:
                Type: Signature
                Rule: "OR('OrdererMSP.member')"
            Admins:
                Type: Signature
                Rule: "OR('OrdererMSP.admin')"

    - &Org1 # 与orderer同理
        # DefaultOrg defines the organization which is used in the sampleconfig
        # of the fabric.git development environment
        Name: Org1MSP

        # ID to load the MSP definition as
        ID: Org1MSP

        MSPDir: crypto-config/peerOrganizations/org1.example.com/msp

        # Policies defines the set of policies at this level of the config tree
        # For organization policies, their canonical path is usually
        #   /Channel/<Application|Orderer>/<OrgName>/<PolicyName>
        Policies:
            Readers:
                Type: Signature
                Rule: "OR('Org1MSP.admin', 'Org1MSP.peer', 'Org1MSP.client')"
            Writers:
                Type: Signature
                Rule: "OR('Org1MSP.admin', 'Org1MSP.client')"
            Admins:
                Type: Signature
                Rule: "OR('Org1MSP.admin')"

        # leave this flag set to true.
        AnchorPeers: # 锚节点，随便指定一个即可
            # AnchorPeers defines the location of peers which can be used
            # for cross org gossip communication.  Note, this value is only
            # encoded in the genesis block in the Application section context
            - Host: peer0.org1.example.com
              Port: 7051 # 端口号不用改

    - &Org2
        # DefaultOrg defines the organization which is used in the sampleconfig
        # of the fabric.git development environment
        Name: Org2MSP

        # ID to load the MSP definition as
        ID: Org2MSP

        MSPDir: crypto-config/peerOrganizations/org2.example.com/msp

        # Policies defines the set of policies at this level of the config tree
        # For organization policies, their canonical path is usually
        #   /Channel/<Application|Orderer>/<OrgName>/<PolicyName>
        Policies:
            Readers:
                Type: Signature
                Rule: "OR('Org2MSP.admin', 'Org2MSP.peer', 'Org2MSP.client')"
            Writers:
                Type: Signature
                Rule: "OR('Org2MSP.admin', 'Org2MSP.client')"
            Admins:
                Type: Signature
                Rule: "OR('Org2MSP.admin')"

        AnchorPeers:
            # AnchorPeers defines the location of peers which can be used
            # for cross org gossip communication.  Note, this value is only
            # encoded in the genesis block in the Application section context
            - Host: peer0.org2.example.com
              Port: 9051

################################################################################
#
#   SECTION: Capabilities
#
#   - This section defines the capabilities of fabric network. This is a new
#   concept as of v1.1.0 and should not be utilized in mixed networks with
#   v1.0.x peers and orderers.  Capabilities define features which must be
#   present in a fabric binary for that binary to safely participate in the
#   fabric network.  For instance, if a new MSP type is added, newer binaries
#   might recognize and validate the signatures from this type, while older
#   binaries without this support would be unable to validate those
#   transactions.  This could lead to different versions of the fabric binaries
#   having different world states.  Instead, defining a capability for a channel
#   informs those binaries without this capability that they must cease
#   processing transactions until they have been upgraded.  For v1.0.x if any
#   capabilities are defined (including a map with all capabilities turned off)
#   then the v1.0.x peer will deliberately crash.
#
################################################################################
Capabilities: # 这个部分是对就版本的兼容性考虑，不用过多修改和考虑
    # Channel capabilities apply to both the orderers and the peers and must be
    # supported by both.
    # Set the value of the capability to true to require it.
    Channel: &ChannelCapabilities
        # V1.4.2 for Channel is a catchall flag for behavior which has been
        # determined to be desired for all orderers and peers running at the v1.4.2
        # level, but which would be incompatible with orderers and peers from
        # prior releases.
        # Prior to enabling V1.4.2 channel capabilities, ensure that all
        # orderers and peers on a channel are at v1.4.2 or later.
        V1_4_2: true

    # Orderer capabilities apply only to the orderers, and may be safely
    # used with prior release peers.
    # Set the value of the capability to true to require it.
    Orderer: &OrdererCapabilities
        # V1.4.2 for Orderer is a catchall flag for behavior which has been
        # determined to be desired for all orderers running at the v1.4.2
        # level, but which would be incompatible with orderers from prior releases.
        # Prior to enabling V1.4.2 orderer capabilities, ensure that all
        # orderers on a channel are at v1.4.2 or later.
        V1_4_2: true

    # Application capabilities apply only to the peer network, and may be safely
    # used with prior release orderers.
    # Set the value of the capability to true to require it.
    Application: &ApplicationCapabilities
        # V1.4.2 for Application enables the new non-backwards compatible
        # features and fixes of fabric v1.4.2.
        V1_4_2: true
        # V1.3 for Application enables the new non-backwards compatible
        # features and fixes of fabric v1.3.
        V1_3: false
        # V1.2 for Application enables the new non-backwards compatible
        # features and fixes of fabric v1.2 (note, this need not be set if
        # later version capabilities are set)
        V1_2: false
        # V1.1 for Application enables the new non-backwards compatible
        # features and fixes of fabric v1.1 (note, this need not be set if
        # later version capabilities are set).
        V1_1: false

################################################################################
#
#   SECTION: Application
#
#   - This section defines the values to encode into a config transaction or
#   genesis block for application related parameters
#
################################################################################
Application: &ApplicationDefaults # 这一部分也是直接使用默认配置即可

    # Organizations is the list of orgs which are defined as participants on
    # the application side of the network
    Organizations:

    # Policies defines the set of policies at this level of the config tree
    # For Application policies, their canonical path is
    #   /Channel/Application/<PolicyName>
    Policies:
        Readers:
            Type: ImplicitMeta
            Rule: "ANY Readers"
        Writers:
            Type: ImplicitMeta
            Rule: "ANY Writers"
        Admins:
            Type: ImplicitMeta
            Rule: "MAJORITY Admins"

    Capabilities:
        <<: *ApplicationCapabilities
################################################################################
#
#   SECTION: Orderer
#
#   - This section defines the values to encode into a config transaction or
#   genesis block for orderer related parameters
#
################################################################################
Orderer: &OrdererDefaults

    # Orderer Type: The orderer implementation to start
    # Available types are "solo" and "kafka"
    OrdererType: solo # 排序方式

    Addresses: # 地址，我们要改为在前一个配置文件中的orderer节点的域名，后面的7050是端口号，不用进行修改
        - orderer.example.com:7050

    # Batch Timeout: The amount of time to wait before creating a batch
    # 批处理超时：创建批处理之前等待的时间量，即当时间超过BatchTimeout时就会创建一个区块
    BatchTimeout: 2s

    # Batch Size: Controls the number of messages batched into a block
    BatchSize:

        # Max Message Count: The maximum number of messages to permit in a batch
        # 最大消息计数：批处理中允许的最大消息数，即当收到的交易数量大于MaxMessageCount时就会传家一个区块
        # 一般设置为100
        MaxMessageCount: 10

        # Absolute Max Bytes: The absolute maximum number of bytes allowed for
        # the serialized messages in a batch.
        # 同理，但
        AbsoluteMaxBytes: 99 MB

        # Preferred Max Bytes: The preferred maximum number of byt资料存放地址：es allowed for
        # the serialized messages in a batch. A message larger than the preferred
        # max bytes will result in a batch larger than preferred max bytes.
        # 这个一般不用去设置它
        PreferredMaxBytes: 512 KB

    Kafka: # 我们设置的时solo所以这个就也不用设置
        # Brokers: A list of Kafka brokers to which the orderer connects
        # NOTE: Use IP:port notation
        Brokers:
            - 127.0.0.1:9092

    # Organizations is the list of orgs which are defined as participants on
    # the orderer side of the network
    Organizations: # 这个空着就好，不用去修改

    # Policies defines the set of policies at this level of the config tree
    # For Orderer policies, their canonical path is
    #   /Channel/Orderer/<PolicyName>
    Policies:
        Readers:
            Type: ImplicitMeta
            Rule: "ANY Readers"
        Writers:
            Type: ImplicitMeta
            Rule: "ANY Writers"
        Admins:
            Type: ImplicitMeta
            Rule: "MAJORITY Admins"
        # BlockValidation specifies what signatures must be included in the block
        # from the orderer for the peer to validate it.
        BlockValidation:
            Type: ImplicitMeta
            Rule: "ANY Writers"

################################################################################
#
#   CHANNEL
#
#   This section defines the values to encode into a config transaction or
#   genesis block for channel related parameters.
#
################################################################################
Channel: &ChannelDefaults # 这个视频中没有提到，所以战士不用修改设置这一项
    # Policies defines the set of policies at this level of the config tree
    # For Channel policies, their canonical path is
    #   /Channel/<PolicyName>
    Policies:
        # Who may invoke the 'Deliver' API
        Readers:
            Type: ImplicitMeta
            Rule: "ANY Readers"
        # Who may invoke the 'Broadcast' API
        Writers:
            Type: ImplicitMeta
            Rule: "ANY Writers"
        # By default, who may modify elements at this config level
        Admins:
            Type: ImplicitMeta
            Rule: "MAJORITY Admins"

    # Capabilities describes the channel level capabilities, see the
    # dedicated Capabilities section elsewhere in this file for a full
    # description
    Capabilities:
        <<: *ChannelCapabilities

################################################################################
#
#   Profile 
#
#   - Different configuration profiles may be encoded here to be specified
#   as parameters to the configtxgen tool
#
################################################################################
Profiles: # 相当于对前面的一个总结
    # 创世区块
    TwoOrgsOrdererGenesis: # 这个创世区块的名字可以进行修改
        <<: *ChannelDefaults
        Orderer:
            <<: *OrdererDefaults
            Organizations:
                - *OrdererOrg
            Capabilities:
                <<: *OrdererCapabilities
        Consortiums: # 联盟
            SampleConsortium: # 联盟的名字，可以进行修改，一般没改，一旦修改了，下面的325行也要进行修改
                Organizations:
                    - *Org1
                    - *Org2
    # 通道
    TwoOrgsChannel: # 这个通道的名字可以进行修改
        Consortium: SampleConsortium
        <<: *ChannelDefaults
        Application:
            <<: *ApplicationDefaults
            Organizations:
                - *Org1
                - *Org2
            Capabilities:
                <<: *ApplicationCapabilities

    SampleDevModeKafka:
        <<: *ChannelDefaults
        Capabilities:
            <<: *ChannelCapabilities
        Orderer:
            <<: *OrdererDefaults
            OrdererType: kafka
            Kafka:
                Brokers:
                - kafka.example.com:9092

            Organizations:
            - *OrdererOrg
            Capabilities:
                <<: *OrdererCapabilities
        Application:
            <<: *ApplicationDefaults
            Organizations:
            - <<: *OrdererOrg
        Consortiums:
            SampleConsortium:
                Organizations:
                - *Org1
                - *Org2
    # 视频中没有，自己读一下，（这个好像是关于集群的）
    SampleMultiNodeEtcdRaft:
        <<: *ChannelDefaults
        Capabilities:
            <<: *ChannelCapabilities
        Orderer:
            <<: *OrdererDefaults
            OrdererType: etcdraft
            EtcdRaft:
                Consenters:
                - Host: orderer.example.com
                  Port: 7050
                  ClientTLSCert: crypto-config/ordererOrganizations/example.com/orderers/orderer.example.com/tls/server.crt
                  ServerTLSCert: crypto-config/ordererOrganizations/example.com/orderers/orderer.example.com/tls/server.crt
                - Host: orderer2.example.com
                  Port: 7050
                  ClientTLSCert: crypto-config/ordererOrganizations/example.com/orderers/orderer2.example.com/tls/server.crt
                  ServerTLSCert: crypto-config/ordererOrganizations/example.com/orderers/orderer2.example.com/tls/server.crt
                - Host: orderer3.example.com
                  Port: 7050
                  ClientTLSCert: crypto-config/ordererOrganizations/example.com/orderers/orderer3.example.com/tls/server.crt
                  ServerTLSCert: crypto-config/ordererOrganizations/example.com/orderers/orderer3.example.com/tls/server.crt
                - Host: orderer4.example.com
                  Port: 7050
                  ClientTLSCert: crypto-config/ordererOrganizations/example.com/orderers/orderer4.example.com/tls/server.crt
                  ServerTLSCert: crypto-config/ordererOrganizations/example.com/orderers/orderer4.example.com/tls/server.crt
                - Host: orderer5.example.com
                  Port: 7050
                  ClientTLSCert: crypto-config/ordererOrganizations/example.com/orderers/orderer5.example.com/tls/server.crt
                  ServerTLSCert: crypto-config/ordererOrganizations/example.com/orderers/orderer5.example.com/tls/server.crt
            Addresses:
                - orderer.example.com:7050
                - orderer2.example.com:7050
                - orderer3.example.com:7050
                - orderer4.example.com:7050
                - orderer5.example.com:7050

            Organizations:
            - *OrdererOrg
            Capabilities:
                <<: *OrdererCapabilities
        Application:
            <<: *ApplicationDefaults
            Organizations:
            - <<: *OrdererOrg
        Consortiums:
            SampleConsortium:
                Organizations:
                - *Org1
                - *Org2
`````

- 现在正式按照我们的需求进行配置

  > 要求在本文开始的地方

  ```yaml
  # Copyright IBM Corp. All Rights Reserved.
  #
  # SPDX-License-Identifier: Apache-2.0
  #
  
  ---
  ################################################################################
  #
  #   Section: Organizations
  #
  #   - This section defines the different organizational identities which will
  #   be referenced later in the configuration.
  #
  ################################################################################
  Organizations: # 组织，这里面的name不用和1中的名字进行对应
  
      
      - &OrdererOrg # &OrdererOrg的作用是，相当于把后面的Name、ID、MSPDir、Policies封装成一个结构体，后面要用时直接*OrdererOrg即可
          # DefaultOrg defines the organization which is used in the sampleconfig
          # of the fabric.git development environment
          Name: OrdererOrg # 组织名
  
          # ID to load the MSP definition as
          ID: OrdererMSP # 
  
          # MSPDir is the filesystem path which contains the MSP configuration
          # 找到orderer节点的msp文件路径复制过来，注意是相对路径
          MSPDir: crypto-config/ordererOrganizations/itcast.com/msp
  
          # Policies defines the set of policies at this level of the config tree
          # For organization policies, their canonical path is usually
          #   /Channel/<Application|Orderer>/<OrgName>/<PolicyName>
          Policies:
              Readers:
                  Type: Signature
                  Rule: "OR('OrdererMSP.member')"
              Writers:
                  Type: Signature
                  Rule: "OR('OrdererMSP.member')"
              Admins:
                  Type: Signature
                  Rule: "OR('OrdererMSP.admin')"
  
      - &Org1 # 与orderer同理
          # DefaultOrg defines the organization which is used in the sampleconfig
          # of the fabric.git development environment
          Name: OrgGoMSP
  
          # ID to load the MSP definition as
          ID: OrgGoMSP
  
          MSPDir: crypto-config/peerOrganizations/orgGo.itcast.com/msp
  
          # Policies defines the set of policies at this level of the config tree
          # For organization policies, their canonical path is usually
          #   /Channel/<Application|Orderer>/<OrgName>/<PolicyName>
          Policies:
              Readers:
                  Type: Signature
                  Rule: "OR('Org1MSP.admin', 'Org1MSP.peer', 'Org1MSP.client')"
              Writers:
                  Type: Signature
                  Rule: "OR('Org1MSP.admin', 'Org1MSP.client')"
              Admins:
                  Type: Signature
                  Rule: "OR('Org1MSP.admin')"
  
          # leave this flag set to true.
          AnchorPeers: # 锚节点，随便指定一个即可
              # AnchorPeers defines the location of peers which can be used
              # for cross org gossip communication.  Note, this value is only
              # encoded in the genesis block in the Application section context
              - Host: peer0.orgGo.itcast.com
                Port: 7051 # 端口号不用改
  
      - &Org2
          # DefaultOrg defines the organization which is used in the sampleconfig
          # of the fabric.git development environment
          Name: OrgCppMSP
  
          # ID to load the MSP definition as
          ID: OrgCppMSP
  
          MSPDir: crypto-config/peerOrganizations/orgCpp.itcast.com/msp
  
          # Policies defines the set of policies at this level of the config tree
          # For organization policies, their canonical path is usually
          #   /Channel/<Application|Orderer>/<OrgName>/<PolicyName>
          Policies:
              Readers:
                  Type: Signature
                  Rule: "OR('Org2MSP.admin', 'Org2MSP.peer', 'Org2MSP.client')"
              Writers:
                  Type: Signature
                  Rule: "OR('Org2MSP.admin', 'Org2MSP.client')"
              Admins:
                  Type: Signature
                  Rule: "OR('Org2MSP.admin')"
  
          AnchorPeers:
              # AnchorPeers defines the location of peers which can be used
              # for cross org gossip communication.  Note, this value is only
              # encoded in the genesis block in the Application section context
              - Host: peer0.orgCpp.itcast.com
                Port: 9051
  
  ################################################################################
  #
  #   SECTION: Capabilities
  #
  #   - This section defines the capabilities of fabric network. This is a new
  #   concept as of v1.1.0 and should not be utilized in mixed networks with
  #   v1.0.x peers and orderers.  Capabilities define features which must be
  #   present in a fabric binary for that binary to safely participate in the
  #   fabric network.  For instance, if a new MSP type is added, newer binaries
  #   might recognize and validate the signatures from this type, while older
  #   binaries without this support would be unable to validate those
  #   transactions.  This could lead to different versions of the fabric binaries
  #   having different world states.  Instead, defining a capability for a channel
  #   informs those binaries without this capability that they must cease
  #   processing transactions until they have been upgraded.  For v1.0.x if any
  #   capabilities are defined (including a map with all capabilities turned off)
  #   then the v1.0.x peer will deliberately crash.
  #
  ################################################################################
  Capabilities: # 这个部分是对就版本的兼容性考虑，不用过多修改和考虑
      # Channel capabilities apply to both the orderers and the peers and must be
      # supported by both.
      # Set the value of the capability to true to require it.
      Channel: &ChannelCapabilities
          # V1.4.2 for Channel is a catchall flag for behavior which has been
          # determined to be desired for all orderers and peers running at the v1.4.2
          # level, but which would be incompatible with orderers and peers from
          # prior releases.
          # Prior to enabling V1.4.2 channel capabilities, ensure that all
          # orderers and peers on a channel are at v1.4.2 or later.
          V1_4_2: true
  
      # Orderer capabilities apply only to the orderers, and may be safely
      # used with prior release peers.
      # Set the value of the capability to true to require it.
      Orderer: &OrdererCapabilities
          # V1.4.2 for Orderer is a catchall flag for behavior which has been
          # determined to be desired for all orderers running at the v1.4.2
          # level, but which would be incompatible with orderers from prior releases.
          # Prior to enabling V1.4.2 orderer capabilities, ensure that all
          # orderers on a channel are at v1.4.2 or later.
          V1_4_2: true
  
      # Application capabilities apply only to the peer network, and may be safely
      # used with prior release orderers.
      # Set the value of the capability to true to require it.
      Application: &ApplicationCapabilities
          # V1.4.2 for Application enables the new non-backwards compatible
          # features and fixes of fabric v1.4.2.
          V1_4_2: true
          # V1.3 for Application enables the new non-backwards compatible
          # features and fixes of fabric v1.3.
          V1_3: false
          # V1.2 for Application enables the new non-backwards compatible
          # features and fixes of fabric v1.2 (note, this need not be set if
          # later version capabilities are set)
          V1_2: false
          # V1.1 for Application enables the new non-backwards compatible
          # features and fixes of fabric v1.1 (note, this need not be set if
          # later version capabilities are set).
          V1_1: false
  
  ################################################################################
  #
  #   SECTION: Application
  #
  #   - This section defines the values to encode into a config transaction or
  #   genesis block for application related parameters
  #
  ################################################################################
  Application: &ApplicationDefaults # 这一部分也是直接使用默认配置即可
  
      # Organizations is the list of orgs which are defined as participants on
      # the application side of the network
      Organizations:
  
      # Policies defines the set of policies at this level of the config tree
      # For Application policies, their canonical path is
      #   /Channel/Application/<PolicyName>
      Policies:
          Readers:
              Type: ImplicitMeta
              Rule: "ANY Readers"
          Writers:
              Type: ImplicitMeta
              Rule: "ANY Writers"
          Admins:
              Type: ImplicitMeta
              Rule: "MAJORITY Admins"
  
      Capabilities:
          <<: *ApplicationCapabilities
  ################################################################################
  #
  #   SECTION: Orderer
  #
  #   - This section defines the values to encode into a config transaction or
  #   genesis block for orderer related parameters
  #
  ################################################################################
  Orderer: &OrdererDefaults
  
      # Orderer Type: The orderer implementation to start
      # Available types are "solo" and "kafka"
      OrdererType: solo # 排序方式
  
      Addresses: # 地址，我们要改为在前一个配置文件中的orderer节点的域名，后面的7050是端口号，不用进行修改
          - orderer.itcast.com:7050
  
      # Batch Timeout: The amount of time to wait before creating a batch
      # 批处理超时：创建批处理之前等待的时间量，即当时间超过BatchTimeout时就会创建一个区块
      BatchTimeout: 2s
  
      # Batch Size: Controls the number of messages batched into a block
      BatchSize:
  
          # Max Message Count: The maximum number of messages to permit in a batch
          # 最大消息计数：批处理中允许的最大消息数，即当收到的交易数量大于MaxMessageCount时就会传家一个区块
          # 一般设置为100
          MaxMessageCount: 10
  
          # Absolute Max Bytes: The absolute maximum number of bytes allowed for
          # the serialized messages in a batch.
          # 同理，但
          AbsoluteMaxBytes: 32 MB
  
          # Preferred Max Bytes: The preferred maximum number of bytes allowed for
          # the serialized messages in a batch. A message larger than the preferred
          # max bytes will result in a batch larger than preferred max bytes.
          # 这个一般不用去设置它
          PreferredMaxBytes: 512 KB
  
      Kafka: # 我们设置的时solo所以这个就也不用设置
          # Brokers: A list of Kafka brokers to which the orderer connects
          # NOTE: Use IP:port notation
          Brokers:
              - 127.0.0.1:9092
  
      # Organizations is the list of orgs which are defined as participants on
      # the orderer side of the network
      Organizations: # 这个空着就好，不用去修改
  
      # Policies defines the set of policies at this level of the config tree
      # For Orderer policies, their canonical path is
      #   /Channel/Orderer/<PolicyName>
      Policies:
          Readers:
              Type: ImplicitMeta
              Rule: "ANY Readers"
          Writers:
              Type: ImplicitMeta
              Rule: "ANY Writers"
          Admins:
              Type: ImplicitMeta
              Rule: "MAJORITY Admins"
          # BlockValidation specifies what signatures must be included in the block
          # from the orderer for the peer to validate it.
          BlockValidation:
              Type: ImplicitMeta
              Rule: "ANY Writers"
  
  ################################################################################
  #
  #   CHANNEL
  #
  #   This section defines the values to encode into a config transaction or
  #   genesis block for channel related parameters.
  #
  ################################################################################
  Channel: &ChannelDefaults # 这个视频中没有提到，所以暂时不用修改设置这一项
      # Policies defines the set of policies at this level of the config tree
      # For Channel policies, their canonical path is
      #   /Channel/<PolicyName>
      Policies:
          # Who may invoke the 'Deliver' API
          Readers:
              Type: ImplicitMeta
              Rule: "ANY Readers"
          # Who may invoke the 'Broadcast' API
          Writers:
              Type: ImplicitMeta
              Rule: "ANY Writers"
          # By default, who may modify elements at this config level
          Admins:
              Type: ImplicitMeta
              Rule: "MAJORITY Admins"
  
      # Capabilities describes the channel level capabilities, see the
      # dedicated Capabilities section elsewhere in this file for a full
      # description
      Capabilities:
          <<: *ChannelCapabilities
  
  ################################################################################
  #
  #   Profile 
  #
  #   - Different configuration profiles may be encoded here to be specified
  #   as parameters to the configtxgen tool
  #
  ################################################################################
  Profiles: # 相当于对前面的一个总结
      # 创世区块
      ItcastOrgsOrdererGenesis: # 这个创世区块的名字可以进行修改
          <<: *ChannelDefaults
          Orderer:
              <<: *OrdererDefaults
              Organizations:
                  - *OrdererOrg
              Capabilities:
                  <<: *OrdererCapabilities
          Consortiums: # 联盟
              SampleConsortium: # 联盟的名字，可以进行修改，一般没改，一旦修改了，下面的325行也要进行修改
                  Organizations:
                      - *Org1
                      - *Org2
      # 通道
      ItcastOrgsChannel: # 这个通道的名字可以进行修改
          Consortium: SampleConsortium
          <<: *ChannelDefaults
          Application:
              <<: *ApplicationDefaults
              Organizations:
                  - *Org1
                  - *Org2
              Capabilities:
                  <<: *ApplicationCapabilities
  
      SampleDevModeKafka:
          <<: *ChannelDefaults
          Capabilities:
              <<: *ChannelCapabilities
          Orderer:
              <<: *OrdererDefaults
              OrdererType: kafka
              Kafka:
                  Brokers:
                  - kafka.example.com:9092
  
              Organizations:
              - *OrdererOrg
              Capabilities:
                  <<: *OrdererCapabilities
          Application:
              <<: *ApplicationDefaults
              Organizations:
              - <<: *OrdererOrg
          Consortiums:
              SampleConsortium:
                  Organizations:
                  - *Org1
                  - *Org2
      # 视频中没有，自己读一下，（这个好像是关于集群的）
      SampleMultiNodeEtcdRaft:
          <<: *ChannelDefaults
          Capabilities:
              <<: *ChannelCapabilities
          Orderer:
              <<: *OrdererDefaults
              OrdererType: etcdraft
              EtcdRaft:
                  Consenters:
                  - Host: orderer.example.com
                    Port: 7050
                    ClientTLSCert: crypto-config/ordererOrganizations/example.com/orderers/orderer.example.com/tls/server.crt
                    ServerTLSCert: crypto-config/ordererOrganizations/example.com/orderers/orderer.example.com/tls/server.crt
                  - Host: orderer2.example.com
                    Port: 7050
                    ClientTLSCert: crypto-config/ordererOrganizations/example.com/orderers/orderer2.example.com/tls/server.crt
                    ServerTLSCert: crypto-config/ordererOrganizations/example.com/orderers/orderer2.example.com/tls/server.crt
                  - Host: orderer3.example.com
                    Port: 7050
                    ClientTLSCert: crypto-config/ordererOrganizations/example.com/orderers/orderer3.example.com/tls/server.crt
                    ServerTLSCert: crypto-config/ordererOrganizations/example.com/orderers/orderer3.example.com/tls/server.crt
                  - Host: orderer4.example.com
                    Port: 7050
                    ClientTLSCert: crypto-config/ordererOrganizations/example.com/orderers/orderer4.example.com/tls/server.crt
                    ServerTLSCert: crypto-config/ordererOrganizations/example.com/orderers/orderer4.example.com/tls/server.crt
                  - Host: orderer5.example.com
                    Port: 7050
                    ClientTLSCert: crypto-config/ordererOrganizations/example.com/orderers/orderer5.example.com/tls/server.crt
                    ServerTLSCert: crypto-config/ordererOrganizations/example.com/orderers/orderer5.example.com/tls/server.crt
              Addresses:
                  - orderer.example.com:7050
                  - orderer2.example.com:7050
                  - orderer3.example.com:7050
                  - orderer4.example.com:7050
                  - orderer5.example.com:7050
  
              Organizations:
              - *OrdererOrg
              Capabilities:
                  <<: *OrdererCapabilities
          Application:
              <<: *ApplicationDefaults
              Organizations:
              - <<: *OrdererOrg
          Consortiums:
              SampleConsortium:
                  Organizations:
                  - *Org1
                  - *Org2
  ```

  > 至此配置文件已经按照需求修改完毕，现在要使用configtxgen模块进行生成创始区块和通道

  - **执行命令生成文件**

    > 要注意的是，要区分系统通道和普通通道，在生成文件时要填写不同的名字，在创建创世区块时的-channelID XXX1要为小写而且此时为系统通道，当在创建通道文件时的-channelID XXX2为不同通道名字同样也要是小写

    1. 先生成创世区块
    
       ```shell
       # 先在当前目录下生成一个文件夹channel-artifacts
       aaa@aaa-G3-3579:~/go/src/myfabric$ configtxgen -channelID sc -profile ItcastOrgsOrdererGenesis -outputBlock ./genesis.block
       2021-05-06 16:55:31.933 CST [common.tools.configtxgen] main -> INFO 001 Loading configuration
       2021-05-06 16:55:32.031 CST [common.tools.configtxgen.localconfig] completeInitialization -> INFO 002 orderer type: solo
       2021-05-06 16:55:32.031 CST [common.tools.configtxgen.localconfig] Load -> INFO 003 Loaded configuration: /home/aaa/go/src/myfabric/configtx.yaml
       2021-05-06 16:55:32.101 CST [common.tools.configtxgen] doOutputBlock -> INFO 004 Generating genesis block
       2021-05-06 16:55:32.102 CST [common.tools.configtxgen] doOutputBlock -> INFO 005 Writing genesis block
       ```

       解释：其中` configtxgen`为模块，`-channelID sc`指定系统通道的ID参数，`-profile ItcastOrgsOrdererGenesis`指定在`configtx.yaml`中配置的创世区块的名字，` -outputBlock ./genesis.block`指定生成的创始区块的输出。执行完成后会发现在当前文件夹下生成了一个`genesis.block`文件。

       > 注意：a、fabric1.4要比视频多一个`-channelID` 参数；b、要将生成的创始块文件放在`channel-artifacts`目录中，这样方便后的面的操作；**c、现在指定的channelID的名字则为我们真正的通道名字。**d、-profile后面的参数就分别是我们在config.yaml末尾的地方配置的名字。

    2. 生成通道文件
    
       ```shell
       aaa@aaa-G3-3579:~/go/src/myfabric$ configtxgen -channelID mc -profile ItcastOrgsChannelDDD -outputCreateChannelTx channel.tx
       2021-05-06 17:13:55.397 CST [common.tools.configtxgen] main -> INFO 001 Loading configuration
       2021-05-06 17:13:55.458 CST [common.tools.configtxgen.localconfig] Load -> INFO 002 Loaded configuration: /home/aaa/go/src/myfabric/configtx.yaml
       2021-05-06 17:13:55.458 CST [common.tools.configtxgen] doOutputChannelCreateTx -> INFO 003 Generating new channel configtx
       2021-05-06 17:13:55.459 CST [common.tools.configtxgen] doOutputChannelCreateTx -> INFO 004 Writing new channel tx
       ```

       不做过多解释，类似于1中。执行完之后可以发现多了一个channel.tx的文件。

    3. 锚节点的更新

       > 这个可以不用操作，只是但我们后面想要对锚节点进行更新的时候再进行配置。所以我现在跳过
    
    至此本步已经完成！

# 3. docker-compose文件的编写

> 因为节点等很多东西运行在docker容器中的，所以我们需要编写一个docker-compose文件对那些容器进行统一的管理和启动。

### 3.1 先对官方例子进行分析

```yaml
# Copyright IBM Corp. All Rights Reserved.
#
# SPDX-License-Identifier: Apache-2.0
#
# 版本号
version: '2'

# 卷，与本地的电脑进行映射
volumes:
  orderer.example.com:
  peer0.org1.example.com:
  peer1.org1.example.com:
  peer0.org2.example.com:
  peer1.org2.example.com:

# 网络，启动的docker容器要在同一个网络中才能进行通信
networks:
  byfn:

# 服务
services:

  orderer.example.com: # 服务名，一般服务名与后面的主机名一致
    extends: # 该服务继承的一些属性
      file:   base/docker-compose-base.yaml
      service: orderer.example.com
    container_name: orderer.example.com # 主机的名字
    networks: # 当前容器处于刚才的的网络中
      - byfn

  peer0.org1.example.com:
    container_name: peer0.org1.example.com
    extends:
      file:  base/docker-compose-base.yaml
      service: peer0.org1.example.com
    networks:
      - byfn

  peer1.org1.example.com:
    container_name: peer1.org1.example.com
    extends:
      file:  base/docker-compose-base.yaml
      service: peer1.org1.example.com
    networks:
      - byfn

  peer0.org2.example.com:
    container_name: peer0.org2.example.com
    extends:
      file:  base/docker-compose-base.yaml
      service: peer0.org2.example.com
    networks:
      - byfn

  peer1.org2.example.com:
    container_name: peer1.org2.example.com
    extends:
      file:  base/docker-compose-base.yaml
      service: peer1.org2.example.com
    networks:
      - byfn


  # 客户端节点，可以是一个linux终端（即命令行的形式），也可以是用goSDK、pythonSDK写
  cli:
    container_name: cli
    image: hyperledger/fabric-tools:$IMAGE_TAG # 当前终端对应一个fabric镜像
    tty: true # 终端是否打开
    stdin_open: true # 标准输入是否打开
    environment: # 环境变量
      - SYS_CHANNEL=$SYS_CHANNEL # 这里填写系统通道名字
      - GOPATH=/opt/gopath
      - CORE_VM_ENDPOINT=unix:///host/var/run/docker.sock
      #- FABRIC_LOGGING_SPEC=DEBUG
      - FABRIC_LOGGING_SPEC=INFO
      - CORE_PEER_ID=cli
      - CORE_PEER_ADDRESS=peer0.org1.example.com:7051
      - CORE_PEER_LOCALMSPID=Org1MSP
      - CORE_PEER_TLS_ENABLED=true
      - CORE_PEER_TLS_CERT_FILE=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/org1.example.com/peers/peer0.org1.example.com/tls/server.crt
      - CORE_PEER_TLS_KEY_FILE=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/org1.example.com/peers/peer0.org1.example.com/tls/server.key
      - CORE_PEER_TLS_ROOTCERT_FILE=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/org1.example.com/peers/peer0.org1.example.com/tls/ca.crt
      - CORE_PEER_MSPCONFIGPATH=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/org1.example.com/users/Admin@org1.example.com/msp
    # 工作目录
    working_dir: /opt/gopath/src/github.com/hyperledger/fabric/peer 
    # 命令
    command: /bin/bash
    # 卷的挂载
    volumes:
        - /var/run/:/host/var/run/
        - ./../chaincode/:/opt/gopath/src/github.com/chaincode # 链代码的位置
        - ./crypto-config:/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/ # 存放所有的证书
        - ./scripts:/opt/gopath/src/github.com/hyperledger/fabric/peer/scripts/ # 脚本，即SDK代码的位置
        - ./channel-artifacts:/opt/gopath/src/github.com/hyperledger/fabric/peer/channel-artifacts # 存放我们之前生成的文件（xxx.tx和xxx.block）
    depends_on: # 决定服务启动的顺序，因为一个服务对应一个容器，所以就是容器启动的顺序
      - orderer.example.com
      - peer0.org1.example.com
      - peer1.org1.example.com
      - peer0.org2.example.com
      - peer1.org2.example.com
    networks:
      - byfn
```



> 注意：a、现在把1和2中生成的文件放入到`channel-artifacts`文件夹中，方便在docker-compose中挂载 ；b、如果用docker-compose启动该配置文件，则有五个服务（容器）被启动：
>
> - orderer.example.com
> - peer0.org1.example.com
>
>   - peer1.org1.example.com
>   - peer0.org2.example.com
>   - peer1.org2.example.com
>   - 还有一个cil容器

- 接下来对`base/docker-compose-base.yaml`和`peer-base`进行解读

  >  因为刚才配置文件中的容器都继承了它，它在`fabric-samples/first-network/base`中
  >
  > **注意：修改环境这几个配置文件的时候一定要慢，一定要读懂每个环境变量的意思**

。。。。。。。





​	**启动容器**

- 第一次（出错）

  ```shell
  aaa@aaa-G3-3579:~/go/src/myfabric$ docker-compose down
  WARNING: The SYS_CHANNEL variable is not set. Defaulting to a blank string.
  Stopping cli                     ... done
  Stopping peer0.orgGo.itcast.com  ... done
  Stopping peer1.orgCpp.itcast.com ... done
  Stopping peer1.orgGo.itcast.com  ... done
  Stopping peer0.orgCpp.itcast.com ... done
  Removing cli                     ... done
  Removing peer0.orgGo.itcast.com  ... done
  Removing peer1.orgCpp.itcast.com ... done
  Removing peer1.orgGo.itcast.com  ... done
  Removing peer0.orgCpp.itcast.com ... done
  Removing orderer.itcast.com      ... done
  Removing network myfabric_byfn
  # 发现出现警告
  # 错误原因是没有设置docker-compose.yaml中的cli服务设置- SYS_CHANNEL=，将我之前创建的通道ID加上后再次启动
  ```

- 第二次（出错）

  ```shell
  aaa@aaa-G3-3579:~/go/src/myfabric$ docker-compose up -d
  Creating network "myfabric_byfn" with the default driver
  Creating peer1.orgGo.itcast.com  ... done
  Creating orderer.itcast.com      ... done
  Creating peer0.orgGo.itcast.com  ... done
  Creating peer0.orgCpp.itcast.com ... done
  Creating peer1.orgCpp.itcast.com ... done
  Creating cli                     ... done
  
  # 居然没有出警告或者错误，但是此时不能说明他们都成功创建，要通过下面的命令查看是否都正常启动
  aaa@aaa-G3-3579:~/go/src/myfabric$ docker-compose ps
           Name                 Command       State                       Ports                    
  -------------------------------------------------------------------------------------------------
  cli                       /bin/bash         Up                                                   
  orderer.itcast.com        orderer           Exit 2                                               
  peer0.orgCpp.itcast.com   peer node start   Up       0.0.0.0:9051->9051/tcp,:::9051->9051/tcp    
  peer0.orgGo.itcast.com    peer node start   Up       0.0.0.0:7051->7051/tcp,:::7051->7051/tcp    
  peer1.orgCpp.itcast.com   peer node start   Up       0.0.0.0:10051->10051/tcp,:::10051->10051/tcp
  peer1.orgGo.itcast.com    peer node start   Up       0.0.0.0:8051->8051/tcp,:::8051->8051/tcp    
  
  # 发现orderer节点容器居然没有启动，只有一个错误信息，无法排查错误，我就去仔细检查一遍配置信息，但是都没有找到结果，突然发现vscode有docker插件，可以看到哪些容器被正常启动。修改后开始第三次。
  
  ```

  > 第二次中的经验：
  >
  > 1.不是看到全是done就是成功启动，而是要通过`docker-compose ps`查看时候都正常up，更简单的方法是在vscode中的docker插件看他们是否正常启动。
  >
  > 2.若出现错误，可以去vscode看那些容器启动时候的日志，可以轻松找出错误。
  >
  > 3.且每重新起一次之前要把之前的容器down掉

没有找出错误！！！！！！！找到错误原因是**通道名称不能有大写的字母！！！！**

> 但是现在又出现两个错误情况：
>
> - 删除所有配置文件，再重新创建的时候，通道的名字即使修改了但是还是以前的那个，但是百度半天也找不到答案，所以耽搁几天，几天后该问题消失，出现下面的问题
> - orderer节点还是起不来，查看log发现报错Not bootstrapping because of 3 existing channels，再去查是就找到了[解决方案](https://blog.csdn.net/jingzi123456789/article/details/109820997)，是因为挂载卷的时候出现的问题
>
> 通过那个教程成功解决，经过猜测，出现第一个问题的原因也多半与卷的挂载有关
>
> 

成功启动！！！！！！！

```shell
aaa@aaa-G3-3579:~/go/src/myfabric$ docker-compose up -d
Creating network "myfabric_byfn" with the default driver
Creating volume "myfabric_orderer.itcast.com" with default driver
Creating peer0.orgGo.itcast.com  ... done
Creating peer1.orgGo.itcast.com  ... done
Creating orderer.itcast.com      ... done
Creating peer0.orgCpp.itcast.com ... done
Creating peer1.orgCpp.itcast.com ... done
Creating cli                     ... done
#虽然出现了六个done，但是此时不能说明他们都成功创建，要通过下面的命令查看是否都正常启动
aaa@aaa-G3-3579:~/go/src/myfabric$ docker-compose ps
         Name                 Command       State              Ports            
--------------------------------------------------------------------------------
cli                       /bin/bash         Up                                  
orderer.itcast.com        orderer           Up      0.0.0.0:7050->7050/tcp,:::7050->7050/tcp                
peer0.orgCpp.itcast.com   peer node start   Up      0.0.0.0:9051->9051/tcp,:::9051->9051/tcp                
peer0.orgGo.itcast.com    peer node start   Up      0.0.0.0:7051->7051/tcp,:::70 51->7051/tcp                
peer1.orgCpp.itcast.com   peer node start   Up      0.0.0.0:10051->10051/tcp,::: 10051->10051/tcp            
peer1.orgGo.itcast.com    peer node start   Up      0.0.0.0:8051->8051/tcp,:::8051->8051/tcp                
```

此步到此完成！！！！！！

> 小tips：有时会出现`2021-05-14 11:56:38.467 UTC [main] InitCmd -> ERRO 001 Cannot run peer because error when setting up MSP of type bccsp from directory /etc/hyperledger/fabric/msp: Setup error: nil conf reference`错误，在启动`docker-compose up -d`时加上`sudo`

# 4. channel管理

> 这一步的操作如下：
>
> 1. 创建通道
>    - 通过客户端链接节点创建通道（可以不用链接orderer节点，但是命令参数里面要用到oerder节点的信息）
>
> 2.  将节点加入通道
>    - 也是通过客户端链接节点让节点加入到通道，由于我们只起了一个cil客户端，所以需要将其分别连接点每个节点，将对应的节点加入到通道中。
>
> 3. 安装链代码，是第五步的事情

### 4.1. 创建通道

####  4.1.1. 进入到客户端节点容器

```shell
$ docker exec -it cli /bin/bash
```

或者在vscode中使用docker插件直接进入到cli容器

#### 4.1.2. 使用peer命令创建通道

先介绍命令含义

```shell
$ peer channel create -o oerder节点的地址:端口 -c 通道名 -f 通道文件 --tls true --cafile orderer节点的pem格式证书文件
```

注意：1.此时cli连接的可以不是orderer节点，但是要用orderer节点的信息；2.里面的路径是cli容器里面的路径，即挂载后的路径，且通道文件可以用相对路径，但是证书文件必须用绝对路径; 3. 通道名就是前面的创建通道文件是否设置的通道名，<font color="red">切记不能有大写字母！！！！！</font>

创建通道

```shell
$ peer channel create -o orderer.itcast.com:7050 -c mc -f /opt/gopath/src/github.com/hyperledger/fabric/peer/channel-artifacts/channel.tx --tls true --cafile /opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/ordererOrganizations/itcast.com/msp/tlscacerts/tlsca.itcast.com-cert.pem
# 启动失败，出一下错误信息
Error: got unexpected status: BAD_REQUEST -- error validating channel creation transaction for new channel 'mc', could not succesfully apply update to template configuration: error authorizing update: error validating DeltaSet: policy for [Group]  /Channel/Application not satisfied: implicit policy evaluation failed - 0 sub-policies were satisfied, but this policy requires 1 of the 'Admins' sub-policies to be satisfied

# 万般无奈下，更换为fabric1.4.11，fabric-ca1.4.9，配置文件已备份在桌面
# 进入到cli容器中再次执行
$ peer channel create -o orderer.itcast.com:7050 -c mc -f /opt/gopath/src/github.com/hyperledger/fabric/peer/channel-artifacts/channel.tx --tls true --cafile /opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/ordererOrganizations/itcast.com/msp/tlscacerts/tlsca.itcast.com-cert.pem
# 创建成功！！！成功的标志是当前目录下会多出一个mc.block文件
```

### 4.2 将节点加入到channel

#### 4.2.1 说明

- 使用到的命令

```shell
$ peer channel join -b xxxx.block
# 上面的xxxx.block为刚才创建通道生成的通道文件
```

- 因为加入节点需要客户端操作，所以需要切换客户端链接不同的节点，<font color="red">即要更换客户端的环境变量来控制多个节点加入到通道中，使用export来修改！！</font>

  > 当然也可以创建四个cli去连接四个节点操作

  ```shell
  # OrgGo组织的peer0.orggo.itcast.com
  export CORE_PEER_ADDRESS=peer0.orggo.itcast.com:7051
  export CORE_PEER_LOCALMSPID=OrgGoMSP  # 组织ID
  export CORE_PEER_TLS_CERT_FILE=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/orggo.itcast.com/peers/peer0.orggo.itcast.com/tls/server.crt
  export CORE_PEER_TLS_KEY_FILE=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/orggo.itcast.com/peers/peer0.orggo.itcast.com/tls/server.key
  export CORE_PEER_TLS_ROOTCERT_FILE=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/orggo.itcast.com/peers/peer0.orggo.itcast.com/tls/ca.crt
  export CORE_PEER_MSPCONFIGPATH=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/orggo.itcast.com/users/Admin@orggo.itcast.com/msp
  
  # OrgGo组织的peer1.orggo.itcast.com
  export CORE_PEER_ADDRESS=peer1.orggo.itcast.com:8051
  export CORE_PEER_LOCALMSPID=OrgGoMSP  # 组织ID
  export CORE_PEER_TLS_CERT_FILE=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/orggo.itcast.com/peers/peer1.orggo.itcast.com/tls/server.crt
  export CORE_PEER_TLS_KEY_FILE=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/orggo.itcast.com/peers/peer1.orggo.itcast.com/tls/server.key
  export CORE_PEER_TLS_ROOTCERT_FILE=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/orggo.itcast.com/peers/peer1.orggo.itcast.com/tls/ca.crt
  export CORE_PEER_MSPCONFIGPATH=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/orggo.itcast.com/users/Admin@orggo.itcast.com/msp
  
  # OrgCpp组织的peer0.orgcpp.itcast.com
  export CORE_PEER_ADDRESS=peer0.orgcpp.itcast.com:9051
  export CORE_PEER_LOCALMSPID=OrgCppMSP  # 组织ID
  export CORE_PEER_TLS_CERT_FILE=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/orgcpp.itcast.com/peers/peer0.orgcpp.itcast.com/tls/server.crt
  export CORE_PEER_TLS_KEY_FILE=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/orgcpp.itcast.com/peers/peer0.orgcpp.itcast.com/tls/server.key
  export CORE_PEER_TLS_ROOTCERT_FILE=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/orgcpp.itcast.com/peers/peer0.orgcpp.itcast.com/tls/ca.crt
  export CORE_PEER_MSPCONFIGPATH=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/orgcpp.itcast.com/users/Admin@orgcpp.itcast.com/msp
  
  # OrgCpp组织的peer1.orgcpp.itcast.com
  export CORE_PEER_ADDRESS=peer1.orgcpp.itcast.com:10051
  export CORE_PEER_LOCALMSPID=OrgCppMSP  # 组织ID
  export CORE_PEER_TLS_CERT_FILE=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/orgcpp.itcast.com/peers/peer1.orgcpp.itcast.com/tls/server.crt
  export CORE_PEER_TLS_KEY_FILE=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/orgcpp.itcast.com/peers/peer1.orgcpp.itcast.com/tls/server.key
  export CORE_PEER_TLS_ROOTCERT_FILE=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/orgcpp.itcast.com/peers/peer1.orgcpp.itcast.com/tls/ca.crt
  export CORE_PEER_MSPCONFIGPATH=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/orgcpp.itcast.com/users/Admin@orgcpp.itcast.com/msp
  ```

  > 使用上面四个环境变量，在cli容器中修改环境变量，用peer1.orggo.itcast.com举例

  ```shell
  # 进入到cli容器中
  $ export CORE_PEER_ADDRESS=peer1.orggo.itcast.com:8051
  $ export CORE_PEER_LOCALMSPID=OrgGoMSP  # 组织ID
  $ export CORE_PEER_TLS_CERT_FILE=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/orggo.itcast.com/peers/peer1.orggo.itcast.com/tls/server.crt
  $ export CORE_PEER_TLS_KEY_FILE=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/orggo.itcast.com/peers/peer1.orggo.itcast.com/tls/server.key
  $ export CORE_PEER_TLS_ROOTCERT_FILE=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/orggo.itcast.com/peers/peer1.orggo.itcast.com/tls/ca.crt
  $ export CORE_PEER_MSPCONFIGPATH=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/orggo.itcast.com/users/Admin@orggo.itcast.com/msp
  
  #环境变量修改成功，可以使用`echo $export CORE_PEER_ADDRESS`查看环境变量是否修改成功
  $ echo $export CORE_PEER_ADDRESS
  输出：peer1.orggo.itcast.com:8051则表示修改成功
  # 接下来节点加入到通道中
  peer cahnnel join -b mc.block
  ```

  按照上面步骤四个节点都成功加入！！！

# 5. chaincode的安装和实例化

> 接第四步，接下来，用到的链码我们现用sample中的链代码，由于在设置的时候忘记改链代码的位置，所以链代码的位置在itcast之外
>
> - 安装、实例化
>
> - 初始化 
>
> - 操作链代码 

#### 5.1安装链代码

> 每个节点都需要安装链代码，我们先从fabric sample1.4.11中把链代码复制到宿主机怪哉链代码的位置，复制成功，在cli容器中查看链码已经正常存在

安装链代码的命令如下截图

![ ](/home/aaa/.config/tencent-qq//AppData/file//sendpix1.jpg)

> <font color="red">尝试几次都失败了，我感觉是因为引入的包有问题，所以我将例子中的chaincode中的abac文件也复制到itcast的chaincode中（为了保险，我也把abac中的go也单独复制过来，具体见下图），结果成功！！</font>

![](/home/aaa/.config/tencent-qq//AppData/file//sendpix3.jpg)



```shell
# 进入到cli容器中
$ root@d1661b5b04cb:/opt/gopath/src/github.com/hyperledger/fabric/peer# peer chaincode install -n testcc -v 0.1 -p github.com/chaincode/
# 最后看到这个时说明成功（200和OK）
2021-05-15 07:33:02.360 UTC [chaincodeCmd] install -> INFO 050 Installed remotely response:<status:200 payload:"OK" > 
```



#### 5.2 初始化链代码

> - <font color="red">初始化链代码需要发布一个交易</font>
> - <font color="red">初始化链代码相当于执行init函数</font>
> - <font color="red">只需要在一个节点上初始化即可</font>

![](/home/aaa/.config/tencent-qq//AppData/file//sendpix2.jpg)

> 注意：
>
> - 通道名、链码名称、版本要与安装链代码的时候一致
> - <font color="red">-P背书策略是指这个在被使用中创建的交易时的策略，而不是本次初始化链代码而产生的交易的策略</font>

```shell
peer chaincode instantiate -o orderer.itcast.com:7050 --tls true --cafile /opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/ordererOrganizations/itcast.com/msp/tlscacerts/tlsca.itcast.com-cert.pem -C mc -n testcc -l goland -v 0.1 -c '{"Args":["init","a","100","b","200"]}' -P "AND ('OrgGoMSP.member', 'OrgCppMSP.member')"
```







> <font color="red">记录一下，标签的概念应该可以通过复合键来实现，！！！</font>