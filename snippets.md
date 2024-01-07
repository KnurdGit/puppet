# Useful Puppet Code Snippets

## File

Create files and parent directories, like `mkdir -p`:
```ruby
file { [
    '/opt/node_exporter',
    '/opt/node_exporter/exporters',
    '/opt/node_exporter/exporters/dist',
  ]:
    ensure => 'directory',
    owner  => 'node_exporter',
    group  => 'node_exporter',
  '/opt/node_exporter/exporters/dist/file':
    ensure => 'file',
    mode   => '0640',
    owner  => 'node_exporter',
    group  => 'node_exporter',
}
```

Create multiple files with same attributes:
```ruby
file {
  default:
    owner => 'root',
    group => 'root',
    mode  => '0444';
  '/etc/ssh/teleport_user_ca.pub':
    source => 'puppet:///modules/ssh/teleport_user_ca.pub';
  '/etc/ssh/ssh_known_hosts':
    source => 'puppet:///modules/ssh/ssh_known_hosts';
}
```

Default section can contain any parameter allowed for `file` resource:
```ruby
file {
  default:
    ensure => 'link';
    owner => 'uwsgi',
    group => 'uwsgi';
    require => Exec['check_uwsgi'];
  '/etc/uwsgi/emperor.ini':
    target => "${project_dir}/emperor.ini";
  '/etc/uwsgi/sites/ams.ini':
    target => "${project_dir}/ams.ini";
  '/etc/systemd/system/uwsgi.service':
    target => "${project_dir}/uwsgi.service";
}
```

Notify multiple services after file changes:
```ruby
file {'/etc/victoriametrics/vminsert.env':
  ensure  => file,
  owner   => 'prometheus',
  group   => 'root',
  content => template('cx_victoriametrics/vminsert.erb'),
  notify  => [
    Service['vminsert@0'],
    Service['vminsert@1'],
    Service['vminsert@2'],
  ],
}
```

## Package

Install multiple packages in one declaration:
```ruby
package {
  default:
    ensure => present;
  'wal-g':
    require => Service['patroni'];
  'consul-template':
    require => Service['patroni'];
  'pgbouncer':
    require => Service['patroni'];
  'python3.8-venv':
    before => Class['::patroni'];
  'prometheus-postgresql-exporter':
    ensure => '0.10.1';
  'pgbouncer-exporter':
    ensure => '0.4.0';
  'pgdoctor':
    ensure => '0.3.0';
}
```

Install or purge packages from an array:
```ruby
$install = [
  'vim',
  'git',
  'curl',
],

$purge   = [
  'nano',
  'pico',
],

package { $install: ensure => 'present' }
package { $purge:   ensure => 'purged'  }
```

## Service

Control multiple services:
```ruby
service {
  default:
    ensure => running,
    enable => true
  'auditd':
    require => Package['auditd'];
  'atop':
    require => Package['atop'];
}
```

Configure provider behavior:
```ruby
service { 'cassandra-exporter':
  ensure    => running,
  name      => 'cassandra-exporter',
  enable    => true,
  hasstatus => true,
  provider  => 'systemd',
  start     => 'systemctl start cassandra-exporter',
  stop      => 'systemctl stop cassandra-exporter',
  status    => 'systemctl status cassandra-exporter',
}

service {"consul-${consul_role}":
  ensure    => 'running',
  enable    => true,
  restart   => 'consul reload',
  subscribe => File['/etc/consul.hcl'],
}
```
