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

```
ViewBag cannot directly insert data into a Model.
Model binding in MVC works only with:

Form fields
Query string
Route values

ViewBag is only for sending data from Controller → View, not for creating or filling a Model automatically.

So this will not work:
ViewBag.Name = "Amit";
Student s = ViewBag;   // ❌ Not possible
```

### 7️⃣ Client-side & Server-side Validation
- Client-side → Runs in browser (fast, user-friendly)
- Server-side → Runs on server (secure, mandatory
```
In View:
@Html.ValidationMessageFor(m => m.Name)

In Controller:
[HttpPost]
public IActionResult Create(Student s)
{
    if (ModelState.IsValid)
    {
        return RedirectToAction("Success");
    }
    return View(s);
}
```

### 8️⃣ Self-Validated Mode
- Model handles its own validation using IValidatableObject.
```
Model handles its own validation using IValidatableObject.

public class Student : IValidatableObject
{
    public int Age { get; set; }

    public IEnumerable<ValidationResult> Validate(ValidationContext context)
    {
        if (Age < 18)
        {
            yield return new ValidationResult("Age must be 18+");
        }
    }
}
```

### 9️⃣ Strongly Typed Views

- View bound to a model.
```
@model Student
<h2>@Model.Name</h2>
<h3>@Model.Age</h3>


Controller:
public IActionResult Details()
{
    Student s = new Student { Name = "Raj", Age = 22 };
    return View(s);
}
```

### 🔟 Scaffold Templates

Scaffolding auto-generates:
```
Controller
Views (Create, Edit, Delete, Details, Index)

CRUD code
Steps:
Right Click Controller → Add → New Scaffolded Item
→ MVC Controller with views using Entity Framework
```

### 1️⃣1️⃣ CRUD using Model

- CRUD = Create, Read, Update, Delete
```
// Create
public IActionResult Create(Student s) { }

// Read
public IActionResult Index() { }

// Update
public IActionResult Edit(int id) { }

// Delete
public IActionResult Delete(int id) { }

```
- With Scaffold, all these are generated automatically.

<hr>

## MVC State Management 

- In MVC, State Management is used to store and transfer data between requests, pages, or users.
- Because HTTP is stateless, we use different techniques to maintain data.

### Server-Side State Management

| Technique       | Scope        | Lifetime                 | Use                              |
| --------------- | ------------ | ------------------------ | -------------------------------- |
| **ViewBag**     | Same request | Only for current request | Pass data from Controller → View |
| **TempData**    | Next request | Till it is read          | Redirect scenarios               |
| **Session**     | Per user     | Until session expires    | Store user-specific data         |
| **Application** | All users    | App lifetime             | Global shared data               |

1. *ViewBag*
ViewBag.Msg = "Hello";
- Works only in the same request.

2.*TempData*
TempData["Name"] = "Amit";
- return RedirectToAction("Next");
Used when redirecting between actions.
```
ViewBag / ViewData → work only for the same request
TempData → works across requests, especially after Redirect
That is why TempData is used with RedirectToAction().
```

3. *Session*
```
HttpContext.Session.SetString("User", "Admin");
string u = HttpContext.Session.GetString("User");
```
- Stores data for a particular user.

4. *Application*
- Application (Global) in ASP.NET refers to application-level events and data that are common for all users and all requests of the web app.
- In classic ASP.NET this is handled in Global.asax.
- In ASP.NET Core, the same idea is implemented in Program.cs / Middleware.
- It is used for:
```
Application start / end events
Global error handling
Session start / end
Storing application-wide data
```
```
Classic ASP.NET (Global.asax)
public class MvcApplication : System.Web.HttpApplication
{
    protected void Application_Start()
    {
        // Runs once when app starts
    }

    protected void Session_Start()
    {
        // Runs when a new user session begins
    }

    protected void Application_Error()
    {
        // Global error handling
    }

    protected void Application_End()
    {
        // Runs when app stops
    }
}

Application State (Global Data)
Application["TotalUsers"] = 0;


This value is:
Shared by all users
Exists for entire lifetime of the application

Compare with others:
| Scope       | Shared With        | Lifetime     |
| ----------- | ------------------ | ------------ |
| ViewBag     | Same request       | One request  |
| TempData    | Next request       | Until read   |
| Session     | One user           | User session |
| Application | All users (Global) | App lifetime |

So Application (Global) is for data and events that belong to the whole website, not to a single user.
```

### 🔹 Client-Side State Management
- Client-side state management means storing and managing data on the user’s browser (client) instead of the server.
- This data remains available while the user is interacting with the web page or site.
| Technique       | Stored On      | Use                |
| --------------- | -------------- | ------------------ |
| **Cookies**     | Client browser | Store small data   |
| **QueryString** | URL            | Pass values in URL |

1. 🍪 Cookies
- Cookies store small data in the user’s browser and are sent with every request.
- Use: Login info, User preferences, Remember me feature
```
Set Cookie (Controller):
Response.Cookies.Append("UserName", "Amit");

Read Cookie:
string name = Request.Cookies["UserName"];

Cookies can have expiry:
Response.Cookies.Append("UserName", "Amit",
    new CookieOptions { Expires = DateTime.Now.AddDays(7) });
```

*Features:*
- Stored in browser
- Can persist for days/months
- Automatically sent with every request
- Limited size (~4KB)

2. 🔗 Query String
- Query String sends data in the URL.
```
Example URL: /Student/Details?id=10&name=Amit

Read in Controller:
public IActionResult Details(int id, string name)
{
    // id = 10, name = Amit
    return View();
}

Or:
string id = Request.Query["id"];
```
*Features:*
- Visible in URL
- Temporary (only for that request)
- Easy to pass small values between pages
- Not secure
```
Summary Line:
ViewBag / TempData / Session / Application → Server-side
Cookies / QueryString → Client-side
```
