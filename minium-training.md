## Javascript Editor

### Scope

When we evaluate some javascript in Minium Developer, all variables declared
we'll be maintained in the evaluation global scope. That means that, the next
time you evaluate something, all previously declared variables are still
available.

For instance, evaluate the following code:

```javascript
var name = "World";
```

Now you can evaluate the following code:

```javascript
"Hello " + name // it evaluate into "Hello world"
```

If you want to clear the global scope, so that all variables are removed from
it, you can do it by clicking in `Run > Clean scope`:

![Clean scope](images/clean-scope.png)

After that, if you try to run the previous code, it will fail:

```javascript
"Hello " + name // ReferenceError: "name" is not defined
```

## Configuration values

In `config/application.yml`, you can find all your Minium configuration (for
instance, which default browser to use, browser window dimensions, etc.).

There is, however, a special configuration block under `minium.config`: that
is configuration you can add and use in your application. In your javascript
code, you can access all its properties under the variable `config`.

**Exercise:**

Edit the `application.yml` and change it to have a property `name` with value
`World`:

```yaml
minium:
  webdriver:
   ...
  config:
    name: World
```

That `name` property is now under the variable `config`, so you can evaluate the
following code:

```javascript
"Hello " + config.name // it evaluate into "Hello World"
```

If you change the name to `Minium`, for instance:

```yaml
minium:
  webdriver:
   ...
  config:
    name: Minium
```

then re-evaluating `"Hello " + config.name` will return `Hello
Minium`.

You can even add complex configuration there:

```yaml
minium:
  webdriver:
   ...
  config:
    users:
      administrator:
        username: admin
        password: strong_password
    fruits:
      - banana
      - orange
      - strawberry
```

and then access it:

```javascript
console.log(config.users['administrator'].username); // prints "admin"

config.fruits.forEach(function (fruit) { console.log(fruit) }) // prints:
// banana
// orange
// strawberry
```

# Developing Minium with Minium Developer

## Selector Gadget

