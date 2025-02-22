<!-- markdownlint-disable MD033 -->
<!-- x-hide-in-docs-start -->
<p align="center">
  <picture>
    <source media="(prefers-color-scheme: dark)" srcset="https://raw.githubusercontent.com/open-feature/community/0e23508c163a6a1ac8c0ced3e4bd78faafe627c7/assets/logo/horizontal/white/openfeature-horizontal-white.svg" />
    <img align="center" alt="OpenFeature Logo" src="https://raw.githubusercontent.com/open-feature/community/0e23508c163a6a1ac8c0ced3e4bd78faafe627c7/assets/logo/horizontal/black/openfeature-horizontal-black.svg" />
  </picture>
</p>

<h2 align="center">OpenFeature Web SDK</h2>

<!-- x-hide-in-docs-end -->
<!-- The 'github-badges' class is used in the docs -->
<p align="center" class="github-badges">
  <a href="https://github.com/open-feature/spec/tree/v0.7.0">
    <img alt="Specification" src="https://img.shields.io/static/v1?label=specification&message=v0.7.0&color=yellow&style=for-the-badge" />
  </a>
  <!-- x-release-please-start-version -->
  <a href="https://github.com/open-feature/js-sdk/releases/tag/web-sdk-v0.4.1">
    <img alt="Release" src="https://img.shields.io/static/v1?label=release&message=v0.4.1&color=blue&style=for-the-badge" />
  </a>
  <!-- x-release-please-end -->
  <br/>
  <a href="https://www.repostatus.org/#wip">
    <img alt="Project Status" src="https://www.repostatus.org/badges/latest/wip.svg" />
  </a>
  <a href="https://open-feature.github.io/js-sdk/modules/OpenFeature_Web_SDK.html">
    <img alt="API Reference" src="https://img.shields.io/badge/reference-teal?logo=javascript&logoColor=white" />
  </a>
  <a href="https://www.npmjs.com/package/@openfeature/web-sdk">
    <img alt="NPM Download" src="https://img.shields.io/npm/dm/%40openfeature%2Fweb-sdk" />
  </a>
  <a href="https://codecov.io/gh/open-feature/js-sdk">
    <img alt="codecov" src="https://codecov.io/gh/open-feature/js-sdk/branch/main/graph/badge.svg?token=3DC5XOEHMY" />
  </a>
    <a href="https://bestpractices.coreinfrastructure.org/projects/6594">
    <img alt="CII Best Practices" src="https://bestpractices.coreinfrastructure.org/projects/6594/badge" />
  </a>
</p>
<!-- x-hide-in-docs-start -->

