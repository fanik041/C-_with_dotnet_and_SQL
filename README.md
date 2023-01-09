# C-_with_dotnet_and_SQL

I tried integrating SQL with C# use the following commands to follow and repeat the process (install dotnet on your device first):

1. Change to your home directory. Create a new .NET Core project. This will create the project directory with a basic .NET Core Program.cs and csproj file.
```
dotnet new console -o SqlServerColumnstoreSample
```

2. You should already have a file called SqlServerColumnstoreSample.csproj in your .NET Core project located at: ~/SqlServerColumnstoreSample

Open this file in your favorite text editor and replace the contents with the code below to add System.Data.SqlClient to your project. Save and close the file.

```

<Project Sdk="Microsoft.NET.Sdk">

  <PropertyGroup>
    <OutputType>Exe</OutputType>
    <TargetFramework>netcoreapp2.0</TargetFramework>
  </PropertyGroup>

  <ItemGroup>
    <PackageReference Include="System.Data.SqlClient" Version="4.4.0" />
  </ItemGroup>

</Project>
```

3. You should already have a file called Program.cs in your .NET Core project located at: ~/SqlServerColumnstoreSample

Open this file in your favorite text editor and replace the contents with the code below. Don’t forget to replace the username and password with your own. Save and close the file.

```
using System;
using System.Collections.Generic;
using System.Data.SqlClient;
using System.Linq;
using System.Text;
using System.Threading.Tasks;

namespace SqlServerColumnstoreSample
{
    class Program
    {
        static void Main(string[] args)
        {
            try
            {
                Console.WriteLine("*** SQL Server Columnstore demo ***");

                // Build connection string
                SqlConnectionStringBuilder builder = new SqlConnectionStringBuilder();
                builder.DataSource = "localhost";   //  update me
                builder.UserID = "sa";              //  update me
                builder.Password = "your_password";      // update me
                builder.InitialCatalog = "master";

                // Connect to SQL
                Console.Write("Connecting to SQL Server ... ");
                using (SqlConnection connection = new SqlConnection(builder.ConnectionString))
                {
                    connection.Open();
                    Console.WriteLine("Done.");

                    // Create a sample database
                    Console.Write("Dropping and creating database 'SampleDB' ... ");
                    String sql = "DROP DATABASE IF EXISTS [SampleDB]; CREATE DATABASE [SampleDB]";
                    using (SqlCommand command = new SqlCommand(sql, connection))
                    {
                        command.ExecuteNonQuery();
                        Console.WriteLine("Done.");
                    }

                    // Insert 5 million rows into the table 'Table_with_5M_rows'
                    Console.Write("Inserting 5 million rows into table 'Table_with_5M_rows'. This takes ~1 minute, please wait ... ");
                    StringBuilder sb = new StringBuilder();
                    sb.Append("USE SampleDB; ");
                    sb.Append("WITH a AS (SELECT * FROM (VALUES(1),(2),(3),(4),(5),(6),(7),(8),(9),(10)) AS a(a))");
                    sb.Append("SELECT TOP(5000000)");
                    sb.Append("ROW_NUMBER() OVER (ORDER BY a.a) AS OrderItemId ");
                    sb.Append(",a.a + b.a + c.a + d.a + e.a + f.a + g.a + h.a AS OrderId ");
                    sb.Append(",a.a * 10 AS Price ");
                    sb.Append(",CONCAT(a.a, N' ', b.a, N' ', c.a, N' ', d.a, N' ', e.a, N' ', f.a, N' ', g.a, N' ', h.a) AS ProductName ");
                    sb.Append("INTO Table_with_5M_rows ");
                    sb.Append("FROM a, a AS b, a AS c, a AS d, a AS e, a AS f, a AS g, a AS h;");
                    sql = sb.ToString();
                    using (SqlCommand command = new SqlCommand(sql, connection))
                    {
                        command.ExecuteNonQuery();
                        Console.WriteLine("Done.");
                    }

                    // Execute SQL query without columnstore index
                    double elapsedTimeWithoutIndex = SumPrice(connection);
                    Console.WriteLine("Query time WITHOUT columnstore index: " + elapsedTimeWithoutIndex + "ms");

                    // Add a Columnstore Index
                    Console.Write("Adding a columnstore to table 'Table_with_5M_rows'  ... ");
                    sql = "CREATE CLUSTERED COLUMNSTORE INDEX columnstoreindex ON Table_with_5M_rows;";
                    using (SqlCommand command = new SqlCommand(sql, connection))
                    {
                        command.ExecuteNonQuery();
                        Console.WriteLine("Done.");
                    }

                    // Execute the same SQL query again after columnstore index was added
                    double elapsedTimeWithIndex = SumPrice(connection);
                    Console.WriteLine("Query time WITH columnstore index: " + elapsedTimeWithIndex + "ms");

                    // Calculate performance gain from adding columnstore index
                    Console.WriteLine("Performance improvement with columnstore index: "
                        + Math.Round(elapsedTimeWithoutIndex / elapsedTimeWithIndex) + "x!");
                }
                Console.WriteLine("All done. Press any key to finish...");
                Console.ReadKey(true);
            }
            catch (Exception e)
            {
                Console.WriteLine(e.ToString());
            }
        }

        public static double SumPrice(SqlConnection connection)
        {
            String sql = "SELECT SUM(Price) FROM Table_with_5M_rows";
            long startTicks = DateTime.Now.Ticks;
            using (SqlCommand command = new SqlCommand(sql, connection))
            {
                try
                {
                    var sum = command.ExecuteScalar();
                    TimeSpan elapsed = TimeSpan.FromTicks(DateTime.Now.Ticks) - TimeSpan.FromTicks(startTicks);
                    return elapsed.TotalMilliseconds;
                }
                catch (Exception e)
                {
                    Console.WriteLine(e.ToString());
                }
            }
            return 0;
        }
    }
}
```

4. Change directories into the project folder and restore the dependencies in the csproj by running the following commands.

```
cd ~/SqlServerColumnstoreSample
dotnet restore
```
