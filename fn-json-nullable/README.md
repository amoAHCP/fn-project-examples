# fn json with null values example

## introduction
Dealing with Json values in Fn in basically pretty simple by defining an Object input and output:

```java
public class HelloFunction {


    public static class Input {
        public String name;
    }

    public static class Result {
        public String salutation;
    }

    public Result handleRequest(Input input) {
        Result result = new Result();
        result.salutation = "Hello " + input.name;
        return result;
    }

}
```
You can even configure the Jackson ObjectMapper for special purposes:

```java
public class HelloFunction {
   @FnConfiguration
    void configureObjectMapper(RuntimeContext ctx){
        ObjectMapper om = new ObjectMapper();
        om.setSerializationInclusion(Include.NON_NULL);

        ctx.setAttribute("com.fnproject.fn.runtime.coercion.jackson.JacksonCoercion.om",om);

    }
    ...
}
```

While this is fine in most cases, I see two issues with this solution:

1. If you dont provide an Input to you function, you'll get an Exception
2. You need to use Jackson

## alternate solution
With the solution in this example you can control the input (dealing with null values and other inputs ), as well as using othe Json parsers than Jackson. The solution is to use an InputBinding:


```java
public class SimpleJsonFunction {
  public static class Result {

    public String salutation;
  }

  public Result handleRequest(@InputBinding(coercion = InputMapper.class) Input input) {
    Result result = new Result();
    result.salutation = "";
    if (input == null || input.name==null) {
      return result;
    }

    result.salutation = "Hello " + input.name;
    return result;
  }
}


public class InputMapper implements InputCoercion<Input> {

public final static ObjectMapper objectMapper = new ObjectMapper();

  @Override
  public Optional<Input> tryCoerceParam(InvocationContext invocationContext, int arg,
      InputEvent inputEvent, MethodWrapper methodWrapper) {

    String jsonValue = inputEvent.consumeBody(is -> {
      try {
        return IOUtils.toString(is, StandardCharsets.UTF_8.toString());
      } catch (IOException e) {
        throw new IllegalArgumentException("Error reading input as string", e);
      }
    });

    if (jsonValue != null && !jsonValue.isEmpty()) {

      try {
        return Optional.of(objectMapper.readValue(jsonValue, Input.class));
      } catch (IOException e) {
        e.printStackTrace();
      }
    } else {
      return Optional.of(new Input());
    }
    return Optional.<Input>empty();
  }
}
```
