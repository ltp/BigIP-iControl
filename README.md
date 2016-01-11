## NAME

BigIP::iControl - A Perl interface to the F5 iControl API

## SYNOPSIS

        use BigIP::iControl;

        my $ic = BigIP::iControl->new(
                                server		=> 'bigip.company.com',
                                username	=> 'api_user',
                                password	=> 'my_password',
                                port		=> 443,
                                proto		=> 'https'
                        );

	my $virtual	= ($ic->get_vs_list())[0];

	my %stats	= $ic->get_vs_statistics_stringified($virtual);;

	print '*'x50,"\nVirtual: $virtual\n",'*'x50,"\nTimestamp: $stats{timestamp}\n";

	foreach my $s (sort keys %{$stats{stats}}) {
		print "$s\t$stats{stats}{$s}\n"
	}

## DESCRIPTION

This package provides a Perl interface to the F5 BigIP iControl API.

The F5 BigIP iControl API is an open SOAP/XML for communicating with supported F5 BigIP products.

The primary aim of this package is to provide a simplified interface to an already simple and
intutive API and to allow the user to do more with less code.  By reducing the API invocations
to methods returning simple types, it is hoped that this module will provide a simple alternative
for common tasks.

The secondary aim for this package is to provide a simple interface for accessing statistical
data from the iControl API for monitoring, recording, archival and display in other systems.
This objective has largely been obsoleted in v11 with the introduction of new statistical
monitoring and display features in the web UI.

This package generally provides two methods for each each task; a raw method typically returning
the response as received from iControl, and a "stringified" method returning a parsed response.

In general, the stringified methods will typically fufill most requirements and should usually
be easier to use.

## METHODS

### new (%args)

	my $ic = BigIP::iControl->new(
				server		=> 'bigip.company.com',
				username	=> 'api_user',
				password	=> 'my_password',
				port		=> 443,
				proto		=> 'https',
				verify_hostname	=> 0
			);

Constructor method.  Creates a new BigIP::iControl object representing a single interface into the iControl 
API of the target system.

Required parameters are:

- server

    The target F5 BIGIP device.  The supplied value may be either an IP address, FQDN or resolvable hostname.

- username

    The username with which to connect to the iControl API.

- password

    The password with which to connect to the iControl API.

- port

    The port on which to connect to the iControl API.  If not specified this value will default to 443.

- proto

    The protocol with to use for communications with the iControl API (should be either http or https).  If not specified
    this value will default to https.

- verify\_hostname

    If TRUE when used with a secure connection then the client will ensure that the target server has a valid certificate 
    matching the expected hostname.

    Defaults to false - that is; no certificate validation is performed.

### get\_product\_information

Gets a ProductInformation struct containing the identifying attributes of installed product.
The struct information is described below;

	Member			Type		Description
	----------		----------	----------
	product_code		String		The identifier describing the installed product.
	product_version 	String		The version of the installed product.
	package_version 	String		The package version of the installed product.
	package_edition 	String		The package edition of the installed product.
	product_features 	String [] 	A list of feature names available in the installed product. 
=cut

sub get\_product\_information  {
	return $\_\[0\]->\_request(module => 'System', interface => 'SystemInfo', method => 'get\_product\_information')
}

### get\_system\_information

Return a SystemInformation struct containing the identifying attributes of the operating system.
The struct information is described below;

	Member					Type		Description
	----------				----------	----------
	system_name				String		The name of the operating system implementation.
	host_name				String		The host name of the system.
	os_release				String		The release level of the operating system.
	os_machine				String		The hardware platform CPU type.
	os_version				String		The version string for the release of the operating system.
	platform				String		The platform of the device.
	product_category			String		The product category of the device.
	chassis_serial				String		The chassis serial number.
	switch_board_serial			String		The serial number of the switch board.
	switch_board_part_revision		String		The part revision number of the switch board.
	host_board_serial			String		The serial number of the host motherboard.
	host_board_part_revision		String		The part revision number of the host board.
	annunciator_board_serial		String		The serial number of the annuciator board.
	annunciator_board_part_revision		String		The part revision number of the annunciator board. 

### get\_system\_id ()

Gets the unique identifier for the system. 

### get\_cpu\_metrics ()

Gets the CPU metrics for the CPU(s) on the platform.

### get\_cpu\_metrics\_stringified ()

Gets the CPU metrics for the CPU(s) on the platform.

