---
layout: post
title: '"Spring Integration in Action" Mark Fisher, Jonas Partner, Marius Bogoevici
  and Iwein Fuld'
date: '2015-01-12T23:06:00.000+01:00'
author: Monik
tags:
- Programming
- Java
- Spring
modified_time: '2016-05-27T10:55:12.823+02:00'
blogger_id: tag:blogger.com,1999:blog-5940427300271272994.post-1611880993513646485
blogger_orig_url: http://learningmonik.blogspot.com/2015/01/spring-integration-in-action.html
commentIssueId: 25
type: book
---

<div class="bg-info panel-body" markdown="1">
These are the notes I took while reading the Manning's <a
        href="https://www.manning.com/books/spring-integration-in-action">"Spring Integration in Action" book (Mark Fisher, Jonas Partner, Marius Bogoevici, and Iwein Fuld)</a>, before taking the Spring Integration exam.<br/><br/>I found the book to be a nice guide to some of the topics, before actually starting to read the Java documentation.<br/><br/>More notes about preparation for the exam itself are
    <a href="http://learningmonik.blogspot.de/2015/01/enterprise-integration-with-spring.html"
       target="_blank">here</a>.
</div>

<h3>Table of contents</h3>
- TOC
{:toc max_level=2}

### 1. Background

#### 1.1. Enterprise Intergration Patterns - introduction

EIP are made of three base patterns:

<ul>
    <li>Message</li>
    <li>Message Channel</li>
    <li>Message Endpoint</li>
</ul>

##### Message

Message consists of:

<ul>
    <li>header - data relevant only to the messaging system</li>
    <li>payload - data to be processed</li>
</ul>

Message types:
<ul>
    <li>command message - tells the receiver to do sth</li>
    <li>event message - notifies that sth has happened</li>
    <li>document message - transfers data</li>
</ul>

The message is a representation of the contract between the sender and the receiver.

##### Message Channel

Manages how (e.g. async or sync) and where the message is delivered, but does not interact with its content. It also decouples the sender from receiver. There are two types of channels:

<ul>
    <li>point-to-point - one message is received by exactly one receiver (but not neccessarily always the same)</li>
    <li>publish-subscribe - one message received by multiple subscribers (zero or more)</li>
</ul>

##### Message Endpoints

They actually do sth with the message.

<ol>
    <li>Channel Adapter - connects an application to the messaging system. The message flow is
        <b>uni-directional</b>.<br/><span
                style="font-family: &quot;courier new&quot; , &quot;courier&quot; , monospace;">&nbsp; &nbsp; &nbsp; &nbsp; |\ &nbsp;&nbsp; &nbsp; _____<br/>APP --&gt; || --&gt; |_____| --&gt; ...<br/>&nbsp; &nbsp; &nbsp; &nbsp; |/</span>
    </li>
    <li>Messaging Gateway - used for bi-directional messaging, e.g. where we want to di asynchronous stuff behind the curtain, and the caller is aware only of sync invocation.<br/><span
            style="font-family: &quot;courier new&quot; , &quot;courier&quot; , monospace;">&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp;sync _____ &nbsp;async</span><br/><span
            style="font-family: &quot;courier new&quot; , &quot;courier&quot; , monospace;">&nbsp; &nbsp; --&gt; |\ --&gt; |_____| --&gt; ...<br/>APP &nbsp; &nbsp; || sync _____ &nbsp;async<br/>&nbsp; &nbsp; &lt;-- |/ &lt;-- |_____| &lt;-- ...</span><br/><i>(inbound gateway)</i>
    </li>
    <li>Service Activator - invokes a service based on the message and sends back the result. It's like
        <i>outbound gateway</i> but with certain purpose and within same Spring Context.<br/><span
                style="font-family: &quot;courier new&quot; , &quot;courier&quot; , monospace;">&nbsp; &nbsp; &nbsp; &nbsp; &nbsp;_____ &nbsp; &nbsp; &nbsp; ___</span><br/><span
                style="font-family: &quot;courier new&quot; , &quot;courier&quot; , monospace;">... --&gt; |_____| --&gt; | o | --&gt;<br/>&nbsp; &nbsp; &nbsp; &nbsp; &nbsp;_____ &nbsp; &nbsp; &nbsp;| &nbsp; | &nbsp; &nbsp; SERVICE<br/>... &lt;-- |_____| &lt;-- | o | &lt;--<br/>&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; '---'</span>
    </li>
    <li>Router - everyone knows, but two points to note: it does not change the message and it is aware of other channels<br/><span
            style="font-family: &quot;courier new&quot; , &quot;courier&quot; , monospace;">&nbsp; &nbsp; &nbsp; &nbsp; ,-----,<br/>&nbsp; &nbsp; &nbsp; &nbsp; | / *-| --&gt; ...<br/>... --&gt; |* &nbsp;*-| --&gt; ...<br/>&nbsp; &nbsp; &nbsp; &nbsp; |___*-| --&gt; ...</span>
    </li>
    <li>Splitter - splits the message<br/><span
            style="font-family: &quot;courier new&quot; , &quot;courier&quot; , monospace;">&nbsp; &nbsp; &nbsp; &nbsp; ,---------,</span><br
            style="font-family: 'Courier New', Courier, monospace;"/><span
            style="font-family: &quot;courier new&quot; , &quot;courier&quot; , monospace;">&nbsp; &nbsp; &nbsp; &nbsp; | &nbsp; &nbsp; &nbsp; []|</span><br
            style="font-family: 'Courier New', Courier, monospace;"/><span
            style="font-family: &quot;courier new&quot; , &quot;courier&quot; , monospace;">... --&gt; |[] --&gt; []| --&gt; ...</span><br
            style="font-family: 'Courier New', Courier, monospace;"/><span
            style="font-family: &quot;courier new&quot; , &quot;courier&quot; , monospace;">&nbsp; &nbsp; &nbsp; &nbsp; | &nbsp; &nbsp; &nbsp; []|<br/>&nbsp; &nbsp; &nbsp; &nbsp; '---------'</span>
    </li>
    <li><span>Aggregator - waits for group of correlated messages and merges them when the group is complete (needs to know correlation id of each message and group size)<br/><span
            style="font-family: &quot;courier new&quot; , &quot;courier&quot; , monospace;">&nbsp; &nbsp; &nbsp; &nbsp; ,---------,</span><br
            style="font-family: 'Courier New', Courier, monospace;"/><span
            style="font-family: &quot;courier new&quot; , &quot;courier&quot; , monospace;">&nbsp; &nbsp; &nbsp; &nbsp; |[] &nbsp; &nbsp; &nbsp; |</span><br
            style="font-family: 'Courier New', Courier, monospace;"/><span
            style="font-family: &quot;courier new&quot; , &quot;courier&quot; , monospace;">... --&gt; |[] --&gt; []| --&gt; ...</span><br
            style="font-family: 'Courier New', Courier, monospace;"/><span
            style="font-family: &quot;courier new&quot; , &quot;courier&quot; , monospace;">&nbsp; &nbsp; &nbsp; &nbsp; |[] &nbsp; &nbsp; &nbsp; |<br/>&nbsp; &nbsp; &nbsp; &nbsp; '---------'</span></span>
    </li>
</ol>

#### 1.2 Event-driven architecture

<div>
    <span>SEDA - Stacked Event-Driven Architecture, i.e. when the events are not only exchanged but also buffered, e.g. in a queue; this can be good idea in case of fluctuating load on the system; it's up to you to choose between EDA and SEDA style;</span>
</div>
<div><span><br/></span></div>
<div>
    <span>(S)EDA helps reducing all kind of coupling, that's why Spring Integration builds around it. But what is <b>coupling</b>? There are two types of coupling, type-level coupling (the one connected with dependency injection, that everyone knows), but also system-level coupling, like e.g. temporal coupling (an external service which is not available freezes all the system).</span>
</div>
<div><span><br/></span></div>

#### 1.3. Integration styles

Actually Spring focuses on number 4, but also supports the other ones.

<ol>
    <li>File based - no transactions, no message medatada, no atomicity; though sometimes can be used</li>
    <li>Shared database - atomic operations, data consistency, domain model specified =&gt; extra coupling<br/>- staging tables, for transferring data in steps<br/>- sharing data
    </li>
    <li>Remote procedure calls - tries to hide the fact that different services are running on different systems; has to serialize objects (XML, Java), network is not always reliable, there is nothing in the middle to take care of assuring delivery</li>
    <li>Message based integration - small data packets are exchanged frequently between endpoints via channels, in async manner</li>
</ol>

### 2. Messaging in Spring Integration

#### 2.1. Message

<ul>
    <li>metadata is in message's <b>headers</b></li>
    <li>the message is immutable! use MessageBuilder</li>
</ul>

#### 2.2. Channels

The main interface goes like this:

<div><span style="font-family: &quot;courier new&quot; , &quot;courier&quot; , monospace;">&lt;MessageChannel&gt;<br/>send (m)<br/>send (m, timeout)</span>
</div>
<div><br/></div>
<div>It's extended with interfaces of <b>two types</b>:</div>
<div><br/></div>
<div><span
        style="font-family: &quot;courier new&quot; , &quot;courier&quot; , monospace;">&lt;SubscribableChannel&gt;</span>
</div>
<div><span
        style="font-family: &quot;courier new&quot; , &quot;courier&quot; , monospace;">subscribe (messageHandler)<br/>unsubscribe (messageHandler)</span>
</div>
<div><br/></div>
<div><span
        style="font-family: &quot;courier new&quot; , &quot;courier&quot; , monospace;">&lt;PollableChannel&gt;</span>
