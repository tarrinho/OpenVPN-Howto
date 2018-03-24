# OpenVPN-Howto
OpenVPN - Howto create and use (server client)

# Thank to C
Install VPN:
    Key Setup:
        # Install the openvpn
        sudo apt install openvpn easy-rsa
        # Setting up our CA
        # Make a dir where our keys will be
        mkdir /etc/openvpn/easy-rsa/
        # Copy all the keys to the new folder
        cp -r /usr/share/easy-rsa/* /etc/openvpn/easy-rsa/
        # Edit our VPN variables
        nano /etc/openvpn/easy-rsa/vars
            export KEY_COUNTRY="NL"
            export KEY_PROVINCE="NH"
            export KEY_CITY="Amsterdam"
            export KEY_ORG="Shabadabadooo"
            export KEY_EMAIL="hackaroo@shabadabadoo.com"
            export KEY_CN=VPN
            export KEY_ALTNAMES=WEBCAFE
            export KEY_NAME=VPN
            export KEY_OU=VPN
        # Generate CA cert and keys
        cd /etc/openvpn/easy-rsa/
        source vars
        ./clean-all
        ./build-ca
        # Generate server cert and keys (and reply yes to all)
        ./build-key-server raygun
        # Build Diffie Hellman parameters
        ./build-dh
        # Copy our keys to openvpn folder
        cd keys/
        cp raygun.crt raygun.key ca.crt dh2048.pem /etc/openvpn

       # Generate client certificates
       cd /etc/openvpn/easy-rsa/
       source vars
       ./build-key clientk
       # Copy them to our computer
       scp username@hostname:/etc/openvpn/easy-rsa/ca.crt /home/user/Downloads/openvpn/ca.crt
       scp username@hostname:/etc/openvpn/easy-rsa/clientk.crt /home/user/Downloads/openvpn/clientk.crt
       scp username@hostname:/etc/openvpn/easy-rsa/clientk.key /home/user/Downloads/openvpn/clientk.key
       # After the copy is done, remove the keys from the server
       rm /etc/openvpn/easy-rsa/clientk.*
   
   Server Configuration:
        # Unroll the configurations
        sudo cp /usr/share/doce/openvpn/examples/sample-config-files/server.conf.gz /etc/openvpn
        sudo gzip -d /etc/openvpn/server.conf.gz
        # Edit the configuration
        nano /etc/openvpn/server.conf
        # Point to the correct certs and keys
        ca ca.crt
        cert raygun.crt
        key raygun.crt
        dh dh2048.pem
        # Enable Ip forwarding
        nano /etc/sysctl.conf
        # Uncomment the line
        #net.ipv4.ip_forward=1
        # Reload sysctl
        sudo sysctl -p /etc/sysctl.conf
        # Start openvpn
        systemctl start openvpn@server
        
    Client Configuration:
        # Install openvpn
        apt install openvpn
        cp /usr/share/doc/openvpn/examples/sample-config-files/client.confg /etc/openvpn
        # Copy all the certs and keys to the correct location
        cp /home/user/Downloads/openvpn/* /etc/openvpn/
        nano /etc/openvpn/client.conf
        ca ca.crt
        cert clientk.crt
        key clientk.key
        #Specify the rest of the client configuration
        # Keywork "client" must be present
        remote labxx.westeurope.cloudapp.azure.com 1194
        cp /home/user/Downloads/openvpn/ca.crt
        # Start openvpn
        systemctl start openvpn@client
        
