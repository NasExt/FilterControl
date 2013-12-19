NasExt/FilterControl
===========================

Data filter control for Nette Framework.

Requirements
------------

NasExt/FilterControl requires PHP 5.3.2 or higher.

- [Nette Framework 2.0.x](https://github.com/nette/nette)

Installation
------------

The best way to install NasExt/FilterControl is using  [Composer](http://getcomposer.org/):

```sh
$ composer require nasext/filter-control:@dev
```


## Usage

```php
class FooPresenter extends Presenter
{

	public function renderDefault()
	{
		/** @var NasExt\Controls\FilterControl $filter */
		$filter = $this['filter'];
		$filterData = $filter->getData(); // data for filter
	}


	/**
	 * @return NasExt\Controls\FilterControl
	 */
	protected function createComponentFilter($name)
	{
		$control = new NasExt\Controls\FilterControl($this, $name);

		$control->setDefaultValues(array('code' => 'some default value for code input'));

		/** @var  Form $form */
		$form = $control['form'];

		$form->addText('code', 'Code');
		$form->addText('name', 'Name');

		return $control;
	}
}
```


###Use dataFilter callback for filtering data in persistent param in url
If you need use special filter input use object in input, like Nette\Datetime,
so you must translate this object to timestamp before set persistent parameter for url, and after load data from persistent parameter
you need transform timestamp to Nette\Datetime object. For this process use setDataFilter().
- FILTER_IN: use when transform object to string in process set persistent parameter for url
- FILTER_OUT: use when transform string from persistent parameter to object
```php
	/**
	 * @return NasExt\Controls\FilterControl
	 */
	protected function createComponentFilter($name)
	{
		$control = new NasExt\Controls\FilterControl($this, $name);

		$control->setDataFilter(function ($values, $filterType) {
			if (array_key_exists('date', $values) && !empty($values['date'])) {
				if ($filterType == FilterControl::FILTER_IN) {
					if ($values['date'] instanceof DateTime) {
						$values['date'] = $values['date']->getTimestamp();
					}else{
						$values['date'] = strtotime($values['date']);
					}
				} elseif ($filterType == FilterControl::FILTER_OUT) {
					if (!$values['date'] instanceof DateTime) {
						$date = new DateTime();
						$values['date'] = $date->setTimestamp($values['date']);
					}
				}
			}

			return $values;
		});

		/** @var  Form $form */
		$form = $control['form'];

		$form->addText('date', 'Date');

		return $control;
	}
}
```

###FilterControl with ajax
For use FilterControl with ajax use setAjaxRequest() and use events onFilter[], onReset[] for invalidateControl
```php
	/**
	 * @return NasExt\Controls\FilterControl
	 */
	protected function createComponentFilter($name)
	{
		$control = new NasExt\Controls\FilterControl($this, $name);
		// enable ajax request, default is false
		$control->setAjaxRequest();

		$that = $this;
		$invalidateControl = function ($component, $values) use ($that) {
			if ($that->isAjax()) {
				$that->invalidateControl();
			}
		};

		$control->onFilter[] = $invalidateControl;
		$control->onReset[] = $invalidateControl;

		/** @var  Form $form */
		$form = $control['form'];

		$form->addText('code', 'Code');
		$form->addText('name', 'Name');

		return $control;
	}
```

###Set templateFile for FilterControl
For set templateFile use setTemplateFile()
```php
	/**
	 * @return NasExt\Controls\FilterControl
	 */
	protected function createComponentFilter($name)
	{
		$control = new NasExt\Controls\FilterControl($this, $name);
		$control->setTemplateFile('myTemplate.latte');

		/** @var  Form $form */
		$form = $control['form'];

		$form->addText('code', 'Code');
		$form->addText('name', 'Name');

		return $control;
	}
```

-----

Repository [http://github.com/nasext/filtercontrol](http://github.com/nasext/filtercontrol).