<p># OpenVPN-Howto<br />OpenVPN - Howto create and use (server client)</p>
<p># Thank to C<br />Install VPN:<br />Key Setup:<br /># Install the openvpn<br />sudo apt install openvpn easy-rsa<br /># Setting up our CA
  <br /># Make a dir where our keys will be
  <br />mkdir /etc/openvpn/easy-rsa/
  <br /># Copy all the keys to the new folder<br />cp -r /usr/share/easy-rsa/* /etc/openvpn/easy-rsa/<br /># Edit our VPN variables<br />nano /etc/openvpn/easy-rsa/vars
  <br />export KEY_COUNTRY="NL"
  <br />export KEY_PROVINCE="NH"
  <br />export KEY_CITY="Amsterdam"
  <br />export KEY_ORG="Shabadabadooo"
  <br />export KEY_EMAIL="hackaroo@shabadabadoo.com"
  <br />export KEY_CN=VPN
  <br />export KEY_ALTNAMES=WEBCAFE
  <br />export KEY_NAME=VPN
  <br />export KEY_OU=VPN
  <br /># Generate CA cert and keys
  <br />cd /etc/openvpn/easy-rsa/
  <br />source vars
  <br />./clean-all
  <br />./build-ca
  <br /># Generate server cert and keys (and reply yes to all)
  <br />./build-key-server raygun
  <br /># Build Diffie Hellman parameters
  <br />./build-dh<br /># Copy our keys to openvpn folder
  <br />cd keys/
  <br />cp raygun.crt raygun.key ca.crt dh2048.pem /etc/openvpn</p>
<p># Generate client certificates
  <br />cd /etc/openvpn/easy-rsa/
  <br />source vars<br />
  ./build-key clientk<br />
  # Copy them to our computer<br />
  scp username@hostname:/etc/openvpn/easy-rsa/keys/ca.crt /home/user/Downloads/openvpn/ca.crt
  <br />scp username@hostname:/etc/openvpn/easy-rsa/keys/clientk.crt /home/user/Downloads/openvpn/clientk.crt
  <br />scp username@hostname:/etc/openvpn/easy-rsa/keys/clientk.key /home/user/Downloads/openvpn/clientk.key
  <br /># After the copy is done, remove the keys from the server
  <br />rm /etc/openvpn/easy-rsa/clientk.*
  <br /><br />Server Configuration:
  <br /># Unroll the configurations
  <br />sudo cp /usr/share/doc/openvpn/examples/sample-config-files/server.conf.gz /etc/openvpn
  <br />sudo gzip -d /etc/openvpn/server.conf.gz
  <br /># Edit the configuration
  <br />vi /etc/openvpn/server.conf
  <br /># Point to the correct certs and keys
  <br />ca ca.crt
  <br />cert raygun.crt
  <br />key raygun.crt
  <br />dh dh2048.pem
  <br />topology subnet 
  <br />push "redirect-gateway autolocal"
  <br />push "dhcp-option DNS 8.8.8.8"
  <br />comp-lzo
  <br /># Enable Ip forwarding
  <br />nano /etc/sysctl.conf
  <br /># Uncomment the line
  <br />#net.ipv4.ip_forward=1
  <br /># Reload sysctl
  <br />sudo sysctl -p /etc/sysctl.conf
  <br /># Start openvpn<br />systemctl start openvpn@server  <- openvpn file name without the .conf 
  <br />tail -f /var/log/syslog
  <br>netstat -pna | grep LISTEN\   <- this is to check for open ports
  <br><br />Client Configuration:
  <br /># Install openvpn
  <br />apt install openvpn
  <br />cp /usr/share/doc/openvpn/examples/sample-config-files/client.confg /etc/openvpn<br /># Copy all the certs and keys to the correct location
  <br />cp /home/user/Downloads/openvpn/* /etc/openvpn/
  <br />vi /etc/openvpn/client.conf
  <br />ca ca.crt
  <br />cert clientk.crt
  <br />key clientk.key
  <br />#Specify the rest of the client configuration
  <br /># Keywork "client" must be present
  <br />remote labxx.westeurope.cloudapp.azure.com 1194
  <br />; tls-auth ta.key 1     <- comment this line
  <br />comp-lzo
  <br />cp /home/user/Downloads/openvpn/ca.crt
  <br /># Start openvpn<br />systemctl start openvpn@client<br /><br /></p>
  <br>
  <br>
  <br> To finish a iptables rule must be enforced
  <br> iptables -t nat -I POSTROUTING -s 10.8.0.0/24 -o eth0 -j MASQUERADE
  <br>
  <br>
  <br> Howto validate what IP are you using in command line?
  <br> $<i> curl -s checkip.dyndns.org | awk '{ print $NF}' | cut -d '<' -f1 </i>
<br>
  <br> by C.
    <br> $ curl ipinfo.io/ip
