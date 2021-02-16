# Heading Component

Following the [Atomic Design](https://bradfrost.com/blog/post/atomic-web-design/)methodology we are going to build a simple component or pattern. The Heading pattern is an atom which prints a string of text as the title for a page, paragraph, or other component.  As we discussed before, most components will need the following files:

| File type | Purpose |
| :--- | :--- |
| `.twig` | Component's HTML and Twig logic |
| `.scss` | Component's styles |
| `.json` or `.yml` | Component's stock content |

Some components may also include:

| File type | Purpose |
| :--- | :--- |
| `.js` | Component's interactive behavior |
| `.md` | Component's annotations or specs details |

## Exercise: Building the component

We'll leave styles/CSS out of this component for now.

### Component's data structure

Let's start by first identifying the content for the component. Since this is a title field, we only need a string of text, but to make this component more flexibile and dynamic, we'll include some options in its data structure.

1. Inside _components_ create a new folder called **heading**
2. Inside the _heading_ folder create a new file called **heading.json**
3. Inside _heading.json_ add the following code:

{% tabs %}
{% tab title="heading.json" %}
```yaml
{
  "heading": {
    "heading_level": "",
    "modifier": "",
    "title": "Welcome to Mediacurrent training!",
    "url": ""
  }
}
```
{% endtab %}
{% endtabs %}

We just created a JSON object for the heading with properties that include **heading\_level, modifier, title**, and **url**.

* The **heading\_level** is something we will use later as we start nesting the heading component into other components. This will allows us to change the headings from say, h1 to h2 if we need to.
* The **modifier** key allows us to pass a modifier CSS class when we make use of this component. The modifier class will make it possible for us to style the heading differently than other headings, if needed.
* the **title** key is the title's string of text that will become the title of a page or a component.
* ... and finally, the **url** key, if present, will allow us to wrap the title in an `<a>` tag, to make it a link.


### Component's Markup

Now let's write some HTML to be able to see the Heading component in the browser.

1. Inside the _heading_ folder create a new file called **heading.twig**
2. Inside `heading.twig` add the following code:

{% tabs %}
{% tab title="heading.twig" %}
```php
<h{{ heading.heading_level|default('2') }} class="heading{{ heading.modifier ? ' ' ~ heading.modifier }}">
  {% if heading.url %}
    <a href="{{ heading.url }}" class="heading__link">
      {{ heading.title }}
    </a>
  {% else %}
    {{ heading.title }}
  {% endif %}
</h{{ heading.heading_level|default('2') }}>
```
{% endtab %}
{% endtabs %}

Wow! What's all this? 😮

* Line 1, makes use of `heading.heading_level` to complete the number part of the heading.  If a value is not provided for `heading_level` in the JSON file, we are setting a default of `2`.  This will ensue that by default we will have a `<h2>` as the title, much better than `<h1>` as we had before.  This value can be changed per use case if needed.  Line 9 closes the heading tag.
* Also in Line 1, we are checking whether there is a value for the `modifier` key in JSON.  If there is, we are passing it to the heading as a CSS class.  If no value is provided nothing will be added.
* In line 2, we check whether a URL was provided in the JSON file, and if so, we wrap the `{{ title }}` variable in a `<a>` tag to turn the title into a link.  The **href** value for the link is `{{ heading.url }}`.  If no URL is provided in the JSON file, we simply print the value of `{{ title }}`as plain text.

### Compiling the code

If Pattern Lab is running you should see the updates to the Heading component.  Otherwise run the commands below from your theme's root directory:

```text
npm run watch
```

At the bottom of the watch command you will notice a list of URLs. In your browser of choice open the following url: [http://localhost:3000](http://localhost:3000). This will open Pattern Lab where you can find the Heading pattern under components.

**Just for fun 💥**

* Try changing the heading level in `heading.json` to anything other than H2.
* Inspect your code to see your changes to the Heading pattern.

**Congratulations!** You just built your first reusable component! 🙌 🎉
