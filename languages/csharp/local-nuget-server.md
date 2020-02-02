[Csharp](/languages/csharp)
# Local NuGet server

### Install NuGet CLI
1. Download `nuget.exe` from https://www.nuget.org/downloads 
2. Place `nuget.exe` in system32 directory `%WINDIR%/system32`

### Create directory for local NuGet feed
1. Create directory where NuGet packages will land; let's name it `C:\local-nuget\`
2. Configure it in Visual Studio as one of the NuGet feeds (as location just add folder path)
3. Tools -> Options -> NuGet Package Manager -> Package Sources -> (add here)

### Generate .nuspec file
1. Go to `.csproj` location from terminal/cmd
2. Run `nuget spec`
3. Edit your `.nuspec` if neccessary (generally `<description>` node will yield errors so place anything there)
4. Run `dotnet pack` (dotnet CLI required I guess) - `dotnet build` before `dotnet pack` is required (`pack` runs only `restore`)
    1. Go to `.\bin\Debug`
    2. Run `nuget add <name-of-package>.nupkg -source C:\local-nuget\`
5. Alternatively, run `nuget pack` in .csproj location and then
    1. Run `nuget add <name-of-package>.nupkg -source C:\local-nuget\`

### Reference from your desired .dll's from NuGet feed from now on
1. Yes, that's it.
