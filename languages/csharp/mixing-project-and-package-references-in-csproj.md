[Csharp](/languages/csharp)
# Mixing project and package references in CSPROJ (Sdk-based)

#### Context:
- we have three projects in solution: Service, NugetA, NugetB
- project Service has project dependencies to NugetA and NugetB
- NugetA and NugetB are distributed as nuget packages
- NugetB has package (nuget) dependency on NugetA (because `dotnet pack` does not pull project references; see: https://github.com/dotnet/cli/issues/3959)

#### Error:
```
NU1106: Unable to satisfy conflicting requests for 'X'
```
Such error will be repeated multiple times for many dependencies stating exactly the same dependency versions.

#### Solution:

Use package references everywhere (in this case, also for Service project). No other solutions so far.

#### References:
- Similiar case: https://github.com/NuGet/Home/issues/5483