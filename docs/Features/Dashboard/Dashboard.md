> [!info] Prerequirements
> Before entering this page you must have an active WebSocket connection via SignalR

## FrontEnd - page structure
---

### Layout

- navbar
- user info
- create / join / quick game button
- list of available matches
- footer

### Functionality

- [ ] **On enter**
	- SignalR -> invoke [[Methods#**JoinDashboard**|Join Dashboard]]
	- SignalR -> Initialize listener to the event [[#^0fcaa9|OnAvailableMatchesUpdated]]
	- retrieve available matches to join
		- calls [[#^c84106|available matches]] to get matches
		- apply deltas if there are any

<!-- -->
- [ ] **Page**
	- [ ] **Create new match button**
		- opens modal to create new match
			- **Fields**
				- see [[#NewMatchRequest]] DTO
			- **Actions**
				- create button
					- calls [[#^a51427|create new match]]
					- 201 Created
						- redirect user to lobby page "*lobby/{new_match_id}*"
				- cancel buttons
			- **Visuals feedback**
				- show validation fields error message
			- **Validations**
				- validation for required fields
	- [ ] **Join by code button**
		- insert code in a minimal form
			- **Fields**
				- see [[#JoinMatchRequest]] DTO
			- **Actions**
				- join button
					- retreive games by code ([[#^c84106|api/games]])
					- calls [[#^937ced|api/games/{id}/join]]
					- move returned match to *my matches list*
			- **Visuals feedback**
				- show validation fields error message
				- not found error message
			- **Validations**
				- validation for required fields

<!-- -->
- [ ] **SignalR Events**
	- [ ] **OnAvailableMatchesUpdated** ^0fcaa9
		- TODO

## BackEnd
---
- [ ] **GET api/matches**^c84106
	- **Response**
		- List<[[#MatchResponse]]>
	- **Execution Flow**
		- MediatR -> DashboardQuery
			- retrieve matches from DB
		- return response

<!-- -->
- [ ] **POST api/matches** ^a51427
	- **Request** ^f4f247
		- [[#NewMatchRequest]]
	- **Response**
		- [[#MatchResponse]]
	- **Execution Flow**
		- MediatR -> DashboardCommand
			- validate required fields -> 400 Bad Request
			- create new match
			- NotifyService -> [[#^d6d92d|NotifyMatchesUpdated]]
				- parameters
					- [[#^f4f247|Response]]
			- return [[#^f4f247|Response]]
		- return 201 Created

<!-- -->
- [ ] **POST api/matches/{id}/players** ^937ced
	- **Response** ^f4f248
		- [[#MatchResponse]]
	- **Execution Flow**
		- MediatR -> DashboardHandler
			- validate required field
			- game doesn't exists -> 404 Not Found
			- is full -> 409 Conflict
			- join user to game (DB)
			- NotifyService -> [[#^d6d92d|NotifyMatchesUpdated]]
				- parameters
					- [[#^f4f248|Response]]
			- return [[#^f4f248|Response]]
		- return 200 OK

<!-- -->
- [ ] **Notify Service**
	- [ ] **NotifyMatchesUpdated** ^d6d92d
		- **Parameters**
			- [[#OnNewGameCreatedEventData]]
		- **Execution Flow**
			- notify the dashboardGroup with new game data


## DTO
---

### MatchResponse
```c#
public class MatchResponse
{
	public Guid Id;
	public string Description;
	public DateOnly CreatedAt;
	public EDifficulty Difficulty;
	public int NumberOfJoinedPlayer;
	public int NumberOfPlayer;
	public Guid[] ParticipantIds;
	public int NumberOfQuestions;
	public int AnswerTimeoutMinutes;
	public EAction Action;
}
```

### NewMatchRequest
```c#
public class NewMatchRequest
{
	public string? Description;
	public EGameTypes Type;
	public EDifficulty Difficulty = EDifficulty.Progressive;
	public string? Password;
	public int? NumberOfPlayer = 2;
	public int? NumberOfQuestions = 7;
	public int? AnswerTimeoutMinutes = 720;
}
```

### JoinMatchRequest
```c#
public class JoinMatchRequest
{
	public Guid Id;
}
```