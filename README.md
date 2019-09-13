# ansible-tor

Installs Tor relays on Debian-based systems.

This role uses the default Tor instance (configuration in _/etc/tor/torrc_)
only for hidden services and optionally a SOCKS proxy. Bridge, middle and exit
relays are created as additional instances using the _tor-instance-create_
script (configuration under _/etc/tor/instances_).

## Requirements

* Debian-based system with systemd
* Offline keys as described below

## Role Variables

Please see [defaults/main.yml](defaults/main.yml) for default values.

### Main Variables

<table>
<tr>
  <th>Variable</th>
  <th>Description</th>
</tr>
<tr>
  <td>tor_instances_bridge</td>
  <td>Bridge instance definitions (see below).</td>
</tr>
<tr>
  <td>tor_instances_middle</td>
  <td>Middle instance definitions (see below).</td>
</tr>
<tr>
  <td>tor_instances_exit</td>
  <td>Exit instance definitions (see below).</td>
</tr>
<tr>
  <td>tor_exit_policy</td>
  <td>List of global <tt>ExitPolicy</tt> values (exits only).</td>
</tr>
<tr>
  <td>tor_reduced_exit_policy</td>
  <td>
    Global <tt>ReducedExitPolicy</tt> value, if <tt>tor_exit_policy</tt> is
    undefined (exits only).
  </td>
</tr>
<tr>
  <td>tor_my_family</td>
  <td>List of fingerprints for <tt>MyFamily</tt>.</td>
</tr>
<tr>
  <td>tor_offline_keys</td>
  <td>Local base directory for offline keys (see below).</td>
</tr>
</table>

### Other Variables

<table>
<tr>
  <th>Variable</th>
  <th>Description</th>
</tr>
<tr>
  <td>tor_apparmor</td>
  <td>
    Disable AppArmor, in case Tor is running in an LXC container.
    (<a href="https://trac.torproject.org/projects/tor/ticket/17754">Bug 17754</a>)
  </td>
</tr>
<tr>
  <td>tor_avoid_disk</td>
  <td>Set <tt>AvoidDiskWrites</tt>, for flash-based systems. </td>
</tr>
<tr>
  <td>tor_default_hidden_services</td>
  <td>Hidden service definitions (see below).</td>
</tr>
<tr>
  <td>tor_default_socks_port</td>
  <td>SOCKS port of the default instance.</td>
</tr>
<tr>
  <td>tor_dist</td>
  <td>Distribution name on <a href="https://deb.torproject.org/torproject.org/dists/">deb.torproject.org</a>.</td>
</tr>
<tr>
  <td>tor_exit_notice</td>
  <td>Local HTML file used as <tt>DirPortFrontPage</tt> on exit instances.</td>
</tr>
<tr>
  <td>tor_exit_policy_blocks</td>
  <td>
    <tt>true</tt> to include <i>/etc/tor/exit-policy-blocks</i> before the exit
    policy on exit instances.
  </td>
</tr>
<tr>
  <td>tor_nameservers</td>
  <td>
    List of nameserver addresses to add to <i>/etc/tor/resolv.conf</i>, which
    will be used by bridge, middle and exit instances as
    <tt>ServerDNSResolvConfFile</tt>.
  </td>
</tr>
<tr>
  <td>tor_num_cpus</td>
  <td><tt>NumCPUs</tt> value.</td>
</tr>
<tr>
  <td>tor_packages_extra</td>
  <td>Extra system packages to install.</td>
</tr>
</table>

## Instance Configuration

Bridge, middle and exit relays are created using the _tor-instance-create_
script and configured via three separate variables in the inventory (see above).

The following properties are common to all instance types (required properties
in **bold**):

<table>
<tr>
  <th>Property</th>
  <th>Description</th>
</tr>
<tr>
  <td><b>name</b></td>
  <td>
    Instance name. Example: the instance name <tt>foo</tt> will create the
    systemd unit <tt>tor@foo</tt> owned by user/group <tt>_tor-foo</tt>.
    The configuration will be in <i>/etc/tor/instances/foo/torrc</i>, the data
    in <i>/var/lib/tor-instances/foo</i>.
  </td>
