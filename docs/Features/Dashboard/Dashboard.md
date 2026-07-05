> [!info] Prerequirement
> Before entering this page you must have an active WebSocket connection via SignalR

## FrontEnd - page structure
---

- [ ] **On enter**
	- Initialize listener to the event *NewGameCreated*
	- calls [[#**HTTP GET** api/games/pending|pending games endpoint]] to retrieve pending games to join
- [ ] **Page**
	- **Actions**
		- new game button (calls [[#**HTTP POST** api/games|new game endpoint]])

## Endpoints
---
### **HTTP GET** api/games/pending
### **HTTP POST** api/games
```json
{
	description: string,
	type: enum(games_type),
	difficulty?: enum(difficulty),
	password?: string,
	number_of_player: number
}
```