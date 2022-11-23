# WIS2-GlobalBroker-NodeRed

The code in this repository is to provide basic reference of a Global Broker as defined in the (upcoming) technical specification of WIS2.
The repo has been cloned from the official NodeRed repository.

## What is here ?

1. This is the source code of the container `golfvert/wis2globalbrokernodered` and the required files to run the deduplication software in front of a Global Broker
2. configuration files to run the container available at `golfvert/wis2globalbrokernodered`

## What does it do ?

1. Listen to subscribed topics from WIS2Node and other Global Brokers (one subscription per container)
2. Optionally verify/discard/ignore the message for its validity compared to approved message format
3. Look at the `id` in the message. 
4. Through a redis request check if that `id` has already been seen in the last 15 minutes. If yes, simply discard the message
5. If not, publish the message to the attached Global Broker
6. It also provides prometheus metrics available at http://@IP:1880/metrics

## How to use it ?

Download 
- docker-compose.yaml

and edit:

Copy & Paste the subscriber_mqtt_1 to get one container per WIS2Node or other Global Brokers to subscribe to. Do NOT subscribe to the local Global Broker.
- Change the name of the container (make sure it is unique!)
- Change all MQTT_SUB_* to connect to the remote broker and to the topic from that source. In the example below `wis2/a/origin/fra/#` will subscribe to all topic from France according to WIS2 agreed topic hierarchy.
- Change ports 1st 1880:1880, 2nd 1881:1880,...

```
  subscriber_mqtt_1:
    container_name: subscriber_mqtt_1
    image: golfvert/wis2globalbrokernodered
    env_file:
      - ./publisher.env
    environment:
      - TZ=Europe/Paris
      - MQTT_SUB_BROKER=Broker_URL   # WIS2Node URL broker such as mqtts://broker.example.com:8883 or wss://broker.example.com:443
      - MQTT_SUB_USERNAME=
      - MQTT_SUB_PASSWORDD=
      - MQTT_SUB_TOPIC=Topic_to_sub   # e.g. wis2/a/origin/FRA/#
      - MQTT_PUB_BROKER=GlobalBroker_URL   # Global Broker URL such as mqtts://globalbroker.site.com:8883 or wss://globalbroker.site.com:443
      - MQTT_PUB_USERNAME=
      - MQTT_PUB_PASSWORD=
      - MQTT_MONIT_CENTREID=Name_of_Center
      - MQTT_MONIT_TOPIC=Topic_to_publish_on_Global_Broker
      - MSG_CHECK_OPTION=verify      # Should messages be "verify" (just add _comment in the message), "discard" (bin the message if not correct), "ignore" (don't check the messages)
    ports:
      - "1880:1880"
    networks:
      - wis2relay
    depends_on:
      - redis
 ```
MQTT_MONIT_TOPIC is optional. If defined, statistics on the status of the subsciption to the remote broker will be published to the Global Broker on the topic MQTT_MONIT_TOPIC/status. And every minute, the time difference (in seconds) between the current time and the time when the last message has been received from the remote broker. This will will be published on MQTT_MONIT_TOPIC/pubsub. If MQTT_MONIT_TOPIC is empty or does not exist no statistics will be published.
MQTT_MONIT_CENTREID is used as a variable in the payload sent to the Global Broker.

When done, save the docker-compose.yaml and start it with `docker compose up -d`

It will subscribe to all "remote" destinations (WIS2node(s), other Global Brokers) and will publish to the local Global Broker.

This is not production ready, just a tool to show how WIS2 and in particular the Global Broker part will work.

## How to modify it ?

This is a fork from the official Node-Red repo. Follow official documentation to tweak it to your needs.
