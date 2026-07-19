> [!info] Prerequirements
> Before entering this page you must have an active WebSocket connection via SignalR

## FrontEnd - page structure
---

### Layout

- navbar
- user info
- create / join / quick game button
- list of available matches and active matches
	- Each active match is an expandable card with match information (minimal lobby)[[#^b47e6a|Minimal lobby]] (host, waiting, your turn, finished)
- footer

### Functionality

- [ ] **On enter**
	- SignalR -> invoke [[Methods#**JoinDashboard**|Join Dashboard]]
	- SignalR -> initialize listener to the event [[#^0fcab8|OnActiveMatchesUpdated]]
	- SignalR -> Initialize listener to the event [[#^0fcaa9|OnAvailableMatchesUpdated]]
	- retrieve active matches
		- calls [[#^c85217|Active matches]] to get matches
		- apply deltas if there are any [[#^e8b760|applyDelta]]
		- retrieve and cache details of joined matches
	- retrieve available matches to join
		- calls [[#^c84106|available matches]] to get matches
		- apply deltas if there are any [[#^e8b760|applyDelta]]
		- retrieve and cache details of joined matches

- [ ] **Page buttons**
	- [ ] **Create new match button**
		- opens modal to create new match
			- **Fields**
				- see [[#NewMatchRequest]]
			- **Actions**
				- create button
					- calls [[#^a51427|create new match]]
					- show match in active match list
				- cancel buttons
			- **Visual feedback**
				- show validation fields error message
			- **Validations**
				- validation for required fields

	- [ ] **Join by code button**
		- insert code in a minimal form
			- **Fields**
				- see [[#JoinMatchRequest]]
			- **Actions**
				- join button
					- retreive games by code ([[#^c84106|api/matches]])
					- calls [[#^937ced|api/matches/{matchId}/players]] -> \[404, 409]
					- insert returned match to *my matches list*
					- greyed out match from available list
			- **Visual feedback**
				- show validation fields error message
				- not found error message
			- **Validations**
				- validation for required fields

	- [ ] **Random match button**
		- **Actions**
			- random match button
				- calls [[#^937abc|api/matches/random/players]] -> \[404, 409]
				- insert returned match to *my matches list*
				- greyed out match from available list
		- **Visual feedback**
			- not found error message

	- [ ] **Join by available match list button**
		- **Actions**
			- join button
				- calls [[#^937ced|api/matches/{matchId}/players]] -> \[404, 409]
				- insert returned match to *my matches list*
				- greyed out match from available list
		- **Visual feedback**
			- not found error message

- [ ] **Card Lobby buttons**^b47e6a
	- [ ] **Cancel match** *(Host only)*
		- **Actions**
			- calls [[#^58e1fc|Delete endpoint]]
			- remove match from list

	- [ ] **Start match** *(Host only)*
		- **Actions**
			- calls *TODO*
		- **Visual feedback**
			- greyed out if there's one participants

	- [ ] **Answer**
		- *TODO*

	- [ ] **Leave match**
		- *TODO*

	- [ ] **Abandon**
		- *TODO*

- [ ] **SignalR Events**
	- [ ] **OnAvailableMatchesUpdated** ^0fcaa9
		- data synchronization by REST is active
			- add delta match to a list
			- return
		- apply delta match to the list of available matches

	- [ ] **OnActiveMatchesUpdated** ^0fcab8
		- data synchronization by REST is active
			- add delta match to a list
			- return
		- apply delta match to the list of active matches

- [ ] **Methods**
	- [ ] applyDelta ^e8b760
		- **Parameters**
			- [[#MatchResponse]]
		- **Execution Flow**
			- Add or remove match from the list based on Action

## BackEnd
---
- [ ] **GET api/matches** ^c84106
	- **Response**
		- List<[[#MatchResponse]]>
	- **Execution Flow**
		- MediatR -> DashboardQuery
			- retrieve matches from DB
		- return response

- [ ] **GET api/matches/me** ^c85217
	- **Response**
		- List<[[#ActiveMatchResponse]]>
	- **Execution Flow**
		- MediatR -> DashboardQuery
			- retrieve matches from DB
		- return response

- [ ] **POST api/matches** ^a51427
	- **Request** ^f4f247
		- [[#NewMatchRequest]]
	- **Response** ^f4f258
		- [[#ActiveMatchResponse]]
	- **Execution Flow**
		- MediatR -> DashboardCommand
			- validate required fields -> 400 Bad Request
			- create new match
			- NotifyService -> [[#^d6d92d|NotifyMatchesUpdated]]
				- parameters
					- [[#MatchResponse]]
			- return [[#^f4f258|Response]]
		- return 201 Created with response

- [ ] **DELETE api/matches/{matchId}** ^58e1fc
	- **Request**
		- route parameter
	- **Execution Flow**
		- user is host? -> 403
		- is started/ended? -> 400
		- retrieve participants ids to notify
		- delete from DB
		- NotifyService -> [[#^d6d92d|NotifyMatchesUpdated]]
			- parameters
				- [[#DeleteMatchResponse]]
		- NotifyService -> [[#^d6d81C|NotifyMatchJoined]]
			- parameters
				- [[#DeleteActiveMatchResponse]]
	- return 200

- [ ] **PUT api/matches/{matchId}/players** ^937ced
	- **Request**
		- route parameter
	- **Response** ^f4f248
		- [[#ActiveMatchResponse]]
	- **Execution Flow**
		- MediatR -> DashboardHandler
			- validate required field
			- match doesn't exists -> 404 Not Found
			- is full -> 409 Conflict
			- join user to match (DB)
			- NotifyService -> [[#^d6d92d|NotifyMatchesUpdated]]
				- parameters
					- [[#MatchResponse]]
			- NotifyService -> [[#^d6d81C|NotifyMatchJoined]]
				- parameters
					- [[#^f4f248|Response]]
			- return [[#^f4f248|Response]]
		- return 200 OK

- [ ] **PUT api/matches/random/players** ^937abc
	- **Response** ^f4f250
		- [[#ActiveMatchResponse]]
	- **Execution Flow**
		- MediatR -> DashboardHandler
			- match not found -> 404 Not Found
			- join user to match (DB)
			- NotifyService -> [[#^d6d92d|NotifyMatchesUpdated]]
				- parameters
					- [[#MatchResponse]]
			- NotifyService -> [[#^d6d81C|NotifyMatchJoined]]
				- parameters
					- [[#^f4f250|Response]]
			- return [[#^f4f250|Response]]
		- return 200 OK

- [ ] **Notify Service**
	- [ ] **NotifyMatchesUpdated** ^d6d92d
		- **Parameters**
			- [[#MatchResponse]]
		- **Execution Flow**
			- notify the dashboardGroup with new match data

	- [ ] **NotifyMatchJoined** ^d6d81C
		- **Parameters**
			- matchId,
			- list of userIds (optional)
		- **Execution Flow**
			- if list of userIds is null fetch them from DB
			- notify the participants with active match data

## DTO
---

### MatchResponse
```c#
public class MatchResponse
{
	public Guid Id;
	public string HostUsername;
	public string CategoryDescription;
	public int NumberOfQuestions;
	public int AnswerTimeoutMinutes;
	public int NumberOfJoinedPlayers;
	public int NumberOfPlayers;
	public DateTime CreatedAt;
	public int version
	public EAction Action;
}
```

### DeleteMatchResponse
```c#
public class DeleteMatchResponse
{
	public Guid Id;
	public EAction Action;
}
```

### ActiveMatchResponse
```c#
public class ActiveMatchResponse
{
	public Guid Id;
	public string Code;
	public string HostUsername;
	public bool IsHost;
	public string CategoryDescription;
	public int NumberOfQuestions;
	public int NumberOfCurrentQuestion;
	public int NumberOfCorrectAnswer;
	public Participant[]? Participants;
	public int AnswerTimeoutMinutes;
	public DateTime CreatedAt;
	public DateTime? StartedAt;
	public DateTime? EndedAt;
	public int version
}
```

### DeleteActiveMatchResponse
```c#
public class DeleteActiveMatchResponse
{
	public Guid Id;
	public EAction Action;
}
```

### Participant
```c#
public class Participant
{
	public string Username;
	public bool? HasAnswered;
	public int? Score;
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