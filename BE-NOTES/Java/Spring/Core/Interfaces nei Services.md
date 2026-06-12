## 4. Interface nei Service  
  
#spring #architecture #best-practices  
  
> [!info] Pattern  
> Separare ==contratto (interface)== da ==implementazione (classe)== nei layer di servizio  
  
### Struttura del Progetto  
  
```  
src/main/java/com/example/  
├── controller/  
│   └── UserController.java  
├── service/  
│   ├── UserService.java  
│   └── UserServiceImpl.java  
├── repository/  
│   └── UserRepository.java  
└── model/  
    └── User.java  
```  
  
### Interfaccia Service  
  
```java  
package com.example.service;  
  
import com.example.model.User;  
import java.util.List;  
import java.util.Optional;  
  
public interface UserService {  
    List<User> findAll();  
    Optional<User> findById(Long id);  
    User save(User user);  
    void deleteById(Long id);  
    User updateUser(Long id, User user);  
}  
```  
  
### Implementazione Service  
  
```java  
package com.example.service;  
  
import com.example.model.User;  
import com.example.repository.UserRepository;  
import org.springframework.stereotype.Service;  
import org.springframework.transaction.annotation.Transactional;  
import lombok.RequiredArgsConstructor;  
  
@Service  
@RequiredArgsConstructor  
@Transactional  
public class UserServiceImpl implements UserService {  
      
    private final UserRepository userRepository;  
      
    @Override  
    public List<User> findAll() {  
        return userRepository.findAll();  
    }  
      
    @Override  
    public Optional<User> findById(Long id) {  
        return userRepository.findById(id);  
    }  
      
    @Override  
    public User save(User user) {  
        if (user.getEmail() == null || user.getEmail().isEmpty()) {  
            throw new IllegalArgumentException("Email is required");  
        }  
        return userRepository.save(user);  
    }  
      
    @Override  
    public void deleteById(Long id) {  
        if (!userRepository.existsById(id)) {  
            throw new RuntimeException("User not found");  
        }  
        userRepository.deleteById(id);  
    }  
      
    @Override  
    public User updateUser(Long id, User user) {  
        User existing = userRepository.findById(id)  
            .orElseThrow(() -> new RuntimeException("User not found"));  
          
        existing.setName(user.getName());  
        existing.setEmail(user.getEmail());  
          
        return userRepository.save(existing);  
    }  
}  
```  
  
### Controller  
  
> [!important] Best Practice  
> Inietta sempre l'==interfaccia==, mai l'implementazione!  
  
```java  
package com.example.controller;  
  
import com.example.model.User;  
import com.example.service.UserService;  
import org.springframework.web.bind.annotation.*;  
import lombok.RequiredArgsConstructor;  
  
@RestController  
@RequestMapping("/api/users")  
@RequiredArgsConstructor  
public class UserController {  
      
    private final UserService userService;  
      
    @GetMapping  
    public List<User> getAllUsers() {  
        return userService.findAll();  
    }  
      
    @GetMapping("/{id}")  
    public User getUserById(@PathVariable Long id) {  
        return userService.findById(id)  
            .orElseThrow(() -> new RuntimeException("User not found"));  
    }  
      
    @PostMapping  
    public User createUser(@RequestBody User user) {  
        return userService.save(user);  
    }  
      
    @PutMapping("/{id}")  
    public User updateUser(@PathVariable Long id, @RequestBody User user) {  
        return userService.updateUser(id, user);  
    }  
      
    @DeleteMapping("/{id}")  
    public void deleteUser(@PathVariable Long id) {  
        userService.deleteById(id);  
    }  
}  
```  
  
### Vantaggi  
  
> [!success] Benefici  
> 1. **Testabilità**: Facile creare mock  
> 2. **Flessibilità**: Sostituire implementazioni  
> 3. **Manutenibilità**: Separazione chiara  
> 4. **Multiple implementazioni**: Prod vs test  
  
#### Implementazioni Multiple  
  
```java  
@Service  
@Profile("prod")  
public class UserServiceImpl implements UserService {  
    // Database reale  
}  
  
@Service  
@Profile("test")  
public class UserServiceMock implements UserService {  
    // Dati in memoria  
    @Override  
    public List<User> findAll() {  
        return Arrays.asList(  
            new User(1L, "Test User", "test@example.com")  
        );  
    }  
}  
```  
  
> [!check] Best Practices  
> - Naming: `UserService` (interface), `UserServiceImpl` (impl)  
> - Inietta sempre l'interfaccia  
> - Logica business nel service  
> - `@Transactional` sull'implementazione  
> - Interface e impl nello stesso package  
  