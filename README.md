![A banner image of a bunch of stylized animal bots](header.png)

# build-a-bot

A production-grade framework for building AI agents.

## Getting Started

This repo includes a simple example agent in `example/` which can be run using `ts-node`:

`$ npx ts-node example/cli.ts`

You will need to set an `OPENAI_API_KEY` environment variable first to run it. By default it uses GPT-4, but this is easy to change in [`example/agent.ts`](example/agent.ts).

Poking around the example agent, a derivative of the Chimaera agent we use in our internal tools at [Science](https://science.xyz), should give you a feel for how the framework works. In particular, our goals were:

- strongly typed
- fully asychronous
- comprehensive handling of hooks; a compact but robust codebase
- easy to prototype complex agent patterns

Creating an agent requires implementing four methods and an optional regex per the `Agent` interface defined in [`src/agent/index.ts`](src/agent/index.ts):

```
  abstract basePrompt: () => string;
  abstract detectPluginUse: (response: string) => false | PluginInvocation;
  abstract handlePluginOutput: (input: PluginInvocation, output: PluginOutput) => void;
  pluginDetectRegex: RegExp | null = null
  abstract run(prompt: string, role?: MessageRoles): void;
```

This allows you to flexibly define a syntax for your models to invoke plugins, handle their output and emit appropriate events, all within the context of your core agent loop.

For an example of a non-trivial core agent loop, look at the [**Daydreaming** example](example/daydreamer/index.ts). The daydreaming idea involves two agents with slightly different prompts (one biased towards idea generation and one biased towards idea filtering) and interconnects them in a converser pattern.

Daydreaming is also provided as a plugin within the framework itself in [`src/plugins/daydream.ts`](src/plugins/daydream.ts), and with smart enough prompting can be invoked automatically when the agent feels like it would benefit from some self-reflection to generate better answers.

## Basic Architecture

Build-a-bot is oriented around four basic components:

- One or more _models_, which make API calls out to LLMs. Models never run in-process; they are always called over the network.
- Zero or more _plugins_, which can be used by agents to access external tools and resources.
- One or more _agents_, which compose models, prompts and plugins into meaningful behavior.
- One or more _interfaces_, which expose functionality of your agent(s) to users, such as via a websocket, Slack bot, over email or so on.

A typical implementation will expose one agent built on a range of models and prompts and able to use a range of plugins via a websocket.

### Plugins

Build-a-bot plugins can be either implemented specifically using Typescript (for example, the [Browser](src/plugins/browser/index.ts), [Daydream](src/plugins/daydream.ts), or [Shell](src/plugins/shell.ts) plugins), or an [OpenAPI-based plugin](src/plugins/openapi.ts) can be loaded from an AI Plugin JSON spec.

## Known Issues

- Observations in the Browser plugin are frequently far too large to fit in model context, and splitting is not currently implemented.
- The Shell plugin is currently a stub.
