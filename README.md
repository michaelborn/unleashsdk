# Unleash SDK

## Library for interacting with [Unleash](https://www.getunleash.io/) feature flags

### Requirements

Adobe ColdFusion 2018+
Lucee 5+
ColdBox 6+

This module takes heavy advantage of ColdBox's async scheduling. There are no plans to make this module work outside of ColdBox.

### Configuration

Here are the settings for the UnleashSDK:

```cfc
settings = {
    "appName": getApplicationName(), // Defaults to `this.name` in `Application.cfc`.
    "instanceId": resolveHostname(), // Attempts to resolve the hostname. "unknown", otherwise.
    "environment": variables.controller.getSetting( "environment" ),
    "contextProvider": "DefaultContextProvider@unleashsdk",
    "apiURL": getSystemSetting( "UNLEASH_API_URL" ),
    "apiToken": getSystemSetting( "UNLEASH_API_TOKEN" ),
    "refreshInterval": 10,
    "metricsInterval": 60
}
```

Any of these settings can be overridden inside your `config/ColdBox.cfc`.
If you don't need to modify the default settings, you can supply your Unleash
configuration directly through environment variables via `UNLEASH_API_URL` and `UNLEASH_API_TOKEN`.

### Usage

The main usage of the UnleashSDK is checking if a feature flag is enabled. This is done using the `isEnabled` method.
(There is also a helper `isDisabled` method for your convenience.)

#### `isEnabled`

Evaluates the feature with the current and provided context to determine if it is enabled for the current request.

Returns: `boolean`

| Name              | Type    | Required | Default | Description                                                                                                                          |
| ----------------- | ------- | -------- | ------- | ------------------------------------------------------------------------------------------------------------------------------------ |
| name              | string  | true     |         | The name of the feature flag to evaluate                                                                                             |
| additionalContext | struct  | false    | `{}`    | Any additional context items for this check. This struct will override any duplicate keys in the context from the `ContextProvider`. |
| defaultValue      | boolean | false    | `false` | The default value to use if the feature does not exist.                                                                              |

### Context and ContextProviders

Context refers to information about the current request that Unleash will use when
evaluating feature toggles. Context includes the following fields:

```cfc
context = {
    "appName"       : getApplicationName(), // Defaults to `this.name` in `Application.cfc`
    "environment"   : variables.environment, // `controller.getSetting( "environment" )`
    "userId"        : "",
    "sessionId"     : getSessionId(), // `session.sessionid`
    "remoteAddress" : CGI.REMOTE_ADDR,
    "hostname"      : resolveHostname() // Attempts to resolve the hostname. "unknown", otherwise.
};
```

Context is generated using a `ContextProvider` and/or the `context` arguments to `isEnabled`.
The `context` provided to `isEnabled` will overwrite any duplicate keys in the context
generated by the `ContextProvider`.

A `ContextProvider` is a component with `getContext()` method. That method should return
the context fields shown above.

You can register a custom `ContextProvider` by the setting `contextProvider` in your `config/ColdBox.cfc`.
A custom `ContextProvider` can enabled you to always provide specific user or session information
for your application without having to pass it in manually each time you call `isEnabled`.
