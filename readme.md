## Duolog Chatbot

|                |                                                         |
| :------------- | :------------------------------------------------------ |
| Owner          | Doug Maloney, Duolog                                    |
| Developer      | [Christopher Barnaby](http://www.cjbarnaby.com.au)      |
| Date           | 17 August 2017                                          |
| Project type   | Proof of Concept                                        |
| Version        | 0.2.0                                                   |
___

### Overview

The chatbot is a static web app that connects a web interface to API.AI chatbot agent/s using the API.AI JavaScript SDK. The chatbot is a standalone, static app, intended to be embedded in Duolog and/or Duolog client-hosted websites via iframe.

### Dependencies

- [jQuery v 3.2.1](http://jquery.com/download/)
- [API.AI JavaScript SDK](https://github.com/api-ai/apiai-javascript-client)
- [Google Material Icons](https://material.io/icons/)

A note regarding the API.AI JavaScript SDK – This library is sufficient for the current implementation but doesn't really bring much to the table, aside from frustratingly obfuscating the process of making calls to the API and not being particular well documented. Handling AJAX requests with bare XMLHttpRequests or jQuery .ajax() functions may indeed be preferable if the application is further modified.

### Usage

The application is designed to take text input from the user via the text input field, send requests to the API.AI chatbot agent matching the access token configured in the app.js file, and handle responses returned by the agent, displaying them in the chatbot window, and allowing further input from the user.

#### Welcome message

Once the document has been loaded by the browser, an event request is sent to API.AI. The response is handled generically, meaning the response be of any valid type handled by the application (text, or custom payload (rich text, card, quick reply)) configured in the API.AI console.

#### Error message

If the ajax request to API.AI resolves to an error, an error message will be temporarily displayed in the chatbot window.

#### Loading

Once any request is sent, a loading animation is displayed in the chatbot window and will remain so until a response is received from the API or the call resolves in an error.

### API.AI JSON payloads

The application is designed to handle the following responses from API.AI:

1. Standard **Text responses** configured in API.AI intents.
2. **Custom JSON payloads** configured in API.AI intents.
3. Contexts configured in any accepted response type.

The application expects JSON returned in custom payloads to meet the following specification:

#### Rich Text

- **type:** 10
- **text:**
  - No character limit.
  - All UTF-8 characters, including emojis accepted.
  - Line-breaks can be created using `\n` and will be parsed as expected.

```JSON
{
  "type": 10,
  "text": "Chatbot text response\nwith line break 😀"
}
```

#### Quick Reply

- **type:** 12
- **title:**
  - No character limit.
  - All UTF-8 characters, including emojis accepted.
  - Line-breaks can be created using `\n` and will be parsed as expected.

- **replies:**
  - 12 character limit per reply.
  - No minimum or maximum number of replies per Quick Reply
  - All UTF-8 characters, including emojis, are accepted.

```JSON
{
  "type": 12,
  "title": "What is your quick reply ?",
  "replies": [
    "Reply 1",
    "Reply 2",
    "Reply 3"
  ]
}
```

#### Card

- **type:** 11
- **imageurl**: Any valid URL for an image on the web.
- **title:** Title will be trimmed when it reaches width of element, truncated by ellipsis (`...`).
- **subtitle:** Subtitles will be trimmed at 80 characters, truncated by ellipsis (`...`).
- **buttons — labels:**
  - No character limit, but recommend < 45 characters for optimal presentation.
  - No minimum or maximum number of buttons per card.
  - All UTF-8 characters, including emojis, are accepted.
- **buttons — web_url:**
  - Including this key in a button will cause the URL value assigned to the key to be opened in a new browser tab when clicked.
  - Can be any valid URL.
- **pbuttons — postback:**
  - Including this key in a button will cause the value assigned to the key to be sent back to the API.AI agent in a new request.

```JSON
{
  "type": 11,
  "imageurl": "http://placehold.it/400x210",
  "title": "This is the title. It should be trimmed at the end of one line. If you're reading this, it wasn't trimmed for some reason.",
  "subtitle": "This is the subheading. It should be trimmed after 80 characters. If you're reading this, it wasn't trimmed for some reason.",
  "buttons": [
    {
      "label": "Button 1",
      "web_url": "http://www.google.com"
    },
    {
      "label": "Button 2",
      "postback": "some text"
    }
  ]
}
```

#### Video

- **type:** 14
- **videourl:**
  - Currently limited to YouTube video URLs.
  - URL must be the '/embed/' URL format.

```JSON
{
  "type": 14,
  "videourl": "https://www.youtube.com/embed/UTfJDg_D8UM"
}
```

#### Contexts

The application will respond to contexts configured for an agent intent in the API.AI console. These should not be added as custom JSON payload data, but configured manually in the console. The application will include any received context in subsequent api calls, deferring to API.AI agent logic for context expiry (default: 5 requests or 10 minutes after context set).

### Implementation

The application has been deployed to [BitBalloon](https://www.bitballoon.com/), a free static website hosting platform.

- [BitBalloon chatbot admin portal](https://www.bitballoon.com/sites/duolog-webdemo)
- [Live application](http://duolog-webdemo.bitballoon.com/)

The application (as deployed on BitBalloon) can be embedded in another website via HTML `<iframe>` without causing cross origin resource sharing errors. Using a CMS system such as SquareSpace, WordPress, etc, a custom page should be built with the HTML set out in the [chatbot container demo repo](https://github.com/cjbarnaby/chatbot_container). Contact the developer for customization as necessary.

For a successful implementation example, see:

- [Github repository](https://github.com/cjbarnaby/chatbot_container)
- [Live site with chatbot embedded via iframe](http://cjbarnaby.com.au/chatbot_container/)

### Configuration

(Note: The end-to-end deployment process is described in 'Deployment' below).

The app has been designed to allow minor changes to be made via text editor by persons with little/no web development experience. The following application properties can be configured by modifying the `chatbot` object in the [](./js/app.js) file:

- **Header image:** Replace URL in the `chatbot.headerImage` property with the URL of any image on the web. While the image will entirely cover the relevant element in the UI (without distortion) regardless of the size and aspect ratio of the source image, it is recommended that source images as close to 500 x 157 pixels as possible are used to ensure optimal appearance with minimal performance impact (eg, due to unnecessarily large file sizes).
- **Chatbot agent:** Replace the token in the `chatbot.accessToken` property. The access token for the agent intended to be connected to the application can be selected in the API.AI console be selecting the agent, then clicking the settings cog next to the agent name and copying the `client access token` to the clipboard. Note: ensure that the client access token is copied and *not* the developer access token.
- **Chatbot background color:** Replace the color in the `chatbot.backgroundColor` property. This can be any color in valid CSS color format (eg, any of `#FFFFFF` or `rgb(0,0,0)` or `rgba(0,0,0,1)` or `white` will work to make the background color white). As of 0.2.0, this will also change the color behind the banner text in the banner.
- **Banner text:** Replace the text in the `chatbot.headerText` property.

For all the changes above, ensure that the value given to the property is enclosed in quote marks (either single (`'`)or double quote (`"`) marks), and ensure that there is a comma after the closing quote mark. Example:

```js

var chatbot = {
  headerImage: "http://i.imgur.com/NggwaAk.png",
  headerText: "Duolog Chatbot",
  backgroundColor: "#FFFFFF",
  accessToken: "b745f3f6e65b458e895add17566b55dc",
  enterKeyCode: 13,
  loading: false,
  error: false
  //etc
}
```
Once the necessary changes have been made, the entire project folder (comprising the index.html, the js folder and all files therein and the CSS folder and all files therein) should be reloaded to BitBalloon. The full, end-to-end process is described below in 'Deployment'.

### Deployment

NOTE: It is recommended that any changes aside from those listed above in 'Configuration' be made by a web developer, or someone who otherwise has experience in deploying code to GitHub. The process set out below is intended to allow minor configuration changes to the abovementioned properties **only** and is by no stretch of the imagination good practice for managing code changes.

1. **Download the entire application code** from the GitHub.

  Go to the [GitHub repository](https://github.com/cjbarnaby/duolog_chatbot) for the application, click 'clone or download', then select 'download zip'.

2. **Make the necessary changes to the relevant files**.

  See 'Configuration', above. It is recommended that a text editor such as [Atom](https://atom.io/) or [Sublime Text](https://www.sublimetext.com/) be used to make changes.

3. **Redeploy the application to BitBalloon**.

  Go to [BitBalloon](https://www.bitballoon.com/sites/duolog-webdemo) and — _in one action_  — drag and drop the entire updated project folder (the **index.html**, the **js** folder and all files therein and the **css** folder and all files therein) to the area that says "Drag & Drop to Update your Site".

The changes should immediately be reflected in the live application (http://duolog-webdemo.bitballoon.com/) and all pages on which the application has been embedded via iframe.

___

### Change Log

| Version | Date             | Changes |
| :------ | :--------------- | :------------- |
| 0.1.0   | 21 August 2017   | Application now distinguishes between — and disregards — messages intended for other platforms, and handles multiple messages per response. |
| 0.2.0   | 6 September 2017 | YouTube video support; configurable bot title; animated responses; compact UI redesign; flex layout; persistent client/session per session. |
