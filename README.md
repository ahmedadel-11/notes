You can create an endpoint that takes a `studentId` as a parameter and returns a list of `Group` objects that the student is registered in. Here's an example:
```
using Microsoft.AspNetCore.Mvc;
using HTI.Core.Entities;
using HTI.Core.Repositories; // assume you have a repository for registrations

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
        public IActionResult GetGroupsByStudentId(int studentId)
        {
            var registrations = _registrationRepository.GetRegistrationsByStudentId(studentId);
            var groupIds = registrations.Select(r => r.GroupId).Distinct();
            var groups = _registrationRepository.GetGroupsByIds(groupIds);

            return Ok(groups);
        }
    }
}
```
In this example, we're using a repository pattern to encapsulate the data access logic. The `IRegistrationRepository` interface would have methods like `GetRegistrationsByStudentId` and `GetGroupsByIds` that return the relevant data.

The `GetGroupsByStudentId` method takes a `studentId` parameter, retrieves the registrations for that student, extracts the unique `GroupId`s from the registrations, and then retrieves the corresponding `Group` objects using the `GetGroupsByIds` method.

Finally, the method returns a list of `Group` objects in the response.

You can then call this endpoint by sending a GET request to `api/registrations/student/{studentId}`, replacing `{studentId}` with the actual student ID.

For example, if you have a student with ID 123, you would call `api/registrations/student/123` and the endpoint would return a list of `Group` objects that the student is registered in.

Note that this assumes you have a repository implementation that can retrieve the data from the database. You'll need to implement the `IRegistrationRepository` interface and provide the necessary logic to retrieve the data.
