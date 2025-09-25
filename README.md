# AlertWise Web SDK

[![NPM version](https://img.shields.io/npm/v/@alertwise/alertwise-web-sdk.svg)](https://www.npmjs.com/package/@alertwise/alertwise-web-sdk)
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)

The official AlertWise Web SDK for integrating web push notifications into your website.

[AlertWise](https://alertwise.net) is a powerful push notification service for web and mobile apps. This SDK makes it
incredibly easy to integrate your website with AlertWise, enabling you to engage with your users through timely and
relevant push notifications.

## Features

- **Easy Integration**: Get up and running in minutes with a simple initialization script.
- **Customizable Prompts**: Choose from a variety of pre-built, customizable opt-in prompts (Banner, Bell, Pop-up, etc.)
  to match your site's design.
- **Subscription Widgets**: Add persistent widgets to allow users to subscribe or unsubscribe at any time.
- **Audience Segmentation**: Group subscribers into segments for targeted messaging.
- **Event Tracking**: Track user interactions and trigger automated notification campaigns.
- **Cross-Browser Support**: Works seamlessly across all major browsers that support web push notifications.
- **In-App Browser Handling**: Gracefully handles scenarios where your site is opened in a social media in-app browser.

## Installation

You can integrate the AlertWise Web SDK in two ways: using the CDN or by installing it as an npm package.

### 1. Using CDN

The easiest way to get started is to add the following script tag to the `<head>` section of your website's HTML file.
[Alertwise CDN Link](https://cdn.alertwise.net/alertwise/integrate/service-worker.js)

```html

<script src="https://cdn.alertwise.net/alertwise/integrate/service-worker.js" async></script>
```

### 2. Using npm/yarn

If you are using a bundler like Webpack or Rollup, you can install the SDK from npm.

```bash
  npm install @alertwise/alertwise-web-sdk
```

or

```bash
  yarn add @alertwise/alertwise-web-sdk
```

Then, import it into your main application file:

```javascript
import '@alertwise/alertwise-web-sdk';
```

## Quick Start

After installing the SDK, you need to initialize it with your Application ID.

### 1. Add the Service Worker

Web push notifications require a service worker file.

1. Download the `service-worker.js` file from the SDK. You can get it from the CDN
   link: https://cdn.alertwise.net/alertwise/integrate/service-worker.js.
2. Place this `service-worker.js` file in the **root directory** of your website.

If you cannot place the file in the root directory, you can specify a different path during initialization.

### 2. Initialize the SDK

Add the following initialization code to your website. If you're using the CDN, place it just before the closing
`</body>` tag.

```html

<script>
    // This array will hold commands until the SDK is fully loaded.
    window.alertwise = window.alertwise || [];

    // Initialize AlertWise with your App ID
    alertwise.push(['init', {
        appId: 'YOUR_APP_ID'
        // Add other configuration options here if needed
    }]);
</script>
```

Replace `'YOUR_APP_ID'` with the Application ID you obtained from your AlertWise dashboard.

That's it! The SDK will now automatically handle showing the opt-in prompt based on the settings you've configured in
your AlertWise dashboard.

## Configuration

The `init` command accepts a configuration object with the following options:

| Option                        | Type      | Required | Description                                                                                      |
|-------------------------------|-----------|----------|--------------------------------------------------------------------------------------------------|
| `appId`                       | `string`  | **Yes**  | Your unique Application ID from the AlertWise dashboard.                                         |
| `serviceWorkerUrl`            | `string`  | No       | The path to your service worker file. Defaults to `/service-worker.js`.                          |
| `scope`                       | `string`  | No       | The scope of the service worker registration. Defaults to `/`.                                   |
| `isSubscriptionOnSubDomain`   | `boolean` | No       | Set to `true` if you are using a custom subdomain for subscriptions. Defaults to `false`.        |
| `isUniversalSubscriptionLink` | `boolean` | No       | Set to `true` if you are using a universal subscription link. Defaults to `false`.               |
| `language`                    | `string`  | No       | Manually set the language for the prompts, overriding the browser's default. (e.g., 'en', 'es'). |
| `defaultNotificationImage`    | `string`  | No       | A URL for a default notification icon if one is not provided in a campaign.                      |
| `alertwiseApiUrl`             | `string`  | No       | For self-hosted or enterprise plans, you can specify a custom API endpoint.                      |

**Example with more options:**

```javascript
alertwise.push(['init', {
    appId: 'YOUR_APP_ID',
    serviceWorkerUrl: '/custom-sw.js'
}]);
```

## API Reference

Once initialized, you can interact with the SDK using the `alertwise` global object.

### `alertwise.push(command)`

This is the main method for interacting with the SDK.

- **Initialization**: `alertwise.push(['init', { appId: '...' }])`
- **Function Execution**: You can push a function to be executed once the SDK is ready.

```javascript
alertwise.push(function () {
    console.log('AlertWise SDK is ready!');
    // You can now safely use other API methods
    alertwise.getSubscriberId().then(id => {
        console.log('Subscriber ID:', id);
    });
});
```

### `alertwise.subscribe()`

Manually trigger the subscription process. This will show the browser's native permission prompt.

```javascript
// Example: Trigger subscription from a custom button click
document.getElementById('my-subscribe-button').addEventListener('click', function () {
    alertwise.subscribe();
});
```

### `alertwise.unsubscribe()`

Unsubscribe the current device from receiving push notifications.

```javascript
alertwise.unsubscribe();
```

### `alertwise.getPermission()`

Returns a `Promise<NotificationPermission>` with the current notification permission status (`'granted'`, `'denied'`, or
`'default'`).

```javascript
alertwise.getPermission().then(permission => {
    console.log('Current permission status:', permission);
});
```

### `alertwise.getSubscriberId()`

Returns a `Promise<string | undefined>` with the unique ID of the current subscriber.

```javascript
alertwise.getSubscriberId().then(subscriberId => {
    if (subscriberId) {
        console.log('User is subscribed with ID:', subscriberId);
    } else {
        console.log('User is not subscribed.');
    }
});
```

### `alertwise.addSegment(segmentId)`

Add the current subscriber to one or more segments.

- `segmentId`: `string | string[]` - A single segment ID or an array of segment IDs.

```javascript
// Add to a single segment
alertwise.addSegment('premium_users');

// Add to multiple segments
alertwise.addSegment(['newsletter_subscribers', 'product_updates']);
```

### `alertwise.removeSegment(segmentId)`

Remove the current subscriber from one or more segments.

- `segmentId`: `string | string[]` - A single segment ID or an array of segment IDs.

```javascript
alertwise.removeSegment('premium_users');
```

## Events

The SDK dispatches `CustomEvent` objects on the `window` for various actions. You can listen for these events to trigger
your own application logic.

| Event Name                          | Description                                                                                                         |
|-------------------------------------|---------------------------------------------------------------------------------------------------------------------|
| `AlertWise.onReady`                 | Fired when the SDK has been successfully initialized.                                                               |
| `AlertWise.onSubscribe`             | Fired when a user successfully subscribes. The `event.detail` object contains `subscriber` and `subscription` info. |
| `AlertWise.onUnsubscribe`           | Fired when a user successfully unsubscribes.                                                                        |
| `AlertWise.onPermissionGranted`     | Fired when the user grants notification permission.                                                                 |
| `AlertWise.onPermissionDenied`      | Fired when the user denies notification permission.                                                                 |
| `AlertWise.onNotificationDisplayed` | Fired when a notification is displayed to the user.                                                                 |
| `AlertWise.onNotificationClick`     | Fired when a user clicks on a notification.                                                                         |
| `AlertWise.onNotificationClose`     | Fired when a user closes a notification.                                                                            |
| `AlertWise.onSDKError`              | Fired when an error occurs within the SDK.                                                                          |

**Example: Listening for the `onSubscribe` event**

```javascript
window.addEventListener('AlertWise.onSubscribe', function (event) {
    console.log('User subscribed!', event.detail);
    const subscriberId = event.detail.subscriber.id;
    // You can now associate this subscriberId with your internal user ID
});
```

## Development

To contribute to the development of the SDK:

1. **Clone the repository:**

   ```bash
   git clone https://github.com/alertwise-admin/alertwise-web-sdk.git
   cd alertwise-web-sdk
   ```

2. **Install dependencies:**

   ```bash
   npm install
   ```

3. **Run the development server:**
   This will start a local server with a test page.

   ```bash
   npm start
   ```

4. **Build for production:**

   ```bash
   npm run build
   ```
   This will create the distributable files in the `/dist` directory.

## License

This project is licensed under the MIT License. See the LICENSE file for details.
