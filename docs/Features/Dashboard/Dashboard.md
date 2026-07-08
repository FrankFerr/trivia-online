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
	- SignalR -> invoke [[#^db6121|JoinDashboardGroup]]
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
		- insert code in a minimal form
			- **Fields**
				- see [[#JoinGameRequest]] DTO
			- **Actions**
				- join button
					- retreive games by code ([[#^c84106|api/games]])
					- calls [[#^937ced|api/games/{id}/join]]
			- **Visuals feedback**
				- show validation fields error message
			- **Validations**
				- validation for required fields
			- **On Complete Actions**
				- join button return 200 OK -> redirect to the lobby page
					- path parameter: /{new_game_id}

- [ ] **SignalR Events**
	- [ ] **OnNewGameCreated** ^ffeabc
		- **Event Data**
			- [[#OnNewGameResponse]]
		- **Execution Flow**
			- add new game to the list, ignore if already exists
	- [ ] **OnPendingGamesUpdated** ^0fcaa9
		- TODO

## BackEnd
---
- [ ] **GET api/games**^c84106
	- **Response**
		- [[#GameResponse - List]]
	- **Execution Flow**
		- MediatR -> DashboardHandler
			- retrieve games from DB

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
		- SignalR -> invoke [[#^d6d92d|NotifyGameCreated]] ([[#OnNewGameCreatedEventData]])

- [ ] **POST api/games/{id}/join** ^937ced
	- **Request**
		- [[#JoinGameRequest]]
	- **Response**
		- [[#JoinGameResponse]]
	- **Execution Flow**
		- MediatR -> DashboardHandler
			- validate required field
			- game doesn't exists -> 404 Not Found, or is full -> 409 Conflict
			- join user to game (DB)
		- SignalR -> invoke [[#^b03fb7|NotifyJoinedGame]] ([[#OnGameUpdatedEventData]])

- [ ] **SignalR Methods**
	- [ ] **JoinDashboardGroup** ^db6121
		- **Execution Flow**
			- add user to the DashboardGroup

	- [ ] **NotifyGameCreated** ^d6d92d
		- **Parameters**
			- [[#OnNewGameCreatedEventData]]
		- **Execution Flow**
			- notify the dashboardGroup with new game data

	- [ ] **NotifyJoinedGame** ^b03fb7
		- **Parameters**
			- [[#OnGameUpdatedEventData]]
			- string ConnectionId
		- **Execution Flow**
			- remove user from dashboard group
			- add user to game group
			- notify dushboard group with updated data


## DTO
---

### GameResponse - List
```c#
public class GameResponse
{
	public Guid Id;
	public string Description;
	public DateOnly CreatedAt;
	public EDifficulty Difficulty;
	public int NumberOfPlayer;
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
	public string ConnectionId;
}
```

### JoinGameResponse
```c#
public class JoinGameResponse
{
	public Guid Id;
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

### OnGameUpdatedEventData
```c#
public class OnGameUpdatedEventData
{
	public Guid Id;
	public int NumberOfPlayer;
}
```