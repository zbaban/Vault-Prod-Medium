# Vault-Prod-Medium

 This is a companion repo to [Deploy a Production Ready Hashicorp Vault Node](https://zaidbaban.medium.com/deploy-a-production-ready-hashicorp-vault-node-1563623956c9) article on Medium.
 

### Step 1 - Install Hashicorp Consul
```
wget https://releases.hashicorp.com/consul/1.9.4/consul_1.9.4_linux_amd64.zip
unzip consul_1.9.4_linux_amd64.zip
ls
sudo mv consul /usr/bin/
consul version
```

### Step 2 - Configure Consul to run as a systemd service
```
sudo vim /etc/systemd/system/consul.service
```
paste the following in the file:
```
[Unit]
Description=Consul
Documentation=https://www.consul.io/
[Service]
ExecStart=/usr/bin/consul agent -server -ui -data-dir=/temp/consul -bootstrap-expect=1 -node=vault -bind=SERVER-PRIVATE-IP-ADDRESS -config-dir=/etc/consul.d/
ExecReload=/bin/kill -HUP $MAINPID
LimitNOFILE=65536
[Install]
WantedBy=multi-user.target
```
insert your private ip address in bind and press :wq to save.
```
sudo mkdir /etc/consul.d
sudo vim /etc/consul.d/ui.json
```
paste the following in the file:
```
{
"addresses": {
"http": "0.0.0.0"
    }
}
```
```
sudo systemctl daemon-reload
sudo systemctl start consul
sudo systemctl status consul
sudo systemctl enable consul
sudo journalctl -f -u consul
```
### Step 3 - Obtain a free TLS/SSL to encrypt Vault
```
sudo apt-get update
sudo apt-get install software-properties-common
sudo add-apt-repository universe
sudo apt-get update
sudo apt-get install certbot
sudo certbot certonly - standalone -d <YOUR DOMIN>
sudo ls /etc/letsencrypt/live/DOMAIN
sudo certbot renew
```
### Step 4 - Install Vault
```
wget https://releases.hashicorp.com/vault/1.7.0-rc1/vault_1.7.0-rc1_linux_amd64.zip
unzip vault_1.7.0-rc1_linux_amd64.zip
ls
sudo mv vault /usr/bin/
vault version
```

### Step 5 - Configure Vault
```
sudo mkdir /etc/vault
sudo vim /etc/vault/config.hcl
```
paste the following in the file:
```
storage "consul" {
address = "127.0.0.1:8500"
path = "vault/"
}
listener "tcp" {
address = "0.0.0.0:443"
tls_disable = 0
tls_cert_file = "/etc/letsencrypt/live/<DOMAIN>/fullchain.pem"
tls_key_file = "/etc/letsencrypt/live/<DOMAIN>/privkey.pem"
}
ui = true
```
replace <DOMAIN> with your domin and press :wq to save.
```
sudo vim /etc/systemd/system/vault.service
```
```
[Unit]
Description=Vault
Documentation=https://www.vault.io/
[Service]
ExecStart=/usr/bin/vault server -config=/etc/vault/config.hcl
ExecReload=/bin/kill -HUP $MAINPID
LimitNOFILE=65536
[Install]
WantedBy=multi-user.target
```
```
sudo systemctl daemon-reload
export VAULT_ADDR="https://DOMAIN:443"
echo "export VAULT_ADDR="https://DOMAIN:443" >> ~/.bashrc"
sudo systemctl start vault
sudo systemctl enable vault
sudo systemctl status vault
```

### Step 6 - Initialize and unseal Vault
```
sudo vault -autocomplete-install
complete -C /usr/bin/vault vault
vault operator init
```
