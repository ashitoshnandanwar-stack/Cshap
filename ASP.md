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

<hr>
## MVC Module 

### 🔹 Partial View
- A Partial View is a reusable piece of UI (view) that can be embedded inside another view.
*Use cases:*
- Header, Footer
- Menu, Sidebar
- Login panel, Cart summary
```
Create a partial view:
File name starts with _ (convention)
_StudentInfo.cshtml

<h3>Student Panel</h3>
<p>Name: @ViewBag.Name</p>

Use it inside another view:
@Html.Partial("_StudentInfo")
or
<partial name="_StudentInfo" />
```
*Benefits:*
- Reusability
- Clean layout
- Easy maintenance

### 🔹 Action Method and Child Action
- An Action Method is a public method in a controller that responds to a request.
```
public class HomeController : Controller
{
    public IActionResult Index()
    {
        return View();
    }
}

URL: /Home/Index
```
*Child Action*
- A Child Action is an action that is not called directly by URL, but is invoked from a view.
```
public class HomeController : Controller
{
    public IActionResult Index()
    {
        return View();
    }

    [NonAction]
    public IActionResult Menu()
    {
        return PartialView("_Menu");
    }
}

In View: @Html.Action("Menu", "Home")
```
*Purpose:*
- Load dynamic content inside a view
- Combine Controller logic + Partial View
```
So:
Concept	Meaning
Partial View	Reusable UI block
Action Method	Handles URL request
Child Action	Action called from a View (not via URL)

Together, they help build modular and dynamic MVC pages.
```

<hr>

### Data Management with ADO.NET 
- ADO.NET is used in MVC (and all .NET apps) to connect, read, write, and manage data from databases like SQL Server.

*1️⃣ Microsoft.Data.SqlClient – Introduction*
- Microsoft.Data.SqlClient is the modern provider for SQL Server in .NET.
- It provides classes like: SqlConnection, SqlCommand, SqlDataReader, SqlDataAdapter
- Install via NuGet: Microsoft.Data.SqlClient
- Use namespace: using Microsoft.Data.SqlClient;

- These four classes belong to ADO.NET and are used to communicate with a SQL Server database : SqlConnection, SqlCommand, SqlDataReader, SqlDataAdapter
```
Application → SqlConnection → SqlCommand → (SqlDataReader / SqlDataAdapter) → Database

1. SqlConnection
Used to connect your application to the database.

SqlConnection con = new SqlConnection(
    "server=.;database=SchoolDB;trusted_connection=true");
con.Open();

Holds connection string
Opens and closes the DB connection
Without this, no DB operation is possible

2. SqlCommand
Used to execute SQL queries (INSERT, UPDATE, DELETE, SELECT).

SqlCommand cmd = new SqlCommand(
    "select * from Student", con);

Can execute:
ExecuteNonQuery() → INSERT, UPDATE, DELETE
ExecuteScalar() → Single value //return for select
ExecuteReader() → Read records //retrun for all

Example (Insert):
SqlCommand cmd = new SqlCommand(
    "insert into Student values('Amit',20)", con);
cmd.ExecuteNonQuery();

3️. SqlDataReader
Used to read data row by row (forward only, read-only).

SqlCommand cmd = new SqlCommand("select * from Student", con);
SqlDataReader dr = cmd.ExecuteReader();
while (dr.Read())
{
    Console.WriteLine(dr["Name"]);
}

Fast
Connected mode
Forward only

4️.  SqlDataAdapter
Used to fill DataSet/DataTable (disconnected mode).

SqlDataAdapter da = new SqlDataAdapter(
    "select * from Student", con);
DataTable dt = new DataTable();
da.Fill(dt);

Works in disconnected mode
Stores data in memory
Good for binding with GridView, DataGrid, etc.
```

*2️⃣ Core ADO.NET Objects*
🔹 Connection Object
```
Used to open a connection with database.
SqlConnection con = new SqlConnection(
    "Server=.;Database=TestDB;Trusted_Connection=True");
con.Open();
```
🔹 Command Object
```
Used to execute SQL queries.
SqlCommand cmd = new SqlCommand("select * from Student", con);
```
🔹 DataReader (Forward-only, Read-only)
```
SqlDataReader dr = cmd.ExecuteReader();
while (dr.Read())
{
    Console.WriteLine(dr["Name"]);
}

Fast
Connected mode
One record at a time
```
🔹 DataAdapter
```
Acts as a bridge between DB and DataSet.
SqlDataAdapter da = new SqlDataAdapter("select * from Student", con);
```
🔹 DataSet & DataTable (Disconnected Mode)
```
DataSet ds = new DataSet();
da.Fill(ds);
DataTable dt = ds.Tables[0];

Stores data in memory
Works even after connection is closed
Used in Grid, Reports, etc.
```

