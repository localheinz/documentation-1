# Templado Documentation

A pragmatic approach to templating for PHP 7.2+

[![Scrutinizer Code Quality](https://scrutinizer-ci.com/g/templado/engine/badges/quality-score.png?b=master)](https://scrutinizer-ci.com/g/templado/engine/?branch=master)
[![Code Coverage](https://scrutinizer-ci.com/g/templado/engine/badges/coverage.png?b=master)](https://scrutinizer-ci.com/g/templado/engine/?branch=master)
[![Build Status](https://scrutinizer-ci.com/g/templado/engine/badges/build.png?b=master)](https://scrutinizer-ci.com/g/templado/engine/build-status/master)

[![Build Status](https://travis-ci.org/templado/engine.svg?branch=master)](https://travis-ci.org/templado/engine)

[![SensioLabsInsight](https://insight.sensiolabs.com/projects/f5b424bf-2a00-4df8-8070-db7f0d03e4e9/mini.png)](https://insight.sensiolabs.com/projects/f5b424bf-2a00-4df8-8070-db7f0d03e4e9)


### Motivation

Most of today's templating engines mix code for the required rendering logic with HTML markup in one file and require
the developers to learn their respective language.

Templado follows a different approach on templating: Being in part inspired by [Tempan](https://github.com/watoki/tempan),
Templado relies solely on plain HTML markup. The limited amount of display logic required is contained with the engine
and triggered by the view model when it's applied to the Page.

### Always ready to preview

As a Templado template is plain HTML, previewing is as easy as opening the HTML file with a browser - example data can
and should be included as the engine will clean it up based on the view model upon rendering.

### No markup duplication
 
Templado features asset support, mapping a list of assets based on their ID into a given HTML Page. To automate this
process [Templado CLI](https://github.com/theseer/templado-cli) can be used. Combined with a File watcher in your IDE,
you can have an always up-to-date set of HTML pages without ever writing a block twice.

### Form handling included

To make form handling even more easy, Templado comes with explicit HTML Form support. Based on supplied Input data,
Templado will repopulate the HTML form and even include your CSRF protection code.

### Custom transformations and Filters

Templado allows for custom transformations, like adding a class to every ```a``` tag and string based replacements upon
serialization.

## Installation

**Note that PHP 7.2+ is required** 

The preferred method for installing Templado is to simply add it to your project using Composer. 

If you are manually creating or editing the composer.json file:

```json
"require" : {
    "templado/engine": "^3.0"
}
```

Or from the command line:

```bash
# composer require templado/engine:^3.0
```

For basic usage, this is all that is necessary. Templado makes use of View Model objects in order to make the magic 
happen. In addition, you have the option of using either JSON, or XML (*More on this later*). In this case you will also 
need to install the Mappers, which are in a separate repository.  


```json
"require" : {
    "templado/engine": "^3.0",
    "templado/mappers": "^1.0"
}
```

or from the command line:

```bash
# composer require templado/mappers:^1.0
```


## Getting Started

As with any templating engine, the goal is to pair a template with a data source. Templado uses View Models and XHTML 
templates to accomplish this. So the first thing we need to do is introduce them to each other.

### Associate A Template With A View Model

Here is an example of code to pair a data model with a template:

```php
try {
    $page = Templado\Engine\Templado::loadHtmlFile(
        new Templado\Engine\FileName(__DIR__ . '/html/viewmodel.xhtml')
    );
    
    $page->applyViewModel(new ViewModel());

    echo $page->asString() . "\n";

} catch (Templado\Engine\TempladoException $e) {
    foreach($e->getErrorList() as $error) {
        echo (string)$error;
    }
}
```

Let's quickly parse this code. 

First, we instantiate a Templado\Engine\Html object. To accomplish this we use the static method 
Templado\Engine\Templado::loadHtmlFile. *Notice that we create a new Templado FileName object to pass into the method.* 

Once loaded, we call the "applyViewModel" method, and pass in our View Model. 

This Object does not have to be called "ViewModel". The name is arbitrary. It is called that here for clarity.

Now that the pairing is complete, you can simply call the "asString" method to output the rendered file. 

Finally, we wrap it in a try/catch, and that's it. With this you are ready to begin templating.

## Basic Usage

One of the great features of Templado is that it relies entirely on HTML markup, with no need for you to learn any 
new language or syntax. Templado has the ability to modify all aspects of your html template. Let's start with basic 
markup elements.

### Elements and Attributes

Templado makes use of four standard attributes in your markup elements ("property", "resource", "prefix", and "typeOf"). 
If you are not familiar, these attributes are all standard recommendations from the [RFDa](https://en.wikipedia.org/wiki/RDFa) 
and are a means of incorporating rich metadata into documents. We will cover each of these, but let's start with the 
"property" attribute since this is likely the most common usage. 

Let's say that you have a header tag in your template. 

```html
<h1>Some Title</h1>
```

Templado looks at each element to see if it has a "property" attribute. If it finds one, it looks for a method in 
your View Model who's name matches the attribute's value.

```html
<h1 property="headline">Some Title</h1>
```

In this example, Templado would access the View Model for this template, and look for a method named "headline".

```php
class ViewModel {
    public function headline() {
        return "The Actual Title";
    }
}
```

Continuing with this example, the method is returning a string, and so Templado would use the returned string as the 
text value for the header:

```html
<h1 property="headline">The Actual Title</h1>
```

It's that simple, and this will work for any HTML element. However it gets much better. Instead of returning a string, 
your View Model method can return an Object representing the HTML element. This Object can have methods that represent 
any of the attributes of your HTML element ...

```html
<h1 property="headline" title="A Title" class="a-class">Some Title</h1>
```

In this example our header now has two attributes - "title" and "class". So we can create a class that looks like this:

```php
class Headline {
    public function title() {
        return "Awsome Title";
    }
    
    public function class() {
        return "cool-class";
    }
    
    public function asString() {
        return "The Actual Title";
    }
}
```

And then return a new instance of our Headline object from the View Model method.

```php
class ViewModel {
    public function headline() {
        return new Headline();
    }
}
```

When an Object is returned Templado will look for methods in the Object that match the names of the attributes of the 
element in the template. 

**Note that Templado looks for a method specifically called "asString" for the text value of the element.**

Now our header will render like this:

```html
<h1 property="headline" title="Awsome Title" class="cool-class">The Actual Title</h1>
```

In addition, if for any reason you have an attribute in your template, but you want to remove it for this data model, 
you can simply return false from the method. So if our Headline object returned false for the class method, our rendered 
header would then look like this:

```html
<h1 property="headline" title="Awsome Title">The Actual Title</h1>
```

Pretty cool, but there is one more feature to discuss about elements and attributes.

Templado captures the values of elements and attributes from the template. And if a method in your Object takes a 
parameter, then the value is passed in. 

So let's say your Object looks like this now: 

```php
class Headline {
    public function title() {
        return "Awsome Title";
    }
    
    public function class($original) {
        return $original . " cool-class";
    }
    
    public function asString() {
        return "The Actual Title";
    }
}
```

You see that we have added a parameter called $original (*the parameter name is up to you*), and we then concatenate the 
the class from the template with our new class, and return that. 

So now our header would render like this:

```html
<h1 property="headline" title="Awsome Title" class="a-class cool-class">The Actual Title</h1>
```

Again this will work with all elements and attributes. 

The fact is that you could stop reading right here, and with only these few features accomplish many of your templating 
goals! You can create an Object to represent any HTML element and all of its attributes. This is cool, but keep reading. 
There is much more. 

### Dynamic Lists

One special case/feature of Templado is that a View Model method can return an array. This allows for multiples of an 
element to be rendered consecutively with different data for each. The most obvious usage for this is with lists. 

Let's say that you need an unordered list of items displayed. When you are creating your template, you have no idea 
of the number of items that will need to be shown. This is not an issue. 

In your template you can create the unordered list with a single list item element, and give it a "property" attribute:

```html
<ul>
    <li property="items">Item 1</li>
</ul> 
```

Then in your View Model, the "items" method can return an array of list items:

```php
class ViewModel {
    public function items() {
        return [
            "Item 1", 
            "Item 2",
            "Item 3" 
        ];
    }
}
```

In this case the rendered elements would like this:

```html
<ul>
    <li property="items">Item 1</li>
    <li property="items">Item 2</li>
    <li property="items">Item 3</li>
</ul> 
```

Notice that in this example we returned an array of strings, so Templado used the strings as the text for each list item.
Just as with our previous element examples, you can also return an array of Objects. These Objects will represent each 
list item element ... just as before.

In the template:

```html
<ul>
    <li property="items" class="odd">Item 1</li>
</ul> 
```

In the View Model:

```php
class ViewModel {
    public function items() {
        return [
            new Item("odd", "Item 1"), 
            new Item("even", "Item 2"),
            new Item("odd", "Item 3")
        ];
    }
}
```

And of course you would need an Item class:

```php
class Item {
    /**
     * @var string
     */
    private $class;
    
    /**
     * @var string
     */
    private $value;
    
    public function __construct($class, $value) {
        $this->$class = $class;
        $this->value = $value;
    }
    
    public function class() {
        return $this->class;
    }
    
    public function asString() {
        return $this->value;
    }
}
```

And now the rendered elements would like this:

```html
<ul>
    <li property="items" class="odd">Item 1</li>
    <li property="items" class="even">Item 2</li>
    <li property="items" class="odd">Item 3</li>
</ul> 
```

One of the nicest features about Templado is that your template files can be rendered in a browser without the 
associated data. You can see how your template looks without concern over what data will be passed to it. 

Given this, there may be times that you want your template to look more fleshed out. For example, even though you don't 
know exactly how many items will be displayed, you are sure that in most cases it will be several. Let's say you expect 
an average of about 5 items. The final page will look very different than your raw template if the template only has one 
item as a placeholder.

This is not a problem. With Templado, you can add as many elements as you deem necessary in the template. 

So your template can look like this:

```html
<ul>
    <li property="items" class="odd">Item 1</li>
    <li property="items" class="even">Item 2</li>
    <li property="items" class="odd">Item 3</li>
    <li property="items" class="even">Item 4</li>
    <li property="items" class="odd">Item 5</li>
</ul> 
```

But your rendered elements will still look like this if we use the View Model from before:

```html
<ul>
    <li property="items" class="odd">Item 1</li>
    <li property="items" class="even">Item 2</li>
    <li property="items" class="odd">Item 3</li>
</ul> 
```

If your View Model method returns ten items, then the rendered list will have ten items. 

The bottom line is that you can design your templates to look realistic, and still know that they will render dynamically. 
This is a nice advantage over other templating engines.


### Nesting

It should be relatively obvious at this point, but to be explicit, just as HTML can have many levels of nesting, so can 
your View Models.

Take this template as an example:

```html
<div property="user">
    <p>Name: <span property="name">Original Name</span></p>
    <div>
        <span>EMail:</span>
        <ul>
            <li property="emailLinks">
                <a property="email" href="mailto:original@domain.tld" class="current">original@domain.tld</a>
            </li>
        </ul>
    </div>
</div>
```
Here the top level element has a property called "user". Within the scope of the "user", we have two properties - "name" 
and "emailLinks". And within the emailLinks scope we have a property called "email". So from a property point of view we 
have 3 levels of nesting. 

So our View Model will follow this schema as follows:

```php
class ViewModel {
    public function user() {
        return new User();
    }
}
```

And the User:

```php
class User {

    public function name() {
        return 'Willi Wichtig';
    }

    public function emailLinks() {
        return [
            new EMailLink(
                new Email('willi@wichtig.de')
            ),
            new EMailLink(
                new Email('second@secondis.de')
            )
        ];
    }

}
```

When the scope changes in the template, it follows in the Models. So now Templado is looking at the User Model for the 
"name" and "emailLinks" methods. We see that the "emailLinks" method is returning an array of EmailLink objects.
So each EmailLink is rendered in the list, and then Templado looks to each of the EmailLink objects to resolve the 
email property. And our EmailLink object looks like this:

```php
class EMailLink {

    /** @var  Email */
    private $email;

    /**
     * @param Email $email
     */
    public function __construct(Email $email) {
        $this->email = $email;
    }

    public function email() {
        return $this->email;
    }
}
```

Finally, our Email object corresponds to an anchor element in our template. So Templado is looking for methods which 
match the attributes of the anchor:

```php
class Email {

    /** 
     * @var string 
     */
    private $addr;

    public function __construct(string $addr) {
        $this->addr = $addr;
    }

    public function asString() {
        return $this->addr;
    }

    public function href() {
        return 'mailto:' . $this->asString();
    }
    
    public function class() {
        return false;
    }
}
``` 

Notice that we returned false for the class property, which removes it from the rendered output. We could have, as another 
example, set a flag in our Email object to signify the "current" or "preferred" email, and then output that 
dynamically. The options become almost limitless for you as you create more complex templates. The nesting and dynamic 
complexity is up to you and the requirements of your project.

### The Other Special Attributes - Prefix, Resource and TypeOf

As we mentioned in the beginning, Templado makes use of four RFDa recommended attributes. We covered the "property" 
attribute so what about these others. As should be clear, the "property" attribute gives Templado a context (scope) 
within which to work. However there may be times (for various reasons) that the context needs to change. Both "prefix" 
and "resource" provide a means to do that. They are all somewhat similar in that way, yet each one has its own 
specific usage. The "typeof" attribute is a little different, so we will save that one for last. Let's go over each of 
these attributes one by one. 

#### Prefix
Let's start with the "prefix" attribute. It is simply a way to alias (or namespace) your properties. Let's say that you 
have a property called "user" that you want to reference multiple times. You can use the "prefix" attribute in place of 
the "property" attribute to accomplish this:

```html
<div prefix="u: user">
    <p>Name: <span property="u:name">Original Name</span></p>
</div>
<div>
    <span>EMail:</span>
    <ul>
        <li property="u:emailLinks">
            <a property="email" href="mailto:original@domain.tld" class="current">original@domain.tld</a>
        </li>
    </ul>
</div>
```

In our prefix attribute we used the letter "u" (*short for user, obviously*) followed by a colon and a space, then 
the property that we want to reference (*in this case "user"*).

Now, since we have created an alias for the user property, notice that we must use the colon notation to reference its 
methods/properties. (*"u:" followed by the method name*)

But now we can reference it from anywhere else in our template. Notice that the "u:emailLinks" property is used 
**outside** of the user element. This can be quite useful if you have the need to reference the user property several 
times (or in several places) within your template. 

#### Resource

Let's move on to the "resource" attribute. It is similar to prefix in that it is also used in place of the 
property attribute, and it is also a way that we can reference a property's methods elsewhere in our template. The 
important difference is that "resource" tells Templado to look back to our main (*top level*) View Model. This is quite 
useful in situations where a property is nested within the scope of another property in your template. 
 
Let's say that you want to display "user" information in two (*or more*) different sections in your template. And each 
of these instances of user in nested within the scope of another property. Rather than duplicating the user methods 
within each of the parent Property Models, your can define the user method within the top level View Model, and then use 
the resource attribute to reference it there.

```html
<div property="sectionOne">
    <h2 property="sectionTitle">Section Title</h2>
    <div resource="user">
        <p>Name: <span property="name">Original Name</span></p>
        <div>
            <span>EMail:</span>
            <ul>
                <li property="emailLinks">
                    <a property="email" href="mailto:original@domain.tld" class="current">original@domain.tld</a>
                </li>
            </ul>
        </div>
    </div>
</div>
<div property="sectionTwo">
    <h2 property="sectionTitle">Section Title</h2>
    <div resource="user">
        <p>Name: <span property="name">Original Name</span></p>
        <div>
            <span>EMail:</span>
            <ul>
                <li property="emailLinks">
                    <a property="email" href="mailto:original@domain.tld" class="current">original@domain.tld</a>
                </li>
            </ul>
        </div>
    </div>
</div>
```

In this case user is nested with the scope of the sections (one and two), but because we used resource instead of 
property, Templado will look to our original View Model, rather than the Section Models. 

```php
class ViewModel {
    public function sectionOne() {
        return new SectionOne();
    }
    
    public function sectionTwo() {
        return new SectionTwo;
    }
    
    public function user() {
        return new User();
    }   
} 
``` 

Each of the Section Models would define their own "name" method in our example, but there is no need to add the user 
methods to them. Templado already knows where to find it!

#### TypeOf

Finally, let's discuss the "typeof" attribute. This one works a bit differently than the others. Rather than replacing 
the "property" attribute, "typeof" is used in conjunction with it. Also, rather than changing the context of the View Model 
being used, the "typeof" attribute gives you a way to conditionally change your template. It allows you to create 
multiple ways that an element could be displayed, and then select which version to use at rendering time.

Let's go back to our User example. Say you have three different kinds of Users. A standard User, a Commenter, and a 
Moderator (to use a very contrived example). When you display a standard User, you want it to look like it did in our 
previous examples - with a name and email addresses. However when you display a Moderator, you want to show that it is a 
Moderator, and you do not want to display the Moderator's email address(es). And when it is a commenter, you only want to 
display the name, and this User's rating - which a number rating from 1 to 5.

In this case, you can create a template that contains all three display versions. And identify each using the "typeof" 
attribute.

```html
... 

<div property="user" typeof="standard">
    <p>Name: <span property="name">Original Name</span></p>
    <div>
        <span>EMail:</span>
        <ul>
            <li property="emailLinks"><a property="email" href="mailto:original@domain.tld" class="current">original@domain.tld</a></li>
        </ul>
    </div>
</div>

<div property="user" typeof="moderator">
    <p>Moderator: <span property="name">Original Name</span></p>
</div>

<div property="user" typeof="commentor">
    <p>
        Commenter: <span property="name">Original Name</span> 
        Rating: <span property="rating">5</span>
    </p>
</div>

... 

```

Notice that each of the three main divs has a "property" of "user". But each has a different "typeof".

When Templado finds a "typeof" attribute, it look to the current View Model for a method specifically called "typeof". 
This "typeof" method should return a string that matches one of template occurrences. 

So now if our User Model is modified to look like this:

```php
class User {

    public function name() {
        return 'Willi Wichtig';
    }

    public function emailLinks() {
        return [
            new EMailLink(
                new Email('willi@wichtig.de')
            ),
            new EMailLink(
                new Email('second@secondis.de')
            )
        ];
    }

    public function rating()
    {
        return '3';
    }

    public function typeof()
    {
        return 'standard';
    }
}
```

Templado would first look for the "typeof" method in the current View Model. It sees that, in this case, "standard" is 
returned. So it looks back to our template, and ONLY renders the User element that has been identified as "standard". 
The other two are removed. 

```html
<div property="user" typeof="standard">
    <p>Name: <span property="name">Willi Wichtig</span></p>
    <div>
        <span>EMail:</span>
        <ul>
            <li property="emailLinks"><a property="email" href="mailto:willi@wichtig.de" class="current">willi@wichtig.de</a></li>
            <li property="emailLinks"><a property="email" href="mailto:second@secondis.de" class="current">second@secondis.de</a></li>
        </ul>
    </div>
</div>
```

If we go back to our User Model and simply change the "typeof" method to return "moderator", then our output would look 
like this:

```html
<div property="user" typeof="moderator">
    <p>Moderator: <span property="name">Willi Wichtig</span></p>
</div>
```

And if we changed the "typeof" method to return "commenter":

```html
<div property="user" typeof="commentor">
    <p>
        Commenter: <span property="name">Willi Wichtig</span> 
        Rating: <span property="rating">3</span>
    </p>
</div>
```

So you see, even from this silly example, that we can easily configure any number of template *views* for a given View Model, 
and then switch views by simply updating the return value for one method. This gives you incredible control over your 
templates, and so many options for how to configure things.

### Forms

Forms are a bit of a special case in templating. Though there may be times that you do want to dynamically set field 
attributes (*which can be handled with the methods discussed above*), one important need is that of populating the form 
fields with the supplied data. And though technically, with some forethought, you could handle the particulars of each 
field with the methods discussed above, Templado simplifies this task for you through the Templado\Engine\FormData Object.

Instantiating a FormData Object is easy. The constructor takes two parameters. The first is an identifier for the form, 
which can be either the form "name" or "id" attribute. The second is simply an array where the keys are the field names, 
and the values are, of course, the field values (*coincidentally, just as the post data of the form would look* ;).

So if you have a form in your template that looks like this:

```html
<form id="user-form">
    <p><label for="name">Name: </label><input id="name" name="name" /></p>
    <p><label for="email">Email: </label><input id="email" name="email" /></p>
    <p><label for="address">Phone: </label><input id="phone" name="phone" /></p>
    <p>
        <label for="gender">Gender: </label>
        <select id="gender" name="gender">
            <option value="female">Female</option>
            <option value="male">Male</option>
        </select>
    </p>
    <p><label for="description">Description: </label></p>
    <p><textarea id="description" name="description" rows="3" cols="32"></textarea></p>
    <p><label for="available">Available: </label><input type="checkbox" id="available" name="available" value="yes" /></p>
    <p>Handed::
        <label for="handed-left">Left: </label><input type="radio" id="handed-left" name="handed" value="left" />
        <label for="handed-right">Right: </label><input type="radio" id="handed-right" name="handed" value="right" />
    </p>
    <button>submit</button>
</form>
```  

And you have form data/values that looks like this:

```php
$formValues = [
    'name' => 'Boo Radley',
    'email' => 'boo@templado.com',
    'phone' => '555-1212',
    'gender' => 'male',
    'description' => 'Reserved, but thoughtful and kind fellow',
    'available' => 'yes',
    'handed' => 'right'
];
``` 

You simply instantiate a FormData Object like this:

```php
$formData = new Templado\Engine\FormData('user-form', $formValues);
```

And going back to our original template and data pairing code, apply it using the "applyFormData" method like this:

```php
try {
    $page = Templado::loadHtmlFile(
        new FileName(__DIR__ . '/html/form.xhtml')
    );
    
    $page->applyFormData($formData);

    echo $page->asString();

} catch (TempladoException $e) {
    foreach($e->getErrorList() as $error) {
        echo (string)$error;
    }
}
```
And now your rendered form will have each field properly populated. *It should be noted that the FormData Object can also 
handle multi-dimensional form data arrays*. 

**It should also be noted that password fields and file type fields are excluded for security reasons.**

While we are on the subject of forms, there is one more Templado feature that applies here. You are hopefully familiar 
with Cross-Site Request Forgery (**CSRF**). A full explanation of this security issue is well beyond the scope of this 
documentation. You can find more information about it [here](https://en.wikipedia.org/wiki/Cross-site_request_forgery)

A suggested method to protect against CSRF is the Synchronizer Token Pattern (**STP**), which involves adding a unique 
(*and unpredictable*) token to a hidden field when posting your form. Then of course, validating that token when processing 
of the form. 

Templado lets you dynamically add the hidden field to your form using the Templado\Engine\CSRFProtector Object. To 
instantiate this Object, you simply pass in a name for your (*soon to be*) field, and a token. **It is important to note 
that how you generate and track your token is up to you**. 

But once you have a token, you can create the CSRFProtector like this:

```php
$csrfProtector = new Templado\Engine\CSRFProtector('csrf-token', 'somelongtokenstring');
```

In this example we are naming our future hidden field 'csrf-token' (*the name is up to you*). *Obviously, the token 
itself would be different.* 

And we can add it to our form using the "applyCSRFProtection" method like this (*following the above form example*):

```php
try {
    $page = Templado::loadHtmlFile(
        new FileName(__DIR__ . '/html/form.xhtml')
    );
    
    $page->applyCSRFProtection($csrfProtector);
    
    $page->applyFormData($formData);

    echo $page->asString();

} catch (TempladoException $e) {
    foreach($e->getErrorList() as $error) {
        echo (string)$error;
    }
}
```

And now a new **hidden** input field will be rendered in your form with the name 'csrf-token', and a value of the token 
you passed in. Now, with some validation on the backend, your form is safer.

**Note: If a CSRF hidden field (with that name) already exists, rather than creating a new field, the existing field will 
be used. Its value will be updated.

---

So at this point we have pretty much covered all that you need to effectively create and manipulate basic templates. It 
is simple to use, clean and leaves you with templates that can be displayed on their own as representational pages. 

In our, final section we will cover a few slightly more abstract features - Snippets, Transformations, and Filters. 
These options give you full control over your final output.

### Snippets

Some templates, depending on your project and the requirements, can get a little complex. Perhaps, in these situations 
it would be nice to break the template down into smaller parts. Or maybe you have reusable static code blocks, and/or 
module elements that you don't want to keep duplicating. This is where Snippets come in. They allow you to write smaller, 
more manageable pieces of code, and then pull them all together on the fly.

Let's start as simply as possible, and you will immediately see how the possibilities of use are almost unlimited.

Say we have the following template code:

```html
<?xml version="1.0" ?>
<!DOCTYPE html>
<html lang="en">
    <head>
        <meta charset="UTF-8" />
        <title>Basic Snippet</title>
    </head>
    <body>
        <div id="header" />
    </body>
</html>
```   

Now you want to add a page title to the existing header div. You do this by first creating a \DOMNode. There are various 
ways to accomplish this. One straight forward way is with \DOMDocument::createElement:

```html
$codeBlock = new \DOMDocument();
$header = $codeBlock->createElement('h1', 'This Is A Simple Snippet Example');
```

Now that we have a \DOMNode, we can easily create a Templado Snippet.

```
$snippet = new \Templado\Engine\SimpleSnippet('header', $element);
```

Notice that the first parameter is the target id (from our template). The second parameter is of course the element to be added.

The method to apply Snippets to a Templado Html object expects a \Templado\Engine\SnippetListCollection. Even though we 
are adding only one Snippet in this example, we will still need to instantiate the Collection, and add our Snippet:

```
$snippetListCollection = new SnippetListCollection();
...
$snippetListCollection->addSnippet($snippet);
```

If we had multiple Snippets to apply, we could simply add them to the Collection. But for this example we can just go ahead 
and apply this one using the \Templado\Engine\Html::applySnippets method. We put it all together like so:

```php
try {
    $page = Templado::loadHtmlFile(
        new FileName(__DIR__ . '/html/basic.xhtml')
    );

    $snippetListCollection = new SnippetListCollection();
        
    $codeBlock = new \DOMDocument();
    $element = $codeBlock->createElement('h1', 'This Is A Simple Snippet Example');
    $snippet = new SimpleSnippet('header', $element);

    $snippetListCollection->addSnippet($snippet);
    
    $page->applySnippets($snippetListCollection);

    echo $page->asString();

} catch (TempladoException $e) {
    foreach($e->getErrorList() as $error) {
        echo (string)$error;
    }
}
```

Our Snippet gets applied, and the final rendered page in our example looks like this:

```html
<?xml version="1.0" ?>
<!DOCTYPE html>
<html lang="en">
    <head>
        <meta charset="UTF-8" />
        <title>Basic Snippet</title>
    </head>
    <body>
        <div id="header">
            <h1>This Is A Simple Snippet Example</h1>
        </div>
    </body>
</html>
```

Of course, in most cases you are are not going to want to add single simple elements one by one. You can easily create more 
complex Snippets by loading a partial HTML string into your DOMDocument, and then extracting the root element that you want:

```
$codeSample = new \DOMDocument();

$htmlString = '<div id="header"><h1>This is a more <span class="red">complex header</span><p>Click <a href="some-link">HERE</a></p></div>';

$codeSample->loadHTML($htmlString);

$element = $codeSample->getElementById('header');
``` 

As long as your HTML string is valid, this method will work! The rest of the process would be the same. Simply create a 
SimpleSnippet from our (more complex) DOMElement, add it to a SnippetListCollection, and apply it to the Page. And now our 
rendered page looks like this:

```html
<?xml version="1.0" ?>
<!DOCTYPE html>
<html lang="en">
    <head>
        <meta charset="UTF-8" />
        <title>Basic Snippet</title>
    </head>
    <body>
        <div id="header">
            <h1>This is a more <span class="red">complex header</span>
            <p>Click <a href="some-link">HERE</a></p>
        </div>
    </body>
</html>
```

In this example we used a single element (with children) as our Snippet. And since the id of the Snippet element 
matched the id of the template element, the template element was replaced.

**Note:** If Templado does not find an element matching our target id, our Snippet is ignored. 

If instead of replacing the template element, you want to append multiple children to it. You could, of course, create 
each of the child elements as individual Snippets and add them all to the Collection, but this would be quite tedious. 
If you wanted to group them as one Snippet there would be an issue because they would not a have common root element. 
Your partial code would be invalid. For this case, we can use \Templado\Engine\TempladoSnippet. 

We can take a partial HTML string, and simply wrap it in a Templado namespace. 

So this string: 

```
$partialHtmlString = '<h1>This is a more <span class="red">complex header</span><p>Click <a href="some-link">HERE</a></p>';
```
Becomes:

```
$partialHtmlString = 
        '<page:snippet xmlns:page="https://templado.io/snippets/1.0" xmlns="http://www.w3.org/1999/xhtml">
            <h1>This is a more <span class="red">complex header</span><p>Click <a href="some-link">HERE</a></p>
        </page:snippet>';
```

Then we load our (now valid) XML into our DOMDocument. 

```
$codeSample = new \DOMDocument();
$codeSample->loadXML($partialHtmlString);
    
$snippet = new TempladoSnippet('header', $codeSample);
```

And again, the rest of the process is the same. We add it to a SnippetListCollection, and apply it to our page. 

This is powerful, but let's get to the real point here. Snippets are way to combine, nest and modify blocks of code. But 
notice that we are still working with the same base object - the Templado\Engine\Html object. And therefore, we can still 
apply View Models to it. 

Let's modify our example again. But this time let's create a more complex fragment:

```php
... 

$xhtml = <<<xhtml
    <page:snippet xmlns:page="https://templado.io/snippets/1.0" xmlns="http://www.w3.org/1999/xhtml">
        <h1 id="main-title">This Is A Simple Snippet Example And Then Some</h1>
        <div property="user">
        <p>Name: <span property="name">Original Name</span></p>
            <span>EMail:</span>
            <ul>
                <li property="emailLinks">
                    <a property="email" href="mailto:original@domain.tld" class="current">original@domain.tld</a>
                </li>
            </ul>
        </div>
    </page:snippet>
xhtml;

$codeSample->loadXML($xhtml);

$snippet = new TempladoSnippet('header', $codeSample);

$snippetListCollection->addSnippet($snippet);

$page->applySnippets($snippetListCollection);
```
 
Once we have added our extended Snippet to our base template, but before we render it, we can apply our View Model:

```php
$page->applyViewModel(new ViewModel());
```

**If you have not read the previous documentation on View Models and the property attribute, now would be a good to do so**. 
Otherwise, you understand that you would now get a fully rendered page.

Of course, it is also important to note that you can also change the order of operations here. You can quite easily load 
partial code as a Templado\Engine\Html Object, apply a ViewModel to get a rendered partial code block. Then load that as 
a Snippet, and apply it to a more complex page document.

This is quite useful if you want to combine static assets with dynamically created Snippets.

Additionally it should be pointed out that, since you can load the partial code from a file, it would be trivial to 
create well organized, reusable code fragment files.   

#### SnippetLoader
Which brings us to one final Snippet feature - Templado\Engine\SnippetLoader.

One way that you can create Snippets from a file, is with Templado\Engine\SnippetLoader. It basically has one public function 
called "load", which takes in a Templado\Engine\FileName.

If all you want to do is add a simple TextNode, then you can just create a file with your text. Of course, you will still 
want to wrap the text in a Templado namespace like in the above example. But other than that, you can simply instantiate 
a SnippetLoader, load your file, and pass the Text Snippet along.

For more complex Snippets, you will want to add a DOCTYPE declaration to your file as well, so that Templado knows what 
kind of document it is working with.

But again, all that is necessary is to call the load function, and you will get back a proper Snippet. 

```php
...

$snippetLoader = new SnippetLoader();

$snippet = $snippetLoader->load(new FileName(__DIR__ . 'code_fragment.xhtml'));

$snippetListCollection->addSnippet($snippet);

$page->applySnippets($snippetListCollection);

...

```

And there you have it. We could keep adding more and more complex examples, but it should be clear at this point just 
how powerful this is. You can easily create more complex blocks of code that can be assembled in any number of ways. 
These code fragments can be reused, and new Snippets can be added to them. As stated earlier, the ways this can be applied 
are almost endless.  

### Transformations and Filters

Occasionally, there may be times when you have the need to manipulate the final output beyond just applying data models. 
Perhaps there is something that needs to be completely stripped out of the document, or changed based on the environment, 
or for temporal reasons. The question of why you would need to change something is basically beside the point. There could 
be any number of reasons. The question is how. 

Templado gives you two options for handling these situations - Transformations and Filters. Let's start with Transformations.

Perhaps we decide that we do not want our final rendered markup to contain any of the RFDa attributes we used. These 
attributes were for our templating purposes, and are superfluous once the rendering is complete. We can do this. 
Templado gives you the option of writing and applying a Transformation.

#### Transformations

Templado\Engine\Transformation is an Interface with just two methods defined - "getSelector" and "apply". 

The "getSelector" method is used to define which elements of the document you would like to affect. And the 
"apply" method defines what you would actually like to do. 

You could implement the Transformation Interface like this:

```php
class StripRDFaAttributesTransformation implements Transformation {

    /** @var string[] */
    private $attributes = ['property', 'resource', 'prefix', 'typeof'];

    public function getSelector(): Selector {
        return new XPathSelector('//*[@' . implode(' or @', $this->attributes) . ']');
    }

    public function apply(DOMNode $context) {
        if (!$context instanceof \DOMElement) {
            return;
        }

        foreach($this->attributes as $attribute) {
            $context->removeAttribute($attribute);
        }
    }

}
```

The signatures of the methods are of course defined in the interface. The "getSelector" method must return a 
Templado\Engine\Selector. Templado has two built in Classes that can help you with that. In this example we are using the 
Templado\Engine\XPathSelector to define the Selector using XPath syntax. If you aren't familiar with this syntax you can 
find a nice explanation [here](https://www.w3schools.com/xml/xpath_syntax.asp)

So in our example above, we instantiate a new XPathSelector using the syntax to select all elements that contain any of 
the four RDFa attributes. As you can see, we have a private array containing the four attributes, and then use PHP's 
"implode" method to create the selector string. And we return that.

Internally, the "apply" method is called with each element (*node*) that matches the selector. We make sure that what was 
passed in is actually a DOMElement, and if so, we loop through the list of attributes and delete them. 

It is easy enough to write a Transformation, but it is even easier to put it to use. 

If we go back to the code where we are manipulating our templates, we can add one more line to apply our Transformation.

```php
try {
    $page = Templado\Engine\Templado::loadHtmlFile(
        new Templado\Engine\FileName(__DIR__ . '/html/viewmodel.xhtml')
    );
    $page->applyViewModel(new ViewModel());
    
    $page->applyTransformation(new PropertyRemoverTransformation());

    echo $page->asString() . "\n";

} catch (Templado\Engine\TempladoException $e) {
    foreach($e->getErrorList() as $error) {
        echo (string)$error;
    }
}
```

And that's it! In the final output, all elements containing the RFDa attributes would be gone.

**Note: if you think that's a pretty cool idea, you are in luck. The Templado\Engine\StripRDFaAttributesTransformation 
class is part of the core Templado code.** If you have other ideas for transforming your templates, you will need to write 
them yourself, but this Transformation can be used out of the box.

I mentioned that there are two Classes in Templado that can be used to create a Selector. We used XPathSelector in this 
example, but the other is Templado\Engine\CSSSelector. It extends the XPathSelector, and allows you to use CSS syntax 
instead of XPath. Either way, Transformations are easy to use and a powerful way to modify your document.

And this brings us to the final way that Templado equips you to control your output. Transformations are a great to 
manipulate DOM nodes, but that heavy lifting can come at an efficiency cost by having to traverse all those nodes. This 
is where Filters come in. And this is the last stop in the process before your document is fully rendered. 

#### Filters

When you call the "asString" method on your Templado\Engine\Html object, your document is serialized. At this point, you 
get one last chance to modify that string. To do this Templado features the Templado\Engine\Filter interface, which has 
only one defined method called "apply". By creating an Object that implements Filter, you can ue the "apply" method to act 
on the final serialized document just before it is output. Since it is a string, you can use regex, and/or any of PHP's 
built in string modifying methods to tweak your document. 

In all of our examples so far, we have simply called the "asString" method on our Html Object to create the final output. 
It is important to note though, that this method takes in an optional Filter parameter. In fact, if you were to look under 
the hood, you would see that Templado has two built in Filters that get run automagically inside the "asString" 
method. One of these Filters truncates any left over empty HTML tags, and the other removes all XML Namespaces.

Let's take a look at the empty element Filter as a usage example:

```php
class EmptyElementsFilter implements Filter {

    public function apply(string $content): string {
        $tagList = [
            'base', 'br', 'meta', 'link', 'img', 'input', 'button', 'hr', 'embed',
            'param', 'source', 'track', 'area', 'keygen',
        ];

        foreach($tagList as $tag) {
            $content = preg_replace(
                "=<{$tag}(.*[^>]?)></{$tag}>=U",
                "<{$tag}\$1 />",
                $content
            );
            if ($content === null) {
                throw new EmptyElementsFilterException(
                    'Error while processing regular expression',
                    preg_last_error()
                );
            }
        }

        return $content;
    }

}
```

And let's walk through this. As you can see, and as stated before, there is only one method to implement. This "apply" 
method takes in the entire serialized document string, and returns it after any modifications.

So in this example, we see that an array of empty tag candidates is created. Then the code loops through each of those 
tags, and performs a "preg_replace" on each one. It looks for tags that have both an open and close, but nothing inside. 
If a match is found, the attributes of the original tag are copied to the short version of the tag, and the code continues. 

**Note that since a replace can fail, this code throws an Exception if something goes wrong. Perhaps even more importantly 
a custom Exception was created for this Class - an important best practice.**

Once the loop is complete, the final modified string is simply returned.

As stated previously, this Filter, and the Namespace Filter are built into the "asString" method. But if you want to 
create a Filter of your own, you can simply pass it in as a parameter. So if you have a custom Filter called 
"BadTagRemoverFilter", you could use it like this:

```php
...

$page->asString(new BadTagRemoverFilter());
```

Your Filter would be applied, and all those pesky bad tags would be gone from your final output. 

And that is Templado! 

### Conclusion 

We have gone over all the ways that Templado can be used to manipulate templates. As should be evident, Templado is 
powerful, yet simple and concise. And most importantly Templado allows you to use plain XHTML documents as templates. 
All of your stakeholders will be able to see and understand.

There is one final thing to point out. In the beginning, the method of using PHP View Models was illustrated. The Templado 
engine relies on these View Models. However there is a separate repository in the Templado family that has two Mappers - 
A JSON Mapper, and an XML Mapper. The usage for either one is mostly the same, and very simple:

```php
$input = file_get_contents(__DIR__ . '/viewmodel/viewmodel.json');
$mapper = new JsonMapper();
$obj = $mapper->fromString($input);

$page->applyViewModel($obj);
```

In the above example a new JsonMapper is instantiated. Then the "fromString"  method is called, and the JSON string is 
passed in. This creates a View Model Object that can then be passed into the "applyViewModel" method. 

For XML it is similar, but with a few slight differences. First the Mapper is called DomDocumentMapper. So you would 
instantiate that one. Then instead of a string, this Mapper expects either a PHP DOMDocument, or a DOMElement. So you 
would need to create one of those from your XML, and then pass it to the appropriate method - either the 
"fromDomDocument" method, or the "fromDomElement". Kind of makes since if you think about it, huh?!

**Note: The mappers are meant as mere helpers to have a generic mapping from json data structures into view models. They 
are not a part of the engine repository, because they are not the recommended way of doing things. It is generally better 
to write your own View Models.**

Anyway, that is it. For more in depth usage of the mappers, look to the documentation for the Mapper repository. Otherwise, 
good luck templating with Templado, and please let us know what you think.

## Examples

Usage examples can be found in the [example project](https://github.com/templado/examples)
