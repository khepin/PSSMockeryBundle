﻿PSSMockeryBundle
================

[![Build Status](https://secure.travis-ci.org/PolishSymfonyCommunity/PSSMockeryBundle.png?branch=master)](http://travis-ci.org/PolishSymfonyCommunity/PSSMockeryBundle)

Symfony2 Mockery integration bundle. Currently it supports service mocking.

Installation
------------

In your deps file add:

    [Mockery]
        git=https://github.com/padraic/mockery.git
        target=/mockery

    [PSSMockeryBundle]
        git=https://github.com/PolishSymfonyCommunity/PSSMockeryBundle.git
        target=/bundles/PSS/Bundle/MockeryBundle


Usage
-----

Add a namespace into the `app/autoload.php`::

    $loader->registerNamespaces(array(
        // ...
        'PSS'     => __DIR__.'/../vendor/bundles',
        'Mockery' => __DIR__.'/../vendor/mockery/library',
    ));

    // Mockery class has no namespace and therefore needs fallback
    
    $loader->registerPrefixes(array(
        //--
        'Mockery'                        => __DIR__ . '/../vendor/mockery/library',
    ));

Replace base container class for test environment in `app/AppKernel.php`::

    /**
     * @return string
     */
    protected function getContainerBaseClass()
    {
        if ('test' == $this->environment) {
            return '\PSS\Bundle\MockeryBundle\DependencyInjection\MockerContainer';
        }

        return parent::getContainerBaseClass();
    }
    
Clear your cache 

To use it with Behat enable sub-context in your `FeatureContext` class::

    /**
     * @param \Symfony\Component\HttpKernel\HttpKernelInterface $kernel
     *
     * @return null
     */
    public function __construct(HttpKernelInterface $kernel)
    {
        parent::__construct($kernel);

        $this->useContext('container', new \PSS\Bundle\MockeryBundle\Behat\Context\MockerContainerContext($kernel));
    }

Example story::

    Feature: Submitting contact request form
      As a Visitor
      I want to contact sales
      In order to receive more information

      Scenario: Submitting the form
        When I go to "/contact-us"
         And I complete the contact us form with following information
           |First name|Last name|Email                |
           |Jakub     |Zalas    |jzalas+spam@gmail.com|
         And CRM API is available
         And I submit the contact us form
        Then the contact request should be sent to the CRM

Step definitions::

    /**
     * @Given /^CRM API is available$/
     *
     * @return null
     */
    public function crmApiIsAvailable()
    {
        $this->getMainContext()->getSubContext('container')
            ->mockService('crm.client', 'PSS\Crm\Client')
            ->shouldReceive('send')
            ->once()
            ->andReturn(true);
    }

    /**
     * @Given /^(the )?contact request should be sent to (the )?CRM$/
     *
     * @return null
     */
    public function theContactRequestShouldBeSentToCrm()
    {
        return new Then(sprintf('the "%s" service should meet my expectations', 'crm.client'));
    }

All the expectations are checked automatically with an `@afterScenario` hook. 
Doing it manually only improves the readability of the scenario and gives better
error message.

To discuss
----------

* Does it have to be a bundle? Currently it's rather a Symfony independent library.
* Is this the right approach/implementation?
* Do we need more features?
