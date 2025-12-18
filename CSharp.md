# Fixed decimal
```c#
int var1 = -123456;
var1.ToString("N2", CultureInfo.CreateSpecificCulture("fi-FI"));  // -123 456,00
var1.ToString("F2", CultureInfo.CreateSpecificCulture("fi-FI"));  // -123456,00
```

# Attribute

Attribute 类将预定义的系统信息或用户定义的自定义信息与目标元素相关联
Attributes 是一种新的描述信息，我们既可以使用 attributes 来定义设计期信息（例如 帮助文件，文档的 URL ），还可以用attributes 定义运行时信息（例如，使 XML 中的元素与类的成员字段关联起来）。我们也可以用 attributes 来创建一个“自描述”的组件

属性所提供的信息也称为元数据。元数据可由应用程序在运行时进行检查以控制程序处理数据的方式，也可以由外部工具在运行前检查以控制应用程序处理或维护自身的方式


特性，用于程序集、类、方法、属性、事件、字段、参数、返回值等的自描述，以便于其他程序通过反射查找这些信息

```cs
using  System; 
using  System.Reflection; 
using  System.Diagnostics; 
 
// attaching Help attribute to entire assembly 
[assembly : Help( " This Assembly demonstrates custom attributes creation and their run - time query. " )] 
 
// our custom attribute class 
public class HelpAttribute : Attribute 
{ 
    public HelpAttribute(String Description_in) 
    { 
        // 
        //  TODO: Add constructor logic here 
        this .description  =  Description_in; 
        // 
    } 
    protected  String description; 
    public  String Description 
    { 
        get 
        { 
            return this.deescription; 
        }             
    }     
} 
 
// attaching Help attribute to our AnyClass 
[HelpString( "This is a do-nothing Class." )] 
public class AnyClass 
{ 
    // attaching Help attribute to our AnyMethod 
    [Help( "This is a do-nothing Method." )] 
    public void AnyMethod() 
    { 
    } 
    // attaching Help attribute to our AnyInt Field 
    [Help( "This is any Integer." )] 
    public int  AnyInt; 
} 
 
class  QueryApp 
{ 
    public   static   void  Main() 
    { 
    } 
} 
```
```cs
class  QueryApp 
{ 
    public   static   void  Main() 
    { 
        Type type  =   typeof (AnyClass); 
        HelpAttribute HelpAttr; 
        // Querying Class Attributes 
        foreach  (Attribute attr  in  type.GetCustomAttributes( true )) 
        { 
            HelpAttr  =  attr  as  HelpAttribute; 
            if ( null != HelpAttr) 
            { 
                Console.WriteLine( " Description of AnyClass:\n{0} " , elpAttr.Description); 
            } 
        } 
        // Querying Class-Method Attributes   
        foreach (MethodInfo method  in  type.GetMethods()) 
        { 
            foreach  (Attribute attr  in  method.GetCustomAttributes(true)) 
            { 
                HelpAttr  =  attr  as  HelpAttribute; 
                if (null != HelpAttr) 
                { 
                    Console.WriteLine( " Description of {0}:\n{1} " , method.Name, HelpAttr.Description); 
                } 
            } 
        } 
        // Querying Class-Field (only public) Attributes 
        foreach (FieldInfo field  in  type.GetFields()) 
        { 
            foreach  (Attribute attr  in  field.GetCustomAttributes(true))
            { 
                HelpAttr =  attr  as  HelpAttribute; 
                if ( null != HelpAttr) 
                { 
                    Console.WriteLine( " Description of {0}:\n{1} " , field.Name,HelpAttr.Description); 
                } 
            } 
        } 
    } 
} 
```
```
The output of the following program is. 
Description of AnyClass:
This is a do-nothing Class.
Description of AnyMethod:
This is a do-nothing Method.
Description of AnyInt:
This is any Integer.
Press any key to continue
```
# this
## Extension Methods
Extension methods enable you to "add" methods to existing types without creating a new derived type, recompiling, or otherwise modifying the original type. Extension methods are static methods, but they're called as if they were instance methods on the extended type. For client code written in C#, F# and Visual Basic, there's no apparent difference between calling an extension method and the methods defined in a type.
```c#
namespace ExtensionMethods
{
    public static class MyExtensions
    {
        public static int WordCount(this string str)
        {
            return str.Split(new char[] { ' ', '.', '?' },
                             StringSplitOptions.RemoveEmptyEntries).Length;
        }
    }
}

using ExtensionMethods;
string s = "Hello Extension Methods";
int i = s.WordCount();

string s = "Hello Extension Methods";
int i = MyExtensions.WordCount(s);
```

# ref
The `ref` keyword indicates that a variable is a reference, or an alias for another object. It's used in five different contexts:

