# Store Procedure

GetAll()

```cs
public List<Company> GetAll() => db.Query<Company>("usp_GetALLCompany", commandType: CommandType.StoredProcedure).ToList();
```

Find(int id)

```cs
public Company Find(int id) => db.Query<Company>("usp_GetCompany", new { CompanyId = id }, commandType: CommandType.StoredProcedure).SingleOrDefault();
```

Remove(int id)
```cs
public void Remove(int id) => db.Execute("usp_RemoveCompany", new { CompanyId = id }, commandType: CommandType.StoredProcedure);
```

Update
```cs
public Company Update(Company company)
{
    var parameters = new DynamicParameters();
    parameters.Add("@CompanyId", company.CompanyId, DbType.Int32);
    parameters.Add("@Name", company.Name);
    parameters.Add("@Address", company.Address);
    parameters.Add("@City", company.City);
    parameters.Add("@State", company.State);
    parameters.Add("@PostalCode", company.PostalCode);
    this.db.Execute("usp_UpdateCompany", parameters, commandType: CommandType.StoredProcedure);
    
    return company;
}
```

```cs
```

```cs
```