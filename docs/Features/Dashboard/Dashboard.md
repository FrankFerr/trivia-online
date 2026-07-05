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
	- Initialize listener to the event *OnNewGameCreated*
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

## BackEnd
---
- [ ] **GET api/games/pending** ^c84106
	- **Response**
		- [[#PendingGamesResponse]]
	- **Execution Flow**
		- DashboardHandler
			- retrieve pending games from DB
		- return 200 OK

- [ ] **POST api/games** ^a51427
	- **Request**
		- [[#NewGameRequest]]
	- **Response**
		- [[#NewGameResponse]]
	- **Execution Flow

## DTO
---

### PendingGamesResponse - List Of
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