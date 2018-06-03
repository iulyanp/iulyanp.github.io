---
layout: post
title:  "How to replace conditionals with polymorphism"
description: "How to replace conditionals with polymorphism and respect the O(open close) principle from SOLID"
slug: 'how-to-replace-conditionals-with-polymorphism'
date:   2018-06-03 12:00:00 +0300
category: clean-code
tags: [clean code, refactoring, open close, polymorphism]
icon: book
---

Let's imagine your boss is coming one day and asks you to write some code to calculate the area of a rectangle.
You say ok, that's easy, and you start first creating a class for a `Rectangle`. 

``` 
class Rectangle {

    public $width;
    public $height;
    
    public function __construct($width, $height) 
    {
        $this->width = $width;
        $this->height = $height;
    }
}
```  

Then because you know the principle from OOP (object oriented programming) that claims a class should do only one 
thing, you create another class for calculating the area for the rectangle. This class will be used as a service class
in your project.

``` 
class AreaCalculator {

    public function calculate($rectangle) 
    {
        return $rectangle->width * $rectangle->height;
    }
}
```

At this moment everything works perfectly, your code is running and your boss is happy. But after a while he tells you
that you have to calculate the area of a circle too. So you go and think how to do this, you first create the 
`Circle` class and then modify also the AreaCalculator.

``` 
class Circle {
    public $radius;
    
    public function __construct($radius) {
        $this->radius = $radius;
    }
}

class AreaCalculator {

    public function calculate($shape) 
    {
        if ($shape instanceof Rectangle) { 
            return $shape->width * $shape->height;
        } else {
           return $shape->radius * $shape->radius * pi();
        }
    }
}
```

At the end you make it, the code is doing what it supposed to do, your boss is happy again. But this isn't feeling 
right to you because you know you violated another principle of OOP, the Open Close principle. 

And also as you're expecting, your boss will tell you later that he needs also to calculate the area of a square. 
You know something doesn't smell good, and here polymorphism will help you. 

> Polymorphism is a pattern in OOP  which describes how classes can have different 
behavior while sharing a common interface.  

In another words, polymorphism just give us a way to adopt a common interface that we can reference without needing
to understand what happens behind the scenes.

### Implement polymorphism

In order to ensure that your classes do conform to the polymorphism principle, we can choose between one of the two 
options of either abstract classes or interfaces.

For our small example we will use an interface `Shape` which will be implemented by all shapes your boss needs and 
forces you to implement a method `calculateArea()`.

```
interface Shape {
  public function calculateArea();
}
```

The rectangle class implements the `Shape` interface by defining the `calculateArea()` method with the formula that 
calculates the area of a rectangle and the circle also implements the `Shape` interface but defining the 
`calculateArea()` method with the formula for calculating the area of a circle. The result of the polymorphism  
implementation on your classes looks like this.

```
class Rectangle implements Shape {

    public $width;
    public $height;
    
    public function __construct($width, $height) 
    {
        $this->width = $width;
        $this->height = $height;
    }
    
    public function calculateArea() {
        return $this->width * $this->height;
    }
}

class Circle implements Shape {
    public $radius;
    
    public function __construct($radius) {
        $this->radius = $radius;
    }
    
    public function calculateArea() {
        return $this->radius * $this->radius * pi();
    }
}
```

Now you can remove the if conditional from the `AreaCalculator` service and replace it with just one single line that
you'll never change when your boss will ask you to calculate the area of a new shape.

``` 
class AreaCalculator {

    public function calculate(Shape $shape) 
    {
        return $shape->calculateArea();
    }
}

$area = new AreaCalculator();
echo $area->calculate(new Rectangle(24, 40)); // 960
echo $area->calculate(new Circle(30)); // 2827.4333882308
```

So if your boss wants now to calculate the area of a square, all you have to do is to create the `Square` class 
that implements the shape interface. No more modifications on the `AreaCalculator` service. 

``` 
class Square implements Shape {
    public $wight;
    
    public function __construct($wight) {
        $this->wight = $wight;
    }
    
    public function calculateArea() {
        return $this->wight * $this->wight;
    }
}

echo $area->calculate(new Square(10)); // 100
```

### Conclusions
This is a very simple principle but a very powerful one. Your code will be so much cleaner and extensible when you're
conforming to the polymorphism principle. 
