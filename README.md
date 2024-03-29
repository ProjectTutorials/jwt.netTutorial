# JWT Basics

Getting started with Json-Web-Token

1. **Create two folders**
	
	1. **[backend](#backend)**

        1. [Controllers](#controllers)

            1. [ProductsController](productscontrollercs)

                1. *[INDEX](#index)*

                2. *[DETAILS](#details)*

                3. *[CREATE](#create)*

                4. *[EDIT](#edit)*

                5. *[DELETE](#delete)*

        2. [Migrations](#migrations)

        3. [Models](#models)

            1. *[Product.cs](#productcs)*

            2. *[ProductManagement.cs](#productmanagementcs)*

	2. **[frontend](#frontend)**

        1. [Controllers](#controllers-1)

            1. [ProductsController](#productscontrollercs-1)

                1. *[INDEX](#index-1)*

                2. *[CREATE](#create-1)*

                3. *[EDIT](#edit-1)*

                4. *[DELETE](#delete-1)*

        2. [Model](#models-1)

            1. *[Product.cs](#productcs-1)*

        3. [Views](#views)

            1. *[Create.cshtml](#createcshtml)*

            2. *[Delete.cshtml](#deletecshtml)*

            3. *[Edit.cshtml](#editcshtml)*

            4. *[Index.cshtml](#indexcshtml)*

2. To Start

	1. **[backend](#backend-1)**

	2. **[frontend](#frontend-1)**

3. [Screenshots](#screenshots)


## backend

This will contain webapi from the project

### Controllers

### `ProductsController.cs`

#### INDEX

```csharp
[Authorize(Roles = "Admin,User")]
[HttpGet]
public IEnumerable<Product> Get()
{
    return productManagementDbContext.Products.ToList();
}
```

`Authorize` validates token to check for user or admin 

#### DETAILS

```csharp
[HttpGet("{id}")]
public Product Get(int id)
{
    return this.productManagementDbContext.Products.Where(product => product.Id == id).FirstOrDefault();
}
```

#### CREATE

```csharp
[HttpPost]
public string Post([FromBody] Product product)
{
    this.productManagementDbContext.Products.Add(product);
    this.productManagementDbContext.SaveChanges();
    return "Product created successfully!";
}
```

#### EDIT

```csharp
[HttpPut("{id}")]
public void Put(int id, [FromBody] Product product)
{
    this.productManagementDbContext.Products.Update(product);
    this.productManagementDbContext.SaveChanges();
}
```

#### DELETE

```csharp
[HttpDelete("{id}")]
public void Delete(int id)
{
    this.productManagementDbContext.Products.Remove(this.productManagementDbContext.Products.Where(product => product.Id == id).FirstOrDefault());
    this.productManagementDbContext.SaveChanges();
}
```

### Migrations

This will contail `entity framework` files for database integration.


### Models

#### `Product.cs`

```csharp
public class Product
{
    public int Id { get; set; }
    public string Name { get; set; }
    public int Price { get; set; }
}
```

#### `ProductManagement.cs`

```csharp
public ProductManagementDbContext(DbContextOptions<ProductManagementDbContext> dbContextOptions) : base(dbContextOptions)
{
}
public DbSet<Product> Products { get; set; }
public DbSet<User> Users { get; set; }
protected override void OnModelCreating(ModelBuilder modelBuilder)
{
    modelBuilder.Entity<User>().HasData(
        new User { Id = 1, Email = "admin@gmail.com", Password = "Passcode1", Role = Roles.Admin },
        new User { Id = 2, Email = "user@gmail.com", Password = "Passcode2", Role = Roles.User }
        );
}
```



## frontend

This will contain mvc from the project

### Controllers

### `ProductsController.cs`

#### INDEX

```csharp
private static HttpClient httpClient = new HttpClient();
public ProductsController(IConfiguration configuration)
{
    Configuration = configuration;
}

public IConfiguration Configuration { get; }
// GET: /<controller>/
[HttpGet]
public async Task<IActionResult> Index()
{
    httpClient.DefaultRequestHeaders.Accept.Add(new MediaTypeWithQualityHeaderValue("application/json"));
    httpClient.DefaultRequestHeaders.Authorization = new AuthenticationHeaderValue("Bearer", HttpContext.Session.GetString("token"));
    var response = await httpClient.GetAsync(Configuration.GetValue<string>("WebAPIBaseUrl") + "/products");
    var content = await response.Content.ReadAsStringAsync();
    if (response.IsSuccessStatusCode)
    {
        var products = new List<Product>();
        if (response.Content.Headers.ContentType.MediaType == "application/json")
        {
            products = JsonConvert.DeserializeObject<List<Product>>(content);
        }
        return View(products);
    }
    else
    {
        return RedirectToAction("Login", "Account");
    }
}
```


#### CREATE

```csharp
public async Task<IActionResult> Create()
{
    return View();
}

[HttpPost]
public async Task<IActionResult> Create(Product product)
{
    if (ModelState.IsValid)
    {
        var serializedProductToCreate = JsonConvert.SerializeObject(product);
        var request = new HttpRequestMessage(HttpMethod.Post, Configuration.GetValue<string>("WebAPIBaseUrl") + "/products");
        request.Headers.Accept.Add(new MediaTypeWithQualityHeaderValue("application/json"));
        request.Content = new StringContent(serializedProductToCreate);
        request.Content.Headers.ContentType = new MediaTypeHeaderValue("application/json");
        var response = await httpClient.SendAsync(request);

        if (response.IsSuccessStatusCode)
        {
            return RedirectToAction("Index", "Products");
        }
        else
        {
            ViewBag.Message = "Admin access required";
            return View("Create");
        }
    }
    else
        return View("Create");
}
```


#### EDIT

```csharp
[HttpGet]
public async Task<IActionResult> Edit(int id)
{
    var response = await httpClient.GetAsync(Configuration.GetValue<string>("WebAPIBaseUrl") + $"/products/{id}");
    response.EnsureSuccessStatusCode();
    var content = await response.Content.ReadAsStringAsync();
    var product = new Product();
    if (response.Content.Headers.ContentType.MediaType == "application/json")
    {
        product = JsonConvert.DeserializeObject<Product>(content);
    }
    return View(product);
}

[HttpPost]
public async Task<IActionResult> Edit(Product product)
{
    if (ModelState.IsValid)
    {
        //httpClient.DefaultRequestHeaders.Accept.Add(new MediaTypeWithQualityHeaderValue("application/json"));
        httpClient.DefaultRequestHeaders.Authorization = new AuthenticationHeaderValue("Bearer", HttpContext.Session.GetString("token"));
        var serializedProductToEdit = JsonConvert.SerializeObject(product);
        var request = new HttpRequestMessage(HttpMethod.Put, Configuration.GetValue<string>("WebAPIBaseUrl") + $"/products/{product.Id}");
        request.Headers.Accept.Add(new MediaTypeWithQualityHeaderValue("application/json"));
        request.Content = new StringContent(serializedProductToEdit);
        request.Content.Headers.ContentType = new MediaTypeHeaderValue("application/json");
        var response = await httpClient.SendAsync(request);
        if (response.IsSuccessStatusCode)
        {
            return RedirectToAction("Index", "Products");
        }
        else
        {
            ViewBag.Message = "Admin access required";
            return View("Edit");
        }
    }
    else
        return View("Edit");
}
```


#### DELETE

```csharp
[HttpGet]
public async Task<IActionResult> Delete(int id)
{
    var response = await httpClient.GetAsync(Configuration.GetValue<string>("WebAPIBaseUrl") + $"/products/{id}");
    response.EnsureSuccessStatusCode();
    var content = await response.Content.ReadAsStringAsync();
    var product = new Product();
    if (response.Content.Headers.ContentType.MediaType == "application/json")
    {
        product = JsonConvert.DeserializeObject<Product>(content);
    }
    return View(product);
}

[HttpPost]
public async Task<IActionResult> Delete(Product product)
{
    if (ModelState.IsValid)
    {
        //httpClient.DefaultRequestHeaders.Accept.Add(new MediaTypeWithQualityHeaderValue("application/json"));
        httpClient.DefaultRequestHeaders.Authorization = new AuthenticationHeaderValue("Bearer", HttpContext.Session.GetString("token"));
        var serializedProductToDelete = JsonConvert.SerializeObject(product);
        var request = new HttpRequestMessage(HttpMethod.Delete, Configuration.GetValue<string>("WebAPIBaseUrl") + $"/products/{product.Id}");
        request.Headers.Accept.Add(new MediaTypeWithQualityHeaderValue("application/json"));
        request.Content = new StringContent(serializedProductToDelete);
        request.Content.Headers.ContentType = new MediaTypeHeaderValue("application/json");
        var response = await httpClient.SendAsync(request);

        if (response.IsSuccessStatusCode)
        {
            return RedirectToAction("Index", "Products");
        }
        else
        {
            ViewBag.Message = "Admin access required";
            return View("Delete");
        }
    }
    else
        ViewBag.Message = "wrong";
    return View("Delete");
}
```



### Models

#### `Product.cs`

```csharp
    public class Product
    {
        public Product()
        {
        }
        public int Id { get; set; }
        public string Name { get; set; }
        public int Price { get; set; }
    }
```

### Views

#### `Create.cshtml`

```html
@{
    ViewBag.Title = "Create";
}
@model Product
<form asp-action="Create" asp-controller="Products" method="post" class="contactForm">
    <fieldset>
        <p>
            <label asp-for="Name"></label>
            <input asp-for="Name" />
        </p>
        <p>
            <label asp-for="Price"></label>
            <input asp-for="Price" />
        </p>
        <p>
            <button type="submit" class="btn btn-primary">Save</button>
        </p>
    </fieldset>
        <h6 style="color:red">@Html.Raw(@ViewBag.Message)</h6>
 
</form>
```


#### `Delete.cshtml`

```html
@{
    ViewBag.Title = "Delete - " + @Model.Name;
}
@model Product

<form asp-action="Delete" asp-controller="Products" method="post" class="contactForm">
    <fieldset>
        <p>
            <input asp-for="Id" hidden readonly />
        </p>
        <p>
            <label asp-for="Name"></label>
            <input asp-for="Name" />
        </p>
        <p>
            <label asp-for="Price"></label>
            <input asp-for="Price" />
        </p>
        <p>
            <button type="submit" class="btn btn-danger">Delete</button>
        </p>
    </fieldset>
    <h6 style="color:red">@Html.Raw(@ViewBag.Message)</h6>
</form>
```


#### `Edit.cshtml`

```html
@{
    ViewBag.Title = "Edit - " + @Model.Name;
}
@model Product

<form asp-action="Edit" asp-controller="Products" method="post" class="contactForm">
    <fieldset>
        <p>
            <input asp-for="Id" hidden readonly />
        </p>
        <p>
            <label asp-for="Name"></label>
            <input asp-for="Name" />
        </p>
        <p>
            <label asp-for="Price"></label>
            <input asp-for="Price" />
        </p>
        <p>
            <button type="submit" class="btn btn-secondary">Update</button>
        </p>
    </fieldset>
    <h6 style="color:red">@Html.Raw(@ViewBag.Message)</h6>
</form>
```


#### `Index.cshtml`

```html
@{

    ViewBag.Title = "Product";
}
@model List<Product>


<div style="justify-content: space-between; display: flex;">
    <a class="btn btn-success" asp-controller="Products" asp-action="Create">CREATE <i class="fa-solid fa-plus"></i></a>
    <a class="btn btn-danger" asp-controller="" asp-action="">LOG OUT</a>
</div>

<table class="table table-striped pushDown">
    <thead>
        <tr>
            <th>ID</th>
            <th>Name</th>
            <th>Price</th>
            <th>Action</th>
        </tr>
    </thead>
    <tbody>
        @foreach (var item in Model)
        {
            <tr>
                <td>@item.Id</td>
                <td>@item.Name</td>
                <td>@item.Price</td>
                <td>
                    <a class="btn btn-primary" asp-controller="Products" asp-action="Edit" asp-route-id="@item.Id">EDIT</a>
                    <a class="btn btn-danger" asp-controller="Products" asp-action="Delete"
                    asp-route-id="@item.Id">DELETE</a>
                </td>
            </tr>
        }
    </tbody>
</table>
```


## To Start

## backend

#### Prerequisites

1. [.NET 6.0](https://dotnet.microsoft.com/en-us/download/dotnet/6.0#:~:text=May%2010%2C%202022-,Build%20apps%20%2D%20SDK,-Tooltip%3A%20Do%20you)

2. [SQL-Server](https://docs.microsoft.com/en-us/sql/ssms/download-sql-server-management-studio-ssms?view=sql-server-ver16#:~:text=Free%20Download%20for%20SQL%20Server%20Management%20Studio%20(SSMS)%2018.12.1)

3. [SSMS](https://www.microsoft.com/en-in/sql-server/sql-server-downloads#:~:text=Download%20now-,Express,-SQL%20Server%202019)

#### To Start 

1. For Database connection 

    Build the solution

    ```ps
    dotnet ef database update
    ```

    Expected output

    ```output
    Build started...
    Build succeeded.
    Done.
    ```

2. Use `postman` or `thunderclient` further
    
3. **POST** method
    
    https://localhost:5001/api/authenticate

    There in *basic auth* use these credentials

    ```http
    UserName: admin@gmail.com
    Password: Passcode1
    ```

    ##### `Token` would be generated

    <details>
    <summary>Screenshots</summary>
    <img  src="https://user-images.githubusercontent.com/76637730/185432139-1499ed0d-742e-49b5-871c-08b974b9127e.png"> <br>       
    Response <br> 
    <img  src="https://user-images.githubusercontent.com/76637730/185439279-51db7471-c966-4dcb-bfb0-5f64e4cb1eac.png"> 
    </details>

4. **GET** method

    https://localhost:5001/api/products

    There in *Bearer Token* use the [token](#token-would-be-generated) generated

    <details>
    <summary>Screenshots</summary>
    <img  src="https://user-images.githubusercontent.com/76637730/185442808-d2874be3-dd02-490a-9c60-b1ded862606e.png">  <br> 
    Response <br> 
    <img  src="https://user-images.githubusercontent.com/76637730/185442861-aea3e58f-5321-44c9-8b82-da3d82e95927.png"> 
    </details>

## frontend

#### Run it using either of the command

```ps
dotnet run
```

```ps
dotnet watch run
```


## Screenshots

1. Login
![image](https://user-images.githubusercontent.com/76637730/185850124-b8f7f06f-4bbc-4ced-847a-fe0184899393.png)

2. Index
![image](https://user-images.githubusercontent.com/76637730/185850168-47e32268-2566-482c-ba02-d4ec566c327b.png)

3. Create
![image](https://user-images.githubusercontent.com/76637730/185850311-5c8adf04-dc3a-4274-9eaf-d5352e5ec16e.png)

4. Edit
![image](https://user-images.githubusercontent.com/76637730/185850218-e0bed450-b83b-4c35-9666-1398e984dc1b.png)
