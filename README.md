# arduino-yun-snmp
configuration snmp  avec oid  arduino Yun

il faut pour configurer le snmp sur l'arduino yun :
# 1) installer snmpdet snmp-utils
faire un ssh sur l'arduino
opkg update
opkg install snmp-utils
opkg install snmpd

# 2) lancer snmpd
/etc/init.d/snmpd enable
/etc/init.d/snmpd restart

# 3) tester snmp : mettre la valeur 100

snmpset -c private -v2c localhost 1.3.6.1.2.1.31.1.1.1.18.1 s 100 2>/dev/null

l'arduino repond : iso.3.6.1.2.1.31.1.1.1.18.1 = STRING: "100"

relire la valeur :
snmpget -c private -v2c localhost 1.3.6.1.2.1.31.1.1.1.18.1 2>/dev/null


l'arduino repond : iso.3.6.1.2.1.31.1.1.1.18.1 = STRING: "100"


# 4)configuration et modification pour avoir son oid 

exemple : nous allons lancer un script pour retrouver le nom hostname avec l'oid 1.3.6.1.4.1.36582.1.1.1.1.101.1
# editer le fichier  snmpd dans /etc/config/snmpd avec nano

nano /etc/config/snmpd

/etc/config/snmpd 
config agent
	option agentaddress UDP:161

config com2sec public
	option secname ro
	option source default
	option community public

config com2sec private
	option secname rw
	option source localhost
	option community private

config group public_v1
	option group public
	option version v1
	option secname ro

config group public_v2c
	option group public
	option version v2c
	option secname ro

config group public_usm
	option group public
	option version usm
	option secname ro

config group private_v1
	option group private
	option version v1
	option secname rw

config group private_v2c
	option group private
	option version v2c
	option secname rw

config group private_usm
	option group private
	option version usm
	option secname rw

config view all
	option viewname all
	option type included
	option oid .1

config access public_access
	option group public
	option context none
	option version any
	option level noauth
	option prefix exact
	option read all
	option write none
	option notify none

config access private_access
	option group private
	option context none
	option version any
	option level noauth
	option prefix exact
	option read all
	option write all
	option notify all

config system
	option sysLocation	'office'
	option sysContact	'bofh@example.com'
	option sysName		'HeartOfGold'
#	option sysServices	72
#	option sysDescr		'adult playground'
#	option sysObjectID	'1.2.3.4'

config exec
	option name	filedescriptors
	option prog	/bin/cat
	option args	/proc/sys/fs/file-nr
#	option miboid	1.2.3.4



rajouter a la fin le code suivant et sauver le fichier (CTRL + X et SAVE Yes)

config exec
	option name hostname_
	option prog	/bin/cat
	option args	/proc/sys/kernel/hostname
	option miboid	.1.3.6.1.4.1.36582.1.1.1.1


# 5) redemarer snmpd

/etc/init.d/snmpd restart

# 6) le fichier snmpd.conf a bien ete alors modifie et a ajoute notre code (ce fichier est reconstruit a chaque reboot eu redemarage de snmpd)

cat /etc/snmp/snmpd.conf 

agentaddress UDP:161
sysLocation office
sysContact bofh@example.com
sysName HeartOfGold
com2sec ro default public
com2sec rw localhost private
group public v1 ro
group public v2c ro
group public usm ro
group private v1 rw
group private v2c rw
group private usm rw
view all included .1 
access public "" any noauth exact all none none
access private "" any noauth exact all all all
exec  filedescriptors /bin/cat /proc/sys/fs/file-nr
exec .1.3.6.1.4.1.36582.1.1.1.1 hostname_ /bin/cat /proc/sys/kernel/hostname



# 7) lancer notre script et recuperer la valeur hostname

la strucutre est la suivante :

snmpwalk -c public -v2c localhost 1.3.6.1.4.1.36582.1.1.1.1 2>/dev/null
iso.3.6.1.4.1.36582.1.1.1.1.1.1 = INTEGER: 1
iso.3.6.1.4.1.36582.1.1.1.1.2.1 = STRING: "hostname_"
iso.3.6.1.4.1.36582.1.1.1.1.3.1 = STRING: "/bin/cat /proc/sys/kernel/hostname"
iso.3.6.1.4.1.36582.1.1.1.1.100.1 = INTEGER: 0
iso.3.6.1.4.1.36582.1.1.1.1.101.1 = STRING: "Arduino"
iso.3.6.1.4.1.36582.1.1.1.1.102.1 = INTEGER: 0
iso.3.6.1.4.1.36582.1.1.1.1.103.1 = ""



snmpget -c public -v2c localhost 1.3.6.1.4.1.36582.1.1.1.1.101.1 2>/dev/null
iso.3.6.1.4.1.36582.1.1.1.1.101.1 = STRING: "Arduino"











