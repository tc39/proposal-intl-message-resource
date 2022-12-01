# Intl.MessageFormat.parseResource()

This is a follow-on proposal to the [Intl.MessageFormat proposal](https://github.com/tc39/proposal-intl-messageformat),
adding support for handling entire resources (aka bundles) of messages,
in addition to supporting individual messages.

## Status

Champion: Eemeli Aro (Mozilla/OpenJS Foundation)

### Stage: 1

## Motivation and Use Cases

In most use cases, systems that have a need to format messages need to format more than one such message.
For example, a dialog box may include a title, description, and one or more buttons with labels.
These messages need to be handled together as a cohesive unit,
both when editing or translating, as well as when formatting.

To enable this, a message resource syntax is being developed in parallel with the MessageFormat 2.0 specification.
This proposal adds a static method `Intl.MessageFormat.parseResource()` that allows for parsing such resources.
Such a method would allow for MF2 messages to be stored and transmitted in a purpose-built container,
rather than needing to be separately parsed for use in JavaScript environments.

## API Description

As a baseline, this proposal presumes the existence of `Intl.MessageFormat` as described in its proposal,
and extends it.

### MessageFormat.parseResource(resource, locales?, options?)

This static method parses a string representation of an MF2 resource,
constructing a Map with a corresponding structure of `MessageFormat` instances for each of the resource's messages.
Its `locales` and `options` arguments are used to construct each such instance.

A `MessageResource` is a Map representing a collection of related messages for a single locale.
Messages can be organized in a flat structure, or in hierarchy, using paths.
Conceptually, it is similar to a file containing a set of messages,
but there are no constrains implied on the underlying implementation.

```ts
type MessageResource = Map<string, Intl.MessageFormat | MessageResource>;

class Intl.MessageFormat {
  static parseResource(
    resource: string,
    locales?: string | string[],
    options?: MessageFormatOptions
  ): MessageResource;

  ...
}
```

## Example

Given an MF2 resource as follows:

```ini
# Note! MF2 syntax is under development; this may still change

greeting = {Hello {$place}!}

new_notifications =
  match {$count}
  when 0   {You have no new notifications}
  when one {You have {$count} new notification}
  when *   {You have {$count} new notifications}
```

This could be used in code like this:

```js
const source = ... // string source of resource as above
const res = Intl.MessageFormat.parseResource(source, ['en']);

const greeting = res.get('greeting').resolveMessage({ place: 'world' });
greeting.toString(); // 'Hello world!'

const notifications = res.get('new_notifications').resolveMessage({ count: 1 });
notifications.toString(); // 'You have 1 new notification'
```
