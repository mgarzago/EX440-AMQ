# EX440-AMQ
AMQ configuration excercices.

## 1. Create cluster
   - Create a cluster using ${HOME_ARTEMIS} at /home/student/exam/brokers directory.
     ```bash
     cd /home/student/exam/
     mkdir brokers/
     cd brokers/
     ```
   - Broker name must be AcmeFactoryBroker.
     ```bash
     ${HOME_ARTEMIS}/bin/artemis create AcmeFactoryBroker
     ```
   - User jboss, password 123, role amq and allow anonymous mode.
     ```bash
     Creating ActiveMQ Artemis instance at: /var/opt/amq-broker/mybroker
     --user: is mandatory with this configuration:
     Please provide the default username:
     jboss
     --password: is mandatory with this configuration:
     Please provide the default password:
     123
     --role: is mandatory with this configuration:
     Please provide the default role:
     amq
     --allow-anonymous | --require-login: is mandatory with this configuration:
     Allow anonymous access? (Y/N):
     Y
     ```

## 2. Create addresses
   - Create an address called "Production." with a queue called "Production". All messages must be received by all consumers.
     ```xml
     <address name="Production">
        <multicast>
           <queue name="Production"/>
        </multicast>
     </address>
     ```
   - Create an address called "QA." with a queue called "QA". Messages must be distributed amongst all consumers.
     ```xml
     <address name="QA">
        <anycast>
           <queue name="QA"/>
        </anycast>
     </address>
     ```

## 3. Handling undelivered messages
   - Undelivered messages from the "Production." address failed after 3 attempts must be sent to a specific queue called "Monitor.Production".
      - Configure "Production." address setting.
        ```xml
        <address-setting match="Production.#">
           <dead-letter-address>Monitor.Production</dead-letter-address>
           <max-delivery-attempts>3</max-delivery-attempts>
        </address-setting>
        ```
      - Create the new queue for undelivered messages.
        ```xml
        <address name="Monitor.Production">
           <anycast>
              <queue name="Monitor.Production"/>
           </anycast>   
        </address>
        ```
   - All other undelivered messages must be sent to the default DLQ after 5 failed attempts.
     ```xml
     <address-setting match="#">
        <dead-letter-address>DLQ</dead-letter-address>
        <max-delivery-attempts>5</max-delivery-attempts>
     </address-setting>
     ```

## 4. Roles
   - Create user admin with password root and role admin.
     ```bash
     ./bin/artemis user add --user admin --password root --role admin
     ```
   - Create user alpha with password 123 and role producer.
     ```bash
     ./bin/artemis user add --user alpha --password 123 --role producer
     ```
   - Create user beta with password 123 and role consumer.
     ```bash
     ./bin/artemis user add --user beta --password 123 --role consumer
     ```
   - Secure the "Production." address so that:
      - User admin can createDurableQueue, deleteDurableQueue, createNonDurableQueue, deleteNonDurableQueue, browse and manage queues.
      - User alpha can send messages.
      - User beta can consume messages.
        ```xml
        <security-setting match="Production.#">
           <permission type="createDurableQueue" roles="amq, admin"/>
           <permission type="deleteDurableQueue" roles="amq, admin"/>
           <permission type="createNonDurableQueue" roles="amq, admin"/>
           <permission type="deleteNonDurableQueue" roles="amq, admin"/>
           <permission type="send" roles="amq, producer"/>
           <permission type="consume" roles="amq, consumer"/>
           <permission type="manage" roles="amq, admin"/>
           <permission type="browse" roles="amq, admin"/>
        </security-setting>
        ```
        
