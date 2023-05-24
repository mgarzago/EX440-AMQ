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

## 6. Configuring Paging

## 7. Clustering

## 8. Configuring HA
