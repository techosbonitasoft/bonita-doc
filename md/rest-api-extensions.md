
# REST API extensions

Create REST API extensions to use third party systems (databases, web services, Bonita Engine, etc) data in forms and pages.

REST API extensions can be used to query [business data](define-and-deploy-the-bdm.md), Bonita Engine APIs, or an external information system (such as a database, web service, LDAP directory...). They also help to keep a clean separation between the front-end (forms, pages, and interfaces visible to users) and the back-end (processes).

## Prerequisites

Prerequisites for developing a REST API extension are:

- Java/Groovy development expertise.

## Example description

The following sections show how to create a REST API extension. As an example, we create a REST API extension that uses the Bonita BPM Engine to provide user information (first name, last name, email address).

There is some additional prerequisites when creating a REST API extension:

- Basic knowledge of Maven
- **Access to [Maven central repository](http://central.maven.org/maven2)**.
- More information on maven configuration [here](configure-maven.md)

### Generate a new REST API extension skeleton

1. In the **Development** menu, choose **REST API Extension** then **New...**.
2. Enter a **Name**, for example _User information REST API Extension_.
3. Enter a **Description**, for example _Query Bonita Engine to retrieve user information_.
4. Enter a package name, use to set the artifact **Group id**, for example: _com.company.rest.api_
5. Enter a **Project name**, for example _userInformationRestAPIExtension_
6. Click **Next**.
7. Enter the **pathTemplate** for this REST API extension, for example _userInformation_. This will be the access point of the API, and follows this pattern: `{bonita_portal_context}/API/extension/userInformation`.
8. As this REST API extension does not access business data you can safely uncheck "Add BDM dependencies" check box.
9. Define a **Permission** name for the extension (replace the default one), for example _read_user_information_. This is the name of the permission the users should have to be granted access to the extension (see [REST API extensions usage](api-extensions.md#usage)
10. Click **Next**
11. This screen defines **URL parameters** that will be passed to the API. By default, _p_ and _c_ parameters are defined to enables paged result, it applies well in our examples as we want to return a list of users.
12. Click **Create**.

### Write the code

#### Main source code

First step would be to remove files and code related to REST API configuration as we don't need to define configuration parameters for our REST API:

1. Delete `configuration.properties` from `src/main/resources` folder
2. Delete `testConfiguration.properties` from `src/test/resources`
3. Remove the setup of configuration file mock. Edit `IndexTest.groovy`, go to `setup()` method and remove the line starting with `resourceProvider...`.
4. Remove the example of configuration usage in `Index.groovy` file (see comment starting with: "Here is an example of you can...").

Now we can add our business logic. In `Index.groovy`, in `doHandle` method, locate the "Your code goes here" comment and add your code below (removing the existing `result` and `return` statement):

```groovy
// Convert parameters from string to int
p = p as int
c = c as int

// Initialize the list to store users information  
def usersInformation = []

// Get the list of user  
List<User> users = context.apiClient.identityAPI.getUsers(p*c, c, UserCriterion.FIRST_NAME_ASC)

// Iterate over each user
for (user in users) {
	// Get user extra information (including email address)
	ContactData contactData = context.apiClient.identityAPI.getUserContactData(user.id, false)

	// Create a map with current user first name, last name and email address
	def userInformation = [firstName: user.firstName, lastName: user.lastName, email: contactData.email]

	// Add current user information to the global list
	usersInformation << userInformation
}

// Prepare the result
def result = [p: p, c: c, userInformation: usersInformation]

int startIndex = p*c
int endIndex = p*c + users.size() - 1

// Send the result as a JSON representation
return buildPagedResponse(responseBuilder, new JsonBuilder(result).toString(), startIndex, endIndex, context.apiClient.identityAPI.numberOfUsers)
```

Make sure you are adding all missing imports (default shortcut CTRL+SHIFT+o).

#### Test source code

Now we need to update the test to verify the behavior of our REST API extension by editing `IndexTest.groovy`.

First step is to define some mocks for our externals dependencies such as Engine Identity API. Add the following mocks declaration after the existing ones:

```groovy
def apiClient = Mock(APIClient)
def identityAPI = Mock(IdentityAPI)
def april = Mock(User)
def william = Mock(User)
def walter = Mock(User)
def contactData = Mock(ContactData)
```

Now we need to define the generic behavior of our mocks. `setup()` method should have the following content:

```groovy
context.apiClient >> apiClient
apiClient.identityAPI >> identityAPI

identityAPI.getUsers(0, 2, _) >> [april, william]
identityAPI.getUsers(1, 2, _) >> [william, walter]
identityAPI.getUsers(2, 2, _) >> [walter]

april.firstName >> "April"
april.lastName >> "Sanchez"
william.firstName >> "William"
william.lastName >> "Jobs"
walter.firstName >> "Walter"
walter.lastName >> "Bates"

identityAPI.getUserContactData(*_) >> contactData
contactData.email >> "test@email"
```

Now you can define a test method. Replace existing test `should_return_a_json_representation_as_result` method with the following one:

```groovy
def should_return_a_json_representation_as_result() {
  given: "a RestAPIController"
  def index = new Index()
  // Simulate a request with a value for each parameter
  httpRequest.getParameter("p") >> "0"
  httpRequest.getParameter("c") >> "2"

  when: "Invoking the REST API"
  def apiResponse = index.doHandle(httpRequest, new RestApiResponseBuilder(), context)

  then: "A JSON representation is returned in response body"
  def jsonResponse = new JsonSlurper().parseText(apiResponse.response)
  // Validate returned response
  apiResponse.httpStatus == 200
  jsonResponse.p == 0
  jsonResponse.c == 2
  jsonResponse.userInformation.equals([
    [firstName:"April", lastName: "Sanchez", email: "test@email"],
    [firstName:"William", lastName: "Jobs", email: "test@email"]
  ]);
}
```

Make sure you are adding all missing imports (default shortcut CTRL+SHIFT+o).

You should now be able to run your unit test. Right click the `IndexTest.groovy` file and click on **REST API Extension** > **Run JUnit Test**. The JUnit view displays the test results. All tests should pass.

### Build, deploy and test the REST API extension

Studio let you build and deploy the REST API extension in the embedded test environment.

First step is to configure security mapping for your extension in Studio embedded test environment:

1. In the **Development** menu, choose **REST API Extension** then **Edit permissions mapping**.
2. Append this line at the end of the file:
   `profile|User=[read_user_information]` This means that anyone logged in with the user profile is granted this permission.
3. Save and close the file.

Now you can actually build and deploy the extension:

1. In the **Development** menu, choose **REST API Extension** > **Deploy...**
2. Select the userInformationRestAPIExtension REST API extension.
3. Click on **Deploy** button.
4. In the coolbar, click the **Portal** icon. This opens the Bonita Portal in your browser.
5. In the Portal, change to the **Administrator** profile.
6. Go to the **Resources** tab, and check that the User information REST API extension is in the list of REST API extension resources.

Now you can finally test your REST API extension:

1. Open a new tab in the web browser
2. Enter the following URL: `http://localhost:8080/bonita/API/extension/userInformation?p=0&c=10`.
3. The JSON response body should be displayed.

The REST API extension can be used in forms and pages in the **UI Designer** using an `External API` variable.

## Example ready to use

You can checkout the Bonita Studio repository that include this extension and a process that use it directly from the Studio by provinding the Git repository URL: <https://github.com/bonitasoft/rest-api-extension-user-information-example>

## BDM and Performance matters

Two maven artifacts are generated from the Business Data Model : **bdm-dao** and **bdm-client**.  
The version of those artifacts is fixed to 1.0.
You have the possibility to edit the group id of those artifacts from the BDM edition wizard.  
Those maven artifacts are meant to be used from REST API extensions, using the following dependencies:  
```xml
<dependency>
	<groupId>[YOUR GROUP ID]</groupId>
	<artifactId>bdm-client</artifactId>
	<version>1.0.0</version>
	<scope>provided</scope>
</dependency>

<dependency>
	<groupId>[YOUR GROUP ID]</groupId>
	<artifactId>bdm-dao</artifactId>
	<version>1.0.0</version>
	<scope>provided</scope>
</dependency>
```
Those dependencies are automatically added when a REST API Extension is created from the Bonita Studio. It allows to manipulate Business Objects from a REST API Extension.  
ℹ️ Only **read operations**  can be performed on business objects from a REST API Extension, even with the dao. Write operations are done through processes. 

Be aware that a poor implementation of a custom REST API accessing BDM objects can lead to poor performance results. See the [best practice](bdm-in-rest-api.md) on this matter.

## Troubleshooting

- I get the following stacktrace when using Java 8 Date types (LocalDate, LocalDateTime...) in my Rest API Extension

```log
java.lang.StackOverflowError
	at java.security.AccessController.doPrivileged(Native Method)
	at java.net.URLClassLoader.findClass(URLClassLoader.java:361)
	at java.lang.ClassLoader.loadClass(ClassLoader.java:424)
	at java.lang.ClassLoader.loadClass(ClassLoader.java:357)
	at org.apache.catalina.loader.WebappClassLoaderBase.loadClass(WebappClassLoaderBase.java:1806)
	at org.apache.catalina.loader.WebappClassLoaderBase.loadClass(WebappClassLoaderBase.java:1735)
	at java.lang.Class.forName0(Native Method)
	at java.lang.Class.forName(Class.java:264)
	at groovy.lang.MetaClassRegistry$MetaClassCreationHandle.createWithCustomLookup(MetaClassRegistry.java:149)
	at groovy.lang.MetaClassRegistry$MetaClassCreationHandle.create(MetaClassRegistry.java:144)
	at org.codehaus.groovy.reflection.ClassInfo.getMetaClassUnderLock(ClassInfo.java:253)
	at org.codehaus.groovy.reflection.ClassInfo.getMetaClass(ClassInfo.java:285)
	at org.codehaus.groovy.reflection.ClassInfo.getMetaClass(ClassInfo.java:295)
	at org.codehaus.groovy.runtime.metaclass.MetaClassRegistryImpl.getMetaClass(MetaClassRegistryImpl.java:261)
	at org.codehaus.groovy.runtime.InvokerHelper.getMetaClass(InvokerHelper.java:871)
	at org.codehaus.groovy.runtime.DefaultGroovyMethods.getMetaPropertyValues(DefaultGroovyMethods.java:364)
	at org.codehaus.groovy.runtime.DefaultGroovyMethods.getProperties(DefaultGroovyMethods.java:383)
	at groovy.json.JsonOutput.writeObject(JsonOutput.java:290)
	at groovy.json.JsonOutput.writeIterator(JsonOutput.java:445)
	at groovy.json.JsonOutput.writeObject(JsonOutput.java:269)
	at groovy.json.JsonOutput.writeMap(JsonOutput.java:424)
	at groovy.json.JsonOutput.writeObject(JsonOutput.java:294)
	at groovy.json.JsonOutput.writeIterator(JsonOutput.java:441)
	at groovy.json.JsonOutput.writeObject(JsonOutput.java:269)
	at groovy.json.JsonOutput.writeMap(JsonOutput.java:424)
	at groovy.json.JsonOutput.writeObject(JsonOutput.java:294)
```

The [`groovy.json.JSONBuilder`](http://docs.groovy-lang.org/2.4.4/html/gapi/groovy/json/JsonBuilder.html) does not support Java 8 Date types serialization for the groovy version currently used by Bonita.

As a workaround you have to format dates in a new data structure before using the JSONBuilder.

Example:

```groovy
def employee = //A given employee object
def result = [
    name:employee.name,
    birthDate:employee.birthDate.format(DateTimeFormatter.ISO_LOCAL_DATE)
]
return buildResponse(responseBuilder, HttpServletResponse.SC_OK,new JsonBuilder(result).toPrettyString())
```

::: info
We do not recommend to manage time zone at the Rest API level, as the local of the Rest API server, the Bonita Engine server, and the End User machine could be different.
So we encourage you to manipulate UTC dates only server-side.
You can see how we [manage the time zone using the date time picker](datetimes-management-tutorial.md#toc2). This time zone should only be managed in the end user interface.
:::