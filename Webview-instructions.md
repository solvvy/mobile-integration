![Solvvy logo](assets/solvvy_logo_180x50.svg)


# Launching Solvvy in a Webview: FAQ and Technical documentation

## FAQ

### Why should Solvvy be launched in a webview?

Currently, the recommended method for integrating Solvvy into a native mobile app is to launch a webview that opens a webpage where Solvvy is already installed and configured to automatically launch.  The main benefits to this approach are:
- Updates to the Solvvy experience are immediately available to mobile users (whereas native SDKs take time to update)
- The full range of Solvvy functionality is available on mobile (the native SDKs have certain limitations around ticket form configurations)

### What exactly is a webview?
A webview is a fully-functional mobile web browser screen that runs in your app.  The only difference between a webview and a normal mobile web browser screen is that the user cannot edit the URL.  If the webpage that is loaded in the webview is very responsive and mobile-friendly, the experience can appear to be fully native.

### Which URL should be opened in the webview?

We will create a blank page where Solvvy is installed and configured to auto-launch.  Your Solutions or Sales Engineer will provide you with the exact URL once the page is set up and ready for you to use in the webview.

### What about the hand-off to support when users cannot self-serve?  What is that experience like when Solvvy is running in a webview?
It depends on what support options you have configured.

#### Email / Ticket
If Solvvy is configured to submit tickets through our back-end connection to your CRM (which is usually the case), then the ticket submission screen is part of the Solvvy flow and works great in a webview.  It feels like a native experience.  There is essentially no difference between mobile and desktop for this support channel.

#### Live Chat
In desktop browsers, Solvvy integrates with various live chat providers so that if the user selects this support option, the live chat widget from the chat provider will be launched and the question the user typed into Solvvy will be copied over as the first message in the live chat.  However, on mobile, the live chat experience is not ideal in a webview because things like push notifications don’t work as well.   For this reason, we recommend using your live chat provider’s mobile SDK to implement the live chat support channel on mobile, and Solvvy will simply close it’s own webview and hand-off control back to your app if the user selects the live chat support option.  Useful metadata, such as the question the user asked and any pre-contact form fields they selected, is made available to your app during that hand-off.

#### Phone
Similarly, for the phone support option, the Solvvy webview also dismisses itself and hands control back to your app.   This enables your app to launch the native phone app (or some other direct audio call experience), which is better than what Solvvy does for the desktop experience, which is just to display the phone number.

### What happens when a user clicks on a KB article link on the Solvvy answers screen?
This is actually one of the main benefits to the webview approach.  When a user clicks on a KB article link, it shows that article in the same webview, which is still in your app.   The user can use the back button on the webview to go back to the Solvvy modal.

### What happens when a user goes back to the app and then wants to come back to Solvvy?
The state is saved for the pre-defined time period of a Solvvy session, which is currently 1 hour.   If they return to the Solvvy flow within that time period, then they see whatever screen they left off at.  After that time period, it will start the Solvvy flow over again.

### What data does Solvvy collect, so we can know what to disclose in the Apple App Store?

Apple's web page about App Privacy Details (https://developer.apple.com/app-store/app-privacy-details/) explains that some data may not need to be disclosed if it meets the following conditions:

#### Optional disclosure
```
Data types that meet all of the following criteria are optional to disclose:

* The data is not used for tracking purposes, meaning the data is not linked with Third-Party Data for advertising or advertising measurement purposes, or shared with a data broker. For details, see the Tracking section.
* The data is not used for Third-Party Advertising, your Advertising or Marketing purposes, or for Other Purposes, as those terms are defined in the Tracking section.
Collection of the data occurs only in infrequent cases that are not part of your app’s primary functionality, and which are optional for the user.
* The data is provided by the user in your app’s interface, it is clear to the user what data is collected, the user’s name or account name is prominently displayed in the submission form alongside the other data elements being submitted, and the user affirmatively chooses to provide the data for collection each time.

If a data type collected by your app meets some, but not all, of the above criteria, it must be disclosed in App Store Connect.

Examples of data that may not need to be disclosed include data collected in optional feedback forms or customer service requests that are unrelated to the primary purpose of the app and meet the other criteria above.
```
All data that is collected by Solvvy meets all of these conditions and therefore does not need to be disclosed. In fact, data from customer support requests is specifically called out as an example of data that may not need to be disclosed.

However, if for some reason you decide to disclose data collected by Solvvy, here are the details:

#### Types of data

Solvvy only collects 2 of the data types spelled out by Apple:

* User Content > Customer support : Data generated by the user during a customer support request

* Usage Data > Product Interaction : Taps, clicks, scrolling information

#### Data use

Solvvy uses the User Content and Usage Data collected for Analytics : to evaluate user behavior, including to understand the effectiveness of existing product features, plan new features, or measure audience size or characteristics.

Solvvy also uses the User Content collected for App Functionality : to allow the user to self-service their support request or to make contact with a customer support agent.

