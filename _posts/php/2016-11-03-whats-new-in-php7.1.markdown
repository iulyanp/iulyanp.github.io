---
layout: post
title:  "Whats new in PHP 7.1"
slug: 'whats-new-in-php7.1'
date:   2016-11-03 12:01:06 +0300
categories: php
tags: [php, php7, modern php]
icon: code
---

Today I was looking on what's new in PHP 7.1 and I realized that things might be interesting also for you.
If you want to play arround you can go and download the Rasmuss Lerdorf's vagrant machine from [here](https://github.com/rlerdorf/php7dev).
Follow the installation steps and you'll be all set up.

### Symmetric Array Destructuring

A nice thing in PHP7.1 is symmetric array destructuring. A fancy name for something very simple that I think you already use in PHP.
Remember that you can do something like this in PHP, when you want to assign the values from an array to different variables?

```
<?php

$person = ['Iulian', 'Popa'];

list($firstname, $lastname) = $person;

var_dump($firstname, $lastname);
```

Now `$firstname` is equal to `Iulian` and `$lasname` is equal to `Popa`. But you already knew that and that's cool.

The new thing in PHP 7.1 is that you can use the short array syntax or you can continue to use `list` if you like.
But I personaly think that you'll rapidly adopt the new way because seems more natural.

```
<?php

$person = ['Iulian', 'Popa'];

[$firstname, $lastname] = $person;

var_dump($firstname, $lastname);
```

The same result as if you used `list` here.

But the good part is that it will work also with associative arrays. So let's see an example.

```
<?php

$person = ['first' => 'Iulian', 'last' => 'Popa'];

['first' => $firstname, 'last' => $lastname] = $person;

var_dump($firstname, $lastname);
```

Nice, right? But that's not all, you can use them in a foreach loop and do something like this.

```
<?php

$persons = [
	['first' => 'Rasmus', 'last' => 'Lerdorf'],
	['first' => 'Fabien', 'last' => 'Potencier'],
	['first' => 'Taylor', 'last' => 'Otwell'],
]

foreach ($persons as ['first' => $firstname, 'last' => $lastname]) {
	var_dump($firstname, $lastname);
}
```

Now you know all you need for destructuring arrays.

### Catching multiple exception types

The next item I wanna talk about is handling different exceptions type in the same way. 
I bet you did something like this in your projects and your code wasn't very clean beacause you had to duplicate the same code over and over for each exception type.

```
<?php

class ChargeRejected extends Exception {}
class InvalidCardNumber extends Exception {}
class AuthenticationError extends Exception {}

class Client {

	public function pay() {
		var_dump('Thank you for buying our book!');
	}
}

function notifyClient($message) {
	var_dump($message);
}

try {
	(new Client)->pay();
} catch (ChargeRejected $e) {
	notifyClient($e->getMessage());
} catch (InvalidCardNumber $e) {
	notifyClient($e->getMessage());	
} catch (AuthenticationError $e) {
	notifyClient($e->getMessage());	
}
```

So that's how your code would look like up until now. However in PHP 7.1 you will be able to do this is a more efficient way.

```
try {
	(new Client)->pay();
} catch (ChargeRejected | InvalidCardNumber | AuthenticationError $e) {
	notifyClient($e->getMessage());
}
```

So if you want to catch multiple exception types and handle them in the exact same way, all you need to know is to use the pipe notation.

### Support for constant visibility

Up until now you couldn't declare in PHP constants visibility. They were made by default as **public**.

```
class Book {
	
	private $id;

	public $title;

	const PAGES = 200; // This is by default public
}
```

In PHP 7.1 were introduced visibility modifiers for constants.
Let's see an example:

```
class Book {
	
	private $id;

	public $title;
 
	public const PAGES = 200; // This is by default public
	
	protected const PRICE = 1000;

	private const RELEASE_DAY = '01-01-2017';
}
```

### Nullable and Return types

Return types were added in PHP 7 and if you don't know about them as a reminder a look on the following code.

```
<?php

class Person {
	
	private $age;

	public function age() : int
	{
		return $this->age;
	}
}

$age = (new Person)->age();

var_dump($age);
```

In this case what I did with `: int` is to be explicit about what result type should the method `age` return. 
Of course this code will blow when you'll try to instanciate the class because the age is `null`. 
So what you can do if you would like to be explicit about the returned type?

```
<?php

class Person {
	
	private $age;

	public function age() : ?int
	{
		return $this->age;
	}
}

$age = (new Person)->age();

var_dump($age);
```

If you want to accept null as a valid return type all you have to do is to mark it with a `?` and you're all set up.

That’s all for now, thanks for reading. 
