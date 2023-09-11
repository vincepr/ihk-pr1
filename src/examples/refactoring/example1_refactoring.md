# Part 1 - Refactoring exercise in Csharp 
goal is to refactor to make code testable.

- Immagine for a given codebase we want to make some logic-changes to `UserServices.cs` but cant most other parts of the codebase.

- No business logic should be touched. (everything should behave as before logically)


## Old code 
```cs
public class UserService {
    public bool AddUser(string firstname, string surname, string email, DateTime dateOfBirth, int clientId) {
        if (string.IsNullOrEmpty(firstname) || string.IsNullOrEmpty(surname))
            return false;
        
        if (!email.Contains("@") && !email.Contains("."))
            return false;
        
        var now = DateTime.Now;
        int age = now.Year - dateOfBirth.Year;
        if (now.Month < dateOfBirth.Month 
            || (now.Month == dateOfBirth.Month && now.Day < dateOfBirth.Day))
        {
            age--;
        }

        if (age <21)
            return false;

        var clientRepository = new ClientRepository();
        var client = clientRepository.GetById(clientId);

        var user = new User{
            Client = client,
            DateOfBirth = dateOfBirth,
            EmailAdress = email,
            Firstname = firstname,
            Surname = surname
        };

        if (client.Name == "VeryImportantClient"){
            // skipp credit check
            user.HasCreditLimit = false;
        }
        else if (client.Name == "ImportantClient"){
            // credit check and double credit limit
            user.HasCreditLimit = true;
            using (var userCreditService = new IserCreditServiceClient()){
                var creditLimit =userCreditService.GetCreditLimit(user.Firstname, user.Surname, user.DateOfBirth);
                creditLimit = creditLimit*2;
                user.CreditLimit = creditLimit;
            } 
        }else {
            // normal credit check
            user.HashcCreditLimit = true;
            using (var userCreditService = new UserCreditServiceClient()){
                var creditLimit =userCreditService.GetCreditLimit(user.Firstname, user.Surname, user.DateOfBirth);
                user.CreditLimit = creditLimit;
            }
        }

        if (user.HasCreditLimit && user.CreditLimit < 500)
            return false
        
        UserDataAccess.AddUser(user);
        return true;
    }
}
```

## Steps to take
1. There are no unit tests currently. So first goal would be to write unit tests, that pass the existing logic. So breaking for existing behavior becomes aparent.

2. `var now = DateTime.Now` is not really testable
3. `var clientRepository = new ClientRepository()` gets created new every request
4. `using (var userCreditService = new UserCreditServiceClient())` not testable like this
5. `UserDataAccess.AddUser(user)` is not testable like this

### DateTime.Now
Avoid `DateTime.Now` as it is really hard to write tests like this. Since tests might excede limits, timezones exist, Testing Containers might report wrong timestamps etc.
- create a new `/Services/IDateTimeProvider.cs`
```cs
public interface IDateTimeProvider{
    public DateTime DateTimeNow { get; }
}
```
- and an implementaitonw `/Services/DateTimeProvider.cs`
```cs
public interface DateTimeProvider : IDateTimeProvider{
    public DateTime DateTimeNow => DateTime.Now
}
```
Now the DateTimeProvider is mockable in our unit tests.

### Interface for the ClientRepository
We create an Interface for the ClientRepository, to make it properly mockable aswell:
- `/Repositories/IClientRepository.cs`
```cs
public interface IClientRepository{
    Client GetById(int id);
}
```
By injecting the clientRepository we can also avoid always fully recreating the Repository.
```cs
var clientRepository = new ClientRepository();
var client = clientRepository.GetById(clientId);
// becomes just:
var client = _clientRepository.GetById(clientId);
```

### UserCreditService
Similar to above there is (in this case) no need to always recreate the Service for each new user.

So we again just put it in an readonly attribute and directly call that without any of the `using (var service = new Service())`

### UserDataAccess the static method we are not allowed to touch
`UserDataAccess` is in this case a static class we can not change (because were not allowed to).

A solution for a Situation like this is: to create an Interface and a Proxy- /wrapper- Class for UserDataAcess.

The Proxy Class will call the static-method but it itself becomes testable/mockable. Without actually changing the underlying class.
- `DataAccess/IUserDataAccess`
```cs
public interface IUserDataAccess{
    void AddUser(User user);
}
```
- `DataAccess/UserDataAccessProxy`
```cs
public class UserDataAccessProxy : IUserDataAccess{
    void AddUser(User user)
        => UserDataAcess.AddUser(user); // we just call the off limit method in here
}
```

### The new constructor
as per request we are not allowed to change the Code that calls our UserService.

This means we cant just do a Constructor and inject our 4 new Interface Implementations.

The best way here is to add 2 constructors, one where one can provivde the 4 Implementors. And one without any params that just implements our above created ones.

## new code so far
```cs
public class UserService{

    private readonly IDateTimeProvider _dateTimeProvider;
    private readonly IClientRepository _clientRepository;
    private readonly IUserCreditService _userCreditService;
    private readonly IUserDataAccess _userDataAccess;

    public UserService() :
        this(
            new DateTimeProvider(),
            new ClientRepository(),
            new UserCreditService(),
            new UserDataAccessProxy()
        ){}

    public UserService(IDateTimeProvider dateTimeProvider, IClientRepository clientRepository, IUserCreditService userCreditService, IUserDataAccess userDataAccess){
        _dateTimeProvider = dateTimeProvider;
        _clientRepository = clientRepository;
        _userCreditService = userCreditService;
        _userDataAccess = userDataAccess;
    }

    public bool AddUser(string firstname, string surname, string email, DateTime dateOfBirth, int clientId){
        if (string.IsNullOrEmpty(firstname) || string.IsNullOrEmpty(surname))
            return false;
        
        if (!email.Contains("@") && !email.Contains("."))
            return false;
        
        var now = _dateTimeProvider.DateTimeNow;
        int age = now.Year - dateOfBirth.Year;
        if (now.Month < dateOfBirth.Month 
            || (now.Month == dateOfBirth.Month && now.Day < dateOfBirth.Day))
        {
            age--;
        }

        if (age <21)
            return false;

        var client = _clientRepository.GetById(clientId);

        var user = new User{
            Client = client,
            DateOfBirth = dateOfBirth,
            EmailAdress = email,
            Firstname = firstname,
            Surname = surname
        };

        if (client.Name == "VeryImportantClient"){
            // skipp credit check
            user.HasCreditLimit = false;
        }
        else if (client.Name == "ImportantClient"){
            // credit check and double credit limit
            user.HasCreditLimit = true;

            var creditLimit = _userCreditService.GetCreditLimit(user.Firstname, user.Surname, user.DateOfBirth);
            creditLimit = creditLimit*2;
            user.CreditLimit = creditLimit;
        }else {
            // normal credit check
            user.HasCreditLimit = true;

            var creditLimit = _userCreditService.GetCreditLimit(user.Firstname, user.Surname, user.DateOfBirth);
            user.CreditLimit = creditLimit;
        }

        if (user.HasCreditLimit && user.CreditLimit < 500)
            return false;
        
        _userDataAccess.AddUser(user);
        return true;
    }
}
```

Additional thoughts so far
- `if (!email.Contains("@") && !email.Contains("."))` is not really a proper way to check for if the email string is a proper email
    - but changes to it will affect business logic. So it **must not** be part of the current refactoring. But should always be it's own refactor/commit etc.
- the whole way how `"VeryImportantClient" || "ImportantClient" || Default` is handled is a violation of the **Open Closed** Principle. And should be separated.
    - this can be changed without affecting business logic so it should be refactored.