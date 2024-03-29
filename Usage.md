Note that if you're using version 1.9.0 or earlier please refer to the [legacy](Usage_Legacy) documentation.

REST Assured is a Java DSL for simplifying testing of REST based services built on top of HTTP Builder. It supports POST, GET, PUT, DELETE, OPTIONS, PATCH and HEAD requests and can be used to validate and verify the response of these requests.

# Contents
1. [Static imports](#static-imports)
1. [Examples](#examples)
  1. [JSON Example](#example-1---json)
  1. [JSON Schema Validation](#json-schema-validation)
  1. [XML Example](#example-2---xml)
  1. [Advanced](#example-3---complex-parsing-and-validation)
    1. [XML](#xml-example)
    2. [JSON](#json-example)
  1. [Additional Examples](#additional-examples)
1. [Note on floats and doubles](#note-on-floats-and-doubles)
1. [Note on syntax](#note-on-syntax) ([syntactic sugar](#syntactic-sugar))
1. [Getting Response Data](#getting-response-data)
  1. [Extracting values from the Response after validation](#extracting-values-from-the-response-after-validation)
  1. [JSON (using JsonPath)](#json-using-jsonpath)
  1. [XML (using XmlPath)](#xml-using-xmlpath)
  1. [Single Path](#single-path)
  1. [Headers, cookies, status etc](#headers-cookies-status-etc)
    1. [Multi-value headers](#multi-value-headers)
    1. [Multi-value cookies](#multi-value-cookies)
    1. [Detailed Cookies](#detailed-cookies)
1. [Specifying Request Data](#specifying-request-data)
  1. [Invoking HTTP resources](#invoking-http-resources)
  1. [Parameters](#parameters)
  1. [Multi-value Parameter](#multi-value-parameter)
  1. [No-value Parameter](#no-value-parameter)
  1. [Path Parameters](#path-parameters)
  1. [Cookies](#cookies)
  1. [Headers](#headers)
  1. [Content-Type](#content-type)
  1. [Request Body](#request-body)
1. [Verifying Response Data](#verifying-response-data)
  1. [Response Body](#response-body)
  1. [Cookies](#cookies-1)
  1. [Status](#status)
  1. [Headers](#headers-1)
  1. [Content-Type](#content-type-1)
  1. [Full body/content matching](#full-bodycontent-matching)
  1. [Use the response to verify other parts of the response](#use-the-response-to-verify-other-parts-of-the-response)
  1. [Measuring response time](#measuring-response-time)
1. [Authentication](#authentication)
  1. [Basic](#basic-authentication)
    1. [Preemptive](#preemptive-basic-authentication)
    1. [Challenged](#challenged-basic-authentication)
  1. [Digest](#digest-authentication)
  1. [Form](#form-authentication)
    1. [CSRF](#csrf)
  1. [OAuth](#oauth)
    1. [OAuth1](#oauth-1)
    1. [OAuth2](#oauth-2)
1. [Multi-part form data](#multi-part-form-data)
1. [Object Mapping](#object-mapping)
  1. [Serialization](#serialization)
    1. [Content-Type based Serialization](#content-type-based-serialization)
    1. [Create JSON from a HashMap](#create-json-from-a-hashmap)
    1. [Using an Explicit Serializer](#using-an-explicit-serializer)
  1. [Deserialization](#deserialization)
    1. [Content-Type based Deserialization](#content-type-based-deserialization)
    1. [Custom Content-Type Deserialization](#custom-content-type-deserialization)
    1. [Using an Explicit Deserializer](#using-an-explicit-deserializer)
  1. [Configuration](#configuration)
  1. [Custom](#custom)
1. Parsers
  1. [Custom](#custom-parsers)
  1. [Default](#default-parser)
1. [Default Values](#default-values)
1. [Specification Re-use](#specification-re-use)
1. [Filters](#filters)
  1. [Ordered Filters](#ordered-filters)
  1. [Response Builder](#response-builder)
1. [Logging](#logging)
  1. [Request Logging](#request-logging)
  1. [Response Logging](#response-logging)
  1. [Log if validation fails](#log-if-validation-fails)
1. [Root Path](#root-path)
  1. [Path Arguments](#path-arguments)
1. [Session Support](#session-support)
  1. [Session Filter](#session-filter)
1. [SSL](#ssl)
  1. [SSL invalid hostname](#ssl-invalid-hostname)
1. [URL Encoding](#url-encoding)
1. [Proxy Configuration](#proxy-configuration)
  1. [Static Proxy Configuration](#static-proxy-configuration)
  1. [Request Specification Proxy Configuration](#request-specification-proxy-configuration)
1. [Detailed configuration](#detailed-configuration)
  1. [Encoder Config](#encoder-config)
  1. [Decoder Config](#decoder-config)
  1. [Session Config](#session-config)
  1. [Redirect DSL](#redirect-dsl)
  1. [Connection Config](#connection-config)
  1. [JSON Config](#json-config)
  1. [HTTP Client Config](#http-client-config)
  1. [SSL Config](#ssl-config)
  1. [Param Config](#param-config)
1. [Spring Mock Mvc Module](#spring-mock-mvc-module)
  1. [Bootstrapping RestAssuredMockMvc](#bootstrapping-restassuredmockmvc)
  1. [Asynchronous Requests](#asynchronous-requests)
  1. [Adding Request Post Processors](#adding-request-post-processors)
  1. [Adding Result Handlers](#adding-result-handlers)
  1. [Using Result Matchers](#using-result-matchers)
  1. [Interceptors](#interceptors)
  1. [Specifications](#specifications)
  1. [Resetting RestAssuredMockMvc](#resetting-restassuredmockmvc)
  1. [Spring MVC Authentication](#spring-mvc-authentication)
     1. [Using Spring Security Test](#using-spring-security-test)
     1. [Injecting a User](#injecting-a-user)
  1. [Note on parameters](#note-on-parameters)
1. [Scala Support Module](#scala-support-module)
1. [Kotlin](#kotlin)
1. [More Info](#more-info)

## Static imports ##

In order to use REST assured effectively it's recommended to statically import methods from the following classes:

```java
io.restassured.RestAssured.*
io.restassured.matcher.RestAssuredMatchers.*
org.hamcrest.Matchers.*
```

If you want to use [Json Schema](http://json-schema.org/) validation you should also statically import these methods:

```java
io.restassured.module.jsv.JsonSchemaValidator.*
```

Refer to [Json Schema Validation](#json-schema-validation) section for more info.

If you're using Spring MVC you can use the [spring-mock-mvc](#spring-mock-mvc-module) module to unit test your Spring Controllers using the Rest Assured DSL. To do this statically import the methods from [RestAssuredMockMvc](http://static.javadoc.io/io.restassured/spring-mock-mvc/3.0.6/io/restassured/module/mockmvc/RestAssuredMockMvc.html) _instead_ of importing the methods from `io.restassured.RestAssured`:

```java
io.restassured.module.mockmvc.RestAssuredMockMvc.*
```
# Examples

## Example 1 - JSON ##
Assume that the GET request (to http://localhost:8080/lotto) returns JSON as:

```javascript
{
"lotto":{
 "lottoId":5,
 "winning-numbers":[2,45,34,23,7,5,3],
 "winners":[{
   "winnerId":23,
   "numbers":[2,45,34,23,3,5]
 },{
   "winnerId":54,
   "numbers":[52,3,12,11,18,22]
 }]
}
}
```

REST assured can then help you to easily make the GET request and verify the response. E.g. if you want to verify that lottoId is equal to 5 you can do like this:

```java
get("/lotto").then().body("lotto.lottoId", equalTo(5));
```
or perhaps you want to check that the winnerId's are 23 and 54:

```java
get("/lotto").then().body("lotto.winners.winnerId", hasItems(23, 54));
```

Note: `equalTo` and `hasItems` are Hamcrest matchers which you should statically import from `org.hamcrest.Matchers`.

Note that the "json path" syntax uses <a href='http://groovy-lang.org/processing-xml.html#_gpath'>Groovy's GPath</a> notation and is not to be confused with Jayway's <a href='https://github.com/jayway/JsonPath'>JsonPath</a> syntax.

### Returning floats and doubles as BigDecimal ###

You can configure Rest Assured and JsonPath to return BigDecimal's instead of float and double for Json Numbers. For example consider the following JSON document:

```javascript
{

    "price":12.12 

}
```

By default  you validate that price is equal to 12.12 as a float like this:

```java
get("/price").then().body("price", is(12.12f));
```

but if you like you can configure REST Assured to use a JsonConfig that returns all Json numbers as BigDecimal:

```java
given().
        config(RestAssured.config().jsonConfig(jsonConfig().numberReturnType(BIG_DECIMAL))).
when().
        get("/price").
then().
        body("price", is(new BigDecimal(12.12));
```

### JSON Schema validation ###

From version `2.1.0` REST Assured has support for [Json Schema](http://json-schema.org/) validation. For example given the following schema located in the classpath as `products-schema.json`:
```javascript
{
    "$schema": "http://json-schema.org/draft-04/schema#",
    "title": "Product set",
    "type": "array",
    "items": {
        "title": "Product",
        "type": "object",
        "properties": {
            "id": {
                "description": "The unique identifier for a product",
                "type": "number"
            },
            "name": {
                "type": "string"
            },
            "price": {
                "type": "number",
                "minimum": 0,
                "exclusiveMinimum": true
            },
            "tags": {
                "type": "array",
                "items": {
                    "type": "string"
                },
                "minItems": 1,
                "uniqueItems": true
            },
            "dimensions": {
                "type": "object",
                "properties": {
                    "length": {"type": "number"},
                    "width": {"type": "number"},
                    "height": {"type": "number"}
                },
                "required": ["length", "width", "height"]
            },
            "warehouseLocation": {
                "description": "Coordinates of the warehouse with the product",
                "$ref": "http://json-schema.org/geo"
            }
        },
        "required": ["id", "name", "price"]
    }
}
```
you can validate that a resource (`/products`) conforms with the schema:
```java
get("/products").then().assertThat().body(matchesJsonSchemaInClasspath("products-schema.json"));
```
`matchesJsonSchemaInClasspath` is statically imported from `io.restassured.module.jsv.JsonSchemaValidator` and it's recommended to statically import all methods from this class. However in order to use it you need to depend on the `json-schema-validator` module by either [downloading](http://dl.bintray.com/johanhaleby/generic/json-schema-validator-3.0.6-dist.zip) it from the download page or add the following dependency from Maven:
```xml
<dependency>
    <groupId>io.rest-assured</groupId>
    <artifactId>json-schema-validator</artifactId>
    <version>3.0.6</version>
</dependency>
```

### JSON Schema Validation Settings ###

REST Assured's `json-schema-validator` module uses Francis Galiegue's [json-schema-validator](https://github.com/fge/json-schema-validator) (`fge`) library to perform validation. If you need to configure the underlying `fge` library you can for example do like this:

```java
// Given
JsonSchemaFactory jsonSchemaFactory = JsonSchemaFactory.newBuilder().setValidationConfiguration(ValidationConfiguration.newBuilder().setDefaultVersion(DRAFTV4).freeze()).freeze();

// When
get("/products").then().assertThat().body(matchesJsonSchemaInClasspath("products-schema.json").using(jsonSchemaFactory));
```

The `using` method allows you to pass in a `jsonSchemaFactory` instance that REST Assured will use during validation. This allows fine-grained configuration for the validation.

The `fge` library also allows the validation to be `checked` or `unchecked`. By default REST Assured uses `checked` validation but if you want to change this you can supply an instance of [JsonSchemaValidatorSettings](http://static.javadoc.io/io.restassured/json-schema-validator/3.0.6/io/restassured/module/jsv/JsonSchemaValidatorSettings.html) to the matcher. For example:

```java
get("/products").then().assertThat().body(matchesJsonSchemaInClasspath("products-schema.json").using(settings().with().checkedValidation(false)));
```

Where the `settings` method is statically imported from the [JsonSchemaValidatorSettings](http://static.javadoc.io/io.restassured/json-schema-validator/3.0.6/io/restassured/module/jsv/JsonSchemaValidatorSettings.html) class.

### Json Schema Validation with static configuration ###

Now imagine that you always want to use `unchecked` validation as well as setting the default json schema version to version 3. Instead of supplying this to all matchers throughout your code you can define it statically. For example:

```java
JsonSchemaValidator.settings = settings().with().jsonSchemaFactory(
        JsonSchemaFactory.newBuilder().setValidationConfiguration(ValidationConfiguration.newBuilder().setDefaultVersion(DRAFTV3).freeze()).freeze()).
        and().with().checkedValidation(false);

get("/products").then().assertThat().body(matchesJsonSchemaInClasspath("products-schema.json"));
```

Now any `matcher` method imported from [JsonSchemaValidator](http://static.javadoc.io/io.restassured/json-schema-validator/3.0.6/io/restassured/module/jsv/JsonSchemaValidatorSettings.html) will use `DRAFTV3` as default version and unchecked validation.

To reset the `JsonSchemaValidator` to its default settings simply call the `reset` method:

```java
JsonSchemaValidator.reset();
```

### Json Schema Validation without REST Assured ###
You can also use the `json-schema-validator` module without depending on REST Assured. As long as you have a JSON document represented as a `String` you can do like this:

```java
import org.junit.Test;
import static io.restassured.module.jsv.JsonSchemaValidator.matchesJsonSchemaInClasspath;
import static org.hamcrest.MatcherAssert.assertThat;
 
public class JsonSchemaValidatorWithoutRestAssuredTest {
 
 
    @Test public void
    validates_schema_in_classpath() {
        // Given
        String json = ... // Greeting response
 
        // Then
        assertThat(json, matchesJsonSchemaInClasspath("greeting-schema.json"));
    }
}
```

Refer to the [getting started](GattingStarted) page for more info on this.

### Anonymous JSON root validation ###

A JSON document doesn't necessarily need a named root attribute. This is for example valid JSON:

```javascript
[1, 2, 3]
```

An anonymous JSON root can be verified by using `$` or an empty string as path. For example let's say that this JSON document is exposed from `http://localhost:8080/json` then we can validate it like this with REST Assured:

```java
when().
     get("/json").
then().
     body("$", hasItems(1, 2, 3)); // An empty string "" would work as well
```

## Example 2 - XML ##
XML can be verified in a similar way. Imagine that a POST request to `http://localhost:8080/greetXML` returns:
```xml
<greeting>
   <firstName>{params("firstName")}</firstName>
   <lastName>{params("lastName")}</lastName>
</greeting>
```
i.e. it sends back a greeting based on the firstName and lastName parameter sent in the request. You can easily perform and verify e.g. the firstName with REST assured:
```java
given().
         parameters("firstName", "John", "lastName", "Doe").
when().
         post("/greetXML").
then().
         body("greeting.firstName", equalTo("John")).
```
If you want to verify both firstName and lastName you may do like this:
```java
given().
         parameters("firstName", "John", "lastName", "Doe").
when().
         post("/greetXML").
then().
         body("greeting.firstName", equalTo("John")).
         body("greeting.lastName", equalTo("Doe"));
```
or a little shorter:
```java
with().parameters("firstName", "John", "lastName", "Doe").when().post("/greetXML").then().body("greeting.firstName", equalTo("John"), "greeting.lastName", equalTo("Doe"));
```

See [this](http://groovy-lang.org/processing-xml.html#_gpath) link for more info about the syntax (it follows Groovy's [GPath](http://groovy-lang.org/processing-xml.html#_gpath) syntax).

### XML namespaces ###
To make body expectations take namespaces into account you need to declare the namespaces using the [io.restassured.config.XmlConfig](http://static.javadoc.io/io.rest-assured/rest-assured/3.0.6/io/restassured/config/XmlConfig.html). For example let's say that a resource called `namespace-example` located at `http://localhost:8080` returns the following XML:
```xml
<foo xmlns:ns="http://localhost/">
  <bar>sudo </bar>
  <ns:bar>make me a sandwich!</ns:bar>
</foo>
```

You can then declare the `http://localhost/` uri and validate the response:
```java
given().
        config(RestAssured.config().xmlConfig(xmlConfig().declareNamespace("test", "http://localhost/"))).
when().
         get("/namespace-example").
then().
         body("foo.bar.text()", equalTo("sudo make me a sandwich!")).
         body(":foo.:bar.text()", equalTo("sudo ")).
         body("foo.test:bar.text()", equalTo("make me a sandwich!"));
```

The path syntax follows Groovy's XmlSlurper syntax. Note that in versions prior to 2.6.0 the path syntax was *not* following Groovy's XmlSlurper syntax. Please see [release notes](https://github.com/rest-assured/rest-assured/wiki/ReleaseNotes26#non-backward-compatible-changes) for versin 2.6.0 to see how the previous syntax looked like.

### XPath ###

You can also verify XML responses using x-path. For example:

```java
given().parameters("firstName", "John", "lastName", "Doe").when().post("/greetXML").then().body(hasXPath("/greeting/firstName", containsString("Jo")));
```

or

```java
given().parameters("firstName", "John", "lastName", "Doe").post("/greetXML").then().body(hasXPath("/greeting/firstName[text()='John']"));
```

To use namespaces in the XPath expression you need to enable them in the configuration, for example:
```java
given().
        config(RestAssured.config().xmlConfig(xmlConfig().with().namespaceAware(true))).
when().
         get("/package-db-xml").
then().
         body(hasXPath("/db:package-database", namespaceContext));
```

Where `namespaceContext` is an instance of [javax.xml.namespace.NamespaceContext](http://docs.oracle.com/javase/7/docs/api/javax/xml/namespace/NamespaceContext.html).

### Schema and DTD validation ###

XML response bodies can also be verified against an XML Schema (XSD) or DTD.

#### XSD example

```java
get("/carRecords").then().assertThat().body(matchesXsd(xsd));
```

#### DTD example

```java
get("/videos").then().assertThat().body(matchesDtd(dtd));
```

The <code>matchesXsd</code> and <code>matchesDtd</code> methods are Hamcrest matchers which you can import from <a href="http://static.javadoc.io/io.rest-assured/rest-assured/3.0.6/io/restassured/matcher/RestAssuredMatchers.html">io.restassured.matcher.RestAssuredMatchers</a>.<br>
</p>

## Example 3 - Complex parsing and validation ##
This is where REST Assured really starts to shine! Since REST Assured is implemented in Groovy it can be really beneficial to take advantage of Groovy’s collection API. Let’s begin by looking at an example in Groovy:

```groovy
def words = ['ant', 'buffalo', 'cat', 'dinosaur']
def wordsWithSizeGreaterThanFour = words.findAll { it.length() > 4 }
```

At the first line we simply define a list with some words but the second line is more interesting. 
Here we search the words list for all words that are longer than 4 characters by calling the findAll with a Groovy closure. 
The closure has an implicit variable called `it` which represents the current item in the list. 
The result is a new list, `wordsWithSizeGreaterThanFour`, containing `buffalo` and `dinosaur`. 

There are other interesting methods that we can use on collections in Groovy as well, for example:

* `find` – finds the first item matching a closure predicate
* `collect` – collect the return value of calling a closure on each item in a collection
* `sum` – Sum all the items in the collection
* `max`/`min` – returns the max/min values of the collection

So how do we take advantage of this when validating our XML or JSON responses with REST Assured?

### XML Example

Let’s say we have a resource at `http://localhost:8080/shopping` that returns the following XML:

```xml
<shopping>
      <category type="groceries">
        <item>Chocolate</item>
        <item>Coffee</item>
      </category>
      <category type="supplies">
        <item>Paper</item>
        <item quantity="4">Pens</item>
      </category>
      <category type="present">
        <item when="Aug 10">Kathryn's Birthday</item>
      </category>
</shopping>
```

Let’s also say we want to write a test that verifies that the category of type groceries has items Chocolate and Coffee. In REST Assured it can look like this:

```java
when().
       get("/shopping").
then().
       body("shopping.category.find { it.@type == 'groceries' }.item", hasItems("Chocolate", "Coffee"));
```

What's going on here? First of all the XML path `shopping.category` returns a list of all categories. 
On this list we invoke a function, `find`, to return the single category that has the XML attribute, `type`, equal to `groceries`. 
On this category we then continue by getting all the items associated with this category. 
Since there are more than one item associated with the groceries category a list will be returned and we verify this list against the `hasItems` Hamcrest matcher.

But what if you want to get the items and not validate them against a Hamcrest matcher? For this purpose you can use [XmlPath](http://static.javadoc.io/io.restassured/xml-path/3.0.6/io/restassured/path/xml/XmlPath.html):

```java
// Get the response body as a String
String response = get("/shopping").asString();
// And get the groceries from the response. "from" is statically imported from the XmlPath class
List<String> groceries = from(response).getList("shopping.category.find { it.@type == 'groceries' }.item");
```

If the list of groceries is the only thing you care about in the response body you can also use a [shortcut](#single-path):

```java
// Get the response body as a String
List<String> groceries = get("/shopping").path("shopping.category.find { it.@type == 'groceries' }.item");
```

#### Depth-first search

It's actually possible to simplify the previous example even further:

```java
when().
       get("/shopping").
then().
       body("**.find { it.@type == 'groceries' }", hasItems("Chocolate", "Coffee"));
```

`**` is a shortcut for doing depth first searching in the XML document. 
We search for the first node that has an attribute named `type` equal to "groceries". Notice also that we don't end the XML path with "item". 
The reason is that `toString()` is called automatically on the category node which returns a list of the item values.

### JSON Example

Let's say we have a resource at `http://localhost:8080/store` that returns the following JSON document:

```javascript
{  
   "store":{  
      "book":[  
         {  
            "author":"Nigel Rees",
            "category":"reference",
            "price":8.95,
            "title":"Sayings of the Century"
         },
         {  
            "author":"Evelyn Waugh",
            "category":"fiction",
            "price":12.99,
            "title":"Sword of Honour"
         },
         {  
            "author":"Herman Melville",
            "category":"fiction",
            "isbn":"0-553-21311-3",
            "price":8.99,
            "title":"Moby Dick"
         },
         {  
            "author":"J. R. R. Tolkien",
            "category":"fiction",
            "isbn":"0-395-19395-8",
            "price":22.99,
            "title":"The Lord of the Rings"
         }
      ]
   }
}
```

#### Example 1
As a first example let's say we want to make the request to "/store" and assert that the titles of the books with a price less than 10 are "Sayings of the Century" and "Moby Dick":

```java
when().
       get("/store").
then().
       body("store.book.findAll { it.price < 10 }.title", hasItems("Sayings of the Century", "Moby Dick"));
```

Just as in the XML examples above we use a closure to find all books with a price less than 10 and then return the titles of all the books. 
We then use the `hasItems` matcher to assert that the titles are the ones we expect. Using [JsonPath](http://static.javadoc.io/io.restassured/json-path/3.0.6/io/restassured/path/json/JsonPath.html) we can return the titles instead:

```java
// Get the response body as a String
String response = get("/store").asString();
// And get all books with price < 10 from the response. "from" is statically imported from the JsonPath class
List<String> bookTitles = from(response).getList("store.book.findAll { it.price < 10 }.title");
```

#### Example 2
 Let's consider instead that we want to assert that the sum of the length of all author names are greater than 50. 
 This is a rather complex question to answer and it really shows the strength of closures and Groovy collections. 
 In REST Assured it looks like this:
 
 ```java
 when().
        get("/store");
 then().
        body("store.book.author.collect { it.length() }.sum()", greaterThan(50));
```

First we get all the authors (`store.book.author`) and invoke the collect method on the resulting list with the closure `{ it.length() }`. 
What it does is to call the `length()` method on each author in the list and returns the result to a new list. 
On this list we simply call the `sum()` method to sum all the length's. 
The end result is `53` and we assert that it's greater than 50 by using the `greaterThan` matcher. 
But it's actually possible to simplify this even further. Consider the "[words](#example-3---complex-parsing-and-validation)" example again:

```groovy
def words = ['ant', 'buffalo', 'cat', 'dinosaur']
```

Groovy has a very handy way of calling a function for each element in the list by using the spread operator, `*`. For example:

```groovy
def words = ['ant', 'buffalo', 'cat', 'dinosaur']
assert [3, 6, 3, 8] == words*.length()
```

I.e. Groovy returns a new list with the lengths of the items in the words list. We can utilize this for the author list in REST Assured as well:

```java
when().
       get("/store");
then().
       body("store.book.author*.length().sum()", greaterThan(50)).
```

And of course we can use [JsonPath](http://static.javadoc.io/io.restassured/json-path/3.0.6/io/restassured/path/json/JsonPath.html) to actually return the result:

```java
// Get the response body as a string
String response = get("/store").asString();
// Get the sum of all author length's as an int. "from" is again statically imported from the JsonPath class
int sumOfAllAuthorLengths = from(response).getInt("store.book.author*.length().sum()");
// We can also assert that the sum is equal to 53 as expected.
assertThat(sumOfAllAuthorLengths, is(53));
```

## Additional Examples ##
Micha Kops has written a really good blog with several examples (including code examples that you can checkout). You can read it [here](http://www.hascode.com/2011/10/testing-restful-web-services-made-easy-using-the-rest-assured-framework/).

Also [Bas Dijkstra](https://www.linkedin.com/in/basdijkstra) has been generous enough to open source his REST Assured workshop. You can read more about this [here](http://www.ontestautomation.com/open-sourcing-my-workshop-an-experiment/) and you can try out, and contribute to, the exercises available in [his](https://github.com/basdijkstra/workshops/) github repository. 


## Note on floats and doubles ##
Floating point numbers must be compared with a Java "float" primitive. For example, if we consider the following JSON object:

```javascript
{

    "price":12.12 

}
```
the following test will fail, because we compare with a "double" instead of a "float":

```java
get("/price").then().assertThat().body("price", equalTo(12.12));
```

Instead, compare with a float with:

```java
get("/price").then().assertThat().body("price", equalTo(12.12f));
```

## Note on syntax ##
When reading blogs about REST Assured you may see a lot of examples using the "given / expect / when" syntax, for example:
```java
given().
        param("x", "y").
expect().
        body("lotto.lottoId", equalTo(5)).
when().
        get("/lotto");
```

This is the so called "legacy syntax" which was the de facto way of writing tests in REST Assured 1.x. While this works fine it turned out to be quite confusing and annoying for many users. The reason for not using "given / when / then" in the first place was mainly technical. So prior to REST Assured 2.0 there was no support "given / when / then" which is more or less the standard approach when you're doing some kind of BDD-like testing. The "given / expect / when" approach still works fine in 2.0 but "given / when / then" reads better and is easier to understand for most people and is thus recommended in most cases. There's however one benefit of using the "given / expect / when" approach and that is that ALL expectation errors can be displayed at the same time which is not possible with the new syntax (since the expectations are defined last). This means that if you would have had multiple expectations in the previous example such as

```java
given().
        param("x", "y").
expect().
        statusCode(400).
        body("lotto.lottoId", equalTo(6)).
when().
        get("/lotto");
```

REST Assured will report that both the status code expectation and the body expectation are wrong. Rewriting this with the new syntax

```java
given().
        param("x", "y").
when().
        get("/lotto").
then().
        statusCode(400).
        body("lotto.lottoId", equalTo(6));
```

will only report an error at the first failed expectation / assertion (that status code was expected to be 400 but it was actually 200). You would have to re-run the test in order to catch the second error.

### Syntactic Sugar ###
Another thing worth mentioning is that REST Assured contains some methods that are only there for syntactic sugar. For example the "and" method which can add readability if you're writing everything in a one-liner, for example:

```java
given().param("x", "y").and().header("z", "w").when().get("/something").then().assertThat().statusCode(200).and().body("x.y", equalTo("z"));
```

This is the same thing as:

```java
given().
        param("x", "y").
        header("z", "w").
when().
        get("/something").
then().
        statusCode(200).
        body("x.y", equalTo("z"));
```

# Getting Response Data #
You can also get the content of a response. E.g. let's say you want to return the body of a get request to "/lotto". You can get it a variety of different ways:
```java
InputStream stream = get("/lotto").asInputStream(); // Don't forget to close this one when you're done
byte[] byteArray = get("/lotto").asByteArray();
String json = get("/lotto").asString();
```

## Extracting values from the Response after validation ##
You can extract values from the response or return the response instance itself after you've done validating the response by using the `extract` method. This is useful for example if you want to use values from the response in sequent requests. For example given that a resource called `title` returns the following JSON
```javascript
 {
     "title" : "My Title",
      "_links": {
              "self": { "href": "/title" },
              "next": { "href": "/title?page=2" }
           }
 }
```
and you want to validate that content type is equal to `JSON` and the title is equal to `My Title`
but you also want to extract the link to the `next` title to use that in a subsequent request. This is how:

```java
String nextTitleLink =
given().
        param("param_name", "param_value").
when().
        get("/title").
then().
        contentType(JSON).
        body("title", equalTo("My Title")).
extract().
        path("_links.next.href");

get(nextTitleLink). ..
```

You could also decide to instead return the entire response if you need to extract multiple values from the response:
```java
Response response = 
given().
        param("param_name", "param_value").
when().
        get("/title").
then().
        contentType(JSON).
        body("title", equalTo("My Title")).
extract().
        response(); 

String nextTitleLink = response.path("_links.next.href");
String headerValue = response.header("headerName");
```

## JSON (using JsonPath) ##
Once we have the response body we can then use the [JsonPath](http://static.javadoc.io/io.restassured/json-path/3.0.6/io/restassured/path/json/JsonPath.html) to get data from the response body:

```java
int lottoId = from(json).getInt("lotto.lottoId");
List<Integer> winnerIds = from(json).get("lotto.winners.winnerId");
```

Or a bit more efficiently:
```java
JsonPath jsonPath = new JsonPath(json).setRoot("lotto");
int lottoId = jsonPath.getInt("lottoId");
List<Integer> winnerIds = jsonPath.get("winners.winnderId");
```

Note that you can use `JsonPath` standalone without depending on REST Assured, see [getting started guide](GettingStarted) for more info on this.

### JsonPath Configuration ###
You can configure object de-serializers etc for JsonPath by configuring it, for example:
```java
JsonPath jsonPath = new JsonPath(SOME_JSON).using(new JsonPathConfig("UTF-8"));
```

It's also possible to configure JsonPath statically so that all instances of JsonPath will shared the same configuration:

```java
JsonPath.config = new JsonPathConfig("UTF-8");
```

You can read more about JsonPath at [this blog](http://www.jayway.com/2013/04/12/whats-new-in-rest-assured-1-8/).

Note that the JsonPath implementation uses <a href='http://groovy-lang.org/processing-xml.html#_gpath'>Groovy's GPath</a> syntax and is not to be confused with Jayway's <a href='https://github.com/jayway/JsonPath'>JsonPath</a> implementation.

## XML (using XmlPath) ##
You also have the corresponding functionality for XML using  [XmlPath](http://static.javadoc.io/io.restassured/xml-path/3.0.6/io/restassured/path/xml/XmlPath.html):

```java
String xml = post("/greetXML?firstName=John&lastName=Doe").andReturn().asString();
// Now use XmlPath to get the first and last name
String firstName = from(xml).get("greeting.firstName");
String lastName = from(xml).get("greeting.firstName");

// or a bit more efficiently:
XmlPath xmlPath = new XmlPath(xml).setRoot("greeting");
String firstName = xmlPath.get("firstName");
String lastName = xmlPath.get("lastName");
```

Note that you can use `XmlPath` standalone without depending on REST Assured, see [getting started guide](GettingStarted) for more info on this.

### XmlPath Configuration ###
You can configure object de-serializers and charset for XmlPath by configuring it, for example:
```java
XmlPath xmlPath = new XmlPath(SOME_XML).using(new XmlPathConfig("UTF-8"));
```

It's also possible to configure XmlPath statically so that all instances of XmlPath will shared the same configuration:

```java
XmlPath.config = new XmlPathConfig("UTF-8");
```

You can read more about XmlPath at [this blog](http://www.jayway.com/2013/04/12/whats-new-in-rest-assured-1-8/).

## Single path ##
If you only want to make a request and return a single path you can use a shortcut:
```java
int lottoId = get("/lotto").path("lotto.lottoid");
```

REST Assured will automatically determine whether to use JsonPath or XmlPath based on the content-type of the response. If no content-type is defined then REST Assured will try to look at the [default parser](#default-parser) if defined. You can also manually decide which path instance to use, e.g.

```java
String firstName = post("/greetXML?firstName=John&lastName=Doe").andReturn().xmlPath().getString("firstName");
```

Options are `xmlPath`, `jsonPath` and `htmlPath`.

## Headers, cookies, status etc ##

You can also get headers, cookies, status line and status code:
```java
Response response = get("/lotto");

// Get all headers
Headers allHeaders = response.getHeaders();
// Get a single header value:
String headerName = response.getHeader("headerName");

// Get all cookies as simple name-value pairs
Map<String, String> allCookies = response.getCookies();
// Get a single cookie value:
String cookieValue = response.getCookie("cookieName");

// Get status line
String statusLine = response.getStatusLine();
// Get status code
int statusCode = response.getStatusCode();
```

## Multi-value headers and cookies ##
A header and a cookie can contain several values for the same name.

### Multi-value headers ###
To get all values for a header you need to first get the [Headers](http://static.javadoc.io/io.rest-assured/rest-assured/3.0.6/io/restassured/http/Headers.html) object from the [Response](http://static.javadoc.io/io.rest-assured/rest-assured/3.0.6/io/restassured/response/Response.html) object. From the `Headers` instance you can get all values using the [Headers.getValues(<header name>)](http://static.javadoc.io/io.rest-assured/rest-assured/3.0.6/io/restassured/http/Headers.html#getValues(java.lang.String)) method which returns a `List` with all header values.

### Multi-value cookies ###
To get all values for a cookie you need to first get the [Cookies](http://static.javadoc.io/io.rest-assured/rest-assured/3.0.6/io/restassured/http/Cookies.html) object from the [Response](http://static.javadoc.io/io.rest-assured/rest-assured/3.0.6/io/restassured/response/Response.html) object. From the `Cookies` instance you can get all values using the [Cookies.getValues(<cookie name>)](http://static.javadoc.io/io.rest-assured/rest-assured/3.0.6/io/restassured/http/Cookies.html#getValues(java.lang.String)) method which returns a `List` with all cookie values.

## Detailed Cookies ##
If you need to get e.g. the comment, path or expiry date etc from a cookie you need get a [detailed cookie](http://static.javadoc.io/io.rest-assured/rest-assured/3.0.6/io/restassured/http/Cookie.html) from REST Assured. To do this you can use the [Response.getDetailedCookie(java.lang.String)](http://static.javadoc.io/io.rest-assured/rest-assured/3.0.6/io/restassured/response/ResponseOptions.html#getDetailedCookie-java.lang.String-) method. The detailed cookie then contains all attributes from the cookie.

You can also get all detailed response [cookies](http://static.javadoc.io/io.rest-assured/rest-assured/3.0.6/io/restassured/http/Cookies.html) using the [Response.getDetailedCookies()](http://static.javadoc.io/io.rest-assured/rest-assured/3.0.6/io/restassured/response/ResponseOptions.html#getDetailedCookies--) method.

# Specifying Request Data #

Besides specifying request parameters you can also specify headers, cookies, body and content type.

## Invoking HTTP resources ##

You typically perform a request by calling any of the "HTTP methods" in the [request specification](http://static.javadoc.io/io.rest-assured/rest-assured/3.0.6/io/restassured/specification/RequestSpecification.html). For example:

```java
when().get("/x"). ..;
```

Where `get` is the HTTP request method.

As of REST Assured 3.0.0 you can use any HTTP verb with your request by making use of the [](http://static.javadoc.io/io.rest-assured/rest-assured/3.0.6/io/restassured/specification/RequestSpecification.html#request-java.lang.String-java.lang.String-) method.

```java
when().
       request("CONNECT", "/somewhere").
then().
       statusCode(200);
```

This will send a "connect" request to the server.

## Parameters ##
Normally you specify parameters like this:

```java
given().
       param("param1", "value1").
       param("param2", "value2").
when().
       get("/something");
```

REST Assured will automatically try to determine which parameter type (i.e. query or form parameter) based on the HTTP method. In case of GET query parameters will automatically be used and in case of POST form parameters will be used. In some cases it's however important to separate between form and query parameters in a PUT or POST. You can then do like this:

```java
given().
       formParam("formParamName", "value1").
       queryParam("queryParamName", "value2").
when().
       post("/something");
```

Parameters can also be set directly on the url:
```java
..when().get("/name?firstName=John&lastName=Doe");
```

For multi-part parameters please refer to the [Multi-part form data](#multi-part-form-data) section.

### Multi-value parameter ###
Multi-value parameters are parameters with more then one value per parameter name (i.e. a list of values per name). You can specify these either by using var-args:

```java
given().param("myList", "value1", "value2"). .. 
```

or using a list:

```java
List<String> values = new ArrayList<String>();
values.add("value1");
values.add("value2");

given().param("myList", values). .. 
```

### No-value parameter ###
You can also specify a query, request or form parameter without a value at all:
```java
given().param("paramName"). ..
```

### Path parameters ###
You can also specify so called path parameters in your request, e.g.
```java
post("/reserve/{hotelId}/{roomNumber}", "My Hotel", 23);
```

These kinds of path parameters are referred to "unnamed path parameters" in REST Assured since they are index based (`hotelId` will be equal to "My Hotel" since it's the first placeholder).

You can also use named path parameters:
```java
given().
        pathParam("hotelId", "My Hotel").
        pathParam("roomNumber", 23).
when(). 
        post("/reserve/{hotelId}/{roomNumber}").
then().
         ..
```

Path parameters makes it easier to read the request path as well as enabling the request path to easily be re-usable in many tests with different parameter values.

As of version 2.8.0 you can mix unnamed and named path parameters:

```java
given().
        pathParam("hotelId", "My Hotel").        
when(). 
        post("/reserve/{hotelId}/{roomNumber}", 23).
then().
         ..
```

Here `roomNumber` will be replaced with `23`.

Note that specifying too few or too many parameters will result in an error message. For advanced use cases you can add, change, remove (even redundant path parameters) from a [filter](#filters).

## Cookies ##
In its simplest form you specify cookies like this:
```java
given().cookie("username", "John").when().get("/cookie").then().body(equalTo("username"));
```

You can also specify a multi-value cookie like this:
```java
given().cookie("cookieName", "value1", "value2"). ..
```
This will create _two_ cookies, `cookieName=value1` and `cookieName=value2`.

You can also specify a detailed cookie using:
```java
Cookie someCookie = new Cookie.Builder("some_cookie", "some_value").setSecured(true).setComment("some comment").build();
given().cookie(someCookie).when().get("/cookie").then().assertThat().body(equalTo("x"));
```

or several detailed cookies at the same time:
```java
Cookie cookie1 = Cookie.Builder("username", "John").setComment("comment 1").build();
Cookie cookie2 = Cookie.Builder("token", 1234).setComment("comment 2").build();
Cookies cookies = new Cookies(cookie1, cookie2);
given().cookies(cookies).when().get("/cookie").then().body(equalTo("username, token"));
```

## Headers ##
```java
given().header("MyHeader", "Something").and(). ..
given().headers("MyHeader", "Something", "MyOtherHeader", "SomethingElse").and(). ..
```

You can also specify a multi-value headers like this:
```java
given().header("headerName", "value1", "value2"). ..
```
This will create _two_ headers, `headerName: value1` and `headerName: value2`.

#### Header Merging/Overwriting ####

By default headers are merged. So for example if you do like this:

```java
given().header("x", "1").header("x", "2"). ..
```

The request will contain two headers, "x: 1" and "x: 2". You can change in this on a per header basis in the [HeaderConfig](http://static.javadoc.io/io.rest-assured/rest-assured/3.0.6/io/restassured/config/HeaderConfig.html). For example:

```java
given().
        config(RestAssuredConfig.config().headerConfig(headerConfig().overwriteHeadersWithName("x"))).
        header("x", "1").
        header("x", "2").
when().
        get("/something").
...
```

This means that only one header, "x: 2", is sent to server.

## Content Type ##
```java
given().contentType(ContentType.TEXT). ..
given().contentType("application/json"). ..
```


## Request Body ##
```java
given().body("some body"). .. // Works for POST, PUT and DELETE requests
given().request().body("some body"). .. // More explicit (optional)
```

```java
given().body(new byte[]{42}). .. // Works for POST, PUT and DELETE
given().request().body(new byte[]{42}). .. // More explicit (optional)
```

You can also serialize a Java object to JSON or XML. Click [here](#serialization) for details.

# Verifying Response Data #
You can also verify status code, status line, cookies, headers, content type and body.
## Response Body ##
See Usage examples, e.g. [JSON](#example-1---json) or [XML](#example-2---xml).

You can also map a response body to a Java Object, click [here](#deserialization) for details.

## Cookies ##
```java
get("/x").then().assertThat().cookie("cookieName", "cookieValue"). ..
get("/x").then().assertThat().cookies("cookieName1", "cookieValue1", "cookieName2", "cookieValue2"). ..
get("/x").then().assertThat().cookies("cookieName1", "cookieValue1", "cookieName2", containsString("Value2")). ..
```

## Status ##
```java
get("/x").then().assertThat().statusCode(200). ..
get("/x").then().assertThat().statusLine("something"). ..
get("/x").then().assertThat().statusLine(containsString("some")). ..
```


## Headers ##

```java
get("/x").then().assertThat().header("headerName", "headerValue"). ..
get("/x").then().assertThat().headers("headerName1", "headerValue1", "headerName2", "headerValue2"). ..
get("/x").then().assertThat().headers("headerName1", "headerValue1", "headerName2", containsString("Value2")). ..
```

It's also possible to use a mapping function when validating headers. For example let's say you want to validate that the `Content-Length` header is less than 1000. You can then use a mapping function to first convert the header value to an int and then use an `Integer` before validating it with a Hamcrest matcher:

```java
get("/something").then().assertThat().header("Content-Length", Integer::parseInt, lessThan(1000));
```

## Content-Type ##
```java
get("/x").then().assertThat().contentType(ContentType.JSON). ..
```


## Full body/content matching ##

```java
get("/x").then().assertThat().body(equalTo("something")). ..
```

## Use the response to verify other parts of the response ##
You can use data from the response to verify another part of the response. For example consider the following JSON document returned from service x:
```javascript
{ "userId" : "some-id", "href" : "http://localhost:8080/some-id" }
```
You may notice that the "href" attribute ends with the value of the "userId" attribute. If we want to verify this we can implement a `io.restassured.matcher.ResponseAwareMatcher` and use it like this:
```java
get("/x").then().body("href", new ResponseAwareMatcher<Response>() {
                                  public Matcher<?> matcher(Response response) {
                                          return equalTo("http://localhost:8080/" + response.path("userId"));
                                  }
                       });
```

If you're using Java 8 you can use a lambda expression instead:

```java
get("/x").then().body("href", response -> equalTo("http://localhost:8080/" + response.path("userId"));
```

There are some predefined matchers that you can use defined in the `io.restassured.matcher.RestAssuredMatchers` (or `io.restassured.module.mockmvc.matcher.RestAssuredMockMvcMatchers` if using the spring-mock-mvc module). For example:
```java
get("/x").then().body("href", endsWithPath("userId"));
```
`ResponseAwareMatchers` can also be composed, either with another `ResponseAwareMatcher` or with a Hamcrest Matcher. For example:
```java
get("/x").then().body("href", and(startsWith("http:/localhost:8080/"), endsWithPath("userId")));
```

The `and` method is statically imported from `io.restassured.matcher.ResponseAwareMatcherComposer`.

## Measuring Response Time ##

As of version 2.8.0 REST Assured has support measuring response time. For example:

```java
long timeInMs = get("/lotto").time()
```

or using a specific time unit:

```java
long timeInSeconds = get("/lotto").timeIn(SECONDS);
```

where `SECONDS` is just a standard `TimeUnit`. You can also validate it using the validation DSL:

```java
when().
      get("/lotto").
then().
      time(lessThan(2000L)); // Milliseconds
```

or

```java
when().
      get("/lotto").
then().
      time(lessThan(2L), SECONDS);
```

Please note that response time measurement should be performed when the JVM is hot! (i.e. running a response time measurement when only running a single test will yield erroneous results). Also note that you can only vaguely regard these measurments to correlate with the server request processing time (since the response time will include the HTTP round trip and REST Assured processing time among other things).

# Authentication #
REST assured also supports several authentication schemes, for example OAuth, digest, certificate, form and preemptive basic authentication. You can either set authentication for each request:
```java
given().auth().basic("username", "password"). ..
```

but you can also define authentication for all requests:
```java
RestAssured.authentication = basic("username", "password");
```
or you can use a [specification](#specification-re-use).

## Basic Authentication ##

There are two types of basic authentication, preemptive and "challenged basic authentication".

### Preemptive Basic Authentication ###
This will send the basic authentication credential even before the server gives an unauthorized response in certain situations, thus reducing the overhead of making an additional connection. This is typically what you want to use in most situations unless you're testing the servers ability to challenge. Example:

```java
given().auth().preemptive().basic("username", "password").when().get("/secured/hello").then().statusCode(200);
```

### Challenged Basic Authentication ###
When using "challenged basic authentication" REST Assured will not supply the credentials unless the server has explicitly asked for it. This means that REST Assured will make an additional request to the server in order to be challenged and then follow up with the same request once more but this time setting the basic credentials in the header.

```java
given().auth().basic("username", "password").when().get("/secured/hello").then().statusCode(200);
```

## Digest Authentication ##
Currently only "challenged digest authentication" is supported. Example:

```java
given().auth().digest("username", "password").when().get("/secured"). ..
```

## Form Authentication ##

[Form authentication](https://en.wikipedia.org/wiki/Form-based_authentication) is very popular on the internet. It's typically associated with a user filling out his credentials (username and password) on a webpage and then pressing a login button of some sort. A very simple HTML page that provide the basis for form authentication may look like this:
```html
<html>
  <head>
    <title>Login</title>
  </head>

  <body>
    <form action="j_spring_security_check" method="POST">
      <table>
        <tr><td>User:&nbsp;</td><td><input type='text' name='j_username'></td></tr>
        <tr><td>Password:</td><td><input type='password' name='j_password'></td></tr>
          <tr><td colspan='2'><input name="submit" type="submit"/></td></tr>
       </table>
        </form>
      </body>
 </html>
```

I.e. the server expects the user to fill-out the "j_username" and "j_password" input fields and then press "submit" to login. With REST Assured you can test a service protected by form authentication like this:

```java
given().
        auth().form("John", "Doe").
when().
        get("/formAuth");
then().
        statusCode(200);
```

While this may work it's not optimal. What happens when form authentication is used like this in REST Assured an additional request have to made to the server in order to retrieve the webpage with the login details. REST Assured will then try to parse this page and look for two input fields (with username and password) as well as the form action URI. This may work or fail depending on the complexity of the webpage. A better option is to supply the these details when setting up the form authentication. In this case one could do:

```java
given().
        auth().form("John", "Doe", new FormAuthConfig("/j_spring_security_check", "j_username", "j_password")).
when().
        get("/formAuth");
then().
        statusCode(200);
```

This way REST Assured doesn't need to make an additional request and parse the webpage. There's also a predefined FormAuthConfig called `springSecurity` that you can use if you're using the default Spring Security properties:

```java
given().
        auth().form("John", "Doe", FormAuthConfig.springSecurity()).
when().
        get("/formAuth");
then().
        statusCode(200);
```

### CSRF ###
Today it's common for the server to supply a [CSRF](https://en.wikipedia.org/wiki/Cross-site_request_forgery) token with the response in order to avoid these kinds of attacks. REST Assured has support for automatically parsing and supplying the CSRF token to the server. In order for this to work REST Assured *must* make an additional request and parse (parts) of the website.

You can enable CSRF support by doing the following:

```java
given().
        auth().form("John", "Doe", formAuthConfig().withAutoDetectionOfCsrf()).
when().
        get("/formAuth");
then().
        statusCode(200);
```

Now REST Assured will automatically try to detect if the webpage contains a CSRF token. In order to assist REST Assured and make the parsing more robust it's possible to supply the CSRF field name (here we imagine that we're using Spring Security default values and thus can make use of the predefined `springSecurity` FormAuthConfig):

```java
given().
        auth().form("John", "Doe", springSecurity().withCsrfFieldName("_csrf")).
when().
        get("/formAuth");
then().
        statusCode(200);
```

We've now told REST Assured to search for the CSRF field name called "_csrf" (which is it both faster and less prone to error).

By default the CSRF value is sent as a form parameter with the request but you can configure to send it as a header instead if that's required:

```java
given().
        auth().form("John", "Doe", springSecurity().withCsrfFieldName("_csrf").sendCsrfTokenAsHeader()).
when().
        get("/formAuth");
then().
        statusCode(200);
```

## OAuth ##

In order to use OAuth 1 and OAuth 2 (for query parameter signing) you need to add [Scribe](https://github.com/fernandezpablo85/scribe-java) to your classpath (if you're using version 2.1.0 or older of REST Assured then please refer to the [legacy](Usage_Legacy#OAuth) documentation). In Maven you can simply add the following dependency:
```xml
<dependency>
            <groupId>com.github.scribejava</groupId>
            <artifactId>scribejava-apis</artifactId>
            <version>2.5.3</version>
            <scope>test</scope>
</dependency>
```

If you're not using Maven [download](https://github.com/fernandezpablo85/scribe-java/releases) a Scribe release manually and put it in your classpath.

### OAuth 1 ###
OAuth 1 requires [Scribe](#oauth) in the classpath. To use auth 1 authentication you can do:
```java
given().auth().oauth(..). ..
```

### OAuth 2 ###
Since version `2.5.0` you can use OAuth 2 authentication without depending on [Scribe](#oauth):
```java
given().auth().oauth2(accessToken). ..
```
This will put the OAuth2 `accessToken` in a header. To be more explicit you can also do:
```java
given().auth().preemptive().oauth2(accessToken). ..
```
There reason why `given().auth().oauth2(..)` still exists is for backward compatibility (they do the same thing). If you need to provide the OAuth2 token in a query parameter you currently need [Scribe](#oauth) in the classpath. Then you can do like this:
```java
given().auth().oauth2(accessToken, OAuthSignature.QUERY_STRING). ..
```

## Custom Authentication ##
Rest Assured allows you to create custom authentication providers. You do this by implementing the `io.restassured.spi.AuthFilter` interface (preferably) and apply it as a [filter](#filters). For example let's say that your security consists of adding together two headers together in a new header called "AUTH" (this is of course not secure). Then you can do that like this (Java 8 syntax):
```java
given().
        filter((requestSpec, responseSpec, ctx) -> {
            String header1 = requestSpec.getHeaders().getValue("header1");
            String header2 = requestSpec.getHeaders().getValue("header2");
            requestSpec.header("AUTH", header1 + header2);
            return ctx.next(requestSpec, responseSpec);
        }).
when().
        get("/customAuth").
then().
  statusCode(200);
```

The reason why you want to use a `AuthFilter` and not `Filter` is that `AuthFilters` are automatically removed when doing `given().auth().none(). ..`.

# Multi-part form data #
When sending larger amount of data to the server it's common to use the multipart form data technique. Rest Assured provide methods called `multiPart` that allows you to specify a file, byte-array, input stream or text to upload. In its simplest form you can upload a file like this:

```java
given().
        multiPart(new File("/path/to/file")).
when().
        post("/upload");
```

It will assume a control name called "file". In HTML the control name is the attribute name of the input tag. To clarify let's look at the following HTML form:

```html
<form id="uploadForm" action="/upload" method="post" enctype="multipart/form-data">
        <input type="file" name="file" size="40">
        <input type=submit value="Upload!">
</form>
```

The control name in this case is the name of the input tag with name "file". If you have a different control name then you need to specify it:

```java
given().
        multiPart("controlName", new File("/path/to/file")).
when().
        post("/upload");
```

It's also possible to supply multiple "multi-parts" entities in the same request:

```java
byte[] someData = ..
given().
        multiPart("controlName1", new File("/path/to/file")).
        multiPart("controlName2", "my_file_name.txt", someData).
        multiPart("controlName3", someJavaObject, "application/json").
when().
        post("/upload");
```

For more advanced use cases you can make use of the [MultiPartSpecBuilder](http://static.javadoc.io/io.rest-assured/rest-assured/3.0.6/io/restassured/builder/MultiPartSpecBuilder.html). For example:

```java
Greeting greeting = new Greeting();
greeting.setFirstName("John");
greeting.setLastName("Doe");

given().
        multiPart(new MultiPartSpecBuilder(greeting, ObjectMapperType.JACKSON_2)
                .fileName("greeting.json")
                .controlName("text")
                .mimeType("application/vnd.custom+json").build()).
when().
        post("/multipart/json").
then().
        statusCode(200);
```

You can specify, among other things, the default `control name` and filename using the [MultiPartConfig](http://static.javadoc.io/io.rest-assured/rest-assured/3.0.6/io/restassured/config/MultiPartConfig.html). For example:

```java
given().config(config().multiPartConfig(multiPartConfig().defaultControlName("something-else"))). ..
```

This will configure the default control name to be "something-else" instead of "file".

For additional info refer to [this](http://blog.jayway.com/2011/09/15/multipart-form-data-file-uploading-made-simple-with-rest-assured/) blog post.

# Object Mapping #
REST Assured supports mapping Java objects to and from JSON and XML. For JSON you need to have either Jackson or Gson in the classpath and for XML you need JAXB.

## Serialization ##
Let's say we have the following Java object:

```java
public class Message {
    private String message;

    public String getMessage() {
        return message;
    }

    public void setMessage(String message) {
        this.message = message;
    }
}
```

and you want to serialize this object to JSON and send it with the request. There are several ways to do this, e.g:

### Content-Type based Serialization ###

```java
Message message = new Message();
message.setMessage("My messagee");
given().
       contentType("application/json").
       body(message).
when().
      post("/message");
```

In this example REST Assured will serialize the object to JSON since the request content-type is set to "application/json". It will first try to use Jackson if found in classpath and if not Gson will be used. If you change the content-type to "application/xml" REST Assured will serialize to XML using JAXB. If no content-type is defined REST Assured will try to serialize in the following order:

  1. JSON using Jackson 2 (Faster Jackson (databind))
  1. JSON using Jackson (databind)
  1. JSON using Gson
  1. XML using JAXB

REST Assured also respects the charset of the content-type. E.g.

```java
Message message = new Message();
message.setMessage("My messagee");
given().
       contentType("application/json; charset=UTF-16").
       body(message).
when().
      post("/message");
```

You can also serialize the `Message` instance as a form parameter:
```java
Message message = new Message();
message.setMessage("My messagee");
given().
       contentType("application/json; charset=UTF-16").
       formParam("param1", message).
when().
      post("/message");
```


The message object will be serialized to JSON using Jackson (databind) (if present) or Gson (if present) with UTF-16 encoding.

### Create JSON from a HashMap ###

You can also create a JSON document by supplying a Map to REST Assured.
```java
Map<String, Object>  jsonAsMap = new HashMap<>();
jsonAsMap.put("firstName", "John");
jsonAsMap.put("lastName", "Doe");

given().
        contentType(JSON).
        body(jsonAsMap).
when().
        post("/somewhere").
then().
        statusCode(200);
```

This will provide a JSON payload as:

```javascript
{ "firstName" : "John", "lastName" : "Doe" }
```

### Using an Explicit Serializer ###
If you have multiple object mappers in the classpath at the same time or don't care about setting the content-type you can specify a serializer explicity. E.g.

```java
Message message = new Message();
message.setMessage("My messagee");
given().
       body(message, ObjectMapperType.JAXB).
when().
      post("/message");
```

In this example the Message object will be serialized to XML using JAXB.

## Deserialization ##
Again let's say we have the following Java object:

```java
public class Message {
    private String message;

    public String getMessage() {
        return message;
    }

    public void setMessage(String message) {
        this.message = message;
    }
}
```

and we want the response body to be deserialized into a Message object.

### Content-Type based Deserialization ###
Let's assume then that the server returns a JSON body like this:
```javascript
{"message":"My message"}
```

To deserialize this to a Message object we simply to like this:
```java
Message message = get("/message").as(Message.class);
```

For this to work the response content-type must be "application/json" (or something that contains "json"). If the server instead returned

```xml
<?xml version="1.0" encoding="UTF-8" standalone="yes"?>
<message>
      <message>My message</message>
</message>
```

and a content-type of "application/xml" you wouldn't have to change the code at all:
```java
Message message = get("/message").as(Message.class);
```

#### Custom Content-Type Deserialization ####
If the server returns a custom content-type, let's say "application/something", and you still want to use the object mapping in REST Assured there are a couple of different ways to go about. You can either use the [explicit](http://code.google.com/p/rest-assured/wiki/Usage#Using_an_Explicit_Deserializer) approach or register a parser for the custom content-type:

```java
Message message = expect().parser("application/something", Parser.XML).when().get("/message").as(Message.class);
```

or

```java
Message message = expect().defaultParser(Parser.XML).when().get("/message").as(Message.class);
```

You can also register a default or custom parser [statically](#default-values) or using [specifications](#specification-re-use).

### Using an Explicit Deserializer ###
If you have multiple object mappers in the classpath at the same time  or don't care about the response content-type you can specify a deserializer explicitly. E.g.

```java
Message message = get("/message").as(Message.class, ObjectMapperType.GSON);
```

## Configuration ##
You can configure the pre-defined object mappers by using a [ObjectMapperConfig](http://static.javadoc.io/io.rest-assured/rest-assured/3.0.6/io/restassured/config/ObjectMapperConfig.html) and pass it to [detailed configuration](#detailed-configuration). For example to change GSON to use lower case with underscores as field naming policy you can do like this:

```java
RestAssured.config = RestAssuredConfig.config().objectMapperConfig(objectMapperConfig().gsonObjectMapperFactory(
                new GsonObjectMapperFactory() {
                    public Gson create(Class cls, String charset) {
                        return new GsonBuilder().setFieldNamingPolicy(LOWER_CASE_WITH_UNDERSCORES).create();
                    }
                }
        ));
```

There are pre-defined object mapper factories for GSON, JAXB, Jackson and Faster Jackson.

## Custom ##
By default REST Assured will scan the classpath to find various object mappers. If you want to integrate an object mapper that is not supported by default or if you've rolled your own you can implement the
[io.restassured.mapper.ObjectMapper](http://static.javadoc.io/io.rest-assured/rest-assured/3.0.6/io/restassured/mapper/ObjectMapper.html) interface. You tell REST Assured to use your object mapper either by passing it as a second parameter to the body:

```java
given().body(myJavaObject, myObjectMapper).when().post("..")
```

or you can define it statically once and for all:
```java
RestAssured.config = RestAssuredConfig.config().objectMapperConfig(new ObjectMapperConfig(myObjectMapper));
```

For an example see [here](https://github.com/rest-assured/rest-assured/blob/master/examples/rest-assured-itest-java/src/test/java/io/restassured/itest/java/CustomObjectMappingITest.java).

# Custom parsers #
REST Assured providers predefined parsers for e.g. HTML, XML and JSON. But you can parse other kinds of content by registering a predefined parser for unsupported content-types by using:
```java
RestAssured.registerParser(<content-type>, <parser>);
```
E.g. to register that mime-type 'application/vnd.uoml+xml' should be parsed using the XML parser do:
```java
RestAssured.registerParser("application/vnd.uoml+xml", Parser.XML);
```
You can also unregister a parser using:
```java
RestAssured.unregisterParser("application/vnd.uoml+xml");
```

Parsers can also be specified per "request":
```java
get(..).then().using().parser("application/vnd.uoml+xml", Parser.XML). ..;
```

and using a [response specification](sSpecification-re-use).

# Default parser #
Sometimes it's useful to specify a default parser, e.g. if the response doesn't contain a content-type at all:

```java
RestAssured.defaultParser = Parser.JSON;
```

You can also specify a default parser for a single request:
```java
get("/x").then().using().defaultParser(Parser.JSON). ..
```

or using a [response specification](#specification-re-use).

# Default values #
By default REST assured assumes host localhost and port 8080 when doing a request. If you want a different port you can do:
```java
given().port(80). ..
```
or simply:
```java
..when().get("http://myhost.org:80/doSomething");
```
You can also change the default base URI, base path, port and authentication scheme for all subsequent requests:
```java
RestAssured.baseURI = "http://myhost.org";
RestAssured.port = 80;
RestAssured.basePath = "/resource";
RestAssured.authentication = basic("username", "password");
RestAssured.rootPath = "x.y.z";
```
This means that a request like e.g. `get("/hello")` goes to: http://myhost.org:80/resource/hello with basic authentication credentials "username" and "password". See [rootPath](http://code.google.com/p/rest-assured/wiki/Usage#Root_path) for more info about setting the root paths. Other default values you can specify are:

```java
RestAssured.filters(..); // List of default filters
RestAssured.requestSpecification = .. // Default request specification
RestAssured.responseSpecification = .. // Default response specification
RestAssured.urlEncodingEnabled = .. // Specify if Rest Assured should URL encoding the parameters
RestAssured.defaultParser = .. // Specify a default parser for response bodies if no registered parser can handle data of the response content-type
RestAssured.registerParser(..) // Specify a parser for the given content-type
RestAssured.unregisterParser(..) // Unregister a parser for the given content-type
```

You can reset to the standard baseURI (localhost), basePath (empty), standard port (8080), standard root path (""), default authentication scheme (none) and url encoding enabled (true) using:
```java
RestAssured.reset();
```

# Specification Re-use #
Instead of having to duplicate response expectations and/or request parameters for different tests you can re-use an entire specification. To do this you define a specification using either the [RequestSpecBuilder](http://static.javadoc.io/io.rest-assured/rest-assured/3.0.6/io/restassured/builder/RequestSpecBuilder.html) or [ResponseSpecBuilder](http://static.javadoc.io/io.rest-assured/rest-assured/3.0.6/io/restassured/builder/ResponseSpecBuilder.html).

E.g. let's say you want to make sure that the expected status code is 200 and that the size of the JSON array "x.y" has size 2 in several tests you can define a ResponseSpecBuilder like this:

```java
ResponseSpecBuilder builder = new ResponseSpecBuilder();
builder.expectStatusCode(200);
builder.expectBody("x.y.size()", is(2));
ResponseSpecification responseSpec = builder.build();

// Now you can re-use the "responseSpec" in many different tests:
when().
       get("/something").
then().
       spec(responseSpec).
       body("x.y.z", equalTo("something"));
```

In this example the data defined in "responseSpec" is merged with the additional body expectation and all expectations must be fulfilled in order for the test to pass.

You can do the same thing if you need to re-use request data in different tests. E.g.
```java
RequestSpecBuilder builder = new RequestSpecBuilder();
builder.addParam("parameter1", "parameterValue");
builder.addHeader("header1", "headerValue");
RequestSpecification requestSpec = builder.build();
  
given().
        spec(requestSpec).
        param("parameter2", "paramValue").
when().
        get("/something").
then().
        body("x.y.z", equalTo("something"));        
```

Here the request's data is merged with the data in the "requestSpec" so the request will contain two parameters ("parameter1" and "parameter2") and one header ("header1").

# Filters #
A filter allows you to inspect and alter a request before it's actually committed and also inspect and [alter](#response-builder) the response before it's returned to the expectations. You can regard it as an "around advice" in AOP terms. Filters can be used to implement custom authentication schemes, session management, logging etc. To create a filter you need to implement the [io.restassured.filter.Filter](http://static.javadoc.io/io.rest-assured/rest-assured/3.0.6/io/restassured/filter/Filter.html) interface. To use a filter you can do:

```java
given().filter(new MyFilter()). ..
```

There are a couple of filters provided by REST Assured that are ready to use:
  1. `io.restassured.filter.log.RequestLoggingFilter`: A filter that'll print the request specification details.
  1. `io.restassured.filter.log.ResponseLoggingFilter`: A filter that'll print the response details if the response matches a given status code.
  1. `io.restassured.filter.log.ErrorLoggingFilter`: A filter that'll print the response body if an error occurred (status code is between 400 and 500).

### Ordered Filters

As of REST Assured 3.0.2 you can implement the [io.restassured.filter.OrderedFilter](http://static.javadoc.io/io.rest-assured/rest-assured/3.0.6/io/restassured/filter/OrderedFilter.html) interface if you need to control the filter ordering. Here you implement the `getOrder` method to return an integer representing the precedence of the filter. A lower value gives higher precedence. The highest precedence you can define is `Integer.MIN_VALUE` and the lowest precedence is `Integer.MAX_VALUE`. Filters not implementing [io.restassured.filter.OrderedFilter](http://static.javadoc.io/io.rest-assured/rest-assured/3.0.6/io/restassured/filter/OrderedFilter.html) will have a default precedence of `1000`. Click [here](https://github.com/rest-assured/rest-assured/blob/master/examples/rest-assured-itest-java/src/test/java/io/restassured/itest/java/OrderedFilterITest.java) for some examples.

### Response Builder ###

If you need to change the [Response](http://static.javadoc.io/io.rest-assured/rest-assured/3.0.6/io/restassured/response/Response.html) from a filter you can use the [ResponseBuilder](http://static.javadoc.io/io.rest-assured/rest-assured/3.0.6/io/restassured/builder/ResponseBuilder.html) to create a new Response based on the original response. For example if you want to change the body of the original response to something else you can do:
```java
Response newResponse = new ResponseBuilder().clone(originalResponse).setBody("Something").build();
```

# Logging #
In many cases it can be useful to print the response and/or request details in order to help you create the correct expectations and send the correct requests. To do help you do this you can use one of the predefined [filters](#filters) supplied with REST Assured or you can use one of the shortcuts.

## Request Logging ##
Since version 1.5 REST Assured supports logging the _[request specification](http://static.javadoc.io/io.rest-assured/rest-assured/3.0.6/io/restassured/specification/RequestSpecification.html)_ before it's sent to the server using the [RequestLoggingFilter](http://static.javadoc.io/io.rest-assured/rest-assured/3.0.6/io/restassured/filter/log/RequestLoggingFilter.html). Note that the HTTP Builder and HTTP Client may add additional headers then what's printed in the log. The filter will _only_ log details specified in the request specification. I.e. you can NOT regard the details logged by the [RequestLoggingFilter](http://static.javadoc.io/io.rest-assured/rest-assured/3.0.6/io/restassured/filter/log/RequestLoggingFilter.html) to be what's actually sent to the server. Also subsequent filters may alter the request _after_ the logging has taken place. If you need to log what's _actually_ sent on the wire refer to the [HTTP Client logging docs](http://hc.apache.org/httpcomponents-client-ga/logging.html) or use an external tool such [Wireshark](http://www.wireshark.org/). Examples:

```java
given().log().all(). .. // Log all request specification details including parameters, headers and body
given().log().params(). .. // Log only the parameters of the request
given().log().body(). .. // Log only the request body
given().log().headers(). .. // Log only the request headers
given().log().cookies(). .. // Log only the request cookies
given().log().method(). .. // Log only the request method
given().log().path(). .. // Log only the request path
```

## Response Logging ##
If you want to print the response body regardless of the status code you can do:

```java
get("/x").then().log().body() ..
```

This will print the response body regardless if an error occurred. If you're only interested in printing the response body if an error occur then you can use:

```java
get("/x").then().log().ifError(). .. 
```

You can also log all details in the response including status line, headers and cookies:

```java
get("/x").then().log().all(). .. 
```

as well as only status line, headers or cookies:
```java
get("/x").then().log().statusLine(). .. // Only log the status line
get("/x").then().log().headers(). .. // Only log the response headers
get("/x").then().log().cookies(). .. // Only log the response cookies
```

You can also configure to log the response only if the status code matches some value:
```java
get("/x").then().log().ifStatusCodeIsEqualTo(302). .. // Only log if the status code is equal to 302
get("/x").then().log().ifStatusCodeMatches(matcher). .. // Only log if the status code matches the supplied Hamcrest matcher
```

## Log if validation fails ##

Since REST Assured 2.3.1 you can log the request or response only if the validation fails. To log the request do:
```java
given().log().ifValidationFails(). ..
```

To log the response do:
```java
.. .then().log().ifValidationFails(). ..
```

It's also possible to enable this for both the request and the response at the same time using the [LogConfig](http://static.javadoc.io/io.rest-assured/rest-assured/3.0.6/io/restassured/config/LogConfig.html):

```java
given().config(RestAssured.config().logConfig(logConfig().enableLoggingOfRequestAndResponseIfValidationFails(HEADERS))). ..
```

This will log only the headers if validation fails.

There's also a shortcut for enabling logging of the request and response for all requests if validation fails:

```java
RestAssured.enableLoggingOfRequestAndResponseIfValidationFails();
```

# Root path #
To avoid duplicated paths in body expectations you can specify a root path. E.g. instead of writing:
```java
when().
         get("/something").
then().
         body("x.y.firstName", is(..)).
         body("x.y.lastName", is(..)).
         body("x.y.age", is(..)).
         body("x.y.gender", is(..));
```

you can use a root path and do:

```java
when().
        get("/something").
then().
         root("x.y"). // You can also use the "root" method
         body("firstName", is(..)).
         body("lastName", is(..)).
         body("age", is(..)).
         body("gender", is(..));
```
You can also set a default root path using:
```java
RestAssured.rootPath = "x.y";
```

In more advanced use cases it may also be useful to append additional root arguments to existing root arguments. To do this you can use the `appendRoot` method, for example:
```java
when().
         get("/jsonStore").
then().
         root("store.%s", withArgs("book")).
         body("category.size()", equalTo(4)).
         appendRoot("%s.%s", withArgs("author", "size()")).
         body(withNoArgs(), equalTo(4));
```

It's also possible to detach a root. For example:
```java
when().
         get("/jsonStore").
then().
         root("store.category").
         body("size()", equalTo(4)).
         detachRoot("category").
         body("size()", equalTo(1));
```

# Path arguments #
Path arguments are useful in situations where you have e.g. pre-defined variables that constitutes the path. For example
```java
String someSubPath = "else";
int index = 1;
get("/x").then().body("something.%s[%d]", withArgs(someSubPath, index), equalTo("some value")). ..
```

will expect that the body path "`something.else[0]`" is equal to "some value".

Another usage is if you have complex [root paths](http://code.google.com/p/rest-assured/wiki/Usage#Root_path) and don't wish to duplicate the path for small variations:
```java
when().
       get("/x").
then().
       root("filters.filterConfig[%d].filterConfigGroups.find { it.name == 'GroupName' }.includes").
       body(withArgs(0), hasItem("first")).
       body(withArgs(1), hasItem("second")).
       ..
```

The path arguments follows the standard [formatting syntax](http://download.oracle.com/javase/1,5.0/docs/api/java/util/Formatter.html#syntax) of Java.

Note that the `withArgs` method can be statically imported from the [io.restassured.RestAssured](http://static.javadoc.io/io.rest-assured/rest-assured/3.0.6/io/restassured/RestAssured.html) class.

Sometimes it's also useful to validate a body without any additional arguments when all arguments have already been specified in the root path. This is where `withNoArgs` come into play. For example:
```java
when().
         get("/jsonStore").
then().
         root("store.%s", withArgs("book")).
         body("category.size()", equalTo(4)).
         appendRoot("%s.%s", withArgs("author", "size()")).
         body(withNoArgs(), equalTo(4));
```

# Session support #
REST Assured provides a simplified way for managing sessions. You can define a session id value in the DSL:
```java
given().sessionId("1234"). .. 
```

This is actually just a short-cut for:
```java
given().cookie("JSESSIONID", "1234"). .. 
```

You can also specify a default `sessionId` that'll be supplied with all subsequent requests:
```java
RestAssured.sessionId = "1234";
```

By default the session id name is `JSESSIONID` but you can change it using the [SessionConfig](#session-config):
```java
RestAssured.config = RestAssured.config().sessionConfig(new SessionConfig().sessionIdName("phpsessionid"));
```

You can also specify a sessionId using the `RequestSpecBuilder` and reuse it in many tests:
```java
RequestSpecBuilder spec = new RequestSpecBuilder().setSessionId("value1").build();
   
// Make the first request with session id equal to value1
given().spec(spec). .. 
// Make the second request with session id equal to value1
given().spec(spec). .. 
```

It's also possible to get the session id from the response object:
```java
String sessionId = get("/something").sessionId();
```

## Session Filter ##
As of version 2.0.0 you can use a [session filter](http://static.javadoc.io/io.rest-assured/rest-assured/3.0.6/io/restassured/filter/session/SessionFilter.html) to automatically capture and apply the session, for example:
```java
SessionFilter sessionFilter = new SessionFilter();

given().
          auth().form("John", "Doe").
          filter(sessionFilter).
when().
          get("/formAuth").
then().
          statusCode(200);


given().
          filter(sessionFilter). // Reuse the same session filter instance to automatically apply the session id from the previous response
when().
          get("/x").
then().
          statusCode(200);
```

To get session id caught by the `SessionFilter` you can do like this:
```java
String sessionId = sessionFilter.getSessionId();
```

# SSL #
In most situations SSL should just work out of the box thanks to the excellent work of HTTP Builder and HTTP Client. There are however some cases where you'll run into trouble. You may for example run into a SSLPeerUnverifiedException if the server is using an invalid certificate. The easiest way to workaround this is to use "relaxed HTTPs validation". For example:
```java
given().relaxedHTTPSValidation().when().get("https://some_server.com"). .. 
```
You can also define this statically for all requests:
```java
RestAssured.useRelaxedHTTPSValidation();
```
or in a [request specification](#specification-re-use).

This will assume an SSLContext protocol of  `SSL`. To change to another protocol use an overloaded versionen of  `relaxedHTTPSValidation`. For example:

```java
given().relaxedHTTPSValidation("TLS").when().get("https://some_server.com"). .. 
```

You can also be more fine-grained and create Java keystore file and use it with REST Assured. It's not too difficult, first follow the guide [here](https://github.com/jgritman/httpbuilder/wiki/SSL) and then use the keystore in Rest Assured like this:

```java
given().keystore("/pathToJksInClassPath", <password>). .. 
```

or you can specify it for every request:

```java
RestAssured.keystore("/pathToJksInClassPath", <password>);
```

You can also define a keystore in a re-usable [specification](http://code.google.com/p/rest-assured/wiki/Usage#Specification_Re-use).

If you already loaded a keystore with a password you can use it as a truststore:
```java
RestAssured.trustStore(keystore);
```

You can find a working example [here](https://github.com/rest-assured/rest-assured/blob/master/examples/rest-assured-itest-java/src/test/java/io/restassured/itest/java/SSLTest.java).

For more advanced SSL Configuration refer to the [SSL Configuration](#ssl-config) section.

## SSL invalid hostname ##
If the certificate is specifying an invalid hostname you don't need to create and import a keystore. As of version `2.2.0` you can do:
```java
RestAssured.config = RestAssured.config().sslConfig(sslConfig().allowAllHostnames());
```

to allow all hostnames for all requests or:

```java
given().config(RestAssured.config().sslConfig(sslConfig().allowAllHostnames()). .. ;
```
for a single request.

Note that if you use "relaxed HTTPs validation" then `allowAllHostnames` is activated by default.

# URL Encoding #
Usually you don't have to think about URL encoding since Rest Assured provides this automatically out of the box. In some cases though it may be useful to turn URL Encoding off. One reason may be that you already the have some parameters encoded before you supply them to Rest Assured. To prevent double URL encoding you need to tell Rest Assured to disable it's URL encoding. E.g.

```java
String response = given().urlEncodingEnabled(false).get("https://jira.atlassian.com:443/rest/api/2.0.alpha1/search?jql=project%20=%20BAM%20AND%20issuetype%20=%20Bug").asString();
..
```

or

```java
RestAssured.baseURI = "https://jira.atlassian.com";
RestAssured.port = 443;
RestAssured.urlEncodingEnabled = false;
final String query = "project%20=%20BAM%20AND%20issuetype%20=%20Bug";
String response = get("/rest/api/2.0.alpha1/search?jql={q}", query);
..
```

# Proxy Configuration #
Starting from version 2.3.2 REST Assured has better support for proxies. For example if you have a proxy at localhost port 8888 you can do:
```java
given().proxy("localhost", 8888). .. 
```

Actually you don't even have to specify the hostname if the server is running on your local environment:

```java
given().proxy(8888). .. // Will assume localhost
```

To use HTTPS you need to supply a third parameter (scheme) or use the `io.restassured.specification.ProxySpecification`. For example:

```java
given().proxy(host("localhost").withScheme("https")). ..
```

where `host` is statically imported from `io.restassured.specification.ProxySpecification`.

Starting from version 2.7.0 you can also specify preemptive basic authentication for proxies. For example:
  
```
given().proxy(auth("username", "password")).when() ..
```

where `auth` is statically imported from [io.restassured.specification.ProxySpecification](http://static.javadoc.io/io.rest-assured/rest-assured/3.0.6/io/restassured/specification/ProxySpecification.html). You can of course also combine authentication with a different host:

```java
given().proxy(host("http://myhost.org").withAuth("username", "password")). ..
```

## Static Proxy Configuration ##

It's also possible to configure a proxy statically for all requests, for example:
```java
RestAssured.proxy("localhost", 8888);    
```

or:

```java
RestAssured.proxy = host("localhost").withPort(8888);
```

## Request Specification Proxy Configuration ##
You can also create a request specification and specify the proxy there:

```java
RequestSpecification specification = new RequestSpecBuilder().setProxy("localhost").build();
given().spec(specification). ..
```

# Detailed configuration #
Detailed configuration is provided by the [RestAssuredConfig](http://static.javadoc.io/io.rest-assured/rest-assured/3.0.6/io/restassured/config/RestAssuredConfig.html) instance with which you can configure the parameters of [HTTP Client](http://static.javadoc.io/io.rest-assured/rest-assured/3.0.6/io/restassured/config/HttpClientConfig.html) as well as [Redirect](http://static.javadoc.io/io.rest-assured/rest-assured/3.0.6/io/restassured/config/RedirectConfig.html), [Log](http://static.javadoc.io/io.rest-assured/rest-assured/3.0.6/io/restassured/config/LogConfig.html), [Encoder](http://static.javadoc.io/io.rest-assured/rest-assured/3.0.6/io/restassured/config/EncoderConfig.html), [Decoder](http://static.javadoc.io/io.rest-assured/rest-assured/3.0.6/io/restassured/config/DecoderConfig.html), [Session](http://static.javadoc.io/io.rest-assured/rest-assured/3.0.6/io/restassured/config/SessionConfig.html), [ObjectMapper](http://static.javadoc.io/io.rest-assured/rest-assured/3.0.6/io/restassured/config/ObjectMapperConfig.html), [Connection](http://static.javadoc.io/io.rest-assured/rest-assured/3.0.6/io/restassured/config/ConnectionConfig.html), [SSL](http://static.javadoc.io/io.rest-assured/rest-assured/3.0.6/io/restassured/config/SSLConfig.html) and [ParamConfig](http://static.javadoc.io/io.rest-assured/rest-assured/3.0.6/io/restassured/config/ParamConfig.html) settings. Examples:

For a specific request:
```java
given().config(RestAssured.config().redirect(redirectConfig().followRedirects(false))). ..
```
or using a RequestSpecBuilder:
```java
RequestSpecification spec = new RequestSpecBuilder().setConfig(RestAssured.config().redirect(redirectConfig().followRedirects(false))).build();
```
or for all requests:
```java
RestAssured.config = config().redirect(redirectConfig().followRedirects(true).and().maxRedirects(0));
```
`config()` and `newConfig()` can be statically imported from `io.restassured.config.RestAssuredConfig`.

## Encoder Config ##
With the [EncoderConfig](http://static.javadoc.io/io.rest-assured/rest-assured/3.0.6/io/restassured/config/EncoderConfig.html) you can specify the default content encoding charset (if it's not specified in the content-type header) and query parameter charset for all requests. If no content charset is specified then ISO-8859-1 is used and if no query parameter charset is specified then UTF-8 is used. Usage example:
```java
RestAssured.config = RestAssured.config().encoderConfig(encoderConfig().defaultContentCharset("US-ASCII"));
```

You can also specify which encoder charset to use for a specific content-type if no charset is defined explicitly for this content-type by using the `defaultCharsetForContentType` method in the [EncoderConfig](http://static.javadoc.io/io.rest-assured/rest-assured/3.0.6/io/restassured/config/EncoderConfig.html). For example:

```java
RestAssured.config = RestAssured.config(config().encoderConfig(encoderConfig().defaultCharsetForContentType("UTF-16", "application/xml")));
```

This will assume UTF-16 encoding for "application/xml" content-types that does explicitly specify a charset. By default "application/json" is specified to use "UTF-8" as default content-type as this is specified by [RFC4627](https://www.ietf.org/rfc/rfc4627.txt).

### Avoid adding the charset to content-type header automatically ###

By default REST Assured adds the charset header automatically. To disable this completely you can configure the `EncoderConfig` like this:

```java
RestAssured.config = RestAssured.config(config().encoderConfig(encoderConfig().appendDefaultContentCharsetToContentTypeIfUndefined(false));
```

## Decoder Config ##
With the [DecoderConfig](http://static.javadoc.io/io.rest-assured/rest-assured/3.0.6/io/restassured/config/DecoderConfig.html) you can set the default response content decoding charset for all responses. This is useful if you expect a different content charset than ISO-8859-1 (which is the default charset) and the response doesn't define the charset in the content-type header. Usage example:
```java
RestAssured.config = RestAssured.config().decoderConfig(decoderConfig().defaultContentCharset("UTF-8"));
```

You can also use the `DecoderConfig` to specify which content decoders to apply. When you do this the `Accept-Encoding` header will be added automatically to the request and the response body will be decoded automatically. By default GZIP and DEFLATE decoders are enabled. To for example to remove GZIP decoding but retain DEFLATE decoding you can do the following:
```java
given().config(RestAssured.config().decoderConfig(decoderConfig().contentDecoders(DEFLATE))). ..
```

You can also specify which decoder charset to use for a specific content-type if no charset is defined explicitly for this content-type by using the "defaultCharsetForContentType" method in the [DecoderConfig](http://static.javadoc.io/io.rest-assured/rest-assured/3.0.6/io/restassured/config/DecoderConfig.html). For example:
  
```java
RestAssured.config = config(config().decoderConfig(decoderConfig().defaultCharsetForContentType("UTF-16", "application/xml")));
 ```
This will assume UTF-16 encoding for "application/xml" content-types that does explicitly specify a charset. By default "application/json" is using "UTF-8" as default charset as this is specified by [RFC4627](https://www.ietf.org/rfc/rfc4627.txt).

## Session Config ##
With the session config you can configure the default session id name that's used by REST Assured. The default session id name is `JSESSIONID` and you only need to change it if the name in your application is different and you want to make use of REST Assured's [session support](#Session_support). Usage:

```java
RestAssured.config = RestAssured.config().sessionConfig(new SessionConfig().sessionIdName("phpsessionid"));
```

## Redirect DSL ##
Redirect configuration can also be specified using the DSL. E.g.
```java
given().redirects().max(12).and().redirects().follow(true).when(). .. 
```

## Connection Config ##
Lets you configure connection settings for REST Assured. For example if you want to force-close the Apache HTTP Client connection after each response. You may want to do this if you make a lot of fast consecutive requests with small amount of data in the response. However if you're downloading (especially large amounts of) chunked data you must not close connections after each response. By default connections are _not_ closed after each response.

```java
RestAssured.config = RestAssured.config().connectionConfig(connectionConfig().closeIdleConnectionsAfterEachResponse());
```

## Json Config ##
[JsonPathConfig](http://static.javadoc.io/io.rest-assured/rest-assured/3.0.6/io/restassured/path/json/config/JsonPathConfig.html) allows you to configure the Json settings either when used by REST Assured or by [JsonPath](http://static.javadoc.io/io.rest-assured/rest-assured/3.0.6/io/restassured/path/json/JsonPath.html). It let's you configure how JSON numbers should be treated.
```java
RestAssured.config = RestAssured.config().jsonConfig(jsonConfig().numberReturnType(NumberReturnType.BIG_DECIMAL))
```

## HTTP Client Config ##
Let's you configure properties for the HTTP Client instance that REST Assured will be using when executing requests. By default REST Assured creates a new instance of http client for each "given" statement. To configure reuse do the following:
```java
RestAssured.config = RestAssured.config().httpClient(httpClientConfig().reuseHttpClientInstance());
```

You can also supply a custom HTTP Client instance by using the `httpClientFactory` method, for example:
```java
RestAssured.config = RestAssured.config().httpClient(httpClientConfig().httpClientFactory(
         new HttpClientConfig.HttpClientFactory() {

            @Override
            public HttpClient createHttpClient() {
                return new SystemDefaultHttpClient();
            }
        }));
```

**Note that currently you need to supply an instance of `AbstractHttpClient`.**

It's also possible to configure default parameters etc.

## SSL Config ##
The [SSLConfig](http://static.javadoc.io/io.rest-assured/rest-assured/3.0.6/io/restassured/config/SSLConfig.html) allows you to specify more advanced SSL configuration such as truststore, keystore type and host name verifier. For example:
```java
RestAssured.config = RestAssured.config().sslConfig(sslConfig().with().keystoreType(<type>).and().strictHostnames());
```

## Param Config ##
[ParamConfig](http://static.javadoc.io/io.rest-assured/rest-assured/3.0.6/io/restassured/config/ParamConfig.html) allows you to configure how different parameter types should be updated on "collision". By default all parameters are merged so if you do:
  
```java
given().queryParam("param1", "value1").queryParam("param1", "value2").when().get("/x"). ...
```
  
REST Assured will send a query string of `param1=value1&param1=value2`. This is not always what you want though so you can configure REST Assured to *replace* values instead:

```java
given().
        config(config().paramConfig(paramConfig().queryParamsUpdateStrategy(REPLACE))).
        queryParam("param1", "value1").
        queryParam("param1", "value2").
when().
        get("/x"). ..
```

REST Assured will now replace `param1` with `value2` (since it's written last) instead of merging them together. You can also configure the update strategy for each type of for all parameter types instead of doing it per individual basis:

```java
given().config(config().paramConfig(paramConfig().replaceAllParameters())). ..
```

This is also supported in the [Spring Mock Mvc Module](#spring-mock-mvc-module) (but the config there is called [MockMvcParamConfig](http://static.javadoc.io/io.restassured/spring-mock-mvc/3.0.6/io/restassured/module/mockmvc/config/MockMvcParamConfig.html).

# Spring Mock Mvc Module #

REST Assured 2.2.0 introduced support for [Spring Mock Mvc](http://docs.spring.io/spring/docs/current/javadoc-api/org/springframework/test/web/servlet/MockMvc.html) using the `spring-mock-mvc` module. This means that you can unit test Spring Mvc Controllers. For example given the following Spring controller:
```java
@Controller
public class GreetingController {

    private static final String template = "Hello, %s!";
    private final AtomicLong counter = new AtomicLong();

    @RequestMapping(value = "/greeting", method = GET)
    public @ResponseBody Greeting greeting(
            @RequestParam(value="name", required=false, defaultValue="World") String name) {
        return new Greeting(counter.incrementAndGet(), String.format(template, name));
    }
}
```
you can test it using [RestAssuredMockMvc](http://static.javadoc.io/io.restassured/spring-mock-mvc/3.0.6/io/restassured/module/mockmvc/RestAssuredMockMvc.html) like this:
```java
given().
        standaloneSetup(new GreetingController()).
        param("name", "Johan").
when().
        get("/greeting").
then().
        statusCode(200).
        body("id", equalTo(1)).
        body("content", equalTo("Hello, Johan!"));  
```
i.e. it's very similar to the standard REST Assured syntax. This makes it really fast to run your tests and it's also easier to bootstrap the environment and use mocks (if needed) than standard REST Assured. Most things that you're used to in standard REST Assured works with RestAssured Mock Mvc as well. For example (certain) configuration, static specifications, logging etc etc. To use it you need to depend on the Spring Mock Mvc module:
```xml
<dependency>
      <groupId>io.rest-assured</groupId>
      <artifactId>spring-mock-mvc</artifactId>
      <version>3.0.6</version>
      <scope>test</scope>
</dependency>
```
Or [download](http://dl.bintray.com/johanhaleby/generic/spring-mock-mvc-3.0.6-dist.zip) it from the download page if you're not using Maven.

## Bootstrapping RestAssuredMockMvc ##

First of all you should statically import methods in:
```java
io.restassured.module.mockmvc.RestAssuredMockMvc.*
io.restassured.module.mockmvc.matcher.RestAssuredMockMvcMatchers.*
```

instead of those defined in

```java
io.restassured.RestAssured.*
io.restassured.matcher.RestAssuredMatchers.*
```

Refer to [static import](#static-imports) section of the documentation for additional static imports.

In order to start a test using RestAssuredMockMvc you need to initialize it with a either a set of Controllers, a MockMvc instance or a WebApplicationContext from Spring. You can do this for a single request as seen in the previous example:

```java
given().standaloneSetup(new GreetingController()). ..
```
or you can do it statically:

```java
RestAssuredMockMvc.standaloneSetup(new GreetingController());
```
If defined statically you don't have to specify any Controllers (or MockMvc or WebApplicationContext instance) in the DSL. This means that the previous example can be written as:
```java
given().
        param("name", "Johan").
when().
        get("/greeting").
then().
        statusCode(200).
        body("id", equalTo(1)).
        body("content", equalTo("Hello, Johan!"));  
```

## Asynchronous Requests ##
As of version `2.5.0` RestAssuredMockMvc has support for asynchronous requests. For example let's say you have the following controller:
```java
@Controller
public class PostAsyncController {

    @RequestMapping(value = "/stringBody", method = POST)
    public @ResponseBody
    Callable<String> stringBody(final @RequestBody String body) {
        return new Callable<String>() {
            public String call() throws Exception {
                return body;
            }
        };
    }
}
```

You can test this like so:

```java
given().
        body("a string").
when().
        async().post("/stringBody").
then().
        body(equalTo("a string"));
```

This will use the default timeout of 1 second. You can change the timeout by using the DSL:
```java
given().
        body("a string").
when().
        async().with().timeout(20, TimeUnit.SECONDS).post("/stringBody").
then().
        body(equalTo("a string"));    
```

It's also possible to configure a default timeout by using the [AsyncConfig](http://static.javadoc.io/io.restassured/spring-mock-mvc/2.4.1/io/restassured/module/mockmvc/config/AsyncConfig.html), for example:

```java
given().
        config(config().asyncConfig(withTimeout(100, TimeUnit.MILLISECONDS))).
        body("a string").
when().
        async().post("/stringBody").
then().
        body(equalTo("a string"));
```

`withTimeout` is statically imported from `io.restassured.module.mockmvc.config.AsyncConfig` and is just a shortcut for creating an `AsyncConfig` with a given timeout. Apply the config globally to apply to all requests:

```java
RestAssuredMockMvc.config = RestAssuredMockMvc.config().asyncConfig(withTimeout(100, TimeUnit.MILLISECONDS));

// Request 1
given().
        body("a string").
when().
        async().post("/stringBody").
then().
        body(equalTo("a string"));

// Request 2
given().
        body("another string").
when().
        async().post("/stringBody").
then().
        body(equalTo("a string"));
```

Both request 1 and 2 will now use the default timeout of 100 milliseconds.

## Adding Request Post Processors ##
Spring MockMvc has support for [Request Post Processors](http://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/test/web/servlet/request/RequestPostProcessor.html) and you can use these in RestAssuredMockMvc as well. For example:
```java
given().postProcessors(myPostProcessor1, myPostProcessor2). ..
```

Note that it's recommended the add `RequestPostProcessors` from `org.springframework.security.test.web.servlet.request.SecurityMockMvcRequestPostProcessors` (i.e. authentication `RequestPostProcessors`) to `auth` instead for better readability (result will be the same):
```java
given().auth().with(httpBasic("username", "password")). ..
```
where httpBasic is statically imported from [SecurityMockMvcRequestPostProcessor](http://docs.spring.io/autorepo/docs/spring-security/current/apidocs/org/springframework/security/test/web/servlet/request/SecurityMockMvcRequestPostProcessors.html).

## Adding Result Handlers ##
Spring MockMvc has support for [Result Handlers](http://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/test/web/servlet/ResultHandler.html) and you can use these in RestAssuredMockMvc as well. For example let's say you want to use the native MockMvc logging:

```java
.. .then().apply(print()). .. 
```

where `print` is statically imported from `org.springframework.test.web.servlet.result.MockMvcResultHandlers`. Note that if you're using REST Assured 2.6.0 or older you used the `resultHandlers` method:

```java
given().resultHandlers(print()). .. 
```
but this was deprected in REST Assured 2.8.0.

## Using Result Matchers ##
Spring MockMvc provides a bunch of [Result Matchers](http://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/test/web/servlet/ResultMatcher.html) that you may find useful. RestAssuredMockMvc has support for these as well if needed. For example let's say that for some reason you want to verify that the status code is equal to 200 using a ResultMatcher:
```java
given().
        param("name", "Johan").
when().
        get("/greeting").
then().
        assertThat(status().isOk()).
        body("id", equalTo(1)).
        body("content", equalTo("Hello, Johan!"));  
```
where `status` is statically imported from `org.springframework.test.web.servlet.result.MockMvcResultMatchers`. Note that you can also use the `expect` method which is the same as `assertThat` but more close to the syntax of native MockMvc.

## Interceptors ##
For more advanced use cases you can also get ahold of and modify the [MockHttpServletRequestBuilder](http://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/test/web/servlet/request/MockHttpServletRequestBuilder.html) before the request is performed. To do this define a [MockHttpServletRequestBuilderInterceptor](http://static.javadoc.io/io.restassured/spring-mock-mvc/3.0.6/io/restassured/module/mockmvc/intercept/MockHttpServletRequestBuilderInterceptor.html) and use it with RestAssuredMockMvc:

```java
given().interceptor(myInterceptor). ..
```

## Specifications ##
Just as with standard Rest Assured you can use [specifications](#specification_re-use) to allow for better re-use. Note that the request specification builder for RestAssuredMockMvc is called [MockMvcRequestSpecBuilder](http://static.javadoc.io/io.restassured/spring-mock-mvc/3.0.6/io/restassured/module/mockmvc/specification/MockMvcRequestSpecBuilder.html). The same [ResponseSpecBuilder](http://static.javadoc.io/io.rest-assured/rest-assured/3.0.6/io/restassured/builder/ResponseSpecBuilder.html) can be used in RestAssuredMockMvc as well though. Specifications can be defined statically as well just as with standard Rest Assured. For example:
```java
RestAssuredMockMvc.requestSpecification = new MockMvcRequestSpecBuilder().addQueryParam("name", "Johan").build();
RestAssuredMockMvc.responseSpecification = new ResponseSpecBuilder().expectStatusCode(200).expectBody("content", equalTo("Hello, Johan!")).build();

given().
        standaloneSetup(new GreetingController()).
when().
        get("/greeting").
then().
        body("id", equalTo(1));
```

## Resetting RestAssuredMockMvc ##
If you've used any static configuration you can easily reset RestAssuredMockMvc to its default state by callin the `RestAssuredMockMvc.reset()` method.

## Spring MVC Authentication ##
Version `2.3.0` of `spring-mock-mvc` supports authentication. For example:
```java
given().auth().principal(..). ..
```
Some authentication methods require Spring Security to be on the classpath (optional). It's also possible to define authentication statically:
```java
RestAssuredMockMvc.authentication = principal("username", "password");
```
where the `principal` method is statically imported from [RestAssuredMockMvc](http://static.javadoc.io/io.restassured/spring-mock-mvc/3.0.6/io/restassured/module/mockmvc/RestAssuredMockMvc.html). It's also possible to define an authentication scheme in a request builder:
```java
MockMvcRequestSpecification spec = new MockMvcRequestSpecBuilder.setAuth(principal("username", "password")).build();
```

### Using Spring Security Test ###

Since version `2.5.0` there's also better support for Spring Security. If you have `spring-security-test` in classpath you can do for example:
```java
given().auth().with(httpBasic("username", "password")). ..
```
where `httpBasic` is statically imported from [SecurityMockMvcRequestPostProcessor](http://docs.spring.io/autorepo/docs/spring-security/current/apidocs/org/springframework/security/test/web/servlet/request/SecurityMockMvcRequestPostProcessors.html). This will apply basic authentication to the request. For this to work you need apply the [SecurityMockMvcConfigurer](http://docs.spring.io/autorepo/docs/spring-security/current/apidocs/org/springframework/security/test/web/servlet/setup/SecurityMockMvcConfigurers.html) to the MockMvc instance. You can either do this manually:
```java
MockMvc mvc = MockMvcBuilders.webAppContextSetup(context).apply(SecurityMockMvcConfigurers.springSecurity()).build();
```

or RESTAssuredMockMvc will automatically try to apply the `springSecurity` configurer automatically if you initalize it with an instance of [AbstractMockMvcBuilder](http://docs.spring.io/spring/docs/current/javadoc-api/org/springframework/test/web/servlet/setup/AbstractMockMvcBuilder.html), for example when configuring a "web app context":
```java
given().webAppContextSetup(context).auth().with(httpBasic("username", "password")). ..
```

Here's a full example:
```java
import io.restassured.module.mockmvc.RestAssuredMockMvc;
import org.junit.After;
import org.junit.Before;
import org.junit.Test;
import org.junit.runner.RunWith;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.test.context.ContextConfiguration;
import org.springframework.test.context.junit4.SpringJUnit4ClassRunner;
import org.springframework.test.context.web.WebAppConfiguration;
import org.springframework.web.context.WebApplicationContext;

import static io.restassured.module.mockmvc.RestAssuredMockMvc.given;
import static org.springframework.security.test.web.servlet.request.SecurityMockMvcRequestPostProcessors.httpBasic;
import static org.springframework.security.test.web.servlet.response.SecurityMockMvcResultMatchers.authenticated;

@RunWith(SpringJUnit4ClassRunner.class)
@ContextConfiguration(classes = MyConfiguration.class)
@WebAppConfiguration
public class BasicAuthExample {

    @Autowired
    private WebApplicationContext context;

    @Before public void
    rest_assured_is_initialized_with_the_web_application_context_before_each_test() {
        RestAssuredMockMvc.webAppContextSetup(context);
    }

    @After public void
    rest_assured_is_reset_after_each_test() {
        RestAssuredMockMvc.reset();
    }

    @Test public void
    basic_auth_example() {
        given().
                auth().with(httpBasic("username", "password")).
        when().
                get("/secured/x").
        then().
                statusCode(200).
                expect(authenticated().withUsername("username"));
    }
}
```

You can also define authentication for all request, for example:
```java
RestAssuredMockMvc.authentication = with(httpBasic("username", "password"));
```
where `with` is statically imported from `io.restassured.module.mockmvc.RestAssuredMockMvc`. It's also possible to use a [request specification](#specifications).

### Injecting a User ###

It's also possible use to of Spring Security test annotations such as [@WithMockUser](http://docs.spring.io/spring-security/site/docs/current/reference/htmlsingle/#test-method-withmockuser) and [@WithUserDetails](http://docs.spring.io/spring-security/site/docs/current/reference/htmlsingle/#test-method-withuserdetails). For example let's say you want to test this controller:

```java
@Controller
public class UserAwareController {

    @RequestMapping(value = "/user-aware", method = GET)
    public
    @ResponseBody
    String userAware(@AuthenticationPrincipal User user) {
        if (user == null || !user.getUsername().equals("authorized_user")) {
            throw new IllegalArgumentException("Not authorized");
        }

        return "Success");
    }
}
```

As you can see the `userAware` method takes a [User](http://docs.spring.io/autorepo/docs/spring-security/current/apidocs/org/springframework/security/core/userdetails/User.html) as argument and we let Spring Security inject it by using the [@AuthenticationPrincipal](http://docs.spring.io/spring-security/site/docs/current/apidocs/org/springframework/security/web/bind/annotation/AuthenticationPrincipal.html) annotation. To generate a test user we could do like this:

```java
@WithMockUser(username = "authorized_user")
@Test public void
spring_security_mock_annotations_example() {
    given().
            webAppContextSetup(context).
     when().
            get("/user-aware").
     then().
            statusCode(200).
            body(equalTo("Success")).
            expect(authenticated().withUsername("authorized_user"));
}
```

Note that it's also possible to not use annotations and instead use a [RequestPostProcessor](http://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/test/web/servlet/request/RequestPostProcessor.html) such as [SecurityMockMvcRequestPostProcessors#user(java.lang.String)](http://docs.spring.io/autorepo/docs/spring-security/4.0.0.RELEASE/apidocs/org/springframework/security/test/web/servlet/request/SecurityMockMvcRequestPostProcessors.html#user(java.lang.String)).

## Note on parameters ##
MockMvc doesn't differentiate between different kinds of parameters so `param`, `formParam` and `queryParam` currently just delegates to param in MockMvc. `formParam` adds the `application/x-www-form-urlencoded` content-type header automatically though just as standard Rest Assured does.

# Scala Support Module #
REST Assured 2.6.0 introduced the [scala-support](http://dl.bintray.com/johanhaleby/generic/scala-support-3.0.6-dist.zip) module that adds an alias to the "then" method defined in the [Response](http://static.javadoc.io/io.rest-assured/rest-assured/3.0.6/io/restassured/response/Response.html) or [MockMvcResponse](http://static.javadoc.io/io.restassured/spring-mock-mvc/3.0.6/io/restassured/module/mockmvc/response/MockMvcResponse.html) called "Then". The reason for this is that `then` might be a reserved keyword in Scala in the future and the compiler issues a warning when using a method with this name. To enable the use of `Then` simply import the `io.restassured.module.scala.RestAssuredSupport.AddThenToResponse` class from the `scala-support` module. For example:

```java
import io.restassured.RestAssured.when
import io.restassured.module.scala.RestAssuredSupport.AddThenToResponse
import org.hamcrest.Matchers.equalTo
import org.junit.Test

@Test
def `trying out rest assured in scala with implicit conversion`() {
  when().
          get("/greetJSON").
  Then().
          statusCode(200).
          body("key", equalTo("value"))
}
```
Note that this is also supported for the [Spring Mock Mvc Module](#spring-mock-mvc-module).

To use it do like this:

#### SBT:
```scala
libraryDependencies += "io.rest-assured" % "scala-support" % "3.0.6"
```

#### Maven:
```xml
<dependency>
    <groupId>io.rest-assured</groupId>
    <artifactId>scala-support</artifactId>
    <version>3.0.6</version>
    <scope>test</scope>
</dependency>
```

#### Gradle:
```xml
testCompile 'io.rest-assured:scala-support:3.0.6'
```

### No build manager:
Download the [distribution file](http://dl.bintray.com/johanhaleby/generic/scala-support-3.0.6-dist.zip) manually.

# Kotlin #
Kotlin is a language developed by [JetBrains](https://www.jetbrains.com/) and it integrates very well with Java and REST Assured. When using it with REST Assured there's one thing that can be a bit annoying. That is you have to escape `when` since it's a reserved keyword in Kotlin. For example:

```kotlin
Test fun kotlin_rest_assured_example() {
    given().
            param("firstName", "Johan").
            param("lastName", "Haleby").
    `when`().
            get("/greeting").
    then().
            statusCode(200).
            body("greeting.firstName", equalTo("Johan")).
            body("greeting.lastName", equalTo("Haleby"))
}
```

To get around this, create an [extension function](https://kotlinlang.org/docs/reference/extensions.html) that creates an alias to `when` called `When`:

```kotlin
fun RequestSpecification.When(): RequestSpecification {
    return this.`when`()
}
```

The code can now be written like this:

```kotlin
Test fun kotlin_rest_assured_example() {
    given().
            param("firstName", "Johan").
            param("lastName", "Haleby").
    When().
            get("/greeting").
    then().
            statusCode(200).
            body("greeting.firstName", equalTo("Johan")).
            body("greeting.lastName", equalTo("Haleby"))
}
```

Notice that we don't need any escaping anymore. For more details refer to [this](http://code.haleby.se/2015/11/06/rest-assured-with-kotlin/) blog post.

# More info #
For more information refer to the [javadoc](http://static.javadoc.io/io.rest-assured/rest-assured/3.0.6/index.html):
  * [RestAssured](http://static.javadoc.io/io.rest-assured/rest-assured/3.0.6/io/restassured/RestAssured.html)
  * [RestAssuredMockMvc Javadoc](http://static.javadoc.io/io.restassured/spring-mock-mvc/3.0.6/io/restassured/module/mockmvc/RestAssuredMockMvc.html)
  * [Specification package](http://static.javadoc.io/io.rest-assured/rest-assured/3.0.6/io/restassured/specification/package-summary.html)

You can also have a look at some code examples:
  * REST Assured [tests](https://github.com/rest-assured/rest-assured/tree/master/examples/rest-assured-itest-java/src/test/java/io/restassured/itest/java)
  * [JsonPathTest](https://github.com/rest-assured/rest-assured/blob/master/json-path/src/test/java/io/restassured/path/json/JsonPathTest.java)
  * [XmlPathTest](https://github.com/rest-assured/rest-assured/blob/master/xml-path/src/test/java/io/restassured/path/xml/XmlPathTest.java)

If you need support then join the [mailing list](http://groups.google.com/group/rest-assured).

For professional support please contact [johanhaleby](https://github.com/johanhaleby).
