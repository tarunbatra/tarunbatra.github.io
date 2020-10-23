---
title: Meteor Publications and Subscriptions
date: 2016-07-21 00:30:00
tags:
  - meteor
  - javascript
category:
  - javascript
photos: /data/images/Meteor-Publications-and-Subscriptions/cover.jpeg
canonical_url: https://codebrahma.com/meteor-publications-and-subscriptions/
---
<sub>Originally published for [Codebrahma](https://codebrahma.com/meteor-publications-and-subscriptions/) on July 21, 2016.</sub>
<br>
[Meteor](https://www.meteor.com/) is a full-stack JavaScript platform, in fact the [11th most popular](http://stats.js.org/) JavaScript project on GitHub at the time of writing. What makes Meteor so disruptive is the mode of data communication between server and client. It's not [RESTful](https://en.wikipedia.org/wiki/Representational_state_transfer) but instead, Meteor uses Publish Subscribe pattern to communicate data between server and client. The protocol used for this communication is Distributed Data Protocol (DDP), which is built in-house by The [Meteor Development Group (MDG)](https://www.meteor.com/company), the startup behind Meteor.

In this chapter specifically, we're going to talk about meteor **publications** and **subscriptions**, and to explain these interwoven concepts, we're going to use them in various scenarios and analyze their results using 3rd party tools to achieve a better understanding of how Meteor publications and subscriptions work and the ways we can use them.

Almost in every case, we'd need to publish a set of data from the server and subscribe to same in the client. Unlike conventional frameworks, Meteor server doesn't serve HTML content, but the data which is used by the client to render the HTML. This feature is called **data on the wire**. The server sends a set of data to the client initially and then keep pushing new data as it changes in the database.

_Code to publish all documents from a collection of posts_

```js
if (Meteor.isServer) {
    var Posts = new Mongo.Collection('posts');

Meteor.publish('listAllPosts', function () {
    return Posts.find();
    });
}
```

_Code to subscribe for all documents from a collection of posts_

```js
if (Meteor.isClient) {
    var Posts = new Mongo.Collection('posts');

    Meteor.subscribe('listAllPosts', {

    onReady: function () {
    // called when data is ready to be fetched
    console.log(Posts.find().fetch());
    },

    onStop: function () {
    // called when data publication is stopped
    }
    });
}
```

If you're new to Meteor, you'd already be sweating seeing that we just created a Mongo Collection on the client. You're right, it doesn't make sense.

We actually created a **MiniMongo** Collection. MiniMongo is a client-side data store with a Mongo-like API. When we talk about server sending data to client, it's MiniMongo we're talking about.

!['Minimongo'][minimongo]

Subscribed data in MiniMongo visualized using [Meteor DevTools](https://github.com/thebakeryio/meteor-devtools) showing the 2 posts that were pushed to MiniMongo

When a subscription becomes ready, the data is stored in MiniMongo and can be accessed by Mongo-like queries. The server keeps the MiniMongo in-sync with the data in back-end by sending additional data as the data changes in the back-end.

!['DDP Monitor'][ddp-monitor]

Syncing of MiniMongo data through DDP showing 1 item being pushed to MiniMongo as it got added on the server

!['MiniMongo synced'][minimongo-synced]

Updated MiniMongo data showing total 3 posts

## Readying a subscription

As shown in the above code, when the subscription becomes ready, `onReady` callback is called. But when does the subscription become ready?

1. When a cursor is returned from the server in a Meteor publication, Meteor automatically makes the subscription ready after pushing the data from the cursor to the MiniMongo.

```js
Meteor.publish('listAllPosts', function () {
    return Posts.find();
    });
```

2. To make the subscription ready manually, or without returning a cursor `this.ready()` is used as shown in the following code snippet.

```js
Meteor.publish('listAllPosts', function () {
    this.ready();
});
```

This informs the client that whatever records which needed to be pushed initially are pushed, and subscription is deemed ready.

## Stopping a subscription

There are use cases when you may want the data not to be pushed to the client and the client to know that the subscription was stopped for some reason. How can that be done?

1. You can manually stop a subscription by calling `this.stop()` from the publish function. This will stop the subscription and call the `onStop` callback.

```js
Meteor.publish('listAllPosts', function () {
    this.stop();
});
```

2. Another way to stop a subscription is to throw a Meteor Error from the publish function, which will result in the `onStop` callback being called with the error as the argument.

```js
//server

Meteor.publish('listAllPosts', function () {
    throw new Meteor.Error('some reason');
});

//client

Meteor.subscribe('listAllPosts', {
    onStop: function(err) {
    // handle the error
});
```

## Advanced concepts

Meteor publications and subscriptions work like a charm if used properly. Syncing data in the background and updating the same in real-time looks pretty good. But it offers much more. You can take control of what gets pushed, when and how.

To gain more control over the data you publish, you need to know what happens when Meteor publishes data. The following screenshot shows all the DDP messages which are exchanged between client and server of our sample Meteor app.

!['DDP'][ddp]

DDP messages showing the publication of data using [Arunoda](https://github.com/arunoda)'s [DDP Analyzer](https://github.com/arunoda/meteor-ddp-analyzer) tool

1. First, the client sends a DDP message to server subscribing to the `listAllPosts` publication.
2. Then the server pushes the first set of data using `added` message.
3. Next, the server readies the subscription using `ready` message.
4. Upon addition of data in the database, server pushes the new document using the same `added` message.

Now, if you're paying attention, you'd know that we used `this.ready()` earlier to ready a subscription. It's responsible for sending the `ready` message. Similarly, we have more methods which can be used to send the other kind of messages.

### Adding a document

You can use `this.added()` in a Meteor publish function to manually send an `added` message. It's signature is:

```js
this.added(collectionName, documentId, fields)
```

### Changing a document

Similarly, you can use `this.changed()` to change a document already pushed to the client. It's signature is:

```js
this.changed(collectionName, documentId, changedFields)
```

### Removing a document

To remove a document from client, Meteor provides `this.removed()` with the following signature:

```js
this.removed(collectionName, documentId)
```

_Code to publish a post and then change it's content_

```js
Meteor.publish("sendCustomPost", function () {
    var doc = {"title": "Post 4", "content": "4th post" };
    // Pushing 4th document to client
    this.added("posts", "some-object-id", doc);

    // Changing the 4th document
    this.changed("posts", "some-object-id", { "content": "custom-content" });

    // Manually readying the subscription
    this.ready();
});
```
!['DDP'][ddp-2]

DDP messages after executing the above code

So, you can control the data you publish to the client. There are more interesting concepts, like intercepting a publication and changing the data but that's out of the scope of this article.

Meteor publications and subscriptions make working with real time data easy and fun. To know more refer [Meteor Documentation](http://docs.meteor.com/) and after you're done with that, maybe [DDP specification](https://github.com/meteor/meteor/blob/devel/packages/ddp/DDP.md) too.


<br>

**❤️ code**

[ddp-monitor]: /data/images/Meteor-Publications-and-Subscriptions/ddp-monitor.png
[minimongo]: /data/images/Meteor-Publications-and-Subscriptions/minimongo.png
[minimongo-synced]: /data/images/Meteor-Publications-and-Subscriptions/minimongo-synced.png
[ddp]: /data/images/Meteor-Publications-and-Subscriptions/ddp.png
[ddp-2]: /data/images/Meteor-Publications-and-Subscriptions/ddp-2.png