# Jargon SDK for Amazon Alexa (nodejs)

The Jargon SDK makes it easy for skill developers to manage their runtime content, and to support
multiple languages from within their skill.

Need help localizing your skills to new languages and locales? Contact Jargon at localization@jargon.com.

## Requirements

This version of the SDK works with Amazon Alexa skills that are built using the [ASK SDK v2 for Node.js](https://github.com/alexa/alexa-skills-kit-sdk-for-nodejs/tree/2.0.x/ask-sdk).

Like the ASK SDK, the Jargon SDK is built using [TypeScript](https://www.typescriptlang.org/index.html),
and includes typing information in the distribution package.

## Core concepts

### Content resources and resource files
Content resources define the text that your skill outputs to users, via Alexa's voice, card content,
or screen content. It's important that these resources live outside of your skill's source code to
make it possible to localize them into other languages.

The Jargon SDK expects resource files to live in the "resources" subdirectory within your lambda
code (i.e., skill_root/lambda/custom/resources). Each locale has a single resouce file, named for
that locale (e.g., "en-US.json").

Resource files are JSON, with a single top-level object (similar to package.json). The keys within that
object are the identifiers you'll use to refer to specific resources within your source code. Nested objects
are supported to help you organize your resources.
```json
{
  "key1":"Text for key 1",
  "key2":"Text for key 2",
  "nestedObjects":{
    "are":{
      "supported":"Use the key 'nestedObjects.are.supported' to refer to this resource"
    }
  }
}
```

### Resource value format
Resource values are in [ICU MessageFormat](http://userguide.icu-project.org/formatparse/messages). This
format supports constructing text at runtime based on parameters passed in from your code, and selecting
alternative forms to handle things like pluralization and gender.

#### Named parameters
```json
{
  "sayHello":"Hello {name}"
}
```
#### Plural forms
```json
{
  "itemCount":"{count, plural, =0 {You have zero items} =1 {You have one item} other {You have # items}}"
}
```

#### Gendered forms
```json
{
  "pronounSelection":"{gender, select, female {She did it!} male {He did it!} other {It did it!}"
}
```

### Variations
It's important for Alexa skills to vary the words they use in response to users, lest they sound robotic. The Jargon SDK
makes ths simple with built-in variation support. Variations are defined using nested objects:
```json
{
  "resourceWithVariations":{
    "v1":"First variation",
    "v2":"Second variation",
    "v3":"Third variation"
  }
}
```

When rendering the key `resourceWithVariations` the Jargon SDK will choose a variation at random (with other more complex
methods coming in future versions). If you render the same resource multiple times within a single request (e.g., for spoken
content and for card or screen content) the SDK will by default consistently choose the same variation.

Note that you can always select a specific variation using its fully-qualified key (e.g., `resourceWithVariations.v1`)

## Runtime interface

### JargonResponseBuilder
The core class you'll work with. JargonResponseBuilder mirror's the ASK SDK response builder, but changes string
parameters containing content presented to users to RenderItems.

### RenderItem
A RenderItem specifies a resource key, optional parameters, and options to control details of the rendering (which
are themselves optional).
```typescript
interface RenderItem {
  /** The resource key to render */
  key: string
  /** Params to use during rendering (optional) */
  params?: RenderParams
  /** Render options (optional) */
  options?: RenderOptions
}
```

`RenderParams` have your parameter names as keys, and either a string or number for values.

The `ri` helper function simplifies constructing a RenderItem:
```javascript
handlerInput.jrb.speak(ri('sayHello', { 'name': 'World' }))
```

### JargonSkillBuilder
This wraps the ASK skill builder, and handles all details of intializing the Jargon SDK,
installing request and response interceptors, and so on.
```javascript
const skillBuilder = new Jargon.JargonSkillBuilder().wrap(Alexa.SkillBuilders.custom())
```

## Adding to an existing skill

### Installation
First add the Jargon SDK as a dependency of your lambda code (skill_root/lambda/custom)
  * npm i --save @jargon/alexa-skill-sdk
  * yarn add @jargon/alexa-skill-sdk

Next, wrap the Alexa skill builder with Jargon's skill builder:
```javascript
// Import the Jargon SDK
const Jargon = require('@jargon/alexa-skill-sdk')

// Wrap the skill builder
const skillBuilder = new Jargon.JargonSkillBuilder().wrap(Alexa.SkillBuilders.custom())
```

### Externalize resources
The content that your skill outputs via speak(), reprompt(), etc., needs to move from wherever
it currently lives in to Jargon resource files. That's currently a manual step, but in the future
we'll have tools to help automate portions of the process.

Resource files go under skill_root/lambda/custom/resources, and are named by the locale they contain
content for (e.g., "en-US.json").

### Switch over to the Jargon response builder
In your skill handlers access the Jargon reponse builder via one of the following methods:
* `handlerInput.jrb`
* `handlerInput.jargonResponseBuilder`
* `handlerInput.attributesManager.getRequestAttributes().jrb`
* `handlerInput.attributesManager.getRequestAttributes().jargonResponseBuilder`

TypeScript users: you'll need to cast `handlerInput` to `JargonHandlerInput` if you want to use one of
the first two forms.


## Setting up a new skill

We'll soon have templates in place for use with the ASK CLI for bootstrapping new skills with
the Jargon SDK pre-installed, along with skeletons for resource files.