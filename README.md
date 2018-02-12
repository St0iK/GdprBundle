# SpecShaper GDPR Bundle

A bundle to aid with the General Data Protection Regulation requirements. 

Features include:

- Written for Symfony verison 3.x.x
- Provides annotation for adding to entity parameter doc blocks
- Records values for Data Protection Impact Assesments of entity parameters.
- Uses SpecShaper\EncryptBundle to encrypt senstive data

## Version History

### Version 1
Version 1of this project used annotations to classify entity parameter personal data.  
This unfortunatly could not be extended to managing live data, it runs into problems where data
become expired. What should get displayed instead? How can live data status be reported with annotations?

Version 1 Features:
- [x] Generate a entity parameter coverage report.
- [x] Generate a summary report of all entity parameters and GDPR annotations.

### Version 2 
Version 2 will use a new PersonalData entity to store the GDPR parameters associated with the personal data parameter.

A custom twig function will then be used to:
- Decrypt any encrypted data
- Display current data in its correct format.
- Display deleted/aggregated/annonymised data once it has been sanitised

Version 2 Features:

- [ ] Create a storage entity
- [ ] Create twig templates for entity to handle displaying expired data.
- [ ] Create a migration command to create new database fields, and convert PersonalData attributes to PersonalData entity rows.
- [ ] Create disposal classes and service
- [ ] Create a command to dispose of data
- [ ] Implement a cron task to dispose of data
- [ ] Generate activity report
- [ ] Create consent forms
- [ ] Generate consent report
- [ ] Export data command

**Warning**
- This bundle has not been unit tested.
- It has only been running on a Symfony2 v3.0.1 project, and not backward
compatibility tested.

## Documentation

The source of the documentation is stored in the `Resources/doc/` folder
in this bundle.

## License

This bundle is under the MIT license. See the complete license in the bundle:

    Resources/meta/LICENSE

## About

GdprBundle has been written for the [SpecShaper](http://about.specshaper.com) and [Parolla](http://parolla.ie) websites
to encode users private data.

## Reporting an issue or a feature request

Issues and feature requests are tracked in the [Github issue tracker](https://github.com/mogilvie/HelpBundle/issues).

When reporting a bug, it may be a good idea to reproduce it in a basic project
built using the [Symfony Standard Edition](https://github.com/symfony/symfony-standard)
to allow developers of the bundle to reproduce the issue by simply cloning it
and following some steps.

# Installation

## Step 1: Download the bundle

Open a command console, enter your project directory and execute the
following command to download the latest version of this bundle:

```
$ composer require specshaper/gdpr-bundle dev-master
```

This command requires you to have Composer installed globally, as explained
in the [installation chapter](https://getcomposer.org/doc/00-intro.md)
of the Composer documentation.

## Step 2: Enable the bundle

Then, enable the bundle by adding it to the list of registered bundles
in the `app/AppKernel.php` file of your project:

```php
<?php
// app/AppKernel.php

// ...
class AppKernel extends Kernel
{
    public function registerBundles()
    {
        $bundles = array(
            // ...
            new SpecShaper\GdprBundle\SpecShaperGdprBundle(),
        );
        // ...
    }
    // ...
}
```

## Step 2: Configure the bundle

Geneate a 256 bit 32 character key using the command tool in the Encrypt bundle
```
$ bin/console encrypt:genkey
```

Add your encryption key to the parameters file.

```yaml
// app/config/parameters.yml

    ...
    encrypt_key: <your_key_here>
    
```

Configure the EncryptBundle to use the GdprBundle encryption subscriber. Or point to your own custom subscriber.
```yaml
// app/config/config.yml

    ...
    spec_shaper_encrypt:
        is_disabled: false
        subscriber_class: 'SpecShaper\GdprBundle\Subscribers\GdprSubscriber'

```   
You can disable encryption of the database by setting deleting is_disabled or setting it true.

Configure the routing to access the reports:  
```yaml
// app/config/routing.yml

    ...
    spec_shaper_gdpr:
        resource: "@SpecShaperGdprBundle/Controller/"
        type:     annotation
        prefix:   /gdpr

```
You should make soure that the /gdpr path is behind a firewall in your security settings.  


## Step 3: Create the entities
Add the Annotation entity to the declared classes in the entity.

```php
<?php
...
use SpecShaper\GdprBundle\Annotations\PersonalData;
```

Add the annotation '@PersonalData' to the parameters that you want to record.

```
    /**
     * @var \DateTime
     *
     * @PersonalData(
     *     isSensitive=false,
     *     isEncrypted=true,
     *     identifiableBy="Can be associated using other records",
     *     providedBy="The employee, revenue, the employer",
     *     purposeFor="Used to check the employee is not under age",
     *     retainFor="6 years",
     *     disposeBy="Aggregate into decades"
     * )
     *
     * @ORM\Column(type="string", nullable=true)
     */
    protected $dateOfBirth;
   
```

For DateTime parameters store the date as a string, and use the getters and setters
to convert that string.

You may also need to create a DataTransformer if you are using the parameter in a form
with the DateType formtype.


## Step 4: Decrypt in templates

If you query a repository using a select method, or get an array result 
then the doctrine onLoad event subscriber will not decyrpt any encrypted
values.

In this case, use the twig filter to decrypt your value when rendering.

```
{{ employee.bankAccountNumber | decrypt }}
```

## Step 5: Reportings

### Coverage Report

Access the coverage report by navigating your browser to '\gdpr\reporting\coverage'.      
This will serve an excel file that contains all the entities and parameters managed by the entity manager.
If any of the parameters contain the PersonalData attribute then it will also list each of the attributes values.

Note that at the moment we are only pulling information from the default entity manager. I need to
improve the coverage report to get all entityManagers.