[OpenFeature](https://openfeature.dev) is an open standard that provides a vendor-agnostic, community-driven API for feature flagging that works with your favorite feature flag management tool.

<!-- x-hide-in-docs-end -->

## 🚀 Quick start

### Requirements

- ES2015-compatible web browser (Chrome, Edge, Firefox, etc)

### Install

#### npm

```sh
npm install --save @openfeature/web-sdk
```

#### yarn

```sh
yarn add @openfeature/web-sdk
```

### Usage

```ts
import { OpenFeature } from '@openfeature/web-sdk';

// Register your feature flag provider
OpenFeature.setProvider(new YourProviderOfChoice());

// create a new client
const client = OpenFeature.getClient();

// Evaluate your feature flag
const v2Enabled = client.getBooleanValue('v2_enabled', false);

if (v2Enabled) {
  console.log("v2 is enabled");
}
```

### API Reference

See [here](https://open-feature.github.io/js-sdk/modules/OpenFeature_Web_SDK.html) for the complete API documentation.

## 🌟 Features

| Status | Features                        | Description                                                                                                                        |
| ------ | ------------------------------- | ---------------------------------------------------------------------------------------------------------------------------------- |
| ✅      | [Providers](#providers)         | Integrate with a commercial, open source, or in-house feature management tool.                                                     |
| ✅      | [Targeting](#targeting-and-context)         | Contextually-aware flag evaluation using [evaluation context](https://openfeature.dev/docs/reference/concepts/evaluation-context). |
| ✅      | [Hooks](#hooks)                 | Add functionality to various stages of the flag evaluation life-cycle.                                                             |
| ✅      | [Logging](#logging)             | Integrate with popular logging packages.                                                                                           |
| ✅      | [Named clients](#named-clients) | Utilize multiple providers in a single application.                                                                                |
| ✅      | [Eventing](#eventing)           | React to state changes in the provider or flag management system.                                                                  |
| ✅      | [Shutdown](#shutdown)           | Gracefully clean up a provider during application shutdown.                                                                        |
| ✅      | [Extending](#extending)         | Extend OpenFeature with custom providers and hooks.                                                                                |

<sub>Implemented: ✅ | In-progress: ⚠️ | Not implemented yet: ❌</sub>

### Providers

[Providers](https://openfeature.dev/docs/reference/concepts/provider) are an abstraction between a flag management system and the OpenFeature SDK.
Look [here](https://openfeature.dev/ecosystem/?instant_search%5BrefinementList%5D%5Btype%5D%5B0%5D=Provider&instant_search%5BrefinementList%5D%5Bcategory%5D%5B0%5D=Client-side&instant_search%5BrefinementList%5D%5Btechnology%5D%5B0%5D=JavaScript) for a complete list of available providers.
If the provider you're looking for hasn't been created yet, see the [develop a provider](#develop-a-provider) section to learn how to build it yourself.

Once you've added a provider as a dependency, it can be registered with OpenFeature like this:

```ts
OpenFeature.setProvider(new MyProvider())
```

In some situations, it may be beneficial to register multiple providers in the same application.
This is possible using [named clients](#named-clients), which is covered in more detail below.

### Flag evaluation flow

When a new provider is added to OpenFeature client the following process happens:

```mermaid
sequenceDiagram
    autonumber
    Client-->+Feature Flag Provider: ResolveAll (context)
    Feature Flag Provider-->-Client: Flags values
```

In (1) the Client sends a request to the provider backend in order to get all values from all feature flags that it has.
Once the provider backend replies (2) the client holds all flag values and therefore the flag evaluation process is synchronous.

In order to prevent flag evaluation to the default value while flags are still being fetched, it is highly recommended to only look for flag value after the provider has emitted the `Ready` event.
The following code snippet provides an example.

```ts
import { OpenFeature, ProviderEvents } from '@openfeature/web-sdk';

OpenFeature.setProvider( /*set a provider*/ );

// OpenFeature API
OpenFeature.addHandler(ProviderEvents.Ready, () => {
  const client = OpenFeature.getClient();
  const stringFlag = client.getStringValue('string-flag', "default value"))

  //use stringFlag from this point
});
```

### Targeting and Context

Sometimes, the value of a flag must consider some dynamic criteria about the application or user, such as the user's location, IP, email address, or the server's location.
In OpenFeature, we refer to this as [targeting](https://openfeature.dev/specification/glossary#targeting).
If the flag management system you're using supports targeting, you can provide the input data using the [evaluation context](https://openfeature.dev/docs/reference/concepts/evaluation-context).

```ts
// Set a value to the global context
await OpenFeature.setContext({ origin: document.location.host });
```

Context is global and setting it is `async`.
Providers may implement an `onContextChanged` method that receives the old context and the newer one.
This method is used internally by the provider to detect if, given the context change, the flags values cached on client side are invalid. If needed a request will be made to the provider with the new context in order to get the correct flags values.

### Hooks

[Hooks](https://openfeature.dev/docs/reference/concepts/hooks) allow for custom logic to be added at well-defined points of the flag evaluation life-cycle
Look [here](https://openfeature.dev/ecosystem/?instant_search%5BrefinementList%5D%5Btype%5D%5B0%5D=Hook&instant_search%5BrefinementList%5D%5Bcategory%5D%5B0%5D=Client-side&instant_search%5BrefinementList%5D%5Btechnology%5D%5B0%5D=JavaScript) for a complete list of available hooks.
If the hook you're looking for hasn't been created yet, see the [develop a hook](#develop-a-hook) section to learn how to build it yourself.

Once you've added a hook as a dependency, it can be registered at the global, client, or flag invocation level.

```ts
import { OpenFeature } from "@openfeature/web-sdk";

// add a hook globally, to run on all evaluations
OpenFeature.addHooks(new ExampleGlobalHook());

// add a hook on this client, to run on all evaluations made by this client
const client = OpenFeature.getClient();
client.addHooks(new ExampleClientHook());

// add a hook for this evaluation only
const boolValue = client.getBooleanValue("bool-flag", false, { hooks: [new ExampleHook()]});
```

### Logging

The JS SDK will log warnings and errors to the console by default.
This behavior can be overridden by passing a custom logger either globally or per client.
A custom logger must implement the [Logger interface](../shared/src/logger/logger.ts).

```ts
import type { Logger } from "@openfeature/web-sdk";

// The logger can be anything that conforms with the Logger interface
const logger: Logger = console;

// Sets a global logger
OpenFeature.setLogger(logger);

// Sets a client logger
const client = OpenFeature.getClient();
client.setLogger(logger);
```

### Named clients

Clients can be given a name.
A name is a logical identifier that can be used to associate clients with a particular provider.
If a name has no associated provider, the global provider is used.

```ts
import { OpenFeature } from "@openfeature/web-sdk";

// Registering the default provider
OpenFeature.setProvider(NewLocalProvider());
// Registering a named provider
OpenFeature.setProvider("clientForCache", new NewCachedProvider());

// A Client backed by default provider
const clientWithDefault = OpenFeature.getClient();
// A Client backed by NewCachedProvider
const clientForCache = OpenFeature.getClient("clientForCache");
```

### Eventing

Events allow you to react to state changes in the provider or underlying flag management system, such as flag definition changes, provider readiness, or error conditions.
Initialization events (`PROVIDER_READY` on success, `PROVIDER_ERROR` on failure) are dispatched for every provider.
Some providers support additional events, such as `PROVIDER_CONFIGURATION_CHANGED`.

Please refer to the documentation of the provider you're using to see what events are supported.

```ts
import { OpenFeature, ProviderEvents } from '@openfeature/web-sdk';

// OpenFeature API
OpenFeature.addHandler(ProviderEvents.Ready, (eventDetails) => {
  console.log(`Ready event from: ${eventDetails?.clientName}:`, eventDetails);
});

// Specific client
const client = OpenFeature.getClient();
client.addHandler(ProviderEvents.Error, (eventDetails) => {
  console.log(`Error event from: ${eventDetails?.clientName}:`, eventDetails);
});
```

### Shutdown

The OpenFeature API provides a close function to perform a cleanup of all registered providers.
This should only be called when your application is in the process of shutting down.

```ts
import { OpenFeature } from '@openfeature/web-sdk';

await OpenFeature.close()
```

## Extending

### Develop a provider

To develop a provider, you need to create a new project and include the OpenFeature SDK as a dependency.
This can be a new repository or included in [the existing contrib repository](https://github.com/open-feature/js-sdk-contrib) available under the OpenFeature organization.
You’ll then need to write the provider by implementing the [Provider interface](./src/provider/provider.ts) exported by the OpenFeature SDK.

```ts
import { JsonValue, Provider, ResolutionDetails } from '@openfeature/web-sdk';

// implement the provider interface
class MyProvider implements Provider {

  readonly metadata = {
    name: 'My Provider',
  } as const;

  // Optional provider managed hooks
  hooks?: Hook<FlagValue>[];

  resolveBooleanEvaluation(flagKey: string, defaultValue: boolean, context: EvaluationContext, logger: Logger): ResolutionDetails<boolean> {
    // code to evaluate a boolean
  }

  resolveStringEvaluation(flagKey: string, defaultValue: string, context: EvaluationContext, logger: Logger): ResolutionDetails<string> {
    // code to evaluate a string
  }

  resolveNumberEvaluation(flagKey: string, defaultValue: number, context: EvaluationContext, logger: Logger): ResolutionDetails<number> {
    // code to evaluate a number
  }

  resolveObjectEvaluation<T extends JsonValue>(flagKey: string, defaultValue: T, context: EvaluationContext, logger: Logger): ResolutionDetails<T> {
    // code to evaluate an object
  }

  status?: ProviderStatus | undefined;
  events?: OpenFeatureEventEmitter | undefined;

  initialize?(context?: EvaluationContext | undefined): Promise<void> {
    // code to initialize your provider
  }

  onClose?(): Promise<void> {
    // code to shut down your provider
  }
}
```

> Built a new provider? [Let us know](https://github.com/open-feature/openfeature.dev/issues/new?assignees=&labels=provider&projects=&template=document-provider.yaml&title=%5BProvider%5D%3A+) so we can add it to the docs!

### Develop a hook

To develop a hook, you need to create a new project and include the OpenFeature SDK as a dependency.
This can be a new repository or included in [the existing contrib repository](https://github.com/open-feature/js-sdk-contrib) available under the OpenFeature organization.
Implement your own hook by conforming to the [Hook interface](../shared/src/hooks/hook.ts).

```ts
import type { Hook, HookContext, EvaluationDetails, FlagValue } from "@openfeature/web-sdk";

export class MyHook implements Hook {
  after(hookContext: HookContext, evaluationDetails: EvaluationDetails<FlagValue>) {
    // code that runs when there's an error during a flag evaluation
  }
}
```

> Built a new hook? [Let us know](https://github.com/open-feature/openfeature.dev/issues/new?assignees=&labels=hook&projects=&template=document-hook.yaml&title=%5BHook%5D%3A+) so we can add it to the docs!
