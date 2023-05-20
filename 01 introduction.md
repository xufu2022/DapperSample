# Introduction

## Key Features of Dapper

- Query and Map
- Performance
- Simplified API
- Any Database

```cs
var contacts = connection.Query<Contact>(
“SELECT Id, FirstName, LastName FROM Contacts WHERE Id = @Id”, new { id });
```