---
title: Web Service
layout: default
---
# Web Service

## Software Architecture

We are using the DTO (Data Transfert Object) and the DAO (Data Access Object) design patterns.

### Model
A model is used to represent a concept manipulated in the web service. 

It only contains attributes and eventually business oriented methods. It doesn't contain any storage or technological references. 

### DTO

A DTO is a object used to carry a model between the web service and the clients.

![dto-schema](img/dto.png)

A DTO does not have any behavior except for storage, retrieval, serialization and deserialization of its own data (mutators, accessors, parsers and serializers). In other words, DTOs are simple objects that should not contain any business logic.

### DAO

A DAO is an object that provides an abstract interface to some type of database or other persistence mechanism. 

By mapping application calls to the persistence layer, the DAO provides some specific data operations without exposing details of the database. This isolation supports the single responsibility principle. It separates what data access the application needs, in terms of domain-specific objects and data types (the public interface of the DAO), from how these needs can be satisfied with a specific DBMS, database schema, etc. (the implementation of the DAO).

![dao-schema](img/dao.png)

It implements the CRUD functions to the different data sources (e.g. triplestore, relational database, nosql, other web services, etc.)
A DAO only manipulates models and don't know anything about DTO objects.

Therefore before calling a DAO, the DTO objects recieved must be converted into models. These models are sent to the DAO. The DAO query the storage and eventually returns models.

These models are converted into DTO objects. They take part in the data returned by the web service to the client.

![dao-dto-model](img/web_service_architecture.png)

## RDF resource definition
To represent and manipulate the triplestore resources (instances or concepts), we have created models, DAO and DTO.

### RDF resource definition model
To manipulate a resource (instance or concept), we extends the RdfResourceDefinition model, such as the following example.

![rdf-resource-definition-model](img/rdfResourceDefinition.png)

### RDF resource definiton DTO
A few services returns more or less the same RDF resource definition JSON format. We have created a few DTOs which extends RdfResourceDefinitionDTO. In the following example, the RadiometricTargetDTO extends RdfResourceDefinitionDTO and RadiometricTargetPostDTO extends RdfResourceDefinitionPostDTO. We have created a specific package to group the DTOs concerning the radiometric targets.

![rdf-resource-definition-dto](img/rdfResourceDefinitionDTO.png)

### Property DAO
To manipulate the properties extracted from a semantic triplestore, we use the PropertyDAO class.

## How to create new services?
You can base your new service on the Provenance and Radiometric Target services.
1. Create the model of the concept manipulated by the service if not existing (e.g. `opensilex.service.view.model.provenance.Provenance`).
2. Create the DTO that your service will manipulate. It must at least extends `AbstractVerifiedClass` (e.g. `opensilex.service.resources.dto.provenance.ProvenanceDTO`).
3. Create the service class extending `ResourceService` in the `resources` package (e.g. `opensilex.service.resources.ProvenanceResourceService`).
4. Create or complete the DAOs corresponding to the models your service manipulates (e.g. `opensilex.service.dao.ProvenanceDAO`). The DAO must at least extends the `DAO` class. The only functions of your DAO class accessible by your resource service class must by one of those define in the `DAO.java` class (create, delete, update, find, findById).

## Java Bean Validation (JSR 380)

