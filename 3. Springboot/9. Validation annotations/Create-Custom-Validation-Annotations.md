## Dependency

```xml
<dependency>
    <groupId>jakarta.validation</groupId>
    <artifactId>jakarta.validation-api</artifactId>
</dependency>
```

---

## Birorta field, o`zgaruvchini validate qiluvchi annotation yaratish:

### ValidLogin
```java
import jakarta.validation.Constraint;
import jakarta.validation.Payload;

import java.lang.annotation.ElementType;
import java.lang.annotation.Retention;
import java.lang.annotation.RetentionPolicy;
import java.lang.annotation.Target;

@Constraint(validatedBy = ValidLoginValidator.class)
@Target({ElementType.FIELD, ElementType.PARAMETER})
@Retention(RetentionPolicy.RUNTIME)
public @interface ValidLogin {
    String message() default "Error. login is not valid.";

    Class<?>[] groups
            () default {};

    Class<? extends Payload>[] payload() default {};
}
```

### ValidLoginValidator
```java
import jakarta.validation.ConstraintValidator;
import jakarta.validation.ConstraintValidatorContext;

public class ValidLoginValidator implements ConstraintValidator<ValidLogin,String> {
    private final UserRepository userRepository;

    public ValidLoginValidator(UserRepository userRepository) {
        this.userRepository = userRepository;
    }

    @Override
    public boolean isValid(String login, ConstraintValidatorContext constraintValidatorContext) {
        if (!userRepository.existsUserByLogin(login)){
            String customMessage = "Error: Invalid login or password.";
            constraintValidatorContext.disableDefaultConstraintViolation();
            constraintValidatorContext
                    .buildConstraintViolationWithTemplate(customMessage)
                    .addConstraintViolation();
            return false;
        }
        return true;
    }
}
```

---

### Class levelda foydalanish mumkin bo'lgan annotation yaratish
```java
@Target(ElementType.TYPE)
```
