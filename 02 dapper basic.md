# Dapper Basics

- Installing Dapper
- Repository skeleton
- Basic query
- CRUD operations
- Dapper Contrib
- Objects with parent-child relationships

```cs
    var sql =
        "INSERT INTO Contacts (FirstName, LastName, Email, Company, Title) VALUES(@FirstName, @LastName, @Email, @Company, @Title); " +
        "SELECT CAST(SCOPE_IDENTITY() as int)";
    var id = this.db.Query<int>(sql, contact).Single();
    contact.Id = id;
    return contact;
```

```cs
    return this.db.Query<Contact>(
        "SELECT * FROM Contacts WHERE Id = @Id", 
            new { id }).SingleOrDefault();
```

```cs
        public Contact GetFullContact(int id)
        {
            var sql =
                "SELECT * FROM Contacts WHERE Id = @Id; " +
                "SELECT * FROM Addresses WHERE ContactId = @Id";

            using (var multipleResults = this.db.QueryMultiple(sql, new { Id = id }))
            {
                var contact = multipleResults.Read<Contact>().SingleOrDefault();

                var addresses = multipleResults.Read<Address>().ToList();
                if (contact != null && addresses != null)
                {
                    contact.Addresses.AddRange(addresses);
                }

                return contact;
            }
        }
```

```cs
        public void Save(Contact contact)
        { 
            using var txScope = new TransactionScope();
                this.db.Execute("DELETE FROM Addresses WHERE Id = @Id", new { addr.Id });
            txScope.Complete();
        } 
```

```cs
using Dapper.Contrib.Extensions;

    //Find
        this.db.Get<Contact>(id);
    //GetAll()
        this.db.GetAll<Contact>().ToList();
    //Remove(int id)
        this.db.Delete(new Contact { Id = id });
    //Update(Contact contact)
        this.db.Update(contact);
        
```