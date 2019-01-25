---
title: "How It Works"
description: ""
images: []
draft: true
menu: main
weight: 20
---

# The 10000 Foot View

With the advent of web components, real-time, and rich web applications, maintaining these technologies can easily put a strain on your team. Puncture is here to make this process accessible by providing the similar component-based development pattern on the server, as you would have on the client. You can stop manually wiring AJAX requests, and start focusing on what really needs to be programmed.

# Example

Below are code examples, one in PHP and one in JavaScript / HTML. The magic happens with the Puncture **Mixin** class. This class triggers data syncing between server and client. So in the code ```@click="puncture('addClick')"```, this calls the PHP class HelloWorldComponent#addClick which entails changes the number of clicks. The changed data is then sent back to the client.

## Server Side Code (PHP)
{{< highlight php >}}
<?php
class HelloWorldComponent implements ComponentInterface
{
    use \Puncture\ComponentTrait;

    public static $componentName = 'hello-world';
    
    public $clicks = 0;
    
    public function addClick(Manager $manager, $params)
    {
        $this->clicks++;
    }
}
{{< /highlight >}}

## Client Side Code (VueJS)
{{< highlight html >}}
<template>
    <div>
        <p>Number of clicks: {{ clicks }}</p>
        <button class="button" @click="puncture('addClick')">Add Click</button>
    </div>
</template>
<script type="text/ecmascript-6">
    import {PunctureMixin} from 'puncture';

    export default {
        name: 'HelloWorld',
        components: {},
        mixins: [PunctureMixin]
    }
</script>
{{< /highlight >}}

# Data Synchronization

With client and server data synchronization, you focus on your code rather than management of requests. On top of that only the changed data is ever transmitted. This speeds up every request as they become very light weight. You decide when the client and server will sync, and are not bound to one way.

# Realtime

Websockets have been around for a while now. Puncture works with both tradditional HTTP servers and PHP event loop libraries. At this time <a href="https://github.com/amphp/websocket-client" target="_blank">AMP/Websocket</a> is supported. By supporting both transport methods you can deploy this on shared hosting, or in the cloud for more responsive experience. As a note while websockets are available in PHP, you then need to commit to asyncronous programming techniques.

# Cross-Component Communication

There will be times where data between components need to be shared. An example is one component has a list of users and another component contains a user from that list. If either component is altered it both components' data should be syncronized. This is accomplished by publish and subscribing to channels. This allows you to send messages between components such as a user being updated.

## Component Subscriber Example
{{< highlight php >}}
<?php
/**
 * Subscriber class to listen for messages
 *
 * @var $pubsub \Puncture\PubSub
 */
$pubsub = $manager->getContainer()->get(PubSub::class);

//-- Setup listener for this account when modified
$pubsub->subscribe($manager, 'accounts:' . $account['id'], $this->getComponentId(), self::class);

//-- Pass the account data from this component into this new component
$manager->createComponent(AccountComponent::class, $account);
{{< /highlight >}}

## Component Pushblisher Example

{{< highlight php >}}
<?php
/**
 * Subscriber class to listen for messages
 * 
 * @var $pubsub \Puncture\PubSub
 */
$pubsub = $manager->getContainer()->get(PubSub::class);

//-- Prepare account data
$alteredAccount = [
    'id'         => $this->id,
    'first_name' => $this->first_name,
    //...
];

//-- Send the account as a message to any subscriber listening to this account channel
$pubsub->publish($manager, 'accounts:' . $this->id, $alteredAccount);

//-- Remove this component
$manager->deleteComponentById($this->getComponentId());
{{< /highlight >}}

# Lightweight

Since data is the only part that is managed by the server, and not the template engine, this frees up a lot more server processing power. 