# Heat
(not an acronym)


# Heat
enables you to deploy
## complete
virtual environments


## Heat
supports two distinct
# formats


# CFN
Amazon CloudFormation compatible template
# HOT
Heat Orchestration Template


HOT is 100%
# YAML


What can we
# do
with Heat?


## `OS::Nova::Server`
Configures Nova guests


```
resources:
  mybox:
    type: "OS::Nova::Server"
    properties:
      name: mybox
      image: trusty-server-cloudimg-amd64
      flavor: m1.small
      key_name: mykey
```


Now we could just
# create
this stack


```sh
heat stack-create -f stack.yml mystack
```


But as it is,

it's not very
# flexible


Let's add some
## parameters


```
parameters:
  flavor:
    type: string
    description: Flavor to use for servers
    default: m1.medium
  image:
    type: string
    description: Image name or ID
    default: trusty-server-cloudimg-amd64
  key_name:
    type: string
    description: Keypair to inject into newly created servers
```


And some
## intrinsic functions


```
resources:
  mybox:
    type: "OS::Nova::Server"
    properties:
      name: mybox
      image: { get_param: image }
      flavor: { get_param: flavor }
      key_name: { get_param: key_name }
```


And now we can
# set
these parameters


```sh
heat stack-create -f stack.yml \
  -P key_name=mykey
  -P image=cirros-0.3.3-x86_64 \
  mystack
```


How about we add some
## network connectivity
Wouldn't that be nice?


### `OS::Neutron::Net`
### `OS::Neutron::Subnet`
Defines Neutron networks


```
  mynet:
    type: "OS::Neutron::Net"
    properties:
      name: management-net

  mysub_net:
    type: "OS::Neutron::Subnet"
    properties:
      name: management-sub-net
      network: { get_resource: management_net }
      cidr: 192.168.122.0/24
      gateway_ip: 192.168.101.1
      enable_dhcp: true
      allocation_pools: 
        - start: "192.168.101.2"
          end: "192.168.101.50"
```


## `get_resource`
Cross-reference between resources

Automatic dependency


### `OS::Neutron::Router`
### `OS::Neutron::RouterGateway`
### `OS::Neutron::RouterInterface`
Configures Neutron routers


```
parameters:
  public_net:
    type: string
    description: Public network ID or name

resources:
  router:
    type: OS::Neutron::Router
  router_gateway:
    type: OS::Neutron::RouterGateway
    properties:
      router: { get_resource: router }
      network: { get_param: public_net }
  router_interface:
    type: OS::Neutron::RouterInterface
    properties:
      router: { get_resource: router }
      subnet: { get_resource: mysub_net }
```


### `OS::Neutron::Port`
Configures Neutron ports


```
  mybox_management_port:
    type: "OS::Neutron::Port"
    properties:
      network: { get_resource: mynet }
```

```
  mybox:
    type: "OS::Nova::Server"
    properties:
      name: deploy
      image: { get_param: image }
      flavor: { get_param: flavor }
      key_name: { get_param: key_name }
      networks:
        - port: { get_resource: mybox_management_port }
```


### `OS::Neutron::FloatingIP`
Allocates floating IP addresses


```
  myfloating_ip:
    type: "OS::Neutron::FloatingIP"
    properties:
      floating_network: { get_param: public_net }
      port: { get_resource: mybox_management_port }
```


### `outputs`
Return stack values or attributes


```
outputs:
  public_ip:
    description: Floating IP address in public network
    value: { get_attr: [ myfloating_ip, floating_ip_address ] }
```


```sh
heat output-show \
  mystack public_ip
```


Integrating
# Heat
with
## `cloud-init`


```
  mybox:
    type: "OS::Nova::Server"
    properties:
      name: deploy
      image: { get_param: image }
      flavor: { get_param: flavor }
      key_name: { get_param: key_name }
      networks:
        - port: { get_resource: mybox_management_port }
      user_data: { get_file: cloud-config.yml }
      user_data_format: RAW
```


### `OS::Heat::CloudConfig`
Manages `cloud-config` directly from Heat


```
resources:
  myconfig:
    type: "OS::Heat::CloudConfig"
    properties:
      cloud_config:
        package_update: true
        package_upgrade: true
```

```
  mybox:
    type: "OS::Nova::Server"
    properties:
      name: deploy
      image: { get_param: image }
      flavor: { get_param: flavor }
      key_name: { get_param: key_name }
      networks:
        - port: { get_resource: mybox_management_port }
      user_data: { get_resource: myconfig }
      user_data_format: RAW
```

