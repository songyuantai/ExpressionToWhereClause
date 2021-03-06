# ExpressionToWhereClause.Net 
A simple tool library for converting the Expression to sql where clause

[![Build Status](https://zhurongbo.visualstudio.com/Normal/_apis/build/status/ETWC%20Publish?branchName=master)](https://zhurongbo.visualstudio.com/Normal/_build/latest?definitionId=9&branchName=master)

Packages
--------

NuGet feed: https://www.nuget.org/packages/ExpressionToWhereClause/

| Package | NuGet Stable | NuGet Pre-release | Downloads |
| ------- | ------------ | ----------------- | --------- |
| [ExpressionToWhereClause](https://www.nuget.org/packages/ExpressionToWhereClause/) | [![ExpressionToWhereClause](https://img.shields.io/nuget/v/ExpressionToWhereClause.svg)](https://www.nuget.org/packages/ExpressionToWhereClause/) | [![ExpressionToWhereClause](https://img.shields.io/nuget/vpre/ExpressionToWhereClause.svg)](https://www.nuget.org/packages/ExpressionToWhereClause/) | [![ExpressionToWhereClause](https://img.shields.io/nuget/dt/ExpressionToWhereClause.svg)](https://www.nuget.org/packages/ExpressionToWhereClause/) |

Features
--------
ExpressionToWhereClause is a [NuGet library](https://www.nuget.org/packages/ExpressionToWhereClause) that you can add into your project that will extend your `Expression<Func<TModel, bool>>` type.

It provides only one Method:

**Explain of Type `Expression<Func<TModel, bool>>` to the `parametric` sql where clause and the parameter list**

```csharp
 public static (string, Dictionary<string, object>) ToWhereClause<T>(this Expression<Func<T, bool>> expression, bool? nonParametric = null) where T : class
```

The the right part of `Func<TModel, bool>` must like:

`[model].[PropertyName]`   `[comparator]`   `[Value]`, or the combinations.

**Example:**
```csharp
u.Name == "Foo"
```
Or
```csharp
u.Name == "Foo" || u.Name == "Bar"
```

The `[Value]` can be from many places, not only the constant. For the detailed information, please see the example usage.

![#f03c15](https://placehold.it/15/f03c15/000000?text=+)  Warning: This library dose not support unary, like `u => !(u.Name == "Foo")`, but support `u => u.Name != "Foo"` and `u => !u.Sex` Sex is bool type
--------




**Example usage:**

```csharp
[TestClass]
public class ExpressionTest
{
    [TestMethod]
    public void ValidateBool()
    {
        Expression<Func<User, bool>> expression = u => !u.Sex;
        
        (string sql, Dictionary<string, object> parameters) = expression.ToWhereClause();
        Dictionary<string, object> expectedParameters = new Dictionary<string, object>();
        expectedParameters.Add("@Sex", false);
        Assert.AreEqual("Sex = @Sex", sql);
        AssertParameters(expectedParameters, parameters);
    }
    [TestMethod]
    public void ValidateBool2()
    {
        Expression<Func<User, bool>> expression = u => u.Sex && u.Name.StartsWith("Foo");
        (string sql, Dictionary<string, object> parameters) = expression.ToWhereClause();
        Dictionary<string, object> expectedParameters = new Dictionary<string, object>();
        expectedParameters.Add("@Sex", true);
        expectedParameters.Add("@Name", "Foo");
        Assert.AreEqual("(Sex = @Sex) AND (Name like @Name'%')", sql);
        AssertParameters(expectedParameters, parameters);

    }

    [TestMethod]
    public void ValidateConstant()
    {
        Expression<Func<User, bool>> expression = u => u.Age >= 20;
        (string whereClause, Dictionary<string, object> parameters) = expression.ToWhereClause();
        Dictionary<string, object> expectedParameters = new Dictionary<string, object>();
        expectedParameters.Add("@Age", 20);
        Assert.AreEqual("Age >= @Age", whereClause);
        AssertParameters(expectedParameters, parameters);
    }

    [TestMethod]
    public void ValidateVariable()
    {
        int age = 20;
        Expression<Func<User, bool>> expression = u => u.Age >= age;
        (string whereClause, Dictionary<string, object> parameters) = expression.ToWhereClause();
        Dictionary<string, object> expectedParameters = new Dictionary<string, object>();
        expectedParameters.Add("@Age", 20);
        Assert.AreEqual("Age >= @Age", whereClause);
        AssertParameters(expectedParameters, parameters);
    }

    [TestMethod]
    public void ValidateAnd()
    {
        Expression<Func<User, bool>> expression = u => u.Sex && u.Age > 20;

        (string sql, Dictionary<string, object> parameters) = expression.ToWhereClause();
        Dictionary<string, object> expectedParameters = new Dictionary<string, object>();
        expectedParameters.Add("@Sex", true);
        expectedParameters.Add("@Age", 20);
        Assert.AreEqual("(Sex = @Sex) AND (Age > @Age)", sql);
        AssertParameters(expectedParameters, parameters);
    }

    [TestMethod]
    public void ValidateOr()
    {
        Expression<Func<User, bool>> expression = u => u.Sex || u.Age > 20;

        (string sql, Dictionary<string, object> parameters) = expression.ToWhereClause();
        Dictionary<string, object> expectedParameters = new Dictionary<string, object>();
        expectedParameters.Add("@Sex", true);
        expectedParameters.Add("@Age", 20);
        Assert.AreEqual("(Sex = @Sex) OR (Age > @Age)", sql);
        AssertParameters(expectedParameters, parameters);
    }

    [TestMethod]
    public void ValidateDuplicateField()
    {
        Expression<Func<User, bool>> expression = u => u.Age < 15 || u.Age > 20;

        (string sql, Dictionary<string, object> parameters) = expression.ToWhereClause();
        Dictionary<string, object> expectedParameters = new Dictionary<string, object>();
        expectedParameters.Add("@Age", 15);
        expectedParameters.Add("@Age1", 20);
        Assert.AreEqual("(Age < @Age) OR (Age > @Age1)", sql);
        AssertParameters(expectedParameters, parameters);
    }

    [TestMethod]
    public void ValidateMemberConstant()
    {
        UserFilter userFilter = new UserFilter();
        userFilter.Age = 20;
        Expression<Func<User, bool>> expression = u => u.Age < userFilter.Age;
        (string whereClause, Dictionary<string, object> parameters) = expression.ToWhereClause();
        Dictionary<string, object> expectedParameters = new Dictionary<string, object>();
        expectedParameters.Add("@Age", 20);
        Assert.AreEqual("Age < @Age", whereClause);
        AssertParameters(expectedParameters, parameters);
    }



    [TestMethod]
    public void ValidateDeepMemberConstant()
    {
        UserFilter userFilter = new UserFilter();
        userFilter.Internal.Age = 20;
        Expression<Func<User, bool>> expression = u => u.Age < userFilter.Internal.Age;
        (string whereClause, Dictionary<string, object> parameters) = expression.ToWhereClause();
        Dictionary<string, object> expectedParameters = new Dictionary<string, object>();
        expectedParameters.Add("@Age", 20);
        Assert.AreEqual("Age < @Age", whereClause);
        AssertParameters(expectedParameters, parameters);
    }

    [TestMethod]
    public void ValidateInstanceMethodConstant()
    {
        Expression<Func<User, bool>> expression = u => u.Age < GetInt();
        (string whereClause, Dictionary<string, object> parameters) = expression.ToWhereClause();
        Dictionary<string, object> expectedParameters = new Dictionary<string, object>();
        expectedParameters.Add("@Age", 20);
        Assert.AreEqual("Age < @Age", whereClause);
        AssertParameters(expectedParameters, parameters);
    }

    [TestMethod]
    public void ValidateStaticMethodConstant()
    {
        Expression<Func<User, bool>> expression = u => u.Age < UserFilter.GetInt(20);
        (string whereClause, Dictionary<string, object> parameters) = expression.ToWhereClause();
        Dictionary<string, object> expectedParameters = new Dictionary<string, object>();
        expectedParameters.Add("@Age", 20);
        Assert.AreEqual("Age < @Age", whereClause);
        AssertParameters(expectedParameters, parameters);
    }

    [TestMethod]
    public void ValidateMethodChainConstant()
    {
        Expression<Func<User, bool>> expression = u => u.Age < int.Parse(GetInt().ToString());
        (string whereClause, Dictionary<string, object> parameters) = expression.ToWhereClause();
        Dictionary<string, object> expectedParameters = new Dictionary<string, object>();
        expectedParameters.Add("@Age", 20);
        Assert.AreEqual("Age < @Age", whereClause);
        AssertParameters(expectedParameters, parameters);
    }

    [TestMethod]
    public void ValidateMethodConstant2()
    {
        UserFilter userFilter = new UserFilter();
        userFilter.Internal.Age = 20;
        Expression<Func<User, bool>> expression = u => u.Age < GetInt(userFilter.Internal.Age);
        (string whereClause, Dictionary<string, object> parameters) = expression.ToWhereClause();
        Dictionary<string, object> expectedParameters = new Dictionary<string, object>();
        expectedParameters.Add("@Age", 20);
        Assert.AreEqual("Age < @Age", whereClause);
        AssertParameters(expectedParameters, parameters);
    }

    [TestMethod]
    public void ValidateEqualMethod()
    {
        UserFilter filter = new UserFilter();
        filter.Name = "Name";
        Expression<Func<User, bool>> expression = u => u.Name.Equals(filter.Name.Substring(1));
        (string whereClause, Dictionary<string, object> parameters) = expression.ToWhereClause();
        Dictionary<string, object> expectedParameters = new Dictionary<string, object>();
        expectedParameters.Add("@Name", "ame");
        Assert.AreEqual("Name = @Name", whereClause);
        AssertParameters(expectedParameters, parameters);
    }

    [TestMethod]
    public void ValidateTernary()
    {
        string name = "Gary";
        Expression<Func<User, bool>> expression = u => u.Name == (name == null ? "Foo" : name);
        (string whereClause, Dictionary<string, object> parameters) = expression.ToWhereClause();
        Dictionary<string, object> expectedParameters = new Dictionary<string, object>();
        expectedParameters.Add("@Name", "Gary");
        Assert.AreEqual("Name = @Name", whereClause);
        AssertParameters(expectedParameters, parameters);
    }

    [TestMethod]
    public void ValidateNotEqual()
    {
        Expression<Func<User, bool>> expression = u => u.Age != 20;
        (string whereClause, Dictionary<string, object> parameters) = expression.ToWhereClause();
        Dictionary<string, object> expectedParameters = new Dictionary<string, object>();
        expectedParameters.Add("@Age", 20);

        Assert.AreEqual("Age <> @Age", whereClause);
        AssertParameters(expectedParameters, parameters);
    }

    [TestMethod]
    public void ValidateStartsWithMethod()
    {
        UserFilter filter = new UserFilter();
        filter.Name = "Name";
        Expression<Func<User, bool>> expression = u => u.Name.StartsWith(filter.Name);
        (string whereClause, Dictionary<string, object> parameters) = expression.ToWhereClause();
        Dictionary<string, object> expectedParameters = new Dictionary<string, object>();
        expectedParameters.Add("@Name", "Name");
        Assert.AreEqual("Name like @Name'%'", whereClause);
        AssertParameters(expectedParameters, parameters);
    }

    [TestMethod]
    public void ValidateStartWith2()
    {
        UserFilter filter = new UserFilter();
        filter.Name = "Name";
        Expression<Func<User, bool>> expression = u => u.Name.StartsWith(filter.Name) || u.Name.Contains("Start");
        (string whereClause, Dictionary<string, object> parameters) = expression.ToWhereClause();
        Dictionary<string, object> expectedParameters = new Dictionary<string, object>();
        expectedParameters.Add("@Name", "Name");
        expectedParameters.Add("@Name1", "Start");
        Assert.AreEqual("(Name like @Name'%') OR (Name like '%'@Name1'%')", whereClause);
        AssertParameters(expectedParameters, parameters);
    }

    [TestMethod]
    public void ValidateEndsWithMethod()
    {
        UserFilter filter = new UserFilter();
        filter.Name = "Name";
        Expression<Func<User, bool>> expression = u => u.Name.EndsWith(filter.Name);
        (string whereClause, Dictionary<string, object> parameters) = expression.ToWhereClause();
        Dictionary<string, object> expectedParameters = new Dictionary<string, object>();
        expectedParameters.Add("@Name", "Name");
        Assert.AreEqual("Name like '%'@Name", whereClause);
        AssertParameters(expectedParameters, parameters);
    }

    [TestMethod]
    public void ValidateContainsMethod()
    {
        UserFilter filter = new UserFilter();
        filter.Name = "Name";
        Expression<Func<User, bool>> expression = u => u.Name.Contains(filter.Name);
        (string whereClause, Dictionary<string, object> parameters) = expression.ToWhereClause();
        Dictionary<string, object> expectedParameters = new Dictionary<string, object>();
        expectedParameters.Add("@Name", "Name");
        Assert.AreEqual("Name like '%'@Name'%'", whereClause);
        AssertParameters(expectedParameters, parameters);
    }

    [TestMethod]
    public void ValidateUnary()
    {
        try
        {
            Expression<Func<User, bool>> expression = u => !u.Name.Contains("Name");
            expression.ToWhereClause();
        }
        catch(Exception e)
        {
            Assert.IsTrue(e.GetType() == typeof(NotSupportedException));
        }
    }

    [TestMethod]
    public void ValidateSqlAdapter()
    {
        ExpressionConfigurations.SetSqlAdapter(new SqlServerAdapter());
        Expression<Func<User, bool>> expression = u => u.Name == "Foo";
        (string whereClause, Dictionary<string, object> parameters) = expression.ToWhereClause();
        Dictionary<string, object> expectedParameters = new Dictionary<string, object>();
        expectedParameters.Add("@Name", "Foo");
        Assert.AreEqual("[Username] = @Name", whereClause);
        AssertParameters(expectedParameters, parameters);
    }

    [TestMethod]
    public void ValidateNonParametric()
    {
        UserFilter filter = new UserFilter();
        filter.Internal.Age = 20;
        filter.Name = "Gary";
        Expression<Func<User, bool>> expression =
            u => ((u.Sex && u.Age > 18) || (!u.Sex && u.Age > filter.Internal.Age))
              && (u.Name == filter.Name || u.Name.Contains(filter.Name.Substring(1, 2)));
        (string whereClause, Dictionary<string, object> parameters) = expression.ToWhereClause(true);
        Dictionary<string, object> expectedParameters = new Dictionary<string, object>();
        expectedParameters.Add("@Sex", true);
        expectedParameters.Add("@Age", 18);
        expectedParameters.Add("@Sex1", false);
        expectedParameters.Add("@Age1", 20);
        expectedParameters.Add("@Name", "Gary");
        expectedParameters.Add("@Name1", "ar");
        Assert.AreEqual("(((Sex = 'True') AND (Age > '18')) OR ((Sex = 'False') AND (Age > '20'))) AND ((Name = 'Gary') OR (Name like '%ar%'))", whereClause);
        Assert.AreEqual(whereClause, expression.ToWhereClauseNonParametric());
        Assert.AreEqual(0, parameters.Count);
    }

    [TestMethod]
    public void ValidateAll()
    {
        UserFilter filter = new UserFilter();
        filter.Internal.Age = 20;
        filter.Name = "Gary";
        Expression<Func<User, bool>> expression = 
            u => ((u.Sex && u.Age > 18) || (u.Sex == false && u.Age > filter.Internal.Age)) 
              && (u.Name == filter.Name || u.Name.Contains(filter.Name.Substring(1,2)));
        (string whereClause, Dictionary<string, object> parameters) = expression.ToWhereClause();
        Dictionary<string, object> expectedParameters = new Dictionary<string, object>();
        expectedParameters.Add("@Sex", true);
        expectedParameters.Add("@Age", 18);
        expectedParameters.Add("@Sex1", false);
        expectedParameters.Add("@Age1", 20);
        expectedParameters.Add("@Name", "Gary");
        expectedParameters.Add("@Name1", "ar");
        Assert.AreEqual("(((Sex = @Sex) AND (Age > @Age)) OR ((Sex = @Sex1) AND (Age > @Age1))) AND ((Name = @Name) OR (Name like '%'@Name1'%'))", whereClause);
        AssertParameters(expectedParameters, parameters);
    }

    [TestMethod]
    public void ValidateMultipleThreads()
    {
        ConcurrentBag<int> countList = new ConcurrentBag<int>();
        int count = 20;
        List<Task> tasks = new List<Task>();
        for (int i = 0; i < count; i++)
        {
            int k = i;
            Task task = Task.Run(() => {
                UserFilter filter = new UserFilter();
                filter.Internal.Age = 20;
                filter.Name = "Gary"+k;
                Expression<Func<User, bool>> expression =
                    u => ((u.Sex && u.Age > 18) || (!u.Sex && u.Age > filter.Internal.Age))
                      && (u.Name == filter.Name || u.Name.Contains(filter.Name.Substring(1, 2)));
                (string whereClause, Dictionary<string, object> parameters) = expression.ToWhereClause(true);
                Dictionary<string, object> expectedParameters = new Dictionary<string, object>();
                expectedParameters.Add("@Sex", true);
                expectedParameters.Add("@Age", 18);
                expectedParameters.Add("@Sex1", false);
                expectedParameters.Add("@Age1", 20);
                expectedParameters.Add("@Name", "Gary"+k);
                expectedParameters.Add("@Name1", "ar");
                Assert.AreEqual("(((Sex = 'True') AND (Age > '18')) OR ((Sex = 'False') AND (Age > '20'))) AND ((Name = 'Gary"+k+"') OR (Name like '%ar%'))", whereClause);
                Assert.AreEqual(whereClause, expression.ToWhereClauseNonParametric());
                Assert.AreEqual(0, parameters.Count);
                countList.Add(0);
            });

            tasks.Add(task);
        }
        for (int i = 0; i < count; i++)
        {
            int k = i;
            Task task = Task.Run(() => {
                UserFilter filter = new UserFilter();
                filter.Internal.Age = 20;
                filter.Name = "Gary"+k;
                Expression<Func<User, bool>> expression =
                    u => ((u.Sex && u.Age > 18) || (u.Sex == false && u.Age > filter.Internal.Age))
                      && (u.Name == filter.Name || u.Name.Contains(filter.Name.Substring(1, 2)));
                (string whereClause, Dictionary<string, object> parameters) = expression.ToWhereClause();
                Dictionary<string, object> expectedParameters = new Dictionary<string, object>();
                expectedParameters.Add("@Sex", true);
                expectedParameters.Add("@Age", 18);
                expectedParameters.Add("@Sex1", false);
                expectedParameters.Add("@Age1", 20);
                expectedParameters.Add("@Name", "Gary"+k);
                expectedParameters.Add("@Name1", "ar");
                Assert.AreEqual("(((Sex = @Sex) AND (Age > @Age)) OR ((Sex = @Sex1) AND (Age > @Age1))) AND ((Name = @Name) OR (Name like '%'@Name1'%'))", whereClause);
                AssertParameters(expectedParameters, parameters);
                countList.Add(0);
            });

            tasks.Add(task);
        }

        Task.WaitAll(tasks.ToArray());
        Assert.AreEqual(count*2, countList.Count);
    }

    [TestCleanup]
    public void TestCleanup()
    {
        ExpressionConfigurations.SetSqlAdapter(new DefaultSqlAdapter());
    }

    public class SqlServerAdapter : DefaultSqlAdapter
    {
        public override string GetColumnName(MemberInfo mi)
        {
            if (mi.IsDefined(typeof(ColumnAttribute)))
            {
                ColumnAttribute columnAttribute = mi.GetCustomAttribute<ColumnAttribute>();
                return $"[{columnAttribute.Name}]";
            }
            else
            {
                return $"[{mi.Name}]";
            }
        }
    }

    private int GetInt()
    {
        return 20;
    }



    private int GetInt(int i)
    {
        return i;
    }

    private void AssertParameters(Dictionary<string, object> expectedParameters, Dictionary<string, object> actualParameters)
    {
        Assert.AreEqual(expectedParameters.Count, actualParameters.Count);
        foreach(string key in expectedParameters.Keys)
        {
            Assert.IsTrue(actualParameters.ContainsKey(key),$"The parameters does not contain key '{key}'");
            Assert.IsTrue(expectedParameters[key].Equals(actualParameters[key]),$"The expected value is {expectedParameters[key]}, the actual value is {actualParameters[key]}");
        }
    }
}

public class UserFilter
{
    public string Name { get; set; }

    public bool Sex { get; set; }

    public int Age { get; set; }

    public Internal Internal { get; set; } = new Internal();

    public static int GetInt(int i)
    {
        return i;
    }
}

public class Internal
{
    public int Age { get; set; }
}
public class User
{
    [Column("Username")]
    public string Name { get;set; }

    public bool Sex { get; set; }

    public int Age { get; set; }
}

public class ColumnAttribute : Attribute
{
    public ColumnAttribute(string name)
    {
        this.Name = name;
    }


    public string Name { get; set; }
}
```
