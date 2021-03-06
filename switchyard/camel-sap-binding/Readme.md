Introduction
============
<p>This quickstart demonstrates the usage of the Camel SAP and it's binding feature, by retrieving a message from SAP ABAP program invocation and creating a flight booking data into SAP tables. This quickstart needs SAP instance and Flight Data Application to be setup on it (It should be done by just executing SAPBC_DATA_GENERATOR from ABAP editor. See http://help.sap.com/saphelp_erp60_sp/helpdata/en/db/7c623cf568896be10000000a11405a/content.htm for more details). The SAP JCo library is also needed to be setup as a JBoss module.</p>
<p>This example application is originally created for JBoss Fuse camel-sap, and then imported as a SwitchYard quickstart. You can also see the original README here<br>
   - https://github.com/fabric8io/fabric8/blob/6.1.x/fabric/fabric8-karaf/src/main/resources/distro/fabric/import/fabric/configs/versions/1.0/profiles/example-camel-sap/ReadMe.md </p>



Building the quickstart
======================

To build the quickstart

    mvn clean install


EAP/WildFly: Creating SAP JCo JBoss module
==========================================
sapjco3.jar, sapidoc3.jar and libsapjco3.so can be downloaded from SAP portal.

1. Copy JCo library files (assuming your platform is linux-x86_64)

    ```
    cp sapjco3.jar sapidoc3.jar $JBOSS_HOME/modules/system/layers/fuse/com/sap/conn/jco/main/
    mkdir -p $JBOSS_HOME/modules/system/layers/fuse/com/sap/conn/jco/main/lib/linux-x86_64
    cp libsapjco3.so $JBOSS_HOME/modules/system/layers/fuse/com/sap/conn/jco/main/lib/linux-x86_64/
    ```

1. Uncomment org.switchyard.component.camel.sap module in the SwitchYard subsystem configuration, e.g. $JBOSS_HOME/standalone/configuration/standalone.xml

    ```
    <!-- Uncomment this module to enable camel-sap binding
    <module identifier="org.switchyard.component.camel.sap" implClass="org.switchyard.component.camel.sap.deploy.CamelSapComponent"/>
    -->
    ```

Fuse/karaf: Adding SAP JCo library to system packages
=====================================================
sapjco3.jar, sapidoc3.jar and libsapjco3.so can be downloaded from SAP portal.

1. Copy JCo library files

    ```
    cp sapjco3.jar sapidoc3.jar libsapjco3.so $KARAF_HOME/lib/
    ```

1. Add following JCo packages into org.osgi.framework.system.packages.extra in $KARAF_HOME/etc/config.properties

    ```
    com.sap.conn.idoc, \
    com.sap.conn.idoc.jco, \
    com.sap.conn.jco, \
    com.sap.conn.jco.ext, \
    com.sap.conn.jco.monitor, \
    com.sap.conn.jco.rt, \
    com.sap.conn.jco.server,
    ```

Registering ABAP program from SAP GUI
=====================================

1. First we need to define a destination for external clients like our camel route
    - Navigate under SAP menu to 'Tools' > 'Administration' > 'Administration' > 'Network' > 'RFC Destinations' (double-click or execute)
    - In RFC Connections expand the 'TCP/IP connections' folder and click "Create" in toolbar (blank page)
    - In RFC Destination Dialog
        -> In 'RFC Destination' put 'JCOSERVER01'
        -> 'Connection Type' should already be filled in for you as type 'T' and 'TCP/IP Connection'
        -> In 'Description 1' put e.g. 'JCo Server Destination'
        -> 'Activation Type' box MUST be 'Registered Server Program'
        -> In 'Program ID' box put 'JCO_SERVER'
        -> The 'CPI-C Timeout' should default to 'Default Gateway Value'
        -> Click SAVE on toolbar
    - So you should now see 'JCOSERVER01' in the list of TCP/IP connections
1. Now it is required to create 2 structures for ABAP program in ABAP Dictionary
    - Navigate to 'Tools' > 'ABAP Workbench' > 'Development' > 'ABAP Dictionary' (double-click / execute)
    - The first structure:
        -> Select 'Data type' and enter 'ZCONNECTION_INFO_STRUCTURE' into text box and click 'Create'
        -> In new displayed checkbox  select 'Structure'
        -> Fill the information similar to ZCONNECTION_INFO_STRUCTURE.png screenshot in EXAMPLE_HOME/Screenshots
        -> click 'SAVE' button and in new dialog box just click on 'Local Object'
    - The second structure:
        -> Select 'Data type' and enter 'ZCONNECTION_INFO_TABLE' into text box and click 'Create'
        -> In new displayed checkbox  select 'TABLE'
        -> Fill the information similar to ZCONNECTION_INFO_TABLE.png screenshot in EXAMPLE_HOME/Screenshots
        -> click 'SAVE' button and in new dialog box just click on 'Local Object' ( package should be predefined)
    - in ABAP Dictionary in toolbar there is MatchStick( Activate). Activate both structures
        -> there will be some warning but it is not important
