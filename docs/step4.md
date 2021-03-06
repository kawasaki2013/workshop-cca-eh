## Step 4: Eh, For Sure

### Sending an Eh

Okay, now you have a contact list. If you tap on an online contact, you'll actually see an error in your foreground page: the custom elements are configured to call a method called `sendEhTo`. Let's add that method inside `main.js`:

    function sendEhTo(contactInfo) {
      console.info('start sending eh to', contactInfo.userid);
      window.opener.sendEh(contactInfo.userid, function() {
        console.info('finish sending eh to', contactInfo.userid);
      });
    }

The `sendEhTo` method needs `sendEh` on the background page. Let's add it inside `background.js`:

    function sendEh(userid, callback) {
      allUsers[userid].outboundEhCount++;
      allUsers[userid].isCurrentlySendingMessageEh = true;

      sendGcmMessage({
        'type': 'sendEh',
        'to_userid': userid
      }, function() {
        allUsers[userid].isCurrentlySendingMessageEh = false;
        callback();
      });
    }

If you run the app, you should be able to Eh everyone using the app! However, we don't yet do anything.

### Receiving an Eh

If you were to receive an Eh right now, your client will simply log a warning saying that the message received is unsupported. Let's fix that inside `background.js`, adding a new case to our `select` block near here:

        // Incoming GCM data: we'll add callback here later [1].
        ...
        switch(msg.data.type) {
          case 'incomingEh': {
            onIncomingEh(msg.data.from_userid);
            break;
          }
          ...

And add a top-level `onIncomingEh` function, which does something sensible and shows a short notification:

    function onIncomingEh(from_userid) {
      var profile = allUsers[from_userid];
      if (!profile) {
        console.warn('no profile for Eh-sender', from_userid);
        return;
      }
      console.log('received Eh from', from_userid);
      profile.inboundEhCount++;

      // Incoming Eh: we'll update the notification here later [4].
      var options = {
        type: 'basic',
        title: 'Eh',
        iconUrl: '/assets/icons/icon128.png',
        message: profile.name + ' x' + profile.inboundEhCount
      };
      chrome.notifications.create(from_userid, options, function() {});

      chrome.notifications.onClicked.addListener(function(id) {
        var windows = chrome.app.window.getAll();
        if (windows) {
          windows[0].restore();
        } else {
          createUiWindow();
        }
        chrome.notifications.clear(id, function() {});
      });
    }

### Permissions

As the very last step, we actually have to enable the `notifications` permission inside `manifest.json`. Without this, you'll see an error in the logs when we try to show a Desktop or mobile notification. Update the permissions section to look like this:

    "permissions": [
      "gcm",
      "identity",
      "notifications",
      "<all_urls>"
    ],

With that, you're all done! Congratulations, eh! Reward yourself with some poutine.

### Next Up...

The code for this step is in [step4](https://github.com/MobileChromeApps/workshop-cca-eh/blob/master/workshop/step4).

You are basically done with your app, Eh!  You can now work on making it look good, add some optional technical extensions, or try to publish your app the the store.

_**Continue to [Extensions &raquo;](https://github.com/MobileChromeApps/workshop-cca-eh/blob/master/docs/extensions.md)**_

or

_**Continue to [Publishing &raquo;](https://github.com/MobileChromeApps/workshop-cca-eh/blob/master/docs/publish.md)**_
