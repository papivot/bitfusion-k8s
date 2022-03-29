# Bitfusion integration with Kubernetes

## Steps

* Enable the Bitfusion server and perform the configurations within vSphere
* Modify the Docckerfile by adding these lines - 

```cmd
######################################
# Start of bitfusion

RUN apt-get install -y --no-install-recommends \
        wget uuid libjsoncpp1 librdmacm1 libssl-dev libibverbs1 libnuma1 libcapstone3 libnl-3-200 libnl-route-3-200 open-vm-tools && \
    rm -rf /var/lib/apt/lists/*

RUN cd /tmp && \
    curl -fSslL -O https://packages.vmware.com/bitfusion/ubuntu/20.04/bitfusion-client-ubuntu2004_4.5.0-4_amd64.deb && \
    apt-get install -y ./bitfusion-client-ubuntu2004_4.5.0-4_amd64.deb && \
    rm -rf /tmp/*
########################################
```

* Validate that the relevent secrets are present in the namespace where the app is getting deployed - 
```cmd
kubectl get secrets -n k8s-papivot-tools

NAME                                   TYPE                                  DATA   AGE
bitfusion-client-secret-ca.crt         Opaque                                1      13h
bitfusion-client-secret-client.yml     Opaque                                1      13h
bitfusion-client-secret-servers.conf   Opaque                                1      13h
```


* Add the following entries to the deployment yaml

```yaml
        volumeMounts:
        - mountPath: /tmp/scratch
          name: cache-volume
        - mountPath: /etc/bitfusion
          name: config-files
        - mountPath: /etc/bitfusion/tls
          name: certificate
...
      volumes:
      - name: config-files
        projected:
          defaultMode: 0640
          sources:
          - secret:
              name: bitfusion-client-secret-client.yml
          - secret:
              name: bitfusion-client-secret-servers.conf
      - name: certificate
        secret:
          secretName: bitfusion-client-secret-ca.crt
          defaultMode: 0640
```

