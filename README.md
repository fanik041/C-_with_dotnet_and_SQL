# C-_with_dotnet_and_SQL

I tried integrating SQL with C# use the following commands to follow and repeat the process (install dotnet on your device first):

1. Change to your home directory. Create a new .NET Core project. This will create the project directory with a basic .NET Core Program.cs and csproj file.
```
dotnet new console -o SqlServerColumnstoreSample
```

2. You should already have a file called SqlServerColumnstoreSample.csproj in your .NET Core project located at: ~/SqlServerColumnstoreSample

Open this file in your favorite text editor and replace the contents with the code below to check if this code is present for the System.Data.SqlClient to your project. Save and close the file if not present.

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

Open this file in your favorite text editor and replace the contents with the code you want to add. follow the pattern shown in the file to make sure your code works. Don’t forget to replace the username and password with your own. Save and close the file.


4. Change directories into the project folder and restore the dependencies in the csproj by running the following commands.

```
cd ~/SqlServerColumnstoreSample
dotnet restore
```


5. build and run

```
dotnet run
```
