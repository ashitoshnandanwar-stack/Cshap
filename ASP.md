# *ASP.net*

##  Understanding Controllers and Actions
A Controller is a C# class that handles user requests.
An Action is a public method inside a controller.
Every request from browser is mapped to a Controller → Action.
Eg.

public class HomeController : Controller
{
    public IActionResult Index()
    {
        return View();
    }

    public IActionResult About()
    {
        return Content("About Page");
    }
}
Here:

HomeController → Controller
Index() and About() → Actions

### How Actions Are Invoked
- Actions are invoked using URL routing: /ControllerName/ActionName


### HttpGet, HttpPost, NoAction Attributes
- These attributes control how and when an action runs.

*HttpGet*
- Used to display a page/form.
```
[HttpGet]
public IActionResult Register()
{
    return View();
}
```

*HttpPost*
- Used to submit form data.
```
[HttpPost]
public IActionResult Register(string name)
{
    return Content("Welcome " + name);
}
```

*NoAction*
- Prevents a method from being treated as an action.
```
[NonAction]
public string GetMessage()
{
    return "Hello";
}

This method cannot be called via URL.
```

### Running Action Result
- Every action returns an IActionResult.

Common results:
| Method               | Use                  |
| -------------------- | -------------------- |
| `View()`             | Show a page          |
| `Content()`          | Return plain text    |
| `RedirectToAction()` | Go to another action |
| `Json()`             | Return JSON data     |

Which one is default?
When you do not apply any attribute to an action method, it is treated as:
✅ HttpGet by default

Example:
```
public IActionResult Index()
{
    return View();
}
This method is automatically a GET action, even though [HttpGet] is not written.
```

<hr>

## Understanding Views & Models 

### 1️⃣ Models & ViewModel

*Model*
A Model represents business data.
Eg.
```
public class Student
{
    public int Id { get; set; }
    public string Name { get; set; }
    public int Age { get; set; }
}
```

*ViewModel*
A ViewModel is used when a view needs combined or customized data.
```
Copy code
public class StudentVM
{
    public Student Student { get; set; }
    public string Title { get; set; }
}
```
Model → Database entity
ViewModel → Data specially prepared for View


### 2️⃣ Creating Razor Views
- Razor view = .cshtml file
```
@{
    ViewData["Title"] = "Home";
}

<h2>Welcome</h2>
<p>This is Razor View</p>
```
Razor allows mixing C# + HTML:
```
<h3>@DateTime.Now</h3>
```

### 3️⃣ HTML Helper Functions
- They generate HTML easily.
```
@Html.TextBox("name")
@Html.Password("pwd")
@Html.Label("Username")
@Html.ActionLink("Home", "Index")
```
In form:
```
@using (Html.BeginForm())
{
    @Html.TextBox("name")
    <input type="submit" />
}
```

### 4️⃣ ViewBag
- ViewBag passes data from Controller to View (dynamic).
```
Controller:

public IActionResult Index()
{
    ViewBag.Message = "Welcome Student";
    return View();
}

View:
<h2>@ViewBag.Message</h2>
```

### 5️⃣ Create View using ViewBag
```
Controller:
public IActionResult Info()
{
    ViewBag.Name = "Amit";
    ViewBag.Age = 20;
    return View();
}

View:
<h3>Name: @ViewBag.Name</h3>
<h3>Age: @ViewBag.Age</h3>
```

### 6️⃣ Validation using Data Annotations
```
In Model:

using System.ComponentModel.DataAnnotations;
public class Student
{
    [Required]
    public string Name { get; set; }

    [Range(18, 60)]
    public int Age { get; set; }

    [EmailAddress]
    public string Email { get; set; }
}
```
- These rules are used for server + client validation.

### 7️⃣ Client-side & Server-side Validation
- Client-side → Runs in browser (fast, user-friendly)
- Server-side → Runs on server (secure, mandatory)
