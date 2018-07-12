# How to Access the inner file in wrapped class .jar

Maven has a [standard directory layout](https://maven.apache.org/guides/introduction/introduction-to-the-standard-directory-layout.html). The directory`src/main/resources`is intended for such application resources. Place your text files into it.

You now basically have two options where exactly to place your files:

1. The resource file belongs to a class.

An example for this is a class representing a GUI element \(a panel\) that needs to also show some images.

In this case place the resource file into the same directory \(package\) as the corresponding class. E.g. for a class named`your.pkg.YourClass`place the resource file into the directory`your/pkg`:

```
src/main
 +-- java/
 |    +-- your/pkg/
 |    |    +-- YourClass.java
 +-- resources/
      +-- your/pkg/
           +-- resource-file.txt
```

You now load the resource via the corresponding class. Inside the class`your.pkg.YourClass`you have the following code snippet for loading:

```
String resource = "resource-file.txt"; // the "file name" without any package or directory
Class<?> clazz = this.getClass(); // or YourClass.class
URL resourceUrl = clazz.getResource(resource);
if (resourceUrl != null) {
    try (InputStream input = resourceUrl.openStream()) {
        // load the resource here from the input stream
    }
}
```

Note: You can also load the resource via the class' class loader:

```
String resource = "your/pkg/resource-file.txt";
ClassLoader loader = this.getClass().getClassLoader(); // or YourClass.class.getClassLoader()
URL resourceUrl = loader.getResource(resource);
if (resourceUrl != null) {
    try (InputStream input = resourceUrl.openStream()) {
        // load the resource here from the input stream
    }
}
```

Choose, what you find more convenient.

1. The resource belongs to the application at whole.

In this case simply place the resource directly into the`src/main/resources`directory or into an appropriate sub directory. Let's look at an example with your lookup file:

```
src/main/resources/
    +-- LookupTables/
         +-- LookUpTable1.txt
```

You then must load the resource via a class loader, using either the current thread's context class loader or the application class loader \(whatever is more appropriate - go and search for articles on this issue if interested\). I will show you both ways:

```
String resource = "LookupTables/LookUpTable1.txt";
ClassLoader ctxLoader = Thread.currentThread().getContextClassLoader();
ClassLoader sysLoader = ClassLoader.getSystemClassLoader();
URL resourceUrl = ctxLoader.getResource(resource); // or sysLoader.getResource(resource)
if (resourceUrl != null) {
    try (InputStream input = resourceUrl.openStream()) {
        // load the resource here from the input stream
    }
}
```

As a first suggestion, use the current thread's context class loader. In a standalone application this will be the system class loader or have the system class loader as a parent. \(The distinction between these class loaders will become important for libraries that also load resources.\)

You should always use a class loader for loading resource. This way you make loading independent from the place \(just take care that the files are inside the class path when launching the application\) and you can package the whole application into a JAR file which still finds the resources.

