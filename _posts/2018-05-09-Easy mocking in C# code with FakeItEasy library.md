---
title: "Easy mocking in C# code with FakeItEasy library"
excerpt: "In this article I would like to present mocking library called Fake It Easy but before I present some code, let me do some introduction about terms from unit tests dictionary."
---

<p align="center">
<img src="/images/devisland/article6/assets/fakeit11.png?raw=true" alt="Easy mocking in C# code with FakeItEasy library"/>
</p>

&nbsp;
<h3><strong>Short introduction</strong></h3>
Unit tests are inseparable element of software development cycle. They enable testing functionality of the program and bug detection on early stage of implementation. In this article I would like to present mocking library called Fake It Easy but before I present some code, let me do some introduction about terms from unit tests dictionary.
<h3><strong>Unit tests dictionary</strong></h3>
<ul>
 	<li><strong>Unit of source code</strong> - the smallest piece of a code that can be tested. In .NET C# it is usually method or class</li>
 	<li><strong>Unit tests</strong> - process through which units of source code are tested to verify if they work properly and are free of bugs</li>
 	<li><strong>Mocking</strong>  - process used in unit testing when the unit being tested has external dependencies (cannot work properly without external source code). In .NET C# example can be object of one class which depends on another object from different class. These dependency objects are called "replacement objects"</li>
</ul>
There are four types of "replacement objects" according to <a href="https://martinfowler.com/articles/mocksArentStubs.html" target="_blank" rel="noopener">Martin Flower's article:</a>
<ul>
 	<li><strong>Dummies </strong>-  objects are passed around but never actually used. Usually they are just used to fill parameter lists in method call</li>
 	<li><strong>Fakes</strong> - have the same behavior as the thing that it replaces. They have working implementations, but usually take some shortcut which makes them not suitable for production (an in memory database is a good example)</li>
 	<li><strong>Stubs</strong> -  objects which have a "fixed" set of "canned" responses that are specific to test you setup. They have hard-coded responses to an expected request</li>
 	<li><strong>Mocks</strong> - have a set of expectations about calls that are made. If these expectations are not met, the test is fail. A mock is not setup in a predetermined way so you have code that does it in your test</li>
</ul>
Tests written with mocks usually follow below pattern:

<em>initialize -&gt; set expectations -&gt; exercise -&gt; verify </em>

Stub would follow below pattern:

<em>initialize -&gt; exercise -&gt; verify.</em>

...and one more thing: Mocks != Stubs
<h3><strong>FakeItEasy library</strong></h3>
<a href="http://fakeiteasy.readthedocs.io/en/stable/" target="_blank" rel="noopener">FakeItEasy</a> is an easy mocking library for .NET which enables creating all types of fake objects, mocks and stubs. I would like to present some cool features of it.

<strong>Create Unit Tests project in Visual Studio</strong>

<img class=" wp-image-515 aligncenter" src="/images/devisland/article6/assets/fakeit12.png?w=300" alt="" width="531" height="326" />

FakeItEasy can be used with different testing frameworks. In this case I will use MS Test framework. Rename inital class to "FakeItEasyTests":

```csharp
   [TestClass]
    public class FakeItEasyTests
    {
        [TestInitialize]
        public void TestInitialize()
        {

        }

        [TestMethod]
        public void TestMethod1()
        {
        }
    }
```

Before we start using FakeItEasy I would like to shortly describe class attributes:
<ul>
 	<li><strong>TestClass</strong> - attribute which indicated class that contains test methods</li>
 	<li><strong>TestInitialize</strong> - method which is invoked before each test</li>
 	<li><strong>TestMethod</strong> - method which contains test code and assertions</li>
</ul>
&nbsp;

<strong>Add FakeItEasy NuGet package</strong>

FakeItEasy library is available through NuGet packages manager. Install it:

<img class=" wp-image-521 aligncenter" src="/images/devisland/article6/assets/fakeit13.png?w=300" alt="" width="523" height="225" />

Project is ready! Now its time to see some nice mocking features provided by FakeItEasy.

<strong>FakeItEasy library in action</strong>

Below I pasted some examples how to use library and how easily you can prepare mocks in the unit tests.

<strong>Create fake objects</strong>

Creating fake objects with is simple. FakeItEasy enables creating fake objects from:
<ul>
 	<li>interfaces</li>
 	<li>classes that are not sealed, static and have at least one public or protected constructor whose arguments FakeItEasy can construct or obtain
delegates</li>
</ul>
Now if fake is created its methods and properties can be overridden under condition that they are:
<ul>
 	<li>virtual</li>
 	<li>abstract</li>
 	<li>an interface method when an interface is being faked</li>