-   In a method signature and in a method call, to pass an argument to a method by reference. For more information, see [Passing an argument by reference](https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/keywords/ref#passing-an-argument-by-reference).
-   In a method signature, to return a value to the caller by reference. For more information, see [Reference return values](https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/keywords/ref#reference-return-values).
-   In a member body, to indicate that a reference return value is stored locally as a reference that the caller intends to modify. Or to indicate that a local variable accesses another value by reference. For more information, see [Ref locals](https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/keywords/ref#ref-locals).
-   In a `struct` declaration, to declare a `ref struct` or a `readonly ref struct`. For more information, see the [`ref struct`](https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/builtin-types/ref-struct) article.
-   In a `ref struct` declaration, to declare that a field is a reference. See the [`ref` field](https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/builtin-types/ref-struct) article.

# to binary
```c#
int value = 8;
string binary = Convert.ToString(value, 2);
```

# Array
## Array from 0 to x
```csharp
int[] arr = Enumerable.Range(0, X+1).ToArray();
```

## Set a Array
```c#
class TestArraysClass
{
    static void Main()
    {
        // Declare a single-dimensional array of 5 integers.
        int[] array1 = new int[5];

        // Declare and set array element values.
        int[] array2 = new int[] { 1, 3, 5, 7, 9 };

        // Alternative syntax.
        int[] array3 = { 1, 2, 3, 4, 5, 6 };

        // Declare a two dimensional array.
        int[,] multiDimensionalArray1 = new int[2, 3];

        // Declare and set array element values.
        int[,] multiDimensionalArray2 = { { 1, 2, 3 }, { 4, 5, 6 } };

        // Declare a jagged array.
        int[][] jaggedArray = new int[6][];

        // Set the values of the first array in the jagged array structure.
        jaggedArray[0] = new int[4] { 1, 2, 3, 4 };
    }
}
```

## Sort
```c#
Array.Sort(array, Comparer<KilowattSnapshot>.Create((k1, k2) => k1.Kilowatt.CompareTo(k2.Kilowatt)));
List<Part> parts = new List<Part>();
parts.Sort(delegate(Part x, Part y)
{
	if (x.PartName == null && y.PartName == null) return 0;
	else if (x.PartName == null) return -1;
	else if (y.PartName == null) return 1;
	else return x.PartName.CompareTo(y.PartName);
});
```
## ToString
```c#
var result = String.Join(", ", names);
```

# Parse
## To string
```csharp
string a = i.ToString();
string b = Convert.ToString(i);
string c = string.Format("{0}", i);
string d = $"{i}";
string e = "" + i;
string f = string.Empty + i;
string g = new StringBuilder().Append(i).ToString();
// List to string
string h = String.Join<int>("", list);
```
## TryParse
```c#
public static bool TryParse (string? s, System.Globalization.NumberStyles style, IFormatProvider? provider, out int result);
bool success = int.TryParse("80c1", NumberStyles.HexNumber, new CultureInfo("en-US"), out int number);
if (success)
 Console.WriteLine($"Converted '{stringToConvert}' to {number}.");	//       Converted '80c1' to 32961.
else
 Console.WriteLine($"Attempted conversion of '{stringToConvert}' failed.");

//------
int tmp;
int.TryParse("", out tmp);
Console.WriteLine(tmp.ToString());  // 0

```
## Convert
```c#
int number = Convert.ToInt32(value, 16);
```
### To Binary
```c#
int value = 8;
string binary = Convert.ToString(value, 2);	// 1000
```

# Dictionary
## GetOrCreate
```c#
public static TValue GetOrCreate<TKey, TValue>(this IDictionary<TKey, TValue> dict, TKey key) 
    where TValue : new()
{
    if (!dict.TryGetValue(key, out TValue val))
    {
        val = new TValue();
        dict.Add(key, val);
    }

    return val;
}
```

# Language Integrated Query (LINQ)
```cs
// Specify the data source.
int[] scores = { 97, 92, 81, 60 };

// Define the query expression.
IEnumerable<int> scoreQuery =
    from score in scores
    where score > 80
    select score;

// Execute the query.
foreach (int i in scoreQuery)
{
    Console.Write(i + " ");
}

// Output: 97 92 81
```


# where (generic type constraint) (C# Reference)
The where clause in a generic definition specifies constraints on the types that are used as arguments for type parameters in a generic type, method, delegate, or local function.  
For example, you can declare a generic class, AGenericClass, such that the type parameter T implements the IComparable<T> interface:

```C#
public class AGenericClass<T> where T : IComparable<T> { }
```
## new constraint
The new constraint specifies that a type argument in a generic class or method declaration must have a public parameterless constructor. To use the new constraint, the type cannot be abstract.
```c#
class ItemFactory<T> where T : new()
{
    public T GetNewItem()
    {
        return new T();
    }
}
```


# String
## Split
```C#
string s = "You win some. You lose some.";
char[] separators = new char[] { ' ', '.' };

string[] subs = s.Split(separators, StringSplitOptions.RemoveEmptyEntries);
```
## Substring
```C#
string str1 = "abcd";
Console.WriteLine(str1.Substring(0,2));
// ab
```

# Swap
```c#
int a = 10;
int b = 2;
(a, b) = (b, a);
```