## 5. Configuring Acceptors and Connectors
   - Secure the broker so that it uses TLS/SSL using port 5500 (?). 
      - The keyStorePtah is ```/home/student/JB440/labs/jb440-review-1/keystore/activemq.example.keystore```
      - The trustStorePath is ```/home/student/JB440/labs/jb440-review-1/keystore/activemq.example.truststore```
      - The password for both files is activemqexample.
     ```xml
     <connectors>
        <connector name="netty-ssl-connector">tcp://localhost:61616?sslEnabled=true;trustStorePath=/home/student/JB440/labs/jb440-review-1/keystore/activemq.example.truststore;trustStorePassword=activemqexample</connector>
     </connectors>
     <acceptors>
        <acceptor name="netty-ssl-acceptor">tcp://localhost:61616?sslEnabled=true;keyStorePath=/home/student/JB440/labs/jb440-review-1/keystore/activemq.example.keystore;keyStorePassword=activemqexample</acceptor>
     </acceptors>
     ```
     
## 6. Configuring Paging
   - "Production." address should have a size in memory allowed of 100000 bytes and page size of 20000 bytes.
     ```xml
     <address-setting match="Production.#">
        <max-size-bytes>100000</max-size-bytes>
        <page-size-bytes>20000</page-size-bytes>
        <address-full-policy>PAGE</address-full-policy>
     </address-setting>
     ```
     
## 7. Clustering
   - Secure the cluster connection using the user cluster-admin with password cluster-root.
     ```xml
     <cluster-user>cluster-admin</cluster-user>
     <cluster-password>cluster-root</cluster-password>
     ```
   - Configure cluster settings for a symetric cluster. Brokers should discover each other using UDP 231.7.8.12.
     ```xml
     <broadcast-groups>
        <broadcast-group name="my-broadcast-group"> 
           <group-address>${udp-address:231.7.8.12}</group-address> 
           <group-port>9876</group-port> 
           <connector-ref>netty-ssl-connector</connector-ref>
        </broadcast-group>
     </broadcast-groups>
     
     <discovery-groups>
        <discovery-group name="my-discovery-group">
           <group-address>${udp-address:231.7.8.12}</group-address>
           <group-port>9876</group-port>
        </discovery-group>
     </discovery-groups>

     <cluster-connections>
        <cluster-connection name="my-cluster">
           <connector-ref>netty-ssl-connector</connector-ref>
           <discovery-group-ref discovery-group-name="my-discovery-group"/>
           <message-load-balancing>STRICT</message-load-balancing>
           <max-hops>1</max-hops>
           <use-duplicate-detection>true</use-duplicate-detection>
        </cluster-connection>
     </cluster-connections>

     <address-setting match="#">
        <redelivery-delay>0</redelivery-delay>
        <redistribution-delay>0</redistribution-delay>
     </address-setting>
     ```
   - Create 2 more brokers named AcmeFactoryBrokerB and AcmeFactoryBrokerC at /home/student/exam/brokers directory (all of the brokers share the exact same configuration except for the ports).
     ```bash
     ${HOME_ARTEMIS}/bin/artemis create AcmeFactoryBrokerB --port-offset 1
     ${HOME_ARTEMIS}/bin/artemis create AcmeFactoryBrokerC --port-offset 2
     ```

## 8. Configuring HA
   - Create 2 more brokers named AcmeFactoryBrokerMaster and AcmeFactoryBrokerSlave at /home/student/exam/brokers directory based on AcmeFactoryBroker configuration.
     ```bash
     ${HOME_ARTEMIS}/bin/artemis create AcmeFactoryBrokerMaster
     ${HOME_ARTEMIS}/bin/artemis create AcmeFactoryBrokerSlave --port-offset 1
     ```
   - Configure High Availability and Failover in both brokers using Replication policy.
      - Configure HA in AcmeFactoryBrokerMaster
        ```xml
        <ha-policy>
           <replication>
              <master>
                 <check-for-live-server>true</check-for-live-server>
              </master>
           </replication>
        </ha-policy>
        ```
      - Configure Failover in AcmeFactoryBrokerSlave
        ```xml
        <ha-policy>
           <replication>
              <slave>
                 <allow-failback>true</allow-failback>
              </slave>
           </replication>
        </ha-policy>
        ```
    
