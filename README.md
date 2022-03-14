# peervpn ubuntu installer
## A serie of ansible roles to build and install peervpn package on Ubuntu 18.04

It consists of several main parts (roles and etc):
1. source-builder (role1)
2. hosting-creator (role2)
3. pkg-installer (role3)
4. sample inventory
5. sample group vars
6. test_build.yaml sample role run script

## source-builder role

1. Installs prerequisites
 - an Ubuntu 18.04 system needs to have git, make, gcc to fetch source from [Peervpn git repo](https://github.com/peervpn/peervpn) and build it
 - package build routine doesn;t support installed libssl 1.1 libs, it needs to have libssl 1.0 or some sort of libressl of supported version. So I just decideded to use a simplier way for tis time (install libssl1.0-dev, a fast downgrade way)
2. Builds packages into {{ peervpn_build_path }} directory. At last it make a peervpn binary file ready to be copied and executed elsewhere.

## hosting-creator role

1. Installs a dpkg-dev package to make some .deb-prepare steps
2. Makes a local apt-repo
3. Deploys nginx http server to host a repo to make it accessible remotely elsewhere.

## pkg-installer role

1. Adds a custom repo from {{ repo_hostname }} path
2. Updates repo list with --allow-insecure-repositories option as it doesn't have a gpg signing
3. Installs a peervpn package from a custom repo

## sample inventory 
inventories/main.yml
It has only one host for me as I tested it there. Could be one host for each role or any other case

## sample group vars
Each role has its defaults/main.yaml vars to make it working, a sample role file has all the variables which one could customize to make roles working the other way:

|Variable name|Default value|Description|
|-------------|-------------|-----------|
|repo_path|/opt/apt-repo|Local repo path|
|repo_ip_addr|127.0.0.1|Hostname-ip /etc/hosts hack to make nginx host a virtual domain config|
|repo_hostname|peervpn.repo|The same thint as earlier option but a hostname part of it|
|peervpn_build_path|/opt/build|Path where peervpn source would be fetched and builded|
|peervpn_deb_version|0.0.1|Version field for apt package|
|peervpn_deb_maintainer|klimblinton|Maintainer field for apt package|
|peervpn_deb_depends|libssl1.0-dev, libcrypto++-dev, zlib1g-dev|Depends field for apt package|
|peervpn_deb_package_name|peervpn-{{ peervpn_deb_version }}_amd64.deb|Apt .deb package filename|
|peervpn_deb_package_fullpath|{{ peervpn_build_path }}/deb/{{ peervpn_deb_package_name }}|The full path where to put and to get .deb package file after creation|
|peervpn_git_repo_path|https://github.com/peervpn/peervpn|Git repo path where to get peervpn source|

## Cleanup before rerun (default values)
- remove /opt/apt-repo
- remove /opt/build
- remove /etc/nginx/conf.d/peervpn_nginx.conf and reload it
- remove package source file /etc/apt/sources.list.d/peervpn.list and run apt-get update
So its now safe to rerun and recheck roles

## TODO

1. Make something like cleanup role to clean role dids to have ability to simplyfy rerun it once again
2. Test with libressl dependency (without libssl downgrading)
3. Make gpg repo signing
4. Make and test roles ability to work good on any redhat/debian/ubuntu platforms