### 3️⃣ Asynchronous Command Execution
- Used to avoid blocking the application while DB work is running.
```
SqlCommand cmd = new SqlCommand("insert into Student values('Amit',20)", con);
await cmd.ExecuteNonQueryAsync();

Other async methods:
ExecuteReaderAsync()
ExecuteScalarAsync()

Benefits:
Better performance
UI remains responsive
Ideal for web apps
```
### 4️⃣ Asynchronous Connections
```
using (SqlConnection con = new SqlConnection(cs))
{
    await con.OpenAsync();

    SqlCommand cmd = new SqlCommand("select * from Student", con);
    SqlDataReader dr = await cmd.ExecuteReaderAsync();

    while (await dr.ReadAsync())
    {
        Console.WriteLine(dr["Name"]);
    }
}
```
| Component     | Purpose             |
| ------------- | ------------------- |
| SqlConnection | Connect to DB       |
| SqlCommand    | Execute SQL         |
| DataReader    | Fast, forward read  |
| DataAdapter   | Fill DataSet        |
| DataSet       | In-memory DB        |
| DataTable     | Table in DataSet    |
| Async Methods | Non-blocking DB ops |

<hr>

## Understanding Routing & Request Life Cycle

*1️⃣ Routing Engine & Routing Table*
- Routing Engine: The component that examines the incoming URL and decides which Controller and Action should handle it.
- Routing Table: A collection of URL patterns registered at application start.
- Example URL: /Student/Details/5
- Routing Engine matches it with a route and maps it to:
```
StudentController → Details(int id = 5)
```
So:
- Routing Engine = Decision maker
- Routing Table = Set of rules

*2️⃣ Routing Pattern in RouteConfig*
- In classic ASP.NET MVC:
```
routes.MapRoute(
    name: "Default",
    url: "{controller}/{action}/{id}",
    defaults: new 
    { 
        controller = "Home", 
        action = "Index", 
        id = UrlParameter.Optional 
    }
);
```
- This pattern allows:
```
/Home/Index
/Student/Create
/Student/Edit/10
```
Meaning:
- {controller} → Controller name
- {action} → Action method
- {id} → Optional parameter  <br>

This builds the Routing Table used by the Routing Engine.

*3️⃣ 404 Error – Resource Not Found*
- A 404 error occurs when MVC cannot find a match for the URL: <br>
No route matches the URL  <br>
Controller does not exist  <br>
Action method is missing  <br>
```
Example: /Home/Test
If Test() does not exist in HomeController, MVC returns:
404 – Not Found
So, 404 = Routing failed to find a valid Controller/Action.
```

*4️⃣ Attribute Routing*
- Routing is defined directly on Controllers or Actions.
```
[Route("student")]
public class StudentController : Controller
{
    [Route("details/{id}")]
    public IActionResult Details(int id)
    {
        return View();
    }
}

URL becomes: /student/details/5

Enable it in config: routes.MapMvcAttributeRoutes();


Advantages:
Clean and meaningful URLs
Better control
Easy to read and maintain
```

*5️⃣ Request Life Cycle in MVC*
Flow of a request:
```
Browser
   ↓
Routing Engine
   ↓
Controller
   ↓
Action Method
   ↓
Model (Business/Data Logic)
   ↓
View
   ↓
HTML Response
   ↓
Browser
```
Step-by-step:
1. User enters URL in browser
2. Request reaches MVC framework
3. Routing Engine checks Routing Table
4. Finds Controller & Action
5. Action executes logic
6. Data is passed to View
7. View generates HTML
8. Response is sent back to browser

- These topics help you clearly understand how MVC handles URLs and processes every request

<hr>

## Layouts, Bundle, Minification

### 1️⃣ Creating Layout and Using with Views
- A Layout is a master page (common structure) for all views.

<img width="800" height="1200" alt="image" src="https://github.com/user-attachments/assets/8eac0cde-b80b-4152-9826-facbe7e95c77" />

- In this image header and footer are common in every page.

```
Create:
_Layout.cshtml

<!DOCTYPE html>
<html>
<head>
    <title>@ViewBag.Title</title>
</head>
<body>
    <header>My Site</header>
    <div>
        @RenderBody()
    </div>
    <footer>© 2026</footer>
</body>
</html>


Use it in a view:

@{
    Layout = "~/Views/Shared/_Layout.cshtml";
}
<h2>Home Page</h2>
```
- All views now share the same header/footer.

