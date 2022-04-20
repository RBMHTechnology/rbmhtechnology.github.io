---
layout: post
title: VIND - COMPLEX FIELDS WITH SIMPLE CODE
author: tkurz
tag: vind
---

## Vind - Complex Fields with simple Code
In our previous post, [*Vind - Information Discovery for Java*](https://rbmhtechnology.github.io/Vind-Information-Discovery-for-Java/),
we have already had a glimpse of how to build a basic search application with
*Vind* and how its guts look like. Most of the basic custom search scenarios
would be covered by a simple implementation as the one described before but,
what happens when we face a not-as-dumb use case?

### Complex Fields? flatify them!
Those used to build search applications would have faced already the difficulty
of storing complex models into a search index.
Lets take an example based on previous post and add more complexity to it.
Instead of having just search on blog posts, we want to extend the fulltext
search on the post comments.

#### 1. Model extension
This comment class contains not just the comment text but also other features such as the user or the date.

{% highlight java %}
class Comment {
  //the comment unique identifier
  private String id;

  //the comment text which we want search in
  private String comment;

  //flag property to give more relevance to the Comment
  private Boolean featured;

  //user who posted the comment
  private String userName;

  //date of the comment
  private ZonedDateTime commentTime;

  public String getComment() {
    return this.comment;
  }

  public Boolean getUserName() {
    return this.userName;
  }
}
{% endhighlight %}

And add to the original post a list of comments:

{% highlight java %}
class Post {

  // The @id annotation is obligatory for the post identification within Vind
  @Id
  private String id;

  // The fulltext annotation means: 'use this for fulltext search'.
  // The language enables a language specific handling on indexing an query
  //time. The boost leads to a a better ranking if (parts of) the title matches
  //the query.
  @FullText(language = Language.English, boost = 1.2f)
  private String title;

  // The @Facet annotation enables faceting and filtering on this value
  @Facet
  private String category;

  // A field which is not annotated is not used for fulltext search but
  //stored, so it can be used for sorting
  private ZonedDateTime created;

  //new property storing the comment objects attached to a post
  private List<Comment> comments = new ArrayList<>();
}
{% endhighlight %}

#### 2. Complex fields
The model is ready but *Vind* does not know how to index a field of type
`Comments.class`, as it is not one of the basic types supported. We need to assist
the library by providing an annotation which specifies how to index the complex
field.

Time to decide how to use this comment information to search has arrived! as starter
lets suppose that the target is just to do fulltext search on blog contents and
comment texts. To do so we will instruct *Vind* to index the comment field as
a fulltext *Vind* field by adding the following annotation:

{% highlight java %}
//Definition of a complex field 'flatification'
@ComplexField(
    fullText = @Operator(
      function = FunctionHelpers.GetterFunction.class, 
      fieldName = {"comment"}
    )
)
private List<Comment> comments;
{% endhighlight %}

This will result in an index with a **flattened structure**, in which the
property *comment* of the complex field has been indexed as the *fulltext*
value of the post comments field, allowing us to do full text searches in both
blog and comments at the same time.

#### 3. Advanced Filters
What about filtering those posts which have a featured comment? By default
*Vind* does filtering on the facet values of a field, but this can be
problematic in the case of complex field as we may want to facet by a different
value of the complex field. To allow so, the *advanceFilter* option has been
added to the annotation. This advance filter defines a filter scope which can be
specified on query time.

{% highlight java %}
//Definition of a complex field 'flatification'
@ComplexField(
    fullText = @Operator(
      function = FunctionHelpers.GetterFunction.class, 
      fieldName = {"comment"}
    ),
    facet = @Operator(
      function = FunctionHelpers.GetterFunction.class,
      fieldName = {"userName"}
    ),
    advanceFilter = @Operator(
      function = FunctionHelpers.GetterFunction.class, 
      fieldName = {"featured"}
    )
)
private List<Comment> comments;
{% endhighlight %}

With the previous annotation it would be possible to facet the post based on
the user who had commented on them and filter out those post without featured
comments.

### A dive in the @ComplexField annotation

What else can you do with the complex fields? Lets have a look in detail to the  `@ComplexField` annotation. It supports a set of options which mostly match with
the existing basic *Vind* annotations: *store, facet, suggestion, fullText and
sort* plus the *advanceFilter* option.

{% highlight java %}
@ComplexField(
  fullText = @Operator(
    function = FunctionHelpers.ConcatFunction.class,
    fieldName = {"name", "synonym"}
  ),
  facet = @Operator(
    function = FunctionHelpers.GetterFunction.class,
    fieldName = {"name"}
  ),
  advanceFilter =  @Operator(
    function = FunctionHelpers.GetterFunction.class,
    fieldName = {"id"}
  )
)
private Concept topic;
{% endhighlight %}

Those options allow us to describe, through the annotation, how the complex field will be indexed in any of the possible search use cases (i.e. in the previous annotation example we specify to index the topic name and its synonym for *fulltext* search and just the name for _faceting_ on the topics).

For each of the aforementioned use cases a way to calculate a value is provided
by the following three options:

* **_function_**: it specifies a method to obtain the value to be indexed. **Vind** includes two functions: *GetterFunction* and *ConcatFunction*. The first one returns the value of the field specified in *fieldName* and the second the concatenation, blank space separated, of the values of the fields listed. To add your own function it is as easy as implementing `java.util.function.Function` or extend Vinds [ParameterFunction](https://github.com/RBMHTechnology/vind/blob/master/annotations/src/main/java/com/rbmhtechnology/vind/annotations/util/FunctionHelpers.java).
* **_fieldName_**: a list of names where the function will be applied. Default value is an empty list.
* **_returnType_**: The expected return type of the function, default is *String*.

#### Real case scenarios
*Vind* with its complex fields in particular is a backbone for Information Discovery within the Red Bull Media 
House Digital Asset Management System and other internal Red Bull Media House information systems. If you want to have a look at other use cases where *Vind* and complex fields are in place visit the [Billiltii website](https://billitii.com/) and download the app.

![billitii logo](https://billitii.com/wp-content/themes/billitii_new/images/BiLLiTii.png)

In the *Billitii* app-model a thread (triggered by a user question) holds a set
of answers which have been *flatified* in order to be able to find an already
existing thread containing an answer fitting a new question being asked. This
way the incoming question may be inmediatly redirected to a thread where it has
been answered.
