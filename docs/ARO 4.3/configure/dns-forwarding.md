# Configure DNS Forwarding on an ARO 4.3 Cluster

You will need to modify the DNS Operator in ARO 4.3 in order for your application pods running inside the ARO 4.3 cluster to resolve names hosted on a private DNS server outside the cluster.  These steps are documented for OpenShift 4.3 [here](https://docs.openshift.com/container-platform/4.3/networking/dns-operator.html).

For example, if you want to forward all DNS requests for *.example.com to be resolved by a DNS server 192.168.100.10, you can edit the operator configuration by running:
 
```bash
oc edit dns.operator/default
```
 
This will launch an editor and you can replace `spec: {}` with:
 
 ```yaml
spec:
  servers:
  - forwardPlugin:
      upstreams:
      - 192.168.100.10
    name: example-dns
    zones:
    - example.com
```

Save the file and exit your editor.

To quickly validate the new DNS forwarding setup, you can do the following:

```bash

# Test the DNS resolution
oc run --generator=run-pod/v1 -it --rm aro-ssh --image=debian
# Once you are in the interactive session, execute the following commands (replace the FQDN with yours)
apt-get update
apt-get install dnsutils -y
nslookup REPLACE.YOURDOMAIN.corp.
# Output should looklike:
# Server:         172.30.0.10
# Address:        172.30.0.10#53

# Non-authoritative answer:
# Name:   REPLACE.YOURDOMAIN.corp
# Address: 10.165.5.4

# Terminate the session
exit

```