</div>
<div><span style="font-family: &quot;courier new&quot; , &quot;courier&quot; , monospace;">receive ()</span></div>
<div><span style="font-family: &quot;courier new&quot; , &quot;courier&quot; , monospace;">receive (timeout)</span>
</div>
<div><span style="font-family: &quot;courier new&quot; , &quot;courier&quot; , monospace;"><br/></span></div>
<div>
    <span>The default configuration in sync message transmission and subscribable channels (don't confuse with pub-sub channel). You should adjust it to your needs. For example in the book they have a workflow which finishes with sending an email, which can be done async, as email does not have to be immediately sent. Making it async is just changing the channel type from direct to queue.</span>
</div>

##### Bridge

<span>With the email example we also want to have more consumers, so we change the channel which puts stuff in the queue to publish-subscribe-channel, and add a <b>bridge </b>right after it and before the queue. This is done so that the pub-sub channel can handover the messages to another thread in the bridge, which guarantees the pub-sub channel does not run out of threads from its thread pool. (I don't get why the threads would get exhausted but ok)</span>

##### Priority queue

It's easy to change a channel into a priority queue by adding "priority-queue" XML tag inside. It has a reference to PriorityComparator.

##### Channel collabolators

They are MessageDispatchers and ChannelInterceptors, which says nothing now, but I promise it's the last additional piece of information on component types.

##### Message Dispatcher

Is a thing which decides what happens once a message arrives at a channel which was a subscribable channel.

<span
            style="font-family: &quot;courier new&quot; , &quot;courier&quot; , monospace;">&lt;MessageDispatcher&gt;</span><br/><span
            style="font-family: &quot;courier new&quot; , &quot;courier&quot; , monospace;">bool addHandler (messageHandler)</span><br/><span
            style="font-family: &quot;courier new&quot; , &quot;courier&quot; , monospace;">bool removeHandler (messaheHandler)</span><br/><span
            style="font-family: &quot;courier new&quot; , &quot;courier&quot; , monospace;">bool dispatch (m)</span><br/>It can either do the "competing consumers" thing or broadcasting. And correspondigly, we have UnicastingDispatcher and BroadcastingDispatcher. If we choose competing consumers we have to define competition rules by adding reference to a LoadBalancingStrategy.<br/><br/><span
            style="font-family: &quot;courier new&quot; , &quot;courier&quot; , monospace;">&lt;LoadBalancingStrategy&gt;</span><br/><span
            style="font-family: &quot;courier new&quot; , &quot;courier&quot; , monospace;">Iterator&lt;MessageHandler&gt; getHandlerIterator(m, List&lt;handlers&gt;)</span><br/><span
            style="font-family: &quot;courier new&quot; , &quot;courier&quot; , monospace;"><br/></span>The available implementation choice is one: RoundRobinLoadBalancingStrategy.&nbsp;<span
           >Yes, in round robin the message is not really important in the method, but if you wanna implement crazy rules, with this interface you can (though they say you typicaly shouldn't).</span><br/><span
           ><br/></span><span
           >They didn't show XML definition. I think normally it's done under the hood so you don't touch this. It's just to know how things work internally.</span><br/>

##### Channel Interceptor

<span           >Seems like extra piece of functionality. You can intercept or filter messages.</span><br/><span
           ><br/></span><span
            style="font-family: &quot;courier new&quot; , &quot;courier&quot; , monospace;">&lt;ChannelInterceptor&gt;</span><br/><span
            style="font-family: &quot;courier new&quot; , &quot;courier&quot; , monospace;">&nbsp; &nbsp;m preSend (m, channel)</span><br/><span
            style="font-family: &quot;courier new&quot; , &quot;courier&quot; , monospace;">&nbsp; &nbsp; &nbsp;postSend (m, channel, wasSent)</span><br/><span
            style="font-family: &quot;courier new&quot; , &quot;courier&quot; , monospace;">bool preReceive (channel)</span><br/><span
            style="font-family: &quot;courier new&quot; , &quot;courier&quot; , monospace;">&nbsp; &nbsp;m postReceive (m, channel)</span><br/><span
            style="font-family: &quot;courier new&quot; , &quot;courier&quot; , monospace;"><br/></span>The "receive" methods are available only for pollable channels. In general, return null instead of message to break the processing. In "preReceive" return false to break the processing, this is called before the message is eve read, that's why it does not deal with the message.<br/><br/>We add interceptors like this:<br/><span
            style="font-family: &quot;courier new&quot; , &quot;courier&quot; , monospace;">&lt;channel ..&gt;</span><br/><span
            style="font-family: &quot;courier new&quot; , &quot;courier&quot; , monospace;">&nbsp; &lt;interceptors&gt;</span><br/><span
            style="font-family: &quot;courier new&quot; , &quot;courier&quot; , monospace;">&nbsp; &nbsp; &lt;beans:ref="...&gt;</span><br/><span
            style="font-family: &quot;courier new&quot; , &quot;courier&quot; , monospace;">...</span><br/><span
            style="font-family: &quot;courier new&quot; , &quot;courier&quot; , monospace;"><br/></span><span
           >There are Spring examples of interceptors.</span><br/>

<ul>
    <li>
        <i>WireTap</i> - non invasively copies the message to another processing path (async); it is also possible to define it globally but they didn't show how;
    </li>
    <li>
        <i>MessageSelectingInterceptor </i>that uses a MessageSelector (bool accept(m)), e.g. PayloadTypeSelector, which allows to implement a Datatype Channel EI pattern; (don't confuse with MessageFilter which does exactly same but as a message endpoint)
    </li>
</ul>

#### 2.3 Message Endpoints
<div>Each message endpoint is an implementation of MessageHandler(.handleMessage(m)), wrapped in an adapter, which connects the endpoint to the channel. Depending of what kind of channel it is, the adapter will have to add appropriate capabilities:<br/>
    <ul>
        <li>if channel is subscribable, the endpoint will be invoked by channel's TaskExecutor - not much to do</li>
        <li>if channel is pollable, it needs to be polled by the endpoint</li>
        <li>if channel is bidirectional, ie expects a reply, then the endpoint should provide it of course</li>
        <li>...</li>
    </ul>
    <div>Below is a nice table from the book, which gives the overview of all the options:</div>
    <div><br/></div>
    <div><span
            style="font-family: &quot;courier new&quot; , &quot;courier&quot; , monospace; font-size: xx-small;"><b>Endpoint &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp;Polling/ &nbsp; &nbsp; &nbsp;Inbound/ &nbsp; Direction &nbsp;Internal/</b></span>
    </div>
    <div><span
            style="font-family: &quot;courier new&quot; , &quot;courier&quot; , monospace; font-size: xx-small;"><b>&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; Event driven &nbsp;Outbound &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp;External</b></span>
    </div>
    <div><span
            style="font-family: &quot;courier new&quot; , &quot;courier&quot; , monospace; font-size: xx-small;">inbound-channel-adapter &nbsp; poll &nbsp; &nbsp; &nbsp; &nbsp; &nbsp;in &nbsp; &nbsp; &nbsp; &nbsp; uni &nbsp; &nbsp; &nbsp; &nbsp;int</span>
    </div>
    <div><span
            style="font-family: &quot;courier new&quot; , &quot;courier&quot; , monospace; font-size: xx-small;">outbound-channel-adapter &nbsp;both &nbsp; &nbsp; &nbsp; &nbsp; &nbsp;out &nbsp; &nbsp; &nbsp; &nbsp;uni &nbsp; &nbsp; &nbsp; &nbsp;int</span>
    </div>
    <div><span
            style="font-family: &quot;courier new&quot; , &quot;courier&quot; , monospace; font-size: xx-small;">gateway &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; event &nbsp; &nbsp; &nbsp; &nbsp; in &nbsp; &nbsp; &nbsp; &nbsp; bi &nbsp; &nbsp; &nbsp; &nbsp; int</span>
    </div>
    <div><span
            style="font-family: &quot;courier new&quot; , &quot;courier&quot; , monospace; font-size: xx-small;">service-activator &nbsp; &nbsp; &nbsp; &nbsp; both &nbsp; &nbsp; &nbsp; &nbsp; &nbsp;out &nbsp; &nbsp; &nbsp; &nbsp;bi &nbsp; &nbsp; &nbsp; &nbsp; int</span>
    </div>
    <div><span
            style="font-family: &quot;courier new&quot; , &quot;courier&quot; , monospace; font-size: xx-small;">http:outbound-gateway &nbsp; &nbsp; both &nbsp; &nbsp; &nbsp; &nbsp; &nbsp;out &nbsp; &nbsp; &nbsp; &nbsp;bi &nbsp; &nbsp; &nbsp; &nbsp; ext</span>
    </div>
    <div><span
            style="font-family: &quot;courier new&quot; , &quot;courier&quot; , monospace; font-size: xx-small;">amqp:inb-channel-adapter &nbsp;event &nbsp; &nbsp; &nbsp; &nbsp; in &nbsp; &nbsp; &nbsp; &nbsp; uni &nbsp; &nbsp; &nbsp; &nbsp;ext</span>
    </div>
    <div><span
            style="font-family: &quot;courier new&quot; , &quot;courier&quot; , monospace; font-size: xx-small;"><br/></span>
    </div>
    <div>
        <ul>
            <li><span>inbound/outbound - from perspective of Spring Integration application</span>
            </li>
            <li><span>internal/external - with respect to application context</span>
            </li>
        </ul></div></div>

##### Polling vs not polling

<div>Remember that:&nbsp;</div>
<div>
    <ul>
        <li>polling endpoint =
            <b>asychronous </b>message hand-off (as you don't know when the messahe arrives so you have to poll in time intervals)
        </li>
        <li>event-driven endpoint = <b>synchronous </b>message hand-off</li>
    </ul>
    <div>For example a web collaboration tool (like Google Docs) should use async hand-off with polling, so that sending an event never blocks the typing (as seeing my own typing is more important than seeing the changes of others).</div>
</div>
<div>
    <ul>
        <li>polling endpoint - polls for new messages periodically, and manages thread(s) which do(es) that; Spring automatically wraps any passive endpoint connected to a PollableChannel into a polling endpoint;</li>
        <li>event-driven endpoint - used in most cases, such endpoint does not take responsibility for thread management; if the subscribable channel sending to this endpoint does not want to wait for message hand-off it can still use a thread pool, but then the trasaction boundary will be broken</li>
    </ul>
    <div>* gateway is always performing sync communication; if you need an async gateway, combine two async channel adapters;</div>
</div>
<div>* REPLY_CHANNEL header is used to know who to return the message to, in case of bidirectional communication;</div>
<div><br/></div>

##### Transaction boundaries

<div>Always when there is an async hand-off (queue, task executor, aggregator), the transaction boundaries and security context continuity are broken.&nbsp;</div>
<div><br/></div>
<div>* Don't try to workaround by putting transaction context in message header, this limitation is there for a reason of promoting good design (but you can use this trick with security context).</div>
<div><br/></div>
<div>Transactions usually start at the poller (if it was cinfugured with transaction manager), when it pools from:</div>
<div>
    <ul>
        <li>a message source - if the message source is transactional, it participates in same transaction</li>
        <li>a queue channel</li>
    </ul>
    <div>The transaction lasts until the message is sent to a something that does not maintain same thread.</div>
</div>
<div><br/></div>

##### Under the hood

<div>All these things you don't have to know as Spring will choose the right endpoint adapters configuration automatically. It has an AbstractConsumerEndpointParser, which parses te XML definition and puts factories (ConsumerEndpointFactoryBean) in place of endpoints, and those factories will know on runtime what kind of adapter is needed for an endpoint. We have PollingConsumer and EventDrivenConsumer adapters.</div>

<div>We also have a Lifecycle object that has upgraded version called SmartLifecycle, which automatically starts the endpoints by calling "subscribe" on the channel, or schedules a poller task. The Lifecycle is bound to Spring Application Context.</div>

### 3. Building Messaging Systems

#### 3.1. Separation of concerns

<div>Is important. AOP did that, IoC did, so Spring Integration also strives to do it: sepration of business logic from integration concerns. Shortly what they point out:</div>

##### Transformers

<ul>
    <li>when you design your business domain don't keep the intergration in mind, it should be independent; a dto should not have a "toXml()" method, rather add an appropriate transformer in your configuration later;</li>
    <li>later you can even build transformers that will call other services to build the message, that's exactly fine;</li>
    <li>a such transformer you'd annotate with @MessageEndpoint, which is also a stereotype for @Component;</li>
    <li>testing of a workflow is done by @Autowiring the channels and sending a message using MessageBuilder;</li>
    <li>normally for every endpoint one of the two is required: output-channel, or reply_to header on the message; if none is set, exception is thrown;</li>
    <li>&lt;mail:header-enricher&gt;&lt;mail:to expression="payload?.emailAddress"/&gt;..</li>
</ul>

##### Service Activators

<ul>
    <li>if the output-channel is not defined, then the service activator will use the message's reply_to header's value to send the reply;</li>
    <li>so switching between chained services and request-reply model is a matter of adding/removing the output-channel from XML</li>
    <li>you can provide&nbsp;output-channel="nullChannel" to ignore the response</li>
</ul>

##### Interceptors that publish messages

<ul>
    <li>@Publisher(channel="targetChannel") - method will additionally publish result to that channel; it's an interceptor;</li>
</ul>

##### Messaging Gateways

<ul>
    <li>if we wanna publish to multiple, we use a gateway; we set the default-request-channel to the one from which the messages will be picked up, and passed to transformer, and next to pub-sub channel; that's just the example configuration;</li>
</ul>

##### Chaining endpoints


<ul>
    <li>we can do &lt;chain&gt; .... &lt;/chain&gt; and put the endpoints inside in proper order, without specifying the channels at all;</li>
    <li>the channels will be sync</li>
    <li>all the channels but the last must return output (returning null is fine)</li>
    <li>last channel must have an output-channel or replyChannel header defined</li>
    <li>a router can only be last</li>
</ul>

#### 3.2. Routing and filtering

##### Filter

<div><span
        style="font-family: &quot;courier new&quot; , &quot;courier&quot; , monospace;">&lt;filter id="cancellationFilter"</span>
</div>
<div><span
        style="font-family: &quot;courier new&quot; , &quot;courier&quot; , monospace;">&nbsp; &nbsp; &nbsp; &nbsp; input-channel="input"</span>
</div>
<div><span
        style="color: #999999; font-family: &quot;courier new&quot; , &quot;courier&quot; , monospace;">&nbsp; &nbsp; &nbsp; &nbsp; ref="cancellationsFilterBean"</span>
</div>
<div><span
        style="font-family: &quot;courier new&quot; , &quot;courier&quot; , monospace;">&nbsp; &nbsp; &nbsp; &nbsp; method="accept"</span>
</div>
<div><span
        style="color: #999999; font-family: &quot;courier new&quot; , &quot;courier&quot; , monospace;">&nbsp; &nbsp; &nbsp; &nbsp; discard-channel="rejected"</span>
</div>
<div><span
        style="font-family: &quot;courier new&quot; , &quot;courier&quot; , monospace;">&nbsp; &nbsp; &nbsp; &nbsp; output-channel="validated"</span>
</div>
<div><span
        style="color: #999999; font-family: &quot;courier new&quot; , &quot;courier&quot; , monospace;">&nbsp; &nbsp; &nbsp; &nbsp; throw-exception-on-rejection="true"</span>
</div>
<div><span style="font-family: &quot;courier new&quot; , &quot;courier&quot; , monospace;"><span
        style="color: #999999;">&nbsp; &nbsp; &nbsp; &nbsp; expression="payload?.reservationCode matches 'GOLD[A-Z0-9]{6}'"</span>/&gt;</span>
</div>
<div><span style="font-family: &quot;courier new&quot; , &quot;courier&quot; , monospace;"><br/></span></div>
<div><span
        style="font-family: &quot;courier new&quot; , &quot;courier&quot; , monospace;">cancellationsFilterBean-&gt;accept(m|payload)</span>
</div>
<div><span style="font-family: &quot;courier new&quot; , &quot;courier&quot; , monospace;"><br/></span></div>
<div>
    <ul>
        <li><span
               >if both "discard channel" and "throw exception" are set, the message will also be sent to the discard channel;</span>
        </li>
    </ul>
</div>

##### Router

<div><span
        style="font-family: &quot;courier new&quot; , &quot;courier&quot; , monospace;">&lt;router method="routePaymentSettlement"</span>
</div>
<div><span
        style="font-family: &quot;courier new&quot; , &quot;courier&quot; , monospace;">&nbsp; &nbsp; &nbsp; &nbsp; input-channel="payments"</span>
</div>
<div><span
        style="color: #999999; font-family: &quot;courier new&quot; , &quot;courier&quot; , monospace;">&nbsp; &nbsp; &nbsp; &nbsp; expression="payload.creditCardType"</span>
</div>
<div><span style="font-family: &quot;courier new&quot; , &quot;courier&quot; , monospace;"><span
        style="color: #999999;">&nbsp; &nbsp; &nbsp; &nbsp; channel-resolver="creditCardChannelResolver"</span>&gt;</span>
</div>
<div><span
        style="color: #999999; font-family: &quot;courier new&quot; , &quot;courier&quot; , monospace;">&nbsp; &nbsp; &lt;beans:bean class="....PaymentSettlementRouter"/</span><span
        style="color: #cccccc; font-family: &quot;courier new&quot; , &quot;courier&quot; , monospace;">&gt;</span>
</div>
<div><span style="font-family: &quot;courier new&quot; , &quot;courier&quot; , monospace;">&lt;/router&gt;</span>
</div>
<div><span style="font-family: &quot;courier new&quot; , &quot;courier&quot; , monospace;"><br/></span></div>
<div><span
        style="font-family: &quot;courier new&quot; , &quot;courier&quot; , monospace;">PaymentSettlementRouter</span>
</div>
<div><span
        style="font-family: &quot;courier new&quot; , &quot;courier&quot; , monospace;">&nbsp; -&gt;String|String[]routePaymentSettlement(m|payload|@Header...)</span>
</div>
<div>
    <ul>
        <li><span>if router's method returns null, the message won't be processed anymore</span>
        </li>
        <li><span>if it returns list of strings, the message will be sent to all these channels</span>
        </li>
        <li><span>all routers are content-based routers</span></li>
        <li>if the resolver is specified, it maps the result name to the real channel name</li>
    </ul>
</div>

##### Payload-type router

<div>
    <ul>
        <li><span>routing based on payload type:</span></li>
    </ul>
</div>
<div><span
        style="font-family: &quot;courier new&quot; , &quot;courier&quot; , monospace;">&lt;payload-type-router input-channel="payments"&gt;</span>
</div>
<div><span
        style="font-family: &quot;courier new&quot; , &quot;courier&quot; , monospace;">&nbsp; &lt;mapping type="....CreditCardPayment"</span>
</div>
<div><span
        style="font-family: &quot;courier new&quot; , &quot;courier&quot; , monospace;">&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp;channel="credit"/&gt;</span>
</div>
<div><span
        style="font-family: &quot;courier new&quot; , &quot;courier&quot; , monospace;">&nbsp; &lt;mapping type="....Invoice"</span>
</div>
<div><span
        style="font-family: &quot;courier new&quot; , &quot;courier&quot; , monospace;">&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp;channel="invoices"/&gt;</span>
</div>
<div><span style="font-family: &quot;courier new&quot; , &quot;courier&quot; , monospace;">&nbsp; ...</span></div>
<div><span
        style="font-family: &quot;courier new&quot; , &quot;courier&quot; , monospace;">&lt;/payload-type-router&gt;</span>
</div>

##### Header value router

<div>
    <ul>
        <li><span>routing based on a specific header's value:</span></li>
    </ul>
</div>
<div><span
        style="font-family: &quot;courier new&quot; , &quot;courier&quot; , monospace;">&lt;header-value-router input-channel="payments"</span>
</div>
<div><span
        style="font-family: &quot;courier new&quot; , &quot;courier&quot; , monospace;">&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp;header-name="PAYMENT_INFO"/&gt;</span>
</div>
<div><span style="font-family: &quot;courier new&quot; , &quot;courier&quot; , monospace;"><br/></span></div>
<div>
    <ul>
        <li><span>often used with a <b>header enricher</b>:</span></li>
    </ul>
    <div><span
            style="font-family: &quot;courier new&quot; , &quot;courier&quot; , monospace;">&lt;header-enricher input-channel="payments"</span>
    </div>
</div>
<div><span
        style="font-family: &quot;courier new&quot; , &quot;courier&quot; , monospace;">&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp;output-channel="enriched-payments"&gt;</span>
</div>
<div><span
        style="font-family: &quot;courier new&quot; , &quot;courier&quot; , monospace;">&nbsp; &nbsp;&lt;header name="</span><span
        style="font-family: &quot;courier new&quot; , &quot;courier&quot; , monospace;">PAYMENT_INFO</span><span
        style="font-family: &quot;courier new&quot; , &quot;courier&quot; , monospace;">"</span></div>
<div><span
        style="font-family: &quot;courier new&quot; , &quot;courier&quot; , monospace;">&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp;ref="enricher"</span>
</div>
<div><span
        style="font-family: &quot;courier new&quot; , &quot;courier&quot; , monospace;">&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp;method="determineProcessingDestination"/&gt;</span>
</div>
<div><span
        style="font-family: &quot;courier new&quot; , &quot;courier&quot; , monospace;">&lt;/header-enricher&gt;</span>
</div>

##### Recipient list router

<div>
    <ul>
        <li><span>sends message to multiple channels:</span></li>
    </ul>
</div>
<div><span
        style="font-family: &quot;courier new&quot; , &quot;courier&quot; , monospace;">&lt;recipient-list-router input-channel="notifications"&gt;</span>
</div>
<div><span
        style="font-family: &quot;courier new&quot; , &quot;courier&quot; , monospace;">&nbsp; &lt;recipient channel="sms"/&gt;</span>
</div>
<div><span
        style="font-family: &quot;courier new&quot; , &quot;courier&quot; , monospace;">&nbsp; &lt;recipient channel="email"/&gt;</span>
</div>
<div><span
        style="font-family: &quot;courier new&quot; , &quot;courier&quot; , monospace;">&nbsp; &lt;recipient channel="phone"/&gt;</span>
</div>
<div><span style="font-family: &quot;courier new&quot; , &quot;courier&quot; , monospace;">&lt;/</span><span
        style="font-family: &quot;courier new&quot; , &quot;courier&quot; , monospace;">recipient-list-router</span><span
        style="font-family: &quot;courier new&quot; , &quot;courier&quot; , monospace;">&gt;</span></div>
<div><span style="font-family: &quot;courier new&quot; , &quot;courier&quot; , monospace;"><br/></span></div>

#### 3.3 Splitting and aggregating

<div>Aggregator and resequencer are
    <b>stateful </b>endpoints. And both aggregator and splitter modify the message (numbers of it).
</div>
<div><br/></div>
<div><span
        style="font-family: &quot;courier new&quot; , &quot;courier&quot; , monospace;">&lt;chain id="splitRecipesIntoIngrdients"</span>
</div>
<div><span
        style="font-family: &quot;courier new&quot; , &quot;courier&quot; , monospace;">&nbsp; &nbsp; &nbsp; &nbsp;input-channel="recipes"</span>
</div>
<div><span
        style="font-family: &quot;courier new&quot; , &quot;courier&quot; , monospace;">&nbsp; &nbsp; &nbsp; &nbsp;output-channel="ingredients"&gt;</span>
</div>
<div><span
        style="font-family: &quot;courier new&quot; , &quot;courier&quot; , monospace;">&nbsp; &lt;header-enricher&gt;</span>
</div>
<div><span
        style="font-family: &quot;courier new&quot; , &quot;courier&quot; , monospace;">&nbsp; &nbsp; &nbsp;&lt;header name="recipe" expression="payload"/&gt;</span>
</div>
<div><span
        style="font-family: &quot;courier new&quot; , &quot;courier&quot; , monospace;">&nbsp; &lt;/header-enricher&gt;</span>
</div>
<div><span style="font-family: &quot;courier new&quot; , &quot;courier&quot; , monospace;">&nbsp; &lt;splitter <span
        style="color: #999999;">expression="payload.ingredients"</span>&gt;</span></div>
<div><span
        style="color: #999999; font-family: &quot;courier new&quot; , &quot;courier&quot; , monospace;">&nbsp; &nbsp; &lt;beans:bean class="....MySplitter"/&gt;</span>
</div>
<div><span
        style="font-family: &quot;courier new&quot; , &quot;courier&quot; , monospace;">&nbsp; &lt;/splitter&gt;</span>
</div>
<div><span style="font-family: &quot;courier new&quot; , &quot;courier&quot; , monospace;"><br/></span></div>
<div><span
       >Splitter will automatically set <i>group size, sequence number</i> and <i>correlation id </i>on split messages, but you don't have to use them in your aggregation logic. In our example above we passed the whole message into the header as we will use it as the correlation key.</span>
</div>
<div><span><br/></span></div>
<div><span
        style="font-family: &quot;courier new&quot; , &quot;courier&quot; , monospace; font-size: x-small;">&lt;aggregator id="kitchen"</span>
</div>
<div><span
        style="font-family: &quot;courier new&quot; , &quot;courier&quot; , monospace; font-size: x-small;">&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; input-channel="products"</span>
</div>
<div><span
        style="font-family: &quot;courier new&quot; , &quot;courier&quot; , monospace; font-size: x-small;">&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; output-channel="meals"</span>
</div>
<div><span
        style="font-family: &quot;courier new&quot; , &quot;courier&quot; , monospace; font-size: x-small;">&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; ref="cook"</span>
</div>
<div><span
        style="font-family: &quot;courier new&quot; , &quot;courier&quot; , monospace; font-size: x-small;">&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; method="prepareMeal"</span>
</div>
<div><span
        style="font-family: &quot;courier new&quot; , &quot;courier&quot; , monospace; font-size: x-small;">&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; correlation-strategy="cook"</span>
</div>
<div><span
        style="font-family: &quot;courier new&quot; , &quot;courier&quot; , monospace; font-size: x-small;">&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; correlation-strategy-method="getCorrelationKey"</span>
</div>
<div><span
        style="font-family: &quot;courier new&quot; , &quot;courier&quot; , monospace; font-size: x-small;">&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; release-strategy="cook"</span>
</div>
<div><span
        style="font-family: &quot;courier new&quot; , &quot;courier&quot; , monospace; font-size: x-small;">&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; release-strategy-method="canCookMeal"/&gt;</span>
</div>
<div><span style="font-family: &quot;courier new&quot; , &quot;courier&quot; , monospace; font-size: x-small;"><br/></span>
</div>
<div><span
       >Evey arriving message from same group the aggregator will internally append to a MessageGroupStore.</span>
</div>
<div>
    <ul>
        <li><span
                style="font-family: &quot;courier new&quot; , &quot;courier&quot; , monospace;">Meal prepareMeal(List&lt;m&gt;)</span><span
               >&nbsp;- returns an aggregated message based on contents of the message group</span>
        </li>
        <li><span
                style="font-family: &quot;courier new&quot; , &quot;courier&quot; , monospace;">Object getCorrelationKey(m)</span><span
               >&nbsp;- returns an identifier of this group</span></li>
        <li><span
                style="font-family: &quot;courier new&quot; , &quot;courier&quot; , monospace;">bool canCookMeal(List&lt;m&gt;)</span><span
               >&nbsp;- returns true if the group is complete</span></li>
    </ul>
    <div>The aggregator internally also uses a CorrelatingMessageHandler. It holds references to Correlation- and ReleaseStrategy, as well as to MessageGroupProcessor, which is processing the released group.</div>
    <div><br/></div>
    <div>The same&nbsp;CorrelatingMessageHandler&nbsp;is also used by the resequencer endpoint. Resequencer buffers incoming messages, waits and tries to assure that the messages are in right order, but since this can be tricky, it can do partial releases, which can be based e.g. on a timeout (releasePartialSequences flag). You can also customize the comparator used to determine the order.</div>
</div>

##### Scatter-gather algorithm

<div>Is about copying same message to many processors, each is specialised in sth else, and then aggregating the results.</div>
<div><br/></div>

### 4. Integrating Existing Systems

#### 4.1. XML

<div>Sometimes it is worth avoiding conversion from and to XML, if both input and output are in XML, and the processing is relatively simple. It's better to use XPath or XSLT directly in such case. Or even, like in the example in the book, if we have to devide an object into a number of small parts and send it as separated XMLs, it may be a better idea to first convert it into XML and split/transform only the XML file.</div>

##### Marshalling and unmarshalling
<div>It's conversion between XML and Object. In Spring it's called OXM (it's just an aggregator of existing solutions).</div>
<div><br/></div>
<div>To marshall, annotate class with:</div>
<div>
    <ul>
        <li><span
                style="font-family: &quot;courier new&quot; , &quot;courier&quot; , monospace;">@javax.xml.bind.annotation.XmlRootElement(name = "rootTagName")</span>
        </li>
    </ul>
</div>
<div>and annotate fields with:</div>
<div>
    <ul>
        <li><span style="font-family: &quot;courier new&quot; , &quot;courier&quot; , monospace;">@XmlElement</span>
        </li>
    </ul>
</div>
<div>If you have a non-standard java types inside your class, you have to add on the field level:</div>
<div>
    <ul>
        <li><span
                style="font-family: &quot;courier new&quot; , &quot;courier&quot; , monospace;">@XmlJavaTypeAdapter(YourAdapter.class)</span>
        </li>
    </ul>
    <div><span>But Spring XOM has a ready marshaller for this, which you configure like this:</span>
    </div>
    <div><span style="font-family: &quot;courier new&quot; , &quot;courier&quot; , monospace;"><br/></span></div>
    <div><span
            style="font-family: &quot;courier new&quot; , &quot;courier&quot; , monospace; font-size: x-small;">&lt;bean id="myMarshaller" class="....Jaxb2Marshaller"&gt;</span>
    </div>
    <div><span
            style="font-family: &quot;courier new&quot; , &quot;courier&quot; , monospace; font-size: x-small;">&nbsp; &lt;p:name="classesToBeBound" value="....ClassToMarshall"/&gt;</span>
    </div>
    <div><span
            style="font-family: &quot;courier new&quot; , &quot;courier&quot; , monospace; font-size: x-small;">&lt;/bean&gt;</span>
    </div>
    <div>
       <b>Spring Integration Support for XMLAfter you configured Spring XOM marshaller and unmarshaller, you can add an endpoint:</b><br/><br/><span
            style="font-family: &quot;courier new&quot; , &quot;courier&quot; , monospace; font-size: x-small;">&lt;si-xml:marshalling-transformer&nbsp;</span><br/><span
            style="font-family: &quot;courier new&quot; , &quot;courier&quot; , monospace; font-size: x-small;">&nbsp; &nbsp; &nbsp; &nbsp;input-channel="input"</span><br/><span
            style="font-family: &quot;courier new&quot; , &quot;courier&quot; , monospace; font-size: x-small;">&nbsp; &nbsp; &nbsp; &nbsp;output-channel="xmlOut"</span><br/><span
            style="font-family: &quot;courier new&quot; , &quot;courier&quot; , monospace; font-size: x-small;">&nbsp; &nbsp; &nbsp; &nbsp;marshaller="myMarshaller"</span><br/><span
            style="font-family: &quot;courier new&quot; , &quot;courier&quot; , monospace; font-size: x-small;"><span
            style="color: #666666;">&nbsp; &nbsp; &nbsp; &nbsp;result-transformer="resultToDocumentTransformer"</span>/&gt;</span><br/><span
            style="font-family: &quot;courier new&quot; , &quot;courier&quot; , monospace; font-size: x-small;"><br/></span><span>You need the result transformer, as the marshaller returns an object of type Result, which you can further convert to String or Document, or whatever. There are two ready result transformers in Spring Integration, ResultToDocumentTransformer and&nbsp;</span>ResultToStringTransformer.<br/><br/>In case you wanted to so an XSLT transformation, use:<br/><br/><span
            style="font-family: &quot;courier new&quot; , &quot;courier&quot; , monospace;">&lt;si-xml:xslt-transformer</span><br/><span
            style="font-family: &quot;courier new&quot; , &quot;courier&quot; , monospace;">&nbsp; &nbsp; input-channel="inputXml"</span><br/><span
            style="font-family: &quot;courier new&quot; , &quot;courier&quot; , monospace;">&nbsp; &nbsp; output-channel="transformedXml"</span><br/><span
            style="font-family: &quot;courier new&quot; , &quot;courier&quot; , monospace;">&nbsp; &nbsp; xsl-resource="classpath:/xsl/blablah.xsl"/&gt;</span><br/><span
            style="font-family: &quot;courier new&quot; , &quot;courier&quot; , monospace;"><br/></span><span>You also may wanna split XML using XPath, you can do it usingXPath splitter:</span><br/><span><br/></span><span
            style="font-family: &quot;courier new&quot; , &quot;courier&quot; , monospace;">&lt;si-xml:xpath-splitter create-documents="true"</span><br/><span
            style="font-family: &quot;courier new&quot; , &quot;courier&quot; , monospace;">&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp;input-channel, output-channel/&gt;</span><br/><span
            style="font-family: &quot;courier new&quot; , &quot;courier&quot; , monospace;">&nbsp; &nbsp;&lt;si-xml:xpath-expression&nbsp;</span><br/><span
            style="font-family: &quot;courier new&quot; , &quot;courier&quot; , monospace;">&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp;expression="parentNodeName"</span><br/><span
            style="color: #666666; font-family: &quot;courier new&quot; , &quot;courier&quot; , monospace;">&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp;ns-prefix="hb"</span><br/><span
            style="color: #666666; font-family: &quot;courier new&quot; , &quot;courier&quot; , monospace;">&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp;ns-uri="http://www.example.com/blablah"</span><br/><span
            style="color: #666666; font-family: &quot;courier new&quot; , &quot;courier&quot; , monospace;">&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp;namespace-map="namespaceMap"</span><br/><span
            style="font-family: &quot;courier new&quot; , &quot;courier&quot; , monospace;">&lt;/si-xml:xpath&gt;</span><br/><span
            style="font-family: &quot;courier new&quot; , &quot;courier&quot; , monospace;"><br/></span><span
            style="color: #666666; font-family: &quot;courier new&quot; , &quot;courier&quot; , monospace; font-size: x-small;">&lt;util:map id="namespaceMap"&gt;</span><br/><span
            style="color: #666666; font-size: x-small;"><span
            style="font-family: &quot;courier new&quot; , &quot;courier&quot; , monospace;">&nbsp; &nbsp;&lt;entry key="hb" value="</span><span
            style="font-family: &quot;courier new&quot; , &quot;courier&quot; , monospace;">http://www.example.com/blablah</span><span
            style="font-family: &quot;courier new&quot; , &quot;courier&quot; , monospace;">"</span></span><br/><span
            style="font-family: &quot;courier new&quot; , &quot;courier&quot; , monospace;"><span
            style="color: #666666; font-size: x-small;">&nbsp; &nbsp;...</span></span><br/><span
            style="font-family: &quot;courier new&quot; , &quot;courier&quot; , monospace;"><span
            style="color: #666666; font-size: x-small;">&lt;/util:map&gt;</span></span><br/><br/>
        <ul>
            <li><span>if create-documents is set to true, each part will be wrapped in a separate XML document, otherwise it will be just raw content</span>
            </li>
        </ul>
        <div>You can also route messages based on XPath expression:</div>
        <div><br/></div>
        <div><span
                style="font-family: &quot;courier new&quot; , &quot;courier&quot; , monospace; font-size: x-small;">&lt;si-xml:xpath-router id="myRouter"</span>
        </div>
        <div><span
                style="font-family: &quot;courier new&quot; , &quot;courier&quot; , monospace; font-size: x-small;">&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp;input-channel="splitXml"</span>
        </div>
        <div><span
                style="font-family: &quot;courier new&quot; , &quot;courier&quot; , monospace; font-size: x-small;">&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp;evaluate-as-string="true"&gt;</span>
        </div>
        <div><span
                style="font-family: &quot;courier new&quot; , &quot;courier&quot; , monospace; font-size: x-small;">&nbsp; &nbsp;&lt;si-xml:xpath-expression expression="local-name(/*)"/&gt;</span>
        </div>
        <div><span style="font-size: x-small;"><span
                style="font-family: &quot;courier new&quot; , &quot;courier&quot; , monospace;">&nbsp; &nbsp;&lt;si-xml:mapping value="carQuote"&nbsp;</span><span
                style="font-family: &quot;courier new&quot; , &quot;courier&quot; , monospace;">channel="carQuoChannel"/&gt;</span></span>
        </div>
        <div><span style="font-family: &quot;courier new&quot; , &quot;courier&quot; , monospace;"><span
                style="font-size: x-small;">&nbsp; &nbsp;&lt;si-xml:mapping value="...</span></span></div>
        <div><span style="font-family: &quot;courier new&quot; , &quot;courier&quot; , monospace;"><span
                style="font-size: x-small;">&lt;/si-xml:xpath-router&gt;</span></span></div>
        <div><span style="font-family: &quot;courier new&quot; , &quot;courier&quot; , monospace;"><span
                style="font-size: x-small;"><br/></span></span></div>
        <div><span>Or validate them:</span></div>
        <div><span><br/></span></div>
        <div><span
                style="font-family: &quot;courier new&quot; , &quot;courier&quot; , monospace; font-size: x-small;">&lt;si-xml:validating-filter id, input-channel, output-channel</span>
        </div>
        <div><span
                style="font-family: &quot;courier new&quot; , &quot;courier&quot; , monospace; font-size: x-small;">&nbsp; &nbsp;discard-channel="invalidReqs"</span>
        </div>
        <div><span
                style="font-family: &quot;courier new&quot; , &quot;courier&quot; , monospace; font-size: x-small;">&nbsp; &nbsp;schema-location="classpath:xsd/fligthQuote.xsd"/&gt;</span>
        </div>
        <div><span
                style="font-family: &quot;courier new&quot; , &quot;courier&quot; , monospace; font-size: x-small;"><br/></span>
        </div>
        <div><span>Spring Integration supports DOM, but not SAX parser, as streaming would introduce problems when combined with messaging. So big XML support is not there.</span>
        </div>
        <div><span><br/></span></div></div></div>

#### 4.2. JMS

<ul>
    <li>general abstraction over MOM (Message Oriented Middleware)</li>
    <li>Spring Integration is doing well already without JMS, but it's worth to be able to connect to existing JMS system; or to have some persistent storage in an independent queue, or between different JVMs, or to actually have transactions, or implicit load balancing</li>
    <li>ActiveMQ is an opensource implementation of JMS</li>
</ul>

##### JMS terminology

<div>JMS message has a lot in common with Spring Integration message. JMS body = Spring payload and JMS properties =&nbsp;Spring&nbsp;headers. Additionally, JMS destination =&nbsp;Spring&nbsp;channel, and point-to-point channel is called Queue, and pub-sub is called Topic.</div>

#### JMS refresh

<ol>
    <li>JmsTemplate<br/><br/>jmsTemplate = new JmsTemplate(connectionFactory);<br/>jmsTemplate.setDefaultDestination(new ActiveMQQueue("siisa.queue"));<br/>jmsTemplate.convertAndSend("Helloo");<br/>String res = jmsTemplate.receiveAndConvert();<br/><br/>If transaction is active and was already opened by some process, this executes in same transaction. Transaction is bound to JMS session.<br/><br/>The conversion is done by default by SimpleMessageConverter, which maps the Java type to the MessageType (TextMessage, MapMessage, BytesMessage, ObjectMessage, etc), but it can be customized. It can even be replaced by Spring XOM MarshallingMessageConverter, to have to conversion from and to XML.<br/><br/>JmsTemplate is sync.
    </li>
    <li>MessageListener<br/><br/>This is async. And provides transaction handling which would be tricky with async.<br/><br/>You'd actually implement&nbsp;MessageListenerAdapter, as it's eliminating some boilerplate code. And even better, define it in XML config:<br/><br/><span
            style="font-family: &quot;courier new&quot; , &quot;courier&quot; , monospace;">&lt;jms:listener-container&gt;<br/>&nbsp; &nbsp;&lt;jms:listener <br/>&nbsp; &nbsp; &nbsp; &nbsp; &nbsp;destination="myQueue" <br/>&nbsp; &nbsp; &nbsp; &nbsp; &nbsp;ref="aPojo" <br/>&nbsp; &nbsp; &nbsp; &nbsp; &nbsp;method="someMethod"/<br/>&lt;/jms:listener-container&gt;</span>
    </li>
</ol>

##### Spring Integration &amp; JMS - one way communication

<div>For sending to JMS use an outbound channel adapter:</div>

<div>&lt;int-jms:outbound-channel-adapter channel="toJms"</div>
<div>&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; destination-name=""samples.queue"<br/><span
        style="color: #666666;">&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; destination=fromSi"</span></div>
<div><span style="color: #666666;">&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; pub-sub-domain="true"</span>/&gt;<br/><br/><span
        style="color: #666666;">&lt;jee:jndi-lookup id="fromSi" jndi-name="jms/queue.fromSi"/&gt;</span>
</div>
<ul>
    <li><span>the responsibilities are handled inernally by JmsTemplate</span>
    </li>
    <li><span>set pub-sub-domain to true if the destination is a topic</span>
    </li>
    <li><span>specify either destination or destination-name</span>
    </li>
</ul>
<div>For receiving use an inbound channel adapter (sync polling):</div>

<div><span
        style="font-family: &quot;courier new&quot; , &quot;courier&quot; , monospace; font-size: x-small;">&lt;int-jms:inbound-channel-adapter&nbsp;</span>
</div>
<div><span
        style="font-family: &quot;courier new&quot; , &quot;courier&quot; , monospace; font-size: x-small;">&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; id="pollingJmsAdapter"</span>
</div>
<div><span
        style="font-family: &quot;courier new&quot; , &quot;courier&quot; , monospace; font-size: x-small;">&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; channel="jmsMessage"</span><br/><span
        style="font-family: &quot;courier new&quot; , &quot;courier&quot; , monospace; font-size: x-small;">&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; destination-name="myQueue"</span><br/><span
        style="font-family: &quot;courier new&quot; , &quot;courier&quot; , monospace; font-size: x-small;"><span
        style="color: #666666;">&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; pub-sub-domain="true"</span>&gt;</span><br/><span
        style="font-family: &quot;courier new&quot; , &quot;courier&quot; , monospace; font-size: x-small;">&nbsp; &nbsp;&lt;int-poller fixed-delay="3000" max-messages-per-poll="1"/&gt;</span><br/><span
        style="font-family: &quot;courier new&quot; , &quot;courier&quot; , monospace; font-size: x-small;">&lt;/int-jms:inbound-channel-adapter&gt;</span><br/>
    <ul>
        <li><span>connection-factory should be set if its name is different than "connectionFactory"</span>
        </li>
    </ul>
</div>
<div>To do it asynchronously, use:<br/><br/><span
        style="font-family: &quot;courier new&quot; , &quot;courier&quot; , monospace;">&lt;int-jms:message-driven-channel-adapter</span><br/><span
        style="font-family: &quot;courier new&quot; , &quot;courier&quot; , monospace;">&nbsp; &nbsp; id, channel, destination-name/&gt;</span><br/></div>


##### Spring Integration &amp; JMS - two ways communication

<span>If you need bi-directional communication, use a gateway:</span><br/><span><br/></span><span
        style="font-family: &quot;courier new&quot; , &quot;courier&quot; , monospace;">&lt;int-jms:outbound-gateway&nbsp;</span>
<br/>
<div><span
        style="font-family: &quot;courier new&quot; , &quot;courier&quot; , monospace;">&nbsp; &nbsp; &nbsp; &nbsp; request-channel="toJms"</span><br/><span
        style="font-family: &quot;courier new&quot; , &quot;courier&quot; , monospace;">&nbsp; &nbsp; &nbsp; &nbsp; reply-channel="jmsReplies"</span><br/><span
        style="font-family: &quot;courier new&quot; , &quot;courier&quot; , monospace;">&nbsp; &nbsp; &nbsp; &nbsp; request-destniation-name="examples.queue"</span><br/><span
        style="font-family: &quot;courier new&quot; , &quot;courier&quot; , monospace;"><span
        style="color: #666666;">&nbsp; &nbsp; &nbsp; &nbsp; request-pub-sub-domain="true"</span>/&gt;</span><br/><br/>
    <ul>
        <li><span>use request-pub-sub-domain if you use JMS topic</span>
        </li>
        <li><span>notice that reply-destination-name is not required; the gateway will add a </span><span
                style="font-family: &quot;courier new&quot; , &quot;courier&quot; , monospace;">JMSReplyTo</span><span> property to each message as property, and set it to a temporary queue which it creates for you</span>
        </li>
    </ul>
</div><span
        style="font-family: &quot;courier new&quot; , &quot;courier&quot; , monospace;">&lt;int-jms:inbound-gateway id</span>
<br/><span
        style="font-family: &quot;courier new&quot; , &quot;courier&quot; , monospace;">&nbsp; &nbsp; &nbsp; &nbsp; request-channel="fromJms"</span>
<br/><span
        style="font-family: &quot;courier new&quot; , &quot;courier&quot; , monospace;">&nbsp; &nbsp; &nbsp; &nbsp; request-destniation-name="examples.queue"</span>
<br/><span
        style="color: #999999; font-family: &quot;courier new&quot; , &quot;courier&quot; , monospace;">&nbsp; &nbsp; &nbsp; &nbsp; default-reply-destination="examples.replies"</span>
<br/><span style="font-family: &quot;courier new&quot; , &quot;courier&quot; , monospace;"><span
        style="color: #999999;">&nbsp; &nbsp; &nbsp; &nbsp; default-reply-queue-name="examples.replies"</span></span>
<br/><span style="font-family: &quot;courier new&quot; , &quot;courier&quot; , monospace;"><span
        style="color: #999999;">&nbsp; &nbsp; &nbsp; &nbsp; default-reply-topic-name="examples.replies"</span></span>
<br/><span style="font-family: &quot;courier new&quot; , &quot;courier&quot; , monospace;"><span
        style="color: #999999;">&nbsp; &nbsp; &nbsp; &nbsp; request-pub-sub-domain="true"</span></span><br/><span
        style="color: #f6b26b; font-family: &quot;courier new&quot; , &quot;courier&quot; , monospace;">&nbsp; &nbsp; &nbsp; &nbsp; concurrent-consumers="5"</span>
<br/><span
        style="color: #f6b26b; font-family: &quot;courier new&quot; , &quot;courier&quot; , monospace;">&nbsp; &nbsp; &nbsp; &nbsp; max-concurrent-consumers="25"</span>
<br/><span style="font-family: &quot;courier new&quot; , &quot;courier&quot; , monospace;"><span
        style="color: #f6b26b;">&nbsp; &nbsp; &nbsp; &nbsp; idle-task-execution-limit="3"</span>/&gt;</span><br/><br/>
<ul>
    <li><span>if JMS has set the JMSReplyTo, it takes precedence over the default-reply-destination</span>
    </li>
    <li><span>reply-channel is not required; if it is not set, the gateway will create implicit channel and set it as the "replyChannel" header on the message</span>
    </li>
    <li><span>connection-factory should be set if its name is different than "connectionFactory"</span>
    </li>
    <li><span>the last three attrs are for concurrency settings</span>
    </li>
</ul>

##### Spring Integration &amp; JMS - "tunnelling"

<div>If two Spring Integration apps want to communicate via JMS, we can reduce some complexity by passing whole Spring message as the JMS message body, instead of converting payload to body and headers to properties and back. In such case add to your both gateways:
    <span style="font-family: &quot;courier new&quot; , &quot;courier&quot; , monospace;">extract-request-payload="false"</span><span>. It's also a good idea to set the </span><span
            style="font-family: &quot;courier new&quot; , &quot;courier&quot; , monospace;">message-converter </span><span>property, so that you don't use Java Serialisation but e.g. XML. You can use e.g. the Spring's MarshallingMessageConverter.</span>
</div>
<div><span><br/></span></div>
<div><span>But in general, this approach is not recommended - :^)</span></div>

##### Transactions with JMS Integration

<div>No matter how you do it, you will either have lost messages or duplicates.<br/><br/>The JMS Session object can be created by JMS connection with two exclusive flags: transacted(bool) and acknowledgeMode(AUTO_ACKNOWLEDGE, CLIENT_ACKNOWLEDGE, DUPS_OK_ACKNOWLEDGE). DUPS_OK will also call aknowledge() automatically but lazily. I skip the details as it was in Spring Core already.<br/><br/>MessageListener has a property "acknowledge", which can take one of 4 values: auto (default), client, dups-ok and transacted. In XML you set it on &lt;jms:listener-container ..&gt;.<br/><br/>Transactions are in general managed by PlatformTransactionManager (JmsTransactionManager or DataSourceTransactionManager). &nbsp;If you want to include a database operation into same transaction, you do not neccessarily have to switch to the complex XA transactions. Often it is possible to just order your calls properly, e.g. in same method receive JMS message and call the database at the end. The tradeoff is that very rarely you may still end up with duplicates, but it's sometimes easier to compensate that than introduce global transactions.<br/><br/></div>

#### 4.3. Email

##### Sending Email

<div><span
        style="font-family: &quot;courier new&quot; , &quot;courier&quot; , monospace;">&lt;mail:outbound-channel-adapter&nbsp;</span>
</div>
<div><span
        style="font-family: &quot;courier new&quot; , &quot;courier&quot; , monospace;">&nbsp; &nbsp; &nbsp; channel="outboundMail"</span>
</div>
<div><span
        style="font-family: &quot;courier new&quot; , &quot;courier&quot; , monospace;">&nbsp; &nbsp; &nbsp; host="${host}"</span>
</div>
<div><span
        style="font-family: &quot;courier new&quot; , &quot;courier&quot; , monospace;">&nbsp; &nbsp; &nbsp; username="${username}"</span>
</div>
<div><span
        style="font-family: &quot;courier new&quot; , &quot;courier&quot; , monospace;">&nbsp; &nbsp; &nbsp; password="${password}"</span>
</div>
<div><span
        style="font-family: &quot;courier new&quot; , &quot;courier&quot; , monospace;">&nbsp; &nbsp; &nbsp; java-mail-properties="properties"/&gt;</span>
</div>
<div>
    <ul>
        <li><span
               >supported payload types: String, byte array (for attachments), MailMessage, MimeMessage</span>
        </li>
        <li><span
               >the from and to, and even email subject, are in message headers; we should fill them in using </span><span
                style="font-family: &quot;courier new&quot; , &quot;courier&quot; , monospace;">mail:header-enricher</span>
        </li>
        <li><span
               >internally it uses JavaMail API, more specifically the Spring JavaMailSender which builds on top of it</span>
        </li>
        <li><span
               >JavaMailSender introduced also MailMessage, next to the low level MimeMessage</span>
        </li>
        <li><span
               >the java-mail-properties are for setting extra properties specific to JavaMail, like </span><span
                style="font-family: &quot;courier new&quot; , &quot;courier&quot; , monospace;">mail.store.protocol=imap</span><span
               > (it will be important)</span></li>
    </ul>
</div>

##### Receiving Email - polling

<div><span style="font-size: x-small;">&nbsp;<span
        style="font-family: &quot;courier new&quot; , &quot;courier&quot; , monospace;">&lt;mail:inbound-channel-adapter</span></span>
</div>
<div><span
        style="font-family: &quot;courier new&quot; , &quot;courier&quot; , monospace; font-size: x-small;">&nbsp; &nbsp; &nbsp; id="mailAdapter"</span>
</div>
<div><span
        style="font-family: &quot;courier new&quot; , &quot;courier&quot; , monospace; font-size: x-small;">&nbsp; &nbsp; &nbsp; store-uri="imaps://..."</span>
</div>
<div><span
        style="font-family: &quot;courier new&quot; , &quot;courier&quot; , monospace; font-size: x-small;">&nbsp; &nbsp; &nbsp; java-mail-properties="properties"</span>
</div>
<div><span
        style="font-family: &quot;courier new&quot; , &quot;courier&quot; , monospace; font-size: x-small;">&nbsp; &nbsp; &nbsp; channel="emails"</span>
</div>
<div><span
        style="font-family: &quot;courier new&quot; , &quot;courier&quot; , monospace; font-size: x-small;">&nbsp; &nbsp; &nbsp; should-delete-messages="true"</span>
</div>
<div><span
        style="font-family: &quot;courier new&quot; , &quot;courier&quot; , monospace; font-size: x-small;">&nbsp; &nbsp; &nbsp; should-mark-messages-as-read="true"&gt;</span>
</div>
<div><span
        style="font-family: &quot;courier new&quot; , &quot;courier&quot; , monospace; font-size: x-small;">&nbsp; &nbsp;&lt;poller max-messages-per-poll="1" fixed-rate="5000"/&gt;</span>
</div>
<div><span
        style="font-family: &quot;courier new&quot; , &quot;courier&quot; , monospace; font-size: x-small;">&lt;/mail:inbound-channel-adapter&gt;</span>
</div>
<div><span style="font-family: &quot;courier new&quot; , &quot;courier&quot; , monospace; font-size: x-small;"><br/></span>
</div>

<ul>
    <li><span
           >POP3 is designed for downloading; therefore it will work only with polling; there is no session, so messages can be downloaded multiple times, that's why we have the option to delete them on read; should-mark-messages-as-read will be ignored here</span>
    </li>
    <li><span
           >IMAP is designed for keeping emails on server; the mailbox maintains state, so duplicates are not a risk; you can use it also with event driven reception.</span>
    </li>
    <li><span>SMTP is also supported</span></li>
</ul>

##### Receiving Email - event driven

<div><span
       >It's not that beautiful as this works max for 30 minutes of client being idle because of a timeout.</span>
</div>
<div><span><br/></span></div>
<div>
    <div><span style="font-size: x-small;">&nbsp;<span
            style="font-family: &quot;courier new&quot; , &quot;courier&quot; , monospace;">&lt;mail:imap-idle-channel-adapter</span></span>
    </div>
    <div><span
            style="font-family: &quot;courier new&quot; , &quot;courier&quot; , monospace; font-size: x-small;">&nbsp; &nbsp; &nbsp; id="mailAdapter"</span>
    </div>
    <div><span
            style="font-family: &quot;courier new&quot; , &quot;courier&quot; , monospace; font-size: x-small;">&nbsp; &nbsp; &nbsp; store-uri="imaps://..."</span>
    </div>
    <div><span
            style="font-family: &quot;courier new&quot; , &quot;courier&quot; , monospace; font-size: x-small;">&nbsp; &nbsp; &nbsp; java-mail-properties="properties"</span>
    </div>
    <div><span
            style="font-family: &quot;courier new&quot; , &quot;courier&quot; , monospace; font-size: x-small;">&nbsp; &nbsp; &nbsp; channel="emails"</span>
    </div>
    <div><span
            style="font-family: &quot;courier new&quot; , &quot;courier&quot; , monospace; font-size: x-small;">&nbsp; &nbsp; &nbsp; should-delete-messages="false"</span>
    </div>
    <div><span
            style="font-family: &quot;courier new&quot; , &quot;courier&quot; , monospace; font-size: x-small;">&nbsp; &nbsp; &nbsp; should-mark-messages-as-read="true"</span>
    </div>
    <div><span
            style="font-family: &quot;courier new&quot; , &quot;courier&quot; , monospace; font-size: x-small;">&nbsp; &nbsp; &nbsp; auto-startup="true"&gt;</span>
    </div>
    <div><span
            style="font-family: &quot;courier new&quot; , &quot;courier&quot; , monospace; font-size: x-small;">&nbsp; &nbsp;&lt;poller max-messages-per-poll="1" fixed-rate="5000"/&gt;</span>
    </div>
    <div><span
            style="font-family: &quot;courier new&quot; , &quot;courier&quot; , monospace; font-size: x-small;">&lt;/</span><span
            style="font-family: &quot;courier new&quot; , &quot;courier&quot; , monospace; font-size: x-small;">mail:imap-idle-channel</span><span
            style="font-family: &quot;courier new&quot; , &quot;courier&quot; , monospace; font-size: x-small;">-adapter&gt;</span>
    </div>
</div>
<div>
    <ul>
        <li><span>the differences are the name and auto-startup property</span></li>
    </ul>
    <div>You may find this handy:</div>
</div>
<div><br/></div>
<div><span
        style="font-family: &quot;courier new&quot; , &quot;courier&quot; , monospace;">&lt;mail:mail-to-string-transformer/&gt;</span>
</div>
<div><span style="font-family: &quot;courier new&quot; , &quot;courier&quot; , monospace;"><br/></span></div>
<div><span
       >it will also copy the headers. You can also implement a custom AbstractMailMessageTransformer, but the headers will be copied as well (the abstract method returns MessageBuilder).</span>
</div>
<div><span style="font-family: &quot;courier new&quot; , &quot;courier&quot; , monospace; font-size: x-small;"><br/></span>
</div>

#### 4.4. Filesystem Integration

<div><span
       >The advantages are that they are fairly simple, disk space is big enough, and data is persisted. Disadvantages: slow, no ACID, and pain in the ass with locking. Still it's simple so should be used when possible (simple!=easy:P).</span>
</div>
<div>
    <ul>
        <li>there's <span
                style="font-family: &quot;courier new&quot; , &quot;courier&quot; , monospace;">file: </span>namespace in Spring context, use it
        </li>
        <li>remember that Java's File is not a psychical file; and that onlt the path of the file is immutable, the rest is read from filesystem, on request</li>
    </ul>
    <div>Spring Integration provides abstraction over all the stupid new BufferedReader(new FileReader(new ...)). This is already cool.</div>
</div>

##### Writing the file

<div><span
        style="font-family: &quot;courier new&quot; , &quot;courier&quot; , monospace;">&lt;file:outbound-channel-adapter</span>
</div>
<div><span
        style="font-family: &quot;courier new&quot; , &quot;courier&quot; , monospace;">&nbsp; &nbsp; channel="outgoingChanges"</span>
</div>
<div><span
        style="font-family: &quot;courier new&quot; , &quot;courier&quot; , monospace;">&nbsp; &nbsp; directory="#{config.diary.store}"</span>
</div>
<div><span
        style="font-family: &quot;courier new&quot; , &quot;courier&quot; , monospace;">&nbsp; &nbsp; auto-create-directory="true"</span>
</div>
<div><span
        style="font-family: &quot;courier new&quot; , &quot;courier&quot; , monospace;">&nbsp; &nbsp; filename-generator="nameGenerator"</span>
</div>
<div><span
        style="font-family: &quot;courier new&quot; , &quot;courier&quot; , monospace;">&nbsp; &nbsp; detele-source-file="false"/&gt;</span>
</div>
<div>
    <ul>
        <li><span>message payload can be of type String, File or byte[]</span></li>
        <li><span
               >delete-source-file - if payload type is File, it tells whether delete that file</span>
        </li>
        <li><span
               >filename-generator should be clever enough; if it generates non-unique names it's the programmer's faul</span><span
               >t</span></li>
        <li><span
               >writing file is first performed to another temp file with special extension, and only at the end the file is renamed (moved)</span>
        </li>
        <li><span>there's also analogical </span><span
                style="font-family: &quot;courier new&quot; , &quot;courier&quot; , monospace;">&lt;file:outbound-gateway/&gt;</span>
        </li>
    </ul></div>

##### Reading the file

<div><span
        style="font-family: &quot;courier new&quot; , &quot;courier&quot; , monospace;">&lt;file:inbound-channel-adapter</span>
</div>
<div><span
        style="font-family: &quot;courier new&quot; , &quot;courier&quot; , monospace;">&nbsp; &nbsp; channel="incomingChanges"</span>
</div>
<div><span
        style="font-family: &quot;courier new&quot; , &quot;courier&quot; , monospace;">&nbsp; &nbsp; directory="#{config.diary.store}"</span>
</div>
<div><span
        style="color: #999999; font-family: &quot;courier new&quot; , &quot;courier&quot; , monospace;">&nbsp; &nbsp; filter="myFilter"</span>
</div>
<div><span style="font-family: &quot;courier new&quot; , &quot;courier&quot; , monospace;"><span
        style="color: #999999;">&nbsp; &nbsp; scanner="myScanner"</span></span></div>
<div><span style="font-family: &quot;courier new&quot; , &quot;courier&quot; , monospace;"><span
        style="color: #999999;">&nbsp; &nbsp; comparator="myComparator"</span></span></div>
<div><span style="font-family: &quot;courier new&quot; , &quot;courier&quot; , monospace;"><span
        style="color: #999999;">&nbsp; &nbsp; filename-pattern="..."</span></span></div>
<div><span style="font-family: &quot;courier new&quot; , &quot;courier&quot; , monospace;"><span
        style="color: #999999;">&nbsp; &nbsp; filename-regex="..."</span>&gt;</span></div>
<div><span
        style="font-family: &quot;courier new&quot; , &quot;courier&quot; , monospace;">&nbsp; &lt;poller /&gt;</span>
</div>
<div><span style="font-family: &quot;courier new&quot; , &quot;courier&quot; , monospace;">&lt;/</span><span
        style="font-family: &quot;courier new&quot; , &quot;courier&quot; , monospace;">file:inbound-channel-adapter&gt;</span>
</div>
<div>
    <ul>
        <li><span
               >by overriding the filter, you override the default F</span>ileListFilters, which are by default set in such a way to match with the outbound-channel and not pick up temporary files
        </li>
        <li>by overriding scanner you also override the filter, as the scanner uses the filter (defines how to scan e.g. subdirectories)</li>
        <li>comparator says in which order to read files</li>
        <li>filename-pattern and regex should not be used together with the custom filter</li>
        <li>by default, in the default filter, Spring remembers all the read files not to read them twice</li>
        <li>the reader maintains an internal queue, which it populated on calling .receive() by listing the directory content; the queue is prioritized by using the comparator, if present</li>
    </ul>
    <div>For preventing reading unfinished files you can also use locking mechanism, but don't use it if you can. If you use Spring's moving file strategy you don't need locking.</div></div>

##### Transformers

<ul>
    <li>FileToByteArrayTransformer (&lt;file:file-to-bytes-transformer/&gt;)</li>
    <li>FileToStringTransformer&nbsp;(&lt;file:file-to-string-transformer/&gt;)</li>
    <li>they both have "delete-files" property for deleting file on consumption</li>
</ul>

#### 4.5. Web Services Integration

<div>The good side is that the information exchange protocol and format is clear: HTTP and XML (or JSON). And no firewalls in between.</div>
<div><br/></div>
<div>Well, ok maybe not everything is exactly clear, namely how to use HTTP to transfer XML: some argue on SOAP and WSDL, others do not give a shit about those technologies anymore and use REST. Spring does give a shit about both approaches.</div>

##### POX and SOAP Web Services
<div>POX means not fox, but Plain Old Xml. It is still not same as REST, as the main difference of those services to REST that they use only POST and/or GET method for all the operations. In other words, they don't put semantics to the HTTP method. Another characeristic is that they are "contract-first" Web Services, and are described by WSDL.</div>
<div><br/></div>
<div>Spring WS already gives support (webServiceClient) for creating e.g. SOAP message, which would be otherwise crazy to do manually (envelope, body, this stuff).</div>
<div><br/></div>
<div>Spring Integration builds on top of Spring WS, and lets create endpoints which behave like WS, or are able to consume WS. Minimal configuration for receiving requests is as follows:</div>
<div><br/></div>
<div><span style="font-family: &quot;courier new&quot; , &quot;courier&quot; , monospace;"><u>web.xml:</u></span>
</div>
<div><span
        style="font-family: &quot;courier new&quot; , &quot;courier&quot; , monospace; font-size: x-small;">&lt;servlet&gt;</span>
</div>
<div><span style="font-family: &quot;courier new&quot; , &quot;courier&quot; , monospace; font-size: x-small;">&nbsp; &lt;servlet-name&gt;<b>si-ws-gateway</b>&lt;/servlet-name&gt;</span>
</div>
<div><span
        style="font-family: &quot;courier new&quot; , &quot;courier&quot; , monospace; font-size: x-small;">&nbsp; &lt;servlet-class&gt;....MessageDispatcherServlet&lt;/servlet-class&gt;</span>
</div>
<div><span
        style="font-family: &quot;courier new&quot; , &quot;courier&quot; , monospace; font-size: x-small;">&nbsp; &nbsp;&lt;init-param&gt;</span>
</div>
<div><span
        style="font-family: &quot;courier new&quot; , &quot;courier&quot; , monospace; font-size: x-small;">&nbsp; &nbsp; &nbsp;&lt;param-name&gt;contextConfigLocation&lt;/param-name&gt;</span>
</div>
<div><span style="font-family: &quot;courier new&quot; , &quot;courier&quot; , monospace;"><span
        style="font-size: x-small;">&nbsp; &nbsp; &nbsp;&lt;param-value&gt;si-ws-gateway-config.xml&lt;/param-value&gt;</span></span>
</div>
<div><span style="font-family: &quot;courier new&quot; , &quot;courier&quot; , monospace;"><span
        style="font-size: x-small;">&nbsp; &lt;load-on-startup&gt;1&lt;/load-on-startup&gt;</span></span></div>
<div><span style="font-family: &quot;courier new&quot; , &quot;courier&quot; , monospace;"><span
        style="font-size: x-small;">&lt;/servlet&gt;</span></span></div>
<div><span style="font-family: &quot;courier new&quot; , &quot;courier&quot; , monospace;"><span
        style="font-size: x-small;"><br/></span></span></div>
<div><span style="font-family: &quot;courier new&quot; , &quot;courier&quot; , monospace;"><span
        style="font-size: x-small;">&lt;servlet-mapping&gt;</span></span></div>
<div><span style="font-family: &quot;courier new&quot; , &quot;courier&quot; , monospace;"><span
        style="font-size: x-small;">&nbsp; &lt;servlet-name&gt;<b>si-ws-gateway</b>&lt;/servlet-name&gt;</span></span>
</div>
<div><span style="font-family: &quot;courier new&quot; , &quot;courier&quot; , monospace;"><span
        style="font-size: x-small;">&nbsp; &lt;url-pattern&gt;/quoteservice&lt;/url-pattern&gt;</span></span></div>
<div><span style="font-family: &quot;courier new&quot; , &quot;courier&quot; , monospace;"><span
        style="font-size: x-small;">&lt;/servlet-mapping&gt;</span></span></div>
<div><span style="font-family: &quot;courier new&quot; , &quot;courier&quot; , monospace;"><span
        style="font-size: x-small;"><br/></span></span></div>
<div><u><span
        style="font-family: &quot;courier new&quot; , &quot;courier&quot; , monospace; font-size: x-small;">si-ws-gateway-config.xml</span><span
        style="font-family: &quot;courier new&quot; , &quot;courier&quot; , monospace; font-size: x-small;">:</span></u>
</div>
<div><span
        style="font-family: &quot;courier new&quot; , &quot;courier&quot; , monospace; font-size: x-small;">&lt;bean class="....UriEndpointMapping"&gt;</span>
</div>
<div><span style="font-family: &quot;courier new&quot; , &quot;courier&quot; , monospace; font-size: x-small;">&nbsp; &lt;property name="defaultEndpoint" ref="<b>ws-inbound-gateway</b>"/&gt;</span>
</div>
<div><span
        style="font-family: &quot;courier new&quot; , &quot;courier&quot; , monospace; font-size: x-small;">&lt;/bean&gt;</span>
</div>
<div><span
        style="font-family: &quot;courier new&quot; , &quot;courier&quot; , monospace; font-size: x-small;"><br/></span>
</div>
<div><span style="font-family: &quot;courier new&quot; , &quot;courier&quot; , monospace; font-size: x-small;">&lt;int-ws:inbound-gateway id="<b>ws-inbound-gateway</b>"</span>
</div>
<div><span
        style="font-family: &quot;courier new&quot; , &quot;courier&quot; , monospace; font-size: x-small;">&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; request-channel="ws-requests"</span>
</div>
<div><span style="font-family: &quot;courier new&quot; , &quot;courier&quot; , monospace; font-size: x-small;"><span
        style="color: #999999;">&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; extract-payload="false"</span>/&gt;</span>
</div>
<div><span
        style="font-family: &quot;courier new&quot; , &quot;courier&quot; , monospace; font-size: x-small;"><br/></span>
</div>
<div><span>If you need to make requests, do it like this:</span></div>
<div><span><br/></span></div>
<div><span
        style="font-family: &quot;courier new&quot; , &quot;courier&quot; , monospace;">&lt;int-ws:outbound-gateway</span>
</div>
<div><span
        style="font-family: &quot;courier new&quot; , &quot;courier&quot; , monospace;">&nbsp; &nbsp;uri="http://blblah"</span>
</div>
<div><span
        style="font-family: &quot;courier new&quot; , &quot;courier&quot; , monospace;">&nbsp; &nbsp;request-channel="requests"</span>
</div>
<div><span
        style="font-family: &quot;courier new&quot; , &quot;courier&quot; , monospace;">&nbsp; &nbsp;reply-channel="responses"/&gt;</span>
</div>

##### HTTP Web Services (meaning REST)

<div>They are also the ones that Spring MVC exposes.</div>
<div><br/></div>
<div>
    <div><span style="font-family: &quot;courier new&quot; , &quot;courier&quot; , monospace;"><u>web.xml:</u></span>
    </div>
    <div><span
            style="font-family: &quot;courier new&quot; , &quot;courier&quot; , monospace; font-size: x-small;">&lt;servlet&gt;</span>
    </div>
    <div><span style="font-family: &quot;courier new&quot; , &quot;courier&quot; , monospace; font-size: x-small;">&nbsp; &lt;servlet-name&gt;<b>http-ws-gateway</b>&lt;/servlet-name&gt;</span>
    </div>
    <div><span
            style="font-family: &quot;courier new&quot; , &quot;courier&quot; , monospace; font-size: x-small;">&nbsp; &lt;servlet-class&gt;....HttpRequestHandlerServlet&lt;/servlet-class&gt;</span>
    </div>
    <div><span
            style="font-family: &quot;courier new&quot; , &quot;courier&quot; , monospace; font-size: x-small;">&nbsp; &nbsp;&lt;init-param&gt;</span>
    </div>
    <div><span
            style="font-family: &quot;courier new&quot; , &quot;courier&quot; , monospace; font-size: x-small;">&nbsp; &nbsp; &nbsp;&lt;param-name&gt;contextConfigLocation&lt;/param-name&gt;</span>
    </div>
    <div><span style="font-family: &quot;courier new&quot; , &quot;courier&quot; , monospace;"><span
            style="font-size: x-small;">&nbsp; &nbsp; &nbsp;&lt;param-value&gt;http-ws-gateway.xml&lt;/param-value&gt;</span></span>
    </div>
    <div><span style="font-family: &quot;courier new&quot; , &quot;courier&quot; , monospace;"><span
            style="font-size: x-small;">&lt;/servlet&gt;</span></span></div>
    <div><span style="font-family: &quot;courier new&quot; , &quot;courier&quot; , monospace;"><span
            style="font-size: x-small;"><br/></span></span></div>
    <div><span style="font-family: &quot;courier new&quot; , &quot;courier&quot; , monospace;"><span
            style="font-size: x-small;">&lt;servlet-mapping&gt;</span></span></div>
    <div><span style="font-family: &quot;courier new&quot; , &quot;courier&quot; , monospace;"><span
            style="font-size: x-small;">&nbsp; &lt;servlet-name&gt;<b>http-ws-gateway</b>&lt;/servlet-name&gt;</span></span>
    </div>
    <div><span style="font-family: &quot;courier new&quot; , &quot;courier&quot; , monospace;"><span
            style="font-size: x-small;">&nbsp; &lt;url-pattern&gt;/httpquote&lt;/url-pattern&gt;</span></span></div>
    <div><span style="font-family: &quot;courier new&quot; , &quot;courier&quot; , monospace;"><span
            style="font-size: x-small;">&lt;/servlet-mapping&gt;</span></span></div>
    <div><span style="font-family: &quot;courier new&quot; , &quot;courier&quot; , monospace;"><span
            style="font-size: x-small;"><br/></span></span></div>
    <div><u><span
            style="font-family: &quot;courier new&quot; , &quot;courier&quot; , monospace; font-size: x-small;">http-ws-gateway.xml</span><span
            style="font-family: &quot;courier new&quot; , &quot;courier&quot; , monospace; font-size: x-small;">:</span></u>
    </div>
    <div><u><span
            style="font-family: &quot;courier new&quot; , &quot;courier&quot; , monospace; font-size: x-small;"><br/></span></u>
    </div>
    <div><span
            style="font-family: &quot;courier new&quot; , &quot;courier&quot; , monospace; font-size: x-small;">&lt;int-http:inbound-gateway&nbsp;</span>
    </div>
    <div><span
            style="font-family: &quot;courier new&quot; , &quot;courier&quot; , monospace; font-size: x-small;">&nbsp; id="http-inbound-gateway"</span>
    </div>
    <div><span
            style="font-family: &quot;courier new&quot; , &quot;courier&quot; , monospace; font-size: x-small;">&nbsp; request-channel="http-request"</span>
    </div>
    <div><span
            style="font-family: &quot;courier new&quot; , &quot;courier&quot; , monospace; font-size: x-small;">&nbsp; reply-channel="http-response"</span>
    </div>
    <div><span style="font-family: &quot;courier new&quot; , &quot;courier&quot; , monospace; font-size: x-small;"><span
            style="color: #999999;">&nbsp; extract-reply-payload="false"</span></span></div>
    <div><span
            style="color: #999999; font-family: &quot;courier new&quot; , &quot;courier&quot; , monospace; font-size: x-small;">&nbsp; view-name="about"</span>
    </div>
    <div><span style="color: #999999;"><span
            style="font-family: &quot;courier new&quot; , &quot;courier&quot; , monospace; font-size: x-small;">&nbsp; reply-key, reply-timeout,</span><span
            style="font-family: &quot;courier new&quot; , &quot;courier&quot; , monospace; font-size: x-small;">message-converters,&nbsp;</span></span>
    </div>
    <div><span
            style="color: #999999; font-family: &quot;courier new&quot; , &quot;courier&quot; , monospace; font-size: x-small;">&nbsp; supported-methods, convert-exceptions,&nbsp;</span>
    </div>
    <div><span
            style="color: #999999; font-family: &quot;courier new&quot; , &quot;courier&quot; , monospace; font-size: x-small;">&nbsp; request-payload-type, error-code, errors-key,</span>
    </div>
    <div><span style="font-family: &quot;courier new&quot; , &quot;courier&quot; , monospace; font-size: x-small;"><span
            style="color: #999999;">&nbsp; header-mapper, name</span>/&gt;</span></div>
    <div>
        <ul>
            <li><span>name - e.g. "/subscribe", so that it allows it to be used with DispatcherServlet</span>
            </li>
            <li><span>view-name - is the Spring MVC view name</span></li>
            <li>you can also use inbound-message-adapter if you don't need two way communication, it uses MessageTemplate</li>
        </ul>
    </div>
    <div><span>If you need to make requests, do it like this:</span></div>
    <div><span><br/></span></div>
    <div><span
            style="font-family: &quot;courier new&quot; , &quot;courier&quot; , monospace;">&lt;int-http:outbound-gateway</span>
    </div>
    <div><span
            style="font-family: &quot;courier new&quot; , &quot;courier&quot; , monospace;">&nbsp; &nbsp; url="http://blblah"</span>
    </div>
    <div><span
            style="font-family: &quot;courier new&quot; , &quot;courier&quot; , monospace;">&nbsp; &nbsp; request-channel="requests"</span>
    </div>
    <div><span
            style="font-family: &quot;courier new&quot; , &quot;courier&quot; , monospace;">&nbsp; &nbsp; http-method="GET"</span>
    </div>
    <div><span
            style="font-family: &quot;courier new&quot; , &quot;courier&quot; , monospace;">&nbsp; &nbsp; expected-response-type="java.lang.String"&gt;</span>
    </div>
    <div><span
            style="font-family: &quot;courier new&quot; , &quot;courier&quot; , monospace;">&nbsp; &nbsp; &nbsp; &lt;int-http:uri-variable&nbsp;</span>
    </div>
    <div><span
            style="font-family: &quot;courier new&quot; , &quot;courier&quot; , monospace;">&nbsp; &nbsp; &nbsp; &nbsp; name="location"&nbsp;</span>
    </div>
    <div><span
            style="font-family: &quot;courier new&quot; , &quot;courier&quot; , monospace;">&nbsp; &nbsp; &nbsp; &nbsp; expression="payload"/&gt;</span>
    </div>
    <div><span
            style="font-family: &quot;courier new&quot; , &quot;courier&quot; , monospace;">&lt;/int:http:outbound-gateway&gt;</span>
    </div>
    <div>
        <ul>
            <li>
                <span>you can use outbound-channel-adapter if you don't need two way communication, it uses RestTemplate;</span>
            </li>
            <li>
                <span>in the case above it's better to override the error handler, as the default one treats only 4** and 5** responses as errors</span>
            </li>
        </ul>
    </div>
</div>

#### 4.6. XMPP and Twitter
<div>XMPP (Extensible Messaging and Presence Protocol) is protocol especially for chatting. It can send messages (both directions same time) and presence notifications. Orginal server implemeentation is called Jabber.</div>
<div><br/></div>
<div>And Twitter has an exposed API.</div>
<div><br/></div>
<div>So Spring Integration is providing endpoints for these as well. The namespaces are <span
        style="font-family: &quot;courier new&quot; , &quot;courier&quot; , monospace;">int-xmpp </span>and <span
        style="font-family: &quot;courier new&quot; , &quot;courier&quot; , monospace;">int-twitter</span>, respectively. I will not go into details. But some things I learned by the way:
</div>
<div>
    <ul>
        <li>annotating a method with @Publisher("channelName"), and enabling &lt;int:annotation-config/&gt; will make the return value of that method be additionally published on that channel</li>
        <li>it's possible to set the global publishing channel, by setting:<br/>&lt;int:annotation-config default-publisher-channel="channelName"/&gt;
        </li>
        <li>an alternative to the above is defining an &lt;int:gateway/&gt; with service-interface set to our custom interface, and such bean will be created automagically and we can autowire it, and call our method.. cool.. and in this method we can even add @Header(XmppHeaders.TO) String username input parameter; but there are also xmpp header enrichers fir that;</li>
        <li>there's sth like xmppConnection bean</li>
        <li>the Twitter API needs authentication, and it is done by configuring the twitterTemplate bean</li>
    </ul>
</div>

### 5. Advanced Topics

(means the ones he could not find a way to group under common name:P)<br/>

#### 5.1. Monitoring

##### Message history

<div>If you add to your spring config this:</div>
<div><br/></div>
<div><span
        style="font-family: &quot;courier new&quot; , &quot;courier&quot; , monospace;">&lt;int:message-history/&gt;</span>
</div>
<div><br/></div>
<div>then every message will have a header called "history" with entries for each endpoint it visited, each entry contains endpoint name, type and timestamp. Neat!</div>

<b>Wire tapCan be used to record e.g. endpoint statistics. It was described above how to use it, it's an interceptor. Remember about transaction boundaries, e.g. if you use this after wire tap:</b><br/><span
                style="font-family: &quot;courier new&quot; , &quot;courier&quot; , monospace;">&lt;channel ...&gt;</span><br/><span
                style="font-family: &quot;courier new&quot; , &quot;courier&quot; , monospace;">&nbsp; &nbsp;&lt;dispatcher task-executor="someExecutor"/&gt;</span><br/><span
                style="font-family: &quot;courier new&quot; , &quot;courier&quot; , monospace;">&lt;/channel&gt;</span><br/><span
                style="font-family: &quot;courier new&quot; , &quot;courier&quot; , monospace;"><br/></span><span
               >then bye bye transaction.</span><br/>

##### JMX

<div><span>I skip now, many adapters</span></div>

##### Control Bus

<div><span
       >I skip now, it's for sending a command message which managed endpoints</span>
</div>

#### 5.2. Scheduling and concurrency

<span>Pollers</span>
<div><span
       >Used for receiving messages transmitted asynchronously, as well as publishing messages, in time intervals. You can define a general poller definition and then only override what you need, e.g.:</span>
</div>
<div><span><br/></span></div>
<div><span
        style="font-family: &quot;courier new&quot; , &quot;courier&quot; , monospace;">&lt;poller id="defaultPoller" fixed-delay="500" default="true"/&gt;</span>
</div>
<div><span style="font-family: &quot;courier new&quot; , &quot;courier&quot; , monospace;"><br/></span></div>
<div><span style="font-family: &quot;courier new&quot; , &quot;courier&quot; , monospace;">...</span></div>
<div><span style="font-family: &quot;courier new&quot; , &quot;courier&quot; , monospace;"><br/></span></div>
<div><span
        style="font-family: &quot;courier new&quot; , &quot;courier&quot; , monospace;">&lt;file:inbound-channel-adapter id, ...&gt;</span>
</div>
<div><span
        style="font-family: &quot;courier new&quot; , &quot;courier&quot; , monospace;">&nbsp; &nbsp;&lt;poller fixed-rate="10000"/&gt;</span>
</div>
<div><span
        style="font-family: &quot;courier new&quot; , &quot;courier&quot; , monospace;">&lt;/file:inbound-channel-adapter&gt;</span>
</div>
<div>
    <ul>
        <li>fixed-delay and fixed-rate are different ways to specify the polling interval; rate happens every x miliseconds, and delay assures that there is a gap of x seconds between each two polls;</li>
        <li>the example above will use "rate" in the end</li>
    </ul>
    <div>You can also define the poller with cron expression:</div>
</div>
<div><br/></div>
<div><span
        style="font-family: &quot;courier new&quot; , &quot;courier&quot; , monospace;">&lt;poller cron="0 0 0 * * *"/&gt;</span>
</div>
<div><span style="font-family: &quot;courier new&quot; , &quot;courier&quot; , monospace;"><br/></span></div>
<div><span>which stands for:</span></div>
<div><span style="font-family: &quot;courier new&quot; , &quot;courier&quot; , monospace;"><br/></span></div>
<div><span style="font-family: &quot;courier new&quot; , &quot;courier&quot; , monospace;"><span
        style="font-size: x-small;">sec, min, hour, dayofthe-month, month, dayofthe-week (, year)</span></span>
</div>
<div><br/></div>
<div>other options:</div>
<div><br/></div>
<div><span
        style="font-family: &quot;courier new&quot; , &quot;courier&quot; , monospace;">&lt;poller fixed-rate="10000"</span>
</div>
<div><span
        style="font-family: &quot;courier new&quot; , &quot;courier&quot; , monospace;">&nbsp; &nbsp; &nbsp; &nbsp; max-messages-per-pool="2"&nbsp;</span>
</div>
<div><span
        style="font-family: &quot;courier new&quot; , &quot;courier&quot; , monospace;">&nbsp; &nbsp; &nbsp; &nbsp; receive-timeout="2000"/&gt;</span>
</div>
<div>
    <ul>
        <li>max messages per pool - how many messages will be processed at one poll operation</li>
        <li>receive timeout - if nothing is there for that amount of time, give up; watch out, also: if you aleady started give up;</li>
    </ul>
    <div>You can use poller for publishing e.g. by including it inside an inbound-channel-adapter. No one calls the adapter, but poller does.</div>
</div>

##### Task Executors

<div>By deafult everything is done in single thread. To change that, either change a channel to a queue channel, or add a task executor:</div>

<div><span style="font-family: &quot;courier new&quot; , &quot;courier&quot; , monospace;">&lt;channel ...&gt;</span><br/><span
            style="font-family: &quot;courier new&quot; , &quot;courier&quot; , monospace;">&nbsp; &nbsp;&lt;dispatcher task-executor="someExecutor"/&gt;</span><br/><span
            style="font-family: &quot;courier new&quot; , &quot;courier&quot; , monospace;">&lt;/channel&gt;</span><br/><span
            style="font-family: &quot;courier new&quot; , &quot;courier&quot; , monospace;"><br/></span><span
           >Executor will introduce less lag, but is also more simple than queue (no prioritization, persistence, blah).</span><br/><span
           ><br/></span><span
            style="font-family: &quot;courier new&quot; , &quot;courier&quot; , monospace;">&lt;task:executor id="someExecutor"</span><br/><span
            style="font-family: &quot;courier new&quot; , &quot;courier&quot; , monospace;">&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp;pool-size="2-5"</span><br/><span
            style="font-family: &quot;courier new&quot; , &quot;courier&quot; , monospace;">&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp;queue-capacity="100"/&gt;</span><br/><br/>
        <ul>
            <li>pool-size - number of threads available, here 2 and if needed can go up to 5</li>
            <li>queue-capacity - how many tasks can be queued waiting for free thread</li>
        </ul>
        <br/><span
               >You can push your existing executor also into another places, like:</span><br/><br/>1. pub sub channel:<br/><br/><span
                style="font-family: &quot;courier new&quot; , &quot;courier&quot; , monospace;">&lt;publish-subscribe-channel id, task-executor="executor"/&gt;</span><br/>
        <ul>
            <li>this will cause that each subscriber will process the message in separate thread</li>
        </ul>
        <div>2. Poller</div>
        <div><br/></div>
        <div><span
                style="font-family: &quot;courier new&quot; , &quot;courier&quot; , monospace;">&lt;poller id, fixed-delay, task-executor="executor"/&gt;</span>
        </div>
        <div>
            <ul>
                <li><span
                       >why would you do that: if polling takes too much time, the other scheduled polls may have to wait; with this solution they will be executed in other threads;</span>
                </li>
            </ul>

##### Task Scheduler</div>

<div><span
            style="font-family: &quot;courier new&quot; , &quot;courier&quot; , monospace; font-size: x-small;">public interface TaskScheduler {</span>
    </div>
    <div><span
            style="font-family: &quot;courier new&quot; , &quot;courier&quot; , monospace; font-size: x-small;">&nbsp; ScheduledFuture schedule(Runnable task, Trigger trigger);</span>
    </div>
    <div><span
            style="font-family: &quot;courier new&quot; , &quot;courier&quot; , monospace; font-size: x-small;">&nbsp; ScheduledFuture schedule(Runnable task, Date startTime);</span>
    </div>
    <div><span
            style="font-family: &quot;courier new&quot; , &quot;courier&quot; , monospace; font-size: x-small;">&nbsp; ScheduledFuture schedule(Runnable task, Date startTime, long period);</span>
    </div>
    <div><span
            style="font-family: &quot;courier new&quot; , &quot;courier&quot; , monospace; font-size: x-small;">&nbsp; ScheduledFuture schedule(Runnable task, long period);</span>
    </div>
    <div><span
            style="font-family: &quot;courier new&quot; , &quot;courier&quot; , monospace; font-size: x-small;">&nbsp; ScheduledFuture schedule(Runnable task, Date startTime, long delay);</span>
    </div>
    <div><span
            style="font-family: &quot;courier new&quot; , &quot;courier&quot; , monospace; font-size: x-small;">&nbsp; ScheduledFuture schedule(Runnable task, long delay);</span>
    </div>
    <div><span
            style="font-family: &quot;courier new&quot; , &quot;courier&quot; , monospace; font-size: x-small;">}</span>
    </div>
    <div><span
            style="font-family: &quot;courier new&quot; , &quot;courier&quot; , monospace; font-size: x-small;"><br/></span>
    </div>
    <div><span
            style="font-family: &quot;courier new&quot; , &quot;courier&quot; , monospace; font-size: x-small;">public interface Trigger {</span>
    </div>
    <div><span
            style="font-family: &quot;courier new&quot; , &quot;courier&quot; , monospace; font-size: x-small;">&nbsp; Date nextExecutionTime(TriggerContext context);</span>
    </div>
    <div><span
            style="font-family: &quot;courier new&quot; , &quot;courier&quot; , monospace; font-size: x-small;">}</span>
    </div>
    <div><span
            style="font-family: &quot;courier new&quot; , &quot;courier&quot; , monospace; font-size: x-small;"><br/></span>
    </div>
    <div><span
            style="font-family: &quot;courier new&quot; , &quot;courier&quot; , monospace; font-size: x-small;">public interface TriggerContext {</span>
    </div>
    <div><span
            style="font-family: &quot;courier new&quot; , &quot;courier&quot; , monospace; font-size: x-small;">&nbsp; Date lastScheduledExecutionTime();</span>
    </div>
    <div><span
            style="font-family: &quot;courier new&quot; , &quot;courier&quot; , monospace; font-size: x-small;">&nbsp; Date lastActualExecutionTime();</span>
    </div>
    <div><span
            style="font-family: &quot;courier new&quot; , &quot;courier&quot; , monospace; font-size: x-small;">&nbsp; Date lastCompletionTime();</span>
    </div>
    <div><span
            style="font-family: &quot;courier new&quot; , &quot;courier&quot; , monospace; font-size: x-small;">}</span>
    </div>
    <div><br/></div>
    <div><span>(no XML example..)</span></div>
</div>

#### 5.3. Spring Batch
<div>
    <b>JobJob - requires no manual intervention, but status should be able to be seen, also restart should be possible in the place when it finished, and stream processing is also important (memory limitations). Spring Batch is the solution.</b>
</div>
<div><br/></div>
<div>JobParameters - identify the job instance, same instance cannot be run twice thats why add there some counter.</div>
<div><br/></div>
<div>Chunks are important concept, we read and write data in chunks of optimal size, and a transaction spans from beginning to the end of a chunk.</div>
<div><br/></div>
<div><span
        style="font-family: &quot;courier new&quot; , &quot;courier&quot; , monospace; font-size: x-small;">&lt;batch:job id="bla"&gt;</span>
</div>
<div><span
        style="font-family: &quot;courier new&quot; , &quot;courier&quot; , monospace; font-size: x-small;">&nbsp; &lt;batch:step id="loadPayments"&gt;</span>
</div>
<div><span
        style="font-family: &quot;courier new&quot; , &quot;courier&quot; , monospace; font-size: x-small;">&nbsp; &nbsp; &lt;batch:tasklet&gt;</span>
</div>
<div><span
        style="font-family: &quot;courier new&quot; , &quot;courier&quot; , monospace; font-size: x-small;">&nbsp; &nbsp; &nbsp; &lt;batch:chunk&nbsp;</span>
</div>
<div><span
        style="font-family: &quot;courier new&quot; , &quot;courier&quot; , monospace; font-size: x-small;">&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; reader="itemReader"&nbsp;</span>
</div>
<div><span
        style="font-family: &quot;courier new&quot; , &quot;courier&quot; , monospace; font-size: x-small;">&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; writer="itemWriter"&nbsp;</span>
</div>
<div><span
        style="font-family: &quot;courier new&quot; , &quot;courier&quot; , monospace; font-size: x-small;">&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; commit-interval="5"/&gt;</span>
</div>
<div><span
        style="font-family: &quot;courier new&quot; , &quot;courier&quot; , monospace; font-size: x-small;">&nbsp; &nbsp; &lt;/batch:tasklet&gt;</span>
</div>
<div><span
        style="font-family: &quot;courier new&quot; , &quot;courier&quot; , monospace; font-size: x-small;">&nbsp; &lt;/batch:step&gt;</span>
</div>
<div><span
        style="font-family: &quot;courier new&quot; , &quot;courier&quot; , monospace; font-size: x-small;">&lt;/batch:job&gt;</span><br/><span
        style="font-family: &quot;courier new&quot; , &quot;courier&quot; , monospace; font-size: x-small;"><br/></span><span
        style="font-family: &quot;courier new&quot; , &quot;courier&quot; , monospace; font-size: xx-small;">&lt;bean id="itemReader" class="....FlatFileItemReader" scope="step"&gt;</span><br/><span
        style="font-size: xx-small;"><span
        style="font-family: &quot;courier new&quot; , &quot;courier&quot; , monospace;">&nbsp; &lt;property name="resource" value="file://#{jobParameters['filena']}</span><span
        style="font-family: &quot;courier new&quot; , &quot;courier&quot; , monospace;">"/&gt;</span></span><br/><span
        style="font-family: &quot;courier new&quot; , &quot;courier&quot; , monospace; font-size: xx-small;">&nbsp; &lt;property name="lineMapper"&gt;</span><br/><span
        style="font-family: &quot;courier new&quot; , &quot;courier&quot; , monospace; font-size: xx-small;"><br/></span><span
        style="font-family: &quot;courier new&quot; , &quot;courier&quot; , monospace; font-size: xx-small;">&nbsp; &nbsp; &lt;bean class="....DefaultLineMapper"&gt;</span><br/><span
        style="font-family: &quot;courier new&quot; , &quot;courier&quot; , monospace; font-size: xx-small;"><br/></span><span
        style="font-family: &quot;courier new&quot; , &quot;courier&quot; , monospace; font-size: xx-small;">&nbsp; &nbsp; &nbsp; &lt;property name="lineTokenizer"&gt;</span><br/><span
        style="font-family: &quot;courier new&quot; , &quot;courier&quot; , monospace; font-size: xx-small;">&nbsp; &nbsp; &nbsp; &nbsp; &lt;bean class="....DelimitedLineTokenizer"&gt;</span><br/><span
        style="font-family: &quot;courier new&quot; , &quot;courier&quot; , monospace; font-size: xx-small;">&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &lt;property name="names" value="source,dest,amount,date"/&gt;</span><br/><span
        style="font-family: &quot;courier new&quot; , &quot;courier&quot; , monospace; font-size: xx-small;">&nbsp; &nbsp; &nbsp; &nbsp; &lt;/bean&gt;</span><br/><span
        style="font-family: &quot;courier new&quot; , &quot;courier&quot; , monospace; font-size: xx-small;">&nbsp; &nbsp; &nbsp; &lt;/property&gt;</span><br/><span
        style="font-family: &quot;courier new&quot; , &quot;courier&quot; , monospace; font-size: xx-small;"><br/></span><span
        style="font-family: &quot;courier new&quot; , &quot;courier&quot; , monospace; font-size: xx-small;">&nbsp; &nbsp; &nbsp; &lt;property name="fieldSetMapper"&gt;</span><br/><span
        style="font-family: &quot;courier new&quot; , &quot;courier&quot; , monospace; font-size: xx-small;">&nbsp; &nbsp; &nbsp; &nbsp; &lt;bean class="....MyMapper"/&gt;</span><br/><span
        style="font-family: &quot;courier new&quot; , &quot;courier&quot; , monospace; font-size: xx-small;">&nbsp; &nbsp; &nbsp; &lt;/property&gt;</span><br/><span
        style="font-family: &quot;courier new&quot; , &quot;courier&quot; , monospace; font-size: xx-small;"><br/></span><span
        style="font-family: &quot;courier new&quot; , &quot;courier&quot; , monospace; font-size: xx-small;">&nbsp; &nbsp; &lt;/bean&gt;</span><br/><span
        style="font-family: &quot;courier new&quot; , &quot;courier&quot; , monospace; font-size: xx-small;"><br/></span><span
        style="font-family: &quot;courier new&quot; , &quot;courier&quot; , monospace; font-size: xx-small;">&nbsp; &lt;/property&gt;</span><br/><span
        style="font-family: &quot;courier new&quot; , &quot;courier&quot; , monospace; font-size: xx-small;">&lt;/bean&gt;</span><br/><br/>
    <ul>
        <li><span>delegating to the tasklet eliminates the boilerplate code connected to events, maintaining state, etc.</span>
        </li>
    </ul>
    <div><span
            style="font-family: &quot;courier new&quot; , &quot;courier&quot; , monospace; font-size: x-small;">public class MyMapper implements FieldSetMapper&lt;Payment&gt;{</span>
    </div>
    <div><span
            style="font-family: &quot;courier new&quot; , &quot;courier&quot; , monospace; font-size: x-small;">&nbsp; @Override</span>
    </div>
    <div><span
            style="font-family: &quot;courier new&quot; , &quot;courier&quot; , monospace; font-size: x-small;">&nbsp; public Payment mapFieldSet(FieldSet fieldSet)&nbsp;</span>
    </div>
    <div><span
            style="font-family: &quot;courier new&quot; , &quot;courier&quot; , monospace; font-size: x-small;">&nbsp; &nbsp; &nbsp; &nbsp;throws BindException {</span>
    </div>
    <div><span
            style="font-family: &quot;courier new&quot; , &quot;courier&quot; , monospace; font-size: x-small;">&nbsp; &nbsp; &nbsp;... = fieldSet.readString("source");</span>
    </div>
    <div><span style="font-family: &quot;courier new&quot; , &quot;courier&quot; , monospace;"><span
            style="font-size: x-small;">&nbsp; &nbsp; &nbsp;... = fieldSet.readBigDecimal("amount");</span></span>
    </div>
    <div><span style="font-family: &quot;courier new&quot; , &quot;courier&quot; , monospace;"><span
            style="font-size: x-small;">&nbsp; &nbsp; &nbsp;... = fieldSet.readDate("date");</span></span></div>
    <div><span
            style="font-family: &quot;courier new&quot; , &quot;courier&quot; , monospace; font-size: x-small;">&nbsp; }</span>
    </div>
    <div><span
            style="font-family: &quot;courier new&quot; , &quot;courier&quot; , monospace; font-size: x-small;">}</span>
    </div>
        </div>

##### Launching the job

<div><span style="font-family: &quot;courier new&quot; , &quot;courier&quot; , monospace; font-size: x-small;">&lt;bean id="jobLancher" class="....SimpleJobLauncher"&gt;</span>
</div>
<div><span
        style="font-family: &quot;courier new&quot; , &quot;courier&quot; , monospace; font-size: x-small;">&nbsp; &nbsp;&lt;property name="jobRepository" ref="jobRepository"/&gt;</span>
</div>
<div><span
        style="font-family: &quot;courier new&quot; , &quot;courier&quot; , monospace; font-size: x-small;">&lt;/bean&gt;</span>
</div>
<div><span
        style="font-family: &quot;courier new&quot; , &quot;courier&quot; , monospace; font-size: x-small;"><br/></span>
</div>
<div><span
        style="font-family: &quot;courier new&quot; , &quot;courier&quot; , monospace; font-size: x-small;">&lt;batch:job-repository&nbsp;</span>
</div>
<div><span
        style="font-family: &quot;courier new&quot; , &quot;courier&quot; , monospace; font-size: x-small;">&nbsp; &nbsp; &nbsp; &nbsp; data-source="dataSource"&nbsp;</span>
</div>
<div><span
        style="font-family: &quot;courier new&quot; , &quot;courier&quot; , monospace; font-size: x-small;">&nbsp; &nbsp; &nbsp; &nbsp; id="jobRepository"&nbsp;</span>
</div>
<div><span
        style="font-family: &quot;courier new&quot; , &quot;courier&quot; , monospace; font-size: x-small;">&nbsp; &nbsp; &nbsp; &nbsp; transaction-manager="transactionManager"&nbsp;</span>
</div>
<div><span
        style="font-family: &quot;courier new&quot; , &quot;courier&quot; , monospace; font-size: x-small;">&nbsp; &nbsp; &nbsp; &nbsp; table-prefix="BATCH_"/&gt;</span>
</div>
<div><br/></div>
<div>To actually launch it you can do:</div>
<div><br/></div>
<div><span
        style="font-family: &quot;courier new&quot; , &quot;courier&quot; , monospace;">JobParametersBuilder jpb = new JobParametersBuilder();</span>
</div>
<div><span
        style="font-family: &quot;courier new&quot; , &quot;courier&quot; , monospace;">jpb.addString('filena', 'payment.xml');</span>
</div>
<div><span
        style="font-family: &quot;courier new&quot; , &quot;courier&quot; , monospace;">JobExecution execution = jobLauncher.run(job, jpb.toJobParameters());</span>
</div>
<div><br/></div>
<div>the launching is sync or async, so in case of async, you can immediately check jobExecution for the status of the job, as this is persisted in db you provided in job repositpry definition.</div>
<div><br/></div>
<div>There is also the Spring Batch Admin web app where you can view job statuses and restart failed ones.</div>

##### Spring Batch and Spring Integration

<div>Spring Integration provides support for everything you can see in Spring Batch Admin.<br/><br/><span
        style="font-family: &quot;courier new&quot; , &quot;courier&quot; , monospace;">&lt;si:service-activator&nbsp;</span><br/><span
        style="font-family: &quot;courier new&quot; , &quot;courier&quot; , monospace;">&nbsp; &nbsp; &nbsp; &nbsp; &nbsp;method="launch"&nbsp;</span><br/><span
        style="font-family: &quot;courier new&quot; , &quot;courier&quot; , monospace;">&nbsp; &nbsp; &nbsp; &nbsp; &nbsp;input-channel="requests"&nbsp;</span><br/><span
        style="font-family: &quot;courier new&quot; , &quot;courier&quot; , monospace;">&nbsp; &nbsp; &nbsp; &nbsp; &nbsp;output-channel="statuses"&gt;</span><br/><span
        style="font-family: &quot;courier new&quot; , &quot;courier&quot; , monospace;">&nbsp; &nbsp;&lt;bean class="....<b>JobLaunchingMessageHandler</b>"/&gt;</span><br/><span
        style="font-family: &quot;courier new&quot; , &quot;courier&quot; , monospace;">&lt;/si:service-activator&gt;</span><br/>
    <ul>
        <li>
            <span>JobLauchingMessageHandler expects a message payload of type JobLaunchRequest, which you can provide for example like this:</span>
        </li>
    </ul>
    <div><span
            style="font-family: &quot;courier new&quot; , &quot;courier&quot; , monospace; font-size: x-small;">...</span>
    </div>
    <div><span
            style="font-family: &quot;courier new&quot; , &quot;courier&quot; , monospace; font-size: x-small;">@Transformer</span>
    </div>
    <div><span
            style="font-family: &quot;courier new&quot; , &quot;courier&quot; , monospace; font-size: x-small;">public JobLaunchRequest toRequest(Message&lt;File&gt; m){</span>
    </div>
    <div><span
            style="font-family: &quot;courier new&quot; , &quot;courier&quot; , monospace; font-size: x-small;">&nbsp; &nbsp;JobParametersBuilder jpb = new JobParametersBuilder();</span>
    </div>
    <div><span
            style="font-family: &quot;courier new&quot; , &quot;courier&quot; , monospace; font-size: x-small;">&nbsp; &nbsp;jpb.addString(fileParameterName, message.getPayload().getAbsolutePath());</span>
    </div>
    <div><span
            style="font-family: &quot;courier new&quot; , &quot;courier&quot; , monospace; font-size: x-small;">&nbsp; &nbsp;return new JobLauchRequest(job, jpb.toJobParameters())</span>
    </div>
    <div><span
            style="font-family: &quot;courier new&quot; , &quot;courier&quot; , monospace; font-size: x-small;">}</span><br/>
    </div>
</div>

##### Event driven integration
<div><span>Spring Batch provides three listeners:</span></div>
<div>
    <ul>
        <li>StepListener</li>
        <li>ChunkListener</li>
        <li>JobExecutionListener</li>
    </ul>
    <div>You use them as gateway and&nbsp;register the gateway at the job:</div>
</div>
<div><br/></div>
<div><span style="font-family: &quot;courier new&quot; , &quot;courier&quot; , monospace;">&lt;<span
        style="font-size: x-small;">batch:job id&gt;</span></span></div>
<div><span
        style="font-family: &quot;courier new&quot; , &quot;courier&quot; , monospace; font-size: x-small;">&nbsp; &nbsp;...</span>
</div>
<div><span
        style="font-family: &quot;courier new&quot; , &quot;courier&quot; , monospace; font-size: x-small;">&nbsp; &nbsp;&lt;batch:listeners&gt;</span>
</div>
<div><span
        style="font-family: &quot;courier new&quot; , &quot;courier&quot; , monospace; font-size: x-small;">&nbsp; &nbsp; &nbsp; &lt;batch:listener ref="notificationListener"/&gt;</span>
</div>
<div><span
        style="font-family: &quot;courier new&quot; , &quot;courier&quot; , monospace; font-size: x-small;">&nbsp; &nbsp;&lt;/batcg:listeners&gt;</span>
</div>
<div><span
        style="font-family: &quot;courier new&quot; , &quot;courier&quot; , monospace; font-size: x-small;">&lt;/batch:job&gt;</span>
</div>
<div><span
        style="font-family: &quot;courier new&quot; , &quot;courier&quot; , monospace; font-size: x-small;"><br/></span>
</div>
<div><span
        style="font-family: &quot;courier new&quot; , &quot;courier&quot; , monospace; font-size: x-small;">&lt;si:gateway id="notificationListener"</span>
</div>
<div><span style="font-family: &quot;courier new&quot; , &quot;courier&quot; , monospace; font-size: x-small;">&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; service-interface="....<b>JobExecutionListener</b>"</span>
</div>
<div><span
        style="font-family: &quot;courier new&quot; , &quot;courier&quot; , monospace; font-size: x-small;">&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; default-request-channel="jobExecutions"/&gt;</span><br/>
</div>

#### 5.3. OSGi

<div>I skip also, not required for the exam.</div>

