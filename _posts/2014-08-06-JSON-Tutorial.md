---
layout: post
title: JSON Tutorial
category: tech
tags: json org.json
year: 2014
month: 8
day: 6
published: true
summary: A quick primer on JSON APIs
---

JSON data format is widely used nowadays. In order to use it with Java, several libraries are available. In this blog post, we will take a look at handling JSON data using the [org.json](http://www.json.org/java/index.html) library.

 - [json.simple](http://code.google.com/p/json-simple)
 - [Gson](http://code.google.com/p/google-gson)
 - [Jackson](http://wiki.fasterxml.com/JacksonHome) - Most feature rich and best performance
 - [org.json](http://www.json.org/java/index.html)  - Lightweight

This tutorial covers handling JSON using the [org.json](http://www.json.org/java/index.html) library.

We will see how you can use classes from org.json package to build JSON objects and also to parse JSON data.
<strong>org.json</strong> package is widely used in Android and avoids dependency on other heavyweight packages like Jackson and such.
Lets dive straight in to some code.

#### 1. Creating JSON Objects
The main class to use here is JSONObject.
{% highlight java %}

JSONObject json = new JSONObject();
json.put("from", "alice@example.com");
json.put("to", "bob@example.com");
json.put("type", "call");
System.out.println("Output = " + json)
{% endhighlight %}


:key: Output

    {"to":"bob@example.com","from":"alice@example.com","type":"call"}

Another code sample
{% highlight java  %}
JSONObject json = new JSONObject();
json.put("from", "alice@example.com");
json.put("to", "bob@example.com");
json.put("type", "call");
json.put("numParams", 2);
json.put("params", new JSONArray().put("v1").put("v2"));

json.append("params", "v3");
json.increment("numParams");

System.out.println("Output = " + json);
{% endhighlight %}

:key: Output

    {"to":"bob@example.com","numParams":3,"params":["v1","v2","v3"],"from":"alice@example.com","type":"call"}

#### 2. Parsing from JSON in string form
If we pass the last output given above to the function below, here is how we can extract the values by first converting the string to JSONObject and then invoking the various methods on it.
We use the class JSONTokener which takes a source string and extracts characters and tokens from it.
{% highlight java  %}
private static void readJSONString(String data) {
  JSONObject json = new JSONObject(new JSONTokener(data));
  String from = json.getString("from");
  String to = json.getString("to");
  int numParams = json.getInt("numParams");

  System.out.println("From: " + from + " To: " + to + " NumParams: " + numParams);
  JSONArray arr = json.getJSONArray("params");
  System.out.print("Params: ");
  for (int i = 0; i < arr.length(); i++) {
    System.out.print(arr.getString(i) + " ");
  }
}
{% endhighlight %}
:key: Output

	From: alice@example.com To: bob@example.com NumParams: 3
	Params: v1 v2 v3


#### 3. Rapidly producing JSON
Both JSONWriter and JSONStringer allow you to construct a JSON object with JSONWriter additionally taking a Writer object which outputs the contents to a stream.
{% highlight java  %}
private static void constructJSON() throws IOException {
  Writer myWriter = new BufferedWriter(new FileWriter(new File("json.out")));
  new JSONWriter(myWriter)
    .object()
      .key("from").value("alice@example.com")
      .key("to").value("bob@test.com")
      .key("params")
        .array()
          .value("v1")
          .value("v2")
          .value("v3")
        .endArray()
    .endObject();

  myWriter.flush();
  myWriter.close();
  System.out.println("Contents written");
}
{% endhighlight %}

:key: Output: Contents of the file json.out

	{"from":"alice@example.com","to":"bob@test.com","params":["v1","v2","v3"]}

#### 4. Maven dependency
org.json is available at Maven central repository, just declares following dependency in your pom.xml file.
{% highlight java  %}
<dependency>
  <groupId>org.json</groupId>
  <artifactId>json</artifactId>
  <version>20140107</version>
</dependency>
{% endhighlight %}


#### 5. Normalization

Suppose you had a JSON string that you want to normalize such that you can save all the values in a `Map`.

:point_right: Input:

{% highlight javascript  %}
{
  "store": {
    "book": [
      {
        "author": "NoOne",
        "title": "SOWhat",
        "category": "reference",
        "price": 8.95
      },
      {
        "author": "SomeOne",
        "title": "HiThere",
        "category": "fiction",
        "price": 8.99,
        "isbn": "0-5667666-3"
      }
    ],
    "bicycle": {
      "price": 19.95,
      "color": "red"
    }
  },
  "expensive": 10
}
{% endhighlight %}

:key: Output

    store.book.author:NoOne
    store.book.title:SOWhat
    store.book.category:reference
    store.book.price:8.95
    store.book.author:SomeOne
    store.book.title:HiThere
    store.book.category:fiction
    store.book.price:8.99
    store.book.isbn:0-5667666-3
    store.bicycle.price:19.95
    store.bicycle.color:red
    expensive:10


Here is the code that can do that.
{% highlight java  %}
import java.util.Iterator;

import org.json.JSONArray;
import org.json.JSONException;
import org.json.JSONObject;


public class JSONNormalize {

  private static void listJson(JSONObject json) throws JSONException {
    listJSONObject("", json);
  }

  private static void listObject(String parent, Object data) throws JSONException {
    if (data instanceof JSONObject) {
      listJSONObject(parent, (JSONObject)data);
    } else if (data instanceof JSONArray) {
      listJSONArray(parent, (JSONArray) data);
    } else {
      listPrimitive(parent, data);
    }    
  }

  private static void listJSONObject(String parent, JSONObject json) throws JSONException {
    Iterator it = json.keys();
    while (it.hasNext()) {
      String key = (String)it.next();
      Object child = json.get(key);
      String childKey = parent.isEmpty() ? key : parent + "." + key;
      listObject(childKey, child);
    }
  }

  private static void listJSONArray(String parent, JSONArray json) throws JSONException {
    for (int i = 0; i < json.length(); i++) {
      Object data = json.get(i);
      listObject(parent, data);
    }
  }

  private static void listPrimitive(String parent, Object obj) {
    System.out.println(parent + ":"  + obj);
  }

  public static void main(String[] args) throws JSONException {
    String data = "{\"store\":{\"book\":[{\"category\":\"reference\",\"author\":\"NoOne\",\"title\":\"SOWhat\",\"price\":8.95},{\"category\":\"fiction\",\"author\":\"SomeOne\",\"title\":\"HiThere\",\"isbn\":\"0-5667666-3\",\"price\":8.99},],\"bicycle\":{\"color\":\"red\",\"price\":19.95}},\"expensive\":10}";
    JSONObject json = new JSONObject(data);    
    System.out.println(json.toString(2));
    listJson(json);
  }

}

{% endhighlight %}


#### 5. Performance
Comparisons between the various libraries can be found <a href="http://eclipsesource.com/blogs/2013/04/18/minimal-json-parser-for-java/" title="here" target="_blank">here</a>
