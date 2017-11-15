---
title: Form Type Twig Extension
summary: Twig Extension to render a form type.
categories: Symfony
tags: [transformer, months, featured]
lastmod:  
links:
 - http://symfony.com/doc/current/templating/twig_extension.html
---

{{ page.summary }}<!--more-->

**FormTypeTwigExtension** 
The twig extension. 

```php
<?php

namespace Calva\ExamplesBundle\Twig;

use Symfony\Component\Form\FormFactory;
use Twig_Extension;
use Twig_SimpleFunction;

class FormTypeTwigExtension extends Twig_Extension
{
    private $formFactory;

    public function __construct(FormFactory $formFactory)
    {
        $this->formFactory = $formFactory;
    }

    public function getFunctions()
    {
        return [
            new Twig_SimpleFunction('form_type', [$this, 'formType']),
        ];
    }

    public function formType($formType)
    {
        return $this->formFactory->create($formType)->createView();
    }

    public function getName()
    {
        return 'form_type_extension';
    }
}
```

**FormTypeTwigExtensionTest** 
Unit tests.

```php
<?php

namespace Calva\ExamplesBundle\Twig;

use Calva\ExamplesBundle\Twig\FormTypeTwigExtension;
use Symfony\Bundle\FrameworkBundle\Test\KernelTestCase;
use Symfony\Component\Form\FormView;
use Twig_Extension;
use Twig_SimpleFunction;

class FormTypeTwigExtensionTest extends KernelTestCase
{
    /** @var FormTypeTwigExtension */
    private $sut;

    protected function setUp()
    {
        self::bootKernel();

        $this->sut = self::$kernel->getContainer()->get('twig.extension.form_type');
    }

    public function testShouldBeInstanceOfTwigExtension()
    {
        $this->assertInstanceOf(Twig_Extension::class, $this->sut);
    }

    public function testShouldReturnInstanceOfFormView()
    {
        $this->assertInstanceOf(
            FormView::class,
            $this->sut->formType('Calva\ExamplesBundle\Form\ExampleFormType')
        );
    }

    public function testShouldReturnOneTwigFunction()
    {
        $this->assertEquals(1, count($this->sut->getFunctions()));
        $this->assertInstanceOf(Twig_SimpleFunction::class, $this->sut->getFunctions()[0]);
    }

    public function testShouldReturnCorrectName()
    {
        $this->assertSame('form_type_extension', $this->sut->getName());
    }
}
```
