# Fujitsu Web of Things Gateway

## Main features
- Intermediates access between internet applications and local WoT devices
- Aggregates local devices and provides single access point for applications

## Overview
In the typical IoT systems, IoT devices are installed in local networks.
These devices are not directly accessible from the Internet.

In the WoT arhitecture, when an application uses a WoT device (Thing), the application needs to consume a TD of the device (reference: [WoT architecture 6.1](https://www.w3.org/TR/wot-architecture/#sec-architecture-overview)).

If the device is installed in the local network, any proxy mechanism is needed to communicates between the application and the device. WoT architecture defines it's a "[intermediary](https://www.w3.org/TR/wot-architecture/#indirect-communication)".

In this case, the application cannot consume the original TD of the device.
Because the URIs in the TD points at the local address of the device. 
Therefore, another TD is required that has similar information of the original TD, but its URIs point at the intermediary.
This TD is called shadow TD.

In short, to use a local device from an application, the application needs to acquire a shadow TD of the device and acceces the device through intermediaries by consuming the shadow TD.

To achive this, this gateway provides three functions.
- Remote and local intermediary (reference: [WoT architecture 9.2.3](https://www.w3.org/TR/wot-architecture/#sec-deployment-cloud))
- Shadow TD generation
- TD directory

By using this gateway, applications can obtain shadow TDs of local devices in multiple fields from a single gateway (remote gateway), and use these devices from the gateway.

In addition, the remote gateway can aggregate remote WoT devices that are already exposed to the internet.
Therefore, applications can use both local devices and remote devices from one entry point (remote gateway).

![Typical IoT system](images/Typical%20IoT%20system.svg "Fig. 1 Typical IoT system")

## License

[APACHE LICENSE, VERSION 2.0](https://www.apache.org/licenses/LICENSE-2.0)

## Prerequisites

### Build

- Java 8
- Maven 3.3.9 or later

### Execute

- Java 8

## How to build

### Build jar file

```
$ git clone <Project name in GitHub>
$ cd <Clone dir>/com.fujitsu.wot.gateway
$ mvn package
```

<font color="Red">【削除】</font>[Download prebuild jar file](https://github.labs.fujitsu.com/wot-oss/WoT-GW/releases)

## How to execute
### Auto setup execution environment

```
$ cd <Clone dir>
$ ./setup.sh
$ cd run
```
<font color="Red">【削除】</font>[Download zip file](https://github.labs.fujitsu.com/wot-oss/WoT-GW/releases)

See [Appendix A](#a-manual-setup) for manual setup

### Configuration

#### Port settings

初期値は80．変更する場合には、以下のファイルを編集。
Edit `configuration/config.ini`

```
jetty.http.port=<HTTP server port>
```

NOTE: Default port (80) would require root authority.

#### Gateway configuration

ゲートウェイのURL等を変更する場合には、以下のファイルに修正部分を記述する？
もしくは、起動時のオプションとして-Dに記述する

a) Create config file  
  `configuration/gateway.properties`

b) Use -D option  
  `$ java -Dcore.baseURL=...`

Configuration options
```
core.baseURL=<Base url for shadow TDs> # default = http://localhost
core.authType=<Additional authentication type ("basic" or empty)>
core.authValue=<Authentication option (ex. user:pass for basic auth)>
core.authName=<Additional authentication scheme name> # default = __auto_added_sc__
websocket.serverURL=<URL for cascade connection (ex. ws://<server>/websocket/)>
websocket.keepAliveInterval=<Keep-alive interval for cascade connection [ms]> # default = 20000
websocket.connectionTimeout=<Connection timeout for cascade connection [ms]> # default = 30000
websocket.reconnectInterval=<Reconnect interval for cascade connection [ms]> # default = 10000
websocket.allowInsecureSSL=<Allows insecure wss for cascade connection ("true" or empty)>
binding.http.allowInsecureSSL=<Allows insecure https connection for WoT http binding ("true" or empty)>
binding.http.requestTimeout=<Request timeout for WoT http binding [ms]> # default 10000
```
See ["How to use: Add authentication"](#b-add-authentication) for core.authXXX options.

### Run

```
$ java -jar org.eclipse.osgi-3.15.100.jar
```

以下のオプションの意味は？

Options
- -DWORK_DIR  
  Configuration and log output directory
- -DLOG_DIR  
  Log output directory

## How to use

### Use-case

#### Uses as a gateway

To use a local device from an application
1. Register TD (original TD) of the WoT device to the gateway
2. Search and get generated shadow TDs from the gateway

#### Uses as a TD directory

To use a local device from an application
1. Register TD (original TD) of the WoT device to the gateway
2. Search and get original TDs from the gateway

### APIs
#### TD registration

Request:
- path: `/Things/register`
- method: `POST`
- content-type: `application/json`
- body: TD

Example:
```
$ curl -X POST -H 'Content-type: application/json' -d '<TD>' http://localhost/Things/register
```

#### TD acquisition

##### Acquire registered TDs

Request:
- path: `/Things`
- method: `GET`

Return:
- List of encoded ids

Query options:
- type=shadow: List of shadow TDs
- type=original: List of original TDs
- type=idList: List of encoded ids (default)
- type=linkList: List of id and URL of TD
- type=linkObject: Key value pairs of id and URL of TD

Example:
```
$ curl http://localhost/Things
["urn:dev:ops:32473-WoTLamp-1234"]
```

##### Acuquire specified TD

Request:
- path: `/Things/<endoded id>`
- method: `GET`

Return:
- Shadow TD

Query Options:
- type=shadow: Get shadow TD (default)
- type=original: Get original TD

Example:
```
$ curl http://localhost/Things/urn:dev:ops:32473-WoTLamp-1234
{
    "@context": "https://www.w3.org/2019/wot/td/v1",
    "id": "urn:dev:ops:32473-WoTLamp-1234",
    "title": "MyLampThing",
...
```

## Appendix
### A. Manual setup
#### Equinox OSGi

Download [Equinox OSGi](https://www.eclipse.org/equinox/)

Version: Oxygen.3a or later

Required files
- org.eclipse.osgi-XXX.jar
- org.eclipse.osgi.util-XXX.jar
- org.eclipse.osgi.services-XXX.jar

#### Jetty boot

Download `jetty-osgi-boot` and `jetty-deploy` from [Maven Repository](https://mvnrepository.com/)

Version: 9.4.12 or later

#### Other libraries

Download depending libraries by `mvn` command

```
$ cd <Clone dir>/com.fujitsu.wot.gateway
$ mvn org.apache.felix:maven-bundle-plugin:bundleall
```

jar files are downloaded to `target/classes`

#### Setup working directory

```
/<Working directory>
|- org.eclipse.osgi-3.15.100.jar
|- org.eclipse.osgi.util-3.5.300.jar
|- org.eclipse.osgi.services-3.8.0.jar
|- configuration/
|  \- config.ini (Copy from '<Clone dir>/config.ini')
|- logs/
|- lib/
|  |- classmate_1.1.0.jar
|  \- ... (Copy libraries, jetty-osgi-boot and jetty-deploy here)
|- gateway/
   \- com.fujitsu.wot.gateway-1.0.0.jar (Build jar file)
```

### B. Add authentication

This gateway can add authentication to expose none authentication devices to the internet.

Following options are corresponding to this function.
```
core.authType=<Additional authentication type ("basic" or empty)>
core.authValue=<Authentication option (ex. user:pass for basic auth)>
core.authName=<Additional authentication scheme name> # default = __auto_added_sc__
```

When the gateway received a TD with no security definitions (nosec only), the gateway generates a shadow with basic security scheme. Otherwise, the gateway bypasses security headers to devices.

### C. HTTPS server setting

Step 1. Extract `jettyhome` from jetty-osgi-boot

```
$ cd configuration
$ jar -xvf ../lib/jetty-osgi-boot-9.4.12.v20180830.jar jettyhome
$ cd jettyhome/etc
```

Step 2. Download `jetty-https.xml`, `jetty-ssl.xml` and `jett-ssl-context.xml` from [github](https://github.com/eclipse/jetty.project/tree/jetty-9.4.x/jetty-server/src/main/config/etc)

NOTICE: Use [jetty-9.4.x-3509-jmx_listener_2](https://github.com/eclipse/jetty.project/tree/jetty-9.4.x-3509-jmx_listener_2/jetty-server/src/main/config/etc) branch for 9.4.12
```
wget https://raw.githubusercontent.com/eclipse/jetty.project/jetty-9.4.x-3509-jmx_listener_2/jetty-server/src/main/config/etc/jetty-ssl.xml
wget https://raw.githubusercontent.com/eclipse/jetty.project/jetty-9.4.x-3509-jmx_listener_2/jetty-server/src/main/config/etc/jetty-ssl-context.xml
wget https://raw.githubusercontent.com/eclipse/jetty.project/jetty-9.4.x-3509-jmx_listener_2/jetty-server/src/main/config/etc/jetty-https.xml
```

Step3. Generate keystore

a) Generate self signed key

```
keytool -keystore keystore -alias jetty -genkey -keyalg RSA -sigalg SHA256withRSA
```

b) Import openssl key
```
keytool -keystore keystore -import -alias jetty -file <keyfile.crt> -trustcacerts
```

See [jetty document](https://www.eclipse.org/jetty/documentation/9.4.x/configuring-ssl.html)

Step 4. Edit `jetty-ssl-context.xml`

```
<Set name="KeyStorePassword">[Keystore password]</Set>
<Set name="KeyManagerPassword">[Key password]</Set>
```

Step 5. Switch comment-out in config.ini
```
#jetty.home.bundle=org.eclipse.jetty.osgi.boot
jetty.home=configuration/jettyhome
jetty.base=configuration/jettyhome
jetty.etc.config.urls=\
    etc/jetty.xml,\
    etc/jetty-http.xml,\
    etc/jetty-deployer.xml,\
    etc/jetty-ssl.xml,\
    etc/jetty-ssl-context.xml,\
    etc/jetty-https.xml
jetty.ssl.port=<SSL port>
jetty.sslContext.needClientAuth=true
```

## Copyright

Copyright FUJITSU LABORATORIES LIMITED 2020
