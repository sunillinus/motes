#CSS Fundamentals

##3 ways to include CSS in HTML:

1. in line with style attribure:
```html
<h1 style="color: fff;">
```

2. In the head via style tags
```html
<head>
  <style>
    h1 {
      color: #fff;
    }
  </style
</head>
```

3. Linking external css file in the head:

```html
<head>
  <link rel="stylesheet" href="styles.css"/>
</head>
```

##3 Primary DOM Selectors
1. Element selector: h1{}
2. Class selector: .header{}
3. ID selector: #id {}

### Compound Selectors:
```css
/* no space between the selectors*/
h1.header {}, .content.home {}, .content#my_id {}
```

## Cascade Order
In the order of increasing priority:
1. External link
2. style tag in the head
3. Inline style attribute
4. !important /* bad */
```css
header {
  background: #e0e2e6 !important;
}
```

If the same selector and CSS property is present in two places (of same kind),
the 2nd one takes precedence.

## Float
- Float takes a DOM element and removes it from the normal document flow
and pushes it to the specified edge of the **parent** element. The other content
of the parent element wraps around it.

- When there is lack of space (in the parent element) the last element (in the HTML)
will break down.

### Clearing Floats
When an element is floated, it is removed from teh normal document flow, which
means, the parent element will not strech it's height to accomdate the floated
items. So you have to force the parent to "clear" the float. 3 ways to do it:

1. Clear {clear: both;} the subsequent element of the parent container with the
floated elements. This won't fix the background/border of the parent with
floated elements.

2. With manual clear - add an empty div with {clear: both;} set as a sibling of
the floated elements (inside the parent container).
This will fix the background/border but requires adding an extra HTML element.

3. Clearfix - Add the 'group' class to the parent container.

```css
.group:before,
.group:after {
  content: "";
  display: table;
}
.group:after {
  clear: both;
}
.group {
  zoom: 1; /* IE6&7 */
}

```

## Inehritance and Specificity

Nested elements automatically inherit parent styles (unless overwritten).

### Nested Selectors
```css
/* space between selectors */
.featured p{}, .content #id{}
```

## Priority of Selectors

When there is a conflict in a CSS property value, the priority of the selectors
determine which value is applied. Priority of the selectors is determined by the
specificity score -

<inline style>, <# of id selectors>, <# of class selectors>, <# of element selectors>

**!important** trumps all, so don't use unless absolutely necessary. It will override
cascade order and will be a maintenance nightmare.

.intro p.article    => 0, 0, 2, 1
.inro ul li.active  => 0, 0, 2, 2
#header             => 0, 1, 0, 0
