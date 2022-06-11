Skyine是一个现代化的dashboard用户交互系统，由skyline-console、skyline-apiserver和nginx组成。

下面讲述skyline部署。

# 1.先决条件

存在一个openstack核心组件的平台环境，至少包括keystone、nova等核心组件。kolla-ansible安装的openstack环境满足这一条件。

# 2.创建数据库

openstack集群使用mariadb数据库，在该数据库软件中创建skyline数据库。
Openstack集群中mariadb数据库是多主同步，所以只需在一个节点创建数据库即可，其他节点会自动同步。

```bash
# 登录到openstack集群某个节点
ssh root@lynxi-node-01 
# 查询数据库root用户的密码
cat /etc/kolla/passwords.yml | grep -e ^database_password:
#database_password: oPbtTqfqzVEmtXMpGo1dTmKllTJO3yTnJBBYU

docker exec -it mariadb bash
# 数据库登录
mysql -u root -p
oPbtTqfqzVEmtXMpGo1dTmKllTJO3yTnJBBYU
# 创建skyline数据库和数据库账号
CREATE DATABASE IF NOT EXISTS skyline DEFAULT CHARACTER SET utf8 DEFAULT COLLATE utf8_general_ci;
GRANT ALL PRIVILEGES ON skyline.* TO 'skyline'@'localhost' IDENTIFIED BY '1Y3PTsuvZCy6Rn3rjZRwQEUj7QqfosRRxcrl6';
GRANT ALL PRIVILEGES ON skyline.* TO 'skyline'@'%'  IDENTIFIED BY '1Y3PTsuvZCy6Rn3rjZRwQEUj7QqfosRRxcrl6';
```

# 3.注册skyline账号

在keystone中创建skyline账号。

```bash
# 通过身份认证（登录）
source /etc/kolla/admin-openrc.sh
# 创建skyline系统的账号
openstack user create --domain default --password-prompt skyline
#TsuvZCy6Rn3rjZRwskyline

openstack role add --project service --user skyline admin
```

# 4.配置skyline.yaml

在skyline.yaml.sample的基础上修改如下内容：

```yaml
  database_url: mysql://skyline:1Y3PTsuvZCy6Rn3rjZRwQEUj7QqfosRRxcrl6@192.168.232.6:3306/skyline

  interface_type: admin
  keystone_url: http://192.168.232.6:35357/v3/

  system_user_name: skyline
  system_user_password: TsuvZCy6Rn3rjZRwskyline
```
以上内容由三部分组成：数据库链接、认证中心接口、skyline系统的账号密码（在keystone中注册）。

将skyline配置文件复制到/etc/skyline/目录下

```bash
sudo  mkdir /etc/skyline -p
sudo cp skyline.yaml /etc/skyline/
```

# 5.部署服务

```bash
# 引导skyline
docker run -d --name skyline_bootstrap -e KOLLA_BOOTSTRAP="" -v /etc/skyline/skyline.yaml:/etc/skyline/skyline.yaml --net=host 99cloud/skyline:latest
# 启动skyline服务
docker run -d --name skyline --restart=always -v /etc/skyline/skyline.yaml:/etc/skyline/skyline.yaml --net=host 99cloud/skyline:latest
```

# 6.体验

Skyline的登录认证和horizon的一样，都是由keyston组件提供身份认证服务。所以登录账号密码和horizon的一样。

![用户界面](./asset/skyline-view.png)

# 附件：skyline.yaml.sample

[skyline-apiserver/skyline.yaml.sample at master · openstack/skyline-apiserver (github.com)](https://github.com/openstack/skyline-apiserver/blob/master/etc/skyline.yaml.sample)

```yaml
default:
  access_token_expire: 3600
  access_token_renew: 1800
  cors_allow_origins: []
  database_url: sqlite:////tmp/skyline.db
  debug: false
  log_dir: ./log
  prometheus_basic_auth_password: ''
  prometheus_basic_auth_user: ''
  prometheus_enable_basic_auth: false
  prometheus_endpoint: http://localhost:9091
  secret_key: aCtmgbcUqYUy_HNVg5BDXCaeJgJQzHJXwqbXr0Nmb2o
  session_name: session
openstack:
  base_domains:
  - heat_user_domain
  base_roles:
  - keystone_system_admin
  - keystone_system_reader
  - keystone_project_admin
  - keystone_project_member
  - keystone_project_reader
  - nova_system_admin
  - nova_system_reader
  - nova_project_admin
  - nova_project_member
  - nova_project_reader
  - cinder_system_admin
  - cinder_system_reader
  - cinder_project_admin
  - cinder_project_member
  - cinder_project_reader
  - glance_system_admin
  - glance_system_reader
  - glance_project_admin
  - glance_project_member
  - glance_project_reader
  - neutron_system_admin
  - neutron_system_reader
  - neutron_project_admin
  - neutron_project_member
  - neutron_project_reader
  - heat_system_admin
  - heat_system_reader
  - heat_project_admin
  - heat_project_member
  - heat_project_reader
  - placement_system_admin
  - placement_system_reader
  - panko_system_admin
  - panko_system_reader
  - panko_project_admin
  - panko_project_member
  - panko_project_reader
  - ironic_system_admin
  - ironic_system_reader
  - octavia_system_admin
  - octavia_system_reader
  - octavia_project_admin
  - octavia_project_member
  - octavia_project_reader
  default_region: RegionOne
  extension_mapping:
    fwaas_v2: neutron_firewall
    vpnaas: neutron_vpn
  interface_type: public
  keystone_url: http://localhost:5000/v3/
  nginx_prefix: /api/openstack
  reclaim_instance_interval: 604800
  service_mapping:
    baremetal: ironic
    compute: nova
    container: zun
    container-infra: magnum
    database: trove
    identity: keystone
    image: glance
    key-manager: barbican
    load-balancer: octavia
    network: neutron
    object-store: swift
    orchestration: heat
    placement: placement
    sharev2: manilav2
    volumev3: cinder
  system_admin_roles:
  - admin
  - system_admin
  system_project: service
  system_project_domain: Default
  system_reader_roles:
  - system_reader
  system_user_domain: Default
  system_user_name: skyline
  system_user_password: password
setting:
  base_settings:
  - flavor_families
  - gpu_models
  - usb_models
  flavor_families:
  - architecture: x86_architecture
    categories:
    - name: general_purpose
      properties: []
    - name: compute_optimized
      properties: []
    - name: memory_optimized
      properties: []
    - name: high_clock_speed
      properties: []
  - architecture: heterogeneous_computing
    categories:
    - name: compute_optimized_type_with_gpu
      properties: []
    - name: visualization_compute_optimized_type_with_gpu
      properties: []
  gpu_models:
  - nvidia_t4
  usb_models:
  - usb_c
```

# 参考文章

[kolla---openstack(Xena)----skyline部署体验_Paniling的博客-CSDN博客_kolla](https://blog.csdn.net/qq_37692227/article/details/124712999) 