#### Data linked to the user

None of the data that Solvvy collects is linked to a particular user.  All Usage Data is anonymized, as is the User Content data Solvvy collects.  In order to do this, Solvvy uses state-of-the-art PII redaction software (Google's DLP service) to remove any personal data from customer support requests before storing them in our system.

#### Tracking

Solvvy does not use any data collected for tracking or advertising purposes.

#### Privacy Links

Solvvy's privacy policy is here: https://solvvy.com/privacy-policy/


## Technical Documentation

### iOS Implementation Guide

1. Open Xcode and create a new Single View App.
2. Go to the Storyboard and select the View Controller.
3. Go to the Editor menu and select Embed in -> Navigation Controller.
4. In your `ViewController.swift` file add the following:
```swift
import UIKit
import WebKit
class ViewController: UIViewController {

  override func viewDidLoad() {
    super.viewDidLoad()

    let webView = WKWebView(frame: .zero)

    view.addSubview(webView)

    let layoutGuide = view.safeAreaLayoutGuide
    webView.translatesAutoresizingMaskIntoConstraints = false
    webView.leadingAnchor.constraint(
      equalTo: layoutGuide.leadingAnchor).isActive = true
    webView.trailingAnchor.constraint(
      equalTo: layoutGuide.trailingAnchor).isActive = true
    webView.topAnchor.constraint(
      equalTo: layoutGuide.topAnchor).isActive = true
    webView.bottomAnchor.constraint(
      equalTo: layoutGuide.bottomAnchor).isActive = true

    if let url = URL(string: "<YOUR WEB TICKET SUBMISSION URL WITH SOLVVY>") {
      webView.load(URLRequest(url: url))
    }
  }
}
```
This is the basic code for opening your ticket submission page (which should auto-launch Solvvy) in a webview.  If Solvvy is not installed on your web ticket submission page or does not auto-launch, contact your Solvvy Sales Engineer or Solutions Engineer.  Note: to customize the behavior of the webview in various ways, please consult the [documentation](https://developer.apple.com/documentation/webkit).

#### Passing data to the webview

In certain situations, it is necessary to pass data to Solvvy running in the webview, e.g. to specify which language the app is using so Solvvy can be properly localized, or helpful metadata that needs to be included on the ticket if the user does not self-serve (like mobile platform, app version, user ID, etc.).  This can easily be accomplished by setting JS variables on the webpage when launching the webview.  Put all your variables in the `window.solvvyConfig` object.  If you want the data to be passed directly into tickets as certain custom fields, please use the custom field ID as the variable name, as below:

```swift
let userContentController = WKUserContentController()
let scriptSource = "window.solvvyConfig = window.solvvyConfig || { " + 
                     "language : 'de'," +
                     "email : 'test@example.com'," +
                     "custom_23793987 : 'test123'," + // Support ID
                     "custom_23873898 : 'iPad'," + // Device Type (Name)
                     "darkMode : true," + // Dark mode (boolean) 
                     "some_array : [ 'item1', 'item2' ]};" + // Some array of strings
                     "window.solvvy = window.solvvy || {};"
let script = WKUserScript(source: scriptSource, injectionTime: .atDocumentStart, forMainFrameOnly: true)
userContentController.addUserScript(script)
let config = WKWebViewConfiguration()
config.userContentController = userContentController
let webView = WKWebView(frame: .zero, configuration: config)
```

#### Getting Data From the Webview

When a user is not able to self-serve, Solvvy presents a list of options (or channels) for contacting support (or automatically defaults to one if only one is configured).  Most of these support options can be handled, or executed, within the Solvvy flow, such as email ticket submission. However, for some support options (e.g. live chat), it may be preferable to execute the support contact flow directly from the native app (e.g. using a 3rd party native SDK).  To facilitate this, your native app needs to find out from the webview whether this native support flow needs to launch after the webview dismisses itself (i.e. if the user was not able to self-serve).  Your native app also needs to get the question that the user typed in at the beginning of the Solvvy flow, so they don't have to re-type their issue.  Both of these things can be accomplished with the following code.

5. Define a callback function to receive the user's original question:
```swift
override func viewDidLoad() {
  super.viewDidLoad()

  let config = WKWebViewConfiguration()
  let userContentController = WKUserContentController()
  ...

  userContentController.add(self, name: "supportOptionHandler")

  config.userContentController = userContentController

  let webView = WKWebView(frame: .zero, configuration: config)
  view.addSubview(webView)
  ...

}

extension ViewController: WKScriptMessageHandler {
  func userContentController(_ userContentController: WKUserContentController,
    didReceive message: WKScriptMessage) {
    if let messageBody = message.body as? [String: Any],
      let userQuestion = messageBody["userQuestion"] as? String,
      let supportOption = messageBody["supportOption"] as? String {
      print("userQuestion: \(userQuestion), supportOption: \(supportOption)")
    }
  }
}
```

Make sure you use the exact name: `name: "supportOptionHandler"` because that is what Solvvy will call when a support option is clicked by the user.


6. Now your `ViewController.swift` should look like this:
```swift
import UIKit
import WebKit

class ViewController: UIViewController, WKNavigationDelegate {

  override func viewDidLoad() {
    super.viewDidLoad()

    let config = WKWebViewConfiguration()
    let userContentController = WKUserContentController()
    let scriptSource = "window.solvvyConfig = window.solvvyConfig || { " + 
                         "language : 'de'," +
                         "email : 'test@example.com'," +
                         "custom_23793987 : 'test123'," + // Support ID
                         "custom_23873898 : 'iPad'," + // Device Type (Name)
                         "darkMode : true," + // Dark mode (boolean) 
                         "some_array : [ 'item1', 'item2' ]};" + // Some array of strings
                         "window.solvvy = window.solvvy || {};"
    let script = WKUserScript(source: scriptSource, injectionTime: .atDocumentStart, forMainFrameOnly: true)
    userContentController.addUserScript(script)
    
    userContentController.add(self, name: "supportOptionHandler")

    config.userContentController = userContentController

    let webView = WKWebView(frame: .zero, configuration: config)

    view.addSubview(webView)

    let layoutGuide = view.safeAreaLayoutGuide
    webView.translatesAutoresizingMaskIntoConstraints = false
    webView.leadingAnchor.constraint(
      equalTo: layoutGuide.leadingAnchor).isActive = true
    webView.trailingAnchor.constraint(
      equalTo: layoutGuide.trailingAnchor).isActive = true
    webView.topAnchor.constraint(
      equalTo: layoutGuide.topAnchor).isActive = true
    webView.bottomAnchor.constraint(
      equalTo: layoutGuide.bottomAnchor).isActive = true

    if let url = URL(string: "<YOUR WEB TICKET SUBMISSION URL WITH SOLVVY>") {
        webView.load(URLRequest(url: url))
    }
  }
}

extension ViewController: WKScriptMessageHandler {
  func userContentController(_ userContentController: WKUserContentController,
    didReceive message: WKScriptMessage) {
    if let messageBody = message.body as? [String: Any],
      let userQuestion = messageBody["userQuestion"] as? String,
      let supportOption = messageBody["supportOption"] as? String {
      print("userQuestion: \(userQuestion), supportOption: \(supportOption)")
    }
  }
}
```

#### Closing the webview when the user self-served

When a user is satisfied with one of the answers returned, they can click "Yes" to indicate they got their answer.  On the next screen, if they click "Close" to end the Solvvy experience, the webview needs to pass control back to the native app.  This happens through another callback function which the Solvvy modal calls when that "Close" button is clicked.  Use the following code to handle that callback:
```swift

...
  userContentController.add(self, name: "exitHandler")
...

extension ViewController: WKScriptMessageHandler {
    func userContentController(_ userContentController: WKUserContentController, didReceive message: WKScriptMessage) {
        // do something like close the webview
        print("EXIT HANDLER CALLED!")
    }
}
```
Please note that the userContentController method is called with only an empty string argument. 

#### Allowing attachments on tickets

If you want to allow Email/Ticket as one of the options for contacting support, and you want to allow the users to add attachments from their local device to the ticket form, then you will need to add the following code:
```swift
import UIKit
import WebKit

class ViewController: UIViewController, WKNavigationDelegate {
    
    var lastTapPosition: CGPoint = CGPoint(x: 0, y: 0)
    
    override func viewDidLoad() {
        super.viewDidLoad()
        
        ...
        
        let webView = WKWebView(frame: .zero, configuration: config)
        
        // Intercept the user tap and store its location so we can use it to position our menu on screen.
        let tapGesture = UITapGestureRecognizer(target: self, action: #selector(webViewTapped(_:)))
        tapGesture.delegate = self
        webView.addGestureRecognizer(tapGesture)
        
        view.addSubview(webView)
        
        ...
    }
    
    @objc
    func webViewTapped(_ sender: UITapGestureRecognizer) {
        // Store the last tap location.
        self.lastTapPosition = sender.location(in: self.view)
    }
    
    // One problem here is that we are not the ones declaring a variable for our UIPopoverPresentationController but
    // it is coming from the web view itself, so we need to find a way to tell the controller that our View Controller
    // will be its delegate for this situation. We can achieve that by overriding the present method from our
    // View Controller to be able to set it as the delegate of the popover.
    override open func present(_ viewControllerToPresent: UIViewController, animated flag: Bool, completion: (() -> Void)? = nil) {
        if #available(iOS 13, *) {
            viewControllerToPresent.popoverPresentationController?.delegate = self
        }
        super.present(viewControllerToPresent, animated: flag, completion: completion)
    }
}

...

extension ViewController: UIPopoverPresentationControllerDelegate {
    func prepareForPopoverPresentation(_ popoverPresentationController: UIPopoverPresentationController) {
        popoverPresentationController.sourceView = self.view // with this code the crash doesn’t happen anymore
        popoverPresentationController.sourceRect = CGRect(origin: self.lastTapPosition, size: CGSize(width: 0, height: 0)) // Display on last tap position on webview
    }
}

// No matter how much we tap the web view, our webViewTapped method will never be called. That’s because our web view
// is already intercepting our taps to perform the necessary web view actions and, if we want to have our
// View Controller to recognize them simultaneously, we have to make our View Controller conform to the
// UIGestureRecognizerDelegate protocol
extension ViewController: UIGestureRecognizerDelegate {
    func gestureRecognizer(_ gestureRecognizer: UIGestureRecognizer, shouldRecognizeSimultaneouslyWith otherGestureRecognizer: UIGestureRecognizer) -> Bool {
        return true
    }
}
```

#### Intercept URL requests from within a webview

First, make something conform to `WKNavigationDelegate`. For example:

```swift
class ViewController: UIViewController, WKNavigationDelegate {
```

Second, make that object the navigation delegate of your web view. If you were using your view controller, you’d write this:

```swift
webView.navigationDelegate = self
```

Finally, implement the `decidePolicyFor` method with whatever logic should decide whether the page is loaded normally or execute deep links to app screens. For deep links make sure you call the `decisionHandler()` closure with `.cancel` so the load halts.

As an example, this implementation will load all links inside the web view as long as they don’t go to the Apple homepage:

```swift
func webView(_ webView: WKWebView,
             decidePolicyFor navigationAction: WKNavigationAction,
             decisionHandler: @escaping (WKNavigationActionPolicy) -> Void) {
    if let host = navigationAction.request.url?.host, host == "www.apple.com" {
        // do stuff or call deep links
        decisionHandler(.cancel)
        return
    }

    decisionHandler(.allow)
}
```


### Android Implementation Guide

1. In Android Studio, go to your `manifests/AndroidManifest.xml` file and add the following:
```xml
<uses-permission android:name="android.permission.INTERNET"/>
```
This allows the WebView to access internet within the app.

If you allow your customers to submit attachments through your Solvvy ticket form using the "Add File" button, you also need to add the following to your AndroidManifest file:
```xml
<uses-permission android:name="android.permission.READ_EXTERNAL_STORAGE"/>
```

2. In your `layout/activity_main.xml` file add the following:
```xml
<WebView
   android:id="@+id/my_web_view"
   android:layout_width="match_parent"
   android:layout_height="match_parent"
   />
```

3. In your `MainActivity.kt` file add the following:
```kotlin
package com.example.myapplication

import android.content.pm.ApplicationInfo
import android.os.Build
import androidx.appcompat.app.AppCompatActivity
import android.os.Bundle
import android.webkit.JavascriptInterface
import android.webkit.WebView
import android.webkit.WebViewClient
import kotlinx.android.synthetic.main.activity_main.*

class MainActivity : AppCompatActivity() {

  override fun onCreate(savedInstanceState: Bundle?) {
    super.onCreate(savedInstanceState)
    setContentView(R.layout.activity_main)

    if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.KITKAT) {
      if (0 != (applicationInfo.flags and
                ApplicationInfo.FLAG_DEBUGGABLE)) {
        WebView.setWebContentsDebuggingEnabled(true)
      }
    }
    my_web_view.settings.javaScriptEnabled = true
    my_web_view.settings.domStorageEnabled = true
    my_web_view.settings.databaseEnabled = true
    my_web_view.webViewClient = object : WebViewClient() {
    }
    my_web_view.loadUrl(BASE_URL)
  }

  companion object {
    private val BASE_URL = "<YOUR WEB TICKET SUBMISSION URL WITH SOLVVY>"
  }
}
```
This is the basic code for opening your ticket submission page (which should auto-launch Solvvy) in a webview.  If Solvvy is not installed on your web ticket submission page or does not auto-launch, contact your Solvvy Sales Engineer or Solutions Engineer.  Note: to customize the behavior of the webview in various ways, please consult the [documentation](https://developer.android.com/reference/android/webkit/WebView).

#### Navigating back to Solvvy after opening a link to a KB article

Users have the ability to see the original KB articles that contain the solutions Solvvy returns by clicking on the "Read Article" link underneath each solution.  On desktop, these links open in a new browser tab.  On mobile, if you follow this guide, those links will open in the same webview.  However, in this scenario, the user needs a way to navigate back to Solvvy.  Add this code to enable the device Back button to trigger the "Back" navigation function in the webview:
```kotlin
  override fun onKeyDown(keyCode: Int, event: KeyEvent?): Boolean {
    // Check if the key event was the Back button and if there's history
    if (keyCode == KeyEvent.KEYCODE_BACK && my_web_view.canGoBack()) {
        my_web_view.goBack()
        return true
    }
    // If it wasn't the Back key or there's no web page history, bubble up to the default
    // system behavior (probably exit the activity)
    return super.onKeyDown(keyCode, event)
  }
```

#### Passing data to the webview

In certain situations, it is necessary to pass data to Solvvy running in the webview, e.g. to specify which language the app is using so Solvvy can be properly localized, or helpful metadata that needs to be included on the ticket if the user does not self-serve (like mobile platform, app version, user ID, etc.).  This can easily be accomplished by setting JS variables on the webpage when launching the webview.  Put all your variables in the `window.solvvyConfig` object.  If you want the data to be passed directly into tickets as certain custom fields, please use the custom field ID as the variable name, as below:

```kotlin
my_web_view.webViewClient = object : WebViewClient() {
    override fun onPageStarted(view: WebView, url: String, favicon: Bitmap?) {
        if (url == BASE_URL) {
            my_web_view.loadUrl("javascript: window.solvvyConfig = window.solvvyConfig || { " +
                         "language : 'de'," +
                         "email : 'test@example.com'," +
                         "custom_23793987 : 'test123'," + // Support ID
                         "custom_23873898 : 'iPad'," + // Device Type (Name)
                         "darkMode : true," + // Dark mode (boolean) 
                         "some_array : [ 'item1', 'item2' ]};" + // Some array of strings
                         "window.solvvy = window.solvvy || {};")
        }
    }
    ...
}
```

#### Getting data from the webview

When a user is not able to self-serve, Solvvy presents a list of options (or channels) for contacting support (or automatically defaults to one if only one is configured).  Most of these support options can be handled, or executed, within the Solvvy flow, such as email ticket submission. However, for some support options (e.g. live chat), it may be preferable to execute the support contact flow directly from the native app (e.g. using a 3rd party native SDK).  To facilitate this, your native app needs to find out from the webview whether this native support flow needs to launch after the webview dismisses itself (i.e. if the user was not able to self-serve).  Your native app also needs to get the question that the user typed in at the beginning of the Solvvy flow, so they don't have to re-type their issue.  Both of these things can be accomplished with the following code.

4. Define a callback function to receive the user's original question:
```kotlin
private inner class SupportOptionHandler {
  @JavascriptInterface
  fun handleSupportOption(supportOption: String, userQuestion: String) {
    // do something with the user question and support option
    println("question: $userQuestion, option: $supportOption")
  }
}
```

5. Make this callback available to the webview:
```kotlin
  override fun onCreate(savedInstanceState: Bundle?) {
    ...
    my_web_view.addJavascriptInterface(SupportOptionHandler(), HANDLER_NAME)
    ...
  }

  companion object {
    ...
    private val HANDLER_NAME = "supportOptionHandler"
  }
```

6. Inject the handler onto the webpage:
```kotlin
  override fun onCreate(savedInstanceState: Bundle?) {
    ...
    my_web_view.webViewClient = object : WebViewClient() {
      override fun onPageFinished(view: WebView, url: String) {
        if (url == BASE_URL) {
          injectJavaScriptFunction()
        }
      }
    }
    ...
  }

  private fun injectJavaScriptFunction() {
    my_web_view.loadUrl(
      "javascript: " +
        "window.solvvy = window.solvvy || {};" +
        "window.solvvy.native = window.solvvy.native || {};" +
        "window.solvvy.native = { androidSupportOptionHandler: {} };" +
        "window.solvvy.native.androidSupportOptionHandler.handle = " +
        "function(option, question) { " +
        HANDLER_NAME + ".handleSupportOption(option, question); };"
    )
  }
```

Make sure you use the exact variable path: `window.solvvy.native.androidSupportOptionHandler.handle` because that is what Solvvy will call when a support option is clicked by the user.

7. Clean up
Be sure to clean up the handler as follows:
```kotlin
  override fun onDestroy() {
    my_web_view.removeJavascriptInterface(HANDLER_NAME)
    super.onDestroy()
  }
```
8. (Optional) Enable webview console logs for debugging
If you want to be able to see the webview browser log messages in your Android Studio for debugging purposes, do this:
```kotlin
  override fun onCreate(savedInstanceState: Bundle?) {
    ...
    my_web_view.setWebChromeClient(object : WebChromeClient() {
      override fun onConsoleMessage(consoleMessage: ConsoleMessage): Boolean {
        android.util.Log.d("WebView", consoleMessage.message())
        return true
      }
    })
    ...
  }
```


9. Now your `MainActivity.kt` should look like this:
```kotlin
package com.example.myapplication

import android.content.pm.ApplicationInfo
import android.os.Build
import androidx.appcompat.app.AppCompatActivity
import android.os.Bundle
import android.webkit.JavascriptInterface
import android.webkit.WebView
import android.webkit.WebViewClient
import kotlinx.android.synthetic.main.activity_main.*
import android.R.id.message
import android.webkit.ConsoleMessage
import android.webkit.WebChromeClient

class MainActivity : AppCompatActivity() {
  override fun onCreate(savedInstanceState: Bundle?) {
    super.onCreate(savedInstanceState)
    setContentView (R.layout.activity_main)
    if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.KITKAT) {
        if (0 != (applicationInfo.flags and ApplicationInfo.FLAG_DEBUGGABLE)) {
            WebView.setWebContentsDebuggingEnabled(true)
        }
    }
    my_web_view.settings.javaScriptEnabled = true
    my_web_view.settings.domStorageEnabled = true
    my_web_view.settings.databaseEnabled = true

    my_web_view.addJavascriptInterface (SupportOptionHandler(), HANDLER_NAME)
    my_web_view.setWebChromeClient(object : WebChromeClient() {
      override fun onConsoleMessage(consoleMessage: ConsoleMessage): Boolean {
        android.util.Log.d("WebView", consoleMessage.message())
        return true
      }
    })
    my_web_view.webViewClient = object : WebViewClient() {
      override fun onPageFinished(view: WebView, url: String) {
        if (url == BASE_URL) {
          injectJavaScriptFunction()
        }
      }
      override fun onPageStarted(view: WebView, url: String, favicon: Bitmap?) {
          if (url == BASE_URL) {
              my_web_view.loadUrl("javascript: window.solvvyConfig = window.solvvyConfig || { " +
                            "language : 'de'," +
                            "email : 'test@example.com'," +
                            "custom_23793987 : 'test123'," + // Support ID
                            "custom_23873898 : 'iPad'," + // Device Type (Name)
                            "darkMode : true," + // Dark mode (boolean) 
                            "some_array : [ 'item1', 'item2' ]};" + // Some array of strings
                            "window.solvvy = window.solvvy || {};")
          }
      }
    }
    my_web_view.loadUrl(BASE_URL)
  }

  override fun onKeyDown(keyCode: Int, event: KeyEvent?): Boolean {
    // Check if the key event was the Back button and if there's history
    if (keyCode == KeyEvent.KEYCODE_BACK && my_web_view.canGoBack()) {
        my_web_view.goBack()
        return true
    }
    // If it wasn't the Back key or there's no web page history, bubble up to the default
    // system behavior (probably exit the activity)
    return super.onKeyDown(keyCode, event)
  }

  override fun onDestroy() {
    my_web_view.removeJavascriptInterface(HANDLER_NAME)
    super.onDestroy()
  }

  private fun injectJavaScriptFunction() {
    my_web_view.loadUrl(
      "javascript: " +
        "window.solvvy = window.solvvy || {};" +
        "window.solvvy.native = window.solvvy.native || {};" +
        "window.solvvy.native = { androidSupportOptionHandler: {} };" +
        "window.solvvy.native.androidSupportOptionHandler.handle = " +
        "function(option, question) { " +
        HANDLER_NAME + ".handleSupportOption(option, question); };"
    )
  }

  private inner class SupportOptionHandler {
    @JavascriptInterface
    fun handleSupportOption(supportOption: String, userQuestion: String) {
      // do something with the user question and support option
      println("question: $userQuestion, option: $supportOption")

    }
  }

  companion object {
    private val HANDLER_NAME = "supportOptionHandler"
    private val BASE_URL = "<YOUR WEB TICKET SUBMISSION URL WITH SOLVVY>"
  }
}
```

#### Closing the webview when the user self-served

When a user is satisfied with one of the answers returned, they can click "Yes" to indicate they got their answer.  On the next screen, if they click "Close" to end the Solvvy experience, the webview needs to pass control back to the native app.  This happens through another callback function which the Solvvy modal calls when that "Close" button is clicked.  Use the following code to handle that callback:
```kotlin
  
  ...
  my_web_view.addJavascriptInterface (ExitHandler(), EXIT_HANDLER_NAME)
  ...
  
  private fun injectJavaScriptFunction() {
      my_web_view.loadUrl("javascript: window.solvvy = window.solvvy || {};" +
          "window.solvvy.native = window.solvvy.native || {};" +
          "window.solvvy.native = { androidExitHandler: {} };" +
          "window.solvvy.native.androidExitHandler.handle = function() { " +
          EXIT_HANDLER_NAME + ".handleExit(); };")
  }

  private inner class ExitHandler {
      @JavascriptInterface
      fun handleExit() {
          // do something like close the webview
          println("EXIT HANDLER CALLED!")
      }
  }

  override fun onDestroy() {
      my_web_view.removeJavascriptInterface(EXIT_HANDLER_NAME)
      super.onDestroy()
  }

   companion object {
        private val EXIT_HANDLER_NAME = "exitHandler"
   }
```

#### Injecting multiple JavaScript functions

In order to inject multiple JavaScript functions into the Webview, like handling support options and exit simlutaneously, you would set up your injectJavaScriptFunction() like this:
```kotlin
private fun injectJavaScriptFunction() {
    my_web_view.loadUrl(
        "javascript: " +
            "window.solvvy = window.solvvy || {};" +
            "window.solvvy.native = window.solvvy.native || {};" +
            "window.solvvy.native = { androidSupportOptionHandler: {}, androidExitHandler: {} };" +
            "window.solvvy.native.androidSupportOptionHandler.handle = " +
            "function(option, question) { " +
            HANDLER_NAME + ".handleSupportOption(option, question); };" +
            "window.solvvy.native.androidExitHandler.handle = function() { " +
            EXIT_HANDLER_NAME + ".handleExit(); };"
    )
}
```


#### Allowing attachments on tickets

If you want to allow Email/Ticket as one of the options for contacting support, and you want to allow the users to add attachments from their local device to the ticket form, then you will need to add the following code:
```kotlin
import android.net.Uri
import android.content.Intent
import android.webkit.ValueCallback
import android.webkit.WebView

class MainActivity : AppCompatActivity() {
    val REQUEST_CODE_LOLIPOP = 1
    private val RESULT_CODE_ICE_CREAM = 2
    private var mFilePathCallback: ValueCallback<Array<Uri>>? = null
    private var mUploadMessage: ValueCallback<Uri>? = null

    ...
    
    override fun onCreate(savedInstanceState: Bundle?) {
    
    ...
    
          my_web_view.setWebChromeClient(object : WebChromeClient() {
            override fun onConsoleMessage(consoleMessage: ConsoleMessage): Boolean {
                android.util.Log.d("WebView", consoleMessage.message())
                return true
            }

            //For Android 5.0 above
            override fun onShowFileChooser(
                webView: WebView?,
                filePathCallback: ValueCallback<Array<Uri>>?,
                fileChooserParams: FileChooserParams?
            )
                    : Boolean {
                return this@MainActivity.onShowFileChooser(
                    webView,
                    filePathCallback,
                    fileChooserParams
                )
            }

            // For Android 3.0+
            fun openFileChooser(uploadMsg: ValueCallback<Uri>) {
                this.openFileChooser(uploadMsg, "*/*")
            }

            // For Android 3.0+
            fun openFileChooser(uploadMsg: ValueCallback<Uri>, acceptType: String) {
                this.openFileChooser(uploadMsg, acceptType, null)
            }

            //For Android 4.1
            fun openFileChooser(
                uploadMsg: ValueCallback<Uri>,
                acceptType: String, capture: String?
            ) {
                this@MainActivity.openFileChooser(uploadMsg, acceptType, capture)
            }
        })
        my_web_view.webViewClient = object : WebViewClient() {
        
        ...
        
      }
      
      ...
      
  }
  
  ...

  fun onShowFileChooser(
      webView: WebView?,
      filePathCallback: ValueCallback<Array<Uri>>?,
      fileChooserParams: WebChromeClient.FileChooserParams?
  )
          : Boolean {

      mFilePathCallback?.onReceiveValue(null)
      mFilePathCallback = filePathCallback
      if (Build.VERSION.SDK_INT >= 21) {
          val intent =fileChooserParams?.createIntent()
          startActivityForResult(intent, REQUEST_CODE_LOLIPOP)
      }

      return true
  }

  fun openFileChooser(
      uploadMsg: ValueCallback<Uri>,
      acceptType: String, capture: String?
  ) {
      mUploadMessage = uploadMsg
      val i = Intent(Intent.ACTION_GET_CONTENT)
      i.addCategory(Intent.CATEGORY_OPENABLE)
      i.type = acceptType
      this@MainActivity.startActivityForResult(
          Intent.createChooser(i, "File Browser"),
          RESULT_CODE_ICE_CREAM
      )
  }

  public override fun onActivityResult(requestCode: Int, resultCode: Int, data: Intent?) {
      super.onActivityResult(requestCode, resultCode, data)
      when (requestCode) {
          RESULT_CODE_ICE_CREAM -> {
              var uri: Uri? = null
              if (data != null) {
                  uri = data.data
              }
              mUploadMessage?.onReceiveValue(uri)
              mUploadMessage = null
          }
          REQUEST_CODE_LOLIPOP -> {

              if (Build.VERSION.SDK_INT >= 21) {
                  val results = WebChromeClient.FileChooserParams.parseResult(resultCode, data)
                  mFilePathCallback?.onReceiveValue(results)
              }
              mFilePathCallback = null
          }
      }
  }
```

####  Intercept URL requests from within a webview   
There are many options to intercept URL's within a Webview on Android, each one depends on the scope and the requirements that you have.  
  
#####  Option 1: Override URL loading  
Give the host application a chance to take control when a URL is about to be loaded in the current WebView. If a WebViewClient is not provided, by default WebView will ask Activity Manager to choose the proper handler for the URL. If a WebViewClient is provided, returning `true` causes the current WebView to abort loading the URL, while returning `false` causes the WebView to continue loading the URL as usual.  
  
```kotlin  
webView.webViewClient = object : WebViewClient() {  
 override fun shouldOverrideUrlLoading( view: WebView, request: WebResourceRequest ): Boolean { return if(request.url.lastPathSegment == "error.html") { view.loadUrl("https//host.com/home.html") true } else { false } }}  
```  
For API<24 please use  
```kotlin  
public boolean shouldOverrideUrlLoading (WebView view,  
 String url)
```  
This option has the next limitations:  
  
 - It does not catch POST request.
 - It is not triggered on any resources loaded inside the page. i.e. images, scripts, etc.
 - It is not triggered on any HTTP request made by JavaScript on the page.  
 
For more informatioon about this method please refer to the [documentation](https://developer.android.com/reference/android/webkit/WebViewClient#shouldOverrideUrlLoading%28android.webkit.WebView,%20android.webkit.WebResourceRequest%29)  
  
#####  Option 2: Redirect resources loading  
Notify the host application that the WebView will load the resource specified by the given url.  
```kotlin  
webView.webViewClient = object : WebViewClient() {  
override fun onLoadResource(view: WebView, url: String) {  
 view.stopLoading() view.loadUrl(newUrl) // this will trigger onLoadResource }}  
```  
`onLoadResource`  providers similar functionality to  `shouldOverrideUrlLoading`. But`onLoadResource`  will be called for any  **resources (images, scripts, etc)** loaded on the current page including  **the page itself**.  
  
**You must put an exit condition on the handling logic since this function will be triggered on `loadUrl(newUrl)`.** For example:  
```kotlin  
webView.webViewClient = object : WebViewClient() {  
override fun onLoadResource(view: WebView, url: String) {  
 // exit the redirect loop if landed on homepage if(url.endsWith("home.html")) return // redirect to home page if the page to load is error page if(url.endsWith("error.html")) { view.stopLoading() view.loadUrl("https//host.com/home.html") } }}  
```  
This option has the next limitations:  
  
 - It is not triggered on any HTTP request made by JavaScript on the page.  
 For more information please refer to the [documentation](https://developer.android.com/reference/android/webkit/WebViewClient#onLoadResource%28android.webkit.WebView,%20java.lang.String%29).  

##### Option 3: Handle all requests  
Notify the host application of a resource request and allow the application to return the data. If the return value is  `null`, the WebView will continue to load the resource as usual. Otherwise, the return response and data will be used.  
  
This callback is invoked for a variety of URL schemes (e.g.,  `http(s):`,  `data:`,  `file:`, etc.), not only those schemes which send requests over the network. This is not called for  `javascript:`  URLs,  `blob:`  URLs, or for assets accessed via  `file:///android_asset/`  or  `file:///android_res/`  URLs.  
  
In the case of redirects, this is only called for the initial resource URL, not any subsequent redirect URLs.  
  
```kotlin  
webView.webViewClient = object : WebViewClient() {  
 override fun shouldInterceptRequest( view: WebView, request: WebResourceRequest ): WebResourceResponse? { return super.shouldInterceptRequest(view, request) }}  
```  
For example, we want to provide a local error page.  
```kotlin  
webView.webViewClient = object : WebViewClient() {  
 override fun shouldInterceptRequest( view: WebView, request: WebResourceRequest ): WebResourceResponse? { return if (request.url.lastPathSegment == "error.html") { WebResourceResponse( "text/html", "utf-8", assets.open("error") ) } else { super.shouldInterceptRequest(view, request) } }}  
```  
  **This function is running in a background thread similar to how you execute an API call in the background thread. Any attempt to modify the content of the WebView inside this function will cause an exception. i.e. `loadUrl, evaluateJavascript, etc`.**  
  
For API<21 please use:  
```kotlin  
public WebResourceResponse shouldInterceptRequest (WebView view, String url)  
```  
  
This option has the next limitations:  
  
 - There is no **payload** field on the `WebResourceRequest`.  For example, if you want to create a new user with a POST API request. You cannot get the POST payload from the `WebResourceRequest`. - This method is called on a thread other than the UI thread so clients should exercise caution when accessing private data or the view system.  
 For more information please refer to the [documentation](https://developer.android.com/reference/android/webkit/WebViewClient#shouldInterceptRequest%28android.webkit.WebView,%20android.webkit.WebResourceRequest%29)  
 ##### Other optionsThere are other no conventional options that can allow us to:  
  
 - Resolve payload for POST requests. - Ensure JS override available on every page. - Inject JS code into each HTML page.  
These options are for more specifics requirements and are using JS or HTML overriding if you want to explore those options, please refer to this [post](https://medium.com/@madmuc/intercept-all-network-traffic-in-webkit-on-android-9c56c9262c85), but most of the scenarios are cover on the above options.