1. Now we create our ABAP program
    - Navigate to 'Tools' > 'ABAP Workbench' > 'Development' > 'ABAP Editor'
    - Select 'Source Code' and enter 'ZBOOK_FLIGHT' for the program name then click 'Create'
    - Enter "Title' and 'Type' is 'Executable program'
    - Choose again 'Local Object'
    - Now you should see Editor window. Copy content of ZBOOK_FLIGHT.txt from EXAMPLE_HOME/Screenshots
        -> Click on 'Check' in the toolbar and wait for compilation and green box on the bottom
        -> Click 'SAVE' button and in new dialog box just click on 'Local Object'
        -> In ABAP Editor in toolbar there is MatchStick(Activate). Activate the program

Running the quickstart
======================
1. Make sure the 'DestinationMetadataProducer.java' and 'ServerMetadataProducer.java' match those of your SAP instance


EAP
----------
1. Start EAP instance

    ```
    cd ${JBOSS_HOME}
    ./bin/standalone.sh
    ```

2. Build and deploy the quickstart

    ```
    cd ${JBOSS_HOME}/quickstarts/camel-sap-binding
    mvn install -Pdeploy
    ```

3. Now 'Execute' the program in the SAP and define there some information
    - you will probably need look into the table to specify good date of flight and destinations with using Data Browser (/nse16).

4. You will see the result of flight booking request on SAP GUI window

5. Undeploy the quickstart

    ```
    mvn clean -Pdeploy
    ```

Fuse
----------
1. Start the Fuse server :

${FUSE_HOME}/bin/fuse

2. Add the features URL for the respective version of SwitchYard.   

    ```
    JBossFuse:karaf@root> features:addurl mvn:org.switchyard.karaf/switchyard/2.1.0.redhat-630187/xml/core-features
    ```

3. Install the feature for the sap-binding quickstart :

    ```
    JBossFuse:karaf@root> features:install switchyard-quickstart-camel-sap-binding
    ```

4. Now 'Execute' the program in the SAP and define there some information
    - you will probably need look into the table to specify good date of flight and destinations with using Data Browser (/nse16).

5. You will see the result of flight booking request on SAP GUI window

6. Undeploy the quickstart:

    ```
    JBossFuse:karaf@root> features:uninstall switchyard-quickstart-camel-sap-binding
    ```

Wildfly
----------
1. Start Wildfly instance

    ```
    cd ${JBOSS_HOME}
    ./bin/standalone.sh
    ```

2. Build and deploy the quickstart

    ```
    cd ${JBOSS_HOME}/quickstarts/camel-sap-binding
    mvn -Pdeploy -Pwildfly install
    ```

3. Now 'Execute' the program in the SAP and define there some information
    - you will probably need look into the table to specify good date of flight and destinations with using Data Browser (/nse16).

4. You will see the result of flight booking request on SAP GUI window

5. Undeploy the quickstart

    ```
    mvn clean -Pdeploy -Pwildfly
    ```

Karaf
----------
1. Start the Karaf server :

    ```
    ${KARAF_HOME}/bin/karaf
    ```

2. Add the features URL for the respective version of SwitchYard.   Replace {SWITCHYARD-VERSION}
with the version of SwitchYard that you are using (ex. 2.0.0): 

    ```
    karaf@root> features:addurl mvn:org.switchyard.karaf/switchyard/{SWITCHYARD-VERSION}/xml/features
    ```

3. Install the feature for the sap-binding quickstart :

    ```
    karaf@root> features:install switchyard-quickstart-camel-sap-binding
    ```

4. Now 'Execute' the program in the SAP and define there some information
    - you will probably need look into the table to specify good date of flight and destinations with using Data Browser (/nse16).

5. You will see the result of flight booking request on SAP GUI window

6. Undeploy the quickstart:

    ```
    karaf@root> features:uninstall switchyard-quickstart-camel-sap-binding
    ```


## Further Reading

1. [SAP Binding Documentation](https://docs.jboss.org/author/display/SWITCHYARD/SAP)
1. [JBoss Fuse SAP Component Documentation] (https://access.redhat.com/site/documentation/en-US/Red_Hat_JBoss_Fuse/6.1/html/Apache_Camel_Component_Reference/files/SAP.html)

