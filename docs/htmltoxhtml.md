HTML follows a much "looser" set of rules than XHTML. For instance:

* **Optional Tags**: You can omit closing tags like `</li>`, `</p>`, or even `</html>` and `</body>` and browsers will still render the page correctly.
* **Unquoted Attributes**: HTML allows attributes without quotes if the value doesn’t contain spaces or special characters.
* **Case Insensitivity**: Tags and attributes aren’t case-sensitive — `<BODY>` and `<body>` are treated the same.
* **"Tag Soup" Tolerance**: Browsers use error-correcting algorithms to interpret malformed markup, often guessing what the author intended.

HTML to XHTML conversion may not "round trip". This means that if you apply `⎕XML` to the XML result of the conversion, the resulting XHTML might not map identically to the original HTML.

!!! Note
    Be mindful that very messed up HTML may not be able to be parsed using either `HTMLtoXHTML` or `HTMLtoXHTML2`.<br/>For instance `'<a> <b </a>'` will cause an error using either function.

### `HTMLtoXHTML`
`HTMLtoXHTML` uses `⎕XML` to parse HTML. When `⎕XML` trips over a non-compliant element `HTMLtoXHTML` parses the error message and takes corrective measures and tries again. This continues until either the conversion completes or we encounter a situation that we couldn't handle. If you run into the latter, please log a [GitHub issue](https://github.com/bpbecker/xmlutils/issues). 

|--|--|
|Syntax|`xml ← {verbose} HTMLtoXHTML html`|
|`html`|A character vector representing the HTML to be converted|
|`verbose`|(optional) A Boolean where a value of `1` indicates to display the error messages `⎕XML` generates during processing|
|`xml`|The `⎕XML` 5-column matrix result representing the converted HTML|
|Examples|<pre style="font-family:APL;">       HTMLtoXHTML '&lt;h2 id="foo">Hello&lt;/h2>'<br/>┌─┬──┬─────┬────────┬─┐<br/>│0│h2│Hello│┌──┬───┐│5│<br/>│ │  │     ││id│foo││ │<br/>│ │  │     │└──┴───┘│ │<br/>└─┴──┴─────┴────────┴─┘</pre>|


### `HTMLtoXHTML2`
`HTMLtoXHTML2` uses `HTMLRenderer` and the JavaScript XMLSerializer object to generate its XHTML result by:

* creating an invisible HTMLRenderer with the supplied HTML argument
* injecting a WebSocket to communicate back to APL
* executing the JavaScript to parse the HTML
* sending the result back to APL over the WebSocket
* finally closing the `HTMLRenderer` 

|--|--|
|Syntax|`xml ← HTMLtoXHTML2 html`|
|`html`|A character vector representing the HTML to be converted|
|`xml`|The `⎕XML` 5-column matrix result representing the converted HTML|
|Examples|<pre style="font-family:APL;">       HTMLtoXHTML '&lt;h2 id="foo">Hello&lt;/h2>'<br/>┌─┬────┬─────┬────────────────────────────────────┬─┐<br/>│0│html│     │┌─────┬────────────────────────────┐│3│<br/>│ │    │     ││xmlns│http://www.w3.org/1999/xhtml││ │<br/>│ │    │     │└─────┴────────────────────────────┘│ │<br/>├─┼────┼─────┼────────────────────────────────────┼─┤<br/>│1│head│     │                                    │1│<br/>├─┼────┼─────┼────────────────────────────────────┼─┤<br/>│1│body│     │                                    │3│<br/>├─┼────┼─────┼────────────────────────────────────┼─┤<br/>│2│h2  │Hello│┌──┬───┐                            │5│<br/>│ │    │     ││id│foo│                            │ │<br/>│ │    │     │└──┴───┘                            │ │<br/>└─┴────┴─────┴────────────────────────────────────┴─┘</pre>|

### Differences between `HTMLtoXHTML` and `HTMLtoXHTML2`
As you can see from the examples above, `HTMLtoXHTML` and `HTMLtoXHTML2` return somewhat different results. In general, the core HTML that is converted will be structurally the same.

* `HTMLtoXHTML2` requires `HTMLRenderer` which might not be installed or enabled.
* `HTMLtoXHTML2` adds `<html>`, `<head>` and `<body>` elements if they don't exist in the source HTML. This may produce different level numbers from `HTMLtoXHTML`.
* `HTMLtoXHTML` and `HTMLtoXHTML2` may return slightly different results when encountering "sloppy" or malformed HTML.

```APL
      XMLUtils.HTMLtoXHTML '<p><span>hello</p>there</span>world'
┌─┬────┬──────────┬───┬─┐
│0│p   │          │   │7│
├─┼────┼──────────┼───┼─┤
│1│span│hello     │   │5│
├─┼────┼──────────┼───┼─┤
│1│    │thereworld│   │4│
└─┴────┴──────────┴───┴─┘
      XMLUtils.HTMLtoXHTML2 '<p><span>hello</p>there</span>world'
┌─┬────┬──────────┬────────────────────────────────────┬─┐
│0│html│          │┌─────┬────────────────────────────┐│3│
│ │    │          ││xmlns│http://www.w3.org/1999/xhtml││ │
│ │    │          │└─────┴────────────────────────────┘│ │
├─┼────┼──────────┼────────────────────────────────────┼─┤
│1│head│          │                                    │1│
├─┼────┼──────────┼────────────────────────────────────┼─┤
│1│body│          │                                    │7│
├─┼────┼──────────┼────────────────────────────────────┼─┤
│2│p   │          │                                    │3│
├─┼────┼──────────┼────────────────────────────────────┼─┤
│3│span│hello     │                                    │5│
├─┼────┼──────────┼────────────────────────────────────┼─┤
│2│    │thereworld│                                    │4│
└─┴────┴──────────┴────────────────────────────────────┴─┘
```