      
## Views separation in JSON Response
Jackson Serialization View Support

It can sometimes be useful to filter contextually the object that will be serialized to the HTTP response body. In order to provide such capability, Spring MVC has built-in support for rendering with Jackson’s Serialization Views.

To use it with an @ResponseBody controller method or controller methods that return ResponseEntity, simply add the @JsonView annotation with a class argument specifying the view class or interface to be used:

@RestController
public class UserController {

    @GetMapping("/user")
    @JsonView(User.WithoutPasswordView.class)
    public User getUser() {
        return new User("eric", "7!jd#h23");
    }
}

public class User {

    public interface WithoutPasswordView {};
    public interface WithPasswordView extends WithoutPasswordView {};

    private String username;
    private String password;

    public User() {
    }

    public User(String username, String password) {
        this.username = username;
        this.password = password;
    }

    @JsonView(WithoutPasswordView.class)
    public String getUsername() {
        return this.username;
    }

    @JsonView(WithPasswordView.class)
    public String getPassword() {
        return this.password;
    }
}