</tr>
<tr>
  <td><b>or_ports</b></td>
  <td>List of <tt>ORPort</tt> values.</td>
</tr>
<tr>
  <td>nickname</td>
  <td>Relay nickname.</td>
</tr>
<tr>
  <td>relay_bandwidth_rate</td>
  <td><tt>RelayBandwidthRate</tt> value.</td>
</tr>
<tr>
  <td>relay_bandwidth_burst</td>
  <td><tt>RelayBandwidthBurst</tt> value.</td>
</tr>
<tr>
  <td>contact_info</td>
  <td><tt>ContactInfo</tt> value.</td>
</tr>
<tr>
  <td>max_mem_in_queues</td>
  <td><tt>MaxMemInQueues</tt> value.</td>
</tr>
</table>

### Bridge Configuration

The following properties are used by bridge instances (required properties in
**bold**):

<table>
<tr>
  <th>Property</th>
  <th>Description</th>
</tr>
<tr>
  <td><b>pluggable_transports</b></td>
  <td>List of pluggable transport definitions (see below).</td>
</tr>
</table>

Example:

    tor_instances_bridge:
      - name: bridge
        nickname: MyCoolBridge
        or_ports: [9000]
        contact_info: "Foo <foo@example.com>"
        pluggable_transports:
          - name: obfs4
            exec: /usr/bin/obfs4proxy
            address: 0.0.0.0:8443

### Middle/Exit Configuration

The following properties are used by middle/exit instances (required properties
in **bold**):

<table>
<tr>
  <th>Property</th>
  <th>Description</th>
</tr>
<tr>
  <td>dir_ports</td>
  <td>
      List of <tt>DirPort</tt> values (only one can be advertised).
  </td>
</tr>
<tr>
  <td>exit_addresses</td>
  <td>
      List of <tt>OutboundBindAddressExit</tt> values (exits only).
  </td>
</tr>
<tr>
  <td>exit_policy</td>
  <td>
    List of <tt>ExitPolicy</tt> values (exits only).<br>
    Overrides global exit variables.
  </td>
</tr>
<tr>
  <td>reduced_exit_policy</td>
  <td>
    <tt>ReducedExitPolicy</tt> value, if <tt>exit_policy</tt> is undefined
    (exits only).<br>
    Overrides global exit variables.
  </td>
</tr>
<tr>
  <td>ipv6_exit</td>
  <td>Allow IPv6 exit traffic (exits only).</td>
</tr>
</table>

Example:

    tor_instances_exit:
      - name: exit
        nickname: MyCoolExit
        contact_info: "Foo <foo@example.com>"
        or_ports:
          - 443
          - "[abcd::1:2:3:4]:443"
        ipv6_exit: 1
        exit_policy:
          - "accept *:80"
          - "accept *:443"
          [...]
          - "reject *:*"

## Offline Keys

A given host's and instance's offline keys are copied from the local directory
`{{ tor_offline_keys }}/{{ inventory_hostname }}/INSTANCE_NAME/keys`.
See `man tor` for information on how to manage offline keys.

Example:

Using the default `tor_offline_keys` value, an inventory hostname of `tor-exit`
and an instance name of `exit`, offline keys would be generated with this
command:

    tor --DataDirectory ~/.tor_offline_keys/tor-exit/exit --keygen

**Please note:** Tor creates the RSA key `secret_id_key` for new relays. This
key is part of the relay identity, so you should create a backup. If the key is
found in the local directory mentioned above, it is also copied to the host.

## Hidden Services

Hidden services are only configured on the default instance via the
`tor_default_hidden_services` variable.

Example:

    tor_default_hidden_services:
      - name: ssh
        authorize_client: basic ssh
        ports: [22]
        version: 3

Result:

    HiddenServiceDir /var/lib/tor/hs_ssh/
    HiddenServiceAuthorizeClient basic ssh
    HiddenServicePort 22
    HiddenServiceVersion 3

The properties `authorize_client` and `version` are optional.

## License

GPLv3