### get\_cpu\_fan\_speed ($cpu) 

Returns the current CPU fan speed in RPM for the specified CPU.

### get\_cpu\_temp ($cpu) 

Returns the current CPU temperature degrees celcius for the specified CPU.

### get\_cpu\_usage\_extended\_information ()

### get\_cpu\_usage\_extended\_information\_stringified ()

### get\_cluster\_list ()

Gets a list of the cluster names.

### get\_failover\_mode ()

Gets the current fail-over mode that the device is running in. 

### get\_failover\_state ()

Gets the current fail-over state that the device is running in. 

### is\_redundant ()

Returns a boolean indicating the redundancy state of the device.

### get\_cluster\_enabled\_state ()

Gets the cluster enabled states. 

### get\_service\_list () 

Returns a list of all supported services on this host.

### get\_service\_status () 

Returns the status of the specified service.

### get\_all\_service\_statuses () 

Returns the status of all services.

### save\_configuration ($filename)

	$ic->save_configuration('backup.ucs');

	# is equivalent to

	$ic->save_configuration('backup');
	
	# Not specifying a filename will use today's date in the
	# format YYYYMMDD as the filename.

	$ic->save_configuration();

	# is equivalent to

	$ic->save_configuration('today');
	

Saves the current configurations on the target device.  

This method takes a single optional parameter; the filename to which the configuration should be saved.  The file
extension __.ucs__ will be suffixed to the filename if missing from the supplied filename.

Specifying no optional filename parameter or using the filename __today__ will use the current date as the filename
of the saved configuration file in the format __YYYYMMDD__.

### save\_base\_configuration ()

        $ic->save_base_configuration();

Saves only the base configuration (VLANs, self IPs...). The filename specified when used with this mode will 
be ignored, since configuration will be saved to /config/bigip\_base.conf by default. 

### save\_high\_level\_configuration ()

        $ic->save_high_level_configuration();

Saves only the high-level configuration (virtual servers, pools, members, monitors...). The filename specified 
when used with this mode will be ignored, since configuration will be saved to /config/bigip.conf by default. 

### download\_configuration ($filename)

This method downloads a saved UCS configuration from the target device.

### get\_configuration\_list ()

	my %config_list = $ic->get_configuration_list();

Returns a list of the configuration archives present on the system.  the list is returned as a hash
with the name of the configuration archive as the key, and the creation date of the configuration 
archive as the value.

The creation date uses the native date format of:

	Day Mon D HH:MM:SS YYYY

Where __Day__ is the three-letter common abbreviation of the day name, __Mon__ is the three letter common
abbreviation of the month name and __D__ has the value range 1-31 with no leading zeros.

### delete\_configuration ()

	$ic->delete_configuration('file.ucs');

Deletes the specified configuration archive from the system.

### download\_file ( $FILE )

	# Print the bigip.conf file to the terminal
	print $ic->download_file('/config/bigip.conf');

This method provides direct access to files on the target system. The method returns a scalar containing
the contents of the file.

This method may be useful for downloading configuration files for versioning or backups.

### delete\_volume ()

Deletes a volume from the system, or from all blades on a chassis. 

### get\_all\_software\_status ()

A structure that contains information on software status. This includes items like product, version, build, and (live install) completion status.

	Member			Type		Description
	----------		----------	----------
	installation_id 	InstallationID	The location for the status.
	product			String		The product you are installing (ex: BIGIP) (or, product which is installed).
	version			String		The version of product (ex: 9.6.0).
	build 			String		The build number you are installing.
	base_build 		String 		The base build (used for hotfixes).
	active 			boolean 	Whether the boot location is active.
	edition 		String 		Gives the edition, e.g."Hotfix HF4" (used for hotfixes).
	status 			String 		A string indicating the status of the live install process. The status strings are "none", "audited", "retry", "upgrade needed", "waiting for image", "installing nn.mmm pct", "complete", "cancelling", "cancelled", and "failed". The "failed" string may have text giving a reason after it. The "waiting for image" string may have further text after it describing the image being awaited. A client should ignore any strings returned other than these. You can use the status field to monitor the completion status of a live install operation in process. When checking status, you should ensure that the product, version, and build reflect the software whose status you are interested in (because there are a few scenarios where the product, version, and build for a volume may not be updated as quickly as you might expect).

### get\_boot\_location ()

Gets the configured default boot location, which will be the location that boots after the system reboots.

