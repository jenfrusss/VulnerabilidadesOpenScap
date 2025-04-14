# VulnerabilidadesOpenScap
Aqui estan unas vulnerabilidades para producir en entorno de practica,importante son la distribucion de Rocky Linux

******************************************************************************************************************************************************************************************************************************************************************************************************************************************************************************************************************************************

--Establecer Permisos Incorrectos en Archivos Críticos

sudo chmod 777 /etc/passwd
sudo chmod 777 /etc/shadow

--Instalar Servicios no Seguros

-HTTP

sudo yum install httpd -y
sudo systemctl start httpd
sudo systemctl enable httpd
sudo echo '<Directory "/var/www/html"> AllowOverride All Require all granted </Directory>' >> /etc/httpd/conf/httpd.conf
sudo systemctl restart httpd

-Telnet

sudo yum install telnet telnet-server -y
sudo systemctl enable telnet.socket
sudo systemctl start telnet.socket

--FTP

sudo yum install vsftpd -y
sudo systemctl start vsftpd
sudo systemctl enable vsftpd
sudo sed -i 's/^#anonymous_enable=NO/anonymous_enable=YES/' /etc/vsftpd/vsftpd.conf
sudo systemctl restart vsftpd

--Dejar un Puerto Abierto con un Servicio Vulnerable:

sudo yum install nc -y
echo -e "#!/bin/bash\nwhile :; do nc -lvp 9999 -e /bin/bash; done" > /tmp/backdoor.sh
chmod +x /tmp/backdoor.sh
sudo /tmp/backdoor.sh &

--Crear Reglas de Firewall Extremadamente Permisivas:

sudo firewall-cmd --add-port=1-65535/tcp --permanent
sudo firewall-cmd --reload

--Activar SNMP con Configuración Predeterminada (Puerto 161):

sudo yum install net-snmp-utils net-snmp -y
sudo sudo sed -i 's|^com2sec notConfigUser  default       public|com2sec notConfigUser  0.0.0.0/0    public|' /etc/snmp/snmpd.conf
sudo systemctl restart snmpd
sudo firewall-cmd --add-port=161/udp --permanent
sudo firewall-cmd --reload

--Habilitar Samba sin Restricciones (Puertos 137-139, 445):

sudo yum install samba -y
sudo cp /etc/samba/smb.conf /etc/samba/smb.conf.bak
sudo cat > /etc/samba/smb.conf <<EOL
[global]
    workgroup = WORKGROUP
    security = user
    map to guest = Bad User

[shared]
    path = /srv/samba/shared
    browsable = yes
    writable = yes
    guest ok = yes
EOL
sudo mkdir -p /srv/samba/shared
sudo chmod -R 777 /srv/samba/shared
sudo systemctl start smb
sudo systemctl enable smb
sudo firewall-cmd --add-port=137-139/tcp --permanent
sudo firewall-cmd --add-port=445/tcp --permanent
sudo firewall-cmd --reload