*2️⃣ Bundling & Minification*
- Bundling → Combine multiple CSS/JS files into one
- Minification → Remove spaces, comments, reduce size
- Benefits:
Faster page load <br>
Fewer HTTP requests <br>
Better performance <br>

*3️⃣ Using BundleConfig File*
```
using System.Web.Optimization;
public class BundleConfig
{
    public static void RegisterBundles(BundleCollection bundles)
    {
        bundles.Add(new ScriptBundle("~/bundles/js")
            .Include("~/Scripts/jquery.js",
                     "~/Scripts/app.js"));

        bundles.Add(new StyleBundle("~/bundles/css")
            .Include("~/Content/site.css",
                     "~/Content/bootstrap.css"));
    }
}

In _Layout.cshtml:

@Styles.Render("~/Content/css")
@Scripts.Render("~/bundles/js")
```
- Bundling in ASP.NET MVC is implemented using classes from the  " System.Web.Optimization " namespace"
- Main classes and objects used are:
| Purpose           | Class Used           |
| ----------------- | -------------------- |
| Hold all bundles  | `BundleCollection`   |
| Create JS bundle  | `ScriptBundle`       |
| Create CSS bundle | `StyleBundle`        |
| Register bundles  | `BundleConfig` class |


*4️⃣ Attaching CSS, JS, Bootstrap in Bundles*
```
bundles.Add(new StyleBundle("~/Content/css")
    .Include("~/Content/bootstrap.css",
             "~/Content/site.css"));

bundles.Add(new ScriptBundle("~/bundles/scripts")
    .Include("~/Scripts/jquery.js",
             "~/Scripts/bootstrap.js"));
```
- This loads all CSS/JS in optimized form.

*5️⃣ Custom Helper Function*
- A Custom Helper Function in ASP.NET MVC is a user-defined helper used in Views to generate reusable HTML code.
- It is mainly created to: Reduce repeated HTML, Make Views clean and readable, Reuse UI logic (buttons, labels, messages, etc.)
```
MVC already has helpers like:
@Html.TextBox("Name")
@Html.Label("Name")
```
- You can create your own helper in the same way.
```
Create a Custom HTML Helper
Create a static class in your project (e.g., Helpers/MyHelper.cs):

using Microsoft.AspNetCore.Html;
using Microsoft.AspNetCore.Mvc.Rendering;

public static class MyHelper
{
    public static IHtmlContent MyButton(this IHtmlHelper html, string text)
    {
        return new HtmlString($"<button class='btn btn-primary'>{text}</button>");
    }
}
```

- This method: Is static, Uses this IHtmlHelper (extension method), Returns HTML

Use in View
```
In your .cshtml file:

@using YourProjectName.Helpers
@Html.MyButton("Save")

Output:
<button class='btn btn-primary'>Save</button>
```

So, Custom Helper Function =
A user-defined method that generates HTML and can be reused in Views like built-in Html helpers.

*6️⃣ Asynchronous Actions*
- Used for non-blocking operations (DB/API calls).
```
public async Task<IActionResult> Index()
{
    await Task.Delay(2000);
    return View();
}

Benefits: Better scalability, Faster response under load
```

*7️⃣ Error Handling with Log Entry*
```
public IActionResult Save()
{
    try
    {
        // risky code
    }
    catch (Exception ex)
    {
        System.IO.File.AppendAllText("log.txt", ex.Message);
        return View("Error");
    }
}
```
- Or use CustomErrors / ExceptionFilter globally.

*8️⃣ Filters & Custom Action Filter*
- Filters run before/after actions.
- Types: <br>
Authorization  <br>
Action  <br>
Result  <br>
Exception  <br>

- Custom Action Filter:
```
public class LogFilter : ActionFilterAttribute
{
    public override void OnActionExecuting(ActionExecutingContext context)
    {
        System.IO.File.AppendAllText("log.txt", "Action Called\n");
    }
}
```

Apply:
```
[LogFilter]
public IActionResult Index()
{
    return View();
}
```
| Filter Type   | When it Executes      | Purpose             |
| ------------- | --------------------- | ------------------- |
| Authorization | Before action         | Security check      |
| Action        | Before & after action | Pre/Post processing |
| Result        | Before & after result | Modify response     |
| Exception     | When error occurs     | Error handling      |


These features make your MVC app: <br>
Consistent (Layout) <br>
Fast (Bundle/Minify) <br>
Reusable (Helpers) <br>
Scalable (Async) <br>
Safe (Error Handling) <br>
Powerful (Filters) <br>
