
# Asp.net Core WebApi Project
Dear reader I am developed this project by using the best practice and methodlogy. I tried to write the clean and maintainable code.
I wish this repsoitory helps to build amazing web api with design pattern architecture used here.
### Install from NuGet
Install the following packages from NuGet package manager.

    Microsoft.EntityFrameworkCore
    Microsoft.EntityFrameworkCore.SqlServer

## Model Binding
If you want to read the data from JSON/Text you need to add [from body] as show in the following example. Without it will not read the product entered as the body of the request.
Since the the method name is post it can be called from URL: /api/product with type POST (Help of POST man for test). (Since the method is post directly sending a post request to the controller will execute this method.)
```sh
[HttpPost("")]
public IActionResult Post(Product product)
{
	return Ok();
}
```
should be (in order to read from the request body.)
```sh
[HttpPost("")]
public IActionResult Post([FromBody]Product product)
{
	return Ok();
}
```

## Middlewares
- in startup we used a middleware, they are injected with app.
- Middleware is software that's assembled into an app pipeline to handle requests and responses. Each component:
        
  - Chooses whether to pass the request to the next component in the pipeline.
  - Can perform work before and after the next component in the pipeline is invoked.
Request delegates are used to build the request pipeline.
```sh
public void Configure(IApplicationBuilder app, IHostingEnvironment env)
{            
	if (env.IsDevelopment())
	{
		app.UseDeveloperExceptionPage();
	}
	// Routing mechanism as Default 
	app.UseMvc(config => 
	{ 
		//mvc middleware
		config.MapRoute("DefaultRoute", "api/{controller}/{action}"); 
		
	});         
}
```

> Control if the person who request has proper authentication or not.

For more information on the related topic visit the Microsoft Msdn Documentation
https://docs.microsoft.com/tr-tr/aspnet/core/fundamentals/middleware/?view=aspnetcore-2.1
 
### Custom Middleware
We are creating a custom middleware to check the authentication
for that we need to rules first Invoke and

we need http context to read the header data 
Request delegate will provide to move between middleware

```sh
public class AuthenticationMiddleware
{
	private readonly RequestDelegate _next;

	public AuthenticationMiddleware(RequestDelegate next)
	{
		_next = next;
	}
	public async Task Invoke(HttpContext context)
	{
		var authHeader = context.Request.Headers["Authorization"];

		// Pass to next middleware
		await _next(context);
	}
}

```
If you send a request header via postman and debug on the await _next you can see the middle ware first read the header
Donot forget to add the middleware in the startup before the test
```sh
public void Configure(IApplicationBuilder app, IHostingEnvironment env)
{
			
	if (env.IsDevelopment())
	{
		app.UseDeveloperExceptionPage();
	}

	// authentication middleware
	app.UseMiddleware<AuthenticationMiddleware>();

	// Routing mechanism as Default
	app.UseMvc(config =>
	{
		// Routing middleware
		config.MapRoute("DefaultRoute", "api/{controller}/{action}");
	});
	app.UseMvc();
}
```

Now we will build a basic authentication _basic Hidayat:1234_ practically in the system we need to get the user from the Database.

in the authentication middleware.
```sh
public class AuthenticationMiddleware
{
    private readonly RequestDelegate _next;

    public AuthenticationMiddleware(RequestDelegate next)
    {
        _next = next;
    }
    public async Task Invoke(HttpContext context)
    {
        string authHeader = context.Request.Headers["Authorization"];
        
        // Check the auth Header
        // There might be more than one authentications (this one basic)
        if (authHeader !=null && authHeader.StartsWith("basic", StringComparison.OrdinalIgnoreCase))
        {
            // Username token
            var token = authHeader.Substring(6).Trim();
            // Convert to normal string
            var credentialString = Encoding.UTF8.GetString(Convert.FromBase64String(token));
            // Splitting the password from username
            var credentials = credentialString.Split(':');

            // Connecting to DB to check if the password and username is avaliable or not
            // at top we have array of creditial with credintital[0]-username and credintial[1] password
            if (credentials[0]=="hidayat" && credentials[1]=="1234")
            {
                // Identity principal
                // claim: Type 
                var claims = new[]
                {
                    new Claim("name", credentials[0]),
                    // Role 
                    // It is good if you get it from the DB
                    //new Claim("name", credentials[0]),
                    new Claim(ClaimTypes.Role, "Admin")
                };
                // Creat Identity
                // ClaimsIdentity wants claims and AuthenticationType
                var identity = new ClaimsIdentity(claims, "basic");

                context.User = new ClaimsPrincipal(identity);
            }
        }
        // If there is no authHeader
        else
        {
            // Incase user isnot authenticated return not authorized 401 status Code
            context.Response.StatusCode = 401;
        }

        // Pass to next middleware
        await _next(context);
    }
}
```


### About me
For any question and technical help with the repository, feel free to contact me via: 

Email: hidayatarg@gmail.com

[![N|Solid| width=5 ](https://image.ibb.co/dzZczz/Webp_net_resizeimage.png)](https://www.linkedin.com/in/hidayatarg/)Linkedin Profile

