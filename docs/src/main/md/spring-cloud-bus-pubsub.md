## Spring Cloud Bus

Using [Cloud Pub/Sub](https://cloud.google.com/pubsub/) as the [Spring
Cloud Bus](https://spring.io/projects/spring-cloud-bus) implementation
is as simple as importing the `spring-cloud-gcp-starter-bus-pubsub`
starter.

This starter brings in the [Spring Cloud Stream binder for Cloud
Pub/Sub](spring-stream.xml#spring-cloud-stream), which is used to both
publish and subscribe to the bus. If the bus topic (named
`springCloudBus` by default) does not exist, the binder automatically
creates it. The binder also creates anonymous subscriptions for each
project using the `spring-cloud-gcp-starter-bus-pubsub` starter.

Maven coordinates, using [Spring Cloud GCP
BOM](getting-started.xml#bill-of-materials):

``` xml
<dependency>
  <groupId>com.google.cloud</groupId>
  <artifactId>spring-cloud-gcp-starter-bus-pubsub</artifactId>
</dependency>
```

Gradle coordinates:

``` groovy
dependencies {
    implementation("com.google.cloud:spring-cloud-gcp-starter-bus-pubsub")
}
```

### Configuration Management with Spring Cloud Config and Spring Cloud Bus

Spring Cloud Bus can be used to push configuration changes from a Spring
Cloud Config server to the clients listening on the same bus.

To use GCP Pub/Sub as the bus implementation, both the configuration
server and the configuration client need the
`spring-cloud-gcp-starter-bus-pubsub` dependency.

All other configuration is standard to [Spring Cloud
Config](https://spring.io/projects/spring-cloud-config).

[Here is a graph](https://github.com/GoogleCloudPlatform/spring-cloud-gcp/blob/main/docs/src/main/asciidoc/images/spring_cloud_bus_over_pubsub.png)
explaining how Spring Cloud Bus over PubSub works.

Spring Cloud Config Server typically runs on port `8888`, and can read
configuration from a [variety of source control
systems](https://cloud.spring.io/spring-cloud-config/spring-cloud-config.html#_environment_repository)
such as GitHub, and even from the local filesystem. When the server is
notified that new configuration is available, it fetches the updated
configuration and sends a notification (`RefreshRemoteApplicationEvent`)
out via Spring Cloud Bus.

When configuration is stored locally, config server polls the parent
directory for changes. With configuration stored in source control
repository, such as GitHub, the config server needs to be notified that
a new version of configuration is available. In a deployed server, this
would be done automatically through a GitHub webhook, but in a local
testing scenario, the `/monitor` HTTP endpoint needs to be invoked
manually.

    curl -X POST http://localhost:8888/monitor -H "X-Github-Event: push" -H "Content-Type: application/json" -d '{"commits": [{"modified": ["application.properties"]}]}'

By adding the `spring-cloud-gcp-starter-bus-pubsub` dependency, you
instruct Spring Cloud Bus to use Cloud Pub/Sub to broadcast
configuration changes. Spring Cloud Bus will then create a topic named
`springCloudBus`, as well as a subscription for each configuration
client.

The configuration server happens to also be a configuration client,
subscribing to the configuration changes that it sends out. Thus, in a
scenario with one configuration server and one configuration client, two
anonymous subscriptions to the `springCloudBus` topic are created.
However, a config server disables configuration refresh by default (see
[ConfigServerBootstrapApplicationListener](https://static.javadoc.io/org.springframework.cloud/spring-cloud-config-server/2.1.0.RELEASE/index.html)
for more details).

A [demo
application](https://github.com/GoogleCloudPlatform/spring-cloud-gcp/tree/main/spring-cloud-gcp-samples/spring-cloud-gcp-pubsub-bus-config-sample)
showing configuration management and distribution over a Cloud
Pub/Sub-powered bus is available. The sample contains two examples of
configuration management with Spring Cloud Bus: one monitoring a local
file system, and the other retrieving configuration from a GitHub
repository.
