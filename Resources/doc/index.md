# Infinity Test Bundle

The aim of the Infinity Test Bundle is to provide a ready to use testing environment based on Behat, Mink, Phpspec and Selenium.

## Install the bundle

---
 Infinity Test Bundle requires `Xvfb` and `Selenium2` to be installed in the system before installing the bundle.
 In order to install `Selenium2` and `Xvfb` in you system run 
 
```sh
yum install selenium
```

 in a terminal before install this bundle.
 
---
Add a few lines in your composer.json to install the bundle

```json
{
    "require": {
        "infinitytracking/test-bundle": "*"
    }
    ...
    "scripts": {
        "post-install-cmd": [
      		"Infinity\\Bundle\\TestBundle\\Composer\\ScriptHandler::installConfigurationFiles",
      		...
        ],
        "post-update-cmd": [
      		"Infinity\\Bundle\\TestBundle\\Composer\\ScriptHandler::installConfigurationFiles",
      		...
        ]
    },
}
```

The post install script installs behat.yml.dist and phpspec.yml.dist under your `app/config` folder. Take a look to this document [Strategy to manage behat and phpspec yml config files](howto_manage_behat_and_phpspec_config.md) to see how you can manage these file.  

The position of the script is important, it must be before the Incenteev\ParameterHandler ScriptHandler, if you're using it as suggested in [Strategy to manage behat and phpspec yml config files](howto_manage_behat_and_phpspec_config.md).  

Finally, install the bundle typing this in a terminal

```sh
./composer.phar update infinitytracking/test-bundle
```

The Test Bundle will install Behat, Phpspec, PHPUnit, Mink, Goutte and Selenium2 drivers and the Behat Symfony2 extension as dependencies.

Register the bundle in AppKernel.php

```php
# /app/AppKernel.php

if ('test' == $this->getEnvironment()) {
    $bundles[] = new Infinity\Bundle\TestBundle\InfinityTestBundle();
}
```

The bundle must be executed in the test environment. The best way to ensure that this happens is to create the file app_test.php under the web folder and use that as entry point for your tests.  
You can copy and paste the following code.

```php
<?php
# /web/app_test.php

use Symfony\Component\HttpFoundation\Request;

$loader = require_once __DIR__.'/../app/bootstrap.php.cache';

require_once __DIR__.'/../app/AppKernel.php';

$kernel = new AppKernel('test', true);
$kernel->loadClassCache();

$request = Request::createFromGlobals();
$response = $kernel->handle($request);
$response->send();
$kernel->terminate($request, $response);
```

## Configuring the bundle
### Configuring behat/mink-bundle