### get\_hotfix\_information ()

Gets information on any hotfixes applied to the system. There may not be any hotfix installed, in which case the returned sequence is empty.

### install\_software\_image\_v2 ()

Initiates an install of a software image on all blades installed on one chassis. 

### get\_interface\_list ()

	my @interfaces = $ic->get_interface_list();

Retuns an ordered list of all interfaces on the target device.

### get\_interface\_enabled\_state ($interface)

Returns the enabled state of the specific interface.

### get\_interface\_media\_status ($interface)

Returns the media status of the specific interface.

### get\_interface\_media\_speed ($interface)

Returns the media speed of the specific interface in Mbps.

### get\_interface\_statistics ($interface)

Returns all statistics for the specified interface as a InterfaceStatistics object.  Unless you specifically
require access to the raw object, consider using __get\_interface\_statistics\_stringified__ for a pre-parsed hash 
in an easy-to-digest format.

### get\_interface\_statistics\_stringified ($interface)

	my $inet	= ($ic->get_interface_list())[0];
	my %stats       = $ic->get_interface_statistics_stringified($inet);

	print "Interface: $inet - Bytes in: $stats{stats}{STATISTIC_BYTES_IN} - Bytes out: STATISTIC_BYTES_OUT";

Returns all statistics for the specified interface as a hash having the following structure;

	{
	timestamp	=> 'YYYY-MM-DD-hh-mm-ss',
	stats		=>	{
				statistic_1	=> value
				...
				statistic_n	=> value
				}
	}

Where the keys of the stats hash are the names of the statistic types defined in a InterfaceStatistics object.
Refer to the official API documentation for the exact structure of the InterfaceStatistics object.

### get\_trunk\_list ()

	my @trunks = $ic->get_trunk_list();

Returns an array of the configured trunks present on the device.

### get\_active\_trunk\_members ()

	print "Trunk $t has " . $ic->get_active_trunk_members() . " active members.\n";

Returns the number of the active members for the specified trunk.

### get\_configured\_trunk\_members ()

	print "Trunk $t has " . $ic->get_configured_trunk_members() . " configured members.\n";

Returns the number of configured members for the specified trunk.

### get\_trunk\_interfaces ()

	my @t_inets = $ic->get_trunk_interfaces();

Returns an array containing the interfaces of the members of the specified trunk.

### get\_trunk\_media\_speed ()

	print "Trunk $t operating at " . $ic->get_trunk_media_speed($t) . "Mbps\n";

Returns the current operational media speed (in Mbps) of the specified trunk.

### get\_trunk\_media\_status ()

	print "Trunk $t media status is " . $ic->get_trunk_media_status($t) . "\n";

Returns the current operational media status of the specified trunk.

### get\_trunk\_lacp\_enabled\_state ()

Returns the enabled state of LACP for the specified trunk.

### get\_trunk\_lacp\_active\_state ()

Returns the active state of LACP for the specified trunk.

### get\_trunk\_statistics ()

Returns the traffic statistics for the specified trunk.  The statistics are returned as a TrunkStatistics object
hence this method is useful where access to raw statistical data is required.

For parsed statistic data, see __get\_trunk\_statistics\_stringified__.

For specific information regarding data and units of measurement for statistics methods, please see the __Notes__ section.

### get\_trunk\_statistics\_stringified ()

Returns all statistics for the specified trunk as a hash of hases with the following structure:

	{	
		timestamp	=> 'yyyy-mm-dd-hh-mm-ss',
		stats		=> {
					stats_1	=> value,
					stats_3	=> value,
					...
					stats_n	=> value
				}
	}
					

This function accepts a single parameter; the trunk for which the statistics are to be returned.

For specific information regarding data and units of measurement for statistics methods, please see the __Notes__ section.

### get\_self\_ip\_list

Returns a list of all self IP addresses on the target device.

### get\_self\_ip\_vlan ( $SELF\_IP )

Returns the VLAN associated with the specified self IP address on the target device.

### get\_vs\_list ()

	my @virtuals	= $ic->get_vs_list();

__Please note__: this method has been deprecated in future releases.  Please use get\_ltm\_vs\_list instead.

Returns an array of all defined LTM virtual servers.

### get\_ltm\_vs\_list ()

	my @ltm_virtuals = $ic->get_ltm_vs_list();

Returns an array of all defined LTM virtual servers.