When you create a new service, you must use the [Java Bean Validation](https://beanvalidation.org/2.0/spec/#builtinconstraints) to validate the input parameters. The JSR 380 uses annotations to validate parameters.

### Use an existing constraint validation

You can add a verification on the DTO for the POST and the ResourceService classes for the GET parameters. To do that, you must add an annotation such as `@Required`.

In the folowing example, the parameter `url` of the method `get` of the `ExampleResourceService` is required (`@Required`) and must be a valid url (`@Url`).

For the `post` method, we needs to validate the list of `ExampleDTO`. The `@Valid` annotation is used to validate the attributes of a complex object. In the following example, the `@Valid` is used to validate the content of the list `examples` of the `post` method, in `ExampleResourceService`.

The `ExampleDTO` class contains the constraint validation of its attributes. The constraint annotations are in our case applied to the `getter` methods. For example, the url attribute is required (`@Required`) and must be a valid url (`@Url`). The secondExample parameter is a complex type. To validate it, we use the `@Valid` annotation as the explained earlier.



```java
public class ExampleResourceService {

  @GET
  public Response get(
    @QueryParam("url") @Required @URL String url
  ) {
    ...
  }

  @POST
  public Response post(ArrayList<ExampleDTO> @Valid examples) {
    ...
  }
}

public class ExampleDTO {
  String url;
  SecondExample secondExample;

  @URL
  @Required
  public String getUrl() {
    return url;
  }

  @Valid
  public SecondExample getSecondExample() {
    return secondExample;
  }
}
```

### Create a custom constraint validation

You can create custom constraint validation. For examplen the `@URL` is define in the following code blocks.

In our case, create the custom validation annotation in the package `service.resources.validation.validators`.

```java
public class URLListValidator implements ConstraintValidator<URL, List<String>> {

    @Override
    public void initialize(URL constraintAnnotation) {
    }

    @Override
    public boolean isValid(List<String> valueList, ConstraintValidatorContext context) {
        if (valueList == null) {
            return true;
        }

        Boolean allValid = true;
        String lastUrlCheck = null;

        // loop over string
        for (String url : valueList) {
            final java.net.URL finalUrl;
            lastUrlCheck = url;
            try {
                finalUrl = new java.net.URL(url);
            } catch (MalformedURLException e1) {
                allValid = false;
            }
        }

        // if not valid returns the last false index
        if (!allValid) {
            context.disableDefaultConstraintViolation();
            context.buildConstraintViolationWithTemplate(
                    "The string at the index [" + valueList.indexOf(lastUrlCheck) + "] is not an URL"
            ).addConstraintViolation();
        }

        return allValid;
    }
}
```

Create the custom validation annotation interface in the package `service.resources.validation.interfaces`.

```java
@Target(value={METHOD,FIELD,ANNOTATION_TYPE,CONSTRUCTOR,PARAMETER}) //The annotation can be applied to a method, a parameter, etc.
@Retention(RUNTIME) //The annotation should be available for reflection at runtime. Example: @Deprecated (see https://docs.oracle.com/javase/7/docs/api/java/lang/annotation/RetentionPolicy.html)
@Constraint(validatedBy = {URLValidator.class,URLListValidator.class}) //The constraint is validated by the URLValidator and the URLListValidator (to validate lists of url)
public @interface URL {
    String message() default "is not an URL"; //The message element value is used to create the error message. (see https://beanvalidation.org/2.0/spec/#validationapi-message)

    Class<?>[] groups() default {}; //Every constraint annotation must define a groups element that specifies the processing groups with which the constraint declaration is associated. The type of the groups parameter is Class<?>[] (see https://beanvalidation.org/2.0/spec/#constraintsdefinitionimplementation-constraintdefinition-properties-groups)

    Class<? extends Payload>[] payload() default {}; //The annotation should be available for reflection at runtime. (see https://beanvalidation.org/2.0/spec/#constraintsdefinitionimplementation-constraintdefinition-properties-payload)

    /**
     * This code block is used to apply the validator to a list of elements.
     */
    @Target(value = {METHOD,FIELD,ANNOTATION_TYPE,CONSTRUCTOR,PARAMETER})
    @Retention(RUNTIME)
    @Documented //Apply @Documented when defining an annotation, to ensure that classes using your annotation show this in their generated JavaDoc.
    public @interface List {
        public URL[] value();
    }
}
```
You can then use your custom constraint validator such as the [built in constraints](https://beanvalidation.org/2.0/spec/#builtinconstraints) by using `@URL`.

When you use a built in constraint validation, please check if the annotation can be applied on lists (not all built in constraint are implemeted for lists).

Several annotations don't work on `List`. For example, `@Email` only works on `String`. You can create your own validator as showed above or use type level annotations. In Bean Validation (2.0), all provided constraints have `ElementType.TYPE_USE`. For more information [reads the definition](https://docs.oracle.com/javase/8/docs/api/java/lang/annotation/ElementType.html) .
For example, if the annotation type `@Email` is meta-annotated with @target(ElementType.TYPE_USE), then  `ArrayList<@Email String>` could be treated by a type checker as indicating that all string of this ArrayList are well formatted e-mails.

The example below shows how to work with list of emails using `@Email`.

```java
public ArrayList<@Email String> getUsersEmails() {
       return usersEmails;
   }
```

For more examples see the others custom constraints validators of the web services in the packages `service.resources.validation.validators` and `service.resources.validation.interfaces`.
