## Manipulating XML
For the following examples, we'll be using the following XML...
```APL
      x ← '<a class="top">Hello<b id="foo">World<c class="middle blue">!<b/></c><d/></b><b class="blue">How</b>Are You?</a>'
      xml ← XMLUtils.HTMLtoXHTML x

      ⎕XML xml
<a class="top">
  Hello
  <b id="foo">
    World
    <c class="middle blue">
      !
      <b></b>
    </c>
    <d></d>
  </b>
  <b class="blue">How</b>
  Are You?
</a>

      'Level' 'Element' 'Content' 'Attributes' 'Type'⍪xml
┌─────┬───────┬────────┬───────────────────┬────┐
│Level│Element│Content │Attributes         │Type│
├─────┼───────┼────────┼───────────────────┼────┤
│0    │a      │        │┌─────┬───┐        │7   │
│     │       │        ││class│top│        │    │
│     │       │        │└─────┴───┘        │    │
├─────┼───────┼────────┼───────────────────┼────┤
│1    │       │Hello   │                   │4   │
├─────┼───────┼────────┼───────────────────┼────┤
│1    │b      │        │┌──┬───┐           │7   │
│     │       │        ││id│foo│           │    │
│     │       │        │└──┴───┘           │    │
├─────┼───────┼────────┼───────────────────┼────┤
│2    │       │World   │                   │4   │
├─────┼───────┼────────┼───────────────────┼────┤
│2    │c      │        │┌─────┬───────────┐│7   │
│     │       │        ││class│middle blue││    │
│     │       │        │└─────┴───────────┘│    │
├─────┼───────┼────────┼───────────────────┼────┤
│3    │       │!       │                   │4   │
├─────┼───────┼────────┼───────────────────┼────┤
│3    │b      │        │                   │1   │
├─────┼───────┼────────┼───────────────────┼────┤
│2    │d      │        │                   │1   │
├─────┼───────┼────────┼───────────────────┼────┤
│1    │b      │How     │┌─────┬────┐       │5   │
│     │       │        ││class│blue│       │    │
│     │       │        │└─────┴────┘       │    │
├─────┼───────┼────────┼───────────────────┼────┤
│1    │       │Are You?│                   │4   │
└─────┴───────┴────────┴───────────────────┴────┘

```

### `Xfind`, `Xselect` and `Xattr`
- Find all the `<b>` elements (and their descendents)
```APL
      ⊢bv ← xml XMLUtils.Xfind '//b'
0 0 1 0 0 0 1 0 1 0
      xml XMLUtils.Xselect bv
┌─────────────────────────────────┬────────────┬────────────────────────┐
│┌─┬─┬─────┬───────────────────┬─┐│┌─┬─┬┬───┬─┐│┌─┬─┬───┬────────────┬─┐│
││1│b│     │┌──┬───┐           │7│││3│b││   │1│││1│b│How│┌─────┬────┐│5││
││ │ │     ││id│foo│           │ ││└─┴─┴┴───┴─┘││ │ │   ││class│blue││ ││
││ │ │     │└──┴───┘           │ ││            ││ │ │   │└─────┴────┘│ ││
│├─┼─┼─────┼───────────────────┼─┤│            │└─┴─┴───┴────────────┴─┘│
││2│ │World│                   │4││            │                        │
│├─┼─┼─────┼───────────────────┼─┤│            │                        │
││2│c│     │┌─────┬───────────┐│7││            │                        │
││ │ │     ││class│middle blue││ ││            │                        │
││ │ │     │└─────┴───────────┘│ ││            │                        │
│├─┼─┼─────┼───────────────────┼─┤│            │                        │
││3│ │!    │                   │4││            │                        │
│└─┴─┴─────┴───────────────────┴─┘│            │                        │
└─────────────────────────────────┴────────────┴────────────────────────┘
```
- Find all the `<b>` elements (but not including their descendents)
```APL
      0 (xml XMLUtils.Xselect) bv
┌─────────────────┬────────────┬────────────────────────┐
│┌─┬─┬┬────────┬─┐│┌─┬─┬┬───┬─┐│┌─┬─┬───┬────────────┬─┐│
││1│b││┌──┬───┐│7│││3│b││   │1│││1│b│How│┌─────┬────┐│5││
││ │ │││id│foo││ ││└─┴─┴┴───┴─┘││ │ │   ││class│blue││ ││
││ │ ││└──┴───┘│ ││            ││ │ │   │└─────┴────┘│ ││
│└─┴─┴┴────────┴─┘│            │└─┴─┴───┴────────────┴─┘│
└─────────────────┴────────────┴────────────────────────┘
```
- Find the "class" and "id" attributes for all `<b>` elements, but not their descendents
```APL
      (0 (xml XMLUtils.Xselect) bv) XMLUtils.Xattr 'class id'
┌────────┬───┬─────────┐
│┌┬─────┐│┌┬┐│┌──────┬┐│
│││┌───┐│││││││┌────┐│││
││││foo│││││││││blue││││
│││└───┘│││││││└────┘│││
│└┴─────┘│└┴┘│└──────┴┘│
└────────┴───┴─────────┘
```

