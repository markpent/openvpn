# openvpn Cookbook

[![Cookbook Version](https://img.shields.io/cookbook/v/openvpn.svg)](https://supermarket.chef.io/cookbooks/openvpn)
[![Build Status](https://img.shields.io/circleci/project/github/sous-chefs/openvpn/master.svg)](https://circleci.com/gh/sous-chefs/openvpn)
[![OpenCollective](https://opencollective.com/sous-chefs/backers/badge.svg)](#backers)
[![OpenCollective](https://opencollective.com/sous-chefs/sponsors/badge.svg)](#sponsors)
[![License](https://img.shields.io/badge/License-Apache%202.0-green.svg)](https://opensource.org/licenses/Apache-2.0)

Installs OpenVPN and sets up a fairly basic configuration. Since OpenVPN is very complex, we provide a baseline only (see **Customizing Server Configuration** below).

## Maintainers

This cookbook is maintained by the Sous Chefs. The Sous Chefs are a community of Chef cookbook maintainers working together to maintain important cookbooks. If you’d like to know more please visit [sous-chefs.org](https://sous-chefs.org/) or come chat with us on the Chef Community Slack in [#sous-chefs](https://chefcommunity.slack.com/messages/C2V7B88SF).

## Requirements

### Platforms

- Debian 8+
- Ubuntu 14.04+
- RHEL 6.x and 7.x w/ (EPEL is enabled as required)
- CentOS 6.x, 7.x
- Fedora
- OpenSUSE 42+ (partial support/WIP)
- Arch Linux
- FreeBSD 11+ (partial support/WIP)

Note: we currently only test the latest minor release for the last 2 major releases of each OS/distribution using Test Kitchen.

### Cookbooks

- sysctl
- yum-epel

### Not Supported

This cookbook is designed to set up a basic installation of OpenVPN that will work for many common use cases. The following configurations are not supported by default with this cookbook:

- setting up routers and other network devices
- ethernet-bridging (tap interfaces)
- dual-factor authentication
- many other advanced OpenVPN configurations

For further modification of the cookbook see **Usage** below.

For more information about OpenVPN, see the [official site](http://openvpn.net/).

## Attributes

These attributes are set by the cookbook by default.

- `node['openvpn']['client_cn']` - The client's Common Name used with the `openvpn::client` recipe (essentially a standalone recipe) for the client certificate and key.
- `node['openvpn']['type']` - Valid values are 'client' (currently a work in progress), 'server' or 'server-bridge'. Default is 'server' and it will create a routed IP tunnel, and use the 'tun' device. 'server-bridge' will create an ethernet bridge and requires a tap0 device bridged with the ethernet interface, and is beyond the scope of this cookbook.
- `node['openvpn']['subnet']` - Used for server mode to configure a VPN subnet to draw client addresses. Default is 10.8.0.0, which is what the sample OpenVPN config package uses.
- `node['openvpn']['netmask']` - Netmask for the subnet, default is 255.255.0.0.
- `node['openvpn']['gateway']` - FQDN for the VPN gateway server. Default is `node['fqdn']`.
- `node['openvpn']['push_routes']` - Array of routes to to push to clients (as `push` statements) in the server.conf, e.g. '192.168.0.0 255.255.255.0'. Default is empty.
- `node['openvpn']['push_options']` - Array of options to push to clients in the server.conf, e.g. [["dhcp-option DNS", ["8.8.8.8"]]]. Default is empty.
- `node['openvpn']['configure_default_server']` - Boolean. Set this to false if you want to create all of your "conf" files with the LWRP.
- `node['openvpn']['git_package']` - Boolean. Whether to use the `openvpn-git` package (Arch Linux only, default false).
- `node['openvpn']['client_prefix']` - String. Name of the config that is created for clients. When imported into most vpn clients, this is the name that will be displayed for the connection. Default is 'vpn-prod'.
- `node['openvpn']['cookbook_user_conf']` - String. The cookbook used by the `openvpn::users` recipe for the `client.conf.erb` template. You can override this to your own, such as your wrapper cookbook. Default is `'openvpn'`.
- `node['openvpn']['key_dir']` - Location to store keys, certificates and related files. Default `/etc/openvpn/keys`.
- `node['openvpn']['signing_ca_cert']` - CA certificate for signing, default `/etc/openvpn/keys/ca.crt`
- `node['openvpn']['signing_ca_key']` - CA key for signing, default `/etc/openvpn/keys/ca.key`
- `node['openvpn']['server_verification']` - Server certificate verification directive, can be anything mentioned [in official doc](https://openvpn.net/index.php/open-source/documentation/howto.html#mitm). By default `nil`.
- `node['openvpn']['config']['local']` - IP to listen on, defaults to `node['ipaddress']`
- `node['openvpn']['config']['proto']` - Valid values are 'udp' or 'tcp', defaults to 'udp'.
- `node['openvpn']['config']['port']` - Port to listen on, defaults to '1194'.
- `node['openvpn']['config']['log']` - Server log file. Default /var/log/openvpn.log
- `node['openvpn']['config']['script-security']` - Script Security setting to use in server config. Default is 1\. The "up" script will not be included in the configuration if this is 0 or 1\. Set it to 2 to use the "up" script.

The following attributes are used to populate the `easy-rsa` vars file. Defaults are the same as the vars file that ships with OpenVPN.

- `node['openvpn']['key']['ca_expire']` - In how many days should the root CA key expire - `CA_EXPIRE`.
- `node['openvpn']['key']['expire']` - In how many days should certificates expire - `KEY_EXPIRE`.
- `node['openvpn']['key"]['size']` - Default key size, set to 2048 if paranoid but will slow down TLS negotiation performance - `KEY_SIZE`.

The following are for the default values for fields place in the certificate from the vars file. Do not leave these blank.

- `node['openvpn']['key']['country']` - `KEY_COUNTRY`
- `node['openvpn']['key']['province']` - `KEY_PROVINCE`
- `node['openvpn']['key']['city']` - `KEY_CITY`
- `node['openvpn']['key']['org']` - `KEY_ORG`
- `node['openvpn']['key']['email']` - `KEY_EMAIL`

The following lets you specify the message digest used for generating certificates by OpenVPN

- `node['openvpn']['key']['message_digest']` - Default is `sha256` for a high-level of security.

The CRL will be generated, and refreshed automatically, allowing you to
revoke certificates

- `node['openvpn']['key']['crl_expire']` - In how many days should the CRL expire? Will be refreshed after half of this time

## Recipes

### `openvpn::default`

Installs the OpenVPN package only.

### `openvpn::install`

Installs the OpenVPN package only.

### `openvpn::server`

Installs and configures OpenVPN as a server.

### `openvpn::client`

Installs and configures OpenVPN as a client.

### `openvpn::service`

Manages the OpenVPN system service (there is no need to use this recipe directly in your run_list).

### `openvpn::users`

Utilizes a data bag called `users` to generate OpenVPN keys for each user. [chef-solo-search](https://github.com/edelight/chef-solo-search) is required in order to use this recipe with Chef-Solo, although it is not a dependency of this cookbook.

### `openvpn::easy_rsa`

Installs the easy-rsa package (a CLI utility to build and manage a PKI CA).

### Usage

Create a role for the OpenVPN server. See above for attributes that can be entered here.

```ruby
name "openvpn"
description "The server that runs OpenVPN"
run_list("recipe[openvpn::server]")
override_attributes(
  "openvpn" => {
    "gateway" => "vpn.example.com",
    "subnet" => "10.8.0.0",
    "netmask" => "255.255.0.0",
    "key" => {
      "country" => "US",
      "province" => "CA",
      "city" => "SanFrancisco",
      "org" => "Fort-Funston",
      "email" => "me@example.com"
    }
  }
)
```

**Note**: If you are using a Red Hat EL distribution, the EPEL repository is automatically enabled by Chef's `recipe[yum::epel]` to install the openvpn package.

To push routes to clients, add `node['openvpn']['push_routes]` as an array attribute, e.g. if the internal network is 192.168.100.0/24:

```ruby
override_attributes(
  "openvpn" => {
    "push_routes" => [
      "192.168.100.0 255.255.255.0"
    ]
  }
)
```

To push other options to clients, use the `node['openvpn']['push_options']` attribute and set an array of hashes or strings. For example:

```ruby
override_attributes(
  "openvpn" => {
    "push_options" => {
      "dhcp-option" => [
        "DOMAIN domain.local",
        "DOMAIN-SEARCH domain.local"
      ],
      "string-option" => "string value"
    }
  }
)
```

This will render a config file that looks like:

```ruby
push "dhcp-option DOMAIN domain.local"
push "dhcp-option DOMAIN-SEARCH domain.local"
push "string-option string value"
```

To automatically create new certificates and configurations for users, create data bags for each user. The only content required is the `id`, but this can be used in conjunction with other cookbooks by Chef Software such as `users` or `samba`. See **SSL Certificates** below for more about generating client certificate sets.

```javascript
{
  "id": "jtimberman"
}
```

This cookbook also provides an 'up' script that runs when OpenVPN is started. This script is for setting up firewall rules and kernel networking parameters as needed for your environment. Modify to suit your needs, upload the cookbook and re-run chef on the openvpn server. For example, you'll probably want to enable IP forwarding (sample Linux setting is commented out). The attribute `node['openvpn']["script_security"]` must be set to 2 or higher to use this otherwise openvpn server startup will fail.

## Resources

### openvpn_user

Implements a resource for creation of users and bundles.

### openvpn_config

Given a hash of config options it writes out individual openvpn config files.

If you don't want to use the default "server.conf" from the default recipe, set `node['openvpn']["configure_default_server"]` to false, then use this resource to configure things as you like.

## Customizing Server Configuration

To further customize the server configuration, there are two templates that can be modified in this cookbook.

- templates/default/server.conf.erb
- templates/default/server.up.sh.erb

The first is the OpenVPN server configuration file. Modify to suit your needs for more advanced features of [OpenVPN](http://openvpn.net). The second is an `up` script run when OpenVPN starts. This is where you can add firewall rules, enable IP forwarding and other OS network settings required for OpenVPN. Attributes in the cookbook are provided as defaults, you can add more via the openvpn role if you need them.

## SSL Certificates

Some of the easy-rsa tools are copied to /etc/openvpn/easy-rsa to provide the minimum to generate the certificates using the default and users recipes. We provide a Rakefile to make it easier to generate client certificate sets if you're not using the data bags above. To generate new client certificates you will need `rake` installed (either as a gem or a package), then run:

```shell
cd /etc/openvpn/easy-rsa
source ./vars
rake client name="CLIENT_NAME" gateway="vpn.example.com"
```

Replace `CLIENT_NAME` and `vpn.example.com` with your desired values. The rake task will generate a tar.gz file with the configuration and certificates for the client.

## Contributors

This project exists thanks to all the people who [contribute.](https://opencollective.com/sous-chefs/contributors.svg?width=890&button=false)

### Backers

Thank you to all our backers!

![https://opencollective.com/sous-chefs#backers](https://opencollective.com/sous-chefs/backers.svg?width=600&avatarHeight=40)

### Sponsors

Support this project by becoming a sponsor. Your logo will show up here with a link to your website.

![https://opencollective.com/sous-chefs/sponsor/0/website](https://opencollective.com/sous-chefs/sponsor/0/avatar.svg?avatarHeight=100)
![https://opencollective.com/sous-chefs/sponsor/1/website](https://opencollective.com/sous-chefs/sponsor/1/avatar.svg?avatarHeight=100)
![https://opencollective.com/sous-chefs/sponsor/2/website](https://opencollective.com/sous-chefs/sponsor/2/avatar.svg?avatarHeight=100)
![https://opencollective.com/sous-chefs/sponsor/3/website](https://opencollective.com/sous-chefs/sponsor/3/avatar.svg?avatarHeight=100)
![https://opencollective.com/sous-chefs/sponsor/4/website](https://opencollective.com/sous-chefs/sponsor/4/avatar.svg?avatarHeight=100)
![https://opencollective.com/sous-chefs/sponsor/5/website](https://opencollective.com/sous-chefs/sponsor/5/avatar.svg?avatarHeight=100)
![https://opencollective.com/sous-chefs/sponsor/6/website](https://opencollective.com/sous-chefs/sponsor/6/avatar.svg?avatarHeight=100)
![https://opencollective.com/sous-chefs/sponsor/7/website](https://opencollective.com/sous-chefs/sponsor/7/avatar.svg?avatarHeight=100)
![https://opencollective.com/sous-chefs/sponsor/8/website](https://opencollective.com/sous-chefs/sponsor/8/avatar.svg?avatarHeight=100)
![https://opencollective.com/sous-chefs/sponsor/9/website](https://opencollective.com/sous-chefs/sponsor/9/avatar.svg?avatarHeight=100)
