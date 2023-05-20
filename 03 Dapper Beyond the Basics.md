# Dapper Beyond the Basics

- Stored procedures and dynamic parameters
- List support for IN operator
- Dynamic capabilities
- Bulk insert
- Literal replacements
- Multi mapping
- Other databases: MySQL
- Async

## Dapper SplitOn

Dapper supports multi-mapping, which allows you to map a single row to multiple objects. It is helpful if you need to retrieve data from multiple tables in a single query

- To do this, we need to use the Dapper splitOn parameter when calling the Query method.
- The splitOn parameter tells Dapper what column(s) to use to split the data into multiple objects

```cs
using(var connection = new SqlConnection(connectionString))
{
    var sql = @"SELECT ProductID, ProductName, p.CategoryID, CategoryName FROM Products p INNER JOIN Categories c ON p.CategoryID = c.CategoryID";
				
    var products = await connection.QueryAsync<Product, Category, Product>(sql, (product, category) => {
        product.Category = category;
        return product;
    }, 
    splitOn: "CategoryId" );
	
    products.ToList().ForEach(product => Console.WriteLine($"Product: {product.ProductName}, Category: {product.Category.CategoryName}"));
	
    Console.ReadLine();
}
```

The splitOn argument tells Dapper to split the data on the CategoryId column. Anything up to that column maps to the first parameter Product, and anything else from that column onward should be mapped to the second input parameter Category.

## Dapper Many To Many Relationships

```cs
using(var connection = new SqlConnection(connectionString))
{
    var sql = @"SELECT p.PostId, Headline, t.TagId, TagName
                FROM Posts p 
                INNER JOIN PostTags pt ON pt.PostId = p.PostId
                INNER JOIN Tags t ON t.TagId = pt.TagId";
				
    var posts = await connection.QueryAsync<Post, Tag, Post>(sql, (post, tag) => {
        post.Tags.Add(tag);
        return post;
    }, splitOn: "TagId");
	
    var result = posts.GroupBy(p => p.PostId).Select(g =>
    {
        var groupedPost = g.First();
        groupedPost.Tags = g.Select(p => p.Tags.Single()).ToList();
        return groupedPost;
    });
	
    foreach(var post in result)
    {
        Console.Write($"{post.Headline}: ");
		
        foreach(var tag in post.Tags)
        {
            Console.Write($" {tag.TagName} ");
        }
		
        Console.Write(Environment.NewLine);
    }
}
```
You use multi-mapping to populate both entities in the same way as previously, but this time, you use a grouping function on the result to combine the duplicate post instances and the tags:


## Example


```cs
    return this.db.Query<Contact>(
        "SELECT * FROM Contacts WHERE Id = @Id", 
            new { id }).SingleOrDefault();
```

```cs
// GetAllAsync
    (await this.db.QueryAsync<Contact>("SELECT * FROM Contacts")).ToList();
```

> GetAllContactsWithAddresses

```cs
        public List<Contact> GetAllContactsWithAddresses()
        {
            var sql = "SELECT * FROM Contacts AS C INNER JOIN Addresses AS A ON A.ContactId = C.Id";

            var contactDict = new Dictionary<int, Contact>();

            var contacts = this.db.Query<Contact, Address, Contact>(sql, (contact, address) =>
            {
                if (!contactDict.TryGetValue(contact.Id, out var currentContact))
                {
                    currentContact = contact;
                    contactDict.Add(currentContact.Id, currentContact);
                }

                currentContact.Addresses.Add(address);
                return currentContact;
            });
            return contacts.Distinct().ToList();
        }
```

```cs
        //public List<dynamic> GetDynamicContactsById(params int[] ids)
        public List<Contact> GetContactsById(params int[] ids)
        {
            return this.db.Query<Contact>("SELECT * FROM Contacts WHERE ID IN @Ids", new { Ids = ids }).ToList();
        }
```

```cs
```

```cs
```