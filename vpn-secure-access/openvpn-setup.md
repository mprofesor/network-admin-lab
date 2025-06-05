# OpenVPN setup

## Goal
Creating VPN using OpenVPN technology.

---

## Table of content

- [Open port for OpenVPN](#step-1---open-port-for-openvpn)
- [Quick configuration](#step-2---quick-configuration-of-openvpn)
- [VPN comparison](#step-3---comparison-of-openvpn-and-wireguard)


---

## Step 1 - open port for OpenVPN

### On the server we need to open specific port in our ufw(firewall):

```bash
sudo ufw allow 1194/udp     # OpenVPN
sudo ufw reload             # Apply new rule
sudo ufw status             # Confirm new rule
```

---

## Step 2 - quick configuration of OpenVPN

### Installation of OpenVPN and easy-rsa:

```bash
sudo apt update
sudo apt install openvpn easy-rsa
```

easy-rsa will help us with key generation for OpenVPN.

### Creating the CA(Certificate Authority)

```bash
make-cadir ~/openvpn-ca     # I'm creating new directory ~/openvpn-ca with already created scripts and boilerplates for certificates generation (easy-rsa).
cd ~/openvpn-ca             # Changing directory to the directory whrere I will be generating keys.
source vars                 # This loads all variables from vars file.
./clean-all                 # This deletes all old certificates and keys from directory keys/.
./build-ca                  # This generates private key for my CA: ca.key and certificate: ca.crt.
./build-key-server server   # This generates private key for a server: server.key and server certificate: server.crt.
./build-key client1         # This generates private key for a client: client1.key and client certificate: client1.crt 
./build-dh                  # This generates file dh2048.pem that contains Diffie-Hellman parameters used for key exchange.
```

| File          | Who is the owner  | Goal                                      |
| ------------- | ------------------| ----------------------------------------- |
| `ca.crt`      | CA                | CA Certificate (client need it)           |
| `ca.key`      | CA                | CA private key (**confidential!**)        |
| `server.crt`  | Server VPN        | Sever certificate                         |
| `server.key`  | Server VPN        | Server private key (**confidential!**)    |
| `client1.crt` | Client            | Client certificate                        |
| `client1.key` | Client            | Client private key (**confidential!**)    |
| `dh2048.pem`  | Server VPN        | For negotiating keys securely             |


### Key differences between Wireguard

| Feature                      | **WireGuard**                    | **OpenVPN + CA**                                     |
| ---------------------------- | -------------------------------- | ---------------------------------------------------- |
| Adding new client            | Just need to add a `PublicKey`   | Need a **signed certificate** by CA                  |
| When server is compromised   | Hacker can easliy add new peers  | Hacker **will not** add new clients without `ca.key` |
| Access Controll Mechanism    | Weak: list of public keys        | Strong: certificates that needs to be signed by CA   |
| Maintainance                 | Very easy                        | Needs managing the CA                                |



### Example client file `client1.ovpn`

```bash
client
dev tun
proto udp
remote YOUR_SERVER_IP 1194
resolv-retry infinite
nobind
persist-key
persist-tun
remote-cert-tls server
cipher AES-256-CBC
verb 3

<ca>
-----BEGIN CERTIFICATE-----
[ZAWARTOŚĆ ca.crt]
-----END CERTIFICATE-----
</ca>

<cert>
-----BEGIN CERTIFICATE-----
[ZAWARTOŚĆ client1.crt]
-----END CERTIFICATE-----
</cert>

<key>
-----BEGIN PRIVATE KEY-----
[ZAWARTOŚĆ client1.key]
-----END PRIVATE KEY-----
</key>
```

### Where to store this file on a client machine?

Recommended place:

```bash
mkdir -p ~/vpn/openvpn-client1
chmod 700 ~/vpn/openvpn-client1
```

Here we can store the file .ovpn or separately .crt, .key, .crt\
After all we need to change the access:

```bash
chmod 600 ~/vpn/openvpn-client1/*
```

Now only person that own the file have access to the certs and keys.

---

## Step 3 - comparison of OpenVPN and WireGuard:

| Feature                   | OpenVPN                       | WireGuard                 |
| ------------------------- | ----------------------------- | ------------------------- |
| Key types                 | Certificates (PKI)            | Public/private keys       |
| Easiness of configuration | More complicated              | Very easy                 |
| Speed                     | Slower                        | Faster  (kernel-space)    |
| Starting                  | `openvpn --config`            | `wg-quick up wg0`         |

---

## Bonus - best practices:

- Keep private keys only on the cliens inside directory with limited access.

- Use UFW to controll ports used by VPN.

- Document IP that are signed to clients (Especially if You are using WireGuard).

- Use Wireguard only if You need a speed connection. For security use OpenVPN.


