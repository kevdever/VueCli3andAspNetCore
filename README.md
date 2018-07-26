# Vue Cli 3 & Asp.Net Core 2.1.3
This project is a basic template using Asp.Net Core to serve a VUE SPA. It is designed for simplicity and for rapid development, leveraging the benefits of Vue Cli 3.

## A [tutorial](https://github.com/kevdever/VueCli3andAspNetCore/blob/master/Tutorial.md) is included in this repository. The file is named `tutorial.md`.

In the interest of keeping this simple and robust to future changes, considering that Vue Cli 3 is still in alpha, the workflows for development and production differ.

## Development Workflow
* From within the ClientApp folder, run `vue serve`
* Develop your app with the benefits of hot reloading
* Run your tests within the ClientApp folder using the plugins for Vue CLI

* When you're ready for production tests or deployment, deploy as you would any other Asp.Net Core app, after you've run `vue build` from within your ClientApp; you can run `dotnet run` from the root to run the webapp on Asp.Net Core in Kestrel.

### Note: this workflow leverages both Asp.Net Core tooling and Vue Cli 3 tooling with the bare minimum of configuration. To achieve this simple workflow, you'll notice that the dev server does not run Asp.Net Core, but Node. Thus, when using the dev workflow, your Asp.Net Core middleware won't be running. 

This separation highlights the conceptual simplicity of a SPA, especially with Vue CLI 3: Vue CLI 3 generates everything you need, js, css, and even an index.html, such that to launch the app from Asp.Net Core, all you need to do is have a controller return the auto-generated index.html.

Thus, it is important to note that the ASP.Net Core HSTS and HTTPS middlewares won't be available when running in dev. The same goes for any cookie middleware. However, you can use that middleware with `dotnet watch run` assuming you've already done a build of the SPA.

#### In this project, the Web API is assumed to be housed in its own project, thus it is a standalone server, and the client/SPA has a standalone.
