---
layout: post
title: Stored SQL function  
summary: Using a stored function to generate an alphanumeric sequence.
tags: [sql, stored function]
featured: true
---

#### Stored Function

```sql
DROP FUNCTION IF EXISTS generateSequence;
DELIMITER //
CREATE FUNCTION generateSequence (vname VARCHAR(30), vprefix VARCHAR(30), vpadding INT)
  RETURNS VARCHAR(50)
BEGIN
   UPDATE plt_sequence SET
     prefix = (@prefix := IF(next < end, prefix, vprefix)),
     next = (@next := IF(next < end, next, 0)) + increment
     WHERE name = vname;
   IF @next = 0 THEN
     SET @next := 1;
   END IF;
   RETURN CONCAT(@prefix, LPAD(@next, vpadding, '0'));
END
//
DELIMITER ;
```

#### Sequence Table Schema and Data 
 
```sql
DROP TABLE IF EXISTS `plt_sequence`;
CREATE TABLE `plt_sequence` (
  `id` int(11) UNSIGNED NOT NULL AUTO_INCREMENT,
  `name` varchar(30) NOT NULL,
  `prefix` varchar(30) NOT NULL,
  `next` int(11) UNSIGNED NOT NULL,
  `end` int(11) UNSIGNED NOT NULL,
  `increment` int(11) UNSIGNED NOT NULL,
  `created_at` timestamp NOT NULL,
  PRIMARY KEY (`id`),
  UNIQUE KEY (`name`)
);

INSERT INTO plt_sequence (name, prefix, next, end, increment)
VALUES ('product_model_code', 'CLW', 1, 10000, 1);
```

# ProductModelCodeHelper

```php
<?php

namespace AppBundle\Helper;

use AppBundle\Entity\Sequence;
use Doctrine\ORM\EntityManager;
use Doctrine\ORM\Query\ResultSetMapping;
use Exception;

class ProductModelCodeHelper
{
    const SEQUENCE_NAME = 'product_model_code';
    const SEQUENCE_PADDING = 4;

    /**
     * @var EntityManager
     */
    private $entityManager;

    public function __construct(EntityManager $em)
    {
        $this->entityManager = $em;
    }

    /**
     * @return string
     * @throws Exception
     */
    public function create(): string
    {
        if (!$code = $this->getNextModelCode()) {
            throw new Exception('Product Model sequence number could not be generated');
        }
        return $code;
    }

    /**
     * @return string
     * @throws Exception
     */
    private function getNextModelCode(): string
    {
        $nextPrefix = $this->getCurrentModelPrefix();
        do {
            $isPrefixUsed = $this->isPrefixAlreadyUsed(++$nextPrefix);
        } while ($isPrefixUsed == true);

        if (!$code = $this->generateSequence($nextPrefix)) {
            throw new Exception('Product Model code could not be created.');
        }
        return $code;
    }

    /**
     * @param string $nextPrefix
     * @return bool
     */
    private function isPrefixAlreadyUsed(string $nextPrefix): bool
    {
        return (bool) $this->entityManager->getRepository('AppBundle:ProductModelPrefixUsed')->findOneBy([
            'prefix' => $nextPrefix
        ]);
    }

    /**
     * @return string
     * @throws Exception
     */
    private function getCurrentModelPrefix(): string
    {
        /** @var Sequence $sequence */
        if (!$sequence = $this->entityManager->getRepository('AppBundle:Sequence')
            ->findOneBy(['name' => self::SEQUENCE_NAME])) {
            throw new Exception('Product Model prefix could not be found.');
        }
        return $sequence->getPrefix();
    }

    /**
     * @param string $nextPrefix
     * @return string
     * @throws \Doctrine\ORM\NoResultException
     * @throws \Doctrine\ORM\NonUniqueResultException
     */
    private function generateSequence(string $nextPrefix): string
    {
        $mapping = (new ResultSetMapping())->addScalarResult('result', 'result');
        $sql = 'select generateSequence(:sequenceName, :nextPrefix, :padding) as result';
        $query = $this->entityManager->createNativeQuery($sql, $mapping);
        $query->setParameter('sequenceName', self::SEQUENCE_NAME);
        $query->setParameter('nextPrefix', $nextPrefix);
        $query->setParameter('padding', self::SEQUENCE_PADDING);
        return $query->getSingleScalarResult();
    }
}
```

# Sequence Entity

```php
<?php
namespace AppBundle\Entity;

class Sequence
{
    /**
     * @var int
     */
    private $id;

    /**
     * @var string
     */
    private $name;

    /**
     * @var string
     */
    private $prefix;

    /**
     * @return int
     */
    public function getId(): int
    {
        return $this->id;
    }

    /**
     * @return string
     */
    public function getName(): string
    {
        return $this->name;
    }

    /**
     * @return string
     */
    public function getPrefix(): string
    {
        return $this->prefix;
    }
}
```