The Test bundle install also [behat/mink-bundle](https://github.com/Behat/MinkBundle) which is used to run web acceptance tests and functional tests.   
Mink-bundle have is own configuration that must be set up in order to use the bundle. Please follow the documentation at [https://github.com/Behat/MinkBundle](https://github.com/Behat/MinkBundle)  
[https://github.com/Behat/MinkBundle/blob/master/Resources/doc/index.rst#bundle-installation--setup]
(https://github.com/Behat/MinkBundle/blob/master/Resources/doc/index.rst#bundle-installation--setup)

### Configuring Behat

If you are not using Incenteev\ParameterHandler as described above, once the bundle is installed copy behat.yml.dist under the app/config folder of your project and make another copy under the root of your project and rename it as behat.yml. You can find behat.yml.dist under TestBundle/Test/config.

The only parameter that you should replace in behat.yml is `base_url` in the MinkExtension configuration. Assign here the base url to your dev container  - followed by app_test.php if you have created that file -.

```
extensions:
    Behat\MinkExtension\Extension:
        base_url: 'http://url.to.my.dev.container/app_test.php'
```

Of course, you might want a more complex suites structure for your tests. In that case make all the required changes in behat.yml.dist and then copy them in behat.yml. Writing all basic configuration in behat.yml.dist is important to share them with the rest of the team, as behat.yml should be ignored by Git.  

Once you're happy with your configuraton you can run `behat --init` to create the folder and file structure.

### Configuring services substitutions

The Infinity Test Bundle allows to replace services in the test environment. This can be very useful to replace elements that cannot be
mocked and cannot be used directly as they are.
The bundle allows to replace them with classes that emulates the original class without the dependencies that make it
impossible to use them in your tests.

The `substitutions` key in the configuration of the bundle is used to list all service to replace.

```yaml
infinity_test:
    substitutions:
        servicename: { class: 'namespace\of\the\class', inherit_arguments: true }
```

Every substitutions is composed with `servicename` the name of the service to replace, `class` the name of the new class to use.
`inherit_arguments` is an optional parameter that can be omitted (default to true) which indicates if the arguments passed to the
original service should be passed to the new class. If you don't need them, just set it to false.

If you don't need to change the value of `inherit_arguments` you can define substitutions using only the name of the class

```yaml
substitutions:
    servicename: 'namespace\of\the\class'
```

### Configuring recipients

When an error occours the bundle try to send an email with a screenshot attached.   
You can specify the email address of the recipients using the `recipients` key in the bundle configuration

```yaml
# /app/config/config_test.yml
infinity_test:
    recipients:
         - email1@example.com
         - email2@example.com
```

This value can be overwrtitten in parameters.yml, which is useful if you want to limit the submission only to your email address when working locally.

```yaml
# /app/config/parameters.yml
infinity_test:
    recipients:
         - myemail@example.com
```

## What the bundle provides
The bundle take care of installing of the basic tools you might need to create behavioural, functional and unit tests.

It provides basic configuration files for Behat and Phpspec that you can then modify as your need.

But it provides also more...

### Utilities
Infinity Test Bundle installs 3 files under the bin folder of your project.    
Files are:

* `start_selenium.sh` which starts Xvfb and Selenium2
* `stop_selenium.sh` which stops Xvfb and Selenium2
* `run_test.sh` which start and stop Xvfb and Selenium and run all Behat, PHPUnit and Phpspec test in your project.

It's important that you launch `run_test.sh` using `. bin/run_test.sh` or `source bin/run_test.sh` otherwise part of what the script does won't be correctly executed.

### Behat base context
Infinity Test Bundle provides a basic context `Infinity\Bundle\TestBundle\Test\Context\BaseContext` that can be extended in your context files.   
BaseContext take care of the `@db` tag, handling the life cycle of the DB. All you need to do is to use one of this tag in your scenarios or steps.  
BaseContext take also care of getting a screenshot if a step fails and send an email to the addresses you have written in the configuration (if any).  
BaseContext extends RawMinkContext and implements KernelAwareContext. So extending this class you get automatically access to Mink and a kernel.

###Helpers
Infinity Test Bundle includes to helper classes `Infinity\Bundle\TestBundle\Test\Helper\DatabaseHelper` and `Infinity\Bundle\TestBundle\Test\Helper\ScreeshotHelper` that can be used to implement those functionalities directly in your PHPUnit and Behat tests.

###Traits
Infinity Test Bundle includes also two traits. The first one `Infinity\Bundle\TestBundle\Test\Doctrine\DoctrineTrait` can be used in Behat and PHPUnit to handle the DB lifecycle.  
The second one `Infinity\Bundle\TestBundle\Test\Doctrine\ScreenshotHelper` can be used with PHPUnit to get screenshots on failure if and only if `selenium2` is the driver used by Mink.

###PHPUnit base test case
Infinity User Bundle includes also two test case to be used with PHPUnit. The fist one `Infinity\Bundle\TestBundle\Test\BaseTestCase` handle the "get screenshot on failure  feature and can be extended directly in your test cases.   
The second one `Infinity\Bundle\TestBundle\Test\DBTestCase` extends `BaseTestCase`, thus it offers the same functionalities in addition to the ability to handle the DB lifecyle using the `setUp` and `teardown` phpunit hooks to create and delete the DB.

---
# Further readings
[Howto create phpsepc tests](howto_create_phpspec_tests.md)   
[Testing with a DB](howto_write_tests_that_rely_on_a_database.md)   