# Source: https://rasa.com/docs/pro/deploy/load-testing

On this page

In order to gather metrics on our system's ability to handle increased loads and users, we have performed tests to evaluate the maximum number of concurrent users a Rasa assistant can handle with certain machine configurations. In each test case we spawned the following number of concurrent users at peak concurrency using a [spawn rate](https://docs.locust.io/en/1.5.0/configuration.html#all-available-configuration-options) of 1000 users per second. In our tests we used the Rasa [HTTP-API](https://rasa.com/docs/reference/api/pro/http-api/) and the [Locust](https://locust.io/) open source load testing tool.

| Users | CPU | Memory |
| --- | --- | --- |
| Up to 50,000 | 6vCPU | 16 GB |
| Up to 80,000 | 6vCPU, with almost 90% CPU usage | 16 GB |

### Debugging bot related issues while scaling up[​](https://rasa.com/docs/pro/deploy/load-testing/#debugging-bot-related-issues-while-scaling-up 'Direct link to Debugging bot related issues while scaling up')

To test the Rasa [HTTP-API](https://rasa.com/docs/reference/api/pro/http-api/) ability to handle a large number of concurrent user activity we used Rasa's [tracing](https://rasa.com/docs/pro/improve/tracing/) capability along with a tracing backend or collector, such as Jaeger, to collect traces for the bot under test.

note

Our team is currently in the process of running additional performance-related tests. More information will be added here as we progress.

- [Debugging bot related issues while scaling up](https://rasa.com/docs/pro/deploy/load-testing/#debugging-bot-related-issues-while-scaling-up)