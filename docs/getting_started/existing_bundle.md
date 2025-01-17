# Migrating your existing bundle to nodecg-io

This guide explains you how to migrate an existing nodecg bundle that already meets your requirements to use nodecg-io.
nodecg-io manages service configuration and authentication so your bundle doesn't have to bother with it.

If you are lucky and used the same library as we do for the service you want to migrate, nodecg-io is a drop-in replacement just doing authentication and configuration.
If you have used another library you'll have to do a little more work to migrate to the library that our service uses.
In that case make sure that the migration is worth the effort.
You can use nodecg-io for one service and use your own library of choice for other services if you like to.

## Install `nodecg-io-core`

The first thing you need to do is to install `nodecg-io-core`. You can do this simply using this npm command:

```shell
npm install nodecg-io-core
```

## Import `requireService`

Now that you have `nodecg-io-core` installed in your bundle you can import the `requireService` function that it exposes:

=== "TypeScript/ES Modules"

    ```typescript
    import { requireService } from "nodecg-io-core";
    ```

=== "CommonJS"

    ```javascript
    const requireService = require("nodecg-io-core").requireService;
    ```

## Import Service type (TypeScript only)

If you're using TypeScript you'll also need to install the package of the service that you want to use.
You need it to be able to import its type for typesafety.

This example uses `twitch-chat` as an example but you can just replace it with the name of the service you need.

=== "TypeScript"

    ```shell
    npm install nodecg-io-twitch-chat
    ```

    ```typescript
    import { TwitchChatServiceClient } from "nodecg-io-twitch-chat";
    ```

Note: You can get the name of the type by using autocomplete in your editor or by looking it up in the service sample.

## Require the service

Now you can finally tell nodecg-io that you want to get access to a service using the `requireService` function.
You'll have to pass your nodecg instance which is used to identify your bundle and the name of the service you want.
In case of TypeScript you'll also need to provide the type of the service.

=== "TypeScript"

    ```typescript
    const twitchChat = requireService<TwitchChatServiceClient>(nodecg, "twitch-chat");
    ```

=== "JavaScript"

    ```javascript
    const twitchChat = requireService(nodecg, "twitch-chat");
    ```

## Accessing the service client

### Using `onAvailable` and `onUnavailable` handlers

You can setup handlers that are executed when the user assigns a service instance to your bundle, removes the assignment or when the service client got updated in some way.

Handlers added with `onAvailable` will get called if there was a change and you got a client:

```typescript
// This is the variable with the return value of requireService().
// You may want to change the variable name for the service you are using.
twitchChat.onAvailable(client => {
    nodecg.log.info("Client was set");
    // You can use the passed client in here to e.g. setup handlers on the client
});
```

`onAvailable` is especially useful to add event handlers to clients.
E.g. if you want to react to donations or chat messages you can add event handlers for these here.

Handlers added with `onUnavailable` will get called if your bundle was not assigned a service instance or if there was an error in service client creation:

```typescript
twitchChat.onUnavailable(() => {
    nodecg.log.info("Client was unset");
});
```

### Using `getClient`

Instead of callbacks you can also get access to the client directly:

```typescript
const twitchChatClient = twitchChat.getClient();
```

`getClient` will return the client if your bundle has an assigned service instance that has produced an service client without error
and will return `undefined` otherwise.
This is useful for when you want to use the client due to some external event or from `onAvailable` handlers of other services.