Minium comes with [Selector Gadget](https://github.com/cantino/selectorgadget).
It allows developers to pick elements in the browser. You can either trigger
that functionality with `Ctrl + Shift + C` or by pressing the button
`Selector Gadget`:

![Selector Gadget](images/selector-gadget.png)

After pressing it, you'll notice that, in the browser for the web driver you're
running, a new toolbar at the bottom of the page will be displayed:

![Selector Gadget Toolbar](images/selector-gadget-toolbar.png)

Besides, you can now start picking elements in the page: just click on a page
element that you would like your selector to match (it will turn green).
Selector Gadget will then generate a minimal CSS selector for that element,
and will highlight (yellow) everything that is matched by the selector. You can
then click on a highlighted element to reject it (red), or click on an
unhighlighted element to add it (green).

Holding 'shift' while moving the mouse will let you select elements inside of
other selected elements.

**Exercise**:

Let's try to select the `Tags` table header cell.

- Ensure your web driver browser is at Minium Mail sample app:

```javascript
browser.get("http://minium.vilt.io/sample-app/");
```

- Position your cursor where you want to insert the generated code
- Start Selector Gadget
- On the web driver browser, click on `Tags` table header cell:

![Table headers](images/table-headers.png)

- You'll notice that the cell you clicked became green, and other table
headers cells became yellow. That means that the CSS selector it generated
also matches those yellow cells
- Now click any of those yellow cells to reject it (for instance, `Subject`)

![Table headers with rejection](images/table-headers-exclusion.png)

- You'll notice that cell became red, and cells that were yellow are no longer
selected. That's because, by rejecting the `Subject` cell, SelectorGagdet tryed
to get a CSS selector that kept matching the green elements but excluded red
elements, and that new CSS selector no longer matches the other yellow cells.
- Now that only one cell is matching, and it is the one we want, we can accept
that CSS selector by accepting it in Minium Developer:

![Selector gadget](images/selector-gadget-accept.png)

- You'll now notice that Minium Developer inserted the following Minium
expression in your editor, at the place where your cursor was:

```javascript
$("th:nth-child(2)")
```

- You can now evaluate it, by ensuring your cursor is at that line, and by
pressing `Ctrl + Enter`

**Exercise**:

Try to use Selector Gadget as much as possible to write instructions for sending
an email. You can use interaction methods, like `.click()`, `.fill(text)`,
`.select(option)` (for select fields), and even use filter methods, like
`.withText(text)` (it will only return matching elements that have that exact
text).

## Select cells in tables

Selecting cells in tables based on their values and columns can be hard. Minium
has some functions that ease that process, as we will see.
So, first things first: let's ensure our browser is at Minium Mail Inbox:

```
browser.get("http://minium.vilt.io/sample-app/#/folders/inbox");
```

Then let's get variables to identify both table headers and table value cells:

```javascript
var headers = $("#mail-list th")
var cells = $("#mail-list td");
```

```javascript
var recipientsHeader = headers.withText("Recipients");
var recipientsCells = cells.below(recipientsHeader);
```

![Table recipients cells](images/table-recipients.png)

We can now filter cells with "Minium Bot" on it, for instance:

```javascript
var recipientCell = recipientsCells.withText("Minium Bot")
```
![Table recipients cell with Minium Bot](images/table-recipients-miniumbot.png)

Now it's easy to click the checkbox of that row:

```javascript
var itemCheckbox = $(":checkbox").leftOf(recipientsCell);
itemCheckbox.click(); // this will toggle the checkbox
```

## JQuery methods

## Base Expression pattern

The concept behing the Base Elements expression is that it should represent the
root elements of the UI that can be interacted with.

For instance, when Modal elements are displayed (for instance, a bootstrap
modal dialog) we want `base` to evaluate to that modal element, therefore
excluding all elements that are behind the backdrop element.

Then, by using base as the root of our elements expressions, we can get some
assurance that we are getting the right elements instead of getting elements
that are not interactable (or should not be interactable) at that point.

To demonstrate how useful this pattern can be, let's do a simple exercise:
we'll start composing an email and then we'll try to click the first button it
finds:

```javascript
browser.get("http://minium.vilt.io/sample-app/");

// this will open the New message modal dialog
$("#compose").click();

// let's try to click the first available button
$("button").click();
```

If we try to evaluate that code, it will fail. That's because
`$("button").click()` will try to click the first matching button (which is the
`Compose` button), and that one is under the modal backdrop and for that reason,
it is not accessible for interactions.

You can try to evaluate `$("button")` and you'll see that lots of buttons in
the page will highlight, both the ones that are in the modal dialog and the ones
in the main page, that are not accessible due to the modal backdrop:

![Buttons](images/buttons.png)

So, let's consider the following base expression:

```javascript
base = $(":root").unless(".modal-backdrop").add(".modal-dialog");
```

Let's

- the first part, `$(":root").unless(".modal-backdrop")`, evaluates the page
  root element when no modal backdrop exist (that is, when no modal dialog is
  open)
- the second part, `.add(".modal-dialog")`, adds modal dialog elements. Note
  that, when a modal dialog is available, this second part evaluates into that
  element, but the first part will evaluate into an empty set because of the
  existence of the modal backdrop

So, basically that expression evaluates into the root element or into an opened
modal dialog, but never both at the same time. If we try to evaluate `base`
when a modal dialog is opened:

![Base expression evaluation when modal dialog is being displayed](images/base-modaldialog.png)

So, if we use `base` as our "root" for finding elements in the page, we can now
restrict them to accessible ones.

Try to evaluate the following expression with the modal dialog open now:

```javascript
base.find("button")
```

You'll see that only buttons inside the modal dialog were highlighted:

![Buttons with base expression](images/buttons-base.png)

And now the following code will evaluate successfully:

```javascript
browser.get("http://minium.vilt.io/sample-app/");

var base = $(":root").unless(".modal-backdrop").add(".modal-dialog");

// this will open the New message modal dialog
base.find("#compose").click();

// let's try to click the first available button
base.find("button").click();
```

## Waiting interactions and presets

Sometimes, it is necessary to wait that some element is displayed on the page
or not. For instance, a spinning wheel is often displayed to indicate that the
application is doing something in background, and therefore you should wait
until it disappears.

In Minium Mail sample app, it shows a spinning wheel after you perform some
operation.

Let's try to delete an email item and then compose another one:

```javascript
browser.get("http://minium.vilt.io/sample-app/");

var mailItemCheckbox = $(":checkbox");
var removeBtn = $("#remove-action");
var composeBtn = $("#compose");

mailItemCheckbox.click();
removeBtn.click();
composeBtn.click();
```

If you run that script all at once (select it all and press `Ctrl + Enter`),
you'll notice it will fail when trying to click the `Compose` button. The
reason is that the spinning wheel is being displayed and it "blocks" elements
behing the backdrop form being interacted with. So, we need to wait for that
spinning wheel to disappear before we can click the `Compose` button. We can do
that with the `.waitForUnexistence()` method:

```javascript
browser.get("http://minium.vilt.io/sample-app/");

var loading = $(".loading").withCss("display", "block");
var mailItemCheckbox = $(":checkbox");
var removeBtn = $("#remove-action");
var composeBtn = $("#compose");

mailItemCheckbox.click();
removeBtn.click();
loading.waitForUnexistence();
composeBtn.click();
```

The interaction `loading.waitForUnexistence()` will wait at most for a specified
amount of time (by default, 5 seconds) that the element doesn't exist. After
that time, it will fail, otherwise, as soon the element disappears, it will
proceed.

However, it is possible that

```javascript
// browser configuration
browser.configure()
  .waitingPreset("fast")
    .timeout(1, timeUnits.SECONDS)
  .done()
  .waitingPreset("slow")
    .timeout(10, timeUnits.SECONDS)
  .done();

var loading = $(".loading").withCss("display", "block");

loading.waitForUnexistence("slow");
```

## Interaction Listeners

### ensureExistence / ensureUnexistence

```javascript
var timeUnits = require("minium/timeunits");

var loading = $(".loading").withCss("display", "block");

var loadingUnexistenceListener = minium.interactionListeners
  .ensureUnexistence(loading);

var timeoutListener = minium.interactionListeners
  .onTimeout()
  .when(loading)
  .waitForUnexistence(loading)
  .withWaitingPreset("slow")
  .thenRetry();

// browser configuration
browser.configure()
  .waitingPreset("fast")
    .timeout(1, timeUnits.SECONDS)
  .done()
  .waitingPreset("slow")
    .timeout(10, timeUnits.SECONDS)
  .done()
  .interactionListeners()
    .add(loadingUnexistenceListener)
    .add(timeoutListener)
  .done();
```

# Files and Modules
