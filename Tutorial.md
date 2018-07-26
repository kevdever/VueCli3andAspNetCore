# Instructions to Recreate this Template (Tutorial)
Note: The commands below were run in Powershell, but they're identical (or almost so) in macOS/Linux. The tools used were Powershell and VS Code with Vetur.

## Scaffold the Asp.Net Core project and set it up to bootstrap the SPA
1. Create a project folder: `> mkdir VueAspNetCoreTemplate`
2. Go to that folder: `> cd .\VueAspnetCoreTemplate`
3. Scaffold an ASP.Net Core MVC project: `dotnet new mvc` (using a controller to serve the index.html bootstrapper keeps this simple, thus the choice of using the MVC template)
4. Delete all of the Views in home, as well as the directory: `> rm .\Views\Home\ -R`
5. In `Controllers\HomeController`, delete all of the handlers _EXCEPT_ for the error handler. We'll be rewriting the Index getter to serve up the SPA static html file.
6. Replace the old `Index()` method with the following: 
```c# 
public IActionResult Index([FromServices] IHostingEnvironment hostingEnvironment)
{
    return PhysicalFile(Path.Combine(hostingEnvironment.ContentRootPath, "index.html"), "text/html");
}
```
This method uses the injected IHostingEnvironment to get the base path (set in program.cs), then fetches the auto-generated index.html and returns that to the client.

7.  Asp.Net Core's MVC project creates a `wwwroot` folder and assumes static files will be served from there.  Vue CLI, however, assumes static files will be in `/dist`.  In order for (1) the HomeController to access the the auto-generated `index.html` file as simply as possible, the `ContentRoot` folder needs to be changed from the default to `\ClientApp\dist` in `program.cs`; (2) and for the SPA to have access to the js, css, and other resources, they need to have access to the same `\ClientApp\dist` folder, which requires updating the `UseStaticFiles` middleware in `startup.cs`.

    a. Delete wwwroot: `> rm .\wwwroot\ -R`
    b. In `program.cs`, update the CreateHostBuilder as follows to set the custom ContentRoot that Vue expects:
    ```c#
    public static IWebHostBuilder CreateWebHostBuilder(string[] args) =>
        WebHost.CreateDefaultBuilder(args)
            .UseContentRoot(Path.Combine(Directory.GetCurrentDirectory(), "ClientApp/dist"))
            .UseStartup<Startup>();
    ```
    c. Add the required import to program.cs: `using System.IO;`

    d. Update `startup.cs` to serve static files from the `ClientApp\dist` folder:
    ```c#
    public void Configure(IApplicationBuilder app, IHostingEnvironment env)
    {
        ...
        var pathToStaticFiles = Path.Combine(Directory.GetCurrentDirectory(), "ClientApp/dist");
        app.UseStaticFiles(new StaticFileOptions{
            FileProvider = new PhysicalFileProvider(pathToStaticFiles)
        });
        ...
    }
    ```
    e. Add the required imports to startup.cs: 
    ```c#
    using System.IO; 
    using Microsoft.Extensions.FileProviders;
    ```

