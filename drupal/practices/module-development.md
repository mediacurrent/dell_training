# Module Development

The following best practices can guide you in your module development process. In addition, [Drupal.org has detailed documentation](https://www.drupal.org/docs/creating-custom-modules) on building custom modules.

## Use the `custom` subdirectory

Profiles, modules, and themes should all take advantage of Drupal's support for custom subdirectories. Modules installed with composer will automatically be placed in their respective `contrib` directories, so custom code can be placed in one of the following locations, depending on what it is:

* `profiles/custom/<profile_name>`
* `modules/custom/<module_name>`
* `themes/custom/<theme_name>`

## Use prefixes for custom modules

To avoid hook namespace collisions with current and future contributed modules, you should prefix custom modules, profiles, and themes with your project name. For instance, if your project name is `olas`, you could create the following:

* `profiles/custom/olas_profile` - Install profile for the project.
* `modules/custom/olas_api_connector` - Custom API connector required by the Olas project.
* `modules/custom/olas_migrate` - Module to handle migrations for the project.
* `themes/custom/olas_theme` - Custom theme for the Olas project.

## Enable error output on local environments

For local environments such as DDEV, it can be helpful to enable full error reporting. In your `settings.local.php`, add the following to output error messages in the browser in addition to normal logging:

```php
error_reporting(E_ALL);
ini_set('display_errors', TRUE);
ini_set('display_startup_errors', TRUE);
```

While this can be very helpful for local environments, it should never be enabled on publicly accessible environments such as production.

## Keep the ".module" file lightweight

Your modules' `<module_name>.module` file will be loaded on every HTTP request if it's enabled. In addition, the file is oriented around procedural code, while Drupal 8 can leverage object-oriented code much more thoroughly than Drupal 7 was able to. In many cases you'll have to place code in the module file due to needing to use Drupal's hooks, but when possible, it's often better to place your code in the `src` directory of the module in a class that's invoked automatically by Drupal or as a service.

## Create an info.yml file, and follow the latest structure for it

Modules should have an `<module_name>.info.yml` file that follows the [latest standards](module-development.md) for how they're structured. For example:

```yaml
name: Hello World Module
description: Creates a page showing "Hello World".
package: Custom

type: module
core: 8.x

dependencies:
  - drupal:link
  - drupal:views
  - paragraphs:paragraphs
  - webform:webform (>=8.x-5.x)

test_dependencies:
 - drupal:image

configure: hello_world.settings

php: 5.6

hidden: true
required: true

# Note: do not add the 'version' or 'project' properties yourself if this is a contributed module.
# They will be added automatically by the packager on drupal.org.
# version: 1.0
# project: 'hello_world'
```

## Avoid writing custom modules and code when possible

In general, it requires less maintenance to use a contributed module when possible, as long as that module is in a relatively stable beta or release state. If that's not possible, custom modules should ideally be written in a re-usable manner so that they can either become new contributed modules or become part of an internal private module that can be re-used across projects to reduce work efforts in the future.

If neither of those cases are possible, a new custom module should be developed.

## Adhere to Drupal code standards and use tests to confirm

Developers should follow Drupal [code standards](https://mediacurrent.gitbook.io/olas2021/drupal/practices/coding-standards) at all times. Code editors such as PhpStorm and VS Code both have settings or extensions to allow those standards to be run directly in the editor, such as every time a file is saved. In addition, projects should include the Coder project and then that can be configured to run on both git commits and as part of automated testing pipelines in case anything was missed by the developer's code editor.

## Include docblock commments

In addition to inline and multiline comments, you should take advantage of [Drupal's support for docblock](https://www.drupal.org/docs/develop/standards/api-documentation-and-comment-standards#s-notes-and-standards) comments and document your functions, classes, and other constructs using the docblock syntax. For example, the following comment could apply to a batch API callback:

```php
/**
 * Perform tasks when a batch is complete.
 *
 * Callback for batch_set().
 *
 * @param bool $success
 *   A boolean indicating whether the batch operation successfully concluded.
 * @param int $results
 *   The results from the batch process.
 * @param array $operations
 *   The batch operations that remained unprocessed. Only relevant if $success
 *   is FALSE.
 *
 * @ingroup callbacks
 */
function callback_batch_finished($success, $results, $operations) {
  // Sample code here.
}
```

## Utilize configuration management

If your module includes custom configuration settings, you should include default configuration for those settings in your `config/install` directory of the module.

## Utilize caching

Drupal 8 has a very robust caching system that can be used in a number of ways. For example:

* For things such as heavy database queries or access tokens on API connections, you can cache those results for faster access on subsequent runs.
* Blocks, forms, and custom render arrays all benefit greatly from implementing cache tags and contexts that allow you to more easily target them for accurate cache invalidation.
* If using a CDN caching layer such as CloudFlare, it's often beneficial to disable Drupal's page cache and rely on Drupal's dynamic page cache instead, in combination with the CDN caching layer taking the place of the static page cache.

## Never hack Drupal core or contributed modules

As a rule, you should never hack custom changes into Drupal core or contributed modules. In scenarios where you need to implement a customization for either core or a contributed module, you should instead create a patch and apply it via compoaser.

## Use render arrays and Twig templates

In Drupal 8, custom templates can be defined with hook\_theme\(\), which will then allow you to make custom render arrays to pass data to your custom Twig templates. This can be especially useful for scenarios such as custom blocks or custom queries that aren't built with Views. In most cases, your rendered controllers or blocks should return a render array rather than raw HTML created by calling Drupal's internal renderer; doing this will allow caching to bubble up to higher contexts appropriately and allow for modifications of those render arrays as needed.

## Use object-oriented programming

A fundamental change in Drupal 8 from earlier versions is its shift to using object-oriented programming principles. The [Services and dependency injection in Drupal 8+](https://www.drupal.org/docs/drupal-apis/services-and-dependency-injection/services-and-dependency-injection-in-drupal-8) page on Drupal.org goes into great detail on how to utilize the two concepts.

As a quick summary, services are best suited for procedural code while dependency injection is suitable for usage in controllers or other object-oriented code.

### Use services in procedural code

When working with procedural code \(such as hooks in the .module file\), you can access object-oriented services using the `\Drupal::service('service.id')` method. [A full list of services for Drupal 9 can be found here](https://api.drupal.org/api/drupal/services/9.2.x).

For example, to connect to the database and execute select queries, you can use code similar to the following:

```php
$connnection = \Drupal::service('database');
$query = $connection->select('redirect', 'redirect')
  ->addField('redirect', 'redirect_source__path', 'source')
  ->addField('reidrect', 'redirect_redirect__uri', 'destination');
$results = $query->fields('redirect', ['status_code'])
  ->condition('redirect.redirect_source__path', 'http://%', 'NOT LIKE')
  ->execute();
foreach ($results as $row) {
  echo "Source: {$row->source}, Destination: {$row->destination}";
}
```

### Use dependency injection in controllers and object-oriented code

When working directly inside a class' code, you can instead use Dependency Injection. For a detailed example of how to use dependency injection, visit the [Dependency Injection for a Form](https://www.drupal.org/docs/drupal-apis/services-and-dependency-injection/dependency-injection-for-a-form) page.

### Defining your own services

To define your own services, you'll need to create a `<module_name>.services.yml` file and include it there. A service follows this format:

```yaml
services:
  olas.database:
    class: Drupal\olas\OlasDatabase
    arguments: ['@database']
```

The class should point to the class your service uses, while arguments include any dependencies required by the service. You can also supply other keys like `tags` and `factory` for more advanced services usage.

For the most part, your service class will behave just like any other class you make. One key difference is that in the `__construct()` method for the class, you'll be able to take the dependencies that were passed to the service and assign them to properties in the class. For example, the service above includes the `@database` dependency that could function similar to the following:

```php
/**
* Class OlasDatabase.
*/
class OlasDatabase {
 /*
  * @var \Drupal\Core\Database\Connection $database
  */
 protected $database;

 /**
  * Constructs a new OlasDatabase object.
  * @param \Drupal\Core\Database\Connection $connection
  */
 public function __construct(Connection $connection) {
   $this->database = $connection;
 }
}
```

In the example above, you could then create additional methods that use the `$this->database` property as if it was the `\Drupal::service('database')` service.

## Using hooks to alter behavior

Drupal provides a large variety of hooks \(see [this page](https://api.drupal.org/api/drupal/core%21core.api.php/group/hooks/9.1.x) for the full list\) that can be used to alter and extend its behavior in a variety of contexts.

For example, variables passed to a twig template for a node can be modified with `hook_preprocess_node()` or a form could be modified to add additional validation with `hook_form_alter()`.

## Defining custom pages and blocks

### Defining pages

See also: [https://www.drupal.org/docs/creating-custom-modules/create-a-custom-page](https://www.drupal.org/docs/creating-custom-modules/create-a-custom-page)

There are two steps to create a custom page. First, you'll need to define a route in `<module_name>.routing.yml` and then define a controller in `<module_name>/src/Controller/ExampleController.php`.

#### Declaring routes

```yaml
olas.my_page:
  path: '/mypage/page'
  defaults:
    _controller: '\Drupal\olas\Controller\OlasController::myPage'
    _title: 'My first page in D8'
  requirements:
    _permission: 'access content'
```

Each route has the following parts:

* **olas.my\_page** - The machine name of the route, in this case, `olas.my_page`. Follow the format `<module_name>.<route_name>`
* **path** - The path to the page on your site \(must include the leading `/`\)
* **defaults** - The page \(\_controller\) and title \(\_title\) callbacks.
* **requirements** - Conditions for the page to be displayed, such as permissions, dependency modules, or other conditions.

#### Adding a controller

Page callbacks are handled by controllers in Drupal 8 and 9. At minimum, the method that your route uses for the callback must be a public method that returns a renderable array or response object.

Building off of the prior route example, this example controller could be placed in `olas/src/Controller/OlasController.php` for our sample Olas module.

```php
<?php
namespace Drupal\olas\Controller;

use Drupal\Core\Controller\ControllerBase;

/**
 * Provides route responses for the Olas module.
 */
class OlasController extends ControllerBase {

  /**
   * Returns a simple page.
   *
   * @return array
   *   A simple renderable array.
   */
  public function myPage() {
    return [
      '#markup' => 'Hello, world',
    ];
  }

}
```

### Defining blocks

Blocks in Drupal 8 and 9 are instances of the block plugin. A [detailed walkthrough of creating custom blocks](https://www.drupal.org/docs/creating-custom-modules/creating-custom-blocks) can be found on Drupal.org

At it's most basic level, a block should be a class that's defined in `<module_name>/src/Plugin/Block/<ClassName>.php`. For example, the following block could be defined with the filename `HelloBlock.php`:

```php
<?php

namespace Drupal\hello_world\Plugin\Block;

use Drupal\Core\Block\BlockBase;

/**
 * Provides a 'Hello' Block.
 *
 * @Block(
 *   id = "hello_block",
 *   admin_label = @Translation("Hello block"),
 *   category = @Translation("Hello World"),
 * )
 */
class HelloBlock extends BlockBase {

  /**
   * {@inheritdoc}
   */
  public function build() {
    return [
      '#markup' => $this->t('Hello, World!'),
    ];
  }

}
```

## Defining settings forms or other forms

Custom forms in Drupal are defined with a Form class that includes several important methods, such as the Build method. Drupal.org includes [an excellent example](https://www.drupal.org/docs/creating-custom-modules/module-config-settings-form) of making an admin configuration form for a module.

In this example, we'll be defining a form at `<module_name>/src/Form/LoremIpsumForm.php`. The example below includes additional inline comments for each major section.

In addition to this example of a configuration form that's extended from ConfigFormBase, you can also create a form usable in any other context by extending the FormBase class instead. See the [Dependency Injection for a Form](https://www.drupal.org/docs/drupal-apis/services-and-dependency-injection/dependency-injection-for-a-form) page for an example of that configuration.

```php
<?php

namespace Drupal\loremipsum\Form;

use Drupal\Core\Form\ConfigFormBase;
use Drupal\Core\Form\FormStateInterface;

class LoremIpsumForm extends ConfigFormBase {

  // getFormId() defines the form's machine name.

  /**
   * {@inheritdoc}
   */
  public function getFormId() {
    return 'loremipsum_form';
  }

  // The buildForm() method defines the form's initial state and fields.

  /**
   * {@inheritdoc}
   */
  public function buildForm(array $form, FormStateInterface $form_state) {
    // Form constructor.
    $form = parent::buildForm($form, $form_state);
    // Default settings.
    $config = $this->config('loremipsum.settings');

    // Page title field.
    $form['page_title'] = [
      '#type' => 'textfield',
      '#title' => $this->t('Lorem ipsum generator page title:'),
      '#default_value' => $config->get('loremipsum.page_title'),
      '#description' => $this->t('Give your lorem ipsum generator page a title.'),
    ];
    // Source text field.
    $form['source_text'] = [
      '#type' => 'textarea',
      '#title' => $this->t('Source text for lorem ipsum generation:'),
      '#default_value' => $config->get('loremipsum.source_text'),
      '#description' => $this->t('Write one sentence per line. Those sentences will be used to generate random text.'),
    ];

    return $form;
  }

  // The validateForm() method is called prior to submit and handles form errors.

  /**
   * {@inheritdoc}
   */
  public function validateForm(array &$form, FormStateInterface $form_state) {

  }

  // The submitForm() method is called when the form successfully submits.

  /**
   * {@inheritdoc}
   */
  public function submitForm(array &$form, FormStateInterface $form_state) {
    $config = $this->config('loremipsum.settings');
    $config->set('loremipsum.source_text', $form_state->getValue('source_text'));
    $config->set('loremipsum.page_title', $form_state->getValue('page_title'));
    $config->save();
    return parent::submitForm($form, $form_state);
  }

  // The getEditableConfigNames() is used to retrieve config names related to the form.

  /**
   * {@inheritdoc}
   */
  protected function getEditableConfigNames() {
    return [
      'loremipsum.settings',
    ];
  }

}
```

## Defining custom field types, widgets, and formatters

To create a custom field, you'll want to follow the basic structure used on the [Creating custom field types, widgets, and formatters](https://www.drupal.org/docs/creating-custom-modules/creating-custom-field-types-widgets-and-formatters) page on Drupal.org. Some high level best practices to keep in mind for fields:

* Search to see if there's a contributed module solution for your field type first. The majority of use cases have already been solved by the community, and this can be a huge time saver.
* For some more complicated fields you may be able to use another solution such as Paragraphs or Field Groups to combine several fields into an entity or group, rather than making a field type with multiple inputs.
* Fields in Drupal 8 are implemented using the Plugins API rather than hooks. For each field you'll need to define a field item, a field formatter, and a field widget.

## Using event subscribers

See the [Subscribe to and dispatch events](https://www.drupal.org/docs/creating-custom-modules/subscribe-to-and-dispatch-events) page on Drupal.org for a detailed breakdown of creating and using event subscribers.

Event subscribers can be thought of as an object-oriented approach to the same issue that hooks have solved in Drupal previously. Not all hooks have been converted into event subscribers, but a substantial portion of them are available there now as well.

### Event Subscribers

Your event subscribers are effectively similar to hooks. Also known as listeners, event subscribers are callable methods or functions that react to an event being propagated by the Event Registry.

### Event Registry

The event registry is where event subscribers are collected and sorted. In Drupal 7, this was handled in the `cache_bootstrap` bin under the `module_implements` id.

### Event Dispatcher

The event dispatcher is the mechanism that triggers events and dispatches them throughout the system. In Drupal 7, this would have been the `module_invoke_all()` function.

### Event Context

Event contexts are used to provide specific data to event subscribers, such as simple values or complex classes with relevant data. In Drupal 7, this would have been equivalent to parameters passed to the hook.

## Writing tests

Automated testing will be discussed in more detail in the DevOps portion of the training, but Drupal.org does have a [detailed documentation](https://www.drupal.org/docs/automated-testing) on creating tests, such as [this article about functional tests](https://www.drupal.org/docs/creating-custom-modules/testing-a-drupal-module).

There are several [different kinds of automated tests](https://www.drupal.org/docs/testing/types-of-tests) for various scenarios:

* **Unit Tests** - Unit tests without and with a database connection. For detailed identification of bugs in your code and building a logical application structure.
* **Kernel Tests** - Kernel tests sit between Unit tests which allows them to interact with the Drupal application in a meaningful way, but they allow the test to only utilize the aspects of Drupal required for the test to function.
* **Functional Tests** - This type of testing is often called System or Functional testing because it tests on the complete system. Drupal 8 ships with two types: BrowserTestBase and JavascriptTestBase.
* External frameworks, such as Behat

## Creating and attaching libraries \(CSS and JS\)

In Drupal 8, stylesheets \(CSS\) and JavaScript \(JS\) should be implemented using libraries. The majority of your project's CSS and JS will likely be in the theme, but there are often scenarios where it's more appropriate to place them in a module. For example, if a particular component needs to function on both the admin theme and the front-end theme, a module is a better place for its code. See [this page](https://www.drupal.org/docs/creating-custom-modules/adding-stylesheets-css-and-javascript-js-to-a-drupal-module) on Drupal.org for more information on CSS and JS in Drupal.

In general, adding CSS and JS to Drupal follows this process:

1. Save the CSS and/or JS to a file in your module, such as `<module_name>/css/foobar-layout.css`
2. Define a library which contains the JS and CSS, along with an dependency declarations
3. Attach the library to a render array, either via a hook or directly on a custom render array

### Libraries

Libraries are defined in a `<module_name>.libraries.yml` file in your module. For example:

```yaml
foobar:
  version: 1.x
  css:
    layout:
      css/foobar-layout.css: {}
    theme:
      css/foobar-theme.css: {}
  js:
    js/foobar.js: {}
```

There are five different levels of weight in the CSS section:

* **base** - `Weight -200` - CSS reset/normalize
* **layout** - `Weight -100` -Arrangement of markup, including grid systems
* **component** - `Weight 0` - Individual and reusable UI elements
* **state** - `Weight 100` - Client-side changes to components
* **theme** - `Weight 200` - Purely visual styling \(look and feel\)

