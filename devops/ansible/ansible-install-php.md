# Install httpd & PHP

## Task

> Nautilus Application development team wants to test the Apache and PHP setup on one of the app servers in Stratos Datacenter. They want the DevOps team to prepare an Ansible playbook to accomplish this task. Below you can find more details about the task.<br><br>There is an inventory file `~/playbooks/inventory` on jump host.<br>Create a playbook `~/playbooks/httpd.yml` on jump host and perform the following tasks on App Server 3.<br>a. Install `httpd` and `php` packages (whatever default version is available in yum repo).<br>b. Change default document root of Apache to `/var/www/html/myroot` in default Apache config `/etc/httpd/conf/httpd.conf`. Make sure `/var/www/html/myroot` path exists (if not please create the same).<br>c. There is a template `~/playbooks/templates/phpinfo.php.j2` on jump host. Copy this template to the Apache document root you created as `phpinfo.php` file and make sure user owner and the group owner for this file is `apache` user.<br>d. Start and enable `httpd` service.

## Preliminary Steps

1. Check the architecture map at https://www.lucidchart.com/documents/view/58e22de2-c446-4b49-ae0f-db79a3318e97/0_0
2. Check server details at https://kodekloudhub.github.io/kodekloud-engineer/docs/projects/nautilus

## Research

* Install packages.
  * https://docs.ansible.com/ansible/latest/modules/package_module.html
* Regex replace.
  * https://docs.ansible.com/ansible/latest/modules/replace_module.html
* Files.
  * https://docs.ansible.com/ansible/latest/modules/file_module.html
* Templates.
  * https://docs.ansible.com/ansible/latest/modules/template_module.html
* Services.
  * https://docs.ansible.com/ansible/latest/modules/service_module.html

## Steps

Follow [Passwordless ssh setup](../../linux-system-administrator/networking/passwordless-ssh-access.md) for every user.

```bash
# Check Ansible is install
ansible --version
```

```
ansible 2.9.9
  config file = /etc/ansible/ansible.cfg
  configured module search path = [u'/root/.ansible/plugins/modules', u'/usr/share/ansible/plugins/modules']
  ansible python module location = /usr/lib/python2.7/site-packages/ansible
  executable location = /bin/ansible
  python version = 2.7.5 (default, Jun 20 2019, 20:27:34) [GCC 4.8.5 20150623 (Red Hat 4.8.5-36)]
```

```bash
# Change to working directory
cd ~/playbooks

# Inspect inventory
cat inventory
```

```
stapp01 ansible_host=172.16.238.10 ansible_ssh_pass=Ir0nM@n ansible_user=tony
stapp02 ansible_host=172.16.238.11 ansible_ssh_pass=Am3ric@ ansible_user=steve
stapp03 ansible_host=172.16.238.12 ansible_ssh_pass=BigGr33n ansible_user=bannerth
```

Looks fine.

```bash
# Test inventory
ansible -i inventory all -m ping
```

```
stapp02 | SUCCESS => {
...
stapp03 | SUCCESS => {
...
stapp01 | SUCCESS => {
...
```

Works fine.

```bash
# Create playbook
cat > httpd.yml
```

```yaml
---
- hosts: stapp03
  become: yes

  tasks:
    - name: Install packages
      package:
        name:
          - httpd
          - php
        state: latest

    - name: Replace httpd DocumentRoot
      replace:
        path: /etc/httpd/conf/httpd.conf
        regexp: 'DocumentRoot "/var/www/html"'
        replace: 'DocumentRoot "/var/www/html/myroot"'

    - name: Make sure new httpd DocumentRoot Exists
      file:
        path: /var/www/html/myroot
        owner: root
        group: root
        mode: '0755'
        state: directory

    - name: Template a file to /etc/files.conf
      template:
        src:  ~/playbooks/templates/phpinfo.php.j2
        dest: /var/www/html/myroot/phpinfo.php
        owner: apache
        group: apache
        mode: '0644'

    - name: Start and enable service httpd.
      service:
        name: httpd
        state: started
        enabled: yes
```

Close the file with control + d i.e. `^D`

```bash
# Run playbook
ansible-playbook -i ./inventory httpd.yml
```

```
...
PLAY RECAP ***********************************************************************
stapp02                    : ok=6    changed=5    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
```

Looks good.

```bash
# Check results
curl stapp03/phpinfo.php
```

<details>
  <summary>Returned HTML</summary>
