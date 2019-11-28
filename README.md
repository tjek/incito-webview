# Incito WebView

This HTML file contains the Incito web renderer, plus a number of JS functions for loading an incito, and accessing information about the DOM once the Incito has been rendered.

Whenever this html file changes, it must be published to `https://incito-webview.shopgun.com/index-1.0.0.html`.

If there are any breaking changes to this file, a duplicate of the document must be created with a different version number, and this must also be hosted on the CDN. We must continue to support the older versions for backwards compatibility.

## Android Implementation

JavaScript interface to receive events and data from javascript
```
public class JavaScriptInterface {
    private Activity activity;

    public JavaScriptInterface(Activity activity) {
        this.activity = activity;
    }

    // Called on click event from JS. 
    // viewIds is an array with all the ids of the view found at the clicked point.
    @JavascriptInterface
    public void viewClicked(String[] viewIds) { }

    // Called on scroll event from JS.
    // progress * 100 will give you the percentage of scrolling (useful for a ProgressBar)
    // scrollOffset is the window.pageYOffset
    @JavascriptInterface
    public void progress(float progress, float scrollOffset) { }
    
    // Called when the rendering is done, so you can dismiss whatever loading view you have
    @JavascriptInterface
    public void initDone() { }
}
```

WebView setup
```
WebSettings webSettings = incitoWebView.getSettings();

// Enable Javascript
webSettings.setJavaScriptEnabled(true);
JavaScriptInterface jsInterface = new JavaScriptInterface(this);
webView.addJavascriptInterface(jsInterface, "androidJSInterface");

webView.setWebViewClient(new WebViewClient(){
    public void onPageFinished(WebView view, String url){
        // Initialize the html doc with the incito JSON string
        incitoWebView.evaluateJavascript("javascript:init(" + incitoJSONString + ")", null);
    }
});

// Load the content of the html doc from your asset folder or from an url
webView.loadUrl(htmlStr);
```

## Swift Implementation

First, initialize your WKWebView to listen for the messages sent back from the .html file:

```
let config = WKWebViewConfiguration()
config.allowsInlineMediaPlayback = true

// init JS interface
let contentController = WKUserContentController()
contentController.add(self, name: "incitoFinishedRendering")
config.userContentController = contentController

let webView = WKWebView(frame: .zero, configuration: config)
```
> Note: You must conform to the `WKScriptMessageHandler` protocol and implement the `userContentController(_:didReceive:)` method in order to listen for the messages (such as `incitoFinishedRendering`).


Then, load the contents of the html doc into your webview:

```
webView.loadHTMLString(htmlStr, baseURL: nil)
```

After the HTML doc is loaded (see `webView(_:didFinish:)` delegate method), it must be initialized with the Incito's JSON string:

```
webView.evaluateJavaScript("window.init(\(incitoJSONString))") { (_, error) in
   …
}
```

Once this succeeds, you will be able to call the various JS functions:

```
// Whatever code you wish to run on the html doc...
let jsCode = "absoluteBoundsOfElementWithId('\(elementId)')"
webView.evaluateJavaScript(jsCode) { (res, err) in
   // `res` is the response of the js code.
   …
}
```
