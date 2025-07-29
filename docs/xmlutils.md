In addition to HTML conversion, `XMLUtils` has a number of utilities to search and manipulate `⎕XML`'s 5-column matrix format. This page presents only descriptions of the utilities. Examples are found on the [examples page](examples.md).

### `Xfind`

`Xfind` finds XML elements matching criteria you specify. The criteria maps closely to the columns of a `⎕XML` matrix.

|--|--|
|Syntax|`bv ← xml Xfind criteria`|
|`criteria`|A delimited character vector where each segment is a filter to be applied to a column of `xml`. The first character in `criteria` is used as the delimiter. Empty segments are not used in the search. All textual searches are case insensitive.<br/><pre style="font-family:APL;">      specs ← '/level/element/content/attribute/attributeValue/type'</pre>Where:<ul><li>`level` is the nesting level (`xml[;1]`). `level` can be any of<ul><li>a single level number</li><li>a space-delimited list of level numbers</li><li>`n+` means any level greater than or equal to `n`</li><li>`n-` means any level less than or equal to `n`</li><li>`n-m` means any level between `n` and `m` inclusive</li></ul></li><li>`element` are space-delimited element names to match (`xml[;2]`). For example `'th td'` will match all `th` and `td` elements.</li><li>`content` is a string to search for in the element's content (`xml[;3]`)</li><li>`attribute` are space-delimited attribute names to match (column 1 of `xml[;4]`)</li><li>`attributeValue` is string to search for in the attribute values (column 2 of `xml[;4]`)</li><li>`type` is a space-delimited list of `⎕XML` types to match (`xml[;5]`)</li></ul>| 
|`xml`|A 5-column `⎕XML` matrix |
|`bv`|A Boolean vector marking rows in `xml` that match the search criteria in `criteria`|


### `Xselect`
Having found XML elements using `Xfind`, you can use `Xselect` to select them and optionally their immediate descendants.

|--|--|
|Syntax|`elements ← {kids} (xml Xselect) bv`|
|`bv`|A Boolean vector with as many elements as rows in `xml`. `1`'s indicate an element to retrieve.|
|`xml`|A 5-column `⎕XML` matrix |
|`kids`|An optional Boolean scalar indicating whether to in include descendent elements. Valid values are:<ul><li>`0` **do not** include descendents</li><li>`1` (the default) include descendent elements|
|`elements`|A vector containing `+/bv` elements, each of which contains the `xml` element corresponding to a `1` in `bv` and, if `kids` is 1, its immediately following descendant elements.|

### `Xattr`

`Xattr` retrieves the values of all attributes in `xml` that match the attribute name(s) you supply.


|--|--|
|Syntax|`values ← {ic} (xml Xattr) attr`|
|`attr`|A vector of space-separated attribute names, or a vector of such vectors.|
|`xml`|A single 5-column `⎕XML`-style matrix or vector of such matrices|
|`ic`|An optional Boolean indicating whether to perform a case-insensitive comparison on the attribute names. Valid values are:<ul><li>`0` (the default) comparisons **are** case sensitive</li><li>`1` comparisons **are not** case sensitive</li></ul>|
|`values`|The shape of `values` depends on the shapes of `xml` and `attr`.<ul><li>If `xml` is a `⎕XML`-style matrix, `values` will be a vector of character vectors containing the values of matching attributes in `xml`.</li><li>If `xml` is a vector of `⎕XML`-style matrices, `values` will be a vector with as many elements as `xml`, with each vector containing a vector of character vectors containing the values of attributes found in each matrix in `xml`|
|Notes|XML and XHTML attribute names are case sensitive, while HTML attribute names are not.|

### `Xattrval`

`Xattrval` is a shortcut function that essentially does an `Xfind`, `Xselect`, and `Xattr`.

|--|--|
|Syntax|`values ← {ic} (xml Xattrval) criteria`|
|`criteria`|The same as `criteria` described in `Xfind` above.<br/>A delimited character vector where each segment is a filter to be applied to a column of `xml`. Empty segments are not used in the search. All textual searches are case insensitive.<br/><pre style="font-family:APL;">      specs ← '/level/element/content/attribute/attributeValue/type'</pre>Where:<ul><li>`level` is the nesting level (`xml[;1]`). `n+` means any level greater than or equal to `n`. `n-` means any level less than or equal to `n`. `n-m` means any level between `n` and `m` inclusive.</li><li>`element` are space-delimited element names to match (`xml[;2]`). For example `'th td'` will match all `th` and `td` elements.</li><li>`content` is a string to search for in the element's content (`xml[;3]`)</li><li>`attribute` are space-delimited attribute names to match (column 1 of `xml[;4]`)</li><li>`attributeValue` is string to search for in the attribute values (column 2 of `xml[;4]`)</li><li>`type` is a space-delimited list of `⎕XML` types to match (`xml[;5]`)</li></ul>|
|`xml`|A single 5-column `⎕XML` matrix |
|`ic`|An optional Boolean indicating whether to perform a case-insensitive comparison on the attribute names. Valid values are:<ul><li>`0` (the default) comparisons **are** case sensitive</li><li>`1` comparisons **are not** case sensitive</li></ul>|
|`values`|The shape of `values` depends on the shapes of `xml` and `attr`.<ul><li>If `xml` is a `⎕XML`-style matrix, `values` will be a vector of character vectors containing the values of matching attributes in `xml`.</li><li>If `xml` is a vector of `⎕XML`-style matrices, `values` will be a vector with as many elements as `xml`, with each vector containing a vector of character vectors containing the values of attributes found in each matrix in `xml`|
|Notes|XML and XHTML attribute names are case sensitive, while HTML attribute names are not.|

### `Xdelete`

`Xdelete` deletes elements and optionally their descendants from an XML structure

|--|--|
|Syntax|`r ← {kids} (xml Xdelete) criteria`|
|`criteria`|Either the same as `criteria` described in `Xfind` above or a Boolean vector marking the elements to be deleted|
|`xml`|Either a single 5-column `⎕XML` matrix, or a string containing valid XML|
|`kids`|An optional Boolean indicating whether to delete just the elements specified in `criteria` or their descendants as well. Valid values are:<ul><li>`0` do **not** delete the descendant elements</li><li>`1` (the default) delete the descendant elements| 
|`r`|The resulting 5-column `⎕XML` matrix after the elements have been deleted|


### `Xinsert`
`Xinsert` inserts elements after selected elements

|--|--|
|Syntax|`r ← ins (xml Xinsert) elements`|
|`elements`|Either the same as `criteria` described in `Xfind` above or a Boolean vector marking the elements after which the new content will be inserted|
|`xml`|Either a single 5-column `⎕XML` matrix, or a string containing valid XML|
|`ins`|Either a single 5-column `⎕XML` matrix, or a string containing valid XML to be inserted|
|`r`|The resulting 5-column `⎕XML` matrix after the elements have been inserted|

### `Xreplace`
`Xreplace` replaces XML elements and their descendents

|--|--|
|Syntax|`r ← repl (xml Xreplace) elements`|
|`elements`|Either the same as `criteria` described in `Xfind` above or a Boolean vector marking the elements to be replaced|
|`xml`|Either a single 5-column `⎕XML` matrix, or a string containing valid XML|
|`repl`|Either a single 5-column `⎕XML` matrix, or a string containing valid XML to be inserted|
|`r`|The resulting 5-column `⎕XML` matrix after the elements have been inserted|
