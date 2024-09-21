# Fixed decimal
```c#
int var1 = -123456;
var1.ToString("N2", CultureInfo.CreateSpecificCulture("fi-FI"));  // -123 456,00
var1.ToString("F2", CultureInfo.CreateSpecificCulture("fi-FI"));  // -123456,00
```
