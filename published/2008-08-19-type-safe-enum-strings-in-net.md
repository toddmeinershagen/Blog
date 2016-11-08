# Type-Safe Enum Strings in .NET

One thing that has always bothered me in .NET is the inability to create a type-safe set of string constants like an Enum. I would like to create a type such as the following:

```csharp
public enum StoredProcedure : string  
{  
   DeleteConsumer = DeleteConsumer,  
   EditConsumer = EditConsumer,  
   GetConsumer = GetConsumer  
}
```

This would be incredibly useful for those situations where you are passing constant strings to a given method and you would like to limit the parameter that is passed in to a finite set of options that can be detected through a type-safe check during compile time.

```csharp
public void ExecuteDataSet(StoredProcedures storedProcedure)  
{  
   SqlCommand command = new SqlCommand();  
   command.CommandType = CommandType.StoredProcedure;  
   command.CommandText = storedProcedure.ToString();  
}
```

Unfortunately, a simple constant does not provide type-safe protection for the method call. If a developer is not aware of the pattern, they may end up sending in a literal string of their choosing. So, instead you end up with the following:

```csharp
public void ExecuteDataSet(string storedProcedure)  
{  
   SqlCommand command = new SqlCommand();  
   command.CommandType = CommandType.StoredProcedure;  
   command.CommandText = storedProcedure.ToString();  
}
```

After working at a heterogeneous shop, I learned that Java has been creating their own type-safe string Enum classes for solving this situation for years before the formal Enum type was added to the 1.5 release of the Java Runtime.

The example below shows how a typical Java developer would implement this concept in Java syntax.

```java
public final class Color {  

    private String name;  

    private Color(String name) {  
        this.name = name;  
    }  

    public String toString() {  
        return this.name;  
 }  

 public static final Color Red = new Color("Red");  
 public static final Color Green = new Color("Green");  
 public static final Color Blue = new Color("Blue");  
}
```

The basic idea is simple:  

1. Define a class representing a single element of the enumerated type

1. Don't provide any public constructors for it.

1. Provide public static final fields, one for each constant in the enumerated type.

Because there is no way for clients to create objects of the type, there will never be any objects of the type besides those exported via the public static final fields.

In order to make this easier to use within .NET, I created an abstract base type, ```StringConstant```, that allows a developer to quickly create the functionality above plus some other goodies such as ```CompareTo()```, ```Equals()```, ```GetHashCode()```, and the == and != operators for comparison to string literals.

In order to use the code, you merely define your class as follows:

```csharp
namespace Shared
{
    public class StoredProcedures : StringConstant 
    {  
        public static readonly StoredProcedures GetConsumer = new StoredProcedures("GetConsumer");  
        public static readonly StoredProcedures EditConsumer = new StoredProcedures("EditConsumer");  
        public static readonly StoredProcedures DeleteConsumer = new StoredProcedures("DeleteConsumer");  
    
        private StoredProcedures(name) : base(name){}  
    }
}
```

The only part that does not work (this is the same for the Java version) is that it cannot be used in switch...case statements. Those require the use of an underlying integral type which is not present. Instead, you should use the if...else if construct to perform the same logic.

Below is the code for the base class for StringConstant.

```csharp
using System;
using System.Collections;
using System.Reflection;

namespace Shared
{
    public abstract class StringConstant : IComparable
    {
        private readonly string _value;

        protected StringConstant()
        {
        }

        protected StringConstant(string value)
        {
            _value = value;
        }

        public string Value
        {
            get { return _value; }
        }

        public override string ToString()
        {
            return Value;
        }

        public override bool Equals(object obj)
        {
            var otherValue = obj as StringConstant;

            if (otherValue == null)
            {
                return false;
            }

            var typeMatches = GetType().Equals(obj.GetType());
            var valueMatches = _value.Equals(otherValue.Value);

            return typeMatches && valueMatches;
        }

        public override int GetHashCode()
        {
            return _value.GetHashCode();
        }

        public virtual int CompareTo(object other)
        {
            return Value.CompareTo(((StringConstant)other).Value);
        }
    }
}
```

In addition to the class, you may use the following xunit tests to explore my assumptions on its use.  

**Note:** These tests require the [xunit](https://www.nuget.org/packages/xunit/) and the [FluentAssertions](https://www.nuget.org/packages/FluentAssertions/) libraries to run properly.

```csharp
using FluentAssertions;
using Shared;
using Xunit;

namespace Tests
{
    public class StringConstantTests
    {
        [Fact]
        public void GIVEN_instance_WHEN_getting_value_THEN_returns_value() 
        {
            StoredProcedures.EditConsumer.Value.Should().Be("EditConsumer");
        }
        
        [Fact]
        public void GIVEN_instance_WHEN_to_stringing_THEN_returns_value()
        {
            StoredProcedures.DeleteConsumer.ToString().Should().Be("DeleteConsumer");
        }

        [Fact]
        public void GIVEN_instance_WHEN_comparing_with_an_equal_instance_THEN_returns_0()
        {
            StoredProcedures.DeleteConsumer.CompareTo(StoredProcedures.DeleteConsumer).Should().Be(0);
        }

        [Fact]
        public void GIVEN_instance_WHEN_comparing_with_a_non_equal_instance_THEN_return_negative_1()
        {
            StoredProcedures.DeleteConsumer.CompareTo(StoredProcedures.EditConsumer).Should().Be(-1);
        }

        [Fact]
        public void GIVEN_instance_WHEN_checking_if_equal_to_an_equal_instance_THEN_returns_true()
        {
            StoredProcedures.DeleteConsumer.Equals(StoredProcedures.DeleteConsumer).Should().BeTrue();
            (StoredProcedures.GetConsumer == StoredProcedures.GetConsumer).Should().BeTrue();
            (StoredProcedures.GetConsumer != StoredProcedures.DeleteConsumer).Should().BeTrue();
        }

        [Fact]
        public void GIVEN_instance_WHEN_checking_if_equal_to_a_non_equal_instance_THEN_returns_false()
        {
            StoredProcedures.DeleteConsumer.Equals(StoredProcedures.EditConsumer).Should().BeFalse();
            (StoredProcedures.GetConsumer == StoredProcedures.EditConsumer).Should().BeFalse();
            (StoredProcedures.GetConsumer != StoredProcedures.GetConsumer).Should().BeFalse();
        }

        [Fact]
        public void GIVEN_enum_type_WHEN_getting_names_THEN_returns_every_instance()
        {
            var items = StringConstant.GetNames(typeof(StoredProcedures));
            items.Should().Contain(StoredProcedures.DeleteConsumer);
            items.Should().Contain(StoredProcedures.EditConsumer);
            items.Should().Contain(StoredProcedures.GetConsumer);
        }

        [Fact]
        public void GIVEN_instance_WHEN_getting_hashcode_THEN_return_hash_of_value()
        {
            StoredProcedures.DeleteConsumer.GetHashCode().Should().Be("DeleteConsumer".GetHashCode());
            StoredProcedures.EditConsumer.GetHashCode().Should().Be("EditConsumer".GetHashCode());
            StoredProcedures.GetConsumer.GetHashCode().Should().Be("GetConsumer".GetHashCode());
        }
    }
}
```

> **Update:**  Jimmy Bogard has [an excellent description](https://lostechies.com/jimmybogard/2008/08/12/enumeration-classes/) of this same concept and more complete base class for Enumeration that you might want to leverage in your own code.

I hope this helps someone out there...