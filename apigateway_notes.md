# pkcs-proxy with APIGateway

## Proxy params

in `/etc/environment`

```shell
export PKCS11_PROXY_SOCKET=tcp://10.8.0.7:54435
```

## JNA and temp dir

JNA needs exec on temp dir. If not:
in `<GATEWAY_DIR>/system/conf/jvm.xml`

```xml
[...]
    <!-- JNA: /tmp is noexec -->
    <VMArg name="-Djna.tmpdir=/var/tmp"/>
[...]
```

## Init slot

```shell
echo 0:/var/lib/softhsm/slot0.db > /etc/softhsm2.conf && softhsm2-util --init-token --pin 123456 --so-pin 12345678 --slot 0 --label SoftHSMToken
```

## Convert priv key to PKCS#8

```shell
openssl pkcs8 -topk8 -in /tmp/wildcard.semperpax.local.key -out /tmp/wildcard.semperpax.local.p8.key -nocrypt
```

## import into SoftHSM

```shell
softhsm2-util --slot 1519744399 --import /tmp/wildcard.semperpax.local.p8.key --label local_wildcard_key --id AA01
```

## define Certificate realm (= link to a private key)

```shell
$ ./keystoreadmin
----------------------------------------------------
              Key Store Administration
----------------------------------------------------

Managing local key stores for:

   Host: ansible-managed-api-gateway.semperpax.local
   Group: BACK
   Instance: BACK-APIGW1

Choose from one of the following:

   Local group instance:

       1) Change group or instance

   HSM provider administration:

       2) List registered HSM providers
       3) Register an HSM provider

   Certificate realm administration:

       4) List Certificate Realms
       5) Create a Certificate Realm

   q) Quit

Select option: 5
Create a Certificate Realm...
Certificate realm name: apigateway

Choose from one of the following:

   1) SoftHSM

Select option: 1
Initializing HSM...
SoftHSM                         , libraryDescription: Implementation of PKCS11        , libraryVersion: major = 2 minor = 4
Getting HSM list of slots...

Choose from one of the following:

   1) 1519744399
   2) 1

Select option: 1
PKCS11 slot PIN (no echo): 
Logging in to HSM...

Choose from one of the following:

   1) local_wildcard_key

   q) Quit

Select option: 1
Certificate realm filename [apigateway.ks]: 
Successfully created the certificate realm: apigateway
Press any key to continue...
```

## Use in Policy Studio / Configuration Studio

![Policy Studio](https://linx.semperpax.com/selif/3eshyn2f.png)

## Start API Gateway with automatic PIN passphrase

```shell
vi <GATEWAY_DIR>/groups/group-2/instance-1/conf/vpkcs11.xml
```

```xml
 [...]
          <EngineCommand  when='preInit'
                            name='REALMS_DIR'
                            value='$VINSTDIR/conf/certrealms'/>

            <EngineCommand when="preInit" name="PASSPHRASE_EXEC"
                 value="$VDISTDIR/hsmpin.sh" />

[...]
```
