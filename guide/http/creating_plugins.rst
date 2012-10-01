==============================
Creating Plugins and Observers
==============================

.. highlight:: php

Guzzle is extremely extendable because of the behavioral modifications that can be added to requests, clients, and commands using an event system.  Before and after the majority of actions are taken in the library, an event is emitted with the name of the event and context surrounding the event.  Observers can subscribe to a subject and modify the subject based on the events received.  Guzzle's event system utilizes the Symfony2 EventDispatcher and is the backbone of it's plugin architecture.

Overview
--------

Plugins must implement the ``Symfony\Component\EventDispatcher\EventSubscriberInterface`` interface.  The EventSubscriberInterface requires that your class implements a static method, ``getSubscribedEvents()``, that returns an associative array mapping events to methods on the object.  See the `Symfony2 documentation <http://symfony.com/doc/2.0/book/internals.html#the-event-dispatcher>`_ for more information.

Plugins can be attached to any subject, or object in Guzzle that implements that ``Guzzle\Common\HasDispatcherInterface``.

Subscribing to a subject
~~~~~~~~~~~~~~~~~~~~~~~~

You can subscribe an instantiated observer to an event by calling ``addSubscriber`` on a subject.

.. code-block:: php

    $testPlugin = new TestPlugin();
    $client->addSubscriber($testPlugin);

You can also subscribe to only specific events using a closure::

    $client->getEventDispatcher()->addListener('request.create', function(Event $event) {
        echo $event->getName();
        echo $event['request'];
    });

``Guzzle\Common\Event`` objects are passed to notified functions.  The Event object has a ``getName()`` method which return the name of the emitted event and may contain contextual information that can be accessed like an array.

Knowing what events to listen to
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Any class that implements the ``Guzzle\Common\HasDispatcherInterface`` must implement a static method, ``getAllEvents()``, that returns an array of the events that are emitted from the object.  You can browse the source to see each event, or you can call the static method directly in your code to get a list of available events.

Event hooks
-----------

+---------------------------------+---------------------------+----------------------------------+-------------------------------------+
| Subject                         | Event                     | Description                      | Arguments                           |
+=================================+===========================+==================================+=====================================+
| ``Guzzle\Http\Client``          | client.create_request     | Client has created a request     | * client: The client                |
|                                 |                           |                                  | * request: The created request      |
+---------------------------------+---------------------------+----------------------------------+-------------------------------------+
| ``Guzzle\Http\Message\Request`` | request.before_send       | About to send request            | * request: Request to be sent       |
|                                 +---------------------------+----------------------------------+-------------------------------------+
|                                 | request.sent              | Sent the request                 | * request: Request that was sent    |
|                                 |                           |                                  | * response: Received response       |
+---------------------------------+---------------------------+----------------------------------+-------------------------------------+

Examples of the event system
----------------------------

Simple Echo plugin
~~~~~~~~~~~~~~~~~~

This simple plugin echos a string containing the request that is about to be sent by listening to the ``request.before_send`` event::

    class EchoPlugin implements Symfony\Component\EventDispatcher\EventSubscriberInterface
    {
        public static function getSubscribedEvents()
        {
            return array(
                'request.before_send' => 'onBeforeSend'
            );
        }

        public function onBeforeSend(Guzzle\Common\Event $event)
        {
            echo 'About to send a request: ' . $event['request'] . "\n";
        }
    }

    $plugin = new EchoPlugin();
    $client = new Guzzle\Service\Client('http://www.test.com/');
    $client->addSubscriber($plugin);
    $client->get('/')->send();


Running the above code will echo a string containing the HTTP request that is about to be sent.