### get\_gtm\_vs\_list ()

	my @gtm_virtuals = $ic->get_gtm_vs_list();

Returns an array of the names of all defined GTM virtual servers.

### get\_vs\_destination ($virtual\_server)

	my $destination	= $ic->get_vs_destination($vs);

Returns the destination of the specified virtual server in the form ipv4\_address%route\_domain:port.

### get\_vs\_enabled\_state ($virtual\_server)

	print "LTM Virtual server $vs is in state ",$ic->get_vs_enabled_state($vs),"\n";

__Please note__: this method has been deprecated in future releases.  Please use the __get\_ltm\_vs\_enabled\_state()__ instead.

Return the enabled state of the specified LTM virtual server.

### get\_ltm\_vs\_enabled\_state ($virtual\_server)

	print "LTM Virtual server $vs is in state ",$ic->get_ltm_vs_enabled_state($vs),"\n";

Return the enabled state of the specified LTM virtual server.

### get\_gtm\_vs\_enabled\_state ($virtual\_server)

	print "GTM Virtual server $vs is in state ",$ic->get_gtm_vs_enabled_state($vs),"\n";

Return the enabled state of the specified GTM virtual server.  The GTM server should be provided as a name only such as that
returned from the __get\_gtm\_vs\_list__ method.

### get\_vs\_all\_statistics ()

__Please Note__: This method has been deprecated in future releases.  Please use __get\_ltm\_vs\_all\_statistics__.

Returns the traffic statistics for all configured LTM virtual servers.  The statistics are returned as 
VirtualServerStatistics struct hence this method is useful where access to raw statistical data is required.

For parsed statistic data, see __get\_ltm\_vs\_statistics\_stringified__.

For specific information regarding data and units of measurement for statistics methods, please see the __Notes__ section.

### get\_ltm\_vs\_all\_statistics ()

Returns the traffic statistics for all configured LTM virtual servers.  The statistics are returned as 
VirtualServerStatistics struct hence this method is useful where access to raw statistical data is required.

For parsed statistic data, see __get\_ltm\_vs\_statistics\_stringified__.

For specific information regarding data and units of measurement for statistics methods, please see the __Notes__ section.

### get\_vs\_statistics ($virtual\_server)

	my $statistics = $ic->get_vs_statistics($vs);

Returns all statistics for the specified virtual server as a VirtualServerStatistics object.  Consider using get\_vs\_statistics\_stringified
for accessing virtual server statistics in a pre-parsed hash structure.	

For specific information regarding data and units of measurement for statistics methods, please see the __Notes__ section.

### get\_vs\_statistics\_stringified ($virtual\_server)

	my $statistics = $ic->get_vs_statistics_stringified($vs);

	foreach (sort keys %{$stats{stats}}) {
		print "$_: $stats{stats}{$_}\n";
	}

Returns all statistics for the specified virtual server as a multidimensional hash (hash of hashes).  The hash has the following structure:

	{
		timestamp	=> 'yyyy-mm-dd-hh-mm-ss',
		stats		=> {
					statistic_1	=> value,
					statistic_2	=> value,
					...
					statistic_n	=> value
				}
	}

This function accepts a single parameter; the virtual server for which the statistics are to be returned.

For specific information regarding data and units of measurement for statistics methods, please see the __Notes__ section.

### get\_ltm\_vs\_rules ($virtual\_server)

### get\_ltm\_snat\_pool ($virtual\_server)

### get\_ltm\_snat\_type ($virtual\_server)

### get\_default\_pool\_name ($virtual\_server)

	print "Virtual Server: $virtual_server\nDefault Pool: ", 
		$ic->get_default_pool_name($virtual_server), "\n";

Returns the default pool names for the specified virtual server.

### get\_pool\_list ()

	print join " ", ($ic->get_pool_list());

Returns a list of all LTM pools in the target system.

Note that this method has been deprecated in future releases - please use __get\_ltm\_vs\_list__ instead.

### get\_ltm\_pool\_list ()

	print join " ", ($ic->get_ltm_pool_list());

Returns a list of all LTM pools in the target system.

### get\_pool\_members ($pool)

	foreach my $pool ($ic->get_pool_list()) {
		print "\n\n$pool:\n";

		foreach my $member ($ic->get_pool_members($pool)) {
			print "\t$member\n";
		}
	}

__Please note__: this method has been deprecated in future releases.  Please use the __get\_ltm\_pool\_members__ method instead.

