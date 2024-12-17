# 2. WeDPR部署脚本介绍

标签: ``部署脚本`` ``wedpr-builder``

----

为了简化部署，WeDPR提供了部署脚本[wedpr-builder](https://github.com/WeBankBlockchain/WeDPR/tree/main/wedpr-builder)来帮助运维人员快速生成平台各个服务的配置、创建脚本和启停脚本，该部署脚本基于Docker构建WeDPR隐私计算平台。

配置文件`config.toml`是wedpr-builder生成服务配置的关键，其包括基础配置、各机构的HDFS、MySQL、网关、站点端服务、PSI服务、MPC服务等配置。本章重点介绍该部署脚本。

```eval_rst
.. note::
   - 最新的部署脚本可通过命令 ``curl -#LO https://github.com/WeBankBlockchain/WeDPR/releases/download/v3.0.0/wedpr-builder.tar.gz && tar -xvf wedpr-builder.tar.gz`` 下载获取
   - 使用python3.x运行部署脚本 
   - 使用部署脚本前，请先运行 ``pip3 install -r requirements.txt`` 安装脚本所需依赖
```

******
## 2.1 部署脚本使用说明

`build_wedpr.py`是部署脚本的主程序，可通过`python build_wedpr.py -h`命令查看部署脚本的使用帮助，如下:


```bash
python build_wedpr.py -h
usage: build_wedpr.py [-h] [-o OPERATION] [-c CONFIG] [-d OUTPUT] [-t TYPE]

examples:
 * generate node config:	 python3 build_wedpr.py -t wedpr-node
 * generate gateway config:	 python3 build_wedpr.py -t wedpr-gateway
 * generate mpc config:	 python3 build_wedpr.py -t wedpr-mpc
 * generate wedpr-site config:	 python3 build_wedpr.py -t wedpr-site
 * generate wedpr-pir config:	 python3 build_wedpr.py -t wedpr-pir
* generate wedpr-model service config:	 python3 build_wedpr.py -t wedpr-model
 * generate gateway config:	 python3 build_wedpr.py -o genconfig -c config.toml -t wedpr-gateway -d wedpr-generated
 * generate node config:	 python3 build_wedpr.py -o genconfig -c config.toml -t wedpr-node -d wedpr-generated

optional arguments:
  -h, --help            show this help message and exit
  -o OPERATION, --operation OPERATION
                        [Optional] specify the command:
                        * supported command list: genconfig
  -c CONFIG, --config CONFIG
                        [Optional] the config file, default is config.toml
  -d OUTPUT, --output OUTPUT
                        [Optional] the output path, default is wedpr-generated
  -t TYPE, --type TYPE  [Required] the service type:
                        * now support: wedpr-node, wedpr-gateway, wedpr-site, wedpr-pir, wedpr-jupyter-worker, wedpr-model, wedpr-mpc
```

各参数选项的功能如下：

- `-h` 或 `--help`: 输出帮助信息
- `-c` 或 `--config`: 指定配置文件，默认是`config.toml`
- `-d` 或 `--output`: 指定配置文件的输出路径，默认是`wedpr-generated`
- `-t` 或 `--type`: 指定生成配置对应的服务类型

`-t`选项指定生成的配置文件类型包括:

- `wedpr-node`: 指定生成PSI服务配置，用于运行PSI算法
- `wedpr-gateway`: 指定生成网关配置，用于支持跨机构的网络通信
- `wedpr-site`: 指定生成站点端管理台配置，包括Jupyter配置，用于管理隐私计算平台各种资源
- `wedpr-pir`: 指定生成PIR服务配置，用于运行PIR服务
- `wedpr-mpc`: 指定生成MPC服务配置，用于运行联合分析等MPC类型任务
- `wedpr-model`: 指定生成联合建模服务配置，用于运行SecureLGBM和SecureLR等类型的建模任务

********
## 2.2 部署脚本基础配置介绍

部署脚本配置示例位于`conf/config-example.toml`中，包括环境配置、各个机构的服务配置等。本节介绍该配置，方便用户根据实际情况修改该配置文件，部署自定义拓扑的隐私计算网络。

********
### 2.2.1 环境配置[env]
`[env]`配置了隐私计算服务的配置生成目录、物料等信息，具体如下：

```eval_rst
.. note::
   非开发者不用关注 ``binary_path``, ``spdz_home`` , ``wedpr_site_dist_path``, ``wedpr_pir_dist_path`` , ``wedpr_model_source_path`` 等配置，这些配置用于支持开发者在开发模式下快速搭建非Docker模式的隐私计算测试环境，入门门槛较高
```

```toml
#### define the binary path ###
# the binary path for ppc-pro-node and ppc-gateway-service
binary_path = "/data/home/wedpr/WeDPR-Component/cpp/build/bin"
# the spdz path
spdz_home = ""
# the dist path for wedpr-site
wedpr_site_dist_path = "/data/home/wedpr/WeDPR/wedpr-site/dist/"
# the dist path for wedpr-dir
wedpr_pir_dist_path = "/data/home/wedpr/WeDPR/wedpr-pir/dist/"
# the wedpr model source path,
# pls clone from https://github.com/WeBankBlockchain/WeDPR-Component
wedpr_model_source_path = "/data/home/wedpr/WeDPR-Component/python/"

# use docker mode or not
docker_mode = true

#### define the docker images desc ###
wedpr_gateway_service_image_desc = "fiscoorg/wedpr-gateway-service:v3.0.0"
wedpr_node_service_image_desc = "fiscoorg/wedpr-pro-node-service:v3.0.0"
wedpr_mpc_service_image_desc = "fiscoorg/wedpr-mpc-service:v3.0.0"
wedpr_jupyter_worker_image_desc = "fiscoorg/wedpr-jupyter-worker:v3.0.0"
wedpr_model_image_desc = "fiscoorg/wedpr-model-service:v3.0.0"
wedpr_site_image_desc = "fiscoorg/wedpr-site:v3.0.0"
wedpr_pir_image_desc = "fiscoorg/wedpr-pir:v3.0.0"


deploy_dir = "wedpr-example"
# the wedpr zone(used to distinguish different privacy computing environments)
zone = "wedpr.zone.default"
```

下面配置主要用于非Docker模式搭建隐私计算环境，非开发者不推荐使用，主要包括:
- `binary_path`: 指定PSI服务二进制(`ppc-pro-node`)和网关服务二进制(`ppc-gateway-service`)所在目录
- `spdz_home`: 指定MPC服务所在路径
- `wedpr_site_dist_path`: 指定站点端服务的Jar包目录
- `wedpr_pir_dist_path`: 指定PIR服务的Jar包目录
- `wedpr_model_source_path`: 指定建模服务的源码路径

*********
下面配置主要用于Docker模式下搭建隐私计算环境时，配置各服务的镜像信息，一般使用`conf/config-example.toml`自带的镜像配置即可, 若升级版本，也仅需修改版本号信息，具体如下：

- `docker_mode`: 是否基于Docker部署WeDPR系统，默认为`true`
- `wedpr_gateway_service_image_desc`: 网关服务镜像信息，`v3.0.0`版本的网关服务镜像信息是`fiscoorg/wedpr-gateway-service:v3.0.0`
- `wedpr_node_service_image_desc`: PSI服务的镜像信息，`v3.0.0`版本的PSI服务镜像信息是`fiscoorg/wedpr-pro-node-service:v3.0.0`
- `wedpr_mpc_service_image_desc`: MPC服务的镜像信息， `v3.0.0`版本的MPC服务镜像信息是`fiscoorg/wedpr-mpc-service:v3.0.0`
- `wedpr_jupyter_worker_image_desc`: Jupyter服务的镜像信息，`v3.0.0`版本的Jupyter服务镜像信息是`fiscoorg/wedpr-jupyter-worker:v3.0.`
- `wedpr_model_image_desc`: 建模服务的镜像信息，`v3.0.0`版本的建模服务镜像信息是`fiscoorg/wedpr-model-service:v3.0.0`
- `wedpr_site_image_desc`: 站点端管理服务的镜像信息，`v3.0.0`版本的站点端服务镜像信息是`fiscoorg/wedpr-site:v3.0.0`
- `wedpr_pir_image_desc`: PIR服务的镜像信息，`v3.0.0`版本的PIR服务镜像信息是`fiscoorg/wedpr-pir:v3.0.0`

********
- `deploy_dir`: 指定配置生成的目录，默认配置为`wedpr-example`
- `zone`: 指定该WeDPR隐私计算系统的名称，用于支持同机构搭建多套隐私计算环境且共用网关时网络上的隔离，默认配置为`wedpr.zone.default`

*******
### 2.2.2 区块链配置[blockchain]

WeDPR隐私计算系统通过[FISCO BCOS v3.0](https://fisco-bcos-doc.readthedocs.io/zh-cn/latest/index.html)区块链系统同步并存证各机构可公开的元数据信息，因此部署WeDPR隐私计算系统前，须先按照FISCO BCOS v3.0的指引，部署好区块链环境，并在WeDPR系统中配置区块链信息。

区块链的配置信息如下:

```toml
[blockchain]
# Required, the group id
blockchain_group = "group0"
# Required, the blockchain peers
blockchain_peers = []
# Required, the blockchain cert path
blockchain_cert_path = ""
# Required, the contract address for recorder factory contract
recorder_factory_contract_address = ""
# Required, the contract address for sequencer contract
sequencer_contract_address = ""
```

- `blockchain_group`: WeDPR使用的区块链节点的群组id，一般为`group0`, 可通过区块链节点的`config.genesis`中的`chain.group_id`选项获取，区块链节点配置可参考[这里](https://fisco-bcos-doc.readthedocs.io/zh-cn/latest/docs/tutorial/air/config.html)
- `blockchain_peers`: 连接的区块链节点列表，可通过区块链节点的`config.ini`中的`rpc.listen_port`配置选项获取，为了提升系统健壮性，建议至少配置两个区块链节点连接，区块链节点配置可参考[这里](https://fisco-bcos-doc.readthedocs.io/zh-cn/latest/docs/tutorial/air/config.html)
- `blockchain_cert_path`: 区块链sdk证书配置路径，该证书获取方式与FISCO BCOS控制台证书获取方式类似，具体可参考[这里](https://fisco-bcos-doc.readthedocs.io/zh-cn/latest/docs/quick_start/air_installation.html#id7)
- `recorder_factory_contract_address`: WeDPR元数据工厂合约地址，用于链上元数据同步，该合约可通过`curl -#LO https://github.com/WeBankBlockchain/WeDPR/releases/download/v3.0.0/wedpr-sol.tar.gz` 命令下载，并在部署WeDPR隐私计算平台前部署于FISCO BCOS区块链上，具体步骤可参考[隐私计算平台部署配置](../quick_start/standalone_installation.html#id4)中的【步骤五：部署隐私计算合约】
- `sequencer_contract_address`: WeDPR定序合约，用于对链上元数据定序，类似于元数据工厂合约，该合约也通过`curl -#LO https://github.com/WeBankBlockchain/WeDPR/releases/download/v3.0.0/wedpr-sol.tar.gz` 命令下载，须在部署WeDPR隐私计算平台前部署与区块链上，具体步骤可参考[隐私计算平台部署配置](../quick_start/standalone_installation.html#id4)中的【步骤五：部署隐私计算合约】


********
## 2.3 部署脚本机构配置

由于WeDPR隐私计算系统部署于多个机构中，该系统的核心配置均位于机构配置`[[agency]]`中，一个WeDPR隐私计算系统至少包括两个机构配置，也可按需部署2+个机构。配置示例`conf/config-example.toml`中包含了两个机构`agency0`和`agency1`的配置。

机构的基础配置包括:

- `[[agency]].name`:  机构名
- `[[agency]].holding_msg_minutes`: 网关消息的超时时间，默认为30
- `[[agency]].wedpr_api_token`: 机构内系统间通信的api token，可不配置，部署脚本会根据一定规则生成该token

除了基础配置外，机构配置还包括网关配置、HDFS服务配置、MYSQL配置、站点端配置、PIR服务配置、建模服务配置、Jupyter服务配置、MPC服务配置、PSI服务配置。下面分别详细介绍这些配置。

**********
### 2.3.1 网关配置[agency.gateway]


网关的配置示例如下，主要包括服务的部署IP, 监听端口，连接信息等。

```toml
    [agency.gateway]
    deploy_ip=["127.0.0.1:2"]
    # gateway listen ip
    listen_ip="0.0.0.0"
    # gateway listen start port
    listen_port=40300
    # the thread count
    thread_count = 4
    # the grpc config
    # gateway grpc server listen ip
    grpc_listen_ip="0.0.0.0"
    # gateway grpc server listen start port
    grpc_listen_port=40600
    # gateway connected peers, should be all of the gateway peers info
    [[agency.gateway.peers]]
        agency = "agency0"
        endpoints = ["127.0.0.1:40300", "127.0.0.1:40301"]
    [[agency.gateway.peers]]
        agency = "agency1"
        endpoints = ["127.0.0.1:40320", "127.0.0.1:40321"]
```

- `deploy_ip`:  网关的部署ip，可配置多个，每个部署ip采用`${ip}:${count}`的格式，配置示例`conf/config-example.toml`默认为每个机构在本机部署两个网关
- `listen_ip`: 网关的监听ip，保持为默认值`0.0.0.0`即可
- `listen_port`: 网关的监听端口，用于进行跨机构通信，请确保该端口开放给了规划机构的网关节点访问
- `thread_count`: 网关线程池数目，保持默认值4即可
- `grpc_listen_ip`: 网关对内的监听ip，保持为默认值`0.0.0.0`即可
- `grpc_listen_port`: 网关对内的监听端口，用于介绍机构内服务的消息，并将其转发至其他机构，请确保该端口不要与其他服务端口冲突

*****
`[[agency.gateway.peers]]`以机构维度配置了网关连接信息，包括该网关连接的机构以及该机构的网关节点访问`endpoint`列表，具体如下：

- `[[agency.gateway.peers]].agency`: 该网关连接的机构名称
- `[[agency.gateway.peers]].endpoints`: 该网关连接的机构中网关节点的连接信息，包括ip和连接端口



*******
### 2.3.2 HDFS服务配置[agency.hdfs]

为了支持用户级别的数据托管和隔离，WeDPR需要自建HDFS，搭建步骤可参考【依赖环境搭建】章节中的[搭建HDFS](pre_installation.html#hdfs)。

HDFS搭建完毕后，需要在`config.toml`中配置HDFS的连接信息。

```eval_rst
.. note::
   - 这里HDFS也可接入使用已有的HDFS环境
   - 在开发专家模式Jupyter应用场景中，建议HDFS开启krb5鉴权
```

```toml
    # configuration for hdfs
    [agency.hdfs]
        user = "root"
        home = "/user/wedpr/agency0"
        name_node = "127.0.0.1"
        name_node_port = 9000
        webfs_port = 50070
        token = ""
        # enable auth or not, default is false
        enable_krb5_auth = false
        # the hdfs kerberos auth principal, used when enable_krb5_auth
        auth_principal = "root@NODE.DC1.CONSUL"
        # the hdfs kerberos auth password, used when enable_krb5_auth
        auth_password = ""
        # the ccache path, used when enable_krb5_auth
        ccache_path = "/tmp/krb5cc_ppc_node"
        # the krb5 conf path
        krb5_conf_path = "conf/krb5.conf"
        # keytab path
        krb5_keytab_path = ""
        # auth host name override
        auth_host_name_override = ""
```

- `user`: HDFS系统的访问用户
- `home`: 该机构所有资源在HDFS上的路径
- `name_node`: HDFS name-node的访问ip
- `name_node_prt`: HDFS name-node的访问端口
- `webfs_port`: HDFS webfs访问端口，一般为50070
- `enable_krb5_auth`: 是否启用krb5鉴权，默认为false
- `auth_principal`: krb5认证主体域名
- `auth_password`: krb5认证口令
- `ccache_path`: krb5私钥缓存路径
- `krb5_conf_path`: krb5配置路径
- `krb5_keytab_path`: krb5私钥路径
- `auth_host_name_override`: HDFS krb5 hostname名称

*****
### 2.3.3 MYSQL服务配置[agency.mysql]

配置了站点端服务、PIR服务、建模服务访问的DB信息(支持国产数据库)。配置示例如下：

```eval_rst
.. note::
   - 部署隐私计算节点前，请确保该配置指定的数据库存在
```

```toml
    # the agency mysql configuration
    [agency.mysql]
        host = "127.0.0.1"
        port = "3306"
        user = "root"
        password = ""
        database = "agency0"
```

- `host`: 数据库的连接ip
- `port`: 数据库连接端口
- `user`: 数据库登录用户
- `password`: 数据库登录密码
- `database`: 该机构使用的数据库名称，默认为机构名

******
### 2.3.4 站点端服务配置[agency.site]

配置了站点端管理服务的基本信息，配置示例如下：

```toml
    #configuration for wedpr site
    [agency.site]
        # the site deploy ip
        deploy_ip=["127.0.0.1:2"]
        # the server port
        server_start_port = "16000"
```

- `deploy_ip`：服务的部署ip，支持多IP部署；配置示例`conf/config-example.toml`默认为每个机构生成2个站点服务
- `server_start_port`: 指定该服务的起始端口，用于端口分配，单机部署时，请确保预留足够端口，防止端口冲突。

****
### 2.3.5 PIR服务配置[agency.pir]

配置了PIR服务的基本信息，配置示例如下：

```toml
    # configuration for wedpr pir
    [agency.pir]
        # the pir deploy ip
        deploy_ip=["127.0.0.1:2"]
        # the server start port
        server_start_port = "17000"
```

- `deploy_ip`：服务的部署ip，支持多IP部署；配置示例`conf/config-example.toml`默认为每个机构生成2个PIR服务
- `server_start_port`: 指定该服务的起始端口，用于端口分配，单机部署时，请确保预留足够端口，防止端口冲突

*******
### 2.3.6 建模服务配置[agency.model]

配置了建模服务的基本信息，配置示例如下：

```toml
    # configuration for wedpr model
    [agency.model]
        deploy_ip = ["127.0.0.1:2"]
        server_start_port = "18000"
```
- `deploy_ip`：服务的部署ip，支持多IP部署；配置示例`conf/config-example.toml`默认为每个机构生成2个建模服务
- `server_start_port`: 指定该服务的起始端口，用于端口分配，单机部署时，请确保预留足够端口，防止端口冲突

******
### 2.3.7 Jupyter服务配置[agency.jupyter_worker]

配置了Jupyter服务的基本信息，配置示例如下：

```eval_rst
.. note::
   - 生成站点端服务配置时，会默认生成Jupyter服务配置，请确保Jupyter服务正确配置
```

```toml
   # configuration for wedpr jupyter worker
    [agency.jupyter_worker]
        deploy_ip = ["127.0.0.1:1"]
        # the server start port
        server_start_port = "19000"
        jupyter_external_ip = ""
```
- `deploy_ip`：服务的部署ip，支持多IP部署；配置示例`conf/config-example.toml`默认为每个机构生成1个Jupyter服务
- `server_start_port`: 指定该服务的起始端口，用于端口分配，单机部署时，请确保预留足够端口，防止端口冲突

******
### 2.3.8 MPC服务配置[agency.mpc]

配置了MPC服务的基本信息，配置示例如下：

```toml
 # configuration for mpc
    [agency.mpc]
        deploy_ip = ["127.0.0.1:1"]
        # the server start port
        server_start_port = "20000"
        external_ip = ""
```
- `deploy_ip`：服务的部署ip，支持多IP部署; 配置示例`conf/config-example.toml`默认为每个机构生成1个MPC服务
- `server_start_port`: 指定该服务的起始端口，用于端口分配，单机部署时，请确保预留足够端口，防止端口冲突

*****
### 2.3.9 PSI服务配置[[agency.node]]

由于PSI节点支持的算法类型比较多，因此PSI服务配置选项比较多；但WeDPR隐私计算平台当前仅用了`CM2020`和`多方ECDH`隐私求交集算法，因此这里仅介绍与此相关的核心配置，没列出来的配置不用关注，保持默认值即可。
配置示例如下：

```toml
    # configuration for the wedpr nodes
    [[agency.node]]
        deploy_ip=["127.0.0.1:2"]
        # node grpc server listen ip
        grpc_listen_ip="0.0.0.0"
        # node grpc server listen port
        grpc_listen_port=40402
        # the rpc config for the node
        [agency.node.rpc]
            listen_ip = "0.0.0.0"
            listen_port = 10200
            thread_count = 4
```

- `deploy_ip`: 部署IP，支持多IP部署; 配置示例`conf/config-example.toml`默认为每个机构生成2个PSI服务
- `grpc_listen_ip`: PSI的grpc监听IP，保持为`0.0.0.0`即可
- `grpc_listen_port`: PSI服务连接网关的监听端口，请确保不要与其他服务监听端口冲突

********
`[agency.node.rpc]`定义了该PSI服务的RPC连接信息，包括:
- `[agency.node.rpc].listen_ip`: 监听ip，保持为`0.0.0.0`即可
- `[agency.node.rpc].listen_port`: 监听端口，请确保不与其他服务监听端口冲突
- `[agency.node.rpc].thread_count`: 线程数目，保持为默认值4即可


综上，本章介绍了`wedpr-builder`部署脚本的所有配置选项，运维和开发同学可按需修改这些配置选项，搭建符合自定义需求的WeDPR隐私计算平台。