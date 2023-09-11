# Api integration testing in Csharp



## first some basic examples
```cs
var builder = WebApplication.CreateBuilder(args);
var app = builder.Build();
app.MapGet("/", () => "Hello World!");
app.MapGet("/sum", (int? n1, int? n2) => n1 + n2);
app.Run();
```

```cs
public class SimpleTesting{
    [TestMethod]
    public async Task DefaultRoute_ReturnsHelloWorld(){
        var webAppFactory = new WebApplicationFactory<Program>();
        _httpClient = webAppFactory.CreateDefaultClient();
        var response = await _httpClient.GetAsync("");
        string result = await response.Content.ReadAsStringAsync();
        Assert.AreEqual("Hello World!", result);
    }

    [TestMethod]
    public async Task Sum_Returns24(){
        var webAppFactory = new WebApplicationFactory<Program>();
        _httpClient = webAppFactory.CreateDefaultClient();
        var response = await _httpClient.GetAsync("/sum?n1=10&n2=14");
        int result = await response.Content.ReadAsStringAsync();
        Assert.AreEqual(16, result);
    }
}
```

## An more involved testing scenario

For a theoretical Api that does some Auth, uses JWT etc...

Packages used:

- `Microsoft.AspNetCore.Mvc.Testing`
- `Microsoft.AspNetCore.App`


IntegrationTest.cs
```cs
public class IntegrationTest
{
    protected readonly HttpClient TestClient;
    
    protected IntegrationTest(){
        var appFactory = new WebApplicationFactory<Startup>()
            .WithWebHostBuilder(builder =>
                {
                    builder.ConfigureServices(services =>
                    {
                        services.RemoveAll(typeof(DataContext));
                        services.AddDbContext<DataContext>(options => { options.UseInMemoryDatabase("TestDb"); });
                    });
                });
        
        TestClient = appFactory.CreateClient();
    }

    protected async Task AutenticateAsync(){
        TestClient.DefaultRequestHeaders.Authorization = new AuthenticationHeaderValue("bearer", await GetJwtAsync());
    }

    protected async Task<PostResponse> CreatePostAsync(CreatePostRequest request){
        var response = await TestClient.PostAsJsonAsync(ApiRoutes.Posts.Create, request);
        return await response.Content.ReadAsAsync<PostResponse>();
    }

    private async Task<string> GetJwtAsync(){
        var response = await TestClient.PostAsJsonAsync(ApiRoutes.Identity.Register, new UserRegistrationRequest{
            Email = "test@integration.com",
            Password = "SomePass1234!"
        });

        var registrationResponse = await response.Content.ReadAsAsync<AuthSuccessResponse>();
        return registrationResponse.Token;
    }
}
```
PostsControllerTests.cs
```cs
public class PostsControllerTests : IntegrationTest{
    [Fact]
    public async Task GetAll_WithoutAnyPosts_ReturnsEmptyResponse(){
        // Arrange
        await AutenticateAsync();

        // Act
        var response = await TestClient.GetAsync(ApiRoutes.Posts.GetAll);

        // Assert
        response.StatusCode.Should().Be(HttpStatusCode.OK);
        (await response.Content.ReadAsAsync<List<Post>>()).Should().BeEmpty();
    }

    [Fact]
    public async Task Get_ReturnsPost_WhenPostExistsInTheDatabase(){
        // Arrange
        await AutenticateAsync();
        var createdPost = await CreatePostAsync(new CreatePostRequest {Name = "Test post"});

        // Act
        var response = await TestClient.GetAsync(ApiRoutes.Posts.Get.Replace("{postId}", createdPost.Id.ToString()));
        
        // Assert
        response.StatusCode.Should().Be(HttpStatusCode.OK);
        var returnedPost = await response.Content.ReadAsAsync<Post>();
        returnedPost.Id.Should().Be(createdPost.Id);
        returnedPost.Name.Should().Be("Test post");
    }
}
```