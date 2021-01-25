# LDAP

## Introduction

### What is LDAP?

**L**ightweight **D**irectory **A**ccess **P**rotocol

### What is Directory Service?

Directory Service or Name Service maps the names of network resources to their respective network addresses.

It is a shared(centralized, integrated) info. infrastructure for organizing items, resources and any other objects.

LDAP is a specifically X.500-based directory services.

LDAP runs over TCP/IP or other connection oriented transfer services

### Use-Cases

* User Authentication Management
* Address book
* Organization Management
* User Information Management
* App./System Configuration Info. Management
* Public Key Infrastructure
* DHCP, DNS
* Document Management
* Image Repository
...

### Directory Structure

[refer.](https://en.wikipedia.org/wiki/Lightweight_Directory_Access_Protocol)

* Entries
* Attributes
* Object Classes
* Object Identifiers(OIDs)

LDAP contains numbers of **Entry** in tree structure.

An **Entry** consists of a set of **Attributes**.

An **Attribute** a name(an *attribute type* or *attribute description*) and one or more values. The **Attributes** are defined in a schema.

Each **Entry** has a unique identifier; **_Distinguished Name (DN)_**.

This consists of its **_Relative Distinguished Name (RDN)_**.

A **_DN_** may change over the lifetime of the entry, for instance, when entries are moved within a tree.

To reliably and unambiguously identify **Entries**, a `UUID` might be provided in the set of the **Entry**'s operational **Attributes**.


* An Entry
```utf8
 dn: cn=John Doe,ou=department,dc=example,dc=com
 cn: John Doe
 givenName: John
 sn: Doe
 telephoneNumber: +1 888 555 6789
 telephoneNumber: +1 888 555 1232
 mail: john@example.com
 manager: cn=Barbara Doe,dc=example,dc=com
 objectClass: inetOrgPerson
 objectClass: organizationalPerson
 objectClass: person
 objectClass: top
```

> * `dn`: **_DN_** of the **Entry**
> * `cn=John Doe` is the **Entry**'s **_RDN_**
> * `dc=example,dc=com`: **_DN_** of the parent **Entry**
> * `dc`: Domain Component
> * The others: **Attributes** of the entry
>   - `cn`: Common Name
>   - `dc`: Domain Compoent
>   - `mail`: E-mail Address
>   - `sn`: Surname


* Tree
```
dc:          com
              |
           example          ## (Organisation)
          /       \
ou:  department   servers   ## (Organisational Units)
      /    \     ..
cn: ..  John Doe            ## (OU-specific data)
```

### Operations

* Basic Operations
  * Add
  * Bind(Authenticate)
  * Delete
  * Search & Compare
  * Modify
* Extended Operations
  * StartTLS
  * Abandon
  * Unbind

### URI Scheme

An LDAP uniform resource identifier (URI) scheme exists, which clients support in varying degrees, and servers return in referrals and continuation references (see RFC 4516):

```url
ldap://host:port/DN?attributes?scope?filter?extensions
```

> Most of the components described below are optional.
> * **_host_** is the FQDN or IP address of the LDAP server to search.
> * **_port_** is the network port (default port 389) of the LDAP server.
> * **_DN_** is the distinguished name to use as the search base.
> * **_attributes_** is a comma-separated list of attributes to retrieve.
> * **_scope_** specifies the search scope and can be "base" (the default), "one" or "sub".
> * **_filter_** is a search filter. For example, `(objectClass=*)` as defined in RFC 4515.
> * **_extensions_** are extensions to the LDAP URL format.
> For example, 
> `ldap://ldap.example.com/cn=John%20Doe,dc=example,dc=com` refers to all user attributes in `John Doe`'s entry in `ldap.example.com`,
>  while `ldap:///dc=example,dc=com??sub?(givenName=John)` searches for the entry in the default server (note the triple slash, omitting the host, and the double question mark, omitting the attributes). As in other URLs, special characters must be percent-encoded.

There is a similar non-standard `ldaps` URI scheme for LDAP over SSL. This should not be confused with LDAP with TLS, which is achieved using the StartTLS operation using the standard `ldap` scheme.


### Schema

The contents of the entries in a subtree are governed by a directory schema, a set of definitions and constraints concerning the structure of the directory information tree (DIT).

The schema of a `Directory Server` defines a set of rules that govern the kinds of information that the server can hold. It has a number of elements, including:

* **_Attribute Syntaxes_**—Provide information about the kind of information that can be stored in an attribute.
* **_Matching Rules_**—Provide information about how to make comparisons against attribute values.
* **_Matching Rule Uses_**—Indicate which attribute types may be used in conjunction with a particular matching rule.
* **_Attribute Types_**—Define an object identifier (OID) and a set of names that may refer to a given attribute, and associates that attribute with a syntax and set of matching rules.
* **_Object Classes_**—Define named collections of attributes and classify them into sets of required and optional attributes.
* **_Name Forms_**—Define rules for the set of attributes that should be included in the RDN for an entry.
* **_Content Rules_**—Define additional constraints about the object classes and attributes that may be used in conjunction with an entry.
* **_Structure Rule_**—Define rules that govern the kinds of subordinate entries that a given entry may have.



## Implementations

### Implementation with `OpenLDAP`

[bitnami/openldap](https://github.com/bitnami/bitnami-docker-openldap)
[list of available tags](https://hub.docker.com/r/bitnami/openldap/tags/?page=1&ordering=last_updated)


* Pull an image
```bash
docker pull bitnami/openldap:latest
```

* Build
```bash
$ docker build -t bitnami/openldap:latest 'https://github.com/bitnami/bitnami-docker-openldap.git#master:2/debian-10'
```

### Deploy OpenLDAP server in k8s

https://docs.bitnami.com/tutorials/create-openldap-server-kubernetes/


* Create a Deployment and a Service


```yml
kubectl apply -f - <<EOF
apiVersion: apps/v1
kind: Deployment
metadata:
  name: openldap
  labels:
    app.kubernetes.io/name: openldap
spec:
  selector:
    matchLabels:
      app.kubernetes.io/name: openldap
  replicas: 1
  template:
    metadata:
      labels:
        app.kubernetes.io/name: openldap
    spec:
      containers:
        - name: openldap
          image: docker.io/bitnami/openldap:latest
          imagePullPolicy: "Always"
          env:
            - name: LDAP_ROOT
              value: "dc=example,dc=org"
            - name: LDAP_ENABLE_TLS
              value: "no"
              # value: "yes"
            - name: LDAP_PORT_NUMBER
              value: "1389"
            - name: LDAP_LDAPS_PORT_NUMBER
              value: "1636"
            - name: LDAP_ADMIN_USERNAME
              value: "admin"
            - name: LDAP_ADMIN_PASSWORD
              value: "adminpassword"
              # valueFrom:
              #   secretKeyRef:
              #     key: adminpassword
              #     name: openldap
            - name: LDAP_USERS
              valueFrom:
                secretKeyRef:
                  key: users
                  name: openldap
            - name: LDAP_PASSWORDS
              valueFrom:
                secretKeyRef:
                  key: passwords
                  name: openldap
          ports:
            - name: tcp-ldap
              containerPort: 1389
            - name: tcp-ldaps
              containerPort: 1636
---
apiVersion: v1
kind: Service
metadata:
  name: openldap
  labels:
    app.kubernetes.io/name: openldap
spec:
  type: ClusterIP
  ports:
    - name: tcp-ldap
      port: 1389
      targetPort: tcp-ldap
      # targetPort: 1389
    - name: tcp-ldaps
      port: 1636
      targetPort: tcp-ldaps
      # targetPort: 1636
  selector:
    app.kubernetes.io/name: openldap
EOF
```

* Create a Secret for openldap

```bash
kubectl create secret generic openldap \
  --from-literal=adminpassword=adminpassword \
  --from-literal=users=user01,user02 \
  --from-literal=passwords=password01,password02
```

### Browse with `Apache Directory Studio` in local

[Download link](https://directory.apache.org/studio/downloads.html)

New -> LDAP Browser -> LDAP Connection

* Hostname: `{LDAP_HOST}`
* Port: `{LDAP_PORT}`
* Bind DN or User: `cn=admin,dc=example,dc=org`
* Bind Password: `{DN_PASSWORD}`


```bash
# Get secret info.:
kubectl get secrets openldap -o jsonpath="{.data.adminpassword}" | base64 -d
# Access to running container(`openldap`):
kubectl -n default exec --stdin --tty "$(kubectl -n default get pods -l app.kubernetes.io/name=openldap -o jsonpath="{.items[0].metadata.name}")" -- /bin/bash
# Query in `openldap`:
ldapsearch -h localhost -p 1389 -s sub -D "cn=admin,dc=example,dc=org" -w adminpassword -b "ou=users,dc=example,dc=org" "objectclass=*"

```

### Implementation with `FusionDirectory`


### Deploy `openldap-fusiondirectory`


```yml
kubectl apply -f - <<EOF
apiVersion: apps/v1
kind: Deployment
metadata:
  name: openldap-fd
  labels:
    app.kubernetes.io/name: openldap-fd
spec:
  selector:
    matchLabels:
      app.kubernetes.io/name: openldap-fd
  replicas: 1
  template:
    metadata:
      labels:
        app.kubernetes.io/name: openldap-fd
    spec:
      containers:
        - name: openldap-fd
          image: docker.io/tiredofit/openldap-fusiondirectory:latest
          imagePullPolicy: "Always"
          env:
            - name: DOMAIN
              value: "example.org"
            - name: ADMIN_PASS
              value: "admin"
            - name: CONFIG_PASS
              value: "config"
            - name: ENABLE_READONLY_USER
              value: "true"
            - name: READONLY_USER_USER
              value: "readonly"
            - name: READONLY_USER_PASS
              value: "readonly"
            - name: SCHEMA_TYPE
              value: "nis"
              # value: "rfc2307bis"
            - name: FUSIONDIRECTORY_ADMIN_USER
              value: "fd-admin"
            - name: FUSIONDIRECTORY_ADMIN_PASS
              value: "admin"
            - name: ORGANIZATION
              value: "Example Organization"
          ports:
            - name: tcp-ldap
              containerPort: 389
            - name: tcp-ldaps
              containerPort: 636
---
apiVersion: v1
kind: Service
metadata:
  name: openldap-fd
  labels:
    app.kubernetes.io/name: openldap-fd
spec:
  type: ClusterIP
  ports:
    - name: tcp-ldap
      port: 389
      targetPort: tcp-ldap
      # targetPort: 1389
    - name: tcp-ldaps
      port: 636
      targetPort: tcp-ldaps
      # targetPort: 1636
  selector:
    app.kubernetes.io/name: openldap-fd
EOF

kubectl -n default rollout restart deployment openldap-fd
```

#### Deploy `fusiondirectory` for Browsing

```bash
# https://hub.docker.com/r/tiredofit/fusiondirectory
docker pull tiredofit/fusiondirectory
```


```yml
kubectl apply -f - <<EOF
apiVersion: apps/v1
kind: Deployment
metadata:
  name: fusiondirectory
  labels:
    app.kubernetes.io/name: fusiondirectory
spec:
  selector:
    matchLabels:
      app.kubernetes.io/name: fusiondirectory
  replicas: 1
  template:
    metadata:
      labels:
        app.kubernetes.io/name: fusiondirectory
    spec:
      containers:
        - name: fusiondirectory
          image: docker.io/tiredofit/fusiondirectory:latest
          imagePullPolicy: "Always"
          env:
            - name: LDAP_DEFAULT
              value: $LDAP1_NAME
            - name: LDAP1_NAME
              value: openldap-bitnami
            - name: LDAP1_HOST
              # value: ldap://openldap.default.svc.cluster.local:1389
              value: "openldap.default.svc.cluster.local"
            - name: LDAP1_TLS
              value: "false"
            - name: LDAP1_SSL
              value: "false"
            - name: LDAP1_PORT
              value: "1389"
            - name: LDAP1_ADMIN_PASS
              value: adminpassword
              # valueFrom:
              #   secretKeyRef:
              #     key: adminpassword
              #     name: openldap
            - name: LDAP1_ADMIN_DN
              value: "cn=admin,dc=example,dc=org"
            - name: LDAP1_BASE_DN
              value: "dc=example,dc=org"
            - name: LDAP2_NAME
              value: openldap-fd
            - name: LDAP2_HOST
              # value: ldap://openldap.default.svc.cluster.local:1389
              value: "openldap-fd.default.svc.cluster.local"
            - name: LDAP2_TLS
              value: "false"
            - name: LDAP2_SSL
              value: "false"
            - name: LDAP2_PORT
              value: "389"
            - name: LDAP2_ADMIN_PASS
              value: admin
              # valueFrom:
              #   secretKeyRef:
              #     key: adminpassword
              #     name: openldap
            - name: LDAP2_ADMIN_DN
              value: "cn=admin,dc=example,dc=org"
            - name: LDAP2_BASE_DN
              value: "dc=example,dc=org"
          ports:
            - name: http
              containerPort: 80
---
apiVersion: v1
kind: Service
metadata:
  name: fusiondirectory
  labels:
    app.kubernetes.io/name: fusiondirectory
spec:
  type: ClusterIP
  ports:
    - name: http
      port: 8080
      targetPort: 80
  selector:
    app.kubernetes.io/name: fusiondirectory
EOF

kubectl -n default rollout restart deployment fusiondirectory
```


```bash
# Port-forwarding `fusiondirectory`:
kubectl -n default port-forward "$(kubectl get pods -n default -l "app.kubernetes.io/name=fusiondirectory" -o jsonpath="{.items[0].metadata.name}")" 8081:80

# Access to running container(`openldap-fd`):
kubectl -n default exec --stdin --tty "$(kubectl -n default get pods -l app.kubernetes.io/name=openldap-fd -o jsonpath="{.items[0].metadata.name}")" -- /bin/bash
# Query in `openldap-fd`:
ldapsearch -h localhost -p 389 -s sub -D "cn=admin,dc=example,dc=org" -w adminpassword -b "ou=users,dc=example,dc=org" "objectclass=*"


# Access to running container(`fusiondirectory`):
kubectl -n default exec --stdin --tty "$(kubectl -n default get pods -l app.kubernetes.io/name=fusiondirectory -o jsonpath="{.items[0].metadata.name}")" -- /bin/bash
# Query in `openldap-fusiondirectory`:
ldapsearch -h openldap-fd.default.svc.cluster.local -p 389 -s sub -D "cn=admin,dc=example,dc=org" -w admin -b "dc=example,dc=org" "objectclass=*"

```
