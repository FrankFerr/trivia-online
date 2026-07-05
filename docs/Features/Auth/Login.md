> [!info] Note
> The registration and login forms will be placed on the same page

## FrontEnd - page structure
---

- [ ] **Login**
	- **Form fields**
		- username (TBD: unique field for username/email?)
		- password
	- **Actions**
		- Log in (calls [[#**HTTP POST** api/auth/login|login endpoint]])
		- switch to register form
	- **Visuals feedback**
		- show validation fields error message
		- a spinner after pressing Log in button
		- show form error message sent back from the server
	- **Validations**
		- blank fields
-  [ ] Redirect
	- if Login action returns 200 OK, redirect user to the dashboard page


## Endpoints
---
### **HTTP POST** api/auth/login
```json
{
	username: string,
	password: string
}
```


## BackEnd
---
- [ ] AuthHandler
	- data validation (blank fields) -> 401 Unauthorized with error message
	- check if username exists -> 401 Unauthorized with error message
	- check if hashed password matches saved password -> 401 Unauthorized with error message
	- generate access_token and refresh_token
	- save refresh_token in ***refresh_tokens***
	- based on Client-Type:
		- Mobile (TBD): return 200 OK [[#LoignResponse]]
		- Web: set cookie HttpOnly for access_token and refresh_token and return 200 OK

## DTO
___
### LoignRequest
```c#
public class LoignRequest
{
	public string Username { get; set; }
	public string Password { get; set; }
}
```

### LoignResponse
```c#
public class LoignResponse
{
	public string AccessToken { get; set; }
	public string RefreshToken { get; set; }
}
```