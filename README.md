# C#
.NET C#

- Value types (int, float, long, double etc) are stored on the stack because they are small, fixed-size, and short-lived.
- Reference types (class, object, string, array etc) interface are stored on the heap because they are larger, dynamic, and shared across the program.
  
| Stack                              | Heap                         |
| ---------------------------------- | ---------------------------- |
| Fast memory                        | Slower than stack            |
| Automatically managed              | Managed by Garbage Collector |
| Stores method calls & local values | Stores objects               |
| Fixed size                         | Dynamic size                 |

- CTS (Common type system) = how data types are declared, used, and managed in the .NET Framework.
- CLS (Common language specification) = set of rules that .NET languages must follow to ensure language interoperability.

![swimlane-architecture-framework](https://github.com/user-attachments/assets/72d75789-4439-4b56-ad20-901a3cd90420)
