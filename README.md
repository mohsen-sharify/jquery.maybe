## Maybe

*Schrödinger's cat is the Ultimate Maybe*

A Maybe is an enhanced `$.Deferred` interface, featuring a simple way to specify *when* it should be resolved and/or rejected. If you want to know about Deferred or refresh your knowledge, head to [this review](http://eng.wealthfront.com/2012/12/jquerydeferred-is-most-important-client.html).

This custom interface relies on an evented conception of the promise associated to the deferred object. When using a Maybe, one is expected to specify *evented conditions for success and failure*. Those conditions will be triggering either the deferrable's *resolution* or *rejection* callback. This feature is made possible by exposing a custom Promise to the outside world (the very Maybe), one that accepts `doIf` and `failIf` condition-factories. As with other legacy Deferred's methods, several conditions may be specified in a row.

By default, a Maybe will always succeed, but if it is provided a grace-period (with the `wait` option), it will wait until this delay is over to succeed. This delay may actually be infinite. In the mean time, the Promise/Maybe could be either early-resolved (when an event matching a `doIf` condition occurs) or rejected (when an event matching a `failIf` condition occurs).

## Example

*Say you want to display a notification to several users connected to your application, asking for applying a change from the master user or something. Still, you want to allow for a 10s delay before actually performing the update, so that users can refuse to update by discarding the notification. You thus want to manage some time for the users to decide whether they really want this forced-setting to apply.*

Using jquery.maybe is kind of a low-tech approach; you would simply write:

``` js
$('.my_notification').maybe({wait: 10000})
                     .failIf('click', '.cancel')
                     .done(success)
                     .fail(failure)
                     .always(hide);
```
     
where `success`,`failure` and `hide` are callbacks of your choice. The first two will be called when the action is respectively resolved or rejected; `hide` will always be called, no matter what.

`failIf()` is used to attach a condition for rejection. In the example, we're listening for any click on `$('.cancel', '.my_notification')` during the 10 s lifetime of the action. If such an event occurs, the action is rejected and `success` never fires; instead, `failure` then `hide` do fire. If no click event is catched while in grace-period, the action is resolved: `success` then `hide` are triggered, whereas `failure` never shows up.

## Available options for `maybe()`

* **wait** - in milliseconds, delay before resolving the action (default: `-1`). When `wait` is set to -1, the action is pending for ever and must be resolved explicitely, by matching a resolution condition (as expressed with `doIf` / `failIf` conditions). If `wait` is set to a positive value, it specifies a deadline before which the Maybe may be (haha!) resolved/rejected; otherwise, it will be automagically resolved when the time is passed.
* **clear** - whether to clear the promise once resolved/rejected (default: `true`). See the note below.

## Custom methods available on the exposed Promise

* **failIf** - specify a condition to reject the action. This matches the flat arguments' flavour of `$.on()` (that is, no map), but *without the handler* (this is jquery.maybe's job). Therefore, just pass an event, and an optionnal element:

``` js
$('.foo').maybe().failIf('my.event', '.my_element');
```

* **doIf** - specify a condition to resolve the action. Same pattern than `failIf`.

Apart from that, this is [a regular, fully-fledged Promise](http://api.jquery.com/category/deferred-object/) as defined by jQuery.

Note that, when initializing the plugin, one may either chain methods:

``` js
$('.my_notification').maybe(// options).done(// cb).fail(// cb);
```

or not:

``` js
$('.my_notification').maybe(// options);
$('.my_notification').done(// callback);
$('.my_notification').fail(// callback);
```

When using the second form, it is important to understand that the exposed promise is memoized by jquery.maybe *until* the action is either resolved or rejected, so in-between calls to `maybe()` on the same element will return the singleton promise, which hopefully leads to the same result as the chaining method.

## On clear

In our example, once resolved or rejected, using legacy Deferred's behaviour, the notification's state would be settled. That is, the exposed promise would be either resolved or rejected, and would remain so for ever. When using jquery.maybe with a non-unique element, such as the `.my_notification` in the previous example (there could be several notifications displayed at once), this could turn really annoying. Therefore, by default, jquery.maybe will clear the "bound" between the element and its settled promise. A new call to `maybe()` on this element will simply generate a new promise, replacing the staled one. If this is not what you want, you may pass `{clear: false}` to leave the promise in peace and stare at it dead until you die too.

## License

MIT, see LICENSE.txt.