Returns a list of the pool members for the specified LTM pool.  This method takes one mandatory parameter; the name of the pool.

Pool member are returned in the format __IP\_address:service\_port__.

### get\_ltm\_pool\_members ($pool)

	foreach my $pool ($ic->get_ltm_pool_list()) {
		print "\n\n$pool:\n";

		foreach my $member ($ic->get_ltm_pool_members($pool)) {
			print "\t$member\n";
		}
	}

Returns a list of the pool members for the specified LTM pool.  This method takes one mandatory parameter; the name of the pool.

Pool member are returned in the format __IP\_address:service\_port__.

### get\_gtm\_pool\_members ($pool)

Returns a list of the pool members for the specified GTM pool.  This method takes one mandatory parameter; the name of the pool.

Pool member are returned in the format __IP\_address:service\_port__.

### get\_pool\_statistics ($pool)

	my %stats = $ic->get_pool_statistics($pool);

Returns the statistics for the specified pool as a PoolStatistics object.  For pre-parsed pool statistics consider using
the __get\_pool\_statistics\_stringified__ method.

### get\_pool\_statistics\_stringified ($pool)

	my %stats = $ic->get_pool_statistics_stringified($pool);
	print "Pool $pool bytes in: $stats{stat}{STATISTIC_SERVER_SIDE_BYTES_OUT}";

Returns a hash containing all pool statistics for the specified pool in a delicious, easily digestable and improved formula.

### get\_pool\_member\_statistics ($pool)

Returns all pool member statistics for the specified pool as an array of MemberStatistics objects.  Unless you feel like 
playing with Data::Dumper on a rainy Sunday afternoon, consider using __get\_pool\_member\_statistics\_stringified__ method.

### get\_pool\_member\_object\_status ($pool)

Returns all pool member stati for the specified pool as an array of MemberObjectStatus objects.

### get\_pool\_member\_statistics\_stringified ($pool)

	my %stats = $ic->get_pool_member_statistics_stringified($pool);

	print "Member\t\t\t\tRequests\n",'-'x5,"\t\t\t\t",'-'x5,"\n";
	
	foreach my $member (sort keys %stats) {
		print "$member\t\t$stats{$member}{stats}{STATISTIC_TOTAL_REQUESTS}\n";
	}

	# Prints a list of requests per pool member

Returns a hash containing all pool member statistics for the specified pool.  The hash has the following
structure;

	member_1 =>	{
			timestamp	=> 'YYYY-MM-DD-hh-mm-ss',
			stats		=>	{
						statistics_1	=> value
						...
						statistic_n	=> value
						}
			}
	member_2 =>	{
			...
			}
	member_n =>	{
			...
			}

Each pool member is specified in the form ipv4\_address%route\_domain:port.

### get\_all\_pool\_member\_statistics ($pool)

Returns all pool member statistics for the specified pool.  This method is analogous to the __get\_pool\_member\_statistics()__
method and the two will likely be merged in a future release.

### get\_ltm\_pool\_status ($pool)

Returns the status of the specified pool as a ObjectStatus object.

For formatted pool status information, see the __get\_ltm\_pool\_status\_as\_string()__ method.

### get\_ltm\_pool\_member\_status ($pool, $member)

Returns the status of the specified member in the specified pool as a ObjectStatus object.

### get\_ltm\_pool\_availability\_status ($pool)

Retuns the availability status of the specified pool.

### get\_ltm\_pool\_enabled\_status ($pool)

Retuns the enabled status of the specified pool.

### get\_ltm\_pool\_status\_description ($pool)

Returns a descriptive status of the specified pool.

### get\_ltm\_pool\_status\_as\_string ($pool)

Returns the pool status as a descriptive string.

### get\_connection\_list ()

Returns a list of active connections as a list of ConnectionID objects.

### get\_all\_active\_connections ()

Gets all active connections in details on the device.

### get\_active\_connections\_count()

Returns the number of all active connections on the device.

### get\_node\_list ()

	print join "\n", ($ic->get_node_list());

Returns a list of all configured nodes in the target system.

Nodes are returned as ipv4 addresses.

### get\_screen\_name ($node)

	foreach ($ic->get_node_list()) {
		print "Node: $_ (" . $ic->get_screen_name($_) . ")\n";
	}

Retuns the screen name of the specified node.

