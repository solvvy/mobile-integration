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
There are a couple of ways to do it:
1. Create a blank page where Solvvy is installed and configured to auto-launch.
2. Use an existing page where Solvvy is installed and configured to auto-launch.  Many of our clients have Solvvy installed and configured to auto-launch on their existing web ticket submission form.  Using this URL would be the easiest option because it requires no additional set up.

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

#### Getting Data From the Webview

When a user is not able to self-serve, Solvvy presents a list of options (or channels) for contacting support (or automatically defaults to one if only one is configured).  Most of these support options can be handled, or executed, within the Solvvy flow, such as email ticket submission. However, for some support options (e.g. live chat), it may be preferable to execute the support contact flow directly from the native app (e.g. using a 3rd party native SDK).  To facilitate this, your native app needs to find out from the webview whether this native support flow needs to launch after the webview dismisses itself (i.e. if the user was not able to self-serve).  Your native app also needs to get the question that the user typed in at the beginning of the Solvvy flow, so they don't have to re-type their issue.  Both of these things can be accomplished with the following code.

5. Define a callback function to receive the user's original question:
```
override func viewDidLoad() {
  super.viewDidLoad()

  let config = WKWebViewConfiguration()
  let userContentController = WKUserContentController()

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


6. The final version of the `ViewController.swift` should look like this:
```
import UIKit
import WebKit

class ViewController: UIViewController, WKNavigationDelegate {

  override func viewDidLoad() {
    super.viewDidLoad()

    let config = WKWebViewConfiguration()
    let userContentController = WKUserContentController()

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


### Android Implementation Guide

1. In Android Studio, go to your `manifests/AndroidManifest.xml` file and add the following:
```xml
<uses-permission android:name="android.permission.INTERNET"/>
```
This allows the WebView to access internet within the app.

2. In your `layout/activity_main.xml` file add the following:
```xml
<WebView
   android:id="@+id/my_web_view"
   android:layout_width="match_parent"
   android:layout_height="match_parent"
   />
```

3. In your `MainActivity.kt` file add the following:
```
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
    my_web_view.webViewClient = object : WebViewClient()
    my_web_view.loadUrl(BASE_URL)
  }

  companion object {
    private val BASE_URL = "<YOUR WEB TICKET SUBMISSION URL WITH SOLVVY>"
  }
}
```
This is the basic code for opening your ticket submission page (which should auto-launch Solvvy) in a webview.  If Solvvy is not installed on your web ticket submission page or does not auto-launch, contact your Solvvy Sales Engineer or Solutions Engineer.  Note: to customize the behavior of the webview in various ways, please consult the [documentation](https://developer.android.com/reference/android/webkit/WebView).

#### Getting Data From the Webview

When a user is not able to self-serve, Solvvy presents a list of options (or channels) for contacting support (or automatically defaults to one if only one is configured).  Most of these support options can be handled, or executed, within the Solvvy flow, such as email ticket submission. However, for some support options (e.g. live chat), it may be preferable to execute the support contact flow directly from the native app (e.g. using a 3rd party native SDK).  To facilitate this, your native app needs to find out from the webview whether this native support flow needs to launch after the webview dismisses itself (i.e. if the user was not able to self-serve).  Your native app also needs to get the question that the user typed in at the beginning of the Solvvy flow, so they don't have to re-type their issue.  Both of these things can be accomplished with the following code.

4. Define a callback function to receive the user's original question:
```
private inner class SupportOptionHandler {
  @JavascriptInterface
  fun handleSupportOption(supportOption: String, userQuestion: String) {
    // do something with the user question and support option
    println("question: $userQuestion, option: $supportOption")
  }
}
```

5. Make this callback available to the webview:
```
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
```
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
```
  override fun onDestroy() {
    my_web_view.removeJavascriptInterface(HANDLER_NAME)
    super.onDestroy()
  }
```
8. (Optional) Enable webview console logs for debugging
If you want to be able to see the webview browser log messages in your Android Studio for debugging purposes, do this:
```
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


9. The final version of the `MainActivity.kt` should look like this:
```
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
    }
    my_web_view.loadUrl(BASE_URL)
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
