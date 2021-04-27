# MQTT Sparkplug-B Driver

This client driver connects to a MQTT Broker and can

* Act as a Sparkplug Primary (or non-primary) SCADA host.
* Subscribe to Sparkplug B devices, create and update tags with Sparkplug metrics.
* Subscribe to regular MQTT topics, create and updated tags with topics with auto-detected payloads like number, string, boolean, JSON, relaxed JSON.
* Subscribe to regular binary MQTT topics to be saved as files on MongoDB-Gridfs.
* Publish tags as regular MQTT topics.
* Publish tags as a Sparkplug B device.

##  Configure a driver instance

To create a new _MQTT-SPARKPLUG-B_ driver instance, use the Admin UI or insert a new document in the _protocolDriverInstances_ collection using a command like this:

    // be sure to be in the right database, normally json_scada
    // uncomment and adjust the next line as necessary
    // use json_scada
    db.protocolDriverInstances.insert({
            protocolDriver: "MQTT-SPARKPLUG-B",
            protocolDriverInstanceNumber: 1,
            enabled: true,
            logLevel: 1,
            nodeNames: ["mainNode"], 
            activeNodeName: "mainNode",
            activeNodeKeepAliveTimeTag: new Date(),
        });

* _**protocolDriver**_ [String] - Name of the protocol driver, must be "MQTT-SPARKPLUG-B". **Mandatory parameter**.
* _**protocolDriverInstanceNumber**_ [Double] - Number of the instance. Use 1 to N to number instances. For the same driver instance numbers should be unique. The instance number makes possible to run use multiple processes of the driver, each one with a distinct configuration. **Mandatory parameter**.
* _**enabled**_ [Boolean] - Controls the enabling of the instance. Use false here to disable the instance. **Mandatory parameter**.
* _**logLevel**_ [Double] - Number code for log level (0=minimum,1=basic,2=detailed,3=debug). Too much logging (levels 2 and 3) can affect performance. **Mandatory parameter**.
* _**nodeNames**_ [Array of Strings]- Array of node names that can run the instance. Use more than one node for redundancy. Each redundant instance running on separate nodes will have the same connections and data enabled for scanning and update. **Mandatory parameter**.
* _**activeNodeName**_ [String] - Name of the protocol driver that is currently active. This is updated by the drivers for redundancy control.**Optional**.
* _**activeNodeKeepAliveTimeTag**_ [Date] - This is updated regularly  by the active driver. **Optional**.

## Configure client connection to MQTT broker

Each instance for this driver can have just one connection defined that must be described in the _protocolConnections_ collection.

    // be sure to be in the right database, normally json_scada
    // uncomment and adjust the next line as necessary
    // use json_scada
    db.protocolConnections.insert({
        protocolDriver: "MQTT-SPARKPLUG-B",
        protocolDriverInstanceNumber: 1,
        protocolConnectionNumber: 1200,
        name: "MQTT-BROKER",
        description: "TELEGRAF Demo",
        enabled: true,
        commandsEnabled: true,
        autoCreateTags: true,
        endpointURLs: ["mqtt://broker.hivemq.com:1883"],
        topics: ["spBv1.0/#"],
        topicsAsFiles: ["docs/#"],
        topicsScripted: [{topic: "sensor1", script: " let obj=JSON.parse(payload); {value: obj.value, qualityOk: true, timestamp: (new Date()).getTime() } "}],
        groupId: "Sparkplug B Devices",
        edgeNodeId: "JSON-SCADA Server",
        deviceId: "JSON-SCADA Device",
        scadaHostId : "Primary Application",
        publishTopicRoot: "EnterpriseName",
        username: "",
        password: "",
        useSecurity: false,
        chainValidation: false,
        rootCertFilePath: "",
        localCertFilePath: "",
        privateKeyFilePath: "",
        pfxFilePath: "",
        passphrase: "",
        allowTLSv10: false,
        allowTLSv11: false,
        allowTLSv12: true,
        allowTLSv13: true,
        cipherList: "",
    });

