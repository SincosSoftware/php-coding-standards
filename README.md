# PHP Coding standards
Common PHP coding standards for use in our PHP projects.

## Setup

In a PHP project, do the following to set up the coding standards check:

#### Require the package
`composer require --dev sincos/php-coding-standards`

#### Add an empty tests output folder
```
mkdir tests/_output
touch tests/_output/.gitkeep
```

PHPCS and PHPMD will dump their reports in this folder.

#### Add phpcs.xml

Here is an example file:

```xml
<?xml version="1.0"?>
<ruleset name="Project name here">
    <config name="installed_paths" value="../../sincos/php-coding-standards/phpcs" />

    <rule ref="Sincos"/>

    <arg name="extensions" value="php"/>
    <file>./src</file>

    <!-- Rule customization/configuration example -->
    <rule ref="SlevomatCodingStandard.Files.TypeNameMatchesFileName">
        <properties>
            <property name="rootNamespaces" type="array">
                <element key="src" value="Sincos\Bring"/>
                <element key="tests" value="Sincos\Bring\Tests"/>
            </property>
        </properties>
    </rule>
</ruleset>
```

#### Add phpmd.xml

```xml
<?xml version="1.0"?>
<ruleset name="Project name here"
         xmlns="http://pmd.sf.net/ruleset/1.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://pmd.sf.net/ruleset/1.0.0 http://pmd.sf.net/ruleset_xml_schema.xsd"
         xsi:noNamespaceSchemaLocation="http://pmd.sf.net/ruleset_xml_schema.xsd">
    <description>
        A nice description here
    </description>

    <exclude-pattern>vendor</exclude-pattern>

    <rule ref="vendor/sincos/php-coding-standards/phpmd/ruleset.xml"/>
</ruleset>
```

#### Add a Makefile
This is optional, but makes the commands easier to run with sensible defaults:

After adding this to `Makefile`, you can run for instance `make lint`:

```make
.DEFAULT_GOAL := help
.PHONY: help

help:
	@awk 'BEGIN {FS = ":.*?## "} /^[a-zA-Z_-]+:.*?## / {printf "\033[36m%-30s\033[0m %s\n", $$1, $$2}' $(MAKEFILE_LIST) | sort

phpcs: ## Check PHP Code Sniffer rules. (Optional parameter PARAMS="-additional --params")
	vendor/bin/phpcs --standard=phpcs.xml -d date.timezone=Europe/Oslo --report=junit --report-file=tests/_output/phpcs.xml -d memory_limit=512M $(PARAMS) -s

phpmd: ## Check PHP Mess Detector rules.
	vendor/bin/phpmd . xml phpmd.xml --suffixes php --reportfile tests/_output/phpmd.xml -d memory_limit=512M

phpcbf: ## Run PHPCBF to fix code
	vendor/bin/phpcbf .

lint: ## Run phpcs and phpmd (Optional parameter "FIX=true" added phpcbf on start)
ifdef FIX
	-make phpcbf
endif
	make phpcs
	make phpmd
```

#### Add all this sauce to your CI

Here is an example from another project using Circle CI, where we add the following to `.circleci/config.yml`:

```yml
# PHP CircleCI 2.0 configuration file
#
# Check https://circleci.com/docs/2.0/language-php/ for more details
#
version: 2
jobs:
  build:
    docker:
      - image: circleci/php:7.2.8-node-browsers

    steps:
      - checkout
      - run: sudo composer self-update
      - run: composer install
      - run: ./vendor/bin/phpunit
      - run: make lint
      - store_test_results:
          path: tests/_output
      - store_artifacts:
          path: tests/_output
```
