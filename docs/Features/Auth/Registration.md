> [!info] Note
> The registration and login forms will be placed on the same page

## FrontEnd - page structure
---

- [ ] **Registration**
	- **Form fields**
		- username
		- email
		- password
		- confirm password
	- **Actions**
		- Sign up (calls [[#**HTTP POST** api/auth/register|register endpoint]])
		- switch to login form
	- **Visuals feedback**
		- show validation fields error message
		- a spinner after pressing Sing up button
		- show form error message sent back from the server
	- **Validations**
		- username length
		- email length and format
		- password length and format
		- confirm password must match password
-  [ ] Redirect
	- if Sign up action returns 200 OK, redirect user to the dashboard page

## Endpoints
---
### **HTTP POST** api/auth/register
```json
{
	username: string,
	email: string,
	password: string
}
```


## BackEnd
---
- [ ]  api/auth/register
	- AuthHandler
		- data validation -> 400 BadRequest with error message
		- cheks if username and email already exists -> 400 BadRequest with error message
		- create a hash for password
		- save user in ***users*** table
		- create access and refresh token and save refresh_token in ***refresh_tokens*** table
		- return [[#RegistrationResponse]]
	- based on Client-Type:
		- Mobile (TBD): return 200 OK [[#RegistrationResponse]]
		- Web: set cookie HttpOnly for access_token and refresh_token and return 200 OK

## DTO
___
### RegistrationRequest
```c#
public class RegistrationRequest
{
	public string Username { get; set; }
	public string Email { get; set; }
	public string Password { get; set; }
}
```

### RegistrationResponse
```c#
public class RegistrationResponse
{
	public string AccessToken { get; set; }
	public string RefreshToken { get; set; }
}
```