### `HTMLtoXHTML` and `Xattrval`
Find all of the linked resources on a web page. Linked resources and pages use the `href` attribute while images use the `src` attribute. `HTMLtoXHTML2` could also be used instead of `HTMLtoXHTML`.

```APL
⍝ get the HTML content of the page
      ]load HttpCommand
      html ← (HttpCommand.Get 'dyalog.com').Data

⍝ convert to XML (XHTML) (⍝ you could also use HTMLtoXHTML2) 
      ⍴ xml ← XMLUtils.HTMLtoXHTML html  
308 5

⍝ find all href and src attribute values
      ≢¨ linked ← xml XMLUtils.Xattrval '////href src'
95 18
```
There are 95 `href` and 18 `src` values.

### `Xinsert`
Insert a `<b>` element after the `<a>` element.

```APL
      '<b/>'('<a><c/></a>'XMLUtils.Xinsert)'//a'
┌─┬─┬┬───┬─┐
│0│a││   │3│
├─┼─┼┼───┼─┤
│1│b││   │1│
├─┼─┼┼───┼─┤
│1│c││   │1│
└─┴─┴┴───┴─┘
```

### `Xdelete`
Using `xml` from above, delete all `<c>` and `<d>` elements and their descendants:
```APL
      xml XMLUtils.Xdelete xml XMLUtils.Xfind '//c d'
┌─┬─┬────────┬────────────┬─┐
│0│a│        │┌─────┬───┐ │7│
│ │ │        ││class│top│ │ │
│ │ │        │└─────┴───┘ │ │
├─┼─┼────────┼────────────┼─┤
│1│ │Hello   │            │4│
├─┼─┼────────┼────────────┼─┤
│1│b│        │┌──┬───┐    │7│
│ │ │        ││id│foo│    │ │
│ │ │        │└──┴───┘    │ │
├─┼─┼────────┼────────────┼─┤
│2│ │World   │            │4│
├─┼─┼────────┼────────────┼─┤
│1│b│How     │┌─────┬────┐│5│
│ │ │        ││class│blue││ │
│ │ │        │└─────┴────┘│ │
├─┼─┼────────┼────────────┼─┤
│1│ │Are You?│            │4│
└─┴─┴────────┴────────────┴─┘
```
Now, do the same, but do not delete the descendants:

```APL
      0 (xml XMLUtils.Xdelete) xml XMLUtils.Xfind '//c d'
┌─┬─┬────────┬────────────┬─┐
│0│a│        │┌─────┬───┐ │7│
│ │ │        ││class│top│ │ │
│ │ │        │└─────┴───┘ │ │
├─┼─┼────────┼────────────┼─┤
│1│ │Hello   │            │4│
├─┼─┼────────┼────────────┼─┤
│1│b│        │┌──┬───┐    │7│
│ │ │        ││id│foo│    │ │
│ │ │        │└──┴───┘    │ │
├─┼─┼────────┼────────────┼─┤
│2│ │World   │            │4│
├─┼─┼────────┼────────────┼─┤
│3│ │!       │            │4│
├─┼─┼────────┼────────────┼─┤
│3│b│        │            │1│
├─┼─┼────────┼────────────┼─┤
│1│b│How     │┌─────┬────┐│5│
│ │ │        ││class│blue││ │
│ │ │        │└─────┴────┘│ │
├─┼─┼────────┼────────────┼─┤
│1│ │Are You?│            │4│
└─┴─┴────────┴────────────┴─┘
```

### `Xreplace`
Replace the element with `id="foo"` (and its descendants) with new elements.

```APL
      '<new>This is new</new>' (xml XMLUtils.Xreplace) xml XMLUtils.Xfind '////id/foo'
┌─┬───┬───────────┬────────────┬─┐
│0│a  │           │┌─────┬───┐ │7│
│ │   │           ││class│top│ │ │
│ │   │           │└─────┴───┘ │ │
├─┼───┼───────────┼────────────┼─┤
│1│   │Hello      │            │4│
├─┼───┼───────────┼────────────┼─┤
│1│new│This is new│            │5│
├─┼───┼───────────┼────────────┼─┤
│1│b  │How        │┌─────┬────┐│5│
│ │   │           ││class│blue││ │
│ │   │           │└─────┴────┘│ │
├─┼───┼───────────┼────────────┼─┤
│1│   │Are You?   │            │4│
└─┴───┴───────────┴────────────┴─┘
```