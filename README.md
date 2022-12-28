# phpipam_get_hosts

Aims at synchronizing hosts registered in phpipam with their reservations in a dhcp server like ISC DHCP

Can be called from another script (called periodically by crontab) containing rougly:

db=sync_subnets.db # A file containing a phpipam subnet number followed by isc dhcpd file
phpipam_hosts_cmd=/path_to/phpipam_get_hosts.sh
dhcp_reload_cmd=/path_to/dhcp-user-reload.sh
rundate=`date --rfc-3339=seconds`

cat $db | grep -Ev '^$|^#' | \
	while read subnetnum iscfile
	do
		$phpipam_hosts_cmd -d -A -u $subnetnum -o $iscfile -r "echo ${rundate} ${iscfile} changed >> dhcp_needs_reload"
	done

if [ -f "dhcp_needs_reload" ]; then
	cat dhcp_needs_reload
	echo "reloading dhcp server"
	$dhcp_reload_cmd && unlink dhcp_needs_reload
fi