8. The `startup.cs` file needs a couple of additional tweaks:
        
    a. Add a fallback for the SPA so that the client, rather than the server, handles 404's:
    ```c#
    public void Configure(IApplicationBuilder app, IHostingEnvironment env)
    {
        ...
        app.UseMvc(routes =>
        {
            routes.MapRoute(
                name: "default",
                template: "{controller=Home}/{action=Index}/{id?}");

            routes.MapSpaFallbackRoute(
                name: "spa-fallback",
                defaults: new { controller = "Home", action = "Index"}
            );
        });
    }
    ```

    b. If you want to disable the GDPR-motivated cookie check while in dev, disable it in the `ConfigureServices` method (_**note**: whether this is a good idea is left unanswered, but this manner of turning it off is quick and dirty. For production, you'll want to revisit this carefuly_).
    ```c#
    public void ConfigureServices(IServiceCollection services)
    {
         services.Configure<CookiePolicyOptions>(options =>
        {
            // This lambda determines whether user consent for non-essential cookies is needed for a given request.
            options.CheckConsentNeeded = context => false;
            options.MinimumSameSitePolicy = SameSiteMode.None;
        });

        services.AddMvc().SetCompatibilityVersion(CompatibilityVersion.Version_2_1);
    }
    ```

<hr/>

### That's it for the server. To confirm that it's working, let's do a simple test and see if it will serve up a static html file where we told it to look . . .
1. Create the `ClientApp` folder and its `dist` subfolder: `> mkdir ClientApp\dist`
2. Create a dummy `index.html` file in the new `dist` static files foler for the `HomeController` to serve up: `> echo '<!DOCTYPE html><body><h1>Hello World!</h1></body></html>' >
 .\ClientApp\dist\index.html`
 3. Start the server and view your new app at **localhost:5000**: `> dotnet run`
4. Stop the server when done.
5. Delete these new files and folders created for the test: `> rm .\clientapp\ -R`
<hr/>

## Scaffolding the Vue app
As noted in the README file, Vue CLI 3 rolls in its own server tooling, so we're using that, not Asp.Net Core while in dev. Let's get that setup and create the basic VUE app.

1. Create a new Vue project: `> vue create clientapp`, then choose the desired options. Here, I chose:
    
    i. Manually Select Features:  `Babel, Router, Vuex, CSS Pre-processors, Linter/Formatter`

    ii. CSS Pre-processor: `SCSS/SASS`

    iii. `ESLint with error prevention only`

    iv. `Lint on save`

    v. `In dedicated config files`

    vi. `Save this as a preset . . . ?` --> No
2. Rename the `clientapp` folder to `ClientApp` (this takes two steps in the cli):
```sh
    > mv  .\clientapp\ .\ClientAppTemp
    > mv  .\ClientAppTemp\ .\ClientApp
```
<hr/>

## Let's take a quick break and get git setup before we proceed.
Vue CLI 3 created a `.gitignore` for us in the `ClientApp` folder, but we want our repo to be in the root folder. We also want to add a bit to the .gitignore. So, let's just create a new .gitignore there and then delete this one:

1. I started with the [boilerplate .gitignore for Visual Studio](https://github.com/github/gitignore/blob/master/VisualStudio.gitignore)

2. Then I added the items that Vue add in its version (with a slight path tweak to include ClientApp):
    
```

    .DS_Store
    ClientApp/node_modules
    ClientApp/dist

    # local env files
    .env.local
    .env.*.local

    # Log files
    npm-debug.log*
    yarn-debug.log*
    yarn-error.log*

    # Editor directories and files
    .idea
    .vscode
    *.suo
    *.ntvs*
    *.njsproj
    *.sln
    *.sw*

```
3. Delete the .gitignore in the ClientApp folder: `> rm .\ClientApp\.gitignore`
4. Initialize git: `> git init`
<hr/>

## Let's get back to Vue. We've already scaffolded a basic Vue app, so let's run it.
0. As of this writing, there's a syntax issue in the files scaffolded by Vue CLI 3. In Home.Vue, there's a path prepended with "@" rather than "./", which is an alias that results in an error when you try to run. We can avoid the error with a different syntax. Let's fix that:
from
```
import HelloWorld from '@/components/HelloWorld.vue'
```
to
```
import HelloWorld from '../components/HelloWorld.vue'
```

1. Navigate to the folder with Vue's entry js file: `> cd .\ClientApp\src\`
2. Run the server: `> vue serve`

    * You might get an error asking to you install vue-cli-service globally. To resolve this: `> npm install -g @vue/cli-service-global`
3. In your browser, view your web app: `http://localhost:8080`
4. Stop the server.
<hr/>

## Production - Running in Kestrel
When you're ready to deploy the webapp, or to test it running with the Asp.Net Core backend on Kestrel, all you have to do is run a production build of the vue app:
1. Navigate up to the ClientApp folder: `> cd ..`
2. Build: `> npm run build`
3. Navigate up to the project root: `> cd ..`
4. Run Kestrel: `> dotnet run` (or `dotnet watch run` if you'd like)
5. In the browser, you'll find your app at `http://localhost:5000`

That's it. Vue generated all of the files it needs in ClientApp/dist, including an index.html file. We've already told the Asp.Net Core backend to look in that folder for the files it needs, so nothing further is needed.

