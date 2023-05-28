# Build and Package manager Tools

How we deploy our software? How we move our code and dependencies needed to run our app onto our server where our application will run?
We package our application in single movable file. That single moveable file is called **artifact**.

Packaging the app is also called building the code.

The storage where we keep our artifact once we built is called **Artifact repository**. Example are Nexus, Jfrog Artifactory.

What kind of file is the artifact?
Artifact file looks different for each programming language. For Java - JAR (Java Archive) or WAR file. It includes whole code plus dependencies.

`npm pack` to build the artifact for nodejs app. Nodejs does not build any artifact it just pack all the code to zip/tar file.


