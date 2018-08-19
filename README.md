# Requirements
- Ansible (Tested on v2.5.5 - May not work on lower versions)
- Vagrant (Tested on v2.1.2)
- Virtual Box

# Quick start
```vagrant up```

# Summary
This project is an example for using Vagrant to provision a HAProxy LB VM along with multiple application VMs that run a simple Hello World PHP application based on the Slim framework. The application servers make use of NGINX and PHP-FPM.

Vagrant utilises Ansible for server side configuration. The following external roles are imported automatically as defined in the VagrantFile:

- HAProxy - https://github.com/entercloudsuite/ansible-haproxy
- NGINX role - https://github.com/nginxinc/ansible-role-nginx
- PHP role - https://github.com/geerlingguy/ansible-role-php
- Composer - https://github.com/geerlingguy/ansible-role-composer

A default number of two application servers are provisioned, although the script is dynamic and will allow for provisioning many more.

Ansible is required on the host machine. Due to the need for provisioning multiple application servers, it's quicker and cleaner to have Ansible configure multiple hosts in parallel rather than on a host by host basis.

The NGINX VHOST is configured to make use of the default `/vagrant` share. The VHOST document root is `/vagrant/slim/`


# Test evidence

The last Ansible provisioner task performs a test to ensure the application servers are accessible. Ansible will query HAProxy to ensure each backend server is enabled. The available backend serves (application servers) are defined by the number of hosts defined in the Ansible 'apps' inventory group.

The final test results in `N` No. of requests targeting HAProxy. If the response body does not contain the string defined by the `haproxy_response_string` variable, the task is marked as fail upon which Ansible will exit.

Included with the installation is a status page provided by HAProxy - http://localhost:8081. Review status page for confirmation that all backend servers are operational and are serving requests.


# Variables
The following variables can be defined within the VagrantFile:
- `N`: Default = 2 - Define the number of application servers to provision
- `ip_lb`: Default = "192.168.77.200" - Define LB private IP. Application servers increment based on this variable e.g. 2 applications servers are assigned "192.168.77.201" & "192.168.77.202"
- `server_name`: Default = "" - Leave blank if you're happy to connect using only localhost, e.g. http://localhost:8080 - Specifying a hostname appends the hostname to the NGINX server_name option, e.g. localhost example.com
- `haproxy_web_port`: Default = 8080 - Define port to use when making requests to HAProxy
- `haproxy_status_port`: Default = 8081 - Define port to use when accessing HAProxy stats page.
- `haproxy_disable_stats`: Default = true - Disable HAProxy stats authentication
- `haproxy_response_string`: Default = "Hello World!" – Define the string expected in the response body upon testing


# Special Thanks
Vagrant can dynamically create the Ansible inventory upon run. A hosts private key is stored within `.vagrant/machines/{{VM}}/virtualbox/private_key` - https://github.com/hashicorp/vagrant/pull/5765#issuecomment-120247738

One issue I ran into is that each Ansible provisioner task results in previous ansible.group definitions being overwritten, which requires code duplication.

Simple NGINX configuration for supporting slim framework - https://www.slimframework.com/docs/v3/start/web-servers.html

Accessing Ansible host variables, useful when configuring HAProxy backends - https://docs.ansible.com/ansible/latest/user_guide/playbooks_variables.html#magic-variables-and-how-to-access-information-about-other-hosts

Handling IP addresses. Previous solution was to perform string splitting before coming across this post - https://stackoverflow.com/questions/23345017/how-to-increment-the-last-octet-in-an-ip-address