* _**protocolDriver**_ [String] - Name of the protocol driver, must be "MQTT-SPARKPLUG-B". **Mandatory parameter**.
* _**protocolDriverInstanceNumber**_ [Double] - Number of the instance. Use 1 to N to number instances. For the same driver instance numbers should be unique. The instance number makes possible to run use multiple processes of the driver, each one with a distinct configuration. **Mandatory parameter**.
* _**protocolConnectionNumber**_ [Double] - Number code for the protocol connection. This must be unique for all connections over all drivers on a system. This number is be used to define the connection that can update a tag. **Mandatory parameter**.
* _**name**_ [String] - Name for a connection. Will be used for logging. **Mandatory parameter**.
* _**description**_ [String] - Description for the purpose of a connection. Just documental. **Optional parameter**.
* _**enabled**_ [Boolean] - Controls the enabling of the connection. Use false here to disable the connection. **Mandatory parameter**.
* _**commandsEnabled**_ [Boolean] - Allows to disable commands (publishing messages in control direction) for a connection. Use false here to disable commands. **Mandatory parameter**.
* _**autoCreateTags**_ [Boolean] - Enables automatic creation of all discovered tags. **Mandatory parameter**.
* _**endpointURLs**_ [Array of Strings] - List of URLS for connection to MQTT brokers. Use more than one broker only in case of redundant brokers. **Mandatory parameter**.
* _**topics**_ [Array of Strings] - List of topics to subscribe on MQTT broker. Sparkplug B devices publish to "spBv1.0/#". Sparkplug B metrics will be converted as tags with topic name (minus "spBv1.0" root and message type) plus metric name. Regular MQTT topics will be converted to tags with the full topic name as object address. **Mandatory parameter**.
* _**topicsAsFiles**_ [Array of Strings] - List of topics to subscribe on MQTT broker to be saved as files on MongoDB (Gridfs). **Mandatory parameter**.
* _**topicsScripted**_ [Array of Objects] - List of topics to subscribe on MQTT broker to be treated with dedicated scripts to extract data. **Mandatory parameter**.
* _**groupId**_ [String] - Group Id for publishing. **Mandatory parameter**.
* _**edgeNodeId**_ [String] - Edge Node Id for publishing. **Mandatory parameter**.
* _**deviceId**_ [String] - Device Id for publishing. **Optional parameter**.
* _**scadaHostId**_ [String] - Scada host Id for Primary Application STATE publishing. Leave empty if not a primary application. **Mandatory parameter**.
* _**publishTopicRoot**_ [String] - Non-Sparkplug MQTT topic root for publishing tags. Leave empty to not publish tags as normal topics. **Mandatory parameter**.
* _**username**_ [String] - The username for the MQTT broker connection. **Optional parameter**.
* _**password**_ [String] - The password for the MQTT broker connection. **Optional parameter**.
* _**useSecurity**_ [Boolean] - Use (true) or not (false) secure encrypted connection. **Mandatory parameter**.
* _**rootCertFilePath**_ [String] - Trusted CA certificates PEM file path (equiv to NodeJS TLS option 'ca'). **Optional parameter**.
* _**privateKeyFilePath**_ [String] - File (*.PEM) that contains the private key corresponding to the local certificate (equiv. to NodeJS TLS option 'key'). **Optional parameter**.
* _**pfxFilePath**_ [String] - PFX or PKCS12 File path to encoded private key and certificate chain. pfx is an alternative to providing key and cert individually. (equiv. to NodeJS TLS option 'pfx'). **Optional parameter**.
* _**passphrase**_ [String] - Shared passphrase used for a single private key and/or a PFX (equiv. to NodeJS TLS option 'passphrase'). **Optional parameter**.
* _**chainValidation**_ [Boolean] - Indicates whether the drivers performs a X509 chain validation against the registered CA certificates (equiv. to NodeJS TLS option 'rejectUnauthorized'). Default: false. **Optional parameter**.
* _**localCertFilePath**_ [String] - File that contains the certificate (*.PEM) that will be presented to the remote side of the connection (equiv. to NodeJS TLS option 'cert'). **Optional parameter**.
* _**allowTLSv10**_ [Boolean] - Allow TLS version 1.0 (default true, recommended false). **Optional parameter**.
* _**allowTLSv11**_ [Boolean] - Allow TLS version 1.1 (default true, recommended false). **Optional parameter**.
* _**allowTLSv12**_ [Boolean] - Allow TLS version 1.2 (default true). **Optional parameter**.
* _**allowTLSv13**_ [Boolean] - Allow TLS version 1.3 (default true). **Optional parameter**.
* _**cipherList**_ [String] - TLS cipher list (equiv. to NodeJS TLS option 'ciphers'). Leave empty to use defaults. **Optional parameter**.

See also NodeJS TLS configuration and Sparkplug-Client original lib.

* https://nodejs.org/api/tls.html
* https://github.com/Cirrus-Link/Sparkplug/tree/master/client_libraries/javascript/sparkplug-client

## Example of JSON-SCADA Protocol Driver Instances and Connections Numbering

![Driver instances and connections](https://github.com/riclolsen/json-scada/raw/master/docs/JSON-SCADA_Connections.png "Driver Instances and Connections Numbering")

## Example of Script to Extract Data from complex Payloads

Given a topic "test/jsonarr" that publishes array of values as JSON like 

    [ 12345.2, 23456.7, 345678.9 ]

The script should extract values and return an array o object like

    [
    {
        objectAddress: 'scrVal1',
        value: 12345.2,
        qualityOk: true,
        timestamp: 1619465592683
    },
    {
        objectAddress: 'scrVal2',
        value: 23456.7,
        qualityOk: true,
        timestamp: 1619465592683
    },
    {
        objectAddress: 'scrVal3',
        value: 345678.9,
        qualityOk: true,
        timestamp: 1619465592683
    }
    ]

It is necessary to subscribe the topic using the topicsScripted property.    

    "topicsScripted": [{ 
        "topic": "C3ET/test/jsonarr", 
        "script": " // remove comments and put all in the same line
                let ret = []; // array of objects to return
                let vals=JSON.parse(info.payload.toString()); 
                let cnt = 1;
                vals.forEach(elem => {
                    ret.push({'objectAddress': 'scrVal'+cnt, 'value': elem, 'qualityOk': true, 'timestamp': (new Date()).getTime() });
                    cnt++;
                })
                ret; // return values in array of objects
                "
        }]

The script is executed in a sandboxed Javascript VM. The MQTT payload is passed as a buffer in the "info.payload" object.

The value of the last expression returns to the main process, this should be an array of objects with at least "value" and "objectAddress" properties set.
