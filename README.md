# Trello API Testing with Postman

A Postman collection that tests the [Trello REST API](https://developer.atlassian.com/cloud/trello/) end to end: **34 requests and 64 test assertions** covering the full create → read → update → delete lifecycle of boards, lists and cards, plus labels, checklists, due dates, a batch call and an unauthorized-access check.

> Personal practice project for learning API testing. Not affiliated with or endorsed by Trello or Atlassian.

## Start here: a 2-minute tour

1. **Open it in Postman.** Click **Import**, drop in [collection/Trello_API_Tests.postman_collection.json](collection/Trello_API_Tests.postman_collection.json), and you will see the 34 requests in order.
2. **Open any request and look at its Tests tab.** That is where the checks live (examples below).
3. **Prefer not to install anything?** Skip to [What one request looks like](#what-one-request-looks-like) and [All 34 requests](#all-34-requests) below.

To actually run it you need your own Trello API key and token (see [How to run](#how-to-run)).

## What it covers

The collection holds two scenarios that run top to bottom:

| # | Scenario | Requests | What it verifies |
|---|---|---|---|
| 1 | CRUD lifecycle (Board → List → Card) | 1–19 | Create each resource, read it back, update it, read it back again, then delete it and confirm it is gone (404) |
| 2 | Workflow and edge cases | 20–34 | Private board, lookup of its To Do / Doing / Done lists, label creation, card with label and due date, checklist with item, moving a card to "Done" and completing its due date, a request without a token (401), a batch request, and clean-up |

| Resource | Operations tested |
|---|---|
| Boards | list, get, create, update, delete |
| Lists | create, get, update, archive, get lists on a board |
| Cards | create (plain, and with label + due date), get, update, move to another list, delete |
| Labels | create, list on board |
| Checklists | create on card, add item |
| Batch | board + card + list fetched in one `GET /batch` call |

## What one request looks like

**Create Board**: `POST {{base_url}}/1/boards/` (Tests tab, shortened, with comments added)

```js
let jsonData = pm.response.json();
pm.environment.set("boardID", jsonData.id);        // chaining: save the ID for later requests

pm.test("Response contains id", function () {
    pm.expect(jsonData).to.have.property("id");
});

let createdboardName = pm.environment.get("BoardName");
pm.test("Board Name is correct", function () {
    pm.expect(jsonData.name).to.eql(createdboardName);   // name matches what the pre-request script generated
});
```

**Get a Board (Deleted)**: the same `GET` as before, now expecting the board to be gone

```js
pm.test("Status code is 404", function () {
    pm.response.to.have.status(404);
});

pm.test("Response time is less than 1000ms", function () {
    pm.expect(pm.response.responseTime).to.be.below(1000);
});
```

## Testing techniques demonstrated

- **Status-code assertions** on 21 requests and **response-time assertions** on 19 (limits of 1000 ms and 1500 ms)
- **Response-body assertions**: `id` and `name` present on created resources, archived list has `closed === true`, moved card sits in the "Done" list with `dueComplete === true`, deleted board returns a `null` value
- **Request chaining**: IDs are captured from responses into environment variables (`boardID`, `listID`, `cardID`, `checklistID`, `bug_label_ID`, `todo_list_ID`, `doing_list_ID`, `done_list_ID`) and reused by later requests, so there are no hard-coded IDs
- **Dynamic test data**: pre-request scripts generate unique board, list and card names and a random future due date on every run
- **Negative testing**: three "resource is really gone" checks (404 after delete) and one request with the token removed (401)
- **Clean-up**: created cards and boards are deleted at the end of each scenario
- **Secrets as variables**: the API key and token live only in the environment, never in the collection

<a id="all-34-requests"></a>
<details>
<summary><strong>All 34 requests and what each one checks</strong> (click to expand)</summary>

| # | Method | Request | Checks |
|---|---|---|---|
| 1 | GET | Get All Boards | no assertions |
| 2 | POST | Create Board | Response contains id; Response contains name; Response contains url; Board Name is correct · saves `boardID` |
| 3 | GET | Get a Board (Created) | Status code is 200; Response time is less than 1000ms |
| 4 | PUT | Update a Board | Status code is 200; Response time is less than 1000ms |
| 5 | GET | Get a Board (Updated) | Status code is 200; Response time is less than 1000ms |
| 6 | POST | Create a List | Response contains id; Response contains name; List Name is correct · saves `listID` |
| 7 | GET | Get a List (Created) | Status code is 200; Response time is less than 1000ms |
| 8 | PUT | Update a List | Status code is 200; Response time is less than 1000ms |
| 9 | GET | Get a List (Updated) | Status code is 200; Response time is less than 1000ms |
| 10 | POST | Create a Card | Response contains id; Response contains name; Card Name is correct · saves `cardID` |
| 11 | GET | Get a Card (Created) | Status code is 200; Response time is less than 1000ms |
| 12 | PUT | Update a Card | Status code is 200; Response time is less than 1000ms |
| 13 | GET | Get a Card (Updated) | Status code is 200; Response time is less than 1000ms |
| 14 | DELETE | Delete a Card | Status code is 200; Response time is less than 1000ms |
| 15 | GET | Get a Card (Deleted) | Status code is 404; Response time is less than 1000ms |
| 16 | PUT | Archive List | List is archived/closed; Status code is 200; Response time is less than 1000ms |
| 17 | DELETE | Delete Board | Status code is 200; Response body is null; Response time is less than 1000ms |
| 18 | GET | Get a List (Deleted) | Status code is 404; Response time is less than 1000ms |
| 19 | GET | Get a Board (Deleted) | Status code is 404; Response time is less than 1000ms |
| 20 | POST | Create a board with Custom creds | Board created successfully · saves `boardID` |
| 21 | GET | Get a Board (Created) | Status code is 200; Response time is less than 1000ms |
| 22 | POST | Create a List | Response contains id; Response contains name; List Name is correct · saves `listID` |
| 23 | GET | Get lists on Board | no assertions · saves `todo_list_ID`, `doing_list_ID`, `done_list_ID` |
| 24 | POST | Create a Label on Board | no assertions · saves `bug_label_ID` |
| 25 | GET | Get Board Labels | no assertions |
| 26 | POST | Create Card with Label, Due Date | Response contains id; Response contains name; Card Name is correct; Card created with correct label · saves `cardID` |
| 27 | POST | Add Checklist to Card | Checklist created successfully; Checklist ID is set · saves `checklistID` |
| 28 | POST | Add Checklist Item | no assertions |
| 29 | GET | Get a Card (Created) | Status code is 200; Response time is less than 1000ms |
| 30 | PUT | Move Card, Complete Due | Card moved to Done and completed |
| 31 | GET | Get Board no Token | Status code is 401 Unauthorized |
| 32 | GET | Batch Request | Batch request processed every item |
| 33 | DELETE | Delete a Card | Status code is 200; Response time is less than 1000ms |
| 34 | DELETE | Delete Board | Status code is 200; Response body is null; Response time is less than 1500ms |

</details>

## Results

![Collection Runner results](docs/runner-results.png)

## Repository layout

```
postman-api-testing/
├── collection/Trello_API_Tests.postman_collection.json
├── environment/Trello_API.postman_environment.template.json   # placeholders only
├── docs/runner-results.png
└── README.md
```

## How to run

1. Create a Trello API key and token by following Trello's [developer documentation](https://developer.atlassian.com/cloud/trello/guides/rest-api/authorization). Use a **throwaway workspace**: the collection creates and deletes real boards in the account the key belongs to.
2. In Postman, import both files from `collection/` and `environment/`.
3. Duplicate the environment template and fill in `key` and `token` (`base_url` is already `https://api.trello.com`). Keep them as **secret** variables.
4. Select that environment, open the collection and use **Run collection**, running every request in order. Order matters: later requests use IDs saved by earlier ones.

## Security notes

- Trello authorizes these calls through `key` and `token` **query parameters**, so they show up in the Postman Console output. Never publish logs or screenshots that include them, and revoke the token in Trello if it leaks.
- Real environment files are git-ignored; only the placeholder template is committed.

## Limitations

- Requests depend on each other's saved IDs, so a failure early in a scenario cascades. If a run is stopped halfway, delete the leftover `myboard…` / `QA_Automation_Board` boards by hand.
- Five requests have no assertions (Get All Boards, Get lists on Board, Create a Label on Board, Get Board Labels, Add Checklist Item), and two of them only save IDs for later requests.
- The three "Create" requests check the response body but not the status code.
- No JSON-schema validation and few invalid-input cases (malformed IDs, empty names, expired token).

## Author

Vladyslav Saulenko, QA Engineer · [GitHub](https://github.com/voidtess) · [LinkedIn](https://www.linkedin.com/in/voidtess)
