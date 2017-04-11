## The following is an easy example of polymorphism



```java
public abstract class Vehicle
{
    public abstract int Wheels;
}

public class Bicycle : Vehicle
{
    public override int Wheels()
    {
        return 2;
    }
}

public class Car : Vehicle
{
    public override int Wheels()
    {
        return 4;
    }
}

public class Truck : Vehicle
{
    public override int Wheels()
    {
        return 18;
    }
}

```

Now create the following in the Main\(\) of the module for the console application:

```java
public void Main()
{
    List<Vehicle> vehicles = new List<Vehicle>();

    vehicles.Add(new Bicycle());
    vehicles.Add(new Car());
    vehicles.Add(new Truck());

    for (Vehicle v : vehicles)
    {
        Console.WriteLine(
            string.Format("A {0} has {1} wheels.",
                v.GetType().Name, v.Wheels));
    }
```



