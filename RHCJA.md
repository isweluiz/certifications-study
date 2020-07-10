## Configure e install Jboss EAP 7

- 1.Create a service user
`sudo useradd --no-create-home --shel /bin/false/ jboss-eap`

- 2.Set the JBOSS_HOME AND JBOSS_USER
`sudo vim /opt/jboss-eap/bin/init.d/jboss-eap.conf`

- 3 - Copy the service files
`sudo cp /opt/jboss-eap/bin/init.d/jboss-eap.conf /etc/default/`
 `sudo cp /opt/jboss-eap/bin/init.d/jboss-eap-rhel.sh /etc/init.d/`

- 4.Make the script executable
`sudo chmod +x /etc/init.d/jboss-eap-rhel.sh`

- 5.Set the service to start automatically
`sudo chkconfig --add jboss-eap-rhel.sh`
`sudo chkconfig jboss-eap.sh on`
`systemctl enable`

- 6.Create the JBoss EAP's run directory and set the ownership
`sudo mkdir /var/run/jboss-eap`
`sudo mkdir /var/run/jboss-eap`

- 7.Change the ownership of the jboss_home directory
`sudo chown -R jboss-eap:jboss-eap /opt/jboss-eap/`
