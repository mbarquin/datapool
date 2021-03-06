DataPool
========

Introduction
------------
This library is intended to be used with PHPUnit tool. DataPool is an iterable object which can be
returned to a standard @dataprovider tagged function or it can be used to get specific datasets from a big datapool.

* It allows keeping low weight indexed array values separated from tests logic.
* It can return specific datasets attending to its dataset index.
* It can encapsulate dataset as an array to avoid large parameters lists.

Installation
------------

You can install the component in the following ways:

* Use the official Github repository (https://github.com/mbarquin/datapool)
* Use composer : composer require mbarquin/datapool --dev

Usage
-----

The main use purpose is via heritage, the final class will only contains a definition
index and a dataArray, we can instance it into our test case as a dataprovider or as
a normal object which vill provide us with predefined test cases.

There are more use cases in the example file /example/tests/src/exampleContactsModelTest.php

### Custom dataPool Example: ###

        /**
         * Contacts datapool file for Testing purposes
         */
        class ContactsDataPool extends \DataPool\DataPool
        {

            /**
             * @var array Datapoolm fields definition to be merged with data
             */
            protected $definition = array(
                'name',
                'surname',
                'phone',
                'result'
            );

            /**
             * @var array Datasets array to be merged with definition and returned to tests
             */
            public $dataArray = array(
                'Test1'                  => array('Jack', 'Travis', '555999666', true),
                'Test2'                  => array('Mathew', 'Jones', '555888555', true),
                'Test3.NameSurnameEmpty' => array('', '', '555666555', false),
                'TestCase1.PhoneToLong'  => array('Gregor', 'Jones', '5550005518899', false),
                'TestCase2.NoName'       => array('', 'Lock', '555000559', false)
            );

        }// End ContactsDataPool.

Protected array $definition is defined to avoid data indexes being duplicated
along all defined datasets. Datasets can be returned with this $definition values as indexes, we can set up
this behaviour with the function __setReturnIndexes(false|true)__, by default setted to FALSE.

Public array $dataArray will contain an array with all possible tests datasets, it can
be classified by index to allow returning smaller portions of this dataArray via getRowsByIndex($index).


###Tests file example:###


    namespace Example\tests\src;

    class exampleContactsModelTest extends \PHPUnit_Framework_TestCase {

        static private $dataPool = null;

        /**
         * Static instance of ContactsDataPool
         *
         * @return \Example\tests\files\ContactsDataPool
         */
        public function getDataPool() {
            // If not instanciated yet...
            if (is_object(self::$dataPool) === false) {
                self::$dataPool = new \Example\tests\files\ContactsDataPool();
            }
            // Avoid mixing behaviors during tests
            self::$dataPool->setReturnArray(false);
            self::$dataPool->setReturnIndexes(false);

            return self::$dataPool;
        }


Function __getDataPool()__ is a dataprovider which sets and returns an iterable object. It
can be used in any common case as standard @dataprovider function's return, it's
configured to avoid any index usage as a usual PHPUnit dataprovider.

We have set:

 __setReturnArray(false)__ in order to get each dataset value as a different test function parameter.

__setReturnIndexes(false)__ to avoid returned dataset indexation with $definition
values (array_combine)

<br><br>

        public function getDataPoolAsArray() {
            $dataPool = $this->getDataPool();
            $dataPool->setReturnArray(true);
            $dataPool->setReturnIndexes(true);

            return $dataPool->getRowsByIndex('Case');
        }

Function __getDataPoolAsArray()__ returns an iterable dataset from previously instanced
dataProvider object. We want to get only test cases indexed by "Case" so we must
return the result array from __getRowsByIndex('Case')__ function.

 We have changed:

 __setReturnArray(true)__ in order to get each dataset fields encapsulated in an array.

__setReturnIndexes(true)__ to force the indexation of each returned dataset with $definition
values (array_combine)

<br><br>

        /**
         * @dataProvider getDataPoolAsArray
         */
        public function testArrayDataProviderInsert($regData) {
            $contModel = new \Example\src\exampleContactsModel();
            $expected  = $regData['result'];
            unset($regData['result']);

            $result = $contModel->insert($regData);
            if ($result === false) {
                $this->assertEquals($expected, $result);
            } else {
                $this->assertTrue(is_integer($result));
            }
        }

As in __testArrayDataProviderInsert__, with data encapsulated as array we can easily perform tests on ORM objects or avoid
large parameters lists in tests with many data values.

<br><br>

        /**
         * @dataProvider getDataPool
         */
        public function testPureDataProviderInsert($name, $surname, $phone, $expected) {
            $contModel      = new \Example\src\exampleContactsModel();

            $reg['name']    = $name;
            $reg['surname'] = $surname;
            $reg['phone']   = $phone;

            $result = $contModel->insert($reg);
            if ($result === false) {
                $this->assertEquals($expected, $result);
            } else {
                $this->assertTrue(is_integer($result));
            }
        }

    }// End exampleContactsModelTest.

Function testPureDataProviderInsert makes assertions using ContactsDataPool object
as a usual dataprovider array.
