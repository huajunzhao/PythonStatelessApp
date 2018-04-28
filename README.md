# PythonStatelessApp


## Getting Started

An example how to run the python exectuable as Service Farbic Stateless application.

The basic guide is https://docs.microsoft.com/en-us/azure/service-fabric/service-fabric-deploy-existing-app#check-your-running-application

Now we get to some hacking. Service Fabric will not run Python.exe in the same way that it runs node.exe in the examples. So what you need to do is create a run.cmd script that basically does this:

python.exe my.py

use that as your EntryPoint.

### Prerequisites

What things you need to install the software and how to install them

Python 3.6.5

your EntryPoint in ServiceManifest.xml looks like this:
```
<EntryPoint>
        <ExeHost>
            <Program>run.cmd</Program>
            <Arguments></Arguments>
            <WorkingFolder>CodePackage</WorkingFolder>
            <ConsoleRedirection FileRetentionCount="5" FileMaxSizeInKb="2048"/> //use this to get logging (very helpful ;)
        </ExeHost>
</EntryPoint>
```

if you want a SetupEntryPoint to install python for you you could do this:

```
<SetupEntryPoint>
  <ExeHost>
    <Program>Setup\setup.bat</Program>
    <WorkingFolder>CodePackage</WorkingFolder>
  </ExeHost>
</SetupEntryPoint>
```

The code in my solution is copied into this structure:

MyGuestExecutablePkg\
    |
    - Code\
        - run.cmd
        - my.py
        - setup.cmd
        - ....
    - Config\
        - Settings.xml
    - ServiceManifest.xml
ApplicationManifest.xml

Policies
Some things need elivated priviledges on the Service Fabric Nodes and this is how you elivate your service in order for it to run .cmd, .bat, Python.exe etc. The following edits are from ApplicationManifest.xml

```
<ServiceManifestImport>
    <ServiceManifestRef ServiceManifestName="MyGuestExecutablePkg" ServiceManifestVersion="1.0.0" />
    <ConfigOverrides />
    <Policies>
      <RunAsPolicy CodePackageRef="Code" UserRef="SetupAdminUser" EntryPointType="Setup" />
      <RunAsPolicy CodePackageRef="Code" UserRef="SetupAdminUser" EntryPointType="Main" />
    </Policies>
  </ServiceManifestImport>
  
```

The first policy is to elevate the SetupEntryPoint while the second is to elicate the EntryPoint, aka. your service. Make sure you define the Principals at the bottom of ApplicationManifest.xml.

```
</DefaultServices>
  <Principals>
    <Users>
      <User Name="SetupAdminUser">
        <MemberOf>
          <SystemGroup Name="Administrators" />
        </MemberOf>
      </User>
    </Users>
  </Principals>
</ApplicationManifest>
```

At last, check out this article https://docs.microsoft.com/en-us/azure/service-fabric/service-fabric-deploy-existing-app#check-your-running-application to know if it works.
