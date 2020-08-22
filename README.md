<p align="center">
 <h2 align="center">CQRS-CleanInstallers</h2>
 <p align="center">A clean way to implement <b><i>dependency injection</b></i> in startup.cs on .Net Core Web Project with CQRS & MediatR. </b></p>
</p>
<br/>

---

Web Project had configured with few of layered class libiary project, swagger installation for the API documentaion and CORS for resource sharing.


<h5> startup.cs - before clean installers </h5>

<pre>
<code>
// This method gets called by the runtime. Use this method to add services to the container.
        public void ConfigureServices(IServiceCollection services)
        {
            services.AddControllers();
            /** Application layer & Infrastructure layer DI reference */
            services.AddApplication();
            services.AddInfrastructure();

            /** Swagger reference */
            services.AddSwaggerGen(c =>
            {
                c.SwaggerDoc("v1", new OpenApiInfo { Title = "CQRS With MediatR APIs", Version = "v1" });
            });

            /** CORS reference */
            services.AddCors(options =>
            {
                options.AddPolicy(name: "MyAllowSpecificOrigins",
                                  builder =>
                                  {
                                      builder.AllowAnyOrigin()
                                             .AllowAnyHeader()
                                             .AllowAnyMethod();
                                  });
            });
        }

</pre>
</code>

<br />

<h5> startup.cs - after clean installers </h5>

<pre>
<code>
// This method gets called by the runtime. Use this method to add services to the container.
        public void ConfigureServices(IServiceCollection services)
        {
            services.InstallServicesInAssembly(Configuration);
        }
</pre>
</code>

<br />

<h5> steps </h5>

1. Create a folder(named as: *Installers*) in the API layer
2. Create an interface
```
 public interface IInstaller
 {
    void InstallServices(IServiceCollection services, IConfiguration configuration);
 }
```
   
3. Create respective DI classes and inherit ***IInstaller*** interface
4. DI
    ```
    public class DIInstaller : IInstaller
    {
        public void InstallServices(IServiceCollection services, IConfiguration configuration)
        {
            services.AddControllers();

            services.AddApplication();
            services.AddInfrastructure();
        }
    }
    ```
5. Swagger
    ```
    public class SwaggerInstaller : IInstaller
    {
        public void InstallServices(IServiceCollection services, IConfiguration configuration)
        {
            services.AddSwaggerGen(c =>
            {
                c.SwaggerDoc("v1", new OpenApiInfo { Title = "CQRS With MediatR APIs", Version = "v1" });
            });
        }
    }
    ```
6. CORS
    ```
    public class CorsInstaller : IInstaller
    {
        public void InstallServices(IServiceCollection services, IConfiguration configuration)
        {
            services.AddCors(options =>
            {
                options.AddPolicy(name: "MyAllowSpecificOrigins",
                                  builder =>
                                  {
                                      builder.AllowAnyOrigin()
                                             .AllowAnyHeader()
                                             .AllowAnyMethod();
                                  });
            });
        }
    }
    ```
7. Create a class that extends ***Microsoft.Extensions.DependencyInjection***
    ```
    public static class InstallerExtensions
    {
        public static void InstallServicesInAssembly(this IServiceCollection services, IConfiguration configuration)
        {
            typeof(Startup).Assembly.ExportedTypes
                .Where(x => typeof(IInstaller).IsAssignableFrom(x) && !x.IsInterface && !x.IsAbstract)
                .Select(Activator.CreateInstance).Cast<IInstaller>()
                .ToList()
                .ForEach(installer => installer.InstallServices(services, configuration));
        }
    }
    ```
8. In startup.cs under services section clean the existing code and include below mentioned code
    ```
    services.InstallServicesInAssembly(Configuration);
    ```

<br />

###### Thats clean DI configuration was finished. similarly we can configuration other DI or we can existing DIs.

<br/> 

<div align="center">

### Show some ❤️ by starring some of the repositories!

</div>