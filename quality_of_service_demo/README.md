# Quality of Service Demos

The demo applications in this package show the various Quality of Service settings available in ROS2.
The intent is to provide a dedicated application for each QoS type, in order to highlight that single feature in isolation.

## History

No History QoS demo app exists yet in this package.

## Reliability

No Reliability QoS demo app exists yet in this package.

## Durability

No Durability QoS demo app exists yet in this package.

## Deadline

`quality_of_service_demo/deadline` demonstrates messages not meeting their required deadline.

The application requires an argument a `deadline_duration` - this is the maximum acceptable time (in positive integer milliseconds) between messages published on the topic.
A `deadline_duration` of `0` means that there is no deadline, so the application will act as a standard Talker/Listener pair.

The Publisher in this demo will pause publishing periodically, which will print deadline expirations from the Subscriber.

Run `quality_of_service_demo/deadline -h` for more usage information.

Examples:
* `ros2 run quality_of_service_demo_cpp deadline 600 --publish-for 5000 --pause-for 2000` _or_
* `ros2 run quality_of_service_demo_py  deadline 600 --publish-for 5000 --pause-for 2000`
  * The publisher will publish for 5 seconds, then pause for 2 - deadline events will start firing on both participants, until the publisher comes back.
* `ros2 run quality_of_service_demo_cpp deadline 600 --publish-for 5000 --pause-for 0` _or_
* `ros2 run quality_of_service_demo_py  deadline 600 --publish-for 5000 --pause-for 0`
  * The publisher doesn't actually pause, no deadline misses should occur.

## Lifespan

`quality_of_service_demo/lifespan` demonstrates messages being deleted from a Publisher's outgoing queue once their configured lifespan expires.

The application requires an argument `lifespan_duration` - this is the maximum time (in positive integer milliseconds) that a message will sit in the Publisher's outgoing queue.

The Publisher in this demo will publish a preset number of messages, then stop publishing.
A Subscriber will start subscribing after a preset amount of time and may receive _fewer_ backfilled messages than were originally published, because some message can outlast their lifespan and be removed.

Run `lifespan -h` for more usage information.

Examples:
* `ros2 run quality_of_service_demo_cpp lifespan 1000 --publish-count 10 --subscribe-after 3000` _or_
* `ros2 run quality_of_service_demo_py  lifespan 1000 --publish-count 10 --subscribe-after 3000`
  * After a few seconds, you should see (approximately) messages 4-9 printed from the Subscriber.
  * The first few messages, with 1 second lifespan, were gone by the time the Subscriber joined after 3 seconds.
* `ros2 run quality_of_service_demo_cpp lifespan 4000 --publish-count 10 --subscribe-after 3000` _or_
* `ros2 run quality_of_service_demo_py  lifespan 4000 --publish-count 10 --subscribe-after 3000`
  * After a few seconds, you should see all of the messages (0-9) printed from the Subscriber.
  * All messages, with their 4 second lifespan, survived until the subscriber joined.

## Liveliness

`quality_of_service_demo/liveliness` demonstrates notification of liveliness changes, based on different Liveliness configurations.

The application requires an argument `lease_duration` that specifies how often (in milliseconds) an entity must "check in" to let the system know it's alive.

The Publisher in this demo will assert its liveliness based on passed in options, and be stopped after some amount of time.
When using `AUTOMATIC` liveliness policy, the Publisher is deleted at this time, in order to stop all automatic liveliness assertions in the rmw implementation - therefore only the Subscription will receive a Liveliness event for `AUTOMATIC`.
For `MANUAL` liveliness policies, the Publisher's assertions are stopped, as well as its message publishing.
Publishing a messages implicitly counts as asserting liveliness, so publishing is stopped in order to allow the Liveliness lease to lapse.
Therefore in the `MANUAL` policy types, you will see Liveliness events from both Publisher and Subscription (in rmw implementations that implement this feature).

Run `quality_of_service_demo/liveliness -h` for more usage information.

Examples:
* `ros2 run quality_of_service_demo_cpp liveliness 1000 --kill-publisher-after 2000` _or_
* `ros2 run quality_of_service_demo_py  liveliness 1000 --kill-publisher-after 2000`
  * After 2 seconds, the publisher will be killed, and the subscriber will receive a callback 1 second after that notifying it that the liveliness has changed.
* `ros2 run quality_of_service_demo_cpp liveliness 250 --node-assert-period 0 --policy MANUAL_BY_NODE` _or_
* `ros2 run quality_of_service_demo_py  liveliness 250 --node-assert-period 0 --policy MANUAL_BY_NODE`
  * With this configuration, the node never explicitly asserts its liveliness, but it publishes messages (implicitly asserting liveliness) less often (500ms) than the liveliness lease, so you will see alternating alive/not-alive messages as the publisher misses its lease and "becomes alive again" when it publishes, until the talker is stopped.
