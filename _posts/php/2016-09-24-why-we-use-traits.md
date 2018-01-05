---
layout: post
title:  "Why we use traits?"
slug: 'why-to-use-traits'
date:   2016-09-24 12:01:06 +0300
categories: php
tags: [trait, php, modern php]
icon: code
---

Since PHP 5.4.0 a new concept was introduced: *Traits*
Many developers I know are skeptical about traits and don't fully understand why and when we should use them.
I know that there are tones of posts about this topic but I'll try to explain it one more time maybe it will help somebody.

### What is a trait?

As many other programming languages, PHP uses inheritance hierarchy. This means that starting from a base class, you can extend and inherit it's behavior in the specialized classes. But PHP can only support single inheritance and sometimes it would be beneficial if you could inherit from multiple classes to prevent code duplications. This is one thing that traits can solve for us gracefully. <br>

I offen say that a trait is exactly like a *twig partial* that you can use over and over, only that's for PHP classes.
A trait behave like a class and looks like an interface. You can use a trait into different classes in order to extend their behavior without duplicating your code. Remember the DRY acronym (Don't repeat yourself) which is considered a good practice to get into.

If it still doesn't clicks for you, I always belive an example is better then theory so imagine this. You have two classes Car and House, that are very different classes and couldn't share the same parent class. However, you have to make both classes geocodable for display them on a map.

1. First thing first. You could try to solve the geocodable problem by creating a common parent class Geocodable and makeing both Car and House classes to inherit that base class. This would be a very bad solution because it forces these two unrelated classes to share the same parent which normaly wouldn't be the case.
2. Another way you could solve the issue is by creating a interface Geocodable that will be implemented by both classes. This is a little bit better than the other solution because allows each class to be independent but doesn't adhere to the DRY principle, each class should duplicate the implementation for geocodable interface.
3. Traits are created exactly for this kind of scenario. They offer you a way to inject/extend unrelated classes with a specific code snippet.
The best solution here would be to create a Geocodable trait with all the necesary methods. This way you could inject this trait in whatever class you need, if later on you'll have a Phone class that you want to display on the map you could do it in seconds. Just make the Phone class use the Geocodable trait and you're done.

### Let's see how we can create a trait
```
trait FirstTrait {
    // add some methods here
}
```
I want to continue with our example and actualy show you how to use the trait in concrete classes.

```
trait Geocodable {

    protected $address;
    protected $geocoder;

    public function setAddress($address) {
        $this->address = $address;

        return $this;
    }

    public function setGeocoder(\Geocoder\GeocoderInterface	$geocoder)
                {
        $this->geocoder = $geocoder;

        return $this;
    }

    public function getLatitude()
    {
        return $this->geocodeAddress()->getLatitude();
    }

    public function getLongitude()
    {
        return $this->geocodeAddress()->getLongitude();
    }

    protected function geocodeAddress()
    {
        return $this->geocoder->geocode($this->address);
    }
}
```
Now use the trait into our classes:

```
class Car {
    use Geocodable;

    // other car proprieties and methods
}
```

```
class House {
    use Geocodable;

    // other house proprieties and methods
}
```

Finaly let's see how we can use the trait methods from the `House` class.

### Example 
```
<?php
$geocoderAdapter = new	\Geocoder\HttpAdapter\CurlHttpAdapter();
$geocoderProvider = new	\Geocoder\Provider\GoogleMapsProvider($geocoderAdapter);
$geocoder = new	\Geocoder\Geocoder($geocoderProvider);

$house = new House();
$house->setAddress('126-128 Ciurchi, Iasi, Romania');
$house->setGeocoder($geocoder);
$houseLatitude = $house->getLatitude();
$houseLongitude = $house->getLongitude();

$car = new Car();
$car->setAddress('Nicolae Iorga nr. 100, Iasi, Romania');
$car->setGeocoder($geocoder);
$carLatitude = $car->getLatitude();
$carLongitude = $car->getLongitude();

echo sprintf('The house is located at: %s Latitude - %s Longitude.', $houseLatitude, $houseLongitude);
echo sprintf('The car is located at: %s Latitude - %s Longitude.', $carLatitude, $carLongitude);
```

### Conclusion

As you saw in our small example traits are an awesome tool to have in your belt. You don't have to be afraid of using them when it's the case. When you don't know if you should create a trait instead of inheriting a base class or creating an interface try to answer these 2 simple questions:

1. Are those classes related and should share a common class that does what I need to do?
2. Does my code adhere to DRY and KISS principles?

If the answer is "No" maybe you should consider having a trait.

That's all for now. Thanks for reading and don't hesitate to leave a comment.
