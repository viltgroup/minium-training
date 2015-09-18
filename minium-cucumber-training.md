# Cucumber Quickstart

### Sample Project

In this section we will try to walk you through on how to create a new
end-to-end testing project using a real-world example.

We'll be using Minium Mail, a sample app that was [adapted from the folks of
flightjs](https://github.com/flightjs/example-app). The application is available
for you to test in
[http://minium.vilt.io/sample-app/](http://minium.vilt.io/sample-app/).

If you just want to jump straight to the finished test project you can check the
complete source code in
[https://github.com/viltgroup/minium-mail-e2e-tests](https://github.com/viltgroup/minium-mail-e2e-tests)

### Writing the first Scenario

As you already know a **scenario** is part of a **feature** file and the
**scenario** itself is made of **steps**.

To illustrate the use of Minium and Cucumber we will create a user story that
tests the functionality of sending emails. The first **scenario** that we will
create is to send an email and verify if the email was really sent.

In this **scenario** the workflow is:

1. **Go to** the page
2. **Click** on New button
3. **Fill** the new email form
4. **Click** on Send button
5. **Go to** section Sent
6. **Verify** is the email exists

The way to write a feature is to describe each action of the user.

```gherkin_en
Feature: Send emails

  Scenario: Send a simple Email
    Given I'm at Minium Mail
    When I click on button "Compose"
    And I fill "Recipients" with "Rui Figueira"
    And I fill "Subject" with "Minium Test"
    And I fill "Message" with "My new Message"
    And I click on button "Send"
    And I navigate to section "Sent"
    Then I should see an email with "Recipients" equals to "Rui Figueira"
    And I should see an email with "Subject" equals to "Minium Test"
    And I should see an email with "Message" equals to "My new Message"
```

## Step definitions

The following **step** represents a click on a button:
```gherkin
When I click on button "Compose"
```

This **step** needs a **step definition** to translate plain text into actions
that will interact with the system.

This **step** will match the step definition below:

```javascript
When(/^I click on button "(.*?)"$/, function(btnLabel) {
  //find a button with some text
  var btn = $("button, .button").withText(btnLabel);

  //click on the button
  btn.click();
});
```

Minium Script must be close to the way we think about what we want to test.

## Fill a form

Now we want to fill the new email form. The steps used to fill the form are the
following:

```gherkin
When I fill the field "Recipients" with "Rui Figueira"
And  I fill the field "Subject" with "Minium Test"
And  I fill the field "Message" with "My new Message"
```

In the figure below we can see the form that we want to fill.

![email-form](images/email-form.png "Email Form")

In the step definition, we need to get all the elements of the form (stored in
variable inputs). Since we can have different types of elements in a form (e.g.
textbox, radio, button, checkbox, option, etc) we need to check the type of
the input before we actually fill the form.

```javascript
When(/^I fill the field "(.*?)" with "(.*?)"$/, function(fldName, value) {
  // get all elements of the form
  var flds = base.find("input, textarea, select");

  // find the element by its label
  var fld = flds.withLabel(fldName);

  // check the type of the element
  if (fld.waitForExistence().is("select")) {
    // if the element is a select box
    fld.select(value);
  } else {
    // if the element is a textbox
    fld.fill(value);
  }
});
```

## Assertions

Now we want to validate if the email that we send was actually sent. For that
we will navigate to the section "Sent" and validate the data.

The steps used to validate if the email was sent are the following:

```gherkin
Then I should see an email with "Recipients" equals to "Rui Figueira"
And I should see an email with "Subject" equals to "Minium Test"
And I should see an email with "Message" equals to "My new Message"
```

In the figure below we can see the table with the sent emails.

![check-message](img/check-message.png "Sent emails list")

The steps above will match the step definition below:

```javascript
Then(/^I should see an email with "(.*?)" equals to "(.*?)"$/, function(field, value) {
	//find the rows of the table
	var rows = base.find("table tr");
	var colName = "." + field;

	// find the element by the colName and with the text value
	var elem = rows.find(colName).withText(value);

	//do the assertion
	expect(elem).to.have.size(1);
});
```


## Deal with loadings, animations and overlays

### Loadings and Animations

When we create a new email, the application has a loading time while the new
email is being created. To perform other actions in the page we need to wait
until the loading disappears.

```javascript
var loading = $(".loading").withCss("display", "block");

// This base expression always returns the scope we're working on: the main
// window unless a modal is visible and a loading is visible
var base = $(":root").unless(loading);
```

For example, many web applications contain asynchronous callbacks. When an
asynchronous callback is made, the page waits for a response from the server
and most of the times waiting icon appears on the page. Before realizing another
action we need to wait for the asynchronous callback to complete, otherwise
the next action can fail. So we need to have a waiting mechanism for this type
of case.

This can be very useful not only when you got **loading animation** on
asynchronous callbacks, but also for other any other animation or overlays.

### Overlays

When a modal appear usually we can only interact with the modal elements.

```javascript
var loading = $(".loading").withCss("display", "block");
```

You can check this first first doing:

```bash
git clone https://github.com/viltgroup/minium-mail-e2e-tests.git
cd minium-mail-e2e-tests
git checkout v1
```

## Autocomplete

If you want to have some help writting your steps, you can add in your application.yml (config/application.yml) of your cucumber project the step's that you are writing. This way you will have some autocompletion features for those step's.

```yaml
minium:
  webdriver:
    desiredCapabilities:
      browserName: chrome
    window:
      maximized: true
  cucumber:
    snippets:
      - name: "Given I'm logged in as ..."
      - name: "When I navigate to \"...\""
      - name: "When I fill the form with:"
        rowsHashKeys:
          - Name
          - Address
      - name: "Then I should see error message \"...\""
      - name: "Given I'm at Minium Mail"
      - name: "Given an email with \"...\" exists"
      - name: "Given an email with \"...\" exists under \"...\""
      - name: "Given an email with \"...\" doesn't exist"
      - name: "Given an email with \"...\" doesn't exist under \"...\""
      - name: "Given I'm at section \"...\""
      - name: "When I click on button \"...\""
      - name: "When I fill \"...\" with \"...\""
      - name: "When I fill:"
      - name: "When I click on the email with:"
      - name: "When I delete an email with Subject \"...\""
      - name: "When I move an email with Subject \"...\" to \"...\""
      - name: "When I navigate to section \"...\""
      - name: "Then I should see an email with:"
      - name: "Then I shouldn't see an email with:"
```

## Refactoring

We can refactor our feature file using tables Tables are useful for specifying
a larger data set and are more readable.

```gherkin
Scenario: Send an Email

  Given I'm at sample app
  When I click on button "New"
  And I fill:
    | Recipients | Rui Figueira   |
    | Subject    | Minium Test    |
    | Message    | My new Message |
  And I click on button "Send"
  Then I navigate to section "Sent"
  And I should see an email with:
    | Recipients | Rui Figueira   |
    | Subject    | Minium Test    |
    | Message    | My new Message |
```

When we are using tables, the step definition needs to be changed. For the
following steps using tables:

```gherkin
When I fill:
  | Recipients | Rui Figueira   |
  | Subject    | Minium Test    |
  | Message    | My new Message |
```

The step definition is now receiving a datatable as argument with the
information of table. What we need to do is iterate over the elements if the
table and fill the form with them.

```javascript
When(/^I fill:$/, function(datatable) {
  var flds = base.find("input, textarea, select");
  // get the values of the datatable creating an object like
  // {"Contact":"Rui Figueira","Subject":"Minium Test","Message":"My new Message"}
  var values = datatable.rowsHash();

  // iterate over the elements of the datatable
  for (var fldName in values) {
    var fld = flds.withLabel(fldName);
    var val = values[prop];

    if (fld.is("select")) {
      fld.select(val);
    } else {
      fld.fill(val);
    }
  }
});
```

You can check the version using tables doing:

```bash
git clone https://github.com/viltgroup/minium-mail-e2e-tests.git
cd minium-mail-e2e-tests
git checkout v2
```




### Modules

With Minium you can also create modules, in order to help you keeping your code
clean, organized and reusable. A module is a standard object literal containing
methods and properties.

For this example we can create a module to fill the form of the email.

```javascript
var forms = {

  fill : function (vals) {
    for (var fldName in vals) {
      var value = vals[fldName];

      var fld = base.find("input, textarea, select").withLabel(fldName);

      if (fld.waitForExistence().is("select")) {
        fld.select(value);
      } else {
        fld.fill(value);
      }
    }
  },

  submit : function () {
    var btn = base.find("submit");

    btn.click();
  }
};

if (typeof module !== 'undefined') module.exports = forms;
```

After creating your module you can simply call it this way:

```javascript
var forms = require("forms");

//  start composing an email
$("#compose").click();

forms.fill({ "Recipients": "Minium Bot" });
forms.submit();
```

So in your **step definition**, you can now use the module.

```javascript

//require the form module
var forms = require("forms");

When(/^I fill:$/, function(datatable) {
  var values = datatable.rowsHash();

  //use the function of the module
  forms.fill(values);
});

```

You can check the version using modules doing:

```bash
git clone https://github.com/viltgroup/minium-mail-e2e-tests.git
cd minium-mail-e2e-tests
git checkout v3
```


### Background

Sometimes you will find yourself repeating the same `Given` steps in all of the scenarios in a feature. 

You can move and group those steps under a Background section before the fisrt scenario. The background runs before each one of your scenarios.

```gherkin
Feature: Delete Emails

  Background: 
    Given I'm at Minium Mail

  Scenario: Delete an email
    Given an email with Subject "Minium Can!" exists
    When I delete an email with Subject "Minium Can!"
    And I navigate to section "Trash"
    Then I should see an email with:
      | Subject | Minium Can! |

  Scenario: Delete an email from trash
    Given I'm at section "Trash"
    And an email with Subject "Phasellus vitae interdum nulla." exists
    When I delete an email with Subject "Phasellus vitae interdum nulla."
    Then I shouldn't see an email with:
      | Subject | Minium Can! |
```



### Scenario Outlines


In the follwoing scenarios, the steps are the same, only the data changes.

```gherkin
Scenario: Send an Email
    When I click on button "Compose"
    And I fill:
      | Recipients | Rui Figueira   |
      | Subject    | Minium Test    |
      | Message    | My new Message |
    And I click on button "Send"
    And I navigate to section "Sent"
    Then I should see an email with:
      | Recipients | Rui Figueira   |
      | Subject    | Minium Test    |
      | Message    | My new Message |

Scenario: Send an Email
    When I click on button "Compose"
    And I fill:
      | Recipients | Mario Lameiras                                         |
      | Subject    | BDD + Minium                                           |
      | Message    | Egestas morbi at. Curabitur aliquet et commodo nonummy |
    And I click on button "Send"
    And I navigate to section "Sent"
    Then I should see an email with:
      | Recipients | Mario Lameiras                                  |
      | Subject    | BDD + Minium                                    |
      | Message    | Egestas morbi at. Curabitur aliquet et commodo  |
```
We can refactor this scenarios using Scenario outlines. Scenario outlines allow us to write more concisely Scenarios through the use of a template with placeholders, using `Examples` with tables and `< >` delimited parameters. The following `Scenario` is the same as the one above.

```gherkin
Scenario Outline: Send simple emails
  When I click on button "Compose"
  And I fill:
    | Recipients | <to>      |
    | Subject    | <subject> |
    | Message    | <message> |
  And I click on button "Send"
  And I navigate to section "Sent"
  Then I should see an email with:
    | Recipients | <to>      |
    | Subject    | <subject> |
    | Message    | <message> |

  Examples: 
    | to             | subject      | message                                        |
    | Rui Figueira   | Minium Test  | My New messages                                |
    | Mario Lameiras | BDD + Minium | Egestas morbi at. Curabitur aliquet et commodo |
```




---

You can check the complete source code in [https://github.com/viltgroup/minium-mail-e2e-tests](https://github.com/viltgroup/minium-mail-e2e-tests)
