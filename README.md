# EX440-AMQ
AMQ configuration excercices.

## 1. Create cluster
   - Create a cluster using ${HOME_ARTEMIS} at /home/student/exam/brokers directory.
     ```
     cd /home/student/exam/
     mkdir brokers/
     cd brokers/
     ```
   - Broker name must be AcmeFactoryBroker.
     ```
     ${HOME_ARTEMIS}/bin/artemis create AcmeFactoryBroker
     ```
   - User admin, password 123, role amq and allow anonymous mode.
     ```
     Creating ActiveMQ Artemis instance at: /var/opt/amq-broker/mybroker
     --user: is mandatory with this configuration:
     Please provide the default username:
     admin
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


## 3. Handling undelivered messages

## 4. Roles

## 5. Configuring Acceptors and Connectors

## 6. Configuring Paging

## 7. Clustering

## 8. Configuring HA
