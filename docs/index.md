`XMLUtils` is a Dyalog APL namespace containing utilities designed to:

* Convert HTML to XHTML. XHTML is essentially HTML that is rewritten to follow the stricter rules of XML. XHTML can be processed by Dyalog APL's `⎕XML` system function to produce a matrix form of the content. This matrix form lends itself to easier manipulation in APL.
* Manipulate matrix form XML data. There are utilities to search, select, insert, replace, and delete elements in the XML matrix data.

!!! note
    While `XMLUtils` itself is `⎕IO` and `⎕ML` insensitive, the examples in this documentation assume an environment of `(⎕IO ⎕ML)←1`.

## Obtaining `XMLUtils`
`XMLUtils` is a single namespace contained in a single APL source file. There are several ways to obtain `XMLUtils`.

* Released versions are available on [GitHub](https://github.com/bpbecker/XMLUtils/releases).
* `XMLUtils` is also available as a [Tatin](https://tatin.dev/) package: bpbecker-XMLUtils.
* The latest "pre-release" version, if any, can be found in the master branch of [https://github.com/BPBecker/XMLUtils](https://github.com/bpbecker/XMLUtils/blob/master/Source/XMLUtils.apln)
* You can also use the `]get` user command to load `XMLUtils` into your active workspace.

```
      ]get https://github.com/bpbecker/XMLUtils/raw/refs/heads/master/Source/XMLUtils.apln
```
## Motivation
`XMLUtils` started as a simple project in a namespace named `xhtml`. I occasionally found data in web pages that I wanted to import into my workspace for further processing. Because HTML has a "looser" rule set than XHTML, a lot of times I couldn't simply run the HTML through `⎕XML`. And thus was born `HTMLtoXHTML` - a utility that attempts to convert HTML, even malformed HTML, into XHTML and then use `⎕XML` to return the data in matrix form. 

Once I've got the XML matrix, what can I do with it? That's where the other part of `XMLUtils` comes in - to search and select  

For example, let's suppose I wanted to find all the stylesheets that are linked to the Dyalog.com home page. (Note that the examples below reflect the content of the Dyalog home page at the time of this writing - it may change.)

First, I'll use [`HttpCommand`](https://dyalog.github.io/HttpCommand) to grab the HTML for the page
```
      ]load HttpCommand
      html←(HttpCommand.Get 'dyalog.com').Data
      50↑html
<!DOCTYPE html PUBLIC "-//W3C//DTD XHTML 1.0 Trans
```
Next, I'll convert it to XML using `HTMLtoXHTML`
```
      ≢xml←XMLUtils.HTMLtoXHTML html
308
```
Now I'll `Xfind` to find all of the `<link>` elements that have a `rel=stylesheet` attribute
```
      +/matches←xml XMLUtils.Xfind '//link//rel/stylesheet'
4
```
I can use `Xsel` to select those elements.
```
      ≢elements←xml XMLUtils.Xsel matches
4
```
Finally, I can get the URLs for all of the stylesheets using the `href` attribute.
```
      ⍪urls←elements XMLUtils.Xattr 'href'
  https://fonts.googleapis.com/css2?family=Exo+2:wght@400;700&display=swap                   
  //cdn.dyalog.com/css/apl385.css                                                            
  /uploads/css/jquery.fancybox.css                                                           
  https://www.dyalog.com/tmp/cache/stylesheet_combined_46f9f0f9723b06cff2425d65f29f50ab.css  
```
Combining it all...
```
      urls←(xml XMLUtils.Xsel xml XMLUtils.Xfind'//link//rel/stylesheet')XMLUtils.Xattr 'href'
```