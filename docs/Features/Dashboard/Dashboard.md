> [!info] Prerequirements
> Before entering this page you must have an active WebSocket connection via SignalR

## FrontEnd - page structure
---

### Layout

- navbar
- user info
- create / join / quick game button
- list of pending games
- footer

### Functionality

- [ ] **On enter**
	- SignalR -> Initialize listener to the event [[#^ffeabc|OnNewGameCreated]]
	- SignalR -> Initialize listener to the event [[#^0fcaa9|OnPendingGamesUpdated]]
	- SignalR -> invoke *JoinDashboardGroup*
	- calls [[#^c84106|pending games]] to retrieve pending games to join
- [ ] **Page**
	- [ ] **Create new game button**
		- opens modal to create new game
			- **Fields**
				- see [[#NewGameRequest]] DTO
			- **Actions**
				- create button (calls [[#^a51427|create new game]])
				- cancel buttons
			- **Visuals feedback**
				- show validation fields error message
			- **Validations**
				- validation for required fields
			- **On Complete Actions**
				- create button return 200 OK -> redirect to the lobby page
					- path parameter: /{new_game_id}

	- [ ] **Join by code button**
		- open modal to insert code
			- **Fields**
				- see [[#JoinGameRequest]] DTO
			- **Actions**
				- join button (calls)

- [ ] **SignalR Events**
	- [ ] **OnNewGameCreated** ^ffeabc
		- **Event Data**
			- [[#OnNewGameResponse]]
		- **Execution Flow**
			- add new game to the list, ignore if already exists
	- [ ] **OnPendingGamesUpdated** ^0fcaa9
		- 

## BackEnd
---
- [ ] **GET api/games/pending** ^c84106
	- **Response**
		- [[#PendingGamesResponse]]
	- **Execution Flow**
		- MediatR -> DashboardHandler
			- retrieve pending games from DB
		- return 200 OK

- [ ] **POST api/games** ^a51427
	- **Request**
		- [[#NewGameRequest]]
	- **Response**
		- [[#NewGameResponse]]
	- **Execution Flow**
		- MediatR -> DashboardHandler
			- validate required fields
			- create new game
			- return ID new game
		- SignalR -> invoke [[#^d6d92d|OnNewGameCreated]] ([[#OnNewGameCreatedEventData]])
		- return [[#NewGameResponse]]

- [ ] **SignalR Methods**
	- [ ] **JoinDashboardGroup**
		- **Execution Flow**
			- add user to the DashboardGroup
	- [ ] **OnNewGameCreated** ^d6d92d
		- **Event Data**
			- [[#OnNewGameCreatedEventData]]
		- **Execution Flow**
			- notify the dashboardGroup with new game data

## DTO
---

### PendingGamesResponse - List
```c#
public class PendingGamesResponse
{
	public Guid Id;
	public string Description;
	public DateOnly CreatedAt;
	public EDifficulty Difficulty;
	public int NumberOfQuestions;
	public int AnswerTimeoutMinutes;
}
```

### NewGameRequest
```c#
{
	public string? Description;
	public EGameTypes Type;
	public EDifficulty? Difficulty;
	public string? Password;
	public int? NumberOfPlayer = 2;
	public int? NumberOfQuestions = 7;
	public int? AnswerTimeoutMinutes = 720;
}
```

### NewGameResponse
```c#
{
	public Guid Id;
}
```

### JoinGameRequest
```c#
public class JoinGameRequest
{
	public string code;
}
```

### OnNewGameCreatedEventData
```c#
public class OnNewGameCreatedEventData
{
	public Guid Id;
	public string Description;
	public DateOnly CreatedAt;
	public EDifficulty Difficulty;
	public int NumberOfQuestions;
	public int AnswerTimeoutMinutes;
}
```