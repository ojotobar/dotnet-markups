using Entities.Enums;
using Entities.Responses;
using Microsoft.AspNetCore.Mvc;
using Moq;
using Services.Contracts;
using Shared.DTO;
using System.Threading.Tasks;
using TeamTaskManager.Presentation.Controllers.v1;
using Xunit;

namespace TeakTaskManagerApi.Tests
{
    public class AuthServiceTests
    {
        private readonly Mock<IServiceManager> _mockService;
      
        public AuthServiceTests()
        {
            _mockService = new Mock<IServiceManager>();
        }

        [Theory]
        [InlineData("valid@email.com", "Valid Name", "ValidPass!", "ValidPass!", 1, true)]
        [InlineData("invalid.email.com", "Valid Name", "ValidPass!", "ValidPass!", 1, false)] //Invalid email
        [InlineData("", "Valid Name", "ValidPass!", "ValidPass!", 2, false)] //No email
        [InlineData("valid@email.com", "", "ValidPass!", "ValidPass!", 1, false)] //No name
        [InlineData("valid@email.com", "Valid Name", "", "ValidPass!", 1, false)] // Invalid password
        [InlineData("valid@email.com", "Valid Name", "InvalidPass", "InvalidPass", 1, false)] // No password
        [InlineData("valid@email.com", "Valid Name", "ValidPass!", "NonMarchingPass", 1, false)] //Non matching passwords
        [InlineData("valid@email.com", "Valid Name", "ValidPass!", "ValidPass!", 3, false)] //Invalid role type
        public async Task Register_Returns_Expected_Results(string email, string name, string password, string confirmPassword, int role, bool expectedSuccess)
        {
            //Arrange
            var registerRequest = new RegistrationDto
            {
                Email = email,
                Name = name,
                Password = password,
                ConfirmPassword = confirmPassword,
                Role = (Roles)role
            };

            _mockService.Setup(x => x.Authentication.Register(registerRequest))
                .ReturnsAsync(expectedSuccess ? new OkResponse<UserToReturnDto>(new UserToReturnDto
                {
                    Name = name,
                    Email = email
                }) : new BadRequestResponse("Invalid inputs"));

            var authController = new AuthenticationController(_mockService.Object);

            //Act
            var result = await authController.Register(registerRequest);

            //Assert
            if (expectedSuccess)
            {
                Assert.IsType<OkObjectResult>(result);
            }
            else
            {
                Assert.IsType<BadRequestObjectResult>(result);
            }
        }

        [Theory]
        [InlineData("valid@email.com", "V@lidP@33", true)]
        [InlineData("invalid@email.com", "V@lidP@33", false)]
        [InlineData("valid@email.com", "", false)]
        [InlineData("valid@email.com", "WrongP@33", false)]
        [InlineData("wrong@email.com", "V@lidP@33", false)]
        public async Task Login_Returns_Expected_Results(string email, string password, bool expectedSuccess)
        {
            //Arrange
            var loginRequest = new LoginDto
            {
                Email = email,
                Password = password
            };

            var expectedUnauthorized = password.Equals("WrongP@33") || email.Equals("wrong@email.com");

            var okResponse = new OkResponse<string>("fake.jwt.token");
            var badRequestResponse = new BadRequestResponse("Invalid inputs");
            var unAuthorizedResponse = new UnAuthorizedResponse("Wrong email or password");

            ApiResponseBase response = expectedSuccess ? okResponse :
                 expectedUnauthorized ? unAuthorizedResponse : badRequestResponse;

            _mockService.Setup(x => x.Authentication.Login(loginRequest))
                .ReturnsAsync(response);

            var authController = new AuthenticationController(_mockService.Object);

            //Act
            var result = await authController.Login(loginRequest);

            //Assert
            if (expectedSuccess)
            {
                Assert.IsType<OkObjectResult>(result);
            }
            else if(!expectedSuccess && expectedUnauthorized)
            {
                Assert.IsType<UnauthorizedObjectResult>(result);
            }
            else
            {
                Assert.IsType<BadRequestObjectResult>(result);
            }
        }
    }
}