### get\_node\_status ($node)

	$ic->get_node_status(

Returns the status of the specified node as a ObjectStatus object.

For formatted node status information, see the __get\_node\_status\_as\_string()__ method.

### get\_node\_availability\_status ($node)

Retuns the availability status of the node.

### get\_node\_enabled\_status ($node)

Retuns the enabled status of the node.

### get\_node\_status\_description ($node)

Returns a descriptive status of the specified node.

### get\_node\_status\_as\_string ($node)

Returns the node status as a descriptive string.

### get\_node\_monitor\_status ($node)

Gets the current availability status of the specified node addresses. 

### get\_node\_statistics ($node)

Returns all statistics for the specified node.

### get\_node\_statistics\_stringified

	my %stats = $ltm->get_node_statistics_stringified($node);

	foreach (sort keys %{stats{stats}}) {
		print "$_:\t$stats{stats}{$_}{high}\t$stats{stats}{$_}{low}\n";
	}

Returns a multidimensional hash containing all current statistics for the specified node.  The hash has the following structure:

	{
		timestamp	=> 'yyyy-mm-dd-hh-mm-ss',
		stats		=> {
					statistic_1	=> value,
					statistic_2	=> value,
					...
					statistic_n	=> value
				}
	}

This function accepts a single parameter; the node for which the statistics are to be returned.

For specific information regarding data and units of measurement for statistics methods, please see the __Notes__ section.

### get\_gtm\_pool\_list ()

Returns a list of GTM pools.

### get\_gtm\_pool\_description ()

Returns a description of the specified GTM pool.

### get\_gtm\_vs\_all\_statistics ()

Returns the traffic statistics for all configured GTM virtual servers.  The statistics are returned as 
VirtualServerStatistics struct hence this method is useful where access to raw statistical data is required.

For parsed statistic data, see __get\_gtm\_vs\_statistics\_stringified__.

For specific information regarding data and units of measurement for statistics methods, please see the __Notes__ section.

### get\_ltm\_address\_class\_list ()

Returns a list of all existing address classes.

### get\_ltm\_string\_class\_list ()

Returns a list of all existing string classes.

### get\_ltm\_string\_class ( $class\_name )

Return the specified LTM string class.

### get\_ltm\_string\_class\_members ( $class )

Returns the specified LTM string class members.

### add\_ltm\_string\_class\_member ( $class, $member )

Add the provided member to the specified class.

### delete\_ltm\_string\_class\_member ( $class, $member )

Deletes the provided member from the specified class.

### set\_ltm\_string\_class\_member ( $class, $member, value )

Sets the value of the member to the provided value in the specified class.

### get\_db\_variable ( $VARIABLE )

	# Prints the value of the configsync.state database variable.
	print "Config state is " . $ic->get_db_variable('configsync.state') . "\n";

Returns the value of the specified db variable.

### get\_sync\_status ( $DEVICE\_GROUP )

Accepts one mandatory parameter; the device group name for which to return the sync status, and returns a SyncStatus struct 
containing information on the ConfigSync status of the specified device group.

### get\_sync\_status\_overview 

Gets the sync status of the system containing information on the sync status of 
the current device's presence in all device groups in which it is a member and 
returns a SyncStatus struct.

### get\_sync\_time\_diff ( $DEVICE\_GROUP )

Accepts one mandatory parameter; the device group name for which to return the sync status and returns the number of seconds
between the oldest successful ConfigSync and the current time for all devices in the device group.

That is; if there are unsynchronised changes in the device group, then the value returned will be the delta in seconds
between the oldest successful synchronisation in the device group and the current time.

### get\_device\_group\_list ( $DEVICE\_GROUP )

Accepts one mandatory parameter; the device group name for which to return the device group list.

### get\_device\_group\_type ( $DEVICE\_GROUP )

Accepts one mandatory parameter; the device group name for which to return the device group type.

### get\_event\_subscription\_list

Returns an array of event subscription IDs for all registered event subscriptions.

### get\_event\_subscription

### remove\_event\_subscription

### get\_event\_subscription\_state

### get\_event\_subscription\_url

### get\_subscription\_list

This method is an analog of __get\_event\_subscription__

### create\_subscription\_list (%args)

        my $subscription = $ic->create_subscription_list (
                                                name                            => 'my_subscription_name',
                                                url                             => 'http://company.com/my/eventnotification/endpoint,
                                                username                        => 'username',
                                                password                        => 'password',
                                                ttl                             => -1,
                                                min_events_per_timeslice        => 10,
                                                max_timeslice                   => 10
                                        );   

Creates an event subscription with the target system.  This method requires the following parameters:

- name 

    A user-friendly name for the subscription.

- url

    The target URL endpoint for the event notification interface to send event notifications.

- username

    The basic authentication username required to access the URL endpoint.

- password

    The basic authentication password required to access the URL endpoint.

- ttl

    The time to live (in seconds) for this subscription. After the ttl is reached, the subscription
    will be removed from the system. A value of -1 indicates an infinite life time.

- min\_events\_per\_timeslice

    The minimum number of events needed to trigger a notification. If this value is 50, then this
    means that when 50 events are queued up they will be sent to the notification endpoint no matter
    what the max\_timeslice is set to.

- max\_timeslice

    This maximum time to wait (in seconds) before event notifications are sent to the notification
    endpoint. If this value is 30, then after 30 seconds a notification will be sent with the events
    in the subscription queue.

### certificate\_add\_pem\_to\_bundle (mode, cert\_ids, pem\_data)

Adds certificates identified by "pem\_data" to the certificate bundles, which are presumed to exist already.

### certificate\_bind (mode, cert\_ids, key\_ids)

Binds/associates the specified keys and certificates.

### certificate\_delete (mode, cert\_ids)

Deletes/uninstalls the specified certificates.

### certificate\_delete\_from\_bundle (mode, cert\_ids, x509\_data)

Deletes certificates, identified by their subject's X509 data, from the certificate bundles.

### certificate\_export\_to\_pem (mode, cert\_ids)

Get the specified certificates as PEM strings.

### certificate\_import\_from\_pem (mode, cert\_ids, pem\_data, overwrite)

Imports/installs the specified certificates from the given PEM-formatted data.

### get\_key\_list (mode)

Get the list of keys in the system.

### get\_certificate\_bundle (mode, file\_names)

Gets the list of all certificates bundled in the certificate files as specified by the file\_names.

### get\_certificate\_list (mode)

Get the list of certificates in the system.

### key\_delete (mode, key\_ids)

Deletes/uninstalls the specified keys.

### key\_export\_to\_pem (mode, key\_ids)

Get the specified certificates as PEM strings.

### key\_import\_from\_pem (mode, key\_ids, pem\_data, overwrite)

Imports/installs the specified keys from the given PEM-formatted data.

### create\_user\_3 ()

Create the specified new users.

### change\_password\_2 ()

Change the user's password.

### delete\_user ()

Delete the specified users.

### get\_user\_list ()

List all users.

### get\_encrypted\_password (user\_names)

Gets the encrypted passwords of the specified users.

### get\_user\_id (user\_names)

Get the User IDs for the given usernames.

### get\_login\_shell (user\_names)

Get the login shells for the given usernames.

### set\_login\_shell (user\_names, shells)

Sets the login shells for the specified users.

### get\_user\_permission (user\_names)

Gets the permissions of the specified users.

### set\_user\_permission (user\_names, permissions)

Sets the permissions of the specified users.

## NOTES

### Statistic Methods

Within iControl, statistical values are a 64-bit unsigned integer represented as a __Common::ULong64__ object.
The ULong64 object is a stuct of two 32-bit values.  This representation is used as there is no native 
support for the encoding of 64-bit numbers in SOAP.

The ULong object has the following structure;

	({
		STATISTIC_NAME	=> {
				high	=> long
				low	=> long
			}
	}, bless Common::ULong64)

Where high is the unsigned 32-bit integer value of the high-order portion of the measured value and low is 
the unsigned 32-bit integer value of the low-order portion of the measured value.

In non-stringified statistic methods, these return values are ULong64 objects as returned by the iControl API.
In stringified statistic method calls, the values are processed on the client side into a local 64-bit representation
of the value using the following form.

	$value = ($high<<32)|$low;

Stringified method calls are guaranteed to return a correct localised 64-bit representation of the value.

It is the callers responsibility to convert the ULong struct for all other non-stringified statistic method calls.

## AUTHOR

Luke Poskitt, <ltp@cpan.org>

Thanks to Eric Welch, <erik.welch@gmail.com>, for input and feedback.

## LICENSE AND COPYRIGHT

This program is free software; you can redistribute it and/or modify it
under the terms of either: the GNU General Public License as published
by the Free Software Foundation; or the Artistic License.

See http://dev.perl.org/licenses/ for more information.
