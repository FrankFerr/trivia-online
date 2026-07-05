> [!info] Note
> The registration and login forms will be placed on the same page

## FrontEnd - page structure
---
### Layout
- navbar
- login form
- registration form
- footer

### Functionality
- [ ] **Login form**
	- **Fields**
		- username (TBD: unique field for username/email?)
		- password
	- **Actions**
		- Log in (calls [[#^4961dc|login]])
		- switch to register form
	- **Visuals feedback**
		- show validation fields error message
		- a spinner after pressing Log in button
		- show form error message sent back from the server
	- **Validations**
		- blank fields
	- **On Complete Actions**
		- Login action returns 200 OK -> redirect user to the dashboard page

## BackEnd
---
- [ ] **POST api/auth/login** ^4961dc
	- **Request**
		- [[#LoignRequest]]
	- **Response** 
		- [[#LoignResponse]]
	- **Execution Flow**
		- AuthHandler
			- data validation (blank fields) -> 400 BadRequest with error message
			- check if username exists -> 401 Unauthorized with error message
			- check if hashed password matches saved password -> 401 Unauthorized with error message
			- generate access_token and refresh_token
			- save refresh_token in ***refresh_tokens***
			- return [[#LoignResponse]]
		- return based on Client-Type:
			- Mobile (TBD): return 200 OK [[#LoignResponse]]
			- Web: set cookie HttpOnly for access_token and refresh_token and return 200 OK

## DTO
___
### LoginRequest
```c#
public class LoginRequest
{
	public string Username;
	public string Password;
}
```

### LoginResponse
```c#
public class LoginResponse
{
	public string AccessToken;
	public string RefreshToken;
}
```
