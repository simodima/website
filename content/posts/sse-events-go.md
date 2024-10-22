---
title: 'Deliver notifications in the user browser'
date: 2021-11-12T15:25:11+02:00
draft: false
tags:
   - events
   - server-sent-events 
   - golang
---

Delivering real-time notifications in the user browser makes your application way more engaging and let the user quickly react to the events happening in the product.<!--more-->

--------------------------------

**This story starts in a meeting with your team**

You currently work for a successful e-commerce company and you’re in the middle of a grooming meeting when the product owner asks you to implement the invoice PDF generation

You know that invoice rendering is a very long-running job, it involves several API calls to fetch all the required data.

The invoice generation is something like:

```go
type Invoice struct {
    model.User
    model.Order
    model.Payment
}

g, ctx := errgroup.WithContext(ctx)
invoice := Invoice{}

g.Go(func() error {
    user, err := userService.Get("user-id")
    if err == nil {
        invoice.User = user 
    }
    return err
})

g.Go(func() error {
    order, err := orderService.Get("order-id")
    if err == nil {
        invoice.Order = order
    }
    return err
})

g.Go(func() error {
    payment, err := paymentService.Get("payment-id")
    if err == nil {
        invoice.Payment = payment
    }
    return err
})

if err := g.Wait(); err != nil {
    return nil, err
}

pdf, err := pdfInvoiceRendered.Render(invoice)

if err != nil {
    return nil, err
}

return pdf, nil
```

To generate an invoice the invoice service needs to do several API calls to the external services that own the Payment, User, Order and so on... After that you can render the PDF using the engine of your choice; You know, all that process may take lots of time to complete the job.

So after a brainstorming, your team decides that it's the right time to implement something to send a notification to the user. 🥳

If you want to jump straight to the solution go to [Notify]({{< ref "sse-events-go.md#notify" >}}). 

### Sending notifications to the browser

There are several ways to implement a notification system for a web application. The easy way is to go through a [polling](https://en.m.wikipedia.org/wiki/Polling_(computer_science)) strategy.
Polling is not beautiful nor efficient because you need to get if there are new notifications at regular intervals and even worse if you have a lot of users you flood your server with huge traffic.
Despite of all the cons, a polling strategy can be good for small products with relatively low traffic.

**Polling**

- it's easy to implement
- do not require a persistent connection

## A more sophisticated solution

If you’re still reading this post maybe you’re interested in a more sophisticated solution than the polling one. So we'll go deeper into the rabbit hole.

### Persistent connection

One of the most significant features that makes a polling implementation simple is that it do not rely on a persistent connection between the client and the server.
Basically, you just need to implement a `GET /notifications` API and let the client call it to fetch new notifications.
The state is preserved in the server where we've hundreds of possibilities to store the notifcations; we can use a database, a cache, the filesystem, or whatever system we want.

On the other hand, if we don't want to flood the server with hundreds of requests to ask for notifications we need to set up a stable communication channel between the client and the server. Talking about HTTP this channel could be a persistent connection to be used to send messages to the client.

```bash
➜  ~ curl -v --keepalive http://localhost:3000/open?channel=test
*   Trying ::1...
* TCP_NODELAY set
* Connected to localhost (::1) port 3000 (#0)
> GET /open?channel=test HTTP/1.1
> Host: localhost:3000
> User-Agent: curl/7.64.1
> Accept: */*
>
< HTTP/1.1 200 OK
< Cache-Control: no-cache
< Connection: keep-alive
< Content-Type: text/event-stream
< Date: Fri, 12 Nov 2021 15:52:22 GMT
< Transfer-Encoding: chunked
```

The [HTTP / 1.1](https://en.wikipedia.org/wiki/HTTP_persistent_connection) version of the protocol supports the persistent connections, basically every connection is treated as persistent unless the client (or the server) sends a `Connection: close` header.
This can be a very useful approach to keep the number of newly opened connection under control, even more so we have thousands of clients that need to receive notifications and constantly ping the server to ask for them.

### Server Sent Events - SSE

Beside the classic HTTP request <-> response flow that allows the client to send a request to the server we also have the server sent events in our swiss army knife.
**Server sent events** allows the server to push new data to the browser as long as they are ready.

### Implementing in golang

Thanks to the powerful stdlib it's very easy to implement SSE in golang, you can write data over the response writer and flush it to send them to the client.

```go
func(w http.ResponseWriter, r *http.Request) {
	w.Header().Set("Access-Control-Allow-Origin", "*")
	w.Header().Set("Access-Control-Allow-Headers", "Content-Type")
	w.Header().Set("Content-Type", "text/event-stream")
	w.Header().Set("Cache-Control", "no-cache")
	w.Header().Set("Connection", "keep-alive")
    if f, ok := w.(http.Flusher); ok {
        f.Flush()
    }

	go func() {
		for {
			time.Sleep(time.Second)

			fmt.Fprint(w, "event: test\n")
			fmt.Fprint(w, "data: {\"data\": 1}\n\n")

			// Flush the write to actually send data to the client
			if f, ok := w.(http.Flusher); ok {
				f.Flush()
			}
		}
	}()

	<-r.Context().Done()
}
```

But in a real-world scenario that snippet could be not enough, you may need a more sophisticated solution for a cloud-native app that requires:

- horizontal scaling through containers
- release new versions with a [blue/geen deployment](https://en.wikipedia.org/wiki/Blue-green_deployment)

### Notify {#notify}

Notify is a golang library that condensates all these concepts, it allows to **send server events** to the browser with ease.
https://github.com/toretto460/notify is easy to use; it supports Redis as a backend to publish and subscribe to events.

[https://github.com/toretto460/notify](https://github.com/toretto460/notify/blob/main/example/web/main.go)

```go
package main

import (
	"context"
	"log"
	"net/http"
	"time"

	"github.com/go-redis/redis/v8"

	"github.com/toretto460/notify"
    "github.com/toretto460/notify/model"
)

var redisCli *redis.Client

func init() {
	redisCli = redis.NewClient(&redis.Options{
		Addr: "localhost:6379",
	})
}

func main() {  
  // Get the channel factory
	chFactory := notify.Redis(redisCli)
  
  // Register the default handler
	http.Handle("/listen", notify.DefaultHandler(chFactory))
  
 
  // Send some messages
  go func() {
		channel, _ := chFactory.Get("test-channel-id")
		channel.Send(
			context.Background(),
			model.NewMessage("test-message", []byte(`{"hello": "world"}`)),
		)
		time.Sleep(time.Second * 10)
	}()
  
	log.Print("Starting web server at :3000")

	if err := http.ListenAndServe(":3000", nil); err != nil {
		log.Fatal(err)
	}
}
```

And then you can listen for messages in the browser

```js
const channel = "test-channel-id"
const source = new EventSource("http://localhost:3000/listen?channel=" + channel)

source.addEventListener('test-message', (event) => {
	console.log(event) // => {"hello": "world"}
})
```

Disclaimer

As of today 11/11/2021 the notify library is still not used in production. Fill a PR or open a discussion here [https://github.com/toretto460/notify/issues](https://github.com/toretto460/notify/issues) for any issue.


Cover photo by [Adam Solomon](https://unsplash.com/@solomac?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText) on [Unsplash](https://unsplash.com/s/photos/notification?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText)
