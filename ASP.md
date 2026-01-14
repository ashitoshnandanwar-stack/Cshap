# *ASP.net*

## 1️⃣ Understanding Controllers and Actions
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

3️⃣ How Actions Are Invoked
- Actions are invoked using URL routing: /ControllerName/ActionName


4️⃣ HttpGet, HttpPost, NoAction Attributes
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

5️⃣ Running Action Result
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
