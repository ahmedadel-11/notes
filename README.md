Here is a more detailed example of how you can implement the endpoint:

**IRegistrationRepository interface**
```
using HTI.Core.Entities;
using System.Collections.Generic;
using System.Threading.Tasks;

namespace HTI.Core.Repositories
{
    public interface IRegistrationRepository
    {
        Task<IEnumerable<Registration>> GetRegistrationsByStudentId(int studentId);
        Task<IEnumerable<Group>> GetGroupsByIds(IEnumerable<int> groupIds);
    }
}
```
**RegistrationRepository implementation**
```
using HTI.Core.Entities;
using System.Collections.Generic;
using System.Linq;
using System.Threading.Tasks;
using Microsoft.EntityFrameworkCore;

namespace HTI.Infrastructure.Repositories
{
    public class RegistrationRepository : IRegistrationRepository
    {
        private readonly DbContext _context;

        public RegistrationRepository(DbContext context)
        {
            _context = context;
        }

        public async Task<IEnumerable<Registration>> GetRegistrationsByStudentId(int studentId)
        {
            return await _context.Registrations
                .Where(r => r.StudentId == studentId)
                .ToListAsync();
        }

        public async Task<IEnumerable<Group>> GetGroupsByIds(IEnumerable<int> groupIds)
        {
            return await _context.Groups
                .Where(g => groupIds.Contains(g.GroupId))
                .ToListAsync();
        }
    }
}
```
**RegistrationsController**
```
using Microsoft.AspNetCore.Mvc;
using HTI.Core.Entities;
using HTI.Core.Repositories;
using System.Threading.Tasks;

namespace HTI.Api.Controllers
{
    [ApiController]
    [Route("api/[controller]")]
    public class RegistrationsController : ControllerBase
    {
        private readonly IRegistrationRepository _registrationRepository;

        public RegistrationsController(IRegistrationRepository registrationRepository)
        {
            _registrationRepository = registrationRepository;
        }

        [HttpGet("student/{studentId}")]
        public async Task<IActionResult> GetGroupsByStudentId(int studentId)
        {
            var registrations = await _registrationRepository.GetRegistrationsByStudentId(studentId);
            var groupIds = registrations.Select(r => r.GroupId).Distinct();
            var groups = await _registrationRepository.GetGroupsByIds(groupIds);

            return Ok(groups);
        }
    }
}
```
**Startup.cs (add repository and controller to DI container)**
```
using Microsoft.AspNetCore.Builder;
using Microsoft.AspNetCore.Hosting;
using Microsoft.Extensions.DependencyInjection;
using HTI.Infrastructure.Repositories;
using HTI.Api.Controllers;

namespace HTI.Api
{
    public class Startup
    {
        public void ConfigureServices(IServiceCollection services)
        {
            services.AddDbContext<MyDbContext>(); // assume you have a DbContext class
            services.AddTransient<IRegistrationRepository, RegistrationRepository>();
            services.AddControllers();
        }

        public void Configure(IApplicationBuilder app, IWebHostEnvironment env)
        {
            app.UseRouting();
            app.UseEndpoints(endpoints =>
            {
                endpoints.MapControllers();
            });
        }
    }
}
```
In this example, we have:

* An `IRegistrationRepository` interface that defines two methods: `GetRegistrationsByStudentId` and `GetGroupsByIds`.
* A `RegistrationRepository` class that implements the interface using Entity Framework Core to retrieve data from the database.
* A `RegistrationsController` class that uses the repository to retrieve the registrations for a given student ID, extracts the unique group IDs, and then retrieves the corresponding group objects.
* In the `Startup.cs` file, we add the repository and controller to the DI container.

When you call the `api/registrations/student/{studentId}` endpoint, the `GetGroupsByStudentId` method will be executed, which will retrieve the registrations for the given student ID, extract the group IDs, and then retrieve the corresponding group objects. The resulting list of group objects will be returned in the response.

Note that this is just a simple example, and you may want to add error handling, caching, and other features depending on your specific requirements.
