vicidial-setup
==============

vicidial from scratch

ViciDial Installation Steps
1.  Disk Partitioning
1.1 Get at least 2 hard drives of equal size
1.2 Create SWAP in both the drives, totalling the double size of the RAM 
(a)	Check the memory size by going to 
#cat /proc/meminfo
1.3 Create one software RAID partition of 500 MB, repeat this process for another drive as well. It’ll need to be in RAID-1.
1.4 Create one more software RAID partition of all the available size, repeat this process for another drive as well. It’ll also need to be in RAID-1.
1.5 There will be now 4 RAID partitions, have 1.1 and 2.1 (1st drive 1st RAID and 2nd drive 1st – 500 MB and mount as /boot, boot flag –on. 0% and Primary Partition
1.6 Have the remaining RAID partition as a Volume Group. Create one volume 100 GB as root (Mount as ‘/’ – ext4, and remaining create volume as var (Mount as ‘/var’ –ext4).
1.7	Write changes and continue.
2.	System Configuration
2.1 Have basic editor installed
#apt-get install vim 
#cat /etc/network/interface
#vi /etc/network/interface
(Press i to go to Insert Mode – Esc and :wq to write and quit or q! to exit without saving)
3.	Install basic packages before you continue (MUST)
#apt-get install build-essential libncurses5-dev emacs23-nox wget bzip2
4.	Kernel compilation
a.	Kernel compilation is only for ViciDial (Skip if you are making just an Asterisk Server)
b.	Always make /usr/src as your working directoty
4.1 Get the latest Kernel from kernel.org (We are using 3.3.7 for this) and configure
#wget http://www.kernel.org/pub/linux/kernel/v3.0/linux-3.3.7.tar.bz2
#tar xvjf linux-3.3.7.tar.bz2
#cd linux-3.3.7
#make clean
#make mrproper
#make menuconfig
	Processor Type and Features:
Tickless System
Symmetric multi-processing support
High Resolution Timer Support 
 
Timer frequency:	1000 HZ
High Memory Support:	64GB
Preemption Model:	“No Forced Preemption (Server)”
Processor family:	`cat /proc/cpuinfo | grep "model name"`
 
#make
#make modules
#make install
#make modules_install
#cd /boot
#mkinitramfs -o initrd.img-3.3.7 3.3.7
#update-grub
#init 6 (Which will Reboot...)
5.	Compile LibPri
#cd /usr/src
#wget http://downloads.asterisk.org/pub/telephony/libpri/releases/libpri-1.4.X.tar.gz
#make
#make install
6.	 Compile DAHDI
#cd /usr/src
#wget 	http://downloads.asterisk.org/pub/telephony/dahdi-linux-complete/releases/dahdi-linux-complete-x.x.x.tar.gz
#cd dahdi-linux-complete-x.x.x
#make
#make install
#make config
7.	 Compile Asterisk	
#cd /usr/src
#wget http://downloads.asterisk.org/pub/telephony/asterisk/releases/asterisk-1.4.4.tar.gz
#tar xvzf asterisk-1.4.4.tar.gz
#cd asterisk-1.4.4
#./configure
# make menuconfig
Instructions: you may need to ssh into the system for this to work.
Check that the meetme function is set true. (if the option is not available check that DAHDI is installed correctly)
Select EN_AU sounds from sound packages.
#make
#make install
#make samples
 
8.	 Install ViciDial Requirements
#apt-get install apache2 apache2-mpm-prefork mysql-client mysql-server phpmyadmin ploticus screen sox sox-* unzip curl libyaml-perl;
#apt-get build-dep libdbd-mysql-perl
#cpan
install MD5
install Digest::MD5
install Digest::SHA1
install readline
install Bundle::CPAN
quit
 
#cpan
o conf commit
force install Scalar::Util
install DBI
force install DBD::mysql
install Net::Server
install Switch
install Time::HiRes
install Net::Telnet
install Unicode::Map
install Jcode
install OLE::Storage_Lite
install Spreadsheet::WriteExcel
install Proc::ProcessTable
install Spreadsheet::ParseExcel
install Mail::Sendmail
install Spreadsheet::XLSX
install Spreadsheet::Read
quit
 
9.	 Compile Asterisk Perl
#cd /usr/src
#wget http://asterisk.gnuinter.net/files/asterisk-perl-0.08.tar.gz
#tar xzvf asterisk-perl-0.08.tar.gz
#cd asterisk-perl-0.08
#perl Makefile.PL
#make all
#make install
10.	Install ViciDial
#cd /usr/src
#wget http://downloads.sourceforge.net/project/astguiclient/astguiclient_2.4rc2.zip 
#unzip astguiclient_2.4rc2.zip
#mv agc_2.4rc2 astguiclient 
#cd astguiclient
 (Or, download rc1 if you are not sure about new release) / ONLY CHOOSE ONE
#wget http://downloads.sourceforge.net/project/astguiclient/astguiclient_2.4rc1.zip
#unzip astguiclient_2.4rc1.zip
#mv agc_2.4rc1 astguiclient
#cd astguiclient
#perl install.pl
(Note: Set webroot to /var/www, cron password to something more secure, sample configuration files to y. All other options can stay on defaults.)
11.	Tweak PHP/Apache2 Settings
 
#emacs /etc/php5/apache2/php.ini
 
Note: Set the following values
upload_max_filesize = 128M
post_max_size = 128M
 
default_socket_timeout = 300
mysql.connect_timeout = 300
max_execution_time = 600
max_input_time = 600
 
memory_limit = 512M
(Press ctrl+x and then Y "enter" to save)
	
12.	Database Configuration
 	#mysqladmin -u root password YOURNEWPASSWORD
#mysql -p (enter the mysql root user's password you set earlier)
> CREATE DATABASE `asterisk` DEFAULT CHARACTER SET utf8 COLLATE utf8_unicode_ci;
> GRANT SELECT,INSERT,UPDATE,DELETE,LOCK TABLES on asterisk.* TO cron@'%' IDENTIFIED BY '1234';
> GRANT SELECT,INSERT,UPDATE,DELETE,LOCK TABLES on asterisk.* TO cron@localhost IDENTIFIED BY '1234';
> use asterisk;
> \. /usr/src/astguiclient/trunk/extras/MySQL_AST_CREATE_tables.sql
> \. /usr/src/astguiclient/trunk/extras/first_server_install.sql
> \. /usr/src/astguiclient/trunk/extras/sip-iax_phones.sql
> quit
 
Add my.cnf
#cd /etc/mysql/
#wget http://download.vicidial.com/ubuntu/ubuntu-my-vici.cnf
#mv ubuntu-my-vici.cnf my.cnf
#/etc/init.d/mysql restart
#/usr/share/astguiclient/ADMIN_update_server_ip.pl --old-server_ip=10.10.10.15
#/usr/share/astguiclient/ADMIN_area_code_populate.pl
#cp /usr/src/astguiclient/trunk/extras/performance_test_leads.txt /usr/share/astguiclient/LEADS_IN/
#/usr/share/astguiclient/VICIDIAL_IN_new_leads_file.pl --forcelistid=107 --forcephonecode=1
 
OR, for step 11, you can alternative do this way;
use asterisk;
 
\. /usr/src/astguiclient/trunk/extras/MySQL_AST_CREATE_tables.sql
	or you may need to run this if you get an error:
	\. /usr/src/astguiclient/agc_2.0.5/extras/MySQL_AST_CREATE_tables.sql
	\. /usr/src/astguiclient/astguiclient/MySQL_AST_CREATE_tables.sql
 
### to load in default IAX and SIP phone accounts run the following query
 
\. /usr/src/astguiclient/trunk/extras/sip-iax_phones.sql
	or you may need to run this if you get an error:
	\. /usr/src/astguiclient/agc_2.0.5/extras/sip-iax_phones.sql
	\. /usr/src/astguiclient/astguiclient/sip-iax_phones.sql
 
### to load the initial server values for this first system install
 
\. /usr/src/astguiclient/trunk/extras/first_server_install.sql
	or you may need to run this if you get an error:
	\. /usr/src/astguiclient/agc_2.0.5/extras/first_server_install.sql
	\. /usr/src/astguiclient/astguiclient/first_server_install.sql
 
quit
 
  to populate the timezone/country table run this command from command line:
   - /usr/share/astguiclient/ADMIN_area_code_populate.pl
 
  to load the performance testing leads run these commands:
   - cp /usr/src/astguiclient/trunk/extras/performance_test_leads.txt /usr/share/astguiclient/LEADS_IN/
       or
	   - cp /usr/src/astguiclient/agc_2.0.5/extras/performance_test_leads.txt /usr/share/astguiclient/LEADS_IN/
	   - cp /usr/src/astguiclient_2.0.5/trunk/extras/performance_test_leads.txt /usr/share/astguiclient/LEADS_IN/
 
   - /usr/share/astguiclient/VICIDIAL_IN_new_leads_file.pl --forcelistid=107 --forcephonecode=1
 
13.	Setting up astguiclient scripts for continuous running
 
#crontab –e
 
### remove old vicidial logs and asterisk logs more than 2 days old
28 0 * * * /usr/bin/find /var/log/astguiclient -maxdepth 1 -type f -mtime +2 -print | xargs rm -f
29 0 * * * /usr/bin/find /var/log/asterisk -maxdepth 3 -type f -mtime +2 -print | xargs rm -f
30 0 * * * /usr/bin/find / -maxdepth 1 -name "screenlog.0*" -mtime +4 -print | xargs rm -f
### fix the vicidial_agent_log once every hour and the full day run at night
33 * * * * /usr/share/astguiclient/AST_cleanup_agent_log.pl
50 0 * * * /usr/share/astguiclient/AST_cleanup_agent_log.pl --last-24hours
## uncomment below if using QueueMetrics
#*/5 * * * * /usr/share/astguiclient/AST_cleanup_agent_log.pl --only-qm-live-call-check
 
### keepalive script for astguiclient processes
* * * * * /usr/share/astguiclient/ADMIN_keepalive_ALL.pl
 
### updater for VICIDIAL hopper
* * * * * /usr/share/astguiclient/AST_VDhopper.pl -q
 
### adjust the GMT offset for the leads in the vicidial_list table
1 1,7 * * * /usr/share/astguiclient/ADMIN_adjust_GMTnow_on_leads.pl --debug --postal-code-gmt
 
### optimize the database tables within the asterisk database
3 1 * * * /usr/share/astguiclient/AST_DB_optimize.pl
 
### VICIDIAL agent time log weekly summary report generation
2 0 * * 0 /usr/share/astguiclient/AST_agent_week.pl
 
### roll logs monthly on high-volume dialing systems
#30 1 1 * * /usr/share/astguiclient/ADMIN_archive_log_tables.pl
 
## uncomment below if using Vtiger
#1 1 * * * /usr/share/astguiclient/Vtiger_optimize_all_tables.pl --quiet
 
### recording mixing/compressing/ftping scripts
#0,3,6,9,12,15,18,21,24,27,30,33,36,39,42,45,48,51,54,57 * * * * /usr/share/astguiclient/AST_CRON_audio_1_move_mix.pl
0,3,6,9,12,15,18,21,24,27,30,33,36,39,42,45,48,51,54,57 * * * * /usr/share/astguiclient/AST_CRON_audio_1_move_mix.pl --MIX
0,3,6,9,12,15,18,21,24,27,30,33,36,39,42,45,48,51,54,57 * * * * /usr/share/astguiclient/AST_CRON_audio_1_move_VDonly.pl
1,4,7,10,13,16,19,22,25,28,31,34,37,40,43,46,49,52,55,58 * * * * /usr/share/astguiclient/AST_CRON_audio_2_compress.pl --MP3
#2,5,8,11,14,17,20,23,26,29,32,35,38,41,44,47,50,53,56,59 * * * * /usr/share/astguiclient/AST_CRON_audio_3_ftp.pl --MP3
 
### kill Hangup script for Asterisk updaters
* * * * * /usr/share/astguiclient/AST_manager_kill_hung_congested.pl
 
### updater for voicemail
* * * * * /usr/share/astguiclient/AST_vm_update.pl
 
### updater for conference validator
* * * * * /usr/share/astguiclient/AST_conf_update.pl
 
### flush queue DB table every hour for entries older than 1 hour
11 * * * * /usr/share/astguiclient/AST_flush_DBqueue.pl -q
 
### reset several temporary-info tables in the database
2 1 * * * /usr/share/astguiclient/AST_reset_mysql_vars.pl
 
### remove old recordings more than 7 days old
#24 0 * * * /usr/bin/find /var/spool/asterisk/monitorDONE -maxdepth 2 -type f -mtime +7 -print | xargs rm -f
 
### Reboot nightly to manage asterisk issues and memory leaks - uncomment if issues arise
#30 6 * * * /sbin/reboot
 
### remove text to speech file more than 4 days old
#20 0 * * * /usr/bin/find /var/lib/asterisk/sounds/tts/ -maxdepth 2 -type f -mtime +4 -print | xargs rm –f
 
 
 
14.	Create vicidial executable in init.d and replace zaptel with DAHDI
 
#cd /etc/init.d/
#wget http://download.vicidial.com/ubuntu/vicidial
Note: modify this if using DAHDI in place of zaptel
#vi vicidial
Note: Replace the below part ZTCFG_BIN=/sbin/ztcfg ----to ----- 
ZTCFG_BIN=/bin/true
Note: Comment the line below 
#if /sbin/lsmod | /bin/grep zaptel
Note: Add this line
if /bin/true
Note: Press Esc and type :wq to exit the editor
#chmod +x vicidial
#update-rc.d -f vicidial defaults
 
15.	Setting up asterisk and helper applications for start-up
 
Note: Make several entries in the rc.local of your system:
		#vi /etc/init.d/rc.local
	Note: add the following entries
	### start time server
	/usr/sbin/ntpdate -u 18.145.0.30
	### sleep for 20 seconds before launching Asterisk
	sleep 20
	### start up asterisk
	/usr/share/astguiclient/start_asterisk_boot.pl
 
16.	OPTIONAL 
   Load eAccelerator PHP-caching application:
   Even so this is technically an optional part, it is strongly recommended that you install eAccelerator
   since it will slash PHPs processing power requirements greatly. Without eAccelerator load on the system 
   can be ten times as high, which can cause all kinds of problems, this is especially true for single 
   system setups. 
   - Go to http://eaccelerator.net and download the most recent package
	- cd /usr/local
	- wget http://bart.eaccelerator.net/source/0.9.5/eaccelerator-0.9.5.3.zip
	- unzip eaccelerator-0.9.5.3.zip
	- cd eaccelerator-0.9.5.3
	- export PHP_PREFIX="/usr/local"
	- $PHP_PREFIX/bin/phpize
	- ./configure --enable-eaccelerator=shared --with-php-config=$PHP_PREFIX/bin/php-config
	- make
	- make install
	- vi /usr/local/lib/php.ini
	    Add the following lines to the dynamic extensions section of php.ini:
	     (you may need to change the extension location depending on your install of php)
		zend_extension="../../../usr/local/eaccelerator-0.9.5/modules/eaccelerator.so"
		eaccelerator.shm_size="48"
		eaccelerator.cache_dir="/tmp/eaccelerator"
		eaccelerator.enable="1"
		eaccelerator.optimizer="1"
		eaccelerator.check_mtime="1"
		eaccelerator.debug="0"
		eaccelerator.filter=""
		eaccelerator.shm_max="0"
		eaccelerator.shm_ttl="0"
		eaccelerator.shm_prune_period="0"
		eaccelerator.shm_only="0"
		eaccelerator.compress="1"
		eaccelerator.compress_level="9"
	- mkdir /tmp/eaccelerator
	- chmod 0777 /tmp/eaccelerator
	# to verify installation:
	- php -v
 
17.	Have the URL redirection for index page, and hardcode phone and provide the user login url only if required.
