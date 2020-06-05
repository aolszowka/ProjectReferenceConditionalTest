# ProjectReferenceConditionalTest
This Toy Repository highlights the fact that Visual Studio 2019 (as of 16.6.1) does not appear to respect the `Condition` attribute on the `<ProjectReference>` element.

This Fails:

```xml
<ProjectReference Include="..\A\A.csproj" Condition="'1'=='2'">
    <Project>{679190e5-cd34-4391-85de-9031f04f50a5}</Project>
    <Name>A</Name>
</ProjectReference>
```

Using MSBuild from the command line appears to work as expected, but within Visual Studio it will show as a dependency.

This can be worked around using the `<Choose>` element like so which works in both Visual Studio and MSBuild as expected:

```xml
  <Choose>
    <When Condition="'1' == '2'">
      <ItemGroup>
        <ProjectReference Include="..\D\D.csproj">
          <Project>{639424b0-0982-424b-8fdf-0ed91e9e6753}</Project>
          <Name>D</Name>
        </ProjectReference>
      </ItemGroup>
    </When>
  </Choose>
```

However these `<Choose>` Elements can become extremely verbose, and it is disconnected from the `<ProjectReference>` tag which is also undesirable.


## Background
This requirement stems from a design/architecture decision in the product.

The original architects intended the product to utilize Dependency Injection heavily.

However they needed someway to have these DI'ed dependencies in up in both the `bin`<sup id="a1">[1](#f1)</sup> and the `_PublishedWebsites` folder.

The path chosen so far has been to add these as ProjectReferences which will not only copy them to the correct location, but will also make sure that they are up-to-date with regards to recent commits.

However some dependencies are difficult to build, and rarely change, in these cases it is usually better if the dependency does not build unless you open a particular solution, or have some of the code checked out. In these scenarios developers rely on the CI build to give them the binaries they need.

Adding these conditional statements (such as performing a `Condition="Exists(Project.csproj)`) would make the most sense on the ProjectReference itself if possible.

## Footnotes
<b id="f1">1</b> A Common Output directory cannot be used because redirecting the Output Directory Breaks IIS Express. This is because IIS Express does not support having a `/bin` folder that is not located directly beside the Project File.

There are several StackOverflow posts on this issue and various solutions, the most elegant winds up being to use a hardlink in Windows to trick the bin folder into looking in the correct location, however this still does not completely solve the issue when binaries exist in two output folders.

See:
- https://stackoverflow.com/questions/13263002/is-there-a-way-to-change-net-mvc-bin-dir-location
- https://stackoverflow.com/questions/28598196/iis-express-c-sharp-asp-dll
- https://stackoverflow.com/questions/1206138/how-do-i-reference-assemblies-outside-the-bin-folder-in-an-asp-net-application

[↩](#a1)