Iron.io Go Client Library
-------------

# IronMQ

[IronMQ](http://www.iron.io/products/mq) is an elastic message queue for managing data and event flow within cloud applications and between systems.

The [full API documentation is here](http://dev.iron.io/mq/reference/api/) and this client tries to stick to the API as
much as possible so if you see an option in the API docs, you can use it in the methods below. 

You can find [Go docs here](http://go.pkgdoc.org/github.com/iron-io/iron_go).

## Getting Started

### Get credentials

To start using iron_go, you need to sign up and get an oauth token.

1. Go to http://iron.io/ and sign up.
2. Create new project at http://hud.iron.io/dashboard
3. Download the iron.json file from "Credentials" block of project

--

### Configure

1\. Reference the library:

```go
import "github.com/iron-io/iron_go/mq"
```

2\. [Setup your Iron.io credentials](http://dev.iron.io/mq/reference/configuration/)

3\. Create an IronMQ client object:

```go
queue := mq.New("test_queue");
```

## The Basics

### Get Queues List

```go
queues, err := mq.ListQueues(0, 100);
for _, element := range queues {
	fmt.Println(element.Name);
}
```

--

### Get a Queue Object

You can have as many queues as you want, each with their own unique set of messages.

```go
queue := mq.New("test_queue");
```

Now you can use it.

--

### Post a Message on a Queue

Messages are placed on the queue in a FIFO arrangement.
If a queue does not exist, it will be created upon the first posting of a message.

```go
id, err := q.PushString("Hello, World!")
```

--

### Retrieve Queue Information

```go
info, err := q.Info()
fmt.Println(info.Name);
```

--

### Get a Message off a Queue

```go
msg, err := q.Get()
fmt.Printf("The message says: %q\n", msg.Body)
```

--

### Delete a Message from a Queue

```go
msg, _ := q.Get()
// perform some actions with a message here
msg.Delete()
```

Be sure to delete a message from the queue when you're done with it.

--

## Queues

### Retrieve Queue Information

```go
info, err := q.Info()
fmt.Println(info.Name);
fmt.Println(info.Size);
```

QueueInfo struct consists of the following fields:

```go
type QueueInfo struct {
	Id            string            `json:"id,omitempty"`
	Name          string            `json:"name,omitempty"`
	PushType      string            `json:"push_type,omitempty"`
	Reserved      int               `json:"reserved,omitempty"`
	RetriesDelay  int               `json:"retries,omitempty"`
	Retries       int               `json:"retries_delay,omitempty"`
	Size          int               `json:"size,omitempty"`
	Subscribers   []QueueSubscriber `json:"subscribers,omitempty"`
	TotalMessages int               `json:"total_messages,omitempty"`
	ErrorQueue    string            `json:"error_queue,omitempty"`
}
```

--

### Delete a Message Queue

As of now the library doesn't support deleting of queues.

--

### Post Messages to a Queue

**Single message:**

```go
id, err := q.PushString("Hello, World!")
// To control parameters like timeout and delay, construct your own message.
id, err := q.PushMessage(&mq.Message{Timeout: 60, Delay: 0, Body: "Hi there"})
```

**Multiple messages:**

You can also pass multiple messages in a single call.

```go
ids, err := q.PushStrings("Message 1", "Message 2")
```

To control parameters like timeout and delay, construct your own message.

```go
ids, err = q.PushMessages(
	&mq.Message{Timeout: 60, Delay: 0,  Body: "The first"},
	&mq.Message{Timeout: 60, Delay: 10, Body: "The second"},
	&mq.Message{Timeout: 60, Delay: 10, Body: "The third"},
	&mq.Message{Timeout: 60, Delay: 0,  Body: "The fifth"},
)
```

**Parameters:**

* `Timeout`: After timeout (in seconds), item will be placed back onto queue.
You must delete the message from the queue to ensure it does not go back onto the queue.
 Default is 60 seconds. Minimum is 30 seconds. Maximum is 86,400 seconds (24 hours).

* `Delay`: The item will not be available on the queue until this many seconds have passed.
Default is 0 seconds. Maximum is 604,800 seconds (7 days).

--

### Get Messages from a Queue

```go
msg, err := q.Get()
fmt.Printf("The message says: %q\n", msg.Body)
```

When you pop/get a message from the queue, it is no longer on the queue but it still exists within the system.
You have to explicitly delete the message or else it will go back onto the queue after the `timeout`.
The default `timeout` is 60 seconds. Minimal `timeout` is 30 seconds.

You also can get several messages at a time:

```go
// get 5 messages
msgs, err := q.GetN(5)
```

### Touch a Message on a Queue

Touching a reserved message extends its timeout by the duration specified when the message was created, which is 60 seconds by default.

```go
msg, _ := q.Get()
err := msg.Touch()
```

There is another way to touch a message without getting it:

```go
err := q.TouchMessage("5987586196292186572")
```

--

### Release Message

```go
msg, _ := q.Get()
delay  := 30
err := msg.release(delay)
```

Or another way to release a message without creation of message object:

```go
delay := 30
err := q.ReleaseMessage("5987586196292186572", delay)
```

**Optional parameters:**

* `delay`: The item will not be available on the queue until this many seconds have passed.
Default is 0 seconds. Maximum is 604,800 seconds (7 days).

--

### Delete a Message from a Queue

```go
msg, _ := q.Get()
// perform some actions with a message here
err := msg.Delete()
```

Or

```go
err := q.DeleteMessage("5987586196292186572")
```

Be sure to delete a message from the queue when you're done with it.

--

<!--- 

TODO: IMPLEMENT IT!

### Peek Messages from a Queue

Peeking at a queue returns the next messages on the queue, but it does not reserve them.

```go
msg, err := q.Peek()

// or

messages, err := q.Peek(13) 
```

**Optional parameters:**

* `n`: The maximum number of messages to peek. Default is 1. Maximum is 100.

- -

-->

### Clear a Queue

```go
err := q.Clear()
```

<!--- 

TODO: IMPLEMENT IT!

### Add an Alert to a Queue

[Check out our Blog Post on Queue Alerts](http://blog.iron.io).

Alerts have now been incorporated into IronMQ. This feature lets developers control actions based on the activity within a queue. With alerts, actions can be triggered when the number of messages in a queue reach a certain threshold. These actions can include things like auto-scaling, failure detection, load-monitoring, and system health.

You may add up to 5 alerts per queue.

**Required parameters:**
* `type`: required - "fixed" or "progressive". In case of alert's type set to "fixed", alert will be triggered when queue size pass value set by trigger parameter. When type set to "progressive", alert will be triggered when queue size pass any of values, calculated by trigger * N where N >= 1. For example, if trigger set to 10, alert will be triggered at queue sizes 10, 20, 30, etc.
* `direction`: required - "asc" or "desc". Set direction in which queue size must be changed when pass trigger value. If direction set to "asc" queue size must growing to trigger alert. When direction is "desc" queue size must decreasing to trigger alert.
* `trigger`: required. It will be used to calculate actual values of queue size when alert must be triggered. See type field description. Trigger must be integer value greater than 0.
* `queue`: required. Name of queue which will be used to post alert messages.

**Optional parameters:**

* `snooze`: Number of seconds between alerts. If alert must be triggered but snooze is still active, alert will be omitted. Snooze must be integer value greater than or equal to 0.

```ruby
queue.add_alert({:type => "progressive",
                  :trigger => 10,
                  :queue => "my_alert_queue",
                  :direction => "asc",
                  :snooze => "0"
                 })
queue.clear #  => #<IronMQ::ResponseBase:0x007f95d3b25438 @raw={"msg"=>"Updated"}, @code=200>
```

- -

-->

## Push Queues

IronMQ push queues allow you to setup a queue that will push to an endpoint, rather than having to poll the endpoint. 
[Here's the announcement for an overview](http://blog.iron.io/2013/01/ironmq-push-queues-reliable-message.html). 

### Update a Message Queue

```go
queueInfo := mq.QueueInfo{
	//...
} 
info, err := q.Update(queueInfo);
```

QueueInfo struct consists of following fields:

```go
type QueueInfo struct {
	PushType      string            `json:"push_type,omitempty"`
	RetriesDelay  int               `json:"retries,omitempty"`
	Retries       int               `json:"retries_delay,omitempty"`
	Subscribers   []QueueSubscriber `json:"subscribers,omitempty"`
	// and some other fields not related to push queues
}
```

**The following parameters are all related to Push Queues:**

* `push_type`: Either `multicast` to push to all subscribers or `unicast` to push to one and only one subscriber. Default is `multicast`.
* `retries`: How many times to retry on failure. Default is 3. Maximum is 100.
* `retries_delay`: Delay between each retry in seconds. Default is 60.
* `subscribers`: An array of `QueueSubscriber` 
This set of subscribers will replace the existing subscribers.
To add or remove subscribers, see the add subscribers endpoint or the remove subscribers endpoint.

QueueSubscriber has the following structure:

```go
type QueueSubscriber struct {
	URL     string            `json:"url"`
	Headers map[string]string `json:"headers,omitempty"`
}
```

--

### Set Subscribers on a Queue

Subscribers can be any HTTP endpoint. `push_type` is one of:

* `multicast`: will push to all endpoints/subscribers
* `unicast`: will push to one and only one endpoint/subscriber

```go
subscription := mq.Subscription {
	PushType: "multicast",
	Retries:  3,
	RetriesDelay: 60,
}
err := q.Subscribe(
	subscription, 
	"http://mysterious-brook-1807.herokuapp.com/ironmq_push_3", 
	"http://mysterious-brook-1807.herokuapp.com/ironmq_push_4")
```

--

<!--- 

TODO: IMPLEMENT IT

### Add/Remove Subscribers on a Queue

```ruby
queue.add_subscriber({:url => "http://nowhere.com"})

queue.add_subscribers([
  {:url => 'http://first.endpoint.xx/process'},
  {:url => 'http://second.endpoint.xx/process'}
])


queue.remove_subscriber({url: "http://nowhere.com"})

queue.remove_subscribers([
  {:url => 'http://first.endpoint.xx/process'},
  {:url => 'http://second.endpoint.xx/process'}
])
```

- -

-->

### Get Message Push Status

After pushing a message:

```go
subscribers, err := message.Subscribers()
```

Returns an array of subscribers with status.

--

### Revert Queue Back to Pull Queue

If you want to revert you queue just update `push_type` to `'pull'`.

```ruby
q.Update(mq.QueueInfo{
	PushType: "pull",	
});
```

--

<!---

TODO: IMPLEMENT IT AND REWRITE IT FOR GO (COPYPASTED FROM RUBY CLIENT LIB)

## Queue Alerts

Queue Alerts allow you to set queue's size levels which are critical
for your application. For example, you want to start processing worker
when queue size grows from 0 to 1. Then add alert of `type` "fixed",
`direction` "asc", and `trigger` 1. In this case, if queue size changed
from 0 to 1 alert message will be put on queue, set by `queue`
parameter of alert's hash. If you want to prevent alerts to be put onto
alert queue in some time after previous alert message - use `snooze`
parameter. For example, to make alert silent in one hour, set `snooze`
to 3600 (seconds).

**Note:** alerts feature are only avalable for Pull (or regular) Queues.

See [Queue Alerts](http://dev.iron.io/mq/reference/queue_alerts/) to learn more.

### Alerts Parameters

Alerts can be configured with the following parameters:

* `type`: string, required. Type of alert. Available types are "fixed"
  and "progressive".
* `direction`: string, optional. Direction of queue fluctuations.
  Available directions are "asc" (alert will be triggered if queue
  size grows) and "desc" (alert will be triggered if queue size
  decreases). Defaults to "asc".
* `trigger`: integer, required. Value which is used to calculate
  actual queue size when alert will be triggered. In case of "fixed"
  type of alert `trigger` itself represents actual queue size. When
  type of alert is "progressive", actual queue sizes are calculated by
  `trigger * N`, where `N` is integer greater than 0. For example,
  type is "progressive" and trigger is 100. Alert messages will be put
  on queue at sizes 100, 200, 300, ...
* `queue`: string, required. Name of a queue which receives alert messages.
* `snooze`: integer, optional. Represents number of seconds alert will
  be silent after latter message, put onto alert queue.

**Note:** IronMQ backend checks for alerts duplications each time you
  add new alerts to a queue. It compares `type`, `direction`, and
  `trigger` parameters to find duplicates. If one or more of new
  alerts duplicates existing, backend return `HTTP 400` error, message
  will be `{"msg": "At least one new alert duplicates current queue alerts."}`.

### Add Alerts to a Queue

To add single alert to a queue.

```ruby
queue.add_alert({
  :type => 'fixed',
  :direction => 'asc',
  :trigger => 1,
  :queue => 'alerts-queue',
  :snooze => 600
})
# => #<IronMQ::ResponseBase:0x007f8d22980420 @raw={"msg"=>"Alerts were added."}, @code=200>
```

To add multiple alerts at a time.

```ruby
queue.add_alerts([
  {
    :type => 'fixed',
    :direction => 'desc',
    :trigger => 1,
    :queue => 'alerts-queue'
  },
  {
    :type => "progressive",
    :trigger => 1000,
    :queue => 'critical-alerts-queue'
  }
])
# => #<IronMQ::ResponseBase:0x00abcdf1980420 @raw={"msg"=>"Alerts were added."}, @code=200>
```

### Remove Alerts from a Queue

To remove single alert by its ID.

```ruby
queue.remove_alert({ :id => '5eee546df4a4140e8638a7e5' })
# => #<IronMQ::ResponseBase:0x007f8d229a1878 @raw={"msg"=>"Alerts were deleted."}, @code=200>
```

Remove multiple alerts by IDs.

```ruby
queue.remove_alerts([
  { :id => '53060b541185ab3eaf04c83f' },
  { :id => '99a50b541185ab3eaf9bcfff' }
])
# => #<IronMQ::ResponseBase:0x093b8d229a18af @raw={"msg"=>"Alerts were deleted."}, @code=200>
```

### Replace and Clear Alerts on a Queue

Following code sample shows how to replace alerts on a queue.

```ruby
queue.replace_alerts([
  {
    :type => 'fixed',
    :trigger => 100,
    :queue => 'alerts'
  }
])
# => #<IronMQ::ResponseBase:0x00008d229a16bf @raw={"msg"=>"Alerts were replaced."}, @code=200>
```

To clear alerts on a queue.

```ruby
queue.clear_alerts
# => #<IronMQ::ResponseBase:0x87ad13ff3a18af @raw={"msg"=>"Alerts were replaced."}, @code=200>
```

**Note:** `Queue#clear_alerts` is a helper, which represents
  `Queue#replace_alerts` call with empty `Array` of alerts.

-->

## Further Links

* [IronMQ Overview](http://dev.iron.io/mq/)
* [IronMQ REST/HTTP API](http://dev.iron.io/mq/reference/api/)
* [Push Queues](http://dev.iron.io/mq/reference/push_queues/)
* [Other Client Libraries](http://dev.iron.io/mq/libraries/)
* [Live Chat, Support & Fun](http://get.iron.io/chat)

-------------
© 2011 - 2014 Iron.io Inc. All Rights Reserved.
