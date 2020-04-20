# kerberos-refresh

Using this in kubernetes as a sidecar container in a deployment:

```
apiVersion: extensions/v1beta1
kind: Deployment
metadata:
  name: kinit-test
spec:
  selector:
    matchLabels:
      app: kinit-test
  template:
    metadata:
      labels:
        app: kinit-test
    spec:
      containers:
      - name: kerberos-refresh
        image: anoop/kinit
        env:
        - name: APPEND_OPTIONS
          value: svc_account@domain.com
        volumeMounts:
        - name: keytab-volume
          mountPath: /krb5/krb5.keytab
          subPath: svc_account.keytab 
        - name: kerberos-cache
          mountPath: /tmp
      - name: app
        image: app:v1.1
        ports:
        - containerPort: 443
        volumeMounts:
        - name: kerberos-volume
          mountPath: /etc/krb5.conf
          subPath: krb5.conf
        - name: keytab-volume
          mountPath: svc_account.keytab 
          readOnly: true
          subPath: svc_account.keytab
        - name: kerberos-cache
          mountPath: /tmp
      volumes:
      - name: kerberos-cache
        emptyDir: {}
      - name: kerberos-volume
        configMap:
          name: kerberos
      - name: keytab-volume
        secret:
          secretName: keytab
```