```html
<!DOCTYPE html PUBLIC "-//W3C//DTD XHTML 1.0 Transitional//EN" "DTD/xhtml1-transitional.dtd">
<html xmlns="http://www.w3.org/1999/xhtml">
  <head>
    <style type="text/css">
      body {background-color: #ffffff; color: #000000;}
      body, td, th, h1, h2 {font-family: sans-serif;}
      pre {margin: 0px; font-family: monospace;}
      a:link {color: #000099; text-decoration: none; background-color: #ffffff;}
      a:hover {text-decoration: underline;}
      table {border-collapse: collapse;}
      .center {text-align: center;}
      .center table { margin-left: auto; margin-right: auto; text-align: left;}
      .center th { text-align: center !important; }
      td, th { border: 1px solid #000000; font-size: 75%; vertical-align: baseline;}
      h1 {font-size: 150%;}
      h2 {font-size: 125%;}
      .p {text-align: left;}
      .e {background-color: #ccccff; font-weight: bold; color: #000000;}
      .h {background-color: #9999cc; font-weight: bold; color: #000000;}
      .v {background-color: #cccccc; color: #000000;}
      .vr {background-color: #cccccc; text-align: right; color: #000000;}
      img {float: right; border: 0px;}
      hr {width: 600px; background-color: #cccccc; border: 0px; height: 1px; color: #000000;}
    </style>
    <title>phpinfo()</title>
    <meta name="ROBOTS" content="NOINDEX,NOFOLLOW,NOARCHIVE" />
  </head>
  <body>
    <div class="center">
      <table border="0" cellpadding="3" width="600">
        <tr class="h">
          <td>
            <a href="http://www.php.net/"><img border="0" src="/phpinfo.php?=PHPE9568F34-D428-11d2-A769-00AA001ACF42" alt="PHP Logo" /></a>
            <h1 class="p">PHP Version 5.4.16</h1>
          </td>
        </tr>
      </table>
      <br />
      <table border="0" cellpadding="3" width="600">
        <tr>
          <td class="e">System </td>
          <td class="v">Linux stapp03.stratos.xfusioncorp.com 5.4.0-1102-gcp #111~18.04.2-Ubuntu SMP Tue Mar 21 16:15:46 UTC 2023 x86_64 </td>
        </tr>
        <tr>
          <td class="e">Build Date </td>
          <td class="v">Apr  1 2020 04:08:16 </td>
        </tr>
        <tr>
          <td class="e">Server API </td>
          <td class="v">Apache 2.0 Handler </td>
        </tr>
        <tr>
          <td class="e">Virtual Directory Support </td>
          <td class="v">disabled </td>
        </tr>
        <tr>
          <td class="e">Configuration File (php.ini) Path </td>
          <td class="v">/etc </td>
        </tr>
        <tr>
          <td class="e">Loaded Configuration File </td>
          <td class="v">/etc/php.ini </td>
        </tr>
        <tr>
          <td class="e">Scan this dir for additional .ini files </td>
          <td class="v">/etc/php.d </td>
        </tr>
        <tr>
          <td class="e">Additional .ini files parsed </td>
          <td class="v">/etc/php.d/curl.ini,
            /etc/php.d/fileinfo.ini,
            /etc/php.d/json.ini,
            /etc/php.d/phar.ini,
            /etc/php.d/zip.ini
          </td>
        </tr>
        <tr>
          <td class="e">PHP API </td>
          <td class="v">20100412 </td>
        </tr>
        <tr>
          <td class="e">PHP Extension </td>
          <td class="v">20100525 </td>
        </tr>
        <tr>
          <td class="e">Zend Extension </td>
          <td class="v">220100525 </td>
        </tr>
        <tr>
          <td class="e">Zend Extension Build </td>
          <td class="v">API220100525,NTS </td>
        </tr>
        <tr>
          <td class="e">PHP Extension Build </td>
          <td class="v">API20100525,NTS </td>
        </tr>
        <tr>
          <td class="e">Debug Build </td>
          <td class="v">no </td>
        </tr>
        <tr>
          <td class="e">Thread Safety </td>
          <td class="v">disabled </td>
        </tr>
        <tr>
          <td class="e">Zend Signal Handling </td>
          <td class="v">disabled </td>
        </tr>
        <tr>
          <td class="e">Zend Memory Manager </td>
          <td class="v">enabled </td>
        </tr>
        <tr>
          <td class="e">Zend Multibyte Support </td>
          <td class="v">disabled </td>
        </tr>
        <tr>
          <td class="e">IPv6 Support </td>
          <td class="v">enabled </td>
        </tr>
        <tr>
          <td class="e">DTrace Support </td>
          <td class="v">disabled </td>
        </tr>
        <tr>
          <td class="e">Registered PHP Streams</td>
          <td class="v">https, ftps, compress.zlib, compress.bzip2, php, file, glob, data, http, ftp, phar, zip</td>
        </tr>
        <tr>
          <td class="e">Registered Stream Socket Transports</td>
          <td class="v">tcp, udp, unix, udg, ssl, sslv3, tls</td>
        </tr>
        <tr>
          <td class="e">Registered Stream Filters</td>
          <td class="v">zlib.*, bzip2.*, convert.iconv.*, string.rot13, string.toupper, string.tolower, string.strip_tags, convert.*, consumed, dechunk</td>
        </tr>
      </table>
      <br />
      <table border="0" cellpadding="3" width="600">
        <tr class="v">
          <td>
            <a href="http://www.zend.com/"><img border="0" src="/phpinfo.php?=PHPE9568F35-D428-11d2-A769-00AA001ACF42" alt="Zend logo" /></a>
            This program makes use of the Zend Scripting Language Engine:<br />Zend&nbsp;Engine&nbsp;v2.4.0,&nbsp;Copyright&nbsp;(c)&nbsp;1998-2013&nbsp;Zend&nbsp;Technologies<br />
          </td>
        </tr>
      </table>
      <br />
      <hr />
      <h1><a href="/phpinfo.php?=PHPB8B5F2A0-3C92-11d3-A3A9-4C7B08C10000">PHP Credits</a></h1>
      <hr />
      <h1>Configuration</h1>
      <h2><a name="module_apache2handler">apache2handler</a></h2>
      <table border="0" cellpadding="3" width="600">
        <tr>
          <td class="e">Apache Version </td>
          <td class="v">Apache/2.4.6 (CentOS) PHP/5.4.16 </td>
        </tr>
        <tr>
          <td class="e">Apache API Version </td>
          <td class="v">20120211 </td>
        </tr>
        <tr>
          <td class="e">Server Administrator </td>
          <td class="v">root@localhost </td>
        </tr>
        <tr>
          <td class="e">Hostname:Port </td>
          <td class="v">stapp03.stratos.xfusioncorp.com:0 </td>
        </tr>
        <tr>
          <td class="e">User/Group </td>
          <td class="v">apache(48)/48 </td>
        </tr>
        <tr>
          <td class="e">Max Requests </td>
          <td class="v">Per Child: 0 - Keep Alive: on - Max Per Connection: 100 </td>
        </tr>
        <tr>
          <td class="e">Timeouts </td>
          <td class="v">Connection: 60 - Keep-Alive: 5 </td>
        </tr>
        <tr>
          <td class="e">Virtual Server </td>
          <td class="v">No </td>
        </tr>
        <tr>
          <td class="e">Server Root </td>
          <td class="v">/etc/httpd </td>
        </tr>
        <tr>
          <td class="e">Loaded Modules </td>
          <td class="v">core mod_so http_core mod_access_compat mod_actions mod_alias mod_allowmethods mod_auth_basic mod_auth_digest mod_authn_anon mod_authn_core mod_authn_dbd mod_authn_dbm mod_authn_file mod_authn_socache mod_authz_core mod_authz_dbd mod_authz_dbm mod_authz_groupfile mod_authz_host mod_authz_owner mod_authz_user mod_autoindex mod_cache mod_cache_disk mod_data mod_dbd mod_deflate mod_dir mod_dumpio mod_echo mod_env mod_expires mod_ext_filter mod_filter mod_headers mod_include mod_info mod_log_config mod_logio mod_mime_magic mod_mime mod_negotiation mod_remoteip mod_reqtimeout mod_rewrite mod_setenvif mod_slotmem_plain mod_slotmem_shm mod_socache_dbm mod_socache_memcache mod_socache_shmcb mod_status mod_substitute mod_suexec mod_unique_id mod_unixd mod_userdir mod_version mod_vhost_alias mod_dav mod_dav_fs mod_dav_lock mod_lua prefork mod_proxy mod_lbmethod_bybusyness mod_lbmethod_byrequests mod_lbmethod_bytraffic mod_lbmethod_heartbeat mod_proxy_ajp mod_proxy_balancer mod_proxy_connect mod_proxy_express mod_proxy_fcgi mod_proxy_fdpass mod_proxy_ftp mod_proxy_http mod_proxy_scgi mod_proxy_wstunnel mod_systemd mod_cgi mod_php5 </td>
        </tr>
      </table>
      <br />
      <table border="0" cellpadding="3" width="600">
        <tr class="h">
          <th>Directive</th>
          <th>Local Value</th>
          <th>Master Value</th>
        </tr>
        <tr>
          <td class="e">engine</td>
          <td class="v">1</td>
          <td class="v">1</td>
        </tr>
        <tr>
          <td class="e">last_modified</td>
          <td class="v">0</td>
          <td class="v">0</td>
        </tr>
        <tr>
          <td class="e">xbithack</td>
          <td class="v">0</td>
          <td class="v">0</td>
        </tr>
      </table>
      <br />
      <h2>Apache Environment</h2>
      <table border="0" cellpadding="3" width="600">
        <tr class="h">
          <th>Variable</th>
          <th>Value</th>
        </tr>
        <tr>
          <td class="e">UNIQUE_ID </td>
          <td class="v">ZDoz4MnCVxnBVG64ISuzGgAAAAE </td>
        </tr>
        <tr>
          <td class="e">HTTP_USER_AGENT </td>
          <td class="v">curl/7.29.0 </td>
        </tr>
        <tr>
          <td class="e">HTTP_HOST </td>
          <td class="v">stapp03 </td>
        </tr>
        <tr>
          <td class="e">HTTP_ACCEPT </td>
          <td class="v">*/* </td>
        </tr>
        <tr>
          <td class="e">PATH </td>
          <td class="v">/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin </td>
        </tr>
        <tr>
          <td class="e">SERVER_SIGNATURE </td>
          <td class="v"><i>no value</i> </td>
        </tr>
        <tr>
          <td class="e">SERVER_SOFTWARE </td>
          <td class="v">Apache/2.4.6 (CentOS) PHP/5.4.16 </td>
        </tr>
        <tr>
          <td class="e">SERVER_NAME </td>
          <td class="v">stapp03 </td>
        </tr>
        <tr>
          <td class="e">SERVER_ADDR </td>
          <td class="v">172.16.238.12 </td>
        </tr>
        <tr>
          <td class="e">SERVER_PORT </td>
          <td class="v">80 </td>
        </tr>
        <tr>
          <td class="e">REMOTE_ADDR </td>
          <td class="v">172.16.238.3 </td>
        </tr>
        <tr>
          <td class="e">DOCUMENT_ROOT </td>
          <td class="v">/var/www/html/myroot </td>
        </tr>
        <tr>
          <td class="e">REQUEST_SCHEME </td>
          <td class="v">http </td>
        </tr>
        <tr>
          <td class="e">CONTEXT_PREFIX </td>
          <td class="v"><i>no value</i> </td>
        </tr>
        <tr>
          <td class="e">CONTEXT_DOCUMENT_ROOT </td>
          <td class="v">/var/www/html/myroot </td>
        </tr>
        <tr>
          <td class="e">SERVER_ADMIN </td>
          <td class="v">root@localhost </td>
        </tr>
        <tr>
          <td class="e">SCRIPT_FILENAME </td>
          <td class="v">/var/www/html/myroot/phpinfo.php </td>
        </tr>
        <tr>
          <td class="e">REMOTE_PORT </td>
          <td class="v">54180 </td>
        </tr>
        <tr>
          <td class="e">GATEWAY_INTERFACE </td>
          <td class="v">CGI/1.1 </td>
        </tr>
        <tr>
          <td class="e">SERVER_PROTOCOL </td>
          <td class="v">HTTP/1.1 </td>
        </tr>
        <tr>
          <td class="e">REQUEST_METHOD </td>
          <td class="v">GET </td>
        </tr>
        <tr>
          <td class="e">QUERY_STRING </td>
          <td class="v"><i>no value</i> </td>
        </tr>
        <tr>
          <td class="e">REQUEST_URI </td>
          <td class="v">/phpinfo.php </td>
        </tr>
        <tr>
          <td class="e">SCRIPT_NAME </td>
          <td class="v">/phpinfo.php </td>
        </tr>
      </table>
      <br />
      <h2>HTTP Headers Information</h2>
      <table border="0" cellpadding="3" width="600">
        <tr class="h">
          <th colspan="2">HTTP Request Headers</th>
        </tr>
        <tr>
          <td class="e">HTTP Request </td>
          <td class="v">GET /phpinfo.php HTTP/1.1 </td>
        </tr>
        <tr>
          <td class="e">User-Agent </td>
          <td class="v">curl/7.29.0 </td>
        </tr>
        <tr>
          <td class="e">Host </td>
          <td class="v">stapp03 </td>
        </tr>
        <tr>
          <td class="e">Accept </td>
          <td class="v">*/* </td>
        </tr>
        <tr class="h">
          <th colspan="2">HTTP Response Headers</th>
        </tr>
        <tr>
          <td class="e">X-Powered-By </td>
          <td class="v">PHP/5.4.16 </td>
        </tr>
      </table>
      <br />
      <h2><a name="module_bz2">bz2</a></h2>
      <table border="0" cellpadding="3" width="600">
        <tr>
          <td class="e">BZip2 Support </td>
          <td class="v">Enabled </td>
        </tr>
        <tr>
          <td class="e">Stream Wrapper support </td>
          <td class="v">compress.bzip2:// </td>
        </tr>
        <tr>
          <td class="e">Stream Filter support </td>
          <td class="v">bzip2.decompress, bzip2.compress </td>
        </tr>
        <tr>
          <td class="e">BZip2 Version </td>
          <td class="v">1.0.6, 6-Sept-2010 </td>
        </tr>
      </table>
      <br />
      <h2><a name="module_calendar">calendar</a></h2>
      <table border="0" cellpadding="3" width="600">
        <tr>
          <td class="e">Calendar support </td>
          <td class="v">enabled </td>
        </tr>
      </table>
      <br />
      <h2><a name="module_Core">Core</a></h2>
      <table border="0" cellpadding="3" width="600">
        <tr>
          <td class="e">PHP Version </td>
          <td class="v">5.4.16 </td>
        </tr>
      </table>
      <br />
      <table border="0" cellpadding="3" width="600">
        <tr class="h">
          <th>Directive</th>
          <th>Local Value</th>
          <th>Master Value</th>
        </tr>
        <tr>
          <td class="e">allow_url_fopen</td>
          <td class="v">On</td>
          <td class="v">On</td>
        </tr>
        <tr>
          <td class="e">allow_url_include</td>
          <td class="v">Off</td>
          <td class="v">Off</td>
        </tr>
        <tr>
          <td class="e">always_populate_raw_post_data</td>
          <td class="v">Off</td>
          <td class="v">Off</td>
        </tr>
        <tr>
          <td class="e">arg_separator.input</td>
          <td class="v">&amp;</td>
          <td class="v">&amp;</td>
        </tr>
        <tr>
          <td class="e">arg_separator.output</td>
          <td class="v">&amp;</td>
          <td class="v">&amp;</td>
        </tr>
        <tr>
          <td class="e">asp_tags</td>
          <td class="v">Off</td>
          <td class="v">Off</td>
        </tr>
        <tr>
          <td class="e">auto_append_file</td>
          <td class="v"><i>no value</i></td>
          <td class="v"><i>no value</i></td>
        </tr>
        <tr>
          <td class="e">auto_globals_jit</td>
          <td class="v">On</td>
          <td class="v">On</td>
        </tr>
        <tr>
          <td class="e">auto_prepend_file</td>
          <td class="v"><i>no value</i></td>
          <td class="v"><i>no value</i></td>
        </tr>
        <tr>
          <td class="e">browscap</td>
          <td class="v"><i>no value</i></td>
          <td class="v"><i>no value</i></td>
        </tr>
        <tr>
          <td class="e">default_charset</td>
          <td class="v"><i>no value</i></td>
          <td class="v"><i>no value</i></td>
        </tr>
        <tr>
          <td class="e">default_mimetype</td>
          <td class="v">text/html</td>
          <td class="v">text/html</td>
        </tr>
        <tr>
          <td class="e">disable_classes</td>
          <td class="v"><i>no value</i></td>
          <td class="v"><i>no value</i></td>
        </tr>
        <tr>
          <td class="e">disable_functions</td>
          <td class="v"><i>no value</i></td>
          <td class="v"><i>no value</i></td>
        </tr>
        <tr>
          <td class="e">display_errors</td>
          <td class="v">Off</td>
          <td class="v">Off</td>
        </tr>
        <tr>
          <td class="e">display_startup_errors</td>
          <td class="v">Off</td>
          <td class="v">Off</td>
        </tr>
        <tr>
          <td class="e">doc_root</td>
          <td class="v"><i>no value</i></td>
          <td class="v"><i>no value</i></td>
        </tr>
        <tr>
          <td class="e">docref_ext</td>
          <td class="v"><i>no value</i></td>
          <td class="v"><i>no value</i></td>
        </tr>
        <tr>
          <td class="e">docref_root</td>
          <td class="v"><i>no value</i></td>
          <td class="v"><i>no value</i></td>
        </tr>
        <tr>
          <td class="e">enable_dl</td>
          <td class="v">Off</td>
          <td class="v">Off</td>
        </tr>
        <tr>
          <td class="e">enable_post_data_reading</td>
          <td class="v">On</td>
          <td class="v">On</td>
        </tr>
        <tr>
          <td class="e">error_append_string</td>
          <td class="v"><i>no value</i></td>
          <td class="v"><i>no value</i></td>
        </tr>
        <tr>
          <td class="e">error_log</td>
          <td class="v"><i>no value</i></td>
          <td class="v"><i>no value</i></td>
        </tr>
        <tr>
          <td class="e">error_prepend_string</td>
          <td class="v"><i>no value</i></td>
          <td class="v"><i>no value</i></td>
        </tr>
        <tr>
          <td class="e">error_reporting</td>
          <td class="v">22527</td>
          <td class="v">22527</td>
        </tr>
        <tr>
          <td class="e">exit_on_timeout</td>
          <td class="v">Off</td>
          <td class="v">Off</td>
        </tr>
        <tr>
          <td class="e">expose_php</td>
          <td class="v">On</td>
          <td class="v">On</td>
        </tr>
        <tr>
          <td class="e">extension_dir</td>
          <td class="v">/usr/lib64/php/modules</td>
          <td class="v">/usr/lib64/php/modules</td>
        </tr>
        <tr>
          <td class="e">file_uploads</td>
          <td class="v">On</td>
          <td class="v">On</td>
        </tr>
        <tr>
          <td class="e">highlight.comment</td>
          <td class="v"><font style="color: #FF8000">#FF8000</font></td>
          <td class="v"><font style="color: #FF8000">#FF8000</font></td>
        </tr>
        <tr>
          <td class="e">highlight.default</td>
          <td class="v"><font style="color: #0000BB">#0000BB</font></td>
          <td class="v"><font style="color: #0000BB">#0000BB</font></td>
        </tr>
        <tr>
          <td class="e">highlight.html</td>
          <td class="v"><font style="color: #000000">#000000</font></td>
          <td class="v"><font style="color: #000000">#000000</font></td>
        </tr>
        <tr>
          <td class="e">highlight.keyword</td>
          <td class="v"><font style="color: #007700">#007700</font></td>
          <td class="v"><font style="color: #007700">#007700</font></td>
        </tr>
        <tr>
          <td class="e">highlight.string</td>
          <td class="v"><font style="color: #DD0000">#DD0000</font></td>
          <td class="v"><font style="color: #DD0000">#DD0000</font></td>
        </tr>
        <tr>
          <td class="e">html_errors</td>
          <td class="v">On</td>
          <td class="v">On</td>
        </tr>
        <tr>
          <td class="e">ignore_repeated_errors</td>
          <td class="v">Off</td>
          <td class="v">Off</td>
        </tr>
        <tr>
          <td class="e">ignore_repeated_source</td>
          <td class="v">Off</td>
          <td class="v">Off</td>
        </tr>
        <tr>
          <td class="e">ignore_user_abort</td>
          <td class="v">Off</td>
          <td class="v">Off</td>
        </tr>
        <tr>
          <td class="e">implicit_flush</td>
          <td class="v">Off</td>
          <td class="v">Off</td>
        </tr>
        <tr>
          <td class="e">include_path</td>
          <td class="v">.:/usr/share/pear:/usr/share/php</td>
          <td class="v">.:/usr/share/pear:/usr/share/php</td>
        </tr>
        <tr>
          <td class="e">log_errors</td>
          <td class="v">On</td>
          <td class="v">On</td>
        </tr>
        <tr>
          <td class="e">log_errors_max_len</td>
          <td class="v">1024</td>
          <td class="v">1024</td>
        </tr>
        <tr>
          <td class="e">mail.add_x_header</td>
          <td class="v">On</td>
          <td class="v">On</td>
        </tr>
        <tr>
          <td class="e">mail.force_extra_parameters</td>
          <td class="v"><i>no value</i></td>
          <td class="v"><i>no value</i></td>
        </tr>
        <tr>
          <td class="e">mail.log</td>
          <td class="v"><i>no value</i></td>
          <td class="v"><i>no value</i></td>
        </tr>
        <tr>
          <td class="e">max_execution_time</td>
          <td class="v">30</td>
          <td class="v">30</td>
        </tr>
        <tr>
          <td class="e">max_file_uploads</td>
          <td class="v">20</td>
          <td class="v">20</td>
        </tr>
        <tr>
          <td class="e">max_input_nesting_level</td>
          <td class="v">64</td>
          <td class="v">64</td>
        </tr>
        <tr>
          <td class="e">max_input_time</td>
          <td class="v">60</td>
          <td class="v">60</td>
        </tr>
        <tr>
          <td class="e">max_input_vars</td>
          <td class="v">1000</td>
          <td class="v">1000</td>
        </tr>
        <tr>
          <td class="e">memory_limit</td>
          <td class="v">128M</td>
          <td class="v">128M</td>
        </tr>
        <tr>
          <td class="e">open_basedir</td>
          <td class="v"><i>no value</i></td>
          <td class="v"><i>no value</i></td>
        </tr>
        <tr>
          <td class="e">output_buffering</td>
          <td class="v">4096</td>
          <td class="v">4096</td>
        </tr>
        <tr>
          <td class="e">output_handler</td>
          <td class="v"><i>no value</i></td>
          <td class="v"><i>no value</i></td>
        </tr>
        <tr>
          <td class="e">post_max_size</td>
          <td class="v">8M</td>
          <td class="v">8M</td>
        </tr>
        <tr>
          <td class="e">precision</td>
          <td class="v">14</td>
          <td class="v">14</td>
        </tr>
        <tr>
          <td class="e">realpath_cache_size</td>
          <td class="v">16K</td>
          <td class="v">16K</td>
        </tr>
        <tr>
          <td class="e">realpath_cache_ttl</td>
          <td class="v">120</td>
          <td class="v">120</td>
        </tr>
        <tr>
          <td class="e">register_argc_argv</td>
          <td class="v">Off</td>
          <td class="v">Off</td>
        </tr>
        <tr>
          <td class="e">report_memleaks</td>
          <td class="v">On</td>
          <td class="v">On</td>
        </tr>
        <tr>
          <td class="e">report_zend_debug</td>
          <td class="v">On</td>
          <td class="v">On</td>
        </tr>
        <tr>
          <td class="e">request_order</td>
          <td class="v">GP</td>
          <td class="v">GP</td>
        </tr>
        <tr>
          <td class="e">sendmail_from</td>
          <td class="v"><i>no value</i></td>
          <td class="v"><i>no value</i></td>
        </tr>
        <tr>
          <td class="e">sendmail_path</td>
          <td class="v">/usr/sbin/sendmail&nbsp;-t&nbsp;-i</td>
          <td class="v">/usr/sbin/sendmail&nbsp;-t&nbsp;-i</td>
        </tr>
        <tr>
          <td class="e">serialize_precision</td>
          <td class="v">17</td>
          <td class="v">17</td>
        </tr>
        <tr>
          <td class="e">short_open_tag</td>
          <td class="v">Off</td>
          <td class="v">Off</td>
        </tr>
        <tr>
          <td class="e">SMTP</td>
          <td class="v">localhost</td>
          <td class="v">localhost</td>
        </tr>
        <tr>
          <td class="e">smtp_port</td>
          <td class="v">25</td>
          <td class="v">25</td>
        </tr>
        <tr>
          <td class="e">sql.safe_mode</td>
          <td class="v">Off</td>
          <td class="v">Off</td>
        </tr>
        <tr>
          <td class="e">track_errors</td>
          <td class="v">Off</td>
          <td class="v">Off</td>
        </tr>
        <tr>
          <td class="e">unserialize_callback_func</td>
          <td class="v"><i>no value</i></td>
          <td class="v"><i>no value</i></td>
        </tr>
        <tr>
          <td class="e">upload_max_filesize</td>
          <td class="v">2M</td>
          <td class="v">2M</td>
        </tr>
        <tr>
          <td class="e">upload_tmp_dir</td>
          <td class="v"><i>no value</i></td>
          <td class="v"><i>no value</i></td>
        </tr>
        <tr>
          <td class="e">user_dir</td>
          <td class="v"><i>no value</i></td>
          <td class="v"><i>no value</i></td>
        </tr>
        <tr>
          <td class="e">user_ini.cache_ttl</td>
          <td class="v">300</td>
          <td class="v">300</td>
        </tr>
        <tr>
          <td class="e">user_ini.filename</td>
          <td class="v">.user.ini</td>
          <td class="v">.user.ini</td>
        </tr>
        <tr>
          <td class="e">variables_order</td>
          <td class="v">GPCS</td>
          <td class="v">GPCS</td>
        </tr>
        <tr>
          <td class="e">xmlrpc_error_number</td>
          <td class="v">0</td>
          <td class="v">0</td>
        </tr>
        <tr>
          <td class="e">xmlrpc_errors</td>
          <td class="v">Off</td>
          <td class="v">Off</td>
        </tr>
        <tr>
          <td class="e">zend.detect_unicode</td>
          <td class="v">On</td>
          <td class="v">On</td>
        </tr>
        <tr>
          <td class="e">zend.enable_gc</td>
          <td class="v">On</td>
          <td class="v">On</td>
        </tr>
        <tr>
          <td class="e">zend.multibyte</td>
          <td class="v">Off</td>
          <td class="v">Off</td>
        </tr>
        <tr>
          <td class="e">zend.script_encoding</td>
          <td class="v"><i>no value</i></td>
          <td class="v"><i>no value</i></td>
        </tr>
      </table>
      <br />
      <h2><a name="module_ctype">ctype</a></h2>
      <table border="0" cellpadding="3" width="600">
        <tr>
          <td class="e">ctype functions </td>
          <td class="v">enabled </td>
        </tr>
      </table>
      <br />
      <h2><a name="module_curl">curl</a></h2>
      <table border="0" cellpadding="3" width="600">
        <tr>
          <td class="e">cURL support </td>
          <td class="v">enabled </td>
        </tr>
        <tr>
          <td class="e">cURL Information </td>
          <td class="v">7.29.0 </td>
        </tr>
        <tr>
          <td class="e">Age </td>
          <td class="v">3 </td>
        </tr>
        <tr>
          <td class="e">Features </td>
        </tr>
        <tr>
          <td class="e">AsynchDNS </td>
          <td class="v">Yes </td>
        </tr>
        <tr>
          <td class="e">Debug </td>
          <td class="v">No </td>
        </tr>
        <tr>
          <td class="e">GSS-Negotiate </td>
          <td class="v">Yes </td>
        </tr>
        <tr>
          <td class="e">IDN </td>
          <td class="v">Yes </td>
        </tr>
        <tr>
          <td class="e">IPv6 </td>
          <td class="v">Yes </td>
        </tr>
        <tr>
          <td class="e">Largefile </td>
          <td class="v">Yes </td>
        </tr>
        <tr>
          <td class="e">NTLM </td>
          <td class="v">Yes </td>
        </tr>
        <tr>
          <td class="e">SPNEGO </td>
          <td class="v">No </td>
        </tr>
        <tr>
          <td class="e">SSL </td>
          <td class="v">Yes </td>
        </tr>
        <tr>
          <td class="e">SSPI </td>
          <td class="v">No </td>
        </tr>
        <tr>
          <td class="e">krb4 </td>
          <td class="v">No </td>
        </tr>
        <tr>
          <td class="e">libz </td>
          <td class="v">Yes </td>
        </tr>
        <tr>
          <td class="e">CharConv </td>
          <td class="v">No </td>
        </tr>
        <tr>
          <td class="e">Protocols </td>
          <td class="v">dict, file, ftp, ftps, gopher, http, https, imap, imaps, ldap, ldaps, pop3, pop3s, rtsp, scp, sftp, smtp, smtps, telnet, tftp </td>
        </tr>
        <tr>
          <td class="e">Host </td>
          <td class="v">x86_64-redhat-linux-gnu </td>
        </tr>
        <tr>
          <td class="e">SSL Version </td>
          <td class="v">NSS/3.36 </td>
        </tr>
        <tr>
          <td class="e">ZLib Version </td>
          <td class="v">1.2.7 </td>
        </tr>
        <tr>
          <td class="e">libSSH Version </td>
          <td class="v">libssh2/1.4.3 </td>
        </tr>
      </table>
      <br />
      <h2><a name="module_date">date</a></h2>
      <table border="0" cellpadding="3" width="600">
        <tr>
          <td class="e">date/time support </td>
          <td class="v">enabled </td>
        </tr>
        <tr>
          <td class="e">&quot;Olson&quot; Timezone Database Version </td>
          <td class="v">0.system </td>
        </tr>
        <tr>
          <td class="e">Timezone Database </td>
          <td class="v">internal </td>
        </tr>
        <tr>
          <td class="e">Default timezone </td>
          <td class="v">UTC </td>
        </tr>
      </table>
      <br />
      <table border="0" cellpadding="3" width="600">
        <tr class="h">
          <th>Directive</th>
          <th>Local Value</th>
          <th>Master Value</th>
        </tr>
        <tr>
          <td class="e">date.default_latitude</td>
          <td class="v">31.7667</td>
          <td class="v">31.7667</td>
        </tr>
        <tr>
          <td class="e">date.default_longitude</td>
          <td class="v">35.2333</td>
          <td class="v">35.2333</td>
        </tr>
        <tr>
          <td class="e">date.sunrise_zenith</td>
          <td class="v">90.583333</td>
          <td class="v">90.583333</td>
        </tr>
        <tr>
          <td class="e">date.sunset_zenith</td>
          <td class="v">90.583333</td>
          <td class="v">90.583333</td>
        </tr>
        <tr>
          <td class="e">date.timezone</td>
          <td class="v"><i>no value</i></td>
          <td class="v"><i>no value</i></td>
        </tr>
      </table>
      <br />
      <h2><a name="module_ereg">ereg</a></h2>
      <table border="0" cellpadding="3" width="600">
        <tr>
          <td class="e">Regex Library </td>
          <td class="v">Bundled library enabled </td>
        </tr>
      </table>
      <br />
      <h2><a name="module_exif">exif</a></h2>
      <table border="0" cellpadding="3" width="600">
        <tr>
          <td class="e">EXIF Support </td>
          <td class="v">enabled </td>
        </tr>
        <tr>
          <td class="e">EXIF Version </td>
          <td class="v">1.4 $Id$ </td>
        </tr>
        <tr>
          <td class="e">Supported EXIF Version </td>
          <td class="v">0220 </td>
        </tr>
        <tr>
          <td class="e">Supported filetypes </td>
          <td class="v">JPEG,TIFF </td>
        </tr>
      </table>
      <br />
      <table border="0" cellpadding="3" width="600">
        <tr class="h">
          <th>Directive</th>
          <th>Local Value</th>
          <th>Master Value</th>
        </tr>
        <tr>
          <td class="e">exif.decode_jis_intel</td>
          <td class="v">JIS</td>
          <td class="v">JIS</td>
        </tr>
        <tr>
          <td class="e">exif.decode_jis_motorola</td>
          <td class="v">JIS</td>
          <td class="v">JIS</td>
        </tr>
        <tr>
          <td class="e">exif.decode_unicode_intel</td>
          <td class="v">UCS-2LE</td>
          <td class="v">UCS-2LE</td>
        </tr>
        <tr>
          <td class="e">exif.decode_unicode_motorola</td>
          <td class="v">UCS-2BE</td>
          <td class="v">UCS-2BE</td>
        </tr>
        <tr>
          <td class="e">exif.encode_jis</td>
          <td class="v"><i>no value</i></td>
          <td class="v"><i>no value</i></td>
        </tr>
        <tr>
          <td class="e">exif.encode_unicode</td>
          <td class="v">ISO-8859-15</td>
          <td class="v">ISO-8859-15</td>
        </tr>
      </table>
      <br />
      <h2><a name="module_fileinfo">fileinfo</a></h2>
      <table border="0" cellpadding="3" width="600">
        <tr>
          <td class="e">fileinfo support </td>
          <td class="v">enabled </td>
        </tr>
        <tr>
          <td class="e">version </td>
          <td class="v">1.0.5 </td>
        </tr>
      </table>
      <br />
      <h2><a name="module_filter">filter</a></h2>
      <table border="0" cellpadding="3" width="600">
        <tr>
          <td class="e">Input Validation and Filtering </td>
          <td class="v">enabled </td>
        </tr>
        <tr>
          <td class="e">Revision </td>
          <td class="v">$Id: 2aa8dd57d9c0c655cd45e6e5872bb95fa5ad76cf $ </td>
        </tr>
      </table>
      <br />
      <table border="0" cellpadding="3" width="600">
        <tr class="h">
          <th>Directive</th>
          <th>Local Value</th>
          <th>Master Value</th>
        </tr>
        <tr>
          <td class="e">filter.default</td>
          <td class="v">unsafe_raw</td>
          <td class="v">unsafe_raw</td>
        </tr>
        <tr>
          <td class="e">filter.default_flags</td>
          <td class="v"><i>no value</i></td>
          <td class="v"><i>no value</i></td>
        </tr>
      </table>
      <br />
      <h2><a name="module_ftp">ftp</a></h2>
      <table border="0" cellpadding="3" width="600">
        <tr>
          <td class="e">FTP support </td>
          <td class="v">enabled </td>
        </tr>
      </table>
      <br />
      <h2><a name="module_gettext">gettext</a></h2>
      <table border="0" cellpadding="3" width="600">
        <tr>
          <td class="e">GetText Support </td>
          <td class="v">enabled </td>
        </tr>
      </table>
      <br />
      <h2><a name="module_gmp">gmp</a></h2>
      <table border="0" cellpadding="3" width="600">
        <tr>
          <td class="e">gmp support </td>
          <td class="v">enabled </td>
        </tr>
        <tr>
          <td class="e">GMP version </td>
          <td class="v">6.0.0 </td>
        </tr>
      </table>
      <br />
      <h2><a name="module_hash">hash</a></h2>
      <table border="0" cellpadding="3" width="600">
        <tr>
          <td class="e">hash support </td>
          <td class="v">enabled </td>
        </tr>
        <tr>
          <td class="e">Hashing Engines </td>
          <td class="v">md2 md4 md5 sha1 sha224 sha256 sha384 sha512 ripemd128 ripemd160 ripemd256 ripemd320 whirlpool tiger128,3 tiger160,3 tiger192,3 tiger128,4 tiger160,4 tiger192,4 snefru snefru256 gost adler32 crc32 crc32b fnv132 fnv164 joaat haval128,3 haval160,3 haval192,3 haval224,3 haval256,3 haval128,4 haval160,4 haval192,4 haval224,4 haval256,4 haval128,5 haval160,5 haval192,5 haval224,5 haval256,5  </td>
        </tr>
      </table>
      <br />
      <h2><a name="module_iconv">iconv</a></h2>
      <table border="0" cellpadding="3" width="600">
        <tr>
          <td class="e">iconv support </td>
          <td class="v">enabled </td>
        </tr>
        <tr>
          <td class="e">iconv implementation </td>
          <td class="v">glibc </td>
        </tr>
        <tr>
          <td class="e">iconv library version </td>
          <td class="v">2.17 </td>
        </tr>
      </table>
      <br />
      <table border="0" cellpadding="3" width="600">
        <tr class="h">
          <th>Directive</th>
          <th>Local Value</th>
          <th>Master Value</th>
        </tr>
        <tr>
          <td class="e">iconv.input_encoding</td>
          <td class="v">ISO-8859-1</td>
          <td class="v">ISO-8859-1</td>
        </tr>
        <tr>
          <td class="e">iconv.internal_encoding</td>
          <td class="v">ISO-8859-1</td>
          <td class="v">ISO-8859-1</td>
        </tr>
        <tr>
          <td class="e">iconv.output_encoding</td>
          <td class="v">ISO-8859-1</td>
          <td class="v">ISO-8859-1</td>
        </tr>
      </table>
      <br />
      <h2><a name="module_json">json</a></h2>
      <table border="0" cellpadding="3" width="600">
        <tr>
          <td class="e">json support </td>
          <td class="v">enabled </td>
        </tr>
        <tr>
          <td class="e">json version </td>
          <td class="v">1.2.1 </td>
        </tr>
      </table>
      <br />
      <h2><a name="module_libxml">libxml</a></h2>
      <table border="0" cellpadding="3" width="600">
        <tr>
          <td class="e">libXML support </td>
          <td class="v">active </td>
        </tr>
        <tr>
          <td class="e">libXML Compiled Version </td>
          <td class="v">2.9.1 </td>
        </tr>
        <tr>
          <td class="e">libXML Loaded Version </td>
          <td class="v">20901 </td>
        </tr>
        <tr>
          <td class="e">libXML streams </td>
          <td class="v">enabled </td>
        </tr>
      </table>
      <br />
      <h2><a name="module_mhash">mhash</a></h2>
      <table border="0" cellpadding="3" width="600">
        <tr>
          <td class="e">MHASH support </td>
          <td class="v">Enabled </td>
        </tr>
        <tr>
          <td class="e">MHASH API Version </td>
          <td class="v">Emulated Support </td>
        </tr>
      </table>
      <br />
      <h2><a name="module_openssl">openssl</a></h2>
      <table border="0" cellpadding="3" width="600">
        <tr>
          <td class="e">OpenSSL support </td>
          <td class="v">enabled </td>
        </tr>
        <tr>
          <td class="e">OpenSSL Library Version </td>
          <td class="v">OpenSSL 1.0.2k-fips  26 Jan 2017 </td>
        </tr>
        <tr>
          <td class="e">OpenSSL Header Version </td>
          <td class="v">OpenSSL 1.0.2k-fips  26 Jan 2017 </td>
        </tr>
      </table>
      <br />
      <h2><a name="module_pcre">pcre</a></h2>
      <table border="0" cellpadding="3" width="600">
        <tr>
          <td class="e">PCRE (Perl Compatible Regular Expressions) Support </td>
          <td class="v">enabled </td>
        </tr>
        <tr>
          <td class="e">PCRE Library Version </td>
          <td class="v">8.32 2012-11-30 </td>
        </tr>
      </table>
      <br />
      <table border="0" cellpadding="3" width="600">
        <tr class="h">
          <th>Directive</th>
          <th>Local Value</th>
          <th>Master Value</th>
        </tr>
        <tr>
          <td class="e">pcre.backtrack_limit</td>
          <td class="v">1000000</td>
          <td class="v">1000000</td>
        </tr>
        <tr>
          <td class="e">pcre.recursion_limit</td>
          <td class="v">100000</td>
          <td class="v">100000</td>
        </tr>
      </table>
      <br />
      <h2><a name="module_Phar">Phar</a></h2>
      <table border="0" cellpadding="3" width="600">
        <tr class="h">
          <th>Phar: PHP Archive support</th>
          <th>enabled</th>
        </tr>
        <tr>
          <td class="e">Phar EXT version </td>
          <td class="v">2.0.1 </td>
        </tr>
        <tr>
          <td class="e">Phar API version </td>
          <td class="v">1.1.1 </td>
        </tr>
        <tr>
          <td class="e">SVN revision </td>
          <td class="v">$Id: c5042cc34acebcc0926625b57dff03deebbe6472 $ </td>
        </tr>
        <tr>
          <td class="e">Phar-based phar archives </td>
          <td class="v">enabled </td>
        </tr>
        <tr>
          <td class="e">Tar-based phar archives </td>
          <td class="v">enabled </td>
        </tr>
        <tr>
          <td class="e">ZIP-based phar archives </td>
          <td class="v">enabled </td>
        </tr>
        <tr>
          <td class="e">gzip compression </td>
          <td class="v">enabled </td>
        </tr>
        <tr>
          <td class="e">bzip2 compression </td>
          <td class="v">enabled </td>
        </tr>
        <tr>
          <td class="e">Native OpenSSL support </td>
          <td class="v">enabled </td>
        </tr>
      </table>
      <br />
      <table border="0" cellpadding="3" width="600">
        <tr class="v">
          <td>
            Phar based on pear/PHP_Archive, original concept by Davey Shafik.<br />Phar fully realized by Gregory Beaver and Marcus Boerger.<br />Portions of tar implementation Copyright (c) 2003-2009 Tim Kientzle.
          </td>
        </tr>
      </table>
      <br />
      <table border="0" cellpadding="3" width="600">
        <tr class="h">
          <th>Directive</th>
          <th>Local Value</th>
          <th>Master Value</th>
        </tr>
        <tr>
          <td class="e">phar.cache_list</td>
          <td class="v"><i>no value</i></td>
          <td class="v"><i>no value</i></td>
        </tr>
        <tr>
          <td class="e">phar.readonly</td>
          <td class="v">On</td>
          <td class="v">On</td>
        </tr>
        <tr>
          <td class="e">phar.require_hash</td>
          <td class="v">On</td>
          <td class="v">On</td>
        </tr>
      </table>
      <br />
      <h2><a name="module_Reflection">Reflection</a></h2>
      <table border="0" cellpadding="3" width="600">
        <tr class="h">
          <th>Reflection</th>
          <th>enabled</th>
        </tr>
        <tr>
          <td class="e">Version </td>
          <td class="v">$Id: 6c4d8062369898a397e4b128348042f5c01b4427 $ </td>
        </tr>
      </table>
      <br />
      <h2><a name="module_session">session</a></h2>
      <table border="0" cellpadding="3" width="600">
        <tr>
          <td class="e">Session Support </td>
          <td class="v">enabled </td>
        </tr>
        <tr>
          <td class="e">Registered save handlers </td>
          <td class="v">files user  </td>
        </tr>
        <tr>
          <td class="e">Registered serializer handlers </td>
          <td class="v">php php_binary  </td>
        </tr>
      </table>
      <br />
      <table border="0" cellpadding="3" width="600">
        <tr class="h">
          <th>Directive</th>
          <th>Local Value</th>
          <th>Master Value</th>
        </tr>
        <tr>
          <td class="e">session.auto_start</td>
          <td class="v">Off</td>
          <td class="v">Off</td>
        </tr>
        <tr>
          <td class="e">session.cache_expire</td>
          <td class="v">180</td>
          <td class="v">180</td>
        </tr>
        <tr>
          <td class="e">session.cache_limiter</td>
          <td class="v">nocache</td>
          <td class="v">nocache</td>
        </tr>
        <tr>
          <td class="e">session.cookie_domain</td>
          <td class="v"><i>no value</i></td>
          <td class="v"><i>no value</i></td>
        </tr>
        <tr>
          <td class="e">session.cookie_httponly</td>
          <td class="v">Off</td>
          <td class="v">Off</td>
        </tr>
        <tr>
          <td class="e">session.cookie_lifetime</td>
          <td class="v">0</td>
          <td class="v">0</td>
        </tr>
        <tr>
          <td class="e">session.cookie_path</td>
          <td class="v">/</td>
          <td class="v">/</td>
        </tr>
        <tr>
          <td class="e">session.cookie_secure</td>
          <td class="v">Off</td>
          <td class="v">Off</td>
        </tr>
        <tr>
          <td class="e">session.entropy_file</td>
          <td class="v">/dev/urandom</td>
          <td class="v">/dev/urandom</td>
        </tr>
        <tr>
          <td class="e">session.entropy_length</td>
          <td class="v">32</td>
          <td class="v">32</td>
        </tr>
        <tr>
          <td class="e">session.gc_divisor</td>
          <td class="v">1000</td>
          <td class="v">1000</td>
        </tr>
        <tr>
          <td class="e">session.gc_maxlifetime</td>
          <td class="v">1440</td>
          <td class="v">1440</td>
        </tr>
        <tr>
          <td class="e">session.gc_probability</td>
          <td class="v">1</td>
          <td class="v">1</td>
        </tr>
        <tr>
          <td class="e">session.hash_bits_per_character</td>
          <td class="v">5</td>
          <td class="v">5</td>
        </tr>
        <tr>
          <td class="e">session.hash_function</td>
          <td class="v">0</td>
          <td class="v">0</td>
        </tr>
        <tr>
          <td class="e">session.name</td>
          <td class="v">PHPSESSID</td>
          <td class="v">PHPSESSID</td>
        </tr>
        <tr>
          <td class="e">session.referer_check</td>
          <td class="v"><i>no value</i></td>
          <td class="v"><i>no value</i></td>
        </tr>
        <tr>
          <td class="e">session.save_handler</td>
          <td class="v">files</td>
          <td class="v">files</td>
        </tr>
        <tr>
          <td class="e">session.save_path</td>
          <td class="v">/var/lib/php/session</td>
          <td class="v"><i>no value</i></td>
        </tr>
        <tr>
          <td class="e">session.serialize_handler</td>
          <td class="v">php</td>
          <td class="v">php</td>
        </tr>
        <tr>
          <td class="e">session.upload_progress.cleanup</td>
          <td class="v">On</td>
          <td class="v">On</td>
        </tr>
        <tr>
          <td class="e">session.upload_progress.enabled</td>
          <td class="v">On</td>
          <td class="v">On</td>
        </tr>
        <tr>
          <td class="e">session.upload_progress.freq</td>
          <td class="v">1%</td>
          <td class="v">1%</td>
        </tr>
        <tr>
          <td class="e">session.upload_progress.min_freq</td>
          <td class="v">1</td>
          <td class="v">1</td>
        </tr>
        <tr>
          <td class="e">session.upload_progress.name</td>
          <td class="v">PHP_SESSION_UPLOAD_PROGRESS</td>
          <td class="v">PHP_SESSION_UPLOAD_PROGRESS</td>
        </tr>
        <tr>
          <td class="e">session.upload_progress.prefix</td>
          <td class="v">upload_progress_</td>
          <td class="v">upload_progress_</td>
        </tr>
        <tr>
          <td class="e">session.use_cookies</td>
          <td class="v">On</td>
          <td class="v">On</td>
        </tr>
        <tr>
          <td class="e">session.use_only_cookies</td>
          <td class="v">On</td>
          <td class="v">On</td>
        </tr>
        <tr>
          <td class="e">session.use_trans_sid</td>
          <td class="v">0</td>
          <td class="v">0</td>
        </tr>
      </table>
      <br />
      <h2><a name="module_shmop">shmop</a></h2>
      <table border="0" cellpadding="3" width="600">
        <tr>
          <td class="e">shmop support </td>
          <td class="v">enabled </td>
        </tr>
      </table>
      <br />
      <h2><a name="module_SimpleXML">SimpleXML</a></h2>
      <table border="0" cellpadding="3" width="600">
        <tr class="h">
          <th>Simplexml support</th>
          <th>enabled</th>
        </tr>
        <tr>
          <td class="e">Revision </td>
          <td class="v">$Id: 692516840b2d7d6e7aedb0bedded1f53b764a99f $ </td>
        </tr>
        <tr>
          <td class="e">Schema support </td>
          <td class="v">enabled </td>
        </tr>
      </table>
      <br />
      <h2><a name="module_sockets">sockets</a></h2>
      <table border="0" cellpadding="3" width="600">
        <tr>
          <td class="e">Sockets Support </td>
          <td class="v">enabled </td>
        </tr>
      </table>
      <br />
      <h2><a name="module_SPL">SPL</a></h2>
      <table border="0" cellpadding="3" width="600">
        <tr class="h">
          <th>SPL support</th>
          <th>enabled</th>
        </tr>
        <tr>
          <td class="e">Interfaces </td>
          <td class="v">Countable, OuterIterator, RecursiveIterator, SeekableIterator, SplObserver, SplSubject </td>
        </tr>
        <tr>
          <td class="e">Classes </td>
          <td class="v">AppendIterator, ArrayIterator, ArrayObject, BadFunctionCallException, BadMethodCallException, CachingIterator, CallbackFilterIterator, DirectoryIterator, DomainException, EmptyIterator, FilesystemIterator, FilterIterator, GlobIterator, InfiniteIterator, InvalidArgumentException, IteratorIterator, LengthException, LimitIterator, LogicException, MultipleIterator, NoRewindIterator, OutOfBoundsException, OutOfRangeException, OverflowException, ParentIterator, RangeException, RecursiveArrayIterator, RecursiveCachingIterator, RecursiveCallbackFilterIterator, RecursiveDirectoryIterator, RecursiveFilterIterator, RecursiveIteratorIterator, RecursiveRegexIterator, RecursiveTreeIterator, RegexIterator, RuntimeException, SplDoublyLinkedList, SplFileInfo, SplFileObject, SplFixedArray, SplHeap, SplMinHeap, SplMaxHeap, SplObjectStorage, SplPriorityQueue, SplQueue, SplStack, SplTempFileObject, UnderflowException, UnexpectedValueException </td>
        </tr>
      </table>
      <br />
      <h2><a name="module_standard">standard</a></h2>
      <table border="0" cellpadding="3" width="600">
        <tr>
          <td class="e">Dynamic Library Support </td>
          <td class="v">enabled </td>
        </tr>
        <tr>
          <td class="e">Path to sendmail </td>
          <td class="v">/usr/sbin/sendmail -t -i </td>
        </tr>
      </table>
      <br />
      <table border="0" cellpadding="3" width="600">
        <tr class="h">
          <th>Directive</th>
          <th>Local Value</th>
          <th>Master Value</th>
        </tr>
        <tr>
          <td class="e">assert.active</td>
          <td class="v">1</td>
          <td class="v">1</td>
        </tr>
        <tr>
          <td class="e">assert.bail</td>
          <td class="v">0</td>
          <td class="v">0</td>
        </tr>
        <tr>
          <td class="e">assert.callback</td>
          <td class="v"><i>no value</i></td>
          <td class="v"><i>no value</i></td>
        </tr>
        <tr>
          <td class="e">assert.quiet_eval</td>
          <td class="v">0</td>
          <td class="v">0</td>
        </tr>
        <tr>
          <td class="e">assert.warning</td>
          <td class="v">1</td>
          <td class="v">1</td>
        </tr>
        <tr>
          <td class="e">auto_detect_line_endings</td>
          <td class="v">0</td>
          <td class="v">0</td>
        </tr>
        <tr>
          <td class="e">default_socket_timeout</td>
          <td class="v">60</td>
          <td class="v">60</td>
        </tr>
        <tr>
          <td class="e">from</td>
          <td class="v"><i>no value</i></td>
          <td class="v"><i>no value</i></td>
        </tr>
        <tr>
          <td class="e">url_rewriter.tags</td>
          <td class="v">a=href,area=href,frame=src,input=src,form=fakeentry</td>
          <td class="v">a=href,area=href,frame=src,input=src,form=fakeentry</td>
        </tr>
        <tr>
          <td class="e">user_agent</td>
          <td class="v"><i>no value</i></td>
          <td class="v"><i>no value</i></td>
        </tr>
      </table>
      <br />
      <h2><a name="module_tokenizer">tokenizer</a></h2>
      <table border="0" cellpadding="3" width="600">
        <tr>
          <td class="e">Tokenizer Support </td>
          <td class="v">enabled </td>
        </tr>
      </table>
      <br />
      <h2><a name="module_xml">xml</a></h2>
      <table border="0" cellpadding="3" width="600">
        <tr>
          <td class="e">XML Support </td>
          <td class="v">active </td>
        </tr>
        <tr>
          <td class="e">XML Namespace Support </td>
          <td class="v">active </td>
        </tr>
        <tr>
          <td class="e">libxml2 Version </td>
          <td class="v">2.9.1 </td>
        </tr>
      </table>
      <br />
      <h2><a name="module_zip">zip</a></h2>
      <table border="0" cellpadding="3" width="600">
        <tr>
          <td class="e">Zip </td>
          <td class="v">enabled </td>
        </tr>
        <tr>
          <td class="e">Extension Version </td>
          <td class="v">$Id: 0c033d4e4613d577409950ed7bf8da4b68286d15 $ </td>
        </tr>
        <tr>
          <td class="e">Zip version </td>
          <td class="v">1.11.0 </td>
        </tr>
        <tr>
          <td class="e">Compiled against libzip version </td>
          <td class="v">0.10.1 </td>
        </tr>
      </table>
      <br />
      <h2><a name="module_zlib">zlib</a></h2>
      <table border="0" cellpadding="3" width="600">
        <tr class="h">
          <th>ZLib Support</th>
          <th>enabled</th>
        </tr>
        <tr>
          <td class="e">Stream Wrapper </td>
          <td class="v">compress.zlib:// </td>
        </tr>
        <tr>
          <td class="e">Stream Filter </td>
          <td class="v">zlib.inflate, zlib.deflate </td>
        </tr>
        <tr>
          <td class="e">Compiled Version </td>
          <td class="v">1.2.7 </td>
        </tr>
        <tr>
          <td class="e">Linked Version </td>
          <td class="v">1.2.7 </td>
        </tr>
      </table>
      <br />
      <table border="0" cellpadding="3" width="600">
        <tr class="h">
          <th>Directive</th>
          <th>Local Value</th>
          <th>Master Value</th>
        </tr>
        <tr>
          <td class="e">zlib.output_compression</td>
          <td class="v">Off</td>
          <td class="v">Off</td>
        </tr>
        <tr>
          <td class="e">zlib.output_compression_level</td>
          <td class="v">-1</td>
          <td class="v">-1</td>
        </tr>
        <tr>
          <td class="e">zlib.output_handler</td>
          <td class="v"><i>no value</i></td>
          <td class="v"><i>no value</i></td>
        </tr>
      </table>
      <br />
      <h2>Additional Modules</h2>
      <table border="0" cellpadding="3" width="600">
        <tr class="h">
          <th>Module Name</th>
        </tr>
      </table>
      <br />
      <h2>Environment</h2>
      <table border="0" cellpadding="3" width="600">
        <tr class="h">
          <th>Variable</th>
          <th>Value</th>
        </tr>
        <tr>
          <td class="e">LANG </td>
          <td class="v">C </td>
        </tr>
        <tr>
          <td class="e">PATH </td>
          <td class="v">/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin </td>
        </tr>
        <tr>
          <td class="e">NOTIFY_SOCKET </td>
          <td class="v">/run/systemd/notify </td>
        </tr>
      </table>
      <br />
      <h2>PHP Variables</h2>
      <table border="0" cellpadding="3" width="600">
        <tr class="h">
          <th>Variable</th>
          <th>Value</th>
        </tr>
        <tr>
          <td class="e">_SERVER["UNIQUE_ID"]</td>
          <td class="v">ZDoz4MnCVxnBVG64ISuzGgAAAAE</td>
        </tr>
        <tr>
          <td class="e">_SERVER["HTTP_USER_AGENT"]</td>
          <td class="v">curl/7.29.0</td>
        </tr>
        <tr>
          <td class="e">_SERVER["HTTP_HOST"]</td>
          <td class="v">stapp03</td>
        </tr>
        <tr>
          <td class="e">_SERVER["HTTP_ACCEPT"]</td>
          <td class="v">*/*</td>
        </tr>
        <tr>
          <td class="e">_SERVER["PATH"]</td>
          <td class="v">/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin</td>
        </tr>
        <tr>
          <td class="e">_SERVER["SERVER_SIGNATURE"]</td>
          <td class="v"><i>no value</i></td>
        </tr>
        <tr>
          <td class="e">_SERVER["SERVER_SOFTWARE"]</td>
          <td class="v">Apache/2.4.6 (CentOS) PHP/5.4.16</td>
        </tr>
        <tr>
          <td class="e">_SERVER["SERVER_NAME"]</td>
          <td class="v">stapp03</td>
        </tr>
        <tr>
          <td class="e">_SERVER["SERVER_ADDR"]</td>
          <td class="v">172.16.238.12</td>
        </tr>
        <tr>
          <td class="e">_SERVER["SERVER_PORT"]</td>
          <td class="v">80</td>
        </tr>
        <tr>
          <td class="e">_SERVER["REMOTE_ADDR"]</td>
          <td class="v">172.16.238.3</td>
        </tr>
        <tr>
          <td class="e">_SERVER["DOCUMENT_ROOT"]</td>
          <td class="v">/var/www/html/myroot</td>
        </tr>
        <tr>
          <td class="e">_SERVER["REQUEST_SCHEME"]</td>
          <td class="v">http</td>
        </tr>
        <tr>
          <td class="e">_SERVER["CONTEXT_PREFIX"]</td>
          <td class="v"><i>no value</i></td>
        </tr>
        <tr>
          <td class="e">_SERVER["CONTEXT_DOCUMENT_ROOT"]</td>
          <td class="v">/var/www/html/myroot</td>
        </tr>
        <tr>
          <td class="e">_SERVER["SERVER_ADMIN"]</td>
          <td class="v">root@localhost</td>
        </tr>
        <tr>
          <td class="e">_SERVER["SCRIPT_FILENAME"]</td>
          <td class="v">/var/www/html/myroot/phpinfo.php</td>
        </tr>
        <tr>
          <td class="e">_SERVER["REMOTE_PORT"]</td>
          <td class="v">54180</td>
        </tr>
        <tr>
          <td class="e">_SERVER["GATEWAY_INTERFACE"]</td>
          <td class="v">CGI/1.1</td>
        </tr>
        <tr>
          <td class="e">_SERVER["SERVER_PROTOCOL"]</td>
          <td class="v">HTTP/1.1</td>
        </tr>
        <tr>
          <td class="e">_SERVER["REQUEST_METHOD"]</td>
          <td class="v">GET</td>
        </tr>
        <tr>
          <td class="e">_SERVER["QUERY_STRING"]</td>
          <td class="v"><i>no value</i></td>
        </tr>
        <tr>
          <td class="e">_SERVER["REQUEST_URI"]</td>
          <td class="v">/phpinfo.php</td>
        </tr>
        <tr>
          <td class="e">_SERVER["SCRIPT_NAME"]</td>
          <td class="v">/phpinfo.php</td>
        </tr>
        <tr>
          <td class="e">_SERVER["PHP_SELF"]</td>
          <td class="v">/phpinfo.php</td>
        </tr>
        <tr>
          <td class="e">_SERVER["REQUEST_TIME_FLOAT"]</td>
          <td class="v">1681535968.546</td>
        </tr>
        <tr>
          <td class="e">_SERVER["REQUEST_TIME"]</td>
          <td class="v">1681535968</td>
        </tr>
      </table>
      <br />
      <h2>PHP License</h2>
      <table border="0" cellpadding="3" width="600">
        <tr class="v">
          <td>
            <p>
              This program is free software; you can redistribute it and/or modify it under the terms of the PHP License as published by the PHP Group and included in the distribution in the file:  LICENSE
            </p>
            <p>This program is distributed in the hope that it will be useful, but WITHOUT ANY WARRANTY; without even the implied warranty of MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE.</p>
            <p>If you did not receive a copy of the PHP license, or have any questions about PHP licensing, please contact license@php.net.</p>
          </td>
        </tr>
      </table>
      <br />
    </div>
  </body>
</html>
```
</details>

Looks good, we are done.
