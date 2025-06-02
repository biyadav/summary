### Error Handling in Spring 

@ExceptionHandler:  is a Spring annotation that provides a mechanism to treat exceptions thrown during execution of handlers (controller operations).
This annotation, if used on methods of controller classes, will serve as the entry point for handling exceptions thrown within this controller only.
ResponseEntityExceptionHandler

```
@RestController
public class FooController {
    //...

    @ResponseStatus(HttpStatus.BAD_REQUEST)
    @ExceptionHandler(CustomException1.class)
    public void handleException() {
        // ...
    }
}
```

@ControllerAdvice is an annotation in Spring and, as the name suggests, is “advice” for multiple controllers. It enables the application of a single ExceptionHandler to multiple controllers.
The subset of controllers affected can be defined by using the following selectors on @ControllerAdvice: annotations(), basePackageClasses(), and basePackages(). ControllerAdvice is applied
globally to all controllers if no selectors are provided

```
@RestControllerAdvice
public class MyGlobalExceptionHandler {
    @ResponseStatus(HttpStatus.BAD_REQUEST)
    @ExceptionHandler(CustomException1.class)
    public void handleException() {
        // ...
    }
}
```

Extending  class ResponseEntityExceptionHandler: 
A convenient base class for @ControllerAdvice classes that wish to provide centralized exception handling across all @RequestMapping methods through @ExceptionHandler methods.
This base class provides an @ExceptionHandler for handling standard Spring MVC exceptions that returns a ResponseEntity to be written with message converters. 
This is in contrast to DefaultHandlerExceptionResolver which returns a ModelAndView instead.

if we dont use  @@RestControllerAdvice then it will return view and model but we dont want in rest app. 

If we  extend ResponseEntityExceptionHandler class there’s no need to add the @RestControllerAdvice annotation to the class since all methods return a ResponseEntity,
so we can use vanilla @ControllerAdvice annotation .

```
@Order(Ordered.HIGHEST_PRECEDENCE)
@ControllerAdvice
public class RestExceptionHandler extends ResponseEntityExceptionHandler {
  
   //other exception handlers
  
   @ExceptionHandler(Throwable.class)
   @ResponseBody
   protected ResponseEntity<ErrorResponse> handleException(
           HttpServletRequestrequest , HttpServletResponse response) {
       ApiError apiError = new ApiError(NOT_FOUND);
       apiError.setMessage(ex.getMessage());
       apiError.setTracingId(ex.getTracer());
       return buildResponseEntity(apiError);
   }

   @ExceptionHandler({ 
        IllegalArgumentException.class, 
        IllegalStateException.class
    })
   @ResponseBody
    ResponseEntity<Object> handleConflict(RuntimeException ex, WebRequest request) {
        String bodyOfResponse = "This should be application specific";
        return super.handleExceptionInternal(ex, bodyOfResponse, 
          new HttpHeaders(), HttpStatus.CONFLICT, request);
    }

    @Override
    protected ResponseEntity<Object> handleHttpMediaTypeNotAcceptable(
      HttpMediaTypeNotAcceptableException ex, HttpHeaders headers, HttpStatusCode status, WebRequest request {
        // ... (customization, maybe invoking the overridden method)
    }

 //  you can override another default handling like handleHttpMessageNotReadable 
}

```