</ul>
Let me show simple example. There are four classes:
<ul>
 	<li>Car</li>
 	<li>PetrolStation</li>
 	<li>Distributor</li>
 	<li>Transaction</li>
</ul>
Car class additionally implements IVehicle interface.

```csharp
    public class Car : IVehicle
    {
        private string _plateNumber;
        public string PlateNumber => _plateNumber;

        public Car(string plateNumber)
        {
            _plateNumber = plateNumber;
        }

        public void CheckEngineStatus()
        {
            Console.WriteLine("Engine OK");
        }

        public void OpenFuelTank()
        {
            Console.WriteLine("Fuel tank opened");
        }
    }
```


```csharp
public class PetrolStation
    {
        public Distributor GetDistributorForVehicle(string plateNumber)
        {
            return new Distributor();
        }
    }
```


```csharp
 public class Distributor
    {
        public virtual decimal getFuelPrice()
        {
            return 4.50M;
        }

        public void RefuelTheVehicle(IVehicle vehicle)
        {
            vehicle.OpenFuelTank();
        }
    }
```


```csharp
 public class Transaction
    {
        private bool _isCompleted;
        public decimal getFuelTotalPrice(Distributor distributor, int liters)
        {
            return liters * distributor.getFuelPrice();
        }

        public virtual void Finish()
        {
            _isCompleted = true;
        }
    }
```


```csharp
    public interface IVehicle
    {
        void CheckEngineStatus();
        void OpenFuelTank();
    }
```

Now I can easily create fakes in the tests:

```
var distributor = A.Fake<Distributor>();
var car = A.Fake<Car>(x => x.WithArgumentsForConstructor(() => new Car("WB 82112")));
var petrolStation = A.Fake<PetrolStation>();
var transaction = A.Fake<Transaction>();
var vehicle = A.Fake<IVehicle>();
```

Please note that I can create fake object from concrete class or from interface - car and vehicle.

Now it is possible to set specific expectations what should happened when invoking concentrate code:

```
A.CallTo(() => distributor.getFuelPrice()).MustHaveHappenedOnceExactly();
A.CallTo(() => transaction.Finish()).DoesNothing();
```

In above fragment of code I set expectations that distributor's "getFuelPrice" method will be called once and "Finish" method call on transaction will have not result or impact.

I can also configure exceptions expectations:

```csharp
   A.CallTo(() => vehicle.CheckEngineStatus()).Throws<NotImplementedException>();

            try
            {
                vehicle.CheckEngineStatus();
            }
            catch (Exception ex)
            {
                var exceptionType = ex.GetType();
                Assert.AreEqual(exceptionType, typeof(NotImplementedException));
            }
```

Whole sample test method can look like below:

```csharp
 [TestMethod]
        public void TestMethod1()
        {
            var distributor = A.Fake<Distributor>();
            var car = A.Fake<Car>(x => x.WithArgumentsForConstructor(() => new Car("WB 82112")));
            var petrolStation = A.Fake<PetrolStation>();
            var transaction = A.Fake<Transaction>();
            var vehicle = A.Fake<IVehicle>();

            transaction.getFuelTotalPrice(distributor, 4);

            A.CallTo(() => distributor.getFuelPrice()).MustHaveHappenedOnceExactly();
            A.CallTo(() => transaction.Finish()).DoesNothing();

            A.CallTo(() => vehicle.CheckEngineStatus()).Throws<NotImplementedException>();

            try
            {
                vehicle.CheckEngineStatus();
            }
            catch (Exception ex)
            {
                var exceptionType = ex.GetType();
                Assert.AreEqual(exceptionType, typeof(NotImplementedException));
            }
        }
```

FakeItEeasy provides easy way to create Dummies too so you can use them as a constructor parameters:

```
distributor.RefuelTheCar(A.Dummy<Car>());
```

It is also possible to specify return values for method calls:

```
A.CallTo(() => distributor.getFuelPrice()).Returns(4.5M).Once();
```

This is just a small piece of features that FakeItEasy offers!
<h3><strong>Wrapping up</strong></h3>
In this article I explained differences between dummies, fakes, mocks and stubs. I presented great mocking library for .NET C# called FakeItEasy. I encourage you to go through official documentation available <a href="https://fakeiteasy.github.io/" target="_blank" rel="noopener">here</a> to see more extra features. Pre-configured sample is available on my <a href="https://github.com/Daniel-Krzyczkowski/NetCsharp/tree/master/FakeItEasySamples" target="_blank" rel="noopener">GitHub</a>.