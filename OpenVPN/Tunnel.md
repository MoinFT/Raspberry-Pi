# OpenVPN

## Structure

1. [Installing OpenVPN-Server and Easy-RSA](#installing-openvpn-server-and-easy-rsa)
    1. [Create the srcipt to create the server certificates](#create-the-srcipt-to-create-the-server-certificates)
    2. [Create the srcipt to create the server configuration](#create-the-srcipt-to-create-the-server-configuration)
2. [Create OpenVPN server certificates and configuration](#create-openvpn-server-certificates-and-configuration)
3. [Configuring the firewall to do NAT](#configuring-the-firewall-to-do-nat-network-address-translation)
4. [Configuring the Firewall to access the OpenVPN-Server](#configuring-the-firewall-to-access-the-openvpn-server)
5. [Systemctl Options for the OpenVPN-Server](#systemctl-options-for-the-openvpn-server)
6. [Create Open VPN Client configuration](#create-open-vpn-client-configuration)
    1. [Create the srcipt to create the Open VPN .OVPN file](#create-the-srcipt-to-create-the-open-vpn-ovpn-file-only-one-time)
    2. [Create the Open VPN .OVPN file](#create-the-open-vpn-ovpn-file)

## Installing OpenVPN-Server and Easy-RSA

1. Installing OpenVPN

    ```shell
    sudo apt-get update && sudo apt-get install openvpn && sudo apt-get install easy-rsa
    ```

2. Creating a directory for easy-rsa

    ```shell
    mkdir ~/easy-rsa
    ln -s /usr/share/easy-rsa/* ~/easy-rsa/
    ```

### Create the srcipt to create the server certificates

```shell
cd ~/
nano makeOpenVPN_Server_Certificates.sh
```

Insert the following code into the file:

```bash
#!/bin/bash

# 1 argument = Server_identifier
cd ~/easy-rsa/

./easyrsa init-pki
echo "The passphrase must be longer then 4 characters."
./easyrsa build-ca

sleep 1

echo "The passphrase must be longer then 4 characters."
./easyrsa build-server-full ${1}

sleep 1

./easyrsa sign-req server ${1}

sleep 1

./easyrsa gen-dh

cd pki/
openvpn --genkey tls-crypt-v2-server private/${1}.pem
```

### Create the srcipt to create the server configuration

```shell
cd ~/
nano makeOpenVPN_Server_Configuration.sh
```

Insert the following code into the file:

```bash
#!/bin/bash

# 1 argument = Server_identifier
# 2 argument = Username

read -p "Server IP-Address: (e.g. 192.168.178.10)" serverAddress
read -p "VPN server port: " port

echo ""

read -p "VPN Network Address: (e.g. 192.168.100.0)" vpnAddress

IFS='.'
read -a serverAddressArray <<< "$serverAddress"
serverNetworkAddress="${serverAddressArray[0]}.${serverAddressArray[1]}.${serverAddressArray[2]}"

IFS='.'
read -a vpnAddressArray <<< "$vpnAddress"
vpnNetworkAddress="${vpnAddressArray[0]}.${vpnAddressArray[1]}.${vpnAddressArray[2]}"

cd /etc/openvpn/server

cat <(echo -e '#--------------------') \
<(echo -e "local ${serverAddress}") \
<(echo -e "") \
<(echo -e "#VPN port") \
<(echo -e "port $port") \
<(echo -e "") \
<(echo -e "#VPN over UDP ") \
<(echo -e "proto udp") \
<(echo -e "") \
<(echo -e "# 'dev tun' will create a routed IP tunnel") \
<(echo -e "dev tun ") \
<(echo -e "") \
<(echo -e "ca ca.crt") \
<(echo -e "cert ${1}.crt") \
<(echo -e "key ${1}.key") \
<(echo -e "tls-crypt-v2 ${1}.pem") \
<(echo -e "dh dh.pem") \
<(echo -e "") \
<(echo -e "# Network for the VPN") \
<(echo -e "server ${vpnNetworkAddress}.0 255.255.255.0 ") \
<(echo -e "ifconfig ${vpnNetworkAddress}.1 ${vpnNetworkAddress}.2") \
<(echo -e "") \
<(echo -e "# Communicate route to home network with all clients") \
<(echo -e "push 'route ${vpnNetworkAddress}.1 255.255.255.255'") \
<(echo -e "push 'route ${vpnNetworkAddress}.0 255.255.255.0'") \
<(echo -e "push 'route ${serverAddress} 255.255.255.0'") \
<(echo -e "push 'route ${serverNetworkAddress}.1 255.255.255.0'") \
<(echo -e "push 'route ${serverNetworkAddress}.0 255.255.255.0'") \
<(echo -e "") \
<(echo -e "# Communicate DNS server to clients") \
<(echo -e "push 'dhcp-option DNS ${serverNetworkAddress}.1'") \
<(echo -e "push 'dhcp-option DNS 1.1.1.1'") \
<(echo -e "push 'dhcp-option DNS 1.0.0.1'") \
<(echo -e "") \
<(echo -e "push 'redirect-gateway def1 bypass-dhcp'") \
<(echo -e "") \
<(echo -e "# Maintain a record of client <-> virtual IP address") \
<(echo -e "") \
<(echo -e "# associations in this file.") \
<(echo -e "ifconfig-pool-persist /var/log/openvpn/ipp.txt") \
<(echo -e "") \
<(echo -e "# Ping every 10 seconds and assume client is down if") \
<(echo -e "# it receives no response in 120 seconds.") \
<(echo -e "keepalive 10 120") \
<(echo -e "") \
<(echo -e "#cryptographic cipher") \
<(echo -e "cipher AES-256-GCM") \
<(echo -e "") \
<(echo -e "#avoid accessing certain resources on restart") \
<(echo -e "persist-key") \
<(echo -e "persist-tun ") \
<(echo -e "") \
<(echo -e "#log of current connections") \
<(echo -e "status /var/log/openvpn/openvpn-status.log") \
<(echo -e "") \
<(echo -e "#log verbose level (0-9)") \
<(echo -e "verb 4") \
<(echo -e "") \
<(echo -e "askpass password.txt") \
<(echo -e "") \
<(echo -e "# Notify the client when the server restarts") \
<(echo -e "explicit-exit-notify 1") \
<(echo -e "#-----------------------------------------") \
> server.conf

read -p "Enter Server Password:" serverPassword

cat <(echo -e "${serverPassword}") \
> password.txt

if [ "${2}" == "root" ]; then
    cd /root/easy-rsa/pki/
else
    cd /home/${2}/easy-rsa/pki/
fi

cp ca.crt dh.pem /etc/openvpn/server/

cd private/
cp ${1}.key ${1}.pem /etc/openvpn/server/
    
cd ../issued/
cp ${1}.crt /etc/openvpn/server/

sysctl -w net.ipv4.ip_forward=1
```

## Create OpenVPN server certificates and configuration

1. Execute the file `makeOpenVPN_Server_Certificates.sh`

    ```shell
    cd ~/
    bash makeOpenVPN_Server_Certificates.sh [Server-Name]
    ```

2. Execute the file `makeOpenVPN_Server_Configuration.sh`

    ```shell
    sudo bash makeOpenVPN_Server_Configuration.sh [Server-Name] [Username]
    ```

## Configuring the firewall to do NAT (Network Address Translation)

1. Install a firewall

    ```shell
    sudo apt-get install ufw
    ```

2. Show the default route list and note the interface for your OpenVPN route

    ```shell
    ip route list default
    ```

3. Entering NAT and Redirect Rules for Open VPN

    ```shell
    sudo nano /etc/ufw/before.rules
    ```

    Add the following lines to the before.rules:

    ```shell
    *nat
    :POSTROUTING ACCEPT [0:0]
    -A POSTROUTING -s [VPN-Address].0/24 -o eth0 -j MASQUERADE
    COMMIT
    ```

4. Change the `DEFAULT_FORWARD_POLICY`

    ```shell
    sudo nano /etc/default/ufw
    ```

    Change there the `DEFAULT_FORWARD_POLICY` from `DROP` to `ACCEPT`

## Configuring the Firewall to access the OpenVPN-Server

Allow access over port 788 with the udp protocol:

```shell
sudo ufw allow 788/udp
sudo ufw allow 22
```

Restart the firewall to activate the changes:

```shell
sudo ufw disable && sudo ufw enable
```

## Systemctl Options for the OpenVPN-Server

- Start:

    ```shell
    sudo systemctl start openvpn-server@server.service
    ```

- Status:

    ```shell
    sudo systemctl status openvpn-server@server.service
    ```

- Restart:

    ```shell
    sudo systemctl restart openvpn-server@server.service
    ```

- Stop:

    ```shell
    sudo systemctl stop openvpn-server@server.service
    ```

- Enable start on boot:

    ```shell
    sudo systemctl enable openvpn-server@server.service
    ```

- Disable start on boot:

    ```shell
    sudo systemctl disable openvpn-server@server.service
    ```

## Create Open VPN Client configuration

### Create the srcipt to create the Open VPN .OVPN file **(Only one time)**

```shell
nano makeOpenVPN_Client.sh
```

Insert the following code into the file:
**Note: Insert your Server-Address and Server-Port**

```bash
#!/bin/bash

# 1 argument = Client_identifier
# 2 argument = Server_identifier
cd ~/easy-rsa/
./easyrsa gen-req ${1}
./easyrsa sign-req client ${1}

cd ~/easy-rsa/pki/
openvpn --tls-crypt-v2 private/${2}.pem --genkey tls-crypt-v2-client private/${1}.pem

mkdir ~/vpn_clients 2>/dev/null
cd ~/vpn_clients

cat <(echo -e 'client') \
<(echo -e 'proto udp') \
<(echo -e 'dev tun') \
<(echo -e 'remote [Server-Address] [Server-Port]') \
<(echo -e 'resolv-retry infinite') \
<(echo -e 'nobind') \
<(echo -e 'persist-key') \
<(echo -e 'persist-tun') \
<(echo -e 'remote-cert-tls server') \
<(echo -e 'cipher AES-256-GCM') \
<(echo -e '#user nobody') \
<(echo -e '#group nobody') \
<(echo -e 'verb 3') \
    <(echo -e '<ca>') \
    ~/easy-rsa/pki/ca.crt \
    <(echo -e '</ca>\n<cert>') \
    ~/easy-rsa/pki/issued/${1}.crt \
    <(echo -e '</cert>\n<key>') \
    ~/easy-rsa/pki/private/${1}.key \
    <(echo -e '</key>\n<tls-crypt-v2>') \
    ~/easy-rsa/pki/private/${1}.pem \
    <(echo -e '</tls-crypt-v2>') \
    > ${1}.ovpn
```

```shell
chmod +x makeOpenVPN_Client.sh
```

Explaining the configuration lines

- “client” indicates that it is an Open VPN client

- “proto udp” indicates that it will use the UDP protocol

- “dev tun” indicates that it will use IP tunnel

- “remote” [Server-Address] [Server-Port]″ indicates the IP of the OpenVPN-Server and the port that will be used

- “resolv-retry infinite” indicates that it will keep trying to resolve the VPN server name

- “nobind” indicates that it will not use a specific port

- “persist-key” and “persist-tun” allows you to preserve connection state in case of restart

- “remote-cert-tls server” indicates the server’s tls

- “cipher AES-256-GCM” indicates the encryption cipher used

- “#user nobody” and “#group nobody” indicates reduced privileges on non-Linux clients
- Remove the “#” if using a client on a machine that does not use the Linux operating system.

- “verb 3” indicates the verbosity of the logs

- The rest of the script indicates that we are going to read the files “`$`{1}.crt”, “`$`{1}.key” and “`$`{1}.pem” , where “`$`{1}” is the identification name you assigned to the client. This client identifier is the first argument that we will pass to our script.

### Create the Open VPN .OVPN file

```shell
./makeOpenVPN_Client.sh [Client-Name] [Server-Name]
```

Get the `.ovpn-File` from the OpenVPN-Server via sFTP on your Client and import the settings in OpenVPN-Connect.
