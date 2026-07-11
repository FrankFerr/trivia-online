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
		- apply deltas if there are any [[#^e8b760|applyDelta]]

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
					- retreive games by code ([[#^c84106|api/matches]])
					- calls [[#^937ced|api/matches/{id}/players]]
					- move returned match to *my matches list*
			- **Visuals feedback**
				- show validation fields error message
				- not found error message
			- **Validations**
				- validation for required fields
	- [ ] **Random match button**
		- **Actions**
			- random match button
				- calls [[#^937abc|api/matches/random/players]] -> \[404, 409]
				- move returned match to *my matches list*
		- **Visuals feedback**
			- not found error message
	- [ ] **Join by available match list button**
		- **Fields**
			- see [[#JoinMatchRequest]] DTO
		- **Actions**
			- join button
				- calls [[#^937ced|api/matches/{id}/players]] -> \[404, 409]
				- move returned match to *my matches list*
		- **Visuals feedback**
			- not found error message

<!-- -->
- [ ] **SignalR Events**
	- [ ] **OnAvailableMatchesUpdated** ^0fcaa9
		- data synchronization by REST is active
			- add delta match to a list
			- return
		- apply delta match to the list

<!-- -->
- [ ] **Methods**
	- [ ] applyDelta ^e8b760
		- **Parameters**
			- [[#MatchResponse]]
		- **Execution Flow**
			- Add or remove match from the list based on Action

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
		- return 201 Created with response

<!-- -->
- [ ] **POST api/matches/{id}/players** ^937ced
	- **Response** ^f4f248
		- [[#MatchResponse]]
	- **Execution Flow**
		- MediatR -> DashboardHandler
			- validate required field
			- match doesn't exists -> 404 Not Found
			- is full -> 409 Conflict
			- join user to match (DB)
			- NotifyService -> [[#^d6d92d|NotifyMatchesUpdated]]
				- parameters
					- [[#^f4f248|Response]]
			- return [[#^f4f248|Response]]
		- return 200 OK

<!-- -->
- [ ] **POST api/matches/random/players** ^937abc
	- **Response** ^f4f250
		- [[#MatchResponse]]
	- **Execution Flow**
		- MediatR -> DashboardHandler
			- match not found -> 404 Not Found
			- join user to match (DB)
			- NotifyService -> [[#^d6d92d|NotifyMatchesUpdated]]
				- parameters
					- [[#^f4f250|Response]]
			- return [[#^f4f250|Response]]
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